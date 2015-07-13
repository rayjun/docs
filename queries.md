# Database: Query Builder

<<<<<<< HEAD
- [简介](#introduction)
- [检索记录](#retrieving-results)
	- [聚合](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where 子句](#where-clauses)
	- [高级的 Where 子句](#advanced-where-clauses)
=======
- [Introduction](#introduction)
- [Retrieving Results](#retrieving-results)
    - [Aggregates](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where Clauses](#where-clauses)
    - [Advanced Where Clauses](#advanced-where-clauses)
>>>>>>> laravel/5.1
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [Pessimistic Locking](#pessimistic-locking)

<a name="introduction"></a>
## 简介

数据库查询构造器为创建和执行数据库查询提供了方便的、流畅的接口。该接口能够在应用程序中执行大多数的数据库操作，并在所有受支持的数据库系统上工作。

> **注意:** laravel 查询构造器使用 PDO 参数绑定,防止应用程序受不利的 SQL 注入攻击。不需要清除字符串传递绑定。

<a name="retrieving-results"></a>
## 检索记录

#### 检索一张表的所有记录

要开始一个流畅的查询，需要使用 `DB` facade 的 `table` 方法。`table` 方法将返回一个流畅的查询构造器实例给指定的表，
可以在查询上链接更多的约束，并最终获取到结果。在这个例子中，只是`获取`一张表的所有记录：

<<<<<<< HEAD
~~~
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
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
~~~
=======
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
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }
>>>>>>> laravel/5.1

如同[原生查询](/docs/{{version}}/database),`get` 方法返回一个结果集数组，每一个结果集是  PHP `StdClass` 对象的一个实例。你可以像访问对象属性那样访问列来存取列的值：

<<<<<<< HEAD
~~~
foreach ($users as $user) {
    echo $user->name;
}
~~~
=======
    foreach ($users as $user) {
        echo $user->name;
    }
>>>>>>> laravel/5.1

#### 检索一张表的一行/列

如果你仅仅需要检索数据库中的一行记录，可以使用 `first` 方法。`first` 方法返回单一的 `StdClass` 对象：

<<<<<<< HEAD
~~~
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
~~~
=======
    $user = DB::table('users')->where('name', 'John')->first();

    echo $user->name;
>>>>>>> laravel/5.1

如果你甚至不需要一整行记录，可以使用 `value` 方法从记录中取出一个单一的值。该方法直接返回这个列的值：

<<<<<<< HEAD
~~~
$email = DB::table('users')->where('name', 'John')->value('email');
~~~
=======
    $email = DB::table('users')->where('name', 'John')->value('email');
>>>>>>> laravel/5.1

#### 分割表结果

如果你需要处理成千上万的数据库记录，可以考虑使用 `chunk` 方法。`chunk` 方法每次只取回记录中的一小 "块"，并把每一块提供 `闭包` 中处理。该方法非常有益于编写处理成千上万的记录[Artisan 命令](/docs/{{version}}/artisan)。例如，每次处理整张 `users` 表中每块为一百条的记录：

<<<<<<< HEAD
~~~
DB::table('users')->chunk(100, function($users) {
    foreach ($users as $user) {
        //
    }
});
~~~
=======
    DB::table('users')->chunk(100, function($users) {
        foreach ($users as $user) {
            //
        }
    });
>>>>>>> laravel/5.1

你可以通过 `闭包` 返回 `false` 来停止处理下一个块：

<<<<<<< HEAD
~~~
DB::table('users')->chunk(100, function($users) {
    // Process the records...

    return false;
});
~~~
=======
    DB::table('users')->chunk(100, function($users) {
        // Process the records...

        return false;
    });
>>>>>>> laravel/5.1

#### 检索一串列的值

如果你喜欢取得包含各列值得数组，你可以使用 `lists` 方法。在这个例子中，我们将取得 role 的 titles 数组：

<<<<<<< HEAD
~~~
$titles = DB::table('roles')->lists('title');

foreach ($titles as $title) {
    echo $title;
}
~~~
=======
    $titles = DB::table('roles')->lists('title');

    foreach ($titles as $title) {
        echo $title;
    }
>>>>>>> laravel/5.1

你可以在返回的数组中指定一个自定义的键列：

<<<<<<< HEAD
~~~
$roles = DB::table('roles')->lists('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
~~~

=======
    $roles = DB::table('roles')->lists('title', 'name');

    foreach ($roles as $name => $title) {
        echo $title;
    }
>>>>>>> laravel/5.1

<a name="aggregates"></a>
### 聚合

查询构造器也提供了各种各样的聚合方法，诸如 `count`, `max`, `min`, `avg`, 和 `sum`。你可以调用任何一个方法在构造查询之后：

~~~
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
~~~

<<<<<<< HEAD

当然了，你可以联结其他的子句创建你的查询：
=======
    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');
>>>>>>> laravel/5.1

~~~
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
~~~

<<<<<<< HEAD
=======
    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');
>>>>>>> laravel/5.1

<a name="selects"></a>
## Selects

#### 检索一个查询子句

当然，你并不是每次都要查询数据库中所有的列。使用`select` 方法，查询指明要的 `select` 子句：

<<<<<<< HEAD
~~~
$users = DB::table('users')->select('name', 'email as user_email')->get();
~~~
=======
    $users = DB::table('users')->select('name', 'email as user_email')->get();
>>>>>>> laravel/5.1

`distinct` 方法强行返回 distinct 记录：

<<<<<<< HEAD
~~~
$users = DB::table('users')->distinct()->get();
~~~
=======
    $users = DB::table('users')->distinct()->get();
>>>>>>> laravel/5.1

如果已存在查询构造器实例，并想要在当前的 select 子句加入一列，可以使用 `addSelect` 方法，：

<<<<<<< HEAD
~~~
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
~~~
=======
    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();
>>>>>>> laravel/5.1

#### 原生表达式

有时候我们需要在查询中使用原生表达式。这些表达式以字符串的形式注入到查询中，所有注意不能创建任何的 SQL注入点！使用 `DB::raw` 方法，创建原生表达式：

~~~
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
~~~

<<<<<<< HEAD
=======
    $users = DB::table('users')
                         ->select(DB::raw('count(*) as user_count, status'))
                         ->where('status', '<>', 1)
                         ->groupBy('status')
                         ->get();
>>>>>>> laravel/5.1

<a name="joins"></a>
## Joins

#### Inner Join 语句

查询构造器也可以用来写 join 语句。执行基本的 SQL "inner join"，需要在查询构造器实例上使用 `join` 方法。`join` 方法的第一个参数传递你需要 join 的表，剩下的参数是指明 join 列的约束。单个查询中可以 join 多个表：

~~~
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
~~~

<<<<<<< HEAD
#### Left Join 语句
=======
    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();
>>>>>>> laravel/5.1

如果你想执行的是 "left join" 而不是 "inner join"，则使用 `leftJoin` 方法。`leftJoin` 方法跟 `join` 方法使用的方式是一样的：

~~~
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
~~~

<<<<<<< HEAD
#### 高级的Join 语句
=======
    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();
>>>>>>> laravel/5.1

你也可以指定更多高级的 join 子句。马上开始，把一个 `闭包` 当做第二个参数 传到 `join` 方法中。该 `闭包` 接受用于指定约束 `join` 子句的 `JoinClause` 对象:

~~~
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
~~~

<<<<<<< HEAD
如果你喜欢在 joins 上使用 "where" 类型的子句，你可以在 joins 使用 `where` 和 `orWhere`方法。这些方法匹配与该列想对应的值：
=======
    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
            })
            ->get();
>>>>>>> laravel/5.1

~~~
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
~~~

<<<<<<< HEAD
=======
    DB::table('users')
            ->join('contacts', function ($join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();
>>>>>>> laravel/5.1

<a name="unions"></a>
## Unions

查询构造器提供一个快速的方法，把两个查询 "联合" 在一起。例如，你可以创建一个初始查询，并使用 `union` 方法联合第二个查询：

<<<<<<< HEAD
~~~
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
~~~
=======
    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();
>>>>>>> laravel/5.1

`unionAll` 方法也可以使用，和 `union`方法使用方式是一样的。

<a name="where-clauses"></a>
## Where 子句

#### 简单 Where 子句

在查询中加入 `where` 子句，在查询构造器上使用 `where` 方法，最简单的方法是调用 `where` 查询的三个参数。第一个参数是列名，第二个参数是操作符，可以是任何数据库支持的操作符。第三个参数是该列所对应的评估值。

这是一个验证 "votes" 列等于100的查询例如。

<<<<<<< HEAD
~~~
$users = DB::table('users')->where('votes', '=', 100)->get();
~~~
=======
    $users = DB::table('users')->where('votes', '=', 100)->get();
>>>>>>> laravel/5.1

为了方便起见，如果你想要验证一列等于跟定的值，你可以直接把该值作为第二个参数传递给 `where` 方法：

<<<<<<< HEAD
~~~
$users = DB::table('users')->where('votes', 100)->get();
~~~
=======
    $users = DB::table('users')->where('votes', 100)->get();
>>>>>>> laravel/5.1

当然，你可以在 `where` 子句中使用各种各样的操作符：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();
~~~
=======
    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();
>>>>>>> laravel/5.1

#### Or 子句
where 约束可以链接在一起使用，也可以添加 `or` 子句。`orWhere` 方法接受和 `where` 方法相同的参数：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
~~~
=======
You may chain where constraints together, as well as add `or` clauses to the query. The `orWhere` method accepts the same arguments as the `where` method:

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();
>>>>>>> laravel/5.1

#### 附加的 Where 子句

**whereBetween**

`whereBetween` 方法验证在两个值之间的一个列的值：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();
~~~
=======
    $users = DB::table('users')
                        ->whereBetween('votes', [1, 100])->get();
>>>>>>> laravel/5.1

**whereNotBetween**

`whereNotBetween` 方法验证在两个值之间以外的一个列的值：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
~~~
=======
    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();
>>>>>>> laravel/5.1

**whereIn / whereNotIn**

`whereIn` 方法验证一个给定列的值包含在给定的数组中：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
~~~
=======
    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();
>>>>>>> laravel/5.1

`whereIn` 方法验证一个给定列的值不包含在给定的数组中：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();
~~~
=======
    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();
>>>>>>> laravel/5.1

**whereNull / whereNotNull**

`whereNull` 方法验证给定列的值是 `NULL`:

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();
~~~
=======
    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();
>>>>>>> laravel/5.1

`whereNotNull` 方法验证给定列的值 **不是** `NULL`:

<<<<<<< HEAD
~~~
$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
~~~
=======
    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();
>>>>>>> laravel/5.1

<a name="advanced-where-clauses"></a>
## 高级的 Where 子句

#### 参数分组

有时候可能需要创建比如 "where exists" 或者嵌入参数分组等高级的 where 子句。Laravel 查询构造器能够很好地处理它们。马上开始，让我们看看在圆括号内的一个分组约束的例子：

~~~
DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function ($query) {
                $query->where('votes', '>', 100)
                      ->where('title', '<>', 'Admin');
            })
            ->get();
~~~

<<<<<<< HEAD
正如你所看到的,在 `orWhere` 里面传递 `闭包`，以命令该查询构造器开始一个约束分组。`闭包` 接收一个查询构造器实例，该实例通常用来设置被包含在括号内的分组约束。上面的例子将产生下列的 SQL：
=======
    DB::table('users')
                ->where('name', '=', 'John')
                ->orWhere(function ($query) {
                    $query->where('votes', '>', 100)
                          ->where('title', '<>', 'Admin');
                })
                ->get();
>>>>>>> laravel/5.1

~~~
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
~~~

<<<<<<< HEAD
#### Exists 子句
=======
    select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
>>>>>>> laravel/5.1

`whereExists` 方法允许编写 `where exist` SQL 子句。`whereExists` 方法接收一个 `闭包` 参数，该参数接收一个可以放置在 "exists" 子句内的自定义查询的查询构造器实例：

~~~
DB::table('users')
            ->whereExists(function ($query) {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
~~~

<<<<<<< HEAD
上面的查询语句将产生下列的 SQL：
=======
    DB::table('users')
                ->whereExists(function ($query) {
                    $query->select(DB::raw(1))
                          ->from('orders')
                          ->whereRaw('orders.user_id = users.id');
                })
                ->get();
>>>>>>> laravel/5.1

~~~
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
~~~

<<<<<<< HEAD
=======
    select * from users
    where exists (
        select 1 from orders where orders.user_id = users.id
    )
>>>>>>> laravel/5.1

<a name="ordering-grouping-limit-and-offset"></a>
## Ordering, Grouping, Limit, & Offset

#### orderBy

`orderBy` 方法允许你通过给定的列来排序所查询到的记录。`orderBy` 方法的第一个参数是你想要排序的列，第二个参数是控制排序的趋势，且只能是`asc` 或者 `desc` 中的其中一个：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
~~~
=======
    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();
>>>>>>> laravel/5.1

#### groupBy / having / havingRaw

`groupBy` 和 `having` 方法通常用于查询结果分组。`having` 方法的使用方式类似于 `where` 方法：

<<<<<<< HEAD
~~~
$users = DB::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
~~~
=======
    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();
>>>>>>> laravel/5.1

`havingRaw` 方法通常用于设置和 `having` 子句值一样的原生字符串。例如，查找最大售价超过$2,500的所有departments：

<<<<<<< HEAD
~~~
$users = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > 2500')
                ->get();
~~~
=======
    $users = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > 2500')
                    ->get();
>>>>>>> laravel/5.1

#### skip / take

使用 `skip` and `take` 方法，限制查询返回的记录数量，或者在查询(`OFFSET`)中跳过指定数量的记录：

~~~
$users = DB::table('users')->skip(10)->take(5)->get();
~~~

<<<<<<< HEAD
=======
    $users = DB::table('users')->skip(10)->take(5)->get();
>>>>>>> laravel/5.1

<a name="inserts"></a>
## Inserts

查询构造器也提供插入记录到数据库的 `insert` 方法。`insert`方法接收一个列名和值的数组。

<<<<<<< HEAD
~~~
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
~~~
=======
    DB::table('users')->insert(
        ['email' => 'john@example.com', 'votes' => 0]
    );
>>>>>>> laravel/5.1

你甚至可以调用一个 `insert` 并传递多维数组，实现插入多条记录。每个数组代表要插入到表中的一行：

<<<<<<< HEAD
~~~
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
~~~
=======
    DB::table('users')->insert([
        ['email' => 'taylor@example.com', 'votes' => 0],
        ['email' => 'dayle@example.com', 'votes' => 0]
    ]);
>>>>>>> laravel/5.1

#### Auto-Incrementing IDs

如果表存在自增 id，使用 `insertGetId` 方法插入一条记录，并取回该记录的ID：

<<<<<<< HEAD
~~~
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
~~~
=======
    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );
>>>>>>> laravel/5.1

>**Note:**当使用 PostgreSQL数据库时，insertGetId 方法期望自增的列名为 `id`，如果你想要取回不同系列的 ID，你可以把系列名当做第二个参数传递给 `insertGetId` 方法。

<a name="updates"></a>
## Updates

当然，除了条件记录到数据中以外，查询构造器也能使用 `update` 方法更新已经存在的记录。`update` 方法和`insert` 方法一样，接收包含要更新的列值对数组。 你可以使用 `where` 子句约束 `update` 查询：

<<<<<<< HEAD
~~~
DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);
~~~
=======
    DB::table('users')
                ->where('id', 1)
                ->update(['votes' => 1]);
>>>>>>> laravel/5.1

#### Increment / Decrement

查询构造器提供了指定列的值增量或减量方法。这是一个简单的捷径，比起手动编写 `update` 语句，提供了富有表现力和简洁的接口。

这两个方法最少要接收一个参数：要更改的列。第二个参数传递控制该列增量/减量的任意数字。

<<<<<<< HEAD
~~~
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
~~~

你也可以在操作期间指定更新额外列：
=======
Both of these methods accept at least one argument: the column to modify. A second argument may optionally be passed to control the amount by which the column should be incremented / decremented.

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);
>>>>>>> laravel/5.1

~~~
DB::table('users')->increment('votes', 1, ['name' => 'John']);
~~~

<<<<<<< HEAD
=======
    DB::table('users')->increment('votes', 1, ['name' => 'John']);
>>>>>>> laravel/5.1

<a name="deletes"></a>
## Deletes

当然，查询构造器也可以通过 `delete` 方法删除表记录：

<<<<<<< HEAD
~~~
DB::table('users')->delete();
~~~
=======
    DB::table('users')->delete();
>>>>>>> laravel/5.1

你可以在调用 `delete` 方法之前，增加 `where` 子句约束 `delete` 语句：

<<<<<<< HEAD
~~~
DB::table('users')->where('votes', '<', 100)->delete();
~~~
=======
    DB::table('users')->where('votes', '<', 100)->delete();
>>>>>>> laravel/5.1

如果你想要清空整张表，删除所有行并重置自增 ID 为 0，你可以使用 `truncate` 方法：

~~~
DB::table('users')->truncate();
~~~

<<<<<<< HEAD
=======
    DB::table('users')->truncate();
>>>>>>> laravel/5.1

<a name="pessimistic-locking"></a>
## Pessimistic Locking

<<<<<<< HEAD
查询构造器包含了少数 `select` 语句使用的 "悲观锁" 的函数。要运行一个 "共享锁" 语句，你可能要在查询上使用 `sharedLock` 方法。共享锁阻止查询修改了但还没事务提交的行：
=======
The query builder also includes a few functions to help you do "pessimistic locking" on your `select` statements. To run the statement with a "shared lock", you may use the `sharedLock` method on a query. A shared lock prevents the selected rows from being modified until your transaction commits:

    DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
>>>>>>> laravel/5.1

~~~
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
~~~
你可以使用 `lockForUpdate` 方法，"for update" 锁可以防止行被修改或者被其他的共享锁查询：

<<<<<<< HEAD
~~~
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
~~~
=======
    DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
>>>>>>> laravel/5.1
