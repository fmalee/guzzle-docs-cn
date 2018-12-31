========
常见问题
========

Guzzle必须要cURL吗？
=========================

不需要。Guzzle可以使用任何HTTP处理器来发送请求。
这意味着Guzzle可以与cURL、PHP的流封装器、套接字和非阻塞库（如 `React <http://reactphp.org/>`_）一起使用。
你只需要配置一个HTTP处理器以使用不同的发送请求的方法。

.. note::

    Guzzle曾经只使用cURL发送HTTP请求。cURL是一个令人惊叹的HTTP客户端（可以说是最好的），Guzzle会在可用时继续默认使用它。
    这种情况很少见，但有些开发人员没有在他们的系统上安装cURL或遇到特定于版本的问题。
    通过允许可交换的HTTP处理器，Guzzle现在可以更加可自定义，并且能够适应更多开发人员的需求。

Guzzle可以发送异步请求吗？
======================================

可以。你可以使用一个客户端的 ``requestAsync``、``sendAsync``、``getAsync``、
``headAsync``、``putAsync``、``postAsync``、``deleteAsync`` 以及
``patchAsync`` 方法来发送异步请求。该客户端将返回一个
``GuzzleHttp\Promise\PromiseInterface`` 对象。
然后你可以链(chain)到的Promise的 ``then`` 函数。

.. code-block:: php

    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $promise->then(function ($response) {
        echo 'Got a response! ' . $response->getStatusCode();
    });

你可以使用返回的Promise的 ``wait()`` 方法来强制完成一个异步响应。

.. code-block:: php

    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $response = $promise->wait();

如何添加自定义cURL选项？
==================================

cURL提供大量的 `可自定义选项 <http://us1.php.net/curl_setopt>`_。
虽然Guzzle在不同处理器中规范化(normalize)了许多这些选项，但有时你需要设置自定义cURL选项。
这可以通过在请求的 **curl** 键中传递cURL设置的关联数组来实现。

例如，假设你需要自定义与客户端一起使用的传出网络接口（outgoing network interface）。

.. code-block:: php

    $client->request('GET', '/', [
        'curl' => [
            CURLOPT_INTERFACE => 'xxx.xxx.xxx.xxx'
        ]
    ]);


如何添加自定义流上下文选项？
============================================

你可以使用请求选项的 **stream_context** 键来传递自定义的
`流上下文选项 <http://www.php.net/manual/en/context.php>`_。
所述 ``stream_context`` 数组是一个关联数组，其中每个键是一个PHP传输，每个值是传输数组项。

例如，假设你需要自定义与客户端一起使用的传出网络接口，并允许自签名证书。

.. code-block:: php

    $client->request('GET', '/', [
        'stream' => true,
        'stream_context' => [
            'ssl' => [
                'allow_self_signed' => true
            ],
            'socket' => [
                'bindto' => 'xxx.xxx.xxx.xxx'
            ]
        ]
    ]);


为什么我会收到SSL验证错误？
===========================================

你需要在磁盘上指定Guzzle用于验证对等证书的CA包的路径。请参阅 :ref:`verify-option`。


什么是最大函数嵌套错误？
============================================

    Maximum function nesting level of '100' reached, aborting

如果安装了 ``XDebug`` 扩展并且在回调中执行了大量请求，则可能会遇到此错误。
此错误消息来自XDebug扩展。PHP本身没有函数嵌套限制。可以在php.ini中更改此设置以增加限制::

    xdebug.max_nesting_level = 1000

为什么我收到一个417响应错误？
======================================

这可能由于多种原因而发生，但如果你使用 ``Expect: 100-Continue``
标头发送PUT、POST或PATCH请求，则不支持此标头的服务器将返回 ``417`` 响应。
你可以通过将 ``expect`` 请求选项设置为 ``false`` 来规避此响应：

.. code-block:: php

    $client = new GuzzleHttp\Client();

    // 在单个请求上禁用expect标头
    $response = $client->request('PUT', '/', ['expect' => false]);

    // 在所有客户端请求上禁用expect标头
    $client = new GuzzleHttp\Client(['expect' => false]);

如何跟踪重定向的请求？
====================================

你可以通过 ``track_redirects`` 选项来启用对重定向的URI和状态代码的跟踪。
每个重定向的URI和状态代码将分别存储在 ``X-Guzzle-Redirect-History`` 和
``X-Guzzle-Redirect-Status-History`` 标头中。

初始请求的URI和最终状态代码将从结果中排除。基于这个原因，你应该能够轻松跟踪请求的完整重定向路径。

例如，假设你需要跟踪重定向并在单个报告中同时提供两个结果：

.. code-block:: php

    // 首先使用重定向跟踪来配置Guzzle并发出请求
    $client = new Client([
        RequestOptions::ALLOW_REDIRECTS => [
            'max'             => 10,        // 允许最多10个重定向
            'strict'          => true,      // 使用“严格”的RFC兼容重定向。
            'referer'         => true,      // 添加Referer标头
            'track_redirects' => true,
        ],
    ]);
    $initialRequest = '/redirect/3'; // 存储请求URI以供后续使用
    $response = $client->request('GET', $initialRequest); // 发出你的请求

    // 检索两个重定向历史标头
    $redirectUriHistory = $response->getHeader('X-Guzzle-Redirect-History')[0]; // 检索重定向URI历史记录
    $redirectCodeHistory = $response->getHeader('X-Guzzle-Redirect-Status-History')[0]; // 检索重定向HTTP状态历史记录

    // 将请求的初始URI添加到URI历史记录(开头)
    array_unshift($redirectUriHistory, $initialRequest);

    // 将最终的HTTP状态代码添加到HTTP响应历史记录的末尾
    array_push($redirectCodeHistory, $response->getStatusCode());

    // （可选）将每个数组的单元组合成一个结果集
    $fullRedirectReport = [];
    foreach ($redirectUriHistory as $key => $value) {
        $fullRedirectReport[$key] = ['location' => $value, 'code' => $redirectCodeHistory[$key]];
    }
    echo json_encode($fullRedirectReport);
