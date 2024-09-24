# アップグレードガイド

- [10.x から 11.0 へのアップグレード](#upgrade-11.0)

<a name="high-impact-changes"></a>
## 高影響の変更

<div class="content-list" markdown="1">

- [依存関係の更新](#updating-dependencies)
- [アプリケーション構造](#application-structure)
- [浮動小数点型](#floating-point-types)
- [カラムの変更](#modifying-columns)
- [SQLite の最低バージョン](#sqlite-minimum-version)
- [Sanctum の更新](#updating-sanctum)

</div>

<a name="medium-impact-changes"></a>
## 中影響の変更

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [パスワードの再ハッシュ](#password-rehashing)
- [秒単位のレート制限](#per-second-rate-limiting)
- [Spatie Once パッケージ](#spatie-once-package)

</div>

<a name="low-impact-changes"></a>
## 低影響の変更

<div class="content-list" markdown="1">

- [Doctrine DBAL の削除](#doctrine-dbal-removal)
- [Eloquent モデルの `casts` メソッド](#eloquent-model-casts-method)
- [空間型](#spatial-types)
- [`Enumerable` コントラクト](#the-enumerable-contract)
- [`UserProvider` コントラクト](#the-user-provider-contract)
- [`Authenticatable` コントラクト](#the-authenticatable-contract)

</div>

<a name="upgrade-11.0"></a>
## 10.x から 11.0 へのアップグレード

<a name="estimated-upgrade-time-??-minutes"></a>
#### 推定アップグレード時間: 15 分

> NOTE:  
> 可能な限りすべての破壊的変更を文書化しようとしています。これらの破壊的変更の一部は、フレームワークのあまり知られていない部分にあるため、これらの変更の一部が実際にアプリケーションに影響を与える可能性があります。時間を節約したいですか？ [Laravel Shift](https://laravelshift.com/) を使用して、アプリケーションのアップグレードを自動化することができます。

<a name="updating-dependencies"></a>
### 依存関係の更新

**影響の可能性: 高**

#### PHP 8.2.0 が必要

Laravel は現在、PHP 8.2.0 以上を必要とします。

#### curl 7.34.0 が必要

Laravel の HTTP クライアントは現在、curl 7.34.0 以上を必要とします。

#### Composer の依存関係

アプリケーションの `composer.json` ファイルで以下の依存関係を更新する必要があります:

<div class="content-list" markdown="1">

- `laravel/framework` を `^11.0` に
- `nunomaduro/collision` を `^8.1` に
- `laravel/breeze` を `^2.0` に (インストールされている場合)
- `laravel/cashier` を `^15.0` に (インストールされている場合)
- `laravel/dusk` を `^8.0` に (インストールされている場合)
- `laravel/jetstream` を `^5.0` に (インストールされている場合)
- `laravel/octane` を `^2.3` に (インストールされている場合)
- `laravel/passport` を `^12.0` に (インストールされている場合)
- `laravel/sanctum` を `^4.0` に (インストールされている場合)
- `laravel/scout` を `^10.0` に (インストールされている場合)
- `laravel/spark-stripe` を `^5.0` に (インストールされている場合)
- `laravel/telescope` を `^5.0` に (インストールされている場合)
- `livewire/livewire` を `^3.4` に (インストールされている場合)
- `inertiajs/inertia-laravel` を `^1.0` に (インストールされている場合)

</div>

アプリケーションが Laravel Cashier Stripe、Passport、Sanctum、Spark Stripe、または Telescope を使用している場合、それらのマイグレーションをアプリケーションに公開する必要があります。Cashier Stripe、Passport、Sanctum、Spark Stripe、および Telescope は、独自のマイグレーションディレクトリからマイグレーションを自動的に読み込まなくなりました。したがって、以下のコマンドを実行して、それらのマイグレーションをアプリケーションに公開する必要があります:

```bash
php artisan vendor:publish --tag=cashier-migrations
php artisan vendor:publish --tag=passport-migrations
php artisan vendor:publish --tag=sanctum-migrations
php artisan vendor:publish --tag=spark-migrations
php artisan vendor:publish --tag=telescope-migrations
```

さらに、これらの各パッケージのアップグレードガイドを確認して、追加の破壊的変更に注意する必要があります:

- [Laravel Cashier Stripe](#cashier-stripe)
- [Laravel Passport](#passport)
- [Laravel Sanctum](#sanctum)
- [Laravel Spark Stripe](#spark-stripe)
- [Laravel Telescope](#telescope)

Laravel インストーラを手動でインストールしている場合、Composer を介してインストーラを更新する必要があります:

```bash
composer global require laravel/installer:^5.6
```

最後に、以前にアプリケーションに追加した `doctrine/dbal` Composer 依存関係を削除することができます。Laravel はもはやこのパッケージに依存していません。

<a name="application-structure"></a>
### アプリケーション構造

Laravel 11 は、デフォルトのアプリケーション構造を新しくし、デフォルトのファイルが少なくなりました。具体的には、新しい Laravel アプリケーションには、サービスプロバイダ、ミドルウェア、および設定ファイルが少なくなりました。

ただし、Laravel 10 から Laravel 11 にアップグレードするアプリケーションがアプリケーション構造を移行しようとすることは**推奨しません**。Laravel 11 は、Laravel 10 のアプリケーション構造もサポートするように慎重に調整されています。

<a name="authentication"></a>
### 認証

<a name="password-rehashing"></a>
#### パスワードの再ハッシュ

Laravel 11 は、ハッシュアルゴリズムの「作業係数」がパスワードが最後にハッシュされてから更新された場合、認証中にユーザーのパスワードを自動的に再ハッシュします。

通常、これはアプリケーションに影響を与えないはずです。ただし、この動作を無効にするには、アプリケーションの `config/hashing.php` 設定ファイルに `rehash_on_login` オプションを追加します:

    'rehash_on_login' => false,

<a name="the-user-provider-contract"></a>
#### `UserProvider` コントラクト

**影響の可能性: 低**

`Illuminate\Contracts\Auth\UserProvider` コントラクトに新しい `rehashPasswordIfRequired` メソッドが追加されました。このメソッドは、アプリケーションのハッシュアルゴリズムの作業係数が変更された場合に、ユーザーのパスワードを再ハッシュしてストレージに保存する役割を果たします。

アプリケーションまたはパッケージがこのインターフェースを実装するクラスを定義している場合、新しい `rehashPasswordIfRequired` メソッドを実装に追加する必要があります。参照実装は `Illuminate\Auth\EloquentUserProvider` クラス内にあります:

```php
public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
```

<a name="the-authenticatable-contract"></a>
#### `Authenticatable` コントラクト

**影響の可能性: 低**

`Illuminate\Contracts\Auth\Authenticatable` コントラクトに新しい `getAuthPasswordName` メソッドが追加されました。このメソッドは、認証可能なエンティティのパスワードカラムの名前を返す役割を果たします。

アプリケーションまたはパッケージがこのインターフェースを実装するクラスを定義している場合、新しい `getAuthPasswordName` メソッドを実装に追加する必要があります:

```php
public function getAuthPasswordName()
{
    return 'password';
}
```

Laravel に含まれるデフォルトの `User` モデルは、このメソッドが `Illuminate\Auth\Authenticatable` トレイトに含まれているため、自動的にこのメソッドを受け取ります。

<a name="the-authentication-exception-class"></a>
#### `AuthenticationException` クラス

**影響の可能性: 非常に低**

`Illuminate\Auth\AuthenticationException` クラスの `redirectTo` メソッドは、最初の引数として `Illuminate\Http\Request` インスタンスを必要とするようになりました。この例外を手動でキャッチして `redirectTo` メソッドを呼び出している場合、コードをそれに応じて更新する必要があります:

```php
if ($e instanceof AuthenticationException) {
    $path = $e->redirectTo($request);
}
```

<a name="cache"></a>
### キャッシュ

<a name="cache-key-prefixes"></a>
#### キャッシュキープレフィックス

**影響の可能性: 非常に低**

以前は、DynamoDB、Memcached、または Redis キャッシュストアにキャッシュキープレフィックスが定義されている場合、Laravel はプレフィックスに `:` を追加していました。Laravel 11 では、キャッシュキープレフィックスに `:` サフィックスが付きません。以前のプレフィックス動作を維持したい場合は、キャッシュキープレフィックスに手動で `:` サフィックスを追加できます。

<a name="collections"></a>
### コレクション

<a name="the-enumerable-contract"></a>
#### `Enumerable` コントラクト

**影響の可能性: 低**

`Illuminate\Support\Enumerable` コントラクトの `dump` メソッドが、可変長引数 `...$args` を受け入れるように更新されました。このインターフェースを実装している場合、実装をそれに応じて更新する必要があります:

```php
public function dump(...$args);
```

<a name="database"></a>
### データベース

<a name="sqlite-minimum-version"></a>
#### SQLite 3.26.0+

**影響の可能性: 高**

アプリケーションが SQLite データベースを利用している場合、SQLite 3.26.0 以上が必要です。

<a name="eloquent-model-casts-method"></a>
#### Eloquent モデルの `casts` メソッド

**影響の可能性: 低**

ベースの Eloquent モデルクラスは、属性キャストの定義をサポートするために `casts` メソッドを定義するようになりました。アプリケーションのモデルの1つが `casts` リレーションを定義している場合、ベースの Eloquent モデルクラスに存在する `casts` メソッドと競合する可能性があります。

<a name="modifying-columns"></a>
#### カラムの変更

**影響の可能性: 高**

カラムを変更する場合、変更後のカラム定義に保持したいすべての修飾子を明示的に含める必要があります。欠落している属性は削除されます。たとえば、`unsigned`、`default`、および `comment` 属性を保持するには、カラムを変更する際に各修飾子を明示的に呼び出す必要があります。以前のマイグレーションによってそれらの属性がカラムに割り当てられていた場合でも同様です。

たとえば、`votes` カラムを `unsigned`、`default`、および `comment` 属性で作成するマイグレーションがあるとします:

```php
Schema::create('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('The vote count');
});
```

後で、カラムを `nullable` にも変更するマイグレーションを作成します:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->nullable()->change();
});
```

Laravel 10 では、このマイグレーションはカラムに `unsigned`、`default`、および `comment` 属性を保持します。しかし、Laravel 11 では、マイグレーションに以前に定義されたすべての属性も含める必要があります。そうしないと、それらは削除されます:

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')
        ->unsigned()
        ->default(1)
        ->comment('The vote count')
        ->nullable()
        ->change();
});
```

`change`メソッドは、カラムのインデックスを変更しません。したがって、カラムを変更する際にインデックスを明示的に追加または削除するために、インデックス修飾子を使用することができます。

```php
// インデックスを追加...
$table->bigIncrements('id')->primary()->change();

// インデックスを削除...
$table->char('postal_code', 10)->unique(false)->change();
```

カラムの既存の属性を保持するために、アプリケーション内のすべての既存の「change」マイグレーションを更新したくない場合は、単に[マイグレーションを圧縮](migrations.md#squashing-migrations)することができます。

```bash
php artisan schema:dump
```

マイグレーションが圧縮されると、Laravelはアプリケーションのスキーマファイルを使用してデータベースを「マイグレート」し、その後に保留中のマイグレーションを実行します。

<a name="floating-point-types"></a>
#### 浮動小数点型

**影響の可能性: 高**

`double`および`float`マイグレーションカラムタイプは、すべてのデータベースで一貫性を持つように書き直されました。

`double`カラムタイプは、合計桁数と小数点以下の桁数（小数点以下の桁数）なしで`DOUBLE`相当のカラムを作成します。これは標準のSQL構文です。したがって、`$total`と`$places`の引数を削除することができます。

```php
$table->double('amount');
```

`float`カラムタイプは、合計桁数と小数点以下の桁数（小数点以下の桁数）なしで`FLOAT`相当のカラムを作成しますが、ストレージサイズを4バイトの単精度カラムまたは8バイトの倍精度カラムとして決定するためのオプションの`$precision`指定があります。したがって、`$total`と`$places`の引数を削除し、データベースのドキュメントに従って希望する値としてオプションの`$precision`を指定することができます。

```php
$table->float('amount', precision: 53);
```

`unsignedDecimal`、`unsignedDouble`、および`unsignedFloat`メソッドは削除されました。これらのカラムタイプの符号なし修飾子はMySQLによって非推奨となり、他のデータベースシステムでは標準化されていませんでした。ただし、これらのカラムタイプに対して非推奨の符号なし属性を引き続き使用したい場合は、カラム定義に`unsigned`メソッドを連鎖させることができます。

```php
$table->decimal('amount', total: 8, places: 2)->unsigned();
$table->double('amount')->unsigned();
$table->float('amount', precision: 53)->unsigned();
```

<a name="dedicated-mariadb-driver"></a>
#### 専用のMariaDBドライバ

**影響の可能性: 非常に低**

MariaDBデータベースに接続する際に常にMySQLドライバを使用する代わりに、Laravel 11はMariaDB用の専用データベースドライバを追加しました。

アプリケーションがMariaDBデータベースに接続している場合、接続設定を新しい`mariadb`ドライバに更新して、将来のMariaDB固有の機能を利用することができます。

```php
'driver' => 'mariadb',
'url' => env('DB_URL'),
'host' => env('DB_HOST', '127.0.0.1'),
'port' => env('DB_PORT', '3306'),
// ...
```

現在、新しいMariaDBドライバは現在のMySQLドライバと同様に動作しますが、1つの例外があります。`uuid`スキーマビルダーメソッドは、`char(36)`カラムの代わりにネイティブのUUIDカラムを作成します。

既存のマイグレーションが`uuid`スキーマビルダーメソッドを使用しており、新しい`mariadb`データベースドライバを使用することを選択した場合、`uuid`メソッドの呼び出しを`char`に更新して、破壊的な変更や予期しない動作を避ける必要があります。

```php
Schema::table('users', function (Blueprint $table) {
    $table->char('uuid', 36);

    // ...
});
```

<a name="spatial-types"></a>
#### 空間型

**影響の可能性: 低**

データベースマイグレーションの空間カラムタイプは、すべてのデータベースで一貫性を持つように書き直されました。したがって、マイグレーションから`point`、`lineString`、`polygon`、`geometryCollection`、`multiPoint`、`multiLineString`、`multiPolygon`、および`multiPolygonZ`メソッドを削除し、代わりに`geometry`または`geography`メソッドを使用することができます。

```php
$table->geometry('shapes');
$table->geography('coordinates');
```

MySQL、MariaDB、およびPostgreSQLでカラムに格納される値のタイプまたは空間参照システム識別子を明示的に制限するために、メソッドに`subtype`と`srid`を渡すことができます。

```php
$table->geometry('dimension', subtype: 'polygon', srid: 0);
$table->geography('latitude', subtype: 'point', srid: 4326);
```

PostgreSQL文法の`isGeometry`および`projection`カラム修飾子は、それに応じて削除されました。

<a name="doctrine-dbal-removal"></a>
#### Doctrine DBALの削除

**影響の可能性: 低**

以下のDoctrine DBAL関連のクラスとメソッドが削除されました。Laravelはもはやこのパッケージに依存せず、以前はカスタムタイプを必要としたさまざまなカラムタイプの適切な作成と変更のために、カスタムDoctrinesタイプの登録は不要になりました。

<div class="content-list" markdown="1">

- `Illuminate\Database\Schema\Builder::$alwaysUsesNativeSchemaOperationsIfPossible`クラスプロパティ
- `Illuminate\Database\Schema\Builder::useNativeSchemaOperationsIfPossible()`メソッド
- `Illuminate\Database\Connection::usingNativeSchemaOperations()`メソッド
- `Illuminate\Database\Connection::isDoctrineAvailable()`メソッド
- `Illuminate\Database\Connection::getDoctrineConnection()`メソッド
- `Illuminate\Database\Connection::getDoctrineSchemaManager()`メソッド
- `Illuminate\Database\Connection::getDoctrineColumn()`メソッド
- `Illuminate\Database\Connection::registerDoctrineType()`メソッド
- `Illuminate\Database\DatabaseManager::registerDoctrineType()`メソッド
- `Illuminate\Database\PDO`ディレクトリ
- `Illuminate\Database\DBAL\TimestampType`クラス
- `Illuminate\Database\Schema\Grammars\ChangeColumn`クラス
- `Illuminate\Database\Schema\Grammars\RenameColumn`クラス
- `Illuminate\Database\Schema\Grammars\Grammar::getDoctrineTableDiff()`メソッド

</div>

さらに、アプリケーションの`database`設定ファイルで`dbal.types`を介してカスタムDoctrineタイプを登録することは不要になりました。

以前にDoctrine DBALを使用してデータベースとその関連テーブルを検査していた場合、代わりにLaravelの新しいネイティブスキーマメソッド（`Schema::getTables()`、`Schema::getColumns()`、`Schema::getIndexes()`、`Schema::getForeignKeys()`など）を使用することができます。

<a name="deprecated-schema-methods"></a>
#### 非推奨のスキーマメソッド

**影響の可能性: 非常に低**

非推奨のDoctrineベースの`Schema::getAllTables()`、`Schema::getAllViews()`、および`Schema::getAllTypes()`メソッドは、新しいLaravelネイティブの`Schema::getTables()`、`Schema::getViews()`、および`Schema::getTypes()`メソッドに置き換えられ、削除されました。

PostgreSQLおよびSQL Serverを使用する場合、新しいスキーマメソッドは3つの部分からなる参照（例：`database.schema.table`）を受け入れません。したがって、代わりに`connection()`を使用してデータベースを宣言する必要があります。

```php
Schema::connection('database')->hasTable('schema.table');
```

<a name="get-column-types"></a>
#### スキーマビルダーの`getColumnType()`メソッド

**影響の可能性: 非常に低**

`Schema::getColumnType()`メソッドは、Doctrine DBAL相当のタイプではなく、指定されたカラムの実際のタイプを常に返すようになりました。

<a name="database-connection-interface"></a>
#### データベース接続インターフェース

**影響の可能性: 非常に低**

`Illuminate\Database\ConnectionInterface`インターフェースに新しい`scalar`メソッドが追加されました。このインターフェースの独自の実装を定義している場合、`scalar`メソッドを実装に追加する必要があります。

```php
public function scalar($query, $bindings = [], $useReadPdo = true);
```

<a name="dates"></a>
### 日付

<a name="carbon-3"></a>
#### Carbon 3

**影響の可能性: 中**

Laravel 11はCarbon 2とCarbon 3の両方をサポートしています。CarbonはLaravelとエコシステム全体のパッケージで広く使用されている日付操作ライブラリです。Carbon 3にアップグレードする場合、`diffIn*`メソッドが浮動小数点数を返し、時間の方向を示すために負の値を返す可能性があることに注意してください。これはCarbon 2からの大きな変更です。これらの変更やその他の変更を処理する方法については、Carbonの[変更ログ](https://github.com/briannesbitt/Carbon/releases/tag/3.0.0)を確認してください。

<a name="mail"></a>
### メール

<a name="the-mailer-contract"></a>
#### `Mailer`コントラクト

**影響の可能性: 非常に低**

`Illuminate\Contracts\Mail\Mailer`コントラクトに新しい`sendNow`メソッドが追加されました。アプリケーションまたはパッケージがこのコントラクトを手動で実装している場合、新しい`sendNow`メソッドを実装に追加する必要があります。

```php
public function sendNow($mailable, array $data = [], $callback = null);
```

<a name="packages"></a>
### パッケージ

<a name="publishing-service-providers"></a>
#### サービスプロバイダをアプリケーションに公開

**影響の可能性: 非常に低**

アプリケーションの`app/Providers`ディレクトリにサービスプロバイダを手動で公開し、アプリケーションの`config/app.php`設定ファイルを手動で変更してサービスプロバイダを登録するLaravelパッケージを作成した場合、パッケージを更新して新しい`ServiceProvider::addProviderToBootstrapFile`メソッドを使用する必要があります。

`addProviderToBootstrapFile`メソッドは、公開したサービスプロバイダをアプリケーションの`bootstrap/providers.php`ファイルに自動的に追加します。これは、`providers`配列が新しいLaravel 11アプリケーションの`config/app.php`設定ファイルに存在しないためです。

```php
use Illuminate\Support\ServiceProvider;

ServiceProvider::addProviderToBootstrapFile(Provider::class);
```

<a name="queues"></a>
### キュー

<a name="the-batch-repository-interface"></a>
#### `BatchRepository`インターフェース

**影響の可能性: 非常に低**

`Illuminate\Bus\BatchRepository`インターフェースに新しい`rollBack`メソッドが追加されました。このインターフェースを独自のパッケージまたはアプリケーション内で実装している場合、このメソッドを実装に追加する必要があります。

```php
public function rollBack();
```

<a name="synchronous-jobs-in-database-transactions"></a>
#### データベーストランザクション内の同期ジョブ

**影響の可能性: 非常に低**

以前は、同期ジョブ（`sync` キュードライバーを使用するジョブ）は、キュー接続の `after_commit` 設定オプションが `true` に設定されているか、ジョブで `afterCommit` メソッドが呼び出されているかに関係なく、即座に実行されていました。

Laravel 11 では、同期キュージョブはキュー接続の「コミット後」設定やジョブに従うようになりました。

<a name="rate-limiting"></a>
### レートリミット

<a name="per-second-rate-limiting"></a>
#### 秒単位のレートリミット

**影響の可能性: 中**

Laravel 11 は、分単位の粒度に制限される代わりに、秒単位のレートリミットをサポートします。この変更に関連する潜在的な破壊的変更について、注意する必要があります。

`GlobalLimit` クラスのコンストラクタは、分ではなく秒を受け入れるようになりました。このクラスは文書化されておらず、通常はアプリケーションで使用されません：

```php
new GlobalLimit($attempts, 2 * 60);
```

`Limit` クラスのコンストラクタも、分ではなく秒を受け入れるようになりました。このクラスの文書化された使用法はすべて、`Limit::perMinute` や `Limit::perSecond` などの静的コンストラクタに制限されています。ただし、このクラスを手動でインスタンス化している場合は、アプリケーションを更新して、クラスのコンストラクタに秒を提供するようにしてください：

```php
new Limit($key, $attempts, 2 * 60);
```

`Limit` クラスの `decayMinutes` プロパティは `decaySeconds` に名前が変更され、分ではなく秒を含むようになりました。

`Illuminate\Queue\Middleware\ThrottlesExceptions` および `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis` クラスのコンストラクタは、分ではなく秒を受け入れるようになりました：

```php
new ThrottlesExceptions($attempts, 2 * 60);
new ThrottlesExceptionsWithRedis($attempts, 2 * 60);
```

<a name="cashier-stripe"></a>
### Cashier Stripe

<a name="updating-cashier-stripe"></a>
#### Cashier Stripe の更新

**影響の可能性: 高**

Laravel 11 は Cashier Stripe 14.x をサポートしなくなりました。したがって、アプリケーションの Laravel Cashier Stripe 依存関係を `composer.json` ファイル内で `^15.0` に更新する必要があります。

Cashier Stripe 15.0 は、独自のマイグレーションディレクトリからマイグレーションを自動的にロードしなくなりました。代わりに、以下のコマンドを実行して、Cashier Stripe のマイグレーションをアプリケーションに公開する必要があります：

```shell
php artisan vendor:publish --tag=cashier-migrations
```

追加の破壊的変更については、完全な [Cashier Stripe アップグレードガイド](https://github.com/laravel/cashier-stripe/blob/15.x/UPGRADE.md) を確認してください。

<a name="spark-stripe"></a>
### Spark (Stripe)

<a name="updating-spark-stripe"></a>
#### Spark Stripe の更新

**影響の可能性: 高**

Laravel 11 は Laravel Spark Stripe 4.x をサポートしなくなりました。したがって、アプリケーションの Laravel Spark Stripe 依存関係を `composer.json` ファイル内で `^5.0` に更新する必要があります。

Spark Stripe 5.0 は、独自のマイグレーションディレクトリからマイグレーションを自動的にロードしなくなりました。代わりに、以下のコマンドを実行して、Spark Stripe のマイグレーションをアプリケーションに公開する必要があります：

```shell
php artisan vendor:publish --tag=spark-migrations
```

追加の破壊的変更については、完全な [Spark Stripe アップグレードガイド](https://spark.laravel.com/docs/spark-stripe/upgrade.html) を確認してください。

<a name="passport"></a>
### Passport

<a name="updating-telescope"></a>
#### Passport の更新

**影響の可能性: 高**

Laravel 11 は Laravel Passport 11.x をサポートしなくなりました。したがって、アプリケーションの Laravel Passport 依存関係を `composer.json` ファイル内で `^12.0` に更新する必要があります。

Passport 12.0 は、独自のマイグレーションディレクトリからマイグレーションを自動的にロードしなくなりました。代わりに、以下のコマンドを実行して、Passport のマイグレーションをアプリケーションに公開する必要があります：

```shell
php artisan vendor:publish --tag=passport-migrations
```

さらに、パスワード付与タイプはデフォルトで無効になっています。アプリケーションの `AppServiceProvider` の `boot` メソッドで `enablePasswordGrant` メソッドを呼び出すことで有効にできます：

    public function boot(): void
    {
        Passport::enablePasswordGrant();
    }

<a name="sanctum"></a>
### Sanctum

<a name="updating-sanctum"></a>
#### Sanctum の更新

**影響の可能性: 高**

Laravel 11 は Laravel Sanctum 3.x をサポートしなくなりました。したがって、アプリケーションの Laravel Sanctum 依存関係を `composer.json` ファイル内で `^4.0` に更新する必要があります。

Sanctum 4.0 は、独自のマイグレーションディレクトリからマイグレーションを自動的にロードしなくなりました。代わりに、以下のコマンドを実行して、Sanctum のマイグレーションをアプリケーションに公開する必要があります：

```shell
php artisan vendor:publish --tag=sanctum-migrations
```

次に、アプリケーションの `config/sanctum.php` 設定ファイルで、`authenticate_session`、`encrypt_cookies`、および `validate_csrf_token` ミドルウェアへの参照を以下のように更新する必要があります：

    'middleware' => [
        'authenticate_session' => Laravel\Sanctum\Http\Middleware\AuthenticateSession::class,
        'encrypt_cookies' => Illuminate\Cookie\Middleware\EncryptCookies::class,
        'validate_csrf_token' => Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
    ],

<a name="telescope"></a>
### Telescope

<a name="updating-telescope"></a>
#### Telescope の更新

**影響の可能性: 高**

Laravel 11 は Laravel Telescope 4.x をサポートしなくなりました。したがって、アプリケーションの Laravel Telescope 依存関係を `composer.json` ファイル内で `^5.0` に更新する必要があります。

Telescope 5.0 は、独自のマイグレーションディレクトリからマイグレーションを自動的にロードしなくなりました。代わりに、以下のコマンドを実行して、Telescope のマイグレーションをアプリケーションに公開する必要があります：

```shell
php artisan vendor:publish --tag=telescope-migrations
```

<a name="spatie-once-package"></a>
### Spatie Once パッケージ

**影響の可能性: 中**

Laravel 11 は、指定されたクロージャが一度だけ実行されることを保証する独自の [`once` 関数](helpers.md#method-once) を提供するようになりました。したがって、アプリケーションが `spatie/once` パッケージに依存している場合、競合を避けるために、アプリケーションの `composer.json` ファイルから削除する必要があります。

<a name="miscellaneous"></a>
### その他

また、`laravel/laravel` [GitHub リポジトリ](https://github.com/laravel/laravel) の変更も確認することをお勧めします。これらの変更の多くは必須ではありませんが、アプリケーションとこれらのファイルを同期させたい場合があります。このアップグレードガイドでは一部の変更がカバーされますが、設定ファイルやコメントの変更などはカバーされません。[GitHub 比較ツール](https://github.com/laravel/laravel/compare/10.x...11.x) を使用して簡単に変更を確認し、どの更新が重要かを選択できます。

