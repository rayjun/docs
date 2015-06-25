# 辅助函数（Helper Function）

- [介绍](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 介绍

Laravel 包含各种的 PHP「辅助」函数，其中许多方法用于框架本身，然而，如果你觉得方便，你可以在你的应用程序中随意使用这些方法。

<a name="available-methods"></a>
## 可用方法

<style>
	.collection-method-list > p {
		column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
		column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
	}

	.collection-method-list a {
		display: block;
	}
</style>

### 数组

<div class="collection-method-list" markdown="1">
[array_add](#method-array-add)
[array_divide](#method-array-divide)
[array_dot](#method-array-dot)
[array_except](#method-array-except)
[array_first](#method-array-first)
[array_flatten](#method-array-flatten)
[array_forget](#method-array-forget)
[array_get](#method-array-get)
[array_only](#method-array-only)
[array_pluck](#method-array-pluck)
[array_pull](#method-array-pull)
[array_set](#method-array-set)
[array_sort](#method-array-sort)
[array_where](#method-array-where)
[head](#method-head)
[last](#method-last)
</div>

### 路径

<div class="collection-method-list" markdown="1">
[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[public_path](#method-public-path)
[storage_path](#method-storage-path)
</div>

### 字符串

<div class="collection-method-list" markdown="1">
[camel_case](#method-camel-case)
[class_basename](#method-class-basename)
[e](#method-e)
[ends_with](#method-ends-with)
[snake_case](#method-snake-case)
[str_limit](#method-str-limit)
[starts_with](#method-starts-with)
[str_contains](#method-str-contains)
[str_finish](#method-str-finish)
[str_is](#method-str-is)
[str_plural](#method-str-plural)
[str_random](#method-str-random)
[str_singular](#method-str-singular)
[str_slug](#method-str-slug)
[studly_case](#method-studly-case)
[trans](#method-trans)
[trans_choice](#method-trans-choice)
</div>

### URLs

<div class="collection-method-list" markdown="1">
[action](#method-action)
[route](#method-route)
[url](#method-url)
</div>

### 其它

<div class="collection-method-list" markdown="1">
[config](#method-config)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[elixir](#method-elixir)
[env](#method-env)
[event](#method-event)
[response](#method-response)
[value](#method-value)
[view](#method-view)
[with](#method-with)
</div>

<a name="method-listing"></a>
## 方法列表

<style>
	#collection-method code {
		font-size: 14px;
	}

	#collection-method:not(.first-collection-method) {
		margin-top: 50px;
	}
</style>

<a name="method-array-add"></a>
#### `array_add()` {#collection-method .first-collection-method}

`array_add` 函数添加一个键/值对到数组，仅当给定的键在数组中还不存在时添加：

	$array = array_add(['name' => 'Desk'], 'price', 100);

	// ['name' => 'Desk', 'price' => 100]

<a name="method-array-divide"></a>
#### `array_divide()` {#collection-method}

`array_divide` 函数返回两个数组，分别包含原始数组的键和值：

	list($keys, $values) = array_divide(['name' => 'Desk']);

	// $keys: ['name']

	// $values: ['Desk']

<a name="method-array-dot"></a>
#### `array_dot()` {#collection-method}
`array_dot` 函数将多维数组压缩成一个水平数组，使用点号来表示路径：

	$array = array_dot(['foo' => ['bar' => 'baz']]);

	// ['foo.bar' => 'baz'];

<a name="method-array-except"></a>
#### `array_except()` {#collection-method}

`array_except` 方法将给定的键/值对从数组中移除：

	$array = ['name' => 'Desk', 'price' => 100];

	$array = array_except($array, ['price']);

	// ['name' => 'Desk']

<a name="method-array-first"></a>
#### `array_first()` {#collection-method}

`array_first` 方法返回通过真值测试的第一项：

	$array = [100, 200, 300];

	$value = array_first($array, function ($key, $value) {
		return $value >= 150;
	});

	// 200

还可以传入一个默认值作为第三个参数，如果没有值通过真值测试，将返回此值：

	$value = array_first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `array_flatten()` {#collection-method}

`array_flatten` 将多维数组压缩成一个水平数组。

	$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

	$array = array_flatten($array);

	// ['Joe', 'PHP', 'Ruby'];

<a name="method-array-forget"></a>
#### `array_forget()` {#collection-method}

`array_forget` 函数使用点号从一个深度嵌套的数组中移除一个键/值对：

	$array = ['products' => ['desk' => ['price' => 100]]];

	array_forget($array, 'products.desk');

	// ['products' => []]

<a name="method-array-get"></a>
#### `array_get()` {#collection-method}

`array_get` 函数使用点号从一个深度嵌套的数组中检索一个值：

	$array = ['products' => ['desk' => ['price' => 100]]];

	$value = array_get($array, 'products.desk');

	// ['price' => 100]

这个函数还接受一个默认值，当指定的键没有找到时返回：

	$value = array_get($array, 'names.john', 'default');

<a name="method-array-only"></a>
#### `array_only()` {#collection-method}

`array_only` 只从给定的数组中返回指定的键/值对：

	$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

	$array = array_only($array, ['name', 'price']);

	// ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `array_pluck()` {#collection-method}

`array_pluck` 从数组中摘取一个键/值对列表：

	$array = [
		['developer' => ['name' => 'Taylor']],
		['developer' => ['name' => 'Abigail']]
	];

	$array = array_pluck($array, 'developer.name');

	// ['Taylor', 'Abigail'];

<a name="method-array-pull"></a>
#### `array_pull()` {#collection-method}

`array_pull` 返回且从数组中移除一个键/值对：

	$array = ['name' => 'Desk', 'price' => 100];

	$name = array_pull($array, 'name');

	// $name: Desk

	// $array: ['price' => 100]

<a name="method-array-set"></a>
#### `array_set()` {#collection-method}

`array_set` 函数使用点号将一个值设置到深度嵌套的数组中：

	$array = ['products' => ['desk' => ['price' => 100]]];

	array_set($array, 'products.desk.price', 200);

	// ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-sort"></a>
#### `array_sort()` {#collection-method}

`array_sort` 函数根据闭包（Closure）的结果对数组排序：

	$array = [
		['name' => 'Desk'],
		['name' => 'Chair'],
	];

	$array = array_values(array_sort($array, function ($value) {
		return $value['name'];
	}));

	/*
		[
			['name' => 'Chair'],
			['name' => 'Desk'],
		]
	*/

<a name="method-array-where"></a>
#### `array_where()` {#collection-method}

`array_where` 根据闭包函数过滤数组：

	$array = [100, '200', 300, '400', 500];

	$array = array_where($array, function ($key, $value) {
		return is_string($value);
	});

	// [1 => 200, 3 => 400]

<a name="method-head"></a>
#### `head()` {#collection-method}

`head` 仅返回给定数组的第一项：

	$array = [100, 200, 300];

	$first = head($array);

	// 100

<a name="method-last"></a>
#### `last()` {#collection-method}

`last` 函数返回数组的最后一项：

	$array = [100, 200, 300];

	$last = last($array);

	// 300

<a name="paths"></a>
## 路径

<a name="method-app-path"></a>
#### `app_path()` {#collection-method}

`app_path` 返回 `app` 目录的完全路径：

	$path = app_path();

你还可以使用 `app_path` 函数根据应用程序目录的相对路径生成一个全路径：

	$path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {#collection-method}

`base_path` 函数返回一个指向项目根目录的全路径：

	$path = base_path();

你还可以使用 `base_path` 函数根据给定程序目录的相对路径生成一个完全路径：

	$path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {#collection-method}

`config_path` 返回指向应用程序配置目录的完全路径：

	$path = config_path();

<a name="method-database-path"></a>
#### `database_path()` {#collection-method}

`database_path` 函数返回指向应用程序数据库目录的完全路径：

	$path = database_path();

<a name="method-public-path"></a>
#### `public_path()` {#collection-method}

`public_path` 函数返回指向 `public` 目录的完全路径：

	$path = public_path();

<a name="method-storage-path"></a>
#### `storage_path()` {#collection-method}

`storage_path` 函数返回指向 `storage` 目录的完全路径：

	$path = storage_path();

你还可以使用 `storage_path` 函数根据存储目录的相对路径成生一个完全路径：

	$path = storage_path('app/file.txt');

<a name="strings"></a>
## Strings

<a name="method-camel-case"></a>
#### `camel_case()` {#collection-method}

`camel_case` 将字符串转换成 `camelCase`:

	$camel = camel_case('foo_bar');

	// fooBar

<a name="method-class-basename"></a>
#### `class_basename()` {#collection-method}

`class_basename` 返回去掉命名空间的类名：

	$class = class_basename('Foo\Bar\Baz');

	// Baz

<a name="method-e"></a>
#### `e()` {#collection-method}

此函数运行对字符串执行 `htmlentities`:

	echo e('<html>foo</html>');

<a name="method-ends-with"></a>
#### `ends_with()` {#collection-method}

判断字符串是否以某个值结尾：

	$value = ends_with('This is my name', 'name');

	// true

<a name="method-snake-case"></a>
#### `snake_case()` {#collection-method}

将字符串转换成 `snake_case` 类型：

	$snake = snake_case('fooBar');

	// foo_bar

<a name="method-str-limit"></a>
#### `str_limit()` {#collection-method}

`str_limit` 用于限制字符串中字符的数量，此函数分别接受一个字符串作为第一个参数，和一个结果字符串字最大符数数量作为第二个参数：

	$value = str_limit('The PHP framework for web artisans.', 7);

	// The PHP...

<a name="method-starts-with"></a>
#### `starts_with()` {#collection-method}

`starts_with` 函数判断给定的字符串是否以某个值开始：

	$value = starts_with('This is my name', 'This');

	// true

<a name="method-str-contains"></a>
#### `str_contains()` {#collection-method}

`str_contains` 函数用于判断给定的字符串中是否包含某个值：

	$value = str_contains('This is my name', 'my');

	// true

<a name="method-str-finish"></a>
#### `str_finish()` {#collection-method}

`str_finish` 函数将某个值的一个实例添加到一个字符串中：

	$string = str_finish('this/string', '/');

	// this/string/

<a name="method-str-is"></a>
#### `str_is()` {#collection-method}

判断某个字符串是否与给定的模式相匹配，星号可以用于表示全匹配：

	$value = str_is('foo*', 'foobar');

	// true

	$value = str_is('baz*', 'foobar');

	// false

<a name="method-str-plural"></a>
#### `str_plural()` {#collection-method}

`str_plural` 将一个字符串转换成复数形式，目前只支持英文：

	$plural = str_plural('car');

	// cars

	$plural = str_plural('child');

	// children

<a name="method-str-random"></a>
#### `str_random()` {#collection-method}

`str_random` 根据指定的长度生成一个随机字符串：

	$string = str_random(40);

<a name="method-str-singular"></a>
#### `str_singular()` {#collection-method}

`str_singular` 函数将一个字符串转换成其单数形式，这个函数现在只支持英文：

	$singular = str_singular('cars');

	// car

<a name="method-str-slug"></a>
#### `str_slug()` {#collection-method}

`str_slug` 函数将一个给定的字符串转换成友好的 URL 短语：

	$title = str_slug("Laravel 5 Framework", "-");

	// laravel-5-framework

<a name="method-studly-case"></a>
#### `studly_case()` {#collection-method}

`studly_case` 函数将给定的字符串转换成 `StudlyCase` 形式：

	$value = studly_case('foo_bar');

	// FooBar

<a name="method-trans"></a>
#### `trans()` {#collection-method}

`trans` 函数根据[本地化语言文件](/docs/{{version}}/localization)翻译给定的语句:

	echo trans('validation.required'):

<a name="method-trans-choice"></a>
#### `trans_choice()` {#collection-method}

`trans_choice` 函数利用反射机制翻译给定的语句：

	$value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {#collection-method}

`action` 函数为某个控制器动作（controller action）生成一个 URL，你不需要将完整的命名空间传入控制器中，而是传入 `App\Http\Controllers` 命名空间下的控制器的名字：

	$url = action('HomeController@getIndex');

如果方法接受路由参数，你可它们作为第二个参数传入这个方法中：

	$url = action('UserController@profile', ['id' => 1]);

<a name="method-route"></a>
#### `route()` {#collection-method}

`route` 函数为给定的命名路由生成一个 URL：

	$url = route('routeName');

如果路由接受参数，你可它们作为第二个参数传入这个方法中：

	$url = route('routeName', ['id' => 1]);

<a name="method-url"></a>
#### `url()` {#collection-method}

`url` 将给定的路径生成全路径 URL：

	echo url('user/profile');

	echo url('user/profile', [1]);

<a name="miscellaneous"></a>
## 其它方法

<a name="method-config"></a>
#### `config()` {#collection-method}

`config` 函数获取配置项的值，可以使用点号连接文件名加配置项的方法来访问，可以指定一个默认值当配置项不存在时返回：

	$value = config('app.timezone');

	$value = config('app.timezone', $default);

<a name="method-csrf-field"></a>
#### `csrf_field()` {#collection-method}

`csrf_field` 生成一个包含 CSRF 标记的`hidden` HTML 输入框，例如，使用[Blade 语法](/docs/{{version}}/blade):

	{!! csrf_field() !!}

<a name="method-csrf-token"></a>
#### `csrf_token()` {#collection-method}

`csrf_token` 获取当前的 CSRF 值：

	$token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {#collection-method}

`dd` 函数输出给定变量的值并中止当前脚本的执行：

	dd($value);

<a name="method-elixir"></a>
#### `elixir()` {#collection-method}

`elixir` 函数获取[Elixir](/docs/{{version}}/elixir) 当前版本文件：

	elixir($file);

<a name="method-env"></a>
#### `env()` {#collection-method}

`env` 函数获取环境变量的值或者返回一个默认值：

	$env = env('APP_ENV');

	// Return a default value if the variable doesn't exist...
	$env = env('APP_ENV', 'production');

<a name="method-event"></a>
#### `event()` {#collection-method}

`event` 函数将[事件](/docs/{{version}}/events)转发给相应的监听器：

	event(new UserRegistered($user));

<a name="method-response"></a>
#### `response()` {#collection-method}

`response` 生成一个[响应](/docs/{{version}}/responses)实例或者获取一个响应工厂的实例：

	return response('Hello World', 200, $headers);

	return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-value"></a>
#### `value()` {#collection-method}

`value` 仅返回给定的值，然后，如果你传入一个闭包，则返回此闭包的执行结果：

	$value = value(function() { return 'bar'; });

<a name="method-view"></a>
#### `view()` {#collection-method}

`view` 函数获取一个[视图](/docs/{{version}}/views)实例：

	return view('auth.login');

<a name="method-with"></a>
#### `with()` {#collection-method}

`with` 这个函数主要用在方法链执行时返回给定的值，否则无返回:

	$value = with(new Foo)->work();
