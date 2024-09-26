# データベース: クエリビルダ

- [はじめに](#introduction)
- [データベースクエリの実行](#running-database-queries)
    - [結果のチャンク化](#chunking-results)
    - [結果の遅延ストリーミング](#streaming-results-lazily)
    - [集計](#aggregates)
- [SELECT文](#select-statements)
- [生の式](#raw-expressions)
- [結合](#joins)
- [ユニオン](#unions)
- [基本的なWHERE句](#basic-where-clauses)
    - [WHERE句](#where-clauses)
    - [OR WHERE句](#or-where-clauses)
    - [WHERE NOT句](#where-not-clauses)
    - [WHERE ANY / ALL / NONE句](#where-any-all-none-clauses)
    - [JSON WHERE句](#json-where-clauses)
    - [追加のWHERE句](#additional-where-clauses)
    - [論理グループ化](#logical-grouping)
- [高度なWHERE句](#advanced-where-clauses)
    - [WHERE EXISTS句](#where-exists-clauses)
    - [サブクエリWHERE句](#subquery-where-clauses)
    - [フルテキストWHERE句](#full-text-where-clauses)
- [ソート、グループ化、制限とオフセット](#ordering-grouping-limit-and-offset)
    - [ソート](#ordering)
    - [グループ化](#grouping)
    - [制限とオフセット](#limit-and-offset)
- [条件付き句](#conditional-clauses)
- [INSERT文](#insert-statements)
    - [アップサート](#upserts)
- [UPDATE文](#update-statements)
    - [JSONカラムの更新](#updating-json-columns)
    - [インクリメントとデクリメント](#increment-and-decrement)
- [DELETE文](#delete-statements)
- [悲観的ロック](#pessimistic-locking)
- [デバッグ](#debugging)

<a name="introduction"></a>
## はじめに

Laravelのデータベースクエリビルダは、データベースクエリの作成と実行のための便利で流暢なインターフェースを提供します。アプリケーション内のほとんどのデータベース操作に使用でき、Laravelがサポートするすべてのデータベースシステムで完璧に動作します。

Laravelのクエリビルダは、PDOパラメータバインディングを使用して、アプリケーションをSQLインジェクション攻撃から保護します。クエリビルダに渡される文字列をクリーンにしたりサニタイズする必要はありません。

> WARNING:  
> PDOはカラム名のバインディングをサポートしていません。したがって、クエリで参照されるカラム名をユーザー入力で決定することは決して許可しないでください。"order by"カラムを含む。

<a name="running-database-queries"></a>
## データベースクエリの実行

<a name="retrieving-all-rows-from-a-table"></a>
#### テーブルからすべての行を取得

`DB`ファサードが提供する`table`メソッドを使用して、クエリを開始できます。`table`メソッドは、指定されたテーブルの流暢なクエリビルダインスタンスを返し、クエリにさらに制約を追加してから、`get`メソッドを使用してクエリの結果を取得できます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\DB;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * アプリケーションのすべてのユーザーのリストを表示します。
         */
        public function index(): View
        {
            $users = DB::table('users')->get();

            return view('user.index', ['users' => $users]);
        }
    }

`get`メソッドは、クエリの結果を含む`Illuminate\Support\Collection`インスタンスを返します。各結果はPHPの`stdClass`オブジェクトのインスタンスであり、オブジェクトのプロパティとしてカラムにアクセスできます。

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')->get();

    foreach ($users as $user) {
        echo $user->name;
    }

> NOTE:  
> Laravelのコレクションは、データのマッピングと削減のための非常に強力なメソッドを提供します。Laravelコレクションの詳細については、[コレクションのドキュメント](collections.md)を確認してください。

<a name="retrieving-a-single-row-column-from-a-table"></a>
#### テーブルから単一の行/カラムを取得

データベーステーブルから単一の行を取得する必要がある場合は、`DB`ファサードの`first`メソッドを使用できます。このメソッドは、単一の`stdClass`オブジェクトを返します。

    $user = DB::table('users')->where('name', 'John')->first();

    return $user->email;

データベーステーブルから単一の行を取得したいが、一致する行が見つからない場合に`Illuminate\Database\RecordNotFoundException`をスローする場合は、`firstOrFail`メソッドを使用できます。`RecordNotFoundException`がキャッチされない場合、404 HTTPレスポンスが自動的にクライアントに返されます。

    $user = DB::table('users')->where('name', 'John')->firstOrFail();

行全体が必要ない場合、`value`メソッドを使用してレコードから単一の値を抽出できます。このメソッドは、指定したカラムの値を直接返します。

    $email = DB::table('users')->where('name', 'John')->value('email');

`id`カラムの値によって単一の行を取得するには、`find`メソッドを使用します。

    $user = DB::table('users')->find(3);

<a name="retrieving-a-list-of-column-values"></a>
#### カラム値のリストを取得

単一のカラムの値を含む`Illuminate\Support\Collection`インスタンスを取得したい場合は、`pluck`メソッドを使用できます。この例では、ユーザーのタイトルのコレクションを取得します。

    use Illuminate\Support\Facades\DB;

    $titles = DB::table('users')->pluck('title');

    foreach ($titles as $title) {
        echo $title;
    }

結果のコレクションが使用するカラムを指定するには、`pluck`メソッドに2番目の引数を提供します。

    $titles = DB::table('users')->pluck('title', 'name');

    foreach ($titles as $name => $title) {
        echo $title;
    }

<a name="chunking-results"></a>
### 結果のチャンク化

何千ものデータベースレコードを操作する必要がある場合は、`DB`ファサードが提供する`chunk`メソッドを使用することを検討してください。このメソッドは一度に小さなチャンクの結果を取得し、各チャンクをクロージャに渡して処理します。たとえば、`users`テーブル全体を一度に100レコードずつ取得しましょう。

    use Illuminate\Support\Collection;
    use Illuminate\Support\Facades\DB;

    DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
        foreach ($users as $user) {
            // ...
        }
    });

クロージャから`false`を返すことで、さらなるチャンクの処理を停止できます。

    DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
        // レコードを処理...

        return false;
    });

結果をチャンク化しながらデータベースレコードを更新する場合、チャンク結果が予期せぬ方法で変更される可能性があります。チャンク化中に取得したレコードを更新する予定がある場合は、`chunkById`メソッドを代わりに使用することを常にお勧めします。このメソッドは、レコードの主キーに基づいて結果を自動的にページングします。

    DB::table('users')->where('active', false)
        ->chunkById(100, function (Collection $users) {
            foreach ($users as $user) {
                DB::table('users')
                    ->where('id', $user->id)
                    ->update(['active' => true]);
            }
        });

> WARNING:  
> チャンクコールバック内でレコードを更新または削除する場合、主キーまたは外部キーへの変更は、チャンククエリに影響を与える可能性があります。これにより、レコードがチャンク結果に含まれない可能性があります。

<a name="streaming-results-lazily"></a>
### 結果の遅延ストリーミング

`lazy`メソッドは、[`chunk`メソッド](#chunking-results)と同様に、クエリをチャンクで実行します。ただし、`lazy()`メソッドは[`LazyCollection`](collections.md#lazy-collections)を返し、結果を単一のストリームとして操作することができます。

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```

繰り返し処理中に取得したレコードを更新する予定がある場合は、代わりに`lazyById`または`lazyByIdDesc`メソッドを使用することをお勧めします。これらのメソッドは、レコードの主キーに基づいて結果を自動的にページングします。

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> WARNING:  
> 繰り返し処理中にレコードを更新または削除する場合、主キーまたは外部キーへの変更は、チャンククエリに影響を与える可能性があります。これにより、レコードが結果に含まれない可能性があります。

<a name="aggregates"></a>
### 集計

クエリビルダは、`count`、`max`、`min`、`avg`、`sum`などのさまざまな集計値を取得するためのメソッドも提供します。クエリを構築した後、これらのメソッドのいずれかを呼び出すことができます。

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')->count();

    $price = DB::table('orders')->max('price');

もちろん、これらのメソッドを他の句と組み合わせて、集計値の計算方法を微調整できます。

    $price = DB::table('orders')
                    ->where('finalized', 1)
                    ->avg('price');

<a name="determining-if-records-exist"></a>
#### レコードの存在を確認

クエリの制約に一致するレコードが存在するかどうかを確認するために`count`メソッドを使用する代わりに、`exists`および`doesntExist`メソッドを使用できます。

    if (DB::table('orders')->where('finalized', 1)->exists()) {
        // ...
    }

    if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
        // ...
    }

<a name="select-statements"></a>
## SELECT文

<a name="specifying-a-select-clause"></a>
#### SELECT句の指定

常にデータベーステーブルからすべてのカラムを選択したいわけではありません。`select`メソッドを使用して、クエリのカスタム"select"句を指定できます。

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')
                ->select('name', 'email as user_email')
                ->get();

`distinct`メソッドを使用すると、クエリが重複しない結果を返すように強制できます。

    $users = DB::table('users')->distinct()->get();

既にクエリビルダインスタンスがあり、その既存のselect句にカラムを追加したい場合は、`addSelect`メソッドを使用できます。

    $query = DB::table('users')->select('name');

    $users = $query->addSelect('age')->get();

<a name="raw-expressions"></a>
## 生のSQL式

時には、クエリに任意の文字列を挿入する必要があるかもしれません。生の文字列式を作成するには、`DB`ファサードが提供する`raw`メソッドを使用できます。

    $users = DB::table('users')
                 ->select(DB::raw('count(*) as user_count, status'))
                 ->where('status', '<>', 1)
                 ->groupBy('status')
                 ->get();

> WARNING:  
> 生のSQL文は文字列としてクエリに挿入されるため、SQLインジェクションの脆弱性を作成しないように十分注意する必要があります。

<a name="raw-methods"></a>
### 生のメソッド

`DB::raw`メソッドを使用する代わりに、以下のメソッドを使用して、クエリのさまざまな部分に生の式を挿入することもできます。**Laravelは、生の式を使用するクエリがSQLインジェクションの脆弱性から保護されていることを保証できないことに注意してください。**

<a name="selectraw"></a>
#### `selectRaw`

`selectRaw`メソッドは、`addSelect(DB::raw(/* ... */))`の代わりに使用できます。このメソッドは、オプションのバインディングの配列を第二引数として受け取ります。

    $orders = DB::table('orders')
                    ->selectRaw('price * ? as price_with_tax', [1.0825])
                    ->get();

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

`whereRaw`と`orWhereRaw`メソッドは、生の"where"句をクエリに挿入するために使用できます。これらのメソッドは、オプションのバインディングの配列を第二引数として受け取ります。

    $orders = DB::table('orders')
                    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                    ->get();

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

`havingRaw`と`orHavingRaw`メソッドは、"having"句の値として生の文字列を提供するために使用できます。これらのメソッドは、オプションのバインディングの配列を第二引数として受け取ります。

    $orders = DB::table('orders')
                    ->select('department', DB::raw('SUM(price) as total_sales'))
                    ->groupBy('department')
                    ->havingRaw('SUM(price) > ?', [2500])
                    ->get();

<a name="orderbyraw"></a>
#### `orderByRaw`

`orderByRaw`メソッドは、"order by"句の値として生の文字列を提供するために使用できます。

    $orders = DB::table('orders')
                    ->orderByRaw('updated_at - created_at DESC')
                    ->get();

<a name="groupbyraw"></a>
### `groupByRaw`

`groupByRaw`メソッドは、`group by`句の値として生の文字列を提供するために使用できます。

    $orders = DB::table('orders')
                    ->select('city', 'state')
                    ->groupByRaw('city, state')
                    ->get();

<a name="joins"></a>
## 結合

<a name="inner-join-clause"></a>
#### 内部結合句

クエリビルダは、クエリに結合句を追加するためにも使用できます。基本的な"内部結合"を実行するには、クエリビルダインスタンスの`join`メソッドを使用できます。`join`メソッドに渡される最初の引数は、結合するテーブルの名前で、残りの引数は結合のカラム制約を指定します。1つのクエリで複数のテーブルを結合することもできます。

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')
                ->join('contacts', 'users.id', '=', 'contacts.user_id')
                ->join('orders', 'users.id', '=', 'orders.user_id')
                ->select('users.*', 'contacts.phone', 'orders.price')
                ->get();

<a name="left-join-right-join-clause"></a>
#### 左結合 / 右結合句

"内部結合"の代わりに"左結合"または"右結合"を実行したい場合は、`leftJoin`または`rightJoin`メソッドを使用します。これらのメソッドは、`join`メソッドと同じシグネチャを持ちます。

    $users = DB::table('users')
                ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

    $users = DB::table('users')
                ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
                ->get();

<a name="cross-join-clause"></a>
#### クロス結合句

`crossJoin`メソッドを使用して"クロス結合"を実行できます。クロス結合は、最初のテーブルと結合されたテーブルの間でデカルト積を生成します。

    $sizes = DB::table('sizes')
                ->crossJoin('colors')
                ->get();

<a name="advanced-join-clauses"></a>
#### 高度な結合句

より高度な結合句を指定することもできます。まず、`join`メソッドの第二引数としてクロージャを渡します。クロージャは、"結合"句に制約を指定できる`Illuminate\Database\Query\JoinClause`インスタンスを受け取ります。

    DB::table('users')
            ->join('contacts', function (JoinClause $join) {
                $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
            })
            ->get();

結合に"where"句を使用したい場合は、`JoinClause`インスタンスが提供する`where`と`orWhere`メソッドを使用できます。これらのメソッドは、2つのカラムを比較する代わりに、カラムを値と比較します。

    DB::table('users')
            ->join('contacts', function (JoinClause $join) {
                $join->on('users.id', '=', 'contacts.user_id')
                     ->where('contacts.user_id', '>', 5);
            })
            ->get();

<a name="subquery-joins"></a>
#### サブクエリ結合

`joinSub`、`leftJoinSub`、`rightJoinSub`メソッドを使用して、クエリをサブクエリに結合できます。これらのメソッドはそれぞれ3つの引数を受け取ります：サブクエリ、そのテーブルエイリアス、および関連するカラムを定義するクロージャ。この例では、各ユーザーのレコードに、ユーザーの最も最近公開されたブログ投稿の`created_at`タイムスタンプを含むユーザーのコレクションを取得します。

    $latestPosts = DB::table('posts')
                       ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                       ->where('is_published', true)
                       ->groupBy('user_id');

    $users = DB::table('users')
            ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
                $join->on('users.id', '=', 'latest_posts.user_id');
            })->get();

<a name="lateral-joins"></a>
#### ラテラル結合

> WARNING:  
> ラテラル結合は現在、PostgreSQL、MySQL >= 8.0.14、およびSQL Serverでサポートされています。

`joinLateral`と`leftJoinLateral`メソッドを使用して、サブクエリとの"ラテラル結合"を実行できます。これらのメソッドはそれぞれ2つの引数を受け取ります：サブクエリとそのテーブルエイリアス。結合条件は、指定されたサブクエリの`where`句内で指定する必要があります。ラテラル結合は各行に対して評価され、サブクエリ外のカラムを参照できます。

この例では、ユーザーのコレクションと、ユーザーの3つの最新のブログ投稿を取得します。各ユーザーは、結果セットで最大3行を生成できます：それぞれの最新のブログ投稿に対して1行。結合条件は、サブクエリ内の`whereColumn`句で指定され、現在のユーザー行を参照します。

    $latestPosts = DB::table('posts')
                       ->select('id as post_id', 'title as post_title', 'created_at as post_created_at')
                       ->whereColumn('user_id', 'users.id')
                       ->orderBy('created_at', 'desc')
                       ->limit(3);

    $users = DB::table('users')
                ->joinLateral($latestPosts, 'latest_posts')
                ->get();

<a name="unions"></a>
## ユニオン

クエリビルダは、2つ以上のクエリを"ユニオン"する便利なメソッドも提供します。例えば、最初のクエリを作成し、`union`メソッドを使用してそれをさらにクエリと結合できます。

    use Illuminate\Support\Facades\DB;

    $first = DB::table('users')
                ->whereNull('first_name');

    $users = DB::table('users')
                ->whereNull('last_name')
                ->union($first)
                ->get();

`union`メソッドに加えて、クエリビルダは`unionAll`メソッドも提供します。`unionAll`メソッドを使用して結合されたクエリは、重複する結果を削除しません。`unionAll`メソッドは、`union`メソッドと同じメソッドシグネチャを持ちます。

<a name="basic-where-clauses"></a>
## 基本的なWhere句

<a name="where-clauses"></a>
### Where句

クエリビルダの`where`メソッドを使用して、クエリに"where"句を追加できます。`where`メソッドの最も基本的な呼び出しは、3つの引数を必要とします。最初の引数はカラムの名前です。2番目の引数は演算子で、データベースがサポートする任意の演算子を指定できます。3番目の引数はカラムの値と比較する値です。

例えば、次のクエリは、`votes`カラムの値が`100`で、`age`カラムの値が`35`より大きいユーザーを取得します。

    $users = DB::table('users')
                    ->where('votes', '=', 100)
                    ->where('age', '>', 35)
                    ->get();

便宜上、カラムが指定された値と`=`であることを確認したい場合、値を`where`メソッドの第二引数として渡すことができます。Laravelは、`=`演算子を使用したいと想定します。

    $users = DB::table('users')->where('votes', 100)->get();

前述のように、データベースシステムがサポートする任意の演算子を使用できます。

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

```php
$users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();
```

`where`関数に条件の配列を渡すこともできます。配列の各要素は、通常`where`メソッドに渡される3つの引数を含む配列であるべきです：

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

> WARNING:  
> PDOはカラム名のバインディングをサポートしていません。したがって、クエリで参照されるカラム名をユーザー入力によって決定させることは絶対に避けてください。"order by"カラムを含む。

<a name="or-where-clauses"></a>
### Or Where Clauses

クエリビルダの`where`メソッドを連鎖させると、"where"句は`and`演算子を使って結合されます。しかし、`orWhere`メソッドを使って`or`演算子を使って句をクエリに結合することができます。`orWhere`メソッドは`where`メソッドと同じ引数を受け付けます：

```php
$users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();
```

もし、括弧内に"or"条件をグループ化する必要がある場合、`orWhere`メソッドの最初の引数としてクロージャを渡すことができます：

```php
$users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere(function (Builder $query) {
                    $query->where('name', 'Abigail')
                          ->where('votes', '>', 50);
                })
                ->get();
```

上記の例は、以下のSQLを生成します：

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> WARNING:  
> グローバルスコープが適用されたときに予期せぬ動作を避けるために、`orWhere`呼び出しを常にグループ化する必要があります。

<a name="where-not-clauses"></a>
### Where Not Clauses

`whereNot`と`orWhereNot`メソッドは、指定されたクエリ制約を否定するために使用できます。例えば、次のクエリは、クリアランス中の商品や価格が10未満の商品を除外します：

```php
$products = DB::table('products')
                    ->whereNot(function (Builder $query) {
                        $query->where('clearance', true)
                              ->orWhere('price', '<', 10);
                    })
                    ->get();
```

<a name="where-any-all-none-clauses"></a>
### Where Any / All / None Clauses

時には、複数のカラムに同じクエリ制約を適用する必要があるかもしれません。例えば、与えられたリストのいずれかのカラムが与えられた値に`LIKE`するすべてのレコードを取得したい場合があります。これは`whereAny`メソッドを使って実現できます：

```php
$users = DB::table('users')
                ->where('active', true)
                ->whereAny([
                    'name',
                    'email',
                    'phone',
                ], 'like', 'Example%')
                ->get();
```

上記のクエリは、以下のSQLを生成します：

```sql
SELECT *
FROM users
WHERE active = true AND (
    name LIKE 'Example%' OR
    email LIKE 'Example%' OR
    phone LIKE 'Example%'
)
```

同様に、`whereAll`メソッドは、与えられたすべてのカラムが与えられた制約に一致するレコードを取得するために使用できます：

```php
$posts = DB::table('posts')
                ->where('published', true)
                ->whereAll([
                    'title',
                    'content',
                ], 'like', '%Laravel%')
                ->get();
```

上記のクエリは、以下のSQLを生成します：

```sql
SELECT *
FROM posts
WHERE published = true AND (
    title LIKE '%Laravel%' AND
    content LIKE '%Laravel%'
)
```

`whereNone`メソッドは、与えられたカラムのいずれも与えられた制約に一致しないレコードを取得するために使用できます：

```php
$posts = DB::table('albums')
                ->where('published', true)
                ->whereNone([
                    'title',
                    'lyrics',
                    'tags',
                ], 'like', '%explicit%')
                ->get();
```

上記のクエリは、以下のSQLを生成します：

```sql
SELECT *
FROM albums
WHERE published = true AND NOT (
    title LIKE '%explicit%' OR
    lyrics LIKE '%explicit%' OR
    tags LIKE '%explicit%'
)
```

<a name="json-where-clauses"></a>
### JSON Where Clauses

Laravelは、JSONカラム型をサポートするデータベースでJSONカラム型のクエリもサポートしています。現在、これにはMariaDB 10.3+、MySQL 8.0+、PostgreSQL 12.0+、SQL Server 2017+、およびSQLite 3.39.0+が含まれます。JSONカラムをクエリするには、`->`演算子を使用します：

```php
$users = DB::table('users')
                    ->where('preferences->dining->meal', 'salad')
                    ->get();
```

`whereJsonContains`を使用してJSON配列をクエリすることができます：

```php
$users = DB::table('users')
                    ->whereJsonContains('options->languages', 'en')
                    ->get();
```

アプリケーションがMariaDB、MySQL、またはPostgreSQLデータベースを使用している場合、`whereJsonContains`メソッドに値の配列を渡すことができます：

```php
$users = DB::table('users')
                    ->whereJsonContains('options->languages', ['en', 'de'])
                    ->get();
```

`whereJsonLength`メソッドを使用して、長さによってJSON配列をクエリすることができます：

```php
$users = DB::table('users')
                    ->whereJsonLength('options->languages', 0)
                    ->get();

$users = DB::table('users')
                    ->whereJsonLength('options->languages', '>', 1)
                    ->get();
```

<a name="additional-where-clauses"></a>
### Additional Where Clauses

**whereLike / orWhereLike / whereNotLike / orWhereNotLike**

`whereLike`メソッドを使用すると、パターンマッチングのための"LIKE"句をクエリに追加できます。これらのメソッドは、大文字と小文字を区別するかどうかを切り替える機能を持つ、データベースに依存しない文字列マッチングクエリを実行する方法を提供します。デフォルトでは、文字列マッチングは大文字と小文字を区別しません：

```php
$users = DB::table('users')
               ->whereLike('name', '%John%')
               ->get();
```

`caseSensitive`引数を介して大文字と小文字を区別する検索を有効にできます：

```php
$users = DB::table('users')
               ->whereLike('name', '%John%', caseSensitive: true)
               ->get();
```

`orWhereLike`メソッドを使用すると、LIKE条件を持つ"or"句を追加できます：

```php
$users = DB::table('users')
               ->where('votes', '>', 100)
               ->orWhereLike('name', '%John%')
               ->get();
```

`whereNotLike`メソッドを使用すると、クエリに"NOT LIKE"句を追加できます：

```php
$users = DB::table('users')
               ->whereNotLike('name', '%John%')
               ->get();
```

同様に、`orWhereNotLike`を使用してNOT LIKE条件を持つ"or"句を追加できます：

```php
$users = DB::table('users')
               ->where('votes', '>', 100)
               ->orWhereNotLike('name', '%John%')
               ->get();
```

> WARNING:
> `whereLike`の大文字と小文字を区別する検索オプションは、現在SQL Serverではサポートされていません。

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

`whereIn`メソッドは、指定されたカラムの値が指定された配列内に含まれていることを検証します：

```php
$users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();
```

`whereNotIn`メソッドは、指定されたカラムの値が指定された配列内に含まれていないことを検証します：

```php
$users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();
```

`whereIn`メソッドの2番目の引数としてクエリオブジェクトを提供することもできます：

```php
$activeUsers = DB::table('users')->select('id')->where('is_active', 1);

$users = DB::table('comments')
                        ->whereIn('user_id', $activeUsers)
                        ->get();
```

上記の例は、以下のSQLを生成します：

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> WARNING:  
> 大量の整数バインディングをクエリに追加する場合、`whereIntegerInRaw`または`whereIntegerNotInRaw`メソッドを使用してメモリ使用量を大幅に削減できます。

**whereBetween / orWhereBetween**

`whereBetween`メソッドは、カラムの値が2つの値の間にあることを検証します：

```php
$users = DB::table('users')
               ->whereBetween('votes', [1, 100])
               ->get();
```

**whereNotBetween / orWhereNotBetween**

`whereNotBetween`メソッドは、カラムの値が2つの値の外側にあることを検証します：

```php
$users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();
```

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

`whereBetweenColumns`メソッドは、カラムの値が同じテーブル行の2つのカラムの2つの値の間にあることを検証します：

```php
$patients = DB::table('patients')
                           ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
                           ->get();
```

`whereNotBetweenColumns`メソッドは、カラムの値が同じテーブル行の2つのカラムの2つの値の外側にあることを検証します：

```php
$patients = DB::table('patients')
                           ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
                           ->get();
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

`whereNull`メソッドは、指定されたカラムの値が`NULL`であることを検証します：

```php
$users = DB::table('users')
                    ->whereNull('updated_at')
                    ->get();
```

`whereNotNull`メソッドは、カラムの値が`NULL`でないことを検証します：

```php
$users = DB::table('users')
                    ->whereNotNull('updated_at')
                    ->get();
```

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate`メソッドは、カラムの値を日付と比較するために使用できます：

```php
$users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();
```

`whereMonth`メソッドは、カラムの値を特定の月と比較するために使用できます。

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay`メソッドは、カラムの値を月の特定の日と比較するために使用できます。

    $users = DB::table('users')
                    ->whereDay('created_at', '31')
                    ->get();

`whereYear`メソッドは、カラムの値を特定の年と比較するために使用できます。

    $users = DB::table('users')
                    ->whereYear('created_at', '2016')
                    ->get();

`whereTime`メソッドは、カラムの値を特定の時間と比較するために使用できます。

    $users = DB::table('users')
                    ->whereTime('created_at', '=', '11:20:45')
                    ->get();

**whereColumn / orWhereColumn**

`whereColumn`メソッドは、2つのカラムが等しいかどうかを検証するために使用できます。

    $users = DB::table('users')
                    ->whereColumn('first_name', 'last_name')
                    ->get();

比較演算子を`whereColumn`メソッドに渡すこともできます。

    $users = DB::table('users')
                    ->whereColumn('updated_at', '>', 'created_at')
                    ->get();

カラム比較の配列を`whereColumn`メソッドに渡すこともできます。これらの条件は`and`演算子を使用して結合されます。

    $users = DB::table('users')
                    ->whereColumn([
                        ['first_name', '=', 'last_name'],
                        ['updated_at', '>', 'created_at'],
                    ])->get();

<a name="logical-grouping"></a>
### 論理グループ化

場合によっては、クエリの望ましい論理グループ化を実現するために、複数の「where」句を括弧内にグループ化する必要があります。実際、`orWhere`メソッドの呼び出しを括弧内にグループ化して、予期しないクエリ動作を避けるべきです。これを行うには、クロージャを`where`メソッドに渡すことができます。

    $users = DB::table('users')
               ->where('name', '=', 'John')
               ->where(function (Builder $query) {
                   $query->where('votes', '>', 100)
                         ->orWhere('title', '=', 'Admin');
               })
               ->get();

`where`メソッドにクロージャを渡すと、クエリビルダーに制約グループの開始を指示します。クロージャは、括弧グループ内に含めるべき制約を設定するために使用できるクエリビルダーインスタンスを受け取ります。上記の例は、次のSQLを生成します。

```sql
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> WARNING:  
> グローバルスコープが適用されたときに予期しない動作を避けるために、`orWhere`の呼び出しを常にグループ化する必要があります。

<a name="advanced-where-clauses"></a>
### 高度なWhere句

<a name="where-exists-clauses"></a>
### Where Exists句

`whereExists`メソッドを使用すると、「where exists」SQL句を記述できます。`whereExists`メソッドはクロージャを受け取り、そのクロージャはクエリビルダーインスタンスを受け取ります。このクロージャ内で、「exists」句内に配置するクエリを定義できます。

    $users = DB::table('users')
               ->whereExists(function (Builder $query) {
                   $query->select(DB::raw(1))
                         ->from('orders')
                         ->whereColumn('orders.user_id', 'users.id');
               })
               ->get();

または、クロージャの代わりにクエリオブジェクトを`whereExists`メソッドに渡すこともできます。

    $orders = DB::table('orders')
                    ->select(DB::raw(1))
                    ->whereColumn('orders.user_id', 'users.id');

    $users = DB::table('users')
                        ->whereExists($orders)
                        ->get();

上記の両方の例は、次のSQLを生成します。

```sql
select * from users
where exists (
    select 1
    from orders
    where orders.user_id = users.id
)
```

<a name="subquery-where-clauses"></a>
### サブクエリWhere句

場合によっては、サブクエリの結果を特定の値と比較する「where」句を構築する必要があります。これを行うには、`where`メソッドにクロージャと値を渡すことができます。たとえば、次のクエリは、特定のタイプの最近の「メンバーシップ」を持つすべてのユーザーを取得します。

    use App\Models\User;
    use Illuminate\Database\Query\Builder;

    $users = User::where(function (Builder $query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('membership.user_id', 'users.id')
            ->orderByDesc('membership.start_date')
            ->limit(1);
    }, 'Pro')->get();

または、カラムをサブクエリの結果と比較する「where」句を構築する必要がある場合があります。これを行うには、カラム、演算子、およびクロージャを`where`メソッドに渡すことができます。たとえば、次のクエリは、金額が平均より少ないすべての収入レコードを取得します。

    use App\Models\Income;
    use Illuminate\Database\Query\Builder;

    $incomes = Income::where('amount', '<', function (Builder $query) {
        $query->selectRaw('avg(i.amount)')->from('incomes as i');
    })->get();

<a name="full-text-where-clauses"></a>
### フルテキストWhere句

> WARNING:  
> フルテキストwhere句は、現在MariaDB、MySQL、およびPostgreSQLでサポートされています。

`whereFullText`および`orWhereFullText`メソッドは、[フルテキストインデックス](migrations.md#available-index-types)を持つカラムに対してフルテキスト「where」句をクエリに追加するために使用できます。これらのメソッドは、Laravelによって基盤となるデータベースシステムに適したSQLに変換されます。たとえば、MariaDBまたはMySQLを使用するアプリケーションの場合、`MATCH AGAINST`句が生成されます。

    $users = DB::table('users')
               ->whereFullText('bio', 'web developer')
               ->get();

<a name="ordering-grouping-limit-and-offset"></a>
## 並べ替え、グループ化、制限、オフセット

<a name="ordering"></a>
### 並べ替え

<a name="orderby"></a>
#### `orderBy`メソッド

`orderBy`メソッドを使用すると、指定したカラムでクエリの結果を並べ替えることができます。`orderBy`メソッドの最初の引数は並べ替えたいカラムで、2番目の引数は並べ替えの方向を決定し、`asc`または`desc`のいずれかになります。

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

複数のカラムで並べ替えるには、必要なだけ`orderBy`を呼び出すことができます。

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->orderBy('email', 'asc')
                    ->get();

<a name="latest-oldest"></a>
#### `latest`および`oldest`メソッド

`latest`および`oldest`メソッドを使用すると、日付で結果を簡単に並べ替えることができます。デフォルトでは、結果はテーブルの`created_at`カラムで並べ替えられます。あるいは、並べ替えたいカラム名を指定することもできます。

    $user = DB::table('users')
                    ->latest()
                    ->first();

<a name="random-ordering"></a>
#### ランダムな並べ替え

`inRandomOrder`メソッドを使用して、クエリ結果をランダムに並べ替えることができます。たとえば、このメソッドを使用してランダムなユーザーをフェッチできます。

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

<a name="removing-existing-orderings"></a>
#### 既存の並べ替えの削除

`reorder`メソッドは、クエリに以前に適用されたすべての「order by」句を削除します。

    $query = DB::table('users')->orderBy('name');

    $unorderedUsers = $query->reorder()->get();

`reorder`メソッドを呼び出す際にカラムと方向を渡すことで、すべての既存の「order by」句を削除し、クエリにまったく新しい順序を適用することができます。

    $query = DB::table('users')->orderBy('name');

    $usersOrderedByEmail = $query->reorder('email', 'desc')->get();

<a name="grouping"></a>
### グループ化

<a name="groupby-having"></a>
#### `groupBy`および`having`メソッド

予想通り、`groupBy`および`having`メソッドを使用してクエリ結果をグループ化できます。`having`メソッドの署名は`where`メソッドの署名と似ています。

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

`havingBetween`メソッドを使用して、指定された範囲内の結果をフィルタリングできます。

    $report = DB::table('orders')
                    ->selectRaw('count(id) as number_of_orders, customer_id')
                    ->groupBy('customer_id')
                    ->havingBetween('number_of_orders', [5, 15])
                    ->get();

複数の引数を`groupBy`メソッドに渡して、複数のカラムでグループ化することもできます。

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

より高度な`having`文を構築するには、[`havingRaw`](#raw-methods)メソッドを参照してください。

<a name="limit-and-offset"></a>
### 制限とオフセット

<a name="skip-take"></a>
#### `skip`および`take`メソッド

`skip`および`take`メソッドを使用して、クエリから返される結果の数を制限したり、クエリ内の指定された数の結果をスキップしたりできます。

    $users = DB::table('users')->skip(10)->take(5)->get();

または、`limit`および`offset`メソッドを使用することもできます。これらのメソッドは、それぞれ`take`および`skip`メソッドと機能的に同等です。

    $users = DB::table('users')
                    ->offset(10)
                    ->limit(5)
                    ->get();

<a name="conditional-clauses"></a>
## 条件付き句

場合によっては、特定の条件に基づいてクエリに特定のクエリ句を適用したいことがあります。例えば、特定の入力値が受信HTTPリクエストに存在する場合にのみ `where` 文を適用したい場合があります。これは `when` メソッドを使用して実現できます:

    $role = $request->input('role');

    $users = DB::table('users')
                    ->when($role, function (Builder $query, string $role) {
                        $query->where('role_id', $role);
                    })
                    ->get();

`when` メソッドは、最初の引数が `true` の場合にのみ指定されたクロージャを実行します。最初の引数が `false` の場合、クロージャは実行されません。したがって、上記の例では、`when` メソッドに渡されたクロージャは、`role` フィールドが受信リクエストに存在し、`true` と評価される場合にのみ呼び出されます。

`when` メソッドに3番目の引数として別のクロージャを渡すこともできます。このクロージャは、最初の引数が `false` と評価された場合にのみ実行されます。この機能の使用方法を説明するために、クエリのデフォルトの順序付けを設定するために使用します:

    $sortByVotes = $request->boolean('sort_by_votes');

    $users = DB::table('users')
                    ->when($sortByVotes, function (Builder $query, bool $sortByVotes) {
                        $query->orderBy('votes');
                    }, function (Builder $query) {
                        $query->orderBy('name');
                    })
                    ->get();

<a name="insert-statements"></a>
## 挿入ステートメント

クエリビルダは、データベーステーブルにレコードを挿入するために使用できる `insert` メソッドも提供します。`insert` メソッドは、カラム名と値の配列を受け取ります:

    DB::table('users')->insert([
        'email' => 'kayla@example.com',
        'votes' => 0
    ]);

配列の配列を渡すことで、一度に複数のレコードを挿入できます。各配列は、テーブルに挿入されるレコードを表します:

    DB::table('users')->insert([
        ['email' => 'picard@example.com', 'votes' => 0],
        ['email' => 'janeway@example.com', 'votes' => 0],
    ]);

`insertOrIgnore` メソッドは、データベースにレコードを挿入する際にエラーを無視します。このメソッドを使用する場合、重複レコードエラーが無視され、他のタイプのエラーもデータベースエンジンに応じて無視される可能性があることに注意してください。例えば、`insertOrIgnore` は [MySQLの厳格モードをバイパス](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution) します:

    DB::table('users')->insertOrIgnore([
        ['id' => 1, 'email' => 'sisko@example.com'],
        ['id' => 2, 'email' => 'archer@example.com'],
    ]);

`insertUsing` メソッドは、サブクエリを使用して挿入するデータを決定しながら、新しいレコードをテーブルに挿入します:

    DB::table('pruned_users')->insertUsing([
        'id', 'name', 'email', 'email_verified_at'
    ], DB::table('users')->select(
        'id', 'name', 'email', 'email_verified_at'
    )->where('updated_at', '<=', now()->subMonth()));

<a name="auto-incrementing-ids"></a>
#### 自動増分ID

テーブルに自動増分IDがある場合、`insertGetId` メソッドを使用してレコードを挿入し、IDを取得します:

    $id = DB::table('users')->insertGetId(
        ['email' => 'john@example.com', 'votes' => 0]
    );

> WARNING:  
> PostgreSQLを使用する場合、`insertGetId` メソッドは自動増分カラムが `id` という名前であることを期待します。別の「シーケンス」からIDを取得したい場合は、`insertGetId` メソッドの2番目のパラメータとしてカラム名を渡すことができます。

<a name="upserts"></a>
### Upserts

`upsert` メソッドは、存在しないレコードを挿入し、指定した新しい値で既存のレコードを更新します。このメソッドの最初の引数は挿入または更新する値で、2番目の引数は関連するテーブル内のレコードを一意に識別するカラムのリストです。このメソッドの3番目の引数は、一致するレコードが既にデータベースに存在する場合に更新する必要があるカラムの配列です:

    DB::table('flights')->upsert(
        [
            ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
            ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
        ],
        ['departure', 'destination'],
        ['price']
    );

上記の例では、Laravelは2つのレコードを挿入しようとします。`departure` と `destination` カラムの値が同じレコードが既に存在する場合、Laravelはそのレコードの `price` カラムを更新します。

> WARNING:  
> SQL Serverを除くすべてのデータベースは、`upsert` メソッドの2番目の引数のカラムに「プライマリ」または「ユニーク」インデックスが必要です。さらに、MariaDBおよびMySQLデータベースドライバは、`upsert` メソッドの2番目の引数を無視し、常にテーブルの「プライマリ」および「ユニーク」インデックスを使用して既存のレコードを検出します。

<a name="update-statements"></a>
## 更新ステートメント

データベースにレコードを挿入するだけでなく、クエリビルダは `update` メソッドを使用して既存のレコードを更新することもできます。`update` メソッドは、`insert` メソッドと同様に、更新するカラムと値のペアの配列を受け取ります。`update` メソッドは、影響を受けた行数を返します。`where` 句を使用して `update` クエリを制約することができます:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['votes' => 1]);

<a name="update-or-insert"></a>
#### 更新または挿入

場合によっては、データベース内の既存のレコードを更新するか、一致するレコードが存在しない場合に新しく作成したいことがあります。このシナリオでは、`updateOrInsert` メソッドを使用できます。`updateOrInsert` メソッドは、2つの引数を受け取ります。1つ目はレコードを見つけるための条件の配列、2つ目は更新するカラムと値のペアの配列です。

`updateOrInsert` メソッドは、最初の引数のカラムと値のペアを使用して一致するデータベースレコードを見つけようとします。レコードが存在する場合、2番目の引数の値で更新されます。レコードが見つからない場合、両方の引数の属性をマージした新しいレコードが挿入されます:

    DB::table('users')
        ->updateOrInsert(
            ['email' => 'john@example.com', 'name' => 'John'],
            ['votes' => '2']
        );

`updateOrInsert` メソッドにクロージャを提供して、一致するレコードの有無に基づいてデータベースに更新または挿入される属性をカスタマイズすることができます:

```php
DB::table('users')->updateOrInsert(
    ['user_id' => $user_id],
    fn ($exists) => $exists ? [
        'name' => $data['name'],
        'email' => $data['email'],
    ] : [
        'name' => $data['name'],
        'email' => $data['email'],
        'marketable' => true,
    ],
);
```

<a name="updating-json-columns"></a>
### JSONカラムの更新

JSONカラムを更新する場合、JSONオブジェクト内の適切なキーを更新するために `->` 構文を使用する必要があります。この操作は、MariaDB 10.3+、MySQL 5.7+、およびPostgreSQL 9.5+でサポートされています:

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 増分と減分

クエリビルダは、指定されたカラムの値を増減するための便利なメソッドも提供しています。これらのメソッドは、少なくとも1つの引数（変更するカラム）を受け取ります。カラムを増減する量を指定するために、2番目の引数を提供することもできます:

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

必要に応じて、増分または減分操作中に更新する追加のカラムを指定することもできます:

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

さらに、`incrementEach` および `decrementEach` メソッドを使用して一度に複数のカラムを増減することもできます:

    DB::table('users')->incrementEach([
        'votes' => 5,
        'balance' => 100,
    ]);

<a name="delete-statements"></a>
## 削除ステートメント

クエリビルダの `delete` メソッドは、テーブルからレコードを削除するために使用できます。`delete` メソッドは、影響を受けた行数を返します。`delete` メソッドを呼び出す前に「where」句を追加して `delete` ステートメントを制約することができます:

    $deleted = DB::table('users')->delete();

    $deleted = DB::table('users')->where('votes', '>', 100)->delete();

テーブル全体を切り捨てたい場合（テーブルからすべてのレコードを削除し、自動増分IDをゼロにリセットする）、`truncate` メソッドを使用できます:

    DB::table('users')->truncate();

<a name="table-truncation-and-postgresql"></a>
#### テーブルの切り捨てとPostgreSQL

PostgreSQLデータベースを切り捨てる場合、`CASCADE` 動作が適用されます。これは、他のテーブルのすべての外部キー関連レコードも削除されることを意味します。

<a name="pessimistic-locking"></a>
## 悲観的ロック

クエリビルダには、`select` ステートメントを実行する際に「悲観的ロック」を実現するためのいくつかの機能も含まれています。「共有ロック」でステートメントを実行するには、`sharedLock` メソッドを呼び出すことができます。共有ロックは、選択された行がトランザクションがコミットされるまで変更されないようにします:

    DB::table('users')
            ->where('votes', '>', 100)
            ->sharedLock()
            ->get();

または、`lockForUpdate` メソッドを使用することもできます。「for update」ロックは、選択されたレコードが変更されたり、別の共有ロックで選択されたりするのを防ぎます:

    DB::table('users')
            ->where('votes', '>', 100)
            ->lockForUpdate()
            ->get();

<a name="debugging"></a>
## デバッグ

クエリを構築する際に、現在のクエリのバインディングとSQLをダンプするために `dd` と `dump` メソッドを使用することができます。`dd` メソッドはデバッグ情報を表示した後、リクエストの実行を停止します。`dump` メソッドはデバッグ情報を表示しますが、リクエストの実行を継続させます:

    DB::table('users')->where('votes', '>', 100)->dd();

    DB::table('users')->where('votes', '>', 100)->dump();

`dumpRawSql` と `ddRawSql` メソッドは、クエリのSQLをすべてのパラメータバインディングが適切に置換された状態でダンプするために呼び出すことができます:

    DB::table('users')->where('votes', '>', 100)->dumpRawSql();

    DB::table('users')->where('votes', '>', 100)->ddRawSql();

