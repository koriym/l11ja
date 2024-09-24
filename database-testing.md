# データベーステスト

- [イントロダクション](#introduction)
    - [各テスト後のデータベースリセット](#resetting-the-database-after-each-test)
- [モデルファクトリ](#model-factories)
- [シーダーの実行](#running-seeders)
- [利用可能なアサーション](#available-assertions)

<a name="introduction"></a>
## イントロダクション

Laravelは、データベース駆動のアプリケーションを簡単にテストできるように、さまざまな便利なツールとアサーションを提供しています。さらに、Laravelのモデルファクトリとシーダーを使用すると、アプリケーションのEloquentモデルとリレーションを使用して、テストデータベースレコードを簡単に作成できます。これらの強力な機能については、以下のドキュメントで説明します。

<a name="resetting-the-database-after-each-test"></a>
### 各テスト後のデータベースリセット

さらに進む前に、各テスト後にデータベースをリセットする方法について説明しましょう。これにより、前のテストのデータが後続のテストに干渉しないようになります。Laravelに含まれる`Illuminate\Foundation\Testing\RefreshDatabase`トレイトが、これを処理します。テストクラスでトレイトを使用するだけです：

===  "Pest"
    ```php
    <?php
    
    use Illuminate\Foundation\Testing\RefreshDatabase;
    
    uses(RefreshDatabase::class);
    
    test('basic example', function () {
        $response = $this->get('/');
    
        // ...
    });
    ```

===  "PHPUnit"

    ```php
    <?php
    
    namespace Tests\Feature;
    
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;
    
    class ExampleTest extends TestCase
    {
        use RefreshDatabase;
    
        /**
         * 基本的な機能テストの例。
         */
        public function test_basic_example(): void
        {
            $response = $this->get('/');
    
            // ...
        }
    }
    ```

`Illuminate\Foundation\Testing\RefreshDatabase`トレイトは、スキーマが最新の場合、データベースをマイグレートしません。代わりに、データベーストランザクション内でテストを実行するだけです。したがって、このトレイトを使用しないテストケースによってデータベースに追加されたレコードは、データベースに残る可能性があります。

データベースを完全にリセットしたい場合は、代わりに`Illuminate\Foundation\Testing\DatabaseMigrations`または`Illuminate\Foundation\Testing\DatabaseTruncation`トレイトを使用できます。ただし、これらのオプションは`RefreshDatabase`トレイトよりも大幅に遅くなります。

<a name="model-factories"></a>
## モデルファクトリ

テストを行う際、テストを実行する前にデータベースにいくつかのレコードを挿入する必要があるかもしれません。このテストデータを作成する際に、各列の値を手動で指定する代わりに、Laravelでは[モデルファクトリ](eloquent-factories.md)を使用して、各[Eloquentモデル](eloquent.md)のデフォルト属性のセットを定義できます。

モデルファクトリの作成と使用方法について詳しくは、完全な[モデルファクトリのドキュメント](eloquent-factories.md)を参照してください。モデルファクトリを定義したら、テスト内でファクトリを使用してモデルを作成できます：

===  "Pest"

    ```php
    use App\Models\User;
    
    test('models can be instantiated', function () {
        $user = User::factory()->create();
    
        // ...
    });
    ```

===  "PHPUnit"

    ```php
    use App\Models\User;
    
    public function test_models_can_be_instantiated(): void
    {
        $user = User::factory()->create();
    
        // ...
    }
    ```

<a name="running-seeders"></a>
## シーダーの実行

機能テスト中に[データベースシーダー](seeding.md)を使用してデータベースをデータで埋めたい場合は、`seed`メソッドを呼び出すことができます。デフォルトでは、`seed`メソッドは`DatabaseSeeder`を実行し、これにより他のすべてのシーダーが実行されます。または、特定のシーダークラス名を`seed`メソッドに渡すこともできます：

===  "Pest"

    ```php
    <?php
    
    use Database\Seeders\OrderStatusSeeder;
    use Database\Seeders\TransactionStatusSeeder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    
    uses(RefreshDatabase::class);
    
    test('orders can be created', function () {
        // DatabaseSeederを実行...
        $this->seed();
    
        // 特定のシーダーを実行...
        $this->seed(OrderStatusSeeder::class);
    
        // ...
    
        // 特定のシーダーの配列を実行...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    });
    ```

===  "PHPUnit"

    ```php
    <?php
    
    namespace Tests\Feature;
    
    use Database\Seeders\OrderStatusSeeder;
    use Database\Seeders\TransactionStatusSeeder;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;
    
    class ExampleTest extends TestCase
    {
        use RefreshDatabase;
    
        /**
         * 新しい注文の作成テスト。
         */
        public function test_orders_can_be_created(): void
        {
            // DatabaseSeederを実行...
            $this->seed();
    
            // 特定のシーダーを実行...
            $this->seed(OrderStatusSeeder::class);
    
            // ...
    
            // 特定のシーダーの配列を実行...
            $this->seed([
                OrderStatusSeeder::class,
                TransactionStatusSeeder::class,
                // ...
            ]);
        }
    }
    ```

または、`RefreshDatabase`トレイトを使用する各テストの前に、Laravelにデータベースを自動的にシードするように指示できます。これは、ベーステストクラスに`$seed`プロパティを定義することで実現できます：

    <?php

    namespace Tests;

    use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

    abstract class TestCase extends BaseTestCase
    {
        /**
         * 各テストの前にデフォルトのシーダーを実行するかどうかを示します。
         *
         * @var bool
         */
        protected $seed = true;
    }

`$seed`プロパティが`true`の場合、`RefreshDatabase`トレイトを使用する各テストの前に`Database\Seeders\DatabaseSeeder`クラスが実行されます。ただし、テストクラスに`$seeder`プロパティを定義することで、特定のシーダーを実行するように指定できます：

    use Database\Seeders\OrderStatusSeeder;

    /**
     * 各テストの前に実行する特定のシーダー。
     *
     * @var string
     */
    protected $seeder = OrderStatusSeeder::class;

<a name="available-assertions"></a>
## 利用可能なアサーション

Laravelは、[Pest](https://pestphp.com)または[PHPUnit](https://phpunit.de)の機能テスト用にいくつかのデータベースアサーションを提供しています。以下では、これらの各アサーションについて説明します。

<a name="assert-database-count"></a>
#### assertDatabaseCount

データベース内のテーブルに指定された数のレコードが含まれていることをアサートします：

    $this->assertDatabaseCount('users', 5);

<a name="assert-database-has"></a>
#### assertDatabaseHas

データベース内のテーブルに、指定されたキー/値のクエリ制約に一致するレコードが含まれていることをアサートします：

    $this->assertDatabaseHas('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

データベース内のテーブルに、指定されたキー/値のクエリ制約に一致するレコードが含まれていないことをアサートします：

    $this->assertDatabaseMissing('users', [
        'email' => 'sally@example.com',
    ]);

<a name="assert-deleted"></a>
#### assertSoftDeleted

`assertSoftDeleted`メソッドは、指定されたEloquentモデルが「ソフトデリート」されたことをアサートするために使用できます：

    $this->assertSoftDeleted($user);

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

`assertNotSoftDeleted`メソッドは、指定されたEloquentモデルが「ソフトデリート」されていないことをアサートするために使用できます：

    $this->assertNotSoftDeleted($user);

<a name="assert-model-exists"></a>
#### assertModelExists

指定されたモデルがデータベースに存在することをアサートします：

    use App\Models\User;

    $user = User::factory()->create();

    $this->assertModelExists($user);

<a name="assert-model-missing"></a>
#### assertModelMissing

指定されたモデルがデータベースに存在しないことをアサートします：

    use App\Models\User;

    $user = User::factory()->create();

    $user->delete();

    $this->assertModelMissing($user);

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

`expectsDatabaseQueryCount`メソッドは、テストの開始時に呼び出され、テスト中に実行されるデータベースクエリの総数を指定できます。実際のクエリ数がこの期待値と一致しない場合、テストは失敗します：

    $this->expectsDatabaseQueryCount(5);

    // テスト...

