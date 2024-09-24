# ロギング

- [はじめに](#introduction)
- [設定](#configuration)
    - [利用可能なチャンネルドライバ](#available-channel-drivers)
    - [チャンネルの前提条件](#channel-prerequisites)
    - [非推奨警告のロギング](#logging-deprecation-warnings)
- [ログスタックの構築](#building-log-stacks)
- [ログメッセージの書き込み](#writing-log-messages)
    - [コンテキスト情報](#contextual-information)
    - [特定のチャンネルへの書き込み](#writing-to-specific-channels)
- [Monologチャンネルのカスタマイズ](#monolog-channel-customization)
    - [チャンネル用のMonologのカスタマイズ](#customizing-monolog-for-channels)
    - [Monologハンドラチャンネルの作成](#creating-monolog-handler-channels)
    - [ファクトリを介したカスタムチャンネルの作成](#creating-custom-channels-via-factories)
- [Pailを使用したログメッセージの追跡](#tailing-log-messages-using-pail)
    - [インストール](#pail-installation)
    - [使用方法](#pail-usage)
    - [ログのフィルタリング](#pail-filtering-logs)

<a name="introduction"></a>
## はじめに

アプリケーション内で何が起こっているかを詳しく知るために、Laravelはファイル、システムエラーログ、さらにはSlackにメッセージをログとして記録し、チーム全体に通知することができる堅牢なロギングサービスを提供しています。

Laravelのロギングは「チャンネル」に基づいています。各チャンネルは、ログ情報を書き込む特定の方法を表します。例えば、`single`チャンネルは単一のログファイルにログを書き込み、`slack`チャンネルはログメッセージをSlackに送信します。ログメッセージは、その重大度に基づいて複数のチャンネルに書き込むことができます。

内部的には、Laravelは[Monolog](https://github.com/Seldaek/monolog)ライブラリを利用しており、さまざまな強力なログハンドラをサポートしています。Laravelはこれらのハンドラを簡単に設定できるようにし、アプリケーションのログ処理をカスタマイズするためにそれらを組み合わせることができます。

<a name="configuration"></a>
## 設定

アプリケーションのロギング動作を制御するすべての設定オプションは、`config/logging.php`設定ファイルに収められています。このファイルでは、アプリケーションのログチャンネルを設定できるため、利用可能な各チャンネルとそのオプションを確認してください。以下では、いくつかの一般的なオプションを紹介します。

デフォルトでは、Laravelはログメッセージを記録する際に`stack`チャンネルを使用します。`stack`チャンネルは、複数のログチャンネルを単一のチャンネルに集約するために使用されます。スタックの構築について詳しくは、[以下のドキュメント](#building-log-stacks)を参照してください。

<a name="available-channel-drivers"></a>
### 利用可能なチャンネルドライバ

各ログチャンネルは「ドライバ」によって動作します。ドライバは、ログメッセージが実際にどのように記録されるかを決定します。以下のログチャンネルドライバは、すべてのLaravelアプリケーションで利用可能です。これらのドライバのほとんどは、アプリケーションの`config/logging.php`設定ファイルに既に存在しているため、このファイルを確認して内容に慣れることをお勧めします。

<div class="overflow-auto" markdown=1>

| 名前         | 説明                                                                 |
| ------------ | -------------------------------------------------------------------- |
| `custom`     | 指定されたファクトリを呼び出してチャンネルを作成するドライバ。         |
| `daily`      | 日次でローテーションする`RotatingFileHandler`ベースのMonologドライバ。 |
| `errorlog`   | `ErrorLogHandler`ベースのMonologドライバ。                           |
| `monolog`    | サポートされている任意のMonologハンドラを使用できるMonologファクトリドライバ。 |
| `papertrail` | `SyslogUdpHandler`ベースのMonologドライバ。                           |
| `single`     | 単一のファイルまたはパスベースのロガーチャンネル（`StreamHandler`）。 |
| `slack`      | `SlackWebhookHandler`ベースのMonologドライバ。                        |
| `stack`      | 「マルチチャンネル」チャンネルを作成するためのラッパー。               |
| `syslog`     | `SyslogHandler`ベースのMonologドライバ。                              |

</div>

> NOTE:  
> [高度なチャンネルカスタマイズ](#monolog-channel-customization)のドキュメントを参照して、`monolog`および`custom`ドライバの詳細を学んでください。

<a name="configuring-the-channel-name"></a>
#### チャンネル名の設定

デフォルトでは、Monologは現在の環境に一致する「チャンネル名」（例：`production`または`local`）でインスタンス化されます。この値を変更するには、チャンネルの設定に`name`オプションを追加できます。

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="channel-prerequisites"></a>
### チャンネルの前提条件

<a name="configuring-the-single-and-daily-channels"></a>
#### SingleおよびDailyチャンネルの設定

`single`および`daily`チャンネルには、3つのオプションの設定オプションがあります：`bubble`、`permission`、および`locking`。

<div class="overflow-auto" markdown=1>

| 名前         | 説明                                                                   | デフォルト |
| ------------ | ----------------------------------------------------------------------------- | ------- |
| `bubble`     | メッセージが処理された後、他のチャンネルにバブルアップするかどうかを示します。 | `true`  |
| `locking`    | ログファイルに書き込む前にロックを試みます。                            | `false` |
| `permission` | ログファイルのパーミッション。                                                   | `0644`  |

</div>

さらに、`daily`チャンネルの保持ポリシーは、`LOG_DAILY_DAYS`環境変数または`days`設定オプションを介して設定できます。

<div class="overflow-auto" markdown=1>

| 名前   | 説明                                                 | デフォルト |
| ------ | ----------------------------------------------------------- | ------- |
| `days` | 日次ログファイルを保持する日数。 | `7`     |

</div>

<a name="configuring-the-papertrail-channel"></a>
#### Papertrailチャンネルの設定

`papertrail`チャンネルには、`host`および`port`設定オプションが必要です。これらは、`PAPERTRAIL_URL`および`PAPERTRAIL_PORT`環境変数を介して定義できます。これらの値は、[Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app)から取得できます。

<a name="configuring-the-slack-channel"></a>
#### Slackチャンネルの設定

`slack`チャンネルには、`url`設定オプションが必要です。この値は、`LOG_SLACK_WEBHOOK_URL`環境変数を介して定義できます。このURLは、Slackチーム用に設定した[インカミングウェブフック](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)のURLと一致する必要があります。

デフォルトでは、Slackは`critical`レベル以上のログのみを受信します。ただし、これは`LOG_LEVEL`環境変数を使用するか、Slackログチャンネルの設定配列内の`level`設定オプションを変更することで調整できます。

<a name="logging-deprecation-warnings"></a>
### 非推奨警告のロギング

PHP、Laravel、およびその他のライブラリは、その機能の一部が非推奨になり、将来のバージョンで削除されることをユーザーに通知することがよくあります。これらの非推奨警告をログに記録したい場合は、`LOG_DEPRECATIONS_CHANNEL`環境変数を使用して、またはアプリケーションの`config/logging.php`設定ファイル内で、優先する`deprecations`ログチャンネルを指定できます。

    'deprecations' => [
        'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
        'trace' => env('LOG_DEPRECATIONS_TRACE', false),
    ],

    'channels' => [
        // ...
    ]

または、`deprecations`という名前のログチャンネルを定義することもできます。この名前のログチャンネルが存在する場合、非推奨警告のログに常に使用されます。

    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],

<a name="building-log-stacks"></a>
## ログスタックの構築

前述のように、`stack`ドライバを使用すると、複数のチャンネルを単一のログチャンネルに結合できます。ログスタックの使用方法を説明するために、本番アプリケーションで見られる設定例を見てみましょう。

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'], // [tl! add]
        'ignore_exceptions' => false,
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```

この設定を詳しく見てみましょう。まず、`stack`チャンネルが`channels`オプションを介して他の2つのチャンネル（`syslog`と`slack`）を集約していることに注目してください。したがって、ログメッセージを記録する際に、これらの両方のチャンネルがメッセージをログに記録する機会を持ちます。ただし、以下で見るように、これらのチャンネルが実際にメッセージをログに記録するかどうかは、メッセージの重大度/「レベル」によって決まる場合があります。

<a name="log-levels"></a>
#### ログレベル

上記の例の`syslog`および`slack`チャンネル設定にある`level`設定オプションに注意してください。このオプションは、メッセージがチャンネルによってログに記録されるために必要な最小「レベル」を決定します。Laravelのロギングサービスを提供するMonologは、[RFC 5424仕様](https://tools.ietf.org/html/rfc5424)で定義されたすべてのログレベルを提供します。重大度の降順で、これらのログレベルは次のとおりです：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info**、および**debug**。

したがって、`debug`メソッドを使用してメッセージをログに記録すると想像してください：

    Log::debug('An informational message.');

与えられた設定に基づき、`syslog`チャンネルはメッセージをシステムログに書き込みます。しかし、エラーメッセージが`critical`以上でないため、Slackには送信されません。しかし、`emergency`メッセージをログに記録する場合、`emergency`レベルは両方のチャンネルの最小レベルのしきい値を超えているため、システムログとSlackの両方に送信されます：

    Log::emergency('The system is down!');

<a name="writing-log-messages"></a>
## ログメッセージの書き込み

`Log` [ファサード](facades.md)を使用して、ログに情報を書き込むことができます。前述のように、ロガーは[RFC 5424仕様](https://tools.ietf.org/html/rfc5424)で定義された8つのロギングレベルを提供します：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info**、**debug**：

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

これらのメソッドのいずれかを呼び出して、対応するレベルのメッセージをログに記録できます。デフォルトでは、メッセージは`logging`設定ファイルで設定されたデフォルトのログチャンネルに書き込まれます：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 指定されたユーザーのプロフィールを表示します。
         */
        public function show(string $id): View
        {
            Log::info('Showing the user profile for user: {id}', ['id' => $id]);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
### コンテキスト情報

コンテキストデータの配列をログメソッドに渡すことができます。このコンテキストデータはフォーマットされ、ログメッセージとともに表示されます：

    use Illuminate\Support\Facades\Log;

    Log::info('User {id} failed to login.', ['id' => $user->id]);

特定のチャンネルで後続のすべてのログエントリに含まれるべきコンテキスト情報を指定したい場合があります。たとえば、アプリケーションへの各受信リクエストに関連付けられたリクエストIDをログに記録したい場合があります。これを実現するには、`Log`ファサードの`withContext`メソッドを呼び出すことができます：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;
    use Symfony\Component\HttpFoundation\Response;

    class AssignRequestId
    {
        /**
         * 受信リクエストを処理します。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            $requestId = (string) Str::uuid();

            Log::withContext([
                'request-id' => $requestId
            ]);

            $response = $next($request);

            $response->headers->set('Request-Id', $requestId);

            return $response;
        }
    }

すべてのログチャンネル間でコンテキスト情報を共有したい場合は、`Log::shareContext()`メソッドを呼び出すことができます。このメソッドは、コンテキスト情報をすべての作成されたチャンネルと、後で作成されるチャンネルに提供します：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Log;
    use Illuminate\Support\Str;
    use Symfony\Component\HttpFoundation\Response;

    class AssignRequestId
    {
        /**
         * 受信リクエストを処理します。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            $requestId = (string) Str::uuid();

            Log::shareContext([
                'request-id' => $requestId
            ]);

            // ...
        }
    }

> NOTE:  
> キューに入れられたジョブを処理する際にログコンテキストを共有する必要がある場合は、[ジョブミドルウェア](queues.md#job-middleware)を利用できます。

<a name="writing-to-specific-channels"></a>
### 特定のチャンネルへの書き込み

アプリケーションのデフォルトチャンネル以外のチャンネルにメッセージをログに記録したい場合があります。`Log`ファサードの`channel`メソッドを使用して、設定ファイルで定義された任意のチャンネルを取得し、ログに記録できます：

    use Illuminate\Support\Facades\Log;

    Log::channel('slack')->info('Something happened!');

複数のチャンネルで構成されるオンデマンドのログスタックを作成したい場合は、`stack`メソッドを使用できます：

    Log::stack(['single', 'slack'])->info('Something happened!');

<a name="on-demand-channels"></a>
#### オンデマンドチャンネル

アプリケーションの`logging`設定ファイルに設定が存在しない状態で、実行時に設定を提供してオンデマンドチャンネルを作成することも可能です。これを実現するには、設定配列を`Log`ファサードの`build`メソッドに渡します：

    use Illuminate\Support\Facades\Log;

    Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ])->info('Something happened!');

オンデマンドチャンネルをオンデマンドログスタックに含めたい場合もあります。これは、オンデマンドチャンネルインスタンスを`stack`メソッドに渡される配列に含めることで実現できます：

    use Illuminate\Support\Facades\Log;

    $channel = Log::build([
      'driver' => 'single',
      'path' => storage_path('logs/custom.log'),
    ]);

    Log::stack(['slack', $channel])->info('Something happened!');

<a name="monolog-channel-customization"></a>
## Monologチャンネルのカスタマイズ

<a name="customizing-monolog-for-channels"></a>
### チャンネルのMonologのカスタマイズ

既存のチャンネルに対してMonologの設定を完全に制御したい場合があります。たとえば、Laravelの組み込み`single`チャンネルに対してカスタムのMonolog `FormatterInterface`実装を設定したい場合です。

まず、チャンネルの設定に`tap`配列を定義します。`tap`配列には、Monologインスタンスの作成後にカスタマイズ（または「タップ」）する機会を持つクラスのリストを含める必要があります。これらのクラスを配置する標準的な場所はありませんので、アプリケーション内にこれらのクラスを含むディレクトリを自由に作成できます：

    'single' => [
        'driver' => 'single',
        'tap' => [App\Logging\CustomizeFormatter::class],
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'replace_placeholders' => true,
    ],

チャンネルの`tap`オプションを設定したら、Monologインスタンスをカスタマイズするクラスを定義する準備が整いました。このクラスには、`Illuminate\Log\Logger`インスタンスを受け取る`__invoke`メソッドが1つ必要です。`Illuminate\Log\Logger`インスタンスは、基礎となるMonologインスタンスへのすべてのメソッド呼び出しをプロキシします：

    <?php

    namespace App\Logging;

    use Illuminate\Log\Logger;
    use Monolog\Formatter\LineFormatter;

    class CustomizeFormatter
    {
        /**
         * 指定されたロガーインスタンスをカスタマイズします。
         */
        public function __invoke(Logger $logger): void
        {
            foreach ($logger->getHandlers() as $handler) {
                $handler->setFormatter(new LineFormatter(
                    '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
                ));
            }
        }
    }

> NOTE:  
> すべての「tap」クラスは[サービスコンテナ](container.md)によって解決されるため、必要なコンストラクタ依存関係は自動的に注入されます。

<a name="creating-monolog-handler-channels"></a>
### Monologハンドラチャンネルの作成

Monologにはさまざまな[利用可能なハンドラ](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler)があり、Laravelにはそれぞれに対応する組み込みチャンネルが含まれていません。特定のMonologハンドラのインスタンスであるカスタムチャンネルを作成したい場合があります。これらのチャンネルは、`monolog`ドライバを使用して簡単に作成できます。

`monolog`ドライバを使用する場合、`handler`設定オプションを使用して、どのハンドラがインスタンス化されるかを指定します。必要に応じて、ハンドラが必要とするコンストラクタパラメータは、`with`設定オプションを使用して指定できます：

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Monologフォーマッタ

`monolog`ドライバを使用する場合、Monologの`LineFormatter`がデフォルトのフォーマッタとして使用されます。ただし、`formatter`および`formatter_with`設定オプションを使用して、ハンドラに渡すフォーマッタのタイプをカスタマイズできます：

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

Monologハンドラが独自のフォーマッタを提供できる場合、`formatter`設定オプションの値を`default`に設定できます：

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="monolog-processors"></a>
#### Monologプロセッサ

Monologは、ログに記録する前にメッセージを処理することもできます。独自のプロセッサを作成するか、Monologが提供する[既存のプロセッサ](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Processor)を使用できます。

`monolog`ドライバのプロセッサをカスタマイズしたい場合は、チャンネルの設定に`processors`設定値を追加してください。

     'memory' => [
         'driver' => 'monolog',
         'handler' => Monolog\Handler\StreamHandler::class,
         'with' => [
             'stream' => 'php://stderr',
         ],
         'processors' => [
             // シンプルな構文...
             Monolog\Processor\MemoryUsageProcessor::class,

             // オプション付き...
             [
                'processor' => Monolog\Processor\PsrLogMessageProcessor::class,
                'with' => ['removeUsedContextFields' => true],
            ],
         ],
     ],

<a name="creating-custom-channels-via-factories"></a>
### ファクトリを介したカスタムチャンネルの作成

Monologのインスタンス化と設定を完全に制御できる完全にカスタムなチャンネルを定義したい場合は、`config/logging.php`設定ファイルで`custom`ドライバタイプを指定できます。設定には、Monologインスタンスを作成するために呼び出されるファクトリクラスの名前を含む`via`オプションを含める必要があります。

    'channels' => [
        'example-custom-channel' => [
            'driver' => 'custom',
            'via' => App\Logging\CreateCustomLogger::class,
        ],
    ],

`custom`ドライバチャンネルを設定したら、Monologインスタンスを作成するクラスを定義する準備が整いました。このクラスには、Monologロガーインスタンスを返す必要がある単一の`__invoke`メソッドが必要です。このメソッドは、チャンネル設定配列を唯一の引数として受け取ります。

    <?php

    namespace App\Logging;

    use Monolog\Logger;

    class CreateCustomLogger
    {
        /**
         * カスタムMonologインスタンスを作成する。
         */
        public function __invoke(array $config): Logger
        {
            return new Logger(/* ... */);
        }
    }

<a name="tailing-log-messages-using-pail"></a>
## Pailを使用したログメッセージのテイル

アプリケーションのログをリアルタイムでテイルする必要がある場合があります。例えば、問題のデバッグや、特定の種類のエラーのアプリケーションログの監視などです。

Laravel Pailは、コマンドラインから直接Laravelアプリケーションのログファイルに簡単にアクセスできるようにするパッケージです。標準の`tail`コマンドとは異なり、PailはSentryやFlareを含む任意のログドライバで動作するように設計されています。さらに、Pailは、探しているものをすばやく見つけるのに役立つ一連の便利なフィルタを提供します。

<img src="https://laravel.com/img/docs/pail-example.png">

<a name="pail-installation"></a>
### インストール

> WARNING:  
> Laravel Pailには[PHP 8.2+](https://php.net/releases/)と[PCNTL](https://www.php.net/manual/en/book.pcntl.php)拡張機能が必要です。

始めるには、Composerパッケージマネージャを使用してPailをプロジェクトにインストールします。

```bash
composer require laravel/pail
```

<a name="pail-usage"></a>
### 使用方法

ログのテイルを開始するには、`pail`コマンドを実行します。

```bash
php artisan pail
```

出力の冗長性を上げ、切り捨て（…）を避けるには、`-v`オプションを使用します。

```bash
php artisan pail -v
```

最大の冗長性と例外スタックトレースを表示するには、`-vv`オプションを使用します。

```bash
php artisan pail -vv
```

ログのテイルを停止するには、いつでも`Ctrl+C`を押します。

<a name="pail-filtering-logs"></a>
### ログのフィルタリング

<a name="pail-filtering-logs-filter-option"></a>
#### `--filter`

ログをタイプ、ファイル、メッセージ、スタックトレースの内容でフィルタリングするには、`--filter`オプションを使用します。

```bash
php artisan pail --filter="QueryException"
```

<a name="pail-filtering-logs-message-option"></a>
#### `--message`

ログをメッセージのみでフィルタリングするには、`--message`オプションを使用します。

```bash
php artisan pail --message="User created"
```

<a name="pail-filtering-logs-level-option"></a>
#### `--level`

ログを[ログレベル](#log-levels)でフィルタリングするには、`--level`オプションを使用します。

```bash
php artisan pail --level=error
```

<a name="pail-filtering-logs-user-option"></a>
#### `--user`

特定のユーザーが認証されている間に書き込まれたログのみを表示するには、ユーザーのIDを`--user`オプションに指定します。

```bash
php artisan pail --user=1
```

