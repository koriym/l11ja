# Laravel Cashier (Paddle)

- [はじめに](#introduction)
- [Cashierのアップグレード](#upgrading-cashier)
- [インストール](#installation)
    - [Paddle Sandbox](#paddle-sandbox)
- [設定](#configuration)
    - [課金可能なモデル](#billable-model)
    - [APIキー](#api-keys)
    - [Paddle JS](#paddle-js)
    - [通貨設定](#currency-configuration)
    - [デフォルトモデルのオーバーライド](#overriding-default-models)
- [クイックスタート](#quickstart)
    - [商品の販売](#quickstart-selling-products)
    - [サブスクリプションの販売](#quickstart-selling-subscriptions)
- [チェックアウトセッション](#checkout-sessions)
    - [オーバーレイチェックアウト](#overlay-checkout)
    - [インラインチェックアウト](#inline-checkout)
    - [ゲストチェックアウト](#guest-checkouts)
- [価格プレビュー](#price-previews)
    - [顧客価格プレビュー](#customer-price-previews)
    - [割引](#price-discounts)
- [顧客](#customers)
    - [顧客のデフォルト設定](#customer-defaults)
    - [顧客の取得](#retrieving-customers)
    - [顧客の作成](#creating-customers)
- [サブスクリプション](#subscriptions)
    - [サブスクリプションの作成](#creating-subscriptions)
    - [サブスクリプションステータスの確認](#checking-subscription-status)
    - [サブスクリプションの単発課金](#subscription-single-charges)
    - [支払い情報の更新](#updating-payment-information)
    - [プランの変更](#changing-plans)
    - [サブスクリプションの数量](#subscription-quantity)
    - [複数商品のサブスクリプション](#subscriptions-with-multiple-products)
    - [複数サブスクリプション](#multiple-subscriptions)
    - [サブスクリプションの一時停止](#pausing-subscriptions)
    - [サブスクリプションのキャンセル](#canceling-subscriptions)
- [サブスクリプショントライアル](#subscription-trials)
    - [事前に支払い方法を設定](#with-payment-method-up-front)
    - [事前に支払い方法を設定しない](#without-payment-method-up-front)
    - [トライアルの延長またはアクティブ化](#extend-or-activate-a-trial)
- [Paddle Webhookの処理](#handling-paddle-webhooks)
    - [Webhookイベントハンドラの定義](#defining-webhook-event-handlers)
    - [Webhook署名の検証](#verifying-webhook-signatures)
- [単発課金](#single-charges)
    - [商品の課金](#charging-for-products)
    - [取引の返金](#refunding-transactions)
    - [取引のクレジット](#crediting-transactions)
- [取引](#transactions)
    - [過去と今後の支払い](#past-and-upcoming-payments)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

> WARNING:  
> このドキュメントは、Cashier Paddle 2.xのPaddle Billingとの統合についてです。Paddle Classicをまだ使用している場合は、[Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x)を使用する必要があります。

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)は、[Paddle](https://paddle.com)のサブスクリプション課金サービスに対して、表現力豊かで流暢なインターフェースを提供します。ほとんどすべての定型サブスクリプション課金コードを処理します。基本的なサブスクリプション管理に加えて、Cashierは以下を処理できます：サブスクリプションの入れ替え、サブスクリプションの「数量」、サブスクリプションの一時停止、キャンセル猶予期間など。

Cashier Paddleを深く掘り下げる前に、Paddleの[コンセプトガイド](https://developer.paddle.com/concepts/overview)と[APIドキュメント](https://developer.paddle.com/api-reference/overview)も確認することをお勧めします。

<a name="upgrading-cashier"></a>
## Cashierのアップグレード

Cashierの新しいバージョンにアップグレードする際は、[アップグレードガイド](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md)を慎重に確認することが重要です。

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを使用してPaddle用のCashierパッケージをインストールします：

```shell
composer require laravel/cashier-paddle
```

次に、`vendor:publish` Artisanコマンドを使用してCashierのマイグレーションファイルを公開します：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

その後、アプリケーションのデータベースマイグレーションを実行します。Cashierのマイグレーションにより、新しい`customers`テーブルが作成されます。さらに、顧客のすべてのサブスクリプションを保存するための新しい`subscriptions`と`subscription_items`テーブルが作成されます。最後に、顧客に関連するすべてのPaddleトランザクションを保存するための新しい`transactions`テーブルが作成されます：

```shell
php artisan migrate
```

> WARNING:  
> CashierがすべてのPaddleイベントを適切に処理できるように、[CashierのWebhook処理を設定](#handling-paddle-webhooks)することを忘れないでください。

<a name="paddle-sandbox"></a>
### Paddle Sandbox

ローカルおよびステージング開発中に、[Paddle Sandboxアカウントを登録](https://sandbox-login.paddle.com/signup)する必要があります。このアカウントは、実際の支払いを行わずにアプリケーションをテストおよび開発するためのサンドボックス環境を提供します。Paddleの[テストカード番号](https://developer.paddle.com/concepts/payment-methods/credit-debit-card)を使用して、さまざまな支払いシナリオをシミュレートできます。

Paddle Sandbox環境を使用する場合、アプリケーションの`.env`ファイル内で`PADDLE_SANDBOX`環境変数を`true`に設定する必要があります：

```ini
PADDLE_SANDBOX=true
```

アプリケーションの開発が完了したら、[Paddleベンダーアカウントを申請](https://paddle.com)できます。アプリケーションを本番環境に移行する前に、Paddleはアプリケーションのドメインを承認する必要があります。

<a name="configuration"></a>
## 設定

<a name="billable-model"></a>
### 課金可能なモデル

Cashierを使用する前に、`Billable`トレイトをユーザーモデル定義に追加する必要があります。このトレイトは、サブスクリプションの作成や支払い方法情報の更新など、一般的な課金タスクを実行するためのさまざまなメソッドを提供します：

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

ユーザー以外の課金可能なエンティティがある場合は、それらのクラスにもトレイトを追加できます：

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

<a name="api-keys"></a>
### APIキー

次に、アプリケーションの`.env`ファイルでPaddleキーを設定する必要があります。PaddleコントロールパネルからPaddle APIキーを取得できます：

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

[PaddleのSandbox環境](#paddle-sandbox)を使用する場合、`PADDLE_SANDBOX`環境変数を`true`に設定する必要があります。アプリケーションを本番環境にデプロイし、Paddleのライブベンダー環境を使用する場合、`PADDLE_SANDBOX`変数を`false`に設定する必要があります。

`PADDLE_RETAIN_KEY`はオプションであり、[Retain](https://developer.paddle.com/paddlejs/retain)でPaddleを使用している場合にのみ設定する必要があります。

<a name="paddle-js"></a>
### Paddle JS

Paddleは、Paddleチェックアウトウィジェットを初期化するために独自のJavaScriptライブラリに依存しています。アプリケーションレイアウトの閉じ`</head>`タグの直前に`@paddleJS` Bladeディレクティブを配置することで、JavaScriptライブラリを読み込むことができます：

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### 通貨設定

請求書に表示する金額のフォーマットに使用するロケールを指定できます。内部的には、Cashierは[PHPの`NumberFormatter`クラス](https://www.php.net/manual/en/class.numberformatter.php)を使用して通貨ロケールを設定します：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> WARNING:  
> `en`以外のロケールを使用するには、サーバーに`ext-intl` PHP拡張機能がインストールおよび設定されていることを確認してください。

<a name="overriding-default-models"></a>
### デフォルトモデルのオーバーライド

Cashierが内部で使用するモデルを独自のモデルで拡張することができます。独自のモデルを定義し、対応するCashierモデルを拡張します：

    use Laravel\Paddle\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

モデルを定義した後、`Laravel\Paddle\Cashier`クラスを介してCashierにカスタムモデルを使用するよう指示できます。通常、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドでCashierにカスタムモデルについて通知します：

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\Transaction;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useTransactionModel(Transaction::class);
    }

<a name="quickstart"></a>
## クイックスタート

<a name="quickstart-selling-products"></a>
### 商品の販売

> NOTE:  
> Paddle Checkoutを利用する前に、Paddleダッシュボードで固定価格の商品を定義する必要があります。さらに、[PaddleのWebhook処理を設定](#handling-paddle-webhooks)する必要があります。

アプリケーションを介して商品やサブスクリプションの課金を提供することは、難しい場合があります。しかし、Cashierと[PaddleのCheckout Overlay](https://www.paddle.com/billing/checkout)のおかげで、現代で堅牢な支払い統合を簡単に構築できます。

顧客に非繰り返しの単発商品を課金するために、Cashierを使用してPaddleのCheckout Overlayで顧客に課金します。顧客は支払い詳細を提供し、購入を確認します。支払いがCheckout Overlayを介して行われると、顧客はアプリケーション内で選択した成功URLにリダイレクトされます：

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $request->user()->checkout('pri_deluxe_album')
            ->returnTo(route('dashboard'));

        return view('buy', ['checkout' => $checkout]);
    })->name('checkout');

上記の例でわかるように、Cashierが提供する`checkout`メソッドを利用して、指定された「価格識別子」に対して、顧客にPaddleのチェックアウトオーバーレイを提示するためのチェックアウトオブジェクトを作成します。Paddleを使用する場合、「価格」とは、特定の製品に対して定義された価格を指します。

必要に応じて、`checkout`メソッドはPaddleに顧客を自動的に作成し、そのPaddleの顧客レコードをアプリケーションのデータベース内の対応するユーザーに接続します。チェックアウトセッションが完了すると、顧客は専用の成功ページにリダイレクトされ、そこで顧客に情報メッセージを表示できます。

`buy`ビューでは、チェックアウトオーバーレイを表示するためのボタンを含めます。`paddle-button` BladeコンポーネントはCashier Paddleに含まれていますが、[手動でオーバーレイチェックアウトをレンダリング](#manually-rendering-an-overlay-checkout)することもできます。

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    製品を購入
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### Paddleチェックアウトにメタデータを提供する

製品を販売する際、完了した注文や購入した製品を、アプリケーションで定義された`Cart`や`Order`モデルを通じて追跡するのが一般的です。顧客をPaddleのチェックアウトオーバーレイにリダイレクトして購入を完了させる際、既存の注文識別子を提供して、顧客がアプリケーションに戻った際に完了した購入を対応する注文に関連付ける必要があるかもしれません。

これを実現するために、`checkout`メソッドにカスタムデータの配列を提供できます。ユーザーがチェックアウトプロセスを開始すると、アプリケーション内で保留中の`Order`が作成されると想像してください。この例での`Cart`と`Order`モデルは説明のためのものであり、Cashierによって提供されるものではありません。これらの概念は、自分のアプリケーションのニーズに基づいて自由に実装できます。

```php
use App\Models\Cart;
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
    $order = Order::create([
        'cart_id' => $cart->id,
        'price_ids' => $cart->price_ids,
        'status' => 'incomplete',
    ]);

    $checkout = $request->user()->checkout($order->price_ids)
        ->customData(['order_id' => $order->id]);

    return view('billing', ['checkout' => $checkout]);
})->name('checkout');
```

上記の例でわかるように、ユーザーがチェックアウトプロセスを開始すると、カート/注文に関連するすべてのPaddle価格識別子を`checkout`メソッドに提供します。もちろん、これらのアイテムを顧客が追加する際に「ショッピングカート」または注文に関連付けるのは、アプリケーションの責任です。また、注文のIDを`customData`メソッドを通じてPaddleチェックアウトオーバーレイに提供します。

もちろん、顧客がチェックアウトプロセスを完了したら、注文を「完了」とマークしたいでしょう。これを実現するには、Paddleによってディスパッチされ、Cashierによってイベントとして発生するWebhookをリッスンして、注文情報をデータベースに保存することができます。

まず、Cashierによってディスパッチされる`TransactionCompleted`イベントをリッスンします。通常、イベントリスナーをアプリケーションの`AppServiceProvider`の`boot`メソッドに登録する必要があります。

```php
use App\Listeners\CompleteOrder;
use Illuminate\Support\Facades\Event;
use Laravel\Paddle\Events\TransactionCompleted;

/**
 * アプリケーションサービスをブートストラップする。
 */
public function boot(): void
{
    Event::listen(TransactionCompleted::class, CompleteOrder::class);
}
```

この例では、`CompleteOrder`リスナーは次のようになります。

```php
namespace App\Listeners;

use App\Models\Order;
use Laravel\Paddle\Cashier;
use Laravel\Paddle\Events\TransactionCompleted;

class CompleteOrder
{
    /**
     * 着信Cashier Webhookイベントを処理します。
     */
    public function handle(TransactionCompleted $event): void
    {
        $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

        $order = Order::findOrFail($orderId);

        $order->update(['status' => 'completed']);
    }
}
```

`transaction.completed`イベントに含まれるデータの詳細については、Paddleのドキュメントを参照してください。

<a name="quickstart-selling-subscriptions"></a>
### サブスクリプションの販売

> NOTE:  
> Paddleチェックアウトを利用する前に、Paddleダッシュボードで固定価格の製品を定義する必要があります。また、[PaddleのWebhook処理を設定](#handling-paddle-webhooks)する必要があります。

アプリケーションを通じて製品やサブスクリプションの請求を提供することは、難しい場合があります。しかし、Cashierと[Paddleのチェックアウトオーバーレイ](https://www.paddle.com/billing/checkout)のおかげで、現代で堅牢な支払い統合を簡単に構築できます。

CashierとPaddleのチェックアウトオーバーレイを使用してサブスクリプションを販売する方法を学ぶために、基本的な月額（`price_basic_monthly`）と年額（`price_basic_yearly`）のプランを持つサブスクリプションサービスのシンプルなシナリオを考えてみましょう。これらの2つの価格は、Paddleダッシュボードで「Basic」製品（`pro_basic`）の下にグループ化されるかもしれません。さらに、サブスクリプションサービスは、`pro_expert`としてExpertプランを提供するかもしれません。

まず、顧客がサービスにサブスクライブする方法を見てみましょう。もちろん、顧客はアプリケーションの価格ページでBasicプランの「サブスクライブ」ボタンをクリックするかもしれません。このボタンは、選択したプランのPaddleチェックアウトオーバーレイを呼び出します。まず、`checkout`メソッドを介してチェックアウトセッションを開始しましょう。

```php
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

`subscribe`ビューでは、チェックアウトオーバーレイを表示するためのボタンを含めます。`paddle-button` BladeコンポーネントはCashier Paddleに含まれていますが、[手動でオーバーレイチェックアウトをレンダリング](#manually-rendering-an-overlay-checkout)することもできます。

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    サブスクライブ
</x-paddle-button>
```

これで、サブスクライブボタンがクリックされると、顧客は支払い詳細を入力し、サブスクリプションを開始できるようになります。サブスクリプションが実際に開始されたことを知るために（一部の支払い方法は処理に数秒かかるため）、[CashierのWebhook処理を設定](#handling-paddle-webhooks)する必要もあります。

これで顧客はサブスクリプションを開始できるようになりましたが、アプリケーションの一部をサブスクライブしたユーザーのみがアクセスできるように制限する必要があります。もちろん、Cashierの`Billable`トレイトによって提供される`subscribed`メソッドを介して、ユーザーの現在のサブスクリプションステータスを常に確認できます。

```blade
@if ($user->subscribed())
    <p>サブスクライブしています。</p>
@endif
```

特定の製品や価格にサブスクライブしているかどうかを簡単に確認することもできます。

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>Basic製品にサブスクライブしています。</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>月額Basicプランにサブスクライブしています。</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### サブスクライブミドルウェアの構築

便宜上、着信リクエストがサブスクライブしたユーザーからのものであるかどうかを判断する[ミドルウェア](middleware.md)を作成したい場合があります。このミドルウェアを定義したら、サブスクライブしていないユーザーがルートにアクセスできないように、簡単にルートに割り当てることができます。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * 着信リクエストを処理する。
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // ユーザーを請求ページにリダイレクトしてサブスクライブを促す...
            return redirect('/subscribe');
        }

        return $next($request);
    }
}
```

ミドルウェアを定義したら、ルートに割り当てることができます。

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 顧客が請求プランを管理できるようにする

もちろん、顧客は別の製品や「階層」にサブスクリプションプランを変更したい場合があります。上記の例では、顧客が月額サブスクリプションから年額サブスクリプションにプランを変更できるようにしたいと考えています。これには、以下のルートにつながるボタンを実装する必要があります。

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // この例では"$price"は"price_basic_yearly"です。

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

プランを交換するだけでなく、顧客がサブスクリプションをキャンセルできるようにする必要もあります。プランを交換するのと同様に、以下のルートにつながるボタンを提供します。

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

これで、サブスクリプションは請求期間の終了時にキャンセルされます。

> NOTE:  
> CashierのWebhook処理を設定している限り、CashierはPaddleからの受信Webhookを調べることで、アプリケーションのCashier関連のデータベーステーブルを自動的に同期させます。たとえば、Paddleのダッシュボードから顧客のサブスクリプションをキャンセルすると、Cashierは対応するWebhookを受信し、アプリケーションのデータベースでそのサブスクリプションを「キャンセル済み」とマークします。

<a name="checkout-sessions"></a>
## チェックアウトセッション

顧客への請求操作のほとんどは、Paddleの[Checkout Overlayウィジェット](https://developer.paddle.com/build/checkout/build-overlay-checkout)を介して、または[インラインチェックアウト](https://developer.paddle.com/build/checkout/build-branded-inline-checkout)を利用して「チェックアウト」を実行します。

Paddleを使用してチェックアウト支払いを処理する前に、Paddleのチェックアウト設定ダッシュボードでアプリケーションの[デフォルト支払いリンク](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link)を定義する必要があります。

<a name="overlay-checkout"></a>
### オーバーレイチェックアウト

Checkout Overlayウィジェットを表示する前に、Cashierを使用してチェックアウトセッションを生成する必要があります。チェックアウトセッションは、実行すべき請求操作をチェックアウトウィジェットに通知します。

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

Cashierには`paddle-button` [Bladeコンポーネント](blade.md#components)が含まれています。このコンポーネントにチェックアウトセッションを「prop」として渡すことができます。そして、このボタンがクリックされると、Paddleのチェックアウトウィジェットが表示されます。

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

デフォルトでは、これによりPaddleのデフォルトスタイリングを使用してウィジェットが表示されます。ウィジェットをカスタマイズするには、`data-theme='light'`属性のような[Paddleがサポートする属性](https://developer.paddle.com/paddlejs/html-data-attributes)をコンポーネントに追加します。

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="light">
    Subscribe
</x-paddle-button>
```

Paddleのチェックアウトウィジェットは非同期です。ユーザーがウィジェット内でサブスクリプションを作成すると、PaddleはアプリケーションにWebhookを送信し、サブスクリプションの状態をアプリケーションのデータベースで適切に更新できるようにします。したがって、Paddleからの状態変更に対応するために、適切に[Webhookを設定する](#handling-paddle-webhooks)ことが重要です。

> WARNING:  
> サブスクリプションの状態が変更された後、対応するWebhookを受信するまでの遅延は通常最小限ですが、チェックアウトを完了した後すぐにユーザーのサブスクリプションが利用できない可能性を考慮して、アプリケーションでこれを考慮する必要があります。

<a name="manually-rendering-an-overlay-checkout"></a>
#### オーバーレイチェックアウトを手動でレンダリングする

Laravelの組み込みBladeコンポーネントを使用せずに、オーバーレイチェックアウトを手動でレンダリングすることもできます。まず、[前述の例](#overlay-checkout)のようにチェックアウトセッションを生成します。

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

次に、Paddle.jsを使用してチェックアウトを初期化します。この例では、`paddle_button`クラスが割り当てられたリンクを作成します。Paddle.jsはこのクラスを検出し、リンクがクリックされたときにオーバーレイチェックアウトを表示します。

```blade
<?php
$items = $checkout->getItems();
$customer = $checkout->getCustomer();
$custom = $checkout->getCustomData();
?>

<a
    href='#!'
    class='paddle_button'
    data-items='{!! json_encode($items) !!}'
    @if ($customer) data-customer-id='{{ $customer->paddle_id }}' @endif
    @if ($custom) data-custom-data='{{ json_encode($custom) }}' @endif
    @if ($returnUrl = $checkout->getReturnUrl()) data-success-url='{{ $returnUrl }}' @endif
>
    Buy Product
</a>
```

<a name="inline-checkout"></a>
### インラインチェックアウト

Paddleの「オーバーレイ」スタイルのチェックアウトウィジェットを使用したくない場合、Paddleはウィジェットをインラインで表示するオプションも提供しています。このアプローチでは、チェックアウトのHTMLフィールドを調整することはできませんが、ウィジェットをアプリケーション内に埋め込むことができます。

インラインチェックアウトを簡単に始めるために、Cashierには`paddle-checkout` Bladeコンポーネントが含まれています。まず、[チェックアウトセッションを生成](#overlay-checkout)します。

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

次に、チェックアウトセッションをコンポーネントの`checkout`属性に渡します。

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

インラインチェックアウトコンポーネントの高さを調整するには、`height`属性をBladeコンポーネントに渡します。

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

インラインチェックアウトのカスタマイズオプションの詳細については、Paddleの[インラインチェックアウトガイド](https://developer.paddle.com/build/checkout/build-branded-inline-checkout)と[利用可能なチェックアウト設定](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings)を参照してください。

<a name="manually-rendering-an-inline-checkout"></a>
#### インラインチェックアウトを手動でレンダリングする

Laravelの組み込みBladeコンポーネントを使用せずに、インラインチェックアウトを手動でレンダリングすることもできます。まず、[前述の例](#inline-checkout)のようにチェックアウトセッションを生成します。

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $user->checkout('pri_34567')
            ->returnTo(route('dashboard'));

        return view('billing', ['checkout' => $checkout]);
    });

次に、Paddle.jsを使用してチェックアウトを初期化します。この例では、[Alpine.js](https://github.com/alpinejs/alpine)を使用していますが、この例を自分のフロントエンドスタックに合わせて自由に変更できます。

```blade
<?php
$options = $checkout->options();

$options['settings']['frameTarget'] = 'paddle-checkout';
$options['settings']['frameInitialHeight'] = 366;
?>

<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open(@json($options));
">
</div>
```

<a name="guest-checkouts"></a>
### ゲストチェックアウト

アプリケーションでアカウントを必要としないユーザーのためにチェックアウトセッションを作成する必要がある場合があります。そのためには、`guest`メソッドを使用します。

    use Illuminate\Http\Request;
    use Laravel\Paddle\Checkout;

    Route::get('/buy', function (Request $request) {
        $checkout = Checkout::guest('pri_34567')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

次に、チェックアウトセッションを[Paddleボタン](#overlay-checkout)または[インラインチェックアウト](#inline-checkout)のBladeコンポーネントに渡します。

<a name="price-previews"></a>
## 価格プレビュー

Paddleでは、通貨ごとに価格をカスタマイズできます。つまり、異なる国に対して異なる価格を設定できます。Cashier Paddleでは、`previewPrices`メソッドを使用してこれらの価格をすべて取得できます。このメソッドは、価格を取得したい価格IDを受け取ります。

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456']);

通貨はリクエストのIPアドレスに基づいて決定されますが、特定の国の価格を取得するためにオプションで国を指定することもできます。

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456'], ['address' => [
        'country_code' => 'BE',
        'postal_code' => '1234',
    ]]);

価格を取得した後、それらを好きなように表示できます。

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

小計価格と税額を別々に表示することもできます。

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

詳細については、Paddleの[価格プレビューに関するAPIドキュメント](https://developer.paddle.com/api-reference/pricing-preview/preview-prices)を確認してください。

<a name="customer-price-previews"></a>
### 顧客の価格プレビュー

ユーザーがすでに顧客であり、その顧客に適用される価格を表示したい場合、顧客インスタンスから直接価格を取得できます。

    use App\Models\User;

    $prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);

内部的には、Cashierはユーザーの顧客IDを使用してその通貨で価格を取得します。たとえば、米国に住むユーザーは米ドルで価格を見ることになり、ベルギーに住むユーザーはユーロで価格を見ることになります。一致する通貨が見つからない場合、商品のデフォルト通貨が使用されます。Paddleのコントロールパネルで商品またはサブスクリプションプランのすべての価格をカスタマイズできます。

<a name="price-discounts"></a>
### 割引

割引後の価格を表示することもできます。`previewPrices`メソッドを呼び出す際に、`discount_id`オプションを介して割引IDを提供します。

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
        'discount_id' => 'dsc_123'
    ]);

そして、計算された価格を表示します。

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## 顧客

<a name="customer-defaults"></a>
### 顧客のデフォルト設定

Cashierを使用すると、チェックアウトセッションを作成する際に、顧客に対していくつかの便利なデフォルトを定義できます。これらのデフォルトを設定することで、顧客のメールアドレスと名前を事前に入力し、チェックアウトウィジェットの支払い部分にすぐに進むことができます。これらのデフォルトは、請求可能なモデルで以下のメソッドをオーバーライドすることで設定できます。

    /**
     * Paddleに関連付ける顧客の名前を取得します。
     */
    public function paddleName(): string|null
    {
        return $this->name;
    }

    /**
     * Paddleに関連付ける顧客のメールアドレスを取得します。
     */
    public function paddleEmail(): string|null
    {
        return $this->email;
    }

これらのデフォルトは、[チェックアウトセッション](#checkout-sessions)を生成するCashierのすべてのアクションで使用されます。

<a name="retrieving-customers"></a>
### 顧客の取得

`Cashier::findBillable`メソッドを使用して、Paddleの顧客IDで顧客を取得できます。このメソッドは、請求可能なモデルのインスタンスを返します。

    use Laravel\Paddle\Cashier;

    $user = Cashier::findBillable($customerId);

<a name="creating-customers"></a>
### 顧客の作成

場合によっては、サブスクリプションを開始せずにPaddleの顧客を作成したいことがあります。これは、`createAsCustomer`メソッドを使用して行うことができます。

    $customer = $user->createAsCustomer();

`Laravel\Paddle\Customer`のインスタンスが返されます。Paddleで顧客が作成されたら、後でサブスクリプションを開始できます。オプションの`$options`配列を渡して、[Paddle APIがサポートする追加の顧客作成パラメータ](https://developer.paddle.com/api-reference/customers/create-customer)を渡すこともできます。

    $customer = $user->createAsCustomer($options);

<a name="subscriptions"></a>
## サブスクリプション

<a name="creating-subscriptions"></a>
### サブスクリプションの作成

サブスクリプションを作成するには、まずデータベースから請求可能なモデルのインスタンスを取得します。これは通常、`App\Models\User`のインスタンスになります。モデルインスタンスを取得したら、`subscribe`メソッドを使用してモデルのチェックアウトセッションを作成できます。

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe($premium = 12345, 'default')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

`subscribe`メソッドに渡される最初の引数は、ユーザーがサブスクライブする特定の価格です。この値は、Paddle内の価格の識別子に対応する必要があります。`returnTo`メソッドは、ユーザーがチェックアウトを正常に完了した後にリダイレクトされるURLを受け取ります。`subscribe`メソッドに渡される2番目の引数は、サブスクリプションの内部「タイプ」です。アプリケーションが単一のサブスクリプションのみを提供する場合、これを`default`または`primary`と呼ぶことがあります。このサブスクリプションタイプは、内部アプリケーションの使用のみを目的としており、ユーザーに表示されることはありません。また、スペースを含めず、サブスクリプションを作成した後は変更しないでください。

サブスクリプションに関するカスタムメタデータを配列で提供するには、`customData`メソッドを使用します。

    $checkout = $request->user()->subscribe($premium = 12345, 'default')
        ->customData(['key' => 'value'])
        ->returnTo(route('home'));

サブスクリプションのチェックアウトセッションが作成されたら、Cashier Paddleに含まれる`paddle-button` [Bladeコンポーネント](#overlay-checkout)にチェックアウトセッションを提供できます。

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

ユーザーがチェックアウトを完了した後、Paddleから`subscription_created`ウェブフックがディスパッチされます。Cashierはこのウェブフックを受信し、顧客のサブスクリプションを設定します。アプリケーションがすべてのウェブフックを適切に受信および処理するように、[ウェブフックの処理](#handling-paddle-webhooks)を正しく設定していることを確認してください。

<a name="checking-subscription-status"></a>
### サブスクリプションステータスの確認

ユーザーがアプリケーションにサブスクライブした後、さまざまな便利なメソッドを使用してサブスクリプションステータスを確認できます。まず、`subscribed`メソッドは、ユーザーが有効なサブスクリプションを持っている場合、たとえサブスクリプションが現在試用期間内であっても、`true`を返します。

    if ($user->subscribed()) {
        // ...
    }

アプリケーションが複数のサブスクリプションを提供する場合、`subscribed`メソッドを呼び出す際にサブスクリプションを指定できます。

    if ($user->subscribed('default')) {
        // ...
    }

`subscribed`メソッドは、ユーザーのサブスクリプションステータスに基づいてルートとコントローラへのアクセスをフィルタリングするために、[ルートミドルウェア](middleware.md)に適した候補です。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureUserIsSubscribed
    {
        /**
         * 受信リクエストを処理します。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->user() && ! $request->user()->subscribed()) {
                // このユーザーは支払い済みの顧客ではありません...
                return redirect('/billing');
            }

            return $next($request);
        }
    }

ユーザーがまだ試用期間内かどうかを確認したい場合は、`onTrial`メソッドを使用できます。このメソッドは、ユーザーがまだ試用期間内であることを示す警告を表示する必要があるかどうかを判断するのに役立ちます。

    if ($user->subscription()->onTrial()) {
        // ...
    }

`subscribedToPrice`メソッドは、ユーザーが指定されたPaddle価格IDに基づいて特定のプランにサブスクライブしているかどうかを判断するために使用できます。この例では、ユーザーの`default`サブスクリプションが月額プランにアクティブにサブスクライブしているかどうかを判断します。

    if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
        // ...
    }

`recurring`メソッドは、ユーザーが現在アクティブなサブスクリプションを持っているかどうか、また試用期間や猶予期間が終了しているかどうかを判断するために使用できます。

    if ($user->subscription()->recurring()) {
        // ...
    }

<a name="canceled-subscription-status"></a>
#### キャンセルされたサブスクリプションステータス

ユーザーが一度アクティブなサブスクライバーであったが、サブスクリプションをキャンセルしたかどうかを判断するには、`canceled`メソッドを使用できます。

    if ($user->subscription()->canceled()) {
        // ...
    }

また、ユーザーがサブスクリプションをキャンセルしたが、サブスクリプションが完全に期限切れになるまでの「猶予期間」にまだいるかどうかを判断することもできます。たとえば、ユーザーが3月5日にサブスクリプションをキャンセルし、本来は3月10日に期限切れになる予定であった場合、ユーザーは3月10日まで「猶予期間」にいます。この間、`subscribed`メソッドはまだ`true`を返します。

    if ($user->subscription()->onGracePeriod()) {
        // ...
    }

<a name="past-due-status"></a>
#### 過去の期限切れステータス

サブスクリプションの支払いが失敗した場合、そのサブスクリプションは`past_due`としてマークされます。サブスクリプションがこの状態にある場合、顧客が支払い情報を更新するまでアクティブではありません。サブスクリプションが過去の期限切れかどうかを判断するには、サブスクリプションインスタンスの`pastDue`メソッドを使用します。

    if ($user->subscription()->pastDue()) {
        // ...
    }

サブスクリプションが`past_due`状態の場合、ユーザーに[支払い情報の更新](#updating-payment-information)を指示する必要があります。

サブスクリプションが`past_due`状態でも有効と見なしたい場合は、Cashierが提供する`keepPastDueSubscriptionsActive`メソッドを使用できます。通常、このメソッドは`AppServiceProvider`の`register`メソッドで呼び出す必要があります。

    use Laravel\Paddle\Cashier;

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        Cashier::keepPastDueSubscriptionsActive();
    }

> WARNING:  
> サブスクリプションが`past_due`状態の場合、支払い情報が更新されるまで変更できません。したがって、`swap`および`updateQuantity`メソッドは、サブスクリプションが`past_due`状態の場合に例外をスローします。

<a name="subscription-scopes"></a>
#### サブスクリプションスコープ

ほとんどのサブスクリプションステータスは、クエリスコープとしても利用できるため、指定された状態のサブスクリプションを簡単にクエリできます。

    // すべての有効なサブスクリプションを取得します...
    $subscriptions = Subscription::query()->valid()->get();

    // ユーザーのキャンセルされたサブスクリプションをすべて取得します...
    $subscriptions = $user->subscriptions()->canceled()->get();

利用可能なスコープの完全なリストは以下の通りです。

    Subscription::query()->valid();
    Subscription::query()->onTrial();
    Subscription::query()->expiredTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->active();
    Subscription::query()->recurring();
    Subscription::query()->pastDue();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->canceled();
    Subscription::query()->notCanceled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="subscription-single-charges"></a>
### サブスクリプションの一括請求

サブスクリプションの一括請求を使用すると、サブスクライバーに対してサブスクリプションに加えて一括請求を行うことができます。`charge`メソッドを呼び出す際に、1つまたは複数の価格IDを提供する必要があります。

    // 単一の価格を請求します...
    $response = $user->subscription()->charge('pri_123');

```php
// 一度に複数の価格を請求する...
$response = $user->subscription()->charge(['pri_123', 'pri_456']);
```

`charge`メソッドは、サブスクリプションの次の請求期間まで、実際には顧客に請求しません。顧客に即座に請求したい場合は、代わりに`chargeAndInvoice`メソッドを使用できます。

```php
$response = $user->subscription()->chargeAndInvoice('pri_123');
```

<a name="updating-payment-information"></a>
### 支払い情報の更新

Paddleは常にサブスクリプションごとに支払い方法を保存します。サブスクリプションのデフォルトの支払い方法を更新したい場合は、サブスクリプションモデルの`redirectToUpdatePaymentMethod`メソッドを使用して、顧客をPaddleのホストされた支払い方法更新ページにリダイレクトする必要があります。

```php
use Illuminate\Http\Request;

Route::get('/update-payment-method', function (Request $request) {
    $user = $request->user();

    return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

ユーザーが情報の更新を完了すると、`subscription_updated`ウェブフックがPaddleによってディスパッチされ、サブスクリプションの詳細がアプリケーションのデータベースで更新されます。

<a name="changing-plans"></a>
### プランの変更

ユーザーがアプリケーションにサブスクライブした後、新しいサブスクリプションプランに変更したい場合があります。ユーザーのサブスクリプションプランを更新するには、Paddleの価格の識別子をサブスクリプションの`swap`メソッドに渡す必要があります。

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

プランを交換し、ユーザーに即座に請求したい場合は、`swapAndInvoice`メソッドを使用できます。

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### 日割り計算

デフォルトでは、Paddleはプラン間の交換時に料金を日割り計算します。`noProrate`メソッドを使用して、料金を日割り計算せずにサブスクリプションを更新できます。

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

日割り計算を無効にし、顧客に即座に請求したい場合は、`swapAndInvoice`メソッドと`noProrate`を組み合わせて使用できます。

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

または、サブスクリプションの変更に対して顧客に請求しない場合は、`doNotBill`メソッドを使用できます。

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

Paddleの日割り計算ポリシーの詳細については、Paddleの[日割り計算ドキュメント](https://developer.paddle.com/concepts/subscriptions/proration)を参照してください。

<a name="subscription-quantity"></a>
### サブスクリプションの数量

サブスクリプションは、「数量」によって影響を受けることがあります。例えば、プロジェクト管理アプリケーションは、プロジェクトごとに月額$10を請求する場合があります。サブスクリプションの数量を簡単に増減するには、`incrementQuantity`と`decrementQuantity`メソッドを使用します。

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// サブスクリプションの現在の数量に5を追加...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// サブスクリプションの現在の数量から5を引く...
$user->subscription()->decrementQuantity(5);
```

または、`updateQuantity`メソッドを使用して特定の数量を設定できます。

```php
$user->subscription()->updateQuantity(10);
```

`noProrate`メソッドを使用して、料金を日割り計算せずにサブスクリプションの数量を更新できます。

```php
$user->subscription()->noProrate()->updateQuantity(10);
```

<a name="quantities-for-subscription-with-multiple-products"></a>
#### 複数の製品を持つサブスクリプションの数量

サブスクリプションが[複数の製品を持つサブスクリプション](#subscriptions-with-multiple-products)である場合、数量を増減する価格のIDをincrement / decrementメソッドの第2引数として渡す必要があります。

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### 複数の製品を持つサブスクリプション

[複数の製品を持つサブスクリプション](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons)を使用すると、単一のサブスクリプションに複数の請求製品を割り当てることができます。例えば、顧客サービスの「ヘルプデスク」アプリケーションがあり、月額$10の基本サブスクリプションがあり、ライブチャットアドオン製品が追加で月額$15で提供されているとします。

サブスクリプションのチェックアウトセッションを作成する際、`subscribe`メソッドの第1引数として価格の配列を渡すことで、特定のサブスクリプションに複数の製品を指定できます。

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe([
        'price_monthly',
        'price_chat',
    ]);

    return view('billing', ['checkout' => $checkout]);
});
```

上記の例では、顧客は`default`サブスクリプションに2つの価格が添付されます。両方の価格は、それぞれの請求間隔で請求されます。必要に応じて、各価格の特定の数量を示すために、キー/値ペアの連想配列を渡すことができます。

```php
$user = User::find(1);

$checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);
```

既存のサブスクリプションに別の価格を追加したい場合は、サブスクリプションの`swap`メソッドを使用する必要があります。`swap`メソッドを呼び出す際には、サブスクリプションの現在の価格と数量も含める必要があります。

```php
$user = User::find(1);

$user->subscription()->swap(['price_chat', 'price_original' => 2]);
```

上記の例では、新しい価格が追加されますが、顧客は次の請求サイクルまで請求されません。即座に顧客に請求したい場合は、`swapAndInvoice`メソッドを使用できます。

```php
$user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);
```

サブスクリプションから価格を削除するには、`swap`メソッドを使用し、削除したい価格を省略します。

```php
$user->subscription()->swap(['price_original' => 2]);
```

> WARNING:  
> サブスクリプションの最後の価格を削除することはできません。代わりに、サブスクリプションを単にキャンセルしてください。

<a name="multiple-subscriptions"></a>
### 複数のサブスクリプション

Paddleでは、顧客が複数のサブスクリプションを同時に持つことができます。例えば、ジムを運営しており、水泳サブスクリプションとダンベルサブスクリプションを提供しているとします。各サブスクリプションは異なる価格設定を持つことができます。もちろん、顧客はどちらか一方または両方のプランにサブスクライブできるはずです。

アプリケーションがサブスクリプションを作成する際、`subscribe`メソッドの第2引数としてサブスクリプションのタイプを提供できます。タイプは、ユーザーが開始しているサブスクリプションのタイプを表す任意の文字列です。

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

この例では、顧客の月額水泳サブスクリプションを開始しています。しかし、後で年間サブスクリプションに切り替えたい場合があります。顧客のサブスクリプションを調整する際に、`swimming`サブスクリプションの価格を交換するだけです。

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

もちろん、サブスクリプションを完全にキャンセルすることもできます。

```php
$user->subscription('swimming')->cancel();
```

<a name="pausing-subscriptions"></a>
### サブスクリプションの一時停止

サブスクリプションを一時停止するには、ユーザーのサブスクリプションの`pause`メソッドを呼び出します。

```php
$user->subscription()->pause();
```

サブスクリプションが一時停止されると、Cashierは自動的にデータベースの`paused_at`カラムを設定します。このカラムは、`paused`メソッドがいつ`true`を返すべきかを決定するために使用されます。例えば、顧客が3月1日にサブスクリプションを一時停止したが、サブスクリプションが3月5日まで再開される予定でなかった場合、`paused`メソッドは3月5日まで`false`を返し続けます。これは、通常、ユーザーが請求サイクルの終了までアプリケーションを使用し続けることが許可されているためです。

デフォルトでは、一時停止は次の請求間隔で行われるため、顧客は支払った期間の残りを使用できます。即座にサブスクリプションを一時停止したい場合は、`pauseNow`メソッドを使用できます。

```php
$user->subscription()->pauseNow();
```

`pauseUntil`メソッドを使用して、サブスクリプションを特定の時点まで一時停止できます。

```php
$user->subscription()->pauseUntil(now()->addMonth());
```

または、`pauseNowUntil`メソッドを使用して、即座にサブスクリプションを特定の時点まで一時停止できます。

```php
$user->subscription()->pauseNowUntil(now()->addMonth());
```

ユーザーがサブスクリプションを一時停止したが、まだ「猶予期間」にいるかどうかを判断するには、`onPausedGracePeriod`メソッドを使用できます。

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

一時停止されたサブスクリプションを再開するには、サブスクリプションの`resume`メソッドを呼び出します。

```php
$user->subscription()->resume();
```

> WARNING:  
> サブスクリプションが一時停止中は変更できません。別のプランに交換したり数量を更新したりする場合は、まずサブスクリプションを再開する必要があります。

<a name="canceling-subscriptions"></a>
### サブスクリプションのキャンセル

サブスクリプションをキャンセルするには、ユーザーのサブスクリプションの`cancel`メソッドを呼び出します。

```php
$user->subscription()->cancel();
```

サブスクリプションがキャンセルされると、Cashierは自動的にデータベース内の`ends_at`カラムを設定します。このカラムは、`subscribed`メソッドが`false`を返すべきタイミングを決定するために使用されます。例えば、顧客が3月1日にサブスクリプションをキャンセルしたが、サブスクリプションの終了予定日は3月5日であった場合、`subscribed`メソッドは3月5日まで`true`を返し続けます。これは、ユーザーが通常、請求サイクルの終了までアプリケーションを使用し続けることが許可されているためです。

ユーザーがサブスクリプションをキャンセルしたが、まだ「猶予期間」にいるかどうかを判断するには、`onGracePeriod`メソッドを使用できます:

    if ($user->subscription()->onGracePeriod()) {
        // ...
    }

サブスクリプションを即座にキャンセルしたい場合は、サブスクリプションの`cancelNow`メソッドを呼び出すことができます:

    $user->subscription()->cancelNow();

猶予期間中のサブスクリプションのキャンセルを停止するには、`stopCancelation`メソッドを呼び出すことができます:

    $user->subscription()->stopCancelation();

> WARNING:  
> Paddleのサブスクリプションは、キャンセル後に再開することはできません。顧客がサブスクリプションを再開したい場合は、新しいサブスクリプションを作成する必要があります。

<a name="subscription-trials"></a>
## サブスクリプションのトライアル

<a name="with-payment-method-up-front"></a>
### 事前に支払い方法を収集するトライアル

顧客にトライアル期間を提供しながら、事前に支払い方法の情報を収集したい場合、顧客がサブスクライブする価格のPaddleダッシュボードでトライアル期間を設定する必要があります。その後、通常通りチェックアウトセッションを開始します:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe('pri_monthly')
                    ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

アプリケーションが`subscription_created`イベントを受け取ると、Cashierはトライアル期間の終了日をアプリケーションのデータベース内のサブスクリプションレコードに設定し、この日付まで顧客への請求を開始しないようPaddleに指示します。

> WARNING:  
> 顧客のサブスクリプションがトライアル終了日までにキャンセルされない場合、トライアルが終了するとすぐに課金されますので、ユーザーにトライアル終了日を通知することを確認してください。

ユーザーがトライアル期間内にいるかどうかは、ユーザーインスタンスの`onTrial`メソッドまたはサブスクリプションインスタンスの`onTrial`メソッドを使用して判断できます。以下の2つの例は同等です:

    if ($user->onTrial()) {
        // ...
    }

    if ($user->subscription()->onTrial()) {
        // ...
    }

既存のトライアルが期限切れかどうかを判断するには、`hasExpiredTrial`メソッドを使用できます:

    if ($user->hasExpiredTrial()) {
        // ...
    }

    if ($user->subscription()->hasExpiredTrial()) {
        // ...
    }

特定のサブスクリプションタイプのトライアル中かどうかを判断するには、`onTrial`または`hasExpiredTrial`メソッドにタイプを指定できます:

    if ($user->onTrial('default')) {
        // ...
    }

    if ($user->hasExpiredTrial('default')) {
        // ...
    }

<a name="without-payment-method-up-front"></a>
### 事前に支払い方法を収集しないトライアル

顧客の支払い方法の情報を事前に収集せずにトライアル期間を提供したい場合、ユーザーに関連付けられた顧客レコードの`trial_ends_at`カラムを希望するトライアル終了日に設定できます。これは通常、ユーザー登録時に行われます:

    use App\Models\User;

    $user = User::create([
        // ...
    ]);

    $user->createAsCustomer([
        'trial_ends_at' => now()->addDays(10)
    ]);

Cashierはこのタイプのトライアルを「ジェネリックトライアル」と呼び、既存のサブスクリプションに関連付けられていません。`User`インスタンスの`onTrial`メソッドは、現在の日付が`trial_ends_at`の値を過ぎていない場合に`true`を返します:

    if ($user->onTrial()) {
        // ユーザーはトライアル期間内です。
    }

ユーザーのために実際のサブスクリプションを作成する準備ができたら、通常通り`subscribe`メソッドを使用できます:

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $user->subscribe('pri_monthly')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });

ユーザーのトライアル終了日を取得するには、`trialEndsAt`メソッドを使用できます。このメソッドは、ユーザーがトライアル中の場合はCarbonの日付インスタンスを返し、そうでない場合は`null`を返します。デフォルト以外の特定のサブスクリプションのトライアル終了日を取得したい場合は、オプションのサブスクリプションタイプパラメータを渡すこともできます:

    if ($user->onTrial('default')) {
        $trialEndsAt = $user->trialEndsAt();
    }

ユーザーがまだ実際のサブスクリプションを作成していない「ジェネリック」トライアル期間内にいるかどうかを知りたい場合は、`onGenericTrial`メソッドを使用できます:

    if ($user->onGenericTrial()) {
        // ユーザーは「ジェネリック」トライアル期間内です。
    }

<a name="extend-or-activate-a-trial"></a>
### トライアルの延長またはアクティベーション

既存のサブスクリプションのトライアル期間を延長するには、`extendTrial`メソッドを呼び出し、トライアルが終了する時点を指定します:

    $user->subscription()->extendTrial(now()->addDays(5));

または、トライアルを終了してサブスクリプションを即座にアクティベートするには、サブスクリプションの`activate`メソッドを呼び出します:

    $user->subscription()->activate();

<a name="handling-paddle-webhooks"></a>
## Paddle Webhooksの処理

Paddleは、さまざまなイベントをWebhooks経由でアプリケーションに通知できます。デフォルトでは、Cashierのサービスプロバイダによって、CashierのWebhookコントローラを指すルートが登録されます。このコントローラは、すべての着信Webhookリクエストを処理します。

デフォルトでは、このコントローラは、失敗した請求が多すぎるサブスクリプションのキャンセル、サブスクリプションの更新、支払い方法の変更を自動的に処理します。しかし、すぐにわかるように、このコントローラを拡張して、好きなPaddle Webhookイベントを処理することができます。

アプリケーションがPaddle Webhooksを処理できるようにするには、[PaddleコントロールパネルでWebhook URLを設定](https://vendors.paddle.com/alerts-webhooks)してください。デフォルトでは、CashierのWebhookコントローラは`/paddle/webhook` URLパスに応答します。Paddleコントロールパネルで有効にする必要があるすべてのWebhooksの完全なリストは以下の通りです:

- Customer Updated
- Transaction Completed
- Transaction Updated
- Subscription Created
- Subscription Updated
- Subscription Paused
- Subscription Canceled

> WARNING:  
> 着信リクエストをCashierの含まれる[Webhook署名検証](cashier-paddle.md#verifying-webhook-signatures)ミドルウェアで保護するようにしてください。

<a name="webhooks-csrf-protection"></a>
#### WebhooksとCSRF保護

Paddle WebhooksはLaravelの[CSRF保護](csrf.md)をバイパスする必要があるため、アプリケーションの`bootstrap/app.php`ファイルで`paddle/*`をCSRF保護から除外するようにしてください:

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->validateCsrfTokens(except: [
            'paddle/*',
        ]);
    })

<a name="webhooks-local-development"></a>
#### Webhooksとローカル開発

Paddleがローカル開発中にアプリケーションにWebhooksを送信できるようにするには、[Ngrok](https://ngrok.com/)や[Expose](https://expose.dev/docs/introduction)などのサイト共有サービスを介してアプリケーションを公開する必要があります。[Laravel Sail](sail.md)を使用してアプリケーションをローカルで開発している場合は、Sailの[サイト共有コマンド](sail.md#sharing-your-site)を使用できます。

<a name="defining-webhook-event-handlers"></a>
### Webhookイベントハンドラの定義

Cashierは、失敗した請求によるサブスクリプションのキャンセルやその他の一般的なPaddle Webhooksを自動的に処理します。ただし、追加のWebhookイベントを処理したい場合は、Cashierが発行する以下のイベントをリッスンすることができます:

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

両方のイベントには、Paddle Webhookの完全なペイロードが含まれています。例えば、`transaction.billed` Webhookを処理したい場合は、イベントを処理する[リスナー](events.md#defining-listeners)を登録できます:

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * Handle received Paddle webhooks.
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['event_type'] === 'transaction.billed') {
                // Handle the incoming event...
            }
        }
    }

Cashierは、受信したWebhookのタイプに特化したイベントも発行します。Paddleからの完全なペイロードに加えて、請求可能なモデル、サブスクリプション、またはレシートなど、Webhookの処理に使用された関連モデルも含まれます:

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

デフォルトの組み込みWebhookルートをオーバーライドするには、アプリケーションの`.env`ファイルで`CASHIER_WEBHOOK`環境変数を定義します。この値は、Webhookルートへの完全なURLである必要があり、Paddleコントロールパネルで設定されたURLと一致する必要があります:

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### Webhook署名の検証

Webhookを安全にするために、[PaddleのWebhook署名検証](https://developer.paddle.com/webhook-reference/verifying-webhooks)を使用できます。便利なことに、CashierにはPaddleのWebhookリクエストを検証するミドルウェアが含まれています。

すべての受信Webhookルートに`Laravel\Paddle\Http\Middleware\VerifyWebhookSignature`ミドルウェアを追加するだけで、Cashierはリクエストの署名を自動的に検証します:

    use Laravel\Paddle\Http\Middleware\VerifyWebhookSignature;

    Route::post('/paddle/webhook', function (Request $request) {
        // ...
    })->middleware(VerifyWebhookSignature::class);

Webhookを保護するために、[PaddleのWebhook署名](https://developer.paddle.com/webhook-reference/verifying-webhooks)を使用することができます。便宜上、Cashierは自動的にミドルウェアを含め、受信したPaddleのWebhookリクエストが有効であることを検証します。

Webhookの検証を有効にするには、アプリケーションの`.env`ファイルで`PADDLE_WEBHOOK_SECRET`環境変数が定義されていることを確認してください。Webhookシークレットは、Paddleアカウントのダッシュボードから取得できます。

<a name="single-charges"></a>
## 一括請求

<a name="charging-for-products"></a>
### 商品の請求

顧客に対して商品の購入を開始したい場合、請求可能なモデルインスタンスの`checkout`メソッドを使用して、購入のためのチェックアウトセッションを生成できます。`checkout`メソッドは、1つまたは複数の価格IDを受け取ります。必要に応じて、連想配列を使用して購入される商品の数量を指定できます：

    use Illuminate\Http\Request;

    Route::get('/buy', function (Request $request) {
        $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

        return view('buy', ['checkout' => $checkout]);
    });

チェックアウトセッションを生成した後、Cashierが提供する`paddle-button` [Bladeコンポーネント](#overlay-checkout)を使用して、ユーザーがPaddleのチェックアウトウィジェットを表示し、購入を完了できるようにすることができます：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy
</x-paddle-button>
```

チェックアウトセッションには`customData`メソッドがあり、基礎となるトランザクション作成に任意のカスタムデータを渡すことができます。カスタムデータを渡す際に利用可能なオプションの詳細については、[Paddleのドキュメント](https://developer.paddle.com/build/transactions/custom-data)を参照してください：

    $checkout = $user->checkout('pri_tshirt')
        ->customData([
            'custom_option' => $value,
        ]);

<a name="refunding-transactions"></a>
### トランザクションの返金

トランザクションの返金は、購入時に使用された顧客の支払い方法に返金額を返します。Paddleの購入を返金する必要がある場合、`Cashier\Paddle\Transaction`モデルの`refund`メソッドを使用できます。このメソッドは、最初の引数として理由を受け取り、オプションの金額とともに1つ以上の価格IDを連想配列として返金します。特定の請求可能モデルのトランザクションを`transactions`メソッドを使用して取得できます。

例えば、価格`pri_123`と`pri_456`の特定のトランザクションを返金したいとします。`pri_123`を全額返金し、`pri_456`を2ドルだけ返金したい場合：

    use App\Models\User;

    $user = User::find(1);

    $transaction = $user->transactions()->first();

    $response = $transaction->refund('Accidental charge', [
        'pri_123', // この価格を全額返金...
        'pri_456' => 200, // この価格を部分的に返金...
    ]);

上記の例は、トランザクション内の特定の明細項目を返金します。トランザクション全体を返金したい場合は、単に理由を指定します：

    $response = $transaction->refund('Accidental charge');

返金の詳細については、[Paddleの返金に関するドキュメント](https://developer.paddle.com/build/transactions/create-transaction-adjustments)を参照してください。

> WARNING:  
> 返金は、完全に処理される前に常にPaddleによって承認される必要があります。

<a name="crediting-transactions"></a>
### トランザクションのクレジット

返金と同様に、トランザクションにクレジットを付与することもできます。トランザクションにクレジットを付与すると、顧客の残高に資金が追加され、将来の購入に使用できるようになります。トランザクションのクレジットは、手動で収集されたトランザクションに対してのみ行うことができ、自動的に収集されたトランザクション（サブスクリプションなど）には行うことができません。Paddleはサブスクリプションのクレジットを自動的に処理します：

    $transaction = $user->transactions()->first();

    // 特定の明細項目を全額クレジット...
    $response = $transaction->credit('Compensation', 'pri_123');

詳細については、[Paddleのクレジットに関するドキュメント](https://developer.paddle.com/build/transactions/create-transaction-adjustments)を参照してください。

> WARNING:  
> クレジットは、手動で収集されたトランザクションに対してのみ適用できます。自動的に収集されたトランザクションは、Paddle自身によってクレジットされます。

<a name="transactions"></a>
## トランザクション

請求可能モデルのトランザクションの配列は、`transactions`プロパティを介して簡単に取得できます：

    use App\Models\User;

    $user = User::find(1);

    $transactions = $user->transactions;

トランザクションは、商品および購入の支払いを表し、請求書が付随しています。完了したトランザクションのみがアプリケーションのデータベースに保存されます。

顧客のトランザクションをリストする際に、トランザクションインスタンスのメソッドを使用して、関連する支払い情報を表示できます。例えば、テーブルにすべてのトランザクションをリストし、ユーザーが請求書を簡単にダウンロードできるようにすることができます：

```html
<table>
    @foreach ($transactions as $transaction)
        <tr>
            <td>{{ $transaction->billed_at->toFormattedDateString() }}</td>
            <td>{{ $transaction->total() }}</td>
            <td>{{ $transaction->tax() }}</td>
            <td><a href="{{ route('download-invoice', $transaction->id) }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

`download-invoice`ルートは、次のようになります：

    use Illuminate\Http\Request;
    use Laravel\Paddle\Transaction;

    Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
        return $transaction->redirectToInvoicePdf();
    })->name('download-invoice');

<a name="past-and-upcoming-payments"></a>
### 過去および今後の支払い

`lastPayment`および`nextPayment`メソッドを使用して、顧客の過去または今後の支払いを取得し、表示できます。これは、繰り返しのサブスクリプションに対して行われます：

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription();

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

これらのメソッドはどちらも`Laravel\Paddle\Payment`のインスタンスを返します。ただし、`lastPayment`はトランザクションがまだWebhookによって同期されていない場合に`null`を返し、`nextPayment`は請求サイクルが終了した場合（サブスクリプションがキャンセルされた場合など）に`null`を返します：

```blade
次の支払い: {{ $nextPayment->amount() }} 支払期限: {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## テスト

テスト中は、請求フローを手動でテストして、統合が期待どおりに機能することを確認する必要があります。

自動テスト（CI環境内で実行されるものを含む）の場合、[LaravelのHTTPクライアント](http-client.md#testing)を使用して、PaddleへのHTTPリクエストをモックすることができます。これにより、実際にPaddleのAPIを呼び出すことなくアプリケーションをテストする方法が提供されますが、Paddleからの実際のレスポンスはテストされません。

