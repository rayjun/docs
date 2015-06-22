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
- [版本控制 / 缓存破坏](#versioning-and-cache-busting)
- [调用既有 Gulp 任务 ](#calling-existing-gulp-tasks)
- [编写 Elixir 扩展程序](#writing-elixir-extensions)

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
## 操作脚本

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

Elixir 还包含 `browserify` 方法，使得可以方便地加载所有模块和使用 EcmaScript 6。

这个任务假设你要编译的脚本文件存放在 `resources/assets/js` 目录中，生成的结果将存放于 `public/js/bundle.js` 中：

```javascript
elixir(function(mix) {
	mix.browserify('index.js');
});
```

<a name="babel"></a>
### Babel

`babel` 方法还可以用于将[EcmaScript 6 and 7](https://babeljs.io/docs/learn-es2015/)文件编译生成纯脚本文件，这个方法接受文件数组作为参数，文件路径相对于 `resources/assets/js` 目录，此方法将生成单个文件 `all.js` 到 `public/js` 目录：

```javascript
elixir(function(mix) {
	mix.babel([
                "order.js",
                "product.js"
        ]);
});
```

改变默认输出路径，只需要指定你希望的的路径作为第二个参数，除了编译方式，Babel 方法的签名和功能与 `mix.scripts()` 相同：


<a name="javascript"></a>
### 脚本

如果你有多个脚本文件想要编译生成单个文件，你可以使用 `scripts` 方法：

`scripts` 方法假设所有的文件路径都相对于 `resources/assets/js` 目录，并且默认会将结果存放于 `public/js/all.js` 中：

```javascript
elixir(function(mix) {
	mix.scripts([
		"jquery.js",
		"app.js"
	]);
});
```

如果你需要将多个脚本集合并为到不同的文件，你可以调用 `scripts` 方法多次，传入方法的第二个参数决定每个合并操作结果的文件名：

```javascript
elixir(function(mix) {
    mix.scripts(['app.js', 'controllers.js'], 'public/js/app.js')
       .scripts(['forum.js', 'threads.js'], 'public/js/forum.js');
});
```

如果你需要将所有的脚本文件合并到指定目录，你可以使用 `scriptsIn` 方法，结果存放于 `public/js/all.js`:

```javascript
elixir(function(mix) {
	mix.scriptsIn("public/js/some/directory");
});
```

<a name="versioning-and-cache-busting"></a>
## 版本控制 / 缓存失效

很多程序员在编译后的 assets 文件名前面加上时间戳或者唯一标记，从而强制浏览器加载最新的 assets 而不是仍旧使用过期的代码。Elixir 通过 `version` 来实现这一点。

`version` 方法接收一个相对于 `public` 目录的文件名且会添加一个唯一的哈希码到文件后，从而使缓存失效，例如，生成的文件名可能像这样 `all-16d570a7.css`：

```javascript
elixir(function(mix) {
	mix.version("css/all.css");
});
```

生成版本文件之后，你还可以在 [视图](/docs/{{version}}/views) 中使用 Laravel 全局方法 `elixir`，帮助你正确加载 asset 文件，`elixir` 方法会自动判断出哈希后的文件：

	<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

#### 版本控制多个文件

可以通过向 `version` 方法传入数组的方式实现多文件版本控制

```javascript
elixir(function(mix) {
	mix.version(["css/all.css", "js/app.js"]);
});
```

一旦文件加入版本控制，你可以使用 `elixir` 方法定位正确的哈希文件，请记住，你只需要传入未哈希的文件，`elixir`将帮助你找到当前哈希后的文件：

	<link rel="stylesheet" href="{{ elixir('css/all.css') }}">

	<script src="{{ elixir('js/app.js') }}"></script>

<a name="calling-existing-gulp-tasks"></a>
## 调用既有 Gulp 任务 

如果你需要通过 Elixir 调用既有 Gulp 任务，你可以使用 `task` 方法，比如，你有一个 Gulp 任务，在调用时只输出一段文字：

```javascript
gulp.task("speak", function() {
	var message = "Tea...Earl Grey...Hot";

	gulp.src("").pipe(shell("say " + message));
});
```

如果你希望通过 Elixir 调用这个任务，使用 `mix.task`，以任务名作为唯一参数传入此方法：

```javascript
elixir(function(mix) {
    mix.task('speak');
});
```

#### 自定义监听器

如果你需要注册一个监听器，每当文件被修改时运行自定义的任务，可以向 `task` 方法中传入一个正则表达式作为第二个参数：

```javascript
elixir(function(mix) {
    mix.task('speak', 'app/**/*.php');
});
```

<a name="writing-elixir-extensions"></a>
## 编写 Elixir 扩展程序

如果你需要比 Elixir 所能提供的更多的灵活性，你可以创建自定义的 Elixir 扩展程序，扩展程序允许你传入参数到自定义任务中，例如，你可以像这样编写扩展：

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

就是这样！你也可以将这段扩展程序放在 Gulp 文件的头部，或者将其抽离出来作为单独的自定义任务文件。例如，如果你将扩展程序放在 `elixir-extensions.js` 中，你可以如下在入口文件 `Gulpfile` 中引用：

```javascript
// File: Gulpfile.js

var elixir = require("laravel-elixir");

require("./elixir-tasks")

elixir(function(mix) {
	mix.speak("Tea, Earl Grey, Hot");
});
```

#### 设置监听器

如果你想通过运行 `gulp watch` 反复触发自定的任务，你可以注册一个监听器：

```javascript
this.registerWatcher("speak", "app/**/*.php");

return this.queueTask("speak");
```
