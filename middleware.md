# HTTP Middleware

- [简介](#introduction)
- [创建中间件](#defining-middleware)
- [注册中间件](#registering-middleware)
- [中间件参数](#middleware-parameters)
- [可终止中间件](#terminable-middleware)

<a name="introduction"></a>
## 简介

HTTP 中间件（Middleware）提供一个便利机制来过滤你应用的 HTTP 请求。例如，Laravel 默认包含了中间件来验证用户身份。如果用户没有通过验证，这个中间件会将用户重定向到登录页面，然而，如果用户已经过验证，中间件则会允许请求进一步前进到应用。

当然，中间件不仅仅是用来验证用户身份的。如 CORS 中间件负责给所有离开应用的响应添加一个正确的头。日志中间件负责记录所有进入你应用的请求。

Laravel 框架默认包含了几个中间件，包括维护，认证，CSRF 保护等，所有这些中间件都位于 `app/Http/Middleware` 目录。

<a name="defining-middleware"></a>
## 创建中间件

要创建一个中间件，可以使用 `make:middleware` 这个Artisan 命令:
 
    php artisan make:middleware OldMiddleware
    
这个命令将会在你的 `app/Http/Middleware` 目录中生成一个名称为 `OldMiddleware` 类。在这个中间件中，我们只允许参数 `age` 大于200的请求才能访问路由。否则，我们会将这个请求重定向到 “home” URI。
    
要创建一个中间件，可以使用 `make:middleware` Artisan 命令:

    php artisan make:middleware OldMiddleware

This command will place a new `OldMiddleware` class within your `app/Http/Middleware` directory. In this middleware, we will only allow access to the route if the supplied `age` is greater than 200. Otherwise, we will redirect the users back to the "home" URI.

    <?php

    namespace App\Http\Middleware;

    use Closure;

	class OldMiddleware
	{
    	/**
    	 * Run the request filter.
    	 *
     	 * @param  \Illuminate\Http\Request  $request
     	 * @param  \Closure  $next
     	 * @return mixed
     	 */
    	 public function handle($request, Closure $next)
    	 {
        	 if ($request->input('age') <= 200) {
             return redirect('home');
         }

         return $next($request);
    	}

 	}
 	
如你所见，如果 `age` 参数小于或者等于200，中间件将会返回 `HTTP` 请求的重定向给客户端，否则，这个请求将会进一步被传递进应用程序。通过调用带有 `$request` 参数的 `$next` 方法，就可以将请求传递到更深层的应用程序(允许通过中间件)。

在 `HTTP` 请求到达你的应用程序之前，最好是可以通过一系列的中间件层。每一层的中间件都可以检查请求甚至完全拒绝它。

### Before / After 中间件


一个中间件运行在一个请求之前还是在一个请求之后取决于这个中间件本身。举个例子，在应用程序处理这个请求之前，这个中间件完成一些前置的任务:

     <?php

	 namespace App\Http\Middleware;

	 use Closure;

	 class BeforeMiddleware
	 {
    	 public function handle($request, Closure $next)
    	 {
        	 // Perform action

        	 return $next($request);
    	 }
	 }
	 
在应用程序处理完一个请求之后，这个中间件完成一些后置任务:

    <?php

    namespace App\Http\Middleware;
<<<<<<< HEAD

    use Closure;
=======
>>>>>>> Laragirl-5.1

    use Closure;
    
	class AfterMiddleware
	{
    	public function handle($request, Closure $next)
    	{
        	$response = $next($request);

	        // Perform action

    	    return $response;
    	}
	}

<a name="registering-middleware"></a>
## 注册中间件

### 全局中间件

如果你想让一个中间件可以处理应用程序中的每一个请求，可以将这个中间件列在 `app/Http/Kernel.php` 中的 `middleware` 的属性内。

### 指派中间件给路由

如果你想给一个特定的路由指定中间件，需要先在 `app/Http/Kernel.php` 为中间件配置一个键值。默认情况下，这个类中的 `$routeMiddleware` 属性中列出了目前 Laravel 中配置的中间件。要增加你自己的配置，只需要在这个列表中增加你自己的键值就行，举个例子:

    // Within App\Http\Kernel Class...

	protected $routeMiddleware = [
    	'auth' => \App\Http\Middleware\Authenticate::class,
    	'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    	'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
	];
	
中间件一旦在 `HTTP kernel` 中定义，你可以在路由选项中通过 `middleware` 键来指派中间件:

	Route::get('admin/profile', ['middleware' => 'auth', function()
	{
    	//
	}]);


<a name="middleware-parameters"></a>
## 中间件参数

中间件也可以接收自定义的参数。举个例子，如果你的应用需要确认认证用户在进行某个操作之前已经拥有了 `role`，你可以创建一个接收 `role` 参数的 `RoleMiddleware` 中间件。

新增加的参数在 `$next` 参数后面:

    <?php

    namespace App\Http\Middleware;

    use Closure;

	class RoleMiddleware
	{
    	/**
     	* Run the request filter.
     	*
     	* @param  \Illuminate\Http\Request  $request
     	* @param  \Closure  $next
     	* @param  string  $role
     	* @return mixed
     	*/
    	public function handle($request, Closure $next, $role)
    	{
        	if (! $request->user()->hasRole($role)) {
            	// Redirect...
        	}

	        return $next($request);
    	}

	}
	
在定义路由的时候就需要指定中间件的参数，通过 `:` 将中间件与参数分隔开来。多个参数应该使用逗号来隔开:

	Route::put('post/{id}', ['middleware' => 'role:editor', function ($id) {
    //
	}]);


<a name="terminable-middleware"></a>
## 可终止中间件
有些情况下中间件需要在 `HTTP` 响应已经被发送到客户端之后才执行。例如 `Laravel` 的 `session` 中间件保存 `session` 数据是在响应被发送到浏览器之后才执行的。为了达到这一点，可以通过定义一个 `terminate` 方法来创建一个可终止的中间件:

	<?php

	namespace Illuminate\Session\Middleware;

    use Closure;
<<<<<<< HEAD


    class StartSession
    {
        public function handle($request, Closure $next)
        {
            return $next($request);
        }

        public function terminate($request, $response)
        {
            // Store the session data...
        }
    }
=======
>>>>>>> Laragirl-5.1



这个 `terminate` 方法应该同时接收请求和响应参数。一旦你定义了一个可终止的中间件，你应该把它加到 `HTTP kernel` 中全局中间件列表中。