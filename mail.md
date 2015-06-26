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

要发送消息，请使用 `Mail` [facade](/docs/{{version}}/facades) 上的 `send` 方法，`send` 方法接受三个参数，首先是包含邮件消息的[视图](/docs/{{version}}/views)名，其次是你希望传入视图中的数据数组，最后是一个 `Closure` 回调，用于接收一个消息实例，允许你自定义收件人，应该是和其它方面的邮件消息：

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

Since we are passing an array containing the `user` key in the example above, we could display the user's name within our e-mail view using the following PHP code:

	<?php echo $user->name; ?>

> **Note:** A `$message` variable is always passed to e-mail views, and allows the [inline embedding of attachments](#attachments). So, you should avoid passing a `message` variable in your view payload.

#### Building The Message

As previously discussed, the third argument given to the `send` method is a `Closure` allowing you to specify various options on the e-mail message itself. Using this Closure you may specify other attributes of the message, such as carbon copies, blind carbon copies, etc:

	Mail::send('emails.welcome', $data, function ($message) {
		$message->from('us@example.com', 'Laravel');

		$message->to('foo@example.com')->cc('bar@example.com');
	});

Here is a list of the available methods on the `$message` message builder instance:

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

> **Note:** The message instance passed to a `Mail::send` Closure extends the SwiftMailer message class, allowing you to call any method on that class to build your e-mail messages.

#### Mailing Plain Text

By default, the view given to the `send` method is assumed to contain HTML. However, by passing an array as the first argument to the `send` method, you may specify a plain text view to send in addition to the HTML view:

	Mail::send(['html.view', 'text.view'], $data, $callback);

Or, if you only need to send a plain text e-mail, you may specify this using the `text` key in the array:

	Mail::send(['text' => 'view'], $data, $callback);

#### Mailing Raw Strings

You may use the `raw` method if you wish to e-mail a raw string directly:

	Mail::raw('Text to e-mail', function ($message) {
		//
	});

<a name="attachments"></a>
### Attachments

To add attachments to an e-mail, use the `attach` method on the `$message` object passed to your Closure. The `attach` method accepts the full path to the file as its first argument:

	Mail::send('emails.welcome', $data, function ($message) {
		//

		$message->attach($pathToFile);
	});

When attaching files to a message, you may also specify the display name and / or MIME type by passing an `array` as the second argument to the `attach` method:

	$message->attach($pathToFile, ['as' => $display, 'mime' => $mime]);

<a name="inline-attachments"></a>
### Inline Attachments

#### Embedding An Image In An E-Mail View

Embedding inline images into your e-mails is typically cumbersome; however, Laravel provides a convenient way to attach images to your e-mails and retrieving the appropriate CID. To embed an inline image, use the `embed` method on the `$message` variable within your e-mail view. Remember, Laravel automatically makes the `$message` variable available to all of your e-mail views:

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

#### Embedding Raw Data In An E-Mail View

If you already have a raw data string you wish to embed into an e-mail message, you may use the `embedData` method on the `$message` variable:

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

<a name="queueing-mail"></a>
### Queueing Mail

#### Queueing A Mail Message

Since sending e-mail messages can drastically lengthen the response time of your application, many developers choose to queue e-mail messages for background sending. Laravel makes this easy using its built-in [unified queue API](/docs/{{version}}/queues). To queue a mail message, use the `queue` method on the `Mail` facade:

	Mail::queue('emails.welcome', $data, function ($message) {
		//
	});

This method will automatically take care of pushing a job onto the queue to send the mail message in the background. Of course, you will need to [configure your queues](/docs/{{version}}/queues) before using this feature.

#### Delayed Message Queueing

If you wish to delay the delivery of a queued e-mail message, you may use the `later` method. To get started, simply pass the number of seconds by which you wish to delay the sending of the message as the first argument to the method:

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
## Mail & Local Development

When developing an application that sends e-mail, you probably don't want to actually send e-mails to live e-mail addresses. Laravel provides several ways to "disable" the actual sending of e-mail messages.

#### Log Driver

One solution is to use the `log` mail driver during local development. This driver will write all e-mail messages to your log files for inspection. For more information on configuring your application per environment, check out the [configuration documentation](/docs/{{version}}/installation#environment-configuration).

#### Universal To

Another solution provided by Laravel is to set a universal recipient of all e-mails sent by the framework. This way, all the emails generated by your application will be sent to a specific address, instead of the address actually specified when sending the message. This can be done via the `to` option in your `config/mail.php` configuration file:

	'to' => [
	    'address' => 'dev@domain.com',
	    'name' => 'Dev Example'
	],

#### Mailtrap

Finally, you may use a service like [Mailtrap](https://mailtrap.io) and the `smtp` driver to send your e-mail messages to a "dummy" mailbox where you may view them in a true e-mail client. This approach has the benefit of allowing you to actually inspect the final e-mails in Mailtrap's message viewer.
