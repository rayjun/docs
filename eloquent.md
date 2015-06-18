# Eloquent: Getting Started

- [Introduction](#introduction)
- [Defining Models](#defining-models)
	- [Eloquent Model Conventions](#eloquent-model-conventions)
- [Retrieving Multiple Models](#retrieving-multiple-models)
- [Retrieving Single Models / Aggregates](#retrieving-single-models)
	- [Retrieving Aggregates](#retrieving-aggregates)
- [Inserting & Updating Models](#inserting-and-updating-models)
	- [Basic Inserts](#basic-inserts)
	- [Basic Updates](#basic-updates)
	- [Mass Assignment](#mass-assignment)
- [Deleting Models](#deleting-models)
	- [Soft Deleting](#soft-deleting)
	- [Querying Soft Deleted Models](#querying-soft-deleted-models)
- [Query Scopes](#query-scopes)
- [Events](#events)

<a name="introduction"></a>
## Introduction

The Eloquent ORM included with Laravel provides a beautiful, simple ActiveRecord implementation for working with your database. Each database table has a corresponding "Model" which is used to interact with that table. Models allow you to query for data in your tables, as well as insert new records into the table.

Before getting started, be sure to configure a database connection in `config/database.php`. For more information on configuring your database, check out [the documentation](/docs/{{version}}/database#configuration).

<a name="defining-models"></a>
## Defining Models

To get started, let's create an Eloquent model. Models typically live in the `app` directory, but you are free to place them anywhere that can be auto-loaded according to your `composer.json` file. All Eloquent models extend `Illuminate\Database\Eloquent\Model` class.

The easiest way to create a model instance is using the `make:model` [Artisan command](/docs/{{version}}/artisan):

	php artisan make:model User

If you would like to generate a [database migration](/docs/{{version}}/schema#database-migrations) when you generate the model, you may use the `--migration` or `-m` option:

	php artisan make:model User --migration

	php artisan make:model User -m

<a name="eloquent-model-conventions"></a>
## [简介](http://laravel.com/docs/5.1/eloquent#introduction)

Eloquent ORM 包含了 laravel 数据库使用提供的一个完美的，简洁的 ActiveRecord 实现。每个数据表都有一个与它相对应、相互作用的"Model"。Models 允许在表中查询数据，及向表内插入新的记录. 

在开始之前，务必在 'config/database.php' 配置数据连接。需要更多数据库配置信息，查看[文档](http://laravel.com/docs/5.1/database#configuration)。

## [定义模型](http://laravel.com/docs/5.1/eloquent#defining-models)

马上开始,让我们创建一个model。 models通常放置在'app'目录下，你可以根据 `composer.json` 文件自由地放置在可以自动加载的任何地方。 并且所有的Eloquent models 都要继承 'Illuminate\Database\Eloquent\Model' 类。

创建一个model实例最简单地方法是使用 `make:model` [Artisan 命名](http://laravel.com/docs/5.1/artisan):

~~~
php artisan make:model User
~~~

如果你想在生成model时生成 [database migration](http://laravel.com/docs/5.1/schema#database-migrations)，使用 `--migration` or `-m` 选项:

~~~
php artisan make:model User --migration

php artisan make:model User -m
~~~



### Eloquent 模型规范

现在,让我们来看一个用于检索和存储 `flights` 数据库表信息的 `Flight` model 类的例子:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
~~~

#### 表名

注意，我们并没有告诉Eloquent那张表用于我们的 `Flight` model。"snake case"，类名的复数将用来当做表名，除非明确指定另一个名称。所以，在这种情况下，Eloquent 采用 `Flight` 模型存储记录在 `flights` 表内。你可以在model定义一个属性 `table` 指向自定义的表。

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The table associated with the model.
     *
     * @var string
     */
    protected $table = 'my_flights';
}
~~~

#### 主键

Eloquent 默认采取了每一个表中的 `id` 列做为该表的主键，你可以定义一个 `$primaryKey` 属性，不顾 Eloquent 主键规范。

#### 时间戳

默认情况下，Eloquent 期望 `created_at` 和 `updated_at` 列存在于表中，如果你不想通过 Eloquent 自动管理这两列。
在model内设置 `$timestamps` 属性值为 `false`:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * Indicates if the model should be timestamped.
     *
     * @var bool
     */
    public $timestamps = false;
}
~~~

如果你需要定制时间戳的格式，需要在model里面设置 `$dateFormat` 属性。 `$dateFormat` 属性决定了数据库中存储的时间属性，这些格式也会随着 model 被系列化时转化为 array 或者 JSON:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
~~~



## [检索多样模型](http://laravel.com/docs/5.1/eloquent#retrieving-multiple-models)

一旦你创建了 model 并[关联了数据库表](http://laravel.com/docs/5.1/schema)，准备开始在数据库中检索数据。认为每个 Eloquent 模型如同一个强大的[查询构造器](http://laravel.com/docs/5.1/queries)，该查询构造器允许流利地查询与 model 相关联的数据库表。例如:

~~~
<?php

namespace App\Http\Controllers;

use App\Flight;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * Show a list of all available flights.
     *
     * @return Response
     */
    public function index()
    {
        $flights = Flight::all();

        return view('flight.index', ['flights' => $flights]);
    }
}
~~~

#### 获取 Column 值

如果你有一个 Eloquent 模型实例，你可以通过访问 model 相对应的属性来获取对应的 column值，例如，让我们来遍历每个 `Flight` 我们查询返回的实例并且输出列名为 `name` 的值:

~~~
foreach ($flights as $flight) {
    echo $flight->name;
}
~~~

#### 添加额外约束

Eloquent `all` 方法返回 model's 表中的所有的结果集。由于每个Eloquent 模型服务相当于一个[查询构造器](http://laravel.com/docs/5.1/queries)，你可以在查询中加入约束条件，然后使用 `get` 方法取回结果集:

~~~
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
~~~

> **注意:** 由于Eloquent模型是查询构造器，你将要复习[查询构造器](http://laravel.com/docs/5.1/queries)所有有效的方法。在 Eloquent 查询中使用这些方法中的任何方法。

#### 集合

Eloquent 方法如 `all` 和 `get` 都是取回多个结果，返回了一个`Illuminate\Database\Eloquent\Collection` 实例。该 `Collection` 类提供了使用于 Eloquent 结果集的 [a variety of helpful methods](http://laravel.com/docs/5.1/eloquent-collections)。当然，你可以像遍历数组一般简单地遍历 collection:

~~~
foreach ($flights as $flight) {
    echo $flight->name;
}
~~~

#### 分割结果

如果你需要处理数以千计的 Eloquent 记录,使用 `chunk` 命令。该 `chunk` 方法将取回 Eloquent 模型的一个 "chunk"，喂养一个给定的`闭包`进行处理。当处理比较大的结果集合时使用 `chunk` 方法比较节约内存。

~~~
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
~~~

该方法的第一个参数设置每一个 "chunk" 有多少条记录。第二个参数是调用从数据库中检索到的每一个数据块的闭包。

## [取得单一模型/集合](http://laravel.com/docs/5.1/eloquent#retrieving-single-models)

当然，除了在给定的表中取得所有的记录以外，你可以使用 `find` 和 `first` 取得单条记录。替代返回的模型集合，`find` 和 `first` 方法返回单一的模型实例:

~~~
// Retrieve a model by its primary key...
$flight = App\Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = App\Flight::where('active', 1)->first();
~~~

#### 未发现异常

有时候你希望当没有找到一个模型时抛出异常，尤其是在路由或者控制器中相当有用。无论如何，`findOrFail` 和 `firstOrFail` 方法都会取得一条查询结果记录。如果没有找到记录就会抛出`Illuminate\Database\Eloquent\ModelNotFoundException` 异常。

~~~
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
~~~

如果没有捕获到异常，将自动发送回给用户 `404` HTTP 响应，所以当我们在使用这些方法时没有必要编写显式检查返回的 `404` 响应:

~~~
Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});
~~~



### 取得集合

当然，你也可以使用查询构造器集合函数像 `count`, `sum`, `max` 和其他[查询构造器](http://laravel.com/docs/5.1/queries)提供的集合函数。这些方法返回适当的数量值代替一个完整的模型实例:

~~~
$count = App\Flight::where('active', 1)->count();

$max = App\Flight::where('active', 1)->max('price');
~~~



## [插入和更新模型](http://laravel.com/docs/5.1/eloquent#inserting-and-updating-models)



###基本插入

在数据库中创建一条新纪录，简单地创造一个新模型实例，设置参数，然后调用 `save` 方法:

~~~
<?php

namespace App\Http\Controllers;

use App\Flight;
use Illuminate\Http\Request;
use App\Http\Controllers\Controller;

class FlightController extends Controller
{
    /**
     * Create a new flight instance.
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}
~~~

在这个例子中，我们把 HTTP 请求进来的 `name` 参数赋值给 `App\Flight` 模型实例的 `name` 变量，当我们调用 `save` 方法时就会向数据库中插入一条记录。当调用 `save` 方法时 `created_at` 和 `updated_at` 时间戳就会自动设置，不需要我们手动去设置。

### 基本更新

`save` 方法可以使用于更新数据库中已经存在的模型。更新模型，首先你必须得到模型，并设置要更新的参数，然后调用 `save` 方法。此外，`updated_at` 时间戳会自动更新，所有不需要手动设置 `updated_at` 值:

~~~
$flight = App\Flight::find(1);

$flight->name = 'New Flight Name';

$flight->save();
~~~

更新也可以执行针对与给定查询相匹配的的任何模型。在下面例子中，所有 `active` 和 `destination` 是 `San Diego` 的 flights 被标记为延时:

~~~
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
~~~

`update`方法期望更新由列值对表示的块数组。

### Mass Assignment

你可以使用 `create` 方法保存新的模型。该方法返回了插入的模型实例，在使用 `create` 保存模型前，需要在模型上指定 `fillable` 或者 `guarded` 属性，所有的 Eloquent 模型都会防卫不利的 mass-assignment。

当用户请求通过意外的 HTTP 参数时会出现 mass-assignment 漏洞，并且那个参数并不是你期望用来修改数据库列的参数，例如，怀有恶意的用户可能通过 HTTP 请求发送映射在 `create` 方法的 `is_admin` 参数，升级为超级管理员。

因此,必须定义一个用于构造 mass assignable 模型的属性。在模型上使用 `$fillable` 属性。例如,在 mass assignable 模型 `Flight` 上构造 `name` 属性:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = ['name'];
}
~~~

一旦我们构造了 mass assignable 属性,我们就能使用 `create` 方法向数据库插入一条新纪录。`create` 方法返回一个保存好的模型实例：

~~~
$flight = App\Flight::create(['name' => 'Flight 10']);
~~~

`$fillable` 属性类似 mass assignable "白名单" 的作用，你可以选择使用 `$guarded` 属性。`$guarded` 属性数组包含不想要 mass assignable 的属性。不在该数组内的所有其他的属性都要 mass assignable。所以，`$guarded` 函数就像一张 "黑名单" 。你只能选择 `$fillable` 和 `$guarded` 其中的一个，不能两个同时使用:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The attributes that aren't mass assignable.
     *
     * @var array
     */
    protected $guarded = ['price'];
}
~~~

在上面的例子中， **除了`price`** 以外所有的属性都会被 mass assignable。

#### 其他创建方法

还可以使用 `firstOrCreate` 和`firstOrNew` 方法创建 mass assigning 模型属性。`firstOrCreate` 方法使用给定的列/值对尝试定位一条数据库记录。如果在数据库中没有找到该模型，则插入一条给定了属性的记录。

`firstOrNew` 方法类似于 `firstOrCreate` 方法，使用给定的属性对尝试定位一条数据库记录。无论如何,即使没有找到模型，也会返回一个新的模拟实例。注意: `firstOrNew` 方法返回的模型也不存在数据库时。需要调 `save`手动保存它:

~~~
// Retrieve the flight by the attributes, or create it if it doesn't exist...
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

// Retrieve the flight by the attributes, or instantiate a new instance...
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);
~~~



## [删除模型](http://laravel.com/docs/5.1/eloquent#deleting-models)

要删除一个模型，只需要在模型实例中调用`delete`方法:

~~~
$flight = App\Flight::find(1);

$flight->delete();
~~~

#### 通过键值删除一个已存在的模型

在上面的例子中，我们在调用 `delete` 方法前取得数据库中的模型。然而，如果你知道模型的主键，你就可以在没有得到模型的情况下删除该模型。像这样调用 `destroy` 方法:

~~~
App\Flight::destroy(1);

App\Flight::destroy([1, 2, 3]);

App\Flight::destroy(1, 2, 3);
~~~

#### 通过查询删除模型

当然了，你也可以在一组模型上运行删除查询。这下面的例子中，我们将删除所有的被标记为inactive的flights:

~~~
$deletedRows = App\Flight::where('votes', '>', 100)->delete();
~~~

### 软删除

除了真实的移除数据库的记录之外，Eloquent 可以"软删除"模型。当模型被软删除时，并没有真正的从你的数据库中删除。而是在模型上设置 `deleted_at` 属性并把它插入到数据库中。如果一个模型的 `deleted_at` 值为非空，那么该模型已经被软删除。若要在模型中启用软删除，则需要在模型中使用`Illuminate\Database\Eloquent\SoftDeletes` 的特性，并把 `deleted_at` 列加入到 `$dates` 属性中：


~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;

    /**
     * The attributes that should be mutated to dates.
     *
     * @var array
     */
    protected $dates = ['deleted_at'];
}
~~~

当然，你可以在你的数据库表中添加 `deleted_at` 列。 Laravel的[构造生成器](http://laravel.com/docs/5.1/schema)包含了创建该列的铺助方法:

~~~
Schema::table('flights', function ($table) {
    $table->softDeletes();
});
~~~

现在，当你在模型上调用 `delete` 方法时，`deleted_at` 列就会被设置成当前日期和时间。还有当我们正使用软删除查询模型时，所有的查询结果就会自动过滤掉软删除模型。

使用 `trashed` 方法，确定一个模型实例是否已经软删除了。

~~~
if ($flight->trashed()) {
    //
}
~~~



### 查询软删除模型

#### 包含软删除模型

如上面所看到的一样，软删除模型会被查询结果自动过滤掉，然而，你可以在查询上使用 `withTrashed` 方法在结果中强行显示软删除:

~~~
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
~~~

`withTrashed` 方法也可以使用[关联](http://laravel.com/docs/5.1/eloquent-relationships)查询:

~~~
$flight->history()->withTrashed()->get();
~~~

#### 检索唯一的软删除模型

`onlyTrashed` 方法用于检索**唯一的**软删除模型:

~~~
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
~~~

#### 恢复软删除模型

有时候你希望 "un-delete" 一个软删除模型。在模型实例上使用 `restore` 方法，恢复软删除模型进入活动状态:

~~~
$flight->restore();
~~~


你也可以在查询中使用 `restore` 方法快速恢复多模型:
~~~
App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
~~~

像 `withTrashed` 方法一样，`restore` 方法也可以使用[关联](http://laravel.com/docs/5.1/eloquent-relationships):

~~~
$flight->history()->restore();
~~~

#### 永久性删除模型

有时候你需要真正的从数据库中移除一个模型。使用 `forceDelete` 方法，永久性的数据库中移除软删除模型:

~~~
// Force deleting a single model instance...
$flight->forceDelete();

// Force deleting all related models...
$flight->history()->forceDelete();
~~~



## [查询范围](http://laravel.com/docs/5.1/eloquent#query-scopes)

范围允许定义常见的约束集，以至于容易地重用于整个应用程序。例如，我们需要经常检索定义一个范围内所有被认为"受欢迎的"的用户。 `scope` 作为 Eloquent 模型方法的前缀:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}
~~~

#### 使用范围检索

一旦定义了scope，当检索模型时你可以调用 scope 方法。然而，调用该方法时不需要包含 `scope` 前缀。甚至可以链式调用多个scopes，例如:

~~~
$users = App\User::popular()->women()->orderBy('created_at')->get();
~~~

#### 动态范围

有时候你可能希望定义一个可以接受参数的scope。马上开始，仅是添加附加参数到你的 scope。Scope 参数必须定义在 `$query` 参数之后:

~~~
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     *
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
~~~

现在，当调用 scope 时你可以传递参数了:

~~~
$users = App\User::ofType('admin')->get();
~~~



## [事件](http://laravel.com/docs/5.1/eloquent#events)

Eloquent 模型激发多个事件，允许在模型的生命周期使用的`creating`, `created`, `updating`, `updated`, `saving`, `saved`,`deleting`, `deleted`, `restoring`, `restored` 方法的各个点挂钩。事件允许你轻松地执行代码，每一次执行都会在数据库中保存或是更新一个特殊的模型类。

### 基本用法

无论何时都会第一时间保存一个新的模型，激发 `creating` 和 `created` 事。 如果模型存在于数据库中，并且`save` 方法被调用了，就会激发 `updating` / `updated` 事件。无论如何，都会激发 `saving` / `saved `事件。

例如，让我们来定义一个 Eloquent 事件监听[服务提供者](http://laravel.com/docs/5.1/providers)。在我们事件监听的里面，我们要在给定的模型上调用 `isValid` 方法，如果该模型是无效的，则返回 `false` 。Eloquent 事件监听者返回的 `false` 将取消 `save` / `update` 操作:

~~~
<?php

namespace App\Providers;

use App\User;
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
        User::creating(function ($user) {
            if ( ! $user->isValid()) {
                return false;
            }
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
~~~
