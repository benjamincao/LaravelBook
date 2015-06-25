Laravel Blade   
====================================  

* [1. Blade模板](#templating)
* [2. Blade控制结构](#control)
* [3. 服务注入](#serviceInject)
* [4. FAQ](#faq)

<h2 id="templating">1. Blade模板</h2>
Blade是Laravel提供的简单但是功能强大的模版引擎。跟控制器布局不同，Blade是模版继承(template inheritance)和区块(section)驱动的。所有的模板文件使用`.balde.php`作为后缀。    

### 1.1 定义模板
我们使用bootstrap作为前端css框架，所以，我们的模板也是基于bootstrap的官方模板修订。    

```html
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->
    <title>App Name - @yield('title')</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">

    <!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
    <!-- WARNING: Respond.js doesn't work if you view the page via file:// -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/html5shiv/3.7.2/html5shiv.min.js"></script>
      <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
    <![endif]-->
  </head>
  <body>
  @section('sidebar')
  This is the master sidebar.
  @show
  <div class="container">
@yield('content')
  </div>

    <!-- jQuery (necessary for Bootstrap's JavaScript plugins) -->
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>
    <!-- Include all compiled plugins (below), or include individual files as needed -->
    <script src="js/bootstrap.min.js"></script>
  </body>
</html>
```

**注意** 布局保存在 `resources/views/layouts/master.blade.php`。  

### 1.2 使用模板

	@extends('layout.master')
	
	@section('title', 'Page Title')

	@section('sidebar')
		@parent
		<p>This is appended to the master sidebar</p>
	@stop

	@section('content')
		<p>This is my body content</p>
	@stop

注意，扩展(extends)了一个Blade布局的视图(view)只是简单的重写了布局中的section。可以使用section中`@parent`_(文档中说这里使用`@@parent`,实际测试，使用一个@即可)_，指令将布局的内容添加到子视图中。在侧边栏或者页脚中可以使用这种技巧。   
有时候，无法确定一个区块是否被定义，可以传递一个默认的值给`yield`指令。  

	@yield('section', 'Default content')




<h2 id="control">2. Blade控制结构</h2>

### 2.1 输出值(Echoing data)

	Hello, {{ $name }}.
	The current UNIX timestamp is {{ time() }}

两个大括号抱起来的内容可以被转义输出。

**注意** 要想从view中将变量传递到blade，可以使用`with('varName', 'varValue')`的方式，同样可以传递array到blade，处理array时，需要注意使用foreach等处理，不可以直接使用`{{ $var }}`的方式输出。   

### 2.2 确认是否存在后输出值
有时候，你想输出一个变量值，但是你不确认这个变量是否已经赋值，可以使用这样的语法。

	{{ isset($name) ? $name : 'Default'}}
	Or 
	{{ $name or 'Default'}}

### 2.3 使用大括号显示原始文本
如果需要显示两个大括号包括的一些文字，可以在大括号前加@，这样的语句不会被Blade处理。  
	
	@{{ This will not be processed by Blade. }}

如果你不需要Blade转义——虽然这样并不安全——可以使用如下语法：

	{!! $name !!}

### 2.4 if语句

	@if (count($records) === 1)
		I have one record.
	@elseif (count($records) > 2) 
		I have multiple records.
	@else
		I have no records.
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

### 2.5 循环(loop)

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i}}.
	@endfor

	@foreach ($users as $user)
		This is the user {{ $user->name }}.
	@endforeach

	@forelse ($users as $user)
		<li>{{ $user->name }}</li>
	@empty
		{{ no user! }}
	@endforelse

	@while (true)
		I am looping for ever.
	@endwhile  

### 2.6 引入子视图(<I></I>ncluding sub-view)

	@include('view.name')

	Or pass some arguments to it.
	@include('view.name', ['some' => 'value'])

### 2.7 重写区块(Overwriting section)
使用`overwrite`语句，完全覆盖一个section。
	
	@extends('list.item.container')

	@section('list.item.content')
		This is a item of type {{ $item->type }}.
	@overwrite


### 2.8 显示语言(Displaying language lines)

	@lang('language.line')
	@choice('language.line', 2)

详见language。

### 2.9 注释

	{{-- The comment will not be rendered to HTML --}}


<h2 id="serviceInject">3. 服务注入</h2>
使用`@inject`指令可以从Laravel服务容器(service container)中获取一个服务。第一个参数是保存服务的变量名，第二个参数是你希望获取的服务的类或者接口名称。

	@inject('metrics', 'App\Service\MetricsService')

	<div>
		Monthly revenue: {{ $metrics->monthlyRevenue() }}.
	</div>


<h2 id="faq">4. FAQ</h2>

### 4.1 yield 和 section的区别
yield和section都可以预定义占位的块。却别在于：
*在父模板中，可以使用yield和section来设置占位符，但是在子模板中，不可以使用yield来输出内容。否则会被当作新的占位符。  

#### yield可以设置默认值，当子模板中没有设置值的时候，则输出默认值。section则不可以设置默认值。        
	
	{{-- layout.master --}}
	@yield('title', 'Default Title')

	{{-- index --}}
	@extends('layout.master')
	


输出`Default Title`.


#### yield只能替换不能扩展，即不能使用`@parent`来扩展，而section可以替换和扩展。   

	
	{{-- layout.master --}}
	@yield('title', 'Default Title')

	{{-- index --}}
	@extends('layout.master')
	@section('title')
	@parent
	New Title
	@stop



只会输出`New Title`。   

	{{-- layout.master --}}
	@section('title')
	Parent title 
	@show

	{{-- index --}}
	@extends('layout.master')
	@section('title')
	@parent
	New Title
	@stop

输出 `Parent title New Title`


### 4.2 section中指令@show, @stop, @append, @overwrite, @parent的功能

#### @show和@stop的区别   
@show表示到此处将section的内容输出到页面。@stop只是进行内容解析，不再处理模板中对该section的处理，除非用@override覆盖或者使用@append来附加。一般来说，在定义section时，用@show，在扩展或者替换时，使用@stop。  


#### @append和@overwirte的却别   
@append可以在已经使用@stop结束的section后，附加内容。   

	@extends('layout.base')



	@section('sidebar')
		<p>this is line one.</p>
	@stop
	
	@section('sidebar')
		<p>this is line three</p>
	@show
	
	@section('sidebar')
		<p>thi is line two</p>
	@append
	
	@section('sidebar')
		<p>this is line three</p>
	@show
	
	
	@section('sidebar')
		<p>this is not shown</p>
	@stop

输出：

	this is line one.

	this is line one.

	thi is line two


@overwrite则是对使用@stop结束的section进行覆盖。

	@extends('layout.base')



	@section('sidebar')
		<p>this is line one.</p>
	@stop
	
	@section('sidebar')
		<p>this is line three</p>
	@show
	
	@section('sidebar')
		<p>thi is line two</p>
	@overwrite
	
	@section('sidebar')
		<p>this is line three</p>
	@show
	
	
	@section('sidebar')
		<p>this is not shown</p>
	@stop

输出：  

	this is line one.

	thi is line two
	
	