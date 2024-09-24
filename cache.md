# キャッシュ

- [はじめに](#introduction)
- [設定](#configuration)
    - [ドライバの前提条件](#driver-prerequisites)
- [キャッシュの使用](#cache-usage)
    - [キャッシュインスタンスの取得](#obtaining-a-cache-instance)
    - [キャッシュからのアイテムの取得](#retrieving-items-from-the-cache)
    - [キャッシュへのアイテムの保存](#storing-items-in-the-cache)
    - [キャッシュからのアイテムの削除](#removing-items-from-the-cache)
    - [キャッシュヘルパー](#the-cache-helper)
- [アトミックロック](#atomic-locks)
    - [ロックの管理](#managing-locks)
    - [プロセス間でのロックの管理](#managing-locks-across-processes)
- [カスタムキャッシュドライバの追加](#adding-custom-cache-drivers)
    - [ドライバの作成](#writing-the-driver)
    - [ドライバの登録](#registering-the-driver)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

アプリケーションが実行するデータの取得や処理のタスクの中には、CPUに負荷がかかったり、完了までに数秒かかるものがあります。このような場合、取得したデータを一定期間キャッシュしておくことで、後続の同じデータのリクエストで迅速に取得できるようになります。キャッシュされたデータは通常、[Memcached](https://memcached.org)や[Redis](https://redis.io)のような非常に高速なデータストアに保存されます。

幸いなことに、Laravelはさまざまなキャッシュバックエンドに対して表現力豊かで統一されたAPIを提供しており、これにより高速なデータ取得とWebアプリケーションのスピードアップを実現できます。

<a name="configuration"></a>
## 設定

アプリケーションのキャッシュ設定ファイルは`config/cache.php`にあります。このファイルで、アプリケーション全体でデフォルトで使用するキャッシュストアを指定できます。Laravelは、[Memcached](https://memcached.org)、[Redis](https://redis.io)、[DynamoDB](https://aws.amazon.com/dynamodb)、リレーショナルデータベースなど、一般的なキャッシュバックエンドをすぐにサポートしています。さらに、ファイルベースのキャッシュドライバも利用可能で、`array`と"null"キャッシュドライバは自動テスト用の便利なキャッシュバックエンドを提供します。

キャッシュ設定ファイルには、他にもさまざまなオプションが含まれていますが、これらについては後で確認できます。デフォルトでは、Laravelは`database`キャッシュドライバを使用するように設定されており、シリアライズされたキャッシュオブジェクトをアプリケーションのデータベースに保存します。

<a name="driver-prerequisites"></a>
### ドライバの前提条件

<a name="prerequisites-database"></a>
#### データベース

`database`キャッシュドライバを使用する場合、キャッシュデータを格納するデータベーステーブルが必要です。通常、これはLaravelのデフォルトの`0001_01_01_000001_create_cache_table.php` [データベースマイグレーション](migrations.md)に含まれています。ただし、アプリケーションにこのマイグレーションが含まれていない場合は、`make:cache-table` Artisanコマンドを使用して作成できます。

```shell
php artisan make:cache-table

php artisan migrate
```

<a name="memcached"></a>
#### Memcached

Memcachedドライバを使用するには、[Memcached PECLパッケージ](https://pecl.php.net/package/memcached)をインストールする必要があります。`config/cache.php`設定ファイルにすべてのMemcachedサーバーをリストアップできます。このファイルには、開始するための`memcached.servers`エントリが既に含まれています。

    'memcached' => [
        // ...

        'servers' => [
            [
                'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                'port' => env('MEMCACHED_PORT', 11211),
                'weight' => 100,
            ],
        ],
    ],

必要に応じて、`host`オプションをUNIXソケットパスに設定できます。その場合、`port`オプションを`0`に設定する必要があります。

    'memcached' => [
        // ...

        'servers' => [
            [
                'host' => '/var/run/memcached/memcached.sock',
                'port' => 0,
                'weight' => 100
            ],
        ],
    ],

<a name="redis"></a>
#### Redis

LaravelでRedisキャッシュを使用する前に、PECLを介してPhpRedis PHP拡張機能をインストールするか、Composerを介して`predis/predis`パッケージ（~2.0）をインストールする必要があります。[Laravel Sail](sail.md)には、この拡張機能が既に含まれています。さらに、公式のLaravelデプロイプラットフォームである[Laravel Forge](https://forge.laravel.com)や[Laravel Vapor](https://vapor.laravel.com)には、デフォルトでPhpRedis拡張機能がインストールされています。

Redisの設定に関する詳細情報は、[Laravelドキュメントページ](redis.md#configuration)を参照してください。

<a name="dynamodb"></a>
#### DynamoDB

[DynamoDB](https://aws.amazon.com/dynamodb)キャッシュドライバを使用する前に、すべてのキャッシュデータを格納するDynamoDBテーブルを作成する必要があります。通常、このテーブルは`cache`という名前にします。ただし、`cache`設定ファイル内の`stores.dynamodb.table`設定値に基づいてテーブル名を指定する必要があります。テーブル名は、`DYNAMODB_CACHE_TABLE`環境変数を介して設定することもできます。

このテーブルには、アプリケーションの`cache`設定ファイル内の`stores.dynamodb.attributes.key`設定項目の値に対応する名前を持つ文字列のパーティションキーも必要です。デフォルトでは、パーティションキーは`key`という名前になります。

次に、LaravelアプリケーションがDynamoDBと通信できるようにAWS SDKをインストールします。

```shell
composer require aws/aws-sdk-php
```

さらに、DynamoDBキャッシュストアの設定オプションに値を指定する必要があります。通常、これらのオプション（`AWS_ACCESS_KEY_ID`や`AWS_SECRET_ACCESS_KEY`など）は、アプリケーションの`.env`設定ファイルで定義する必要があります。

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

<a name="cache-usage"></a>
## キャッシュの使用

<a name="obtaining-a-cache-instance"></a>
### キャッシュインスタンスの取得

キャッシュストアインスタンスを取得するには、`Cache`ファサードを使用できます。これは、このドキュメント全体で使用するものです。`Cache`ファサードは、Laravelのキャッシュ契約の基礎となる実装への便利で簡潔なアクセスを提供します。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * アプリケーションのすべてのユーザーのリストを表示します。
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

<a name="accessing-multiple-cache-stores"></a>
#### 複数のキャッシュストアへのアクセス

`Cache`ファサードを使用すると、`store`メソッドを介してさまざまなキャッシュストアにアクセスできます。`store`メソッドに渡されるキーは、`cache`設定ファイル内の`stores`設定配列にリストされているストアのいずれかに対応している必要があります。

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 600); // 10分

<a name="retrieving-items-from-the-cache"></a>
### キャッシュからのアイテムの取得

`Cache`ファサードの`get`メソッドは、キャッシュからアイテムを取得するために使用されます。アイテムがキャッシュに存在しない場合、`null`が返されます。必要に応じて、`get`メソッドに2番目の引数を渡して、アイテムが存在しない場合に返すデフォルト値を指定できます。

    $value = Cache::get('key');

    $value = Cache::get('key', 'default');

デフォルト値としてクロージャを渡すこともできます。指定されたアイテムがキャッシュに存在しない場合、クロージャの結果が返されます。クロージャを渡すことで、データベースや他の外部サービスからデフォルト値を取得するのを遅延させることができます。

    $value = Cache::get('key', function () {
        return DB::table(/* ... */)->get();
    });

<a name="determining-item-existence"></a>
#### アイテムの存在の確認

`has`メソッドを使用して、アイテムがキャッシュに存在するかどうかを確認できます。アイテムが存在するがその値が`null`の場合、このメソッドは`false`を返します。

    if (Cache::has('key')) {
        // ...
    }

<a name="incrementing-decrementing-values"></a>
#### 値の増減

`increment`および`decrement`メソッドを使用して、キャッシュ内の整数アイテムの値を調整できます。これらのメソッドは、アイテムの値を増減する量を示すオプションの2番目の引数を受け入れます。

    // 値が存在しない場合は初期化します...
    Cache::add('key', 0, now()->addHours(4));

    // 値を増減します...
    Cache::increment('key');
    Cache::increment('key', $amount);
    Cache::decrement('key');
    Cache::decrement('key', $amount);

<a name="retrieve-store"></a>
#### 取得と保存

キャッシュからアイテムを取得したいが、要求されたアイテムが存在しない場合はデフォルト値を保存したい場合があります。たとえば、すべてのユーザーをキャッシュから取得するか、存在しない場合はデータベースから取得してキャッシュに追加する場合です。これは、`Cache::remember`メソッドを使用して行うことができます。

    $value = Cache::remember('users', $seconds, function () {
        return DB::table('users')->get();
    });

キャッシュにアイテムが存在しない場合、`remember`メソッドに渡されたクロージャが実行され、その結果がキャッシュに配置されます。

`rememberForever`メソッドを使用して、キャッシュからアイテムを取得するか、存在しない場合は永続的に保存することもできます。

    $value = Cache::rememberForever('users', function () {
        return DB::table('users')->get();
    });

<a name="swr"></a>
#### Stale While Revalidate

`Cache::remember`メソッドを使用する際、キャッシュされた値が期限切れになった場合、一部のユーザーは遅い応答時間を経験するかもしれません。特定の種類のデータに対しては、キャッシュされた値が再計算されている間に部分的に古いデータを提供することで、キャッシュされた値が計算されている間に一部のユーザーが遅い応答時間を経験することを防ぐことができます。これはしばしば「stale-while-revalidate」パターンと呼ばれ、`Cache::flexible`メソッドはこのパターンの実装を提供します。

flexibleメソッドは、キャッシュされた値が「新鮮」であると見なされる期間と、「古く」なる期間を指定する配列を受け取ります。配列の最初の値はキャッシュが新鮮であると見なされる秒数を表し、2番目の値は再計算が必要になるまで古いデータとして提供できる期間を定義します。

リクエストが新鮮な期間内（最初の値の前）に行われた場合、キャッシュは再計算なしで即座に返されます。リクエストが古い期間内（2つの値の間）に行われた場合、古い値がユーザーに提供され、応答がユーザーに送信された後にキャッシュされた値を更新するための[遅延関数](helpers.md#deferred-functions)が登録されます。リクエストが2番目の値の後に行われた場合、キャッシュは期限切れと見なされ、値は即座に再計算されます。これにより、ユーザーにとって遅い応答が発生する可能性があります。

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### 取得と削除

キャッシュからアイテムを取得してから削除する必要がある場合は、`pull`メソッドを使用できます。`get`メソッドと同様に、アイテムがキャッシュに存在しない場合は`null`が返されます。

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### キャッシュへのアイテムの保存

`Cache`ファサードの`put`メソッドを使用して、キャッシュにアイテムを保存できます。

```php
Cache::put('key', 'value', $seconds = 10);
```

保存時間が`put`メソッドに渡されない場合、アイテムは無期限に保存されます。

```php
Cache::put('key', 'value');
```

秒数を整数で渡す代わりに、キャッシュされたアイテムの希望する有効期限を表す`DateTime`インスタンスを渡すこともできます。

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

<a name="store-if-not-present"></a>
#### 存在しない場合に保存

`add`メソッドは、アイテムがまだキャッシュストアに存在しない場合にのみ、アイテムをキャッシュに追加します。アイテムが実際にキャッシュに追加された場合、メソッドは`true`を返します。それ以外の場合、メソッドは`false`を返します。`add`メソッドはアトミック操作です。

```php
Cache::add('key', 'value', $seconds);
```

<a name="storing-items-forever"></a>
#### 永続的に保存

`forever`メソッドを使用して、アイテムをキャッシュに永続的に保存できます。これらのアイテムは期限切れにならないため、`forget`メソッドを使用して手動でキャッシュから削除する必要があります。

```php
Cache::forever('key', 'value');
```

> NOTE:  
> Memcachedドライバを使用している場合、「永続的に」保存されたアイテムは、キャッシュがサイズ制限に達したときに削除される可能性があります。

<a name="removing-items-from-the-cache"></a>
### キャッシュからのアイテムの削除

`forget`メソッドを使用して、キャッシュからアイテムを削除できます。

```php
Cache::forget('key');
```

また、有効期限の秒数としてゼロまたは負の数を指定することで、アイテムを削除することもできます。

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

`flush`メソッドを使用して、キャッシュ全体をクリアすることができます。

```php
Cache::flush();
```

> WARNING:  
> キャッシュのフラッシュは、設定されたキャッシュの「プレフィックス」を尊重せず、キャッシュ内のすべてのエントリを削除します。他のアプリケーションと共有されているキャッシュをクリアする際には、この点を慎重に考慮してください。

<a name="the-cache-helper"></a>
### キャッシュヘルパー

`Cache`ファサードを使用するだけでなく、グローバルな`cache`関数を使用してキャッシュを介してデータを取得および保存することもできます。`cache`関数が単一の文字列引数で呼び出されると、指定されたキーの値が返されます。

```php
$value = cache('key');
```

キーと値のペアの配列と有効期限を関数に渡すと、指定された期間の間、キャッシュに値が保存されます。

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

`cache`関数が引数なしで呼び出されると、`Illuminate\Contracts\Cache\Factory`の実装のインスタンスが返され、他のキャッシュメソッドを呼び出すことができます。

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> NOTE:  
> グローバルな`cache`関数の呼び出しをテストする際には、[ファサードのテスト](mocking.md#mocking-facades)と同様に`Cache::shouldReceive`メソッドを使用できます。

<a name="atomic-locks"></a>
## アトミックロック

> WARNING:  
> この機能を利用するには、アプリケーションが`memcached`、`redis`、`dynamodb`、`database`、`file`、または`array`キャッシュドライバをアプリケーションのデフォルトキャッシュドライバとして使用している必要があります。さらに、すべてのサーバーが同じ中央キャッシュサーバーと通信している必要があります。

<a name="managing-locks"></a>
### ロックの管理

アトミックロックを使用すると、競合状態を心配することなく分散ロックを操作できます。たとえば、[Laravel Forge](https://forge.laravel.com)はアトミックロックを使用して、一度に1つのサーバーでリモートタスクが実行されることを保証します。`Cache::lock`メソッドを使用してロックを作成および管理できます。

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // ロックが10秒間取得されました...

    $lock->release();
}
```

`get`メソッドはクロージャも受け取ります。クロージャが実行された後、Laravelは自動的にロックを解放します。

```php
Cache::lock('foo', 10)->get(function () {
    // ロックが10秒間取得され、自動的に解放されます...
});
```

ロックがリクエスト時に利用できない場合、Laravelに指定された秒数待機するように指示できます。指定された時間制限内にロックを取得できない場合、`Illuminate\Contracts\Cache\LockTimeoutException`がスローされます。

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // 最大5秒間待機してロックが取得されました...
} catch (LockTimeoutException $e) {
    // ロックを取得できませんでした...
} finally {
    $lock->release();
}
```

この例は、クロージャを`block`メソッドに渡すことで簡略化できます。このメソッドにクロージャが渡されると、Laravelは指定された秒数の間ロックを取得しようとし、クロージャが実行された後に自動的にロックを解放します。

```php
Cache::lock('foo', 10)->block(5, function () {
    // 最大5秒間待機してロックが取得されました...
});
```

<a name="managing-locks-across-processes"></a>
### プロセス間でのロックの管理

場合によっては、あるプロセスでロックを取得し、別のプロセスで解放することが望ましい場合があります。たとえば、Webリクエスト中にロックを取得し、そのリクエストによってトリガーされたキュージョブの終了時にロックを解放することがあります。このシナリオでは、ロックのスコープ付き「所有者トークン」をキュージョブに渡し、ジョブが指定されたトークンを使用してロックを再インスタンス化できるようにする必要があります。

以下の例では、ロックが正常に取得された場合にキュージョブをディスパッチします。さらに、ロックの`owner`メソッドを介してキュージョブにロックの所有者トークンを渡します。

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

アプリケーションの`ProcessPodcast`ジョブ内で、所有者トークンを使用してロックを復元し、解放します。

```php
Cache::restoreLock('processing', $this->owner)->release();
```

現在の所有者を尊重せずにロックを解放したい場合は、`forceRelease`メソッドを使用できます。

```php
Cache::lock('processing')->forceRelease();
```

<a name="adding-custom-cache-drivers"></a>
## カスタムキャッシュドライバの追加

<a name="writing-the-driver"></a>
### ドライバの作成

カスタムキャッシュドライバを作成するには、まず`Illuminate\Contracts\Cache\Store`[契約](contracts.md)を実装する必要があります。したがって、MongoDBキャッシュの実装は次のようになります。

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

各メソッドをMongoDB接続を使用して実装する必要があります。各メソッドの実装例については、[Laravelフレームワークのソースコード](https://github.com/laravel/framework)内の`Illuminate\Cache\MemcachedStore`を参照してください。実装が完了したら、`Cache`ファサードの`extend`メソッドを呼び出してカスタムドライバの登録を完了できます。

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> NOTE:  
> カスタムキャッシュドライバのコードをどこに配置するか迷った場合、`app`ディレクトリ内に`Extensions`名前空間を作成することができます。ただし、Laravelには厳格なアプリケーション構造がなく、アプリケーションを好みに合わせて自由に整理できることに注意してください。

<a name="registering-the-driver"></a>
### ドライバの登録

Laravelにカスタムキャッシュドライバを登録するために、`Cache`ファサードの`extend`メソッドを使用します。他のサービスプロバイダが`boot`メソッド内でキャッシュされた値を読み取ろうとする可能性があるため、カスタムドライバを`booting`コールバック内で登録します。`booting`コールバックを使用することで、アプリケーションのサービスプロバイダの`boot`メソッドが呼び出される直前に、すべてのサービスプロバイダの`register`メソッドが呼び出された後に、カスタムドライバが登録されることを保証できます。アプリケーションの`App\Providers\AppServiceProvider`クラスの`register`メソッド内で`booting`コールバックを登録します。

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }

    /**
     * 任意のアプリケーションサービスを起動します。
     */
    public function boot(): void
    {
        // ...
    }
}
```

`extend`メソッドに渡される最初の引数はドライバの名前です。これは`config/cache.php`設定ファイル内の`driver`オプションに対応します。2番目の引数は`Illuminate\Cache\Repository`インスタンスを返すべきクロージャです。クロージャには`$app`インスタンスが渡されます。これは[サービスコンテナ](container.md)のインスタンスです。

拡張機能が登録されたら、アプリケーションの`config/cache.php`設定ファイル内の`CACHE_STORE`環境変数または`default`オプションを拡張機能の名前に更新します。

<a name="events"></a>
## イベント

すべてのキャッシュ操作でコードを実行するには、キャッシュによってディスパッチされる様々な[イベント](events.md)をリッスンすることができます。

<div class="overflow-auto" markdown=1>

| イベント名 |
| --- |
| `Illuminate\Cache\Events\CacheHit` |
| `Illuminate\Cache\Events\CacheMissed` |
| `Illuminate\Cache\Events\KeyForgotten` |
| `Illuminate\Cache\Events\KeyWritten` |

</div>

パフォーマンスを向上させるために、アプリケーションの`config/cache.php`設定ファイル内で特定のキャッシュストアの`events`設定オプションを`false`に設定することで、キャッシュイベントを無効にすることができます。

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```

