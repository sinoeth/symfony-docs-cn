如何在 git 中创建和存储 Symfony2 项目
==========================

.. 技巧::

             该文档特别介绍 git， 如果你使用 Subversion 存储项目通用原则是相同的。

在您阅读过 :doc:`/book/page_creation` 并且熟悉 Symfony 使用，您一定贮备好开始您自己的项目。  
在本 菜谱 文章中， 你将学会开始新  Symfony2 项目的最佳方法， 也就是使用 `git`_ 代码管理控制系统。

建立出示项目
------

开始之前，您需要下载 Symfony 并初始化本地  git 版本库：、

1. 下载不包含 vendors 的 `Symfony2 Standard Edition`_ 。

2. Unzip/untar 下载版本。 包括你的新项目、配置文件等内容，名为 Symfony 的文件夹被创建。
         根据你的需要将其重命名。

3. 在你的新项目的根目录创建一个名为 ``.gitignore`` 的文件
   （例如： 在 ``composer.json`` 所在目录）并且复制如下内容。 符合下述表述的文件将被 git 忽略：

   .. code-block:: text

        /web/bundles/
        /app/bootstrap*
        /app/cache/*
        /app/logs/*
        /vendor/  
        /app/config/parameters.yml

.. 技巧::

         您可能觉得有必要建立一个使用与全系统的  .gitignore 文件，
         你可以在 `Github .gitignore`_ 找到相关信息
         在这里您可以排除经常被 IDE 用到的所有项目 需要排除的文件和文件夹

4. 复制 ``app/config/parameters.yml`` 为 ``app/config/parameters.yml.dist``.
   ``parameters.yml`` 文件会被 git （如上文规定） 忽略，这样数据库密码之类的信息就不会被提交 （commit）
       通过建立 ``parameters.yml.dist`` 
       文件，新开发者可以快速复制项目，拷贝该文件为
   ``parameters.yml``，加以修改，开始开发工作。

5. 初始化你的 git 版本库：

   .. code-block:: bash

        $ git init

6. 将初始文件添加至 git ：

   .. code-block:: bash

        $ git add .

7. 建立您的信项目的初始化提交：

   .. code-block:: bash

        $ git commit -m "Initial commit"

8. 最后下载第三方代码库 （vendor）。请查阅 :ref:`installation-updating-vendors`。

至此，您有了一个正确提交至 git 的完整功能 Symfony2
你可以立刻开始开发工作，将新近更改提交至 git 版本库。

你可以跟随 :doc:`/book/page_creation` 一章的讲解
学习更多关于如何在您的应用中设置和开发。

.. 技巧::

    Symfony2 标准版（Standard Edition）配有一些功能性范例。 按照 `Standard Edition Readme`_ 的讲解移除这些范例代码。

.. _cookbook-managing-vendor-libraries:

.. include:: _vendor_deps.rst.inc

第三方代码库（Vendors）与子模块（Submodules）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

除了使用 ``composer.json`` 系统来管理第三方代码库，您也可以选择使用原始的 `git submodules`_。 
这样做并没有任何错误， 但是使用 ``composer.json`` 系统是官方解决方案，更加容易一些。
与 git 子模块相比， 使用 ``Composer`` 能够更加智能地分析出代码库之间的依附关系。

在远程服务器上存储您的项目
-------------

您现在有了一个存储于 git 的功能完善的 Symfony2 项目。但是，
很多情形下，出于备份或者与其他开发者进行项目协作的需要，您还想讲您的项目存储到远程服务器上。

在远程服务器存储您的项目的最简单的方式是通过 `GitHub`_。
公共版本库（Public repositories）是免费的，但是您需要为私人版本库（Private repositories)按月支付费用。

除此之外，您可以在任何服务器上建立 `barebones repository`_ 来存储您的 git 版本库。
用于此管理目的的库是 `Gitolite`_。

.. _`git`: http://git-scm.com/
.. _`Symfony2 Standard Edition`: http://symfony.com/download
.. _`Standard Edition Readme`: https://github.com/symfony/symfony-standard/blob/master/README.md
.. _`git submodules`: http://book.git-scm.com/5_submodules.html
.. _`GitHub`: https://github.com/
.. _`barebones repository`: http://progit.org/book/ch4-4.html
.. _`Gitolite`: https://github.com/sitaramc/gitolite
.. _`Github .gitignore`: http://help.github.com/ignore-files/
