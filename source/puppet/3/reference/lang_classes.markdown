---
layout: default
title: "Language: 类"
---

<!-- TODO: Need better link for hiera -->
[hiera]: https://github.com/puppetlabs/hiera
<!-- TODO: need better link for site.pp -->
[sitedotpp]: ./lang_summary.html#files
[collector_override]: ./lang_resources.html#amending-attributes-with-a-collector
[namespace]: ./lang_namespaces.html
[enc]: /guides/external_nodes.html
[tags]: ./lang_tags.html
[allowed]: ./lang_reserved.html#类-and-types
[function]: ./lang_functions.html
[modules]: ./modules_fundamentals.html
[contains]: ./lang_containment.html
[contains_float]: ./lang_containment.html#known-issues
[function]: ./lang_functions.html
[multi_ref]: ./lang_datatypes.html#multi-resource-references
[add_attribute]: ./lang_resources.html#adding-or-modifying-attributes
[undef]: ./lang_datatypes.html#undef
[relationships]: ./lang_relationships.html
[qualified_var]: ./lang_variables.html#accessing-out-of-scope-variables
[variable]: ./lang_variables.html
[variable_assignment]: ./lang_variables.html#assignment
[resource_reference]: ./lang_datatypes.html#resource-references
[node]: ./lang_node_definitions.html
[resource_declaration]: ./lang_resources.html
[scope]: ./lang_scope.html
[parent_scope]: ./lang_scope.html#scope-lookup-rules
[definedtype]: ./lang_defined_types.html
[metaparameters]: ./lang_resources.html#metaparameters
[catalog]: ./lang_summary.html#compilation-and-catalogs
[facts]: ./lang_variables.html#facts-and-built-in-variables
[import]: ./lang_import.html
[declare]: #declaring-类
[setting_parameters]: #include-like-vs-resource-like
[override]: #using-resource-like-declarations
[ldap_nodes]: http://projects.puppetlabs.com/projects/1/wiki/Ldap_Nodes
<!-- TODO point these to correct places in hiera docs -->
[hiera]: https://github.com/puppetlabs/hiera
[external_data]: https://github.com/puppetlabs/hiera
[array_search]: https://github.com/puppetlabs/hiera
[hiera_hierarchy]: https://github.com/puppetlabs/hiera



**类**是一段全名了的Puppet代码，它保存在[模块][modules]中以供稍后使用.并且除非通过名称调用，它是不会被应用到节点中的，你可以通过在manifests中**声明**或ENC中**分配**的方式将它们添加到节点的[catalog][]。

类通常用于配置大型或中等大小的功能，如一整个应用程序需要的程序包，配置文件和守护进程。 

定义类
-----

定义一个类使它可以在以后使用。定义后，类默认是不会被添加到catalog中的，必须要[声明][declare]或通过[ENC指定][enc]才可以。

### 语法

{% highlight ruby %}
    # 一个没有参数的类
    class base::linux {
      file { '/etc/passwd':
        owner => 'root',
        group => 'root',
        mode  => '0644',
      }
      file { '/etc/shadow':
        owner => 'root',
        group => 'root',
        mode  => '0440',
      }
    }
{% endhighlight %}

{% highlight ruby %}
    # 一个带参数的类
    class apache ($version = 'latest') {
      package {'httpd':
        ensure => $version, # Using the class parameter from above
        before => File['/etc/httpd.conf'],
      }
      file {'/etc/httpd.conf':
        ensure  => file,
        owner   => 'httpd',
        content => template('apache/httpd.conf.erb'), # Template from a module
      }
      service {'httpd':
        ensure => running,
        enable => true,
        subscribe => File['/etc/httpd.conf'],
      }
    }
{% endhighlight %}

定义一个类的标准格式为：

* `class`关键字
* [类名][allowed]
* 一组可选的**参数**：
    * 左小括号
    * 逗号分隔的**参数**：
        * 一个新的[变量][variable]名，包括`$`前缀
        * 一个可选的等于号(=)和**默认值**(任意数据类型)
    * 最后的参数后的逗号可选
    * 右小括号
* 可选的`inherits`关键字和紧跟的一个类名
* 左花括号
* 一段任意的Puppet代码，一般至少包含一个[资源定义][resource_declaration]
* 右花括号


### 类的参数和变量


**参数**使类可以获取外部数据，如果一个类需要[facts][]以外的数据来配置它自己，那么这个数据通常需要通过参数获得。

在定义时，每个类的参数都可以在内部以正常[变量][variable]的形式使用。这些变量并不是通过[正常的赋值语句][variable_assignment]或从全局/节点域中读取赋值，而是[在类被声明时自动设置][setting_parameters]. 

注意如果一个类的参数没有设置默认值，使用这个类(模块)时，**必须**指定它的值(either in their [external data][external_data] or an [override][]).因此，你应当尽可能的写上默认值。

### 位置

类的定义应该保存在模块中，Puppet对[模块][modules]中的类是自动感知并可以自动加载的。

类应该保存在它自己模块的`manifests/`目录中的，每个类一个文件且文件名可以映射到类的名字。详情查看[模块基础][modules]和[命名空间和自动加载][namespace] 

> #### 其它位置
>
> 大多数的用户应该**只**从模块中加载类。但，你仍然可以在下列的位置定义类：
> 
> * [The site manifest][sitedotpp]. 如果放在这里，它们可以放在这个文件的任何位置，并且不会有编译顺序依赖。
> * [Imported manifests][import]. 如果放在这里，你必须在声明一个类前[import][]包含它的定义的文件。
> * 其它类的定义 这会使内部类在外部类的[命名空间][namespace]下，导致它真实的名字与定义的不一致。这样的话，内部类并不会随着外部类的声明而自动声明。嵌套的类不能被自动加载，因为它对于Puppet来说是不可见的，要使用这个类，包含它的manifest必须强制加载( 自动加载最外层的类，使用[import][]语法导入文件或是将整个嵌套结构放到主站点manifest中)。尽管嵌套类还没有正式的被废弃掉，但**非常强烈**的不建议使用。

### 容器

一个类[包含][contains]了它自己所有的资源，意思是说赋予给类的所有[顺序关系][relationships]都会添加到整个类的所有资源上。

注意类不能包含或者它，这是一个已知的设计问题。详情见[容器的相关事项][contains_float].

### 自动标签

每个类中的资源都会自动获得类名的[标签][tags](和其[命名空间节][namespace]).

### 继承

类可以通过`inherits`关键字从其它类衍生，这可以使你从一个通用的基类中扩展功能形成特定功能的类。

> 注: Puppet 3 不支持将有参数的类做为可继承的基类，基类**必须**是无参数的。

继承会引发3件事： 

* 当衍生类被声明时，它的基类也会**先**自动的声明（如果它没有已经在没的地方声明）。
* 基类成为衍生类的[父域][parent_scope]，因此新类会收到一份来自于基类的所有变量和资源默认值的拷贝。
* 衍生类中的代码获得覆盖基类中任意资源属性的特殊权限。

> #### Aside: 什么时候继承
>
> 类的继承应当仅在下列几种情况下**非常甚重**的使用：
> 
> * 当你需要覆盖基类的资源属性时。
> * 让一个“参数类”为其它类提供参数的默认值：
> 
>       class example ($my_param = $example::params::myparam) inherits example::params { ... }
> 
>   这个工作模式用于保证Puppet在尝试运算主类的参数列表前，首先运算”参数类“中的代码。这在你想要参数的默认值根据Facts或其它数据的不同而变化时特别有用，能使你减少条件逻辑的使用。
> 
> **除了上面的情况，继承增加的复杂度就没那么必要了**，如果你需要某些类的资源如果首先声明，把放[include][]进其它类的定义中就好；需要从其它类读取外部数据，可以使用[变量全名][qualified_var]替代从父域赋值；想使用”反类“模式（如停止一个正常情况下运行的服务），可以使用类的参数来覆盖标准的动作。 
> 
> 记住你仍然可以在不相关的类中[使用resource collectors来覆盖资源属性][collector_override]，nutp这个特性需要小心使用。

#### 覆盖资源属性

The attributes of any resource in the base class can be overridden with a [reference][resource_reference] to the resource you wish to override, followed by a set of curly braces containing attribute => value pairs:

{% highlight ruby %}
    class base::freebsd inherits base::unix {
      File['/etc/passwd'] {
        group => 'wheel'
      }
      File['/etc/shadow'] {
        group => 'wheel'
      }
    }
{% endhighlight %}

This is identical to the syntax for [adding attributes to an existing resource][add_attribute], but in a derived class, it gains the ability to rewrite resources instead of just adding to them. Note that you can also use [multi-resource references][multi_ref] here.

You can remove an attribute's previous value without setting a new one by overriding it with the special value [`undef`][undef]:

{% highlight ruby %}
    class base::freebsd inherits base::unix {
      File['/etc/passwd'] {
        group => undef,
      }
    }
{% endhighlight %}

This causes the attribute to be unmanaged by Puppet. 

> **Note:** If a base class declares other 类 with the resource-like syntax, a class derived from it cannot override the class parameters of those inner 类. This is a known bug. 

#### Appending to Resource Attributes

Some resource attributes (such as the [relationship metaparameters][relationships]) can accept multiple values in an array. When overriding attributes in a derived class, you can add to the existing values instead of replacing them by using the `+>` ("plusignment") keyword instead of the standard `=>` hash rocket:

{% highlight ruby %}
    class apache {
      service {'apache':
        require => Package['httpd'],
      }
    }

    class apache::ssl inherits apache {
      # host certificate is required for SSL to function
      Service['apache'] {
        require +> [ File['apache.pem'], File['httpd.conf'] ],
        # Since `require` will retain its previous values, this is equivalent to: 
        # require => [ Package['httpd'], File['apache.pem'], File['httpd.conf'] ],
      }
    }
{% endhighlight %}



Declaring 类
-----

**Declaring** a class in a Puppet manifest adds all of its resources to the catalog. You can declare 类 in [node definitions][node], at top scope in the [site manifest][sitedotpp], and in other 类 or [defined types][definedtype]. Declaring 类 isn't the only way to add them to the catalog; you can also [assign 类 to nodes with an ENC](#assigning-类-from-an-enc).

类 are singletons --- although a given class may have very different behavior depending on how its parameters are set, the resources in it will only be evaluated **once per compilation.** 

### Include-Like vs. Resource-Like

Puppet has two main ways to declare 类: include-like and resource-like. 

> **Note:** These two behaviors **should not be mixed** for a given class. Puppet's behavior when declaring or assigning a class with both styles is undefined, and will sometimes work and sometimes cause compilation failures. 

#### Include-Like Behavior

[include-like]: #include-like-behavior

The `include`, `require`, and `hiera_include` functions let you safely declare a class **multiple times;** no matter how many times you declare it, a class will only be added to the catalog once. This can allow 类 or defined types to manage their own dependencies, and lets you create overlapping "role" 类 where a given node may have more than one role.

Include-like behavior relies on [external data][external_data] and defaults for class parameter values, which allows the external data source to act like cascading configuration files for all of your 类. When a class is declared, Puppet will try the following for each of its parameters:

1. Request a value from [the external data source][external_data], using the key `<class name>::<parameter name>`. (For example, to get the `apache` class's `version` parameter, Puppet would search for `apache::version`.)
2. Use the default value. 
3. Fail compilation with an error if no value can be found.

> **Aside: Best Practices**
>
> **Most** users in **most** situations should use include-like declarations and set parameter values in their external data. However, compatibility with earlier versions of Puppet may require compromises. See [Aside: Writing for Multiple Puppet Versions][aside_history] below for details.

> **Version Note:** Automatic external parameter lookup is a new feature in Puppet 3. Puppet 2.7 and earlier could only use default values or override values from resource-like declarations. [See below for more details.][aside_history]

#### Resource-like Behavior

[resource-like]: #resource-like-behavior

Resource-like class declarations require that you **only declare a given class once.** They allow you to override class parameters at compile time, and will fall back to [external data][external_data] for any parameters you don't override.  When a class is declared, Puppet will try the following for each of its parameters:

1. Use the override value from the declaration, if present.
2. Request a value from [the external data source][external_data], using the key `<class name>::<parameter name>`. (For example, to get the `apache` class's `version` parameter, Puppet would search for `apache::version`.)
3. Use the default value. 
4. Fail compilation with an error if no value can be found.

> **Aside: Why Do Resource-Like Declarations Have to Be Unique?**
>
> This is necessary to avoid paradoxical or conflicting parameter values. Since overridden values from the class declaration always win, are computed at compile-time, and do not have a built-in hierarchy for resolving conflicts, allowing repeated overrides would cause catalog compilation to be unreliable and parse-order dependent.
> 
> This was the original reason for adding external data bindings to include-like declarations: since external data is set **before** compile-time and has a **fixed hierarchy,** the compiler can safely rely on it without risk of conflicts. 


### Using `include`

The `include` [function][] is the standard way to declare 类.

{% highlight ruby %}
    include base::linux
    include base::linux # no additional effect; the class is only declared once

    include base::linux, apache # including a list

    $my_类 = ['base::linux', 'apache']
    include $my_类 # including an array
{% endhighlight %}

The `include` function uses [include-like behavior][include-like]. (Multiple declarations OK; relies on external data for parameters.) It can accept:

* A single class
* A comma-separated list of 类
* An array of 类

### Using `require`

The `require` function (not to be confused with the [`require` metaparameter][relationships]) declares one or more 类, then causes them to become a [dependency][relationships] of the surrounding container.

{% highlight ruby %}
    define apache::vhost ($port, $docroot, $servername, $vhost_name) {
      require apache
      ...
    }
{% endhighlight %}

In the above example, Puppet will ensure that every resource in the `apache` class gets applied before every resource in **any** `apache::vhost` instance. 

The `require` function uses [include-like behavior][include-like]. (Multiple declarations OK; relies on external data for parameters.) It can accept:

* A single class
* A comma-separated list of 类
* An array of 类

### Using `hiera_include`

The `hiera_include` function requests a list of class names from [Hiera][], then declares all of them. Since it uses the [array resolution type][array_search], it will get a combined list that includes 类 from **every level** of the [hierarchy][hiera_hierarchy]. This allows you to abandon [node definitions][node] and use Hiera like a lightweight ENC. 

    # /etc/puppetlabs/puppet/hiera.yaml
    ...
    hierarchy:
      - %{::clientcert}
      - common

    # /etc/puppetlabs/puppet/hieradata/web01.example.com.yaml
    ---
    类:
      - apache
      - memcached
      - wordpress

    # /etc/puppetlabs/puppet/hieradata/common.yaml
    ---
    类:
      - base::linux

{% highlight ruby %}
    # /etc/puppetlabs/puppet/manifests/site.pp
    hiera_include(类)
{% endhighlight %}

On the node `web01.example.com`, the example above would declare the 类 `apache`, `memcached`, `wordpress`, and `base::linux`. On other nodes, it would only declare `base::linux`.

The `hiera_include` function uses [include-like behavior][include-like]. (Multiple declarations OK; relies on external data for parameters.) It accepts a single lookup key.

### Using Resource-Like Declarations

Resource-like declarations look like [normal resource declarations][resource_declaration], using the special `class` pseudo-resource type. 

{% highlight ruby %}
    # Overriding a parameter:
    class {'apache':
      version => '2.2.21',
    }
    # Declaring a class with no parameters:
    class {'base::linux':}
{% endhighlight %}

Resource-like declarations use [resource-like behavior][resource-like]. (Multiple declarations prohibited; parameters may be overridden at compile-time.) You can provide a value for any class parameter by specifying it as resource attribute; any parameters not specified will follow the normal external/default/fail lookup path.

In addition to class-specific parameters, you can also specify a value for any [metaparameter][metaparameters]. In such cases, every resource contained in the class will also have that metaparameter:

{% highlight ruby %}
    # Cause the entire class to be noop:
    class {'apache':
      noop => true,
    }
{% endhighlight %}

However, note that:

* Any resource can specifically override metaparameter values received from its container.
* Metaparameters which can take more than one value (like the [relationship][relationships] metaparameters) will merge the values from the container and any resource-specific values.



Assigning 类 From an ENC
-----

类 can also be assigned to nodes by [external node classifiers][enc] and [LDAP node data][ldap_nodes]. Note that most ENCs assign 类 with include-like behavior, and some ENCs assign them with resource-like behaior. See the [documentation of the ENC interface][enc] or the documentation of your specific ENC for complete details.





[aside_history]: #aside-writing-for-multiple-puppet-versions

> Aside: Writing for Multiple Puppet Versions
> -----
> 
> Hiera integration and automatic parameter lookup are new features in Puppet 3; older versions may install the Hiera functions as an add-on, but will not automatically find parameters. If you are writing code for multiple Puppet versions, you have several options:
> 
> ### Expect Users to Handle Parameters
> 
> The simplest approach is to not look back, and expect Puppet 2.x users to use resource-like declarations. This isn't the friendliest approach, but many modules did this even before auto-parameters were available, and users are accustomed to a subset of their modules requiring it.
> 
> ### Use Hiera Functions in Default Values
> 
> If you are willing to require Hiera and the `hiera-puppet` add-on package for pre-3.0 users, you can emulate Puppet 3's behavior by using a `hiera` function call in each parameter's default value:

{% highlight ruby %}
    class example ( $parameter_one = hiera('example::parameter_one'), $parameter_two = hiera('example::parameter_two') ) {
      ...
    }
{% endhighlight %}

> Be sure to use 3.0-compatible lookup keys (`<class name>::<parameter>`). This will let 2.x users declare the class with `include`, and their Hiera data will continue to work without changes once they upgrade to Puppet 3.
> 
> This approach can also be combined with the "params class" pattern, if default values are necessary:

{% highlight ruby %}
    class example ( 
      $parameter_one = hiera('example::parameter_one', $example::params::parameter_one),
      $parameter_two = hiera('example::parameter_two', $example::params::parameter_two)
    ) inherits example::params { # Inherit the params class to let the parameter list see its variables.
      ...
    }
{% endhighlight %}

> The drawbacks of this approach are:
> 
> * It requires 2.x users to install Hiera and `hiera-puppet`.
> * It's slower on Puppet 3 --- if you don't set a value in your external data, Puppet will do _two_ searches before falling back to the default value.
> 
> However, depending on your needs, it can be a useful stopgap until Puppet 3 is widely adopted. 
> 
> ### Avoid Class Parameters
> 
> Prior to Puppet 2.6, 类 could only request data by reading arbitrary variables outside their local [scope][]. It is still possible to design 类 like this. **However,** since dynamic scope was removed in Puppet 3, old-style 类 can only read **top-scope or node-scope** variables, which makes them less flexible than they were in previous versions. Your best options for using old-style 类 with Puppet 3 are to use an ENC to set your 类' variables, or to manually insert `$special_variable = hiera('class::special_variable')` calls at top scope in your site manifest.
