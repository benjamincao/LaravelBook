Errors & Logging
==============================================
* [1.介绍](#introduction)
* [2.配置](#configuration)
* [3.异常处理](#exceptionHandler)
    * [3.1 Report方法](#reportMethod)
    * [3.2 Render方法](#renderMethod)
* [4.HTTP异常](#httpException)
* [5.日志](#logging)

<h2 id="introduction">1. 介绍</h2>
当启动新的Laravel项目时，错误和异常处理已经配置好了。另外，Laravel集成了Monolog日志，这个库提供了大量的功能强大的log处理器。
<h2 id="configuration">2.配置</h2>
### Error Detail
应用通过浏览器显示的错误细节的数量是通过在`config/app.php`中的`debug`配置项控制的。默认情况下，这个配置项取值为`APP_DEBUG`环境变量，这个变量保存在`.env`文件中。  
对于开发者，应该将`APP_DEBUG`设置为`true`，在生产环境，应该设置为`false`。
### Log Modes
不需要经过配置，Laravel支持`signle`, `daily`, `syslog`, `errorlog`等几种日志模式。例如，如果需要使用daily日志文件取代single日志文件，只需要将`config\app.php`配置文件中的`log`设置一下就可以。  

```php    
    `log` => 'daily',
```

### Custom Monolog Configuration
可以通过使用应用的`configureMonologUsing`方法来对Monolog进行完全的控制。在`bootstrap/app.php`文件中的`$app`返回之前，调用这个方法即可。

```php
    $app->configureMonologUsing(function($monolog) {
        $monolog->pushHandler(...);
    });

    return $app;
```

<h2 id="ExceptionHandler">3.异常处理</h2>
所有的异常都是被`App\Exception\Handler`类来处理的，这个类包含两个方法：`report`和`renderMethod`。
<h3 id="reportMethod">3.1 Report方法</h3>
`report`方法用来记录异常，或者将异常发送到例如`BugSnag`的外部服务。默认情况下，`report`方法只是简单的将异常传递给记录异常的基础的类。然而，我们可以依据自己的想法来记录所有的异常。   
例如，如果需要对各种exception单独处理，可以使用PHP的`instanceof`操作符。

```php
    /**
     * Report or log an exception.
     *
     * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
     *
     * @param  \Exception  $e
     * @return void
     */
    public function report(Exception $e)
    {
        if ($e instanceof CustomException) {
            //
        }

        return parent::report($e);
}

```

### 忽略某些类型的exception
Handler类的`$dontReport`属性用来记录不需要记录日志的异常类型，它是一个数组。默认情况下，404错误的异常不会被记录到log文件中。可以按需添加任何的exception到这个数组中。

<h3 id="renderMethod">3.2 Render方法</h3>
`render`方法是用来将一个给定的exception转化为一个HTTP响应，并将这个响应返回到浏览器。默认情况下，异常传送给基类，它生成一个response。但是，你也可以根据异常类型不同返回定制的响应。

```php
    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $e
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $e)
    {
        if ($e instanceof CustomException) {
            return response()->view('errors.custom', [], 500);
        }

        return parent::render($request, $e);
}
```

<h2 id="httpException">4.HTTP异常</h2>
一些异常描述了服务器端的HTTP错误码，例如，404（page not found），401（unauthorized error）甚至500错误。为了在应用中的任何地方都能生成这样的响应，利用如下代码：

```php
    abort(404);
```

`abort`方法会立刻生成一个异常，并被异常处理器的render方法处理。第二个参数可以设定异常内容：

```php
    abort(403, 'Unauthorized action.');
```

这个方法可以在request的生命周期的任何时候调用。

### 自定义HTTP错误页(Custom HTTP Error Pages)
使用Laravel，可以很容易的对不同的HTTP状态码生成不同的错误页。例如，如果想自定义404页面，创建文件`resources\views\errors\404.blade.php`，这个页面会被应用产生的所有404错误使用。  
在这个目录中的视图，需要被命名为匹配HTTP状态码。

<h2 id="logging">5.日志</h2>
Laravel的日志记录功能提供了基于功能强大Monolog的简单的封装。默认情况下，laravel创建每日log，并保存在`storage/logs`目录下。可以使用`Log`外观模式来写入log。

```php
    namespace App\Http\Controllers;

    use Log;
    use App\User;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $id
         * @return Response
         */
        public function showProfile($id)
        {
            Log::info('Showing user profile for user: '.$id);

            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }
```

log提供了RFC5424中定义的八种log等级。emergency, alert, critical, error, warning, notice, info, dubug

```php
    Log::emergency($error);
    Log::alert($error);
    Log::critical($error);
    Log::error($error);
    Log::warning($error);
    Log::notice($error);
    Log::info($error);
    Log::debug($error);
```

### 上下文信息(Contextual Information)
可以传递一个上下文相关的数据给log方法，这些上下文数据会被格式化后显示在log信息中

```php
    Log::info('User failed to login.', ['id' => $user->id]);
```

### 访问底层的Monolog实例(Accessing the underlying Monolog Instance)

Monolog提供了大量的处理器老记录log。使用如下方式获取实例。

```php
    $monolog = Log::getMonolog();
```




