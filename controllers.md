# HTTP 控制器

- [简介](#introduction)
- [基础控制器](#basic-controllers)
- [控制器中间件](#controller-middleware)
- [RESTful 资源控制器](#restful-resource-controllers)
	- [部分资源路由](#restful-partial-resource-routes)
	- [命名资源路由](#restful-naming-resource-routes)
	- [嵌套资源](#restful-nested-resources)
	- [附加资源控制器](#restful-supplementing-resource-controllers)
- [隐式控制器](#implicit-controllers)
- [依赖注入和控制器](#dependency-injection-and-controllers)
- [R路由缓存](#route-caching)

<a name="introduction"></a>
## 简介

除了在单一的 `routes.php` 中定义所有的请求处理逻辑之外，你可能希望使用控制器类来组织此行为。控制器可以将相关的 HTTP 请求处理逻辑组成一个类。控制器通常放在 `app/Http/Controllers` 此目录中。

<a name="basic-controllers"></a>
## 基础控制器

这里是一个基础控制类的例子，所有的 Laravel 控制器都应该继承默认 Laravel 基础控制器类，它包含在 Laravel 的预设安装中：

    <?php

    namespace App\Http\Controllers;

    use App\User;
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
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

我们可以通过如下方式引导路由至对应的控制器动作：

    Route::get('user/{id}', 'UserController@showProfile');

现在当一个请求匹配到特定的路由 URI ，`UserController` 类中的 `showProfile` 方法将被执行，当然该路由参数也会被传到方法中。

#### 控制器和命名空间

有一点非常重要，当定义控制器路由时，我们无需指明完整的控制器命名空间，只需定义「根」命名空间 `App\Http\Controllers` 之后的部分类名称。`RouteServiceProvider` 默认会在包含根控制器命名空间的路由群组中，加载 routes.php 文件。

若你要在 `App\Http\Controllers` 此目录内层中使用 PHP 命名空间来嵌套化或组织你的控制器，只需要使用相对于 `App\Http\Controllers` 根命名空间的特定类名即可。因此，若你的控制类全名为 `App\Http\Controllers\Photos\AdminControlle`，你可以这样注册一个路由：

    Route::get('foo', 'Photos\AdminController@method');

#### 命名控制器路由

像闭包路由一样，你可以指定控制器路由的名称。

    Route::get('foo', ['uses' => 'FooController@method', 'as' => 'name']);

一旦你为控制器路由指定了一个名称，就能简单地产生指向控制器行为的 URL 。要产生一个指向控制器行为的 URL ，可使用 `action` 辅助方法。同样的，我们只需要指定基础命名空间 `App\Http\Controllers` 之后的部分类名称即可：

    $url = action('FooController@method');

也可以使用 `route` 辅助方法，产生已经命名的控制器路由的 URL ：

    $url = route('name');

<a name="controller-middleware"></a>
## 控制器中间件

[中间件](/docs/{{version}}/middleware) 可在控制器路由中指定，例如：

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'UserController@showProfile'
    ]);

然而，在你的控制器的构造器中指定中间件更加方便。从你的控制器的构造器使用 `middleware` 方法，你能很容易的指定中间件给控制器。你可以限制中间件，仅将它提供给控制器类中的某些方法：

    class UserController extends Controller
    {
        /**
         * Instantiate a new UserController instance.
         *
         * @return void
         */
        public function __construct()
        {
            $this->middleware('auth');

            $this->middleware('log', ['only' => ['fooAction', 'barAction']]);

            $this->middleware('subscribed', ['except' => ['fooAction', 'barAction']]);
        }
    }

<a name="restful-resource-controllers"></a>
## RESTful 资源控制器

资源控制器能让你无痛建立和资源相关的 RESTful 控制器。例如，你可能希望创建一个控制器，可用来处理针对你的应用程序所保存「相片」的 HTTP 请求。使用 `make:controller` Artisan 命令，快速创建这样的控制器：

    php artisan make:controller PhotoController

这个 Artisan 命令将在 `app/Http/Controllers/PhotoController.php` 产生一个控制器文件，这个控制器对每个可用的资源操作都包括一个方法。

接下来，你可以注册一个指向该控制器的资源路由：

    Route::resource('photo', 'PhotoController');

此单一路由声明创建了多个路由，用来处理各式各样和「相片」资源相关的 RESTful 行为。同样地，产生的控制器已有各种和这些行为绑定的方法，包含用来通知你它们处理了那些 URI 及动词。

#### 由资源控制器处理的行为

动词      | 路径                  | 行为         | 路由名称
----------|-----------------------|--------------|---------------------
GET       | `/photo`              | index        | photo.index
GET       | `/photo/create`       | create       | photo.create
POST      | `/photo`              | store        | photo.store
GET       | `/photo/{photo}`      | show         | photo.show
GET       | `/photo/{photo}/edit` | edit         | photo.edit
PUT/PATCH | `/photo/{photo}`      | update       | photo.update
DELETE    | `/photo/{photo}`      | destroy      | photo.destroy

<a name="restful-partial-resource-routes"></a>
#### 部分资源路由

宣告资源路由时，你可以让此路由仅处理一部分行为：

    Route::resource('photo', 'PhotoController',
                    ['only' => ['index', 'show']]);

    Route::resource('photo', 'PhotoController',
                    ['except' => ['create', 'store', 'update', 'destroy']]);

<a name="restful-naming-resource-routes"></a>
#### 命名资源路由

所有的资源控制器行为都预设一个路由名称；不过你可以在选项中传递一个 `names` 数组来重写这些名称：

    Route::resource('photo', 'PhotoController',
                    ['names' => ['create' => 'photo.build']]);

<a name="restful-nested-resources"></a>
#### 嵌套资源

有时你可能需要对「嵌套」资源定义路由。例如，相片资源可能会附加多个「评论」。要「嵌套」资源控制器，可在路由宣告中使用「点」记号：

    Route::resource('photos.comments', 'PhotoCommentController');

这个路由将注册一个「嵌套化」资源，可通过下面这样的 URL 来存取它：`photos/{photos}/comments/{comments}`。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;

    class PhotoCommentController extends Controller
    {
        /**
         * Show the specified photo comment.
         *
         * @param  int  $photoId
         * @param  int  $commentId
         * @return Response
         */
        public function show($photoId, $commentId)
        {
            //
        }
    }

<a name="restful-supplementing-resource-controllers"></a>
#### 附加资源控制器

如果必须在默认的资源路由之外添加额外的路由，你需要在 `Route:resource` 之前定义这些路由。否则，由 `resource` 方法定义的路由可能会意外地覆盖你添加的路由。

    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

<a name="implicit-controllers"></a>
## 隐式控制器

Laravel 允许你轻易的定义单一路由来处理控制器类中的各种行为。首先，使用 `Route::controller` 方法来定义路由。`controller` 方法接受两个参数。第一个参数是控制器处理的基础URI，第二个是控制器的类别名称：

    Route::controller('users', 'UserController');

接下来，仅需要在控制器中加入你的方法。方法的名称应该由它们所响应的 HTTP 动词开头，紧跟着首字母大写的 URI 所组成：

    <?php

    namespace App\Http\Controllers;

    class UserController extends Controller
    {
        /**
         * Responds to requests to GET /users
         */
        public function getIndex()
        {
            //
        }

        /**
         * Responds to requests to GET /users/show/1
         */
        public function getShow($id)
        {
            //
        }

        /**
         * Responds to requests to GET /users/admin-profile
         */
        public function getAdminProfile()
        {
            //
        }

        /**
         * Responds to requests to POST /users/profile
         */
        public function postProfile()
        {
            //
        }
    }

正如你在以上例子看到的，`index` 方法会响应控制器所处理的根 URI，在这个例子中是 `users`。

#### 指定路由名称

如果你想要 [命名](/docs/{{version}}/routing#named-routes) 控制器中的某些路由，你可以在 `controller` 方法中传入一个名称数组作为第三个参数：

    Route::controller('users', 'UserController', [
        'getShow' => 'user.show',
    ]);

<a name="dependency-injection-and-controllers"></a>
## 依赖注入和控制器

#### 构造器注入

Laravel [服务容器](/docs/{{version}}/container) 用于解析所有的 Laravel 控制器。因此，你可以在控制器所需要的构造器中，对依赖做任何的类型限制。这些依赖会自动被解析并注入到控制器之中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;
    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * The user repository instance.
         */
        protected $users;

        /**
         * Create a new controller instance.
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

当然，你也可以对 [Laravel contract](/docs/{{version}}/contracts) 作类型限制。只要容器能解析它，你就能对它作类型限制。

#### 方法注入

除了构造器注入之外，你也可以对控制器方法的依赖做类型限制。例如，让我们对 `Illuminate\Http\Request` 实例中的一个方法做类型限制：

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

如果你的控制器方法也能从路由参数中获取预期的输入，只要你在其他的依赖之后列出你的路由参数即可。例如，你的路由如下定义：

    Route::put('user/{id}', 'UserController@update');

你更可以对 `Illuminate\Http\Request` 作类型限制，同时通过定义控制器方法获取你的路由参数 `id` ，如下：

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

<a name="route-caching"></a>
## 路由缓存

<<<<<<< HEAD
若您的应用只使用了控制器基础路由，你可利用 Laravel 的路由缓存。使用路由缓存，将大幅降低注册应用程序所有路由所需要的时间。某些情况下，路由注册甚至可以快上 100 倍。要产生路由缓存，只要执行 `route:cache` Artisan 命令：
=======
If your application is exclusively using controller based routes, you may take advantage of Laravel's route cache. Using the route cache will drastically decrease the amount of time it takes to register all of your application's routes. In some cases, your route registration may even be up to 100x faster! To generate a route cache, just execute the `route:cache` Artisan command:
>>>>>>> laravel/5.1

    php artisan route:cache

就是这样！你的缓存路由文件将会被用来代替 `app/Http/routes.php` 文件。记住，如果你添加了任何新的路由，你就必须产生一个路由缓存。因此在项目部署时，你可能会只希望执行 `route:cache` 命令。

要移除路由缓存而且不需要新的缓存，使用 `route:clear` 命令：

    php artisan route:clear
