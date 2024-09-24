# Laravel Pulse

- [はじめに](#introduction)
- [インストール](#installation)
    - [設定](#configuration)
- [ダッシュボード](#dashboard)
    - [認可](#dashboard-authorization)
    - [カスタマイズ](#dashboard-customization)
    - [ユーザーの解決](#dashboard-resolving-users)
    - [カード](#dashboard-cards)
- [エントリのキャプチャ](#capturing-entries)
    - [レコーダー](#recorders)
    - [フィルタリング](#filtering)
- [パフォーマンス](#performance)
    - [別のデータベースの使用](#using-a-different-database)
    - [Redis インジェスト](#ingest)
    - [サンプリング](#sampling)
    - [トリミング](#trimming)
    - [Pulse 例外の処理](#pulse-exceptions)
- [カスタムカード](#custom-cards)
    - [カードコンポーネント](#custom-card-components)
    - [スタイリング](#custom-card-styling)
    - [データのキャプチャと集計](#custom-card-data)

<a name="introduction"></a>
## はじめに

[Laravel Pulse](https://github.com/laravel/pulse) は、アプリケーションのパフォーマンスと使用状況に関する一目でわかる洞察を提供します。Pulseを使用すると、遅いジョブやエンドポイントなどのボトルネックを追跡し、最もアクティブなユーザーを見つけることができます。

個々のイベントの詳細なデバッグについては、[Laravel Telescope](telescope.md)をチェックしてください。

<a name="installation"></a>
## インストール

> WARNING:  
> Pulseのファーストパーティストレージ実装は現在、MySQL、MariaDB、またはPostgreSQLデータベースが必要です。異なるデータベースエンジンを使用している場合、Pulseデータ用に別のMySQL、MariaDB、またはPostgreSQLデータベースが必要になります。

Composerパッケージマネージャを使用してPulseをインストールできます：

```sh
composer require laravel/pulse
```

次に、`vendor:publish` Artisanコマンドを使用してPulseの設定ファイルとマイグレーションファイルを公開する必要があります：

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

最後に、Pulseのデータを保存するために必要なテーブルを作成するために`migrate`コマンドを実行する必要があります：

```shell
php artisan migrate
```

Pulseのデータベースマイグレーションが実行されたら、`/pulse`ルートを介してPulseダッシュボードにアクセスできます。

> NOTE:  
> アプリケーションのプライマリデータベースにPulseデータを保存したくない場合、[専用のデータベース接続を指定](#using-a-different-database)できます。

<a name="configuration"></a>
### 設定

Pulseの多くの設定オプションは、環境変数を使用して制御できます。利用可能なオプションを確認したり、新しいレコーダーを登録したり、高度なオプションを設定したりするには、`config/pulse.php`設定ファイルを公開できます：

```sh
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## ダッシュボード

<a name="dashboard-authorization"></a>
### 認可

Pulseダッシュボードは`/pulse`ルートを介してアクセスできます。デフォルトでは、`local`環境でのみこのダッシュボードにアクセスできるため、本番環境では`'viewPulse'`認可ゲートをカスタマイズして認可を設定する必要があります。これは、アプリケーションの`app/Providers/AppServiceProvider.php`ファイル内で行うことができます：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

<a name="dashboard-customization"></a>
### カスタマイズ

Pulseダッシュボードのカードとレイアウトは、ダッシュボードビューを公開することで設定できます。ダッシュボードビューは`resources/views/vendor/pulse/dashboard.blade.php`に公開されます：

```sh
php artisan vendor:publish --tag=pulse-dashboard
```

ダッシュボードは[Livewire](https://livewire.laravel.com/)によって動作し、JavaScriptアセットを再構築することなくカードとレイアウトをカスタマイズできます。

このファイル内で、`<x-pulse>`コンポーネントはダッシュボードをレンダリングし、カードのグリッドレイアウトを提供します。ダッシュボードを画面全体に広げたい場合は、コンポーネントに`full-width`プロパティを指定できます：

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

デフォルトでは、`<x-pulse>`コンポーネントは12列のグリッドを作成しますが、`cols`プロパティを使用してこれをカスタマイズできます：

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

各カードは、スペースと位置を制御するために`cols`と`rows`プロパティを受け入れます：

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

ほとんどのカードは、スクロールではなく全カードを表示するための`expand`プロパティも受け入れます：

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### ユーザーの解決

アプリケーション使用状況カードなど、ユーザーに関する情報を表示するカードの場合、PulseはユーザーのIDのみを記録します。ダッシュボードをレンダリングする際、Pulseはデフォルトの`Authenticatable`モデルから`name`と`email`フィールドを解決し、Gravatarウェブサービスを使用してアバターを表示します。

アプリケーションの`App\Providers\AppServiceProvider`クラス内で`Pulse::user`メソッドを呼び出すことで、フィールドとアバターをカスタマイズできます。

`user`メソッドは、表示される`Authenticatable`モデルを受け取り、ユーザーの`name`、`extra`、`avatar`情報を含む配列を返すクロージャを受け入れます：

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

> NOTE:  
> 認証されたユーザーのキャプチャと取得方法を完全にカスタマイズするには、`Laravel\Pulse\Contracts\ResolvesUsers`契約を実装し、Laravelの[サービスコンテナ](container.md#binding-a-singleton)にバインドします。

<a name="dashboard-cards"></a>
### カード

<a name="servers-card"></a>
#### サーバー

`<livewire:pulse.servers />`カードは、`pulse:check`コマンドを実行しているすべてのサーバーのシステムリソース使用状況を表示します。システムリソースのレポートに関する詳細については、[サーバーレコーダー](#servers-recorder)のドキュメントを参照してください。

インフラストラクチャ内のサーバーを交換する場合、一定期間後に非アクティブなサーバーをPulseダッシュボードに表示しないようにすることができます。これは、`ignore-after`プロパティを使用して行うことができます。このプロパティは、非アクティブなサーバーをPulseダッシュボードから削除するまでの秒数を受け入れます。または、`1 hour`や`3 days and 1 hour`などの相対時間形式の文字列を指定することもできます：

```blade
<livewire:pulse.servers ignore-after="3 hours" />
```

<a name="application-usage-card"></a>
#### アプリケーション使用状況

`<livewire:pulse.usage />`カードは、アプリケーションにリクエストを送信し、ジョブをディスパッチし、遅いリクエストを経験した上位10人のユーザーを表示します。

すべての使用状況メトリクスを同時に画面に表示したい場合は、カードを複数回含めて`type`属性を指定できます：

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

Pulseがユーザー情報を取得して表示する方法をカスタマイズする方法については、[ユーザーの解決](#dashboard-resolving-users)のドキュメントを参照してください。

> NOTE:  
> アプリケーションが大量のリクエストを受け取るか、大量のジョブをディスパッチする場合、[サンプリング](#sampling)を有効にすることをお勧めします。詳細については、[ユーザーリクエストレコーダー](#user-requests-recorder)、[ユーザージョブレコーダー](#user-jobs-recorder)、および[遅いジョブレコーダー](#slow-jobs-recorder)のドキュメントを参照してください。

<a name="exceptions-card"></a>
#### 例外

`<livewire:pulse.exceptions />`カードは、アプリケーションで発生した例外の頻度と最近性を表示します。デフォルトでは、例外は例外クラスと発生した場所に基づいてグループ化されます。詳細については、[例外レコーダー](#exceptions-recorder)のドキュメントを参照してください。

<a name="queues-card"></a>
#### キュー

`<livewire:pulse.queues />`カードは、アプリケーション内のキューのスループットを表示します。これには、キューに入れられたジョブ、処理中のジョブ、処理されたジョブ、リリースされたジョブ、失敗したジョブの数が含まれます。詳細については、[キューレコーダー](#queues-recorder)のドキュメントを参照してください。

<a name="slow-requests-card"></a>
#### 遅いリクエスト

`<livewire:pulse.slow-requests />`カードは、設定されたしきい値（デフォルトでは1,000ms）を超えるアプリケーションへの受信リクエストを表示します。詳細については、[遅いリクエストレコーダー](#slow-requests-recorder)のドキュメントを参照してください。

<a name="slow-jobs-card"></a>
#### 遅いジョブ

`<livewire:pulse.slow-jobs />`カードは、設定されたしきい値（デフォルトでは1,000ms）を超えるアプリケーション内のキューに入れられたジョブを表示します。詳細については、[遅いジョブレコーダー](#slow-jobs-recorder)のドキュメントを参照してください。

<a name="slow-queries-card"></a>
#### 遅いクエリ

`<livewire:pulse.slow-queries />`カードは、設定されたしきい値（デフォルトでは1,000ms）を超えるアプリケーション内のデータベースクエリを表示します。

デフォルトでは、遅いクエリはSQLクエリ（バインディングなし）と発生した場所に基づいてグループ化されますが、SQLクエリのみに基づいてグループ化する場合は、場所のキャプチャを選択しないこともできます。

非常に大きなSQLクエリが構文ハイライトを受け取ることによるレンダリングパフォーマンスの問題が発生した場合、`without-highlighting`プロパティを追加することでハイライトを無効にできます：

```blade
<livewire:pulse.slow-queries without-highlighting />
```

詳細については、[遅いクエリレコーダー](#slow-queries-recorder)のドキュメントを参照してください。

<a name="slow-outgoing-requests-card"></a>
#### 遅い送信リクエスト

`<livewire:pulse.slow-outgoing-requests />`カードは、Laravelの[HTTPクライアント](http-client.md)を使用して行われた送信リクエストのうち、設定されたしきい値（デフォルトでは1,000ms）を超えるものを表示します。

デフォルトでは、エントリは完全なURLでグループ化されます。ただし、正規表現を使用して類似の送信リクエストを正規化またはグループ化したい場合があります。詳細については、[遅い送信リクエストレコーダー](#slow-outgoing-requests-recorder)のドキュメントを参照してください。

<a name="cache-card"></a>
#### キャッシュ

`<livewire:pulse.cache />` カードは、アプリケーションのキャッシュヒットとミスの統計を、グローバルにおよび個々のキーごとに表示します。

デフォルトでは、エントリはキーでグループ化されます。ただし、正規表現を使用して類似のキーを正規化またはグループ化したい場合があります。詳細については、[キャッシュインタラクションレコーダー](#cache-interactions-recorder)のドキュメントを参照してください。

<a name="capturing-entries"></a>
## エントリのキャプチャ

ほとんどのPulseレコーダーは、Laravelによってディスパッチされるフレームワークイベントに基づいて自動的にエントリをキャプチャします。ただし、[サーバーレコーダー](#servers-recorder)や一部のサードパーティカードは、情報を定期的にポーリングする必要があります。これらのカードを使用するには、すべての個々のアプリケーションサーバーで `pulse:check` デーモンを実行する必要があります：

```php
php artisan pulse:check
```

> NOTE:  
> `pulse:check` プロセスをバックグラウンドで永続的に実行させるには、Supervisorなどのプロセスモニターを使用して、コマンドが停止しないようにする必要があります。

`pulse:check` コマンドは長時間実行されるプロセスであるため、再起動しない限りコードベースの変更を認識しません。アプリケーションのデプロイプロセス中に `pulse:restart` コマンドを呼び出して、コマンドを適切に再起動する必要があります：

```sh
php artisan pulse:restart
```

> NOTE:  
> Pulseは[キャッシュ](cache.md)を使用して再起動シグナルを保存するため、この機能を使用する前に、アプリケーションに適切にキャッシュドライバーが設定されていることを確認する必要があります。

<a name="recorders"></a>
### レコーダー

レコーダーは、アプリケーションからエントリをキャプチャし、Pulseデータベースに記録する役割を担います。レコーダーは、[Pulse設定ファイル](#configuration)の `recorders` セクションで登録および設定されます。

<a name="cache-interactions-recorder"></a>
#### キャッシュインタラクション

`CacheInteractions` レコーダーは、アプリケーションで発生する[キャッシュ](cache.md)のヒットとミスに関する情報をキャプチャし、[キャッシュ](#cache-card)カードに表示します。

オプションで、[サンプリングレート](#sampling)と無視するキーパターンを調整できます。

また、キーのグループ化を設定して、類似のキーを単一のエントリとしてグループ化することもできます。たとえば、同じタイプの情報をキャッシュするキーから一意のIDを削除したい場合があります。グループは、正規表現を使用してキーの一部を「検索して置換」することで設定されます。設定ファイルに例が含まれています：

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

最初に一致したパターンが使用されます。パターンが一致しない場合、キーはそのままキャプチャされます。

<a name="exceptions-recorder"></a>
#### 例外

`Exceptions` レコーダーは、アプリケーションで発生する報告可能な例外に関する情報をキャプチャし、[例外](#exceptions-card)カードに表示します。

オプションで、[サンプリングレート](#sampling)と無視する例外パターンを調整できます。また、例外が発生した場所をキャプチャするかどうかを設定することもできます。キャプチャされた場所はPulseダッシュボードに表示され、例外の発生元を追跡するのに役立ちます。ただし、同じ例外が複数の場所で発生した場合、それぞれの一意の場所ごとに複数回表示されます。

<a name="queues-recorder"></a>
#### キュー

`Queues` レコーダーは、アプリケーションのキューに関する情報をキャプチャし、[キュー](#queues-card)カードに表示します。

オプションで、[サンプリングレート](#sampling)と無視するジョブパターンを調整できます。

<a name="slow-jobs-recorder"></a>
#### 遅いジョブ

`SlowJobs` レコーダーは、アプリケーションで発生する遅いジョブに関する情報をキャプチャし、[遅いジョブ](#slow-jobs-recorder)カードに表示します。

オプションで、遅いジョブのしきい値、[サンプリングレート](#sampling)、および無視するジョブパターンを調整できます。

他のジョブよりも時間がかかることが予想されるジョブがある場合、ジョブごとのしきい値を設定できます：

```php
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

正規表現パターンがジョブのクラス名に一致しない場合、`'default'` 値が使用されます。

<a name="slow-outgoing-requests-recorder"></a>
#### 遅い送信リクエスト

`SlowOutgoingRequests` レコーダーは、Laravelの[HTTPクライアント](http-client.md)を使用して行われた送信HTTPリクエストのうち、設定されたしきい値を超えるものに関する情報をキャプチャし、[遅い送信リクエスト](#slow-outgoing-requests-card)カードに表示します。

オプションで、遅い送信リクエストのしきい値、[サンプリングレート](#sampling)、および無視するURLパターンを調整できます。

他の送信リクエストよりも時間がかかることが予想されるリクエストがある場合、リクエストごとのしきい値を設定できます：

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

正規表現パターンがリクエストのURLに一致しない場合、`'default'` 値が使用されます。

また、URLのグループ化を設定して、類似のURLを単一のエントリとしてグループ化することもできます。たとえば、URLパスから一意のIDを削除したり、ドメインのみでグループ化したりすることができます。グループは、正規表現を使用してURLの一部を「検索して置換」することで設定されます。設定ファイルにいくつかの例が含まれています：

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

最初に一致したパターンが使用されます。パターンが一致しない場合、URLはそのままキャプチャされます。

<a name="slow-queries-recorder"></a>
#### 遅いクエリ

`SlowQueries` レコーダーは、アプリケーション内の設定されたしきい値を超えるデータベースクエリに関する情報をキャプチャし、[遅いクエリ](#slow-queries-card)カードに表示します。

オプションで、遅いクエリのしきい値、[サンプリングレート](#sampling)、および無視するクエリパターンを調整できます。また、クエリの場所をキャプチャするかどうかを設定することもできます。キャプチャされた場所はPulseダッシュボードに表示され、クエリの発生元を追跡するのに役立ちます。ただし、同じクエリが複数の場所で実行された場合、それぞれの一意の場所ごとに複数回表示されます。

他のクエリよりも時間がかかることが予想されるクエリがある場合、クエリごとのしきい値を設定できます：

```php
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

正規表現パターンがクエリのSQLに一致しない場合、`'default'` 値が使用されます。

<a name="slow-requests-recorder"></a>
#### 遅いリクエスト

`Requests` レコーダーは、アプリケーションに対して行われたリクエストに関する情報をキャプチャし、[遅いリクエスト](#slow-requests-card)カードと[アプリケーション使用状況](#application-usage-card)カードに表示します。

オプションで、遅いルートのしきい値、[サンプリングレート](#sampling)、および無視するパスを調整できます。

他のリクエストよりも時間がかかることが予想されるリクエストがある場合、リクエストごとのしきい値を設定できます：

```php
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

正規表現パターンがリクエストのURLに一致しない場合、`'default'` 値が使用されます。

<a name="servers-recorder"></a>
#### サーバー

`Servers` レコーダーは、アプリケーションを動かすサーバーのCPU、メモリ、ストレージの使用状況をキャプチャし、[サーバー](#servers-card)カードに表示します。このレコーダーは、監視したい各サーバーで[`pulse:check` コマンド](#capturing-entries)を実行する必要があります。

各レポートサーバーには一意の名前が必要です。デフォルトでは、PulseはPHPの `gethostname` 関数によって返される値を使用します。これをカスタマイズしたい場合は、`PULSE_SERVER_NAME` 環境変数を設定できます：

```env
PULSE_SERVER_NAME=load-balancer
```

Pulse設定ファイルでは、監視するディレクトリをカスタマイズすることもできます。

<a name="user-jobs-recorder"></a>
#### ユーザージョブ

`UserJobs` レコーダーは、アプリケーションでジョブをディスパッチするユーザーに関する情報をキャプチャし、[アプリケーション使用状況](#application-usage-card)カードに表示します。

オプションで、[サンプリングレート](#sampling)と無視するジョブパターンを調整できます。

<a name="user-requests-recorder"></a>
#### ユーザーリクエスト

`UserRequests` レコーダーは、アプリケーションにリクエストを行うユーザーに関する情報をキャプチャし、[アプリケーション使用状況](#application-usage-card)カードに表示します。

オプションで、[サンプリングレート](#sampling)と無視するジョブパターンを調整できます。

<a name="filtering"></a>
### フィルタリング

見てきたように、多くの[レコーダー](#recorders)は、設定を通じて、リクエストのURLなどの値に基づいて受信エントリを「無視」する機能を提供しています。しかし、現在認証されているユーザーなど、他の要因に基づいてレコードをフィルタリングすることが有用な場合があります。これらのレコードをフィルタリングするには、クロージャをPulseの `filter` メソッドに渡すことができます。通常、`filter` メソッドはアプリケーションの `AppServiceProvider` の `boot` メソッド内で呼び出されるべきです：

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

// ...
```

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

<a name="performance"></a>
## パフォーマンス

Pulseは、既存のアプリケーションに組み込むために設計されており、追加のインフラストラクチャを必要としません。ただし、高トラフィックのアプリケーションの場合、Pulseがアプリケーションのパフォーマンスに与える影響を排除するためのいくつかの方法があります。

<a name="using-a-different-database"></a>
### 別のデータベースの使用

高トラフィックのアプリケーションでは、アプリケーションデータベースに影響を与えないように、Pulse用に専用のデータベース接続を使用することをお勧めします。

`PULSE_DB_CONNECTION`環境変数を設定することで、Pulseが使用する[データベース接続](database.md#configuration)をカスタマイズできます。

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Redisによる取り込み

> WARNING:  
> Redisによる取り込みには、Redis 6.2以上と、アプリケーションの設定済みRedisクライアントドライバとして`phpredis`または`predis`が必要です。

デフォルトでは、PulseはHTTPレスポンスがクライアントに送信された後、またはジョブが処理された後に、[設定されたデータベース接続](#using-a-different-database)に直接エントリを保存します。ただし、PulseのRedis取り込みドライバを使用して、エントリをRedisストリームに送信することができます。これは、`PULSE_INGEST_DRIVER`環境変数を設定することで有効にできます：

```
PULSE_INGEST_DRIVER=redis
```

Pulseはデフォルトで[Redis接続](redis.md#configuration)を使用しますが、`PULSE_REDIS_CONNECTION`環境変数を介してこれをカスタマイズできます：

```
PULSE_REDIS_CONNECTION=pulse
```

Redis取り込みを使用する場合、ストリームを監視し、RedisからPulseのデータベーステーブルにエントリを移動するために、`pulse:work`コマンドを実行する必要があります。

```php
php artisan pulse:work
```

> NOTE:  
> `pulse:work`プロセスをバックグラウンドで永続的に実行させるには、Supervisorなどのプロセスモニタを使用して、Pulseワーカーが停止しないようにする必要があります。

`pulse:work`コマンドは長時間実行されるプロセスであるため、再起動しない限りコードベースの変更を認識しません。アプリケーションのデプロイプロセス中に`pulse:restart`コマンドを呼び出して、コマンドを正常に再起動する必要があります：

```sh
php artisan pulse:restart
```

> NOTE:  
> Pulseは再起動シグナルを保存するために[キャッシュ](cache.md)を使用するため、この機能を使用する前に、アプリケーションに対してキャッシュドライバが適切に設定されていることを確認する必要があります。

<a name="sampling"></a>
### サンプリング

デフォルトでは、Pulseはアプリケーションで発生するすべての関連イベントをキャプチャします。高トラフィックのアプリケーションでは、特に長期間の場合、ダッシュボードで数百万のデータベース行を集計する必要があることになります。

代わりに、特定のPulseデータレコーダーで「サンプリング」を有効にすることを選択できます。たとえば、[`User Requests`](#user-requests-recorder)レコーダーのサンプルレートを`0.1`に設定すると、アプリケーションへのリクエストの約10%のみが記録されます。ダッシュボードでは、値はスケールアップされ、近似値であることを示すために`~`が前に付けられます。

一般に、特定のメトリックに対してより多くのエントリがあるほど、精度を大幅に犠牲にすることなくサンプルレートを安全に低く設定できます。

<a name="trimming"></a>
### トリミング

Pulseは、ダッシュボードのウィンドウ外にあるエントリが保存されると、自動的にそれらをトリミングします。トリミングは、Pulseの[設定ファイル](#configuration)でカスタマイズ可能な抽選システムを使用してデータを取り込む際に発生します。

<a name="pulse-exceptions"></a>
### Pulse例外の処理

Pulseデータのキャプチャ中に例外が発生した場合、例えばストレージデータベースに接続できない場合、Pulseはアプリケーションに影響を与えないように静かに失敗します。

これらの例外の処理方法をカスタマイズしたい場合は、`handleExceptionsUsing`メソッドにクロージャを提供できます：

```php
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

<a name="custom-cards"></a>
## カスタムカード

Pulseでは、アプリケーションの特定のニーズに関連するデータを表示するためのカスタムカードを構築できます。Pulseは[Livewire](https://livewire.laravel.com)を使用しているため、最初のカスタムカードを構築する前に[そのドキュメント](https://livewire.laravel.com/docs)を確認することをお勧めします。

<a name="custom-card-components"></a>
### カードコンポーネント

Laravel Pulseでカスタムカードを作成するには、基本の`Card` Livewireコンポーネントを拡張し、対応するビューを定義することから始めます：

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

Livewireの[遅延読み込み](https://livewire.laravel.com/docs/lazy)機能を使用する場合、`Card`コンポーネントは自動的にプレースホルダを提供し、コンポーネントに渡される`cols`と`rows`属性を尊重します。

Pulseカードの対応するビューを記述する際には、一貫した外観と操作性のためにPulseのBladeコンポーネントを活用できます：

```blade
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

`$cols`、`$rows`、`$class`、および`$expand`変数は、それぞれのBladeコンポーネントに渡される必要があります。これにより、ダッシュボードビューからカードのレイアウトをカスタマイズできます。また、カードを自動的に更新するために、ビューに`wire:poll.5s=""`属性を含めることもできます。

Livewireコンポーネントとテンプレートを定義したら、カードを[ダッシュボードビュー](#dashboard-customization)に含めることができます：

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> NOTE:  
> カードがパッケージに含まれている場合、`Livewire::component`メソッドを使用してコンポーネントをLivewireに登録する必要があります。

<a name="custom-card-styling"></a>
### スタイリング

カードにPulseに含まれるクラスやコンポーネント以外の追加のスタイリングが必要な場合、カスタムCSSを含めるためのいくつかのオプションがあります。

<a name="custom-card-styling-vite"></a>
#### Laravel Vite統合

カスタムカードがアプリケーションのコードベース内にあり、Laravelの[Vite統合](vite.md)を使用している場合、`vite.config.js`ファイルを更新して、カード用の専用CSSエントリポイントを含めることができます：

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

その後、[ダッシュボードビュー](#dashboard-customization)で`@vite` Bladeディレクティブを使用し、カードのCSSエントリポイントを指定できます：

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### CSSファイル

他のユースケース（Pulseカードがパッケージに含まれている場合を含む）では、Livewireコンポーネントに`css`メソッドを定義して、CSSファイルのファイルパスを返すことで、Pulseに追加のスタイルシートをロードするよう指示できます：

```php
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

このカードがダッシュボードに含まれる場合、Pulseは自動的にこのファイルの内容を`<style>`タグ内に含めるため、`public`ディレクトリに公開する必要はありません。

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

Tailwind CSSを使用する場合、不要なCSSの読み込みやPulseのTailwindクラスとの競合を避けるために、専用のTailwind設定ファイルを作成する必要があります：

```js
export default {
    darkMode: 'class',
    important: '#top-sellers',
    content: [
        './resources/views/livewire/pulse/top-sellers.blade.php',
    ],
    corePlugins: {
        preflight: false,
    },
};
```

その後、CSSエントリポイントで設定ファイルを指定できます：

```css
@config "../../tailwind.top-sellers.config.js";
@tailwind base;
@tailwind components;
@tailwind utilities;
```

また、Tailwindの[`important`セレクタ戦略](https://tailwindcss.com/docs/configuration#selector-strategy)に渡されるセレクタに一致する`id`または`class`属性をカードのビューに含める必要があります：

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

<a name="custom-card-data"></a>
### データのキャプチャと集計

カスタムカードはどこからでもデータを取得して表示できますが、Pulseの強力で効率的なデータ記録と集計システムを活用したい場合があります。

<a name="custom-card-data-capture"></a>
#### エントリのキャプチャ

Pulseでは、`Pulse::record`メソッドを使用して「エントリ」を記録できます：

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

`record`メソッドに提供される最初の引数は、記録しているエントリの`type`であり、2番目の引数は集計データをグループ化する方法を決定する`key`です。ほとんどの集計メソッドでは、集計する`value`も指定する必要があります。上記の例では、集計される値は`$sale->amount`です。その後、Pulseが後で効率的に取得できるように、事前に集計された値を「バケット」にキャプチャするために、1つ以上の集計メソッド（`sum`など）を呼び出すことができます。

利用可能な集計メソッドは以下の通りです：

* `avg`
* `count`
* `max`
* `min`
* `sum`

> NOTE:  
> 現在認証されているユーザーIDをキャプチャするカードパッケージを構築する場合、アプリケーションに対して行われた[ユーザーリゾルバのカスタマイズ](#dashboard-resolving-users)を尊重する`Pulse::resolveAuthenticatedUserId()`メソッドを使用する必要があります。

<a name="custom-card-data-retrieval"></a>
#### 集計データの取得

Pulseの`Card` Livewireコンポーネントを拡張する際、ダッシュボードで表示されている期間の集計データを取得するために`aggregate`メソッドを使用できます：

```php
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])
        ]);
    }
}
```

`aggregate`メソッドはPHPの`stdClass`オブジェクトのコレクションを返します。各オブジェクトには、以前にキャプチャされた`key`プロパティと、要求された各集計のキーが含まれます：

```
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulseは主に事前集計されたバケットからデータを取得します。したがって、指定された集計は`Pulse::record`メソッドを使用して事前にキャプチャされている必要があります。最も古いバケットは通常、期間の一部を外れているため、Pulseは最も古いエントリを集計してギャップを埋め、各ポーリングリクエストで期間全体を集計することなく、期間全体の正確な値を提供します。

また、`aggregateTotal`メソッドを使用して、特定のタイプの合計値を取得することもできます。例えば、以下のメソッドはユーザーごとにグループ化するのではなく、すべてのユーザー売上の合計を取得します。

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### ユーザーの表示

キーとしてユーザーIDを記録する集計を扱う場合、`Pulse::resolveUsers`メソッドを使用してキーをユーザーレコードに解決できます：

```php
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

`find`メソッドは`name`、`extra`、`avatar`キーを含むオブジェクトを返します。これらのキーは、オプションで`<x-pulse::user-card>` Bladeコンポーネントに直接渡すことができます：

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### カスタムレコーダー

パッケージの作者は、ユーザーがデータのキャプチャを設定できるようにレコーダークラスを提供することを望むかもしれません。

レコーダーは、アプリケーションの`config/pulse.php`設定ファイルの`recorders`セクションに登録されます：

```php
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

レコーダーは`$listen`プロパティを指定することでイベントをリッスンできます。Pulseは自動的にリスナーを登録し、レコーダーの`record`メソッドを呼び出します：

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * リッスンするイベント。
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * デプロイメントを記録する。
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```
