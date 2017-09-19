## 8.28
 * 开启80跟3306端口 //centos7的防火墙改成了firewall,不再叫iptables
 
		firewall-cmd --zone=public --add-port=80/tcp --permanent 
		firewall-cmd --zone=public --add-port=3306/tcp --permanent 
		systemctl restart firewalld.service//重启防火墙
* 在防火墙配置文件/etc/firewalld/zones/public.xml添加以下内容，将80端口指定到httpd服务

		<rule family="ipv4">
	    	<source address="116.196.71.39" />
	    	<service name="httpd" />
	    	<accept />
 		</rule>

 * 安装mysql（yum）
 
		yum install mysql mysql-server 
		service mysqld restart //重启
		mysql -uroot -p //登录
 
 * 安装httpd（yum） //一直启动失败 ，原因是云服务器已有httpd服务 
	  
		kill -进程ID //杀死原进程 
		yum install httpd
		service httpd restart//重启

 * 安装php（yum）
 
		yum install php //版本为5.4，所以需要重新安装

* 配置httpd.conf 

		//添加
		AddType application/x-httpd-php .php .phtml .php3 .inc
		AddType application/x-httpd-php-source .phps
		------------------------------------------
		AddHandler cgi-script .cgi　修改为： AddHandler cgi-script .cgi .pl //允许扩展名为.pl的CGI脚本运行
		LoadModule rewrite_module modules/mod_rewrite.so //支持.htaccess
		AddDefaultCharset UTF-8　修改为：AddDefaultCharset GB2312　//添加GB2312为默认编码
		DirectoryIndex index.html index.html.var  修改为：  DirectoryIndex index.html index.htm Default.html Default.htm index.php Default.php index.html.var  //设置默认首页文件，增加index.php

* 配置php.ini

		magic_quotes_gpc = On                        #打开magic_quotes_gpc来防止SQL注入
		log_errors = On                                         #记录错误日志
		error_log = error_log.log   #设置错误日志存放目录
 
##8.29

* yum安装的apache/2.4.6不允许.htaccess（伪静态），导致laravel路由不可以正常使用
* 卸载后重新编译安装，现在apache不能正常打开php文件 //打开页面提示下载

* 安装gcc等必备程序包

		yum install -y gcc gcc-c++ make automake 

* 编译安装apache
		
		//所需资源，均从google下载
		1. httpd-2.4.4.tar.gz
		2. apr-1.4.6.tar.gz和apr-util-1.5.1.tar.gz
		3. pcre-8.32.tar.gz
		
		//先装gcc和make
		yum -y install gcc
		yum -y install make
		yum -y install gcc-c++ //没有这个gcc-c++一会编译不prce
		
		//安装apr:
		tar -zvxf apr-1.4.6.tar.gz
		cd apr-1.4.6
		./configure --prefix=/usr/local/apr
		make && make install
		
		//安装apr-util
		tar -zvxf apr-util-1.5.1.tar.gz
		cd apr-util-1.5.1
		./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
		make && make install
		
		//安装pcre
		tar -zvxf pcre-8.32.tar.gz
		cd pcre-8.32
		./configure
		make && make install

		//安装apache 一定要先装上面那三个不然编译不了
		tar -zvxf httpd-2.4.4.tar.gz
		cd httpd-2.4.4
		./configure --prefix=/usr/local/apache –with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util
		make && make install

		//将httpd添加为系统服务
		cp apachectl /etc/init.d/httpd
		/etc/init.d/

* 编译安装php

		//先安装依赖包libxml2 资源google

		tar zxvf libxml2-2.8.0.tar.gz 进入解压后的目录
		./configure --prefix=/usr/local/libxml2
		make && make install

		//安装PHP
		tar zxvf php-5.6.25.tar.gz
		./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-libxml-dir=/usr/local/libxml2 --with-apxs2=/usr/local/apache/bin/apxs --enable-inline-optimization --enable-shared --enable-opcache --enable-fpm  --enable-bcmath --enable-soap --enable-pcntl --enable-shmop --enable-sysvmsg --enable-sysvsem --enable-sysvshm --enable-sockets --enable-zip --with-mysql=mysqlnd --with-pdo-mysql=mysqlnd
		make && make install
	
		//安装成功之后
		cp php.ini-development /usr/local/php/etc/php.ini

		//配置apache
		LoadModule php5_module module/libphp5.so // apache/modules 文件夹里有libphp5.so说明php编译安装成功
		AddType application/x-httpd-php .php

* 添加open_ssl扩展

		//进入PHP的openssl扩展模块目录
		cd php-5.6.22/ext/openssl/
		
		/usr/loacl/php/bin/phpize //这里为你自己的phpize路径，如果找不到，使用whereis phpize查找
		
		// 执行后，发现错误 无法找到config.m4 ，config0.m4就是config.m4。直接重命名
		cp config0.m4 config.m4
		usr/local/php/bin/phpize
		./configure --with-openssl --with-php-config=/usr/local/php/bin/php-config
		make && make install

		//安装完成后，会返回一个.so文件（openssl.so）的目录。在此目录下把openssl.so 文件拷贝到你在php.ini 中指定的 extension_dir 下（在php.ini文件中查找：extension_dir =），我这里的目录是 var/www/php5/lib/php/extensions
		
		//编辑php.ini文件，在文件最后添加
		extension=openssl.so

* 添加curl扩展

		cd php-5.6.22/ext/openssl/ //进入扩展目录
		/usr/local/php/bin/phpize  // 运行phpize
		./configure -with-curl=/usr/local/curl --with-php-config=/usr/local/php/bin/php-config
		make && make install //编译安装
		
		//修改配置文件
		extension=curl.so 

* 安装memcache扩展

		1、安装libmemcached库
     	 yum install libmemcached
		2、下载并解压memcache文件
     	wget http://pecl.php.net/get/memcache-3.0.8.tgz  
     	tar xzvf memcache-3.0.8.tgz  
     	cd memcache-3.0.8
		3、执行phpize扩展安装程序，假设phpzie的路径为/usr/local/php/bin/phpize,具体的路径得根据自己的环境修改。

		4、开始安装扩展memcache（基于PHP）
		./configure --enable-memcache --with-php-config=/usr/local/php/bin/php-config --with-zlib-dir  
		make && make install  
 
		//安装完成后，提示
		   
		Build complete.
		Don't forget to run 'make test'.
		Installing shared extensions:     /usr/local/php/lib/php/extensions/no-debug-non-zts-20100525/、

		5.安装memcached服务（基于linux系统）
		yum -y install memcached
		service memcached restart //重启
		chkconfig memcached on //开机启动
		memcached -d -m 100 -u root -l 127.0.0.1 -p 11211 -c 512 -P /tmp/memcached.pid //连接memcache服务
 
		6、最后修改php.ini文件，
		extension=memcache.so

##9.4


* mysql-5.7.19 不能编译安装，很容易内存不足 只能二进制安装

>资源：mysql-5.7.19-linux-glibc2.12-x86_64.tar.gz，
>步骤百度 Google

//安装完后把不需要把配置文件复制到/etc/my.cnf，否则报错

1. 下载 mysql57-community-release-el7-8.noarch.rpm 的 yum 源：

		wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm

2. 安装 mysql57-community-release-el7-8.noarch.rpm：

		rpm -ivh mysql57-community-release-el7-8.noarch.rpm

3. 安装 MySQL：

		yum -y install mysql mysql-server mysql-devel 

		//安装完毕后，完成MySQL的重启后会在 /var/log/mysqld.log 文件中会自动生成一个随机的密码。 

	可能出现的错误：

		Failed to restart mysqld.service: Unit not found.//意思找不到mariadb这个服务。之所以找不到，是因为mariadb的安装本身就没有完成，执行以下命令，查看mariadb的依赖情况：
		$ sudo yum search mariadb 
		$ sudo yum list |grep mariadb //查看已安装的 缺哪个就装哪个
		
		或者执行以下，全部安装
		yum install mariadb-embedded mariadb-libs mariadb-bench mariadb mariadb-sever

		yum install mariadb* //注意后面的‘*’号


	重启代码：

		service mysqld restart

4. 登录 MySQL 服务端并更新用户 root 的密码：

		mysql -u root -p

	完成后会提示输入密码，输入原始密码即可，打印出 MySQL 的版本号即表明已登录。 
	更新 MySQL 的用户 root的密码：

		SET PASSWORD FOR 'root'@'localhost' = PASSWORD('secret_password') 
	可能出现的错误，错误是由于你曾经升级过数据库，升级完后没有使用
	mysql_upgrade升级数据结构造成的。：

	ERROR 3009 (HY000): Column count of mysql.user is wrong. Expected 45, found 43. Created with MySQL 5
	
	解决：mysql_upgrade -u root -p //日志可以查看初始密码


	你想myuser使用mypassword从任何主机连接到mysql服务器的话。

		GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION; //记得修改语句中的“myuser”和“mypassword”

	刷新权限使之生效：

		flush privileges;
			
5.现在的密码：Jiawei520. //满足安全条件

	

6.数据库配置

		skip-grant-tables//忘记密码时需要
		bind-address=0.0.0.0 //允许其他ip远程访问3306端口
		

#####我日你妈的黄子韬，阿里云安全组没配置3306访问规则，导致不能远程访问，快崩溃了；
	netstat -talnp--查看端口

	//几个重要目录
		数据库目录
		/var/lib/mysql/
		配置文件
		/etc/my.cnf
		日志
		/var/log/mysql.log


##9.19

* 安装nginx

		//先安装pcre 跟 zlib的包
		yum -y install pcre* zlib* #或者进行编译安装
		//编译安装nginx，下载地址：http://nginx.org/en/download.html 此次安装为最新稳定版nginx-1.8.0
		或者
		// wget http://nginx.org/download/nginx-1.4.2.tar.gz   #1.4版本
		./configure --user=nginx --group=nginx --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
		make && make install
		//启动
		/usr/local/nginx/sbin/nginx
		//配置文件
		/usr/local/nginx/conf/nginx.conf

		//解析php文件
		修改配置文件
		yum install php-fpm //安装php-fpm服务