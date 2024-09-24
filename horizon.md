# Laravel Horizon

- [はじめに](#introduction)
- [インストール](#installation)
    - [設定](#configuration)
    - [バランシング戦略](#balancing-strategies)
    - [ダッシュボードの認可](#dashboard-authorization)
    - [サイレンスジョブ](#silenced-jobs)
- [Horizonのアップグレード](#upgrading-horizon)
- [Horizonの実行](#running-horizon)
    - [Horizonのデプロイ](#deploying-horizon)
- [タグ](#tags)
- [通知](#notifications)
- [メトリクス](#metrics)
- [失敗したジョブの削除](#deleting-failed-jobs)
- [キューからのジョブのクリア](#clearing-jobs-from-queues)

<a name="introduction"></a>
## はじめに

> NOTE:  
> Laravel Horizonを掘り下げる前に、Laravelの基本[キューサービス](queues.md)に慣れておくべきです。HorizonはLaravelのキューに追加機能を提供しますが、それらの機能はLaravelが提供する基本的なキュー機能に慣れていないと混乱するかもしれません。

[Laravel Horizon](https://github.com/laravel/horizon)は、Laravelが動作する[Redisキュー](queues.md)のための美しいダッシュボードとコード駆動の設定を提供します。Horizonを使用すると、ジョブのスループット、実行時間、ジョブの失敗など、キューシステムの主要なメトリクスを簡単に監視できます。

Horizonを使用すると、すべてのキューワーカーの設定が単一のシンプルな設定ファイルに保存されます。アプリケーションのワーカー設定をバージョン管理されたファイルで定義することで、アプリケーションをデプロイする際にキューワーカーを簡単にスケールまたは変更できます。

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## インストール

> WARNING:  
> Laravel Horizonでは、キューを動作させるために[Redis](https://redis.io)を使用する必要があります。したがって、キュー接続がアプリケーションの`config/queue.php`設定ファイルで`redis`に設定されていることを確認してください。

Composerパッケージマネージャを使用して、プロジェクトにHorizonをインストールできます:

```shell
composer require laravel/horizon
```

Horizonをインストールした後、`horizon:install` Artisanコマンドを使用してそのアセットを公開します:

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### 設定

Horizonのアセットを公開した後、その主要な設定ファイルは`config/horizon.php`に配置されます。この設定ファイルでは、アプリケーションのキューワーカーオプションを設定できます。各設定オプションにはその目的の説明が含まれているため、このファイルを徹底的に確認してください。

> WARNING:  
> Horizonは内部で`horizon`という名前のRedis接続を使用します。このRedis接続名は予約されており、`database.php`設定ファイル内または`horizon.php`設定ファイルの`use`オプションの値として別のRedis接続に割り当てるべきではありません。

<a name="environments"></a>
#### 環境

インストール後、主なHorizon設定オプションで親しむべきは`environments`設定オプションです。この設定オプションは、アプリケーションが実行される環境の配列であり、各環境のワーカープロセスオプションを定義します。デフォルトでは、このエントリには`production`と`local`環境が含まれています。ただし、必要に応じてさらに環境を追加することができます:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

また、他の一致する環境が見つからない場合に使用されるワイルドカード環境（`*`）を定義することもできます:

    'environments' => [
        // ...

        '*' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

Horizonを起動すると、アプリケーションが実行されている環境のワーカープロセス設定オプションが使用されます。通常、環境は`APP_ENV`[環境変数](configuration.md#determining-the-current-environment)の値によって決定されます。たとえば、デフォルトの`local` Horizon環境は3つのワーカープロセスを開始し、各キューに割り当てられたワーカープロセスの数を自動的にバランスさせるように設定されています。デフォルトの`production`環境は最大10のワーカープロセスを開始し、各キューに割り当てられたワーカープロセスの数を自動的にバランスさせるように設定されています。

> WARNING:  
> Horizonを実行する予定の各[環境](configuration.md#environment-configuration)に対して、`horizon`設定ファイルの`environments`部分にエントリが含まれていることを確認する必要があります。

<a name="supervisors"></a>
#### スーパーバイザ

Horizonのデフォルト設定ファイルでわかるように、各環境には1つ以上の「スーパーバイザ」を含めることができます。デフォルトでは、このスーパーバイザは`supervisor-1`として定義されています。ただし、スーパーバイザには自由に名前を付けることができます。各スーパーバイザは基本的にワーカープロセスのグループを「監視」し、ワーカープロセスをキュー間でバランスさせる役割を果たします。

特定の環境に追加のスーパーバイザを追加することができます。これは、その環境で実行される新しいワーカープロセスのグループを定義したい場合に行います。これを行う場合、アプリケーションが使用する特定のキューに対して異なるバランシング戦略またはワーカープロセス数を定義することができます。

<a name="maintenance-mode"></a>
#### メンテナンスモード

アプリケーションが[メンテナンスモード](configuration.md#maintenance-mode)にある間、キューに入れられたジョブはHorizonによって処理されません。ただし、スーパーバイザの`force`オプションがHorizon設定ファイル内で`true`と定義されている場合を除きます:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                // ...
                'force' => true,
            ],
        ],
    ],

<a name="default-values"></a>
#### デフォルト値

Horizonのデフォルト設定ファイル内で、`defaults`設定オプションが見つかります。この設定オプションは、アプリケーションの[スーパーバイザ](#supervisors)のデフォルト値を指定します。スーパーバイザのデフォルト設定値は、各環境のスーパーバイザの設定にマージされ、スーパーバイザを定義する際の不要な繰り返しを避けることができます。

<a name="balancing-strategies"></a>
### バランシング戦略

Laravelのデフォルトキューシステムとは異なり、Horizonでは3つのワーカーバランシング戦略から選択できます: `simple`、`auto`、`false`。`simple`戦略は、受信ジョブをワーカープロセス間で均等に分割します:

    'balance' => 'simple',

`auto`戦略は、設定ファイルのデフォルトであり、キューの現在の負荷に基づいて各キューのワーカープロセス数を調整します。たとえば、`notifications`キューに1,000の保留中のジョブがあり、`render`キューが空の場合、Horizonは`notifications`キューにより多くのワーカーを割り当て、キューが空になるまで続けます。

`auto`戦略を使用する場合、`minProcesses`と`maxProcesses`設定オプションを定義して、各キューの最小プロセス数と、Horizonがスケールアップおよびスケールダウンするワーカープロセスの最大数を制御できます:

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'autoScalingStrategy' => 'time',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],

`autoScalingStrategy`設定値は、Horizonがキューをクリアするのにかかる合計時間（`time`戦略）に基づいて、またはキュー上のジョブの合計数（`size`戦略）に基づいて、より多くのワーカープロセスをキューに割り当てるかどうかを決定します。

`balanceMaxShift`と`balanceCooldown`設定値は、Horizonがワーカーの需要に合わせてどれだけ迅速にスケールするかを決定します。上記の例では、最大で1つの新しいプロセスが3秒ごとに作成または破棄されます。アプリケーションのニーズに応じて、これらの値を自由に調整できます。

`balance`オプションが`false`に設定されている場合、デフォルトのLaravelの動作が使用されます。これは、キューが設定にリストされている順序で処理されることを意味します。

<a name="dashboard-authorization"></a>
### ダッシュボードの認可

Horizonダッシュボードは`/horizon`ルートからアクセスできます。デフォルトでは、このダッシュボードには`local`環境でのみアクセスできます。ただし、`app/Providers/HorizonServiceProvider.php`ファイル内には[認可ゲート](authorization.md#gates)の定義があります。この認可ゲートは、**非ローカル**環境でのHorizonへのアクセスを制御します。Horizonのインストールへのアクセスを制限するために、必要に応じてこのゲートを自由に変更できます:

    /**
     * Register the Horizon gate.
     *
     * This gate determines who can access Horizon in non-local environments.
     */
    protected function gate(): void
    {
        Gate::define('viewHorizon', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="alternative-authentication-strategies"></a>
#### 代替認証戦略

Laravelは自動的に認証されたユーザーをゲートクロージャに注入することを覚えておいてください。アプリケーションがIP制限などの別の方法でHorizonのセキュリティを提供している場合、Horizonユーザーは「ログイン」する必要がないかもしれません。したがって、上記の`function (User $user)`クロージャシグネチャを`function (User $user = null)`に変更して、Laravelに認証を要求しないようにする必要があります。

<a name="silenced-jobs"></a>
### サイレンスされたジョブ

時には、アプリケーションやサードパーティパッケージによってディスパッチされる特定のジョブを表示することに興味がない場合があります。これらのジョブが「完了したジョブ」リストにスペースを取らないように、それらをサイレンスすることができます。始めるには、ジョブのクラス名をアプリケーションの `horizon` 設定ファイルの `silenced` 設定オプションに追加します。

    'silenced' => [
        App\Jobs\ProcessPodcast::class,
    ],

または、サイレンスしたいジョブが `Laravel\Horizon\Contracts\Silenced` インターフェースを実装することもできます。ジョブがこのインターフェースを実装している場合、`silenced` 設定配列に存在しなくても自動的にサイレンスされます。

    use Laravel\Horizon\Contracts\Silenced;

    class ProcessPodcast implements ShouldQueue, Silenced
    {
        use Queueable;

        // ...
    }

<a name="upgrading-horizon"></a>
## Horizonのアップグレード

新しいメジャーバージョンのHorizonにアップグレードする際は、[アップグレードガイド](https://github.com/laravel/horizon/blob/master/UPGRADE.md)を注意深く確認することが重要です。

<a name="running-horizon"></a>
## Horizonの実行

アプリケーションの `config/horizon.php` 設定ファイルでスーパーバイザとワーカーを設定したら、`horizon` Artisanコマンドを使用してHorizonを起動できます。この単一のコマンドは、現在の環境に対して設定されたすべてのワーカープロセスを開始します。

```shell
php artisan horizon
```

`horizon:pause` および `horizon:continue` Artisanコマンドを使用して、Horizonプロセスを一時停止し、ジョブの処理を続行するように指示できます。

```shell
php artisan horizon:pause

php artisan horizon:continue
```

特定のHorizon [スーパーバイザ](#supervisors) を一時停止および続行するには、`horizon:pause-supervisor` および `horizon:continue-supervisor` Artisanコマンドを使用できます。

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

`horizon:status` Artisanコマンドを使用して、Horizonプロセスの現在のステータスを確認できます。

```shell
php artisan horizon:status
```

`horizon:terminate` Artisanコマンドを使用して、Horizonプロセスを正常に終了できます。現在処理中のジョブは完了し、その後Horizonは実行を停止します。

```shell
php artisan horizon:terminate
```

<a name="deploying-horizon"></a>
### Horizonのデプロイ

Horizonをアプリケーションの実際のサーバーにデプロイする準備ができたら、`php artisan horizon` コマンドを監視し、予期せず終了した場合に再起動するようにプロセスモニタを設定する必要があります。心配はいりません。以下でプロセスモニタのインストール方法について説明します。

アプリケーションのデプロイプロセス中に、Horizonプロセスに終了するように指示して、プロセスモニタによって再起動され、コードの変更を受け取るようにする必要があります。

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### Supervisorのインストール

SupervisorはLinuxオペレーティングシステムのプロセスモニタであり、`horizon` プロセスが実行を停止した場合に自動的に再起動します。UbuntuにSupervisorをインストールするには、次のコマンドを使用できます。Ubuntu以外を使用している場合は、オペレーティングシステムのパッケージマネージャを使用してSupervisorをインストールできます。

```shell
sudo apt-get install supervisor
```

> NOTE:  
> Supervisorの設定が難しいと感じる場合は、[Laravel Forge](https://forge.laravel.com)の使用を検討してください。これにより、Laravelプロジェクトに対して自動的にSupervisorがインストールおよび設定されます。

<a name="supervisor-configuration"></a>
#### Supervisorの設定

Supervisorの設定ファイルは通常、サーバーの `/etc/supervisor/conf.d` ディレクトリに保存されます。このディレクトリ内に、Supervisorにプロセスの監視方法を指示する任意の数の設定ファイルを作成できます。例えば、`horizon` プロセスを開始および監視する `horizon.conf` ファイルを作成しましょう。

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

Supervisorの設定を定義する際に、`stopwaitsecs` の値が最も長い実行ジョブによって消費される秒数よりも大きいことを確認する必要があります。そうしないと、Supervisorはジョブが処理を完了する前にジョブを強制終了する可能性があります。

> WARNING:  
> 上記の例はUbuntuベースのサーバーに有効ですが、Supervisor設定ファイルの場所と期待されるファイル拡張子は、他のサーバーオペレーティングシステムによって異なる場合があります。サーバーのドキュメントを参照して詳細を確認してください。

<a name="starting-supervisor"></a>
#### Supervisorの起動

設定ファイルが作成されたら、次のコマンドを使用してSupervisorの設定を更新し、監視対象のプロセスを開始できます。

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> NOTE:  
> Supervisorの実行に関する詳細は、[Supervisorのドキュメント](http://supervisord.org/index.html)を参照してください。

<a name="tags"></a>
## タグ

Horizonでは、メール、ブロードキャストイベント、通知、キューイベントリスナーなどのジョブに「タグ」を割り当てることができます。実際、HorizonはEloquentモデルがジョブに添付されている場合、ほとんどのジョブをインテリジェントに自動的にタグ付けします。例えば、次のジョブを見てみましょう。

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Queue\Queueable;

    class RenderVideo implements ShouldQueue
    {
        use Queueable;

        /**
         * Create a new job instance.
         */
        public function __construct(
            public Video $video,
        ) {}

        /**
         * Execute the job.
         */
        public function handle(): void
        {
            // ...
        }
    }

このジョブが `id` 属性が `1` の `App\Models\Video` インスタンスでキューに入れられた場合、自動的に `App\Models\Video:1` というタグを受け取ります。これは、Horizonがジョブのプロパティを検索し、Eloquentモデルが見つかった場合、モデルのクラス名と主キーを使用してインテリジェントにジョブにタグ付けするためです。

    use App\Jobs\RenderVideo;
    use App\Models\Video;

    $video = Video::find(1);

    RenderVideo::dispatch($video);

<a name="manually-tagging-jobs"></a>
#### ジョブの手動タグ付け

キュー可能なオブジェクトの1つに対して手動でタグを定義したい場合は、クラスに `tags` メソッドを定義できます。

    class RenderVideo implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the job.
         *
         * @return array<int, string>
         */
        public function tags(): array
        {
            return ['render', 'video:'.$this->video->id];
        }
    }

<a name="manually-tagging-event-listeners"></a>
#### イベントリスナーの手動タグ付け

キューイベントリスナーのタグを取得する際、Horizonは自動的にイベントインスタンスを `tags` メソッドに渡し、イベントデータをタグに追加できるようにします。

    class SendRenderNotifications implements ShouldQueue
    {
        /**
         * Get the tags that should be assigned to the listener.
         *
         * @return array<int, string>
         */
        public function tags(VideoRendered $event): array
        {
            return ['video:'.$event->video->id];
        }
    }

<a name="notifications"></a>
## 通知

> WARNING:  
> Horizonを設定してSlackまたはSMS通知を送信する場合、関連する通知チャネルの[前提条件](notifications.md)を確認する必要があります。

キューの待ち時間が長い場合に通知を受け取りたい場合は、`Horizon::routeMailNotificationsTo`、`Horizon::routeSlackNotificationsTo`、および `Horizon::routeSmsNotificationsTo` メソッドを使用できます。これらのメソッドは、アプリケーションの `App\Providers\HorizonServiceProvider` の `boot` メソッドから呼び出すことができます。

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        parent::boot();

        Horizon::routeSmsNotificationsTo('15556667777');
        Horizon::routeMailNotificationsTo('example@example.com');
        Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
    }

<a name="configuring-notification-wait-time-thresholds"></a>
#### 通知待ち時間のしきい値の設定

アプリケーションの `config/horizon.php` 設定ファイル内で、「長い待ち時間」と見なされる秒数を設定できます。このファイル内の `waits` 設定オプションを使用すると、各接続/キューの組み合わせの長い待ち時間のしきい値を制御できます。定義されていない接続/キューの組み合わせは、デフォルトで60秒の長い待ち時間のしきい値を持ちます。

    'waits' => [
        'redis:critical' => 30,
        'redis:default' => 60,
        'redis:batch' => 120,
    ],

<a name="metrics"></a>
## メトリクス

Horizonには、ジョブとキューの待ち時間およびスループットに関する情報を提供するメトリクスダッシュボードが含まれています。このダッシュボードを設定するには、アプリケーションの `routes/console.php` ファイル内で、Horizonの `snapshot` Artisanコマンドが5分ごとに実行されるように設定する必要があります。

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('horizon:snapshot')->everyFiveMinutes();

<a name="deleting-failed-jobs"></a>
## 失敗したジョブの削除

失敗したジョブを削除したい場合は、`horizon:forget` コマンドを使用できます。`horizon:forget` コマンドは、失敗したジョブのIDまたはUUIDを引数として受け取ります。

```shell
php artisan horizon:forget 5
```

失敗したすべてのジョブを削除したい場合は、`horizon:forget` コマンドに `--all` オプションを指定できます。

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## キューからジョブをクリアする

アプリケーションのデフォルトキューからすべてのジョブを削除したい場合は、`horizon:clear` Artisanコマンドを使用できます。

```shell
php artisan horizon:clear
```

特定のキューからジョブを削除するために、`queue`オプションを指定することもできます。

```shell
php artisan horizon:clear --queue=emails
```
