===============
请求选项
===============

你可以通过设置 ``Client`` 的 **请求选项**
来自定义请求，请求参数控制请求的各个方面，包括头信息、查询字符串参数、超时、请求主体等。

下述所有的列子都使用下面的 ``Client``：

.. code-block:: php

    $client = new GuzzleHttp\Client(['base_uri' => 'http://httpbin.org']);

.. _allow_redirects-option:

allow_redirects
---------------

:摘要: 描述一个请求的重定向行为
:类型:
    - 布尔值
    - 数组
:默认值:

    ::

        [
            'max'             => 5,
            'strict'          => false,
            'referer'         => false,
            'protocols'       => ['http', 'https'],
            'track_redirects' => false
        ]

:常量: ``GuzzleHttp\RequestOptions::ALLOW_REDIRECTS``

设置成 ``false`` 禁用可以重定向。

.. code-block:: php

    $res = $client->request('GET', '/redirect/3', ['allow_redirects' => false]);
    echo $res->getStatusCode();
    // 302

设置成 ``true`` (默认设置) 来启用默认最大次数为 ``5`` 的重定向。

.. code-block:: php

    $res = $client->request('GET', '/redirect/3');
    echo $res->getStatusCode();
    // 200

你也可以传送一个包含了以下键值对的关联数组：

- ``max``: (``int``, 默认为 ``5``) 允许重定向次数的最大值。
- ``strict``: (``bool``, 默认为 ``false``) 设置成 ``true`` 来使用严格模式重定向。
  严格RFC模式重定向表示使用POST请求重定向POST请求 vs 大部分浏览器使用GET请求重定向POST请求。
- ``referer``: (``bool``, 默认为 ``false``) 设置成 ``false`` 来在重定向时禁止添加Refer标头。
- ``protocols``: (``array``, 默认为 ``['http', 'https']``) 指定允许重定向的协议。
- ``on_redirect``: (``callable``) 发生重定向时调用的PHP回调，包含了原始的请求以及接收到重定向的响应,
  ``on_redirect`` 的任何返回将会被忽略。
- ``track_redirects``: (``bool``) 当设置为 ``true`` 时，每个重定向的URI和状态代码将分别在
  ``X-Guzzle-Redirect-History`` 以及 ``X-Guzzle-Redirect-Status-History``
  标头中被跟踪。所有URI和状态代码将按重定向遇到的顺序存储。

  注意：当跟踪重定向时，``X-Guzzle-Redirect-History``
  标头将排除初始请求的URI，而 ``X-Guzzle-Redirect-Status-History`` 标头将排除最终状态代码。

.. code-block:: php

    use Psr\Http\Message\RequestInterface;
    use Psr\Http\Message\ResponseInterface;
    use Psr\Http\Message\UriInterface;

    $onRedirect = function(
        RequestInterface $request,
        ResponseInterface $response,
        UriInterface $uri
    ) {
        echo 'Redirecting! ' . $request->getUri() . ' to ' . $uri . "\n";
    };

    $res = $client->request('GET', '/redirect/3', [
        'allow_redirects' => [
            'max'             => 10,        // 允许最多10次重定向
            'strict'          => true,      // 使用“严格”符合RFC的重定向
            'referer'         => true,      // 添加Referer标头
            'protocols'       => ['https'], // 仅允许https网址
            'on_redirect'     => $onRedirect,
            'track_redirects' => true
        ]
    ]);

    echo $res->getStatusCode();
    // 200

    echo $res->getHeaderLine('X-Guzzle-Redirect-History');
    // http://first-redirect, http://second-redirect, etc...

    echo $res->getHeaderLine('X-Guzzle-Redirect-Status-History');
    // 301, 302等等...

.. warning::

    仅你的处理器具有 ``GuzzleHttp\Middleware::redirect`` 中间件时此选项才起作用。
    默认情况下，在创建客户端时没有处理器的情况下会添加此中间件，并且在使用
    ``GuzzleHttp\HandlerStack::create`` 来创建处理器时也会默认添加此中间件。

auth
----

:摘要: 传入一个HTTP认证参数的数组来使用请求，该数组索引 ``[0]`` 为用户名、索引 ``[1]``
    为密码，索引 ``[2]`` 为可选的内置认证类型。传入 ``null`` 可以禁用当前请求的认证。
:类型:
    - 数组
    - 字符串
    - null
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::AUTH``

内置认证类型如下:

``basic``
    在 ``Authorization`` 标头使用
    `HTTP基础认证 <http://www.ietf.org/rfc/rfc2069.txt>`_ (如果没有指定的话为默认设置)。

.. code-block:: php

    $client->request('GET', '/get', ['auth' => ['username', 'password']]);

``digest``
    使用 `摘要式认证 <http://www.ietf.org/rfc/rfc2069.txt>`_ (必须被HTTP处理器支持)。

.. code-block:: php

    $client->request('GET', '/get', [
        'auth' => ['username', 'password', 'digest']
    ]);

.. note::

    目前仅在使用cURL处理器时支持此类型，但计划创建可与任何HTTP处理器一起使用的替代。

``ntlm``
    使用
    `Microsoft NTLM认证 <https://msdn.microsoft.com/en-us/library/windows/desktop/aa378749(v=vs.85).aspx>`_
    (必须被HTTP处理器支持)。

.. code-block:: php

    $client->request('GET', '/get', [
        'auth' => ['username', 'password', 'ntlm']
    ]);

.. note::

    目前仅在使用cURL处理器时支持此类型。

body
----

:摘要: ``body`` 选项用来控制一个请求(比如：PUT, POST, PATCH)的正文部分。
:类型:
    - 数组
    - ``fopen()`` 资源
    - ``Psr\Http\Message\StreamInterface``
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::BODY``

可以设置成下述类型：

- 字符串

  .. code-block:: php

      // 你可以发送使用字符串作为消息正文的请求。
      $client->request('PUT', '/put', ['body' => 'foo']);

- 从 ``fopen()`` 中返回的资源

  .. code-block:: php

      // 你可以发送使用流资源作为正文的请求。
      $resource = fopen('http://httpbin.org', 'r');
      $client->request('PUT', '/put', ['body' => $resource]);

- ``Psr\Http\Message\StreamInterface``

  .. code-block:: php

      // 你可以发送使用Guzzle流对象作为正文的请求
      $stream = GuzzleHttp\Psr7\stream_for('contents...');
      $client->request('POST', '/post', ['body' => $stream]);

.. note::

    此选项不能和 ``form_params``、``multipart`` 以及 ``json`` 一起使用

.. _cert-option:

cert
----

:摘要: 设置一个字符串来指定PEM格式认证文件的路径。
    如果需要密码，则需要设置成一个数组，其中PEM文件在第一个元素，密码在第二个元素。
:类型:
    - 字符串
    - 数组
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::CERT``

.. code-block:: php

    $client->request('GET', '/', ['cert' => ['/path/server.pem', 'password']]);

.. _cookies-option:

cookies
-------

:摘要: 声明是否在请求中使用cookie，可以是要使用的cookie jar，或者要发送的cookie。
:类型: ``GuzzleHttp\Cookie\CookieJarInterface``
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::COOKIES``

你必须指定 ``cookie`` 选项为 ``GuzzleHttp\Cookie\CookieJarInterface`` 或 ``false``。

.. code-block:: php

    $jar = new \GuzzleHttp\Cookie\CookieJar();
    $client->request('GET', '/get', ['cookies' => $jar]);

.. warning::

    仅你的处理器具有 ``GuzzleHttp\Middleware::cookies`` 中间件时此选项才起作用。
    默认情况下，在创建客户端时没有处理器的情况下会添加此中间件，并且在使用
    ``GuzzleHttp\default_handler`` 来创建处理器时也会默认添加此中间件。

.. tip::

    创建一个客户端时，可以将默认 ``cookie`` 选项设置 ``true``，以便与关联的客户端共享cookie会话。

.. _connect_timeout-option:

connect_timeout
---------------

:摘要: 表示等待服务器响应超时的最大值，使用 ``0`` 将无限等待 (默认行为).
:类型: 浮点
:默认值: ``0``
:常量: ``GuzzleHttp\RequestOptions::CONNECT_TIMEOUT``

.. code-block:: php

    // 如果客户端无法在3.14秒内连接到服务器，则超时。
    $client->request('GET', '/delay/5', ['connect_timeout' => 3.14]);

.. note::

    用于发送请求的HTTP处理器必须支持此设置。目前只有内置的cURL处理器支持此选项。

.. _debug-option:

debug
-----

:摘要: 设置成 ``true`` 或设置成一个 ``fopen()`` 返回的流来启用对发送请求的处理器的调试输出。
    比如，当使用cURL传输请求，cURL的 ``CURLOPT_VERBOSE`` 的冗长将会发出，当使用PHP流，流处理的提示将会发生。
    如果设置为 ``true``，输出将写入到PHP标准输出文件，如果提供了PHP流，将会输出到流。
:类型:
        - 布尔值
        - ``fopen()`` 资源
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::DEBUG``

.. code-block:: php

    $client->request('GET', '/get', ['debug' => true]);

执行上面的例子将会输出类似下面的结果：

::

    * About to connect() to httpbin.org port 80 (#0)
    *   Trying 107.21.213.98... * Connected to httpbin.org (107.21.213.98) port 80 (#0)
    > GET /get HTTP/1.1
    Host: httpbin.org
    User-Agent: Guzzle/4.0 curl/7.21.4 PHP/5.5.7

    < HTTP/1.1 200 OK
    < Access-Control-Allow-Origin: *
    < Content-Type: application/json
    < Date: Sun, 16 Feb 2014 06:50:09 GMT
    < Server: gunicorn/0.17.4
    < Content-Length: 335
    < Connection: keep-alive
    <
    * Connection #0 to host httpbin.org left intact


.. _decode_content-option:

decode_content
--------------

:摘要: 指定是否自动解码  ``Content-Encoding`` 响应 (gzip, deflate等) 。
:类型:
    - 字符串
    - 布尔值
:默认值: ``true``
:常量: ``GuzzleHttp\RequestOptions::DECODE_CONTENT``

该选项可以用来控制 ``Content-Encoding`` 如何响应主体的。默认情况下，``decode_content``
设置为 ``true``，表示Guzzle将自动解码 ``gzip``、``deflate`` 等响应。

当设置成 ``false``，响应的主体将不会被解码，意味着字节将毫无变化的通过处理器。

.. code-block:: php

    // 请求gzip压缩的数据，但在下载时不解码
    $client->request('GET', '/foo.js', [
        'headers'        => ['Accept-Encoding' => 'gzip'],
        'decode_content' => false
    ]);

当设置成字符串时，响应的字节将被解码，提供 ``decode_content``
选项的字符串将被传递为请求的 ``Accept-Encoding`` 标头。

.. code-block:: php

    // 将“gzip”作为Accept-Encoding标头传递。
    $client->request('GET', '/foo.js', ['decode_content' => 'gzip']);

.. _delay-option:

delay
-----

:摘要: 发送请求之前延迟的毫秒数值
:类型:
    - 整数
    - 浮点
:默认值: ``null``
:常量: ``GuzzleHttp\RequestOptions::DELAY``

.. _expect-option:

expect
------

:摘要: 控制 ``Expect: 100-Continue`` 标头的行为。
:类型:
    - 布尔值
    - 整数
:默认值: ``1048576``
:常量: ``GuzzleHttp\RequestOptions::EXPECT``

设置成 ``true`` 来为所有发送主体的请求启用 ``Expect: 100-Continue`` 标头。
设置成 ``false`` 来为所有的请求禁用 ``Expect: 100-Continue`` 标头。
设置成一个数值，有效载荷的大小必须大于预计发送的标头。
设置成数值将会为所有不确定有效载荷大小或主体不能确定指针位置的请求发送 ``Expect`` 标头。

默认情况下，当请求的主体大于 ``1MB`` 以及请求使用 ``HTTP/1.1`` 时，Guzzle将会添加
``Expect: 100-Continue`` 标头。

.. note::

    此选项仅在使用 ``HTTP/1.1`` 时生效。``HTTP/1.0`` 和 ``HTTP/2.0``
    协议不支持 ``Expect: 100-Continue`` 标头。支持处理 ``Expect: 100-Continue``
    的工作流必须由客户端使用的Guzzle HTTP处理器实现。

force_ip_resolve
----------------

:摘要: 如果希望HTTP处理器仅使用ipv4协议，请设置为 ``v4``，如果是ipv6协议，则设置为 ``v6``。
:类型: 字符串
:默认值: ``null``
:常量: ``GuzzleHttp\RequestOptions::FORCE_IP_RESOLVE``

.. code-block:: php

    // 强制为ipv4协议
    $client->request('GET', '/foo', ['force_ip_resolve' => 'v4']);

    // 强制为ipv6协议
    $client->request('GET', '/foo', ['force_ip_resolve' => 'v6']);

.. note::

    用于发送请求的HTTP处理器必须支持此设置。目前只有内置的cURL和流处理器支持 ``force_ip_resolve``。

form_params
-----------

:摘要: 用来发送一个 ``application/x-www-form-urlencoded`` POST请求.
:类型: 数组
:常量: ``GuzzleHttp\RequestOptions::FORM_PARAMS``

关联数组由表单字段键值对构成，每个字段值可以是一个字符串或一个包含字符串元素的数组。
当没有预设 "Content-Type" 标头的时候，会将其设置为 ``application/x-www-form-urlencoded``。

.. code-block:: php

    $client->request('POST', '/post', [
        'form_params' => [
            'foo' => 'bar',
            'baz' => ['hi', 'there!']
        ]
    ]);

.. note::

    ``form_params`` 不能与 ``multipart`` 选项一起使用。你只能使用其中一个。
    对 ``application/x-www-form-urlencoded`` 请求使用
    ``form_params``，对 ``multipart/form-data`` 请求使用 ``multipart``。

    此选项不能与 ``body``、``multipart``、``json`` 一起使用。

headers
-------

:摘要: 要添加到请求的标头的关联数组，每个键名是一个标头的名称，每个键值是一个字符串或代表标头字段值的数组。
:类型: 数组
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::HEADERS``

.. code-block:: php

    // 在请求上设置各种标头
    $client->request('GET', '/get', [
        'headers' => [
            'User-Agent' => 'testing/1.0',
            'Accept'     => 'application/json',
            'X-Foo'      => ['Bar', 'Baz']
        ]
    ]);

创建一个客户端的时候，标头可以作为默认选项添加。
当标头被作为默认选项使用时，它们只能在没有包含指定标头的请求中生效，这包括了传递给客户端的
``send()`` 和 ``sendAsync()`` 方法的请求，以及由客户端创建的请求(比如 ``request()`` 和 ``requestAsync()``)。

.. code-block:: php

    $client = new GuzzleHttp\Client(['headers' => ['X-Foo' => 'Bar']]);

    // 将使用X-Foo标头来发送请求。
    $client->request('GET', '/get');

    // 将X-Foo标头设置为“test”，这会阻止已应用的默认标头。
    $client->request('GET', '/get', ['headers' => ['X-Foo' => 'test']]);

    // 将禁用添加的默认标头
    $client->request('GET', '/get', ['headers' => null]);

    // 不会重写 X-Foo 标头，因为它只是包含在消息中。
    use GuzzleHttp\Psr7\Request;
    $request = new Request('GET', 'http://foo.com', ['X-Foo' => 'test']);
    $client->send($request);

    // 将使用send方法中提供的请求选项来重写该 X-Foo 标头。
    use GuzzleHttp\Psr7\Request;
    $request = new Request('GET', 'http://foo.com', ['X-Foo' => 'test']);
    $client->send($request, ['headers' => ['X-Foo' => 'overwrite']]);

.. _http-errors-option:

http_errors
-----------

:摘要: 设置成 ``false`` 来禁用在一个HTTP协议出错(如 ``4xx`` 和 ``5xx`` 响应)时抛出异常。
    默认情况下，HTTP协议出错时会抛出异常。
:类型: 布尔值
:默认值: ``true``
:常量: ``GuzzleHttp\RequestOptions::HTTP_ERRORS``

.. code-block:: php

    $client->request('GET', '/status/500');
    // 抛出一个 GuzzleHttp\Exception\ServerException

    $res = $client->request('GET', '/status/500', ['http_errors' => false]);
    echo $res->getStatusCode();
    // 500

.. warning::

    仅你的处理器具有 ``GuzzleHttp\Middleware::httpErrors`` 中间件时此选项才起作用。
    默认情况下，在创建客户端时没有处理器的情况下会添加此中间件，并且在使用
    ``GuzzleHttp\default_handler`` 来创建处理器时也会默认添加此中间件。

json
----

:摘要: ``json`` 选项用来轻松将JSON编码的数据作为请求的主体上传，如果消息中没有预设
    ``Content-Type`` 标头，则会将其设置为 ``application/json``。
:类型: 能够被PHP的 ``json_encode()`` 函数操作的任何PHP类型。
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::JSON``

.. code-block:: php

    $response = $client->request('PUT', '/put', ['json' => ['foo' => 'bar']]);

这里的例子使用了 ``tap`` 中间件用来查看发送了什么请求。

.. code-block:: php

    use GuzzleHttp\Middleware;

    // 获取客户端的处理器实例。
    $clientHandler = $client->getConfig('handler');
    // 创建一个回应(echoes)请求的部分的中间件。
    $tapMiddleware = Middleware::tap(function ($request) {
        echo $request->getHeaderLine('Content-Type');
        // application/json
        echo $request->getBody();
        // {"foo":"bar"}
    });

    $response = $client->request('PUT', '/put', [
        'json'    => ['foo' => 'bar'],
        'handler' => $tapMiddleware($clientHandler)
    ]);

.. note::

    此请求选项不支持自定义 ``Content-Type`` 标头或PHP的
    `json_encode() <http://www.php.net/manual/en/function.json-encode.php>`_
    函数中的任何选项。
    如果需要自定义这些设置，则必须使用 ``body``
    请求选项来自行将JSON编码数据传递到请求中，并且必须使用 ``headers``
    请求选项来指定正确的 ``Content-Type`` 标头。

    此选项不能与 ``body``、``form_params``、``multipart`` 一起使用。

multipart
---------

:摘要: 设置请求的主体为 ``multipart/form-data`` 表单。
:类型: 数组
:常量: ``GuzzleHttp\RequestOptions::MULTIPART``

``multipart`` 的值是一个关联数组，每个元素包含以下键值对：

- ``name``: (字符串, 必需) 表单字段名称
- ``contents``: (StreamInterface/资源/字符串, 必需) 表单元素中要使用的数据
- ``headers``: (数组) 可选，表单元素要使用的键值对数组
- ``filename``: (字符串) 可选，要发送的文件名称

.. code-block:: php

    $client->request('POST', '/post', [
        'multipart' => [
            [
                'name'     => 'foo',
                'contents' => 'data',
                'headers'  => ['X-Baz' => 'bar']
            ],
            [
                'name'     => 'baz',
                'contents' => fopen('/path/to/file', 'r')
            ],
            [
                'name'     => 'qux',
                'contents' => fopen('/path/to/file', 'r'),
                'filename' => 'custom_filename.txt'
            ],
        ]
    ]);

.. note::

    ``multipart`` 不能与 ``form_params`` 选项一起使用。你只能使用其中一个。
    对 ``application/x-www-form-urlencoded`` 请求使用
    ``form_params``，对 ``multipart/form-data`` 请求使用 ``multipart``。

    此选项不能与 ``body``、``form_params``、``json`` 一起使用。

.. _on-headers:

on_headers
----------

:摘要: 一个回调函数，当响应的HTTP标头被接收且主体部分还未开始下载的时候调用。
:类型: - 回调
:常量: ``GuzzleHttp\RequestOptions::ON_HEADERS``

该回调接受一个 ``Psr\Http\ResponseInterface`` 对象。
如果该回调抛出异常，则与该响应相关的Promise将会接收到一个封装着被抛出的异常的
``GuzzleHttp\Exception\RequestException``。

在数据写入下游(sink)之前，你应该需要知道接收到的标头与状态码。

.. code-block:: php

    // 拒绝大于1024字节的响应
    $client->request('GET', 'http://httpbin.org/stream/1024', [
        'on_headers' => function (ResponseInterface $response) {
            if ($response->getHeaderLine('Content-Length') > 1024) {
                throw new \Exception('The file is too big!');
            }
        }
    ]);

.. note::

    在编写HTTP处理器时，``on_headers`` 函数必须在将数据写入响应主体之前调用。

.. _on_stats:

on_stats
--------

:摘要: ``on_stats`` 允许你获取请求的传输数据统计以及处理器在底层传输的详情。
    ``on_stats`` 在处理器完成发送一个请求的时候被调用的一个回调。
    调用该回调时会传递关于请求、接收到响应，或遇到错误的传输数据统计，以及发送请求数据时间的总量。
:类型: - 回调
:常量: ``GuzzleHttp\RequestOptions::ON_STATS``

该回调接受一个 ``GuzzleHttp\TransferStats`` 对象。

.. code-block:: php

    use GuzzleHttp\TransferStats;

    $client = new GuzzleHttp\Client();

    $client->request('GET', 'http://httpbin.org/stream/1024', [
        'on_stats' => function (TransferStats $stats) {
            echo $stats->getEffectiveUri() . "\n";
            echo $stats->getTransferTime() . "\n";
            var_dump($stats->getHandlerStats());

            // 你必须在使用响应对象之前检查是否收到了一个响应。
            if ($stats->hasResponse()) {
                echo $stats->getResponse()->getStatusCode();
            } else {
                // 错误的数据特定于处理器。因此在使用此值之前，你需要知道处理器使用的错误数据类型。
                var_dump($stats->getHandlerErrorData());
            }
        }
    ]);

progress
--------

:摘要: 定义在创建传输进程(progress)时要调用的函数。
:类型: - 回调
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::PROGRESS``

该函数接受以下位置参数：

- 预期要下载的总字节数
- 到目前为止下载的字节数
- 预期上传的总字节数
- 到目前为止上传的字节数

.. code-block:: php

    // 发送GET请求到 /get?foo=bar
    $result = $client->request(
        'GET',
        '/',
        [
            'progress' => function(
                $downloadTotal,
                $downloadedBytes,
                $uploadTotal,
                $uploadedBytes
            ) {
                //做一些处理
            },
        ]
    );

.. _proxy-option:

proxy
-----

:摘要: 传入字符串来指定一个HTTP代理，或者是一个为不同协议指定不同代理的数组。
:类型:
    - 字符串
    - 数组
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::PROXY``

传入一个字符串为所有协议指定一个代理：

.. code-block:: php

    $client->request('GET', '/', ['proxy' => 'tcp://localhost:8125']);

传入关联数组来为指定的URI Scheme指定特定的HTTP代理(比如"http", "https")。
可以提供一个 ``no`` 键值对来定义一组不需要使用代理的主机名。

.. note::

    Guzzle会自动使用你的环境的 ``NO_PROXY`` 环境变量填充此值。
    但是，当提供一个 ``proxy`` 请求选项时，需要由你提供从 ``NO_PROXY`` 环境变量解析的
    ``no`` 值（例如，``explode(',', getenv('NO_PROXY'))``）。

.. code-block:: php

    $client->request('GET', '/', [
        'proxy' => [
            'http'  => 'tcp://localhost:8125', // 将此代理用于”http“，
            'https' => 'tcp://localhost:9124', // 将此代理用于”https“，
            'no' => ['.mit.edu', 'foo.com']    // 这些主机不使用代理
        ]
    ]);

.. note::

    你可以提供包含一个模式(scheme)、用户名和密码的代理URL。例如，"http://username:password@192.168.16.1:10"。

query
-----

:摘要: 要添加到请求的查询字符串的关联数组或查询字符串。
:类型:
    - 数组
    - 字符串
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::QUERY``

.. code-block:: php

    // 发送GET请求到 /get?foo=bar
    $client->request('GET', '/get', ['query' => ['foo' => 'bar']]);

在 ``query`` 选项中指定的查询字符串，将会重写请求的URI中的所有查询字符串值。

.. code-block:: php

    // 发送GET请求到 /get?foo=bar
    $client->request('GET', '/get?abc=123', ['query' => ['foo' => 'bar']]);

read_timeout
------------

:摘要: 描述读取流式正文时使用的超时
:类型: 浮点
:默认值: 默认为PHP ini设置的 ``default_socket_timeout`` 值
:常量: ``GuzzleHttp\RequestOptions::READ_TIMEOUT``

超时适用于一个流式正文(streamed body)上的单个读取操作（在启用 ``stream`` 选项时）。

.. code-block:: php

    $response = $client->request('GET', '/stream', [
        'stream' => true,
        'read_timeout' => 10,
    ]);

    $body = $response->getBody();

    // 超时时返回false
    $data = $body->read(1024);

    // 超时时返回false
    $line = fgets($body->detach());

.. _sink-option:

sink
----

:摘要: 指定响应的主体部分将要保存的位置。
:类型:
    - 字符串 (磁盘上的文件的路径)
    - ``fopen()`` 资源
    - ``Psr\Http\Message\StreamInterface``

:默认值: PHP temp stream
:常量: ``GuzzleHttp\RequestOptions::SINK``

传入字符串来指定将要保存响应主体内容的文件的路径：

.. code-block:: php

    $client->request('GET', '/stream/20', ['sink' => '/path/to/file']);

传入一个从 ``fopen()`` 返回的资源以将响应写入PHP流：

.. code-block:: php

    $resource = fopen('/path/to/file', 'w');
    $client->request('GET', '/stream/20', ['sink' => $resource]);

传入一个 ``Psr\Http\Message\StreamInterface`` 对象以将响应写入到打开的PSR-7流：

.. code-block:: php

    $resource = fopen('/path/to/file', 'w');
    $stream = GuzzleHttp\Psr7\stream_for($resource);
    $client->request('GET', '/stream/20', ['save_to' => $stream]);

.. note::

    ``save_to`` 请求选项已被弃用，取而代之的是 ``sink`` 请求选项。提供的
    ``save_to`` 选项现在是 ``sink`` 的别名。


.. _ssl_key-option:

ssl_key
-------

:摘要: 指定一个包含私有SSL密钥的PEM格式的文件的路径的字符串。
    如果需要密码，则设置成一个数组，数组第一个元素为链接到私有SSL密钥的PEM格式的文件的路径，第二个元素为认证密码。
:类型:
    - 字符串
    - 数组
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::SSL_KEY``

.. note::

    ``ssl_key`` 由HTTP处理器实现。目前只有cURL处理器支持此功能，但其他第三方处理器可能支持此功能。

.. _stream-option:

stream
------

:摘要: 设置成 ``true`` 则是一个流响应，而非下载响应。
:类型: 布尔值
:默认值: ``false``
:常量: ``GuzzleHttp\RequestOptions::STREAM``

.. code-block:: php

    $response = $client->request('GET', '/stream/20', ['stream' => true]);
    // 从流中读取字节，直到到达流的末尾
    $body = $response->getBody();
    while (!$body->eof()) {
        echo $body->read(1024);
    }

.. note::

    流式响应支持必须由客户端使用的HTTP处理器实现。
    每个HTTP处理器可能都不支持此选项，但响应对象的接口保持不变，无论处理器是否支持它。

synchronous
-----------

:摘要: 设置成 ``true`` 来通知HTTP处理器你要等待响应，这有利于优化。
:类型: 布尔值
:默认值: 无
:常量: ``GuzzleHttp\RequestOptions::SYNCHRONOUS``


.. _verify-option:

verify
------

:摘要: 指定请求时验证SSL证书的行为。

    - 设置成 ``true`` 以启用SSL证书验证，默认使用操作系统提供的CA包。
    - 设置成 ``false`` 以禁用证书验证(这是不安全的！)。
    - 设置成一个字符串会启用验证，并使用该字符串作为自定义证书CA包的路径。
:类型:
    - 布尔值
    - 字符串
:默认值: ``true``
:常量: ``GuzzleHttp\RequestOptions::VERIFY``

.. code-block:: php

    // 使用系统的CA包（这是默认设置）
    $client->request('GET', '/', ['verify' => true]);

    // 使用在磁盘上的自定义SSL证书。
    $client->request('GET', '/', ['verify' => '/path/to/cert.pem']);

    // 完全禁用验证（不要这样做！）。
    $client->request('GET', '/', ['verify' => false]);

并非所有的系统磁盘上都存在CA包，比如，Windows和OS X并没有通用的本地CA包。
当设置 ``verify`` 为 ``true`` 时，Guzzle将尽力在你的操作系统中找到合适的CA包.
当使用cURL或PHP 5.6以上版本的流时，将使用默认以上行为。
当使用PHP 5.6以下版本的流时，Guzzle将按以下顺序尝试查找CA包：

1. 检查 ``php.ini`` 文件中是否设置了 ``openssl.cafile``。
2. 检查 ``php.ini`` 文件中是否设置了 ``curl.cainfo``。
3. 检查 ``/etc/pki/tls/certs/ca-bundle.crt``
   是否存在 (Red Hat, CentOS, Fedora; 由 ``ca-certificates`` 包提供)
4. 检查 ``/etc/ssl/certs/ca-certificates.crt``
   是否存在 (Ubuntu, Debian; 由 ``ca-certificates`` 包提供)
5. 检查 ``/usr/local/share/certs/ca-root-nss.crt`` 是否存在 (FreeBSD; 由 ``ca_root_nss`` 包提供)
6. 检查 ``/usr/local/etc/openssl/cert.pem`` 是否存在 (OS X; 由 ``homebrew`` 提供)
7. 检查 ``C:\windows\system32\curl-ca-bundle.crt`` 是否存在 (Windows)
8. 检查 ``C:\windows\curl-ca-bundle.crt`` 是否存在 (Windows)

查找的结果将缓存在内存中，以便同一进程后续快速调用。
然而在有些服务器如Apache中每个请求都在独立的进程中，你应该考虑设置 ``openssl.cafile``
环境变量来指定到磁盘文件，以便整个过程都跳过。

如果你不需要特殊的证书包，可以使用Mozilla提供的通用CA包，你可以在
`这里 <https://raw.githubusercontent.com/bagder/ca-bundle/master/ca-bundle.crt>`_
下载(由cURL的维护者提供)。一旦磁盘有了CA包，你可以设置PHP ini配置文件，指定该文件的路径到变量
``openssl.cafile`` 中，这样就可以在请求中省略 ``verify`` 参数。你可以在
`cURL 网站 <http://curl.haxx.se/docs/sslcerts.html>`_
发现更多关于SSL证书的细节。

.. _timeout-option:

timeout
-------

:摘要: 请求超时的秒数。使用 ``0`` 标识无限期的等待(默认行为)。
:类型: 浮点
:默认值: ``0``
:常量: ``GuzzleHttp\RequestOptions::TIMEOUT``

.. code-block:: php

    // 如果服务器在3.14秒内没有返回响应，则超时。
    $client->request('GET', '/delay/5', ['timeout' => 3.14]);
    // PHP致命错误：Uncaught exception 'GuzzleHttp\Exception\RequestException'

.. _version-option:

version
-------

:摘要: 请求要使用到的协议版本。
:类型:
    - 字符串
    - 浮点值
:默认值: ``1.1``
:常量: ``GuzzleHttp\RequestOptions::VERSION``

.. code-block:: php

    // 强制使用 HTTP/1.0
    $request = $client->request('GET', '/get', ['version' => 1.0]);
