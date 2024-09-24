# HTTPレスポンス

- [レスポンスの作成](#creating-responses)
    - [ヘッダーをレスポンスに添付する](#attaching-headers-to-responses)
    - [クッキーをレスポンスに添付する](#attaching-cookies-to-responses)
    - [クッキーと暗号化](#cookies-and-encryption)
- [リダイレクト](#redirects)
    - [名前付きルートへのリダイレクト](#redirecting-named-routes)
    - [コントローラアクションへのリダイレクト](#redirecting-controller-actions)
    - [外部ドメインへのリダイレクト](#redirecting-external-domains)
    - [セッションデータをフラッシュしてリダイレクト](#redirecting-with-flashed-session-data)
- [その他のレスポンスタイプ](#other-response-types)
    - [ビューレスポンス](#view-responses)
    - [JSONレスポンス](#json-responses)
    - [ファイルダウンロード](#file-downloads)
    - [ファイルレスポンス](#file-responses)
    - [ストリーミングレスポンス](#streamed-responses)
- [レスポンスマクロ](#response-macros)

<a name="creating-responses"></a>
## レスポンスの作成

<a name="strings-arrays"></a>
#### 文字列と配列

すべてのルートとコントローラは、ユーザーのブラウザに送り返されるレスポンスを返す必要があります。Laravelは、レスポンスを返すためのいくつかの異なる方法を提供しています。最も基本的なレスポンスは、ルートまたはコントローラから文字列を返すことです。フレームワークは自動的に文字列を完全なHTTPレスポンスに変換します：

    Route::get('/', function () {
        return 'Hello World';
    });

ルートやコントローラから文字列を返すだけでなく、配列を返すこともできます。フレームワークは自動的に配列をJSONレスポンスに変換します：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> NOTE:  
> ルートやコントローラから[Eloquentコレクション](eloquent-collections.md)を返すこともできることを知っていましたか？これらは自動的にJSONに変換されます。試してみてください！

<a name="response-objects"></a>
#### レスポンスオブジェクト

通常、ルートアクションから単純な文字列や配列を返すだけではありません。代わりに、完全な`Illuminate\Http\Response`インスタンスまたは[ビュー](views.md)を返します。

完全な`Response`インスタンスを返すことで、レスポンスのHTTPステータスコードやヘッダーをカスタマイズできます。`Response`インスタンスは`Symfony\Component\HttpFoundation\Response`クラスを継承しており、HTTPレスポンスを構築するためのさまざまなメソッドを提供しています：

    Route::get('/home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });

<a name="eloquent-models-and-collections"></a>
#### Eloquentモデルとコレクション

ルートやコントローラから直接[Eloquent ORM](eloquent.md)モデルとコレクションを返すこともできます。そうすると、Laravelは自動的にモデルとコレクションをJSONレスポンスに変換し、モデルの[非表示属性](eloquent-serialization.md#hiding-attributes-from-json)を尊重します：

    use App\Models\User;

    Route::get('/user/{user}', function (User $user) {
        return $user;
    });

<a name="attaching-headers-to-responses"></a>
### レスポンスにヘッダーを添付する

ほとんどのレスポンスメソッドはチェーン可能であり、レスポンスインスタンスを流暢に構築できます。たとえば、`header`メソッドを使用して、ユーザーに送り返す前に一連のヘッダーをレスポンスに追加できます：

    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');

または、`withHeaders`メソッドを使用して、レスポンスに追加するヘッダーの配列を指定できます：

    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);

<a name="cache-control-middleware"></a>
#### キャッシュコントロールミドルウェア

Laravelには`cache.headers`ミドルウェアが含まれており、これを使用してルートのグループに対して`Cache-Control`ヘッダーをすばやく設定できます。ディレクティブは、対応するキャッシュコントロールディレクティブの「スネークケース」相当で提供され、セミコロンで区切る必要があります。ディレクティブのリストに`etag`が指定されている場合、レスポンス内容のMD5ハッシュが自動的にETag識別子として設定されます：

    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('/privacy', function () {
            // ...
        });

        Route::get('/terms', function () {
            // ...
        });
    });

<a name="attaching-cookies-to-responses"></a>
### レスポンスにクッキーを添付する

送信される`Illuminate\Http\Response`インスタンスにクッキーを添付するには、`cookie`メソッドを使用します。このメソッドには、名前、値、およびクッキーが有効とみなされる分数を渡す必要があります：

    return response('Hello World')->cookie(
        'name', 'value', $minutes
    );

`cookie`メソッドは、より頻繁に使用されないいくつかの引数も受け入れます。一般に、これらの引数はPHPのネイティブ[setcookie](https://secure.php.net/manual/en/function.setcookie.php)メソッドに与えられる引数と同じ目的と意味を持っています：

    return response('Hello World')->cookie(
        'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
    );

送信されるレスポンスにクッキーを添付することを確認したいが、まだそのレスポンスのインスタンスがない場合は、`Cookie`ファサードを使用してクッキーを「キュー」に入れることができます。`queue`メソッドは、クッキーインスタンスを作成するために必要な引数を受け入れます。これらのクッキーは、ブラウザに送信される前に送信レスポンスに添付されます：

    use Illuminate\Support\Facades\Cookie;

    Cookie::queue('name', 'value', $minutes);

<a name="generating-cookie-instances"></a>
#### クッキーインスタンスの生成

後でレスポンスインスタンスに添付できる`Symfony\Component\HttpFoundation\Cookie`インスタンスを生成したい場合は、グローバルな`cookie`ヘルパーを使用できます。このクッキーは、レスポンスインスタンスに添付されない限り、クライアントに送り返されません：

    $cookie = cookie('name', 'value', $minutes);

    return response('Hello World')->cookie($cookie);

<a name="expiring-cookies-early"></a>
#### クッキーの早期期限切れ

送信レスポンスの`withoutCookie`メソッドを介してクッキーを期限切れにすることで、クッキーを削除できます：

    return response('Hello World')->withoutCookie('name');

送信レスポンスのインスタンスがまだない場合は、`Cookie`ファサードの`expire`メソッドを使用してクッキーを期限切れにすることができます：

    Cookie::expire('name');

<a name="cookies-and-encryption"></a>
### クッキーと暗号化

デフォルトでは、Laravelによって生成されるすべてのクッキーは、`Illuminate\Cookie\Middleware\EncryptCookies`ミドルウェアのおかげで暗号化され、署名されているため、クライアントによって変更または読み取られることはありません。アプリケーションによって生成されるクッキーのサブセットの暗号化を無効にしたい場合は、アプリケーションの`bootstrap/app.php`ファイルで`encryptCookies`メソッドを使用できます：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->encryptCookies(except: [
            'cookie_name',
        ]);
    })

<a name="redirects"></a>
## リダイレクト

リダイレクトレスポンスは`Illuminate\Http\RedirectResponse`クラスのインスタンスであり、ユーザーを別のURLにリダイレクトするために必要な適切なヘッダーを含んでいます。`RedirectResponse`インスタンスを生成する方法はいくつかあります。最も簡単な方法は、グローバルな`redirect`ヘルパーを使用することです：

    Route::get('/dashboard', function () {
        return redirect('/home/dashboard');
    });

ユーザーを以前の場所にリダイレクトしたい場合があります。たとえば、送信されたフォームが無効な場合などです。グローバルな`back`ヘルパー関数を使用してこれを行うことができます。この機能は[セッション](session.md)を利用しているため、`back`関数を呼び出すルートが`web`ミドルウェアグループを使用していることを確認してください：

    Route::post('/user/profile', function () {
        // リクエストの検証...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 名前付きルートへのリダイレクト

パラメータを指定せずに`redirect`ヘルパーを呼び出すと、`Illuminate\Routing\Redirector`のインスタンスが返され、`Redirector`インスタンスの任意のメソッドを呼び出すことができます。たとえば、名前付きルートへの`RedirectResponse`を生成するには、`route`メソッドを使用できます：

    return redirect()->route('login');

ルートにパラメータがある場合は、それらを`route`メソッドの2番目の引数として渡すことができます：

    // 次のようなURIのルートの場合: /profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### Eloquentモデルによるパラメータの設定

Eloquentモデルから「ID」パラメータを設定するルートにリダイレクトする場合は、モデル自体を渡すことができます。IDは自動的に抽出されます：

    // 次のようなURIのルートの場合: /profile/{id}

    return redirect()->route('profile', [$user]);

ルートパラメータに配置される値をカスタマイズしたい場合は、ルートパラメータ定義（`/profile/{id:slug}`）で列を指定するか、Eloquentモデルで`getRouteKey`メソッドをオーバーライドできます：

    /**
     * モデルのルートキーの値を取得します。
     */
    public function getRouteKey(): mixed
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### コントローラアクションへのリダイレクト

[コントローラアクション](controllers.md)へのリダイレクトを生成することもできます。そのためには、コントローラとアクションの名前を`action`メソッドに渡します：

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

もしコントローラーのルートがパラメータを必要とする場合、それらを`action`メソッドの第二引数として渡すことができます。

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### 外部ドメインへのリダイレクト

アプリケーション外のドメインにリダイレクトする必要がある場合があります。その場合、`away`メソッドを呼び出すことで、追加のURLエンコーディング、検証、または検証なしで`RedirectResponse`を作成できます。

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### セッションデータをフラッシュしてリダイレクト

新しいURLにリダイレクトし、[セッションにデータをフラッシュする](session.md#flash-data)ことは、通常同時に行われます。これは通常、アクションが正常に実行された後に成功メッセージをセッションにフラッシュする場合に行われます。便宜上、`RedirectResponse`インスタンスを作成し、セッションにデータをフラッシュすることができます。

    Route::post('/user/profile', function () {
        // ...

        return redirect('/dashboard')->with('status', 'Profile updated!');
    });

ユーザーがリダイレクトされた後、[セッション](session.md)からフラッシュされたメッセージを表示できます。例えば、[Blade構文](blade.md)を使用して：

    @if (session('status'))
        <div class="alert alert-success">
            {{ session('status') }}
        </div>
    @endif

<a name="redirecting-with-input"></a>
#### 入力を伴うリダイレクト

`RedirectResponse`インスタンスが提供する`withInput`メソッドを使用して、ユーザーを新しい場所にリダイレクトする前に現在のリクエストの入力データをセッションにフラッシュすることができます。これは通常、ユーザーがバリデーションエラーに遭遇した場合に行われます。入力がセッションにフラッシュされると、次のリクエスト中に簡単に[取得](requests.md#retrieving-old-input)してフォームを再入力することができます。

    return back()->withInput();

<a name="other-response-types"></a>
## その他のレスポンスタイプ

`response`ヘルパーは、他のタイプのレスポンスインスタンスを生成するために使用できます。`response`ヘルパーが引数なしで呼び出されると、`Illuminate\Contracts\Routing\ResponseFactory`[契約](contracts.md)の実装が返されます。この契約は、レスポンスを生成するためのいくつかの便利なメソッドを提供します。

<a name="view-responses"></a>
### ビューレスポンス

レスポンスのステータスとヘッダーを制御する必要があるが、レスポンスの内容として[ビュー](views.md)も返す必要がある場合は、`view`メソッドを使用する必要があります。

    return response()
                ->view('hello', $data, 200)
                ->header('Content-Type', $type);

もちろん、カスタムHTTPステータスコードやカスタムヘッダーを渡す必要がない場合は、グローバルな`view`ヘルパー関数を使用できます。

<a name="json-responses"></a>
### JSONレスポンス

`json`メソッドは、`Content-Type`ヘッダーを自動的に`application/json`に設定し、与えられた配列を`json_encode` PHP関数を使用してJSONに変換します。

    return response()->json([
        'name' => 'Abigail',
        'state' => 'CA',
    ]);

JSONPレスポンスを作成する場合は、`json`メソッドと`withCallback`メソッドを組み合わせて使用できます。

    return response()
                ->json(['name' => 'Abigail', 'state' => 'CA'])
                ->withCallback($request->input('callback'));

<a name="file-downloads"></a>
### ファイルダウンロード

`download`メソッドは、指定されたパスのファイルをユーザーのブラウザにダウンロードさせるレスポンスを生成するために使用できます。`download`メソッドは、メソッドの第二引数としてファイル名を受け取り、ユーザーがファイルをダウンロードする際に表示されるファイル名を決定します。最後に、メソッドの第三引数としてHTTPヘッダーの配列を渡すことができます。

    return response()->download($pathToFile);

    return response()->download($pathToFile, $name, $headers);

> WARNING:  
> Symfony HttpFoundation（ファイルダウンロードを管理する）は、ダウンロードされるファイルがASCIIファイル名を持つ必要があります。

<a name="file-responses"></a>
### ファイルレスポンス

`file`メソッドは、画像やPDFなどのファイルをダウンロードを開始する代わりにユーザーのブラウザに直接表示するために使用できます。このメソッドは、ファイルへの絶対パスを第一引数として受け取り、ヘッダーの配列を第二引数として受け取ります。

    return response()->file($pathToFile);

    return response()->file($pathToFile, $headers);

<a name="streamed-responses"></a>
### ストリーミングレスポンス

データを生成されると同時にクライアントにストリーミングすることで、特に非常に大きなレスポンスの場合にメモリ使用量を大幅に削減し、パフォーマンスを向上させることができます。ストリーミングレスポンスにより、サーバーがデータの送信を完了する前にクライアントがデータの処理を開始できます。

    function streamedContent(): Generator {
        yield 'Hello, ';
        yield 'World!';
    }

    Route::get('/stream', function () {
        return response()->stream(function (): void {
            foreach (streamedContent() as $chunk) {
                echo $chunk;
                ob_flush();
                flush();
                sleep(2); // チャンク間の遅延をシミュレート...
            }
        }, 200, ['X-Accel-Buffering' => 'no']);
    });

> NOTE:
> 内部的には、LaravelはPHPの出力バッファリング機能を利用しています。上記の例でわかるように、バッファリングされたコンテンツをクライアントにプッシュするために`ob_flush`と`flush`関数を使用する必要があります。

<a name="streamed-json-responses"></a>
#### ストリーミングJSONレスポンス

JSONデータをインクリメンタルにストリーミングする必要がある場合は、`streamJson`メソッドを利用できます。このメソッドは、特にJavaScriptで簡単に解析できる形式でブラウザに徐々に送信する必要がある大規模なデータセットに特に便利です。

    use App\Models\User;

    Route::get('/users.json', function () {
        return response()->streamJson([
            'users' => User::cursor(),
        ]);
    });

<a name="streamed-downloads"></a>
#### ストリーミングダウンロード

特定の操作の文字列レスポンスをディスクに書き込むことなくダウンロード可能なレスポンスに変換したい場合があります。このような場合には、`streamDownload`メソッドを使用できます。このメソッドは、コールバック、ファイル名、およびオプションのヘッダー配列を引数として受け取ります。

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
                    ->contents()
                    ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="response-macros"></a>
## レスポンスマクロ

さまざまなルートやコントローラーで再利用できるカスタムレスポンスを定義したい場合は、`Response`ファサードの`macro`メソッドを使用できます。通常、このメソッドは、`App\Providers\AppServiceProvider`サービスプロバイダーなど、アプリケーションの[サービスプロバイダー](providers.md)の`boot`メソッドから呼び出す必要があります。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Response;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションサービスの初期化処理
         */
        public function boot(): void
        {
            Response::macro('caps', function (string $value) {
                return Response::make(strtoupper($value));
            });
        }
    }

`macro`関数は、名前を第一引数として受け取り、クロージャを第二引数として受け取ります。マクロのクロージャは、`ResponseFactory`実装または`response`ヘルパーからマクロ名を呼び出すと実行されます。

    return response()->caps('foo');
