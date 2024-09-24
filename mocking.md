# モック

- [イントロダクション](#introduction)
- [オブジェクトのモック](#mocking-objects)
- [ファサードのモック](#mocking-facades)
    - [ファサードのスパイ](#facade-spies)
- [時間とのやり取り](#interacting-with-time)

<a name="introduction"></a>
## イントロダクション

Laravelアプリケーションをテストする際、特定のアプリケーションの側面を「モック」して、特定のテスト中に実際に実行されないようにしたい場合があります。たとえば、イベントをディスパッチするコントローラをテストする場合、イベントリスナーをモックして、テスト中に実際に実行されないようにしたい場合があります。これにより、イベントリスナーの実行について心配することなく、コントローラのHTTPレスポンスのみをテストできます。イベントリスナーは独自のテストケースでテストできます。

Laravelは、イベント、ジョブ、その他のファサードをモックするための便利なメソッドを最初から提供しています。これらのヘルパーは主に、手動で複雑なMockeryメソッド呼び出しを行う必要がないように、Mockeryの便利なレイヤーを提供します。

<a name="mocking-objects"></a>
## オブジェクトのモック

Laravelの[サービスコンテナ](container.md)を介してアプリケーションに注入されるオブジェクトをモックする場合、モックされたインスタンスをコンテナに`instance`バインディングとしてバインドする必要があります。これにより、コンテナにオブジェクト自体を構築する代わりに、モックされたインスタンスを使用するように指示されます。

===  "Pest"
```php
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->shouldReceive('process')->once();
        })
    );
});
```

===  "PHPUnit"
```php
use App\Service;
use Mockery;
use Mockery\MockInterface;

public function test_something_can_be_mocked(): void
{
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->shouldReceive('process')->once();
        })
    );
}
```

これをより便利にするために、Laravelのベーステストケースクラスによって提供される`mock`メソッドを使用できます。たとえば、次の例は上記の例と同等です。

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

オブジェクトのいくつかのメソッドのみをモックする必要がある場合は、`partialMock`メソッドを使用できます。モックされていないメソッドは、呼び出されたときに通常通り実行されます。

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->partialMock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

同様に、オブジェクトを[スパイ](http://docs.mockery.io/en/latest/reference/spies.html)したい場合、Laravelのベーステストケースクラスは、`Mockery::spy`メソッドの便利なラッパーとして`spy`メソッドを提供します。スパイはモックと似ていますが、スパイはスパイとテスト中のコード間のすべての相互作用を記録し、コードの実行後にアサーションを行うことができます。

    use App\Service;

    $spy = $this->spy(Service::class);

    // ...

    $spy->shouldHaveReceived('process');

<a name="mocking-facades"></a>
## ファサードのモック

従来の静的メソッド呼び出しとは異なり、[ファサード](facades.md)（[リアルタイムファサード](facades.md#real-time-facades)を含む）はモックできます。これにより、従来の静的メソッドよりも大きな利点が得られ、従来の依存性注入を使用している場合と同じテスト容易性が得られます。テスト時に、コントローラの1つで発生するLaravelファサードへの呼び出しをモックしたい場合があります。たとえば、次のコントローラアクションを考えてみましょう。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * アプリケーションのすべてのユーザーのリストを取得します。
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

`shouldReceive`メソッドを使用して`Cache`ファサードへの呼び出しをモックできます。これにより、[Mockery](https://github.com/padraic/mockery)モックのインスタンスが返されます。ファサードは実際にはLaravelの[サービスコンテナ](container.md)によって解決および管理されているため、従来の静的クラスよりもはるかにテスト容易性が高くなります。たとえば、`Cache`ファサードの`get`メソッドへの呼び出しをモックしてみましょう。

=== "Pest"

  ```php
  <?php
  
  use Illuminate\Support\Facades\Cache;
  
  test('get index', function () {
      Cache::shouldReceive('get')
                  ->once()
                  ->with('key')
                  ->andReturn('value');
  
      $response = $this->get('/users');
  
      // ...
  });
  ```

=== "PHPUnit"

  ```php
  <?php
  
  namespace Tests\Feature;
  
  use Illuminate\Support\Facades\Cache;
  use Tests\TestCase;
  
  class UserControllerTest extends TestCase
  {
      public function test_get_index(): void
      {
          Cache::shouldReceive('get')
                      ->once()
                      ->with('key')
                      ->andReturn('value');
  
          $response = $this->get('/users');
  
          // ...
      }
  }
  ```

> WARNING:  
> `Request`ファサードをモックしてはいけません。代わりに、テストを実行する際に`get`や`post`などの[HTTPテストメソッド](http-tests.md)に必要な入力を渡してください。同様に、`Config`ファサードをモックする代わりに、テストで`Config::set`メソッドを呼び出してください。

<a name="facade-spies"></a>
### ファサードのスパイ

ファサードを[スパイ](http://docs.mockery.io/en/latest/reference/spies.html)したい場合は、対応するファサードで`spy`メソッドを呼び出すことができます。スパイはモックと似ていますが、スパイはスパイとテスト中のコード間のすべての相互作用を記録し、コードの実行後にアサーションを行うことができます。

===  "Pest"

  ```php
  <?php
  
  use Illuminate\Support\Facades\Cache;
  
  test('values are be stored in cache', function () {
      Cache::spy();
  
      $response = $this->get('/');
  
      $response->assertStatus(200);
  
      Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
  });
  ```

===  "PHPUnit"

  ```php
  use Illuminate\Support\Facades\Cache;
  
  public function test_values_are_be_stored_in_cache(): void
  {
      Cache::spy();
  
      $response = $this->get('/');
  
      $response->assertStatus(200);
  
      Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
  }
  ```

<a name="interacting-with-time"></a>
## 時間とのやり取り

テスト時に、`now`や`Illuminate\Support\Carbon::now()`などのヘルパーによって返される時間を変更する必要がある場合があります。幸いなことに、Laravelのベース機能テストクラスには、現在の時間を操作できるヘルパーが含まれています。

===  "Pest"

  ```php
  test('time can be manipulated', function () {
      // 未来に移動...
      $this->travel(5)->milliseconds();
      $this->travel(5)->seconds();
      $this->travel(5)->minutes();
      $this->travel(5)->hours();
      $this->travel(5)->days();
      $this->travel(5)->weeks();
      $this->travel(5)->years();
  
      // 過去に移動...
      $this->travel(-5)->hours();
  
      // 明示的な時間に移動...
      $this->travelTo(now()->subHours(6));
  
      // 現在の時間に戻る...
      $this->travelBack();
  });
  ```

===  "PHPUnit"

  ```php
  public function test_time_can_be_manipulated(): void
  {
      // 未来に移動...
      $this->travel(5)->milliseconds();
      $this->travel(5)->seconds();
      $this->travel(5)->minutes();
      $this->travel(5)->hours();
      $this->travel(5)->days();
      $this->travel(5)->weeks();
      $this->travel(5)->years();
  
      // 過去に移動...
      $this->travel(-5)->hours();
  
      // 明示的な時間に移動...
      $this->travelTo(now()->subHours(6));
  
      // 現在の時間に戻る...
      $this->travelBack();
  }
  ```

さまざまな時間旅行メソッドにクロージャを提供することもできます。クロージャは、指定された時間で時間が凍結された状態で呼び出されます。クロージャが実行されると、時間は通常に戻ります。

    $this->travel(5)->days(function () {
        // 5日後に何かをテスト...
    });

    $this->travelTo(now()->subDays(10), function () {
        // 特定の瞬間に何かをテスト...
    });

`freezeTime`メソッドを使用して現在の時間を凍結することができます。同様に、`freezeSecond`メソッドは現在の時間を凍結しますが、現在の秒の開始時に凍結します。

    use Illuminate\Support\Carbon;

    // 時間を凍結し、クロージャの実行後に通常の時間に戻す...
    $this->freezeTime(function (Carbon $time) {
        // ...
    });

    // 現在の秒を凍結し、クロージャの実行後に通常の時間に戻す...
    $this->freezeSecond(function (Carbon $time) {
        // ...
    })

予想通り、上記のすべてのメソッドは、時間に敏感なアプリケーションの動作をテストするのに主に役立ちます。たとえば、ディスカッションフォーラムの非アクティブな投稿をロックするなどです。

===  "Pest"

  ```php
  use App\Models\Thread;
  
  test('forum threads lock after one week of inactivity', function () {
      $thread = Thread::factory()->create();
  
      $this->travel(1)->week();
  
      expect($thread->isLockedByInactivity())->toBeTrue();
  });
  ```

===  "PHPUnit"

  ```php
  use App\Models\Thread;
  
  public function test_forum_threads_lock_after_one_week_of_inactivity()
  {
      $thread = Thread::factory()->create();
  
      $this->travel(1)->week();
  
      $this->assertTrue($thread->isLockedByInactivity());
  }
  ```

