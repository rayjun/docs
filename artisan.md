# Artisan Console

- [介绍](#introduction)
- [编写命令](#writing-commands)
    - [命令结构](#command-structure)
- [命令输入/输出](#command-io)
    - [定义输入要求](#defining-input-expectations)
    - [获取输入](#retrieving-input)
    - [提示输入](#prompting-for-input)
    - [编写输出](#writing-output)
- [注册命令](#registering-commands)
- [通过代码调用命令](#calling-commands-via-code)

<a name="introduction"></a>
## 介绍

Artisan 是 Laravel  中命令行接口的名字，提供了一些有用的命令供你在开发程序时使用，它是由强大的 Symfony 控制台（Console）模块来驱动的，查看可用的 Artisan 命令列表，可以使用 `list` 命令：

    php artisan list

每个命令还包括一个帮助屏，显示和描述命令的可用参数和选项，查看帮助屏，只用在命令的名字前加上 `help`：

    php artisan help migrate

<a name="writing-commands"></a>
## 编写命令

除 Artisan 提供的命令外，你还可以为你的应用程序构建你自定义的命令，可以将自定义的命令放置于 `app/Console/Commands` 目录，不过你可以自由选择存放路径，只要这些命令能根据 `composer.json` 中的定义被自动加载就行。

你可以使用 Artisan 的 `make:console` 创建一个新的命令，它会为你生成一个基本的命令类：

    php artisan make:console SendEmails

上面的命令将生一个 `app/Console/Commands/SendEmails.php`，在创建新命令时， `--command` 可以用来指定要创建的命令的名字：

    php artisan make:console SendEmails --command=emails:send

<a name="command-structure"></a>
### 命令结构

一旦命令被生成，你应该向这个类中加入 `signature` 和 `description` 属性，用于在 `list` 屏上显示该命令。

当命令被执行时 `handle` 方法会被调用，你可以向这个方法中加任何命令逻辑，让我们来看一个例子。

注意，我们可以向一个命令的构造函数中注入任何我们需要的依赖，Laravel [服务容器](/docs/{{version}}/container)将自动注入所有在构造函数中类型暗示的(type-hinted)依赖，为了使代码更具重用性，好的做法是保持命令类轻量，将实际的工作留给程序中服务类（Services）来完成。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;
    use Illuminate\Foundation\Inspiring;

    class Inspire extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * The drip e-mail service.
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * Create a new command instance.
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * Execute the console command.
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="command-io"></a>
## 命令输入/输出

<a name="defining-input-expectations"></a>
### 定义输入要求

当编写控制台命令时，通常命令的输入来自用户输入的参数或选项，Laravel 通过在命令上使用 `signature` 属性，使得定义你期望从用户获得输入的格式变得很方便，`signature` 允许你以一种单一，具有表述力，类似于路由的语法，为命令命名，定义参数及选项。

所有用户提供的参数和选择都包含在了花括号中，如下：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

在这个例子中，命令中定义了一个**必需**参数：`user`，你还可以让参数可选和给可选参数设定默认值：

    // Optional argument...
    email:send {user?}

    // Optional argument with default value...
    email:send {user=foo}

可选项（Options），像参数一样，也是一种用户输入，只不过他们在命令行上被用到时前面要加两个连字符 (`--`)，我们可以这样在命令签名（signature）上定义可选项：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在这个例子中，当运行 Artisan 命令时指定 `--queue` 开关，一旦传入了 `--queue` 开关，这个可选项的值将为 `true`，否则，它的值为 `false`:

    php artisan email:send 1 --queue

你还可以把这个可选项定义成用户必须赋值的，通过在这个选项的名称后台加一个 `=` 符号，表示需要提示一个值:

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在这个例子中，用户可以像这样给这个可选项传值：

    php artisan email:send 1 --queue=default

你还可以给可选项赋默认值：

    email:send {user} {--queue=default}

#### 输入描述

你可以通过使用冒号将参数和描述分隔开的方式给参数和选项添加描述：

    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="retrieving-input"></a>
### 获取输入

当你的命令在执行时，很显然你需要从你的命令中获取参数与选项的值，要这样做，你可以使用 `argument` 和 `option` 方法：

获取参数的值，使用 `argument` 方法：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果你需要以数组形式获取所有参数，调用 `argument` 不需要传入任何参数：

    $arguments = $this->argument();

用获 `option` 方法获取选项的值的方法跟获取参数的方法一样简单，像 `argument` 方法一样，为了以数组的方式获取所有选项的值，你可以调用无参的 `option` 方法：

    // Retrieve a specific option...
    $queueName = $this->option('queue');

    // Retrieve all options...
    $options = $this->option();

如果参数和选项不存在，将会返回 `null` 值。

<a name="prompting-for-input"></a>
### 提示输入

除了显示输出外，你还可以在命令执行的过程中要求用户输入信息。`ask` 会提示用户给定的询问，接收用户的输入，然后将用户的输入返回给命令：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法与 `ask` 相似，只是当用户在控制台中输入的信息不可见，这个方法对于要求用户输入敏感信息很有用，例如密码：

    $password = $this->secret('What is the password?');

#### 请求确认

如果你需要用户作一个简单的确认，你可以使用 `confirm` 方法，默认情况下，这个方法将返回 `false`，然而，如果在应答时输入`y`，该方法将返回 `true`。

    if ($this->confirm('Do you wish to continue? [y|N]')) {
        //
    }

#### 给用户选择

`anticipate` 方法可以用于自动补全可能的选择，用户仍然可以挑选其它任何答案，而不必理会这些选择。

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

如果你需要给用户提供一个预定义的选项集合，你可以使用 `choice` 方法。用户选择答案的索引，返回给你的是答案的值。你可以设置默认返回值，当什么也不选时返回：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], false);

<a name="writing-output"></a>
### 编写输出

要发送输出到控制台，请使用 `info`, `comment`, `question` and `error` 方法，这些方法将根据它们的意图返回适合的 ANSI 颜色。

要显示消息信息给用户，请使用 `info` 方法，通常，这个方法将以绿色文本的方式显示在控制台中：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

要显示错误消息，请使用 `error` 方法，出错消息的文本通常是用红色来显示的：

    $this->error('Something went wrong!');

#### 表格布局

`table` 方法使得正确地格式化多行 / 多列的数据变得简单，只要将表头（headers）和行（rows）传入该方法中，表格的宽和高将根据给定的数据被自动计算出来：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 进度条

对于运行时间长的任务来说，显示一个进度条会很有帮助。使用输入的对象，我们可以开始（start）, 前进（advance） 和 停止（stop）进度条，当你开启进度条时，你需要定义进度的数量，然后在每一步完成之后将进度条向前推进：

    $users = App\User::all();

    $this->output->progressStart(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $this->output->progressAdvance();
    }

    $this->output->progressFinish();

更多选项，请查看[Symfony 进度条模块文档](http://symfony.com/doc/2.7/components/console/helpers/progressbar.html).

<a name="registering-commands"></a>
## 注册命令

一旦你的命令编写完成，你需要将其注册到 Artisan 中以方便使用，这个定义 `app/Console/Kernel.php` 文件中。

在这个文件中，你会发现 `commands` 属性中有一个命令列表，注册命令，只需将命令的类名加入这个列表中，当 Artisan 启动时，所有列在 `commands` 属性中的命令都将被[服务容器](container) 实例化并注册到 Artisan 中：

    protected $commands = [
        'App\Console\Commands\SendEmails'
    ];

<a name="calling-commands-via-code"></a>
## 通过代码调用命令

有时你可能想在命令行之外执行 Artisan 命令，例如，你可能想从一个路由或是控制中启动一个 Artisan 命令，你可以调用 `Artisan` facade 上的 `call` 方法来做到这一点。这个 `call` 方法接收命令的名字作为第一个参数，还有一个命令参数数组作为第二个参数，退出代码（code）将会返回：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

使用 `Artisan` facade 上的 `queue` 方法，你甚至可以将 Artisan 命令加入队列，使得他们可以通过[queue workers](/docs/{{version}}/queues)在后台运行：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

If you need to specify the value of an option that does not accept string values, such as the `--force` flag on the `migrate:refresh` command, you may pass a booelan `true` or `false`:

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

### 在命令中调用命令

有时你可能想从一个已经存在的命令中调用其它命令，你可以通过调用 `call` 方法来做到这一点，`call` 接收命令的名字和一个命令参数数组作为参数：

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        $this->call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    }

如果想调用其它命令且抑制其所有的输出，你可以使用 `callSilent` 方法，这个方法与 `call` 方法具有相同的方法签名：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
