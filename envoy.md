# Laravel Envoy

- [はじめに](#introduction)
- [インストール](#installation)
- [タスクの記述](#writing-tasks)
    - [タスクの定義](#defining-tasks)
    - [複数のサーバー](#multiple-servers)
    - [セットアップ](#setup)
    - [変数](#variables)
    - [ストーリー](#stories)
    - [フック](#completion-hooks)
- [タスクの実行](#running-tasks)
    - [タスク実行の確認](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## はじめに

[Laravel Envoy](https://github.com/laravel/envoy) は、リモートサーバーで実行する一般的なタスクを実行するためのツールです。[Blade](blade.md) スタイルの構文を使用して、デプロイ、Artisan コマンドなどのタスクを簡単にセットアップできます。現在、Envoy は Mac および Linux オペレーティングシステムのみをサポートしています。ただし、[WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) を使用して Windows サポートを実現できます。

<a name="installation"></a>
## インストール

まず、Composer パッケージマネージャーを使用して Envoy をプロジェクトにインストールします。

```shell
composer require laravel/envoy --dev
```

Envoy がインストールされると、アプリケーションの `vendor/bin` ディレクトリに Envoy バイナリが利用可能になります。

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## タスクの記述

<a name="defining-tasks"></a>
### タスクの定義

タスクは Envoy の基本的な構成要素です。タスクは、タスクが呼び出されたときにリモートサーバーで実行されるべきシェルコマンドを定義します。例えば、アプリケーションのすべてのキューワーカーサーバーで `php artisan queue:restart` コマンドを実行するタスクを定義することができます。

すべての Envoy タスクは、アプリケーションのルートにある `Envoy.blade.php` ファイルで定義する必要があります。以下は、開始するための例です。

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

ご覧のように、ファイルの先頭に `@servers` の配列が定義されており、タスク宣言の `on` オプションを介してこれらのサーバーを参照できます。`@servers` 宣言は常に1行で記述する必要があります。`@task` 宣言内には、タスクが呼び出されたときにサーバーで実行されるべきシェルコマンドを配置します。

<a name="local-tasks"></a>
#### ローカルタスク

サーバーの IP アドレスを `127.0.0.1` と指定することで、スクリプトをローカルコンピューターで実行するよう強制できます。

```blade
@servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### Envoy タスクのインポート

`@import` ディレクティブを使用して、他の Envoy ファイルをインポートし、それらのストーリーとタスクを自分のものに追加できます。ファイルがインポートされた後、それらに含まれるタスクを自分の Envoy ファイルで定義されたかのように実行できます。

```blade
@import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### 複数のサーバー

Envoy では、複数のサーバーでタスクを簡単に実行できます。まず、追加のサーバーを `@servers` 宣言に追加します。各サーバーには一意の名前を割り当てる必要があります。追加のサーバーを定義したら、タスクの `on` 配列に各サーバーをリストアップできます。

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="parallel-execution"></a>
#### 並列実行

デフォルトでは、タスクは各サーバーで順次実行されます。つまり、最初のサーバーでタスクが完了するまで、次のサーバーで実行されません。複数のサーバーでタスクを並列に実行したい場合は、タスク宣言に `parallel` オプションを追加します。

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

<a name="setup"></a>
### セットアップ

Envoy タスクを実行する前に、任意の PHP コードを実行する必要がある場合があります。`@setup` ディレクティブを使用して、タスクの前に実行される PHP コードのブロックを定義できます。

```php
@setup
    $now = new DateTime;
@endsetup
```

タスクが実行される前に他の PHP ファイルを要求する必要がある場合は、`Envoy.blade.php` ファイルの先頭で `@include` ディレクティブを使用できます。

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

<a name="variables"></a>
### 変数

必要に応じて、コマンドラインで Envoy を呼び出すときに引数を Envoy タスクに渡すことができます。

```shell
php vendor/bin/envoy run deploy --branch=master
```

オプションには、Blade の "echo" 構文を使用してタスク内でアクセスできます。また、タスク内で Blade の `if` 文やループを定義することもできます。例えば、`git pull` コマンドを実行する前に `$branch` 変数の存在を確認しましょう。

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### ストーリー

ストーリーは、一連のタスクを1つの便利な名前でグループ化します。例えば、`deploy` ストーリーは `update-code` と `install-dependencies` タスクを実行するために、タスク名を定義内にリストアップします。

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

ストーリーが書かれたら、タスクを呼び出すのと同じ方法で呼び出すことができます。

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### フック

タスクとストーリーが実行されると、いくつかのフックが実行されます。Envoy がサポートするフックの種類は `@before`、`@after`、`@error`、`@success`、`@finished` です。これらのフック内のすべてのコードは PHP として解釈され、ローカルで実行されます。タスクが対話するリモートサーバーでは実行されません。

これらのフックは、Envoy スクリプトに表示される順序で実行されます。

<a name="hook-before"></a>
#### `@before`

各タスクの実行前に、Envoy スクリプトに登録されたすべての `@before` フックが実行されます。`@before` フックは、実行されるタスクの名前を受け取ります。

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

各タスクの実行後に、Envoy スクリプトに登録されたすべての `@after` フックが実行されます。`@after` フックは、実行されたタスクの名前を受け取ります。

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

すべてのタスクの失敗（終了ステータスコードが `0` より大きい）後に、Envoy スクリプトに登録されたすべての `@error` フックが実行されます。`@error` フックは、実行されたタスクの名前を受け取ります。

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### `@success`

すべてのタスクがエラーなく実行された場合、Envoy スクリプトに登録されたすべての `@success` フックが実行されます。

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### `@finished`

すべてのタスクが実行された後（終了ステータスに関係なく）、すべての `@finished` フックが実行されます。`@finished` フックは、完了したタスクのステータスコードを受け取ります。これは `null` または `0` 以上の整数です。

```blade
@finished
    if ($exitCode > 0) {
        // タスクのいずれかでエラーが発生しました...
    }
@endfinished
```

<a name="running-tasks"></a>
## タスクの実行

アプリケーションの `Envoy.blade.php` ファイルで定義されたタスクまたはストーリーを実行するには、Envoy の `run` コマンドを実行し、実行したいタスクまたはストーリーの名前を渡します。Envoy はタスクを実行し、タスクの実行中にリモートサーバーからの出力を表示します。

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### タスク実行の確認

サーバーで特定のタスクを実行する前に確認を求めるプロンプトを表示する場合は、タスク宣言に `confirm` ディレクティブを追加する必要があります。このオプションは、特に破壊的な操作に役立ちます。

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## 通知

<a name="slack"></a>
### Slack

Envoy は、各タスクの実行後に [Slack](https://slack.com) に通知を送信することをサポートしています。`@slack` ディレクティブは、Slack の Webhook URL とチャンネル / ユーザー名を受け取ります。Webhook URL は、Slack のコントロールパネルで "Incoming WebHooks" 統合を作成することで取得できます。

`@slack` ディレクティブには、Webhook URL 全体を最初の引数として渡す必要があります。`@slack` ディレクティブに渡す2番目の引数は、チャンネル名（`#channel`）またはユーザー名（`@user`）です。

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

デフォルトでは、Envoy 通知は実行されたタスクを説明するメッセージを通知チャンネルに送信します。ただし、`@slack` ディレクティブに3番目の引数を渡すことで、このメッセージをカスタムメッセージで上書きできます。

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy は、各タスクの実行後に [Discord](https://discord.com) に通知を送信することもサポートしています。`@discord` ディレクティブは、Discord のフック URL とメッセージを受け取ります。サーバー設定で「Webhook」を作成し、Webhook が投稿するチャンネルを選択することで、Webhook URL を取得できます。Webhook URL 全体を `@discord` ディレクティブに渡す必要があります。

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### Telegram

Envoy は、各タスクの実行後に [Telegram](https://telegram.org) に通知を送信することもサポートしています。`@telegram` ディレクティブは、Telegram の Bot ID と Chat ID を受け取ります。[BotFather](https://t.me/botfather) を使用して新しいボットを作成することで、Bot ID を取得できます。有効な Chat ID は、[@username_to_id_bot](https://t.me/username_to_id_bot) を使用して取得できます。Bot ID と Chat ID 全体を `@telegram` ディレクティブに渡す必要があります。

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy は、各タスクの実行後に [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) に通知を送信することもサポートしています。`@microsoftTeams` ディレクティブは、Teams Webhook（必須）、メッセージ、テーマカラー（成功、情報、警告、エラー）、およびオプションの配列を受け取ります。Teams Webhook は、新しい [incoming webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) を作成することで取得できます。Teams API には、タイトル、サマリー、セクションなど、メッセージボックスをカスタマイズするための他の多くの属性があります。詳細については、[Microsoft Teams のドキュメント](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message) を参照してください。Webhook URL 全体を `@microsoftTeams` ディレクティブに渡す必要があります。

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
