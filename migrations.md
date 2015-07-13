# Database: Migrations

- [Introduction](#introduction)
- [Generating Migrations](#generating-migrations)
- [Migration Structure](#migration-structure)
<<<<<<< HEAD
- [执行迁移](#running-migrations)
	- [回滚迁移](#rolling-back-migrations)
- [编写迁移](#writing-migrations)
	- [创建表](#creating-tables)
	- [重命名/删除表](#renaming-and-dropping-tables)
	- [创建列](#creating-columns)
	- [修改列](#modifying-columns)
	- [删除列](#dropping-columns)
	- [创建索引](#creating-indexes)
	- [删除索引](#dropping-indexes)
	- [外键约束](#foreign-key-constraints)
=======
- [Running Migrations](#running-migrations)
    - [Rolling Back Migrations](#rolling-back-migrations)
- [Writing Migrations](#writing-migrations)
    - [Creating Tables](#creating-tables)
    - [Renaming / Dropping Tables](#renaming-and-dropping-tables)
    - [Creating Columns](#creating-columns)
    - [Modifying Columns](#modifying-columns)
    - [Dropping Columns](#dropping-columns)
    - [Creating Indexes](#creating-indexes)
    - [Dropping Indexes](#dropping-indexes)
    - [Foreign Key Constraints](#foreign-key-constraints)
>>>>>>> laravel/5.1

<a name="introduction"></a>
## 简介

迁移如同数据库版本控制，允许一个团队更容易地修改和共享应用程序数据库模式。迁移一般与 Laravel 模式构造器搭配，更容易地构造你的应用程序数据库模式。流畅的 API 贯穿整个  Laravel 支持的数据库系统。

Laravel `Schema` [facade](/docs/{{version}}/facades) 提供了与数据库无关的创建和操纵表。它共享相同的表达式，

<a name="generating-migrations"></a>
## Generating Migrations

创建一个迁移，使用`make:migration` [Artisan command](/docs/{{version}}/artisan):

<<<<<<< HEAD
~~~
php artisan make:migration create_users_table
~~~
=======
    php artisan make:migration create_users_table
>>>>>>> laravel/5.1

新创建的迁移被放置在 `database/migrations` 目录中，每个迁移文件包含了 Laravel 决定迁移顺序的时间戳。

`--table` 和 `--create` 选项分别用于指出表名和是否迁移生成一张新表。这些选项简单地预填充指明表产生迁移的 stub 文件：

<<<<<<< HEAD
~~~
php artisan make:migration add_votes_to_users_table --table=users

php artisan make:migration create_users_table --create=users
~~~
=======
    php artisan make:migration add_votes_to_users_table --table=users

    php artisan make:migration create_users_table --create=users

If you would like to specify a custom output path for the generated migration, you may use the `--path` option when executing the `make:migration` command. The provided path should be relative to your application's base path.
>>>>>>> laravel/5.1

<a name="Migration-Structure"></a>
## Migration Structure

<<<<<<< HEAD
一个迁移类包含 `up` 和 `down` 两个方法。`up` 方法用于添加新的表、列、或者数据库索引，`down` 方法简单地逆转 `up` 方法执行的操作。

在这两个方法的内部，使用Laravel 模式构造器来表示创建和修改的表。[查看这份文档](/docs/{{version}}/migrations#creating-tables)，学习 `模式` 构造器上所有有用的方法。例如，让我们看创建一张 `flights` 表简单迁移：

~~~
<?php

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
~~~
=======
A migration class contains two methods: `up` and `down`. The `up` method is used to add new tables, columns, or indexes to your database, while the `down` method should simply reverse the operations performed by the `up` method.

Within both of these methods you may use the Laravel schema builder to expressively create and modify tables. To learn about all of the methods available on the `Schema` builder, [check out its documentation](#creating-tables). For example, let's look at a sample migration that creates a `flights` table:

    <?php

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Migrations\Migration;

    class CreateFlightsTable extends Migration
    {
        /**
         * Run the migrations.
         *
         * @return void
         */
        public function up()
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->increments('id');
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * Reverse the migrations.
         *
         * @return void
         */
        public function down()
        {
            Schema::drop('flights');
        }
    }
>>>>>>> laravel/5.1


<a name="running-migrations"></a>
## 执行迁移

执行应用程序中所有杰出的迁移，使用 `migrate` Artisan 命令。如果你使用的是 [Homestead 虚拟机]，你应该(/docs/{{version}}/homestead),你就可以在虚拟机内执行执行这个命令:

<<<<<<< HEAD
=======
    php artisan migrate
>>>>>>> laravel/5.1

~~~
php artisan migrate
~~~

如果当你执行迁移时接收到一个 "class not found" 错误，尝试执行 `composer dump-autoload` 命令，并重新使用该 migrate 命令.


#### Forcing Migrations To Run In Production

有些迁移具有毁灭性操作，这意外地它们可能会导致丢失数据。为了保护执行对生产数据库不利的命令，系统会提示你输入确认。使用 `--force` 标记强制执行没带提示的命令:

~~~
php artisan migrate --force
~~~

<<<<<<< HEAD
=======
    php artisan migrate --force
>>>>>>> laravel/5.1

<a name="rolling-back-migrations"></a>
### 回滚迁移

回滚最后的迁移"操作"，使用 `rollback` 命令，注意：回滚将执行迁移最后的 "一批"，其中可能包含多个迁移文件：

<<<<<<< HEAD
~~~
php artisan migrate:rollback
~~~
=======
    php artisan migrate:rollback
>>>>>>> laravel/5.1

`migrate:reset` 命令将回滚所有的应用程序迁移：

<<<<<<< HEAD
~~~
php artisan migrate:reset
~~~
=======
    php artisan migrate:reset
>>>>>>> laravel/5.1

#### 单一命令的回滚/迁移

`migrate:refresh` 命令将首先回滚所有的数据库迁移，然后执行 `migrate` 命令，这个命令有效地重建整个数据库：

<<<<<<< HEAD
~~~
php artisan migrate:refresh

php artisan migrate:refresh --seed
~~~
=======
    php artisan migrate:refresh

    php artisan migrate:refresh --seed
>>>>>>> laravel/5.1

<a name="writing-migrations"></a>
## 编写迁移

<a name="creating-tables"></a>
### 创建表

创建一张新的数据库表，在`Schema` facade 上使用 `create` 方法。`create` 方法 接收两个参数。第一个参数是表名，第二个参数是闭包，该闭包接收一个用于定义新表的 `Blueprint` 对象：

<<<<<<< HEAD
~~~
Schema::create('users', function ($table) {
    $table->increments('id');
});
~~~
=======
    Schema::create('users', function ($table) {
        $table->increments('id');
    });
>>>>>>> laravel/5.1

当然了，当创建表时，你可以使用任何的模式构造器[列方法]定义表的列。(/docs/{{version}}/migrations#creating-columns)

#### 检查表/列是否存在

你可以使用 `hasTable` 和 `hasColumn` 方法来检查表或列是否存在：

<<<<<<< HEAD
~~~
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
~~~
=======
    if (Schema::hasTable('users')) {
        //
    }

    if (Schema::hasColumn('users', 'email')) {
        //
    }
>>>>>>> laravel/5.1

#### Connection & Storage Engine

如果你想要在非默认的数据库连接上执行 schema 操作，使用 `connection` 方法：

<<<<<<< HEAD
~~~
Schema::connection('foo')->create('users', function ($table) {
    $table->increments('id');
});
~~~
=======
    Schema::connection('foo')->create('users', function ($table) {
        $table->increments('id');
    });
>>>>>>> laravel/5.1

为一张表设置存储引擎，需要在模式构造器上设置 `engine` 属性：

<<<<<<< HEAD
~~~
Schema::create('users', function ($table) {
    $table->engine = 'InnoDB';

    $table->increments('id');
});
~~~
=======
    Schema::create('users', function ($table) {
        $table->engine = 'InnoDB';

        $table->increments('id');
    });
>>>>>>> laravel/5.1

<a name="renaming-and-dropping-tables"></a>
### 重命名/删除表

使用 `rename` 方法重新命名一张已存在的数据库表：

<<<<<<< HEAD
~~~
Schema::rename($from, $to);
~~~
=======
    Schema::rename($from, $to);
>>>>>>> laravel/5.1

要删除一张存在的表，你可以使用 `drop` 或者 `dropIfExists` 方法：

<<<<<<< HEAD
~~~
Schema::drop('users');

Schema::dropIfExists('users');
~~~


<a name="creating-columns"></a>
### 创建列

要更新一张存在的表，我们将在  `Schema` facade 上使用 `table` 方法。和 `create` 方法一样，`table`方法接收两个参数：表名和一个闭包，该闭包接收一个 `Blueprint` 实例把列添加到表中：

~~~
Schema::table('users', function ($table) {
    $table->string('email');
});
~~~

#### 可用的列类型

当然，当你创建表时，模式构造器包含了多种多样的列类型可供使用：

| Command | Description |
| --- | --- |
| `$table->bigIncrements('id');` | Incrementing ID using a "big integer" equivalent. |
| `$table->bigInteger('votes');` | BIGINT equivalent for the database. |
| `$table->binary('data');` | BLOB equivalent for the database. |
| `$table->boolean('confirmed');` | BOOLEAN equivalent for the database. |
| `$table->char('name', 4);` | CHAR equivalent with a length. |
| `$table->date('created_at');` | DATE equivalent for the database. |
| `$table->dateTime('created_at');` | DATETIME equivalent for the database. |
| `$table->decimal('amount', 5, 2);` | DECIMAL equivalent with a precision and scale. |
| `$table->double('column', 15, 8);` | DOUBLE equivalent with precision, 15 digits in total and 8 after the decimal point. |
| `$table->enum('choices', ['foo', 'bar']);` | ENUM equivalent for the database. |
| `$table->float('amount');` | FLOAT equivalent for the database. |
| `$table->increments('id');` | Incrementing ID for the database (primary key). |
| `$table->integer('votes');` | INTEGER equivalent for the database. |
| `$table->json('options');` | JSON equivalent for the database. |
| `$table->jsonb('options');` | JSONB equivalent for the database. |
| `$table->longText('description');` | LONGTEXT equivalent for the database. |
| `$table->mediumInteger('numbers');` | MEDIUMINT equivalent for the database. |
| `$table->mediumText('description');` | MEDIUMTEXT equivalent for the database. |
| `$table->morphs('taggable');` | Adds INTEGER `taggable_id` and STRING `taggable_type`. |
| `$table->nullableTimestamps();` | Same as `timestamps()`, except allows NULLs. |
| `$table->rememberToken();` | Adds `remember_token` as VARCHAR(100) NULL. |
| `$table->smallInteger('votes');` | SMALLINT equivalent for the database. |
| `$table->softDeletes();` | Adds `deleted_at` column for soft deletes. |
| `$table->string('email');` | VARCHAR equivalent column. |
| `$table->string('name', 100);` | VARCHAR equivalent with a length. |
| `$table->text('description');` | TEXT equivalent for the database. |
| `$table->time('sunrise');` | TIME equivalent for the database. |
| `$table->tinyInteger('numbers');` | TINYINT equivalent for the database. |
| `$table->timestamp('added_on');` | TIMESTAMP equivalent for the database. |
| `$table->timestamps();` | Adds `created_at` and `updated_at` columns. |

#### 修改器

除了上面列举的列类型之外，这还有几个列 "修改器"可供添加使用。例如，使列 "nullable"，你可以使用 `nullable` 方法：

~~~
Schema::table('users', function ($table) {
    $table->string('email')->nullable();
});
~~~

下面是所有有用的列修改器列表。该列表不包含 [index modifiers](/docs/{{version}}/migrations#adding-indexes):

| Modifier | Description |
| --- | --- |
| `->first()` | Place the column "first" in the table (MySQL Only) |
| `->after('column')` | Place the column "after" another column (MySQL Only) |
| `->nullable()` | Allow NULL values to be inserted into the column |
| `->default($value)` | Specify a "default" value for the column |
| `->unsigned()` | Set `integer` columns to `UNSIGNED` |


=======
    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="creating-columns"></a>
### Creating Columns

To update an existing table, we will use the `table` method on the `Schema` facade. Like the `create` method, the `table` method accepts two arguments: the name of the table and a `Closure` that receives a `Blueprint` instance we can use to add columns to the table:

    Schema::table('users', function ($table) {
        $table->string('email');
    });

#### Available Column Types

Of course, the schema builder contains a variety of column types that you may use when building your tables:

Command  | Description
------------- | -------------
`$table->bigIncrements('id');`  |  Incrementing ID using a "big integer" equivalent.
`$table->bigInteger('votes');`  |  BIGINT equivalent for the database.
`$table->binary('data');`  |  BLOB equivalent for the database.
`$table->boolean('confirmed');`  |  BOOLEAN equivalent for the database.
`$table->char('name', 4);`  |  CHAR equivalent with a length.
`$table->date('created_at');`  |  DATE equivalent for the database.
`$table->dateTime('created_at');`  |  DATETIME equivalent for the database.
`$table->decimal('amount', 5, 2);`  |  DECIMAL equivalent with a precision and scale.
`$table->double('column', 15, 8);`  |  DOUBLE equivalent with precision, 15 digits in total and 8 after the decimal point.
`$table->enum('choices', ['foo', 'bar']);` | ENUM equivalent for the database.
`$table->float('amount');`  |  FLOAT equivalent for the database.
`$table->increments('id');`  |  Incrementing ID for the database (primary key).
`$table->integer('votes');`  |  INTEGER equivalent for the database.
`$table->json('options');`  |  JSON equivalent for the database.
`$table->jsonb('options');`  |  JSONB equivalent for the database.
`$table->longText('description');`  |  LONGTEXT equivalent for the database.
`$table->mediumInteger('numbers');`  |  MEDIUMINT equivalent for the database.
`$table->mediumText('description');`  |  MEDIUMTEXT equivalent for the database.
`$table->morphs('taggable');`  |  Adds INTEGER `taggable_id` and STRING `taggable_type`.
`$table->nullableTimestamps();`  |  Same as `timestamps()`, except allows NULLs.
`$table->rememberToken();`  |  Adds `remember_token` as VARCHAR(100) NULL.
`$table->smallInteger('votes');`  |  SMALLINT equivalent for the database.
`$table->softDeletes();`  |  Adds `deleted_at` column for soft deletes.
`$table->string('email');`  |  VARCHAR equivalent column.
`$table->string('name', 100);`  |  VARCHAR equivalent with a length.
`$table->text('description');`  |  TEXT equivalent for the database.
`$table->time('sunrise');`  |  TIME equivalent for the database.
`$table->tinyInteger('numbers');`  |  TINYINT equivalent for the database.
`$table->timestamp('added_on');`  |  TIMESTAMP equivalent for the database.
`$table->timestamps();`  |  Adds `created_at` and `updated_at` columns.

#### Column Modifiers

In addition to the column types listed above, there are several other column "modifiers" which you may use while adding the column. For example, to make the column "nullable", you may use the `nullable` method:

    Schema::table('users', function ($table) {
        $table->string('email')->nullable();
    });

Below is a list of all the available column modifiers. This list does not include the [index modifiers](#adding-indexes):

Modifier  | Description
------------- | -------------
`->first()`  |  Place the column "first" in the table (MySQL Only)
`->after('column')`  |  Place the column "after" another column (MySQL Only)
`->nullable()`  |  Allow NULL values to be inserted into the column
`->default($value)`  |  Specify a "default" value for the column
`->unsigned()`  |  Set `integer` columns to `UNSIGNED`

<a name="changing-columns"></a>
>>>>>>> laravel/5.1
<a name="modifying-columns"></a>
### 修改列

#### 预备知识

在修改列之前，一定要在 `composer.json` 文件中添加 `doctrine/dbal` 依赖。该 Doctrine DBAL 库通常确定列的当前状态和创建 SQL 查询所需要指明调整的列。

#### 更新列属性

`change` 方法允许修改已有列的类型或者列的属性，例如，你期望增加字符串列的尺寸。参考活动中的 `change` 方法，增长 `name` 列的尺寸从25 添加到 50：

<<<<<<< HEAD
~~~
Schema::table('users', function ($table) {
    $table->string('name', 50)->change();
});
~~~
=======
    Schema::table('users', function ($table) {
        $table->string('name', 50)->change();
    });
>>>>>>> laravel/5.1

我们也会修改一列为nullable：

<<<<<<< HEAD
~~~
Schema::table('users', function ($table) {
    $table->string('name', 50)->nullable()->change();
});
~~~
=======
    Schema::table('users', function ($table) {
        $table->string('name', 50)->nullable()->change();
    });
>>>>>>> laravel/5.1

<a name="renaming-and-dropping-tables"></a>
#### 重命名列

要重命名一列。你可能要在模式构造器上使用 `renameColumn` 方法。在修改列之前，一定要在` composer.json` 添加` doctrine/dbal` 库依赖：

<<<<<<< HEAD
~~~
Schema::table('users', function ($table) {
    $table->renameColumn('from', 'to');
});
~~~
=======
    Schema::table('users', function ($table) {
        $table->renameColumn('from', 'to');
    });
>>>>>>> laravel/5.1

> **注意：**当前不支持在一张表中重命名带 `enum` 的列。

<a name="renaming-and-dropping-tables"></a>
### 删除列

要删除一列，需要在模式构造器上使用 `dropColumn` 方法：

<<<<<<< HEAD
~~~
Schema::table('users', function ($table) {
    $table->dropColumn('votes');
});
~~~
=======
    Schema::table('users', function ($table) {
        $table->dropColumn('votes');
    });
>>>>>>> laravel/5.1

你可以通过传递列名数组给 `dropColumn` 方法删除一张表的多列：

<<<<<<< HEAD
~~~
Schema::table('users', function ($table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
~~~
=======
    Schema::table('users', function ($table) {
        $table->dropColumn(['votes', 'avatar', 'location']);
    });
>>>>>>> laravel/5.1

> **Note:** 在 SQLite 数据库中删除列，一定要在 `composer.json` 文件中添加 `doctrine/dbal` 依赖，并在安装该库的终端执行 `composer update` 命名。

<a name="creating-indexes"></a>
### 创建索引

模式构造器支持几个索引类型，让我们看看指明一列的值为唯一的例子。创建索引，我们只需要在列定义之上链接上`unique` 方法：

<<<<<<< HEAD
~~~
$table->string('email')->unique();
~~~
=======
    $table->string('email')->unique();
>>>>>>> laravel/5.1

非此即彼，你可以在定义列之后创建索引，例如：

<<<<<<< HEAD
~~~
$table->unique('email');
~~~
=======
    $table->unique('email');
>>>>>>> laravel/5.1

你甚至可以传递列数组到索引方法来创建组合索引：

<<<<<<< HEAD
~~~
$table->index(['account_id', 'created_at']);
~~~
=======
    $table->index(['account_id', 'created_at']);
>>>>>>> laravel/5.1

####有用的所有类型

| Command | Description |
| --- | --- |
| `$table->primary('id');` | Add a primary key. |
| `$table->primary(['first', 'last']);` | Add composite keys. |
| `$table->unique('email');` | Add a unique index. |
| `$table->index('state');` | Add a basic index. |


<a name="dropping-indexes"></a>
### 删除索引

删除索引，必须首先指明索引的名字，默认下，Laravel自动分配一个合理的名字给索引，简单连结的表名，索引上中的列名，和列的类型，这里有一些实例：

| Command | Description |
| --- | --- |
| `$table->dropPrimary('users_id_primary');` | Drop a primary key from the "users" table. |
| `$table->dropUnique('users_email_unique');` | Drop a unique index from the "users" table. |
| `$table->dropIndex('geo_state_index');` | Drop a basic index from the "geo" table. |


<a href="foreign-key-constraints"></a>
### 外键约束

Laravel也提供支持创建外键约束，这是用于促使数据库级别的引用完整性。例如，我们参照 `users` 表上的 `id` 列在 `posts` 上定义一个 `user_id` 方法：

<<<<<<< HEAD
~~~
Schema::table('posts', function ($table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
~~~
=======
    Schema::table('posts', function ($table) {
        $table->integer('user_id')->unsigned();

        $table->foreign('user_id')->references('id')->on('users');
    });
>>>>>>> laravel/5.1

你也可以指定期望的 "on delete" 和"on update" 属性约束动作：

<<<<<<< HEAD
~~~
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
~~~
=======
    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');
>>>>>>> laravel/5.1

要删除一个外键，你可以使用 `dropForeign` 方法，外键约束使用相同的命名规定作为索引，所以，我们将链接表名和列约束，然后后缀名为 "_foreign"：

<<<<<<< HEAD
~~~
$table->dropForeign('posts_user_id_foreign');
~~~
=======
    $table->dropForeign('posts_user_id_foreign');
>>>>>>> laravel/5.1
