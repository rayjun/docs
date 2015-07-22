# 加密

- [配置](#configuration)
- [基本用法](#basic-usage)

<a name="configuration"></a>
## 配置

在使用 Laravel 加密器之前，你需要将 `config/app.php` 配置文件中的 key 设置为 32 位的随机字符串，如果此配置项未正确设置，所有被 Laravel 加密的值将会不安全：

<a name="basic-usage"></a>
## 基本用法

#### 加密

你可以使用 `Crypt` [facade](/docs/{{version}}/facades) 加密码一个值，所有被加密码的值使用的是 OpenSSL 和 `AES-256-CBC` 算法，此外，所有被加密的值署有一个消息验证码（MAC），用来侦测任何对加密字符串的篡改：

例如，我们可以使用 `encrypt` 方法对密码加密且将其存储到[Eloquent 模型](/docs/{{version}}/eloquent)中：

    <?php

    namespace App\Http\Controllers;

    use Crypt;
    use App\User;
    use Illuminate\Http\Request;
    use App\Http\Controllers\Controller;

    class UserController extends Controller
    {
        /**
         * Store a secret message for the user.
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => Crypt::encrypt($request->secret)
            ])->save();
        }
    }

#### 解密

当然，你可以通过 `Crypt` facade 上的 `decrypt` 方法将一个值解密，如果此值未被正确加密，例如，当 MAC 无效时，一个 `Illuminate\Contracts\Encryption\DecryptException` 异常将被抛出：

    use Illuminate\Contracts\Encryption\DecryptException;

    try {
        $decrypted = Crypt::decrypt($encryptedValue);
    } catch (DecryptException $e) {
        //
    }
