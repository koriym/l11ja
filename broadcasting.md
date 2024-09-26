# ブロードキャスト

- [はじめに](#introduction)
- [サーバーサイドのインストール](#server-side-installation)
    - [設定](#configuration)
    - [Reverb](#reverb)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
- [クライアントサイドのインストール](#client-side-installation)
    - [Reverb](#client-reverb)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [概念の概要](#concept-overview)
    - [サンプルアプリケーションの使用](#using-example-application)
- [ブロードキャストイベントの定義](#defining-broadcast-events)
    - [ブロードキャスト名](#broadcast-name)
    - [ブロードキャストデータ](#broadcast-data)
    - [ブロードキャストキュー](#broadcast-queue)
    - [ブロードキャスト条件](#broadcast-conditions)
    - [ブロードキャストとデータベーストランザクション](#broadcasting-and-database-transactions)
- [チャンネルの認可](#authorizing-channels)
    - [認可コールバックの定義](#defining-authorization-callbacks)
    - [チャンネルクラスの定義](#defining-channel-classes)
- [イベントのブロードキャスト](#broadcasting-events)
    - [他のユーザーのみ](#only-to-others)
    - [接続のカスタマイズ](#customizing-the-connection)
    - [匿名イベント](#anonymous-events)
- [ブロードキャストの受信](#receiving-broadcasts)
    - [イベントのリスニング](#listening-for-events)
    - [チャンネルからの退出](#leaving-a-channel)
    - [名前空間](#namespaces)
- [プレゼンスチャンネル](#presence-channels)
    - [プレゼンスチャンネルの認可](#authorizing-presence-channels)
    - [プレゼンスチャンネルへの参加](#joining-presence-channels)
    - [プレゼンスチャンネルへのブロードキャスト](#broadcasting-to-presence-channels)
- [モデルのブロードキャスト](#model-broadcasting)
    - [モデルブロードキャストの規約](#model-broadcasting-conventions)
    - [モデルブロードキャストのリスニング](#listening-for-model-broadcasts)
- [クライアントイベント](#client-events)
- [通知](#notifications)

<a name="introduction"></a>
## はじめに

多くの現代のWebアプリケーションでは、リアルタイムでライブアップデートされるユーザーインターフェースを実装するためにWebSocketsが使用されています。サーバー上のデータが更新されると、通常、WebSocket接続を介してメッセージが送信され、クライアントによって処理されます。WebSocketsは、UIに反映されるべきデータの変更を反映するために、アプリケーションのサーバーに継続的にポーリングするよりも効率的な代替手段を提供します。

例えば、アプリケーションがユーザーのデータをCSVファイルにエクスポートし、メールで送信することができると想像してください。しかし、このCSVファイルの作成には数分かかるため、[キュージョブ](queues.md)内でCSVの作成とメール送信を行うことにします。CSVが作成され、ユーザーにメールで送信されたとき、イベントブロードキャストを使用して`App\Events\UserDataExported`イベントをディスパッチし、アプリケーションのJavaScriptで受信できます。イベントが受信されると、ユーザーにCSVがメールで送信されたことをメッセージで表示し、ページをリフレッシュする必要がなくなります。

このような機能を構築するのを支援するために、LaravelはサーバーサイドのLaravel [イベント](events.md)をWebSocket接続を介して「ブロードキャスト」することを容易にします。Laravelイベントのブロードキャストにより、サーバーサイドのLaravelアプリケーションとクライアントサイドのJavaScriptアプリケーション間で同じイベント名とデータを共有できます。

ブロードキャストの背後にある中心的な概念はシンプルです：クライアントはフロントエンドで名前付きチャンネルに接続し、Laravelアプリケーションはバックエンドでこれらのチャンネルにイベントをブロードキャストします。これらのイベントには、フロントエンドで利用可能にしたい追加のデータを含めることができます。

<a name="supported-drivers"></a>
#### サポートされているドライバ

デフォルトで、Laravelには3つのサーバーサイドのブロードキャストドライバが含まれています：[Laravel Reverb](https://reverb.laravel.com)、[Pusher Channels](https://pusher.com/channels)、および[Ably](https://ably.com)。

> NOTE:  
> イベントブロードキャストに入る前に、Laravelの[イベントとリスナー](events.md)のドキュメントを読むことを確認してください。

<a name="server-side-installation"></a>
## サーバーサイドのインストール

Laravelのイベントブロードキャストを使用するためには、Laravelアプリケーション内でいくつかの設定を行い、いくつかのパッケージをインストールする必要があります。

イベントブロードキャストは、サーバーサイドのブロードキャストドライバによって実現され、Laravelイベントをブロードキャストして、ブラウザクライアント内で受信できるようにします。心配しないでください。インストールプロセスの各部分を段階的に説明します。

<a name="configuration"></a>
### 設定

アプリケーションのすべてのイベントブロードキャスト設定は、`config/broadcasting.php`設定ファイルに保存されます。このディレクトリがアプリケーションに存在しない場合でも心配しないでください。`install:broadcasting` Artisanコマンドを実行すると作成されます。

Laravelはいくつかのブロードキャストドライバをデフォルトでサポートしています：[Laravel Reverb](reverb.md)、[Pusher Channels](https://pusher.com/channels)、[Ably](https://ably.com)、およびローカル開発とデバッグ用の`log`ドライバ。さらに、テスト中にブロードキャストを無効にするための`null`ドライバも含まれています。これらのドライバの設定例は、`config/broadcasting.php`設定ファイルに含まれています。

<a name="installation"></a>
#### インストール

デフォルトでは、ブロードキャストは新しいLaravelアプリケーションでは有効になっていません。`install:broadcasting` Artisanコマンドを使用してブロードキャストを有効にできます：

```shell
php artisan install:broadcasting
```

`install:broadcasting`コマンドは、`config/broadcasting.php`設定ファイルを作成します。さらに、アプリケーションのブロードキャスト認可ルートとコールバックを登録できる`routes/channels.php`ファイルを作成します。

<a name="queue-configuration"></a>
#### キュー設定

イベントをブロードキャストする前に、最初に[キューワーカー](queues.md)を設定して実行する必要があります。すべてのイベントブロードキャストはキュージョブを介して行われるため、アプリケーションの応答時間はイベントのブロードキャストによって深刻に影響を受けません。

<a name="reverb"></a>
### Reverb

`install:broadcasting`コマンドを実行すると、[Laravel Reverb](reverb.md)のインストールを求められます。もちろん、Composerパッケージマネージャを使用して手動でReverbをインストールすることもできます。

```sh
composer require laravel/reverb
```

パッケージがインストールされたら、Reverbのインストールコマンドを実行して設定を公開し、Reverbの必要な環境変数を追加し、アプリケーションでイベントブロードキャストを有効にできます：

```sh
php artisan reverb:install
```

Reverbのインストールと使用方法の詳細な手順は、[Reverbドキュメント](reverb.md)で確認できます。

<a name="pusher-channels"></a>
### Pusher Channels

イベントを[Pusher Channels](https://pusher.com/channels)を使用してブロードキャストする場合、Composerパッケージマネージャを使用してPusher Channels PHP SDKをインストールする必要があります：

```shell
composer require pusher/pusher-php-server
```

次に、`config/broadcasting.php`設定ファイルでPusher Channelsの認証情報を設定する必要があります。このファイルには、キー、シークレット、アプリケーションIDをすばやく指定できるPusher Channelsの設定例が既に含まれています。通常、Pusher Channelsの認証情報はアプリケーションの`.env`ファイルで設定する必要があります：

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```

`config/broadcasting.php`ファイルの`pusher`設定では、クラスタなど、Channelsでサポートされている追加の`options`を指定することもできます。

次に、アプリケーションの`.env`ファイルで`BROADCAST_CONNECTION`環境変数を`pusher`に設定します：

```ini
BROADCAST_CONNECTION=pusher
```

最後に、クライアントサイドでブロードキャストイベントを受信するために、[Laravel Echo](#client-side-installation)をインストールして設定する準備が整いました。

<a name="ably"></a>
### Ably

> NOTE:  
> 以下のドキュメントでは、Ablyを「Pusher互換」モードで使用する方法について説明します。ただし、Ablyチームは、Ablyが提供する独自の機能を活用できるブロードキャスタとEchoクライアントを推奨しています。Ablyが提供するドライバの使用に関する詳細は、[AblyのLaravelブロードキャスタドキュメント](https://github.com/ably/laravel-broadcaster)を参照してください。

イベントを[Ably](https://ably.com)を使用してブロードキャストする場合、Composerパッケージマネージャを使用してAbly PHP SDKをインストールする必要があります：

```shell
composer require ably/ably-php
```

次に、`config/broadcasting.php`設定ファイルでAblyの認証情報を設定する必要があります。このファイルには、キーをすばやく指定できるAblyの設定例が既に含まれています。通常、この値は`ABLY_KEY`[環境変数](configuration.md#environment-configuration)を介して設定する必要があります：

```ini
ABLY_KEY=your-ably-key
```

次に、アプリケーションの`.env`ファイルで`BROADCAST_CONNECTION`環境変数を`ably`に設定します：

```ini
BROADCAST_CONNECTION=ably
```

最後に、クライアントサイドでブロードキャストイベントを受信するために、[Laravel Echo](#client-side-installation)をインストールして設定する準備が整いました。

<a name="client-side-installation"></a>
## クライアントサイドのインストール

<a name="client-reverb"></a>
### Reverb

[Laravel Echo](https://github.com/laravel/echo)は、チャンネルを購読し、サーバーサイドのブロードキャストドライバによってブロードキャストされたイベントをリスニングすることを容易にするJavaScriptライブラリです。NPMパッケージマネージャを介してEchoをインストールできます。この例では、ReverbはWebSocketのサブスクリプション、チャンネル、メッセージにPusherプロトコルを使用するため、`pusher-js`パッケージもインストールします：

```shell
npm install --save-dev laravel-echo pusher-js
```

Echoがインストールされたら、アプリケーションのJavaScriptで新しいEchoインスタンスを作成する準備が整いました。これを行う良い場所は、Laravelフレームワークに含まれている`resources/js/bootstrap.js`ファイルの最後です。デフォルトでは、Echoの設定例がこのファイルに既に含まれています - 単にそれをアンコメントし、`broadcaster`設定オプションを`reverb`に更新するだけです：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT,
    wssPort: import.meta.env.VITE_REVERB_PORT,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

次に、アプリケーションのアセットをコンパイルする必要があります：

```shell
npm run build
```

> WARNING:  
> Laravel Echoの`reverb`ブロードキャスターにはlaravel-echo v1.16.0以上が必要です。

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo)は、サーバーサイドのブロードキャストドライバによってブロードキャストされるイベントを購読し、リッスンするのを容易にするJavaScriptライブラリです。Echoはまた、WebSocketのサブスクリプション、チャンネル、メッセージのためにPusherプロトコルを実装するために`pusher-js` NPMパッケージを活用します。

`install:broadcasting` Artisanコマンドは自動的に`laravel-echo`と`pusher-js`パッケージをインストールしますが、これらのパッケージは手動でNPMを介してインストールすることもできます：

```shell
npm install --save-dev laravel-echo pusher-js
```

Echoがインストールされたら、アプリケーションのJavaScriptで新しいEchoインスタンスを作成する準備が整いました。`install:broadcasting`コマンドは`resources/js/echo.js`にEcho設定ファイルを作成しますが、このファイルのデフォルト設定はLaravel Reverb向けです。以下の設定をコピーして、設定をPusherに移行することができます：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

次に、アプリケーションの`.env`ファイルでPusher環境変数に適切な値を定義する必要があります。これらの変数がまだ`.env`ファイルに存在しない場合は、追加する必要があります：

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"

VITE_APP_NAME="${APP_NAME}"
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

Echoの設定をアプリケーションのニーズに合わせて調整したら、アプリケーションのアセットをコンパイルすることができます：

```shell
npm run build
```

> NOTE:  
> アプリケーションのJavaScriptアセットのコンパイルについて詳しく学ぶには、[Vite](vite.md)のドキュメントを参照してください。

<a name="using-an-existing-client-instance"></a>
#### 既存のクライアントインスタンスの使用

既に設定済みのPusher Channelsクライアントインスタンスがあり、それをEchoで利用したい場合は、`client`設定オプションを介してEchoに渡すことができます：

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> NOTE:  
> 以下のドキュメントでは、Ablyを「Pusher互換」モードで使用する方法について説明します。ただし、Ablyチームは、Ablyが提供する独自の機能を活用できるブロードキャスターとEchoクライアントを推奨し、メンテナンスしています。Ablyがメンテナンスするドライバーの使用について詳しくは、[AblyのLaravelブロードキャスタードキュメント](https://github.com/ably/laravel-broadcaster)を参照してください。

[Laravel Echo](https://github.com/laravel/echo)は、サーバーサイドのブロードキャストドライバによってブロードキャストされるイベントを購読し、リッスンするのを容易にするJavaScriptライブラリです。Echoはまた、WebSocketのサブスクリプション、チャンネル、メッセージのためにPusherプロトコルを実装するために`pusher-js` NPMパッケージを活用します。

`install:broadcasting` Artisanコマンドは自動的に`laravel-echo`と`pusher-js`パッケージをインストールしますが、これらのパッケージは手動でNPMを介してインストールすることもできます：

```shell
npm install --save-dev laravel-echo pusher-js
```

**続行する前に、Ablyアプリケーションの設定でPusherプロトコルサポートを有効にする必要があります。この機能は、Ablyアプリケーションの設定ダッシュボードの「Protocol Adapter Settings」部分で有効にできます。**

Echoがインストールされたら、アプリケーションのJavaScriptで新しいEchoインスタンスを作成する準備が整いました。`install:broadcasting`コマンドは`resources/js/echo.js`にEcho設定ファイルを作成しますが、このファイルのデフォルト設定はLaravel Reverb向けです。以下の設定をコピーして、設定をAblyに移行することができます：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

Ably Echo設定で`VITE_ABLY_PUBLIC_KEY`環境変数を参照していることに注意してください。この変数の値は、Ablyの公開鍵である必要があります。公開鍵は、Ablyのキーのうち`:`文字の前にある部分です。

Echoの設定をニーズに合わせて調整したら、アプリケーションのアセットをコンパイルすることができます：

```shell
npm run dev
```

> NOTE:  
> アプリケーションのJavaScriptアセットのコンパイルについて詳しく学ぶには、[Vite](vite.md)のドキュメントを参照してください。

<a name="concept-overview"></a>
## 概念の概要

Laravelのイベントブロードキャストにより、サーバーサイドのLaravelイベントをWebSocketを使用してクライアントサイドのJavaScriptアプリケーションにブロードキャストすることができます。現在、Laravelには[Pusher Channels](https://pusher.com/channels)と[Ably](https://ably.com)のドライバが付属しています。イベントは、[Laravel Echo](#client-side-installation) JavaScriptパッケージを使用してクライアントサイドで簡単に消費できます。

イベントは「チャンネル」を介してブロードキャストされ、チャンネルは公開または非公開として指定できます。アプリケーションの訪問者は、認証や承認なしに公開チャンネルを購読できます。ただし、非公開チャンネルを購読するには、ユーザーは認証され、そのチャンネルをリッスンする権限が必要です。

<a name="using-example-application"></a>
### サンプルアプリケーションの使用

イベントブロードキャストの各コンポーネントに飛び込む前に、eコマースストアを例にして高レベルの概要を説明しましょう。

アプリケーションで、ユーザーが注文の配送状況を表示できるページがあるとします。また、配送状況の更新がアプリケーションによって処理されるときに`OrderShipmentStatusUpdated`イベントが発生するとします：

    use App\Events\OrderShipmentStatusUpdated;

    OrderShipmentStatusUpdated::dispatch($order);

<a name="the-shouldbroadcast-interface"></a>
#### `ShouldBroadcast`インターフェース

ユーザーが注文の一つを表示しているとき、状況の更新を表示するためにページを更新する必要はありません。代わりに、更新が作成されたときにアプリケーションにブロードキャストしたいと考えています。そのため、`OrderShipmentStatusUpdated`イベントに`ShouldBroadcast`インターフェースをマークする必要があります。これにより、Laravelにイベントが発生したときにブロードキャストするよう指示します：

    <?php

    namespace App\Events;

    use App\Models\Order;
    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\InteractsWithSockets;
    use Illuminate\Broadcasting\PresenceChannel;
    use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
    use Illuminate\Queue\SerializesModels;

    class OrderShipmentStatusUpdated implements ShouldBroadcast
    {
        /**
         * 注文インスタンス。
         *
         * @var \App\Models\Order
         */
        public $order;
    }

`ShouldBroadcast`インターフェースは、イベントに`broadcastOn`メソッドを定義する必要があります。このメソッドは、イベントがブロードキャストされるチャンネルを返す役割を果たします。生成されたイベントクラスには、このメソッドの空のスタブが既に定義されているため、詳細を埋めるだけです。注文の作成者だけが状況の更新を見ることができるように、注文に関連付けられた非公開チャンネルでイベントをブロードキャストします：

    use Illuminate\Broadcasting\Channel;
    use Illuminate\Broadcasting\PrivateChannel;

    /**
     * イベントがブロードキャストされるチャンネルを取得します。
     */
    public function broadcastOn(): Channel
    {
        return new PrivateChannel('orders.'.$this->order->id);
    }

イベントを複数のチャンネルでブロードキャストしたい場合は、代わりに`array`を返すことができます：

    use Illuminate\Broadcasting\PrivateChannel;

    /**
     * イベントがブロードキャストされるチャンネルを取得します。
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('orders.'.$this->order->id),
            // ...
        ];
    }

<a name="example-application-authorizing-channels"></a>
#### チャンネルの認可

ユーザーが非公開チャンネルを購読できるようにするには、チャンネルの認可ルールを定義する必要があります。これは、`routes/channels.php`ファイルで行います。この例では、`orders.1`チャンネルを購読しようとするユーザーが実際にその注文の所有者であることを確認する必要があります：

```php
use App\Models\Order;

Broadcast::channel('orders.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel`メソッドは2つの引数を取ります：チャンネルの名前と、ユーザーがチャンネルを購読できるかどうかを決定する`true`または`false`を返すクロージャです。

認可クロージャは、現在認証されているユーザーと、チャンネル名に含まれるすべてのワイルドカードパラメータを受け取ります。この例では、`{orderId}`プレースホルダーを使用して、チャンネル名の「order ID」部分がワイルドカードであることを示しています。
<a name="listening-for-event-broadcasts"></a>
#### イベントブロードキャストのリッスン

次に、残っているのはJavaScriptアプリケーションでイベントをリッスンすることです。これは [Laravel Echo](#client-side-installation) を使って行うことができます。まず、`private` メソッドを使用してプライベートチャンネルを購読します。次に、`listen` メソッドを使用して `OrderShipmentStatusUpdated` イベントをリッスンします。デフォルトでは、イベントのすべてのパブリックプロパティがブロードキャストイベントに含まれます。

```js
Echo.private(`orders.${orderId}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order);
    });
```

<a name="defining-broadcast-events"></a>
## ブロードキャストイベントの定義

Laravelに特定のイベントをブロードキャストするよう指示するには、イベントクラスに `Illuminate\Contracts\Broadcasting\ShouldBroadcast` インターフェースを実装する必要があります。このインターフェースは、フレームワークによって生成されるすべてのイベントクラスにインポートされているため、簡単に任意のイベントに追加できます。

`ShouldBroadcast` インターフェースでは、`broadcastOn` メソッドを実装する必要があります。`broadcastOn` メソッドは、イベントをブロードキャストするチャンネルまたはチャンネルの配列を返す必要があります。チャンネルは `Channel`、`PrivateChannel`、または `PresenceChannel` のインスタンスである必要があります。`Channel` のインスタンスは誰でも購読できるパブリックチャンネルを表し、`PrivateChannels` と `PresenceChannels` は [チャンネルの承認](#authorizing-channels) が必要なプライベートチャンネルを表します。

```php
<?php

namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('user.'.$this->user->id),
        ];
    }
}
```

`ShouldBroadcast` インターフェースを実装した後、通常どおり [イベントを発火](events.md) するだけです。イベントが発火されると、[キュージョブ](queues.md) が自動的に指定されたブロードキャストドライバを使用してイベントをブロードキャストします。

<a name="broadcast-name"></a>
### ブロードキャスト名

デフォルトでは、Laravelはイベントのクラス名を使用してイベントをブロードキャストします。ただし、イベントに `broadcastAs` メソッドを定義することでブロードキャスト名をカスタマイズできます。

```php
/**
 * The event's broadcast name.
 */
public function broadcastAs(): string
{
    return 'server.created';
}
```

`broadcastAs` メソッドを使用してブロードキャスト名をカスタマイズする場合、リスナーを先頭に `.` 文字を付けて登録する必要があります。これにより、Echoにアプリケーションの名前空間をイベントに追加しないように指示します。

```js
.listen('.server.created', function (e) {
    ....
});
```

<a name="broadcast-data"></a>
### ブロードキャストデータ

イベントがブロードキャストされると、そのすべての `public` プロパティは自動的にシリアライズされ、イベントのペイロードとしてブロードキャストされます。これにより、JavaScriptアプリケーションからそのパブリックデータにアクセスできます。したがって、例えばイベントにEloquentモデルを含む単一の `$user` パブリックプロパティがある場合、イベントのブロードキャストペイロードは次のようになります。

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

ただし、ブロードキャストペイロードをより細かく制御したい場合は、イベントに `broadcastWith` メソッドを追加できます。このメソッドは、イベントのペイロードとしてブロードキャストしたいデータの配列を返す必要があります。

```php
/**
 * Get the data to broadcast.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(): array
{
    return ['id' => $this->user->id];
}
```

<a name="broadcast-queue"></a>
### ブロードキャストキュー

デフォルトでは、各ブロードキャストイベントは `queue.php` 設定ファイルで指定されたデフォルトのキュー接続のデフォルトキューに配置されます。ブロードキャストに使用されるキュー接続と名前をカスタマイズするには、イベントクラスに `connection` および `queue` プロパティを定義します。

```php
/**
 * The name of the queue connection to use when broadcasting the event.
 *
 * @var string
 */
public $connection = 'redis';

/**
 * The name of the queue on which to place the broadcasting job.
 *
 * @var string
 */
public $queue = 'default';
```

または、イベントに `broadcastQueue` メソッドを定義することでキュー名をカスタマイズできます。

```php
/**
 * The name of the queue on which to place the broadcasting job.
 */
public function broadcastQueue(): string
{
    return 'default';
}
```

デフォルトのキュードライバの代わりに `sync` キューを使用してイベントをブロードキャストしたい場合は、`ShouldBroadcast` の代わりに `ShouldBroadcastNow` インターフェースを実装できます。

```php
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class OrderShipmentStatusUpdated implements ShouldBroadcastNow
{
    // ...
}
```

<a name="broadcast-conditions"></a>
### ブロードキャスト条件

特定の条件が真の場合にのみイベントをブロードキャストしたい場合があります。これらの条件は、イベントクラスに `broadcastWhen` メソッドを追加することで定義できます。

```php
/**
 * Determine if this event should broadcast.
 */
public function broadcastWhen(): bool
{
    return $this->order->value > 100;
}
```

<a name="broadcasting-and-database-transactions"></a>
#### ブロードキャストとデータベーストランザクション

ブロードキャストイベントがデータベーストランザクション内でディスパッチされると、データベーストランザクションがコミットされる前にキューによって処理される可能性があります。この場合、データベーストランザクション中にモデルやデータベースレコードに加えた更新がまだデータベースに反映されていない可能性があります。また、トランザクション内で作成されたモデルやデータベースレコードがまだデータベースに存在しない可能性があります。イベントがこれらのモデルに依存している場合、イベントをブロードキャストするジョブが処理されるときに予期しないエラーが発生する可能性があります。

キュー接続の `after_commit` 設定オプションが `false` に設定されている場合でも、イベントクラスに `ShouldDispatchAfterCommit` インターフェースを実装することで、すべての開いているデータベーストランザクションがコミットされた後に特定のブロードキャストイベントをディスパッチすることを示すことができます。

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast, ShouldDispatchAfterCommit
{
    use SerializesModels;
}
```

> NOTE:  
> これらの問題を回避する方法について詳しくは、[キュージョブとデータベーストランザクション](queues.md#jobs-and-database-transactions) に関するドキュメントを参照してください。

<a name="authorizing-channels"></a>
## チャンネルの承認

プライベートチャンネルは、現在認証されているユーザーが実際にそのチャンネルをリッスンできるかどうかを承認する必要があります。これは、チャンネル名を含むHTTPリクエストをLaravelアプリケーションに送信し、アプリケーションがユーザーがそのチャンネルをリッスンできるかどうかを判断することで実現されます。[Laravel Echo](#client-side-installation) を使用する場合、プライベートチャンネルへの購読を承認するためのHTTPリクエストは自動的に行われます。

ブロードキャストが有効になっている場合、Laravelは `/broadcasting/auth` ルートを自動的に登録して承認リクエストを処理します。`/broadcasting/auth` ルートは自動的に `web` ミドルウェアグループ内に配置されます。

<a name="defining-authorization-callbacks"></a>
### 承認コールバックの定義

次に、現在認証されているユーザーが特定のチャンネルをリッスンできるかどうかを実際に判断するロジックを定義する必要があります。これは、`install:broadcasting` Artisanコマンドによって作成された `routes/channels.php` ファイルで行います。このファイルで、`Broadcast::channel` メソッドを使用してチャンネル承認コールバックを登録できます。

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` メソッドは、チャンネルの名前と、ユーザーがチャンネルをリッスンする権限があるかどうかを示す `true` または `false` を返すコールバックの2つの引数を受け取ります。

すべての認証コールバックは、現在認証されているユーザーを最初の引数として受け取り、それに続いて追加のワイルドカードパラメータを受け取ります。この例では、チャンネル名の「ID」部分がワイルドカードであることを示すために、`{orderId}`プレースホルダを使用しています。

アプリケーションのブロードキャスト認証コールバックのリストは、`channel:list` Artisanコマンドを使用して表示できます：

```shell
php artisan channel:list
```

<a name="authorization-callback-model-binding"></a>
#### 認証コールバックモデルバインディング

HTTPルートと同様に、チャンネルルートも暗黙的および明示的な[ルートモデルバインディング](routing.md#route-model-binding)を利用できます。例えば、文字列または数値の注文IDを受け取る代わりに、実際の`Order`モデルインスタンスを要求できます：

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{order}', function (User $user, Order $order) {
    return $user->id === $order->user_id;
});
```

> WARNING:  
> HTTPルートモデルバインディングとは異なり、チャンネルモデルバインディングは自動的な[暗黙的モデルバインディングスコープ](routing.md#implicit-model-binding-scoping)をサポートしていません。ただし、これはほとんどのチャンネルが単一モデルのユニークなプライマリキーに基づいてスコープされるため、問題になることはほとんどありません。

<a name="authorization-callback-authentication"></a>
#### 認証コールバック認証

プライベートおよびプレゼンスブロードキャストチャンネルは、アプリケーションのデフォルトの認証ガードを介して現在のユーザーを認証します。ユーザーが認証されていない場合、チャンネル認証は自動的に拒否され、認証コールバックは実行されません。ただし、必要に応じて、受信リクエストを認証するために複数のカスタムガードを割り当てることができます：

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### チャンネルクラスの定義

アプリケーションが多くの異なるチャンネルを消費している場合、`routes/channels.php`ファイルは肥大化する可能性があります。そのため、チャンネルを認証するためにクロージャを使用する代わりに、チャンネルクラスを使用できます。チャンネルクラスを生成するには、`make:channel` Artisanコマンドを使用します。このコマンドは新しいチャンネルクラスを`App/Broadcasting`ディレクトリに配置します。

```shell
php artisan make:channel OrderChannel
```

次に、チャンネルを`routes/channels.php`ファイルに登録します：

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('orders.{order}', OrderChannel::class);
```

最後に、チャンネルクラスの`join`メソッドにチャンネルの認証ロジックを配置できます。この`join`メソッドには、通常チャンネル認証クロージャに配置するロジックを配置します。チャンネルモデルバインディングを利用することもできます：

```php
<?php

namespace App\Broadcasting;

use App\Models\Order;
use App\Models\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     */
    public function __construct() {}

    /**
     * Authenticate the user's access to the channel.
     */
    public function join(User $user, Order $order): array|bool
    {
        return $user->id === $order->user_id;
    }
}
```

> NOTE:  
> Laravelの他の多くのクラスと同様に、チャンネルクラスは[サービスコンテナ](container.md)によって自動的に解決されます。したがって、チャンネルのコンストラクタで必要な依存関係をタイプヒントで指定できます。

<a name="broadcasting-events"></a>
## イベントのブロードキャスト

イベントを定義し、`ShouldBroadcast`インターフェースでマークしたら、イベントのディスパッチメソッドを使用してイベントを発火するだけです。イベントディスパッチャは、イベントが`ShouldBroadcast`インターフェースでマークされていることを認識し、イベントをブロードキャストのためにキューに入れます：

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### 他のユーザーのみにブロードキャスト

イベントブロードキャストを利用するアプリケーションを構築する際、特定のチャンネルのすべてのサブスクライバーにイベントをブロードキャストする必要がある場合がありますが、現在のユーザーを除く必要があります。`broadcast`ヘルパーと`toOthers`メソッドを使用してこれを実現できます：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

`toOthers`メソッドをいつ使用するかをよりよく理解するために、タスク名を入力して新しいタスクを作成できるタスクリストアプリケーションを想像してみましょう。タスクを作成するために、アプリケーションはタスクの作成をブロードキャストし、新しいタスクのJSON表現を返す`/task` URLにリクエストを送信するかもしれません。JavaScriptアプリケーションがエンドポイントからのレスポンスを受け取ると、次のように新しいタスクをタスクリストに直接挿入するかもしれません：

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

ただし、タスクの作成もブロードキャストしていることを忘れないでください。JavaScriptアプリケーションがこのイベントをリッスンしてタスクリストにタスクを追加している場合、リストに重複したタスクが表示されます：エンドポイントからのものとブロードキャストからのものです。これを解決するために、`toOthers`メソッドを使用してブロードキャスタに現在のユーザーにイベントをブロードキャストしないように指示できます。

> WARNING:  
> `toOthers`メソッドを呼び出すには、イベントが`Illuminate\Broadcasting\InteractsWithSockets`トレイトを使用している必要があります。

<a name="only-to-others-configuration"></a>
#### 設定

Laravel Echoインスタンスを初期化すると、ソケットIDが接続に割り当てられます。グローバルな[Axios](https://github.com/mzabriskie/axios)インスタンスを使用してJavaScriptアプリケーションからHTTPリクエストを行う場合、ソケットIDは自動的にすべての送信リクエストに`X-Socket-ID`ヘッダとして添付されます。その後、`toOthers`メソッドを呼び出すと、LaravelはヘッダからソケットIDを抽出し、ブロードキャスタにそのソケットIDを持つ接続にイベントをブロードキャストしないように指示します。

グローバルなAxiosインスタンスを使用していない場合は、JavaScriptアプリケーションを手動で設定して、すべての送信リクエストに`X-Socket-ID`ヘッダを送信する必要があります。ソケットIDは`Echo.socketId`メソッドを使用して取得できます：

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### 接続のカスタマイズ

アプリケーションが複数のブロードキャスト接続と対話し、デフォルト以外のブロードキャスタを使用してイベントをブロードキャストする場合、`via`メソッドを使用してイベントをプッシュする接続を指定できます：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

または、イベントのコンストラクタ内で`broadcastVia`メソッドを呼び出してイベントのブロードキャスト接続を指定することもできます。ただし、その前に、イベントクラスが`InteractsWithBroadcasting`トレイトを使用していることを確認してください：

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithBroadcasting;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    use InteractsWithBroadcasting;

    /**
     * Create a new event instance.
     */
    public function __construct()
    {
        $this->broadcastVia('pusher');
    }
}
```

<a name="anonymous-events"></a>
### 匿名イベント

場合によっては、専用のイベントクラスを作成せずに、アプリケーションのフロントエンドに単純なイベントをブロードキャストしたいことがあります。これに対応するために、`Broadcast`ファサードを使用して「匿名イベント」をブロードキャストできます：

```php
Broadcast::on('orders.'.$order->id)->send();
```

上記の例は、次のイベントをブロードキャストします：

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

`as`および`with`メソッドを使用して、イベントの名前とデータをカスタマイズできます：

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

上記の例は、次のようなイベントをブロードキャストします：

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

匿名イベントをプライベートまたはプレゼンスチャンネルでブロードキャストする場合は、`private`および`presence`メソッドを使用できます：

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

`send`メソッドを使用して匿名イベントをブロードキャストすると、イベントはアプリケーションの[キュー](queues.md)にディスパッチされて処理されます。ただし、イベントを即座にブロードキャストしたい場合は、`sendNow`メソッドを使用できます：

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

現在認証されているユーザーを除くすべてのチャンネルサブスクライバーにイベントをブロードキャストするには、`toOthers`メソッドを呼び出すことができます：

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="receiving-broadcasts"></a>
## ブロードキャストの受信

<a name="listening-for-events"></a>
### イベントのリッスン

[Laravel Echoのインストールと初期化](#client-side-installation)が完了したら、Laravelアプリケーションからブロードキャストされるイベントのリッスンを開始する準備が整いました。まず、`channel`メソッドを使用してチャンネルのインスタンスを取得し、次に`listen`メソッドを呼び出して指定されたイベントをリッスンします：

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

プライベートチャンネルでイベントをリッスンしたい場合は、代わりに`private`メソッドを使用してください。単一のチャンネルで複数のイベントをリッスンするために、`listen`メソッドの呼び出しを連鎖させ続けることができます。

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### イベントリッスンの停止

[チャンネルを離れる](#leaving-a-channel)ことなく、特定のイベントのリッスンを停止したい場合は、`stopListening`メソッドを使用できます。

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated')
```

<a name="leaving-a-channel"></a>
### チャンネルの離脱

チャンネルを離れるには、Echoインスタンスの`leaveChannel`メソッドを呼び出すことができます。

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

チャンネルとそれに関連するプライベートおよびプレゼンスチャンネルも離れたい場合は、`leave`メソッドを呼び出すことができます。

```js
Echo.leave(`orders.${this.order.id}`);
```

<a name="namespaces"></a>
### 名前空間

上記の例では、イベントクラスの完全な`App\Events`名前空間を指定していないことに気づいたかもしれません。これは、Echoがイベントが`App\Events`名前空間にあると自動的に想定するためです。ただし、Echoをインスタンス化する際に`namespace`設定オプションを渡すことで、ルート名前空間を設定できます。

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

あるいは、Echoを使用してイベントを購読する際に、イベントクラスに`.`をプレフィックスとして付けることができます。これにより、常に完全修飾クラス名を指定できます。

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="presence-channels"></a>
## プレゼンスチャンネル

プレゼンスチャンネルは、プライベートチャンネルのセキュリティを基にしながら、チャンネルを購読しているユーザーを認識するという追加機能を提供します。これにより、他のユーザーが同じページを閲覧しているときにユーザーに通知したり、チャットルームの居住者をリストアップしたりするなど、強力で協力的なアプリケーション機能を簡単に構築できます。

<a name="authorizing-presence-channels"></a>
### プレゼンスチャンネルの認可

すべてのプレゼンスチャンネルはプライベートチャンネルでもあるため、ユーザーは[それらにアクセスするために認可される必要があります](#authorizing-channels)。ただし、プレゼンスチャンネルの認可コールバックを定義する際に、ユーザーがチャンネルに参加することが認可されている場合に`true`を返すのではなく、ユーザーに関するデータの配列を返す必要があります。

認可コールバックによって返されたデータは、JavaScriptアプリケーションのプレゼンスチャンネルイベントリスナーで利用可能になります。ユーザーがプレゼンスチャンネルに参加することが認可されていない場合は、`false`または`null`を返す必要があります。

```php
use App\Models\User;

Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### プレゼンスチャンネルへの参加

プレゼンスチャンネルに参加するには、Echoの`join`メソッドを使用できます。`join`メソッドは`PresenceChannel`実装を返します。これにより、`listen`メソッドを公開するだけでなく、`here`、`joining`、`leaving`イベントを購読できます。

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // ...
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

`here`コールバックは、チャンネルが正常に参加されるとすぐに実行され、現在チャンネルを購読している他のすべてのユーザーのユーザー情報を含む配列を受け取ります。`joining`メソッドは新しいユーザーがチャンネルに参加したときに実行され、`leaving`メソッドはユーザーがチャンネルを離れたときに実行されます。`error`メソッドは、認証エンドポイントが200以外のHTTPステータスコードを返した場合、または返されたJSONの解析に問題がある場合に実行されます。

<a name="broadcasting-to-presence-channels"></a>
### プレゼンスチャンネルへのブロードキャスト

プレゼンスチャンネルは、公開チャンネルやプライベートチャンネルと同様にイベントを受信できます。チャットルームの例を使用して、`NewMessage`イベントをルームのプレゼンスチャンネルにブロードキャストしたい場合、イベントの`broadcastOn`メソッドから`PresenceChannel`のインスタンスを返します。

```php
/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PresenceChannel('chat.'.$this->message->room_id),
    ];
}
```

他のイベントと同様に、`broadcast`ヘルパーと`toOthers`メソッドを使用して、現在のユーザーがブロードキャストを受信しないようにすることができます。

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

他の種類のイベントと同様に、Echoの`listen`メソッドを使用してプレゼンスチャンネルに送信されるイベントをリッスンできます。

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        // ...
    });
```

<a name="model-broadcasting"></a>
## モデルのブロードキャスト

> WARNING:  
> モデルのブロードキャストに関する以下のドキュメントを読む前に、Laravelのモデルブロードキャストサービスの一般的な概念と、ブロードキャストイベントを手動で作成してリッスンする方法について慣れておくことをお勧めします。

アプリケーションの[Eloquentモデル](eloquent.md)が作成、更新、または削除されたときにイベントをブロードキャストすることは一般的です。もちろん、これは手動で[Eloquentモデルの状態変化に対するカスタムイベントを定義](eloquent.md#events)し、それらのイベントに`ShouldBroadcast`インターフェースを付けることで簡単に実現できます。

ただし、アプリケーションでこれらのイベントを他の目的で使用していない場合、ブロードキャストするためだけにイベントクラスを作成するのは面倒な場合があります。これを解決するために、LaravelではEloquentモデルがその状態変化を自動的にブロードキャストするように指示できます。

これを開始するには、Eloquentモデルは`Illuminate\Database\Eloquent\BroadcastsEvents`トレイトを使用する必要があります。さらに、モデルはモデルのイベントをブロードキャストするチャンネルの配列を返す`broadcastOn`メソッドを定義する必要があります。

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Get the user that the post belongs to.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the channels that model events should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>
     */
    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

モデルがこのトレイトを含み、そのブロードキャストチャンネルを定義すると、モデルインスタンスが作成、更新、削除、ゴミ箱に入れられた、または復元されたときにイベントが自動的にブロードキャストされるようになります。

さらに、`broadcastOn`メソッドが文字列の`$event`引数を受け取ることに気づいたかもしれません。この引数にはモデルで発生したイベントのタイプが含まれ、`created`、`updated`、`deleted`、`trashed`、または`restored`の値を持ちます。この変数の値を検査することで、特定のイベントに対してモデルがどのチャンネル（もしあれば）にブロードキャストするかを決定できます。

```php
/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<string, array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>>
 */
public function broadcastOn(string $event): array
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### モデルブロードキャストイベント作成のカスタマイズ

Laravelがモデルブロードキャストイベントを作成する方法をカスタマイズしたい場合があります。これは、Eloquentモデルに`newBroadcastableEvent`メソッドを定義することで実現できます。このメソッドは`Illuminate\Database\Eloquent\BroadcastableModelEventOccurred`インスタンスを返す必要があります。

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Create a new broadcastable model event for the model.
 */
protected function newBroadcastableEvent(string $event): BroadcastableModelEventOccurred
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### モデルブロードキャストの規約

<a name="model-broadcasting-channel-conventions"></a>
#### チャンネルの規約

上記のモデル例では、`broadcastOn`メソッドが`Channel`インスタンスを返していないことに気づいたかもしれません。代わりに、Eloquentモデルが直接返されています。モデルの`broadcastOn`メソッドによってEloquentモデルインスタンスが返される場合（またはメソッドによって返される配列に含まれる場合）、Laravelは自動的にモデルのクラス名と主キー識別子を使用してチャンネル名としてプライベートチャンネルインスタンスを作成します。

つまり、`App\Models\User`モデルで`id`が`1`の場合、`Illuminate\Broadcasting\PrivateChannel`インスタンスに変換され、名前は`App.Models.User.1`になります。もちろん、モデルの`broadcastOn`メソッドからEloquentモデルインスタンスを返すだけでなく、完全な`Channel`インスタンスを返すことで、モデルのチャンネル名を完全に制御することができます。

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * モデルイベントがブロードキャストされるチャンネルを取得する。
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(string $event): array
{
    return [
        new PrivateChannel('user.'.$this->id)
    ];
}
```

モデルの`broadcastOn`メソッドからチャンネルインスタンスを明示的に返す場合、Eloquentモデルインスタンスをチャンネルのコンストラクタに渡すことができます。そうすると、Laravelは上記で説明したモデルチャンネルの規約を使用して、Eloquentモデルをチャンネル名の文字列に変換します。

```php
return [new Channel($this->user)];
```

モデルのチャンネル名を決定する必要がある場合、任意のモデルインスタンスで`broadcastChannel`メソッドを呼び出すことができます。例えば、このメソッドは`App\Models\User`モデルで`id`が`1`の場合、文字列`App.Models.User.1`を返します。

```php
$user->broadcastChannel()
```

<a name="model-broadcasting-event-conventions"></a>
#### イベントの規約

モデルブロードキャストイベントは、アプリケーションの`App\Events`ディレクトリ内の「実際の」イベントに関連付けられていないため、規約に基づいて名前とペイロードが割り当てられます。Laravelの規約では、モデルのクラス名（名前空間を除く）とブロードキャストをトリガーしたモデルイベントの名前を使用してイベントをブロードキャストします。

したがって、例えば、`App\Models\Post`モデルの更新は、クライアントサイドアプリケーションに`PostUpdated`というイベントを以下のペイロードでブロードキャストします。

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId",
}
```

`App\Models\User`モデルの削除は、`UserDeleted`という名前のイベントをブロードキャストします。

必要に応じて、モデルに`broadcastAs`と`broadcastWith`メソッドを追加することで、カスタムのブロードキャスト名とペイロードを定義できます。これらのメソッドは、発生しているモデルイベント/操作の名前を受け取り、各モデル操作に対してイベントの名前とペイロードをカスタマイズできます。`broadcastAs`メソッドから`null`が返されると、Laravelは上記で説明したモデルブロードキャストイベントの名前の規約を使用してイベントをブロードキャストします。

```php
/**
 * モデルイベントのブロードキャスト名。
 */
public function broadcastAs(string $event): string|null
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * モデルのブロードキャストデータを取得する。
 *
 * @return array<string, mixed>
 */
public function broadcastWith(string $event): array
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### モデルブロードキャストのリスニング

`BroadcastsEvents`トレイトをモデルに追加し、モデルの`broadcastOn`メソッドを定義したら、クライアントサイドアプリケーション内でブロードキャストされたモデルイベントのリスニングを開始する準備が整いました。開始する前に、[イベントのリスニング](#listening-for-events)に関する完全なドキュメントを確認することをお勧めします。

まず、`private`メソッドを使用してチャンネルのインスタンスを取得し、`listen`メソッドを呼び出して特定のイベントをリスニングします。通常、`private`メソッドに与えられるチャンネル名は、Laravelの[モデルブロードキャスト規約](#model-broadcasting-conventions)に対応する必要があります。

チャンネルインスタンスを取得したら、`listen`メソッドを使用して特定のイベントをリッスンできます。モデルブロードキャストイベントは、アプリケーションの`App\Events`ディレクトリ内の「実際の」イベントに関連付けられていないため、[イベント名](#model-broadcasting-event-conventions)の前に`.`を付けて、特定の名前空間に属していないことを示す必要があります。各モデルブロードキャストイベントには、モデルのすべてのブロードキャスト可能なプロパティを含む`model`プロパティがあります。

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.PostUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="client-events"></a>
## クライアントイベント

> NOTE:  
> [Pusher Channels](https://pusher.com/channels)を使用する場合、クライアントイベントを送信するには、[アプリケーションダッシュボード](https://dashboard.pusher.com/)の「App Settings」セクションで「Client Events」オプションを有効にする必要があります。

場合によっては、Laravelアプリケーションにアクセスせずに、他の接続されたクライアントにイベントをブロードキャストしたいことがあります。これは、例えば「入力中」通知のようなもので、特定の画面で他のユーザーがメッセージを入力していることをアプリケーションのユーザーに警告したい場合に特に便利です。

クライアントイベントをブロードキャストするには、Echoの`whisper`メソッドを使用できます。

```js
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

クライアントイベントをリッスンするには、`listenForWhisper`メソッドを使用できます。

```js
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

<a name="notifications"></a>
## 通知

イベントブロードキャストと[通知](notifications.md)を組み合わせることで、JavaScriptアプリケーションは、ページを更新することなく新しい通知を受け取ることができます。開始する前に、[ブロードキャスト通知チャンネル](notifications.md#broadcast-notifications)の使用に関するドキュメントを確認してください。

ブロードキャストチャンネルを使用するように通知を設定したら、Echoの`notification`メソッドを使用してブロードキャストイベントをリッスンできます。通知を受け取るエンティティのクラス名と一致するチャンネル名を使用することを忘れないでください。

```js
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

この例では、`broadcast`チャンネルを介して`App\Models\User`インスタンスに送信されたすべての通知がコールバックによって受信されます。`App.Models.User.{id}`チャンネルのチャンネル認証コールバックは、アプリケーションの`routes/channels.php`ファイルに含まれています。

