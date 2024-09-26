# Eloquent: ミューテータとキャスト

- [はじめに](#introduction)
- [アクセサとミューテータ](#accessors-and-mutators)
    - [アクセサの定義](#defining-an-accessor)
    - [ミューテータの定義](#defining-a-mutator)
- [属性キャスト](#attribute-casting)
    - [配列とJSONのキャスト](#array-and-json-casting)
    - [日付のキャスト](#date-casting)
    - [Enumのキャスト](#enum-casting)
    - [暗号化のキャスト](#encrypted-casting)
    - [クエリ時のキャスト](#query-time-casting)
- [カスタムキャスト](#custom-casts)
    - [値オブジェクトのキャスト](#value-object-casting)
    - [配列 / JSON シリアライズ](#array-json-serialization)
    - [インバウンドキャスト](#inbound-casting)
    - [キャストパラメータ](#cast-parameters)
    - [キャスト可能](#castables)

<a name="introduction"></a>
## はじめに

アクセサ、ミューテータ、および属性キャストを使用すると、Eloquentモデルインスタンスで属性値を取得または設定する際に、それらの値を変換できます。例えば、データベースに保存される際に[Laravelの暗号化機能](encryption.md)を使用して値を暗号化し、Eloquentモデルでアクセスする際に自動的に復号化することができます。または、データベースに保存されているJSON文字列を、Eloquentモデル経由でアクセスする際に配列に変換することもできます。

<a name="accessors-and-mutators"></a>
## アクセサとミューテータ

<a name="defining-an-accessor"></a>
### アクセサの定義

アクセサは、Eloquent属性値がアクセスされる際にその値を変換します。アクセサを定義するには、モデル上に保護されたメソッドを作成し、アクセスする属性を表現します。このメソッド名は、該当する場合、実際の基礎となるモデル属性/データベースカラムの「キャメルケース」表現に対応する必要があります。

この例では、`first_name`属性のアクセサを定義します。アクセサは、`first_name`属性の値を取得しようとする際にEloquentによって自動的に呼び出されます。すべての属性アクセサ/ミューテータメソッドは、`Illuminate\Database\Eloquent\Casts\Attribute`の戻り値の型ヒントを宣言する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * ユーザーのファーストネームを取得します。
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```

すべてのアクセサメソッドは、属性のアクセス方法と、オプションで変更方法を定義する`Attribute`インスタンスを返します。この例では、属性のアクセス方法のみを定義しています。そのために、`Attribute`クラスのコンストラクタに`get`引数を渡します。

ご覧のように、カラムの元の値がアクセサに渡され、値を操作して返すことができます。アクセサの値にアクセスするには、モデルインスタンスの`first_name`属性に単純にアクセスします。

```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```

> NOTE:  
> これらの計算された値をモデルの配列/JSON表現に追加したい場合は、[それらを追加する必要があります](eloquent-serialization.md#appending-values-to-json)。

<a name="building-value-objects-from-multiple-attributes"></a>
#### 複数の属性から値オブジェクトを構築する

時には、アクセサが複数のモデル属性を1つの「値オブジェクト」に変換する必要があるかもしれません。そのためには、`get`クロージャが2番目の引数として`$attributes`を受け取ることができます。これは、クロージャに自動的に提供され、モデルの現在のすべての属性の配列を含みます。

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * ユーザーの住所とやり取りします。
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### アクセサのキャッシュ

アクセサから値オブジェクトを返す場合、値オブジェクトに加えられた変更は、モデルが保存される前に自動的にモデルに同期されます。これは、Eloquentがアクセサによって返されたインスタンスを保持し、アクセサが呼び出されるたびに同じインスタンスを返すことができるためです。

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Updated Address Line 1 Value';
$user->address->lineTwo = 'Updated Address Line 2 Value';

$user->save();
```

ただし、特に計算量が多い場合、文字列やブール値などのプリミティブ値のキャッシュを有効にしたい場合があります。これを実現するには、アクセサを定義する際に`shouldCache`メソッドを呼び出すことができます。

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

属性のオブジェクトキャッシュ動作を無効にしたい場合は、属性を定義する際に`withoutObjectCaching`メソッドを呼び出すことができます。

```php
/**
 * ユーザーの住所とやり取りします。
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="defining-a-mutator"></a>
### ミューテータの定義

ミューテータは、Eloquent属性の値が設定される際にその値を変換します。ミューテータを定義するには、属性を定義する際に`set`引数を提供することができます。`first_name`属性のミューテータを定義しましょう。このミューテータは、モデルの`first_name`属性の値を設定しようとする際に自動的に呼び出されます。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * ユーザーのファーストネームとやり取りします。
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```

ミューテータクロージャは、属性に設定される値を受け取り、値を操作して返すことができます。ミューテータを使用するには、Eloquentモデルの`first_name`属性を設定するだけです。

```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```

この例では、`set`コールバックは値`Sally`で呼び出されます。ミューテータは名前に`strtolower`関数を適用し、その結果の値をモデルの内部`$attributes`配列に設定します。

<a name="mutating-multiple-attributes"></a>
#### 複数の属性を変更する

時には、ミューテータが基礎となるモデルに複数の属性を設定する必要があるかもしれません。そのためには、`set`クロージャから配列を返すことができます。配列の各キーは、モデルに関連付けられた基礎となる属性/データベースカラムに対応する必要があります。

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * ユーザーの住所とやり取りします。
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

<a name="attribute-casting"></a>
## 属性キャスト

属性キャストは、アクセサやミューテータと同様の機能を提供しますが、モデルに追加のメソッドを定義する必要がありません。代わりに、モデルの`casts`メソッドが、属性を一般的なデータ型に変換する便利な方法を提供します。

`casts`メソッドは、キーがキャストする属性の名前で、値がカラムをキャストしたい型である配列を返す必要があります。サポートされているキャスト型は以下の通りです。

<div class="content-list" markdown="1">

- `array`
- `AsStringable::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- <code>decimal:&lt;precision&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `hashed`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

</div>

属性キャストを実演するために、データベースに整数(`0`または`1`)として保存されている`is_admin`属性をブール値にキャストしましょう。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```

キャストを定義すると、`is_admin`属性は常にブール値にキャストされます。基礎となる値がデータベースに整数として保存されていてもです。

```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```

実行時に新しい一時的なキャストを追加する必要がある場合は、`mergeCasts`メソッドを使用することができます。これらのキャスト定義は、モデルに既に定義されているキャストに追加されます。

```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'object',
]);
```

> WARNING:  
> `null`の属性はキャストされません。また、リレーションと同じ名前のキャスト（または属性）を定義したり、モデルの主キーにキャストを割り当てたりしないでください。

<a name="stringable-casting"></a>
#### Stringable キャスト

モデル属性を[流れるような `Illuminate\Support\Stringable` オブジェクト](strings.md#fluent-strings-method-list)にキャストするために、`Illuminate\Database\Eloquent\Casts\AsStringable` キャストクラスを使用できます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\AsStringable;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * キャストする必要がある属性を取得します。
         *
         * @return array<string, string>
         */
        protected function casts(): array
        {
            return [
                'directory' => AsStringable::class,
            ];
        }
    }

<a name="array-and-json-casting"></a>
### 配列とJSONのキャスト

`array` キャストは、シリアライズされたJSONとして保存される列を扱う際に特に便利です。例えば、データベースにシリアライズされたJSONを含む `JSON` または `TEXT` フィールドタイプがある場合、その属性に `array` キャストを追加すると、Eloquentモデルでアクセスしたときに属性が自動的にPHP配列にデシリアライズされます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * キャストする必要がある属性を取得します。
         *
         * @return array<string, string>
         */
        protected function casts(): array
        {
            return [
                'options' => 'array',
            ];
        }
    }

キャストが定義されると、`options` 属性にアクセスすると、自動的にJSONからPHP配列にデシリアライズされます。`options` 属性の値を設定すると、指定された配列は自動的にJSONにシリアライズされて保存されます。

    use App\Models\User;

    $user = User::find(1);

    $options = $user->options;

    $options['key'] = 'value';

    $user->options = $options;

    $user->save();

より簡潔な構文でJSON属性の単一フィールドを更新するには、[属性を一括割り当て可能に](eloquent.md#mass-assignment-json-columns)し、`update` メソッドを呼び出す際に `->` 演算子を使用できます。

    $user = User::find(1);

    $user->update(['options->key' => 'value']);

<a name="array-object-and-collection-casting"></a>
#### 配列オブジェクトとコレクションのキャスト

標準の `array` キャストは多くのアプリケーションで十分ですが、いくつかの欠点があります。`array` キャストはプリミティブ型を返すため、配列のオフセットを直接変更することはできません。例えば、次のコードはPHPエラーをトリガーします。

    $user = User::find(1);

    $user->options['key'] = $value;

これを解決するために、LaravelはJSON属性を [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php) クラスにキャストする `AsArrayObject` キャストを提供します。この機能はLaravelの[カスタムキャスト](#custom-casts)実装を使用して実現され、Laravelは変更されたオブジェクトをインテリジェントにキャッシュおよび変換し、個々のオフセットを変更してもPHPエラーが発生しないようにします。`AsArrayObject` キャストを使用するには、単に属性に割り当てます。

    use Illuminate\Database\Eloquent\Casts\AsArrayObject;

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsArrayObject::class,
        ];
    }

同様に、LaravelはJSON属性をLaravel [Collection](collections.md) インスタンスにキャストする `AsCollection` キャストを提供します。

    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsCollection::class,
        ];
    }

Laravelの基本コレクションクラスの代わりにカスタムコレクションクラスをインスタンス化する場合は、キャスト引数としてコレクションクラス名を指定できます。

    use App\Collections\OptionCollection;
    use Illuminate\Database\Eloquent\Casts\AsCollection;

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsCollection::using(OptionCollection::class),
        ];
    }

<a name="date-casting"></a>
### 日付のキャスト

デフォルトでは、Eloquentは `created_at` および `updated_at` 列を [Carbon](https://github.com/briannesbitt/Carbon) インスタンスにキャストします。これはPHPの `DateTime` クラスを拡張し、さまざまな便利なメソッドを提供します。モデルの `casts` メソッド内で追加の日付キャストを定義することで、追加の日付属性をキャストできます。通常、日付は `datetime` または `immutable_datetime` キャストタイプを使用してキャストする必要があります。

`date` または `datetime` キャストを定義する際に、日付のフォーマットを指定することもできます。このフォーマットは、[モデルが配列またはJSONにシリアライズされる](eloquent-serialization.md)際に使用されます。

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'created_at' => 'datetime:Y-m-d',
        ];
    }

列が日付としてキャストされる場合、対応するモデル属性の値をUNIXタイムスタンプ、日付文字列（`Y-m-d`）、日時文字列、または `DateTime` / `Carbon` インスタンスに設定できます。日付の値は正しく変換され、データベースに保存されます。

モデルのすべての日付のデフォルトのシリアライズフォーマットをカスタマイズするには、モデルに `serializeDate` メソッドを定義します。このメソッドは、データベースに日付が格納される方法には影響しません。

    /**
     * 配列 / JSON シリアライズ用に日付を準備します。
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d');
    }

データベース内にモデルの日付を実際に格納する際に使用するフォーマットを指定するには、モデルに `$dateFormat` プロパティを定義する必要があります。

    /**
     * モデルの日付列のストレージフォーマット。
     *
     * @var string
     */
    protected $dateFormat = 'U';

<a name="date-casting-and-timezones"></a>
#### 日付のキャスト、シリアライズ、およびタイムゾーン

デフォルトでは、`date` および `datetime` キャストは、アプリケーションの `timezone` 設定オプションで指定されたタイムゾーンに関係なく、UTC ISO-8601 日付文字列（`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`）に日付をシリアライズします。このシリアライズフォーマットを常に使用し、アプリケーションのタイムゾーンをデフォルトの `UTC` 値から変更せずにUTCタイムゾーンに日付を保存することを強くお勧めします。アプリケーション全体でUTCタイムゾーンを一貫して使用することで、PHPおよびJavaScriptで記述された他の日付操作ライブラリとの最大限の相互運用性が得られます。

`date` または `datetime` キャストにカスタムフォーマットが適用されている場合（例：`datetime:Y-m-d H:i:s`）、日付のシリアライズ中にCarbonインスタンスの内部タイムゾーンが使用されます。通常、これはアプリケーションの `timezone` 設定オプションで指定されたタイムゾーンになります。ただし、`created_at` や `updated_at` などの `timestamp` 列はこの動作から除外され、アプリケーションのタイムゾーン設定に関係なく常にUTCでフォーマットされることに注意してください。

<a name="enum-casting"></a>
### Enumのキャスト

Eloquentでは、属性値をPHPの [Enums](https://www.php.net/manual/en/language.enumerations.backed.php) にキャストすることもできます。これを行うには、モデルの `casts` メソッドでキャストしたい属性とEnumを指定します。

    use App\Enums\ServerStatus;

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'status' => ServerStatus::class,
        ];
    }

モデルにキャストを定義すると、指定された属性は自動的にEnumとの間でキャストされます。

    if ($server->status == ServerStatus::Provisioned) {
        $server->status = ServerStatus::Ready;

        $server->save();
    }

<a name="casting-arrays-of-enums"></a>
#### Enumの配列のキャスト

モデルが単一の列にEnum値の配列を保存する必要がある場合があります。これを行うには、Laravelが提供する `AsEnumArrayObject` または `AsEnumCollection` キャストを利用できます。

    use App\Enums\ServerStatus;
    use Illuminate\Database\Eloquent\Casts\AsEnumCollection;

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'statuses' => AsEnumCollection::of(ServerStatus::class),
        ];
    }

<a name="encrypted-casting"></a>
### 暗号化されたキャスト

`encrypted` キャストは、Laravelの組み込みの[暗号化](encryption.md)機能を使用してモデルの属性値を暗号化します。さらに、`encrypted:array`、`encrypted:collection`、`encrypted:object`、`AsEncryptedArrayObject`、および `AsEncryptedCollection` キャストは、対応する非暗号化のキャストと同様に動作します。ただし、予想通り、基になる値はデータベースに保存される際に暗号化されます。

暗号化されたテキストの最終的な長さは予測不可能であり、プレーンテキストの対応物よりも長くなるため、関連するデータベース列が `TEXT` 型以上であることを確認してください。また、値がデータベースで暗号化されているため、暗号化された属性値をクエリまたは検索することはできません。

<a name="key-rotation"></a>
#### キーのローテーション

Laravelは、アプリケーションの`app`設定ファイルで指定された`key`設定値を使用して文字列を暗号化します。通常、この値は`APP_KEY`環境変数の値に対応します。アプリケーションの暗号化キーをローテーションする必要がある場合は、新しいキーを使用して暗号化された属性を手動で再暗号化する必要があります。

<a name="query-time-casting"></a>
### クエリ時のキャスト

テーブルから生の値を選択する場合など、クエリの実行中にキャストを適用する必要がある場合があります。例えば、次のクエリを考えてみましょう。

    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();

このクエリの結果の`last_posted_at`属性は単純な文字列になります。クエリの実行時にこの属性に`datetime`キャストを適用できれば素晴らしいでしょう。幸いなことに、`withCasts`メソッドを使用してこれを実現できます。

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->withCasts([
        'last_posted_at' => 'datetime'
    ])->get();

<a name="custom-casts"></a>
## カスタムキャスト

Laravelには、さまざまな組み込みの便利なキャストタイプがありますが、独自のキャストタイプを定義する必要がある場合もあります。キャストを作成するには、`make:cast` Artisanコマンドを実行します。新しいキャストクラスは`app/Casts`ディレクトリに配置されます。

```shell
php artisan make:cast Json
```

すべてのすべてのカスタムキャストクラスは`CastsAttributes`インターフェースを実装する必要があります。このインターフェースを実装するクラスは、`get`メソッドと`set`メソッドを定義する必要があります。`get`メソッドはデータベースからの生の値をキャストされた値に変換する役割を持ち、`set`メソッドはキャストされた値をデータベースに保存できる生の値に変換する必要があります。例として、組み込みの`json`キャストタイプをカスタムキャストタイプとして再実装します。

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use Illuminate\Database\Eloquent\Model;

    class Json implements CastsAttributes
    {
        /**
         * 指定された値をキャストします。
         *
         * @param  array<string, mixed>  $attributes
         * @return array<string, mixed>
         */
        public function get(Model $model, string $key, mixed $value, array $attributes): array
        {
            return json_decode($value, true);
        }

        /**
         * 指定された値を保存用に準備します。
         *
         * @param  array<string, mixed>  $attributes
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): string
        {
            return json_encode($value);
        }
    }

カスタムキャストタイプを定義したら、クラス名を使用してモデル属性にアタッチできます。

    <?php

    namespace App\Models;

    use App\Casts\Json;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * キャストする必要がある属性を取得します。
         *
         * @return array<string, string>
         */
        protected function casts(): array
        {
            return [
                'options' => Json::class,
            ];
        }
    }

<a name="value-object-casting"></a>
### 値オブジェクトのキャスト

値をプリミティブ型にキャストするだけでなく、値をオブジェクトにキャストすることもできます。カスタムキャストを使用して値をオブジェクトにキャストすることは、プリミティブ型にキャストすることと非常によく似ていますが、`set`メソッドはキー/値のペアの配列を返す必要があります。これは、モデルに設定される生の保存可能な値に使用されます。

例として、複数のモデル値を1つの`Address`値オブジェクトにキャストするカスタムキャストクラスを定義します。`Address`値オブジェクトには`lineOne`と`lineTwo`という2つのパブリックプロパティがあると仮定します。

    <?php

    namespace App\Casts;

    use App\ValueObjects\Address as AddressValueObject;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
    use Illuminate\Database\Eloquent\Model;
    use InvalidArgumentException;

    class Address implements CastsAttributes
    {
        /**
         * 指定された値をキャストします。
         *
         * @param  array<string, mixed>  $attributes
         */
        public function get(Model $model, string $key, mixed $value, array $attributes): AddressValueObject
        {
            return new AddressValueObject(
                $attributes['address_line_one'],
                $attributes['address_line_two']
            );
        }

        /**
         * 指定された値を保存用に準備します。
         *
         * @param  array<string, mixed>  $attributes
         * @return array<string, string>
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): array
        {
            if (! $value instanceof AddressValueObject) {
                throw new InvalidArgumentException('The given value is not an Address instance.');
            }

            return [
                'address_line_one' => $value->lineOne,
                'address_line_two' => $value->lineTwo,
            ];
        }
    }

値オブジェクトにキャストする場合、値オブジェクトに加えられた変更は、モデルが保存される前に自動的にモデルに同期されます。

    use App\Models\User;

    $user = User::find(1);

    $user->address->lineOne = 'Updated Address Value';

    $user->save();

> NOTE:  
> EloquentモデルをJSONまたは配列にシリアライズする予定がある場合、値オブジェクトに`Illuminate\Contracts\Support\Arrayable`と`JsonSerializable`インターフェースを実装する必要があります。

<a name="value-object-caching"></a>
#### 値オブジェクトのキャッシュ

値オブジェクトにキャストされた属性が解決されると、Eloquentによってキャッシュされます。したがって、属性が再度アクセスされると、同じオブジェクトインスタンスが返されます。

カスタムキャストクラスのオブジェクトキャッシュ動作を無効にしたい場合は、カスタムキャストクラスにパブリックな`withoutObjectCaching`プロパティを宣言できます。

```php
class Address implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="array-json-serialization"></a>
### 配列 / JSON シリアライズ

Eloquentモデルが`toArray`および`toJson`メソッドを使用して配列またはJSONに変換される場合、カスタムキャスト値オブジェクトは通常、`Illuminate\Contracts\Support\Arrayable`と`JsonSerializable`インターフェースを実装している限りシリアライズされます。ただし、サードパーティライブラリが提供する値オブジェクトを使用している場合、これらのインターフェースをオブジェクトに追加する権限がない場合があります。

したがって、カスタムキャストクラスが値オブジェクトのシリアライズを担当するように指定できます。そのためには、カスタムキャストクラスが`Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes`インターフェースを実装する必要があります。このインターフェースは、クラスに`serialize`メソッドを含むことを要求し、このメソッドは値オブジェクトのシリアライズされた形式を返す必要があります。

    /**
     * 値のシリアライズされた表現を取得します。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function serialize(Model $model, string $key, mixed $value, array $attributes): string
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### インバウンドキャスト

モデルに設定されている値のみを変換し、属性がモデルから取得されるときに操作を実行しないカスタムキャストクラスを作成する必要がある場合があります。

インバウンド専用のカスタムキャストは、`CastsInboundAttributes`インターフェースを実装する必要があります。これは、`set`メソッドのみを定義する必要があります。`make:cast` Artisanコマンドは、`--inbound`オプションを使用してインバウンドのみのキャストクラスを生成できます。

```shell
php artisan make:cast Hash --inbound
```

インバウンドのみのキャストの古典的な例は、「ハッシュ」キャストです。例えば、指定されたアルゴリズムを介してインバウンド値をハッシュするキャストを定義できます。

    <?php

    namespace App\Casts;

    use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
    use Illuminate\Database\Eloquent\Model;

    class Hash implements CastsInboundAttributes
    {
        /**
         * 新しいキャストクラスインスタンスを作成します。
         */
        public function __construct(
            protected string|null $algorithm = null,
        ) {}

        /**
         * 指定された値を保存用に準備します。
         *
         * @param  array<string, mixed>  $attributes
         */
        public function set(Model $model, string $key, mixed $value, array $attributes): string
        {
            return is_null($this->algorithm)
                        ? bcrypt($value)
                        : hash($this->algorithm, $value);
        }
    }

<a name="cast-parameters"></a>
### キャストパラメータ

モデルにカスタムキャストをアタッチする際、クラス名を`:`文字で区切り、複数のパラメータをカンマで区切ることでキャストパラメータを指定できます。パラメータはキャストクラスのコンストラクタに渡されます。

    /**
     * キャストする必要がある属性を取得します。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'secret' => Hash::class.':sha256',
        ];
    }

<a name="castables"></a>
### Castables

アプリケーションの値オブジェクトが独自のカスタムキャストクラスを定義できるようにしたい場合があります。カスタムキャストクラスをモデルにアタッチする代わりに、`Illuminate\Contracts\Database\Eloquent\Castable`インターフェースを実装する値オブジェクトクラスをアタッチすることもできます。

    use App\ValueObjects\Address;

    protected function casts(): array
    {
        return [
            'address' => Address::class,
        ];
    }

`Castable` インターフェースを実装するオブジェクトは、`Castable` クラスとの間でキャストを行うためのカスタムキャスタークラスのクラス名を返す `castUsing` メソッドを定義する必要があります。

    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * このキャスト対象との間でキャストする際に使用するキャスタークラスの名前を取得します。
         *
         * @param  array<string, mixed>  $arguments
         */
        public static function castUsing(array $arguments): string
        {
            return AddressCast::class;
        }
    }

`Castable` クラスを使用する場合でも、`casts` メソッドの定義で引数を提供することができます。引数は `castUsing` メソッドに渡されます。

    use App\ValueObjects\Address;

    protected function casts(): array
    {
        return [
            'address' => Address::class.':argument',
        ];
    }

<a name="anonymous-cast-classes"></a>
#### Castables と匿名キャストクラス

"castables" を PHP の [匿名クラス](https://www.php.net/manual/en/language.oop5.anonymous.php) と組み合わせることで、値オブジェクトとそのキャストロジックを単一の castable オブジェクトとして定義できます。これを実現するには、値オブジェクトの `castUsing` メソッドから匿名クラスを返します。匿名クラスは `CastsAttributes` インターフェースを実装する必要があります。

    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * このキャスト対象との間でキャストする際に使用するキャスタークラスを取得します。
         *
         * @param  array<string, mixed>  $arguments
         */
        public static function castUsing(array $arguments): CastsAttributes
        {
            return new class implements CastsAttributes
            {
                public function get(Model $model, string $key, mixed $value, array $attributes): Address
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }

                public function set(Model $model, string $key, mixed $value, array $attributes): array
                {
                    return [
                        'address_line_one' => $value->lineOne,
                        'address_line_two' => $value->lineTwo,
                    ];
                }
            };
        }
    }
