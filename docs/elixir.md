Elixir
=============================================

* 1. 介绍
* 2. 安装
* 3. 运行Elixir
* 4. 处理Stylesheets
    * 4.1 Less
    * 4.2 Sass
    * 4.3 普通css（plain css）
    * 4.4 Source Map
* 5. 处理Scripts
    * 5.1 CoffeeScript
    * 5.2 Browserify
    * 5.3 Babel
    * 5.4 Script
* 6. 版本化 / 破除缓存(versioning / cache busting)
* 7. 调用已存在的Gulp任务
* 8. 编写Elixir扩展


## 1. 介绍
Laravel Elixir提供了一套简洁流畅的API来定义基本的Gulp任务。Elixir支持很多常用的Css和Javascript的预编译器(pre-processors)，以及测试工具。使用任务链，Elixir允许你定义你的资源管道（asset pipeline）。
    
    elixir(function(mix) {
        mix.sass('app.scss')
            .coffee('app.coffee');
    });

如果你曾经对如何使用gulp以及怎么进行资源编译(asset compilation)感到困惑，那么你一定会爱上Laravel Elixir。

## 2. 安装
### 2.1 Node
node是必须的，如果没安装，请到[download](http://nodejs.org/download/)下载。

    node -v 

### 2.2 Gulp
需要安装gulp，作为npm的一个全局包。

    npm install --global gulp

### 2.3 Laravel Elixir
使用npm来安装Laravel Elixir，你会在项目根目录发现一个文件`package.json`，这是npm的配置文件。直接运行命令安装即可。

    npm install

**注意，在安装过程中会提示需要python命令，到python官网下载python安装，并指定PATH就可以了。**


## 3. 运行Elixir
Elixir基于Gulp构建，所以只需要在终端运行`gulp`命令即可运行Elixir命令。添加参数 `--production`，会自动构建Css和JavaScript的min版本。

    // Run all tasks...
    gulp

    // Run all tasks and minify all CSS and JavaScript...
    gulp --production
    

## 4. 处理Stylesheets
### 4.1 Less
### 4.2 Sass
### 4.3 普通css（plain css）
### 4.4 Source Map
## 5. 处理Scripts
### 5.1 CoffeeScript
### 5.2 Browserify
### 5.3 Babel
### 5.4 Script
## 6. 版本化 / 破除缓存(versioning / cache ##usting)
## 7. 调用已存在的Gulp任务
## 8. 编写Elixir扩展