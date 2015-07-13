# 哈希

- [介绍](#introduction)
- [基本用法](#basic-usage)

<a name="introduction"></a>
## 介绍

Laravel 的 `Hash` [facade](/docs/{{version}}/facades) 提供安全的 Bcrypt 哈希算法对用户密码加密，如果你使用 Laravel 自带的 `AuthController`，在注册和认证时将自动使用 Bcrypt 算法：

Bcrypt 算法是对密码加密的一个非常好的选择，因为其「加密系数（work factor）」是可调节的，这意味着其用于生成哈希值的时间会随硬件性能的提升而加长。

<a name="basic-usage"></a>
## 基本用法

你可以通过调用 `Hash` facade 上的 `make` 方法来对密码加密：

    <?php

    namespace App\Http\Controllers;

    use Hash;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Update the password for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function updatePassword(Request $request, $id)
        {
            $user = User::findOrFail($id);

            // Validate the new password length...

            $user->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }


或者, 你也可以使用全局的 `bcrypt` 辅助函数：

    bcrypt('plain-text');

#### 根据哈希值验证密码

`check` 方法验证一个给定纯文本字符串是否与一个哈希值相匹配，如果使用 [Laravel 自带的](/docs/{{version}}/authentication) `AuthController`，你不必直接使用 `check` 方法，因为控制器将自动调用此验证方法：

    if (Hash::check('plain-text', $hashedPassword)) {
        // The passwords match...
    }

#### 检查密码是否需要重新加密码

`needsRehash` 函数用于判断自密码加密码以来，加密器使用的加密系数是否发生改变：

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }
