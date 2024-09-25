# コントローラ

- [イントロダクション](#introduction)
- [コントローラの作成](#writing-controllers)
    - [基本的なコントローラ](#basic-controllers)
    - [シングルアクションコントローラ](#single-action-controllers)
- [コントローラミドルウェア](#controller-middleware)
- [リソースコントローラ](#resource-controllers)
    - [部分的なリソースルート](#restful-partial-resource-routes)
    - [ネストされたリソース](#restful-nested-resources)
    - [リソースルートの命名](#restful-naming-resource-routes)
    - [リソースルートパラメータの命名](#restful-naming-resource-route-parameters)
    - [リソースルートのスコープ](#restful-scoping-resource-routes)
    - [リソースURIのローカライズ](#restful-localizing-resource-uris)
    - [リソースコントローラの補足](#restful-supplementing-resource-controllers)
    - [シングルトンリソースコントローラ](#singleton-resource-controllers)
- [依存性注入とコントローラ](#dependency-injection-and-controllers)

<a name="introduction"></a>
## イントロダクション

ルートファイル内ですべてのリクエスト処理ロジックをクロージャとして定義する代わりに、この動作を整理するために「コントローラ」クラスを使用することができます。コントローラは、関連するリクエスト処理ロジックを1つのクラスにグループ化することができます。例えば、`UserController`クラスは、ユーザーに関連するすべての受信リクエストを処理することができます。デフォルトでは、コントローラは`app/Http/Controllers`ディレクトリに保存されます。

<a name="writing-controllers"></a>
## コントローラの作成

<a name="basic-controllers"></a>
### 基本的なコントローラ

新しいコントローラをすばやく生成するには、`make:controller` Artisanコマンドを実行できます。デフォルトでは、アプリケーションのすべてのコントローラは`app/Http/Controllers`ディレクトリに保存されます。

```shell
php artisan make:controller UserController
```

基本的なコントローラの例を見てみましょう。コントローラは、受信HTTPリクエストに応答する任意の数のパブリックメソッドを持つことができます。

    <?php

    namespace App\Http\Controllers;

    use App\Models\User;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示します。
         */
        public function show(string $id): View
        {
            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

コントローラクラスとメソッドを作成したら、次のようにコントローラメソッドへのルートを定義できます。

    use App\Http\Controllers\UserController;

    Route::get('/user/{id}', [UserController::class, 'show']);

受信リクエストが指定されたルートURIと一致すると、`App\Http\Controllers\UserController`クラスの`show`メソッドが呼び出され、ルートパラメータがメソッドに渡されます。

> NOTE:  
> コントローラは、基本クラスを**継承する必要はありません**。ただし、すべてのコントローラ間で共有されるメソッドを含む基本コントローラクラスを継承すると便利な場合があります。

<a name="single-action-controllers"></a>
### シングルアクションコントローラ

コントローラアクションが特に複雑な場合、そのアクション専用のコントローラクラスを作成すると便利な場合があります。これを実現するには、コントローラ内に単一の`__invoke`メソッドを定義します。

    <?php

    namespace App\Http\Controllers;

    class ProvisionServer extends Controller
    {
        /**
         * 新しいウェブサーバーをプロビジョニングします。
         */
        public function __invoke()
        {
            // ...
        }
    }

シングルアクションコントローラのルートを登録する場合、コントローラメソッドを指定する必要はありません。代わりに、ルーターにコントローラの名前を渡すだけです。

    use App\Http\Controllers\ProvisionServer;

    Route::post('/server', ProvisionServer::class);

`make:controller` Artisanコマンドの`--invokable`オプションを使用して、呼び出し可能なコントローラを生成できます。

```shell
php artisan make:controller ProvisionServer --invokable
```

> NOTE:  
> コントローラスタブは、[スタブの公開](artisan.md#stub-customization)を使用してカスタマイズできます。

<a name="controller-middleware"></a>
## コントローラミドルウェア

[ミドルウェア](middleware.md)は、ルートファイル内のコントローラのルートに割り当てることができます。

    Route::get('/profile', [UserController::class, 'show'])->middleware('auth');

または、コントローラクラス内でミドルウェアを指定すると便利な場合があります。これを行うには、コントローラが`HasMiddleware`インターフェースを実装する必要があります。これは、コントローラが静的な`middleware`メソッドを持つべきであることを指示します。このメソッドから、コントローラのアクションに適用されるミドルウェアの配列を返すことができます。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Routing\Controllers\HasMiddleware;
    use Illuminate\Routing\Controllers\Middleware;

    class UserController extends Controller implements HasMiddleware
    {
        /**
         * コントローラに割り当てるべきミドルウェアを取得します。
         */
        public static function middleware(): array
        {
            return [
                'auth',
                new Middleware('log', only: ['index']),
                new Middleware('subscribed', except: ['store']),
            ];
        }

        // ...
    }

コントローラミドルウェアをクロージャとして定義することもできます。これにより、ミドルウェアクラス全体を書くことなく、インラインミドルウェアを定義する便利な方法が提供されます。

    use Closure;
    use Illuminate\Http\Request;

    /**
     * コントローラに割り当てるべきミドルウェアを取得します。
     */
    public static function middleware(): array
    {
        return [
            function (Request $request, Closure $next) {
                return $next($request);
            },
        ];
    }

<a name="resource-controllers"></a>
## リソースコントローラ

アプリケーション内の各Eloquentモデルを「リソース」と考えると、アプリケーション内の各リソースに対して同じ一連のアクションを実行するのが一般的です。例えば、アプリケーションに`Photo`モデルと`Movie`モデルが含まれているとします。ユーザーは、これらのリソースを作成、読み取り、更新、または削除できる可能性があります。

この一般的なユースケースのため、Laravelのリソースルーティングは、1行のコードで典型的な作成、読み取り、更新、削除（"CRUD"）ルートをコントローラに割り当てます。開始するには、`make:controller` Artisanコマンドの`--resource`オプションを使用して、これらのアクションを処理するコントローラをすばやく作成できます。

```shell
php artisan make:controller PhotoController --resource
```

このコマンドは、`app/Http/Controllers/PhotoController.php`にコントローラを生成します。コントローラには、利用可能な各リソース操作のメソッドが含まれます。次に、コントローラを指すリソースルートを登録できます。

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);

この単一のルート宣言により、リソースに対するさまざまなアクションを処理する複数のルートが作成されます。生成されたコントローラには、これらのアクションごとにスタブ化されたメソッドが既に含まれています。`route:list` Artisanコマンドを実行することで、アプリケーションのルートの概要をいつでも確認できます。

`resources`メソッドに配列を渡すことで、一度に多くのリソースコントローラを登録することもできます。

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controllers"></a>
#### リソースコントローラによって処理されるアクション

<div class="overflow-auto" markdown=1>

| メソッド    | URI                    | アクション  | ルート名       |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

</div>

<a name="customizing-missing-model-behavior"></a>
#### モデルが見つからない場合の動作のカスタマイズ

通常、暗黙的にバインドされたリソースモデルが見つからない場合、404 HTTPレスポンスが生成されます。ただし、リソースルートを定義する際に`missing`メソッドを呼び出すことで、この動作をカスタマイズできます。`missing`メソッドは、暗黙的にバインドされたモデルがリソースのいずれのルートでも見つからない場合に呼び出されるクロージャを受け取ります。

    use App\Http\Controllers\PhotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('photos', PhotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('photos.index');
            });

<a name="soft-deleted-models"></a>
#### ソフトデリートされたモデル

通常、暗黙的なモデルバインディングは、[ソフトデリート](eloquent.md#soft-deleting)されたモデルを取得しません。代わりに404 HTTPレスポンスを返します。ただし、リソースルートを定義する際に`withTrashed`メソッドを呼び出すことで、フレームワークにソフトデリートされたモデルを許可するよう指示できます。

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->withTrashed();

引数なしで`withTrashed`を呼び出すと、ソフトデリートされたモデルが`show`、`edit`、および`update`リソースルートで許可されます。`withTrashed`メソッドに配列を渡すことで、これらのルートのサブセットを指定できます。

    Route::resource('photos', PhotoController::class)->withTrashed(['show']);

<a name="specifying-the-resource-model"></a>
#### リソースモデルの指定

[route model binding](routing.md#route-model-binding)を使用していて、リソースコントローラのメソッドにモデルインスタンスをタイプヒントする場合、コントローラを生成する際に`--model`オプションを使用できます。

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### フォームリクエストの生成

リソースコントローラを生成する際に`--requests`オプションを指定すると、Artisanにコントローラのストレージおよび更新メソッド用の[フォームリクエストクラス](validation.md#form-request-validation)を生成するよう指示できます。

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### 部分的なリソースルート

リソースルートを宣言する際、コントローラが処理すべきアクションのサブセットを指定できます。デフォルトの全アクションセットの代わりに、次のように指定します。

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

<a name="api-resource-routes"></a>
#### APIリソースルート

APIによって消費されるリソースルートを宣言する場合、通常は`create`や`edit`のようなHTMLテンプレートを表示するルートを除外したいでしょう。便宜上、`apiResource`メソッドを使用してこれらの2つのルートを自動的に除外できます。

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

`apiResources`メソッドに配列を渡すことで、一度に多くのAPIリソースコントローラを登録できます。

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

`create`や`edit`メソッドを含まないAPIリソースコントローラを素早く生成するには、`make:controller`コマンドを実行する際に`--api`スイッチを使用します。

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### ネストされたリソース

時には、ネストされたリソースへのルートを定義する必要があるかもしれません。例えば、写真リソースは、写真に添付される複数のコメントを持つことができます。リソースコントローラをネストするには、ルート宣言で「ドット」記法を使用します。

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

このルートは、次のようなURIでアクセスできるネストされたリソースを登録します。

```
/photos/{photo}/comments/{comment}
```

<a name="scoping-nested-resources"></a>
#### ネストされたリソースのスコープ

Laravelの[暗黙的なモデルバインディング](routing.md#implicit-model-binding-scoping)機能は、自動的にネストされたバインディングをスコープし、解決された子モデルが親モデルに属していることを確認できます。ネストされたリソースを定義する際に`scoped`メソッドを使用することで、自動スコーピングを有効にし、Laravelに子リソースを取得するフィールドを指示できます。これを実現する方法の詳細については、[リソースルートのスコーピング](#restful-scoping-resource-routes)に関するドキュメントを参照してください。

<a name="shallow-nesting"></a>
#### 浅いネスト

多くの場合、URI内に親と子の両方のIDを持つ必要はありません。子IDはすでに一意の識別子であるためです。URIセグメント内で自動インクリメント主キーなどの一意の識別子を使用してモデルを識別する場合は、「浅いネスト」を使用することを選択できます。

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

このルート定義は、次のルートを定義します。

<div class="overflow-auto" markdown=1>

| 動詞      | URI                               | アクション  | ルート名             |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/photos/{photo}/comments`        | index   | photos.comments.index  |
| GET       | `/photos/{photo}/comments/create` | create  | photos.comments.create |
| POST      | `/photos/{photo}/comments`        | store   | photos.comments.store  |
| GET       | `/comments/{comment}`             | show    | comments.show          |
| GET       | `/comments/{comment}/edit`        | edit    | comments.edit          |
| PUT/PATCH | `/comments/{comment}`             | update  | comments.update        |
| DELETE    | `/comments/{comment}`             | destroy | comments.destroy       |

</div>

<a name="restful-naming-resource-routes"></a>
### リソースルートの命名

デフォルトでは、すべてのリソースコントローラのアクションにはルート名が付けられています。ただし、`names`配列を渡すことで、これらの名前を上書きできます。

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### リソースルートパラメータの命名

デフォルトでは、`Route::resource`はリソース名の「単数形」バージョンに基づいてリソースルートのパラメータを作成します。リソースごとにこれを簡単に上書きするには、`parameters`メソッドを使用します。`parameters`メソッドに渡される配列は、リソース名とパラメータ名の連想配列である必要があります。

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

上記の例では、リソースの`show`ルートに対して次のURIが生成されます。

```
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### リソースルートのスコーピング

Laravelの[スコープ付き暗黙的なモデルバインディング](routing.md#implicit-model-binding-scoping)機能は、自動的にネストされたバインディングをスコープし、解決された子モデルが親モデルに属していることを確認できます。ネストされたリソースを定義する際に`scoped`メソッドを使用することで、自動スコーピングを有効にし、Laravelに子リソースを取得するフィールドを指示できます。

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

このルートは、次のようなURIでアクセスできるスコープ付きネストされたリソースを登録します。

```
/photos/{photo}/comments/{comment:slug}
```

ネストされたルートパラメータとしてカスタムキー付きの暗黙的なバインディングを使用する場合、Laravelは自動的にクエリをスコープし、親モデルのリレーション名を推測して子モデルを取得します。この場合、`Photo`モデルには`comments`（ルートパラメータ名の複数形）というリレーションがあると想定され、これを使用して`Comment`モデルを取得します。

<a name="restful-localizing-resource-uris"></a>
### リソースURIのローカライズ

デフォルトでは、`Route::resource`はリソースURIを英語の動詞と複数形規則を使用して作成します。`create`と`edit`アクション動詞をローカライズする必要がある場合は、`Route::resourceVerbs`メソッドを使用できます。これは、アプリケーションの`App\Providers\AppServiceProvider`の`boot`メソッドの先頭で行うことができます。

```php
/**
 * 任意のアプリケーションサービスのブートストラップ。
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

Laravelの複数形化サポートは、[必要に応じて設定できるいくつかの異なる言語](localization.md#pluralization-language)をサポートしています。動詞と複数形化言語をカスタマイズした後、`Route::resource('publicacion', PublicacionController::class)`のようなリソースルート登録は、次のURIを生成します。

```
/publicacion/crear

/publicacion/{publicaciones}/editar
```

<a name="restful-supplementing-resource-controllers"></a>
### リソースコントローラの補足

デフォルトのリソースルートに加えて、リソースコントローラに追加のルートを追加する必要がある場合は、`Route::resource`メソッドを呼び出す前にそれらのルートを定義する必要があります。そうしないと、`resource`メソッドによって定義されたルートが意図せずに補足ルートよりも優先される可能性があります。

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> NOTE:  
> コントローラの焦点を絞り込むことを忘れないでください。リソースアクションの典型的なセット以外のメソッドを頻繁に必要とする場合は、コントローラを2つに分割することを検討してください。

<a name="singleton-resource-controllers"></a>
### シングルトンリソースコントローラ

時には、アプリケーションが単一のインスタンスしか持たないリソースを持つことがあります。例えば、ユーザーの「プロフィール」は編集または更新できますが、ユーザーは複数の「プロフィール」を持つことはできません。同様に、画像は1つの「サムネイル」しか持つことができません。これらのリソースは「シングルトンリソース」と呼ばれ、リソースのインスタンスは1つしか存在しません。このようなシナリオでは、「シングルトン」リソースコントローラを登録できます。

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

上記のシングルトンリソース定義は、次のルートを登録します。ご覧の通り、シングルトンリソースに対して「作成」ルートは登録されず、登録されたルートはリソースのインスタンスが1つしか存在しないため、識別子を受け入れません。

<div class="overflow-auto" markdown=1>

| 動詞      | URI                               | アクション  | ルート名             |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/profile`                        | show    | profile.show           |
| GET       | `/profile/edit`                   | edit    | profile.edit           |
| PUT/PATCH | `/profile`                        | update  | profile.update         |

</div>

<div class="overflow-auto" markdown=1>

| 動詞      | URI             | アクション | ルート名       |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | 表示   | profile.show   |
| GET       | `/profile/edit` | 編集   | profile.edit   |
| PUT/PATCH | `/profile`      | 更新   | profile.update |

</div>

シングルトンリソースは、標準リソース内にネストすることもできます。

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

この例では、`photos`リソースは[標準リソースルート](#actions-handled-by-resource-controllers)をすべて受け取りますが、`thumbnail`リソースは以下のルートを持つシングルトンリソースとなります。

<div class="overflow-auto" markdown=1>

| 動詞      | URI                              | アクション | ルート名              |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | 表示   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | 編集   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | 更新   | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### 作成可能なシングルトンリソース

場合によっては、シングルトンリソースに対して作成および保存のルートを定義したいことがあります。これを実現するには、シングルトンリソースルートを登録する際に`creatable`メソッドを呼び出します。

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

この例では、以下のルートが登録されます。作成可能なシングルトンリソースに対して`DELETE`ルートも登録されることがわかります。

<div class="overflow-auto" markdown=1>

| 動詞      | URI                                | アクション  | ルート名               |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | 作成  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | 保存   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | 表示    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | 編集    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | 更新  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | 削除 | photos.thumbnail.destroy |

</div>

シングルトンリソースに対して`DELETE`ルートを登録したいが、作成や保存のルートは登録したくない場合は、`destroyable`メソッドを使用できます。

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### APIシングルトンリソース

`apiSingleton`メソッドは、APIを介して操作されるシングルトンリソースを登録するために使用できます。そのため、`create`および`edit`ルートは不要です。

```php
Route::apiSingleton('profile', ProfileController::class);
```

もちろん、APIシングルトンリソースは`creatable`であり、リソースに対して`store`および`destroy`ルートを登録します。

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

<a name="dependency-injection-and-controllers"></a>
## 依存性注入とコントローラ

<a name="constructor-injection"></a>
#### コンストラクタインジェクション

Laravelの[サービスコンテナ](container.md)は、すべてのLaravelコントローラを解決するために使用されます。その結果、コントローラが必要とする依存関係をコンストラクタで型宣言することができます。宣言された依存関係は自動的に解決され、コントローラインスタンスに注入されます。

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 新しいコントローラインスタンスの作成
         */
        public function __construct(
            protected UserRepository $users,
        ) {}
    }

<a name="method-injection"></a>
#### メソッドインジェクション

コンストラクタインジェクションに加えて、コントローラのメソッドに対して依存関係を型宣言することもできます。メソッドインジェクションの一般的な使用例は、コントローラメソッドに`Illuminate\Http\Request`インスタンスを注入することです。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 新しいユーザーを保存
         */
        public function store(Request $request): RedirectResponse
        {
            $name = $request->name;

            // ユーザーを保存...

            return redirect('/users');
        }
    }

コントローラメソッドがルートパラメータからの入力も期待している場合は、他の依存関係の後にルート引数をリストします。例えば、ルートが次のように定義されている場合：

    use App\Http\Controllers\UserController;

    Route::put('/user/{id}', [UserController::class, 'update']);

`Illuminate\Http\Request`を型宣言し、`id`パラメータにアクセスするには、コントローラメソッドを次のように定義します：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーを更新
         */
        public function update(Request $request, string $id): RedirectResponse
        {
            // ユーザーを更新...

            return redirect('/users');
        }
    }
