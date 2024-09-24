# デプロイ

- [はじめに](#introduction)
- [サーバー要件](#server-requirements)
- [サーバー設定](#server-configuration)
    - [Nginx](#nginx)
    - [FrankenPHP](#frankenphp)
    - [ディレクトリ権限](#directory-permissions)
- [最適化](#optimization)
    - [設定のキャッシュ](#optimizing-configuration-loading)
    - [イベントのキャッシュ](#caching-events)
    - [ルートのキャッシュ](#optimizing-route-loading)
    - [ビューのキャッシュ](#optimizing-view-loading)
- [デバッグモード](#debug-mode)
- [ヘルスルート](#the-health-route)
- [Forge / Vapor による簡単なデプロイ](#deploying-with-forge-or-vapor)

<a name="introduction"></a>
## はじめに

Laravelアプリケーションを本番環境にデプロイする準備ができたら、アプリケーションが可能な限り効率的に実行されるようにするために、いくつか重要なことを行うことができます。このドキュメントでは、Laravelアプリケーションが適切にデプロイされるようにするためのいくつかの優れた出発点を紹介します。

<a name="server-requirements"></a>
## サーバー要件

Laravelフレームワークにはいくつかのシステム要件があります。Webサーバーに以下の最小限のPHPバージョンと拡張機能があることを確認する必要があります。

<div class="content-list" markdown="1">

- PHP >= 8.2
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

</div>

<a name="server-configuration"></a>
## サーバー設定

<a name="nginx"></a>
### Nginx

アプリケーションをNginxを実行しているサーバーにデプロイする場合、Webサーバーの設定の出発点として以下の設定ファイルを使用できます。ほとんどの場合、このファイルはサーバーの設定に応じてカスタマイズする必要があります。**サーバーの管理を手助けしてほしい場合は、[Laravel Forge](https://forge.laravel.com)のようなLaravelの公式サーバー管理およびデプロイサービスの使用を検討してください。**

以下の設定のように、Webサーバーがすべてのリクエストをアプリケーションの`public/index.php`ファイルに転送するようにしてください。`index.php`ファイルをプロジェクトのルートに移動しようとしないでください。プロジェクトのルートからアプリケーションを提供すると、多くの機密設定ファイルが公開インターネットに公開されることになります。

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) もLaravelアプリケーションの提供に使用できます。FrankenPHPはGoで書かれたモダンなPHPアプリケーションサーバーです。FrankenPHPを使用してLaravel PHPアプリケーションを提供するには、単に`php-server`コマンドを呼び出すことができます。

```shell
frankenphp php-server -r public/
```

FrankenPHPのより強力な機能、例えば[Laravel Octane](octane.md)統合、HTTP/3、モダンな圧縮、Laravelアプリケーションをスタンドアロンバイナリとしてパッケージ化する機能などを利用するには、FrankenPHPの[Laravelドキュメント](https://frankenphp.dev/docs/laravel/)を参照してください。

<a name="directory-permissions"></a>
### ディレクトリ権限

Laravelは`bootstrap/cache`と`storage`ディレクトリに書き込みが必要なので、Webサーバープロセスの所有者がこれらのディレクトリに書き込み権限を持っていることを確認してください。

<a name="optimization"></a>
## 最適化

アプリケーションを本番環境にデプロイする際、設定、イベント、ルート、ビューなど、さまざまなファイルをキャッシュする必要があります。Laravelは、これらすべてのファイルをキャッシュするための単一の便利な`optimize` Artisanコマンドを提供します。このコマンドは通常、アプリケーションのデプロイプロセスの一部として呼び出されるべきです。

```shell
php artisan optimize
```

`optimize:clear`メソッドを使用して、`optimize`コマンドによって生成されたすべてのキャッシュファイルと、デフォルトのキャッシュドライバー内のすべてのキーを削除できます。

```shell
php artisan optimize:clear
```

以下のドキュメントでは、`optimize`コマンドによって実行される各粒度の最適化コマンドについて説明します。

<a name="optimizing-configuration-loading"></a>
### 設定のキャッシュ

アプリケーションを本番環境にデプロイする際、デプロイプロセス中に`config:cache` Artisanコマンドを実行する必要があります。

```shell
php artisan config:cache
```

このコマンドは、Laravelのすべての設定ファイルを単一のキャッシュファイルに結合し、フレームワークが設定値を読み込む際にファイルシステムにアクセスする回数を大幅に減らします。

> WARNING:  
> デプロイプロセス中に`config:cache`コマンドを実行する場合、設定ファイル内でのみ`env`関数を呼び出すようにしてください。設定がキャッシュされると、`.env`ファイルは読み込まれず、`.env`変数に対する`env`関数の呼び出しはすべて`null`を返します。

<a name="caching-events"></a>
### イベントのキャッシュ

デプロイプロセス中に、アプリケーションの自動検出されたイベントからリスナーへのマッピングをキャッシュする必要があります。これは、デプロイ中に`event:cache` Artisanコマンドを呼び出すことで実現できます。

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### ルートのキャッシュ

多くのルートを持つ大規模なアプリケーションを構築している場合、デプロイプロセス中に`route:cache` Artisanコマンドを実行する必要があります。

```shell
php artisan route:cache
```

このコマンドは、すべてのルート登録をキャッシュファイル内の単一のメソッド呼び出しに減らし、数百のルートを登録する際のパフォーマンスを向上させます。

<a name="optimizing-view-loading"></a>
### ビューのキャッシュ

アプリケーションを本番環境にデプロイする際、デプロイプロセス中に`view:cache` Artisanコマンドを実行する必要があります。

```shell
php artisan view:cache
```

このコマンドは、すべてのBladeビューを事前にコンパイルし、ビューが返されるたびにオンデマンドでコンパイルされないようにします。これにより、各リクエストのパフォーマンスが向上します。

<a name="debug-mode"></a>
## デバッグモード

`config/app.php`設定ファイルのdebugオプションは、エラーに関するどの程度の情報がユーザーに表示されるかを決定します。デフォルトでは、このオプションはアプリケーションの`.env`ファイルに保存されている`APP_DEBUG`環境変数の値を尊重するように設定されています。

> WARNING:  
> **本番環境では、この値は常に`false`である必要があります。本番環境で`APP_DEBUG`変数が`true`に設定されている場合、アプリケーションのエンドユーザーに機密設定値が公開されるリスクがあります。**

<a name="the-health-route"></a>
## ヘルスルート

Laravelには、アプリケーションのステータスを監視するために使用できる組み込みのヘルスチェックルートが含まれています。本番環境では、このルートを使用して、アプリケーションのステータスをアップタイムモニター、ロードバランサー、またはKubernetesなどのオーケストレーションシステムに報告できます。

デフォルトでは、ヘルスチェックルートは`/up`で提供され、アプリケーションが例外なく起動した場合は200 HTTPレスポンスを返します。それ以外の場合は500 HTTPレスポンスが返されます。このルートのURIは、アプリケーションの`bootstrap/app`ファイルで設定できます。

    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up', // [tl! remove]
        health: '/status', // [tl! add]
    )

このルートに対してHTTPリクエストが行われると、Laravelは`Illuminate\Foundation\Events\DiagnosingHealth`イベントをディスパッチし、アプリケーションに関連する追加のヘルスチェックを実行できるようにします。このイベントの[リスナー](events.md)内で、アプリケーションのデータベースやキャッシュのステータスをチェックできます。アプリケーションで問題を検出した場合、リスナーから例外を投げるだけです。

<a name="deploying-with-forge-or-vapor"></a>
## Forge / Vapor による簡単なデプロイ

<a name="laravel-forge"></a>
#### Laravel Forge

サーバーの設定を自分で管理する準備ができていない場合や、Laravelアプリケーションを実行するために必要なさまざまなサービスを設定するのに慣れていない場合、[Laravel Forge](https://forge.laravel.com)は素晴らしい代替手段です。

Laravel Forgeは、DigitalOcean、Linode、AWSなどのさまざまなインフラストラクチャプロバイダーでサーバーを作成できます。さらに、ForgeはNginx、MySQL、Redis、Memcached、Beanstalkなど、堅牢なLaravelアプリケーションを構築するために必要なすべてのツールをインストールおよび管理します。

> NOTE:  
> Laravel Forgeでのデプロイに関する完全なガイドが欲しいですか？[Laravel Bootcamp](https://bootcamp.laravel.com/deploying)とForgeの[Laracastsで利用可能なビデオシリーズ](https://laracasts.com/series/learn-laravel-forge-2022-edition)をチェックしてください。

<a name="laravel-vapor"></a>
#### Laravel Vapor

もしあなたがLaravel向けに調整された、完全サーバーレスで自動スケーリング可能なデプロイメントプラットフォームをお探しなら、[Laravel Vapor](https://vapor.laravel.com)をチェックしてみてください。Laravel Vaporは、AWSによって動力を与えられたLaravelのためのサーバーレスデプロイメントプラットフォームです。VaporでLaravelのインフラを立ち上げ、サーバーレスのスケーラブルなシンプルさに恋をしてください。Laravel Vaporは、Laravelのクリエイターによってフレームワークとシームレスに連携するように微調整されているため、今まで通りにLaravelアプリケーションを書き続けることができます。

