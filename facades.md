# Facades

- [はじめに](#introduction)
- [Facadesをいつ利用するか](#when-to-use-facades)
    - [Facades vs. 依存性注入](#facades-vs-dependency-injection)
    - [Facades vs. ヘルパー関数](#facades-vs-helper-functions)
- [Facadesの仕組み](#how-facades-work)
- [リアルタイムFacades](#real-time-facades)
- [Facadeクラスリファレンス](#facade-class-reference)

<a name="introduction"></a>
## はじめに

Laravelのドキュメントを読んでいると、"facades"を介してLaravelの機能と対話するコードの例が見られるでしょう。Facadesは、アプリケーションの[サービスコンテナ](container.md)で利用可能なクラスに対する"静的"インターフェースを提供します。Laravelには、Laravelのほぼすべての機能にアクセスできる多くのfacadesが付属しています。

Laravelのfacadesは、サービスコンテナ内の基礎となるクラスへの"静的プロキシ"として機能し、従来の静的メソッドよりも簡潔で表現力豊かな構文の利点を提供しながら、よりテスト容易性と柔軟性を維持します。Facadesの仕組みを完全に理解していなくても問題ありません。Laravelについて学び続けることが大切です。

Laravelのすべてのfacadesは、`Illuminate\Support\Facades`名前空間で定義されています。したがって、次のように簡単にfacadeにアクセスできます。

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Laravelのドキュメント全体で、多くの例でfacadesを使用してフレームワークのさまざまな機能を説明しています。

<a name="helper-functions"></a>
#### ヘルパー関数

Facadesに加えて、Laravelはさまざまなグローバル"ヘルパー関数"を提供し、一般的なLaravel機能との対話をさらに容易にします。`view`、`response`、`url`、`config`など、よく対話するヘルパー関数がいくつかあります。Laravelが提供する各ヘルパー関数は、対応する機能とともに文書化されていますが、完全なリストは専用の[ヘルパードキュメント](helpers.md)で利用できます。

例えば、`Illuminate\Support\Facades\Response` facadeを使用してJSONレスポンスを生成する代わりに、単に`response`関数を使用できます。ヘルパー関数はグローバルに利用可能なため、使用するためにクラスをインポートする必要はありません。

    use Illuminate\Support\Facades\Response;

    Route::get('/users', function () {
        return Response::json([
            // ...
        ]);
    });

    Route::get('/users', function () {
        return response()->json([
            // ...
        ]);
    });

<a name="when-to-use-facades"></a>
## Facadesをいつ利用するか

Facadesには多くの利点があります。簡潔で覚えやすい構文を提供し、長いクラス名を覚えたり手動で注入や設定する必要なく、Laravelの機能を使用できます。さらに、PHPの動的メソッドの独自の使用により、テストが容易です。

ただし、facadesを使用する際にはいくつかの注意が必要です。facadesの主な危険性は、クラスの"スコープクリープ"です。facadesは非常に使いやすく、注入が不要なため、クラスが成長し、1つのクラスで多くのfacadesを使用することが容易になります。依存性注入を使用すると、大きなコンストラクタがクラスが大きくなりすぎていることを視覚的に通知します。したがって、facadesを使用する場合は、クラスのサイズに特に注意を払い、その責任範囲が狭く保たれるようにしてください。クラスが大きくなりすぎる場合は、複数の小さなクラスに分割することを検討してください。

<a name="facades-vs-dependency-injection"></a>
### Facades vs. 依存性注入

依存性注入の主な利点の1つは、注入されたクラスの実装を交換できることです。これはテスト中にモックやスタブを注入し、さまざまなメソッドがスタブで呼び出されたことをアサートするのに便利です。

通常、本当の静的クラスメソッドをモックまたはスタブすることはできません。しかし、facadesは動的メソッドを使用してサービスコンテナから解決されたオブジェクトへメソッド呼び出しをプロキシするため、実際には注入されたクラスインスタンスと同様にfacadesをテストできます。例えば、次のルートがあるとします。

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

Laravelのfacadeテストメソッドを使用して、`Cache::get`メソッドが期待する引数で呼び出されたことを確認する次のテストを記述できます。

===  "Pest"
```php
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

===  "PHPUnit"
```php
use Illuminate\Support\Facades\Cache;

/**
 * 基本的な機能テストの例
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="facades-vs-helper-functions"></a>
### Facades vs. ヘルパー関数

Facadesに加えて、Laravelにはビューの生成、イベントの発火、ジョブのディスパッチ、HTTPレスポンスの送信などの一般的なタスクを実行できるさまざまな"ヘルパー"関数が含まれています。多くのヘルパー関数は、対応するfacadeと同じ機能を実行します。例えば、このfacade呼び出しとヘルパー呼び出しは同等です。

    return Illuminate\Support\Facades\View::make('profile');

    return view('profile');

Facadesとヘルパー関数の間に実際的な違いはありません。ヘルパー関数を使用する場合でも、対応するfacadeと同様にテストできます。例えば、次のルートがあるとします。

    Route::get('/cache', function () {
        return cache('key');
    });

`cache`ヘルパーは、`Cache` facadeの背後にあるクラスの`get`メソッドを呼び出します。したがって、ヘルパー関数を使用していても、次のテストを記述して、メソッドが期待する引数で呼び出されたことを確認できます。

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本的な機能テストの例
     */
    public function test_basic_example(): void
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $response = $this->get('/cache');

        $response->assertSee('value');
    }

<a name="how-facades-work"></a>
## Facadesの仕組み

Laravelアプリケーションでは、facadeはコンテナからのオブジェクトへのアクセスを提供するクラスです。これを機能させる仕組みは、`Facade`クラスにあります。Laravelのfacadesと作成するカスタムfacadesは、すべて基本クラス`Illuminate\Support\Facades\Facade`を拡張します。

`Facade`基本クラスは、`__callStatic()`マジックメソッドを使用して、facadeからコンテナから解決されたオブジェクトへの呼び出しを遅延します。以下の例では、Laravelのキャッシュシステムへの呼び出しが行われます。このコードを見ると、`Cache`クラスの静的`get`メソッドが呼び出されていると思われるかもしれません。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Cache;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示します。
         */
        public function showProfile(string $id): View
        {
            $user = Cache::get('user:'.$id);

            return view('profile', ['user' => $user]);
        }
    }

ファイルの上部近くで、`Cache` facadeを"インポート"していることに注意してください。このfacadeは、`Illuminate\Contracts\Cache\Factory`インターフェースの基礎となる実装へのプロキシとして機能します。facadeを使用して行うすべての呼び出しは、Laravelのキャッシュサービスの基礎となるインスタンスに渡されます。

`Illuminate\Support\Facades\Cache`クラスを見ると、静的メソッド`get`がないことがわかります。

    class Cache extends Facade
    {
        /**
         * コンポーネントの登録名を取得します。
         */
        protected static function getFacadeAccessor(): string
        {
            return 'cache';
        }
    }

代わりに、`Cache` facadeは基本クラス`Facade`を拡張し、`getFacadeAccessor()`メソッドを定義します。このメソッドの役割は、サービスコンテナのバインディング名を返すことです。ユーザーが`Cache` facadeの任意の静的メソッドを参照すると、Laravelは[サービスコンテナ](container.md)から`cache`バインディングを解決し、要求されたメソッド（この場合は`get`）をそのオブジェクトに対して実行します。

<a name="real-time-facades"></a>
## リアルタイムFacades

リアルタイムfacadesを使用すると、アプリケーション内の任意のクラスをfacadeのように扱うことができます。これがどのように使用できるかを説明するために、まずリアルタイムfacadesを使用しないコードを見てみましょう。例えば、`Podcast`モデルに`publish`メソッドがあるとします。ただし、ポッドキャストを公開するために、`Publisher`インスタンスを注入する必要があります。

    <?php

    namespace App\Models;

    use App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * ポッドキャストを公開します。
         */
        public function publish(Publisher $publisher): void
        {
            $this->update(['publishing' => now()]);

            $publisher->publish($this);
        }
    }

メソッドにパブリッシャー実装を注入することで、注入されたパブリッシャーをモックすることでメソッドを個別に簡単にテストできます。ただし、`publish`メソッドを呼び出すたびにパブリッシャーインスタンスを常に渡す必要があります。リアルタイムfacadesを使用すると、同じテスト容易性を維持しながら、明示的に`Publisher`インスタンスを渡す必要がなくなります。リアルタイムfacadeを生成するには、インポートしたクラスの名前空間に`Facades`をプレフィックスとして付けます。

    <?php

    namespace App\Models;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * ポッドキャストを公開します。
         */
        public function publish(): void
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

```php
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * ポッドキャストを公開する。
     */
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

        Publisher::publish($this); // [tl! add]
    }
}
```

リアルタイムファサードが使用されると、パブリッシャーの実装は、`Facades`プレフィックスの後に現れるインターフェースまたはクラス名の部分を使用して、サービスコンテナから解決されます。テスト時には、Laravelの組み込みファサードテストヘルパーを使用して、このメソッド呼び出しをモックすることができます。

===  "Pest"
```php
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('ポッドキャストを公開できる', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * テスト例。
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

<a name="facade-class-reference"></a>
## ファサードクラスリファレンス

以下に、すべてのファサードとその基礎となるクラスを示します。これは、特定のファサードルートのAPIドキュメントをすばやく掘り下げるのに便利なツールです。該当する場合は、[サービスコンテナのバインディング](container.md)キーも含まれています。

<div class="overflow-auto" markdown=1>

| ファサード | クラス | サービスコンテナのバインディング |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (インスタンス) | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (インスタンス) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (インスタンス) | [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://laravel.com/api/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://laravel.com/api/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (インスタンス) | [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (インスタンス) | [Illuminate\Contracts\Debug\ExceptionHandler](https://laravel.com/api/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://laravel.com/api/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (インスタンス) | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (インスタンス) | [Illuminate\Pipeline\Pipeline](https://laravel.com/api/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://laravel.com/api/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (基底クラス) | [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (インスタンス) | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://laravel.com/api/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (インスタンス) | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (インスタンス) | [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://laravel.com/api/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (インスタンス) | [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (インスタンス) | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (インスタンス) | [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (インスタンス) | [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://laravel.com/api/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |

</div>

