#简介

Twig是一个高效、安全并可扩展的PHP模板引擎。

如果你具有Smarty, Django或者Jinja这养的基于文本的模板语言的经验，那么你对Twig会感觉一见如故。在遵循PHP理念的基础之上，提供了具备强大功能的模板环境，不论是对开发者还是设计者来说，这一引擎都具有良好的可用性。

关键特性：

- 快速：Twig把模板编译成为优化后的PHP代码。相对普通PHP代码来说，其额外开销非常轻微。
- 安全：Twig会使用一个沙箱模式来运行不信任的模板代码。这使得Twig可以在用户可以修改模板设计的应用中工作良好。
- 弹性：Twig试用了弹性的词法和语法分析器。开发者可以定义自己的标记和过滤，并创建自己的DSL。

##需求

Twig需要的最小运行环境为**PHP 5.2.4**。

##安装

推荐使用Composer安装Twig：

	composer require "twig/twig:~1.0"

> 要了解其他的安装方式，可以阅读[安装](installation.md)一节，其中还包含了Twig C扩展的安装方式。

##基础API用法

本节对Twig PHP API做一个简单介绍。

```twig

	require_once '/path/to/vendor/autoload.php';
	
	$loader = new Twig_Loader_Array(
	    'index' => 'Hello {{ name }}!',
	);
	$twig = new Twig_Environment($loader);
	
	echo $twig->render('index', array('name' => 'Fabien'));

Twig使用一个**加载器**(`Twig_Loader_Array`)来定位模板文件，使用一个**环境**(Twig_Environment)来存储配置。
`render()`方法的第一个参数指定了模板，后面的参数则为渲染提供了变量。
模板一般是保存在文件系统中，所以Twig提供了一个文件系统加载器：

	$loader = new Twig_Loader_Filesystem('/path/to/templates');
	$twig = new Twig_Environment($loader, array(
	    'cache' => '/path/to/compilation_cache',
	));
	
	echo $twig->render('index.html', array('name' => 'Fabien'));

> 如果没有使用Composer，可以使用Twig的内置加载器：

 	require_once '/path/to/lib/Twig/Autoloader.php';
	Twig_Autoloader::register();
