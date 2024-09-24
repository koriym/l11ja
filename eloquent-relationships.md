# Eloquent: リレーションシップ

- [はじめに](#introduction)
- [リレーションシップの定義](#defining-relationships)
    - [1対1](#one-to-one)
    - [1対多](#one-to-many)
    - [1対多（逆） / 所属](#one-to-many-inverse)
    - [多の中の1](#has-one-of-many)
    - [中間テーブルを介した1対1](#has-one-through)
    - [中間テーブルを介した1対多](#has-many-through)
- [多対多のリレーションシップ](#many-to-many)
    - [中間テーブルのカラムの取得](#retrieving-intermediate-table-columns)
    - [中間テーブルのカラムによるクエリのフィルタリング](#filtering-queries-via-intermediate-table-columns)
    - [中間テーブルのカラムによるクエリのソート](#ordering-queries-via-intermediate-table-columns)
    - [カスタム中間テーブルモデルの定義](#defining-custom-intermediate-table-models)
- [ポリモーフィックリレーションシップ](#polymorphic-relationships)
    - [1対1](#one-to-one-polymorphic-relations)
    - [1対多](#one-to-many-polymorphic-relations)
    - [多の中の1](#one-of-many-polymorphic-relations)
    - [多対多](#many-to-many-polymorphic-relations)
    - [カスタムポリモーフィックタイプ](#custom-polymorphic-types)
- [動的リレーションシップ](#dynamic-relationships)
- [リレーションシップのクエリ](#querying-relations)
    - [リレーションシップメソッド vs. 動的プロパティ](#relationship-methods-vs-dynamic-properties)
    - [リレーションシップの存在のクエリ](#querying-relationship-existence)
    - [リレーションシップの不在のクエリ](#querying-relationship-absence)
    - [モーフ対リレーションシップのクエリ](#querying-morph-to-relationships)
- [関連モデルの集約](#aggregating-related-models)
    - [関連モデルのカウント](#counting-related-models)
    - [その他の集約関数](#other-aggregate-functions)
    - [モーフ対リレーションシップの関連モデルのカウント](#counting-related-models-on-morph-to-relationships)
- [Eagerローディング](#eager-loading)
    - [Eagerロードの制約](#constraining-eager-loads)
    - [遅延Eagerローディング](#lazy-eager-loading)
    - [遅延ローディングの防止](#preventing-lazy-loading)
- [関連モデルの挿入と更新](#inserting-and-updating-related-models)
    - [`save`メソッド](#the-save-method)
    - [`create`メソッド](#the-create-method)
    - [所属リレーションシップ](#updating-belongs-to-relationships)
    - [多対多のリレーションシップ](#updating-many-to-many-relationships)
- [親のタイムスタンプの更新](#touching-parent-timestamps)

<a name="introduction"></a>
## はじめに

データベーステーブルはしばしば互いに関連しています。例えば、ブログ投稿には多くのコメントがあるかもしれませんし、注文はそれを行ったユーザーに関連しているかもしれません。Eloquentはこれらのリレーションシップの管理と操作を容易にし、さまざまな一般的なリレーションシップをサポートしています。

<div class="content-list" markdown="1">

- [1対1](#one-to-one)
- [1対多](#one-to-many)
- [多対多](#many-to-many)
- [中間テーブルを介した1対1](#has-one-through)
- [中間テーブルを介した1対多](#has-many-through)
- [1対1（ポリモーフィック）](#one-to-one-polymorphic-relations)
- [1対多（ポリモーフィック）](#one-to-many-polymorphic-relations)
- [多対多（ポリモーフィック）](#many-to-many-polymorphic-relations)

</div>

<a name="defining-relationships"></a>
## リレーションシップの定義

Eloquentのリレーションシップは、Eloquentモデルクラスのメソッドとして定義されます。リレーションシップは強力な[クエリビルダ](queries.md)としても機能するため、リレーションシップをメソッドとして定義することで、強力なメソッドチェーンとクエリ機能が提供されます。例えば、この`posts`リレーションシップに追加のクエリ制約をチェーンすることができます。

    $user->posts()->where('active', 1)->get();

しかし、リレーションシップの使用に深く入る前に、Eloquentがサポートする各タイプのリレーションシップを定義する方法を学びましょう。

<a name="one-to-one"></a>
### 1対1

1対1のリレーションシップは、非常に基本的なタイプのデータベースリレーションシップです。例えば、`User`モデルは1つの`Phone`モデルに関連付けられている可能性があります。このリレーションシップを定義するために、`User`モデルに`phone`メソッドを配置します。`phone`メソッドは`hasOne`メソッドを呼び出し、その結果を返す必要があります。`hasOne`メソッドは、モデルの`Illuminate\Database\Eloquent\Model`基底クラスを介してモデルで利用可能です。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasOne;

    class User extends Model
    {
        /**
         * ユーザーに関連付けられた電話を取得します。
         */
        public function phone(): HasOne
        {
            return $this->hasOne(Phone::class);
        }
    }

`hasOne`メソッドに渡される最初の引数は、関連モデルクラスの名前です。リレーションシップが定義されると、Eloquentの動的プロパティを使用して関連レコードを取得できます。動的プロパティを使用すると、リレーションシップメソッドにアクセスすることができます。

    $phone = User::find(1)->phone;

Eloquentは、リレーションシップの外部キーを親モデル名に基づいて決定します。この場合、`Phone`モデルは自動的に`user_id`外部キーを持つと想定されます。この規約をオーバーライドしたい場合は、`hasOne`メソッドに2番目の引数を渡すことができます。

    return $this->hasOne(Phone::class, 'foreign_key');

さらに、Eloquentは外部キーが親の主キーカラムの値と一致することを想定しています。つまり、Eloquentはユーザーの`id`カラムの値を`Phone`レコードの`user_id`カラムで探します。リレーションシップが`id`以外の主キー値を使用する場合、またはモデルの`$primaryKey`プロパティ以外の主キー値を使用する場合は、`hasOne`メソッドに3番目の引数を渡すことができます。

    return $this->hasOne(Phone::class, 'foreign_key', 'local_key');

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### リレーションシップの逆の定義

これで、`User`モデルから`Phone`モデルにアクセスできるようになりました。次に、`Phone`モデルにリレーションシップを定義して、電話を所有するユーザーにアクセスできるようにしましょう。`hasOne`リレーションシップの逆を定義するには、`belongsTo`メソッドを使用します。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Phone extends Model
    {
        /**
         * 電話を所有するユーザーを取得します。
         */
        public function user(): BelongsTo
        {
            return $this->belongsTo(User::class);
        }
    }

`user`メソッドを呼び出すと、Eloquentは`User`モデルを見つけようとします。このモデルは、`Phone`モデルの`user_id`カラムと一致する`id`を持っています。

Eloquentは、リレーションシップメソッドの名前を調べ、メソッド名に`_id`を付けることで外部キー名を決定します。したがって、この場合、Eloquentは`Phone`モデルに`user_id`カラムがあると想定します。ただし、`Phone`モデルの外部キーが`user_id`でない場合は、`belongsTo`メソッドにカスタムキー名を2番目の引数として渡すことができます。

    /**
     * 電話を所有するユーザーを取得します。
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'foreign_key');
    }

親モデルが`id`を主キーとして使用していない場合、または関連モデルを見つけるために別のカラムを使用したい場合は、`belongsTo`メソッドに親テーブルのカスタムキーを3番目の引数として渡すことができます。

    /**
     * 電話を所有するユーザーを取得します。
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
    }

<a name="one-to-many"></a>
### 1対多

1対多のリレーションシップは、1つのモデルが1つ以上の子モデルの親となるリレーションシップを定義するために使用されます。例えば、ブログ投稿には無限の数のコメントがある可能性があります。他のすべてのEloquentリレーションシップと同様に、1対多のリレーションシップはEloquentモデルにメソッドを定義することで定義されます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasMany;

    class Post extends Model
    {
        /**
         * ブログ投稿のコメントを取得します。
         */
        public function comments(): HasMany
        {
            return $this->hasMany(Comment::class);
        }
    }

Eloquentは、`Comment`モデルの適切な外部キーカラムを自動的に決定します。慣例により、Eloquentは親モデルの「スネークケース」名を取り、それに`_id`を付けます。したがって、この例では、Eloquentは`Comment`モデルの外部キーカラムが`post_id`であると想定します。

リレーションシップメソッドが定義されると、`comments`プロパティにアクセスすることで関連するコメントの[コレクション](eloquent-collections.md)にアクセスできます。Eloquentは「動的リレーションシッププロパティ」を提供するため、リレーションシップメソッドにアクセスできます。

    use App\Models\Post;

    $comments = Post::find(1)->comments;

    foreach ($comments as $comment) {
        // ...
    }

すべてのリレーションシップはクエリビルダとしても機能するため、`comments`メソッドを呼び出し、クエリに条件をチェーンすることでリレーションシップクエリにさらに制約を追加できます。

    $comment = Post::find(1)->comments()
                        ->where('title', 'foo')
                        ->first();

`hasOne`メソッドと同様に、外部キーとローカルキーをオーバーライドするために`hasMany`メソッドに追加の引数を渡すこともできます。

    return $this->hasMany(Comment::class, 'foreign_key');

    return $this->hasMany(Comment::class, 'foreign_key', 'local_key');

<a name="automatically-hydrating-parent-models-on-children"></a>
#### 子モデル上の親モデルの自動ハイドレーション

EloquentのEagerローディングを利用していても、子モデルをループしながら親モデルにアクセスしようとすると、「N + 1」クエリの問題が発生する可能性があります。

```php
$posts = Post::with('comments')->get();

```php
foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

上記の例では、"N + 1" クエリ問題が発生しています。なぜなら、`Post` モデルのコメントは一括読み込みされていますが、Eloquent は自動的に各 `Comment` モデルに親の `Post` をハイドレートしないからです。

もし Eloquent に親モデルを子モデルに自動的にハイドレートさせたい場合、`hasMany` リレーションを定義する際に `chaperone` メソッドを呼び出すことができます。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * ブログ投稿のコメントを取得します。
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

または、実行時に親モデルの自動ハイドレーションをオプトインする場合、リレーションを一括読み込みする際に `chaperone` メソッドを呼び出すことができます。

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### 一対多 (逆) / Belongs To

これで投稿のすべてのコメントにアクセスできるようになったので、コメントが親投稿にアクセスできるようにするためのリレーションを定義しましょう。`hasMany` リレーションの逆を定義するには、子モデルに `belongsTo` メソッドを呼び出すリレーションメソッドを定義します。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * コメントを所有する投稿を取得します。
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

リレーションが定義されたら、`post` の「動的リレーションプロパティ」にアクセスすることで、コメントの親投稿を取得できます。

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

上記の例では、Eloquent は `Comment` モデルの `post_id` 列と一致する `id` を持つ `Post` モデルを見つけようとします。

Eloquent は、リレーションメソッドの名前を調べ、メソッド名に `_` を付けて親モデルの主キーカラム名を追加することで、デフォルトの外部キー名を決定します。したがって、この例では、Eloquent は `comments` テーブルの `Post` モデルの外部キーが `post_id` であると仮定します。

ただし、リレーションの外部キーがこれらの規約に従っていない場合、`belongsTo` メソッドにカスタム外部キー名を2番目の引数として渡すことができます。

```php
/**
 * コメントを所有する投稿を取得します。
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

親モデルが `id` を主キーとして使用していない場合、または異なるカラムを使用して関連モデルを見つけたい場合、`belongsTo` メソッドに親テーブルのカスタムキーを3番目の引数として渡すことができます。

```php
/**
 * コメントを所有する投稿を取得します。
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

<a name="default-models"></a>
#### デフォルトモデル

`belongsTo`、`hasOne`、`hasOneThrough`、および `morphOne` リレーションには、指定されたリレーションが `null` の場合に返されるデフォルトモデルを定義できます。このパターンは、[Null Object パターン](https://en.wikipedia.org/wiki/Null_Object_pattern)と呼ばれ、コード内の条件チェックを削減するのに役立ちます。以下の例では、`user` リレーションは `Post` モデルにユーザーがアタッチされていない場合、空の `App\Models\User` モデルを返します。

```php
/**
 * 投稿の作成者を取得します。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

デフォルトモデルに属性を設定するには、`withDefault` メソッドに配列またはクロージャを渡すことができます。

```php
/**
 * 投稿の作成者を取得します。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

/**
 * 投稿の作成者を取得します。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

<a name="querying-belongs-to-relationships"></a>
#### Belongs To リレーションのクエリ

"belongs to" リレーションの子をクエリする場合、対応する Eloquent モデルを取得するために `where` 句を手動で構築できます。

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

ただし、`whereBelongsTo` メソッドを使用すると、指定されたモデルの適切なリレーションと外部キーを自動的に決定できるため、より便利です。

```php
$posts = Post::whereBelongsTo($user)->get();
```

また、[コレクション](eloquent-collections.md)インスタンスを `whereBelongsTo` メソッドに提供することもできます。その場合、Laravel はコレクション内の親モデルのいずれかに属するモデルを取得します。

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

デフォルトでは、Laravel はモデルのクラス名に基づいて関連するリレーションを決定します。ただし、`whereBelongsTo` メソッドの2番目の引数としてリレーション名を手動で指定することもできます。

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### 多対一の中の一つ

モデルが多くの関連モデルを持つ場合がありますが、リレーションの中で「最新」または「最古」の関連モデルを簡単に取得したい場合があります。たとえば、`User` モデルは多くの `Order` モデルに関連付けられているかもしれませんが、ユーザーが最後に注文した注文を簡単に操作する方法を定義したい場合があります。これは、`hasOne` リレーションタイプと `ofMany` メソッドを組み合わせて実現できます。

```php
/**
 * ユーザーの最新の注文を取得します。
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

同様に、リレーションの「最古」または最初の関連モデルを取得するメソッドを定義できます。

```php
/**
 * ユーザーの最古の注文を取得します。
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

デフォルトでは、`latestOfMany` および `oldestOfMany` メソッドは、モデルの主キーに基づいて最新または最古の関連モデルを取得します。これはソート可能である必要があります。ただし、異なるソート基準を使用して大きなリレーションから単一のモデルを取得したい場合があります。

たとえば、`ofMany` メソッドを使用して、ユーザーの最も高価な注文を取得できます。`ofMany` メソッドは、ソート可能なカラムを最初の引数として受け取り、関連モデルをクエリする際に適用する集約関数 (`min` または `max`) を受け取ります。

```php
/**
 * ユーザーの最も高価な注文を取得します。
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> WARNING:  
> PostgreSQL は UUID カラムに対して `MAX` 関数を実行することをサポートしていないため、現在 PostgreSQL UUID カラムと一緒に one-of-many リレーションを使用することはできません。

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### 多対一のリレーションを一対一のリレーションに変換する

`latestOfMany`、`oldestOfMany`、または `ofMany` メソッドを使用して単一のモデルを取得する場合、同じモデルに対して既に「多対一」のリレーションが定義されていることがよくあります。便宜上、Laravel では、リレーションに `one` メソッドを呼び出すことで、このリレーションを「一対一」のリレーションに簡単に変換できます。

```php
/**
 * ユーザーの注文を取得します。
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * ユーザーの最も高価な注文を取得します。
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### 高度な多対一の中の一つのリレーション

より高度な「多対一の中の一つ」のリレーションを構築することも可能です。たとえば、`Product` モデルは多くの関連する `Price` モデルを持つことがあり、新しい価格が公開された後もシステムに保持される場合があります。さらに、製品の新しい価格データは、将来の日付に効果を発揮するために事前に公開されることがあります。これは `published_at` カラムを介して行われます。

つまり、公開日が未来ではない最新の公開価格を取得する必要があります。さらに、2つの価格が同じ公開日を持つ場合、ID が最も大きい価格を優先します。これを実現するには、ソート可能なカラムを含む配列を `ofMany` メソッドに渡す必要があります。これにより、最新の価格が決定されます。さらに、`ofMany` メソッドの2番目の引数としてクロージャを提供します。このクロージャは、リレーションクエリに追加の公開日制約を追加する責任を負います。

```php
/**
 * 製品の現在の価格を取得します。
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

<a name="has-one-through"></a>
### 一対一 (中間)

「一対一 (中間)」リレーションは、別のモデルとの一対一のリレーションを定義します。ただし、このリレーションは、宣言モデルが第三のモデルを介して別のモデルのインスタンスと一致できることを示します。

例えば、車両修理店のアプリケーションでは、各 `Mechanic` モデルは1つの `Car` モデルに関連付けられ、各 `Car` モデルは1つの `Owner` モデルに関連付けられるかもしれません。メカニックとオーナーはデータベース内で直接の関係を持っていませんが、メカニックは `Car` モデルを通じてオーナーにアクセスできます。この関係を定義するために必要なテーブルを見てみましょう：

    mechanics
        id - integer
        name - string

    cars
        id - integer
        model - string
        mechanic_id - integer

    owners
        id - integer
        name - string
        car_id - integer

関係のためのテーブル構造を見たので、`Mechanic` モデルでこの関係を定義しましょう：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasOneThrough;

    class Mechanic extends Model
    {
        /**
         * 車のオーナーを取得する。
         */
        public function carOwner(): HasOneThrough
        {
            return $this->hasOneThrough(Owner::class, Car::class);
        }
    }

`hasOneThrough` メソッドに渡される最初の引数は、アクセスしたい最終モデルの名前で、2番目の引数は中間モデルの名前です。

また、関係に関わるすべてのモデルで関係がすでに定義されている場合、`through` メソッドを呼び出し、それらの関係の名前を指定することで、"has-one-through" 関係を流れるように定義できます。例えば、`Mechanic` モデルが `cars` 関係を持ち、`Car` モデルが `owner` 関係を持つ場合、メカニックとオーナーを接続する "has-one-through" 関係を次のように定義できます：

```php
// 文字列ベースの構文...
return $this->through('cars')->has('owner');

// 動的な構文...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### キーの規約

関係のクエリを実行する際には、典型的なEloquentの外部キー規約が使用されます。関係のキーをカスタマイズしたい場合は、`hasOneThrough` メソッドに3番目と4番目の引数として渡すことができます。3番目の引数は中間モデルの外部キーの名前です。4番目の引数は最終モデルの外部キーの名前です。5番目の引数はローカルキーで、6番目の引数は中間モデルのローカルキーです：

    class Mechanic extends Model
    {
        /**
         * 車のオーナーを取得する。
         */
        public function carOwner(): HasOneThrough
        {
            return $this->hasOneThrough(
                Owner::class,
                Car::class,
                'mechanic_id', // carsテーブルの外部キー...
                'car_id', // ownersテーブルの外部キー...
                'id', // mechanicsテーブルのローカルキー...
                'id' // carsテーブルのローカルキー...
            );
        }
    }

また、前述のように、関係に関わるすべてのモデルで関係がすでに定義されている場合、`through` メソッドを呼び出し、それらの関係の名前を指定することで、"has-one-through" 関係を流れるように定義できます。このアプローチは、既存の関係で定義されたキー規約を再利用する利点があります：

```php
// 文字列ベースの構文...
return $this->through('cars')->has('owner');

// 動的な構文...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### Has Many Through

"has-many-through" 関係は、中間の関係を通じて遠い関係にアクセスする便利な方法を提供します。例えば、[Laravel Vapor](https://vapor.laravel.com) のようなデプロイメントプラットフォームを構築しているとしましょう。`Project` モデルは、中間の `Environment` モデルを通じて多くの `Deployment` モデルにアクセスするかもしれません。この例を使用して、特定のプロジェクトのすべてのデプロイメントを簡単に集めることができます。この関係を定義するために必要なテーブルを見てみましょう：

    projects
        id - integer
        name - string

    environments
        id - integer
        project_id - integer
        name - string

    deployments
        id - integer
        environment_id - integer
        commit_hash - string

関係のためのテーブル構造を見たので、`Project` モデルでこの関係を定義しましょう：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\HasManyThrough;

    class Project extends Model
    {
        /**
         * プロジェクトのすべてのデプロイメントを取得する。
         */
        public function deployments(): HasManyThrough
        {
            return $this->hasManyThrough(Deployment::class, Environment::class);
        }
    }

`hasManyThrough` メソッドに渡される最初の引数は、アクセスしたい最終モデルの名前で、2番目の引数は中間モデルの名前です。

また、関係に関わるすべてのモデルで関係がすでに定義されている場合、`through` メソッドを呼び出し、それらの関係の名前を指定することで、"has-many-through" 関係を流れるように定義できます。例えば、`Project` モデルが `environments` 関係を持ち、`Environment` モデルが `deployments` 関係を持つ場合、プロジェクトとデプロイメントを接続する "has-many-through" 関係を次のように定義できます：

```php
// 文字列ベースの構文...
return $this->through('environments')->has('deployments');

// 動的な構文...
return $this->throughEnvironments()->hasDeployments();
```

`Deployment` モデルのテーブルには `project_id` 列が含まれていませんが、`hasManyThrough` 関係は `$project->deployments` を通じてプロジェクトのデプロイメントにアクセスできます。これらのモデルを取得するために、Eloquent は中間の `Environment` モデルのテーブルの `project_id` 列を調べます。関連する環境IDを見つけた後、それらを使用して `Deployment` モデルのテーブルをクエリします。

<a name="has-many-through-key-conventions"></a>
#### キーの規約

関係のクエリを実行する際には、典型的なEloquentの外部キー規約が使用されます。関係のキーをカスタマイズしたい場合は、`hasManyThrough` メソッドに3番目と4番目の引数として渡すことができます。3番目の引数は中間モデルの外部キーの名前です。4番目の引数は最終モデルの外部キーの名前です。5番目の引数はローカルキーで、6番目の引数は中間モデルのローカルキーです：

    class Project extends Model
    {
        public function deployments(): HasManyThrough
        {
            return $this->hasManyThrough(
                Deployment::class,
                Environment::class,
                'project_id', // environmentsテーブルの外部キー...
                'environment_id', // deploymentsテーブルの外部キー...
                'id', // projectsテーブルのローカルキー...
                'id' // environmentsテーブルのローカルキー...
            );
        }
    }

また、前述のように、関係に関わるすべてのモデルで関係がすでに定義されている場合、`through` メソッドを呼び出し、それらの関係の名前を指定することで、"has-many-through" 関係を流れるように定義できます。このアプローチは、既存の関係で定義されたキー規約を再利用する利点があります：

```php
// 文字列ベースの構文...
return $this->through('environments')->has('deployments');

// 動的な構文...
return $this->throughEnvironments()->hasDeployments();
```

<a name="many-to-many"></a>
## 多対多関係

多対多関係は、`hasOne` や `hasMany` 関係よりも少し複雑です。多対多関係の例としては、ユーザーが多くの役割を持ち、それらの役割がアプリケーション内の他のユーザーと共有される場合があります。例えば、ユーザーは "Author" と "Editor" の役割を割り当てられるかもしれませんが、それらの役割は他のユーザーにも割り当てられるかもしれません。つまり、ユーザーは多くの役割を持ち、役割は多くのユーザーを持ちます。

<a name="many-to-many-table-structure"></a>
#### テーブル構造

この関係を定義するには、`users`、`roles`、`role_user` の3つのデータベーステーブルが必要です。`role_user` テーブルは関連するモデル名のアルファベット順から派生し、`user_id` と `role_id` 列を含みます。このテーブルは、ユーザーと役割をリンクする中間テーブルとして使用されます。

役割が多くのユーザーに属することができるため、`roles` テーブルに `user_id` 列を単純に配置することはできません。これは、役割が単一のユーザーにのみ属することを意味します。複数のユーザーに役割を割り当てるためのサポートを提供するために、`role_user` テーブルが必要です。関係のテーブル構造を次のようにまとめることができます：

    users
        id - integer
        name - string

    roles
        id - integer
        name - string

    role_user
        user_id - integer
        role_id - integer

<a name="many-to-many-model-structure"></a>
#### モデル構造

多対多関係は、`belongsToMany` メソッドの結果を返すメソッドを書くことで定義されます。`belongsToMany` メソッドは、アプリケーションのすべてのEloquentモデルが使用する `Illuminate\Database\Eloquent\Model` 基底クラスによって提供されます。例えば、`User` モデルに `roles` メソッドを定義しましょう。このメソッドに渡される最初の引数は、関連するモデルクラスの名前です：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class User extends Model
    {
        /**
         * ユーザーに属する役割。
         */
        public function roles(): BelongsToMany
        {
            return $this->belongsToMany(Role::class);
        }
    }

リレーションが定義されると、`roles` ダイナミックリレーションシッププロパティを使用してユーザーのロールにアクセスできます:

    use App\Models\User;

    $user = User::find(1);

    foreach ($user->roles as $role) {
        // ...
    }

すべてのリレーションはクエリビルダとしても機能するため、`roles` メソッドを呼び出してクエリにさらに条件を追加することで、リレーションクエリにさらに制約を追加できます:

    $roles = User::find(1)->roles()->orderBy('name')->get();

リレーションの中間テーブルのテーブル名を決定するために、Eloquent は2つの関連するモデル名をアルファベット順に結合します。ただし、この規則を自由に上書きできます。`belongsToMany` メソッドに2番目の引数を渡すことでこれを行うことができます:

    return $this->belongsToMany(Role::class, 'role_user');

中間テーブルの名前をカスタマイズするだけでなく、テーブルのキーの列名もカスタマイズできます。`belongsToMany` メソッドに追加の引数を渡すことでこれを行います。3番目の引数は、リレーションを定義しているモデルの外部キー名で、4番目の引数は、結合しているモデルの外部キー名です:

    return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### リレーションの逆の定義

多対多リレーションの「逆」を定義するには、関連モデルにも `belongsToMany` メソッドの結果を返すメソッドを定義する必要があります。ユーザー / ロールの例を完了するために、`Role` モデルに `users` メソッドを定義しましょう:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class Role extends Model
    {
        /**
         * ロールに属するユーザー。
         */
        public function users(): BelongsToMany
        {
            return $this->belongsToMany(User::class);
        }
    }

ご覧のとおり、リレーションは `User` モデルの対応物とまったく同じように定義されていますが、`App\Models\User` モデルを参照する点が異なります。`belongsToMany` メソッドを再利用しているため、多対多リレーションの「逆」を定義するときに、すべての通常のテーブルとキーのカスタマイズオプションを利用できます。

<a name="retrieving-intermediate-table-columns"></a>
### 中間テーブルの列の取得

すでに学んだように、多対多の関係を操作するには中間テーブルが必要です。Eloquent はこのテーブルと対話するための非常に便利な方法をいくつか提供しています。たとえば、`User` モデルが関連する多くの `Role` モデルを持っているとします。この関係にアクセスした後、モデルの `pivot` 属性を使用して中間テーブルにアクセスできます:

    use App\Models\User;

    $user = User::find(1);

    foreach ($user->roles as $role) {
        echo $role->pivot->created_at;
    }

取得した各 `Role` モデルには自動的に `pivot` 属性が割り当てられていることに注意してください。この属性には中間テーブルを表すモデルが含まれています。

デフォルトでは、`pivot` モデルにはモデルキーのみが存在します。中間テーブルに追加の属性が含まれている場合は、リレーションを定義するときにそれらを指定する必要があります:

    return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');

中間テーブルに Eloquent によって自動的に維持される `created_at` と `updated_at` のタイムスタンプを持たせたい場合は、リレーションを定義するときに `withTimestamps` メソッドを呼び出します:

    return $this->belongsToMany(Role::class)->withTimestamps();

> WARNING:  
> Eloquent の自動的に維持されるタイムスタンプを利用する中間テーブルには、`created_at` と `updated_at` のタイムスタンプ列の両方が必要です。

<a name="customizing-the-pivot-attribute-name"></a>
#### `pivot` 属性名のカスタマイズ

前述のように、中間テーブルの属性は `pivot` 属性を介してモデルでアクセスできます。ただし、この属性の名前をアプリケーション内での目的をよりよく反映するように自由にカスタマイズできます。

たとえば、アプリケーションにポッドキャストを購読できるユーザーが含まれている場合、ユーザーとポッドキャストの間に多対多の関係がある可能性があります。この場合、中間テーブルの属性を `pivot` ではなく `subscription` に名前を変更したい場合があります。これは、リレーションを定義するときに `as` メソッドを使用して行うことができます:

    return $this->belongsToMany(Podcast::class)
                    ->as('subscription')
                    ->withTimestamps();

カスタム中間テーブル属性を指定したら、カスタマイズされた名前を使用して中間テーブルデータにアクセスできます:

    $users = User::with('podcasts')->get();

    foreach ($users->flatMap->podcasts as $podcast) {
        echo $podcast->subscription->created_at;
    }

<a name="filtering-queries-via-intermediate-table-columns"></a>
### 中間テーブルの列を介したクエリのフィルタリング

`belongsToMany` リレーションクエリによって返される結果をフィルタリングするために、リレーションを定義するときに `wherePivot`、`wherePivotIn`、`wherePivotNotIn`、`wherePivotBetween`、`wherePivotNotBetween`、`wherePivotNull`、および `wherePivotNotNull` メソッドを使用できます:

    return $this->belongsToMany(Role::class)
                    ->wherePivot('approved', 1);

    return $this->belongsToMany(Role::class)
                    ->wherePivotIn('priority', [1, 2]);

    return $this->belongsToMany(Role::class)
                    ->wherePivotNotIn('priority', [1, 2]);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNull('expired_at');

    return $this->belongsToMany(Podcast::class)
                    ->as('subscriptions')
                    ->wherePivotNotNull('expired_at');

<a name="ordering-queries-via-intermediate-table-columns"></a>
### 中間テーブルの列を介したクエリの並べ替え

`belongsToMany` リレーションクエリによって返される結果を並べ替えるために、`orderByPivot` メソッドを使用できます。次の例では、ユーザーの最新のバッジをすべて取得します:

    return $this->belongsToMany(Badge::class)
                    ->where('rank', 'gold')
                    ->orderByPivot('created_at', 'desc');

<a name="defining-custom-intermediate-table-models"></a>
### カスタム中間テーブルモデルの定義

多対多リレーションの中間テーブルを表すためにカスタムモデルを定義したい場合は、リレーションを定義するときに `using` メソッドを呼び出すことができます。カスタムピボットモデルを使用すると、ピボットモデルに追加の動作を定義する機会が得られます。

カスタム多対多ピボットモデルは `Illuminate\Database\Eloquent\Relations\Pivot` クラスを拡張する必要がありますが、カスタムポリモーフィック多対多ピボットモデルは `Illuminate\Database\Eloquent\Relations\MorphPivot` クラスを拡張する必要があります。たとえば、カスタム `RoleUser` ピボットモデルを使用する `Role` モデルを定義できます:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsToMany;

    class Role extends Model
    {
        /**
         * ロールに属するユーザー。
         */
        public function users(): BelongsToMany
        {
            return $this->belongsToMany(User::class)->using(RoleUser::class);
        }
    }

`RoleUser` モデルを定義するときは、`Illuminate\Database\Eloquent\Relations\Pivot` クラスを拡張する必要があります:

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Relations\Pivot;

    class RoleUser extends Pivot
    {
        // ...
    }

> WARNING:  
> ピボットモデルは `SoftDeletes` トレイトを使用できません。ピボットレコードをソフトデリートする必要がある場合は、ピボットモデルを実際の Eloquent モデルに変換することを検討してください。

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### カスタムピボットモデルとインクリメントID

カスタムピボットモデルを使用する多対多リレーションを定義し、そのピボットモデルに自動インクリメントプライマリキーがある場合は、カスタムピボットモデルクラスに `incrementing` プロパティを `true` に設定する必要があります。

    /**
     * ID が自動インクリメントされるかどうかを示します。
     *
     * @var bool
     */
    public $incrementing = true;

<a name="polymorphic-relationships"></a>
## ポリモーフィックリレーションシップ

ポリモーフィックリレーションシップにより、子モデルは単一の関連付けを使用して複数のタイプのモデルに属することができます。たとえば、ブログ投稿とビデオを共有するアプリケーションを構築しているとします。このようなアプリケーションでは、`Comment` モデルが `Post` モデルと `Video` モデルの両方に属することができます。

<a name="one-to-one-polymorphic-relations"></a>
### 一対一 (ポリモーフィック)

<a name="one-to-one-polymorphic-table-structure"></a>
#### テーブル構造

一対一のポリモーフィックリレーションは、通常の一対一のリレーションに似ています。ただし、子モデルは単一の関連付けを使用して複数のタイプのモデルに属することができます。たとえば、ブログ `Post` と `User` が `Image` モデルへのポリモーフィックリレーションを共有するとします。一対一のポリモーフィックリレーションを使用すると、投稿とユーザーに関連付けられた一意の画像の単一のテーブルを持つことができます。まず、テーブル構造を見てみましょう:

    posts
        id - integer
        name - string

    users
        id - integer
        name - string

    images
        id - integer
        url - string
        imageable_id - integer
        imageable_type - string

`images` テーブルの `imageable_id` と `imageable_type` カラムに注目してください。`imageable_id` カラムには投稿やユーザーのID値が格納され、`imageable_type` カラムには親モデルのクラス名が格納されます。Eloquentは `imageable_type` カラムを使用して、`imageable` リレーションにアクセスする際にどの「タイプ」の親モデルを返すかを決定します。この場合、カラムには `App\Models\Post` または `App\Models\User` が格納されます。

<a name="one-to-one-polymorphic-model-structure"></a>
#### モデル構造

次に、このリレーションを構築するために必要なモデル定義を見てみましょう：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * 親の imageable モデル（ユーザーまたは投稿）を取得します。
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    /**
     * 投稿の画像を取得します。
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    /**
     * ユーザーの画像を取得します。
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>
#### リレーションの取得

データベーステーブルとモデルが定義されたら、モデルを介してリレーションにアクセスできます。例えば、投稿の画像を取得するには、`image` 動的リレーションプロパティにアクセスします：

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

ポリモーフィックモデルの親を取得するには、`morphTo` を呼び出すメソッドの名前にアクセスします。この場合、`Image` モデルの `imageable` メソッドです。そのメソッドを動的リレーションプロパティとしてアクセスします：

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

`Image` モデルの `imageable` リレーションは、画像を所有するモデルのタイプに応じて、`Post` または `User` インスタンスを返します。

<a name="morph-one-to-one-key-conventions"></a>
#### キーの規約

必要に応じて、ポリモーフィック子モデルによって使用される「id」と「type」カラムの名前を指定できます。その場合、`morphTo` メソッドにリレーションの名前を最初の引数として常に渡してください。通常、この値はメソッド名と一致する必要があるため、PHPの `__FUNCTION__` 定数を使用できます：

```php
/**
 * 画像が属するモデルを取得します。
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}
```

<a name="one-to-many-polymorphic-relations"></a>
### 一対多（ポリモーフィック）

<a name="one-to-many-polymorphic-table-structure"></a>
#### テーブル構造

一対多のポリモーフィックリレーションは、通常の一対多のリレーションと似ていますが、子モデルは単一の関連付けを使用して複数のタイプのモデルに属することができます。例えば、アプリケーションのユーザーが投稿やビデオに「コメント」できると想像してください。ポリモーフィックリレーションを使用すると、投稿とビデオの両方にコメントを含むために単一の `comments` テーブルを使用できます。まず、このリレーションを構築するために必要なテーブル構造を見てみましょう：

```
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

<a name="one-to-many-polymorphic-model-structure"></a>
#### モデル構造

次に、このリレーションを構築するために必要なモデル定義を見てみましょう：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * 親の commentable モデル（投稿またはビデオ）を取得します。
     */
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    /**
     * 投稿のすべてのコメントを取得します。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Video extends Model
{
    /**
     * ビデオのすべてのコメントを取得します。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>
#### リレーションの取得

データベーステーブルとモデルが定義されたら、モデルの動的リレーションプロパティを介してリレーションにアクセスできます。例えば、投稿のすべてのコメントにアクセスするには、`comments` 動的プロパティを使用します：

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```

ポリモーフィック子モデルの親を取得するには、`morphTo` を呼び出すメソッドの名前にアクセスします。この場合、`Comment` モデルの `commentable` メソッドです。そのメソッドを動的リレーションプロパティとしてアクセスして、コメントの親モデルにアクセスします：

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

`Comment` モデルの `commentable` リレーションは、コメントの親モデルのタイプに応じて、`Post` または `Video` インスタンスを返します。

<a name="polymorphic-automatically-hydrating-parent-models-on-children"></a>
#### 子モデルに親モデルを自動的にハイドレート

Eloquent の eager loading を利用しても、子モデルをループしながら親モデルにアクセスしようとすると、「N + 1」クエリ問題が発生する可能性があります：

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

上記の例では、「N + 1」クエリ問題が発生しています。なぜなら、すべての `Post` モデルに対してコメントが eager loaded されていますが、Eloquent は自動的に各子 `Comment` モデルに親 `Post` をハイドレートしないためです。

Eloquent に親モデルを子モデルに自動的にハイドレートさせたい場合、`hasMany` リレーションを定義する際に `chaperone` メソッドを呼び出すことができます：

```php
class Post extends Model
{
    /**
     * 投稿のすべてのコメントを取得します。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}
```

または、実行時に親モデルの自動ハイドレートをオプトインする場合、リレーションを eager loading する際に `chaperone` メソッドを呼び出すことができます：

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-of-many-polymorphic-relations"></a>
### 多対一（ポリモーフィック）

モデルが多くの関連モデルを持つ場合がありますが、リレーションの「最新」または「最古」の関連モデルと簡単にやり取りしたい場合があります。例えば、`User` モデルは多くの `Image` モデルに関連しているかもしれませんが、ユーザーがアップロードした最新の画像と簡単にやり取りする方法を定義したい場合があります。`morphOne` リレーションタイプと `ofMany` メソッドを組み合わせてこれを実現できます：

```php
/**
 * ユーザーの最新の画像を取得します。
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

同様に、リレーションの「最古」または最初の関連モデルを取得するメソッドを定義できます：

```php
/**
 * ユーザーの最古の画像を取得します。
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

デフォルトでは、`latestOfMany` および `oldestOfMany` メソッドは、モデルの主キーに基づいて最新または最古の関連モデルを取得します。これはソート可能である必要があります。ただし、異なるソート基準を使用してより大きなリレーションから単一のモデルを取得したい場合があります。

例えば、`ofMany` メソッドを使用して、ユーザーの最も「いいね」された画像を取得できます。`ofMany` メソッドは、ソート可能なカラムを最初の引数として受け取り、関連モデルをクエリする際に適用する集約関数（`min` または `max`）を受け取ります：

```php
/**
 * ユーザーの最も人気のある画像を取得します。
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> NOTE:  
> より高度な「多対一」リレーションを構築することも可能です。詳細については、[has one of many ドキュメント](#advanced-has-one-of-many-relationships)を参照してください。

<a name="many-to-many-polymorphic-relations"></a>
### 多対多（ポリモーフィック）

<a name="many-to-many-polymorphic-table-structure"></a>
#### テーブル構造
```

多対多のポリモーフィックリレーションは、「モルフ1」と「モルフ多」のリレーションよりも少し複雑です。例えば、`Post`モデルと`Video`モデルが`Tag`モデルとポリモーフィックリレーションを共有することができます。この状況で多対多のポリモーフィックリレーションを使用すると、アプリケーションは投稿やビデオに関連付けられる一意のタグの単一のテーブルを持つことができます。まず、このリレーションを構築するために必要なテーブル構造を見てみましょう。

```plaintext
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

> NOTE:  
> ポリモーフィックな多対多リレーションに飛び込む前に、通常の[多対多リレーション](#many-to-many)に関するドキュメントを読むことで恩恵を受けるかもしれません。

<a name="many-to-many-polymorphic-model-structure"></a>
#### モデル構造

次に、モデルにリレーションを定義する準備が整いました。`Post`モデルと`Video`モデルは、どちらも`tags`メソッドを持ち、これはEloquentモデルクラスが提供する`morphToMany`メソッドを呼び出します。

`morphToMany`メソッドは、関連モデルの名前と「リレーション名」を受け取ります。中間テーブル名とそれに含まれるキーに基づいて、リレーションを「taggable」と呼ぶことにします。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * 投稿のすべてのタグを取得する。
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>
#### リレーションの逆の定義

次に、`Tag`モデルで、可能な親モデルごとにメソッドを定義する必要があります。この例では、`posts`メソッドと`videos`メソッドを定義します。これらのメソッドは、`morphedByMany`メソッドの結果を返す必要があります。

`morphedByMany`メソッドは、関連モデルの名前と「リレーション名」を受け取ります。中間テーブル名とそれに含まれるキーに基づいて、リレーションを「taggable」と呼ぶことにします。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * このタグが割り当てられているすべての投稿を取得する。
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * このタグが割り当てられているすべてのビデオを取得する。
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>
#### リレーションの取得

データベーステーブルとモデルが定義されたら、モデルを介してリレーションにアクセスできます。例えば、投稿のすべてのタグにアクセスするには、`tags`動的リレーションプロパティを使用できます。

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

ポリモーフィックな子モデルからポリモーフィックリレーションの親を取得するには、`morphedByMany`を呼び出すメソッドの名前にアクセスします。この場合、`Tag`モデルの`posts`メソッドまたは`videos`メソッドです。

```php
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
    // ...
}

foreach ($tag->videos as $video) {
    // ...
}
```

<a name="custom-polymorphic-types"></a>
### カスタムポリモーフィックタイプ

デフォルトでは、Laravelは関連モデルの完全修飾クラス名を使用して「タイプ」を保存します。例えば、上記の1対多のリレーションの例では、`Comment`モデルが`Post`または`Video`モデルに属する可能性があり、デフォルトの`commentable_type`はそれぞれ`App\Models\Post`または`App\Models\Video`になります。ただし、アプリケーションの内部構造からこれらの値を切り離したい場合があります。

例えば、モデル名を「タイプ」として使用する代わりに、`post`や`video`のような単純な文字列を使用することができます。これにより、モデルがリネームされても、データベース内のポリモーフィック「タイプ」列の値は有効なままです。

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

`enforceMorphMap`メソッドは、`App\Providers\AppServiceProvider`クラスの`boot`メソッドで呼び出すか、別のサービスプロバイダを作成することができます。

実行時に特定のモデルのモルフアリアスを決定するには、モデルの`getMorphClass`メソッドを使用します。逆に、モルフアリアスに関連付けられた完全修飾クラス名を決定するには、`Relation::getMorphedModel`メソッドを使用します。

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> WARNING:  
> 既存のアプリケーションに「モルフアリアスマップ」を追加する場合、データベース内のすべてのポリモーフィックな`*_type`列の値が完全修飾クラス名のままである場合、それらを「マップ」名に変換する必要があります。

<a name="dynamic-relationships"></a>
### 動的リレーション

`resolveRelationUsing`メソッドを使用して、実行時にEloquentモデル間のリレーションを定義することができます。通常のアプリケーション開発では推奨されませんが、Laravelパッケージの開発では時々役立つことがあります。

`resolveRelationUsing`メソッドは、最初の引数として目的のリレーション名を受け取ります。メソッドに渡される2番目の引数は、モデルインスタンスを受け取り、有効なEloquentリレーション定義を返すクロージャである必要があります。通常、動的リレーションは[サービスプロバイダ](providers.md)の`boot`メソッド内で設定する必要があります。

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> WARNING:  
> 動的リレーションを定義する場合、常にEloquentリレーションメソッドに明示的なキー名の引数を提供してください。

<a name="querying-relations"></a>
## リレーションのクエリ

すべてのEloquentリレーションはメソッドを介して定義されるため、それらのメソッドを呼び出して、関連モデルを実際にクエリを実行せずにリレーションのインスタンスを取得できます。さらに、すべてのタイプのEloquentリレーションも[クエリビルダ](queries.md)として機能し、リレーションクエリに制約を追加してから、最終的にデータベースに対してSQLクエリを実行できます。

例えば、`User`モデルが多くの`Post`モデルに関連付けられているブログアプリケーションを想像してみてください。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * ユーザーのすべての投稿を取得する。
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

`posts`リレーションをクエリし、リレーションに追加の制約を追加することができます。

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

リレーションにはLaravelの[クエリビルダ](queries.md)の任意のメソッドを使用できるため、クエリビルダのドキュメントを確認して、利用可能なすべてのメソッドを学んでください。

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### リレーションの後に`orWhere`句をチェーンする

上記の例で示したように、リレーションに追加の制約を自由に追加できます。ただし、`orWhere`句をリレーションにチェーンする場合は注意が必要です。`orWhere`句は、リレーション制約と同じレベルで論理的にグループ化されます。

```php
$user->posts()
        ->where('active', 1)
        ->orWhere('votes', '>=', 100)
        ->get();
```

上記の例では、次のSQLが生成されます。`or`句は、100以上の投票を持つ_任意の_投稿を返すようにクエリに指示します。クエリは特定のユーザーに制約されなくなります。

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

ほとんどの状況では、条件チェックを括弧で囲む[論理グループ](queries.md#logical-grouping)を使用して、条件チェックをグループ化する必要があります。

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
        ->where(function (Builder $query) {
            return $query->where('active', 1)
                         ->orWhere('votes', '>=', 100);
        })
        ->get();
```

上記の例では、次のSQLが生成されます。論理グループが制約を適切にグループ化し、クエリが特定のユーザーに制約されたままであることに注意してください。

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### リレーションメソッド vs. 動的プロパティ

Eloquentリレーションクエリに追加の制約を追加する必要がない場合、リレーションにプロパティとしてアクセスできます。例えば、`User`モデルと`Post`モデルを引き続き使用する場合、ユーザーのすべての投稿に次のようにアクセスできます。

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

動的なリレーションシッププロパティは「遅延読み込み」を行います。つまり、実際にアクセスしたときにのみリレーションシップデータを読み込みます。このため、開発者はしばしば[積極的な読み込み](#eager-loading)を使用して、モデルを読み込んだ後にアクセスすることがわかっているリレーションシップを事前に読み込みます。積極的な読み込みは、モデルのリレーションシップを読み込むために実行する必要があるSQLクエリを大幅に削減します。

<a name="querying-relationship-existence"></a>
### リレーションシップの存在をクエリする

モデルレコードを取得する際、リレーションシップの存在に基づいて結果を制限したい場合があります。たとえば、少なくとも1つのコメントがあるすべてのブログ投稿を取得したいとします。そのためには、リレーションシップの名前を`has`および`orHas`メソッドに渡すことができます。

    use App\Models\Post;

    // 少なくとも1つのコメントがあるすべての投稿を取得する
    $posts = Post::has('comments')->get();

さらに、演算子とカウント値を指定してクエリをカスタマイズすることもできます。

    // 3つ以上のコメントがあるすべての投稿を取得する
    $posts = Post::has('comments', '>=', 3)->get();

ネストされた`has`ステートメントは、「ドット」記法を使用して構築できます。たとえば、少なくとも1つのコメントが少なくとも1つの画像を持つすべての投稿を取得することができます。

    // 少なくとも1つのコメントが画像を持つ投稿を取得する
    $posts = Post::has('comments.images')->get();

さらに強力な機能が必要な場合は、`whereHas`および`orWhereHas`メソッドを使用して、`has`クエリに追加のクエリ制約を定義することができます。たとえば、コメントの内容を検査することができます。

    use Illuminate\Database\Eloquent\Builder;

    // 少なくとも1つのコメントがcode%のような単語を含む投稿を取得する
    $posts = Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    })->get();

    // 少なくとも10のコメントがcode%のような単語を含む投稿を取得する
    $posts = Post::whereHas('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    }, '>=', 10)->get();

> WARNING:  
> Eloquentは現在、データベースをまたいだリレーションシップの存在クエリをサポートしていません。リレーションシップは同じデータベース内に存在する必要があります。

<a name="inline-relationship-existence-queries"></a>
#### インラインリレーションシップの存在クエリ

リレーションシップの存在を単一の簡単なwhere条件でクエリする場合、`whereRelation`、`orWhereRelation`、`whereMorphRelation`、および`orWhereMorphRelation`メソッドを使用すると便利です。たとえば、承認されていないコメントを持つすべての投稿をクエリすることができます。

    use App\Models\Post;

    $posts = Post::whereRelation('comments', 'is_approved', false)->get();

もちろん、クエリビルダーの`where`メソッドの呼び出しと同様に、演算子を指定することもできます。

    $posts = Post::whereRelation(
        'comments', 'created_at', '>=', now()->subHour()
    )->get();

<a name="querying-relationship-absence"></a>
### リレーションシップの不在をクエリする

モデルレコードを取得する際、リレーションシップの不在に基づいて結果を制限したい場合があります。たとえば、コメントが**ない**すべてのブログ投稿を取得したいとします。そのためには、リレーションシップの名前を`doesntHave`および`orDoesntHave`メソッドに渡すことができます。

    use App\Models\Post;

    $posts = Post::doesntHave('comments')->get();

さらに強力な機能が必要な場合は、`whereDoesntHave`および`orWhereDoesntHave`メソッドを使用して、`doesntHave`クエリに追加のクエリ制約を追加することができます。たとえば、コメントの内容を検査することができます。

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::whereDoesntHave('comments', function (Builder $query) {
        $query->where('content', 'like', 'code%');
    })->get();

「ドット」記法を使用して、ネストされたリレーションシップに対してクエリを実行することができます。たとえば、次のクエリはコメントがないすべての投稿を取得します。ただし、禁止されていない著者からのコメントを持つ投稿は結果に含まれます。

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
        $query->where('banned', 0);
    })->get();

<a name="querying-morph-to-relationships"></a>
### Morph To リレーションシップのクエリ

「morph to」リレーションシップの存在をクエリするために、`whereHasMorph`および`whereDoesntHaveMorph`メソッドを使用することができます。これらのメソッドは、最初の引数としてリレーションシップの名前を受け取ります。次に、クエリに含めたい関連モデルの名前を受け取ります。最後に、リレーションシップクエリをカスタマイズするクロージャを提供することができます。

    use App\Models\Comment;
    use App\Models\Post;
    use App\Models\Video;
    use Illuminate\Database\Eloquent\Builder;

    // 投稿またはビデオに関連するコメントで、タイトルがcode%のようなものを取得する
    $comments = Comment::whereHasMorph(
        'commentable',
        [Post::class, Video::class],
        function (Builder $query) {
            $query->where('title', 'like', 'code%');
        }
    )->get();

    // 投稿に関連するコメントで、タイトルがcode%のようでないものを取得する
    $comments = Comment::whereDoesntHaveMorph(
        'commentable',
        Post::class,
        function (Builder $query) {
            $query->where('title', 'like', 'code%');
        }
    )->get();

関連するポリモーフィックモデルの「タイプ」に基づいてクエリ制約を追加する必要がある場合があります。`whereHasMorph`メソッドに渡されるクロージャは、2番目の引数として`$type`値を受け取ることができます。この引数を使用して、構築されているクエリの「タイプ」を検査することができます。

    use Illuminate\Database\Eloquent\Builder;

    $comments = Comment::whereHasMorph(
        'commentable',
        [Post::class, Video::class],
        function (Builder $query, string $type) {
            $column = $type === Post::class ? 'content' : 'title';

            $query->where($column, 'like', 'code%');
        }
    )->get();

<a name="querying-all-morph-to-related-models"></a>
#### すべての関連モデルのクエリ

ポリモーフィックモデルの配列を渡す代わりに、ワイルドカード値として`*`を指定することができます。これにより、Laravelにデータベースからすべての可能なポリモーフィックタイプを取得するよう指示されます。Laravelはこの操作を実行するために追加のクエリを実行します。

    use Illuminate\Database\Eloquent\Builder;

    $comments = Comment::whereHasMorph('commentable', '*', function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    })->get();

<a name="aggregating-related-models"></a>
## 関連モデルの集約

<a name="counting-related-models"></a>
### 関連モデルのカウント

特定のリレーションシップの関連モデルの数をカウントしたい場合がありますが、実際にモデルを読み込む必要はありません。これを実現するために、`withCount`メソッドを使用することができます。`withCount`メソッドは、結果のモデルに`{relation}_count`属性を配置します。

    use App\Models\Post;

    $posts = Post::withCount('comments')->get();

    foreach ($posts as $post) {
        echo $post->comments_count;
    }

配列を`withCount`メソッドに渡すことで、複数のリレーションシップの「カウント」を追加し、クエリに追加の制約を追加することもできます。

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
        $query->where('content', 'like', 'code%');
    }])->get();

    echo $posts[0]->votes_count;
    echo $posts[0]->comments_count;

リレーションシップのカウント結果に別名を付けることもできます。これにより、同じリレーションシップに対して複数のカウントを行うことができます。

    use Illuminate\Database\Eloquent\Builder;

    $posts = Post::withCount([
        'comments',
        'comments as pending_comments_count' => function (Builder $query) {
            $query->where('approved', false);
        },
    ])->get();

    echo $posts[0]->comments_count;
    echo $posts[0]->pending_comments_count;

<a name="deferred-count-loading"></a>
#### 遅延カウント読み込み

`loadCount`メソッドを使用すると、親モデルがすでに取得された後にリレーションシップのカウントを読み込むことができます。

    $book = Book::first();

    $book->loadCount('genres');

カウントクエリに追加のクエリ制約を設定する必要がある場合は、カウントしたいリレーションシップをキーとする配列を渡すことができます。配列の値は、クエリビルダーインスタンスを受け取るクロージャである必要があります。

    $book->loadCount(['reviews' => function (Builder $query) {
        $query->where('rating', 5);
    }])

<a name="relationship-counting-and-custom-select-statements"></a>
#### リレーションシップのカウントとカスタムselectステートメント

`withCount`を`select`ステートメントと組み合わせる場合、`select`メソッドの後に`withCount`を呼び出すようにしてください。

    $posts = Post::select(['title', 'body'])
                    ->withCount('comments')
                    ->get();

<a name="other-aggregate-functions"></a>
### その他の集約関数

`withCount`メソッドに加えて、Eloquentは`withMin`、`withMax`、`withAvg`、`withSum`、および`withExists`メソッドを提供します。これらのメソッドは、結果のモデルに`{relation}_{function}_{column}`属性を配置します。

    use App\Models\Post;

    $posts = Post::withSum('comments', 'votes')->get();

    foreach ($posts as $post) {
        echo $post->comments_sum_votes;
    }

集約関数の結果に別の名前を付けたい場合は、独自のエイリアスを指定することができます。

    $posts = Post::withSum('comments as total_comments', 'votes')->get();

    foreach ($posts as $post) {
        echo $post->total_comments;
    }

`loadCount`メソッドと同様に、これらのメソッドの遅延バージョンも利用できます。これらの追加の集約操作は、すでに取得されたEloquentモデルに対して実行できます。

    $post = Post::first();

    $post->loadSum('comments', 'votes');

これらの集約メソッドを`select`ステートメントと組み合わせる場合、`select`メソッドの後に集約メソッドを呼び出すようにしてください。

    $posts = Post::select(['title', 'body'])
                    ->withExists('comments')
                    ->get();

<a name="counting-related-models-on-morph-to-relationships"></a>
### Morph To リレーションシップに関連するモデルのカウント

「morph to」リレーションシップを一緒に読み込みたい場合、そのリレーションシップによって返される可能性のあるさまざまなエンティティの関連モデルのカウントも取得したい場合は、`with`メソッドと`morphTo`リレーションシップの`morphWithCount`メソッドを組み合わせて使用できます。

この例では、`Photo`モデルと`Post`モデルが`ActivityFeed`モデルを作成する可能性があるとします。`ActivityFeed`モデルは、`parentable`という「morph to」リレーションシップを定義しており、これにより特定の`ActivityFeed`インスタンスの親`Photo`または`Post`モデルを取得できます。さらに、`Photo`モデルは「多くの」`Tag`モデルを持ち、`Post`モデルは「多くの」`Comment`モデルを持つとします。

さて、`ActivityFeed`インスタンスを取得し、各`ActivityFeed`インスタンスの`parentable`親モデルを一緒に読み込みたいとします。さらに、各親フォトに関連付けられたタグの数と、各親投稿に関連付けられたコメントの数を取得したいとします。

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::with([
        'parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWithCount([
                Photo::class => ['tags'],
                Post::class => ['comments'],
            ]);
        }])->get();

<a name="morph-to-deferred-count-loading"></a>
#### 遅延カウント読み込み

すでに一連の`ActivityFeed`モデルを取得しており、現在はアクティビティフィードに関連付けられたさまざまな`parentable`モデルのネストされたリレーションシップのカウントを読み込みたいとします。これを実現するには、`loadMorphCount`メソッドを使用できます。

    $activities = ActivityFeed::with('parentable')->get();

    $activities->loadMorphCount('parentable', [
        Photo::class => ['tags'],
        Post::class => ['comments'],
    ]);

<a name="eager-loading"></a>
## 一緒に読み込み（Eager Loading）

Eloquentリレーションシップにプロパティとしてアクセスすると、関連するモデルは「遅延読み込み」されます。これは、プロパティに最初にアクセスするまでリレーションシップデータが実際に読み込まれないことを意味します。しかし、Eloquentは親モデルをクエリするときにリレーションシップを「一緒に読み込む」ことができます。一緒に読み込みは、「N + 1」クエリ問題を軽減します。N + 1クエリ問題を説明するために、`Book`モデルが`Author`モデルに「属する」と考えてみましょう。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Book extends Model
    {
        /**
         * 本を書いた著者を取得します。
         */
        public function author(): BelongsTo
        {
            return $this->belongsTo(Author::class);
        }
    }

さて、すべての本とその著者を取得してみましょう。

    use App\Models\Book;

    $books = Book::all();

    foreach ($books as $book) {
        echo $book->author->name;
    }

このループは、データベーステーブル内のすべての本を取得するために1つのクエリを実行し、次に各本の著者を取得するために別のクエリを実行します。したがって、25冊の本がある場合、上記のコードは26回のクエリを実行します。1つは元の本のクエリで、25回は各本の著者を取得するための追加のクエリです。

幸いなことに、一緒に読み込みを使用してこの操作を2つのクエリに減らすことができます。クエリを構築するときに、`with`メソッドを使用してどのリレーションシップを一緒に読み込むかを指定できます。

    $books = Book::with('author')->get();

    foreach ($books as $book) {
        echo $book->author->name;
    }

この操作では、2つのクエリのみが実行されます。1つはすべての本を取得するためのクエリで、もう1つはすべての本の著者を取得するためのクエリです。

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### 複数のリレーションシップを一緒に読み込む

時には、いくつかの異なるリレーションシップを一緒に読み込む必要があるかもしれません。その場合は、リレーションシップの配列を`with`メソッドに渡すだけです。

    $books = Book::with(['author', 'publisher'])->get();

<a name="nested-eager-loading"></a>
#### ネストされた一緒に読み込み

リレーションシップのリレーションシップを一緒に読み込むには、「ドット」構文を使用できます。たとえば、すべての本の著者とすべての著者の個人連絡先を一緒に読み込むには、次のようにします。

    $books = Book::with('author.contacts')->get();

または、`with`メソッドにネストされた配列を提供することで、複数のネストされたリレーションシップを一緒に読み込むことができます。これは、複数のネストされたリレーションシップを一緒に読み込む場合に便利です。

    $books = Book::with([
        'author' => [
            'contacts',
            'publisher',
        ],
    ])->get();

<a name="nested-eager-loading-morphto-relationships"></a>
#### ネストされた一緒に読み込み `morphTo` リレーションシップ

`morphTo`リレーションシップを一緒に読み込みたい場合、そのリレーションシップによって返される可能性のあるさまざまなエンティティのネストされたリレーションシップも一緒に読み込みたい場合は、`with`メソッドと`morphTo`リレーションシップの`morphWith`メソッドを組み合わせて使用できます。このメソッドを説明するために、次のモデルを考えてみましょう。

    <?php

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class ActivityFeed extends Model
    {
        /**
         * アクティビティフィードレコードの親を取得します。
         */
        public function parentable(): MorphTo
        {
            return $this->morphTo();
        }
    }

この例では、`Event`、`Photo`、`Post`モデルが`ActivityFeed`モデルを作成する可能性があるとします。さらに、`Event`モデルは`Calendar`モデルに属し、`Photo`モデルは`Tag`モデルに関連付けられ、`Post`モデルは`Author`モデルに属するとします。

これらのモデル定義とリレーションシップを使用して、`ActivityFeed`モデルインスタンスを取得し、すべての`parentable`モデルとそれぞれのネストされたリレーションシップを一緒に読み込むことができます。

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $activities = ActivityFeed::query()
        ->with(['parentable' => function (MorphTo $morphTo) {
            $morphTo->morphWith([
                Event::class => ['calendar'],
                Photo::class => ['tags'],
                Post::class => ['author'],
            ]);
        }])->get();

<a name="eager-loading-specific-columns"></a>
#### 特定のカラムを一緒に読み込む

取得するリレーションシップから常にすべてのカラムが必要なわけではないかもしれません。このため、Eloquentではリレーションシップから取得したいカラムを指定できます。

    $books = Book::with('author:id,name,book_id')->get();

> WARNING:  
> この機能を使用する場合、常に`id`カラムと関連する外部キーカラムを取得したいカラムのリストに含める必要があります。

<a name="eager-loading-by-default"></a>
#### デフォルトで一緒に読み込む

モデルを取得するときに常にいくつかのリレーションシップを読み込みたい場合があります。これを実現するには、モデルに`$with`プロパティを定義できます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\BelongsTo;

    class Book extends Model
    {
        /**
         * 常に読み込まれるべきリレーションシップ。
         *
         * @var array
         */
        protected $with = ['author'];

        /**
         * 本を書いた著者を取得します。
         */
        public function author(): BelongsTo
        {
            return $this->belongsTo(Author::class);
        }

        /**
         * 本のジャンルを取得します。
         */
        public function genre(): BelongsTo
        {
            return $this->belongsTo(Genre::class);
        }
    }

単一のクエリで`$with`プロパティからアイテムを削除したい場合は、`without`メソッドを使用できます。

    $books = Book::without('author')->get();

単一のクエリで`$with`プロパティ内のすべてのアイテムを上書きしたい場合は、`withOnly`メソッドを使用できます。

    $books = Book::withOnly('genre')->get();

<a name="constraining-eager-loads"></a>
### 一緒に読み込みの制約

時には、リレーションシップを一緒に読み込みたいが、一緒に読み込みクエリに追加のクエリ条件を指定したい場合があります。これを実現するには、リレーションシップ名をキーとし、一緒に読み込みクエリに追加の制約を追加するクロージャを値とする配列を`with`メソッドに渡します。

    use App\Models\User;
    use Illuminate\Contracts\Database\Eloquent\Builder;

    $users = User::with(['posts' => function (Builder $query) {
        $query->where('title', 'like', '%code%');
    }])->get();

この例では、Eloquentは投稿の`title`カラムに`code`という単語を含む投稿のみを一緒に読み込みます。他の[クエリビルダ](queries.md)メソッドを呼び出して、一緒に読み込み操作をさらにカスタマイズすることができます。

    $users = User::with(['posts' => function (Builder $query) {
        $query->orderBy('created_at', 'desc');
    }])->get();

<a name="constraining-eager-loading-of-morph-to-relationships"></a>
#### `morphTo` リレーションシップの一緒に読み込みの制約

`morphTo`リレーションシップを一緒に読み込む場合、Eloquentは各タイプの関連モデルを取得するために複数のクエリを実行します。`MorphTo`リレーションシップの`constrain`メソッドを使用して、これらの各クエリに追加の制約を追加できます。

    use Illuminate\Database\Eloquent\Relations\MorphTo;

    $comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
        $morphTo->constrain([
            Post::class => function ($query) {
                $query->whereNull('hidden_at');
            },
            Video::class => function ($query) {
                $query->where('type', 'educational');
            },
        ]);
    }])->get();

この例では、Eloquentは非表示になっていない投稿と`type`が"educational"の動画のみを積極的にロードします。

<a name="constraining-eager-loads-with-relationship-existence"></a>
#### リレーションの存在による積極的なロードの制約

リレーションの存在をチェックしながら、同じ条件に基づいてリレーションをロードする必要がある場合があります。例えば、特定のクエリ条件に一致する子の`Post`モデルを持つ`User`モデルのみを取得し、同時に一致する投稿を積極的にロードしたい場合があります。これは`withWhereHas`メソッドを使用して実現できます:

    use App\Models\User;

    $users = User::withWhereHas('posts', function ($query) {
        $query->where('featured', true);
    })->get();

<a name="lazy-eager-loading"></a>
### 遅延積極的ロード

親モデルが既に取得された後にリレーションを積極的にロードする必要がある場合があります。例えば、関連モデルをロードするかどうかを動的に決定する場合に便利です:

    use App\Models\Book;

    $books = Book::all();

    if ($someCondition) {
        $books->load('author', 'publisher');
    }

積極的なロードクエリに追加のクエリ制約を設定する必要がある場合、ロードしたいリレーションをキーとする配列を渡すことができます。配列の値はクエリインスタンスを受け取るクロージャインスタンスであるべきです:

    $author->load(['books' => function (Builder $query) {
        $query->orderBy('published_date', 'asc');
    }]);

リレーションがまだロードされていない場合にのみリレーションをロードするには、`loadMissing`メソッドを使用します:

    $book->loadMissing('author');

<a name="nested-lazy-eager-loading-morphto"></a>
#### ネストされた遅延積極的ロードと`morphTo`

`morphTo`リレーションを積極的にロードし、そのリレーションによって返される可能性のあるさまざまなエンティティのネストされたリレーションも積極的にロードしたい場合、`loadMorph`メソッドを使用できます。

このメソッドは、`morphTo`リレーションの名前を最初の引数として受け取り、モデル/リレーションのペアの配列を2番目の引数として受け取ります。このメソッドを説明するために、次のモデルを考えてみましょう:

    <?php

    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Database\Eloquent\Relations\MorphTo;

    class ActivityFeed extends Model
    {
        /**
         * Get the parent of the activity feed record.
         */
        public function parentable(): MorphTo
        {
            return $this->morphTo();
        }
    }

この例では、`Event`、`Photo`、`Post`モデルが`ActivityFeed`モデルを作成する可能性があると仮定します。さらに、`Event`モデルは`Calendar`モデルに属し、`Photo`モデルは`Tag`モデルに関連付けられ、`Post`モデルは`Author`モデルに属していると仮定します。

これらのモデル定義とリレーションを使用して、`ActivityFeed`モデルインスタンスを取得し、すべての`parentable`モデルとそれらのネストされたリレーションを積極的にロードできます:

    $activities = ActivityFeed::with('parentable')
        ->get()
        ->loadMorph('parentable', [
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);

<a name="preventing-lazy-loading"></a>
### 遅延ロードの防止

前述のように、リレーションの積極的なロードは、アプリケーションのパフォーマンスに大きなメリットをもたらすことが多いです。したがって、Laravelにリレーションの遅延ロードを常に防止するよう指示することができます。これを実現するには、Eloquentモデルクラスが提供する`preventLazyLoading`メソッドを呼び出すことができます。通常、このメソッドはアプリケーションの`AppServiceProvider`クラスの`boot`メソッド内で呼び出すべきです。

`preventLazyLoading`メソッドは、遅延ロードを防止するかどうかを示すオプションのブール引数を受け取ります。例えば、遅延ロードを非本番環境でのみ無効にしたい場合があります。これにより、本番環境は遅延ロードされたリレーションが誤って本番コードに含まれていても正常に機能し続けます:

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

遅延ロードを防止した後、アプリケーションがEloquentリレーションを遅延ロードしようとすると、`Illuminate\Database\LazyLoadingViolationException`例外がスローされます。

`handleLazyLoadingViolationsUsing`メソッドを使用して、遅延ロード違反の動作をカスタマイズすることができます。例えば、このメソッドを使用して、遅延ロード違反が例外で中断される代わりにログに記録されるように指示することができます:

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-models"></a>
## 関連モデルの挿入と更新

<a name="the-save-method"></a>
### `save`メソッド

Eloquentは、リレーションに新しいモデルを追加するための便利なメソッドを提供します。例えば、投稿に新しいコメントを追加する必要がある場合です。`Comment`モデルに`post_id`属性を手動で設定する代わりに、リレーションの`save`メソッドを使用してコメントを挿入できます:

    use App\Models\Comment;
    use App\Models\Post;

    $comment = new Comment(['message' => 'A new comment.']);

    $post = Post::find(1);

    $post->comments()->save($comment);

動的プロパティとして`comments`リレーションにアクセスするのではなく、`comments`メソッドを呼び出してリレーションのインスタンスを取得したことに注意してください。`save`メソッドは自動的に新しい`Comment`モデルに適切な`post_id`値を追加します。

複数の関連モデルを保存する必要がある場合、`saveMany`メソッドを使用できます:

    $post = Post::find(1);

    $post->comments()->saveMany([
        new Comment(['message' => 'A new comment.']),
        new Comment(['message' => 'Another new comment.']),
    ]);

`save`および`saveMany`メソッドは、指定されたモデルインスタンスを永続化しますが、新しく永続化されたモデルを親モデルに既にロードされているメモリ内のリレーションに追加しません。`save`または`saveMany`メソッドを使用した後にリレーションにアクセスする予定がある場合、`refresh`メソッドを使用してモデルとそのリレーションを再読み込みすることができます:

    $post->comments()->save($comment);

    $post->refresh();

    // 新しく保存されたコメントを含むすべてのコメント...
    $post->comments;

<a name="the-push-method"></a>
#### 再帰的にモデルとリレーションを保存

モデルとそのすべての関連リレーションを`save`したい場合、`push`メソッドを使用できます。この例では、`Post`モデルとそのコメント、およびコメントの著者が保存されます:

    $post = Post::find(1);

    $post->comments[0]->message = 'Message';
    $post->comments[0]->author->name = 'Author Name';

    $post->push();

`pushQuietly`メソッドを使用して、イベントを発生させずにモデルとその関連リレーションを保存することができます:

    $post->pushQuietly();

<a name="the-create-method"></a>
### `create`メソッド

`save`および`saveMany`メソッドに加えて、属性の配列を受け取り、モデルを作成し、データベースに挿入する`create`メソッドを使用することもできます。`save`と`create`の違いは、`save`が完全なEloquentモデルインスタンスを受け取るのに対し、`create`はプレーンなPHPの`array`を受け取ることです。`create`メソッドによって新しく作成されたモデルが返されます:

    use App\Models\Post;

    $post = Post::find(1);

    $comment = $post->comments()->create([
        'message' => 'A new comment.',
    ]);

`createMany`メソッドを使用して複数の関連モデルを作成できます:

    $post = Post::find(1);

    $post->comments()->createMany([
        ['message' => 'A new comment.'],
        ['message' => 'Another new comment.'],
    ]);

`createQuietly`および`createManyQuietly`メソッドを使用して、イベントを発生させずにモデルを作成することができます:

    $user = User::find(1);

    $user->posts()->createQuietly([
        'title' => 'Post title.',
    ]);

    $user->posts()->createManyQuietly([
        ['title' => 'First post.'],
        ['title' => 'Second post.'],
    ]);

`findOrNew`、`firstOrNew`、`firstOrCreate`、および`updateOrCreate`メソッドを使用して、[リレーション上のモデルを作成および更新](eloquent.md#upserts)することもできます。

> NOTE:  
> `create`メソッドを使用する前に、[マスアサインメント](eloquent.md#mass-assignment)のドキュメントを確認してください。

<a name="updating-belongs-to-relationships"></a>
### Belongs To リレーションの更新

子モデルを新しい親モデルに割り当てたい場合、`associate`メソッドを使用できます。この例では、`User`モデルは`Account`モデルへの`belongsTo`リレーションを定義しています。この`associate`メソッドは、子モデルの外部キーを設定します:

    use App\Models\Account;

    $account = Account::find(10);

    $user->account()->associate($account);

    $user->save();

親モデルを子モデルから削除するには、`dissociate`メソッドを使用できます。このメソッドは、リレーションの外部キーを`null`に設定します:

    $user->account()->dissociate();

    $user->save();

<a name="updating-many-to-many-relationships"></a>
### 多対多リレーションの更新

<a name="attaching-detaching"></a>
#### アタッチ / デタッチ
```

Eloquentは、多対多のリレーションシップをより便利に扱うためのメソッドも提供しています。例えば、ユーザーが多くの役割を持ち、役割が多くのユーザーを持つ場合を想像してみましょう。`attach`メソッドを使用して、リレーションシップの中間テーブルにレコードを挿入することで、ユーザーに役割を付与できます。

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

モデルにリレーションシップを付与する際、中間テーブルに挿入される追加データの配列を渡すこともできます。

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

ユーザーから役割を削除する必要がある場合もあります。多対多のリレーションシップレコードを削除するには、`detach`メソッドを使用します。`detach`メソッドは中間テーブルから適切なレコードを削除しますが、両方のモデルはデータベースに残ります。

```php
// ユーザーから単一の役割を外す...
$user->roles()->detach($roleId);

// ユーザーからすべての役割を外す...
$user->roles()->detach();
```

便宜上、`attach`と`detach`はIDの配列を入力として受け取ることもできます。

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
```

<a name="syncing-associations"></a>
#### 関連付けの同期

多対多の関連付けを構築するために、`sync`メソッドを使用することもできます。`sync`メソッドは、中間テーブルに配置するIDの配列を受け取ります。指定された配列に含まれないIDは、中間テーブルから削除されます。したがって、この操作が完了した後、指定された配列内のIDのみが中間テーブルに存在することになります。

```php
$user->roles()->sync([1, 2, 3]);
```

IDとともに追加の中間テーブルの値を渡すこともできます。

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

同期する各モデルIDに同じ中間テーブルの値を挿入したい場合は、`syncWithPivotValues`メソッドを使用できます。

```php
$user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);
```

指定された配列に含まれない既存のIDを切り離したくない場合は、`syncWithoutDetaching`メソッドを使用できます。

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### 関連付けのトグル

多対多のリレーションシップは、指定された関連モデルIDの付与状態を「トグル」する`toggle`メソッドも提供しています。指定されたIDが現在付与されている場合は切り離され、現在切り離されている場合は付与されます。

```php
$user->roles()->toggle([1, 2, 3]);
```

IDとともに追加の中間テーブルの値を渡すこともできます。

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="updating-a-record-on-the-intermediate-table"></a>
#### 中間テーブルのレコードを更新する

リレーションシップの中間テーブルの既存の行を更新する必要がある場合は、`updateExistingPivot`メソッドを使用できます。このメソッドは、中間レコードの外部キーと更新する属性の配列を受け取ります。

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## 親のタイムスタンプを更新する

モデルが別のモデルへの`belongsTo`または`belongsToMany`リレーションシップを定義している場合、例えば`Comment`が`Post`に属している場合、子モデルが更新されたときに親のタイムスタンプを自動的に更新すると便利なことがあります。

例えば、`Comment`モデルが更新されたとき、所有する`Post`の`updated_at`タイムスタンプを現在の日付と時刻に設定したい場合があります。これを実現するには、子モデルに`touches`プロパティを追加し、子モデルが更新されたときに`updated_at`タイムスタンプを更新する必要があるリレーションシップの名前を含めます。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * 更新されるリレーションシップ
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * コメントが属する投稿を取得する
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

> WARNING:  
> 親モデルのタイムスタンプは、子モデルがEloquentの`save`メソッドを使用して更新された場合にのみ更新されます。

