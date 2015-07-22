# 本地化

<<<<<<< HEAD
- [介绍](#introduction)
- [基本用法](#basic-usage)
	- [复数化](#pluralization)
- [覆写提供商语言文件](#overriding-package-language-files)
=======
- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
    - [Pluralization](#pluralization)
- [Overriding Vendor Language Files](#overriding-vendor-language-files)
>>>>>>> laravel/5.1

<a name="introduction"></a>
## 介绍

Laravel 的本地化特性提供了一种方便的方式从各种语言文件中获取字符串，使你可以很方便地在你的应用程序中支持多语言。

语言字符串存储在 `resources/lang` 目录下的文件中，在这个目录中，需要为每一种程序所支持的语言创建一个子目录：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

所有语言文件仅返回一个键值关联数组，例如：

    <?php

    return [
        'welcome' => 'Welcome to our application'
    ];

#### 配置本地化

应用程序的默认语言存储在 `config/app.php` 配置文件中，当然，你可以修改这个值来满足你的应用程序的需要。你还可以使用 `App` facade 上的 `setLocale` 方法，在运行时更改当前语言：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

你还可以配置一个「备用」语言，当现行语言没有包含某个语言时将会使用，如同默认语言，备用语言也在 `config/app.php` 配置文件中配置：

    'fallback_locale' => 'en',

<a name="basic-usage"></a>
## 基本用法

你可以通过使用 `trans` 辅助函数从语言文件中获取语言串，此方法接受文件和语言串的键作为第一个参数，例如，让我们来获取 `resources/lang/messages.php` 文件中的 `welcome` 语言串：

    echo trans('messages.welcome');

当然，如果你正在使用 [Blade 模板引擎](/docs/{{version}}/blade)，你可使用这种 `{{ }}` 语法来输出语言串：

    {{ trans('messages.welcome') }}

如果指定的语言串不存在，`trans` 函数将只返回语言串的键，所以，如上例，当语言串不存时 `trans` 方法将返回 `messages.welcome`。

#### 替换言语串中的参数 

如果你愿意，你可以在你的语言串中定义占位符，所有的占位符都以 `:` 打头。例如，你可以定义一个带有占位符的欢迎字符串：

    'welcome' => 'Welcome, :name',

要在获取一个言语串时替换占位符，传入一个替代值数组作为第二个参数到 `trans` 函数中：

    echo trans('messages.welcome', ['name' => 'Dayle']);

<a name="pluralization"></a>
### 复数化

复数化是一个复杂的问题，因为每种语言都有许多关于复数的复杂规则，通过使用「管道」符，你可以将一个语言串的单数与复数形工区别开来:

    'apples' => 'There is one apple|There are many apples',

然后，你可以使用 `trans_choice` 函数根据一个给定的「数量」来获取语言串。在这个例子中，因为数量大于一，返回语言串的复数形式：

    echo trans_choice('messages.apples', 10);

因为 Laravel 翻译器是由 Symfony 翻译模块提供，你可以创建更为复杂的复数化规则：

    'apples' => '{0} There are none|[1,19] There are some|[20,Inf] There are many',

<<<<<<< HEAD
<a name="overriding-package-language-files"></a>
## 覆写提供商语言文件
=======
<a name="overriding-vendor-language-files"></a>
## Overriding Vendor Language Files
>>>>>>> laravel/5.1

有些发布包带有自己的语言文件，调整这些语言串，不是直接修改包中的核心文件，而是可以通过将我们自己的语言文件放置于 `resources/lang/vendor/{package}/{locale}` 目录来将其覆盖。

所以，假如你需要覆写 `messages.php` 文件中一个包的名字 `skyrim/hearthfire` 的英文语言，你可以在此处 `resources/lang/vendor/hearthfire/en/messages.php` 放置你修改的语言，在这个文件中，你只需要定义你希望覆写的语言串，任何你没有覆写的语言串仍然会从包的原始语言文件中获取。
