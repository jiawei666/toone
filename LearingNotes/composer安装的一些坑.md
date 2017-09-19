* 官网下载composer.exe安装，有个步骤需要指定php.exe
>一般可以正常运行，如果不行则需要在系统环境变量将php跟composer设置

* 装完后用wox打开cmd窗口，composer命令没反应（坑）
* 需要在菜单运行cmd窗口
* 使用国内镜像资源
		
		composer config -g repo.packagist composer https://packagist.phpcomposer.com  


* Laravel 框架有一些系统上的需求：

		PHP 版本 >= 5.4
		Mcrypt PHP 扩展
		OpenSSL PHP 扩展
		Mbstring PHP 扩展
		Tokenizer PHP 扩展
* 输入命令安装laravel
		
		composer global require "laravel/installer=~1.1"

* 一切正常则可用laravel命令创建项目目录(当前目录)

		laravel new player 