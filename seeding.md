# Database: Seeding

<<<<<<< HEAD
- [简介](#introduction)
- [编写填充](#writing-seeders)
	- [使用模型工厂](#using-model-factories)
	- [调用附加的填充](#calling-additional-seeders)
- [执行填充](#running-seeders)
=======
- [Introduction](#introduction)
- [Writing Seeders](#writing-seeders)
    - [Using Model Factories](#using-model-factories)
    - [Calling Additional Seeders](#calling-additional-seeders)
- [Running Seeders](#running-seeders)
>>>>>>> laravel/5.1

<a name="introduction"></a>
## 简介

Laravel 包含了一个使用 seed 类向数据库中填充测试数据的简单方法。所以的seed 类都存储在 `database/seeds` 中，seed 类可能有我们需要的任何名字，但是必须跟随一些明显的惯例，比如 `UserTableSeeder`等等，默认下，`DatabaseSeeder` 类由你的定义。通过这次课程，你可以使用 `call` 方法执行其他 seed 类，运行控制填充顺序。

<a name="writing-seeders"></a>
## 编写填充

产生数据填充，使用 `make:seeder` [Artisan 命令](/docs/{{version}}/artisan)框架产生的所有的数据填充都会放置在 `database/seeders` 目录中：

    php artisan make:seeder UserTableSeeder

一个 seeder 类仅仅包含一个默认 `run` 的方法。当执行 `db:seed ` [Artisan 命令](/docs/{{version}}/artisan)时这个方法就会被调用。你可以在 `run` 方法里面插入你想要插入的数据到数据库中，你可以使用[查询构造器](/docs/{{version}}/queries)自动插入数据，或者使用[Eloquent 模式工厂](/docs/{{version}}/testing#model-factories)

就像这个例子，让我们修改包含默认安装的 Laravel `DatabaseSeeder` 类。让我们添加一个数据库插入语句到 `run` 方法中：

    <?php

    use DB;
    use Illuminate\Database\Seeder;
    use Illuminate\Database\Eloquent\Model;

    class DatabaseSeeder extends Seeder
    {
        /**
         * Run the database seeds.
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => str_random(10),
                'email' => str_random(10).'@gmail.com',
                'password' => bcrypt('secret'),
            ]);
        }
    }

<a name="using-model-factories"></a>
### 使用模型工厂

当然，手动指明每一个模型的属性填充是相当麻烦的。反而，你可以使用 [模型工厂](/docs/{{version}}/testing#model-factories)更加便利地产生大量的数据库记录，首先，复习[模型工厂文档](/docs/{{version}}/testing#model-factories)如何定义自己的工厂，一旦你有自定义的工厂，你可能使用 `factory` 助手方法插入记录到数据中。

例如，让我们创建五十个用户，并关联上其他用户：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        factory('App\User', 50)->create()->each(function($u) {
            $u->posts()->save(factory('App\Post')->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 调用附加的填充

在 `DatabaseSeeder` 类里面，你可以使用 `call` 方法执行附加的填充类，使用 `call` 方法允许分裂数据库填充到多个文件里面，所以适合极为庞大并非单一的填充类。简单地传递你想要执行填充的类名：

    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        Model::unguard();

        $this->call(UserTableSeeder::class);
        $this->call(PostsTableSeeder::class);
        $this->call(CommentsTableSeeder::class);
    }

<a name="running-seeders"></a>
## 执行填充

一旦你有自己编写的填充类，你可以使用 `db:seed` Artisan 命令填充数据库，默认下，`db:seed` 命令执行 `DatabaseSeeder` 类，其中通常用于调用其它 seed 类，然而，你可以使用 `--class` 选项指明一个单独执行的特殊填充类：

    php artisan db:seed

    php artisan db:seed --class=UserTableSeeder

你也可以使用 `migrate:refresh` 命令填充数据库，其中会回滚和重新执行所有的迁移，该命令对彻底地重建数据库非常有用：

    php artisan migrate:refresh --seed
