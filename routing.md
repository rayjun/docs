# 路由

- [基本路由](#basic-routing)
- [路由参数](#route-parameters)
    - [必选参数](#required-parameters)
	- [可选参数](#parameters-optional-parameters)
	- [正则表达式约束](#parameters-regular-expression-constraints)
- [命名空间](#named-routes)
- [路由分组](#route-groups)
	- [中间件](#route-group-middleware)
	- [命名空间](#route-group-namespaces)
	- [子域名路由](#route-group-sub-domain-routing)
	- [路由前缀](#route-group-prefixes)
- [CSRF保护](#csrf-protection)
	- [介绍](#csrf-introduction)
	- [Excluding URIs](#csrf-excluding-uris)
	- [X-CSRF-Token](#csrf-x-csrf-token)
	- [X-XSRF-Token](#csrf-x-xsrf-token)
- [方法欺骗](#form-method-spoofing)
- [抛出404错误](#throwing-404-errors)

<a name="basic-routing"></a>
## 基本路由

你可以在 `app/Http/routes.php` 中为你的应用定义路由规则，路由规则由 `App\Providers\RouteServiceProvider` 进行加载. 通常情况下，Laravel 路由需要一个URL和一个闭包函数：

	Route::get('/', function () {
		return 'Hello World';
	});

	Route::post('foo/bar', function () {
		return 'Hello World';
	});

	Route::put('foo/bar', function () {
		//
	});

	Route::delete('foo/bar', function () {
		//
	});

#### 为多种请求注册路由

有时候你需要用一个路由去匹配多种请求，你可以在 Route 中使用 `match` [facade](/docs/{{version}}/facades)：

	Route::match(['get', 'post'], '/', function () {
		return 'Hello World';
	});

或者你可以直接使用 `any` 去匹配所有请求：

	Route::any('foo', function () {
		return 'Hello World';
	});

#### 生成URL

你可以生成用 `url` 方法去生成一个 URL：

	$url = url('foo');

<a name="route-parameters"></a>
## 路由参数

<a name="required-parameters"></a>
### 必选参数

有时候你需要在 URL 中获取需要的参数，比如你需要在URL中获取用户的 ID ，你可以这样定义路由：

	Route::get('user/{id}', function ($id) {
		return 'User '.$id;
	});

或者获取多个参数：

	Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
		//
	});

路由中的参数需要放在 `{}` 中， 	参数会被传递到闭包函数。

> **Note:** 路由中不应该用 `-`，而是用 `_`。

<a name="parameters-optional-parameters"></a>
### 可选参数

有时你不确定URL中是否存在这个参数，你可以使用 `?` 来初始化这个参数：

	Route::get('user/{name?}', function ($name = null) {
		return $name;
	});

	Route::get('user/{name?}', function ($name = 'John') {
		return $name;
	});

<a name="parameters-regular-expression-constraints"></a>
### 正则表达式约束

你可以使用正则表达式约束传递来的参数，`where` 方法接受两个参数，需要约束的变量名和正则表达式，或者直接使用数组：

	Route::get('user/{name}', function ($name) {
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function ($id) {
		//
	})
	->where('id', '[0-9]+');

	Route::get('user/{id}/{name}', function ($id, $name) {
		//
	})
	->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全局约束

如果你想要自定义一个全局的约束，可以使用 `pattern` 的方法。这个方法在`RouteServiceProvider` 的 `boot` 方法中进行定义：

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @param  \Illuminate\Routing\Router  $router
     * @return void
     */
    public function boot(Router $router)
    {
		$router->pattern('id', '[0-9]+');

        parent::boot($router);
    }

定义之后，所有的指定名称都会被约束：

	Route::get('user/{id}', function ($id) {
		// 只验证$id是否为数字.
	});

<a name="named-routes"></a>
## 命名路由

命名路线将一个URL指定到一个特定的路由。你可以使用 `as` 将一个URL指定为另一个URL：

	Route::get('user/profile', ['as' => 'profile', function () {
		//
	}]);

你也可以指定到某个控制器：

	Route::get('user/profile', [
		'as' => 'profile', 'uses' => 'UserController@showProfile'
	]);

#### 路由分组和命名路由

如果你使用 [路由分组](#route-groups)，你可以在路由组中指定一个 `as` 关键词，并允许组内的路由共享这个关键词：

	Route::group(['as' => 'admin::'], function () {
		Route::get('dashboard', ['as' => 'dashboard', function () {
			// Route named "admin::dashboard"
		}]);
	});

#### 生成命名路由URL

如果你指定了一个路由，可以使用 `route` 函数生成或重定向到该路由：

	$url = route('profile');

	$redirect = redirect()->route('profile');

如果这条路由定义了参数，你可以将参数作为第二个参数传递给 `route` 方法。传递的参数会自动插入到URL中：

	Route::get('user/{id}/profile', ['as' => 'profile', function ($id) {
		//
	}]);

	$url = route('profile', ['id' => 1]);

<a name="route-groups"></a>
## 路由分组

路由分组允许你分享公共的路由属性，比如中间件、命名空间，无需再为每个路由依次设置。将共享的属性做为数组传递给 `Route::group`。

想要了解更多的路由分组，我们接下来会介绍几种常见的功能。

<a name="route-group-middleware"></a>
### 中间件

你可以在路由分组中使用中间件，用数组的形式传递中间件，路由会依次去执行：

	Route::group(['middleware' => 'auth'], function () {
		Route::get('/', function ()	{
			// 使用Auth中间件
		});

		Route::get('user/profile', function () {
			// 使用Auth中间件
		});
	});

<a name="route-group-namespaces"></a>
### 命名空间

你还可以在路由分组中分配命名空间，同样以数组的形式去传递

	Route::group(['namespace' => 'Admin'], function()
	{
		// Controllers Within The "App\Http\Controllers\Admin" Namespace

		Route::group(['namespace' => 'User'], function()
		{
			// Controllers Within The "App\Http\Controllers\Admin\User" Namespace
		});
	});

默认情况下 `RouteServiceProvider` 会包含 `routes.php` 的命名空间，所以你只需要指定 `App\Http\Controllers` 之后的部分。

<a name="route-group-sub-domain-routing"></a>
### 子域名路由

路由分组也可以去匹配子域名。使用通配符可以捕获域名的一部分并传递给路由或控制器：

	Route::group(['domain' => '{account}.myapp.com'], function () {
		Route::get('user/{id}', function ($account, $id) {
			//
		});
	});

<a name="route-group-prefixes"></a>
### 路由前缀

你可以为分组内的route增加前缀。下面的示例为组内的路由增加 `admin` 前缀：

	Route::group(['prefix' => 'admin'], function () {
		Route::get('users', function ()	{
			// 匹配为 "/admin/users" 
		});
	});

你可以在前缀中匹配某些参数：

	Route::group(['prefix' => 'accounts/{account_id}'], function () {
		Route::get('detail', function ($account_id)	{
			// Matches The accounts/{account_id}/detail URL
		});
	});

<a name="csrf-protection"></a>
## CSRF 防护

<a name="csrf-introduction"></a>
### CSRF 介绍

Laravel 提供简单的方法，可以保护您的应用程序不受到 [跨站攻击](http://en.wikipedia.org/wiki/Cross-site_request_forgery)。跨网站请求伪造是一种恶意的攻击，伪造已经通过身份验证的用户执行授权的命令。

Laravel会自动生成一个 token 存放在用户 session 中。这个 token 可以保证请求是通过应用正常生成的请求，你可以使用 `csrf`：

	<?php echo csrf_field(); ?>

你也可以在HTML中使用：

	<input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

当然也可以使用 Blade [模板引擎](/docs/{{version}}/blade)：

	{!! csrf_field() !!}

在POST、PUT、DELETE请求中你不需要手动验证CSRF令牌. `VerifyCsrfToken` [HTTP中间件](/docs/{{version}}/middleware) 会自动验证 session 中的CSRF令牌.

<a name="csrf-excluding-uris"></a>
### 不使URIs受CSRF保护

有时你希望特定的URL不受CSRF的保护。比如，如果你正在使用 [Stripe](https://stripe.com) 来处理支付或者 webhook 系统，你需要去除这些URL的SCRF的保护 

你可以将他们添加到 `$except` 的 `VerifyCsrfToken` 中间件中去：

	<?php

	namespace App\Http\Middleware;

	use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as BaseVerifier;

	class VerifyCsrfToken extends BaseVerifier
	{
	    /**
	     * The URIs that should be excluded from CSRF verification.
	     *
	     * @var array
	     */
	    protected $except = [
	        'stripe/*',
	    ];
	}

<a name="csrf-x-csrf-token"></a>
### X-CSRF-TOKEN

除了检查 CSRF令牌 ，`VerifyCsrfToken` 中间件也可以检查 `X-CSRF-TOKEN` 请求。比如将令牌存在 "meta" 标签中：

	<meta name="csrf-token" content="{{ csrf_token() }}">

如果你创建了 `meta` 标签，你可以使用类似 jQuery 去添加所有的请求头：

	$.ajaxSetup({
			headers: {
				'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
			}
	});

<a name="csrf-x-xsrf-token"></a>
### X-XSRF-TOKEN

Laravel还在 cookie 中储存了 `XSRF-TOKEN` 。 你可以使用cookie的值来设置 `X-XSRF-TOKEN` 请求头. 一些 JavaScript 框架，比如Angular 会自动为你处理这些事情。

<a name="form-method-spoofing"></a>
## 方法欺骗

HTML 表单没有支持 `PUT` 、`PATCH` 或 `DELETE` 请求。所以，你可以在表单中定义一个 `_method` 字段，这个字段的值会被当做HTTP的请求去处理：

	<form action="/foo/bar" method="POST">
		<input type="hidden" name="_method" value="PUT">
		<input type="hidden" name="_token" value="{{ csrf_token() }}">
	</form>

<a name="throwing-404-errors"></a>
## 抛出404错误

有两种方法可以让你手动在路由中抛出404错误。 第一种，使用 `abort` 函数. 这个函数可以简单的抛出 `Symfony\Component\HttpFoundation\Exception\HttpException` 中规定的状态码：

	abort(404);

第二种，你可以手动抛出实体 `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`。

更多信息的有关如何处理 404 异常状况和自定响应，可以参考 [错误](/docs/{{version}}/errors#http-exceptions) 部分的文档。
