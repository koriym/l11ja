# URL生成

- [はじめに](#introduction)
- [基本](#the-basics)
    - [URLの生成](#generating-urls)
    - [現在のURLへのアクセス](#accessing-the-current-url)
- [名前付きルートのURL](#urls-for-named-routes)
    - [署名付きURL](#signed-urls)
- [コントローラアクションのURL](#urls-for-controller-actions)
- [デフォルト値](#default-values)

<a name="introduction"></a>
## はじめに

Laravelは、アプリケーションのURLを生成するのに役立ついくつかのヘルパーを提供しています。これらのヘルパーは、主にテンプレートやAPIレスポンスでリンクを構築するとき、またはアプリケーションの別の部分にリダイレクトレスポンスを生成するときに役立ちます。

<a name="the-basics"></a>
## 基本

<a name="generating-urls"></a>
### URLの生成

`url`ヘルパーは、アプリケーションの任意のURLを生成するために使用できます。生成されるURLは、現在処理されているリクエストのスキーム（HTTPまたはHTTPS）とホストを自動的に使用します。

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

クエリ文字列パラメータを含むURLを生成するには、`query`メソッドを使用できます。

    echo url()->query('/posts', ['search' => 'Laravel']);

    // https://example.com/posts?search=Laravel

    echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

    // http://example.com/posts?sort=latest&search=Laravel

パスに既に存在するクエリ文字列パラメータを提供すると、それらの既存の値が上書きされます。

    echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

    // http://example.com/posts?sort=oldest

値の配列をクエリパラメータとして渡すこともできます。これらの値は、生成されるURLで適切にキー付けされてエンコードされます。

    echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

    // http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

    echo urldecode($url);

    // http://example.com/posts?columns[0]=title&columns[1]=body

<a name="accessing-the-current-url"></a>
### 現在のURLへのアクセス

パスが`url`ヘルパーに提供されていない場合、`Illuminate\Routing\UrlGenerator`インスタンスが返され、現在のURLに関する情報にアクセスできます。

    // クエリ文字列なしの現在のURLを取得
    echo url()->current();

    // クエリ文字列を含む現在のURLを取得
    echo url()->full();

    // 前のリクエストの完全なURLを取得
    echo url()->previous();

これらの各メソッドは、`URL` [ファサード](facades.md)を介してもアクセスできます。

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## 名前付きルートのURL

`route`ヘルパーは、[名前付きルート](routing.md#named-routes)へのURLを生成するために使用できます。名前付きルートを使用すると、ルートに定義された実際のURLに結合せずにURLを生成できます。したがって、ルートのURLが変更されても、`route`関数の呼び出しを変更する必要はありません。例えば、アプリケーションに次のように定義されたルートが含まれているとします。

    Route::get('/post/{post}', function (Post $post) {
        // ...
    })->name('post.show');

このルートへのURLを生成するには、次のように`route`ヘルパーを使用できます。

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

もちろん、`route`ヘルパーは複数のパラメータを持つルートのURLを生成するためにも使用できます。

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        // ...
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

ルートの定義パラメータに対応しない追加の配列要素は、URLのクエリ文字列に追加されます。

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Eloquentモデル

URLを生成する際に、[Eloquentモデル](eloquent.md)のルートキー（通常は主キー）を使用することがよくあります。このため、Eloquentモデルをパラメータ値として渡すことができます。`route`ヘルパーは、モデルのルートキーを自動的に抽出します。

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### 署名付きURL

Laravelを使用すると、名前付きルートへの「署名付き」URLを簡単に作成できます。これらのURLには、クエリ文字列に「署名」ハッシュが追加され、LaravelがURLが作成されてから変更されていないことを確認できます。署名付きURLは、特にURL操作に対する保護レイヤーが必要な公開アクセス可能なルートに役立ちます。

例えば、顧客にメールで送信する公開の「退会」リンクを実装するために、署名付きURLを使用できます。名前付きルートへの署名付きURLを作成するには、`URL`ファサードの`signedRoute`メソッドを使用します。

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

署名付きURLハッシュからドメインを除外するには、`signedRoute`メソッドに`absolute`引数を指定します。

    return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);

指定された時間が経過した後に期限切れになる一時的な署名付きルートURLを生成する場合は、`temporarySignedRoute`メソッドを使用できます。Laravelが一時的な署名付きルートURLを検証するとき、署名付きURLにエンコードされた有効期限のタイムスタンプが経過していないことを確認します。

    use Illuminate\Support\Facades\URL;

    return URL::temporarySignedRoute(
        'unsubscribe', now()->addMinutes(30), ['user' => 1]
    );

<a name="validating-signed-route-requests"></a>
#### 署名付きルートリクエストの検証

受信リクエストに有効な署名があるかどうかを確認するには、受信`Illuminate\Http\Request`インスタンスで`hasValidSignature`メソッドを呼び出す必要があります。

    use Illuminate\Http\Request;

    Route::get('/unsubscribe/{user}', function (Request $request) {
        if (! $request->hasValidSignature()) {
            abort(401);
        }

        // ...
    })->name('unsubscribe');

クライアントサイドのページネーションなど、アプリケーションのフロントエンドが署名付きURLにデータを追加できるようにする必要がある場合があります。したがって、`hasValidSignatureWhileIgnoring`メソッドを使用して、署名付きURLの検証時に無視するリクエストクエリパラメータを指定できます。無視されたパラメータは、リクエスト上で誰でも変更できることに注意してください。

    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }

受信リクエストインスタンスを使用して署名付きURLを検証する代わりに、`signed` (`Illuminate\Routing\Middleware\ValidateSignature`) [ミドルウェア](middleware.md)をルートに割り当てることができます。受信リクエストに有効な署名がない場合、ミドルウェアは自動的に`403` HTTPレスポンスを返します。

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');

署名付きURLにドメインがURLハッシュに含まれていない場合、ミドルウェアに`relative`引数を指定する必要があります。

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed:relative');

<a name="responding-to-invalid-signed-routes"></a>
#### 無効な署名付きルートへの対応

誰かが期限切れの署名付きURLにアクセスすると、`403` HTTPステータスコードの一般的なエラーページが表示されます。ただし、アプリケーションの`bootstrap/app.php`ファイルで`InvalidSignatureException`例外のカスタム「レンダリング」クロージャを定義することで、この動作をカスタマイズできます。

    use Illuminate\Routing\Exceptions\InvalidSignatureException;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->render(function (InvalidSignatureException $e) {
            return response()->view('errors.link-expired', status: 403);
        });
    })

<a name="urls-for-controller-actions"></a>
## コントローラアクションのURL

`action`関数は、指定されたコントローラアクションのURLを生成します。

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

コントローラメソッドがルートパラメータを受け取る場合、関数の2番目の引数としてルートパラメータの連想配列を渡すことができます。

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="default-values"></a>
## デフォルト値

一部のアプリケーションでは、特定のURLパラメータに対してリクエスト全体のデフォルト値を指定したい場合があります。例えば、多くのルートが`{locale}`パラメータを定義しているとします。

    Route::get('/{locale}/posts', function () {
        // ...
    })->name('post.index');

`locale`を毎回`route`ヘルパーに渡すのは面倒です。そこで、`URL::defaults`メソッドを使用して、現在のリクエスト中に常に適用されるこのパラメータのデフォルト値を定義できます。[ルートミドルウェア](middleware.md#assigning-middleware-to-routes)からこのメソッドを呼び出して、現在のリクエストにアクセスできるようにすることをお勧めします。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\URL;
    use Symfony\Component\HttpFoundation\Response;

    class SetDefaultLocaleForUrls
    {
        /**
         * 受信リクエストの処理
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            URL::defaults(['locale' => $request->user()->locale]);

            return $next($request);
        }
    }

`locale`パラメータのデフォルト値が設定されると、`route`ヘルパーを介してURLを生成する際にその値を渡す必要がなくなります。

<a name="url-defaults-middleware-priority"></a>
#### URLのデフォルト値とミドルウェアの優先順位

URLのデフォルト値を設定すると、Laravelの暗黙のモデルバインディングの処理に干渉する可能性があります。そのため、URLのデフォルト値を設定するミドルウェアを、Laravel自身の`SubstituteBindings`ミドルウェアよりも前に実行するように[優先順位を設定](middleware.md#sorting-middleware)する必要があります。これは、アプリケーションの`bootstrap/app.php`ファイル内で`priority`ミドルウェアメソッドを使用して実現できます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \App\Http\Middleware\SetDefaultLocaleForUrls::class, // [tl! add]
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

