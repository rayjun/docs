# Request Lifecycle

- [简介](#introduction)
- [生命周期概述](#lifecycle-overview)
- [Focus On Service Providers](#focus-on-service-providers)

<a name="introduction"></a>
## 简介

在现实生活中，如果你了解正在使用的工具的工作原理，你会更自信使用它。应用开发也不例外。
当你理解开发工具如何运作，你会更得心应手。

这篇文档的目的是给你一个对于Laravel原理的高维度的概括。对整理更好的把握，
让Laravel变得不那么"神奇"，同时让你更自信构建自己的应用。

如果你现在不能理解所有的术语，别灰心。只要大体上理解概念，学习其他章节的时候，
知识自然会得到积累。


<a name="lifecycle-overview"></a>
## 生命周期概述

#### 基本内容

Laravel应用所有请求的入口是`public/index.php`文件。所有的请求都被你的web服务器
定向到这个文件。`index.php`文件并没有太多代码，它仅仅是一个加载框架的启动文件。
`index.php`加载了Composer生成的自动加载定义，并且从`bootstrap/app.php`脚本中寻找
Laravel应用实例。Laravel的第一步动作就是创建一个应用实例/[服务容器](/docs/{{version}}/container)


#### HTTP / Console 核心

接着，根据请求的类型，请求被分发到HTTP核心或者Console核心。这两个核心的处于中心地位，所有的
请求都会经过核心。现在，我们先看位于`app/Http/Kernel.php`的HTTP核心。

HTTP核心类继承了`Illuminate\Foundation\Http\Kernel`，其中定义了一个数组`bootstrappers`，数组中的启动器会在请求被处理之前执行。
这些启动器配置了错误处理、日志、检测环境，等其他需要在请求被处理之前被初始化的任务。

HTTP核心定义了一组HTTP[中间件](/docs/{{version}}/middleware)，所有的请求在被处理之前必须
经过这些中间件。中间件可以读写HTTP session，判断应用是否处于维护状态，验证CSRF token等。

HTTP核心的`handle`方法非常简单：接受一个`Request`并且返回一个`Response`。把核心想象为一个
黑盒，它代表整个应用。给它传入HTTP请求，它就会返回HTTP响应。

#### 服务提供者

核心初始化中非常重要的一部分就是为应用加载服务提供者。所有的服务提供者在`config/app.php`
文件中的`providers`数组中配置。首先，`register`方法会被服务提供者调用。然后，当所有
的服务提供者被注册之后，`boot`方法才会被调用。

#### 分发请求

当应用初始化完成，并且所有服务提供者被注册后，`Request`会被交给路由器分发。
路由器会分发请求到控制器，同时会运行针对路径的中间件。

<a name="focus-on-service-providers"></a>
## 关注服务提供者

服务提供者是初始化Laravel应用的关键所在。应用运行过程很简单：创建应用实例，注册服务提供者，
请求被交给初始化后的应用处理。

清晰了解Laravel应用是如何通过服务提供者被构建，初始化的是非常重要的。当然，默认的服务提供者
在`app/Providers`目录下。

`AppServiceProvider`默认是空的。这个服务提供者是添加你自己的初始化过程和绑定服务容器的绝佳位置。
当然，对于大型应用，你可能会希望创建自己的服务提供者，
每个服务提供者可以有其各自的初始化粒度。
