# Laravel Pennant

- [はじめに](#introduction)
- [インストール](#installation)
- [設定](#configuration)
- [機能の定義](#defining-features)
    - [クラスベースの機能](#class-based-features)
- [機能の確認](#checking-features)
    - [条件付き実行](#conditional-execution)
    - [The `HasFeatures` Trait](#the-has-features-trait)
    - [Bladeディレクティブ](#blade-directive)
    - [ミドルウェア](#middleware)
    - [機能チェックのインターセプト](#intercepting-feature-checks)
    - [インメモリキャッシュ](#in-memory-cache)
- [スコープ](#scope)
    - [スコープの指定](#specifying-the-scope)
    - [デフォルトスコープ](#default-scope)
    - [Nullableスコープ](#nullable-scope)
    - [スコープの識別](#identifying-scope)
    - [スコープのシリアライズ](#serializing-scope)
- [リッチな機能値](#rich-feature-values)
- [複数の機能の取得](#retrieving-multiple-features)
- [イーガーローディング](#eager-loading)
- [値の更新](#updating-values)
    - [一括更新](#bulk-updates)
    - [機能のパージ](#purging-features)
- [テスト](#testing)
- [カスタムPennantドライバの追加](#adding-custom-pennant-drivers)
    - [ドライバの実装](#implementing-the-driver)
    - [ドライバの登録](#registering-the-driver)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

[Laravel Pennant](https://github.com/laravel/pennant)は、シンプルで軽量な機能フラグパッケージです。機能フラグを使用すると、新しいアプリケーション機能を自信を持って段階的に展開したり、A/Bテストで新しいインターフェースデザインを試したり、トランクベースの開発戦略を補完したり、その他多くのことができます。

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを使用してPennantをプロジェクトにインストールします。

```shell
composer require laravel/pennant
```

次に、`vendor:publish` Artisanコマンドを使用してPennantの設定ファイルとマイグレーションファイルを公開する必要があります。

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

最後に、アプリケーションのデータベースマイグレーションを実行します。これにより、Pennantが`database`ドライバを動作させるために使用する`features`テーブルが作成されます。

```shell
php artisan migrate
```

<a name="configuration"></a>
## 設定

Pennantのアセットを公開した後、その設定ファイルは`config/pennant.php`に配置されます。この設定ファイルでは、Pennantが解決された機能フラグ値を保存するために使用するデフォルトのストレージメカニズムを指定できます。

Pennantは、`array`ドライバを介してインメモリ配列に解決された機能フラグ値を保存することをサポートしています。また、`database`ドライバを介してリレーショナルデータベースに解決された機能フラグ値を永続的に保存することもできます。これは、Pennantが使用するデフォルトのストレージメカニズムです。

<a name="defining-features"></a>
## 機能の定義

機能を定義するには、`Feature`ファサードが提供する`define`メソッドを使用できます。機能の名前と、機能の初期値を解決するために呼び出されるクロージャを提供する必要があります。

通常、機能は`Feature`ファサードを使用してサービスプロバイダで定義されます。クロージャは、機能チェックの「スコープ」を受け取ります。最も一般的には、スコープは現在認証されているユーザーです。この例では、新しいAPIをアプリケーションのユーザーに段階的に展開するための機能を定義します。

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

ご覧のように、機能には以下のルールがあります。

- すべての内部チームメンバーは新しいAPIを使用する必要があります。
- 高トラフィックの顧客は新しいAPIを使用してはいけません。
- それ以外の場合、機能は1/100の確率でユーザーにランダムに割り当てられます。

`new-api`機能が特定のユーザーに対して初めてチェックされると、クロージャの結果がストレージドライバによって保存されます。同じユーザーに対して次回機能がチェックされると、値はストレージから取得され、クロージャは呼び出されません。

便宜上、機能定義が単に抽選を返す場合、クロージャを完全に省略できます。

    Feature::define('site-redesign', Lottery::odds(1, 1000));

<a name="class-based-features"></a>
### クラスベースの機能

Pennantでは、クラスベースの機能も定義できます。クロージャベースの機能定義とは異なり、クラスベースの機能をサービスプロバイダに登録する必要はありません。クラスベースの機能を作成するには、`pennant:feature` Artisanコマンドを呼び出すことができます。デフォルトでは、機能クラスはアプリケーションの`app/Features`ディレクトリに配置されます。

```shell
php artisan pennant:feature NewApi
```

機能クラスを書くとき、`resolve`メソッドを定義するだけで済みます。このメソッドは、特定のスコープに対する機能の初期値を解決するために呼び出されます。ここでも、スコープは通常、現在認証されているユーザーです。

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

クラスベースの機能のインスタンスを手動で解決したい場合は、`Feature`ファサードの`instance`メソッドを呼び出すことができます。

```php
use Illuminate\Support\Facades\Feature;

$instance = Feature::instance(NewApi::class);
```

> NOTE:   
> 機能クラスは[コンテナ](container.md)を介して解決されるため、必要に応じて機能クラスのコンストラクタに依存関係を注入できます。

#### 保存された機能名のカスタマイズ

デフォルトでは、Pennantは機能クラスの完全修飾クラス名を保存します。アプリケーションの内部構造から保存された機能名を切り離したい場合は、機能クラスに`$name`プロパティを指定できます。このプロパティの値は、クラス名の代わりに保存されます。

```php
<?php

namespace App\Features;

class NewApi
{
    /**
     * The stored name of the feature.
     *
     * @var string
     */
    public $name = 'new-api';

    // ...
}
```

<a name="checking-features"></a>
## 機能の確認

機能がアクティブかどうかを確認するには、`Feature`ファサードの`active`メソッドを使用できます。デフォルトでは、機能は現在認証されているユーザーに対してチェックされます。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

機能を別のユーザーや[スコープ](#scope)に対してチェックすることも簡単です。これを行うには、`Feature`ファサードが提供する`for`メソッドを使用します。

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

Pennantは、機能がアクティブかどうかを判断するために役立ついくつかの追加の便利なメソッドも提供しています。

```php
// 指定されたすべての機能がアクティブかどうかを判断する...
Feature::allAreActive(['new-api', 'site-redesign']);

// 指定されたいずれかの機能がアクティブかどうかを判断する...
Feature::someAreActive(['new-api', 'site-redesign']);

// 機能が非アクティブかどうかを判断する...
Feature::inactive('new-api');

// 指定されたすべての機能が非アクティブかどうかを判断する...
Feature::allAreInactive(['new-api', 'site-redesign']);

// 指定されたいずれかの機能が非アクティブかどうかを判断する...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> NOTE:  
> HTTPコンテキスト外でPennantを使用する場合、例えばArtisanコマンドやキュージョブなどでは、通常、機能のスコープを[明示的に指定する](#specifying-the-scope)必要があります。または、認証済みHTTPコンテキストと未認証コンテキストの両方を考慮した[デフォルトスコープ](#default-scope)を定義することもできます。

<a name="checking-class-based-features"></a>
#### クラスベースの機能の確認

クラスベースの機能の場合、機能をチェックするときにクラス名を指定する必要があります。

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
                ? $this->resolveNewApiResponse($request)
                : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

<a name="conditional-execution"></a>
### 条件付き実行

`when`メソッドは、機能がアクティブな場合に指定されたクロージャを流暢に実行するために使用できます。さらに、機能が非アクティブな場合に実行される2番目のクロージャを提供することもできます。

    <?php

    namespace App\Http\Controllers;

    use App\Features\NewApi;
    use Illuminate\Http\Request;
    use Illuminate\Http\Response;
    use Laravel\Pennant\Feature;

    class PodcastController
    {
        /**
         * リソースの一覧を表示します。
         */
        public function index(Request $request): Response
        {
            return Feature::when(NewApi::class,
                fn () => $this->resolveNewApiResponse($request),
            fn () => $this->resolveLegacyApiResponse($request),
        );
    }

    // ...
}
```

`unless`メソッドは`when`メソッドの逆で、機能が非アクティブの場合に最初のクロージャを実行します：

```php
return Feature::unless(NewApi::class,
    fn () => $this->resolveLegacyApiResponse($request),
    fn () => $this->resolveNewApiResponse($request),
);
```

<a name="the-has-features-trait"></a>
### `HasFeatures`トレイト

Pennantの`HasFeatures`トレイトは、アプリケーションの`User`モデル（または他の機能を持つモデル）に追加して、モデルから直接機能をチェックするための流暢で便利な方法を提供することができます：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;

    // ...
}
```

トレイトをモデルに追加したら、`features`メソッドを呼び出すことで機能を簡単にチェックできます：

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

もちろん、`features`メソッドは、機能と対話するための他の多くの便利なメソッドへのアクセスも提供します：

```php
// 値...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// 状態...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// 条件付き実行...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```

<a name="blade-directive"></a>
### Bladeディレクティブ

Bladeで機能をチェックするためのシームレスな体験を提供するために、Pennantは`@feature`ディレクティブを提供します：

```blade
@feature('site-redesign')
    <!-- 'site-redesign'がアクティブ -->
@else
    <!-- 'site-redesign'が非アクティブ -->
@endfeature
```

<a name="middleware"></a>
### ミドルウェア

Pennantには、現在認証されているユーザーがルートが呼び出される前に機能にアクセスできるかどうかを検証するために使用できる[ミドルウェア](middleware.md)も含まれています。ミドルウェアをルートに割り当て、そのルートにアクセスするために必要な機能を指定できます。現在認証されているユーザーに対して指定された機能のいずれかが非アクティブの場合、ルートは`400 Bad Request` HTTPレスポンスを返します。複数の機能を静的な`using`メソッドに渡すことができます。

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>
#### レスポンスのカスタマイズ

ミドルウェアによってリストされた機能のいずれかが非アクティブの場合に返されるレスポンスをカスタマイズしたい場合は、`EnsureFeaturesAreActive`ミドルウェアによって提供される`whenInactive`メソッドを使用できます。通常、このメソッドはアプリケーションのサービスプロバイダの`boot`メソッド内で呼び出す必要があります：

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );

    // ...
}
```

<a name="intercepting-feature-checks"></a>
### 機能チェックのインターセプト

特定の機能の保存された値を取得する前に、いくつかのインメモリチェックを実行すると便利な場合があります。例えば、機能フラグの背後に新しいAPIを開発していて、保存された機能値を失うことなく新しいAPIを無効にする機能が必要だとします。新しいAPIにバグが見つかった場合、内部チームメンバーを除く全員に対して簡単に無効にし、バグを修正してから以前に機能にアクセスしていたユーザーに対して新しいAPIを再び有効にすることができます。

これは、[クラスベースの機能](#class-based-features)の`before`メソッドで実現できます。存在する場合、`before`メソッドは常にインメモリで実行され、ストレージから値を取得する前に実行されます。メソッドから非`null`の値が返された場合、その値はリクエストの期間中、機能の保存された値の代わりに使用されます：

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * 保存された値が取得される前に常にインメモリでチェックを実行します。
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }
    }

    /**
     * 機能の初期値を解決します。
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

この機能を使用して、以前は機能フラグの背後にあった機能のグローバルロールアウトをスケジュールすることもできます：

```php
<?php

namespace App\Features;

use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Config;

class NewApi
{
    /**
     * 保存された値が取得される前に常にインメモリでチェックを実行します。
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }

        if (Carbon::parse(Config::get('features.new-api.rollout-date'))->isPast()) {
            return true;
        }
    }

    // ...
}
```

<a name="in-memory-cache"></a>
### インメモリキャッシュ

機能をチェックすると、Pennantは結果のインメモリキャッシュを作成します。`database`ドライバを使用している場合、これは同じリクエスト内で同じ機能フラグを再チェックしても追加のデータベースクエリがトリガーされないことを意味します。これにより、リクエストの期間中、機能が一貫した結果を持つことが保証されます。

インメモリキャッシュを手動でフラッシュする必要がある場合は、`Feature`ファサードによって提供される`flushCache`メソッドを使用できます：

```php
Feature::flushCache();
```

<a name="scope"></a>
## スコープ

<a name="specifying-the-scope"></a>
### スコープの指定

説明したように、機能は通常、現在認証されているユーザーに対してチェックされます。しかし、これが常にあなたのニーズに合うとは限りません。したがって、`Feature`ファサードの`for`メソッドを介して、特定の機能をチェックするためのスコープを指定することが可能です：

```php
return Feature::for($user)->active('new-api')
        ? $this->resolveNewApiResponse($request)
        : $this->resolveLegacyApiResponse($request);
```

もちろん、機能スコープは「ユーザー」に限定されません。例えば、新しい請求体験を構築し、個々のユーザーではなくチーム全体にロールアウトしているとします。古いチームよりも新しいチームに対してより遅いロールアウトを行いたい場合があります。機能解決クロージャは次のようになるかもしれません：

```php
use App\Models\Team;
use Carbon\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

定義したクロージャは`User`ではなく`Team`モデルを期待していることに気付くでしょう。ユーザーのチームに対してこの機能がアクティブかどうかを判断するには、`Feature`ファサードによって提供される`for`メソッドにチームを渡す必要があります：

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect('/billing/v2');
}

// ...
```

<a name="default-scope"></a>
### デフォルトスコープ

Pennantが機能をチェックするために使用するデフォルトスコープをカスタマイズすることも可能です。例えば、すべての機能が現在認証されているユーザーのチームではなくユーザーに対してチェックされる場合があります。`Feature::for($user->team)`を毎回呼び出す代わりに、チームをデフォルトスコープとして指定することができます。通常、これはアプリケーションのサービスプロバイダの1つで行う必要があります：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 任意のアプリケーションサービスをブートストラップします。
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);

        // ...
    }
}
```

`for`メソッドを介して明示的にスコープが提供されない場合、機能チェックは現在認証されているユーザーのチームをデフォルトスコープとして使用します：

```php
Feature::active('billing-v2');

// これは次と同等です...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>
### Nullableスコープ

機能をチェックする際に提供するスコープが`null`であり、機能の定義がnullable型またはunion型に`null`を含むことによって`null`をサポートしていない場合、Pennantは自動的に`false`を機能の結果値として返します。

つまり、機能に渡すスコープが潜在的に `null` であり、機能の値リゾルバが呼び出されるようにしたい場合、機能の定義でそれを考慮する必要があります。Artisanコマンド、キューに入れられたジョブ、または認証されていないルート内で機能をチェックする場合、`null` スコープが発生する可能性があります。これらのコンテキストでは通常、認証されたユーザーが存在しないため、デフォルトのスコープは `null` になります。

常に[スコープを明示的に指定](#specifying-the-scope)しない場合は、スコープの型が「nullable」であり、機能定義ロジック内で `null` スコープ値を処理できるようにする必要があります：

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User|null $user) => match (true) {
    $user === null => true,
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

<a name="identifying-scope"></a>
### スコープの識別

Pennantの組み込みの `array` および `database` ストレージドライバは、すべてのPHPデータ型およびEloquentモデルのスコープ識別子を適切に保存する方法を知っています。ただし、アプリケーションがサードパーティのPennantドライバを使用している場合、そのドライバはEloquentモデルまたはアプリケーション内の他のカスタムタイプの識別子を適切に保存する方法を知らない可能性があります。

この点を考慮して、Pennantでは、Pennantスコープとして使用されるアプリケーション内のオブジェクトに `FeatureScopeable` 契約を実装することで、スコープ値をストレージ用にフォーマットできます。

例えば、単一のアプリケーションで組み込みの `database` ドライバとサードパーティの「Flag Rocket」ドライバの2つを使用しているとします。「Flag Rocket」ドライバはEloquentモデルを適切に保存する方法を知らず、代わりに `FlagRocketUser` インスタンスを必要とします。`FeatureScopeable` 契約で定義された `toFeatureIdentifier` を実装することで、アプリケーションで使用される各ドライバに提供されるストア可能なスコープ値をカスタマイズできます：

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * オブジェクトを指定されたドライバの機能スコープ識別子にキャストします。
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
            'database' => $this,
            'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```

<a name="serializing-scope"></a>
### スコープのシリアライズ

デフォルトでは、PennantはEloquentモデルに関連付けられた機能を保存する際に完全修飾クラス名を使用します。すでに[Eloquentモルフマップ](eloquent-relationships.md#custom-polymorphic-types)を使用している場合、Pennantもモルフマップを使用して、保存された機能をアプリケーション構造から切り離すことを選択できます。

これを実現するには、サービスプロバイダでEloquentモルフマップを定義した後、`Feature` ファサードの `useMorphMap` メソッドを呼び出します：

```php
use Illuminate\Database\Eloquent\Relations\Relation;
use Laravel\Pennant\Feature;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);

Feature::useMorphMap();
```

<a name="rich-feature-values"></a>
## リッチな機能値

これまで、主に機能をバイナリ状態（「アクティブ」または「非アクティブ」）として示してきましたが、Pennantではリッチな値も保存できます。

例えば、アプリケーションの「今すぐ購入」ボタンの新しい3つの色をテストしているとします。機能定義から `true` または `false` を返す代わりに、文字列を返すことができます：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

`purchase-button` 機能の値を取得するには、`value` メソッドを使用します：

```php
$color = Feature::value('purchase-button');
```

Pennantの組み込みBladeディレクティブを使用すると、機能の現在の値に基づいてコンテンツを条件付きでレンダリングするのも簡単です：

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' がアクティブ -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' がアクティブ -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' がアクティブ -->
@endfeature
```

> NOTE:   
> リッチな値を使用する場合、機能は `false` 以外の値を持つ場合に「アクティブ」と見なされることに注意してください。

[条件付き `when`](#conditional-execution) メソッドを呼び出す場合、機能のリッチな値が最初のクロージャに提供されます：

    Feature::when('purchase-button',
        fn ($color) => /* ... */,
        fn () => /* ... */,
    );

同様に、条件付き `unless` メソッドを呼び出す場合、機能のリッチな値がオプションの2番目のクロージャに提供されます：

    Feature::unless('purchase-button',
        fn () => /* ... */,
        fn ($color) => /* ... */,
    );

<a name="retrieving-multiple-features"></a>
## 複数の機能の取得

`values` メソッドを使用すると、指定されたスコープの複数の機能を取得できます：

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
// ]
```

または、`all` メソッドを使用して、指定されたスコープのすべての定義された機能の値を取得できます：

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

ただし、クラスベースの機能は動的に登録され、Pennantには明示的にチェックされるまで知られていません。これは、現在のリクエスト中に既にチェックされていない場合、アプリケーションのクラスベースの機能が `all` メソッドによって返される結果に表示されない可能性があることを意味します。

`all` メソッドを使用する際に常に機能クラスが含まれるようにしたい場合、Pennantの機能発見機能を使用できます。まず、アプリケーションのサービスプロバイダの1つで `discover` メソッドを呼び出します：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Pennant\Feature;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスをブートストラップします。
         */
        public function boot(): void
        {
            Feature::discover();

            // ...
        }
    }

`discover` メソッドは、アプリケーションの `app/Features` ディレクトリ内のすべての機能クラスを登録します。`all` メソッドは、現在のリクエスト中にチェックされたかどうかに関係なく、これらのクラスを結果に含めます：

```php
Feature::all();

// [
//     'App\Features\NewApi' => true,
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

<a name="eager-loading"></a>
## イーガーローディング

Pennantは、単一のリクエストに対して解決されたすべての機能のインメモリキャッシュを保持しますが、パフォーマンスの問題が発生する可能性があります。これを軽減するために、Pennantは機能値をイーガーロードする機能を提供します。

これを説明するために、ループ内で機能がアクティブかどうかをチェックしているとします：

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

データベースドライバを使用していると仮定すると、このコードはループ内の各ユーザーに対してデータベースクエリを実行します - 数百のクエリを実行する可能性があります。ただし、Pennantの `load` メソッドを使用すると、ユーザーまたはスコープのコレクションの機能値をイーガーロードすることで、この潜在的なパフォーマンスのボトルネックを取り除くことができます：

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

まだロードされていない場合にのみ機能値をロードするには、`loadMissing` メソッドを使用できます：

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

すべての定義された機能をロードするには、`loadAll` メソッドを使用できます：

```php
Feature::for($user)->loadAll();
```

<a name="updating-values"></a>
## 値の更新

機能の値が最初に解決されると、基礎となるドライバは結果をストレージに保存します。これは、リクエスト間でユーザーに一貫したエクスペリエンスを提供するために必要です。ただし、場合によっては、機能の保存された値を手動で更新したいことがあります。

これを実現するには、`activate` および `deactivate` メソッドを使用して、機能を「オン」または「オフ」に切り替えることができます：

```php
use Laravel\Pennant\Feature;

// デフォルトスコープの機能をアクティブにします...
Feature::activate('new-api');

// 指定されたスコープの機能を非アクティブにします...
Feature::for($user->team)->deactivate('billing-v2');
```

また、`activate` メソッドに2番目の引数を指定することで、機能のリッチな値を手動で設定することもできます：

```php
Feature::activate('purchase-button', 'seafoam-green');
```

Pennantに機能の保存された値を忘れさせるには、`forget` メソッドを使用できます。機能が再度チェックされると、Pennantは機能定義から機能の値を解決します：

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>
### 一括更新

保存された機能値を一括で更新するには、`activateForEveryone` および `deactivateForEveryone` メソッドを使用できます。

例えば、`new-api`機能の安定性に自信が持て、チェックアウトフローの`'purchase-button'`の色に最適なものが見つかったと想像してください。その場合、すべてのユーザーに対して保存された値を次のように更新できます：

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

また、すべてのユーザーに対して機能を無効にすることもできます：

```php
Feature::deactivateForEveryone('new-api');
```

> NOTE:   
> これにより、Pennantのストレージドライバによって保存された機能値のみが更新されます。アプリケーション内の機能定義も更新する必要があります。

<a name="purging-features"></a>
### 機能のパージ

時には、ストレージから機能全体をパージすることが有用な場合があります。これは通常、アプリケーションから機能を削除した場合や、すべてのユーザーにロールアウトしたい機能の定義を調整した場合に必要です。

`purge`メソッドを使用して、機能のすべての保存された値を削除できます：

```php
// 単一の機能をパージする...
Feature::purge('new-api');

// 複数の機能をパージする...
Feature::purge(['new-api', 'purchase-button']);
```

ストレージから_すべての_機能をパージしたい場合は、引数なしで`purge`メソッドを呼び出すことができます：

```php
Feature::purge();
```

アプリケーションのデプロイパイプラインの一部として機能をパージすることが有用な場合があるため、Pennantには`pennant:purge` Artisanコマンドが含まれており、指定された機能をストレージからパージします：

```sh
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

また、特定の機能リストに含まれるものを除くすべての機能をパージすることも可能です。例えば、すべての機能をパージしたいが、"new-api"と"purchase-button"の機能の値をストレージに保持したい場合、それらの機能名を`--except`オプションに渡すことで実現できます：

```sh
php artisan pennant:purge --except=new-api --except=purchase-button
```

便宜上、`pennant:purge`コマンドは`--except-registered`フラグもサポートしています。このフラグは、サービスプロバイダで明示的に登録された機能を除くすべての機能をパージすることを示します：

```sh
php artisan pennant:purge --except-registered
```

<a name="testing"></a>
## テスト

機能フラグとやり取りするコードをテストする場合、テスト内で機能フラグの返される値を制御する最も簡単な方法は、単に機能を再定義することです。例えば、アプリケーションのサービスプロバイダの1つで次のような機能が定義されていると想像してください：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

テスト内で機能の返される値を変更するには、テストの最初で機能を再定義します。以下のテストは常にパスしますが、`Arr::random()`の実装はサービスプロバイダにまだ存在します：

===  "Pest"
```php
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define('purchase-button', 'seafoam-green');

    expect(Feature::value('purchase-button'))->toBe('seafoam-green');
});
```

===  "PHPUnit"
```php
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');

    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

同じアプローチは、クラスベースの機能にも使用できます：

===  "Pest"
```php
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define(NewApi::class, true);

    expect(Feature::value(NewApi::class))->toBeTrue();
});
```

===  "PHPUnit"
```php
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);

    $this->assertTrue(Feature::value(NewApi::class));
}
```

機能が`Lottery`インスタンスを返す場合、いくつかの便利な[テストヘルパー](helpers.md#testing-lotteries)が利用可能です。

<a name="store-configuration"></a>
#### ストアの設定

Pennantがテスト中に使用するストアは、アプリケーションの`phpunit.xml`ファイル内で`PENNANT_STORE`環境変数を定義することで設定できます：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
        <env name="PENNANT_STORE" value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```

<a name="adding-custom-pennant-drivers"></a>
## カスタムPennantドライバの追加

<a name="implementing-the-driver"></a>
#### ドライバの実装

Pennantの既存のストレージドライバがアプリケーションのニーズに合わない場合、独自のストレージドライバを書くことができます。カスタムドライバは、`Laravel\Pennant\Contracts\Driver`インターフェースを実装する必要があります：

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

ここで、Redis接続を使用してこれらの各メソッドを実装する必要があります。これらの各メソッドの実装例については、[Pennantソースコード](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php)の`Laravel\Pennant\Drivers\DatabaseDriver`を参照してください。

> NOTE:  
> Laravelには拡張機能を格納するためのディレクトリは付属していません。好きな場所に配置することができます。この例では、`RedisFeatureDriver`を格納するために`Extensions`ディレクトリを作成しました。

<a name="registering-the-driver"></a>
#### ドライバの登録

ドライバが実装されたら、Laravelに登録する準備が整いました。Pennantに追加のドライバを追加するには、`Feature`ファサードが提供する`extend`メソッドを使用できます。`extend`メソッドは、アプリケーションの[サービスプロバイダ](providers.md)の`boot`メソッドから呼び出す必要があります：

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

ドライバが登録されたら、アプリケーションの`config/pennant.php`設定ファイルで`redis`ドライバを使用できます：

    'stores' => [

        'redis' => [
            'driver' => 'redis',
            'connection' => null,
        ],

        // ...

    ],

<a name="events"></a>
## イベント

Pennantは、アプリケーション全体で機能フラグを追跡する際に有用なさまざまなイベントをディスパッチします。

### `Laravel\Pennant\Events\FeatureRetrieved`

このイベントは、[機能がチェックされる](#checking-features)たびにディスパッチされます。このイベントは、アプリケーション全体での機能フラグの使用に対するメトリクスを作成および追跡するのに役立ちます。

### `Laravel\Pennant\Events\FeatureResolved`

このイベントは、特定のスコープに対して機能の値が初めて解決されたときにディスパッチされます。

### `Laravel\Pennant\Events\UnknownFeatureResolved`

このイベントは、特定のスコープに対して不明な機能が初めて解決されたときにディスパッチされます。このイベントをリッスンすることは、機能フラグを削除しようとしたが、アプリケーション全体に誤って残された参照がある場合に役立ちます：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnknownFeatureResolved;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(function (UnknownFeatureResolved $event) {
            Log::error("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyRegisteringFeatureClass`

このイベントは、リクエスト中に[クラスベースの機能](#class-based-features)が初めて動的にチェックされたときにディスパッチされます。

### `Laravel\Pennant\Events\UnexpectedNullScopeEncountered`

このイベントは、[nullをサポートしない](#nullable-scope)機能定義に`null`スコープが渡されたときにディスパッチされます。

この状況は適切に処理され、機能は`false`を返します。ただし、この機能のデフォルトの適切な動作をオプトアウトしたい場合は、アプリケーションの`AppServiceProvider`の`boot`メソッドでこのイベントのリスナーを登録できます：

```php
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnexpectedNullScopeEncountered;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(UnexpectedNullScopeEncountered::class, fn () => abort(500));
}
```

### `Laravel\Pennant\Events\FeatureUpdated`

このイベントは、通常`activate`または`deactivate`を呼び出すことで、スコープの機能が更新されたときにディスパッチされます。

### `Laravel\Pennant\Events\FeatureUpdatedForAllScopes`

このイベントは、すべてのスコープの機能が更新されたときにディスパッチされます。

このイベントは、通常 `activateForEveryone` または `deactivateForEveryone` を呼び出すことによって、すべてのスコープの機能を更新する際にディスパッチされます。

### `Laravel\Pennant\Events\FeatureDeleted`

このイベントは、通常 `forget` を呼び出すことによって、スコープの機能を削除する際にディスパッチされます。

### `Laravel\Pennant\Events\FeaturesPurged`

このイベントは、特定の機能をパージする際にディスパッチされます。

### `Laravel\Pennant\Events\AllFeaturesPurged`

このイベントは、すべての機能をパージする際にディスパッチされます。
