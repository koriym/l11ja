# サービスコンテナ

- [はじめに](#introduction)
    - [ゼロコンフィグレーションの解決](#zero-configuration-resolution)
    - [コンテナをいつ使うか](#when-to-use-the-container)
- [バインディング](#binding)
    - [バインディングの基本](#binding-basics)
    - [インターフェースの実装へのバインディング](#binding-interfaces-to-implementations)
    - [コンテキストバインディング](#contextual-binding)
    - [コンテキスト属性](#contextual-attributes)
    - [プリミティブのバインディング](#binding-primitives)
    - [型付き可変引数のバインディング](#binding-typed-variadics)
    - [タグ付け](#tagging)
    - [バインディングの拡張](#extending-bindings)
- [解決](#resolving)
    - [Makeメソッド](#the-make-method)
    - [自動注入](#automatic-injection)
- [メソッドの呼び出しと注入](#method-invocation-and-injection)
- [コンテナイベント](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## はじめに

Laravelのサービスコンテナは、クラスの依存関係を管理し、依存性の注入（ディペンデンシーインジェクション）を実行するための強力なツールです。依存性注入とは、あるクラスの依存関係がコンストラクタや、場合によっては「セッター」メソッドを介して「注入」されることを意味する、ちょっとした格好いい言葉です。

簡単な例を見てみましょう：

    <?php

    namespace App\Http\Controllers;

    use App\Services\AppleMusic;
    use Illuminate\View\View;

    class PodcastController extends Controller
    {
        /**
         * 新しいコントローラインスタンスの作成
         */
        public function __construct(
            protected AppleMusic $apple,
        ) {}

        /**
         * 指定されたポッドキャストの情報を表示
         */
        public function show(string $id): View
        {
            return view('podcasts.show', [
                'podcast' => $this->apple->findPodcast($id)
            ]);
        }
    }

この例では、`PodcastController`はApple Musicのようなデータソースからポッドキャストを取得する必要があります。そこで、ポッドキャストを取得できるサービスを**注入**します。サービスが注入されているため、アプリケーションをテストする際に`AppleMusic`サービスのダミー実装を簡単に作成できます。

Laravelのサービスコンテナを深く理解することは、強力で大規模なアプリケーションを構築するために不可欠であり、Laravelコア自体に貢献するためにも重要です。

<a name="zero-configuration-resolution"></a>
### ゼロコンフィグレーション（設定不要）の解決

クラスが依存関係を持たないか、他の具象クラス（インターフェースではない）にのみ依存している場合、コンテナはそのクラスを解決する方法を指示する必要はありません。たとえば、以下のコードを`routes/web.php`ファイルに配置できます：

    <?php

    class Service
    {
        // ...
    }

    Route::get('/', function (Service $service) {
        die($service::class);
    });

この例では、アプリケーションの`/`ルートにアクセスすると、`Service`クラスが自動的に解決され、ルートのハンドラに注入されます。これは画期的です。つまり、依存性注入を利用してアプリケーションを開発できるだけでなく、肥大化した設定ファイルを心配する必要がありません。

幸いなことに、Laravelアプリケーションを構築する際に書く多くのクラスは、コンテナを介して自動的に依存関係を受け取ります。これには、[コントローラ](controllers.md)、[イベントリスナ](events.md)、[ミドルウェア](middleware.md)などが含まれます。さらに、[キュージョブ](queues.md)の`handle`メソッドで依存関係を型付きで指定することもできます。自動的でゼロコンフィグレーションの依存性注入の力を味わうと、それなしで開発することは不可能に感じるでしょう。

<a name="when-to-use-the-container"></a>
### コンテナの使用タイミング

ゼロコンフィグレーションの解決のおかげで、ルート、コントローラ、イベントリスナなどで依存関係を型付きで指定することが多く、手動でコンテナを操作することはほとんどありません。たとえば、ルート定義で`Illuminate\Http\Request`オブジェクトを型付きで指定して、現在のリクエストに簡単にアクセスできるようにすることができます。このコードを書く際にコンテナを操作する必要はありませんが、コンテナはこれらの依存関係の注入をバックグラウンドで管理しています：

    use Illuminate\Http\Request;

    Route::get('/', function (Request $request) {
        // ...
    });

多くの場合、自動的な依存性注入と[ファサード](facades.md)のおかげで、Laravelアプリケーションを構築する際に**コンテナから手動で何かをバインドしたり解決したりすることはありません**。では、いつ手動でコンテナを操作するのでしょうか？2つの状況を見てみましょう。

まず、インターフェースを実装するクラスを書き、そのインターフェースをルートやクラスのコンストラクタで型付きで指定したい場合、[コンテナにそのインターフェースを解決する方法を指示する](#binding-interfaces-to-implementations)必要があります。次に、他のLaravel開発者と共有する予定の[Laravelパッケージ](packages.md)を書いている場合、パッケージのサービスをコンテナにバインドする必要があるかもしれません。

<a name="binding"></a>
## バインディング

<a name="binding-basics"></a>
### バインディングの基本

<a name="simple-bindings"></a>
#### シンプルなバインディング

ほとんどのサービスコンテナのバインディングは、[サービスプロバイダ](providers.md)内で登録されます。そのため、これらの例のほとんどは、コンテナをそのコンテキストで使用することを示します。

サービスプロバイダ内では、常に`$this->app`プロパティを介してコンテナにアクセスできます。`bind`メソッドを使用してバインディングを登録できます。登録したいクラスまたはインターフェース名と、クラスのインスタンスを返すクロージャを渡します：

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->bind(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

コンテナ自体がリゾルバの引数として渡されることに注意してください。その後、コンテナを使用して、構築中のオブジェクトのサブ依存関係を解決できます。

前述のように、通常はサービスプロバイダ内でコンテナを操作しますが、サービスプロバイダ外でコンテナを操作したい場合は、`App`[ファサード](facades.md)を介して行うことができます：

    use App\Services\Transistor;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\App;

    App::bind(Transistor::class, function (Application $app) {
        // ...
    });

`bindIf`メソッドを使用して、指定された型のバインディングがまだ登録されていない場合にのみコンテナのバインディングを登録できます：

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

> NOTE:  
> インターフェースに依存しないクラスをコンテナにバインドする必要はありません。コンテナはリフレクションを使用してこれらのオブジェクトを自動的に解決できるため、コンテナにこれらのオブジェクトの構築方法を指示する必要はありません。

<a name="binding-a-singleton"></a>
#### シングルトンのバインディング

`singleton`メソッドは、クラスまたはインターフェースをコンテナにバインドします。これは一度だけ解決されるべきものです。シングルトンバインディングが解決されると、同じオブジェクトインスタンスがコンテナへの後続の呼び出しで返されます：

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->singleton(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

`singletonIf`メソッドを使用して、指定された型のバインディングがまだ登録されていない場合にのみシングルトンコンテナのバインディングを登録できます：

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="binding-scoped"></a>
#### スコープ付きシングルトンのバインディング

`scoped`メソッドは、クラスまたはインターフェースをコンテナにバインドします。これは、指定されたLaravelリクエスト/ジョブのライフサイクル内で一度だけ解決されるべきものです。このメソッドは`singleton`メソッドに似ていますが、`scoped`メソッドを使用して登録されたインスタンスは、Laravelアプリケーションが新しい「ライフサイクル」を開始するたびにフラッシュされます。たとえば、[Laravel Octane](octane.md)ワーカーが新しいリクエストを処理するとき、または[Laravelキューワーカー](queues.md)が新しいジョブを処理するときなどです：

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->scoped(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

<a name="binding-instances"></a>
#### インスタンスのバインディング

既存のオブジェクトインスタンスを`instance`メソッドを使用してコンテナにバインドすることもできます。指定されたインスタンスは、コンテナへの後続の呼び出しで常に返されます：

    use App\Services\Transistor;
    use App\Services\PodcastParser;

    $service = new Transistor(new PodcastParser);

    $this->app->instance(Transistor::class, $service);

<a name="binding-interfaces-to-implementations"></a>
### インターフェースの実装へのバインディング

サービスコンテナの非常に強力な機能は、インターフェースを特定の実装にバインドする能力です。たとえば、`EventPusher`インターフェースとその`RedisEventPusher`実装があるとします。このインターフェースの`RedisEventPusher`実装をコーディングしたら、次のようにサービスコンテナに登録できます：

    use App\Contracts\EventPusher;
    use App\Services\RedisEventPusher;

    $this->app->bind(EventPusher::class, RedisEventPusher::class);

このステートメントは、コンテナに対して、クラスが`EventPusher`の実装を必要とする場合に`RedisEventPusher`を注入するよう指示します。これで、コンテナによって解決されるクラスのコンストラクタで`EventPusher`インターフェースをタイプヒントできます。Laravelアプリケーション内のコントローラ、イベントリスナー、ミドルウェア、その他さまざまなタイプのクラスは、常にコンテナを使用して解決されることを覚えておいてください。

```php
use App\Contracts\EventPusher;

/**
 * 新しいクラスインスタンスを作成します。
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

<a name="contextual-binding"></a>
### コンテキストバインディング

時には、同じインターフェースを利用する2つのクラスがあり、それぞれのクラスに異なる実装を注入したい場合があります。たとえば、2つのコントローラが`Illuminate\Contracts\Filesystem\Filesystem` [契約](contracts.md)の異なる実装に依存するかもしれません。Laravelは、この動作を定義するためのシンプルで流暢なインターフェースを提供します。

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

<a name="contextual-attributes"></a>
### コンテキスト属性

コンテキストバインディングは、多くの場合、ドライバの実装や設定値の注入に使用されるため、Laravelは、サービスプロバイダでコンテキストバインディングを手動で定義することなく、これらのタイプの値を注入できるさまざまなコンテキストバインディング属性を提供します。

たとえば、`Storage`属性を使用して、特定の[ストレージディスク](filesystem.md)を注入できます。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    )
    {
        // ...
    }
}
```

`Storage`属性に加えて、Laravelは`Auth`、`Cache`、`Config`、`DB`、`Log`、および[`Tag`](#tagging)属性を提供します。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Log;
use Illuminate\Container\Attributes\Tag;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Contracts\Database\Connection;
use Psr\Log\LoggerInterface;

class PhotoController extends Controller
{
    public function __construct(
        #[Auth('web')] protected Guard $auth,
        #[Cache('redis')] protected Repository $cache,
        #[Config('app.timezone')] protected string $timezone,
        #[DB('mysql')] protected Connection $connection,
        #[Log('daily')] protected LoggerInterface $log,
        #[Tag('reports')] protected iterable $reports,
    )
    {
        // ...
    }
}
```

さらに、Laravelは、現在認証されているユーザーを特定のルートまたはクラスに注入するための`CurrentUser`属性を提供します。

```php
use App\Models\User;
use Illuminate\Container\Attributes\CurrentUser;

Route::get('/user', function (#[CurrentUser] User $user) {
    return $user;
})->middleware('auth');
```

<a name="defining-custom-attributes"></a>
#### カスタム属性の定義

`Illuminate\Contracts\Container\ContextualAttribute`契約を実装することで、独自のコンテキスト属性を作成できます。コンテナは属性の`resolve`メソッドを呼び出し、その属性を利用するクラスに注入されるべき値を解決します。以下の例では、Laravelの組み込みの`Config`属性を再実装します。

```php
<?php

namespace App\Attributes;

use Illuminate\Contracts\Container\ContextualAttribute;

#[Attribute(Attribute::TARGET_PARAMETER)]
class Config implements ContextualAttribute
{
    /**
     * 新しい属性インスタンスを作成します。
     */
    public function __construct(public string $key, public mixed $default = null)
    {
    }

    /**
     * 設定値を解決します。
     *
     * @param  self  $attribute
     * @param  \Illuminate\Contracts\Container\Container  $container
     * @return mixed
     */
    public static function resolve(self $attribute, Container $container)
    {
        return $container->make('config')->get($attribute->key, $attribute->default);
    }
}
```

<a name="binding-primitives"></a>
### プリミティブ値のバインディング

クラスがいくつかの注入されたクラスを受け取るが、整数などの注入されたプリミティブ値も必要な場合があります。コンテキストバインディングを使用して、クラスが必要とするあらゆる値を簡単に注入できます。

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
          ->needs('$variableName')
          ->give($value);
```

クラスが[タグ付けされた](#tagging)インスタンスの配列に依存する場合、`giveTagged`メソッドを使用して、そのタグを持つすべてのコンテナバインディングを簡単に注入できます。

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

アプリケーションの設定ファイルのいずれかから値を注入する必要がある場合は、`giveConfig`メソッドを使用できます。

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

<a name="binding-typed-variadics"></a>
### 型付き可変引数のバインディング

時には、クラスが可変引数のコンストラクタ引数を使用して型付きオブジェクトの配列を受け取る場合があります。

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    /**
     * フィルタインスタンス。
     *
     * @var array
     */
    protected $filters;

    /**
     * 新しいクラスインスタンスを作成します。
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}
```

コンテキストバインディングを使用して、この依存関係を解決するには、解決された`Filter`インスタンスの配列を返すクロージャを`give`メソッドに提供します。

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function (Application $app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
          });
```

便宜上、`Firewall`が`Filter`インスタンスを必要とするときにコンテナによって解決されるクラス名の配列を提供することもできます。

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              ProfanityFilter::class,
              TooLongFilter::class,
          ]);
```

<a name="variadic-tag-dependencies"></a>
#### 可変長引数のタグ依存関係

クラスが特定のクラス（`Report ...$reports`）として型付けされた可変依存関係を持つ場合、`needs`および`giveTagged`メソッドを使用して、その依存関係に対して[タグ付けされた](#tagging)すべてのコンテナバインディングを簡単に注入できます。

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

<a name="tagging"></a>
### タグ付け

時には、特定の「カテゴリ」のすべてのバインディングを解決する必要があるかもしれません。たとえば、多くの異なる`Report`インターフェース実装の配列を受け取るレポートアナライザを構築しているかもしれません。`Report`実装を登録した後、`tag`メソッドを使用してタグを割り当てることができます。

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

サービスにタグが付けられたら、コンテナの`tagged`メソッドを介して簡単にすべて解決できます。

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### バインディングの拡張

`extend`メソッドを使用すると、解決されたサービスの変更が可能です。たとえば、サービスが解決されたときに、サービスを装飾または設定するための追加のコードを実行できます。`extend`メソッドは、拡張するサービスクラスと、変更されたサービスを返すべきクロージャの2つの引数を受け取ります。クロージャは、解決されるサービスとコンテナインスタンスを受け取ります。

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## 解決

<a name="the-make-method"></a>
### `make`メソッド

コンテナからクラスインスタンスを解決するには、`make`メソッドを使用できます。`make`メソッドは、解決したいクラスまたはインターフェースの名前を受け取ります。

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

クラスの依存関係の一部がコンテナによって解決できない場合、`makeWith`メソッドに連想配列として渡すことで注入できます。たとえば、`Transistor`サービスに必要な`$id`コンストラクタ引数を手動で渡すことができます。

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

```php
$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

`bound`メソッドは、クラスやインターフェースがコンテナに明示的にバインドされているかどうかを判断するために使用できます:

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

コードの場所がサービスプロバイダの外であり、`$app`変数にアクセスできない場合、`App` [ファサード](facades.md)または`app` [ヘルパー](helpers.md#method-app)を使用して、コンテナからクラスインスタンスを解決できます:

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

Laravelコンテナインスタンス自体を、コンテナによって解決されるクラスに注入したい場合は、クラスのコンストラクタに`Illuminate\Container\Container`クラスをタイプヒントすることができます:

```php
use Illuminate\Container\Container;

/**
 * 新しいクラスインスタンスを作成します。
 */
public function __construct(
    protected Container $container,
) {}
```

<a name="automatic-injection"></a>
### 自動注入

あるいは、重要なことに、コンテナによって解決されるクラスのコンストラクタに依存関係をタイプヒントすることができます。これには、[コントローラ](controllers.md)、[イベントリスナ](events.md)、[ミドルウェア](middleware.md)などが含まれます。さらに、[キュージョブ](queues.md)の`handle`メソッドに依存関係をタイプヒントすることもできます。実際には、これがコンテナによってほとんどのオブジェクトが解決される方法です。

例えば、コントローラのコンストラクタにアプリケーションで定義されたサービスをタイプヒントすることができます。サービスは自動的に解決され、クラスに注入されます:

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * 新しいコントローラインスタンスを作成します。
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * 指定されたポッドキャストに関する情報を表示します。
     */
    public function show(string $id): Podcast
    {
        return $this->apple->findPodcast($id);
    }
}
```

<a name="method-invocation-and-injection"></a>
## メソッドの呼び出しと注入

オブジェクトインスタンスのメソッドを呼び出したいが、そのメソッドの依存関係をコンテナに自動的に注入させたい場合があります。例えば、次のクラスがあるとします:

```php
<?php

namespace App;

use App\Services\AppleMusic;

class PodcastStats
{
    /**
     * 新しいポッドキャスト統計レポートを生成します。
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}
```

`generate`メソッドは、コンテナを介して次のように呼び出すことができます:

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;

$stats = App::call([new PodcastStats, 'generate']);
```

`call`メソッドは、任意のPHP callableを受け入れます。コンテナの`call`メソッドは、クロージャを呼び出す際にもその依存関係を自動的に注入するために使用できます:

```php
use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

<a name="container-events"></a>
## コンテナイベント

サービスコンテナは、オブジェクトを解決するたびにイベントを発火します。このイベントは`resolving`メソッドを使用してリッスンできます:

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // "Transistor"タイプのオブジェクトがコンテナによって解決されるときに呼び出されます...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // 任意のタイプのオブジェクトがコンテナによって解決されるときに呼び出されます...
});
```

ご覧のとおり、解決されるオブジェクトはコールバックに渡され、オブジェクトが消費者に渡される前に追加のプロパティを設定できます。

<a name="psr-11"></a>
## PSR-11

Laravelのサービスコンテナは[PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md)インターフェースを実装しています。したがって、PSR-11コンテナインターフェースをタイプヒントして、Laravelコンテナのインスタンスを取得できます:

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

指定された識別子を解決できない場合、例外がスローされます。識別子がバインドされていない場合、例外は`Psr\Container\NotFoundExceptionInterface`のインスタンスになります。識別子はバインドされているが解決できない場合、`Psr\Container\ContainerExceptionInterface`のインスタンスがスローされます。

