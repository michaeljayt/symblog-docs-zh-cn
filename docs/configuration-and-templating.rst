[Part 1] - Symfony2 配置与模板
================================================

总览
--------

这一节将会介绍建立 Symfony2 网站的几个步骤，这里下载和使用的是 Symfony2
`标准版 <http://symfony.com/doc/current/glossary.html#term-distribution>`_,
我们将使用这个版本建立博客软件包(Bundle)以及存放主模板。在本节结束后，你将制作出一个配置好的 Symfony2 网站，
能够通过类似 ``http://symblog.dev/`` 的开发用域名访问。这个网站会包含博客的主要网页结构及部分测试信息。

本节将会展示如下主题：

    1. 安装一个 Symfony2 应用
    2. 配置一个开发用域名
    3. Symfony2 软件包
    4. 路由
    5. 控制器
    6. 使用 Twig 制作模板

下载与安装
------------------

如同上面提到的那样，我们会使用 Symfony2 标准版。这个版本包含了完整的核心函数库与建立大多数网站所需要的
软件包。你可以从官方网站 `下载 <http://symfony.com/download>`_ 。本教程不希望重复官方网站上的
在线手册说明，所以请先参考官方网站的 `安装与配置 Symfony2 <http://symfony.com/doc/current/book/installation.html>`_
一节了解如何配置环境。在线手册会引导你下载选择软件包、安装必要外部软件包与设置文件夹的权限等。

.. warning::

    需要特别注意的是安装说明的 `配置权限 <http://symfony.com/doc/current/book/installation.html#configuration-and-setup>`_
    部分，此处会说明一些设定 ``app/cache`` 与 ``app/logs`` 权限的方法，让 web 服务器用户与命令行用户都能有写入的权限。
    

建立一个开发用域名
-----------------------------

本教程中会使用开发用域名 ``http://symblog.dev/`` ，不过你可以选择自己想要使用的域名。
下列步骤主要针对 `Apache <http://httpd.apache.org/>`_ 服务器说明，同时假设你已经在自己主机安装好并且正在运行
Apache。如果你已经知道如何设定开发用域名，或是使用了类似 `nginx <http://nginx.net/>`_ 的网页服务器，你也可以跳过此部分。

.. note::

    这些步骤是在特定的 Linux 发行版 Fedora 中完成的，因此路径名称等信息可能会跟你的操作系统不一样。

首先，要开始建立一个 Apache 的虚拟主机，要找到 Apache 的配置文件，在文件的最后添加以下配置。请依据自己的环境调整
``DocumentRoot`` 与 ``Directory`` 的路径，Apache 配置文件的位置与名称会因操作系统的不同而有所差异，
 Fedora 系统放在 ``/etc/httpd/conf/httpd.conf`` ，编辑此文件需要通过 ``sudo`` 提升权限才能进行。

.. code-block:: text

    # /etc/httpd/conf/httpd.conf

    NameVirtualHost 127.0.0.1

    <VirtualHost 127.0.0.1>
      ServerName symblog.dev
      DocumentRoot "/var/www/html/symblog.dev/web"
      DirectoryIndex app.php
      <Directory "/var/www/html/symblog.dev/web">
        AllowOverride All
        Allow from All
      </Directory>
    </VirtualHost>

接著在 ``/etc/hosts`` 文件的最下面新增一条域名规则，编辑此文件同样需要通过 ``sudo`` 提升权限才能进行。

.. code-block:: text

    # /etc/hosts
    127.0.0.1     symblog.dev

最后，请不要忘记重启 Apache 服务，才会重新加载刚才改动的配置。

.. code-block:: bash

    $ sudo service httpd restart

.. tip::

    如果发现自己经常建立虚拟主机，还可以参考 `Dynamic virtual hosts <http://blog.dsyph3r.com/2010/11/apache-dynamic-virtual-hosts.html>`_
    的有关说明简化这些步骤。

你现在应该可以访问到 ``http://symblog.dev/app_dev.php/`` 了。

.. image:: /_static/images/part_1/welcome.jpg
    :align: center
    :alt: Symfony2 welcome page

如果这是你第一次看到 Symfony2 的欢迎页面，也可以花些时间看看每个示例，这些页面会提供相关代码来展示
网页背后的运作原理。

.. note::

    你同时也会发现在欢迎页下方有一个工具栏。这是一系列开发者工具，可以向你提供许多有关应用程序状态的重要信息。
    通过此工具栏可以看到包括页面执行时间、内存使用量、数据库查询、认证状态等等信息。这个工具栏默认
    只会在运行 ``dev`` 环境时显示，而在生产环境中，提供这个工具栏会造成很大的安全风险，因为它揭露了应用程
    序的许多细节。工具栏的参考信息会在介绍新功能时提到。

配置 Symfony: Web 界面
----------------------------------


Symfony2 添加了一个网页界面来进行网站的多项设定，例如数据库。我们这个项目会需要一个数据库，所以现在就要开始使用这个配置工具。

浏览 ``http://symblog.dev/app_dev.php/`` 并且点击 Configure 按钮，进入数据库详细配置（本教程假设使用 MySQL
数据库，不过你可以选择任何可用的数据库），接下来在下一页会产生一个 CSRF token，你会看到 Symfony2 生成的
各种参数。注意此页面上的说明，如果你的 ``app/config/parameters.ini`` 文件无法直接写入，就需要将页面显示的配置信息复制到 ``app/config/parameters.ini`` 文件（直接覆盖文件中的原有配置）。


软件包：构成 Symfony2 的基石
----------------------------------

软件包是任何 Symfony2 应用的基石，而事实上 Symfony2 框架本身也是个软件包。软件包让我们可以分离各项功能
提供可复用的代码单位。它封装了整个软件包所需要的东西，用来提供特定的功能。它包含控制器、模型和模板等
代码，以及许多像是图片、CSS 一类的资源。我们会建立一个使用 Blogger 命名空间的软件包来制作网站。如果
你不熟悉 PHP 的命名空间，你应该花些时间去阅读相关文档，因为 Symfony2 中大量使用到了命名空间。可以参考
`Symfony2 autoloader <http://symfony.com/doc/current/cookbook/tools/autoloader.html>`_
了解 Symfony2 如何实现自动载入(Autoloading)功能。

.. tip::

    切实了解命名空间有助于排除一些常见问题，比如文件夹结构与命名空间结构不一致等问题。


建立软件包
~~~~~~~~~~~~~~~~~~~

为了封装博客的功能，我们要建立一个 Blog 软件包。它将包含所有需要的文件，这样一来可以轻易的将这个软件包移植到另一个
Symfony2 应用程序中。Symfony2 提供了许多工具来协助我们执行常规任务，其中一个就是软件包生成器。

要执行软件包生成器请执行下列命令，屏幕上会出现一些提示，让你设置软件包的配置信息。我们这里就使用各项提示的默认值。

.. code-block:: bash

    $ php app/console generate:bundle --namespace=Blogger/BlogBundle --format=yml

生成器执行后，Symfony2 会为软件包建立基本的文件目录结构，这里需要注意一些重要的改动。

.. tip::

    其实并不一定要使用 Symfony2 提供的生成器工具，它们只是起到了一些辅助作用，你完全可以手动建立软件包的
    文件夹结构与文件。虽然这么说，使用生成器还是有好处的，比如可以快速的执行所有必要工作——生成软件包并执行相应命令。这其中一个例子就是注册软件包。

注册软件包
......................


我们的新软件包 ``BloggerBlogBundle`` 已经注册在核心文件 ``app/AppKernel.php`` 中，Symfony2 要求我们明确注册所有要使用的软件包，你也会注意到一些软件包只在 ``dev`` 或 ``test`` 环境下注册，在正式环境 ``prod`` 中载入这些软
件包会加载一些用不到的功能，而徒增系统负担。下面这段代码显示如何注册 ``BloggerBlogBundle`` 。

.. code-block:: php

    // app/AppKernel.php
    class AppKernel extends Kernel
    {
        public function registerBundles()
        {
            $bundles = array(
            // ..
                new Blogger\BlogBundle\BloggerBlogBundle(),
            );
            // ..

            return $bundles;
        }

        // ..
    }

路由
.......

软件包的路由配置已经导入到应用程序的主路由文件 ``app/config/routing.yml`` 中。

.. code-block:: yaml

    # app/config/routing.yml
    BloggerBlogBundle:
        resource: "@BloggerBlogBundle/Resources/config/routing.yml"
        prefix:   /

prefix 选项可以让我们将整个 ``BloggerBlogBundle`` 路由在挂载时带上前缀，例子中已经选择挂载在预设的 ``/`` 上。
如果你想要所有的路由以 ``/blogger`` 开头，可以将 prefix 改为 ``prefix: /blogger`` 。

默认结构
.................

 ``src`` 文件夹中已经建立了默认的软件包结构，一开始是最上层的 ``Blogger`` 文件夹，它直接对应到我们为软件包设置的
 ``Blogger`` 命名空间，其中还可以看到包含实际软件包的 ``BlogBundle`` 文件夹，这里面内容结构中有一部分名称自身就解释了自己的用途。

默认控制器
~~~~~~~~~~~~~~~~~~~~~~

在软件包生成器生成的文件中， Symfony2 建立了一个默认的控制器，我们可以通过浏览
 ``http://symblog.dev/app_dev.php/hello/symblog`` 来执行它。你可以在浏览器中看到一个简单的欢迎页面。请试着将网址 ``symblog`` 部分改为你想要的名称，我们就可以在比较高的层次上检查页面的生成情况。

路由
......

 ``BloggerBlogBundle`` 的路由文件放在 ``src/Blogger/BlogBundle/Resources/config/routing.yml`` ，包含了下面的
默认路由规则。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /hello/{name}
        defaults: { _controller: BloggerBlogBundle:Default:index }

路由包含一项匹配模式与一组默认选项，模式用来匹配对应的网址，默认选项则指定在网址匹配时应该要执行的控制器。在模式
 ``/hello/{name}`` 中， ``{name}`` 占位符没有配置特殊条件，所以对应任意字符。路由也没有指定任何国别(culture)、格式或 HTTP 方法。而没有指定 HTTP 方法，就表示来自 GET 、 POST 、 PUT 等方式的请求都会匹配到该模式。

如果网址符合所有指定的条件，就会执行配置选项 _controller 中的控制器， _controller 选项指定了控制器的逻辑名，让
Symfony2 可以对应到指定的文件。上面的例子中会执行 ``Default`` 控制器中的 ``index`` 操作，文件位于
 ``src/Blogger/BlogBundle/Controller/DefaultController.php`` 。

控制器
..............

本例中的控制器非常简单， ``DefaultController`` 继承了 ``Controller`` 类，提供了一些有用的方法，例如下面用到的
``render`` 方法。由于我们的路由定义了一个占位符 ``$name`` ，它会被送到相应的方法中作为参数。 ``index`` 方法会调用 ``render``
方法来指定位于 ``BloggerBlogBundle`` 默认模板文件夹中的 ``index.html.twig`` 模板显示内容。模板名称的格式是
``bundle:controller:template`` ，在我们的例子中， ``BloggerBlogBundle:Default:index.html.twig`` 会对应到 ``BloggerBlogBundle``
 ``Default`` 模板文件夹的 ``index.html.twig`` 模板文件，这个文件实际路径为
``src/Blogger/BlogBundle/Resources/views/Default/index.html.twig`` 。应用和软件包可以在渲染模板时指定多种模板格式，本节后面会做相应介绍。

我们也可以通过 ``array`` 选项将变量 ``$name`` 传递到模板。

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/DefaultController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('BloggerBlogBundle:Default:index.html.twig', array('name' => $name));
        }
    }

模板 (视图)
.......................

如你所见，这个模板非常简单，它只会打印出 Hello 以及控制器传送过来的 name 参数。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Default/index.html.twig #}
    Hello {{ name }}!

清理
~~~~~~~~~~~

由于我们不需要生成器生成出的某些文件，可以去清理这些文件。

控制器文件 ``src/Blogger/BlogBundle/Controller/DefaultController.php`` 是可以删除的，模板文件夹
``src/Blogger/BlogBundle/Resources/views/Default/`` 与其中的内容也可以删除，而最后，需要删除定义在
``src/Blogger/BlogBundle/Resources/config/routing.yml`` 的路由信息。

模板
----------

在 Symfony2 中，模板引擎默认有 `Twig <http://www.twig-project.org/>`_ 与 PHP 两中选择，不同的函数库可以有不同的选择，这要感谢 Symfony2 实现了 `依赖注入容器 <http://symfony.com/doc/current/book/service_container.html>`_
基于下面理由，我们选择使用 Twig 模板引擎：

1. Twig 非常快，Twig 模板会被编译成普通 PHP 对象，所以使用 Twig 模板不会造成太大的负担。
2. Twig 非常简洁， Twig 让我们可以通过少量代码执行模板功能， PHP 模板在部分情况下则会相对冗长。
3. Twig 支援模板继承，这是笔者个人喜爱的特色之一。模板能够继承与重写其他模板，让子模板可以修改来自父模板的默认值。
4. Twig 非常安全， Twig 默认启用了输出的检查，甚至还为导入的模板提供了一个沙箱环境。
5. Twig 容易扩展，Twig 带来了许多你期待模板所拥有的常见核心功能，而其他的功能 Twig 也可以轻易地写出扩展。

这只是使用 Twig 的部分好处，更多采用 Twig 的理由，可以参考 `Twig <http://www.twig-project.org/>`_ 官方网站。

布局结构
~~~~~~~~~~~~~~~~

由于 Twig 支援模板继承，我们接下来将使用 `三层继承 <http://symfony.com/doc/current/book/templating.html#three-level-inheritance>`_
方法，让我们可以在应用程序中通过三个独立的层次调整视图，提供更多的定制性。

主模板 - 第 1 层
.......................

现在就开始建立 symblog 的基本区域模板层。这里需要用到两种文件，模板与 CSS 文件。由于 Symfony2 支持 `HTML5 <http://diveintohtml5.org/>`_
，我们也会使用到它。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html" charset="utf-8" />
            <title>{% block title %}symblog{% endblock %} - symblog</title>
            <!--[if lt IE 9]>
                <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
            <![endif]-->
            {% block stylesheets %}
                <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
                <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
                <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
            <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        </head>
        <body>

            <section id="wrapper">
                <header id="header">
                    <div class="top">
                        {% block navigation %}
                            <nav>
                                <ul class="navigation">
                                    <li><a href="#">Home</a></li>
                                    <li><a href="#">About</a></li>
                                    <li><a href="#">Contact</a></li>
                                </ul>
                            </nav>
                        {% endblock %}
                    </div>

                    <hgroup>
                        <h2>{% block blog_title %}<a href="#">symblog</a>{% endblock %}</h2>
                        <h3>{% block blog_tagline %}<a href="#">creating a blog in Symfony2</a>{% endblock %}</h3>
                    </hgroup>
                </header>

                <section class="main-col">
                    {% block body %}{% endblock %}
                </section>
                <aside class="sidebar">
                    {% block sidebar %}{% endblock %}
                </aside>

                <div id="footer">
                    {% block footer %}
                        Symfony2 blog tutorial - created by <a href="https://github.com/dsyph3r">dsyph3r</a>
                    {% endblock %}
                </div>
            </section>

            {% block javascripts %}{% endblock %}
        </body>
    </html>

.. note::

    模板中引用了三个外部文件， 1 个 JavaScript 与 2 个 CSS，这个 JavaScript 脚本修正了 IE9 以前的版本不支持
    HTML5 的问题， 2 个导入的 CSS 文件是来自 `Google Web font <http://www.google.com/webfonts>`_ 的字体。
    
此模板标示了博客网站的主要结构。大部分的模板由 HTML 组成，并包含少量的 Twig 指令，我们接下来将查看这些 Twig 指令。

先将焦点放在文件的 HEAD 标签部分，从 title 标签开始：

.. code-block:: html

    <title>{% block title %}symblog{% endblock %} - symblog</title>

第一个会注意到的是奇怪的 ``{%`` 标签，它即不是 HTML，也不可能是 PHP 。这是 3 个 Twig 标签中的一个，这个标签在 Twig 中表示
 ``Do something``，也就是用来放置执行控制语法或是定义区块元素的指令。完整的
`控制结构 <http://www.twig-project.org/doc/templates.html#list-of-control-structures>`_
可以在 Twig 手册看到。我们在 title 标签定义的 Twig 区域会做两件事情，它不仅将区域命名为 title，而且在 block 与 endblock
之间提供了一个默认输出指令，通过划分各个区域，我们就可以充分利用 Twig 继承模式的好处。举例来说，在一篇博客文章我们想要反映页面标题
，就可以继承该模板并且覆盖 title 区域。

.. code-block:: html

    {% extends '::base.html.twig' %}

    {% block title %}The blog title goes here{% endblock %}

在上面的例子中，我们继承了应用程序的基本模板和前面定义的 title 区域。你会注意到 ``extends`` 使用的模板格式少了
``Bundle`` 与 ``Controller`` 部分，而基本模板格式是 ``bundle:controller:template``。如果省略了 ``Bundle`` 与
 ``Controller`` 部分，就是指定要使用应用程序层级的模板，存放位置为 ``app/Resources/views/`` 。

接下来我们定义另外一个 title 区域，并且放入一些内容，这里是放入博客的标题。由于父模板已经包含了 title 区域，它会被我们
的新模板所覆盖， title 区域现在会输出 'The blog title goes here - symblog' 。Twig 提供的这项功能在建立模板时可以灵活运用。

在样式表区块我们加入了下一个 Twig 标签 ``{{`` ，也被称之为 ``Say something`` 标签。

.. code-block:: html

    <link href="{{ asset('css/screen.css') }}" type="text/css" rel="stylesheet" />

这个标签用来打印出变量或表达式的值。上面的例子中会打印 ``asset`` 方法回传的值，这提供了一个链接应用
程序资源的便捷方法，例如 CSS 、 JavaScript 与图片文件。

``{{`` 标签也可以配合过滤器在输出前处理内容。

.. code-block:: html

    {{ blog.created|date("d-m-Y") }}

完整的过滤器清单可以参考
`Twig 手册 <http://www.twig-project.org/doc/templates.html#list-of-built-in-filters>`_ 。

最后一个 Twig 标签并没有出现在模板中，它是备注标签 ``{#`` ，像这样使用：

.. code-block:: html

    {# The quick brown fox jumps over the lazy dog #}

模板中不会再有其他概念，这个模板已经准备好主要布局，让我们可以在需要时进行定制。

接下来添加一些样式，先建立一个样式表并保存为 ``web/css/screen.css`` 文件，然后添加下面内容，这会在主要模板中加入一些样式。

.. code-block:: css

    html,body,div,span,applet,object,iframe,h1,h2,h3,h4,h5,h6,p,blockquote,pre,a,abbr,acronym,address,big,cite,code,del,dfn,em,img,ins,kbd,q,s,samp,small,strike,strong,sub,sup,tt,var,b,u,i,center,dl,dt,dd,ol,ul,li,fieldset,form,label,legend,table,caption,tbody,tfoot,thead,tr,th,td,article,aside,canvas,details,embed,figure,figcaption,footer,header,hgroup,menu,nav,output,ruby,section,summary,time,mark,audio,video{border:0;font-size:100%;font:inherit;vertical-align:baseline;margin:0;padding:0}article,aside,details,figcaption,figure,footer,header,hgroup,menu,nav,section{display:block}body{line-height:1}ol,ul{list-style:none}blockquote,q{quotes:none}blockquote:before,blockquote:after,q:before,q:after{content:none}table{border-collapse:collapse;border-spacing:0}

    body { line-height: 1;font-family: Arial, Helvetica, sans-serif;font-size: 12px; width: 100%; height: 100%; color: #000; font-size: 14px; }
    .clear { clear: both; }

    #wrapper { margin: 10px auto; width: 1000px; }
    #wrapper a { text-decoration: none; color: #F48A00; }
    #wrapper span.highlight { color: #F48A00; }

    #header { border-bottom: 1px solid #ccc; margin-bottom: 20px; }
    #header .top { border-bottom: 1px solid #ccc; margin-bottom: 10px; }
    #header ul.navigation { list-style: none; text-align: right; }
    #header .navigation li { display: inline }
    #header .navigation li a { display: inline-block; padding: 10px 15px; border-left: 1px solid #ccc; }
    #header h2 { font-family: 'Irish Grover', cursive; font-size: 92px; text-align: center; line-height: 110px; }
    #header h2 a { color: #000; }
    #header h3 { text-align: center; font-family: 'La Belle Aurore', cursive; font-size: 24px; margin-bottom: 20px; font-weight: normal; }

    .main-col { width: 700px; display: inline-block; float: left; border-right: 1px solid #ccc; padding: 20px; margin-bottom: 20px; }
    .sidebar { width: 239px; padding: 10px; display: inline-block; }

    .main-col a { color: #F48A00; }
    .main-col h1,
    .main-col h2
        { line-height: 1.2em; font-size: 32px; margin-bottom: 10px; font-weight: normal; color: #F48A00; }
    .main-col p { line-height: 1.5em; margin-bottom: 20px; }

    #footer { border-top: 1px solid #ccc; clear: both; text-align: center; padding: 10px; color: #aaa; }

软件包模板 - 第 2 层
.........................

我们现在继续建立博客软件包的布局，建立一个文件 ``src/Blogger/BlogBundle/Resources/views/layout.html.twig``
然后放入下面内容。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    {% extends '::base.html.twig' %}

    {% block sidebar %}
        Sidebar content
    {% endblock %}


第一眼看到这个模板时也许会觉得过于简单，但简单正是它的关键之处。
第一、它继承了我们之前建立的应用程序基本模板，其次、
它用一些测试内容覆盖了原本的 sidebar 区块。由于 sidebar 会出现在博客的所有页面，通常这个阶段在这一层上做些定制是正确的做法。
你也许会问为什么不把定制的部分放在之前的应用程序模板，这样一来就可以出现在所有页面。原因很简单，应用程序并不知道
软件包的任何信息，也不应该知道这些信息。软件包应该要自己包含自身的所有功能，产生 sidebar 区块就是自身的功能之一。
至于为什么不将它放在每一页的模板，这也很简单，因为这样一来我们建立一个新页面时就得复制 sidebar 一次。更进一步来说，
这个第 2 层模板提供了相应的灵活度，让我们可以添加所有子模板都会用到的信息并且让它们能够继承。举例来说，我们也许想要调整每一页的页尾，这时就适合在这个层面上调整。

页面模板 - 第 3 层
.......................

最后我们准备好制作控制器的布局，这些布局通常会跟控制器的方法产生关联，例如 show 方法就会对应到
一个博客的模板 show 文件。

我们开始建立首页的控制器与模板，由于这是我们第一个建立的页面，我们需要同时建立对应的控制器。为控制器
建立 ``src/Blogger/BlogBundle/Controller/PageController.php`` 文件，并且写入下面内容：

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Controller/PageController.php

    namespace Blogger\BlogBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class PageController extends Controller
    {
        public function indexAction()
        {
            return $this->render('BloggerBlogBundle:Page:index.html.twig');
        }
    }

接下来建立该操作的模板。如同在控制器对应操作中看到的，我们要建立用来产生首页的模板，并把它放在 ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` 文件中。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        Blog homepage
    {% endblock %}


这里导入了我们最后指定的模板。在这个例子中，
模板 ``BloggerBlogBundle::layout.html.twig``
继承来源的名称省略了 ``Controller`` 部分。当我们排除了 ``Controller`` 部分时，我们就是在指定使用软件包
层上的模板，它被放在 ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` 。

接下来为我们的首页新增一个路由，更新软件包路由配置在 ``src/Blogger/BlogBundle/Resources/config/routing.yml``

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_homepage:
        pattern:  /
        defaults: { _controller: BloggerBlogBundle:Page:index }
        requirements:
            _method:  GET

最后我们需要移除默认的 Symfony2 欢迎页面路由，也就是移除路由文件 ``app/config/routing_dev.yml`` 中 ``dev`` 区域的  ``_welcome`` 路由。

我们现在已经可以查看博客的模板，用你的浏览器打开 ``http://symblog.dev/app_dev.php/`` 。

.. image:: /_static/images/part_1/homepage.jpg
    :align: center
    :alt: symblog main template layout

你应该可以看到博客的基本模板，包含我们在相关模板重写的主要内容与 sidebar 对应的区域。

“关于”页面
--------------

本章的最后一项任务就是建立一个“关于”的静态页面，这将展示如何将页面链接在一起，并进一步强调我们所用的三层继承法。

路由
~~~~~~~~~


建立一个新页面时，首先应该建立对应的路由。打开 ``BloggerBlogBundle`` 的路由文件
``src/Blogger/BlogBundle/Resources/config/routing.yml`` 并且添加下面的路由规则。

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    BloggerBlogBundle_about:
        pattern:  /about
        defaults: { _controller: BloggerBlogBundle:Page:about }
        requirements:
            _method:  GET

控制器
~~~~~~~~~~~~~~

接著开启 ``Page`` 控制器，文件位于 ``src/Blogger/BlogBundle/Controller/PageController.php`` 并且新增处理“关于”
页面的方法。


.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    class PageController extends Controller
    {
        //  ..

        public function aboutAction()
        {
            return $this->render('BloggerBlogBundle:Page:about.html.twig');
        }
    }

视图
~~~~~~~~

要建立视图，在 ``src/Blogger/BlogBundle/Resources/views/Page/about.html.twig`` 建立模板，并且复制下面内容。

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/about.html.twig #}
    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block title %}About{% endblock%}

    {% block body %}
        <header>
            <h1>About symblog</h1>
        </header>
        <article>
            <p>Donec imperdiet ante sed diam consequat et dictum erat faucibus. Aliquam sit
            amet vehicula leo. Morbi urna dui, tempor ac posuere et, rutrum at dui.
            Curabitur neque quam, ultricies ut imperdiet id, ornare varius arcu. Ut congue
            urna sit amet tellus malesuada nec elementum risus molestie. Donec gravida
            tellus sed tortor adipiscing fringilla. Donec nulla mauris, mollis egestas
            condimentum laoreet, lacinia vel lorem. Morbi vitae justo sit amet felis
            vehicula commodo a placerat lacus. Mauris at est elit, nec vehicula urna. Duis a
            lacus nisl. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices
            posuere cubilia Curae.</p>
        </article>
    {% endblock %}

“关于”页面并没有特别之处，唯一的用处只是用测试内容产生模板文件。不过它依然可以带我们继续前进到下一个任务。


链接这些页面
~~~~~~~~~~~~~~~~~


我们现在已经有了“关于”页面，可以直接查看 ``http://symblog.dev/app_dev.php/about`` 上的信息。一般用户看不到这个页面
，除非像我们这样手动输入完整的地址。可以预料到的是 Symfony2 提供了两处对等的路由功能，它可以让我们比对所看到的路径，也可以产生
这些路径所对应的网址。建议一定使用 Symfony2 的路由功能，而不要在程序中使用如下链接：


.. code-block:: html+php

    <a href="/contact">Contact</a>

    <?php $this->redirect("/contact"); ?>

你也许想知道这个方法错在哪里。上面也许是你经常用来链接页面的方式，不过这个方法会有下面问题：

1. 它使用了绝对链接并且完全忽略了 Symfony2 的路由系统，如果你想要修改联系我们页面的位置，你必需要找到所有使用绝对
   链接的位置并且进行修改。
   
2. 它会忽略环境中的控制器，虽然我们还没解释环境是什么，但是你已经在使用了。 ``app_dev.php`` 前端控制器
   让我们可以在 ``dev`` 环境中访问应用程序。如果你把 ``app_dev.php`` 改为 ``app.php`` ，应用程序就会在 ``prod`` 环境
   下执行。环境的重要性会在后面的教程中作更多的说明，不过现在重要的是需要注意，上面定义的实际链接不会依据我们目前的环境调整，因为前端控制器并没有包含在网址中。

链接页面的正确方法是使用 Twig 提供的 ``path`` 与 ``url`` 方法，它们都很类似，只是 ``url`` 方法会给我们完整的网址。我们来
调整主应用程序模板 ``app/Resources/views/base.html.twig`` 为“关于”与首页建立链接。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    {% block navigation %}
        <nav>
            <ul class="navigation">
                <li><a href="{{ path('BloggerBlogBundle_homepage') }}">Home</a></li>
                <li><a href="{{ path('BloggerBlogBundle_about') }}">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </nav>
    {% endblock %}

接下来刷新浏览器，就可以看到 Home 与 About 页面链接就可以运作了，如果你查看页面源代码就会发现，链接的前面都加上了 ``/app_dev.php/`` ，
这就是上面提到的前端控制器，而且会看到 ``path`` 的使用会处理这个部分。

最后让我们更新首页的主视图链接，更新位于 ``app/Resources/views/base.html.twig`` 的模板。

.. code-block:: html

    <!-- app/Resources/views/base.html.twig -->
    <hgroup>
        <h2>{% block blog_title %}<a href="{{ path('BloggerBlogBundle_homepage') }}">symblog</a>{% endblock %}</h2>
        <h3>{% block blog_tagline %}<a href="{{ path('BloggerBlogBundle_homepage') }}">creating a blog in Symfony2</a>{% endblock %}</h3>
    </hgroup>

结论
----------

我们已经涵盖了 Symfony2 应用程序的基础部分，包括配置与执行，并且开始探索 Symfony2 应用程序背后的一些基础概念，如路由与 Twig 模板引擎。

接着，我们会介绍如何建立一个“联系我们”页面，这个页面比“关于”更加深入，它让使用者可以通过网页表单发送请求互动，下一节会介绍字段验证与表单的有关内容。
