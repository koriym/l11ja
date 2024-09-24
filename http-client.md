# HTTPクライアント

- [はじめに](#introduction)
- [リクエストの作成](#making-requests)
    - [リクエストデータ](#request-data)
    - [ヘッダー](#headers)
    - [認証](#authentication)
    - [タイムアウト](#timeout)
    - [リトライ](#retries)
    - [エラーハンドリング](#error-handling)
    - [Guzzleミドルウェア](#guzzle-middleware)
    - [Guzzleオプション](#guzzle-options)
- [同時リクエスト](#concurrent-requests)
- [マクロ](#macros)
- [テスト](#testing)
    - [レスポンスのフェイク](#faking-responses)
    - [リクエストの検査](#inspecting-requests)
    - [ストレイリクエストの防止](#preventing-stray-requests)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

Laravelは、[Guzzle HTTPクライアント](http://docs.guzzlephp.org/en/stable/)を中心に、表現力豊かで最小限のAPIを提供しており、他のWebアプリケーションと通信するために、すばやく送信HTTPリクエストを作成できます。LaravelのGuzzleラッパーは、最も一般的なユースケースと素晴らしい開発者体験に焦点を当てています。

<a name="making-requests"></a>
## リクエストの作成

リクエストを作成するには、`Http`ファサードが提供する`head`、`get`、`post`、`put`、`patch`、`delete`メソッドを使用できます。まず、他のURLに対して基本的な`GET`リクエストを作成する方法を見てみましょう。

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

`get`メソッドは`Illuminate\Http\Client\Response`のインスタンスを返し、これにはレスポンスを検査するために使用できるさまざまなメソッドが用意されています。

    $response->body() : string;
    $response->json($key = null, $default = null) : mixed;
    $response->object() : object;
    $response->collect($key = null) : Illuminate\Support\Collection;
    $response->resource() : resource;
    $response->status() : int;
    $response->successful() : bool;
    $response->redirect(): bool;
    $response->failed() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

`Illuminate\Http\Client\Response`オブジェクトはPHPの`ArrayAccess`インターフェースも実装しているため、レスポンス上のJSONレスポンスデータに直接アクセスできます。

    return Http::get('http://example.com/users/1')['name'];

上記のレスポンスメソッドに加えて、以下のメソッドを使用して、レスポンスが特定のステータスコードを持っているかどうかを判断できます。

    $response->ok() : bool;                  // 200 OK
    $response->created() : bool;             // 201 Created
    $response->accepted() : bool;            // 202 Accepted
    $response->noContent() : bool;           // 204 No Content
    $response->movedPermanently() : bool;    // 301 Moved Permanently
    $response->found() : bool;               // 302 Found
    $response->badRequest() : bool;          // 400 Bad Request
    $response->unauthorized() : bool;        // 401 Unauthorized
    $response->paymentRequired() : bool;     // 402 Payment Required
    $response->forbidden() : bool;           // 403 Forbidden
    $response->notFound() : bool;            // 404 Not Found
    $response->requestTimeout() : bool;      // 408 Request Timeout
    $response->conflict() : bool;            // 409 Conflict
    $response->unprocessableEntity() : bool; // 422 Unprocessable Entity
    $response->tooManyRequests() : bool;     // 429 Too Many Requests
    $response->serverError() : bool;         // 500 Internal Server Error

<a name="uri-templates"></a>
#### URIテンプレート

HTTPクライアントでは、[URIテンプレート仕様](https://www.rfc-editor.org/rfc/rfc6570)を使用してリクエストURLを構築することもできます。URIテンプレートで展開可能なURLパラメータを定義するには、`withUrlParameters`メソッドを使用できます。

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '11.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### リクエストのダンプ

送信前に送信リクエストインスタンスをダンプし、スクリプトの実行を終了したい場合は、リクエスト定義の先頭に`dd`メソッドを追加できます。

    return Http::dd()->get('http://example.com');

<a name="request-data"></a>
### リクエストデータ

もちろん、`POST`、`PUT`、`PATCH`リクエストを作成する際に、リクエストと一緒に追加のデータを送信するのが一般的です。これらのメソッドは、データの配列を2番目の引数として受け取ります。デフォルトでは、データは`application/json`コンテンツタイプを使用して送信されます。

    use Illuminate\Support\Facades\Http;

    $response = Http::post('http://example.com/users', [
        'name' => 'Steve',
        'role' => 'Network Administrator',
    ]);

<a name="get-request-query-parameters"></a>
#### GETリクエストのクエリパラメータ

`GET`リクエストを作成する際に、クエリ文字列をURLに直接追加するか、`get`メソッドの2番目の引数としてキー/値のペアの配列を渡すことができます。

    $response = Http::get('http://example.com/users', [
        'name' => 'Taylor',
        'page' => 1,
    ]);

または、`withQueryParameters`メソッドを使用することもできます。

    Http::retry(3, 100)->withQueryParameters([
        'name' => 'Taylor',
        'page' => 1,
    ])->get('http://example.com/users')

<a name="sending-form-url-encoded-requests"></a>
#### フォームURLエンコードリクエストの送信

`application/x-www-form-urlencoded`コンテンツタイプを使用してデータを送信したい場合は、リクエストを作成する前に`asForm`メソッドを呼び出す必要があります。

    $response = Http::asForm()->post('http://example.com/users', [
        'name' => 'Sara',
        'role' => 'Privacy Consultant',
    ]);

<a name="sending-a-raw-request-body"></a>
#### 生のリクエストボディの送信

リクエストを作成する際に生のリクエストボディを提供したい場合は、`withBody`メソッドを使用できます。コンテンツタイプはメソッドの2番目の引数を介して提供できます。

    $response = Http::withBody(
        base64_encode($photo), 'image/jpeg'
    )->post('http://example.com/photo');

<a name="multi-part-requests"></a>
#### マルチパートリクエスト

ファイルをマルチパートリクエストとして送信したい場合は、リクエストを作成する前に`attach`メソッドを呼び出す必要があります。このメソッドは、ファイルの名前とその内容を受け取ります。必要に応じて、3番目の引数をファイル名として、4番目の引数をファイルに関連付けられたヘッダーとして提供できます。

    $response = Http::attach(
        'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
    )->post('http://example.com/attachments');

ファイルの生の内容の代わりに、ストリームリソースを渡すこともできます。

    $photo = fopen('photo.jpg', 'r');

    $response = Http::attach(
        'attachment', $photo, 'photo.jpg'
    )->post('http://example.com/attachments');

<a name="headers"></a>
### ヘッダー

`withHeaders`メソッドを使用してリクエストにヘッダーを追加できます。この`withHeaders`メソッドは、キー/値のペアの配列を受け取ります。

    $response = Http::withHeaders([
        'X-First' => 'foo',
        'X-Second' => 'bar'
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
    ]);

アプリケーションがリクエストに対して期待するコンテンツタイプを指定するには、`accept`メソッドを使用できます。

    $response = Http::accept('application/json')->get('http://example.com/users');

利便性のために、`acceptJson`メソッドを使用して、アプリケーションがリクエストに対して`application/json`コンテンツタイプを期待していることをすばやく指定できます。

    $response = Http::acceptJson()->get('http://example.com/users');

`withHeaders`メソッドは、新しいヘッダーをリクエストの既存のヘッダーにマージします。必要に応じて、`replaceHeaders`メソッドを使用してすべてのヘッダーを完全に置き換えることができます。

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### 認証

`withBasicAuth`メソッドと`withDigestAuth`メソッドをそれぞれ使用して、基本認証とダイジェスト認証の資格情報を指定できます。

    // 基本認証...
    $response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

    // ダイジェスト認証...
    $response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);

<a name="bearer-tokens"></a>
#### ベアラートークン

リクエストの`Authorization`ヘッダーにベアラートークンをすばやく追加したい場合は、`withToken`メソッドを使用できます。

    $response = Http::withToken('token')->post(/* ... */);

<a name="timeout"></a>
### タイムアウト

`timeout`メソッドを使用して、レスポンスを待つ最大秒数を指定できます。デフォルトでは、HTTPクライアントは30秒後にタイムアウトします。

    $response = Http::timeout(3)->get(/* ... */);

指定されたタイムアウトを超えると、`Illuminate\Http\Client\ConnectionException`のインスタンスがスローされます。

サーバーへの接続を試みる間に待機する最大秒数を指定するには、`connectTimeout`メソッドを使用できます。

    $response = Http::connectTimeout(3)->get(/* ... */);

<a name="retries"></a>
### リトライ

HTTPクライアントがクライアントまたはサーバーエラーが発生した場合にリクエストを自動的に再試行するようにしたい場合は、`retry`メソッドを使用できます。`retry`メソッドは、リクエストを試行する最大回数とLaravelが再試行間に待機するミリ秒数を受け取ります。

    $response = Http::retry(3, 100)->post(/* ... */);

再試行間に待機するミリ秒数を手動で計算したい場合は、`retry`メソッドの2番目の引数としてクロージャを渡すことができます。

    use Exception;

    $response = Http::retry(3, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    })->post(/* ... */);

便宜上、`retry` メソッドの最初の引数として配列を提供することもできます。この配列は、後続の試行の間に何ミリ秒スリープするかを決定するために使用されます：

    $response = Http::retry([100, 200])->post(/* ... */);

必要に応じて、`retry` メソッドに3番目の引数を渡すことができます。3番目の引数は、再試行を実際に試みるかどうかを決定する callable である必要があります。例えば、最初のリクエストが `ConnectionException` に遭遇した場合にのみリクエストを再試行したい場合があります：

    use Exception;
    use Illuminate\Http\Client\PendingRequest;

    $response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
        return $exception instanceof ConnectionException;
    })->post(/* ... */);

リクエストの試行が失敗した場合、新しい試行が行われる前にリクエストに変更を加えたい場合があります。これは、`retry` メソッドに提供した callable に渡されるリクエスト引数を変更することで実現できます。例えば、最初の試行が認証エラーを返した場合に新しい認証トークンでリクエストを再試行したい場合があります：

    use Exception;
    use Illuminate\Http\Client\PendingRequest;
    use Illuminate\Http\Client\RequestException;

    $response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
        if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
            return false;
        }

        $request->withToken($this->getNewToken());

        return true;
    })->post(/* ... */);

すべてのリクエストが失敗した場合、`Illuminate\Http\Client\RequestException` のインスタンスがスローされます。この動作を無効にしたい場合は、`throw` 引数に `false` の値を指定できます。無効にすると、すべての再試行が試行された後にクライアントが受信した最後のレスポンスが返されます：

    $response = Http::retry(3, 100, throw: false)->post(/* ... */);

> WARNING:  
> 接続の問題ですべてのリクエストが失敗した場合でも、`throw` 引数が `false` に設定されている場合でも、`Illuminate\Http\Client\ConnectionException` がスローされます。

<a name="error-handling"></a>
### エラー処理

Guzzle のデフォルトの動作とは異なり、Laravel の HTTP クライアントラッパーはクライアントまたはサーバーエラー（サーバーからの `400` および `500` レベルのレスポンス）で例外をスローしません。これらのエラーのいずれかが返されたかどうかは、`successful`、`clientError`、または `serverError` メソッドを使用して判断できます：

    // ステータスコードが >= 200 かつ < 300 かどうかを判定...
    $response->successful();

    // ステータスコードが >= 400 かどうかを判定...
    $response->failed();

    // レスポンスが 400 レベルのステータスコードを持つかどうかを判定...
    $response->clientError();

    // レスポンスが 500 レベルのステータスコードを持つかどうかを判定...
    $response->serverError();

    // クライアントまたはサーバーエラーが発生した場合、指定されたコールバックを即座に実行...
    $response->onError(callable $callback);

<a name="throwing-exceptions"></a>
#### 例外のスロー

レスポンスインスタンスがあり、レスポンスステータスコードがクライアントまたはサーバーエラーを示している場合に `Illuminate\Http\Client\RequestException` のインスタンスをスローしたい場合は、`throw` または `throwIf` メソッドを使用できます：

    use Illuminate\Http\Client\Response;

    $response = Http::post(/* ... */);

    // クライアントまたはサーバーエラーが発生した場合に例外をスロー...
    $response->throw();

    // エラーが発生し、指定された条件が true の場合に例外をスロー...
    $response->throwIf($condition);

    // エラーが発生し、指定されたクロージャが true を返す場合に例外をスロー...
    $response->throwIf(fn (Response $response) => true);

    // エラーが発生し、指定された条件が false の場合に例外をスロー...
    $response->throwUnless($condition);

    // エラーが発生し、指定されたクロージャが false を返す場合に例外をスロー...
    $response->throwUnless(fn (Response $response) => false);

    // レスポンスが特定のステータスコードを持つ場合に例外をスロー...
    $response->throwIfStatus(403);

    // レスポンスが特定のステータスコードを持たない場合に例外をスロー...
    $response->throwUnlessStatus(200);

    return $response['user']['id'];

`Illuminate\Http\Client\RequestException` インスタンスには、返されたレスポンスを検査できるようにするパブリックの `$response` プロパティがあります。

`throw` メソッドは、エラーが発生しなかった場合にレスポンスインスタンスを返し、`throw` メソッドに他の操作を連鎖させることができます：

    return Http::post(/* ... */)->throw()->json();

例外がスローされる前に追加のロジックを実行したい場合は、`throw` メソッドにクロージャを渡すことができます。クロージャが呼び出された後、例外は自動的にスローされるため、クロージャ内から例外を再スローする必要はありません：

    use Illuminate\Http\Client\Response;
    use Illuminate\Http\Client\RequestException;

    return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
        // ...
    })->json();

<a name="guzzle-middleware"></a>
### Guzzle ミドルウェア

Laravel の HTTP クライアントは Guzzle によって動作しているため、[Guzzle Middleware](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) を利用して、送信リクエストを操作したり、受信レスポンスを検査したりできます。送信リクエストを操作するために、`withRequestMiddleware` メソッドを介して Guzzle ミドルウェアを登録します：

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\RequestInterface;

    $response = Http::withRequestMiddleware(
        function (RequestInterface $request) {
            return $request->withHeader('X-Example', 'Value');
        }
    )->get('http://example.com');

同様に、`withResponseMiddleware` メソッドを介してミドルウェアを登録することで、受信 HTTP レスポンスを検査できます：

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\ResponseInterface;

    $response = Http::withResponseMiddleware(
        function (ResponseInterface $response) {
            $header = $response->getHeader('X-Example');

            // ...

            return $response;
        }
    )->get('http://example.com');

<a name="global-middleware"></a>
#### グローバルミドルウェア

すべての送信リクエストと受信レスポンスに適用されるミドルウェアを登録したい場合があります。これを実現するには、`globalRequestMiddleware` および `globalResponseMiddleware` メソッドを使用します。通常、これらのメソッドはアプリケーションの `AppServiceProvider` の `boot` メソッドで呼び出す必要があります：

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Guzzle オプション

`withOptions` メソッドを使用して、送信リクエストに追加の [Guzzle リクエストオプション](http://docs.guzzlephp.org/en/stable/request-options.html)を指定できます。`withOptions` メソッドは、キー/値ペアの配列を受け取ります：

    $response = Http::withOptions([
        'debug' => true,
    ])->get('http://example.com/users');

<a name="global-options"></a>
#### グローバルオプション

すべての送信リクエストに対してデフォルトのオプションを設定するには、`globalOptions` メソッドを利用できます。通常、このメソッドはアプリケーションの `AppServiceProvider` の `boot` メソッドから呼び出す必要があります：

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::globalOptions([
        'allow_redirects' => false,
    ]);
}
```

<a name="concurrent-requests"></a>
## 並行リクエスト

複数の HTTP リクエストを同時に行いたい場合があります。つまり、リクエストを順次ではなく同時に発行したい場合です。これにより、遅い HTTP API とのやり取りで大幅なパフォーマンス向上が見込めます。

幸いなことに、`pool` メソッドを使用してこれを実現できます。`pool` メソッドは、`Illuminate\Http\Client\Pool` インスタンスを受け取るクロージャを受け取り、リクエストプールにリクエストを簡単に追加して発行できます：

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->get('http://localhost/first'),
        $pool->get('http://localhost/second'),
        $pool->get('http://localhost/third'),
    ]);

    return $responses[0]->ok() &&
           $responses[1]->ok() &&
           $responses[2]->ok();

ご覧のとおり、各レスポンスインスタンスは、プールに追加された順序に基づいてアクセスできます。必要に応じて、`as` メソッドを使用してリクエストに名前を付けることができ、対応するレスポンスに名前でアクセスできます：

    use Illuminate\Http\Client\Pool;
    use Illuminate\Support\Facades\Http;

    $responses = Http::pool(fn (Pool $pool) => [
        $pool->as('first')->get('http://localhost/first'),
        $pool->as('second')->get('http://localhost/second'),
        $pool->as('third')->get('http://localhost/third'),
    ]);

    return $responses['first']->ok();

<a name="customizing-concurrent-requests"></a>
#### 並行リクエストのカスタマイズ

`pool` メソッドは、`withHeaders` や `middleware` メソッドなどの他の HTTP クライアントメソッドと連鎖できません。プールされたリクエストにカスタムヘッダーやミドルウェアを適用したい場合は、プール内の各リクエストでそれらのオプションを設定する必要があります：

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="macros"></a>
## マクロ

LaravelのHTTPクライアントでは、"マクロ"を定義できます。これは、アプリケーション全体でサービスとやり取りする際に、一般的なリクエストパスやヘッダーを設定するための流暢で表現力豊かなメカニズムとして機能します。始めるには、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッド内でマクロを定義します。

```php
use Illuminate\Support\Facades\Http;

/**
 * アプリケーションサービスのブートストラップ
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

マクロを設定したら、アプリケーション内のどこからでも、指定された設定で保留中のリクエストを作成するために呼び出すことができます。

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## テスト

多くのLaravelサービスは、テストを簡単かつ表現力豊かに書くための機能を提供しており、LaravelのHTTPクライアントも例外ではありません。`Http`ファサードの`fake`メソッドを使用すると、HTTPクライアントにリクエストが行われたときにスタブ/ダミーのレスポンスを返すように指示できます。

<a name="faking-responses"></a>
### レスポンスのフェイク

例えば、すべてのリクエストに対して空の`200`ステータスコードのレスポンスを返すようにHTTPクライアントに指示するには、引数なしで`fake`メソッドを呼び出します。

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(/* ... */);

<a name="faking-specific-urls"></a>
#### 特定のURLのフェイク

また、`fake`メソッドに配列を渡すこともできます。配列のキーはフェイクしたいURLパターンを表し、それに対応するレスポンスを指定します。`*`文字はワイルドカード文字として使用できます。フェイクされていないURLに対するリクエストは実際に実行されます。`Http`ファサードの`response`メソッドを使用して、これらのエンドポイントのスタブ/ダミーレスポンスを構築できます。

    Http::fake([
        // GitHubエンドポイントのJSONレスポンスをスタブする
        'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

        // Googleエンドポイントの文字列レスポンスをスタブする
        'google.com/*' => Http::response('Hello World', 200, $headers),
    ]);

すべての一致しないURLに対してフォールバックURLパターンを指定したい場合は、単一の`*`文字を使用できます。

    Http::fake([
        // GitHubエンドポイントのJSONレスポンスをスタブする
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // 他のすべてのエンドポイントの文字列レスポンスをスタブする
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

<a name="faking-response-sequences"></a>
#### レスポンスシーケンスのフェイク

特定のURLが特定の順序で一連のフェイクレスポンスを返す必要がある場合、`Http::sequence`メソッドを使用してレスポンスを構築できます。

    Http::fake([
        // GitHubエンドポイントの一連のレスポンスをスタブする
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->pushStatus(404),
    ]);

レスポンスシーケンス内のすべてのレスポンスが消費された場合、それ以降のリクエストはレスポンスシーケンスが例外をスローする原因となります。シーケンスが空の場合に返すデフォルトのレスポンスを指定したい場合は、`whenEmpty`メソッドを使用できます。

    Http::fake([
        // GitHubエンドポイントの一連のレスポンスをスタブする
        'github.com/*' => Http::sequence()
                                ->push('Hello World', 200)
                                ->push(['foo' => 'bar'], 200)
                                ->whenEmpty(Http::response()),
    ]);

レスポンスのシーケンスをフェイクしたいが、特定のURLパターンをフェイクする必要がない場合は、`Http::fakeSequence`メソッドを使用できます。

    Http::fakeSequence()
            ->push('Hello World', 200)
            ->whenEmpty(Http::response());

<a name="fake-callback"></a>
#### フェイクコールバック

特定のエンドポイントに対してどのようなレスポンスを返すかを決定するためにより複雑なロジックが必要な場合、`fake`メソッドにクロージャを渡すことができます。このクロージャは`Illuminate\Http\Client\Request`のインスタンスを受け取り、レスポンスインスタンスを返す必要があります。クロージャ内で、どのようなタイプのレスポンスを返すかを決定するために必要なロジックを実行できます。

    use Illuminate\Http\Client\Request;

    Http::fake(function (Request $request) {
        return Http::response('Hello World', 200);
    });

<a name="preventing-stray-requests"></a>
### ストレイリクエストの防止

HTTPクライアントを介して送信されるすべてのリクエストが、個々のテストまたは完全なテストスイート全体でフェイクされていることを確認したい場合、`preventStrayRequests`メソッドを呼び出すことができます。このメソッドを呼び出した後、対応するフェイクレスポンスがないリクエストは、実際のHTTPリクエストを行う代わりに例外をスローします。

    use Illuminate\Support\Facades\Http;

    Http::preventStrayRequests();

    Http::fake([
        'github.com/*' => Http::response('ok'),
    ]);

    // "ok"レスポンスが返される
    Http::get('https://github.com/laravel/framework');

    // 例外がスローされる
    Http::get('https://laravel.com');

<a name="inspecting-requests"></a>
### リクエストの検査

レスポンスをフェイクする際、クライアントが受け取るリクエストを検査して、アプリケーションが正しいデータやヘッダーを送信していることを確認したい場合があります。これを実現するには、`Http::fake`を呼び出した後に`Http::assertSent`メソッドを呼び出します。

`assertSent`メソッドは、`Illuminate\Http\Client\Request`インスタンスを受け取り、リクエストが期待に一致するかどうかを示すブール値を返すクロージャを受け取ります。テストがパスするためには、少なくとも1つのリクエストが指定された期待に一致する必要があります。

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::withHeaders([
        'X-First' => 'foo',
    ])->post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertSent(function (Request $request) {
        return $request->hasHeader('X-First', 'foo') &&
               $request->url() == 'http://example.com/users' &&
               $request['name'] == 'Taylor' &&
               $request['role'] == 'Developer';
    });

必要に応じて、`assertNotSent`メソッドを使用して特定のリクエストが送信されなかったことをアサートできます。

    use Illuminate\Http\Client\Request;
    use Illuminate\Support\Facades\Http;

    Http::fake();

    Http::post('http://example.com/users', [
        'name' => 'Taylor',
        'role' => 'Developer',
    ]);

    Http::assertNotSent(function (Request $request) {
        return $request->url() === 'http://example.com/posts';
    });

`assertSentCount`メソッドを使用して、テスト中に「送信」されたリクエストの数をアサートできます。

    Http::fake();

    Http::assertSentCount(5);

または、`assertNothingSent`メソッドを使用して、テスト中にリクエストが送信されなかったことをアサートできます。

    Http::fake();

    Http::assertNothingSent();

<a name="recording-requests-and-responses"></a>
#### リクエスト/レスポンスの記録

`recorded`メソッドを使用して、すべてのリクエストとそれに対応するレスポンスを収集できます。`recorded`メソッドは、`Illuminate\Http\Client\Request`と`Illuminate\Http\Client\Response`のインスタンスを含む配列のコレクションを返します。

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

さらに、`recorded`メソッドは、`Illuminate\Http\Client\Request`と`Illuminate\Http\Client\Response`のインスタンスを受け取るクロージャを受け取り、期待に基づいてリクエスト/レスポンスのペアをフィルタリングするために使用できます。

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="events"></a>
## イベント

Laravelは、HTTPリクエストの送信プロセス中に3つのイベントを発火します。`RequestSending`イベントはリクエストが送信される前に発火し、`ResponseReceived`イベントは特定のリクエストに対するレスポンスが受信された後に発火します。`ConnectionFailed`イベントは、特定のリクエストに対してレスポンスが受信されなかった場合に発火します。

`RequestSending`と`ConnectionFailed`イベントはどちらも、`Illuminate\Http\Client\Request`インスタンスを検査するために使用できるパブリックな`$request`プロパティを持っています。同様に、`ResponseReceived`イベントは`$request`プロパティと`$response`プロパティを持っており、`Illuminate\Http\Client\Response`インスタンスを検査するために使用できます。これらのイベントに対して[イベントリスナー](events.md)を作成できます。

    use Illuminate\Http\Client\Events\RequestSending;

    class LogRequest
    {
        /**
         * 指定されたイベントを処理します。
         */
        public function handle(RequestSending $event): void
        {
            // $event->request ...
        }
    }
