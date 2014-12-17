#安装

Twig有多种安装方式。

##PHP包方式安装Twig（推荐）

安装[Composer](https://getcomposer.org/download/)并运行下面的命令，获得最新版本：

	composer require twig/twig:~1.0

##安装最新的压缩包

- 从[下载页面](https://github.com/twigphp/Twig/tags)获取最新的压缩包。
- [验证](http://fabien.potencier.org/article/73/signing-project-releases)压缩包的完整性。
- 解压缩。
- 把文件移动到你的项目中。


##安装开发版本

	git clone git://github.com/twigphp/Twig.git

##安装PEAR包

> Twig的PEAR方式已经被放弃，PEAR上的Twig最新版本是1.15.1，请使用Composer代替PEAR。

	pear channel-discover pear.twig-project.org
	pear install twig/Twig

##安装C扩展

*C扩展在Twig 1.4时加入。*

> C扩展是可选的，但是这一扩展加入了很多性能方面的改进，可以在生产环境使用。

Twig有一个能够增强Twig引擎运行性能的C扩展，跟其他PHP扩展使用同样的安装方式：

	cd ext/twig
	phpize
	./configure
	make
	make install

> 还可以用PEAR方式安装C扩展(注意，这一方式已经废弃，Twig已经不再为PEAR提供新版本)

	pear channel-discover pear.twig-project.org
	pear install twig/CTwig
	
Windows用户：

- 遵照[PHP文档](https://wiki.php.net/internals/windows/stepbystepbuild)设置Build环境。
- 把Twig的C扩展源码放入：```C:\php-sdk\phpdev\vcXX\x86\php-source-directory\ext\twig```
- 在第14步中使用```configure --disable-all --enable-cli --enable-twig=shared```。
- ```nmake```
- 拷贝```C:\php-sdk\phpdev\vcXX\x86\php-source-directory\Release_TS\php_twig.dll```到你的PHP环境中。

> 在Windows下的ZendServer上，ZTS并没有像[Zend Server FAQ](http://www.zend.com/en/products/server/faq#faqD6)中提到那样被激活。
> 要使用```configure --disable-all --disable-zts --enable-cli --enable-twig=shared```来为ZendServer生成twig的C扩展。
> 生成的DLL会保存在```C:\\php-sdk\\phpdev\\vcXX\\x86\\php-source-directory\\Release```

最后，在php.ini中启用扩展：

	extension=twig.so #For Unix systems
	extension=php_twig.dll #For Windows systems
	
现在Twig会利用C扩展自动编译你的模板。注意，这个扩展不会替换PHP代码，只是提供了一个```Twig_Template::getAttribute()```方法的优化版本。