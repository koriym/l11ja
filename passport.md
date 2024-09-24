# Laravel Passport

- [はじめに](#introduction)
    - [Passport か Sanctum か？](#passport-or-sanctum)
- [インストール](#installation)
    - [Passport のデプロイ](#deploying-passport)
    - [Passport のアップグレード](#upgrading-passport)
- [設定](#configuration)
    - [クライアントシークレットのハッシュ化](#client-secret-hashing)
    - [トークンの有効期限](#token-lifetimes)
    - [デフォルトモデルのオーバーライド](#overriding-default-models)
    - [ルートのオーバーライド](#overriding-routes)
- [アクセストークンの発行](#issuing-access-tokens)
    - [クライアントの管理](#managing-clients)
    - [トークンのリクエスト](#requesting-tokens)
    - [トークンのリフレッシュ](#refreshing-tokens)
    - [トークンの取り消し](#revoking-tokens)
    - [トークンのパージ](#purging-tokens)
- [PKCE を使用した認証コードグラント](#code-grant-pkce)
    - [クライアントの作成](#creating-a-auth-pkce-grant-client)
    - [トークンのリクエスト](#requesting-auth-pkce-grant-tokens)
- [パスワードグラントトークン](#password-grant-tokens)
    - [パスワードグラントクライアントの作成](#creating-a-password-grant-client)
    - [トークンのリクエスト](#requesting-password-grant-tokens)
    - [すべてのスコープのリクエスト](#requesting-all-scopes)
    - [ユーザープロバイダのカスタマイズ](#customizing-the-user-provider)
    - [ユーザー名フィールドのカスタマイズ](#customizing-the-username-field)
    - [パスワード検証のカスタマイズ](#customizing-the-password-validation)
- [暗黙的グラントトークン](#implicit-grant-tokens)
- [クライアントクレデンシャルグラントトークン](#client-credentials-grant-tokens)
- [パーソナルアクセストークン](#personal-access-tokens)
    - [パーソナルアクセスクライアントの作成](#creating-a-personal-access-client)
    - [パーソナルアクセストークンの管理](#managing-personal-access-tokens)
- [ルートの保護](#protecting-routes)
    - [ミドルウェアによる保護](#via-middleware)
    - [アクセストークンの渡し方](#passing-the-access-token)
- [トークンスコープ](#token-scopes)
    - [スコープの定義](#defining-scopes)
    - [デフォルトスコープ](#default-scope)
    - [トークンへのスコープの割り当て](#assigning-scopes-to-tokens)
    - [スコープのチェック](#checking-scopes)
- [JavaScript での API の利用](#consuming-your-api-with-javascript)
- [イベント](#events)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

[Laravel Passport](https://github.com/laravel/passport) は、Laravel アプリケーションに対して数分で完全な OAuth2 サーバー実装を提供します。Passport は、Andy Millington と Simon Hamp によってメンテナンスされている [League OAuth2 サーバー](https://github.com/thephpleague/oauth2-server) の上に構築されています。

> WARNING:  
> このドキュメントは、あなたがすでに OAuth2 に精通していることを前提としています。OAuth2 について何も知らない場合は、続行する前に OAuth2 の一般的な [用語](https://oauth2.thephpleague.com/terminology/) と機能について理解を深めることを検討してください。

<a name="passport-or-sanctum"></a>
### Passport か Sanctum か？

始める前に、アプリケーションが Laravel Passport と [Laravel Sanctum](sanctum.md) のどちらでよりよくサービスを受けられるかを判断することをお勧めします。アプリケーションが絶対に OAuth2 をサポートする必要がある場合は、Laravel Passport を使用する必要があります。

ただし、シングルページアプリケーション、モバイルアプリケーションを認証したり、API トークンを発行したりする場合は、[Laravel Sanctum](sanctum.md) を使用する必要があります。Laravel Sanctum は OAuth2 をサポートしていませんが、はるかにシンプルな API 認証開発体験を提供します。

<a name="installation"></a>
## インストール

Laravel Passport は、`install:api` Artisan コマンドを介してインストールできます:

```shell
php artisan install:api --passport
```

このコマンドは、アプリケーションが OAuth2 クライアントとアクセストークンを保存するために必要なテーブルを作成するために必要なデータベースマイグレーションを公開し、実行します。また、セキュアなアクセストークンを生成するために必要な暗号化キーも作成します。

さらに、このコマンドは、Passport の `Client` モデルの主キー値として UUID を使用するか、自動インクリメント整数を使用するかを尋ねます。

`install:api` コマンドを実行した後、`App\Models\User` モデルに `Laravel\Passport\HasApiTokens` トレイトを追加します。このトレイトは、認証済みユーザーのトークンとスコープを検査するためのいくつかのヘルパーメソッドをモデルに提供します:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Factories\HasFactory;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, HasFactory, Notifiable;
    }

最後に、アプリケーションの `config/auth.php` 設定ファイルで、`api` 認証ガードを定義し、`driver` オプションを `passport` に設定する必要があります。これにより、アプリケーションは受信 API リクエストを認証する際に Passport の `TokenGuard` を使用するように指示されます:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

<a name="deploying-passport"></a>
### Passport のデプロイ

Passport をアプリケーションのサーバーに初めてデプロイする際には、`passport:keys` コマンドを実行する必要があります。このコマンドは、Passport がアクセストークンを生成するために必要な暗号化キーを生成します。生成されたキーは通常、ソース管理には含まれません:

```shell
php artisan passport:keys
```

必要に応じて、Passport のキーをロードするパスを定義できます。これを行うには、`Passport::loadKeysFrom` メソッドを使用します。通常、このメソッドはアプリケーションの `App\Providers\AppServiceProvider` クラスの `boot` メソッドから呼び出す必要があります:

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

<a name="loading-keys-from-the-environment"></a>
#### 環境からのキーのロード

あるいは、`vendor:publish` Artisan コマンドを使用して Passport の設定ファイルを公開することもできます:

```shell
php artisan vendor:publish --tag=passport-config
```

設定ファイルが公開された後、環境変数としてアプリケーションの暗号化キーをロードできます:

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### Passport のアップグレード

新しいメジャーバージョンの Passport にアップグレードする際には、[アップグレードガイド](https://github.com/laravel/passport/blob/master/UPGRADE.md) を注意深く確認することが重要です。

<a name="configuration"></a>
## 設定

<a name="client-secret-hashing"></a>
### クライアントシークレットのハッシュ化

データベースに保存する際にクライアントのシークレットをハッシュ化したい場合は、`App\Providers\AppServiceProvider` クラスの `boot` メソッドで `Passport::hashClientSecrets` メソッドを呼び出す必要があります:

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

有効にすると、すべてのクライアントシークレットは、作成直後にのみユーザーに表示されるようになります。プレーンテキストのクライアントシークレット値はデータベースに保存されないため、シークレットの値が失われた場合、その値を回復することはできません。

<a name="token-lifetimes"></a>
### トークンの有効期限

デフォルトでは、Passport は1年間有効な長期間有効なアクセストークンを発行します。トークンの有効期限を長くまたは短く設定したい場合は、`tokensExpireIn`、`refreshTokensExpireIn`、`personalAccessTokensExpireIn` メソッドを使用できます。これらのメソッドは、アプリケーションの `App\Providers\AppServiceProvider` クラスの `boot` メソッドから呼び出す必要があります:

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> WARNING:  
> Passport のデータベーステーブルの `expires_at` カラムは読み取り専用で、表示目的のみで使用されます。トークンを発行する際、Passport は署名され暗号化されたトークン内に有効期限情報を保存します。トークンを無効にする必要がある場合は、[取り消し](#revoking-tokens) を行う必要があります。

<a name="overriding-default-models"></a>
### デフォルトモデルのオーバーライド

Passport が内部で使用するモデルを自由に拡張できます。独自のモデルを定義し、対応する Passport モデルを拡張することで行えます:

    use Laravel\Passport\Client as PassportClient;

    class Client extends PassportClient
    {
        // ...
    }

モデルを定義した後、`Laravel\Passport\Passport` クラスを介して Passport にカスタムモデルを使用するように指示できます。通常、これはアプリケーションの `App\Providers\AppServiceProvider` クラスの `boot` メソッドで行います:

    use App\Models\Passport\AuthCode;
    use App\Models\Passport\Client;
    use App\Models\Passport\PersonalAccessClient;
    use App\Models\Passport\RefreshToken;
    use App\Models\Passport\Token;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::useTokenModel(Token::class);
        Passport::useRefreshTokenModel(RefreshToken::class);
        Passport::useAuthCodeModel(AuthCode::class);
        Passport::useClientModel(Client::class);
        Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
    }

<a name="overriding-routes"></a>
### ルートのオーバーライド

Passport によって定義されたルートをカスタマイズしたい場合は、まずアプリケーションの `AppServiceProvider` の `register` メソッドに `Passport::ignoreRoutes` を追加して、Passport によって登録されたルートを無視する必要があります:

    use Laravel\Passport\Passport;

    /**
     * アプリケーションサービスの登録
     */
    public function register(): void
    {
        Passport::ignoreRoutes();
    }

その後、Passportが[そのルートファイル](https://github.com/laravel/passport/blob/11.x/routes/web.php)で定義したルートを、アプリケーションの`routes/web.php`ファイルにコピーし、好みに合わせて修正することができます。

    Route::group([
        'as' => 'passport.',
        'prefix' => config('passport.path', 'oauth'),
        'namespace' => '\Laravel\Passport\Http\Controllers',
    ], function () {
        // Passportのルート...
    });

<a name="issuing-access-tokens"></a>
## アクセストークンの発行

OAuth2を介した認証コードの使用は、ほとんどの開発者がOAuth2に精通している方法です。認証コードを使用する場合、クライアントアプリケーションはユーザーをサーバーにリダイレクトし、ユーザーはクライアントにアクセストークンを発行するリクエストを承認または拒否します。

<a name="managing-clients"></a>
### クライアントの管理

まず、アプリケーションのAPIと対話する必要があるアプリケーションを構築する開発者は、アプリケーションに登録する必要があります。これには通常、アプリケーションの名前と、ユーザーが認証リクエストを承認した後にリダイレクトされるURLを提供することが含まれます。

<a name="the-passportclient-command"></a>
#### `passport:client`コマンド

クライアントを作成する最も簡単な方法は、`passport:client` Artisanコマンドを使用することです。このコマンドは、OAuth2機能をテストするために独自のクライアントを作成するために使用できます。`client`コマンドを実行すると、Passportはクライアントに関する詳細情報を求め、クライアントIDとシークレットを提供します。

```shell
php artisan passport:client
```

**リダイレクトURL**

クライアントに複数のリダイレクトURLを許可したい場合は、`passport:client`コマンドでURLを求められたときにカンマ区切りのリストを指定できます。カンマを含むURLはURLエンコードする必要があります。

```shell
http://example.com/callback,http://examplefoo.com/callback
```

<a name="clients-json-api"></a>
#### JSON API

アプリケーションのユーザーは`client`コマンドを利用できないため、Passportはクライアントを作成するためのJSON APIを提供します。これにより、クライアントの作成、更新、削除のためのコントローラを手動でコーディングする手間が省けます。

ただし、PassportのJSON APIを独自のフロントエンドと組み合わせて、ユーザーがクライアントを管理できるダッシュボードを提供する必要があります。以下では、クライアントを管理するためのすべてのAPIエンドポイントを確認します。便宜上、エンドポイントへのHTTPリクエストを作成するために[Axios](https://github.com/axios/axios)を使用します。

JSON APIは`web`と`auth`ミドルウェアによって保護されています。したがって、独自のアプリケーションからのみ呼び出すことができます。外部ソースから呼び出すことはできません。

<a name="get-oauthclients"></a>
#### `GET /oauth/clients`

このルートは、認証されたユーザーのすべてのクライアントを返します。これは主に、ユーザーがクライアントを編集または削除できるように、ユーザーのすべてのクライアントをリストアップするために役立ちます。

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

このルートは新しいクライアントを作成するために使用されます。これには、クライアントの`name`と`redirect` URLの2つのデータが必要です。`redirect` URLは、ユーザーが認証リクエストを承認または拒否した後にリダイレクトされる場所です。

クライアントが作成されると、クライアントIDとクライアントシークレットが発行されます。これらの値は、アプリケーションからアクセストークンをリクエストする際に使用されます。クライアント作成ルートは、新しいクライアントインスタンスを返します。

```js
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // レスポンスのエラーをリストアップ...
    });
```

<a name="put-oauthclientsclient-id"></a>
#### `PUT /oauth/clients/{client-id}`

このルートはクライアントを更新するために使用されます。これには、クライアントの`name`と`redirect` URLの2つのデータが必要です。`redirect` URLは、ユーザーが認証リクエストを承認または拒否した後にリダイレクトされる場所です。ルートは更新されたクライアントインスタンスを返します。

```js
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // レスポンスのエラーをリストアップ...
    });
```

<a name="delete-oauthclientsclient-id"></a>
#### `DELETE /oauth/clients/{client-id}`

このルートはクライアントを削除するために使用されます。

```js
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        // ...
    });
```

<a name="requesting-tokens"></a>
### トークンのリクエスト

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### 認証のためのリダイレクト

クライアントが作成されると、開発者はクライアントIDとシークレットを使用して、アプリケーションから認証コードとアクセストークンをリクエストできます。まず、消費アプリケーションは次のようにアプリケーションの`/oauth/authorize`ルートにリダイレクトリクエストを行う必要があります。

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

`prompt`パラメータは、Passportアプリケーションの認証動作を指定するために使用できます。

`prompt`の値が`none`の場合、PassportはユーザーがPassportアプリケーションでまだ認証されていない場合、常に認証エラーをスローします。値が`consent`の場合、Passportは、すべてのスコープが以前に消費アプリケーションに付与されていたとしても、常に認証承認画面を表示します。値が`login`の場合、Passportアプリケーションは、ユーザーが既にセッションを持っている場合でも、常にユーザーにアプリケーションへの再ログインを促します。

`prompt`の値が提供されていない場合、ユーザーは、以前に消費アプリケーションへのアクセスを要求されたスコープに対して承認されていない場合にのみ、承認を求められます。

> NOTE:  
> `/oauth/authorize`ルートはPassportによって既に定義されていることに注意してください。このルートを手動で定義する必要はありません。

<a name="approving-the-request"></a>
#### リクエストの承認

認証リクエストを受け取ると、Passportは`prompt`パラメータの値（存在する場合）に基づいて自動的に応答し、ユーザーが認証リクエストを承認または拒否できるテンプレートを表示する場合があります。リクエストが承認された場合、ユーザーは消費アプリケーションによって指定された`redirect_uri`にリダイレクトされます。`redirect_uri`は、クライアントが作成されたときに指定された`redirect` URLと一致する必要があります。

認証承認画面をカスタマイズしたい場合は、`vendor:publish` Artisanコマンドを使用してPassportのビューを公開できます。公開されたビューは`resources/views/vendor/passport`ディレクトリに配置されます。

```shell
php artisan vendor:publish --tag=passport-views
```

認証プロンプトをスキップしたい場合があります。たとえば、ファーストパーティクライアントを承認する場合です。これは、[`Client`モデルを拡張](#overriding-default-models)し、`skipsAuthorization`メソッドを定義することで実現できます。`skipsAuthorization`が`true`を返す場合、クライアントは承認され、ユーザーは`redirect_uri`に即座にリダイレクトされます。ただし、消費アプリケーションが認証リダイレクト時に`prompt`パラメータを明示的に設定していない場合に限ります。

```php
<?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * クライアントが認証プロンプトをスキップするかどうかを判断します。
     */
    public function skipsAuthorization(): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### 認証コードをアクセストークンに変換

ユーザーが認証リクエストを承認した場合、ユーザーは消費アプリケーションにリダイレクトされます。消費者は最初に`state`パラメータをリダイレクト前に保存した値と照合する必要があります。`state`パラメータが一致する場合、消費者はアプリケーションに`POST`リクエストを発行してアクセストークンをリクエストする必要があります。リクエストには、ユーザーが認証リクエストを承認したときにアプリケーションによって発行された認証コードを含める必要があります。

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

この `/oauth/token` ルートは、`access_token`、`refresh_token`、および `expires_in` 属性を含む JSON レスポンスを返します。`expires_in` 属性には、アクセストークンの有効期限が切れるまでの秒数が含まれています。

> NOTE:  
> `/oauth/authorize` ルートと同様に、`/oauth/token` ルートは Passport によって定義されています。このルートを手動で定義する必要はありません。

<a name="tokens-json-api"></a>
#### JSON API

Passport には、認可されたアクセストークンを管理するための JSON API も含まれています。これを独自のフロントエンドと組み合わせて、ユーザーにアクセストークンを管理するためのダッシュボードを提供することができます。便宜上、エンドポイントへの HTTP リクエストを行うために [Axios](https://github.com/mzabriskie/axios) を使用します。JSON API は `web` および `auth` ミドルウェアによって保護されています。したがって、自分のアプリケーションからのみ呼び出すことができます。

<a name="get-oauthtokens"></a>
#### `GET /oauth/tokens`

このルートは、認証されたユーザーが作成したすべての認可されたアクセストークンを返します。これは主に、ユーザーのすべてのトークンをリストアップして、それらを取り消すことができるようにするために便利です：

```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="delete-oauthtokenstoken-id"></a>
#### `DELETE /oauth/tokens/{token-id}`

このルートは、認可されたアクセストークンとそれに関連するリフレッシュトークンを取り消すために使用できます：

```js
axios.delete('/oauth/tokens/' + tokenId);
```

<a name="refreshing-tokens"></a>
### トークンのリフレッシュ

アプリケーションが短期間のアクセストークンを発行する場合、ユーザーはアクセストークンが発行されたときに提供されたリフレッシュトークンを介してアクセストークンをリフレッシュする必要があります：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ]);

    return $response->json();

この `/oauth/token` ルートは、`access_token`、`refresh_token`、および `expires_in` 属性を含む JSON レスポンスを返します。`expires_in` 属性には、アクセストークンの有効期限が切れるまでの秒数が含まれています。

<a name="revoking-tokens"></a>
### トークンの取り消し

`Laravel\Passport\TokenRepository` の `revokeAccessToken` メソッドを使用してトークンを取り消すことができます。トークンのリフレッシュトークンは、`Laravel\Passport\RefreshTokenRepository` の `revokeRefreshTokensByAccessTokenId` メソッドを使用して取り消すことができます。これらのクラスは、Laravel の [サービスコンテナ](container.md) を使用して解決できます：

    use Laravel\Passport\TokenRepository;
    use Laravel\Passport\RefreshTokenRepository;

    $tokenRepository = app(TokenRepository::class);
    $refreshTokenRepository = app(RefreshTokenRepository::class);

    // アクセストークンを取り消す...
    $tokenRepository->revokeAccessToken($tokenId);

    // トークンのすべてのリフレッシュトークンを取り消す...
    $refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);

<a name="purging-tokens"></a>
### トークンのパージ

トークンが取り消されたり期限切れになったりした場合、データベースからそれらをパージしたい場合があります。Passport に含まれる `passport:purge` Artisan コマンドは、これを行うことができます：

```shell
# 取り消されたトークンと期限切れのトークン、および認証コードをパージする...
php artisan passport:purge

# 6時間以上経過した期限切れのトークンのみをパージする...
php artisan passport:purge --hours=6

# 取り消されたトークンと認証コードのみをパージする...
php artisan passport:purge --revoked

# 期限切れのトークンと認証コードのみをパージする...
php artisan passport:purge --expired
```

また、アプリケーションの `routes/console.php` ファイルに [スケジュールされたジョブ](scheduling.md) を設定して、トークンを自動的にパージすることもできます：

    use Laravel\Support\Facades\Schedule;

    Schedule::command('passport:purge')->hourly();

<a name="code-grant-pkce"></a>
## PKCE を使用した認証コード付与

"Proof Key for Code Exchange" (PKCE) を使用した認証コード付与は、シングルページアプリケーションやネイティブアプリケーションが API にアクセスするために認証するための安全な方法です。この付与は、クライアントシークレットが機密に保存されることを保証できない場合や、認証コードが攻撃者によって傍受される脅威を軽減するために使用する必要があります。"code verifier" と "code challenge" の組み合わせが、アクセストークンと交換する際のクライアントシークレットを置き換えます。

<a name="creating-a-auth-pkce-grant-client"></a>
### クライアントの作成

アプリケーションが PKCE を使用した認証コード付与を介してトークンを発行する前に、PKCE 対応のクライアントを作成する必要があります。これは、`passport:client` Artisan コマンドに `--public` オプションを付けて行うことができます：

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### トークンのリクエスト

<a name="code-verifier-code-challenge"></a>
#### コード検証子とコードチャレンジ

この認証付与はクライアントシークレットを提供しないため、開発者はトークンをリクエストするためにコード検証子とコードチャレンジの組み合わせを生成する必要があります。

コード検証子は、43 文字から 128 文字の間のランダムな文字列で、文字、数字、および `"-"`, `"."`, `"_"`, `"~"` 文字を含む必要があります。これは [RFC 7636 仕様](https://tools.ietf.org/html/rfc7636) で定義されています。

コードチャレンジは、URL とファイル名に安全な文字を含む Base64 エンコードされた文字列である必要があります。末尾の `'='` 文字は削除され、改行、空白、またはその他の追加文字は含まれてはなりません。

    $encoded = base64_encode(hash('sha256', $code_verifier, true));

    $codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### 認証のためのリダイレクト

クライアントが作成されたら、クライアント ID と生成されたコード検証子およびコードチャレンジを使用して、アプリケーションから認証コードとアクセストークンをリクエストできます。まず、消費アプリケーションはアプリケーションの `/oauth/authorize` ルートにリダイレクトリクエストを行う必要があります：

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $request->session()->put(
            'code_verifier', $code_verifier = Str::random(128)
        );

        $codeChallenge = strtr(rtrim(
            base64_encode(hash('sha256', $code_verifier, true))
        , '='), '+/', '-_');

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            'code_challenge' => $codeChallenge,
            'code_challenge_method' => 'S256',
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### 認証コードをアクセストークンに変換する

ユーザーが認証リクエストを承認した場合、消費アプリケーションにリダイレクトされます。コンシューマは、リダイレクト前に保存された値に対して `state` パラメータを検証する必要があります。これは、標準の認証コード付与と同様です。

`state` パラメータが一致する場合、コンシューマはアプリケーションに `POST` リクエストを発行してアクセストークンをリクエストする必要があります。リクエストには、ユーザーが認証リクエストを承認したときにアプリケーションによって発行された認証コードと、最初に生成されたコード検証子を含める必要があります：

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Http;

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'code_verifier' => $codeVerifier,
            'code' => $request->code,
        ]);

        return $response->json();
    });

<a name="password-grant-tokens"></a>
## パスワード付与トークン

> WARNING:  
> パスワード付与トークンの使用は推奨されなくなりました。代わりに、OAuth2 サーバーが現在推奨している [付与タイプ](https://oauth2.thephpleague.com/authorization-server/which-grant/) を選択する必要があります。

OAuth2 パスワード付与により、モバイルアプリケーションなどの他のファーストパーティクライアントは、メールアドレス / ユーザー名とパスワードを使用してアクセストークンを取得できます。これにより、ユーザーが OAuth2 認証コードリダイレクトフロー全体を経ることなく、ファーストパーティクライアントにアクセストークンを安全に発行できます。

パスワード付与を有効にするには、アプリケーションの `App\Providers\AppServiceProvider` クラスの `boot` メソッドで `enablePasswordGrant` メソッドを呼び出します：

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Passport::enablePasswordGrant();
    }

<a name="creating-a-password-grant-client"></a>
### パスワード付与クライアントの作成

アプリケーションがパスワード付与を介してトークンを発行する前に、パスワード付与クライアントを作成する必要があります。これは、`passport:client` Artisan コマンドに `--password` オプションを付けて行うことができます。**すでに `passport:install` コマンドを実行している場合は、このコマンドを実行する必要はありません：**

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### トークンのリクエスト

パスワードグラントクライアントを作成したら、ユーザーのメールアドレスとパスワードを指定して、`/oauth/token` ルートに `POST` リクエストを発行することでアクセストークンをリクエストできます。このルートは Passport によってすでに登録されているため、手動で定義する必要はありません。リクエストが成功すると、サーバーからの JSON レスポンスに `access_token` と `refresh_token` が含まれます。

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '',
]);

return $response->json();
```

> NOTE:  
> アクセストークンはデフォルトで長期間有効です。ただし、必要に応じて[アクセストークンの最大有効期間を設定](#configuration)することができます。

<a name="requesting-all-scopes"></a>
### すべてのスコープのリクエスト

パスワードグラントまたはクライアントクレデンシャルグラントを使用する場合、アプリケーションがサポートするすべてのスコープに対してトークンを承認したい場合があります。これを行うには、`*` スコープをリクエストします。`*` スコープをリクエストすると、トークンインスタンスの `can` メソッドは常に `true` を返します。このスコープは、`password` または `client_credentials` グラントを使用して発行されたトークンにのみ割り当てることができます。

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### ユーザープロバイダのカスタマイズ

アプリケーションが複数の[認証ユーザープロバイダ](authentication.md#introduction)を使用している場合、パスワードグラントクライアントが使用するユーザープロバイダを `--provider` オプションを指定して `artisan passport:client --password` コマンドで作成することができます。指定されたプロバイダ名は、アプリケーションの `config/auth.php` 設定ファイルで定義された有効なプロバイダと一致する必要があります。その後、[ミドルウェアを使用してルートを保護](#via-middleware)し、ガードで指定されたプロバイダのユーザーのみが認証されるようにすることができます。

<a name="customizing-the-username-field"></a>
### ユーザー名フィールドのカスタマイズ

パスワードグラントを使用して認証する場合、Passport は認証可能なモデルの `email` 属性を「ユーザー名」として使用します。ただし、この動作をカスタマイズするには、モデルに `findForPassport` メソッドを定義します。

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 指定されたユーザー名に対するユーザーインスタンスを検索します。
     */
    public function findForPassport(string $username): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### パスワード検証のカスタマイズ

パスワードグラントを使用して認証する場合、Passport はモデルの `password` 属性を使用して指定されたパスワードを検証します。モデルに `password` 属性がない場合や、パスワード検証ロジックをカスタマイズしたい場合は、モデルに `validateForPassportPasswordGrant` メソッドを定義できます。

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Passport パスワードグラントのユーザーのパスワードを検証します。
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant-tokens"></a>
## 暗黙的グラントトークン

> WARNING:  
> 暗黙的グラントトークンの使用は推奨されなくなりました。代わりに、OAuth2 Server が現在推奨している[グラントタイプ](https://oauth2.thephpleague.com/authorization-server/which-grant/)を選択してください。

暗黙的グラントは認証コードグラントに似ていますが、トークンは認証コードを交換することなくクライアントに返されます。このグラントは、クライアントクレデンシャルを安全に保存できない JavaScript またはモバイルアプリケーションで最も一般的に使用されます。グラントを有効にするには、アプリケーションの `App\Providers\AppServiceProvider` クラスの `boot` メソッドで `enableImplicitGrant` メソッドを呼び出します。

```php
/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

グラントが有効になったら、開発者はクライアント ID を使用してアプリケーションからアクセストークンをリクエストできます。消費アプリケーションは、次のようにアプリケーションの `/oauth/authorize` ルートにリダイレクトリクエストを行う必要があります。

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => '',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

> NOTE:  
> `/oauth/authorize` ルートは Passport によってすでに定義されているため、手動でこのルートを定義する必要はありません。

<a name="client-credentials-grant-tokens"></a>
## クライアントクレデンシャルグラントトークン

クライアントクレデンシャルグラントは、マシン間認証に適しています。たとえば、API を介してメンテナンスタスクを実行するスケジュールされたジョブでこのグラントを使用する場合があります。

アプリケーションがクライアントクレデンシャルグラントを介してトークンを発行する前に、クライアントクレデンシャルグラントクライアントを作成する必要があります。これは、`passport:client` Artisan コマンドの `--client` オプションを使用して行うことができます。

```shell
php artisan passport:client --client
```

次に、このグラントタイプを使用するために、`CheckClientCredentials` ミドルウェアのミドルウェアエイリアスを登録します。ミドルウェアエイリアスは、アプリケーションの `bootstrap/app.php` ファイルで定義できます。

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'client' => CheckClientCredentials::class
    ]);
})
```

その後、ミドルウェアをルートにアタッチします。

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client');
```

ルートへのアクセスを特定のスコープに制限するには、`client` ミドルウェアをルートにアタッチする際に、必要なスコープのカンマ区切りリストを指定できます。

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client:check-status,your-scope');
```

<a name="retrieving-tokens"></a>
### トークンの取得

このグラントタイプを使用してトークンを取得するには、`oauth/token` エンドポイントにリクエストを行います。

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => 'your-scope',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## パーソナルアクセストークン

場合によっては、ユーザーが通常の認証コードリダイレクトフローを経ずに自分自身にアクセストークンを発行したい場合があります。ユーザーがアプリケーションの UI を介して自分自身にトークンを発行できるようにすることは、ユーザーが API を試すために便利であったり、一般的なアクセストークン発行のより簡単なアプローチとなる場合があります。

> NOTE:  
> アプリケーションが主に Passport を使用してパーソナルアクセストークンを発行する場合、[Laravel Sanctum](sanctum.md) の使用を検討してください。Laravel の軽量な API アクセストークン発行ライブラリです。

<a name="creating-a-personal-access-client"></a>
### パーソナルアクセスクライアントの作成

アプリケーションがパーソナルアクセストークンを発行する前に、パーソナルアクセスクライアントを作成する必要があります。これは、`passport:client` Artisan コマンドに `--personal` オプションを指定して実行することで行うことができます。`passport:install` コマンドを既に実行している場合は、このコマンドを実行する必要はありません。

```shell
php artisan passport:client --personal
```

パーソナルアクセスクライアントを作成した後、クライアントの ID とプレーンテキストのシークレット値をアプリケーションの `.env` ファイルに配置します。

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

<a name="managing-personal-access-tokens"></a>
### パーソナルアクセストークンの管理

パーソナルアクセスクライアントを作成したら、`App\Models\User` モデルインスタンスの `createToken` メソッドを使用して、指定されたユーザーのトークンを発行できます。`createToken` メソッドは、トークンの名前を最初の引数として受け取り、オプションの[スコープ](#token-scopes)の配列を2番目の引数として受け取ります。

```php
use App\Models\User;

$user = User::find(1);

// スコープなしでトークンを作成する...
$token = $user->createToken('Token Name')->accessToken;

// スコープ付きでトークンを作成する...
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

<a name="personal-access-tokens-json-api"></a>
#### JSON API

Passport には、パーソナルアクセストークンを管理するための JSON API も含まれています。これにより、パーソナルアクセストークンを管理するためのフロントエンドを構築するための API を使用して、ユーザーと対話することができます。簡単にするために、Passport の JSON API を呼び出すための Vue コンポーネントを提供します。

この Vue コンポーネントを使用するには、`resources/js/app.js` ファイルに Passport コンポーネントを登録する必要があります。

<a name="get-oauthscopes"></a>
#### `GET /oauth/scopes`

このルートは、アプリケーションに定義されたすべての[スコープ](#token-scopes)を返します。このルートを使用して、ユーザーがパーソナルアクセストークンに割り当てることができるスコープをリストアップできます。

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

<a name="get-oauthpersonal-access-tokens"></a>
#### `GET /oauth/personal-access-tokens`

このルートは、認証されたユーザーが作成したすべてのパーソナルアクセストークンを返します。これは主に、ユーザーのすべてのトークンをリストアップして、編集または取り消しができるようにするために便利です。

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthpersonal-access-tokens"></a>
#### `POST /oauth/personal-access-tokens`

このルートは新しいパーソナルアクセストークンを作成します。トークンの`name`とトークンに割り当てるべき`scopes`の2つのデータが必要です。

```js
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // エラーをリストアップ...
    });
```

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>
#### `DELETE /oauth/personal-access-tokens/{token-id}`

このルートは、パーソナルアクセストークンを取り消すために使用できます。

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

<a name="protecting-routes"></a>
## ルートの保護

<a name="via-middleware"></a>
### ミドルウェア経由

Passportには、受信リクエストのアクセストークンを検証する[認証ガード](authentication.md#adding-custom-guards)が含まれています。`api`ガードを`passport`ドライバーを使用するように設定したら、有効なアクセストークンを必要とするルートに`auth:api`ミドルウェアを指定するだけです。

    Route::get('/user', function () {
        // ...
    })->middleware('auth:api');

> WARNING:  
> [クライアント認証情報付与](#client-credentials-grant-tokens)を使用している場合は、`auth:api`ミドルウェアの代わりに[`client`ミドルウェア](#client-credentials-grant-tokens)を使用してルートを保護する必要があります。

<a name="multiple-authentication-guards"></a>
#### 複数の認証ガード

アプリケーションが完全に異なるEloquentモデルを使用する異なるタイプのユーザーを認証する場合、アプリケーション内の各ユーザープロバイダータイプに対してガード設定を定義する必要があるかもしれません。これにより、特定のユーザープロバイダーを対象とするリクエストを保護できます。例えば、以下のガード設定が`config/auth.php`設定ファイルにある場合：

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

以下のルートは、`customers`ユーザープロバイダーを使用する`api-customers`ガードを使用して、受信リクエストを認証します。

    Route::get('/customer', function () {
        // ...
    })->middleware('auth:api-customers');

> NOTE:  
> Passportで複数のユーザープロバイダーを使用する方法の詳細については、[パスワード付与ドキュメント](#customizing-the-user-provider)を参照してください。

<a name="passing-the-access-token"></a>
### アクセストークンの渡し方

Passportによって保護されたルートを呼び出す際、アプリケーションのAPI消費者は、リクエストの`Authorization`ヘッダーに`Bearer`トークンとしてアクセストークンを指定する必要があります。例えば、Guzzle HTTPライブラリを使用する場合：

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

<a name="token-scopes"></a>
## トークンスコープ

スコープにより、APIクライアントはアカウントへのアクセスをリクエストする際に、特定の権限のセットをリクエストできます。例えば、eコマースアプリケーションを構築している場合、すべてのAPIクライアントが注文を行う権限を必要とするわけではありません。代わりに、クライアントが注文の配送状況にアクセスする権限のみをリクエストできるようにすることができます。言い換えれば、スコープにより、アプリケーションのユーザーは、第三者アプリケーションが自分に代わって実行できるアクションを制限できます。

<a name="defining-scopes"></a>
### スコープの定義

アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドで`Passport::tokensCan`メソッドを使用して、APIのスコープを定義できます。`tokensCan`メソッドは、スコープ名とスコープ説明の配列を受け取ります。スコープ説明は、承認承認画面にユーザーに表示されるもので、好きなものを指定できます。

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::tokensCan([
            'place-orders' => 'Place orders',
            'check-status' => 'Check order status',
        ]);
    }

<a name="default-scope"></a>
### デフォルトスコープ

クライアントが特定のスコープをリクエストしない場合、Passportサーバーを設定して、`setDefaultScope`メソッドを使用してトークンにデフォルトスコープを添付するようにできます。通常、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドからこのメソッドを呼び出す必要があります。

    use Laravel\Passport\Passport;

    Passport::tokensCan([
        'place-orders' => 'Place orders',
        'check-status' => 'Check order status',
    ]);

    Passport::setDefaultScope([
        'check-status',
        'place-orders',
    ]);

> NOTE:  
> Passportのデフォルトスコープは、ユーザーが生成したパーソナルアクセストークンには適用されません。

<a name="assigning-scopes-to-tokens"></a>
### トークンへのスコープの割り当て

<a name="when-requesting-authorization-codes"></a>
#### 認証コードをリクエストする際

認証コード付与を使用してアクセストークンをリクエストする際、クライアントは`scope`クエリ文字列パラメーターとして希望するスコープを指定する必要があります。`scope`パラメーターは、スペースで区切られたスコープのリストである必要があります。

    Route::get('/redirect', function () {
        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://example.com/callback',
            'response_type' => 'code',
            'scope' => 'place-orders check-status',
        ]);

        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });

<a name="when-issuing-personal-access-tokens"></a>
#### パーソナルアクセストークンを発行する際

`App\Models\User`モデルの`createToken`メソッドを使用してパーソナルアクセストークンを発行する場合、希望するスコープの配列をメソッドの第2引数として渡すことができます。

    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="checking-scopes"></a>
### スコープの確認

Passportには、指定されたスコープが付与されたトークンで受信リクエストが認証されているかどうかを確認するための2つのミドルウェアが含まれています。開始するには、アプリケーションの`bootstrap/app.php`ファイルに以下のミドルウェアエイリアスを定義します。

    use Laravel\Passport\Http\Middleware\CheckForAnyScope;
    use Laravel\Passport\Http\Middleware\CheckScopes;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'scopes' => CheckScopes::class,
            'scope' => CheckForAnyScope::class,
        ]);
    })

<a name="check-for-all-scopes"></a>
#### すべてのスコープの確認

`scopes`ミドルウェアは、受信リクエストのアクセストークンにリストされたすべてのスコープがあるかどうかを確認するためにルートに割り当てることができます。

    Route::get('/orders', function () {
        // アクセストークンに "check-status" と "place-orders" の両方のスコープがある...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);

<a name="check-for-any-scopes"></a>
#### いずれかのスコープの確認

`scope`ミドルウェアは、受信リクエストのアクセストークンにリストされたスコープの*少なくとも1つ*があるかどうかを確認するためにルートに割り当てることができます。

    Route::get('/orders', function () {
        // アクセストークンに "check-status" または "place-orders" のいずれかのスコープがある...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

<a name="checking-scopes-on-a-token-instance"></a>
#### トークンインスタンスでのスコープの確認

アクセストークンで認証されたリクエストがアプリケーションに入った後、認証された`App\Models\User`インスタンスの`tokenCan`メソッドを使用して、トークンに特定のスコープがあるかどうかを確認できます。

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            // ...
        }
    });

<a name="additional-scope-methods"></a>
#### 追加のスコープメソッド

`scopeIds`メソッドは、すべての定義されたID / 名前の配列を返します。

    use Laravel\Passport\Passport;

    Passport::scopeIds();

`scopes`メソッドは、すべての定義されたスコープを`Laravel\Passport\Scope`インスタンスの配列として返します。

    Passport::scopes();

`scopesFor`メソッドは、指定されたID / 名前に一致する`Laravel\Passport\Scope`インスタンスの配列を返します。

    Passport::scopesFor(['place-orders', 'check-status']);

`hasScope`メソッドを使用して、指定されたスコープが定義されているかどうかを確認できます。

    Passport::hasScope('place-orders');

<a name="consuming-your-api-with-javascript"></a>
## JavaScriptでのAPIの利用

APIを構築する際、自分のJavaScriptアプリケーションから自分のAPIを利用できることは非常に便利です。このAPI開発のアプローチにより、自分のアプリケーションが世界と共有しているのと同じAPIを利用できます。同じAPIは、Webアプリケーション、モバイルアプリケーション、サードパーティアプリケーション、およびさまざまなパッケージマネージャーに公開するSDKから利用される可能性があります。

通常、JavaScriptアプリケーションから自分のAPIを利用したい場合、アクセストークンを手動でアプリケーションに送信し、各リクエストとともに渡す必要があります。しかし、Passportにはこれを処理するミドルウェアが含まれています。必要なのは、アプリケーションの`bootstrap/app.php`ファイル内の`web`ミドルウェアグループに`CreateFreshApiToken`ミドルウェアを追加することだけです：

    use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->web(append: [
            CreateFreshApiToken::class,
        ]);
    })

> WARNING:  
> `CreateFreshApiToken`ミドルウェアがミドルウェアスタックの最後にリストされていることを確認する必要があります。

このミドルウェアは、送信するレスポンスに`laravel_token`クッキーを添付します。このクッキーには、PassportがJavaScriptアプリケーションからのAPIリクエストを認証するために使用する暗号化されたJWTが含まれています。JWTの有効期間は、`session.lifetime`設定値と同じです。現在、ブラウザは自動的にすべての後続リクエストとともにクッキーを送信するため、明示的にアクセストークンを渡すことなく、アプリケーションのAPIにリクエストを行うことができます：

    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>
#### クッキー名のカスタマイズ

必要に応じて、`Passport::cookie`メソッドを使用して`laravel_token`クッキーの名前をカスタマイズできます。通常、このメソッドはアプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドから呼び出す必要があります：

    /**
     * 任意のアプリケーションサービスのブートストラップ。
     */
    public function boot(): void
    {
        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>
#### CSRF保護

この認証方法を使用する場合、有効なCSRFトークンヘッダーがリクエストに含まれていることを確認する必要があります。デフォルトのLaravel JavaScriptスキャフォールディングにはAxiosインスタンスが含まれており、暗号化された`XSRF-TOKEN`クッキー値を使用して同一生成元リクエストに`X-XSRF-TOKEN`ヘッダーを自動的に送信します。

> NOTE:  
> `X-XSRF-TOKEN`の代わりに`X-CSRF-TOKEN`ヘッダーを送信する場合は、`csrf_token()`によって提供される暗号化されていないトークンを使用する必要があります。

<a name="events"></a>
## イベント

Passportは、アクセストークンとリフレッシュトークンを発行する際にイベントを発生させます。データベース内の他のアクセストークンを整理または取り消すために、[これらのイベントをリッスン](events.md)できます：

<div class="overflow-auto" markdown=1>

| イベント名 |
| --- |
| `Laravel\Passport\Events\AccessTokenCreated` |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## テスト

Passportの`actingAs`メソッドは、現在認証されているユーザーとそのスコープを指定するために使用できます。`actingAs`メソッドに渡される最初の引数はユーザーインスタンスで、2番目の引数はユーザーのトークンに付与されるスコープの配列です：

===  "Pest"
```php
use App\Models\User;
use Laravel\Passport\Passport;

test('サーバーを作成できる', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['create-servers']
    );

    $response = $this->post('/api/create-server');

    $response->assertStatus(201);
});
```

===  "PHPUnit"
```php
use App\Models\User;
use Laravel\Passport\Passport;

public function test_サーバーを作成できる(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['create-servers']
    );

    $response = $this->post('/api/create-server');

    $response->assertStatus(201);
}
```

Passportの`actingAsClient`メソッドは、現在認証されているクライアントとそのスコープを指定するために使用できます。`actingAsClient`メソッドに渡される最初の引数はクライアントインスタンスで、2番目の引数はクライアントのトークンに付与されるスコープの配列です：

===  "Pest"
```php
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('注文を取得できる', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['check-status']
    );

    $response = $this->get('/api/orders');

    $response->assertStatus(200);
});
```

===  "PHPUnit"
```php
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_注文を取得できる(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['check-status']
    );

    $response = $this->get('/api/orders');

    $response->assertStatus(200);
}
```
