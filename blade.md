# Blade 模板引擎

- [介绍](#introduction)
- [模板继承](#template-inheritance)
	- [定义布局模板](#defining-a-layout)
	- [扩展布局模板](#extending-a-layout)
- [显示数据](#displaying-data)
- [流程控制](#control-structures)
- [服务注入](#service-injection)
- [扩展Blade](#extending-blade)

<a name="introduction"></a>
## 介绍

Blade 模板是由 Laravel 提供的简单又强大的模板引擎。跟其它流行的 PHP 模板引擎不同的是， Blade 不会限制你在视图中使用普通的PHP代码。在被修改之前，所有的视图都会被转换为被编译成普通的 PHP 代码并生成缓存，这代表着 Blade 对整个系统而言不会增加额外的负担。Blade 视图文件使用 `.blade.php` 作为扩展名，存储在 `resources/views` 文件夹。

<a name="template-inheritance"></a>
## 模板继承

<a name="defining-a-layout"></a>
### 定义布局模板

Blade 有两个主要优点：模板继承（template inheritance） 和 视图片段（sections）。首先，我们可以查看一个简单的示例 "master" 布局模板，因为大部分网站都会在页面之间使用相同的布局，所以将这个布局定义在一个单独的 Blade 视图中方便其他页面使用


	<!-- Stored in resources/views/layouts/master.blade.php -->

	<html>
		<head>
			<title>App Name - @yield('title')</title>
		</head>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

如你所见, 这个文件包含了典型的 HTML 语法。然而, 我们注意到了 `@section` 和 `@yield` 指令, 和她们的名字一样, `@section` 指定了一个内容部分, `@yield` 显示一个已经定义的内容区域

现在我们已经为应用定义了一个布局, 接下来我们定义一个继承布局的子页面。

<a name="extending-a-layout"></a>
### 扩展布局模板

当定义一个子页面时, 你可以使用 Blade 的 `@extends` 指令去继承一个布局。 在你想要替换内容布局的部分使用 `@section` 指令，记住，在这个例子中可以看到， `@section` 的内容会显示在原本 `@yield` 的地方:

	<!-- Stored in resources/views/layouts/child.blade.php -->

	@extends('layouts.master')

	@section('title', 'Page Title')

	@section('sidebar')
		@@parent

		<p>This is appended to the master sidebar.</p>
	@endsection

	@section('content')
		<p>This is my body content.</p>
	@endsection

在上面的示例中, `@sidebar` 利用了 @@parent 指令显示了（ master.blade.php ）的內容. `@@parent` 指令会将父页面的内容渲染到视图中
当然，就像一般的 PHP 视图， Blade 可以在 routes 中使用 view 函数进行显示：

	Route::get('blade', function () {
		return view('child');
	});

<a name="displaying-data"></a>
## 显示数据
你可以用数组形式传递数据到 Blade 视图中。举例来说，就像下面的路由设定

	Route::get('greeting', function () {
		return view('welcome', ['name' => 'Samantha']);
	});

你可以这样显示name变量的内容:

	Hello, {{ $name }}.

当然，你不仅仅只局限于展示传递来的变量，你也可以直接输出任何PHP的函数。事实上，你可以在 Blade 模板的双花括号表达式中调用任何 PHP 代码：

	The current UNIX timestamp is {{ time() }}.

> **Note:** Blade 中的 {{ }} 表达式的返回值将被自动传递给 PHP 的 htmlentities 函数进行处理，以防止 XSS 攻击。

#### Blade 和 JavaScript 框架共同使用

因为很多 JavaScript 框架也使用 `{{}}` 来展示变量内容，你可以使用 `@` 符号来告知 Blade 这个表达式需要原样输出，就像下面的示例

	<h1>Laravel</h1>

	Hello, @{{ name }}.

在这个示例中， `@` 符号会 Blade 被移除，同时会在浏览器上直接輸出 `{{ name }}` ，这样就可以被 JavaScript 框架解析了。

#### 输出存在的数据

有时你想要输出一个变量, 但是你不确定这个变量是否被赋值. 我们可以使用如下的 PHP 代码:

	{{ isset($name) ? $name : 'Default' }}

如果你不想使用三元运算符，Blade 也提供了更简单更方便的办法

	{{ $name or 'Default' }}

在示例中, 如果 `name` 已经被赋值, 就会直接显示变量值，否则会直接显示 Default 

#### 显示原始数据

默认情况下为了防止XSS攻击，Blade中使用 `{{}}` 会自动套用PHP的 `htmlentities` 函数，如果你想要原始数据，你可以使用如下方法

	Hello, {!! $name !!}.

> **Note:** 一定要小心用户上传的内容. 推荐一直使用 `{{}}` 去转义HTML内容。

<a name="control-structures"></a>
## 流程控制

除了模板继承和显示数据，Blade也提供了方便简洁流程控制命令，比如条件语句和循环.这些指令将提供非常干净简洁的PHP流程控制方式，使用方法和PHP几乎一致

#### IF 语句

和PHP语法类似，你可以使用 `@if`,`@elseif`,`@else` 和 `@endif` 指令去构建IF语句

	@if (count($records) === 1)
		I have one record!
	@elseif (count($records) > 1)
		I have multiple records!
	@else
		I don't have any records!
	@endif

为了方便, Blade 也提供了 `@unless` 指令：

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### 循环

除了条件语句, Blade 也提供了和PHP一致的循环表达式

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@forelse ($users as $user)
		<li>{{ $user->name }}</li>
	@empty
		<p>No users</p>
	@endforelse

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### 引入子视图

Blade 中的 `@include` 指令, 允许你方便的引入一个 Blade 视图到当前视图中。 父视图可用的变量在子视图中也可以使用：

	<div>
		@include('shared.errors')

		<form>
			<!-- Form Contents -->
		</form>
	</div>

虽然引入视图将继承父视图中的所有数据，但你也可以通过数组包含额外的数据到引入的视图中

	@include('view.name', ['some' => 'data'])

#### 注释

Blade 也允许你在视图中写注释。然而，和HTML注释不同，Blade的注释不会被引入到最终的HTML应用中

	{{-- This comment will not be present in the rendered HTML --}}

<a name="service-injection"></a>
## 服务注入


`@inject` 指令可以取出 Laravel 的服务容器(/docs/{{version}}/container)。在 `@inject` 的第一个参数表示服务容器在页面所代表的变量名；第二个参数表示这个服务容器中的解析位置：

	@inject('metrics', 'App\Services\MetricsService')

	<div>
		Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
	</div>

<a name="extending-blade"></a>
## 扩展Blade

Blade 允许你自定义更多的指令。你可以使用 `directive` 方法注册一个指令,当 Blade 解析到该指令时，她会传回她的参数

下面会创建一个 `@datetime($var)` 指令：

	<?php

	namespace App\Providers;

	use Blade;
	use Illuminate\Support\ServiceProvider;

	class AppServiceProvider extends ServiceProvider
	{
		/**
		 * Perform post-registration booting of services.
		 *
		 * @return void
		 */
		public function boot()
		{
			Blade::directive('datetime', function($expression) {
				return "<?php echo with{$expression}->format('m/d/Y H:i'); ?>";
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
	
如你所见, Laravel的 `with` 函数被用在这个指令中。 `with` 函数会简单地传回对象或属性, 并允许使用链式方法. 最后生成如下指令:

	<?php echo with($var)->format('m/d/Y H:i'); ?>
