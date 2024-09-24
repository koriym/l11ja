# ミドルウェア

- [イントロダクション](#introduction)
- [ミドルウェアの定義](#defining-middleware)
- [ミドルウェアの登録](#registering-middleware)
    - [グローバルミドルウェア](#global-middleware)
    - [ルートへのミドルウェアの割り当て](#assigning-middleware-to-routes)
    - [ミドルウェアグループ](#middleware-groups)
    - [ミドルウェアエイリアス](#middleware-aliases)
    - [ミドルウェアのソート](#sorting-middleware)
- [ミドルウェアパラメータ](#middleware-parameters)
- [終了可能なミドルウェア](#terminable-middleware)

<a name="introduction"></a>
## イントロダクション

ミドルウェアは、アプリケーションに入るHTTPリクエストを検査およびフィルタリングするための便利なメカニズムを提供します。たとえば、Laravelには、アプリケーションのユーザーが認証されているかどうかを確認するミドルウェアが含まれています。ユーザーが認証されていない場合、ミドルウェアはユーザーをアプリケーションのログイン画面にリダイレクトします。ただし、ユーザーが認証されている場合、ミドルウェアはリクエストがアプリケーションに進むことを許可します。

認証以外にも、さまざまなタスクを実行するために追加のミドルウェアを記述できます。たとえば、ロギングミドルウェアは、アプリケーションへのすべての受信リクエストをログに記録する場合があります。Laravelには、認証やCSRF保護のためのミドルウェアが含まれています。ただし、すべてのユーザー定義ミドルウェアは、通常、アプリケーションの`app/Http/Middleware`ディレクトリに配置されます。

<a name="defining-middleware"></a>
## ミドルウェアの定義

新しいミドルウェアを作成するには、`make:middleware` Artisanコマンドを使用します。

```shell
php artisan make:middleware EnsureTokenIsValid
```

このコマンドは、`app/Http/Middleware`ディレクトリ内に新しい`EnsureTokenIsValid`クラスを配置します。このミドルウェアでは、提供された`token`入力が指定された値と一致する場合にのみ、ルートへのアクセスを許可します。それ以外の場合、ユーザーを`/home` URIにリダイレクトします。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureTokenIsValid
    {
        /**
         * 受信リクエストを処理します。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->input('token') !== 'my-secret-token') {
                return redirect('/home');
            }

            return $next($request);
        }
    }

ご覧のとおり、与えられた`token`がシークレットトークンと一致しない場合、ミドルウェアはクライアントにHTTPリダイレクトを返します。それ以外の場合、リクエストはアプリケーションにさらに渡されます。リクエストをアプリケーションにさらに渡す（ミドルウェアが「通過」できるようにする）には、`$next`コールバックに`$request`を渡す必要があります。

ミドルウェアは、HTTPリクエストがアプリケーションに到達する前に通過しなければならない一連の「層」として想像するのが最善です。各層はリクエストを検査し、完全に拒否することもできます。

> NOTE:  
> すべてのミドルウェアは[サービスコンテナ](container.md)を介して解決されるため、ミドルウェアのコンストラクタ内で必要な依存関係をタイプヒントで指定できます。

<a name="middleware-and-responses"></a>
#### ミドルウェアとレスポンス

もちろん、ミドルウェアは、リクエストをアプリケーションにさらに渡す前後にタスクを実行できます。たとえば、次のミドルウェアは、リクエストがアプリケーションによって処理される**前に**いくつかのタスクを実行します。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class BeforeMiddleware
    {
        public function handle(Request $request, Closure $next): Response
        {
            // タスクを実行

            return $next($request);
        }
    }

ただし、このミドルウェアは、リクエストがアプリケーションによって処理された**後に**そのタスクを実行します。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class AfterMiddleware
    {
        public function handle(Request $request, Closure $next): Response
        {
            $response = $next($request);

            // タスクを実行

            return $response;
        }
    }

<a name="registering-middleware"></a>
## ミドルウェアの登録

<a name="global-middleware"></a>
### グローバルミドルウェア

アプリケーションへのすべてのHTTPリクエスト中にミドルウェアを実行したい場合は、アプリケーションの`bootstrap/app.php`ファイル内のグローバルミドルウェアスタックに追加できます。

    use App\Http\Middleware\EnsureTokenIsValid;

    ->withMiddleware(function (Middleware $middleware) {
         $middleware->append(EnsureTokenIsValid::class);
    })

`withMiddleware`クロージャに提供される`$middleware`オブジェクトは、`Illuminate\Foundation\Configuration\Middleware`のインスタンスであり、アプリケーションのルートに割り当てられたミドルウェアを管理する役割を担います。`append`メソッドは、ミドルウェアをグローバルミドルウェアのリストの末尾に追加します。リストの先頭にミドルウェアを追加したい場合は、`prepend`メソッドを使用する必要があります。

<a name="manually-managing-laravels-default-global-middleware"></a>
#### Laravelのデフォルトグローバルミドルウェアの手動管理

Laravelのグローバルミドルウェアスタックを手動で管理したい場合は、Laravelのデフォルトのグローバルミドルウェアスタックを`use`メソッドに提供できます。そして、必要に応じてデフォルトのミドルウェアスタックを調整できます。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->use([
            // \Illuminate\Http\Middleware\TrustHosts::class,
            \Illuminate\Http\Middleware\TrustProxies::class,
            \Illuminate\Http\Middleware\HandleCors::class,
            \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
            \Illuminate\Http\Middleware\ValidatePostSize::class,
            \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
            \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        ]);
    })

<a name="assigning-middleware-to-routes"></a>
### ルートへのミドルウェアの割り当て

特定のルートにミドルウェアを割り当てたい場合は、ルートを定義するときに`middleware`メソッドを呼び出すことができます。

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::get('/profile', function () {
        // ...
    })->middleware(EnsureTokenIsValid::class);

`middleware`メソッドにミドルウェア名の配列を渡すことで、複数のミドルウェアをルートに割り当てることができます。

    Route::get('/', function () {
        // ...
    })->middleware([First::class, Second::class]);

<a name="excluding-middleware"></a>
#### ミドルウェアの除外

ルートのグループにミドルウェアを割り当てる場合、グループ内の個々のルートにミドルウェアを適用しないようにする必要がある場合があります。`withoutMiddleware`メソッドを使用してこれを実現できます。

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::middleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/', function () {
            // ...
        });

        Route::get('/profile', function () {
            // ...
        })->withoutMiddleware([EnsureTokenIsValid::class]);
    });

また、特定のミドルウェアのセットを[グループ](routing.md#route-groups)全体のルート定義から除外することもできます。

    use App\Http\Middleware\EnsureTokenIsValid;

    Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
        Route::get('/profile', function () {
            // ...
        });
    });

`withoutMiddleware`メソッドはルートミドルウェアのみを削除し、[グローバルミドルウェア](#global-middleware)には適用されません。

<a name="middleware-groups"></a>
### ミドルウェアグループ

複数のミドルウェアを1つのキーの下にグループ化して、ルートに割り当てやすくすることができます。これは、アプリケーションの`bootstrap/app.php`ファイル内で`appendToGroup`メソッドを使用して実現できます。

    use App\Http\Middleware\First;
    use App\Http\Middleware\Second;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->appendToGroup('group-name', [
            First::class,
            Second::class,
        ]);

        $middleware->prependToGroup('group-name', [
            First::class,
            Second::class,
        ]);
    })

ミドルウェアグループは、個々のミドルウェアと同じ構文を使用してルートとコントローラアクションに割り当てることができます。

    Route::get('/', function () {
        // ...
    })->middleware('group-name');

    Route::middleware(['group-name'])->group(function () {
        // ...
    });

<a name="laravels-default-middleware-groups"></a>
#### Laravelのデフォルトミドルウェアグループ

Laravelには、一般的なミドルウェアをWebルートとAPIルートに適用するための事前定義された`web`および`api`ミドルウェアグループが含まれています。Laravelは、対応する`routes/web.php`および`routes/api.php`ファイルにこれらのミドルウェアグループを自動的に適用することを覚えておいてください。

<div class="overflow-auto" markdown=1>

| `web`ミドルウェアグループ |
| --- |
| `Illuminate\Cookie\Middleware\EncryptCookies` |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession` |
| `Illuminate\View\Middleware\ShareErrorsFromSession` |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

<div class="overflow-auto" markdown=1>

| `api`ミドルウェアグループ |
| --- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

これらのグループにミドルウェアを追加または先頭に追加する場合は、アプリケーションの`bootstrap/app.php`ファイル内で`web`および`api`メソッドを使用できます。`web`および`api`メソッドは、`appendToGroup`メソッドの便利な代替手段です。


```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

Laravelのデフォルトのミドルウェアグループのエントリの1つを、独自のカスタムミドルウェアに置き換えることもできます。

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

または、ミドルウェアを完全に削除することもできます。

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### Laravelのデフォルトミドルウェアグループの手動管理

Laravelのデフォルトの`web`と`api`ミドルウェアグループ内のすべてのミドルウェアを手動で管理したい場合は、グループを完全に再定義することができます。以下の例では、`web`と`api`ミドルウェアグループをデフォルトのミドルウェアで定義し、必要に応じてカスタマイズできるようにします。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```

> NOTE:  
> デフォルトでは、`web`と`api`ミドルウェアグループは、`bootstrap/app.php`ファイルによって、アプリケーションの対応する`routes/web.php`と`routes/api.php`ファイルに自動的に適用されます。

<a name="middleware-aliases"></a>
### ミドルウェアのエイリアス

アプリケーションの`bootstrap/app.php`ファイルでミドルウェアにエイリアスを割り当てることができます。ミドルウェアのエイリアスを使用すると、長いクラス名を持つミドルウェアに短いエイリアスを定義できます。

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

ミドルウェアのエイリアスがアプリケーションの`bootstrap/app.php`ファイルで定義されると、ルートにミドルウェアを割り当てる際にエイリアスを使用できます。

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

便宜上、Laravelの一部の組み込みミドルウェアはデフォルトでエイリアスが設定されています。例えば、`auth`ミドルウェアは`Illuminate\Auth\Middleware\Authenticate`ミドルウェアのエイリアスです。以下はデフォルトのミドルウェアエイリアスのリストです。

<div class="overflow-auto" markdown=1>

| エイリアス | ミドルウェア |
| --- | --- |
| `auth` | `Illuminate\Auth\Middleware\Authenticate` |
| `auth.basic` | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth` |
| `auth.session` | `Illuminate\Session\Middleware\AuthenticateSession` |
| `cache.headers` | `Illuminate\Http\Middleware\SetCacheHeaders` |
| `can` | `Illuminate\Auth\Middleware\Authorize` |
| `guest` | `Illuminate\Auth\Middleware\RedirectIfAuthenticated` |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword` |
| `precognitive` | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests` |
| `signed` | `Illuminate\Routing\Middleware\ValidateSignature` |
| `subscribed` | `\Spark\Http\Middleware\VerifyBillableIsSubscribed` |
| `throttle` | `Illuminate\Routing\Middleware\ThrottleRequests` または `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified` | `Illuminate\Auth\Middleware\EnsureEmailIsVerified` |

</div>

<a name="sorting-middleware"></a>
### ミドルウェアのソート

まれに、ミドルウェアを特定の順序で実行する必要があるが、ルートに割り当てられたときにその順序を制御できない場合があります。このような状況では、アプリケーションの`bootstrap/app.php`ファイルで`priority`メソッドを使用してミドルウェアの優先順位を指定できます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

<a name="middleware-parameters"></a>
## ミドルウェアのパラメータ

ミドルウェアは追加のパラメータを受け取ることもできます。例えば、アプリケーションが特定のアクションを実行する前に認証されたユーザーが特定の「役割」を持っていることを確認する必要がある場合、役割名を追加の引数として受け取る`EnsureUserHasRole`ミドルウェアを作成できます。

追加のミドルウェアパラメータは、`$next`引数の後に渡されます。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

ミドルウェアパラメータは、ルートを定義する際にミドルウェア名とパラメータを`:`で区切ることで指定できます。

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

複数のパラメータはカンマで区切ることができます。

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor,publisher');
```

<a name="terminable-middleware"></a>
## 終了可能なミドルウェア

HTTPレスポンスがブラウザに送信された後にミドルウェアが作業を行う必要がある場合があります。ミドルウェアに`terminate`メソッドを定義し、WebサーバーがFastCGIを使用している場合、`terminate`メソッドはレスポンスがブラウザに送信された後に自動的に呼び出されます。

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

`terminate`メソッドは、リクエストとレスポンスの両方を受け取る必要があります。終了可能なミドルウェアを定義したら、アプリケーションの`bootstrap/app.php`ファイルのルートまたはグローバルミドルウェアのリストに追加する必要があります。

ミドルウェアの`handle`メソッドと`terminate`メソッドが呼び出されるときに同じミドルウェアインスタンスを使用したい場合は、コンテナの`singleton`メソッドを使用してコンテナにミドルウェアを登録します。通常、これは`AppServiceProvider`の`register`メソッドで行うべきです。

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```

