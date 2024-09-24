# ルーティング

- [基本ルーティング](#basic-routing)
    - [デフォルトのルートファイル](#the-default-route-files)
    - [リダイレクトルート](#redirect-routes)
    - [ビュールート](#view-routes)
    - [ルートのリスト表示](#listing-your-routes)
    - [ルーティングのカスタマイズ](#routing-customization)
- [ルートパラメータ](#route-parameters)
    - [必須パラメータ](#required-parameters)
    - [オプションパラメータ](#parameters-optional-parameters)
    - [正規表現制約](#parameters-regular-expression-constraints)
- [名前付きルート](#named-routes)
- [ルートグループ](#route-groups)
    - [ミドルウェア](#route-group-middleware)
    - [コントローラー](#route-group-controllers)
    - [サブドメインルーティング](#route-group-subdomain-routing)
    - [ルートプレフィックス](#route-group-prefixes)
    - [ルート名プレフィックス](#route-group-name-prefixes)
- [ルートモデルバインディング](#route-model-binding)
    - [暗黙のバインディング](#implicit-binding)
    - [暗黙の列挙型バインディング](#implicit-enum-binding)
    - [明示的なバインディング](#explicit-binding)
- [フォールバックルート](#fallback-routes)
- [レートリミット](#rate-limiting)
    - [レートリミッターの定義](#defining-rate-limiters)
    - [ルートへのレートリミッターのアタッチ](#attaching-rate-limiters-to-routes)
- [フォームメソッドスプーフィング](#form-method-spoofing)
- [現在のルートへのアクセス](#accessing-the-current-route)
- [クロスオリジンリソース共有 (CORS)](#cors)
- [ルートキャッシュ](#route-caching)

<a name="basic-routing"></a>
## 基本ルーティング

最も基本的なLaravelルートは、URIとクロージャを受け取り、複雑なルーティング設定ファイルを必要とせずに、非常にシンプルで表現力豊かな方法でルートと動作を定義します。

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Hello World';
    });

<a name="the-default-route-files"></a>
### デフォルトのルートファイル

すべてのLaravelルートは、`routes`ディレクトリにあるルートファイルで定義されます。これらのファイルは、アプリケーションの`bootstrap/app.php`ファイルで指定された設定を使用して、Laravelによって自動的にロードされます。`routes/web.php`ファイルは、Webインターフェース用のルートを定義します。これらのルートには、セッション状態やCSRF保護などの機能を提供する`web`[ミドルウェアグループ](middleware.md#laravels-default-middleware-groups)が割り当てられています。

ほとんどのアプリケーションでは、`routes/web.php`ファイルでルートの定義を開始します。`routes/web.php`で定義されたルートには、ブラウザで定義されたルートのURLを入力することでアクセスできます。例えば、ブラウザで`http://example.com/user`に移動することで、次のルートにアクセスできます。

    use App\Http\Controllers\UserController;

    Route::get('/user', [UserController::class, 'index']);

<a name="api-routes"></a>
#### APIルート

アプリケーションがステートレスなAPIも提供する場合、`install:api` Artisanコマンドを使用してAPIルーティングを有効にできます。

```shell
php artisan install:api
```

`install:api`コマンドは、サードパーティのAPIコンシューマー、SPA、またはモバイルアプリケーションを認証するために使用できる堅牢でシンプルなAPIトークン認証ガードを提供する[Laravel Sanctum](sanctum.md)をインストールします。さらに、`install:api`コマンドは`routes/api.php`ファイルを作成します。

    Route::get('/user', function (Request $request) {
        return $request->user();
    })->middleware('auth:sanctum');

`routes/api.php`のルートはステートレスであり、`api`[ミドルウェアグループ](middleware.md#laravels-default-middleware-groups)に割り当てられています。さらに、`/api` URIプレフィックスがこれらのルートに自動的に適用されるため、ファイル内のすべてのルートに手動で適用する必要はありません。プレフィックスは、アプリケーションの`bootstrap/app.php`ファイルを変更することで変更できます。

    ->withRouting(
        api: __DIR__.'/../routes/api.php',
        apiPrefix: 'api/admin',
        // ...
    )

<a name="available-router-methods"></a>
#### 利用可能なルーターメソッド

ルーターを使用すると、任意のHTTP動詞に応答するルートを登録できます。

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

複数のHTTP動詞に応答するルートを登録する必要がある場合は、`match`メソッドを使用します。また、すべてのHTTP動詞に応答するルートを登録するには、`any`メソッドを使用します。

    Route::match(['get', 'post'], '/', function () {
        // ...
    });

    Route::any('/', function () {
        // ...
    });

> NOTE:  
> 同じURIを共有する複数のルートを定義する場合、`get`、`post`、`put`、`patch`、`delete`、および`options`メソッドを使用するルートは、`any`、`match`、および`redirect`メソッドを使用するルートの前に定義する必要があります。これにより、受信リクエストが正しいルートに一致するようになります。

<a name="dependency-injection"></a>
#### 依存性注入

ルートのコールバック署名でルートが必要とする依存関係をタイプヒントで指定できます。宣言された依存関係は、Laravel[サービスコンテナ](container.md)によって自動的に解決され、コールバックに注入されます。例えば、`Illuminate\Http\Request`クラスをタイプヒントで指定して、現在のHTTPリクエストをルートコールバックに自動的に注入できます。

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### CSRF保護

`web`ルートファイルで定義された`POST`、`PUT`、`PATCH`、または`DELETE`ルートを指すすべてのHTMLフォームには、CSRFトークンフィールドを含める必要があることに注意してください。そうしないと、リクエストは拒否されます。CSRF保護の詳細については、[CSRFドキュメント](csrf.md)を参照してください。

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### リダイレクトルート

別のURIにリダイレクトするルートを定義する場合は、`Route::redirect`メソッドを使用できます。このメソッドは、単純なリダイレクトを実行するために完全なルートまたはコントローラーを定義する必要がない便利なショートカットを提供します。

    Route::redirect('/here', '/there');

デフォルトでは、`Route::redirect`は`302`ステータスコードを返します。オプションの第三引数を使用してステータスコードをカスタマイズできます。

    Route::redirect('/here', '/there', 301);

または、`Route::permanentRedirect`メソッドを使用して`301`ステータスコードを返すこともできます。

    Route::permanentRedirect('/here', '/there');

> WARNING:  
> リダイレクトルートでルートパラメータを使用する場合、Laravelによって予約されている次のパラメータは使用できません: `destination` と `status`。

<a name="view-routes"></a>
### ビュールート

ルートが[ビュー](views.md)のみを返す必要がある場合は、`Route::view`メソッドを使用できます。`redirect`メソッドと同様に、このメソッドは完全なルートまたはコントローラーを定義する必要がない便利なショートカットを提供します。`view`メソッドは、最初の引数としてURIを受け取り、2番目の引数としてビュー名を受け取ります。さらに、オプションの第三引数としてビューに渡すデータの配列を指定できます。

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> WARNING:  
> ビュールートでルートパラメータを使用する場合、Laravelによって予約されている次のパラメータは使用できません: `view`、`data`、`status`、および`headers`。

<a name="listing-your-routes"></a>
### ルートのリスト表示

`route:list` Artisanコマンドを使用すると、アプリケーションで定義されているすべてのルートの概要を簡単に確認できます。

```shell
php artisan route:list
```

デフォルトでは、各ルートに割り当てられたルートミドルウェアは`route:list`の出力に表示されません。ただし、コマンドに`-v`オプションを追加することで、Laravelにルートミドルウェアとミドルウェアグループ名を表示するよう指示できます。

```shell
php artisan route:list -v

# ミドルウェアグループを展開...
php artisan route:list -vv
```

また、Laravelに特定のURIで始まるルートのみを表示するよう指示することもできます。

```shell
php artisan route:list --path=api
```

さらに、`route:list`コマンドを実行する際に`--except-vendor`オプションを指定することで、サードパーティパッケージによって定義されたルートを非表示にするようLaravelに指示できます。

```shell
php artisan route:list --except-vendor
```

同様に、`route:list`コマンドを実行する際に`--only-vendor`オプションを指定することで、サードパーティパッケージによって定義されたルートのみを表示するようLaravelに指示できます。

```shell
php artisan route:list --only-vendor
```

<a name="routing-customization"></a>
### ルーティングのカスタマイズ

デフォルトでは、アプリケーションのルートは`bootstrap/app.php`ファイルによって設定およびロードされます。

```php
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

ただし、アプリケーションのルートのサブセットを含むまったく新しいファイルを定義したい場合があります。これを実現するには、`withRouting`メソッドに`then`クロージャを提供します。このクロージャ内で、アプリケーションに必要な追加のルートを登録できます。

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

または、`withRouting`メソッドに`using`クロージャを提供することで、ルート登録を完全に制御することもできます。この引数が渡されると、フレームワークによってHTTPルートは登録されず、すべてのルートを手動で登録する責任があります。

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    using: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

<a name="route-parameters"></a>
## ルートパラメータ

<a name="required-parameters"></a>
### 必須パラメータ

ルート内でURIのセグメントをキャプチャする必要がある場合があります。たとえば、URLからユーザーのIDをキャプチャする必要がある場合です。ルートパラメータを定義することでそれが可能です。

    Route::get('/user/{id}', function (string $id) {
        return 'User '.$id;
    });

ルートに必要なだけのルートパラメータを定義できます。

    Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
        // ...
    });

ルートパラメータは常に`{}`波括弧で囲まれ、アルファベット文字で構成される必要があります。アンダースコア（`_`）もルートパラメータ名に使用できます。ルートパラメータは、その順序に基づいてルートコールバック/コントローラに注入されます。ルートコールバック/コントローラの引数の名前は重要ではありません。

<a name="parameters-and-dependency-injection"></a>
#### パラメータと依存性注入

ルートにLaravelサービスコンテナが自動的に注入したい依存関係がある場合、依存関係の後にルートパラメータをリストする必要があります。

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, string $id) {
        return 'User '.$id;
    });

<a name="parameters-optional-parameters"></a>
### オプションパラメータ

URIに常に存在するとは限らないルートパラメータを指定する必要がある場合があります。パラメータ名の後に`?`マークを付けることでそれが可能です。ルートの対応する変数にデフォルト値を指定してください。

    Route::get('/user/{name?}', function (?string $name = null) {
        return $name;
    });

    Route::get('/user/{name?}', function (?string $name = 'John') {
        return $name;
    });

<a name="parameters-regular-expression-constraints"></a>
### 正規表現制約

ルートインスタンスの`where`メソッドを使用して、ルートパラメータのフォーマットを制約することができます。`where`メソッドはパラメータの名前と、パラメータをどのように制約するかを定義する正規表現を受け取ります。

    Route::get('/user/{name}', function (string $name) {
        // ...
    })->where('name', '[A-Za-z]+');

    Route::get('/user/{id}', function (string $id) {
        // ...
    })->where('id', '[0-9]+');

    Route::get('/user/{id}/{name}', function (string $id, string $name) {
        // ...
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

一般的に使用される正規表現パターンには、ルートにパターン制約を迅速に追加できるヘルパーメソッドがあります。

    Route::get('/user/{id}/{name}', function (string $id, string $name) {
        // ...
    })->whereNumber('id')->whereAlpha('name');

    Route::get('/user/{name}', function (string $name) {
        // ...
    })->whereAlphaNumeric('name');

    Route::get('/user/{id}', function (string $id) {
        // ...
    })->whereUuid('id');

    Route::get('/user/{id}', function (string $id) {
        // ...
    })->whereUlid('id');

    Route::get('/category/{category}', function (string $category) {
        // ...
    })->whereIn('category', ['movie', 'song', 'painting']);

    Route::get('/category/{category}', function (string $category) {
        // ...
    })->whereIn('category', CategoryEnum::cases());

受信リクエストがルートパターン制約に一致しない場合、404 HTTPレスポンスが返されます。

<a name="parameters-global-constraints"></a>
#### グローバル制約

ルートパラメータを常に特定の正規表現で制約したい場合、`pattern`メソッドを使用できます。これらのパターンは、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドで定義する必要があります。

    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Route::pattern('id', '[0-9]+');
    }

パターンが定義されると、そのパラメータ名を使用するすべてのルートに自動的に適用されます。

    Route::get('/user/{id}', function (string $id) {
        // {id}が数値の場合のみ実行されます...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### エンコードされたフォワードスラッシュ

Laravelのルーティングコンポーネントは、`/`を除くすべての文字をルートパラメータ値内に含めることができます。`where`条件の正規表現を使用して、プレースホルダーの一部として`/`を明示的に許可する必要があります。

    Route::get('/search/{search}', function (string $search) {
        return $search;
    })->where('search', '.*');

> WARNING:  
> エンコードされたフォワードスラッシュは、最後のルートセグメント内でのみサポートされます。

<a name="named-routes"></a>
## 名前付きルート

名前付きルートを使用すると、特定のルートのURLやリダイレクトを便利に生成できます。ルート定義に`name`メソッドを連結することで、ルートに名前を指定できます。

    Route::get('/user/profile', function () {
        // ...
    })->name('profile');

コントローラアクションに対してもルート名を指定できます。

    Route::get(
        '/user/profile',
        [UserProfileController::class, 'show']
    )->name('profile');

> WARNING:  
> ルート名は常に一意である必要があります。

<a name="generating-urls-to-named-routes"></a>
#### 名前付きルートへのURL生成

ルートに名前を割り当てたら、Laravelの`route`および`redirect`ヘルパー関数を介して、ルートの名前を使用してURLやリダイレクトを生成できます。

    // URLの生成...
    $url = route('profile');

    // リダイレクトの生成...
    return redirect()->route('profile');

    return to_route('profile');

名前付きルートがパラメータを定義している場合、`route`関数の第2引数としてパラメータを渡すことができます。指定されたパラメータは、生成されたURLの正しい位置に自動的に挿入されます。

    Route::get('/user/{id}/profile', function (string $id) {
        // ...
    })->name('profile');

    $url = route('profile', ['id' => 1]);

配列に追加のパラメータを渡すと、それらのキー/値ペアが生成されたURLのクエリ文字列に自動的に追加されます。

    Route::get('/user/{id}/profile', function (string $id) {
        // ...
    })->name('profile');

    $url = route('profile', ['id' => 1, 'photos' => 'yes']);

    // /user/1/profile?photos=yes

> NOTE:  
> 場合によっては、URLパラメータにリクエスト全体のデフォルト値（現在のロケールなど）を指定したいことがあります。これを実現するには、[`URL::defaults`メソッド](urls.md#default-values)を使用できます。

<a name="inspecting-the-current-route"></a>
#### 現在のルートの検査

現在のリクエストが特定の名前付きルートにルーティングされたかどうかを判断したい場合、Routeインスタンスの`named`メソッドを使用できます。たとえば、ルートミドルウェアから現在のルート名をチェックできます。

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->route()->named('profile')) {
            // ...
        }

        return $next($request);
    }

<a name="route-groups"></a>
## ルートグループ

ルートグループを使用すると、ミドルウェアなどのルート属性を大量のルートに共有できます。個々のルートにそれらの属性を定義する必要がありません。

ネストされたグループは、親グループとの属性を賢く「マージ」しようとします。ミドルウェアと`where`条件はマージされ、名前とプレフィックスは追加されます。名前空間の区切り文字とURIプレフィックスのスラッシュは、必要に応じて自動的に追加されます。

<a name="route-group-middleware"></a>
### ミドルウェア

グループ内のすべてのルートに[ミドルウェア](middleware.md)を割り当てるには、グループを定義する前に`middleware`メソッドを使用します。ミドルウェアは配列にリストされた順序で実行されます。

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // first & secondミドルウェアを使用...
        });

        Route::get('/user/profile', function () {
            // first & secondミドルウェアを使用...
        });
    });

<a name="route-group-controllers"></a>
### コントローラ

グループ内のすべてのルートが同じ[コントローラ](controllers.md)を使用する場合、`controller`メソッドを使用して、グループ内のすべてのルートに共通のコントローラを定義できます。その後、ルートを定義する際に、呼び出すコントローラメソッドのみを指定する必要があります。

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### サブドメインルーティング

ルートグループは、サブドメインルーティングにも使用できます。サブドメインはルートURIと同様にルートパラメータを割り当てることができ、サブドメインの一部をルートまたはコントローラで使用するためにキャプチャできます。サブドメインは、グループを定義する前に`domain`メソッドを呼び出すことで指定できます。

    Route::domain('{account}.example.com')->group(function () {
        Route::get('/user/{id}', function (string $account, string $id) {
            // ...
        });
    });

> WARNING:  
> サブドメインのルートが到達可能であることを確保するために、ルートドメインのルートを登録する前にサブドメインのルートを登録する必要があります。これにより、同じURIパスを持つサブドメインのルートがルートドメインのルートによって上書きされるのを防ぐことができます。

<a name="route-group-prefixes"></a>
### ルートプレフィックス

`prefix`メソッドは、グループ内の各ルートに指定されたURIをプレフィックスとして付けるために使用できます。例えば、グループ内のすべてのルートURIに`admin`をプレフィックスとして付けたい場合があります:

    Route::prefix('admin')->group(function () {
        Route::get('/users', function () {
            // "/admin/users" URLにマッチ
        });
    });

<a name="route-group-name-prefixes"></a>
### ルート名プレフィックス

`name`メソッドは、グループ内の各ルート名に指定された文字列をプレフィックスとして付けるために使用できます。例えば、グループ内のすべてのルート名に`admin`をプレフィックスとして付けたい場合があります。指定された文字列は、指定されたとおりにルート名にプレフィックスとして付けられるため、プレフィックスに末尾の`.`文字を提供するようにします:

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // "admin.users"という名前が割り当てられたルート
        })->name('users');
    });

<a name="route-model-binding"></a>
## ルートモデルバインディング

ルートまたはコントローラアクションにモデルIDを注入する場合、そのIDに対応するモデルを取得するためにデータベースをクエリすることがよくあります。Laravelのルートモデルバインディングは、モデルインスタンスをルートに直接自動的に注入する便利な方法を提供します。例えば、ユーザーのIDを注入する代わりに、指定されたIDに一致する`User`モデルインスタンス全体を注入できます。

<a name="implicit-binding"></a>
### 暗黙的バインディング

Laravelは、ルートまたはコントローラアクションで定義されたEloquentモデルを自動的に解決します。これらのモデルの型付き変数名は、ルートセグメント名と一致します。例えば:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

`$user`変数は`App\Models\User` Eloquentモデルとして型付けされ、変数名が`{user}` URIセグメントと一致するため、Laravelは自動的にリクエストURIから対応する値と一致するIDを持つモデルインスタンスを注入します。データベースに一致するモデルインスタンスが見つからない場合、404 HTTPレスポンスが自動的に生成されます。

もちろん、コントローラメソッドを使用する場合も暗黙的バインディングは可能です。ここでも、`{user}` URIセグメントが`App\Models\User`型付きの`$user`変数と一致することに注意してください:

    use App\Http\Controllers\UserController;
    use App\Models\User;

    // ルート定義...
    Route::get('/users/{user}', [UserController::class, 'show']);

    // コントローラメソッド定義...
    public function show(User $user)
    {
        return view('user.profile', ['user' => $user]);
    }

<a name="implicit-soft-deleted-models"></a>
#### ソフトデリートされたモデル

通常、暗黙的モデルバインディングは、[ソフトデリート](eloquent.md#soft-deleting)されたモデルを取得しません。ただし、ルートの定義に`withTrashed`メソッドをチェーンすることで、これらのモデルを取得するように暗黙的バインディングに指示できます:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    })->withTrashed();

<a name="customizing-the-default-key-name"></a>
#### キーのカスタマイズ

場合によっては、`id`以外のカラムを使用してEloquentモデルを解決したいことがあります。その場合、ルートパラメータ定義でカラムを指定できます:

    use App\Models\Post;

    Route::get('/posts/{post:slug}', function (Post $post) {
        return $post;
    });

特定のモデルクラスを取得する際に常に`id`以外のデータベースカラムを使用するようにモデルバインディングを設定したい場合は、Eloquentモデルの`getRouteKeyName`メソッドをオーバーライドできます:

    /**
     * モデルのルートキーを取得します。
     */
    public function getRouteKeyName(): string
    {
        return 'slug';
    }

<a name="implicit-model-binding-scoping"></a>
#### カスタムキーとスコープ

単一のルート定義で複数のEloquentモデルを暗黙的にバインドする場合、2番目のEloquentモデルが前のEloquentモデルの子である必要があるようにスコープを設定したい場合があります。例えば、特定のユーザーのブログ投稿をスラッグで取得するルート定義を考えてみましょう:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    });

ネストされたルートパラメータとしてカスタムキー付きの暗黙的バインディングを使用する場合、Laravelは親モデルのリレーション名を推測するための規約を使用して、親モデルによってネストされたモデルを取得するためのクエリを自動的にスコープします。この場合、`User`モデルには`posts`というリレーション（ルートパラメータ名の複数形）があると想定され、これを使用して`Post`モデルを取得できます。

カスタムキーが提供されていない場合でも、Laravelに「子」バインディングをスコープするように指示することができます。そのためには、ルートを定義する際に`scopeBindings`メソッドを呼び出します:

    use App\Models\Post;
    use App\Models\User;

    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    })->scopeBindings();

または、ルート定義のグループ全体にスコープ付きバインディングを使用するように指示することもできます:

    Route::scopeBindings()->group(function () {
        Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
            return $post;
        });
    });

同様に、`withoutScopedBindings`メソッドを呼び出すことで、バインディングをスコープしないようにLaravelに明示的に指示することもできます:

    Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
        return $post;
    })->withoutScopedBindings();

<a name="customizing-missing-model-behavior"></a>
#### モデルが見つからない場合の動作をカスタマイズ

通常、暗黙的にバインドされたモデルが見つからない場合、404 HTTPレスポンスが生成されます。ただし、ルートを定義する際に`missing`メソッドを呼び出すことで、この動作をカスタマイズできます。`missing`メソッドは、暗黙的にバインドされたモデルが見つからない場合に呼び出されるクロージャを受け取ります:

    use App\Http\Controllers\LocationsController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
            ->name('locations.view')
            ->missing(function (Request $request) {
                return Redirect::route('locations.index');
            });

<a name="implicit-enum-binding"></a>
### 暗黙的Enumバインディング

PHP 8.1では、[Enums](https://www.php.net/manual/en/language.enumerations.backed.php)のサポートが導入されました。この機能を補完するために、Laravelではルート定義に[backed Enum](https://www.php.net/manual/en/language.enumerations.backed.php)を型付けし、ルートセグメントが有効なEnum値に対応する場合にのみルートが呼び出されるようにすることができます。それ以外の場合、Laravelは404 HTTPレスポンスを自動的に返します。例えば、次のEnumがあるとします:

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

`{category}`ルートセグメントが`fruits`または`people`である場合にのみ呼び出されるルートを定義できます。それ以外の場合、Laravelは404 HTTPレスポンスを返します:

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### 明示的バインディング

モデルバインディングにLaravelの暗黙的な規約ベースの解決を使用する必要はありません。ルートパラメータがモデルにどのように対応するかを明示的に定義することもできます。明示的なバインディングを登録するには、ルーターの`model`メソッドを使用して、指定されたパラメータのクラスを指定します。明示的なモデルバインディングは、`AppServiceProvider`クラスの`boot`メソッドの先頭で定義する必要があります:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * アプリケーションサービスを起動します。
     */
    public function boot(): void
    {
        Route::model('user', User::class);
    }

次に、`{user}`パラメータを含むルートを定義します:

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        // ...
    });

すべての`{user}`パラメータを`App\Models\User`モデルにバインドしているため、そのクラスのインスタンスがルートに注入されます。したがって、例えば`users/1`へのリクエストは、IDが`1`の`User`インスタンスをデータベースから注入します。

データベースに一致するモデルインスタンスが見つからない場合、404 HTTPレスポンスが自動的に生成されます。

<a name="customizing-the-resolution-logic"></a>
#### 解決ロジックのカスタマイズ

独自のモデルバインディング解決ロジックを定義したい場合は、`Route::bind`メソッドを使用できます。`bind`メソッドに渡すクロージャは、URIセグメントの値を受け取り、ルートに注入されるクラスのインスタンスを返す必要があります。このカスタマイズは、アプリケーションの`AppServiceProvider`の`boot`メソッドで行う必要があります:

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * アプリケーションサービスを起動します。
     */
    public function boot(): void
    {
        Route::bind('user', function (string $value) {
            return User::where('name', $value)->firstOrFail();
        });
    }

または、Eloquentモデルの`resolveRouteBinding`メソッドをオーバーライドすることもできます。このメソッドはURIセグメントの値を受け取り、ルートに注入されるクラスのインスタンスを返す必要があります:

```php
    /**
     * バインドされた値のモデルを取得する。
     *
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value, $field = null)
    {
        return $this->where('name', $value)->firstOrFail();
    }
```

もしルートが[暗黙のモデルバインディングスコープ](#implicit-model-binding-scoping)を利用している場合、`resolveChildRouteBinding`メソッドが親モデルの子バインディングを解決するために使用されます。

```php
    /**
     * バインドされた値の子モデルを取得する。
     *
     * @param  string  $childType
     * @param  mixed  $value
     * @param  string|null  $field
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveChildRouteBinding($childType, $value, $field)
    {
        return parent::resolveChildRouteBinding($childType, $value, $field);
    }
```

<a name="fallback-routes"></a>
## フォールバックルート

`Route::fallback`メソッドを使用することで、他のどのルートにも一致しない場合に実行されるルートを定義できます。通常、未処理のリクエストはアプリケーションの例外ハンドラを介して自動的に「404」ページをレンダリングします。しかし、`fallback`ルートは通常`routes/web.php`ファイル内で定義されるため、`web`ミドルウェアグループ内のすべてのミドルウェアがルートに適用されます。必要に応じて、このルートに追加のミドルウェアを自由に追加できます。

```php
Route::fallback(function () {
    // ...
});
```

> WARNING:  
> フォールバックルートは常にアプリケーションによって登録される最後のルートであるべきです。

<a name="rate-limiting"></a>
## レートリミット

<a name="defining-rate-limiters"></a>
### レートリミッタの定義

Laravelには、特定のルートまたはルートグループのトラフィック量を制限するために利用できる強力でカスタマイズ可能なレートリミットサービスが含まれています。まず、アプリケーションのニーズに合うレートリミッタの設定を定義する必要があります。

レートリミッタは、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッド内で定義できます。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * 任意のアプリケーションサービスをブートストラップする。
 */
protected function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

レートリミッタは、`RateLimiter`ファサードの`for`メソッドを使用して定義されます。`for`メソッドは、レートリミッタ名と、ルートに割り当てられたレートリミッタに適用する制限設定を返すクロージャを受け取ります。制限設定は`Illuminate\Cache\RateLimiting\Limit`クラスのインスタンスです。このクラスには、制限を迅速に定義できる便利な「ビルダー」メソッドが含まれています。レートリミッタ名は、あなたが希望する任意の文字列です。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * 任意のアプリケーションサービスをブートストラップする。
 */
protected function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });
}
```

着信リクエストが指定されたレート制限を超えた場合、Laravelは自動的に429 HTTPステータスコードのレスポンスを返します。レート制限によって返されるべき独自のレスポンスを定義したい場合は、`response`メソッドを使用できます。

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('Custom response...', 429, $headers);
    });
});
```

レートリミッタのクロージャは着信HTTPリクエストインスタンスを受け取るため、着信リクエストまたは認証済みユーザーに基づいて適切なレート制限を動的に構築できます。

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});
```

<a name="segmenting-rate-limits"></a>
#### レート制限のセグメント化

時には、任意の値によってレート制限をセグメント化したい場合があります。例えば、ユーザーがIPアドレスごとに1分間に特定のルートに100回アクセスできるようにしたい場合です。これを実現するには、レート制限を構築する際に`by`メソッドを使用します。

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100)->by($request->ip());
});
```

この機能を別の例で説明すると、認証済みユーザーIDごとに1分間に100回、ゲストの場合はIPアドレスごとに1分間に10回アクセスを制限できます。

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
});
```

<a name="multiple-rate-limits"></a>
#### 複数のレート制限

必要に応じて、特定のレートリミッタ設定に対してレート制限の配列を返すことができます。各レート制限は、配列内に配置された順序に基づいてルートに対して評価されます。

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

<a name="attaching-rate-limiters-to-routes"></a>
### ルートへのレートリミッタのアタッチ

レートリミッタは、`throttle`[ミドルウェア](middleware.md)を使用してルートまたはルートグループにアタッチできます。throttleミドルウェアは、ルートに割り当てたいレートリミッタの名前を受け取ります。

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});
```

<a name="throttling-with-redis"></a>
#### Redisによるスロットリング

デフォルトでは、`throttle`ミドルウェアは`Illuminate\Routing\Middleware\ThrottleRequests`クラスにマッピングされています。ただし、アプリケーションのキャッシュドライバとしてRedisを使用している場合、LaravelにRedisを使用してレート制限を管理するよう指示したい場合があります。そのためには、アプリケーションの`bootstrap/app.php`ファイルで`throttleWithRedis`メソッドを使用する必要があります。このメソッドは、`throttle`ミドルウェアを`Illuminate\Routing\Middleware\ThrottleRequestsWithRedis`ミドルウェアクラスにマッピングします。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->throttleWithRedis();
    // ...
})
```

<a name="form-method-spoofing"></a>
## フォームメソッドのスプーフィング

HTMLフォームは`PUT`、`PATCH`、または`DELETE`アクションをサポートしていません。したがって、HTMLフォームから呼び出される`PUT`、`PATCH`、または`DELETE`ルートを定義する場合、フォームに隠し`_method`フィールドを追加する必要があります。`_method`フィールドの値はHTTPリクエストメソッドとして使用されます。

```html
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

便宜上、`@method`[Bladeディレクティブ](blade.md)を使用して`_method`入力フィールドを生成できます。

```html
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>
```

<a name="accessing-the-current-route"></a>
## 現在のルートへのアクセス

`Route`ファサードの`current`、`currentRouteName`、および`currentRouteAction`メソッドを使用して、着信リクエストを処理しているルートに関する情報にアクセスできます。

```php
use Illuminate\Support\Facades\Route;

$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

ルーターとルートクラスで利用可能なすべてのメソッドを確認するには、[Routeファサードの基底クラス](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)と[Routeインスタンス](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html)のAPIドキュメントを参照してください。

<a name="cors"></a>
## クロスオリジンリソース共有 (CORS)

Laravelは、設定した値に基づいてCORS `OPTIONS` HTTPリクエストに自動的に応答できます。`OPTIONS`リクエストは、アプリケーションのグローバルミドルウェアスタックに自動的に含まれる`HandleCors`[ミドルウェア](middleware.md)によって自動的に処理されます。

時には、アプリケーションのCORS設定値をカスタマイズする必要があるかもしれません。これを行うには、`config:publish` Artisanコマンドを使用して`cors`設定ファイルを公開します。

```shell
php artisan config:publish cors
```

このコマンドは、アプリケーションの`config`ディレクトリ内に`cors.php`設定ファイルを配置します。

> NOTE:  
> CORSとCORSヘッダーの詳細については、[MDNのCORSに関するドキュメント](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers)を参照してください。

<a name="route-caching"></a>
## ルートキャッシュ

アプリケーションを本番環境にデプロイする際、Laravelのルートキャッシュを利用する必要があります。ルートキャッシュを使用すると、アプリケーションのすべてのルートを登録するのにかかる時間が大幅に短縮されます。ルートキャッシュを生成するには、`route:cache` Artisanコマンドを実行します。

```shell
php artisan route:cache
```

このコマンドを実行すると、キャッシュされたルートファイルがすべてのリクエストで読み込まれます。新しいルートを追加する場合は、新しいルートキャッシュを生成する必要があることに注意してください。このため、`route:cache`コマンドはプロジェクトのデプロイ中にのみ実行するべきです。

ルートキャッシュをクリアするには、`route:clear`コマンドを使用します。

```shell
php artisan route:clear
```

