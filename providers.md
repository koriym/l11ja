# サービスプロバイダ

- [イントロダクション](#introduction)
- [サービスプロバイダの作成](#writing-service-providers)
    - [Registerメソッド](#the-register-method)
    - [Bootメソッド](#the-boot-method)
- [プロバイダの登録](#registering-providers)
- [遅延プロバイダ](#deferred-providers)

<a name="introduction"></a>
## イントロダクション

サービスプロバイダは、すべてのLaravelアプリケーションの初期起動の中心です。あなた自身のアプリケーション、そしてLaravelのコアサービスもすべて、サービスプロバイダを介して初期起動されます。

しかし、「初期起動」とは何を意味するのでしょうか？一般的に、**登録**を意味します。つまり、サービスコンテナのバインディング、イベントリスナー、ミドルウェア、さらにはルートの登録です。サービスプロバイダは、アプリケーションの設定の中心地です。

Laravelは、メーラー、キュー、キャッシュなどのコアサービスを初期起動するために、内部的に数十のサービスプロバイダを使用しています。これらのプロバイダの多くは「遅延」プロバイダであり、つまり、すべてのリクエストで読み込まれるのではなく、提供するサービスが実際に必要になったときにのみ読み込まれます。

すべてのユーザー定義のサービスプロバイダは、`bootstrap/providers.php`ファイルに登録されています。次のドキュメントでは、独自のサービスプロバイダを作成し、Laravelアプリケーションに登録する方法を学びます。

> NOTE:  
> Laravelがリクエストを処理し、内部でどのように動作するかについて詳しく知りたい場合は、Laravelの[リクエストライフサイクル](lifecycle.md)に関するドキュメントをチェックしてください。

<a name="writing-service-providers"></a>
## サービスプロバイダの作成

すべてのサービスプロバイダは、`Illuminate\Support\ServiceProvider`クラスを拡張します。ほとんどのサービスプロバイダには、`register`メソッドと`boot`メソッドが含まれています。`register`メソッド内では、**[サービスコンテナ](container.md)にバインディングを登録するだけ**にしてください。`register`メソッド内でイベントリスナー、ルート、またはその他の機能を登録しようとしないでください。

Artisan CLIは、`make:provider`コマンドを介して新しいプロバイダを生成できます。Laravelは自動的に、アプリケーションの`bootstrap/providers.php`ファイルに新しいプロバイダを登録します。

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Registerメソッド

前述のように、`register`メソッド内では、**[サービスコンテナ](container.md)にバインディングを登録するだけ**にしてください。`register`メソッド内でイベントリスナー、ルート、またはその他の機能を登録しようとしないでください。そうしないと、まだ読み込まれていないサービスプロバイダによって提供されるサービスを誤って使用する可能性があります。

基本的なサービスプロバイダを見てみましょう。サービスプロバイダのメソッド内では、常にサービスコンテナにアクセスできる`$app`プロパティを使用できます。

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスを登録します。
         */
        public function register(): void
        {
            $this->app->singleton(Connection::class, function (Application $app) {
                return new Connection(config('riak'));
            });
        }
    }

このサービスプロバイダは、`register`メソッドのみを定義し、そのメソッドを使用して、`App\Services\Riak\Connection`の実装をサービスコンテナに定義します。Laravelのサービスコンテナにまだ慣れていない場合は、[そのドキュメント](container.md)をチェックしてください。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings`と`singletons`プロパティ

サービスプロバイダが多くの単純なバインディングを登録する場合、`bindings`と`singletons`プロパティを使用することを選択できます。フレームワークがサービスプロバイダを読み込むとき、これらのプロパティを自動的にチェックし、それらのバインディングを登録します。

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 登録すべきすべてのコンテナバインディング。
         *
         * @var array
         */
        public $bindings = [
            ServerProvider::class => DigitalOceanServerProvider::class,
        ];

        /**
         * 登録すべきすべてのコンテナシングルトン。
         *
         * @var array
         */
        public $singletons = [
            DowntimeNotifier::class => PingdomDowntimeNotifier::class,
            ServerProvider::class => ServerToolsProvider::class,
        ];
    }

<a name="the-boot-method"></a>
### Bootメソッド

では、サービスプロバイダ内で[ビューコンポーザ](views.md#view-composers)を登録する必要がある場合はどうでしょうか？それは`boot`メソッド内で行うべきです。**このメソッドは、他のすべてのサービスプロバイダが登録された後に呼び出されます**。つまり、フレームワークによって登録された他のすべてのサービスにアクセスできます。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;
    use Illuminate\Support\ServiceProvider;

    class ComposerServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスを起動します。
         */
        public function boot(): void
        {
            View::composer('view', function () {
                // ...
            });
        }
    }

<a name="boot-method-dependency-injection"></a>
#### Bootメソッドの依存性注入

サービスプロバイダの`boot`メソッドに依存関係をタイプヒントすることができます。[サービスコンテナ](container.md)は、必要な依存関係を自動的に注入します。

    use Illuminate\Contracts\Routing\ResponseFactory;

    /**
     * 任意のアプリケーションサービスを起動します。
     */
    public function boot(ResponseFactory $response): void
    {
        $response->macro('serialized', function (mixed $value) {
            // ...
        });
    }

<a name="registering-providers"></a>
## プロバイダの登録

すべてのサービスプロバイダは、`bootstrap/providers.php`設定ファイルに登録されています。このファイルは、アプリケーションのサービスプロバイダのクラス名を含む配列を返します。

    <?php

    return [
        App\Providers\AppServiceProvider::class,
    ];

`make:provider` Artisanコマンドを呼び出すと、Laravelは自動的に生成されたプロバイダを`bootstrap/providers.php`ファイルに追加します。ただし、プロバイダクラスを手動で作成した場合は、配列にプロバイダクラスを手動で追加する必要があります。

    <?php

    return [
        App\Providers\AppServiceProvider::class,
        App\Providers\ComposerServiceProvider::class, // [tl! add]
    ];

<a name="deferred-providers"></a>
## 遅延プロバイダ

もし、プロバイダが**[サービスコンテナ](container.md)にバインディングを登録するだけ**である場合、その登録を遅延させることを選択できます。遅延読み込みされたプロバイダは、アプリケーションのパフォーマンスを向上させます。なぜなら、それはすべてのリクエストでファイルシステムから読み込まれるのではなく、登録されたバインディングのいずれかが実際に必要になったときにのみ読み込まれるからです。

Laravelは、遅延サービスプロバイダによって提供されるすべてのサービスのリストと、そのサービスプロバイダクラスの名前をコンパイルして保存します。その後、これらのサービスのいずれかを解決しようとすると、Laravelはサービスプロバイダを読み込みます。

プロバイダの読み込みを遅延させるには、`\Illuminate\Contracts\Support\DeferrableProvider`インターフェースを実装し、`provides`メソッドを定義します。`provides`メソッドは、プロバイダによって登録されたサービスコンテナのバインディングを返す必要があります。

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Contracts\Support\DeferrableProvider;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
    {
        /**
         * 任意のアプリケーションサービスを登録します。
         */
        public function register(): void
        {
            $this->app->singleton(Connection::class, function (Application $app) {
                return new Connection($app['config']['riak']);
            });
        }

        /**
         * プロバイダによって提供されるサービスを取得します。
         *
         * @return array<int, string>
         */
        public function provides(): array
        {
            return [Connection::class];
        }
    }

