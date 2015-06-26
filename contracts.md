# Contracts

*   [简介](https://github.com/Laragirl/docs/blob/5.1/contracts.md#introduction)
*   [为何要用 Contracts?](https://github.com/Laragirl/docs/blob/5.1/contracts.md#why-contracts)
*   [Contract 参考](https://github.com/Laragirl/docs/blob/5.1/contracts.md#contract-reference)
*   [如何使用 Contracts](https://github.com/Laragirl/docs/blob/5.1/contracts.md#how-to-use-contracts)



## [简介](https://github.com/Laragirl/docs/blob/5.1/contracts.md#简介)

Laravel 的 Contracts 是一组定义了框架核心服务的介面（ interfaces ）。例如，`Illuminate\Contracts\Queue\Queue` contract 定义了队列任务所需要的方法，而 `Illuminate\Contracts\Mail\Mailer` contract 定义了寄送 e-mail 需要的方法。

框架对于每个 contract 都有提供对应的实作，例如，Laravel 提供各种驱动程式的队列实作，以及由 [SwiftMailer](http://swiftmailer.org/) 提供的 mailer 实作。

Laravel 所有的 contracts 都放在 [各自的 GitHub 储存库](https://github.com/illuminate/contracts)。除了提供所有可用的 contracts 一个快速的参考，也可以单独作为一个低藕合的套件让其他套件开发者使用。

### [](https://github.com/Laragirl/docs/blob/5.1/contracts.md#contracts-vs-facades)Contracts Vs. Facades

Laravel 的 [facades](https://github.com/Laragirl/docs/blob/5.1/docs/%7B%7Bversion%7D%7D/facades) 提供一个简单的方法来使用服务，而不需要使用型别提示和在服务容器之外解析 contracts。然而，使用 contracts 可以明显地定义出类别的依赖，对大部分应用程序而言，使用 facade 就很足够了，然而，若你实在需要特别的低藕合，使用 contracts 可以做到这一点，就让我们继续看下去！



## [为何要用 Contracts?](https://github.com/Laragirl/docs/blob/5.1/contracts.md#为何要用-contracts)

你可能有很多关于 contracts 的问题。像是为什麽要使用介面？使用介面会不会变的更复杂？

让我们用下面的标题来解释为什麽要使用介面：低藕合和简单性。

### [低藕合](https://github.com/Laragirl/docs/blob/5.1/contracts.md#低藕合)

首先，让我们来检视这一段和快取功能有高藕合的代码，如下：

~~~
<?php

namespace App\Orders;

class Repository
{
    /**
     * 快取功能.
     */
    protected $cache;

    /**
     * 建立一个新的仓库实体
     *
     * @param  \SomePackage\Cache\Memcached  $cache
     * @return void
     */
    public function __construct(\SomePackage\Cache\Memcached $cache)
    {
        $this->cache = $cache;
    }

    /**
     * 藉由 ID 取得订单资讯
     *
     * @param  int  $id
     * @return Order
     */
    public function find($id)
    {
        if ($this->cache->has($id)) {
            //
        }
    }
}

~~~

在此类别中，程式和快取实作之间是高藕合。因为它是依赖于套件库的特定快取类别。一旦这个套件的 API 更改了，我们的代码也要跟著改变。

同样的，如果想要将底层的快取技术（比如 Memcached ）抽换成另一种（像 Redis ），又一次的我们必须修改这个 repository 类别。我们的 储存库不应该知道这麽多关于谁提供了资料，或是如何提供等等细节。

**比起上面的做法，我们可以改用一个简单、和套件无关的介面来改进代码：**

~~~
<?php

namespace App\Orders;

use Illuminate\Contracts\Cache\Repository as Cache;

class Repository
{
    /**
     * 建立一个新的仓库实体
     *
     * @param  Cache  $cache
     * @return void
     */
    public function __construct(Cache $cache)
    {
        $this->cache = $cache;
    }
}

~~~

现在上面的代码没有跟任何套件藕合，甚至是 Laravel。既然 contracts 套件没有包含实作和任何依赖，你可以很简单的对任何 contract 进行实作，你可以很简单的写一个替换的实作，甚至是替换 contracts，让你可以替换快取实作而不用修改任何用到快取的代码。

### [简单性](https://github.com/laragirl/docs/blob/5.1/contracts.md#简单性)

当所有的 Laravel 服务都简洁的使用简单的介面定义，就能够很简单的决定一个服务需要提供的功能。 **可以将 contracts 视为说明框架特色的简洁文件。**

除此之外，当你依赖简洁的介面，你的代码能够很简单的被了解和维护。比起搜寻一个大型复杂的类别里有哪些可用的方法，你有一个简单，干净的介面可以参考。



## [Contract 参考](https://github.com/Laragirl/docs/blob/5.1/contracts.md#contract-参考)

以下是大部分 Laravel Contracts 的参考，以及相对应的「facade」：

| Contract | 对应的 Facade |
| --- | --- |
| [Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/master/Auth/Guard.php) | Auth |
| [Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/master/Auth/PasswordBroker.php) | Password |
| [Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/master/Bus/Dispatcher.php) | Bus |
| [Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/master/Broadcasting/Broadcaster.php) |
| [Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/master/Cache/Repository.php) | Cache |
| [Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/master/Cache/Factory.php) | Cache::driver() |
| [Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/master/Config/Repository.php) | Config |
| [Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/master/Container/Container.php) | App |
| [Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/master/Cookie/Factory.php) | Cookie |
| [Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/master/Cookie/QueueingFactory.php) | Cookie::queue() |
| [Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/master/Encryption/Encrypter.php) | Crypt |
| [Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/master/Events/Dispatcher.php) | Event |
| [Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/master/Filesystem/Cloud.php) |
| [Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/master/Filesystem/Factory.php) | File |
| [Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/master/Filesystem/Filesystem.php) | File |
| [Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/master/Foundation/Application.php) | App |
| [Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/master/Hashing/Hasher.php) | Hash |
| [Illuminate\Contracts\Logging\Log](https://github.com/illuminate/contracts/blob/master/Logging/Log.php) | Log |
| [Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/master/Mail/MailQueue.php) | Mail::queue() |
| [Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/master/Mail/Mailer.php) | Mail |
| [Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/master/Queue/Factory.php) | Queue::driver() |
| [Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/master/Queue/Queue.php) | Queue |
| [Illuminate\Contracts\Redis\Database](https://github.com/illuminate/contracts/blob/master/Redis/Database.php) | Redis |
| [Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/master/Routing/Registrar.php) | Route |
| [Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/master/Routing/ResponseFactory.php) | Response |
| [Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/master/Routing/UrlGenerator.php) | URL |
| [Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/master/Support/Arrayable.php) |
| [Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/master/Support/Jsonable.php) |
| [Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/master/Support/Renderable.php) |
| [Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/master/Validation/Factory.php) | Validator::make() |
| [Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/master/Validation/Validator.php) |
| [Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/master/View/Factory.php) | View::make() |
| [Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/master/View/View.php) |



## [如何使用 Contracts](https://github.com/laragirl/docs/blob/5.1/contracts.md#如何使用-contracts)

所以，要如何实作一个 contract 呢？实际上非常的简单。

很多 Laravel 的类别都是经由[服务容器](https://github.com/laragirl/docs/blob/5.1/docs/%7B%7Bversion%7D%7D/container) 来解析，包含控制器，事件监听，中介层，队列任务，甚至是路由闭包。所以，要实作一个 contract，你可以在类别的建构子使用「型别提示」解析类别。

例如，我们来看看这个事件监听程式：

~~~
<?php

namespace App\Listeners;

use App\User;
use App\Events\NewUserRegistered;
use Illuminate\Contracts\Redis\Database;

class CacheUserInformation
{
    /**
     * 实作 Redis 资料库
     */
    protected $redis;

    /**
     * 建立一个新的事件处理物件
     *
     * @param  Database  $redis
     * @return void
     */
    public function __construct(Database $redis)
    {
        $this->redis = $redis;
    }

    /**
     * 处理事件
     *
     * @param  NewUserRegistered  $event
     * @return void
     */
    public function handle(NewUserRegistered $event)
    {
        //
    }
}

~~~

当事件监听被解析时，服务容器会经由类别建构子参数的型别提示，注入适当的值。要知道怎麽注册更多服务容器，参考[这份文件](https://github.com/laragirl/docs/blob/5.1/docs/%7B%7Bversion%7D%7D/container).