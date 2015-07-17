# Database: Getting Started 开始

- [介绍](#introduction)
- [Running Raw SQL Queries](#running-queries)
    - [Listening For Query Events](#listening-for-query-events)
- [Database Transactions](#database-transactions)
- [Using Multiple Database Connections](#accessing-connections)

<a name="introduction"></a>
## 介绍

Laravel 非常简单地与数据库后端通过Raw SQL、[fluent query builder 查询生成器](/docs/{{version}}/queries) 以及 [Eloquent ORM](/docs/{{version}}/eloquent).连接和查询, 目前 Laravel支持四个数据库系统:

- MySQL
- Postgres
- SQLite
- SQL Server

<a name="configuration"></a>
### 配置

Laravel非常简单地与数据库连接和查询。应用数据库配置在`config/database.php`文件中。这个文件中你可以配置可支持的多个数据库连接方式,并指定默认连接。

默认情况下，Laravel 的[环境配置](/docs/{{version}}/installation#environment-configuration)是现成的 [Laravel Homestead](/docs/{{version}}/homestead)样例，
[Laravel Homestead] 是方便本地Laravel开发的虚拟机。当然，你可以自由的修改本地数据库配置。

<a name="read-write-connections"></a>
#### 读 / 写连接

有时候你希望使用一个数据连接处理查询语句，另外一个数据库连接处理插入、更新、和删除语句。Laravel 让它变得轻而易举，并且无论你使用的是 raw 查询，查询构造器，或是 Eloquent ORM ，都有适合的连接可供使用。

如何配置 读/写 连接，请看下面的例子：
example:

    'mysql' => [
        'read' => [
            'host' => '192.168.1.1',
        ],
        'write' => [
            'host' => '196.168.1.2'
        ],
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8',
        'collation' => 'utf8_unicode_ci',
        'prefix'    => '',
    ],

注意： `读` 和 `写` 两个键被添加在配置数组中，两个键都包含单一的键 `host` 的数组值。其余的数据库选项都合并在主 `mysql` 数组中。

所以，如果想要覆写主数组中的值，只需要把项放置 `读` and `写` 数组中。在这个案例中，`192.168.1.1` 将用于 "读" 连接，`192.168.1.2` 将用于 "写" 连接。数据库凭证、前缀、字符集设置和 `mysql`数组中的其它选项被两个连接所共享。

<a name="running-queries"></a>
## 执行 Raw SQL 查询

一旦配置了数据库连接，就可以使用 `DB` facade 执行查询。`DB` facade 提供各种查询方法，如 `select`, `update`, `insert`, and `statement`。

#### 执行一个 Select 查询

执行基本的查询，在 `DB` facade 上使用 `select` 方法：

    <?php

    namespace App\Http\Controllers;

    use DB;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Show a list of all of the application's users.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

`select` 方法的第一个参数是 raw SQL 查询，第二个参数是任意需要绑定查询的参数绑定。最有代表性地是 `where` 子句约束的值，参数绑定防止不利的 SQL 注入。

`select` 返回的是 `数组` 记录。数组内的每一条记录是一个 PHP `StdClass` 对象，允许存取记录的值：

    foreach ($users as $user) {
        echo $user->name;
    }

#### 使用命名绑定

可以使用命名绑定执行查询，替换`?`参数绑定：

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

#### 执行插入语句

在 `DB` facade 上使用 `insert` 方法，执行一条插入语句。和 `select` 一样，`insert` 方法的第一个参数是 raw SQL 查询，第二个参数是绑定：

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);

#### 执行一条更新语句

`update` 方法用于更新数据库中已有记录。该方法返回的是影响行数：

    $affected = DB::update('update users set votes = 100 where name = ?', ['John']);

#### 执行一条删除语句

`delete` 方法用于删除数据库中的记录。和 `update` 方法一样，返回影响行数：

    $deleted = DB::delete('delete from users');

#### 执行一般的语句

一些数据库语句没有返回值。对于这样的操作，可以在 `DB` facade 上使用 `statement` 方法：

    DB::statement('drop table users');

<a name="listening-for-query-events"></a>
### 查询事件监听

如果你想得到应用程序执行的每个SQL查询，可是使用 `listen` 方法。`listen` 方法对日记记录和调试非常有用。你可以注册查询监听到[服务供应者](/docs/{{version}}/providers)中:

    <?php

    namespace App\Providers;

    use DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         *
         * @return void
         */
        public function boot()
        {
            DB::listen(function($sql, $bindings, $time) {
                //
            });
        }

        /**
         * Register the service provider.
         *
         * @return void
         */
        public function register()
        {
            //
        }
    }

<a name="database-transactions"></a>
## 数据库事务

要在数据库事务执行一组操作，可以使用在 `DB` facade 上使用 `transaction` 方法。如果事务 `闭包` 内抛出异常，则事务自动回滚。如果闭包执行成功，则事务自动提交。当使用 `transaction` 方法时，你不需要为手动地回滚或提交而担心：

    DB::transaction(function () {
        DB::table('users')->update(['votes' => 1]);

        DB::table('posts')->delete();
    });

#### 手动使用事务

如果你喜欢手动开始一个事务和完全控制回滚和提交，可以在 `DB` facade 上使用 `beginTransaction` 方法：

    DB::beginTransaction();

You can rollback the transaction via the `rollBack` method:

    DB::rollBack();

Lastly, you can commit a transaction via the `commit` method:

    DB::commit();

> **主要:** 使用 `DB` facade 事务方法也可以控制 [查询构造器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent)事务。

<a name="accessing-connections"></a>
## 使用多种数据库连接

当使用多种数据库连接时，可以 `DB` facade 上使用 `connection` 方法取得每个连接。并把与 `config/database.php` 配置文件中的连接列表相应的一个值 `name` 传递给 `connection` 方法。

    $users = DB::connection('foo')->select(...);

也可以在 connection 实例上使用 `getPdo` 方法，取得raw, 基本的 PDO 实例：

    $pdo = DB::connection()->getPdo();
