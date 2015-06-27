# 邮件

- [介绍](#introduction)
- [发送邮件](#sending-mail)
	- [附件](#attachments)
	- [内嵌附件](#inline-attachments)
	- [邮件队列](#queueing-mail)
- [邮件与本地开发](#mail-and-local-development)

<a name="introduction"></a>
## 介绍

Laravel 提供了一个基于 [SwiftMailer](http://swiftmailer.org) 函数库的简洁 API，且为 Mandrill, Amazon SES, PHP 的 `mail` 函数以及 `sendmail` 提供了驱动，使你可以通过本地或基于云端的服务，快速实现邮件发送。

### 驱动必备件条

基于 API 的驱动，如 Mailgun 和 Mandrill 通常比 SMTP 服务器要更简单快捷，所有 API 型的驱动都要求你的应用程序安装 Guzzle HTTP 函数库，你可以通过向你的 `composer.json` 文件中添加下面一行代码来安装 Guzzle：

	"guzzlehttp/guzzle": "~5.3|~6.0"

#### Mailgun 驱动

要使用 Mailgun 驱动，首先安装 Guzzle，然后将你的 `config/mail.php` 配置文件中的 `driver` 项配置成 `mailgun`，下一步，确保你的 `config/services.php` 配置文件中包含如下选项：

	'mailgun' => [
		'domain' => 'your-mailgun-domain',
		'secret' => 'your-mailgun-key',
	],

#### Mandrill 驱动

要使用 Mandrill 驱动，首先安装 Guzzle，然后设置你的 `config/mail.php` 文件中的 `driver` 项为 `mandrill`，下一步，确保你的 `config/services.php` 配置文件中包含如下选项：

	'mandrill' => [
		'secret' => 'your-mandrill-key',
	],

#### SES 驱动

要使用 Amazon SES 驱动，安装 PHP 版 Amazon AWS SDK，你可以通过向  `composer.json` 文件的  `require`  区域添加如下行来安装：

	"aws/aws-sdk-php": "~3.0"

接下来，设置你的 `config/mail.php` 配置文件中的 `driver` 项为 `ses`，然后，确保你的 `config/services.php` 配置文件包含以配置项：

	'ses' => [
		'key' => 'your-ses-key',
		'secret' => 'your-ses-secret',
		'region' => 'ses-region',  // e.g. us-east-1
	],

<a name="sending-mail"></a>
## 发送邮件

Laravel 允许你将你的电子邮件消息存储为[视图](/docs/{{version}}/views)格式，例如，为了组织你的邮件，你可以在 `resources/views` 目录中创建一个 `emails` 目录：

要发送消息，请使用 `Mail` [facade](/docs/{{version}}/facades) 上的 `send` 方法，`send` 方法接受三个参数，首先是包含邮件消息的[视图](/docs/{{version}}/views)名，其次是你希望传入视图中的数据数组，最后是一个 `Closure` 回调，用于接收一个消息实例，允许你自定义收件人，主题以及邮件的其它信息：

	<?php

	namespace App\Http\Controllers;

	use Mail;
	use App\User;
	use Illuminate\Http\Request;
	use App\Http\Controllers\Controller;

	class UserController extends Controller
	{
		/**
		 * Send an e-mail reminder to the user.
		 *
		 * @param  Request  $request
		 * @param  int  $id
		 * @return Response
		 */
		public function sendEmailReminder(Request $request, $id)
		{
			$user = User::findOrFail($id);

			Mail::send('emails.reminder', ['user' => $user], function ($m) use ($user) {
				$m->to($user->email, $user->name)->subject('Your Reminder!');
			});
		}
	}

在上例中，因为我们传入了一个包含 `user` 键的数组，我们可以使用如下 PHP 代码将用户的名称显示在电子邮件视图中：

	<?php echo $user->name; ?>

> **注意:** `$message` 变量常被传入邮件视图，并且允许[内联嵌入附件](#attachments)，所以你应当避免传入 `message` 变量。

#### 构造邮件消息

如上讨论，第三个传入到 `send` 方法的是一个 `Clousure`，使你可以指定不同的参数项到电子邮件消息上。使用这个闭包，你可以指定消息的其它属性，例如副本，创建副本如下：

	Mail::send('emails.welcome', $data, function ($message) {
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');
	});

这是 `$message` 构造器实例上的可用方法列表：

	$message->from($address, $name = null);
	$message->sender($address, $name = null);
	$message->to($address, $name = null);
	$message->cc($address, $name = null);
	$message->bcc($address, $name = null);
	$message->replyTo($address, $name = null);
	$message->subject($subject);
	$message->priority($level);
	$message->attach($pathToFile, array $options = []);

	// Attach a file from a raw $data string...
	$message->attachData($data, $name, array $options = []);

	// Get the underlying SwiftMailer message instance...
	$message->getSwiftMessage();

> **注意:** 传入 `Mail::send` 闭包中的消息实例继承自 SwiftMailer 消息类，使你可以调用这个类上的任何方法来构造你的电子邮件消息。

#### 纯文本邮件

默认情况下，传入 `send` 方法的视图假设包含的是 HTML，然而通过向 `send` 方法传入一个数组作为第一个参数，除 HTML 视图外，你可以指定一个纯文本视图：

	Mail::send(['html.view', 'text.view'], $data, $callback);

或者，如果你只需要发送纯文本邮件，你可以在数组中以 `text` 键来指定：

	Mail::send(['text' => 'view'], $data, $callback);

#### 原始字符串邮件

如果你希望直接发送原始字符串的邮件，你可以使用 `raw` 方法：

	Mail::raw('Text to e-mail', function ($message) {
		//
	});

<a name="attachments"></a>
### 附件

添加附件到一封邮件中，请使用 `$message` 对象上的 `attach` 方法，`attach` 接受传入文件的全路径作为第一个参数：

	Mail::send('emails.welcome', $data, function ($message) {
		//

		$message->attach($pathToFile);
	});

当给消息添加附件时，你还可以通过向 `attch` 方法传入一个 `array` 作为第二个参数，来指定显示的名称或者 MIME 类型：

	$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

<a name="inline-attachments"></a>
### 内联附件

#### 在邮件视图中嵌入图片

将图片内嵌到邮件中通常很繁琐，然后，Laravel 提供了方便的方式将图片添加到邮件中和获取适当的 CID，要嵌入内联图片，请使用 `embed` 邮件视图中，`$message` 对象上的 `embed` 方法，请记住，Laravel 自动让 `$message` 变量在所有的邮件视图中可用：

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### 在邮件视图中嵌入原始数据

如果你已经有原始数据希望可以嵌入邮件消息中，你可以使用 `$message` 上的`embedData` 方法：

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

<a name="queueing-mail"></a>
### 邮件队列

#### 将邮件加入队列

因为发送消息可能严重影响应用程序的响应时间，许多程序员选择将邮件消息加入队列而在后台发送，Laravel 通过内建的[统一队列 API](/docs/{{version}}/queues) 让这件事变得非常容易，要将邮件加入队列，请使用 `Mail` facade 上的 `queue` 方法：

	Mail::queue('emails.welcome', $data, function ($message) {
		//
	});

这个方法负责自动将一个任务（job）加入队列而在后台将邮件发送，当免烧砖机，在使用此功能前，你需要先[配置你的队列](/docs/{{version}}/queues) ：

#### 延时邮件队列

你如果你希望将邮件队列消息的发送延迟，你可使用 `later` 方法，通过向此方法的第一个参数传入一个分钟数来表示延迟的发送的时间：

	Mail::later(5, 'emails.welcome', $data, function ($message) {
		//
	});

#### Pushing To Specific Queues

If you wish to specify a specific queue on which to push the message, you may do so using the `queueOn` and `laterOn` methods:

	Mail::queueOn('queue-name', 'emails.welcome', $data, function ($message) {
		//
	});

	Mail::laterOn('queue-name', 5, 'emails.welcome', $data, function ($message) {
		//
	});

<a name="mail-and-local-development"></a>
## 邮件与本地开发

当开发邮件发送程序时，你可能不想实际上发送邮件到真实的邮件地址，Laravel 提供多种方法「屏蔽」实际的邮件发送：

#### 日志驱动

一种解决方案是在本地开发中使用 `log` 邮件驱动，此驱动会将所以有邮件消息配置文档写入日志文件以供调查。对于更多的关于应用环境的配置信息，请查看[配置文档](/docs/{{version}}/installation#environment-configuration)。

#### 全局发送

Laravel 提供的另一种解决方案是设置邮件的的全局收件人，通过这种方式，由程序生成的所有邮件将发送到指定地址，而不是发送邮件实际指定的地址，可以通过 `config/mail.php` 配置文件中的 `to` 配置来设置：

	'to' => [
	    'address' => 'dev@domain.com',
	    'name' => 'Dev Example'
	],

#### Mailtrap

最后，你可以使用 [Mailtrap](https://mailtrap.io) 这样的服务和 `smtp` 驱动，将你的邮件消息发送到「模拟」邮箱，然后在邮件客户端查看，这个方法的好处是你可以实际上在 Mailtrap 的消息查看器上查看邮件。
