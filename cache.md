# Cache

- [配置](#configuration)
- [缓存使用](#cache-usage)
	- [获取一个缓存实例](#obtaining-a-cache-instance)
	- [从缓存中检索缓存项](#retrieving-items-from-the-cache)
	- [存储缓存项](#storing-items-in-the-cache)
	- [移除缓存项](#removing-items-from-the-cache)
- [添加定义缓存驱动](#adding-custom-cache-drivers)

<a name="configuration"></a>
## 配置

Laravel 为各种缓存系统提供了统一的 API, 缓存的配置放置于 `config/cache.php` 中，在这个文件中你可以指定在整个应用程序中默认被使用的缓存驱动，Laravel 支持流行的缓存后端程序如[Memcached](http://memcached.org/) 和[Redis](http://redis.io/) ，开箱即用。

缓存配置文件还包含各种其它的选项，这些选项在文件中都有注释，一定要读一遍这些选项。默认情况下，Laravel 配置的是使用 `file` 缓存驱动，存储序列化的，缓存的对象到文件系统中。对于较大的应用程序，推荐你使用一个内存（in-memory）缓存，例如 Memcached 或者 APC。你甚至可以为同一个驱动配置多个缓存配置。

### 缓存必备条件

#### 数据库

当使用 `database` 缓存驱动，你需要建一个表来存储缓存项，以下是这个表 `Schema` 申明示例：


	Schema::create('cache', function($table) {
		$table->string('key')->unique();
		$table->text('value');
		$table->integer('expiration');
	});

#### Memcached

使用 Memcached 缓存需要先安装[Memcached PECL 包](http://pecl.php.net/package/memcached)。.

默认[配置](cache#configuration) 使用基于 TCP/IP 的[Memcached::addServer](http://php.net/manual/en/memcached.addserver.php):

	'memcached' => [
		[
			'host' => '127.0.0.1',
			'port' => 11211,
			'weight' => 100
		],
	],

你还可以将 `host` 设置为 UNIX 套接字（socket）路径，如果你这样做，`port` 应该设置为 `0`:

	'memcached' => [
		[
			'host' => '/var/run/memcached/memcached.sock',
			'port' => 0,
			'weight' => 100
		],
	],

#### Redis

Before using a Redis cache with Laravel, you will need to install the `predis/predis` package (~1.0) via Composer.

For more information on configuring Redis, consult its [Laravel documentation page](/docs/{{version}}/redis#configuration).

<a name="cache-usage"></a>
## 缓存用法

<a name="obtaining-a-cache-instance"></a>
### 获取一个缓存实例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache\Repository` [contracts](/docs/{{version}}/contracts) 提供对 Laravel 的缓存服务的访问，`Factory` 提供对配置文件中定义的所有缓存驱动的访问。`Repository` Contract 通常是一个对 `cache` 配置文件中所指定的，应用程序默认缓存驱动的一个实现。

然而，你还可以使用 `Cache` facade，这是我们将在整个文档中使用的方式。`Cache` facade 提供方便，简洁的方式，访问 Laravel 缓存 Contracts 的底层实现。

例如，让我们导入 `Cache` facade 到一个控制器中：

	<?php

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

#### 访问多个缓存存储

使用 `Cache` facade，你可以通过 `store` 方法访问各种缓存存储，传入到 `store` 中的键值应该对应于 `cache` 配置文件中 `stores` 数组中的一个：

	$value = Cache::store('file')->get('foo');

	Cache::store('redis')->put('bar', 'baz', 10);

<a name="retrieving-items-from-the-cache"></a>
### 从缓存中检索缓存项

`Cache` facade 上的 `get` 方法被用于从缓存中检索获取缓存项，如果缓项在缓存中不存在，将返回 `null` 值，如果你想在缓存项不存在的时返回一个你自定义的默认值，你可以向 `get` 方法传入第二个参数来指定：

	$value = Cache::get('key');

	$value = Cache::get('key', 'default');


你甚至可以传入一个 `Closure` 作为默认值，如果指定的缓存项在缓存中不存，将返回 `Closure` 的结果，传入闭包使得你可以将默认值的获取委托给数据库或其它外部服务：

	$value = Cache::get('key', function() {
		return DB::table(...)->get();
	});

#### 检查缓存项是否存在

`has` 方法可能被用来判断一个缓存项在缓存中是否存在：

	if (Cache::has('key')) {
		//
	}

#### 自增 / 自减值

`increment` 和 `decrement` 方法可以被用来调整缓存中的整数型缓存项的大小，两个方法都可以接收第二个参数表示增加或减少量：


	Cache::increment('key');

	Cache::increment('key', $amount);

	Cache::decrement('key');

	Cache::decrement('key', $amount);

#### 检索或者更新

有时你可能想从缓存中获取一个缓存项，当请求的缓存项不存在时存储一个默认值，例如，你可能想从缓存中获取所有的用户，当它们不存在时，从数据库中检索出来并且将其加入到缓存中，你可以用 `Cache::remember` 方法做到这一点：

	$value = Cache::remember('users', $minutes, function() {
		return DB::table('users')->get();
	});

如果这个缓存项在缓存中不存在，传入 `remember` 方法中的 `Closure` 将会被执行且其结果会被放到缓存中。

你还可以将 `remember` 和 `forever` 方法合并：


	$value = Cache::rememberForever('users', function() {
		return DB::table('users')->get();
	});

#### 检索和删除

如果你想从缓存中检索出一个缓存项然后再将其删除，你可以使用 `pull` 方法，跟 `get` 方法一样，如果缓存中不存在此缓存项将返回 `null`：

	$value = Cache::pull('key');

<a name="storing-items-in-the-cache"></a>
### 存储缓存项

你可以使用 `Cache` facade 上的 `put` 的方法来将缓存项存储到缓存中，当加入缓存项时，你需要指定缓存项将被缓存多少分钟：

	Cache::put('key', 'value', $minutes);

你还可以传入一个 PHP `DateTime` 的实例来表示缓存项的过期时间，而不是多少分钟后过期：

	$expiresAt = Carbon::now()->addMinutes(10);

	Cache::put('key', 'value', $expiresAt);

`add` 方法只有当缓存项还不存在时才会将其添加到缓存存储中，如果被实际添加将返回 `true`，否则，返回 `false`:

	Cache::add('key', 'value', $minutes);

`forever` 可以将缓存项永久性地存储到缓存中，这些值必须使用 `forget` 方法手动将其从缓存中移除：

	Cache::forever('key', 'value');

<a name="removing-items-from-the-cache"></a>
### 移除缓存项

你可以使用 `Cache` facade 上的 `forget` 方法将缓存项从缓存中移除：

	Cache::forget('key');

<a name="adding-custom-cache-drivers"></a>
## 添加定义缓存驱动

给 Laravel 缓存扩展一个自定义的驱动，我们将使用 `Cache` facade 上的 `extend` 方法，将实例化自定义驱动绑定到驱动管理器上，通常，这个由[服务提供者](/docs/{{version}}/providers)来做的。

例如，注册一个名为 "mongo" 的新缓存驱动：

	<?php

	namespace App\Providers;

	use Cache;
	use App\Extensions\MongoStore;
	use Illuminate\Support\ServiceProvider;

	class CacheServiceProvider extends ServiceProvider
	{
		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Cache::extend('mongo', function($app) {
				return Cache::repository(new MongoStore);
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

传入 `extend` 方法的第一个参数是驱动器的名字，这个将对应于你在 `config/cache.php` 配置文件中定义的 `driver` 选项，第二个参数是一个闭包函数，应该返回一个 `Illuminate\Cache\Repository` 实例，此闭包函数将被传入一个 `$app`，是[服务容器](/docs/{{version}}/container)的一个实例：

对 `Cache::extend` 调用可以放在默认的 `App\Providers\AppServiceProvider` 的 `boot` 方法中，这个服务提供类是 Laravel 自带的，或者你可以创建你自己的服务提供者来控制这个缓存扩展，只是别忘了将这个提供者类注册类到 `config/app.php` 的提供者数组中。

要创建我们自己的缓存驱动，首先需要实现 `Illuminate\Contracts\Cache\Store`[Contract](contracts),所以，我们的 MongoDB 缓存实现应该像这样：

The first argument passed to the `extend` method is the name of the driver. This will correspond to your `driver` option in the `config/cache.php` configuration file. The second argument is a Closure that should return an `Illuminate\Cache\Repository` instance. The Closure will be passed an `$app` instance, which is an instance of the [service container](/docs/{{version}}/contracts).

	<?php

	namespace App\Extensions;

	class MongoStore implements \Illuminate\Contracts\Cache\Store
	{
		public function get($key) {}
		public function put($key, $value, $minutes) {}
		public function increment($key, $value = 1) {}
		public function decrement($key, $value = 1) {}
		public function forever($key, $value) {}
		public function forget($key) {}
		public function flush() {}
	}

我们只需要使用 MongoDB 连接来实现其中的每一个方法，一旦开发完，我们可以完成对自定义驱动的注册：

	Cache::extend('mongo', function($app) {
		return Cache::repository(new MongoStore);
	});

一旦你的扩展完成，只需要将你的 `config/cache.php` 配置文件中的 `driver` 选项，更新为此扩展的名字即可。

如果你正在考虑将你自定义的驱动代码放在哪里，可以考虑将其放到 Packagist 上! 或者，你可以在你的 `app` 目录中创建一个 `Extensions` 命名空间。然后，请记住，Laravel 没有一个严格规则的应用程序目录结构，你可以根据你的喜好自由组织你的应用程序。