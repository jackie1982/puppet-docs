---
layout: default
title: "Language: Visual Index"
---


[resource]: ./lang_resources.html
[type]: ./lang_resources.html#type
[title]: ./lang_resources.html#title
[attribute]: ./lang_resources.html#attributes
[value]: ./lang_resources.html#attributes
[string]: ./lang_datatypes.html#strings
[function]: ./lang_functions.html
[rvalue]: ./lang_functions.html#behavior
[template_func]: /guides/templating.html
[module]: modules_fundamentals.html
[argument]: ./lang_functions.html#arguments
[relationship_meta]: ./lang_relationships.html#relationship-metaparameters
[refs]: ./lang_datatypes.html#resource-references
[chaining]: ./lang_relationships.html#chaining-arrows
[variable]: ./lang_variables.html
[array]: ./lang_datatypes.html#arrays
[hash]: ./lang_datatypes.html#hashes
[interpolation]: ./lang_datatypes.html#variable-interpolation
[class_def]: ./lang_classes.html#defining-a-class
[class_decl]: ./lang_classes.html#defining-a-class
[defined_type]: ./lang_defined_types.html
[namespace]: ./lang_namespaces.html
[defined_resource]: ./lang_defined_types.html#declaring-an-instance
[node]: ./lang_node_definitions.html
[regex_node]: ./lang_node_definitions.html#regular-expression-names
[import]: ./lang_import.html
[comments]: ./lang_comments.html
[if]: ./lang_conditional.html#if-statements
[expressions]: ./lang_expressions.html
[built_in]: ./lang_variables.html#facts-and-built-in-variables
[facts]: ./lang_variables.html#facts
[regex]: ./lang_datatypes.html#regular-expressions
[regex_match]: ./lang_expressions.html#regex-match
[in]: ./lang_expressions.html#in
[case]: ./lang_conditional.html#case-statements
[selector]: ./lang_conditional.html#selectors
[collector]: ./lang_collectors.html
[export_collector]: ./lang_collectors.html#exported-resource-collectors
[export]: ./lang_exported.html
[defaults]: ./lang_defaults.html
[override]: ./lang_classes.html#overriding-resource-attributes
[inherits]: ./lang_classes.html#inheritance
[coll_override]: ./lang_resources.html#amending-attributes-with-a-collector
[virtual]: ./lang_virtual.html

此页可以在你不记得语法元素的名字的时候帮助你找到它们。


{% highlight ruby %}
    file {'ntp.conf':
      path    => '/etc/ntp.conf',
      ensure  => file,
      content => template('ntp/ntp.conf'),
      owner   => root,
      mode    => 0644,
    }
{% endhighlight %}

↑ 一个[资源声明][resource].

* `file`: [资源类型][type]
* `ntp.conf`: [标题][title]
* `path`: [属性][attribute]
* `'/etc/ntp.conf'`: [值][value]; 在这里，它是一个[字符串][string]
* `template('ntp/ntp.conf')`: [函数][function]调用，[返回一个值][rvalue]; 在这里使用的是[template][template_func]函数, [参数][argument]为ntp[模块][module]中g一个模板的名字。

{% highlight ruby %}
    package {'ntp':
      ensure => installed,
      before => File['ntp.conf'],
    }
    service {'ntpd': 
      ensure    => running,
      subscribe => File['ntp.conf'],
    }
{% endhighlight %}

↑ 两个使用了`before`和`subscribe`[关系元参数][relationship_meta]的资源 (它们接受[资源引用][refs]).

{% highlight ruby %}
    Package['ntp'] -> File['ntp.conf'] ~> Service['ntpd']
{% endhighlight %}

↑ 通过[顺序链箭头][chaining]指定3个资源间的顺序关系, 使用[资源引用][refs]的形式。

{% highlight ruby %}
    $package_list = ['ntp', 'apache2', 'vim-nox', 'wget']
{% endhighlight %}

↑ 将一个[数组][array]赋值给一个[变量][variable]。

{% highlight ruby %}
    $myhash = { key => { subkey => 'b' }}
{% endhighlight %}

↑ 将一个[哈希][hash]赋值给一个[变量][variable]。

{% highlight ruby %}
    ...
    content => "Managed by puppet master version ${serverversion}"
{% endhighlight %}

↑ 将一个Master提供的[内置变量][built-in variable] [内插进双引号括起来的字符串中][interpolation] (带可选的花括号).


{% highlight ruby %}
    class ntp {
      package {'ntp':
        ...
      }
      ...
    }
{% endhighlight %}

↑ [定义类][class_def]，定义完成后类可以在以后使用。

{% highlight ruby %}
    include ntp
    require ntp
    class {'ntp':}
{% endhighlight %}

↑ 以3种不同的方式[声明类][class_decl]: 通过include函数，require函数和resource-like语法. 声明类后，其中的资源将会增加进当前节点使用Puppet管理。


{% highlight ruby %}
    define apache::vhost ($port, $docroot, $servername = $title, $vhost_name = '*') {
      include apache
      include apache::params
      $vhost_dir = $apache::params::vhost_dir
      file { "${vhost_dir}/${servername}.conf":
          content => template('apache/vhost-default.conf.erb'), 
          owner   => 'www',
          group   => 'www',
          mode    => '644',
          require => Package['httpd'],
          notify  => Service['httpd'],
      }
    }
{% endhighlight %}

↑ [自定义类型][defined type], 创建一个新的资源类型。注意这个类型的名字有两个[命名空间节][namespace]。

{% highlight ruby %}
    apache::vhost {'homepages':
      port    => 8081,
      docroot => '/var/www-testhost',
    }
{% endhighlight %}

↑ 声明一个上面已经定义了的[自定义类型的资源][defined_resource] (或实例化)。

{% highlight ruby %}
    Apache::Vhost['homepages']
{% endhighlight %}

↑ 上面已经声明的自定义类型资源的[资源实例引用][refs]，注意每个[命令空间节][namespace]都必须是首字母大写。

{% highlight ruby %}
    node 'www1.example.com' {
      include common
      include apache
      include squid
    }
{% endhighlight %}

↑ 一个[节点定义][node].

{% highlight ruby %}
    node /^www\d+$/ {
      include common
    }
{% endhighlight %}

↑ 一个[正则表达式形式的节点定义][regex_node]。

{% highlight ruby %}
    import nodes/*.pp
{% endhighlight %}

↑ 一个[import语句][import]，建议仅在少数的几种情况下使用。

{% highlight ruby %}
    # comment
    /* comment */
{% endhighlight %}

↑ 两条[注释][comments]。


{% highlight ruby %}
    if $is_virtual == 'true' {
      warn( 'Tried to include class ntp on virtual machine; this node may be misclassified.' )
    }
    elsif $operatingsystem == 'Darwin' {
      warn ( 'This NTP module does not yet work on our Mac laptops.' )
    else {
      include ntp
    }
{% endhighlight %}

↑ 一条[if语句][if], 条件是一个使用了agent提供的[facts][]的[表达式][expressions]。


{% highlight ruby %}
    if $hostname =~ /^www(\d+)\./ {
      notify { "Welcome web server #$1": }
    }
{% endhighlight %}

↑ 一条使用了[正则表达式][regex]和[匹配操作符][regex_match]的[if语句][if]。

{% highlight ruby %}
    if 'www' in $hostname {
      ...
    }
{% endhighlight %}

↑ 一条使用了[`in`表达式][in]的[if语句][if]。

{% highlight ruby %}
    case $operatingsystem {
      'Solaris':          { include role::solaris }
      'RedHat', 'CentOS': { include role::redhat  }
      /^(Debian|Ubuntu)$/:{ include role::debian  }
      default:            { include role::generic }
    }
{% endhighlight %}

↑ 一条[case语句][case].

{% highlight ruby %}
    $rootgroup = $osfamily ? {
        'Solaris'          => 'wheel',
        /(Darwin|FreeBSD)/ => 'wheel',
        default            => 'root',
    }
{% endhighlight %}

↑ 一条用于定义`$rootgroup`[变量][variable]值的[selector语句][selector]。

{% highlight ruby %}
    User <| groups == 'admin' |>
{% endhighlight %}

↑ 一个[resource collector][collector]，又叫”飞船操作符”。

{% highlight ruby %}
    Concat::Fragment <<| tag == "bacula-storage-dir-${bacula_director}" |>>
{% endhighlight %}

↑ 一个[exported resource collector][export_collector]，与[exported resources][export]一同使用。

{% highlight ruby %}
    Exec { 
      path        => '/usr/bin:/bin:/usr/sbin:/sbin',
      environment => 'RUBYLIB=/opt/puppet/lib/ruby/site_ruby/1.8/',
      logoutput   => true,
      timeout     => 180,
    }
{% endhighlight %}

↑ 一个`exec`资源类型的[资源默认值][defaults]。

{% highlight ruby %}
    Exec['update_migrations'] {
      environment => 'RUBYLIB=/usr/lib/ruby/site_ruby/1.8/',
    }
{% endhighlight %}

↑ 一个[resource override][override]，只可在[子类][inherits]中使用。

{% highlight ruby %}
    Exec <| title == 'update_migrations' |> {
      environment => 'RUBYLIB=/usr/lib/ruby/site_ruby/1.8/',
    }
{% endhighlight %}

↑ 一个[resource override using a collector][coll_override]，可以使用在任何位置。危险，但在某些时候非常有用。


{% highlight ruby %}
    @user {'deploy':
      uid     => 2004,
      comment => 'Deployment User',
      group   => www-data,
      groups  => ["enterprise"],
      tag     => [deploy, web],
    }
{% endhighlight %}

↑ 一个[虚拟资源][virtual].


{% highlight ruby %}
    @@nagios_service { "check_zfs${hostname}":
      use                 => 'generic-service',
      host_name           => "$fqdn",
      check_command       => 'check_nrpe_1arg!check_zfs',
      service_description => "check_zfs${hostname}",
      target              => '/etc/nagios3/conf.d/nagios_service.cfg',
      notify              => Service[$nagios::params::nagios_service],
    }
{% endhighlight %}

↑ 一个[exported resource][export]声明。

