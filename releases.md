# リリースノート

- [バージョン管理方式](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 11](#laravel-11)

<a name="versioning-scheme"></a>
## バージョン管理方式

Laravelとその他のファーストパーティパッケージは、[セマンティックバージョニング](https://semver.org)に従います。メジャーフレームワークリリースは毎年（〜第1四半期）リリースされ、マイナーおよびパッチリリースは毎週のように頻繁にリリースされる可能性があります。マイナーおよびパッチリリースには、**決して**破壊的変更を含めるべきではありません。

アプリケーションまたはパッケージからLaravelフレームワークまたはそのコンポーネントを参照する場合、Laravelのメジャーリリースには破壊的変更が含まれるため、常に`^11.0`のようなバージョン制約を使用する必要があります。ただし、新しいメジャーリリースへのアップグレードを1日以内に完了できるように努めています。

<a name="named-arguments"></a>
#### 名前付き引数

[名前付き引数](https://www.php.net/manual/ja/functions.arguments.php#functions.named-arguments)は、Laravelの後方互換性ガイドラインの対象外です。必要に応じて、Laravelコードベースを改善するために関数引数の名前を変更することを選択する場合があります。したがって、Laravelメソッドを呼び出す際に名前付き引数を使用する場合は、慎重に行い、パラメータ名が将来変更される可能性があることを理解しておく必要があります。

<a name="support-policy"></a>
## サポートポリシー

すべてのLaravelリリースについて、バグ修正は18ヶ月間、セキュリティ修正は2年間提供されます。Lumenを含むすべての追加ライブラリについては、最新のメジャーリリースのみがバグ修正を受け取ります。さらに、Laravelでサポートされているデータベースバージョンについては、[Laravelのデータベースサポート](database.md#introduction)を確認してください。

<div class="overflow-auto" markdown=1>

| バージョン | PHP (*) | リリース日 | バグ修正期限 | セキュリティ修正期限 |
| --- | --- | --- | --- | --- |
| 9 | 8.0 - 8.2 | 2022年2月8日 | 2023年8月8日 | 2024年2月6日 |
| 10 | 8.1 - 8.3 | 2023年2月14日 | 2024年8月6日 | 2025年2月4日 |
| 11 | 8.2 - 8.3 | 2024年3月12日 | 2025年9月3日 | 2026年3月12日 |
| 12 | 8.2 - 8.3 | 2025年第1四半期 | 2026年第3四半期 | 2027年第1四半期 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>サポート終了</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>セキュリティ修正のみ</div>
    </div>
</div>

(*) サポートされているPHPバージョン

<a name="laravel-11"></a>
## Laravel 11

Laravel 11は、Laravel 10.xで行われた改善を継続し、合理化されたアプリケーション構造、秒単位のレートリミット、ヘルスルーティング、エレガントな暗号化キーのローテーション、キューテストの改善、[Resend](https://resend.com)メールトランスポート、Promptバリデータの統合、新しいArtisanコマンドなどを導入しています。さらに、Laravel ReverbというファーストパーティのスケーラブルなWebSocketサーバが導入され、アプリケーションに堅牢なリアルタイム機能を提供します。

<a name="php-8"></a>
### PHP 8.2

Laravel 11.xは、最低限必要なPHPバージョンが8.2です。

<a name="structure"></a>
### 合理化されたアプリケーション構造

_Laravelの合理化されたアプリケーション構造は、[Taylor Otwell](https://github.com/taylorotwell)と[Nuno Maduro](https://github.com/nunomaduro)によって開発されました。_

Laravel 11では、既存のアプリケーションに変更を加えることなく、**新しい**Laravelアプリケーションに対して合理化されたアプリケーション構造が導入されています。新しいアプリケーション構造は、より軽量でモダンな体験を提供することを目的としており、Laravel開発者がすでに慣れ親しんでいる多くの概念を保持しています。以下では、Laravelの新しいアプリケーション構造のハイライトについて説明します。

#### アプリケーションブートストラップファイル

`bootstrap/app.php`ファイルは、コードファーストのアプリケーション設定ファイルとして復活しました。このファイルから、アプリケーションのルーティング、ミドルウェア、サービスプロバイダ、例外処理などをカスタマイズできます。このファイルは、以前はアプリケーションのファイル構造全体に散在していたさまざまな高レベルのアプリケーション動作設定を統一します。

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

<a name="service-providers"></a>
#### サービスプロバイダ

デフォルトのLaravelアプリケーション構造に含まれていた5つのサービスプロバイダの代わりに、Laravel 11には`AppServiceProvider`が1つだけ含まれています。以前のサービスプロバイダの機能は、`bootstrap/app.php`に組み込まれ、フレームワークによって自動的に処理されるか、アプリケーションの`AppServiceProvider`に配置されます。

たとえば、イベントディスカバリーはデフォルトで有効になっており、イベントとそのリスナーを手動で登録する必要がほとんどなくなりました。ただし、イベントを手動で登録する必要がある場合は、`AppServiceProvider`で簡単に行うことができます。同様に、以前は`AuthServiceProvider`に登録されていたルートモデルバインディングや認可ゲートも、`AppServiceProvider`に登録できます。

<a name="opt-in-routing"></a>
#### オプトインAPIおよびブロードキャストルーティング

`api.php`と`channels.php`のルートファイルは、デフォルトでは存在しなくなりました。多くのアプリケーションではこれらのファイルが必要ないためです。代わりに、簡単なArtisanコマンドを使用して作成できます。

```shell
php artisan install:api

php artisan install:broadcasting
```

<a name="middleware"></a>
#### ミドルウェア

以前は、新しいLaravelアプリケーションには9つのミドルウェアが含まれていました。これらのミドルウェアは、リクエストの認証、入力文字列のトリミング、CSRFトークンの検証など、さまざまなタスクを実行しました。

Laravel 11では、これらのミドルウェアはフレームワーク自体に移動され、アプリケーションの構造に余分なものを追加しないようになりました。これらのミドルウェアの動作をカスタマイズするための新しいメソッドがフレームワークに追加され、アプリケーションの`bootstrap/app.php`ファイルから呼び出すことができます。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(
        except: ['stripe/*']
    );

    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ])
})
```

すべてのミドルウェアはアプリケーションの`bootstrap/app.php`を介して簡単にカスタマイズできるため、別のHTTP「カーネル」クラスの必要性がなくなりました。

<a name="scheduling"></a>
#### スケジューリング

新しい`Schedule`ファサードを使用して、スケジュールされたタスクをアプリケーションの`routes/console.php`ファイルで直接定義できるようになり、別のコンソール「カーネル」クラスの必要性がなくなりました。

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
```

<a name="exception-handling"></a>
#### 例外処理

ルーティングやミドルウェアと同様に、例外処理はアプリケーションの`bootstrap/app.php`ファイルからカスタマイズできるようになり、別の例外ハンドラクラスの必要性がなくなり、新しいLaravelアプリケーションに含まれるファイルの総数が減少しました。

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport(MissedFlightException::class);

    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

<a name="base-controller-class"></a>
#### ベース`Controller`クラス

新しいLaravelアプリケーションに含まれるベースコントローラは、簡素化されました。Laravelの内部`Controller`クラスを継承しなくなり、`AuthorizesRequests`と`ValidatesRequests`トレイトも削除されました。これらは、必要に応じてアプリケーションの個々のコントローラに含めることができます。

```php
<?php

namespace App\Http\Controllers;

abstract class Controller
{
    //
}
```

<a name="application-defaults"></a>
#### アプリケーションのデフォルト

デフォルトでは、新しいLaravelアプリケーションはSQLiteをデータベースストレージに使用し、Laravelのセッション、キャッシュ、キューには`database`ドライバを使用します。これにより、新しいLaravelアプリケーションを作成した直後にアプリケーションの構築を開始でき、追加のソフトウェアのインストールや追加のデータベースマイグレーションを必要としません。

さらに、これらのLaravelサービスの`database`ドライバは、多くのアプリケーションコンテキストで本番環境での使用に十分なほど堅牢になっており、ローカル環境と本番環境の両方に対して賢明で統一された選択肢を提供します。

<a name="reverb"></a>
### Laravel Reverb

_Laravel Reverbは、[Joe Dixon](https://github.com/joedixon)によって開発されました。_

[Laravel Reverb](https://reverb.laravel.com)は、高速でスケーラブルなリアルタイムWebSocket通信をLaravelアプリケーションに直接もたらし、Laravel EchoなどのLaravelの既存のイベントブロードキャストツールとのシームレスな統合を提供します。

```shell
php artisan reverb:start
```

さらに、ReverbはRedisのパブリッシュ/サブスクライブ機能を介して水平スケーリングをサポートし、WebSocketトラフィックを複数のバックエンドReverbサーバーに分散させ、単一の高需要アプリケーションをサポートできるようになります。

Laravel Reverbの詳細については、完全な[Reverbドキュメント](reverb.md)を参照してください。

<a name="rate-limiting"></a>
### 秒単位のレートリミット

_秒単位のレートリミットは、[Tim MacDonald](https://github.com/timacdonald)によって提供されました。_

Laravelは、HTTPリクエストとキュージョブを含むすべてのレートリミッターに対して「秒単位」のレートリミットをサポートするようになりました。以前は、Laravelのレートリミッターは「分単位」の粒度に制限されていました。

```php
RateLimiter::for('invoices', function (Request $request) {
    return Limit::perSecond(1);
});
```

Laravelのレートリミットの詳細については、[レートリミットドキュメント](routing.md#rate-limiting)を確認してください。

<a name="health"></a>
### ヘルスルーティング

_ヘルスルーティングは、[Taylor Otwell](https://github.com/taylorotwell)によって提供されました。_

新しいLaravel 11アプリケーションには、`health`ルーティングディレクティブが含まれており、LaravelにサードパーティのアプリケーションヘルスモニタリングサービスやKubernetesのようなオーケストレーションシステムによって呼び出される可能性のあるシンプルなヘルスチェックエンドポイントを定義するよう指示します。デフォルトでは、このルートは`/up`で提供されます：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

このルートにHTTPリクエストが行われると、Laravelは`DiagnosingHealth`イベントもディスパッチし、アプリケーションに関連する追加のヘルスチェックを実行できるようになります。

<a name="encryption"></a>
### グレースフルな暗号化キーのローテーション

_グレースフルな暗号化キーのローテーションは[Taylor Otwell](https://github.com/taylorotwell)によって提供されました_。

Laravelはすべてのクッキー（アプリケーションのセッションクッキーを含む）を暗号化するため、基本的にLaravelアプリケーションへのすべてのリクエストは暗号化に依存しています。しかし、このため、アプリケーションの暗号化キーをローテーションすると、すべてのユーザーがアプリケーションからログアウトされます。さらに、以前の暗号化キーで暗号化されたデータの復号化は不可能になります。

Laravel 11では、アプリケーションの以前の暗号化キーを`APP_PREVIOUS_KEYS`環境変数を介してカンマ区切りのリストとして定義できます。

値を暗号化する際、Laravelは常に`APP_KEY`環境変数内の「現在の」暗号化キーを使用します。値を復号化する際、Laravelはまず現在のキーを試します。現在のキーを使用した復号化が失敗した場合、Laravelはすべての以前のキーを試し、いずれかのキーが値を復号化できるまで続けます。

このグレースフルな復号化のアプローチにより、暗号化キーがローテーションされてもユーザーはアプリケーションを中断することなく使用し続けることができます。

Laravelの暗号化について詳しくは、[暗号化のドキュメント](encryption.md)を確認してください。

<a name="automatic-password-rehashing"></a>
### 自動パスワード再ハッシュ

_自動パスワード再ハッシュは[Stephen Rees-Carter](https://github.com/valorin)によって提供されました_。

Laravelのデフォルトのパスワードハッシュアルゴリズムはbcryptです。bcryptハッシュの「作業係数」は、`config/hashing.php`設定ファイルまたは`BCRYPT_ROUNDS`環境変数を介して調整できます。

通常、CPU/GPUの処理能力が向上するにつれて、bcryptの作業係数を時間とともに増やすべきです。アプリケーションのbcrypt作業係数を増やす場合、Laravelはユーザーがアプリケーションに認証する際に、パスワードをグレースフルに自動的に再ハッシュします。

<a name="prompt-validation"></a>
### プロンプトのバリデーション

_プロンプトバリデータの統合は[Andrea Marco Sartori](https://github.com/cerbero90)によって提供されました_。

[Laravel Prompts](prompts.md)は、プレースホルダーテキストやバリデーションなどのブラウザライクな機能を備えた、美しくユーザーフレンドリーなコマンドラインアプリケーションのフォームを追加するためのPHPパッケージです。

Laravel Promptsはクロージャを介して入力のバリデーションをサポートしています：

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

ただし、多くの入力や複雑なバリデーションシナリオを扱う場合、これは煩雑になる可能性があります。そのため、Laravel 11では、プロンプト入力のバリデーション時にLaravelの[バリデータ](validation.md)のフルパワーを利用できます：

```php
$name = text('What is your name?', validate: [
    'name' => 'required|min:3|max:255',
]);
```

<a name="queue-interaction-testing"></a>
### キューインタラクションのテスト

_キューインタラクションのテストは[Taylor Otwell](https://github.com/taylorotwell)によって提供されました_。

以前は、キューに入れられたジョブがリリース、削除、または手動で失敗したことをテストするのは煩雑で、カスタムキューフェイクやスタブの定義が必要でした。しかし、Laravel 11では、`withFakeQueueInteractions`メソッドを使用してこれらのキューインタラクションを簡単にテストできます：

```php
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
```

キューに入れられたジョブのテストについて詳しくは、[キューのドキュメント](queues.md#testing)を確認してください。

<a name="new-artisan-commands"></a>
### 新しいArtisanコマンド

_クラス作成Artisanコマンドは[Taylor Otwell](https://github.com/taylorotwell)によって提供されました_。

新しいArtisanコマンドが追加され、クラス、列挙型、インターフェース、およびトレイトをすばやく作成できるようになりました：

```shell
php artisan make:class
php artisan make:enum
php artisan make:interface
php artisan make:trait
```

<a name="model-cast-improvements"></a>
### モデルキャストの改善

_モデルキャストの改善は[Nuno Maduro](https://github.com/nunomaduro)によって提供されました_。

Laravel 11では、プロパティではなくメソッドを使用してモデルのキャストを定義できます。これにより、特に引数付きのキャストを使用する場合に、ストリームライン化された流暢なキャスト定義が可能になります：

```php
/**
 * キャストする必要がある属性を取得します。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::using(OptionCollection::class),
                  // AsEncryptedCollection::using(OptionCollection::class),
                  // AsEnumArrayObject::using(OptionEnum::class),
                  // AsEnumCollection::using(OptionEnum::class),
    ];
}
```

属性キャストについて詳しくは、[Eloquentのドキュメント](eloquent-mutators.md#attribute-casting)を確認してください。

<a name="the-once-function"></a>
### `once`関数

_`once`ヘルパーは[Taylor Otwell](https://github.com/taylorotwell)と[Nuno Maduro](https://github.com/nunomaduro)によって提供されました_。

`once`ヘルパー関数は、指定されたコールバックを実行し、リクエストの期間中メモリ内に結果をキャッシュします。同じコールバックで`once`関数を後続で呼び出すと、以前にキャッシュされた結果が返されます：

    function random(): int
    {
        return once(function () {
            return random_int(1, 1000);
        });
    }

    random(); // 123
    random(); // 123 (cached result)
    random(); // 123 (cached result)

`once`ヘルパーについて詳しくは、[ヘルパーのドキュメント](helpers.md#method-once)を確認してください。

<a name="database-performance"></a>
### インメモリデータベースでのテスト時のパフォーマンス向上

_インメモリデータベースのテストパフォーマンス向上は[Anders Jenbo](https://github.com/AJenbo)によって提供されました_。

Laravel 11では、テスト時に`:memory:` SQLiteデータベースを使用する際に大幅な速度向上が提供されます。これを実現するために、LaravelはPHPのPDOオブジェクトへの参照を維持し、接続全体で再利用するようになり、テストの合計実行時間を半分に削減することがよくあります。

<a name="mariadb"></a>
### MariaDBのサポートの改善

_MariaDBのサポートの改善は[Jonas Staudenmeir](https://github.com/staudenmeir)と[Julius Kiekbusch](https://github.com/Jubeki)によって提供されました_。

Laravel 11には、MariaDBのサポートが改善されています。以前のLaravelリリースでは、LaravelのMySQLドライバを介してMariaDBを使用できました。しかし、Laravel 11にはこのデータベースシステムに最適なデフォルトを提供する専用のMariaDBドライバが含まれています。

Laravelのデータベースドライバについて詳しくは、[データベースのドキュメント](database.md)を確認してください。

<a name="inspecting-database"></a>
### データベースの検査とスキーマ操作の改善

_スキーマ操作とデータベースの検査の改善は[Hafez Divandari](https://github.com/hafezdivandari)によって提供されました_。

Laravel 11は、ネイティブの列の変更、名前変更、削除を含む追加のデータベーススキーマ操作と検査メソッドを提供します。さらに、高度な空間型、デフォルト以外のスキーマ名、およびテーブル、ビュー、列、インデックス、外部キーを操作するネイティブスキーマメソッドが提供されます：

    use Illuminate\Support\Facades\Schema;

    $tables = Schema::getTables();
    $views = Schema::getViews();
    $columns = Schema::getColumns('users');
    $indexes = Schema::getIndexes('users');
    $foreignKeys = Schema::getForeignKeys('users');

