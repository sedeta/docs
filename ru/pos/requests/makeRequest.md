## Сделать кросс-доменный запрос

<details open>
<summary>Пример запроса</summary>

```javascript
    Poster.makeRequest('http://mywebsite.com', {
        headers: [
            'Content-Type: application/json'
        ],
        method: 'post',
        data: { foo: 'bar' },
        timeout: 10000
    }, (answer) => {
        if (answer && Number(answer.status) === 200) {
            console.log(answer.result);
        }
    });
```

</details>


Метод делает HTTP-запросы на любой удаленный адрес. Запросы проксируются через бекенд Poster.

### Аргументы

Аргумент | Описание
-------- | --------
1-й, url | URL адрес удаленного ресурса
2-й, options | Объект с настройками запроса
3-й, callback | Функция-обработчик ответа


Объект `options` содержит следующие параметры:

Параметр | Описание
-------- | --------
headers | Опциональный параметр, массив заголовков которые нужно передать в запросе
method | Тип запроса: `get`, `post`, `put`, `delete`. По умолчанию `get`.
data | Тело запроса, передается только в `post` и `put` запросах. Чтобы отправить GET-параметр, передавайте его прямо в ссылке   
timeout | Опциональный параметр, максимальное время выполнения запроса, указывается в миллисекундах. По умолчанию не ограничен по времени.
localRequest | Опциональный параметр, передавайте `true` — когда нужно сделать запрос на устройство в локальной сети. По умолчанию `false`. 


### Ответ

В `callback`-функцию приходит ответ от сервера, объект содержит следующие параметры:
 
Параметр | Описание
-------- | --------
result | Тело ответа от сервера, `false` — если запрос отпал по таймауту или мы не смогли пропарсить JSON 
code | HTTP-статус ответа. Возвращает: `0` – если запрос отпал по таймауту, или устройство не подключено к интернету, `1` — если в ответе не валидный JSON.


### Подпись

Если вы работаете с чувствительными операциями: оплаты, списание или начисление бонусов, вашему приложению нужно знать что запрос пришел из надежного источника. 
Для этого используется подпись запроса. Метод `Poster.makeRequest` проксирует запросы через сервер Poster, где запрос подписывается и отправляется к вам на сервер. 

!> Запросы в локальной сети не подписываются

Запрос подписывается таким алгоритмом. По очереди конкатенируют:

1. Ссылку запроса, полностью вместе с GET параметрами
2. JSON строка созданная из тела запроса, если GET запрос то тело не участвует в подписи
3. Время когда отправлен запрос, передается в заголовке `X-Poster-Time` в формате Unix Timestamp
4. Application secret, это ключ который вы получили в момент регистрации приложения
 
Полученная строка хешируется методом md5 и отправляется вместе с запросом в заголовке `X-Poster-Signature`. 

Для вашего удобства, к каждому запросу проставляются заголовки:

 - `X-Poster-Url` — название аккаунта 
 - `X-Poster-Spot-Id` — id заведения из которого отправили запрос
 - `X-Poster-Tablet-Id` — id терминала из которого отправили запрос


<details open>
<summary>Пример функции для проверки подписи</summary>

<!-- tabs:start -->

#### ** JS **

```javascript
var md5 = require('md5');

function checkSign(url, headers, body, secret) {
    var signStr = url + (body ? JSON.stringify(body) : "") + headers['x-poster-time'] + secret;
    var signature = md5(signStr);

    return signature === headers['x-poster-signature'];
}
```

#### ** PHP **

```php
<?php       

function checkSign($url, $headers, $body, $secret) {
    $signatureStr = $url . ($body ? json_encode($body) : "") . $headers['X-Poster-Time'] . $secret;
    $signature = md5($signatureStr);
    
    return $headers['X-Poster-Signature'] == $signature;
}
```

<!-- tabs:end -->

</details>
