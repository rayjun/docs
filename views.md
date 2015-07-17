# 视图

- [基本用法](#basic-usage)
	- [传递数据到视图](#passing-data-to-views)
	- [共享数据到全部视图](#sharing-data-with-all-views)
- [视图组件](#view-composers)

<a name="basic-usage"></a>
## 基本用法

视图包含你应用程序所提供的 HTML 代码，它能从你呈现的逻辑中分离你的控制器和应用程序逻辑。视图存放在 `resources/views` 文件夹下。

一个简单的视图看起来可能像这样：

	<!-- View stored in resources/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

当视图被存放在 `resources/views/greeting.php` ，我们能使用 `view` 全局辅助方法，如下：

	Route::get('/', function ()	{
		return view('greeting', ['name' => 'James']);
	});

如你所见，`view` 辅助法方法的第一个参数对应 `resources/views` 文件夹内试图文件的名称。传入到 `view` 辅助方法的第二个参数是一个能在视图内获取的数组。在这个例子中，我们传入 `name` 这个变量，然后在视图里简单的用 `echo` 来显示它。

当然，视图文件也可以被存放在 `resources/views` 的子目录中。`.` 可以被用来表示子目录内的视图文件。例如，如果你的视图存放在 `resources/views/admin/profile.php` ，你可以如下使用它：

	return view('admin.profile', $data);

#### 判断视图是否存在

如果你需要判断视图是否存在，你可以使用 `exists` 方法在一个没有参数的 `view` 辅助方法之后。当视图文件存在这个方法将返回 `true` ：

	if (view()->exists('emails.customer')) {
		//
	}

当 `view` 辅助方法不传入参数时，将返回一个 `Illuminate\Contracts\View\Factory` 的实例，以便你能使用这个 Factory 的任何方法。

<a name="view-data"></a>
### 视图数据

<a name="passing-data-to-views"></a>
#### 传递数据到视图

正如你之前看到的那个例子，你能简单地传入一个数组到视图：

	return view('greetings', ['name' => 'Victoria']);

当你用这种方法传递信息时，`$data` 必须是一个键值对数组。在视图内，你可以使用对应的键名来取值，如：`<?php echo $key; ?>` 。你也可以使用另一个可代替的方法来传递一个数组，在 `view` 辅助方法中使用 `with` 方法来给视图添加单独的一份数据。

	$view = view('greeting')->with('name', 'Victoria');

<a name="sharing-data-with-all-views"></a>
#### 共享数据到全部视图

有时你可能需要共享一份数据给你应用程序所表现的所有视图。你能通过使用视图 factory 的 `share` 方法。通常，你需要把 `share` 方法的代码写在一个服务提供者的 `boot` 方法中。你可以自由地在 `AppServiceProvider` 中添加或者自己写一个单独的服务提供者来存放它们：

	<?php

	namespace App\Providers;

	class AppServiceProvider extends ServiceProvider
	{
	    /**
	     * Bootstrap any application services.
	     *
	     * @return void
	     */
		public function boot()
		{
			view()->share('key', 'value');
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

<a name="view-composers"></a>
## 视图组件

视图组件就是在视图被渲染前被呼叫的闭包或类方法。若你想每次渲染视图时都绑定数据，一个视图组件可以把这些逻辑组织在一个地方。

让我们在一个[服务提供者](/docs/{{version}}/providers)内注册我们的视图组件。我们使用 `view` 辅助方法来获取底层的 `Illuminate\Contracts\View\Factory` 装置。记住，Laravel没有一个默认的目录来存放视图组件。你可以自由的组织它们。例如，你可以创建一个 `App\Http\ViewComposers` 目录：

	<?php

	namespace App\Providers;

	use Illuminate\Support\ServiceProvider;

	class ComposerServiceProvider extends ServiceProvider
	{
		/**
		 * Register bindings in the container.
		 *
		 * @return void
		 */
		public function boot()
		{
			// Using class based composers...
			view()->composer(
				'profile', 'App\Http\ViewComposers\ProfileComposer'
			);

			// Using Closure based composers...
			view()->composer('dashboard', function ($view) {

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

注意，如果你创建了一个新的服务提供者来包含你注册的视图组件，你需要添加这个服务提供者到 `config/app.php` 下的 `providers` 数组中。

现在我们以及注册了组件，每次 `profile` 视图渲染的时候，`ProfileComposer@compose` 方法将被执行。下面来我们来定义这个组件类：

	<?php

	namespace App\Http\ViewComposers;

	use Illuminate\Contracts\View\View;
	use Illuminate\Users\Repository as UserRepository;

	class ProfileComposer
	{
		/**
		 * The user repository implementation.
		 *
		 * @var UserRepository
		 */
		protected $users;

		/**
		 * Create a new profile composer.
		 *
		 * @param  UserRepository  $users
		 * @return void
		 */
		public function __construct(UserRepository $users)
		{
			// Dependencies automatically resolved by service container...
			$this->users = $users;
		}

		/**
		 * Bind data to the view.
		 *
		 * @param  View  $view
		 * @return void
		 */
		public function compose(View $view)
		{
			$view->with('count', $this->users->count());
		}
	}

在视图渲染前，组件的 `compose` 方法就会被调用，并且传入一个 `Illuminate\Contracts\View\View` 的实例。你可以使用 `with` 方法来绑定数据到视图。

> **备注：** 所有的视图组件都会被[服务提供者](/docs/{{version}}/container)所解析，所以你需要在视图组件的构造器类型中类型限制所有的

#### 对多个视图附加组件

你可以在 `composer` 方法的第一个参数传入一个视图数组，来一次对多个视图附加视图组件。

	view()->composer(
		['profile', 'dashboard'],
		'App\Http\ViewComposers\MyViewComposer'
	);

`view` 的 `composer` 方法可以接受 `*` 作为通配符，允许你附加组件到所有的视图：

	view()->composer('*', function ($view) {
		//
	});

### 视图创建者

视图 **创建者** 几乎和视图组件一样；只是视图创建者会在视图初始化之后立即执行。要注入一个创建者，可以使用 `creator` 方法：

	view()->creator('profile', 'App\Http\ViewCreators\ProfileCreator');
