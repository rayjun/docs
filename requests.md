# HTTP 请求

- [获取请求](#accessing-the-request)
	- [基础请求信息](#basic-request-information)
	- [PSR-7 请求](#psr7-requests)
	- [获取输入数据](#retrieving-input)
	- [旧输入数据](#old-input)
	- [Cookies](#cookies)
	- [上传文件](#files)

<a name="accessing-the-request"></a>
## 取得请求

通过依赖注入的方法获取 HTTP 请求的实例，你需要在控制器的构造函数或方法中使用 `Illuminate\Http\Request` 类作类型提示。当前的请求就会被自动注入[服务容器](/docs/{{version}}/container)：

	<?php

	namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
		/**
		 * Store a new user.
		 *
		 * @param  Request  $request
		 * @return Response
		 */
		public function store(Request $request)
		{
			$name = $request->input('name');

			//
		}
	}

如果你的控制器方法也要从路由参数获取输入数据，只需要将路由参数放在其他依赖之后。例如，你的路由像这样定义：

	Route::put('user/{id}', 'UserController@update');

像下面一样定义控制器方法，你一样可以使用 `Illuminate\Http\Request` 类型提示，同时获取你的路由参数 `id`：

	<?php

	namespace App\Http\Controllers;

	use Illuminate\Http\Request;
	use Illuminate\Routing\Controller;

	class UserController extends Controller
	{
		/**
		 * Update the specified user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function update(Request $request, $id)
		{
			//
		}
	}

<a name="basic-request-information"></a>
### 基础请求信息

`Illuminate\Http\Request` 实例提供了各种方法来检查应用程序的 HTTP 请求。Laravel 的 `Illuminate\Http\Request` 继承了 `Symfony\Component\HttpFoundation\Request` 类。这里是该类的一些有用的方法。

#### 获取请求的 URI

`path` 方法返回请求的 URI。所以，如果接受到的请求目标是 `http://domain.com/foo/bar`，`path` 方法将返回 `foo/bar`:

	$uri = $request->path();

`is` 方法可以检验接受的请求 URL 与给定的规则是否匹配。使用此方法时你能将 `*` 作为通配符：

	if ($request->is('admin/*')) {
		//
	}

要得到完整的 URL，不仅仅是路径信息，你可以对请求实例使用 `url` 方法：

	$url = $request->url();

#### 获取请求的方法

`method` 方法将返回 HTTP 动作（方法）。你也可以使用 `isMethod` 方法来验证 HTTP 动作是否匹配给定的字符：

	$method = $request->method();

	if ($request->isMethod('post')) {
		//
	}

<a name="psr7-requests"></a>
### PSR-7 请求

PSR-7 标准指定了 HTTP 信息接口，包括请求和响应。如果你想得到一个 PSR-7 的实例对象，你首先需要安装几个库。 Laravel 使用 Symfony 的 HTTP Message Bridge 组件来
将原来的 Laravel 请求和响应转换为 PSR-7 兼容实现。

	composer require symfony/psr-http-message-bridge

	composer require zendframework/zend-diactoros

一旦你安装了这些库，你就可以在你的路由或者控制器中，通过简单的请求类型提示获取 PSR-7 请求：

	use Psr\Http\Message\ServerRequestInterface;

	Route::get('/', function (ServerRequestInterface $request) {
		//
	});

如果从路由或控制器返回一个 PSR-7 响应，它将自动被框架转换回 Laravel 的响应实例。

<a name="retrieving-input"></a>
## 获取输入数据

#### 获取输入值

使用几个简单的方法，你就能从 `Illuminate\Http\Request` 实例中获取所有的用户输入。你不需要考虑请求的 HTTP 动作，获取输入的方式都是相同的。

	$name = $request->input('name');

你可以传入一个默认值到 `input` 方法的第二个参数。如果请求的输入值不存在于该请求，默认值将被返回：

	$name = $request->input('name', 'Sally');

如果输入值是数组，你可以使用 `.` 来获取数组值：

	$input = $request->input('products.0.name');

#### 确认输入值是否存在

要判断数据是否存在于当前请求，你可以使用 `has` 方法。如果数据存在**同时**不是空字符串`has` 方法将返回 `true`：

	if ($request->has('name')) {
		//
	}

#### 获取所有输入数据

你可以使用 `all` 方法以 `数组` 的形式获取所有的输入值：

	$input = $request->all();

#### 获取部分输入数据

如果你想获取输入数据的一个子集，你可以使用 `only` 和 `except` 方法。这两种方法都接受一个 `数组` 作为唯一的参数：

	$input = $request->only('username', 'password');

	$input = $request->except('credit_card');

<a name="old-input"></a>
### 旧输入数据

Laravel 可以让你保留这次的输入数据直到下次请求前。这个特性对于表单验证非常有用。然而，如果你使用 Laravel 自带的[验证服务](/docs/{{version}}/validation)，你就不需要手动使用这些方法，因为 Laravel 的内置验证器会自动调用它们。

#### 将数据存为一次性 Session

`Illuminate\Http\Request` 实例的 `flash` 方法将当前的输入数据闪存到 [Session](/docs/{{version}}/session)中，所以到下次请求发送到应用程序它都是可用的：

	$request->flash();

你也可以使用 `flashOnly` 和 `flashExcept` 方法来将请求数据的子集闪存到 `Session`：

	$request->flashOnly('username', 'email');

	$request->flashExcept('password');

#### 快闪及重定向

你可能常常想要将输入资料快闪并重定向到前一页，你只要在重定向方法后串接 `withInput` 就行了：

	return redirect('form')->withInput();

	return redirect('form')->withInput($request->except('password'));

#### 获取旧输入数据

想要获取上次请求所快闪的输入数据，使用 `Request` 实例中的 `old` 方法。`old` 方法提供一个简单的方式从 [Session](/docs/{{version}}/session) 中取出被快闪的输入数据：

	$username = $request->old('username');

Laravel 也提供一个全局辅助方法 `old` 。如果你想在 [Blade template](/docs/{{version}}/blade) 中显示旧输入数据，使用 `old` 辅助方法是更方便的：

	{{ old('username') }}

<a name="cookies"></a>
### Cookies

#### 从请求获取 Cookie 值

Laravel 框架所创建的所有 Cookie 都会加密而且加上认证记号，意味着被客户端更改 Cookie 会失效。为了从请求中获取 Cookie ，你可以使用 `Illuminate\Http\Request` 实例的 `cookie` 方法：

	$value = $request->cookie('name');

#### 附加新的 Cookie 到响应

Laravel 提供全局辅助方法 `cookie` ，是通过简单工厂来产生新的 `Symfony\Component\HttpFoundation\Cookie` 实例。可以在 `Illuminate\Http\Request` 实例之后连接 `withCookie` 方法带入 Cookie 至响应：

	$response = new Illuminate\Http\Response('Hello World');

	$response->withCookie(cookie('name', 'value', $minutes));

	return $response;

要创建长久有效的，为期五年的 Cookie ，你可以先调用 `cookie` 辅助方法且不带任何参数，在使用 Cookie 工厂类的 `forever` 方法，接着将 `forever` 方法接在 `cookie` 之后：

	$response->withCookie(cookie()->forever('name', 'value'));

<a name="files"></a>
### 上传文件

#### 获取上传文件

你可以使用 `Illuminate\Http\Request` 实例中的 `file` 方法获取上传的文件。`file` 方法返回的对象是 `Symfony\Component\HttpFoundation\File\UploadedFile` 类的实例，该类继承了 PHP 的 `SplFileInfo` 类同时提供了许多与上传文件交互的方法：

	$file = $request->file('photo');

#### 确认文件是否上传

你可以使用 `hasFile` 方法来验证请求上的文件是否存在：

	if ($request->hasFile('photo')) {
		//
	}

#### 确认上传的文件是否有效

除了检查上传的文件是否存在外，你可以通过 `isValid` 方法来验证上传的文件是否有效：

	if ($request->file('photo')->isValid())
	{
		//
	}

#### 移动上传的文件

想要哟东上传的文件至新的位置，你必须使用 `move` 方法。该方法会从文件的临时上传位置（由你 PHP 的设置决定）将文件移动到你选择的永久目的位置：

	$request->file('photo')->move($destinationPath);

	$request->file('photo')->move($destinationPath, $fileName);

#### 其他上传文件的方法

`UploadedFile` 的实例还有许多可用的方法，可以至该对象的 [API 文档](http://api.symfony.com/2.7/Symfony/Component/HttpFoundation/File/UploadedFile.html)了解有关这些方法的详细信息。
