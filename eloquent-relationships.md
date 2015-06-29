# Eloquent: Relationships

- [简介](#introduction)
- [定义关联](#defining-relationships)
	- [一对一](#one-to-one)
	- [一对多](#one-to-many)
	- [多对多](#many-to-many)
	- [Has Many Through](#has-many-through)
	- [多态关联](#polymorphic-relations)
	- [多对多的多态关联](#many-to-many-polymorphic-relations)
- [Querying Relations](#querying-relations)
	- [Eager Loading](#eager-loading)
	- [Constraining Eager Loads](#constraining-eager-loads)
	- [Lazy Eager Loading](#lazy-eager-loading)
- [Inserting Related Models](#inserting-related-models)
	- [Many To Many Relationships](#inserting-many-to-many-relationships)

<a name="introduction"></a>
## 简介

数据库表经常是相互关联的。例如，一个博客有很多评论，或者一个订单关联着下单用户。Eloquent 使得管理和使用这些关联变得很容易，并支持多种不同的关联：

- [一对一](#one-to-one)
- [一对多](#one-to-many)
- [多对多](#many-to-many)
- [Has Many Through](#has-many-through)
- [多态关联](#polymorphic-relations)
- [多对多的多态关联](#many-to-many-polymorphic-relations)

<a name="defining-relationships"></a>
## 定义关联

Eloquent 关联定义跟 Eloquent 模型类的函数一样。因此，类似于 Eloquent 模型，关联也作为强大的[查询构造器](/docs/{{version}}/queries)，定义关联就像函数提供强大的方法联结和查询能力。例如：
Eloquent relationships are defined as functions on your Eloquent model classes. Since, like Eloquent models themselves, relationships also serve as powerful [query builders](/docs/{{version}}/queries), defining relationships as functions provides powerful method chaining and querying capabilities. For example:

	$user->posts()->where('active', 1)->get();

但是，之前太深入使用关联，让我们来学习另一种定义方式：
But, before diving too deep into using relationships, let's learn how to define each type:

<a name="one-to-one"></a>
### 一对一

一对一关联是最基本的关联。例如，一个用户可能只有一个相对应的 `电话`。要定义这种关联，我们只需要在 `User` 模型上置入一个 `phone`方法。该 `phone` 方法必须返回基础的 Eloquent 模型类上的 `hasOne` 方法的记录：
A one-to-one relationship is a very basic relation. For example, a `User` model might be associated with one `Phone`. To define this relationship, we place a `phone` method on the `User` model. The `phone` method should return the results of the `hasOne` method on the base Eloquent model class:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Get the phone record associated with the user.
		 */
		public function phone()
		{
			return $this->hasOne('App\Phone');
		}
	}

`hasOne` 方法的第一个参数是关联的模型名称。一旦定义了关联，我们必须使用[动态属性](#dynamic-properties)接收关联的记录。动态属性允许你访问关联方法犹如这些属性定义在模型上一样：
The first argument passed to the `hasOne` method is the name of the related model. Once the relationship is defined, we may retrieve the related record using Eloquent's [dynamic properties](#dynamic-properties). Dynamic properties allow you to access relationship functions as if they were properties defined on the model:

	$phone = User::find(1)->phone;

Eloquent 认为关联的外键以模型名为基础。因此，`Phone` 方法自动认为有一个`user_id` 外键。如果你想要覆盖这个约定，你必须传递第二个参数给 `hasOne` 方法：
Eloquent assumes the foreign key of the relationship based on the model name. In this case, the `Phone` model is automatically assumed to have a `user_id` foreign key. If you wish to override this convention, you may pass a second argument to the `hasOne` method:

	return $this->hasOne('App\Phone', 'foreign_key');

此外，Eloquent 认为该外键必须有一个与父级 `id` 列相匹配的值，另外，Eloquent 将检测 `Phone` 记录的 `user_id` 列中的用户 `id` 列的值。如果你喜欢使用关联除了 `id` 外的一个值，你必须传递自定义的键当做第三个参数给 `hasOne` 方法：
Additionally, Eloquent assumes that the foreign key should have a value matching the `id` column of the parent. In other words, Eloquent will look for the value of the user's `id` column in the `user_id` column of the `Phone` record. If you would like the relationship to use a value other than `id`, you may pass a third argument to the `hasOne` method specifying your custom key:

	return $this->hasOne('App\Phone', 'foreign_key', 'local_key');

#### 定义相反的关联

所以，我们能访问 `User` 的 `Phone` 模型。现在，让我们在 `Phone` 模型上定义一个关联，那样我们就能访问 `User` 所拥有的电话。我们能够使用 `belongsTo` 方法定义一个相反的 `hasOne` 关联：
So, we can access the `Phone` model from our `User`. Now, let's define a relationship on the `Phone` model that will let us access the `User` the owns the phone. We can define the inverse of a `hasOne` relationship using the `belongsTo` method:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Phone extends Model
	{
		/**
		 * Get the user that owns the phone.
		 */
		public function user()
		{
			return $this->belongsTo('App\User');
		}
	}

在上面的例子中，Eloquent 将尝试把 `Phone` 模型的 `user_id` 和 `User` 模型的一个 `id` 相匹配。Eloquent 将通过检验关联方法名和加后缀 `_id` 的方法名来决定默认的外键名。然而，如果 `Phone` 模型的外键不是 `user_id`，你可能传递一个自定义的键作为 `belongsTo` 方法的第二个参数：
In the example above, Eloquent will try to match the `user_id` from the `Phone` model to an `id` on the `User` model. Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with `_id`. However, if the foreign key on the `Phone` model is not `user_id`, you may pass a custom key name as the second argument to the `belongsTo` method:

	/**
	 * Get the user that owns the phone.
	 */
	public function user()
	{
		return $this->belongsTo('App\User', 'foreign_key');
	}

如果父级模型没有使用 `id` 作为主键，或是你想要连接到子模型的不同的列，
你必须把父级表的一个自定义值作为第三个参数传递给 `belongsTo` 方法：
If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

	/**
	 * Get the user that owns the phone.
	 */
	public function user()
	{
		return $this->belongsTo('App\User', 'foreign_key', 'other_key');
	}

<a name="one-to-many"></a>
### 一对多

一对多模型经常定义单一模型拥有任意数量的其它模型的关联。例如，一篇博客帖子用无限数量的评论。像所以的 Eloquent 关联一样，一对多关联被定义成 Eloquent 模型的一个函数：
A "one-to-many" relationship is used to define relationships where a single model owns any amount of other models. For example, a blog post may have an infinite number of comments. Like all other Eloquent relationships, one-to-many relationships are defined by placing a function on your Eloquent model:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Post extends Model
	{
		/**
		 * Get the comments for the blog post.
		 */
		public function comments()
		{
			return $this->hasMany('App\Comment');
		}
	}

记住，Eloquent 总是自动决定 `Comment` 模型的的适当的外键列。按照惯例，Eloquent 将获得拥有的模型和使用后缀 `_id` 的 "snake case" 名。所以，
比如这个例子，Eloquent 将决定 `Comment` 模型的外键是 `post_id`。
Remember, Eloquent will automatically determine the proper foreign key column on the `Comment` model. By convention, Eloquent will take the "snake case" name of the owning model and suffix it with `_id`. So, for this example, Eloquent will assume the foreign key on the `Comment` model is `post_id`.

一旦定义了关联，我们就能通过访问 `comments` 属性获取评论集，记住了，由于
Eloquent 提供了 "动态属性"，我们能够像访问模型的属性那样访问关联方法
Once the relationship has been defined, we can access the collection of comments by accessing the `comments` property. Remember, since Eloquent provides "dynamic properties", we can access relationship functions as if they were defined as properties on the model:

	$comments = App\Post::find(1)->comments;

	foreach ($comments as $comment) {
		//
	}

当然，因为所以的关联也可充当查询构造器，你可以在调用 `comments` 方法取回的评论上添加约束和继续在查询上链接条件：
Of course, since all relationships also serve as query builders, you can add further constraints to which comments are retrieved by calling the `comments` method and continuing to chain conditions onto the query:

	$comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

类似 `hasOne` 方法，你可以通过传递额外参数给 `hasMany` 方法的方式覆盖外键和主键
：
Like the `hasOne` method, you may also override the foreign and local keys by passing additional arguments to the `hasMany` method:

	return $this->hasMany('App\Comment', 'foreign_key');

	return $this->hasMany('App\Comment', 'foreign_key', 'local_key');

#### 定义相反的关联

我们现在能够访问一个帖子的所有评论，让我们定义一个允许一条评论访问它父级贴子的关联。要定义一个相反的 `hasMany` 关联，需要定义调用了 `belongsTo` 方法的
子模型的关联方法，
Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `hasMany` relationship, define a relationship function on the child model which calls the `belongsTo` method:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Comment extends Model
	{
		/**
		 * Get the post that owns the comment.
		 */
		public function post()
		{
			return $this->belongsTo('App\Post');
		}
	}

一旦定义了关联，我们可以通过访问 `post` 的"动态属性"获取 `Post` 模型的 `Comment`：
Once the relationship has been defined, we can retrieve the `Post` model for a `Comment` by accessing the `post` "dynamic property":

	$comment = App\Comment::find(1);

	echo $comment->post->title;

在上面的例子中，Eloquent 将尝试匹配来自 `Comment` 模型的 `post_id` 和 `Post` 模型的 `id`。Eloquent 将通过检验关联方法名和加后缀 `_id` 的方法名来决定默认的外键名。然而，如果 `Phone` 模型的外键不是 `user_id`，你可能传递一个自定义的键作为 `belongsTo` 方法的第二个参数：
In the example above, Eloquent will try to match the `post_id` from the `Comment` model to an `id` on the `Post` model. Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with `_id`. However, if the foreign key on the `Comment` model is not `post_id`, you may pass a custom key name as the second argument to the `belongsTo` method:

	/**
	 * Get the post that owns the comment.
	 */
	public function post()
	{
		return $this->belongsTo('App\Post', 'foreign_key');
	}

如果父级模型没有使用 `id` 作为主键，或是你想要连接到子模型的不同的列，
你必须把父级表的一个自定义值作为第三个参数传递给 `belongsTo` 方法：
If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

	/**
	 * Get the post that owns the comment.
	 */
	public function post()
	{
		return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
	}

<a name="many-to-many"></a>
### 多对多

多对多关联比 `hasOne` 和 `hasMany` 关联稍微更加复杂。
这种关联的一个例子是一个用户有很多角色，其中这些角色也分享给其他用户。例如，多个用户角色都是"管理员"。要定义这样的关联，需要 `users`, `roles`, 和 `role_user` 三个数据表。其中 `role_user` 表来源于相关的模型名称字母顺序，并且包含 `user_id` 和 `role_id` 列。
Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and contains the `user_id` and `role_id` columns.

通过编写调用底层 Eloquent 类的 `belongsToMany` 方法的一个方法来定义多对多关联。
例如，让我们来定义 `User` 模型的 `roles` 方法：
Many-to-many relationships are defined by writing a method that calls the `belongsToMany` method on the base Eloquent class. For example, let's define the `roles` method on our `User` model:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * The roles that belong to the user.
		 */
		public function roles()
		{
			return $this->belongsToMany('App\Role');
		}
	}

一旦定义了关联，你就可以使用 `roles` 动态属性获取 用户的角色：
Once the relationship is defined, you may access the user's roles using the `roles` dynamic property:

	$user = App\User::find(1);

	foreach ($user->roles as $role) {
		//
	}

当然，像所以的关联类型一样，你可以调用 `roles` 方法上继续连接查询约束在关联上：
Of course, like all other relationship types, you may call the `roles` method to continue chaining query constraints onto the relationship:

	$roles = App\User::find(1)->roles()->orderBy('name')->get();

正如前面提起到的，要决定关联连接的表的表名，Eloquent 将按字母顺序连接两个相关的模型名。然而，你可以传递第二参数给 `belongsToMany` 模型，自由地覆盖这样约定：
As mentioned previously, to determine the table name of the relationship's joining table, Eloquent will join the two related model names in alphabetical order. However, you are free to override this convention. You may do so by passing a second argument to the `belongsToMany` method:

	return $this->belongsToMany('App\Role', 'user_roles');

除了自定义关联表名称之外，你可以通过传递额外的参数给 `belongsToMany` 方法自定义表的键列名。第三个参数是在定义关联的模型外键名，第四个参数是要关联模型的外键名：
In addition to customizing the name of the joining table, you may also customize the column names of the keys on the table by passing additional arguments to the `belongsToMany` method. The third argument is the foreign key name of the model on which you are defining the relationship, while the fourth argument is the foreign key name of the model that you are joining to:

	return $this->belongsToMany('App\Role', 'user_roles', 'user_id', 'role_id');

#### 定义相反的关联

要定义相反的多对多关联，需要简单地调用其它关联模型的 `belongsToMany`。继续我们的角色案例，让我们定义 `Role` 模型的 `users` 方法：
To define the inverse of a many-to-many relationship, you simply place another call to `belongsToMany` on your related model. To continue our user roles example, let's define the `users` method on the `Role` model:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Role extends Model
	{
		/**
		 * The users that belong to the role.
		 */
		public function users()
		{
			return $this->belongsToMany('App\User');
		}
	}

正如你所见到的一样，该关联定义恰恰与 `User` 一样，使用简单地参考 `App\User` 模型的例外。所有我们再使用 `belongsToMany`  方法，当定义相反的多对多关联时，
所有的表和自定义选项键都是可得到的。
As you can see, the relationship is defined exactly the same as its `User` counterpart, with the exception of simply referencing the `App\User` model. Since we're reusing the `belongsToMany` method, all of the usual table and key customization options are available when defining the inverse of many-to-many relationships.

#### 检索中间表的列

正如您已经了解到的一样，使用 多对多关联查询存在的中间表。Eloquent 提供了一些非常有帮助的与表交互的方法。例如，我们假定 `User` 对象有许多关联的 `Role` 对象。在取得关联之后，使用模型的 `pivot` 属性获取中间表：
As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the intermediate table using the `pivot` attribute on the models:

	$user = App\User::find(1);

	foreach ($user->roles as $role) {
		echo $role->pivot->created_at;
	}

注意，我们得到的每一个 `Role` 模型都自动分配一个 `pivot` 属性。
该属性包含了一个模型代表的中间表，并像其它 Eloquent 模型一样被使用。
Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used like any other Eloquent model.

默认情况下，模型键仅仅存在于 `pivot` 对象上。如果枢轴表包含其它属性，当定义关联时，你必须指明它们：
By default, only the model keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

	return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');

如果你想要枢轴表自动维护 `created_at` 和 `updated_at` 时间戳，在定义关联上使用 `withTimestamps` 方法：
If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `withTimestamps` method on the relationship definition:

	return $this->belongsToMany('App\Role')->withTimestamps();

<a name="has-many-through"></a>
### 有多凭证

"has-many-through" 关联提供一个方便的,快捷的捷径访问遥远的关联的中间关联。
例如，一个 `国家` 模型必须有多个 `帖子` 模型透过中间 `用户` 模型。
在这个例子中，你能轻而易举地收集给定的国家的所有的博客帖子。让我们看定义该关联必需的表：
The "has-many-through" relationship provides a convenient short-cut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Post` models through an intermediate `User` model. In this example, you could easily gather all blog posts for a given country. Let's look at the tables required to define this relationship:

	countries
		id - integer
		name - string

	users
		id - integer
		country_id - integer
		name - string

	posts
		id - integer
		user_id - integer
		title - string

尽管 `帖子` 不包含 `country_id` 列，`hasManyThrough` 关联提供了访问国家帖子的渠道 `$country->posts` 。执行这个查询，Eloquent 检查中间 `users` 表的 `country_id`。查找匹配的用户 IDS 之后，它们被用于查询 `posts` 表。
Though `posts` does not contain a `country_id` column, the `hasManyThrough` relation provides access to a country's posts via `$country->posts`. To perform this query, Eloquent inspects the `country_id` on the intermediate `users` table. After finding the matching user IDs, they are used to query the `posts` table.

现在，我们有检查过的关联表结构，让我们在 `Country` 模型上定义它：
Now that we have examined the table structure for the relationship, let's define it on the `Country` model:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Country extends Model
	{
		/**
		 * Get all of the posts for the country.
		 */
		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User');
		}
	}


传递给 `hasManyThrough` 方法的第一个参数是我们希望获取的最后的模型名称，
而第二个参数是中间模型的名称。
The first argument passed to the `hasManyThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

当执行关联查询时，将使用典型的 Eloquent 外键约束。如果你喜欢自定义关联的键，
你必须把它们作为第三参数和第四参数传递给 `hasManyThrough` 方法。第三个参数是
中间模型的外键名，而第四个参数是最后模拟的外键名。
Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasManyThrough` method. The third argument is the name of the foreign key on the intermediate model, while the fourth argument is the name of the foreign key on the final model.

	class Country extends Model
	{
		public function posts()
		{
			return $this->hasManyThrough('App\Post', 'App\User', 'country_id', 'user_id');
		}
	}

<a name="polymorphic-relations"></a>
### 多态关联

#### 表结构

多态关联允许一个模型属于一个以上单一关联的其他模型。例如，设想你想要保存职员相册以及产品。使用动态关联，可以使用单一 `相册` 表的两套方案。
第一，让我们检查所需的表结构建立这种关联：
Polymorphic relations allow a model to belong to more than one other model on a single association. For example, imagine you want to store photos for your staff members and for your products. Using polymorphic relationships, you can use a single `photos` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

	staff
		id - integer
		name - string

	products
		id - integer
		price - integer

	photos
		id - integer
		path - string
		imageable_id - integer
		imageable_type - string

需要注意的两个重要的列分别是 `photos` 表的 `imageable_id` 和 `imageable_type` 列。`imageable_id` 列包含职员或产品的 ID 值，`imageable_type` 包含模型的类名，当访问 `imageable` 关联时，`imageable_type` 列是如何返回 ORM 决定的模型"类型"。
Two important columns to note are the `imageable_id` and `imageable_type` columns on the `photos` table. The `imageable_id` column will contain the ID value of the owning staff or product, while the `imageable_type` column will contain the class name of the owning model. The `imageable_type` column is how the ORM determines which "type" of owning model to return when accessing the `imageable` relation.

#### 模型结构

下一步，让我们检查需要建立这种关联的模型定义：
Next, let's examine the model definitions needed to build this relationship:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Photo extends Model
	{
		/**
		 * Get all of the owning imageable models.
		 */
		public function imageable()
		{
			return $this->morphTo();
		}
	}

	class Staff extends Model
	{
		/**
		 * Get all of the staff member's photos.
		 */
		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}
	}

	class Product extends Model
	{
		/**
		 * Get all of the product's photos.
		 */
		public function photos()
		{
			return $this->morphMany('App\Photo', 'imageable');
		}
	}

#### 检索多态关联

一旦定义了数据库表和模型，你可以通过模型访问关联。例如，获取一个职员的所以图片，可以简单地使用 `photos` 动态属性：
Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the photos for a staff member, we can simply use the `photos` dynamic property:

	$staff = App\Staff::find(1);

	foreach ($staff->photos as $photo) {
		//
	}

你可以通过访问执行调用我们例子中的 `morphTo` 方法名获取动态模型的动态关联
。这是 `Photo` 模型的 `imageable` 方法。所以，我们将像动态属性那样获取方法：
You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphTo`. In our case, that is the `imageable` method on the `Photo` model. So, we will access that method as a dynamic property:

	$photo = App\Photo::find(1);

	$imageable = $photo->imageable;

`Photo` 模型的 `imageable` 关联返回 `Staff` 或者 `Product` 两者之间的一个实例，取决于拥有的图片模型类型。
The `imageable` relation on the `Photo` model will return either a `Staff` or `Product` instance, depending on which type of model owns the photo.

<a name="many-to-many-polymorphic-relations"></a>
### 多对多动态关联

#### 表结构

除了传统的动态关联之外，你也可以定义"多对多"动态关联，例如，一个博客 `帖子` and `视频` 模型可以多态关联一个 `标题` 模型。使用多对多动态关联允许有
In addition to traditional polymorphic relations, you may also define "many-to-many" polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. First, let's examine the table structure:

	posts
		id - integer
		name - string

	videos
		id - integer
		name - string

	tags
		id - integer
		name - string

	taggables
		tag_id - integer
		taggable_id - integer
		taggable_type - string

#### Model Structure

Next, we're ready to define the relationships on the model. The `Post` and `Video` models will both have a `tags` method that calls the `morphToMany` method on the base Eloquent class:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Post extends Model
	{
		/**
		 * Get all of the tags for the post.
		 */
		public function tags()
		{
			return $this->morphToMany('App\Tag', 'taggable');
		}
	}

#### Defining The Inverse Of The Relationship

Next, on the `Tag` model, you should define a method for each of its related models. So, for this example, we will define a `posts` method and a `videos` method:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Tag extends Model
	{
		/**
		 * Get all of the posts that are assigned this tag.
		 */
		public function posts()
		{
			return $this->morphedByMany('App\Post', 'taggable');
		}

		/**
		 * Get all of the videos that are assigned this tag.
		 */
		public function videos()
		{
			return $this->morphedByMany('App\Video', 'taggable');
		}
	}

#### Retrieving The Relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you can simply use the `tags` dynamic property:

	$post = App\Post::find(1);

	foreach ($post->tags as $tag) {
		//
	}

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the method that performs the call to `morphedByMany`. In our case, that is the `posts` or `videos` methods on the `Tag` model. So, you will access those methods as dynamic properties:

	$tag = App\Tag::find(1);

	foreach ($tag->videos as $video) {
		//
	}

<a name="querying-relations"></a>
## Querying Relations

Since all types of Eloquent relationships are defined via functions, you may call those functions to obtain an instance of the relationship without actually executing the relationship queries. In addition, all types of Eloquent relationships also serve as [query builders](/docs/{{version}}/queries), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL against your database.

For example, imagine a blog system in which a `User` model has many associated `Post` models:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class User extends Model
	{
		/**
		 * Get all of the posts for the user.
		 */
		public function posts()
		{
			return $this->hasMany('App\Post');
		}
	}

You may query the `posts` relationship and add additional constraints to the relationship like so:

	$user = App\User::find(1);

	$user->posts()->where('active', 1)->get();

Note that you are able to use any of the [query builder](/docs/{{version}}/queries) on the relationship!

#### Relationship Methods Vs. Dynamic Properties

If you do not need to add additional constraints to an Eloquent relationship query, you may simply access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts like so:

	$user = App\User::find(1);

	foreach ($user->posts as $post) {
		//
	}

Dynamic properties are "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

#### Querying Relationship Existence

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` method:

	// Retrieve all posts that have at least one comment...
	$posts = App\Post::has('comments')->get();

You may also specify an operator and count to further customize the query:

	// Retrieve all posts that have three or more comments...
	$posts = Post::has('comments', '>=', 3)->get();

Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:

	// Retrieve all posts that have at least one comment with votes...
	$posts = Post::has('comments.votes')->get();

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint, such as checking the content of a comment:

	// Retrieve all posts with at least one comment containing words like foo%
	$posts = Post::whereHas('comments', function ($q) {
		$q->where('content', 'like', 'foo%');
	})->get();

<a name="eager-loading"></a>
### Eager Loading

When accessing Eloquent relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model. Eager loading alleviates the N + 1 query problem. To illustrate the N + 1 query problem, consider a `Book` model that is related to `Author`:

	<?php

	namespace App;

	use Illuminate\Database\Eloquent\Model;

	class Book extends Model
	{
		/**
		 * Get the author that wrote the book.
		 */
		public function author()
		{
			return $this->belongsTo('App\Author');
		}
	}

Now, let's retrieve all books and their authors:

	$books = App\Book::all();

	foreach ($books as $book) {
		echo $book->author->name;
	}

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries: 1 for the original book, and 25 additional queries to retrieve the author of each book.

Thankfully, we can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

	$books = App\Book::with('author')->get();

	foreach ($books as $book) {
		echo $book->author->name;
	}

For this operation, only two queries will be executed:

	select * from books

	select * from authors where id in (1, 2, 3, 4, 5, ...)

#### Eager Loading Multiple Relationships

Sometimes you may need to eager load several different relationships in a single operation. To do so, just pass additional arguments to the `with` method:

	$books = App\Book::with('author', 'publisher')->get();

#### Nested Eager Loading

To eager load nested relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts in one Eloquent statement:

	$books = Book::with('author.contacts')->get();

<a name="constraining-eager-loads"></a>
### Constraining Eager Loads

Sometimes you may wish to eager load a relationship, but also specify additional query constraints for the eager loading query. Here's an example:

	$users = App\User::with(['posts' => function ($query) {
		$query->where('title', 'like', '%first%');

	}])->get();

In this example, Eloquent will only eager load posts that if the post's `title` column contains the word `first`. Of course, you may call other [query builder](/docs/{{version}}/queries) to further customize the eager loading operation:

	$users = App\User::with(['posts' => function ($query) {
		$query->orderBy('created_at', 'desc');

	}])->get();

<a name="lazy-eager-loading"></a>
### Lazy Eager Loading

Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

	$books = App\Book::all();

	if ($someCondition) {
		$books->load('author', 'publisher');
	}

If you need set additional query constraints on the eager loading query, you may pass a `Closure` to the `load` method:

	$books->load(['author' => function ($query) {
		$query->orderBy('published_date', 'asc');
	}]);

<a name="inserting-related-models"></a>
## Inserting Related Models

#### The Save Method

Eloquent provides convenient methods for adding new models to relationships. For example, perhaps you need to insert a new `Comment` for a `Post` model. Instead of manually setting the `post_id` attribute on the `Comment`, you may insert the `Comment` directly from the relationship's `save` method:

	$comment = new App\Comment(['message' => 'A new comment.']);

	$post = App\Post::find(1);

	$comment = $post->comments()->save($comment);

Notice that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `save` method will automatically add the appropriate `post_id` value to the new `Comment` model.

If you need to save multiple related models, you may use the `saveMany` method:

	$post = App\Post::find(1);

	$post->comments()->saveMany([
		new App\Comment(['message' => 'A new comment.']),
		new App\Comment(['message' => 'Another comment.']),
	]);

#### Save & Many To Many Relationships

When working with a many-to-many relationship, the `save` method accepts an array of additional intermediate table attributes as its second argument:

	App\User::find(1)->roles()->save($role, ['expires' => $expires]);

#### The Create Method

In addition to the `save` and `saveMany` methods, you may also use the `create` method, which accepts an array of attributes, creates a model, and inserts it into the database. Again, the difference between `save` and `create` is that `save` accepts a full Eloquent model instance while `create` accepts a plain PHP `array`:

	$post = App\Post::find(1);

	$comment = $post->comments()->create([
		'message' => 'A new comment.',
	]);

Before using the `create` method, be sure to review the documentation on attribute [mass assignment](/docs/{{version}}/eloquent#mass-assignment).

<a name="updating-belongs-to-relationships"></a>
#### Updating "Belongs To" Relationships

When updating a `belongsTo` relationship, you may use the `associate` method. This method will set the foreign key on the child model:

	$account = App\Account::find(10);

	$user->account()->associate($account);

	$user->save();

When removing a `belongsTo` relationship, you may use the `dissociate` method. This method will reset the foreign key as well as the relation on the child model:

	$user->account()->dissociate();

	$user->save();

<a name="inserting-many-to-many-relationships"></a>
### Many To Many Relationships

#### Attaching / Detaching

When working with many-to-many relationships, Eloquent provides a few additional helper methods to make working with related models more convenient. For example, let's imagine a user can have many roles and a role can have many users. To attach a role to a user by inserting a record in the intermediate table that joins the models, use the `attach` method:

	$user = App\User::find(1);

	$user->roles()->attach($roleId);

When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

	$user->roles()->attach($roleId, ['expires' => $expires]);

Of course, sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will remove the appropriate record out of the intermediate table; however, both models will remain in the database:

	// Detach a single role from the user...
	$user->roles()->detach($roleId);

	// Detach all roles from the user...
	$user->roles()->detach();

For convenience, `attach` and `detach` also accept arrays of IDs as input:

	$user = App\User::find(1);

	$user->roles()->detach([1, 2, 3]);

	$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);

#### Syncing For Convenience

You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the array will exist in the intermediate table:

	$user->roles()->sync([1, 2, 3]);

You may also pass additional intermediate table values with the IDs:

	$user->roles()->sync([1 => ['expires' => true], 2, 3]);
