Session
======================================

* 1.介绍
    * 1.1 配置
    * 1.2 各驱动先决条件
        * 1.2.1 Database
        * 1.2.2 Redis
    * 1.3 其他session注意事项
* 2. 基础用法
    * 2.1 访问Session
    * 2.2 确定Session是否包含值
    * 2.3 在Session中保存值
    * 2.4 Push数据到Session的数组类型
    * 2.5 获取并删除一个值
    * 2.6删除Session中的值
    * 2.7 重新生成Session ID
* 3. Flash data
* 4. 自定义Session驱动（driver）


## 1.介绍

由于HTTP是无状态（stateless）的协议，所以在HTTP驱动的应用中需要使用session来保存用户在不同请求中需要共享的信息。用户可以通过简洁，统一的API来利用Laravel自带的多种session后端（back-ends）。Laravel支持流行的后端，例如Memcached，Redis等，而数据库更是天生支持（out of box）。

### 1.1 配置
Session的配置文件在`config/session.php`，请确保你了解此文件中的所有已经文档化的选项。Laravel默认使用`file`作为Session驱动，这可以满足大部分的应用需要。在生产环境，请考虑使用`memcached` `redis`作为Session驱动，以此获取更快的session性能。    

session驱动，定义了每次请求的session数据保存在哪里。Laravel框架支持很多特别好的驱动：    

* `file`: Session保存在文件`storage/framework/sessions`。
* `cookie`: Session保存在安全的，加密过的cookies中。
* `database`: Session保存在数据库中。
* `memcached / redis`: Session保存在快速的，基于缓存的存储中。
* `array`: Session保存在一个php的数组中，不能在不同请求中固化（persistent）。   

**Note：array驱动一般用在测试中，就是为了保证session不被固化**

### 1.2 各驱动先决条件
#### 1.2.1 Database
如果使用`database`作为Session驱动，必须创建保存Session的数据库表。
    
```php
    Schema::create('sessions', function ($table) {
        $table->string('id')->unique();
        $table->text('payload');
        $table->integer('last_activity');
    });
```

可以使用Artisan命令 `session:table`来自动生成migration。

```bash
    php artisan session:table
    composer dumpautoload
    php artisan migrate
```


#### 1.2.2 Redis
在使用`redis`作为session驱动之前，需要通过`composer`安装`predis/predis(~1.0)`.


### 1.3 其他session注意事项
Laravel框架内部使用`flash`关键字作为session的key，添加session时，不可以使用这个key。    
   

如果需要加密保存所有的session，将`encrypt`设置为`true`。

## 2. 基本用法

### 2.1 访问Session
可以通过HTTP的request来访问session的对象。

```php
    namespace App\Http\Controllers;
    
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function showProfile(Request $request, $id)
        {
            $value = $request->session()->get('key');
    
            //
        }
    }
```

当获取session值的时候，可以传入一个默认值作为`get`方法的第二个值，当从session中无法获取值时，返回这个默认值。第二个参数也可以是一个匿名函数（Closure），其返回值在无法获取指定值时返回。

```php
    $value = $request->session()->get('key', 'default');
    $value = $request->session()->get('key', function() {
        reutrn 'default';
    });
```

如果想获取所有的值，可以使用`all`方法的第二个值，当从session中无法获取值时，返回这个默认值。第二个参数也可以是一个匿名函数（Closure），其返回值在无法获取指定值时返回。

```php
    $data = $request->session()->all();
```

我们也可以使用PHP的全局方法`session`来获取或者存储数据。

```php
    Route::get('home', function() {
        $value = session('key');

        session(['key' => 'value']);
    });
```


### 2.2 确定Session是否包含值
使用`has`方法检测Session中是否包含某key，如果存在，将返回`true`。

```php
    if ($request->session()->has('users')) {

    }
```


### 2.3 在Session中保存值
使用`put`方法将值保存到session中。

```php
    $request->session()->put('key', 'value');
```



### 2.4 Push数据到Session的数组类型

如果session的值是一个数组，可以使用`push`方法将一个新的值push到数组中。

```php
    $request->session()->push('user.teams', 'developers');
```


### 2.5 获取然后删除一个值
使用`pull`方法获取然后删除这个key对应的session值。

```php
    $request->session()->pull('key', 'default');
```

### 2.6删除Session中的值

使用`forget`方法删除session中的值，如果要清空session，使用flush方法。

```php
    $request->session()->forget('key');
    $request->session()->flush();
```

    
### 2.7 重新生成Session ID

使用`regenerate`方法重新生成session ID。

```php
    $request->session()->regenerate();
```

## 3. Flash Data
如果只是需要将保存在session中的值用在下一个请求中，可以使用`flash`方法。使用这个方法保存在session中的值，只可以在随后的请求中使用，然后就被删除，Flash data主要用在短期有效的状态消息。

```php
    $request->session()->flash('status', 'task was successful');
```

如果你想继续为下一个的请求保留Flash data，可以使用`reflash`方法，这样可以将flash data保持到下一个请求。如果只是想保留特定的flash data，可以使用`keep`方法。

```php
    $request->session()->reflash();
    $request->session()->keep(['username', 'email']);
```

## 4. 自定义Session驱动

要想增加Laravel的session后端，可以使用Session facade的`extend` 方法。我们可以在一个service provider的`boot`方法中调用这个`extend`方法。

```php
    <?php namespace App\Providers;
    
    use Session;
    use App\Extensions\MongoSessionStroe;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot() {
            Session::extend('mongo', function($app) {
                // Return implementation of SessionHandlerInterface...
                return new MongoSessionStore;
            });
        }

        /**
         * Register bindings in the container.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }
```

自定义的session driver需要实现接口`SessionHandlerInterface`。这个接口包含一些我们需要实现的简单接口。大概的MongoDB的实现如下：

```php
    <?php

    namespace App\Extensions;
    
    class MongoSessionStore implements SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }
```

这些接口并不如Cache的`StroeInterface`那么容易理解，下面解释一下这些方法都做了什么：

* `open`： 主要用在基于`file`的session保存系统中。由于Laravel自带了一个`file`session驱动，你几乎从来也不用在这个方法中添加任何代码。直接留空即可。这是一个不好的接口定义的事实，但是PHP语法要求必须实现这个方法（interface）。
* `close`： 跟open方法一样，基本不用管，大部分的驱动不需要这个方法。
* `read`： 返回`$sessionId`关联的session数据的字符串格式。我们不需要序列化或者做任何的编码来处理写入或者读取出来的数据，Laravel会处理这一切。
* `write`： 将`$data` 写入跟`$sessionId`关联的固化存储中，例如MongoDB，Dynamo等。
* `destroy`： 删除跟`$sessionId`相关联的所有的已经固化的数据。
* `gc`： 删除所有比`$lifetime`更早的session数据。对于自过期(self-expiring)系统，例如Memcached，Redis来说，这个方法直接留空即可。


只要这个session driver被注册了，你就可以在`config/session.php`配置文件中使用`mongo`驱动了。