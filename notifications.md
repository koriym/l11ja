# 通知

- [はじめに](#introduction)
- [通知の生成](#generating-notifications)
- [通知の送信](#sending-notifications)
    - [Notifiableトレイトの使用](#using-the-notifiable-trait)
    - [Notificationファサードの使用](#using-the-notification-facade)
    - [配信チャネルの指定](#specifying-delivery-channels)
    - [通知のキューイング](#queueing-notifications)
    - [オンデマンド通知](#on-demand-notifications)
- [メール通知](#mail-notifications)
    - [メールメッセージのフォーマット](#formatting-mail-messages)
    - [送信者のカスタマイズ](#customizing-the-sender)
    - [受信者のカスタマイズ](#customizing-the-recipient)
    - [件名のカスタマイズ](#customizing-the-subject)
    - [メーラーのカスタマイズ](#customizing-the-mailer)
    - [テンプレートのカスタマイズ](#customizing-the-templates)
    - [添付ファイル](#mail-attachments)
    - [タグとメタデータの追加](#adding-tags-metadata)
    - [Symfonyメッセージのカスタマイズ](#customizing-the-symfony-message)
    - [Mailableの使用](#using-mailables)
    - [メール通知のプレビュー](#previewing-mail-notifications)
- [Markdownメール通知](#markdown-mail-notifications)
    - [メッセージの生成](#generating-the-message)
    - [メッセージの記述](#writing-the-message)
    - [コンポーネントのカスタマイズ](#customizing-the-components)
- [データベース通知](#database-notifications)
    - [前提条件](#database-prerequisites)
    - [データベース通知のフォーマット](#formatting-database-notifications)
    - [通知のアクセス](#accessing-the-notifications)
    - [通知を既読としてマーク](#marking-notifications-as-read)
- [ブロードキャスト通知](#broadcast-notifications)
    - [前提条件](#broadcast-prerequisites)
    - [ブロードキャスト通知のフォーマット](#formatting-broadcast-notifications)
    - [通知のリスニング](#listening-for-notifications)
- [SMS通知](#sms-notifications)
    - [前提条件](#sms-prerequisites)
    - [SMS通知のフォーマット](#formatting-sms-notifications)
    - [Unicodeコンテンツ](#unicode-content)
    - [「From」番号のカスタマイズ](#customizing-the-from-number)
    - [クライアント参照の追加](#adding-a-client-reference)
    - [SMS通知のルーティング](#routing-sms-notifications)
- [Slack通知](#slack-notifications)
    - [前提条件](#slack-prerequisites)
    - [Slack通知のフォーマット](#formatting-slack-notifications)
    - [Slackのインタラクティブ性](#slack-interactivity)
    - [Slack通知のルーティング](#routing-slack-notifications)
    - [外部Slackワークスペースへの通知](#notifying-external-slack-workspaces)
- [通知のローカライズ](#localizing-notifications)
- [テスト](#testing)
- [通知イベント](#notification-events)
- [カスタムチャネル](#custom-channels)

<a name="introduction"></a>
## はじめに

Laravelは、[メール送信](mail.md)に加えて、メール、SMS（[Vonage](https://www.vonage.com/communications-apis/)経由、以前はNexmoとして知られていました）、[Slack](https://slack.com)など、さまざまな配信チャネルで通知を送信するためのサポートを提供しています。さらに、[コミュニティが構築した通知チャネル](https://laravel-notification-channels.com/about/#suggesting-a-new-channel)を使用して、何十もの異なるチャネルで通知を送信することができます。通知は、Webインターフェースに表示されるようにデータベースに保存することもできます。

通常、通知はアプリケーションで発生した何かをユーザーに通知する短い情報メッセージです。たとえば、請求アプリケーションを作成している場合、ユーザーに「請求書支払い済み」通知をメールとSMSチャネルで送信することができます。

<a name="generating-notifications"></a>
## 通知の生成

Laravelでは、各通知は通常`app/Notifications`ディレクトリに格納される単一のクラスで表されます。アプリケーションにこのディレクトリが表示されなくても心配しないでください。`make:notification` Artisanコマンドを実行すると、作成されます。

```shell
php artisan make:notification InvoicePaid
```

このコマンドは、`app/Notifications`ディレクトリに新しい通知クラスを配置します。各通知クラスには`via`メソッドと、`toMail`や`toDatabase`などのメッセージ構築メソッドが含まれており、通知を特定のチャネルに合わせたメッセージに変換します。

<a name="sending-notifications"></a>
## 通知の送信

<a name="using-the-notifiable-trait"></a>
### Notifiableトレイトの使用

通知は、`Notifiable`トレイトの`notify`メソッドを使用するか、`Notification`[ファサード](facades.md)を使用する2つの方法で送信できます。`Notifiable`トレイトは、アプリケーションの`App\Models\User`モデルにデフォルトで含まれています。

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;
    }

このトレイトが提供する`notify`メソッドは、通知インスタンスを受け取ることを期待しています。

    use App\Notifications\InvoicePaid;

    $user->notify(new InvoicePaid($invoice));

> NOTE:  
> 覚えておいてください。`Notifiable`トレイトは、`User`モデルにのみ含める必要はありません。どのモデルにも含めることができます。

<a name="using-the-notification-facade"></a>
### Notificationファサードの使用

また、`Notification`[ファサード](facades.md)を介して通知を送信することもできます。このアプローチは、ユーザーのコレクションなど、複数の通知可能なエンティティに通知を送信する必要がある場合に便利です。ファサードを使用して通知を送信するには、すべての通知可能なエンティティと通知インスタンスを`send`メソッドに渡します。

    use Illuminate\Support\Facades\Notification;

    Notification::send($users, new InvoicePaid($invoice));

`sendNow`メソッドを使用して通知を即座に送信することもできます。このメソッドは、通知が`ShouldQueue`インターフェースを実装していても、即座に通知を送信します。

    Notification::sendNow($developers, new DeploymentCompleted($deployment));

<a name="specifying-delivery-channels"></a>
### 配信チャネルの指定

すべての通知クラスには、通知が配信されるチャネルを決定する`via`メソッドがあります。通知は、`mail`、`database`、`broadcast`、`vonage`、`slack`チャネルで送信できます。

> NOTE:  
> TelegramやPusherなどの他の配信チャネルを使用したい場合は、コミュニティ主導の[Laravel Notification Channelsウェブサイト](http://laravel-notification-channels.com)をチェックしてください。

`via`メソッドは、通知が送信されるクラスのインスタンスである`$notifiable`インスタンスを受け取ります。`$notifiable`を使用して、通知を配信するチャネルを決定できます。

    /**
     * 通知の配信チャネルを取得します。
     *
     * @return array<int, string>
     */
    public function via(object $notifiable): array
    {
        return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
    }

<a name="queueing-notifications"></a>
### 通知のキューイング

> WARNING:  
> 通知をキューに入れる前に、キューを設定し、[ワーカーを起動](queues.md#running-the-queue-worker)する必要があります。

通知の送信には時間がかかる場合があります。特に、チャネルが通知を配信するために外部API呼び出しを行う必要がある場合です。アプリケーションの応答時間を高速化するために、`ShouldQueue`インターフェースと`Queueable`トレイトをクラスに追加して、通知をキューに入れることができます。`make:notification`コマンドを使用して生成されたすべての通知に対して、インターフェースとトレイトはすでにインポートされているため、すぐに通知クラスに追加できます。

    <?php

    namespace App\Notifications;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification implements ShouldQueue
    {
        use Queueable;

        // ...
    }

`ShouldQueue`インターフェースが通知クラスに追加されると、通常どおり通知を送信できます。Laravelはクラスの`ShouldQueue`インターフェースを検出し、通知の配信を自動的にキューに入れます。

    $user->notify(new InvoicePaid($invoice));

通知をキューに入れると、受信者とチャネルの組み合わせごとにキューに入れられたジョブが作成されます。たとえば、通知に3人の受信者と2つのチャネルがある場合、6つのジョブがキューにディスパッチされます。

<a name="delaying-notifications"></a>
#### 通知の遅延

通知の配信を遅延させたい場合は、通知インスタンスに`delay`メソッドをチェーンできます。

    $delay = now()->addMinutes(10);

    $user->notify((new InvoicePaid($invoice))->delay($delay));

特定のチャネルの遅延量を指定するために、配列を`delay`メソッドに渡すことができます。

    $user->notify((new InvoicePaid($invoice))->delay([
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ]));

または、通知クラス自体に`withDelay`メソッドを定義することもできます。`withDelay`メソッドは、チャネル名と遅延値の配列を返す必要があります。

    /**
     * 通知の配信遅延を決定します。
     *
     * @return array<string, \Illuminate\Support\Carbon>
     */
    public function withDelay(object $notifiable): array
    {
        return [
            'mail' => now()->addMinutes(5),
            'sms' => now()->addMinutes(10),
        ];
    }

<a name="customizing-the-notification-queue-connection"></a>
#### 通知キュー接続のカスタマイズ

デフォルトでは、キューに入れられた通知は、アプリケーションのデフォルトのキュー接続を使用してキューに入れられます。特定の通知に対して異なる接続を使用する場合は、通知のコンストラクタから `onConnection` メソッドを呼び出すことができます。

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * 新しい通知インスタンスを作成します。
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```

また、通知がサポートする各通知チャネルに対して使用される特定のキュー接続を指定したい場合は、通知に `viaConnections` メソッドを定義できます。このメソッドは、チャネル名とキュー接続名のペアの配列を返す必要があります。

```php
/**
 * 各通知チャネルに使用する接続を決定します。
 *
 * @return array<string, string>
 */
public function viaConnections(): array
{
    return [
        'mail' => 'redis',
        'database' => 'sync',
    ];
}
```

<a name="customizing-notification-channel-queues"></a>
#### 通知チャネルのキューのカスタマイズ

通知がサポートする各通知チャネルに対して使用される特定のキューを指定したい場合は、通知に `viaQueues` メソッドを定義できます。このメソッドは、チャネル名とキュー名のペアの配列を返す必要があります。

```php
/**
 * 各通知チャネルに使用するキューを決定します。
 *
 * @return array<string, string>
 */
public function viaQueues(): array
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

<a name="queued-notification-middleware"></a>
#### キュー通知のミドルウェア

キュー通知は、[キュージョブ](queues.md#job-middleware)のようにミドルウェアを定義できます。開始するには、通知クラスに `middleware` メソッドを定義します。`middleware` メソッドは `$notifiable` と `$channel` 変数を受け取り、通知の送信先に基づいて返されるミドルウェアをカスタマイズできます。

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * 通知ジョブが通過する必要があるミドルウェアを取得します。
 *
 * @return array<int, object>
 */
public function middleware(object $notifiable, string $channel)
{
    return match ($channel) {
        'email' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

<a name="queued-notifications-and-database-transactions"></a>
#### キュー通知とデータベーストランザクション

キュー通知がデータベーストランザクション内でディスパッチされると、データベーストランザクションがコミットされる前にキューによって処理される可能性があります。この場合、データベーストランザクション中にモデルやデータベースレコードに加えた更新がまだデータベースに反映されていない可能性があります。さらに、トランザクション内で作成されたモデルやデータベースレコードがデータベースに存在しない可能性があります。通知がこれらのモデルに依存している場合、キュー通知を送信するジョブが処理されるときに予期しないエラーが発生する可能性があります。

キュー接続の `after_commit` 設定オプションが `false` に設定されている場合でも、特定のキュー通知がすべての開いているデータベーストランザクションがコミットされた後にディスパッチされるようにするには、通知を送信するときに `afterCommit` メソッドを呼び出すことができます。

```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```

または、通知のコンストラクタから `afterCommit` メソッドを呼び出すこともできます。

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * 新しい通知インスタンスを作成します。
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> NOTE:  
> これらの問題に対処する方法の詳細については、[キュージョブとデータベーストランザクション](queues.md#jobs-and-database-transactions)に関するドキュメントを確認してください。

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### キュー通知を送信するかどうかの判断

キュー通知がキューにディスパッチされ、バックグラウンド処理のためにキューに入れられた後、通常はキューワーカーによって受け入れられ、意図した受信者に送信されます。

ただし、キューワーカーによって処理された後にキュー通知を送信するかどうかを最終的に判断したい場合は、通知クラスに `shouldSend` メソッドを定義できます。このメソッドが `false` を返す場合、通知は送信されません。

```php
/**
 * 通知を送信する必要があるかどうかを判断します。
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

<a name="on-demand-notifications"></a>
### オンデマンド通知

アプリケーションの「ユーザー」として保存されていない人に通知を送信する必要がある場合があります。`Notification` ファサードの `route` メソッドを使用して、通知を送信する前にアドホックな通知ルーティング情報を指定できます。

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
            ->route('vonage', '5555555555')
            ->route('slack', '#slack-channel')
            ->route('broadcast', [new Channel('channel-name')])
            ->notify(new InvoicePaid($invoice));
```

`mail` ルートにオンデマンド通知を送信する際に受信者の名前を指定したい場合は、メールアドレスをキーとし、名前を配列の最初の要素の値として含む配列を提供できます。

```php
Notification::route('mail', [
    'barrett@example.com' => 'Barrett Blair',
])->notify(new InvoicePaid($invoice));
```

`routes` メソッドを使用して、複数の通知チャネルに対して一度にアドホックなルーティング情報を提供できます。

```php
Notification::routes([
    'mail' => ['barrett@example.com' => 'Barrett Blair'],
    'vonage' => '5555555555',
])->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>
## メール通知

<a name="formatting-mail-messages"></a>
### メールメッセージのフォーマット

通知がメールとして送信されることをサポートしている場合、通知クラスに `toMail` メソッドを定義する必要があります。このメソッドは `$notifiable` エンティティを受け取り、`Illuminate\Notifications\Messages\MailMessage` インスタンスを返す必要があります。

`MailMessage` クラスには、トランザクションメールメッセージを構築するのに役立ついくつかのシンプルなメソッドが含まれています。メールメッセージには、テキスト行と「アクションの呼び出し」を含めることができます。`toMail` メソッドの例を見てみましょう。

```php
/**
 * 通知のメール表現を取得します。
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
```

> NOTE:  
> `toMail` メソッドで `$this->invoice->id` を使用していることに注意してください。通知がメッセージを生成するために必要なデータは、通知のコンストラクタに渡すことができます。

この例では、挨拶文、テキスト行、アクションの呼び出し、そして別のテキスト行を登録しています。`MailMessage` オブジェクトによって提供されるこれらのメソッドにより、小さなトランザクションメールを簡単かつ迅速にフォーマットできます。メールチャネルは、メッセージコンポーネントを美しいレスポンシブHTMLメールテンプレートに変換し、プレーンテキストの対応物を提供します。これは、`mail` チャネルによって生成されたメールの例です。

<img src="https://laravel.com/img/docs/notification-example-2.png">

> NOTE:  
> メール通知を送信する際には、`config/app.php` 設定ファイルの `name` 設定オプションを設定してください。この値は、メール通知メッセージのヘッダーとフッターで使用されます。

<a name="error-messages"></a>
#### エラーメッセージ

一部の通知は、例えば請求書の支払いが失敗したなどのエラーをユーザーに通知します。メールメッセージがエラーに関するものであることを示すために、メッセージを構築する際に `error` メソッドを呼び出すことができます。メールメッセージで `error` メソッドを使用すると、アクションの呼び出しボタンが黒ではなく赤になります。

```php
/**
 * 通知のメール表現を取得します。
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
                ->error()
                ->subject('Invoice Payment Failed')
                ->line('...');
}
```

<a name="other-mail-notification-formatting-options"></a>
#### その他のメール通知のフォーマットオプション

通知クラスにテキストの「行」を定義する代わりに、通知メールのレンダリングに使用するカスタムテンプレートを指定するために `view` メソッドを使用できます。

```php
/**
 * 通知のメール表現を取得します。
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        'mail.invoice.paid', ['invoice' => $this->invoice]
    );
}
```

メールメッセージにプレーンテキストビューを指定するには、`view`メソッドに渡される配列の2番目の要素としてビュー名を指定します。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->view(
            ['mail.invoice.paid', 'mail.invoice.paid-text'],
            ['invoice' => $this->invoice]
        );
    }

また、メッセージがプレーンテキストビューのみを持つ場合は、`text`メソッドを使用できます。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)->text(
            'mail.invoice.paid-text', ['invoice' => $this->invoice]
        );
    }

<a name="customizing-the-sender"></a>
### 送信者のカスタマイズ

デフォルトでは、メールの送信者/送信元アドレスは`config/mail.php`設定ファイルで定義されています。ただし、`from`メソッドを使用して特定の通知の送信元アドレスを指定できます。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->from('barrett@example.com', 'Barrett Blair')
                    ->line('...');
    }

<a name="customizing-the-recipient"></a>
### 受信者のカスタマイズ

`mail`チャネル経由で通知を送信する場合、通知システムは自動的に通知可能なエンティティの`email`プロパティを探します。通知の配信に使用されるメールアドレスをカスタマイズするには、通知可能なエンティティに`routeNotificationForMail`メソッドを定義します。

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * メールチャネルの通知ルート
         *
         * @return  array<string, string>|string
         */
        public function routeNotificationForMail(Notification $notification): array|string
        {
            // メールアドレスのみを返す...
            return $this->email_address;

            // メールアドレスと名前を返す...
            return [$this->email_address => $this->name];
        }
    }

<a name="customizing-the-subject"></a>
### 件名のカスタマイズ

デフォルトでは、メールの件名は通知クラスの名前が「タイトルケース」にフォーマットされたものです。したがって、通知クラスが`InvoicePaid`という名前の場合、メールの件名は`Invoice Paid`になります。メッセージに異なる件名を指定したい場合は、メッセージを構築する際に`subject`メソッドを呼び出します。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->subject('Notification Subject')
                    ->line('...');
    }

<a name="customizing-the-mailer"></a>
### メーラーのカスタマイズ

デフォルトでは、メール通知は`config/mail.php`設定ファイルで定義されたデフォルトのメーラーを使用して送信されます。ただし、メッセージを構築する際に`mailer`メソッドを呼び出すことで、実行時に異なるメーラーを指定できます。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->mailer('postmark')
                    ->line('...');
    }

<a name="customizing-the-templates"></a>
### テンプレートのカスタマイズ

メール通知で使用されるHTMLとプレーンテキストテンプレートを変更するには、通知パッケージのリソースを公開します。このコマンドを実行すると、メール通知テンプレートは`resources/views/vendor/notifications`ディレクトリに配置されます。

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### 添付ファイル

メール通知に添付ファイルを追加するには、メッセージを構築する際に`attach`メソッドを使用します。`attach`メソッドは、ファイルへの絶対パスを最初の引数として受け取ります。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file');
    }

> NOTE:  
> 通知メールメッセージが提供する`attach`メソッドは、[添付可能なオブジェクト](mail.md#attachable-objects)も受け入れます。詳細については、[添付可能なオブジェクトのドキュメント](mail.md#attachable-objects)を参照してください。

メッセージにファイルを添付する際に、表示名と/またはMIMEタイプを指定するには、`attach`メソッドに2番目の引数として`array`を渡します。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }

メール可能オブジェクトとは異なり、ストレージディスクから直接ファイルを添付するために`attachFromStorage`を使用することはできません。代わりに、ストレージディスク上のファイルへの絶対パスを指定して`attach`メソッドを使用するか、`toMail`メソッドから[メール可能](mail.md#generating-mailables)を返すことができます。

    use App\Mail\InvoicePaid as InvoicePaidMailable;

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email)
                    ->attachFromStorage('/path/to/file');
    }

必要に応じて、`attachMany`メソッドを使用してメッセージに複数のファイルを添付できます。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachMany([
                        '/path/to/forge.svg',
                        '/path/to/vapor.svg' => [
                            'as' => 'Logo.svg',
                            'mime' => 'image/svg+xml',
                        ],
                    ]);
    }

<a name="raw-data-attachments"></a>
#### 生データの添付

`attachData`メソッドを使用して、生のバイト文字列を添付ファイルとして添付できます。`attachData`メソッドを呼び出す際に、添付ファイルに割り当てるファイル名を指定する必要があります。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Hello!')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }

<a name="adding-tags-metadata"></a>
### タグとメタデータの追加

MailgunやPostmarkなどの一部のサードパーティメールプロバイダは、メッセージの「タグ」と「メタデータ」をサポートしており、アプリケーションから送信されたメールをグループ化して追跡するために使用できます。`tag`メソッドと`metadata`メソッドを介して、メールメッセージにタグとメタデータを追加できます。

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->greeting('Comment Upvoted!')
                    ->tag('upvote')
                    ->metadata('comment_id', $this->comment->id);
    }

アプリケーションがMailgunドライバを使用している場合は、Mailgunのドキュメントを参照して、[タグ](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1)と[メタデータ](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages)の詳細を確認できます。同様に、Postmarkのドキュメントも、[タグ](https://postmarkapp.com/blog/tags-support-for-smtp)と[メタデータ](https://postmarkapp.com/support/article/1125-custom-metadata-faq)のサポートについて確認できます。

アプリケーションがAmazon SESを使用してメールを送信している場合は、`metadata`メソッドを使用してメッセージに[SES「タグ」](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html)を添付する必要があります。

<a name="customizing-the-symfony-message"></a>
### Symfonyメッセージのカスタマイズ

`MailMessage`クラスの`withSymfonyMessage`メソッドを使用すると、メッセージが送信される前にSymfonyメッセージインスタンスで呼び出されるクロージャを登録できます。これにより、メッセージが配信される前にメッセージを深くカスタマイズする機会が得られます。

    use Symfony\Component\Mime\Email;

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): MailMessage
    {
        return (new MailMessage)
                    ->withSymfonyMessage(function (Email $message) {
                        $message->getHeaders()->addTextHeader(
                            'Custom-Header', 'Header Value'
                        );
                    });
    }

<a name="using-mailables"></a>
### メール可能オブジェクトの使用

必要に応じて、通知の`toMail`メソッドから完全な[メール可能オブジェクト](mail.md)を返すことができます。`MailMessage`の代わりに`Mailable`を返す場合、メール可能オブジェクトの`to`メソッドを使用してメッセージの受信者を指定する必要があります。

    use App\Mail\InvoicePaid as InvoicePaidMailable;
    use Illuminate\Mail\Mailable;

    /**
     * 通知のメール表現を取得する
     */
    public function toMail(object $notifiable): Mailable
    {
        return (new InvoicePaidMailable($this->invoice))
                    ->to($notifiable->email);
    }

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
                ->to($notifiable->email);
}
```

<a name="mailables-and-on-demand-notifications"></a>
#### Mailablesとオンデマンド通知

[オンデマンド通知](#on-demand-notifications)を送信する場合、`toMail`メソッドに渡される`$notifiable`インスタンスは、`Illuminate\Notifications\AnonymousNotifiable`のインスタンスになります。これは、オンデマンド通知の送信先のメールアドレスを取得するために使用できる`routeNotificationFor`メソッドを提供します。

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Notifications\AnonymousNotifiable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    $address = $notifiable instanceof AnonymousNotifiable
            ? $notifiable->routeNotificationFor('mail')
            : $notifiable->email;

    return (new InvoicePaidMailable($this->invoice))
                ->to($address);
}
```

<a name="previewing-mail-notifications"></a>
### メール通知のプレビュー

メール通知テンプレートを設計する際、典型的なBladeテンプレートのように、レンダリングされたメールメッセージをブラウザですばやくプレビューすると便利です。このため、Laravelでは、ルートクロージャまたはコントローラからメール通知によって生成されたメールメッセージを直接返すことができます。`MailMessage`が返されると、レンダリングされてブラウザに表示され、実際のメールアドレスに送信する必要なく、そのデザインをすばやくプレビューできます。

```php
use App\Models\Invoice;
use App\Notifications\InvoicePaid;

Route::get('/notification', function () {
    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))
                ->toMail($invoice->user);
});
```

<a name="markdown-mail-notifications"></a>
## Markdownメール通知

Markdownメール通知を使用すると、メール通知の事前構築済みテンプレートを利用しながら、より長くカスタマイズされたメッセージを記述する自由度が得られます。メッセージはMarkdownで記述されるため、Laravelはメッセージのために美しいレスポンシブHTMLテンプレートをレンダリングするだけでなく、プレーンテキストの対応物も自動的に生成できます。

<a name="generating-the-message"></a>
### メッセージの生成

対応するMarkdownテンプレートを含む通知を生成するには、`make:notification` Artisanコマンドの`--markdown`オプションを使用できます。

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

他のすべてのメール通知と同様に、Markdownテンプレートを使用する通知は、通知クラスに`toMail`メソッドを定義する必要があります。ただし、通知を構築するために`line`および`action`メソッドを使用する代わりに、使用するMarkdownテンプレートの名前を指定するために`markdown`メソッドを使用します。テンプレートで利用可能にしたいデータの配列をメソッドの2番目の引数として渡すことができます。

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="writing-the-message"></a>
### メッセージの記述

Markdownメール通知は、BladeコンポーネントとMarkdown構文を組み合わせて使用し、Laravelの事前構築済み通知コンポーネントを活用しながら、通知を簡単に構築できるようにします。

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

<a name="button-component"></a>
#### ボタンコンポーネント

ボタンコンポーネントは、中央揃えのボタンリンクをレンダリングします。コンポーネントは2つの引数を受け取ります。`url`とオプションの`color`です。サポートされている色は`primary`、`green`、`red`です。通知には必要なだけボタンコンポーネントを追加できます。

```blade
<x-mail::button :url="$url" color="green">
View Invoice
</x-mail::button>
```

<a name="panel-component"></a>
#### パネルコンポーネント

パネルコンポーネントは、通知の残りの部分とは少し異なる背景色を持つパネル内に指定されたテキストブロックをレンダリングします。これにより、指定されたテキストブロックに注意を引くことができます。

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### テーブルコンポーネント

テーブルコンポーネントを使用すると、MarkdownテーブルをHTMLテーブルに変換できます。コンポーネントはMarkdownテーブルをコンテンツとして受け取ります。テーブル列の配置は、デフォルトのMarkdownテーブル配置構文を使用してサポートされます。

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### コンポーネントのカスタマイズ

Markdown通知コンポーネントをすべて自分のアプリケーションにエクスポートしてカスタマイズすることができます。コンポーネントをエクスポートするには、`vendor:publish` Artisanコマンドを使用して`laravel-mail`アセットタグを公開します。

```shell
php artisan vendor:publish --tag=laravel-mail
```

このコマンドは、Markdownメールコンポーネントを`resources/views/vendor/mail`ディレクトリに公開します。`mail`ディレクトリには、`html`と`text`ディレクトリが含まれ、それぞれに利用可能なすべてのコンポーネントの表現が含まれます。これらのコンポーネントは自由にカスタマイズできます。

<a name="customizing-the-css"></a>
#### CSSのカスタマイズ

コンポーネントをエクスポートした後、`resources/views/vendor/mail/html/themes`ディレクトリに`default.css`ファイルが含まれます。このファイルのCSSをカスタマイズすると、Markdown通知のHTML表現内にスタイルが自動的にインライン化されます。

LaravelのMarkdownコンポーネントのためにまったく新しいテーマを構築したい場合、`html/themes`ディレクトリ内にCSSファイルを配置できます。CSSファイルに名前を付けて保存した後、`mail`設定ファイルの`theme`オプションを新しいテーマの名前と一致するように更新します。

個々の通知のためにテーマをカスタマイズするには、通知のメールメッセージを構築する際に`theme`メソッドを呼び出すことができます。`theme`メソッドは、通知の送信時に使用するテーマの名前を受け取ります。

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
                ->theme('invoice')
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="database-notifications"></a>
## データベース通知

<a name="database-prerequisites"></a>
### 前提条件

`database`通知チャネルは、通知情報をデータベーステーブルに保存します。このテーブルには、通知タイプや通知を説明するJSONデータ構造などの情報が含まれます。

アプリケーションのユーザーインターフェースに通知を表示するためにテーブルをクエリすることができます。ただし、その前に、通知を保持するためのデータベーステーブルを作成する必要があります。`make:notifications-table`コマンドを使用して、適切なテーブルスキーマを持つ[マイグレーション](migrations.md)を生成できます。

```shell
php artisan make:notifications-table

php artisan migrate
```

> NOTE:  
> 通知可能なモデルが[UUIDまたはULIDプライマリキー](eloquent.md#uuid-and-ulid-keys)を使用している場合、通知テーブルのマイグレーションで`morphs`メソッドを[`uuidMorphs`](migrations.md#column-method-uuidMorphs)または[`ulidMorphs`](migrations.md#column-method-ulidMorphs)に置き換える必要があります。

<a name="formatting-database-notifications"></a>
### データベース通知のフォーマット

通知がデータベーステーブルに保存されることをサポートしている場合、通知クラスに`toDatabase`または`toArray`メソッドを定義する必要があります。このメソッドは`$notifiable`エンティティを受け取り、プレーンなPHP配列を返す必要があります。返された配列はJSONにエンコードされ、`notifications`テーブルの`data`列に保存されます。`toArray`メソッドの例を見てみましょう。

```php
/**
 * Get the array representation of the notification.
 *
 * @return array<string, mixed>
 */
public function toArray(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

通知がアプリケーションのデータベースに保存されると、`type`列に通知のクラス名が入力されます。ただし、通知クラスに`databaseType`メソッドを定義することで、この動作をカスタマイズできます。

```php
/**
 * Get the notification's database type.
 *
 * @return string
 */
public function databaseType(object $notifiable): string
{
    return 'invoice-paid';
}
```

<a name="todatabase-vs-toarray"></a>
#### `toDatabase` vs. `toArray`

`toArray`メソッドは、JavaScript駆動のフロントエンドにブロードキャストするデータを決定するために`broadcast`チャネルによっても使用されます。`database`チャネルと`broadcast`チャネルに対して2つの異なる配列表現を持ちたい場合は、`toArray`メソッドの代わりに`toDatabase`メソッドを定義する必要があります。

<a name="accessing-the-notifications"></a>
### 通知へのアクセス

通知がデータベースに保存された後、通知可能なエンティティからそれらにアクセスするための便利な方法が必要です。Laravelのデフォルトの`App\Models\User`モデルには、`Illuminate\Notifications\Notifiable`トレイトが含まれており、`notifications` Eloquentリレーションを返す`notifications`メソッドが含まれています。このメソッドを使用して通知にアクセスするには、他のEloquentリレーションと同様に使用できます。デフォルトでは、通知は`created_at`タイムスタンプに基づいて並べ替えられます。

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

未読通知のみを取得したい場合は、`unreadNotifications`リレーションを使用できます。この場合も、通知は`created_at`タイムスタンプに基づいて並べ替えられます。

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

> NOTE:  
> JavaScriptクライアントから通知にアクセスするには、現在のユーザーなどの通知可能なエンティティの通知を返す通知コントローラを定義する必要があります。その後、JavaScriptクライアントからそのコントローラのURLにHTTPリクエストを送信できます。

通知がデータベースに保存されたら、通知可能なエンティティからそれらにアクセスする便利な方法が必要です。Laravelのデフォルトの`App\Models\User`モデルに含まれている`Illuminate\Notifications\Notifiable`トレイトには、エンティティの通知を返す`notifications` [Eloquentリレーション](eloquent-relationships.md)が含まれています。通知を取得するには、他のEloquentリレーションと同様にこのメソッドにアクセスできます。デフォルトでは、通知は`created_at`タイムスタンプでソートされ、最新の通知がコレクションの先頭になります。

    $user = App\Models\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

「未読」の通知のみを取得したい場合は、`unreadNotifications`リレーションを使用できます。これらの通知も`created_at`タイムスタンプでソートされ、最新の通知がコレクションの先頭になります。

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> NOTE:  
> JavaScriptクライアントから通知にアクセスするには、アプリケーションに通知コントローラを定義し、現在のユーザーなどの通知可能なエンティティの通知を返す必要があります。その後、JavaScriptクライアントからそのコントローラのURLにHTTPリクエストを行うことができます。

<a name="marking-notifications-as-read"></a>
### 通知を既読にする

通常、ユーザーが通知を閲覧したら、その通知を「既読」としてマークしたいでしょう。`Illuminate\Notifications\Notifiable`トレイトは、通知のデータベースレコードの`read_at`列を更新する`markAsRead`メソッドを提供します。

    $user = App\Models\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

ただし、各通知をループする代わりに、通知のコレクションに直接`markAsRead`メソッドを使用できます。

    $user->unreadNotifications->markAsRead();

データベースから通知を取得せずに、すべての通知を既読として一括更新するクエリを使用することもできます。

    $user = App\Models\User::find(1);

    $user->unreadNotifications()->update(['read_at' => now()]);

通知を完全にテーブルから削除するために`delete`することもできます。

    $user->notifications()->delete();

<a name="broadcast-notifications"></a>
## ブロードキャスト通知

<a name="broadcast-prerequisites"></a>
### 前提条件

通知をブロードキャストする前に、Laravelの[イベントブロードキャスト](broadcasting.md)サービスを設定し、理解しておく必要があります。イベントブロードキャストは、JavaScriptフロントエンドからサーバーサイドのLaravelイベントに反応する方法を提供します。

<a name="formatting-broadcast-notifications"></a>
### ブロードキャスト通知のフォーマット

`broadcast`チャンネルは、Laravelの[イベントブロードキャスト](broadcasting.md)サービスを使用して通知をブロードキャストし、JavaScriptフロントエンドがリアルタイムで通知をキャッチできるようにします。通知がブロードキャストをサポートしている場合、通知クラスに`toBroadcast`メソッドを定義できます。このメソッドは`$notifiable`エンティティを受け取り、`BroadcastMessage`インスタンスを返す必要があります。`toBroadcast`メソッドが存在しない場合、`toArray`メソッドがブロードキャストするデータを収集するために使用されます。返されたデータはJSONにエンコードされ、JavaScriptフロントエンドにブロードキャストされます。`toBroadcast`メソッドの例を見てみましょう。

    use Illuminate\Notifications\Messages\BroadcastMessage;

    /**
     * 通知のブロードキャスト可能な表現を取得します。
     */
    public function toBroadcast(object $notifiable): BroadcastMessage
    {
        return new BroadcastMessage([
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ]);
    }

<a name="broadcast-queue-configuration"></a>
#### ブロードキャストキューの設定

すべてのブロードキャスト通知は、ブロードキャストのためにキューに入れられます。ブロードキャスト操作に使用されるキュー接続またはキュー名を設定したい場合は、`BroadcastMessage`の`onConnection`および`onQueue`メソッドを使用できます。

    return (new BroadcastMessage($data))
                    ->onConnection('sqs')
                    ->onQueue('broadcasts');

<a name="customizing-the-notification-type"></a>
#### 通知タイプのカスタマイズ

指定したデータに加えて、すべてのブロードキャスト通知には通知の完全なクラス名を含む`type`フィールドも含まれます。通知の`type`をカスタマイズしたい場合は、通知クラスに`broadcastType`メソッドを定義できます。

    /**
     * ブロードキャストされる通知のタイプを取得します。
     */
    public function broadcastType(): string
    {
        return 'broadcast.message';
    }

<a name="listening-for-notifications"></a>
### 通知のリスニング

通知は、`{notifiable}.{id}`という規則を使用してフォーマットされたプライベートチャンネルでブロードキャストされます。したがって、IDが`1`の`App\Models\User`インスタンスに通知を送信する場合、通知は`App.Models.User.1`プライベートチャンネルでブロードキャストされます。[Laravel Echo](broadcasting.md#client-side-installation)を使用する場合、`notification`メソッドを使用してチャンネルで通知を簡単にリスニングできます。

    Echo.private('App.Models.User.' + userId)
        .notification((notification) => {
            console.log(notification.type);
        });

<a name="customizing-the-notification-channel"></a>
#### 通知チャンネルのカスタマイズ

エンティティのブロードキャスト通知がブロードキャストされるチャンネルをカスタマイズしたい場合は、通知可能なエンティティに`receivesBroadcastNotificationsOn`メソッドを定義できます。

    <?php

    namespace App\Models;

    use Illuminate\Broadcasting\PrivateChannel;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * ユーザーが通知ブロードキャストを受信するチャンネル。
         */
        public function receivesBroadcastNotificationsOn(): string
        {
            return 'users.'.$this->id;
        }
    }

<a name="sms-notifications"></a>
## SMS通知

<a name="sms-prerequisites"></a>
### 前提条件

LaravelでのSMS通知の送信は、[Vonage](https://www.vonage.com/)（旧Nexmo）によって提供されます。Vonageを介して通知を送信する前に、`laravel/vonage-notification-channel`および`guzzlehttp/guzzle`パッケージをインストールする必要があります。

    composer require laravel/vonage-notification-channel guzzlehttp/guzzle

このパッケージには[設定ファイル](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php)が含まれています。ただし、この設定ファイルを自分のアプリケーションにエクスポートする必要はありません。単に`VONAGE_KEY`および`VONAGE_SECRET`環境変数を使用して、Vonageの公開鍵と秘密鍵を定義できます。

鍵を定義した後、`VONAGE_SMS_FROM`環境変数を設定して、SMSメッセージをデフォルトで送信する電話番号を定義する必要があります。この電話番号は、Vonageコントロールパネル内で生成できます。

    VONAGE_SMS_FROM=15556666666

<a name="formatting-sms-notifications"></a>
### SMS通知のフォーマット

通知がSMSとして送信されることをサポートしている場合、通知クラスに`toVonage`メソッドを定義する必要があります。このメソッドは`$notifiable`エンティティを受け取り、`Illuminate\Notifications\Messages\VonageMessage`インスタンスを返す必要があります。

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * 通知のVonage / SMS表現を取得します。
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content');
    }

<a name="unicode-content"></a>
#### Unicodeコンテンツ

SMSメッセージにUnicode文字が含まれる場合、`VonageMessage`インスタンスを構築する際に`unicode`メソッドを呼び出す必要があります。

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * 通知のVonage / SMS表現を取得します。
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your unicode message')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### "From"番号のカスタマイズ

いくつかの通知を`VONAGE_SMS_FROM`環境変数で指定された電話番号とは異なる電話番号から送信したい場合は、`VonageMessage`インスタンスの`from`メソッドを呼び出すことができます。

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * 通知のVonage / SMS表現を取得します。
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->content('Your SMS message content')
                    ->from('15554443333');
    }

<a name="adding-a-client-reference"></a>
### クライアント参照の追加

ユーザー、チーム、またはクライアントごとのコストを追跡したい場合、通知に「クライアント参照」を追加できます。Vonageでは、このクライアント参照を使用してレポートを生成できるため、特定の顧客のSMS使用状況をよりよく理解できます。クライアント参照は、最大40文字の任意の文字列です。

    use Illuminate\Notifications\Messages\VonageMessage;

    /**
     * 通知のVonage / SMS表現を取得します。
     */
    public function toVonage(object $notifiable): VonageMessage
    {
        return (new VonageMessage)
                    ->clientReference((string) $notifiable->id)
                    ->content('Your SMS message content');
    }

<a name="routing-sms-notifications"></a>
### SMS通知のルーティング

Vonage通知を適切な電話番号にルーティングするには、通知可能なエンティティに`routeNotificationForVonage`メソッドを定義します。

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Vonageチャンネルの通知をルーティングする。
         */
        public function routeNotificationForVonage(Notification $notification): string
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Slack通知

<a name="slack-prerequisites"></a>
### 前提条件

Slack通知を送信する前に、Composerを介してSlack通知チャンネルをインストールする必要があります。

```shell
composer require laravel/slack-notification-channel
```

さらに、Slackワークスペース用に[Slackアプリ](https://api.slack.com/apps?new_app=1)を作成する必要があります。

アプリが作成された同じSlackワークスペースにのみ通知を送信する場合は、アプリに`chat:write`、`chat:write.public`、および`chat:write.customize`スコープがあることを確認する必要があります。これらのスコープは、Slack内の「OAuth & Permissions」アプリ管理タブから追加できます。

次に、アプリの「Bot User OAuth Token」をコピーし、アプリケーションの`services.php`設定ファイル内の`slack`設定配列に配置します。このトークンは、Slack内の「OAuth & Permissions」タブで見つけることができます。

    'slack' => [
        'notifications' => [
            'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
            'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
        ],
    ],

<a name="slack-app-distribution"></a>
#### アプリの配布

アプリケーションがアプリケーションのユーザーが所有する外部Slackワークスペースに通知を送信する場合、Slackを介してアプリを「配布」する必要があります。アプリの配布は、Slack内のアプリの「Manage Distribution」タブから管理できます。アプリが配布されたら、[Socialite](socialite.md)を使用して、アプリケーションのユーザーに代わって[Slack Botトークンを取得](socialite.md#slack-bot-scopes)できます。

<a name="formatting-slack-notifications"></a>
### Slack通知のフォーマット

通知がSlackメッセージとして送信されることをサポートする場合、通知クラスに`toSlack`メソッドを定義する必要があります。このメソッドは`$notifiable`エンティティを受け取り、`Illuminate\Notifications\Slack\SlackMessage`インスタンスを返す必要があります。[SlackのBlock Kit API](https://api.slack.com/block-kit)を使用してリッチな通知を構築できます。以下の例は、[SlackのBlock Kitビルダー](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D)でプレビューできます。

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * 通知のSlack表現を取得する。
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                    $block->field("*Invoice No:*\n1000")->markdown();
                    $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
                })
                ->dividerBlock()
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('Congratulations!');
                });
    }

<a name="slack-interactivity"></a>
### Slackのインタラクティブ機能

SlackのBlock Kit通知システムは、[ユーザーのインタラクションを処理する](https://api.slack.com/interactivity/handling)強力な機能を提供します。これらの機能を利用するには、Slackアプリで「インタラクティブ性」を有効にし、アプリケーションによって提供されるURLを指す「リクエストURL」を設定する必要があります。これらの設定は、Slack内の「Interactivity & Shortcuts」アプリ管理タブから管理できます。

以下の例では、`actionsBlock`メソッドを使用していますが、SlackはボタンをクリックしたSlackユーザー、クリックされたボタンのIDなどを含むペイロードを持つ`POST`リクエストを「リクエストURL」に送信します。その後、アプリケーションはペイロードに基づいてアクションを決定できます。また、リクエストがSlackによって行われたことを[検証する](https://api.slack.com/authentication/verifying-requests-from-slack)必要があります。

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * 通知のSlack表現を取得する。
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                     // IDはデフォルトで "button_acknowledge_invoice" になります...
                    $block->button('Acknowledge Invoice')->primary();

                    // IDを手動で設定...
                    $block->button('Deny')->danger()->id('deny_invoice');
                });
    }

<a name="slack-confirmation-modals"></a>
#### 確認モーダル

ユーザーがアクションを実行する前に確認を求める場合、ボタンを定義する際に`confirm`メソッドを呼び出すことができます。`confirm`メソッドはメッセージとクロージャを受け取り、クロージャは`ConfirmObject`インスタンスを受け取ります。

    use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
    use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
    use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
    use Illuminate\Notifications\Slack\SlackMessage;

    /**
     * 通知のSlack表現を取得する。
     */
    public function toSlack(object $notifiable): SlackMessage
    {
        return (new SlackMessage)
                ->text('One of your invoices has been paid!')
                ->headerBlock('Invoice Paid')
                ->contextBlock(function (ContextBlock $block) {
                    $block->text('Customer #1234');
                })
                ->sectionBlock(function (SectionBlock $block) {
                    $block->text('An invoice has been paid.');
                })
                ->actionsBlock(function (ActionsBlock $block) {
                    $block->button('Acknowledge Invoice')
                        ->primary()
                        ->confirm(
                            'Acknowledge the payment and send a thank you email?',
                            function (ConfirmObject $dialog) {
                                $dialog->confirm('Yes');
                                $dialog->deny('No');
                            }
                        );
                });
    }

<a name="inspecting-slack-blocks"></a>
#### Slackブロックの検査

構築したブロックをすばやく検査したい場合、`SlackMessage`インスタンスで`dd`メソッドを呼び出すことができます。`dd`メソッドは、ペイロードと通知のプレビューをブラウザで表示する[Block Kitビルダー](https://app.slack.com/block-kit-builder/)へのURLを生成してダンプします。`dd`メソッドに`true`を渡すと、生のペイロードをダンプします。

    return (new SlackMessage)
            ->text('One of your invoices has been paid!')
            ->headerBlock('Invoice Paid')
            ->dd();

<a name="routing-slack-notifications"></a>
### Slack通知のルーティング

Slack通知を適切なSlackチームとチャンネルに送信するには、通知可能なモデルに`routeNotificationForSlack`メソッドを定義します。このメソッドは次の3つの値のいずれかを返すことができます。

- `null` - 通知自体で設定されたチャンネルへのルーティングを延期します。通知を構築する際に`to`メソッドを使用して、通知内でチャンネルを設定できます。
- 通知を送信するSlackチャンネルを指定する文字列（例：`#support-channel`）。
- `SlackRoute`インスタンス - OAuthトークンとチャンネル名を指定できます（例：`SlackRoute::make($this->slack_channel, $this->slack_token)`）。このメソッドは、外部ワークスペースに通知を送信する場合に使用する必要があります。

例えば、`routeNotificationForSlack`メソッドから`#support-channel`を返すと、アプリケーションの`services.php`設定ファイルにあるBot User OAuthトークンに関連付けられたワークスペースの`#support-channel`チャンネルに通知が送信されます。

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Slackチャンネルへの通知ルート
         */
        public function routeNotificationForSlack(Notification $notification): mixed
        {
            return '#support-channel';
        }
    }

<a name="notifying-external-slack-workspaces"></a>
### 外部のSlackワークスペースへの通知

> NOTE:  
> 外部のSlackワークスペースに通知を送信する前に、Slackアプリが[配布](#slack-app-distribution)されている必要があります。

もちろん、アプリケーションのユーザーが所有するSlackワークスペースに通知を送信したい場合が多いでしょう。そのためには、まずユーザーのSlack OAuthトークンを取得する必要があります。幸いなことに、[Laravel Socialite](socialite.md)にはSlackドライバーが含まれており、アプリケーションのユーザーをSlackで簡単に認証し、[ボットトークンを取得](socialite.md#slack-bot-scopes)できます。

ボットトークンを取得してアプリケーションのデータベースに保存したら、`SlackRoute::make`メソッドを使用して、ユーザーのワークスペースに通知をルーティングできます。さらに、アプリケーションはユーザーが通知を送信するチャンネルを指定する機会を提供する必要があるでしょう。

    <?php

    namespace App\Models;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Illuminate\Notifications\Notification;
    use Illuminate\Notifications\Slack\SlackRoute;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * Slackチャンネルへの通知ルート
         */
        public function routeNotificationForSlack(Notification $notification): mixed
        {
            return SlackRoute::make($this->slack_channel, $this->slack_token);
        }
    }

<a name="localizing-notifications"></a>
## 通知のローカライズ

Laravelでは、HTTPリクエストの現在のロケール以外のロケールで通知を送信できます。通知がキューに入れられている場合、このロケールも記憶されます。

これを実現するために、`Illuminate\Notifications\Notification`クラスは、希望する言語を設定するための`locale`メソッドを提供します。通知が評価されている間、アプリケーションはこのロケールに変更され、評価が完了すると前のロケールに戻ります。

    $user->notify((new InvoicePaid($invoice))->locale('es'));

複数の通知可能なエントリのローカライズも、`Notification`ファサードを介して行うことができます。

    Notification::locale('es')->send(
        $users, new InvoicePaid($invoice)
    );

<a name="user-preferred-locales"></a>
### ユーザーの優先ロケール

アプリケーションによっては、各ユーザーの優先ロケールを保存している場合があります。`HasLocalePreference`コントラクトを通知可能なモデルに実装することで、Laravelに通知送信時にこの保存されたロケールを使用するよう指示できます。

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * ユーザーの優先ロケールを取得
         */
        public function preferredLocale(): string
        {
            return $this->locale;
        }
    }

このインターフェースを実装すると、Laravelは通知やメールをモデルに送信する際に自動的に優先ロケールを使用します。したがって、このインターフェースを使用する場合、`locale`メソッドを呼び出す必要はありません。

    $user->notify(new InvoicePaid($invoice));

<a name="testing"></a>
## テスト

`Notification`ファサードの`fake`メソッドを使用して、通知が送信されないようにすることができます。通常、通知の送信は実際にテストしているコードとは無関係です。ほとんどの場合、Laravelに特定の通知を送信するよう指示されたことを単にアサートするだけで十分です。

`Notification`ファサードの`fake`メソッドを呼び出した後、通知がユーザーに送信されるように指示されたことをアサートし、通知が受け取ったデータを検査することもできます。

===  "Pest"
```php
<?php

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;

test('orders can be shipped', function () {
    Notification::fake();

    // 注文の発送を実行...

    // 通知が送信されなかったことをアサート...
    Notification::assertNothingSent();

    // 指定されたユーザーに通知が送信されたことをアサート...
    Notification::assertSentTo(
        [$user], OrderShipped::class
    );

    // 通知が送信されなかったことをアサート...
    Notification::assertNotSentTo(
        [$user], AnotherNotification::class
    );

    // 指定された数の通知が送信されたことをアサート...
    Notification::assertCount(3);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Notification::fake();

        // 注文の発送を実行...

        // 通知が送信されなかったことをアサート...
        Notification::assertNothingSent();

        // 指定されたユーザーに通知が送信されたことをアサート...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // 通知が送信されなかったことをアサート...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // 指定された数の通知が送信されたことをアサート...
        Notification::assertCount(3);
    }
}
```

`assertSentTo`または`assertNotSentTo`メソッドにクロージャを渡して、指定された「真実テスト」を通過する通知が送信されたことをアサートすることもできます。指定された真実テストを通過する通知が少なくとも1つ送信された場合、アサーションは成功します。

    Notification::assertSentTo(
        $user,
        function (OrderShipped $notification, array $channels) use ($order) {
            return $notification->order->id === $order->id;
        }
    );

<a name="on-demand-notifications"></a>
#### オンデマンド通知

テストしているコードが[オンデマンド通知](#on-demand-notifications)を送信する場合、`assertSentOnDemand`メソッドを使用して、オンデマンド通知が送信されたことをテストできます。

    Notification::assertSentOnDemand(OrderShipped::class);

`assertSentOnDemand`メソッドにクロージャを第2引数として渡すことで、オンデマンド通知が正しい「ルート」アドレスに送信されたかどうかを判断できます。

    Notification::assertSentOnDemand(
        OrderShipped::class,
        function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
            return $notifiable->routes['mail'] === $user->email;
        }
    );

<a name="notification-events"></a>
## 通知イベント

<a name="notification-sending-event"></a>
#### 通知送信イベント

通知が送信されると、通知システムによって`Illuminate\Notifications\Events\NotificationSending`イベントがディスパッチされます。これには「通知可能」エンティティと通知インスタンス自体が含まれます。このイベントの[イベントリスナー](events.md)をアプリケーション内に作成できます。

    use Illuminate\Notifications\Events\NotificationSending;

    class CheckNotificationStatus
    {
        /**
         * 指定されたイベントを処理
         */
        public function handle(NotificationSending $event): void
        {
            // ...
        }
    }

`NotificationSending`イベントのイベントリスナーが`handle`メソッドから`false`を返す場合、通知は送信されません。

    /**
     * 指定されたイベントを処理
     */
    public function handle(NotificationSending $event): bool
    {
        return false;
    }

イベントリスナー内で、イベントの`notifiable`、`notification`、および`channel`プロパティにアクセスして、通知の受信者や通知自体について詳しく知ることができます。

    /**
     * 指定されたイベントを処理
     */
    public function handle(NotificationSending $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
    }

<a name="notification-sent-event"></a>
#### 通知送信済みイベント

通知が送信されると、通知システムによって`Illuminate\Notifications\Events\NotificationSent`[イベント](events.md)がディスパッチされます。これには「通知可能」エンティティと通知インスタンス自体が含まれます。このイベントの[イベントリスナー](events.md)をアプリケーション内に作成できます。

    use Illuminate\Notifications\Events\NotificationSent;

    class LogNotification
    {
        /**
         * 指定されたイベントを処理
         */
        public function handle(NotificationSent $event): void
        {
            // ...
        }
    }

イベントリスナー内で、イベントの`notifiable`、`notification`、`channel`、および`response`プロパティにアクセスして、通知の受信者や通知自体について詳しく知ることができます。

    /**
     * 指定されたイベントを処理
     */
    public function handle(NotificationSent $event): void
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## カスタムチャンネル


Laravelにはいくつかの通知チャンネルが同梱されていますが、他のチャンネルを介して通知を配信するために独自のドライバを作成したい場合があります。Laravelはこれを簡単に行えるようにしています。まず、`send`メソッドを含むクラスを定義します。このメソッドは、`$notifiable`と`$notification`の2つの引数を受け取る必要があります。

`send`メソッド内で、通知のメソッドを呼び出してチャンネルが理解できるメッセージオブジェクトを取得し、その後、希望する方法で`$notifiable`インスタンスに通知を送信できます。

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * 指定された通知を送信する。
     */
    public function send(object $notifiable, Notification $notification): void
    {
        $message = $notification->toVoice($notifiable);

        // $notifiableインスタンスに通知を送信...
    }
}
```

通知チャンネルクラスを定義したら、任意の通知の`via`メソッドからクラス名を返すことができます。この例では、通知の`toVoice`メソッドは音声メッセージを表すオブジェクトを返すことができます。例えば、これらのメッセージを表すために独自の`VoiceMessage`クラスを定義することができます。

```php
<?php

namespace App\Notifications;

use App\Notifications\Messages\VoiceMessage;
use App\Notifications\VoiceChannel;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * 通知チャンネルを取得する。
     */
    public function via(object $notifiable): string
    {
        return VoiceChannel::class;
    }

    /**
     * 通知の音声表現を取得する。
     */
    public function toVoice(object $notifiable): VoiceMessage
    {
        // ...
    }
}
```

