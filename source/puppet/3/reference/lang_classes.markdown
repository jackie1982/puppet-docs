---
layout: default
title: "语法: 类"
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
[declare]: #declaring-class
[setting_parameters]: #include-like-vs-resource-like
[override]: #using-resource-like-declarations
[ldap_nodes]: http://projects.puppetlabs.com/projects/1/wiki/Ldap_Nodes
<!-- TODO point these to correct places in hiera docs -->
[hiera]: https://github.com/puppetlabs/hiera
[external_data]: https://github.com/puppetlabs/hiera
[array_search]: https://github.com/puppetlabs/hiera
[hiera_hierarchy]: https://github.com/puppetlabs/hiera



**类**是一段命名了的Puppet代码，它保存在[模块][modules]中以供稍后使用.并且除非通过名称调用，它是不会被应用到节点中的，你可以通过在manifests中**声明**或ENC中**分配**的方式将它们添加到节点的[catalog][]。

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


**参数**使类可以获取外部数据(hiera)，如果一个类需要[facts][]以外的数据来配置它自己，那么这个数据通常需要通过参数获得。

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

类可以通过`inherits`关键字从其它类派生，这可以使你从一个通用的基类中扩展功能形成特定功能的类。

> 注: Puppet 3 不支持将有参数的类做为可继承的基类，基类**必须**是无参数的。

继承会引发3件事： 

* 当派生类被声明时，它的基类也会**先**自动的声明（如果它没有已经在没的地方声明）。
* 基类成为派生类的[父域][parent_scope]，因此新类会收到一份来自于基类的所有变量和资源默认值的拷贝。
* 派生类中的代码获得覆盖基类中任意资源属性的特殊权限。

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
> **除了上面的情况，继承增加的复杂度就没那么必要了**，如果你需要某些类的资源如果首先声明，把放[include][]进其它类的定义中就好；需要从其它类读取外部数据(hiera)，可以使用[变量全名][qualified_var]替代从父域赋值；想使用”反类“模式（如停止一个正常情况下运行的服务），可以使用类的参数来覆盖标准的动作。 
> 
> 记住你仍然可以在不相关的类中[使用resource collectors来覆盖资源属性][collector_override]，nutp这个特性需要小心使用。

#### 覆盖资源属性

基类中任意资源的属性都可以通过资源[引用][resource_reference]的方式覆盖，使用的方式是引用后面加括号括起来的的”属性=>值"对：

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

它和[给现有资源添加属性][add_attribute]的语法是完全相同，但只能用在派生类里。它提供了重写资源属性的能力而不只是添加，注意在这里我们也可以使用[多资源引用]的形式。

如果你想删除前面设置的属性的值，可以将它的值覆盖成特殊的[`undef`][undef]:

{% highlight ruby %}
    class base::freebsd inherits base::unix {
      File['/etc/passwd'] {
        group => undef,
      }
    }
{% endhighlight %}

上面的代码会使Puppet不再管理这个group属性。 

> **注意：** 如果基类中使用资源式的方式声明了其它的类，那么它的派生类不能覆盖那些内部类的参数。这是一个已知的Bug。

#### 添加资源属性的值

一些资源的属性（比如[关系元参数][relationships])可以通过数组的形式接受多个值。当在派生类中覆盖属性时，你可以通过将标准的`=>`关键字换成`+>`的方式，来在原有属性值的基础上附加新的值而不是替代它们。

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



声明类
-----

在Puppet manifests中**声明**一个类会将它里面所有的资源添加到catalog中，你可以在[节点定义][node]中声明类，或是在[站点级manifests][sitedotpp]的全局域中，在其它的类或[自定义类型][definedtype]中，声明类并不是唯一的添加它们到catalog的方式，你也可以使用[通过ENC分配类到节点](#assigning-class-from-an-enc).

类是单例的，尽管一个给定的类会因为参数的不同而有很大差别的行为，每次编译，类中的资源都只会被**运算一次**。 

### Include式与Resource式

Puppet有两种主要的方式来声明类: include式和resource式。 

> **注意：** 对于一个给定的类而言，这两种声明方式**不可以混用**。Puppet并未定义同时使用这两种方式来声明或分配类的行为，如果你真这么做了，有可能能正常工作，有可能会编译失败。

#### Include式的行为

[include式]: #include-like-behavior

`include`, `require`, 和`hiera_include`函数让你可以安全的**多次**声明同一个类，但它只会被添加到catalog中一次。这可以允许类或自定义类型管理它们自己的依赖关系，并允许你在某个节点拥有多个角色时，重复添加“角色”类。

Include-like behavior relies on [external data][external_data] and defaults for class parameter values, which allows the external data source to act like cascading configuration files for all of your class. 当类被声明时，Puppet将尝试通过下列的步骤给类的参数赋值：

1. 从[外部数据(hiera)源][external_data]获取值，使用`<class name>::<parameter name>`的形式。(例如，获取`apache`类的`version`参数，Puppet会查找`apache::version`。)
2. 使用默认值。
3. 如果没找到任何一个值，编译将会失败。

> **Aside: 最佳实践**
>
> **大多数**用户在**大多数**情况下都应该使用include式的声明，并且在外部数据(hiera)中设置参数的值。无论如何，要兼容以前的老版本就需要付出妥协。详情[Aside: 为多个Puppet版本编写代码][aside_history]。

> **版本注意事项:** 自动外部参数查找是Puppet 3中的新特性，2.7和之前的版本只能用默认值或是resource式声明的方式来覆盖参数值。[详情见下面的内容][aside_history]。

#### Resource式的行为

[resource式]: #resource-like-behavior

Resource式类的声明需要你**给定的类只声明一次**。这允许你在编译的时候覆盖参数默认值，而没有指定覆盖的参数将会回退到查找[外部数据(hiera)][external_data]的步骤。当类被声明时，Puppet会尝试下列的方式为类的参数赋值：

1. 如果指定了值，参数的默认值会被覆盖。 
2. 从[外部数据(hiera)源][external_data]获取值，使用`<class name>::<parameter name>`的形式。(例如，获取`apache`类的`version`参数，Puppet会查找`apache::version`。)
3. 使用默认值。
4. 如果没找到任何一个值，编译将会失败。

> **Aside: 为什么Resource式的声明必须是唯一的?**
>
> 这是为了避免参数值发生诡异或冲突的情况，通过声明类的方式覆盖参数值总是排在首位的，在编译时计算，并且不会与内置的hierarchy的解析发生冲突，允许重复覆盖会导致catalog编译过程变的不那么可靠。
> 
> 这就是设计外部数据(hiera)(绑定数据到include式声明的类）的原因，这样外部数据会在编译之前设置，并且有固定的分层结构，编译器就可以安全的依赖于这个体系，避免掉参数值冲突的风险。


### 使用`include`

`include`[函数][function]最标准的声明类的方式。

{% highlight ruby %}
    include base::linux
    include base::linux # no additional effect; the class is only declared once

    include base::linux, apache # including a list

    $my_类 = ['base::linux', 'apache']
    include $my_class # including an array
{% endhighlight %}

`include`函数的行为是[include式][include-like]式的（可以多次声明，参数依赖于外部数据（hiera)）。它可以接受：

* 一个单独的类
* 一个逗号分隔的类的列表
* 一个类的数组

### 使用`require`

`require`函数（不要与[`require`元参数][relationships]搞混)声明一个或多个类，然后使之变成当前容器的[依赖项][relationships]。

{% highlight ruby %}
    define apache::vhost ($port, $docroot, $servername, $vhost_name) {
      require apache
      ...
    }
{% endhighlight %}

在上面的例子中，Puppet会确保`apache`类的每一个资源都在**任意一个**`apache::vhost`实例的所有资源前执行。 

`require`函数的行为是[include式][include-like]式的（可以多次声明，参数依赖于外部数据（hiera)）。它可以接受：

* 一个单独的类
* 一个逗号分隔的类的列表
* 一个类的数组

### 使用`hiera_include`

`hiera_include`函数从[Hiera][]获取类名的列表，然后声明所有的，使用了[array resolution type][array_search]之后，它将获得从分层结构的各层include了的类的一份组合列表。它可以让你放弃[节点定义][node]转而使用像是轻量级别ENC的Hiera。

    # /etc/puppetlabs/puppet/hiera.yaml
    ...
    hierarchy:
      - %{::clientcert}
      - common

    # /etc/puppetlabs/puppet/hieradata/web01.example.com.yaml
    ---
    class:
      - apache
      - memcached
      - wordpress

    # /etc/puppetlabs/puppet/hieradata/common.yaml
    ---
    class:
      - base::linux

{% highlight ruby %}
    # /etc/puppetlabs/puppet/manifests/site.pp
    hiera_include(class)
{% endhighlight %}

在上面的例子中，`web01.example.com`将会声明`apache`, `memcached`, `wordpress`, 和`base::linux`类。其它节点将只会声明`base::linux`类。

`hiera_include`函数 函数的行为是[include式][include-like]式的（可以多次声明，参数依赖于外部数据（hiera)）。它接受单个lookup key。

### 使用Resource式声明

Resource式声明看起来像[正常的资源声明][resource_declaration]，使用特殊的`class`伪资源类型。 

{% highlight ruby %}
    # Overriding a parameter:
    class {'apache':
      version => '2.2.21',
    }
    # Declaring a class with no parameters:
    class {'base::linux':}
{% endhighlight %}

Resource式声明的行为是[resource式][resource-like]的(不允许多次声明，参数可以在编译时覆盖。)。你可以以资源属性的形式指定任意一个参数的值，未指定的参数将会走"外部/默认/失败"(external/default/fail)的路。

另外，关于类的参数，你也可以指定任意一个[元参数][metaparameters]。这种情况下，每个类里的资源都会拥有这个元参数:

{% highlight ruby %}
    # Cause the entire class to be noop:
    class {'apache':
      noop => true,
    }
{% endhighlight %}

不管如何，记住:

* Any resource can specifically override metaparameter values received from its container.
* Metaparameters which can take more than one value (like the [relationship][relationships] metaparameters) will merge the values from the container and any resource-specific values.



通过ENC分配类
-----

类也可以通过[ENC][enc]和[LDAP节点][ldap_nodes]分配到节点。Note that most ENCs assign 类 with include-like behavior, and some ENCs assign them with resource-like behaior. See the [documentation of the ENC interface][enc] or the documentation of your specific ENC for complete details.





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
