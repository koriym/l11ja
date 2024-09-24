# テスト: はじめに

- [イントロダクション](#introduction)
- [環境](#environment)
- [テストの作成](#creating-tests)
- [テストの実行](#running-tests)
    - [テストの並列実行](#running-tests-in-parallel)
    - [テストカバレッジのレポート](#reporting-test-coverage)
    - [テストのプロファイリング](#profiling-tests)

<a name="introduction"></a>
## イントロダクション

Laravelはテストを念頭に置いて構築されています。実際、[Pest](https://pestphp.com)と[PHPUnit](https://phpunit.de)によるテストのサポートはデフォルトで含まれており、アプリケーション用に`phpunit.xml`ファイルもすでに設定されています。また、フレームワークには、アプリケーションを表現豊かにテストできる便利なヘルパーメソッドも付属しています。

デフォルトでは、アプリケーションの`tests`ディレクトリには、`Feature`と`Unit`の2つのディレクトリが含まれています。ユニットテストは、コードの非常に小さく、孤立した部分に焦点を当てたテストです。実際、ほとんどのユニットテストは単一のメソッドに焦点を当てている可能性があります。"Unit"テストディレクトリ内のテストは、Laravelアプリケーションを起動しないため、アプリケーションのデータベースやその他のフレームワークサービスにアクセスできません。

機能テストは、コードのより大きな部分をテストすることができ、複数のオブジェクトがどのように相互作用するか、またはJSONエンドポイントへの完全なHTTPリクエストを含むこともできます。**一般的に、ほとんどのテストは機能テストであるべきです。これらのタイプのテストは、システム全体が意図したとおりに機能していることを最も確信を持たせてくれます。**

`Feature`と`Unit`テストディレクトリの両方に`ExampleTest.php`ファイルが用意されています。新しいLaravelアプリケーションをインストールした後、`vendor/bin/pest`、`vendor/bin/phpunit`、または`php artisan test`コマンドを実行してテストを実行します。

<a name="environment"></a>
## 環境

テストを実行すると、Laravelは自動的に[設定環境](configuration.md#environment-configuration)を`testing`に設定します。これは、`phpunit.xml`ファイルで定義された環境変数のためです。また、Laravelはセッションとキャッシュを`array`ドライバに自動的に設定し、テスト中にセッションやキャッシュデータが永続化されないようにします。

必要に応じて、他のテスト環境の設定値を自由に定義できます。`testing`環境変数は、アプリケーションの`phpunit.xml`ファイルで設定できますが、テストを実行する前に`config:clear` Artisanコマンドを使用して設定キャッシュをクリアすることを忘れないでください！

<a name="the-env-testing-environment-file"></a>
#### `.env.testing`環境ファイル

さらに、プロジェクトのルートに`.env.testing`ファイルを作成することもできます。このファイルは、PestとPHPUnitテストを実行するとき、または`--env=testing`オプションを使用してArtisanコマンドを実行するときに、`.env`ファイルの代わりに使用されます。

<a name="creating-tests"></a>
## テストの作成

新しいテストケースを作成するには、`make:test` Artisanコマンドを使用します。デフォルトでは、テストは`tests/Feature`ディレクトリに配置されます：

```shell
php artisan make:test UserTest
```

`tests/Unit`ディレクトリ内にテストを作成したい場合は、`make:test`コマンドを実行する際に`--unit`オプションを使用できます：

```shell
php artisan make:test UserTest --unit
```

> NOTE:  
> テストスタブは、[スタブの公開](artisan.md#stub-customization)を使用してカスタマイズできます。

テストが生成されたら、PestまたはPHPUnitを使用して通常どおりにテストを定義できます。テストを実行するには、ターミナルから`vendor/bin/pest`、`vendor/bin/phpunit`、または`php artisan test`コマンドを実行します：

===  "Pest"
```php
<?php

test('basic', function () {
    expect(true)->toBeTrue();
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 基本的なテスト例。
     */
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```

> WARNING:  
> テストクラス内で独自の`setUp` / `tearDown`メソッドを定義する場合、親クラスの`parent::setUp()` / `parent::tearDown()`メソッドをそれぞれ呼び出すようにしてください。通常、独自の`setUp`メソッドの先頭で`parent::setUp()`を、`tearDown`メソッドの末尾で`parent::tearDown()`を呼び出すべきです。

<a name="running-tests"></a>
## テストの実行

前述のように、テストを書いたら、`pest`または`phpunit`を使用して実行できます：

```shell tab=Pest
./vendor/bin/pest
```

```shell tab=PHPUnit
./vendor/bin/phpunit
```

`pest`または`phpunit`コマンドに加えて、`test` Artisanコマンドを使用してテストを実行することもできます。Artisanテストランナーは、開発とデバッグを容易にするために詳細なテストレポートを提供します：

```shell
php artisan test
```

`pest`または`phpunit`コマンドに渡すことができる引数は、Artisanの`test`コマンドにも渡すことができます：

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>
### テストの並列実行

デフォルトでは、LaravelとPest / PHPUnitはテストを単一のプロセス内で順次実行します。しかし、テストを複数のプロセスで同時に実行することで、テストの実行時間を大幅に短縮できます。まず、`brianium/paratest` Composerパッケージを"dev"依存関係としてインストールする必要があります。その後、`test` Artisanコマンドを実行する際に`--parallel`オプションを含めます：

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

デフォルトでは、Laravelはマシン上で利用可能なCPUコアの数と同じ数のプロセスを作成します。ただし、`--processes`オプションを使用してプロセスの数を調整できます：

```shell
php artisan test --parallel --processes=4
```

> WARNING:  
> テストを並列実行する場合、一部のPest / PHPUnitオプション（例：`--do-not-cache-result`）は利用できない場合があります。

<a name="parallel-testing-and-databases"></a>
#### 並列テストとデータベース

プライマリデータベース接続を設定している場合、Laravelは自動的にテストを実行する各並列プロセスのためにテストデータベースを作成および移行します。テストデータベースには、プロセスごとに一意のプロセストークンが付加されます。たとえば、2つの並列テストプロセスがある場合、Laravelは`your_db_test_1`と`your_db_test_2`のテストデータベースを作成して使用します。

デフォルトでは、テストデータベースは`test` Artisanコマンドの呼び出し間で保持されるため、後続の`test`呼び出しで再利用できます。ただし、`--recreate-databases`オプションを使用して再作成することもできます：

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### 並列テストフック

アプリケーションのテストで使用される特定のリソースを準備し、複数のテストプロセスで安全に使用できるようにする必要がある場合があります。

`ParallelTesting`ファサードを使用すると、プロセスまたはテストケースの`setUp`と`tearDown`で実行されるコードを指定できます。指定されたクロージャは、プロセストークンと現在のテストケースを含む`$token`と`$testCase`変数を受け取ります：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Artisan;
    use Illuminate\Support\Facades\ParallelTesting;
    use Illuminate\Support\ServiceProvider;
    use PHPUnit\Framework\TestCase;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            ParallelTesting::setUpProcess(function (int $token) {
                // ...
            });

            ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
                // ...
            });

            // テストデータベースが作成されたときに実行されます...
            ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
                Artisan::call('db:seed');
            });

            ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
                // ...
            });

            ParallelTesting::tearDownProcess(function (int $token) {
                // ...
            });
        }
    }

<a name="accessing-the-parallel-testing-token"></a>
#### 並列テストトークンへのアクセス

アプリケーションのテストコード内の他の場所から現在の並列プロセスの"トークン"にアクセスしたい場合は、`token`メソッドを使用できます。このトークンは、個々のテストプロセスの一意の文字列識別子であり、並列テストプロセス間でリソースを分割するために使用できます。たとえば、Laravelは自動的に各並列テストプロセスによって作成されたテストデータベースの末尾にこのトークンを付加します：

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>
### テストカバレッジのレポート

> WARNING:  
> この機能には[Xdebug](https://xdebug.org)または[PCOV](https://pecl.php.net/package/pcov)が必要です。

アプリケーションのテストを実行する際に、テストケースが実際にアプリケーションコードをカバーしているか、どれだけのアプリケーションコードがテスト実行時に使用されているかを確認したい場合があります。これを実現するには、`test`コマンドを呼び出す際に`--coverage`オプションを提供します：

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### 最小カバレッジ閾値の強制

`--min`オプションを使用して、アプリケーションの最小テストカバレッジ閾値を定義できます。この閾値を満たさない場合、テストスイートは失敗します：

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>
### テストのプロファイリング

Artisanテストランナーには、アプリケーションの最も遅いテストをリストアップする便利なメカニズムも含まれています。`test`コマンドに`--profile`オプションを付けて呼び出すと、10個の最も遅いテストが表示され、テストスイートの速度を向上させるためにどのテストを改善できるかを簡単に調査できます：

```shell
php artisan test --profile
```
