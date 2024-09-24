# Laravel Telescope

- [はじめに](#introduction)
- [インストール](#installation)
    - [ローカル限定インストール](#local-only-installation)
    - [設定](#configuration)
    - [データの整理](#data-pruning)
    - [ダッシュボードの認可](#dashboard-authorization)
- [Telescopeのアップグレード](#upgrading-telescope)
- [フィルタリング](#filtering)
    - [エントリ](#filtering-entries)
    - [バッチ](#filtering-batches)
- [タグ付け](#tagging)
- [利用可能なウォッチャー](#available-watchers)
    - [バッチウォッチャー](#batch-watcher)
    - [キャッシュウォッチャー](#cache-watcher)
    - [コマンドウォッチャー](#command-watcher)
    - [ダンプウォッチャー](#dump-watcher)
    - [イベントウォッチャー](#event-watcher)
    - [例外ウォッチャー](#exception-watcher)
    - [ゲートウォッチャー](#gate-watcher)
    - [HTTPクライアントウォッチャー](#http-client-watcher)
    - [ジョブウォッチャー](#job-watcher)
    - [ログウォッチャー](#log-watcher)
    - [メールウォッチャー](#mail-watcher)
    - [モデルウォッチャー](#model-watcher)
    - [通知ウォッチャー](#notification-watcher)
    - [クエリウォッチャー](#query-watcher)
    - [Redisウォッチャー](#redis-watcher)
    - [リクエストウォッチャー](#request-watcher)
    - [スケジュールウォッチャー](#schedule-watcher)
    - [ビューウォッチャー](#view-watcher)
- [ユーザーアバターの表示](#displaying-user-avatars)

<a name="introduction"></a>
## はじめに

[Laravel Telescope](https://github.com/laravel/telescope) は、ローカルのLaravel開発環境に最適な相棒です。Telescopeは、アプリケーションに入ってくるリクエスト、例外、ログエントリ、データベースクエリ、キュージョブ、メール、通知、キャッシュ操作、スケジュールされたタスク、変数ダンプなどについての洞察を提供します。

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## インストール

Composerパッケージマネージャを使用して、TelescopeをLaravelプロジェクトにインストールできます：

```shell
composer require laravel/telescope
```

Telescopeをインストールした後、`telescope:install` Artisanコマンドを使用してそのアセットとマイグレーションを公開します。Telescopeをインストールした後、Telescopeのデータを保存するために必要なテーブルを作成するために`migrate`コマンドも実行する必要があります：

```shell
php artisan telescope:install

php artisan migrate
```

最後に、`/telescope`ルートからTelescopeダッシュボードにアクセスできます。

<a name="local-only-installation"></a>
### ローカル限定インストール

ローカル開発のみでTelescopeを使用する予定の場合は、`--dev`フラグを使用してTelescopeをインストールできます：

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

`telescope:install`を実行した後、アプリケーションの`bootstrap/providers.php`設定ファイルから`TelescopeServiceProvider`サービスプロバイダの登録を削除する必要があります。代わりに、`App\Providers\AppServiceProvider`クラスの`register`メソッドでTelescopeのサービスプロバイダを手動で登録します。現在の環境が`local`であることを確認してからプロバイダを登録します：

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        if ($this->app->environment('local')) {
            $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

最後に、Telescopeパッケージが[自動検出](packages.md#package-discovery)されないように、`composer.json`ファイルに以下を追加する必要があります：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

<a name="configuration"></a>
### 設定

Telescopeのアセットを公開した後、その主要な設定ファイルは`config/telescope.php`に配置されます。この設定ファイルでは、[ウォッチャーオプション](#available-watchers)を設定できます。各設定オプションにはその目的の説明が含まれているため、このファイルを徹底的に確認してください。

必要に応じて、`enabled`設定オプションを使用してTelescopeのデータ収集を完全に無効にすることができます：

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### データの整理

整理を行わない場合、`telescope_entries`テーブルはレコードを非常に迅速に蓄積する可能性があります。これを軽減するために、`telescope:prune` Artisanコマンドを毎日実行するように[スケジュール](scheduling.md)する必要があります：

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('telescope:prune')->daily();

デフォルトでは、24時間以上前のすべてのエントリが整理されます。コマンドを呼び出す際に`hours`オプションを使用して、Telescopeデータを保持する期間を決定できます。たとえば、次のコマンドは48時間以上前に作成されたすべてのレコードを削除します：

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('telescope:prune --hours=48')->daily();

<a name="dashboard-authorization"></a>
### ダッシュボードの認可

Telescopeダッシュボードは`/telescope`ルートからアクセスできます。デフォルトでは、このダッシュボードには`local`環境でのみアクセスできます。`app/Providers/TelescopeServiceProvider.php`ファイル内には、[認可ゲート](authorization.md#gates)の定義があります。この認可ゲートは、**非ローカル**環境でのTelescopeへのアクセスを制御します。必要に応じて、このゲートを自由に変更してTelescopeのインストールへのアクセスを制限できます：

    use App\Models\User;

    /**
     * Telescopeゲートを登録します。
     *
     * このゲートは、非ローカル環境で誰がTelescopeにアクセスできるかを決定します。
     */
    protected function gate(): void
    {
        Gate::define('viewTelescope', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

> WARNING:  
> 本番環境では、`APP_ENV`環境変数を`production`に変更するようにしてください。そうしないと、Telescopeのインストールが公開されてしまいます。

<a name="upgrading-telescope"></a>
## Telescopeのアップグレード

Telescopeの新しいメジャーバージョンにアップグレードする際には、[アップグレードガイド](https://github.com/laravel/telescope/blob/master/UPGRADE.md)を注意深く確認することが重要です。

さらに、新しいTelescopeバージョンにアップグレードする際には、Telescopeのアセットを再公開する必要があります：

```shell
php artisan telescope:publish
```

アセットを最新の状態に保ち、将来のアップデートで問題が発生しないようにするために、アプリケーションの`composer.json`ファイルの`post-update-cmd`スクリプトに`vendor:publish --tag=laravel-assets`コマンドを追加することができます：

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

<a name="filtering"></a>
## フィルタリング

<a name="filtering-entries"></a>
### エントリ

`App\Providers\TelescopeServiceProvider`クラスで定義された`filter`クロージャを介して、Telescopeによって記録されるデータをフィルタリングできます。デフォルトでは、このクロージャは`local`環境ですべてのデータを記録し、例外、失敗したジョブ、スケジュールされたタスク、および監視されたタグを持つデータを他のすべての環境で記録します：

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filter(function (IncomingEntry $entry) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
        });
    }

<a name="filtering-batches"></a>
### バッチ

`filter`クロージャは個々のエントリのデータをフィルタリングしますが、`filterBatch`メソッドを使用して、特定のリクエストまたはコンソールコマンドのすべてのデータをフィルタリングするクロージャを登録できます。クロージャが`true`を返す場合、すべてのエントリがTelescopeによって記録されます：

    use Illuminate\Support\Collection;
    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::filterBatch(function (Collection $entries) {
            if ($this->app->environment('local')) {
                return true;
            }

            return $entries->contains(function (IncomingEntry $entry) {
                return $entry->isReportableException() ||
                    $entry->isFailedJob() ||
                    $entry->isScheduledTask() ||
                    $entry->isSlowQuery() ||
                    $entry->hasMonitoredTag();
                });
        });
    }

<a name="tagging"></a>
## タグ付け

Telescopeでは、"タグ"によってエントリを検索できます。多くの場合、タグはEloquentモデルのクラス名や認証されたユーザーのIDであり、Telescopeは自動的にエントリに追加します。ときには、エントリに独自のカスタムタグを添付したい場合があります。これを実現するために、`Telescope::tag`メソッドを使用できます。`tag`メソッドは、タグの配列を返すべきクロージャを受け取ります。クロージャによって返されたタグは、Telescopeが自動的にエントリに添付するタグとマージされます。通常、`App\Providers\TelescopeServiceProvider`クラスの`register`メソッド内で`tag`メソッドを呼び出す必要があります：

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        $this->hideSensitiveRequestDetails();

        Telescope::tag(function (IncomingEntry $entry) {
            return $entry->type === 'request'
                        ? ['status:'.$entry->content['response_status']]
                        : [];
        });
     }

<a name="available-watchers"></a>
## 利用可能なウォッチャー

Telescopeの「ウォッチャー」は、リクエストやコンソールコマンドが実行されたときにアプリケーションデータを収集します。`config/telescope.php`設定ファイル内で、有効にしたいウォッチャーのリストをカスタマイズできます。

    'watchers' => [
        Watchers\CacheWatcher::class => true,
        Watchers\CommandWatcher::class => true,
        ...
    ],

一部のウォッチャーでは、追加のカスタマイズオプションを提供しています。

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="batch-watcher"></a>
### Batch Watcher

Batchウォッチャーは、キューに入れられた[バッチ](queues.md#job-batching)に関する情報を記録します。これには、ジョブと接続情報が含まれます。

<a name="cache-watcher"></a>
### Cache Watcher

Cacheウォッチャーは、キャッシュキーがヒット、ミス、更新、削除されたときにデータを記録します。

<a name="command-watcher"></a>
### Command Watcher

Commandウォッチャーは、Artisanコマンドが実行されたときに引数、オプション、終了コード、および出力を記録します。特定のコマンドをウォッチャーによる記録から除外したい場合は、`config/telescope.php`ファイル内の`ignore`オプションで指定できます。

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### Dump Watcher

Dumpウォッチャーは、変数のダンプをTelescopeに記録して表示します。Laravelを使用している場合、グローバルな`dump`関数を使用して変数をダンプできます。Dumpウォッチャータブがブラウザで開いている場合にのみ、ダンプが記録されます。それ以外の場合、ダンプはウォッチャーによって無視されます。

<a name="event-watcher"></a>
### Event Watcher

Eventウォッチャーは、アプリケーションによってディスパッチされた[イベント](events.md)のペイロード、リスナー、およびブロードキャストデータを記録します。Laravelフレームワークの内部イベントは、Eventウォッチャーによって無視されます。

<a name="exception-watcher"></a>
### Exception Watcher

Exceptionウォッチャーは、アプリケーションによってスローされた報告可能な例外のデータとスタックトレースを記録します。

<a name="gate-watcher"></a>
### Gate Watcher

Gateウォッチャーは、アプリケーションによる[ゲートとポリシー](authorization.md)のチェックのデータと結果を記録します。特定の能力をウォッチャーによる記録から除外したい場合は、`config/telescope.php`ファイル内の`ignore_abilities`オプションで指定できます。

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="http-client-watcher"></a>
### HTTP Client Watcher

HTTP Clientウォッチャーは、アプリケーションによって行われた送信[HTTPクライアントリクエスト](http-client.md)を記録します。

<a name="job-watcher"></a>
### Job Watcher

Jobウォッチャーは、アプリケーションによってディスパッチされた[ジョブ](queues.md)のデータとステータスを記録します。

<a name="log-watcher"></a>
### Log Watcher

Logウォッチャーは、アプリケーションによって書き込まれた[ログデータ](logging.md)を記録します。

デフォルトでは、Telescopeは`error`レベル以上のログのみを記録します。ただし、アプリケーションの`config/telescope.php`設定ファイル内の`level`オプションを変更して、この動作を変更できます。

    'watchers' => [
        Watchers\LogWatcher::class => [
            'enabled' => env('TELESCOPE_LOG_WATCHER', true),
            'level' => 'debug',
        ],

        // ...
    ],

<a name="mail-watcher"></a>
### Mail Watcher

Mailウォッチャーを使用すると、アプリケーションによって送信された[メール](mail.md)のブラウザ内プレビューと関連データを表示できます。また、メールを`.eml`ファイルとしてダウンロードすることもできます。

<a name="model-watcher"></a>
### Model Watcher

Modelウォッチャーは、Eloquent [モデルイベント](eloquent.md#events)がディスパッチされるたびにモデルの変更を記録します。ウォッチャーの`events`オプションを介して、どのモデルイベントを記録するかを指定できます。

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

特定のリクエスト中にハイドレートされたモデルの数を記録したい場合は、`hydrations`オプションを有効にします。

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
            'hydrations' => true,
        ],
        ...
    ],

<a name="notification-watcher"></a>
### Notification Watcher

Notificationウォッチャーは、アプリケーションによって送信されたすべての[通知](notifications.md)を記録します。通知がメールをトリガーし、Mailウォッチャーが有効になっている場合、メールはMailウォッチャー画面でもプレビューできます。

<a name="query-watcher"></a>
### Query Watcher

Queryウォッチャーは、アプリケーションによって実行されたすべてのクエリの生のSQL、バインディング、および実行時間を記録します。ウォッチャーは、100ミリ秒以上かかるクエリに`slow`タグを付けます。ウォッチャーの`slow`オプションを使用して、遅いクエリのしきい値をカスタマイズできます。

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 50,
        ],
        ...
    ],

<a name="redis-watcher"></a>
### Redis Watcher

Redisウォッチャーは、アプリケーションによって実行されたすべての[Redis](redis.md)コマンドを記録します。キャッシュにRedisを使用している場合、キャッシュコマンドもRedisウォッチャーによって記録されます。

<a name="request-watcher"></a>
### Request Watcher

Requestウォッチャーは、アプリケーションによって処理されたリクエストに関連するリクエスト、ヘッダー、セッション、およびレスポンスデータを記録します。`size_limit`（キロバイト単位）オプションを介して、記録されるレスポンスデータを制限できます。

    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],

<a name="schedule-watcher"></a>
### Schedule Watcher

Scheduleウォッチャーは、アプリケーションによって実行された[スケジュールされたタスク](scheduling.md)のコマンドと出力を記録します。

<a name="view-watcher"></a>
### View Watcher

Viewウォッチャーは、ビューのレンダリング時に使用された[ビュー](views.md)の名前、パス、データ、および「コンポーザー」を記録します。

<a name="displaying-user-avatars"></a>
## ユーザーアバターの表示

Telescopeダッシュボードは、特定のエントリが保存されたときに認証されたユーザーのユーザーアバターを表示します。デフォルトでは、TelescopeはGravatarウェブサービスを使用してアバターを取得します。ただし、`App\Providers\TelescopeServiceProvider`クラスにコールバックを登録することで、アバターのURLをカスタマイズできます。コールバックはユーザーのIDとメールアドレスを受け取り、ユーザーのアバター画像のURLを返す必要があります。

    use App\Models\User;
    use Laravel\Telescope\Telescope;

    /**
     * アプリケーションサービスの登録
     */
    public function register(): void
    {
        // ...

        Telescope::avatar(function (string $id, string $email) {
            return '/avatars/'.User::find($id)->avatar_path;
        });
    }
