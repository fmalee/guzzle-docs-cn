==========
快速入门
==========

该页面提供了Guzzle的快速入门以及列子，如果你还没有安装Guzzle请前往 :ref:`installation` 页面。

创建请求
================

你可以使用Guzzle的 ``GuzzleHttp\ClientInterface`` 对象来发送请求。

创建客户端
-----------------

.. code-block:: php

    use GuzzleHttp\Client;

    $client = new Client([
        // 基础URI使用相对的请求
        'base_uri' => 'http://httpbin.org',
        // 可以设置任意数量的默认请求选项
        'timeout'  => 2.0,
    ]);

客户端在Guzzle6中是不可变的，这意味着你无法在客户端被创建后更改其使用的默认值。

客户端构造函数接受一个关联的选项数组：

``base_uri``
    (string|UriInterface) 基础URI用来合并到相关URI，可以是一个字符串或者 ``UriInterface`` 实例。
    当提供了一个相对的URI，则将合并到基础URI，遵循的规则请参考 `RFC 3986, section 2 <http://tools.ietf.org/html/rfc3986#section-5.2>`_。

    .. code-block:: php

        // 使用一个基础URI来创建客户端
        $client = new GuzzleHttp\Client(['base_uri' => 'https://foo.com/api/']);
        // 发送一个请求到 https://foo.com/api/test
        $response = $client->request('GET', 'test');
        // 发送一个请求到 https://foo.com/root
        $response = $client->request('GET', '/root');

    不想阅读 RFC 3986？这里有一些关于使用他URI来解析 ``base_uri`` 的快速例子：

    =======================  ==================  ===============================
    base_uri                 URI                 结果
    =======================  ==================  ===============================
    ``http://foo.com``       ``/bar``            ``http://foo.com/bar``
    ``http://foo.com/foo``   ``/bar``            ``http://foo.com/bar``
    ``http://foo.com/foo``   ``bar``             ``http://foo.com/bar``
    ``http://foo.com/foo/``  ``bar``             ``http://foo.com/foo/bar``
    ``http://foo.com``       ``http://baz.com``  ``http://baz.com``
    ``http://foo.com/?bar``  ``bar``             ``http://foo.com/bar``
    =======================  ==================  ===============================

``handler``
    (callable) 传输HTTP请求的回调函数。该函数被调用的时候包含一个 ``Psr7\Http\Message\RequestInterface``
    以及传输选项数组，并且必须返回 ``GuzzleHttp\Promise\PromiseInterface``，成功的话使用
    ``Psr7\Http\Message\ResponseInterface`` 填充。``handler`` 是一个构造方法，不能在请求参数里被重写。

``...``
    (mixed) 构造方法中传入的其他所有参数都被用来当作每次请求的默认参数。

发送请求
----------------

客户端的魔术方法可以很容易的发送同步请求：

.. code-block:: php

    $response = $client->get('http://httpbin.org/get');
    $response = $client->delete('http://httpbin.org/delete');
    $response = $client->head('http://httpbin.org/get');
    $response = $client->options('http://httpbin.org/get');
    $response = $client->patch('http://httpbin.org/patch');
    $response = $client->post('http://httpbin.org/post');
    $response = $client->put('http://httpbin.org/put');

你可以创建一个请求，一切就绪后再将请求传送给客户端：

.. code-block:: php

    use GuzzleHttp\Psr7\Request;

    $request = new Request('PUT', 'http://httpbin.org/put');
    $response = $client->send($request, ['timeout' => 2]);

``Client`` 对象为传输请求提供了非常灵活的处理器方式，包括请求参数、每次请求使用的中间件以及传送多个相对请求的基础URI。

你可以在 :doc:`handlers-and-middleware` 页面找到更多关于中间件的内容。

异步请求
--------------

你可以使用 ``Client`` 提供的魔术方法来发送异步请求：

.. code-block:: php

    $promise = $client->getAsync('http://httpbin.org/get');
    $promise = $client->deleteAsync('http://httpbin.org/delete');
    $promise = $client->headAsync('http://httpbin.org/get');
    $promise = $client->optionsAsync('http://httpbin.org/get');
    $promise = $client->patchAsync('http://httpbin.org/patch');
    $promise = $client->postAsync('http://httpbin.org/post');
    $promise = $client->putAsync('http://httpbin.org/put');

你也可以使用一个客户端的 ``sendAsync()`` 和 ``requestAsync()`` 方法：

.. code-block:: php

    use GuzzleHttp\Psr7\Request;

    // 创建一个PSR-7请求对象以用以发送
    $headers = ['X-Foo' => 'Bar'];
    $body = 'Hello!';
    $request = new Request('HEAD', 'http://httpbin.org/head', $headers, $body);
    $promise = $client->sendAsync($request);

    // 或者，不需要传入请求实例：
    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');

这些方法返回了 ``Promise`` 对象，该对象实现了由
`Guzzle promises library <https://github.com/guzzle/promises>`_
提供的 `Promises/A+ spec <https://promisesaplus.com/>`_，这意味着你可以使用 ``then()`` 链来调用返回值，成功则使用  ``Psr\Http\Message\ResponseInterface`` 填充，否则抛出一个异常。

.. code-block:: php

    use Psr\Http\Message\ResponseInterface;
    use GuzzleHttp\Exception\RequestException;

    $promise = $client->requestAsync('GET', 'http://httpbin.org/get');
    $promise->then(
        function (ResponseInterface $res) {
            echo $res->getStatusCode() . "\n";
        },
        function (RequestException $e) {
            echo $e->getMessage() . "\n";
            echo $e->getRequest()->getMethod();
        }
    );

并发请求
-------------------

你可以使用Promise和异步请求来同时发送多个请求。

.. code-block:: php

    use GuzzleHttp\Client;
    use GuzzleHttp\Promise;

    $client = new Client(['base_uri' => 'http://httpbin.org/']);

    // 启动每个请求但不阻止(block)
    $promises = [
        'image' => $client->getAsync('/image'),
        'png'   => $client->getAsync('/image/png'),
        'jpeg'  => $client->getAsync('/image/jpeg'),
        'webp'  => $client->getAsync('/image/webp')
    ];

    // 等待请求完成; 如果有任何一个请求失败，则抛出 ConnectException
    $responses = Promise\unwrap($promises);

    // 等待请求完成，即使其中一些请求已经失败
    $responses = Promise\settle($promises)->wait();

    // 你可以使用 promise 的键来访问每个响应
    echo $responses['image']->getHeader('Content-Length')[0]
    echo $responses['png']->getHeader('Content-Length')[0]

当你想发送不确定数量的请求时，可以使用 ``GuzzleHttp\Pool`` 对象：

.. code-block:: php

    use GuzzleHttp\Client;
    use GuzzleHttp\Exception\RequestException;
    use GuzzleHttp\Pool;
    use GuzzleHttp\Psr7\Request;
    use GuzzleHttp\Psr7\Response;

    $client = new Client();

    $requests = function ($total) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield new Request('GET', $uri);
        }
    };

    $pool = new Pool($client, $requests(100), [
        'concurrency' => 5,
        'fulfilled' => function (Response $response, $index) {
            // 这是每次成功的响应传递的
        },
        'rejected' => function (RequestException $reason, $index) {
            // 这是每个失败的请求传递的
        },
    ]);

    // 启动传输并创建一个promise
    $promise = $pool->promise();

    // 强制完成请求池。
    $promise->wait();

或者使用一个闭包，一旦池调用闭包，它将返回一个 ``Promise`` 对象。

.. code-block:: php

    $client = new Client();

    $requests = function ($total) use ($client) {
        $uri = 'http://127.0.0.1:8126/guzzle-server/perf';
        for ($i = 0; $i < $total; $i++) {
            yield function() use ($client, $uri) {
                return $client->getAsync($uri);
            };
        }
    };

    $pool = new Pool($client, $requests(100));


使用响应
===============

前面的例子里，我们获取了一个 ``$response`` 变量，或者从Promise得到了一个响应。
该 ``Response`` 对象实现了 ``Psr\Http\Message\ResponseInterface`` PSR-7接口，其中包含了很多有用的信息。

你可以获取这个响应的状态码和和原因短语(reason phrase)：

.. code-block:: php

    $code = $response->getStatusCode(); // 200
    $reason = $response->getReasonPhrase(); // OK

你可以从响应中获取标头：

.. code-block:: php

    // 检查标头是否存在
    if ($response->hasHeader('Content-Length')) {
        echo "It exists";
    }

    // 从响应中获取标头
    echo $response->getHeader('Content-Length')[0];

    //获取所有响应标头
    foreach ($response->getHeaders() as $name => $values) {
        echo $name . ': ' . implode(', ', $values) . "\r\n";
    }


使用 ``getBody`` 方法可以获取响应的正文(body)，主体可以当成一个字符串或流对象使用。

.. code-block:: php

    $body = $response->getBody();
    // 将正文隐式投射到一个字符串并echo它
    echo $body;
    // 将正文显式地转换为字符串
    $stringBody = (string) $body;
    // 从正文中读取10个字节
    $tenBytes = $body->read(10);
    // 以字符串形式读取正文的剩余内容
    $remainingBytes = $body->getContents();

查询字符串参数
=======================

你可以有多种方式来提供请求的查询字符串。

你可以在请求的URI中设置查询字符串：

.. code-block:: php

    $response = $client->request('GET', 'http://httpbin.org?foo=bar');

你可以使用 ``query`` 请求参数来指定查询字符串参数：

.. code-block:: php

    $client->request('GET', 'http://httpbin.org', [
        'query' => ['foo' => 'bar']
    ]);

提供的数组参数将会使用PHP的 ``http_build_query`` 函数来格式化该查询字符串：

最后，你可以提供一个字符串作为 ``query`` 请求选项：

.. code-block:: php

    $client->request('GET', 'http://httpbin.org', ['query' => 'foo=bar']);


上传数据
==============

Guzzle为上传数据提供了一些方法。

你可以发送一个包含数据流的请求，将一个字符串、``fopen`` 返回的资源、或者一个
``Psr\Http\Message\StreamInterface`` 的实例设置为 ``body`` 请求选项。

.. code-block:: php

    // 提供字符串作为正文
    $r = $client->request('POST', 'http://httpbin.org/post', [
        'body' => 'raw data'
    ]);

    // 提供一个fopen资源
    $body = fopen('/path/to/file', 'r');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);

    // 使用 stream_for() 函数创建一个PSR-7流。
    $body = \GuzzleHttp\Psr7\stream_for('hello!');
    $r = $client->request('POST', 'http://httpbin.org/post', ['body' => $body]);

上传JSON数据以及设置合适的标头的简单方式就是使用 ``json`` 请求选项：

.. code-block:: php

    $r = $client->request('PUT', 'http://httpbin.org/put', [
        'json' => ['foo' => 'bar']
    ]);

POST/表单请求
------------------

除了使用 ``body`` 请求选项来指定请求的原始数据外，Guzzle还为发送POST数据提供了其他有用的方法。

发送表单字段
~~~~~~~~~~~~~~~~~~~

发送 ``application/x-www-form-urlencoded`` POST请求需要你传入一个指定了POST的字段的
``form_params`` 请求选项数组。

.. code-block:: php

    $response = $client->request('POST', 'http://httpbin.org/post', [
        'form_params' => [
            'field_name' => 'abc',
            'other_field' => '123',
            'nested_field' => [
                'nested' => 'hello'
            ]
        ]
    ]);

发送表单文件
~~~~~~~~~~~~~~~~~~

你可以通过使用 ``multipart`` 请求选项来发送表单文件(表单 ``enctype`` 属性需要设置为
``multipart/form-data``)，该选项接收一个包含多个关联数组的数组，每个关联数组包含以下键：

- ``name``: (必需，字符串) 映射到表单字段名称的键。
- ``contents``: (必需，混合) 提供一个字符串，则以字符串形式发送文件内容。
  提供一个 ``fopen`` 资源，则以从PHP流中获取的流来传输内容。
  或提供一个 ``Psr\Http\Message\StreamInterface``，则以从PSR-7流中获取的内容来传输内容。

.. code-block:: php

    $response = $client->request('POST', 'http://httpbin.org/post', [
        'multipart' => [
            [
                'name'     => 'field_name',
                'contents' => 'abc'
            ],
            [
                'name'     => 'file_name',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'other_file',
                'contents' => 'hello',
                'filename' => 'filename.txt',
                'headers'  => [
                    'X-Foo' => 'this is an extra header to include'
                ]
            ]
        ]
    ]);

Cookies
=======

Guzzle可以使用 ``cookies`` 请求选项为你维护一个Cookie会话。
当发送一个请求时，``cookies`` 选项必须设置成一个 ``GuzzleHttp\Cookie\CookieJarInterface`` 实例。

.. code-block:: php

    // 使用特定的cookie jar
    $jar = new \GuzzleHttp\Cookie\CookieJar;
    $r = $client->request('GET', 'http://httpbin.org/cookies', [
        'cookies' => $jar
    ]);

如果你想为所有请求使用一个共享的Cookie Jar，可以在客户端的构造函数中将 ``cookies`` 设置为 ``true``。

.. code-block:: php

    // 使用客户端共享的 Cookie Jar
    $client = new \GuzzleHttp\Client(['cookies' => true]);
    $r = $client->request('GET', 'http://httpbin.org/cookies');

存在以下不同的 ``GuzzleHttp\Cookie\CookieJarInterface`` 实现：

- ``GuzzleHttp\Cookie\CookieJar`` 类将Cookie储存为数组。
- ``GuzzleHttp\Cookie\FileCookieJar`` 类使用一个JSON格式的文件来持久非会话的Cookie。
- ``GuzzleHttp\Cookie\SessionCookieJar`` 类以客户端的会话来持久Cookie。

你可以使用命名构造器 ``fromArray(array $cookies, $domain)`` 来手动将Cookie设置为Cookie Jar。

.. code-block:: php

    $jar = \GuzzleHttp\Cookie\CookieJar::fromArray(
        [
            'some_cookie' => 'foo',
            'other_cookie' => 'barbaz1234'
        ],
        'example.org'
    );

你可以使用 ``getCookieByName($name)`` 方法来通过名称获取一个Cookie。该方法返回一个
``GuzzleHttp\Cookie\SetCookie`` 实例。

.. code-block:: php

    $cookie = $jar->getCookieByName('some_cookie');

    $cookie->getValue(); // 'foo'
    $cookie->getDomain(); // 'example.org'
    $cookie->getExpires(); // 作为Unix时间戳的到期日期

得益于 ``toArray()`` 方法，Cookie也可以被提取到数组中。
``GuzzleHttp\Cookie\CookieJarInterface`` 接口继承了
``Traversable``，因此它可以在 ``foreach`` 循环中迭代。

重定向
=========

如果你没有告诉Guzzle不要重定向，Guzzle会自动的进行重定向。
你可以使用 ``allow_redirects`` 请求选项来自定义重定向行为。

- 设置成 ``true`` 时将启用最大数量为 ``5`` 的重定向，这是默认设置。
- 设置成 ``false`` 可以禁用重定向。
- 传入一个包含 ``max`` 键名的关联数组来声明最大重定向次数，并且提供可选的 ``strict``
  键名来声明是否使用严格的RFC标准重定向 (即使用POST请求来重定向POST请求 vs
  大部分浏览器使用GET请求来重定向POST请求)。

.. code-block:: php

    $response = $client->request('GET', 'http://github.com');
    echo $response->getStatusCode();
    // 200

下面的列子表示重定向被禁止：

.. code-block:: php

    $response = $client->request('GET', 'http://github.com', [
        'allow_redirects' => false
    ]);
    echo $response->getStatusCode();
    // 301

异常
==========

**继承树**

以下视图树描述了Guzzle的异常是如何相互依赖的。

.. code-block:: none

    . \RuntimeException
    ├── SeekException (实现了 GuzzleException)
    └── TransferException (实现了 GuzzleException)
        └── RequestException
            ├── BadResponseException
            │   ├── ServerException
            │   └── ClientException
            ├── ConnectException
            └── TooManyRedirectsException

如果请求传输过程中出现错误，则Guzzle将会抛出异常。

- 在发生网络错误(连接超时、DNS错误等)时，将会抛出 ``GuzzleHttp\Exception\RequestException``
  异常，该异常继承自 ``GuzzleHttp\Exception\TransferException``。
  捕获这个异常，将可以捕获在传输请求过程中抛出的任何异常。

  .. code-block:: php

      use GuzzleHttp\Psr7;
      use GuzzleHttp\Exception\RequestException;

      try {
          $client->request('GET', 'https://github.com/_abc_123_404');
      } catch (RequestException $e) {
          echo Psr7\str($e->getRequest());
          if ($e->hasResponse()) {
              echo Psr7\str($e->getResponse());
          }
      }

- 发生网络错误时会抛出一个 ``GuzzleHttp\Exception\ConnectException`` 异常，该异常继承自
  ``GuzzleHttp\Exception\RequestException``。

- 如果将 ``http_errors`` 请求选项设置成 ``true``，则将在发生 ``400``
  级别的错误时抛出 ``GuzzleHttp\Exception\ClientException`` 异常，该异常继承自 ``GuzzleHttp\Exception\BadResponseException``，而
  ``GuzzleHttp\Exception\BadResponseException`` 则继承自 ``GuzzleHttp\Exception\RequestException``。

  .. code-block:: php

      use GuzzleHttp\Psr7;
      use GuzzleHttp\Exception\ClientException;

      try {
          $client->request('GET', 'https://github.com/_abc_123_404');
      } catch (ClientException $e) {
          echo Psr7\str($e->getRequest());
          echo Psr7\str($e->getResponse());
      }

- 如果将 ``http_errors`` 请求选项设置成 ``true``，则将在发生 ``500``
  级别的错误时抛出 ``GuzzleHttp\Exception\ServerException`` 异常，该异常继承自 ``GuzzleHttp\Exception\BadResponseException``。

- ``GuzzleHttp\Exception\TooManyRedirectsException`` 异常发生在重定向次数过多时，该异常继承自
  ``GuzzleHttp\Exception\RequestException``。

上述所有异常均继承自 ``GuzzleHttp\Exception\TransferException``。

环境变量
=====================

Guzzle提供了一些可以自定义行为的环境变量：

``GUZZLE_CURL_SELECT_TIMEOUT``
    当在curl处理器使用 ``curl_multi_select()`` 时控制了 ``curl_multi_*`` 处理器需要使用到的持续时间。
    有些系统的 ``curl_multi_select()`` PHP实现存在问题，调用该函数时总是等待超时的最大值。
``HTTP_PROXY``
    定义了使用 ``http`` 协议发送请求时使用的代理。

    注意：因为 ``HTTP_PROXY`` 变量可能在某些（CGI）环境中包含任意用户输入，所以该变量仅适用于CLI SAPI。
    有关更多信息，请参阅 https://httpoxy.org。
``HTTPS_PROXY``
    定义了使用 ``https`` 协议发送请求时使用的代理。
``NO_PROXY``
    定义不应使用代理的URL。请参阅 :ref:`proxy-option` 以了解其用法。

相关ini设置
---------------------

配置客户端时，Guzzle可以利用PHP的ini配置。

``openssl.cafile``
    当发送 ``https`` 协议的请求时需要用到指定磁盘上PEM格式的CA文件，参考： https://wiki.php.net/rfc/tls-peer-verification#phpini_defaults
