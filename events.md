# イベント

- [イントロダクション](#introduction)
- [イベントとリスナーの生成](#generating-events-and-listeners)
- [イベントとリスナーの登録](#registering-events-and-listeners)
    - [イベントの自動検出](#event-discovery)
    - [イベントの手動登録](#manually-registering-events)
    - [クロージャリスナー](#closure-listeners)
- [イベントの定義](#defining-events)
- [リスナーの定義](#defining-listeners)
- [キューイベントリスナー](#queued-event-listeners)
    - [キューとの手動操作](#manually-interacting-with-the-queue)
    - [キューイベントリスナーとデータベーストランザクション](#queued-event-listeners-and-database-transactions)
    - [失敗したジョブの処理](#handling-failed-jobs)
- [イベントのディスパッチ](#dispatching-events)
    - [データベーストランザクション後のイベントディスパッチ](#dispatching-events-after-database-transactions)
- [イベントサブスクライバー](#event-subscribers)
    - [イベントサブスクライバーの作成](#writing-event-subscribers)
    - [イベントサブスクライバーの登録](#registering-event-subscribers)
- [テスト](#testing)
    - [イベントの一部をフェイクする](#faking-a-subset-of-events)
    - [スコープ付きイベントフェイク](#scoped-event-fakes)

<a name="introduction"></a>
## イントロダクション

Laravelのイベントは、シンプルなオブザーバーパターンの実装を提供し、アプリケーション内で発生するさまざまなイベントを購読してリッスンできるようにします。イベントクラスは通常、`app/Events`ディレクトリに保存され、それらのリスナーは`app/Listeners`に保存されます。これらのディレクトリがアプリケーションに表示されない場合でも心配しないでください。Artisanコンソールコマンドを使用してイベントとリスナーを生成すると、自動的に作成されます。

イベントは、アプリケーションのさまざまな側面を切り離すための優れた方法です。単一のイベントは、互いに依存しない複数のリスナーを持つことができるためです。たとえば、注文が発送されるたびにSlack通知をユーザーに送信したい場合があります。注文処理コードをSlack通知コードに結合する代わりに、`App\Events\OrderShipped`イベントを発生させ、リスナーが受信してSlack通知を送信するようにできます。

<a name="generating-events-and-listeners"></a>
## イベントとリスナーの生成

イベントとリスナーを素早く生成するには、`make:event`と`make:listener`のArtisanコマンドを使用できます：

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

便宜上、追加の引数なしで`make:event`と`make:listener`のArtisanコマンドを呼び出すこともできます。そうすると、Laravelは自動的にクラス名を尋ね、リスナーを作成する際にリッスンすべきイベントを尋ねます：

```shell
php artisan make:event

php artisan make:listener
```

<a name="registering-events-and-listeners"></a>
## イベントとリスナーの登録

<a name="event-discovery"></a>
### イベントの自動検出

デフォルトでは、Laravelはアプリケーションの`Listeners`ディレクトリをスキャンすることで、イベントリスナーを自動的に見つけて登録します。Laravelは、`handle`または`__invoke`で始まるリスナークラスメソッドを見つけると、それらのメソッドをメソッドのシグネチャで型ヒントされたイベントのイベントリスナーとして登録します：

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * イベントを処理します。
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

リスナーを別のディレクトリに保存する場合や、複数のディレクトリ内に保存する場合は、アプリケーションの`bootstrap/app.php`ファイルで`withEvents`メソッドを使用して、Laravelにそれらのディレクトリをスキャンするように指示できます：

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

`event:list`コマンドを使用して、アプリケーション内に登録されているすべてのリスナーをリストアップできます：

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### 本番環境でのイベントの自動検出

アプリケーションの速度を向上させるために、`optimize`または`event:cache`のArtisanコマンドを使用して、アプリケーションのすべてのリスナーのマニフェストをキャッシュする必要があります。通常、このコマンドはアプリケーションの[デプロイプロセス](deployment.md#optimization)の一部として実行する必要があります。このマニフェストは、フレームワークがイベント登録プロセスを高速化するために使用されます。`event:clear`コマンドを使用してイベントキャッシュを破棄できます。

<a name="manually-registering-events"></a>
### イベントの手動登録

`Event`ファサードを使用して、アプリケーションの`AppServiceProvider`の`boot`メソッド内でイベントとそれに対応するリスナーを手動で登録できます：

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

`event:list`コマンドを使用して、アプリケーション内に登録されているすべてのリスナーをリストアップできます：

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### クロージャリスナー

通常、リスナーはクラスとして定義されますが、アプリケーションの`AppServiceProvider`の`boot`メソッド内でクロージャベースのイベントリスナーを手動で登録することもできます：

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

<a name="queuable-anonymous-event-listeners"></a>
#### キュー可能な匿名イベントリスナー

クロージャベースのイベントリスナーを登録する際に、リスナークロージャを`Illuminate\Events\queueable`関数でラップして、Laravelに[キュー](queues.md)を使用してリスナーを実行するように指示できます：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

キューされたジョブと同様に、`onConnection`、`onQueue`、`delay`メソッドを使用して、キューされたリスナーの実行をカスタマイズできます：

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));
```

匿名のキューされたリスナーが失敗した場合に処理する場合は、`queueable`リスナーを定義する際に`catch`メソッドにクロージャを提供できます。このクロージャは、イベントインスタンスとリスナーの失敗を引き起こした`Throwable`インスタンスを受け取ります：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // キューされたリスナーが失敗しました...
}));
```

<a name="wildcard-event-listeners"></a>
#### ワイルドカードイベントリスナー

`*`文字をワイルドカードパラメータとして使用してリスナーを登録することもできます。これにより、同じリスナーで複数のイベントをキャッチできます。ワイルドカードリスナーは、イベント名を最初の引数として受け取り、イベントデータの配列全体を2番目の引数として受け取ります：

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

<a name="defining-events"></a>
## イベントの定義

イベントクラスは基本的に、イベントに関連する情報を保持するデータコンテナです。たとえば、`App\Events\OrderShipped`イベントが[Eloquent ORM](eloquent.md)オブジェクトを受け取るとします：

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * 新しいイベントインスタンスを作成します。
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

ご覧のとおり、このイベントクラスにはロジックは含まれていません。購入された`App\Models\Order`インスタンスのコンテナです。イベントで使用される`SerializesModels`トレイトは、イベントオブジェクトがPHPの`serialize`関数を使用してシリアライズされる場合、Eloquentモデルを適切にシリアライズします。たとえば、[キューイベントリスナー](#queued-event-listeners)を使用する場合などです。

<a name="defining-listeners"></a>
## リスナーの定義

次に、例のイベントのリスナーを見てみましょう。イベントリスナーは、`handle`メソッドでイベントインスタンスを受け取ります。`make:listener`のArtisanコマンドは、`--event`オプションを指定して呼び出すと、適切なイベントクラスを自動的にインポートし、`handle`メソッドでイベントを型ヒントします。`handle`メソッド内で、イベントに応答するために必要なアクションを実行できます：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * イベントリスナーを作成します。
     */
    public function __construct() {}

    /**
     * イベントを処理します。
     */
    public function handle(OrderShipped $event): void
    {
        // $event->orderを使用して注文にアクセスします...
    }
}
```

> NOTE:  
> イベントリスナーは、コンストラクタで必要な依存関係を型ヒントすることもできます。すべてのイベントリスナーはLaravelの[サービスコンテナ](container.md)を介して解決されるため、依存関係は自動的に注入されます。

<a name="stopping-the-propagation-of-an-event"></a>
#### イベントの伝播を停止する

時には、イベントが他のリスナーに伝播されるのを止めたい場合があるかもしれません。その場合、リスナーの `handle` メソッドから `false` を返すことで実現できます。

<a name="queued-event-listeners"></a>
## キューイベントリスナー

リスナーがメールの送信やHTTPリクエストの実行などの遅いタスクを実行する場合、リスナーをキューに入れることが有益です。キューリスナーを使用する前に、[キューの設定](queues.md)を行い、サーバーまたはローカル開発環境でキューワーカーを起動しておく必要があります。

リスナーをキューに入れるように指定するには、リスナークラスに `ShouldQueue` インターフェースを追加します。`make:listener` Artisanコマンドで生成されたリスナーは、このインターフェースが既に現在の名前空間にインポートされているため、すぐに使用できます：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        // ...
    }

これで完了です！このリスナーによって処理されるイベントがディスパッチされると、リスナーはLaravelの[キューシステム](queues.md)を使用してイベントディスパッチャによって自動的にキューに入れられます。リスナーがキューによって実行されたときに例外がスローされない場合、キューに入れられたジョブは処理が完了した後に自動的に削除されます。

<a name="customizing-the-queue-connection-queue-name"></a>
#### キュー接続、キュー名、遅延時間のカスタマイズ

イベントリスナーのキュー接続、キュー名、またはキュー遅延時間をカスタマイズしたい場合は、リスナークラスに `$connection`、`$queue`、または `$delay` プロパティを定義できます：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        /**
         * ジョブを送信する接続名。
         *
         * @var string|null
         */
        public $connection = 'sqs';

        /**
         * ジョブを送信するキュー名。
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * ジョブが処理されるまでの時間（秒）。
         *
         * @var int
         */
        public $delay = 60;
    }

リスナーのキュー接続、キュー名、または遅延時間を実行時に定義したい場合は、リスナーに `viaConnection`、`viaQueue`、または `withDelay` メソッドを定義できます：

    /**
     * リスナーのキュー接続名を取得します。
     */
    public function viaConnection(): string
    {
        return 'sqs';
    }

    /**
     * リスナーのキュー名を取得します。
     */
    public function viaQueue(): string
    {
        return 'listeners';
    }

    /**
     * ジョブが処理されるまでの秒数を取得します。
     */
    public function withDelay(OrderShipped $event): int
    {
        return $event->highPriority ? 0 : 60;
    }

<a name="conditionally-queueing-listeners"></a>
#### 条件付きでリスナーをキューに入れる

時には、実行時にのみ利用可能なデータに基づいてリスナーをキューに入れるかどうかを決定する必要があるかもしれません。これを実現するには、リスナーに `shouldQueue` メソッドを追加して、リスナーをキューに入れるかどうかを決定できます。`shouldQueue` メソッドが `false` を返す場合、リスナーはキューに入れられません：

    <?php

    namespace App\Listeners;

    use App\Events\OrderCreated;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class RewardGiftCard implements ShouldQueue
    {
        /**
         * 顧客にギフトカードを贈る。
         */
        public function handle(OrderCreated $event): void
        {
            // ...
        }

        /**
         * リスナーをキューに入れるかどうかを決定します。
         */
        public function shouldQueue(OrderCreated $event): bool
        {
            return $event->order->subtotal >= 5000;
        }
    }

<a name="manually-interacting-with-the-queue"></a>
### キューとの手動操作

リスナーの基礎となるキュージョブの `delete` および `release` メソッドに手動でアクセスする必要がある場合は、`Illuminate\Queue\InteractsWithQueue` トレイトを使用して実現できます。このトレイトは、生成されたリスナーにデフォルトでインポートされ、これらのメソッドへのアクセスを提供します：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * イベントを処理します。
         */
        public function handle(OrderShipped $event): void
        {
            if (true) {
                $this->release(30);
            }
        }
    }

<a name="queued-event-listeners-and-database-transactions"></a>
### キューイベントリスナーとデータベーストランザクション

キューリスナーがデータベーストランザクション内でディスパッチされると、データベーストランザクションがコミットされる前にキューによって処理される可能性があります。この場合、データベーストランザクション中にモデルまたはデータベースレコードに加えた更新が、まだデータベースに反映されていない可能性があります。さらに、トランザクション内で作成されたモデルまたはデータベースレコードは、データベースに存在しない可能性があります。リスナーがこれらのモデルに依存している場合、キューリスナーをディスパッチするジョブが処理されるときに予期しないエラーが発生する可能性があります。

キュー接続の `after_commit` 設定オプションが `false` に設定されている場合でも、リスナークラスに `ShouldQueueAfterCommit` インターフェースを実装することで、すべての開いているデータベーストランザクションがコミットされた後に特定のキューリスナーをディスパッチするように指定できます：

    <?php

    namespace App\Listeners;

    use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueueAfterCommit
    {
        use InteractsWithQueue;
    }

> NOTE:  
> これらの問題を回避する方法について詳しくは、[キュージョブとデータベーストランザクション](queues.md#jobs-and-database-transactions)に関するドキュメントをご覧ください。

<a name="handling-failed-jobs"></a>
### 失敗したジョブの処理

時には、キューイベントリスナーが失敗することがあります。キューリスナーがキューワーカーによって定義された最大試行回数を超えると、リスナーの `failed` メソッドが呼び出されます。`failed` メソッドは、イベントインスタンスと失敗の原因となった `Throwable` を受け取ります：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;
    use Throwable;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * イベントを処理します。
         */
        public function handle(OrderShipped $event): void
        {
            // ...
        }

        /**
         * ジョブの失敗を処理します。
         */
        public function failed(OrderShipped $event, Throwable $exception): void
        {
            // ...
        }
    }

<a name="specifying-queued-listener-maximum-attempts"></a>
#### キューリスナーの最大試行回数の指定

キューリスナーのいずれかでエラーが発生した場合、無限に再試行し続けることは望ましくないでしょう。そのため、Laravelはリスナーの試行回数や試行時間を指定するためのさまざまな方法を提供しています。

リスナークラスに `$tries` プロパティを定義して、リスナーが失敗と見なされるまでに試行できる回数を指定できます：

    <?php

    namespace App\Listeners;

    use App\Events\OrderShipped;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Queue\InteractsWithQueue;

    class SendShipmentNotification implements ShouldQueue
    {
        use InteractsWithQueue;

        /**
         * キューリスナーが試行できる回数。
         *
         * @var int
         */
        public $tries = 5;
    }

リスナーが試行されなくなる時間を定義する代わりに、リスナーが試行される時間枠内で任意の回数試行できるようにすることもできます。これを実現するには、リスナークラスに `retryUntil` メソッドを追加します。このメソッドは `DateTime` インスタンスを返す必要があります：

    use DateTime;

    /**
     * リスナーがタイムアウトする時間を決定します。
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(5);
    }

<a name="dispatching-events"></a>
## イベントのディスパッチ

イベントをディスパッチするには、イベントの静的 `dispatch` メソッドを呼び出すことができます。このメソッドは、`Illuminate\Foundation\Events\Dispatchable` トレイトによってイベントに提供されます。`dispatch` メソッドに渡された引数は、イベントのコンストラクタに渡されます：

    <?php

    namespace App\Http\Controllers;

    use App\Events\OrderShipped;
    use App\Http\Controllers\Controller;
    use App\Models\Order;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class OrderShipmentController extends Controller
    {
        /**
         * 指定された注文を出荷します。
         */
        public function store(Request $request): RedirectResponse
        {
            $order = Order::findOrFail($request->order_id);

            // 注文出荷のロジック...

            OrderShipped::dispatch($order);

            return redirect('/orders');
        }
    }

条件付きでイベントをディスパッチしたい場合は、`dispatchIf` および `dispatchUnless` メソッドを使用できます：

    OrderShipped::dispatchIf($condition, $order);

    OrderShipped::dispatchUnless($condition, $order);

> NOTE:  
> テスト時に、特定のイベントがディスパッチされたことをアサートするだけで、実際にはそのリスナーをトリガーしないことが役立つ場合があります。Laravelの[組み込みテストヘルパー](#testing)を使えば、簡単に実現できます。

<a name="dispatching-events-after-database-transactions"></a>
### データベーストランザクション後のイベントディスパッチ

場合によっては、アクティブなデータベーストランザクションがコミットされた後にのみイベントをディスパッチするようにLaravelに指示したいことがあります。そのためには、イベントクラスに `ShouldDispatchAfterCommit` インターフェースを実装します。

このインターフェースは、現在のデータベーストランザクションがコミットされるまでイベントをディスパッチしないようにLaravelに指示します。トランザクションが失敗した場合、イベントは破棄されます。イベントがディスパッチされたときにデータベーストランザクションが進行中でない場合、イベントはすぐにディスパッチされます。

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
    use Illuminate\Foundation\Events\Dispatchable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped implements ShouldDispatchAfterCommit
    {
        use Dispatchable, InteractsWithSockets, SerializesModels;

        /**
         * 新しいイベントインスタンスの作成
         */
        public function __construct(
            public Order $order,
        ) {}
    }

<a name="event-subscribers"></a>
## イベントサブスクライバ

<a name="writing-event-subscribers"></a>
### イベントサブスクライバの作成

イベントサブスクライバは、サブスクライバクラス自体から複数のイベントをサブスクライブできるクラスであり、単一のクラス内で複数のイベントハンドラを定義できます。サブスクライバは `subscribe` メソッドを定義する必要があり、これにはイベントディスパッチャインスタンスが渡されます。与えられたディスパッチャの `listen` メソッドを呼び出して、イベントリスナーを登録できます。

    <?php

    namespace App\Listeners;

    use Illuminate\Auth\Events\Login;
    use Illuminate\Auth\Events\Logout;
    use Illuminate\Events\Dispatcher;

    class UserEventSubscriber
    {
        /**
         * ユーザーログインイベントの処理
         */
        public function handleUserLogin(Login $event): void {}

        /**
         * ユーザーログアウトイベントの処理
         */
        public function handleUserLogout(Logout $event): void {}

        /**
         * サブスクライバのリスナーを登録
         */
        public function subscribe(Dispatcher $events): void
        {
            $events->listen(
                Login::class,
                [UserEventSubscriber::class, 'handleUserLogin']
            );

            $events->listen(
                Logout::class,
                [UserEventSubscriber::class, 'handleUserLogout']
            );
        }
    }

サブスクライバ自体にイベントリスナーメソッドが定義されている場合、サブスクライバの `subscribe` メソッドからイベントとメソッド名の配列を返す方が便利な場合があります。Laravelは、イベントリスナーを登録する際に自動的にサブスクライバのクラス名を決定します。

    <?php

    namespace App\Listeners;

    use Illuminate\Auth\Events\Login;
    use Illuminate\Auth\Events\Logout;
    use Illuminate\Events\Dispatcher;

    class UserEventSubscriber
    {
        /**
         * ユーザーログインイベントの処理
         */
        public function handleUserLogin(Login $event): void {}

        /**
         * ユーザーログアウトイベントの処理
         */
        public function handleUserLogout(Logout $event): void {}

        /**
         * サブスクライバのリスナーを登録
         *
         * @return array<string, string>
         */
        public function subscribe(Dispatcher $events): array
        {
            return [
                Login::class => 'handleUserLogin',
                Logout::class => 'handleUserLogout',
            ];
        }
    }

<a name="registering-event-subscribers"></a>
### イベントサブスクライバの登録

サブスクライバを作成した後、イベントディスパッチャに登録する準備が整いました。`Event` ファサードの `subscribe` メソッドを使用してサブスクライバを登録できます。通常、これはアプリケーションの `AppServiceProvider` の `boot` メソッド内で行うべきです。

    <?php

    namespace App\Providers;

    use App\Listeners\UserEventSubscriber;
    use Illuminate\Support\Facades\Event;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスのブートストラップ
         */
        public function boot(): void
        {
            Event::subscribe(UserEventSubscriber::class);
        }
    }

<a name="testing"></a>
## テスト

イベントをディスパッチするコードをテストする際、イベントのリスナーを実際に実行せずに、イベントがディスパッチされたことをアサートしたい場合があります。リスナーのコードは直接テストでき、対応するイベントをディスパッチするコードとは別にテストできるためです。もちろん、リスナー自体をテストするには、リスナーインスタンスを作成し、テスト内で直接 `handle` メソッドを呼び出すことができます。

`Event` ファサードの `fake` メソッドを使用すると、リスナーの実行を防ぎ、テスト対象のコードを実行し、`assertDispatched`、`assertNotDispatched`、および `assertNothingDispatched` メソッドを使用してアプリケーションによってディスパッチされたイベントをアサートできます。

===  "Pest"
```php
<?php

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // 注文の発送を実行...

    // イベントがディスパッチされたことをアサート...
    Event::assertDispatched(OrderShipped::class);

    // イベントが2回ディスパッチされたことをアサート...
    Event::assertDispatched(OrderShipped::class, 2);

    // イベントがディスパッチされなかったことをアサート...
    Event::assertNotDispatched(OrderFailedToShip::class);

    // イベントがディスパッチされなかったことをアサート...
    Event::assertNothingDispatched();
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 注文の発送をテスト
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // 注文の発送を実行...

        // イベントがディスパッチされたことをアサート...
        Event::assertDispatched(OrderShipped::class);

        // イベントが2回ディスパッチされたことをアサート...
        Event::assertDispatched(OrderShipped::class, 2);

        // イベントがディスパッチされなかったことをアサート...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // イベントがディスパッチされなかったことをアサート...
        Event::assertNothingDispatched();
    }
}
```

`assertDispatched` または `assertNotDispatched` メソッドにクロージャを渡して、特定の「真実テスト」をパスするイベントがディスパッチされたことをアサートすることもできます。少なくとも1つのイベントが与えられた真実テストをパスした場合、アサーションは成功します。

    Event::assertDispatched(function (OrderShipped $event) use ($order) {
        return $event->order->id === $order->id;
    });

特定のイベントにリスナーが登録されていることを単にアサートしたい場合は、`assertListening` メソッドを使用できます。

    Event::assertListening(
        OrderShipped::class,
        SendShipmentNotification::class
    );

> WARNING:  
> `Event::fake()` を呼び出した後、イベントリスナーは実行されません。したがって、UUIDをモデルの `creating` イベント中に作成するなど、イベントに依存するモデルファクトリをテストで使用する場合、ファクトリを使用した**後に** `Event::fake()` を呼び出す必要があります。

<a name="faking-a-subset-of-events"></a>
### イベントのサブセットのフェイク

特定のイベントセットに対してのみイベントリスナーをフェイクしたい場合は、`fake` または `fakeFor` メソッドにそれらを渡すことができます。

===  "Pest"
```php
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // 他のイベントは通常通りディスパッチされます...
    $order->update([...]);
});
```

===  "PHPUnit"
```php
/**
 * 注文処理をテスト
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // 他のイベントは通常通りディスパッチされます...
    $order->update([...]);
}
```

`except` メソッドを使用して、指定されたイベントを除くすべてのイベントをフェイクすることができます。

    Event::fake()->except([
        OrderCreated::class,
    ]);

<a name="scoped-event-fakes"></a>
### スコープ付きイベントフェイク

テストの一部に対してのみイベントリスナーをフェイクしたい場合は、`fakeFor` メソッドを使用できます。

===  "Pest"
```php
<?php

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;

test('orders can be processed', function () {
    $order = Event::fakeFor(function () {
        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        return $order;
    });

    // イベントは通常通りディスパッチされ、オブザーバーが実行されます...
    $order->update([...]);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 注文処理をテスト
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // イベントは通常通りディスパッチされ、オブザーバーが実行されます...
        $order->update([...]);
    }
}
```
