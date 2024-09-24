# HTTPセッション

- [はじめに](#introduction)
    - [設定](#configuration)
    - [ドライバの前提条件](#driver-prerequisites)
- [セッションとのやり取り](#interacting-with-the-session)
    - [データの取得](#retrieving-data)
    - [データの保存](#storing-data)
    - [フラッシュデータ](#flash-data)
    - [データの削除](#deleting-data)
    - [セッションIDの再生成](#regenerating-the-session-id)
- [セッションブロッキング](#session-blocking)
- [カスタムセッションドライバの追加](#adding-custom-session-drivers)
    - [ドライバの実装](#implementing-the-driver)
    - [ドライバの登録](#registering-the-driver)

<a name="introduction"></a>
## はじめに

HTTP駆動のアプリケーションはステートレスであるため、セッションはユーザーに関する情報を複数のリクエストにわたって保存する方法を提供します。そのユーザー情報は通常、後続のリクエストからアクセスできる永続的なストア/バックエンドに配置されます。

Laravelには、表現力豊かで統一されたAPIを介してアクセスできるさまざまなセッションバックエンドが付属しています。[Memcached](https://memcached.org)、[Redis](https://redis.io)、データベースなどの一般的なバックエンドのサポートが含まれています。

<a name="configuration"></a>
### 設定

アプリケーションのセッション設定ファイルは`config/session.php`に保存されています。このファイルで利用可能なオプションを確認してください。デフォルトでは、Laravelは`database`セッションドライバを使用するように設定されています。

セッションの`driver`設定オプションは、各リクエストのセッションデータがどこに保存されるかを定義します。Laravelにはさまざまなドライバが含まれています：

<div class="content-list" markdown="1">

- `file` - セッションは`storage/framework/sessions`に保存されます。
- `cookie` - セッションは安全で暗号化されたクッキーに保存されます。
- `database` - セッションはリレーショナルデータベースに保存されます。
- `memcached` / `redis` - セッションはこれらの高速なキャッシュベースのストアのいずれかに保存されます。
- `dynamodb` - セッションはAWS DynamoDBに保存されます。
- `array` - セッションはPHP配列に保存され、永続化されません。

</div>

> NOTE:  
> 配列ドライバは主に[テスト](testing.md)中に使用され、セッションに保存されたデータが永続化されないようにします。

<a name="driver-prerequisites"></a>
### ドライバの前提条件

<a name="database"></a>
#### データベース

`database`セッションドライバを使用する場合、セッションデータを格納するデータベーステーブルが必要です。通常、これはLaravelのデフォルトの`0001_01_01_000000_create_users_table.php`[データベースマイグレーション](migrations.md)に含まれています。ただし、何らかの理由で`sessions`テーブルがない場合は、`make:session-table` Artisanコマンドを使用してこのマイグレーションを生成できます：

```shell
php artisan make:session-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

LaravelでRedisセッションを使用する前に、PECLを介してPhpRedis PHP拡張機能をインストールするか、Composerを介して`predis/predis`パッケージ（~1.0）をインストールする必要があります。Redisの設定についての詳細は、Laravelの[Redisドキュメント](redis.md#configuration)を参照してください。

> NOTE:  
> `SESSION_CONNECTION`環境変数、または`session.php`設定ファイルの`connection`オプションを使用して、セッションストレージに使用するRedis接続を指定できます。

<a name="interacting-with-the-session"></a>
## セッションとのやり取り

<a name="retrieving-data"></a>
### データの取得

Laravelでセッションデータを操作するには、グローバルな`session`ヘルパと`Request`インスタンスの2つの主要な方法があります。まず、ルートクロージャまたはコントローラメソッドでタイプヒントを付けてセッションにアクセスする方法を見てみましょう。コントローラメソッドの依存関係は、Laravelの[サービスコンテナ](container.md)を介して自動的に注入されることに注意してください：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示します。
         */
        public function show(Request $request, string $id): View
        {
            $value = $request->session()->get('key');

            // ...

            $user = $this->users->find($id);

            return view('user.profile', ['user' => $user]);
        }
    }

セッションからアイテムを取得する際に、`get`メソッドに2番目の引数としてデフォルト値を渡すこともできます。このデフォルト値は、指定されたキーがセッションに存在しない場合に返されます。`get`メソッドにクロージャをデフォルト値として渡し、要求されたキーが存在しない場合、クロージャが実行され、その結果が返されます：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

<a name="the-global-session-helper"></a>
#### グローバルセッションヘルパ

セッション内のデータを取得および保存するために、グローバルな`session` PHP関数を使用することもできます。`session`ヘルパが単一の文字列引数で呼び出されると、そのセッションキーの値が返されます。ヘルパがキー/値ペアの配列で呼び出されると、それらの値がセッションに保存されます：

    Route::get('/home', function () {
        // セッションからデータを取得する...
        $value = session('key');

        // デフォルト値を指定する...
        $value = session('key', 'default');

        // セッションにデータを保存する...
        session(['key' => 'value']);
    });

> NOTE:  
> HTTPリクエストインスタンスを介してセッションを使用するか、グローバルな`session`ヘルパを使用するかに実際的な違いはほとんどありません。どちらの方法も、すべてのテストケースで利用可能な`assertSessionHas`メソッドを介して[テスト可能](testing.md)です。

<a name="retrieving-all-session-data"></a>
#### すべてのセッションデータの取得

セッション内のすべてのデータを取得したい場合は、`all`メソッドを使用できます：

    $data = $request->session()->all();

<a name="retrieving-a-portion-of-the-session-data"></a>
#### セッションデータの一部の取得

`only`および`except`メソッドを使用して、セッションデータのサブセットを取得できます：

    $data = $request->session()->only(['username', 'email']);

    $data = $request->session()->except(['username', 'email']);

<a name="determining-if-an-item-exists-in-the-session"></a>
#### セッション内にアイテムが存在するかどうかの判定

アイテムがセッション内に存在するかどうかを判定するには、`has`メソッドを使用できます。`has`メソッドは、アイテムが存在し、`null`でない場合に`true`を返します：

    if ($request->session()->has('users')) {
        // ...
    }

アイテムがセッション内に存在し、その値が`null`であっても存在するかどうかを判定するには、`exists`メソッドを使用できます：

    if ($request->session()->exists('users')) {
        // ...
    }

アイテムがセッション内に存在しないかどうかを判定するには、`missing`メソッドを使用できます。`missing`メソッドは、アイテムが存在しない場合に`true`を返します：

    if ($request->session()->missing('users')) {
        // ...
    }

<a name="storing-data"></a>
### データの保存

セッションにデータを保存するには、通常、リクエストインスタンスの`put`メソッドまたはグローバルな`session`ヘルパを使用します：

    // リクエストインスタンスを介して...
    $request->session()->put('key', 'value');

    // グローバルな "session" ヘルパを介して...
    session(['key' => 'value']);

<a name="pushing-to-array-session-values"></a>
#### 配列セッション値へのプッシュ

`push`メソッドを使用して、セッション値が配列である新しい値をプッシュできます。たとえば、`user.teams`キーにチーム名の配列が含まれている場合、次のように新しい値を配列にプッシュできます：

    $request->session()->push('user.teams', 'developers');

<a name="retrieving-deleting-an-item"></a>
#### アイテムの取得と削除

`pull`メソッドは、セッションからアイテムを1つのステートメントで取得して削除します：

    $value = $request->session()->pull('key', 'default');

<a name="incrementing-and-decrementing-session-values"></a>
#### セッション値のインクリメントとデクリメント

セッションデータにインクリメントまたはデクリメントしたい整数が含まれている場合、`increment`および`decrement`メソッドを使用できます：

    $request->session()->increment('count');

    $request->session()->increment('count', $incrementBy = 2);

    $request->session()->decrement('count');

    $request->session()->decrement('count', $decrementBy = 2);

<a name="flash-data"></a>
### フラッシュデータ

次のリクエストのためにセッションにアイテムを保存したい場合があります。`flash`メソッドを使用してそうすることができます。この方法でセッションに保存されたデータは、すぐに利用可能で、次のHTTPリクエスト中に利用可能です。次のHTTPリクエストの後、フラッシュデータは削除されます。フラッシュデータは主に短命のステータスメッセージに役立ちます：

    $request->session()->flash('status', 'Task was successful!');

フラッシュデータを複数のリクエストにわたって保持する必要がある場合は、`reflash`メソッドを使用して、すべてのフラッシュデータを追加のリクエストに保持できます。特定のフラッシュデータのみを保持する場合は、`keep`メソッドを使用できます：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

フラッシュデータを現在のリクエストのみに保持する場合は、`now`メソッドを使用できます：

    $request->session()->now('status', 'Task was successful!');

<a name="deleting-data"></a>
### データの削除

`forget`メソッドは、セッションからデータを削除します。セッションからすべてのデータを削除する場合は、`flush`メソッドを使用できます：

    // 単一のキーを忘れる...
    $request->session()->forget('name');

    // 複数のキーを忘れる...
    $request->session()->forget(['name', 'status']);

    $request->session()->flush();

<a name="regenerating-the-session-id"></a>
### セッションIDの再生成

セッションIDを再生成するには、通常、セッションハイジャックから保護するために使用されます。Laravelは、`regenerate`メソッドを使用してセッションIDを再生成します：

    $request->session()->regenerate();

セッションIDの再生成は、悪意のあるユーザーがアプリケーションに対して[セッション固定](https://owasp.org/www-community/attacks/Session_fixation)攻撃を仕掛けるのを防ぐためによく行われます。

Laravelは、Laravelの[アプリケーションスターターキット](starter-kits.md)または[Laravel Fortify](fortify.md)を使用している場合、認証時に自動的にセッションIDを再生成します。ただし、セッションIDを手動で再生成する必要がある場合は、`regenerate`メソッドを使用できます。

    $request->session()->regenerate();

セッションIDを再生成し、セッション内のすべてのデータを1つのステートメントで削除する場合は、`invalidate`メソッドを使用できます。

    $request->session()->invalidate();

<a name="session-blocking"></a>
## セッションブロッキング

> WARNING:  
> セッションブロッキングを利用するには、アプリケーションが[アトミックロック](cache.md#atomic-locks)をサポートするキャッシュドライバを使用している必要があります。現在、それらのキャッシュドライバには、`memcached`、`dynamodb`、`redis`、`database`、`file`、および`array`ドライバが含まれます。また、`cookie`セッションドライバを使用することはできません。

デフォルトでは、Laravelは同じセッションを使用するリクエストを同時に実行できます。たとえば、JavaScriptのHTTPライブラリを使用してアプリケーションに対して2つのHTTPリクエストを行う場合、それらは両方とも同時に実行されます。多くのアプリケーションでは、これは問題になりません。ただし、セッションデータの損失が発生する可能性があるアプリケーションの小さなサブセットでは、2つの異なるアプリケーションエンドポイントに対して同時にリクエストを行い、どちらもセッションにデータを書き込む場合があります。

これを緩和するために、Laravelは、特定のセッションに対する同時リクエストを制限する機能を提供します。開始するには、ルート定義に`block`メソッドを単純にチェーンするだけです。この例では、`/profile`エンドポイントへの着信リクエストはセッションロックを取得します。このロックが保持されている間、同じセッションIDを共有する`/profile`または`/order`エンドポイントへの着信リクエストは、最初のリクエストが実行を完了するまで待機してから実行を続行します。

    Route::post('/profile', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

`block`メソッドは2つのオプションの引数を受け入れます。`block`メソッドが受け入れる最初の引数は、セッションロックを解放する前に保持する最大秒数です。もちろん、リクエストがこの時間より前に実行を完了した場合、ロックはより早く解放されます。

`block`メソッドが受け入れる2番目の引数は、リクエストがセッションロックを取得しようとする間に待機する秒数です。指定された秒数以内にリクエストがセッションロックを取得できない場合、`Illuminate\Contracts\Cache\LockTimeoutException`がスローされます。

これらの引数のいずれも渡されない場合、ロックは最大10秒間取得され、リクエストはロックを取得しようとする間に最大10秒間待機します。

    Route::post('/profile', function () {
        // ...
    })->block()

<a name="adding-custom-session-drivers"></a>
## カスタムセッションドライバの追加

<a name="implementing-the-driver"></a>
### ドライバの実装

既存のセッションドライバのいずれもアプリケーションのニーズに合わない場合、Laravelでは独自のセッションハンドラを書くことが可能です。カスタムセッションドライバは、PHPの組み込みの`SessionHandlerInterface`を実装する必要があります。このインターフェースには、いくつかのシンプルなメソッドが含まれています。スタブ化されたMongoDBの実装は次のようになります。

    <?php

    namespace App\Extensions;

    class MongoSessionHandler implements \SessionHandlerInterface
    {
        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}
    }

> NOTE:  
> Laravelには拡張機能を格納するためのディレクトリは付属していません。お好きな場所に配置できます。この例では、`MongoSessionHandler`を格納するために`Extensions`ディレクトリを作成しました。

これらのメソッドの目的はすぐには理解しにくいため、各メソッドが何を行うかを簡単に説明しましょう。

<div class="content-list" markdown="1">

- `open`メソッドは通常、ファイルベースのセッションストアシステムで使用されます。Laravelには`file`セッションドライバが付属しているため、このメソッドにはほとんど何も入れる必要はありません。単純にこのメソッドを空のままにしておくことができます。
- `close`メソッドは、`open`メソッドと同様に通常は無視できます。多くのドライバでは必要ありません。
- `read`メソッドは、指定された`$sessionId`に関連付けられたセッションデータの文字列バージョンを返す必要があります。ドライバでセッションデータを取得または保存する際に、シリアライズやその他のエンコーディングを行う必要はありません。Laravelがシリアライズを行います。
- `write`メソッドは、指定された`$sessionId`に関連付けられた`$data`文字列をMongoDBなどの永続ストレージシステムに書き込む必要があります。ここでも、シリアライズを行う必要はありません。Laravelがすでにそれを処理しています。
- `destroy`メソッドは、永続ストレージから`$sessionId`に関連付けられたデータを削除する必要があります。
- `gc`メソッドは、指定された`$lifetime`（UNIXタイムスタンプ）より古いすべてのセッションデータを破棄する必要があります。MemcachedやRedisのような自己期限切れシステムの場合、このメソッドは空のままにしておくことができます。

</div>

<a name="registering-the-driver"></a>
### ドライバの登録

ドライバが実装されたら、Laravelに登録する準備が整いました。Laravelのセッションバックエンドに追加のドライバを追加するには、`Session`[ファサード](facades.md)が提供する`extend`メソッドを使用できます。`extend`メソッドは、[サービスプロバイダ](providers.md)の`boot`メソッドから呼び出す必要があります。これは、既存の`App\Providers\AppServiceProvider`から行うか、まったく新しいプロバイダを作成することができます。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoSessionHandler;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Session;
    use Illuminate\Support\ServiceProvider;

    class SessionServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスの起動。
         */
        public function boot(): void
        {
            Session::extend('mongo', function (Application $app) {
                // SessionHandlerInterfaceの実装を返す...
                return new MongoSessionHandler;
            });
        }
    }

セッションドライバが登録されたら、アプリケーションのセッションドライバとして`mongo`ドライバを`SESSION_DRIVER`環境変数またはアプリケーションの`config/session.php`設定ファイル内で指定できます。

