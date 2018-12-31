.. title:: Guzzle, 基于PHP的HTTP客户端

====================
Guzzle中文文档
====================

Guzzle是一个PHP的HTTP客户端，用来轻而易举地发送请求，并集成到我们的WEB服务上。

- 接口简单：构建查询语句、POST请求、分流上传下载大文件、使用HTTP cookies、上传JSON数据等等。
- 发送同步或异步的请求均使用相同的接口。
- 使用PSR-7接口来请求、响应、分流，允许你使用其他兼容的PSR-7类库与Guzzle共同开发。
- 抽象了底层的HTTP传输，允许你改变环境以及其他的代码，如：对cURL与PHP的流或socket并非重度依赖，非阻塞事件循环。
- 中间件系统允许你创建构成客户端行为。

.. code-block:: php

    $client = new GuzzleHttp\Client();
    $res = $client->request('GET', 'https://api.github.com/user', [
        'auth' => ['user', 'pass']
    ]);
    echo $res->getStatusCode();
    // "200"
    echo $res->getHeader('content-type')[0];
    // 'application/json; charset=utf8'
    echo $res->getBody();
    // {"type":"User"...'

    // 发送一个异步请求
    $request = new \GuzzleHttp\Psr7\Request('GET', 'http://httpbin.org');
    $promise = $client->sendAsync($request)->then(function ($response) {
        echo 'I completed! ' . $response->getBody();
    });
    $promise->wait();


用户指南
==========

.. toctree::
    :maxdepth: 3

    overview
    quickstart
    request-options
    psr7
    handlers-and-middleware
    testing
    faq
