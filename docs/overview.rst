========
概述
========

需求
============

#. PHP 7.2.5
#. 使用PHP的流处理器时， ``allow_url_fopen`` 必须在php.ini中启用。
#. 要使用cURL处理器时，你必须已经有版本cURL >= 7.19.4，并且编译了OpenSSL 与 zlib。

.. note::

    Guzzle不再需要cURL来发送HTTP请求。如果未安装cURL，Guzzle将使用PHP流包装器发送HTTP请求。
    或者你也可以提供自己的发送HTTP请求的处理方式。请记住，发送并发请求仍然需要cURL。

.. _installation:

安装
============

推荐使用 `Composer <https://getcomposer.org>`_ 来安装Guzzle，Composer是PHP的依赖管理工具，允许你在项目中声明依赖关系，并安装这些依赖。

.. code-block:: bash

    # 安装 Composer
    curl -sS https://getcomposer.org/installer | php

你可以使用Composer客户端将Guzzle作为依赖添加到项目：

.. code-block:: bash

    composer require guzzlehttp/guzzle:^7.0

或者，你可以编辑项目中已存在的 ```composer.json`` 文件，添加 ``Guzzle`` 作为依赖：

.. code-block:: js

    {
      "require": {
         "guzzlehttp/guzzle": "^7.0"
      }
   }

安装完毕后，你需要引入Composer的自动加载文件：

.. code-block:: php

    require 'vendor/autoload.php';

你可以在 `getcomposer.org <https://getcomposer.org>`_
发现更多关于怎样安装Composer、配置自动加载以及其他有用的东西。

开发版
-------------

开发期间，你也可以安装 ``master`` 分支下的最新内容，只需要将Guzzle版本设置成 ``^7.0@dev``：

.. code-block:: js

   {
      "require": {
         "guzzlehttp/guzzle": "^7.0@dev"
      }
   }


升级
=========
Git存储库包含一个 `升级指南`__，详细说明了主要版本之间的变更。

__ https://github.com/guzzle/guzzle/blob/master/UPGRADING.md


许可证
=======

使用基于 `MIT许可 <https://opensource.org/licenses/MIT>`_ 的许可证.

    Copyright (c) 2015 Michael Dowling <https://github.com/mtdowling>

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.

贡献
============

指导方针
----------

1. Guzzle使用PSR-1，PSR-2，PSR-4和PSR-7。
2. Guzzle意味着精简和快速，并且只有很少的依赖。这意味着不会接受每个功能请求。
3. Guzzle具有PHP 7.2的最低PHP版本要求。拉取请求不得要求PHP版本大于PHP 7.2，除非可以有条件地使用该功能，并且文件可以被PHP 7.2解析。
4. 所有拉取请求必须包括单元测试，以确保更改按预期工作并防止回归(regression)。

测试
-----------------

为了做出贡献，你需要从GitHub检出源代码并使用Composer安装Guzzle的依赖：

.. code-block:: bash

    git clone https://github.com/guzzle/guzzle.git
    cd guzzle && composer install

Guzzle是使用PHPUnit进行单元测试的。使用Makefile来运行测试：

.. code-block:: bash

    make test

.. note::

    你需要安装node.js v8或更高版本才能对Guzzle的HTTP处理器执行集成测试。

报告安全漏洞
==================================

我们希望确保Guzzle是一个适合每个人的安全的HTTP客户端库。
如果你在Guzzle中发现了一个安全漏洞，我们非常希望你以
`负责任的方式 <https://en.wikipedia.org/wiki/Responsible_disclosure>`_
向我们披露此漏洞。

公开披露漏洞可能会使整个社区面临风险。如果你发现了安全问题，请发送电子邮件至security@guzzlephp.org。
我们将与你合作，确保我们了解问题的范围，并完全解决你的问题。
我们承诺发送给security@guzzlephp.org的邮件，会是我们的最高优先级，并且尽快的努力解决出现的任何问题。

更正一个安全漏洞后，将尽快部署安全修补程序版本。
