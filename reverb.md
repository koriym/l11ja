# Laravel Reverb

- [はじめに](#introduction)
- [インストール](#installation)
- [設定](#configuration)
    - [アプリケーションの認証情報](#application-credentials)
    - [許可されたオリジン](#allowed-origins)
    - [追加のアプリケーション](#additional-applications)
    - [SSL](#ssl)
- [サーバーの実行](#running-server)
    - [デバッグ](#debugging)
    - [再起動](#restarting)
- [監視](#monitoring)
- [本番環境でのReverbの実行](#production)
    - [オープンファイル](#open-files)
    - [イベントループ](#event-loop)
    - [Webサーバー](#web-server)
    - [ポート](#ports)
    - [プロセス管理](#process-management)
    - [スケーリング](#scaling)

<a name="introduction"></a>
## はじめに

[Laravel Reverb](https://github.com/laravel/reverb) は、Laravelアプリケーションに直接、高速でスケーラブルなリアルタイムWebSocket通信をもたらし、Laravelの既存の[イベントブロードキャストツール](broadcasting.md)とのシームレスな統合を提供します。

<a name="installation"></a>
## インストール

`install:broadcasting` Artisanコマンドを使用してReverbをインストールできます:

```
php artisan install:broadcasting
```

<a name="configuration"></a>
## 設定

内部的には、`install:broadcasting` Artisanコマンドは`reverb:install`コマンドを実行し、デフォルトの設定オプションでReverbをインストールします。設定を変更したい場合は、Reverbの環境変数を更新するか、`config/reverb.php`設定ファイルを更新することで行えます。

<a name="application-credentials"></a>
### アプリケーションの認証情報

Reverbへの接続を確立するためには、クライアントとサーバー間でReverbの「アプリケーション」認証情報のセットを交換する必要があります。これらの認証情報はサーバー上で設定され、クライアントからのリクエストを検証するために使用されます。以下の環境変数を使用してこれらの認証情報を定義できます:

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### 許可されたオリジン

クライアントリクエストが発信されるオリジンを定義することもできます。`config/reverb.php`設定ファイルの`apps`セクション内の`allowed_origins`設定値を更新することで行えます。許可されたオリジンにリストされていないオリジンからのリクエストは拒否されます。すべてのオリジンを許可するには`*`を使用できます:

```php
'apps' => [
    [
        'id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### 追加のアプリケーション

通常、Reverbはインストールされたアプリケーション用のWebSocketサーバーを提供します。しかし、単一のReverbインストールで複数のアプリケーションを提供することも可能です。

例えば、Reverbを介して複数のアプリケーションにWebSocket接続性を提供する単一のLaravelアプリケーションを維持したい場合、アプリケーションの`config/reverb.php`設定ファイルで複数の`apps`を定義することで実現できます:

```php
'apps' => [
    [
        'app_id' => 'my-app-one',
        // ...
    ],
    [
        'app_id' => 'my-app-two',
        // ...
    ],
],
```

<a name="ssl"></a>
### SSL

ほとんどの場合、安全なWebSocket接続はリクエストがReverbサーバーにプロキシされる前に、上流のWebサーバー（Nginxなど）によって処理されます。

しかし、ローカル開発中など、Reverbサーバーが直接安全な接続を処理することが有用な場合もあります。[Laravel Herd](https://herd.laravel.com)の安全なサイト機能を使用している場合、または[Laravel Valet](valet.md)を使用していて、アプリケーションに対して[secureコマンド](valet.md#securing-sites)を実行している場合、サイト用に生成されたHerd / Valet証明書を使用してReverb接続を保護できます。そのためには、`REVERB_HOST`環境変数をサイトのホスト名に設定するか、Reverbサーバーを起動する際にホスト名オプションを明示的に渡します:

```sh
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

HerdとValetドメインは`localhost`に解決されるため、上記のコマンドを実行すると、Reverbサーバーは安全なWebSocketプロトコル（`wss`）を介して`wss://laravel.test:8080`でアクセス可能になります。

また、アプリケーションの`config/reverb.php`設定ファイルで`tls`オプションを定義することで、証明書を手動で選択することもできます。`tls`オプションの配列内で、[PHPのSSLコンテキストオプション](https://www.php.net/manual/en/context.ssl.php)でサポートされている任意のオプションを提供できます:

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## サーバーの実行

Reverbサーバーは`reverb:start` Artisanコマンドを使用して起動できます:

```sh
php artisan reverb:start
```

デフォルトでは、Reverbサーバーは`0.0.0.0:8080`で起動し、すべてのネットワークインターフェースからアクセスできるようになります。

カスタムホストまたはポートを指定する必要がある場合は、サーバーを起動する際に`--host`および`--port`オプションを使用して指定できます:

```sh
php artisan reverb:start --host=127.0.0.1 --port=9000
```

または、アプリケーションの`.env`設定ファイルで`REVERB_SERVER_HOST`および`REVERB_SERVER_PORT`環境変数を定義することもできます。

`REVERB_SERVER_HOST`および`REVERB_SERVER_PORT`環境変数は、`REVERB_HOST`および`REVERB_PORT`と混同しないでください。前者はReverbサーバー自体が実行されるホストとポートを指定し、後者のペアはLaravelにブロードキャストメッセージを送信する場所を指示します。例えば、本番環境では、パブリックReverbホスト名のポート`443`から`0.0.0.0:8080`で動作するReverbサーバーにリクエストをルーティングする場合、環境変数は次のように定義されます:

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### デバッグ

パフォーマンスを向上させるため、Reverbはデフォルトでデバッグ情報を出力しません。Reverbサーバーを通過するデータのストリームを確認したい場合は、`reverb:start`コマンドに`--debug`オプションを提供できます:

```sh
php artisan reverb:start --debug
```

<a name="restarting"></a>
### 再起動

Reverbは長時間実行されるプロセスであるため、コードの変更はサーバーを再起動しない限り反映されません。`reverb:restart` Artisanコマンドを使用してサーバーを再起動します。

`reverb:restart`コマンドは、サーバーを停止する前にすべての接続が正常に終了することを保証します。SupervisorなどのプロセスマネージャでReverbを実行している場合、すべての接続が終了した後、プロセスマネージャによってサーバーが自動的に再起動されます:

```sh
php artisan reverb:restart
```

<a name="monitoring"></a>
## 監視

Reverbは[Laravel Pulse](pulse.md)との統合を介して監視できます。ReverbのPulse統合を有効にすることで、サーバーが処理している接続数とメッセージ数を追跡できます。

統合を有効にするには、まず[Pulseをインストール](pulse.md#installation)していることを確認してください。次に、Reverbのレコーダーをアプリケーションの`config/pulse.php`設定ファイルに追加します:

```php
use Laravel\Reverb\Pulse\Recorders\ReverbConnections;
use Laravel\Reverb\Pulse\Recorders\ReverbMessages;

'recorders' => [
    ReverbConnections::class => [
        'sample_rate' => 1,
    ],

    ReverbMessages::class => [
        'sample_rate' => 1,
    ],

    ...
],
```

次に、各レコーダーのPulseカードを[Pulseダッシュボード](pulse.md#dashboard-customization)に追加します:

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

<a name="production"></a>
## 本番環境でのReverbの実行

WebSocketサーバーの長時間実行の性質上、Reverbサーバーがサーバー上で利用可能なリソースに対して最適な接続数を効果的に処理できるように、サーバーとホスティング環境にいくつかの最適化を行う必要があるかもしれません。

> NOTE:  
> サイトが[Laravel Forge](https://forge.laravel.com)によって管理されている場合、「Application」パネルからReverbのためにサーバーを自動的に最適化できます。Reverb統合を有効にすると、Forgeはサーバーが本番環境に対応できるようにし、必要な拡張機能のインストールや許可された接続数の増加を含めます。

<a name="open-files"></a>
### オープンファイル

各WebSocket接続は、クライアントまたはサーバーが切断するまでメモリ内に保持されます。UnixおよびUnixライクな環境では、各接続はファイルによって表されます。しかし、オープンファイルの許可数には、オペレーティングシステムとアプリケーションレベルの両方で制限があります。

<a name="operating-system"></a>
#### オペレーティングシステム

Unixベースのオペレーティングシステムでは、`ulimit`コマンドを使用して許可されたオープンファイルの数を確認できます:

```sh
ulimit -n
```

このコマンドは、異なるユーザーに対して許可されたオープンファイルの制限を表示します。これらの値は、`/etc/security/limits.conf`ファイルを編集することで更新できます。例えば、`forge`ユーザーの最大オープンファイル数を10,000に更新する場合、次のようになります:

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### イベントループ

内部的には、ReverbはReactPHPイベントループを使用してサーバー上のWebSocket接続を管理します。デフォルトでは、このイベントループは`stream_select`によって動力を得ており、追加の拡張機能を必要としません。しかし、`stream_select`は通常1,024のオープンファイルに制限されています。そのため、1,000以上の同時接続を処理する予定がある場合、同じ制限に縛られない代替のイベントループを使用する必要があります。

Reverbは、利用可能な場合、`ext-uv`によって動力を得たループに自動的に切り替えます。このPHP拡張機能はPECLを介してインストールできます:

```sh
pecl install uv
```

<a name="web-server"></a>
### Webサーバー

ほとんどの場合、Reverbはサーバー上のWebに面していないポートで動作します。そのため、Reverbにトラフィックをルーティングするには、リバースプロキシを設定する必要があります。Reverbがホスト `0.0.0.0` とポート `8080` で動作しており、サーバーがNginx Webサーバーを使用していると仮定すると、以下のNginxサイト設定を使用してReverbサーバーのリバースプロキシを定義できます：

```nginx
server {
    ...

    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://0.0.0.0:8080;
    }

    ...
}
```

> WARNING:  
> ReverbはWebSocket接続を `/app` で、APIリクエストを `/apps` で待ち受けます。Reverbリクエストを処理するWebサーバーがこれらのURIを提供できることを確認する必要があります。[Laravel Forge](https://forge.laravel.com)を使用してサーバーを管理している場合、Reverbサーバーはデフォルトで正しく設定されます。

通常、Webサーバーはサーバーの過負荷を防ぐために、許可される接続数を制限するように設定されています。Nginx Webサーバーで許可される接続数を10,000に増やすには、`nginx.conf`ファイルの`worker_rlimit_nofile`と`worker_connections`の値を更新する必要があります：

```nginx
user forge;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 10000;

events {
  worker_connections 10000;
  multi_accept on;
}
```

上記の設定により、プロセスごとに最大10,000のNginxワーカーを生成できます。さらに、この設定によりNginxのオープンファイルの制限が10,000に設定されます。

<a name="ports"></a>
### ポート

Unixベースのオペレーティングシステムは通常、サーバー上で開くことができるポートの数を制限しています。以下のコマンドで現在許可されている範囲を確認できます：

 ```sh
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

上記の出力は、サーバーが最大28,231（60,999 - 32,768）の接続を処理できることを示しています。各接続には空きポートが必要です。許可される接続数を増やすために[水平スケーリング](#scaling)を推奨しますが、サーバーの`/etc/sysctl.conf`設定ファイルで許可されるポート範囲を更新することで、利用可能なオープンポートの数を増やすことができます。

<a name="process-management"></a>
### プロセス管理

ほとんどの場合、Reverbサーバーが常に実行されていることを確認するために、Supervisorなどのプロセスマネージャを使用する必要があります。Supervisorを使用してReverbを実行している場合、サーバーの`supervisor.conf`ファイルの`minfds`設定を更新して、SupervisorがReverbサーバーへの接続を処理するために必要なファイルを開くことができるようにする必要があります：

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### スケーリング

単一のサーバーで許可される接続数を超えて処理する必要がある場合、Reverbサーバーを水平方向にスケーリングすることができます。Redisのパブリッシュ/サブスクライブ機能を利用することで、Reverbは複数のサーバー間で接続を管理できます。アプリケーションのReverbサーバーの1つがメッセージを受信すると、そのサーバーはRedisを使用して受信メッセージを他のすべてのサーバーにパブリッシュします。

水平スケーリングを有効にするには、アプリケーションの`.env`設定ファイルで`REVERB_SCALING_ENABLED`環境変数を`true`に設定する必要があります：

```env
REVERB_SCALING_ENABLED=true
```

次に、すべてのReverbサーバーが通信する専用の中央Redisサーバーを用意する必要があります。Reverbは、アプリケーション用に設定された[デフォルトのRedis接続](redis.md#configuration)を使用して、すべてのReverbサーバーにメッセージをパブリッシュします。

Reverbのスケーリングオプションを有効にし、Redisサーバーを設定したら、Redisサーバーと通信できる複数のサーバーで`reverb:start`コマンドを実行するだけです。これらのReverbサーバーは、受信リクエストをサーバー間で均等に分散するロードバランサーの背後に配置する必要があります。

