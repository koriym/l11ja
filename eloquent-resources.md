# Eloquent: API Resources

- [はじめに](#introduction)
- [リソースの生成](#generating-resources)
- [概念の概要](#concept-overview)
    - [リソースコレクション](#resource-collections)
- [リソースの記述](#writing-resources)
    - [データのラッピング](#data-wrapping)
    - [ページネーション](#pagination)
    - [条件付き属性](#conditional-attributes)
    - [条件付きリレーションシップ](#conditional-relationships)
    - [メタデータの追加](#adding-meta-data)
- [リソースのレスポンス](#resource-responses)

<a name="introduction"></a>
## はじめに

APIを構築する際、Eloquentモデルと実際にアプリケーションのユーザーに返されるJSONレスポンスの間に変換レイヤーが必要になる場合があります。例えば、特定の属性を一部のユーザーに表示し、他のユーザーには表示しないようにしたい場合や、モデルのJSON表現に常に特定のリレーションシップを含めたい場合があります。Eloquentのリソースクラスを使用すると、モデルとモデルコレクションをJSONに簡単かつ表現力豊かに変換できます。

もちろん、Eloquentモデルやコレクションを`toJson`メソッドを使用してJSONに変換することは常に可能です。しかし、EloquentリソースはモデルとそのリレーションシップのJSONシリアライズをより細かく、堅牢に制御できます。

<a name="generating-resources"></a>
## リソースの生成

リソースクラスを生成するには、`make:resource` Artisanコマンドを使用できます。デフォルトでは、リソースはアプリケーションの`app/Http/Resources`ディレクトリに配置されます。リソースは`Illuminate\Http\Resources\Json\JsonResource`クラスを拡張します。

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### リソースコレクション

個々のモデルを変換するリソースを生成するだけでなく、モデルのコレクションを変換する責任を持つリソースを生成することもできます。これにより、JSONレスポンスにリンクやその他のメタ情報を含めることができます。

リソースコレクションを作成するには、リソースを作成する際に`--collection`フラグを使用するか、リソース名に`Collection`という単語を含めることで、Laravelにコレクションリソースを作成するよう指示できます。コレクションリソースは`Illuminate\Http\Resources\Json\ResourceCollection`クラスを拡張します。

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## 概念の概要

> NOTE:  
> これはリソースとリソースコレクションの概要です。リソースが提供するカスタマイズとパワーを深く理解するために、このドキュメントの他のセクションを読むことを強くお勧めします。

リソースがLaravel内でどのように使用されるかを詳細に説明する前に、まずリソースの使い方を高レベルで見てみましょう。リソースクラスは、単一のモデルをJSON構造に変換する必要があることを表します。例えば、以下はシンプルな`UserResource`リソースクラスです。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * リソースを配列に変換します。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

すべてのリソースクラスは、リソースがルートまたはコントローラーメソッドからレスポンスとして返されるときにJSONに変換されるべき属性の配列を返す`toArray`メソッドを定義します。

`$this`変数からモデルのプロパティに直接アクセスできることに注意してください。これは、リソースクラスが基礎となるモデルへのプロパティとメソッドのアクセスを自動的にプロキシするためです。リソースが定義されると、ルートまたはコントローラーから返すことができます。リソースは、コンストラクターを介して基礎となるモデルインスタンスを受け取ります。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

<a name="resource-collections"></a>
### リソースコレクション

リソースのコレクションまたはページネーションされたレスポンスを返す場合、ルートまたはコントローラーでリソースインスタンスを作成する際に、リソースクラスが提供する`collection`メソッドを使用する必要があります。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

これにより、コレクションに必要なカスタムメタデータの追加ができなくなります。リソースコレクションのレスポンスをカスタマイズしたい場合は、コレクションを表す専用のリソースを作成できます。

```shell
php artisan make:resource UserCollection
```

リソースコレクションクラスが生成されたら、レスポンスに含めるべきメタデータを簡単に定義できます。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * リソースコレクションを配列に変換します。
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

リソースコレクションを定義した後、ルートまたはコントローラーから返すことができます。

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

<a name="preserving-collection-keys"></a>
#### コレクションキーの保持

ルートからリソースコレクションを返すと、Laravelはコレクションのキーをリセットして数値順に並べます。ただし、コレクションの元のキーを保持するかどうかを示す`preserveKeys`プロパティをリソースクラスに追加できます。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * リソースのコレクションキーを保持するかどうかを示します。
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```

`preserveKeys`プロパティが`true`に設定されている場合、コレクションのキーはルートまたはコントローラーから返されるときに保持されます。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

<a name="customizing-the-underlying-resource-class"></a>
#### 基礎となるリソースクラスのカスタマイズ

通常、リソースコレクションの`$this->collection`プロパティは、コレクションの各アイテムを単一のリソースクラスにマッピングした結果で自動的に設定されます。単一のリソースクラスは、コレクションのクラス名から末尾の`Collection`部分を除いたものと推定されます。さらに、個人的な好みに応じて、単一のリソースクラスに`Resource`という接尾辞が付く場合と付かない場合があります。

例えば、`UserCollection`は指定されたユーザーインスタンスを`UserResource`リソースにマッピングしようとします。この動作をカスタマイズするには、リソースコレクションの`$collects`プロパティをオーバーライドできます。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * このリソースが収集するリソース。
     *
     * @var string
     */
    public $collects = Member::class;
}
```

<a name="writing-resources"></a>
## リソースの記述

> NOTE:  
> [概念の概要](#concept-overview)をまだ読んでいない場合は、このドキュメントを進める前に読むことを強くお勧めします。

リソースは、指定されたモデルを配列に変換する必要があります。したがって、各リソースには、モデルの属性をAPIに適した配列に変換する`toArray`メソッドが含まれています。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * リソースを配列に変換します。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

リソースが定義されると、ルートまたはコントローラーから直接返すことができます。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

<a name="relationships"></a>
#### リレーションシップ

レスポンスに関連リソースを含めたい場合は、リソースの`toArray`メソッドが返す配列にそれらを追加できます。この例では、`PostResource`リソースの`collection`メソッドを使用して、ユーザーのブログ投稿をリソースレスポンスに追加します。

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * リソースを配列に変換する。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

> NOTE:  
> リレーションが既にロードされている場合にのみリレーションを含めたい場合は、[条件付きリレーション](#conditional-relationships)に関するドキュメントを確認してください。

<a name="writing-resource-collections"></a>
#### リソースコレクション

リソースは単一のモデルを配列に変換しますが、リソースコレクションはモデルのコレクションを配列に変換します。しかし、すべてのモデルに対してリソースコレクションクラスを定義する必要はありません。すべてのリソースは、「アドホック」なリソースコレクションを即座に生成するための `collection` メソッドを提供しています。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

ただし、コレクションとともに返されるメタデータをカスタマイズする必要がある場合は、独自のリソースコレクションを定義する必要があります。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * リソースコレクションを配列に変換する。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

単一のリソースと同様に、リソースコレクションはルートまたはコントローラーから直接返すことができます。

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

<a name="data-wrapping"></a>
### データのラッピング

デフォルトでは、リソースレスポンスがJSONに変換されるとき、最も外側のリソースは `data` キーでラップされます。したがって、典型的なリソースコレクションレスポンスは次のようになります。

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

最も外側のリソースのラッピングを無効にしたい場合は、基本の `Illuminate\Http\Resources\Json\JsonResource` クラスで `withoutWrapping` メソッドを呼び出す必要があります。通常、このメソッドは `AppServiceProvider` またはアプリケーションへのすべてのリクエストでロードされる他の[サービスプロバイダ](providers.md)から呼び出す必要があります。

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 任意のアプリケーションサービスを登録する。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 任意のアプリケーションサービスを起動する。
     */
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}
```

> WARNING:  
> `withoutWrapping` メソッドは最も外側のレスポンスにのみ影響し、自分でリソースコレクションに手動で追加した `data` キーは削除しません。

<a name="wrapping-nested-resources"></a>
#### ネストされたリソースのラッピング

リソースのリレーションがどのようにラップされるかを自由に決定できます。すべてのリソースコレクションを `data` キーでラップしたい場合、ネストに関係なく、各リソースに対してリソースコレクションクラスを定義し、コレクションを `data` キー内で返す必要があります。

最も外側のリソースが2つの `data` キーでラップされるかどうかを心配する必要はありません。Laravelはリソースが誤って二重にラップされることを防ぐため、変換しているリソースコレクションのネストレベルについて心配する必要はありません。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * リソースコレクションを配列に変換する。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }
}
```

<a name="data-wrapping-and-pagination"></a>
#### データのラッピングとページネーション

リソースレスポンスを介してページネーションされたコレクションを返す場合、Laravelは `withoutWrapping` メソッドが呼び出された場合でも、リソースデータを `data` キーでラップします。これは、ページネータの状態に関する情報を含む `meta` および `links` キーをページネーションレスポンスに常に含めるためです。

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### ページネーション

Laravelのページネータインスタンスをリソースの `collection` メソッドまたはカスタムリソースコレクションに渡すことができます。

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

ページネーションされたレスポンスには、ページネータの状態に関する情報を含む `meta` および `links` キーが常に含まれます。

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="customizing-the-pagination-information"></a>
#### ページネーション情報のカスタマイズ

ページネーションレスポンスの `links` または `meta` キーに含まれる情報をカスタマイズしたい場合は、リソースに `paginationInformation` メソッドを定義できます。このメソッドは `$paginated` データと `$default` 情報の配列を受け取ります。これは `links` および `meta` キーを含む配列です。

```php
/**
 * リソースのページネーション情報をカスタマイズする。
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  array $paginated
 * @param  array $default
 * @return array
 */
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';

    return $default;
}
```

<a name="conditional-attributes"></a>
### 条件付き属性

特定の条件が満たされた場合にのみ属性をリソースレスポンスに含めたい場合があります。たとえば、現在のユーザーが「管理者」である場合にのみ値を含めたい場合です。Laravelはこの状況を支援するためのさまざまなヘルパーメソッドを提供しています。`when` メソッドは、条件付きで属性をリソースレスポンスに追加するために使用できます。

```php
/**
 * リソースを配列に変換する。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

この例では、認証されたユーザーの `isAdmin` メソッドが `true` を返す場合にのみ、`secret` キーが最終的なリソースレスポンスに返されます。メソッドが `false` を返す場合、`secret` キーはクライアントに送信される前にリソースレスポンスから削除されます。`when` メソッドを使用すると、配列を構築する際に条件文に頼ることなく、リソースを表現的に定義できます。

`when` メソッドは、2番目の引数としてクロージャも受け入れ、指定された条件が `true` の場合にのみ結果の値を計算できます。

```php
'secret' => $this->when($request->user()->isAdmin(), function () {
    return 'secret-value';
}),
```

`whenHas` メソッドは、属性が実際に基礎となるモデルに存在する場合に属性を含めるために使用できます。

```php
'name' => $this->whenHas('name'),
```

さらに、`whenNotNull` メソッドは、属性が null でない場合にリソースレスポンスに属性を含めるために使用できます。

```php
'name' => $this->whenNotNull($this->name),
```

<a name="merging-conditional-attributes"></a>
#### 条件付き属性のマージ

条件付き属性をマージする必要がある場合があります。たとえば、現在のユーザーが管理者である場合にのみ、特定の配列をリソースレスポンスに含めたい場合です。Laravelはこのための `mergeWhen` メソッドを提供しています。このメソッドは、指定された条件が `true` の場合にのみ、属性をレスポンスに含めます。

```php
/**
 * リソースを配列に変換する。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
    ];
}
```

> WARNING:  
> このメソッドは、ソートされていない配列で使用する場合にのみ適しています。`mergeWhen` メソッドをソートされた配列で使用すると、期待される出力順序が壊れる可能性があります。

場合によっては、同じ条件に基づいてリソースレスポンスにのみ含まれるべき複数の属性があるかもしれません。この場合、`mergeWhen`メソッドを使用して、指定された条件が`true`の場合にのみ属性をレスポンスに含めることができます。

```php
/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

繰り返しますが、指定された条件が`false`の場合、これらの属性はクライアントに送信される前にリソースレスポンスから削除されます。

> WARNING:  
> `mergeWhen`メソッドは、文字列キーと数値キーが混在する配列内で使用してはいけません。さらに、順序付けられていない数値キーを持つ配列内でも使用してはいけません。

<a name="conditional-relationships"></a>
### 条件付きリレーションシップ

属性の条件付き読み込みに加えて、モデルにリレーションシップが既に読み込まれているかどうかに基づいて、リソースレスポンスにリレーションシップを条件付きで含めることができます。これにより、コントローラーがモデルに読み込むべきリレーションシップを決定し、リソースは実際に読み込まれた場合にのみそれらを簡単に含めることができます。最終的に、リソース内の「N+1」クエリ問題を簡単に回避できます。

`whenLoaded`メソッドを使用して、リレーションシップを条件付きで読み込むことができます。不要なリレーションシップの読み込みを避けるために、このメソッドはリレーションシップ自体ではなく、リレーションシップの名前を受け取ります。

```php
use App\Http\Resources\PostResource;

/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

この例では、リレーションシップが読み込まれていない場合、`posts`キーはクライアントに送信される前にリソースレスポンスから削除されます。

<a name="conditional-relationship-counts"></a>
#### 条件付きリレーションシップのカウント

リレーションシップの条件付き読み込みに加えて、モデルにリレーションシップのカウントが読み込まれているかどうかに基づいて、リソースレスポンスにリレーションシップの「カウント」を条件付きで含めることができます。

```php
new UserResource($user->loadCount('posts'));
```

`whenCounted`メソッドを使用して、リソースレスポンスにリレーションシップのカウントを条件付きで含めることができます。このメソッドは、リレーションシップのカウントが存在しない場合に不要な属性を含めないようにします。

```php
/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts_count' => $this->whenCounted('posts'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

この例では、`posts`リレーションシップのカウントが読み込まれていない場合、`posts_count`キーはクライアントに送信される前にリソースレスポンスから削除されます。

他の集計タイプ、例えば`avg`、`sum`、`min`、`max`も、`whenAggregated`メソッドを使用して条件付きで読み込むことができます。

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-information"></a>
#### 条件付きピボット情報

リソースレスポンスにリレーションシップ情報を条件付きで含めることに加えて、`whenPivotLoaded`メソッドを使用して、多対多リレーションシップの中間テーブルからデータを条件付きで含めることができます。`whenPivotLoaded`メソッドは、ピボットテーブルの名前を最初の引数として受け取ります。2番目の引数は、ピボット情報がモデルで利用可能な場合に返される値を返すクロージャである必要があります。

```php
/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

リレーションシップが[カスタム中間テーブルモデル](eloquent-relationships.md#defining-custom-intermediate-table-models)を使用している場合、中間テーブルモデルのインスタンスを`whenPivotLoaded`メソッドの最初の引数として渡すことができます。

```php
'expires_at' => $this->whenPivotLoaded(new Membership, function () {
    return $this->pivot->expires_at;
}),
```

中間テーブルが`pivot`以外のアクセサを使用している場合、`whenPivotLoadedAs`メソッドを使用できます。

```php
/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

<a name="adding-meta-data"></a>
### メタデータの追加

一部のJSON API標準では、リソースやリソースコレクションのレスポンスにメタデータを追加する必要があります。これには、リソースや関連リソースへの`links`や、リソース自体に関するメタデータが含まれることがあります。リソースに追加のメタデータを返す必要がある場合は、`toArray`メソッドに含めます。例えば、リソースコレクションを変換する際に`links`情報を含めることができます。

```php
/**
 * リソースを配列に変換します。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

リソースから追加のメタデータを返す場合、ページネーションレスポンスを返す際にLaravelが自動的に追加する`links`や`meta`キーを誤って上書きすることを心配する必要はありません。定義した追加の`links`は、ページネーターによって提供されるリンクとマージされます。

<a name="top-level-meta-data"></a>
#### トップレベルのメタデータ

リソースが返される最も外側のリソースである場合にのみ、特定のメタデータをリソースレスポンスに含めたい場合があります。通常、これにはレスポンス全体に関するメタ情報が含まれます。このメタデータを定義するには、リソースクラスに`with`メソッドを追加します。このメソッドは、リソースが最も外側のリソースとして変換される場合にのみ、リソースレスポンスに含まれるメタデータの配列を返す必要があります。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * リソースコレクションを配列に変換します。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * リソース配列と共に返されるべき追加データを取得します。
     *
     * @return array<string, mixed>
     */
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

<a name="adding-meta-data-when-constructing-resources"></a>
#### リソース構築時にメタデータを追加

ルートまたはコントローラーでリソースインスタンスを構築する際に、トップレベルのデータを追加することもできます。`additional`メソッドは、すべてのリソースで利用可能で、リソースレスポンスに追加されるべきデータの配列を受け取ります。

```php
return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
```

<a name="resource-responses"></a>
## リソースレスポンス

すでに読んだように、リソースはルートやコントローラーから直接返すことができます。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

ただし、クライアントに送信される前に送信HTTPレスポンスをカスタマイズする必要がある場合があります。これを実現するには2つの方法があります。まず、リソースに`response`メソッドをチェーンすることができます。このメソッドは`Illuminate\Http\JsonResponse`インスタンスを返し、レスポンスのヘッダーを完全に制御できます。

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```

あるいは、リソース自体に`withResponse`メソッドを定義することもできます。このメソッドは、リソースがレスポンスとして最も外側のリソースとして返されるときに呼び出されます。

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * リソースを配列に変換します。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

    /**
     * リソースレスポンスをカスタマイズします。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     * @return void
     */
    public function withResponse(Request $request, $response)
    {
        $response->header('X-Value', 'True');
    }
}
```

```php
namespace App\Http\Resources;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * リソースを配列に変換します。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * リソースの送信レスポンスをカスタマイズします。
     */
    public function withResponse(Request $request, JsonResponse $response): void
    {
        $response->header('X-Value', 'True');
    }
}
```

