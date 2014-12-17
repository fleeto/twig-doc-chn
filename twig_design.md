#模板设计人员的Twig

本文描述了模板引擎中涉及到的语法和语义，对Twig模板的设计会很有帮助。

##概要


模板是一个简单的文本文件。它能够生成任何文本格式（HTML, XML, CSV, LaTeX等）。他没有固定的扩展名，`html`、`xml`都没关系。

模板中包含`变量`和`表达式`在模板被处理时会被替换为真实的值，`tags`则对模板的逻辑进行控制。

下面用一个小例子展示一些基础内容，当然，后面会做进一步的深入。

	<!DOCTYPE html>
	<html>
	    <head>
	        <title>My Webpage</title>
	    </head>
	    <body>
	        <ul id="navigation">
	        {% for item in navigation %}
	            <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
	        {% endfor %}
	        </ul>
	
	        <h1>My Webpage</h1>
	        {{ a_variable }}
	    </body>
	</html>
	
上面代码中有两种分隔符：`{% ... %}`和`{{ ... }}`。第一种用于执行for循环之类的语句，第二种用于输出表达式结果。

##IDE集成

很多IDE为Twig提供了语法高亮和自动完成支持：

- *Textmate*：[Twig bundle](https://github.com/Anomareh/PHP-Twig.tmbundle)
- *Vim*：[Jinja syntax plugin](http://jinja.pocoo.org/docs/integration/#vim)或[vim-twig插件](https://github.com/evidens/vim-twig)
- *Netbeans*：[Twig语法插件](http://plugins.netbeans.org/plugin/37069/php-twig)（7.1版之前，7.2开始集成）。
- *PhpStorm*：2.1开始集成。
- *Eclipse*：[Twig插件](https://github.com/pulse00/Twig-Eclipse-Plugin)
- *Sublime Text*：[Twig Bundle](https://github.com/Anomareh/PHP-Twig.tmbundle)
- *GtkSourceView*：[Twig语言定义](https://github.com/gabrielcorpse/gedit-twig-template-language) (gedit和其他项目试用了该库)
- *Coda*和*SubEthaEdit*：[Twig Sytax](https://github.com/bobthecow/Twig-HTML.mode)
- *Coda 2*：[Twig语法模式](https://github.com/muxx/Twig-HTML.mode)
- *Komodo*和*Komodo Edit*：Twig高亮/语法检查
- *Notepad++*：[Notepad++ Twig高亮](https://github.com/Banane9/notepadplusplus-twig)
- *Emacs*：[web-mode.el](http://web-mode.org/)
- *Atom*：[PHP-twig for atom](https://github.com/reesef/php-twig)

##变量

应用会把变量传递给模板进行处理。变量自身也可以拥有属性或元素供外部访问。变量的具体形态取决于提供变量的应用。

可以利用句点（.）来访问变量的属性（相当于PHP对象的属性或方法，或者PHP数组的成员），或者也可以使用下标语法（[]）：

	{{ foo.bar }}
	{{ foo['bar'] }}

当属性名字中包含了特别字符（例如“-”会被当做减号处理）的时候，可以使用`attribute`函数来访问变量的属性：

	{# 等价于不能工作的：foo.data-foo #}
	{{ attribute(foo, 'data-foo') }}

> 大括号是打印指令，而不是变量的组成部分。当在tag中访问变量的时候，不要在Tag中访问变量的时候使用这种语法。

在`strict_variables`选项为false的时候，如果变量或属性不存在，会返回null；而如果该选项设置为true，这种情况就会抛出错误（参考[环境选项](http://twig.sensiolabs.org/doc/api.html#environment-options)）。

> ###实现
> foo.bar在PHP这一层次做了如下事情：
> 
> - 检查foo是不是数组，bar是不是其中的一个有效元素；
> - 如果不是，且foo是一个对象，检查bar是不是有效的属性；
> - 如果不是，且foo是一个对象，检查bar是不是有效的方法（即使bar是构造器——所以请使用__construct()）；
> - 如果不是，且foo是一个对象，检查getBar是不是有效的方法；
> - 如果不是，且foo是一个对象，检查isBar是不是有效的方法；
> - 如果都不是，返回null。
> 
> 另一方面，foo['bar']只对PHP数组工作
> - 检查foo是不是一个数组，bar是不是一个有效元素；
> - 如果不是，返回null。

*如果要访问变量的动态属性，应该使用[attribute()函数](http://twig.sensiolabs.org/doc/functions/attribute.html)。*

##全局变量

模板中随时可以使用如下的变量：

- _self：当前的模板；
- _context：当前的上下文；
- _charset：当前字符集。

##设置变量

可以在代码中为变量赋值。赋值过程使用[set](http://twig.sensiolabs.org/doc/tags/set.html)tag完成：

	{% set foo = 'foo' %}
	{% set foo = [1, 2] %}
	{% set foo = {'foo': 'bar'} %}

##Filters

变量可以用`filters`进行修改。Filters和变量之间用管道符（|）进行分隔，可以用括号来包含参数。多个Filter可能进行链式调用——前一个Filter的输出可作为下一个Filter的输入。

下面的例子清理所有的HTML标记，并做标题大写处理：

	{{ name|striptags|title }}
	
Filter可以用括号的形式接收参数。这个例子用逗号连接一个列表：

	{{ list|join(', ') }}

要对一段代码使用Filter，可以用[filter tag](http://twig.sensiolabs.org/doc/tags/filter.html)操作：

	{% filter upper %}
	    This text becomes uppercase
	{% endfilter %}
	
可以前往[Filter页面](http://twig.sensiolabs.org/doc/filters/index.html)，获取更多内置Filter的信息。

##函数

调用函数可以生成内容。调用函数的方式是：函数名称加括号，括号中可能包含参数。

例如`range`函数返回一个递增的整数列表：

	{% for i in range(0, 3) %}
	    {{ i }},
	{% endfor %}
	
前往[functions](http://twig.sensiolabs.org/doc/functions/index.html)一节可以学习更多内置函数的内容。

##具名参数

1.12版中新增

	{% for i in range(low=1, high=10, step=2) %}
	    {{ i }},
	{% endfor %}
	
有了具名参数，传递给模板的参数可以更加明确，让模板更加易用。


	{{ data|convert_encoding('UTF-8', 'iso-2022-jp') }}
	
	{# 相比： #}
	
	{{ data|convert_encoding(from='iso-2022-jp', to='UTF-8') }}
	
具名参数还能在不传递某些参数时候直接跳过：

	{# 第一个参数是日期格式，如果赋值为null，则采用标准日期格式 #}
	{{ "now"|date(null, "Europe/Paris") }}
	
	{# 可以利用具名参数跳过第一个参数： #}
	{{ "now"|date(timezone="Europe/Paris") }}
	
还可以用顺序参数和具名参数夹杂的方式进行调用，这里要保证顺序参数必须比具名参数先出现：

	{{ "now"|date('d/m/Y H:i', timezone="Europe/Paris") }}
	
>每个函数和Filter的文档中，如果该函数或者Filter支持具名参数，都会有一节列出所有的参数名称。

##控制结构

控制结构指的是所有能够控制程序流程的元素——条件判断（也就是if/elseif/else）、for循环，以及语句块等。控制结构用`{% ... %}`包装。

例如，要显示一个users变量中所有的列表成员，使用[for](http://twig.sensiolabs.org/doc/tags/for.html)：

	<h1>Members</h1>
	<ul>
	    {% for user in users %}
	        <li>{{ user.username|e }}</li>
	    {% endfor %}
	</ul>

可以用[if](http://twig.sensiolabs.org/doc/tags/if.html)进行条件判断：

	{% if users|length > 0 %}
	    <ul>
	        {% for user in users %}
	            <li>{{ user.username|e }}</li>
	        {% endfor %}
	    </ul>
	{% endif %}
	
[Tags](http://twig.sensiolabs.org/doc/tags/index.html)页中描述了各种内置Tag。

##注释

在模板中进行注释，使用注释语法`{# ... #}。这对于调试和为自己和其他设计者提供信息是很有帮助的：

	{# 注：禁用这一块东西，不再需要了
	    {% for user in users %}
	        ...
	    {% endfor %}
	#}
	
##包含其他模板

[Include Tag]用于包含一个模板，并把该模板的渲染结果返回到本页面：

	{% include 'sidebar.html' %}

缺省状况下，当前的上下文会被传递给被包含的模板。

传递给被包含模板中的上下文包含当前模板中定义的变量：

	{% for box in boxes %}
	    {% include "render_box.html" %}
	{% endfor %}
	
这个例子中，`render_box.html`可以访问box变量。

模板文件名的选取，取决于模板加载器。比如`Twig_Loader_Filesystem`允许通过文件名访问其他的模板。可以使用子目录来访问其他模板：

	{% include "sections/articles/sidebar.html" %}
	
这一行为依赖于调用Twig的应用。

##模板继承

Twig最强大的能力就是继承。这一个能力可以让你建立一个包含所有基础元素的基础模板，并定义可以被子模板覆盖的`block`。

这部分内容并不像听起来那么复杂。下面的例子能让你更好的理解这个概念。

我们创建一个名为`base.html`的基本模板，在其中定义了一个简单的HTML结构，后面可以用在一个简单的两列布局的页面上：

	<!DOCTYPE html>
	<html>
	    <head>
	        {% block head %}
	            <link rel="stylesheet" href="style.css" />
	            <title>{% block title %}{% endblock %} - My Webpage</title>
	        {% endblock %}
	    </head>
	    <body>
	        <div id="content">{% block content %}{% endblock %}</div>
	        <div id="footer">
	            {% block footer %}
	                &copy; Copyright 2011 by <a href="http://domain.invalid/">you</a>.
	            {% endblock %}
	        </div>
	    </body>
	</html>
	
在这个例子中，[Block标记](http://twig.sensiolabs.org/doc/tags/block.html)定义了四个Block，子模板可以对这四个Block进行填充。所有的Block标记的作用就在于告诉模板引擎——子模板可能对模板的这一部分进行覆盖。

一个子模板看起来大致如此：

	{% extends "base.html" %}
	
	{% block title %}Index{% endblock %}
	{% block head %}
	    {{ parent() }}
	    <style type="text/css">
	        .important { color: #336699; }
	    </style>
	{% endblock %}
	{% block content %}
	    <h1>Index</h1>
	    <p class="important">
	        Welcome to my awesome homepage.
	    </p>
	{% endblock %}

此处的关键就在于[Extends tag](http://twig.sensiolabs.org/doc/tags/extends.html)。他让模板引擎得知这个模板是“扩展自其他模板”。当模板系统处理这一模板的时候，就会首先定位父模板。Extends标记应该是子模板的第一个标记。

另外因为子模板中没有定义`footer` block，所以此处会继续使用父模板的内容。

也可以用[Parent函数](http://twig.sensiolabs.org/doc/functions/parent.html)来渲染父模板Block的内容。这一调用会返回父模板中的Block结果：

	{% block sidebar %}
	    <h3>Table Of Contents</h3>
	    ...
	    {{ parent() }}
	{% endblock %}
	
> [extend文档](http://twig.sensiolabs.org/doc/tags/extends.html)中介绍了更深入的特性，例如Block嵌套、作用域、动态继承以及条件继承等。


*Twig还支持多重继承，这种关系被称为水平重用，这一特性使用[use tag](http://twig.sensiolabs.org/doc/tags/use.html)来完成。这一技巧非常深入，在正常的模板中很难用到。*


##HTML转义

从模板中生成HTML的时候，存在一种风险：一个变量可能包含了HTML代码，会影响到结果中的HTML。有两种方式来解决这个问题：人工转义每一个变量，或者缺省情况下自动转义所有变量。Twig对两种方式都支持，缺省开启第二种。

> 自动转义只有在`escaper`扩展激活的情况下才生效。

###人工转义

如果启用了人工转义，那么对需要进行处理的变量进行转义就成了开发者的责任。那么什么需要被转义呢？答案是所有不信任的变量。人工转义的方法是把变量传送给[escape filter](http://twig.sensiolabs.org/doc/filters/escape.html)或者e filter进行处理。

	{{ user.username|e }}

缺省情况下，escape filter使用html策略，不过针对不同的需求，也可以显式的指定其他的可用策略：

	{{ user.username|e('js') }}
	{{ user.username|e('css') }}
	{{ user.username|e('url') }}
	{{ user.username|e('html_attr') }}
	
###自动转义

不管是否启用了自动转义，都可以利用[autoescape标记](http://twig.sensiolabs.org/doc/tags/autoescape.html)来标记模板中的一段内容。

	{% autoescape %}
	    Everything will be automatically escaped in this block (using the HTML strategy)
	{% endautoescape %}
	
缺省情况下，自动转义同样使用的html策略。如果输出内容属于其他上下文，则需要显式的指定其他的转义策略：

	{% autoescape 'js' %}
	    Everything will be automatically escaped in this block (using the JS strategy)
	{% endautoescape %}
	
##转义

让Twig能够忽略一部分本应当做是block或者变量来处理的东西，是个常见甚至必要的需求。例如想要以原始形态输出`{{`这样被语法占用的关键字，而不想被当做变量的开始，就可以使用一个小技巧。

最简单的方式就是使用一个变量表达式：

	{{ '{{' }}

如果是比较大段的内容，可以标记一个Block为[verbatim](http://twig.sensiolabs.org/doc/tags/verbatim.html)

##宏

在版本1.12中出现：支持缺省参数值。

宏和和其他语言中的函数类似。它可以将重复的HTML碎片组织起来进行重用，免去重复黏贴之苦。

使用[macro标记](http://twig.sensiolabs.org/doc/tags/macro.html)可以定义宏。下面是`forms.html`中的一个小例子：利用宏渲染一个form元素：

	{% macro input(name, value, type, size) %}
	    <input type="{{ type|default('text') }}" name="{{ name }}" value="{{ value|e }}" size="{{ size|default(20) }}" />
	{% endmacro %}
	
可以在任何模板中定义宏，然后使用[import标记](http://twig.sensiolabs.org/doc/tags/import.html)进行导入即可：

	{% import "forms.html" as forms %}
	
	<p>{{ forms.input('username') }}</p>

在从其他模板中进行引入的时候，还可以利用[From标记](http://twig.sensiolabs.org/doc/tags/from.html)为其赋予别名：

	{% from 'forms.html' import input as input_field %}
	
	<dl>
	    <dt>Username</dt>
	    <dd>{{ input_field('username') }}</dd>
	    <dt>Password</dt>
	    <dd>{{ input_field('password', '', 'password') }}</dd>
	</dl>
	
还可以在为宏指定缺省参数：

	{% macro input(name, value = "", type = "text", size = 20) %}
	    <input type="{{ type }}" name="{{ name }}" value="{{ value|e }}" size="{{ size }}" />
	{% endmacro %}
	
##表达式

Twig允许使用表达式，工作方式类似普通的PHP，即使你对PHP没那么熟悉，也可以轻松面对。

> 运算符优先级按递增顺序排列如下：`b-and, b-xor, b-or, or, and, ==, !=, <, >, >=, <=, in, matches, starts with, ends with, .., +, -, ~, *, /, //, %, is, `, |, [], .`

	{% set greeting = 'Hello ' %}
	{% set name = 'Fabien' %}
	
	{{ greeting ~ name|lower }}   {# Hello fabien #}
	
	{# use parenthesis to change precedence #}
	{{ (greeting ~ name)|lower }} {# hello fabien #}

##常量

在1.5中新增：支持以名称和表达式作为哈希键，在1.5中加入。

表达式的最简单形式就是常量了。他是PHP类型的一种实现，包含字符串、数字以及数组。以下是一些例子

- "Hello world"：单引号或双引号之间的内容是字符串。可以在任何需要字符串的情况下使用（例如作为函数或者Filter的参数，或者在extend以及include过程中使用）。
- 42 / 42.23：整数和浮点数。有小数点的是浮点数；否则就是整数。
- ["foo", "bar"]：数组由方括号包含的一组逗号分隔的表达式组成。
- {"foo": "bar"}：同数组类似，不过是以花括号进行包装。

	{# 字符串为键 #}
	{ 'foo': 'foo', 'bar': 'bar' }
	
	{# 同前一个等价 —— Twig 1.5加入 #}
	{ foo: 'foo', bar: 'bar' }
	
	{# 以整数为键 #}
	{ 2: 'foo', 4: 'bar' }
	
	{# 以表达式为键 #}
	{ (1 + 1): 'foo', (a ~ 'b'): 'bar' }
	
- true / false：你懂的，不解释。
- null：表明无特定值。当访问一个不存在的变量时，就会返回null，none是null的别名。


数组和哈希可以嵌套：

	{% set foo = [1, {"foo": "bar"}] %}

> 字符串使用单双引号对性能没什么影响，但是只有双引号才支持字符串差值。

##算数

Twig可以进行运算。这个功能很少用得到，不过为了完整起见，还是提供了这个功能。目前支持下列操作符：

- `+`：把两个对象加到一起（操作数会被转换为数字）。`{{1 + 1}}` 等于2。
- `-`：从第一个操作数中减去第二个。`{{ 3 - 2}}` 等于1。
- `/`：两个数相除。结果是一个浮点数。`{{ 1 / 2 }}` 等于 `{{ 0.5 }}`。
- `%`：求余。`{{ 11 % 7 }}` 等于 4.
- `//`：整除，返回两个数相除得到的结果的整数部分。`{{ 20 // 7 }}`等于2，`{{ -20 // 7}}`等于-3(这只是[round filter](http://twig.sensiolabs.org/doc/filters/round.html)的变形)。
- `\`*：相乘，`{{2 * 2}}` 返回 4。
- `\*\`*：乘方，`{{ 2 ` 3}}`等于8。

##逻辑操作

可以用下面的操作符连接多个表达式：

- `and`：如果左右表达式都返回true，则返回true。
- `or`：如果两个表达式中有一个为true，则返回true。
- `not`：否定词。
- `(表达式)`：为表达式分组。

> Twig还支持位操作：（b-and, b-xor 以及 b-or）。

##比较操作
为表达式的比较提供了下列操作符： `==`, `!=`, `<`, `>`, `>=`, 以及 `<=`。
还可以对字符串使用`starts with`或者`end with`：

	{% if 'Fabien' starts with 'F' %}
	{% endif %}
	
	{% if 'Fabien' ends with 'n' %}
	
> 对于复杂的比较，`matches`操作符提供了[正则表达式](http://php.net/manual/en/pcre.pattern.php)匹配的能力：

	{% if phone matches '/^[\\d\\.]+$/' %}
	{% endif %}

##包含操作符

操作符`in`提供了包含测试功能。

如果右侧操作数包含左侧的操作数，则返回true：

	{# 返回 true #}
	
	{{ 1 in [1, 2, 3] }}
	
	{{ 'cd' in 'abcde' }} 	  

> 可以使用这个filter对字符串、数组或者实现了`Traversable`接口的对象进行包含测试。

要进行不包含的测试，只须使用`not`操作符：
	
	{% if 1 not in [1, 2, 3] %}
	
	{# 等价于 #}
	{% if not (1 in [1, 2, 3]) %}
	
##测试操作符

`is`操作符用于测试。可以用于测试一个变量和表达式，右侧的操作数就是测试的名字：

	{# 检查是否奇数 #}
	
	{{ name is odd }}
	
测试也可以接受参数：

	{% if post.status is constant('Post::PUBLISHED') %}

可以使用`is not`来进行否定测试：

	{% if post.status is not constant('Post::PUBLISHED') %}
	
	{# 等价于 #}
	{% if not (post.status is constant('Post::PUBLISHED')) %}
	
可以去[test](http://twig.sensiolabs.org/doc/tests/index.html)页面来了解一下内置的测试。

##其他操作符

1.12.0加入：1.12.0中加入了扩展的三元操作符。

下列操作符非常有用，但是不属于上文中的任何一类：

- `..:`：以左右两个操作数为基础生成一个序列（[range函数](http://twig.sensiolabs.org/doc/functions/range.html)的变形）。
- `|:`：Filter。
- `~`：把所有操作数转换为字符串并连接起来。`{{ "Hello " ~ name ~ "!"  }}`会返回(假设`name`是`'John'`)`Hello John!`。
- `.,[]`：获取对象的属性。
- `?:`：三元操作符：

	{{ foo ? 'yes' : 'no' }}
	
	{# Twig 1.12.0中： #}
	{{ foo ?: 'no' }} is the same as {{ foo ? foo : 'no' }}
	{{ foo ? 'yes' }} is the same as {{ foo ? 'yes' : '' }}

##字符串插值

1.5中新增：字符串插值在Twig1.5中加入。

字符串差值(#{表达式})允许在**双引号**内的字符串中插入任何有效的表达式。表达式的结果会出现在最终的字符串中：

	{{ "foo #{bar} baz" }}
	{{ "foo #{1 + 2} baz" }}

##空白控制

1.1中新增：Tag级别的空白控制。

跟PHP类似，模板标记后面的第一个空行会被自动删除。模板引擎目前已经不再自动清理空白，所有空白（空格、制表符、换行符等）都会原样输出。

使用`spaceless`标记可以移除HTML标记之间的空白：

	{% spaceless %}
	    <div>
	        <strong>foo bar</strong>
	    </div>
	{% endspaceless %}
	
	{# 输出： <div><strong>foo bar</strong></div> #}

在`spaceless`标记之外，还可以在每个标记的级别控制空白。在标记上使用空白控制修饰符，可以清除前导或跟随的空白：

	{% set value = 'no spaces' %}
	{#- 没有前导或后续空白 -#}
	{%- if true -%}
	    {{- value -}}
	{%- endif -%}
	
	{# 输出 'no spaces' #}
	
上面的例子演示了缺省的空白控制修饰符，以及如何移除标记周围的空白。还可以只清除Tag一边的空白：

	{% set value = 'no spaces' %}
	<li>    {{- value }}    </li>
	
	{# 输出 '<li>no spaces    </li>' #}
	
##扩展

Twig的扩展很方便。

如果你要查找新的标记、filter或者函数，可以浏览官方的[扩展仓库](http://github.com/twigphp/Twig-extensions)。

如果想要创建自己的扩展，请阅读[创建扩展章节](http://twig.sensiolabs.org/doc/advanced_legacy.html#creating-extensions)。