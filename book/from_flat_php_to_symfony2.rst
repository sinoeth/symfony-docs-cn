Symfony2 与 直接使用 PHP
===================

**为什么用 Symfony2 开发比打开一个文件直接编写 PHP 代码好的？**

如果您没有接触过 PHP |framework|, 也不清楚 MVC 理论，或者对 Symfony2 的花团锦簇感到疑惑，那么这一章就是写给您的。
这里您不会被 *告知* Symfony2 让您的开发更快速、更好的软件，您将亲身见证。

这一章中，您将用 PHP 直接编写一个简单的 |application| ，之后将其重组并优化它。 
穿越这个旅程， 您将看到在过去的几年中，网页开发一步步发展至今的步步抉择。

最后，您将看到 Symfony2 会从乏味的工作中解脱出来，并重新成为您的代码的控制者。

PHP 直接编写的一个简单博客
---------------

这一章，您将直接使用 PHP 创建 token 博客 |application| 。
首先，创建一个现实数据库已经存储的博客条目的简单页面。 直接用 PHP 来写虽然块但并不讲究：

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);
    ?>

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php while ($row = mysql_fetch_assoc($result)): ?>
                <li>
                    <a href="/show.php?id=<?php echo $row['id'] ?>">
                        <?php echo $row['title'] ?>
                    </a>
                </li>
                <?php endwhile; ?>
            </ul>
        </body>
    </html>

    <?php
    mysql_close($link);

写起来快，执行起来也快，但是随着 |app| 不断扩大，维护将变得不可能。 会出现如下几个问题：

* |**No error-checking**| ： 如果数据连结失败怎么办？

* |**Poor organization**| ：随之 |application| 的增长，但以文件将变得越来越难以维护。
  处理表单提交的代码放在哪？如何验证数据？  发送电子邮件的代码放在哪里？

* |**Difficult to reuse code**| ： 因为一切都在一个文件之中，博客的其他 “页面”没办法重用 |application| 的任何部分。

.. note::
    另一个没有提及的问题原自与 MySQL 相连的事实。 这里虽然没有涉及，但 Symfony2 完全集成了一个精于数据库抽象和映射的库 `Doctrine`_ 。

下面来着手解决上面的问题。

分离呈现层
~~~~~

可以通过分离 |application| “逻辑” 和准备 HTML “呈现”的代码获得：

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);

    $posts = array();
    while ($row = mysql_fetch_assoc($result)) {
        $posts[] = $row;
    }

    mysql_close($link);

    // include the HTML presentation code
    require 'templates/list.php';

现在， HTML 代码存储于另外的文件（ ``templates/list.php`` ），这是一个类似于 |template| ，使用 PHP 句法的 HTML 文件：

.. code-block:: html+php

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php foreach ($posts as $post): ?>
                <li>
                    <a href="/read?id=<?php echo $post['id'] ?>">
                        <?php echo $post['title'] ?>
                    </a>
                </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

约定俗成地将包含 |application| 逻辑的文件 - ``index.php``称作“ |controller| [controller]”。 
无论您使用哪种编程语言或者 |framework| ,您都将反反复复地听到 :term:`controller`这个名字。 
它是 *您的* 代码中，负责处理您的输入并准备 |response| 的部分。

在这里， |controller| 准备来自数据库的数据，引用一个模板并呈现这些数据。 |controller| 得到了分离后， 
在需要以不同格式（例如：为JSON格式建立 ``list.json.php`` ）呈现博客记录时就可以 *仅仅* 修改 |template| 。

分离 |application| （ |domain| （Domain）） 逻辑
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

目前 |application| 只有一个页面。但是如果第二个页面需要使用相同的数据库连接，或者相同的博客帖子该怎么办？
通过重构代码将 |application| 的核心行为和数据库访问函数独立存放于一个新文件 ``model.php`` ：

.. code-block:: html+php

    <?php
    // model.php

    function open_database_connection()
    {
        $link = mysql_connect('localhost', 'myuser', 'mypassword');
        mysql_select_db('blog_db', $link);

        return $link;
    }

    function close_database_connection($link)
    {
        mysql_close($link);
    }

    function get_all_posts()
    {
        $link = open_database_connection();

        $result = mysql_query('SELECT id, title FROM post', $link);
        $posts = array();
        while ($row = mysql_fetch_assoc($result)) {
            $posts[] = $row;
        }
        close_database_connection($link);

        return $posts;
    }

.. tip::

   文件被命名为 ``model.php`` 是因为一个 |application| 的逻辑和数据库访问被称作 “ |model| （model）” 层。
   在一个组织良好的 |application| 之中，用来支撑 “业务逻辑”的大部分代码应该位于 |model| （决不能存放于 |controller| ）。
   与此例不同，通常数据库访问之占 |model| 的一小部分（甚至没有）。

现在， |controller| （ ``index.php`` ）非常简单：

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $posts = get_all_posts();

    require 'templates/list.php';

现在， |controller| 唯一的工作就是从 |model| 层获取数据并调用一个 |template| 来呈现数据。
这是一个极其简单的 |model| - |view| - |controller| 的例子。

分离 |layout|
~~~~~~~~~~~

到目前为止， |application| 已经被重构为三个部分，它们提供了很多优势，也为不同页面间重用几乎所有内容提供了机会。

唯一 *不能* 被重用的是页面 |layout| 。 创建一个 ``layout.php`` 文件就能解决这个问题：

.. code-block:: html+php

    <!-- templates/layout.php -->
    <html>
        <head>
            <title><?php echo $title ?></title>
        </head>
        <body>
            <?php echo $content ?>
        </body>
    </html>

现在 |template| （ ``templates/list.php`` ）可以简单地 “ |extend| ” |layout| ：

.. code-block:: html+php

    <?php $title = 'List of Posts' ?>

    <?php ob_start() ?>
        <h1>List of Posts</h1>
        <ul>
            <?php foreach ($posts as $post): ?>
            <li>
                <a href="/read?id=<?php echo $post['id'] ?>">
                    <?php echo $post['title'] ?>
                </a>
            </li>
            <?php endforeach; ?>
        </ul>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

您现在已经知晓了一个允许重用 |layout| 的方法。 
不幸的是，您不得不在 |template| 中使用一点不太讲究的 PHP 函数（ ``ob_start()``, ``ob_get_clean()`` ）
Symfony2 使用一个  ``Templating`` |component| 是做到这点清新和轻松。很很快就会在 |action| 中看到。

添加一个博客 “show”页面
---------------

现在博客 “list”页面已被重构，代码被更好地组织，提高了重用性。
为了证实这一点，添加一个博客 “show”页面用以通过 ``id`` 参数显示单个博客帖子。

首先，在 文件中创建一个新的函数用以基于 id 获取制定的博客帖子::

    // model.php
    function get_post_by_id($id)
    {
        $link = open_database_connection();

        $id = mysql_real_escape_string($id);
        $query = 'SELECT date, title, body FROM post WHERE id = '.$id;
        $result = mysql_query($query);
        $row = mysql_fetch_assoc($result);

        close_database_connection($link);

        return $row;
    }

其次，创建一个名为 ``show.php`` 的新文件 —— 这个新页面的 |controller| ：

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $post = get_post_by_id($_GET['id']);

    require 'templates/show.php';

最后，创建一个新的 |template| 文件—— ``templates/show.php`` ，来呈现这个博客：

.. code-block:: html+php

    <?php $title = $post['title'] ?>

    <?php ob_start() ?>
        <h1><?php echo $post['title'] ?></h1>

        <div class="date"><?php echo $post['date'] ?></div>
        <div class="body">
            <?php echo $post['body'] ?>
        </div>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

创建第二个页面非常容易而且没有出现重复的代码。 这个页面存在一个不可姑息的问题，但是 |framework| 会帮你解决。
例如： 一个丢失或者不合规则的 ``id`` 参数将会造成页面崩溃。
最好这种情况一个 404 页面被呈现，但却不是很容易做到的。
更麻烦的是，你有没有忘记使用 ``mysql_real_escape_string()`` 函数清楚掉 ``id`` ？
一个 SQL |injection| 攻击就可能导致这个数据库的危险。

另一个大问题是每个 |controller| 文件必须引用 ``model.php`` 文件。
 如果每个 |controller| 文件突然需要引用另一个文件或者执行某些全局任务 （例如：加强安全），又该如何呢？
目前来看，需要添加到每个 |controller| 文件。 如果您忘了给一个文件添加的情形，希望不是安全相关的......

|front Controller| 来救助
----------------------

解决方案就是使用 :term:`front controller`： 一个简单的 PHP 文件接收 *所有* |request| 并加以处理。
有了 |front controller| ， |application| 的 URI 会有一点小变化，但这却添加了灵活的特性：

.. code-block:: text

    Without a front controller
    /index.php          => Blog post list page (index.php executed)
    /show.php           => Blog post show page (show.php executed)

    With index.php as the front controller
    /index.php          => Blog post list page (index.php executed)
    /index.php/show     => Blog post show page (index.php executed)

.. tip::
    The ``index.php`` portion of the URI can be removed if using Apache
    rewrite rules (or equivalent). In that case, the resulting URI of the
    blog show page would be simply ``/show``.

当使用了 |controller| ，一个假单文件（这里的 ``index.php`` ）接收 *每一个* |request| 。
对于博客的 show 页面，``/index.php/show`` 实际将会执行的是 ``index.php`` 文件，
它现在的工作是基于 URI 进行内部 |routing| 。 
您将看到， |front controller| 是一个非常强大的工具。

创建 |front controller|
~~~~~~~~~~~~~~~~~~~~~

您将伴随这个 |application| 迈出重要的一步。 用一个文件处理所有的 |request| ，
您便可以做一种控制的事情，例如处理安全问题，配置调用和 |routing| 。
在这个 |application| 中， ``index.php`` 必须足够聪明，
以便基于 |request| 的 URI 呈现博客帖子 list 页面*或者*博客帖子显示页面：

.. code-block:: html+php

    <?php
    // index.php

    // load and initialize any global libraries
    require_once 'model.php';
    require_once 'controllers.php';

    // route the request internally
    $uri = $_SERVER['REQUEST_URI'];
    if ($uri == '/index.php') {
        list_action();
    } elseif ($uri == '/index.php/show' && isset($_GET['id'])) {
        show_action($_GET['id']);
    } else {
        header('Status: 404 Not Found');
        echo '<html><body><h1>Page Not Found</h1></body></html>';
    }

出于条例化目的， 将两个 |controller| （前面的 ``index.php`` 和 ``show.php`` ） 作为 PHP 函数移动到另一个文件， ``controllers.php`` ：

.. code-block:: php

    function list_action()
    {
        $posts = get_all_posts();
        require 'templates/list.php';
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        require 'templates/show.php';
    }

作为 |front controller| ， ``index.php`` 的角色也起了巨大的变化：
调用核心库并且 |routing| 该 |application| 来调用两个 |controller| （``list_action()`` 和 ``show_action()`` 函数）
事实上， |front controller| 已经在外观和工作方面很像 Symfony2 处理和 |routing| |request| 机制。

.. tip::

   |front controller| 的另一个优势是赋予 URL 灵活性。
   博客帖子的 show 页面可以在一个地方做更改，就由 ``/show`` 改为 ``/read``，而不需要改文件名。
   其实，在 Symfony2 中 URL 的灵活性不止于此。

到此为止， |application| 已经从一个单独的 PHP 文件进化为有条理性并允许代码重用的结构。
您应该很高兴吧！但是还有更令你振奋的。 例如： “ |routing| ”系统已与更改，
list 页面 （ ``/index.php`` ）可以通过认不出的  ``/`` 来访问（如果 Apache |rewrite| 功能已启用）。
但是，很多时间并不是花在开发博客之上，而是用在代码的 “构架” （例如： |routing| 、 调用 |controller| 以及 |template| 等等）。
更多的时间花在处理标单提交、输入验证、登陆和安全。
为什么不为这些惯常的问题创造出解决方案的？

接触 Symfony2
~~~~~~~~~~~

Symfony2 就是这个救星。在使用 Symfony2 之前，您需要确保 PHP 能够找到 Symfony2 |class| 。
这是通过 Symfony 提供的 autoloader 实现的。 autoloader 是一个在不需要明确引用包含 |class| 的文件的情形下使用 PHP |class| 的工具

首先， _`下载 symfony` 并将其存放在 ``vendor/symfony/symfony/`` 目录下。
接下来，创建 ``app/bootstrap.php`` 文件。 用它来 ``require[引用]`` |application| 的两个文件并被指 autoloader：

.. code-block:: html+php

    <?php
    // bootstrap.php
    require_once 'model.php';
    require_once 'controllers.php';
    require_once 'vendor/symfony/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $loader = new Symfony\Component\ClassLoader\UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony' => __DIR__.'/../vendor/symfony/symfony/src',
    ));

    $loader->register();

这是告诉 autoloader ``Symfony`` |class| 在哪里。 
这样一来，您就可以在无需使用 ``require`` 语句调用文件的情况下就能使用 Symfony |class| 。

Symfony 理念的核心是 |application| 的主要工作是解读每个 |request| 并返回一个 |response| 。
为了这一点， Symfony2 提供了  :class:`Symfony\\Component\\HttpFoundation\\Request` 和
:class:`Symfony\\Component\\HttpFoundation\\Response` |class| 。
它们是以 |object-oriented| 呈现原始的 HTTP |request| 并返回 HTTP |response| 。
可以用它们来改进博客：

.. code-block:: html+php

    <?php
    // index.php
    require_once 'app/bootstrap.php';

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $request = Request::createFromGlobals();

    $uri = $request->getPathInfo();
    if ($uri == '/') {
        $response = list_action();
    } elseif ($uri == '/show' && $request->query->has('id')) {
        $response = show_action($request->query->get('id'));
    } else {
        $html = '<html><body><h1>Page Not Found</h1></body></html>';
        $response = new Response($html, 404);
    }

    // echo the headers and send the response
    $response->send();

|controller| 现在负责返回一个 ``Response`` |object| 。
为了另工作供轻松，您可以添加一个 ``render_template()`` 函数，它很类似于 Symfony2 的 |template| 引擎：

.. code-block:: php

    // controllers.php
    use Symfony\Component\HttpFoundation\Response;

    function list_action()
    {
        $posts = get_all_posts();
        $html = render_template('templates/list.php', array('posts' => $posts));

        return new Response($html);
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        $html = render_template('templates/show.php', array('post' => $post));

        return new Response($html);
    }

    // helper function to render templates
    function render_template($path, array $args)
    {
        extract($args);
        ob_start();
        require $path;
        $html = ob_get_clean();

        return $html;
    }

仅仅接触了 Symfony2 一点点， |application| 就更加灵活和可靠。
``Request`` 提供了访问 HTTP |request| 的可靠途径。
具体讲，``getPathInfo()`` 方法返回了一个整洁的 URI （总是返回  ``/show`` ，永远不返回 ``/index.php/show``）
这样，就算用户访问  ``/index.php/show`` ， |application| 会机智地 |request| |route| 到  ``show_action()`` 。

``Response`` 对象在构建 HTTP |response| 时带来灵活性：允许通过 |object-oriented| 界面来添加 HTTP |header| 和 内容。
|application| 的 |response| 简单时或许不明显，但随着 |application| 的扩展，灵活性的益处便不断明显。

Symfony2 中的范例 |application|
~~~~~~~~~~~~~~~~~~~~~~~~~~~

这个博客走过了一条 *漫长* 的路， 这样一个简单的应用仍然包含了很多代码。
这一路走来，我们创造了一个简单的 |routing| 系统以及使用 ``ob_start()`` 和 ``ob_get_clean()`` 来实现 |template| 呈现。
如果您需要继续构建这个“ |framework| ”，您至少还需要用到已经解决了很多问题的 Symfony 的独立 |component| ： `Routing`_ 和 `Templating`_ 。
与其重新解决惯常的问题，倒不如将它们交给 Symfony2。
这是用 Symfony2 建立的一个范例 |application| ：

.. code-block:: html+php

    <?php
    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function listAction()
        {
            $posts = $this->get('doctrine')->getEntityManager()
                ->createQuery('SELECT p FROM AcmeBlogBundle:Post p')
                ->execute();

            return $this->render('AcmeBlogBundle:Blog:list.html.php', array('posts' => $posts));
        }

        public function showAction($id)
        {
            $post = $this->get('doctrine')
                ->getEntityManager()
                ->getRepository('AcmeBlogBundle:Post')
                ->find($id);

            if (!$post) {
                // cause the 404 page not found to be displayed
                throw $this->createNotFoundException();
            }

            return $this->render('AcmeBlogBundle:Blog:show.html.php', array('post' => $post));
        }
    }

两个 |controller| 都是轻量级的。 都使用 Doctrine ORM 库从数据库调用 |object| ，
使用  ``Templating`` |component| 呈现模板并返回一个 ``Response`` |object| 。
list |template| 现在变得更加简单：

.. code-block:: html+php

    <!-- src/Acme/BlogBundle/Resources/views/Blog/list.html.php -->
    <?php $view->extend('::layout.html.php') ?>

    <?php $view['slots']->set('title', 'List of Posts') ?>

    <h1>List of Posts</h1>
    <ul>
        <?php foreach ($posts as $post): ?>
        <li>
            <a href="<?php echo $view['router']->generate('blog_show', array('id' => $post->getId())) ?>">
                <?php echo $post->getTitle() ?>
            </a>
        </li>
        <?php endforeach; ?>
    </ul>

|layout| 几乎是唯一的：

.. code-block:: html+php

    <!-- app/Resources/views/layout.html.php -->
    <html>
        <head>
            <title><?php echo $view['slots']->output('title', 'Default title') ?></title>
        </head>
        <body>
            <?php echo $view['slots']->output('_content') ?>
        </body>
    </html>

.. note::

    我们将把 show |template| 留下来做练习题，应该可以根据 list |template| 如法炮制。

当 Symfony2 引擎 （被称为 |``Kernel``| [ ``Kernel`` ]）被启动，需要一个清楚基于 |request| 信息确定需要使用的 |controller| 的地图。
需要一个以一个可读的格式提供这一信息的 |routing| 配置地图：

.. code-block:: yaml

    # app/config/routing.yml
    blog_list:
        pattern:  /blog
        defaults: { _controller: AcmeBlogBundle:Blog:list }

    blog_show:
        pattern:  /blog/show/{id}
        defaults: { _controller: AcmeBlogBundle:Blog:show }

现在 Symfony2 可以调用所有这些繁琐的任务。 |front controller| 实在太简单了，工作也是如此至少，
以至于它一旦被创建，您永远没有必要再去理会它。（如果您使用 Symfony2 发布版本，您根本都不需要连创建它！）：

.. code-block:: html+php

    <?php
    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(Request::createFromGlobals())->send();

|front controller| 的唯一工作就是初始化 Symfony2 引擎（``Kernel``）并将 ``Request`` |object| 传递进取加以处理。
之后 Symfony2 核心就会使用 |routing| 地图确定调用那一个 |controller| 。
与之前相同， |controller| |method| 负责发挥最终的  ``Response`` 对象。
真的没有什么其他的了。

形象地了解 Symfony2 是如何处理每个 |request| ，请参见
:ref:`request flow diagram<request-flow-figure>` 。

Symfony2 带来了什么
~~~~~~~~~~~~~~

在接下来的章节中，您将更多地学习 Symfony 的每个部分是如何工作的，以及推荐的 |project| 组织结构。
现在来看一看，将博客从直接书写的 PHP 转化为 Symfony2 是如何改善开发体验的：

* 您的应用现在变得 **清晰且代码被统一管理** （尽管 Symfony 不强迫您这样做）。
  这提高了 **重用性** 并且允许新的开发者更快速高效地进入您的 |project| 。

* 您写的代码 100% 是为了 *您的* |application| 。 您 **不需要开发或者维护低端应用**
  例如： :ref:`autoloading<autoloading-introduction-sidebar>` ，
  :doc:`routing</book/routing>` 以及 :doc:`controllers</book/controller>` 。

* Symfony2 给您提供了对于类似 Doctrine、 |template| 、 安全 、表单、验证以及翻译 |component| （仅列举了很小一部分）之类的 **开放源代码工具访问** 。

* ``Routing`` |component| 令这个 |application| 现在支持 **完全灵活的 URL** 。

* Symfony2 的集中化结构带来了一些强大的工具，例如： **Symfony2 内部 HTTP 缓存** 提供的 **HTTP 缓存** 和更加强大的 `Varnish`_ 。这将在 :doc:`caching</book/http_cache>` 一章中讲解。

可能最有价值的是，通过 Symfony2，您可以访问一个完整的  **Symfony2 社区开发的高质量开放源代码工具** 集！
Symfony2 社区工具及可以在 `KnpBundles.com`_ 找到。

更好的 |template|
--------------

如果您决定选择 Symfony2，一个名为 `Twig`_ 的独立 |template| 引擎另 |template| 更快，而且更易书写和审读。
这意味着，范例 |application| 可以进一步缩减代码！以 list |template| 为例，用 Twig 编写：

.. code-block:: html+jinja

    {# src/Acme/BlogBundle/Resources/views/Blog/list.html.twig #}

    {% extends "::layout.html.twig" %}
    {% block title %}List of Posts{% endblock %}

    {% block body %}
        <h1>List of Posts</h1>
        <ul>
            {% for post in posts %}
            <li>
                <a href="{{ path('blog_show', { 'id': post.id }) }}">
                    {{ post.title }}
                </a>
            </li>
            {% endfor %}
        </ul>
    {% endblock %}

同理， ``layout.html.twig`` |template| 也更容易写：

.. code-block:: html+jinja

    {# app/Resources/views/layout.html.twig #}

    <html>
        <head>
            <title>{% block title %}Default title{% endblock %}</title>
        </head>
        <body>
            {% block body %}{% endblock %}
        </body>
    </html>

Symfony2 很好地支持 Twig，也支持 PHP |template| 。我们将继续讨论 Twig 的优势。
更多信息，请查阅 :doc:`templating chapter</book/templating>` 。

从 Cookbook 进一步学习
----------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/service`

.. _`Doctrine`: http://www.doctrine-project.org
.. _`download symfony`: http://symfony.com/download
.. _`Routing`: https://github.com/symfony/Routing
.. _`Templating`: https://github.com/symfony/Templating
.. _`KnpBundles.com`: http://knpbundles.com/
.. _`Twig`: http://twig.sensiolabs.org
.. _`Varnish`: http://www.varnish-cache.org
.. _`PHPUnit`: http://www.phpunit.de

.. include:: ../_terminology.rst