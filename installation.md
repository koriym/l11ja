# インストール

- [Laravelについて](#meet-laravel)
    - [なぜLaravelなのか？](#why-laravel)
- [Laravelプロジェクトの作成](#creating-a-laravel-project)
- [初期設定](#initial-configuration)
    - [環境ベースの設定](#environment-based-configuration)
    - [データベースとマイグレーション](#databases-and-migrations)
    - [ディレクトリ設定](#directory-configuration)
- [Herdを使用したローカルインストール](#local-installation-using-herd)
    - [macOSでのHerd](#herd-on-macos)
    - [WindowsでのHerd](#herd-on-windows)
- [Sailを使用したDockerインストール](#docker-installation-using-sail)
    - [macOSでのSail](#sail-on-macos)
    - [WindowsでのSail](#sail-on-windows)
    - [LinuxでのSail](#sail-on-linux)
    - [Sailサービスの選択](#choosing-your-sail-services)
- [IDEサポート](#ide-support)
- [次のステップ](#next-steps)
    - [Laravel、フルスタックフレームワーク](#laravel-the-fullstack-framework)
    - [Laravel、APIバックエンド](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## Laravelについて

Laravelは、表現力豊かでエレガントな構文を持つWebアプリケーションフレームワークです。Webフレームワークは、アプリケーションを作成するための構造と出発点を提供し、詳細を気にせずに素晴らしいものを作成することに集中できるようにします。

Laravelは、驚くべき開発者体験を提供すると同時に、徹底的な依存性注入、表現力豊かなデータベース抽象化レイヤー、キューとスケジュールされたジョブ、ユニットテストと統合テストなどの強力な機能を提供することを目指しています。

PHPのWebフレームワークを初めて使用する方でも、何年もの経験を持つ方でも、Laravelはあなたと共に成長するフレームワークです。Web開発者としての最初の一歩を踏み出すのを手助けしたり、専門知識を次のレベルに引き上げるのを支援します。あなたが何を作るかを見るのが楽しみです。

> NOTE:  
> Laravelを初めて使う方は、[Laravel Bootcamp](https://bootcamp.laravel.com)をチェックしてください。フレームワークのハンズオンツアーを通じて、初めてのLaravelアプリケーションを構築する過程を一緒に進めます。

<a name="why-laravel"></a>
### なぜLaravelなのか？

Webアプリケーションを構築する際には、さまざまなツールやフレームワークが利用可能です。しかし、私たちはLaravelが現代のフルスタックWebアプリケーションを構築するための最良の選択肢であると考えています。

#### プログレッシブフレームワーク

Laravelを「プログレッシブ」フレームワークと呼ぶことがあります。つまり、Laravelはあなたと共に成長します。Web開発の最初の一歩を踏み出す場合、Laravelの膨大なドキュメント、ガイド、[ビデオチュートリアル](https://laracasts.com)が、圧倒されることなく学習をサポートします。

シニア開発者であれば、Laravelは[依存性注入](container.md)、[ユニットテスト](testing.md)、[キュー](queues.md)、[リアルタイムイベント](broadcasting.md)などの強力なツールを提供します。Laravelは、プロフェッショナルなWebアプリケーションを構築するために微調整されており、企業のワークロードに対応できる準備が整っています。

#### スケーラブルなフレームワーク

Laravelは非常にスケーラブルです。PHPのスケーリングに適した性質と、Laravelの組み込みサポートであるRedisのような高速で分散型のキャッシュシステムのおかげで、Laravelでの水平スケーリングは簡単です。実際、Laravelアプリケーションは月に何億ものリクエストを簡単に処理できるようにスケーリングされています。

極端なスケーリングが必要ですか？[Laravel Vapor](https://vapor.laravel.com)のようなプラットフォームでは、AWSの最新のサーバーレステクノロジーを使用して、ほぼ無制限のスケールでLaravelアプリケーションを実行できます。

#### コミュニティフレームワーク

Laravelは、PHPエコシステムの最高のパッケージを組み合わせて、最も堅牢で開発者に優しいフレームワークを提供します。さらに、世界中から何千人もの才能ある開発者が[フレームワークに貢献](https://github.com/laravel/framework)しています。誰もがLaravelの貢献者になるかもしれません。

<a name="creating-a-laravel-project"></a>
## Laravelプロジェクトの作成

最初のLaravelプロジェクトを作成する前に、ローカルマシンにPHPと[Composer](https://getcomposer.org)がインストールされていることを確認してください。macOSまたはWindowsで開発している場合、PHP、Composer、Node、NPMは[Laravel Herd](#local-installation-using-herd)を介して数分でインストールできます。

PHPとComposerをインストールした後、Composerの`create-project`コマンドを使用して新しいLaravelプロジェクトを作成できます：

```nothing
composer create-project laravel/laravel example-app
```

または、[Laravelインストーラー](https://github.com/laravel/installer)をComposerを介してグローバルにインストールすることで、新しいLaravelプロジェクトを作成できます。Laravelインストーラーを使用すると、新しいアプリケーションを作成する際に、好みのテストフレームワーク、データベース、スターターキットを選択できます：

```nothing
composer global require laravel/installer

laravel new example-app
```

プロジェクトが作成されたら、Laravel Artisanの`serve`コマンドを使用してLaravelのローカル開発サーバーを起動します：

```nothing
cd example-app

php artisan serve
```

Artisan開発サーバーを起動したら、アプリケーションはWebブラウザで[http://localhost:8000](http://localhost:8000)からアクセスできます。次に、[Laravelエコシステムの次のステップに進む](#next-steps)準備が整いました。もちろん、[データベースの設定](#databases-and-migrations)も行いたいかもしれません。

> NOTE:  
> Laravelアプリケーションの開発を始める際に先を急ぎたい場合は、[スターターキット](starter-kits.md)のいずれかを使用することを検討してください。Laravelのスターターキットは、新しいLaravelアプリケーションのためのバックエンドとフロントエンドの認証スキャフォールディングを提供します。

<a name="initial-configuration"></a>
## 初期設定

Laravelフレームワークのすべての設定ファイルは、`config`ディレクトリに保存されています。各オプションにはドキュメントがあるので、ファイルを見て、利用可能なオプションに慣れ親しんでください。

Laravelは、ほとんど追加の設定を必要としません。自由に開発を始めることができます！ただし、`config/app.php`ファイルとそのドキュメントを確認することをお勧めします。アプリケーションに応じて変更したい`timezone`や`locale`などのオプションが含まれています。

<a name="environment-based-configuration"></a>
### 環境ベースの設定

Laravelの多くの設定オプションの値は、アプリケーションがローカルマシンで実行されているか、本番Webサーバーで実行されているかによって異なる場合があります。そのため、多くの重要な設定値は、アプリケーションのルートに存在する`.env`ファイルを使用して定義されます。

`.env`ファイルは、アプリケーションのソース管理にコミットしないでください。アプリケーションを使用する各開発者/サーバーは、異なる環境設定を必要とする可能性があるためです。さらに、侵入者がソース管理リポジトリにアクセスした場合、機密の認証情報が公開されるリスクがあります。

> NOTE:  
> `.env`ファイルと環境ベースの設定についての詳細は、完全な[設定ドキュメント](configuration.md#environment-configuration)を確認してください。

<a name="databases-and-migrations"></a>
### データベースとマイグレーション

Laravelアプリケーションを作成したので、おそらくデータベースにデータを保存したいと思うでしょう。デフォルトでは、アプリケーションの`.env`設定ファイルは、LaravelがSQLiteデータベースと対話することを指定しています。

プロジェクトの作成中に、Laravelは`database/database.sqlite`ファイルを作成し、アプリケーションのデータベーステーブルを作成するために必要なマイグレーションを実行しました。

MySQLやPostgreSQLなどの別のデータベースドライバを使用する場合は、`.env`設定ファイルを更新して適切なデータベースを使用するようにします。たとえば、MySQLを使用する場合は、`.env`設定ファイルの`DB_*`変数を次のように更新します：

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

SQLite以外のデータベースを使用する場合は、データベースを作成し、アプリケーションの[データベースマイグレーション](migrations.md)を実行する必要があります：

```shell
php artisan migrate
```

> NOTE:  
> macOSまたはWindowsで開発しており、MySQL、PostgreSQL、またはRedisをローカルにインストールする必要がある場合は、[Herd Pro](https://herd.laravel.com/#plans)の使用を検討してください。

<a name="directory-configuration"></a>
### ディレクトリ設定

Laravelは、Webサーバー用に設定された「Webディレクトリ」のルートから常に提供されるべきです。Laravelアプリケーションを「Webディレクトリ」のサブディレクトリから提供しようとしないでください。そうすると、アプリケーション内に存在する機密ファイルが公開される可能性があります。

<a name="local-installation-using-herd"></a>
## Herdを使用したローカルインストール

[Laravel Herd](https://herd.laravel.com)は、macOSとWindows用の超高速なネイティブLaravelおよびPHP開発環境です。Herdには、Laravel開発を始めるために必要なすべてが含まれています。PHPとNginxを含む。

Herdをインストールしたら、Laravelでの開発を始める準備が整います。Herdには、`php`、`composer`、`laravel`、`expose`、`node`、`npm`、`nvm`のコマンドラインツールが含まれています。

> NOTE:  
> [Herd Pro](https://herd.laravel.com/#plans)は、Herdをさらに強力な機能で拡張します。例えば、ローカルのMySQL、Postgres、Redisデータベースの作成と管理、ローカルメールの表示、ログの監視などが可能です。

<a name="herd-on-macos"></a>
### macOSでのHerd

macOSで開発している場合、[HerdのWebサイト](https://herd.laravel.com)からHerdインストーラーをダウンロードできます。インストーラーは、最新バージョンのPHPを自動的にダウンロードし、Macを常に[Nginx](https://www.nginx.com/)をバックグラウンドで実行するように設定します。

macOS用のHerdは、「パークされた」ディレクトリをサポートするために[dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq)を使用します。パークされたディレクトリ内のLaravelアプリケーションは、自動的にHerdによって提供されます。デフォルトでは、Herdは`~/Herd`にパークされたディレクトリを作成し、このディレクトリ内のLaravelアプリケーションには、ディレクトリ名を使用して`.test`ドメインでアクセスできます。

<a name="herd-on-windows"></a>
### WindowsでのHerd

Windowsで開発している場合、[HerdのWebサイト](https://herd.laravel.com)からHerdインストーラーをダウンロードできます。インストーラーは、最新バージョンのPHPを自動的にダウンロードし、Windowsを常に[Nginx](https://www.nginx.com/)をバックグラウンドで実行するように設定します。

Windows用のHerdは、「パークされた」ディレクトリをサポートするために[dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq)を使用します。パークされたディレクトリ内のLaravelアプリケーションは、自動的にHerdによって提供されます。デフォルトでは、Herdは`~/Herd`にパークされたディレクトリを作成し、このディレクトリ内のLaravelアプリケーションには、ディレクトリ名を使用して`.test`ドメインでアクセスできます。

<a name="docker-installation-using-sail"></a>
## Sailを使用したDockerインストール

[Laravel Sail](https://laravel.com/docs/sail)は、LaravelのデフォルトのDocker開発環境です。Sailは、Dockerを使用してLaravelアプリケーションを開発するための簡単な方法を提供します。

Sailを使用してLaravelプロジェクトを作成するには、Composerの`create-project`コマンドを使用します：

```nothing
composer create-project laravel/laravel example-app
```

プロジェクトが作成されたら、Sailを起動します：

```nothing
cd example-app

./vendor/bin/sail up
```

Sailを起動したら、アプリケーションはWebブラウザで[http://localhost](http://localhost)からアクセスできます。次に、[Laravelエコシステムの次のステップに進む](#next-steps)準備が整いました。もちろん、[データベースの設定](#databases-and-migrations)も行いたいかもしれません。

<a name="sail-on-macos"></a>
### macOSでのSail

macOSで開発している場合、[Docker Desktop](https://www.docker.com/products/docker-desktop)をインストールする必要があります。Docker Desktopをインストールしたら、Sailを使用してLaravelプロジェクトを作成できます。

<a name="sail-on-windows"></a>
### WindowsでのSail

Windowsで開発している場合、[Docker Desktop](https://www.docker.com/products/docker-desktop)をインストールする必要があります。Docker Desktopをインストールしたら、Sailを使用してLaravelプロジェクトを作成できます。

<a name="sail-on-linux"></a>
### LinuxでのSail

Linuxで開発している場合、[Docker](https://www.docker.com/products/docker-desktop)をインストールする必要があります。Dockerをインストールしたら、Sailを使用してLaravelプロジェクトを作成できます。

<a name="choosing-your-sail-services"></a>
### Sailサービスの選択

Sailを使用してLaravelプロジェクトを作成する際に、使用するサービスを選択できます。デフォルトでは、SailはMySQL、Redis、Memcached、MeiliSearch、MailHog、およびNodeを使用します。

サービスを選択するには、`create-project`コマンドに`--services`オプションを追加します：

```nothing
composer create-project laravel/laravel example-app --services=mysql,redis
```

<a name="ide-support"></a>
## IDEサポート

Laravelは、[VS Code](https://code.visualstudio.com/)や[PHPStorm](https://www.jetbrains.com/phpstorm/)などの一般的なIDEをサポートしています。Laravelアプリケーションを開発する際に、これらのIDEを使用することを強くお勧めします。

<a name="next-steps"></a>
## 次のステップ

Laravelアプリケーションの開発を始めたら、次のステップに進む準備が整いました。

<a name="laravel-the-fullstack-framework"></a>
### Laravel、フルスタックフレームワーク

Laravelは、フルスタックフレームワークとして使用できます。Laravelを使用して、バックエンドとフロントエンドの両方を開発できます。Laravelは、バックエンドの開発に必要なすべてのツールを提供し、フロントエンドの開発に必要なツールも提供します。

<a name="laravel-the-api-backend"></a>
### Laravel、APIバックエンド

Laravelは、APIバックエンドとして使用できます。Laravelを使用して、フロントエンドアプリケーションのAPIバックエンドを開発できます。Laravelは、APIバックエンドの開発に必要なすべてのツールを提供します。

Herdをインストールした後、新しいLaravelプロジェクトを作成する最も速い方法は、HerdにバンドルされているLaravel CLIを使用することです。

```nothing
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

もちろん、パークされたディレクトリやその他のPHP設定は、システムトレイのHerdメニューから開くことができるHerdのUIを介していつでも管理できます。

Herdの詳細については、[Herdのドキュメント](https://herd.laravel.com/docs)を確認してください。

<a name="herd-on-windows"></a>
### WindowsでのHerd

Windows用のインストーラーは、[Herdのウェブサイト](https://herd.laravel.com/windows)からダウンロードできます。インストールが完了したら、Herdを起動してオンボーディングプロセスを完了し、初めてHerdのUIにアクセスできます。

HerdのUIには、Herdのシステムトレイアイコンを左クリックすることでアクセスできます。右クリックすると、毎日必要なすべてのツールにアクセスできるクイックメニューが開きます。

インストール中に、Herdはホームディレクトリに`%USERPROFILE%\Herd`に「パークされた」ディレクトリを作成します。パークされたディレクトリ内のLaravelアプリケーションは自動的にHerdによって提供され、このディレクトリ内の任意のLaravelアプリケーションには、そのディレクトリ名を使用して`.test`ドメインでアクセスできます。

Herdをインストールした後、新しいLaravelプロジェクトを作成する最も速い方法は、HerdにバンドルされているLaravel CLIを使用することです。始めるには、Powershellを開いて次のコマンドを実行してください：

```nothing
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

Herdの詳細については、[Windows用のHerdドキュメント](https://herd.laravel.com/docs/windows)を確認してください。

<a name="docker-installation-using-sail"></a>
## Dockerを使用したインストール（Sailを使用）

お好みのオペレーティングシステムに関係なく、Laravelをできるだけ簡単に始められるようにしたいと考えています。そのため、ローカルマシンでLaravelプロジェクトを開発して実行するためのさまざまなオプションが用意されています。後でこれらのオプションを探求することもできますが、Laravelは[Sail](sail.md)を提供しており、これはLaravelプロジェクトを[Docker](https://www.docker.com)を使用して実行するための組み込みのソリューションです。

Dockerは、アプリケーションやサービスを小さく軽量な「コンテナ」で実行するためのツールで、ローカルマシンにインストールされたソフトウェアや設定に干渉しません。つまり、ローカルマシンにWebサーバーやデータベースなどの複雑な開発ツールを設定する心配はありません。始めるには、[Docker Desktop](https://www.docker.com/products/docker-desktop)をインストールするだけです。

Laravel Sailは、LaravelのデフォルトのDocker設定と対話するための軽量なコマンドラインインターフェースです。Sailは、Dockerの経験がなくてもPHP、MySQL、Redisを使用してLaravelアプリケーションを構築するための素晴らしい出発点を提供します。

> NOTE:  
> すでにDockerの専門家ですか？心配はいりません！Sailのすべては、Laravelに含まれる`docker-compose.yml`ファイルを使用してカスタマイズできます。

<a name="sail-on-macos"></a>
### macOSでのSail

Macで開発していて、[Docker Desktop](https://www.docker.com/products/docker-desktop)がすでにインストールされている場合、シンプルなターミナルコマンドを使用して新しいLaravelプロジェクトを作成できます。たとえば、"example-app"という名前のディレクトリに新しいLaravelアプリケーションを作成するには、ターミナルで次のコマンドを実行します：

```shell
curl -s "https://laravel.build/example-app" | bash
```

もちろん、このURLの"example-app"は好きなものに変更できますが、アプリケーション名には英数字、ダッシュ、アンダースコアのみを含めるようにしてください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリ内に作成されます。

Sailのインストールには、Sailのアプリケーションコンテナがローカルマシン上で構築される間、数分かかる場合があります。

プロジェクトが作成されたら、アプリケーションディレクトリに移動し、Laravel Sailを起動できます。Laravel Sailは、LaravelのデフォルトのDocker設定と対話するためのシンプルなコマンドラインインターフェースを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

アプリケーションのDockerコンテナが起動したら、アプリケーションの[データベースマイグレーション](migrations.md)を実行する必要があります：

```shell
./vendor/bin/sail artisan migrate
```

最後に、アプリケーションにはWebブラウザでhttp://localhostにアクセスできます。

> NOTE:  
> Laravel Sailの詳細を学び続けるには、その[完全なドキュメント](sail.md)を確認してください。

<a name="sail-on-windows"></a>
### WindowsでのSail

Windowsマシンで新しいLaravelアプリケーションを作成する前に、[Docker Desktop](https://www.docker.com/products/docker-desktop)をインストールしてください。次に、Windows Subsystem for Linux 2 (WSL2)がインストールされ、有効になっていることを確認してください。WSLは、Windows 10上でLinuxバイナリ実行ファイルをネイティブに実行できるようにします。WSL2のインストールと有効化方法については、Microsoftの[開発者環境ドキュメント](https://docs.microsoft.com/en-us/windows/wsl/install-win10)に記載されています。

> NOTE:  
> WSL2をインストールして有効にした後、Docker Desktopが[WSL2バックエンドを使用するように設定されている](https://docs.docker.com/docker-for-windows/wsl/)ことを確認してください。

次に、最初のLaravelプロジェクトを作成する準備が整いました。[Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab)を起動し、WSL2 Linuxオペレーティングシステムの新しいターミナルセッションを開始します。次に、シンプルなターミナルコマンドを使用して新しいLaravelプロジェクトを作成できます。たとえば、"example-app"という名前のディレクトリに新しいLaravelアプリケーションを作成するには、ターミナルで次のコマンドを実行します：

```shell
curl -s https://laravel.build/example-app | bash
```

もちろん、このURLの"example-app"は好きなものに変更できますが、アプリケーション名には英数字、ダッシュ、アンダースコアのみを含めるようにしてください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリ内に作成されます。

Sailのインストールには、Sailのアプリケーションコンテナがローカルマシン上で構築される間、数分かかる場合があります。

プロジェクトが作成されたら、アプリケーションディレクトリに移動し、Laravel Sailを起動できます。Laravel Sailは、LaravelのデフォルトのDocker設定と対話するためのシンプルなコマンドラインインターフェースを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

アプリケーションのDockerコンテナが起動したら、アプリケーションの[データベースマイグレーション](migrations.md)を実行する必要があります：

```shell
./vendor/bin/sail artisan migrate
```

最後に、アプリケーションにはWebブラウザでhttp://localhostにアクセスできます。

> NOTE:  
> Laravel Sailの詳細を学び続けるには、その[完全なドキュメント](sail.md)を確認してください。

#### WSL2内での開発

もちろん、WSL2インストール内に作成されたLaravelアプリケーションファイルを変更できる必要があります。これを実現するために、Microsoftの[Visual Studio Code](https://code.visualstudio.com)エディタと、[Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)用の公式拡張機能を使用することをお勧めします。

これらのツールがインストールされたら、Windows Terminalを使用してアプリケーションのルートディレクトリから`code .`コマンドを実行することで、任意のLaravelプロジェクトを開くことができます。

<a name="sail-on-linux"></a>
### LinuxでのSail

Linuxで開発していて、[Docker Compose](https://docs.docker.com/compose/install/)がすでにインストールされている場合、シンプルなターミナルコマンドを使用して新しいLaravelプロジェクトを作成できます。

まず、Docker Desktop for Linuxを使用している場合、次のコマンドを実行してください。Docker Desktop for Linuxを使用していない場合は、このステップをスキップできます：

```shell
docker context use default
```

次に、"example-app"という名前のディレクトリに新しいLaravelアプリケーションを作成するには、ターミナルで次のコマンドを実行します：

```shell
curl -s https://laravel.build/example-app | bash
```

もちろん、このURLの"example-app"は好きなものに変更できますが、アプリケーション名には英数字、ダッシュ、アンダースコアのみを含めるようにしてください。Laravelアプリケーションのディレクトリは、コマンドを実行したディレクトリ内に作成されます。

Sailのインストールには、Sailのアプリケーションコンテナがローカルマシン上で構築される間、数分かかる場合があります。

プロジェクトが作成されたら、アプリケーションディレクトリに移動し、Laravel Sailを起動できます。Laravel Sailは、LaravelのデフォルトのDocker設定と対話するためのシンプルなコマンドラインインターフェースを提供します：

```shell
cd example-app

./vendor/bin/sail up
```

アプリケーションのDockerコンテナが起動したら、アプリケーションの[データベースマイグレーション](migrations.md)を実行する必要があります：

```shell
./vendor/bin/sail artisan migrate
```

最後に、アプリケーションにはWebブラウザでhttp://localhostにアクセスできます。

> NOTE:  
> Laravel Sailの詳細を学び続けるには、その[完全なドキュメント](sail.md)を確認してください。

<a name="choosing-your-sail-services"></a>
### Sailサービスの選択

新しいLaravelアプリケーションをSailを介して作成する際、`with`クエリ文字列変数を使用して、新しいアプリケーションの`docker-compose.yml`ファイルに設定するサービスを選択できます。利用可能なサービスには、`mysql`、`pgsql`、`mariadb`、`redis`、`memcached`、`meilisearch`、`typesense`、`minio`、`selenium`、`mailpit`が含まれます：

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

設定するサービスを指定しない場合、`mysql`、`redis`、`meilisearch`、`mailpit`、`selenium`のデフォルトスタックが設定されます。

`devcontainer`パラメータをURLに追加することで、デフォルトの[Devcontainer](sail.md#using-devcontainers)をインストールするようにSailに指示できます：

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="ide-support"></a>
## IDEサポート

Laravelアプリケーションの開発には、お好きなコードエディタを自由にお使いいただけますが、[PhpStorm](https://www.jetbrains.com/phpstorm/laravel/)はLaravelとそのエコシステムに対して広範なサポートを提供しており、[Laravel Pint](https://www.jetbrains.com/help/phpstorm/using-laravel-pint.html)も含まれています。

さらに、コミュニティがメンテナンスしている[Laravel Idea](https://laravel-idea.com/)のPhpStormプラグインは、コード生成、Eloquent構文の補完、バリデーションルールの補完など、さまざまな便利なIDE拡張機能を提供しています。

<a name="next-steps"></a>
## 次のステップ

これでLaravelプロジェクトを作成できましたが、次に何を学ぶべきか迷っているかもしれません。まず、Laravelの動作を理解するために、以下のドキュメントを読むことを強くお勧めします。

<div class="content-list" markdown="1">

- [リクエストライフサイクル](lifecycle.md)
- [設定](configuration.md)
- [ディレクトリ構造](structure.md)
- [フロントエンド](frontend.md)
- [サービスコンテナ](container.md)
- [ファサード](facades.md)

</div>

Laravelをどのように使うかによっても、次のステップは変わってきます。Laravelにはさまざまな使い方がありますが、以下ではフレームワークの主な2つの使用例を紹介します。

> NOTE:  
> Laravel初心者ですか？[Laravel Bootcamp](https://bootcamp.laravel.com)で、フレームワークのハンズオンツアーを体験しながら、初めてのLaravelアプリケーションを構築する方法を学ぶことができます。

<a name="laravel-the-fullstack-framework"></a>
### Laravelをフルスタックフレームワークとして使用する

Laravelはフルスタックフレームワークとして機能します。「フルスタック」フレームワークとは、Laravelを使用してアプリケーションへのリクエストをルーティングし、[Bladeテンプレート](blade.md)または[Inertia](https://inertiajs.com)のようなシングルページアプリケーションのハイブリッド技術を介してフロントエンドをレンダリングすることを意味します。これはLaravelを使用する最も一般的な方法であり、私たちの意見では、Laravelを使用する最も生産的な方法です。

この方法でLaravelを使用する場合、[フロントエンド開発](frontend.md)、[ルーティング](routing.md)、[ビュー](views.md)、または[Eloquent ORM](eloquent.md)に関するドキュメントをチェックしてみてください。さらに、[Livewire](https://livewire.laravel.com)や[Inertia](https://inertiajs.com)のようなコミュニティパッケージに興味があるかもしれません。これらのパッケージを使用すると、Laravelをフルスタックフレームワークとして使用しながら、シングルページJavaScriptアプリケーションが提供する多くのUIの利点を享受できます。

Laravelをフルスタックフレームワークとして使用する場合、アプリケーションのCSSとJavaScriptを[Vite](vite.md)を使用してコンパイルする方法を学ぶことも強くお勧めします。

> NOTE:  
> アプリケーションの構築を早めに開始したい場合は、公式の[アプリケーションスターターキット](starter-kits.md)のいずれかをチェックしてください。

<a name="laravel-the-api-backend"></a>
### LaravelをAPIバックエンドとして使用する

Laravelは、JavaScriptのシングルページアプリケーションやモバイルアプリケーションのAPIバックエンドとしても機能します。例えば、Laravelを[Next.js](https://nextjs.org)アプリケーションのAPIバックエンドとして使用することができます。このコンテキストでは、Laravelを使用してアプリケーションの[認証](sanctum.md)とデータの保存/取得を提供し、キューやメール、通知などのLaravelの強力なサービスを活用することができます。

この方法でLaravelを使用する場合、[ルーティング](routing.md)、[Laravel Sanctum](sanctum.md)、および[Eloquent ORM](eloquent.md)に関するドキュメントをチェックしてみてください。

> NOTE:  
> LaravelバックエンドとNext.jsフロントエンドのスキャフォールディングを早めに開始したいですか？Laravel Breezeは[APIスタック](starter-kits.md#breeze-and-next)と[Next.jsフロントエンド実装](https://github.com/laravel/breeze-next)を提供しており、数分で開始できます。

