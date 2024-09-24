# Laravel Sanctum

- [はじめに](#introduction)
    - [仕組み](#how-it-works)
- [インストール](#installation)
- [設定](#configuration)
    - [デフォルトモデルのオーバーライド](#overriding-default-models)
- [APIトークン認証](#api-token-authentication)
    - [APIトークンの発行](#issuing-api-tokens)
    - [トークンの権限](#token-abilities)
    - [ルートの保護](#protecting-routes)
    - [トークンの失効](#revoking-tokens)
    - [トークンの有効期限](#token-expiration)
- [SPA認証](#spa-authentication)
    - [設定](#spa-configuration)
    - [認証](#spa-authenticating)
    - [ルートの保護](#protecting-spa-routes)
    - [プライベートブロードキャストチャンネルの認可](#authorizing-private-broadcast-channels)
- [モバイルアプリケーション認証](#mobile-application-authentication)
    - [APIトークンの発行](#issuing-mobile-api-tokens)
    - [ルートの保護](#protecting-mobile-api-routes)
    - [トークンの失効](#revoking-mobile-api-tokens)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

[Laravel Sanctum](https://github.com/laravel/sanctum) は、SPA（シングルページアプリケーション）、モバイルアプリケーション、およびシンプルなトークンベースのAPIのための軽量認証システムを提供します。Sanctumを使用すると、アプリケーションの各ユーザーが自分のアカウントに対して複数のAPIトークンを生成できます。これらのトークンには、トークンが実行を許可されているアクションを指定する権限/スコープを付与できます。

<a name="how-it-works"></a>
### 仕組み

Laravel Sanctumは、2つの異なる問題を解決するために存在します。まずはそれぞれについて詳しく見ていきましょう。

<a name="how-it-works-api-tokens"></a>
#### APIトークン

まず、SanctumはOAuthの複雑さを避けてユーザーにAPIトークンを発行するために使用できるシンプルなパッケージです。この機能は、GitHubや他のアプリケーションが発行する「パーソナルアクセストークン」にインスパイアされています。例えば、アプリケーションの「アカウント設定」に、ユーザーが自分のアカウント用のAPIトークンを生成できる画面があるとします。Sanctumを使用してそれらのトークンを生成および管理できます。これらのトークンは通常、非常に長い有効期限（数年）を持ちますが、ユーザーはいつでも手動で失効させることができます。

Laravel Sanctumは、ユーザーAPIトークンを単一のデータベーステーブルに保存し、有効なAPIトークンを含む`Authorization`ヘッダーを介して受信HTTPリクエストを認証することで、この機能を提供します。

<a name="how-it-works-spa-authentication"></a>
#### SPA認証

次に、SanctumはLaravelを利用したAPIと通信する必要があるシングルページアプリケーション（SPA）を認証するためのシンプルな方法を提供します。これらのSPAは、Laravelアプリケーションと同じリポジトリに存在するか、Next.jsやNuxtなどを使用して作成された完全に別のリポジトリに存在する可能性があります。

この機能のために、Sanctumはどのようなトークンも使用しません。代わりに、Laravelの組み込みのクッキーベースのセッション認証サービスを使用します。通常、SanctumはLaravelの`web`認証ガードを使用してこれを実現します。これにより、CSRF保護、セッション認証、およびXSSを介した認証資格情報の漏洩を防ぐことができます。

Sanctumは、受信リクエストが自分のSPAフロントエンドから発信された場合にのみ、クッキーを使用して認証を試みます。Sanctumが受信HTTPリクエストを検査するとき、まず認証クッキーをチェックし、存在しない場合は`Authorization`ヘッダーで有効なAPIトークンをチェックします。

> NOTE:  
> SanctumはAPIトークン認証のみ、またはSPA認証のみを使用することが完全に問題ありません。Sanctumを使用することは、提供される両方の機能を使用する必要があることを意味するものではありません。

<a name="installation"></a>
## インストール

Laravel Sanctumは、`install:api` Artisanコマンドを介してインストールできます:

```shell
php artisan install:api
```

次に、Sanctumを使用してSPAを認証する予定の場合は、このドキュメントの[SPA認証](#spa-authentication)セクションを参照してください。

<a name="configuration"></a>
## 設定

<a name="overriding-default-models"></a>
### デフォルトモデルのオーバーライド

通常は必要ありませんが、Sanctumが内部的に使用する`PersonalAccessToken`モデルを自由に拡張できます:

    use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

    class PersonalAccessToken extends SanctumPersonalAccessToken
    {
        // ...
    }

そして、Sanctumにカスタムモデルを使用するよう指示するには、Sanctumが提供する`usePersonalAccessTokenModel`メソッドを使用します。通常、このメソッドはアプリケーションの`AppServiceProvider`ファイルの`boot`メソッド内で呼び出す必要があります:

    use App\Models\Sanctum\PersonalAccessToken;
    use Laravel\Sanctum\Sanctum;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
    }

<a name="api-token-authentication"></a>
## APIトークン認証

> NOTE:  
> 自分のファーストパーティSPAを認証するためにAPIトークンを使用しないでください。代わりに、Sanctumの組み込みの[SPA認証機能](#spa-authentication)を使用してください。

<a name="issuing-api-tokens"></a>
### APIトークンの発行

Sanctumを使用すると、APIリクエストを認証するために使用できるAPIトークン/パーソナルアクセストークンを発行できます。APIトークンを使用してリクエストを行う場合、トークンは`Authorization`ヘッダーに`Bearer`トークンとして含める必要があります。

ユーザーに対してトークンの発行を開始するには、Userモデルに`Laravel\Sanctum\HasApiTokens`トレイトを使用する必要があります:

    use Laravel\Sanctum\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

トークンを発行するには、`createToken`メソッドを使用できます。`createToken`メソッドは、`Laravel\Sanctum\NewAccessToken`インスタンスを返します。APIトークンは、データベースに保存される前にSHA-256ハッシュを使用してハッシュ化されますが、`NewAccessToken`インスタンスの`plainTextToken`プロパティを使用してトークンの平文値にアクセスできます。トークンが作成された直後にこの値をユーザーに表示する必要があります:

    use Illuminate\Http\Request;

    Route::post('/tokens/create', function (Request $request) {
        $token = $request->user()->createToken($request->token_name);

        return ['token' => $token->plainTextToken];
    });

`HasApiTokens`トレイトによって提供される`tokens` Eloquentリレーションを使用して、ユーザーのすべてのトークンにアクセスできます:

    foreach ($user->tokens as $token) {
        // ...
    }

<a name="token-abilities"></a>
### トークンの権限

Sanctumを使用すると、トークンに「権限」を割り当てることができます。権限は、OAuthの「スコープ」と同様の目的を果たします。`createToken`メソッドに対する第2引数として、文字列の権限の配列を渡すことができます:

    return $user->createToken('token-name', ['server:update'])->plainTextToken;

Sanctumによって認証された受信リクエストを処理する場合、`tokenCan`メソッドを使用してトークンが特定の権限を持っているかどうかを判断できます:

    if ($user->tokenCan('server:update')) {
        // ...
    }

<a name="token-ability-middleware"></a>
#### トークン権限ミドルウェア

Sanctumには、受信リクエストが特定の権限を持つトークンで認証されているかどうかを確認するために使用できる2つのミドルウェアが含まれています。まず、アプリケーションの`bootstrap/app.php`ファイルで以下のミドルウェアエイリアスを定義します:

    use Laravel\Sanctum\Http\Middleware\CheckAbilities;
    use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'abilities' => CheckAbilities::class,
            'ability' => CheckForAnyAbility::class,
        ]);
    })

`abilities`ミドルウェアは、受信リクエストのトークンがリストされたすべての権限を持っているかどうかを確認するためにルートに割り当てることができます:

    Route::get('/orders', function () {
        // トークンは「check-status」と「place-orders」の両方の権限を持っています...
    })->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);

`ability`ミドルウェアは、受信リクエストのトークンがリストされた権限の*少なくとも1つ*を持っているかどうかを確認するためにルートに割り当てることができます:

    Route::get('/orders', function () {
        // トークンは「check-status」または「place-orders」の権限を持っています...
    })->middleware(['auth:sanctum', 'ability:check-status,place-orders']);

<a name="first-party-ui-initiated-requests"></a>
#### ファーストパーティUIからのリクエスト

便宜上、受信リクエストが自分のファーストパーティSPAからのものであり、Sanctumの組み込みの[SPA認証](#spa-authentication)を使用している場合、`tokenCan`メソッドは常に`true`を返します。

ただし、これは必ずしもアプリケーションがユーザーにアクションを実行することを許可する必要があることを意味するわけではありません。通常、アプリケーションの[認可ポリシー](authorization.md#creating-policies)は、トークンが権限を実行するための許可を与えられているかどうか、およびユーザーインスタンス自体がアクションを実行することを許可されているかどうかを確認します。

例えば、サーバーを管理するアプリケーションを想像してみましょう。これは、トークンがサーバーを更新するための許可を持っているかどうか、およびサーバーがユーザーに属しているかどうかを確認することを意味します:

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

最初、`tokenCan`メソッドが呼び出され、ファーストパーティUIからのリクエストに対して常に`true`を返すことは奇妙に思えるかもしれません。しかし、APIトークンが常に利用可能であり、`tokenCan`メソッドを介して検査できると仮定することは便利です。このアプローチを取ることで、アプリケーションの認可ポリシー内で常に`tokenCan`メソッドを呼び出すことができ、リクエストがアプリケーションのUIから発信されたか、APIのサードパーティ消費者によって開始されたかを気にする必要がなくなります。

<a name="protecting-routes"></a>
### ルートの保護

ルートを保護して、すべての受信リクエストが認証されるようにするには、`routes/web.php` および `routes/api.php` ルートファイル内の保護されたルートに `sanctum` 認証ガードをアタッチする必要があります。このガードは、受信リクエストがステートフルなクッキー認証リクエストとして認証されるか、サードパーティからのリクエストの場合は有効な API トークンヘッダーを含むことを確認します。

アプリケーションの `routes/web.php` ファイル内のルートを `sanctum` ガードを使用して認証することを推奨する理由について疑問に思われるかもしれません。Sanctum は、まず受信リクエストを Laravel の通常のセッション認証クッキーを使用して認証しようとします。そのクッキーが存在しない場合、Sanctum はリクエストの `Authorization` ヘッダーにあるトークンを使用して認証を試みます。さらに、Sanctum を使用してすべてのリクエストを認証することで、現在認証されているユーザーインスタンスで常に `tokenCan` メソッドを呼び出すことができます。

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-tokens"></a>
### トークンの失効

`Laravel\Sanctum\HasApiTokens` トレイトによって提供される `tokens` リレーションを使用して、データベースからトークンを削除することで、トークンを「失効」させることができます。

```php
// すべてのトークンを失効させる...
$user->tokens()->delete();

// 現在のリクエストを認証するために使用されたトークンを失効させる...
$request->user()->currentAccessToken()->delete();

// 特定のトークンを失効させる...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="token-expiration"></a>
### トークンの有効期限

デフォルトでは、Sanctum トークンには有効期限がなく、[トークンを失効させる](#revoking-tokens)ことによってのみ無効にすることができます。ただし、アプリケーションの API トークンに有効期限を設定したい場合は、アプリケーションの `sanctum` 設定ファイルで定義された `expiration` 設定オプションを介して設定できます。この設定オプションは、発行されたトークンが失効と見なされるまでの分数を定義します。

```php
'expiration' => 525600,
```

各トークンの有効期限を個別に指定したい場合は、`createToken` メソッドの第3引数として有効期限を指定することができます。

```php
return $user->createToken(
    'token-name', ['*'], now()->addWeek()
)->plainTextToken;
```

アプリケーションにトークンの有効期限を設定した場合、アプリケーションの失効したトークンを整理するために[タスクをスケジュール](scheduling.md)することもできます。幸いなことに、Sanctum にはこれを実現するために使用できる `sanctum:prune-expired` Artisan コマンドが含まれています。たとえば、失効してから少なくとも24時間経過したすべての失効したトークンデータベースレコードを削除するようにスケジュールされたタスクを設定できます。

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

<a name="spa-authentication"></a>
## SPA 認証

Sanctum は、Laravel で動作する API と通信する必要があるシングルページアプリケーション (SPA) を認証するための簡単な方法も提供します。これらの SPA は、Laravel アプリケーションと同じリポジトリに存在するか、まったく別のリポジトリに存在する可能性があります。

この機能のために、Sanctum はどのような種類のトークンも使用しません。代わりに、Sanctum は Laravel の組み込みのクッキーベースのセッション認証サービスを使用します。この認証方法は、CSRF 保護、セッション認証の利点を提供し、XSS による認証資格情報の漏洩を防ぎます。

> WARNING:  
> 認証するためには、SPA と API が同じトップレベルドメインを共有している必要があります。ただし、異なるサブドメインに配置されていても構いません。さらに、リクエストに `Accept: application/json` ヘッダーと `Referer` または `Origin` ヘッダーを送信するようにしてください。

<a name="spa-configuration"></a>
### 設定

<a name="configuring-your-first-party-domains"></a>
#### ファーストパーティドメインの設定

まず、SPA がリクエストを行うドメインを設定する必要があります。`sanctum` 設定ファイルの `stateful` 設定オプションを使用して、これらのドメインを設定できます。この設定は、Laravel のセッションクッキーを使用して「ステートフル」な認証を維持するドメインを決定します。

> WARNING:  
> ポート (`127.0.0.1:8000` など) を含む URL を介してアプリケーションにアクセスしている場合は、ドメインにポート番号を含めるようにしてください。

<a name="sanctum-middleware"></a>
#### Sanctum ミドルウェア

次に、Laravel に、SPA からの受信リクエストが Laravel のセッションクッキーを使用して認証できるように指示し、サードパーティまたはモバイルアプリケーションからのリクエストが API トークンを使用して認証できるようにする必要があります。これは、アプリケーションの `bootstrap/app.php` ファイルで `statefulApi` ミドルウェアメソッドを呼び出すことで簡単に実現できます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->statefulApi();
})
```

<a name="cors-and-cookies"></a>
#### CORS とクッキー

別のサブドメインで実行されている SPA からアプリケーションの認証に問題がある場合は、CORS (Cross-Origin Resource Sharing) またはセッションクッキーの設定が正しく行われていない可能性があります。

`config/cors.php` 設定ファイルはデフォルトでは公開されていません。Laravel の CORS オプションをカスタマイズする必要がある場合は、`config:publish` Artisan コマンドを使用して完全な `cors` 設定ファイルを公開する必要があります。

```bash
php artisan config:publish cors
```

次に、アプリケーションの CORS 設定が `Access-Control-Allow-Credentials` ヘッダーを `True` の値で返すようにする必要があります。これは、アプリケーションの `config/cors.php` 設定ファイル内の `supports_credentials` オプションを `true` に設定することで実現できます。

さらに、アプリケーションのグローバル `axios` インスタンスで `withCredentials` および `withXSRFToken` オプションを有効にする必要があります。通常、これは `resources/js/bootstrap.js` ファイルで実行されます。HTTP リクエストを行うために Axios を使用していない場合は、独自の HTTP クライアントで同等の設定を行う必要があります。

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

最後に、アプリケーションのセッションクッキードメイン設定がルートドメインの任意のサブドメインをサポートするようにする必要があります。これは、アプリケーションの `config/session.php` 設定ファイル内のドメインに先頭に `.` を付けることで実現できます。

```php
'domain' => '.domain.com',
```

<a name="spa-authenticating"></a>
### 認証

<a name="csrf-protection"></a>
#### CSRF 保護

SPA を認証するために、SPA の「ログイン」ページは最初に `/sanctum/csrf-cookie` エンドポイントにリクエストを送信して、アプリケーションの CSRF 保護を初期化する必要があります。

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // ログイン...
});
```

このリクエスト中に、Laravel は現在の CSRF トークンを含む `XSRF-TOKEN` クッキーを設定します。このトークンは、後続のリクエストで `X-XSRF-TOKEN` ヘッダーに渡される必要があります。Axios や Angular HttpClient などの一部の HTTP クライアントライブラリは、これを自動的に行います。JavaScript HTTP ライブラリがこの値を自動的に設定しない場合は、`X-XSRF-TOKEN` ヘッダーをこのルートによって設定された `XSRF-TOKEN` クッキーの値と一致するように手動で設定する必要があります。

<a name="logging-in"></a>
#### ログイン

CSRF 保護が初期化されたら、Laravel アプリケーションの `/login` ルートに `POST` リクエストを送信する必要があります。この `/login` ルートは、[手動で実装](authentication.md#authenticating-users)するか、[Laravel Fortify](fortify.md) などのヘッドレス認証パッケージを使用して実装できます。

ログインリクエストが成功した場合、認証され、後続のリクエストは Laravel アプリケーションがクライアントに発行したセッションクッキーを介して自動的に認証されます。さらに、アプリケーションがすでに `/sanctum/csrf-cookie` ルートにリクエストを送信しているため、後続のリクエストは JavaScript HTTP クライアントが `XSRF-TOKEN` クッキーの値を `X-XSRF-TOKEN` ヘッダーに送信する限り、自動的に CSRF 保護を受けます。

もちろん、ユーザーのセッションがアクティビティの不足により期限切れになった場合、後続のリクエストは Laravel アプリケーションから 401 または 419 HTTP エラー応答を受け取る可能性があります。この場合、ユーザーを SPA のログインページにリダイレクトする必要があります。

> WARNING:  
> 独自の `/login` エンドポイントを自由に記述できます。ただし、ユーザーを認証するために Laravel が提供する標準の[セッションベースの認証サービス](authentication.md#authenticating-users)を使用するようにしてください。通常、これは `web` 認証ガードを使用することを意味します。

<a name="protecting-spa-routes"></a>
### ルートの保護

すべての受信リクエストが認証されるようにルートを保護するには、`routes/api.php` ファイル内の API ルートに `sanctum` 認証ガードをアタッチする必要があります。このガードは、受信リクエストが SPA からのステートフルな認証リクエストとして認証されるか、サードパーティからのリクエストの場合は有効な API トークンヘッダーを含むことを確認します。

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="authorizing-private-broadcast-channels"></a>
### プライベートブロードキャストチャンネルの認可

もしSPAが[プライベート／プレゼンスブロードキャストチャンネル](broadcasting.md#authorizing-channels)で認証する必要がある場合、アプリケーションの`bootstrap/app.php`ファイルに含まれる`withRouting`メソッドから`channels`エントリを削除する必要があります。代わりに、`withBroadcasting`メソッドを呼び出して、アプリケーションのブロードキャストルートに適切なミドルウェアを指定する必要があります。

    return Application::configure(basePath: dirname(__DIR__))
        ->withRouting(
            web: __DIR__.'/../routes/web.php',
            // ...
        )
        ->withBroadcasting(
            __DIR__.'/../routes/channels.php',
            ['prefix' => 'api', 'middleware' => ['api', 'auth:sanctum']],
        )

次に、Pusherの認証リクエストが成功するためには、[Laravel Echo](broadcasting.md#client-side-installation)を初期化する際にカスタムPusher `authorizer`を提供する必要があります。これにより、アプリケーションはPusherを設定して、[クロスドメインリクエストに適切に設定された](#cors-and-cookies) `axios`インスタンスを使用するようにできます。

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})
```

<a name="mobile-application-authentication"></a>
## モバイルアプリケーション認証

Sanctumトークンを使用して、モバイルアプリケーションのAPIリクエストを認証することもできます。モバイルアプリケーションのリクエストを認証するプロセスは、サードパーティのAPIリクエストを認証するプロセスと似ていますが、APIトークンを発行する方法に若干の違いがあります。

<a name="issuing-mobile-api-tokens"></a>
### APIトークンの発行

まず、ユーザーのメールアドレス/ユーザー名、パスワード、およびデバイス名を受け取り、それらの資格情報を新しいSanctumトークンと交換するルートを作成します。このエンドポイントに与えられる「デバイス名」は情報提供のためのもので、任意の値を指定できます。一般的に、デバイス名の値はユーザーが認識できる名前であるべきです（例：「NunoのiPhone 12」）。

通常、モバイルアプリケーションの「ログイン」画面からトークンエンドポイントにリクエストを送信します。エンドポイントはプレーンテキストのAPIトークンを返し、それをモバイルデバイスに保存して、追加のAPIリクエストに使用できます。

    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Validation\ValidationException;

    Route::post('/sanctum/token', function (Request $request) {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
            'device_name' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (! $user || ! Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        return $user->createToken($request->device_name)->plainTextToken;
    });

モバイルアプリケーションがトークンを使用してAPIリクエストをアプリケーションに送信する場合、トークンを`Authorization`ヘッダーに`Bearer`トークンとして渡す必要があります。

> NOTE:  
> モバイルアプリケーションのトークンを発行する際には、[トークンの能力](#token-abilities)を指定することも自由です。

<a name="protecting-mobile-api-routes"></a>
### ルートの保護

前述の通り、`sanctum`認証ガードをルートにアタッチすることで、すべての受信リクエストが認証されるようにルートを保護できます。

    Route::get('/user', function (Request $request) {
        return $request->user();
    })->middleware('auth:sanctum');

<a name="revoking-mobile-api-tokens"></a>
### トークンの取り消し

ユーザーがモバイルデバイスに発行されたAPIトークンを取り消せるようにするには、WebアプリケーションのUIの「アカウント設定」部分に、名前と「取り消し」ボタンを一覧表示できます。ユーザーが「取り消し」ボタンをクリックすると、データベースからトークンを削除できます。`Laravel\Sanctum\HasApiTokens`トレイトによって提供される`tokens`リレーションを介して、ユーザーのAPIトークンにアクセスできることを覚えておいてください。

    // すべてのトークンを取り消す...
    $user->tokens()->delete();

    // 特定のトークンを取り消す...
    $user->tokens()->where('id', $tokenId)->delete();

<a name="testing"></a>
## テスト

テスト中に、`Sanctum::actingAs`メソッドを使用してユーザーを認証し、そのトークンに付与する能力を指定できます。

===  "Pest"
```php
use App\Models\User;
use Laravel\Sanctum\Sanctum;

test('task list can be retrieved', function () {
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
});
```

===  "PHPUnit"
```php
use App\Models\User;
use Laravel\Sanctum\Sanctum;

public function test_task_list_can_be_retrieved(): void
{
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
}
```

トークンにすべての能力を付与したい場合は、`actingAs`メソッドに提供する能力リストに`*`を含める必要があります。

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );
