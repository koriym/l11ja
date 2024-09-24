# Laravel Sail

- [はじめに](#introduction)
- [インストールとセットアップ](#installation)
    - [既存のアプリケーションにSailをインストールする](#installing-sail-into-existing-applications)
    - [Sailイメージの再構築](#rebuilding-sail-images)
    - [シェルエイリアスの設定](#configuring-a-shell-alias)
- [Sailの起動と停止](#starting-and-stopping-sail)
- [コマンドの実行](#executing-sail-commands)
    - [PHPコマンドの実行](#executing-php-commands)
    - [Composerコマンドの実行](#executing-composer-commands)
    - [Artisanコマンドの実行](#executing-artisan-commands)
    - [Node / NPMコマンドの実行](#executing-node-npm-commands)
- [データベースとの連携](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [Redis](#redis)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [ファイルストレージ](#file-storage)
- [テストの実行](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [メールのプレビュー](#previewing-emails)
- [コンテナCLI](#sail-container-cli)
- [PHPバージョン](#sail-php-versions)
- [Nodeバージョン](#sail-node-versions)
- [サイトの共有](#sharing-your-site)
- [Xdebugによるデバッグ](#debugging-with-xdebug)
  - [Xdebug CLIの使用](#xdebug-cli-usage)
  - [Xdebugブラウザの使用](#xdebug-browser-usage)
- [カスタマイズ](#sail-customization)

<a name="introduction"></a>
## はじめに

[Laravel Sail](https://github.com/laravel/sail)は、LaravelのデフォルトのDocker開発環境と対話するための軽量のコマンドラインインターフェースです。Sailは、Dockerの経験がなくてもPHP、MySQL、Redisを使用してLaravelアプリケーションを構築するための優れた出発点を提供します。

Sailの中核は、プロジェクトのルートに保存されている`docker-compose.yml`ファイルと`sail`スクリプトです。`sail`スクリプトは、`docker-compose.yml`ファイルで定義されたDockerコンテナと対話するための便利なメソッドを持つCLIを提供します。

Laravel SailはmacOS、Linux、および[WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)経由のWindowsでサポートされています。

<a name="installation"></a>
## インストールとセットアップ

Laravel Sailはすべての新しいLaravelアプリケーションに自動的にインストールされるため、すぐに使用を開始できます。新しいLaravelアプリケーションを作成する方法については、お使いのオペレーティングシステムのLaravelの[インストールドキュメント](installation.md#docker-installation-using-sail)を参照してください。インストール中に、アプリケーションが対話するSailがサポートするサービスを選択するよう求められます。

<a name="installing-sail-into-existing-applications"></a>
### 既存のアプリケーションにSailをインストールする

既存のLaravelアプリケーションでSailを使用したい場合は、Composerパッケージマネージャを使用してSailをインストールできます。もちろん、これらの手順は、既存のローカル開発環境でComposerの依存関係をインストールできることを前提としています。

```shell
composer require laravel/sail --dev
```

Sailがインストールされたら、`sail:install` Artisanコマンドを実行できます。このコマンドは、Sailの`docker-compose.yml`ファイルをアプリケーションのルートに公開し、Dockerサービスに接続するために必要な環境変数で`.env`ファイルを変更します。

```shell
php artisan sail:install
```

最後に、Sailを起動できます。Sailの使用方法を続けて学ぶには、このドキュメントの残りの部分を読み進めてください。

```shell
./vendor/bin/sail up
```

> WARNING:  
> Docker Desktop for Linuxを使用している場合は、次のコマンドを実行して`default` Dockerコンテキストを使用する必要があります: `docker context use default`。

<a name="adding-additional-services"></a>
#### 追加のサービスを追加する

既存のSailインストールに追加のサービスを追加したい場合は、`sail:add` Artisanコマンドを実行できます。

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### Devcontainersを使用する

[Devcontainer](https://code.visualstudio.com/docs/remote/containers)内で開発したい場合は、`sail:install`コマンドに`--devcontainer`オプションを指定できます。`--devcontainer`オプションは、`sail:install`コマンドにアプリケーションのルートにデフォルトの`.devcontainer/devcontainer.json`ファイルを公開するよう指示します。

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### Sailイメージの再構築

場合によっては、イメージのすべてのパッケージとソフトウェアが最新であることを確認するために、Sailイメージを完全に再構築したいことがあります。これは`build`コマンドを使用して行うことができます。

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### シェルエイリアスの設定

デフォルトでは、Sailコマンドはすべての新しいLaravelアプリケーションに含まれる`vendor/bin/sail`スクリプトを使用して実行されます。

```shell
./vendor/bin/sail up
```

ただし、`vendor/bin/sail`を繰り返し入力する代わりに、Sailのコマンドをより簡単に実行できるようにシェルエイリアスを設定したい場合があります。

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

これが常に利用可能になるように、これをホームディレクトリのシェル設定ファイル（例: `~/.zshrc` または `~/.bashrc`）に追加し、シェルを再起動してください。

シェルエイリアスが設定されたら、単に`sail`と入力するだけでSailコマンドを実行できます。このドキュメントの残りの部分の例では、このエイリアスが設定されていることを前提としています。

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## Sailの起動と停止

Laravel Sailの`docker-compose.yml`ファイルは、Laravelアプリケーションの構築に役立つさまざまなDockerコンテナを定義しています。これらのコンテナは、`docker-compose.yml`ファイルの`services`設定内のエントリです。`laravel.test`コンテナは、アプリケーションを提供するプライマリアプリケーションコンテナです。

Sailを起動する前に、ローカルコンピュータで他のWebサーバーやデータベースが実行されていないことを確認してください。アプリケーションの`docker-compose.yml`ファイルで定義されたすべてのDockerコンテナを起動するには、`up`コマンドを実行します。

```shell
sail up
```

すべてのDockerコンテナをバックグラウンドで起動するには、Sailを「デタッチ」モードで起動できます。

```shell
sail up -d
```

アプリケーションのコンテナが起動したら、Webブラウザでプロジェクトにアクセスできます: http://localhost。

すべてのコンテナを停止するには、単にControl + Cを押してコンテナの実行を停止するか、コンテナがバックグラウンドで実行されている場合は`stop`コマンドを使用できます。

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## コマンドの実行

Laravel Sailを使用する場合、アプリケーションはDockerコンテナ内で実行され、ローカルコンピュータから分離されます。ただし、Sailは、任意のPHPコマンド、Artisanコマンド、Composerコマンド、およびNode / NPMコマンドなど、アプリケーションに対してさまざまなコマンドを実行する便利な方法を提供します。

**Laravelのドキュメントを読むとき、Composer、Artisan、およびNode / NPMコマンドへの参照がSailを参照しないことがよくあります。** これらの例は、これらのツールがローカルコンピュータにインストールされていることを前提としています。ローカルのLaravel開発環境にSailを使用している場合は、これらのコマンドをSailを使用して実行する必要があります。

```shell
# ローカルでArtisanコマンドを実行する...
php artisan queue:work

# Laravel Sail内でArtisanコマンドを実行する...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### PHPコマンドの実行

PHPコマンドは`php`コマンドを使用して実行できます。もちろん、これらのコマンドはアプリケーション用に設定されたPHPバージョンを使用して実行されます。Laravel Sailで利用可能なPHPバージョンの詳細については、[PHPバージョンドキュメント](#sail-php-versions)を参照してください。

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### Composerコマンドの実行

Composerコマンドは`composer`コマンドを使用して実行できます。Laravel SailのアプリケーションコンテナにはComposerがインストールされています。

```nothing
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### 既存のアプリケーションのComposer依存関係をインストールする

チームでアプリケーションを開発している場合、あなたが最初にLaravelアプリケーションを作成したわけではないかもしれません。したがって、アプリケーションのリポジトリをローカルコンピュータにクローンした後、アプリケーションのComposer依存関係（Sailを含む）はインストールされません。

アプリケーションのディレクトリに移動し、次のコマンドを実行してアプリケーションの依存関係をインストールできます。このコマンドは、PHPとComposerを含む小さなDockerコンテナを使用してアプリケーションの依存関係をインストールします。

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php83-composer:latest \
    composer install --ignore-platform-reqs
```

`laravelsail/phpXX-composer`イメージを使用する場合、アプリケーションで使用する予定のPHPバージョン（`80`、`81`、`82`、または`83`）を使用する必要があります。

<a name="executing-artisan-commands"></a>
### Artisanコマンドの実行

Laravel Artisanコマンドは`artisan`コマンドを使用して実行できます。

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### Node / NPMコマンドの実行

Nodeコマンドは`node`コマンドを使用して実行でき、NPMコマンドは`npm`コマンドを使用して実行できます。

```shell
sail node --version

sail npm run dev
```

必要に応じて、NPMの代わりにYarnを使用できます。

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## データベースとの連携

<a name="mysql"></a>
### MySQL

ご存知かもしれませんが、アプリケーションの `docker-compose.yml` ファイルには MySQL コンテナのエントリが含まれています。このコンテナは [Docker ボリューム](https://docs.docker.com/storage/volumes/) を使用しているため、コンテナを停止して再起動しても、データベースに保存されたデータは永続化されます。

さらに、MySQL コンテナが初めて起動するときに、2 つのデータベースが作成されます。最初のデータベースは `DB_DATABASE` 環境変数の値を使用して名前が付けられ、ローカル開発用です。2 つ目は専用のテストデータベースで、名前は `testing` で、テストが開発データに干渉しないようにします。

コンテナを起動したら、アプリケーションの `.env` ファイル内で `DB_HOST` 環境変数を `mysql` に設定することで、アプリケーション内の MySQL インスタンスに接続できます。

ローカルマシンからアプリケーションの MySQL データベースに接続するには、[TablePlus](https://tableplus.com) などのグラフィカルなデータベース管理アプリケーションを使用できます。デフォルトでは、MySQL データベースは `localhost` のポート 3306 でアクセス可能で、アクセス資格情報は `DB_USERNAME` および `DB_PASSWORD` 環境変数の値に対応しています。または、`root` ユーザーとして接続することもできます。この場合も、パスワードとして `DB_PASSWORD` 環境変数の値が使用されます。

<a name="redis"></a>
### Redis

アプリケーションの `docker-compose.yml` ファイルには、[Redis](https://redis.io) コンテナのエントリも含まれています。このコンテナは [Docker ボリューム](https://docs.docker.com/storage/volumes/) を使用しているため、コンテナを停止して再起動しても、Redis に保存されたデータは永続化されます。コンテナを起動したら、アプリケーションの `.env` ファイル内で `REDIS_HOST` 環境変数を `redis` に設定することで、アプリケーション内の Redis インスタンスに接続できます。

ローカルマシンからアプリケーションの Redis データベースに接続するには、[TablePlus](https://tableplus.com) などのグラフィカルなデータベース管理アプリケーションを使用できます。デフォルトでは、Redis データベースは `localhost` のポート 6379 でアクセス可能です。

<a name="meilisearch"></a>
### Meilisearch

Sail のインストール時に [Meilisearch](https://www.meilisearch.com) サービスをインストールすることを選択した場合、アプリケーションの `docker-compose.yml` ファイルには、[Laravel Scout](scout.md) と統合されたこの強力な検索エンジンのエントリが含まれます。コンテナを起動したら、`MEILISEARCH_HOST` 環境変数を `http://meilisearch:7700` に設定することで、アプリケーション内の Meilisearch インスタンスに接続できます。

ローカルマシンから、Web ベースの Meilisearch 管理パネルに `http://localhost:7700` でアクセスできます。

<a name="typesense"></a>
### Typesense

Sail のインストール時に [Typesense](https://typesense.org) サービスをインストールすることを選択した場合、アプリケーションの `docker-compose.yml` ファイルには、[Laravel Scout](scout.md#typesense) とネイティブに統合されたこの高速なオープンソース検索エンジンのエントリが含まれます。コンテナを起動したら、以下の環境変数を設定することで、アプリケーション内の Typesense インスタンスに接続できます：

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

ローカルマシンから、`http://localhost:8108` で Typesense の API にアクセスできます。

<a name="file-storage"></a>
## ファイルストレージ

アプリケーションを本番環境で実行する際に Amazon S3 を使用してファイルを保存する予定の場合、Sail のインストール時に [MinIO](https://min.io) サービスをインストールすることをお勧めします。MinIO は、本番環境の S3 環境に「テスト」ストレージバケットを作成することなく、Laravel の `s3` ファイルストレージドライバを使用してローカルで開発するために使用できる S3 互換 API を提供します。MinIO をインストールすることを選択した場合、MinIO の設定セクションがアプリケーションの `docker-compose.yml` ファイルに追加されます。

デフォルトでは、アプリケーションの `filesystems` 設定ファイルには、`s3` ディスクのディスク設定が既に含まれています。Amazon S3 とやり取りするためにこのディスクを使用するだけでなく、MinIO などの S3 互換ファイルストレージサービスとやり取りするためにも使用できます。そのためには、関連する環境変数を単純に変更します。例えば、MinIO を使用する場合、ファイルシステムの環境変数設定は次のように定義されます：

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

Laravel の Flysystem 統合が MinIO を使用する際に適切な URL を生成できるようにするには、`AWS_URL` 環境変数を定義して、アプリケーションのローカル URL と一致させ、URL パスにバケット名を含める必要があります：

```ini
AWS_URL=http://localhost:9000/local
```

MinIO コンソールを介してバケットを作成できます。MinIO コンソールは `http://localhost:8900` で利用できます。MinIO コンソールのデフォルトユーザー名は `sail` で、デフォルトパスワードは `password` です。

> WARNING:  
> `temporaryUrl` メソッドを介して一時的なストレージ URL を生成することは、MinIO を使用する場合はサポートされていません。

<a name="running-tests"></a>
## テストの実行

Laravel は、ボックスから出してすぐに素晴らしいテストサポートを提供します。Sail の `test` コマンドを使用して、アプリケーションの [機能テストと単体テスト](testing.md) を実行できます。Pest / PHPUnit が受け入れるすべての CLI オプションも `test` コマンドに渡すことができます：

```shell
sail test

sail test --group orders
```

Sail の `test` コマンドは、`test` Artisan コマンドを実行するのと同じです：

```shell
sail artisan test
```

デフォルトでは、Sail は専用の `testing` データベースを作成し、テストが現在のデータベースの状態に干渉しないようにします。デフォルトの Laravel インストールでは、Sail は `phpunit.xml` ファイルを設定して、テストの実行時にこのデータベースを使用するようにします：

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](dusk.md) は、表現力豊かで使いやすいブラウザ自動化およびテスト API を提供します。Sail のおかげで、ローカルコンピュータに Selenium やその他のツールをインストールすることなく、これらのテストを実行できます。始めるには、アプリケーションの `docker-compose.yml` ファイル内の Selenium サービスのコメントを外します：

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

次に、アプリケーションの `docker-compose.yml` ファイル内の `laravel.test` サービスに `depends_on` エントリを追加して、`selenium` に依存するようにします：

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後に、Sail を起動して `dusk` コマンドを実行することで、Dusk テストスイートを実行できます：

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Apple Silicon 上の Selenium

ローカルマシンに Apple Silicon チップが搭載されている場合、`selenium` サービスは `selenium/standalone-chromium` イメージを使用する必要があります：

```yaml
selenium:
    image: 'selenium/standalone-chromium'
    extra_hosts:
        - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## メールのプレビュー

Laravel Sail のデフォルトの `docker-compose.yml` ファイルには、[Mailpit](https://github.com/axllent/mailpit) のサービスエントリが含まれています。Mailpit は、ローカル開発中にアプリケーションから送信されたメールを傍受し、ブラウザでメールメッセージをプレビューできる便利な Web インターフェースを提供します。Sail を使用する場合、Mailpit のデフォルトホストは `mailpit` で、ポート 1025 で利用できます：

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

Sail が実行されている場合、Mailpit の Web インターフェースに `http://localhost:8025` でアクセスできます。

<a name="sail-container-cli"></a>
## コンテナ CLI

アプリケーションのコンテナ内で Bash セッションを開始したい場合があります。`shell` コマンドを使用してアプリケーションのコンテナに接続し、そのファイルやインストールされたサービスを検査したり、コンテナ内で任意のシェルコマンドを実行したりできます：

```shell
sail shell

sail root-shell
```

新しい [Laravel Tinker](https://github.com/laravel/tinker) セッションを開始するには、`tinker` コマンドを実行できます：

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP バージョン

Sail は現在、PHP 8.3、8.2、8.1、または PHP 8.0 を介してアプリケーションを提供することをサポートしています。Sail で使用されるデフォルトの PHP バージョンは現在 PHP 8.3 です。アプリケーションを提供するために使用される PHP バージョンを変更するには、アプリケーションの `docker-compose.yml` ファイル内の `laravel.test` コンテナの `build` 定義を更新する必要があります：

```yaml
# PHP 8.3
context: ./vendor/laravel/sail/runtimes/8.3

# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0
```

さらに、アプリケーションで使用される PHP バージョンを反映するように `image` 名を更新することもできます。このオプションもアプリケーションの `docker-compose.yml` ファイルで定義されています：

```yaml
image: sail-8.2/app
```

アプリケーションの `docker-compose.yml` ファイルを更新した後、コンテナイメージを再構築する必要があります：

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node バージョン

Sail はデフォルトで Node 20 をインストールします。イメージを構築する際にインストールされる Node バージョンを変更するには、アプリケーションの `docker-compose.yml` ファイル内の `laravel.test` サービスの `build.args` 定義を更新する必要があります：

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

アプリケーションの `docker-compose.yml` ファイルを更新した後、コンテナイメージを再構築する必要があります：

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## サイトの共有

時には、サイトを公開して、同僚にサイトをプレビューしてもらったり、アプリケーションとのWebhook統合をテストしたりする必要があるかもしれません。サイトを共有するには、`share`コマンドを使用できます。このコマンドを実行すると、アプリケーションにアクセスするために使用できるランダムな`laravel-sail.site` URLが発行されます。

```shell
sail share
```

`share`コマンドでサイトを共有する場合、アプリケーションの`bootstrap/app.php`ファイル内で`trustProxies`ミドルウェアメソッドを使用して、アプリケーションの信頼できるプロキシを設定する必要があります。そうしないと、`url`や`route`などのURL生成ヘルパーは、URL生成時に使用すべき正しいHTTPホストを決定できません。

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(at: '*');
})
```

共有サイトのサブドメインを選択したい場合は、`share`コマンドを実行する際に`subdomain`オプションを指定できます。

```shell
sail share --subdomain=my-sail-site
```

> NOTE:  
> `share`コマンドは、[BeyondCode](https://beyondco.de)によるオープンソースのトンネリングサービスである[Expose](https://github.com/beyondcode/expose)を利用しています。

<a name="debugging-with-xdebug"></a>
## Xdebugによるデバッグ

Laravel SailのDocker設定には、PHP用の人気で強力なデバッガーである[Xdebug](https://xdebug.org/)のサポートが含まれています。Xdebugを有効にするには、アプリケーションの`.env`ファイルにいくつかの変数を追加して、[Xdebugを設定](https://xdebug.org/docs/step_debug#mode)する必要があります。Xdebugを有効にするには、Sailを起動する前に適切なモードを設定する必要があります。

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

#### LinuxホストのIP設定

内部的には、`XDEBUG_CONFIG`環境変数は`client_host=host.docker.internal`と定義されており、XdebugはMacとWindows（WSL2）で正しく設定されます。ローカルマシンがLinuxで、Docker 20.10+を使用している場合、`host.docker.internal`は利用可能であり、手動設定は必要ありません。

Docker 20.10より古いバージョンでは、Linuxで`host.docker.internal`はサポートされておらず、ホストIPを手動で定義する必要があります。これを行うには、`docker-compose.yml`ファイルでカスタムネットワークを定義して、コンテナに静的IPを設定します。

```yaml
networks:
  custom_network:
    ipam:
      config:
        - subnet: 172.20.0.0/16

services:
  laravel.test:
    networks:
      custom_network:
        ipv4_address: 172.20.0.2
```

静的IPを設定したら、アプリケーションの`.env`ファイル内で`SAIL_XDEBUG_CONFIG`変数を定義します。

```ini
SAIL_XDEBUG_CONFIG="client_host=172.20.0.2"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLIの使用

`sail debug`コマンドを使用して、Artisanコマンドを実行する際にデバッグセッションを開始できます。

```shell
# XdebugなしでArtisanコマンドを実行する...
sail artisan migrate

# Xdebugを使用してArtisanコマンドを実行する...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebugブラウザの使用

Webブラウザを介してアプリケーションと対話しながらアプリケーションをデバッグするには、Xdebugが提供する[指示](https://xdebug.org/docs/step_debug#web-application)に従って、WebブラウザからXdebugセッションを開始します。

PhpStormを使用している場合は、JetBrainsのドキュメントを参照して、[ゼロコンフィグレーションデバッグ](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html)について確認してください。

> WARNING:  
> Laravel Sailはアプリケーションを提供するために`artisan serve`に依存しています。`artisan serve`コマンドは、Laravelバージョン8.53.0以降でのみ`XDEBUG_CONFIG`と`XDEBUG_MODE`変数を受け付けます。Laravelの古いバージョン（8.52.0以下）はこれらの変数をサポートせず、デバッグ接続を受け付けません。

<a name="sail-customization"></a>
## カスタマイズ

SailはDockerなので、ほぼすべての部分を自由にカスタマイズできます。Sailの独自のDockerfileを公開するには、`sail:publish`コマンドを実行できます。

```shell
sail artisan sail:publish
```

このコマンドを実行すると、Laravel Sailが使用するDockerfileやその他の設定ファイルが、アプリケーションのルートディレクトリ内の`docker`ディレクトリに配置されます。Sailのインストールをカスタマイズした後、アプリケーションの`docker-compose.yml`ファイルでアプリケーションコンテナのイメージ名を変更することをお勧めします。その後、`build`コマンドを使用してアプリケーションのコンテナを再構築します。アプリケーションイメージに一意の名前を付けることは、Sailを使用して1台のマシン上で複数のLaravelアプリケーションを開発する場合に特に重要です。

```shell
sail build --no-cache
```

