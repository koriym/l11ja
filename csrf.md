# CSRF保護

- [はじめに](#csrf-introduction)
- [CSRFリクエストの防止](#preventing-csrf-requests)
    - [URIの除外](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## はじめに

クロスサイトリクエストフォージェリは、認証されたユーザーに代わって不正なコマンドが実行される悪意のある攻撃の一種です。幸いなことに、Laravelはアプリケーションを[クロスサイトリクエストフォージェリ](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻撃から簡単に保護できます。

<a name="csrf-explanation"></a>
#### 脆弱性の説明

クロスサイトリクエストフォージェリに慣れていない場合は、この脆弱性がどのように悪用されるかの例を説明しましょう。アプリケーションに認証されたユーザーのメールアドレスを変更するための `POST` リクエストを受け付ける `/user/email` ルートがあるとします。おそらく、このルートはユーザーが使用したいメールアドレスを含む `email` 入力フィールドを期待しています。

CSRF保護がない場合、悪意のあるウェブサイトはアプリケーションの `/user/email` ルートを指すHTMLフォームを作成し、悪意のあるユーザー自身のメールアドレスを送信することができます:

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

悪意のあるウェブサイトがページの読み込み時に自動的にフォームを送信する場合、悪意のあるユーザーはアプリケーションの信頼できないユーザーを自分のウェブサイトに誘導するだけで、アプリケーション内でメールアドレスが変更されることになります。

この脆弱性を防ぐために、悪意のあるアプリケーションがアクセスできない秘密のセッション値をすべての着信 `POST`、`PUT`、`PATCH`、または `DELETE` リクエストに対して検査する必要があります。

<a name="preventing-csrf-requests"></a>
## CSRFリクエストの防止

Laravelは、アプリケーションによって管理される各アクティブな[ユーザーセッション](session.md)に対して自動的にCSRF「トークン」を生成します。このトークンは、認証されたユーザーが実際にアプリケーションにリクエストを行っていることを確認するために使用されます。このトークンはユーザーのセッションに保存され、セッションが再生成されるたびに変更されるため、悪意のあるアプリケーションはアクセスできません。

現在のセッションのCSRFトークンは、リクエストのセッションまたは `csrf_token` ヘルパー関数を介してアクセスできます:

    use Illuminate\Http\Request;

    Route::get('/token', function (Request $request) {
        $token = $request->session()->token();

        $token = csrf_token();

        // ...
    });

アプリケーションで「POST」、「PUT」、「PATCH」、または「DELETE」HTMLフォームを定義するたびに、フォームに隠しCSRF `_token` フィールドを含めて、CSRF保護ミドルウェアがリクエストを検証できるようにする必要があります。便宜上、`@csrf` Bladeディレクティブを使用して隠しトークン入力フィールドを生成できます:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- これは以下と同等です... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` [ミドルウェア](middleware.md)は、デフォルトで `web` ミドルウェアグループに含まれており、リクエスト入力のトークンがセッションに保存されているトークンと一致することを自動的に検証します。これらの2つのトークンが一致する場合、認証されたユーザーがリクエストを開始していることがわかります。

<a name="csrf-tokens-and-spas"></a>
### CSRFトークンとSPA

LaravelをAPIバックエンドとして利用するSPAを構築している場合は、APIでの認証とCSRF脆弱性からの保護に関する情報について、[Laravel Sanctumのドキュメント](sanctum.md)を参照する必要があります。

<a name="csrf-excluding-uris"></a>
### URIの除外

CSRF保護から一連のURIを除外したい場合があります。たとえば、[Stripe](https://stripe.com)を使用して支払いを処理し、そのウェブフックシステムを利用している場合、StripeのウェブフックハンドラールートをCSRF保護から除外する必要があります。なぜなら、StripeはどのCSRFトークンをルートに送信すべきかわからないからです。

通常、このようなルートは、Laravelが `routes/web.php` ファイル内のすべてのルートに適用する `web` ミドルウェアグループの外に配置する必要があります。ただし、アプリケーションの `bootstrap/app.php` ファイルで `validateCsrfTokens` メソッドにURIを提供することで、特定のルートを除外することもできます:

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->validateCsrfTokens(except: [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ]);
    })

> NOTE:  
> 便宜上、[テストを実行](testing.md)する際にすべてのルートに対してCSRFミドルウェアは自動的に無効になります。

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` ミドルウェアは、デフォルトで `web` ミドルウェアグループに含まれており、CSRFトークンをPOSTパラメータとしてチェックするだけでなく、`X-CSRF-TOKEN` リクエストヘッダもチェックします。たとえば、トークンをHTML `meta` タグに保存できます:

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

そして、jQueryなどのライブラリにトークンをすべてのリクエストヘッダに自動的に追加するよう指示できます。これにより、レガシーなJavaScript技術を使用したAJAXベースのアプリケーションに対して、簡単で便利なCSRF保護が提供されます:

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravelは現在のCSRFトークンを暗号化された `XSRF-TOKEN` クッキーに保存し、フレームワークによって生成される各レスポンスに含めます。このクッキーの値を使用して `X-XSRF-TOKEN` リクエストヘッダを設定できます。

このクッキーは主に開発者の利便性のために送信されます。なぜなら、AngularやAxiosなどの一部のJavaScriptフレームワークやライブラリは、同一生成元のリクエストで自動的にその値を `X-XSRF-TOKEN` ヘッダに配置するからです。

> NOTE:  
> デフォルトでは、`resources/js/bootstrap.js` ファイルにはAxios HTTPライブラリが含まれており、自動的に `X-XSRF-TOKEN` ヘッダを送信します。

