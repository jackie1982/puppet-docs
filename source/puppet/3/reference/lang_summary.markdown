---
layout: default
title: "Language: Summary"
---


[autoload]: ./lang_namespaces.html#autoloader-behavior
[config]: /guides/configuring.html
[usecacheonfailure]: /references/latest/configuration.html#usecacheonfailure
[fileserve]: ./modules_fundamentals.html#files
[sitepp]: /references/glossary.html#site-manifest
[classes]: ./lang_classes.html
[enc]: /guides/external_nodes.html
[resources]: ./lang_resources.html
[chaining]: ./lang_relationships.html#chaining-arrows
[modules]: ./modules_fundamentals.html
[package]: /references/latest/type.html#package
[file]: /references/latest/type.html#file
[service]: /references/latest/type.html#service
[case]: ./lang_conditional.html#case-statements
[fact]: ./lang_variables.html#facts-and-built-in-variables
[variables]: ./lang_variables.html
[relationships]: ./lang_relationships.html
[ordering]: ./lang_relationships.html#ordering-and-notification
[notification]: ./lang_relationships.html#ordering-and-notification
[declared]: /references/glossary.html#declare
[string_newline]: ./lang_datatypes.html#line-breaks
[node]: ./lang_node_definitions.html
<!-- TODO improve hiera link -->
[hiera]: https://github.com/puppetlabs/hiera

Puppet使用自己创建的配置语言，语法灵感来源于Nagios配置文件格式。它更贴近于系统管理员，不需要太多的编程经验就可以使用。

资源，类和节点
-----

Puppet语言的核心是**声明[资源][resource]**。语言中其它每一个部分都是为了使资源可以更加灵活和方便的声明。

一组资源可以组织成更大一级的配置管理单元 -- **[类][classes]**，一个资源可以用于描述一个单独的文件或是程序包，而一个类可以描述一整个服务或应用程序所用到的所有内容（包含任意数量的程序包，配置文件，守护进程和维护任务）。小的类可以整合进更大的类，以描述某个系统角色，如“数据库服务器”或”WEB应用服务器“。

不同角色的节点一般会加载不同的类，配置那些类应用到那些节点的过程就叫做**节点分类(classification)**。<!-- TODO link to a more general node classification guide --> 在Puppet语言中，可以使用[节点定义][node]的方式分类节点，也可以通过manifests外部的节点相关配置信息来分类，如[ENC][]或[Hiera][]。


顺序
-----

Puppet语言主要是在**声明**而不是列出一系列的步骤并去执行，Manifests描述的是要达到的最终状态。

manifest中的资源顺序是可以随意书写的，它们不会按照你写的顺序执行。因为Puppet认为大部分资源是互相没有关系的，如果一个资源依赖于另外一个，[你需要明确的指出][relationships](如果你想使用简便的方式让资源以书写的顺序执行，可以使用[关系链箭头][chaining])。

尽管资源的书写顺序没有关系，但Puppet语言中的某些部分依然和编译顺序有关。最常见的是变量的使用，你必须先定义赋值一个变量后，才可以使用它。

文件
-----

Puppet语言文件叫做**manifests**，扩展名为`.pp`。Manifest文件:

* 应当以UTF8编码
* 换行符可以是Unix (LF)或Windows (CRLF)格式。(注意Puppet不会自动转换[字符串中的换行符格式][string_newline])

Puppet总是从一个单一的文件开始编译，当使用puppet master模式时，这个文件是[site.pp][sitepp] ;当使用puppet apply命令时，它是命令行中执行的文件。

任意一个保存在[模块][modules]中的[类][classes]，在被从site.pp[声明][declared]后，都会在编译时[自动加载][autoload]。同时，Puppet也会自动加载在[ENC][enc]中声明的类。

因此，最简单的Puppet配置可以是一个独立的manifest文件，包含几个资源。也可以随着整合资源成模块，分组节点，定义不同类别等工作而慢慢增加复杂度。

编译和Catalogs
-----

Puppet manifests 可以使用条件逻辑来一次性的描述很多节点的配置。在配置一个节点前，Puppet会编译manifests成**catalog**，它只包含单一节点相关的配置，并且不包含条件逻辑，变量选择等内容。

Catalogs是只包含资源和顺序关系的静态文件，在Puppet执行的不同环节中，catalog会在内存中以Ruby对象形式，在传输时以JSON形式，持久化到磁盘时以YAML形式存在。当前Puppet版本使用的catalog格式尚未文件化，也没有任何规范可供查询。 

在标准的agent/master架构中，节点从Master请求catalogs，Master收到请求后编译此节点需要的配置。当Puppet以单机模式运行中，执行puppet apply命令，catalogs会在本地编译时被立即执行。

Agent节点会缓存它们最近收到的catalog。master在接收到节点的请求并编译失败时，它们会重新使用已缓存的catalog。这个行为的配置项为[puppet.conf][config]中的[`usecacheonfailure`][usecacheonfailure]。当测试manifests的更改项时，你可以关掉它以节省时间。 


例子
-----

下面这个简短的例子用于管理NTP。它使用了[package][]，file][]，和[service][]资源; 一个使用了[fact][]的[case语句][case]; [变量][variables]; [顺序][ordering]和[通知][notification]关系元参数; 和[从模块中获取文件内容][fileserve].

{% highlight ruby %}
    case $operatingsystem {
      centos, redhat: { $service_name = 'ntpd' }
      debian, ubuntu: { $service_name = 'ntp' }
    }
    
    package { 'ntp':
      ensure => installed,
    }
    
    service { 'ntp':
      name      => $service_name,
      ensure    => running,
      enable    => true,
      subscribe => File['ntp.conf'],
    }
    
    file { 'ntp.conf':
      path    => '/etc/ntp.conf',
      ensure  => file,
      require => Package['ntp'],
      source  => "puppet:///modules/ntp/ntp.conf",
      # This source file would be located on the puppet master at
      # /etc/puppetlabs/puppet/modules/ntp/files/ntp.conf (in Puppet Enterprise)
      # or
      # /etc/puppet/modules/ntp/files/ntp.conf (in open source Puppet)
    }
{% endhighlight %}


