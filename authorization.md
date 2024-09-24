# 認可

- [はじめに](#introduction)
- [ゲート](#gates)
    - [ゲートの記述](#writing-gates)
    - [アクションの認可](#authorizing-actions-via-gates)
    - [ゲートのレスポンス](#gate-responses)
    - [ゲートチェックのインターセプト](#intercepting-gate-checks)
    - [インライン認可](#inline-authorization)
- [ポリシーの作成](#creating-policies)
    - [ポリシーの生成](#generating-policies)
    - [ポリシーの登録](#registering-policies)
- [ポリシーの記述](#writing-policies)
    - [ポリシーメソッド](#policy-methods)
    - [ポリシーのレスポンス](#policy-responses)
    - [モデルなしのメソッド](#methods-without-models)
    - [ゲストユーザー](#guest-users)
    - [ポリシーフィルター](#policy-filters)
- [ポリシーを使用したアクションの認可](#authorizing-actions-using-policies)
    - [Userモデル経由](#via-the-user-model)
    - [Gateファサード経由](#via-the-gate-facade)
    - [ミドルウェア経由](#via-middleware)
    - [Bladeテンプレート経由](#via-blade-templates)
    - [追加のコンテキストの提供](#supplying-additional-context)
- [認可とInertia](#authorization-and-inertia)

<a name="introduction"></a>
## はじめに

Laravelは、組み込みの[認証](authentication.md)サービスに加えて、特定のリソースに対するユーザーアクションを認可する簡単な方法も提供します。例えば、ユーザーが認証されていても、アプリケーションが管理する特定のEloquentモデルやデータベースレコードを更新または削除する権限がない場合があります。Laravelの認可機能は、この種の認可チェックを管理するための簡単で整理された方法を提供します。

Laravelは、アクションを認可するための主な2つの方法を提供しています：[ゲート](#gates)と[ポリシー](#creating-policies)です。ゲートとポリシーは、ルートとコントローラーのようなものだと考えてください。ゲートは、クロージャベースのシンプルな認可アプローチを提供し、ポリシーは、特定のモデルやリソースに関するロジックをグループ化します。このドキュメントでは、まずゲートを説明し、次にポリシーを説明します。

アプリケーションを構築する際に、ゲートのみを使用するか、ポリシーのみを使用するかを選択する必要はありません。ほとんどのアプリケーションは、ゲートとポリシーの混合を含む可能性があり、それは完全に問題ありません！ゲートは、モデルやリソースに関連しないアクション（例えば、管理者ダッシュボードの表示）に最も適しています。対照的に、特定のモデルやリソースに対するアクションを認可したい場合は、ポリシーを使用する必要があります。

<a name="gates"></a>
## ゲート

<a name="writing-gates"></a>
### ゲートの記述

> WARNING:  
> ゲートは、Laravelの認可機能の基本を学ぶのに最適な方法です。ただし、堅牢なLaravelアプリケーションを構築する際には、認可ルールを整理するために[ポリシー](#creating-policies)の使用を検討する必要があります。

ゲートは、ユーザーが特定のアクションを実行する権限があるかどうかを判断するクロージャです。通常、ゲートは`Gate`ファサードを使用して、`App\Providers\AppServiceProvider`クラスの`boot`メソッド内で定義されます。ゲートは常にユーザーインスタンスを最初の引数として受け取り、関連するEloquentモデルなどの追加の引数をオプションで受け取ることができます。

この例では、特定の`App\Models\Post`モデルを更新できるかどうかを判断するゲートを定義します。ゲートは、ユーザーの`id`を投稿を作成したユーザーの`user_id`と比較することでこれを実現します：

    use App\Models\Post;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Gate::define('update-post', function (User $user, Post $post) {
            return $user->id === $post->user_id;
        });
    }

コントローラーのように、ゲートはクラスコールバックの配列を使用して定義することもできます：

    use App\Policies\PostPolicy;
    use Illuminate\Support\Facades\Gate;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Gate::define('update-post', [PostPolicy::class, 'update']);
    }

<a name="authorizing-actions-via-gates"></a>
### アクションの認可

ゲートを使用してアクションを認可するには、`Gate`ファサードによって提供される`allows`または`denies`メソッドを使用する必要があります。現在認証されているユーザーをこれらのメソッドに渡す必要はありません。Laravelは自動的にユーザーをゲートクロージャに渡します。認可が必要なアクションを実行する前に、アプリケーションのコントローラー内でゲートの認可メソッドを呼び出すのが一般的です：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Gate;

    class PostController extends Controller
    {
        /**
         * 指定された投稿を更新します。
         */
        public function update(Request $request, Post $post): RedirectResponse
        {
            if (! Gate::allows('update-post', $post)) {
                abort(403);
            }

            // 投稿を更新...

            return redirect('/posts');
        }
    }

現在認証されているユーザー以外のユーザーがアクションを実行する権限があるかどうかを判断したい場合は、`Gate`ファサードの`forUser`メソッドを使用できます：

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // ユーザーは投稿を更新できます...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // ユーザーは投稿を更新できません...
    }

`any`または`none`メソッドを使用して、一度に複数のアクションを認可することもできます：

    if (Gate::any(['update-post', 'delete-post'], $post)) {
        // ユーザーは投稿を更新または削除できます...
    }

    if (Gate::none(['update-post', 'delete-post'], $post)) {
        // ユーザーは投稿を更新または削除できません...
    }

<a name="authorizing-or-throwing-exceptions"></a>
#### 認可または例外のスロー

アクションを認可しようとして、ユーザーが指定されたアクションを実行できない場合に自動的に`Illuminate\Auth\Access\AuthorizationException`をスローしたい場合は、`Gate`ファサードの`authorize`メソッドを使用できます。`AuthorizationException`のインスタンスは、Laravelによって自動的に403 HTTPレスポンスに変換されます：

    Gate::authorize('update-post', $post);

    // アクションは認可されました...

<a name="gates-supplying-additional-context"></a>
#### 追加のコンテキストの提供

アクションを認可するためのゲートメソッド（`allows`、`denies`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）と認可[Bladeディレクティブ](#via-blade-templates)（`@can`、`@cannot`、`@canany`）は、配列を2番目の引数として受け取ることができます。これらの配列要素はゲートクロージャにパラメーターとして渡され、認可決定を行う際の追加のコンテキストとして使用できます：

    use App\Models\Category;
    use App\Models\User;
    use Illuminate\Support\Facades\Gate;

    Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
        if (! $user->canPublishToGroup($category->group)) {
            return false;
        } elseif ($pinned && ! $user->canPinPosts()) {
            return false;
        }

        return true;
    });

    if (Gate::check('create-post', [$category, $pinned])) {
        // ユーザーは投稿を作成できます...
    }

<a name="gate-responses"></a>
### ゲートのレスポンス

これまで、ゲートが単純なブール値を返す例を見てきました。しかし、場合によっては、より詳細なレスポンスを返したい場合があります。そのためには、ゲートから`Illuminate\Auth\Access\Response`を返すことができます：

    use App\Models\User;
    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function (User $user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::deny('You must be an administrator.');
    });

ゲートから認可レスポンスを返しても、`Gate::allows`メソッドは依然として単純なブール値を返します。ただし、`Gate::inspect`メソッドを使用して、ゲートから返された完全な認可レスポンスを取得することができます：

    $response = Gate::inspect('edit-settings');

    if ($response->allowed()) {
        // アクションは認可されました...
    } else {
        echo $response->message();
    }

`Gate::authorize`メソッドを使用すると、アクションが認可されない場合に`AuthorizationException`をスローし、認可レスポンスによって提供されたエラーメッセージがHTTPレスポンスに伝播されます：

    Gate::authorize('edit-settings');

    // アクションは認可されました...

<a name="customizing-gate-response-status"></a>
#### HTTPレスポンスステータスのカスタマイズ

ゲートによってアクションが拒否された場合、`403` HTTPレスポンスが返されます。ただし、失敗した認可チェックに対して別のHTTPステータスコードを返すと便利な場合があります。`Illuminate\Auth\Access\Response`クラスの`denyWithStatus`静的コンストラクタを使用して、HTTPステータスコードをカスタマイズできます：

    use App\Models\User;
    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function (User $user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::denyWithStatus(404);
    });

Webアプリケーションでリソースを非表示にするために`404`レスポンスが一般的であるため、便宜上`denyAsNotFound`メソッドが提供されています：

    use App\Models\User;
    use Illuminate\Auth\Access\Response;
    use Illuminate\Support\Facades\Gate;

    Gate::define('edit-settings', function (User $user) {
        return $user->isAdmin
                    ? Response::allow()
                    : Response::denyAsNotFound();
    });

<a name="intercepting-gate-checks"></a>
### ゲートチェックのインターセプト

特定のユーザーに対して常に特定のアクションを許可または拒否したい場合があります。このために、ゲートチェックをインターセプトするための`before`メソッドを使用できます。このメソッドは、他のすべての認可チェックの前に実行されます：

    use Illuminate\Support\Facades\Gate;

    Gate::before(function ($user, $ability) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

`before`クロージャが`null`以外の結果を返す場合、その結果はチェックの結果として扱われます。

ゲートチェックの後に実行される`after`メソッドを定義することもできます：

    Gate::after(function ($user, $ability, $result, $arguments) {
        if ($user->isAdministrator()) {
            return true;
        }
    });

`before`メソッドと同様に、`after`クロージャが`null`以外の結果を返す場合、その結果はチェックの結果として扱われます。

場合によっては、特定のユーザーにすべての権限を付与したいことがあります。すべての他の認可チェックの前に実行されるクロージャを定義するために、`before`メソッドを使用できます。

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`before`クロージャがnull以外の結果を返す場合、その結果は認可チェックの結果と見なされます。

すべての他の認可チェックの後に実行されるクロージャを定義するために、`after`メソッドを使用できます。

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`before`メソッドと同様に、`after`クロージャがnull以外の結果を返す場合、その結果は認可チェックの結果と見なされます。

<a name="inline-authorization"></a>
### インライン認可

時には、現在認証されているユーザーが特定のアクションを実行する権限があるかどうかを、専用のゲートを書かずに判断したいことがあります。Laravelでは、`Gate::allowIf`と`Gate::denyIf`メソッドを介して、この種の「インライン」認可チェックを実行できます。インライン認可は、定義された["before"または"after"の認可フック](#intercepting-gate-checks)を実行しません。

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```

アクションが認可されていない場合、または現在認証されているユーザーがいない場合、Laravelは自動的に`Illuminate\Auth\Access\AuthorizationException`例外をスローします。`AuthorizationException`のインスタンスは、Laravelの例外ハンドラによって自動的に403 HTTPレスポンスに変換されます。

<a name="creating-policies"></a>
## ポリシーの作成

<a name="generating-policies"></a>
### ポリシーの生成

ポリシーは、特定のモデルまたはリソースに関する認可ロジックを整理するクラスです。例えば、アプリケーションがブログである場合、`App\Models\Post`モデルと、ユーザーが投稿の作成や更新などのアクションを認可するための対応する`App\Policies\PostPolicy`があるかもしれません。

`make:policy` Artisanコマンドを使用してポリシーを生成できます。生成されたポリシーは`app/Policies`ディレクトリに配置されます。このディレクトリがアプリケーションに存在しない場合、Laravelはそれを作成します。

```shell
php artisan make:policy PostPolicy
```

`make:policy`コマンドは空のポリシークラスを生成します。リソースの表示、作成、更新、削除に関連するポリシーメソッドの例を含むクラスを生成したい場合、コマンドを実行する際に`--model`オプションを指定できます。

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### ポリシーの登録

<a name="policy-discovery"></a>
#### ポリシーの自動検出

デフォルトでは、Laravelはモデルとポリシーが標準のLaravel命名規則に従っている限り、ポリシーを自動的に検出します。具体的には、ポリシーはモデルを含むディレクトリと同じかそれ以上の階層にある`Policies`ディレクトリに配置する必要があります。例えば、モデルは`app/Models`ディレクトリに配置され、ポリシーは`app/Policies`ディレクトリに配置されます。この場合、Laravelはまず`app/Models/Policies`でポリシーを探し、次に`app/Policies`で探します。さらに、ポリシー名はモデル名と一致し、`Policy`サフィックスを持つ必要があります。したがって、`User`モデルは`UserPolicy`ポリシークラスに対応します。

独自のポリシー検出ロジックを定義したい場合、`Gate::guessPolicyNamesUsing`メソッドを使用してカスタムポリシー検出コールバックを登録できます。通常、このメソッドはアプリケーションの`AppServiceProvider`の`boot`メソッドから呼び出す必要があります。

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // 指定されたモデルのポリシークラス名を返す...
});
```

<a name="manually-registering-policies"></a>
#### 手動でのポリシー登録

`Gate`ファサードを使用して、アプリケーションの`AppServiceProvider`の`boot`メソッド内でポリシーとそれに対応するモデルを手動で登録できます。

```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```

<a name="writing-policies"></a>
## ポリシーの記述

<a name="policy-methods"></a>
### ポリシーメソッド

ポリシークラスが登録されたら、それぞれのアクションに対してメソッドを追加できます。例えば、`PostPolicy`に`update`メソッドを定義し、指定された`App\Models\User`が指定された`App\Models\Post`インスタンスを更新できるかどうかを判断します。

`update`メソッドは`User`と`Post`インスタンスを引数として受け取り、ユーザーが指定された`Post`を更新する権限があるかどうかを示す`true`または`false`を返す必要があります。したがって、この例では、ユーザーの`id`が投稿の`user_id`と一致するかどうかを確認します。

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

必要に応じて、ポリシーに追加のメソッドを定義できます。例えば、`view`や`delete`メソッドを定義して、さまざまな`Post`関連のアクションを認可することができますが、ポリシーメソッドには好きな名前を付けることができます。

Artisanコンソールで`--model`オプションを使用してポリシーを生成した場合、それには`viewAny`、`view`、`create`、`update`、`delete`、`restore`、および`forceDelete`アクションのメソッドが既に含まれています。

> NOTE:  
> すべてのポリシーはLaravelの[サービスコンテナ](container.md)を介して解決されるため、ポリシーのコンストラクタで必要な依存関係をタイプヒントすることで、自動的に注入されます。

<a name="policy-responses"></a>
### ポリシーのレスポンス

これまで、単純なブール値を返すポリシーメソッドのみを検討してきました。しかし、時にはより詳細なレスポンスを返したい場合があります。そのためには、ポリシーメソッドから`Illuminate\Auth\Access\Response`インスタンスを返すことができます。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::deny('You do not own this post.');
}
```

ポリシーから認可レスポンスを返す場合、`Gate::allows`メソッドは依然として単純なブール値を返しますが、`Gate::inspect`メソッドを使用して、ゲートから返される完全な認可レスポンスを取得できます。

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```

`Gate::authorize`メソッドを使用する場合、アクションが認可されていない場合に`AuthorizationException`をスローし、認可レスポンスによって提供されるエラーメッセージはHTTPレスポンスに伝播されます。

```php
Gate::authorize('update', $post);

// The action is authorized...
```

<a name="customizing-policy-response-status"></a>
#### HTTPレスポンスステータスのカスタマイズ

ポリシーメソッドによってアクションが拒否された場合、`403` HTTPレスポンスが返されますが、時には別のHTTPステータスコードを返すと便利な場合があります。`Illuminate\Auth\Access\Response`クラスの`denyWithStatus`静的コンストラクタを使用して、失敗した認可チェックに対して返されるHTTPステータスコードをカスタマイズできます。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::denyWithStatus(404);
}
```

Webアプリケーションでリソースを`404`レスポンスで隠すのが一般的なパターンであるため、便宜上`denyAsNotFound`メソッドが提供されています。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * Determine if the given post can be updated by the user.
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### モデルを持たないメソッド

一部のポリシーメソッドは、現在認証されているユーザーのインスタンスのみを受け取ります。この状況は、`create`アクションを認可する場合に最も一般的です。例えば、ブログを作成している場合、ユーザーが投稿を作成する権限があるかどうかを判断したい場合があります。このような状況では、ポリシーメソッドはユーザーインスタンスのみを受け取る必要があります。

```php
/**
 * Determine if the given user can create posts.
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### ゲストユーザー

デフォルトでは、すべてのゲートとポリシーは、受信したHTTPリクエストが認証されたユーザーによって開始されていない場合、自動的に`false`を返します。しかし、ユーザー引数の定義に「オプション」のタイプヒントを宣言するか、`null`のデフォルト値を提供することで、これらの認可チェックをゲートとポリシーに通過させることができます。

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 指定された投稿がユーザーによって更新できるかどうかを判定します。
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### ポリシーフィルター

特定のユーザーに対して、特定のポリシー内のすべてのアクションを認可したい場合があります。これを実現するには、ポリシーに`before`メソッドを定義します。`before`メソッドは、ポリシー内の他のメソッドよりも前に実行され、意図したポリシーメソッドが実際に呼び出される前にアクションを認可する機会を与えます。この機能は、アプリケーションの管理者がすべてのアクションを実行できるように認可するために最も一般的に使用されます。

```php
use App\Models\User;

/**
 * 事前認可チェックを実行します。
 */
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

特定のタイプのユーザーに対してすべての認可チェックを拒否したい場合は、`before`メソッドから`false`を返すことができます。`null`が返された場合、認可チェックはポリシーメソッドに渡されます。

> WARNING:  
> ポリシークラスに、チェックされる能力の名前と一致する名前のメソッドが含まれていない場合、ポリシークラスの`before`メソッドは呼び出されません。

<a name="authorizing-actions-using-policies"></a>
## ポリシーを使用したアクションの認可

<a name="via-the-user-model"></a>
### ユーザーモデル経由

Laravelアプリケーションに含まれる`App\Models\User`モデルには、アクションを認可するための2つの便利なメソッドが含まれています：`can`と`cannot`です。`can`と`cannot`メソッドは、認可したいアクションの名前と関連するモデルを受け取ります。例えば、ユーザーが特定の`App\Models\Post`モデルを更新する権限があるかどうかを判定します。通常、これはコントローラメソッド内で行われます。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 指定された投稿を更新します。
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // 投稿を更新...

        return redirect('/posts');
    }
}
```

指定されたモデルに対して[ポリシーが登録されている](#registering-policies)場合、`can`メソッドは自動的に適切なポリシーを呼び出し、ブール値の結果を返します。モデルに対してポリシーが登録されていない場合、`can`メソッドは指定されたアクション名に一致するクロージャベースのゲートを呼び出そうとします。

<a name="user-model-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

`create`のようなポリシーメソッドは、モデルインスタンスを必要としない場合があります。このような状況では、`can`メソッドにクラス名を渡すことができます。クラス名は、アクションを認可する際に使用するポリシーを決定するために使用されます。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 投稿を作成します。
     */
    public function store(Request $request): RedirectResponse
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // 投稿を作成...

        return redirect('/posts');
    }
}
```

<a name="via-the-gate-facade"></a>
### `Gate`ファサード経由

`App\Models\User`モデルに提供される便利なメソッドに加えて、`Gate`ファサードの`authorize`メソッドを介してアクションを認可することもできます。

`can`メソッドと同様に、このメソッドは認可したいアクションの名前と関連するモデルを受け取ります。アクションが認可されていない場合、`authorize`メソッドは`Illuminate\Auth\Access\AuthorizationException`例外をスローします。この例外は、Laravelの例外ハンドラによって自動的に403ステータスコードのHTTPレスポンスに変換されます。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * 指定されたブログ投稿を更新します。
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', $post);

        // 現在のユーザーはブログ投稿を更新できます...

        return redirect('/posts');
    }
}
```

<a name="controller-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

前述のように、`create`のようなポリシーメソッドはモデルインスタンスを必要としない場合があります。このような状況では、クラス名を`authorize`メソッドに渡す必要があります。クラス名は、アクションを認可する際に使用するポリシーを決定するために使用されます。

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * 新しいブログ投稿を作成します。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // 現在のユーザーはブログ投稿を作成できます...

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### ミドルウェア経由

Laravelには、受信リクエストがルートやコントローラに到達する前にアクションを認可できるミドルウェアが含まれています。デフォルトでは、`Illuminate\Auth\Middleware\Authorize`ミドルウェアは、`can`[ミドルウェアエイリアス](middleware.md#middleware-aliases)を使用してルートにアタッチできます。これは、Laravelによって自動的に登録されます。ユーザーが投稿を更新できるかどうかを認可するために`can`ミドルウェアを使用する例を見てみましょう。

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーは投稿を更新できます...
})->middleware('can:update,post');
```

この例では、`can`ミドルウェアに2つの引数を渡しています。最初の引数は認可したいアクションの名前で、2番目の引数はルートパラメータで、ポリシーメソッドに渡したいものです。この場合、[暗黙のモデルバインディング](routing.md#implicit-binding)を使用しているため、`App\Models\Post`モデルがポリシーメソッドに渡されます。ユーザーが指定されたアクションを実行する権限がない場合、ミドルウェアによって403ステータスコードのHTTPレスポンスが返されます。

便宜上、`can`メソッドを使用してルートに`can`ミドルウェアをアタッチすることもできます。

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーは投稿を更新できます...
})->can('update', 'post');
```

<a name="middleware-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

繰り返しになりますが、`create`のようなポリシーメソッドはモデルインスタンスを必要としない場合があります。このような状況では、ミドルウェアにクラス名を渡すことができます。クラス名は、アクションを認可する際に使用するポリシーを決定するために使用されます。

```php
Route::post('/post', function () {
    // 現在のユーザーは投稿を作成できます...
})->middleware('can:create,App\Models\Post');
```

文字列のミドルウェア定義内で完全なクラス名を指定することは、扱いにくい場合があります。そのため、`can`メソッドを使用してルートに`can`ミドルウェアをアタッチすることを選択できます。

```php
use App\Models\Post;

Route::post('/post', function () {
    // 現在のユーザーは投稿を作成できます...
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### Bladeテンプレート経由

Bladeテンプレートを書いているとき、特定のアクションを実行する権限がある場合にのみページの一部を表示したい場合があります。例えば、ユーザーが実際に投稿を更新できる場合にのみ、ブログ投稿の更新フォームを表示したい場合があります。このような状況では、`@can`と`@cannot`ディレクティブを使用できます。

```blade
@can('update', $post)
    <!-- 現在のユーザーは投稿を更新できます... -->
@elsecan('create', App\Models\Post::class)
    <!-- 現在のユーザーは新しい投稿を作成できます... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- 現在のユーザーは投稿を更新できません... -->
@elsecannot('create', App\Models\Post::class)
    <!-- 現在のユーザーは新しい投稿を作成できません... -->
@endcannot
```

これらのディレクティブは、`@if`と`@unless`ステートメントを書くための便利なショートカットです。上記の`@can`と`@cannot`ステートメントは、以下のステートメントと同等です。

```blade
@if (Auth::user()->can('update', $post))
    <!-- 現在のユーザーは投稿を更新できます... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- 現在のユーザーは投稿を更新できません... -->
@endunless
```

また、ユーザーが指定されたアクションの配列から任意のアクションを実行する権限があるかどうかを判定することもできます。これを実現するには、`@canany`ディレクティブを使用します。

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- 現在のユーザーは投稿を更新、表示、または削除できます... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- 現在のユーザーは新しい投稿を作成できます... -->
@endcanany
```

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- 現在のユーザーは投稿を更新、閲覧、または削除できます... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成できます... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

他のほとんどの認可メソッドと同様に、アクションがモデルインスタンスを必要としない場合は、`@can` および `@cannot` ディレクティブにクラス名を渡すことができます。

```blade
@can('create', App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成できます... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成できません... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### 追加のコンテキストを提供する

ポリシーを使用してアクションを認可する場合、さまざまな認可関数やヘルパーに2番目の引数として配列を渡すことができます。配列の最初の要素は呼び出すべきポリシーを決定するために使用され、残りの配列要素はポリシーメソッドにパラメーターとして渡され、認可決定を行う際の追加のコンテキストとして使用できます。例えば、以下の `PostPolicy` メソッド定義には追加の `$category` パラメータが含まれています。

    /**
     * 指定された投稿をユーザーが更新できるかどうかを判定します。
     */
    public function update(User $user, Post $post, int $category): bool
    {
        return $user->id === $post->user_id &&
               $user->canUpdateCategory($category);
    }

認証されたユーザーが特定の投稿を更新できるかどうかを判断しようとする場合、このポリシーメソッドを次のように呼び出すことができます。

    /**
     * 指定されたブログ投稿を更新します。
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', [$post, $request->category]);

        // 現在のユーザーはブログ投稿を更新できます...

        return redirect('/posts');
    }

<a name="authorization-and-inertia"></a>
## 認可とInertia

認可は常にサーバー側で処理される必要がありますが、アプリケーションのUIを適切にレンダリングするためにフロントエンドアプリケーションに認可データを提供することが便利な場合がよくあります。Laravelは、Inertiaを使用したフロントエンドに認可情報を公開するための必須の規約を定義していません。

ただし、LaravelのInertiaベースの[スターターキット](starter-kits.md)の1つを使用している場合、アプリケーションにはすでに `HandleInertiaRequests` ミドルウェアが含まれています。このミドルウェアの `share` メソッド内で、アプリケーション内のすべてのInertiaページに提供される共有データを返すことができます。この共有データは、ユーザーの認可情報を定義するための便利な場所として機能します。

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * デフォルトで共有されるプロパティを定義します。
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}
```

