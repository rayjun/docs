# Testing

<<<<<<< HEAD
- [简介](#introduction)
- [测试应用程式](#application-testing)
	- [与你的应用程式进行互动](#interacting-with-your-application)
	- [测试 JSON APIs](#testing-json-apis)
	- [Sessions 和认证](#sessions-and-authentication)
	- [停用中介层](#disabling-middleware)
	- [自订 HTTP 请求](#custom-http-requests)
- [使用资料库](#working-with-databases)
	- [每次测试结束后重置资料库](#resetting-the-database-after-each-test)
	- [模型工厂](#model-factories)
- [模拟](#mocking)
	- [模拟事件](#mocking-events)
	- [模拟任务](#mocking-jobs)
	- [模拟 Facades](#mocking-facades)
=======
- [Introduction](#introduction)
- [Application Testing](#application-testing)
    - [Interacting With Your Application](#interacting-with-your-application)
    - [Testing JSON APIs](#testing-json-apis)
    - [Sessions / Authentication](#sessions-and-authentication)
    - [Disabling Middleware](#disabling-middleware)
    - [Custom HTTP Requests](#custom-http-requests)
- [Working With Databases](#working-with-databases)
    - [Resetting The Database After Each Test](#resetting-the-database-after-each-test)
    - [Model Factories](#model-factories)
- [Mocking](#mocking)
    - [Mocking Events](#mocking-events)
    - [Mocking Jobs](#mocking-jobs)
    - [Mocking Facades](#mocking-facades)
>>>>>>> laravel/5.1

<a name="introduction"></a>
## 简介

Laravel 在建立时就已考虑到测试的部分。事实上，预设就支援以 PHPUnit 做测试，而且已经为你的应用程式建立了`phpunit.xml` 档案。框架还提供了一些便利的辅助方法，让你更直观的测试你的应用程式。

在 `tests` 目录中有提供一个 `ExampleTest.php` 的范例档案。安装新的 Laravel 应用程式之后，只要在命令列上执行`phpunit`，就可以进行测试。

### 测试环境

在执行测试时，Laravel 会自动将环境变数设定为 `testing`，并将 Session 及快取存入`阵列`形式，也就是说在测试时不会有任何 Session 或快取资料被储存。

你可以自由建立其他必要的测试环境设定。`testing` 的环境变数可以在 `phpunit.xml` 档案中做修改。.

### 定义并执行测试

要建立一个测试案例，只需要在 `tests` 目录下建立新的测试档案。测试类别必须继承 `TestCase`，接著就可以像平常使用 PHPUnit 一样定义测试方法。要执行测试只需要在终端机上执行 `phpunit` 指令：

    <?php

    class FooTest extends TestCase
    {
        public function testSomethingIsTrue()
        {
            $this->assertTrue(true);
        }
    }

> **注意：** 如果你要在你的类别定义自己的 `setUp` 方法，请确定有呼叫 `parent::setUp`。

<a name="application-testing"></a>
## 测试应用程式

Laravel 对于产生 HTTP 请求并送至应用程式，检查输出，甚至填写表单，都提供了非常流利的 API。 举例来说，你可以看看 `tests` 目录中的 `ExampleTest.php` 档案：

    <?php

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
                 ->see('Laravel 5')
                 ->dontSee('Rails');
        }
    }

`visit` 方法会制造 `GET` 请求至应用程式，`see` 方法则断言在应用程式回传的回应中，有指定的文字。这是 Laravel 所提供最基本的应用程式测试。

<a name="interacting-with-your-application"></a>
### 与你的应用程式进行互动

当然，除了断言文字是否存在于一个给定的回应，你可以做更多的互动。让我们看看点击连结及填写表单的范例：

#### 点击连结

在这个测试中，我们会产生一个请求送到应用程式，并「点击」回传回应中的连结，接著断言我们会停留在指定的 URI。举个例子，假设在回应中有个连结，并写著「About Us」：

    <a href="/about-us">About Us</a>

现在，我们撰写一个测试，点击连结并断言使用者会停留在正确的页面：

    public function testBasicExample()
    {
        $this->visit('/')
             ->click('About Us')
             ->seePageIs('/about-us');
    }

#### 使用表单

Laravel 还提供了几种用于测试表单的方法。透过 `type`、`select`、`check`、`attach` 及 `press` 方法让你与表单所有的输入栏进行互动。举个例子，让我们想像一下有个表单在应用程式的注册页面：

    <form action="/register" method="POST">
        {!! csrf_field() !!}

        <div>
            Name: <input type="text" name="name">
        </div>

        <div>
            <input type="checkbox" value="yes" name="terms"> Accept Terms
        </div>

        <div>
            <input type="submit" value="Register">
        </div>
    </form>

我们可以撰写一个测试来填写此表单，并检查结果：

    public function testNewUserRegistration()
    {
        $this->visit('/register')
             ->type('Taylor', 'name')
             ->check('terms')
             ->press('Register')
             ->seePageIs('/dashboard');
    }

当然，如果你的表单包含像是单选栏或下拉式选单的其他输入栏，你也可以很轻鬆的填入这些类型的区域。以下是每个表单操作方法的列表：

| 方法 | 说明 |
| --- | --- |
| `$this->type($text, $elementName)` | 「输入（type）」文字在一个给定的区域 |
| `$this->select($value, $elementName)` | 「选择（select）」一个单选栏或下拉式选单的区域 |
| `$this->check($elementName)` | 「勾选（Check）」一个複选栏的区域 |
| `$this->attach($pathToFile, $elementName)` | 「附上（Attach）」一个档案至表单 |
| `$this->press($buttonTextOrElementName)` | 「按下（Press）」一个指定文字或名称的按钮 |

#### 使用附件

如果你的表单包含 `file` 的输入栏类型，可以使用 `attach` 方法附上档案：

    public function testPhotoCanBeUploaded()
    {
        $this->visit('/upload')
             ->name('File Name', 'name')
             ->attach($absolutePathToFile, 'photo')
             ->press('Upload')
             ->see('Upload Successful!');
    }

<a name="testing-json-apis"></a>
### 测试 JSON APIs

Laravel 也提供了几个辅助方法测试 JSON APIs 及其回应。举例来说，`get`、`post`、`put`、`patch` 及 `delete` 方法可以用于发出各种 HTTP 动词的请求。你也可以很轻鬆的传入资料或是标头到这些方法。首先，我们撰写一个测试，发出`POST` 请求至 `/user` ，并断言会回传JSON 格式的指定阵列：

    <?php

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

`seeJson` 方法会将传入的阵列转换成 JSON，并验证该 JSON 片段是否存在于应用程式回传的 JSON 回应中的**任何位置**。也就是说，即使有其他的属性在 JSON 回应中，但是只要给定的片段存在，此测试仍然会通过。

#### 验证 JSON 完全匹配

如果你想验证传入的阵列要与应用程式回传的 JSON **完全**匹配，可以使用 `seeJsonEquals` 方法：

    <?php

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

<a name="sessions-and-authentication"></a>
### Sessions 和认证

Laravel 提供了几个辅助方法在测试时使用 Session。首先，你需要设定 Session 资料至给定的阵列，并使用 `withSession`方法。对于要测试送到应用程式的请求之前，先将资料载入 session 上相当有用：

    <?php

    class ExampleTest extends TestCase
    {
        public function testApplication()
        {
            $this->withSession(['foo' => 'bar'])
                 ->visit('/');
        }
    }

当然，一般使用 Session 时都是用于保持使用者的状态，像是认证使用者。`actingAs` 辅助方法提供了简单的方式，让指定的使用者认证为当前的使用者。举个例子，我们可以使用[模型工厂](/docs/{{version}}/testing.md#model-factories)来产生并认证使用者：

    <?php

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

<a name="disabling-middleware"></a>
### 停用中介层
测试应用程式时，你会发现，在某些测试中停用[中介层](/docs/{{version}}/middleware)是很方便的。让你可以隔离任何中介层的影响，来测试路由及控制器。Laravel 包含一个简洁的 `WithoutMiddlewate` trait，你能够使用它自动在测试类别中停用所有的中介层：

    <?php

    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Illuminate\Foundation\Testing\DatabaseTransactions;

    class ExampleTest extends TestCase
    {
        use WithoutMiddleware;

        //
    }

如果你只想要在某几个测试方法中停用中介层，你可以在测试方法中呼叫 `withoutMiddleware` 方法：

    <?php

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

<a name="custom-http-requests"></a>
### 自订 HTTP 请求

如果你想要建立一个自订的 HTTP 请求至你的应用程式，并取得完整的 `Illuminate\Http\Response` 物件，你可以使用`call` 方法：

    public function testApplication()
    {
        $response = $this->call('GET', '/');

        $this->assertEquals(200, $response->status());
    }

I如果你建立的是 `POST`、`PUT`、或是 `PATCH` 请求，你可以在请求时传入一个阵列作为输入资料。当然，你可在你的路由及控制器中透过[请求实例](/docs/{{version}}/requests)取用这些资料:

       $response = $this->call('POST', '/user', ['name' => 'Taylor']);

<a name="working-with-databases"></a>
##使用资料库

Laravel 也提供了多种有用的工具，让你更容易测试使用资料库的应用程式。首先，你可以使用 `seeInDatabase` 辅助方法，来断言资料库中是否存在与指定条件互相匹配的资料。举例来说，如果我们想验证 `users` 资料表中是否存在 `email`值为 `sally@example.com` 的资料，我们可以按照以下的方式：

    public function testDatabase()
    {
        // Make call to application...

        $this->seeInDatabase('users', ['email' => 'sally@foo.com']);
    }

当然，使用 `seeInDatabase` 方法及其他的辅助方法只是基于方便。你可以自由使用任何 PHPUnit 内建的断言方法来扩充你的测试。

<a name="resetting-the-database-after-each-test"></a>
### 每次测试结束后重置资料库

在每次测试结束后都重置你的资料是相当有用的，这样前次的测试资料就不会干扰之后的测试。

#### 使用迁移

其中一种方式就是在每次测试后都还原资料库，并在下次测试前进行迁移。Laravel 提供了简洁的 `DatabaseMigrations`trait，它会自动帮你处理这些操作。你只需要在测试类别中使用此 trait：

    <?php

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

#### 使用交易

另一个方式，就是将每个测试案例包装在资料库交易中。当然，Laravel 提供了一个简洁的 `DatabaseTransactions` trait，它会自动帮你处理这些操作：

    <?php

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

<a name="model-factories"></a>
### 模型工厂

测试时，常常需要在执行测试之前写入一些资料到资料库中。建立测试资料时，除了手动设定每个栏位的值，Laravel 让你可以使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的「工厂」设定每个属性的预设值。开始之前，你可以查看应用程式的`database/factories/ModelFactory.php` 档案。此档案包含一个现成的工厂定义：

    $factory->define(App\User::class, function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
        ];
    });

闭包内为工厂的定义，你可以回传模型所有属性的预设测试值。该闭包内会接收到 [Faker](https://github.com/fzaninotto/Faker) PHP 函式库的实例，它可以让你方便的产生各种随机的资料以进行测试。

当然，你可以自由的增加自己额外的工厂至 `ModelFactory.php` 档案。

#### 多个工厂类型

有时你可能希望针对同一个 Eloquent 模型类别，能建立多个工厂。例如，除了一般使用者的工厂之外，还有「管理员」的工厂。你可以使用 `defineAs` 方法定义这个工厂：

    $factory->defineAs(App\User::class, 'admin', function ($faker) {
        return [
            'name' => $faker->name,
            'email' => $faker->email,
            'password' => str_random(10),
            'remember_token' => str_random(10),
            'admin' => true,
        ];
    });

除了从一般使用者工厂複製所有基底属性，你也可以使用 `raw` 方法来取得基底属性。一旦你取得这些属性，就可以轻鬆的增加额外任何你需要的值：

    $factory->defineAs(App\User::class, 'admin', function ($faker) use ($factory) {
        $user = $factory->raw(App\User::class);

        return array_merge($user, ['admin' => true]);
    });

#### 在测试中使用工厂

定义工厂后，就可以在测试或是资料库填充档案中，透过全域 `factory` 函式产生模型实例。那麽让我们来看看几个建立模型的例子。首先我们会使用 `make` 方法建立模型，但是不将它们储存至资料库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->make();

        // Use model in tests...
    }

如果你想覆写模型中的某些预设值，你可以传递一个包含数值的阵列至 `make` 方法。只有指定的数值会被替换，其它剩馀的数值则会按照工厂指定的预设值设定：

    $user = factory(App\User::class)->make([
        'name' => 'Abigail',
       ]);

你也可以建立一个含有多个模型的集合，或建立一个给定类型的模型：

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->make();

    // Create an App\User "admin" instance...
    $user = factory(App\User::class, 'admin')->make();

    // Create three App\User "admin" instances...
    $users = factory(App\User::class, 'admin', 3)->make();

#### 保存工厂模型

`create` 方法不仅建立模型实例，也可以使用 Eloquent 的 `save` 方法将它们储存至资料库：

    public function testDatabase()
    {
        $user = factory(App\User::class)->create();

        // Use model in tests...
    }

同样的，你可以在传递阵列至 `create` 方法时覆写模型的属性：

    $user = factory(App\User::class)->create([
        'name' => 'Abigail',
       ]);

#### 增加关联至模型

你甚至能保存多个模型至资料库。在本例中，我们还会增加关联至我们建立的模型。当使用 `create` 方法建立多个模型时，它会回传一个 Eloquent [集合实例](/docs/{{version}}/eloquent-collections)，让你能够使用集合所提供的方便函式，像是 `each`：

    $users = factory(App\User::class, 3)
               ->create()
               ->each(function($u) {
                    $u->posts()->save(factory(App\Post::class)->make());
                });

<a name="mocking"></a>
## 模拟

<a name="mocking-events"></a>
### 模拟事件

如果你正在频繁地使用 Laravel 的事件系统，你可能希望在测试时停止或是模拟某些事件。举例来说，如果你正在测试使用者注册功能，你可能不希望所有 `UserRegistered` 事件的处理程序被执行，因为它们会发送「欢迎」的电子邮件等等。

Laravel 提供了简洁的 `expectsEvents` 方法，验证预期的事件有被执行，但是防止该事件的任何处理程序被执行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->expectsEvents(App\Events\UserRegistered::class);

            // Test user registration code...
        }
    }


如果你希望防止所有事件的处理程序被执行，你可以使用 `withoutEvents` 方法：

    <?php

    class ExampleTest extends TestCase
    {
        public function testUserRegistration()
        {
            $this->withoutEvents();

            // Test user registration code...
        }
    }

<a name="mocking-jobs"></a>
### 模拟任务

有时你可能希望当发出请求至你的应用程式时，简单的测试由你的控制器所派送的任务。这麽做能够使你隔离并测试你的路由或控制器－设定除了任务以外的逻辑。当然，你之后可以在一个单独的测试案例测试该任务。

Laravel 提供了一个简洁的 `expectsJobs` 方法，验证预期的任务有被派送，但是任务本身不会被执行：

    <?php

    class ExampleTest extends TestCase
    {
        public function testPurchasePodcast()
        {
            $this->expectsJobs(App\Jobs\PurchasePodcast::class);

            // Test purchase podcast code...
        }
    }

> **注意：** 此方法只检测被 `DispatchesCommands` trait 的派送方法所派送的任务。它并不会检测直接发送到 `Queue::push`的任务。

<a name="mocking-facades"></a>
### 模拟 Facades

测试时，你可能时常需要模拟呼叫一个 Laravel [facade](/docs/{{version}}/facades). 。举个例子，参考下方的控制器行为：

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

我们可以透过 `shouldReceive` 方法模拟呼叫 `Cache` facade，它会回传一个 [Mockery](https://github.com/padraic/mockery) 模拟的实例。因为 facades 实际上已经被 Laravel [服务容器](/docs/{{version}}/container)解决及管理，它们比起一般的静态类别更有可测试性。举个例子，让我们模拟我们呼叫 `Cache`facade：

    <?php

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

> **注意：** 你不应该模拟 `Request` facade。应该在测试时使用像是 `call` 及 `post` 的 HTTP 辅助方法来传递你想要的资料。
