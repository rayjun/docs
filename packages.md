# 组件开发

- [介绍](#introduction)
- [服务容器](#service-providers)
- [路由](#routing)
- [资源](#resources)
	- [视图](#views)
	- [翻译](#translations)
	- [配置](#configuration)
- [公共资产](#public-assets)
- [Publishing File Groups](#publishing-file-groups)

<a name="introduction"></a>
## Introduction

组件开发是 Laravel 扩展功能的主要方式，从操作日期的 [Carbon](https://github.com/briannesbitt/Carbon)，到类似 [Behat](https://github.com/Behat/Behat) 这种完整的测试框架，组件可以是任何形式的。

当然，有许多不同类型的组件，有些组件是独立的，意味着它们可以跟任何框架一起使用，而不仅是 Laravel。Carbon 和 Behat 都是独立组件的例子，所有这些组件，只需要将其依赖加入你的 `composer.json` 文件就可以在 Laravel 中使用。

另一方面，其它一些组件是专门为了在 Laravel 中使用而开发的，这些组件可能包含路由，控制器，视图和配置，是专门为了增加 Laravel 程序而开发的，这份指南主要涵盖这一类组件的开发内容。

<a name="service-providers"></a>
## 服务提供者

[服务提供者](/docs/{{version}}/providers)是 Laravel 和组件的连接点，服务提供者负责将组件绑定到 Laravel 的[服务容器](/docs/{{version}}/container)中且告知 Laravel 如何加组件的资源，如视图，配置和本地化语言文件。

一个服务提供者继承于 `Illuminate\Support\ServiceProvider` 类且包含两个方法： `register` 和 `boot`，基础类 `ServiceProvider` 位于 `illuminate/support` Composer 组件中，你需要将此组件的依赖加入你自己的包依赖中。

要了解更多关于服务提供者的结构与用途，请查看[相关文档](/docs/{{version}}/providers)。

<a name="routing"></a>
## 路由

为组件定义路由，你只需在组件服务提供者的 `boot` 方法中 `require` 相关的路由配置文件即可，在路由配置文件中，你可以使用 `Route` facade，和在一般 Laravel 应用程序中一样来[注册路由](/docs/{{version}}/routing)。

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		if (! $this->app->routesAreCached()) {
			require __DIR__.'/../../routes.php';
		}
	}

<a name="resources"></a>
## 资源

<a name="views"></a>
### 视图

要将你的组件中的视图注册到 Laravel 中，你需要告诉 Laravel 视图的位置，你可以使用服务容器的 `loadViewsFrom` 方法来加载视图，这个方法接受两个参数：指向视图模板的路径和你的组件名，例如，如果你的包名是「courier」，将下面的代码添加到你的服务提供者中：
To register your package's [views](/docs/{{version}}/views) with Laravel, you need to tell Laravel where the views are located. You may do this using the service provider's `loadViewsFrom` method. The `loadViewsFrom` method accepts two arguments: the path to your view templates and your package's name. For example, if your package name is "courier", add the following to your service provider's `boot` method:

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
	}

Package views are referenced using a double-colon `package::view` syntax. So, you may load the `admin` view from the `courier` package like so:

	Route::get('admin', function () {
		return view('courier::admin');
	});

#### Overriding Package Views

When you use the `loadViewsFrom` method, Laravel actually registers **two** locations for your views: one in the application's `resources/views/vendor` directory and one in the directory you specify. So, using our `courier` example: when requesting a package view, Laravel will first check if a custom version of the view has been provided by the developer in `resources/views/vendor/courier`. Then, if the view has not been customized, Laravel will search the package view directory you specified in your call to `loadViewsFrom`. This makes it easy for end-users to customize / override your package's views.

#### Publishing Views

If you would like to make your views available for publishing to the application's `resources/views/vendor` directory, you may use the service provider's `publishes` method. The `publishes` method accepts an array of package view paths and their corresponding publish locations.

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

		$this->publishes([
			__DIR__.'/path/to/views' => base_path('resources/views/vendor/courier'),
		]);
	}

Now, when users of your package execute Laravel's `vendor:publish` Artisan command, your views package's will be copied to the specified location.

<a name="translations"></a>
### Translations

If your package contains [translation files](/docs/{{version}}/localization), you may use the `loadTranslationsFrom` method to inform Laravel how to load them. For example, if your package is named "courier", you should add the following to your service provider's `boot` method:

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
	}

Package translations are referenced using a double-colon `package::file.line` syntax. So, you may load the `courier` package's `welcome` line from the `messages` file like so:

	echo trans('courier::messages.welcome');

<a name="configuration"></a>
### Configuration

Typically, you will want to publish your package's configuration file to the application's own `config` directory. This will allow users of your package to easily override your default configuration options. To publish a configuration file, just use the `publishes` method from the `boot` method of your service provider:

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->publishes([
			__DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
		]);
	}

Now, when users of your package execute Laravel's `vendor:publish` command, your file will be copied to the specified location. Of course, once your configuration has been published, it can be accessed like any other configuration file:

	$value = config('courier.option');

#### Default Package Configuration

You may also choose to merge your own package configuration file with the application's copy. This allows your users to include only the options they actually want to override in the published copy of the configuration. To merge the configurations, use the `mergeConfigFrom` method within your service provider's `register` method:

	/**
	 * Register bindings in the container.
	 *
	 * @return void
	 */
	public function register()
	{
		$this->mergeConfigFrom(
			__DIR__.'/path/to/config/courier.php', 'courier'
		);
	}

<a name="public-assets"></a>
## Public Assets

Your packages may have assets such as JavaScript, CSS, and images. To publish these assets to the application's `public` directory, use the service provider's `publishes` method. In this example, we will also add a `public` asset group tag, which may be used to publish groups of related assets:

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->publishes([
			__DIR__.'/path/to/assets' => public_path('vendor/courier'),
		], 'public');
	}

Now, when your package's users execute the `vendor:publish` command, your assets will be copied to the specified location. Since you typically will need to overwrite the assets every time the package is updated, you may use the `--force` flag:

	php artisan vendor:publish --tag=public --force

If you would like to make sure your public assets are always up-to-date, you can add this command to the `post-update-cmd` list in your `composer.json` file.

<a name="publishing-file-groups"></a>
## Publishing File Groups

You may want to publish groups of package assets and resources separately. For instance, you might want your users to be able to publish your package's configuration files without being forced to publish your package's assets at the same time. You may do this by "tagging" them when calling the `publishes` method. For example, let's define two publish groups in the `boot` method of a package service provider:

	/**
	 * Perform post-registration booting of services.
	 *
	 * @return void
	 */
	public function boot()
	{
		$this->publishes([
			__DIR__.'/../config/package.php' => config_path('package.php')
		], 'config');

		$this->publishes([
			__DIR__.'/../database/migrations/' => database_path('/migrations')
		], 'migrations');
	}

Now your users may publish these groups separately by referencing their tag name when using the `vendor:publish` Artisan command:

	php artisan vendor:publish --provider="Vendor\Providers\PackageServiceProvider" --tag="config"
