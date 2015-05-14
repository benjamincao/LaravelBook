
# 集成开发环境配置  
Laravel的目的就是让开发这的开发体验更愉快，开发过程更加简单。在环境搭建上也是如此。    
Laravel homestead是一个官方发布的vagrant封装包——一个虚拟机——提供完备的开发环境。在homestead配置完成后，不需要在本地环境中配置任何的开发工具，vagrant可以搞定一切。因为是虚拟机，不会对本地系统产生任何的破坏。同时如果出现故障，可以在几分钟内销毁并重建虚拟机。  
homestead是跨平台的，可以在windows，Linux，Mac上运行，其中集成了nginx，php5.6，MySql，postgres，Redis，Memcached，HHVM等所有开发laravel所需的软件。   
本文档是针对windows环境下homestead的配置运行。


## 1. 本地环境设置  

### 1.1 编辑器  
建议使用[sublime Text 3](http://www.sublimetext.com/3)，会有单独文档来说明如何使用Sublime Text高效开发php。


### 1.2 php  
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


### 1.3 composer  
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


### 1.4 Git  
安装Git的目的是使用Git Bash，homestead的启动脚本是bash shell，windows下面使用Git Bash正好。   
点击[这里](http://www.git-scm.com/download/win)下载，然后安装。安装完毕就可以使用了。   



## 2. Homestead 软件安装

### 2.1 vagrant  

### 2.2 virtualbox

### 2.3 homestead
直接使用composer安装  

	composer global require "laravel/homestead=~2.0"
