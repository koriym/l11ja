# 並行処理

- [はじめに](#introduction)
- [並行タスクの実行](#running-concurrent-tasks)
- [並行タスクの延期](#deferring-concurrent-tasks)

<a name="introduction"></a>
## はじめに

> WARNING:
> Laravelの`Concurrency`ファサードは現在ベータ版であり、コミュニティからのフィードバックを収集しています。

時には、互いに依存しないいくつかの遅いタスクを実行する必要があるかもしれません。多くの場合、これらのタスクを並行して実行することで、大幅なパフォーマンス向上を実現できます。Laravelの`Concurrency`ファサードは、クロージャを並行して実行するためのシンプルで便利なAPIを提供します。

<a name="how-it-works"></a>
#### 仕組み

Laravelは、与えられたクロージャをシリアライズし、それらをバックグラウンドで実行されるArtisan CLIコマンドにディスパッチすることで並行処理を実現します。このコマンドは、クロージャをアンシリアライズし、独自のPHPプロセス内で呼び出します。クロージャが呼び出された後、結果の値は親プロセスにシリアライズされて戻されます。

`Concurrency`ファサードは、`process`（デフォルト）、`fork`、`sync`の3つのドライバをサポートしています。

`fork`ドライバは、デフォルトの`process`ドライバよりもパフォーマンスが向上しますが、PHPのCLIコンテキスト内でのみ使用できます。これは、PHPがWebリクエスト中にフォークをサポートしていないためです。`fork`ドライバを使用する前に、`spatie/fork`パッケージをインストールする必要があります：

```bash
composer require spatie/fork
```

`sync`ドライバは、すべての並行処理を無効にして、与えられたクロージャを親プロセス内で順次実行したい場合、主にテスト時に有用です。

<a name="running-concurrent-tasks"></a>
## 並行タスクの実行

並行タスクを実行するには、`Concurrency`ファサードの`run`メソッドを呼び出します。`run`メソッドは、子PHPプロセスで同時に実行されるべきクロージャの配列を受け取ります：

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

特定のドライバを使用するには、`driver`メソッドを使用できます：

```php
$results = Concurrency::driver('fork')->run(...);
```

または、デフォルトの並行処理ドライバを変更したい場合は、`config:publish` Artisanコマンドを介して`concurrency`設定ファイルを公開し、ファイル内の`default`オプションを更新する必要があります：

```bash
php artisan config:publish concurrency
```

<a name="deferring-concurrent-tasks"></a>
## 並行タスクの延期

クロージャの配列を並行して実行したいが、それらのクロージャが返す結果に興味がない場合は、`defer`メソッドの使用を検討してください。`defer`メソッドが呼び出されると、与えられたクロージャはすぐに実行されません。代わりに、LaravelはHTTPレスポンスがユーザーに送信された後にクロージャを並行して実行します：

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```

