# Laravel Socialite

- [はじめに](#introduction)
- [インストール](#installation)
- [Socialiteのアップグレード](#upgrading-socialite)
- [設定](#configuration)
- [認証](#authentication)
    - [ルーティング](#routing)
    - [認証と保存](#authentication-and-storage)
    - [アクセススコープ](#access-scopes)
    - [Slack Botスコープ](#slack-bot-scopes)
    - [オプションパラメータ](#optional-parameters)
- [ユーザー詳細の取得](#retrieving-user-details)

<a name="introduction"></a>
## はじめに

Laravelは、従来のフォームベースの認証に加えて、[Laravel Socialite](https://github.com/laravel/socialite)を使用してOAuthプロバイダーで認証するためのシンプルで便利な方法も提供しています。Socialiteは現在、Facebook、X、LinkedIn、Google、GitHub、GitLab、Bitbucket、Slackを介した認証をサポートしています。

> NOTE:  
> 他のプラットフォーム用のアダプターは、コミュニティ主導の[Socialite Providers](https://socialiteproviders.com/)ウェブサイトから入手できます。

<a name="installation"></a>
## インストール

Socialiteを使い始めるには、Composerパッケージマネージャーを使用して、プロジェクトの依存関係にパッケージを追加します。

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## Socialiteのアップグレード

Socialiteを新しいメジャーバージョンにアップグレードする際は、[アップグレードガイド](https://github.com/laravel/socialite/blob/master/UPGRADE.md)を注意深く確認することが重要です。

<a name="configuration"></a>
## 設定

Socialiteを使用する前に、アプリケーションが利用するOAuthプロバイダーの認証情報を追加する必要があります。通常、これらの認証情報は、認証するサービスのダッシュボード内で「開発者アプリケーション」を作成することで取得できます。

これらの認証情報は、アプリケーションの`config/services.php`設定ファイルに配置し、キー`facebook`、`x`、`linkedin-openid`、`google`、`github`、`gitlab`、`bitbucket`、`slack`、または`slack-openid`を使用します。これは、アプリケーションが必要とするプロバイダーによります。

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],

> NOTE:  
> `redirect`オプションに相対パスが含まれている場合、自動的に完全修飾URLに解決されます。

<a name="authentication"></a>
## 認証

<a name="routing"></a>
### ルーティング

OAuthプロバイダーを使用してユーザーを認証するには、ユーザーをOAuthプロバイダーにリダイレクトするためのルートと、認証後にプロバイダーからのコールバックを受け取るための別のルートが必要です。以下の例のルートは、両方のルートの実装を示しています。

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/redirect', function () {
        return Socialite::driver('github')->redirect();
    });

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // $user->token
    });

`Socialite`ファサードが提供する`redirect`メソッドは、ユーザーをOAuthプロバイダーにリダイレクトする役割を果たし、`user`メソッドは、認証要求を承認した後にプロバイダーからの受信リクエストを調べ、ユーザーの情報を取得します。

<a name="authentication-and-storage"></a>
### 認証と保存

OAuthプロバイダーからユーザーが取得されたら、アプリケーションのデータベースにユーザーが存在するかどうかを判断し、[ユーザーを認証](authentication.md#authenticate-a-user-instance)できます。アプリケーションのデータベースにユーザーが存在しない場合、通常はユーザーを表す新しいレコードをデータベースに作成します。

    use App\Models\User;
    use Illuminate\Support\Facades\Auth;
    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $githubUser = Socialite::driver('github')->user();

        $user = User::updateOrCreate([
            'github_id' => $githubUser->id,
        ], [
            'name' => $githubUser->name,
            'email' => $githubUser->email,
            'github_token' => $githubUser->token,
            'github_refresh_token' => $githubUser->refreshToken,
        ]);

        Auth::login($user);

        return redirect('/dashboard');
    });

> NOTE:  
> 特定のOAuthプロバイダーからどのようなユーザー情報が利用可能かについての詳細は、[ユーザー詳細の取得](#retrieving-user-details)のドキュメントを参照してください。

<a name="access-scopes"></a>
### アクセススコープ

ユーザーをリダイレクトする前に、`scopes`メソッドを使用して認証リクエストに含めるべき「スコープ」を指定できます。このメソッドは、以前に指定されたすべてのスコープを指定したスコープとマージします。

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

認証リクエストのすべての既存のスコープを上書きするには、`setScopes`メソッドを使用します。

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="slack-bot-scopes"></a>
### Slack Botスコープ

SlackのAPIは、それぞれ独自の[権限スコープ](https://api.slack.com/scopes)を持つ[異なるタイプのアクセストークン](https://api.slack.com/authentication/token-types)を提供します。Socialiteは、以下の両方のSlackアクセストークンタイプと互換性があります。

<div class="content-list" markdown="1">

- Bot（`xoxb-`で始まる）
- User（`xoxp-`で始まる）

</div>

デフォルトでは、`slack`ドライバーは`user`トークンを生成し、ドライバーの`user`メソッドを呼び出すとユーザーの詳細が返されます。

Botトークンは、主にアプリケーションがアプリケーションのユーザーが所有する外部のSlackワークスペースに通知を送信する場合に便利です。Botトークンを生成するには、ユーザーをSlackにリダイレクトして認証する前に`asBotUser`メソッドを呼び出します。

    return Socialite::driver('slack')
        ->asBotUser()
        ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
        ->redirect();

さらに、Slackが認証後にユーザーをアプリケーションにリダイレクトした後に`user`メソッドを呼び出す前に、`asBotUser`メソッドを呼び出す必要があります。

    $user = Socialite::driver('slack')->asBotUser()->user();

Botトークンを生成する場合、`user`メソッドは依然として`Laravel\Socialite\Two\User`インスタンスを返します。ただし、`token`プロパティのみがハイドレートされます。このトークンは、[認証されたユーザーのSlackワークスペースに通知を送信する](notifications.md#notifying-external-slack-workspaces)ために保存できます。

<a name="optional-parameters"></a>
### オプションパラメータ

多くのOAuthプロバイダーは、リダイレクトリクエストで他のオプションパラメーターをサポートしています。リクエストにオプションパラメーターを含めるには、連想配列を指定して`with`メソッドを呼び出します。

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')
        ->with(['hd' => 'example.com'])
        ->redirect();

> WARNING:  
> `with`メソッドを使用する場合、`state`や`response_type`などの予約語を渡さないように注意してください。

<a name="retrieving-user-details"></a>
## ユーザー詳細の取得

ユーザーがアプリケーションの認証コールバックルートにリダイレクトされた後、Socialiteの`user`メソッドを使用してユーザーの詳細を取得できます。`user`メソッドによって返されるユーザーオブジェクトは、独自のデータベースにユーザーに関する情報を保存するために使用できるさまざまなプロパティとメソッドを提供します。

このオブジェクトで利用可能なプロパティとメソッドは、認証に使用しているOAuthプロバイダーがOAuth 1.0またはOAuth 2.0をサポートしているかによって異なります。

    use Laravel\Socialite\Facades\Socialite;

    Route::get('/auth/callback', function () {
        $user = Socialite::driver('github')->user();

        // OAuth 2.0プロバイダー...
        $token = $user->token;
        $refreshToken = $user->refreshToken;
        $expiresIn = $user->expiresIn;

        // OAuth 1.0プロバイダー...
        $token = $user->token;
        $tokenSecret = $user->tokenSecret;

        // すべてのプロバイダー...
        $user->getId();
        $user->getNickname();
        $user->getName();
        $user->getEmail();
        $user->getAvatar();
    });

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### トークンからユーザー詳細を取得

ユーザーの有効なアクセストークンが既にある場合、Socialiteの`userFromToken`メソッドを使用してユーザーの詳細を取得できます。

    use Laravel\Socialite\Facades\Socialite;

    $user = Socialite::driver('github')->userFromToken($token);

iOSアプリケーションを介してFacebook Limited Loginを使用している場合、Facebookはアクセストークンの代わりにOIDCトークンを返します。アクセストークンと同様に、OIDCトークンを`userFromToken`メソッドに提供してユーザー詳細を取得できます。

<a name="stateless-authentication"></a>
#### ステートレス認証

`stateless`メソッドを使用してセッション状態の検証を無効にすることができます。これは、Cookieベースのセッションを使用しないステートレスAPIにソーシャル認証を追加する場合に便利です。

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('google')->stateless()->user();

