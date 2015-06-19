# Database: Query Builder

- [Introduction](#introduction)
- [Retrieving Results](#retrieving-results)
	- [Aggregates](#aggregates)
- [Selects](#selects)
- [Joins](#joins)
- [Unions](#unions)
- [Where Clauses](#where-clauses)
	- [Advanced Where Clauses](#advanced-where-clauses)
- [Ordering, Grouping, Limit, & Offset](#ordering-grouping-limit-and-offset)
- [Inserts](#inserts)
- [Updates](#updates)
- [Deletes](#deletes)
- [Pessimistic Locking](#pessimistic-locking)

<a name="introduction"></a>
## 简介

数据库查询构造器为创建和执行数据库查询提供了方便的、流畅的接口。该接口能够在应用程序中执行大多数的数据库操作，并在所有受支持的数据库系统上工作。
The database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application, and works on all supported database systems.

> **注意:** laravel 查询构造器使用 PDO 参数绑定,防止应用程序受不利的 SQL 注入攻击。不需要清除字符串传递绑定。
> **Note:** The Laravel query builder uses PDO parameter binding to protect your application against SQL injection attacks. There is no need to clean strings being passed as bindings.



## 检索结果

#### 检索一张表的所有记录

流畅的查询，在 `DB` facade上使用 `table` 方法。`table` 方法将返回一个流畅的查询构造器实例给指定的表，
可以在查询上链接更多的约束，并最终获取到结果。在这个例子中，只是`获取`一张表的所有记录：
To begin a fluent query, use the `table` method on the `DB` facade. The `table` method returns a fluent query builder instance for the given table, allowing you to chain more constraints onto the query and then finally get the results. In this example, let's just `get` all records from a table:

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

如同[原生查询](http://laravel.com/docs/5.1/database),`get` 方法返回一个结果集数组，每一个结果集是  PHP `StdClass` 对象的一个实例。你可以像访问对象属性那样访问列来存取列的值：
 the `get` method returns an `array` of results where each result is an instance of the PHP `StdClass` object. You may access each column's value by accessing the column as a property of the object:

~~~
foreach ($users as $user) {
    echo $user->name;
}
~~~

#### 检索一张表的一行/列

如果你仅仅需要检索数据库中的一行记录，可以使用 `first` 方法。`first` 方法返回单一的 `StdClass` 对象：
If you just need to retrieve a single row from the database table, you may use the `first` method. This method will return a single `StdClass` object:

~~~
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
~~~

如果你甚至不需要一整行记录，可以使用 `value` 方法从记录中取出一个单一的值。该方法直接返回这个列的值：
If you don't even need an entire row, you may extract a single value from a record using the `value`method. This method will return the value of the column directly:

~~~
$email = DB::table('users')->where('name', 'John')->value('email');
~~~

#### 分割表结果

如果你需要处理成千上万的数据库记录，可以考虑使用 `chunk` 方法。`chunk` 方法每次只取回记录中的一小 "块"，并把每一块提供 `闭包` 中处理。该方法非常有益于编写处理成千上万的记录[Artisan 命令](http://laravel.com/docs/5.1/artisan)。例如，每次处理整张 `users` 表中每块为一百条的记录：
If you need to work with thousands of database records, consider using the `chunk` method. This method retrieves a small "chunk" of the results at a time, and feeds each chunk into a `Closure` for processing. This method is very useful for writing [Artisan commands](http://laravel.com/docs/5.1/artisan) that process thousands of records. For example, let's work with the entire `users` table in chunks of 100 records at a time:

~~~
DB::table('users')->chunk(100, function($users) {
    foreach ($users as $user) {
        //
    }
});
~~~

你可以通过 `闭包` 返回 `false` 来停止处理下一个块：
You may stop further chunks from being processed by returning `false` from the `Closure`:

~~~
DB::table('users')->chunk(100, function($users) {
    // Process the records...

    return false;
});
~~~

#### 检索一串列的值

如果你喜欢取得包含各列值得数组，你可以使用 `lists` 方法。在这个例子中，我们将取得 role 的 titles 数组：
If you would like to retrieve an array containing the values of a single column, you may use the`lists` method. In this example, we'll retrieve an array of role titles:

~~~
$titles = DB::table('roles')->lists('title');

foreach ($titles as $title) {
    echo $title;
}
~~~

你可以在返回的数组中指定一个自定义的键列：
You may also specify a custom key column for the returned array:

~~~
$roles = DB::table('roles')->lists('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
~~~



### 聚合

查询构造器也提供了各种各样的聚合方法，诸如 `count`, `max`, `min`, `avg`, 和 `sum`。你可以调用任何一个方法在构造查询之后：
The query builder also provides a variety of aggregate methods, such as `count`, `max`, `min`, `avg`, and `sum`. You may call any of these methods after constructing your query:

~~~
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
~~~


Of course, you may combine these methods with other clauses to build your query:

~~~
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
~~~



## Selects

#### 检索一个查询子句

当然，你并不是每次都要查询数据库中所有的列。使用`select` 方法，查询指明要的 `select` 子句：
Of course, you may not always want to select all columns from a database table. Using the `select`method, you can specify a custom `select` clause for the query:

~~~
$users = DB::table('users')->select('name', 'email as user_email')->get();
~~~

`distinct` 方法强行返回 distinct 记录：
The `distinct` method allows you to force the query to return distinct results:

~~~
$users = DB::table('users')->distinct()->get();
~~~

如果已存在查询构造器实例，并想要在当前的 select 子句加入一列，可以使用 `addSelect` 方法，：
If you already have a query builder instance and you wish to add a column to its existing select clause, you may use the `addSelect` method:

~~~
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
~~~

#### 原生表达式

有时候我们需要在查询中使用原生表达式。这些表达式以字符串的形式注入到查询中，所有注意不能创建任何的 SQL注入点！使用 `DB::raw` 方法，创建原生表达式：
Sometimes you may need to use a raw expression in a query. These expressions will be injected into the query as strings, so be careful not to create any SQL injection points! To create a raw expression, you may use the `DB::raw` method:

~~~
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
~~~



## Joins

#### Inner Join 语句

查询构造器也可以用来写 join 语句。执行基本的 SQL "inner join"，需要在查询构造器实例上使用 `join` 方法。`join` 方法的第一个参数传递你需要 join 的表，剩下的参数是指明 join 列的约束。单个查询中可以 join 多个表：
The query builder may also be used to write join statements. To perform a basic SQL "inner join", you may use the `join` method on a query builder instance. The first argument passed to the `join`method is the name of the table you need to join to, while the remaining arguments specify the column constraints for the join. Of course, as you can see, you can join to multiple tables in a single query:

~~~
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
~~~

#### Left Join 语句

如果你想执行的是 "left join" 而不是 "inner join"，则使用 `leftJoin` 方法。`leftJoin` 方法跟 `join` 方法使用的方式是一样的：
If you would like to perform a "left join" instead of an "inner join", use the `leftJoin` method. The`leftJoin` method has the same signature as the `join` method:

~~~
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
~~~

#### 高级的Join 语句

你也可以指定更多高级的 join 子句。马上开始，把一个 `闭包` 当做第二个参数 传到 `join` 方法中。该 `闭包` 接受用于指定约束 `join` 子句的 `JoinClause` 对象:
You may also specify more advanced join clauses. To get started, pass a `Closure` as the second argument into the `join` method. The `Closure` will receive a `JoinClause` object which allows you to specify constraints on the `join` clause:

~~~
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
~~~

如果你喜欢在 joins 上使用 "where" 类型的子句，你可以在 joins 使用 `where` 和 `orWhere`方法。这些方法匹配与该列想对应的值：
If you would like to use a "where" style clause on your joins, you may use the `where` and `orWhere`methods on a join. Instead of comparing two columns, these methods will compare the column against a value:

~~~
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
~~~



## Unions

查询构造器提供一个快速的方法，把两个查询 "联合" 在一起。例如，你可以创建一个初始查询，并使用 `union` 方法联合第二个查询：
The query builder also provides a quick way to "union" two queries together. For example, you may create an initial query, and then use the `union` method to union it with a second query:

~~~
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
~~~

`unionAll` 方法也可以使用，和 `union`方法使用方式是一样的。
 `unionAll` method is also available and has the same method signature as `union`.



## Where 子句

#### 简单 Where 子句

在查询中加入 `where` 子句，在查询构造器上使用 `where` 方法，最简单的方法是调用 `where` 查询的三个参数。第一个参数是列名，第二个参数是操作符，可以是任何数据库支持的操作符。第三个参数是该列所对应的评估值。
To add `where` clauses to the query, use the `where` method on a query builder instance. The most basic call to `where` requires three arguments. The first argument is the name of the column. The second argument is an operator, which can be any of the database's supported operators. The third argument is the value to evaluate against the column.

这是一个验证 "votes" 列等于100的查询例如。
For example, here is a query that verifies the value of the "votes" column is equal to 100:

~~~
$users = DB::table('users')->where('votes', '=', 100)->get();
~~~

为了方便起见，如果你想要验证一列等于跟定的值，你可以直接把该值作为第二个参数传递给 `where` 方法：
For convenience, if you simply want to verify that a column is equal to a given value, you may pass the value directly as the second argument to the `where` method:

~~~
$users = DB::table('users')->where('votes', 100)->get();
~~~

当然，你可以在 `where` 子句中使用各种各样的操作符：
Of course, you may use a variety of other operators when writing a `where` clause:

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

#### Or 子句
where 约束可以链接在一起使用，也可以添加 `or` 子句。`orWhere` 方法接受和 `where` 方法相同的参数：
You may chain where constraints together, as well as add `or` clauses to the query. The `orWhere`method accepts the same arguments as the `where` method:

~~~
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
~~~

#### 附加的 Where 子句

**whereBetween**

`whereBetween` 方法验证在两个值之间的一个列的值：
The `whereBetween` method verifies that a column's value is between two values:

~~~
$users = DB::table('users')
                    ->whereBetween('votes', [1, 100])->get();
~~~

**whereNotBetween**

`whereNotBetween` 方法验证在两个值之间以外的一个列的值：
The `whereNotBetween` method verifies that a column's value lies outside of two values:

~~~
$users = DB::table('users')
                    ->whereNotBetween('votes', [1, 100])
                    ->get();
~~~

**whereIn / whereNotIn**

`whereIn` 方法验证一个给定列的值包含在给定的数组中：
The `whereIn` method verifies that a given column's value is contained within the given array:

~~~
$users = DB::table('users')
                    ->whereIn('id', [1, 2, 3])
                    ->get();
~~~

`whereIn` 方法验证一个给定列的值不包含在给定的数组中：
The `whereNotIn` method verifies that the given column's value is **not** contained in the given array:

~~~
$users = DB::table('users')
                    ->whereNotIn('id', [1, 2, 3])
                    ->get();
~~~

**whereNull / whereNotNull**

`whereNull` 方法验证给定列的值是 `NULL`:
The `whereNull` method verifies that the value of the given column is `NULL`:

~~~
$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();
~~~

`whereNotNull` 方法验证给定列的值 **不是** `NULL`:
The `whereNotNull` method verifies that the column's value is **not** `NULL`:

~~~
$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
~~~



## 高级的 Where 子句

#### 参数分组

有时候可能需要创建比如 "where exists" 或者嵌入参数分组等高级的 where 子句。Laravel 查询构造器能够很好地处理它们。马上开始，让我们看看在圆括号内的一个分组约束的例子：
Sometimes you may need to create more advanced where clauses such as "where exists" or nested parameter groupings. The Laravel query builder can handle these as well. To get started, let's look at an example of grouping constraints within parenthesis:

~~~
DB::table('users')
            ->where('name', '=', 'John')
            ->orWhere(function ($query) {
                $query->where('votes', '>', 100)
                      ->where('title', '<>', 'Admin');
            })
            ->get();
~~~

正如你所看到的,在 `orWhere` 里面传递 `闭包`，以命令该查询构造器开始一个约束分组。`闭包` 接收一个查询构造器实例，该实例通常用来设置被包含在括号内的分组约束。上面的例子将产生下列的 SQL：
As you can see, passing `Closure` into the `orWhere` method instructs the query builder to begin a constraint group. The `Closure` will receive a query builder instance which you can use to set the constraints that should be contained within the parenthesis group. The example above will produce the following SQL:

~~~
select * from users where name = 'John' or (votes > 100 and title <> 'Admin')
~~~

#### Exists 子句

`whereExists` 方法允许编写 `where exist` SQL 子句。`whereExists` 方法接收一个 `闭包` 参数，该参数接收一个可以放置在 "exists" 子句内的自定义查询的查询构造器实例：
The `whereExists` method allows you to write `where exist` SQL clauses.
The `whereExists`method accepts a `Closure` argument, which will receive a query builder instance allowing you to define the query that should be placed inside of the "exists" clause:

~~~
DB::table('users')
            ->whereExists(function ($query) {
                $query->select(DB::raw(1))
                      ->from('orders')
                      ->whereRaw('orders.user_id = users.id');
            })
            ->get();
~~~

上面的查询语句将产生下列的 SQL：
The query above will produce the following SQL:

~~~
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
~~~



## Ordering, Grouping, Limit, & Offset

#### orderBy

`orderBy` 方法允许你通过给定的列来排序所查询到的记录。`orderBy` 方法的第一个参数是你想要排序的列，第二个参数是控制排序的趋势，且只能是`asc` 或者 `desc` 中的其中一个：
The `orderBy` method allows you to sort the result of the query by a given column. The first argument to the `orderBy` method should be the column you wish to sort by, while the second argument controls the direction of the sort and may be either `asc` or `desc`:

~~~
$users = DB::table('users')
                ->orderBy('name', 'desc')
                ->get();
~~~

#### groupBy / having / havingRaw

`groupBy` 和 `having` 方法通常用于查询结果分组。`having` 方法的使用方式类似于 `where` 方法：
The `groupBy` and `having` methods may be used to group the query results. The `having`method's signature is similar to that of the `where` method:

~~~
$users = DB::table('users')
                ->groupBy('account_id')
                ->having('account_id', '>', 100)
                ->get();
~~~

`havingRaw` 方法通常用于设置和 `having` 子句值一样的原生字符串。例如，查找最大售价超过$2,500的所有departments：
The `havingRaw` method may be used to set a raw string as the value of the `having` clause. For example, we can find all of the departments with sales greater than $2,500:

~~~
$users = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > 2500')
                ->get();
~~~

#### skip / take

使用 `skip` and `take` 方法，限制查询返回的记录数量，或者在查询(`OFFSET`)中跳过指定数量的记录：
To limit the number of results returned from the query, or to skip a given number of results in the query (`OFFSET`), you may use the `skip` and `take` methods:

~~~
$users = DB::table('users')->skip(10)->take(5)->get();
~~~



## Inserts

查询构造器也提供插入记录到数据库的 `insert` 方法。`insert`方法接收一个列名和值的数组。
The query builder also provides an `insert` method for inserting records into the database table. The `insert` method accepts an array of column names and values to insert:

~~~
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
~~~

你甚至可以调用一个 `insert` 并传递多维数组，实现插入多条记录。每个数组代表要插入到表中的一行：
You may even insert several records into the table with a single call to `insert` by passing an array of arrays. Each array represents a row to be inserted into the table:

~~~
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
~~~

#### Auto-Incrementing IDs

如果表存在自增 id，使用 `insertGetId` 方法插入一条记录，并取回该记录的ID：
If the table has an auto-incrementing id, use the `insertGetId` method to insert a record and then retrieve the ID:

~~~
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
~~~

>**Note:**当使用 PostgreSQL数据库时，insertGetId 方法期望自增的列名为 `id`，如果你想要取回不同系列的 ID，你可以把系列名当做第二个参数传递给 `insertGetId` 方法。

> **Note:** When using PostgreSQL the insertGetId method expects the auto-incrementing column to be named `id`. If you would like to retrieve the ID from a different "sequence", you may pass the sequence name as the second parameter to the `insertGetId` method.



## Updates

当然，除了条件记录到数据中以外，查询构造器也能使用 `update` 方法更新已经存在的记录。`update` 方法和`insert` 方法一样，接收包含要更新的列值对数组。 你可以使用 `where` 子句约束 `update` 查询：
Of course, in addition to inserting records into the database, the query builder can also update existing records using the `update` method. The `update` method, like the `insert` method, accepts an array of column and value pairs containing the columns to be updated. You may constrain the`update` query using `where` clauses:

~~~
DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);
~~~

#### Increment / Decrement

查询构造器提供了指定列的值增量或减量方法。这是一个简单的捷径，比起手动编写 `update` 语句，提供了富有表现力和简洁的接口。
The query builder also provides convenient methods for incrementing or decrementing the value of a given column. This is simply a short-cut, providing a more expressive and terse interface compared to manually writing the `update` statement.

这两个方法最少要接收一个参数：要更改的列。第二个参数传递控制该列增量/减量的任意数字。
Both of these methods accept at least one argument: the column to modify. An second argument may optionally be passed to control the amount by which the column should be incremented / decremented.

~~~
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
~~~

你也可以在操作期间指定更新额外列：
You may also specify additional columns to update during the operation:

~~~
DB::table('users')->increment('votes', 1, ['name' => 'John']);
~~~



## Deletes

当然，查询构造器也可以通过 `delete` 方法删除表记录：
Of course, the query builder may also be used to delete records from the table via the `delete`method:

~~~
DB::table('users')->delete();
~~~

你可以在调用 `delete` 方法之前，增加 `where` 子句约束 `delete` 语句：
You may constrain `delete` statements by adding `where` clauses before calling the `delete`method:

~~~
DB::table('users')->where('votes', '<', 100)->delete();
~~~

如果你想要清空整张表，删除所有行并重置自增 ID 为 0，你可以使用 `truncate` 方法：
If you wish to truncate the entire table, which will remove all rows and reset the auto-incrementing ID to zero, you may use the `truncate` method:

~~~
DB::table('users')->truncate();
~~~



## Pessimistic Locking

查询构造器包含了少数 `select` 语句使用的 "悲观锁" 的函数。要运行一个 "共享锁" 语句，你可能要在查询上使用 `sharedLock` 方法。共享锁阻止查询修改了但还没事务提交的行：
The query builder also includes a few functions to help you do "pessimistic locking" on your `select`statements. To run the statement with a "shared lock", you may use the `sharedLock` method on a query. A shared lock prevents the selected rows from being modified until your transaction commits:

~~~
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
~~~
你可以使用 `lockForUpdate` 方法，"for update" 锁可以防止行被修改或者被其他的共享锁查询：
Alternatively, you may use the `lockForUpdate` method. A "for update" lock prevents the rows from being modified or from being selected with another shared lock:

~~~
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
~~~
