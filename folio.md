# Laravel Folio

- [はじめに](#introduction)
- [インストール](#installation)
    - [ページパス / URI](#page-paths-uris)
    - [サブドメインルーティング](#subdomain-routing)
- [ルートの作成](#creating-routes)
    - [ネストされたルート](#nested-routes)
    - [インデックスルート](#index-routes)
- [ルートパラメータ](#route-parameters)
- [ルートモデルバインディング](#route-model-binding)
    - [ソフトデリートされたモデル](#soft-deleted-models)
- [レンダーフック](#render-hooks)
- [名前付きルート](#named-routes)
- [ミドルウェア](#middleware)
- [ルートキャッシュ](#route-caching)

<a name="introduction"></a>
## はじめに

[Laravel Folio](https://github.com/laravel/folio) は、Laravelアプリケーションのルーティングを簡素化するために設計された強力なページベースのルーターです。Laravel Folioを使用すると、ルートを生成することは、アプリケーションの `resources/views/pages` ディレクトリ内にBladeテンプレートを作成するのと同じくらい簡単になります。

例えば、`/greeting` URLでアクセス可能なページを作成するには、アプリケーションの `resources/views/pages` ディレクトリに `greeting.blade.php` ファイルを作成するだけです：

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを使用してFolioをプロジェクトにインストールします：

```bash
composer require laravel/folio
```

Folioをインストールした後、`folio:install` Artisanコマンドを実行することができます。これにより、Folioのサービスプロバイダがアプリケーションにインストールされます。このサービスプロバイダは、Folioがルート/ページを検索するディレクトリを登録します：

```bash
php artisan folio:install
```

<a name="page-paths-uris"></a>
### ページパス / URI

デフォルトでは、Folioはアプリケーションの `resources/views/pages` ディレクトリからページを提供しますが、Folioサービスプロバイダの `boot` メソッドでこれらのディレクトリをカスタマイズすることができます。

例えば、同じLaravelアプリケーション内で複数のFolioパスを指定すると便利な場合があります。アプリケーションの「管理」エリア用に別のディレクトリを持ち、他のアプリケーションページには別のディレクトリを使用することができます。

これは、`Folio::path` と `Folio::uri` メソッドを使用して実現できます。`path` メソッドは、Folioが受信HTTPリクエストをルーティングする際にページをスキャンするディレクトリを登録し、`uri` メソッドはそのディレクトリのページの「ベースURI」を指定します：

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',

            // ...
        ],
    ]);
```

<a name="subdomain-routing"></a>
### サブドメインルーティング

受信リクエストのサブドメインに基づいてページにルーティングすることもできます。例えば、`admin.example.com` からのリクエストを他のFolioページとは異なるページディレクトリにルーティングしたい場合があります。これは、`Folio::path` メソッドを呼び出した後に `domain` メソッドを呼び出すことで実現できます：

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

`domain` メソッドでは、ドメインまたはサブドメインの一部をパラメータとしてキャプチャすることもできます。これらのパラメータはページテンプレートに注入されます：

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## ルートの作成

FolioのマウントされたディレクトリのいずれかにBladeテンプレートを配置することで、Folioルートを作成できます。デフォルトでは、Folioは `resources/views/pages` ディレクトリをマウントしますが、Folioサービスプロバイダの `boot` メソッドでこれらのディレクトリをカスタマイズできます。

BladeテンプレートがFolioのマウントされたディレクトリに配置されると、ブラウザからすぐにアクセスできます。例えば、`pages/schedule.blade.php` に配置されたページは、ブラウザで `http://example.com/schedule` でアクセスできます。

すべてのFolioページ/ルートのリストをすばやく表示するには、`folio:list` Artisanコマンドを呼び出すことができます：

```bash
php artisan folio:list
```

<a name="nested-routes"></a>
### ネストされたルート

Folioのディレクトリ内に1つ以上のディレクトリを作成することで、ネストされたルートを作成できます。例えば、`/user/profile` でアクセス可能なページを作成するには、`pages/user` ディレクトリ内に `profile.blade.php` テンプレートを作成します：

```bash
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### インデックスルート

場合によっては、特定のページをディレクトリの「インデックス」にすることがあります。Folioディレクトリ内に `index.blade.php` テンプレートを配置することで、そのディレクトリのルートへのリクエストはそのページにルーティングされます：

```bash
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## ルートパラメータ

多くの場合、受信リクエストのURLのセグメントをページに注入して、それらと対話する必要があります。例えば、表示されているプロフィールのユーザーの「ID」にアクセスする必要がある場合があります。これを実現するには、ページのファイル名のセグメントを角括弧で囲みます：

```bash
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

キャプチャされたセグメントは、Bladeテンプレート内で変数としてアクセスできます：

```html
<div>
    User {{ $id }}
</div>
```

複数のセグメントをキャプチャするには、セグメントを囲むセグメントに3つのドット `...` を付けることができます：

```bash
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

複数のセグメントをキャプチャする場合、キャプチャされたセグメントはページに配列として注入されます：

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## ルートモデルバインディング

ページテンプレートのファイル名のワイルドカードセグメントがアプリケーションのEloquentモデルの1つに対応する場合、Folioは自動的にLaravelのルートモデルバインディング機能を利用し、解決されたモデルインスタンスをページに注入しようとします：

```bash
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

キャプチャされたモデルは、Bladeテンプレート内で変数としてアクセスできます。モデルの変数名は「キャメルケース」に変換されます：

```html
<div>
    User {{ $user->id }}
</div>
```

#### キーのカスタマイズ

場合によっては、`id` 以外のカラムを使用してバインドされたEloquentモデルを解決したい場合があります。そのためには、ページのファイル名でカラムを指定できます。例えば、`[Post:slug].blade.php` というファイル名のページは、`id` カラムではなく `slug` カラムを介してバインドされたモデルを解決しようとします。

Windowsでは、モデル名とキーを区切るために `-` を使用する必要があります：`[Post-slug].blade.php`。

#### モデルの場所

デフォルトでは、Folioはアプリケーションの `app/Models` ディレクトリ内でモデルを検索します。ただし、必要に応じて、テンプレートのファイル名に完全修飾モデルクラス名を指定できます：

```bash
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### ソフトデリートされたモデル

デフォルトでは、ソフトデリートされたモデルは暗黙のモデルバインディングを解決する際に取得されません。ただし、必要に応じて、ページのテンプレート内で `withTrashed` 関数を呼び出すことで、ソフトデリートされたモデルを取得するようにFolioに指示できます：

```php
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>

<div>
    User {{ $user->id }}
</div>
```

<a name="render-hooks"></a>
## レンダーフック

デフォルトでは、FolioはページのBladeテンプレートの内容を受信リクエストへのレスポンスとして返します。ただし、ページのテンプレート内で `render` 関数を呼び出すことで、レスポンスをカスタマイズできます。

`render` 関数は、Folioによってレンダリングされる `View` インスタンスを受け取るクロージャを受け入れ、ビューに追加のデータを追加したり、レスポンス全体をカスタマイズしたりできます。`View` インスタンスに加えて、追加のルートパラメータまたはモデルバインディングも `render` クロージャに提供されます：

```php
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>

<div>
    This author has also taken {{ count($photos) }} photos.
</div>
```

<a name="named-routes"></a>
## 名前付きルート

`name` 関数を使用して、特定のページのルートに名前を指定できます：

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

Laravelの名前付きルートと同様に、`route` 関数を使用して、名前が割り当てられたFolioページのURLを生成できます：

```php
<a href="{{ route('users.index') }}">
    All Users
</a>
```

ページにパラメータがある場合、それらの値を `route` 関数に渡すだけです：

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## ミドルウェア

ページのテンプレート内で `middleware` 関数を呼び出すことで、特定のページにミドルウェアを適用できます：

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

または、ページのグループにミドルウェアを割り当てるには、`Folio::path` メソッドを呼び出した後に `middleware` メソッドをチェーンすることができます。

ミドルウェアを適用するページを指定するには、ミドルウェアの配列を、適用するページの対応するURLパターンをキーとして使用します。ワイルドカード文字 `*` を使用できます：

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        // ...
    ],
]);

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

配列のミドルウェアにクロージャを含めて、インラインの匿名ミドルウェアを定義することもできます。

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

<a name="route-caching"></a>
## ルートキャッシュ

Folioを使用する場合、常に[Laravelのルートキャッシュ機能](routing.md#route-caching)を利用するべきです。Folioは`route:cache` Artisanコマンドを監視し、Folioのページ定義とルート名が適切にキャッシュされ、最大のパフォーマンスを確保します。

