.. index::
   single: Installation

安装和配置 Symfony
=============

本章的目标是帮助您在 Symfony 之上构建一个完好运行的 |application|
幸亏 Symfony 提供 “ |distribution| ”,它是一个您可以下载并立刻开始开发工作的 Symfony “初始” |project| 。

.. tip::

    如果您再找如何最好地创建新项目并通过 |source control| 存储的指南，请查阅 `Using Source Control`_ 。

下载 Symfony2 |distribution|
--------------------------

.. tip::

    首先，检查你已经安装和配置了 |web server| （例如： Apache）和 PHP 5.3.2 或更高版本。
    更多关于 Symfony2 安装需求的信息，请参见 :doc:`requirements reference</reference/requirements>` 。
    更多关于配置您的 |web server| 文档根目录的信息，请参见文档： `Apache`_ | `Nginx`_ 。

Symfony2 “ |distribution| ”包是一个包含 Symfony2 核心库、 |bundle| 选集、一个合理目录结构和一些默认配置，功能齐全的 |application| ，
当你现在了一个 Symfony2 |distribution| ，您下载了一个可以立即用于开发您的 |application| 的 功能型 |application| 骨架。

首先访问 Symfony2 下载页面  `http://symfony.com/download`_ 。
在这个页面上，您将看到 Symfony2 的主力 |distribution| —— |*Symfony Standard Edition*| 。
这里，您需要做出选择：

* 下载 ``.tgz`` 或者 ``.zip`` 压缩文档 —— 两者完全一样， 下载您觉得习惯用的一个；

* 下载包含或者不带 |vendors| 的 |distribution| 。 如果您安装了  `Git`_ ，
  您应该下载“ |without vendors| ”的 Symfony2，因为这样可以更加灵活地加载第三方（vendor）库。

将一个压缩包下载到您的本地 |web server| 根目录并对其解压。 在 UNIX 命令行中，可以使用如下一个命令实现（以您的实际文件名替代 ``###`` ）：

.. code-block:: bash

    # for .tgz file
    tar zxvf Symfony_Standard_Vendors_2.0.###.tgz

    # for a .zip file
    unzip Symfony_Standard_Vendors_2.0.###.zip

完成上面的工作后，您应该有一个 ``Symfony/`` 目录，其目录结构是这样的：

.. code-block:: text

    www/ <- your web root directory
        Symfony/ <- the unpacked archive
            app/
                cache/
                config/
                logs/
            src/
                ...
            vendor/
                ...
            web/
                app.php
                ...

.. _installation-updating-vendors:

更新 |Vendors|
~~~~~~~~~~~~

第 1 步：获取 `Composer`_ （一个优秀的 PHP 打包系统）

.. code-block:: bash

    curl -s http://getcomposer.org/installer | php

确保将 ``composer.phar`` 下载都 ``composer.json`` 所在文件夹（即 Symfony |project| 的默认根目录）。

第 2 步：安装 |vendors|

.. code-block:: bash

    php composer.phar install

该命令下载所有必要的 |vendor| 库 —— 包括 Symfony 自身 —— 到 ``vendor/`` 目录。

.. note::

	如果您没有安装 ``curl`` ，可以从  http://getcomposer.org/installer 下载 ``installer`` ，
   将其存放于 |project| 目录并运行：

	.. code-block:: bash

		php installer
		php composer.phar install

配置与设置
~~~~~

到此为止，所有的第三方库都存放在 ``vendor/`` 目录。
您还有一个默认的 |application| 设置 ``app/`` 以及 ``src/`` 目录下的一些范例代码。

Symfony2 带有一个可视的服务器配置测试，用来帮助确保 |web server| 和 PHP 按照 Symfony 要求被设置。
访问下面的 URL 测试您的配置：

.. code-block:: text

    http://localhost/Symfony/web/config.php

如果出现问题，逐一更正它们后继续。

.. sidebar:: 设置 |permission|

    一个常见问题是 ``app/cache`` 和 ``app/logs`` 目录对于 |web server| 和 命令行用户必须是可写的。
    在 UNIX 系统中，如果您有不同的 |web server| 用户和命令行用户，可以在 |project| 中运行下面的命令一次确保 |permission| 被正确设置。 更改 |web server| 用户的  ``www-data`` ：

    **1. 在允许 chmod +a 的系统上使用 ACL**

    很多系统允许使用 ``chmod +a`` 命令。先这样试一下，如果得到错误信息尝试下一种方法：
    and if you get an error - try the next method:

    .. code-block:: bash

        rm -rf app/cache/*
        rm -rf app/logs/*

        sudo chmod +a "www-data allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
        sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs

    **2. 在不允许 chmod +a 的系统上使用 ACL**

    有些系统不支持 ``chmod +a`` ，但是支持另一个名为  ``setfacl`` 的工具。您可需要在使用前，在分区上  `启用 ACL 支持`_ 并安装 （Ubuntu就是这样），如下所示：

    .. code-block:: bash

        sudo setfacl -R -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs
        sudo setfacl -dR -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs

    **3. 不使用 ACL**

    如果不能访问目录的 ACL ，可以更改 umask 使得 cache 和 log 两个目录称为组可写或者全局可写（取决于 |web server| 用户和命令行用户是否在一个组）
    通过将下列放在 ``app/console``、    ``web/app.php`` 和 ``web/app_dev.php`` 文件的开始做到：

    .. code-block:: php

        umask(0002); // This will let the permissions be 0775

        // or

        umask(0000); // This will let the permissions be 0777

    建议在可以访问目录的 ACL 的情形下使用 ACL，因为更改 umask 会带来安全隐患。

一切就绪之后，点击 "Go to the Welcome page" 第一次 |request| “真正的” Symfony2 网页：

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

Symfony2 会欢迎并且为您之前的努力成果表示恭喜！

.. image:: /images/quick_tour/welcome.jpg

开始开发
----

现在您有了一个全功能的 Symfony2 |application|, 可以开始开发了！
|distribution| 中应该包含一些范例代码 —— 查看 |distribution| 中的
``README.rst`` 文件（以文本文件方式打开），查看 |distribution| 中包含哪些范例代码以及如何移除它们。

如果您是 Symfony 新用户，随我们一起进入 ":doc:`page_creation`" ，
学习如何创建页面、更改配置以及其他新 |application| 需要做的工作。

使用 |source control|
-------------------

如果您在使用类似于 ``Git`` 和 ``Subversion`` 的 |version control| 系统，在设置 |version control| ，在设置 |version control| 系统后就可以开始正常提交 |project|
|Symfony Standard Edition| *是* 新 |project| 的起点。

关于如何最好地使用 git 存储项目的指南，请参见 :doc:`/cookbook/workflow/new_project_git`.

忽略 ``vendor/`` 目录
~~~~~~~~~~~~~~~~~

如果下载的是 |*without vendors*| 的压缩包，整个 ``vendor/`` 目录可以安全地被忽略并提交至 |source control| 。
 就 ``Git`` 而言，通过创建 ``.gitignore`` 文件并添加如下内容来实现：

.. code-block:: text

    vendor/

现在， vendor 目录不会被提交至 |source control| 。 
因为其他人 |clone| 或者 |check out| |project| 之后，
可以简单地运行 ``php composer.phar install`` 下载所需的 |vendor| 库。

.. _`启用 ACL 支持`: https://help.ubuntu.com/community/FilePermissionsACLs
.. _`http://symfony.com/download`: http://symfony.com/download
.. _`Git`: http://git-scm.com/
.. _`GitHub Bootcamp`: http://help.github.com/set-up-git-redirect
.. _`Composer`: http://getcomposer.org/
.. _`Apache`: http://httpd.apache.org/docs/current/mod/core.html#documentroot
.. _`Nginx`: http://wiki.nginx.org/HttpCoreModule#root

.. include:: ../_terminology.rst