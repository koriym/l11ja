# データベース: はじめに

- [導入](#introduction)
    - [設定](#configuration)
    - [読み取りと書き込みの接続](#read-and-write-connections)
- [SQLクエリの実行](#running-queries)
    - [複数のデータベース接続の使用](#using-multiple-database-connections)
    - [クエリイベントのリッスン](#listening-for-query-events)
    - [累積クエリ時間の監視](#monitoring-cumulative-query-time)
- [データベーストランザクション](#database-transactions)
- [データベースCLIへの接続](#connecting-to-the-database-cli)
- [データベースの検査](#inspecting-your-databases)
- [データベースの監視](#monitoring-your-databases)

<a name="introduction"></a>
## 導入

ほとんどの現代のWebアプリケーションはデータベースと連携します。Laravelは、生のSQL、[流れるようなクエリビルダ](queries.md)、および[Eloquent ORM](eloquent.md)を使用して、サポートされているさまざまなデータベースとの連携を非常に簡単にします。現在、Laravelは以下の5つのデータベースに対して公式サポートを提供しています。

<div class="content-list" markdown="1">

- MariaDB 10.3+ ([バージョンポリシー](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([バージョンポリシー](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 10.0+ ([バージョンポリシー](https://www.postgresql.org/support/versioning/))
- SQLite 3.26.0+
- SQL Server 2017+ ([バージョンポリシー](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

<a name="configuration"></a>
### 設定

Laravelのデータベースサービスの設定は、アプリケーションの`config/database.php`設定ファイルにあります。このファイルでは、すべてのデータベース接続を定義し、デフォルトで使用する接続を指定できます。このファイル内のほとんどの設定オプションは、アプリケーションの環境変数の値によって制御されます。Laravelがサポートするほとんどのデータベースシステムの例がこのファイルに提供されています。

デフォルトでは、Laravelのサンプル[環境設定](configuration.md#environment-configuration)は、[Laravel Sail](sail.md)で使用する準備ができています。これは、ローカルマシン上でLaravelアプリケーションを開発するためのDocker設定です。ただし、必要に応じてローカルデータベースの設定を自由に変更できます。

<a name="sqlite-configuration"></a>
#### SQLite設定

SQLiteデータベースは、ファイルシステム上の単一のファイルに含まれています。ターミナルで`touch`コマンドを使用して新しいSQLiteデータベースを作成できます: `touch database/database.sqlite`。データベースが作成されたら、環境変数を設定してこのデータベースを指すようにすることができます。データベースへの絶対パスを`DB_DATABASE`環境変数に配置します:

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

デフォルトでは、SQLite接続の外部キー制約が有効になっています。無効にしたい場合は、`DB_FOREIGN_KEYS`環境変数を`false`に設定してください:

```ini
DB_FOREIGN_KEYS=false
```

> NOTE:  
> [Laravelインストーラー](installation.md#creating-a-laravel-project)を使用してLaravelアプリケーションを作成し、SQLiteをデータベースとして選択した場合、Laravelは自動的に`database/database.sqlite`ファイルを作成し、デフォルトの[データベースマイグレーション](migrations.md)を実行します。

<a name="mssql-configuration"></a>
#### Microsoft SQL Server設定

Microsoft SQL Serverデータベースを使用するには、`sqlsrv`および`pdo_sqlsrv` PHP拡張機能と、それらが必要とする依存関係（Microsoft SQL ODBCドライバなど）がインストールされていることを確認する必要があります。

<a name="configuration-using-urls"></a>
#### URLを使用した設定

通常、データベース接続は`host`、`database`、`username`、`password`などの複数の設定値を使用して設定されます。これらの各設定値には、対応する環境変数があります。これは、本番サーバーでデータベース接続情報を設定する場合、複数の環境変数を管理する必要があることを意味します。

AWSやHerokuなどの一部の管理されたデータベースプロバイダは、データベースのすべての接続情報を単一の文字列に含むデータベース「URL」を提供します。データベースURLの例は次のようになります:

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

これらのURLは通常、次の標準スキーマ規則に従います:

```html
driver://username:password@host:port/database?options
```

利便性のために、LaravelはこれらのURLを複数の設定オプションでデータベースを設定する代わりに使用することをサポートしています。`url`（または対応する`DB_URL`環境変数）設定オプションが存在する場合、データベース接続と資格情報の情報を抽出するために使用されます。

<a name="read-and-write-connections"></a>
### 読み取りと書き込みの接続

SELECTステートメントには一つのデータベース接続を使用し、INSERT、UPDATE、DELETEステートメントには別のデータベース接続を使用したい場合があります。Laravelはこれを簡単にし、生のクエリ、クエリビルダ、またはEloquent ORMを使用しているかどうかに関係なく、適切な接続が常に使用されます。

読み取り/書き込み接続を設定する方法を見てみましょう。この例を見てください:

    'mysql' => [
        'read' => [
            'host' => [
                '192.168.1.1',
                '196.168.1.2',
            ],
        ],
        'write' => [
            'host' => [
                '196.168.1.3',
            ],
        ],
        'sticky' => true,

        'database' => env('DB_DATABASE', 'laravel'),
        'username' => env('DB_USERNAME', 'root'),
        'password' => env('DB_PASSWORD', ''),
        'unix_socket' => env('DB_SOCKET', ''),
        'charset' => env('DB_CHARSET', 'utf8mb4'),
        'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
        'prefix' => '',
        'prefix_indexes' => true,
        'strict' => true,
        'engine' => null,
        'options' => extension_loaded('pdo_mysql') ? array_filter([
            PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
        ]) : [],
    ],

設定配列に3つのキーが追加されていることに注意してください: `read`、`write`、および`sticky`。`read`と`write`キーには、`host`というキーを含む配列値があります。`read`と`write`接続の残りのデータベースオプションは、メインの`mysql`設定配列からマージされます。

メインの`mysql`配列から値を上書きしたい場合にのみ、`read`と`write`配列にアイテムを配置する必要があります。したがって、この場合、`192.168.1.1`が「読み取り」接続のホストとして使用され、`192.168.1.3`が「書き込み」接続のホストとして使用されます。メインの`mysql`配列内のデータベース資格情報、プレフィックス、文字セット、およびその他のすべてのオプションは、両方の接続で共有されます。`host`設定配列に複数の値が存在する場合、データベースホストは各リクエストに対してランダムに選択されます。

<a name="the-sticky-option"></a>
#### `sticky`オプション

`sticky`オプションは、*オプションの*値で、現在のリクエストサイクル中にデータベースに書き込まれたレコードをすぐに読み取ることを可能にするために使用できます。`sticky`オプションが有効になっていて、現在のリクエストサイクル中にデータベースに対して「書き込み」操作が実行された場合、それ以降の「読み取り」操作は「書き込み」接続を使用します。これにより、リクエストサイクル中に書き込まれたデータを同じリクエスト中にすぐにデータベースから読み戻すことができます。これがアプリケーションにとって望ましい動作であるかどうかは、あなた次第です。

<a name="running-queries"></a>
## SQLクエリの実行

データベース接続を設定したら、`DB`ファサードを使用してクエリを実行できます。`DB`ファサードは、各タイプのクエリに対して`select`、`update`、`insert`、`delete`、および`statement`メソッドを提供します。

<a name="running-a-select-query"></a>
#### SELECTクエリの実行

基本的なSELECTクエリを実行するには、`DB`ファサードの`select`メソッドを使用できます:

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * アプリケーションのすべてのユーザーのリストを表示します。
         */
        public function index(): View
        {
            $users = DB::select('select * from users where active = ?', [1]);

            return view('user.index', ['users' => $users]);
        }
    }

`select`メソッドに渡される最初の引数はSQLクエリで、2番目の引数はクエリにバインドする必要があるパラメータバインディングです。通常、これらは`where`句の制約の値です。パラメータバインディングはSQLインジェクションから保護します。

`select`メソッドは常に結果の`array`を返します。配列内の各結果は、データベースのレコードを表すPHPの`stdClass`オブジェクトになります:

    use Illuminate\Support\Facades\DB;

    $users = DB::select('select * from users');

    foreach ($users as $user) {
        echo $user->name;
    }

<a name="selecting-scalar-values"></a>
#### スカラー値の選択

データベースクエリが単一のスカラー値を返す場合、レコードオブジェクトからクエリのスカラー結果を取得する必要がないように、Laravelでは`scalar`メソッドを使用してこの値を直接取得できます:

    $burgers = DB::scalar(
        "select count(case when food = 'burger' then 1 end) as burgers from menu"
    );

<a name="selecting-multiple-result-sets"></a>
#### 複数の結果セットの選択

アプリケーションが複数の結果セットを返すストアドプロシージャを呼び出す場合、`selectResultSets`メソッドを使用してストアドプロシージャによって返されるすべての結果セットを取得できます。

    [$options, $notifications] = DB::selectResultSets(
        "CALL get_user_options_and_notifications(?)", $request->user()->id
    );

<a name="using-named-bindings"></a>
#### 名前付きバインディングの使用

パラメータバインディングを表すために `?` を使用する代わりに、名前付きバインディングを使用してクエリを実行できます。

    $results = DB::select('select * from users where id = :id', ['id' => 1]);

<a name="running-an-insert-statement"></a>
#### Insert文の実行

`insert` 文を実行するには、`DB` ファサードの `insert` メソッドを使用します。`select` と同様に、このメソッドは最初の引数に SQL クエリを、2番目の引数にバインディングを受け取ります。

    use Illuminate\Support\Facades\DB;

    DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);

<a name="running-an-update-statement"></a>
#### Update文の実行

データベース内の既存のレコードを更新するには、`update` メソッドを使用します。このメソッドは、ステートメントによって影響を受けた行数を返します。

    use Illuminate\Support\Facades\DB;

    $affected = DB::update(
        'update users set votes = 100 where name = ?',
        ['Anita']
    );

<a name="running-a-delete-statement"></a>
#### Delete文の実行

データベースからレコードを削除するには、`delete` メソッドを使用します。`update` と同様に、影響を受けた行数がメソッドによって返されます。

    use Illuminate\Support\Facades\DB;

    $deleted = DB::delete('delete from users');

<a name="running-a-general-statement"></a>
#### 一般的なステートメントの実行

値を返さないデータベースステートメントについては、`DB` ファサードの `statement` メソッドを使用できます。

    DB::statement('drop table users');

<a name="running-an-unprepared-statement"></a>
#### 未準備ステートメントの実行

値をバインドせずに SQL ステートメントを実行したい場合は、`DB` ファサードの `unprepared` メソッドを使用できます。

    DB::unprepared('update users set votes = 100 where name = "Dries"');

> WARNING:  
> 未準備ステートメントはパラメータをバインドしないため、SQL インジェクションに対して脆弱になる可能性があります。未準備ステートメント内でユーザーが制御する値を許可しないでください。

<a name="implicit-commits-in-transactions"></a>
#### 暗黙的なコミット

`DB` ファサードの `statement` および `unprepared` メソッドをトランザクション内で使用する場合、[暗黙的なコミット](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)を引き起こすステートメントに注意する必要があります。これらのステートメントは、データベースエンジンが間接的にトランザクション全体をコミットし、Laravel がデータベースのトランザクションレベルを認識できなくなります。データベーステーブルを作成するなどの例があります。

    DB::unprepared('create table a (col varchar(1) null)');

暗黙的なコミットを引き起こすすべてのステートメントのリストについては、MySQL マニュアルを参照してください。

<a name="using-multiple-database-connections"></a>
### 複数のデータベース接続の使用

アプリケーションが `config/database.php` 設定ファイルで複数の接続を定義している場合、`DB` ファサードが提供する `connection` メソッドを介して各接続にアクセスできます。`connection` メソッドに渡される接続名は、`config/database.php` 設定ファイルにリストされている接続のいずれか、または `config` ヘルパを使用して実行時に設定された接続に対応する必要があります。

    use Illuminate\Support\Facades\DB;

    $users = DB::connection('sqlite')->select(/* ... */);

接続インスタンスの `getPdo` メソッドを使用して、接続の生の基礎となる PDO インスタンスにアクセスできます。

    $pdo = DB::connection()->getPdo();

<a name="listening-for-query-events"></a>
### クエリイベントのリスニング

アプリケーションが実行する各 SQL クエリに対して呼び出されるクロージャを指定したい場合は、`DB` ファサードの `listen` メソッドを使用できます。このメソッドは、クエリのログ記録やデバッグに役立ちます。クエリリスナーのクロージャは、[サービスプロバイダ](providers.md)の `boot` メソッドに登録できます。

    <?php

    namespace App\Providers;

    use Illuminate\Database\Events\QueryExecuted;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            DB::listen(function (QueryExecuted $query) {
                // $query->sql;
                // $query->bindings;
                // $query->time;
                // $query->toRawSql();
            });
        }
    }

<a name="monitoring-cumulative-query-time"></a>
### 累積クエリ時間の監視

現代のウェブアプリケーションにおける一般的なパフォーマンスのボトルネックは、データベースのクエリに費やす時間の量です。幸いなことに、Laravel では単一のリクエスト中にデータベースのクエリに時間がかかりすぎる場合に、選択したクロージャまたはコールバックを呼び出すことができます。使用するには、クエリ時間のしきい値（ミリ秒単位）とクロージャを `whenQueryingForLongerThan` メソッドに提供します。このメソッドは、[サービスプロバイダ](providers.md)の `boot` メソッドで呼び出してください。

    <?php

    namespace App\Providers;

    use Illuminate\Database\Connection;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\Database\Events\QueryExecuted;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
                // 開発チームに通知...
            });
        }
    }

<a name="database-transactions"></a>
## データベーストランザクション

`DB` ファサードが提供する `transaction` メソッドを使用して、データベーストランザクション内で一連の操作を実行できます。トランザクションクロージャ内で例外がスローされた場合、トランザクションは自動的にロールバックされ、例外が再スローされます。クロージャが正常に実行された場合、トランザクションは自動的にコミットされます。`transaction` メソッドを使用する際に、手動でロールバックまたはコミットする必要はありません。

    use Illuminate\Support\Facades\DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    });

<a name="handling-deadlocks"></a>
#### デッドロックの処理

`transaction` メソッドは、デッドロックが発生した場合にトランザクションを再試行する回数を定義するオプションの2番目の引数を受け取ります。これらの試行が使い果たされると、例外がスローされます。

    use Illuminate\Support\Facades\DB;

    DB::transaction(function () {
        DB::update('update users set votes = 1');

        DB::delete('delete from posts');
    }, 5);

<a name="manually-using-transactions"></a>
#### 手動でのトランザクションの使用

トランザクションを手動で開始し、ロールバックとコミットを完全に制御したい場合は、`DB` ファサードが提供する `beginTransaction` メソッドを使用できます。

    use Illuminate\Support\Facades\DB;

    DB::beginTransaction();

トランザクションをロールバックするには、`rollBack` メソッドを使用します。

    DB::rollBack();

最後に、トランザクションをコミットするには、`commit` メソッドを使用します。

    DB::commit();

> NOTE:  
> `DB` ファサードのトランザクションメソッドは、[クエリビルダ](queries.md)と [Eloquent ORM](eloquent.md) の両方のトランザクションを制御します。

<a name="connecting-to-the-database-cli"></a>
## データベース CLI への接続

データベースの CLI に接続したい場合は、`db` Artisan コマンドを使用できます。

```shell
php artisan db
```

必要に応じて、デフォルトの接続ではないデータベース接続に接続するために、データベース接続名を指定できます。

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## データベースの検査

`db:show` および `db:table` Artisan コマンドを使用して、データベースとその関連テーブルに関する貴重な洞察を得ることができます。データベースの概要（サイズ、タイプ、オープン接続数、およびテーブルの要約）を見るには、`db:show` コマンドを使用できます。

```shell
php artisan db:show
```

検査するデータベース接続を指定するには、`--database` オプションを介してデータベース接続名をコマンドに渡します。

```shell
php artisan db:show --database=pgsql
```

コマンドの出力にテーブルの行数とデータベースビューの詳細を含めたい場合は、それぞれ `--counts` および `--views` オプションを指定できます。大規模なデータベースでは、行数とビューの詳細の取得に時間がかかる可能性があります。

```shell
php artisan db:show --counts --views
```

さらに、以下の `Schema` メソッドを使用してデータベースを検査できます。

    use Illuminate\Support\Facades\Schema;

    $tables = Schema::getTables();
    $views = Schema::getViews();
    $columns = Schema::getColumns('users');
    $indexes = Schema::getIndexes('users');
    $foreignKeys = Schema::getForeignKeys('users');

アプリケーションのデフォルト接続ではないデータベース接続を検査したい場合は、`connection` メソッドを使用できます。

    $columns = Schema::connection('sqlite')->getColumns('users');

<a name="table-overview"></a>
#### テーブルの概要
```

個々のテーブルの概要をデータベース内で取得したい場合は、`db:table` Artisanコマンドを実行することができます。このコマンドは、データベーステーブルの一般的な概要を提供します。これには、そのカラム、型、属性、キー、およびインデックスが含まれます。

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## データベースの監視

`db:monitor` Artisanコマンドを使用すると、データベースが指定された数以上のオープン接続を管理している場合に、Laravelに`Illuminate\Database\Events\DatabaseBusy`イベントをディスパッチするよう指示できます。

まず、`db:monitor`コマンドを[毎分実行するようにスケジュール](scheduling.md)する必要があります。このコマンドは、監視したいデータベース接続設定の名前と、イベントをディスパッチする前に許容される最大オープン接続数を受け取ります。

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

このコマンドをスケジュールするだけでは、オープン接続数に関する通知がトリガーされるわけではありません。コマンドが閾値を超えるオープン接続数を持つデータベースに遭遇すると、`DatabaseBusy`イベントがディスパッチされます。このイベントをアプリケーションの`AppServiceProvider`内でリッスンし、通知を自分や開発チームに送信する必要があります。

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', 'dev@example.com')
                ->notify(new DatabaseApproachingMaxConnections(
                    $event->connectionName,
                    $event->connections
                ));
    });
}
```
