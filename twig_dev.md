#开发者的Twig

本章讲述的是Twig的API，而不是Twig的模板语言。这些内容会对为应用实现模板接口的工作很有帮助，对模板的制作工作就没什么意义了。

##基础

Twig的中心对象为**environment**（是`Twig_Environment`类的实例）。这个类的实例用于存储配置和扩展，并用来从文件系统或者其他位置载入模板。

绝大多数应用需要在应用初始化的时候，创建一个`Twig_Environment`对象，然后用它来加载模板。在有些案例中，会有多个环境同时并存，用来处理不同的配置。

下面举个简单例子，配置Twig来为应用载入模板：

	require_once '/path/to/lib/Twig/Autoloader.php';
	Twig_Autoloader::register();
	
	$loader = new Twig_Loader_Filesystem('/path/to/templates');
	$twig = new Twig_Environment($loader, array(
	    'cache' => '/path/to/compilation_cache',
	));
	
上面的代码会创建一个缺省设置的环境，然后加载器会去`/path/to/templates/`目录查找模板。有不同的加载器可以使用，你可以实现自己的加载器用来从数据库或者其他什么资源中获取模板。

> 注意`environment`的第二个参数就是一个存储设置项的数组。`cache`选项声明了一个目录，用于存储Twig编译的模板，这一缓存能够减少解析的开销。这种缓存跟你希望在处理模板过程中加入的缓存是不一样的，所以如果需要自行缓存，请使用其他的PHP缓存库。

要从环境中载入模板，只需要调用`loadTemplate()`方法，这个方法会返回一个`Twig_Template`实例：

	$template = $twig->loadTemplate('index.html');

要使用变量来渲染这个模板，可以使用`render()`方法：

	echo $template->render(array('the' => 'variables', 'go' => 'here'));

> `display()`方法是直接输出模板的一个快捷方法。

还可以一口气完成加载和渲染：

	echo $twig->render('index.html', array('the' => 'variables', 'go' => 'here'));
	
##环境选项

当创建一个新的`Twig_Environment`实例时，可以创建一个选项数组，把他作为第二个参数传递给构造函数：

$twig = new Twig_Environment($loader, array('debug' => true));

下面列出可用的选项：

- `debug`：当设置为true，生成的模板会有一个`__toString()`方法，可以用来显示生成的Node（缺省为false）。
- `charset`：模板的字符集，缺省为utf-8。
- `base_template_class`：生成模板时用的基模板（缺省为`Twig_Template`）。
- `cache`：用来保存编译后模板的绝对路径，缺省值为false，也就是关闭缓存。
- `auto_reload`：当用Twig开发时，是有必要在每次模板变更之后都重新编译的。如果不提供一个`auto_reload`参数，他会从`debug`选项中取值。
- `strict_varibles`：如果设置为false，Twig会忽略无效的变量（无效指的是不存在的变量或者属性/方法），并将其替换为null。如果这个选项设置为true，那么遇到这种情况的时候，Twig会抛出异常。
- `autoescape`：如果设置为true, 则会为所有模板缺省启用自动转义（缺省为ture）。在Twig 1.8中，可以设置转义策略（html或者js，要关闭可以设置为false），在Twig 1.9中的转移策略，可以设置为css，url，html_attr，甚至还可以设置为回调函数，该函数需要接受一个模板文件名为参数，且必须返回要使用的转义策略，回调命名应该避免同内置的转义策略冲突。
- `optimizations`：用于指出选择使用什么优化方式（缺省为-1，代表使用所有优化；设置为0则禁止）。

##加载器

加载器用于从文件系统等资源中载入模板。

##编译缓存

所有模板加载器都能把编译后的模板缓存到文件系统中，以备重用。因为模板只编译一次，所以大大的提高了Twig的效率；如果你同时使用了APC这样的PHP加速器，这种提升会更加显著。参考上文提到的`Twig_Environment`中的`cache`以及`auto_reload`选项能够获得更多信息。

###内置加载器

这里有一个内置加载器的列表：

####Twig_Loader_Filesystem

*在1.10中新增：`prependPath()`以及命名空间支持在1.10中加入。*

`Twig_Loader_Filesystem`从文件系统中载入模板。这个加载器能够在目录中查找模板，也是推荐使用的加载器：

	$loader = new Twig_Loader_Filesystem($templateDir);

这个加载器还可以在一组目录中进行查找：

	$loader = new Twig_Loader_Filesystem(array($templateDir1, $templateDir2));

在这个配置中，Twig首先会在`$templateDir1`中查找，如果没找到，则会在`$templateDir2`中查找。

还可以使用`addPath()`和`prependPath()`方法来追加或者插入路径：

	$loader->addPath($templateDir3);
	$loader->prependPath($templateDir4);
	
文件系统加载器还支持命名空间模板。这一特性允许用户把模板分到处于不同模板路径下的不同命名空间中。

当使用`setPaths()`、`addPath()`以及`prependPath()`方法时，把命名空间作为第二个参数进行传递（如果没有传递，缺省命名空间为"main"）：

	$loader->addPath($templateDir, 'admin');

有命名空间的模板可以使用特别的`@namespace_name/template_path`注解来访问：

	$twig->render('@admin/index.html', array());
	
####Twig_Loader_Array

`Twig_Loader_Array`会从PHP数组中载入模板。数组以字符串键作为模板名称：

	$loader = new Twig_Loader_Array(array(
	    'index.html' => 'Hello {{ name }}!',
	));
	$twig = new Twig_Environment($loader);
	
	echo $twig->render('index.html', array('name' => 'Fabien'));

这个加载器对单元测试来说非常有用。一些项目中会把所有模板保存在同一个PHP文件中，这种小项目就很适合这个装载器。

> 当在开启缓存的情况下使用`Array`或者`String`加载器时，应该要注意，因为缓存使用模板源码作为缓存键，所以
每次模板内容发生变动时，都会创建新的缓存Key，如果不想缓存增长超出控制，就需要注意对过期缓存内容进行清理了。

####Twig_Loader_Chain

`Twig_Loader_Chain`委托其它加载器进行模板的加载：

	$loader1 = new Twig_Loader_Array(array(
	    'base.html' => '{% block content %}{% endblock %}',
	));
	$loader2 = new Twig_Loader_Array(array(
	    'index.html' => '{% extends "base.html" %}{% block content %}Hello {{ name }}{% endblock %}',
	    'base.html'  => 'Will never be loaded',
	));
	
	$loader = new Twig_Loader_Chain(array($loader1, $loader2));
	
	$twig = new Twig_Environment($loader);

当查找一个模板的时候，Twig会顺序尝试每一个加载器，并在找到模板之后立刻返回。当上面的例子要渲染`index.html`模板时，Twig会的话，使用`$loader2`；而会用`$loader1`加载base.html。

`Twig_loader_Chain`可以接受任何实现了`Twig_LoaderInterface`的加载器作为参数。

> 你也可以利用`addLoader()`方法来加入装载器。


###创建自己的加载器

所有的加载器都应该实现`Twig_LoaderInterface`：

		interface Twig_LoaderInterface
		{
		    /**
		     * 获取模板代码，并进行命名
		     *
		     * @param  string $name 字符串，要载入的模板的名称。		     *
		     * @return string 模板的源代码。
		     */
		    function getSource($name);
		
		    /**
		     * 通过一个模板名称，获取缓存键。
		     *
		     * @param  string $name 要载入的模板的名称
		     *
		     * @return string 缓存键
		     */
		    function getCacheKey($name);
		
		    /**
		     * 如果模板仍然是最新的，则返回true。
		     *
		     * @param string    $name 末班名称
		     * @param timestamp $time 被缓存模板的更新时间
		     */
		    function isFresh($name, $time);
		}

如果模板还是最新的，`isFresh()`方法必须返回true；否则应该返回最后一次修改时间，或返回false。

> Twig 1.11.0，还可以实现`Twig_ExistsLoaderInterface`接口，让加载器在同链式加载器协同工作时稍快一些。

##使用扩展

Twig扩展是打包起来的一些新功能。使用扩展很简单，例如`addExtension()`方法：

	$twig->addExtension(new Twig_Extension_Sandbox());
	
Twig自带了以下扩展：

- `Twig_Extension_Core`：定义所有核心特性。
- `Twig_Extension_Escaper`：添加自动输出转义，以及可能转义/反转义的Block代码。
- `Twig_Extension_Sandbox`：为缺省的Twig环境添加沙箱模式，让非信任代码的运行更安全。
- `Twig_Extension_Optimizer`：在编译之前对节点树进行优化。

核心、转义以及优化扩展缺省就会被激活，不需要加入到Twig环境中。

##内置扩展

本节会讲述内置扩展的功能。

> 也可以从本章获得如何创建自有扩展的相关知识 。

###核心扩展

`core`扩展定义了Twig的所有核心特性：

- [Tags](http://twig.sensiolabs.org/doc/tags/index.html)
- [Filters](http://twig.sensiolabs.org/doc/filters/index.html)
- [Functions](http://twig.sensiolabs.org/doc/functions/index.html)
- [Tests](http://twig.sensiolabs.org/doc/tests/index.html)

###转义扩展

`escaper`扩展为Twig加入了对输出内容进行自动转义的能力。他定义了`autoescape`这一标记，还定义了`raw`filter。自行创建转移扩展时，可以打开或者关闭全局的输出转义策略：

	$escaper = new Twig_Extension_Escaper('html');
	$twig->addExtension($escaper);

如果设置为`html`，所有模板中的变量都会使用`html`转义策略进行转义，除非是使用了`raw`filter的输出：

	{{ article.to_html|raw }}
	
还可以使用`autoescape`标记改变转义模式。（参考[autoescape文档](http://twig.sensiolabs.org/doc/tags/autoescape.html)）：

	{% autoescape 'html' %}
	    {{ var }}
	    {{ var|raw }}      {# var 不会被转义 #}
	    {{ var|escape }}   {# var 不会被再次转义 #}
	{% endautoescape %}
	
> `autoescape`标记对包含的文件无效。

转义规则是这样实现的：

- 模板中的常量（整数、布尔、数组...）会直接当做变量或filter的参数，不会被自动转义：

	{{ "Twig<br />" }} {# won't be escaped #}
	
	{% set text = "Twig<br />" %}
	{{ text }} {# will be escaped #}
	
- 如果一个表达式的结果永远是一个常量，或者是一个被标记为安全的变量，那么也不会被自动转义：

	{{ foo ? "Twig<br />" : "<br />Twig" }} {# 不会被转义 #}
	
	{% set text = "Twig<br />" %}
	{{ foo ? text : "<br />Twig" }} {# 会被转义 #}
	
	{% set text = "Twig<br />" %}
	{{ foo ? text|raw : "<br />Twig" }} {# 不会被转义 #}
	
	{% set text = "Twig<br />" %}
	{{ foo ? text|escape : "<br />Twig" }} {# 不会被转义 #}
	
- 转义在所有Filter执行之后，在输出之前：

	{{ var|upper }} {# is equivalent to {{ var|upper|escape }} #}
	
- `raw` filter应该在filter链的最后一个来执行：

	{{ var|raw|upper }} {# 会被转义 #}
	
	{{ var|upper|raw }} {# 不会被转义 #}

- 如果最后一个filter在当前上下文中（例如html或者js）被标记为安全。`escape`和`escape('html')`被HTML标记为安全，`escape('js')`被JavaScript标记为安全，`raw`对所有内容都被标记为安全。

	{% autoescape 'js' %}
	    {{ var|escape('html') }} {# 会被js和html全部转义 #}
	    {{ var }} {# 会被JavaScript转义 #}
	    {{ var|escape('js') }} {# 不会被多次转义 #}
	{% endautoescape %}

> 注意自动转义是是在表达式运行之后发生，因此有局限性的。例如当进行连续操作时，`{{ foo|raw ~ bar }}`因为转义是最后发生的，因此无法给出期待的结果（这里的`raw` filter没有任何效果）。

###沙箱扩展

沙箱扩展用于运行不受信任的代码。会限制访问不安全的属性和方法。沙箱安全性由一个策略实例来管理。缺省情况下，Twig会带有一个策略类：`Twig_Sandbox_SecurityPolicy`。这个类允许把某些标记、Filter、属性以及方法放入白名单：

	$tags = array('if');
	$filters = array('upper');
	$methods = array(
	    'Article' => array('getTitle', 'getBody'),
	);
	$properties = array(
	    'Article' => array('title', 'body'),
	);
	$functions = array('range');
	$policy = new Twig_Sandbox_SecurityPolicy($tags, $filters, $methods, $properties, $functions);

在前面的配置中，安全策略只允许使用`if`标记，以及`upper`filter。另外，这个模板只能调用`Artical`对象的`getTitle()`以及`getBody()`方法，以及`title`和`body`两个公开属性。

策略对象是沙箱构造函数的第一个参数：

	$sandbox = new Twig_Extension_Sandbox($policy);
	$twig->addExtension($sandbox);
	
沙箱模式缺省是关闭的，当使用不受信的模板时，可以使用`sandbox`标记来激活沙箱：

	{% sandbox %}
	    {% include 'user.html' %}
	{% endsandbox %}

可以利用扩展构造器的第二个参数，如果传递为true，则为所有模板启用沙箱：

	$sandbox = new Twig_Extension_Sandbox($policy, true);
	
###优化扩展

`optimizer`扩展会在编译模板之前对节点树进行优化：

	$twig->addExtension(new Twig_Extension_Optimizer());
	
所有优化都缺省被打开。也可以在构造器中指定要启用的项目：

	$optimizer = new Twig_Extension_Optimizer(Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR);
	
	$twig->addExtension($optimizer);
	
Twig支持以下的优化：

- `Twig_NodeVisitor_Optimizer::OPTIMIZE_ALL`， 缺省值，启用所有优化。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_NONE`，禁止所有优化，会节约编译时间，不过会增加执行时间和内存消耗。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_FOR`， 尽可能减少`loop`变量的创建，以提高`for`标记的小吕。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_RAW_FILTER`，尽可能移除`raw` filter。
- `Twig_NodeVisitor_Optimizer::OPTIMIZE_VAR_ACCESS`，简化变量的创建和访问。

##异常

Twig能抛出的异常：

- `Twig_Error`：所以异常的基础。
- `Twig_Error_Syntax`：语法错误。
- `Twig_Error_Runtime`：运行时错误（当前实例中不包含某个filter）。
- `Twig_Error_Loader`：当模板载入过程出错时抛出。
- `Twig_Sandbox_SecurityError`: 沙箱中调用不被允许的标记、filter或者方法时出现。