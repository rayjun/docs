# Authentication

- [介绍](#introduction)
- [认证快速入门](#authentication-quickstart)
    - [路由](#included-routing)
    - [视图](#included-views)
    - [认证](#included-authenticating)
    - [检索通过认证的用户](#retrieving-the-authenticated-user)
    - [保护路由](#protecting-routes)
    - [Authentication Throttling](#authentication-throttling)
- [手动认证用户](#authenticating-users)
    - [记住用户](#remembering-users)
    - [其它认证方法](#other-authentication-methods)
- [HTTP 基本认证](#http-basic-authentication)
     - [无状态 HTTP 基本认证](#stateless-http-basic-authentication)
- [重置密码](#resetting-passwords)
    - [数据库注意事项](#resetting-database)
    - [路由](#resetting-routing)
    - [视图](#resetting-views)
    - [密码重置之后](#after-resetting-passwords)
- [社交网站账号登陆认证](#social-authentication)
- [添加自定义的认证驱动](#adding-custom-authentication-drivers)

<a name="introduction"></a>
## 介绍

Laravel 使得认证变得非常简单，事实上，几乎所有东西都是开箱即用的，认证的配置文件被放置在`config/auth.php`, 里面包含了一些带有注释说明的配置选项来调整认证服务。

### 数据库注意事项

默认情况下，Laravel 在你的 `app` 目录下包含了一个 `App\User` [Eloquent 模型](/docs/{{version}}/eloquent)。这个模型可以使用默认的 Eloquent 认证驱动。如果你的应用不使用 Eloquent，你可以使用 `database` 作为认证驱动，它使用 Laravel 查询构造器。

当创建 `App\User` 数据库模型时，确保 password 列至少 60 个字符的长度。

你还需要确保你的 `users`（或者相关）表包含一个长度为 100 个字符，允许为空（nullable）的 `remember_token` 字符串列，这一列在应用程序维护一些需要`记住我`的会话时存储 token 信息，在 migration 中可以用 `$table->rememberToken();` 来做到。

<a name="authentication-quickstart"></a>
## 认证快速入门

Laravel 自带两个认证控制器，它们被放置在 `App\Http\Controllers\Auth` 命名空间中，`AuthController` 处理新用户注册和认证，而 `PasswordController` 包含了帮助已经存在的用户重置他们已经忘记的密码的逻辑，这两个控制器都使用了 trait 来包含它们需要的方法（methods）。对于很多应用程序，你不需要对这些控制器作任何修改。

<a name="included-routing"></a>
### 路由

默认情况下，没有[路由](/docs/{{version}}/routing) 配置将请求指向这些认证控制器，你可以手动将它们添加到你的 `app/Http/routes.php` 文件中：

    // Authentication routes...
    Route::get('auth/login', 'Auth\AuthController@getLogin');
    Route::post('auth/login', 'Auth\AuthController@postLogin');
    Route::get('auth/logout', 'Auth\AuthController@getLogout');

    // Registration routes...
    Route::get('auth/register', 'Auth\AuthController@getRegister');
    Route::post('auth/register', 'Auth\AuthController@postRegister');

<a name="included-views"></a>
### 视图

尽管框架中包含了认证控制器，你需提供 [视图](/docs/{{version}}/views) 来显示这些控制器的结果，这些视图应该放到 `resources/views/auth` 目录中。你可以凭自己的喜好随意更改这些视图，登陆视图应该放置于 `resources/views/auth/login.blade.php` 中， 注册视图在 `resources/views/auth/register.blade.php` 中。

#### 认证表单样例

    <!-- resources/views/auth/login.blade.php -->

    <form method="POST" action="/auth/login">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password" id="password">
        </div>

        <div>
            <input type="checkbox" name="remember"> Remember Me
        </div>

        <div>
            <button type="submit">Login</button>
        </div>
    </form>

#### 注册表单样例

    <!-- resources/views/auth/register.blade.php -->

    <form method="POST" action="/auth/register">
        {!! csrf_field() !!}

        <div class="col-md-6">
            Name
            <input type="text" name="name" value="{{ old('name') }}">
        </div>

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            Password
            <input type="password" name="password">
        </div>

        <div class="col-md-6">
            Confirm Password
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">Register</button>
        </div>
    </form>

<a name="included-authenticating"></a>
### 认证

由于你已经为这些自带的认证控制器搭建了路由和视图，你可以为你的应用程序注册和认证新用户了，你可以只在浏览器中访问已经定义的路由，认证控制器已经包含了认证现在用户和存储新用户到数据的逻辑（通过它们的 traits）。

当一个用被成功认证，会被重定向到`/home`，为此你需要注册一个路由来处理这个重定向请求，你可以通过在 `AuthController`中定义 `redirectTo` 属性，来自定义认证后的重定向位置：

    protected $redirectPath = '/dashboard';

When a user is not successfully authenticated, they will be redirected to the `/auth/login` URI. You can customize the failed post-authentication redirect location by defining a `loginPath` property on the `AuthController`:

    protected $loginPath = '/login';

#### 自定义

要修改当一个新用户注册时所需的表单字段，或者自定义新用户记录被插入数据库的方式，你可以修改 `AuthController` 控制器类，应用程序中的这个类负责验证和创建新用户。

`AuthController` 类中的 `validator` 方法包含了对程序中新用户的验证规则，你可以自由修改这个方法。

`AuthController` 中的 `create` 方法使用[Eloquent ORM](/docs/{{version}}/eloquent) 负责在数据中生成新 `App\User` 记录。

<a name="retrieving-the-authenticated-user"></a>
### 检索通过认证的用户

你可以能过 `Auth` facade 来访问已经通过认证的用户：

    $user = Auth::user();

或者，当一个用户被认证，你可以通过一个 `Illuminate\Http\Request` 实例来访问这个已经被认证的用户：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Routing\Controller;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function updateProfile(Request $request)
        {
            if ($request->user()) {
                // $request->user() returns an instance of the authenticated user...
            }
        }
    }

#### 确认当前用户是否通过认证

如果用户已经登陆到你的应用程序中，你可以使用 `Auth` 中的 `check` 方法，如果用户已经通过认证方法会返回 `true`：

    if (Auth::check()) {
        // The user is logged in...
    }

然而，你可以在一个用户访问特定路由/控制器之前，使用 middleware 检查该用户是否通过验证，了解更多关于这个，查看[保护路由](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### 保护路由

[保护路由](/docs/{{version}}/middleware) 可以被用来只允许通过验证的用户来访问某个给定的路由。can be used to allow only authenticated users to access a given route. Laravel 自带的 `auth` middleware, 定义在 `app\Http\Middleware\Authenticate.php` 中. 你需要做的只是将这个 middleware 添加加到你的路由定义当中:

    // Using A Route Closure...

    Route::get('profile', ['middleware' => 'auth', function() {
        // Only authenticated users may enter...
    }]);

    // Using A Controller...

    Route::get('profile', [
        'middleware' => 'auth',
        'uses' => 'ProfileController@show'
    ]);

当然，如果你使用[控制器类](/docs/{{version}}/controllers)，你可以在控制器的构造函数中调用 `middleware` 方法，而需要将其直接加到路由的定义中：

    public function __construct()
    {
        $this->middleware('auth');
    }

<a name="authentication-throttling"></a>
### Authentication Throttling

If you are using Laravel's built-in `AuthController` class, the `Illuminate\Foundation\Auth\ThrottlesLogins` trait may be used to throttle login attempts to your application. By default, the user will not be able to login for one minute if they fail to provide the correct credentials after several attempts. The throttling is unique to the user's username / e-mail address and their IP address:

    <?php

    namespace App\Http\Controllers\Auth;

    use App\User;
    use Validator;
    use App\Http\Controllers\Controller;
    use Illuminate\Foundation\Auth\ThrottlesLogins;
    use Illuminate\Foundation\Auth\AuthenticatesAndRegistersUsers;

    class AuthController extends Controller
    {
        use AuthenticatesAndRegistersUsers, ThrottlesLogins;

        // Rest of AuthController class...
    }

<a name="authenticating-users"></a>
## 手动认证用户

当然，使用 Laravel 中自带的认证控制器不是必须的，如果你可以选择去掉这些控制器，你需要直接使用 Laravel 的认证类来管理用户认证。不过担心，这只是小事一桩！

我们将通过 `Auth` [facade](/docs/{{version}}/facades) 来访问 Laravel 的认证服务，所以我们需要确保在类的顶部将 `Auth` facade 导入。接下来，让我们来看一下 `attempt` 方法：

    <?php

    namespace App\Http\Controllers;

    use Auth;
    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

这个 `attempt` 方法接收一个键值对的数组作为第一个参数，数组中的值将会被用来在数据库的表中寻找这个用户，所以，在上面的例子中，这个用户会根据 `email`列的值被检索出来，如果用户被找到，数据库中存储的哈希化的密码将会与通过数组传入方法中密码被哈希化后的值进行比较，如果两个哈希化的密码匹配，将会为这个用户开启一个被认证的会话。

如果认证成功，这个 `attempt` 方法将返回 `true`，否则， 运回 `false`。

redirector 上的 `intended` 方法会在被认证过滤器拦截之前将用户重定向到他想要访问的 URL, 当URL 不可用时，一个回退 URL 参数可能被给到这个方法，防止想访问的地址不可用。

如果你愿意，除了用户的 e-mail 和 password 之外，你也可以向身份认证查询中添加额外的查询条件。例如，我们可能只验证标记为 “活跃（active）”的用户：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

将用户从程序中登出，你可以使用 `Auth` 上的 `logout` 方法，这个方法会将用户会话中的认证信息清除：

    Auth::logout();

> **注意:** 在这些例子中, 并不一定要使用 `email` 字段, 这只是作为示例。你可以使用数据库中的任何相当于”username" 的列。

<a name="remembering-users"></a>
## 记住用户

如果你想在应用程序中提供“记住我”的功能，你可以传入一个布尔值作为 attempt 方法的第二个参数，这样就可以一直保留用户的认证信息，直到用户手动登出为止。当然，你的 `users` 表必须包含 `remember_token` 字符串列，来存储“记住我”的 token 值。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

如果要“记住”用户，你可以使用 `viaRemember` 方法，通过 "记住我"的 cookie 来判断用户是否已经被认证：


    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### 其它认证方法

#### 单用户实例认证

如果你需要将一个已经存在的用户实例登入到应用程序中，你可以调用`login` 方法传入这个用户实例，传入的对象必须是 `Illuminate\Contracts\Auth\AuthenticatableContract` [contract](/docs/{{version}}/contracts)的一个实现类，当然，Laravel 中自带的 `App\User` 类已经实现了这个接口。

    Auth::login($user);

#### 用户 ID 认证

通过用户 ID 将一个用户登陆到应用程序中，你可以使用 `loginUsingId` 方法，这个方法仅接收你想要验证的用户的主键作为参数：

    Auth::loginUsingId(1);

#### 单次用户认证

你可以使用 `once` 方法对单个请求将一个用户登陆到应用程序中，不会使用 sessions 或者 cookies, 这种方式在构建无状态的 API 时很有帮助， `once` 方法与 `attempt` 具有相同的方法签名：

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP 基本认证

[HTTP 基本认证](http://en.wikipedia.org/wiki/Basic_access_authentication) 证提供了一个快速的方式来认证用户而不用特定设置一个「登入」页，将 `auth.basic`[middleware](/docs/{{version}}/middleware) 附加到你的路由上就可以启动了。 `auth.basic` 中间件包含在 Laravel 框架中，所以你不需要去定义它：

    Route::get('profile', ['middleware' => 'auth.basic', function() {
        // Only authenticated users may enter...
    }]);

一旦中间件被附加到了路由上，当你在浏览器中访问路由时将会自动提示你输入凭证。默认情况下， `auth.basic` 中间件将会使用用户记录上 `email` 字段作为 "username"。

#### 一个关于 FastCGI 的备注

如果你正在使 PHP FastCGI，直接使用 HTTP 基本认证可能无法正常工作，应该在 `.htaccess` 文件中添加以下几行：

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 无状态 HTTP 基本认证

你也可以使用 HTTP 基本认证而不在会话中设置一个用户标识符，这对于 API 认证特别有用。若要这样做, 请[定义中间件](/docs/{{version}}/middleware)调用 `onceBasic` 方法，请求可以被进一步传递到应用程序中：

    <?php

    namespace Illuminate\Auth\Middleware;

    use Auth;
    use Closure;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

下一步，[注册路由中间件](/docs/{{version}}/middleware#registering-middleware) 并将其附加到路由中：

    Route::get('api/user', ['middleware' => 'auth.basic.once', function() {
        // Only authenticated users may enter...
    }]);

<a name="resetting-passwords"></a>
## 重置密码

<a name="resetting-database"></a>
### 数据库注意事项

大多数的 web 应用程序都为用户提供方法重置忘记的密码，Laravel 提供方便的方法来发送密码提醒和执行密码重置，而不是迫使你你在每个应用程序中重新实现这个。

若要开始，请验证你的 `App\User` 模型实现了 `Illuminate\Contracts\Auth\CanResetPassword` contract。当然， 框架中已经包含的 `App\User` 模型已经实现了这个 contract，并使用了包含所需的方法的 `Illuminate\Auth\Passwords\CanResetPassword` trait 来实现该接口。


#### 生成重置 Token 数据表迁移

接下来，必须创建表来存储密码重置标记，此表的迁移(migration)已经被包含在了 Laravel 中开箱即用的，并且被放置于`database/migrations` 目录中，所以你需要执行迁移：

    php artisan migrate

<a name="resetting-routing"></a>
### 路由

Laravel 自带 `Auth\PasswordController`，其中包含重置用户密码必要的逻辑。你需要定义路由请求指向此控制器：

    // Password reset link request routes...
    Route::get('password/email', 'Auth\PasswordController@getEmail');
    Route::post('password/email', 'Auth\PasswordController@postEmail');

    // Password reset routes...
    Route::get('password/reset/{token}', 'Auth\PasswordController@getReset');
    Route::post('password/reset', 'Auth\PasswordController@postReset');

<a name="resetting-views"></a>
### 视图

除了为 `PasswordController` 定义路由外，你需要提供可以经由此控制器返回的视图。别担心，我们将提供示例来帮助你入门使用。当然，你可以自由地根据你的意愿定义表单。

#### 密码重置链接请求表单示例

你将需要为密码重置请求表单提供一个 HTML 视图，这个视图应用放置在 `resources/views/auth/password.blade.php`。这个表单提供单个输入框给用户填写邮件地址，允许他们请求获取一个重置链接：

    <!-- resources/views/auth/password.blade.php -->

    <form method="POST" action="/password/email">
        {!! csrf_field() !!}

        <div>
            Email
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <button type="submit">
                Send Password Reset Link
            </button>
        </div>
    </form>

当用户提交了重置密码请求，他们将收到一封电子邮件，包含一个指向 `PasswordController` 的 `getReset` 方法（通常在 `/password/reset` 中路由）的链接。你将需要在 `resources/views/emails/password.blade.php` 创建邮件的视图。这个视图将接收 `$token` 变量，用来在重置密码的请求中配置相应的用户。这是一个邮件视图的例子：

    <!-- resources/views/emails/password.blade.php -->

    Click here to reset your password: {{ url('password/reset/'.$token) }}

#### 密码重置表单示例

当用户点击邮件中的链接重置密码时，将会看到一个密码重置表单，这个视图应该被放置于 `resources/views/auth/reset.blade.php`。

这里是一个密码重置表单的示例：

    <!-- resources/views/auth/reset.blade.php -->

    <form method="POST" action="/password/reset">
        {!! csrf_field() !!}
        <input type="hidden" name="token" value="{{ $token }}">

        <div>
            <input type="email" name="email" value="{{ old('email') }}">
        </div>

        <div>
            <input type="password" name="password">
        </div>

        <div>
            <input type="password" name="password_confirmation">
        </div>

        <div>
            <button type="submit">
                Reset Password
            </button>
        </div>
    </form>

<a name="after-resetting-passwords"></a>
### 密码重置之后

一旦你已经定义了路由与视图重置用户的密码，你浏览器中就可以访问这些路由。框架自带的 `PasswordController` 已经包含了发送密码重置链接邮件以及更新数据库中密码的逻辑

密码重置之后，用户将会被自动登陆到应用程序中并且重定向到 `/home`。通过在 `PasswordController` 中定义 `redirectTo` 属性，你可以自定义密码重置重定向的位置：

    protected $redirectTo = '/dashboard';

> **注意:** 默认情况下，密码重置标志将会在一个小时后过期，你可通过更改你的 `config/auth.php` 文件中 `reminder.expire` 参数来修改这个过期时间。

<a name="social-authentication"></a>
## 社交网站账号登陆认证

除了典型的，基于表单认证，Laravel 也提供了一个使用[Laravel Socialite](https://github.com/laravel/socialite)来认证每三方的 OAuth，Socialite 目前支持 Facebook, Twitter, LinkedIn, Google, GitHub 和 Bitbucket 认证。

使用 Socialite，将组件依赖加入到 `composer.json` 文件中：

    composer require laravel/socialite

### 配置

安装了 Socialite 组件库之后，将 `Laravel\Socialite\SocialiteServiceProvider` 注册到 `config/app.php` 配置文件中：

    'providers' => [
        // Other service providers...

        Laravel\Socialite\SocialiteServiceProvider::class,
    ],

并且，将 `Socialite` 的facade添加到 `app` 配置文件是 `aliases` 数组中：

    'Socialite' => Laravel\Socialite\Facades\Socialite::class,

你还需要给你的应用程序使用的 OAuth 服务添加凭证信息。这些凭证信息应该放在 `config/services.php` 配置文件中，并且需要根据你的应用程序的需要，使用 `facebook`, `twitter`, `linkedin`, `google`, or `github` 作为相应的 key, 例如：

    'github' => [
        'client_id' => 'your-github-app-id',
        'client_secret' => 'your-github-app-secret',
        'redirect' => 'http://your-callback-url',
    ],

### Basic Usage

### 基本用法

接下来，你准备要认证用户了！你需要两个路由：一个用于将用户重定向到 OAuth 提供方，另一个用于身份验证之后从提供方接收回调，我们将使用 `Socialite` [facade](/docs/{{version}}/facades)来访问 Socialite：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Routing\Controller;

    class AuthController extends Controller
    {
        /**
         * Redirect the user to the GitHub authentication page.
         *
         * @return Response
         */
        public function redirectToProvider()
        {
            return Socialite::driver('github')->redirect();
        }

        /**
         * Obtain the user information from GitHub.
         *
         * @return Response
         */
        public function handleProviderCallback()
        {
            $user = Socialite::driver('github')->user();

            // $user->token;
        }
    }

`redirect` 负责发送用户到 OAuth 提供方，而 `user` 方法会读取进入的请求并从提供方获取用户的信息。在发送用户之前，你还可以在使用 `scope` 方法在请求上设置 「范围」:

    return Socialite::driver('github')
                ->scopes(['scope1', 'scope2'])->redirect();

Of course, you will need to define routes to your controller methods:

    <?php

        Route::get('auth/github', 'Auth\AuthController@redirectToProvider');
        Route::get('auth/github/callback', 'Auth\AuthController@handleProviderCallback');


#### 获取用户详细信息

一旦你得到了一个用户实例，你可以获得一些更多关于用户的信息：

    $user = Socialite::driver('github')->user();

    // OAuth Two Providers
    $token = $user->token;

    // OAuth One Providers
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All Providers
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();

<a name="adding-custom-authentication-drivers"></a>
## 添加自定义的认证驱动

如果你不使用传统的关系型数据库来存储你的用户，你需要用自己的认证驱动程序来扩展 Laravel，我们将使用 `Auth` 上 `extend` 方法来定义一个驱动程序，你应该将 `extend` 的调用放在一个[服务提供程者](/docs/{{version}}/providers)中：

    <?php

    namespace App\Providers;

    use Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Support\ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Perform post-registration booting of services.
         *
         * @return void
         */
        public function boot()
        {
            Auth::extend('riak', function($app) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...
                return new RiakUserProvider($app['riak.connection']);
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

你使用 `extend` 方法注册了驱动程序之后，你可以在 `config/auth.php` 配置文件中切换到新的驱动程序。

### User Provider 接口
 
`Illuminate\Contracts\Auth\UserProvider` 接口的实现类仅负责从持久存储系统中获取一个 `Illuminate\Contracts\Auth\Authenticatable` 的实现类，例如 MySQL, Riak, 等等。这两个接口使得 Laravel 认证机制可以继续工作而不用管用户的数据是如何存储或者什么类型的类用来表示它。

让我们来看一下 `Illuminate\Contracts\Auth\UserProvider` 接口：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

`retrieveById` 方法通常接收一个 key 表示代表用户，例如一个 MySQL 数据库中的自增 ID， `Authenticatable` 的实现类通过这个方法检索匹配的 ID　并返回。

`retrieveByToken` 方法通过它们唯一的 `$identifier` and "记住我" `$token` 检索用户，这个值存储在数据库的 `remember_token` 字段中。如同前一个方法，返回 `Authenticatable` 实现类应该被返回。

`updateRememberToken` 方法用一个新的 `$token`，更新 `$user` 中 `remember_token` 字段。这个新的标记(token)可以是一个全新的标记，由 "记住我" 的登陆的 attempt 成功方法返回赋值的，或者是一个 null 值，当用户登出时。

当尝试登陆到应用程序时，`retrieveByCredentials` 方法接收从 `Auth::attempt` 传入的一个用户凭证数组，此方法之后应该从底层的持久化存储中"查询"与那些凭证相匹配的用户。通常，这个方法会在 `$credentials['username']` 上执行一个带 where 条件的查询。这个方法然后应该返回一个 `UserInterface` 接口的实现，**这个方法不应该尝试执行任何密码校验和认证**。

`validateCredentials` 方法应该比较给定的 `$user` 与 `$credentials` 对用户进行认证，例如，这个方法可能比较 `$user->getAuthPassword()` 字符串与 `$credentials['password']` 的 `Hash::make` 值。此方法只应验证用户的凭据并返回布尔值。

### Authenticatable 接口

既然我们已经看过 `UserProvider` 的每一个方法，让我们一起来看一下 `Authenticatable`，记住，提供者程序应该从 `retrieveById` 和 `retrieveByCredentials` 方法返回这个接口的实现：:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

此接口很简单，'GetAuthIdentifier' 方法应返回用户的"主键"。在 MySQL 的后端，再次，这可能是自增的主键。'GetAuthPassword' 应返回用户的哈希化的密码。此接口允许认证系统可以作用到任何用户类上，不管穿上类是 ORM 还是你使用的存储抽象层。默认情况下，Laravel 的 'app' 目录包含一个用户类已经实现了这个接口，你可以参考这个类作为实现的示例。
