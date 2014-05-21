用 Symfony2 建立一个博客
===========================

介绍
------------

本教程通过建立一个完整功能的博客网站来引导你使用 `Symfony2 <http://symfony.com/>`_框架。
教程使用的是 Symfony2 的标准版本，它包含了在建立你自己网站时需要的主要组件。本教程分为多个部分
，涵盖了 Symfony2 的各项特征与组件。本教程参考了 symfony 1 的
`Jobeet <http://www.symfony-project.org/jobeet/1_4/Doctrine/en/>`_ 并以类似的方式编写。

教程章节
~~~~~~~~~~~~~~

.. toctree::
    :maxdepth: 1

    docs/configuration-and-templating
    docs/validators-and-forms
    docs/doctrine-2-the-blog-model
    docs/extending-the-model-blog-comments
    docs/customising-the-view-more-with-twig
    docs/testing-unit-and-functional-phpunit

演示网站
------------

本网站 symblog 可以在 `http://symblog.co.uk <http://symblog.co.uk/>`_ 浏览，源代码存放在
`Github <https://github.com/dsyph3r/symblog>`_ ，按照教程的内容划分。

涵盖范围
--------

本教程的目标是介绍你在使用 Symfony2 构建网站常会遇到的任务。

    1.  软件包(Bundle)
    2.  控制器(Controllers)
    3.  模板(Using TWIG)
    4.  模型 - Doctrine 2
    5.  迁移
    6.  数据固件
    7.  验证
    8.  表单
    9.  路由
    10. 资源管理
    11. 发送邮件
    12. 环境
    13. 自定义错误页
    14. 安全
    15. 用户与会话
    16. 生成增删改查骨架
    17. 缓存
    18. 测试
    19. 部署

Symfony2 可以高度定制，并提供许多不同的途径完成同样的工作，像是配置文件格式的选择有 YAML 、 XML 、
PHP 或注释，建立模板可以使用 Twig 或 PHP 。为了保持教程的简洁，我们会使用 YAML 与注释格式，
，模板则是 Twig 。 `Symfony 手册 <http://symfony.com/doc/current/book/index.html>`_ 提供了
丰富的资源与范例，来说明如何使用其他方法。如果有人想要参与完成这个教学或提供其他方式，只需要在
`Github <https://github.com/dsyph3r/symblog-docs>`_ 建立一个衍生版本，然后发出 pull requests。


翻译
------------


西班牙语
~~~~~~~

Symblog 由 `Lisper <https://twitter.com/#!/esymfony>`_ 翻译成 `西班牙语 <http://symblog.site90.net/>`_ 。

法语
~~~~~~~

Symblog 由 `Clement Keirua <https://twitter.com/clemkeirua>`_ 翻译为 `法语 <http://keiruaprod.fr/symblog-fr/>`_ 。


葡萄牙语
~~~~~~~~~
Symblog 由 `TotLab - Pesquisa e Desenvolvimento Web <http://totlab.com.br>`_ 翻译为 `葡萄牙语(巴西) <http://symfony2blog.totlab.com.br/>`_ 。


简体中文
~~~~~~~~~
Symblog 由 `michaeljayt <https://github.com/michaeljayt/symblog-docs-zh-cn/>`_ 翻译为 `简体中文 <https://github.com/michaeljayt/symblog-docs-zh-cn/>`_ 。


作者
------

本教程由 `dsyph3r <http://twitter.com/#!/dsyph3r>`_ 建立。

贡献
------------

本教程的 `源代码 <https://github.com/dsyph3r/symblog-docs>`_ 放在 Github 上，如果你想要改进或者
扩展本教程，只要建立一个衍生版本，然后发出回推请求。你也可以用 `GitHub Issue Tracker <https://github.com/dsyph3r/symblog-docs/issues>`_
发表问题。如果你对于改善本教程的的视觉设计感兴趣，请 `与我联系 <http://twitter.com/#!/dsyph3r>`_!

Credits
-------

Special thanks to all the contributors of the
`Official Symfony2 documentation <http://symfony.com/doc/current/>`_. This
provided an invaluable resource of information.

国旗图示来自 `famfamfam <http://www.famfamfam.com/lab/icons/flags/>`_ 。

查找
---------

在查找一个特定主题吗？使用 :ref:`search` 。
