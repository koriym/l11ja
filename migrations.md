# データベース: マイグレーション

- [はじめに](#introduction)
- [マイグレーションの生成](#generating-migrations)
    - [マイグレーションの圧縮](#squashing-migrations)
- [マイグレーションの構造](#migration-structure)
- [マイグレーションの実行](#running-migrations)
    - [マイグレーションのロールバック](#rolling-back-migrations)
- [テーブル](#tables)
    - [テーブルの作成](#creating-tables)
    - [テーブルの更新](#updating-tables)
    - [テーブルのリネームと削除](#renaming-and-dropping-tables)
- [カラム](#columns)
    - [カラムの作成](#creating-columns)
    - [利用可能なカラムタイプ](#available-column-types)
    - [カラム修飾子](#column-modifiers)
    - [カラムの変更](#modifying-columns)
    - [カラムのリネーム](#renaming-columns)
    - [カラムの削除](#dropping-columns)
- [インデックス](#indexes)
    - [インデックスの作成](#creating-indexes)
    - [インデックスのリネーム](#renaming-indexes)
    - [インデックスの削除](#dropping-indexes)
    - [外部キー制約](#foreign-key-constraints)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

マイグレーションはデータベースのバージョン管理のようなもので、チームがアプリケーションのデータベーススキーマ定義を共有し、変更を加えることを可能にします。ソース管理から変更を取り込んだ後、チームメイトに手動でローカルデータベーススキーマにカラムを追加するように伝える必要があったことがあるなら、データベースマイグレーションが解決する問題に直面したことになります。

Laravelの`Schema` [ファサード](facades.md)は、Laravelがサポートするすべてのデータベースシステムに対して、テーブルの作成と操作をデータベースに依存しない形で提供します。通常、マイグレーションはこのファサードを使用して、データベーステーブルとカラムを作成および変更します。

<a name="generating-migrations"></a>
## マイグレーションの生成

データベースマイグレーションを生成するには、`make:migration` [Artisanコマンド](artisan.md)を使用します。新しいマイグレーションは`database/migrations`ディレクトリに配置されます。各マイグレーションファイル名にはタイムスタンプが含まれており、Laravelはこれを使用してマイグレーションの順序を決定します。

```shell
php artisan make:migration create_flights_table
```

Laravelはマイグレーション名を使用して、テーブル名とマイグレーションが新しいテーブルを作成するかどうかを推測しようとします。Laravelがマイグレーション名からテーブル名を推測できる場合、Laravelは生成されたマイグレーションファイルに指定されたテーブルを事前に入力します。それ以外の場合は、マイグレーションファイルでテーブルを手動で指定するだけです。

生成されたマイグレーションのカスタムパスを指定したい場合は、`make:migration`コマンドを実行する際に`--path`オプションを使用できます。指定されたパスは、アプリケーションのベースパスからの相対パスである必要があります。

> NOTE:  
> マイグレーションスタブは、[スタブの公開](artisan.md#stub-customization)を使用してカスタマイズできます。

<a name="squashing-migrations"></a>
### マイグレーションの圧縮

アプリケーションを構築するにつれて、時間の経過とともにマイグレーションが増えていく可能性があります。これにより、`database/migrations`ディレクトリが数百のマイグレーションで肥大化する可能性があります。必要に応じて、マイグレーションを1つのSQLファイルに「圧縮」することができます。開始するには、`schema:dump`コマンドを実行します。

```shell
php artisan schema:dump

# 現在のデータベーススキーマをダンプし、既存のすべてのマイグレーションを削除する...
php artisan schema:dump --prune
```

このコマンドを実行すると、Laravelはアプリケーションの`database/schema`ディレクトリに「スキーマ」ファイルを書き込みます。スキーマファイルの名前は、データベース接続に対応します。これで、データベースをマイグレーションしようとしても、他のマイグレーションが実行されていない場合、Laravelは最初に使用しているデータベース接続のスキーマファイル内のSQL文を実行します。スキーマファイルのSQL文が実行された後、Laravelはスキーマダンプの一部ではなかった残りのマイグレーションを実行します。

アプリケーションのテストが、ローカル開発中に通常使用するものとは異なるデータベース接続を使用している場合、そのデータベース接続を使用してスキーマファイルをダンプしていることを確認する必要があります。これにより、テストがデータベースを構築できるようになります。ローカル開発中に通常使用するデータベース接続をダンプした後に、これを行うことをお勧めします。

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

データベーススキーマファイルをソース管理にコミットして、チームの他の新しい開発者がアプリケーションの初期データベース構造をすばやく作成できるようにする必要があります。

> WARNING:  
> マイグレーションの圧縮は、MariaDB、MySQL、PostgreSQL、およびSQLiteデータベースでのみ利用可能であり、データベースのコマンドラインクライアントを利用します。

<a name="migration-structure"></a>
## マイグレーションの構造

マイグレーションクラスには、`up`と`down`の2つのメソッドが含まれています。`up`メソッドは、データベースに新しいテーブル、カラム、またはインデックスを追加するために使用され、`down`メソッドは`up`メソッドによって実行された操作を元に戻す必要があります。

これらのメソッドの両方で、Laravelのスキーマビルダを使用して、テーブルの作成と変更を表現的に行うことができます。`Schema`ビルダで利用可能なすべてのメソッドについては、[そのドキュメント](#creating-tables)を参照してください。例えば、次のマイグレーションは`flights`テーブルを作成します。

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * マイグレーションを実行する。
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * マイグレーションを元に戻す。
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### マイグレーション接続の設定

マイグレーションがアプリケーションのデフォルトのデータベース接続以外のデータベース接続と対話する場合、マイグレーションの`$connection`プロパティを設定する必要があります。

```php
/**
 * マイグレーションで使用するデータベース接続。
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * マイグレーションを実行する。
 */
public function up(): void
{
    // ...
}
```

<a name="running-migrations"></a>
## マイグレーションの実行

保留中のすべてのマイグレーションを実行するには、`migrate` Artisanコマンドを実行します。

```shell
php artisan migrate
```

これまでに実行されたマイグレーションを確認したい場合は、`migrate:status` Artisanコマンドを使用できます。

```shell
php artisan migrate:status
```

マイグレーションが実行されるSQL文を実際に実行せずに確認したい場合は、`migrate`コマンドに`--pretend`フラグを指定できます。

```shell
php artisan migrate --pretend
```

#### マイグレーション実行の分離

アプリケーションを複数のサーバーにデプロイし、デプロイプロセスの一部としてマイグレーションを実行する場合、2つのサーバーが同時にデータベースをマイグレーションしようとしないようにする必要があります。これを回避するには、`migrate`コマンドを呼び出す際に`isolated`オプションを使用できます。

`isolated`オプションが提供されると、Laravelはアプリケーションのキャッシュドライバを使用してアトミックロックを取得し、マイグレーションの実行を試みる前に保持します。そのロックが保持されている間に`migrate`コマンドを実行しようとする他のすべての試みは実行されません。ただし、コマンドは成功した終了ステータスコードで終了します。

```shell
php artisan migrate --isolated
```

> WARNING:  
> この機能を利用するには、アプリケーションが`memcached`、`redis`、`dynamodb`、`database`、`file`、または`array`キャッシュドライバをアプリケーションのデフォルトキャッシュドライバとして使用する必要があります。さらに、すべてのサーバーが同じ中央キャッシュサーバーと通信する必要があります。

<a name="forcing-migrations-to-run-in-production"></a>
#### 本番環境でのマイグレーションの強制実行

一部のマイグレーション操作は破壊的であり、データを失う可能性があります。本番データベースに対してこれらのコマンドを実行することを保護するために、コマンドの実行前に確認を求められます。確認なしでコマンドを強制実行するには、`--force`フラグを使用します。

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### マイグレーションのロールバック

最新のマイグレーション操作をロールバックするには、`rollback` Artisanコマンドを使用できます。このコマンドは、最後の「バッチ」のマイグレーションをロールバックします。これには複数のマイグレーションファイルが含まれる場合があります。

```shell
php artisan migrate:rollback
```

`rollback`コマンドに`step`オプションを指定することで、限られた数のマイグレーションをロールバックできます。例えば、次のコマンドは最後の5つのマイグレーションをロールバックします。

```shell
php artisan migrate:rollback --step=5
```

`rollback`コマンドに`batch`オプションを指定することで、特定の「バッチ」のマイグレーションをロールバックできます。`batch`オプションは、アプリケーションの`migrations`データベーステーブル内のバッチ値に対応します。例えば、次のコマンドはバッチ3のすべてのマイグレーションをロールバックします。

```shell
php artisan migrate:rollback --batch=3
```

マイグレーションが実行されるSQL文を実際に実行せずに確認したい場合は、`migrate:rollback`コマンドに`--pretend`フラグを指定できます。

```shell
php artisan migrate:rollback --pretend
```

`migrate:reset`コマンドは、アプリケーションのすべてのマイグレーションをロールバックします。

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### 単一のコマンドでのロールバックとマイグレーション

`migrate:refresh`コマンドは、すべてのマイグレーションをロールバックし、`migrate`コマンドを実行します。このコマンドは、データベース全体を効果的に再作成します。

```shell
php artisan migrate:refresh
```

# データベースをリフレッシュし、すべてのデータベースシードを実行する...
php artisan migrate:refresh --seed
```

`refresh`コマンドに`step`オプションを指定することで、限られた数のマイグレーションのみをロールバックして再マイグレーションすることができます。例えば、以下のコマンドは最後の5つのマイグレーションをロールバックして再マイグレーションします：

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### すべてのテーブルを削除してマイグレーションを実行

`migrate:fresh`コマンドは、データベースからすべてのテーブルを削除し、その後`migrate`コマンドを実行します：

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

デフォルトでは、`migrate:fresh`コマンドはデフォルトのデータベース接続からテーブルを削除します。ただし、`--database`オプションを使用して、マイグレーションするデータベース接続を指定できます。データベース接続名は、アプリケーションの`database`[設定ファイル](configuration.md)で定義された接続に対応する必要があります：

```shell
php artisan migrate:fresh --database=admin
```

> WARNING:  
> `migrate:fresh`コマンドは、プレフィックスに関係なくすべてのデータベーステーブルを削除します。このコマンドは、他のアプリケーションと共有されているデータベースで開発する際には注意して使用してください。

<a name="tables"></a>
## テーブル

<a name="creating-tables"></a>
### テーブルの作成

新しいデータベーステーブルを作成するには、`Schema`ファサードの`create`メソッドを使用します。`create`メソッドは2つの引数を受け取ります：最初の引数はテーブルの名前で、2番目の引数は新しいテーブルを定義するために使用できる`Blueprint`オブジェクトを受け取るクロージャです：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email');
        $table->timestamps();
    });

テーブルを作成する際に、スキーマビルダーの[カラムメソッド](#creating-columns)を使用してテーブルのカラムを定義できます。

<a name="determining-table-column-existence"></a>
#### テーブル/カラムの存在確認

`hasTable`、`hasColumn`、`hasIndex`メソッドを使用して、テーブル、カラム、またはインデックスの存在を確認できます：

    if (Schema::hasTable('users')) {
        // "users" テーブルが存在する...
    }

    if (Schema::hasColumn('users', 'email')) {
        // "users" テーブルが存在し、"email" カラムを持つ...
    }

    if (Schema::hasIndex('users', ['email'], 'unique')) {
        // "users" テーブルが存在し、"email" カラムに一意のインデックスがある...
    }

<a name="database-connection-table-options"></a>
#### データベース接続とテーブルオプション

アプリケーションのデフォルト接続以外のデータベース接続でスキーマ操作を実行したい場合は、`connection`メソッドを使用します：

    Schema::connection('sqlite')->create('users', function (Blueprint $table) {
        $table->id();
    });

さらに、テーブル作成の他の側面を定義するために、いくつかの他のプロパティとメソッドを使用できます。`engine`プロパティは、MariaDBまたはMySQLを使用する際にテーブルのストレージエンジンを指定するために使用できます：

    Schema::create('users', function (Blueprint $table) {
        $table->engine('InnoDB');

        // ...
    });

`charset`と`collation`プロパティは、MariaDBまたはMySQLを使用する際に作成されるテーブルの文字セットと照合順序を指定するために使用できます：

    Schema::create('users', function (Blueprint $table) {
        $table->charset('utf8mb4');
        $table->collation('utf8mb4_unicode_ci');

        // ...
    });

`temporary`メソッドは、テーブルが「一時的」であることを示すために使用できます。一時テーブルは現在の接続のデータベースセッションにのみ表示され、接続が閉じられると自動的に削除されます：

    Schema::create('calculations', function (Blueprint $table) {
        $table->temporary();

        // ...
    });

データベーステーブルに「コメント」を追加したい場合は、テーブルインスタンスの`comment`メソッドを呼び出すことができます。テーブルコメントは現在、MariaDB、MySQL、およびPostgreSQLでのみサポートされています：

    Schema::create('calculations', function (Blueprint $table) {
        $table->comment('Business calculations');

        // ...
    });

<a name="updating-tables"></a>
### テーブルの更新

`Schema`ファサードの`table`メソッドは、既存のテーブルを更新するために使用できます。`create`メソッドと同様に、`table`メソッドは2つの引数を受け取ります：テーブルの名前と、テーブルにカラムやインデックスを追加するために使用できる`Blueprint`インスタンスを受け取るクロージャです：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->integer('votes');
    });

<a name="renaming-and-dropping-tables"></a>
### テーブルのリネーム/削除

既存のデータベーステーブルの名前を変更するには、`rename`メソッドを使用します：

    use Illuminate\Support\Facades\Schema;

    Schema::rename($from, $to);

既存のテーブルを削除するには、`drop`または`dropIfExists`メソッドを使用できます：

    Schema::drop('users');

    Schema::dropIfExists('users');

<a name="renaming-tables-with-foreign-keys"></a>
#### 外部キーを持つテーブルのリネーム

テーブルの名前を変更する前に、テーブル上の外部キー制約がマイグレーションファイル内で明示的な名前を持っていることを確認してください。そうでない場合、外部キー制約名は古いテーブル名を参照することになります。

<a name="columns"></a>
## カラム

<a name="creating-columns"></a>
### カラムの作成

`Schema`ファサードの`table`メソッドは、既存のテーブルを更新するために使用できます。`create`メソッドと同様に、`table`メソッドは2つの引数を受け取ります：テーブルの名前と、テーブルにカラムを追加するために使用できる`Illuminate\Database\Schema\Blueprint`インスタンスを受け取るクロージャです：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->integer('votes');
    });

<a name="available-column-types"></a>
### 利用可能なカラムタイプ

スキーマビルダーブループリントは、データベーステーブルに追加できるさまざまなタイプのカラムに対応するさまざまなメソッドを提供します。以下の表に、利用可能な各メソッドを示します：

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[binary](#column-method-binary)
[boolean](#column-method-boolean)
[char](#column-method-char)
[dateTimeTz](#column-method-dateTimeTz)
[dateTime](#column-method-dateTime)
[date](#column-method-date)
[decimal](#column-method-decimal)
[double](#column-method-double)
[enum](#column-method-enum)
[float](#column-method-float)
[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[geography](#column-method-geography)
[geometry](#column-method-geometry)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[ipAddress](#column-method-ipAddress)
[json](#column-method-json)
[jsonb](#column-method-jsonb)
[longText](#column-method-longText)
[macAddress](#column-method-macAddress)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[mediumText](#column-method-mediumText)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)
[nullableTimestamps](#column-method-nullableTimestamps)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)
[rememberToken](#column-method-rememberToken)
[set](#column-method-set)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[softDeletesTz](#column-method-softDeletesTz)
[softDeletes](#column-method-softDeletes)
[string](#column-method-string)
[text](#column-method-text)
[timeTz](#column-method-timeTz)
[time](#column-method-time)
[timestampTz](#column-method-timestampTz)
[timestamp](#column-method-timestamp)
[timestampsTz](#column-method-timestampsTz)
[timestamps](#column-method-timestamps)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[tinyText](#column-method-tinyText)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)
[ulidMorphs](#column-method-ulidMorphs)
[uuidMorphs](#column-method-uuidMorphs)
[ulid](#column-method-ulid)
[uuid](#column-method-uuid)
[year](#column-method-year)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

`bigIncrements`メソッドは、自動増分する`UNSIGNED BIGINT`（主キー）相当のカラムを作成します：

    $table->bigIncrements('id');

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

`bigInteger`メソッドは、`BIGINT`相当のカラムを作成します：

    $table->bigInteger('votes');

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

`binary`メソッドは、`BLOB`相当のカラムを作成します：

    $table->binary('photo');

MySQL、MariaDB、またはSQL Serverを使用する場合、`length`と`fixed`引数を渡して、`VARBINARY`または`BINARY`相当のカラムを作成できます：

    $table->binary('data', length: 16); // VARBINARY(16)

    $table->binary('data', length: 16, fixed: true); // BINARY(16)

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

`boolean`メソッドは、`BOOLEAN`相当のカラムを作成します。

    $table->boolean('confirmed');

<a name="column-method-char"></a>
#### `char()` {.collection-method}

`char`メソッドは、指定された長さの`CHAR`相当のカラムを作成します。

    $table->char('name', length: 100);

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

`dateTimeTz`メソッドは、オプションの小数秒精度を持つ`DATETIME`（タイムゾーン付き）相当のカラムを作成します。

    $table->dateTimeTz('created_at', precision: 0);

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

`dateTime`メソッドは、オプションの小数秒精度を持つ`DATETIME`相当のカラムを作成します。

    $table->dateTime('created_at', precision: 0);

<a name="column-method-date"></a>
#### `date()` {.collection-method}

`date`メソッドは、`DATE`相当のカラムを作成します。

    $table->date('created_at');

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

`decimal`メソッドは、指定された精度（全体の桁数）とスケール（小数点以下の桁数）を持つ`DECIMAL`相当のカラムを作成します。

    $table->decimal('amount', total: 8, places: 2);

<a name="column-method-double"></a>
#### `double()` {.collection-method}

`double`メソッドは、`DOUBLE`相当のカラムを作成します。

    $table->double('amount');

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

`enum`メソッドは、指定された有効な値を持つ`ENUM`相当のカラムを作成します。

    $table->enum('difficulty', ['easy', 'hard']);

<a name="column-method-float"></a>
#### `float()` {.collection-method}

`float`メソッドは、指定された精度を持つ`FLOAT`相当のカラムを作成します。

    $table->float('amount', precision: 53);

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

`foreignId`メソッドは、`UNSIGNED BIGINT`相当のカラムを作成します。

    $table->foreignId('user_id');

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

`foreignIdFor`メソッドは、指定されたモデルクラスに対して`{column}_id`相当のカラムを追加します。カラムの型は、モデルのキータイプに応じて`UNSIGNED BIGINT`、`CHAR(36)`、または`CHAR(26)`になります。

    $table->foreignIdFor(User::class);

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

`foreignUlid`メソッドは、`ULID`相当のカラムを作成します。

    $table->foreignUlid('user_id');

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

`foreignUuid`メソッドは、`UUID`相当のカラムを作成します。

    $table->foreignUuid('user_id');

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

`geography`メソッドは、指定された空間タイプとSRID（空間参照系識別子）を持つ`GEOGRAPHY`相当のカラムを作成します。

    $table->geography('coordinates', subtype: 'point', srid: 4326);

> NOTE:  
> 空間タイプのサポートは、データベースドライバに依存します。詳細については、データベースのドキュメントを参照してください。アプリケーションがPostgreSQLデータベースを使用している場合、`geography`メソッドを使用する前に[PostGIS](https://postgis.net)拡張機能をインストールする必要があります。

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

`geometry`メソッドは、指定された空間タイプとSRID（空間参照系識別子）を持つ`GEOMETRY`相当のカラムを作成します。

    $table->geometry('positions', subtype: 'point', srid: 0);

> NOTE:  
> 空間タイプのサポートは、データベースドライバに依存します。詳細については、データベースのドキュメントを参照してください。アプリケーションがPostgreSQLデータベースを使用している場合、`geometry`メソッドを使用する前に[PostGIS](https://postgis.net)拡張機能をインストールする必要があります。

<a name="column-method-id"></a>
#### `id()` {.collection-method}

`id`メソッドは、`bigIncrements`メソッドのエイリアスです。デフォルトでは、`id`カラムを作成しますが、カラムに別の名前を付けたい場合はカラム名を渡すことができます。

    $table->id();

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

`increments`メソッドは、自動増分する`UNSIGNED INTEGER`相当のカラムを主キーとして作成します。

    $table->increments('id');

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

`integer`メソッドは、`INTEGER`相当のカラムを作成します。

    $table->integer('votes');

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

`ipAddress`メソッドは、`VARCHAR`相当のカラムを作成します。

    $table->ipAddress('visitor');

PostgreSQLを使用する場合、`INET`カラムが作成されます。

<a name="column-method-json"></a>
#### `json()` {.collection-method}

`json`メソッドは、`JSON`相当のカラムを作成します。

    $table->json('options');

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

`jsonb`メソッドは、`JSONB`相当のカラムを作成します。

    $table->jsonb('options');

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

`longText`メソッドは、`LONGTEXT`相当のカラムを作成します。

    $table->longText('description');

MySQLまたはMariaDBを使用する場合、`binary`文字セットをカラムに適用して`LONGBLOB`相当のカラムを作成できます。

    $table->longText('data')->charset('binary'); // LONGBLOB

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

`macAddress`メソッドは、MACアドレスを保持することを意図したカラムを作成します。PostgreSQLなどの一部のデータベースシステムには、このタイプのデータに対して専用のカラムタイプがあります。他のデータベースシステムでは、文字列相当のカラムが使用されます。

    $table->macAddress('device');

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

`mediumIncrements`メソッドは、自動増分する`UNSIGNED MEDIUMINT`相当のカラムを主キーとして作成します。

    $table->mediumIncrements('id');

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

`mediumInteger`メソッドは、`MEDIUMINT`相当のカラムを作成します。

    $table->mediumInteger('votes');

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

`mediumText`メソッドは、`MEDIUMTEXT`相当のカラムを作成します。

    $table->mediumText('description');

MySQLまたはMariaDBを使用する場合、`binary`文字セットをカラムに適用して`MEDIUMBLOB`相当のカラムを作成できます。

    $table->mediumText('data')->charset('binary'); // MEDIUMBLOB

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

`morphs`メソッドは、`{column}_id`相当のカラムと`{column}_type`の`VARCHAR`相当のカラムを追加する便利なメソッドです。`{column}_id`のカラムタイプは、モデルのキータイプに応じて`UNSIGNED BIGINT`、`CHAR(36)`、または`CHAR(26)`になります。

このメソッドは、ポリモーフィックな[Eloquentリレーション](eloquent-relationships.md)に必要なカラムを定義する際に使用されます。以下の例では、`taggable_id`と`taggable_type`カラムが作成されます。

    $table->morphs('taggable');

<a name="column-method-nullableTimestamps"></a>
#### `nullableTimestamps()` {.collection-method}

`nullableTimestamps`メソッドは、[timestamps](#column-method-timestamps)メソッドのエイリアスです。

    $table->nullableTimestamps(precision: 0);

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

このメソッドは、[morphs](#column-method-morphs)メソッドと似ていますが、作成されるカラムは「nullable」になります。

    $table->nullableMorphs('taggable');

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

このメソッドは、[ulidMorphs](#column-method-ulidMorphs)メソッドと似ていますが、作成されるカラムは「nullable」になります。

    $table->nullableUlidMorphs('taggable');

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

このメソッドは、[uuidMorphs](#column-method-uuidMorphs)メソッドと似ていますが、作成されるカラムは「nullable」になります。

    $table->nullableUuidMorphs('taggable');

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

`rememberToken`メソッドは、現在の「remember me」[認証トークン](authentication.md#remembering-users)を保存することを意図した、nullableの`VARCHAR(100)`相当のカラムを作成します。

    $table->rememberToken();

<a name="column-method-set"></a>
#### `set()` {.collection-method}

`set`メソッドは、指定された有効な値のリストを持つ`SET`相当のカラムを作成します。

    $table->set('flavors', ['strawberry', 'vanilla']);

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

`smallIncrements`メソッドは、自動増分する`UNSIGNED SMALLINT`相当のカラムを主キーとして作成します。

    $table->smallIncrements('id');

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

`smallInteger`メソッドは、`SMALLINT`相当のカラムを作成します。

    $table->smallInteger('votes');

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

`softDeletesTz`メソッドは、nullableの`deleted_at`の`TIMESTAMP`（タイムゾーン付き）相当のカラムを、オプションの小数秒精度で追加します。このカラムは、Eloquentの「ソフトデリート」機能に必要な`deleted_at`タイムスタンプを保存することを意図しています。

    $table->softDeletesTz('deleted_at', precision: 0);

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

`softDeletes`メソッドは、nullableの`deleted_at`の`TIMESTAMP`相当のカラムを、オプションの小数秒精度で追加します。このカラムは、Eloquentの「ソフトデリート」機能に必要な`deleted_at`タイムスタンプを保存することを意図しています。

    $table->softDeletes('deleted_at', precision: 0);

<a name="column-method-string"></a>
#### `string()` {.collection-method}

`string`メソッドは、指定された長さの`VARCHAR`相当のカラムを作成します。

    $table->string('name', length: 100);

<a name="column-method-text"></a>
#### `text()` {.collection-method}

`text`メソッドは、`TEXT`相当のカラムを作成します。

    $table->text('description');

MySQLまたはMariaDBを使用する場合、カラムに`binary`文字セットを適用して、`BLOB`相当のカラムを作成できます。

    $table->text('data')->charset('binary'); // BLOB

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

`timeTz`メソッドは、オプションの小数点以下の秒精度を持つ`TIME`（タイムゾーン付き）相当のカラムを作成します。

    $table->timeTz('sunrise', precision: 0);

<a name="column-method-time"></a>
#### `time()` {.collection-method}

`time`メソッドは、オプションの小数点以下の秒精度を持つ`TIME`相当のカラムを作成します。

    $table->time('sunrise', precision: 0);

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

`timestampTz`メソッドは、オプションの小数点以下の秒精度を持つ`TIMESTAMP`（タイムゾーン付き）相当のカラムを作成します。

    $table->timestampTz('added_at', precision: 0);

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

`timestamp`メソッドは、オプションの小数点以下の秒精度を持つ`TIMESTAMP`相当のカラムを作成します。

    $table->timestamp('added_at', precision: 0);

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

`timestampsTz`メソッドは、オプションの小数点以下の秒精度を持つ`created_at`と`updated_at` `TIMESTAMP`（タイムゾーン付き）相当のカラムを作成します。

    $table->timestampsTz(precision: 0);

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

`timestamps`メソッドは、オプションの小数点以下の秒精度を持つ`created_at`と`updated_at` `TIMESTAMP`相当のカラムを作成します。

    $table->timestamps(precision: 0);

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

`tinyIncrements`メソッドは、主キーとして自動インクリメントする`UNSIGNED TINYINT`相当のカラムを作成します。

    $table->tinyIncrements('id');

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

`tinyInteger`メソッドは、`TINYINT`相当のカラムを作成します。

    $table->tinyInteger('votes');

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

`tinyText`メソッドは、`TINYTEXT`相当のカラムを作成します。

    $table->tinyText('notes');

MySQLまたはMariaDBを使用する場合、カラムに`binary`文字セットを適用して、`TINYBLOB`相当のカラムを作成できます。

    $table->tinyText('data')->charset('binary'); // TINYBLOB

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

`unsignedBigInteger`メソッドは、`UNSIGNED BIGINT`相当のカラムを作成します。

    $table->unsignedBigInteger('votes');

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

`unsignedInteger`メソッドは、`UNSIGNED INTEGER`相当のカラムを作成します。

    $table->unsignedInteger('votes');

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

`unsignedMediumInteger`メソッドは、`UNSIGNED MEDIUMINT`相当のカラムを作成します。

    $table->unsignedMediumInteger('votes');

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

`unsignedSmallInteger`メソッドは、`UNSIGNED SMALLINT`相当のカラムを作成します。

    $table->unsignedSmallInteger('votes');

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

`unsignedTinyInteger`メソッドは、`UNSIGNED TINYINT`相当のカラムを作成します。

    $table->unsignedTinyInteger('votes');

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

`ulidMorphs`メソッドは、`{column}_id` `CHAR(26)`相当のカラムと`{column}_type` `VARCHAR`相当のカラムを追加する便利なメソッドです。

このメソッドは、ULID識別子を使用するポリモーフィックな[Eloquentリレーション](eloquent-relationships.md)に必要なカラムを定義するために使用されます。以下の例では、`taggable_id`と`taggable_type`カラムが作成されます。

    $table->ulidMorphs('taggable');

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

`uuidMorphs`メソッドは、`{column}_id` `CHAR(36)`相当のカラムと`{column}_type` `VARCHAR`相当のカラムを追加する便利なメソッドです。

このメソッドは、UUID識別子を使用するポリモーフィックな[Eloquentリレーション](eloquent-relationships.md)に必要なカラムを定義するために使用されます。以下の例では、`taggable_id`と`taggable_type`カラムが作成されます。

    $table->uuidMorphs('taggable');

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

`ulid`メソッドは、`ULID`相当のカラムを作成します。

    $table->ulid('id');

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

`uuid`メソッドは、`UUID`相当のカラムを作成します。

    $table->uuid('id');

<a name="column-method-year"></a>
#### `year()` {.collection-method}

`year`メソッドは、`YEAR`相当のカラムを作成します。

    $table->year('birth_year');

<a name="column-modifiers"></a>
### カラム修飾子

上記のカラムタイプに加えて、データベーステーブルにカラムを追加する際に使用できるいくつかのカラム「修飾子」があります。例えば、カラムを「NULL許容」にするには、`nullable`メソッドを使用できます。

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

以下の表に、利用可能なすべてのカラム修飾子を示します。このリストには、[インデックス修飾子](#creating-indexes)は含まれていません。

<div class="overflow-auto" markdown=1>

| 修飾子                            | 説明                                                                                    |
| ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| `->after('column')`                 | カラムを別のカラムの「後」に配置する（MariaDB / MySQL）。                                     |
| `->autoIncrement()`                 | `INTEGER`カラムを自動インクリメント（主キー）に設定する。                                      |
| `->charset('utf8mb4')`              | カラムの文字セットを指定する（MariaDB / MySQL）。                                      |
| `->collation('utf8mb4_unicode_ci')` | カラムの照合順序を指定する。                                                            |
| `->comment('my comment')`           | カラムにコメントを追加する（MariaDB / MySQL / PostgreSQL）。                                      |
| `->default($value)`                 | カラムの「デフォルト」値を指定する。                                                      |
| `->first()`                         | カラムをテーブルの「最初」に配置する（MariaDB / MySQL）。                                       |
| `->from($integer)`                  | 自動インクリメントフィールドの開始値を設定する（MariaDB / MySQL / PostgreSQL）。           |
| `->invisible()`                     | カラムを`SELECT *`クエリで「不可視」にする（MariaDB / MySQL）。                           |
| `->nullable($value = true)`         | カラムに`NULL`値を挿入できるようにする。                                            |
| `->storedAs($expression)`           | 保存された生成カラムを作成する（MariaDB / MySQL / PostgreSQL / SQLite）。                      |
| `->unsigned()`                      | `INTEGER`カラムを`UNSIGNED`に設定する（MariaDB / MySQL）。                                         |
| `->useCurrent()`                    | `TIMESTAMP`カラムが`CURRENT_TIMESTAMP`をデフォルト値として使用するように設定する。                           |
| `->useCurrentOnUpdate()`            | `TIMESTAMP`カラムがレコードが更新されたときに`CURRENT_TIMESTAMP`を使用するように設定する（MariaDB / MySQL）。 |
| `->virtualAs($expression)`          | 仮想生成カラムを作成する（MariaDB / MySQL / SQLite）。                                  |
| `->generatedAs($expression)`        | 指定されたシーケンスオプションを持つIDカラムを作成する（PostgreSQL）。                        |
| `->always()`                        | IDカラムのシーケンス値と入力の優先順位を定義する（PostgreSQL）。      |

</div>

<a name="default-expressions"></a>
#### デフォルト式

`default`修飾子は、値または`Illuminate\Database\Query\Expression`インスタンスを受け入れます。`Expression`インスタンスを使用すると、Laravelが値を引用符で囲むのを防ぎ、データベース固有の関数を使用できるようになります。これは、特にJSONカラムにデフォルト値を割り当てる必要がある場合に特に便利です。

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Query\Expression;
    use Illuminate\Database\Migrations\Migration;

```php
return new class extends Migration
{
    /**
     * マイグレーションを実行する。
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

> WARNING:  
> デフォルト式のサポートは、データベースドライバ、データベースのバージョン、およびフィールドタイプに依存します。データベースのドキュメントを参照してください。

<a name="column-order"></a>
#### カラムの順序

MariaDBまたはMySQLデータベースを使用する場合、スキーマ内の既存のカラムの後にカラムを追加するために`after`メソッドを使用できます。

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

<a name="modifying-columns"></a>
### カラムの変更

`change`メソッドを使用すると、既存のカラムのタイプと属性を変更できます。例えば、`string`カラムのサイズを増やしたい場合があります。`change`メソッドを実際に見るために、`name`カラムのサイズを25から50に増やしてみましょう。これを行うには、カラムの新しい状態を定義してから`change`メソッドを呼び出します。

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

カラムを変更する際には、カラム定義に保持したいすべての修飾子を明示的に含める必要があります。欠落している属性は削除されます。例えば、`unsigned`、`default`、および`comment`属性を保持するには、カラムを変更する際に各修飾子を明示的に呼び出す必要があります。

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

`change`メソッドはカラムのインデックスを変更しません。したがって、カラムを変更する際にインデックス修飾子を使用して明示的にインデックスを追加または削除することができます。

```php
// インデックスを追加...
$table->bigIncrements('id')->primary()->change();

// インデックスを削除...
$table->char('postal_code', 10)->unique(false)->change();
```

<a name="renaming-columns"></a>
### カラムの名前変更

カラムの名前を変更するには、スキーマビルダーが提供する`renameColumn`メソッドを使用できます。

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="dropping-columns"></a>
### カラムの削除

カラムを削除するには、スキーマビルダーの`dropColumn`メソッドを使用できます。

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

`dropColumn`メソッドにカラム名の配列を渡すことで、テーブルから複数のカラムを削除できます。

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### 利用可能なコマンドエイリアス

Laravelは、一般的なタイプのカラムを削除するための便利なメソッドをいくつか提供しています。以下の表に各メソッドが記載されています。

<div class="overflow-auto" markdown=1>

| コマンド                             | 説明                                           |
| ----------------------------------- | --------------------------------------------- |
| `$table->dropMorphs('morphable');`  | `morphable_id`と`morphable_type`カラムを削除。 |
| `$table->dropRememberToken();`      | `remember_token`カラムを削除。                 |
| `$table->dropSoftDeletes();`        | `deleted_at`カラムを削除。                     |
| `$table->dropSoftDeletesTz();`      | `dropSoftDeletes()`メソッドのエイリアス。      |
| `$table->dropTimestamps();`         | `created_at`と`updated_at`カラムを削除。       |
| `$table->dropTimestampsTz();`       | `dropTimestamps()`メソッドのエイリアス。       |

</div>

<a name="indexes"></a>
## インデックス

<a name="creating-indexes"></a>
### インデックスの作成

Laravelのスキーマビルダーは、いくつかのタイプのインデックスをサポートしています。以下の例では、新しい`email`カラムを作成し、その値が一意であることを指定しています。インデックスを作成するために、カラム定義に`unique`メソッドをチェーンできます。

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

または、カラムを定義した後にインデックスを作成することもできます。そのためには、スキーマビルダーブループリントの`unique`メソッドを呼び出す必要があります。このメソッドは、一意のインデックスを受け取るカラムの名前を受け取ります。

```php
$table->unique('email');
```

インデックスメソッドにカラムの配列を渡すことで、複合（または複合）インデックスを作成できます。

```php
$table->index(['account_id', 'created_at']);
```

インデックスを作成する際、Laravelは自動的にテーブル名、カラム名、およびインデックスタイプに基づいてインデックス名を生成しますが、メソッドの2番目の引数を渡してインデックス名を自分で指定することもできます。

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### 利用可能なインデックスタイプ

Laravelのスキーマビルダーブループリントクラスは、Laravelがサポートする各タイプのインデックスを作成するためのメソッドを提供しています。各インデックスメソッドは、インデックスの名前を指定するためのオプションの2番目の引数を受け取ります。省略した場合、名前はテーブル名、カラム名、およびインデックスタイプから派生します。以下の表に、利用可能な各インデックスメソッドが記載されています。

<div class="overflow-auto" markdown=1>

| コマンド                                          | 説明                                                    |
| ------------------------------------------------ | ------------------------------------------------------- |
| `$table->primary('id');`                         | 主キーを追加。                                           |
| `$table->primary(['id', 'parent_id']);`          | 複合キーを追加。                                         |
| `$table->unique('email');`                       | 一意のインデックスを追加。                               |
| `$table->index('state');`                        | インデックスを追加。                                     |
| `$table->fullText('body');`                      | 全文インデックスを追加（MariaDB / MySQL / PostgreSQL）。 |
| `$table->fullText('body')->language('english');` | 指定された言語の全文インデックスを追加（PostgreSQL）。   |
| `$table->spatialIndex('location');`              | 空間インデックスを追加（SQLite以外）。                   |

</div>

<a name="renaming-indexes"></a>
### インデックスの名前変更

インデックスの名前を変更するには、スキーマビルダーブループリントが提供する`renameIndex`メソッドを使用できます。このメソッドは、現在のインデックス名を最初の引数として、希望する名前を2番目の引数として受け取ります。

```php
$table->renameIndex('from', 'to')
```

<a name="dropping-indexes"></a>
### インデックスの削除

インデックスを削除するには、インデックスの名前を指定する必要があります。デフォルトでは、Laravelは自動的にテーブル名、インデックスされたカラムの名前、およびインデックスタイプに基づいてインデックス名を割り当てます。以下にいくつかの例を示します。

<div class="overflow-auto" markdown=1>

| コマンド                                                  | 説明                                                 |
| -------------------------------------------------------- | ---------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | "users"テーブルから主キーを削除。                    |
| `$table->dropUnique('users_email_unique');`              | "users"テーブルから一意のインデックスを削除。        |
| `$table->dropIndex('geo_state_index');`                  | "geo"テーブルから基本インデックスを削除。            |
| `$table->dropFullText('posts_body_fulltext');`           | "posts"テーブルから全文インデックスを削除。          |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | "geo"テーブルから空間インデックスを削除（SQLite以外）。 |

</div>

インデックスを削除するメソッドにカラムの配列を渡すと、テーブル名、カラム、およびインデックスタイプに基づいて従来のインデックス名が生成されます。

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // 'geo_state_index'インデックスを削除
});
```

<a name="foreign-key-constraints"></a>
### 外部キー制約

Laravelは、データベースレベルで参照整合性を強制するために使用される外部キー制約の作成もサポートしています。例えば、`posts`テーブルに`user_id`カラムを定義し、`users`テーブルの`id`カラムを参照します。

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

この構文はかなり冗長であるため、Laravelは規約を使用して開発者エクスペリエンスを向上させる追加の簡潔なメソッドを提供しています。`foreignId`メソッドを使用してカラムを作成する場合、上記の例は次のように書き換えることができます。

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

`foreignId`メソッドは`UNSIGNED BIGINT`相当のカラムを作成し、`constrained`メソッドは規約を使用して参照されるテーブルとカラムを決定します。テーブル名がLaravelの規約に一致しない場合は、`constrained`メソッドに手動で指定することができます。さらに、生成されるインデックスに割り当てるべき名前を指定することもできます。

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

"on delete" および "on update" プロパティの制約に対して、希望するアクションを指定することもできます:

```php
$table->foreignId('user_id')
      ->constrained()
      ->onUpdate('cascade')
      ->onDelete('cascade');
```

これらのアクションに対して、代替的で表現力豊かな構文も提供されています:

<div class="overflow-auto" markdown=1>

| メソッド                        | 説明                                       |
| ----------------------------- | ------------------------------------------------- |
| `$table->cascadeOnUpdate();`  | 更新時にカスケードする。                           |
| `$table->restrictOnUpdate();` | 更新時に制限する。                     |
| `$table->noActionOnUpdate();` | 更新時に何もしない。                             |
| `$table->cascadeOnDelete();`  | 削除時にカスケードする。                           |
| `$table->restrictOnDelete();` | 削除時に制限する。                     |
| `$table->nullOnDelete();`     | 削除時に外部キーの値をnullに設定する。 |

</div>

追加の[カラム修飾子](#column-modifiers)は、`constrained`メソッドの前に呼び出す必要があります:

```php
$table->foreignId('user_id')
      ->nullable()
      ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### 外部キーの削除

外部キーを削除するには、`dropForeign`メソッドを使用し、削除する外部キー制約の名前を引数として渡します。外部キー制約はインデックスと同じ命名規則を使用します。つまり、外部キー制約名はテーブル名と制約内のカラム名に基づき、その後に "\_foreign" サフィックスが付きます:

```php
$table->dropForeign('posts_user_id_foreign');
```

または、外部キーを保持するカラム名を含む配列を`dropForeign`メソッドに渡すこともできます。配列はLaravelの制約命名規則を使用して外部キー制約名に変換されます:

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### 外部キー制約の切り替え

マイグレーション内で外部キー制約を有効または無効にするには、以下のメソッドを使用します:

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // このクロージャ内では制約が無効になります...
});
```

> WARNING:  
> SQLiteはデフォルトで外部キー制約を無効にしています。SQLiteを使用する場合、マイグレーションで外部キーを作成しようとする前に、データベース設定で[外部キーのサポートを有効にする](database.md#configuration)ようにしてください。

<a name="events"></a>
## イベント

便宜上、各マイグレーション操作は[イベント](events.md)を発行します。以下のすべてのイベントは、基本クラス`Illuminate\Database\Events\MigrationEvent`を拡張しています:

<div class="overflow-auto" markdown=1>

| クラス                                            | 説明                                      |
| ------------------------------------------------ | ------------------------------------------------ |
| `Illuminate\Database\Events\MigrationsStarted`   | マイグレーションのバッチが実行されようとしています。   |
| `Illuminate\Database\Events\MigrationsEnded`     | マイグレーションのバッチが実行を終了しました。    |
| `Illuminate\Database\Events\MigrationStarted`    | 単一のマイグレーションが実行されようとしています。      |
| `Illuminate\Database\Events\MigrationEnded`      | 単一のマイグレーションが実行を終了しました。       |
| `Illuminate\Database\Events\NoPendingMigrations` | マイグレーションコマンドが保留中のマイグレーションを見つけませんでした。 |
| `Illuminate\Database\Events\SchemaDumped`        | データベーススキーマのダンプが完了しました。            |
| `Illuminate\Database\Events\SchemaLoaded`        | 既存のデータベーススキーマのダンプが読み込まれました。     |

</div>

