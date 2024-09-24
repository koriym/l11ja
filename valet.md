# Laravel Valet

- [はじめに](#introduction)
- [インストール](#installation)
    - [Valetのアップグレード](#upgrading-valet)
- [サイトの提供](#serving-sites)
    - [The "Park" コマンド](#the-park-command)
    - [The "Link" コマンド](#the-link-command)
    - [TLSでサイトを保護する](#securing-sites)
    - [デフォルトサイトの提供](#serving-a-default-site)
    - [サイトごとのPHPバージョン](#per-site-php-versions)
- [サイトの共有](#sharing-sites)
    - [ローカルネットワーク上でのサイト共有](#sharing-sites-on-your-local-network)
- [サイト固有の環境変数](#site-specific-environment-variables)
- [プロキシサービス](#proxying-services)
- [カスタムValetドライバー](#custom-valet-drivers)
    - [ローカルドライバー](#local-drivers)
- [その他のValetコマンド](#other-valet-commands)
- [Valetのディレクトリとファイル](#valet-directories-and-files)
    - [ディスクアクセス](#disk-access)

<a name="introduction"></a>
## はじめに

> NOTE:  
> macOSやWindowsでLaravelアプリケーションを開発するためのさらに簡単な方法を探していますか？ [Laravel Herd](https://herd.laravel.com)をチェックしてください。Herdには、Laravel開発を始めるために必要なすべてが含まれています。Valet、PHP、Composerなどが含まれています。

[Laravel Valet](https://github.com/laravel/valet)は、macOSミニマリストのための開発環境です。Laravel Valetは、マシンの起動時に常に[Nginx](https://www.nginx.com/)をバックグラウンドで実行するようにMacを設定します。そして、[DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)を使用して、`*.test`ドメインへのすべてのリクエストをローカルマシンにインストールされたサイトにプロキシします。

言い換えれば、Valetは約7MBのRAMを使用する、非常に高速なLaravel開発環境です。Valetは[Sail](sail.md)や[Homestead](homestead.md)の完全な代替品ではありませんが、柔軟な基本機能を好む場合、極端な速度を好む場合、またはRAMが限られたマシンで作業している場合には、優れた代替手段を提供します。

デフォルトで、Valetは以下を含むがこれに限定されないサポートを提供します：

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [ConcreteCMS](https://www.concretecms.com/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- 静的HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

ただし、独自の[カスタムドライバー](#custom-valet-drivers)を使用してValetを拡張することもできます。

<a name="installation"></a>
## インストール

> WARNING:  
> ValetはmacOSと[Homebrew](https://brew.sh/)を必要とします。インストール前に、ApacheやNginxなどの他のプログラムがローカルマシンのポート80をバインドしていないことを確認してください。

まず、Homebrewを`update`コマンドを使用して最新の状態にする必要があります：

```shell
brew update
```

次に、Homebrewを使用してPHPをインストールする必要があります：

```shell
brew install php
```

PHPをインストールした後、[Composerパッケージマネージャ](https://getcomposer.org)をインストールする準備が整います。さらに、`$HOME/.composer/vendor/bin`ディレクトリがシステムの"PATH"に含まれていることを確認する必要があります。Composerがインストールされたら、Laravel ValetをグローバルComposerパッケージとしてインストールできます：

```shell
composer global require laravel/valet
```

最後に、Valetの`install`コマンドを実行できます。これにより、ValetとDnsMasqが設定およびインストールされます。さらに、Valetが依存するデーモンがシステムの起動時に起動するように設定されます：

```shell
valet install
```

Valetがインストールされたら、ターミナルで`ping foobar.test`などのコマンドを使用して、任意の`*.test`ドメインにpingを打ってみてください。Valetが正しくインストールされていれば、このドメインは`127.0.0.1`で応答するはずです。

Valetは、マシンの起動時に必要なサービスを自動的に開始します。

<a name="php-versions"></a>
#### PHPバージョン

> NOTE:  
> グローバルなPHPバージョンを変更する代わりに、`isolate` [コマンド](#per-site-php-versions)を介してValetにサイトごとのPHPバージョンを使用するよう指示できます。

Valetでは、`valet use php@version`コマンドを使用してPHPバージョンを切り替えることができます。Valetは、まだインストールされていない場合、Homebrewを介して指定されたPHPバージョンをインストールします：

```shell
valet use php@8.2

valet use php
```

プロジェクトのルートに`.valetrc`ファイルを作成することもできます。`.valetrc`ファイルには、サイトが使用するPHPバージョンを含める必要があります：

```shell
php=php@8.2
```

このファイルが作成されたら、単に`valet use`コマンドを実行するだけで、コマンドはファイルを読み取ってサイトの優先PHPバージョンを決定します。

> WARNING:  
> Valetは一度に1つのPHPバージョンのみを提供します。複数のPHPバージョンがインストールされている場合でも同様です。

<a name="database"></a>
#### データベース

アプリケーションがデータベースを必要とする場合は、[DBngin](https://dbngin.com)をチェックしてください。DBnginは、MySQL、PostgreSQL、Redisを含む無料のオールインワンデータベース管理ツールを提供します。DBnginがインストールされたら、`127.0.0.1`でデータベースに接続できます。ユーザー名は`root`、パスワードは空文字列を使用します。

<a name="resetting-your-installation"></a>
#### インストールのリセット

Valetのインストールが正しく実行されない場合、`composer global require laravel/valet`コマンドを実行してから`valet install`を実行すると、インストールをリセットし、さまざまな問題を解決できます。まれに、`valet uninstall --force`を実行してから`valet install`を実行することで、Valetを「ハードリセット」する必要があるかもしれません。

<a name="upgrading-valet"></a>
### Valetのアップグレード

ターミナルで`composer global require laravel/valet`コマンドを実行することで、Valetのインストールをアップグレードできます。アップグレード後、`valet install`コマンドを実行して、必要に応じてValetが設定ファイルに追加のアップグレードを行うようにするのが良い習慣です。

<a name="upgrading-to-valet-4"></a>
#### Valet 4へのアップグレード

Valet 3からValet 4にアップグレードする場合は、以下の手順に従ってValetのインストールを適切にアップグレードしてください：

<div class="content-list" markdown="1">

- サイトのPHPバージョンをカスタマイズするために`.valetphprc`ファイルを追加した場合、各`.valetphprc`ファイルの名前を`.valetrc`に変更してください。そして、`.valetrc`ファイルの既存の内容の前に`php=`を付けてください。
- カスタムドライバーを新しいドライバーシステムの名前空間、拡張子、タイプヒント、および戻り値のタイプヒントに一致するように更新してください。Valetの[SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php)を例として参照できます。
- PHP 7.1 - 7.4を使用してサイトを提供している場合、Valetが一部のスクリプトを実行するために使用するバージョン8.0以上のPHPをHomebrewを使用してインストールしていることを確認してください。これは、プライマリリンクされたバージョンではない場合でも、Valetがこのバージョンを使用することを意味します。

</div>

<a name="serving-sites"></a>
## サイトの提供

Valetがインストールされたら、Laravelアプリケーションの提供を開始する準備が整いました。Valetは、アプリケーションを提供するのに役立つ2つのコマンドを提供します：`park`と`link`。

<a name="the-park-command"></a>
### The `park` コマンド

`park`コマンドは、マシン上のディレクトリを登録します。このディレクトリに含まれるすべてのディレクトリは、`http://<directory-name>.test`でWebブラウザからアクセスできるようになります：

```shell
cd ~/Sites

valet park
```

それでおしまいです。これで、「パークされた」ディレクトリ内に作成したすべてのアプリケーションは、`http://<directory-name>.test`の規則を使用して自動的に提供されます。したがって、パークされたディレクトリに「laravel」という名前のディレクトリが含まれている場合、そのディレクトリ内のアプリケーションは`http://laravel.test`でアクセスできます。さらに、Valetはワイルドカードサブドメイン（`http://foo.laravel.test`）を使用してサイトにアクセスできるように自動的に許可します。

<a name="the-link-command"></a>
### The `link` コマンド

`link`コマンドは、Laravelアプリケーションを提供するためにも使用できます。このコマンドは、ディレクトリ全体ではなく、単一のサイトを提供する場合に便利です：

```shell
cd ~/Sites/laravel

valet link
```

`link`コマンドを使用してValetにアプリケーションをリンクした後、アプリケーションにはディレクトリ名を使用してアクセスできます。したがって、上記の例でリンクされたサイトは`http://laravel.test`でアクセスできます。さらに、Valetはワイルドカードサブドメイン（`http://foo.laravel.test`）を使用してサイトにアクセスできるように自動的に許可します。

アプリケーションを別のホスト名で提供したい場合は、`link`コマンドにホスト名を渡すことができます。たとえば、次のコマンドを実行して、アプリケーションを`http://application.test`で利用できるようにすることができます：

```shell
cd ~/Sites/laravel

valet link application
```

もちろん、`link`コマンドを使用してサブドメインでアプリケーションを提供することもできます：

```shell
valet link api.application
```

リンクされたすべてのディレクトリのリストを表示するには、`links`コマンドを実行できます：

```shell
valet links
```

サイトのシンボリックリンクを破棄するには、`unlink`コマンドを使用できます：

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### TLSでサイトを保護する

デフォルトで、ValetはHTTP経由でサイトを提供します。ただし、HTTP/2を使用して暗号化されたTLSでサイトを提供したい場合は、`secure`コマンドを使用できます。たとえば、サイトが`laravel.test`ドメインでValetによって提供されている場合、次のコマンドを実行してサイトを保護する必要があります：

```shell
valet secure laravel
```

サイトを「非セキュア」にし、プレーンなHTTP経由でトラフィックを提供するように戻すには、`unsecure`コマンドを使用します。`secure`コマンドと同様に、このコマンドはセキュアにしたいホスト名を受け付けます：

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### デフォルトサイトの提供

未知の`test`ドメインにアクセスした際に`404`ではなく、「デフォルト」サイトを提供するようにValetを設定したい場合があります。これを実現するには、`~/.config/valet/config.json`設定ファイルに`default`オプションを追加し、デフォルトサイトとして提供するサイトのパスを指定します：

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### サイトごとのPHPバージョン

デフォルトでは、ValetはグローバルなPHPインストールを使用してサイトを提供します。しかし、複数のサイトで異なるPHPバージョンをサポートする必要がある場合、`isolate`コマンドを使用して特定のサイトが使用するPHPバージョンを指定できます。`isolate`コマンドは、現在の作業ディレクトリにあるサイトに対して指定されたPHPバージョンを使用するようにValetを設定します：

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

サイト名がそのディレクトリ名と一致しない場合、`--site`オプションを使用してサイト名を指定できます：

```shell
valet isolate php@8.0 --site="site-name"
```

便宜上、`valet php`、`valet composer`、`valet which-php`コマンドを使用して、サイトの設定されたPHPバージョンに基づいて適切なPHP CLIまたはツールへのプロキシコールを行うことができます：

```shell
valet php
valet composer
valet which-php
```

`isolated`コマンドを実行して、すべての分離されたサイトとそのPHPバージョンのリストを表示できます：

```shell
valet isolated
```

サイトをValetのグローバルにインストールされたPHPバージョンに戻すには、サイトのルートディレクトリから`unisolate`コマンドを呼び出します：

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## サイトの共有

Valetには、ローカルサイトを世界と共有するためのコマンドが含まれており、モバイルデバイスでのサイトテストやチームメンバーやクライアントとの共有を簡単に行うことができます。

デフォルトでは、Valetはサイトの共有をngrokまたはExpose経由でサポートしています。サイトを共有する前に、`share-tool`コマンドを使用してValetの設定を更新し、`ngrok`または`expose`を指定します：

```shell
valet share-tool ngrok
```

ツールを選択し、Homebrew（ngrokの場合）またはComposer（Exposeの場合）経由でインストールしていない場合、Valetは自動的にインストールを促します。もちろん、両方のツールでは、サイトの共有を開始する前にngrokまたはExposeアカウントを認証する必要があります。

サイトを共有するには、ターミナルでサイトのディレクトリに移動し、Valetの`share`コマンドを実行します。公開可能なURLがクリップボードにコピーされ、ブラウザに直接貼り付けるか、チームと共有する準備が整います：

```shell
cd ~/Sites/laravel

valet share
```

サイトの共有を停止するには、`Control + C`を押します。

> WARNING:  
> カスタムDNSサーバー（例：`1.1.1.1`）を使用している場合、ngrokの共有が正しく機能しないことがあります。この場合、Macのシステム設定を開き、ネットワーク設定に移動し、詳細設定を開き、DNSタブに移動して、`127.0.0.1`を最初のDNSサーバーとして追加してください。

<a name="sharing-sites-via-ngrok"></a>
#### Ngrok経由でのサイト共有

ngrokを使用してサイトを共有するには、[ngrokアカウントを作成](https://dashboard.ngrok.com/signup)し、[認証トークンを設定](https://dashboard.ngrok.com/get-started/your-authtoken)する必要があります。認証トークンを取得したら、Valetの設定をそのトークンで更新できます：

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> NOTE:  
> `valet share --region=eu`のように、追加のngrokパラメータをshareコマンドに渡すことができます。詳細については、[ngrokのドキュメント](https://ngrok.com/docs)を参照してください。

<a name="sharing-sites-via-expose"></a>
#### Expose経由でのサイト共有

Exposeを使用してサイトを共有するには、[Exposeアカウントを作成](https://expose.dev/register)し、[認証トークンを使用してExposeを認証](https://expose.dev/docs/getting-started/getting-your-token)する必要があります。

追加のコマンドラインパラメータについては、[Exposeのドキュメント](https://expose.dev/docs)を参照してください。

<a name="sharing-sites-on-your-local-network"></a>
### ローカルネットワーク上でのサイト共有

Valetは、デフォルトでは内部の`127.0.0.1`インターフェースへの着信トラフィックを制限するため、開発マシンがインターネットからのセキュリティリスクにさらされることはありません。

ローカルネットワーク上の他のデバイスが、マシンのIPアドレス（例：`192.168.1.10/application.test`）を介してValetサイトにアクセスできるようにするには、そのサイトの適切なNginx設定ファイルを手動で編集して、`listen`ディレクティブの制限を削除する必要があります。ポート80と443の`listen`ディレクティブから`127.0.0.1:`プレフィックスを削除する必要があります。

プロジェクトに対して`valet secure`を実行していない場合、すべての非HTTPSサイトのネットワークアクセスを開放するには、`/usr/local/etc/nginx/valet/valet.conf`ファイルを編集します。ただし、プロジェクトサイトをHTTPS経由で提供している場合（サイトに対して`valet secure`を実行している場合）、`~/.config/valet/Nginx/app-name.test`ファイルを編集する必要があります。

Nginxの設定を更新したら、`valet restart`コマンドを実行して設定の変更を適用します。

<a name="site-specific-environment-variables"></a>
## サイト固有の環境変数

他のフレームワークを使用する一部のアプリケーションは、サーバー環境変数に依存している場合がありますが、プロジェクト内でそれらの変数を設定する方法を提供していないことがあります。Valetでは、プロジェクトのルートに`.valet-env.php`ファイルを追加することで、サイト固有の環境変数を設定できます。このファイルは、配列を返す必要があり、配列内の各サイトに対してグローバルな`$_SERVER`配列に追加される環境変数のペアを指定します：

    <?php

    return [
        // Set $_SERVER['key'] to "value" for the laravel.test site...
        'laravel' => [
            'key' => 'value',
        ],

        // Set $_SERVER['key'] to "value" for all sites...
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## サービスのプロキシ

ValetとDockerが同時にポート80をバインドできないため、Valetを実行しながらDockerで別のサイトを実行する必要がある場合があります。これを解決するには、`proxy`コマンドを使用してプロキシを生成できます。例えば、`http://elasticsearch.test`から`http://127.0.0.1:9200`へのすべてのトラフィックをプロキシするには：

```shell
# HTTP経由でプロキシ...
valet proxy elasticsearch http://127.0.0.1:9200

# TLS + HTTP/2経由でプロキシ...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

プロキシを削除するには、`unproxy`コマンドを使用します：

```shell
valet unproxy elasticsearch
```

プロキシされたすべてのサイト設定をリストするには、`proxies`コマンドを使用します：

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## カスタムValetドライバ

独自のValet「ドライバ」を作成して、ValetにネイティブでサポートされていないフレームワークやCMSで実行されているPHPアプリケーションを提供することができます。Valetをインストールすると、`~/.config/valet/Drivers`ディレクトリが作成され、その中に`SampleValetDriver.php`ファイルが含まれます。このファイルには、カスタムドライバの実装方法を示すサンプルドライバの実装が含まれています。ドライバを作成するには、`serves`、`isStaticFile`、`frontControllerPath`の3つのメソッドを実装する必要があります。

これらの3つのメソッドは、`$sitePath`、`$siteName`、`$uri`の値を引数として受け取ります。`$sitePath`は、マシン上で提供されるサイトへの完全修飾パスです（例：`/Users/Lisa/Sites/my-project`）。`$siteName`は、ドメインの「ホスト」/「サイト名」部分です（`my-project`）。`$uri`は、着信リクエストのURIです（`/foo/bar`）。

カスタムValetドライバを作成したら、`FrameworkValetDriver.php`の命名規則に従って`~/.config/valet/Drivers`ディレクトリに配置します。例えば、WordPress用のカスタムValetドライバを作成する場合、ファイル名は`WordPressValetDriver.php`とする必要があります。

カスタムValetドライバが実装する必要のある各メソッドのサンプル実装を見てみましょう。

<a name="the-serves-method"></a>
#### `serves`メソッド

`serves`メソッドは、ドライバが着信リクエストを処理する必要がある場合に`true`を返す必要があります。それ以外の場合、メソッドは`false`を返す必要があります。そのため、このメソッド内で、指定された`$sitePath`に試行しているプロジェクトのタイプが含まれているかどうかを判断する必要があります。

例えば、`WordPressValetDriver`を作成するとします。`serves`メソッドは次のようになります：

    /**
     * Determine if the driver serves the request.
     */
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>
#### `isStaticFile`メソッド

`isStaticFile`メソッドは、着信リクエストが画像やスタイルシートなどの「静的」ファイルであるかどうかを判断する必要があります。静的ファイルである場合、メソッドはディスク上の静的ファイルへの完全修飾パスを返す必要があります。着信リクエストが静的ファイルでない場合、メソッドは`false`を返す必要があります：

    /**
     * Determine if the incoming request is for a static file.
     *
     * @return string|false
     */
    public function isStaticFile(string $sitePath, string $siteName, string $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> WARNING:  
> `isStaticFile`メソッドは、`serves`メソッドが着信リクエストに対して`true`を返し、リクエストURIが`/`でない場合にのみ呼び出されます。

<a name="the-frontcontrollerpath-method"></a>
#### `frontControllerPath`メソッド

`frontControllerPath`メソッドは、アプリケーションの「フロントコントローラ」への完全修飾パスを返す必要があります。通常は「index.php」ファイルです：

    /**
     * Get the fully resolved path to the application's front controller.
     */
    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### ローカルドライバ

単一のアプリケーションに対してカスタムValetドライバを定義したい場合は、アプリケーションのルートディレクトリに`LocalValetDriver.php`ファイルを作成します。カスタムドライバは、基本の`ValetDriver`クラスを拡張するか、`LaravelValetDriver`などの既存のアプリケーション固有のドライバを拡張することができます。

    use Valet\Drivers\LaravelValetDriver;

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * ドライバがリクエストを処理するかどうかを判断します。
         */
        public function serves(string $sitePath, string $siteName, string $uri): bool
        {
            return true;
        }

        /**
         * アプリケーションのフロントコントローラへの完全解決パスを取得します。
         */
        public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## その他のValetコマンド

<div class="overflow-auto" markdown=1>

| コマンド | 説明 |
| --- | --- |
| `valet list` | すべてのValetコマンドのリストを表示します。 |
| `valet diagnose` | Valetのデバッグに役立つ診断情報を出力します。 |
| `valet directory-listing` | ディレクトリリスティングの動作を決定します。デフォルトは "off" で、ディレクトリに対して404ページを表示します。 |
| `valet forget` | "parked" ディレクトリからこのコマンドを実行して、parkedディレクトリリストから削除します。 |
| `valet log` | Valetのサービスによって書き込まれたログのリストを表示します。 |
| `valet paths` | すべての "parked" パスを表示します。 |
| `valet restart` | Valetデーモンを再起動します。 |
| `valet start` | Valetデーモンを起動します。 |
| `valet stop` | Valetデーモンを停止します。 |
| `valet trust` | BrewとValetのsudoersファイルを追加して、パスワードを要求せずにValetコマンドを実行できるようにします。 |
| `valet uninstall` | Valetをアンインストールします。手動でのアンインストール手順を表示します。`--force`オプションを渡すと、Valetのすべてのリソースを積極的に削除します。 |

</div>

<a name="valet-directories-and-files"></a>
## Valetのディレクトリとファイル

Valet環境のトラブルシューティング中に、以下のディレクトリとファイル情報が役立つ場合があります。

#### `~/.config/valet`

Valetのすべての設定が含まれています。このディレクトリのバックアップを維持することをお勧めします。

#### `~/.config/valet/dnsmasq.d/`

このディレクトリには、DNSMasqの設定が含まれています。

#### `~/.config/valet/Drivers/`

このディレクトリには、Valetのドライバが含まれています。ドライバは、特定のフレームワーク/CMSがどのように提供されるかを決定します。

#### `~/.config/valet/Nginx/`

このディレクトリには、ValetのすべてのNginxサイト設定が含まれています。これらのファイルは、`install`および`secure`コマンドを実行すると再構築されます。

#### `~/.config/valet/Sites/`

このディレクトリには、[リンクされたプロジェクト](#the-link-command)のすべてのシンボリックリンクが含まれています。

#### `~/.config/valet/config.json`

このファイルは、Valetのマスター設定ファイルです。

#### `~/.config/valet/valet.sock`

このファイルは、ValetのNginxインストールで使用されるPHP-FPMソケットです。PHPが正しく実行されている場合にのみ存在します。

#### `~/.config/valet/Log/fpm-php.www.log`

このファイルは、PHPエラーのユーザーログです。

#### `~/.config/valet/Log/nginx-error.log`

このファイルは、Nginxエラーのユーザーログです。

#### `/usr/local/var/log/php-fpm.log`

このファイルは、PHP-FPMエラーのシステムログです。

#### `/usr/local/var/log/nginx`

このディレクトリには、Nginxのアクセスログとエラーログが含まれています。

#### `/usr/local/etc/php/X.X/conf.d`

このディレクトリには、さまざまなPHP設定の`*.ini`ファイルが含まれています。

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

このファイルは、PHP-FPMプール設定ファイルです。

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

このファイルは、サイトのSSL証明書を構築するために使用されるデフォルトのNginx設定です。

<a name="disk-access"></a>
### ディスクアクセス

macOS 10.14以降、[一部のファイルとディレクトリへのアクセスはデフォルトで制限されています](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf)。これらの制限には、デスクトップ、ドキュメント、ダウンロードディレクトリが含まれます。さらに、ネットワークボリュームとリムーバブルボリュームへのアクセスも制限されています。したがって、Valetはサイトフォルダがこれらの保護された場所の外にあることを推奨します。

しかし、これらの場所からサイトを提供したい場合は、Nginxに「フルディスクアクセス」権限を与える必要があります。そうしないと、特に静的アセットの提供時に、サーバーエラーやその他の予期せぬ動作がNginxで発生する可能性があります。通常、macOSは自動的にこれらの場所へのフルアクセスをNginxに許可するようプロンプトを表示します。または、手動で`システム環境設定` > `セキュリティとプライバシー` > `プライバシー`から`フルディスクアクセス`を選択し、メインウィンドウペインで`nginx`のエントリを有効にすることもできます。
