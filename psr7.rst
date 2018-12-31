================
Guzzle 与 PSR-7
================

Guzzle使用 ``PSR-7`` 作为HTTP消息接口。这允许Guzzle与使用PSR-7消息接口的任何其他库一起工作。

Guzzle是一个HTTP客户端，它将HTTP请求发送到服务器并接收HTTP响应。请求和响应都称为 **消息**。

Guzzle依赖于 ``guzzlehttp/psr7`` Composer包来实现 ``PSR-7`` 的消息。

你可以使用 ``GuzzleHttp\Psr7\Request`` 类来创建一个请求：

.. code-block:: php

    use GuzzleHttp\Psr7\Request;

    $request = new Request('GET', 'http://httpbin.org/get');

    // 你还可以提供其他可选的构造函数参数
    $headers = ['X-Foo' => 'Bar'];
    $body = 'hello!';
    $request = new Request('PUT', 'http://httpbin.org/put', $headers, $body);

你可以使用 ``GuzzleHttp\Psr7\Response`` 类来创建一个响应：

.. code-block:: php

    use GuzzleHttp\Psr7\Response;

    // 该构造函数不添加参数
    $response = new Response();
    echo $response->getStatusCode(); // 200
    echo $response->getProtocolVersion(); // 1.1

    // 你还可以提供任意数量的可选参数
    $status = 200;
    $headers = ['X-Foo' => 'Bar'];
    $body = 'hello!';
    $protocol = '1.1';
    $response = new Response($status, $headers, $body, $protocol);

标头
=======

请求和响应消息 **都** 包含HTTP标头。

访问标头
-----------------

你可以使用 ``hasHeader()`` 方法检查请求或响应是否具有特定标头。

.. code-block:: php

    use GuzzleHttp\Psr7;

    $request = new Psr7\Request('GET', '/', ['X-Foo' => 'bar']);

    if ($request->hasHeader('X-Foo')) {
        echo 'It is there';
    }

你可以使用 ``getHeader()`` 方法将所有标头值检索为字符串数组。
You can retrieve all the header values as an array of strings using
``getHeader()``.

.. code-block:: php

    $request->getHeader('X-Foo'); // ['bar']

    // 检索缺失的标头会返回一个空数组。
    $request->getHeader('X-Bar'); // []

你可以使用 ``getHeaders()`` 方法来迭代消息的标头。

.. code-block:: php

    foreach ($request->getHeaders() as $name => $values) {
        echo $name . ': ' . implode(', ', $values) . "\r\n";
    }

复杂标头
---------------

某些标头包含其他键值对信息。例如，``Link`` 标头包含一个链接和几个键值对：

::

    <http://foo.com>; rel="thing"; type="image/jpeg"

Guzzle提供了一个便利功能，可用于解析这些类型的标头：

.. code-block:: php

    use GuzzleHttp\Psr7;

    $request = new Psr7\Request('GET', '/', [
        'Link' => '<http:/.../front.jpeg>; rel="front"; type="image/jpeg"'
    ]);

    $parsed = Psr7\parse_header($request->getHeader('Link'));
    var_export($parsed);

将输出：

.. code-block:: php

    array (
      0 =>
      array (
        0 => '<http:/.../front.jpeg>',
        'rel' => 'front',
        'type' => 'image/jpeg',
      ),
    )

结果包含一个键值对的散列。没有键的标头值值（即链接）以数字方式索引，而形成键值对的标头部分则作为键值对添加。

正文
====

请求和响应消息 **都** 可以包含正文。

你可以使用 ``getBody()`` 方法检索消息的正文：

.. code-block:: php

    $response = GuzzleHttp\get('http://httpbin.org/get');
    echo $response->getBody();
    // JSON字符串: { ... }

请求和响应对象中使用的正文是 ``Psr\Http\Message\StreamInterface``。
此流用于上传数据和下载数据。默认情况下，Guzzle会将消息正文存储在使用PHP临时流的流中。
当正文的大小超过 ``2MB`` 时，流将自动将数据切换存储在磁盘，而不是内存中（保护应用以免内存耗尽）。

为消息创建正文的最简单方法是使用 ``GuzzleHttp\Psr7`` 命名空间中的 ``stream_for``
函数(``GuzzleHttp\Psr7\stream_for``)。
此函数接受字符串、资源、回调、迭代器以及其他流(Stremable)，并返回一个
``Psr\Http\Message\StreamInterface`` 实例。

可以将请求或响应的正文强制转换为字符串，也可以根据需要从流中读取和写入字节。

.. code-block:: php

    use GuzzleHttp\Stream\Stream;
    $response = $client->request('GET', 'http://httpbin.org/get');

    echo $response->getBody()->read(4);
    echo $response->getBody()->read(4);
    echo $response->getBody()->read(1024);
    var_export($response->eof());

请求
========

请求从一个客户端发送到一个服务器。请求包括要应用于资源的方法，资源的标识符以及要使用的协议版本。

请求方法
---------------

创建请求时，你需要提供要执行的HTTP方法。你可以指定任何你想要的方法，包括可能不属于
RFC 7231 的自定义方法（如“MOVE”）。

.. code-block:: php

    // 使用完全自定义的HTTP方法来创建请求
    $request = new \GuzzleHttp\Psr7\Request('MOVE', 'http://httpbin.org/move');

    echo $request->getMethod();
    // MOVE

你可以使用映射到你希望使用的HTTP方法的客户端上的方法来创建和发送请求。

:GET: ``$client->get('http://httpbin.org/get', [/** 选项 **/])``
:POST: ``$client->post('http://httpbin.org/post', [/** 选项 **/])``
:HEAD: ``$client->head('http://httpbin.org/get', [/** 选项 **/])``
:PUT: ``$client->put('http://httpbin.org/put', [/** 选项 **/])``
:DELETE: ``$client->delete('http://httpbin.org/delete', [/** 选项 **/])``
:OPTIONS: ``$client->options('http://httpbin.org/get', [/** 选项 **/])``
:PATCH: ``$client->patch('http://httpbin.org/put', [/** 选项 **/])``

例如：

.. code-block:: php

    $response = $client->patch('http://httpbin.org/patch', ['body' => 'content']);

请求URI
-----------

请求URI由一个 ``Psr\Http\Message\UriInterface`` 对象表示。Guzzle使用
``GuzzleHttp\Psr7\Uri`` 类来提供此接口的实现。

创建请求时，你可以将URI作为字符串或 ``Psr\Http\Message\UriInterface`` 实例来提供。

.. code-block:: php

    $response = $client->request('GET', 'http://httpbin.org/get?q=foo');

模式
------

一个请求的 `scheme <http://tools.ietf.org/html/rfc3986#section-3.1>`_
用以指定发送请求时要使用的协议。使用Guzzle时，模式可以设置为 ``http`` 或 ``https``。

.. code-block:: php

    $request = new Request('GET', 'http://httpbin.org');
    echo $request->getUri()->getScheme(); // http
    echo $request->getUri(); // http://httpbin.org

主机
----

可以使用请求拥有的URI或访问 ``Host`` 标头来访问主机。

.. code-block:: php

    $request = new Request('GET', 'http://httpbin.org');
    echo $request->getUri()->getHost(); // httpbin.org
    echo $request->getHeader('Host'); // httpbin.org

端口
----

使用 ``http`` 或 ``https`` 模式时无需端口。

.. code-block:: php

    $request = new Request('GET', 'http://httpbin.org:8080');
    echo $request->getUri()->getPort(); // 8080
    echo $request->getUri(); // http://httpbin.org:8080


路径
----

可以通过URI对象来访问请求的路径。

.. code-block:: php

    $request = new Request('GET', 'http://httpbin.org/get');
    echo $request->getUri()->getPath(); // /get

将自动过滤路径的内容以确保路径中仅存在允许的字符。路径中不允许的任何字符都将根据
`RFC 3986 section 3.3 <https://tools.ietf.org/html/rfc3986#section-3.3>`_
进行百分比编码(percent-encoded)。

查询字符串
------------

可以使用请求拥有的URI对象的 ``getQuery()`` 方法来访问请求的查询字符串。

.. code-block:: php

    $request = new Request('GET', 'http://httpbin.org/?foo=bar');
    echo $request->getUri()->getQuery(); // foo=bar

将自动过滤查询字符串的内容以确保查询字符串中仅存在允许的字符。查询字符串中不允许的任何字符都将根据
`RFC 3986 section 3.4 <https://tools.ietf.org/html/rfc3986#section-3.4>`_
进行百分比编码(percent-encoded)。

回应
=========

响应是客户端在发送HTTP请求消息后从服务器接收的HTTP消息。

起始行
----------

一个响应的起始行(start-line)包含协议、协议版本、状态代码和原因短语。

.. code-block:: php

    $client = new \GuzzleHttp\Client();
    $response = $client->request('GET', 'http://httpbin.org/get');

    echo $response->getStatusCode(); // 200
    echo $response->getReasonPhrase(); // OK
    echo $response->getProtocolVersion(); // 1.1

正文
----

如前所述，你可以使用 ``getBody()`` 方法来获取响应的正文。

.. code-block:: php

    $body = $response->getBody();
    echo $body;
    // 转换为字符串: { ... }
    $body->seek(0);
    // Rewind the body
    $body->read(1024);
    // 读取正文的字节

流
=======

Guzzle使用PSR-7流对象来表示请求和响应的消息正文。这些流对象允许你使用通用接口来处理各种类型的数据。

HTTP消息由 **起始行**、**标头** 和 **正文** 组成。HTTP消息的正文可能非常小或非常大。
尝试将消息的正文表示为字符串很容易消耗比预期更多的内存，因为正文必须完全存储在内存中。
尝试将请求或响应的正文存储在内存中将阻止使用能够使用大型消息正文的那些实现。
而 ``StreamInterface`` 用于隐藏读取或写入数据流的位置的实现细节。

PSR-7的 ``Psr\Http\Message\StreamInterface`` 暴露了几种方法，可以有效地读取、写入和遍历流。

数据流通过三种方法暴露自己的能力：``isReadable()``、``isWritable()`` 以及
``isSeekable()``。流协作者(collaborator)可以使用这些方法来确定一个流是否能够满足其要求。

每个流实例都具有各种功能：它们可以是只读、只写、读写、允许任意随机访问（向前或向后来寻找任何位置），或仅允许顺序访问（例如在套接字或管道的情况下）。

Guzzle使用 ``guzzlehttp/psr7`` 包提供流支持。有关使用流、创建流、将流转换为PHP流资源以及流装饰器的更多信息，请参阅
`Guzzle PSR-7文档 <https://github.com/guzzle/psr7/blob/master/README.md>`_。

创建流
----------------

创建流的最佳方法是使用 ``GuzzleHttp\Psr7\stream_for`` 函数。此函数接受字符串、从 ``fopen()``
中返回的资源、实现 ``__toString()`` 的对象、迭代器、回调以及和
``Psr\Http\Message\StreamInterface`` 实例。

.. code-block:: php

    use GuzzleHttp\Psr7;

    $stream = Psr7\stream_for('string data');
    echo $stream;
    // 字符串数据
    echo $stream->read(3);
    // str
    echo $stream->getContents();
    // ing data
    var_export($stream->eof());
    // true
    var_export($stream->tell());
    // 11

你可以从迭代器创建流。迭代器每次迭代可以产生(yield)任意数量的字节。
迭代器返回的任何未被流消费者请求的多余字节将被缓冲，直到后续读取。

.. code-block:: php

    use GuzzleHttp\Psr7;

    $generator = function ($bytes) {
        for ($i = 0; $i < $bytes; $i++) {
            yield '.';
        }
    };

    $iter = $generator(1024);
    $stream = Psr7\stream_for($iter);
    echo $stream->read(3); // ...

元数据
--------

流通过 ``getMetadata()`` 方法暴露流元数据。此方法提供在调用PHP的
`stream_get_meta_data()函数 <http://php.net/manual/en/function.stream-get-meta-data.php>`_
时将检索的数据，并且可以选择暴露其他自定义数据。

.. code-block:: php

    use GuzzleHttp\Psr7;

    $resource = fopen('/path/to/file', 'r');
    $stream = Psr7\stream_for($resource);
    echo $stream->getMetadata('uri');
    // /path/to/file
    var_export($stream->isReadable());
    // true
    var_export($stream->isWritable());
    // false
    var_export($stream->isSeekable());
    // true

流装饰器
-----------------

使用流装饰器时，向流添加自定义功能非常简单。Guzzle提供了几个内置装饰器，可提供额外的流功能。

- `AppendStream <https://github.com/guzzle/psr7#appendstream>`_
- `BufferStream <https://github.com/guzzle/psr7#bufferstream>`_
- `CachingStream <https://github.com/guzzle/psr7#cachingstream>`_
- `DroppingStream <https://github.com/guzzle/psr7#droppingstream>`_
- `FnStream <https://github.com/guzzle/psr7#fnstream>`_
- `InflateStream <https://github.com/guzzle/psr7#inflatestream>`_
- `LazyOpenStream <https://github.com/guzzle/psr7#lazyopenstream>`_
- `LimitStream <https://github.com/guzzle/psr7#limitstream>`_
- `NoSeekStream <https://github.com/guzzle/psr7#noseekstream>`_
- `PumpStream <https://github.com/guzzle/psr7#pumpstream>`_
