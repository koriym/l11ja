# Laravel Fortify

- [はじめに](#introduction)
    - [Fortifyとは？](#what-is-fortify)
    - [Fortifyをいつ使うべきか？](#when-should-i-use-fortify)
- [インストール](#installation)
    - [Fortifyの機能](#fortify-features)
    - [ビューの無効化](#disabling-views)
- [認証](#authentication)
    - [ユーザー認証のカスタマイズ](#customizing-user-authentication)
    - [認証パイプラインのカスタマイズ](#customizing-the-authentication-pipeline)
    - [リダイレクトのカスタマイズ](#customizing-authentication-redirects)
- [二要素認証](#two-factor-authentication)
    - [二要素認証の有効化](#enabling-two-factor-authentication)
    - [二要素認証による認証](#authenticating-with-two-factor-authentication)
    - [二要素認証の無効化](#disabling-two-factor-authentication)
- [登録](#registration)
    - [登録のカスタマイズ](#customizing-registration)
- [パスワードリセット](#password-reset)
    - [パスワードリセットリンクのリクエスト](#requesting-a-password-reset-link)
    - [パスワードのリセット](#resetting-the-password)
    - [パスワードリセットのカスタマイズ](#customizing-password-resets)
- [メール確認](#email-verification)
    - [ルートの保護](#protecting-routes)
- [パスワード確認](#password-confirmation)

<a name="introduction"></a>
## はじめに

[Laravel Fortify](https://github.com/laravel/fortify)は、Laravelのフロントエンドに依存しない認証バックエンドの実装です。Fortifyは、ログイン、登録、パスワードリセット、メール確認など、Laravelのすべての認証機能を実装するために必要なルートとコントローラを登録します。Fortifyをインストールした後、`route:list` Artisanコマンドを実行して、Fortifyが登録したルートを確認できます。

Fortifyは独自のユーザーインターフェースを提供しないため、登録したルートにリクエストを送信する独自のユーザーインターフェースと組み合わせて使用することを想定しています。このドキュメントの残りの部分では、これらのルートにリクエストを送信する方法について詳しく説明します。

> NOTE:  
> 覚えておいてください、FortifyはLaravelの認証機能の実装をスタートさせるためのパッケージです。**必ずしも使用する必要はありません。** [認証](authentication.md)、[パスワードリセット](passwords.md)、[メール確認](verification.md)のドキュメントに従って、Laravelの認証サービスを手動で操作することも常に自由です。

<a name="what-is-fortify"></a>
### Fortifyとは？

前述のように、Laravel Fortifyは、Laravelのフロントエンドに依存しない認証バックエンドの実装です。Fortifyは、ログイン、登録、パスワードリセット、メール確認など、Laravelのすべての認証機能を実装するために必要なルートとコントローラを登録します。

**Laravelの認証機能を使用するためにFortifyを使用する必要はありません。** [認証](authentication.md)、[パスワードリセット](passwords.md)、[メール確認](verification.md)のドキュメントに従って、Laravelの認証サービスを手動で操作することも常に自由です。

Laravelを初めて使用する場合は、Laravel Fortifyを使用する前に、[Laravel Breeze](starter-kits.md)アプリケーションスターターキットを試してみることをお勧めします。Laravel Breezeは、[Tailwind CSS](https://tailwindcss.com)で構築されたユーザーインターフェースを含む、アプリケーションの認証スキャフォールディングを提供します。Fortifyとは異なり、Breezeはルートとコントローラを直接アプリケーションに公開します。これにより、Laravel Fortifyを使用してこれらの機能を実装する前に、Laravelの認証機能を研究し、慣れることができます。

Laravel Fortifyは、基本的にLaravel Breezeのルートとコントローラをユーザーインターフェースを含まないパッケージとして提供します。これにより、特定のフロントエンドの意見に縛られることなく、アプリケーションの認証層のバックエンド実装を迅速にスキャフォールディングできます。

<a name="when-should-i-use-fortify"></a>
### Fortifyをいつ使うべきか？

Laravel Fortifyをいつ使用するか疑問に思うかもしれません。まず、Laravelの[アプリケーションスターターキット](starter-kits.md)のいずれかを使用している場合、Laravel Fortifyをインストールする必要はありません。すべてのLaravelアプリケーションスターターキットは、すでに完全な認証実装を提供しています。

アプリケーションスターターキットを使用しておらず、アプリケーションに認証機能が必要な場合、2つの選択肢があります。アプリケーションの認証機能を手動で実装するか、Laravel Fortifyを使用してこれらの機能のバックエンド実装を提供するかです。

Fortifyをインストールすることを選択した場合、ユーザーインターフェースは、このドキュメントに詳述されているFortifyの認証ルートにリクエストを送信して、ユーザーを認証および登録します。

代わりにLaravelの認証サービスを手動で操作することを選択した場合、[認証](authentication.md)、[パスワードリセット](passwords.md)、[メール確認](verification.md)のドキュメントに従って行うことができます。

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel FortifyとLaravel Sanctum

一部の開発者は、[Laravel Sanctum](sanctum.md)とLaravel Fortifyの違いについて混乱することがあります。2つのパッケージは2つの異なるが関連する問題を解決するため、Laravel FortifyとLaravel Sanctumは相互に排他的でも競合するパッケージでもありません。

Laravel Sanctumは、APIトークンの管理と、セッションクッキーまたはトークンを使用した既存ユーザーの認証にのみ関心があります。Sanctumは、ユーザー登録、パスワードリセットなどを処理するルートを提供しません。

APIを提供するアプリケーションや、シングルページアプリケーションのバックエンドとして機能するアプリケーションの認証層を手動で構築しようとしている場合、Laravel Fortify（ユーザー登録、パスワードリセットなど）とLaravel Sanctum（APIトークン管理、セッション認証）の両方を利用することが完全に可能です。

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを使用してFortifyをインストールします。

```shell
composer require laravel/fortify
```

次に、`fortify:install` Artisanコマンドを使用してFortifyのリソースを公開します。

```shell
php artisan fortify:install
```

このコマンドは、Fortifyのアクションを`app/Actions`ディレクトリに公開します。このディレクトリは存在しない場合に作成されます。さらに、`FortifyServiceProvider`、設定ファイル、および必要なすべてのデータベースマイグレーションが公開されます。

次に、データベースをマイグレートする必要があります。

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Fortifyの機能

`fortify`設定ファイルには、`features`設定配列が含まれています。この配列は、デフォルトでFortifyが公開するバックエンドルート/機能を定義します。[Laravel Jetstream](https://jetstream.laravel.com)と組み合わせてFortifyを使用していない場合、ほとんどのLaravelアプリケーションによって提供される基本的な認証機能のみを有効にすることをお勧めします。

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### ビューの無効化

デフォルトでは、Fortifyはログイン画面や登録画面などのビューを返すことを意図したルートを定義します。ただし、JavaScript駆動のシングルページアプリケーションを構築している場合、これらのルートは必要ないかもしれません。そのため、アプリケーションの`config/fortify.php`設定ファイル内の`views`設定値を`false`に設定することで、これらのルートを完全に無効にできます。

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### ビューの無効化とパスワードリセット

Fortifyのビューを無効にし、アプリケーションのパスワードリセット機能を実装する場合でも、アプリケーションの「パスワードリセット」ビューを表示するために`password.reset`という名前のルートを定義する必要があります。これは、Laravelの`Illuminate\Auth\Notifications\ResetPassword`通知が`password.reset`という名前のルートを介してパスワードリセットURLを生成するために必要です。

<a name="authentication"></a>
## 認証

まず、Fortifyに「ログイン」ビューを返す方法を指示する必要があります。Fortifyはヘッドレス認証ライブラリであることを思い出してください。Laravelの認証機能のフロントエンド実装がすでに完了している場合は、[アプリケーションスターターキット](starter-kits.md)を使用することをお勧めします。

すべての認証ビューのレンダリングロジックは、`Laravel\Fortify\Fortify`クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。Fortifyは、このビューを返す`/login`ルートの定義を処理します。

```php
use Laravel\Fortify\Fortify;

/**
 * 任意のアプリケーションサービスのブートストラップ
 */
public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

ログインテンプレートには、`/login`にPOSTリクエストを送信するフォームを含める必要があります。`/login`エンドポイントは、文字列の`email` / `username`と`password`を期待します。メール/ユーザー名フィールドの名前は、`config/fortify.php`設定ファイル内の`username`値と一致する必要があります。さらに、ユーザーが「ログイン状態を維持」機能を使用したいことを示すために、ブール値の`remember`フィールドを提供できます。

ログイン試行が成功した場合、Fortifyはアプリケーションの`fortify`設定ファイル内の`home`設定オプションで設定されたURIにリダイレクトします。ログインリクエストがXHRリクエストであった場合、200 HTTPレスポンスが返されます。

リクエストが成功しなかった場合、ユーザーはログイン画面にリダイレクトされ、バリデーションエラーは共有された`$errors` [Bladeテンプレート変数](validation.md#quick-displaying-the-validation-errors)を通じて利用可能になります。または、XHRリクエストの場合、バリデーションエラーは422 HTTPレスポンスとともに返されます。

<a name="customizing-user-authentication"></a>
### ユーザー認証のカスタマイズ

Fortifyは、提供された資格情報とアプリケーションに設定された認証ガードに基づいて、自動的にユーザーを取得して認証します。しかし、ログイン資格情報の認証方法とユーザーの取得方法を完全にカスタマイズしたい場合があります。幸いなことに、Fortifyは`Fortify::authenticateUsing`メソッドを使用してこれを簡単に実現できます。

このメソッドは、受信したHTTPリクエストを受け取るクロージャを受け取ります。このクロージャは、リクエストに添付されたログイン資格情報を検証し、関連するユーザーインスタンスを返す責任があります。資格情報が無効であるか、ユーザーが見つからない場合、クロージャは`null`または`false`を返すべきです。通常、このメソッドは`FortifyServiceProvider`の`boot`メソッドから呼び出すべきです。

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

<a name="authentication-guard"></a>
#### 認証ガード

アプリケーションの`fortify`設定ファイル内でFortifyが使用する認証ガードをカスタマイズできます。ただし、設定されたガードが`Illuminate\Contracts\Auth\StatefulGuard`の実装であることを確認する必要があります。Laravel Fortifyを使用してSPAを認証しようとしている場合、Laravelのデフォルトの`web`ガードを[Laravel Sanctum](https://laravel.com/docs/sanctum)と組み合わせて使用する必要があります。

<a name="customizing-the-authentication-pipeline"></a>
### 認証パイプラインのカスタマイズ

Laravel Fortifyは、呼び出し可能なクラスのパイプラインを通じてログインリクエストを認証します。必要に応じて、ログインリクエストがパイプされるカスタムパイプラインを定義できます。各クラスは、受信した`Illuminate\Http\Request`インスタンスと、[ミドルウェア](middleware.md)のように、リクエストをパイプライン内の次のクラスに渡すために呼び出される`$next`変数を受け取る`__invoke`メソッドを持つ必要があります。

カスタムパイプラインを定義するには、`Fortify::authenticateThrough`メソッドを使用できます。このメソッドは、ログインリクエストをパイプするクラスの配列を返すクロージャを受け取ります。通常、このメソッドは`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出すべきです。

以下の例には、独自の変更を加える際の出発点として使用できるデフォルトのパイプライン定義が含まれています。

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\CanonicalizeUsername;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Features;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            config('fortify.lowercase_usernames') ? CanonicalizeUsername::class : null,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

#### 認証のスロットリング

デフォルトでは、Fortifyは`EnsureLoginIsNotThrottled`ミドルウェアを使用して認証試行をスロットリングします。このミドルウェアは、ユーザー名とIPアドレスの組み合わせに固有の試行をスロットリングします。

一部のアプリケーションでは、IPアドレスのみでスロットリングするなど、認証試行のスロットリング方法を変更する必要がある場合があります。したがって、Fortifyは`fortify.limiters.login`設定オプションを介して独自の[レートリミッタ](routing.md#rate-limiting)を指定できます。もちろん、この設定オプションはアプリケーションの`config/fortify.php`設定ファイルにあります。

> NOTE:  
> スロットリング、[二要素認証](fortify.md#two-factor-authentication)、および外部のWebアプリケーションファイアウォール（WAF）を組み合わせて使用することで、正当なアプリケーションユーザーに対して最も堅牢な防御が提供されます。

<a name="customizing-authentication-redirects"></a>
### リダイレクトのカスタマイズ

ログイン試行が成功した場合、Fortifyはアプリケーションの`fortify`設定ファイル内の`home`設定オプションで設定されたURIにリダイレクトします。ログインリクエストがXHRリクエストであった場合、200 HTTPレスポンスが返されます。ユーザーがアプリケーションからログアウトした後、ユーザーは`/` URIにリダイレクトされます。

この動作を高度にカスタマイズする必要がある場合、`LoginResponse`および`LogoutResponse`契約の実装をLaravelの[サービスコンテナ](container.md)にバインドできます。通常、これはアプリケーションの`App\Providers\FortifyServiceProvider`クラスの`register`メソッド内で行うべきです。

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse($request)
        {
            return redirect('/');
        }
    });
}
```

<a name="two-factor-authentication"></a>
## 二要素認証

Fortifyの二要素認証機能が有効になっている場合、ユーザーは認証プロセス中に6桁の数値トークンを入力する必要があります。このトークンは、Google AuthenticatorなどのTOTP互換のモバイル認証アプリケーションを使用して取得できる時間ベースのワンタイムパスワード（TOTP）を使用して生成されます。

始める前に、まずアプリケーションの`App\Models\User`モデルが`Laravel\Fortify\TwoFactorAuthenticatable`トレイトを使用していることを確認する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
 ```

次に、ユーザーが二要素認証設定を管理できる画面をアプリケーション内に構築する必要があります。この画面では、ユーザーが二要素認証を有効または無効にしたり、二要素認証のリカバリーコードを再生成したりできるようにする必要があります。

> デフォルトでは、`fortify`設定ファイルの`features`配列は、二要素認証設定の変更前にパスワードの確認を要求するようにFortifyの二要素認証設定を指示します。したがって、アプリケーションは続行する前にFortifyの[パスワード確認](#password-confirmation)機能を実装する必要があります。

<a name="enabling-two-factor-authentication"></a>
### 二要素認証の有効化

二要素認証の有効化を開始するには、アプリケーションはFortifyによって定義された`/user/two-factor-authentication`エンドポイントにPOSTリクエストを行う必要があります。リクエストが成功した場合、ユーザーは前のURLにリダイレクトされ、`status`セッション変数が`two-factor-authentication-enabled`に設定されます。この`status`セッション変数をテンプレート内で検出して、適切な成功メッセージを表示できます。リクエストがXHRリクエストであった場合、`200` HTTPレスポンスが返されます。

二要素認証を有効にすることを選択した後、ユーザーはまだ二要素認証設定を「確認」する必要があります。そのため、「成功」メッセージには、二要素認証の確認がまだ必要であることをユーザーに通知する必要があります。

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        二要素認証の設定を完了してください。
    </div>
@endif
```

次に、ユーザーが認証アプリケーションにスキャンするための二要素認証QRコードを表示する必要があります。Bladeを使用してアプリケーションのフロントエンドをレンダリングしている場合、ユーザーインスタンスで利用可能な`twoFactorQrCodeSvg`メソッドを使用してQRコードSVGを取得できます。

```php
$request->user()->twoFactorQrCodeSvg();
```

JavaScriptを使用したフロントエンドを構築している場合、`/user/two-factor-qr-code`エンドポイントにXHR GETリクエストを行って、ユーザーの二要素認証QRコードを取得できます。このエンドポイントは、`svg`キーを含むJSONオブジェクトを返します。

<a name="confirming-two-factor-authentication"></a>
#### 二要素認証の確認

ユーザーの二要素認証QRコードを表示するだけでなく、ユーザーが有効な認証コードを入力して二要素認証設定を「確認」できるテキスト入力を提供する必要があります。このコードは、Fortifyによって定義された`/user/confirmed-two-factor-authentication`エンドポイントにPOSTリクエストを介してLaravelアプリケーションに提供される必要があります。

リクエストが成功した場合、ユーザーは前のURLにリダイレクトされ、`status`セッション変数が`two-factor-authentication-confirmed`に設定されます。

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        二要素認証が確認され、正常に有効化されました。
    </div>
@endif
```

二要素認証確認エンドポイントへのリクエストがXHRリクエストを介して行われた場合、`200` HTTPレスポンスが返されます。

<a name="displaying-the-recovery-codes"></a>
#### リカバリーコードの表示

ユーザーの二要素リカバリーコードも表示する必要があります。これらのリカバリーコードにより、ユーザーはモバイルデバイスにアクセスできなくなった場合でも認証できます。Bladeを使用してアプリケーションのフロントエンドをレンダリングしている場合、認証済みユーザーインスタンスを介してリカバリーコードにアクセスできます。

```php
(array) $request->user()->recoveryCodes()
```

JavaScriptを使用したフロントエンドを構築している場合、`/user/two-factor-recovery-codes`エンドポイントにXHR GETリクエストを行うことができます。このエンドポイントは、ユーザーのリカバリーコードを含むJSON配列を返します。

ユーザーのリカバリーコードを再生成するには、アプリケーションは`/user/two-factor-recovery-codes`エンドポイントにPOSTリクエストを行う必要があります。

<a name="authenticating-with-two-factor-authentication"></a>
### 二要素認証による認証

認証プロセス中、Fortifyは自動的にユーザーをアプリケーションの二要素認証チャレンジ画面にリダイレクトします。ただし、アプリケーションがXHRログインリクエストを行っている場合、認証試行が成功した後に返されるJSONレスポンスには、`two_factor`ブール値プロパティを持つJSONオブジェクトが含まれます。この値を調べて、アプリケーションの二要素認証チャレンジ画面にリダイレクトする必要があるかどうかを判断する必要があります。

二要素認証機能の実装を開始するには、Fortifyに二要素認証チャレンジビューを返す方法を指示する必要があります。Fortifyのすべての認証ビューのレンダリングロジックは、`Laravel\Fortify\Fortify`クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortifyは、このビューを返す`/two-factor-challenge`ルートを定義します。`two-factor-challenge`テンプレートには、`/two-factor-challenge`エンドポイントにPOSTリクエストを行うフォームを含める必要があります。`/two-factor-challenge`アクションは、有効なTOTPトークンを含む`code`フィールドまたはユーザーのリカバリーコードの1つを含む`recovery_code`フィールドを期待します。

ログイン試行が成功した場合、Fortifyはユーザーをアプリケーションの`fortify`設定ファイル内の`home`設定オプションを介して設定されたURIにリダイレクトします。ログインリクエストがXHRリクエストであった場合、`204` HTTPレスポンスが返されます。

リクエストが成功しなかった場合、ユーザーは二要素認証チャレンジ画面にリダイレクトされ、検証エラーは共有された`$errors` [Bladeテンプレート変数](validation.md#quick-displaying-the-validation-errors)を介して利用できます。または、XHRリクエストの場合、検証エラーは`422` HTTPレスポンスとともに返されます。

<a name="disabling-two-factor-authentication"></a>
### 二要素認証の無効化

二要素認証を無効にするには、アプリケーションは`/user/two-factor-authentication`エンドポイントにDELETEリクエストを行う必要があります。Fortifyの二要素認証エンドポイントは、呼び出される前に[パスワード確認](#password-confirmation)を必要とすることを忘れないでください。

<a name="registration"></a>
## 登録

アプリケーションの登録機能の実装を開始するには、Fortifyに「登録」ビューを返す方法を指示する必要があります。Fortifyはヘッドレス認証ライブラリであることを忘れないでください。Laravelの認証機能のフロントエンド実装がすでに完了している場合は、[アプリケーションスターターキット](starter-kits.md)を使用する必要があります。

Fortifyのすべてのビューのレンダリングロジックは、`Laravel\Fortify\Fortify`クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドは`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortifyは、このビューを返す`/register`ルートを定義します。`register`テンプレートには、Fortifyによって定義された`/register`エンドポイントにPOSTリクエストを行うフォームを含める必要があります。

`/register`エンドポイントは、文字列の`name`、文字列のメールアドレス/ユーザー名、`password`、および`password_confirmation`フィールドを期待します。メール/ユーザー名フィールドの名前は、アプリケーションの`fortify`設定ファイル内で定義された`username`設定値と一致する必要があります。

登録試行が成功した場合、Fortifyはユーザーをアプリケーションの`fortify`設定ファイル内の`home`設定オプションを介して設定されたURIにリダイレクトします。リクエストがXHRリクエストであった場合、`201` HTTPレスポンスが返されます。

リクエストが成功しなかった場合、ユーザーは登録画面にリダイレクトされ、検証エラーは共有された`$errors` [Bladeテンプレート変数](validation.md#quick-displaying-the-validation-errors)を介して利用できます。または、XHRリクエストの場合、検証エラーは`422` HTTPレスポンスとともに返されます。

<a name="customizing-registration"></a>
### 登録のカスタマイズ

ユーザーの検証と作成プロセスは、Laravel Fortifyをインストールしたときに生成された`App\Actions\Fortify\CreateNewUser`アクションを変更することでカスタマイズできます。

<a name="password-reset"></a>
## パスワードリセット

<a name="requesting-a-password-reset-link"></a>
### パスワードリセットリンクのリクエスト

アプリケーションのパスワードリセット機能の実装を開始するには、Fortifyに「パスワードを忘れた」ビューを返す方法を指示する必要があります。Fortifyはヘッドレス認証ライブラリであることを忘れないでください。Laravelの認証機能のフロントエンド実装がすでに完了している場合は、[アプリケーションスターターキット](starter-kits.md)を使用する必要があります。

Fortifyのすべてのビューのレンダリングロジックは、`Laravel\Fortify\Fortify`クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortifyは、このビューを返す`/forgot-password`エンドポイントを定義します。`forgot-password`テンプレートには、`/forgot-password`エンドポイントにPOSTリクエストを行うフォームを含める必要があります。

`/forgot-password`エンドポイントは、文字列の`email`フィールドを期待します。このフィールド/データベースカラムの名前は、アプリケーションの`fortify`設定ファイル内で定義された`email`設定値と一致する必要があります。

<a name="handling-the-password-reset-link-request-response"></a>
#### パスワードリセットリンクリクエストのレスポンスの処理

パスワードリセットリンクのリクエストが成功した場合、Fortifyはユーザーを`/forgot-password`エンドポイントにリダイレクトし、ユーザーにパスワードをリセットするための安全なリンクをメールで送信します。リクエストがXHRリクエストであった場合、`200` HTTPレスポンスが返されます。

リクエストが成功した後に`/forgot-password`エンドポイントにリダイレクトされた後、`status`セッション変数を使用してパスワードリセットリンクリクエストの試行のステータスを表示できます。

`$status`セッション変数の値は、アプリケーションの`passwords` [言語ファイル](localization.md)内で定義された翻訳文字列の1つと一致します。この値をカスタマイズしたい場合、Laravelの言語ファイルを公開していない場合は、`lang:publish` Artisanコマンドを介して公開できます。

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

リクエストが成功しなかった場合、ユーザーはパスワードリセットリンクリクエスト画面にリダイレクトされ、検証エラーは共有された`$errors` [Bladeテンプレート変数](validation.md#quick-displaying-the-validation-errors)を介して利用できます。または、XHRリクエストの場合、検証エラーは`422` HTTPレスポンスとともに返されます。

<a name="resetting-the-password"></a>
### パスワードのリセット

アプリケーションのパスワードリセット機能の実装を完了するには、Fortifyに「パスワードリセット」ビューを返す方法を指示する必要があります。

Fortifyのすべてのビューのレンダリングロジックは、`Laravel\Fortify\Fortify`クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの`App\Providers\FortifyServiceProvider`クラスの`boot`メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify は、このビューを表示するためのルートを定義する役割を担います。`reset-password` テンプレートには、`/reset-password` への POST リクエストを行うフォームを含める必要があります。

`/reset-password` エンドポイントは、文字列 `email` フィールド、`password` フィールド、`password_confirmation` フィールド、および `request()->route('token')` の値を含む `token` という名前の非表示フィールドを期待しています。「email」フィールド/データベースカラムの名前は、アプリケーションの `fortify` 設定ファイル内で定義された `email` 設定値と一致する必要があります。

<a name="handling-the-password-reset-response"></a>
#### パスワードリセットレスポンスの処理

パスワードリセットリクエストが成功した場合、Fortify はユーザーが新しいパスワードでログインできるように、`/login` ルートにリダイレクトします。さらに、`status` セッション変数が設定され、ログイン画面でリセットの成功ステータスを表示できるようになります。

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

リクエストが XHR リクエストだった場合、200 HTTP レスポンスが返されます。

リクエストが成功しなかった場合、ユーザーはパスワードリセット画面にリダイレクトされ、バリデーションエラーは共有された `$errors` [Blade テンプレート変数](validation.md#quick-displaying-the-validation-errors)を介して利用できます。または、XHR リクエストの場合、バリデーションエラーは 422 HTTP レスポンスとともに返されます。

<a name="customizing-password-resets"></a>
### パスワードリセットのカスタマイズ

パスワードリセットプロセスは、Laravel Fortify をインストールしたときに生成された `App\Actions\ResetUserPassword` アクションを変更することでカスタマイズできます。

<a name="email-verification"></a>
## メール検証

登録後、ユーザーがアプリケーションにアクセスし続ける前にメールアドレスを検証することを希望する場合があります。まず、`fortify` 設定ファイルの `features` 配列で `emailVerification` 機能が有効になっていることを確認してください。次に、`App\Models\User` クラスが `Illuminate\Contracts\Auth\MustVerifyEmail` インターフェースを実装していることを確認してください。

これらの2つのセットアップ手順が完了すると、新しく登録されたユーザーには、メールアドレスの所有権を検証するためのメールが送信されます。ただし、ユーザーにメール内の検証リンクをクリックするように指示するメール検証画面を Fortify に表示させる方法を教える必要があります。

Fortify のすべてのビューのレンダリングロジックは、`Laravel\Fortify\Fortify` クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの `App\Providers\FortifyServiceProvider` クラスの `boot` メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}
```

Fortify は、ユーザーが Laravel の組み込み `verified` ミドルウェアによって `/email/verify` エンドポイントにリダイレクトされたときに、このビューを表示するルートを定義します。

`verify-email` テンプレートには、ユーザーにメールアドレスに送信されたメール検証リンクをクリックするように指示する情報メッセージを含める必要があります。

<a name="resending-email-verification-links"></a>
#### メール検証リンクの再送信

必要に応じて、アプリケーションの `verify-email` テンプレートにボタンを追加して、`/email/verification-notification` エンドポイントへの POST リクエストをトリガーすることができます。このエンドポイントがリクエストを受け取ると、新しい検証メールリンクがユーザーにメールで送信され、ユーザーが前のリンクを誤って削除または紛失した場合に新しい検証リンクを取得できるようになります。

検証リンクのメール再送信リクエストが成功した場合、Fortify はユーザーを `/email/verify` エンドポイントにリダイレクトし、`status` セッション変数を設定して、操作が成功したことをユーザーに通知する情報メッセージを表示できるようにします。リクエストが XHR リクエストだった場合、202 HTTP レスポンスが返されます。

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

<a name="protecting-routes"></a>
### ルートの保護

ユーザーがメールアドレスを検証していることを要求するルートまたはルートグループを指定するには、Laravel の組み込み `verified` ミドルウェアをルートにアタッチする必要があります。`verified` ミドルウェアのエイリアスは、Laravel によって自動的に登録され、`Illuminate\Auth\Middleware\EnsureEmailIsVerified` ミドルウェアのエイリアスとして機能します。

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## パスワード確認

アプリケーションを構築する際、アクションを実行する前にユーザーがパスワードを確認する必要がある場合があります。通常、これらのルートは Laravel の組み込み `password.confirm` ミドルウェアによって保護されます。

パスワード確認機能の実装を開始するには、Fortify にアプリケーションの「パスワード確認」ビューを返す方法を指示する必要があります。Fortify はヘッドレス認証ライブラリであることを忘れないでください。Laravel の認証機能のフロントエンド実装が既に完了している場合は、[アプリケーションスターターキット](starter-kits.md)を使用する必要があります。

Fortify のすべてのビューのレンダリングロジックは、`Laravel\Fortify\Fortify` クラスを介して利用可能な適切なメソッドを使用してカスタマイズできます。通常、このメソッドはアプリケーションの `App\Providers\FortifyServiceProvider` クラスの `boot` メソッドから呼び出す必要があります。

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify は、このビューを返す `/user/confirm-password` エンドポイントを定義します。`confirm-password` テンプレートには、`/user/confirm-password` エンドポイントへの POST リクエストを行うフォームを含める必要があります。`/user/confirm-password` エンドポイントは、ユーザーの現在のパスワードを含む `password` フィールドを期待しています。

パスワードがユーザーの現在のパスワードと一致する場合、Fortify はユーザーをアクセスしようとしていたルートにリダイレクトします。リクエストが XHR リクエストだった場合、201 HTTP レスポンスが返されます。

リクエストが成功しなかった場合、ユーザーはパスワード確認画面にリダイレクトされ、バリデーションエラーは共有された `$errors` Blade テンプレート変数を介して利用できます。または、XHR リクエストの場合、バリデーションエラーは 422 HTTP レスポンスとともに返されます。

