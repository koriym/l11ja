# Laravel Dusk

- [はじめに](#introduction)
- [インストール](#installation)
    - [ChromeDriverのインストール管理](#managing-chromedriver-installations)
    - [他のブラウザの使用](#using-other-browsers)
- [はじめる](#getting-started)
    - [テストの生成](#generating-tests)
    - [各テスト後のデータベースリセット](#resetting-the-database-after-each-test)
    - [テストの実行](#running-tests)
    - [環境の取り扱い](#environment-handling)
- [ブラウザの基本](#browser-basics)
    - [ブラウザの作成](#creating-browsers)
    - [ナビゲーション](#navigation)
    - [ブラウザウィンドウのサイズ変更](#resizing-browser-windows)
    - [ブラウザマクロ](#browser-macros)
    - [認証](#authentication)
    - [クッキー](#cookies)
    - [JavaScriptの実行](#executing-javascript)
    - [スクリーンショットの撮影](#taking-a-screenshot)
    - [コンソール出力をディスクに保存](#storing-console-output-to-disk)
    - [ページソースをディスクに保存](#storing-page-source-to-disk)
- [要素とのインタラクション](#interacting-with-elements)
    - [Duskセレクタ](#dusk-selectors)
    - [テキスト、値、属性](#text-values-and-attributes)
    - [フォームとのインタラクション](#interacting-with-forms)
    - [ファイルの添付](#attaching-files)
    - [ボタンの押下](#pressing-buttons)
    - [リンクのクリック](#clicking-links)
    - [キーボードの使用](#using-the-keyboard)
    - [マウスの使用](#using-the-mouse)
    - [JavaScriptダイアログ](#javascript-dialogs)
    - [インラインフレームとのインタラクション](#interacting-with-iframes)
    - [セレクタのスコープ](#scoping-selectors)
    - [要素の待機](#waiting-for-elements)
    - [要素を表示範囲にスクロール](#scrolling-an-element-into-view)
- [利用可能なアサーション](#available-assertions)
- [ページ](#pages)
    - [ページの生成](#generating-pages)
    - [ページの設定](#configuring-pages)
    - [ページへのナビゲーション](#navigating-to-pages)
    - [ショートハンドセレクタ](#shorthand-selectors)
    - [ページメソッド](#page-methods)
- [コンポーネント](#components)
    - [コンポーネントの生成](#generating-components)
    - [コンポーネントの使用](#using-components)
- [継続的インテグレーション](#continuous-integration)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)
    - [Chipper CI](#running-tests-on-chipper-ci)

<a name="introduction"></a>
## はじめに

[Laravel Dusk](https://github.com/laravel/dusk) は、表現力豊かで使いやすいブラウザ自動化およびテストAPIを提供します。デフォルトでは、DuskはローカルコンピュータにJDKやSeleniumをインストールする必要はありません。代わりに、Duskはスタンドアロンの[ChromeDriver](https://sites.google.com/chromium.org/driver)を使用します。ただし、他のSelenium互換のドライバを自由に使用することもできます。

<a name="installation"></a>
## インストール

始めるには、[Google Chrome](https://www.google.com/chrome)をインストールし、`laravel/dusk` Composer依存関係をプロジェクトに追加する必要があります:

```shell
composer require laravel/dusk --dev
```

> WARNING:  
> Duskのサービスプロバイダを手動で登録する場合、**本番環境では決して登録しないでください**。登録すると、任意のユーザーがアプリケーションに認証できるようになる可能性があります。

Duskパッケージをインストールした後、`dusk:install` Artisanコマンドを実行します。`dusk:install`コマンドは、`tests/Browser`ディレクトリ、Duskテストの例、およびオペレーティングシステム用のChrome Driverバイナリをインストールします:

```shell
php artisan dusk:install
```

次に、アプリケーションの`.env`ファイルで`APP_URL`環境変数を設定します。この値は、ブラウザでアプリケーションにアクセスするために使用するURLと一致する必要があります。

> NOTE:  
> ローカル開発環境を管理するために[Laravel Sail](sail.md)を使用している場合は、[Duskテストの設定と実行](sail.md#laravel-dusk)に関するSailのドキュメントも参照してください。

<a name="managing-chromedriver-installations"></a>
### ChromeDriverのインストール管理

Laravel Duskによって`dusk:install`コマンドでインストールされるものとは異なるバージョンのChromeDriverをインストールしたい場合は、`dusk:chrome-driver`コマンドを使用できます:

```shell
# オペレーティングシステム用の最新バージョンのChromeDriverをインストール...
php artisan dusk:chrome-driver

# オペレーティングシステム用の特定のバージョンのChromeDriverをインストール...
php artisan dusk:chrome-driver 86

# サポートされているすべてのOS用の特定のバージョンのChromeDriverをインストール...
php artisan dusk:chrome-driver --all

# オペレーティングシステムで検出されたChrome / Chromiumのバージョンに一致するChromeDriverをインストール...
php artisan dusk:chrome-driver --detect
```

> WARNING:  
> Duskは`chromedriver`バイナリが実行可能である必要があります。Duskの実行に問題がある場合は、次のコマンドを使用してバイナリが実行可能であることを確認してください: `chmod -R 0755 vendor/laravel/dusk/bin/`。

<a name="using-other-browsers"></a>
### 他のブラウザの使用

デフォルトでは、DuskはGoogle Chromeとスタンドアロンの[ChromeDriver](https://sites.google.com/chromium.org/driver)を使用してブラウザテストを実行します。ただし、独自のSeleniumサーバーを起動し、任意のブラウザに対してテストを実行することもできます。

始めるには、アプリケーションのベースDuskテストケースである`tests/DuskTestCase.php`ファイルを開きます。このファイル内で、`startChromeDriver`メソッドの呼び出しを削除します。これにより、Duskが自動的にChromeDriverを起動するのを停止します:

    /**
     * Duskテスト実行の準備。
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }

次に、`driver`メソッドを変更して、選択したURLとポートに接続できるようにします。さらに、WebDriverに渡すべき「desired capabilities」を変更することもできます:

    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * RemoteWebDriverインスタンスの作成。
     */
    protected function driver(): RemoteWebDriver
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## はじめる

<a name="generating-tests"></a>
### テストの生成

Duskテストを生成するには、`dusk:make` Artisanコマンドを使用します。生成されたテストは`tests/Browser`ディレクトリに配置されます:

```shell
php artisan dusk:make LoginTest
```

<a name="resetting-the-database-after-each-test"></a>
### 各テスト後のデータベースリセット

書くほとんどのテストは、アプリケーションのデータベースからデータを取得するページと対話します。ただし、Duskテストは`RefreshDatabase`トレイトを使用してはいけません。`RefreshDatabase`トレイトは、HTTPリクエスト全体で適用または利用できないデータベーストランザクションを利用します。代わりに、`DatabaseMigrations`トレイトと`DatabaseTruncation`トレイトの2つのオプションがあります。

<a name="reset-migrations"></a>
#### データベースマイグレーションの使用

`DatabaseMigrations`トレイトは、各テストの前にデータベースマイグレーションを実行します。ただし、各テストのたびにデータベーステーブルをドロップして再作成することは、通常、テーブルを切り捨てるよりも遅くなります:

===  "Pest"
```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

uses(DatabaseMigrations::class);

//
```

===  "PHPUnit"
```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```

> WARNING:  
> Duskテストを実行する際にSQLiteインメモリデータベースを使用することはできません。ブラウザは独自のプロセス内で実行されるため、他のプロセスのインメモリデータベースにアクセスできません。

<a name="reset-truncation"></a>
#### データベース切り捨ての使用

`DatabaseTruncation`トレイトは、最初のテストでデータベースをマイグレーションして、データベーステーブルが正しく作成されていることを確認します。ただし、後続のテストでは、データベーステーブルは単に切り捨てられます - すべてのデータベースマイグレーションを再実行するよりも高速化を提供します:

===  "Pest"
```php
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

uses(DatabaseTruncation::class);

//
```

===  "PHPUnit"
```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;

    //
}
```

デフォルトでは、このトレイトは`migrations`テーブルを除くすべてのテーブルを切り捨てます。切り捨てるテーブルをカスタマイズしたい場合は、テストクラスに`$tablesToTruncate`プロパティを定義できます:

> NOTE:  
> Pestを使用している場合、プロパティやメソッドはベースの`DuskTestCase`クラスまたはテストファイルが拡張するクラスに定義する必要があります。

    /**
     * 切り捨てるテーブルを示す。
     *
     * @var array
     */
    protected $tablesToTruncate = ['users'];

または、テストクラスに`$exceptTables`プロパティを定義して、切り捨てから除外するテーブルを指定することもできます:

    /**
     * 切り捨てから除外するテーブルを示す。
     *
     * @var array
     */
    protected $exceptTables = ['users'];

テーブルを切り捨てるデータベース接続を指定するには、テストクラスに`$connectionsToTruncate`プロパティを定義できます:

    /**
     * テーブルを切り捨てる接続を示す。
     *
     * @var array
     */
    protected $connectionsToTruncate = ['mysql'];

データベース切り捨ての前後にコードを実行したい場合は、テストクラスに`beforeTruncatingDatabase`または`afterTruncatingDatabase`メソッドを定義できます:

```php
/**
 * データベースが切り捨てられる前に行うべき作業を実行する。
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * データベースの切り捨てが完了した後に行うべき作業を実行する。
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

<a name="running-tests"></a>
### テストの実行

ブラウザテストを実行するには、`dusk` Artisanコマンドを実行します:

```shell
php artisan dusk
```

前回の`dusk`コマンド実行時にテストが失敗した場合、`dusk:fails`コマンドを使用して失敗したテストを最初に再実行することで時間を節約できます:

```shell
php artisan dusk:fails
```

`dusk`コマンドは、Pest / PHPUnitテストランナーが通常受け入れる引数を受け入れます。例えば、特定の[グループ](https://docs.phpunit.de/en/10.5/annotations.html#group)のテストのみを実行できます:

```shell
php artisan dusk --group=foo
```

> NOTE:  
> ローカル開発環境を管理するために[Laravel Sail](sail.md)を使用している場合は、Sailのドキュメントで[Duskテストの設定と実行](sail.md#laravel-dusk)に関する情報を確認してください。

<a name="manually-starting-chromedriver"></a>
#### ChromeDriverの手動起動

デフォルトでは、Duskは自動的にChromeDriverを起動しようとします。特定のシステムでこれが機能しない場合は、`dusk`コマンドを実行する前に手動でChromeDriverを起動することができます。ChromeDriverを手動で起動することを選択した場合は、`tests/DuskTestCase.php`ファイルの次の行をコメントアウトする必要があります:

```php
/**
 * Duskテスト実行の準備をする。
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

さらに、ChromeDriverを9515以外のポートで起動する場合は、同じクラスの`driver`メソッドを修正して正しいポートを反映する必要があります:

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * RemoteWebDriverインスタンスを作成する。
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```

<a name="environment-handling"></a>
### 環境の取り扱い

テスト実行時にDuskが独自の環境ファイルを使用するように強制するには、プロジェクトのルートに`.env.dusk.{environment}`ファイルを作成します。例えば、`local`環境から`dusk`コマンドを開始する場合は、`.env.dusk.local`ファイルを作成する必要があります。

テストを実行すると、Duskは`.env`ファイルをバックアップし、Dusk環境を`.env`に名前変更します。テストが完了すると、`.env`ファイルが復元されます。

<a name="browser-basics"></a>
## ブラウザの基本

<a name="creating-browsers"></a>
### ブラウザの作成

まず、アプリケーションにログインできることを確認するテストを書いてみましょう。テストを生成した後、ログインページに移動し、いくつかの資格情報を入力し、「ログイン」ボタンをクリックするように変更できます。ブラウザインスタンスを作成するには、Duskテスト内から`browse`メソッドを呼び出します:

===  "Pest"
```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

uses(DatabaseMigrations::class);

test('basic example', function () {
    $user = User::factory()->create([
        'email' => 'taylor@laravel.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Login')
                ->assertPathIs('/home');
    });
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * 基本的なブラウザテストの例。
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => 'taylor@laravel.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                    ->type('email', $user->email)
                    ->type('password', 'password')
                    ->press('Login')
                    ->assertPathIs('/home');
        });
    }
}
```

上記の例でわかるように、`browse`メソッドはクロージャを受け入れます。Duskによってブラウザインスタンスが自動的にこのクロージャに渡され、このオブジェクトはアプリケーションとの対話やアサーションを行うために使用される主要なオブジェクトです。

<a name="creating-multiple-browsers"></a>
#### 複数のブラウザの作成

場合によっては、テストを正しく実行するために複数のブラウザが必要になることがあります。例えば、websocketと対話するチャット画面をテストするために複数のブラウザが必要になることがあります。複数のブラウザを作成するには、`browse`メソッドに渡されるクロージャのシグネチャにブラウザ引数を追加するだけです:

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
          ->visit('/home')
          ->waitForText('Message');

    $second->loginAs(User::find(2))
           ->visit('/home')
           ->waitForText('Message')
           ->type('message', 'Hey Taylor')
           ->press('Send');

    $first->waitForText('Hey Taylor')
          ->assertSee('Jeffrey Way');
});
```

<a name="navigation"></a>
### ナビゲーション

`visit`メソッドを使用して、アプリケーション内の特定のURIに移動できます:

```php
$browser->visit('/login');
```

`visitRoute`メソッドを使用して、[名前付きルート](routing.md#named-routes)に移動できます:

```php
$browser->visitRoute($routeName, $parameters);
```

`back`メソッドと`forward`メソッドを使用して、「戻る」と「進む」を行うことができます:

```php
$browser->back();

$browser->forward();
```

`refresh`メソッドを使用して、ページを更新できます:

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>
### ブラウザウィンドウのサイズ変更

`resize`メソッドを使用して、ブラウザウィンドウのサイズを調整できます:

```php
$browser->resize(1920, 1080);
```

`maximize`メソッドを使用して、ブラウザウィンドウを最大化できます:

```php
$browser->maximize();
```

`fitContent`メソッドは、ブラウザウィンドウのサイズをコンテンツに合わせて調整します:

```php
$browser->fitContent();
```

テストが失敗した場合、Duskは自動的にブラウザをコンテンツに合わせてサイズ変更し、スクリーンショットを撮ります。この機能を無効にするには、テスト内で`disableFitOnFailure`メソッドを呼び出します:

```php
$browser->disableFitOnFailure();
```

`move`メソッドを使用して、ブラウザウィンドウを画面上の別の位置に移動できます:

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>
### ブラウザマクロ

さまざまなテストで再利用できるカスタムブラウザメソッドを定義したい場合は、`Browser`クラスの`macro`メソッドを使用できます。通常、このメソッドは[サービスプロバイダ](providers.md)の`boot`メソッドから呼び出す必要があります:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Duskのブラウザマクロを登録する。
     */
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

`macro`関数は、最初の引数として名前を受け取り、2番目の引数としてクロージャを受け取ります。マクロのクロージャは、`Browser`インスタンスでマクロをメソッドとして呼び出すと実行されます:

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
            ->scrollToElement('#credit-card-details')
            ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### 認証

多くの場合、認証が必要なページをテストすることになります。Duskの`loginAs`メソッドを使用することで、すべてのテストでアプリケーションのログイン画面と対話する必要を避けることができます。`loginAs`メソッドは、認証可能なモデルに関連付けられた主キーまたは認証可能なモデルインスタンスを受け取ります:

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
          ->visit('/home');
});
```

> WARNING:  
> `loginAs`メソッドを使用した後、ファイル内のすべてのテストでユーザーセッションが維持されます。

<a name="cookies"></a>
### クッキー

`cookie`メソッドを使用して、暗号化されたクッキーの値を取得または設定できます。デフォルトでは、Laravelによって作成されたすべてのクッキーは暗号化されます:

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

`plainCookie`メソッドを使用して、暗号化されていないクッキーの値を取得または設定できます:

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

`deleteCookie`メソッドを使用して、指定されたクッキーを削除できます:

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>
### JavaScriptの実行

`script`メソッドを使用して、ブラウザ内で任意のJavaScriptステートメントを実行できます:

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

<a name="taking-a-screenshot"></a>
### スクリーンショットの撮影

`screenshot`メソッドを使用して、スクリーンショットを撮り、指定されたファイル名で保存できます。すべてのスクリーンショットは、`tests/Browser/screenshots`ディレクトリに保存されます:

```php
$browser->screenshot('filename');
```

`responsiveScreenshots`メソッドは、さまざまなブレークポイントで一連のスクリーンショットを撮るために使用できます。

    $browser->responsiveScreenshots('filename');

`screenshotElement`メソッドは、ページ上の特定の要素のスクリーンショットを撮るために使用できます。

    $browser->screenshotElement('#selector', 'filename');

<a name="storing-console-output-to-disk"></a>
### コンソール出力をディスクに保存

`storeConsoleLog`メソッドを使用して、現在のブラウザのコンソール出力を指定されたファイル名でディスクに書き込むことができます。コンソール出力は`tests/Browser/console`ディレクトリ内に保存されます。

    $browser->storeConsoleLog('filename');

<a name="storing-page-source-to-disk"></a>
### ページソースをディスクに保存

`storeSource`メソッドを使用して、現在のページのソースを指定されたファイル名でディスクに書き込むことができます。ページソースは`tests/Browser/source`ディレクトリ内に保存されます。

    $browser->storeSource('filename');

<a name="interacting-with-elements"></a>
## 要素とのインタラクション

<a name="dusk-selectors"></a>
### Duskセレクタ

要素とのインタラクションに適したCSSセレクタを選択することは、Duskテストを書く上で最も難しい部分の一つです。時間が経つにつれて、フロントエンドの変更により、以下のようなCSSセレクタがテストを壊す可能性があります。

    // HTML...

    <button>Login</button>

    // テスト...

    $browser->click('.login-page .container div > button');

Duskセレクタを使用すると、CSSセレクタを覚える代わりに効果的なテストを書くことに集中できます。セレクタを定義するには、HTML要素に`dusk`属性を追加します。そして、Duskブラウザとインタラクションする際に、セレクタに`@`を付けてテスト内の添付された要素を操作します。

    // HTML...

    <button dusk="login-button">Login</button>

    // テスト...

    $browser->click('@login-button');

必要に応じて、Duskセレクタが使用するHTML属性を`selectorHtmlAttribute`メソッドでカスタマイズできます。通常、このメソッドはアプリケーションの`AppServiceProvider`の`boot`メソッドから呼び出すべきです。

    use Laravel\Dusk\Dusk;

    Dusk::selectorHtmlAttribute('data-dusk');

<a name="text-values-and-attributes"></a>
### テキスト、値、属性

<a name="retrieving-setting-values"></a>
#### 値の取得と設定

Duskは、ページ上の要素の現在の値、表示テキスト、属性とのインタラクションのためのいくつかのメソッドを提供します。例えば、指定されたCSSまたはDuskセレクタに一致する要素の「値」を取得するには、`value`メソッドを使用します。

    // 値を取得...
    $value = $browser->value('selector');

    // 値を設定...
    $browser->value('selector', 'value');

指定されたフィールド名を持つ入力要素の「値」を取得するには、`inputValue`メソッドを使用できます。

    $value = $browser->inputValue('field');

<a name="retrieving-text"></a>
#### テキストの取得

`text`メソッドは、指定されたセレクタに一致する要素の表示テキストを取得するために使用できます。

    $text = $browser->text('selector');

<a name="retrieving-attributes"></a>
#### 属性の取得

最後に、`attribute`メソッドは、指定されたセレクタに一致する要素の属性の値を取得するために使用できます。

    $attribute = $browser->attribute('selector', 'value');

<a name="interacting-with-forms"></a>
### フォームとのインタラクション

<a name="typing-values"></a>
#### 値の入力

Duskは、フォームと入力要素とのインタラクションのためのさまざまなメソッドを提供します。まず、入力フィールドにテキストを入力する例を見てみましょう。

    $browser->type('email', 'taylor@laravel.com');

`type`メソッドにCSSセレクタを必ずしも渡す必要はないことに注意してください。CSSセレクタが提供されない場合、Duskは指定された`name`属性を持つ`input`または`textarea`フィールドを検索します。

フィールドの内容をクリアせずにテキストを追加するには、`append`メソッドを使用できます。

    $browser->type('tags', 'foo')
            ->append('tags', ', bar, baz');

入力の値をクリアするには、`clear`メソッドを使用します。

    $browser->clear('email');

`typeSlowly`メソッドを使用して、Duskにゆっくりと入力させることができます。デフォルトでは、Duskはキー入力間に100ミリ秒待機します。キー入力間の時間をカスタマイズするには、メソッドの3番目の引数として適切なミリ秒数を渡すことができます。

    $browser->typeSlowly('mobile', '+1 (202) 555-5555');

    $browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);

ゆっくりとテキストを追加するには、`appendSlowly`メソッドを使用できます。

    $browser->type('tags', 'foo')
            ->appendSlowly('tags', ', bar, baz');

<a name="dropdowns"></a>
#### ドロップダウン

`select`要素で利用可能な値を選択するには、`select`メソッドを使用できます。`type`メソッドと同様に、`select`メソッドは完全なCSSセレクタを必要としません。`select`メソッドに値を渡す際には、表示テキストではなく、基礎となるオプションの値を渡すべきです。

    $browser->select('size', 'Large');

2番目の引数を省略することで、ランダムなオプションを選択できます。

    $browser->select('size');

`select`メソッドの2番目の引数として配列を渡すことで、複数のオプションを選択するように指示できます。

    $browser->select('categories', ['Art', 'Music']);

<a name="checkboxes"></a>
#### チェックボックス

チェックボックス入力を「チェック」するには、`check`メソッドを使用できます。他の多くの入力関連メソッドと同様に、完全なCSSセレクタは必要ありません。CSSセレクタの一致が見つからない場合、Duskは一致する`name`属性を持つチェックボックスを検索します。

    $browser->check('terms');

チェックボックス入力の「チェックを外す」には、`uncheck`メソッドを使用できます。

    $browser->uncheck('terms');

<a name="radio-buttons"></a>
#### ラジオボタン

`radio`入力オプションを「選択」するには、`radio`メソッドを使用できます。他の多くの入力関連メソッドと同様に、完全なCSSセレクタは必要ありません。CSSセレクタの一致が見つからない場合、Duskは一致する`name`と`value`属性を持つ`radio`入力を検索します。

    $browser->radio('size', 'large');

<a name="attaching-files"></a>
### ファイルの添付

`attach`メソッドは、`file`入力要素にファイルを添付するために使用できます。他の多くの入力関連メソッドと同様に、完全なCSSセレクタは必要ありません。CSSセレクタの一致が見つからない場合、Duskは一致する`name`属性を持つ`file`入力を検索します。

    $browser->attach('photo', __DIR__.'/photos/mountains.png');

> WARNING:  
> attach関数を使用するには、`Zip` PHP拡張機能がサーバーにインストールされ、有効になっている必要があります。

<a name="pressing-buttons"></a>
### ボタンの押下

`press`メソッドは、ページ上のボタン要素をクリックするために使用できます。`press`メソッドに渡される引数は、ボタンの表示テキストまたはCSS/Duskセレクタのいずれかです。

    $browser->press('Login');

フォームを送信する際、多くのアプリケーションはボタンを押された後にボタンを無効にし、フォーム送信のHTTPリクエストが完了したときにボタンを再度有効にします。ボタンを押してボタンが再び有効になるのを待つには、`pressAndWaitFor`メソッドを使用できます。

    // ボタンを押して、最大5秒間有効になるのを待つ...
    $browser->pressAndWaitFor('Save');

    // ボタンを押して、最大1秒間有効になるのを待つ...
    $browser->pressAndWaitFor('Save', 1);

<a name="clicking-links"></a>
### リンクのクリック

リンクをクリックするには、ブラウザインスタンスで`clickLink`メソッドを使用できます。`clickLink`メソッドは、指定された表示テキストを持つリンクをクリックします。

    $browser->clickLink($linkText);

指定された表示テキストを持つリンクがページ上に表示されているかどうかを判断するには、`seeLink`メソッドを使用できます。

    if ($browser->seeLink($linkText)) {
        // ...
    }

> WARNING:  
> これらのメソッドはjQueryとインタラクションします。ページ上でjQueryが利用できない場合、Duskはテストの期間中、自動的にページに注入します。

<a name="using-the-keyboard"></a>
### キーボードの使用

`keys`メソッドを使用すると、`type`メソッドで通常許可されるよりも複雑な入力シーケンスを指定された要素に提供できます。例えば、修飾キーを押しながら値を入力するようにDuskに指示できます。この例では、`shift`キーが押された状態で`taylor`が指定されたセレクタに一致する要素に入力されます。`taylor`が入力された後、`swift`が修飾キーなしで入力されます。

    $browser->keys('selector', ['{shift}', 'taylor'], 'swift');

`keys`メソッドのもう一つの価値ある使用例は、アプリケーションのプライマリCSSセレクタに「キーボードショートカット」の組み合わせを送信することです。

    $browser->keys('.app', ['{command}', 'j']);

> NOTE:  
> すべての修飾キー（例：`{command}`）は`{}`文字で囲まれ、`Facebook\WebDriver\WebDriverKeys`クラスで定義された定数に一致します。これは[GitHubで見つける](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php)ことができます。

<a name="fluent-keyboard-interactions"></a>
#### フルエントなキーボードインタラクション

Duskはまた、`Laravel\Dusk\Keyboard`クラスを介して複雑なキーボードインタラクションをフルエントに実行できる`withKeyboard`メソッドも提供します。`Keyboard`クラスは、`press`、`release`、`type`、および`pause`メソッドを提供します。

    use Laravel\Dusk\Keyboard;

    $browser->withKeyboard(function (Keyboard $keyboard) {
        $keyboard->press('c')
            ->pause(1000)
            ->release('c')
            ->type(['c', 'e', 'o']);
    });

<a name="keyboard-macros"></a>
#### キーボードマクロ

テストスイート全体で簡単に再利用できるカスタムキーボードインタラクションを定義したい場合は、`Keyboard`クラスが提供する`macro`メソッドを使用できます。通常、このメソッドは[サービスプロバイダ](providers.md)の`boot`メソッドから呼び出すべきです。

    <?php

    namespace App\Providers;

    use Facebook\WebDriver\WebDriverKeys;
    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Keyboard;
    use Laravel\Dusk\OperatingSystem;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 全アプリケーションサービスの初期化処理
         *
         * @return void
         */
        public function boot()
        {
            Keyboard::macro('openSearch', function () {
                $this->press(WebDriverKeys::CONTROL)
                    ->press('f')
                    ->release(WebDriverKeys::CONTROL)
                    ->release('f');
            });
        }
    }

```php
class DuskServiceProvider extends ServiceProvider
{
    /**
     * Duskのブラウザマクロを登録する
     */
    public function boot(): void
    {
        Keyboard::macro('copy', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'c',
            ]);

            return $this;
        });

        Keyboard::macro('paste', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'v',
            ]);

            return $this;
        });
    }
}
```

`macro`関数は、最初の引数として名前を、2番目の引数としてクロージャを受け取ります。マクロのクロージャは、`Keyboard`インスタンスにメソッドとしてマクロを呼び出したときに実行されます。

```php
$browser->click('@textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
    ->click('@another-textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());
```

<a name="using-the-mouse"></a>
### マウスの使用

<a name="clicking-on-elements"></a>
#### 要素のクリック

`click`メソッドは、指定されたCSSまたはDuskセレクタに一致する要素をクリックするために使用できます。

```php
$browser->click('.selector');
```

`clickAtXPath`メソッドは、指定されたXPath式に一致する要素をクリックするために使用できます。

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

`clickAtPoint`メソッドは、ブラウザの表示可能領域を基準とした指定された座標の最上位の要素をクリックするために使用できます。

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

`doubleClick`メソッドは、マウスのダブルクリックをシミュレートするために使用できます。

```php
$browser->doubleClick();
$browser->doubleClick('.selector');
```

`rightClick`メソッドは、マウスの右クリックをシミュレートするために使用できます。

```php
$browser->rightClick();
$browser->rightClick('.selector');
```

`clickAndHold`メソッドは、マウスボタンがクリックされて押され続けることをシミュレートするために使用できます。後続の`releaseMouse`メソッドの呼び出しは、この動作を元に戻し、マウスボタンを解放します。

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
        ->pause(1000)
        ->releaseMouse();
```

`controlClick`メソッドは、ブラウザ内で`ctrl+click`イベントをシミュレートするために使用できます。

```php
$browser->controlClick();
$browser->controlClick('.selector');
```

<a name="mouseover"></a>
#### マウスオーバー

`mouseover`メソッドは、指定されたCSSまたはDuskセレクタに一致する要素の上にマウスを移動する必要がある場合に使用できます。

```php
$browser->mouseover('.selector');
```

<a name="drag-drop"></a>
#### ドラッグアンドドロップ

`drag`メソッドは、指定されたセレクタに一致する要素を別の要素にドラッグするために使用できます。

```php
$browser->drag('.from-selector', '.to-selector');
```

または、要素を単一の方向にドラッグすることもできます。

```php
$browser->dragLeft('.selector', $pixels = 10);
$browser->dragRight('.selector', $pixels = 10);
$browser->dragUp('.selector', $pixels = 10);
$browser->dragDown('.selector', $pixels = 10);
```

最後に、指定されたオフセットで要素をドラッグすることもできます。

```php
$browser->dragOffset('.selector', $x = 10, $y = 10);
```

<a name="javascript-dialogs"></a>
### JavaScriptダイアログ

Duskは、JavaScriptダイアログと対話するためのさまざまなメソッドを提供します。例えば、`waitForDialog`メソッドを使用して、JavaScriptダイアログが表示されるのを待つことができます。このメソッドは、ダイアログが表示されるまで待機する秒数を示すオプションの引数を受け取ります。

```php
$browser->waitForDialog($seconds = null);
```

`assertDialogOpened`メソッドは、ダイアログが表示され、指定されたメッセージを含むことをアサートするために使用できます。

```php
$browser->assertDialogOpened('Dialog message');
```

JavaScriptダイアログにプロンプトが含まれている場合、`typeInDialog`メソッドを使用してプロンプトに値を入力できます。

```php
$browser->typeInDialog('Hello World');
```

"OK"ボタンをクリックして開いているJavaScriptダイアログを閉じるには、`acceptDialog`メソッドを呼び出すことができます。

```php
$browser->acceptDialog();
```

"キャンセル"ボタンをクリックして開いているJavaScriptダイアログを閉じるには、`dismissDialog`メソッドを呼び出すことができます。

```php
$browser->dismissDialog();
```

<a name="interacting-with-iframes"></a>
### インラインフレームとの対話

iframe内の要素と対話する必要がある場合、`withinFrame`メソッドを使用できます。`withinFrame`メソッドに提供されたクロージャ内で行われるすべての要素操作は、指定されたiframeのコンテキストにスコープされます。

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242')
        ->type('input[name="exp-date"]', '1224')
        ->type('input[name="cvc"]', '123')
        ->press('Pay');
});
```

<a name="scoping-selectors"></a>
### セレクタのスコープ

特定のセレクタ内で複数の操作を実行したい場合があります。例えば、テーブル内にいくつかのテキストが存在することをアサートし、そのテーブル内のボタンをクリックしたい場合があります。`with`メソッドを使用してこれを実現できます。`with`メソッドに提供されたクロージャ内で行われるすべての操作は、元のセレクタにスコープされます。

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')
          ->clickLink('Delete');
});
```

現在のスコープ外でアサーションを実行する必要がある場合があります。これを実現するには、`elsewhere`および`elsewhereWhenAvailable`メソッドを使用できます。

```php
$browser->with('.table', function (Browser $table) {
    // 現在のスコープは `body .table`...

    $browser->elsewhere('.page-title', function (Browser $title) {
        // 現在のスコープは `body .page-title`...
        $title->assertSee('Hello World');
    });

    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        // 現在のスコープは `body .page-title`...
        $title->assertSee('Hello World');
    });
});
```

<a name="waiting-for-elements"></a>
### 要素の待機

JavaScriptを多用するアプリケーションをテストする場合、特定の要素やデータが利用可能になるまでテストを「待機」する必要があることがよくあります。Duskはこれを簡単にします。さまざまなメソッドを使用して、要素がページに表示されるのを待つか、指定されたJavaScript式が`true`に評価されるまで待つことができます。

<a name="waiting"></a>
#### 待機

指定されたミリ秒数だけテストを一時停止する必要がある場合は、`pause`メソッドを使用します。

```php
$browser->pause(1000);
```

指定された条件が`true`の場合にのみテストを一時停止する必要がある場合は、`pauseIf`メソッドを使用します。

```php
$browser->pauseIf(App::environment('production'), 1000);
```

同様に、指定された条件が`true`でない限りテストを一時停止する必要がある場合は、`pauseUnless`メソッドを使用できます。

```php
$browser->pauseUnless(App::environment('testing'), 1000);
```

<a name="waiting-for-selectors"></a>
#### セレクタの待機

`waitFor`メソッドは、指定されたCSSまたはDuskセレクタに一致する要素がページに表示されるまでテストの実行を一時停止するために使用できます。デフォルトでは、これによりテストは最大5秒間一時停止され、その後例外がスローされます。必要に応じて、メソッドの2番目の引数としてカスタムタイムアウトしきい値を渡すことができます。

```php
// セレクタを最大5秒間待機...
$browser->waitFor('.selector');

// セレクタを最大1秒間待機...
$browser->waitFor('.selector', 1);
```

指定されたセレクタに一致する要素が指定されたテキストを含むまで待機することもできます。

```php
// セレクタが指定されたテキストを含むまで最大5秒間待機...
$browser->waitForTextIn('.selector', 'Hello World');

// セレクタが指定されたテキストを含むまで最大1秒間待機...
$browser->waitForTextIn('.selector', 'Hello World', 1);
```

指定されたセレクタがページから欠落するまで待機することもできます。

```php
// セレクタが欠落するまで最大5秒間待機...
$browser->waitUntilMissing('.selector');

// セレクタが欠落するまで最大1秒間待機...
$browser->waitUntilMissing('.selector', 1);
```

または、指定されたセレクタが有効または無効になるまで待機することもできます。

```php
// セレクタが有効になるまで最大5秒間待機...
$browser->waitUntilEnabled('.selector');

// セレクタが有効になるまで最大1秒間待機...
$browser->waitUntilEnabled('.selector', 1);

// セレクタが無効になるまで最大5秒間待機...
$browser->waitUntilDisabled('.selector');

// セレクタが無効になるまで最大1秒間待機...
$browser->waitUntilDisabled('.selector', 1);
```

<a name="scoping-selectors-when-available"></a>
#### 利用可能なセレクタのスコープ

指定されたセレクタに一致する要素が表示されるのを待ち、その要素と対話する必要がある場合があります。例えば、モーダルウィンドウが利用可能になるのを待ち、そのモーダル内の"OK"ボタンを押したい場合があります。`whenAvailable`メソッドを使用してこれを実現できます。クロージャ内で行われるすべての要素操作は、元のセレクタにスコープされます。

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
          ->press('OK');
});
```

<a name="waiting-for-text"></a>
#### テキストの待機

`waitForText`メソッドは、指定されたテキストがページに表示されるまで待機するために使用できます。

```php
// テキストを最大5秒間待機...
$browser->waitForText('Hello World');

// テキストを最大1秒間待機...
$browser->waitForText('Hello World', 1);
```

表示されたテキストがページから削除されるまで待機するには、`waitUntilMissingText`メソッドを使用できます。

    $browser->waitUntilMissingText('Hello World');

    // テキストが削除されるまで最大5秒間待つ...
    $browser->waitUntilMissingText('Hello World');

    // テキストが削除されるまで最大1秒間待つ...
    $browser->waitUntilMissingText('Hello World', 1);

<a name="waiting-for-links"></a>
### リンクの待機

`waitForLink`メソッドは、指定されたリンクテキストがページに表示されるまで待機するために使用できます。

    // リンクが表示されるまで最大5秒間待つ...
    $browser->waitForLink('Create');

    // リンクが表示されるまで最大1秒間待つ...
    $browser->waitForLink('Create', 1);

<a name="waiting-for-inputs"></a>
### 入力の待機

`waitForInput`メソッドは、指定された入力フィールドがページに表示されるまで待機するために使用できます。

    // 入力が表示されるまで最大5秒間待つ...
    $browser->waitForInput($field);

    // 入力が表示されるまで最大1秒間待つ...
    $browser->waitForInput($field, 1);

<a name="waiting-on-the-page-location"></a>
### ページの場所の待機

`$browser->assertPathIs('/home')`のようなパスアサーションを行う場合、`window.location.pathname`が非同期に更新されているとアサーションが失敗することがあります。`waitForLocation`メソッドを使用して、場所が指定された値になるまで待機できます。

    $browser->waitForLocation('/secret');

`waitForLocation`メソッドは、現在のウィンドウの場所が完全修飾URLになるまで待機するためにも使用できます。

    $browser->waitForLocation('https://example.com/path');

また、[名前付きルート](routing.md#named-routes)の場所を待機することもできます。

    $browser->waitForRoute($routeName, $parameters);

<a name="waiting-for-page-reloads"></a>
### ページの再読み込みの待機

アクションを実行した後にページが再読み込みされるのを待つ必要がある場合は、`waitForReload`メソッドを使用します。

    use Laravel\Dusk\Browser;

    $browser->waitForReload(function (Browser $browser) {
        $browser->press('Submit');
    })
    ->assertSee('Success!');

ページの再読み込みを待つ必要があるのは通常、ボタンをクリックした後なので、便宜上`clickAndWaitForReload`メソッドを使用できます。

    $browser->clickAndWaitForReload('.selector')
            ->assertSee('something');

<a name="waiting-on-javascript-expressions"></a>
### JavaScript式の待機

特定のJavaScript式が`true`に評価されるまでテストの実行を一時停止したい場合があります。`waitUntil`メソッドを使用すると、これを簡単に実現できます。このメソッドに式を渡すときは、`return`キーワードや終了セミコロンを含める必要はありません。

    // 式がtrueになるまで最大5秒間待つ...
    $browser->waitUntil('App.data.servers.length > 0');

    // 式がtrueになるまで最大1秒間待つ...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="waiting-on-vue-expressions"></a>
### Vue式の待機

`waitUntilVue`および`waitUntilVueIsNot`メソッドは、[Vueコンポーネント](https://vuejs.org)の属性が指定された値になるまで待機するために使用できます。

    // コンポーネント属性が指定された値を含むまで待つ...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // コンポーネント属性が指定された値を含まなくなるまで待つ...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

<a name="waiting-for-javascript-events"></a>
### JavaScriptイベントの待機

`waitForEvent`メソッドは、JavaScriptイベントが発生するまでテストの実行を一時停止するために使用できます。

    $browser->waitForEvent('load');

イベントリスナーは現在のスコープにアタッチされます。デフォルトでは`body`要素です。スコープ付きセレクターを使用する場合、イベントリスナーは一致する要素にアタッチされます。

    $browser->with('iframe', function (Browser $iframe) {
        // iframeのloadイベントを待つ...
        $iframe->waitForEvent('load');
    });

`waitForEvent`メソッドの2番目の引数としてセレクターを指定することで、特定の要素にイベントリスナーをアタッチできます。

    $browser->waitForEvent('load', '.selector');

`document`および`window`オブジェクトのイベントを待機することもできます。

    // ドキュメントがスクロールされるまで待つ...
    $browser->waitForEvent('scroll', 'document');

    // ウィンドウがリサイズされるまで最大5秒間待つ...
    $browser->waitForEvent('resize', 'window', 5);

<a name="waiting-with-a-callback"></a>
### コールバックを使用した待機

Duskの多くの「待機」メソッドは、基礎となる`waitUsing`メソッドに依存しています。このメソッドを直接使用して、指定されたクロージャが`true`を返すまで待機できます。`waitUsing`メソッドは、待機する最大秒数、クロージャを評価する間隔、クロージャ、およびオプションの失敗メッセージを受け取ります。

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="scrolling-an-element-into-view"></a>
### 要素を表示領域にスクロール

ブラウザの表示領域外にあるために要素をクリックできない場合があります。`scrollIntoView`メソッドは、指定されたセレクターの要素が表示領域内になるまでブラウザウィンドウをスクロールします。

    $browser->scrollIntoView('.selector')
            ->click('.selector');

<a name="available-assertions"></a>
## 利用可能なアサーション

Duskは、アプリケーションに対して行うことができるさまざまなアサーションを提供します。以下のリストに、利用可能なすべてのアサーションを示します。

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathEndsWith](#assert-path-ends-with)
[assertPathContains](#assert-path-contains)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeContains](#assert-attribute-contains)
[assertAttributeDoesntContain](#assert-attribute-doesnt-contain)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesntContain](#assert-vue-doesnt-contain)

</div>

<a name="assert-title"></a>
#### assertTitle

ページタイトルが指定されたテキストと一致することをアサートします。

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>
#### assertTitleContains

ページタイトルに指定されたテキストが含まれることをアサートします。

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>
#### assertUrlIs

現在のURL（クエリ文字列を除く）が指定された文字列と一致することをアサートします。

    $browser->assertUrlIs($url);

<a name="assert-scheme-is"></a>
#### assertSchemeIs

現在のURLスキームが指定されたスキームと一致することをアサートします。

    $browser->assertSchemeIs($scheme);

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

現在のURLスキームが指定されたスキームと一致しないことをアサートします。

    $browser->assertSchemeIsNot($scheme);

<a name="assert-host-is"></a>
#### assertHostIs

現在のURLホストが指定されたホストと一致することをアサートします。

    $browser->assertHostIs($host);

<a name="assert-host-is-not"></a>
#### assertHostIsNot

現在のURLホストが指定されたホストと一致しないことをアサートします。

    $browser->assertHostIsNot($host);

<a name="assert-port-is"></a>
#### assertPortIs

現在のURLのポートが指定されたポートと一致することをアサートします:

    $browser->assertPortIs($port);

<a name="assert-port-is-not"></a>
#### assertPortIsNot

現在のURLのポートが指定されたポートと一致しないことをアサートします:

    $browser->assertPortIsNot($port);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

現在のURLのパスが指定されたパスで始まることをアサートします:

    $browser->assertPathBeginsWith('/home');

<a name="assert-path-ends-with"></a>
#### assertPathEndsWith

現在のURLのパスが指定されたパスで終わることをアサートします:

    $browser->assertPathEndsWith('/home');

<a name="assert-path-contains"></a>
#### assertPathContains

現在のURLのパスが指定されたパスを含むことをアサートします:

    $browser->assertPathContains('/home');

<a name="assert-path-is"></a>
#### assertPathIs

現在のパスが指定されたパスと一致することをアサートします:

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>
#### assertPathIsNot

現在のパスが指定されたパスと一致しないことをアサートします:

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>
#### assertRouteIs

現在のURLが指定された[名前付きルート](routing.md#named-routes)のURLと一致することをアサートします:

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

指定されたクエリ文字列パラメータが存在することをアサートします:

    $browser->assertQueryStringHas($name);

指定されたクエリ文字列パラメータが存在し、指定された値を持つことをアサートします:

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

指定されたクエリ文字列パラメータが存在しないことをアサートします:

    $browser->assertQueryStringMissing($name);

<a name="assert-fragment-is"></a>
#### assertFragmentIs

URLの現在のハッシュフラグメントが指定されたフラグメントと一致することをアサートします:

    $browser->assertFragmentIs('anchor');

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

URLの現在のハッシュフラグメントが指定されたフラグメントで始まることをアサートします:

    $browser->assertFragmentBeginsWith('anchor');

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

URLの現在のハッシュフラグメントが指定されたフラグメントと一致しないことをアサートします:

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

指定された暗号化されたクッキーが存在することをアサートします:

    $browser->assertHasCookie($name);

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

指定された暗号化されていないクッキーが存在することをアサートします:

    $browser->assertHasPlainCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

指定された暗号化されたクッキーが存在しないことをアサートします:

    $browser->assertCookieMissing($name);

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

指定された暗号化されていないクッキーが存在しないことをアサートします:

    $browser->assertPlainCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

暗号化されたクッキーが指定された値を持つことをアサートします:

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

暗号化されていないクッキーが指定された値を持つことをアサートします:

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>
#### assertSee

指定されたテキストがページ上に存在することをアサートします:

    $browser->assertSee($text);

<a name="assert-dont-see"></a>
#### assertDontSee

指定されたテキストがページ上に存在しないことをアサートします:

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>
#### assertSeeIn

指定されたテキストがセレクタ内に存在することをアサートします:

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

指定されたテキストがセレクタ内に存在しないことをアサートします:

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

セレクタ内に何らかのテキストが存在することをアサートします:

    $browser->assertSeeAnythingIn($selector);

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

セレクタ内にテキストが存在しないことをアサートします:

    $browser->assertSeeNothingIn($selector);

<a name="assert-script"></a>
#### assertScript

指定されたJavaScript式が指定された値に評価されることをアサートします:

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

<a name="assert-source-has"></a>
#### assertSourceHas

指定されたソースコードがページ上に存在することをアサートします:

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

指定されたソースコードがページ上に存在しないことをアサートします:

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

指定されたリンクがページ上に存在することをアサートします:

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

指定されたリンクがページ上に存在しないことをアサートします:

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

指定された入力フィールドが指定された値を持つことをアサートします:

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

指定された入力フィールドが指定された値を持たないことをアサートします:

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>
#### assertChecked

指定されたチェックボックスがチェックされていることをアサートします:

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

指定されたチェックボックスがチェックされていないことをアサートします:

    $browser->assertNotChecked($field);

<a name="assert-indeterminate"></a>
#### assertIndeterminate

指定されたチェックボックスが不定状態であることをアサートします:

    $browser->assertIndeterminate($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

指定されたラジオフィールドが選択されていることをアサートします:

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

指定されたラジオフィールドが選択されていないことをアサートします:

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

指定されたドロップダウンが指定された値を選択していることをアサートします:

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

指定されたドロップダウンが指定された値を選択していないことをアサートします:

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

指定された値の配列が選択可能であることをアサートします:

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

指定された値の配列が選択不可能であることをアサートします:

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

指定されたフィールドで指定された値が選択可能であることをアサートします:

    $browser->assertSelectHasOption($field, $value);

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

指定された値が選択不可能であることをアサートします:

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-value"></a>
#### assertValue

指定されたセレクタに一致する要素が指定された値を持つことをアサートします:

    $browser->assertValue($selector, $value);

<a name="assert-value-is-not"></a>
#### assertValueIsNot

指定されたセレクタに一致する要素が指定された値を持たないことをアサートします:

    $browser->assertValueIsNot($selector, $value);

<a name="assert-attribute"></a>
#### assertAttribute

指定されたセレクタに一致する要素が指定された属性に指定された値を持つことをアサートします:

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

指定されたセレクタに一致する要素が指定された属性に指定された値を含むことをアサートします:

    $browser->assertAttributeContains($selector, $attribute, $value);

<a name="assert-attribute-doesnt-contain"></a>
#### assertAttributeDoesntContain

指定されたセレクタに一致する要素が指定された属性に指定された値を含まないことをアサートします:

    $browser->assertAttributeDoesntContain($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

指定されたセレクタに一致する要素が指定されたaria属性に指定された値を持つことをアサートします:

    $browser->assertAriaAttribute($selector, $attribute, $value);

例えば、マークアップ `<button aria-label="Add"></button>` がある場合、`aria-label` 属性に対して次のようにアサートできます:

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>
#### assertDataAttribute

指定されたセレクタに一致する要素が指定されたデータ属性に指定された値を持つことをアサートします:

    $browser->assertDataAttribute($selector, $attribute, $value);

例えば、マークアップ `<tr id="row-1" data-content="attendees"></tr>` がある場合、`data-label` 属性に対して次のようにアサートできます:

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>
#### assertVisible

指定されたセレクタに一致する要素が表示されていることをアサートします:

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

指定されたセレクタに一致する要素がソース内に存在することをアサートします:

    $browser->assertPresent($selector);

<a name="assert-not-present"></a>
#### assertNotPresent

指定されたセレクタに一致する要素がソース内に存在しないことをアサートします:

    $browser->assertNotPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

指定されたセレクタに一致する要素が表示されていないことをアサートします:

    $browser->assertMissing($selector);

<a name="assert-input-present"></a>
#### assertInputPresent

指定された名前の入力が存在することをアサートします:

    $browser->assertInputPresent($name);

<a name="assert-input-missing"></a>
#### assertInputMissing

指定された名前の入力がソースに存在しないことをアサートします:

    $browser->assertInputMissing($name);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

指定されたメッセージのJavaScriptダイアログが開かれたことをアサートします:

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

指定されたフィールドが有効であることをアサートします:

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

指定されたフィールドが無効であることをアサートします:

    $browser->assertDisabled($field);

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

指定されたボタンが有効であることをアサートします:

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

指定されたボタンが無効であることをアサートします:

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>
#### assertFocused

指定されたフィールドがフォーカスされていることをアサートします:

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

指定されたフィールドがフォーカスされていないことをアサートします:

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>
#### assertAuthenticated

ユーザーが認証されていることをアサートします:

    $browser->assertAuthenticated();

<a name="assert-guest"></a>
#### assertGuest

ユーザーが認証されていないことをアサートします:

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

ユーザーが指定されたユーザーとして認証されていることをアサートします:

    $browser->assertAuthenticatedAs($user);

<a name="assert-vue"></a>
#### assertVue

Duskでは、[Vueコンポーネント](https://vuejs.org)の状態に対してアサーションを行うこともできます。例えば、アプリケーションに以下のVueコンポーネントが含まれているとします:

    // HTML...

    <profile dusk="profile-component"></profile>

    // コンポーネント定義...

    Vue.component('profile', {
        template: '<div>{{ user.name }}</div>',

        data: function () {
            return {
                user: {
                    name: 'Taylor'
                }
            };
        }
    });

Vueコンポーネントの状態に対してアサーションを行うことができます:

===  "Pest"
```php
test('vue', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertVue('user.name', 'Taylor', '@profile-component');
    });
});
```

===  "PHPUnit"
```php
/**
 * Vueテストの基本例.
 */
public function test_vue(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}
```

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

指定されたVueコンポーネントのデータプロパティが指定された値と一致しないことをアサートします:

    $browser->assertVueIsNot($property, $value, $componentSelector = null);

<a name="assert-vue-contains"></a>
#### assertVueContains

指定されたVueコンポーネントのデータプロパティが配列であり、指定された値を含むことをアサートします:

    $browser->assertVueContains($property, $value, $componentSelector = null);

<a name="assert-vue-doesnt-contain"></a>
#### assertVueDoesntContain

指定されたVueコンポーネントのデータプロパティが配列であり、指定された値を含まないことをアサートします:

    $browser->assertVueDoesntContain($property, $value, $componentSelector = null);

<a name="pages"></a>
## ページ

テストによっては、いくつかの複雑なアクションを順番に実行する必要がある場合があります。これにより、テストが読みにくく、理解しにくくなる可能性があります。Duskのページを使用すると、単一のメソッドで実行できる表現力豊かなアクションを定義できます。ページでは、アプリケーションまたは単一のページの一般的なセレクターへのショートカットを定義することもできます。

<a name="generating-pages"></a>
### ページの生成

ページオブジェクトを生成するには、`dusk:page` Artisanコマンドを実行します。すべてのページオブジェクトは、アプリケーションの`tests/Browser/Pages`ディレクトリに配置されます:

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### ページの設定

デフォルトでは、ページには`url`、`assert`、`elements`の3つのメソッドがあります。ここでは、`url`と`assert`メソッドについて説明します。`elements`メソッドについては、[後ほど詳しく説明します](#shorthand-selectors)。

<a name="the-url-method"></a>
#### `url`メソッド

`url`メソッドは、ページを表すURLのパスを返す必要があります。Duskは、ブラウザでこのURLに移動するときにこのURLを使用します:

    /**
     * ページのURLを取得します。
     */
    public function url(): string
    {
        return '/login';
    }

<a name="the-assert-method"></a>
#### `assert`メソッド

`assert`メソッドは、ブラウザが実際に指定されたページにあることを確認するために必要なアサーションを行うことができます。このメソッドに何も配置する必要はありませんが、必要に応じてこれらのアサーションを自由に行うことができます。これらのアサーションは、ページに移動すると自動的に実行されます:

    /**
     * ブラウザがページにあることをアサートします。
     */
    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### ページへの移動

ページが定義されたら、`visit`メソッドを使用してそのページに移動できます:

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

場合によっては、特定のページに既にいて、そのページのセレクタとメソッドを現在のテストコンテキストに「ロード」する必要があります。これは、ボタンを押して明示的に移動することなく特定のページにリダイレクトされる場合によくあります。この状況では、`on`メソッドを使用してページをロードできます:

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### ショートハンドセレクタ

ページクラス内の`elements`メソッドを使用すると、ページ上の任意のCSSセレクタに対して、簡単に覚えやすいショートカットを定義できます。例えば、アプリケーションのログインページの「email」入力フィールドのショートカットを定義しましょう:

    /**
     * ページの要素ショートカットを取得します。
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

ショートカットが定義されると、通常のCSSセレクタを使用する場所ならどこでもショートハンドセレクタを使用できます:

    $browser->type('@email', 'taylor@laravel.com');

<a name="global-shorthand-selectors"></a>
#### グローバルショートハンドセレクタ

Duskをインストールすると、ベースの`Page`クラスが`tests/Browser/Pages`ディレクトリに配置されます。このクラスには、アプリケーション全体で利用可能なグローバルショートハンドセレクタを定義するために使用できる`siteElements`メソッドが含まれています:

    /**
     * サイトのグローバル要素ショートカットを取得します。
     *
     * @return array<string, string>
     */
    public static function siteElements(): array
    {
        return [
            '@element' => '#selector',
        ];
    }

<a name="page-methods"></a>
### ページメソッド

ページには、デフォルトで定義されているメソッドに加えて、テスト全体で使用できる追加のメソッドを定義することもできます。例えば、音楽管理アプリケーションを構築しているとします。アプリケーションの1つのページで一般的なアクションは、プレイリストを作成することです。各テストでプレイリストを作成するロジックを繰り返し書く代わりに、ページクラスに`createPlaylist`メソッドを定義できます:

    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Page;

    class Dashboard extends Page
    {
        // その他のページメソッド...

        /**
         * 新しいプレイリストを作成します。
         */
        public function createPlaylist(Browser $browser, string $name): void
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }

メソッドが定義されると、ページを利用する任意のテストでそれを使用できます。ブラウザインスタンスは、カスタムページメソッドに自動的に最初の引数として渡されます:

    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');

<a name="components"></a>
## コンポーネント

コンポーネントはDuskの「ページオブジェクト」に似ていますが、アプリケーション全体で再利用されるUIや機能のピース、例えばナビゲーションバーや通知ウィンドウに対して意図されています。そのため、コンポーネントは特定のURLにバインドされません。

<a name="generating-components"></a>
### コンポーネントの生成

コンポーネントを生成するには、`dusk:component` Artisanコマンドを実行します。新しいコンポーネントは、`tests/Browser/Components`ディレクトリに配置されます:

    php artisan dusk:component DatePicker

上記のように、「日付ピッカー」は、アプリケーション全体のさまざまなページに存在する可能性のあるコンポーネントの例です。数十のテストで日付を選択するためのブラウザ自動化ロジックを手動で書くのは面倒です。代わりに、Duskコンポーネントを定義して、そのロジックをコンポーネント内にカプセル化することができます:

    <?php

    namespace Tests\Browser\Components;

    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Component as BaseComponent;

    class DatePicker extends BaseComponent
    {
        /**
         * コンポーネントのルートセレクタを取得します。
         */
        public function selector(): string
        {
            return '.date-picker';
        }

        /**
         * ブラウザページにコンポーネントが含まれていることをアサートします。
         */
        public function assert(Browser $browser): void
        {
            $browser->assertVisible($this->selector());
        }
    
        /**
         * コンポーネントの要素ショートカットを取得します。
         *
         * @return array<string, string>
         */
        public function elements(): array
        {
            return [
                '@date-field' => 'input.datepicker-input',
                '@year-list' => 'div > div.datepicker-years',
                '@month-list' => 'div > div.datepicker-months',
                '@day-list' => 'div > div.datepicker-days',
            ];
        }

        /**
         * 指定された日付を選択します。
         */
        public function selectDate(Browser $browser, int $year, int $month, int $day): void
        {
            $browser->click('@date-field')
                    ->within('@year-list', function (Browser $browser) use ($year) {
                        $browser->click($year);
                    })
                    ->within('@month-list', function (Browser $browser) use ($month) {
                        $browser->click($month);
                    })
                    ->within('@day-list', function (Browser $browser) use ($day) {
                        $browser->click($day);
                    });
        }
    }

<a name="using-components"></a>
### コンポーネントの使用

コンポーネントが定義されたら、テスト内で日付ピッカーから簡単に日付を選択できます。また、日付選択に必要なロジックが変更された場合、コンポーネントのみを更新する必要があります。

===  "Pest"
```php
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;

uses(DatabaseMigrations::class);

test('basic example', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->within(new DatePicker, function (Browser $browser) {
                    $browser->selectDate(2019, 1, 30);
                })
                ->assertSee('January');
    });
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * 基本的なコンポーネントテストの例。
     */
    public function test_basic_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->within(new DatePicker, function (Browser $browser) {
                        $browser->selectDate(2019, 1, 30);
                    })
                    ->assertSee('January');
        });
    }
}
```

<a name="continuous-integration"></a>
## 継続的インテグレーション

> WARNING:  
> ほとんどのDusk継続的インテグレーションの設定では、Laravelアプリケーションが組み込みのPHP開発サーバーを使用してポート8000で提供されることを期待しています。したがって、続行する前に、継続的インテグレーション環境に`APP_URL`環境変数の値が`http://127.0.0.1:8000`であることを確認する必要があります。

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

[Heroku CI](https://www.heroku.com/continuous-integration)でDuskテストを実行するには、以下のGoogle ChromeビルドパックとスクリプトをHerokuの`app.json`ファイルに追加します。

```json
{
  "environments": {
    "test": {
      "buildpacks": [
        { "url": "heroku/php" },
        { "url": "https://github.com/heroku/heroku-buildpack-chrome-for-testing" }
      ],
      "scripts": {
        "test-setup": "cp .env.testing .env",
        "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux --port=9515 > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
      }
    }
  }
}
```

<a name="running-tests-on-travis-ci"></a>
### Travis CI

[Travis CI](https://travis-ci.org)でDuskテストを実行するには、以下の`.travis.yml`設定を使用します。Travis CIはグラフィカル環境ではないため、Chromeブラウザを起動するためにいくつかの追加手順が必要です。さらに、`php artisan serve`を使用してPHPの組み込みWebサーバーを起動します。

```yaml
language: php

php:
  - 8.2

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

[GitHub Actions](https://github.com/features/actions)を使用してDuskテストを実行する場合、以下の設定ファイルを出発点として使用できます。TravisCIと同様に、`php artisan serve`コマンドを使用してPHPの組み込みWebサーバーを起動します。

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v4
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux --port=9515 &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: console
          path: tests/Browser/console
```

<a name="running-tests-on-chipper-ci"></a>
### Chipper CI

[Chipper CI](https://chipperci.com)を使用してDuskテストを実行する場合、以下の設定ファイルを出発点として使用できます。PHPの組み込みサーバーを使用してLaravelを実行し、リクエストをリッスンします。

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# Include Chrome in the build environment
services:
  - dusk

# Build all commits
on:
   push:
      branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # Create a dusk env file, ensuring APP_URL uses BUILD_HOST
      cp -v .env .env.dusk.ci
      sed -i "s@APP_URL=.*@APP_URL=http://$BUILD_HOST:8000@g" .env.dusk.ci

  - name: Compile Assets
    cmd: |
      npm ci --no-audit
      npm run build

  - name: Browser Tests
    cmd: |
      php -S [::0]:8000 -t public 2>server.log &
      sleep 2
      php artisan dusk:chrome-driver $CHROME_DRIVER
      php artisan dusk --env=ci
```

Chipper CIでDuskテストを実行する方法、特にデータベースの使用方法について詳しく知るには、[公式のChipper CIドキュメント](https://chipperci.com/docs/testing/laravel-dusk-new/)を参照してください。

