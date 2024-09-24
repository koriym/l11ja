# データベース: ページネーション

- [イントロダクション](#introduction)
- [基本的な使用法](#basic-usage)
    - [クエリビルダの結果をページネーションする](#paginating-query-builder-results)
    - [Eloquentの結果をページネーションする](#paginating-eloquent-results)
    - [カーソルページネーション](#cursor-pagination)
    - [ページネータを手動で作成する](#manually-creating-a-paginator)
    - [ページネーションURLのカスタマイズ](#customizing-pagination-urls)
- [ページネーション結果の表示](#displaying-pagination-results)
    - [ページネーションリンクウィンドウの調整](#adjusting-the-pagination-link-window)
    - [結果をJSONに変換する](#converting-results-to-json)
- [ページネーションビューのカスタマイズ](#customizing-the-pagination-view)
    - [Bootstrapの使用](#using-bootstrap)
- [ページネータとLengthAwarePaginatorインスタンスメソッド](#paginator-instance-methods)
- [カーソルページネータインスタンスメソッド](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## イントロダクション

他のフレームワークでは、ページネーションは非常に苦痛なものになることがあります。Laravelのページネーションは、新鮮な息吹を感じてもらえることを願っています。Laravelのページネータは、[クエリビルダ](queries.md)と[Eloquent ORM](eloquent.md)と統合されており、ゼロコンフィグでデータベースレコードのページネーションを簡単に行えます。

デフォルトでは、ページネータによって生成されるHTMLは、[Tailwind CSSフレームワーク](https://tailwindcss.com/)と互換性があります。ただし、Bootstrapのページネーションサポートも利用できます。

<a name="tailwind-jit"></a>
#### Tailwind JIT

LaravelのデフォルトのTailwindページネーションビューを使用していて、Tailwind JITエンジンを使用している場合は、アプリケーションの`tailwind.config.js`ファイルの`content`キーがLaravelのページネーションビューを参照していることを確認してください。これにより、それらのTailwindクラスが削除されないようになります。

```js
content: [
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
    './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
],
```

<a name="basic-usage"></a>
## 基本的な使用法

<a name="paginating-query-builder-results"></a>
### クエリビルダの結果をページネーションする

アイテムをページネーションする方法はいくつかあります。最も簡単な方法は、[クエリビルダ](queries.md)または[Eloquentクエリ](eloquent.md)で`paginate`メソッドを使用することです。`paginate`メソッドは、ユーザーが現在閲覧しているページに基づいて、クエリの「limit」と「offset」を自動的に設定します。デフォルトでは、現在のページはHTTPリクエストの`page`クエリ文字列引数の値によって検出されます。この値はLaravelによって自動的に検出され、ページネータによって生成されるリンクにも自動的に挿入されます。

この例では、`paginate`メソッドに渡される唯一の引数は、「ページごと」に表示したいアイテムの数です。この場合、`15`アイテムをページごとに表示するように指定しましょう。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * すべてのアプリケーションユーザーを表示します。
         */
        public function index(): View
        {
            return view('user.index', [
                'users' => DB::table('users')->paginate(15)
            ]);
        }
    }

<a name="simple-pagination"></a>
#### シンプルページネーション

`paginate`メソッドは、データベースからレコードを取得する前に、クエリに一致するレコードの総数をカウントします。これは、ページネータがレコードの総ページ数を知るために行われます。ただし、アプリケーションのUIに総ページ数を表示する予定がない場合、レコード数のクエリは不要です。

したがって、アプリケーションのUIにシンプルな「次へ」と「前へ」のリンクのみを表示する必要がある場合は、`simplePaginate`メソッドを使用して、単一の効率的なクエリを実行できます。

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### Eloquentの結果をページネーションする

[Eloquent](eloquent.md)クエリをページネーションすることもできます。この例では、`App\Models\User`モデルをページネーションし、15レコードをページごとに表示することを示します。クエリビルダの結果をページネーションする場合とほぼ同じ構文であることがわかります。

    use App\Models\User;

    $users = User::paginate(15);

もちろん、クエリに他の制約（`where`句など）を設定した後に`paginate`メソッドを呼び出すこともできます。

    $users = User::where('votes', '>', 100)->paginate(15);

Eloquentモデルをページネーションする場合にも、`simplePaginate`メソッドを使用できます。

    $users = User::where('votes', '>', 100)->simplePaginate(15);

同様に、Eloquentモデルをカーソルページネーションするために`cursorPaginate`メソッドを使用できます。

    $users = User::where('votes', '>', 100)->cursorPaginate(15);

<a name="multiple-paginator-instances-per-page"></a>
#### 1ページに複数のページネータインスタンス

アプリケーションによってレンダリングされる単一の画面に2つの独立したページネータをレンダリングする必要がある場合があります。ただし、両方のページネータインスタンスが現在のページを格納するために`page`クエリ文字列パラメータを使用している場合、2つのページネータは競合します。この競合を解決するには、`paginate`、`simplePaginate`、および`cursorPaginate`メソッドに提供される3番目の引数に、ページネータの現在のページを格納するために使用するクエリ文字列パラメータの名前を渡します。

    use App\Models\User;

    $users = User::where('votes', '>', 100)->paginate(
        $perPage = 15, $columns = ['*'], $pageName = 'users'
    );

<a name="cursor-pagination"></a>
### カーソルページネーション

`paginate`と`simplePaginate`はSQLの「offset」句を使用してクエリを作成しますが、カーソルページネーションはクエリに含まれる順序付けられた列の値を比較する「where」句を構築することで機能し、Laravelのすべてのページネーション方法の中で最も優れたデータベースパフォーマンスを提供します。このページネーション方法は、特に大規模なデータセットと「無限」スクロールユーザーインターフェースに適しています。

オフセットベースのページネーションとは異なり、カーソルベースのページネーションはページネータによって生成されるURLのクエリ文字列にページ番号を含めません。代わりに、クエリ文字列に「カーソル」文字列を配置します。カーソルは、次のページネーションクエリがページネーションを開始する場所と方向を含むエンコードされた文字列です。

```nothing
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

クエリビルダによって提供される`cursorPaginate`メソッドを介して、カーソルベースのページネータインスタンスを作成できます。このメソッドは、`Illuminate\Pagination\CursorPaginator`のインスタンスを返します。

    $users = DB::table('users')->orderBy('id')->cursorPaginate(15);

カーソルページネータインスタンスを取得したら、`paginate`および`simplePaginate`メソッドを使用する場合と同様に、[ページネーション結果を表示](#displaying-pagination-results)できます。カーソルページネータによって提供されるインスタンスメソッドの詳細については、[カーソルページネータインスタンスメソッドのドキュメント](#cursor-paginator-instance-methods)を参照してください。

> WARNING:  
> カーソルページネーションを利用するには、クエリに「order by」句が含まれている必要があります。さらに、クエリが順序付けられる列は、ページネーションするテーブルに属している必要があります。

<a name="cursor-vs-offset-pagination"></a>
#### カーソル vs. オフセットページネーション

オフセットページネーションとカーソルページネーションの違いを説明するために、いくつかの例のSQLクエリを見てみましょう。以下の両方のクエリは、`id`で順序付けられた`users`テーブルの「2ページ目」の結果を表示します。

```sql
# オフセットページネーション...
select * from users order by id asc limit 15 offset 15;

# カーソルページネーション...
select * from users where id > 15 order by id asc limit 15;
```

カーソルページネーションクエリは、オフセットページネーションに比べて以下の利点があります。

- 大規模なデータセットの場合、カーソルページネーションは「order by」列がインデックス付けされている場合により優れたパフォーマンスを提供します。これは、「offset」句が以前に一致したすべてのデータをスキャンするためです。
- 頻繁に書き込まれるデータセットの場合、オフセットページネーションは、結果が最近ユーザーが現在閲覧しているページに追加または削除された場合、レコードをスキップしたり重複したりする可能性があります。

ただし、カーソルページネーションには以下の制限があります。

- `simplePaginate`と同様に、カーソルページネーションは「次へ」と「前へ」のリンクを表示するためにのみ使用でき、ページ番号を持つリンクを生成することはできません。
- 順序付けが少なくとも1つの一意の列または一意の列の組み合わせに基づいている必要があります。`null`値を持つ列はサポートされていません。
- 「order by」句のクエリ式は、エイリアス化されて「select」句にも追加されている場合にのみサポートされます。
- パラメータを持つクエリ式はサポートされていません。

<a name="manually-creating-a-paginator"></a>
### ページネータを手動で作成する

メモリ内に既にあるアイテムの配列を使用して、ページネーションインスタンスを手動で作成したい場合があります。これを行うには、`Illuminate\Pagination\Paginator`、`Illuminate\Pagination\LengthAwarePaginator`、または`Illuminate\Pagination\CursorPaginator`インスタンスを作成します。これは、ニーズに応じて異なります。

`Paginator`と`CursorPaginator`クラスは、結果セット内のアイテムの総数を知る必要がありません。ただし、このため、これらのクラスには最後のページのインデックスを取得するメソッドがありません。`LengthAwarePaginator`は、`Paginator`とほぼ同じ引数を受け入れますが、結果セット内のアイテムの総数のカウントが必要です。

言い換えれば、`Paginator`はクエリビルダの`simplePaginate`メソッドに対応し、`CursorPaginator`は`cursorPaginate`メソッドに対応し、`LengthAwarePaginator`は`paginate`メソッドに対応します。

> WARNING:  
> ページネーターインスタンスを手動で作成する場合、ページネーターに渡す結果の配列を手動で「スライス」する必要があります。これを行う方法がわからない場合は、[array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP関数を確認してください。

<a name="customizing-pagination-urls"></a>
### ページネーションURLのカスタマイズ

デフォルトでは、ページネーターによって生成されるリンクは現在のリクエストのURIに一致します。しかし、ページネーターの`withPath`メソッドを使用すると、ページネーターがリンクを生成する際に使用するURIをカスタマイズできます。例えば、ページネーターが`http://example.com/admin/users?page=N`のようなリンクを生成するようにしたい場合、`/admin/users`を`withPath`メソッドに渡す必要があります:

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->withPath('/admin/users');

        // ...
    });

<a name="appending-query-string-values"></a>
#### クエリ文字列値の追加

`appends`メソッドを使用して、ページネーションリンクのクエリ文字列に追加することができます。例えば、各ページネーションリンクに`sort=votes`を追加するには、次のように`appends`を呼び出す必要があります:

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->appends(['sort' => 'votes']);

        // ...
    });

現在のリクエストのすべてのクエリ文字列値をページネーションリンクに追加したい場合は、`withQueryString`メソッドを使用できます:

    $users = User::paginate(15)->withQueryString();

<a name="appending-hash-fragments"></a>
#### ハッシュフラグメントの追加

ページネーターによって生成されるURLに「ハッシュフラグメント」を追加する必要がある場合は、`fragment`メソッドを使用できます。例えば、各ページネーションリンクの末尾に`#users`を追加するには、次のように`fragment`メソッドを呼び出す必要があります:

    $users = User::paginate(15)->fragment('users');

<a name="displaying-pagination-results"></a>
## ページネーション結果の表示

`paginate`メソッドを呼び出すと、`Illuminate\Pagination\LengthAwarePaginator`のインスタンスが返され、`simplePaginate`メソッドを呼び出すと、`Illuminate\Pagination\Paginator`のインスタンスが返されます。また、`cursorPaginate`メソッドを呼び出すと、`Illuminate\Pagination\CursorPaginator`のインスタンスが返されます。

これらのオブジェクトは、結果セットを説明するいくつかのメソッドを提供します。これらのヘルパーメソッドに加えて、ページネーターインスタンスはイテレータであり、配列としてループできます。したがって、結果を取得したら、[Blade](blade.md)を使用して結果を表示し、ページリンクをレンダリングできます:

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

`links`メソッドは、結果セットの残りのページへのリンクをレンダリングします。これらの各リンクには、適切な`page`クエリ文字列変数が既に含まれています。`links`メソッドによって生成されるHTMLは、[Tailwind CSSフレームワーク](https://tailwindcss.com)と互換性があることを覚えておいてください。

<a name="adjusting-the-pagination-link-window"></a>
### ページネーションリンクウィンドウの調整

ページネーターがページネーションリンクを表示するとき、現在のページ番号と、現在のページの前後3ページのリンクが表示されます。`onEachSide`メソッドを使用すると、ページネーターが生成する中央のスライディングウィンドウ内のリンクの両側に表示される追加リンクの数を制御できます:

```blade
{{ $users->onEachSide(5)->links() }}
```

<a name="converting-results-to-json"></a>
### 結果をJSONに変換

Laravelのページネータークラスは、`Illuminate\Contracts\Support\Jsonable`インターフェース契約を実装し、`toJson`メソッドを公開しているため、ページネーション結果をJSONに変換するのは非常に簡単です。ルートまたはコントローラーアクションから返すことで、ページネーターインスタンスをJSONに変換することもできます:

    use App\Models\User;

    Route::get('/users', function () {
        return User::paginate();
    });

ページネーターからのJSONには、`total`、`current_page`、`last_page`などのメタ情報が含まれます。結果レコードはJSON配列の`data`キーを介して利用できます。ルートからページネーターインスタンスを返すことで作成されるJSONの例を次に示します:

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // Record...
            },
            {
                // Record...
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## ページネーションビューのカスタマイズ

デフォルトでは、ページネーションリンクを表示するためにレンダリングされるビューは、[Tailwind CSS](https://tailwindcss.com)フレームワークと互換性があります。ただし、Tailwindを使用していない場合は、これらのリンクをレンダリングするために独自のビューを自由に定義できます。ページネーターインスタンスで`links`メソッドを呼び出すときに、ビュー名をメソッドの最初の引数として渡すことができます:

```blade
{{ $paginator->links('view.name') }}

<!-- ビューに追加データを渡す... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

ただし、ページネーションビューをカスタマイズする最も簡単な方法は、`vendor:publish`コマンドを使用して`resources/views/vendor`ディレクトリにエクスポートすることです:

```shell
php artisan vendor:publish --tag=laravel-pagination
```

このコマンドは、ビューをアプリケーションの`resources/views/vendor/pagination`ディレクトリに配置します。このディレクトリ内の`tailwind.blade.php`ファイルは、デフォルトのページネーションビューに対応します。このファイルを編集して、ページネーションHTMLを変更できます。

別のファイルをデフォルトのページネーションビューとして指定したい場合は、`App\Providers\AppServiceProvider`クラスの`boot`メソッド内でページネーターの`defaultView`および`defaultSimpleView`メソッドを呼び出すことができます:

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Paginator::defaultView('view-name');

            Paginator::defaultSimpleView('view-name');
        }
    }

<a name="using-bootstrap"></a>
### Bootstrapの使用

Laravelには、[Bootstrap CSS](https://getbootstrap.com/)を使用して構築されたページネーションビューも含まれています。これらのビューをデフォルトのTailwindビューの代わりに使用するには、`App\Providers\AppServiceProvider`クラスの`boot`メソッド内でページネーターの`useBootstrapFour`または`useBootstrapFive`メソッドを呼び出すことができます:

    use Illuminate\Pagination\Paginator;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Paginator::useBootstrapFive();
        Paginator::useBootstrapFour();
    }

<a name="paginator-instance-methods"></a>
## ページネーター / LengthAwarePaginatorインスタンスメソッド

各ページネーターインスタンスは、以下のメソッドを介して追加のページネーション情報を提供します:

<div class="overflow-auto" markdown=1>

| メソッド | 説明 |
| --- | --- |
| `$paginator->count()` | 現在のページのアイテム数を取得します。 |
| `$paginator->currentPage()` | 現在のページ番号を取得します。 |
| `$paginator->firstItem()` | 結果内の最初のアイテムの結果番号を取得します。 |
| `$paginator->getOptions()` | ページネーターオプションを取得します。 |
| `$paginator->getUrlRange($start, $end)` | ページネーションURLの範囲を作成します。 |
| `$paginator->hasPages()` | 複数のページに分割するのに十分なアイテムがあるかどうかを判断します。 |
| `$paginator->hasMorePages()` | データストアにさらにアイテムがあるかどうかを判断します。 |
| `$paginator->items()` | 現在のページのアイテムを取得します。 |
| `$paginator->lastItem()` | 結果内の最後のアイテムの結果番号を取得します。 |
| `$paginator->lastPage()` | 最後の利用可能なページのページ番号を取得します。(`simplePaginate`を使用する場合は利用できません) |
| `$paginator->nextPageUrl()` | 次のページのURLを取得します。 |
| `$paginator->onFirstPage()` | ページネーターが最初のページにあるかどうかを判断します。 |
| `$paginator->perPage()` | ページごとに表示されるアイテムの数。 |
| `$paginator->previousPageUrl()` | 前のページのURLを取得します。 |
| `$paginator->total()` | データストア内の一致するアイテムの総数を判断します。(`simplePaginate`を使用する場合は利用できません) |
| `$paginator->url($page)` | 指定されたページ番号のURLを取得します。 |
| `$paginator->getPageName()` | ページを格納するために使用されるクエリ文字列変数を取得します。 |
| `$paginator->setPageName($name)` | ページを格納するために使用されるクエリ文字列変数を設定します。 |
| `$paginator->through($callback)` | コールバックを使用して各アイテムを変換します。 |

</div>

<a name="cursor-paginator-instance-methods"></a>
## カーソルページネーターインスタンスメソッド

各カーソルページネーターインスタンスは、以下のメソッドを介して追加のページネーション情報を提供します:

<div class="overflow-auto" markdown=1>

| メソッド | 説明 |
| --- | --- |
| `$paginator->count()` | 現在のページのアイテム数を取得します。 |
| `$paginator->currentPage()` | 現在のページ番号を取得します。 |
| `$paginator->firstItem()` | 結果内の最初のアイテムの結果番号を取得します。 |
| `$paginator->getOptions()` | ページネーターオプションを取得します。 |
| `$paginator->getUrlRange($start, $end)` | ページネーションURLの範囲を作成します。 |
| `$paginator->hasPages()` | 複数のページに分割するのに十分なアイテムがあるかどうかを判断します。 |
| `$paginator->hasMorePages()` | データストアにさらにアイテムがあるかどうかを判断します。 |
| `$paginator->items()` | 現在のページのアイテムを取得します。 |
| `$paginator->lastItem()` | 結果内の最後のアイテムの結果番号を取得します。 |
| `$paginator->lastPage()` | 最後の利用可能なページのページ番号を取得します。(`simplePaginate`を使用する場合は利用できません) |
| `$paginator->nextPageUrl()` | 次のページのURLを取得します。 |
| `$paginator->onFirstPage()` | ページネーターが最初のページにあるかどうかを判断します。 |
| `$paginator->perPage()` | ページごとに表示されるアイテムの数。 |
| `$paginator->previousPageUrl()` | 前のページのURLを取得します。 |
| `$paginator->total()` | データストア内の一致するアイテムの総数を判断します。(`simplePaginate`を使用する場合は利用できません) |
| `$paginator->url($page)` | 指定されたページ番号のURLを取得します。 |
| `$paginator->getPageName()` | ページを格納するために使用されるクエリ文字列変数を取得します。 |
| `$paginator->setPageName($name)` | ページを格納するために使用されるクエリ文字列変数を設定します。 |
| `$paginator->through($callback)` | コールバックを使用して各アイテムを変換します。 |

</div>

| メソッド                          | 説明                                                                 |
| ------------------------------- | ------------------------------------------------------------------- |
| `$paginator->count()`           | 現在のページのアイテム数を取得します。                                |
| `$paginator->cursor()`          | 現在のカーソルインスタンスを取得します。                             |
| `$paginator->getOptions()`      | ページネータのオプションを取得します。                                |
| `$paginator->hasPages()`        | 複数のページに分割するのに十分なアイテムがあるかどうかを判定します。 |
| `$paginator->hasMorePages()`    | データストアにさらにアイテムがあるかどうかを判定します。             |
| `$paginator->getCursorName()`   | カーソルを保存するために使用されるクエリ文字列変数を取得します。     |
| `$paginator->items()`           | 現在のページのアイテムを取得します。                                |
| `$paginator->nextCursor()`      | 次のアイテムセットのカーソルインスタンスを取得します。              |
| `$paginator->nextPageUrl()`     | 次のページのURLを取得します。                                       |
| `$paginator->onFirstPage()`     | ページネータが最初のページにあるかどうかを判定します。              |
| `$paginator->onLastPage()`      | ページネータが最後のページにあるかどうかを判定します。              |
| `$paginator->perPage()`         | 1ページあたりに表示されるアイテム数を取得します。                   |
| `$paginator->previousCursor()`  | 前のアイテムセットのカーソルインスタンスを取得します。              |
| `$paginator->previousPageUrl()` | 前のページのURLを取得します。                                       |
| `$paginator->setCursorName()`   | カーソルを保存するために使用されるクエリ文字列変数を設定します。     |
| `$paginator->url($cursor)`      | 指定されたカーソルインスタンスのURLを取得します。                   |

</div>
