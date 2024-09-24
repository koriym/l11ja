# Laravel Homestead

- [はじめに](#introduction)
- [インストールとセットアップ](#installation-and-setup)
    - [最初のステップ](#first-steps)
    - [Homesteadの設定](#configuring-homestead)
    - [Nginxサイトの設定](#configuring-nginx-sites)
    - [サービスの設定](#configuring-services)
    - [Vagrant Boxの起動](#launching-the-vagrant-box)
    - [プロジェクトごとのインストール](#per-project-installation)
    - [オプション機能のインストール](#installing-optional-features)
    - [エイリアス](#aliases)
- [Homesteadの更新](#updating-homestead)
- [日常的な使用](#daily-usage)
    - [SSH経由での接続](#connecting-via-ssh)
    - [追加サイトの追加](#adding-additional-sites)
    - [環境変数](#environment-variables)
    - [ポート](#ports)
    - [PHPバージョン](#php-versions)
    - [データベースへの接続](#connecting-to-databases)
    - [データベースのバックアップ](#database-backups)
    - [Cronスケジュールの設定](#configuring-cron-schedules)
    - [Mailpitの設定](#configuring-mailpit)
    - [Minioの設定](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [環境の共有](#sharing-your-environment)
- [デバッグとプロファイリング](#debugging-and-profiling)
    - [Xdebugを使用したWebリクエストのデバッグ](#debugging-web-requests)
    - [CLIアプリケーションのデバッグ](#debugging-cli-applications)
    - [Blackfireを使用したアプリケーションのプロファイリング](#profiling-applications-with-blackfire)
- [ネットワークインターフェース](#network-interfaces)
- [Homesteadの拡張](#extending-homestead)
- [プロバイダ固有の設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## はじめに

Laravelは、ローカル開発環境を含む、PHP開発全体の経験を楽しくすることを目指しています。[Laravel Homestead](https://github.com/laravel/homestead)は、公式の事前パッケージ化されたVagrant Boxで、PHPやWebサーバー、その他のサーバーソフトウェアをローカルマシンにインストールすることなく、素晴らしい開発環境を提供します。

[Vagrant](https://www.vagrantup.com)は、仮想マシンの管理とプロビジョニングを簡単かつエレガントに行う方法を提供します。Vagrant Boxは完全に使い捨て可能です。何か問題が発生した場合、数分でBoxを破棄して再作成できます。

Homesteadは、Windows、macOS、Linuxのいずれのシステムでも動作し、Nginx、PHP、MySQL、PostgreSQL、Redis、Memcached、Nodeなど、素晴らしいLaravelアプリケーションを開発するために必要なすべてのソフトウェアが含まれています。

> WARNING:  
> Windowsを使用している場合、ハードウェア仮想化（VT-x）を有効にする必要があるかもしれません。通常、BIOSから有効にできます。UEFIシステムでHyper-Vを使用している場合、VT-xにアクセスするためにHyper-Vを無効にする必要があるかもしれません。

<a name="included-software"></a>
### 含まれるソフトウェア

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 22.04
- Git
- PHP 8.3
- PHP 8.2
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 15
- Composer
- Docker
- Node（Yarn、Bower、Grunt、Gulpを含む）
- Redis
- Memcached
- Beanstalkd
- Mailpit
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

</div>

<a name="optional-software"></a>
### オプションのソフトウェア

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Elasticsearch
- EventStoreDB
- Flyway
- Gearman
- Go
- Grafana
- InfluxDB
- Logstash
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- Rust
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader（PHP拡張）
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## インストールとセットアップ

<a name="first-steps"></a>
### 最初のステップ

Homestead環境を起動する前に、[Vagrant](https://developer.hashicorp.com/vagrant/downloads)と、以下のサポートされているプロバイダのいずれかをインストールする必要があります：

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- [Parallels](https://www.parallels.com/products/desktop/)

これらのソフトウェアパッケージは、すべての主要なオペレーティングシステムに対応した簡単なインストーラを提供します。

Parallelsプロバイダを使用するには、[Parallels Vagrantプラグイン](https://github.com/Parallels/vagrant-parallels)をインストールする必要があります。これは無料です。

<a name="installing-homestead"></a>
#### Homesteadのインストール

Homesteadをインストールするには、Homesteadリポジトリをホストマシンにクローンします。リポジトリを「ホーム」ディレクトリ内の`Homestead`フォルダにクローンすることを検討してください。Homestead仮想マシンは、すべてのLaravelアプリケーションのホストとして機能します。このドキュメント全体で、このディレクトリを「Homesteadディレクトリ」と呼びます：

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

Laravel Homesteadリポジトリをクローンした後、`release`ブランチをチェックアウトする必要があります。このブランチには、常にHomesteadの最新の安定リリースが含まれています：

```shell
cd ~/Homestead

git checkout release
```

次に、Homesteadディレクトリから`bash init.sh`コマンドを実行して、`Homestead.yaml`設定ファイルを作成します。`Homestead.yaml`ファイルは、Homesteadインストールのすべての設定を行う場所です。このファイルはHomesteadディレクトリに配置されます：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### Homesteadの設定

<a name="setting-your-provider"></a>
#### プロバイダの設定

`Homestead.yaml`ファイルの`provider`キーは、使用するVagrantプロバイダを示します：`virtualbox`または`parallels`：

    provider: virtualbox

> WARNING:  
> Apple Siliconを使用している場合、Parallelsプロバイダが必要です。

<a name="configuring-shared-folders"></a>
#### 共有フォルダの設定

`Homestead.yaml`ファイルの`folders`プロパティは、Homestead環境と共有したいすべてのフォルダをリストアップします。これらのフォルダ内のファイルが変更されると、ローカルマシンとHomestead仮想環境の間で同期されます。必要に応じて、任意の数の共有フォルダを設定できます：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> WARNING:  
> Windowsユーザーは`~/`パス構文を使用せず、代わりに`C:\Users\user\Code\project1`のようなプロジェクトへの完全なパスを使用するべきです。

個々のアプリケーションをそれぞれのフォルダマッピングにマップすることを常に検討してください。フォルダをマップすると、仮想マシンはフォルダ内のすべてのファイルのディスクIOを追跡する必要があります。フォルダ内に多数のファイルがある場合、パフォーマンスが低下する可能性があります：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> WARNING:  
> Homesteadを使用する際には、`.`（現在のディレクトリ）をマウントしないでください。これにより、Vagrantが現在のフォルダを`/vagrant`にマップできなくなり、オプション機能が壊れたり、プロビジョニング中に予期しない結果が発生する可能性があります。

[NFS](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs)を有効にするには、フォルダマッピングに`type`オプションを追加できます：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> WARNING:  
> WindowsでNFSを使用する場合、[vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd)プラグインのインストールを検討してください。このプラグインは、Homestead仮想マシン内のファイルとディレクトリの正しいユーザー/グループの権限を維持します。

Vagrantの[同期フォルダ](https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage)でサポートされている任意のオプションを、`options`キーの下にリストアップして渡すことができます：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

<a name="configuring-nginx-sites"></a>
### Nginxサイトの設定

Nginxに慣れていないですか？問題ありません。`Homestead.yaml`ファイルの`sites`プロパティを使用すると、「ドメイン」をHomestead環境内のフォルダに簡単にマップできます。`Homestead.yaml`ファイルには、サンプルのサイト設定が含まれています。繰り返しますが、Homestead環境に必要な数のサイトを追加できます。Homesteadは、作業中のすべてのLaravelアプリケーションに対して便利な仮想化環境として機能できます：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

Homestead仮想マシンのプロビジョニング後に`sites`プロパティを変更した場合、ターミナルで`vagrant reload --provision`コマンドを実行して、仮想マシン上のNginx設定を更新する必要があります。

> WARNING:  
> Homesteadスクリプトは、可能な限り冪等性を持つように構築されています。しかし、プロビジョニング中に問題が発生した場合、`vagrant destroy && vagrant up`コマンドを実行してマシンを破棄して再構築する必要があります。

<a name="hostname-resolution"></a>
#### ホスト名の解決

Homestead は、ホスト名を `mDNS` を使用して自動的にホスト解決用に公開します。`Homestead.yaml` ファイルで `hostname: homestead` を設定した場合、ホストは `homestead.local` で利用可能になります。macOS、iOS、および Linux のデスクトップディストリビューションには、デフォルトで `mDNS` がサポートされています。Windows を使用している場合は、[Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US) をインストールする必要があります。

自動ホスト名は、Homestead の[プロジェクトごとのインストール](#per-project-installation)に最適です。単一の Homestead インスタンスで複数のサイトをホストしている場合は、Web サイトの "ドメイン" をマシンの `hosts` ファイルに追加することができます。`hosts` ファイルは、Homestead サイトへのリクエストを Homestead 仮想マシンにリダイレクトします。macOS と Linux では、このファイルは `/etc/hosts` にあります。Windows では、`C:\Windows\System32\drivers\etc\hosts` にあります。このファイルに追加する行は次のようになります：

    192.168.56.56  homestead.test

リストされている IP アドレスが `Homestead.yaml` ファイルで設定されているものであることを確認してください。ドメインを `hosts` ファイルに追加し、Vagrant ボックスを起動すると、Web ブラウザからサイトにアクセスできるようになります：

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### サービスの設定

Homestead はデフォルトでいくつかのサービスを起動しますが、プロビジョニング中に有効または無効にするサービスをカスタマイズすることができます。たとえば、PostgreSQL を有効にし、MySQL を無効にするには、`Homestead.yaml` ファイル内の `services` オプションを変更します：

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

指定されたサービスは、`enabled` および `disabled` ディレクティブの順序に基づいて起動または停止されます。

<a name="launching-the-vagrant-box"></a>
### Vagrant Box の起動

`Homestead.yaml` を編集したら、Homestead ディレクトリから `vagrant up` コマンドを実行します。Vagrant は仮想マシンを起動し、共有フォルダと Nginx サイトを自動的に設定します。

マシンを破棄するには、`vagrant destroy` コマンドを使用できます。

<a name="per-project-installation"></a>
### プロジェクトごとのインストール

Homestead をグローバルにインストールし、すべてのプロジェクトで同じ Homestead 仮想マシンを共有する代わりに、管理する各プロジェクトに対して Homestead インスタンスを設定することができます。プロジェクトに `Vagrantfile` を同梱し、プロジェクトのリポジトリをクローンした後に他の人がすぐに `vagrant up` できるようにする場合、プロジェクトごとに Homestead をインストールすることが有益です。

Composer パッケージマネージャを使用して Homestead をプロジェクトにインストールできます：

```shell
composer require laravel/homestead --dev
```

Homestead がインストールされたら、Homestead の `make` コマンドを呼び出してプロジェクトの `Vagrantfile` と `Homestead.yaml` ファイルを生成します。これらのファイルはプロジェクトのルートに配置されます。`make` コマンドは、`Homestead.yaml` ファイル内の `sites` および `folders` ディレクティブを自動的に設定します：

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

次に、ターミナルで `vagrant up` コマンドを実行し、ブラウザで `http://homestead.test` にアクセスしてプロジェクトにアクセスします。自動[ホスト名解決](#hostname-resolution)を使用していない場合は、`/etc/hosts` ファイルに `homestead.test` または選択したドメインのエントリを追加する必要があることを忘れないでください。

<a name="installing-optional-features"></a>
### オプション機能のインストール

オプションのソフトウェアは、`Homestead.yaml` ファイル内の `features` オプションを使用してインストールされます。ほとんどの機能はブール値で有効または無効にできますが、一部の機能は複数の設定オプションを許可します：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - dragonflydb: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - flyway: true
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - logstash: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rustc: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

<a name="elasticsearch"></a>
#### Elasticsearch

サポートされているバージョンの Elasticsearch を指定できます。これは、正確なバージョン番号（major.minor.patch）である必要があります。デフォルトのインストールでは、'homestead' という名前のクラスタが作成されます。Elasticsearch には、オペレーティングシステムのメモリの半分以上を割り当てないでください。したがって、Homestead 仮想マシンには、Elasticsearch の割り当て量の少なくとも2倍のメモリが必要です。

> NOTE:  
> [Elasticsearch ドキュメント](https://www.elastic.co/guide/en/elasticsearch/reference/current)を参照して、設定をカスタマイズする方法を学んでください。

<a name="mariadb"></a>
#### MariaDB

MariaDB を有効にすると、MySQL が削除され、MariaDB がインストールされます。MariaDB は通常、MySQL のドロップイン代替品として機能するため、アプリケーションのデータベース設定で `mysql` データベースドライバを引き続き使用する必要があります。

<a name="mongodb"></a>
#### MongoDB

デフォルトの MongoDB インストールでは、データベースのユーザー名が `homestead` に設定され、対応するパスワードが `secret` に設定されます。

<a name="neo4j"></a>
#### Neo4j

デフォルトの Neo4j インストールでは、データベースのユーザー名が `homestead` に設定され、対応するパスワードが `secret` に設定されます。Neo4j ブラウザにアクセスするには、Web ブラウザで `http://homestead.test:7474` にアクセスします。ポート `7687`（Bolt）、`7474`（HTTP）、および `7473`（HTTPS）は、Neo4j クライアントからのリクエストを処理する準備ができています。

<a name="aliases"></a>
### エイリアス

Bash エイリアスを Homestead 仮想マシンに追加するには、Homestead ディレクトリ内の `aliases` ファイルを変更します：

```shell
alias c='clear'
alias ..='cd ..'
```

`aliases` ファイルを更新した後、Homestead 仮想マシンを `vagrant reload --provision` コマンドを使用して再プロビジョニングする必要があります。これにより、新しいエイリアスがマシンで利用可能になります。

<a name="updating-homestead"></a>
## Homestead の更新

Homestead の更新を開始する前に、Homestead ディレクトリで次のコマンドを実行して、現在の仮想マシンを削除する必要があります：

```shell
vagrant destroy
```

次に、Homestead のソースコードを更新する必要があります。リポジトリをクローンした場合、最初にリポジトリをクローンした場所で次のコマンドを実行できます：

```shell
git fetch

git pull origin release
```

これらのコマンドは、GitHub リポジトリから最新の Homestead コードをプルし、最新のタグを取得し、最新のタグ付きリリースをチェックアウトします。Homestead の最新の安定リリースバージョンは、Homestead の [GitHub リリースページ](https://github.com/laravel/homestead/releases)で見つけることができます。

プロジェクトの `composer.json` ファイルを介して Homestead をインストールした場合、`composer.json` ファイルに `"laravel/homestead": "^12"` が含まれていることを確認し、依存関係を更新する必要があります：

```shell
composer update
```

次に、`vagrant box update` コマンドを使用して Vagrant ボックスを更新する必要があります：

```shell
vagrant box update
```

Vagrant ボックスを更新した後、Homestead ディレクトリから `bash init.sh` コマンドを実行して、Homestead の追加設定ファイルを更新する必要があります。既存の `Homestead.yaml`、`after.sh`、および `aliases` ファイルを上書きするかどうかを尋ねられます：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

最後に、最新の Vagrant インストールを利用するために Homestead 仮想マシンを再生成する必要があります：

```shell
vagrant up
```

<a name="daily-usage"></a>
## 日常的な使用

<a name="connecting-via-ssh"></a>
### SSH 経由での接続

Homestead ディレクトリから `vagrant ssh` ターミナルコマンドを実行することで、仮想マシンに SSH 接続できます。

<a name="adding-additional-sites"></a>
### 追加サイトの追加

Homestead 環境がプロビジョニングされて実行されたら、他の Laravel プロジェクト用に追加の Nginx サイトを追加することができます。単一の Homestead 環境で必要な数の Laravel プロジェクトを実行できます。追加サイトを追加するには、サイトを `Homestead.yaml` ファイルに追加します。

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> WARNING:  
> サイトを追加する前に、プロジェクトのディレクトリに対して[フォルダマッピング](#configuring-shared-folders)を設定していることを確認してください。

Vagrant が "hosts" ファイルを自動的に管理していない場合は、新しいサイトをそのファイルにも追加する必要があります。macOS と Linux では、このファイルは `/etc/hosts` にあります。Windows では、`C:\Windows\System32\drivers\etc\hosts` にあります：

    192.168.56.56  homestead.test
    192.168.56.56  another.test

サイトを追加したら、Homestead ディレクトリから `vagrant reload --provision` ターミナルコマンドを実行します。

<a name="site-types"></a>
#### サイトタイプ

Homestead は、Laravel 以外のプロジェクトを簡単に実行できるように、いくつかの「タイプ」のサイトをサポートしています。たとえば、`statamic` サイトタイプを使用して、Homestead に Statamic アプリケーションを簡単に追加できます：

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

利用可能なサイトタイプは、`apache`、`apache-proxy`、`apigility`、`expressive`、`laravel`（デフォルト）、`proxy`（nginx 用）、`silverstripe`、`statamic`、`symfony2`、`symfony4`、および `zf` です。

<a name="site-parameters"></a>
#### サイトパラメータ

追加の Nginx `fastcgi_param` 値をサイトに追加するには、`params` サイトディレクティブを使用します：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

<a name="environment-variables"></a>
### 環境変数

`Homestead.yaml` ファイルに追加することで、グローバル環境変数を定義できます。

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

`Homestead.yaml` ファイルを更新した後、`vagrant reload --provision` コマンドを実行してマシンを再プロビジョニングすることを忘れないでください。これにより、インストールされているすべての PHP バージョンの PHP-FPM 設定が更新され、`vagrant` ユーザーの環境も更新されます。

<a name="ports"></a>
### ポート

デフォルトでは、以下のポートが Homestead 環境に転送されます。

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; 80 に転送
- **HTTPS:** 44300 &rarr; 443 に転送

</div>

<a name="forwarding-additional-ports"></a>
#### 追加のポートを転送する

必要に応じて、`Homestead.yaml` ファイル内に `ports` 設定エントリを定義することで、追加のポートを Vagrant ボックスに転送できます。`Homestead.yaml` ファイルを更新した後、`vagrant reload --provision` コマンドを実行してマシンを再プロビジョニングすることを忘れないでください。

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

以下は、ホストマシンから Vagrant ボックスにマッピングすることができる追加の Homestead サービスポートのリストです。

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; 22 に転送
- **ngrok UI:** 4040 &rarr; 4040 に転送
- **MySQL:** 33060 &rarr; 3306 に転送
- **PostgreSQL:** 54320 &rarr; 5432 に転送
- **MongoDB:** 27017 &rarr; 27017 に転送
- **Mailpit:** 8025 &rarr; 8025 に転送
- **Minio:** 9600 &rarr; 9600 に転送

</div>

<a name="php-versions"></a>
### PHP バージョン

Homestead は、同じ仮想マシン上で複数のバージョンの PHP を実行することをサポートしています。`Homestead.yaml` ファイル内で特定のサイトに使用する PHP バージョンを指定できます。利用可能な PHP バージョンは、"5.6"、"7.0"、"7.1"、"7.2"、"7.3"、"7.4"、"8.0"、"8.1"、"8.2"、および "8.3" です（デフォルト）。

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[Homestead 仮想マシン内で](#connecting-via-ssh)、CLI を介してサポートされている PHP バージョンのいずれかを使用できます。

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
php8.2 artisan list
php8.3 artisan list
```

CLI で使用される PHP のデフォルトバージョンを変更するには、Homestead 仮想マシン内で以下のコマンドを実行します。

```shell
php56
php70
php71
php72
php73
php74
php80
php81
php82
php83
```

<a name="connecting-to-databases"></a>
### データベースへの接続

MySQL と PostgreSQL の両方に対して `homestead` データベースがデフォルトで設定されています。ホストマシンのデータベースクライアントから MySQL または PostgreSQL データベースに接続するには、`127.0.0.1` のポート `33060`（MySQL）または `54320`（PostgreSQL）に接続します。両方のデータベースのユーザー名とパスワードは `homestead` / `secret` です。

> WARNING:  
> ホストマシンからデータベースに接続する場合にのみ、これらの非標準ポートを使用する必要があります。Laravel アプリケーションの `database` 設定ファイルでは、Laravel が仮想マシン内で実行されているため、デフォルトの 3306 および 5432 ポートを使用します。

<a name="database-backups"></a>
### データベースのバックアップ

Homestead は、Homestead 仮想マシンが破棄されたときにデータベースを自動的にバックアップできます。この機能を利用するには、Vagrant 2.1.0 以降を使用する必要があります。または、Vagrant の古いバージョンを使用している場合は、`vagrant-triggers` プラグインをインストールする必要があります。自動データベースバックアップを有効にするには、`Homestead.yaml` ファイルに以下の行を追加します。

    backup: true

設定が完了すると、Homestead は `vagrant destroy` コマンドが実行されたときにデータベースを `.backup/mysql_backup` および `.backup/postgres_backup` ディレクトリにエクスポートします。これらのディレクトリは、Homestead をインストールしたフォルダ内、または [プロジェクトごとのインストール](#per-project-installation) 方法を使用している場合はプロジェクトのルートにあります。

<a name="configuring-cron-schedules"></a>
### Cron スケジュールの設定

Laravel は、1 分ごとに実行される `schedule:run` Artisan コマンドをスケジュールすることで、[cron ジョブをスケジュールする](scheduling.md) 便利な方法を提供します。`schedule:run` コマンドは、`routes/console.php` ファイルで定義されたジョブスケジュールを調べて、どのスケジュールされたタスクを実行するかを決定します。

Homestead サイトの `schedule:run` コマンドを実行したい場合は、サイトを定義するときに `schedule` オプションを `true` に設定できます。

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

サイトの cron ジョブは、Homestead 仮想マシンの `/etc/cron.d` ディレクトリに定義されます。

<a name="configuring-mailpit"></a>
### Mailpit の設定

[Mailpit](https://github.com/axllent/mailpit) を使用すると、送信メールを傍受して調査し、実際にメールを受信者に送信することなく確認できます。開始するには、アプリケーションの `.env` ファイルを以下のメール設定を使用するように更新します。

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Mailpit が設定されたら、`http://localhost:8025` で Mailpit ダッシュボードにアクセスできます。

<a name="configuring-minio"></a>
### Minio の設定

[Minio](https://github.com/minio/minio) は、Amazon S3 互換の API を持つオープンソースのオブジェクトストレージサーバーです。Minio をインストールするには、`Homestead.yaml` ファイルを以下の設定オプションで更新します。

    minio: true

デフォルトでは、Minio はポート 9600 で利用可能です。Minio コントロールパネルには、`http://localhost:9600` にアクセスしてアクセスできます。デフォルトのアクセスキーは `homestead` で、デフォルトのシークレットキーは `secretkey` です。Minio にアクセスする場合は、常にリージョン `us-east-1` を使用する必要があります。

Minio を使用するには、`.env` ファイルに以下のオプションがあることを確認してください。

```ini
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=http://localhost:9600
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
```

Minio を使用した "S3" バケットをプロビジョニングするには、`Homestead.yaml` ファイルに `buckets` ディレクティブを追加します。バケットを定義した後、ターミナルで `vagrant reload --provision` コマンドを実行する必要があります。

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

サポートされている `policy` 値には、`none`、`download`、`upload`、および `public` が含まれます。

<a name="laravel-dusk"></a>
### Laravel Dusk

Homestead 内で [Laravel Dusk](dusk.md) テストを実行するには、Homestead 設定で [`webdriver` 機能](#installing-optional-features) を有効にする必要があります。

```yaml
features:
    - webdriver: true
```

`webdriver` 機能を有効にした後、ターミナルで `vagrant reload --provision` コマンドを実行する必要があります。

<a name="sharing-your-environment"></a>
### 環境の共有

現在作業中の内容を同僚やクライアントと共有したい場合があります。Vagrant には、`vagrant share` コマンドを介してこれをサポートする組み込み機能があります。ただし、`Homestead.yaml` ファイルに複数のサイトが設定されている場合、これは機能しません。

この問題を解決するために、Homestead には独自の `share` コマンドが含まれています。開始するには、`vagrant ssh` を介して [Homestead 仮想マシンに SSH 接続](#connecting-via-ssh) し、`share homestead.test` コマンドを実行します。これにより、`Homestead.yaml` 設定ファイルから `homestead.test` サイトが共有されます。`homestead.test` の代わりに他の設定済みサイトを指定できます。

```shell
share homestead.test
```

コマンドを実行すると、アクティビティログと共有サイトの公開 URL を含む Ngrok 画面が表示されます。カスタムリージョン、サブドメイン、またはその他の Ngrok ランタイムオプションを指定する場合は、`share` コマンドに追加できます。

```shell
share homestead.test -region=eu -subdomain=laravel
```

HTTPS ではなく HTTP でコンテンツを共有する必要がある場合は、`share` の代わりに `sshare` コマンドを使用します。

> WARNING:  
> 覚えておいてください。Vagrant は本質的に安全ではなく、`share` コマンドを実行すると仮想マシンがインターネットに公開されます。

<a name="debugging-and-profiling"></a>
## デバッグとプロファイリング

<a name="debugging-web-requests"></a>
### Xdebug を使用した Web リクエストのデバッグ

Homestead は、[Xdebug](https://xdebug.org) を使用したステップデバッグをサポートしています。例えば、ブラウザでページにアクセスし、PHP が IDE に接続して実行中のコードの検査と変更を可能にします。

デフォルトでは、Xdebug はすでに実行されており、接続を受け入れる準備ができています。CLI で Xdebug を有効にする必要がある場合は、Homestead 仮想マシン内で `sudo phpenmod xdebug` コマンドを実行します。次に、IDE の指示に従ってデバッグを有効にします。最後に、ブラウザ拡張機能または [ブックマークレット](https://www.jetbrains.com/phpstorm/marklets/) を使用して Xdebug をトリガーするようにブラウザを設定します。

> WARNING:  
> Xdebug は PHP の実行速度を大幅に低下させます。Xdebug を無効にするには、Homestead 仮想マシン内で `sudo phpdismod xdebug` を実行し、FPM サービスを再起動します。

<a name="autostarting-xdebug"></a>
#### Xdebug の自動起動

Web サーバーへのリクエストを行う機能テストをデバッグする場合、デバッグを自動的に開始する方が簡単です。テストにカスタムヘッダーやクッキーを渡してデバッグをトリガーするよりも簡単です。Xdebug を強制的に自動的に開始するには、Homestead 仮想マシン内の `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` ファイルを変更し、以下の設定を追加します。

```ini
; Homestead.yaml に IP アドレスのための異なるサブネットが含まれている場合、このアドレスは異なる可能性があります...
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

<a name="debugging-cli-applications"></a>
### CLI アプリケーションのデバッグ

PHP CLI アプリケーションをデバッグするには、Homestead 仮想マシン内で `xphp` シェルエイリアスを使用します:

    xphp /path/to/script

<a name="profiling-applications-with-blackfire"></a>
### Blackfire によるアプリケーションのプロファイリング

[Blackfire](https://blackfire.io/docs/introduction) は、Web リクエストと CLI アプリケーションをプロファイリングするためのサービスです。プロファイルデータをコールグラフとタイムラインで表示するインタラクティブなユーザーインターフェースを提供します。開発、ステージング、本番環境で使用でき、エンドユーザーにオーバーヘッドをかけることなく動作します。さらに、Blackfire はコードと `php.ini` 設定のパフォーマンス、品質、セキュリティチェックを提供します。

[Blackfire Player](https://blackfire.io/docs/player/index) は、Web クローリング、Web テスト、Web スクレイピングを行うオープンソースのアプリケーションで、Blackfire と連携してプロファイリングシナリオをスクリプト化することができます。

Blackfire を有効にするには、Homestead 設定ファイルで "features" 設定を使用します:

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire のサーバー資格情報とクライアント資格情報は [Blackfire アカウント](https://blackfire.io/signup) が必要です。Blackfire は、CLI ツールやブラウザ拡張機能を含むアプリケーションをプロファイリングするためのさまざまなオプションを提供します。詳細については [Blackfire のドキュメントを確認してください](https://blackfire.io/docs/php/integrations/laravel/index)。

<a name="network-interfaces"></a>
## ネットワークインターフェース

`Homestead.yaml` ファイルの `networks` プロパティは、Homestead 仮想マシンのネットワークインターフェースを設定します。必要に応じて、インターフェースをいくつでも設定できます:

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

[ブリッジ](https://developer.hashicorp.com/vagrant/docs/networking/public_network) インターフェースを有効にするには、ネットワークの `bridge` 設定を構成し、ネットワークタイプを `public_network` に変更します:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

[DHCP](https://developer.hashicorp.com/vagrant/docs/networking/public_network#dhcp) を有効にするには、設定から `ip` オプションを削除します:

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

ネットワークが使用するデバイスを更新するには、ネットワークの設定に `dev` オプションを追加できます。デフォルトの `dev` 値は `eth0` です:

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
      dev: "enp2s0"
```

<a name="extending-homestead"></a>
## Homestead の拡張

Homestead ディレクトリのルートにある `after.sh` スクリプトを使用して、Homestead を拡張できます。このファイル内に、仮想マシンを適切に設定およびカスタマイズするために必要なすべてのシェルコマンドを追加できます。

Homestead をカスタマイズする際、Ubuntu はパッケージの元の設定を保持するか、新しい設定ファイルで上書きするかを尋ねる場合があります。これを避けるために、Homestead によって以前に書き込まれた設定を上書きしないように、パッケージをインストールする際に次のコマンドを使用する必要があります:

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### ユーザーカスタマイズ

チームで Homestead を使用する場合、個人的な開発スタイルに合わせて Homestead を調整したい場合があります。これを実現するには、Homestead ディレクトリのルートに `user-customizations.sh` ファイルを作成します（`Homestead.yaml` ファイルを含む同じディレクトリ）。このファイル内で、好きなカスタマイズを行うことができますが、`user-customizations.sh` はバージョン管理されるべきではありません。

<a name="provider-specific-settings"></a>
## プロバイダ固有の設定

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

デフォルトでは、Homestead は `natdnshostresolver` 設定を `on` にします。これにより、Homestead はホストオペレーティングシステムの DNS 設定を使用できます。この動作をオーバーライドしたい場合は、`Homestead.yaml` ファイルに次の設定オプションを追加します:

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```
