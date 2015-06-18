# Eloquent: Serialization

- [Introduction](#introduction)
- [Basic Usage](#basic-usage)
- [Hiding Attributes From JSON](#hiding-attributes-from-json)
- [Appending Values To JSON](#appending-values-to-json)

<a name="introduction"></a>
## 介绍

在构建JSON API时,你会经常需要将模型和关系转换为数组或JSON。Eloquent可以方便的进行转换,以及设置序列化中的关系。

<a name="basic-usage"></a>
## 基本用法

#### 模型转换为数组

将模型及其加载  [关系relationships](/docs/{{version}}/eloquent-relationships) 转化为一个数组,你可以使用`toArray`方法。这是递归的方法,所以所有属性和关系将被转换成数组:

	$user = App\User::with('roles')->first();

	return $user->toArray();

你也可以将[集合collections](/docs/{{version}}/eloquent-collections)转换为数组:

	$users = App\User::all();

	return $users->toArray();

#### 模型转换为JSON

模型使用 `toJson` 方法转换为JSON。 同 `toArray`方法, `toJson` 方法也是递归式, 所有属性和关系都被转换成JSON:

	$user = App\User::find(1);

	return $user->toJson();

或者,你可以强制把一个模型或集合转成字符串,它将自动调用 `toJson` 方法:

	$user = App\User::find(1);

	return (string) $user;

从应用的路由或控制器返回的对象，可以直接转换为JSON。

	Route::get('users', function () {
		return App\User::all();
	});

<a name="hiding-attributes-from-json"></a>
## 隐藏 JSON 中的属性

有时候可能希望隐藏模型、数组或 JSON 中的某个属性, 比如密码, 可以在模型中添加 `$hidden` 属性:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The attributes that should be hidden for arrays.
		 *
		 * @var array
		 */
		protected $hidden = ['password'];
	}

> **Note:** When hiding relationships, use the relationship's **method** name, not its dynamic property name.

Alternatively, you may use the `visible` property to define a white-list of attributes that should be included in your model's array and JSON representation:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The attributes that should be visible in arrays.
		 *
		 * @var array
		 */
		protected $visible = ['first_name', 'last_name'];
	}

<a name="appending-values-to-json"></a>
## Appending Values To JSON

Occasionally, you may need to add array attributes that do not have a corresponding column in your database. To do so, first define an [accessor](/docs/{{version}}/eloquent-mutators) for the value:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Get the administrator flag for the user.
		 *
		 * @return bool
		 */
		public function getIsAdminAttribute()
		{
			return $this->attributes['admin'] == 'yes';
		}
	}

Once you have created the accessor, add the attribute name to the `appends` property on the model:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The accessors to append to the model's array form.
		 *
		 * @var array
		 */
		protected $appends = ['is_admin'];
	}

Once the attribute has been added to the `appends` list, it will be included in both the model's array and JSON forms. Attributes in the `appends` array will also respect the `visible` and `hidden` settings configured on the model.
