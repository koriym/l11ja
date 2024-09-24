# パッケージ開発

- [はじめに](#introduction)
    - [ファサードについての注意](#a-note-on-facades)
- [パッケージの発見](#package-discovery)
- [サービスプロバイダ](#service-providers)
- [リソース](#resources)
    - [設定](#configuration)
    - [マイグレーション](#migrations)
    - [ルート](#routes)
    - [言語ファイル](#language-files)
    - [ビュー](#views)
    - [ビューコンポーネント](#view-components)
    - ["About" Artisanコマンド](#about-artisan-command)
- [コマンド](#commands)
- [公開アセット](#public-assets)
- [ファイルグループの公開](#publishing-file-groups)

<a name="introduction"></a>
## はじめに

パッケージは、Laravelに機能を追加するための主要な方法です。パッケージは、[Carbon](https://github.com/briannesbitt/Carbon)のような日付を扱う素晴らしい方法から、Spatieの[Laravel Media Library](https://github.com/spatie/laravel-medialibrary)のようにEloquentモデルにファイルを関連付けることができるパッケージまで、さまざまなものがあります。

パッケージにはさまざまな種類があります。一部のパッケージはスタンドアロンであり、どのPHPフレームワークでも動作します。CarbonやPestはスタンドアロンパッケージの例です。これらのパッケージは、`composer.json`ファイルで要求することでLaravelで使用できます。

一方、他のパッケージはLaravelでの使用を特に意図しています。これらのパッケージには、Laravelアプリケーションを強化するために意図されたルート、コントローラ、ビュー、設定が含まれる場合があります。このガイドでは、主にLaravel固有のパッケージの開発について説明します。

<a name="a-note-on-facades"></a>
### ファサードについての注意

Laravelアプリケーションを書く際、コントラクトを使用するかファサードを使用するかは、通常、テスト可能性の観点からほぼ同等であるため、問題になりません。しかし、パッケージを書く場合、パッケージは通常、Laravelのすべてのテストヘルパーにアクセスできません。パッケージが典型的なLaravelアプリケーション内にインストールされているかのようにパッケージのテストを書きたい場合は、[Orchestral Testbench](https://github.com/orchestral/testbench)パッケージを使用できます。

<a name="package-discovery"></a>
## パッケージの発見

Laravelアプリケーションの`bootstrap/providers.php`ファイルには、Laravelによってロードされるべきサービスプロバイダのリストが含まれています。しかし、ユーザーが手動でサービスプロバイダをリストに追加する必要がないように、パッケージの`composer.json`ファイルの`extra`セクションでプロバイダを定義することができます。これにより、Laravelによって自動的にロードされます。サービスプロバイダに加えて、登録したい[ファサード](facades.md)をリストすることもできます：

```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

パッケージが発見のために設定されると、Laravelはインストール時に自動的にそのサービスプロバイダとファサードを登録し、パッケージのユーザーに便利なインストール体験を提供します。

<a name="opting-out-of-package-discovery"></a>
#### パッケージの発見をオプトアウトする

パッケージの利用者として、パッケージの発見を無効にしたい場合は、アプリケーションの`composer.json`ファイルの`extra`セクションにパッケージ名をリストすることができます：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

アプリケーションの`dont-discover`ディレクティブ内で`*`文字を使用して、すべてのパッケージの発見を無効にすることもできます：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

<a name="service-providers"></a>
## サービスプロバイダ

[サービスプロバイダ](providers.md)は、パッケージとLaravelの接続点です。サービスプロバイダは、Laravelの[サービスコンテナ](container.md)にバインドし、パッケージのリソース（ビュー、設定、言語ファイルなど）をロードする場所をLaravelに通知する役割を担います。

サービスプロバイダは`Illuminate\Support\ServiceProvider`クラスを拡張し、`register`と`boot`の2つのメソッドを含みます。ベースの`ServiceProvider`クラスは`illuminate/support` Composerパッケージにあり、独自のパッケージの依存関係に追加する必要があります。サービスプロバイダの構造と目的について詳しくは、[そのドキュメント](providers.md)を参照してください。

<a name="resources"></a>
## リソース

<a name="configuration"></a>
### 設定

通常、パッケージの設定ファイルをアプリケーションの`config`ディレクトリに公開する必要があります。これにより、パッケージのユーザーはデフォルトの設定オプションを簡単にオーバーライドできます。設定ファイルを公開できるようにするには、サービスプロバイダの`boot`メソッドから`publishes`メソッドを呼び出します：

    /**
     * パッケージサービスのあらゆる初期化処理を行う。
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../config/courier.php' => config_path('courier.php'),
        ]);
    }

これで、パッケージのユーザーがLaravelの`vendor:publish`コマンドを実行すると、ファイルが指定された公開場所にコピーされます。設定が公開されると、その値は他の設定ファイルと同様にアクセスできます：

    $value = config('courier.option');

> WARNING:  
> 設定ファイルにクロージャを定義してはいけません。ユーザーが`config:cache` Artisanコマンドを実行すると、正しくシリアライズできません。

<a name="default-package-configuration"></a>
#### デフォルトのパッケージ設定

独自のパッケージ設定ファイルをアプリケーションの公開されたコピーとマージすることもできます。これにより、ユーザーは設定ファイルの公開されたコピーで実際にオーバーライドしたいオプションのみを定義できます。設定ファイルの値をマージするには、サービスプロバイダの`register`メソッド内で`mergeConfigFrom`メソッドを使用します。

`mergeConfigFrom`メソッドは、パッケージの設定ファイルへのパスを最初の引数として受け取り、アプリケーションの設定ファイルの名前を2番目の引数として受け取ります：

    /**
     * アプリケーションサービスの登録。
     */
    public function register(): void
    {
        $this->mergeConfigFrom(
            __DIR__.'/../config/courier.php', 'courier'
        );
    }

> WARNING:  
> このメソッドは設定配列の最初のレベルのみをマージします。ユーザーが多次元設定配列を部分的に定義した場合、不足しているオプションはマージされません。

<a name="routes"></a>
### ルート

パッケージにルートが含まれている場合、`loadRoutesFrom`メソッドを使用してそれらをロードできます。このメソッドは、アプリケーションのルートがキャッシュされているかどうかを自動的に判断し、ルートが既にキャッシュされている場合はルートファイルをロードしません：

    /**
     * パッケージサービスのあらゆる初期化処理を行う。
     */
    public function boot(): void
    {
        $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
    }

<a name="migrations"></a>
### マイグレーション

パッケージに[データベースマイグレーション](migrations.md)が含まれている場合、`publishesMigrations`メソッドを使用して、指定されたディレクトリまたはファイルにマイグレーションが含まれていることをLaravelに通知できます。Laravelがマイグレーションを公開すると、ファイル名内のタイムスタンプが現在の日付と時刻に自動的に更新されます：

    /**
     * パッケージサービスのあらゆる初期化処理を行う。
     */
    public function boot(): void
    {
        $this->publishesMigrations([
            __DIR__.'/../database/migrations' => database_path('migrations'),
        ]);
    }

<a name="language-files"></a>
### 言語ファイル

パッケージに[言語ファイル](localization.md)が含まれている場合、`loadTranslationsFrom`メソッドを使用してLaravelにそれらをロードする方法を通知できます。たとえば、パッケージが`courier`という名前の場合、サービスプロバイダの`boot`メソッドに次のように追加する必要があります：

    /**
     * パッケージサービスのあらゆる初期化処理を行う。
     */
    public function boot(): void
    {
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
    }

パッケージの翻訳行は、`package::file.line`構文規則を使用して参照されます。したがって、`courier`パッケージの`messages`ファイルから`welcome`行を次のようにロードできます：

    echo trans('courier::messages.welcome');

`loadJsonTranslationsFrom`メソッドを使用して、パッケージのJSON翻訳ファイルを登録することもできます。このメソッドは、パッケージのJSON翻訳ファイルを含むディレクトリへのパスを受け取ります：

```php
/**
 * パッケージサービスのあらゆる初期化処理を行う。
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

<a name="publishing-language-files"></a>
#### 言語ファイルの公開

パッケージの言語ファイルをアプリケーションの`lang/vendor`ディレクトリに公開したい場合は、サービスプロバイダの`publishes`メソッドを使用できます。`publishes`メソッドは、パッケージパスとその希望する公開場所の配列を受け取ります。たとえば、`courier`パッケージの言語ファイルを公開するには、次のようにします：

    /**
     * パッケージサービスのあらゆる初期化処理を行う。
     */
    public function boot(): void
    {
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

        $this->publishes([
            __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
        ]);
    }

これで、パッケージのユーザーがLaravelの`vendor:publish` Artisanコマンドを実行すると、パッケージの言語ファイルが指定された公開場所に公開されます。

<a name="views"></a>
### ビュー

Laravelにパッケージの[ビュー](views.md)を登録するには、Laravelにビューがどこにあるかを教える必要があります。サービスプロバイダの`loadViewsFrom`メソッドを使用してこれを行うことができます。`loadViewsFrom`メソッドは、ビューテンプレートへのパスとパッケージの名前の2つの引数を受け取ります。例えば、パッケージの名前が`courier`の場合、サービスプロバイダの`boot`メソッドに以下を追加します:

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
    }

パッケージのビューは、`package::view`構文規則を使用して参照されます。したがって、サービスプロバイダでビューパスが登録されると、`courier`パッケージから`dashboard`ビューを次のようにロードできます:

    Route::get('/dashboard', function () {
        return view('courier::dashboard');
    });

<a name="overriding-package-views"></a>
#### パッケージビューのオーバーライド

`loadViewsFrom`メソッドを使用すると、Laravelは実際にビューの2つの場所を登録します: アプリケーションの`resources/views/vendor`ディレクトリと、指定したディレクトリです。したがって、`courier`パッケージを例にすると、Laravelはまず開発者によって`resources/views/vendor/courier`ディレクトリにカスタムバージョンのビューが配置されているかどうかを確認します。そして、ビューがカスタマイズされていない場合、Laravelは`loadViewsFrom`の呼び出しで指定したパッケージビューディレクトリを検索します。これにより、パッケージユーザーがパッケージのビューを簡単にカスタマイズ/オーバーライドできます。

<a name="publishing-views"></a>
#### ビューの公開

ビューをアプリケーションの`resources/views/vendor`ディレクトリに公開できるようにしたい場合は、サービスプロバイダの`publishes`メソッドを使用できます。`publishes`メソッドは、パッケージビューパスとその希望する公開場所の配列を受け取ります:

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
        ]);
    }

これで、パッケージのユーザーがLaravelの`vendor:publish` Artisanコマンドを実行すると、パッケージのビューが指定された公開場所にコピーされます。

<a name="view-components"></a>
### ビューコンポーネント

Bladeコンポーネントを利用するパッケージを構築している場合、またはコンポーネントを非標準的なディレクトリに配置している場合、Laravelがコンポーネントを見つける場所を知るために、コンポーネントクラスとそのHTMLタグエイリアスを手動で登録する必要があります。通常、パッケージのサービスプロバイダの`boot`メソッドでコンポーネントを登録する必要があります:

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }

コンポーネントが登録されると、そのタグエイリアスを使用してレンダリングできます:

```blade
<x-package-alert/>
```

<a name="autoloading-package-components"></a>
#### パッケージコンポーネントの自動読み込み

または、`componentNamespace`メソッドを使用して、規約に従ってコンポーネントクラスを自動読み込みすることもできます。例えば、`Nightshade`パッケージには`Calendar`と`ColorPicker`コンポーネントがあり、これらは`Nightshade\Views\Components`名前空間内にあります:

    use Illuminate\Support\Facades\Blade;

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

これにより、`package-name::`構文を使用してベンダー名前空間でパッケージコンポーネントを使用できます:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Bladeは、コンポーネント名をパスカルケースにして、このコンポーネントにリンクされたクラスを自動的に検出します。サブディレクトリも「ドット」記法でサポートされています。

<a name="anonymous-components"></a>
#### 匿名コンポーネント

パッケージに匿名コンポーネントが含まれている場合、それらはパッケージの「ビュー」ディレクトリの`components`ディレクトリ内に配置する必要があります（[`loadViewsFrom`メソッド](#views)で指定されたとおり）。その後、パッケージのビュー名前空間をプレフィックスとして付けてレンダリングできます:

```blade
<x-courier::alert />
```

<a name="about-artisan-command"></a>
### "About" Artisanコマンド

Laravelの組み込み`about` Artisanコマンドは、アプリケーションの環境と設定の概要を提供します。パッケージは`AboutCommand`クラスを介してこのコマンドの出力に追加情報をプッシュできます。通常、この情報はパッケージサービスプロバイダの`boot`メソッドから追加されます:

    use Illuminate\Foundation\Console\AboutCommand;

    /**
     * アプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
    }

<a name="commands"></a>
## コマンド

パッケージのArtisanコマンドをLaravelに登録するには、`commands`メソッドを使用できます。このメソッドはコマンドクラス名の配列を期待します。コマンドが登録されると、[Artisan CLI](artisan.md)を使用して実行できます:

    use Courier\Console\Commands\InstallCommand;
    use Courier\Console\Commands\NetworkCommand;

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                InstallCommand::class,
                NetworkCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## 公開アセット

パッケージには、JavaScript、CSS、画像などのアセットが含まれている場合があります。これらのアセットをアプリケーションの`public`ディレクトリに公開するには、サービスプロバイダの`publishes`メソッドを使用します。この例では、関連するアセットのグループを簡単に公開できるように、`public`アセットグループタグも追加します:

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../public' => public_path('vendor/courier'),
        ], 'public');
    }

これで、パッケージのユーザーが`vendor:publish`コマンドを実行すると、アセットが指定された公開場所にコピーされます。パッケージが更新されるたびにアセットを上書きする必要があるため、`--force`フラグを使用できます:

```shell
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## ファイルグループの公開

パッケージのアセットとリソースのグループを個別に公開したい場合があります。例えば、パッケージの設定ファイルを公開させたいが、パッケージのアセットを公開させたくない場合があります。これを行うには、パッケージのサービスプロバイダから`publishes`メソッドを呼び出す際に「タグ」を付けます。例えば、`courier`パッケージのサービスプロバイダの`boot`メソッドで、2つの公開グループ（`courier-config`と`courier-migrations`）を定義します:

    /**
     * パッケージサービスのブートストラップ
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../config/package.php' => config_path('package.php')
        ], 'courier-config');

        $this->publishesMigrations([
            __DIR__.'/../database/migrations/' => database_path('migrations')
        ], 'courier-migrations');
    }

これで、ユーザーは`vendor:publish`コマンドを実行する際にタグを参照することで、これらのグループを個別に公開できます:

```shell
php artisan vendor:publish --tag=courier-config
```
