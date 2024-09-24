# メール

- [はじめに](#introduction)
    - [設定](#configuration)
    - [ドライバの前提条件](#driver-prerequisites)
    - [フェイルオーバー設定](#failover-configuration)
    - [ラウンドロビン設定](#round-robin-configuration)
- [メールブル生成](#generating-mailables)
- [メールブルの記述](#writing-mailables)
    - [送信者の設定](#configuring-the-sender)
    - [ビューの設定](#configuring-the-view)
    - [ビューデータ](#view-data)
    - [添付ファイル](#attachments)
    - [インライン添付ファイル](#inline-attachments)
    - [添付可能なオブジェクト](#attachable-objects)
    - [ヘッダー](#headers)
    - [タグとメタデータ](#tags-and-metadata)
    - [Symfonyメッセージのカスタマイズ](#customizing-the-symfony-message)
- [Markdownメールブル](#markdown-mailables)
    - [Markdownメールブルの生成](#generating-markdown-mailables)
    - [Markdownメッセージの記述](#writing-markdown-messages)
    - [コンポーネントのカスタマイズ](#customizing-the-components)
- [メールの送信](#sending-mail)
    - [メールのキューイング](#queueing-mail)
- [メールブルのレンダリング](#rendering-mailables)
    - [ブラウザでのメールブルのプレビュー](#previewing-mailables-in-the-browser)
- [メールブルのローカライズ](#localizing-mailables)
- [テスト](#testing-mailables)
    - [メールブルの内容のテスト](#testing-mailable-content)
    - [メールブルの送信のテスト](#testing-mailable-sending)
- [メールとローカル開発](#mail-and-local-development)
- [イベント](#events)
- [カスタムトランスポート](#custom-transports)
    - [追加のSymfonyトランスポート](#additional-symfony-transports)

<a name="introduction"></a>
## はじめに

メールの送信は、必ずしも複雑である必要はありません。Laravelは、人気のある[Symfony Mailer](https://symfony.com/doc/7.0/mailer.html)コンポーネントを利用した、クリーンでシンプルなメールAPIを提供します。LaravelとSymfony Mailerは、SMTP、Mailgun、Postmark、Resend、Amazon SES、`sendmail`を介してメールを送信するためのドライバを提供し、ローカルまたはクラウドベースのサービスを介してメールをすばやく送信できるようにします。

<a name="configuration"></a>
### 設定

Laravelのメールサービスは、アプリケーションの`config/mail.php`設定ファイルを介して設定できます。このファイル内で設定された各メーラーは、独自の固有の設定を持つことができ、独自の固有の「トランスポート」を持つことができます。これにより、アプリケーションは特定のメールメッセージを送信するために異なるメールサービスを使用できます。たとえば、アプリケーションはPostmarkを使用してトランザクションメールを送信し、Amazon SESを使用して一括メールを送信する場合があります。

`mail`設定ファイル内で、`mailers`設定配列を見つけることができます。この配列には、Laravelがサポートする主要なメールドライバ/トランスポートごとにサンプル設定エントリが含まれています。一方、`default`設定値は、アプリケーションがメールメッセージを送信する必要がある場合にデフォルトで使用するメーラーを決定します。

<a name="driver-prerequisites"></a>
### ドライバ/トランスポートの前提条件

MailgunやPostmarkなどのAPIベースのドライバは、SMTPサーバーを介してメールを送信するよりもシンプルで高速です。可能な限り、これらのドライバのいずれかを使用することをお勧めします。

<a name="mailgun-driver"></a>
#### Mailgunドライバ

Mailgunドライバを使用するには、Composerを介してSymfonyのMailgun Mailerトランスポートをインストールします。

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

次に、アプリケーションの`config/mail.php`設定ファイルで`default`オプションを`mailgun`に設定し、`mailers`の配列に次の設定配列を追加します。

    'mailgun' => [
        'transport' => 'mailgun',
        // 'client' => [
        //     'timeout' => 5,
        // ],
    ],

アプリケーションのデフォルトメーラーを設定した後、`config/services.php`設定ファイルに次のオプションを追加してください。

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
        'scheme' => 'https',
    ],

米国以外の[Mailgunリージョン](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions)を使用していない場合は、`services`設定ファイルでリージョンのエンドポイントを定義できます。

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
        'scheme' => 'https',
    ],

<a name="postmark-driver"></a>
#### Postmarkドライバ

[Postmark](https://postmarkapp.com/)ドライバを使用するには、Composerを介してSymfonyのPostmark Mailerトランスポートをインストールします。

```shell
composer require symfony/postmark-mailer symfony/http-client
```

次に、アプリケーションの`config/mail.php`設定ファイルで`default`オプションを`postmark`に設定します。アプリケーションのデフォルトメーラーを設定した後、`config/services.php`設定ファイルに次のオプションが含まれていることを確認してください。

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

特定のメーラーによって使用されるべきPostmarkメッセージストリームを指定したい場合は、メーラーの設定配列に`message_stream_id`設定オプションを追加できます。この設定配列は、アプリケーションの`config/mail.php`設定ファイルにあります。

    'postmark' => [
        'transport' => 'postmark',
        'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
        // 'client' => [
        //     'timeout' => 5,
        // ],
    ],

このようにして、異なるメッセージストリームを持つ複数のPostmarkメーラーを設定できます。

<a name="resend-driver"></a>
#### Resendドライバ

[Resend](https://resend.com/)ドライバを使用するには、Composerを介してResendのPHP SDKをインストールします。

```shell
composer require resend/resend-php
```

次に、アプリケーションの`config/mail.php`設定ファイルで`default`オプションを`resend`に設定します。アプリケーションのデフォルトメーラーを設定した後、`config/services.php`設定ファイルに次のオプションが含まれていることを確認してください。

    'resend' => [
        'key' => env('RESEND_KEY'),
    ],

<a name="ses-driver"></a>
#### SESドライバ

Amazon SESドライバを使用するには、まずAmazon AWS SDK for PHPをインストールする必要があります。このライブラリは、Composerパッケージマネージャを介してインストールできます。

```shell
composer require aws/aws-sdk-php
```

次に、`config/mail.php`設定ファイルで`default`オプションを`ses`に設定し、`config/services.php`設定ファイルに次のオプションが含まれていることを確認してください。

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

AWS [一時的な認証情報](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)をセッショントークンを介して利用するには、アプリケーションのSES設定に`token`キーを追加できます。

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'token' => env('AWS_SESSION_TOKEN'),
    ],

SESの[サブスクリプション管理機能](https://docs.aws.amazon.com/ses/latest/dg/sending-email-subscription-management.html)と対話するには、メールメッセージの[`headers`](#headers)メソッドによって返される配列に`X-Ses-List-Management-Options`ヘッダーを返すことができます。

```php
/**
 * メッセージヘッダーを取得します。
 */
public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Ses-List-Management-Options' => 'contactListName=MyContactList;topicName=MyTopic',
        ],
    );
}
```

Laravelがメールを送信する際にAWS SDKの`SendEmail`メソッドに渡す必要がある[追加オプション](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail)を定義したい場合は、`ses`設定内に`options`配列を定義できます。

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'options' => [
            'ConfigurationSetName' => 'MyConfigurationSet',
            'EmailTags' => [
                ['Name' => 'foo', 'Value' => 'bar'],
            ],
        ],
    ],

<a name="mailersend-driver"></a>
#### MailerSendドライバ

[MailerSend](https://www.mailersend.com/)は、トランザクションメールとSMSサービスであり、Laravel用のAPIベースのメールドライバを独自に管理しています。ドライバを含むパッケージは、Composerパッケージマネージャを介してインストールできます。

```shell
composer require mailersend/laravel-driver
```

パッケージがインストールされたら、アプリケーションの`.env`ファイルに`MAILERSEND_API_KEY`環境変数を追加します。さらに、`MAIL_MAILER`環境変数を`mailersend`として定義する必要があります。

```shell
MAIL_MAILER=mailersend
MAIL_FROM_ADDRESS=app@yourdomain.com
MAIL_FROM_NAME="App Name"

MAILERSEND_API_KEY=your-api-key
```

最後に、アプリケーションの`config/mail.php`設定ファイルの`mailers`配列にMailerSendを追加します。

```php
'mailersend' => [
    'transport' => 'mailersend',
],
```

MailerSendの詳細については、ホストされたテンプレートの使用方法を含め、[MailerSendドライバのドキュメント](https://github.com/mailersend/mailersend-laravel-driver#usage)を参照してください。

<a name="failover-configuration"></a>
### フェイルオーバー設定

場合によっては、アプリケーションのメールを送信するために設定した外部サービスがダウンすることがあります。このような場合、プライマリデリバリードライバがダウンした場合に使用される1つ以上のバックアップメール配信設定を定義すると便利です。

これを実現するには、アプリケーションの`mail`設定ファイル内に`failover`トランスポートを使用するメーラーを定義する必要があります。アプリケーションの`failover`メーラーの設定配列には、設定されたメーラーが配信のために選択される順序を参照する`mailers`の配列が含まれている必要があります。


```php
    'mailers' => [
        'failover' => [
            'transport' => 'failover',
            'mailers' => [
                'postmark',
                'mailgun',
                'sendmail',
            ],
        ],

        // ...
    ],
```

フェイルオーバーメーラーを定義したら、アプリケーションの `mail` 設定ファイル内の `default` 設定キーの値としてその名前を指定することで、このメーラーをアプリケーションが使用するデフォルトのメーラーとして設定する必要があります。

```php
    'default' => env('MAIL_MAILER', 'failover'),
```

<a name="round-robin-configuration"></a>
### ラウンドロビン設定

`roundrobin` トランスポートを使用すると、複数のメーラー間でメーリングの負荷を分散できます。まず、`roundrobin` トランスポートを使用するメーラーをアプリケーションの `mail` 設定ファイル内で定義します。アプリケーションの `roundrobin` メーラーの設定配列には、配信に使用する設定済みのメーラーを参照する `mailers` の配列を含める必要があります。

```php
    'mailers' => [
        'roundrobin' => [
            'transport' => 'roundrobin',
            'mailers' => [
                'ses',
                'postmark',
            ],
        ],

        // ...
    ],
```

ラウンドロビンメーラーを定義したら、アプリケーションの `mail` 設定ファイル内の `default` 設定キーの値としてその名前を指定することで、このメーラーをアプリケーションが使用するデフォルトのメーラーとして設定する必要があります。

```php
    'default' => env('MAIL_MAILER', 'roundrobin'),
```

ラウンドロビントランスポートは、設定されたメーラーのリストからランダムにメーラーを選択し、後続の各メールに対して次に利用可能なメーラーに切り替えます。`failover` トランスポートが *[高可用性](https://en.wikipedia.org/wiki/High_availability)* を実現するのに役立つのに対し、`roundrobin` トランスポートは *[負荷分散](https://en.wikipedia.org/wiki/Load_balancing_(computing))* を提供します。

<a name="generating-mailables"></a>
## Mailableの生成

Laravelアプリケーションを構築する際、アプリケーションによって送信される各タイプのメールは "mailable" クラスとして表されます。これらのクラスは `app/Mail` ディレクトリに保存されます。このディレクトリがアプリケーションに表示されない場合でも心配はいりません。`make:mail` Artisanコマンドを使用して最初のmailableクラスを作成すると、自動的に生成されます。

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## Mailableの記述

mailableクラスを生成したら、それを開いて内容を確認しましょう。mailableクラスの設定は、`envelope`、`content`、`attachments` などのいくつかのメソッドで行われます。

`envelope` メソッドは、メッセージの件名と、場合によっては受信者を定義する `Illuminate\Mail\Mailables\Envelope` オブジェクトを返します。`content` メソッドは、メッセージの内容を生成するために使用される [Bladeテンプレート](blade.md) を定義する `Illuminate\Mail\Mailables\Content` オブジェクトを返します。

<a name="configuring-the-sender"></a>
### 送信者の設定

<a name="using-the-envelope"></a>
#### Envelopeの使用

まず、メールの送信者を設定する方法を見てみましょう。つまり、メールが「誰から」送信されるかです。送信者を設定する方法は2つあります。まず、メッセージのenvelopeに「from」アドレスを指定することができます。

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * メッセージのenvelopeを取得します。
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

必要に応じて、`replyTo` アドレスを指定することもできます。

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

<a name="using-a-global-from-address"></a>
#### グローバルな `from` アドレスの使用

ただし、アプリケーションがすべてのメールに同じ「from」アドレスを使用している場合、生成する各mailableクラスにそれを追加するのは面倒になる可能性があります。代わりに、`config/mail.php` 設定ファイルでグローバルな「from」アドレスを指定することができます。mailableクラス内で他の「from」アドレスが指定されていない場合、このアドレスが使用されます。

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

さらに、`config/mail.php` 設定ファイル内でグローバルな「reply_to」アドレスを定義することもできます。

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

<a name="configuring-the-view"></a>
### ビューの設定

mailableクラスの `content` メソッド内で、`view` を定義するか、メールの内容をレンダリングする際に使用するテンプレートを指定します。各メールは通常、その内容をレンダリングするために [Bladeテンプレート](blade.md) を使用するため、メールのHTMLを構築する際にBladeテンプレートエンジンのパワーと利便性をフルに活用できます。

```php
/**
 * メッセージの内容定義を取得します。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

> NOTE:  
> すべてのメールテンプレートを格納するために `resources/views/emails` ディレクトリを作成することをお勧めします。ただし、`resources/views` ディレクトリ内の好きな場所に配置することができます。

<a name="plain-text-emails"></a>
#### プレーンテキストメール

メールのプレーンテキストバージョンを定義したい場合は、メッセージの `Content` 定義を作成する際にプレーンテキストテンプレートを指定できます。`view` パラメータと同様に、`text` パラメータはメールの内容をレンダリングするために使用されるテンプレート名である必要があります。HTMLとプレーンテキストの両方のバージョンのメッセージを自由に定義できます。

```php
/**
 * メッセージの内容定義を取得します。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

明確にするために、`html` パラメータを `view` パラメータのエイリアスとして使用できます。

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### ビューデータ

<a name="via-public-properties"></a>
#### パブリックプロパティ経由

通常、メールのHTMLをレンダリングする際に使用できるように、ビューにいくつかのデータを渡したいと思うでしょう。データをビューで利用できるようにする方法は2つあります。まず、mailableクラスで定義されたパブリックプロパティは自動的にビューで利用できるようになります。したがって、たとえば、データをmailableクラスのコンストラクタに渡し、そのデータをクラスで定義されたパブリックプロパティに設定できます。

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 新しいメッセージインスタンスを作成します。
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * メッセージの内容定義を取得します。
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

データがパブリックプロパティに設定されると、ビューで自動的に利用できるようになります。したがって、Bladeテンプレート内で他のデータにアクセスするのと同じようにアクセスできます。

```html
<div>
    Price: {{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### `with` パラメータ経由

メールのデータの形式をテンプレートに送信する前にカスタマイズしたい場合は、`Content` 定義の `with` パラメータを介してデータを手動でビューに渡すことができます。通常、データはmailableクラスのコンストラクタを介して渡されます。ただし、このデータを `protected` または `private` プロパティに設定して、データが自動的にテンプレートで利用できないようにする必要があります。

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 新しいメッセージインスタンスを作成します。
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * メッセージの内容定義を取得します。
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```

データが `with` メソッドに渡されると、ビューで自動的に利用できるようになります。したがって、Bladeテンプレート内で他のデータにアクセスするのと同じようにアクセスできます。

```html
<div>
    Price: {{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### 添付ファイル

メールに添付ファイルを追加するには、メッセージの `attachments` メソッドによって返される配列に添付ファイルを追加します。まず、`Attachment` クラスによって提供される `fromPath` メソッドにファイルパスを指定することで、添付ファイルを追加できます。

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * メッセージの添付ファイルを取得します。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

メッセージにファイルを添付する際、`as` メソッドと `withMime` メソッドを使用して、添付ファイルの表示名やMIMEタイプを指定することもできます。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromPath('/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="attaching-files-from-disk"></a>
#### ディスクからのファイル添付

ファイルを[ファイルシステムディスク](filesystem.md)のいずれかに保存している場合、`fromStorage` 添付メソッドを使用してメールに添付することができます。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file'),
        ];
    }

もちろん、添付ファイルの名前とMIMEタイプを指定することもできます。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

デフォルトのディスク以外のストレージディスクを指定する必要がある場合は、`fromStorageDisk` メソッドを使用できます。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorageDisk('s3', '/path/to/file')
                    ->as('name.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="raw-data-attachments"></a>
#### 生データの添付

`fromData` 添付メソッドを使用して、生のバイト文字列を添付ファイルとして添付することができます。例えば、メモリ内でPDFを生成し、ディスクに書き込まずにメールに添付したい場合にこのメソッドを使用できます。`fromData` メソッドは、添付ファイルに割り当てるべき名前と共に、生データバイトを解決するクロージャを受け取ります。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
                    ->withMime('application/pdf'),
        ];
    }

<a name="inline-attachments"></a>
### インライン添付

メールにインライン画像を埋め込むことは通常、面倒ですが、Laravelはメールに画像を添付する便利な方法を提供しています。インライン画像を埋め込むには、メールテンプレート内で `$message` 変数の `embed` メソッドを使用します。Laravelは自動的に `$message` 変数をすべてのメールテンプレートで利用できるようにしているため、手動で渡す必要はありません。

```blade
<body>
    ここに画像があります:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> WARNING:  
> `$message` 変数はプレーンテキストメッセージテンプレートでは利用できません。プレーンテキストメッセージはインライン添付を使用しないためです。

<a name="embedding-raw-data-attachments"></a>
#### 生データ添付の埋め込み

すでに生の画像データ文字列を持っており、それをメールテンプレートに埋め込みたい場合は、`$message` 変数の `embedData` メソッドを呼び出すことができます。`embedData` メソッドを呼び出す際には、埋め込み画像に割り当てるべきファイル名を指定する必要があります。

```blade
<body>
    ここに生データからの画像があります:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### 添付可能なオブジェクト

ファイルを添付する際に単純な文字列パスを使用することはしばしば十分ですが、多くの場合、アプリケーション内の添付可能なエンティティはクラスによって表現されます。例えば、アプリケーションがメッセージに写真を添付する場合、アプリケーションにはその写真を表す `Photo` モデルがあるかもしれません。その場合、`Photo` モデルを `attach` メソッドに渡すだけで便利ではないでしょうか？添付可能なオブジェクトを使用すると、それが可能になります。

まず、メッセージに添付可能なオブジェクトに `Illuminate\Contracts\Mail\Attachable` インターフェースを実装します。このインターフェースは、クラスが `Illuminate\Mail\Attachment` インスタンスを返す `toMailAttachment` メソッドを定義することを要求します。

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Mail\Attachable;
    use Illuminate\Database\Eloquent\Model;
    use Illuminate\Mail\Attachment;

    class Photo extends Model implements Attachable
    {
        /**
         * モデルの添付可能な表現を取得する。
         */
        public function toMailAttachment(): Attachment
        {
            return Attachment::fromPath('/path/to/file');
        }
    }

添付可能なオブジェクトを定義したら、メールメッセージを構築する際に `attachments` メソッドからそのオブジェクトのインスタンスを返すことができます。

    /**
     * メッセージの添付ファイルを取得する。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [$this->photo];
    }

もちろん、添付データはAmazon S3などのリモートファイルストレージサービスに保存されている場合があります。そのため、Laravelはアプリケーションの[ファイルシステムディスク](filesystem.md)に保存されているデータから添付ファイルインスタンスを生成することも可能です。

    // デフォルトディスク上のファイルから添付ファイルを作成する...
    return Attachment::fromStorage($this->path);

    // 特定のディスク上のファイルから添付ファイルを作成する...
    return Attachment::fromStorageDisk('backblaze', $this->path);

さらに、メモリ内にあるデータを介して添付ファイルインスタンスを作成することもできます。これを行うには、`fromData` メソッドにクロージャを提供します。クロージャは添付ファイルを表す生データを返す必要があります。

    return Attachment::fromData(fn () => $this->content, 'Photo Name');

Laravelはまた、添付ファイルをカスタマイズするために使用できる追加のメソッドを提供しています。例えば、`as` メソッドと `withMime` メソッドを使用して、ファイルの名前とMIMEタイプをカスタマイズできます。

    return Attachment::fromPath('/path/to/file')
            ->as('Photo Name')
            ->withMime('image/jpeg');

<a name="headers"></a>
### ヘッダー

送信メッセージに追加のヘッダーを添付する必要がある場合があります。例えば、カスタムの `Message-Id` やその他の任意のテキストヘッダーを設定する必要がある場合です。

これを実現するには、mailableに `headers` メソッドを定義します。`headers` メソッドは `Illuminate\Mail\Mailables\Headers` インスタンスを返す必要があります。このクラスは `messageId`、`references`、`text` パラメータを受け取ります。もちろん、特定のメッセージに必要なパラメータのみを提供することができます。

    use Illuminate\Mail\Mailables\Headers;

    /**
     * メッセージのヘッダーを取得する。
     */
    public function headers(): Headers
    {
        return new Headers(
            messageId: 'custom-message-id@example.com',
            references: ['previous-message@example.com'],
            text: [
                'X-Custom-Header' => 'Custom Value',
            ],
        );
    }

<a name="tags-and-metadata"></a>
### タグとメタデータ

MailgunやPostmarkなどの一部のサードパーティメールプロバイダは、メッセージの「タグ」と「メタデータ」をサポートしており、アプリケーションから送信されたメールをグループ化して追跡するために使用できます。`Envelope` 定義を介してメールメッセージにタグとメタデータを追加できます。

    use Illuminate\Mail\Mailables\Envelope;

    /**
     * メッセージのエンベロープを取得する。
     *
     * @return \Illuminate\Mail\Mailables\Envelope
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Order Shipped',
            tags: ['shipment'],
            metadata: [
                'order_id' => $this->order->id,
            ],
        );
    }

アプリケーションがMailgunドライバを使用している場合は、Mailgunのドキュメントを参照して、[タグ](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1)と[メタデータ](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages)に関する詳細情報を確認できます。同様に、Postmarkのドキュメントも、[タグ](https://postmarkapp.com/blog/tags-support-for-smtp)と[メタデータ](https://postmarkapp.com/support/article/1125-custom-metadata-faq)のサポートに関する詳細情報を確認できます。

アプリケーションがAmazon SESを使用してメールを送信している場合は、`metadata` メソッドを使用してメッセージに[SES「タグ」](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html)を添付する必要があります。

<a name="customizing-the-symfony-message"></a>
### Symfonyメッセージのカスタマイズ

Laravelのメール機能はSymfony Mailerによって提供されています。Laravelでは、メッセージを送信する前にSymfonyメッセージインスタンスで呼び出されるカスタムコールバックを登録できます。これにより、メッセージを送信する前に深くカスタマイズする機会が得られます。これを実現するには、`Envelope` 定義に `using` パラメータを定義します。

    use Illuminate\Mail\Mailables\Envelope;
    use Symfony\Component\Mime\Email;

    /**
     * メッセージのエンベロープを取得する。
     */
    public function envelope(): Envelope
    {
        return new Envelope(
            subject: 'Order Shipped',
            using: [
                function (Email $message) {
                    // ...
                },
            ]
        );
    }

<a name="markdown-mailables"></a>
## Markdownメール

Markdownメールは、定義済みのテンプレートとメール通知のコンポーネントを利用できるため、メールメッセージを簡単に作成できます。Markdownを使用してメッセージを記述すると、Laravelは美しくレスポンシブなHTMLテンプレートをレンダリングし、同時にプレーンテキストバージョンを自動的に生成します。

Markdown形式のメールメッセージを使用すると、[メール通知](notifications.md#mail-notifications)の事前に構築されたテンプレートとコンポーネントをメールで利用できます。メッセージはMarkdownで記述されているため、Laravelはメッセージのために美しくレスポンシブなHTMLテンプレートをレンダリングするだけでなく、プレーンテキストの対応版も自動的に生成します。

<a name="generating-markdown-mailables"></a>
### Markdown Mailablesの生成

Markdownテンプレートに対応するmailableを生成するには、`make:mail` Artisanコマンドの`--markdown`オプションを使用できます。

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

次に、mailableの`content`メソッド内で`content`定義を設定する際に、`view`パラメータの代わりに`markdown`パラメータを使用します。

    use Illuminate\Mail\Mailables\Content;

    /**
     * メッセージのコンテンツ定義を取得する。
     */
    public function content(): Content
    {
        return new Content(
            markdown: 'mail.orders.shipped',
            with: [
                'url' => $this->orderUrl,
            ],
        );
    }

<a name="writing-markdown-messages"></a>
### Markdownメッセージの記述

Markdown形式のメールは、BladeコンポーネントとMarkdown構文を組み合わせて使用し、Laravelの事前に構築されたメールUIコンポーネントを活用しながら、メールメッセージを簡単に構築できます。

```blade
<x-mail::message>
# 注文出荷

あなたの注文が出荷されました！

<x-mail::button :url="$url">
注文を見る
</x-mail::button>

ありがとうございます,<br>
{{ config('app.name') }}
</x-mail::message>
```

> NOTE:  
> Markdownメールを記述する際には、過剰なインデントを使用しないでください。Markdown標準に従い、Markdownパーサーはインデントされたコンテンツをコードブロックとしてレンダリングします。

<a name="button-component"></a>
#### ボタンコンポーネント

ボタンコンポーネントは、中央揃えのボタンリンクをレンダリングします。このコンポーネントは、`url`とオプションの`color`の2つの引数を受け取ります。サポートされている色は`primary`、`success`、`error`です。メッセージには必要なだけボタンコンポーネントを追加できます。

```blade
<x-mail::button :url="$url" color="success">
注文を見る
</x-mail::button>
```

<a name="panel-component"></a>
#### パネルコンポーネント

パネルコンポーネントは、メッセージの残りの部分とは少し異なる背景色を持つパネル内に指定されたテキストブロックをレンダリングします。これにより、特定のテキストブロックに注意を引くことができます。

```blade
<x-mail::panel>
これはパネルの内容です。
</x-mail::panel>
```

<a name="table-component"></a>
#### テーブルコンポーネント

テーブルコンポーネントを使用すると、MarkdownテーブルをHTMLテーブルに変換できます。このコンポーネントは、Markdownテーブルをコンテンツとして受け取ります。テーブルの列の配置は、デフォルトのMarkdownテーブル配置構文を使用してサポートされています。

```blade
<x-mail::table>
| Laravel       | テーブル         | 例             |
| ------------- | :-----------: | ------------: |
| 列2は         | 中央揃え        | $10           |
| 列3は         | 右揃え          | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### コンポーネントのカスタマイズ

Markdownメールコンポーネントをすべて自分のアプリケーションにエクスポートしてカスタマイズすることができます。コンポーネントをエクスポートするには、`vendor:publish` Artisanコマンドを使用して`laravel-mail`アセットタグを公開します。

```shell
php artisan vendor:publish --tag=laravel-mail
```

このコマンドは、Markdownメールコンポーネントを`resources/views/vendor/mail`ディレクトリに公開します。`mail`ディレクトリには、`html`と`text`のディレクトリが含まれ、それぞれに利用可能なすべてのコンポーネントの表現が含まれています。これらのコンポーネントは自由にカスタマイズできます。

<a name="customizing-the-css"></a>
#### CSSのカスタマイズ

コンポーネントをエクスポートした後、`resources/views/vendor/mail/html/themes`ディレクトリに`default.css`ファイルが含まれます。このファイルのCSSをカスタマイズすると、MarkdownメールメッセージのHTML表現内のインラインCSSスタイルに自動的に変換されます。

LaravelのMarkdownコンポーネント用にまったく新しいテーマを構築したい場合は、`html/themes`ディレクトリ内にCSSファイルを配置できます。CSSファイルに名前を付けて保存した後、アプリケーションの`config/mail.php`設定ファイルの`theme`オプションを新しいテーマの名前と一致するように更新します。

個々のmailableのテーマをカスタマイズするには、そのmailableクラスの`$theme`プロパティを、そのmailableを送信する際に使用するテーマの名前に設定できます。

<a name="sending-mail"></a>
## メールの送信

メッセージを送信するには、`Mail` [ファサード](facades.md)の`to`メソッドを使用します。`to`メソッドは、メールアドレス、ユーザーインスタンス、またはユーザーのコレクションを受け取ります。オブジェクトまたはオブジェクトのコレクションを渡すと、メーラーはメールの受信者を決定する際に自動的にそれらの`email`および`name`プロパティを使用するため、これらの属性がオブジェクトで利用可能であることを確認してください。受信者を指定したら、mailableクラスのインスタンスを`send`メソッドに渡すことができます。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Mail\OrderShipped;
    use App\Models\Order;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Mail;

    class OrderShipmentController extends Controller
    {
        /**
         * 指定された注文を出荷する。
         */
        public function store(Request $request): RedirectResponse
        {
            $order = Order::findOrFail($request->order_id);

            // 注文を出荷する...

            Mail::to($request->user())->send(new OrderShipped($order));

            return redirect('/orders');
        }
    }

メッセージを送信する際に、"to"受信者のみを指定することに限定されません。"to"、"cc"、"bcc"受信者を自由に設定するために、それぞれのメソッドをチェーンできます。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->send(new OrderShipped($order));

<a name="looping-over-recipients"></a>
#### 受信者をループで処理する

場合によっては、受信者/メールアドレスのリストを反復処理してmailableを送信する必要があります。しかし、`to`メソッドはmailableの受信者リストにメールアドレスを追加するため、ループの各反復で以前のすべての受信者に別のメールが送信されます。したがって、各受信者に対して常に新しいmailableインスタンスを再作成する必要があります。

    foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
        Mail::to($recipient)->send(new OrderShipped($order));
    }

<a name="sending-mail-via-a-specific-mailer"></a>
#### 特定のメーラー経由でメールを送信する

デフォルトでは、Laravelはアプリケーションの`mail`設定ファイルで`default`として設定されたメーラーを使用してメールを送信します。しかし、`mailer`メソッドを使用して、特定のメーラー設定を使用してメッセージを送信できます。

    Mail::mailer('postmark')
            ->to($request->user())
            ->send(new OrderShipped($order));

<a name="queueing-mail"></a>
### メールのキューイング

<a name="queueing-a-mail-message"></a>
#### メールメッセージのキューイング

メールメッセージの送信はアプリケーションのレスポンス時間に悪影響を与える可能性があるため、多くの開発者はメールメッセージをバックグラウンドで送信するためにキューイングします。Laravelは、組み込みの[統一キューAPI](queues.md)を使用してこれを簡単にします。メッセージの受信者を指定した後、`Mail`ファサードの`queue`メソッドを使用してメッセージをキューに入れることができます。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue(new OrderShipped($order));

このメソッドは、メッセージがバックグラウンドで送信されるように、ジョブをキューに自動的にプッシュします。この機能を使用する前に、[キューを設定](queues.md)する必要があります。

<a name="delayed-message-queueing"></a>
#### 遅延メッセージキューイング

キューに入れられたメールメッセージの配信を遅延させたい場合は、`later`メソッドを使用できます。`later`メソッドの最初の引数は、メッセージを送信する時期を示す`DateTime`インスタンスを受け取ります。

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->later(now()->addMinutes(10), new OrderShipped($order));

<a name="pushing-to-specific-queues"></a>
#### 特定のキューにプッシュする

`make:mail`コマンドを使用して生成されたすべてのmailableクラスは、`Illuminate\Bus\Queueable`トレイトを使用しているため、任意のmailableクラスインスタンスで`onQueue`および`onConnection`メソッドを呼び出して、メッセージの接続とキュー名を指定できます。

    $message = (new OrderShipped($order))
                    ->onConnection('sqs')
                    ->onQueue('emails');

    Mail::to($request->user())
        ->cc($moreUsers)
        ->bcc($evenMoreUsers)
        ->queue($message);

<a name="queueing-by-default"></a>
#### デフォルトでキューイングする

常にキューに入れたいmailableクラスがある場合は、クラスに`ShouldQueue`コントラクトを実装できます。これで、メールを送信する際に`send`メソッドを呼び出しても、mailableはキューに入れられます。

    use Illuminate\Contracts\Queue\ShouldQueue;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        // ...
    }

<a name="queued-mailables-and-database-transactions"></a>
#### キューに入れられたMailablesとデータベーストランザクション

データベーストランザクション内でキューに入れられたメールがディスパッチされると、データベーストランザクションがコミットされる前にキューによって処理される可能性があります。この場合、データベーストランザクション中にモデルやデータベースレコードに加えた更新がまだデータベースに反映されていない可能性があります。また、トランザクション内で作成されたモデルやデータベースレコードがデータベースに存在しない可能性もあります。メールがこれらのモデルに依存している場合、キューに入れられたメールを送信するジョブが処理される際に予期しないエラーが発生する可能性があります。

キュー接続の `after_commit` 設定オプションが `false` に設定されている場合でも、メールメッセージを送信する際に `afterCommit` メソッドを呼び出すことで、すべての開いているデータベーストランザクションがコミットされた後に特定のキューに入れられたメールをディスパッチするように指示できます。

    Mail::to($request->user())->send(
        (new OrderShipped($order))->afterCommit()
    );

または、メールのコンストラクタから `afterCommit` メソッドを呼び出すこともできます。

    <?php

    namespace App\Mail;

    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Mail\Mailable;
    use Illuminate\Queue\SerializesModels;

    class OrderShipped extends Mailable implements ShouldQueue
    {
        use Queueable, SerializesModels;

        /**
         * 新しいメッセージインスタンスを作成する。
         */
        public function __construct()
        {
            $this->afterCommit();
        }
    }

> NOTE:  
> これらの問題を回避する方法について詳しく知りたい場合は、[キューされたジョブとデータベーストランザクション](queues.md#jobs-and-database-transactions)に関するドキュメントを確認してください。

<a name="rendering-mailables"></a>
## メールのレンダリング

メールを送信せずにメールのHTMLコンテンツをキャプチャしたい場合があります。これを実現するには、メールの `render` メソッドを呼び出します。このメソッドは、メールの評価済みHTMLコンテンツを文字列として返します。

    use App\Mail\InvoicePaid;
    use App\Models\Invoice;

    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### ブラウザでのメールのプレビュー

メールのテンプレートを設計する際、通常のBladeテンプレートのようにブラウザでレンダリングされたメールをすばやくプレビューすると便利です。このため、Laravelでは、ルートクロージャまたはコントローラから直接メールを返すことができます。メールが返されると、ブラウザにレンダリングされて表示され、実際のメールアドレスに送信する必要なくデザインをすばやくプレビューできます。

    Route::get('/mailable', function () {
        $invoice = App\Models\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="localizing-mailables"></a>
## メールのローカライズ

Laravelでは、リクエストの現在のロケール以外のロケールでメールを送信できます。メールがキューに入れられている場合でも、このロケールを記憶します。

これを実現するために、`Mail` ファサードは、希望する言語を設定するための `locale` メソッドを提供します。メールのテンプレートが評価されると、アプリケーションはこのロケールに変更され、評価が完了すると前のロケールに戻ります。

    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

<a name="user-preferred-locales"></a>
### ユーザーの優先ロケール

アプリケーションは、各ユーザーの優先ロケールを保存する場合があります。1つ以上のモデルに `HasLocalePreference` コントラクトを実装することで、Laravelにメール送信時にこの保存されたロケールを使用するよう指示できます。

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * ユーザーの優先ロケールを取得する。
         */
        public function preferredLocale(): string
        {
            return $this->locale;
        }
    }

このインターフェースを実装すると、Laravelはメールと通知をモデルに送信する際に自動的に優先ロケールを使用します。したがって、このインターフェースを使用する場合、`locale` メソッドを呼び出す必要はありません。

    Mail::to($request->user())->send(new OrderShipped($order));

<a name="testing-mailables"></a>
## テスト

<a name="testing-mailable-content"></a>
### メールの内容のテスト

Laravelは、メールの構造を検査するためのさまざまなメソッドを提供します。さらに、Laravelは、メールに期待するコンテンツが含まれていることをテストするための便利なメソッドを提供します。これらのメソッドは、`assertSeeInHtml`、`assertDontSeeInHtml`、`assertSeeInOrderInHtml`、`assertSeeInText`、`assertDontSeeInText`、`assertSeeInOrderInText`、`assertHasAttachment`、`assertHasAttachedData`、`assertHasAttachmentFromStorage`、および `assertHasAttachmentFromStorageDisk` です。

予想されるように、「HTML」アサーションは、メールのHTMLバージョンに特定の文字列が含まれていることをアサートし、「テキスト」アサーションは、メールのプレーンテキストバージョンに特定の文字列が含まれていることをアサートします。

===  "Pest"
```php
use App\Mail\InvoicePaid;
use App\Models\User;

test('mailable content', function () {
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertSeeInHtml('Invoice Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
});
```

===  "PHPUnit"
```php
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content(): void
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertSeeInHtml('Invoice Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}
```

<a name="testing-mailable-sending"></a>
### メールの送信のテスト

メールの内容を別々にテストすることをお勧めします。特定のメールが特定のユーザーに「送信」されたことをアサートするテストとは別にテストします。通常、メールの内容はテストしているコードには関係ありません。Laravelに特定のメールを送信するよう指示されたことをアサートするだけで十分です。

`Mail` ファサードの `fake` メソッドを使用して、メールが送信されないようにすることができます。`Mail` ファサードの `fake` メソッドを呼び出した後、メールがユーザーに送信されるように指示されたことをアサートし、メールが受け取ったデータを検査することもできます。

===  "Pest"
```php
<?php

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

test('orders can be shipped', function () {
    Mail::fake();

    // 注文の発送を実行...

    // メールが送信されなかったことをアサート...
    Mail::assertNothingSent();

    // メールが送信されたことをアサート...
    Mail::assertSent(OrderShipped::class);

    // メールが2回送信されたことをアサート...
    Mail::assertSent(OrderShipped::class, 2);

    // メールがメールアドレスに送信されたことをアサート...
    Mail::assertSent(OrderShipped::class, 'example@laravel.com');

    // メールが複数のメールアドレスに送信されたことをアサート...
    Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

    // メールが送信されなかったことをアサート...
    Mail::assertNotSent(AnotherMailable::class);

    // 合計3通のメールが送信されたことをアサート...
    Mail::assertSentCount(3);
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // 注文の発送を実行...

        // メールが送信されなかったことをアサート...
        Mail::assertNothingSent();

        // メールが送信されたことをアサート...
        Mail::assertSent(OrderShipped::class);

        // メールが2回送信されたことをアサート...
        Mail::assertSent(OrderShipped::class, 2);

        // メールがメールアドレスに送信されたことをアサート...
        Mail::assertSent(OrderShipped::class, 'example@laravel.com');

        // メールが複数のメールアドレスに送信されたことをアサート...
        Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

        // メールが送信されなかったことをアサート...
        Mail::assertNotSent(AnotherMailable::class);

        // 合計3通のメールが送信されたことをアサート...
        Mail::assertSentCount(3);
    }
}
```

メールをバックグラウンドで配信するためにキューに入れている場合は、`assertSent`の代わりに`assertQueued`メソッドを使用する必要があります。

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

`assertSent`、`assertNotSent`、`assertQueued`、または`assertNotQueued`メソッドにクロージャを渡して、特定の「真実テスト」をパスするメールが送信されたことをアサートすることができます。少なくとも1通のメールが指定された真実テストをパスした場合、アサーションは成功します。

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

`Mail`ファサードのアサーションメソッドを呼び出す際、提供されたクロージャによって受け入れられるメールインスタンスは、メールを調査するための便利なメソッドを公開しています。

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...');
});
```

メールインスタンスには、メールに添付されたファイルを調査するためのいくつかの便利なメソッドも含まれています。

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/path/to/file')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});
```

メールが送信されなかったことをアサートするための2つのメソッド、`assertNotSent`と`assertNotQueued`があることに気づいたかもしれません。時には、メールが送信**または**キューに入れられなかったことをアサートしたい場合があります。これを実現するために、`assertNothingOutgoing`と`assertNotOutgoing`メソッドを使用できます。

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## メールとローカル開発

メールを送信するアプリケーションを開発する際、実際のメールアドレスにメールを送信したくない場合があります。Laravelは、ローカル開発中にメールの実際の送信を「無効化」するいくつかの方法を提供しています。

<a name="log-driver"></a>
#### ログドライバ

メールの代わりに、`log`メールドライバはすべてのメールメッセージをログファイルに書き込みます。通常、このドライバはローカル開発中にのみ使用されます。環境ごとにアプリケーションを設定する方法の詳細については、[設定ドキュメント](configuration.md#environment-configuration)を確認してください。

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

または、[HELO](https://usehelo.com)や[Mailtrap](https://mailtrap.io)のようなサービスと`smtp`ドライバを使用して、メールメッセージを「ダミー」のメールボックスに送信し、真のメールクライアントで表示することもできます。このアプローチには、Mailtrapのメッセージビューアで実際のメールを確認できるという利点があります。

[Laravel Sail](sail.md)を使用している場合、[Mailpit](https://github.com/axllent/mailpit)を使用してメッセージをプレビューできます。Sailが実行中の場合、Mailpitインターフェースには以下のURLからアクセスできます: `http://localhost:8025`。

<a name="using-a-global-to-address"></a>
#### グローバルな`to`アドレスの使用

最後に、`Mail`ファサードが提供する`alwaysTo`メソッドを呼び出して、グローバルな「to」アドレスを指定することができます。通常、このメソッドはアプリケーションのサービスプロバイダの`boot`メソッドから呼び出す必要があります。

```php
use Illuminate\Support\Facades\Mail;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}
```

<a name="events"></a>
## イベント

Laravelは、メールメッセージを送信する際に2つのイベントをディスパッチします。`MessageSending`イベントはメッセージが送信される前にディスパッチされ、`MessageSent`イベントはメッセージが送信された後にディスパッチされます。これらのイベントは、メールが*送信*されるときにディスパッチされ、キューに入れられるときではありません。これらのイベントの[イベントリスナー](events.md)をアプリケーション内で作成できます。

```php
use Illuminate\Mail\Events\MessageSending;
// use Illuminate\Mail\Events\MessageSent;

class LogMessage
{
    /**
     * 指定されたイベントを処理します。
     */
    public function handle(MessageSending $event): void
    {
        // ...
    }
}
```

<a name="custom-transports"></a>
## カスタムトランスポート

Laravelにはさまざまなメールトランスポートが含まれていますが、Laravelがデフォルトでサポートしていない他のサービスを介してメールを送信するために独自のトランスポートを作成したい場合があります。これを実現するために、`Symfony\Component\Mailer\Transport\AbstractTransport`クラスを拡張するクラスを定義します。次に、トランスポートの`doSend`メソッドと`__toString()`メソッドを実装します。

```php
use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * 新しいMailchimpトランスポートインスタンスを作成します。
     */
    public function __construct(
        protected ApiClient $client,
    ) {
        parent::__construct();
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function (Address $email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * トランスポートの文字列表現を取得します。
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

カスタムトランスポートを定義したら、`Mail`ファサードが提供する`extend`メソッドを介して登録できます。通常、これはアプリケーションの`AppServiceProvider`サービスプロバイダの`boot`メソッド内で行う必要があります。`extend`メソッドに提供されるクロージャには、アプリケーションの`config/mail.php`設定ファイルでメーラーに対して定義された設定配列を含む`$config`引数が渡されます。

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        return new MailchimpTransport(/* ... */);
    });
}
```

カスタムトランスポートを定義して登録したら、アプリケーションの`config/mail.php`設定ファイル内に新しいトランスポートを使用するメーラー定義を作成できます。

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    // ...
],
```

<a name="additional-symfony-transports"></a>
### 追加のSymfonyトランスポート

Laravelは、MailgunやPostmarkなどの既存のSymfony管理のメールトランスポートをサポートしています。ただし、Laravelを追加のSymfony管理のトランスポートで拡張したい場合があります。これを行うには、必要なSymfonyメーラーをComposerでインストールし、Laravelにトランスポートを登録します。例えば、「Brevo」（旧「Sendinblue」）Symfonyメーラーをインストールして登録します。

```none
composer require symfony/brevo-mailer symfony/http-client
```

Brevoメーラーパッケージをインストールしたら、アプリケーションの`services`設定ファイルにBrevo API資格情報のエントリを追加します。

```php
'brevo' => [
    'key' => 'your-api-key',
],
```

次に、`Mail`ファサードの`extend`メソッドを使用して、Laravelにトランスポートを登録します。通常、これはサービスプロバイダの`boot`メソッド内で行う必要があります。

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Mail::extend('brevo', function () {
        return (new BrevoTransportFactory)->create(
            new Dsn(
                'brevo+api',
                'default',
                config('services.brevo.key')
            )
        );
    });
}
```

トランスポートを登録したら、アプリケーションの`config/mail.php`設定ファイル内に新しいトランスポートを使用するメーラー定義を作成できます。

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```

