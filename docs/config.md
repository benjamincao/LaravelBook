
 集成开发环境配置  
===============================

* [1. 本地环境设置](#localConfigig)
	* [1.1 编辑器](#sublimetext)
	* [1.2 php](#php)
	* [1.3 composer](#composer)
	* [1.4 putty](#putty)
* [2. Homestead 相关软件安装](#homesteadSoftware)
    * [2.1 vagrant](#vagrant)
    * [2.2 virtualbox](#virtualbox)
    * [2.3 homestead](#homestead)
    * [2.4 Git](#git)
* [3. 环境启动](#startup)
    * [3.1 使用vagrant添加homestead镜像文件](#vagrantImage)
    * [3.2 生成rsa key](#keygen)
    * [3.3 配置并启动homestead](#configHomestead)
    * [3.4 登录说明](#login)
    	* [3.4.1 ssh登录  ](#sshLogin)
    	* [3.4 1 mysql密码](#mysqlLogin)
* [4. 命令详解](#command)
	* [4.1 homestead命令](#homesteadCommand)
   
* * *


Laravel的目的就是让开发这的开发体验更愉快，开发过程更加简单。在环境搭建上也是如此。    
Laravel homestead是一个官方发布的vagrant封装包——一个虚拟机——提供完备的开发环境。在homestead配置完成后，不需要在本地环境中配置任何的开发工具，vagrant可以搞定一切。因为是虚拟机，不会对本地系统产生任何的破坏。同时如果出现故障，可以在几分钟内销毁并重建虚拟机。  
homestead是跨平台的，可以在windows，Linux，Mac上运行，其中集成了nginx，php5.6，MySql，postgres，Redis，Memcached，HHVM等所有开发laravel所需的软件。   
本文档是针对windows环境下homestead的配置运行。


<h2 id="localConfig">1. 本地环境设置</h2>

<h3 id="sublimetext">1.1 编辑器</h3>
建议使用[sublime Text 3](http://www.sublimetext.com/3)，会有单独文档来说明如何使用Sublime Text高效开发php。


<h3 id="php">1.2 php</h3>
运行composer必须要有PHP，建议使用最新版本的发行版php，目前最新版本5.6.8。php windows版本分为32位和64位的，64位目前仍处于试验阶段，我们选择稳定的32位版本。其中又分为thread safe和not thread safe版本，在web开发中我们选择not thread safe版本，点击[这里](http://windows.php.net/downloads/releases/php-5.6.8-nts-Win32-VC11-x86.zip)下载。   
**注意 php windows版本使用VC11编译，系统需要有Visual C++ Redistributable for Visual Studio 2012 x86 or x64，如果系统没有安装这个包，请点击[这里](http://www.microsoft.com/en-us/download/details.aspx?id=30679)下载安装。**  
（1） 下载压缩包解压到C盘根目录  
（2） 配置环境变量，添加C:\php-5.6.8-nts-Win32-VC11-x86到PATH。  
（3） 修改php配置。拷贝php.ini-development 为php.ini。去`extension=php_openssl.dll`（composer需要） `extension=php_mbstring.dll`（laravel需要）前的注释。找到`; extension_dir = "ext"`，修改为`extension_dir = "C:\php-5.6.8-nts-Win32-VC11-x86\ext"`。   
（4）打开console，运行`php -version`确认php安装成功。  

	C:\>php -version
	PHP 5.6.8 (cli) (built: Apr 15 2015 15:07:05)  
	Copyright (c) 1997-2015 The PHP Group  
	Zend Engine v2.6.0, Copyright (c) 1998-2015 Zend Technologies  


<h3 id="composer">1.3 composer</h3>
点击[这里](https://getcomposer.org/Composer-Setup.exe)下载。     
安装完成后，打开console，运行`composer --version`确认安装成功。  

	C:\>composer  --version
	Composer version 1.0-dev (bc45d9185513575434021527d7756420e9f4b2cf) 2015-05-11 14:49:39   

composer默认会从·http://packagist.org/· 下载依赖包，速度比较慢，可以采用国内的镜像包来替代源。  
composer的全局配置文件位于 `C:\Users\benjamincao\AppData\Roaming\Composer\composer.json`，两个速度比较快的可选的源配置如下：

```javascript
{
	"repositories":[
		{
			"type":"composer",
			"url":"https://toran.reimu.io/repo/packagist/"
		},
		{
			"packagist":false
		}
	]
}
```

或者  
	
```javascript
{
	"repositories": [
		{
			"type": "composer", 
			"url": "http://comproxy.cn/repo/packagist"
		},
		{
			"packagist": false
		}
	]
}
```

**在开发laravel项目时，也可以在当前项目的composer.json文件中添加这些国内镜像，可以加快对包的更新**

<h3 id="putty">1.4 putty</h3>
putty是免费的ssh客户端，登录homestead虚拟机的利器。  
点击[这里](http://the.earth.li/~sgtatham/putty/latest/x86/putty-0.64-installer.exe)下载。  
**注意** 如果不使用putty登录，用homestead ssh登录，也可以。


<h2 id="homesteadSoftware">2. Homestead 相关软件安装</h2>

<h3 id="vagrant">2.1 vagrant  </h3>
vagrant是一个虚拟机管理工具。在添加虚拟机后，可以启动虚拟机的镜像，如果出现错误，可以随时销毁重建开发环境。  

从[这里](http://www.vagrantup.com/downloads.html) 下载。下载后直接安装。  
vagrant安装确认：

	C:\>vagrant --version
	Vagrant 1.7.2

<h3 id="virtualbox">2.2 virtualbox</h3>
virtualbox是作为vagrant的一个provider，安装后，在启动homestead时候，vagrant会自动启动virtualbox。  

从[这里](https://www.virtualbox.org/wiki/Downloads) 下载，下载直接安装。   


<h3 id="homestead">2.3 homestead</h3>
直接使用composer安装  

	composer global require "laravel/homestead=~2.0"

homestead将会被安装在`C:\Users\benjamincao\AppData\Roaming\Composer\vendor\laravel\homestead`

<h3 id="git">2.4 Git  </h3>
安装Git的目的是使用Git Bash，homestead的启动脚本是bash shell，windows下面使用Git Bash正好。   

点击[这里](http://www.git-scm.com/download/win)下载，然后安装。安装完毕就可以使用了。   


<h2 id="startup">3. 环境启动</h2>

<h3 id="vagrantImage">3.1 使用vagrant添加homestead镜像文件  </h3>

	vagrant box add laravel/homestead

选择virtualbox作为provider。  

下载的虚拟机，位于`C:\Users\benjamincao\VirtualBox VMs\homestead`  
**注意**虚拟机文件较大，下载需要较长时间，可以直接拷贝后，添加到vagrant中。   
		
	vagrant box add laravel/homestead file:///d:/hbox/virtualbox.box


<h3 id="keygen">3.2 生成rsa key  </h3>
通过ssh登录homestead需要这个。  

打开Git Bash，这是一个比较完备的Windows下的shell工具。在运行homestead的命令的时候都需要在这个bash中运行。    


	ssh-keygen.exe  -t rsa -C "caojianghui@carnetmotor.com"


<h3 id="configHomestead">3.3 配置并启动homestead</h3>

	
	cd AppData/Roaming/Composer/vendor/laravel/homestead/
	homestead init

生成homestead的配置文件，位于`C:\Users\benjamincao\.homestead\Homestead.yaml`

	homestead edit

使用编辑器打开创建的配置文件。  


	---
	# 虚拟机配置。
	ip: "192.168.10.10"
	memory: 2048
	cpus: 1
	provider: virtualbox

	authorize: ~/.ssh/id_rsa.pub

	keys:
	    - ~/.ssh/id_rsa
	
	# 共享文件夹配置，map是本地文件夹，此处修改为本地项目文件夹；to是虚拟机的文件夹，一般不需要修改，因为homestead中的nginx的配置也是如此。
	folders:
	    - map: ~/Code
	      to: /home/vagrant/Code
	
	# nginx的配置。需要在本地hosts中添加域名的解析。
	sites:
	    - map: homestead.app
	      to: /home/vagrant/Code/Laravel/public
	      hhvm: true
	
	# mysql的数据库配置
	databases:
	    - homestead
	
	# 这个不知道是干什么用的，猜测是环境变量设置。
	variables:
	    - key: APP_ENV
	      value: local

启动homestead  

	homestead up



<h3 id="login">3.4 登录说明</h3>
登录主要包括ssh的登录和mysql的登录。 

<h4 id="sshLogin">3.4.1 ssh登录  </h4>
	
	homestead ssh

我这里在使用`homestead ssh`时，总是会出现屏幕卡死的情况，推荐使用putty登录。    
初始用户名密码  `vagrant / vagrant`  


<h4 id="mysqlLogin">3.4 1 mysql密码  </h4>
`homestead  / secret`  
`root / secret`  


<h3 id="advanced">3.5 高级配置</h3>

### 3.5 增加新的网站
* 方法一  
在`homestead.yaml`中添加站点配置。然后再homestead目录执行命令`vagrant provision`。   
**`vagrant provision`这个命令是有破坏性的，它会重新构建数据库**  

* 方法二  
ssh登录homestead后，执行serve命令。
```shell
serve domain.app /home/vagrant/Code/path/to/public/directory 80
```

<h2 id="command">4. 命令详解</h2>

<h3 id="homesteadCommand">4.1 homestead命令</h3>


 命令      |    解释
:------    |    :------   
 up        |    启动homestead  
 halt	   |    停止homestead  
 init	   |    创建初始化的homestead.yaml  	
 edit	   |    编辑homestead.yaml  
 suspend   |   	挂起homestead  
 resume	   |   	继续挂起的homestead  
 ssh	   |   	通过ssh登录homestead  
 run	   |   	通过ssh在homestead上运行命令  
 status	   |   	获取homestead的状态  
 list	   |   	列表命令   
 help	   |   	显示命令的帮助  
 provision |   	重新配置homestead  
 destory   |   	销毁homestead		  
 update	   |   	更新homestead镜像  


******************************************

* [1. 本地环境设置](#localConfigig)
	* [1.1 编辑器](#sublimetext)
	* [1.2 php](#php)
	* [1.3 composer](#composer)
	* [1.4 putty](#putty)
* [2. Homestead 相关软件安装](#homesteadSoftware)
    * [2.1 vagrant](#vagrant)
    * [2.2 virtualbox](#virtualbox)
    * [2.3 homestead](#homestead)
    * [2.4 Git](#git)
* [3. 环境启动](#startup)
    * [3.1 使用vagrant添加homestead镜像文件](#vagrantImage)
    * [3.2 生成rsa key](#keygen)
    * [3.3 配置并启动homestead](#configHomestead)
    * [3.4 登录说明](#login)
    	* [3.4.1 ssh登录  ](#sshLogin)
    	* [3.4 1 mysql密码](#mysqlLogin)
* [4. 命令详解](#command)
	* [4.1 homestead命令](#homesteadCommand)
