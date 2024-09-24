# Laravel Octane

- [はじめに](#introduction)
- [インストール](#installation)
- [サーバーの前提条件](#server-prerequisites)
    - [FrankenPHP](#frankenphp)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [アプリケーションの提供](#serving-your-application)
    - [HTTPS経由でのアプリケーションの提供](#serving-your-application-via-https)
    - [Nginx経由でのアプリケーションの提供](#serving-your-application-via-nginx)
    - [ファイル変更の監視](#watching-for-file-changes)
    - [ワーカー数の指定](#specifying-the-worker-count)
    - [最大リクエスト数の指定](#specifying-the-max-request-count)
    - [ワーカーのリロード](#reloading-the-workers)
    - [サーバーの停止](#stopping-the-server)
- [依存性注入とOctane](#dependency-injection-and-octane)
    - [コンテナの注入](#container-injection)
    - [リクエストの注入](#request-injection)
    - [設定リポジトリの注入](#configuration-repository-injection)
- [メモリリークの管理](#managing-memory-leaks)
- [並行タスク](#concurrent-tasks)
- [ティックとインターバル](#ticks-and-intervals)
- [Octaneキャッシュ](#the-octane-cache)
- [テーブル](#tables)

<a name="introduction"></a>
## はじめに

[Laravel Octane](https://github.com/laravel/octane) は、[FrankenPHP](https://frankenphp.dev/)、[Open Swoole](https://openswoole.com/)、[Swoole](https://github.com/swoole/swoole-src)、[RoadRunner](https://roadrunner.dev) などの高性能アプリケーションサーバーを使用して、アプリケーションのパフォーマンスを向上させます。Octaneはアプリケーションを一度起動し、メモリ内に保持し、超高速でリクエストを処理します。

<a name="installation"></a>
## インストール

OctaneはComposerパッケージマネージャーを介してインストールできます:

```shell
composer require laravel/octane
```

Octaneをインストールした後、`octane:install` Artisanコマンドを実行して、Octaneの設定ファイルをアプリケーションにインストールできます:

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## サーバーの前提条件

> WARNING:  
> Laravel Octane は [PHP 8.1+](https://php.net/releases/) を必要とします。

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev) は、Goで書かれたPHPアプリケーションサーバーで、early hints、Brotli、Zstandard圧縮などの最新のWeb機能をサポートしています。Octaneをインストールし、FrankenPHPをサーバーとして選択すると、Octaneは自動的にFrankenPHPバイナリをダウンロードしてインストールします。

<a name="frankenphp-via-laravel-sail"></a>
#### Laravel Sail経由のFrankenPHP

[Laravel Sail](sail.md)を使用してアプリケーションを開発する場合、以下のコマンドを実行してOctaneとFrankenPHPをインストールする必要があります:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

次に、`octane:install` Artisanコマンドを使用してFrankenPHPバイナリをインストールする必要があります:

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

最後に、アプリケーションの`docker-compose.yml`ファイルの`laravel.test`サービス定義に`SUPERVISOR_PHP_COMMAND`環境変数を追加します。この環境変数には、SailがPHP開発サーバーの代わりにOctaneを使用してアプリケーションを提供するために使用するコマンドが含まれます:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port=80" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

HTTPS、HTTP/2、HTTP/3を有効にするには、代わりにこれらの変更を適用します:

```yaml
services:
  laravel.test:
    ports:
        - '${APP_PORT:-80}:80'
        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        - '443:443' # [tl! add]
        - '443:443/udp' # [tl! add]
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

通常、FrankenPHP Sailアプリケーションには`https://localhost`経由でアクセスする必要があります。`https://127.0.0.1`を使用するには追加の設定が必要であり、[推奨されません](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker)。

<a name="frankenphp-via-docker"></a>
#### Docker経由のFrankenPHP

FrankenPHPの公式Dockerイメージを使用すると、パフォーマンスが向上し、FrankenPHPの静的インストールに含まれていない追加の拡張機能を使用できます。また、公式Dockerイメージは、FrankenPHPがネイティブにサポートしていないプラットフォーム（Windowsなど）での実行をサポートしています。FrankenPHPの公式Dockerイメージは、ローカル開発と本番環境の両方に適しています。

FrankenPHPを搭載したLaravelアプリケーションをコンテナ化するための出発点として、以下のDockerfileを使用できます:

```dockerfile
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # 他のPHP拡張機能をここに追加...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

次に、開発中に以下のDocker Composeファイルを使用してアプリケーションを実行できます:

```yaml
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

FrankenPHPをDockerで実行する方法の詳細については、[公式のFrankenPHPドキュメント](https://frankenphp.dev/docs/docker/)を参照してください。

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) は、Goで構築されたRoadRunnerバイナリによって動作します。RoadRunnerベースのOctaneサーバーを初めて起動すると、OctaneはRoadRunnerバイナリのダウンロードとインストールを提案します。

<a name="roadrunner-via-laravel-sail"></a>
#### Laravel Sail経由のRoadRunner

[Laravel Sail](sail.md)を使用してアプリケーションを開発する場合、以下のコマンドを実行してOctaneとRoadRunnerをインストールする必要があります:

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
```

次に、Sailシェルを起動し、`rr`実行可能ファイルを使用して最新のLinuxベースのRoadRunnerバイナリを取得する必要があります:

```shell
./vendor/bin/sail shell

# Sailシェル内で...
./vendor/bin/rr get-binary
```

次に、アプリケーションの`docker-compose.yml`ファイルの`laravel.test`サービス定義に`SUPERVISOR_PHP_COMMAND`環境変数を追加します。この環境変数には、SailがPHP開発サーバーの代わりにOctaneを使用してアプリケーションを提供するために使用するコマンドが含まれます:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port=80" # [tl! add]
```

最後に、`rr`バイナリが実行可能であることを確認し、Sailイメージをビルドします:

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

Laravel Octaneアプリケーションを提供するためにSwooleアプリケーションサーバーを使用する場合、Swoole PHP拡張機能をインストールする必要があります。通常、これはPECLを介して行うことができます:

```shell
pecl install swoole
```

<a name="openswoole"></a>
#### Open Swoole

Laravel Octaneアプリケーションを提供するためにOpen Swooleアプリケーションサーバーを使用する場合、Open Swoole PHP拡張機能をインストールする必要があります。通常、これはPECLを介して行うことができます:

```shell
pecl install openswoole
```

Open Swooleを使用してLaravel Octaneを実行すると、Swooleによって提供される並行タスク、ティック、インターバルなどの同じ機能が提供されます。

<a name="swoole-via-laravel-sail"></a>
#### Laravel Sail経由のSwoole

> WARNING:  
> Sailを介してOctaneアプリケーションを提供する前に、最新バージョンのLaravel Sailがあることを確認し、アプリケーションのルートディレクトリ内で`./vendor/bin/sail build --no-cache`を実行してください。

または、[Laravel Sail](sail.md)を使用してSwooleベースのOctaneアプリケーションを開発することもできます。Laravel SailにはデフォルトでSwoole拡張機能が含まれています。ただし、Sailが使用する`docker-compose.yml`ファイルを調整する必要があります。

開始するには、アプリケーションの`docker-compose.yml`ファイルの`laravel.test`サービス定義に`SUPERVISOR_PHP_COMMAND`環境変数を追加します。この環境変数には、SailがPHP開発サーバーの代わりにOctaneを使用してアプリケーションを提供するために使用するコマンドが含まれます:

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=80" # [tl! add]
```

最後に、Sailイメージをビルドします:

```shell
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Swooleの設定

Swooleは、必要に応じて`octane`設定ファイルに追加できるいくつかの追加設定オプションをサポートしています。これらのオプションはほとんど変更する必要がないため、デフォルトの設定ファイルには含まれていません:

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## アプリケーションの提供

Octaneサーバーは、`octane:start` Artisanコマンドを介して起動できます。デフォルトでは、このコマンドはアプリケーションの`octane`設定ファイルの`server`設定オプションで指定されたサーバーを使用します:

```shell
php artisan octane:start
```

デフォルトでは、Octaneはサーバーをポート8000で起動します。そのため、`http://localhost:8000`を介してWebブラウザでアプリケーションにアクセスできます。

<a name="serving-your-application-via-https"></a>
### HTTPS経由でのアプリケーションの提供

デフォルトでは、Octaneを介して実行されるアプリケーションは、`http://`がプレフィックスされたリンクを生成します。アプリケーションの`config/octane.php`設定ファイル内で使用される`OCTANE_HTTPS`環境変数は、アプリケーションをHTTPS経由で提供する場合に`true`に設定できます。この設定値が`true`に設定されている場合、OctaneはLaravelにすべての生成されたリンクに`https://`をプレフィックスするよう指示します。

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### Nginx経由でのアプリケーションの提供

> NOTE:  
> まだ自分のサーバー設定を管理する準備ができていない場合、または、堅牢なLaravel Octaneアプリケーションを実行するために必要なさまざまなサービスを設定するのに慣れていない場合は、[Laravel Forge](https://forge.laravel.com)をチェックしてください。

本番環境では、NginxやApacheなどの従来のWebサーバーの背後でOctaneアプリケーションを提供する必要があります。これにより、Webサーバーは画像やスタイルシートなどの静的アセットを提供し、SSL証明書の終端処理を管理できます。

以下のNginx設定例では、Nginxはサイトの静的アセットを提供し、ポート8000で実行されているOctaneサーバーにリクエストをプロキシします。

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-for-file-changes"></a>
### ファイル変更の監視

Octaneはアプリケーションを一度メモリにロードし、リクエストを提供しながらそのまま保持するため、アプリケーションのファイルに加えられた変更は、ブラウザをリフレッシュしても反映されません。たとえば、`routes/web.php`ファイルに追加されたルート定義は、サーバーが再起動されるまで反映されません。便宜上、`--watch`フラグを使用して、アプリケーション内のファイルに変更があるたびにOctaneにサーバーを自動的に再起動するよう指示できます。

```shell
php artisan octane:start --watch
```

この機能を使用する前に、ローカル開発環境に[Node](https://nodejs.org)がインストールされていることを確認する必要があります。さらに、プロジェクト内に[Chokidar](https://github.com/paulmillr/chokidar)ファイル監視ライブラリをインストールする必要があります。

```shell
npm install --save-dev chokidar
```

アプリケーションの`config/octane.php`設定ファイル内の`watch`設定オプションを使用して、監視するディレクトリとファイルを設定できます。

<a name="specifying-the-worker-count"></a>
### ワーカー数の指定

デフォルトでは、Octaneはマシンが提供する各CPUコアに対して1つのアプリケーションリクエストワーカーを起動します。これらのワーカーは、アプリケーションに入ってくるHTTPリクエストを処理するために使用されます。`octane:start`コマンドを呼び出す際に`--workers`オプションを使用して、起動したいワーカー数を手動で指定できます。

```shell
php artisan octane:start --workers=4
```

Swooleアプリケーションサーバーを使用している場合、起動したい["タスクワーカー"](#concurrent-tasks)の数も指定できます。

```shell
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### 最大リクエスト数の指定

メモリリークを防ぐために、Octaneは500リクエストを処理した後にワーカーを正常に再起動します。この数値を調整するには、`--max-requests`オプションを使用します。

```shell
php artisan octane:start --max-requests=250
```

<a name="reloading-the-workers"></a>
### ワーカーの再読み込み

Octaneサーバーのアプリケーションワーカーを正常に再起動するには、`octane:reload`コマンドを使用します。通常、これは新しくデプロイされたコードがメモリにロードされ、後続のリクエストに使用されるようにするためにデプロイ後に行われます。

```shell
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### サーバーの停止

Octaneサーバーを停止するには、`octane:stop` Artisanコマンドを使用します。

```shell
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### サーバーステータスの確認

Octaneサーバーの現在のステータスを確認するには、`octane:status` Artisanコマンドを使用します。

```shell
php artisan octane:status
```

<a name="dependency-injection-and-octane"></a>
## 依存性注入とOctane

Octaneはアプリケーションを一度起動し、リクエストを提供しながらそのままメモリに保持するため、アプリケーションを構築する際に考慮すべきいくつかの注意点があります。たとえば、アプリケーションのサービスプロバイダの`register`および`boot`メソッドは、リクエストワーカーが最初に起動したときにのみ実行されます。後続のリクエストでは、同じアプリケーションインスタンスが再利用されます。

この点を考慮して、アプリケーションサービスコンテナまたはリクエストを他のオブジェクトのコンストラクタに注入する際には特に注意が必要です。そうすることで、そのオブジェクトは後続のリクエストで古いバージョンのコンテナまたはリクエストを持つ可能性があります。

Octaneは、リクエスト間でフレームワークのファーストパーティの状態を自動的にリセットします。ただし、Octaneは常にアプリケーションによって作成されたグローバル状態をリセットする方法を知っているわけではありません。したがって、Octaneに適した方法でアプリケーションを構築する方法について注意する必要があります。以下では、Octaneを使用する際に問題を引き起こす可能性のある最も一般的な状況について説明します。

<a name="container-injection"></a>
### コンテナの注入

一般的に、アプリケーションサービスコンテナまたはHTTPリクエストインスタンスを他のオブジェクトのコンストラクタに注入することは避けるべきです。たとえば、以下のバインディングは、アプリケーションサービスコンテナ全体をシングルトンとしてバインドされたオブジェクトに注入します。

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * アプリケーションサービスの登録
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

この例では、`Service`インスタンスがアプリケーションのブートプロセス中に解決されると、コンテナがサービスに注入され、その同じコンテナが後続のリクエストで`Service`インスタンスに保持されます。これは、特定のアプリケーションにとって問題にならない場合がありますが、ブートサイクルの後半または後続のリクエストによって追加されたバインディングがコンテナに予期せず欠落する可能性があります。

回避策として、バインディングをシングルトンとして登録しないようにするか、常に最新のコンテナインスタンスを解決するコンテナリゾルバクロージャをサービスに注入することができます。

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

グローバルな`app`ヘルパーと`Container::getInstance()`メソッドは、常に最新バージョンのアプリケーションコンテナを返します。

<a name="request-injection"></a>
### リクエストの注入

一般的に、アプリケーションサービスコンテナまたはHTTPリクエストインスタンスを他のオブジェクトのコンストラクタに注入することは避けるべきです。たとえば、以下のバインディングは、リクエストインスタンス全体をシングルトンとしてバインドされたオブジェクトに注入します。

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * アプリケーションサービスの登録
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

この例では、`Service`インスタンスがアプリケーションのブートプロセス中に解決されると、HTTPリクエストがサービスに注入され、その同じリクエストが後続のリクエストで`Service`インスタンスに保持されます。したがって、すべてのヘッダー、入力、クエリ文字列データ、およびその他のリクエストデータが不正確になります。

回避策として、バインディングをシングルトンとして登録しないようにするか、常に最新のリクエストインスタンスを解決するリクエストリゾルバクロージャをサービスに注入することができます。または、最も推奨されるアプローチは、オブジェクトのメソッドに実行時に必要な特定のリクエスト情報を渡すことです。

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// または...

$service->method($request->input('name'));
```

グローバルな `request` ヘルパーは、常にアプリケーションが現在処理しているリクエストを返すため、アプリケーション内で安全に使用できます。

> WARNING:  
> コントローラーメソッドやルートクロージャーに `Illuminate\Http\Request` インスタンスをタイプヒントすることは許容されています。

<a name="configuration-repository-injection"></a>
### 設定リポジトリのインジェクション

一般的に、他のオブジェクトのコンストラクターに設定リポジトリインスタンスを注入することは避けるべきです。例えば、以下のバインディングは、設定リポジトリをシングルトンとしてバインドされたオブジェクトに注入しています。

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * アプリケーションサービスの登録
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

この例では、リクエスト間で設定値が変更された場合、そのサービスは元のリポジトリインスタンスに依存しているため、新しい値にアクセスできません。

回避策として、バインディングをシングルトンとして登録しないようにするか、クラスに設定リポジトリリゾルバークロージャーを注入することができます。

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

グローバルな `config` は常に設定リポジトリの最新バージョンを返すため、アプリケーション内で安全に使用できます。

<a name="managing-memory-leaks"></a>
### メモリリークの管理

Octane はリクエスト間でアプリケーションをメモリ内に保持することを忘れないでください。したがって、静的に維持される配列にデータを追加すると、メモリリークが発生します。例えば、以下のコントローラーは、アプリケーションへの各リクエストが静的な `$data` 配列にデータを追加し続けるため、メモリリークが発生します。

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * 受信リクエストの処理
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

アプリケーションを構築する際には、このようなメモリリークを作成しないように特に注意する必要があります。ローカル開発中にアプリケーションのメモリ使用量を監視して、新しいメモリリークがアプリケーションに導入されていないことを確認することをお勧めします。

<a name="concurrent-tasks"></a>
## 並行タスク

> WARNING:  
> この機能には [Swoole](#swoole) が必要です。

Swoole を使用する場合、軽量のバックグラウンドタスクを介して操作を並行して実行できます。これは、Octane の `concurrently` メソッドを使用して実現できます。このメソッドを PHP の配列分解と組み合わせて、各操作の結果を取得できます。

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Octane によって処理される並行タスクは、Swoole の「タスクワーカー」を利用し、受信リクエストとは完全に異なるプロセス内で実行されます。並行タスクを処理するために利用可能なワーカーの数は、`octane:start` コマンドの `--task-workers` ディレクティブによって決定されます。

```shell
php artisan octane:start --workers=4 --task-workers=6
```

`concurrently` メソッドを呼び出す際には、Swoole のタスクシステムによって課される制限により、1024 タスク以上を提供しないでください。

<a name="ticks-and-intervals"></a>
## ティックとインターバル

> WARNING:  
> この機能には [Swoole](#swoole) が必要です。

Swoole を使用する場合、指定された秒数ごとに実行される「ティック」操作を登録できます。`tick` メソッドを介して「ティック」コールバックを登録できます。`tick` メソッドに提供される最初の引数は、ティッカーの名前を表す文字列である必要があります。2 番目の引数は、指定された間隔で呼び出される callable である必要があります。

この例では、10 秒ごとに呼び出されるクロージャーを登録します。通常、`tick` メソッドはアプリケーションのサービスプロバイダーの `boot` メソッド内で呼び出されるべきです。

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10);
```

`immediate` メソッドを使用すると、Octane サーバーが最初に起動したときにティックコールバックを即座に呼び出し、その後 N 秒ごとに呼び出すように Octane に指示できます。

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10)
        ->immediate();
```

<a name="the-octane-cache"></a>
## Octane キャッシュ

> WARNING:  
> この機能には [Swoole](#swoole) が必要です。

Swoole を使用する場合、Octane キャッシュドライバーを利用できます。このキャッシュドライバーは、最大で 200 万回の操作/秒の読み取りと書き込み速度を提供します。したがって、キャッシュ層から極端な読み取り/書き込み速度が必要なアプリケーションにとって、このキャッシュドライバーは優れた選択肢です。

このキャッシュドライバーは [Swoole テーブル](https://www.swoole.co.uk/docs/modules/swoole-table) によって動力を得ています。キャッシュに保存されたすべてのデータは、サーバー上のすべてのワーカーで利用可能です。ただし、キャッシュされたデータはサーバーが再起動されるとフラッシュされます。

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> NOTE:  
> Octane キャッシュで許可される最大エントリ数は、アプリケーションの `octane` 設定ファイルで定義できます。

<a name="cache-intervals"></a>
### キャッシュインターバル

Laravel のキャッシュシステムが提供する一般的なメソッドに加えて、Octane キャッシュドライバーはインターバルベースのキャッシュを備えています。これらのキャッシュは指定されたインターバルで自動的にリフレッシュされ、アプリケーションのサービスプロバイダーの `boot` メソッド内で登録する必要があります。例えば、以下のキャッシュは 5 秒ごとにリフレッシュされます。

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## テーブル

> WARNING:  
> この機能には [Swoole](#swoole) が必要です。

Swoole を使用する場合、独自の任意の [Swoole テーブル](https://www.swoole.co.uk/docs/modules/swoole-table) を定義して操作できます。Swoole テーブルは極端なパフォーマンススループットを提供し、これらのテーブル内のデータはサーバー上のすべてのワーカーでアクセスできます。ただし、これらのデータはサーバーが再起動されると失われます。

テーブルは、アプリケーションの `octane` 設定ファイルの `tables` 設定配列内で定義する必要があります。最大 1000 行を許可する例のテーブルがすでに設定されています。文字列カラムの最大サイズは、以下に示すようにカラムタイプの後にカラムサイズを指定することで設定できます。

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

テーブルにアクセスするには、`Octane::table` メソッドを使用できます。

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> WARNING:  
> Swoole テーブルでサポートされているカラムタイプは、`string`、`int`、および `float` です。

