# Eloquent: はじめに

- [イントロダクション](#introduction)
- [モデルクラスの生成](#generating-model-classes)
- [Eloquentモデルの規約](#eloquent-model-conventions)
    - [テーブル名](#table-names)
    - [主キー](#primary-keys)
    - [UUIDとULIDキー](#uuid-and-ulid-keys)
    - [タイムスタンプ](#timestamps)
    - [データベース接続](#database-connections)
    - [デフォルトの属性値](#default-attribute-values)
    - [Eloquentの厳格性の設定](#configuring-eloquent-strictness)
- [モデルの取得](#retrieving-models)
    - [コレクション](#collections)
    - [結果のチャンク](#chunking-results)
    - [レイジーコレクションを使用したチャンク](#chunking-using-lazy-collections)
    - [カーソル](#cursors)
    - [高度なサブクエリ](#advanced-subqueries)
- [単一モデル / 集計の取得](#retrieving-single-models)
    - [モデルの取得または作成](#retrieving-or-creating-models)
    - [集計の取得](#retrieving-aggregates)
- [モデルの挿入と更新](#inserting-and-updating-models)
    - [挿入](#inserts)
    - [更新](#updates)
    - [マスアサインメント](#mass-assignment)
    - [アップサート](#upserts)
- [モデルの削除](#deleting-models)
    - [ソフトデリート](#soft-deleting)
    - [ソフトデリートされたモデルのクエリ](#querying-soft-deleted-models)
- [モデルの整理](#pruning-models)
- [モデルの複製](#replicating-models)
- [クエリスコープ](#query-scopes)
    - [グローバルスコープ](#global-scopes)
    - [ローカルスコープ](#local-scopes)
- [モデルの比較](#comparing-models)
- [イベント](#events)
    - [クロージャの使用](#events-using-closures)
    - [オブザーバー](#observers)
    - [イベントの無効化](#muting-events)

<a name="introduction"></a>
## イントロダクション

LaravelにはEloquentが含まれており、これはデータベースとのやり取りを楽しくするオブジェクト関係マッパー（ORM）です。Eloquentを使用する場合、各データベーステーブルにはそのテーブルとやり取りするために使用される対応する「モデル」があります。データベーステーブルからレコードを取得するだけでなく、Eloquentモデルを使用してテーブルからレコードを挿入、更新、削除することもできます。

> NOTE:  
> 始める前に、アプリケーションの`config/database.php`設定ファイルでデータベース接続を設定してください。データベースの設定について詳しくは、[データベース設定ドキュメント](database.md#configuration)を確認してください。

#### Laravel Bootcamp

Laravelを初めて使う方は、[Laravel Bootcamp](https://bootcamp.laravel.com)に飛び込んでみてください。Laravel Bootcampでは、Eloquentを使用して最初のLaravelアプリケーションを構築する手順を説明します。LaravelとEloquentが提供するすべての機能を紹介するのに最適な方法です。

<a name="generating-model-classes"></a>
## モデルクラスの生成

始めるには、Eloquentモデルを作成しましょう。モデルは通常`app\Models`ディレクトリにあり、`Illuminate\Database\Eloquent\Model`クラスを拡張します。新しいモデルを生成するには、`make:model` [Artisanコマンド](artisan.md)を使用できます:

```shell
php artisan make:model Flight
```

モデルを生成する際に[データベースマイグレーション](migrations.md)も一緒に作成したい場合は、`--migration`または`-m`オプションを使用できます:

```shell
php artisan make:model Flight --migration
```

モデルを生成する際に、ファクトリー、シーダー、ポリシー、コントローラー、フォームリクエストなど、さまざまな種類のクラスを生成できます。さらに、これらのオプションを組み合わせて一度に複数のクラスを作成することもできます:

```shell
# モデルとFlightFactoryクラスを生成する...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# モデルとFlightSeederクラスを生成する...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# モデルとFlightControllerクラスを生成する...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# モデル、FlightControllerリソースクラス、フォームリクエストクラスを生成する...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# モデルとFlightPolicyクラスを生成する...
php artisan make:model Flight --policy

# モデルとマイグレーション、ファクトリー、シーダー、コントローラーを生成する...
php artisan make:model Flight -mfsc

# モデル、マイグレーション、ファクトリー、シーダー、ポリシー、コントローラー、フォームリクエストを生成するショートカット...
php artisan make:model Flight --all
php artisan make:model Flight -a

# ピボットモデルを生成する...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

<a name="inspecting-models"></a>
#### モデルの検査

モデルのコードをスキャンするだけでは、モデルの利用可能なすべての属性とリレーションを判断するのが難しい場合があります。代わりに、`model:show` Artisanコマンドを試してみてください。これにより、モデルのすべての属性とリレーションの便利な概要が提供されます:

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Eloquentモデルの規約

`make:model`コマンドによって生成されたモデルは、`app/Models`ディレクトリに配置されます。基本的なモデルクラスを見て、Eloquentのいくつかの主要な規約について説明しましょう:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        // ...
    }

<a name="table-names"></a>
### テーブル名

上記の例では、`Flight`モデルに対応するデータベーステーブルを明示的に指定していないことにお気づきでしょう。慣例により、クラスの「スネークケース」の複数形の名前がテーブル名として使用されます。ただし、別の名前が明示的に指定されていない限りです。したがって、この場合、Eloquentは`Flight`モデルが`flights`テーブルにレコードを格納し、`AirTrafficController`モデルが`air_traffic_controllers`テーブルにレコードを格納すると仮定します。

モデルに対応するデータベーステーブルがこの規約に適合しない場合は、モデルに`table`プロパティを定義して、モデルのテーブル名を手動で指定できます:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * モデルに関連付けられたテーブル。
         *
         * @var string
         */
        protected $table = 'my_flights';
    }

<a name="primary-keys"></a>
### 主キー

Eloquentは、各モデルの対応するデータベーステーブルに`id`という名前の主キーカラムがあると仮定します。必要に応じて、モデルに`$primaryKey`プロパティを定義して、モデルの主キーとして別のカラムを指定できます:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class Flight extends Model
    {
        /**
         * テーブルに関連付けられた主キー。
         *
         * @var string
         */
        protected $primaryKey = 'flight_id';
    }

さらに、Eloquentは主キーが増分する整数値であると仮定します。つまり、Eloquentは自動的に主キーを整数にキャストします。非増分または非数値の主キーを使用する場合は、モデルに`$incrementing`プロパティを定義し、`false`に設定する必要があります:

    <?php

    class Flight extends Model
    {
        /**
         * モデルのIDが自動増分するかどうかを示す。
         *
         * @var bool
         */
        public $incrementing = false;
    }

モデルの主キーが整数でない場合は、モデルに`$keyType`プロパティを定義する必要があります。このプロパティの値は`string`である必要があります:

    <?php

    class Flight extends Model
    {
        /**
         * 主キーIDのデータ型。
         *
         * @var string
         */
        protected $keyType = 'string';
    }

<a name="composite-primary-keys"></a>
#### 「複合」主キー

Eloquentは各モデルに少なくとも1つの一意に識別できる「ID」を持つことを要求します。これは主キーとして機能します。Eloquentモデルは「複合」主キーをサポートしていません。ただし、テーブルの一意に識別する主キーに加えて、テーブルに追加の複数列の一意インデックスを自由に追加できます。

<a name="uuid-and-ulid-keys"></a>
### UUIDとULIDキー

Eloquentモデルの主キーとして自動増分する整数を使用する代わりに、UUIDを使用することもできます。UUIDは、36文字の長さのユニバーサルユニークなアルファニューメリック識別子です。

モデルにUUIDキーを使用する場合は、モデルに`Illuminate\Database\Eloquent\Concerns\HasUuids`トレイトを使用できます。もちろん、モデルに[UUID相当の主キーカラム](migrations.md#column-method-uuid)があることを確認する必要があります:

    use Illuminate\Database\Eloquent\Concerns\HasUuids;
    use Illuminate\Database\Eloquent\Model;

    class Article extends Model
    {
        use HasUuids;

        // ...
    }

    $article = Article::create(['title' => 'Traveling to Europe']);

    $article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"

デフォルトでは、`HasUuids`トレイトはモデルに["ordered" UUID](strings.md#method-str-ordered-uuid)を生成します。これらのUUIDは、辞書順にソートできるため、インデックス付きのデータベースストレージに効率的です。

特定のモデルのUUID生成プロセスをオーバーライドするには、モデルに`newUniqueId`メソッドを定義します。さらに、モデルに`uniqueIds`メソッドを定義して、UUIDを受け取るべきカラムを指定できます:

    use Ramsey\Uuid\Uuid;

    /**
     * モデルの新しいUUIDを生成する。
     */
    public function newUniqueId(): string
    {
        return (string) Uuid::uuid4();
    }

    /**
     * 一意の識別子を受け取るべきカラムを取得する。
     *
     * @return array<int, string>
     */
    public function uniqueIds(): array
    {
        return ['id', 'discount_code'];
    }

もし希望するなら、UUIDの代わりに「ULID」を利用することもできます。ULIDはUUIDに似ていますが、長さは26文字です。順序付きUUIDと同様に、ULIDはデータベースのインデックス作成に効率的な辞書順にソート可能です。ULIDを利用するには、モデルに`Illuminate\Database\Eloquent\Concerns\HasUlids`トレイトを使用する必要があります。また、モデルが[ULIDに相当する主キーカラム](migrations.md#column-method-ulid)を持つことを確認する必要があります。

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

<a name="timestamps"></a>
### タイムスタンプ

デフォルトでは、Eloquentはモデルに対応するデータベーステーブルに`created_at`と`updated_at`カラムが存在することを期待します。Eloquentは、モデルが作成または更新されるときにこれらのカラムの値を自動的に設定します。これらのカラムをEloquentで自動的に管理したくない場合は、モデルに`$timestamps`プロパティを`false`の値で定義する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * モデルにタイムスタンプを付けるかどうかを示す
     *
     * @var bool
     */
    public $timestamps = false;
}
```

モデルのタイムスタンプのフォーマットをカスタマイズする必要がある場合は、モデルに`$dateFormat`プロパティを設定します。このプロパティは、日付属性がデータベースに保存される方法と、モデルが配列またはJSONにシリアライズされるときのフォーマットを決定します。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * モデルの日付カラムの保存フォーマット
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

タイムスタンプを保存するために使用されるカラムの名前をカスタマイズする必要がある場合は、モデルに`CREATED_AT`と`UPDATED_AT`定数を定義できます。

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}
```

モデルの`updated_at`タイムスタンプを変更せずにモデル操作を実行したい場合は、`withoutTimestamps`メソッドに渡されたクロージャ内でモデルを操作できます。

```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

<a name="database-connections"></a>
### データベース接続

デフォルトでは、すべてのEloquentモデルはアプリケーションに設定されたデフォルトのデータベース接続を使用します。特定のモデルとのやり取り時に別の接続を使用する場合は、モデルに`$connection`プロパティを定義する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * モデルで使用されるデータベース接続
     *
     * @var string
     */
    protected $connection = 'mysql';
}
```

<a name="default-attribute-values"></a>
### デフォルトの属性値

新しくインスタンス化されたモデルインスタンスには、デフォルトで属性値が含まれません。モデルの一部の属性にデフォルト値を定義したい場合は、モデルに`$attributes`プロパティを定義できます。`$attributes`配列に配置された属性値は、データベースから読み取られたかのように、そのままの「保存可能な」形式である必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * モデルの属性のデフォルト値
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

<a name="configuring-eloquent-strictness"></a>
### Eloquentの厳格性の設定

Laravelは、さまざまな状況でEloquentの動作と「厳格性」を設定できるいくつかのメソッドを提供しています。

まず、`preventLazyLoading`メソッドは、遅延読み込みを防止するかどうかを示すオプションのブール値引数を受け入れます。たとえば、本番環境では遅延読み込みを無効にせず、本番コードに誤って遅延読み込みされた関係が存在しても本番環境が正常に機能し続けるように、非本番環境でのみ遅延読み込みを無効にすることができます。通常、このメソッドはアプリケーションの`AppServiceProvider`の`boot`メソッドで呼び出す必要があります。

```php
use Illuminate\Database\Eloquent\Model;

/**
 * 任意のアプリケーションサービスをブートストラップする
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

また、`preventSilentlyDiscardingAttributes`メソッドを呼び出すことで、設定不可能な属性を設定しようとしたときに例外をスローするようにLaravelに指示できます。これにより、ローカル開発中にモデルの`fillable`配列に追加されていない属性を設定しようとしたときに、予期せぬエラーを防ぐことができます。

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

<a name="retrieving-models"></a>
## モデルの取得

モデルを作成し、[それに関連するデータベーステーブル](migrations.md#generating-migrations)を作成したら、データベースからデータを取得する準備が整いました。各Eloquentモデルを、モデルに関連するデータベーステーブルを流暢にクエリできる強力な[クエリビルダ](queries.md)と考えることができます。モデルの`all`メソッドは、モデルに関連するデータベーステーブルからすべてのレコードを取得します。

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

<a name="building-queries"></a>
#### クエリの構築

Eloquentの`all`メソッドは、モデルのテーブルからすべての結果を返します。ただし、各Eloquentモデルは[クエリビルダ](queries.md)として機能するため、クエリに追加の制約を追加し、`get`メソッドを呼び出して結果を取得できます。

```php
$flights = Flight::where('active', 1)
               ->orderBy('name')
               ->take(10)
               ->get();
```

> NOTE:  
> Eloquentモデルはクエリビルダであるため、Laravelの[クエリビルダ](queries.md)が提供するすべてのメソッドを確認する必要があります。Eloquentクエリを記述する際に、これらのメソッドを使用できます。

<a name="refreshing-models"></a>
#### モデルのリフレッシュ

データベースから取得したEloquentモデルのインスタンスがすでにある場合、`fresh`および`refresh`メソッドを使用してモデルを「リフレッシュ」できます。`fresh`メソッドは、モデルをデータベースから再取得します。既存のモデルインスタンスには影響しません。

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh`メソッドは、データベースからの新鮮なデータを使用して既存のモデルを再ハイドレートします。さらに、そのすべての読み込まれたリレーションもリフレッシュされます。

```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### コレクション

これまで見てきたように、Eloquentの`all`や`get`などのメソッドは、データベースから複数のレコードを取得します。ただし、これらのメソッドはプレーンなPHP配列を返すのではなく、`Illuminate\Database\Eloquent\Collection`のインスタンスを返します。

Eloquentの`Collection`クラスは、Laravelの基本`Illuminate\Support\Collection`クラスを拡張しており、データコレクションと対話するための[さまざまな便利なメソッド](collections.md#available-methods)を提供します。たとえば、`reject`メソッドを使用して、呼び出されたクロージャの結果に基づいてコレクションからモデルを削除できます。

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

Laravelの基本コレクションクラスが提供するメソッドに加えて、Eloquentコレクションクラスは、Eloquentモデルのコレクションと対話するために特に意図された[いくつかの追加メソッド](eloquent-collections.md#available-methods)を提供します。

LaravelのすべてのコレクションはPHPの反復可能なインターフェースを実装しているため、コレクションを配列のようにループすることができます。

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### 結果のチャンキング

`all`または`get`メソッドを使用して数万のEloquentレコードを読み込もうとすると、アプリケーションのメモリが不足する可能性があります。これらのメソッドの代わりに、`chunk`メソッドを使用して大量のモデルをより効率的に処理できます。

`chunk`メソッドは、Eloquentモデルのサブセットを取得し、それらをクロージャに渡して処理します。現在のチャンクのEloquentモデルのみが一度に取得されるため、`chunk`メソッドは大量のモデルを扱う際に大幅にメモリ使用量を削減します。

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

`chunk`メソッドに渡される最初の引数は、各「チャンク」で受け取りたいレコードの数です。クロージャとして渡される2番目の引数は、データベースから取得された各チャンクに対して呼び出されます。データベースクエリは、クロージャに渡される各チャンクのレコードを取得するために実行されます。

`chunk`メソッドの結果をフィルタリングする際に、イテレーション中に更新するカラムに基づいてフィルタリングする場合は、`chunkById`メソッドを使用するべきです。このようなシナリオで`chunk`メソッドを使用すると、予期しない結果や一貫性のない結果を招く可能性があります。内部的には、`chunkById`メソッドは常に前のチャンクの最後のモデルよりも`id`カラムが大きいモデルを取得します。

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, $column = 'id');
```

<a name="chunking-using-lazy-collections"></a>
### 遅延コレクションを使用したチャンク処理

`lazy`メソッドは、[`chunk`メソッド](#chunking-results)と同様に、内部的にはクエリをチャンクで実行します。ただし、各チャンクを直接コールバックに渡すのではなく、`lazy`メソッドはEloquentモデルのフラット化された[`LazyCollection`](collections.md#lazy-collections)を返します。これにより、結果を単一のストリームとして操作できます。

```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

`lazy`メソッドの結果をフィルタリングする際に、イテレーション中に更新するカラムに基づいてフィルタリングする場合は、`lazyById`メソッドを使用するべきです。内部的には、`lazyById`メソッドは常に前のチャンクの最後のモデルよりも`id`カラムが大きいモデルを取得します。

```php
Flight::where('departed', true)
    ->lazyById(200, $column = 'id')
    ->each->update(['departed' => false]);
```

`lazyByIdDesc`メソッドを使用して、`id`の降順で結果をフィルタリングすることもできます。

<a name="cursors"></a>
### カーソル

`lazy`メソッドと同様に、`cursor`メソッドは、何万ものEloquentモデルレコードを反復処理する際に、アプリケーションのメモリ消費を大幅に削減するために使用できます。

`cursor`メソッドは、単一のデータベースクエリのみを実行します。ただし、個々のEloquentモデルは、実際に反復処理されるまでハイドレートされません。したがって、反復処理中には常に1つのEloquentモデルのみがメモリに保持されます。

> WARNING:  
> `cursor`メソッドは常に1つのEloquentモデルのみをメモリに保持するため、リレーションを積極的にロードすることはできません。リレーションを積極的にロードする必要がある場合は、代わりに[`lazy`メソッド](#chunking-using-lazy-collections)を使用することを検討してください。

内部的には、`cursor`メソッドはPHPの[ジェネレータ](https://www.php.net/manual/en/language.generators.overview.php)を使用してこの機能を実装しています。

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

`cursor`は`Illuminate\Support\LazyCollection`インスタンスを返します。[遅延コレクション](collections.md#lazy-collections)を使用すると、通常のLaravelコレクションで利用可能な多くのコレクションメソッドを使用できますが、一度に1つのモデルのみをメモリにロードします。

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

`cursor`メソッドは通常のクエリよりもはるかに少ないメモリを使用します（一度に1つのEloquentモデルのみをメモリに保持するため）が、最終的にはメモリ不足になる可能性があります。これは、[PHPのPDOドライバが内部的にすべての生のクエリ結果をバッファにキャッシュするため](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php)です。非常に多数のEloquentレコードを扱う場合は、代わりに[`lazy`メソッド](#chunking-using-lazy-collections)を使用することを検討してください。

<a name="advanced-subqueries"></a>
### 高度なサブクエリ

<a name="subquery-selects"></a>
#### サブクエリの選択

Eloquentは、高度なサブクエリサポートも提供しています。これにより、単一のクエリで関連テーブルから情報を引き出すことができます。例えば、フライトの`destinations`テーブルと`flights`テーブルがあるとします。`flights`テーブルには、目的地に到着した時間を示す`arrived_at`カラムがあります。

クエリビルダの`select`および`addSelect`メソッドで利用可能なサブクエリ機能を使用すると、単一のクエリですべての`destinations`と、その目的地に最後に到着したフライトの名前を選択できます。

```php
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```

<a name="subquery-ordering"></a>
#### サブクエリによる順序付け

さらに、クエリビルダの`orderBy`関数はサブクエリをサポートしています。フライトの例を続けると、この機能を使用して、最後のフライトが到着した時間に基づいてすべての目的地を並べ替えることができます。これも、単一のデータベースクエリで実行できます。

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

<a name="retrieving-single-models"></a>
## 単一モデル / 集計の取得

特定のクエリに一致するすべてのレコードを取得するだけでなく、`find`、`first`、または`firstWhere`メソッドを使用して単一のレコードを取得することもできます。これらのメソッドは、モデルのコレクションではなく、単一のモデルインスタンスを返します。

```php
use App\Models\Flight;

// 主キーでモデルを取得...
$flight = Flight::find(1);

// クエリ制約に一致する最初のモデルを取得...
$flight = Flight::where('active', 1)->first();

// クエリ制約に一致する最初のモデルを取得する代替方法...
$flight = Flight::firstWhere('active', 1);
```

結果が見つからない場合に何らかのアクションを実行したい場合があります。`findOr`および`firstOr`メソッドは、単一のモデルインスタンスを返すか、結果が見つからない場合に指定されたクロージャを実行します。クロージャが返す値は、メソッドの結果と見なされます。

```php
$flight = Flight::findOr(1, function () {
    // ...
});

$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```

<a name="not-found-exceptions"></a>
#### 見つからない例外

モデルが見つからない場合に例外をスローしたい場合があります。これは、特にルートやコントローラで役立ちます。`findOrFail`および`firstOrFail`メソッドは、クエリの最初の結果を取得しますが、結果が見つからない場合は`Illuminate\Database\Eloquent\ModelNotFoundException`がスローされます。

```php
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

`ModelNotFoundException`がキャッチされない場合、404 HTTPレスポンスが自動的にクライアントに返されます。

```php
use App\Models\Flight;

Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});
```

<a name="retrieving-or-creating-models"></a>
### モデルの取得または作成

`firstOrCreate`メソッドは、指定されたカラム/値のペアを使用してデータベースレコードを検索しようとします。モデルがデータベースに見つからない場合、最初の配列引数とオプションの2番目の配列引数をマージした属性を持つレコードが挿入されます。

`firstOrNew`メソッドは、`firstOrCreate`と同様に、指定された属性に一致するレコードをデータベースで検索しようとします。ただし、モデルが見つからない場合、新しいモデルインスタンスが返されます。`firstOrNew`によって返されるモデルは、まだデータベースに保存されていません。保存するには、手動で`save`メソッドを呼び出す必要があります。

```php
use App\Models\Flight;

// 名前でフライトを取得するか、存在しない場合は作成...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// 名前でフライトを取得するか、名前、遅延、到着時間の属性で作成...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// 名前でフライトを取得するか、新しいFlightインスタンスをインスタンス化...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// 名前でフライトを取得するか、名前、遅延、到着時間の属性でインスタンス化...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

<a name="retrieving-aggregates"></a>
### 集計の取得

Eloquentモデルを操作する際に、Laravelの[クエリビルダ](queries.md)が提供する`count`、`sum`、`max`、その他の[集計メソッド](queries.md#aggregates)を使用することもできます。これらのメソッドは、Eloquentモデルインスタンスではなく、スカラー値を返すことが期待されます。

```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## モデルの挿入と更新

<a name="inserts"></a>
### 挿入

もちろん、Eloquentを使用する場合、データベースからモデルを取得するだけでなく、新しいレコードを挿入する必要もあります。幸いなことに、Eloquentはこれを簡単にしてくれます。データベースに新しいレコードを挿入するには、新しいモデルインスタンスをインスタンス化し、モデルに属性を設定します。そして、モデルインスタンスの`save`メソッドを呼び出します。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * データベースに新しいフライトを保存します。
     */
    public function store(Request $request): RedirectResponse
    {
        // リクエストの検証...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
}
```

```php
$flight->name = $request->name;

$flight->save();

return redirect('/flights');
```

この例では、受信したHTTPリクエストからの`name`フィールドを`App\Models\Flight`モデルインスタンスの`name`属性に代入しています。`save`メソッドを呼び出すと、データベースにレコードが挿入されます。モデルの`created_at`と`updated_at`タイムスタンプは、`save`メソッドが呼び出されたときに自動的に設定されるため、手動で設定する必要はありません。

あるいは、`create`メソッドを使用して、1つのPHPステートメントで新しいモデルを「保存」することもできます。`create`メソッドによって挿入されたモデルインスタンスは、`create`メソッドによって返されます。

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

ただし、`create`メソッドを使用する前に、モデルクラスに`fillable`または`guarded`プロパティを指定する必要があります。これらのプロパティは、すべてのEloquentモデルがデフォルトでマスアサインメントの脆弱性から保護されているために必要です。マスアサインメントについて詳しく知りたい場合は、[マスアサインメントのドキュメント](#mass-assignment)を参照してください。

<a name="updates"></a>
### 更新

`save`メソッドは、データベースに既に存在するモデルを更新するためにも使用できます。モデルを更新するには、それを取得し、更新したい属性を設定します。その後、モデルの`save`メソッドを呼び出します。この場合も、`updated_at`タイムスタンプは自動的に更新されるため、手動で設定する必要はありません。

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

時には、既存のモデルを更新するか、一致するモデルが存在しない場合に新しいモデルを作成する必要があるかもしれません。`firstOrCreate`メソッドと同様に、`updateOrCreate`メソッドはモデルを永続化するため、手動で`save`メソッドを呼び出す必要はありません。

以下の例では、`departure`が`Oakland`で`destination`が`San Diego`のフライトが存在する場合、その`price`と`discounted`カラムが更新されます。そのようなフライトが存在しない場合、最初の引数の配列と2番目の引数の配列をマージした属性を持つ新しいフライトが作成されます。

```php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

<a name="mass-updates"></a>
#### マス更新

特定のクエリに一致するモデルに対しても更新を実行できます。この例では、`active`で`destination`が`San Diego`のすべてのフライトが遅延としてマークされます。

```php
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```

`update`メソッドは、更新するカラムと値のペアの配列を期待します。`update`メソッドは、影響を受けた行数を返します。

> WARNING:  
> Eloquentを介してマス更新を発行する場合、更新されたモデルに対して`saving`、`saved`、`updating`、`updated`モデルイベントは発生しません。これは、マス更新を発行する際にモデルが実際に取得されることがないためです。

<a name="examining-attribute-changes"></a>
#### 属性の変更を調べる

Eloquentは、モデルの内部状態を調べ、その属性が最初に取得されてからどのように変化したかを判断するための`isDirty`、`isClean`、`wasChanged`メソッドを提供します。

`isDirty`メソッドは、モデルが取得されてからモデルの属性のいずれかが変更されたかどうかを判断します。特定の属性名または属性の配列を`isDirty`メソッドに渡して、それらの属性のいずれかが「ダーティ」であるかどうかを判断できます。`isClean`メソッドは、モデルが取得されてから属性が変更されていないかどうかを判断します。このメソッドもオプションの属性引数を受け入れます。

```php
use App\Models\User;

$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true

$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false

$user->save();

$user->isDirty(); // false
$user->isClean(); // true
```

`wasChanged`メソッドは、現在のリクエストサイクル内でモデルが最後に保存されたときに属性が変更されたかどうかを判断します。必要に応じて、特定の属性が変更されたかどうかを確認するために属性名を渡すことができます。

```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->save();

$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```

`getOriginal`メソッドは、モデルが取得されてからの変更に関係なく、モデルの元の属性を含む配列を返します。必要に応じて、特定の属性の元の値を取得するため、属性名を渡すことができます。

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = "Jack";
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // 元の属性の配列...
```

<a name="mass-assignment"></a>
### マスアサインメント

`create`メソッドを使用して、1つのPHPステートメントで新しいモデルを「保存」することができます。挿入されたモデルインスタンスは、メソッドによって返されます。

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

ただし、`create`メソッドを使用する前に、モデルクラスに`fillable`または`guarded`プロパティを指定する必要があります。これらのプロパティは、すべてのEloquentモデルがデフォルトでマスアサインメントの脆弱性から保護されているために必要です。

マスアサインメントの脆弱性は、ユーザーが意図しないHTTPリクエストフィールドを送信し、そのフィールドによって予期せぬデータベースカラムが変更される場合に発生します。例えば、悪意のあるユーザーがHTTPリクエストを介して`is_admin`パラメータを送信し、それがモデルの`create`メソッドに渡されると、ユーザーは自分自身を管理者に昇格させることができます。

そのため、始めるには、どのモデル属性をマスアサイン可能にするかを定義する必要があります。これは、モデルの`$fillable`プロパティを使用して行うことができます。例えば、`Flight`モデルの`name`属性をマスアサイン可能にしましょう。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * マスアサイン可能な属性。
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```

マスアサイン可能な属性を指定したら、`create`メソッドを使用してデータベースに新しいレコードを挿入できます。`create`メソッドは、新しく作成されたモデルインスタンスを返します。

```php
$flight = Flight::create(['name' => 'London to Paris']);
```

既にモデルインスタンスがある場合は、`fill`メソッドを使用して属性の配列でそれを埋めることができます。

```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

<a name="mass-assignment-json-columns"></a>
#### マスアサインメントとJSONカラム

JSONカラムを割り当てる場合、各カラムのマスアサイン可能なキーをモデルの`$fillable`配列に指定する必要があります。セキュリティ上の理由から、Laravelは`guarded`プロパティを使用する場合にネストされたJSON属性の更新をサポートしていません。

```php
/**
 * マスアサイン可能な属性。
 *
 * @var array
 */
protected $fillable = [
    'options->enabled',
];
```

<a name="allowing-mass-assignment"></a>
#### マスアサインメントの許可

すべての属性をマスアサイン可能にする場合は、モデルの`$guarded`プロパティを空の配列として定義できます。モデルをアンガードすることを選択した場合、Eloquentの`fill`、`create`、`update`メソッドに渡される配列を常に手動で作成するように特別な注意を払う必要があります。

```php
/**
 * マスアサイン不可能な属性。
 *
 * @var array
 */
protected $guarded = [];
```

<a name="mass-assignment-exceptions"></a>
#### マスアサインメントの例外

デフォルトでは、`$fillable`配列に含まれていない属性は、マスアサインメント操作を実行する際に静かに破棄されます。本番環境ではこれが期待される動作ですが、ローカル開発中には、モデルの変更が効果を発揮しない理由について混乱を招く可能性があります。

必要に応じて、Laravelに、フィル可能でない属性を試みて埋める際に例外をスローするように指示できます。これは、アプリケーションの`AppServiceProvider`クラスの`boot`メソッドで`preventSilentlyDiscardingAttributes`メソッドを呼び出すことで実行できます。

```php
use Illuminate\Database\Eloquent\Model;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
}
```

<a name="upserts"></a>
### アップサート

Eloquentの`upsert`メソッドは、単一のアトミック操作でレコードを更新または作成するために使用できます。メソッドの最初の引数は、挿入または更新する値で構成され、2番目の引数は、関連するテーブル内でレコードを一意に識別するカラムのリストです。メソッドの3番目の引数は、データベース内に一致するレコードが既に存在する場合に更新する必要があるカラムの配列です。`upsert`メソッドは、モデルでタイムスタンプが有効になっている場合、自動的に`created_at`と`updated_at`タイムスタンプを設定します。

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], ['departure', 'destination'], ['price']);
```

    Flight::upsert([
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ], uniqueBy: ['departure', 'destination'], update: ['price']);

> WARNING:  
> `upsert`メソッドの2番目の引数にある列には、SQL Serverを除くすべてのデータベースで「主キー」または「ユニーク」インデックスが必要です。さらに、MariaDBおよびMySQLデータベースドライバは、`upsert`メソッドの2番目の引数を無視し、常にテーブルの「主キー」と「ユニーク」インデックスを使用して既存のレコードを検出します。

<a name="deleting-models"></a>
## モデルの削除

モデルを削除するには、モデルインスタンスで`delete`メソッドを呼び出すことができます:

    use App\Models\Flight;

    $flight = Flight::find(1);

    $flight->delete();

モデルに関連するすべてのデータベースレコードを削除するには、`truncate`メソッドを呼び出すことができます。`truncate`操作は、モデルに関連するテーブルの自動インクリメントIDもリセットします:

    Flight::truncate();

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### 主キーによる既存モデルの削除

上記の例では、`delete`メソッドを呼び出す前にデータベースからモデルを取得しています。ただし、モデルの主キーがわかっている場合は、`destroy`メソッドを呼び出すことで、モデルを明示的に取得せずにモデルを削除できます。`destroy`メソッドは、単一の主キー、複数の主キー、主キーの配列、または主キーの[コレクション](collections.md)を受け入れます:

    Flight::destroy(1);

    Flight::destroy(1, 2, 3);

    Flight::destroy([1, 2, 3]);

    Flight::destroy(collect([1, 2, 3]));

[ソフトデリートモデル](#soft-deleting)を利用している場合、`forceDestroy`メソッドを介してモデルを完全に削除できます:

    Flight::forceDestroy(1);

> WARNING:  
> `destroy`メソッドは各モデルを個別にロードし、`delete`メソッドを呼び出すため、各モデルに対して`deleting`および`deleted`イベントが適切に発行されます。

<a name="deleting-models-using-queries"></a>
#### クエリを使用したモデルの削除

もちろん、クエリの条件に一致するすべてのモデルを削除するためにEloquentクエリを構築できます。この例では、非アクティブとマークされたすべてのフライトを削除します。マス更新と同様に、マス削除は削除されたモデルに対してモデルイベントを発行しません:

    $deleted = Flight::where('active', 0)->delete();

> WARNING:  
> Eloquentを介してマス削除ステートメントを実行する場合、削除されたモデルに対して`deleting`および`deleted`モデルイベントは発行されません。これは、削除ステートメントの実行時にモデルが実際に取得されることがないためです。

<a name="soft-deleting"></a>
### ソフトデリート

データベースから実際にレコードを削除するだけでなく、Eloquentはモデルを「ソフトデリート」することもできます。モデルがソフトデリートされると、実際にはデータベースから削除されません。代わりに、モデルが「削除」された日時を示す`deleted_at`属性がモデルに設定されます。モデルのソフトデリートを有効にするには、モデルに`Illuminate\Database\Eloquent\SoftDeletes`トレイトを追加します:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\SoftDeletes;

    class Flight extends Model
    {
        use SoftDeletes;
    }

> NOTE:  
> `SoftDeletes`トレイトは、`deleted_at`属性を自動的に`DateTime` / `Carbon`インスタンスにキャストします。

また、データベーステーブルに`deleted_at`列を追加する必要があります。Laravelの[スキーマビルダ](migrations.md)には、この列を作成するためのヘルパーメソッドが含まれています:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('flights', function (Blueprint $table) {
        $table->softDeletes();
    });

    Schema::table('flights', function (Blueprint $table) {
        $table->dropSoftDeletes();
    });

これで、モデルで`delete`メソッドを呼び出すと、`deleted_at`列に現在の日時が設定されます。ただし、モデルのデータベースレコードはテーブルに残ります。ソフトデリートを使用するモデルをクエリする場合、ソフトデリートされたモデルは自動的にすべてのクエリ結果から除外されます。

特定のモデルインスタンスがソフトデリートされたかどうかを判断するには、`trashed`メソッドを使用できます:

    if ($flight->trashed()) {
        // ...
    }

<a name="restoring-soft-deleted-models"></a>
#### ソフトデリートされたモデルの復元

ソフトデリートされたモデルを「復元」したい場合があります。ソフトデリートされたモデルを復元するには、モデルインスタンスで`restore`メソッドを呼び出すことができます。`restore`メソッドは、モデルの`deleted_at`列を`null`に設定します:

    $flight->restore();

クエリで`restore`メソッドを使用して複数のモデルを復元することもできます。他の「マス」操作と同様に、これにより復元されたモデルに対してモデルイベントは発行されません:

    Flight::withTrashed()
            ->where('airline_id', 1)
            ->restore();

[リレーション](eloquent-relationships.md)クエリを構築する際にも`restore`メソッドを使用できます:

    $flight->history()->restore();

<a name="permanently-deleting-models"></a>
#### モデルの完全削除

データベースからモデルを完全に削除する必要がある場合があります。ソフトデリートされたモデルをデータベーステーブルから完全に削除するには、`forceDelete`メソッドを使用できます:

    $flight->forceDelete();

Eloquentリレーションクエリを構築する際にも`forceDelete`メソッドを使用できます:

    $flight->history()->forceDelete();

<a name="querying-soft-deleted-models"></a>
### ソフトデリートされたモデルのクエリ

<a name="including-soft-deleted-models"></a>
#### ソフトデリートされたモデルを含める

前述のように、ソフトデリートされたモデルは自動的にクエリ結果から除外されます。ただし、クエリで`withTrashed`メソッドを呼び出すことで、ソフトデリートされたモデルをクエリ結果に含めることができます:

    use App\Models\Flight;

    $flights = Flight::withTrashed()
                    ->where('account_id', 1)
                    ->get();

[リレーション](eloquent-relationships.md)クエリを構築する際にも`withTrashed`メソッドを呼び出すことができます:

    $flight->history()->withTrashed()->get();

<a name="retrieving-only-soft-deleted-models"></a>
#### ソフトデリートされたモデルのみを取得

`onlyTrashed`メソッドは、**ソフトデリートされたモデルのみ**を取得します:

    $flights = Flight::onlyTrashed()
                    ->where('airline_id', 1)
                    ->get();

<a name="pruning-models"></a>
## モデルの整理

不要になったモデルを定期的に削除したい場合があります。これを実現するには、定期的に整理したいモデルに`Illuminate\Database\Eloquent\Prunable`または`Illuminate\Database\Eloquent\MassPrunable`トレイトを追加します。モデルにトレイトを追加した後、不要になったモデルを解決するEloquentクエリビルダを返す`prunable`メソッドを実装します:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Prunable;

    class Flight extends Model
    {
        use Prunable;

        /**
         * 整理可能なモデルクエリを取得します。
         */
        public function prunable(): Builder
        {
            return static::where('created_at', '<=', now()->subMonth());
        }
    }

`Prunable`としてモデルをマークする場合、モデルに`pruning`メソッドを定義することもできます。このメソッドは、モデルが削除される前に呼び出されます。このメソッドは、モデルがデータベースから完全に削除される前に、モデルに関連する追加のリソース（例えば、保存されたファイル）を削除するのに役立ちます:

    /**
     * モデルの整理の準備をします。
     */
    protected function pruning(): void
    {
        // ...
    }

整理可能なモデルを設定した後、アプリケーションの`routes/console.php`ファイルに`model:prune` Artisanコマンドをスケジュールする必要があります。このコマンドを実行する適切な間隔を自由に選択できます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('model:prune')->daily();

舞台裏では、`model:prune`コマンドは自動的にアプリケーションの`app/Models`ディレクトリ内の「Prunable」モデルを検出します。モデルが別の場所にある場合は、`--model`オプションを使用してモデルクラス名を指定できます:

    Schedule::command('model:prune', [
        '--model' => [Address::class, Flight::class],
    ])->daily();

整理中に特定のモデルを除外したい場合は、`--except`オプションを使用できます:

    Schedule::command('model:prune', [
        '--except' => [Address::class, Flight::class],
    ])->daily();

`--pretend`オプションを使用して`model:prune`コマンドを実行することで、`prunable`クエリをテストできます。pretendモードでは、`model:prune`コマンドは、コマンドが実際に実行された場合に何件のレコードが整理されるかを報告するだけです:

```shell
php artisan model:prune --pretend
```

> WARNING:  
> ソフトデリートされたモデルは、整理可能なクエリに一致する場合、完全に削除（`forceDelete`）されます。

<a name="mass-pruning"></a>
#### マス整理

`Illuminate\Database\Eloquent\MassPrunable`トレイトでモデルがマークされている場合、モデルはマス削除クエリを使用してデータベースから削除されます。したがって、`pruning`メソッドは呼び出されず、`deleting`および`deleted`モデルイベントも発行されません。これは、モデルが削除前に実際に取得されないため、整理プロセスがより効率的になるためです:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\MassPrunable;

    class Flight extends Model
    {
        use MassPrunable;

        /**
         * 整理可能なモデルクエリを取得します。
         */
        public function prunable(): Builder
        {
            return static::where('created_at', '<=', now()->subMonth());
        }
    }
```

<a name="replicating-models"></a>
## モデルの複製

既存のモデルインスタンスの未保存のコピーを作成するには、`replicate`メソッドを使用できます。このメソッドは、多くの同じ属性を共有するモデルインスタンスがある場合に特に便利です。

```php
use App\Models\Address;

$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

新しいモデルに複製されないようにする属性を1つ以上除外するには、`replicate`メソッドに配列を渡すことができます。

```php
$flight = Flight::create([
    'destination' => 'LAX',
    'origin' => 'LHR',
    'last_flown' => '2020-03-04 11:00:00',
    'last_pilot_id' => 747,
]);

$flight = $flight->replicate([
    'last_flown',
    'last_pilot_id'
]);
```

<a name="query-scopes"></a>
## クエリスコープ

<a name="global-scopes"></a>
### グローバルスコープ

グローバルスコープを使用すると、特定のモデルのすべてのクエリに制約を追加できます。Laravelの[ソフトデリート](#soft-deleting)機能は、グローバルスコープを使用して、データベースから「削除されていない」モデルのみを取得します。独自のグローバルスコープを記述することで、特定のモデルのすべてのクエリが特定の制約を受けるようにする簡単な方法を提供できます。

<a name="generating-scopes"></a>
#### スコープの生成

新しいグローバルスコープを生成するには、`make:scope` Artisanコマンドを呼び出すことができます。これにより、生成されたスコープがアプリケーションの`app/Models/Scopes`ディレクトリに配置されます。

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### グローバルスコープの記述

グローバルスコープの記述は簡単です。まず、`make:scope`コマンドを使用して、`Illuminate\Database\Eloquent\Scope`インターフェースを実装するクラスを生成します。`Scope`インターフェースでは、`apply`メソッドを実装する必要があります。`apply`メソッドは、必要に応じて`where`制約や他の種類の句をクエリに追加できます。

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->subYears(2000));
    }
}
```

> NOTE:  
> グローバルスコープがクエリのselect句に列を追加する場合、`select`の代わりに`addSelect`メソッドを使用する必要があります。これにより、クエリの既存のselect句が意図せずに置き換えられるのを防ぐことができます。

<a name="applying-global-scopes"></a>
#### グローバルスコープの適用

モデルにグローバルスコープを割り当てるには、モデルに`ScopedBy`属性を配置するだけです。

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Attributes\ScopedBy;

#[ScopedBy([AncientScope::class])]
class User extends Model
{
    //
}
```

または、モデルの`booted`メソッドをオーバーライドして、モデルの`addGlobalScope`メソッドを呼び出すことで、グローバルスコープを手動で登録することもできます。`addGlobalScope`メソッドは、スコープのインスタンスを唯一の引数として受け取ります。

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope(new AncientScope);
    }
}
```

上記の例で`App\Models\User`モデルにスコープを追加した後、`User::all()`メソッドを呼び出すと、次のSQLクエリが実行されます。

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### 匿名グローバルスコープ

Eloquentでは、クロージャを使用してグローバルスコープを定義することもできます。これは、独自のクラスを必要としない単純なスコープに特に便利です。クロージャを使用してグローバルスコープを定義する場合、`addGlobalScope`メソッドの最初の引数として、独自に選択したスコープ名を指定する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope('ancient', function (Builder $builder) {
            $builder->where('created_at', '<', now()->subYears(2000));
        });
    }
}
```

<a name="removing-global-scopes"></a>
#### グローバルスコープの削除

特定のクエリのグローバルスコープを削除したい場合は、`withoutGlobalScope`メソッドを使用できます。このメソッドは、グローバルスコープのクラス名を唯一の引数として受け取ります。

```php
User::withoutGlobalScope(AncientScope::class)->get();
```

または、クロージャを使用してグローバルスコープを定義した場合は、グローバルスコープに割り当てた文字列名を渡す必要があります。

```php
User::withoutGlobalScope('ancient')->get();
```

クエリのグローバルスコープのいくつかまたはすべてを削除したい場合は、`withoutGlobalScopes`メソッドを使用できます。

```php
// Remove all of the global scopes...
User::withoutGlobalScopes()->get();

// Remove some of the global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

<a name="local-scopes"></a>
### ローカルスコープ

ローカルスコープを使用すると、アプリケーション全体で簡単に再利用できる共通のクエリ制約のセットを定義できます。たとえば、「人気がある」と見なされるすべてのユーザーを頻繁に取得する必要がある場合があります。スコープを定義するには、Eloquentモデルメソッドに`scope`プレフィックスを付けます。

スコープは常に同じクエリビルダインスタンスを返すか、`void`を返す必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     */
    public function scopePopular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     */
    public function scopeActive(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

<a name="utilizing-a-local-scope"></a>
#### ローカルスコープの利用

スコープが定義されたら、モデルをクエリするときにスコープメソッドを呼び出すことができます。ただし、メソッドを呼び出すときに`scope`プレフィックスを含めるべきではありません。さまざまなスコープへの呼び出しを連鎖させることもできます。

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

複数のEloquentモデルスコープを`or`クエリ演算子で組み合わせるには、正しい[論理グループ化](queries.md#logical-grouping)を実現するためにクロージャを使用する必要があります。

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

ただし、これは面倒な場合があるため、Laravelはクロージャを使用せずにスコープを流暢に連鎖させることができる「高次」の`orWhere`メソッドを提供します。

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### 動的スコープ

パラメータを受け取るスコープを定義したい場合があります。まず、スコープメソッドのシグネチャに追加のパラメータを追加します。スコープパラメータは、`$query`パラメータの後に定義する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     */
    public function scopeOfType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

スコープメソッドのシグネチャに期待される引数を追加したら、スコープを呼び出すときに引数を渡すことができます。

```php
$users = User::ofType('admin')->get();
```

<a name="comparing-models"></a>
## モデルの比較

時には、2つのモデルが「同じ」かどうかを判断する必要があるかもしれません。`is`メソッドと`isNot`メソッドを使用して、2つのモデルが同じ主キー、テーブル、およびデータベース接続を持っているかどうかを迅速に確認できます。

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

`is`メソッドと`isNot`メソッドは、`belongsTo`、`hasOne`、`morphTo`、および`morphOne`[リレーション](eloquent-relationships.md)を使用する場合にも利用できます。このメソッドは、関連モデルを取得するクエリを発行せずに関連モデルを比較する場合に特に便利です。

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## イベント

> NOTE:  
> Eloquentイベントをクライアントサイドアプリケーションに直接ブロードキャストしたいですか？Laravelの[モデルイベントブロードキャスト](broadcasting.md#model-broadcasting)をチェックしてください。

Eloquentモデルは、いくつかのイベントをディスパッチします。これにより、モデルのライフサイクルの次のポイントにフックすることができます：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`trashed`、`forceDeleting`、`forceDeleted`、`restoring`、`restored`、および`replicating`。

`retrieved` イベントは、既存のモデルがデータベースから取得されたときにディスパッチされます。新しいモデルが初めて保存されるときは、`creating` と `created` イベントがディスパッチされます。既存のモデルが変更され、`save` メソッドが呼び出されると、`updating` / `updated` イベントがディスパッチされます。モデルが作成または更新されるときには `saving` / `saved` イベントがディスパッチされます - モデルの属性が変更されていない場合でも同様です。`-ing` で終わるイベント名は、モデルへの変更が永続化される前にディスパッチされ、`-ed` で終わるイベント名は、モデルへの変更が永続化された後にディスパッチされます。

モデルイベントのリスニングを開始するには、Eloquentモデルに `$dispatchesEvents` プロパティを定義します。このプロパティは、Eloquentモデルのライフサイクルの様々なポイントを独自の [イベントクラス](events.md) にマッピングします。各モデルイベントクラスは、コンストラクタを介して影響を受けるモデルのインスタンスを受け取ることが期待されます。

    <?php

    namespace App\Models;

    use App\Events\UserDeleted;
    use App\Events\UserSaved;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * モデルのイベントマップ
         *
         * @var array<string, string>
         */
        protected $dispatchesEvents = [
            'saved' => UserSaved::class,
            'deleted' => UserDeleted::class,
        ];
    }

Eloquentイベントを定義してマッピングした後、[イベントリスナー](events.md#defining-listeners) を使用してイベントを処理できます。

> WARNING:  
> Eloquentを介してマスアップデートまたはマス削除クエリを発行する場合、影響を受けるモデルに対して `saved`、`updated`、`deleting`、`deleted` モデルイベントはディスパッチされません。これは、マスアップデートまたはマス削除を実行する際にモデルが実際に取得されないためです。

<a name="events-using-closures"></a>
### クロージャの使用

カスタムイベントクラスを使用する代わりに、様々なモデルイベントがディスパッチされたときに実行されるクロージャを登録できます。通常、これらのクロージャはモデルの `booted` メソッドに登録する必要があります。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * モデルの "booted" メソッド
         */
        protected static function booted(): void
        {
            static::created(function (User $user) {
                // ...
            });
        }
    }

必要に応じて、モデルイベントを登録する際に [キュー可能な匿名イベントリスナー](events.md#queuable-anonymous-event-listeners) を利用できます。これにより、Laravelはアプリケーションの [キュー](queues.md) を使用してバックグラウンドでモデルイベントリスナーを実行するよう指示されます。

    use function Illuminate\Events\queueable;

    static::created(queueable(function (User $user) {
        // ...
    }));

<a name="observers"></a>
### オブザーバー

<a name="defining-observers"></a>
#### オブザーバーの定義

特定のモデルで多くのイベントをリスニングしている場合、オブザーバーを使用してすべてのリスナーを1つのクラスにグループ化できます。オブザーバークラスのメソッド名は、リスニングしたいEloquentイベントを反映します。これらの各メソッドは、影響を受けるモデルを唯一の引数として受け取ります。`make:observer` Artisanコマンドは、新しいオブザーバークラスを作成する最も簡単な方法です。

```shell
php artisan make:observer UserObserver --model=User
```

このコマンドは、新しいオブザーバーを `app/Observers` ディレクトリに配置します。このディレクトリが存在しない場合、Artisanが自動的に作成します。新しいオブザーバーは次のようになります。

    <?php

    namespace App\Observers;

    use App\Models\User;

    class UserObserver
    {
        /**
         * User "created" イベントの処理
         */
        public function created(User $user): void
        {
            // ...
        }

        /**
         * User "updated" イベントの処理
         */
        public function updated(User $user): void
        {
            // ...
        }

        /**
         * User "deleted" イベントの処理
         */
        public function deleted(User $user): void
        {
            // ...
        }

        /**
         * User "restored" イベントの処理
         */
        public function restored(User $user): void
        {
            // ...
        }

        /**
         * User "forceDeleted" イベントの処理
         */
        public function forceDeleted(User $user): void
        {
            // ...
        }
    }

オブザーバーを登録するには、対応するモデルに `ObservedBy` 属性を配置することができます。

    use App\Observers\UserObserver;
    use Illuminate\Database\Eloquent\Attributes\ObservedBy;

    #[ObservedBy([UserObserver::class])]
    class User extends Authenticatable
    {
        //
    }

または、観察したいモデルで `observe` メソッドを呼び出して手動でオブザーバーを登録することもできます。アプリケーションの `AppServiceProvider` クラスの `boot` メソッドでオブザーバーを登録することができます。

    use App\Models\User;
    use App\Observers\UserObserver;

    /**
     * アプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        User::observe(UserObserver::class);
    }

> NOTE:  
> オブザーバーは `saving` や `retrieved` などの追加のイベントをリスニングできます。これらのイベントは [イベント](#events) ドキュメント内で説明されています。

<a name="observers-and-database-transactions"></a>
#### オブザーバーとデータベーストランザクション

モデルがデータベーストランザクション内で作成されている場合、データベーストランザクションがコミットされた後にのみオブザーバーのイベントハンドラを実行するよう指示したい場合があります。これを実現するには、オブザーバーに `ShouldHandleEventsAfterCommit` インターフェースを実装します。データベーストランザクションが進行中でない場合、イベントハンドラはすぐに実行されます。

    <?php

    namespace App\Observers;

    use App\Models\User;
    use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

    class UserObserver implements ShouldHandleEventsAfterCommit
    {
        /**
         * User "created" イベントの処理
         */
        public function created(User $user): void
        {
            // ...
        }
    }

<a name="muting-events"></a>
### イベントのミュート

モデルによって発生するすべてのイベントを一時的に「ミュート」する必要がある場合があります。これは `withoutEvents` メソッドを使用して実現できます。`withoutEvents` メソッドは、唯一の引数としてクロージャを受け取ります。このクロージャ内で実行されるコードはモデルイベントをディスパッチせず、クロージャによって返される値は `withoutEvents` メソッドによって返されます。

    use App\Models\User;

    $user = User::withoutEvents(function () {
        User::findOrFail(1)->delete();

        return User::find(2);
    });

<a name="saving-a-single-model-without-events"></a>
#### イベントなしで単一のモデルを保存

イベントをディスパッチせずに特定のモデルを「保存」したい場合があります。これは `saveQuietly` メソッドを使用して実現できます。

    $user = User::findOrFail(1);

    $user->name = 'Victoria Faith';

    $user->saveQuietly();

また、イベントをディスパッチせずに特定のモデルを「更新」、「削除」、「ソフト削除」、「復元」、「複製」することもできます。

    $user->deleteQuietly();
    $user->forceDeleteQuietly();
    $user->restoreQuietly();

