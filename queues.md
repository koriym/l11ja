# キュー

- [はじめに](#introduction)
    - [接続とキュー](#connections-vs-queues)
    - [ドライバの注意事項と前提条件](#driver-prerequisites)
- [ジョブの作成](#creating-jobs)
    - [ジョブクラスの生成](#generating-job-classes)
    - [クラス構造](#class-structure)
    - [ユニークジョブ](#unique-jobs)
    - [暗号化ジョブ](#encrypted-jobs)
- [ジョブミドルウェア](#job-middleware)
    - [レート制限](#rate-limiting)
    - [ジョブの重複防止](#preventing-job-overlaps)
    - [例外のスロットリング](#throttling-exceptions)
    - [ジョブのスキップ](#skipping-jobs)
- [ジョブのディスパッチ](#dispatching-jobs)
    - [遅延ディスパッチ](#delayed-dispatching)
    - [同期ディスパッチ](#synchronous-dispatching)
    - [ジョブとデータベーストランザクション](#jobs-and-database-transactions)
    - [ジョブの連鎖](#job-chaining)
    - [キューと接続のカスタマイズ](#customizing-the-queue-and-connection)
    - [最大試行回数 / タイムアウト値の指定](#max-job-attempts-and-timeout)
    - [エラー処理](#error-handling)
- [ジョブのバッチ処理](#job-batching)
    - [バッチ可能なジョブの定義](#defining-batchable-jobs)
    - [バッチのディスパッチ](#dispatching-batches)
    - [連鎖とバッチ](#chains-and-batches)
    - [バッチへのジョブの追加](#adding-jobs-to-batches)
    - [バッチの検査](#inspecting-batches)
    - [バッチのキャンセル](#cancelling-batches)
    - [バッチの失敗](#batch-failures)
    - [バッチの整理](#pruning-batches)
    - [DynamoDBへのバッチの保存](#storing-batches-in-dynamodb)
- [クロージャのキューイング](#queueing-closures)
- [キューワーカーの実行](#running-the-queue-worker)
    - [`queue:work`コマンド](#the-queue-work-command)
    - [キューの優先度](#queue-priorities)
    - [キューワーカーとデプロイ](#queue-workers-and-deployment)
    - [ジョブの有効期限とタイムアウト](#job-expirations-and-timeouts)
- [Supervisorの設定](#supervisor-configuration)
- [失敗したジョブの処理](#dealing-with-failed-jobs)
    - [失敗したジョブの後処理](#cleaning-up-after-failed-jobs)
    - [失敗したジョブの再試行](#retrying-failed-jobs)
    - [モデルの欠落を無視する](#ignoring-missing-models)
    - [失敗したジョブの整理](#pruning-failed-jobs)
    - [DynamoDBへの失敗したジョブの保存](#storing-failed-jobs-in-dynamodb)
    - [失敗したジョブの保存の無効化](#disabling-failed-job-storage)
    - [失敗したジョブのイベント](#failed-job-events)
- [キューからのジョブのクリア](#clearing-jobs-from-queues)
- [キューの監視](#monitoring-your-queues)
- [テスト](#testing)
    - [ジョブの一部をフェイクする](#faking-a-subset-of-jobs)
    - [ジョブの連鎖のテスト](#testing-job-chains)
    - [ジョブのバッチのテスト](#testing-job-batches)
    - [ジョブ / キューの相互作用のテスト](#testing-job-queue-interactions)
- [ジョブイベント](#job-events)

<a name="introduction"></a>
## はじめに

Webアプリケーションを構築していると、CSVファイルの解析やアップロードされたファイルの保存など、一般的なWebリクエスト中に実行するには時間がかかりすぎるタスクが発生することがあります。幸いなことに、Laravelでは簡単にキューに入れられるジョブを作成し、バックグラウンドで処理することができます。時間のかかるタスクをキューに移すことで、アプリケーションはWebリクエストに対して高速に応答し、顧客により良いユーザーエクスペリエンスを提供することができます。

Laravelのキューは、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io)、リレーショナルデータベースなど、さまざまなキューバックエンドに対して統一されたキューイングAPIを提供します。

Laravelのキュー設定オプションは、アプリケーションの`config/queue.php`設定ファイルに保存されています。このファイルには、フレームワークに含まれる各キュードライバの接続設定が含まれています。これには、データベース、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io)、[Beanstalkd](https://beanstalkd.github.io/)ドライバ、およびジョブを即座に実行する同期ドライバ（ローカル開発用）が含まれます。`null`キュードライバも含まれており、キューに入れられたジョブを破棄します。

> NOTE:  
> Laravelは現在、Redisを使用したキューのための美しいダッシュボードと設定システムであるHorizonを提供しています。詳細については、完全な[Horizonドキュメント](horizon.md)を確認してください。

<a name="connections-vs-queues"></a>
### 接続とキュー

Laravelのキューを始める前に、「接続」と「キュー」の違いを理解することが重要です。`config/queue.php`設定ファイルには、`connections`設定配列があります。このオプションは、Amazon SQS、Beanstalk、Redisなどのバックエンドキューサービスへの接続を定義します。ただし、特定のキュー接続には、キューに入れられたジョブの異なるスタックまたはパイルと考えることができる複数の「キュー」があります。

`queue`設定ファイルの各接続設定例には、`queue`属性が含まれていることに注意してください。これは、ジョブが特定の接続に送信されるときにデフォルトでディスパッチされるキューです。言い換えると、ジョブをディスパッチするときに明示的にどのキューにディスパッチするかを定義しない場合、ジョブは接続設定の`queue`属性で定義されたキューに配置されます。

```php
use App\Jobs\ProcessPodcast;

// このジョブはデフォルト接続のデフォルトキューに送信されます...
ProcessPodcast::dispatch();

// このジョブはデフォルト接続の "emails" キューに送信されます...
ProcessPodcast::dispatch()->onQueue('emails');
```

一部のアプリケーションでは、複数のキューにジョブをプッシュする必要がないかもしれませんが、代わりに単純なキューを1つ持つことを好みます。しかし、複数のキューにジョブをプッシュすることは、ジョブの処理方法を優先順位付けしたり、セグメント化したりすることを望むアプリケーションにとって特に便利です。Laravelのキューワーカーでは、処理するキューを優先順位で指定できるためです。たとえば、`high`キューにジョブをプッシュする場合、より高い処理優先度を持つワーカーを実行できます。

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### ドライバの注意事項と前提条件

<a name="database"></a>
#### データベース

`database`キュードライバを使用するには、ジョブを保持するためのデータベーステーブルが必要です。通常、これはLaravelのデフォルトの`0001_01_01_000002_create_jobs_table.php`[データベースマイグレーション](migrations.md)に含まれています。ただし、アプリケーションにこのマイグレーションが含まれていない場合は、`make:queue-table` Artisanコマンドを使用して作成できます。

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

`redis`キュードライバを使用するには、`config/database.php`設定ファイルでRedisデータベース接続を設定する必要があります。

> WARNING:  
> `serializer`と`compression`のRedisオプションは、`redis`キュードライバではサポートされていません。

**Redisクラスタ**

Redisキュー接続がRedisクラスタを使用している場合、キュー名には[キーのハッシュタグ](https://redis.io/docs/reference/cluster-spec/#hash-tags)を含める必要があります。これは、特定のキューのすべてのRedisキーが同じハッシュスロットに配置されるようにするために必要です。

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

**ブロッキング**

Redisキューを使用する場合、`block_for`設定オプションを使用して、ドライバがジョブが利用可能になるまで待機する時間を指定できます。これにより、ワーカーループを繰り返し、Redisデータベースを再ポーリングする前に待機するよりも効率的になります。たとえば、ジョブが利用可能になるまで5秒間ブロックするように値を`5`に設定できます。

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

> WARNING:  
> `block_for`を`0`に設定すると、キューワーカーはジョブが利用可能になるまで無期限にブロックします。これにより、次のジョブが処理されるまで`SIGTERM`などのシグナルが処理されなくなります。

<a name="other-driver-prerequisites"></a>
#### その他のドライバの前提条件

以下の依存関係は、リストされたキュードライバに必要です。これらの依存関係は、Composerパッケージマネージャを介してインストールできます。

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` または phpredis PHP拡張

</div>

<a name="creating-jobs"></a>
## ジョブの作成

<a name="generating-job-classes"></a>
### ジョブクラスの生成

デフォルトでは、アプリケーションのすべてのキュー可能なジョブは、`app/Jobs`ディレクトリに保存されます。`app/Jobs`ディレクトリが存在しない場合、`make:job` Artisanコマンドを実行すると作成されます。

```shell
php artisan make:job ProcessPodcast
```

生成されたクラスは、`Illuminate\Contracts\Queue\ShouldQueue`インターフェースを実装しており、Laravelにジョブを非同期で実行するためにキューにプッシュする必要があることを示します。

> NOTE:  
> ジョブスタブは、[スタブの公開](artisan.md#stub-customization)を使用してカスタマイズできます。

<a name="class-structure"></a>
### クラス構造

ジョブクラスは非常にシンプルで、通常、ジョブがキューによって処理されるときに呼び出される`handle`メソッドのみを含みます。始めるために、ジョブクラスの例を見てみましょう。この例では、ポッドキャストの公開サービスを管理しており、アップロードされたポッドキャストファイルを公開する前に処理する必要があると想定します。

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * 新しいジョブインスタンスの作成
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * ジョブの実行
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor): void
    {
        // ポッドキャストファイルの処理
    }
}
```

```php
use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * 新しいジョブインスタンスの生成
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * ジョブの実行
     */
    public function handle(AudioProcessor $processor): void
    {
        // アップロードされたポッドキャストの処理...
    }
}
```

この例では、キューに入れられたジョブのコンストラクタに直接[Eloquentモデル](eloquent.md)を渡すことができることに注目してください。ジョブが使用している`Queueable`トレイトのおかげで、Eloquentモデルとその読み込まれたリレーションは、ジョブが処理される際に適切にシリアライズおよびアンシリアライズされます。

キューに入れられたジョブがコンストラクタでEloquentモデルを受け取る場合、モデルの識別子のみがキューにシリアライズされます。ジョブが実際に処理されるとき、キューシステムは自動的にデータベースから完全なモデルインスタンスとその読み込まれたリレーションを再取得します。このモデルのシリアライズ方法により、キュードライバに送信されるジョブのペイロードを大幅に小さくすることができます。

<a name="handle-method-dependency-injection"></a>
#### `handle`メソッドの依存性注入

`handle`メソッドは、ジョブがキューによって処理されるときに呼び出されます。`handle`メソッドのジョブに依存関係をタイプヒントできることに注意してください。Laravelの[サービスコンテナ](container.md)が自動的にこれらの依存関係を注入します。

コンテナが`handle`メソッドに依存関係を注入する方法を完全に制御したい場合は、コンテナの`bindMethod`メソッドを使用できます。`bindMethod`メソッドは、ジョブとコンテナを受け取るコールバックを受け取ります。コールバック内で、`handle`メソッドを好きなように呼び出すことができます。通常、このメソッドは`App\Providers\AppServiceProvider`[サービスプロバイダ](providers.md)の`boot`メソッドから呼び出すべきです：

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> WARNING:  
> 生の画像コンテンツなどのバイナリデータは、キューに入れられたジョブに渡される前に`base64_encode`関数を通過させる必要があります。そうしないと、ジョブがキューに配置されるときにJSONに正しくシリアライズされない可能性があります。

<a name="handling-relationships"></a>
#### キューに入れられたリレーション

ジョブがキューに入れられるとき、読み込まれたすべてのEloquentモデルのリレーションもシリアライズされるため、シリアライズされたジョブ文字列が非常に大きくなることがあります。さらに、ジョブがデシリアライズされ、モデルのリレーションがデータベースから再取得されるとき、それらは完全に取得されます。ジョブがキューに入れられるプロセス中にモデルがシリアライズされる前に適用された以前のリレーション制約は、ジョブがデシリアライズされるときに適用されません。したがって、特定のリレーションのサブセットで作業したい場合は、キューに入れられたジョブ内でそのリレーションを再制約する必要があります。

または、リレーションがシリアライズされないようにするために、プロパティ値を設定するときにモデルの`withoutRelations`メソッドを呼び出すことができます。このメソッドは、読み込まれたリレーションを持たないモデルのインスタンスを返します：

```php
/**
 * 新しいジョブインスタンスの生成
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

PHPのコンストラクタプロパティプロモーションを使用していて、Eloquentモデルがそのリレーションをシリアライズしないように指定したい場合は、`WithoutRelations`属性を使用できます：

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * 新しいジョブインスタンスの生成
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

ジョブが単一のモデルではなく、Eloquentモデルのコレクションまたは配列を受け取る場合、そのコレクション内のモデルは、ジョブがデシリアライズおよび実行されるときにそのリレーションが復元されません。これは、多数のモデルを扱うジョブで過剰なリソース使用を防ぐためです。

<a name="unique-jobs"></a>
### ユニークジョブ

> WARNING:  
> ユニークジョブには、[ロック](cache.md#atomic-locks)をサポートするキャッシュドライバが必要です。現在、`memcached`、`redis`、`dynamodb`、`database`、`file`、および`array`キャッシュドライバがアトミックロックをサポートしています。さらに、ユニークジョブの制約はバッチ内のジョブには適用されません。

特定のジョブのインスタンスがキューに一度に1つしか存在しないようにしたい場合があります。そのためには、ジョブクラスに`ShouldBeUnique`インターフェースを実装することができます。このインターフェースは、クラスに追加のメソッドを定義する必要はありません：

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    ...
}
```

上記の例では、`UpdateSearchIndex`ジョブはユニークです。したがって、ジョブの別のインスタンスがすでにキューにあり、処理が完了していない場合、ジョブはディスパッチされません。

特定のケースでは、ジョブをユニークにする特定の「キー」を定義したり、ジョブがユニークでなくなるまでのタイムアウトを指定したい場合があります。これを実現するために、ジョブクラスに`uniqueId`および`uniqueFor`プロパティまたはメソッドを定義できます：

```php
<?php

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * 製品インスタンス
     *
     * @var \App\Product
     */
    public $product;

    /**
     * ジョブのユニークロックが解除されるまでの秒数
     *
     * @var int
     */
    public $uniqueFor = 3600;

    /**
     * ジョブのユニークIDを取得
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```

上記の例では、`UpdateSearchIndex`ジョブは製品IDによってユニークです。したがって、同じ製品IDを持つジョブの新しいディスパッチは、既存のジョブが処理を完了するまで無視されます。さらに、既存のジョブが1時間以内に処理されない場合、ユニークロックは解除され、同じユニークキーを持つ別のジョブをキューにディスパッチできます。

> WARNING:  
> アプリケーションが複数のWebサーバーまたはコンテナからジョブをディスパッチする場合、すべてのサーバーが同じ中央キャッシュサーバーと通信するようにして、Laravelがジョブがユニークであるかどうかを正確に判断できるようにする必要があります。

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### 処理が開始されるまでジョブをユニークに保つ

デフォルトでは、ユニークジョブはジョブが処理を完了するか、すべての再試行が失敗した後に「ロック解除」されます。ただし、ジョブが処理される直前にジョブのロックを解除したい場合があります。これを実現するために、ジョブは`ShouldBeUnique`インターフェースの代わりに`ShouldBeUniqueUntilProcessing`インターフェースを実装する必要があります：

```php
<?php

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-job-locks"></a>
#### ユニークジョブロック

舞台裏では、`ShouldBeUnique`ジョブがディスパッチされると、Laravelは`uniqueId`キーを使用して[ロック](cache.md#atomic-locks)を取得しようとします。ロックが取得されない場合、ジョブはディスパッチされません。このロックは、ジョブが処理を完了するか、すべての再試行が失敗した後に解放されます。デフォルトでは、Laravelはこのロックを取得するためにデフォルトのキャッシュドライバを使用します。ただし、ロックを取得するために別のドライバを使用したい場合は、`uniqueVia`メソッドを定義して、使用するキャッシュドライバを返すことができます：

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    ...

    /**
     * ユニークジョブロックのためのキャッシュドライバを取得
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}
```

> NOTE:  
> ジョブの同時処理を制限するだけの場合は、代わりに[`WithoutOverlapping`](queues.md#preventing-job-overlaps)ジョブミドルウェアを使用してください。

<a name="encrypted-jobs"></a>
### 暗号化されたジョブ

Laravelでは、[暗号化](encryption.md)を介してジョブのデータのプライバシーと整合性を確保できます。開始するには、ジョブクラスに`ShouldBeEncrypted`インターフェースを追加するだけです。このインターフェースがクラスに追加されると、Laravelはジョブをキューにプッシュする前に自動的にジョブを暗号化します：

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="job-middleware"></a>
## ジョブミドルウェア

ジョブミドルウェアを使用すると、キューに入れられたジョブの実行を囲むカスタムロジックをラップでき、ジョブ自体のボイラープレートを減らすことができます。たとえば、LaravelのRedisレート制限機能を利用して、5秒ごとに1つのジョブのみを処理する`handle`メソッドを考えてみましょう：

```php
use Illuminate\Support\Facades\Redis;
```

```php
    /**
     * ジョブを実行する。
     */
    public function handle(): void
    {
        Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
            info('Lock obtained...');

            // ジョブを処理する...
        }, function () {
            // ロックを取得できなかった...

            return $this->release(5);
        });
    }
```

このコードは有効ですが、Redisのレート制限ロジックが混ざっているため、`handle`メソッドの実装が煩雑になります。さらに、このレート制限ロジックは、レート制限したい他のジョブにも重複して記述する必要があります。

代わりに、`handle`メソッド内でレート制限を行うのではなく、レート制限を処理するジョブミドルウェアを定義することができます。Laravelにはジョブミドルウェアのデフォルトの場所がないため、アプリケーション内のどこにでもジョブミドルウェアを配置できます。この例では、ミドルウェアを`app/Jobs/Middleware`ディレクトリに配置します。

```php
<?php

namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * キューに入れられたジョブを処理する。
     *
     * @param  \Closure(object): void  $next
     */
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
                ->block(0)->allow(1)->every(5)
                ->then(function () use ($job, $next) {
                    // ロックを取得した...

                    $next($job);
                }, function () use ($job) {
                    // ロックを取得できなかった...

                    $job->release(5);
                });
    }
}
```

ご覧の通り、[ルートミドルウェア](middleware.md)と同様に、ジョブミドルウェアは処理中のジョブと、ジョブの処理を続行するために呼び出す必要があるコールバックを受け取ります。

ジョブミドルウェアを作成した後、ジョブの`middleware`メソッドから返すことでジョブにアタッチできます。このメソッドは`make:job` Artisanコマンドでスキャフォールドされたジョブには存在しないため、ジョブクラスに手動で追加する必要があります。

```php
use App\Jobs\Middleware\RateLimited;

/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited];
}
```

> NOTE:  
> ジョブミドルウェアは、キュー可能なイベントリスナー、メール、通知にも割り当てることができます。

<a name="rate-limiting"></a>
### レート制限

Laravelには実際には、ジョブのレート制限に利用できるレート制限ミドルウェアが含まれています。[ルートレートリミッター](routing.md#defining-rate-limiters)と同様に、ジョブレートリミッターは`RateLimiter`ファサードの`for`メソッドを使用して定義されます。

例えば、ユーザーがデータを1時間に1回バックアップできるようにしたいが、プレミアム顧客にはそのような制限を課したくない場合、`AppServiceProvider`の`boot`メソッドで`RateLimiter`を定義できます。

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

/**
 * 任意のアプリケーションサービスをブートストラップする。
 */
public function boot(): void
{
    RateLimiter::for('backups', function (object $job) {
        return $job->user->vipCustomer()
                    ? Limit::none()
                    : Limit::perHour(1)->by($job->user->id);
    });
}
```

上記の例では、1時間ごとのレート制限を定義しました。ただし、`perMinute`メソッドを使用して分単位でレート制限を簡単に定義することもできます。さらに、レート制限の`by`メソッドには、任意の値を渡すことができます。ただし、この値は通常、顧客ごとにレート制限を分割するために使用されます。

```php
return Limit::perMinute(50)->by($job->user->id);
```

レート制限を定義したら、`Illuminate\Queue\Middleware\RateLimited`ミドルウェアを使用してジョブにレートリミッターをアタッチできます。このミドルウェアは、ジョブがレート制限を超えるたびに、レート制限期間に基づいて適切な遅延でジョブをキューに戻します。

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

レート制限されたジョブをキューに戻すと、ジョブの合計`attempts`数が増加します。ジョブクラスの`tries`および`maxExceptions`プロパティを適切に調整するか、[`retryUntil`メソッド](#time-based-attempts)を使用して、ジョブが再試行されなくなるまでの時間を定義することをお勧めします。

レート制限された場合にジョブを再試行したくない場合は、`dontRelease`メソッドを使用できます。

```php
/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

> NOTE:  
> Redisを使用している場合は、`Illuminate\Queue\Middleware\RateLimitedWithRedis`ミドルウェアを使用できます。これはRedis向けに微調整されており、基本的なレート制限ミドルウェアよりも効率的です。

<a name="preventing-job-overlaps"></a>
### ジョブの重複を防ぐ

Laravelには、任意のキーに基づいてジョブの重複を防ぐための`Illuminate\Queue\Middleware\WithoutOverlapping`ミドルウェアが含まれています。これは、一度に1つのジョブだけが変更できるリソースをキューに入れられたジョブが変更する場合に便利です。

例えば、ユーザーの信用スコアを更新するキューに入れられたジョブがあり、同じユーザーIDに対する信用スコア更新ジョブの重複を防ぎたいとします。これを実現するには、ジョブの`middleware`メソッドから`WithoutOverlapping`ミドルウェアを返します。

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

同じタイプの重複するジョブはすべてキューに戻されます。指定された秒数が経過するまで、解放されたジョブが再試行されるようにすることもできます。

```php
/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

重複するジョブを再試行しないようにするには、`dontRelease`メソッドを使用できます。

```php
/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

`WithoutOverlapping`ミドルウェアはLaravelのアトミックロック機能によって動作します。時々、ジョブが予期せず失敗したりタイムアウトしたりして、ロックが解放されないことがあります。したがって、`expireAfter`メソッドを使用して明示的にロックの有効期限を定義できます。例えば、以下の例では、ジョブが処理を開始してから3分後に`WithoutOverlapping`ロックを解放するようLaravelに指示します。

```php
/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

> WARNING:  
> `WithoutOverlapping`ミドルウェアには、[ロック](cache.md#atomic-locks)をサポートするキャッシュドライバが必要です。現在、`memcached`、`redis`、`dynamodb`、`database`、`file`、`array`キャッシュドライバはアトミックロックをサポートしています。

<a name="sharing-lock-keys"></a>
#### ジョブクラス間でロックキーを共有する

デフォルトでは、`WithoutOverlapping`ミドルウェアは同じクラスのジョブの重複を防ぎます。したがって、2つの異なるジョブクラスが同じロックキーを使用していても、重複を防ぐことはできません。ただし、`shared`メソッドを使用してLaravelにジョブクラス間でキーを適用するよう指示できます。

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}

class ProviderIsUp
{
    // ...

    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

<a name="throttling-exceptions"></a>
### 例外のスロットリング

Laravelには、例外をスロットリングするための`Illuminate\Queue\Middleware\ThrottlesExceptions`ミドルウェアが含まれています。ジョブが指定された数の例外をスローすると、それ以降のジョブの実行は、指定された時間間隔が経過するまで遅延されます。このミドルウェアは、不安定なサードパーティサービスとやり取りするジョブに特に便利です。

例えば、キューに入れられたジョブがサードパーティAPIとやり取りし始めて例外をスローするとします。例外をスロットリングするには、ジョブの`middleware`メソッドから`ThrottlesExceptions`ミドルウェアを返します。通常、このミドルウェアは[時間ベースの試行](#time-based-attempts)を実装するジョブと組み合わせる必要があります。

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * ジョブが通過する必要があるミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * ジョブがタイムアウトする時間を決定する。
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(5);
}
```

ミドルウェアが受け入れる最初のコンストラクタ引数は、ジョブがスロットルされるまでに投げることができる例外の数であり、2番目のコンストラクタ引数は、ジョブがスロットルされた後に再試行されるまでに経過する秒数です。上記のコード例では、ジョブが5分以内に10回例外を投げた場合、ジョブを再試行する前に5分待ちます。

ジョブが例外を投げたが、例外のしきい値にまだ達していない場合、通常はジョブがすぐに再試行されます。しかし、ミドルウェアをジョブにアタッチする際に`backoff`メソッドを呼び出すことで、そのようなジョブが遅延する分数を指定できます：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

内部的には、このミドルウェアはLaravelのキャッシュシステムを使用してレート制限を実装し、ジョブのクラス名がキャッシュの「キー」として利用されます。ミドルウェアをジョブにアタッチする際に`by`メソッドを呼び出すことで、このキーを上書きできます。これは、複数のジョブが同じサードパーティサービスとやり取りしており、共通のスロットリング「バケット」を共有したい場合に便利です：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

デフォルトでは、このミドルウェアはすべての例外をスロットルします。ミドルウェアをジョブにアタッチする際に`when`メソッドを呼び出すことで、この動作を変更できます。例外は、`when`メソッドに提供されたクロージャが`true`を返す場合にのみスロットルされます：

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

スロットルされた例外をアプリケーションの例外ハンドラに報告したい場合、ミドルウェアをジョブにアタッチする際に`report`メソッドを呼び出すことでそれを行うことができます。オプションで、`report`メソッドにクロージャを提供し、指定されたクロージャが`true`を返す場合にのみ例外が報告されるようにすることもできます：

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->report(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

> NOTE:  
> Redisを使用している場合、`Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis`ミドルウェアを使用できます。これはRedisに最適化されており、基本的な例外スロットリングミドルウェアよりも効率的です。

<a name="skipping-jobs"></a>
### ジョブのスキップ

`Skip`ミドルウェアを使用すると、ジョブのロジックを変更することなく、ジョブをスキップ/削除するように指定できます。`Skip::when`メソッドは、指定された条件が`true`と評価された場合にジョブを削除し、`Skip::unless`メソッドは、条件が`false`と評価された場合にジョブを削除します：

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 */
public function middleware(): array
{
    return [
        Skip::when($someCondition),
    ];
}
```

`when`および`unless`メソッドに`Closure`を渡して、より複雑な条件評価を行うこともできます：

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * ジョブが通過すべきミドルウェアを取得する。
 */
public function middleware(): array
{
    return [
        Skip::when(function (): bool {
            return $this->shouldSkip();
        }),
    ];
}
```

<a name="dispatching-jobs"></a>
## ジョブのディスパッチ

ジョブクラスを作成したら、ジョブ自体の`dispatch`メソッドを使用してディスパッチできます。`dispatch`メソッドに渡された引数は、ジョブのコンストラクタに渡されます：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 新しいポッドキャストを保存する。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast);

        return redirect('/podcasts');
    }
}
```

条件付きでジョブをディスパッチしたい場合は、`dispatchIf`および`dispatchUnless`メソッドを使用できます：

```php
ProcessPodcast::dispatchIf($accountActive, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

新しいLaravelアプリケーションでは、`sync`ドライバがデフォルトのキュードライバです。このドライバは、現在のリクエストのフォアグラウンドでジョブを同期的に実行します。これは、ローカル開発中に便利です。ジョブをバックグラウンド処理のために実際にキューに入れたい場合は、アプリケーションの`config/queue.php`設定ファイル内で異なるキュードライバを指定できます。

<a name="delayed-dispatching"></a>
### 遅延ディスパッチ

ジョブがキューワーカーによってすぐに処理されるべきでない場合、ディスパッチ時に`delay`メソッドを使用して指定できます。たとえば、ジョブがディスパッチされてから10分後に処理可能になるように指定します：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 新しいポッドキャストを保存する。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));

        return redirect('/podcasts');
    }
}
```

いくつかのケースでは、ジョブにデフォルトの遅延が設定されている場合があります。この遅延をバイパスしてジョブを即時処理のためにディスパッチしたい場合は、`withoutDelay`メソッドを使用できます：

```php
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

> WARNING:  
> Amazon SQSキューサービスの最大遅延時間は15分です。

<a name="dispatching-after-the-response-is-sent-to-browser"></a>
#### レスポンスがブラウザに送信された後にディスパッチ

あるいは、`dispatchAfterResponse`メソッドを使用して、HTTPレスポンスがユーザーのブラウザに送信された後にジョブをディスパッチすることもできます。これにより、キューに入れられたジョブがまだ実行中であっても、ユーザーはアプリケーションの使用を開始できます。これは通常、約1秒かかるジョブにのみ使用されるべきです。例えば、メールの送信などです。これらは現在のHTTPリクエスト内で処理されるため、この方法でディスパッチされたジョブは、キューワーカーが実行されている必要はありません：

```php
use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

また、`dispatch`ヘルパーにクロージャを渡し、`afterResponse`メソッドをチェーンして、HTTPレスポンスがブラウザに送信された後にクロージャを実行することもできます：

```php
use App\Mail\WelcomeMessage;
use Illuminate\Support\Facades\Mail;

dispatch(function () {
    Mail::to('taylor@example.com')->send(new WelcomeMessage);
})->afterResponse();
```

<a name="synchronous-dispatching"></a>
### 同期ディスパッチ

ジョブを即座に（同期的に）ディスパッチしたい場合は、`dispatchSync`メソッドを使用できます。このメソッドを使用すると、ジョブはキューに入れられず、現在のプロセス内で即座に実行されます：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 新しいポッドキャストを保存する。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatchSync($podcast);

        return redirect('/podcasts');
    }
}
```

<a name="jobs-and-database-transactions"></a>
### ジョブとデータベーストランザクション

データベーストランザクション内でジョブをディスパッチすることは完全に問題ありませんが、ジョブが実際に成功することができるかどうかに特別な注意を払う必要があります。トランザクション内でジョブをディスパッチすると、親トランザクションがコミットされる前にジョブがワーカーによって処理される可能性があります。この場合、データベーストランザクション中に行ったモデルやデータベースレコードの更新が、まだデータベースに反映されていない可能性があります。さらに、トランザクション中に作成されたモデルやデータベースレコードは、データベースに存在しない可能性があります。

幸いなことに、Laravelはこの問題を回避するためのいくつかの方法を提供しています。まず、キュー接続の設定配列で`after_commit`接続オプションを設定できます：

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],
```

`after_commit`オプションが`true`の場合、データベーストランザクション内でジョブをディスパッチできます。ただし、Laravelはジョブを実際にディスパッチする前に、開いている親データベーストランザクションがコミットされるのを待ちます。もちろん、現在開いているデータベーストランザクションがない場合、ジョブはすぐにディスパッチされます。

トランザクション中に例外が発生してトランザクションがロールバックされた場合、そのトランザクション中にディスパッチされたジョブは破棄されます。

> NOTE:  
> `after_commit`設定オプションを`true`に設定すると、キューに入れられたイベントリスナー、メール、通知、およびブロードキャストイベントも、すべての開いているデータベーストランザクションがコミットされた後にディスパッチされます。

<a name="specifying-commit-dispatch-behavior-inline"></a>
#### インラインでのコミットディスパッチ動作の指定

`after_commit`キュー接続設定オプションを`true`に設定しない場合でも、特定のジョブがすべての開いているデータベーストランザクションがコミットされた後にディスパッチされるように指定できます。これを実現するには、ディスパッチ操作に`afterCommit`メソッドをチェーンします：

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

同様に、`after_commit`設定オプションが`true`に設定されている場合、特定のジョブが開いているデータベーストランザクションのコミットを待たずにすぐにディスパッチされるように指定できます：

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

<a name="job-chaining"></a>
### ジョブの連鎖

ジョブの連鎖により、プライマリジョブが正常に実行された後に順番に実行されるキュージョブのリストを指定できます。シーケンス内の1つのジョブが失敗した場合、残りのジョブは実行されません。キュージョブの連鎖を実行するには、`Bus`ファサードが提供する`chain`メソッドを使用できます。Laravelのコマンドバスは、キュージョブのディスパッチが構築される低レベルのコンポーネントです：

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

ジョブクラスインスタンスの連鎖に加えて、クロージャを連鎖することもできます：

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();
```

> WARNING:  
> ジョブ内で`$this->delete()`メソッドを使用してジョブを削除しても、連鎖されたジョブの処理は妨げられません。連鎖は、連鎖内のジョブが失敗した場合にのみ実行を停止します。

<a name="chain-connection-queue"></a>
#### 連鎖接続とキュー

連鎖されたジョブに使用する接続とキューを指定したい場合、`onConnection`と`onQueue`メソッドを使用できます。これらのメソッドは、キュージョブが別の接続/キューに明示的に割り当てられていない限り、使用されるキュー接続とキュー名を指定します：

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

<a name="adding-jobs-to-the-chain"></a>
#### 連鎖へのジョブの追加

連鎖内の別のジョブから既存のジョブ連鎖にジョブを追加または前に追加する必要がある場合があります。これは、`prependToChain`と`appendToChain`メソッドを使用して実現できます：

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    // 現在の連鎖に前に追加し、現在のジョブの直後にジョブを実行...
    $this->prependToChain(new TranscribePodcast);

    // 現在の連鎖に追加し、連鎖の最後にジョブを実行...
    $this->appendToChain(new TranscribePodcast);
}
```

<a name="chain-failures"></a>
#### 連鎖の失敗

ジョブの連鎖時に、連鎖内のジョブが失敗した場合に呼び出されるクロージャを指定するために`catch`メソッドを使用できます。指定されたコールバックは、ジョブの失敗を引き起こした`Throwable`インスタンスを受け取ります：

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // 連鎖内のジョブが失敗しました...
})->dispatch();
```

> WARNING:  
> 連鎖コールバックはシリアライズされ、後でLaravelキューによって実行されるため、連鎖コールバック内で`$this`変数を使用しないでください。

<a name="customizing-the-queue-and-connection"></a>
### キューと接続のカスタマイズ

<a name="dispatching-to-a-particular-queue"></a>
#### 特定のキューへのディスパッチ

異なるキューにジョブをプッシュすることで、キュージョブを「分類」し、どのキューにどれだけのワーカーを割り当てるかを優先順位付けできます。これにより、キュー設定ファイルで定義された異なるキュー「接続」にジョブをプッシュするのではなく、単一の接続内の特定のキューにジョブをプッシュします。キューを指定するには、ジョブをディスパッチする際に`onQueue`メソッドを使用します：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');

        return redirect('/podcasts');
    }
}
```

あるいは、ジョブのコンストラクタ内で`onQueue`メソッドを呼び出して、ジョブのキューを指定することもできます：

```php
<?php

namespace App\Jobs;

 use Illuminate\Contracts\Queue\ShouldQueue;
 use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

<a name="dispatching-to-a-particular-connection"></a>
#### 特定の接続へのディスパッチ

アプリケーションが複数のキュー接続と対話する場合、`onConnection`メソッドを使用してジョブをプッシュする接続を指定できます：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');

        return redirect('/podcasts');
    }
}
```

`onConnection`と`onQueue`メソッドを一緒にチェーンして、ジョブの接続とキューを指定できます：

```php
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

あるいは、ジョブのコンストラクタ内で`onConnection`メソッドを呼び出して、ジョブの接続を指定することもできます：

```php
<?php

namespace App\Jobs;

 use Illuminate\Contracts\Queue\ShouldQueue;
 use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

<a name="max-job-attempts-and-timeout"></a>
### 最大ジョブ試行回数/タイムアウト値の指定

<a name="max-attempts"></a>
#### 最大試行回数

キューに入れられたジョブの1つがエラーに遭遇した場合、無期限に再試行し続けることは望ましくないでしょう。そのため、Laravelはジョブを試行できる回数や時間を指定するためのさまざまな方法を提供しています。

ジョブが試行できる最大回数を指定する1つの方法は、Artisanコマンドラインの`--tries`スイッチを使用することです。これは、処理されるジョブが試行回数を指定していない限り、ワーカーによって処理されるすべてのジョブに適用されます：

```shell
php artisan queue:work --tries=3
```

ジョブが最大試行回数を超えると、「失敗した」ジョブと見なされます。失敗したジョブの処理についての詳細は、[失敗したジョブのドキュメント](#dealing-with-failed-jobs)を参照してください。`queue:work`コマンドに`--tries=0`が指定されている場合、ジョブは無期限に再試行されます。

ジョブクラス自体にジョブが試行できる最大回数を定義することで、より詳細なアプローチを取ることができます。ジョブに最大試行回数が指定されている場合、コマンドラインで指定された`--tries`の値よりも優先されます：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

特定のジョブの最大試行回数を動的に制御する必要がある場合は、ジョブに`tries`メソッドを定義できます：

```php
/**
 * Determine number of times the job may be attempted.
 */
public function tries(): int
{
    return 5;
}
```

<a name="time-based-attempts"></a>
#### 時間ベースの試行

ジョブが失敗するまでに何回試行できるかを定義する代わりに、ジョブが試行されなくなる時間を定義することもできます。これにより、指定した期間内でジョブを何度でも試行できます。ジョブが試行されなくなる時間を定義するには、ジョブクラスに`retryUntil`メソッドを追加します。このメソッドは`DateTime`インスタンスを返す必要があります。

```php
use DateTime;

/**
 * ジョブがタイムアウトする時間を決定します。
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(10);
}
```

> NOTE:  
> [キューイベントリスナー](events.md#queued-event-listeners)に`tries`プロパティまたは`retryUntil`メソッドを定義することもできます。

<a name="max-exceptions"></a>
#### 最大例外数

ジョブを何度も試行させたいが、指定した数の未処理例外が発生した場合に失敗させたい場合があります（`release`メソッドによって直接リリースされるのではなく）。これを実現するには、ジョブクラスに`maxExceptions`プロパティを定義します。

```php
<?php

namespace App\Jobs;

use Illuminate\Support\Facades\Redis;

class ProcessPodcast implements ShouldQueue
{
    /**
     * ジョブが試行できる回数。
     *
     * @var int
     */
    public $tries = 25;

    /**
     * 失敗するまでに許容する未処理例外の最大数。
     *
     * @var int
     */
    public $maxExceptions = 3;

    /**
     * ジョブを実行します。
     */
    public function handle(): void
    {
        Redis::throttle('key')->allow(10)->every(60)->then(function () {
            // ロックが取得され、ポッドキャストを処理します...
        }, function () {
            // ロックが取得できませんでした...
            return $this->release(10);
        });
    }
}
```

この例では、アプリケーションがRedisロックを取得できない場合、ジョブは10秒間リリースされ、最大25回再試行されます。ただし、ジョブが3つの未処理例外をスローした場合、ジョブは失敗します。

<a name="timeout"></a>
#### タイムアウト

キューに入れられたジョブがどのくらいの時間かかるか、おおよその時間がわかっていることがよくあります。そのため、Laravelでは「タイムアウト」値を指定できます。デフォルトでは、タイムアウト値は60秒です。ジョブがタイムアウト値で指定された秒数より長く処理されると、ジョブを処理しているワーカーはエラーで終了します。通常、ワーカーは[サーバーで設定されたプロセスマネージャ](#supervisor-configuration)によって自動的に再起動されます。

ジョブが実行できる最大秒数は、Artisanコマンドラインで`--timeout`スイッチを使用して指定できます。

```shell
php artisan queue:work --timeout=30
```

ジョブがタイムアウトを繰り返し超えて最大試行回数を超えた場合、ジョブは失敗としてマークされます。

ジョブクラス自体でジョブが実行できる最大秒数を定義することもできます。ジョブでタイムアウトが指定されている場合、コマンドラインで指定されたタイムアウトよりも優先されます。

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * ジョブがタイムアウトするまでの秒数。
     *
     * @var int
     */
    public $timeout = 120;
}
```

時には、ソケットや外部HTTP接続などのIOブロッキングプロセスが指定したタイムアウトを無視することがあります。そのため、これらの機能を使用する場合は、常にそれらのAPIを使用してタイムアウトを指定する必要があります。例えば、Guzzleを使用する場合は、常に接続とリクエストのタイムアウト値を指定する必要があります。

> WARNING:  
> ジョブのタイムアウトを指定するには、`pcntl` PHP拡張機能がインストールされている必要があります。また、ジョブの「タイムアウト」値は、常に["retry after"](#job-expiration)値よりも小さくする必要があります。そうしないと、ジョブが実際に終了する前に再試行される可能性があります。

<a name="failing-on-timeout"></a>
#### タイムアウト時に失敗

タイムアウト時にジョブを[失敗](#dealing-with-failed-jobs)としてマークしたい場合は、ジョブクラスに`$failOnTimeout`プロパティを定義できます。

```php
/**
 * タイムアウト時にジョブを失敗としてマークするかどうかを示します。
 *
 * @var bool
 */
public $failOnTimeout = true;
```

<a name="error-handling"></a>
### エラー処理

ジョブの処理中に例外がスローされた場合、ジョブは自動的にキューに戻され、再試行されます。ジョブは、アプリケーションで許可されている最大試行回数に達するまで継続的にリリースされます。最大試行回数は、`queue:work` Artisanコマンドで使用される`--tries`スイッチによって定義されます。または、ジョブクラス自体で最大試行回数を定義することもできます。キューワーカーの実行に関する詳細情報は、[以下](#running-the-queue-worker)で見つけることができます。

<a name="manually-releasing-a-job"></a>
#### 手動でジョブをリリース

時には、ジョブを手動でキューに戻して、後で再試行したい場合があります。これは、`release`メソッドを呼び出すことで実現できます。

```php
/**
 * ジョブを実行します。
 */
public function handle(): void
{
    // ...

    $this->release();
}
```

デフォルトでは、`release`メソッドはジョブを即座にキューに戻します。ただし、整数または日付インスタンスを`release`メソッドに渡すことで、ジョブが処理されるまでの秒数を指定できます。

```php
$this->release(10);

$this->release(now()->addSeconds(10));
```

<a name="manually-failing-a-job"></a>
#### 手動でジョブを失敗

時には、ジョブを手動で「失敗」としてマークする必要があります。これを行うには、`fail`メソッドを呼び出します。

```php
/**
 * ジョブを実行します。
 */
public function handle(): void
{
    // ...

    $this->fail();
}
```

キャッチした例外のためにジョブを失敗としてマークしたい場合は、例外を`fail`メソッドに渡すことができます。また、便宜上、エラーメッセージを文字列で渡すこともでき、これは例外に変換されます。

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

> NOTE:  
> 失敗したジョブの詳細については、[失敗したジョブの処理に関するドキュメント](#dealing-with-failed-jobs)を確認してください。

<a name="job-batching"></a>
## ジョブのバッチ処理

Laravelのジョブバッチ処理機能を使用すると、ジョブのバッチを簡単に実行し、バッチのジョブが完了したときに何らかのアクションを実行できます。まず、ジョブバッチに関するメタ情報を格納するテーブルを構築するためのデータベースマイグレーションを作成する必要があります。このマイグレーションは、`make:queue-batches-table` Artisanコマンドを使用して生成できます。

```shell
php artisan make:queue-batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### バッチ可能なジョブの定義

バッチ可能なジョブを定義するには、通常の[キュー可能なジョブ](#creating-jobs)を作成する必要があります。ただし、ジョブクラスに`Illuminate\Bus\Batchable`トレイトを追加する必要があります。このトレイトは、ジョブが実行されている現在のバッチを取得するために使用できる`batch`メソッドへのアクセスを提供します。

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

    /**
     * ジョブを実行します。
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            // バッチがキャンセルされたかどうかを判断します...

            return;
        }

        // CSVファイルの一部をインポートします...
    }
}
```

<a name="dispatching-batches"></a>
### バッチのディスパッチ

ジョブのバッチをディスパッチするには、`Bus`ファサードの`batch`メソッドを使用する必要があります。もちろん、バッチ処理は主に完了コールバックと組み合わせて使用するのが便利です。そのため、`then`、`catch`、`finally`メソッドを使用してバッチの完了コールバックを定義できます。これらの各コールバックは、呼び出されたときに`Illuminate\Bus\Batch`インスタンスを受け取ります。この例では、CSVファイルから行を処理するジョブのバッチをキューに入れることを想像しています。

```php
use App\Jobs\ImportCsv;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsv(1, 100),
    new ImportCsv(101, 200),
    new ImportCsv(201, 300),
    new ImportCsv(301, 400),
    new ImportCsv(401, 500),
])->before(function (Batch $batch) {
    // バッチが作成されましたが、ジョブはまだ追加されていません...
})->progress(function (Batch $batch) {
    // 単一のジョブが正常に完了しました...
})->then(function (Batch $batch) {
    // すべてのジョブが正常に完了しました...
})->catch(function (Batch $batch, Throwable $e) {
    // 最初のバッチジョブの失敗が検出されました...
})->finally(function (Batch $batch) {
    // バッチの実行が完了しました...
})->dispatch();

return $batch->id;
```

バッチのIDは、`$batch->id`プロパティを介してアクセスできます。これは、バッチがディスパッチされた後に[Laravelコマンドバスをクエリ](#inspecting-batches)してバッチに関する情報を取得するために使用できます。

> WARNING:  
> バッチコールバックはシリアライズされ、後でLaravelキューによって実行されるため、コールバック内で`$this`変数を使用しないでください。また、バッチジョブはデータベーストランザクション内でラップされるため、暗黙的なコミットをトリガーするデータベースステートメントはジョブ内で実行しないでください。

<a name="naming-batches"></a>
#### バッチの命名

特定のバッチを識別しやすくするために、バッチに名前を付けることができます。これにより、バッチの進行状況を監視する際に、バッチを識別しやすくなります。

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // すべてのジョブが正常に完了しました...
})->name('Import CSV')->dispatch();
```

Laravel HorizonやLaravel Telescopeなどのツールは、バッチに名前が付けられている場合、よりユーザーフレンドリーなデバッグ情報を提供することがあります。バッチに任意の名前を割り当てるには、バッチを定義する際に`name`メソッドを呼び出すことができます。

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // すべてのジョブが正常に完了しました...
    })->name('CSVのインポート')->dispatch();

<a name="batch-connection-queue"></a>
#### バッチの接続とキュー

バッチされたジョブに使用される接続とキューを指定したい場合は、`onConnection`メソッドと`onQueue`メソッドを使用できます。すべてのバッチされたジョブは同じ接続とキュー内で実行される必要があります。

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // すべてのジョブが正常に完了しました...
    })->onConnection('redis')->onQueue('imports')->dispatch();

<a name="chains-and-batches"></a>
### チェーンとバッチ

バッチ内に[チェーンされたジョブ](#job-chaining)のセットを定義するには、チェーンされたジョブを配列内に配置します。たとえば、2つのジョブチェーンを並行して実行し、両方のジョブチェーンが処理を完了したときにコールバックを実行できます。

    use App\Jobs\ReleasePodcast;
    use App\Jobs\SendPodcastReleaseNotification;
    use Illuminate\Bus\Batch;
    use Illuminate\Support\Facades\Bus;

    Bus::batch([
        [
            new ReleasePodcast(1),
            new SendPodcastReleaseNotification(1),
        ],
        [
            new ReleasePodcast(2),
            new SendPodcastReleaseNotification(2),
        ],
    ])->then(function (Batch $batch) {
        // ...
    })->dispatch();

逆に、[チェーン](#job-chaining)内でバッチされたジョブを実行するには、チェーン内にバッチを定義します。たとえば、まず複数のポッドキャストをリリースするためのバッチされたジョブを実行し、次にリリース通知を送信するためのバッチされたジョブを実行できます。

    use App\Jobs\FlushPodcastCache;
    use App\Jobs\ReleasePodcast;
    use App\Jobs\SendPodcastReleaseNotification;
    use Illuminate\Support\Facades\Bus;

    Bus::chain([
        new FlushPodcastCache,
        Bus::batch([
            new ReleasePodcast(1),
            new ReleasePodcast(2),
        ]),
        Bus::batch([
            new SendPodcastReleaseNotification(1),
            new SendPodcastReleaseNotification(2),
        ]),
    ])->dispatch();

<a name="adding-jobs-to-batches"></a>
### バッチへのジョブの追加

バッチされたジョブ内からバッチに追加のジョブを追加すると便利な場合があります。このパターンは、数千のジョブをバッチ処理する必要があり、Webリクエスト中にディスパッチするには時間がかかりすぎる場合に役立ちます。その代わりに、バッチにさらにジョブを追加する「ローダー」ジョブの初期バッチをディスパッチすることができます。

    $batch = Bus::batch([
        new LoadImportBatch,
        new LoadImportBatch,
        new LoadImportBatch,
    ])->then(function (Batch $batch) {
        // すべてのジョブが正常に完了しました...
    })->name('連絡先のインポート')->dispatch();

この例では、`LoadImportBatch`ジョブを使用してバッチに追加のジョブを追加します。これを実現するには、ジョブの`batch`メソッドを介してアクセスできるバッチインスタンスの`add`メソッドを使用できます。

    use App\Jobs\ImportContacts;
    use Illuminate\Support\Collection;

    /**
     * ジョブの実行
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            return;
        }

        $this->batch()->add(Collection::times(1000, function () {
            return new ImportContacts;
        }));
    }

> WARNING:  
> 同じバッチに属するジョブ内からのみ、バッチにジョブを追加できます。

<a name="inspecting-batches"></a>
### バッチの検査

バッチ完了コールバックに提供される`Illuminate\Bus\Batch`インスタンスには、指定されたバッチされたジョブと対話したり検査したりするためのさまざまなプロパティとメソッドがあります。

    // バッチのUUID...
    $batch->id;

    // バッチの名前（該当する場合）...
    $batch->name;

    // バッチに割り当てられたジョブの数...
    $batch->totalJobs;

    // キューによってまだ処理されていないジョブの数...
    $batch->pendingJobs;

    // 失敗したジョブの数...
    $batch->failedJobs;

    // これまでに処理されたジョブの数...
    $batch->processedJobs();

    // バッチの完了率（0-100）...
    $batch->progress();

    // バッチが実行を完了したかどうかを示す...
    $batch->finished();

    // バッチの実行をキャンセルする...
    $batch->cancel();

    // バッチがキャンセルされたかどうかを示す...
    $batch->cancelled();

<a name="returning-batches-from-routes"></a>
#### ルートからのバッチの返却

すべての`Illuminate\Bus\Batch`インスタンスはJSONシリアライズ可能であり、アプリケーションのルートの1つから直接返すことで、バッチに関する情報（完了進捗を含む）を含むJSONペイロードを取得できます。これにより、アプリケーションのUIにバッチの完了進捗に関する情報を表示するのが便利になります。

バッチをそのIDで取得するには、`Bus`ファサードの`findBatch`メソッドを使用できます。

    use Illuminate\Support\Facades\Bus;
    use Illuminate\Support\Facades\Route;

    Route::get('/batch/{batchId}', function (string $batchId) {
        return Bus::findBatch($batchId);
    });

<a name="cancelling-batches"></a>
### バッチのキャンセル

場合によっては、特定のバッチの実行をキャンセルする必要があります。これは、`Illuminate\Bus\Batch`インスタンスの`cancel`メソッドを呼び出すことで実現できます。

    /**
     * ジョブの実行
     */
    public function handle(): void
    {
        if ($this->user->exceedsImportLimit()) {
            return $this->batch()->cancel();
        }

        if ($this->batch()->cancelled()) {
            return;
        }
    }

前の例で見たように、バッチされたジョブは通常、実行を継続する前に対応するバッチがキャンセルされたかどうかを判断する必要があります。ただし、便宜上、代わりに`SkipIfBatchCancelled`[ミドルウェア](#job-middleware)をジョブに割り当てることができます。その名前が示すように、このミドルウェアはLaravelに対応するバッチがキャンセルされた場合にジョブを処理しないように指示します。

    use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

    /**
     * ジョブが通過するミドルウェアを取得
     */
    public function middleware(): array
    {
        return [new SkipIfBatchCancelled];
    }

<a name="batch-failures"></a>
### バッチの失敗

バッチされたジョブが失敗すると、`catch`コールバック（割り当てられている場合）が呼び出されます。このコールバックは、バッチ内で最初に失敗したジョブに対してのみ呼び出されます。

<a name="allowing-failures"></a>
#### 失敗の許可

バッチ内のジョブが失敗すると、Laravelは自動的にバッチを「キャンセル済み」としてマークします。この動作を無効にして、ジョブの失敗が自動的にバッチをキャンセル済みとしてマークしないようにすることができます。これは、バッチをディスパッチする際に`allowFailures`メソッドを呼び出すことで実現できます。

    $batch = Bus::batch([
        // ...
    ])->then(function (Batch $batch) {
        // すべてのジョブが正常に完了しました...
    })->allowFailures()->dispatch();

<a name="retrying-failed-batch-jobs"></a>
#### 失敗したバッチジョブの再試行

便宜上、Laravelは指定されたバッチのすべての失敗したジョブを簡単に再試行できる`queue:retry-batch` Artisanコマンドを提供しています。`queue:retry-batch`コマンドは、再試行する失敗したジョブのバッチのUUIDを受け取ります。

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### バッチの整理

整理を行わない場合、`job_batches`テーブルは非常に迅速にレコードを蓄積する可能性があります。これを軽減するために、`queue:prune-batches` Artisanコマンドを毎日実行するように[スケジュール](scheduling.md)する必要があります。

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('queue:prune-batches')->daily();

デフォルトでは、24時間以上経過したすべての完了したバッチが整理されます。コマンドを呼び出す際に`hours`オプションを使用して、バッチデータを保持する期間を決定できます。たとえば、次のコマンドは48時間以上前に完了したすべてのバッチを削除します。

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('queue:prune-batches --hours=48')->daily();

場合によっては、`jobs_batches`テーブルが正常に完了しなかったバッチのレコードを蓄積することがあります。たとえば、ジョブが失敗し、そのジョブが正常に再試行されなかったバッチなどです。`queue:prune-batches`コマンドに`unfinished`オプションを使用して、これらの未完了のバッチレコードを整理するように指示できます。

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('queue:prune-batches --hours=48 --unfinished=72')->daily();

同様に、`jobs_batches`テーブルはキャンセルされたバッチのレコードを蓄積することもあります。`queue:prune-batches`コマンドに`cancelled`オプションを使用して、これらのキャンセルされたバッチレコードを整理するように指示できます。

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('queue:prune-batches --hours=48 --cancelled=72')->daily();

<a name="storing-batches-in-dynamodb"></a>
### DynamoDBへのバッチの保存

Laravelは、バッチのメタ情報をリレーショナルデータベースの代わりに[DynamoDB](https://aws.amazon.com/dynamodb)に保存するためのサポートも提供しています。ただし、すべてのバッチレコードを保存するためのDynamoDBテーブルを手動で作成する必要があります。

通常、このテーブルは`job_batches`という名前になりますが、アプリケーションの`queue`設定ファイル内の`queue.batching.table`設定値に基づいてテーブルに名前を付ける必要があります。

<a name="dynamodb-batch-table-configuration"></a>
#### DynamoDBバッチテーブルの設定

`job_batches` テーブルは、`application` という名前の文字列の主パーティションキーと、`id` という名前の文字列の主ソートキーを持つべきです。キーの `application` 部分には、アプリケーションの `app` 設定ファイル内の `name` 設定値で定義されたアプリケーション名が含まれます。アプリケーション名が DynamoDB テーブルのキーの一部であるため、複数の Laravel アプリケーションのジョブバッチを同じテーブルに保存することができます。

さらに、[自動ジョブバッチのプルーニング](#pruning-batches-in-dynamodb)を利用したい場合、テーブルに `ttl` 属性を定義することができます。

<a name="dynamodb-configuration"></a>
#### DynamoDB の設定

次に、Laravel アプリケーションが Amazon DynamoDB と通信できるように、AWS SDK をインストールします。

```shell
composer require aws/aws-sdk-php
```

そして、`queue.batching.driver` 設定オプションの値を `dynamodb` に設定します。さらに、`batching` 設定配列内で `key`、`secret`、`region` 設定オプションを定義する必要があります。これらのオプションは AWS への認証に使用されます。`dynamodb` ドライバを使用する場合、`queue.batching.database` 設定オプションは不要です。

```php
'batching' => [
    'driver' => env('QUEUE_BATCHING_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
],
```

<a name="pruning-batches-in-dynamodb"></a>
#### DynamoDB でのバッチのプルーニング

[DynamoDB](https://aws.amazon.com/dynamodb) を使用してジョブバッチ情報を保存する場合、リレーショナルデータベースに保存されたバッチをプルーニングするために使用される通常のプルーニングコマンドは機能しません。代わりに、[DynamoDB のネイティブ TTL 機能](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)を利用して、古いバッチのレコードを自動的に削除することができます。

DynamoDB テーブルに `ttl` 属性を定義した場合、Laravel にバッチレコードのプルーニング方法を指示するための設定パラメータを定義できます。`queue.batching.ttl_attribute` 設定値は TTL を保持する属性の名前を定義し、`queue.batching.ttl` 設定値はレコードが最後に更新されてからの秒数を定義し、その後バッチレコードを DynamoDB テーブルから削除できるようにします。

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
    'ttl_attribute' => 'ttl',
    'ttl' => 60 * 60 * 24 * 7, // 7日間...
],
```

<a name="queueing-closures"></a>
## クロージャのキューイング

ジョブクラスをキューにディスパッチする代わりに、クロージャをディスパッチすることもできます。これは、現在のリクエストサイクル外で実行する必要がある簡単なタスクに最適です。クロージャをキューにディスパッチする場合、クロージャのコード内容は暗号化されて署名されるため、転送中に変更されることはありません。

```php
$podcast = App\Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

`catch` メソッドを使用して、キューに入れられたクロージャがすべての[設定された再試行回数](#max-job-attempts-and-timeout)を使い果たしても正常に完了しなかった場合に実行されるクロージャを提供できます。

```php
use Throwable;

dispatch(function () use ($podcast) {
    $podcast->publish();
})->catch(function (Throwable $e) {
    // このジョブは失敗しました...
});
```

> WARNING:  
> `catch` コールバックはシリアライズされ、後で Laravel キューによって実行されるため、`catch` コールバック内で `$this` 変数を使用しないでください。

<a name="running-the-queue-worker"></a>
## キューワーカーの実行

<a name="the-queue-work-command"></a>
### `queue:work` コマンド

Laravel には、キューワーカーを起動し、キューにプッシュされた新しいジョブを処理する Artisan コマンドが含まれています。`queue:work` Artisan コマンドを使用してワーカーを実行できます。`queue:work` コマンドが開始されると、手動で停止するか、ターミナルを閉じるまで実行し続けます。

```shell
php artisan queue:work
```

> NOTE:  
> `queue:work` プロセスをバックグラウンドで永続的に実行し続けるためには、[Supervisor](#supervisor-configuration) などのプロセスモニターを使用して、キューワーカーが停止しないようにする必要があります。

`queue:work` コマンドを呼び出す際に `-v` フラグを含めると、処理されたジョブの ID がコマンドの出力に含まれるようになります。

```shell
php artisan queue:work -v
```

キューワーカーは長時間実行されるプロセスであり、ブートされたアプリケーションの状態をメモリに保存します。その結果、起動後にコードベースの変更を検出しません。したがって、デプロイプロセス中に[キューワーカーを再起動](#queue-workers-and-deployment)することを忘れないでください。また、アプリケーションによって作成または変更された静的な状態は、ジョブ間で自動的にリセットされないことに注意してください。

あるいは、`queue:listen` コマンドを実行することもできます。`queue:listen` コマンドを使用する場合、更新されたコードをリロードしたり、アプリケーションの状態をリセットしたりするためにワーカーを手動で再起動する必要はありません。ただし、このコマンドは `queue:work` コマンドよりも大幅に効率が悪いです。

```shell
php artisan queue:listen
```

<a name="running-multiple-queue-workers"></a>
#### 複数のキューワーカーの実行

キューに複数のワーカーを割り当ててジョブを並行して処理するには、単純に複数の `queue:work` プロセスを開始するだけです。これは、ローカルではターミナルの複数のタブを使用して行うことができますし、本番環境ではプロセスマネージャの設定を使用して行うことができます。[Supervisor を使用する場合](#supervisor-configuration)、`numprocs` 設定値を使用できます。

<a name="specifying-the-connection-queue"></a>
#### 接続とキューの指定

ワーカーが使用するキュー接続を指定することもできます。`work` コマンドに渡される接続名は、`config/queue.php` 設定ファイルで定義された接続のいずれかに対応する必要があります。

```shell
php artisan queue:work redis
```

デフォルトでは、`queue:work` コマンドは指定された接続のデフォルトキューのジョブのみを処理します。ただし、特定の接続の特定のキューのみを処理するようにキューワーカーをさらにカスタマイズすることができます。たとえば、すべてのメールが `redis` キュー接続の `emails` キューで処理される場合、次のコマンドを発行して、そのキューのみを処理するワーカーを開始できます。

```shell
php artisan queue:work redis --queue=emails
```

<a name="processing-a-specified-number-of-jobs"></a>
#### 指定された数のジョブの処理

`--once` オプションを使用して、ワーカーにキューから単一のジョブのみを処理するように指示できます。

```shell
php artisan queue:work --once
```

`--max-jobs` オプションを使用して、ワーカーに指定された数のジョブを処理してから終了するように指示できます。このオプションは、[Supervisor](#supervisor-configuration) と組み合わせて使用すると便利です。これにより、指定された数のジョブを処理した後にワーカーが自動的に再起動され、蓄積されたメモリが解放されます。

```shell
php artisan queue:work --max-jobs=1000
```

<a name="processing-all-queued-jobs-then-exiting"></a>
#### すべてのキューイングされたジョブを処理してから終了

`--stop-when-empty` オプションを使用して、ワーカーにすべてのジョブを処理してから正常に終了するように指示できます。このオプションは、キューが空になった後にコンテナをシャットダウンする場合に、Docker コンテナ内で Laravel キューを処理する際に便利です。

```shell
php artisan queue:work --stop-when-empty
```

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### 指定された秒数のジョブの処理

`--max-time` オプションを使用して、ワーカーに指定された秒数のジョブを処理してから終了するように指示できます。このオプションは、[Supervisor](#supervisor-configuration) と組み合わせて使用すると便利です。これにより、指定された時間のジョブを処理した後にワーカーが自動的に再起動され、蓄積されたメモリが解放されます。

```shell
# 1時間ジョブを処理してから終了...
php artisan queue:work --max-time=3600
```

<a name="worker-sleep-duration"></a>
#### ワーカーのスリープ時間

キューにジョブがある場合、ワーカーはジョブ間の遅延なくジョブを処理し続けます。ただし、`sleep` オプションは、ジョブが利用できない場合にワーカーが「スリープ」する秒数を決定します。スリープ中、ワーカーは新しいジョブを処理しません。

```shell
php artisan queue:work --sleep=3
```

<a name="maintenance-mode-queues"></a>
#### メンテナンスモードとキュー

アプリケーションが[メンテナンスモード](configuration.md#maintenance-mode)にある間、キューイングされたジョブは処理されません。アプリケーションがメンテナンスモードを終了すると、ジョブは通常どおり処理されます。

メンテナンスモードが有効な場合でもキューワーカーがジョブを処理するように強制するには、`--force` オプションを使用できます。

```shell
php artisan queue:work --force
```

<a name="resource-considerations"></a>
#### リソースの考慮事項

デーモンキューワーカーは、各ジョブを処理する前にフレームワークを「再起動」しません。したがって、各ジョブが完了した後に重いリソースを解放する必要があります。たとえば、GD ライブラリを使用して画像操作を行う場合、画像の処理が完了した後に `imagedestroy` を使用してメモリを解放する必要があります。

<a name="queue-priorities"></a>
### キューの優先度

キューの処理順序を優先することがあります。たとえば、`config/queue.php` 設定ファイルで、`redis` 接続のデフォルト `queue` を `low` に設定することができます。ただし、次のように `high` 優先度キューにジョブをプッシュすることがあります。

```php
dispatch((new Job)->onQueue('high'));
```

`high`キューのすべてのジョブが処理されるまで、`low`キューのジョブに進まないようにするワーカーを開始するには、キュー名のカンマ区切りリストを`work`コマンドに渡します:

```shell
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### キューワーカーとデプロイ

キューワーカーは長時間実行されるプロセスであるため、コードの変更を再起動しない限り認識しません。したがって、キューワーカーを使用してアプリケーションをデプロイする最も簡単な方法は、デプロイプロセス中にワーカーを再起動することです。`queue:restart`コマンドを発行することで、すべてのワーカーを正常に再起動できます:

```shell
php artisan queue:restart
```

このコマンドは、現在処理中のジョブが完了した後にすべてのキューワーカーに正常に終了するよう指示するため、既存のジョブが失われることはありません。`queue:restart`コマンドが実行されるとキューワーカーは終了するため、[Supervisor](#supervisor-configuration)などのプロセスマネージャを実行して、キューワーカーを自動的に再起動する必要があります。

> NOTE:  
> キューは[キャッシュ](cache.md)を使用して再起動信号を保存するため、この機能を使用する前に、アプリケーションに対してキャッシュドライバが適切に設定されていることを確認する必要があります。

<a name="job-expirations-and-timeouts"></a>
### ジョブの有効期限とタイムアウト

<a name="job-expiration"></a>
#### ジョブの有効期限

`config/queue.php`設定ファイルでは、各キュー接続は`retry_after`オプションを定義します。このオプションは、キュー接続が処理中のジョブを再試行するまで待機する秒数を指定します。たとえば、`retry_after`の値が`90`に設定されている場合、ジョブは90秒間処理されていても解放または削除されない場合、キューに戻されます。通常、`retry_after`の値は、ジョブが合理的に完了するまでにかかる最大秒数に設定する必要があります。

> WARNING:  
> `retry_after`値を含まない唯一のキュー接続はAmazon SQSです。SQSは、AWSコンソール内で管理される[デフォルトの可視性タイムアウト](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)に基づいてジョブを再試行します。

<a name="worker-timeouts"></a>
#### ワーカーのタイムアウト

`queue:work` Artisanコマンドは、`--timeout`オプションを公開します。デフォルトでは、`--timeout`の値は60秒です。ジョブがタイムアウト値で指定された秒数より長く処理されている場合、ジョブを処理しているワーカーはエラーで終了します。通常、ワーカーはサーバー上で設定された[プロセスマネージャ](#supervisor-configuration)によって自動的に再起動されます:

```shell
php artisan queue:work --timeout=60
```

`retry_after`設定オプションと`--timeout` CLIオプションは異なりますが、ジョブが失われず、ジョブが1回だけ正常に処理されるように連携して動作します。

> WARNING:  
> `--timeout`値は、常に`retry_after`設定値より少なくとも数秒短くする必要があります。これにより、凍結したジョブを処理しているワーカーが、ジョブが再試行される前に常に終了するようになります。`--timeout`オプションが`retry_after`設定値より長い場合、ジョブが2回処理される可能性があります。

<a name="supervisor-configuration"></a>
## Supervisorの設定

本番環境では、`queue:work`プロセスを実行し続ける方法が必要です。`queue:work`プロセスは、ワーカータイムアウトを超えたり、`queue:restart`コマンドが実行されたりなど、さまざまな理由で停止する可能性があります。

このため、`queue:work`プロセスが終了したときにそれらを自動的に再起動できるプロセスモニタを設定する必要があります。さらに、プロセスモニタは、同時に実行したい`queue:work`プロセスの数を指定できます。Supervisorは、Linux環境で一般的に使用されるプロセスモニタであり、次のドキュメントでその設定方法について説明します。

<a name="installing-supervisor"></a>
#### Supervisorのインストール

SupervisorはLinuxオペレーティングシステムのプロセスモニタであり、`queue:work`プロセスが失敗した場合に自動的に再起動します。UbuntuにSupervisorをインストールするには、次のコマンドを使用できます:

```shell
sudo apt-get install supervisor
```

> NOTE:  
> Supervisorの設定と管理が難しいと感じる場合は、[Laravel Forge](https://forge.laravel.com)の使用を検討してください。これにより、本番環境のLaravelプロジェクトに対して自動的にSupervisorがインストールおよび設定されます。

<a name="configuring-supervisor"></a>
#### Supervisorの設定

Supervisorの設定ファイルは通常、`/etc/supervisor/conf.d`ディレクトリに保存されます。このディレクトリ内で、Supervisorにプロセスの監視方法を指示する設定ファイルをいくつでも作成できます。たとえば、`queue:work`プロセスを開始および監視する`laravel-worker.conf`ファイルを作成しましょう:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

この例では、`numprocs`ディレクティブはSupervisorに8つの`queue:work`プロセスを実行し、すべてを監視し、失敗した場合に自動的に再起動するよう指示します。設定の`command`ディレクティブを、希望するキュー接続とワーカーオプションを反映するように変更する必要があります。

> WARNING:  
> `stopwaitsecs`の値が、最も長い実行中のジョブが消費する秒数より大きいことを確認する必要があります。そうでない場合、Supervisorはジョブが処理を完了する前にジョブを強制終了する可能性があります。

<a name="starting-supervisor"></a>
#### Supervisorの起動

設定ファイルが作成されたら、次のコマンドを使用してSupervisorの設定を更新し、プロセスを開始できます:

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

Supervisorの詳細については、[Supervisorのドキュメント](http://supervisord.org/index.html)を参照してください。

<a name="dealing-with-failed-jobs"></a>
## 失敗したジョブの処理

時には、キューに入れられたジョブが失敗することがあります。心配しないでください。物事は常に計画通りに進むわけではありません！Laravelには、ジョブを再試行する最大回数を[指定する便利な方法](#max-job-attempts-and-timeout)が含まれています。非同期ジョブがこの試行回数を超えると、`failed_jobs`データベーステーブルに挿入されます。[同期ディスパッチされたジョブ](queues.md#synchronous-dispatching)は失敗してもこのテーブルに保存されず、例外はアプリケーションによって即座に処理されます。

`failed_jobs`テーブルを作成するためのマイグレーションは、通常、新しいLaravelアプリケーションに既に存在します。ただし、アプリケーションにこのテーブルのマイグレーションが含まれていない場合は、`make:queue-failed-table`コマンドを使用してマイグレーションを作成できます:

```shell
php artisan make:queue-failed-table

php artisan migrate
```

[キューワーカー](#running-the-queue-worker)プロセスを実行するとき、`queue:work`コマンドの`--tries`スイッチを使用して、ジョブを再試行する最大回数を指定できます。`--tries`オプションの値を指定しない場合、ジョブは1回だけ試行されるか、ジョブクラスの`$tries`プロパティで指定された回数だけ試行されます:

```shell
php artisan queue:work redis --tries=3
```

`--backoff`オプションを使用すると、例外が発生したジョブを再試行する前にLaravelが待機する秒数を指定できます。デフォルトでは、ジョブは即座にキューに戻されて再試行されます:

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

例外が発生したジョブを再試行する前にLaravelが待機する秒数をジョブごとに設定したい場合は、ジョブクラスに`backoff`プロパティを定義することでそれを行うことができます:

    /**
     * ジョブを再試行する前に待機する秒数。
     *
     * @var int
     */
    public $backoff = 3;

ジョブのバックオフ時間を決定するためのより複雑なロジックが必要な場合は、ジョブクラスに`backoff`メソッドを定義できます:

    /**
    * ジョブを再試行する前に待機する秒数を計算します。
    */
    public function backoff(): int
    {
        return 3;
    }

`backoff`メソッドからバックオフ値の配列を返すことで、「指数関数的」なバックオフを簡単に設定できます。この例では、最初の再試行の遅延は1秒、2回目の再試行の遅延は5秒、3回目の再試行の遅延は10秒、それ以降の再試行の遅延は10秒になります（もっと試行回数が残っている場合）:

    /**
    * ジョブを再試行する前に待機する秒数を計算します。
    *
    * @return array<int, int>
    */
    public function backoff(): array
    {
        return [1, 5, 10];
    }

<a name="cleaning-up-after-failed-jobs"></a>
### 失敗したジョブの後処理

特定のジョブが失敗した場合、ユーザーにアラートを送信したり、ジョブによって部分的に完了したアクションを元に戻したりすることができます。これを実現するには、ジョブクラスに`failed`メソッドを定義できます。ジョブが失敗した原因となった`Throwable`インスタンスは、`failed`メソッドに渡されます:

    <?php

    namespace App\Jobs;

    use App\Models\Podcast;
    use App\Services\AudioProcessor;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Queue\Queueable;
    use Throwable;

    class ProcessPodcast implements ShouldQueue
    {
        use Queueable;

        /**
         * 新しいジョブインスタンスを作成します。
         */
        public function __construct(
            public Podcast $podcast,
        ) {}

        /**
         * ジョブを実行します。
         */
        public function handle(AudioProcessor $processor): void
        {
            // アップロードされたポッドキャストを処理します...
        }

        /**
         * ジョブが失敗したときに処理します。
         */
        public function failed(Throwable $exception): void
        {
            // 失敗したジョブの後処理を行います...
        }
    }

```php
/**
 * ジョブの失敗を処理する。
 */
public function failed(?Throwable $exception): void
{
    // 失敗に関するユーザー通知など...
}
```

> WARNING:  
> `failed` メソッドが呼び出される前にジョブの新しいインスタンスがインスタンス化されます。したがって、`handle` メソッド内で発生した可能性のあるクラスプロパティの変更は失われます。

<a name="retrying-failed-jobs"></a>
### 失敗したジョブの再試行

`failed_jobs` データベーステーブルに挿入されたすべての失敗したジョブを表示するには、`queue:failed` Artisan コマンドを使用できます。

```shell
php artisan queue:failed
```

`queue:failed` コマンドは、ジョブ ID、接続、キュー、失敗時間、およびその他のジョブに関する情報を一覧表示します。ジョブ ID を使用して、失敗したジョブを再試行できます。たとえば、`ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece` という ID を持つ失敗したジョブを再試行するには、次のコマンドを発行します。

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

必要に応じて、コマンドに複数の ID を渡すことができます。

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

特定のキューのすべての失敗したジョブを再試行することもできます。

```shell
php artisan queue:retry --queue=name
```

すべての失敗したジョブを再試行するには、`queue:retry` コマンドを実行し、ID として `all` を渡します。

```shell
php artisan queue:retry all
```

失敗したジョブを削除したい場合は、`queue:forget` コマンドを使用できます。

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d
```

> NOTE:  
> [Horizon](horizon.md) を使用している場合、失敗したジョブを削除するには `queue:forget` コマンドの代わりに `horizon:forget` コマンドを使用する必要があります。

`failed_jobs` テーブルからすべての失敗したジョブを削除するには、`queue:flush` コマンドを使用できます。

```shell
php artisan queue:flush
```

<a name="ignoring-missing-models"></a>
### 存在しないモデルの無視

ジョブに Eloquent モデルを注入すると、モデルはキューに入れられる前に自動的にシリアライズされ、ジョブが処理されるときにデータベースから再取得されます。ただし、ジョブがワーカーによって処理されるのを待っている間にモデルが削除された場合、ジョブは `ModelNotFoundException` で失敗する可能性があります。

便宜上、存在しないモデルを持つジョブを自動的に削除するように選択できます。これを行うには、ジョブの `deleteWhenMissingModels` プロパティを `true` に設定します。このプロパティが `true` に設定されている場合、Laravel は例外を発生させることなくジョブを静かに破棄します。

```php
/**
 * モデルが存在しない場合にジョブを削除します。
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

<a name="pruning-failed-jobs"></a>
### 失敗したジョブの整理

アプリケーションの `failed_jobs` テーブル内のレコードを整理するには、`queue:prune-failed` Artisan コマンドを呼び出すことができます。

```shell
php artisan queue:prune-failed
```

デフォルトでは、24 時間以上前のすべての失敗したジョブレコードが整理されます。コマンドに `--hours` オプションを指定すると、最後の N 時間以内に挿入された失敗したジョブレコードのみが保持されます。たとえば、次のコマンドは 48 時間以上前に挿入されたすべての失敗したジョブレコードを削除します。

```shell
php artisan queue:prune-failed --hours=48
```

<a name="storing-failed-jobs-in-dynamodb"></a>
### 失敗したジョブを DynamoDB に保存する

Laravel は、失敗したジョブレコードをリレーショナルデータベーステーブルの代わりに [DynamoDB](https://aws.amazon.com/dynamodb) に保存するためのサポートも提供しています。ただし、失敗したジョブレコードをすべて保存するための DynamoDB テーブルを手動で作成する必要があります。通常、このテーブルは `failed_jobs` という名前になりますが、アプリケーションの `queue` 設定ファイル内の `queue.failed.table` 設定値に基づいてテーブルに名前を付ける必要があります。

`failed_jobs` テーブルには、`application` という名前の文字列の主パーティションキーと、`uuid` という名前の文字列の主ソートキーが必要です。`application` 部分のキーには、アプリケーションの `app` 設定ファイル内の `name` 設定値で定義されたアプリケーション名が含まれます。アプリケーション名は DynamoDB テーブルのキーの一部であるため、同じテーブルを使用して複数の Laravel アプリケーションの失敗したジョブを保存できます。

さらに、Laravel アプリケーションが Amazon DynamoDB と通信できるように、AWS SDK をインストールしてください。

```shell
composer require aws/aws-sdk-php
```

次に、`queue.failed.driver` 設定オプションの値を `dynamodb` に設定します。さらに、失敗したジョブ設定配列内に `key`、`secret`、および `region` 設定オプションを定義する必要があります。これらのオプションは AWS との認証に使用されます。`dynamodb` ドライバを使用する場合、`queue.failed.database` 設定オプションは不要です。

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

<a name="disabling-failed-job-storage"></a>
### 失敗したジョブの保存を無効にする

失敗したジョブを保存せずに破棄するように Laravel に指示するには、`queue.failed.driver` 設定オプションの値を `null` に設定します。通常、これは `QUEUE_FAILED_DRIVER` 環境変数を介して行うことができます。

```ini
QUEUE_FAILED_DRIVER=null
```

<a name="failed-job-events"></a>
### 失敗したジョブのイベント

ジョブが失敗したときに呼び出されるイベントリスナーを登録したい場合は、`Queue` ファサードの `failing` メソッドを使用できます。たとえば、Laravel に含まれる `AppServiceProvider` の `boot` メソッドからこのイベントにクロージャをアタッチすることができます。

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 任意のアプリケーションサービスを起動します。
     */
    public function boot(): void
    {
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

<a name="clearing-jobs-from-queues"></a>
## キューからジョブをクリアする

> NOTE:  
> [Horizon](horizon.md) を使用している場合、キューからジョブをクリアするには `queue:clear` コマンドの代わりに `horizon:clear` コマンドを使用する必要があります。

デフォルトの接続のデフォルトのキューからすべてのジョブを削除したい場合は、`queue:clear` Artisan コマンドを使用して行うことができます。

```shell
php artisan queue:clear
```

特定の接続とキューからジョブを削除するには、`connection` 引数と `queue` オプションを指定することもできます。

```shell
php artisan queue:clear redis --queue=emails
```

> WARNING:  
> キューからジョブをクリアする機能は、SQS、Redis、およびデータベースキュードライバでのみ利用可能です。さらに、SQS メッセージの削除プロセスには最大 60 秒かかるため、キューをクリアしてから 60 秒以内に SQS キューに送信されたジョブも削除される可能性があります。

<a name="monitoring-your-queues"></a>
## キューの監視

ジョブの急増を受け取ると、キューが圧倒され、ジョブの完了までの待ち時間が長くなる可能性があります。必要に応じて、キューのジョブ数が指定したしきい値を超えたときに Laravel が通知するように設定できます。

まず、`queue:monitor` コマンドを [毎分実行するようにスケジュール](scheduling.md) する必要があります。このコマンドは、監視したいキューの名前と、希望するジョブ数のしきい値を受け取ります。

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100
```

このコマンドをスケジュールするだけでは、キューが圧倒された状態を通知するアラートはトリガーされません。コマンドがしきい値を超えたジョブ数のキューを検出すると、`Illuminate\Queue\Events\QueueBusy` イベントがディスパッチされます。アプリケーションの `AppServiceProvider` 内でこのイベントをリッスンして、通知を開発チームに送信することができます。

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * 任意のアプリケーションサービスを起動します。
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
                ->notify(new QueueHasLongWaitTime(
                    $event->connection,
                    $event->queue,
                    $event->size
                ));
    });
}
```

<a name="testing"></a>
## テスト

ジョブをディスパッチするコードをテストする際に、Laravel にジョブ自体を実行させたくない場合があります。ジョブのコードは、ディスパッチするコードとは別に直接テストできるからです。もちろん、ジョブ自体をテストするには、ジョブインスタンスを作成し、テスト内で `handle` メソッドを直接呼び出すことができます。

`Queue` ファサードの `fake` メソッドを使用して、キューにジョブが実際にプッシュされないようにすることができます。`Queue` ファサードの `fake` メソッドを呼び出した後、アプリケーションがジョブをキューにプッシュしようとしたことをアサートできます。

===  "Pest"
```php
<?php

use App\Jobs\AnotherJob;
use App\Jobs\FinalJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;

test('orders can be shipped', function () {
    Queue::fake();

    // 注文の出荷を実行...

    // ジョブがプッシュされなかったことをアサート...
    Queue::assertNothingPushed();

    // 特定のキューにジョブがプッシュされたことをアサート...
    Queue::assertPushedOn('queue-name', ShipOrder::class);
```

```php
// ジョブが2回プッシュされたことをアサートする...
Queue::assertPushed(ShipOrder::class, 2);

// ジョブがプッシュされなかったことをアサートする...
Queue::assertNotPushed(AnotherJob::class);

// クロージャがキューにプッシュされたことをアサートする...
Queue::assertClosurePushed();

// プッシュされたジョブの総数をアサートする...
Queue::assertCount(3);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
use App\Jobs\FinalJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Queue::fake();

        // 注文の出荷を実行...

        // ジョブがプッシュされなかったことをアサートする...
        Queue::assertNothingPushed();

        // 特定のキューにジョブがプッシュされたことをアサートする...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // ジョブが2回プッシュされたことをアサートする...
        Queue::assertPushed(ShipOrder::class, 2);

        // ジョブがプッシュされなかったことをアサートする...
        Queue::assertNotPushed(AnotherJob::class);

        // クロージャがキューにプッシュされたことをアサートする...
        Queue::assertClosurePushed();

        // プッシュされたジョブの総数をアサートする...
        Queue::assertCount(3);
    }
}
```

`assertPushed` または `assertNotPushed` メソッドにクロージャを渡して、特定の「真偽テスト」をパスするジョブがプッシュされたことをアサートすることができます。指定された真偽テストをパスするジョブが少なくとも1つプッシュされた場合、アサーションは成功します。

    Queue::assertPushed(function (ShipOrder $job) use ($order) {
        return $job->order->id === $order->id;
    });

<a name="faking-a-subset-of-jobs"></a>
### ジョブのサブセットをフェイクする

通常のジョブの実行を許可しながら、特定のジョブのみをフェイクする必要がある場合は、`fake` メソッドにフェイクするジョブのクラス名を渡すことができます。

===  "Pest"
```php
test('orders can be shipped', function () {
    Queue::fake([
        ShipOrder::class,
    ]);

    // 注文の出荷を実行...

    // ジョブが2回プッシュされたことをアサートする...
    Queue::assertPushed(ShipOrder::class, 2);
});
```

===  "PHPUnit"
```php
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // 注文の出荷を実行...

    // ジョブが2回プッシュされたことをアサートする...
    Queue::assertPushed(ShipOrder::class, 2);
}
```

`except` メソッドを使用して、指定されたジョブを除くすべてのジョブをフェイクすることができます。

    Queue::fake()->except([
        ShipOrder::class,
    ]);

<a name="testing-job-chains"></a>
### ジョブチェーンのテスト

ジョブチェーンをテストするには、`Bus` ファサードのフェイク機能を利用する必要があります。`Bus` ファサードの `assertChained` メソッドを使用して、[ジョブチェーン](queues.md#job-chaining)がディスパッチされたことをアサートすることができます。`assertChained` メソッドは、チェーンされたジョブの配列を最初の引数として受け取ります。

    use App\Jobs\RecordShipment;
    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Support\Facades\Bus;

    Bus::fake();

    // ...

    Bus::assertChained([
        ShipOrder::class,
        RecordShipment::class,
        UpdateInventory::class
    ]);

上記の例でわかるように、チェーンされたジョブの配列は、ジョブのクラス名の配列である場合があります。ただし、実際のジョブインスタンスの配列を提供することもできます。その場合、Laravel はジョブインスタンスが同じクラスであり、アプリケーションによってディスパッチされたチェーンされたジョブと同じプロパティ値を持っていることを確認します。

    Bus::assertChained([
        new ShipOrder,
        new RecordShipment,
        new UpdateInventory,
    ]);

`assertDispatchedWithoutChain` メソッドを使用して、ジョブがチェーンなしでプッシュされたことをアサートすることができます。

    Bus::assertDispatchedWithoutChain(ShipOrder::class);

<a name="testing-chain-modifications"></a>
#### チェーンの変更のテスト

チェーンされたジョブが[既存のチェーンにジョブを追加または削除](#adding-jobs-to-the-chain)する場合、ジョブの `assertHasChain` メソッドを使用して、ジョブが期待される残りのチェーンを持っていることをアサートすることができます。

```php
$job = new ProcessPodcast;

$job->handle();

$job->assertHasChain([
    new TranscribePodcast,
    new OptimizePodcast,
    new ReleasePodcast,
]);
```

`assertDoesntHaveChain` メソッドを使用して、ジョブの残りのチェーンが空であることをアサートすることができます。

```php
$job->assertDoesntHaveChain();
```

<a name="testing-chained-batches"></a>
#### チェーンされたバッチのテスト

ジョブチェーンに[バッチが含まれている](#chains-and-batches)場合、チェーンされたバッチが期待どおりであることをアサートするために、チェーンアサーション内に `Bus::chainedBatch` 定義を挿入することができます。

    use App\Jobs\ShipOrder;
    use App\Jobs\UpdateInventory;
    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::assertChained([
        new ShipOrder,
        Bus::chainedBatch(function (PendingBatch $batch) {
            return $batch->jobs->count() === 3;
        }),
        new UpdateInventory,
    ]);

<a name="testing-job-batches"></a>
### ジョブバッチのテスト

`Bus` ファサードの `assertBatched` メソッドを使用して、[ジョブバッチ](queues.md#job-batching)がディスパッチされたことをアサートすることができます。`assertBatched` メソッドに渡されるクロージャは、バッチ内のジョブを検査するために使用できる `Illuminate\Bus\PendingBatch` のインスタンスを受け取ります。

    use Illuminate\Bus\PendingBatch;
    use Illuminate\Support\Facades\Bus;

    Bus::fake();

    // ...

    Bus::assertBatched(function (PendingBatch $batch) {
        return $batch->name == 'import-csv' &&
               $batch->jobs->count() === 10;
    });

`assertBatchCount` メソッドを使用して、指定された数のバッチがディスパッチされたことをアサートすることができます。

    Bus::assertBatchCount(3);

`assertNothingBatched` を使用して、バッチがディスパッチされなかったことをアサートすることができます。

    Bus::assertNothingBatched();

<a name="testing-job-batch-interaction"></a>
#### ジョブ / バッチの相互作用のテスト

さらに、個々のジョブがその基礎となるバッチとどのように相互作用するかをテストする必要がある場合があります。たとえば、ジョブがそのバッチのさらなる処理をキャンセルしたかどうかをテストする必要がある場合です。これを実現するには、`withFakeBatch` メソッドを介してジョブにフェイクバッチを割り当てる必要があります。`withFakeBatch` メソッドは、ジョブインスタンスとフェイクバッチを含むタプルを返します。

    [$job, $batch] = (new ShipOrder)->withFakeBatch();

    $job->handle();

    $this->assertTrue($batch->cancelled());
    $this->assertEmpty($batch->added);

<a name="testing-job-queue-interactions"></a>
### ジョブ / キューの相互作用のテスト

キューに入れられたジョブが[自分自身をキューに再リリースする](#manually-releasing-a-job)か、ジョブが自分自身を削除するかをテストする必要がある場合があります。これらのキューの相互作用をテストするには、ジョブをインスタンス化し、`withFakeQueueInteractions` メソッドを呼び出すことができます。

ジョブのキューの相互作用がフェイクされたら、ジョブの `handle` メソッドを呼び出すことができます。ジョブを呼び出した後、`assertReleased`、`assertDeleted`、`assertNotDeleted`、`assertFailed`、および `assertNotFailed` メソッドを使用して、ジョブのキューの相互作用に対してアサーションを行うことができます。

```php
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertNotDeleted();
$job->assertFailed();
$job->assertNotFailed();
```

<a name="job-events"></a>
## ジョブイベント

`Queue` [ファサード](facades.md)の `before` および `after` メソッドを使用して、キューに入れられたジョブが処理される前後に実行されるコールバックを指定できます。これらのコールバックは、追加のログを実行したり、ダッシュボードの統計を増やしたりするのに最適な機会です。通常、これらのメソッドは [サービスプロバイダ](providers.md) の `boot` メソッドから呼び出す必要があります。たとえば、Laravel に含まれている `AppServiceProvider` を使用できます。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Queue;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Queue\Events\JobProcessed;
    use Illuminate\Queue\Events\JobProcessing;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションのすべてのサービスを登録します。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * アプリケーションのすべてのサービスを起動します。
         */
        public function boot(): void
        {
            Queue::before(function (JobProcessing $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });

            Queue::after(function (JobProcessed $event) {
                // $event->connectionName
                // $event->job
                // $event->job->payload()
            });
        }
    }

`Queue` [ファサード](facades.md)の `looping` メソッドを使用して、ワーカーがキューからジョブを取得しようとする前に実行されるコールバックを指定できます。たとえば、以前に失敗したジョブによって残された未解決のトランザクションをロールバックするクロージャを登録できます。

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Queue;

    Queue::looping(function () {
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
