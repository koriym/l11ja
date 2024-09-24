# パスワードのリセット

- [はじめに](#introduction)
    - [モデルの準備](#model-preparation)
    - [データベースの準備](#database-preparation)
    - [信頼できるホストの設定](#configuring-trusted-hosts)
- [ルーティング](#routing)
    - [パスワードリセットリンクのリクエスト](#requesting-the-password-reset-link)
    - [パスワードのリセット](#resetting-the-password)
- [期限切れトークンの削除](#deleting-expired-tokens)
- [カスタマイズ](#password-customization)

<a name="introduction"></a>
## はじめに

ほとんどのウェブアプリケーションは、ユーザーが忘れたパスワードをリセットする方法を提供しています。すべてのアプリケーションに手動で再実装するのではなく、Laravelはパスワードリセットリンクの送信とパスワードの安全なリセットのための便利なサービスを提供しています。

> NOTE:  
> すぐに始めたいですか？新しいLaravelアプリケーションにLaravelの[アプリケーションスターターキット](starter-kits.md)をインストールしてください。Laravelのスターターキットは、忘れたパスワードのリセットを含む、認証システム全体のスキャフォールディングを行います。

<a name="model-preparation"></a>
### モデルの準備

Laravelのパスワードリセット機能を使用する前に、アプリケーションの`App\Models\User`モデルが`Illuminate\Notifications\Notifiable`トレイトを使用している必要があります。通常、このトレイトは新しいLaravelアプリケーションで作成されるデフォルトの`App\Models\User`モデルに既に含まれています。

次に、`App\Models\User`モデルが`Illuminate\Contracts\Auth\CanResetPassword`コントラクトを実装していることを確認してください。フレームワークに含まれる`App\Models\User`モデルは、このインターフェースを既に実装しており、`Illuminate\Auth\Passwords\CanResetPassword`トレイトを使用して、インターフェースを実装するために必要なメソッドを含んでいます。

<a name="database-preparation"></a>
### データベースの準備

アプリケーションのパスワードリセットトークンを保存するためのテーブルを作成する必要があります。通常、これはLaravelのデフォルトの`0001_01_01_000000_create_users_table.php`データベースマイグレーションに含まれています。

<a name="configuring-trusted-hosts"></a>
### 信頼できるホストの設定

デフォルトでは、LaravelはHTTPリクエストの`Host`ヘッダの内容に関係なく、受信したすべてのリクエストに応答します。さらに、`Host`ヘッダの値は、Webリクエスト中にアプリケーションへの絶対URLを生成する際に使用されます。

通常、NginxやApacheなどのWebサーバーを設定して、特定のホスト名に一致するリクエストのみをアプリケーションに送信するようにする必要があります。ただし、Webサーバーを直接カスタマイズする権限がなく、Laravelに特定のホスト名にのみ応答するように指示する必要がある場合は、アプリケーションの`bootstrap/app.php`ファイルで`trustHosts`ミドルウェアメソッドを使用して行うことができます。これは、特にアプリケーションがパスワードリセット機能を提供する場合に重要です。

このミドルウェアメソッドの詳細については、[`TrustHosts`ミドルウェアのドキュメント](requests.md#configuring-trusted-hosts)を参照してください。

<a name="routing"></a>
## ルーティング

ユーザーがパスワードをリセットできるようにするためのサポートを適切に実装するために、いくつかのルートを定義する必要があります。まず、ユーザーがメールアドレスを介してパスワードリセットリンクをリクエストできるようにするためのルートのペアを定義します。次に、ユーザーがパスワードリセットリンクをクリックし、パスワードリセットフォームを完了した後に、実際にパスワードをリセットするためのルートのペアを定義します。

<a name="requesting-the-password-reset-link"></a>
### パスワードリセットリンクのリクエスト

<a name="the-password-reset-link-request-form"></a>
#### パスワードリセットリンクリクエストフォーム

まず、パスワードリセットリンクをリクエストするために必要なルートを定義します。始めに、パスワードリセットリンクリクエストフォームを含むビューを返すルートを定義します。

    Route::get('/forgot-password', function () {
        return view('auth.forgot-password');
    })->middleware('guest')->name('password.request');

このルートによって返されるビューには、ユーザーが特定のメールアドレスのパスワードリセットリンクをリクエストできるようにする`email`フィールドを含むフォームが必要です。

<a name="password-reset-link-handling-the-form-submission"></a>
#### フォーム送信の処理

次に、「パスワードを忘れた」ビューからのフォーム送信リクエストを処理するルートを定義します。このルートは、メールアドレスを検証し、対応するユーザーにパスワードリセットリクエストを送信する役割を担います。

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Password;

    Route::post('/forgot-password', function (Request $request) {
        $request->validate(['email' => 'required|email']);

        $status = Password::sendResetLink(
            $request->only('email')
        );

        return $status === Password::RESET_LINK_SENT
                    ? back()->with(['status' => __($status)])
                    : back()->withErrors(['email' => __($status)]);
    })->middleware('guest')->name('password.email');

次に進む前に、このルートをより詳しく見てみましょう。まず、リクエストの`email`属性が検証されます。次に、Laravelの組み込みの「パスワードブローカー」（`Password`ファサードを介して）を使用して、ユーザーにパスワードリセットリンクを送信します。パスワードブローカーは、指定されたフィールド（この場合はメールアドレス）でユーザーを取得し、Laravelの組み込みの[通知システム](notifications.md)を介してユーザーにパスワードリセットリンクを送信します。

`sendResetLink`メソッドは、「ステータス」スラッグを返します。このステータスは、Laravelの[ローカリゼーション](localization.md)ヘルパーを使用して翻訳され、ユーザーに対してリクエストのステータスに関するユーザーフレンドリーなメッセージを表示することができます。パスワードリセットステータスの翻訳は、アプリケーションの`lang/{lang}/passwords.php`言語ファイルによって決定されます。ステータススラッグの可能な各値のエントリは、`passwords`言語ファイル内にあります。

> NOTE:  
> デフォルトでは、Laravelアプリケーションのスケルトンには`lang`ディレクトリは含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisanコマンドを介して公開することができます。

`Password`ファサードの`sendResetLink`メソッドを呼び出すときに、Laravelがどのようにしてアプリケーションのデータベースからユーザーレコードを取得するか疑問に思うかもしれません。Laravelのパスワードブローカーは、認証システムの「ユーザープロバイダ」を利用してデータベースレコードを取得します。パスワードブローカーによって使用されるユーザープロバイダは、`config/auth.php`設定ファイルの`passwords`設定配列内で設定されます。カスタムユーザープロバイダの作成について詳しくは、[認証ドキュメント](authentication.md#adding-custom-user-providers)を参照してください。

> NOTE:  
> パスワードリセットを手動で実装する場合、ビューとルートの内容を自分で定義する必要があります。すべての必要な認証と検証ロジックを含むスキャフォールディングを希望する場合は、[Laravelアプリケーションスターターキット](starter-kits.md)を確認してください。

<a name="resetting-the-password"></a>
### パスワードのリセット

<a name="the-password-reset-form"></a>
#### パスワードリセットフォーム

次に、ユーザーがパスワードリセットリンクをクリックし、新しいパスワードを提供した後に、実際にパスワードをリセットするために必要なルートを定義します。まず、パスワードリセットリンクをクリックしたときに表示されるパスワードリセットフォームを返すルートを定義します。このルートは、後でパスワードリセットリクエストを検証するために使用する`token`パラメータを受け取ります。

    Route::get('/reset-password/{token}', function (string $token) {
        return view('auth.reset-password', ['token' => $token]);
    })->middleware('guest')->name('password.reset');

このルートによって返されるビューには、`email`フィールド、`password`フィールド、`password_confirmation`フィールド、および秘密の`$token`値を含む`token`フィールドを含むフォームが必要です。

<a name="password-reset-handling-the-form-submission"></a>
#### フォーム送信の処理

もちろん、パスワードリセットフォームの送信を実際に処理するためのルートを定義する必要があります。このルートは、受信リクエストを検証し、データベース内のユーザーのパスワードを更新する役割を担います。

    use App\Models\User;
    use Illuminate\Auth\Events\PasswordReset;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Password;
    use Illuminate\Support\Str;

    Route::post('/reset-password', function (Request $request) {
        $request->validate([
            'token' => 'required',
            'email' => 'required|email',
            'password' => 'required|min:8|confirmed',
        ]);

        $status = Password::reset(
            $request->only('email', 'password', 'password_confirmation', 'token'),
            function (User $user, string $password) {
                $user->forceFill([
                    'password' => Hash::make($password)
                ])->setRememberToken(Str::random(60));

                $user->save();

                event(new PasswordReset($user));
            }
        );

        return $status === Password::PASSWORD_RESET
                    ? redirect()->route('login')->with('status', __($status))
                    : back()->withErrors(['email' => [__($status)]]);
    })->middleware('guest')->name('password.update');

次に進む前に、このルートをより詳しく見てみましょう。まず、リクエストの`token`、`email`、および`password`属性が検証されます。次に、Laravelの組み込みの「パスワードブローカー」（`Password`ファサードを介して）を使用して、パスワードリセットリクエストの資格情報を検証します。

与えられたトークン、メールアドレス、パスワードがパスワードブローカーに対して有効であれば、`reset`メソッドに渡されたクロージャが呼び出されます。このクロージャ内で、パスワードリセットフォームに提供されたユーザーインスタンスと平文のパスワードを受け取り、データベース内のユーザーのパスワードを更新することができます。

`reset`メソッドは「ステータス」スラッグを返します。このステータスは、Laravelの[ローカリゼーション](localization.md)ヘルパーを使用して翻訳し、ユーザーにリクエストのステータスに関するユーザーフレンドリーなメッセージを表示することができます。パスワードリセットステータスの翻訳は、アプリケーションの`lang/{lang}/passwords.php`言語ファイルによって決定されます。ステータススラッグの各可能な値のエントリは、`passwords`言語ファイル内にあります。アプリケーションに`lang`ディレクトリが含まれていない場合は、`lang:publish` Artisanコマンドを使用して作成できます。

次に進む前に、Laravelが`Password`ファサードの`reset`メソッドを呼び出す際に、どのようにアプリケーションのデータベースからユーザーレコードを取得するのか疑問に思うかもしれません。Laravelのパスワードブローカーは、認証システムの「ユーザープロバイダー」を利用してデータベースレコードを取得します。パスワードブローカーによって使用されるユーザープロバイダーは、`config/auth.php`設定ファイルの`passwords`設定配列内で設定されます。カスタムユーザープロバイダーの書き方について詳しく知りたい場合は、[認証ドキュメント](authentication.md#adding-custom-user-providers)を参照してください。

<a name="deleting-expired-tokens"></a>
## 期限切れトークンの削除

期限切れのパスワードリセットトークンは、データベース内に残っています。しかし、`auth:clear-resets` Artisanコマンドを使用してこれらのレコードを簡単に削除できます：

```shell
php artisan auth:clear-resets
```

このプロセスを自動化したい場合は、コマンドをアプリケーションの[スケジューラ](scheduling.md)に追加することを検討してください：

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('auth:clear-resets')->everyFifteenMinutes();

<a name="password-customization"></a>
## カスタマイズ

<a name="reset-link-customization"></a>
#### リセットリンクのカスタマイズ

`ResetPassword`通知クラスが提供する`createUrlUsing`メソッドを使用して、パスワードリセットリンクのURLをカスタマイズできます。このメソッドは、通知を受け取るユーザーインスタンスとパスワードリセットリンクトークンを受け取るクロージャを受け入れます。通常、このメソッドは`App\Providers\AppServiceProvider`サービスプロバイダの`boot`メソッドから呼び出す必要があります：

    use App\Models\User;
    use Illuminate\Auth\Notifications\ResetPassword;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        ResetPassword::createUrlUsing(function (User $user, string $token) {
            return 'https://example.com/reset-password?token='.$token;
        });
    }

<a name="reset-email-customization"></a>
#### リセットメールのカスタマイズ

パスワードリセットリンクをユーザーに送信するために使用される通知クラスを簡単に変更できます。まず、`App\Models\User`モデルの`sendPasswordResetNotification`メソッドをオーバーライドします。このメソッド内で、独自に作成した[通知クラス](notifications.md)を使用して通知を送信できます。パスワードリセットの`$token`は、このメソッドが受け取る最初の引数です。この`$token`を使用して、選択したパスワードリセットURLを構築し、ユーザーに通知を送信できます：

    use App\Notifications\ResetPasswordNotification;

    /**
     * Send a password reset notification to the user.
     *
     * @param  string  $token
     */
    public function sendPasswordResetNotification($token): void
    {
        $url = 'https://example.com/reset-password?token='.$token;

        $this->notify(new ResetPasswordNotification($url));
    }
