HTTP Middleware
===================================

* 中间件介绍
* 定义中间件
* 注册中间件
* 中间件参数
* 可中止中间件(terminable)

## 中间件介绍
HTTP中间件提供了一个过滤进入Application的请求的方便的机制。例如，Laravel包含了一个验证应用用户是否经过认证的中间件。如果用户通过认证，则中间件允许请求进入应用，否则，页面跳转到登录页面。   
  
当然，除了认证外，可以编写执行不同任务的中间件。一个CORS中间件可以给所有发送的response添加正确的header。一个logging中间件可以用来记录所有进入应用的请求。
   
在Laravel框架中，存在很多中间件，包括管理、认证、CSRF保护等等。所有中间件都位于`App/Http/Middleware`目录。

## 定义中间件
使用`make:middleware`命令创建一个中间件。 

```sh
    php artisan make:middleware OldMiddleware
```

这个命令会在`App\Http\Middleware`目录生成一个新的`OldMiddleware`类，在这个中间件中，我们年龄大于20的人访问路由，否则将会把用户重定向到`home` URI。

```php
    namespace App\Http\Middleware;

    use Closure;

    class OldMiddleware {
        public function handler($request, Closure $next) {
            if ($request->input('age') <= 20) {
                return redirect('home');
            }

            return $next($request);
        }
    }

```

如果`age`小于等于20，中间件将返回一个HTTP重定向；否则请求将会传送到应用的下一步。将请求发送到应用的更深层处理（允许本中间件‘pass’），只需要调用`$next`回调函数，以$request作为参数。
   
将middleware想象为一系列的层，所有的http请求都需要穿过这些层，然后到达应用。每一层都将检测请求，甚至完全的拒绝。

### Before / After中间件
一个中间件在请求之前还是之后运行，取决于中间件本身。例如，下列中间件在请求被应用处理前执行

```php
    namespace App\Http\Middleware;

    use Closure;

    class BeforeMiddle
    {
        public function handler($request, Closure $next) {
            // Perform action

            return $next($request);
        }
    }
```

然而，下列中间件在请求被应用处理完后执行。

```php
    namespace App\Http\Middleware;

    use Closure;

    class AfterMiddleware
    {
        public function handler($request, Closure $next)
        {
            $response = $next($request);
            // Perform action 

            return $response;
        }
    }
    
```


## 注册中间件

### 全局（global Middleware）
如果某个中间件需要在每个HTTP请求时执行，只需要将这个中间件类写在`app/Http/Kernel.php`类的`$middleware`属性中。

### 路由专属中间件(Assigning Middleware to Routes)
如果需要将中间件指派给特定的route，首先，需要在`app/Http/Kernel.php`文件中制定一个简写的key，默认情况下，这个类的`$routeMiddleware`属性包含了Laravel的中间件的条目。要想增加新的中间件，把它添加在这个属性中，指定一个简写的key就好。例如：

```php
    // Within App\Http\Kernel Class...

    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    ];

```
在HTTP kernel中定义了中间件后，就可以在route的可选数组中使用这个中间件了。

```php
    Route::get('admin/profile',  ['middleware' => 'auth', function() {
        //
    }]);
```

## 中间件参数
中间件也可以接收格外的用户参数。例如，如果你的应用需要在执行指定action前，验证已经经过认证的用户属于某角色，你可以创建一个RoleMiddleware，这个中间件接收一个角色名作为格外的参数。

格外的middleware参数将会在`$next`参数后被传递到中间件

```php
    namespace App\Http\Middleware;
    use Closure;

    class RoleMiddleware
    {
        /**
         * Run the request filter.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @param  string  $role
         * @return mixed
         */
        public function handle($request, Closure $next, $role)
        {
            if (! $request->user()->hasRole($role)) {
                // Redirect...
            }
    
            return $next($request);
        }
    }
```

中间件参数可以在定义route时候，通过用`:`分隔开middleware和参数来指定。多个参数使逗号分隔。

```php
    Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
        //
    }]);

```


## 可中止中间件
有时候，需要在response已经发送给浏览器后，执行一些工作，例如Laravel中的session中间件需要在response已经发送给浏览器后将session数据写入存储中。未完成这一点，可以将中间件定义为可终止的“terminable”，只需要加一个`terminate`方法就可以了

```php
    namespace Illuminate\Session\Middleware;
    
    use Closure;
    
    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }
    
        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }

```

`terminate`方法接收`request`和`response`两个参数。一旦定义了一个可中止的中间件，需要把它添加到`app\Http\Kernel.php`的全局的middleware中。
            
当调用中间件中的`terminate`方法时，Laravel会从`service container`中生成一个全新的middleware实例。如果你想在调用中间件的`handle`和`terminate`方法时使用同一个中间件实体，那么可以使用service container的`singleton`方法来注册这个中间件。