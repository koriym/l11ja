# Redis

- [はじめに](#introduction)
- [設定](#configuration)
    - [クラスター](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Redisとのやり取り](#interacting-with-redis)
    - [トランザクション](#transactions)
    - [パイプラインコマンド](#pipelining-commands)
- [Pub / Sub](#pubsub)

<a name="introduction"></a>
## はじめに

[Redis](https://redis.io) は、オープンソースの高度なキーバリューストアです。キーには、[文字列](https://redis.io/docs/data-types/strings/)、[ハッシュ](https://redis.io/docs/data-types/hashes/)、[リスト](https://redis.io/docs/data-types/lists/)、[セット](https://redis.io/docs/data-types/sets/)、[ソート済みセット](https://redis.io/docs/data-types/sorted-sets/)などのデータ構造を含めることができるため、データ構造サーバーとも呼ばれます。

LaravelでRedisを使用する前に、[PhpRedis](https://github.com/phpredis/phpredis) PHP拡張機能をPECL経由でインストールし、使用することをお勧めします。この拡張機能は、「ユーザーランド」のPHPパッケージに比べてインストールが複雑ですが、Redisを多用するアプリケーションではパフォーマンスが向上する可能性があります。[Laravel Sail](sail.md)を使用している場合、この拡張機能はアプリケーションのDockerコンテナに既にインストールされています。

PhpRedis拡張機能をインストールできない場合は、Composerを介して`predis/predis`パッケージをインストールできます。Predisは、PHPで完全に記述されたRedisクライアントであり、追加の拡張機能は必要ありません。

```shell
composer require predis/predis:^2.0
```

<a name="configuration"></a>
## 設定

アプリケーションのRedis設定は、`config/database.php`設定ファイルを介して行います。このファイル内に、アプリケーションが使用するRedisサーバーを含む`redis`配列が表示されます。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        'default' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'username' => env('REDIS_USERNAME'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_DB', '0'),
        ],

        'cache' => [
            'url' => env('REDIS_URL'),
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'username' => env('REDIS_USERNAME'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', '6379'),
            'database' => env('REDIS_CACHE_DB', '1'),
        ],

    ],

設定ファイルで定義された各Redisサーバーには、名前、ホスト、ポートが必要です。ただし、Redis接続を表す単一のURLを定義する場合は例外です。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

<a name="configuring-the-connection-scheme"></a>
#### 接続スキームの設定

デフォルトでは、Redisクライアントは`tcp`スキームを使用してRedisサーバーに接続します。ただし、Redisサーバーの設定配列で`scheme`設定オプションを指定することで、TLS / SSL暗号化を使用できます。

    'default' => [
        'scheme' => 'tls',
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

<a name="clusters"></a>
### クラスター

アプリケーションがRedisサーバーのクラスターを利用している場合、これらのクラスターをRedis設定の`clusters`キー内で定義する必要があります。この設定キーはデフォルトでは存在しないため、アプリケーションの`config/database.php`設定ファイル内に作成する必要があります。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        ],

        'clusters' => [
            'default' => [
                [
                    'url' => env('REDIS_URL'),
                    'host' => env('REDIS_HOST', '127.0.0.1'),
                    'username' => env('REDIS_USERNAME'),
                    'password' => env('REDIS_PASSWORD'),
                    'port' => env('REDIS_PORT', '6379'),
                    'database' => env('REDIS_DB', '0'),
                ],
            ],
        ],

        // ...
    ],

デフォルトでは、LaravelはネイティブのRedisクラスタリングを使用します。`options.cluster`設定値が`redis`に設定されているためです。Redisクラスタリングは、フェイルオーバーを適切に処理するため、優れたデフォルトオプションです。

Laravelはクライアントサイドのシャーディングもサポートしています。ただし、クライアントサイドのシャーディングはフェイルオーバーを処理しません。したがって、主に他のプライマリデータストアから利用可能な一時的なキャッシュデータに適しています。

ネイティブのRedisクラスタリングの代わりにクライアントサイドのシャーディングを使用したい場合は、アプリケーションの`config/database.php`設定ファイル内の`options.cluster`設定値を削除できます。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            // ...
        ],

        // ...
    ],

<a name="predis"></a>
### Predis

アプリケーションがPredisパッケージを介してRedisとやり取りする場合、`REDIS_CLIENT`環境変数の値を`predis`に設定する必要があります。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // ...
    ],

デフォルトの設定オプションに加えて、Predisは各Redisサーバーに対して定義できる追加の[接続パラメータ](https://github.com/nrk/predis/wiki/Connection-Parameters)をサポートしています。これらの追加の設定オプションを利用するには、アプリケーションの`config/database.php`設定ファイル内のRedisサーバー設定に追加します。

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
        'read_write_timeout' => 60,
    ],

<a name="phpredis"></a>
### PhpRedis

デフォルトでは、LaravelはPhpRedis拡張機能を使用してRedisと通信します。LaravelがRedisと通信するために使用するクライアントは、`redis.client`設定オプションの値によって決まります。これは通常、`REDIS_CLIENT`環境変数の値を反映します。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // ...
    ],

デフォルトの設定オプションに加えて、PhpRedisは以下の追加の接続パラメータをサポートしています：`name`、`persistent`、`persistent_id`、`prefix`、`read_timeout`、`retry_interval`、`timeout`、および`context`。`config/database.php`設定ファイル内のRedisサーバー設定にこれらのオプションを追加できます。

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

<a name="phpredis-serialization"></a>
#### PhpRedisのシリアライゼーションと圧縮

PhpRedis拡張機能は、さまざまなシリアライザと圧縮アルゴリズムを使用するように設定することもできます。これらのアルゴリズムは、Redis設定の`options`配列を介して設定できます。

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
            'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
            'serializer' => Redis::SERIALIZER_MSGPACK,
            'compression' => Redis::COMPRESSION_LZ4,
        ],

        // ...
    ],

現在サポートされているシリアライザには、`Redis::SERIALIZER_NONE`（デフォルト）、`Redis::SERIALIZER_PHP`、`Redis::SERIALIZER_JSON`、`Redis::SERIALIZER_IGBINARY`、および`Redis::SERIALIZER_MSGPACK`が含まれます。

サポートされている圧縮アルゴリズムには、`Redis::COMPRESSION_NONE`（デフォルト）、`Redis::COMPRESSION_LZF`、`Redis::COMPRESSION_ZSTD`、および`Redis::COMPRESSION_LZ4`が含まれます。

<a name="interacting-with-redis"></a>
## Redisとのやり取り

`Redis` [ファサード](facades.md)でさまざまなメソッドを呼び出すことで、Redisとやり取りできます。`Redis`ファサードは動的メソッドをサポートしているため、ファサードで任意の[Redisコマンド](https://redis.io/commands)を呼び出すことができ、コマンドは直接Redisに渡されます。この例では、`Redis`ファサードの`get`メソッドを呼び出してRedisの`GET`コマンドを呼び出します。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示します。
         */
        public function show(string $id): View
        {
            return view('user.profile', [
                'user' => Redis::get('user:profile:'.$id)
            ]);
        }
    }

上述のように、`Redis`ファサードでRedisのコマンドを呼び出すことができます。Laravelはマジックメソッドを使用して、コマンドをRedisサーバーに渡します。Redisコマンドが引数を期待する場合、それらをファサードの対応するメソッドに渡す必要があります。

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

または、`Redis`ファサードの`command`メソッドを使用してサーバーにコマンドを渡すこともできます。このメソッドは、コマンド名を最初の引数として、値の配列を2番目の引数として受け取ります。

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### 複数のRedis接続の使用

アプリケーションの`config/database.php`設定ファイルで、複数のRedis接続/サーバーを定義できます。`Redis`ファサードの`connection`メソッドを使用して、特定のRedis接続に接続できます。

```php
$redis = Redis::connection('connection-name');
```

デフォルトのRedis接続のインスタンスを取得するには、追加の引数なしで`connection`メソッドを呼び出します。

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### トランザクション

`Redis`ファサードの`transaction`メソッドは、Redisのネイティブな`MULTI`および`EXEC`コマンドの便利なラッパーを提供します。`transaction`メソッドは、唯一の引数としてクロージャを受け取ります。このクロージャはRedis接続インスタンスを受け取り、このインスタンスに対して任意のコマンドを発行できます。クロージャ内で発行されたすべてのRedisコマンドは、単一のアトミックトランザクションで実行されます。

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> WARNING:  
> Redisトランザクションを定義する際に、Redis接続から値を取得することはできません。トランザクションは単一のアトミック操作として実行され、その操作はクロージャがすべてのコマンドの実行を終了するまで実行されないことを覚えておいてください。

#### Luaスクリプト

`eval`メソッドは、複数のRedisコマンドを単一のアトミック操作で実行する別の方法を提供します。ただし、`eval`メソッドには、その操作中にRedisのキー値と対話したり検査したりできるという利点があります。Redisスクリプトは[Luaプログラミング言語](https://www.lua.org)で書かれています。

`eval`メソッドは最初は少し怖いかもしれませんが、基本的な例を見てみましょう。`eval`メソッドはいくつかの引数を期待します。まず、Luaスクリプト（文字列として）をメソッドに渡す必要があります。次に、スクリプトが対話するキーの数（整数として）を渡す必要があります。第三に、それらのキーの名前を渡す必要があります。最後に、スクリプト内でアクセスするために必要な追加の引数を渡すことができます。

この例では、カウンターをインクリメントし、その新しい値を検査し、最初のカウンターの値が5より大きい場合に2番目のカウンターをインクリメントします。最後に、最初のカウンターの値を返します。

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

> WARNING:  
> Redisスクリプトについての詳細は、[Redisドキュメント](https://redis.io/commands/eval)を参照してください。

<a name="pipelining-commands"></a>
### パイプラインコマンド

場合によっては、数十のRedisコマンドを実行する必要があるかもしれません。各コマンドに対してRedisサーバーへのネットワークトリップを行う代わりに、`pipeline`メソッドを使用できます。`pipeline`メソッドは、Redisインスタンスを受け取るクロージャを1つの引数として受け取ります。このRedisインスタンスに対してすべてのコマンドを発行し、それらはすべてRedisサーバーに同時に送信され、サーバーへのネットワークトリップを減らします。コマンドは発行された順序で実行されます。

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## パブリッシュ / サブスクライブ

Laravelは、Redisの`publish`および`subscribe`コマンドに対する便利なインターフェースを提供します。これらのRedisコマンドを使用すると、特定の「チャネル」でメッセージをリッスンできます。他のアプリケーションや別のプログラミング言語を使用して、チャネルにメッセージを公開できます。これにより、アプリケーションやプロセス間の簡単な通信が可能になります。

まず、`subscribe`メソッドを使用してチャネルリスナーを設定しましょう。このメソッド呼び出しは、`subscribe`メソッドを呼び出すと長時間実行されるプロセスが開始されるため、[Artisanコマンド](artisan.md)内に配置します。

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * コンソールコマンドの名前とシグネチャ。
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * コンソールコマンドの説明。
     *
     * @var string
     */
    protected $description = 'Redisチャネルのサブスクライブ';

    /**
     * コンソールコマンドの実行。
     */
    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

これで、`publish`メソッドを使用してチャネルにメッセージを公開できます。

```php
use Illuminate\Support\Facades\Redis;

Route::get('/publish', function () {
    // ...

    Redis::publish('test-channel', json_encode([
        'name' => 'Adam Wathan'
    ]));
});
```

<a name="wildcard-subscriptions"></a>
#### ワイルドカードサブスクリプション

`psubscribe`メソッドを使用すると、ワイルドカードチャネルをサブスクライブできます。これは、すべてのチャネルですべてのメッセージをキャッチするのに便利です。チャネル名は、提供されたクロージャの2番目の引数として渡されます。

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```

