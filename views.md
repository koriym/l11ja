# ビュー

- [イントロダクション](#introduction)
    - [React / Vue でのビューの記述](#writing-views-in-react-or-vue)
- [ビューの作成とレンダリング](#creating-and-rendering-views)
    - [ネストされたビューディレクトリ](#nested-view-directories)
    - [最初に利用可能なビューの作成](#creating-the-first-available-view)
    - [ビューが存在するかどうかの判定](#determining-if-a-view-exists)
- [ビューへのデータの受け渡し](#passing-data-to-views)
    - [すべてのビューとデータの共有](#sharing-data-with-all-views)
- [ビューコンポーザ](#view-composers)
    - [ビュークリエイター](#view-creators)
- [ビューの最適化](#optimizing-views)

<a name="introduction"></a>
## イントロダクション

もちろん、ルートやコントローラから直接HTMLドキュメント全体の文字列を返すのは現実的ではありません。幸いなことに、ビューはすべてのHTMLを別々のファイルに配置する便利な方法を提供します。

ビューは、コントローラ/アプリケーションロジックをプレゼンテーションロジックから分離し、`resources/views`ディレクトリに保存されます。Laravelを使用する場合、ビューテンプレートは通常[Bladeテンプレート言語](blade.md)を使用して記述されます。シンプルなビューは次のようになります：

```blade
<!-- ビューはresources/views/greeting.blade.phpに保存されています -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

このビューが`resources/views/greeting.blade.php`に保存されている場合、グローバルな`view`ヘルパーを使用して次のように返すことができます：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> NOTE:  
> Bladeテンプレートの記述方法について詳しく知りたい場合は、[Bladeドキュメント](blade.md)を参照してください。

<a name="writing-views-in-react-or-vue"></a>
### React / Vue でのビューの記述

Bladeを介してPHPでフロントエンドテンプレートを記述する代わりに、多くの開発者はテンプレートをReactまたはVueを使用して記述することを好みます。Laravelは[Inertia](https://inertiajs.com/)のおかげで、React/VueのフロントエンドをLaravelのバックエンドに結びつけるのが簡単になります。

BreezeとJetstreamの[スターターキット](starter-kits.md)は、Inertiaを搭載した次のLaravelアプリケーションのための素晴らしい出発点を提供します。さらに、[Laravel Bootcamp](https://bootcamp.laravel.com)は、VueとReactの例を含む、Inertiaを搭載したLaravelアプリケーションの構築の完全なデモンストレーションを提供します。

<a name="creating-and-rendering-views"></a>
## ビューの作成とレンダリング

アプリケーションの`resources/views`ディレクトリに`.blade.php`拡張子を持つファイルを配置するか、`make:view` Artisanコマンドを使用してビューを作成できます：

```shell
php artisan make:view greeting
```

`.blade.php`拡張子は、ファイルが[Bladeテンプレート](blade.md)を含んでいることをフレームワークに通知します。BladeテンプレートにはHTMLと、値を簡単に出力したり、"if"文を作成したり、データを反復処理したりするためのBladeディレクティブが含まれます。

ビューを作成したら、アプリケーションのルートまたはコントローラからグローバルな`view`ヘルパーを使用して返すことができます：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

ビューは`View`ファサードを使用して返すこともできます：

    use Illuminate\Support\Facades\View;

    return View::make('greeting', ['name' => 'James']);

ご覧のように、`view`ヘルパーに渡される最初の引数は、`resources/views`ディレクトリ内のビューファイルの名前に対応しています。2番目の引数は、ビューで利用可能にする必要があるデータの配列です。この場合、`name`変数を渡しています。これは[Blade構文](blade.md)を使用してビューに表示されます。

<a name="nested-view-directories"></a>
### ネストされたビューディレクトリ

ビューは`resources/views`ディレクトリのサブディレクトリ内にもネストされることがあります。"ドット"記法を使用してネストされたビューを参照できます。たとえば、ビューが`resources/views/admin/profile.blade.php`に保存されている場合、アプリケーションのルート/コントローラから次のように返すことができます：

    return view('admin.profile', $data);

> WARNING:  
> ビューディレクトリ名には`.`文字を含めないでください。

<a name="creating-the-first-available-view"></a>
### 最初に利用可能なビューの作成

`View`ファサードの`first`メソッドを使用して、指定されたビューの配列の中で最初に存在するビューを作成できます。これは、アプリケーションまたはパッケージがビューをカスタマイズまたは上書きできる場合に便利です：

    use Illuminate\Support\Facades\View;

    return View::first(['custom.admin', 'admin'], $data);

<a name="determining-if-a-view-exists"></a>
### ビューが存在するかどうかの判定

ビューが存在するかどうかを判定する必要がある場合は、`View`ファサードを使用できます。`exists`メソッドは、ビューが存在する場合に`true`を返します：

    use Illuminate\Support\Facades\View;

    if (View::exists('admin.profile')) {
        // ...
    }

<a name="passing-data-to-views"></a>
## ビューへのデータの受け渡し

前の例で見たように、ビューにデータの配列を渡して、そのデータをビューで利用可能にすることができます：

    return view('greetings', ['name' => 'Victoria']);

この方法で情報を渡す場合、データはキー/値のペアを持つ配列である必要があります。ビューにデータを提供した後、データのキーを使用してビュー内で各値にアクセスできます。例えば`<?php echo $name; ?>`のように。

`view`ヘルパー関数に完全なデータ配列を渡す代わりに、`with`メソッドを使用してビューに個々のデータを追加することもできます。`with`メソッドはビューオブジェクトのインスタンスを返すため、ビューを返す前にメソッドを連鎖させることができます：

    return view('greeting')
                ->with('name', 'Victoria')
                ->with('occupation', 'Astronaut');

<a name="sharing-data-with-all-views"></a>
### すべてのビューとデータの共有

アプリケーションによってレンダリングされるすべてのビューとデータを共有する必要がある場合があります。`View`ファサードの`share`メソッドを使用して行うことができます。通常、`share`メソッドの呼び出しはサービスプロバイダの`boot`メソッド内に配置する必要があります。`App\Providers\AppServiceProvider`クラスに追加するか、それらを格納するための別のサービスプロバイダを生成することができます：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            View::share('key', 'value');
        }
    }

<a name="view-composers"></a>
## ビューコンポーザ

ビューコンポーザは、ビューがレンダリングされるときに呼び出されるコールバックまたはクラスメソッドです。ビューがレンダリングされるたびにビューにバインドしたいデータがある場合、ビューコンポーザはそのロジックを1つの場所に整理するのに役立ちます。ビューコンポーザは、アプリケーション内の複数のルートまたはコントローラから同じビューが返され、常に特定のデータが必要な場合に特に便利です。

通常、ビューコンポーザはアプリケーションの[サービスプロバイダ](providers.md)の1つに登録されます。この例では、`App\Providers\AppServiceProvider`がこのロジックを格納すると仮定します。

`View`ファサードの`composer`メソッドを使用してビューコンポーザを登録します。Laravelにはクラスベースのビューコンポーザのためのデフォルトディレクトリは含まれていないため、自由に整理することができます。たとえば、アプリケーションのすべてのビューコンポーザを格納するために`app/View/Composers`ディレクトリを作成することができます：

    <?php

    namespace App\Providers;

    use App\View\Composers\ProfileComposer;
    use Illuminate\Support\Facades;
    use Illuminate\Support\ServiceProvider;
    use Illuminate\View\View;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            // クラスベースのコンポーザを使用...
            Facades\View::composer('profile', ProfileComposer::class);

            // クロージャベースのコンポーザを使用...
            Facades\View::composer('welcome', function (View $view) {
                // ...
            });

            Facades\View::composer('dashboard', function (View $view) {
                // ...
            });
        }
    }

これで、ビューコンポーザが登録されたので、`profile`ビューがレンダリングされるたびに`App\View\Composers\ProfileComposer`クラスの`compose`メソッドが実行されます。ビューコンポーザクラスの例を見てみましょう：

    <?php

    namespace App\View\Composers;

    use App\Repositories\UserRepository;
    use Illuminate\View\View;

    class ProfileComposer
    {
        /**
         * 新しいプロフィールコンポーザを作成します。
         */
        public function __construct(
            protected UserRepository $users,
        ) {}

        /**
         * ビューにデータをバインドします。
         */
        public function compose(View $view): void
        {
            $view->with('count', $this->users->count());
        }
    }

ご覧のように、すべてのビューコンポーザは[サービスコンテナ](container.md)を介して解決されるため、コンポーザのコンストラクタ内で必要な依存関係をタイプヒントで指定できます。

<a name="attaching-a-composer-to-multiple-views"></a>
#### 複数のビューへのコンポーザのアタッチ

`composer`メソッドの最初の引数としてビューの配列を渡すことで、一度に複数のビューにビューコンポーザをアタッチできます：

    use App\Views\Composers\MultiComposer;
    use Illuminate\Support\Facades\View;

    View::composer(
        ['profile', 'dashboard'],
        MultiComposer::class
    );

```php
View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

`composer`メソッドは、ワイルドカードとして`*`文字も受け入れるため、コンポーザーをすべてのビューにアタッチすることができます:

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

<a name="view-creators"></a>
### ビュークリエイター

ビュー「クリエイター」はビューコンポーザーと非常によく似ていますが、ビューがレンダリングされる直前ではなく、ビューがインスタンス化された直後に実行されます。ビュークリエイターを登録するには、`creator`メソッドを使用します:

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

<a name="optimizing-views"></a>
## ビューの最適化

デフォルトでは、Bladeテンプレートビューは必要に応じてコンパイルされます。ビューをレンダリングするリクエストが実行されると、Laravelはビューのコンパイル済みバージョンが存在するかどうかを判断します。ファイルが存在する場合、Laravelは未コンパイルのビューがコンパイル済みビューよりも最近に変更されたかどうかを判断します。コンパイル済みビューが存在しないか、未コンパイルビューが変更されている場合、Laravelはビューを再コンパイルします。

リクエスト中にビューをコンパイルすると、パフォーマンスにわずかなマイナスの影響がある可能性があります。そのため、Laravelは`view:cache` Artisanコマンドを提供し、アプリケーションで使用されるすべてのビューを事前にコンパイルします。パフォーマンスを向上させるために、このコマンドをデプロイプロセスの一部として実行することをお勧めします:

```shell
php artisan view:cache
```

ビューキャッシュをクリアするには、`view:clear`コマンドを使用できます:

```shell
php artisan view:clear
```

