=======================
处理器和中间件
=======================

Guzzle客户端使用处理器和中间件系统来发送HTTP请求。

处理器
========

一个处理器函数接受一个 ``Psr\Http\Message\RequestInterface``
和一个请求选项数组，并返回一个用 ``Psr\Http\Message\ResponseInterface`` 填充的
``GuzzleHttp\Promise\PromiseInterface``，或被一个异常拒绝。

你可以使用客户端构造函数的 ``handler`` 选项为客户端提供一个自定义处理器。
重要的是要理解Guzzle使用的几个请求选项要求用特定的中间件来封装客户端要使用的处理器。
你可以通过在 ``GuzzleHttp\HandlerStack::create(callable $handler = null)``
静态方法中封装处理器来确保你提供给客户端的处理器会使用默认中间件。

.. code-block:: php

    use GuzzleHttp\Client;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Handler\CurlHandler;

    $handler = new CurlHandler();
    $stack = HandlerStack::create($handler); // 封装 w/ 中间件
    $client = new Client(['handler' => $stack]);

``create`` 方法将默认处理器添加到 ``HandlerStack``。当 ``HandlerStack``
被解析后(resolved)，这些处理器将按照以下顺序执行：

1. 发送请求：

  1. ``http_errors`` - 发送一个请求时没有操作。
     当一个响应Promise返回到堆栈时，会在响应处理中检查响应状态代码。
  2. ``allow_redirects`` - 发送一个请求时没有操作。当一个响应Promise返回到堆栈时，会发生重定向。
  3. ``cookies`` - 为请求添加Cookie。
  4. ``prepare_body`` - 将预设HTTP请求的主体（例如，添加
     ``Content-Length``、``Content-Type`` 等默认标头）。
  5. <使用处理器发送请求>

2. 处理响应：

  1. ``prepare_body`` - 没有关于响应处理的操作。
  2. ``cookies`` - 将响应Cookie提取到Cookie Jar中。
  3. ``allow_redirects`` - 遵循重定向。
  4. ``http_errors`` - 当响应状态码 ``>=`` 300时抛出异常。

如果没有提供 ``$handler`` 参数，``GuzzleHttp\HandlerStack::create()``
将根据你的系统上可用的扩展来选择最合适的处理器。

.. important::

    提供给客户端的处理器确定了如何为客户端发送的每个请求来应用和使用请求选项。
    例如，如果未将Cookie中间件与客户端关联，则设置 ``cookies``
    请求选项将不会对请求产生任何影响。

中间件
==========

中间件通过在生成响应的过程中调用它们来增强处理器的功能。中间件实现为一个高阶函数，采用以下形式：

.. code-block:: php

    use Psr\Http\Message\RequestInterface;

    function my_middleware()
    {
        return function (callable $handler) {
            return function (RequestInterface $request, array $options) use ($handler) {
                return $handler($request, $options);
            };
        };
    }

中间件函数返回一个函数，该函数接受下一个要调用的处理器。
这个返回的函数接着返回另一个充当组合(composed)处理器的函数 --
它接受一个请求和选项，并返回一个用响应填充的Promise。
你的组合的中间件可以修改请求、添加自定义请求选项，以及修改下游处理器返回的Promise。

这是为每个请求都添加一个标头的示例：

.. code-block:: php

    use Psr\Http\Message\RequestInterface;

    function add_header($header, $value)
    {
        return function (callable $handler) use ($header, $value) {
            return function (
                RequestInterface $request,
                array $options
            ) use ($handler, $header, $value) {
                $request = $request->withHeader($header, $value);
                return $handler($request, $options);
            };
        };
    }

一旦创建了中间件，你可以通过封装客户端使用的处理器或通过装饰一个处理器堆栈来将其添加到客户端。

.. code-block:: php

    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Handler\CurlHandler;
    use GuzzleHttp\Client;

    $stack = new HandlerStack();
    $stack->setHandler(new CurlHandler());
    $stack->push(add_header('X-Foo', 'bar'));
    $client = new Client(['handler' => $stack]);

现在，当你发送一个请求时，客户端将使用由你添加的中间件组成的处理器，来为每个请求添加标头。

以下是创建修改下游处理器响应的中间件的示例。此示例为响应添加一个标头。

.. code-block:: php

    use Psr\Http\Message\RequestInterface;
    use Psr\Http\Message\ResponseInterface;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Handler\CurlHandler;
    use GuzzleHttp\Client;

    function add_response_header($header, $value)
    {
        return function (callable $handler) use ($header, $value) {
            return function (
                RequestInterface $request,
                array $options
            ) use ($handler, $header, $value) {
                $promise = $handler($request, $options);
                return $promise->then(
                    function (ResponseInterface $response) use ($header, $value) {
                        return $response->withHeader($header, $value);
                    }
                );
            };
        };
    }

    $stack = new HandlerStack();
    $stack->setHandler(new CurlHandler());
    $stack->push(add_response_header('X-Foo', 'bar'));
    $client = new Client(['handler' => $stack]);

使用 ``GuzzleHttp\Middleware::mapRequest()`` 中间件可以更加简单的创建一个用于修改请求的中间件。
此中间件接受一个请求参数并返回要发送的请求的函数。

.. code-block:: php

    use Psr\Http\Message\RequestInterface;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Handler\CurlHandler;
    use GuzzleHttp\Client;
    use GuzzleHttp\Middleware;

    $stack = new HandlerStack();
    $stack->setHandler(new CurlHandler());

    $stack->push(Middleware::mapRequest(function (RequestInterface $request) {
        return $request->withHeader('X-Foo', 'bar');
    }));

    $client = new Client(['handler' => $stack]);

使用 ``GuzzleHttp\Middleware::mapResponse()`` 中间件使得修改响应更加简单。

.. code-block:: php

    use Psr\Http\Message\ResponseInterface;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Handler\CurlHandler;
    use GuzzleHttp\Client;
    use GuzzleHttp\Middleware;

    $stack = new HandlerStack();
    $stack->setHandler(new CurlHandler());

    $stack->push(Middleware::mapResponse(function (ResponseInterface $response) {
        return $response->withHeader('X-Foo', 'bar');
    }));

    $client = new Client(['handler' => $stack]);

处理器堆栈
============

处理器堆栈表示一个要应用于基本处理器函数的中间件堆栈。
你可以将中间件推送(push)到堆栈以将其添加到堆栈的顶部，也可以将中间件卸载(unshift)到堆栈中以将其添加到堆栈的底部。
堆栈被解析后，该处理器将被推送到堆栈。然后从堆栈中弹出(popped)每个值，并封装从堆栈中弹出的前一个值。
When the stack is resolved, the handler is pushed onto the stack.
Each value is then popped off of the stack, wrapping the previous value popped off of the
stack.

.. code-block:: php

    use Psr\Http\Message\RequestInterface;
    use GuzzleHttp\HandlerStack;
    use GuzzleHttp\Middleware;
    use GuzzleHttp\Client;

    $stack = new HandlerStack();
    $stack->setHandler(\GuzzleHttp\choose_handler());

    $stack->push(Middleware::mapRequest(function (RequestInterface $r) {
        echo 'A';
        return $r;
    });

    $stack->push(Middleware::mapRequest(function (RequestInterface $r) {
        echo 'B';
        return $r;
    });

    $stack->push(Middleware::mapRequest(function (RequestInterface $r) {
        echo 'C';
        return $r;
    });

    $client->request('GET', 'http://httpbin.org/');
    // echoes 'ABC';

    $stack->unshift(Middleware::mapRequest(function (RequestInterface $r) {
        echo '0';
        return $r;
    });

    $client = new Client(['handler' => $stack]);
    $client->request('GET', 'http://httpbin.org/');
    // echoes '0ABC';

你可以为中间件提供一个名称，以允许你在其他命名的中间件前面、后面添加中间件，或者按名称来移除中间件。

.. code-block:: php

    use Psr\Http\Message\RequestInterface;
    use GuzzleHttp\Middleware;

    // 使用名称来添加一个中间件
    $stack->push(Middleware::mapRequest(function (RequestInterface $r) {
        return $r->withHeader('X-Foo', 'Bar');
    }, 'add_foo');

    // 在命名的中间件之前添加中间件(unshift before).
    $stack->before('add_foo', Middleware::mapRequest(function (RequestInterface $r) {
        return $r->withHeader('X-Baz', 'Qux');
    }, 'add_baz');

    // 在命名的中间件之后添加一个中间件 (pushed after)
    $stack->after('add_baz', Middleware::mapRequest(function (RequestInterface $r) {
        return $r->withHeader('X-Lorem', 'Ipsum');
    });

    // 按名称移除中间件
    $stack->remove('add_foo');


创建处理器
==================

如前所述，一个处理器是一个函数，它接受一个 ``Psr\Http\Message\RequestInterface``
和一个请求选项数组，并返回一个用 ``Psr\Http\Message\ResponseInterface`` 填充的
``GuzzleHttp\Promise\PromiseInterface``，或被一个异常拒绝。

一个处理器负责应用以下 :doc:`request-options`。这些请求选项是称为“传输选项”的请求选项的子集。

- :ref:`cert-option`
- :ref:`connect_timeout-option`
- :ref:`debug-option`
- :ref:`delay-option`
- :ref:`decode_content-option`
- :ref:`expect-option`
- :ref:`proxy-option`
- :ref:`sink-option`
- :ref:`timeout-option`
- :ref:`ssl_key-option`
- :ref:`stream-option`
- :ref:`verify-option`
