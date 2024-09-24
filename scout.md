# Laravel Scout

- [はじめに](#introduction)
- [インストール](#installation)
    - [キューイング](#queueing)
- [ドライバの前提条件](#driver-prerequisites)
    - [Algolia](#algolia)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [設定](#configuration)
    - [モデルインデックスの設定](#configuring-model-indexes)
    - [検索可能データの設定](#configuring-searchable-data)
    - [モデルIDの設定](#configuring-the-model-id)
    - [モデルごとの検索エンジンの設定](#configuring-search-engines-per-model)
    - [ユーザーの識別](#identifying-users)
- [データベース / コレクションエンジン](#database-and-collection-engines)
    - [データベースエンジン](#database-engine)
    - [コレクションエンジン](#collection-engine)
- [インデックス作成](#indexing)
    - [バッチインポート](#batch-import)
    - [レコードの追加](#adding-records)
    - [レコードの更新](#updating-records)
    - [レコードの削除](#removing-records)
    - [インデックス作成の一時停止](#pausing-indexing)
    - [条件付き検索可能なモデルインスタンス](#conditionally-searchable-model-instances)
- [検索](#searching)
    - [Where句](#where-clauses)
    - [ページネーション](#pagination)
    - [ソフトデリート](#soft-deleting)
    - [エンジン検索のカスタマイズ](#customizing-engine-searches)
- [カスタムエンジン](#custom-engines)

<a name="introduction"></a>
## はじめに

[Laravel Scout](https://github.com/laravel/scout) は、[Eloquentモデル](eloquent.md)にフルテキスト検索を追加するためのシンプルでドライバベースのソリューションを提供します。モデルオブザーバーを使用して、ScoutはEloquentレコードと検索インデックスを自動的に同期します。

現在、Scoutは [Algolia](https://www.algolia.com/)、[Meilisearch](https://www.meilisearch.com)、[Typesense](https://typesense.org)、およびMySQL / PostgreSQL (`database`) ドライバとともに提供されています。さらに、Scoutにはローカル開発用に設計された「コレクション」ドライバが含まれており、外部依存関係やサードパーティサービスを必要としません。また、カスタムドライバの作成は簡単であり、独自の検索実装でScoutを拡張することができます。

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを介してScoutをインストールします：

```shell
composer require laravel/scout
```

Scoutをインストールした後、`vendor:publish` Artisanコマンドを使用してScoutの設定ファイルを公開する必要があります。このコマンドは、`scout.php`設定ファイルをアプリケーションの`config`ディレクトリに公開します：

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

最後に、検索可能にしたいモデルに`Laravel\Scout\Searchable`トレイトを追加します。このトレイトは、モデルを検索ドライバと自動的に同期させるモデルオブザーバーを登録します：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### キューイング

Scoutを使用するために必須ではありませんが、ライブラリを使用する前に[キュードライバ](queues.md)を設定することを強く検討する必要があります。キューワーカーを実行すると、Scoutはモデル情報を検索インデックスに同期するすべての操作をキューに入れることができ、アプリケーションのWebインターフェースの応答時間が大幅に向上します。

キュードライバを設定したら、`config/scout.php`設定ファイルの`queue`オプションの値を`true`に設定します：

    'queue' => true,

`queue`オプションが`false`に設定されている場合でも、AlgoliaやMeilisearchのような一部のScoutドライバは常にレコードを非同期でインデックス化することを覚えておくことが重要です。つまり、Laravelアプリケーション内でインデックス操作が完了しても、検索エンジン自体は新しいレコードや更新されたレコードをすぐに反映しない場合があります。

Scoutジョブが使用する接続とキューを指定するには、`queue`設定オプションを配列として定義できます：

    'queue' => [
        'connection' => 'redis',
        'queue' => 'scout'
    ],

もちろん、Scoutジョブが使用する接続とキューをカスタマイズした場合、その接続とキューでジョブを処理するためにキューワーカーを実行する必要があります：

    php artisan queue:work redis --queue=scout

<a name="driver-prerequisites"></a>
## ドライバの前提条件

<a name="algolia"></a>
### Algolia

Algoliaドライバを使用する場合、`config/scout.php`設定ファイルでAlgoliaの`id`と`secret`の認証情報を設定する必要があります。認証情報を設定したら、Composerパッケージマネージャを介してAlgolia PHP SDKもインストールする必要があります：

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) は、非常に高速でオープンソースの検索エンジンです。ローカルマシンにMeilisearchをインストールする方法がわからない場合は、Laravelの公式サポートされているDocker開発環境である[Laravel Sail](sail.md#meilisearch)を使用できます。

Meilisearchドライバを使用する場合、Composerパッケージマネージャを介してMeilisearch PHP SDKをインストールする必要があります：

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

次に、`SCOUT_DRIVER`環境変数とMeilisearchの`host`および`key`認証情報をアプリケーションの`.env`ファイル内に設定します：

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

Meilisearchに関する詳細情報については、[Meilisearchのドキュメント](https://docs.meilisearch.com/learn/getting_started/quick_start.html)を参照してください。

さらに、`meilisearch/meilisearch-php`のバージョンがMeilisearchバイナリバージョンと互換性があることを確認する必要があります。互換性については、[Meilisearchのバイナリ互換性に関するドキュメント](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch)を確認してください。

> WARNING:  
> Meilisearchを使用するアプリケーションでScoutをアップグレードする場合、常にMeilisearchサービス自体の[追加の破壊的変更](https://github.com/meilisearch/Meilisearch/releases)を確認する必要があります。

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) は、非常に高速でオープンソースの検索エンジンであり、キーワード検索、セマンティック検索、地理検索、ベクトル検索をサポートしています。

Typesenseは[自己ホスト](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting)することも、[Typesense Cloud](https://cloud.typesense.org)を使用することもできます。

ScoutでTypesenseを使い始めるには、Composerパッケージマネージャを介してTypesense PHP SDKをインストールします：

```shell
composer require typesense/typesense-php
```

次に、`SCOUT_DRIVER`環境変数とTypesenseのホストおよびAPIキー認証情報をアプリケーションの`.env`ファイル内に設定します：

```env
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

[Laravel Sail](sail.md)を使用している場合、`TYPESENSE_HOST`環境変数をDockerコンテナ名に合わせて調整する必要があるかもしれません。また、インストールのポート、パス、プロトコルをオプションで指定することもできます：

```env
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

Typesenseコレクションの追加設定とスキーマ定義は、アプリケーションの`config/scout.php`設定ファイル内にあります。Typesenseに関する詳細情報については、[Typesenseのドキュメント](https://typesense.org/docs/guide/#quick-start)を参照してください。

<a name="preparing-data-for-storage-in-typesense"></a>
#### Typesenseへのデータ保存の準備

Typesenseを使用する場合、検索可能なモデルは、モデルの主キーを文字列にキャストし、作成日をUNIXタイムスタンプにキャストする`toSearchableArray`メソッドを定義する必要があります：

```php
/**
 * モデルのインデックス可能なデータ配列を取得します。
 *
 * @return array<string, mixed>
 */
public function toSearchableArray()
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

また、Typesenseコレクションスキーマをアプリケーションの`config/scout.php`ファイルで定義する必要があります。コレクションスキーマは、Typesenseを介して検索可能な各フィールドのデータ型を記述します。利用可能なすべてのスキーマオプションについては、[Typesenseのドキュメント](https://typesense.org/docs/latest/api/collections.html#schema-parameters)を参照してください。

Typesenseコレクションのスキーマを定義した後に変更する必要がある場合は、`scout:flush`と`scout:import`を実行して、既存のインデックス化されたデータをすべて削除し、スキーマを再作成するか、TypesenseのAPIを使用してインデックス化されたデータを削除せずにコレクションのスキーマを変更することができます。

検索可能なモデルがソフトデリート可能な場合、アプリケーションの`config/scout.php`設定ファイル内のモデルに対応するTypesenseスキーマに`__soft_deleted`フィールドを定義する必要があります：

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### 動的検索パラメータ

Typesenseでは、`options`メソッドを介して検索操作を実行する際に[検索パラメータ](https://typesense.org/docs/latest/api/search.html#search-parameters)を動的に変更できます：

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="configuration"></a>
## 設定

<a name="configuring-model-indexes"></a>
### モデルインデックスの設定

各Eloquentモデルは、特定の検索「インデックス」と同期されます。このインデックスには、そのモデルのすべての検索可能なレコードが含まれています。言い換えれば、各インデックスはMySQLのテーブルのようなものだと考えることができます。デフォルトでは、各モデルはモデルの典型的な「テーブル」名に一致するインデックスに永続化されます。通常、これはモデル名の複数形ですが、モデルのインデックスをカスタマイズするために、モデルの`searchableAs`メソッドをオーバーライドすることができます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * モデルに関連付けられたインデックスの名前を取得します。
         */
        public function searchableAs(): string
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 検索可能なデータの設定

デフォルトでは、指定されたモデルの`toArray`形式全体が検索インデックスに永続化されます。検索インデックスに同期されるデータをカスタマイズしたい場合は、モデルの`toSearchableArray`メソッドをオーバーライドすることができます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * モデルのインデックス可能なデータ配列を取得します。
         *
         * @return array<string, mixed>
         */
        public function toSearchableArray(): array
        {
            $array = $this->toArray();

            // データ配列をカスタマイズ...

            return $array;
        }
    }

Meilisearchなどの一部の検索エンジンは、正しい型のデータに対してのみフィルタ操作（`>`, `<`, など）を実行します。したがって、これらの検索エンジンを使用して検索可能なデータをカスタマイズする場合、数値が正しい型にキャストされていることを確認する必要があります。

    public function toSearchableArray()
    {
        return [
            'id' => (int) $this->id,
            'name' => $this->name,
            'price' => (float) $this->price,
        ];
    }

<a name="configuring-filterable-data-for-meilisearch"></a>
#### フィルタ可能なデータとインデックス設定の設定 (Meilisearch)

Scoutの他のドライバとは異なり、Meilisearchでは、フィルタ可能な属性、ソート可能な属性、および[その他のサポートされている設定フィールド](https://docs.meilisearch.com/reference/api/settings.html)などのインデックス検索設定を事前に定義する必要があります。

フィルタ可能な属性は、Scoutの`where`メソッドを呼び出す際にフィルタリングする予定の属性であり、ソート可能な属性は、Scoutの`orderBy`メソッドを呼び出す際にソートする予定の属性です。インデックス設定を定義するには、アプリケーションの`scout`設定ファイル内の`meilisearch`設定エントリの`index-settings`部分を調整します。

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // その他の設定フィールド...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

特定のインデックスの基になるモデルがソフトデリート可能であり、`index-settings`配列に含まれている場合、Scoutは自動的にそのインデックスでソフトデリートされたモデルのフィルタリングをサポートします。ソフトデリート可能なモデルインデックスに他のフィルタ可能またはソート可能な属性を定義する必要がない場合、そのモデルの`index-settings`配列に空のエントリを追加するだけで済みます。

```php
'index-settings' => [
    Flight::class => []
],
```

アプリケーションのインデックス設定を構成した後、`scout:sync-index-settings` Artisanコマンドを実行する必要があります。このコマンドは、現在構成されているインデックス設定をMeilisearchに通知します。便宜上、このコマンドをデプロイプロセスの一部にすることをお勧めします。

```shell
php artisan scout:sync-index-settings
```

<a name="configuring-the-model-id"></a>
### モデルIDの設定

デフォルトでは、Scoutはモデルの主キーを、検索インデックスに保存されるモデルの一意のID / キーとして使用します。この動作をカスタマイズする必要がある場合は、モデルの`getScoutKey`および`getScoutKeyName`メソッドをオーバーライドできます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * モデルのインデックスに使用される値を取得します。
         */
        public function getScoutKey(): mixed
        {
            return $this->email;
        }

        /**
         * モデルのインデックスに使用されるキー名を取得します。
         */
        public function getScoutKeyName(): mixed
        {
            return 'email';
        }
    }

<a name="configuring-search-engines-per-model"></a>
### モデルごとの検索エンジンの設定

検索時に、Scoutは通常、アプリケーションの`scout`設定ファイルで指定されたデフォルトの検索エンジンを使用します。ただし、特定のモデルの検索エンジンは、モデルの`searchableUsing`メソッドをオーバーライドすることで変更できます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Engines\Engine;
    use Laravel\Scout\EngineManager;
    use Laravel\Scout\Searchable;

    class User extends Model
    {
        use Searchable;

        /**
         * モデルのインデックスに使用されるエンジンを取得します。
         */
        public function searchableUsing(): Engine
        {
            return app(EngineManager::class)->engine('meilisearch');
        }
    }

<a name="identifying-users"></a>
### ユーザーの識別

Scoutでは、[Algolia](https://algolia.com)を使用する際にユーザーを自動的に識別することもできます。認証されたユーザーを検索操作に関連付けることは、Algoliaのダッシュボード内で検索分析を表示する際に役立つ場合があります。ユーザー識別を有効にするには、アプリケーションの`.env`ファイルで`SCOUT_IDENTIFY`環境変数を`true`として定義します。

```ini
SCOUT_IDENTIFY=true
```

この機能を有効にすると、リクエストのIPアドレスと認証されたユーザーの主識別子がAlgoliaに渡され、ユーザーが行ったすべての検索リクエストに関連付けられます。

<a name="database-and-collection-engines"></a>
## データベース / コレクションエンジン

<a name="database-engine"></a>
### データベースエンジン

> WARNING:  
> データベースエンジンは現在、MySQLとPostgreSQLをサポートしています。

アプリケーションが中小規模のデータベースと対話している場合や、軽いワークロードを持っている場合、Scoutの「データベース」エンジンを使用して始める方が便利かもしれません。データベースエンジンは、既存のデータベースから結果をフィルタリングする際に「where like」句と全文インデックスを使用して、クエリに適用される検索結果を決定します。

データベースエンジンを使用するには、`SCOUT_DRIVER`環境変数の値を`database`に設定するか、アプリケーションの`scout`設定ファイルで`database`ドライバを直接指定します。

```ini
SCOUT_DRIVER=database
```

データベースエンジンを好みのドライバとして指定したら、[検索可能なデータの設定](#configuring-searchable-data)を行う必要があります。その後、モデルに対して[検索クエリの実行](#searching)を開始できます。Algolia、Meilisearch、Typesenseインデックスのシードに必要なインデックス作成などの検索エンジンインデックス作成は、データベースエンジンを使用する場合は不要です。

#### データベース検索戦略のカスタマイズ

デフォルトでは、データベースエンジンは、[検索可能なデータとして設定された](#configuring-searchable-data)すべてのモデル属性に対して「where like」クエリを実行します。ただし、状況によっては、これによりパフォーマンスが低下する可能性があります。したがって、データベースエンジンの検索戦略を設定して、特定の列が全文検索クエリを使用したり、文字列のプレフィックス（`example%`）のみを検索する「where like」制約を使用したり、文字列全体（`%example%`）を検索する代わりに使用するように設定できます。

この動作を定義するには、モデルの`toSearchableArray`メソッドにPHP属性を割り当てることができます。追加の検索戦略動作が割り当てられていない列は、デフォルトの「where like」戦略を引き続き使用します。

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * モデルのインデックス可能なデータ配列を取得します。
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

> WARNING:  
> 列に全文クエリ制約を指定する前に、その列が[全文インデックス](migrations.md#available-index-types)に割り当てられていることを確認してください。

<a name="collection-engine"></a>
### コレクションエンジン

ローカル開発中にAlgolia、Meilisearch、Typesense検索エンジンを自由に使用できますが、「コレクション」エンジンを使用して始める方が便利かもしれません。コレクションエンジンは、既存のデータベースからの結果に対して「where」句とコレクションフィルタリングを使用して、クエリに適用される検索結果を決定します。このエンジンを使用する場合、検索可能なモデルを「インデックス」する必要はありません。ローカルデータベースから単純に取得されます。

コレクションエンジンを使用するには、`SCOUT_DRIVER`環境変数の値を`collection`に設定するか、アプリケーションの`scout`設定ファイルで`collection`ドライバを直接指定します。

```ini
SCOUT_DRIVER=collection
```

コレクションドライバを優先ドライバとして指定したら、モデルに対して[検索クエリを実行](#searching)できます。Algolia、Meilisearch、Typesenseインデックスのシードに必要なインデックス作成など、検索エンジンのインデックス作成は、コレクションエンジンを使用する場合は不要です。

#### データベースエンジンとの違い

一見すると、「データベース」と「コレクション」エンジンは非常によく似ています。どちらもデータベースと直接やり取りして検索結果を取得します。しかし、コレクションエンジンはフルテキストインデックスや`LIKE`句を使用して一致するレコードを見つけることはありません。代わりに、可能なすべてのレコードを取得し、Laravelの`Str::is`ヘルパーを使用して、検索文字列がモデル属性値内に存在するかどうかを判断します。

コレクションエンジンは、Laravelがサポートするすべてのリレーショナルデータベース（SQLiteやSQL Serverを含む）で動作するため、最も移植性の高い検索エンジンです。ただし、Scoutのデータベースエンジンよりも効率は低くなります。

<a name="indexing"></a>
## インデックス作成

<a name="batch-import"></a>
### バッチインポート

既存のプロジェクトにScoutをインストールする場合、既にデータベースにレコードが存在する可能性があります。Scoutは、既存のレコードを検索インデックスにインポートするために使用できる`scout:import` Artisanコマンドを提供しています。

```shell
php artisan scout:import "App\Models\Post"
```

`flush`コマンドを使用して、モデルのすべてのレコードを検索インデックスから削除できます。

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### インポートクエリの変更

バッチインポートに使用されるクエリを変更したい場合は、モデルに`makeAllSearchableUsing`メソッドを定義できます。これは、モデルをインポートする前に必要な関係の読み込みを追加するのに最適な場所です。

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * モデルを検索可能にする際に使用されるクエリを変更する。
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> WARNING:  
> `makeAllSearchableUsing`メソッドは、モデルをバッチインポートするためにキューを使用する場合には適用されない可能性があります。ジョブによって処理されるモデルコレクションの場合、関係は[復元されません](queues.md#handling-relationships)。

<a name="adding-records"></a>
### レコードの追加

`Laravel\Scout\Searchable`トレイトをモデルに追加したら、モデルインスタンスを`save`または`create`するだけで、自動的に検索インデックスに追加されます。[キューを使用する](#queueing)ようにScoutを設定している場合、この操作はキューワーカーによってバックグラウンドで実行されます。

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### クエリによるレコードの追加

Eloquentクエリを介してモデルのコレクションを検索インデックスに追加したい場合は、Eloquentクエリに`searchable`メソッドをチェーンできます。`searchable`メソッドはクエリの結果を[チャンク分割](eloquent.md#chunking-results)し、レコードを検索インデックスに追加します。ここでも、Scoutをキューを使用するように設定している場合、すべてのチャンクはキューワーカーによってバックグラウンドでインポートされます。

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

Eloquentリレーションインスタンスに`searchable`メソッドを呼び出すこともできます。

```php
$user->orders()->searchable();
```

または、メモリ内に既にEloquentモデルのコレクションがある場合は、コレクションインスタンスに`searchable`メソッドを呼び出して、モデルインスタンスを対応するインデックスに追加できます。

```php
$orders->searchable();
```

> NOTE:  
> `searchable`メソッドは「upsert」操作と考えることができます。つまり、モデルレコードが既にインデックスに存在する場合は更新されます。インデックスに存在しない場合は、インデックスに追加されます。

<a name="updating-records"></a>
### レコードの更新

検索可能なモデルを更新するには、モデルインスタンスのプロパティを更新し、データベースに`save`するだけです。Scoutは自動的に変更を検索インデックスに保持します。

```php
use App\Models\Order;

$order = Order::find(1);

// 注文を更新...

$order->save();
```

Eloquentクエリインスタンスに`searchable`メソッドを呼び出して、モデルのコレクションを更新することもできます。モデルが検索インデックスに存在しない場合は、作成されます。

```php
Order::where('price', '>', 100)->searchable();
```

リレーション内のすべてのモデルの検索インデックスレコードを更新したい場合は、リレーションインスタンスに`searchable`を呼び出すことができます。

```php
$user->orders()->searchable();
```

または、メモリ内に既にEloquentモデルのコレクションがある場合は、コレクションインスタンスに`searchable`メソッドを呼び出して、モデルインスタンスを対応するインデックスで更新できます。

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### インポート前のレコードの変更

検索可能にする前にモデルのコレクションを準備する必要がある場合があります。たとえば、関係を事前に読み込んで、関係データを効率的に検索インデックスに追加したい場合です。これを実現するには、対応するモデルに`makeSearchableUsing`メソッドを定義します。

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * 検索可能にするモデルのコレクションを変更する。
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

<a name="removing-records"></a>
### レコードの削除

インデックスからレコードを削除するには、データベースからモデルを単に`delete`するだけです。これは、[ソフトデリート](eloquent.md#soft-deleting)モデルを使用している場合でも可能です。

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

レコードを削除する前にモデルを取得したくない場合は、Eloquentクエリインスタンスに`unsearchable`メソッドを使用できます。

```php
Order::where('price', '>', 100)->unsearchable();
```

リレーション内のすべてのモデルの検索インデックスレコードを削除したい場合は、リレーションインスタンスに`unsearchable`を呼び出すことができます。

```php
$user->orders()->unsearchable();
```

または、メモリ内に既にEloquentモデルのコレクションがある場合は、コレクションインスタンスに`unsearchable`メソッドを呼び出して、モデルインスタンスを対応するインデックスから削除できます。

```php
$orders->unsearchable();
```

モデルレコードを対応するインデックスからすべて削除するには、`removeAllFromSearch`メソッドを呼び出すことができます。

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### インデックス作成の一時停止

モデルデータを検索インデックスに同期せずに、モデルに対してEloquent操作のバッチを実行する必要がある場合があります。`withoutSyncingToSearch`メソッドを使用してこれを行うことができます。このメソッドは、すぐに実行される単一のクロージャを受け取ります。クロージャ内で発生するモデル操作は、モデルのインデックスに同期されません。

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // モデル操作を実行...
});
```

<a name="conditionally-searchable-model-instances"></a>
### 条件付き検索可能なモデルインスタンス

特定の条件でのみモデルを検索可能にする必要がある場合があります。たとえば、`App\Models\Post`モデルが「下書き」と「公開済み」の2つの状態のいずれかになるとします。「公開済み」の投稿のみを検索可能にする場合があります。これを実現するには、モデルに`shouldBeSearchable`メソッドを定義します。

```php
/**
 * モデルが検索可能であるべきかどうかを判断する。
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

`shouldBeSearchable`メソッドは、`save`および`create`メソッド、クエリ、またはリレーションを介してモデルを操作する場合にのみ適用されます。`searchable`メソッドを使用してモデルまたはコレクションを直接検索可能にすると、`shouldBeSearchable`メソッドの結果が上書きされます。

> WARNING:  
> `shouldBeSearchable`メソッドは、Scoutの「データベース」エンジンを使用する場合には適用されません。検索可能なデータは常にデータベースに保存されるためです。データベースエンジンを使用する場合に同様の動作を実現するには、代わりに[where句](#where-clauses)を使用する必要があります。

<a name="searching"></a>
## 検索

モデルの検索を開始するには、`search`メソッドを使用します。検索メソッドは、モデルを検索するために使用される単一の文字列を受け取ります。その後、検索クエリに`get`メソッドをチェーンして、指定された検索クエリに一致するEloquentモデルを取得する必要があります。

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

Scoutの検索はEloquentモデルのコレクションを返すため、ルートまたはコントローラから直接結果を返すことができ、自動的にJSONに変換されます。

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

Eloquentモデルに変換される前に生の検索結果を取得したい場合は、`raw`メソッドを使用できます。

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### カスタムインデックス

検索クエリは通常、モデルの[`searchableAs`](#configuring-model-indexes)メソッドで指定されたインデックスで実行されます。ただし、代わりに検索するカスタムインデックスを指定するには、`within`メソッドを使用できます。

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Where句

Scoutを使用すると、検索クエリに単純な「where」句を追加できます。現在、これらの句は基本的な数値の等価性チェックのみをサポートしており、主に所有者IDによる検索クエリのスコープに役立ちます。検索インデックスはリレーショナルデータベースではないため、より高度な「where」句は現在サポートされていません。

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

`whereIn`メソッドを使用して、結果を特定の値のセットに制限できます。

```php
$orders = Order::search('Star Trek')->whereIn('status', ['paid', 'open'])->get();
```

Scoutのwhere句は、Algoliaのような検索エンジンでのみ機能します。Scoutの「コレクション」エンジンを使用する場合、`where`メソッドは無視されます。

Scoutでは、検索クエリにシンプルな「where」句を追加できます。現在、これらの句は基本的な数値の等価チェックのみをサポートしており、主に所有者IDによる検索クエリのスコープ指定に役立ちます。

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

さらに、`whereIn`メソッドを使用して、指定された列の値が指定された配列内に含まれていることを確認できます。

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

`whereNotIn`メソッドは、指定された列の値が指定された配列内に含まれていないことを確認します。

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

検索インデックスはリレーショナルデータベースではないため、より高度な「where」句は現在サポートされていません。

> WARNING:  
> アプリケーションがMeilisearchを使用している場合、Scoutの「where」句を利用する前に、アプリケーションの[フィルタ可能な属性を設定](#configuring-filterable-data-for-meilisearch)する必要があります。

<a name="pagination"></a>
### ページネーション

モデルのコレクションを取得するだけでなく、`paginate`メソッドを使用して検索結果をページネーションすることもできます。このメソッドは、[従来のEloquentクエリをページネーション](pagination.md)した場合と同様に、`Illuminate\Pagination\LengthAwarePaginator`インスタンスを返します。

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

`paginate`メソッドの最初の引数として、ページごとに取得するモデルの数を指定できます。

```php
$orders = Order::search('Star Trek')->paginate(15);
```

結果を取得したら、[Blade](blade.md)を使用して結果を表示し、ページリンクをレンダリングできます。従来のEloquentクエリをページネーションした場合と同様です。

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

もちろん、ページネーションの結果をJSONとして取得したい場合は、ルートまたはコントローラからページネータインスタンスを直接返すことができます。

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> WARNING:  
> 検索エンジンはEloquentモデルのグローバルスコープ定義を認識しないため、Scoutのページネーションを利用するアプリケーションでグローバルスコープを使用しないでください。または、Scoutで検索する際にグローバルスコープの制約を再現する必要があります。

<a name="soft-deleting"></a>
### ソフトデリート

インデックス付けされたモデルが[ソフトデリート](eloquent.md#soft-deleting)されており、ソフトデリートされたモデルを検索する必要がある場合は、`config/scout.php`設定ファイルの`soft_delete`オプションを`true`に設定します。

```php
'soft_delete' => true,
```

この設定オプションが`true`の場合、Scoutはソフトデリートされたモデルを検索インデックスから削除しません。代わりに、インデックス付けされたレコードに非表示の`__soft_deleted`属性を設定します。その後、検索時に`withTrashed`または`onlyTrashed`メソッドを使用して、ソフトデリートされたレコードを取得できます。

```php
use App\Models\Order;

// 結果を取得する際にソフトデリートされたレコードを含める...
$orders = Order::search('Star Trek')->withTrashed()->get();

// 結果を取得する際にソフトデリートされたレコードのみを含める...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> NOTE:  
> ソフトデリートされたモデルが`forceDelete`を使用して完全に削除されると、Scoutは自動的に検索インデックスからそれを削除します。

<a name="customizing-engine-searches"></a>
### エンジンの検索のカスタマイズ

エンジンの検索動作を高度にカスタマイズする必要がある場合、`search`メソッドの第2引数としてクロージャを渡すことができます。たとえば、このコールバックを使用して、検索クエリがAlgoliaに渡される前に、地理的位置データを検索オプションに追加できます。

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];

        return $algolia->search($query, $options);
    }
)->get();
```

<a name="customizing-the-eloquent-results-query"></a>
#### Eloquent結果クエリのカスタマイズ

Scoutがアプリケーションの検索エンジンから一致するEloquentモデルのリストを取得した後、Eloquentはそれらの主キーを使用してすべての一致するモデルを取得します。`query`メソッドを呼び出してこのクエリをカスタマイズできます。`query`メソッドは、Eloquentクエリビルダインスタンスを引数として受け取るクロージャを受け取ります。

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

このコールバックは、関連するモデルがすでにアプリケーションの検索エンジンから取得された後に呼び出されるため、`query`メソッドは結果の「フィルタリング」には使用しないでください。代わりに、[Scoutのwhere句](#where-clauses)を使用する必要があります。

<a name="custom-engines"></a>
## カスタムエンジン

<a name="writing-the-engine"></a>
#### エンジンの作成

組み込みのScout検索エンジンのいずれもニーズに合わない場合、独自のカスタムエンジンを作成し、Scoutに登録することができます。エンジンは`Laravel\Scout\Engines\Engine`抽象クラスを拡張する必要があります。この抽象クラスには、カスタムエンジンが実装する必要のある8つのメソッドが含まれています。

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

`Laravel\Scout\Engines\AlgoliaEngine`クラスでこれらのメソッドの実装を確認すると役立つでしょう。このクラスは、独自のエンジンでこれらのメソッドを実装するための良い出発点を提供します。

<a name="registering-the-engine"></a>
#### エンジンの登録

カスタムエンジンを作成したら、Scoutエンジンマネージャの`extend`メソッドを使用してScoutに登録できます。Scoutのエンジンマネージャは、Laravelサービスコンテナから解決できます。`App\Providers\AppServiceProvider`クラスまたはアプリケーションで使用される他のサービスプロバイダの`boot`メソッドから`extend`メソッドを呼び出す必要があります。

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * 任意のアプリケーションサービスのブートストラップ
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

エンジンを登録したら、アプリケーションの`config/scout.php`設定ファイルでデフォルトのScout`driver`として指定できます。

```php
'driver' => 'mysql',
```

