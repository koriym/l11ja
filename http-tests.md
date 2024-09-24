# HTTPテスト

- [はじめに](#introduction)
- [リクエストの作成](#making-requests)
    - [リクエストヘッダのカスタマイズ](#customizing-request-headers)
    - [クッキー](#cookies)
    - [セッション / 認証](#session-and-authentication)
    - [レスポンスのデバッグ](#debugging-responses)
    - [例外処理](#exception-handling)
- [JSON APIのテスト](#testing-json-apis)
    - [流暢なJSONテスト](#fluent-json-testing)
- [ファイルアップロードのテスト](#testing-file-uploads)
- [ビューのテスト](#testing-views)
    - [Bladeとコンポーネントのレンダリング](#rendering-blade-and-components)
- [利用可能なアサーション](#available-assertions)
    - [レスポンスのアサーション](#response-assertions)
    - [認証のアサーション](#authentication-assertions)
    - [バリデーションのアサーション](#validation-assertions)

<a name="introduction"></a>
## はじめに

Laravelは、アプリケーションに対してHTTPリクエストを行い、レスポンスを調査するための非常に流暢なAPIを提供します。例えば、以下に定義された機能テストを見てみましょう。

===  "Pest"
```php
<?php

test('the application returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

`get`メソッドはアプリケーションに対して`GET`リクエストを行い、`assertStatus`メソッドは返されたレスポンスが指定されたHTTPステータスコードを持つべきであることをアサートします。この単純なアサーションに加えて、Laravelにはレスポンスヘッダ、コンテンツ、JSON構造などを検査するためのさまざまなアサーションが含まれています。

<a name="making-requests"></a>
## リクエストの作成

アプリケーションにリクエストを行うには、テスト内で`get`、`post`、`put`、`patch`、または`delete`メソッドを呼び出すことができます。これらのメソッドは実際にはアプリケーションに対して「本物の」HTTPリクエストを発行しません。代わりに、ネットワークリクエスト全体が内部的にシミュレートされます。

`Illuminate\Http\Response`インスタンスを返す代わりに、テストリクエストメソッドは`Illuminate\Testing\TestResponse`のインスタンスを返します。これは、アプリケーションのレスポンスを検査するための[さまざまな便利なアサーション](#available-assertions)を提供します。

===  "Pest"
```php
<?php

test('basic request', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_a_basic_request(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

一般的に、各テストはアプリケーションに対して1つのリクエストのみを行うべきです。1つのテストメソッド内で複数のリクエストが実行されると、予期しない動作が発生する可能性があります。

> NOTE:  
> 便宜上、CSRFミドルウェアはテスト実行時に自動的に無効になります。

<a name="customizing-request-headers"></a>
### リクエストヘッダのカスタマイズ

`withHeaders`メソッドを使用して、リクエストがアプリケーションに送信される前にリクエストのヘッダをカスタマイズできます。このメソッドを使用すると、リクエストに任意のカスタムヘッダを追加できます。

===  "Pest"
```php
<?php

test('interacting with headers', function () {
    $response = $this->withHeaders([
        'X-Header' => 'Value',
    ])->post('/user', ['name' => 'Sally']);

    $response->assertStatus(201);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的な機能テスト例。
     */
    public function test_interacting_with_headers(): void
    {
        $response = $this->withHeaders([
            'X-Header' => 'Value',
        ])->post('/user', ['name' => 'Sally']);

        $response->assertStatus(201);
    }
}
```

<a name="cookies"></a>
### クッキー

`withCookie`または`withCookies`メソッドを使用して、リクエストを行う前にクッキーの値を設定できます。`withCookie`メソッドはクッキー名と値を2つの引数として受け取り、`withCookies`メソッドは名前と値のペアの配列を受け取ります。

===  "Pest"
```php
<?php

test('interacting with cookies', function () {
    $response = $this->withCookie('color', 'blue')->get('/');

    $response = $this->withCookies([
        'color' => 'blue',
        'name' => 'Taylor',
    ])->get('/');

    //
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_cookies(): void
    {
        $response = $this->withCookie('color', 'blue')->get('/');

        $response = $this->withCookies([
            'color' => 'blue',
            'name' => 'Taylor',
        ])->get('/');

        //
    }
}
```

<a name="session-and-authentication"></a>
### セッション / 認証

Laravelは、HTTPテスト中にセッションを操作するためのいくつかのヘルパーを提供します。まず、`withSession`メソッドを使用してセッションデータを指定された配列に設定できます。これは、アプリケーションにリクエストを発行する前にセッションにデータをロードするのに便利です。

===  "Pest"
```php
<?php

test('interacting with the session', function () {
    $response = $this->withSession(['banned' => false])->get('/');

    //
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_the_session(): void
    {
        $response = $this->withSession(['banned' => false])->get('/');

        //
    }
}
```

Laravelのセッションは通常、現在認証されているユーザーの状態を維持するために使用されます。したがって、`actingAs`ヘルパーメソッドは、指定されたユーザーを現在のユーザーとして認証する簡単な方法を提供します。例えば、[モデルファクトリ](eloquent-factories.md)を使用してユーザーを生成し、認証することができます。

===  "Pest"
```php
<?php

use App\Models\User;

test('an action that requires authentication', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
                     ->withSession(['banned' => false])
                     ->get('/');

    //
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_an_action_that_requires_authentication(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
                         ->withSession(['banned' => false])
                         ->get('/');

        //
    }
}
```

`actingAs`メソッドの第2引数としてガード名を指定することで、指定されたユーザーを認証するために使用するガードを指定できます。`actingAs`メソッドに提供されたガードは、テストの間もデフォルトのガードとなります。

    $this->actingAs($user, 'web')

<a name="debugging-responses"></a>
### レスポンスのデバッグ

アプリケーションに対してテストリクエストを行った後、`dump`、`dumpHeaders`、および`dumpSession`メソッドを使用してレスポンスの内容を調査およびデバッグできます。

===  "Pest"
```php
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dumpHeaders();

    $response->dumpSession();

    $response->dump();
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dumpHeaders();

        $response->dumpSession();

        $response->dump();
    }
}
```

あるいは、`dd`、`ddHeaders`、および`ddSession`メソッドを使用してレスポンスに関する情報をダンプし、実行を停止することもできます。

===  "Pest"
```php
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->ddHeaders();

    $response->ddSession();

    $response->dd();
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->ddHeaders();

        $response->ddSession();

        $response->dd();
    }
}
```

<a name="exception-handling"></a>
### 例外処理

アプリケーションが特定の例外をスローすることをテストする必要がある場合があります。これを実現するために、`Exceptions`ファサードを介して例外ハンドラを「フェイク」することができます。例外ハンドラがフェイクされた後、リクエスト中にスローされた例外に対して`assertReported`および`assertNotReported`メソッドを使用してアサーションを行うことができます。

===  "Pest"
```php
<?php

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('exception is thrown', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // 例外がスローされたことをアサート...
    Exceptions::assertReported(InvalidOrderException::class);

    // 例外に対してアサート...
    Exceptions::assertReported(function (InvalidOrderException $e) {
        return $e->getMessage() === 'The order was invalid.';
    });
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_exception_is_thrown(): void
    {
        Exceptions::fake();

        $response = $this->get('/');

        // 例外がスローされたことをアサート...
        Exceptions::assertReported(InvalidOrderException::class);

        // 例外に対するアサーション...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'The order was invalid.';
        });
    }
}
```

`assertNotReported` および `assertNothingReported` メソッドを使用して、特定の例外がリクエスト中にスローされなかったこと、または例外がスローされなかったことをアサートすることができます:

```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```

特定のリクエストに対して例外処理を完全に無効にするには、リクエストを行う前に `withoutExceptionHandling` メソッドを呼び出します:

```php
$response = $this->withoutExceptionHandling()->get('/');
```

さらに、アプリケーションがPHP言語やアプリケーションが使用しているライブラリによって非推奨となった機能を使用していないことを確認したい場合、リクエストを行う前に `withoutDeprecationHandling` メソッドを呼び出すことができます。非推奨の処理が無効になっている場合、非推奨の警告は例外に変換され、テストが失敗する原因となります:

```php
$response = $this->withoutDeprecationHandling()->get('/');
```

`assertThrows` メソッドを使用して、指定されたクロージャ内のコードが指定されたタイプの例外をスローすることをアサートすることができます:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```

スローされた例外を検査してアサーションを行いたい場合、`assertThrows` メソッドの第二引数としてクロージャを提供することができます:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```

<a name="testing-json-apis"></a>
## JSON APIのテスト

Laravelは、JSON APIとそのレスポンスをテストするためのいくつかのヘルパーも提供しています。例えば、`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson`、および `optionsJson` メソッドを使用して、さまざまなHTTPメソッドでJSONリクエストを発行することができます。これらのメソッドにデータやヘッダーを簡単に渡すこともできます。始めるために、`/api/user` に `POST` リクエストを行い、期待されるJSONデータが返されたことをアサートするテストを書いてみましょう:

===  "Pest"
```php
<?php

test('making an api request', function () {
    $response = $this->postJson('/api/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJson([
            'created' => true,
         ]);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_making_an_api_request(): void
    {
        $response = $this->postJson('/api/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

さらに、JSONレスポンスデータはレスポンス上の配列変数としてアクセスできるため、JSONレスポンス内で返される個々の値を簡単に検査することができます:

===  "Pest"
```php
expect($response['created'])->toBeTrue();
```

===  "PHPUnit"
```php
$this->assertTrue($response['created']);
```

> NOTE:  
> `assertJson` メソッドは、アプリケーションによって返されたJSONレスポンス内に指定された配列が存在することを確認するために、レスポンスを配列に変換します。したがって、JSONレスポンスに他のプロパティが存在する場合でも、指定されたフラグメントが存在する限り、このテストは合格します。

<a name="verifying-exact-match"></a>
#### 完全一致のJSONアサーション

前述のように、`assertJson` メソッドを使用して、JSONレスポンス内にJSONのフラグメントが存在することをアサートすることができます。指定された配列がアプリケーションによって返されたJSONと**完全に一致する**ことを確認したい場合は、`assertExactJson` メソッドを使用する必要があります:

===  "Pest"
```php
<?php

test('asserting an exact json match', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertExactJson([
            'created' => true,
        ]);
});

```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_an_exact_json_match(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

<a name="verifying-json-paths"></a>
#### JSONパスのアサーション

JSONレスポンスに指定されたパスに指定されたデータが含まれていることを確認したい場合は、`assertJsonPath` メソッドを使用する必要があります:

===  "Pest"
```php
<?php

test('asserting a json path value', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJsonPath('team.owner.name', 'Darian');
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_a_json_paths_value(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJsonPath('team.owner.name', 'Darian');
    }
}
```

`assertJsonPath` メソッドは、アサーションが通過するかどうかを動的に決定するためにクロージャも受け入れます:

```php
$response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);
```

<a name="fluent-json-testing"></a>
### 流暢なJSONテスト

Laravelは、アプリケーションのJSONレスポンスを流暢にテストするための美しい方法も提供しています。始めるために、`assertJson` メソッドにクロージャを渡します。このクロージャは、アプリケーションによって返されたJSONに対してアサーションを行うために使用できる `Illuminate\Testing\Fluent\AssertableJson` のインスタンスで呼び出されます。`where` メソッドを使用してJSONの特定の属性に対してアサーションを行い、`missing` メソッドを使用して特定の属性がJSONから欠落していることをアサートすることができます:

===  "Pest"
```php
use Illuminate\Testing\Fluent\AssertableJson;

test('fluent json', function () {
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                 ->where('name', 'Victoria Faith')
                 ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                 ->whereNot('status', 'pending')
                 ->missing('password')
                 ->etc()
        );
});
```

===  "PHPUnit"
```php
use Illuminate\Testing\Fluent\AssertableJson;

/**
 * A basic functional test example.
 */
public function test_fluent_json(): void
{
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                 ->where('name', 'Victoria Faith')
                 ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                 ->whereNot('status', 'pending')
                 ->missing('password')
                 ->etc()
        );
}
```

#### `etc` メソッドの理解

上記の例では、アサーションチェーンの最後に `etc` メソッドを呼び出したことに気づいたかもしれません。このメソッドは、JSONオブジェクトに他の属性が存在する可能性があることをLaravelに通知します。`etc` メソッドが使用されない場合、アサーションを行わなかった他の属性がJSONオブジェクトに存在すると、テストは失敗します。

この動作の意図は、アサーションを明示的に行うか、`etc` メソッドを介して明示的に追加の属性を許可することで、意図せずにJSONレスポンスで機密情報を公開することを防ぐことです。

ただし、アサーションチェーンに `etc` メソッドを含めないことは、JSONオブジェクト内にネストされた配列に追加の属性が追加されないことを保証するものではないことに注意してください。`etc` メソッドは、`etc` メソッドが呼び出されたネストレベルにおいて、追加の属性が存在しないことを保証するだけです。

<a name="asserting-json-attribute-presence-and-absence"></a>
#### 属性の存在 / 不在のアサーション

属性の存在または不在をアサートするには、`has` および `missing` メソッドを使用できます:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('data')
         ->missing('message')
);
```

さらに、`hasAll` および `missingAll` メソッドを使用して、複数の属性の存在または不在を同時にアサートすることができます:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->hasAll(['status', 'data'])
         ->missingAll(['message', 'code'])
);
```

指定されたリストの属性の少なくとも1つが存在することを確認するには、`hasAny` メソッドを使用できます:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('status')
         ->hasAny('data', 'message', 'code')
);
```

<a name="asserting-against-json-collections"></a>
#### JSONコレクションに対するアサーション

多くの場合、ルートは複数のアイテム（例えば複数のユーザー）を含むJSONレスポンスを返します:

```php
Route::get('/users', function () {
    return User::all();
});
```

これらの状況では、流暢なJSONオブジェクトの `has` メソッドを使用して、レスポンスに含まれるユーザーに対してアサーションを行うことができます。例えば、JSONレスポンスに3人のユーザーが含まれていることをアサートしましょう。次に、コレクションの最初のユーザーに対して `first` メソッドを使用していくつかのアサーションを行います。`first` メソッドは、クロージャを受け取り、このクロージャはJSONコレクションの最初のオブジェクトに対してアサーションを行うために使用できる別のアサーション可能なJSON文字列を受け取ります:

    $response
        ->assertJson(fn (AssertableJson $json) =>
    $json->has(3)
         ->first(fn (AssertableJson $json) =>
            $json->where('id', 1)
                 ->where('name', 'Victoria Faith')
                 ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                 ->missing('password')
                 ->etc()
         )
);

<a name="scoping-json-collection-assertions"></a>
#### JSONコレクションのアサーションのスコープ

アプリケーションのルートが名前付きキーに割り当てられたJSONコレクションを返す場合があります。

```php
Route::get('/users', function () {
    return [
        'meta' => [...],
        'users' => User::all(),
    ];
});
```

これらのルートをテストする場合、`has`メソッドを使用してコレクション内のアイテム数に対してアサーションを行うことができます。さらに、`has`メソッドを使用してアサーションのチェーンをスコープすることもできます。

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
             ->has('users', 3)
             ->has('users.0', fn (AssertableJson $json) =>
                $json->where('id', 1)
                     ->where('name', 'Victoria Faith')
                     ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                     ->missing('password')
                     ->etc()
             )
    );
```

ただし、`users`コレクションに対して2つの別々の`has`メソッド呼び出しを行う代わりに、3番目のパラメータとしてクロージャを提供する単一の呼び出しを行うことができます。その場合、クロージャは自動的に呼び出され、コレクションの最初のアイテムにスコープされます。

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
             ->has('users', 3, fn (AssertableJson $json) =>
                $json->where('id', 1)
                     ->where('name', 'Victoria Faith')
                     ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                     ->missing('password')
                     ->etc()
             )
    );
```

<a name="asserting-json-types"></a>
#### JSONタイプのアサーション

JSONレスポンスのプロパティが特定のタイプであることをアサートしたい場合があります。`Illuminate\Testing\Fluent\AssertableJson`クラスは、`whereType`と`whereAllType`メソッドを提供してそれを行います。

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
         ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);
```

`|`文字を使用して複数のタイプを指定したり、`whereType`メソッドに2番目のパラメータとしてタイプの配列を渡すことができます。アサーションは、レスポンス値がリストされたタイプのいずれかである場合に成功します。

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('name', 'string|null')
         ->whereType('id', ['string', 'integer'])
);
```

`whereType`と`whereAllType`メソッドは、以下のタイプを認識します: `string`, `integer`, `double`, `boolean`, `array`, `null`。

<a name="testing-file-uploads"></a>
## ファイルアップロードのテスト

`Illuminate\Http\UploadedFile`クラスは、ダミーファイルや画像を生成するために使用できる`fake`メソッドを提供します。これは、`Storage`ファサードの`fake`メソッドと組み合わせることで、ファイルアップロードのテストが大幅に簡素化されます。たとえば、これら2つの機能を組み合わせて、アバターアップロードフォームを簡単にテストできます。

===  "Pest"
```php
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('avatars can be uploaded', function () {
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $response = $this->post('/avatar', [
        'avatar' => $file,
    ]);

    Storage::disk('avatars')->assertExists($file->hashName());
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_avatars_can_be_uploaded(): void
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.jpg');

        $response = $this->post('/avatar', [
            'avatar' => $file,
        ]);

        Storage::disk('avatars')->assertExists($file->hashName());
    }
}
```

指定されたファイルが存在しないことをアサートしたい場合は、`Storage`ファサードによって提供される`assertMissing`メソッドを使用できます。

```php
Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');
```

<a name="fake-file-customization"></a>
#### フェイクファイルのカスタマイズ

`UploadedFile`クラスが提供する`fake`メソッドを使用してファイルを作成する場合、画像の幅、高さ、およびサイズ（キロバイト単位）を指定して、アプリケーションのバリデーションルールをより良くテストすることができます。

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

画像以外のタイプのファイルを作成するには、`create`メソッドを使用できます。

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

必要に応じて、メソッドに`$mimeType`引数を渡して、ファイルによって返されるべきMIMEタイプを明示的に定義できます。

```php
UploadedFile::fake()->create(
    'document.pdf', $sizeInKilobytes, 'application/pdf'
);
```

<a name="testing-views"></a>
## ビューのテスト

Laravelでは、アプリケーションに模擬HTTPリクエストを行わずにビューをレンダリングすることもできます。これを行うには、テスト内で`view`メソッドを呼び出します。`view`メソッドは、ビュー名とオプションのデータ配列を受け取ります。メソッドは`Illuminate\Testing\TestView`のインスタンスを返し、ビューの内容に関するアサーションを便利に行うためのいくつかのメソッドを提供します。

===  "Pest"
```php
<?php

test('a welcome view can be rendered', function () {
    $view = $this->view('welcome', ['name' => 'Taylor']);

    $view->assertSee('Taylor');
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_a_welcome_view_can_be_rendered(): void
    {
        $view = $this->view('welcome', ['name' => 'Taylor']);

        $view->assertSee('Taylor');
    }
}
```

`TestView`クラスは、以下のアサーションメソッドを提供します: `assertSee`, `assertSeeInOrder`, `assertSeeText`, `assertSeeTextInOrder`, `assertDontSee`, `assertDontSeeText`。

必要に応じて、`TestView`インスタンスを文字列にキャストして、レンダリングされたビューの生の内容を取得できます。

```php
$contents = (string) $this->view('welcome');
```

<a name="sharing-errors"></a>
#### エラーの共有

一部のビューは、Laravelが提供する[グローバルエラーバッグ](validation.md#quick-displaying-the-validation-errors)で共有されるエラーに依存している場合があります。エラーバッグにエラーメッセージを追加するには、`withViewErrors`メソッドを使用できます。

```php
$view = $this->withViewErrors([
    'name' => ['Please provide a valid name.']
])->view('form');

$view->assertSee('Please provide a valid name.');
```

<a name="rendering-blade-and-components"></a>
### Bladeとコンポーネントのレンダリング

必要に応じて、`blade`メソッドを使用して生の[Blade](blade.md)文字列を評価してレンダリングできます。`view`メソッドと同様に、`blade`メソッドは`Illuminate\Testing\TestView`のインスタンスを返します。

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');
```

[Bladeコンポーネント](blade.md#components)を評価してレンダリングするには、`component`メソッドを使用できます。`component`メソッドは`Illuminate\Testing\TestComponent`のインスタンスを返します。

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');
```

<a name="available-assertions"></a>
## 利用可能なアサーション

<a name="response-assertions"></a>
### レスポンスのアサーション

Laravelの`Illuminate\Testing\TestResponse`クラスは、アプリケーションをテストする際に利用できるさまざまなカスタムアサーションメソッドを提供します。これらのアサーションは、`json`, `get`, `post`, `put`, `delete`テストメソッドによって返されるレスポンスでアクセスできます。

<style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertAccepted](#assert-accepted)
[assertBadRequest](#assert-bad-request)
[assertConflict](#assert-conflict)
[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertDownload](#assert-download)
[assertExactJson](#assert-exact-json)
[assertExactJsonStructure](#assert-exact-json-structure)
[assertForbidden](#assert-forbidden)
[assertFound](#assert-found)
[assertGone](#assert-gone)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertInternalServerError](#assert-internal-server-error)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonIsArray](#assert-json-is-array)
[assertJsonIsObject](#assert-json-is-object)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonMissingPath](#assert-json-missing-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertJsonValidationErrorFor](#assert-json-validation-error-for)
[assertLocation](#assert-location)
[assertMethodNotAllowed](#assert-method-not-allowed)
[assertMovedPermanently](#assert-moved-permanently)
[assertContent](#assert-content)
[assertNoContent](#assert-no-content)
[assertStreamedContent](#assert-streamed-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPaymentRequired](#assert-payment-required)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertRedirectContains](#assert-redirect-contains)
[assertRedirectToRoute](#assert-redirect-to-route)
[assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
[assertRequestTimeout](#assert-request-timeout)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
[assertServerError](#assert-server-error)
[assertServiceUnavailable](#assert-server-unavailable)
[assertSessionHas](#assert-session-has)
[assertSessionHasInput](#assert-session-has-input)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionHasNoErrors](#assert-session-has-no-errors)
[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertTooManyRequests](#assert-too-many-requests)
[assertUnauthorized](#assert-unauthorized)
[assertUnprocessable](#assert-unprocessable)
[assertUnsupportedMediaType](#assert-unsupported-media-type)
[assertValid](#assert-valid)
[assertInvalid](#assert-invalid)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-bad-request"></a>
#### assertBadRequest

レスポンスが不正なリクエスト（400）HTTPステータスコードを持っていることをアサートします:

    $response->assertBadRequest();

<a name="assert-accepted"></a>
#### assertAccepted

レスポンスが受け入れられた（202）HTTPステータスコードを持っていることをアサートします:

    $response->assertAccepted();

<a name="assert-conflict"></a>
#### assertConflict

レスポンスが競合（409）HTTPステータスコードを持っていることをアサートします:

    $response->assertConflict();

<a name="assert-cookie"></a>
#### assertCookie

レスポンスが指定されたクッキーを含んでいることをアサートします:

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

レスポンスが指定されたクッキーを含み、それが期限切れであることをアサートします:

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

レスポンスが指定されたクッキーを含み、それが期限切れでないことをアサートします:

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

レスポンスが指定されたクッキーを含んでいないことをアサートします:

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>
#### assertCreated

レスポンスが201 HTTPステータスコードを持っていることをアサートします:

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

指定された文字列がアプリケーションから返されたレスポンスに含まれていないことをアサートします。このアサーションは、第二引数に `false` を渡さない限り、指定された文字列を自動的にエスケープします:

    $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

指定された文字列がレスポンステキストに含まれていないことをアサートします。このアサーションは、第二引数に `false` を渡さない限り、指定された文字列を自動的にエスケープします。このメソッドは、アサーションを行う前にレスポンス内容を `strip_tags` PHP関数に渡します:

    $response->assertDontSeeText($value, $escaped = true);

<a name="assert-download"></a>
#### assertDownload

レスポンスが「ダウンロード」であることをアサートします。通常、これはレスポンスを返したルートが `Response::download` レスポンス、`BinaryFileResponse`、または `Storage::download` レスポンスを返したことを意味します:

    $response->assertDownload();

もし望むなら、ダウンロードされたファイルに指定されたファイル名が割り当てられていることをアサートすることもできます:

    $response->assertDownload('image.jpg');

<a name="assert-exact-json"></a>
#### assertExactJson

レスポンスが指定されたJSONデータと完全に一致することをアサートします:

    $response->assertExactJson(array $data);

<a name="assert-exact-json-structure"></a>
#### assertExactJsonStructure

レスポンスが指定されたJSON構造と完全に一致することをアサートします:

    $response->assertExactJsonStructure(array $data);

このメソッドは、[assertJsonStructure](#assert-json-structure) のより厳格なバリアントです。`assertJsonStructure` とは対照的に、このメソッドはレスポンスに期待されたJSON構造に明示的に含まれていないキーが含まれている場合に失敗します。

<a name="assert-forbidden"></a>
#### assertForbidden

レスポンスが禁止された（403）HTTPステータスコードを持っていることをアサートします:

    $response->assertForbidden();

<a name="assert-found"></a>
#### assertFound

レスポンスが見つかった（302）HTTPステータスコードを持っていることをアサートします:

    $response->assertFound();

<a name="assert-gone"></a>
#### assertGone

レスポンスが過ぎ去った（410）HTTPステータスコードを持っていることをアサートします:

    $response->assertGone();

<a name="assert-header"></a>
#### assertHeader

レスポンスに指定されたヘッダーと値が存在することをアサートします:

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

レスポンスに指定されたヘッダーが存在しないことをアサートします:

    $response->assertHeaderMissing($headerName);

<a name="assert-internal-server-error"></a>
#### assertInternalServerError

レスポンスが「内部サーバーエラー」（500）HTTPステータスコードを持っていることをアサートします:

    $response->assertInternalServerError();

<a name="assert-json"></a>
#### assertJson

レスポンスが指定されたJSONデータを含んでいることをアサートします:

    $response->assertJson(array $data, $strict = false);

`assertJson` メソッドは、レスポンスを配列に変換して、指定された配列がアプリケーションから返されたJSONレスポンス内に存在することを確認します。したがって、JSONレスポンスに他のプロパティがある場合でも、このテストは指定されたフラグメントが存在する限り合格します。

<a name="assert-json-count"></a>
#### assertJsonCount

レスポンスのJSONが指定されたキーに期待されるアイテム数の配列を持っていることをアサートします:

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

レスポンスがレスポンス内のどこかに指定されたJSONデータを含んでいることをアサートします:

    Route::get('/users', function () {
        return [
            'users' => [
                [
                    'name' => 'Taylor Otwell',
                ],
            ],
        ];
    });

    $response->assertJsonFragment(['name' => 'Taylor Otwell']);

<a name="assert-json-is-array"></a>
#### assertJsonIsArray

レスポンスのJSONが配列であることをアサートします:

    $response->assertJsonIsArray();

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

レスポンスのJSONがオブジェクトであることをアサートします:

    $response->assertJsonIsObject();

<a name="assert-json-missing"></a>
#### assertJsonMissing

レスポンスが指定されたJSONデータを含んでいないことをアサートします:

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

レスポンスが指定された正確なJSONデータを含んでいないことをアサートします:

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

レスポンスが指定されたキーに対するJSONバリデーションエラーを持っていないことをアサートします:

    $response->assertJsonMissingValidationErrors($keys);

> NOTE:  
> より一般的な [assertValid](#assert-valid) メソッドは、レスポンスがJSONとして返されたバリデーションエラーを持たず、セッションストレージにエラーがフラッシュされていないことをアサートするために使用できます。

<a name="assert-json-path"></a>
#### assertJsonPath

レスポンスが指定されたパスに指定されたデータを含んでいることをアサートします:

    $response->assertJsonPath($path, $expectedValue);

例えば、アプリケーションから次のJSONレスポンスが返された場合:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

`user` オブジェクトの `name` プロパティが指定された値と一致することを次のようにアサートできます:

    $response->assertJsonPath('user.name', 'Steve Schoger');

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

指定されたパスがレスポンスに含まれていないことをアサートします:

    $response->assertJsonMissingPath($path);

例えば、アプリケーションから以下のJSONレスポンスが返された場合:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

`user`オブジェクトの`email`プロパティが含まれていないことをアサートできます:

    $response->assertJsonMissingPath('user.email');

<a name="assert-json-structure"></a>
#### assertJsonStructure

レスポンスが指定されたJSON構造を持っていることをアサートします:

    $response->assertJsonStructure(array $structure);

例えば、アプリケーションから返されるJSONレスポンスに以下のデータが含まれている場合:

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

JSON構造が期待通りであることを以下のようにアサートできます:

    $response->assertJsonStructure([
        'user' => [
            'name',
        ]
    ]);

アプリケーションから返されるJSONレスポンスにオブジェクトの配列が含まれている場合もあります:

```json
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

この場合、配列内のすべてのオブジェクトの構造に対してアサートするために`*`文字を使用できます:

    $response->assertJsonStructure([
        'user' => [
            '*' => [
                 'name',
                 'age',
                 'location'
            ]
        ]
    ]);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

レスポンスが指定されたキーに対するJSONバリデーションエラーを持っていることをアサートします。このメソッドは、バリデーションエラーがセッションにフラッシュされるのではなく、JSON構造として返されるレスポンスに対して使用する必要があります:

    $response->assertJsonValidationErrors(array $data, $responseKey = 'errors');

> NOTE:  
> より汎用的な[assertInvalid](#assert-invalid)メソッドを使用して、レスポンスがJSONとして返されるバリデーションエラー**または**セッションストレージにフラッシュされたエラーをアサートできます。

<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

レスポンスが指定されたキーに対するJSONバリデーションエラーを持っていることをアサートします:

    $response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');

<a name="assert-method-not-allowed"></a>
#### assertMethodNotAllowed

レスポンスがメソッドが許可されていない（405）HTTPステータスコードを持っていることをアサートします:

    $response->assertMethodNotAllowed();

<a name="assert-moved-permanently"></a>
#### assertMovedPermanently

レスポンスが恒久的に移動した（301）HTTPステータスコードを持っていることをアサートします:

    $response->assertMovedPermanently();

<a name="assert-location"></a>
#### assertLocation

レスポンスが`Location`ヘッダーに指定されたURI値を持っていることをアサートします:

    $response->assertLocation($uri);

<a name="assert-content"></a>
#### assertContent

指定された文字列がレスポンス内容と一致することをアサートします:

    $response->assertContent($value);

<a name="assert-no-content"></a>
#### assertNoContent

レスポンスが指定されたHTTPステータスコードを持ち、内容がないことをアサートします:

    $response->assertNoContent($status = 204);

<a name="assert-streamed-content"></a>
#### assertStreamedContent

指定された文字列がストリーミングされたレスポンス内容と一致することをアサートします:

    $response->assertStreamedContent($value);

<a name="assert-not-found"></a>
#### assertNotFound

レスポンスが見つからない（404）HTTPステータスコードを持っていることをアサートします:

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

レスポンスが200 HTTPステータスコードを持っていることをアサートします:

    $response->assertOk();

<a name="assert-payment-required"></a>
#### assertPaymentRequired

レスポンスが支払いが必要（402）HTTPステータスコードを持っていることをアサートします:

    $response->assertPaymentRequired();

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

レスポンスが指定された暗号化されていないクッキーを含んでいることをアサートします:

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

レスポンスが指定されたURIにリダイレクトしていることをアサートします:

    $response->assertRedirect($uri = null);

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

レスポンスが指定された文字列を含むURIにリダイレクトしているかどうかをアサートします:

    $response->assertRedirectContains($string);

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

レスポンスが指定された[名前付きルート](routing.md#named-routes)にリダイレクトしていることをアサートします:

    $response->assertRedirectToRoute($name, $parameters = []);

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

レスポンスが指定された[署名付きルート](urls.md#signed-urls)にリダイレクトしていることをアサートします:

    $response->assertRedirectToSignedRoute($name = null, $parameters = []);

<a name="assert-request-timeout"></a>
#### assertRequestTimeout

レスポンスがリクエストタイムアウト（408）HTTPステータスコードを持っていることをアサートします:

    $response->assertRequestTimeout();

<a name="assert-see"></a>
#### assertSee

指定された文字列がレスポンス内に含まれていることをアサートします。このアサーションは、第二引数に`false`を渡さない限り、自動的に指定された文字列をエスケープします:

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

指定された文字列がレスポンス内に順番に含まれていることをアサートします。このアサーションは、第二引数に`false`を渡さない限り、自動的に指定された文字列をエスケープします:

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>
#### assertSeeText

指定された文字列がレスポンステキスト内に含まれていることをアサートします。このアサーションは、第二引数に`false`を渡さない限り、自動的に指定された文字列をエスケープします。レスポンス内容は、アサーションが行われる前に`strip_tags` PHP関数に渡されます:

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

指定された文字列がレスポンステキスト内に順番に含まれていることをアサートします。このアサーションは、第二引数に`false`を渡さない限り、自動的に指定された文字列をエスケープします。レスポンス内容は、アサーションが行われる前に`strip_tags` PHP関数に渡されます:

    $response->assertSeeTextInOrder(array $values, $escaped = true);

<a name="assert-server-error"></a>
#### assertServerError

レスポンスがサーバーエラー（>= 500, < 600）HTTPステータスコードを持っていることをアサートします:

    $response->assertServerError();

<a name="assert-server-unavailable"></a>
#### assertServiceUnavailable

レスポンスが「サービス利用不可」（503）HTTPステータスコードを持っていることをアサートします:

    $response->assertServiceUnavailable();

<a name="assert-session-has"></a>
#### assertSessionHas

セッションに指定されたデータが含まれていることをアサートします:

    $response->assertSessionHas($key, $value = null);

必要に応じて、`assertSessionHas`メソッドの第二引数にクロージャを提供できます。クロージャが`true`を返す場合、アサーションはパスします:

    $response->assertSessionHas($key, function (User $value) {
        return $value->name === 'Taylor Otwell';
    });

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

セッションに[フラッシュされた入力配列](responses.md#redirecting-with-flashed-session-data)に指定された値が含まれていることをアサートします:

    $response->assertSessionHasInput($key, $value = null);

必要に応じて、`assertSessionHasInput`メソッドの第二引数にクロージャを提供できます。クロージャが`true`を返す場合、アサーションはパスします:

    use Illuminate\Support\Facades\Crypt;

    $response->assertSessionHasInput($key, function (string $value) {
        return Crypt::decryptString($value) === 'secret';
    });

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

セッションに指定されたキー/値ペアの配列が含まれていることをアサートします:

    $response->assertSessionHasAll(array $data);

例えば、アプリケーションのセッションに`name`と`status`のキーが含まれている場合、以下のように両方が存在し、指定された値を持っていることをアサートできます:

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

セッションに指定された`$keys`に対するエラーが含まれていることをアサートします。`$keys`が連想配列の場合、セッションに各フィールド（キー）に対する特定のエラーメッセージ（値）が含まれていることをアサートします。このメソッドは、バリデーションエラーがJSON構造として返されるのではなく、セッションにフラッシュされるルートをテストする場合に使用する必要があります:

    $response->assertSessionHasErrors(
        array $keys = [], $format = null, $errorBag = 'default'
    );

例えば、`name`と`email`フィールドにバリデーションエラーメッセージがセッションにフラッシュされたことをアサートするには、以下のように`assertSessionHasErrors`メソッドを呼び出します:

    $response->assertSessionHasErrors(['name', 'email']);

または、特定のフィールドが特定のバリデーションエラーメッセージを持っていることをアサートできます:

    $response->assertSessionHasErrors([
        'name' => 'The given name was invalid.'
    ]);

> NOTE:  
> より汎用的な[assertInvalid](#assert-invalid)メソッドを使用して、レスポンスがJSONとして返されるバリデーションエラー**または**セッションストレージにフラッシュされたエラーをアサートできます。

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

セッションに指定された`$keys`に対するエラーが特定の[エラーバッグ](validation.md#named-error-bags)内に含まれていることをアサートします。`$keys`が連想配列の場合、セッションに各フィールド（キー）に対する特定のエラーメッセージ（値）がエラーバッグ内に含まれていることをアサートします:

    $response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

セッションにバリデーションエラーがないことをアサートする:

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

セッションに指定されたキーのバリデーションエラーがないことをアサートする:

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

> NOTE:  
> より汎用的な[assertValid](#assert-valid)メソッドは、レスポンスにJSONとして返されたバリデーションエラーがないこと、およびセッションストレージにエラーがフラッシュされていないことをアサートするために使用できます。

<a name="assert-session-missing"></a>
#### assertSessionMissing

セッションに指定されたキーが含まれていないことをアサートする:

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

レスポンスが指定されたHTTPステータスコードを持つことをアサートする:

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

レスポンスが成功（>= 200 かつ < 300）のHTTPステータスコードを持つことをアサートする:

    $response->assertSuccessful();

<a name="assert-too-many-requests"></a>
#### assertTooManyRequests

レスポンスがリクエスト過多（429）のHTTPステータスコードを持つことをアサートする:

    $response->assertTooManyRequests();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

レスポンスが認証されていない（401）のHTTPステータスコードを持つことをアサートする:

    $response->assertUnauthorized();

<a name="assert-unprocessable"></a>
#### assertUnprocessable

レスポンスが処理不能なエンティティ（422）のHTTPステータスコードを持つことをアサートする:

    $response->assertUnprocessable();

<a name="assert-unsupported-media-type"></a>
#### assertUnsupportedMediaType

レスポンスがサポートされていないメディアタイプ（415）のHTTPステータスコードを持つことをアサートする:

    $response->assertUnsupportedMediaType();

<a name="assert-valid"></a>
#### assertValid

レスポンスに指定されたキーのバリデーションエラーがないことをアサートする。このメソッドは、バリデーションエラーがJSON構造として返されるレスポンスや、バリデーションエラーがセッションにフラッシュされたレスポンスに対してアサートするために使用できます:

    // バリデーションエラーが存在しないことをアサートする...
    $response->assertValid();

    // 指定されたキーにバリデーションエラーがないことをアサートする...
    $response->assertValid(['name', 'email']);

<a name="assert-invalid"></a>
#### assertInvalid

レスポンスに指定されたキーのバリデーションエラーがあることをアサートする。このメソッドは、バリデーションエラーがJSON構造として返されるレスポンスや、バリデーションエラーがセッションにフラッシュされたレスポンスに対してアサートするために使用できます:

    $response->assertInvalid(['name', 'email']);

また、特定のキーが特定のバリデーションエラーメッセージを持つことをアサートすることもできます。その際、メッセージ全体またはメッセージの一部を提供できます:

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);

<a name="assert-view-has"></a>
#### assertViewHas

レスポンスビューに指定されたデータが含まれていることをアサートする:

    $response->assertViewHas($key, $value = null);

`assertViewHas`メソッドの第2引数としてクロージャを渡すと、特定のビューデータを検査してアサーションを行うことができます:

    $response->assertViewHas('user', function (User $user) {
        return $user->name === 'Taylor';
    });

さらに、ビューデータはレスポンスの配列変数としてアクセスでき、便利に検査できます:

===  "Pest"
```php
expect($response['name'])->toBe('Taylor');
```

===  "PHPUnit"
```php
$this->assertEquals('Taylor', $response['name']);
```

<a name="assert-view-has-all"></a>
#### assertViewHasAll

レスポンスビューに指定されたリストのデータがあることをアサートする:

    $response->assertViewHasAll(array $data);

このメソッドは、ビューが指定されたキーに一致するデータを単に含んでいることをアサートするために使用できます:

    $response->assertViewHasAll([
        'name',
        'email',
    ]);

または、ビューデータが存在し、特定の値を持つことをアサートすることもできます:

    $response->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

<a name="assert-view-is"></a>
#### assertViewIs

指定されたビューがルートによって返されたことをアサートする:

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

アプリケーションのレスポンスで返されたビューに指定されたデータキーが利用できないことをアサートする:

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### 認証アサーション

Laravelは、アプリケーションの機能テスト内で利用できるさまざまな認証関連のアサーションも提供しています。これらのメソッドは、`get`や`post`などのメソッドによって返される`Illuminate\Testing\TestResponse`インスタンスではなく、テストクラス自体に対して呼び出されることに注意してください。

<a name="assert-authenticated"></a>
#### assertAuthenticated

ユーザーが認証されていることをアサートする:

    $this->assertAuthenticated($guard = null);

<a name="assert-guest"></a>
#### assertGuest

ユーザーが認証されていないことをアサートする:

    $this->assertGuest($guard = null);

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

特定のユーザーが認証されていることをアサートする:

    $this->assertAuthenticatedAs($user, $guard = null);

<a name="validation-assertions"></a>
## バリデーションアサーション

Laravelは、リクエストで提供されたデータが有効か無効かを確認するために使用できる2つの主要なバリデーション関連のアサーションを提供しています。

<a name="validation-assert-valid"></a>
#### assertValid

レスポンスに指定されたキーのバリデーションエラーがないことをアサートする。このメソッドは、バリデーションエラーがJSON構造として返されるレスポンスや、バリデーションエラーがセッションにフラッシュされたレスポンスに対してアサートするために使用できます:

    // バリデーションエラーが存在しないことをアサートする...
    $response->assertValid();

    // 指定されたキーにバリデーションエラーがないことをアサートする...
    $response->assertValid(['name', 'email']);

<a name="validation-assert-invalid"></a>
#### assertInvalid

レスポンスに指定されたキーのバリデーションエラーがあることをアサートする。このメソッドは、バリデーションエラーがJSON構造として返されるレスポンスや、バリデーションエラーがセッションにフラッシュされたレスポンスに対してアサートするために使用できます:

    $response->assertInvalid(['name', 'email']);

また、特定のキーが特定のバリデーションエラーメッセージを持つことをアサートすることもできます。その際、メッセージ全体またはメッセージの一部を提供できます:

    $response->assertInvalid([
        'name' => 'The name field is required.',
        'email' => 'valid email address',
    ]);
