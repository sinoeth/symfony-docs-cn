.. index::
   single: Page creation

在 Symfony2 中创建页面
================

在 Symfony2 中建立新页面只需要简单地两步：

* |*Create a route*| ： |route| 定义页面对应的 URL （例如：``/about`` ）
  以及接收到的 |request| 与 |route| 匹配时该执行哪个 |controller| （某个 PHP 函数）；

* |*Create a controller*| ： |controller| 就是将收到的 |request| 
  转化为 Symfony2 ``Response`` |object| 的一个 PHP 函数 A controller is a PHP function that takes the incoming

这样的做法是很完美的，因为它与 |web| 的工作模式相吻合。
|web| 的每一个交互都是由 |HTTP request| 发起。
|application| 的工作就是解读 |request| 并返回恰当的 HTTP |response| .

Symfony2 追随这样的理念，并随着用户的增加以及 |application| 的复杂化，保持 |application| 的条理。

听起来很简单？让我们深入其中吧！

.. index::
   single: Page creation; Example

"Hello Symfony!" [Symfony 你好] 页面
--------------------------------

让我们从经典的 “Hello World！” |application| 开始吧！
完成后，用户访问下面的链接，将得到个性化的问候（例如： “Symfony 你好”）：

.. code-block:: text

    http://localhost/app_dev.php/hello/Symfony

事实上，可以用任何其他名字替代 ``Symfony`` 获得相应的问候。
建立这个页面需要依照两个步骤。

.. note::

    这个教程假设您已经下载了 Symfony2并配置了 |web server| 。
    上面的链接假设 ``localhost`` 指向 Symfony2 |project| ``web`` 目录。
    具体信息请参阅 |web server| 文档。这是一些可能用到的 |web server| 的相关文档：
    
    * For Apache HTTP Server, refer to `Apache's DirectoryIndex documentation`_.
    * For Nginx, refer to `Nginx HttpCoreModule location documentation`_.

开始之前： 建立 |bundle|
~~~~~~~~~~~~~~~~~

开始之前，需要先建立 |*bundle*| 。
 在 Symfony2 中， a :term:`bundle` 就像是一个插件，只是其中装载的是 |application| 的全部代码。

|bundle| 仅仅是一个存储与特定功能相关的一切（包括 PHP |class| ，配置，样式以及 Javascript 文件 ）的目录
（请参阅 :ref:`page-creation-bundles`）。

运行下面的命令并根据屏幕上的指导（使用所有默认选项）建立一个名为 ``AcmeHelloBundle`` 的 |bundle| 
（这一章中建立的一个联系 |bundle| ）：

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/HelloBundle --format=yml

在后台， ``src/Acme/HelloBundle`` 目录将为该 |bundle| 建立。
在 ``app/AppKernel.php`` 之中被添加了新的一行，将 |bundle| 注册到了 |kernel| 之中：

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Acme\HelloBundle\AcmeHelloBundle(),
        );
        // ...

        return $bundles;
    }

现在 |bundle| 已经被建立，可以在 |bundle| 内部开始构建 |application| 了。

第 1 步：建立 |route|
~~~~~~~~~~~~~~~~

默认的 Symfony2 |application| |route| 配置位于 ``app/config/routing.yml`` 。
正如 Symfony2 所有配置一样，可以选择使用 XML、PHP 不受限制地配置 |route| 。

如果查看主 |route| 文件，会发现 Symfony 已经在建立 ``AcmeHelloBundle`` |bundle| 时添加了一条记录：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        AcmeHelloBundle:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addCollection(
            $loader->import('@AcmeHelloBundle/Resources/config/routing.php'),
            '/',
        );

        return $collection;

这条记录很简单：告诉 Symfony 从``AcmeHelloBundle`` 之中的 ``Resources/config/routing.yml`` 文件调用 |route| 配置。
这意味着， |route| 配置在 ``app/config/routing.yml`` 文件之中，从这里引用整个 |application| 的 |route| 集中管理。

现在 |bundle| 的 ``routing.yml`` 文件已被导入，可以在这里定义将要建立的每个页面的 |route| ：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

|route| 包含连个基本组成部分： |route| 用以匹配的 |``pattern``| 和一个制定应该执行的 |controller| 的 |``defaults``| 数组。
|pattern| （``{name}``） 的内容是一个通配符，意味着 ``/hello/Ryan`` 、 ``/hello/Fabien`` 之类的 URL 将会与该 |route| 匹配。
``{name}`` 参数也将被一同传递，以便用该值给与用于个性化问候。

.. note::

  |route| 系统还有很多为 |application| 提供灵活强大的 URL结构的优秀功能。
  更进一步的信息，请参阅相关章节 :doc:`Routing </book/routing>` 。

第 2 步：创建 |controller|
~~~~~~~~~~~~~~~~~~~~~

当 |application| 处理一个类似于 ``/hello/Ryan``  的 URL， |route| ``hello`` 被匹配， 
|framework| 将会执行 ``AcmeHelloBundle:Hello:index`` |controller| 。
创建页面的第二步就是创建这个 |controller| 。

|controller| 的 *逻辑* 名是 ``AcmeHelloBundle:Hello:index``，
它将映射到名为 ``Acme\HelloBundle\Controller\Hello`` 的 PHP |class| 之中的 ``indexAction`` |method| 。
在 ``AcmeHelloBundle``中创建这个文件 ::

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
    }

事实上， |controller| 就是一个由 Symfony 执行的 PHP |method| 。 这是代码利用 |request| 信息构建并准备所 |request| 的资源的地方。
除了一些高端应用的情形， |controller| 的最终产品都是一样的： 一个 Symfony2  ``Response`` |object| 。

创建在 ``hello`` |route| 被匹配后 Symfony 将运行的 ``indexAction`` |method| ::

    // src/Acme/HelloBundle/Controller/HelloController.php

    // ...
    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

|controller| 很简单：创建一个新 ``Response`` |object| ，第一个变量是用于构建 |response| 的内容（这个例子中是一个简单的 HTML 页面）。

恭喜！在仅仅建立了一条 |route| 和一个 |controller| 之后，已经有了一个全功能的页面！如果一切设置正确， |application| 应该问候如下：

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

.. tip::

    You can also view your app in the "prod" :ref:`environment<environments-summary>`
    by visiting:

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    If you get an error, it's likely because you need to clear your cache
    by running:

    .. code-block:: bash

        php app/console cache:clear --env=prod --no-debug

一个通常的，但是可选的第三步就是创建一个 |template| 。

.. note::

   |controller| 是代码的主进入点，也是构建页面时的重要组成部分。更多信息请参阅：
   :doc:`Controller Chapter </book/controller>`.

可选的第 3 步： 创建 |template|
~~~~~~~~~~~~~~~~~~~~~~~

通过 |template| 可以将所有呈现（例如： HTML 代码）存放于独立的文件并重用页面 |layout| 的不同部分。
以 |render| |template| 的方式，避免了在 |controller| 中书写 HTML：

.. code-block:: php
    :linenos:

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

            // render a PHP template instead
            // return $this->render('AcmeHelloBundle:Hello:index.html.php', array('name' => $name));
        }
    }

.. note::

   为了使用  ``render()`` |method| ，  |controller| 必须继承
   ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` |class| （API
   文档： :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller` ），
   |controller| 经常做的任务，这里都有快捷方式。
   通过上面例子中第4行的 ``use`` 语句引用后，在第6行完成 ``Controller`` 的继承。

``render()`` |method| 使用给出的内容， |render| |template| 创建一个 ``Response`` |object|
与其它 |controller| 相同，将返回一个 ``Response`` 对象。

应该注意到，有两种 |render| |template| 的方法。
默认情况， Symfony2 支持两种 |template| 语言： 经典的 PHP |template| 和 简明且强大的 `Twig`_ |template| 。
不要担心，可以自由的在同一个项目中使用任何一种，甚至于混用两种。

|controller| 为下面的命名规则 |render| ``AcmeHelloBundle:Hello:index.html.twig`` |template| ：

    **BundleName**:**ControllerName**:**TemplateName**

这是 |template| 的 *logical* 名，根据命名规则对应着下面的物理地址。

    **/path/to/BundleName**/Resources/views/**ControllerName**/**TemplateName**

在这个例子中， ``AcmeHelloBundle`` 是 |bundle| 名，  ``Hello`` 是 |controller| ， |template| 是 ``index.html.twig`` 。

.. configuration-block::

    .. code-block:: jinja
       :linenos:

        {# src/Acme/HelloBundle/Resources/views/Hello/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            Hello {{ name }}!
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        Hello <?php echo $view->escape($name) ?>!

让我们一行行理解 Twig |template| ：

* *第 2 行*： ``extends`` |token| 定义一个父 |template| ， 是一个明确定义 |layout| 的文件。

* *第 4 行*：  ``block`` |token| 的意思是，其中的一切都应该被放置于名为 ``body`` 的 |block| 中。
  正如看到的，父  |template| （ ``base.html.twig`` ） 将负责最终 |render| 这个名为 ``body`` 的 |block| 。

父 |template| ``::base.html.twig`` 的名字中既没有 **BundleName** ，也没有 **ControllerName**
（因此使用 ``::`` 开头）。这意味着它是一个不位于任何 |bundle| 之中的 |template| ，它位于 ``app`` 目录：

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Welcome!{% endblock %}</title>
                {% block stylesheets %}{% endblock %}
                <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
            </head>
            <body>
                {% block body %}{% endblock %}
                {% block javascripts %}{% endblock %}
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Welcome!') ?></title>
                <?php $view['slots']->output('stylesheets') ?>
                <link rel="shortcut icon" href="<?php echo $view['assets']->getUrl('favicon.ico') ?>" />
            </head>
            <body>
                <?php $view['slots']->output('_content') ?>
                <?php $view['slots']->output('stylesheets') ?>
            </body>
        </html>

基础[base] |template| 文件定义了 HTML |layout| 并 |render| ``index.html.twig`` |template| 中定义的 ``body`` |block| 。
它也同时 |render| 名为 ``title`` 的 |block| ，也可以在 ``index.html.twig`` 中定义该 |block| 。
这里没有在子 |template| 中定义  ``title`` |block| ，所以为默认的 "Welcome!"。

|template| 是 |render| 和组织页面内容的一种强大方式。 
|template| 可以 |render| 包括 HTML标注， CSS代码以及任何其他 Controller 需要返回的内容。

在处理 |request| 的生命周期内， |template| 引擎只是一个可选的工具。 
前文曾提到 |controller| 的目标是返回 ``Response`` |object| ,
|template| 则是为 ``Response`` |object| 创建内容的一个可选的强大工具。

.. index::
   single: Directory Structure

目录结构
----

仅仅几节内容，已经介绍了在 Symfony2 中创建和 |render| 页面的思想。
也初步了解了 Symfony2 |project| 的结构的组织方式。
这一节将介绍不同类型的文件应从什么地方存放和查找，以及这样安置的原因。

在非常灵活的前提下，每个 Symfony :term:`application` 有着相同的基本目录结构，这也是推荐的目录结构：

* ``app/`` ： 该目录存放 |application| 配置；

* ``src/`` ：所有 |project| 的 PHP 代码都存在这个目录之下；

* ``vendor/`` ： 根据惯例，所有 |vendor| 库都存放在这里；

* ``web/`` ： 这是 |web| 的根目录，存放所有允许公开访问的文件；

web 目录
~~~~~~

web 目录是包括图片，样式文件[stylesheet]以及 JavaScript 文件的公开静态文件的存放位置。
也是每个 :term:`front controller` 的所在::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

|front controller| 文件 （ 这个例子中的 ``app.php`` ） 是在使用 Symfony2 |application| 时，被执行的真实 PHP 文件。 
其工作就是调用 |kernel| |class| —— ``AppKernel`` ， 以 |bootstrap| |application| 。

.. tip::

    使用 |front controller| 意味着使用与经典直接书写的 PHP |application| 完全不同，且更加灵活的 URL。 
    在使用 |front controller| 时， URL 格式化为如下的形式：

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    |front controller| 被执行,通过 |route| 配置 |route| “内部的” URL ``/hello/Ryan`` 。
    通过使用 Apache ``mod_rewrite`` 规则，可以不再 URL 中特别指出的时候，强制执行 ``app.php`` 文件：

    .. code-block:: text

        http://localhost/hello/Ryan

尽管 |front controller| 是处理每个 |request| 的必由之路，但却几乎不需要更改，甚至于不需要被考虑。
在 `Environments`_ 章节中，会被再次简要地提及。 

|application| （ ``app`` ）目录
~~~~~~~~~~~~~~~~~~~~~~~~~~~

正如在 |front controller| 之中所见， ``AppKernel`` |class| 是 |application| 的主要介入点，负责所有配置。
因此，它位于 ``app/`` 目录。

这个 |class| 必须通过两个 |method| 定义 Symfony 需要了解该 |application| 的所有一切。
不用担心它们 —— Symfony 已经它们的所有默认内容编写好。

* ``registerBundles()`` ： 返回运行这个 |application| 所需的所有 |bundle| （参见 :ref:`page-creation-bundles` ）；

* ``registerContainerConfiguration()`` ： 调用主 |application| 配置的资源文件（参见：`Application Configuration`_ 一节）。

在日常开发中，主要使用 ``app/`` 目录更改配置， |routing| 文件在 ``app/config/`` 目录下（参见：`Application Configuration`_）。
其中还有一个 |cache| 目录 （ ``app/cache`` ）和一个   |log| 目录 （``app/logs``），
以及一个 |application| 级别资源文件夹，例如：存放 |template| 的（``app/Resources``）。
这些内容都会后面章节中讲解。

.. _autoloading-introduction-sidebar:

.. sidebar:: |autoload|

    在装载 Symfony 之时，特定文件 ``app/autoload.php`` 被引用。
    该文件负责配置自动装载 ``src/`` 目录以及 ``vendor/`` 目录的第三方库的  autoloader [|autoloader|]。

    有了 autoloader ，在不需要使用 ``include`` 和 ``require`` 语句。 
    Symfony2 使用 |class| 的 |namespace| 确定其位置并自动引用相应文件，实例化所需的 |class| 。

    autoloader 已被配置为查看 ``src/`` 目录寻找 PHP |class| 。为确保 |autoload| 工作，需要按照下面的格式提供 |class| 名和文件路径：

    .. code-block:: text

        Class Name:
            Acme\HelloBundle\Controller\HelloController
        Path:
            src/Acme/HelloBundle/Controller/HelloController.php

    通常来讲，只有在引用 ``vendor/`` 之中新的第三方库时，才需要顾及 ``app/autoload.php``文件。
    更多关于 |autoload| 的信息，请参阅   :doc:`How to autoload Classes</components/class_loader>` 。

|source| （ ``src`` ）目录
~~~~~~~~~~~~~~~~~~~~~~

简单来讲， ``src/`` 目录存放所有实际驱动 |application| 的代码（ PHP 代码、模板、配置以及样式文件，等等）
在开发过程中，主要工作都是在这个目录下所创建的一个或者多个 |bundle| 之中进行的。

那么 :term:`bundle` 是什么呢？

.. _page-creation-bundles:

|bundle| 系统
-----------

|bundle| 类似于其他软件中的 |plugin| ，但却更好一些。
 核心的区别是，包括 |framework| 核心功能和为 |application| 编写的代码，Symfony 中的 *一切* 都是 |bundle| 。
|bundle| 是 Symfony2 的头等公民。 这为使用预构的 `third-party bundles`_ 或者分发您自己开发的 |bundle| 提供了灵活性。
可以轻松地为 |application| 选取功能并加以优化满足需求。

.. note::

   这里仅作基础性的介绍，cookbook 则完全为管理和最好地使用 :doc:`bundles</cookbook/bundles/best_practices>` 而编写。

|bundle| 就是在同一个目录中为实现一个 |feature| 的一组由结构的文件。
可以创建 ``BlogBundle`` 、 ``ForumBundle`` 或者一个用于用户管理的 |bundle| （它们很多都已经是开放源代码的 |bundle| ）。
每个目录中的一切，包括  PHP 文件、模板、样式和、JavaScript、测试以及其他内容，都是与其 |feature| 相关的。
|feature| 的一切都在一个 |bundle| 之中，每个 |feature| 都在 |bundle| 之中。

|application| 就是由 ``AppKernel`` |class| 的 ``registerBundles()`` |method| 定义的许多 |bundle| 堆积起来的 ::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

通过 ``registerBundles()`` |method| ，可以完全掌控 |application| 所使用的所有 |bundle| （包括 Symfony 核心 |bundle| ）。

.. tip::

   只要可以被 |autoload| （通过配置于 ``app/autoload.php`` 文件的 autoloader ）， |bundle| 可以被存放在任何位置。

创建 |bundle|
~~~~~~~~~~~

|Symfony Standard Edition| 有一个方便的 |task| 用以创建全功能的 |bundle| 。
当然，纯手工创建 |bundle| 也非常容易。

创建一个名为 ``AcmeTestBundle`` 的 |bundle| ，来看一下 |bundle| 系统是怎样的。

.. tip::

    ``Acme`` 部分只是一个假定的名字，应该被 体现开发者或者开发公司的 “ |vendor| ”名所取代（例如： ``ABC`` 公司的 ``ABCTestBundle``）。

首先创建 ``src/Acme/TestBundle/`` 目录并添加名为 ``AcmeTestBundle.php`` 的新文件::

    // src/Acme/TestBundle/AcmeTestBundle.php
    namespace Acme\TestBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AcmeTestBundle extends Bundle
    {
    }

.. tip::

   名称 ``AcmeTestBundle`` 组照标准的 :ref:` |bundle| 命名规则<bundles-naming-conventions>` 。
   也可以将名称简化为 ``TestBundle`` ，该 |class| 的名称则相应更改为 ``TestBundle`` （该文件则命名为 ``TestBundle.php`` ）。

这个空 |class| 只是新 |bundle| 所需的一个部分。 尽管它通常是空的，但是 |class| 却是个性化 |bundle| 行为的利器。

|bundle| 已经建立，通过 ``AppKernel`` |class| 启用之 ::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...

            // register your bundles
            new Acme\TestBundle\AcmeTestBundle(),
        );
        // ...

        return $bundles;
    }

尽管它还不能做什么，但是 ``AcmeTestBundle`` 已经做好了应用的准备。

同样简单， Symfony 还提供了一个命令行界面用以生成基础 |bundle| 骨架：

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/TestBundle

|bundle| 骨架由一组可以个性化设置的基本 |controller| 、 |template| 以及 |routing| 资源构成。
后面会对 Symfony2 命令行工具进一步介绍。

.. tip::

   在创建一个新 |bundle| 或者开始使用一个第三方 |bundle| 时，总是要确保在 ``registerBundles()`` 中启用之。
   在使用 ``generate:bundle`` 命令时，这个设备自动完成。

|bundle| 目录结构
~~~~~~~~~~~~~

|bundle| 的目录结构简单而且灵活。默认情况， |bundle| 系统遵循保持 Symfony2 |bundle| 一致性的一套规则。
看一下 ``AcmeHelloBundle``，包括下列通用 |bundle| 原色：

* ``Controller/`` 存放 |bundle| 的 |controller| （例如：``HelloController.php``  ）；

* ``Resources/config/`` 存放配置信息，包括 |routing| 配置（例如： ``routing.yml`` ）；

* ``Resources/views/`` 存放以 |controller| 名命名管理的 |template| （例如： ``Hello/index.html.twig`` ）；

* ``Resources/public/`` 存放网站资源（图片、样式等等）并将 通过 ``assets:install`` 命令被复制或链接到 |project| 的 ``web/`` 目录；

* ``Tests/`` 存放所有 |bundle| 测试。

 其中只有实现 |feature| 所需的文件， |bundle| 可能很大，也可能很小。

本书中将陆续介绍，如何为 |application| 与数据库交互、创建和验证表单、创建翻译以及编写测试等等，每一项工作都在 |bundle| 中有自己的存放位置并充当相应的角色。

|application| 配置
----------------

|application| 由用以实现其各种 |feature| 和功能的一系列 |bundle| 组成。
每个 |bundle| 都可以通过 YAML、XML 或者 PHP 格式的配置文件个性化。
默认情况，主配置文件位于 ``app/config/`` 目录，取决于所使用的格式，名为 ``config.yml`` 、 ``config.xml`` 或者 ``config.php``：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            - { resource: parameters.yml }
            - { resource: security.yml }

        framework:
            secret:          %secret%
            charset:         UTF-8
            router:          { resource: "%kernel.root_dir%/config/routing.yml" }
            # ...

        # Twig Configuration
        twig:
            debug:            %kernel.debug%
            strict_variables: %kernel.debug%

        # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="parameters.yml" />
            <import resource="security.yml" />
        </imports>

        <framework:config charset="UTF-8" secret="%secret%">
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework:config>

        <!-- Twig Configuration -->
        <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%" />

        <!-- ... -->

    .. code-block:: php

        $this->import('parameters.yml');
        $this->import('security.yml');

        $container->loadFromExtension('framework', array(
            'secret'          => '%secret%',
            'charset'         => 'UTF-8',
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            // ...
            ),
        ));

        // Twig Configuration
        $container->loadFromExtension('twig', array(
            'debug'            => '%kernel.debug%',
            'strict_variables' => '%kernel.debug%',
        ));

        // ...

.. note::

   You'll learn exactly how to load each file/format in the next section
   `Environments`_.

``framework`` 和 ``twig`` 这些顶级键定义特定 |bundle| 的配置。 ``framework`` 键定义 Symfony 核心 ``FrameworkBundle`` 的配置，
包括对于 |routing| 、 |template| 以及其他核心系统的引用配置。

目前不要担心每个部分的具体配置，默认配置已经设置好。 在后面的学习中随之探索 Symfony2 的各个部分，将学到每个 |feature| 的配置。

.. sidebar:: 设置文件

    在所有章节中，每个配置范例都将以所有三种格式（YAML、XML 和 PHP）给出。 每一种都有其优劣之处，可以自由选择：

    * *YAML*: 简单、干净、可读性强；

    * *XML*: 有时比 YAML 强大而且支持 IDE 自动完成功能；

    * *PHP*: 非常强大，但与标准配置文件相比，可读性差。

默认配置 |dump|
~~~~~~~~~~~

.. versionadded:: 2.1
    The ``config:dump-reference`` command was added in Symfony 2.1

可以通过 ``config:dump-reference`` 命令提取一个 |bundle| 的默认配置。这是一个 |dump| FrameworkBundle 默认配置的例子：

.. code-block:: text

    app/console config:dump-reference FrameworkBundle

也可以使用扩展别名（配置键名）：

.. code-block:: text

    app/console config:dump-reference framework

.. note::

    参阅 cookbook 文章： :doc:`How to expose a Semantic Configuration for
    a Bundle</cookbook/bundles/extension>` 学习为 |bundle| 增加配置。

.. index::
   single: Environments; Introduction

.. _environments-summary:

环境
--

一个 |application| 可以在不同的环境中运行。 不同的环境共享相同的 PHP 代码（除了 |front controller| ）但使用不同的配置。
例如：  ``dev`` 环境将 |log| 所有的警告和错误， 环境只 |log| 错误。 
在 ``prod`` 环境中将为每次 |request| 重构某些文件（为了开发的便利），
但却在 ``prod`` 环境中 |cache| 它们。 所有环境都在一台机器上执行相同的 |application| 。

Symfony2 |project| 通常由三个环境开始（ ``dev`` 、 ``test`` 和 ``prod`` ） ，但是建立新环境是非常简单的。
仅通过访问不同的 |front controller| 就可以在不同环境中查看 |application| 。
通过开发 |front controller| 访问 |application| ，就可以在  ``dev`` 环境访问 |application|:

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

如果希望查成品环境下的 |application| ，可以访问名为 ``prod`` 的 |front controller| ：

.. code-block:: text

    http://localhost/app.php/hello/Ryan

因为 ``prod`` 环境是速度优化的： 配置、 |routing| 和 Twig |template| 都被编译为 PHP |class| 并被缓存。
为了在 ``prod`` 环境中查看变化，需要清楚缓存的文件，以便重新构建它们::

    php app/console cache:clear --env=prod --no-debug

.. note::

   打开 ``web/app.php`` 文件，将看到使用 ``prod`` 环境的清晰配置::

       $kernel = new AppKernel('prod', false);

   可以创建一个新的 |front controller| ，拷贝这个文件后将  ``prod`` 改为其他值。

.. note::

    ``test`` 环境用于运行自动测试，不能从 browser 访问它。更详细的信息请查阅 :doc:`testing chapter</book/testing>` 。

.. index::
   single: Environments; Configuration

环境配置
~~~~

``AppKernel`` |class| 负责夹在所选的配置文件::

    // app/AppKernel.php
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
    }

如果希望使用 XML 或者 PHP 编写配置，正如之前已经介绍过的，可以 ``.yml`` 扩展名更改为 ``.xml`` 或者 ``.php`` 。
每个环境家在其自身的配置文件。就 ``dev`` 环境而言，配置文件如下。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        framework:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        # ...

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <framework:config>
            <framework:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- ... -->

    .. code-block:: php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('framework', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        // ...

``imports`` 键类似于 PHP 的 ``include`` 语句，要确保主配置文件（ ``config.yml`` ）最先被调用。
该文件的其余部分在默认配置基础上为开发环境增加 |log| 和其他一些设置。

``prod`` 和 ``test`` 环境有着相似的模式：它们都倒入基础配置文件，然后为特定环境的需要更改设置的值。
这只是惯例而已，但却做到重用大部分配置并通过很少的修改就为不同环境作了个性化设置。

总结
--

恭喜！ 已经介绍了 Symfony2 的所有基础内容，并且应该已经意识到其便捷和灵活性。
很多 |feature| 将不断涌现，但要记住如下基本点：

* 创建一个页面需要三个步骤： |**route**| 、 |**controller**| 和可选的 |**template**| 。

* 每个 |project| 都有几个主要的目录： ``web/`` （网站资源和所有 |front controller| ）、 
  ``app/`` （配置）、 ``src/`` （被开发的 |bundle| ）以及 ``vendor/`` （第三方代码）（还有一个 ``bin/`` 目录用以帮助更新 |vendor| 库）；

* Symfony2 的每个 |feature| （包括 Symfony2 |framework| 的核心）都以 |*bundle*| 形式出现，这是一个实现 |feature| 的文件结构组；

* 每个 |bundle| 的 **configuration** 都存储于 ``app/config`` 目录并且可以使用 YAML、 XML 或者 PHP 格式；

* 每个 **环境** 可以都通过不同的 |front controller| （例如： ``app.php`` 和 ``app_dev.php``）调用不同的配置文件来访问。

此后的各章将介绍更加强大的工具和超前的概念。 随着更多地了解 Symfony2，将会为其构架的灵活性和提高开发 |application| 的速度的能力愈发折服。

.. _`Twig`: http://twig.sensiolabs.org
.. _`third-party bundles`: http://symfony2bundles.org/
.. _`Symfony Standard Edition`: http://symfony.com/download
.. _`Apache's DirectoryIndex documentation`: http://httpd.apache.org/docs/2.0/mod/mod_dir.html
.. _`Nginx HttpCoreModule location documentation`: http://wiki.nginx.org/HttpCoreModule#location

.. include:: ../_terminology.rst