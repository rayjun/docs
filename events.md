# 事件

- [介绍](#introduction)
- [注册事件 / 监听器](#registering-events-and-listeners)
- [定义事件](#defining-events)
- [定义监听器](#defining-listeners)
	- [队列事件监听器](#queued-event-listeners)
- [触发事件](#firing-events)
- [广播事件](#broadcasting-events)
	- [配置](#broadcast-configuration)
	- [标记事件广播](#marking-events-for-broadcast)
	- [广播数据](#broadcast-data)
	- [接收事件广播](#consuming-event-broadcasts)
- [事件订阅](#event-subscribers)

<a name="introduction"></a>
## 介绍

Laravel 事件提供一个简单的观察者模式的实现，允许你订阅和监听应用程序中的事件，事件类一般存储在 `app/Events` 目录中，而相应的监听器存储在 `app/Listeners`中。

<a name="registering-events-and-listeners"></a>
## 注册事件 / 监听器

Laravel 中的 `EventServiceProvider` 提供一个方便的地方，用于注册所有的事件监听器，`listen` 属性包含一个所有事件（键）和相应监听器（值）的数组，所以，你可以根据应用程序的需要添加事件到数组中，例如，让我们来添加 `PodcastWasPurchased` 事件：

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

### 生成事件 / 监听器类

当然，手动为每个事件和监听器生成文件很麻烦，另一种方式，你可以只添加监听器和事件到 `EventServiceProvider` 然后使用 `event:generate` 命令，这个命令将生成 `EventServiceProvider` 中列出的所有事件和监听器，当然，已经存在的事件和进监听器不包含在内：

	php artisan event:generate

<a name="defining-events"></a>
## 定义事件

一个事件类只是一个数据容器，持有与事件相关的信息，例如，假设一下我们生成的 `PodcastWasPurchased` 事件接受一个[Eloquent ORM](/docs/{{version}}/eloquent) 对象：

	<?php

	namespace App\Events;

	use App\Podcast;
	use App\Events\Event;
	use Illuminate\Queue\SerializesModels;

	class PodcastWasPurchased extends Event
	{
	    use SerializesModels;

	    public $podcast;

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

正如你所见，这个事件类没有特殊逻辑，只是所购买的 `Podcast` 对象的一个容器，如果事件对象被 PHP 的 `serialize` 函数序列化，事件类引入的 `SerializesModels` trait 将会优雅地将所有 Eloquent 模型序列化。

<a name="defining-listeners"></a>
## 定义监听器

接下来，让我们来看一下示例事件对应的监听器，事件监听器通过 `handle` 方法接收相应的事件实例，`event:generate` 命令将自动导入正确的事件类且在 `handle` 方法上类型提示（type-hint）相应的事件类。在 `handle` 方法中，你可以执行任何必要的逻辑来响应事件：

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

	    /**
	     * Handle the event.
	     *
	     * @param  PodcastWasPurchased  $event
	     * @return void
	     */
	    public function handle(PodcastWasPurchased $event)
	    {
	        // Access the podcast using $event->podcast...
	    }
	}

你的事件监听器还可以在构造函数上类型提示任何需要的依赖，所有的事件监听器都将通过 Laravel [服务容器](/docs/{{version}}/container) 来实例化，所以依赖将被自动注入：

	use Illuminate\Contracts\Mail\Mailer;

	public function __construct(Mailer $mailer)
	{
		$this->mailer = $mailer;
	}

#### 阻止事件传播

有时候，你可能希望组织事件传播到其他监听器，你可以通过从监听器的 `handle` 方法返回 `false` 来做到这一点。

<a name="queued-event-listeners"></a>
### 队列事件监听器

需要将事件监听器加入[队列](/docs/{{version}}/queues)吗? 这个再容易不过了，只需要将 `ShouldQueue` 接口加入监听器类。由 `event:generate` Artisan 命令生成的监听器类已经将这个接口导入到了当前的命名空间中，所以你可以直接使用：

	<?php

	namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		//
	}

就是这样！现在，当监听器因为一个事件而被调用时，这个监听器将会被使用自 Laravel[队列系统](/docs/{{version}}/queues)的事件转发器自动加入队列。如果当监听器被队列执行的过程当中没有报出异常，这个被队列的任务将自动害处理完后被删除。

#### 手动访问队列

If you need to access the underlying queue job's `delete` and `release` methods manually, you may do so. The `Illuminate\Queue\InteractsWithQueue` trait, which is imported by default on generated listeners, gives you access to these methods:

	<?php

	namespace App\Listeners;

	use App\Events\PodcastWasPurchased;
	use Illuminate\Queue\InteractsWithQueue;
	use Illuminate\Contracts\Queue\ShouldQueue;

	class EmailPurchaseConfirmation implements ShouldQueue
	{
		use InteractsWithQueue;

		public function handle(PodcastWasPurchased $event)
		{
			if (true) {
				$this->release(30);
			}
		}
	}

<a name="firing-events"></a>
## Firing Events

To fire an event, you may use the `Event` [facade](/docs/{{version}}/facades), passing an instance of the event to the `fire` method. The `fire` method will dispatch the event to all of its registered listeners:

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
		public function purchasePodcast($userId, $podcastId)
		{
			$podcast = Podcast::findOrFail($podcastId);

			// Purchase podcast logic...

			Event::fire(new PodcastWasPurchased($podcast));
		}
	}

Alternatively, you may use the global `event` helper function to fire events:

	event(new PodcastWasPurchased($podcast));

<a name="broadcasting-events"></a>
## Broadcasting Events

In many modern web applications, web sockets are used to implement real-time, live-updating user interfaces. When some data is updated on the server, a message is typically sent over a websocket connection to be handled by the client.

To assist you in building these types of applications, Laravel makes it easy to "broadcast" your events over a websocket connection. Broadcasting your Laravel events allows you to share the same event names between your server-side code and your client-side JavaScript framework.

<a name="broadcast-configuration"></a>
### Configuration

All of the event broadcasting configuration options are stored in the `config/broadcasting.php` configuration file. Laravel supports several broadcast drivers out of the box: [Pusher](https://pusher.com), [Redis](/docs/{{version}}/redis), and a `log` driver for local development and debugging. A configuration example is included for each of these drivers.

#### Broadcast Prerequisites

The following dependencies are needed for event broadcasting:

- Pusher: `pusher/pusher-php-server ~2.0`
- Redis: `predis/predis ~1.0`

#### Queue Prerequisites

Before broadcasting events, you will also need to configure and run a [queue listener](/docs/{{version}}/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected.

<a name="marking-events-for-broadcast"></a>
### Marking Events For Broadcast

To inform Laravel that a given event should be broadcast, implement the `Illuminate\Contracts\Broadcasting\ShouldBroadcast` interface on the event class. The `ShouldBroadcast` interface requires you to implement a single method: `broadcastOn`. The `broadcastOn` method should return an array of "channel" names that the event should be broadcast on:

	<?php

	namespace App\Events;

	use App\User;
	use App\Events\Event;
	use Illuminate\Queue\SerializesModels;
	use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

	class ServerCreated extends Event implements ShouldBroadcast
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
	     * Get the channels the event should be broadcast on.
	     *
	     * @return array
	     */
	    public function broadcastOn()
	    {
	        return ['user.'.$this->user->id];
	    }
	}

Then, you only need to [fire the event](#firing-events) as you normally would. Once the event has been fired, a [queued job](/docs/{{version}}/queues) will automatically broadcast the event over your specified broadcast driver.

<a name="broadcast-data"></a>
### Broadcast Data

When an event is broadcast, all of its `public` properties are automatically serialized and broadcast as the event's payload, allowing you to access any of its public data from your JavaScript application. So, for example, if your event has a single public `$user` property that contains an Eloquent model, the broadcast payload would be:

	{
		"user": {
			"id": 1,
			"name": "Jonathan Banks"
			...
		}
	}

However, if you wish to have even more fine-grained control over your broadcast payload, you may add a `broadcastWith` method to your event. This method should return the array of data that you wish to broadcast with the event:

    /**
     * Get the data to broadcast.
     *
     * @return array
     */
    public function broadcastWith()
    {
        return ['user' => $this->user->id];
    }

<a name="consuming-event-broadcasts"></a>
### Consuming Event Broadcasts

#### Pusher

You may conveniently consume events broadcast using the [Pusher](https://pusher.com) driver using Pusher's JavaScript SDK. For example, let's consume the `App\Events\ServerCreated` event from our previous examples:

	this.pusher = new Pusher('pusher-key');

	this.pusherChannel = this.pusher.subscribe('user.' + USER_ID);

	this.pusherChannel.bind('App\\Events\\ServerCreated', function(message) {
		console.log(message.user);
	});

#### Redis

If you are using the Redis broadcaster, you will need to write your own Redis pub/sub consumer to receive the messages and broadcast them using the websocket technology of your choice. For example, you may choose to use the popular [Socket.io](http://socket.io) library which is written in Node.

Using the `socket.io` and `ioredis` Node libraries, you can quickly write an event broadcaster to publish all events that are broadcast by your Laravel application:

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

	redis.on('pmessage', function(subscribed, channel, message) {
		message = JSON.parse(message);
		io.emit(channel + ':' + message.event, message.data);
	});

<a name="event-subscribers"></a>
## Event Subscribers

Event subscribers are classes that may subscribe to multiple events from within the class itself, allowing you to define several event handlers within a single class. Subscribers should define a `subscribe` method, which will be passed an event dispatcher instance:

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

		/**
		 * Register the listeners for the subscriber.
		 *
		 * @param  Illuminate\Events\Dispatcher  $events
		 * @return array
		 */
		public function subscribe($events)
		{
			$events->listen(
				'App\Events\UserLoggedIn',
				'App\Listeners\UserEventListener@onUserLogin'
			);

			$events->listen(
				'App\Events\UserLoggedOut',
				'App\Listeners\UserEventListener@onUserLogout'
			);
		}

	}

#### Registering An Event Subscriber

Once the subscriber has been defined, it may be registered with the event dispatcher. You may register subscribers using the `$subscribe` property on the `EventServiceProvider`. For example, let's add the `UserEventListener`.

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Events\Dispatcher as DispatcherContract;
    use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

    class EventServiceProvider extends ServiceProvider
    {
        /**
         * The event listener mappings for the application.
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
