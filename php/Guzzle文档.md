##### 需求

- `>=`PHP 5.5.0 
- 使用PHP的流， `allow_url_fopen` 必须在php.ini中启用。
- 要使用cURL，你必须已经有版本cURL >= 7.19.4，并且编译了OpenSSL 与 zlib。
- 如果没有安装cURL，Guzzle处理HTTP请求的时候不再使用cURL，而是使用PHP流处理，或者你也可以提供自己的发送HTTP请求的处理方式

##### 安装

```php
composer require guzzlehttp/guzzle:~6.0
```



##### 使用

> 创建一个对象

```php
$client = new Client(['base_uri' => 'http://127.0.0.1:9900/']);
```

> 发起一个请求

```php
$response = $client->request('GET', 'test');
```

> 响应头信息

```php
$response->getHeaders()
```

>响应的主体

```php
$response->getBody()
```

##### 简单的请求

```php
$response = $client->get('http://127.0.0.1:9900/get');
$response = $client->delete('http://127.0.0.1:9900/delete');
$response = $client->head('http://127.0.0.1:9900/get');
$response = $client->options('http://127.0.0.1:9900/get');
$response = $client->patch('http://127.0.0.1:9900/patch');
$response = $client->post('http://127.0.0.1:9900/post');
$response = $client->put('http://127.0.0.1:9900/put');
```

##### 异步请求

```php
$promise = $client->getAsync('http://127.0.0.1:9900/get');
$promise = $client->deleteAsync('http://127.0.0.1:9900/delete');
$promise = $client->headAsync('http://127.0.0.1:9900/get');
$promise = $client->optionsAsync('http://127.0.0.1:9900/get');
$promise = $client->patchAsync('http://127.0.0.1:9900/patch');
$promise = $client->postAsync('http://127.0.0.1:9900/post');
$promise = $client->putAsync('http://127.0.0.1:9900/put');
```

##### 并发请求

```php
use GuzzleHttp\Client;
use GuzzleHttp\Promise;

$client = new Client(['base_uri' => 'http://127.0.0.1:9900/']);
// 启动多个请求
$promises = [
    'image' => $client->getAsync('/image'),
    'png'   => $client->getAsync('/image/png'),
    'jpeg'  => $client->getAsync('/image/jpeg'),
    'webp'  => $client->getAsync('/image/webp')
];

// 等待请求的结果
$results = Promise\unwrap($promises);

// 请求的结果
echo $results['image']->getHeader('Content-Length');
echo $results['png']->getHeader('Content-Length');
```

##### 表单的请求

```php
// application/x-www-form-urlenc 
$response = $client->request('POST', 'http://127.0.0.1:9900/user/', [
    'form_params' => [
        'username' => 'xiaowu',
        'password' => md5(123456),
    ]
]);

// multipart
$response = $client->request('POST', 'http://127.0.0.1:9900/user/', [
    'multipart' => [
        [
            'username'     => 'xiaowu',
            'password' => md5(123456),
        ],
        [
            'name'     => 'file_name',
            'contents' => fopen('/path/to/file', 'r')
        ]
    ]
]);
```

##### 异常

```php
use GuzzleHttp\Exception\RequestException;
```

- `GuzzleHttp\Exception\ConnectException` 异常发生在网络错误时， 该异常继承自 `GuzzleHttp\Exception\RequestException` 。

- 如果 `http_errors` 请求参数设置成true，在400级别的错误的时候将会抛出 `GuzzleHttp\Exception\ClientException` 异常， 该异常继承自 `GuzzleHttp\Exception\BadResponseException` `GuzzleHttp\Exception\BadResponseException` 继承自 `GuzzleHttp\Exception\RequestException` 。

  

```php
use GuzzleHttp\Exception\ClientException;
```

- 如果 `http_errors` 请求参数设置成true，在500级别的错误的时候将会抛出 `GuzzleHttp\Exception\ServerException` 异常。 该异常继承自 `GuzzleHttp\Exception\BadResponseException` 。
- `GuzzleHttp\Exception\TooManyRedirectsException` 异常发生在重定向次数过多时， 该异常继承自 `GuzzleHttp\Exception\RequestException` 。

上述所有异常均继承自 `GuzzleHttp\Exception\TransferException` 。