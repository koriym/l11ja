# 暗号化

- [はじめに](#introduction)
- [設定](#configuration)
    - [暗号化キーのグレースフルなローテーション](#gracefully-rotating-encryption-keys)
- [暗号化の使用](#using-the-encrypter)

<a name="introduction"></a>
## はじめに

Laravelの暗号化サービスは、OpenSSLを使用してAES-256およびAES-128暗号化でテキストを暗号化および復号化するためのシンプルで便利なインターフェースを提供します。Laravelのすべての暗号化された値は、メッセージ認証コード（MAC）を使用して署名されるため、暗号化された後に基になる値を変更または改ざんすることはできません。

<a name="configuration"></a>
## 設定

Laravelの暗号化機能を使用する前に、`config/app.php`設定ファイルで`key`設定オプションを設定する必要があります。この設定値は`APP_KEY`環境変数によって駆動されます。`php artisan key:generate`コマンドを使用してこの変数の値を生成する必要があります。`key:generate`コマンドは、PHPの安全なランダムバイトジェネレータを使用して、アプリケーションの暗号的に安全なキーを構築します。通常、`APP_KEY`環境変数の値は[Laravelのインストール](installation.md)中に自動的に生成されます。

<a name="gracefully-rotating-encryption-keys"></a>
### 暗号化キーのグレースフルなローテーション

アプリケーションの暗号化キーを変更すると、すべての認証済みユーザーセッションがアプリケーションからログアウトされます。これは、セッションクッキーを含むすべてのクッキーがLaravelによって暗号化されるためです。さらに、以前の暗号化キーで暗号化されたデータを復号化することはできなくなります。

この問題を軽減するために、Laravelではアプリケーションの`APP_PREVIOUS_KEYS`環境変数に以前の暗号化キーをリストアップできます。この変数には、以前のすべての暗号化キーのカンマ区切りリストを含めることができます：

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

この環境変数を設定すると、Laravelは値を暗号化するときに常に「現在」の暗号化キーを使用します。ただし、値を復号化するとき、Laravelはまず現在のキーを試し、現在のキーで復号化に失敗した場合、Laravelはすべての以前のキーを試し、いずれかのキーが値を復号化できるまで続けます。

このグレースフルな復号化のアプローチにより、暗号化キーがローテーションされてもユーザーはアプリケーションを中断することなく使用し続けることができます。

<a name="using-the-encrypter"></a>
## 暗号化の使用

<a name="encrypting-a-value"></a>
#### 値の暗号化

`Crypt`ファサードによって提供される`encryptString`メソッドを使用して値を暗号化できます。すべての暗号化された値は、OpenSSLとAES-256-CBC暗号を使用して暗号化されます。さらに、すべての暗号化された値はメッセージ認証コード（MAC）で署名されます。統合されたメッセージ認証コードは、悪意のあるユーザーによって改ざんされた値の復号化を防ぎます：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * ユーザーのDigitalOcean APIトークンを保存します。
         */
        public function store(Request $request): RedirectResponse
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();

            return redirect('/secrets');
        }
    }

<a name="decrypting-a-value"></a>
#### 値の復号化

`Crypt`ファサードによって提供される`decryptString`メソッドを使用して値を復号化できます。メッセージ認証コードが無効な場合など、値を適切に復号化できない場合、`Illuminate\Contracts\Encryption\DecryptException`がスローされます：

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        // ...
    }

