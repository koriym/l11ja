# ディレクトリ構造

- [はじめに](#introduction)
- [ルートディレクトリ](#the-root-directory)
    - [`app`ディレクトリ](#the-root-app-directory)
    - [`bootstrap`ディレクトリ](#the-bootstrap-directory)
    - [`config`ディレクトリ](#the-config-directory)
    - [`database`ディレクトリ](#the-database-directory)
    - [`public`ディレクトリ](#the-public-directory)
    - [`resources`ディレクトリ](#the-resources-directory)
    - [`routes`ディレクトリ](#the-routes-directory)
    - [`storage`ディレクトリ](#the-storage-directory)
    - [`tests`ディレクトリ](#the-tests-directory)
    - [`vendor`ディレクトリ](#the-vendor-directory)
- [Appディレクトリ](#the-app-directory)
    - [`Broadcasting`ディレクトリ](#the-broadcasting-directory)
    - [`Console`ディレクトリ](#the-console-directory)
    - [`Events`ディレクトリ](#the-events-directory)
    - [`Exceptions`ディレクトリ](#the-exceptions-directory)
    - [`Http`ディレクトリ](#the-http-directory)
    - [`Jobs`ディレクトリ](#the-jobs-directory)
    - [`Listeners`ディレクトリ](#the-listeners-directory)
    - [`Mail`ディレクトリ](#the-mail-directory)
    - [`Models`ディレクトリ](#the-models-directory)
    - [`Notifications`ディレクトリ](#the-notifications-directory)
    - [`Policies`ディレクトリ](#the-policies-directory)
    - [`Providers`ディレクトリ](#the-providers-directory)
    - [`Rules`ディレクトリ](#the-rules-directory)

<a name="introduction"></a>
## はじめに

Laravelのデフォルトアプリケーション構造は、大規模なアプリケーションと小規模なアプリケーションの両方にとって素晴らしい出発点を提供することを目的としています。ただし、アプリケーションをどのように整理するかは自由です。Laravelは、Composerがクラスを自動ロードできる限り、どこにクラスが配置されているかについてほとんど制限を課しません。

> NOTE:  
> Laravelを初めて使う方は、[Laravel Bootcamp](https://bootcamp.laravel.com)をチェックしてください。フレームワークのハンズオンツアーを通じて、最初のLaravelアプリケーションを構築する方法を学ぶことができます。

<a name="the-root-directory"></a>
## ルートディレクトリ

<a name="the-root-app-directory"></a>
#### `app`ディレクトリ

`app`ディレクトリには、アプリケーションのコアコードが含まれています。このディレクトリについては後ほど詳しく説明しますが、アプリケーションのほとんどのクラスがこのディレクトリに配置されます。

<a name="the-bootstrap-directory"></a>
#### `bootstrap`ディレクトリ

`bootstrap`ディレクトリには、フレームワークをブートストラップする`app.php`ファイルが含まれています。また、このディレクトリには、ルートやサービスのキャッシュファイルなど、パフォーマンス最適化のためにフレームワークが生成するファイルを含む`cache`ディレクトリもあります。

<a name="the-config-directory"></a>
#### `config`ディレクトリ

`config`ディレクトリには、アプリケーションのすべての設定ファイルが含まれています。これらのファイルをすべて読んで、利用可能なオプションをすべて理解することをお勧めします。

<a name="the-database-directory"></a>
#### `database`ディレクトリ

`database`ディレクトリには、データベースのマイグレーション、モデルファクトリ、シードが含まれています。必要に応じて、SQLiteデータベースをこのディレクトリに保存することもできます。

<a name="the-public-directory"></a>
#### `public`ディレクトリ

`public`ディレクトリには、すべてのリクエストがアプリケーションに入るエントリーポイントである`index.php`ファイルが含まれています。また、このディレクトリには画像、JavaScript、CSSなどのアセットも保存されます。

<a name="the-resources-directory"></a>
#### `resources`ディレクトリ

`resources`ディレクトリには、[ビュー](views.md)や、未コンパイルのアセット（CSSやJavaScriptなど）が含まれています。

<a name="the-routes-directory"></a>
#### `routes`ディレクトリ

`routes`ディレクトリには、アプリケーションのすべてのルート定義が含まれています。デフォルトでは、`web.php`と`console.php`の2つのルートファイルがLaravelに含まれています。

`web.php`ファイルには、Laravelが`web`ミドルウェアグループに配置するルートが含まれています。このグループは、セッション状態、CSRF保護、およびクッキーの暗号化を提供します。アプリケーションがステートレスなRESTful APIを提供しない場合、ほとんどのルートは`web.php`ファイルで定義されるでしょう。

`console.php`ファイルは、すべてのクロージャーベースのコンソールコマンドを定義する場所です。各クロージャーはコマンドインスタンスにバインドされており、各コマンドのIOメソッドと簡単にやり取りできるようになっています。このファイルはHTTPルートを定義しませんが、アプリケーションへのコンソールベースのエントリーポイント（ルート）を定義します。また、`console.php`ファイルで[スケジュール](scheduling.md)タスクを定義することもできます。

必要に応じて、`install:api`および`install:broadcasting` Artisanコマンドを介して、APIルート（`api.php`）とブロードキャストチャンネル（`channels.php`）の追加のルートファイルをインストールできます。

`api.php`ファイルには、ステートレスであることを意図したルートが含まれています。したがって、これらのルートを通じてアプリケーションに入るリクエストは、[トークン](sanctum.md)を介して認証され、セッション状態にアクセスできないことを意図しています。

`channels.php`ファイルは、アプリケーションがサポートするすべての[イベントブロードキャスト](broadcasting.md)チャンネルを登録する場所です。

<a name="the-storage-directory"></a>
#### `storage`ディレクトリ

`storage`ディレクトリには、ログ、コンパイルされたBladeテンプレート、ファイルベースのセッション、ファイルキャッシュ、およびフレームワークによって生成されたその他のファイルが含まれています。このディレクトリは、`app`、`framework`、および`logs`ディレクトリに分けられています。`app`ディレクトリは、アプリケーションによって生成されたファイルを保存するために使用できます。`framework`ディレクトリは、フレームワークによって生成されたファイルとキャッシュを保存するために使用されます。最後に、`logs`ディレクトリには、アプリケーションのログファイルが含まれています。

`storage/app/public`ディレクトリは、プロフィールアバターなど、ユーザーが生成したファイルを保存するために使用できます。これらのファイルは公開されるべきです。このディレクトリへのシンボリックリンクを`public/storage`に作成する必要があります。このリンクは、`php artisan storage:link` Artisanコマンドを使用して作成できます。

<a name="the-tests-directory"></a>
#### `tests`ディレクトリ

`tests`ディレクトリには、自動テストが含まれています。[Pest](https://pestphp.com)または[PHPUnit](https://phpunit.de/)のユニットテストと機能テストの例がデフォルトで提供されています。各テストクラスには、`Test`という単語を接尾辞として付ける必要があります。テストは、`/vendor/bin/pest`または`/vendor/bin/phpunit`コマンドを使用して実行できます。また、テスト結果をより詳細で美しい表現で表示したい場合は、`php artisan test` Artisanコマンドを使用してテストを実行できます。

<a name="the-vendor-directory"></a>
#### `vendor`ディレクトリ

`vendor`ディレクトリには、[Composer](https://getcomposer.org)の依存関係が含まれています。

<a name="the-app-directory"></a>
## Appディレクトリ

アプリケーションの大部分は、`app`ディレクトリに配置されます。デフォルトでは、このディレクトリは`App`名前空間の下にあり、Composerによって[PSR-4自動ロード規格](https://www.php-fig.org/psr/psr-4)を使用して自動ロードされます。

デフォルトでは、`app`ディレクトリには`Http`、`Models`、および`Providers`ディレクトリが含まれています。ただし、時間が経つにつれて、make Artisanコマンドを使用してクラスを生成すると、`app`ディレクトリ内にさまざまなディレクトリが生成されます。たとえば、`app/Console`ディレクトリは、`make:command` Artisanコマンドを実行してコマンドクラスを生成するまで存在しません。

`Console`ディレクトリと`Http`ディレクトリについては、後ほどそれぞれのセクションで詳しく説明しますが、`Console`ディレクトリと`Http`ディレクトリは、アプリケーションのコアに対するAPIを提供すると考えてください。HTTPプロトコルとCLIは、どちらもアプリケーションとやり取りするためのメカニズムですが、実際のアプリケーションロジックは含まれていません。言い換えれば、これらはアプリケーションにコマンドを発行する2つの方法です。`Console`ディレクトリには、すべてのArtisanコマンドが含まれていますが、`Http`ディレクトリには、コントローラー、ミドルウェア、およびリクエストが含まれています。

> NOTE:  
> `app`ディレクトリ内の多くのクラスは、Artisanコマンドを介して生成できます。利用可能なコマンドを確認するには、ターミナルで`php artisan list make`コマンドを実行してください。

<a name="the-broadcasting-directory"></a>
#### `Broadcasting`ディレクトリ

`Broadcasting`ディレクトリには、アプリケーションのすべてのブロードキャストチャンネルクラスが含まれています。これらのクラスは、`make:channel`コマンドを使用して生成されます。このディレクトリはデフォルトでは存在しませんが、最初のチャンネルを作成するときに作成されます。チャンネルの詳細については、[イベントブロードキャスト](broadcasting.md)のドキュメントを確認してください。

<a name="the-console-directory"></a>
#### `Console`ディレクトリ

`Console`ディレクトリには、アプリケーションのすべてのカスタムArtisanコマンドが含まれています。これらのコマンドは、`make:command`コマンドを使用して生成できます。

<a name="the-events-directory"></a>
#### `Events`ディレクトリ

このディレクトリはデフォルトでは存在しませんが、`event:generate`および`make:event` Artisanコマンドによって作成されます。`Events`ディレクトリには、[イベントクラス](events.md)が含まれています。イベントは、特定のアクションが発生したことをアプリケーションの他の部分に通知するために使用され、柔軟性とデカップリングを提供します。

<a name="the-exceptions-directory"></a>
#### `Exceptions`ディレクトリ

`Exceptions`ディレクトリには、アプリケーションのすべてのカスタム例外が含まれています。これらの例外は、`make:exception`コマンドを使用して生成できます。

<a name="the-http-directory"></a>
#### `Http`ディレクトリ

`Http`ディレクトリには、コントローラー、ミドルウェア、およびフォームリクエストが含まれています。アプリケーションに入るほとんどのリクエストを処理するロジックは、このディレクトリに配置されます。

<a name="the-jobs-directory"></a>
#### `Jobs`ディレクトリ

`Jobs`ディレクトリには、アプリケーションのすべてのジョブが含まれています。ジョブは、アプリケーションのキューに入れられるか、現在のリクエストサイクル内で同期的に実行される可能性があります。デフォルトでは、このディレクトリは存在しませんが、`make:job` Artisanコマンドを使用して最初のジョブを生成すると作成されます。

<a name="the-listeners-directory"></a>
#### `Listeners`ディレクトリ

`Listeners`ディレクトリには、イベントを処理するクラスが含まれています。リスナーは、イベントが発生したときにロジックを実行するように登録されます。デフォルトでは、このディレクトリは存在しませんが、`event:generate`または`make:listener` Artisanコマンドを使用して最初のリスナーを生成すると作成されます。

<a name="the-mail-directory"></a>
#### `Mail`ディレクトリ

`Mail`ディレクトリには、アプリケーションが送信するメールを表すクラスが含まれています。メールは、`make:mail` Artisanコマンドを使用して生成できます。

<a name="the-models-directory"></a>
#### `Models`ディレクトリ

`Models`ディレクトリには、すべてのEloquentモデルクラスが含まれています。Eloquent ORMは、データベースとやり取りするための美しくシンプルなActiveRecord実装を提供します。デフォルトでは、このディレクトリは存在しませんが、`make:model` Artisanコマンドを使用して最初のモデルを生成すると作成されます。

<a name="the-notifications-directory"></a>
#### `Notifications`ディレクトリ

`Notifications`ディレクトリには、アプリケーション内で発生するすべての「トランザクション」通知が含まれています。通知は、メール、Slack、SMSなどのチャネルを介してユーザーに情報を送信する方法を表します。デフォルトでは、このディレクトリは存在しませんが、`make:notification` Artisanコマンドを使用して最初の通知を生成すると作成されます。

<a name="the-policies-directory"></a>
#### `Policies`ディレクトリ

`Policies`ディレクトリには、アプリケーションの認可ポリシークラスが含まれています。ポリシーは、ユーザーが特定のリソースに対してアクションを実行できるかどうかを判断するために使用されます。デフォルトでは、このディレクトリは存在しませんが、`make:policy` Artisanコマンドを使用して最初のポリシーを生成すると作成されます。

<a name="the-providers-directory"></a>
#### `Providers`ディレクトリ

`Providers`ディレクトリには、アプリケーションのすべてのサービスプロバイダが含まれています。サービスプロバイダは、アプリケーションの起動プロセス中にサービスをブートストラップし、コンテナへのサービスの登録、イベントのリッスン、その他のタスクを実行します。

<a name="the-rules-directory"></a>
#### `Rules`ディレクトリ

`Rules`ディレクトリには、アプリケーションのカスタムバリデーションルールオブジェクトが含まれています。ルールは、複雑なバリデーションロジックをシンプルなオブジェクトにカプセル化するために使用されます。デフォルトでは、このディレクトリは存在しませんが、`make:rule` Artisanコマンドを使用して最初のルールを生成すると作成されます。

このディレクトリはデフォルトでは存在しませんが、`make:job` Artisanコマンドを実行すると作成されます。`Jobs`ディレクトリには、アプリケーションの[キュー可能なジョブ](queues.md)が格納されます。ジョブはアプリケーションによってキューに入れられるか、現在のリクエストのライフサイクル内で同期的に実行されます。現在のリクエスト中に同期的に実行されるジョブは、[コマンドパターン](https://en.wikipedia.org/wiki/Command_pattern)の実装であるため、「コマンド」と呼ばれることもあります。

<a name="the-listeners-directory"></a>
#### Listenersディレクトリ

このディレクトリはデフォルトでは存在しませんが、`event:generate`または`make:listener` Artisanコマンドを実行すると作成されます。`Listeners`ディレクトリには、[イベント](events.md)を処理するクラスが含まれます。イベントリスナーはイベントインスタンスを受け取り、イベントが発生したことに対応してロジックを実行します。例えば、`UserRegistered`イベントは`SendWelcomeEmail`リスナーによって処理されるかもしれません。

<a name="the-mail-directory"></a>
#### Mailディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:mail` Artisanコマンドを実行すると作成されます。`Mail`ディレクトリには、アプリケーションによって送信される[メールを表すクラス](mail.md)がすべて含まれます。メールオブジェクトを使用すると、メールの構築に関するすべてのロジックを`Mail::send`メソッドを使用して送信できる単一のシンプルなクラスにカプセル化できます。

<a name="the-models-directory"></a>
#### Modelsディレクトリ

`Models`ディレクトリには、すべての[Eloquentモデルクラス](eloquent.md)が含まれます。Laravelに含まれるEloquent ORMは、データベースを操作するための美しくシンプルなActiveRecord実装を提供します。各データベーステーブルには、そのテーブルと対話するために使用される対応する「モデル」があります。モデルを使用すると、テーブル内のデータをクエリしたり、新しいレコードをテーブルに挿入したりできます。

<a name="the-notifications-directory"></a>
#### Notificationsディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:notification` Artisanコマンドを実行すると作成されます。`Notifications`ディレクトリには、アプリケーション内で発生するイベントに関するシンプルな通知など、アプリケーションによって送信されるすべての「トランザクション」[通知](notifications.md)が含まれます。Laravelの通知機能は、メール、Slack、SMS、またはデータベースに保存されるなど、さまざまなドライバーを介して通知を送信することを抽象化します。

<a name="the-policies-directory"></a>
#### Policiesディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:policy` Artisanコマンドを実行すると作成されます。`Policies`ディレクトリには、アプリケーションの[認可ポリシークラス](authorization.md)が含まれます。ポリシーは、ユーザーがリソースに対して特定のアクションを実行できるかどうかを判断するために使用されます。

<a name="the-providers-directory"></a>
#### Providersディレクトリ

`Providers`ディレクトリには、アプリケーションのすべての[サービスプロバイダ](providers.md)が含まれます。サービスプロバイダは、サービスコンテナにサービスをバインドしたり、イベントを登録したり、その他のタスクを実行して、アプリケーションを受信リクエストに備えるためにアプリケーションをブートストラップします。

新しいLaravelアプリケーションでは、このディレクトリにはすでに`AppServiceProvider`が含まれています。必要に応じて、独自のプロバイダをこのディレクトリに自由に追加できます。

<a name="the-rules-directory"></a>
#### Rulesディレクトリ

このディレクトリはデフォルトでは存在しませんが、`make:rule` Artisanコマンドを実行すると作成されます。`Rules`ディレクトリには、アプリケーションのカスタムバリデーションルールオブジェクトが含まれます。ルールは、複雑なバリデーションロジックをシンプルなオブジェクトにカプセル化するために使用されます。詳細については、[バリデーションドキュメント](validation.md)を確認してください。

