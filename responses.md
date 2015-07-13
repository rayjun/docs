# HTTP 响应

- [基础响应](#basic-responses)
	- [附加标头到响应](#attaching-headers-to-responses)
	- [附加 Cookie 到响应](#attaching-Cookie-to-responses)
- [其它响应类型](#other-response-types)
	- [视图响应](#view-responses)
	- [JSON 响应](#json-responses)
	- [文件下载](#file-downloads)
- [重定向](#redirects)
	- [重定向到命名路由](#redirecting-named-routes)
	- [重定向到控制器行为](#redirecting-controller-actions)
	- [重定向与快闪 Session 数据](#redirecting-with-flashed-session-data)
- [响应宏](#response-macros)

<a name="basic-responses"></a>
## 基本响应

当然，所有的路由和控制器都应该返回某种类型的响应并发送回用户的浏览器。Laravel 提供了一些不同的方法来返回响应。最基础的响应是从路由或控制器返回一段字符串：

	Route::get('/', function () {
		return 'Hello World';
	});

给定的字符串将会被框架自动的转换为 HTTP 响应。

但是对大部分路由和控制器来说，你需要返回一个完成的 `Illuminate\Http\Response` 实例或者一个[视图](/docs/{{version}}/views)。返回一个完整的 `Response` 实例能让你定制响应的 HTTP 状态码和标头。一个 `Response` 继承自 `Symfony\Component\HttpFoundation\Response` 类，该类提供各种方法来建立 HTTP 响应：

	use Illuminate\Http\Response;

	Route::get('home', function () {
		return (new Response($content, $status))
		              ->header('Content-Type', $value);
	});

为了方便起见，你可以使用辅助方法 `response` ：

	Route::get('home', function () {
		return response($content, $status)
		              ->header('Content-Type', $value);
	});

> **注意：** 有关完整的 `Response` 方法列表请参照 [API 文档](http://laravel.com/api/master/Illuminate/Http/Response.html)和 [Symfony API documentation](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/Response.html)。

<a name="attaching-headers-to-responses"></a>
#### 附加标头到响应

记住大部分的响应方法都是可链接的，让你建立流利的响应。例如，你可以在响应返回用户之前使用 `header` 方法来添加一系列的标头：

	return response($content)
				->header('Content-Type', $type)
				->header('X-Header-One', 'Header Value')
				->header('X-Header-Two', 'Header Value');


<a name="attaching-Cookie-to-responses"></a>
#### 附加 Cookie 到响应

响应实例的 `withCookie` 方法能让你轻松的附加 Cookie 到响应。例如，你使用 `withCookie` 方法来产生一个 Cookie 并且附加到响应实例：

	return response($content)->header('Content-Type', $type)
                     ->withCookie('name', 'value');

`withCookie` 方法接受额外的可选参数，来允许你进一步定制 Cookie 的属性：

	->withCookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

默认情况下：所有 Laravel 产生的 Cookie 都会被加密并加上签名，所以不能被客户端修改或读取。如果你想将应用程序产生的 Cookie 中的某个子集停止加密，你可以使用 `App\Http\Middleware\EncryptCookie` 中间件的 `$except` 属性：

    /**
     * The names of the Cookie that should not be encrypted.
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="other-response-types"></a>
## 其他响应类型

使用 `response` 辅助方法可以方便的产生其他类型的响应实例。当调用 `response` 辅助方法不传参数时，将返回 `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) 的实例。此 contract 提供一些有用的方法来产生响应。

<a name="view-responses"></a>
#### 视图响应

如果你想要控制响应状态码和标头，但是也想返回一个视图作为响应内容，你可以使用 `view` 方法：

	return response()->view('hello', $data)->header('Content-Type', $type);

当然，如果你没有自定响应码和标头的需求，你可以简单的使用全局辅助方法 `view` 。

<a name="json-responses"></a>
#### JSON 响应

`json` 方法会自动的设置 `Content-Type` 标头为 `application/json` ，也通过 PHP 的 `json_encode` 方法将数组转化为 JSON：

	return response()->json(['name' => 'Abigail', 'state' => 'CA']);

如果你想创建一个 JSONP 响应，你可以使用 `json` 方法并加上 `setCallback`：

	return response()->json(['name' => 'Abigail', 'state' => 'CA'])
	                 ->setCallback($request->input('callback'));

<a name="file-downloads"></a>
#### 文件下载

`download` 方法可产生用于强制用户浏览器来下载给定路径文件的响应。`download` 方法接受文件名称为第二个参数，此名称为用户下载文件时所见的文件名。最后，你可以传递一个 HTTP 标头作为第三个参数：

	return response()->download($pathToFile);

	return response()->download($pathToFile, $name, $headers);

> **注意：** Symfony HttpFoundation，用来管理文件下载，要求下载文件名为 ASCII

<a name="redirects"></a>
## 重定向

重定向响应是 `Illuminate\Http\RedirectResponse` 类的实例，并且包含用户要重定向的另一个 URL 的标头。这里有几种产生 `RedirectResponse` 实例方法的方法。最简单的方法是全局辅助方法 `redirect` ：

	Route::get('dashboard', function () {
		return redirect('home/dashboard');
	});

有时你可能希望用户重定向到前一个位置，例如当提交一个无效的表单之后。你可以使用全局的辅助方法 `back` 来达到这个目的：

	Route::post('user/profile', function () {
		// Validate the request...

		return back()->withInput();
	});

<a name="redirecting-named-routes"></a>
#### 重定向至命名路由

当你调用辅助方法 `redirect` 而且不带任何参数时，返回一个 `Illuminate\Routing\Redirector` 的实例，你可以调用 `Redirector` 的任何方法。例如，要产生一个 `RedirectResponse` 到一个命名路由，你可以使用 `route` 方法：

	return redirect()->route('login');

如果你的路由有参数， 你可以将参数放进 `route` 方法的第二个参数：

	// For a route with the following URI: profile/{id}

	return redirect()->route('profile', [1]);

如果你要重定向到路由且路由的参数为 Eloquent 模型的 ID 参数，那么你可以直接传入模型本身。这个 ID 会被自动获取：

	return redirect()->route('profile', [$user]);

<a name="redirecting-controller-actions"></a>
#### 重定向到控制器行为

你可能想产生重定向到[控制器行为](/docs/{{version}}/controllers)。要做到这一点，只需传入控制器和行为名称到 `action` 方法。记住，你不需要指出完整的命名空间，因为 Laravel 的 `RouteServiceProvider` 自动设定默认的控制器命名空间：

	return redirect()->action('HomeController@index');

当然，如果你的控制器需要参数，你可以将其传入 `action` 方法的第二个参数：

	return redirect()->action('UserController@profile', [1]);

<a name="redirecting-with-flashed-session-data"></a>
#### 重定向和快闪 Session 数据

通常重定向到新的 URL 时同时 [快闪数据到 session](/docs/{{version}}/session#flash-data)。所以为了方便，你可以在一个方法链中创建一个 `RedirectResponse` 实例**同时**闪存数据到 session 中。

	Route::post('user/profile', function () {
		// Update the user's profile...

		return redirect('dashboard')->with('status', 'Profile updated!');
	});

当然，在用户被重定向到一个新的页面，你可以从 [session](/docs/{{version}}/session)中获取显示闪存的信息。例如，使用 [Blade syntax](/docs/{{version}}/blade)：

	@if (session('status'))
		<div class="alert alert-success">
			{{ session('status') }}
		</div>
	@endif

<a name="response-macros"></a>
## 响应宏

如果你想定义一个定制的能在路由和控制器重复使用的响应，你可以使用 `lluminate\Contracts\Routing\ResponseFactory` 实例的 `macro` 方法。

举个例子，来自一个[服务提供者](/docs/{{version}}/providers)的 `boot` 方法：

	<?php

	namespace App\Providers;

	use Illuminate\Support\ServiceProvider;
	use Illuminate\Contracts\Routing\ResponseFactory;

	class ResponseMacroServiceProvider extends ServiceProvider
	{
		/**
		 * Perform post-registration booting of services.
		 *
		 * @param  ResponseFactory  $factory
		 * @return void
		 */
		public function boot(ResponseFactory $factory)
		{
			$factory->macro('caps', function ($value) use ($factory) {
				return $factory->make(strtoupper($value));
			});
		}
	}

`macro` 函数的第一个参数为宏名称，第二个参数为闭包函数。闭包函数会在 `ResponseFactory` 的实做或辅助方法 `response` 调用宏名称的时候被执行：

	return response()->caps('foo');
