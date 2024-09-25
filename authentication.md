# 認証

- [はじめに](#introduction)
    - [スターターキット](#starter-kits)
    - [データベースの考慮事項](#introduction-database-considerations)
    - [エコシステムの概要](#ecosystem-overview)
- [認証のクイックスタート](#authentication-quickstart)
    - [スターターキットのインストール](#install-a-starter-kit)
    - [認証済みユーザーの取得](#retrieving-the-authenticated-user)
    - [ルートの保護](#protecting-routes)
    - [ログインのスロットリング](#login-throttling)
- [ユーザーの手動認証](#authenticating-users)
    - [ユーザーの記憶](#remembering-users)
    - [その他の認証方法](#other-authentication-methods)
- [HTTP基本認証](#http-basic-authentication)
    - [ステートレスなHTTP基本認証](#stateless-http-basic-authentication)
- [ログアウト](#logging-out)
    - [他のデバイス上のセッションの無効化](#invalidating-sessions-on-other-devices)
- [パスワード確認機能](#password-confirmation)
    - [設定](#password-confirmation-configuration)
    - [ルーティング](#password-confirmation-routing)
    - [ルートの保護](#password-confirmation-protecting-routes)
- [カスタムガードの追加](#adding-custom-guards)
    - [クロージャリクエストガード](#closure-request-guards)
- [カスタムユーザープロバイダの追加](#adding-custom-user-providers)
    - [ユーザープロバイダ契約](#the-user-provider-contract)
    - [Authenticatable契約](#the-authenticatable-contract)
- [自動パスワード再ハッシュ](#automatic-password-rehashing)
- [ソーシャル認証](socialite.md)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

多くのWebアプリケーションは、ユーザーがアプリケーションに認証し、「ログイン」する方法を提供しています。Webアプリケーションでこの機能を実装することは、複雑で潜在的にリスクのある作業になる可能性があります。そのため、Laravelは、認証を迅速かつ安全かつ簡単に実装するために必要なツールを提供することを目指しています。

Laravelの認証機能の中核は、「ガード」と「プロバイダ」で構成されています。ガードは、各リクエストに対してユーザーがどのように認証されるかを定義します。例えば、Laravelにはセッションストレージとクッキーを使用して状態を維持する`session`ガードが付属しています。

プロバイダは、ユーザーが永続的なストレージからどのように取得されるかを定義します。Laravelには、[Eloquent](eloquent.md)とデータベースクエリビルダを使用してユーザーを取得するためのサポートが付属しています。ただし、アプリケーションに必要に応じて追加のプロバイダを自由に定義できます。

アプリケーションの認証設定ファイルは、`config/auth.php`にあります。このファイルには、Laravelの認証サービスの動作を微調整するためのいくつかのよく文書化されたオプションが含まれています。

> NOTE:  
> ガードとプロバイダは、「ロール」と「権限」と混同しないでください。権限を介してユーザーアクションを承認する方法の詳細については、[承認](authorization.md)のドキュメントを参照してください。

<a name="starter-kits"></a>
### スターターキット

すぐに始めたいですか？新しいLaravelアプリケーションに[Laravelアプリケーションスターターキットについて](starter-kits.md)をインストールしてください。データベースをマイグレーションした後、ブラウザで`/register`またはアプリケーションに割り当てられた他のURLに移動してください。スターターキットは、認証システム全体のスキャフォールディングを行います！

**最終的なLaravelアプリケーションでスターターキットを使用しない場合でも、[Laravel Breeze](starter-kits.md#laravel-breeze)スターターキットをインストールすることは、Laravelの認証機能を実際のLaravelプロジェクトでどのように実装するかを学ぶための素晴らしい機会です。** Laravel Breezeは認証コントローラ、ルート、ビューを作成するため、これらのファイル内のコードを調べて、Laravelの認証機能がどのように実装されるかを学ぶことができます。

<a name="introduction-database-considerations"></a>
### データベースの考慮事項

デフォルトで、Laravelには`app/Models`ディレクトリに`App\Models\User` [Eloquentモデル](eloquent.md)が含まれています。このモデルは、デフォルトのEloquent認証ドライバと共に使用できます。アプリケーションがEloquentを使用していない場合は、Laravelクエリビルダを使用する`database`認証プロバイダを使用できます。

`App\Models\User`モデルのデータベーススキーマを構築する際に、パスワード列の長さが少なくとも60文字であることを確認してください。もちろん、新しいLaravelアプリケーションに含まれる`users`テーブルのマイグレーションは、この長さを超える列を既に作成しています。

また、`users`（または同等の）テーブルに、null許容の100文字の`remember_token`列が含まれていることを確認してください。この列は、アプリケーションにログインする際に「ログイン状態を維持」オプションを選択したユーザーのトークンを保存するために使用されます。繰り返しになりますが、新しいLaravelアプリケーションに含まれるデフォルトの`users`テーブルのマイグレーションには、この列が既に含まれています。

<a name="ecosystem-overview"></a>
### エコシステムの概要

Laravelは、認証に関連するいくつかのパッケージを提供しています。続行する前に、Laravelの認証エコシステムの概要と各パッケージの目的について説明します。

まず、認証がどのように機能するかを考えてみましょう。Webブラウザを使用する場合、ユーザーはログインフォームを介してユーザー名とパスワードを提供します。これらの資格情報が正しい場合、アプリケーションは認証済みユーザーに関する情報をユーザーの[セッション](session.md)に保存します。ブラウザに発行されるクッキーにはセッションIDが含まれており、後続のアプリケーションへのリクエストでユーザーを正しいセッションに関連付けることができます。セッションクッキーを受け取った後、アプリケーションはセッションIDに基づいてセッションデータを取得し、認証情報がセッションに保存されていることを確認し、ユーザーを「認証済み」と見なします。

リモートサービスがAPIにアクセスするために認証する必要がある場合、通常、認証にクッキーは使用されません。代わりに、リモートサービスは各リクエストでAPIトークンをAPIに送信します。アプリケーションは、有効なAPIトークンのテーブルに対して受信トークンを検証し、そのAPIトークンに関連付けられたユーザーによってリクエストが実行されたと「認証」します。

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravelの組み込みブラウザ認証サービスの概要

Laravelには、`Auth`と`Session`ファサードを介してアクセスできる組み込みの認証およびセッションサービスが含まれています。これらの機能は、Webブラウザから開始されるリクエストに対してクッキーベースの認証を提供します。これらは、ユーザーの資格情報を検証し、ユーザーを認証するためのメソッドを提供します。さらに、これらのサービスはユーザーのセッションに適切な認証データを自動的に保存し、ユーザーのセッションクッキーを発行します。これらのサービスの使用方法についての説明は、このドキュメントに含まれています。

**アプリケーションスターターキットについて**

このドキュメントで説明されているように、これらの認証サービスを手動で操作して、アプリケーションの独自の認証レイヤーを構築することができます。ただし、より迅速に開始するために、[無料のパッケージ](starter-kits.md)をリリースしています。これらのパッケージは、認証レイヤー全体の堅牢な、現代的なスキャフォールディングを提供します。これらのパッケージは、[Laravel Breeze](starter-kits.md#laravel-breeze)、[Laravel Jetstream](starter-kits.md#laravel-jetstream)、および[Laravel Fortify](fortify.md)です。

_Laravel Breeze_は、ログイン、登録、パスワードリセット、メール確認、パスワード確認など、Laravelのすべての認証機能のシンプルで最小限の実装です。Laravel Breezeのビューレイヤーは、[Bladeテンプレート](blade.md)で構成され、[Tailwind CSS](https://tailwindcss.com)でスタイリングされています。開始するには、Laravelの[アプリケーションスターターキットについて](starter-kits.md)のドキュメントを確認してください。

_Laravel Fortify_は、Laravelのヘッドレス認証バックエンドで、クッキーベースの認証や2要素認証、メール確認など、このドキュメントに記載されている多くの機能を実装しています。FortifyはLaravel Jetstreamの認証バックエンドを提供するか、[Laravel Sanctum](sanctum.md)と組み合わせて使用することで、Laravelで認証する必要があるSPAの認証バックエンドを提供できます。

_[Laravel Jetstream](https://jetstream.laravel.com)_は、[Tailwind CSS](https://tailwindcss.com)、[Livewire](https://livewire.laravel.com)、および/または[Inertia](https://inertiajs.com)によって強化された美しい、現代的なUIを備えた堅牢なアプリケーションスターターキットについてで、Laravel Fortifyの認証サービスを消費および公開します。Laravel Jetstreamには、2要素認証、チームサポート、ブラウザセッション管理、プロフィール管理、および[Laravel Sanctum](sanctum.md)との組み込み統合を含むAPIトークン認証のオプションサポートが含まれています。LaravelのAPI認証オファリングについては、以下で説明します。

<a name="laravels-api-authentication-services"></a>
#### LaravelのAPI認証サービスの概要

Laravelは、APIトークンの管理とAPIトークンを使用したリクエストの認証を支援するための2つのオプションパッケージを提供しています：[Passport](passport.md)と[Sanctum](sanctum.md)です。これらのライブラリとLaravelの組み込みのクッキーベースの認証ライブラリは相互に排他的ではないことに注意してください。これらのライブラリは主にAPIトークン認証に焦点を当てており、組み込みの認証サービスはクッキーベースのブラウザ認証に焦点を当てています。多くのアプリケーションは、Laravelの組み込みのクッキーベースの認証サービスとLaravelのAPI認証パッケージの両方を使用します。

**Passport**

PassportはOAuth2認証プロバイダであり、さまざまな種類のトークンを発行できるようにするさまざまなOAuth2の「付与タイプ」を提供します。一般的に、これはAPI認証のための堅牢で複雑なパッケージです。しかし、ほとんどのアプリケーションはOAuth2仕様が提供する複雑な機能を必要とせず、ユーザーや開発者の両方にとって混乱を招く可能性があります。さらに、開発者はこれまで、PassportのようなOAuth2認証プロバイダを使用してSPAアプリケーションやモバイルアプリケーションを認証する方法について混乱してきました。

**Sanctum**

OAuth2の複雑さと開発者の混乱に対応して、よりシンプルで合理化された認証パッケージを構築し、ウェブブラウザからのファーストパーティのウェブリクエストとトークンを介したAPIリクエストの両方を処理できるようにすることを目指しました。この目標は、[Laravel Sanctum](sanctum.md)のリリースによって実現されました。これは、ファーストパーティのウェブUIとAPIを提供するアプリケーション、またはバックエンドのLaravelアプリケーションとは別に存在するシングルページアプリケーション（SPA）によって駆動されるアプリケーション、またはモバイルクライアントを提供するアプリケーションにとって、推奨される認証パッケージと考えるべきです。

Laravel Sanctumは、ウェブ/API認証のハイブリッドパッケージであり、アプリケーションの認証プロセス全体を管理できます。これが可能なのは、Sanctumベースのアプリケーションがリクエストを受け取ると、Sanctumはまず、認証されたセッションを参照するセッションクッキーがリクエストに含まれているかどうかを判断するためです。Sanctumは、以前に説明したLaravelの組み込み認証サービスを呼び出すことでこれを実現します。セッションクッキーを介してリクエストが認証されていない場合、SanctumはリクエストにAPIトークンが含まれているかどうかを検査します。APIトークンが存在する場合、Sanctumはそのトークンを使用してリクエストを認証します。このプロセスの詳細については、Sanctumの["how it works"](sanctum.md#how-it-works)ドキュメントを参照してください。

Laravel Sanctumは、[Laravel Jetstream](https://jetstream.laravel.com)アプリケーションスターターキットについてに含めるAPIパッケージとして選択されました。これは、ほとんどのウェブアプリケーションの認証ニーズに最適であると考えているためです。

<a name="summary-choosing-your-stack"></a>
#### 認証スタックの選択ガイド

まとめると、アプリケーションがブラウザを介してアクセスされ、モノリシックなLaravelアプリケーションを構築している場合、アプリケーションはLaravelの組み込み認証サービスを使用します。

次に、アプリケーションがサードパーティによって消費されるAPIを提供する場合、アプリケーションのAPIトークン認証を提供するために[Passport](passport.md)または[Sanctum](sanctum.md)のいずれかを選択します。一般的に、可能な限りSanctumを優先するべきです。これは、API認証、SPA認証、モバイル認証のためのシンプルで完全なソリューションであり、「スコープ」や「能力」のサポートも含まれています。

Laravelバックエンドによって駆動されるシングルページアプリケーション（SPA）を構築している場合、[Laravel Sanctum](sanctum.md)を使用するべきです。Sanctumを使用する場合、[バックエンド認証ルートを手動で実装する](#authenticating-users)か、[Laravel Fortify](fortify.md)を使用して、登録、パスワードリセット、メール確認などの機能のためのルートとコントローラを提供するヘッドレス認証バックエンドサービスとして利用する必要があります。

アプリケーションがOAuth2仕様が提供するすべての機能を絶対に必要とする場合、Passportを選択することができます。

そして、迅速に開始したい場合、[Laravel Breeze](starter-kits.md#laravel-breeze)をお勧めします。これは、Laravelの組み込み認証サービスとLaravel Sanctumを使用した新しいLaravelアプリケーションをすぐに開始するための簡単な方法です。

<a name="authentication-quickstart"></a>
## 認証のクイックスタート

> WARNING:  
> この部分のドキュメントでは、[Laravelアプリケーションスターターキットについて](starter-kits.md)を介してユーザーを認証する方法について説明します。これには、迅速に開始できるようにUIのスキャフォールディングが含まれています。Laravelの認証システムと直接統合したい場合は、[ユーザーの手動認証](#authenticating-users)に関するドキュメントを確認してください。

<a name="install-a-starter-kit"></a>
### スターターキットのインストール

まず、[Laravelアプリケーションスターターキットについてをインストール](starter-kits.md)する必要があります。現在のスターターキットであるLaravel BreezeとLaravel Jetstreamは、新しいLaravelアプリケーションに認証を組み込むための美しくデザインされた出発点を提供します。

Laravel Breezeは、ログイン、登録、パスワードリセット、メール確認、パスワード確認を含む、Laravelのすべての認証機能の最小限のシンプルな実装です。Laravel Breezeのビューレイヤーは、[Tailwind CSS](https://tailwindcss.com)でスタイルされたシンプルな[Bladeテンプレート](blade.md)で構成されています。さらに、Breezeは[Livewire](https://livewire.laravel.com)または[Inertia](https://inertiajs.com)に基づくスキャフォールディングオプションを提供し、InertiaベースのスキャフォールディングにはVueまたはReactを使用する選択肢があります。

[Laravel Jetstream](https://jetstream.laravel.com)は、より堅牢なアプリケーションスターターキットについてであり、[Livewire](https://livewire.laravel.com)または[InertiaとVue](https://inertiajs.com)を使用してアプリケーションのスキャフォールディングをサポートします。さらに、Jetstreamは、二要素認証、チーム、プロフィール管理、ブラウザセッション管理、[Laravel Sanctum](sanctum.md)を介したAPIサポート、アカウント削除などのオプション機能を備えています。

<a name="retrieving-the-authenticated-user"></a>
### 認証済みユーザーの取得

認証スターターキットをインストールし、ユーザーがアプリケーションに登録して認証できるようにした後、現在認証されているユーザーとやり取りする必要があることがよくあります。受信リクエストを処理する際、`Auth`ファサードの`user`メソッドを介して認証済みユーザーにアクセスできます:

    use Illuminate\Support\Facades\Auth;

    // 現在認証されているユーザーを取得...
    $user = Auth::user();

    // 現在認証されているユーザーのIDを取得...
    $id = Auth::id();

または、ユーザーが認証されると、`Illuminate\Http\Request`インスタンスを介して認証済みユーザーにアクセスできます。型付きのクラスは自動的にコントローラメソッドに注入されることを覚えておいてください。`Illuminate\Http\Request`オブジェクトを型付けすることで、リクエストの`user`メソッドを介してアプリケーション内の任意のコントローラメソッドから認証済みユーザーに便利にアクセスできます:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class FlightController extends Controller
    {
        /**
         * 既存のフライトのフライト情報を更新します。
         */
        public function update(Request $request): RedirectResponse
        {
            $user = $request->user();

            // ...

            return redirect('/flights');
        }
    }

<a name="determining-if-the-current-user-is-authenticated"></a>
#### 現在のユーザーが認証されているかどうかの確認

受信HTTPリクエストを行っているユーザーが認証されているかどうかを判断するには、`Auth`ファサードの`check`メソッドを使用できます。このメソッドは、ユーザーが認証されている場合に`true`を返します:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // ユーザーはログインしています...
    }

> NOTE:  
> `check`メソッドを使用してユーザーが認証されているかどうかを判断することは可能ですが、通常はユーザーが特定のルート/コントローラにアクセスする前に認証されていることを確認するためにミドルウェアを使用します。これについて詳しくは、[ルートの保護](authentication.md#protecting-routes)に関するドキュメントを確認してください。

<a name="protecting-routes"></a>
### ルートの保護

[ルートミドルウェア](middleware.md)を使用して、認証済みユーザーのみが特定のルートにアクセスできるようにすることができます。Laravelには`auth`ミドルウェアが付属しており、これは`Illuminate\Auth\Middleware\Authenticate`クラスの[ミドルウェアエイリアス](middleware.md#middleware-aliases)です。このミドルウェアはLaravelによって内部的にエイリアスされているため、ミドルウェアをルート定義にアタッチするだけで済みます:

    Route::get('/flights', function () {
        // 認証済みユーザーのみがこのルートにアクセスできます...
    })->middleware('auth');

<a name="redirecting-unauthenticated-users"></a>
#### 認証されていないユーザーのリダイレクト

`auth`ミドルウェアが認証されていないユーザーを検出すると、ユーザーを`login`[名前付きルート](routing.md#named-routes)にリダイレクトします。この動作は、アプリケーションの`bootstrap/app.php`ファイルの`redirectGuestsTo`メソッドを使用して変更できます:

    use Illuminate\Http\Request;

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->redirectGuestsTo('/login');

        // クロージャを使用する場合...
        $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
    })

<a name="specifying-a-guard"></a>
#### ガードの指定

`auth`ミドルウェアをルートにアタッチする際、ユーザーを認証するために使用する「ガード」を指定することもできます。指定されたガードは、`auth.php`設定ファイルの`guards`配列のキーのいずれかに対応する必要があります:

    Route::get('/flights', function () {
        // 認証済みユーザーのみがこのルートにアクセスできます...
    })->middleware('auth:admin');

<a name="login-throttling"></a>
### ログインのスロットリング

Laravel BreezeやLaravel Jetstreamの[スターターキット](starter-kits.md)を使用している場合、ログイン試行に対してレート制限が自動的に適用されます。デフォルトでは、ユーザーが何度か正しい認証情報を提供できなかった場合、1分間ログインできなくなります。スロットリングはユーザーのユーザー名/メールアドレスとIPアドレスに固有です。

> NOTE:  
> アプリケーション内の他のルートにレート制限を適用したい場合は、[レート制限のドキュメント](routing.md#rate-limiting)を確認してください。

<a name="authenticating-users"></a>
## ユーザーの手動認証

Laravelの[アプリケーションスターターキットについて](starter-kits.md)に含まれる認証スカフォールディングを使用する必要はありません。このスカフォールディングを使用しないことを選択した場合、Laravelの認証クラスを直接使用してユーザー認証を管理する必要があります。心配はいりません、簡単です！

Laravelの認証サービスには、`Auth` [ファサード](facades.md)を介してアクセスしますので、クラスの先頭で`Auth`ファサードをインポートする必要があります。次に、`attempt`メソッドを見てみましょう。`attempt`メソッドは通常、アプリケーションの「ログイン」フォームからの認証試行を処理するために使用されます。認証が成功した場合、[セッション](session.md)を再生成して[セッション固定攻撃](https://en.wikipedia.org/wiki/Session_fixation)を防ぐ必要があります：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * 認証試行を処理します。
         */
        public function authenticate(Request $request): RedirectResponse
        {
            $credentials = $request->validate([
                'email' => ['required', 'email'],
                'password' => ['required'],
            ]);

            if (Auth::attempt($credentials)) {
                $request->session()->regenerate();

                return redirect()->intended('dashboard');
            }

            return back()->withErrors([
                'email' => '提供された認証情報が記録と一致しません。',
            ])->onlyInput('email');
        }
    }

`attempt`メソッドは、キー/値ペアの配列を最初の引数として受け取ります。配列内の値は、データベーステーブル内のユーザーを見つけるために使用されます。したがって、上記の例では、`email`列の値でユーザーが取得されます。ユーザーが見つかった場合、データベースに保存されたハッシュ化されたパスワードは、配列を介してメソッドに渡された`password`値と比較されます。受信リクエストの`password`値をハッシュ化する必要はありません。フレームワークは、データベース内のハッシュ化されたパスワードと比較する前に、値を自動的にハッシュ化します。2つのハッシュ化されたパスワードが一致する場合、ユーザーの認証済みセッションが開始されます。

Laravelの認証サービスは、認証ガードの「プロバイダ」設定に基づいてデータベースからユーザーを取得します。デフォルトの`config/auth.php`設定ファイルでは、Eloquentユーザープロバイダが指定されており、ユーザーを取得する際に`App\Models\User`モデルを使用するように指示されています。アプリケーションのニーズに基づいて、設定ファイル内でこれらの値を変更できます。

`attempt`メソッドは、認証が成功した場合は`true`を、失敗した場合は`false`を返します。

Laravelのリダイレクタが提供する`intended`メソッドは、認証ミドルウェアによってインターセプトされる前にユーザーがアクセスしようとしていたURLにユーザーをリダイレクトします。意図された宛先が利用できない場合、このメソッドにフォールバックURIを指定できます。

<a name="specifying-additional-conditions"></a>
#### 追加の条件の指定

必要に応じて、ユーザーのメールとパスワードに加えて、認証クエリに追加のクエリ条件を追加することもできます。これを行うには、`attempt`メソッドに渡される配列にクエリ条件を単純に追加します。たとえば、ユーザーが「アクティブ」とマークされていることを確認できます：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 認証が成功しました...
    }

複雑なクエリ条件の場合、認証情報の配列にクロージャを提供できます。このクロージャはクエリインスタンスで呼び出され、アプリケーションのニーズに基づいてクエリをカスタマイズできます：

    use Illuminate\Database\Eloquent\Builder;

    if (Auth::attempt([
        'email' => $email,
        'password' => $password,
        fn (Builder $query) => $query->has('activeSubscription'),
    ])) {
        // 認証が成功しました...
    }

> WARNING:  
> これらの例では、`email`は必須オプションではありません。単に例として使用されています。データベーステーブルの「ユーザー名」に対応する列名を使用する必要があります。

`attemptWhen`メソッドは、2番目の引数としてクロージャを受け取り、実際にユーザーを認証する前に潜在的なユーザーをより徹底的に検査するために使用できます。クロージャは潜在的なユーザーを受け取り、ユーザーが認証されるかどうかを示すために`true`または`false`を返す必要があります：

    if (Auth::attemptWhen([
        'email' => $email,
        'password' => $password,
    ], function (User $user) {
        return $user->isNotBanned();
    })) {
        // 認証が成功しました...
    }

<a name="accessing-specific-guard-instances"></a>
#### 特定のガードインスタンスへのアクセス

`Auth`ファサードの`guard`メソッドを介して、ユーザーを認証する際に使用するガードインスタンスを指定できます。これにより、完全に別々の認証可能なモデルまたはユーザーテーブルを使用して、アプリケーションの別々の部分の認証を管理できます。

`guard`メソッドに渡されるガード名は、`auth.php`設定ファイルで設定されたガードのいずれかに対応する必要があります：

    if (Auth::guard('admin')->attempt($credentials)) {
        // ...
    }

<a name="remembering-users"></a>
### ユーザーの記憶

多くのWebアプリケーションは、ログインフォームに「記憶する」チェックボックスを提供しています。アプリケーションで「記憶する」機能を提供したい場合、`attempt`メソッドにブール値を2番目の引数として渡すことができます。

この値が`true`の場合、Laravelはユーザーを無期限に認証したままにするか、手動でログアウトするまで認証したままにします。`users`テーブルには、「記憶する」トークンを保存するために使用される文字列の`remember_token`列が含まれている必要があります。新しいLaravelアプリケーションに含まれる`users`テーブルマイグレーションには、すでにこの列が含まれています：

    use Illuminate\Support\Facades\Auth;

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // ユーザーは記憶されています...
    }

アプリケーションが「ログイン状態を維持する」機能を提供している場合、`viaRemember`メソッドを使用して、現在認証されているユーザーが「記憶する」クッキーを使用して認証されたかどうかを判断できます：

    use Illuminate\Support\Facades\Auth;

    if (Auth::viaRemember()) {
        // ...
    }

<a name="other-authentication-methods"></a>
### その他の認証方法

<a name="authenticate-a-user-instance"></a>
#### ユーザーインスタンスの認証

既存の既存のユーザーインスタンスを現在認証されているユーザーとして設定したい場合、ユーザーインスタンスを`Auth`ファサードの`login`メソッドに渡すことができます。指定されたユーザーインスタンスは、`Illuminate\Contracts\Auth\Authenticatable` [契約](contracts.md)の実装である必要があります。Laravelに含まれる`App\Models\User`モデルは、すでにこのインターフェースを実装しています。この認証方法は、ユーザーがアプリケーションに登録した直後など、既に有効なユーザーインスタンスが存在する場合に便利です：

    use Illuminate\Support\Facades\Auth;

    Auth::login($user);

`login`メソッドにブール値を2番目の引数として渡すことができます。この値は、認証されたセッションに「記憶する」機能が必要かどうかを示します。これは、セッションが無期限に認証されたままになるか、ユーザーが手動でアプリケーションからログアウトするまで認証されたままになることを意味します：

    Auth::login($user, $remember = true);

必要に応じて、`login`メソッドを呼び出す前に認証ガードを指定できます：

    Auth::guard('admin')->login($user);

<a name="authenticate-a-user-by-id"></a>
#### IDによるユーザーの認証

データベースレコードの主キーを使用してユーザーを認証するには、`loginUsingId`メソッドを使用できます。このメソッドは、認証したいユーザーの主キーを受け取ります：

    Auth::loginUsingId(1);

`loginUsingId`メソッドの`remember`引数にブール値を渡すことができます。この値は、認証されたセッションに「記憶する」機能が必要かどうかを示します。これは、セッションが無期限に認証されたままになるか、ユーザーが手動でアプリケーションからログアウトするまで認証されたままになることを意味します：

    Auth::loginUsingId(1, remember: true);

<a name="authenticate-a-user-once"></a>
#### ユーザーの一時的な認証

`once`メソッドを使用して、ユーザーをアプリケーションに対して単一のリクエストで認証することができます。このメソッドを呼び出す際にセッションやクッキーは使用されません：

    if (Auth::once($credentials)) {
        // ...
    }

<a name="http-basic-authentication"></a>
## HTTP基本認証

[HTTP基本認証](https://en.wikipedia.org/wiki/Basic_access_authentication)は、専用の「ログイン」ページを設定せずにアプリケーションのユーザーを認証するための迅速な方法を提供します。これを開始するには、`auth.basic` [ミドルウェア](middleware.md)をルートにアタッチします。`auth.basic`ミドルウェアはLaravelフレームワークに含まれているため、定義する必要はありません：

    Route::get('/profile', function () {
        // 認証されたユーザーのみがこのルートにアクセスできます...
    })->middleware('auth.basic');

ミドルウェアがこのミドルウェアがルートにアタッチされると、ブラウザでそのルートにアクセスする際に自動的に認証情報の入力が求められます。デフォルトでは、`auth.basic`ミドルウェアは、`users`データベーステーブルの`email`カラムをユーザーの「ユーザー名」とみなします。

<a name="a-note-on-fastcgi"></a>
#### FastCGIに関する注意

LaravelアプリケーションをPHP FastCGIとApacheで提供している場合、HTTP Basic認証が正しく機能しない可能性があります。これらの問題を修正するには、アプリケーションの`.htaccess`ファイルに以下の行を追加してください。

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### ステートレスなHTTP Basic認証

セッションにユーザー識別子のクッキーを設定せずに、ステートレスなHTTP Basic認証を使用することもできます。これは主に、アプリケーションのAPIへのリクエストを認証するためにHTTP認証を使用する場合に便利です。これを実現するには、`onceBasic`メソッドを呼び出す[ミドルウェアを定義](middleware.md)します。`onceBasic`メソッドがレスポンスを返さない場合、リクエストはアプリケーションにさらに渡されます。

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;
    use Symfony\Component\HttpFoundation\Response;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 受信リクエストの処理
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

次に、ミドルウェアをルートにアタッチします。

    Route::get('/api/user', function () {
        // 認証されたユーザーのみがこのルートにアクセスできます...
    })->middleware(AuthenticateOnceWithBasicAuth::class);

<a name="logging-out"></a>
## ログアウト

アプリケーションから手動でユーザーをログアウトさせるには、`Auth`ファサードが提供する`logout`メソッドを使用できます。これにより、ユーザーのセッションから認証情報が削除され、後続のリクエストが認証されなくなります。

`logout`メソッドを呼び出すだけでなく、ユーザーのセッションを無効にし、[CSRFトークン](csrf.md)を再生成することを推奨します。ユーザーをログアウトした後、通常はアプリケーションのルートにユーザーをリダイレクトします。

    use Illuminate\Http\Request;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Support\Facades\Auth;

    /**
     * アプリケーションからユーザーをログアウトさせる
     */
    public function logout(Request $request): RedirectResponse
    {
        Auth::logout();

        $request->session()->invalidate();

        $request->session()->regenerateToken();

        return redirect('/');
    }

<a name="invalidating-sessions-on-other-devices"></a>
### 他のデバイスでのセッションの無効化

Laravelは、ユーザーの現在のデバイスのセッションを維持したまま、他のデバイスでアクティブなセッションを無効化して「ログアウト」させるメカニズムも提供しています。この機能は、通常、ユーザーがパスワードを変更または更新した際に、他のデバイスのセッションを無効化したいが、現在のデバイスは認証されたままにしたい場合に利用されます。

開始する前に、`Illuminate\Session\Middleware\AuthenticateSession`ミドルウェアがセッション認証を受けるルートに含まれていることを確認する必要があります。通常、このミドルウェアをルートグループの定義に配置して、アプリケーションの大部分のルートに適用できるようにします。デフォルトでは、`AuthenticateSession`ミドルウェアは`auth.session`[ミドルウェアエイリアス](middleware.md#middleware-aliases)を使用してルートにアタッチできます。

    Route::middleware(['auth', 'auth.session'])->group(function () {
        Route::get('/', function () {
            // ...
        });
    });

次に、`Auth`ファサードが提供する`logoutOtherDevices`メソッドを使用できます。このメソッドは、ユーザーが現在のパスワードを確認する必要があり、アプリケーションは入力フォームを介してこのパスワードを受け入れる必要があります。

    use Illuminate\Support\Facades\Auth;

    Auth::logoutOtherDevices($currentPassword);

`logoutOtherDevices`メソッドが呼び出されると、ユーザーの他のセッションは完全に無効化され、以前に認証されていたすべてのガードから「ログアウト」されます。

<a name="password-confirmation"></a>
## パスワード確認機能

アプリケーションを構築する際、ユーザーがアクションを実行する前、またはユーザーがアプリケーションの機密性の高い領域にリダイレクトされる前に、パスワード確認機能を求めるアクションがあるかもしれません。Laravelには、このプロセスを簡単にするための組み込みミドルウェアが含まれています。この機能を実装するには、2つのルートを定義する必要があります。1つはユーザーにパスワード確認機能を求めるビューを表示するルート、もう1つはパスワードが有効であることを確認し、ユーザーを目的の宛先にリダイレクトするルートです。

> NOTE:  
> 以下のドキュメントでは、Laravelのパスワード確認機能と直接統合する方法について説明します。ただし、より迅速に開始したい場合は、[Laravelアプリケーションスターターキットについて](starter-kits.md)にはこの機能が含まれています。

<a name="password-confirmation-configuration"></a>
### 設定

パスワードを確認した後、ユーザーは3時間以内に再度パスワード確認機能を求められることはありません。ただし、アプリケーションの`config/auth.php`設定ファイル内の`password_timeout`設定値を変更することで、ユーザーが再度パスワード確認機能を求められるまでの時間を設定できます。

<a name="password-confirmation-routing"></a>
### ルーティング

<a name="the-password-confirmation-form"></a>
#### パスワード確認フォーム

まず、ユーザーにパスワード確認機能を求めるビューを表示するルートを定義します。

    Route::get('/confirm-password', function () {
        return view('auth.confirm-password');
    })->middleware('auth')->name('password.confirm');

予想されるように、このルートによって返されるビューには`password`フィールドを含むフォームが必要です。さらに、ユーザーがアプリケーションの保護された領域に入力しており、パスワード確認機能が必要であることを説明するテキストをビューに含めることもできます。

<a name="confirming-the-password"></a>
#### パスワード確認機能

次に、「パスワード確認」ビューからのフォームリクエストを処理するルートを定義します。このルートは、パスワードを検証し、ユーザーを目的の宛先にリダイレクトする役割を果たします。

    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Facades\Redirect;

    Route::post('/confirm-password', function (Request $request) {
        if (! Hash::check($request->password, $request->user()->password)) {
            return back()->withErrors([
                'password' => ['提供されたパスワードが記録と一致しません。']
            ]);
        }

        $request->session()->passwordConfirmed();

        return redirect()->intended();
    })->middleware(['auth', 'throttle:6,1']);

このルートについて詳しく見てみましょう。まず、リクエストの`password`フィールドが認証されたユーザーのパスワードと実際に一致するかどうかを確認します。パスワードが有効な場合、Laravelのセッションにユーザーがパスワードを確認したことを通知する必要があります。`passwordConfirmed`メソッドは、ユーザーが最後にパスワードを確認した時刻を示すタイムスタンプをユーザーのセッションに設定します。最後に、ユーザーを目的の宛先にリダイレクトできます。

<a name="password-confirmation-protecting-routes"></a>
### ルートの保護

最近のパスワード確認を必要とするアクションを実行するルートには、`password.confirm`ミドルウェアを割り当てる必要があります。このミドルウェアはLaravelのデフォルトインストールに含まれており、ユーザーの目的の宛先をセッションに自動的に保存するため、ユーザーはパスワードを確認した後にその場所にリダイレクトされます。セッションにユーザーの目的の宛先を保存した後、ミドルウェアはユーザーを`password.confirm`[名前付きルート](routing.md#named-routes)にリダイレクトします。

    Route::get('/settings', function () {
        // ...
    })->middleware(['password.confirm']);

    Route::post('/settings', function () {
        // ...
    })->middleware(['password.confirm']);

<a name="adding-custom-guards"></a>
## カスタムガードの追加

`Auth`ファサードの`extend`メソッドを使用して独自の認証ガードを定義できます。[サービスプロバイダ](providers.md)内で`extend`メソッドの呼び出しを配置する必要があります。Laravelにはすでに`AppServiceProvider`が付属しているため、そのプロバイダにコードを配置できます。

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        // ...

        /**
         * 任意のアプリケーションサービスのブートストラップ
         */
        public function boot(): void
        {
            Auth::extend('jwt', function (Application $app, string $name, array $config) {
                // Illuminate\Contracts\Auth\Guardのインスタンスを返す...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

上記の例でわかるように、`extend`メソッドに渡されるコールバックは、`Illuminate\Contracts\Auth\Guard`の実装を返す必要があります。このインターフェースには、カスタムガードを定義するために実装する必要があるいくつかのメソッドが含まれています。カスタムガードを定義したら、`auth.php`設定ファイルの`guards`設定でそのガードを参照できます。

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="closure-request-guards"></a>
### クロージャリクエストガード

カスタムのHTTPリクエストベースの認証システムを実装する最も簡単な方法は、`Auth::viaRequest`メソッドを使用することです。このメソッドを使用すると、単一のクロージャを使用して認証プロセスを迅速に定義できます。

始めるには、アプリケーションの`AppServiceProvider`の`boot`メソッド内で`Auth::viaRequest`メソッドを呼び出します。`viaRequest`メソッドは、認証ドライバ名を最初の引数として受け取ります。この名前は、カスタムガードを説明する任意の文字列にすることができます。メソッドに渡される2番目の引数は、受信したHTTPリクエストを受け取り、認証が成功した場合はユーザーインスタンスを、失敗した場合は`null`を返すクロージャである必要があります。

    use App\Models\User;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Auth::viaRequest('custom-token', function (Request $request) {
            return User::where('token', (string) $request->token)->first();
        });
    }

カスタム認証ドライバを定義したら、`auth.php`設定ファイルの`guards`設定内でドライバとして設定できます。

    'guards' => [
        'api' => [
            'driver' => 'custom-token',
        ],
    ],

最後に、認証ミドルウェアをルートに割り当てる際にガードを参照できます。

    Route::middleware('auth:api')->group(function () {
        // ...
    });

<a name="adding-custom-user-providers"></a>
## カスタムユーザープロバイダの追加

ユーザーを保存するために従来のリレーショナルデータベースを使用していない場合は、Laravelを独自の認証ユーザープロバイダで拡張する必要があります。`Auth`ファサードの`provider`メソッドを使用して、カスタムユーザープロバイダを定義します。ユーザープロバイダリゾルバは、`Illuminate\Contracts\Auth\UserProvider`の実装を返す必要があります。

    <?php

    namespace App\Providers;

    use App\Extensions\MongoUserProvider;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        // ...

        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Auth::provider('mongo', function (Application $app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new MongoUserProvider($app->make('mongo.connection'));
            });
        }
    }

`provider`メソッドを使用してプロバイダを登録した後、`auth.php`設定ファイルで新しいユーザープロバイダに切り替えることができます。まず、新しいドライバを使用するプロバイダを定義します。

    'providers' => [
        'users' => [
            'driver' => 'mongo',
        ],
    ],

最後に、このプロバイダを`guards`設定で参照できます。

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### ユーザープロバイダ契約

`Illuminate\Contracts\Auth\UserProvider`の実装は、MySQL、MongoDBなどの永続ストレージシステムから`Illuminate\Contracts\Auth\Authenticatable`の実装を取得する責任があります。これらの2つのインターフェースにより、ユーザーデータがどのように保存されているか、または認証されたユーザーを表すためにどのようなクラスが使用されているかに関係なく、Laravelの認証メカニズムが機能し続けることができます。

`Illuminate\Contracts\Auth\UserProvider`契約を見てみましょう。

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider
    {
        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);
        public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
    }

`retrieveById`関数は通常、MySQLデータベースからの自動増分IDなど、ユーザーを表すキーを受け取ります。IDに一致する`Authenticatable`の実装がメソッドによって取得され、返される必要があります。

`retrieveByToken`関数は、ユーザーの一意の`$identifier`と「remember me」`$token`によってユーザーを取得します。通常、これは`remember_token`というデータベース列に保存されます。前のメソッドと同様に、トークン値が一致する`Authenticatable`の実装がこのメソッドによって返される必要があります。

`updateRememberToken`メソッドは、`$user`インスタンスの`remember_token`を新しい`$token`で更新します。新しいトークンは、ユーザーが「remember me」認証の試みに成功したとき、またはユーザーがログアウトしたときに割り当てられます。

`retrieveByCredentials`メソッドは、アプリケーションに認証を試みるときに`Auth::attempt`メソッドに渡される資格情報の配列を受け取ります。このメソッドは、その資格情報に一致するユーザーを基礎となる永続ストレージに「クエリ」する必要があります。通常、このメソッドは、`$credentials['username']`の値に一致する「username」を持つユーザーレコードを検索する「where」条件を持つクエリを実行します。このメソッドは、`Authenticatable`の実装を返す必要があります。**このメソッドは、パスワードの検証や認証を試みるべきではありません。**

`validateCredentials`メソッドは、指定された`$user`を`$credentials`と比較してユーザーを認証する必要があります。例えば、このメソッドは通常、`Hash::check`メソッドを使用して、`$user->getAuthPassword()`の値を`$credentials['password']`の値と比較します。このメソッドは、パスワードが有効かどうかを示す`true`または`false`を返す必要があります。

`rehashPasswordIfRequired`メソッドは、必要であれば指定された`$user`のパスワードを再ハッシュする必要があります。例えば、このメソッドは通常、`Hash::needsRehash`メソッドを使用して、`$credentials['password']`の値を再ハッシュする必要があるかどうかを判断します。パスワードを再ハッシュする必要がある場合、このメソッドは`Hash::make`メソッドを使用してパスワードを再ハッシュし、基礎となる永続ストレージ内のユーザーレコードを更新する必要があります。

<a name="the-authenticatable-contract"></a>
### Authenticatable契約

`UserProvider`の各メソッドについて調べたので、`Authenticatable`契約を見てみましょう。`retrieveById`、`retrieveByToken`、`retrieveByCredentials`メソッドからは、このインターフェースの実装を返す必要があります。

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable
    {
        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPasswordName();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();
    }

このインターフェースはシンプルです。`getAuthIdentifierName`メソッドは、ユーザーの「主キー」列の名前を返す必要があり、`getAuthIdentifier`メソッドはユーザーの「主キー」を返す必要があります。MySQLバックエンドを使用する場合、これはユーザーレコードに割り当てられた自動増分主キーである可能性があります。`getAuthPasswordName`メソッドは、ユーザーのパスワード列の名前を返す必要があります。`getAuthPassword`メソッドは、ユーザーのハッシュ化されたパスワードを返す必要があります。

このインターフェースにより、認証システムはORMまたはストレージ抽象化レイヤーに関係なく、任意の「ユーザー」クラスで動作できます。デフォルトでは、Laravelには`app/Models`ディレクトリにこのインターフェースを実装する`App\Models\User`クラスが含まれています。

<a name="automatic-password-rehashing"></a>
## 自動パスワード再ハッシュ

Laravelのデフォルトのパスワードハッシュアルゴリズムはbcryptです。bcryptハッシュの「作業係数」は、アプリケーションの`config/hashing.php`設定ファイルまたは`BCRYPT_ROUNDS`環境変数を介して調整できます。

通常、bcrypt作業係数は、CPU/GPU処理能力の向上に伴い時間とともに増加する必要があります。アプリケーションのbcrypt作業係数を増やす場合、Laravelは、Laravelのスターターキットを介してユーザーがアプリケーションに認証するとき、または`attempt`メソッドを介して[手動でユーザーを認証する](#authenticating-users)ときに、ユーザーパスワードを優雅に自動的に再ハッシュします。

通常、自動パスワード再ハッシュはアプリケーションに影響を与えるべきではありませんが、この動作を無効にすることができます。`hashing`設定ファイルを公開することで無効にできます。

```shell
php artisan config:publish hashing
```

設定ファイルが公開されたら、`rehash_on_login`設定値を`false`に設定できます。

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## イベント

Laravelは、認証プロセス中にさまざまな[イベント](events.md)をディスパッチします。次のイベントのいずれかに[リスナーを定義](events.md)することができます。

<div class="overflow-auto" markdown=1>

| イベント名 |
| --- |
| `Illuminate\Auth\Events\Registered` |
| `Illuminate\Auth\Events\Attempting` |
| `Illuminate\Auth\Events\Authenticated` |
| `Illuminate\Auth\Events\Login` |
| `Illuminate\Auth\Events\Failed` |
| `Illuminate\Auth\Events\Validated` |
| `Illuminate\Auth\Events\Verified` |
| `Illuminate\Auth\Events\Logout` |
| `Illuminate\Auth\Events\CurrentDeviceLogout` |
| `Illuminate\Auth\Events\OtherDeviceLogout` |
| `Illuminate\Auth\Events\Lockout` |
| `Illuminate\Auth\Events\PasswordReset` |

</div>
