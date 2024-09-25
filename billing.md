# Laravel Cashier (Stripe)

- [はじめに](#introduction)
- [Cashierのアップグレード](#upgrading-cashier)
- [インストール](#installation)
- [設定](#configuration)
    - [課金可能なモデル](#billable-model)
    - [APIキー](#api-keys)
    - [通貨設定](#currency-configuration)
    - [税金設定](#tax-configuration)
    - [ログ](#logging)
    - [カスタムモデルの使用](#using-custom-models)
- [クイックスタート](#quickstart)
    - [商品の販売](#quickstart-selling-products)
    - [サブスクリプションの販売](#quickstart-selling-subscriptions)
- [顧客](#customers)
    - [顧客の取得](#retrieving-customers)
    - [顧客の作成](#creating-customers)
    - [顧客の更新](#updating-customers)
    - [残高](#balances)
    - [税務ID](#tax-ids)
    - [顧客データのStripeとの同期](#syncing-customer-data-with-stripe)
    - [請求ポータル](#billing-portal)
- [支払い方法](#payment-methods)
    - [支払い方法の保存](#storing-payment-methods)
    - [支払い方法の取得](#retrieving-payment-methods)
    - [支払い方法の存在確認](#payment-method-presence)
    - [デフォルトの支払い方法の更新](#updating-the-default-payment-method)
    - [支払い方法の追加](#adding-payment-methods)
    - [支払い方法の削除](#deleting-payment-methods)
- [サブスクリプション](#subscriptions)
    - [サブスクリプションの作成](#creating-subscriptions)
    - [サブスクリプションのステータス確認](#checking-subscription-status)
    - [価格の変更](#changing-prices)
    - [サブスクリプションの数量](#subscription-quantity)
    - [複数商品のサブスクリプション](#subscriptions-with-multiple-products)
    - [複数のサブスクリプション](#multiple-subscriptions)
    - [従量課金](#metered-billing)
    - [サブスクリプションの税金](#subscription-taxes)
    - [サブスクリプションのアンカー日](#subscription-anchor-date)
    - [サブスクリプションのキャンセル](#cancelling-subscriptions)
    - [サブスクリプションの再開](#resuming-subscriptions)
- [サブスクリプションのトライアル](#subscription-trials)
    - [事前に支払い方法を登録](#with-payment-method-up-front)
    - [事前に支払い方法を登録しない](#without-payment-method-up-front)
    - [トライアル期間の延長](#extending-trials)
- [Stripe Webhookの処理](#handling-stripe-webhooks)
    - [Webhookイベントハンドラの定義](#defining-webhook-event-handlers)
    - [Webhook署名の検証](#verifying-webhook-signatures)
- [一括支払い](#single-charges)
    - [シンプルな支払い](#simple-charge)
    - [請求書付きの支払い](#charge-with-invoice)
    - [支払いインテントの作成](#creating-payment-intents)
    - [支払いの返金](#refunding-charges)
- [チェックアウト](#checkout)
    - [商品のチェックアウト](#product-checkouts)
    - [一括支払いのチェックアウト](#single-charge-checkouts)
    - [サブスクリプションのチェックアウト](#subscription-checkouts)
    - [税務IDの収集](#collecting-tax-ids)
    - [ゲストチェックアウト](#guest-checkouts)
- [請求書](#invoices)
    - [請求書の取得](#retrieving-invoices)
    - [次回の請求書](#upcoming-invoices)
    - [サブスクリプション請求書のプレビュー](#previewing-subscription-invoices)
    - [請求書PDFの生成](#generating-invoice-pdfs)
- [失敗した支払いの処理](#handling-failed-payments)
    - [支払いの確認](#confirming-payments)
- [強力な顧客認証 (SCA)](#strong-customer-authentication)
    - [追加確認が必要な支払い](#payments-requiring-additional-confirmation)
    - [セッション外の支払い通知](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) は、[Stripe](https://stripe.com) のサブスクリプション課金サービスに対して、表現力豊かで流暢なインターフェースを提供します。サブスクリプション課金コードのほとんどの定型コードを処理します。サブスクリプション管理に加えて、Cashierはクーポン、サブスクリプションの切り替え、サブスクリプションの「数量」、キャンセル猶予期間、さらには請求書のPDF生成まで処理できます。

<a name="upgrading-cashier"></a>
## Cashierのアップグレード

Cashierを新しいバージョンにアップグレードする際には、[アップグレードガイド](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md)を注意深く確認することが重要です。

> WARNING:  
> 重大な変更を防ぐために、Cashierは固定のStripe APIバージョンを使用します。Cashier 15はStripe APIバージョン `2023-10-16` を使用しています。Stripe APIバージョンは、新しいStripe機能と改善を利用するためにマイナーリリースで更新されます。

<a name="installation"></a>
## インストール

まず、Composerパッケージマネージャを使用してStripe用のCashierパッケージをインストールします:

```shell
composer require laravel/cashier
```

パッケージをインストールした後、`vendor:publish` Artisanコマンドを使用してCashierのマイグレーションを公開します:

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

次に、データベースをマイグレートします:

```shell
php artisan migrate
```

Cashierのマイグレーションにより、`users`テーブルにいくつかのカラムが追加されます。また、すべての顧客のサブスクリプションを保持するための新しい`subscriptions`テーブルと、複数の価格を持つサブスクリプションのための`subscription_items`テーブルが作成されます。

必要に応じて、`vendor:publish` Artisanコマンドを使用してCashierの設定ファイルを公開することもできます:

```shell
php artisan vendor:publish --tag="cashier-config"
```

最後に、CashierがすべてのStripeイベントを適切に処理できるように、[CashierのWebhook処理の設定](#handling-stripe-webhooks)を忘れずに行ってください。

> WARNING:  
> Stripeは、Stripe識別子を格納するために使用されるカラムが大文字と小文字を区別することを推奨しています。したがって、MySQLを使用する場合は、`stripe_id`カラムの照合順序が`utf8_bin`に設定されていることを確認してください。これに関する詳細情報は、[Stripeのドキュメント](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible)で確認できます。

<a name="configuration"></a>
## 設定

<a name="billable-model"></a>
### 課金可能なモデル

Cashierを使用する前に、課金可能なモデル定義に`Billable`トレイトを追加してください。通常、これは`App\Models\User`モデルになります。このトレイトは、サブスクリプションの作成、クーポンの適用、支払い方法情報の更新など、一般的な課金タスクを実行するためのさまざまなメソッドを提供します:

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashierは、課金可能なモデルがLaravelに付属の`App\Models\User`クラスであると想定しています。これを変更したい場合は、`useCustomerModel`メソッドを介して別のモデルを指定できます。このメソッドは通常、`AppServiceProvider`クラスの`boot`メソッドで呼び出されます:

    use App\Models\Cashier\User;
    use Laravel\Cashier\Cashier;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Cashier::useCustomerModel(User::class);
    }

> WARNING:  
> Laravelの提供する`App\Models\User`モデル以外のモデルを使用する場合は、[Cashierのマイグレーション](#installation)を公開し、代替モデルのテーブル名に合わせて変更する必要があります。

<a name="api-keys"></a>
### APIキー

次に、アプリケーションの`.env`ファイルでStripe APIキーを設定する必要があります。StripeコントロールパネルからStripe APIキーを取得できます:

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> WARNING:  
> アプリケーションの`.env`ファイルで`STRIPE_WEBHOOK_SECRET`環境変数を定義して、受信するWebhookが実際にStripeからのものであることを確認する必要があります。

<a name="currency-configuration"></a>
### 通貨設定

Cashierのデフォルト通貨は米ドル（USD）です。アプリケーションの`.env`ファイル内で`CASHIER_CURRENCY`環境変数を設定することで、デフォルト通貨を変更できます:

```ini
CASHIER_CURRENCY=eur
```

Cashierの通貨を設定するだけでなく、請求書に表示される金額の書式設定に使用されるロケールを指定することもできます。内部では、Cashierは[PHPの`NumberFormatter`クラス](https://www.php.net/manual/en/class.numberformatter.php)を使用して通貨ロケールを設定します:

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> WARNING:  
> `en`以外のロケールを使用するには、`ext-intl` PHP拡張機能がサーバーにインストールおよび設定されていることを確認してください。

<a name="tax-configuration"></a>
### 税金設定

[Stripe Tax](https://stripe.com/tax)のおかげで、Stripeによって生成されるすべての請求書に対して自動的に税金を計算することが可能です。アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドで`calculateTaxes`メソッドを呼び出すことで、自動税金計算を有効にできます:

    use Laravel\Cashier\Cashier;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Cashier::calculateTaxes();
    }

税金計算が有効になると、新しいサブスクリプションと生成される一括請求書に対して自動税金計算が行われます。

この機能が正しく機能するためには、顧客の請求詳細（顧客の名前、住所、税務IDなど）をStripeに同期する必要があります。Cashierが提供する[顧客データの同期](#syncing-customer-data-with-stripe)と[税務ID](#tax-ids)のメソッドを使用してこれを実現できます。

<a name="logging"></a>
### ログ

Cashierでは、致命的なStripeエラーのログに使用されるログチャネルを指定できます。アプリケーションの`.env`ファイル内で`CASHIER_LOGGER`環境変数を定義することで、ログチャネルを指定できます:

```ini
CASHIER_LOGGER=stack
```

StripeへのAPI呼び出しで生成される例外は、アプリケーションのデフォルトのログチャネルを通じてログに記録されます。

<a name="using-custom-models"></a>
### カスタムモデルの使用

自分のモデルを定義し、対応するCashierモデルを拡張することで、Cashierが内部的に使用するモデルを自由に拡張できます。

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

モデルを定義した後、`Laravel\Cashier\Cashier`クラスを介してカスタムモデルを使用するようにCashierに指示できます。通常、アプリケーションの`App\Providers\AppServiceProvider`クラスの`boot`メソッドでCashierにカスタムモデルについて通知する必要があります。

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\SubscriptionItem;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useSubscriptionItemModel(SubscriptionItem::class);
}
```

<a name="quickstart"></a>
## クイックスタート

<a name="quickstart-selling-products"></a>
### 商品の販売

> NOTE:  
> Stripe Checkoutを利用する前に、Stripeダッシュボードで固定価格の商品を定義する必要があります。さらに、[CashierのWebhook処理を設定](#handling-stripe-webhooks)する必要があります。

アプリケーションを通じて商品やサブスクリプションの請求を行うことは、難しい場合があります。しかし、Cashierと[Stripe Checkout](https://stripe.com/payments/checkout)のおかげで、現代で堅牢な支払いの統合を簡単に構築できます。

顧客に非定期的な、一括払いの商品を請求するために、Cashierを使用して顧客をStripe Checkoutにリダイレクトし、そこで支払い詳細を提供し、購入を確認します。支払いがCheckoutを通じて行われると、顧客はアプリケーション内で選択した成功URLにリダイレクトされます。

```php
use Illuminate\Http\Request;

Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';

    $quantity = 1;

    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');

Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

上記の例でわかるように、Cashierが提供する`checkout`メソッドを使用して、指定された「価格識別子」に対して顧客をStripe Checkoutにリダイレクトします。Stripeを使用する場合、「価格」は[特定の商品の定義された価格](https://stripe.com/docs/products-prices/how-products-and-prices-work)を指します。

必要に応じて、`checkout`メソッドは自動的にStripeに顧客を作成し、そのStripe顧客レコードをアプリケーションのデータベース内の対応するユーザーに接続します。チェックアウトセッションが完了すると、顧客は専用の成功またはキャンセルページにリダイレクトされ、そこで情報メッセージを表示できます。

<a name="providing-meta-data-to-stripe-checkout"></a>
#### Stripe Checkoutにメタデータを提供する

商品を販売する際、アプリケーションで定義された`Cart`や`Order`モデルを介して完了した注文や購入した商品を追跡するのが一般的です。顧客をStripe Checkoutにリダイレクトして購入を完了させる際、既存の注文識別子を提供して、顧客がアプリケーションに戻った際に対応する注文と購入を関連付ける必要がある場合があります。

これを実現するために、`checkout`メソッドに`metadata`の配列を提供できます。顧客がチェックアウトプロセスを開始すると、保留中の`Order`がアプリケーション内に作成されると想像してください。この例の`Cart`と`Order`モデルは説明のためのものであり、Cashierによって提供されるものではありません。独自のアプリケーションのニーズに基づいてこれらの概念を自由に実装できます。

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

    return $request->user()->checkout($order->price_ids, [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
        'metadata' => ['order_id' => $order->id],
    ]);
})->name('checkout');
```

上記の例でわかるように、顧客がチェックアウトプロセスを開始する際、カート/注文に関連するすべてのStripe価格識別子を`checkout`メソッドに提供します。もちろん、これらのアイテムを顧客が追加する際に「ショッピングカート」または注文に関連付けるのはアプリケーションの責任です。また、`metadata`配列を介して注文のIDをStripe Checkoutセッションに提供します。最後に、Checkout成功ルートに`CHECKOUT_SESSION_ID`テンプレート変数を追加しました。Stripeが顧客をアプリケーションにリダイレクトする際、このテンプレート変数は自動的にCheckoutセッションIDで埋められます。

次に、Checkout成功ルートを構築しましょう。これは、顧客がStripe Checkoutを通じて購入を完了した後にリダイレクトされるルートです。このルート内で、Stripe CheckoutセッションIDと関連するStripe Checkoutインスタンスを取得し、提供されたメタデータにアクセスして顧客の注文を適切に更新できます。

```php
use App\Models\Order;
use Illuminate\Http\Request;
use Laravel\Cashier\Cashier;

Route::get('/checkout/success', function (Request $request) {
    $sessionId = $request->get('session_id');

    if ($sessionId === null) {
        return;
    }

    $session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);

    if ($session->payment_status !== 'paid') {
        return;
    }

    $orderId = $session['metadata']['order_id'] ?? null;

    $order = Order::findOrFail($orderId);

    $order->update(['status' => 'completed']);

    return view('checkout-success', ['order' => $order]);
})->name('checkout-success');
```

Checkoutセッションオブジェクトに含まれるデータの詳細については、Stripeのドキュメントを参照してください。

<a name="quickstart-selling-subscriptions"></a>
### サブスクリプションの販売

> NOTE:  
> Stripe Checkoutを利用する前に、Stripeダッシュボードで固定価格の商品を定義する必要があります。さらに、[CashierのWebhook処理を設定](#handling-stripe-webhooks)する必要があります。

アプリケーションを通じて商品やサブスクリプションの請求を行うことは、難しい場合があります。しかし、Cashierと[Stripe Checkout](https://stripe.com/payments/checkout)のおかげで、現代で堅牢な支払いの統合を簡単に構築できます。

CashierとStripe Checkoutを使用してサブスクリプションを販売する方法を学ぶために、基本的な月次（`price_basic_monthly`）と年次（`price_basic_yearly`）のプランを持つサブスクリプションサービスのシンプルなシナリオを考えてみましょう。これらの2つの価格は、Stripeダッシュボードで「Basic」商品（`pro_basic`）の下にグループ化される可能性があります。さらに、サブスクリプションサービスは`pro_expert`としてExpertプランを提供するかもしれません。

まず、顧客がサービスにサブスクライブする方法を見てみましょう。もちろん、顧客はアプリケーションの価格ページでBasicプランの「サブスクライブ」ボタンをクリックする可能性があります。このボタンまたはリンクは、選択したプランのStripe Checkoutセッションを作成するLaravelルートにユーザーを誘導する必要があります。

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

上記の例でわかるように、顧客をBasicプランにサブスクライブさせるために、Stripe Checkoutセッションにリダイレクトします。チェックアウトが成功またはキャンセルされると、顧客は`checkout`メソッドに提供したURLにリダイレクトされます。サブスクリプションが実際に開始された時点を知るために（一部の支払い方法は処理に数秒かかるため）、[CashierのWebhook処理を設定](#handling-stripe-webhooks)する必要もあります。

顧客がサブスクリプションを開始できるようになったので、アプリケーションの一部をサブスクライブしたユーザーのみがアクセスできるように制限する必要があります。もちろん、Cashierの`Billable`トレイトが提供する`subscribed`メソッドを介してユーザーの現在のサブスクリプションステータスを常に確認できます。

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

特定の商品または価格にサブスクライブしているかどうかを簡単に確認することもできます。

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### サブスクライブ済みミドルウェアの構築

便宜上、受信リクエストがサブスクライブしたユーザーからのものであるかどうかを判断する[ミドルウェア](middleware.md)を作成したい場合があります。このミドルウェアを定義したら、サブスクライブしていないユーザーがルートにアクセスできないように、ルートに簡単に割り当てることができます。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // このユーザーはサブスクライブしていません。
            return redirect('subscribe');
        }

        return $next($request);
    }
}
```

```php
class Subscribed
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // Redirect user to billing page and ask them to subscribe...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

ミドルウェアが定義されたら、ルートに割り当てることができます。

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 顧客が請求プランを管理できるようにする

もちろん、顧客は別の製品や「階層」にサブスクリプションプランを変更したいと思うかもしれません。これを許可する最も簡単な方法は、顧客をStripeの[Customer Billing Portal](https://stripe.com/docs/no-code/customer-portal)に誘導することです。これは、顧客が請求書をダウンロードしたり、支払い方法を更新したり、サブスクリプションプランを変更したりできるホストされたユーザーインターフェースを提供します。

まず、ユーザーをBilling Portalセッションを開始するために使用するLaravelルートに誘導するアプリケーション内にリンクまたはボタンを定義します。

```blade
<a href="{{ route('billing') }}">
    Billing
</a>
```

次に、Stripe Customer Billing Portalセッションを開始し、ユーザーをPortalにリダイレクトするルートを定義しましょう。`redirectToBillingPortal`メソッドは、ユーザーがPortalを終了したときに戻るべきURLを受け取ります。

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> NOTE:  
> CashierのWebhook処理を設定している限り、CashierはStripeからの着信Webhookを調査することで、アプリケーションのCashier関連のデータベーステーブルを自動的に同期させます。したがって、例えば、ユーザーがStripeのCustomer Billing Portalを介してサブスクリプションをキャンセルした場合、Cashierは対応するWebhookを受け取り、アプリケーションのデータベースでサブスクリプションを「キャンセル済み」とマークします。

<a name="customers"></a>
## 顧客

<a name="retrieving-customers"></a>
### 顧客の取得

`Cashier::findBillable`メソッドを使用して、Stripe IDで顧客を取得できます。このメソッドは、課金可能なモデルのインスタンスを返します。

```php
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

<a name="creating-customers"></a>
### 顧客の作成

場合によっては、サブスクリプションを開始せずにStripe顧客を作成したいことがあります。これは、`createAsStripeCustomer`メソッドを使用して行うことができます。

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

顧客がStripeで作成されたら、後でサブスクリプションを開始できます。Stripe APIがサポートする追加の[顧客作成パラメータ](https://stripe.com/docs/api/customers/create)を渡すために、オプションの`$options`配列を提供することもできます。

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

課金可能なモデルのStripe顧客オブジェクトを返したい場合は、`asStripeCustomer`メソッドを使用できます。

```php
$stripeCustomer = $user->asStripeCustomer();
```

指定された課金可能なモデルのStripe顧客オブジェクトを取得したいが、そのモデルがすでにStripe内の顧客であるかどうかわからない場合は、`createOrGetStripeCustomer`メソッドを使用できます。このメソッドは、Stripeにまだ顧客が存在しない場合、新しい顧客を作成します。

```php
$stripeCustomer = $user->createOrGetStripeCustomer();
```

<a name="updating-customers"></a>
### 顧客の更新

場合によっては、追加情報を使用してStripe上の顧客情報を直接更新したいことがあります。これは、`updateStripeCustomer`メソッドを使用して行うことができます。このメソッドは、[Stripe APIがサポートする顧客更新オプション](https://stripe.com/docs/api/customers/update)の配列を受け取ります。

```php
$stripeCustomer = $user->updateStripeCustomer($options);
```

<a name="balances"></a>
### 残高

Stripeでは、顧客の「残高」にクレジットまたはデビットを行うことができます。後で、この残高は新しい請求書でクレジットまたはデビットされます。顧客の総残高を確認するには、課金可能なモデルで利用可能な`balance`メソッドを使用できます。`balance`メソッドは、顧客の通貨での残高のフォーマットされた文字列表現を返します。

```php
$balance = $user->balance();
```

顧客の残高にクレジットを行うには、`creditBalance`メソッドに値を提供します。必要に応じて、説明を提供することもできます。

```php
$user->creditBalance(500, 'Premium customer top-up.');
```

`debitBalance`メソッドに値を提供すると、顧客の残高がデビットされます。

```php
$user->debitBalance(300, 'Bad usage penalty.');
```

`applyBalance`メソッドは、顧客の新しい顧客残高トランザクションを作成します。これらのトランザクションレコードを`balanceTransactions`メソッドを使用して取得することができます。これは、顧客が確認するためのクレジットとデビットのログを提供するのに便利です。

```php
// すべてのトランザクションを取得...
$transactions = $user->balanceTransactions();

foreach ($transactions as $transaction) {
    // トランザクション金額...
    $amount = $transaction->amount(); // $2.31

    // 関連する請求書を取得（利用可能な場合）...
    $invoice = $transaction->invoice();
}
```

<a name="tax-ids"></a>
### 税ID

Cashierは、顧客の税IDを管理する簡単な方法を提供します。例えば、`taxIds`メソッドを使用して、顧客に割り当てられたすべての[税ID](https://stripe.com/docs/api/customer_tax_ids/object)をコレクションとして取得できます。

```php
$taxIds = $user->taxIds();
```

顧客の特定の税IDをその識別子で取得することもできます。

```php
$taxId = $user->findTaxId('txi_belgium');
```

有効な[タイプ](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type)と値を`createTaxId`メソッドに提供することで、新しい税IDを作成できます。

```php
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

`createTaxId`メソッドは、VAT IDを顧客のアカウントに即座に追加します。[VAT IDの検証もStripeによって行われます](https://stripe.com/docs/invoicing/customer/tax-ids#validation)が、これは非同期プロセスです。検証の更新については、`customer.tax_id.updated` Webhookイベントを購読し、[VAT IDの`verification`パラメータ](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification)を検査することで通知を受けることができます。Webhookの処理に関する詳細については、[Webhookハンドラの定義に関するドキュメント](#handling-stripe-webhooks)を参照してください。

税IDは、`deleteTaxId`メソッドを使用して削除できます。

```php
$user->deleteTaxId('txi_belgium');
```

<a name="syncing-customer-data-with-stripe"></a>
### 顧客データをStripeと同期する

通常、アプリケーションのユーザーが名前、メールアドレス、またはその他の情報を更新し、それがStripeにも保存されている場合、Stripeに更新を通知する必要があります。これにより、Stripeの情報がアプリケーションの情報と同期します。

これを自動化するには、課金可能なモデルの`updated`イベントに反応するイベントリスナーを定義できます。その後、イベントリスナー内で、モデルの`syncStripeCustomerDetails`メソッドを呼び出すことができます。

```php
use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * The "booted" method of the model.
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
```

これで、顧客モデルが更新されるたびに、その情報がStripeと同期されます。便宜上、Cashierは顧客の初期作成時に自動的に顧客の情報をStripeと同期します。

Stripeとの顧客情報の同期に使用される列をカスタマイズするには、Cashierが提供するさまざまなメソッドをオーバーライドします。例えば、`stripeName`メソッドをオーバーライドして、Cashierが顧客情報をStripeと同期する際に顧客の「名前」と見なすべき属性をカスタマイズできます。

```php
/**
 * Get the customer name that should be synced to Stripe.
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

同様に、`stripeEmail`、`stripePhone`、`stripeAddress`、および`stripePreferredLocales`メソッドをオーバーライドできます。これらのメソッドは、[Stripe顧客オブジェクトの更新](https://stripe.com/docs/api/customers/update)時に対応する顧客パラメータに情報を同期します。顧客情報の同期プロセスを完全に制御したい場合は、`syncStripeCustomerDetails`メソッドをオーバーライドできます。

<a name="billing-portal"></a>
### 請求ポータル

Stripeは、[請求ポータルを簡単に設定する方法](https://stripe.com/docs/billing/subscriptions/customer-portal)を提供しています。これにより、顧客はサブスクリプション、支払い方法、請求履歴を管理できます。コントローラまたはルートから課金可能なモデルの`redirectToBillingPortal`メソッドを呼び出すことで、ユーザーを請求ポータルにリダイレクトできます。

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

デフォルトでは、ユーザーがサブスクリプションの管理を終えると、アプリケーションの`home`ルートに戻るためのリンクがStripe請求ポータル内に表示されます。`redirectToBillingPortal`メソッドに引数としてURLを渡すことで、ユーザーが戻るべきカスタムURLを指定できます。

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

請求ポータルへのURLを生成したいが、HTTPリダイレクトレスポンスを生成したくない場合は、`billingPortalUrl`メソッドを呼び出すことができます:

    $url = $request->user()->billingPortalUrl(route('billing'));

<a name="payment-methods"></a>
## 支払い方法

<a name="storing-payment-methods"></a>
### 支払い方法の保存

Stripeを使用してサブスクリプションを作成したり、「一括」請求を行うには、支払い方法を保存し、その識別子をStripeから取得する必要があります。これを実現するためのアプローチは、支払い方法をサブスクリプションに使用するか、単一の請求に使用するかによって異なります。それぞれについて以下で説明します。

<a name="payment-methods-for-subscriptions"></a>
#### サブスクリプションのための支払い方法

顧客のクレジットカード情報をサブスクリプションで将来使用するために保存する場合、Stripeの「Setup Intents」APIを使用して、顧客の支払い方法の詳細を安全に収集する必要があります。「Setup Intent」は、顧客の支払い方法に請求する意図をStripeに示します。Cashierの`Billable`トレイトには、新しいSetup Intentを簡単に作成するための`createSetupIntent`メソッドが含まれています。このメソッドは、顧客の支払い方法の詳細を収集するフォームをレンダリングするルートまたはコントローラから呼び出す必要があります:

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

Setup Intentを作成し、それをビューに渡した後、そのシークレットを支払い方法を収集する要素に添付する必要があります。例えば、この「支払い方法を更新する」フォームを考えてみましょう:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

次に、Stripe.jsライブラリを使用して、[Stripe Element](https://stripe.com/docs/stripe-js)をフォームに添付し、顧客の支払い詳細を安全に収集することができます:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

次に、カードを検証し、[Stripeの`confirmCardSetup`メソッド](https://stripe.com/docs/js/setup_intents/confirm_card_setup)を使用して、Stripeから安全な「支払い方法識別子」を取得できます:

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // ユーザーに "error.message" を表示...
    } else {
        // カードが正常に検証されました...
    }
});
```

カードがStripeによって検証された後、結果の`setupIntent.payment_method`識別子をLaravelアプリケーションに渡すことができます。これは、顧客に支払い方法を[新しく追加する](#adding-payment-methods)か、[デフォルトの支払い方法を更新する](#updating-the-default-payment-method)ために使用できます。また、支払い方法識別子をすぐに使用して[新しいサブスクリプションを作成する](#creating-subscriptions)こともできます。

> NOTE:  
> Setup Intentsと顧客の支払い詳細の収集について詳しく知りたい場合は、Stripeが提供する[この概要](https://stripe.com/docs/payments/save-and-reuse#php)を確認してください。

<a name="payment-methods-for-single-charges"></a>
#### 単一請求のための支払い方法

もちろん、顧客の支払い方法に対して単一の請求を行う場合、支払い方法識別子を一度だけ使用する必要があります。Stripeの制限により、顧客の保存されたデフォルトの支払い方法を単一の請求に使用することはできません。顧客にStripe.jsライブラリを使用して支払い方法の詳細を入力してもらう必要があります。例えば、次のフォームを考えてみましょう:

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

このようなフォームを定義した後、Stripe.jsライブラリを使用して、[Stripe Element](https://stripe.com/docs/stripe-js)をフォームに添付し、顧客の支払い詳細を安全に収集することができます:

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

次に、カードを検証し、[Stripeの`createPaymentMethod`メソッド](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method)を使用して、Stripeから安全な「支払い方法識別子」を取得できます:

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // ユーザーに "error.message" を表示...
    } else {
        // カードが正常に検証されました...
    }
});
```

カードが正常に検証された場合、`paymentMethod.id`をLaravelアプリケーションに渡し、[単一の請求を処理する](#simple-charge)ことができます。

<a name="retrieving-payment-methods"></a>
### 支払い方法の取得

請求可能なモデルインスタンスの`paymentMethods`メソッドは、`Laravel\Cashier\PaymentMethod`インスタンスのコレクションを返します:

    $paymentMethods = $user->paymentMethods();

デフォルトでは、このメソッドはすべてのタイプの支払い方法を返します。特定のタイプの支払い方法を取得するには、`type`を引数としてメソッドに渡すことができます:

    $paymentMethods = $user->paymentMethods('sepa_debit');

顧客のデフォルトの支払い方法を取得するには、`defaultPaymentMethod`メソッドを使用できます:

    $paymentMethod = $user->defaultPaymentMethod();

請求可能なモデルに添付されている特定の支払い方法を取得するには、`findPaymentMethod`メソッドを使用できます:

    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="payment-method-presence"></a>
### 支払い方法の存在確認

請求可能なモデルがアカウントにデフォルトの支払い方法を添付しているかどうかを確認するには、`hasDefaultPaymentMethod`メソッドを呼び出します:

    if ($user->hasDefaultPaymentMethod()) {
        // ...
    }

請求可能なモデルがアカウントに少なくとも1つの支払い方法を添付しているかどうかを確認するには、`hasPaymentMethod`メソッドを使用できます:

    if ($user->hasPaymentMethod()) {
        // ...
    }

このメソッドは、請求可能なモデルに支払い方法があるかどうかを判断します。特定のタイプの支払い方法が存在するかどうかを判断するには、`type`を引数としてメソッドに渡すことができます:

    if ($user->hasPaymentMethod('sepa_debit')) {
        // ...
    }

<a name="updating-the-default-payment-method"></a>
### デフォルトの支払い方法の更新

`updateDefaultPaymentMethod`メソッドを使用して、顧客のデフォルトの支払い方法情報を更新できます。このメソッドはStripeの支払い方法識別子を受け取り、新しい支払い方法をデフォルトの請求支払い方法として割り当てます:

    $user->updateDefaultPaymentMethod($paymentMethod);

Stripeの顧客のデフォルトの支払い方法情報とデフォルトの支払い方法情報を同期するには、`updateDefaultPaymentMethodFromStripe`メソッドを使用できます:

    $user->updateDefaultPaymentMethodFromStripe();

> WARNING:  
> 顧客のデフォルトの支払い方法は、請求書の発行と新しいサブスクリプションの作成にのみ使用できます。Stripeによる制限により、単一の請求には使用できません。

<a name="adding-payment-methods"></a>
### 支払い方法の追加

新しい支払い方法を追加するには、請求可能なモデルの`addPaymentMethod`メソッドを呼び出し、支払い方法識別子を渡すことができます:

    $user->addPaymentMethod($paymentMethod);

> NOTE:  
> 支払い方法識別子の取得方法については、[支払い方法の保存に関するドキュメント](#storing-payment-methods)を確認してください。

<a name="deleting-payment-methods"></a>
### 支払い方法の削除

支払い方法を削除するには、削除したい`Laravel\Cashier\PaymentMethod`インスタンスの`delete`メソッドを呼び出すことができます:

    $paymentMethod->delete();

請求可能なモデルから特定の支払い方法を削除するには、`deletePaymentMethod`メソッドを使用できます:

    $user->deletePaymentMethod('pm_visa');

請求可能なモデルのすべての支払い方法情報を削除するには、`deletePaymentMethods`メソッドを使用できます:

    $user->deletePaymentMethods();

デフォルトでは、このメソッドはすべてのタイプの支払い方法を削除します。特定のタイプの支払い方法を削除するには、`type`を引数としてメソッドに渡すことができます:

    $user->deletePaymentMethods('sepa_debit');

> WARNING:  
> ユーザーがアクティブなサブスクリプションを持っている場合、アプリケーションはデフォルトの支払い方法を削除することを許可しないでください。

<a name="subscriptions"></a>
## サブスクリプション

サブスクリプションは、顧客に定期的な支払いを設定する方法を提供します。Stripeのサブスクリプションは、Cashierによって管理され、複数のサブスクリプション価格、サブスクリプション数量、トライアルなどをサポートしています。

<a name="creating-subscriptions"></a>
### サブスクリプションの作成

サブスクリプションを作成するには、まず請求可能なモデルのインスタンスを取得します。これは通常、`App\Models\User`のインスタンスになります。モデルインスタンスを取得したら、`newSubscription`メソッドを使用してモデルのサブスクリプションを作成できます:

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription(
        'default', 'price_monthly'
    )->create($request->paymentMethodId);

    // ...
});
```

`newSubscription` メソッドに渡される最初の引数は、サブスクリプションの内部タイプです。アプリケーションが単一のサブスクリプションのみを提供する場合、これを `default` または `primary` と呼ぶかもしれません。このサブスクリプションタイプは、内部アプリケーションの使用のみを目的としており、ユーザーに表示されることを意図していません。さらに、スペースを含めず、サブスクリプションを作成した後に変更してはいけません。2番目の引数は、ユーザーがサブスクライブしている特定の価格です。この値は、Stripe における価格の識別子に対応している必要があります。

[Stripe の支払い方法識別子](#storing-payment-methods)または Stripe の `PaymentMethod` オブジェクトを受け入れる `create` メソッドは、サブスクリプションを開始し、請求可能なモデルの Stripe 顧客 ID やその他の関連する請求情報でデータベースを更新します。

> WARNING:  
> 支払い方法識別子を直接 `create` サブスクリプションメソッドに渡すと、自動的にユーザーの保存された支払い方法に追加されます。

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### 請求書メールによる定期支払いの回収

顧客の定期支払いを自動的に回収する代わりに、Stripe に顧客に請求書をメールで送信し、定期支払いが期日になるたびに顧客が手動で請求書を支払うように指示することができます。顧客は、請求書による定期支払いを回収する際に、事前に支払い方法を提供する必要はありません。

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice();
```

顧客がサブスクリプションがキャンセルされる前に請求書を支払うために与えられる時間は、`days_until_due` オプションによって決定されます。デフォルトでは30日ですが、このオプションに特定の値を提供することもできます。

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
    'days_until_due' => 30
]);
```

<a name="subscription-quantities"></a>
#### 数量

サブスクリプションを作成する際に価格に特定の[数量](https://stripe.com/docs/billing/subscriptions/quantities)を設定したい場合は、サブスクリプションビルダーで `quantity` メソッドを呼び出してからサブスクリプションを作成する必要があります。

```php
$user->newSubscription('default', 'price_monthly')
     ->quantity(5)
     ->create($paymentMethod);
```

<a name="additional-details"></a>
#### 追加の詳細

Stripe がサポートする追加の[顧客](https://stripe.com/docs/api/customers/create)または[サブスクリプション](https://stripe.com/docs/api/subscriptions/create)オプションを指定したい場合は、それらを `create` メソッドの2番目と3番目の引数として渡すことができます。

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Some extra information.'],
]);
```

<a name="coupons"></a>
#### クーポン

サブスクリプションを作成する際にクーポンを適用したい場合は、`withCoupon` メソッドを使用できます。

```php
$user->newSubscription('default', 'price_monthly')
     ->withCoupon('code')
     ->create($paymentMethod);
```

または、[Stripe のプロモーションコード](https://stripe.com/docs/billing/subscriptions/discounts/codes)を適用したい場合は、`withPromotionCode` メソッドを使用できます。

```php
$user->newSubscription('default', 'price_monthly')
     ->withPromotionCode('promo_code_id')
     ->create($paymentMethod);
```

指定されたプロモーションコード ID は、プロモーションコードに割り当てられた Stripe API ID である必要があり、顧客向けのプロモーションコードではありません。顧客向けのプロモーションコードに基づいてプロモーションコード ID を見つける必要がある場合は、`findPromotionCode` メソッドを使用できます。

```php
// 顧客向けのコードでプロモーションコード ID を見つける...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// 顧客向けのコードでアクティブなプロモーションコード ID を見つける...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

上記の例では、返される `$promotionCode` オブジェクトは `Laravel\Cashier\PromotionCode` のインスタンスです。このクラスは、基礎となる `Stripe\PromotionCode` オブジェクトを装飾します。プロモーションコードに関連するクーポンを取得するには、`coupon` メソッドを呼び出すことができます。

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

クーポンインスタンスを使用して、割引額やクーポンが固定割引かパーセントベースの割引かを判断できます。

```php
if ($coupon->isPercentage()) {
    return $coupon->percentOff().'%'; // 21.5%
} else {
    return $coupon->amountOff(); // $5.99
}
```

また、現在顧客またはサブスクリプションに適用されている割引を取得することもできます。

```php
$discount = $billable->discount();

$discount = $subscription->discount();
```

返される `Laravel\Cashier\Discount` インスタンスは、基礎となる `Stripe\Discount` オブジェクトインスタンスを装飾します。この割引に関連するクーポンを取得するには、`coupon` メソッドを呼び出すことができます。

```php
$coupon = $subscription->discount()->coupon();
```

顧客またはサブスクリプションに新しいクーポンまたはプロモーションコードを適用したい場合は、`applyCoupon` または `applyPromotionCode` メソッドを介して行うことができます。

```php
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');

$subscription->applyCoupon('coupon_id');
$subscription->applyPromotionCode('promotion_code_id');
```

覚えておいてください。プロモーションコードに割り当てられた Stripe API ID を使用し、顧客向けのプロモーションコードではありません。特定の時点で顧客またはサブスクリプションに適用できるクーポンまたはプロモーションコードは1つだけです。

この件に関する詳細は、Stripe のドキュメントを参照してください。[クーポン](https://stripe.com/docs/billing/subscriptions/coupons)と[プロモーションコード](https://stripe.com/docs/billing/subscriptions/coupons/codes)に関するドキュメントです。

<a name="adding-subscriptions"></a>
#### サブスクリプションの追加

顧客がすでにデフォルトの支払い方法を持っている場合、サブスクリプションビルダーで `add` メソッドを呼び出して、サブスクリプションを追加することができます。

```php
use App\Models\User;

$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->add();
```

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### Stripe ダッシュボードからのサブスクリプションの作成

Stripe ダッシュボード自体からサブスクリプションを作成することもできます。その場合、Cashier は新しく追加されたサブスクリプションを同期し、それらに `default` タイプを割り当てます。ダッシュボードで作成されたサブスクリプションに割り当てられるサブスクリプションタイプをカスタマイズするには、[webhook イベントハンドラを定義](#defining-webhook-event-handlers)します。

さらに、Stripe ダッシュボードを介して1つのタイプのサブスクリプションのみを作成できます。アプリケーションが異なるタイプを使用する複数のサブスクリプションを提供する場合、Stripe ダッシュボードを介して追加できるのは1つのタイプのサブスクリプションのみです。

最後に、アプリケーションが提供するサブスクリプションタイプごとに1つのアクティブなサブスクリプションのみを追加するようにしてください。顧客が2つの `default` サブスクリプションを持っている場合、Cashier は最新に追加されたサブスクリプションのみを使用しますが、両方ともアプリケーションのデータベースと同期されます。

<a name="checking-subscription-status"></a>
### サブスクリプションステータスの確認

顧客がアプリケーションにサブスクライブした後、さまざまな便利なメソッドを使用してサブスクリプションステータスを簡単に確認できます。まず、`subscribed` メソッドは、顧客がアクティブなサブスクリプションを持っている場合、たとえサブスクリプションが現在試用期間内であっても `true` を返します。`subscribed` メソッドは、サブスクリプションのタイプを最初の引数として受け取ります。

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` メソッドは、[ルートミドルウェア](middleware.md)に適した候補でもあり、ユーザーのサブスクリプションステータスに基づいてルートとコントローラへのアクセスをフィルタリングできます。

```php
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
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // このユーザーは有料顧客ではありません...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

ユーザーがまだ試用期間内かどうかを判断したい場合は、`onTrial` メソッドを使用できます。このメソッドは、ユーザーがまだ試用期間内であることを示す警告を表示する必要があるかどうかを判断するのに役立ちます。

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

`subscribedToProduct` メソッドは、ユーザーが指定された Stripe 製品の識別子に基づいて特定の製品にサブスクライブしているかどうかを判断するために使用できます。Stripe では、製品は価格のコレクションです。この例では、ユーザーの `default` サブスクリプションがアプリケーションの "premium" 製品にアクティブにサブスクライブしているかどうかを判断します。指定された Stripe 製品識別子は、Stripe ダッシュボード内の製品の識別子の1つに対応している必要があります。

```php
if ($user->subscribedToProduct('prod_premium', 'default')) {
    // ...
}
```

`subscribedToProduct` メソッドに配列を渡すことで、ユーザーの `default` サブスクリプションがアプリケーションの "basic" または "premium" 製品にアクティブにサブスクライブしているかどうかを判断できます。

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

`subscribedToPrice`メソッドは、顧客のサブスクリプションが特定の価格IDに対応しているかどうかを判断するために使用できます:

    if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
        // ...
    }

`recurring`メソッドは、ユーザーが現在サブスクライブしており、試用期間を超えているかどうかを判断するために使用できます:

    if ($user->subscription('default')->recurring()) {
        // ...
    }

> WARNING:  
> ユーザーが同じタイプの2つのサブスクリプションを持っている場合、`subscription`メソッドは常に最新のサブスクリプションを返します。例えば、ユーザーが`default`というタイプの2つのサブスクリプションレコードを持っているかもしれません。しかし、そのうちの1つは古く、期限切れのサブスクリプションであり、もう1つは現在、アクティブなサブスクリプションです。最新のサブスクリプションが常に返され、古いサブスクリプションはデータベースに保持されて履歴の確認に使用されます。

<a name="cancelled-subscription-status"></a>
#### キャンセルされたサブスクリプションのステータス

ユーザーが一度アクティブなサブスクライバーであったが、サブスクリプションをキャンセルしたかどうかを判断するには、`canceled`メソッドを使用できます:

    if ($user->subscription('default')->canceled()) {
        // ...
    }

また、ユーザーがサブスクリプションをキャンセルしたが、まだサブスクリプションが完全に期限切れになるまでの「猶予期間」にいるかどうかを判断することもできます。例えば、ユーザーが3月5日にサブスクリプションをキャンセルし、元々3月10日に期限切れになる予定であった場合、ユーザーは3月10日まで「猶予期間」にいます。この間、`subscribed`メソッドは依然として`true`を返します:

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

ユーザーがサブスクリプションをキャンセルし、「猶予期間」を超えているかどうかを判断するには、`ended`メソッドを使用できます:

    if ($user->subscription('default')->ended()) {
        // ...
    }

<a name="incomplete-and-past-due-status"></a>
#### 不完全および過去のステータス

サブスクリプションの作成後に二次的な支払いアクションが必要な場合、サブスクリプションは`incomplete`とマークされます。サブスクリプションのステータスは、Cashierの`subscriptions`データベーステーブルの`stripe_status`列に保存されます。

同様に、価格を変更する際に二次的な支払いアクションが必要な場合、サブスクリプションは`past_due`とマークされます。サブスクリプションがこれらの状態のいずれかにある場合、顧客が支払いを確認するまでアクティブになりません。請求可能なモデルまたはサブスクリプションインスタンスの`hasIncompletePayment`メソッドを使用して、サブスクリプションに不完全な支払いがあるかどうかを判断できます:

    if ($user->hasIncompletePayment('default')) {
        // ...
    }

    if ($user->subscription('default')->hasIncompletePayment()) {
        // ...
    }

サブスクリプションに不完全な支払いがある場合、ユーザーをCashierの支払い確認ページに誘導し、`latestPayment`識別子を渡す必要があります。サブスクリプションインスタンスで利用可能な`latestPayment`メソッドを使用して、この識別子を取得できます:

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    支払いを確認してください。
</a>
```

サブスクリプションが`past_due`または`incomplete`状態にある場合でも、サブスクリプションをアクティブと見なしたい場合は、Cashierが提供する`keepPastDueSubscriptionsActive`および`keepIncompleteSubscriptionsActive`メソッドを使用できます。通常、これらのメソッドは`App\Providers\AppServiceProvider`の`register`メソッドで呼び出す必要があります:

    use Laravel\Cashier\Cashier;

    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        Cashier::keepPastDueSubscriptionsActive();
        Cashier::keepIncompleteSubscriptionsActive();
    }

> WARNING:  
> サブスクリプションが`incomplete`状態にある場合、支払いが確認されるまで変更できません。したがって、`swap`および`updateQuantity`メソッドは、サブスクリプションが`incomplete`状態にある場合に例外をスローします。

<a name="subscription-scopes"></a>
#### サブスクリプションのスコープ

ほとんどのサブスクリプションのステータスは、クエリスコープとしても利用可能であり、特定の状態のサブスクリプションを簡単にデータベースからクエリできます:

    // すべてのアクティブなサブスクリプションを取得...
    $subscriptions = Subscription::query()->active()->get();

    // ユーザーのキャンセルされたすべてのサブスクリプションを取得...
    $subscriptions = $user->subscriptions()->canceled()->get();

利用可能なスコープの完全なリストは以下の通りです:

    Subscription::query()->active();
    Subscription::query()->canceled();
    Subscription::query()->ended();
    Subscription::query()->incomplete();
    Subscription::query()->notCanceled();
    Subscription::query()->notOnGracePeriod();
    Subscription::query()->notOnTrial();
    Subscription::query()->onGracePeriod();
    Subscription::query()->onTrial();
    Subscription::query()->pastDue();
    Subscription::query()->recurring();

<a name="changing-prices"></a>
### 価格の変更

顧客がアプリケーションにサブスクライブした後、新しいサブスクリプション価格に変更したい場合があります。顧客を新しい価格に変更するには、Stripe価格の識別子を`swap`メソッドに渡します。価格を変更する際、ユーザーが以前にキャンセルしたサブスクリプションを再アクティブ化したいと想定されます。指定された価格識別子は、Stripeダッシュボードで利用可能なStripe価格識別子に対応する必要があります:

    use App\Models\User;

    $user = App\Models\User::find(1);

    $user->subscription('default')->swap('price_yearly');

顧客が試用期間中である場合、試用期間は維持されます。また、サブスクリプションに「数量」が存在する場合、その数量も維持されます。

価格を変更し、顧客が現在の試用期間をキャンセルする場合は、`skipTrial`メソッドを呼び出すことができます:

    $user->subscription('default')
            ->skipTrial()
            ->swap('price_yearly');

価格を変更し、顧客に次の請求サイクルを待たずに即座に請求する場合は、`swapAndInvoice`メソッドを使用できます:

    $user = User::find(1);

    $user->subscription('default')->swapAndInvoice('price_yearly');

<a name="prorations"></a>
#### 按分計算

デフォルトでは、Stripeは価格間の変更時に按分計算された料金を請求します。`noProrate`メソッドを使用して、按分計算された料金を請求せずにサブスクリプションの価格を更新できます:

    $user->subscription('default')->noProrate()->swap('price_yearly');

サブスクリプションの按分計算についての詳細は、[Stripeのドキュメント](https://stripe.com/docs/billing/subscriptions/prorations)を参照してください。

> WARNING:  
> `noProrate`メソッドを`swapAndInvoice`メソッドの前に実行しても、按分計算には影響しません。請求書は常に発行されます。

<a name="subscription-quantity"></a>
### サブスクリプションの数量

サブスクリプションは、「数量」によって影響を受けることがあります。例えば、プロジェクト管理アプリケーションがプロジェクトごとに月額$10を請求する場合があります。`incrementQuantity`および`decrementQuantity`メソッドを使用して、サブスクリプションの数量を簡単に増減できます:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // サブスクリプションの現在の数量に5を追加...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // サブスクリプションの現在の数量から5を減算...
    $user->subscription('default')->decrementQuantity(5);

または、`updateQuantity`メソッドを使用して特定の数量を設定することもできます:

    $user->subscription('default')->updateQuantity(10);

`noProrate`メソッドを使用して、按分計算された料金を請求せずにサブスクリプションの数量を更新できます:

    $user->subscription('default')->noProrate()->updateQuantity(10);

サブスクリプションの数量についての詳細は、[Stripeのドキュメント](https://stripe.com/docs/subscriptions/quantities)を参照してください。

<a name="quantities-for-subscription-with-multiple-products"></a>
#### 複数の製品を持つサブスクリプションの数量

サブスクリプションが[複数の製品を持つサブスクリプション](#subscriptions-with-multiple-products)である場合、数量を増減する価格のIDを増減メソッドの第2引数として渡す必要があります:

    $user->subscription('default')->incrementQuantity(1, 'price_chat');

<a name="subscriptions-with-multiple-products"></a>
### 複数の製品を持つサブスクリプション

[複数の製品を持つサブスクリプション](https://stripe.com/docs/billing/subscriptions/multiple-products)を使用すると、単一のサブスクリプションに複数の請求製品を割り当てることができます。例えば、顧客サービス「ヘルプデスク」アプリケーションが月額$10の基本サブスクリプション価格を持ち、ライブチャットアドオン製品を追加で月額$15で提供する場合を想像してください。複数の製品を持つサブスクリプションの情報は、Cashierの`subscription_items`データベーステーブルに保存されます。

`newSubscription`メソッドに価格の配列を第2引数として渡すことで、特定のサブスクリプションに複数の製品を指定できます:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', [
            'price_monthly',
            'price_chat',
        ])->create($request->paymentMethodId);

        // ...
    });

上記の例では、顧客は`default`サブスクリプションに2つの価格が添付されます。それぞれの価格は、それぞれの請求間隔で請求されます。必要に応じて、`quantity`メソッドを使用して各価格の特定の数量を示すことができます:

    $user = User::find(1);

    $user->newSubscription('default', ['price_monthly', 'price_chat'])
        ->quantity(5, 'price_chat')
        ->create($paymentMethod);

既存のサブスクリプションに別の価格を追加する場合は、サブスクリプションの`addPrice`メソッドを呼び出すことができます:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat');

上記の例では、新しい価格を追加し、次の請求サイクルで顧客に請求されます。顧客に即座に請求したい場合は、`addPriceAndInvoice`メソッドを使用できます:

    $user->subscription('default')->addPriceAndInvoice('price_chat');

特定の数量で価格を追加したい場合は、`addPrice`または`addPriceAndInvoice`メソッドの第2引数として数量を渡すことができます:

    $user = User::find(1);

    $user->subscription('default')->addPrice('price_chat', 5);

サブスクリプションから価格を削除するには、`removePrice`メソッドを使用します:

    $user->subscription('default')->removePrice('price_chat');

> WARNING:  
> サブスクリプションの最後の価格を削除することはできません。代わりに、サブスクリプションをキャンセルする必要があります。

<a name="swapping-prices"></a>
#### 価格の入れ替え

複数の製品が添付されたサブスクリプションの価格を変更することもできます。例えば、顧客が`price_basic`サブスクリプションと`price_chat`アドオン製品を持っており、顧客を`price_basic`から`price_pro`にアップグレードしたいとします:

    use App\Models\User;

    $user = User::find(1);

    $user->subscription('default')->swap(['price_pro', 'price_chat']);

上記の例を実行すると、`price_basic`の基礎となるサブスクリプションアイテムが削除され、`price_chat`のアイテムが保持されます。さらに、`price_pro`の新しいサブスクリプションアイテムが作成されます。

サブスクリプションアイテムのオプションを指定するには、`swap`メソッドにキー/値のペアの配列を渡すことができます。例えば、サブスクリプション価格の数量を指定する必要がある場合:

    $user = User::find(1);

    $user->subscription('default')->swap([
        'price_pro' => ['quantity' => 5],
        'price_chat'
    ]);

サブスクリプションの単一の価格を入れ替えたい場合は、サブスクリプションアイテム自体の`swap`メソッドを使用して行うことができます。このアプローチは、サブスクリプションの他の価格の既存のメタデータをすべて保持したい場合に特に便利です:

    $user = User::find(1);

    $user->subscription('default')
            ->findItemOrFail('price_basic')
            ->swap('price_pro');

<a name="proration"></a>
#### 日割り計算

デフォルトでは、Stripeは複数の製品を持つサブスクリプションから価格を追加または削除する際に日割り請求を行います。日割り計算なしで価格調整を行いたい場合は、価格操作に`noProrate`メソッドを連鎖させる必要があります:

    $user->subscription('default')->noProrate()->removePrice('price_chat');

<a name="swapping-quantities"></a>
#### 数量の更新

個々のサブスクリプション価格の数量を更新したい場合は、価格のIDをメソッドに追加の引数として渡すことで、[既存の数量メソッド](#subscription-quantity)を使用して行うことができます:


    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'price_chat');

    $user->subscription('default')->decrementQuantity(3, 'price_chat');

    $user->subscription('default')->updateQuantity(10, 'price_chat');

> WARNING:  
> サブスクリプションに複数の価格がある場合、`Subscription`モデルの`stripe_price`および`quantity`属性は`null`になります。個々の価格属性にアクセスするには、`Subscription`モデルで利用可能な`items`リレーションを使用する必要があります。

<a name="subscription-items"></a>
#### サブスクリプションアイテム

サブスクリプションに複数の価格がある場合、データベースの`subscription_items`テーブルに複数のサブスクリプション「アイテム」が保存されます。これらには、サブスクリプションの`items`リレーションを介してアクセスできます:

    use App\Models\User;

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // 特定のアイテムのStripe価格と数量を取得...
    $stripePrice = $subscriptionItem->stripe_price;
    $quantity = $subscriptionItem->quantity;
```

特定の価格を取得するには、`findItemOrFail`メソッドを使用することもできます:

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');

<a name="multiple-subscriptions"></a>
### 複数のサブスクリプション

Stripeでは、顧客が同時に複数のサブスクリプションを持つことができます。例えば、ジムを運営しており、水泳サブスクリプションとダンベルサブスクリプションを提供しているとします。それぞれのサブスクリプションは異なる価格設定を持つことができます。もちろん、顧客はどちらか一方または両方のプランに加入できるべきです。

アプリケーションがサブスクリプションを作成する際、`newSubscription`メソッドにサブスクリプションのタイプを指定できます。タイプは、ユーザーが開始しているサブスクリプションのタイプを表す任意の文字列です:

    use Illuminate\Http\Request;

    Route::post('/swimming/subscribe', function (Request $request) {
        $request->user()->newSubscription('swimming')
            ->price('price_swimming_monthly')
            ->create($request->paymentMethodId);

        // ...
    });

この例では、顧客のために月次の水泳サブスクリプションを開始しました。しかし、後で年次サブスクリプションに切り替えたい場合があります。顧客のサブスクリプションを調整する際、`swimming`サブスクリプションの価格を入れ替えるだけです:

    $user->subscription('swimming')->swap('price_swimming_yearly');

もちろん、サブスクリプションを完全にキャンセルすることもできます:

    $user->subscription('swimming')->cancel();

<a name="metered-billing"></a>
### 従量課金制

[従量課金制](https://stripe.com/docs/billing/subscriptions/metered-billing)を使用すると、請求サイクル中の製品使用量に基づいて顧客に請求できます。例えば、顧客が月に送信するテキストメッセージやメールの数に基づいて請求することができます。

従量課金制を開始するには、まずStripeダッシュボードで従量課金制の価格を持つ新しい製品を作成する必要があります。次に、`meteredPrice`を使用して、従量課金制の価格IDを顧客のサブスクリプションに追加します:

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default')
            ->meteredPrice('price_metered')
            ->create($request->paymentMethodId);

        // ...
    });

[Stripe Checkout](#checkout)を介して従量課金制のサブスクリプションを開始することもできます:

    $checkout = Auth::user()
            ->newSubscription('default', [])
            ->meteredPrice('price_metered')
            ->checkout();

    return view('your-checkout-view', [
        'checkout' => $checkout,
    ]);

<a name="reporting-usage"></a>
#### 使用量の報告

顧客がアプリケーションを使用すると、正確に請求できるようにStripeに使用量を報告する必要があります。従量課金制のサブスクリプションの使用量を増やすには、`reportUsage`メソッドを使用できます:

    $user = User::find(1);

    $user->subscription('default')->reportUsage();

デフォルトでは、請求期間に1の「使用量」が追加されます。代わりに、顧客の請求期間に追加する特定の「使用量」を渡すことができます:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(15);

アプリケーションが単一のサブスクリプションで複数の価格を提供する場合、使用量を報告する従量課金制の価格を指定するために`reportUsageFor`メソッドを使用する必要があります:

    $user = User::find(1);

    $user->subscription('default')->reportUsageFor('price_metered', 15);

場合によっては、以前に報告した使用量を更新する必要があります。これを行うには、`reportUsage`の第2引数としてタイムスタンプまたは`DateTimeInterface`インスタンスを渡すことができます。そうすると、Stripeはその時点で報告された使用量を更新します。指定された日付と時刻が現在の請求期間内にある限り、以前の使用量レコードを更新し続けることができます:

    $user = User::find(1);

    $user->subscription('default')->reportUsage(5, $timestamp);

<a name="retrieving-usage-records"></a>
#### 使用量レコードの取得

顧客の過去の使用量を取得するには、サブスクリプションインスタンスの`usageRecords`メソッドを使用できます:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecords();

アプリケーションが単一のサブスクリプションで複数の価格を提供する場合、使用量レコードを取得する従量課金制の価格を指定するために`usageRecordsFor`メソッドを使用する必要があります:

    $user = User::find(1);

    $usageRecords = $user->subscription('default')->usageRecordsFor('price_metered');

`usageRecords`および`usageRecordsFor`メソッドは、使用量レコードの連想配列を含むCollectionインスタンスを返します。この配列を反復処理して、顧客の総使用量を表示できます:

    @foreach ($usageRecords as $usageRecord)
        - 期間開始: {{ $usageRecord['period']['start'] }}
        - 期間終了: {{ $usageRecord['period']['end'] }}
        - 総使用量: {{ $usageRecord['total_usage'] }}
    @endforeach

使用量データの完全なリファレンスとStripeのカーソルベースのページネーションの使用方法については、[公式のStripe APIドキュメント](https://stripe.com/docs/api/usage_records/subscription_item_summary_list)を参照してください。

<a name="subscription-taxes"></a>
### サブスクリプションの税金

> WARNING:  
> 手動で税率を計算する代わりに、[Stripe Taxを使用して自動的に税金を計算](#tax-configuration)できます。

ユーザーがサブスクリプションに支払う税金を指定するには、課金可能なモデルに`taxRates`メソッドを実装し、Stripeの税率IDを含む配列を返す必要があります。これらの税率は、[Stripeダッシュボード](https://dashboard.stripe.com/test/tax-rates)で定義できます:

    /**
     * 顧客のサブスクリプションに適用される税金。
     *
     * @return array<int, string>
     */
    public function taxRates(): array
    {
        return ['txr_id'];
    }

`taxRates`メソッドを使用すると、顧客ごとに税率を適用できます。これは、複数の国と税率を持つユーザーベースに役立つかもしれません。

複数の製品を含むサブスクリプションを提供している場合、請求可能なモデルに`priceTaxRates`メソッドを実装することで、各価格に異なる税率を定義できます：

    /**
     * 顧客のサブスクリプションに適用されるべき税率
     *
     * @return array<string, array<int, string>>
     */
    public function priceTaxRates(): array
    {
        return [
            'price_monthly' => ['txr_id'],
        ];
    }

> WARNING:  
> `taxRates`メソッドはサブスクリプション料金にのみ適用されます。「一括請求」を行うためにCashierを使用している場合、その時点で税率を手動で指定する必要があります。

<a name="syncing-tax-rates"></a>
#### 税率の同期

`taxRates`メソッドが返すハードコードされた税率IDを変更する場合、ユーザーの既存のサブスクリプションの税設定はそのまま残ります。既存のサブスクリプションの税額を新しい`taxRates`の値で更新したい場合は、ユーザーのサブスクリプションインスタンスで`syncTaxRates`メソッドを呼び出す必要があります：

    $user->subscription('default')->syncTaxRates();

これにより、複数の製品を持つサブスクリプションのアイテム税率も同期されます。アプリケーションが複数の製品を持つサブスクリプションを提供している場合、請求可能なモデルが[前述](#subscription-taxes)の`priceTaxRates`メソッドを実装していることを確認する必要があります。

<a name="tax-exemption"></a>
#### 税免除

Cashierは、顧客が税免除かどうかを判断するための`isNotTaxExempt`、`isTaxExempt`、および`reverseChargeApplies`メソッドも提供します。これらのメソッドは、顧客の税免除ステータスを決定するためにStripe APIを呼び出します：

    use App\Models\User;

    $user = User::find(1);

    $user->isTaxExempt();
    $user->isNotTaxExempt();
    $user->reverseChargeApplies();

> WARNING:  
> これらのメソッドは、`Laravel\Cashier\Invoice`オブジェクトでも利用可能です。ただし、`Invoice`オブジェクトで呼び出される場合、メソッドは請求書が作成された時点での免除ステータスを決定します。

<a name="subscription-anchor-date"></a>
### サブスクリプションのアンカー日付

デフォルトでは、請求サイクルのアンカーはサブスクリプションが作成された日付、または試用期間が使用されている場合は試用が終了する日付です。請求アンカー日付を変更したい場合は、`anchorBillingCycleOn`メソッドを使用できます：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $anchor = Carbon::parse('first day of next month');

        $request->user()->newSubscription('default', 'price_monthly')
                    ->anchorBillingCycleOn($anchor->startOfDay())
                    ->create($request->paymentMethodId);

        // ...
    });

サブスクリプションの請求サイクルを管理するための詳細については、[Stripeの請求サイクルドキュメント](https://stripe.com/docs/billing/subscriptions/billing-cycle)を参照してください。

<a name="cancelling-subscriptions"></a>
### サブスクリプションのキャンセル

サブスクリプションをキャンセルするには、ユーザーのサブスクリプションで`cancel`メソッドを呼び出します：

    $user->subscription('default')->cancel();

サブスクリプションがキャンセルされると、Cashierは自動的に`subscriptions`データベーステーブルの`ends_at`カラムを設定します。このカラムは、`subscribed`メソッドがいつ`false`を返すべきかを知るために使用されます。

例えば、顧客が3月1日にサブスクリプションをキャンセルしたが、サブスクリプションは3月5日まで終了予定でなかった場合、`subscribed`メソッドは3月5日まで`true`を返し続けます。これは、通常、顧客が請求サイクルの終了までアプリケーションを使用し続けることが許可されているためです。

`onGracePeriod`メソッドを使用して、ユーザーがサブスクリプションをキャンセルしたが、まだ「猶予期間」にいるかどうかを判断できます：

    if ($user->subscription('default')->onGracePeriod()) {
        // ...
    }

サブスクリプションを即座にキャンセルしたい場合は、ユーザーのサブスクリプションで`cancelNow`メソッドを呼び出します：

    $user->subscription('default')->cancelNow();

サブスクリプションを即座にキャンセルし、未請求の従量制使用料金や新規/保留中の按分請求項目を請求したい場合は、ユーザーのサブスクリプションで`cancelNowAndInvoice`メソッドを呼び出します：

    $user->subscription('default')->cancelNowAndInvoice();

特定の時点でサブスクリプションをキャンセルすることもできます：

    $user->subscription('default')->cancelAt(
        now()->addDays(10)
    );

最後に、関連するユーザーモデルを削除する前に、常にユーザーサブスクリプションをキャンセルする必要があります：

    $user->subscription('default')->cancelNow();

    $user->delete();

<a name="resuming-subscriptions"></a>
### サブスクリプションの再開

顧客がサブスクリプションをキャンセルし、再開したい場合は、サブスクリプションで`resume`メソッドを呼び出すことができます。顧客は、サブスクリプションを再開するためにまだ「猶予期間」内でなければなりません：

    $user->subscription('default')->resume();

顧客がサブスクリプションをキャンセルし、サブスクリプションが完全に期限切れになる前にそのサブスクリプションを再開する場合、顧客はすぐに請求されません。代わりに、サブスクリプションは再アクティブ化され、元の請求サイクルで請求されます。

<a name="subscription-trials"></a>
## サブスクリプションの試用期間

<a name="with-payment-method-up-front"></a>
### 事前に支払い方法を収集する

顧客に試用期間を提供しながら、支払い方法の情報を事前に収集したい場合、サブスクリプションを作成する際に`trialDays`メソッドを使用する必要があります：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription('default', 'price_monthly')
                    ->trialDays(10)
                    ->create($request->paymentMethodId);

        // ...
    });

このメソッドは、データベース内のサブスクリプションレコードに試用期間の終了日を設定し、Stripeにこの日付まで顧客への請求を開始しないよう指示します。`trialDays`メソッドを使用すると、CashierはStripeで設定された価格のデフォルトの試用期間を上書きします。

> WARNING:  
> 顧客のサブスクリプションが試用終了日の前にキャンセルされない場合、試用が終了するとすぐに請求されますので、ユーザーに試用終了日を通知することを確認してください。

`trialUntil`メソッドを使用すると、試用期間がいつ終了するかを指定する`DateTime`インスタンスを提供できます：

    use Carbon\Carbon;

    $user->newSubscription('default', 'price_monthly')
                ->trialUntil(Carbon::now()->addDays(10))
                ->create($paymentMethod);

ユーザーが試用期間内かどうかを判断するには、ユーザーインスタンスの`onTrial`メソッドまたはサブスクリプションインスタンスの`onTrial`メソッドを使用できます。以下の2つの例は同等です：

    if ($user->onTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->onTrial()) {
        // ...
    }

サブスクリプションの試用を即座に終了させるには、`endTrial`メソッドを使用できます：

    $user->subscription('default')->endTrial();

既存の試用期間が期限切れかどうかを判断するには、`hasExpiredTrial`メソッドを使用できます：

    if ($user->hasExpiredTrial('default')) {
        // ...
    }

    if ($user->subscription('default')->hasExpiredTrial()) {
        // ...
    }

<a name="defining-trial-days-in-stripe-cashier"></a>
#### Stripe / Cashierでの試用日数の定義

Stripeダッシュボードで価格の試用日数を定義するか、常にCashierを使用して明示的に渡すかを選択できます。Stripeで価格の試用日数を定義することを選択した場合、過去にサブスクリプションを持っていた顧客の新しいサブスクリプションを含む新しいサブスクリプションは、`skipTrial()`メソッドを明示的に呼び出さない限り、常に試用期間を受け取ることに注意してください。

<a name="without-payment-method-up-front"></a>
### 事前に支払い方法を収集しない

顧客の支払い方法の情報を事前に収集せずに試用期間を提供したい場合、ユーザーレコードの`trial_ends_at`カラムを希望する試用終了日に設定できます。これは通常、ユーザー登録時に行われます：

    use App\Models\User;

    $user = User::create([
        // ...
        'trial_ends_at' => now()->addDays(10),
    ]);

> WARNING:  
> 請求可能なモデルのクラス定義内で`trial_ends_at`属性に[日付キャスト](eloquent-mutators.md#date-casting)を追加することを確認してください。

Cashierはこのタイプの試用を「汎用試用」と呼びます。これは、既存のサブスクリプションに添付されていないためです。請求可能なモデルインスタンスの`onTrial`メソッドは、現在の日付が`trial_ends_at`の値を過ぎていない場合に`true`を返します：

    if ($user->onTrial()) {
        // ユーザーは試用期間内です...
    }

ユーザーの準備ができたら、通常どおり`newSubscription`メソッドを使用して実際のサブスクリプションを作成できます：

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->create($paymentMethod);

ユーザーの試用終了日を取得するには、`trialEndsAt`メソッドを使用できます。このメソッドは、ユーザーが試用期間内であればCarbon日付インスタンスを返し、そうでなければ`null`を返します。また、デフォルト以外の特定のサブスクリプションの試用終了日を取得したい場合は、オプションのサブスクリプションタイプパラメータを渡すこともできます：

    if ($user->onTrial()) {
        $trialEndsAt = $user->trialEndsAt('main');
    }

ユーザーがまだ「汎用」試用期間内であり、実際のサブスクリプションをまだ作成していないかどうかを知りたい場合は、`onGenericTrial`メソッドを使用することもできます：

    if ($user->onGenericTrial()) {
        // ユーザーは「汎用」試用期間内です...
    }

<a name="extending-trials"></a>
### 試用期間の延長

`extendTrial`メソッドを使用すると、サブスクリプションが作成された後にサブスクリプションの試用期間を延長することができます。試用期間が既に終了し、顧客が既にサブスクリプションの請求を受けている場合でも、延長された試用期間を提供することができます。試用期間内に費やした時間は、顧客の次の請求書から差し引かれます。

    use App\Models\User;

    $subscription = User::find(1)->subscription('default');

    // 試用期間を今から7日後に終了させる...
    $subscription->extendTrial(
        now()->addDays(7)
    );

    // 試用期間にさらに5日追加する...
    $subscription->extendTrial(
        $subscription->trial_ends_at->addDays(5)
    );

<a name="handling-stripe-webhooks"></a>
## Stripe Webhookの処理

> NOTE:  
> ローカル開発中にWebhookをテストするために、[Stripe CLI](https://stripe.com/docs/stripe-cli)を使用することができます。

Stripeは、Webhookを介してアプリケーションにさまざまなイベントを通知することができます。デフォルトでは、CashierのWebhookコントローラーを指すルートがCashierサービスプロバイダーによって自動的に登録されます。このコントローラーは、すべての受信Webhookリクエストを処理します。

デフォルトでは、CashierのWebhookコントローラーは、Stripeの設定で定義された失敗した請求が多すぎるサブスクリプションのキャンセル、顧客の更新、顧客の削除、サブスクリプションの更新、および支払い方法の変更を自動的に処理します。ただし、すぐにわかるように、このコントローラーを拡張して、お好みのStripe Webhookイベントを処理することができます。

アプリケーションがStripe Webhookを処理できるようにするには、StripeコントロールパネルでWebhook URLを設定してください。デフォルトでは、CashierのWebhookコントローラーは`/stripe/webhook` URLパスに応答します。Stripeコントロールパネルで有効にする必要があるすべてのWebhookの完全なリストは次のとおりです。

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

便宜上、Cashierには`cashier:webhook` Artisanコマンドが含まれています。このコマンドは、Cashierに必要なすべてのイベントをリッスンするStripeにWebhookを作成します。

```shell
php artisan cashier:webhook
```

デフォルトでは、作成されたWebhookは`APP_URL`環境変数とCashierに含まれる`cashier.webhook`ルートで定義されたURLを指します。コマンドを呼び出す際に別のURLを使用したい場合は、`--url`オプションを指定できます。

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

作成されたWebhookは、Cashierのバージョンと互換性のあるStripe APIバージョンを使用します。別のStripeバージョンを使用したい場合は、`--api-version`オプションを指定できます。

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

作成後、Webhookはすぐにアクティブになります。準備が整うまでWebhookを無効にしておきたい場合は、コマンドを呼び出す際に`--disabled`オプションを指定できます。

```shell
php artisan cashier:webhook --disabled
```

> WARNING:  
> Cashierに含まれる[Webhook署名検証](#verifying-webhook-signatures)ミドルウェアを使用して、受信Stripe Webhookリクエストを保護するようにしてください。

<a name="webhooks-csrf-protection"></a>
#### WebhooksとCSRF保護

Stripe WebhookはLaravelの[CSRF保護](csrf.md)をバイパスする必要があるため、アプリケーションの`bootstrap/app.php`ファイルで`stripe/*`をCSRF保護から除外するようにしてください。

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->validateCsrfTokens(except: [
            'stripe/*',
        ]);
    })

<a name="defining-webhook-event-handlers"></a>
### Webhookイベントハンドラの定義

Cashierは、失敗した請求に対するサブスクリプションのキャンセルやその他の一般的なStripe Webhookイベントを自動的に処理します。ただし、追加のWebhookイベントを処理したい場合は、Cashierによってディスパッチされる次のイベントをリッスンすることで行うことができます。

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

両方のイベントには、Stripe Webhookの完全なペイロードが含まれています。例えば、`invoice.payment_succeeded` Webhookを処理したい場合は、イベントを処理する[リスナー](events.md#defining-listeners)を登録することができます。

    <?php

    namespace App\Listeners;

    use Laravel\Cashier\Events\WebhookReceived;

    class StripeEventListener
    {
        /**
         * Handle received Stripe webhooks.
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['type'] === 'invoice.payment_succeeded') {
                // Handle the incoming event...
            }
        }
    }

<a name="verifying-webhook-signatures"></a>
### Webhook署名の検証

Webhookを保護するために、[StripeのWebhook署名](https://stripe.com/docs/webhooks/signatures)を使用することができます。便宜上、Cashierには受信Stripe Webhookリクエストが有効であることを検証するミドルウェアが自動的に含まれています。

Webhook検証を有効にするには、アプリケーションの`.env`ファイルに`STRIPE_WEBHOOK_SECRET`環境変数が設定されていることを確認してください。Webhook `secret`は、Stripeアカウントのダッシュボードから取得できます。

<a name="single-charges"></a>
## 一括請求

<a name="simple-charge"></a>
### 単一請求

顧客に対して一括請求を行いたい場合は、請求可能なモデルインスタンスで`charge`メソッドを使用できます。`charge`メソッドの第2引数として、[支払い方法識別子](#payment-methods-for-single-charges)を指定する必要があります。

    use Illuminate\Http\Request;

    Route::post('/purchase', function (Request $request) {
        $stripeCharge = $request->user()->charge(
            100, $request->paymentMethodId
        );

        // ...
    });

`charge`メソッドは、第3引数として配列を受け取り、基礎となるStripe請求作成に渡すオプションを指定できます。請求作成時に利用可能なオプションについては、[Stripeドキュメント](https://stripe.com/docs/api/charges/create)を参照してください。

    $user->charge(100, $paymentMethod, [
        'custom_option' => $value,
    ]);

また、顧客やユーザーを基にせずに`charge`メソッドを使用することもできます。そのためには、アプリケーションの請求可能モデルの新しいインスタンスで`charge`メソッドを呼び出します。

    use App\Models\User;

    $stripeCharge = (new User)->charge(100, $paymentMethod);

`charge`メソッドは、請求が失敗した場合に例外をスローします。請求が成功した場合、メソッドから`Laravel\Cashier\Payment`のインスタンスが返されます。

    try {
        $payment = $user->charge(100, $paymentMethod);
    } catch (Exception $e) {
        // ...
    }

> WARNING:  
> `charge`メソッドは、アプリケーションで使用される通貨の最小単位で支払い金額を受け取ります。例えば、顧客が米ドルで支払う場合、金額はペニーで指定する必要があります。

<a name="charge-with-invoice"></a>
### 請求書付き請求

時には、一括請求を行い、顧客にPDF請求書を提供する必要があるかもしれません。`invoicePrice`メソッドを使用すると、それが可能になります。例えば、顧客に5枚の新しいTシャツを請求してみましょう。

    $user->invoicePrice('price_tshirt', 5);

請求書は、顧客のデフォルト支払い方法に対して即座に請求されます。`invoicePrice`メソッドは、第3引数として配列を受け取ります。この配列には、請求書アイテムの請求オプションが含まれます。メソッドの第4引数も配列で、請求書自体の請求オプションが含まれます。

    $user->invoicePrice('price_tshirt', 5, [
        'discounts' => [
            ['coupon' => 'SUMMER21SALE']
        ],
    ], [
        'default_tax_rates' => ['txr_id'],
    ]);

`invoicePrice`と同様に、`tabPrice`メソッドを使用して、複数のアイテム（請求書ごとに最大250アイテム）に対して一括請求を行うことができます。これにより、顧客の「タブ」にアイテムを追加し、顧客に請求することができます。例えば、顧客に5枚のTシャツと2つのマグカップを請求してみましょう。

    $user->tabPrice('price_tshirt', 5);
    $user->tabPrice('price_mug', 2);
    $user->invoice();

また、`invoiceFor`メソッドを使用して、顧客のデフォルト支払い方法に対して「一括請求」を行うこともできます。

    $user->invoiceFor('One Time Fee', 500);

`invoiceFor`メソッドは使用可能ですが、事前に定義された価格を使用する`invoicePrice`および`tabPrice`メソッドを使用することをお勧めします。これにより、Stripeダッシュボード内で製品ごとの販売に関するより良い分析とデータにアクセスできます。

> WARNING:  
> `invoice`、`invoicePrice`、および`invoiceFor`メソッドは、失敗した請求を再試行するStripe請求書を作成します。請求が失敗した場合に請求書を再試行しないようにするには、最初の請求が失敗した後にStripe APIを使用して請求書を閉じる必要があります。

<a name="creating-payment-intents"></a>
### 支払いインテントの作成

請求可能なモデルインスタンスで`pay`メソッドを呼び出すことで、新しいStripe支払いインテントを作成できます。このメソッドを呼び出すと、`Laravel\Cashier\Payment`インスタンスにラップされた支払いインテントが作成されます。

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->pay(
            $request->get('amount')
        );

        return $payment->client_secret;
    });

支払いインテントを作成した後、クライアントシークレットをアプリケーションのフロントエンドに返すことで、ユーザーがブラウザで支払いを完了できるようになります。Stripeの支払いインテントを使用して完全な支払いフローを構築する方法について詳しく知りたい場合は、[Stripeのドキュメント](https://stripe.com/docs/payments/accept-a-payment?platform=web)を参照してください。

`pay`メソッドを使用する場合、Stripeダッシュボード内で有効になっているデフォルトの支払い方法が顧客に提供されます。代わりに、特定の支払い方法のみを許可したい場合は、`payWith`メソッドを使用できます：

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->payWith(
            $request->get('amount'), ['card', 'bancontact']
        );

        return $payment->client_secret;
    });

> WARNING:  
> `pay`と`payWith`メソッドは、アプリケーションで使用される通貨の最小単位で支払い金額を受け付けます。例えば、顧客が米ドルで支払う場合、金額はペニーで指定する必要があります。

<a name="refunding-charges"></a>
### 返金

Stripeの請求を返金する必要がある場合、`refund`メソッドを使用できます。このメソッドは、最初の引数としてStripeの[支払いインテントID](#payment-methods-for-single-charges)を受け取ります：

    $payment = $user->charge(100, $paymentMethodId);

    $user->refund($payment->id);

<a name="invoices"></a>
## 請求書

<a name="retrieving-invoices"></a>
### 請求書の取得

`invoices`メソッドを使用して、課金可能なモデルの請求書の配列を簡単に取得できます。`invoices`メソッドは、`Laravel\Cashier\Invoice`インスタンスのコレクションを返します：

    $invoices = $user->invoices();

結果に保留中の請求書を含めたい場合は、`invoicesIncludingPending`メソッドを使用できます：

    $invoices = $user->invoicesIncludingPending();

`findInvoice`メソッドを使用して、IDで特定の請求書を取得できます：

    $invoice = $user->findInvoice($invoiceId);

<a name="displaying-invoice-information"></a>
#### 請求書情報の表示

顧客の請求書をリストアップする際に、請求書のメソッドを使用して関連する請求書情報を表示できます。例えば、テーブルにすべての請求書をリストアップし、ユーザーが簡単にダウンロードできるようにすることができます：

    <table>
        @foreach ($invoices as $invoice)
            <tr>
                <td>{{ $invoice->date()->toFormattedDateString() }}</td>
                <td>{{ $invoice->total() }}</td>
                <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
            </tr>
        @endforeach
    </table>

<a name="upcoming-invoices"></a>
### 今後の請求書

顧客の今後の請求書を取得するには、`upcomingInvoice`メソッドを使用できます：

    $invoice = $user->upcomingInvoice();

同様に、顧客が複数のサブスクリプションを持っている場合、特定のサブスクリプションの今後の請求書を取得することもできます：

    $invoice = $user->subscription('default')->upcomingInvoice();

<a name="previewing-subscription-invoices"></a>
### サブスクリプション請求書のプレビュー

`previewInvoice`メソッドを使用して、価格変更を行う前に請求書をプレビューできます。これにより、特定の価格変更が行われた場合に顧客の請求書がどのように見えるかを確認できます：

    $invoice = $user->subscription('default')->previewInvoice('price_yearly');

複数の新しい価格で請求書をプレビューするために、`previewInvoice`メソッドに価格の配列を渡すこともできます：

    $invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

<a name="generating-invoice-pdfs"></a>
### 請求書PDFの生成

請求書PDFを生成する前に、Composerを使用してDompdfライブラリをインストールする必要があります。これは、Cashierのデフォルトの請求書レンダラーです：

```php
composer require dompdf/dompdf
```

ルートまたはコントローラ内から、`downloadInvoice`メソッドを使用して、指定された請求書のPDFダウンロードを生成できます。このメソッドは、請求書のダウンロードに必要な適切なHTTPレスポンスを自動的に生成します：

    use Illuminate\Http\Request;

    Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId);
    });

デフォルトでは、請求書上のすべてのデータは、Stripeに保存された顧客と請求書のデータから派生します。ファイル名は`app.name`設定値に基づいています。ただし、`downloadInvoice`メソッドに2番目の引数として配列を提供することで、このデータの一部をカスタマイズできます。この配列を使用して、会社や製品の詳細などの情報をカスタマイズできます：

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => 'Your Company',
        'product' => 'Your Product',
        'street' => 'Main Str. 1',
        'location' => '2000 Antwerp, Belgium',
        'phone' => '+32 499 00 00 00',
        'email' => 'info@example.com',
        'url' => 'https://example.com',
        'vendorVat' => 'BE123456789',
    ]);

`downloadInvoice`メソッドは、3番目の引数を介してカスタムファイル名も許可します。このファイル名には、自動的に`.pdf`が付加されます：

    return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

<a name="custom-invoice-render"></a>
#### カスタム請求書レンダラー

Cashierでは、カスタム請求書レンダラーを使用することも可能です。デフォルトでは、Cashierは`DompdfInvoiceRenderer`実装を使用し、[dompdf](https://github.com/dompdf/dompdf) PHPライブラリを使用してCashierの請求書を生成します。ただし、`Laravel\Cashier\Contracts\InvoiceRenderer`インターフェースを実装することで、任意のレンダラーを使用できます。例えば、サードパーティのPDFレンダリングサービスへのAPIコールを使用して請求書PDFをレンダリングすることができます：

    use Illuminate\Support\Facades\Http;
    use Laravel\Cashier\Contracts\InvoiceRenderer;
    use Laravel\Cashier\Invoice;

    class ApiInvoiceRenderer implements InvoiceRenderer
    {
        /**
         * 指定された請求書をレンダリングし、生のPDFバイトを返します。
         */
        public function render(Invoice $invoice, array $data = [], array $options = []): string
        {
            $html = $invoice->view($data)->render();

            return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
        }
    }

請求書レンダラー契約を実装したら、アプリケーションの`config/cashier.php`設定ファイル内の`cashier.invoices.renderer`設定値を更新する必要があります。この設定値は、カスタムレンダラー実装のクラス名に設定する必要があります。

<a name="checkout"></a>
## チェックアウト

Cashier Stripeは、[Stripe Checkout](https://stripe.com/payments/checkout)もサポートしています。Stripe Checkoutは、支払いを受け付けるためのカスタムページを実装する手間を省いてくれる、事前に構築されたホストされた支払いページを提供します。

以下のドキュメントには、CashierでStripe Checkoutを使い始める方法についての情報が含まれています。Stripe Checkoutについてより詳しく知りたい場合は、[StripeのCheckoutに関する独自のドキュメント](https://stripe.com/docs/payments/checkout)も確認することを検討してください。

<a name="product-checkouts"></a>
### 商品のチェックアウト

Stripeダッシュボード内で作成された既存の商品のチェックアウトを実行するには、課金可能なモデルの`checkout`メソッドを使用します。`checkout`メソッドは、新しいStripe Checkoutセッションを開始します。デフォルトでは、Stripe Price IDを渡す必要があります：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout('price_tshirt');
    });

必要に応じて、商品の数量を指定することもできます：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 15]);
    });

顧客がこのルートにアクセスすると、Stripeのチェックアウトページにリダイレクトされます。ユーザーが購入を正常に完了するかキャンセルすると、デフォルトで`home`ルートの場所にリダイレクトされますが、`success_url`と`cancel_url`オプションを使用してカスタムコールバックURLを指定できます：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
    });

`success_url`チェックアウトオプションを定義する際に、StripeにチェックアウトセッションIDをクエリ文字列パラメータとして追加するよう指示できます。そのためには、`success_url`クエリ文字列にリテラル文字列`{CHECKOUT_SESSION_ID}`を追加します。Stripeはこのプレースホルダを実際のチェックアウトセッションIDに置き換えます：

    use Illuminate\Http\Request;
    use Stripe\Checkout\Session;
    use Stripe\Customer;

    Route::get('/product-checkout', function (Request $request) {
        return $request->user()->checkout(['price_tshirt' => 1], [
            'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
            'cancel_url' => route('checkout-cancel'),
        ]);
    });

    Route::get('/checkout-success', function (Request $request) {
        $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

        return view('checkout.success', ['checkoutSession' => $checkoutSession]);
    })->name('checkout-success');

<a name="checkout-promotion-codes"></a>
#### プロモーションコード

デフォルトでは、Stripe Checkoutは[ユーザーが利用可能なプロモーションコード](https://stripe.com/docs/billing/subscriptions/discounts/codes)を許可しません。幸いなことに、チェックアウトページでこれらを有効にする簡単な方法があります。そのためには、`allowPromotionCodes`メソッドを呼び出すことができます：

    use Illuminate\Http\Request;

    Route::get('/product-checkout', function (Request $request) {
    return $request->user()
        ->allowPromotionCodes()
        ->checkout('price_tshirt');
});

<a name="single-charge-checkouts"></a>
### 単発課金のチェックアウト

また、Stripeダッシュボードで作成されていないアドホックな商品の単純な課金を行うこともできます。そのためには、請求可能なモデルの`checkoutCharge`メソッドを使用し、課金可能な金額、商品名、およびオプションの数量を渡します。顧客がこのルートにアクセスすると、Stripeのチェックアウトページにリダイレクトされます。

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
});
```

> WARNING:  
> `checkoutCharge`メソッドを使用する場合、Stripeは常に新しい商品と価格をStripeダッシュボードに作成します。したがって、Stripeダッシュボードで事前に商品を作成し、代わりに`checkout`メソッドを使用することをお勧めします。

<a name="subscription-checkouts"></a>
### サブスクリプションのチェックアウト

> WARNING:  
> Stripe Checkoutを使用してサブスクリプションを開始するには、Stripeダッシュボードで`customer.subscription.created`ウェブフックを有効にする必要があります。このウェブフックは、データベースにサブスクリプションレコードを作成し、関連するすべてのサブスクリプションアイテムを保存します。

Stripe Checkoutを使用してサブスクリプションを開始することもできます。Cashierのサブスクリプションビルダーメソッドでサブスクリプションを定義した後、`checkout`メソッドを呼び出すことができます。顧客がこのルートにアクセスすると、Stripeのチェックアウトページにリダイレクトされます。

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout();
});
```

商品のチェックアウトと同様に、成功とキャンセルのURLをカスタマイズできます。

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

もちろん、サブスクリプションのチェックアウトにプロモーションコードを有効にすることもできます。

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->allowPromotionCodes()
        ->checkout();
});
```

> WARNING:  
> 残念ながら、Stripe Checkoutはサブスクリプションを開始する際にすべてのサブスクリプションの課金オプションをサポートしていません。サブスクリプションビルダーで`anchorBillingCycleOn`メソッドを使用したり、日割り計算の動作を設定したり、支払いの動作を設定したりしても、Stripe Checkoutセッション中には何の効果もありません。利用可能なパラメータについては、[Stripe CheckoutセッションAPIドキュメント](https://stripe.com/docs/api/checkout/sessions/create)を参照してください。

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkoutとトライアル期間

もちろん、Stripe Checkoutを使用して完了するサブスクリプションを構築する際に、トライアル期間を定義できます。

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

ただし、トライアル期間は少なくとも48時間でなければなりません。これは、Stripe Checkoutがサポートする最小のトライアル時間です。

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### サブスクリプションとウェブフック

StripeとCashierはウェブフックを介してサブスクリプションのステータスを更新することを忘れないでください。したがって、顧客が支払い情報を入力した後にアプリケーションに戻ったときに、サブスクリプションがまだアクティブでない可能性があります。このシナリオを処理するために、支払いまたはサブスクリプションが保留中であることをユーザーに通知するメッセージを表示することができます。

<a name="collecting-tax-ids"></a>
### 税IDの収集

チェックアウトは、顧客の税IDの収集もサポートしています。セッションを作成する際に`collectTaxIds`メソッドを呼び出すことで、これをチェックアウトセッションで有効にできます。

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

このメソッドが呼び出されると、顧客が会社として購入しているかどうかを示す新しいチェックボックスが顧客に表示されます。もしそうなら、税ID番号を提供する機会があります。

> WARNING:  
> アプリケーションのサービスプロバイダで[自動税収集](#tax-configuration)を既に設定している場合、この機能は自動的に有効になり、`collectTaxIds`メソッドを呼び出す必要はありません。

<a name="guest-checkouts"></a>
### ゲストチェックアウト

`Checkout::guest`メソッドを使用すると、アプリケーションのゲスト（アカウントを持たないユーザー）のチェックアウトセッションを開始できます。

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()->create('price_tshirt', [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

既存のユーザーのチェックアウトセッションを作成する場合と同様に、`Laravel\Cashier\CheckoutBuilder`インスタンスで利用可能な追加のメソッドを使用して、ゲストチェックアウトセッションをカスタマイズできます。

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()
        ->withPromotionCode('promo-code')
        ->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

ゲストチェックアウトが完了した後、Stripeは`checkout.session.completed`ウェブフックイベントをディスパッチできます。そのため、このイベントをアプリケーションに実際に送信するように[Stripeウェブフックを設定](https://dashboard.stripe.com/webhooks)してください。Stripeダッシュボード内でウェブフックが有効になったら、[Cashierでウェブフックを処理](#handling-stripe-webhooks)できます。ウェブフックペイロードに含まれるオブジェクトは、顧客の注文を履行するために検査できる[`checkout`オブジェクト](https://stripe.com/docs/api/checkout/sessions/object)になります。

<a name="handling-failed-payments"></a>
## 失敗した支払いの処理

時には、サブスクリプションや単発の課金の支払いが失敗することがあります。この場合、Cashierは`Laravel\Cashier\Exceptions\IncompletePayment`例外をスローし、これが発生したことを通知します。この例外をキャッチした後、2つの選択肢があります。

まず、顧客をCashierに含まれる専用の支払い確認ページにリダイレクトすることができます。このページには、Cashierのサービスプロバイダを介して登録された名前付きルートが既にあります。したがって、`IncompletePayment`例外をキャッチし、ユーザーを支払い確認ページにリダイレクトできます。

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', 'price_monthly')
                            ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route(
        'cashier.payment',
        [$exception->payment->id, 'redirect' => route('home')]
    );
}
```

支払い確認ページで、顧客はクレジットカード情報を再度入力し、Stripeが要求する追加のアクション（例：「3D Secure」の確認）を実行するよう求められます。支払いを確認した後、ユーザーは上記で指定した`redirect`パラメータで提供されたURLにリダイレクトされます。リダイレクト時に、`message`（文字列）と`success`（整数）のクエリ文字列変数がURLに追加されます。支払いページは現在、以下の支払い方法タイプをサポートしています。

<div class="content-list" markdown="1">

- クレジットカード
- アリペイ
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

</div>

あるいは、Stripeに支払い確認を処理させることもできます。この場合、支払い確認ページにリダイレクトする代わりに、[Stripeの自動請求メール](https://dashboard.stripe.com/account/billing/automatic)をStripeダッシュボードで設定できます。ただし、`IncompletePayment`例外がキャッチされた場合、ユーザーにさらなる支払い確認手順を含むメールを受け取ることを通知する必要があります。

支払い例外は、`Billable`トレイトを使用するモデルの`charge`、`invoiceFor`、および`invoice`メソッドでスローされる可能性があります。サブスクリプションとのやり取りでは、`SubscriptionBuilder`の`create`メソッド、および`Subscription`と`SubscriptionItem`モデルの`incrementAndInvoice`と`swapAndInvoice`メソッドで不完全な支払い例外がスローされる可能性があります。

既存のサブスクリプションに不完全な支払いがあるかどうかを判断するには、請求可能モデルまたはサブスクリプションインスタンスの`hasIncompletePayment`メソッドを使用できます。

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

不完全な支払いの特定のステータスを取得するには、例外インスタンスの`payment`プロパティを検査できます。

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $user->charge(1000, 'pm_card_threeDSecure2Required');
} catch (IncompletePayment $exception) {
    // 支払いインテントのステータスを取得...
    $exception->payment->status;

    // 特定の条件をチェック...
    if ($exception->payment->requiresPaymentMethod()) {
        // ...
    } elseif ($exception->payment->requiresConfirmation()) {
        // ...
    }
}
```

<a name="confirming-payments"></a>
### 支払いの確認

一部の支払い方法では、支払いを確認するために追加のデータが必要です。例えば、SEPA支払い方法では、支払い処理中に追加の「mandate」データが必要です。このデータは、`withPaymentConfirmationOptions`メソッドを使用してCashierに提供できます。

    $subscription->withPaymentConfirmationOptions([
        'mandate_data' => '...',
    ])->swap('price_xxx');

支払いを確認する際に受け入れられるすべてのオプションを確認するには、[Stripe APIドキュメント](https://stripe.com/docs/api/payment_intents/confirm)を参照してください。

<a name="strong-customer-authentication"></a>
## 強力な顧客認証

あなたの事業や顧客のいずれかがヨーロッパに拠点を置いている場合、EUの強力な顧客認証（SCA）規制に従う必要があります。これらの規制は、2019年9月に欧州連合によって支払い詐欺を防ぐために課されました。幸いなことに、StripeとCashierはSCAに準拠したアプリケーションの構築に対応しています。

> WARNING:  
> 始める前に、[StripeのPSD2とSCAに関するガイド](https://stripe.com/guides/strong-customer-authentication)と、[新しいSCA APIに関するドキュメント](https://stripe.com/docs/strong-customer-authentication)を確認してください。

<a name="payments-requiring-additional-confirmation"></a>
### 追加の確認を必要とする支払い

SCA規制では、支払いを確認して処理するために追加の検証が必要になることがあります。この場合、Cashierは追加の検証が必要であることを通知する`Laravel\Cashier\Exceptions\IncompletePayment`例外をスローします。これらの例外の処理方法についての詳細は、[失敗した支払いの処理](#handling-failed-payments)に関するドキュメントで確認できます。

StripeまたはCashierによって提示される支払い確認画面は、特定の銀行やカード発行者の支払いフローに合わせて調整されることがあり、追加のカード確認、一時的な小額の請求、別のデバイスでの認証、またはその他の形式の検証が含まれることがあります。

<a name="incomplete-and-past-due-state"></a>
#### 不完全および遅延状態

支払いに追加の確認が必要な場合、サブスクリプションは`incomplete`または`past_due`状態のままとなり、その状態は`stripe_status`データベースカラムで示されます。Cashierは、支払い確認が完了し、アプリケーションがStripeからのWebhookによって完了を通知されると、顧客のサブスクリプションを自動的に有効化します。

`incomplete`と`past_due`状態についての詳細は、[これらの状態に関する追加のドキュメント](#incomplete-and-past-due-status)を参照してください。

<a name="off-session-payment-notifications"></a>
### セッション外支払い通知

SCA規制では、顧客がサブスクリプションがアクティブな間でも支払い詳細を定期的に確認する必要があります。たとえば、サブスクリプションの更新時に発生することがあります。Cashierは、セッション外支払い確認が必要な場合に顧客に通知を送信できます。Cashierの支払い通知は、`CASHIER_PAYMENT_NOTIFICATION`環境変数を通知クラスに設定することで有効にできます。デフォルトでは、この通知は無効になっています。もちろん、Cashierにはこの目的で使用できる通知クラスが含まれていますが、必要に応じて独自の通知クラスを提供することも自由です。

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

セッション外支払い確認通知が配信されるようにするには、[Stripe Webhookがアプリケーションに対して設定されている](#handling-stripe-webhooks)こと、およびStripeダッシュボードで`invoice.payment_action_required` Webhookが有効になっていることを確認してください。さらに、`Billable`モデルはLaravelの`Illuminate\Notifications\Notifiable`トレイトを使用する必要があります。

> WARNING:  
> 顧客が手動で追加の確認を必要とする支払いを行っている場合でも、通知は送信されます。残念ながら、Stripeは支払いが手動で行われたか「セッション外」で行われたかを知る方法がありません。しかし、顧客が既に支払いを確認した後に支払いページにアクセスすると、「支払い成功」のメッセージが表示されます。顧客は同じ支払いを誤って2回確認し、誤って2回目の請求を受けることはできません。

<a name="stripe-sdk"></a>
## Stripe SDK

Cashierの多くのオブジェクトは、Stripe SDKオブジェクトのラッパーです。Stripeオブジェクトと直接やり取りしたい場合は、`asStripe`メソッドを使用して便利に取得できます。

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

Stripeサブスクリプションを直接更新するには、`updateStripeSubscription`メソッドを使用することもできます。

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

`Stripe\StripeClient`クライアントを直接使用したい場合は、`Cashier`クラスの`stripe`メソッドを呼び出すことができます。たとえば、このメソッドを使用して`StripeClient`インスタンスにアクセスし、Stripeアカウントから価格のリストを取得できます。

    use Laravel\Cashier\Cashier;

    $prices = Cashier::stripe()->prices->all();

<a name="testing"></a>
## テスト

Cashierを使用するアプリケーションをテストする際、Stripe APIへの実際のHTTPリクエストをモックすることができます。ただし、これにはCashier自身の動作を部分的に再実装する必要があります。したがって、テストが実際のStripe APIにアクセスすることを推奨します。これは遅くなりますが、アプリケーションが期待通りに動作していることをより確信できます。遅いテストは、Pest / PHPUnitの独自のテストグループ内に配置することができます。

テスト時には、Cashier自体にはすでに優れたテストスイートがあるため、アプリケーションのサブスクリプションと支払いフローのテストに集中し、Cashierのすべての基礎的な動作をテストする必要はありません。

始めるには、Stripeシークレットの**テスト**バージョンを`phpunit.xml`ファイルに追加します。

    <env name="STRIPE_SECRET" value="sk_test_<your-key>"/>

これで、テスト中にCashierとやり取りするたびに、実際のAPIリクエストがStripeのテスト環境に送信されます。便宜上、テスト中に使用できるサブスクリプションや価格をStripeのテストアカウントに事前に入力しておく必要があります。

> NOTE:  
> クレジットカードの拒否や失敗など、さまざまな請求シナリオをテストするために、Stripeが提供する幅広い[テスト用カード番号とトークン](https://stripe.com/docs/testing)を使用できます。

