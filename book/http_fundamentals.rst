.. index::
   single: Symfony2 Fundamentals

Symfony2 与 HTTP 基础
==================

恭喜！ 学习 Symfony2 将令你踏上更加  *高产* ， *全面* 和 *受欢迎* 的网页开发者（其实，最终的实现仍取决于你自己）之路。
Symfony2 构建的基本思想是： 开发出不具负面影响，而令你更快速开发并创建健全应用的工具。
Symfony 是基于许多技术的优秀思想： 您即将洞悉的，集万千人多年努力的思想和工具。
换言之，您所学习的不只是  "Symfony"， 您在学习 Symfony2 ，但不局限于 Symfony2 的网页基础，最佳的开发方式以及如何使用不可思议的新 PHP 库。
好啦！准备好！

这一章将讲解对于 Symfony2 哲学也不能例外的 网页开发通用基本概念： HTTP。无论您的背景和偏好的编程语言，
这一章对每位学习者都是 **必读**

HTTP 很简单
--------

HTTP （学究口中的 Hypertext Transfer Protocol） 是一个令两台机器互相沟通的文本语言
就是这样！ 例如，在访问 `xkcd`_ 漫画网时，（基本上）如下的会话就发生了：

.. image:: /images/http-xkcd.png
   :align: center

只是实际上使用的语言更加正式一些， 但仍旧异常简单。
HTTP 就是指这样简单地基于文本的语言。 而且无论您如何开发网页，您的服务器的目标 *总是* 去
理解简单的文字 |request| ，然后给与简单的文本回复。

Symfony2 是构建于这个基本的情形。 无论您是否意识到， 您天天都在使用 HTTP。
跟随 Symfony2，您将掌握它。

.. index::
   single: HTTP; Request-response paradigm

第一步： 客户端发出一个 |request|
~~~~~~~~~~~~~~~~~~~~~~

网站的每一个会话都发起自一个  *请求* 。 请求是客户端（比如：一个浏览器，一个 iPhone 应用程序等）创建的一个特殊格式——HTTP 的文本信息。
客户端把这个请求发送给服务器，之后等待回复。

来看一下一个浏览器与 xkcd 网站服务器交互的第一个部分——请求：

.. image:: /images/http-xkcd-request.png
   :align: center

就 HTTP 而言，这个 HTTP 请求看起来是这样的：

.. code-block:: text

    GET / HTTP/1.1
    Host: xkcd.com
    Accept: text/html
    User-Agent: Mozilla/5.0 (Macintosh)

这段简单的信息传递了客户请求的确切资源所需要的*一切*
HTTP 请求第一行是最为重要的，包含着两重信息： URI 和  HTTP 方法。

URI （例如： ``/`` ， ``/contact`` 等）是用于指定客户所需要的资源的唯一地址或者位置。
HTTP 方法（比如：``GET`` ）定义你要对指定资源*做* 什么。 
HTTP 方法是请求的 *动作* ，即您可以对资源进行的通用操作：

+----------+-----------------------------------------+
| *GET*    | |Retrieve the resource from the server| |
+----------+-----------------------------------------+
| *POST*   | |Create a resource on the server|       |
+----------+-----------------------------------------+
| *PUT*    | |Update the resource on the server|     |
+----------+-----------------------------------------+
| *DELETE* | |Delete the resource from the server|   |
+----------+-----------------------------------------+

本着这样的思想，你可以想象用于删除一条博客的 HTTP 请求是怎样的：

.. code-block:: text

    DELETE /blog/15 HTTP/1.1

.. note::

    其实，HTTP 规范规定了 9 种 HTTP 方法
    但是其中大部分并没有得到广泛应用和支持。实际上，很多现代
    浏览器都不支持 ``PUT`` 和 ``DELETE`` 方法。

在第一行之后，HTTP 请求总是跟随着另外的信息行，被称作请求 |request headers| .
|header| 用以提供很多信息，例如：请求的 |``Host``| ，客户端接受（ |``Accept``| ）的 |response| 格式，
以及客户端用以发出 |request| 的应用 （ |``User-Agent``| ）。
还有很多 |header| 可以在 Wikipedia的文章  `List of HTTP header fields`_ 中找到。

第2步：服务器返回 |Response|
~~~~~~~~~~~~~~~~~~~~

一旦服务器受到请求，便知晓了客户所需要的确切资源以及客户端希望如何处理该资源（通过 URI ）。
例如： 收到一个 GET |request| ,服务器准备资源并将其放在 |HTTP response| 当中返回。
来看一下 xkcd 服务器的 |response| :

.. image:: /images/http-xkcd.png
   :align: center

用 HTTP 语言来表述， 返回给浏览器的应答应该大体是这样的： 

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 02 Apr 2011 21:05:05 GMT
    Server: lighttpd/1.4.19
    Content-Type: text/html

    <html>
      <!-- HTML for the xkcd comic -->
    </html>

HTTP |response| 不仅包含请求的资源（ 这个例子中的 HTTP ），也包含了其他关于 |response| 的信息。
非常重要的第一行包含了 HTTP |response| 状态码 （此例中是200）。状态码传达了 |request| 总体结果给客户端。
|request| 是否成功？ 放生错误没有？ 不同的状态码标示着：成功、错误以及客户端需要做什么（例如：转到另一页面）。
完整的列表可以在  Wikipedia 的 `List of HTTP status codes`_ 一文中找到。

与 |request| 类似，一个 HTTP |response| 也包含着名为 HTTP |header| 的额外信息。
例如： ``Content-Type`` 就是一个重要的 HTTP |response| |header| 。
同一资源可以以多种不同的格式被提供： HTML、XML 或者 JSON，而 ``Content-Type`` |header|
类似于 ``text/html`` 的则用 |Internet Media Types| 告诉客户端返回资源的格式。
|Common Media Types| 列表可以在 Wikipedia 的 `List of common media types`_ 一文中找到。

还有很多其他的 |header| , 而且很多非常有用。 例如：有的 |header| 可以用于生产强大的 |caching| 系统。

|request| , |response| 和 |web development|
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

|request| - |response| 对话是 |web| 所有对话的基础。
尽管它是如此的重要和强大，却其实异常简单。

最重要的事实是：无论您使用何种语言，构建了何种类型的应用（web, mobile, JSON API），或是基于怎样的开发思想，
终极目标 **总是** 理解 |request| 、创建并返回恰当的 |response|

Symfony 就是本着这样的事实构建的。

.. tip::

        阅读 `HTTP 1.1 RFC`_ 学习 HTTP 规范，或者很好地解读规范的 `HTTP Bis`_
        Firefox（火狐浏览器）的扩展 `Live HTTP Headers`_ 是查看 |request| 和 |response| 的利器。
    
.. index::
   single: Symfony2 Fundamentals; Requests and responses

PHP 中的 |request| 和 |response|
-----------------------------

那么，在使用 PHP 时，如何与 ''|request|'' 交互并创建 "|response|" 呢？
现实中， PHP 抽象化了整个过程：

.. code-block:: php

    <?php
    $uri = $_SERVER['REQUEST_URI'];
    $foo = $_GET['foo'];

    header('Content-type: text/html');
    echo 'The URI requested is: '.$uri;
    echo 'The value of the "foo" parameter is: '.$foo;

听起来诧异，这个小小的应用其实接收了 |HTTP request| 并且用其构建了 |HTTP response| 。
PHP 准备了类似于 ``$_SERVER`` 和 ``$_GET`` 的超级全局变量来承载所有 |request| 信息，
因此没有必要处理原始的 |HTTP request| 信息。 
同理，可以使用  ``header()`` 函数来创建 |response| |header| ,直接生成 |response| 信息中需要的内容，
不必书写 HTTP 格式的 |response| 文本。PHP 将创建一个实实在在的 HTTP |response| 并将其返回给客户端：

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 03 Apr 2011 02:14:33 GMT
    Server: Apache/2.2.17 (Unix)
    Content-Type: text/html

    The URI requested is: /testing?foo=symfony
    The value of the "foo" parameter is: symfony

Symfony 中的 |request| 和 |response|
---------------------------------

Symfony 提供了替代 PHP 处理方法的两个 |class| ，可以通过他们更加容易地进行 |HTTP request| 和 |response| 交互。
:class:`Symfony\\Component\\HttpFoundation\\Request` |class| 简单地将 |HTTP request| 以 |object-oriented| 方式呈现。
有了它，所有的 |request| 信息都在你的股掌之间::

    use Symfony\Component\HttpFoundation\Request;

    $request = Request::createFromGlobals();

    // the URI being requested (e.g. /about) minus any query parameters
    $request->getPathInfo();

    // retrieve GET and POST variables respectively
    $request->query->get('foo');
    $request->request->get('bar', 'default value if bar does not exist');

    // retrieve SERVER variables
    $request->server->get('HTTP_HOST');

    // retrieves an instance of UploadedFile identified by foo
    $request->files->get('foo');

    // retrieve a COOKIE value
    $request->cookies->get('PHPSESSID');

    // retrieve an HTTP request header, with normalized, lowercase keys
    $request->headers->get('host');
    $request->headers->get('content_type');

    $request->getMethod();          // GET, POST, PUT, DELETE, HEAD
    $request->getLanguages();       // an array of languages the client accepts

不仅如此， ``Request`` |class| 作了很多幕后工作，完全不用你操心。
例如： ``isSecure()`` |method| 通过检查 PHP 的 *3* 个不同的值来确定用户是否正以安全连接方式进行访问 （例如： ``https`` ）。

.. sidebar:: |ParameterBags| 和 |request| 属性

    如上所见，变量 ``$_GET`` 和 ``$_POST`` 是通过 |public| 属性  ``query`` 和 ``request`` 访问的。
    他们都是 :class:`Symfony\\Component\\HttpFoundation\\ParameterBag` |object| ，都有如下方法：
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::get` ，
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::has` ，
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::all` 等等。
    事实上，前面的例子中的所有 |public| ParameterBag 的 |instance|.
    
    .. _book-fundamentals-attributes:
    
    Request |class| 还有一个 |public| |``attributes``| 用于存放 |application| 内部如何工作的特殊数据。
    例如： Symfony2 |framework| 的 |``attributes``| 储存对应 |route| 返回的 ``_controller`` ，
    ``id`` （如果设置了  ``{id}`` 匹配），以及对应的 ``_route`` 。
    |``attributes``| 的使命就是用于准备和存储 |request| 的 |context-specific| 信息。
    

Symfony 还提供了一个  ``Response`` |class| : 简单地以 PHP 承载 |HTTP response| 信息
这样你的 |application| 就可以通过 |object-oriented| 界面来构建需要返回给客户端的 |response|::

    use Symfony\Component\HttpFoundation\Response;
    $response = new Response();

    $response->setContent('<html><body><h1>Hello world!</h1></body></html>');
    $response->setStatusCode(200);
    $response->headers->set('Content-Type', 'text/html');

    // prints the HTTP headers followed by the content
    $response->send();

不考虑 Symfony 带来的其它功能，你至少已经有了方便访问 |request| 信息的工具包和用于创建 |response| 的 |object-oriented| 界面。
尽管您将学到很多强大的 Symfony 功能，但不要忘记您的 |application| 的宗旨始终是
 *解析 |request| 并基于 |application| 逻辑创建恰当的 |response| *

.. tip::

     ``Request`` 和 ``Response`` |class| 是 Symfony 中名为  ``HttpFoundation`` 的独立 |component| 的一部分。
     该 |component| 可以完全独立于 Symfony 使用，用于提供处理 |session| 和 文件上传的 |class| 。

从 |request| 到 |response| 的旅程
----------------------------

就如同 HTTP 本身， ``Request`` 和 ``Response`` |object| 非常地简单。
构建 |application| 的困难之处在于编写他们之间的内容。
换句话来讲，真正的工作在于编写解析 |request| 信息并创建 |response| 的代码。

您的 |application| 或许会做很多事情，比如：发电子邮件、处理提交内容、进行数据库存储、生成 HTML 页面以及保护内容安全。
您如何才能在确保代码有条不紊且易于维护的前提下做到这一切呢？

Symfony 就是为了解决这些问题而创造的，您再也不必操心它们。

|front controller|
~~~~~~~~~~~~~~~~~~

传统方式构建的 |application| ，网站的每个 "页面" 就是它自身的物理文件:

.. code-block:: text

    index.php
    contact.php
    blog.php

这种做法有一些问题，比如：缺乏 URL 灵活性 （如何在不打破链接的前提下将 ``blog.php`` 改为 ``news.php`` ？），
为了确保安全、连接数据库以及确保网站的一致外观 *必须* 在每一个文件中手工引用一些列核心文件。

更好的解决方式是应用 :term:|`front controller`|: 处理每个到访您的 |application| 的 |request| 的一个简单的 PHP 文件。 例如：

+------------------------+--------------------------+
| ``/index.php``         | |executes| ``index.php`` |
+------------------------+--------------------------+
| ``/index.php/contact`` | |executes| ``index.php`` |
+------------------------+--------------------------+
| ``/index.php/blog``    | |executes| ``index.php`` |
+------------------------+--------------------------+

.. tip::

            使用  Apache ``mod_rewrite`` （或者其他网站服务的类似功能），
    URL 可以轻松地被清理为 ``/`` 、 ``/contact`` 和 ``/blog`` 。

这样，每个 |request| 都被同样地处理。 不再是每个 URL 执行不同的 PHP 文件，而是每次 *总是* 执行 |front controller| ，
在其内部，不同的 URL 被 |routing| 到 |application| 的不同部分。
这样上文提到的传统做法的两个问题都得到了解决。
几乎所有现代网络应用都是这样做的 - 包括类似 WordPress 的应用。

确保条例
~~~~

但是，在您的 |front controller| 内部， 如何获悉应该处理哪个页面，又该如何理智地处理呢？
无论如何，您都需要查看 URI 并根据其内容来确定执行您的代码的不同部分。 这个工作惊人的快：

.. code-block:: php

    // index.php

    $request = Request::createFromGlobals();
    $path = $request->getPathInfo(); // the URI path being requested

    if (in_array($path, array('', '/')) {
        $response = new Response('Welcome to the homepage.');
    } elseif ($path == '/contact') {
        $response = new Response('Contact us');
    } else {
        $response = new Response('Page not found.', 404);
    }
    $response->send();

解决这个问题会比较困难。 幸运地是， Symfony *就* 是为解决这个困难设计的。

Symfony |Application| 工作流程
~~~~~~~~~~~~~~~~~~~~~~~~~~

当你将处理 |request| 的工作交给 Symfony ,一切变得容易了。
Symfony 对于每一个 |request| 都奉行相同的模式：

.. _request-flow-figure:

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   收到的 |request| 被 |routing| 解读并被传递给返回 ``Response`` |object| 的 |controller| 函数。

网站的每个 "页面" 都在 |routing| 配置文件中定义，从而将每个 URL 与不同的 PHP 函数对应起来。 
每个 PHP 函数，被称为 :term:`controller` ，的工作就是使用 |request| 提供的信息，
连同 Symfony 提供的一切其他工具，创建并返回一个 ``Response`` |object| 。
换言之， |controller| 就是 *您* 存代码的地方： 在那里解读 |request| 并创建 |response| 。

就是这么简单！ 复习一下：

* 每个 |request| 执行（同一个） |front controller| 文件；

* |routing| 系统决定根据 |request| 信息和 |routing| 设置决定执行哪个 PHP 函数；

* 执行应执行的 PHP 函数，该函数的 代码创建并返回恰当的  ``Response`` |object| 。

|action| 中处理一个 Symfony |request|
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

让我们深入浅出地了解一下 |action| 的工作过程。 
假设您想要为您的 Symfony |application| 添加一个 ``/contact`` 页面。
首先，在 |routing| 配置文件中为 ``/contact`` 添加一条 |entry| ：

.. code-block:: yaml

    contact:
        pattern:  /contact
        defaults: { _controller: AcmeDemoBundle:Main:contact }

.. note::

          这个例子使用  :doc:`YAML</components/yaml>` 定义 |routing| 设置。 
   |routing| 设置还可以使用其他格式来书写，例如： XML 和 PHP。

当 ``/contact`` 页面被访问，该 |route| 被匹配，所指定的 |controller| 被执行。 
正如您将在 |:doc:`routing chapter</book/routing>`| 一章中学到，
the ``AcmeDemoBundle:Main:contact`` 字符串是一个指向 ``contactAction`` 
|class| 之中名为  ``MainController`` 的 PHP |method| 的简单 |syntax| ：

.. code-block:: php

    class MainController
    {
        public function contactAction()
        {
            return new Response('<h1>Contact us!</h1>');
        }
    }

在这个简单的例子中， |controller| 仅仅建立了一个包含 HTML “<h1>Contact us!</h1>” 的 ``Response`` |object| 。
在  |:doc:`controller chapter</book/controller>`| 一章中, 您将学到如何使用 |controller| |render| |template| ，
并且允许从不同的 |template| 文件调用代码（书写 HTML 的一切内容）实现 “呈现” 工作。
这使得 controller 被解放出来，并专注在难点之上：数据库交互，提交数据处理以及发送邮件信息。 

Symfony2：创建您的 |app| ,而不是您的工具。
-----------------------------

您已经清楚任何一个 |app| 的目标都是解读收到的 |request| 并创建一个恰当的 |response| 。
随着 |application| 扩展，确保代码的条理性和维护性的难度不断增加。 不变的是，相同的工作被不断地重复：
在数据库中存储数据， |render| 和重用 |template| ，处理提交数据，发送电子邮件，验证用户输入并确保安全性。

令人欣喜的是，这些问题都是普遍存在的。 Symfony 提供了一个包含各种工具的 |framework| 
，您可以用它来构建您的 |application| 而无需顾忌工具的创建
使用 Symfony2 后，您不再有烦恼：你可以自由地使用这个 Symfony |framework| ，或者只是单独使用 Symfony 的一个片断。

.. index::
   single: Symfony2 Components

独立工具： Symfony2: |*component*|
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

那么 Symfony2 是什么呢？ 首先， Symfony2 是 20 余个可以在*任何* PHP 项目中使用的独立库文件的集合。
无论您是怎样开发项目的，在几乎任何情形之下，这些被称为 *Symfony2 |component| *的库都有用武之地。
举几个例子：

* `HttpFoundation`_ - 包含 ``Request`` 、 ``Response`` 和一些其他 |class| 用于处理 |session| 和文件上传；

* `Routing`_ - 将特定 URL （例如： ``/contact`` ）映射为
       如何处理 |request| 的信息（例如执行 ``contactAction()`` |method| ）的强大、快速的 |routing| 系统;

* `Form`_ - 一个创建表单并且处理提交数的，功能齐全而且灵活的架构；

* `Validator`_ 一个用以创建数据规则，并以之验证用户提交数据的系统；

* `ClassLoader`_ 无需使用 ``require`` 引用文件的方式就可以自动调用 PHP |class| 的库；

* `Templating`_ |render| |template| ，处理 |template| 继承（通过布局装饰 |template| ）
      并且执行其他通用 |template| 工作的工具集；

* `Security`_ - 一个功能强大，处理 |application| 内部所有安全问题的库；

* `Translation`_ 一个用于翻译 |application| 中字符串的 |framework| 。

无论是否在使用 Symfony2 架构，这些 |component| 都是可以单独地使用于 *任何* PHP 项目之中的。
任何部分都可根据需要随意调配。

完整的解决方案： The Symfony2 |*framework*|
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

那么， Symfony2 |*framework*| *是* 什么呢？ |*Symfony2 framework*| 是一个实现下面两个独立工作的 PHP 库：

#. 提供 |component| （Symfony2 |component| ）以及第三方库（例如： 用于发送邮件的 ``Swiftmailer`` ）供选择；

#. 提供切合实际的配置并将各个部分“粘合”在一起的库。

|framework| 的目标是将许多独立工具整合起来提供给开发者一个综合体验。
|framework| 本身就是一个 |bundle| ，可以被配置也可以被完全删除。

Symfony2 提供了快速开发 |web| |application| 的工具集，丝毫不会增加 |application| 的负担
通常用户可以快速地使用 Symfony2 进行开发，直接提供了一个默认的合理项目构架。
对于高端用户来讲，更是天高任鸟飞。

.. _`xkcd`: http://xkcd.com/
.. _`HTTP 1.1 RFC`: http://www.w3.org/Protocols/rfc2616/rfc2616.html
.. _`HTTP Bis`: http://datatracker.ietf.org/wg/httpbis/
.. _`Live HTTP Headers`: https://addons.mozilla.org/en-US/firefox/addon/3829/
.. _`List of HTTP status codes`: http://en.wikipedia.org/wiki/List_of_HTTP_status_codes
.. _`List of HTTP header fields`: http://en.wikipedia.org/wiki/List_of_HTTP_header_fields
.. _`List of common media types`: http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types
.. _`HttpFoundation`: https://github.com/symfony/HttpFoundation
.. _`Routing`: https://github.com/symfony/Routing
.. _`Form`: https://github.com/symfony/Form
.. _`Validator`: https://github.com/symfony/Validator
.. _`ClassLoader`: https://github.com/symfony/ClassLoader
.. _`Templating`: https://github.com/symfony/Templating
.. _`Security`: https://github.com/symfony/Security
.. _`Translation`: https://github.com/symfony/Translation

.. include:: ../_terminology.rst