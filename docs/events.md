Events
==================================================


## 1. 介绍
Laravel的事件机制提供了一种简单的观察者模式实现，我们可以在应用中订阅和监听事件。事件类保存在`app/Events`目录，而监听者保存在`app/Listeners`目录。

## 2. 注册事件 / 监听者 （Registering Events / Listeners）
在Laravel应用中包含的`EventServiceProvider`提供了一个注册所有事件监听者的适当的地方。变量`listen`是一个包含所有事件(key)及其监听者(values)的数组，可以增加任意多的事件到这个数组中。例如：

```php
    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'App\Events\PodcastWasPurchased' => [
            'App\Listeners\EmailPurchaseConfirmation',
        ],
    ];
```

### 2.1 生成时间/监听者类
使用php artisan的`event:generate`命令，可以自动生成所有在`EventServiceProvider`中列出来的事件和监听器，已经存在的事件和监听器不会收到影响。
    
    php artisan event:generate

这意味着，我们需要先将事件和监听器写到`EventServiceProvider`中，然后执行命令。

## 3. 定义事件(Defining Events)
事件类就是一个数据容器(data container)，包含了事件相关的信息。例如，假设我们实例中生成的`PodcaseWasPurchased`事件，接收一个`Eloquent ORM`对象。

```php
    <?php
    namesapce App\Events;

    use App\PodCase;
    use App\Events\Event;
    use Illuminate\Queue\SerializesModels;

    class PodcaseWasPurchased extends Event
    {
        Use SerializesModels;

        public $podcost;

        /**
         * Create a new event instance.
         *
         * @param  Podcast  $podcast
         * @return void
         */
        public function __construct(Podcast $podcast)
        {
            $this->podcast = $podcast;
        }

    }
```

正如所见，事件类并没包含任何特殊的逻辑，仅仅包含了被购买的`Podcast`对象。事件类使用的`SerializesModels` trait会在事件对象使用PHP的`serialize`函数序列化时优雅的序列化Eloquent model。

## 4. 定义监听器(Defining Listener)

### 4.1 普通监听器
事件监听器在`handle`方法中接收事件对象。`event:generate`命令会自动导入相应的事件类并且会在`handel`方法中引入事件对象。可以在`handle`方法中执行必要的逻辑来响应事件。

```php
    <?php

    namespace App\Listeners;
    
    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class EmailPurchaseConfirmation
    {
        /**
         * Create the event listener.
         *
         * @return void
         */
        public function __construct()
        {
            //
        }
    
        /   *  * 
            *    Handle the event.
            *  
            *  @param  PodcastWasPurchased  $event
            *  @return void
            */ 
        public function handle(PodcastWasPurchased event)    
        {   
            // Access the podcast using <1event->   </1event->podcast...
        }
    }
```

事件监听器也可以在构造函数中引入(type-hint)任何的依赖。所有的事件监听器都是通过service container解析的，所以，依赖会自动注入。

```php
    use Illuminate\Contracts\Mail\Mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }
```

#### 停止事件传播
只要`handle`方法返回`false`，事件就不会传播到其他的监听器了。

### 4.2 定义队列事件监听器
需要将事件监听器排队？特别简单。只需要事件监听器实现`ShouldQueue`接口就可以了。使用artisan的`event:generate`命令生成的监听器已经导入了这个接口：

```php
    <?php

    namespace App\Listeners;
    
    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class EmailPurchaseConfirmation implements  ShouldQueue
    {
        //
    }


```

OK，当监听器被调用时，事件分配器将自动使用Laravel的队列系统来对其进行排队。如果在事件监听器执行过程中没有异常抛出，排队后的工作(queued job)将在执行后自动删除。

#### 手动访问队列
如果需要手动访问队列话工作的`delete`和`release`方法，也没问题。`Illuminate\Query\InteractsWithQueue` trait，提供了访问这些放大的路径。

```php
    namespace App\Listeners;

    use App\Events\PodcastWasPurchased;
    use Illuminate\Queue\InteractsWithQueue;
    use Illuminate\Contracts\Queue\ShouldQueue;
    
    class EmailPurchaseConfirmation implements  ShouldQueue
    {
        use InteractsWithQueue;
    
        public function handle(PodcastWasPurchased  $event)
        {
            if (true) {
                $this->release(30);
            }
        }
    }
```

## 5. 触发事件（Firing Events）
要想触发一个事件，使用Event外观模式(Event Facade)，将事件的实例作为参数传递给`fire`方法。`fire`方法将事件分发到所有它的已注册监听器。

```php
    <?php
    
    namespace App\Http\Controllers;
    
    use Event;
    use App\Podcast;
    use App\Events\PodcastWasPurchased;
    use App\Http\Controllers\Controller;
    
    class UserController extends Controller
    {
        /**
         * Show the profile for the given user.
         *
         * @param  int  $userId
         * @param  int  $podcastId
         * @return Response
         */
        public function purchasePodcast($userId,    $podcastId)    
        {   
               $podcast = Podcast::findOrFail(<1podca> </1podca>stId);
    
            // Purchase podcast logic...
    
            Event::fire(new     <PodcastWasPurcha>  </PodcastWasPurcha>sed($podcast));
        }   
    }
```

另外，可以使用全局的`event`方法来触发事件。

```php
    event(new PodcastWasPurchased($podcast));
```

## 5. 广播事件（Broadcasting Events）
在许多现代的web应用中，使用web socket来实现即时的(real-time)，实时更新(live-updating)的用户接口。当数据在服务器端更新时，往往通过客户端处理的websocket连接发送一条消息。    

Laravel提供简单的方法来通过一个websocket连接来广播事件，以此协助创建这样的应用。通过广播Laravel事件，你可以在服务端代码(server-side code)和客户端的javascript框架(client-side javascript framework)间共享同样的事件名。

### 5.1 设置(Configuration)
事件广播相关的配置都在文件`config/broadcasting.php`文件中。Laravel自带支持很多broadcast驱动，`Pusher``Redis`以及为本地开发和debug使用的`log`驱动。

#### 广播的必要条件
要广播需要以下依赖
* Pusher `pusher/pusher-php-server ~2.0`
* Redis: `predis/predis ~1.0`

#### 队列的必要条件
在广播事件之前，需要配置并运行一个队列监听器(queue listener)，所有的事件广播是通过队列化的job来执行，这样，应用的相应事件不会造成严重影响。

### 5.2 设置事件广播(Making Events for Broadcast)
要通知Laravel一个给定事件需要广播，实现`Illuminate\Contracts\Broadcasting\ShouldBroadcast`接口。这个接口要求实现一个方法`broadcastOn`，这个方法返回时间应该在那个通道(channel)广播的数组。

```php
    <?php

    namespace App\Events;
    
    use     App\User;
    use     App\Events\Event;
    use     Illuminate\Queue\SerializesModels;
    use     Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    
    class ServerCreated extends Event implements    ShouldBroadcast
    {
        use SerializesModels;
    
        public $user;
    
        /**
         * Create a new event instance.
         *
         * @return void
         */
        public function __construct(User $user)
        {
            $this->user = $user;
        }
    
        /** 
         * Get the channels the event should be     broadcast on.
         *
         * @return array
         */
        public function broadcastOn()
        {
            return ['user.'.$this->user->id];
        }
    }
```

然后，需要像平常一样触发事件，一旦事件被触发，一个队列作业(queued job)将在你指定的broadcast 驱动上自动广播事件。


### 5.3 广播数据(Broadcast Data)
当一个事件被广播时，所有的公共属性都会自动序列化并作为事件的payload被广播。你可以在JavaScript代码中访问任何的`public`数据。所以，如果你的事件有一个单独的$user属性，这个属性是一个Model，广播的负载是：

```javascript
        {
        "user": {
            "id": 1,
            "name": "Jonathan Banks"
            ...
        }
    }
```

然而，如果你希望对你的广播负载有更细粒度的控制，可以增加一个`broadcastWith`方法。这个方法返回一个需要广播的数据的数组。

```php
    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }


```

### 5.4 消费事件广播(Consuming Event Broadcasts)
#### Pusher
通过使用Pusher的Javascript SDK，可以方便的获取使用Pusher驱动广播的数据，例如，我们使用前面例子中的`App\Event\ServerCreated`事件的数据：

```javascript
    this.pusher = new Pusher('pusher-key');
    
    this.pusherChannel = this.pusher.subscribe('user.'  + USER_ID);
    
    this.pusherChannel.bind('App\Events\ServerCreated ', function(message) {
        console.log(message.user);
    });
```

#### Redis
如果要使用Redis广播驱动，需要自己编码Redis的p
ub/sub消费者(consumer)来接收信息并通过websocket技术来广播它。例如可以选择使用用Node编写的`Socket.io`库。
使用`socket.io`和`ioredis`node库，可以很快的写一个事件广播器来发布Laravel应用广播的所有的事件。

```javascript
    var app = require('http').createServer(handler);
    var io = require('socket.io')(app);
    
    var Redis = require('ioredis');
    var redis = new Redis();
    
    app.listen(6001, function() {
        console.log('Server is running!');
    });
    
    function handler(req, res) {
        res.writeHead(200);
        res.end('');
    }
    
    io.on('connection', function(socket) {
        //
    });
    
    redis.psubscribe('*', function(err, count) {
        //
    });
    
    redis.on('pmes  sage', function(subscribed, channel,  message) {   
        message = JSON.parse(message);  
        io.emit(channel + ':' + message.event, message. data);
    });
```

## 6. 事件订阅者(Event Subscribers)
事件订阅者是可以订阅类本身出发的多个事件的类，这样可以在一个类中订阅多个事件监听器（define several event handlers with a signle class）。事件订阅者应该定义一个`subscribe`方法，需要传一个事件分发器(event dispatcher)作为参数。

```php
    <?php
    
    namespace App\Listeners;
    
    class UserEventListener
    {
        /**
         * Handle user login events.
         */
        public function onUserLogin($event) {}
    
        /**
         * Handle user logout events.
         */
        public function onUserLogout($event) {}
    
        /*  * 
         *   Register the listeners for the subscri   ber.
         *  
         *  @para    m  Illuminate\Events\Dispatcher     $e vents    
            *  @return array    
            */ 
        public function subscribe($events)  
        {   
            $events->listen(    
                'App\Events\UserLoggedIn',  
                'App\Listeners\UserEventListener@onUser Login'
            );
    
            $events->listen(    
                'App\Events\UserLoggedOut', 
                'App\Listeners\UserEventListener@onUser Logout'
            );
        }
    
    }


```

#### 注册事件订阅者
一旦定义好订阅者，可以将它跟事件分发器注册在一起。可以使用`EventServiceProvider`的`$subscribe`属性来注册订阅者。

```php
    <?php

    namespace App\Providers;
    
    use Illuminate\Contract s\Events\Dispatcher as DispatcherContract;   
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
    
    class EventServiceProvider extends ServiceP rovider
    {   
        /** 
         * The event listener mappings for the  application.
         *
         * @var array
         */
        protected $listen = [
            //
        ];
    
        /**
         * The subscriber classes to register.
         *
         * @var array
         */
        protected $subscribe = [
            'App\Listeners\UserEventListener',
        ];
    }
```

