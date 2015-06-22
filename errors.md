# Errors & Logging

- [介绍](#introduction)
- [配置](#configuration)
- [异常处理](#the-exception-handler)
	- [Report 方法](#report-method)
	- [Render 方法](#render-method)
- [HTTP 异常](#http-exceptions)
	- [自定义 HTTP 错误页面](#custom-http-error-pages)
- [日志](#logging)

<a name="introduction"></a>
## 介绍

当你开始一个新的 Laravel 项目时，错误和异常处理已经为你配置好了，除些之外，Laravel 还集成了[Monolog](https://github.com/Seldaek/monolog)日志工具库，提供各种强大的日志处理程序。

<a name="configuration"></a>
## 配置

#### 错误详情

你的应用程序通过浏览器显示的错误详情的量，是由 `config/app.php` 配置文件中的 `debug` 选项来控制的，默认情况下，这个配置项是遵循 `APP_DEBUG` 环境变量来设置的，这个变量存放在 `.env` 文件中。

对于本地开发，应该将 `APP_DEBUG` 环境变量设置为 `true`。在生产环境中，这个值应该总是为 `false`。

#### 日志模式

在框架之外，Laravel 支持 `single`, `daily`, `syslog` 和 `errorlog` 多种日志模式，例如，如果你希望使用按天的日志文件而不是单个文件，你只需要将 `config/app.php` 中的 `log` 配置为：

	'log' => 'daily'

#### 配置 Monolog 

如果你希望完全控制应用程序中 Monolog 的配置方式，你可以使用程序中 `configureMonologUsing` 方法，你应该在 `bootstrap/app.php` 文件中，返回 `$app` 变量之前，调用此方法：

	$app->configureMonologUsing(function($monolog) {
		$monolog->pushHandler(...);
	});

	return $app;

<a name="the-exception-handler"></a>
##  异常处理

所有异常都是由 `App\Exceptions\Handler` 类来处理的，这个类包含有两个方法：`report` 和 `render`，后面我们将详细查看这些方法。

<a name="report-method"></a>
### Report 方法

`report` 方法像[BugSnag](https://bugsnag.com)一样，用于记录或发送异常信息到外部的服务，默认情况下，`report` 仅将异常传入基本类中记录日志的地方，然而，你可以你喜欢的方式记录异常信息。

例如，如果你需要以不同的方式报告不同类型的异常，你可以使用 PHP `instanceof` 比较操作符：

	/**
	 * Report or log an exception.
	 *
	 * This is a great spot to send exceptions to Sentry, Bugsnag, etc.
	 *
	 * @param  \Exception  $e
	 * @return void
	 */
	public function report(Exception $e)
	{
		if ($e instanceof CustomException) {
			//
		}

		return parent::report($e);
	}

#### 根据类型忽略异常

异常处理程序的 `$dontReport` 属性包含一个异常类型数组，这些异常类型将不会被日志记录，默认情况下，由 404 错误产生的异常将不会被写入日志文件中，根据需要你可以添加其它异常类型到这个组中。

<a name="render-method"></a>
### Render 方法

`render` 方法负责将一个给定的异常转化为一个应该返回给浏览器的 HTTP 响应，默认情况下，异常会被传入一个基础类，这个类将生成一个响应，然而，你可以随意检查异常类型或者返回你自定义的响应：

    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $e
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $e)
    {
    	if ($e instanceof CustomException) {
    		return response()->view('errors.custom', [], 500);
    	}

        return parent::render($request, $e);
    }

<a name="http-exceptions"></a>
## HTTP 异常

一些异常描述来自服务器的 HTTP 错误码，例如，这个可以是⎡页面未找到⎦错误（404），“未经授权错误”（401）或者甚至程序员生成的 500 错误，为了在程序的任何地方生成这样的一个响应，请使用如下代码：

	abort(404);

`abort` 方法会立即引发一个异常，这个异常将由异常处理程序发出，可选地，你可以提供一个响应的文本：

	abort(403, 'Unauthorized action.');

这个方法可能会在请求的生命周期的任何时间被用到。

<a name="custom-http-error-pages"></a>
### 自定义 HTTP 错误页面

Laravel 很容易根据各种 HTTP 状态码返回自定义的错误页面，例如，如果你希望自定义 404 HTTP 状态返回码的错误页面，创建一个 `resources/views/errors/404.blade.php`，当任何时候应用程序生成 404 错误时返回此页面。

在这个目录下的所有视图，应该被命名为与相应的 HTTP 状态码相匹配的名字。

<a name="logging"></a>
## 日志

Laravel 日志工具提供一个简单的控制层，构建于强大的[Monolog](http://github.com/seldaek/monolog)库之上，默认情况下，Laravel 配置为按天生成日志文件，存放于 `storage/logs` 目录，你可以使用 `Log` [facade](/docs/{{version}}/facades)向日志中写入信息：

	<?php

	namespace App\Http\Controllers;

	use Log;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * Show the profile for the given user.
		 *
		 * @param  int  $id
		 * @return Response
		 */
		public function showProfile($id)
		{
			Log::info('Showing user profile for user: '.$id);

			return view('user.profile', ['user' => User::findOrFail($id)]);
		}
	}

[RFC 5424](http://tools.ietf.org/html/rfc5424)定义了日志程序提供的七层日志级别：**debug**, **info**, **notice**, **warning**, **error**, **critical**, and **alert**。

	Log::debug($error);
	Log::info($error);
	Log::notice($error);
	Log::warning($error);
	Log::error($error);
	Log::critical($error);
	Log::alert($error);

#### 上下文信息

上下文数据数组也可以被传入日志方法，这个上下文数据将被格式化并和日志消息一起被显示：

	Log::info('User failed to login.', ['id' => $user->id]);

#### 访问底层 Monolog 实例

Monolog 包含各种额外的，你可能在日志记录中会用到的处理程序，如果需要，你可以访问 Laravel 的底层 Monolog 实例：

	$monolog = Log::getMonolog();
