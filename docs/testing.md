Testing
===========================================

## 1.介绍
Laravel框架骨子里就是经过严格测试的(Laravel is built with testing in mind)，实际上，PHPUnit是框架自带的，`phpunit.xml`配置文件已经安装到应用中。Laravel自带了很多方便的辅助方法(helper)来帮助你自如的测试应用。  

`test`目录包含了`ExampleTest.php`文件，在创建新的Laravel应用后，在命令行中直接运行`phpunit`，就可以运行测试了。

### 1.1 测试环境(Test Environment)
运行test时，Laravel自动将配置环境设置为`testing`，自动的将会话和缓存设置为`array`驱动，这意味着，测试中，不会有会话或者缓存的数据被固化。  
     
可以自由的设置测试环境，可以在`phpunit.xml`文件中设置`testing`环境变量。

### 1.2 定义并运行test(Defining & Running Tests)
只需要在`test`目录创建新的test文件，就可以创建一个test case。测试类需要继承自TestCase。然后就可以像平常使用PhpUnit一样定义测试方法了。只需要在终端中运行`phpunit`命令（位于`vendor/bin/phpunit`），就可以运行测试了。

```php
    class FooTest extends TestCase
    {
        public function testSomethingIsTrue()
        {
            $this->assertTrue(true);
        }
    }

```

**注意，如果你在test case中定义了自己的`setUp`方法，请确保调用`parent::setUp`方法。**


## 2. 应用测试
Laravel提供了非常简便的方法来发起HTTP请求，检查输出，甚至填写表格。例如，参考`test`目录的`ExampleTest.php`：

```php
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;
    
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }


```
`visit`方法发起一个`GET`请求，`see`方法确保我们在应用的回应中可以获取给定的文字，这是在Laravle中最基本的应用测试。


### 2.1 与应用交互(Interacting with your Application)
当然，仅仅断言响应中存在文本是最基本的，我们还可以点击链接，填充表格等。

#### 2.1.1 点击连接（Clicking Links）
这个测试中，我们发起一次请求，然后点击响应中的链接，然后断言我们确实达到指定的URI，例如，响应如下：

```html
    <a href="/about-us">About Us</a>
```

测试代码如下：

```php
    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }
```

#### 2.1.2 处理表单（Working with Forms）
Laravel提供了许多方法来处理表单，`type`,`select`,`check`,`attach`和`press`方法允许我们可以跟表单的input交互，假设应用注册表单如下：

```html
    <form action="/register" method="POST">
        {!! csrf_field() !!}
    
        <div>
            Name: <input type="text" name="name">
        </div>
    
        <div>   
            <input type="checkbox" value="yes"  name="terms"> Accept Terms
        </div>
    
        <div>
            <input type="submit" value="Register">
        </div>
    </form>

```

测试代码如下：

```php
    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }


```

当然如果表单包含Radio Button或者下拉框，都可以容易的填充，支持的表单防范如下：

|Mothod|Description|
--------|------------
|`$this->type($text, $elementName)`      | "Type"text into a given field|
|`$this->select($value, $elementName)`    | "Select" a radio button or Drop-down field   |
|`$this->check($element)`     | "Check" a checkbox field      |
|`$this->attach($pathToFile, $elementName)`   | "Attach" a file to form |
|`$this->press($buttonTextOrElementName)`     | "Press" a button with the given text or name      |

#### 2.1.3 处理附件（Working with Attachments）
如果表单包含type为`file`的input，那么可以使用`attach`方法来附加文件。

```php
    public function testPhotoCanBeUploaded()
    {
        $this->visit('/upload')
             ->name('File Name', 'name')
             ->attach($absolutePathToFile, 'photo')
             ->press('Upload')
             ->see('Upload Successful!');
    }

```
### 2.2 测试JSON APIs
Laravel提供了许多方法来处理JSON AOI和它们的响应，例如`get`, `post`, `put`, `patch`和`delete`等方法可以用来附带不同的HTTP变量来发起请求。数据和HTTP头（header）也很容易传递给这些方法。例子中，我们发起一个`post`请求到`/user`,并断言返回值是JSON格式。

```php
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->post('/user', ['name' => 'Sally'])
                 ->seeJson([
                    'created' => true,
                 ]);
        }
    }

```
`seeJson`方法将给定array转化为JSON然后验证JSON段是否出现在应用返回的整个JSON响应中。所以，如果在JSON响应中存在其他属性，只要给定JSON段存在，测试就会通过。


#### 2.2.1 校验JSON的准确匹配（Verify Exact JSON Match）
如果需要准确匹配返回的JSON，使用`seeJsonEquals`方法。

```php
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->post('/user', ['name' => 'Sally'])
                 ->seeJsonEquals([
                    'created' => true,
                 ]);
        }
    }

```

### 2.3 会话 / 认证（Sessions / Authentication）
Laravel提供了很多方法来处理测试过程中的session。首先使用`withSession`方法设定一个数组到session，这对于在测试请求前在session中加载数据很有用。

```php
    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $this->withSession(['foo' => 'bar'])
                 ->visit('/');
        }
    }

```

当然session的主要用途是管理用户状态，例如已验证用户等。`actingAs`方法提供了将给定用户指定为当前用户的简单的方法，例如，我们可以使用`model factory`来生成并认证用户。

```php
    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $user = factory('App\User')->create();
    
            $this->actingAs($user)
                 ->withSession(['foo' => 'bar'])
                 ->visit('/')
                 ->see('Hello, '.$user->name);
        }
    }
```

### 2.4 禁用中间件（Disabling Middleware）
测试中，有的情况下禁用中间件会使得测试特别方便。这样可以排除中间件的影响而使得route和controller完全隔离。Lavavel包含一个简单的trait`WithoutMiddleware`，通过它可以自动的在测试类中禁用middleware。

```php
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;
    
    class ExampleTest extends TestCase
    {
        use WithoutMiddleware;
    
        //
    }
```
如果只想在某些测试方法中禁用中间件，可以在这些测试方法中调用`withoutMiddleware`方法。

```php
    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->withoutMiddleware();
    
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }
```

### 2.5 定制HTTP请求（Custom HTTP Requests）
如果需要发起一个定制的HTTP请求，并获取一个完整的`Illuminate\Http\Response`对象，可以使用`call`方法。

```php
    public function testApplication()
    {
        $response = $this->call('GET', '/');
    
        $this->assertEquals(200, $response->status());
    }

```

如果发起`POST`,`PUT`,`PATCH`等请求，可以使用一个数组来作为input的数据，当然，这些数据可以从route或者controller中通过Request 实例获取。

```php
    $response = $this->call('POST', '/user', ['name' => 'Taylor']);

``` 

## 3. 与数据库交互
Laravel提供了很多有用的工具来方便的测试数据库驱动的应用。首先，你可以使用`seeInDatabase`方法来断言数据库中是否存在匹配一系列标准的数据。例如，我们想查看是否表`user`中是否存在`email`值为`sally@example.com`的记录。

```php
    public function testDatabase()
    {
        // Make call to application...
    
        $this->seeInDatabase('users', ['email' => 'sally@foo.com']);
    }

```

`seeInDatabase`等辅助方法只是为了测试更加方便，你可以使用任何其他的PHPUnit内建的断言来增加你的测试内容。

### 3.1 在每次测试后重置数据库
在每次测试后重置数据是很有用的，这样可以避免前面的测试干扰后续的测试。

#### 3.1.1 使用migrations
一个选择是在每次测试之前，回滚数据库，并migrate它。Laravel提供了一件简单的trait`DatabaseMigrations`来自动处理这一切，你只需要在你的测试类中使用这个trait。

```php
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;
    
    class ExampleTest extends TestCase
    {
        use DatabaseMigrations;
    
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }
```
#### 3.1.2 使用事务（Transactions）
另一个选择是将每个test case包裹在一个database 事务中。Laravel提供了一个trait`DatabaseTransactions`来自动处理这一切。

```php
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Illuminate\Foundation\Testing\DatabaseTransactions;
    
    class ExampleTest extends TestCase
    {
        use DatabaseTransactions;
    
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $this->visit('/')
                 ->see('Laravel 5');
        }
    }
```

### 3.2 Model工厂
在测试时，我们通常需要在执行测试前，插入很多数据到数据库中。我们不需要手动的指定数据的每一列，Laravel允许你使用`factories`来定义你的`Eloquent Model`的每条记录的指定属性的默认值。开始前，先看看`database/factories/ModelFactory.php`文件。默认的这个文件包含了一个工厂定义。

```php
    $factory->define(App\User::calss, function($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10），
        ];
    });
```

这个闭包（Closure）也就是模型工厂，可以返回模型的所有的默认的测试值。模型接受一个Faker php库的对象，这个对象可以方便的生成各种随机值。
   
任何的工厂都可以添加在`ModelFactory.php`文件中。

#### 3.2.1 多工厂类型（Multiple Factory Types）
有时候需要定义同一个`Eloquent Model`的多个工厂，例如，可以定义一个基于普通用户的`Administrator`模型工厂，可以使用`defineAs`方法。

```php
    $factory->defineAs(App\User::class, 'admin',    function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
            'admin' => true,
        ];
    });
```

当然不需要重复写基础的user工厂的每个属性，`raw`方法就是干这个的。一旦有了基础的属性，就可以添加任何需要的属性了。

```php
    $factory->defineAs(App\User::class, 'admin',    function ($faker) use ($factory) {
        $user = $factory->raw(App\User::class);
    
        return array_merge($user, ['admin' => true]);
    });
```

#### 3.2.2 测试中使用工厂
一旦定义好了工厂，就可以在测试中，或者database seed文件中来使用全局`factory`方法来生成模型实例。首先，使用`make`方法，这个方法创建实例，但是不会保存到数据库中。

```php
    public function testDatabase()
    {
        $user = factory(App\User::class)->make();
    
        // Use model in tests...
    }
```

如果想覆盖某些模型的默认值，只需要传递一个数组给`make`方法，只有指定属性会被重新设值而其余的值保持为工厂定义时候的默认值。

```php
    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
    ]);
```

还可以创建许多模型的集合，或者给定类型的多个模型。

```php
    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();
    
    // Create an App\User "admin" instance...
    $user = factory(App\User::class, 'admin')->make();
    
    // Create three App\User "admin" instances...
    $users = factory(App\User::class, 'admin', 3)->make();
```

#### 3.2.3 固化工厂模型（persisting Factory Models）
`create`方法不仅创建模型实例，而且使用Eloquent的`save`方法将实例保存到数据库中。

```php
    public function testDatabase()
    {
        $user = factory(App\User::class)->create();
    
        // Use model in tests...
    }
```

当然可以传一个数组给`create`方法来覆盖模型的某些属性。

#### 3.2.4 对模型增加关系（Add Relations To Models）
甚至，可以固化多个模型到数据库中。例如，可以附加一个关系(attach a relation)到创建模型中。当使用`create`方法创建多个模型时，返回一个Eloquent的`collection`实例。这样可以使用任何collection提供的方便的方法，例如`each`。

```php
    $users = factory(App\User::class, 3)
               ->create()
               ->each(function($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });
```

## 4. 模拟
### 4.1 模拟事件（Mocking Events）
如果你的应用重度依赖Laravel的事件模型，可能希望在测试时取消event或者模拟event。例如，如果在测试用户注册时，可能并不希望所有的`UserRegistered`事件的处理器都被触发，因为可能处理器只是在发送一封欢迎邮件等。   
  
Laravel提供了一个方柏霓的方法`expectsEvents`来验证希望的事件被触发，但是阻止任何事件的任何处理器运行。

```php
    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->expectsEvents(App\Events\UserRegistered::class);
    
            // Test user registration code...
        }
    }
```

使用`withoutEvent`方法可以阻止任何事件处理器运行，事件也不会被触发。

```php
    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->withoutEvents();
    
            // Test user registration code...
        }
    }
```

### 4.2 模拟作业（Mocking Jobs）
有时候，需要测试发起请求时，某些作业被controller分发。这可以将routes/ controllers从作业的逻辑中隔离开。当然我们可以在单独的类中测试作业本身。
   
Laravel提供了一个方便的方法`expectsJobs`，来确认作业被分发，但是作业本身并不会被执行。

```php
    class ExampleTest extends TestCase
    {
        public function testPurchasePodcast()
        {
            $this->expectsJobs(App\Jobs\PurchasePodcast::class);
    
            // Test purchase podcast code...
        }
    }
```

**注意，这个方法只是测试作业通过`DispatchesJobs`trait的dispatch方法被分发，并不会探测作业是否被直接发送到`Queue::push`。**


### 4.3 模拟Facades（Mocking Facades）
当测试时，会经常想模拟一个到Laravel facade的调用。例如，考虑如下的controller action：

```php
namespace App\Http\Controllers;

use Cache;
use Illuminate\Routing\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all users of the application.
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');
    
            //
        }
    }

```

我们可以通过使用`shouldReceive`方法来模拟调用`Cache`facade。这个方法返回一个Mockery实例。因为facade都是通过Laravel的服务容器来解析和管理的，它们比一个典型的静态类有更多的可测试性，例如模拟一次对Cache facade的调用：

```php
    class FooTest extends TestCase
    {
        public function testGetIndex()
        {
            Cache::shouldReceive('get')
                        ->once()
                        ->with('key')
                        ->andReturn('value');
    
            $this->visit('/users')->see('value');
        }
    }
```

**注意，`Request`方法不应该被模拟，相反，在测试时，应该将input传到HTTP的`call``post`等方法。**
