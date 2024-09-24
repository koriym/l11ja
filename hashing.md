# ハッシュ化

- [イントロダクション](#introduction)
- [設定](#configuration)
- [基本的な使い方](#basic-usage)
    - [パスワードのハッシュ化](#hashing-passwords)
    - [パスワードがハッシュに一致するかの確認](#verifying-that-a-password-matches-a-hash)
    - [パスワードの再ハッシュが必要かの判断](#determining-if-a-password-needs-to-be-rehashed)
- [ハッシュアルゴリズムの検証](#hash-algorithm-verification)

<a name="introduction"></a>
## イントロダクション

Laravelの`Hash` [ファサード](facades.md)は、ユーザーパスワードの保存に安全なBcryptとArgon2のハッシュ化を提供します。[Laravelアプリケーションスターターキット](starter-kits.md)のいずれかを使用している場合、デフォルトでは登録と認証にBcryptが使用されます。

Bcryptはパスワードのハッシュ化に適しています。なぜならその「作業係数」は調整可能であり、ハードウェアの性能が向上するにつれてハッシュ生成にかかる時間を増やすことができるからです。パスワードのハッシュ化において、遅いことは良いことです。アルゴリズムがパスワードをハッシュ化するのに時間がかかるほど、悪意のあるユーザーがアプリケーションに対するブルートフォース攻撃に使用される可能性のあるすべての文字列ハッシュ値の「レインボーテーブル」を生成するのに時間がかかります。

<a name="configuration"></a>
## 設定

Laravelはデフォルトでデータのハッシュ化に`bcrypt`ハッシュドライバを使用します。しかし、他にも[`argon`](https://en.wikipedia.org/wiki/Argon2)や[`argon2id`](https://en.wikipedia.org/wiki/Argon2)など、いくつかのハッシュドライバがサポートされています。

アプリケーションのハッシュドライバは、`HASH_DRIVER`環境変数を使用して指定できます。ただし、Laravelのすべてのハッシュドライバオプションをカスタマイズしたい場合は、`config:publish` Artisanコマンドを使用して完全な`hashing`設定ファイルを公開する必要があります。

```bash
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## 基本的な使い方

<a name="hashing-passwords"></a>
### パスワードのハッシュ化

`Hash`ファサードの`make`メソッドを呼び出すことで、パスワードをハッシュ化できます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class PasswordController extends Controller
    {
        /**
         * ユーザーのパスワードを更新する。
         */
        public function update(Request $request): RedirectResponse
        {
            // 新しいパスワードの長さを検証...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();

            return redirect('/profile');
        }
    }

<a name="adjusting-the-bcrypt-work-factor"></a>
#### Bcrypt作業係数の調整

Bcryptアルゴリズムを使用している場合、`make`メソッドで`rounds`オプションを使用してアルゴリズムの作業係数を管理できます。ただし、Laravelが管理するデフォルトの作業係数はほとんどのアプリケーションに適しています。

    $hashed = Hash::make('password', [
        'rounds' => 12,
    ]);

<a name="adjusting-the-argon2-work-factor"></a>
#### Argon2作業係数の調整

Argon2アルゴリズムを使用している場合、`make`メソッドで`memory`、`time`、`threads`オプションを使用してアルゴリズムの作業係数を管理できます。ただし、Laravelが管理するデフォルト値はほとんどのアプリケーションに適しています。

    $hashed = Hash::make('password', [
        'memory' => 1024,
        'time' => 2,
        'threads' => 2,
    ]);

> NOTE:  
> これらのオプションの詳細については、[Argonハッシュに関する公式PHPドキュメント](https://secure.php.net/manual/en/function.password-hash.php)を参照してください。

<a name="verifying-that-a-password-matches-a-hash"></a>
### パスワードがハッシュに一致するかの確認

`Hash`ファサードが提供する`check`メソッドを使用すると、指定された平文文字列が指定されたハッシュに対応するかどうかを確認できます。

    if (Hash::check('plain-text', $hashedPassword)) {
        // パスワードが一致します...
    }

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### パスワードの再ハッシュが必要かの判断

`Hash`ファサードが提供する`needsRehash`メソッドを使用すると、パスワードがハッシュ化されてからハッシャーが使用する作業係数が変更されたかどうかを判断できます。一部のアプリケーションでは、このチェックをアプリケーションの認証プロセス中に実行することを選択します。

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('plain-text');
    }

<a name="hash-algorithm-verification"></a>
## ハッシュアルゴリズムの検証

ハッシュアルゴリズムの操作を防ぐために、Laravelの`Hash::check`メソッドは最初に指定されたハッシュがアプリケーションの選択されたハッシュアルゴリズムを使用して生成されたかどうかを検証します。アルゴリズムが異なる場合、`RuntimeException`例外がスローされます。

これは、ハッシュアルゴリズムが変更されることを期待していないほとんどのアプリケーションにとって期待される動作であり、異なるアルゴリズムは悪意のある攻撃の兆候である可能性があります。ただし、アプリケーション内で複数のハッシュアルゴリズムをサポートする必要がある場合、例えばあるアルゴリズムから別のアルゴリズムに移行する場合など、`HASH_VERIFY`環境変数を`false`に設定することでハッシュアルゴリズムの検証を無効にできます。

```ini
HASH_VERIFY=false
```
