# HTTPリクエスト

- [はじめに](#introduction)
- [リクエストとのやり取り](#interacting-with-the-request)
    - [リクエストの取得](#accessing-the-request)
    - [リクエストパスとメソッド](#request-path-and-method)
    - [リクエストヘッダ](#request-headers)
    - [リクエストIPアドレス](#request-ip-address)
    - [コンテンツネゴシエーション](#content-negotiation)
    - [PSR-7リクエスト](#psr7-requests)
- [入力](#input)
    - [入力の取得](#retrieving-input)
    - [入力の存在確認](#input-presence)
    - [追加入力のマージ](#merging-additional-input)
    - [古い入力](#old-input)
    - [クッキー](#cookies)
    - [入力のトリミングと正規化](#input-trimming-and-normalization)
- [ファイル](#files)
    - [アップロードされたファイルの取得](#retrieving-uploaded-files)
    - [アップロードされたファイルの保存](#storing-uploaded-files)
- [信頼できるプロキシの設定](#configuring-trusted-proxies)
- [信頼できるホストの設定](#configuring-trusted-hosts)

<a name="introduction"></a>
## はじめに

Laravelの`Illuminate\Http\Request`クラスは、アプリケーションが処理している現在のHTTPリクエストと、リクエストと共に送信された入力、クッキー、ファイルを取得するためのオブジェクト指向の方法を提供します。

<a name="interacting-with-the-request"></a>
## リクエストとのやり取り

<a name="accessing-the-request"></a>
### リクエストの取得

依存性注入を介して現在のHTTPリクエストのインスタンスを取得するには、ルートクロージャまたはコントローラメソッドに`Illuminate\Http\Request`クラスをタイプヒントする必要があります。受信リクエストインスタンスは、Laravelの[サービスコンテナ](container.md)によって自動的に注入されます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 新しいユーザーを保存します。
         */
        public function store(Request $request): RedirectResponse
        {
            $name = $request->input('name');

            // ユーザーを保存...

            return redirect('/users');
        }
    }

前述のように、ルートクロージャに`Illuminate\Http\Request`クラスをタイプヒントすることもできます。サービスコンテナは、クロージャが実行されるときに自動的に受信リクエストを注入します。

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

<a name="dependency-injection-route-parameters"></a>
#### 依存性注入とルートパラメータ

コントローラメソッドがルートパラメータからの入力も期待している場合は、他の依存関係の後にルートパラメータをリストする必要があります。たとえば、ルートが次のように定義されている場合：

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

`Illuminate\Http\Request`をタイプヒントし、`id`ルートパラメータにアクセスするには、コントローラメソッドを次のように定義します：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーを更新します。
         */
        public function update(Request $request, string $id): RedirectResponse
        {
            // ユーザーを更新...

            return redirect('/users');
        }
    }

<a name="request-path-and-method"></a>
### リクエストパスとメソッド

`Illuminate\Http\Request`インスタンスは、受信HTTPリクエストを検査するためのさまざまなメソッドを提供し、`Symfony\Component\HttpFoundation\Request`クラスを拡張します。以下では、最も重要なメソッドのいくつかについて説明します。

<a name="retrieving-the-request-path"></a>
#### リクエストパスの取得

`path`メソッドは、リクエストのパス情報を返します。したがって、受信リクエストが`http://example.com/foo/bar`をターゲットにしている場合、`path`メソッドは`foo/bar`を返します：

    $uri = $request->path();

<a name="inspecting-the-request-path"></a>
#### リクエストパス/ルートの検査

`is`メソッドを使用すると、受信リクエストパスが指定されたパターンと一致するかどうかを確認できます。このメソッドを使用する際には、ワイルドカードとして`*`文字を使用できます：

    if ($request->is('admin/*')) {
        // ...
    }

`routeIs`メソッドを使用すると、受信リクエストが[名前付きルート](routing.md#named-routes)と一致したかどうかを判断できます：

    if ($request->routeIs('admin.*')) {
        // ...
    }

<a name="retrieving-the-request-url"></a>
#### リクエストURLの取得

受信リクエストの完全なURLを取得するには、`url`または`fullUrl`メソッドを使用できます。`url`メソッドはクエリ文字列なしでURLを返し、`fullUrl`メソッドはクエリ文字列を含みます：

    $url = $request->url();

    $urlWithQueryString = $request->fullUrl();

現在のURLにクエリ文字列データを追加する場合は、`fullUrlWithQuery`メソッドを呼び出すことができます。このメソッドは、指定されたクエリ文字列変数の配列を現在のクエリ文字列とマージします：

    $request->fullUrlWithQuery(['type' => 'phone']);

指定されたクエリ文字列パラメータなしで現在のURLを取得する場合は、`fullUrlWithoutQuery`メソッドを使用できます：

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="retrieving-the-request-host"></a>
#### リクエストホストの取得

受信リクエストの「ホスト」を取得するには、`host`、`httpHost`、および`schemeAndHttpHost`メソッドを使用できます：

    $request->host();
    $request->httpHost();
    $request->schemeAndHttpHost();

<a name="retrieving-the-request-method"></a>
#### リクエストメソッドの取得

`method`メソッドは、リクエストのHTTP動詞を返します。`isMethod`メソッドを使用して、HTTP動詞が指定された文字列と一致するかどうかを確認できます：

    $method = $request->method();

    if ($request->isMethod('post')) {
        // ...
    }

<a name="request-headers"></a>
### リクエストヘッダ

`Illuminate\Http\Request`インスタンスからリクエストヘッダを取得するには、`header`メソッドを使用できます。リクエストにヘッダが存在しない場合、`null`が返されます。ただし、`header`メソッドは、リクエストにヘッダが存在しない場合に返されるオプションの2番目の引数を受け入れます：

    $value = $request->header('X-Header-Name');

    $value = $request->header('X-Header-Name', 'default');

`hasHeader`メソッドを使用して、リクエストに指定されたヘッダが含まれているかどうかを判断できます：

    if ($request->hasHeader('X-Header-Name')) {
        // ...
    }

便宜上、`bearerToken`メソッドを使用して、`Authorization`ヘッダからベアラートークンを取得できます。そのようなヘッダが存在しない場合、空の文字列が返されます：

    $token = $request->bearerToken();

<a name="request-ip-address"></a>
### リクエストIPアドレス

`ip`メソッドを使用して、アプリケーションにリクエストを送信したクライアントのIPアドレスを取得できます：

    $ipAddress = $request->ip();

プロキシによって転送されたすべてのクライアントIPアドレスを含むIPアドレスの配列を取得する場合は、`ips`メソッドを使用できます。「元の」クライアントIPアドレスは配列の最後にあります：

    $ipAddresses = $request->ips();

一般に、IPアドレスは信頼できない、ユーザー制御の入力と見なされ、情報提供の目的でのみ使用されるべきです。

<a name="content-negotiation"></a>
### コンテンツネゴシエーション

Laravelは、`Accept`ヘッダを介して受信リクエストの要求されたコンテンツタイプを検査するためのいくつかのメソッドを提供します。最初に、`getAcceptableContentTypes`メソッドは、リクエストによって受け入れられるすべてのコンテンツタイプを含む配列を返します：

    $contentTypes = $request->getAcceptableContentTypes();

`accepts`メソッドは、コンテンツタイプの配列を受け入れ、リクエストによっていずれかのコンテンツタイプが受け入れられる場合に`true`を返します。それ以外の場合、`false`が返されます：

    if ($request->accepts(['text/html', 'application/json'])) {
        // ...
    }

`prefers`メソッドを使用して、指定されたコンテンツタイプの配列の中で、リクエストによって最も好まれるコンテンツタイプを判断できます。提供されたコンテンツタイプのいずれもリクエストによって受け入れられない場合、`null`が返されます：

    $preferred = $request->prefers(['text/html', 'application/json']);

多くのアプリケーションはHTMLまたはJSONのみを提供するため、`expectsJson`メソッドを使用して、受信リクエストがJSONレスポンスを期待しているかどうかを迅速に判断できます：

    if ($request->expectsJson()) {
        // ...
    }

<a name="psr7-requests"></a>
### PSR-7リクエスト

[PSR-7標準](https://www.php-fig.org/psr/psr-7/)は、リクエストやレスポンスなどのHTTPメッセージのインターフェースを指定します。Laravelリクエストの代わりにPSR-7リクエストのインスタンスを取得したい場合は、最初にいくつかのライブラリをインストールする必要があります。Laravelは、*Symfony HTTP Message Bridge*コンポーネントを使用して、通常のLaravelリクエストとレスポンスをPSR-7互換の実装に変換します：

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

これらのライブラリをインストールしたら、ルートクロージャまたはコントローラメソッドにリクエストインターフェースをタイプヒントすることで、PSR-7リクエストを取得できます：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        // ...
    });

> NOTE:  
> ルートまたはコントローラからPSR-7レスポンスインスタンスを返すと、自動的にLaravelレスポンスインスタンスに変換され、フレームワークによって表示されます。

<a name="input"></a>
## 入力

<a name="retrieving-input"></a>
### 入力の取得

<a name="retrieving-all-input-data"></a>
#### すべての入力データの取得

`all`メソッドを使用して、受信リクエストのすべての入力データを`array`として取得できます。このメソッドは、受信リクエストがHTMLフォームからのものであるか、XHRリクエストであるかに関係なく使用できます：

    $input = $request->all();

`collect`メソッドを使用すると、受信リクエストのすべての入力データを[コレクション](collections.md)として取得できます。

    $input = $request->collect();

`collect`メソッドを使用して、受信リクエストの入力のサブセットをコレクションとして取得することもできます。

    $request->collect('users')->each(function (string $user) {
        // ...
    });

<a name="retrieving-an-input-value"></a>
#### 入力値の取得

いくつかの簡単なメソッドを使用して、リクエストに使用されたHTTPメソッドを気にすることなく、`Illuminate\Http\Request`インスタンスからすべてのユーザー入力にアクセスできます。HTTPメソッドに関係なく、`input`メソッドを使用してユーザー入力を取得できます。

    $name = $request->input('name');

`input`メソッドの第2引数としてデフォルト値を渡すことができます。リクエストに要求された入力値が存在しない場合、この値が返されます。

    $name = $request->input('name', 'Sally');

配列入力を含むフォームを操作する場合、"ドット"記法を使用して配列にアクセスできます。

    $name = $request->input('products.0.name');

    $names = $request->input('products.*.name');

引数なしで`input`メソッドを呼び出すことで、すべての入力値を連想配列として取得できます。

    $input = $request->input();

<a name="retrieving-input-from-the-query-string"></a>
#### クエリ文字列からの入力取得

`input`メソッドはクエリ文字列を含むリクエストペイロード全体から値を取得しますが、`query`メソッドはクエリ文字列からのみ値を取得します。

    $name = $request->query('name');

要求されたクエリ文字列の値が存在しない場合、このメソッドの第2引数が返されます。

    $name = $request->query('name', 'Helen');

引数なしで`query`メソッドを呼び出すことで、すべてのクエリ文字列の値を連想配列として取得できます。

    $query = $request->query();

<a name="retrieving-json-input-values"></a>
#### JSON入力値の取得

アプリケーションにJSONリクエストを送信する場合、`Content-Type`ヘッダーが適切に`application/json`に設定されていれば、`input`メソッドを介してJSONデータにアクセスできます。JSON配列/オブジェクト内にネストされた値を取得するために、"ドット"記法を使用することもできます。

    $name = $request->input('user.name');

<a name="retrieving-stringable-input-values"></a>
#### 文字列化可能な入力値の取得

リクエストの入力データをプリミティブな`string`としてではなく、`string`メソッドを使用して[`Illuminate\Support\Stringable`](strings.md)のインスタンスとして取得することができます。

    $name = $request->string('name')->trim();

<a name="retrieving-integer-input-values"></a>
#### 整数入力値の取得

入力値を整数として取得するには、`integer`メソッドを使用できます。このメソッドは入力値を整数にキャストしようとします。入力が存在しないかキャストに失敗した場合、指定したデフォルト値を返します。これはページネーションやその他の数値入力に特に便利です。

    $perPage = $request->integer('per_page');

<a name="retrieving-boolean-input-values"></a>
#### ブール入力値の取得

チェックボックスのようなHTML要素を扱う場合、アプリケーションは実際には文字列である"真"の値を受け取ることがあります。例えば、"true"や"on"です。便宜上、`boolean`メソッドを使用してこれらの値をブール値として取得できます。`boolean`メソッドは、1、"1"、true、"true"、"on"、"yes"に対して`true`を返します。その他のすべての値は`false`を返します。

    $archived = $request->boolean('archived');

<a name="retrieving-date-input-values"></a>
#### 日付入力値の取得

便宜上、日付/時刻を含む入力値は、`date`メソッドを使用してCarbonインスタンスとして取得できます。リクエストに指定された名前の入力値が存在しない場合、`null`が返されます。

    $birthday = $request->date('birthday');

`date`メソッドの第2引数と第3引数は、それぞれ日付のフォーマットとタイムゾーンを指定するために使用できます。

    $elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');

入力値が存在するが無効なフォーマットの場合、`InvalidArgumentException`がスローされます。したがって、`date`メソッドを呼び出す前に入力を検証することをお勧めします。

<a name="retrieving-enum-input-values"></a>
#### 列挙型入力値の取得

[PHPの列挙型](https://www.php.net/manual/en/language.types.enumerations.php)に対応する入力値もリクエストから取得できます。リクエストに指定された名前の入力値が存在しないか、列挙型に入力値に一致するバッキング値がない場合、`null`が返されます。`enum`メソッドは、入力値の名前と列挙型クラスを第1引数と第2引数として受け取ります。

    use App\Enums\Status;

    $status = $request->enum('status', Status::class);

<a name="retrieving-input-via-dynamic-properties"></a>
#### 動的プロパティによる入力の取得

`Illuminate\Http\Request`インスタンスの動的プロパティを使用してユーザー入力にアクセスすることもできます。例えば、アプリケーションのフォームに`name`フィールドが含まれている場合、次のようにフィールドの値にアクセスできます。

    $name = $request->name;

動的プロパティを使用する場合、Laravelは最初にリクエストペイロード内でパラメータの値を探します。存在しない場合、Laravelはマッチしたルートのパラメータ内でフィールドを探します。

<a name="retrieving-a-portion-of-the-input-data"></a>
#### 入力データの一部の取得

入力データのサブセットを取得する必要がある場合、`only`メソッドと`except`メソッドを使用できます。これらのメソッドは、単一の`array`または動的な引数リストを受け取ります。

    $input = $request->only(['username', 'password']);

    $input = $request->only('username', 'password');

    $input = $request->except(['credit_card']);

    $input = $request->except('credit_card');

> WARNING:  
> `only`メソッドは要求したすべてのキー/値のペアを返しますが、リクエストに存在しないキー/値のペアは返しません。

<a name="input-presence"></a>
### 入力の存在

`has`メソッドを使用して、リクエストに値が存在するかどうかを判断できます。`has`メソッドは、値がリクエストに存在する場合に`true`を返します。

    if ($request->has('name')) {
        // ...
    }

配列を指定すると、`has`メソッドは指定されたすべての値が存在するかどうかを判断します。

    if ($request->has(['name', 'email'])) {
        // ...
    }

`hasAny`メソッドは、指定された値のいずれかが存在する場合に`true`を返します。

    if ($request->hasAny(['name', 'email'])) {
        // ...
    }

`whenHas`メソッドは、リクエストに値が存在する場合に指定されたクロージャを実行します。

    $request->whenHas('name', function (string $input) {
        // ...
    });

`whenHas`メソッドには、指定された値がリクエストに存在しない場合に実行される第2クロージャを渡すことができます。

    $request->whenHas('name', function (string $input) {
        // "name"値が存在する...
    }, function () {
        // "name"値が存在しない...
    });

リクエストに値が存在し、空の文字列でないかどうかを判断したい場合は、`filled`メソッドを使用できます。

    if ($request->filled('name')) {
        // ...
    }

リクエストに値が存在しないか、空の文字列であるかどうかを判断したい場合は、`isNotFilled`メソッドを使用できます。

    if ($request->isNotFilled('name')) {
        // ...
    }

配列を指定すると、`isNotFilled`メソッドは指定されたすべての値が存在しないか空であるかどうかを判断します。

    if ($request->isNotFilled(['name', 'email'])) {
        // ...
    }

`anyFilled`メソッドは、指定された値のいずれかが空の文字列でない場合に`true`を返します。

    if ($request->anyFilled(['name', 'email'])) {
        // ...
    }

`whenFilled`メソッドは、リクエストに値が存在し、空の文字列でない場合に指定されたクロージャを実行します。

    $request->whenFilled('name', function (string $input) {
        // ...
    });

`whenFilled`メソッドには、指定された値が"filled"でない場合に実行される第2クロージャを渡すことができます。

    $request->whenFilled('name', function (string $input) {
        // "name"値がfilled...
    }, function () {
        // "name"値がfilledでない...
    });

リクエストに指定されたキーが存在しないかどうかを判断するには、`missing`メソッドと`whenMissing`メソッドを使用できます。

    if ($request->missing('name')) {
        // ...
    }

    $request->whenMissing('name', function () {
        // "name"値が存在しない...
    }, function () {
        // "name"値が存在する...
    });

<a name="merging-additional-input"></a>
### 追加入力のマージ

リクエストの既存の入力データに手動で追加入力をマージする必要がある場合、`merge`メソッドを使用できます。指定された入力キーがリクエストに既に存在する場合、`merge`メソッドに提供されたデータによって上書きされます。

    $request->merge(['votes' => 0]);

`mergeIfMissing`メソッドは、対応するキーがリクエストの入力データ内にまだ存在しない場合に入力をリクエストにマージするために使用できます。

    $request->mergeIfMissing(['votes' => 0]);

<a name="old-input"></a>
### 古い入力

Laravelでは、次のリクエスト中に1つのリクエストからの入力を保持することができます。この機能は、検証エラーを検出した後にフォームを再入力する場合に特に便利です。ただし、Laravelの組み込みの[検証機能](validation.md)を使用している場合、これらのセッション入力フラッシュメソッドを直接使用する必要がない可能性があります。Laravelの一部の組み込み検証機能は、自動的にこれらのメソッドを呼び出すためです。

<a name="flashing-input-to-the-session"></a>
#### セッションへの入力のフラッシュ

セッションに入力をフラッシュする方法については、Laravelのドキュメントを参照してください。

`Illuminate\Http\Request`クラスの`flash`メソッドは、現在の入力を[セッション](session.md)に保存します。これにより、ユーザーが次にアプリケーションにリクエストを送信したときにそのデータが利用可能になります。

    $request->flash();

また、`flashOnly`と`flashExcept`メソッドを使用して、リクエストデータの一部をセッションに保存することもできます。これらのメソッドは、パスワードなどの機密情報をセッションから除外するのに便利です。

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

<a name="flashing-input-then-redirecting"></a>
#### 入力を保存してからリダイレクト

多くの場合、入力をセッションに保存してから前のページにリダイレクトすることがあります。`withInput`メソッドを使用して、リダイレクトに入力保存を簡単に連鎖させることができます。

    return redirect('/form')->withInput();

    return redirect()->route('user.create')->withInput();

    return redirect('/form')->withInput(
        $request->except('password')
    );

<a name="retrieving-old-input"></a>
#### 古い入力の取得

前回のリクエストから保存された入力を取得するには、`Illuminate\Http\Request`のインスタンスで`old`メソッドを呼び出します。`old`メソッドは、以前に保存された入力データを[セッション](session.md)から取得します。

    $username = $request->old('username');

Laravelはまた、グローバルな`old`ヘルパーも提供しています。[Bladeテンプレート](blade.md)内で古い入力を表示する場合、`old`ヘルパーを使用してフォームを再入力すると便利です。指定されたフィールドに古い入力が存在しない場合、`null`が返されます。

    <input type="text" name="username" value="{{ old('username') }}">

<a name="cookies"></a>
### クッキー

<a name="retrieving-cookies-from-requests"></a>
#### リクエストからクッキーを取得

Laravelフレームワークによって作成されたすべてのクッキーは、認証コードで暗号化されて署名されています。つまり、クライアントによって変更された場合、無効と見なされます。リクエストからクッキー値を取得するには、`Illuminate\Http\Request`インスタンスの`cookie`メソッドを使用します。

    $value = $request->cookie('name');

<a name="input-trimming-and-normalization"></a>
## 入力のトリミングと正規化

デフォルトでは、Laravelにはアプリケーションのグローバルミドルウェアスタックに`Illuminate\Foundation\Http\Middleware\TrimStrings`と`Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull`ミドルウェアが含まれています。これらのミドルウェアは、リクエストのすべての文字列入力フィールドを自動的にトリミングし、空の文字列フィールドを`null`に変換します。これにより、ルートやコントローラでこれらの正規化の問題を気にする必要がなくなります。

#### 入力の正規化を無効にする

すべてのリクエストに対してこの動作を無効にしたい場合は、アプリケーションのミドルウェアスタックから2つのミドルウェアを削除することができます。これは、アプリケーションの`bootstrap/app.php`ファイルで`$middleware->remove`メソッドを呼び出すことで行います。

    use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
    use Illuminate\Foundation\Http\Middleware\TrimStrings;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->remove([
            ConvertEmptyStringsToNull::class,
            TrimStrings::class,
        ]);
    })

アプリケーションへのリクエストのサブセットに対して文字列のトリミングと空の文字列の変換を無効にしたい場合は、アプリケーションの`bootstrap/app.php`ファイル内で`trimStrings`と`convertEmptyStringsToNull`ミドルウェアメソッドを使用できます。どちらのメソッドも、入力の正規化をスキップするかどうかを示す`true`または`false`を返すクロージャの配列を受け取ります。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->convertEmptyStringsToNull(except: [
            fn (Request $request) => $request->is('admin/*'),
        ]);

        $middleware->trimStrings(except: [
            fn (Request $request) => $request->is('admin/*'),
        ]);
    })

<a name="files"></a>
## ファイル

<a name="retrieving-uploaded-files"></a>
### アップロードされたファイルの取得

`Illuminate\Http\Request`インスタンスからアップロードされたファイルを取得するには、`file`メソッドまたは動的プロパティを使用します。`file`メソッドは、PHPの`SplFileInfo`クラスを拡張し、ファイルと対話するためのさまざまなメソッドを提供する`Illuminate\Http\UploadedFile`クラスのインスタンスを返します。

    $file = $request->file('photo');

    $file = $request->photo;

`hasFile`メソッドを使用して、リクエストにファイルが存在するかどうかを確認できます。

    if ($request->hasFile('photo')) {
        // ...
    }

<a name="validating-successful-uploads"></a>
#### アップロードの成功を検証

ファイルが存在するかどうかを確認するだけでなく、`isValid`メソッドを介してファイルのアップロードに問題がなかったかどうかを検証できます。

    if ($request->file('photo')->isValid()) {
        // ...
    }

<a name="file-paths-extensions"></a>
#### ファイルパスと拡張子

`UploadedFile`クラスには、ファイルの完全修飾パスとその拡張子にアクセスするためのメソッドも含まれています。`extension`メソッドは、ファイルの内容に基づいてファイルの拡張子を推測しようとします。この拡張子は、クライアントから提供された拡張子と異なる場合があります。

    $path = $request->photo->path();

    $extension = $request->photo->extension();

<a name="other-file-methods"></a>
#### その他のファイルメソッド

`UploadedFile`インスタンスには、他にもさまざまなメソッドが利用可能です。これらのメソッドに関する詳細情報については、[クラスのAPIドキュメント](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php)を確認してください。

<a name="storing-uploaded-files"></a>
### アップロードされたファイルの保存

アップロードされたファイルを保存するには、通常、設定された[ファイルシステム](filesystem.md)のいずれかを使用します。`UploadedFile`クラスには、アップロードされたファイルをディスクの1つに移動する`store`メソッドがあります。これは、ローカルファイルシステム上の場所やAmazon S3などのクラウドストレージ場所である可能性があります。

`store`メソッドは、ファイルシステムの設定されたルートディレクトリからの相対パスでファイルを保存する場所を受け取ります。このパスにはファイル名を含めるべきではありません。一意のIDが自動的に生成され、ファイル名として使用されます。

`store`メソッドは、ファイルを保存するために使用するディスクの名前を指定するためのオプションの2番目の引数も受け取ります。このメソッドは、ディスクのルートからの相対パスを返します。

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

自動生成されたファイル名を使用したくない場合は、`storeAs`メソッドを使用できます。このメソッドは、パス、ファイル名、ディスク名を引数として受け取ります。

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

> NOTE:  
> Laravelでのファイルストレージの詳細については、完全な[ファイルストレージドキュメント](filesystem.md)を確認してください。

<a name="configuring-trusted-proxies"></a>
## 信頼できるプロキシの設定

TLS / SSL証明書を終端するロードバランサーの背後でアプリケーションを実行している場合、`url`ヘルパーを使用するときにアプリケーションがHTTPSリンクを生成しないことに気付くかもしれません。通常、これはアプリケーションがロードバランサーからポート80でトラフィックを転送されており、安全なリンクを生成する必要があることを認識していないためです。

これを解決するために、Laravelアプリケーションに含まれる`Illuminate\Http\Middleware\TrustProxies`ミドルウェアを有効にすることができます。これにより、アプリケーションが信頼するロードバランサーやプロキシをすばやくカスタマイズできます。信頼できるプロキシは、アプリケーションの`bootstrap/app.php`ファイルで`trustProxies`ミドルウェアメソッドを使用して指定する必要があります。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(at: [
            '192.168.1.1',
            '10.0.0.0/8',
        ]);
    })

信頼できるプロキシを設定するだけでなく、信頼するプロキシヘッダーも設定できます。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
            Request::HEADER_X_FORWARDED_HOST |
            Request::HEADER_X_FORWARDED_PORT |
            Request::HEADER_X_FORWARDED_PROTO |
            Request::HEADER_X_FORWARDED_AWS_ELB
        );
    })

> NOTE:  
> AWS Elastic Load Balancingを使用している場合、`headers`の値は`Request::HEADER_X_FORWARDED_AWS_ELB`である必要があります。ロードバランサーが[RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4)の標準`Forwarded`ヘッダーを使用している場合、`headers`の値は`Request::HEADER_FORWARDED`である必要があります。`headers`の値で使用できる定数の詳細については、Symfonyの[信頼できるプロキシ](https://symfony.com/doc/7.0/deployment/proxies.html)に関するドキュメントを確認してください。

<a name="trusting-all-proxies"></a>
#### すべてのプロキシを信頼

Amazon AWSや他の「クラウド」ロードバランサープロバイダーを使用している場合、実際のバランサーのIPアドレスがわからない場合があります。この場合、`*`を使用してすべてのプロキシを信頼することができます。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(at: '*');
    })

<a name="configuring-trusted-hosts"></a>
## 信頼できるホストの設定

デフォルトでは、Laravelは受信するすべてのリクエストに応答し、HTTPリクエストの`Host`ヘッダーの内容に関係なく、アプリケーションへの絶対URLを生成する際に`Host`ヘッダーの値を使用します。

通常、この動作は望ましいものですが、一部の環境では、アプリケーションが特定のホスト名のみに応答するように制限する必要がある場合があります。これを実現するために、Laravelには`Illuminate\Http\Middleware\TrustHosts`ミドルウェアが含まれています。このミドルウェアを有効にすると、アプリケーションが信頼するホストを指定できます。

`TrustHosts` ミドルウェアを有効にするには、アプリケーションの `bootstrap/app.php` ファイルで `trustHosts` ミドルウェアメソッドを呼び出す必要があります。 このメソッドの `at` 引数を使用して、アプリケーションが応答すべきホスト名を指定することができます。 その他の `Host` ヘッダーを含む受信リクエストは拒否されます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test']);
})
```

デフォルトでは、アプリケーションのURLのサブドメインからのリクエストも自動的に信頼されます。この動作を無効にしたい場合は、`subdomains`引数を使用できます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test'], subdomains: false);
})
```

信頼されるホストを決定するためにアプリケーションの設定ファイルやデータベースにアクセスする必要がある場合は、`at`引数にクロージャを提供できます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: fn () => config('app.trusted_hosts'));
})
```

