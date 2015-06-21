# Laravel Elixir

- [介绍](#introduction)
- [安装设置](#installation)
- [运行 Elixir](#running-elixir)
- [操作样式表](#working-with-stylesheets)
	- [Less](#less)
	- [Sass](#sass)
	- [纯 CSS 文件](#plain-css)
	- [Source Maps](#css-source-maps)
- [操作脚本](#working-with-scripts)
	- [CoffeeScript](#coffeescript)
	- [Browserify](#browserify)
	- [Babel](#babel)
	- [Scripts](#javascript)
- [版本控制 / Cache 破坏](#versioning-and-cache-busting)
- [调用已有的 Gulp 任务](#calling-existing-gulp-tasks)
- [编写 Elixir 扩展](#writing-elixir-extensions)

<a name="introduction"></a>
## 介绍

Laravel Elixir 提供了简洁、流畅的 API 为你的 Laravel 应用程序定义基本的[Gulp](http://gulpjs.com) 任务，Elixir 支持一些常见的 CSS 和 JavaScript 预处理器，甚至测试工具，使用方法链，Elixir 能让你流畅地定义 asset pipeline，例如：

```javascript
elixir(function(mix) {
	mix.sass('app.scss')
	   .coffee('app.coffee');
});
```

如果你曾对如何使用 Gulp 和 asset 编译感到迷惑，你会爱上 Lavavel Elixir，然而，没有要求你在开发应用程序时一定要使用它，你可以使用任何 asset pipeline 工具，或甚至什么都不用。

<a name="installation"></a>
## 安装设置

### 安装节点

在触发 Elixir 之前，你必须首先确保你的机器上安装了 Node.js。

    node -v

默认情况下，Laravel Homestead 包括所有你需要的，然而，如果你不使用 Vagrant，你可以通过访问[Node 下载页](http://nodejs.org/download/)可以很容易地安装 Node。

### Gulp

接下来，你需要下载[Gulp](http://gulpjs.com)为一个全局的 NPM 包：

    npm install --global gulp

### Laravel Elixir

最后一步是安装 Elixir，在新创建的 Laravel 程序中，你可以在根目录下找到 `package.json` 文件，把它想像成 `composer.json` 文件，只不过它定义的是 Node 的依赖关系而不是 PHP 的，你可以通过运行以下命令安装所有依赖：

	npm install

<a name="running-elixir"></a>
## 运行 Elixir

Elixir 创建于[Gulp](http://gulpjs.com)之上，所以运行 Elixir 任务你只需要在命令终端运行 [Gulp](http://gulpjs.com)，在命令后面加上 `--production` 标识表示压缩 CSS 和 Javascript 文件：

	// Run all tasks...
	gulp

	// Run all tasks and minify all CSS and JavaScript...
	gulp --production

#### 监听 Assets 修改

因为每次对 assets 修改后都要在终端运行 `gulp` 命令不方便，你可以使用 `gulp watch`，这个命令将一起在终端运行并且监听 assets 的任何修改。当修改发生时，将自动编译生成新的文件：

	gulp watch

<a name="working-with-stylesheets"></a>
## 操作样式表

在项目根目录下的 `gulpfile.js` 文件包含所有 Elixir 任务，Elixir 任务可以串连起来定义 assets 应该被编译。

<a name="less"></a>
### Less

将[Less](http://lesscss.org/)编译成 CSS，你可以使用 `less` 方法，`less` 方法假定你的 Less 文件存放在 `resources/assets/less` 中，默认情况下，编译生成的 CSS 存放在 `public/css/app.css` 中：

```javascript
elixir(function(mix) {
	mix.less("app.less");
});
```

你还可以合并多个 Less 文件成一个单独的 CSS 文件，再次，生成的 CSS 将被存放于 `public/css/app.css` 中，如果你希望自定义 CSS 编译的输出路径，你可以向 `less` 传入第二个参数指定：

```javascript
elixir(function(mix) {
	mix.less([
		"app.less",
		"controllers.less"
	], "public/assets/css");
});
```

<a name="sass"></a>
### Sass

`sass` 方法允许你将[Sass](http://sass-lang.com/)编译生成 CSS，假设你的 Sass 文件存放在 `resources/assets/sass` 中，你可以像这样使用这个方法：

```javascript
elixir(function(mix) {
	mix.sass("app.scss");
});
```

同样，像 `less` 方法一样，你可以将多个脚本文件编译生成单个 CSS 文件，甚至还可以自定议结果 CSS 文件的输出目录：

```javascript
elixir(function(mix) {
	mix.sass([
		"app.scss",
		"controllers.scss"
	], "public/assets/css");
});
```

#### Ruby Sass

在底层，Elixir 使用 LibSass 库编译，在某些情况下，依靠 Ruby 版本可能比较方便，尽管较慢，但是功能更加丰富。假设你已经安装了 Ruby 和 Sass gem(`gem install sass`)，你可以像这样使用 Ruby 编译器：

```javascript
elixir(function(mix) {
	mix.rubySass("app.scss");
});
```

<a name="plain-css"></a>
### 纯 CSS 文件

如果你只是想将一些纯 CSS　样式文件合并成一个单独的文件，你可以使用 `styles` 方法，传入些方法的文件路径都是相对于 `resources/assets/css`　的路径，且结果 CSS 将存放于 `public/css/all.css`：

```javascript
elixir(function(mix) {
	mix.styles([
		"normalize.css",
		"main.css"
	]);
});
```

当然，你还可以生成结果文件到一个自定义的路径，通过向 `styles` 方法传入第二个参数：

```javascript
elixir(function(mix) {
	mix.styles([
		"normalize.css",
		"main.css"
	], "public/assets/css");
});
```

<a name="css-source-maps"></a>
### Source Maps

Source maps 功能是开箱即用的，所以，对于每个编译生成文件，在同目录下都找到一个相匹配的 `*.css.map` 文件，这个匹配关系允许你在浏览器中调试过程中，追踪编译后样式表选择器到原始的 Sass 或者 Less 文件。

如果你不想为 CSS 文件生成 source maps，你可以使用简单的配置选项将其屏蔽：

```javascript
elixir.config.sourcemaps = false;

elixir(function(mix) {
	mix.sass("app.scss");
});
```

<a name="working-with-scripts"></a>
## 操作 Scripts

Elixir 还提供了一些方法来帮助你操作脚本文件，例如编译 ECMAScript 6，编译 CoffeeScript, Browserify, minification 和仅仅连结纯脚文件。

<a name="coffeescript"></a>
### CoffeeScript

`coffee` 方法被用于将[CoffeeScript](http://coffeescript.org/)编译成纯 JavaScript，`coffee` 方法接受一个 CoffeeScript 文件数组，文件路径相对于 `resources/assets/coffee` 目录且将生成单个的 `app.js` 文件到 `public/js` 目录中：

```javascript
elixir(function(mix) {
	mix.coffee(['app.coffee', 'controllers.coffee']);
});
```

<a name="browserify"></a>
### Browserify

Elixir 还包含 `browserify` 方法，将加载模块的所有特点都在浏览器中发挥出来，并且它使用的是 EcmaScript 6。
Elixir also ships with a `browserify` method, which gives you all the benefits of  requiring modules in the browser and using .

This task assumes that your scripts are stored in `resources/assets/js` and will place the resulting file in `public/js/bundle.js`:

```javascript
elixir(function(mix) {
	mix.browserify('index.js');
});
```

<a name="babel"></a>
### Babel

The `babel` method may be used to compile [EcmaScript 6 and 7](https://babeljs.io/docs/learn-es2015/) into plain JavaScript. This function accepts an array of files relative to the `resources/assets/js` directory, and generates a single `all.js` file in the `public/js` directory:

```javascript
elixir(function(mix) {
	mix.babel([
                "order.js",
                "product.js"
        ]);
});
```

To choose a different output location, simply specify your desired path as the second argument. The signature and functionality of this method are identical to `mix.scripts()`, excluding the Babel compilation.


<a name="javascript"></a>
### Scripts

If you have multiple JavaScript files that you would like to combine into a single file, you may use the `scripts` method.

The `scripts` method assumes all paths are relative to the `resources/assets/js` directory, and will place the resulting JavaScript in `public/js/all.js` by default:

```javascript
elixir(function(mix) {
	mix.scripts([
		"jquery.js",
		"app.js"
	]);
});
```

If you need to combine multiple sets of scripts into different files, you may make multiple calls to the `scripts` method. The second argument given to the method determines the resulting file name for each concatenation:

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

If you need to combine all of the scripts in a given directory, you may use the `scriptsIn` method. The resulting JavaScript will be placed in `public/js/all.js`:

```javascript
elixir(function(mix) {
	mix.scriptsIn("public/js/some/directory");
});
```

<a name="versioning-and-cache-busting"></a>
## Versioning / Cache Busting

Many developers suffix their compiled assets with a timestamp or unique token to force browsers to load the fresh assets instead of serving stale copies of the code. Elixir can handle this for you using the `version` method.

The `version` method accepts a file name relative to the `public` directory, and will append a unique hash to the filename, allowing for cache-busting. For example, the generated file name will look something like: `all-16d570a7.css`:

```javascript
elixir(function(mix) {
	mix.version("css/all.css");
});
```

After generating the versioned file, you may use Laravel's global `elixir` PHP helper function within your [views](/docs/{{version}}/views) to load the appropriately hashed asset. The `elixir` function will automatically determine the name of the hashed file:

	<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### Versioning Multiple Files

You may pass an array to the `version` method to version multiple files:

```javascript
elixir(function(mix) {
	mix.version(["css/all.css", "js/app.js"]);
});
```

Once the files have been versioned, you may use the `elixir` helper function to generate links to the proper hashed files. Remember, you only need to pass the name of the un-hashed file to the `elixir` helper function. The helper will use the un-hashed name to determine the current hashed version of the file:

	<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

	<script src="{{ elixir('js/app.js') }}"></script>

<a name="calling-existing-gulp-tasks"></a>
## Calling Existing Gulp Tasks

If you need to call an existing Gulp task from Elixir, you may use the `task` method. As an example, imagine that you have a Gulp task that simply speaks a bit of text when called:

```javascript
gulp.task("speak", function() {
	var message = "Tea...Earl Grey...Hot";

	gulp.src("").pipe(shell("say " + message));
});
```

If you wish to call this task from Elixir, use the `mix.task` method and pass the name of the task as the only argument to the method:

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

#### Custom Watchers

If you need to register a watcher to run your custom task each time some files are modified, pass a regular expression as the second argument to the `task` method:

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

<a name="writing-elixir-extensions"></a>
## Writing Elixir Extensions

If you need more flexibility than Elixir's `task` method can provide, you may create custom Elixir extensions. Elixir extensions allow you to pass arguments to your custom tasks. For example, you could write an extension like so:

```javascript
// File: elixir-extensions.js

var gulp = require("gulp");
var shell = require("gulp-shell");
var elixir = require("laravel-elixir");

elixir.extend("speak", function(message) {

	gulp.task("speak", function() {
		gulp.src("").pipe(shell("say " + message));
	});

	return this.queueTask("speak");

 });
```

That's it! You may either place this at the top of your Gulpfile, or instead extract it to a custom tasks file. For example, if you place your extensions in `elixir-extensions.js`, you may require the file from your main `Gulpfile` like so:

```javascript
// File: Gulpfile.js

var elixir = require("laravel-elixir");

require("./elixir-tasks")

elixir(function(mix) {
	mix.speak("Tea, Earl Grey, Hot");
});
```

#### Custom Watchers

If you would like your custom task to be re-triggered while running `gulp watch`, you may register a watcher:

```javascript
this.registerWatcher("speak", "app/**/*.php");

return this.queueTask("speak");
```
