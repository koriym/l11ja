# メール認証

- [はじめに](#introduction)
    - [モデルの準備](#model-preparation)
    - [データベースの準備](#database-preparation)
- [ルーティング](#verification-routing)
    - [メール認証通知](#the-email-verification-notice)
    - [メール認証ハンドラ](#the-email-verification-handler)
    - [認証メールの再送信](#resending-the-verification-email)
    - [ルートの保護](#protecting-routes)
- [カスタマイズ](#customization)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

多くのウェブアプリケーションでは、ユーザーがアプリケーションを使用する前にメールアドレスを確認する必要があります。Laravelは、各アプリケーションでこの機能を手動で再実装することを強制するのではなく、メール認証リクエストの送信と確認のための便利な組み込みサービスを提供します。

> NOTE:  
> すぐに始めたいですか？新しいLaravelアプリケーションに[Laravelアプリケーションスターターキット](starter-kits.md)のいずれかをインストールしてください。スターターキットは、メール認証サポートを含む、認証システム全体のスキャフォールディングを行います。

<a name="model-preparation"></a>
### モデルの準備

始める前に、`App\Models\User`モデルが`Illuminate\Contracts\Auth\MustVerifyEmail`契約を実装していることを確認してください。

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

このインターフェースがモデルに追加されると、新しく登録されたユーザーには自動的にメール認証リンクを含むメールが送信されます。これは、Laravelが`Illuminate\Auth\Events\Registered`イベントのために`Illuminate\Auth\Listeners\SendEmailVerificationNotification`[リスナー](events.md)を自動的に登録するためです。

[スターターキット](starter-kits.md)を使用せずにアプリケーション内で手動で登録を実装している場合、ユーザーの登録が成功した後に`Illuminate\Auth\Events\Registered`イベントをディスパッチすることを確認する必要があります。

    use Illuminate\Auth\Events\Registered;

    event(new Registered($user));

<a name="database-preparation"></a>
### データベースの準備

次に、`users`テーブルに`email_verified_at`カラムを含めて、ユーザーのメールアドレスが確認された日時を保存する必要があります。通常、これはLaravelのデフォルトの`0001_01_01_000000_create_users_table.php`データベースマイグレーションに含まれています。

<a name="verification-routing"></a>
## ルーティング

メール認証を適切に実装するために、3つのルートを定義する必要があります。まず、ユーザーに登録後にLaravelから送信されたメール認証リンクをクリックするように通知するビューを返すルートが必要です。

次に、ユーザーがメール内のメール認証リンクをクリックしたときに生成されるリクエストを処理するルートが必要です。

最後に、ユーザーが最初の認証リンクを誤って失った場合に認証リンクを再送信するためのルートが必要です。

<a name="the-email-verification-notice"></a>
### メール認証通知

前述のように、ユーザーに登録後にLaravelから送信されたメール認証リンクをクリックするように通知するビューを返すルートを定義する必要があります。このビューは、ユーザーがメールアドレスを確認せずにアプリケーションの他の部分にアクセスしようとしたときに表示されます。リンクは、`App\Models\User`モデルが`MustVerifyEmail`インターフェースを実装している限り、自動的にユーザーにメールで送信されます。

    Route::get('/email/verify', function () {
        return view('auth.verify-email');
    })->middleware('auth')->name('verification.notice');

メール認証通知を返すルートは、`verification.notice`という名前を付ける必要があります。このルート名を割り当てることは重要です。なぜなら、Laravelに含まれる`verified`ミドルウェアは、ユーザーがメールアドレスを確認していない場合、自動的にこのルート名にリダイレクトするからです。

> NOTE:  
> メール認証を手動で実装する場合、認証通知ビューの内容を自分で定義する必要があります。認証と確認のためのすべての必要なビューを含むスキャフォールディングが必要な場合は、[Laravelアプリケーションスターターキット](starter-kits.md)をチェックしてください。

<a name="the-email-verification-handler"></a>
### メール認証ハンドラ

次に、ユーザーがメール内のメール認証リンクをクリックしたときに生成されるリクエストを処理するルートを定義する必要があります。このルートは、`verification.verify`という名前を付け、`auth`と`signed`ミドルウェアを割り当てる必要があります。

    use Illuminate\Foundation\Auth\EmailVerificationRequest;

    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();

        return redirect('/home');
    })->middleware(['auth', 'signed'])->name('verification.verify');

次に、このルートについて詳しく見ていきましょう。まず、通常の`Illuminate\Http\Request`インスタンスの代わりに、`EmailVerificationRequest`リクエストタイプを使用していることに気づくでしょう。`EmailVerificationRequest`は、Laravelに含まれる[フォームリクエスト](validation.md#form-request-validation)です。このリクエストは、リクエストの`id`と`hash`パラメータの検証を自動的に行います。

次に、リクエストの`fulfill`メソッドを直接呼び出すことができます。このメソッドは、認証されたユーザーの`markEmailAsVerified`メソッドを呼び出し、`Illuminate\Auth\Events\Verified`イベントをディスパッチします。`markEmailAsVerified`メソッドは、`Illuminate\Foundation\Auth\User`基底クラスを介してデフォルトの`App\Models\User`モデルで利用可能です。ユーザーのメールアドレスが確認されたら、どこにでもリダイレクトできます。

<a name="resending-the-verification-email"></a>
### 認証メールの再送信

ユーザーが誤ってメールアドレス認証メールを紛失したり削除したりすることがあります。これに対応するために、ユーザーが認証メールの再送信をリクエストできるルートを定義することができます。そして、[認証通知ビュー](#the-email-verification-notice)内にシンプルなフォーム送信ボタンを配置して、このルートにリクエストを送信することができます。

    use Illuminate\Http\Request;

    Route::post('/email/verification-notification', function (Request $request) {
        $request->user()->sendEmailVerificationNotification();

        return back()->with('message', 'Verification link sent!');
    })->middleware(['auth', 'throttle:6,1'])->name('verification.send');

<a name="protecting-routes"></a>
### ルートの保護

[ルートミドルウェア](middleware.md)を使用して、確認済みのユーザーのみが特定のルートにアクセスできるようにすることができます。Laravelには、`Illuminate\Auth\Middleware\EnsureEmailIsVerified`ミドルウェアクラスのエイリアスである`verified`[ミドルウェアエイリアス](middleware.md#middleware-aliases)が含まれています。このエイリアスは、Laravelによって自動的に登録されるため、必要なのは`verified`ミドルウェアをルート定義にアタッチすることだけです。通常、このミドルウェアは`auth`ミドルウェアと組み合わせて使用されます。

    Route::get('/profile', function () {
        // 確認済みのユーザーのみがこのルートにアクセスできます...
    })->middleware(['auth', 'verified']);

未確認のユーザーがこのミドルウェアが割り当てられたルートにアクセスしようとすると、自動的に`verification.notice`[名前付きルート](routing.md#named-routes)にリダイレクトされます。

<a name="customization"></a>
## カスタマイズ

<a name="verification-email-customization"></a>
#### 認証メールのカスタマイズ

デフォルトのメール認証通知は、ほとんどのアプリケーションの要件を満たすはずですが、Laravelではメール認証メールメッセージの構築方法をカスタマイズすることができます。

始めるには、`Illuminate\Auth\Notifications\VerifyEmail`通知によって提供される`toMailUsing`メソッドにクロージャを渡します。クロージャは、通知を受け取る通知可能なモデルインスタンスと、ユーザーがメールアドレスを確認するためにアクセスする必要がある署名付きメール認証URLを受け取ります。クロージャは、`Illuminate\Notifications\Messages\MailMessage`のインスタンスを返す必要があります。通常、`toMailUsing`メソッドは、アプリケーションの`AppServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。

    use Illuminate\Auth\Notifications\VerifyEmail;
    use Illuminate\Notifications\Messages\MailMessage;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // ...

        VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
            return (new MailMessage)
                ->subject('Verify Email Address')
                ->line('Click the button below to verify your email address.')
                ->action('Verify Email Address', $url);
        });
    }

> NOTE:  
> メール通知の詳細については、[メール通知のドキュメント](notifications.md#mail-notifications)を参照してください。

<a name="events"></a>
## イベント

[Laravelアプリケーションスターターキット](starter-kits.md)を使用すると、メール認証プロセス中に`Illuminate\Auth\Events\Verified`[イベント](events.md)がディスパッチされます。アプリケーションのメール認証を手動で処理している場合、認証が完了した後にこれらのイベントを手動でディスパッチすることができます。

