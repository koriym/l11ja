# スターターキット

- [はじめに](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [インストール](#laravel-breeze-installation)
    - [BreezeとBlade](#breeze-and-blade)
    - [BreezeとLivewire](#breeze-and-livewire)
    - [BreezeとReact / Vue](#breeze-and-inertia)
    - [BreezeとNext.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## はじめに

新しいLaravelアプリケーションの構築をスムーズに始められるように、認証とアプリケーションのスターターキットを提供しています。これらのキットは、アプリケーションのユーザーを登録し認証するために必要なルート、コントローラ、ビューを自動的にスキャフォールディングします。

これらのスターターキットを使用することは歓迎されますが、必須ではありません。新しいLaravelのコピーをインストールするだけで、アプリケーションを一から構築することも自由です。どちらの方法でも、素晴らしいものを作ることができることを知っています！

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze)は、Laravelのすべての[認証機能](authentication.md)を含む最小限かつシンプルな実装です。ログイン、登録、パスワードリセット、メール検証、パスワード確認などが含まれます。さらに、Breezeにはユーザーが名前、メールアドレス、パスワードを更新できるシンプルな「プロフィール」ページが含まれています。

Laravel Breezeのデフォルトのビューレイヤーは、[Tailwind CSS](https://tailwindcss.com)でスタイルされたシンプルな[Bladeテンプレート](blade.md)で構成されています。さらに、Breezeは[Livewire](https://livewire.laravel.com)または[Inertia](https://inertiajs.com)に基づくスキャフォールディングオプションを提供し、InertiaベースのスキャフォールディングにはVueまたはReactを選択できます。

<img src="https://laravel.com/img/docs/breeze-register.png">

#### Laravel Bootcamp

Laravelを初めて使う方は、[Laravel Bootcamp](https://bootcamp.laravel.com)に飛び込んでみてください。Laravel Bootcampでは、Breezeを使って最初のLaravelアプリケーションを構築する手順を説明します。LaravelとBreezeが提供するすべての機能を体験するのに最適な方法です。

<a name="laravel-breeze-installation"></a>
### インストール

まず、[新しいLaravelアプリケーションを作成](installation.md)する必要があります。[Laravelインストーラ](installation.md#creating-a-laravel-project)を使用してアプリケーションを作成する場合、インストールプロセス中にLaravel Breezeのインストールを促されます。そうでない場合は、以下の手動インストール手順に従う必要があります。

すでにスターターキットなしで新しいLaravelアプリケーションを作成している場合は、Composerを使用してLaravel Breezeを手動でインストールできます：

```shell
composer require laravel/breeze --dev
```

ComposerがLaravel Breezeパッケージをインストールした後、`breeze:install` Artisanコマンドを実行する必要があります。このコマンドは、認証ビュー、ルート、コントローラ、その他のリソースをアプリケーションに公開します。Laravel Breezeはすべてのコードをアプリケーションに公開するため、その機能と実装を完全に制御し、可視化することができます。

`breeze:install`コマンドは、お好みのフロントエンドスタックとテストフレームワークを尋ねます：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

<a name="breeze-and-blade"></a>
### BreezeとBlade

デフォルトのBreeze「スタック」はBladeスタックで、シンプルな[Bladeテンプレート](blade.md)を使用してアプリケーションのフロントエンドをレンダリングします。Bladeスタックは、追加の引数なしで`breeze:install`コマンドを実行し、Bladeフロントエンドスタックを選択することでインストールできます。Breezeのスキャフォールディングがインストールされた後、アプリケーションのフロントエンドアセットもコンパイルする必要があります：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

次に、アプリケーションの`/login`または`/register`URLにブラウザでアクセスできます。Breezeのすべてのルートは`routes/auth.php`ファイル内で定義されています。

> NOTE:  
> アプリケーションのCSSとJavaScriptをコンパイルする方法について詳しく知りたい場合は、Laravelの[Viteドキュメント](vite.md#running-vite)を確認してください。

<a name="breeze-and-livewire"></a>
### BreezeとLivewire

Laravel Breezeは、[Livewire](https://livewire.laravel.com)スキャフォールディングも提供します。Livewireは、PHPのみを使用して動的でリアクティブなフロントエンドUIを構築する強力な方法です。

Livewireは、主にBladeテンプレートを使用し、VueやReactのようなJavaScript駆動のSPAフレームワークの代わりにシンプルな選択肢を探しているチームにとって最適です。

Livewireスタックを使用するには、`breeze:install` Artisanコマンドを実行する際にLivewireフロントエンドスタックを選択します。Breezeのスキャフォールディングがインストールされた後、データベースマイグレーションを実行する必要があります：

```shell
php artisan breeze:install

php artisan migrate
```

<a name="breeze-and-inertia"></a>
### BreezeとReact / Vue

Laravel Breezeは、[Inertia](https://inertiajs.com)フロントエンド実装を介してReactとVueのスキャフォールディングも提供します。Inertiaを使用すると、クラシックなサーバーサイドルーティングとコントローラを使用して、モダンなシングルページのReactとVueアプリケーションを構築できます。

Inertiaを使用すると、ReactとVueのフロントエンドの力と、Laravelのバックエンドの生産性と[Vite](https://vitejs.dev)のコンパイル速度を組み合わせることができます。Inertiaスタックを使用するには、`breeze:install` Artisanコマンドを実行する際にVueまたはReactフロントエンドスタックを選択します。

VueまたはReactフロントエンドスタックを選択すると、Breezeインストーラーは[Inertia SSR](https://inertiajs.com/server-side-rendering)またはTypeScriptサポートを希望するかどうかも尋ねます。Breezeのスキャフォールディングがインストールされた後、アプリケーションのフロントエンドアセットもコンパイルする必要があります：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

次に、アプリケーションの`/login`または`/register`URLにブラウザでアクセスできます。Breezeのすべてのルートは`routes/auth.php`ファイル内で定義されています。

<a name="breeze-and-next"></a>
### BreezeとNext.js / API

Laravel Breezeは、[Next](https://nextjs.org)、[Nuxt](https://nuxt.com)などのモダンなJavaScriptアプリケーションを認証するための認証APIのスキャフォールディングも提供します。始めるには、`breeze:install` Artisanコマンドを実行する際にAPIスタックを希望するスタックとして選択します：

```shell
php artisan breeze:install

php artisan migrate
```

インストール中、Breezeはアプリケーションの`.env`ファイルに`FRONTEND_URL`環境変数を追加します。このURLは、JavaScriptアプリケーションのURLである必要があります。これは通常、ローカル開発中は`http://localhost:3000`です。さらに、`APP_URL`が`http://localhost:8000`に設定されていることを確認してください。これは、`serve` Artisanコマンドで使用されるデフォルトのURLです。

<a name="next-reference-implementation"></a>
#### Next.jsリファレンス実装

最後に、このバックエンドを選択したフロントエンドとペアリングする準備が整いました。BreezeフロントエンドのNext.jsリファレンス実装は、[GitHubで利用可能](https://github.com/laravel/breeze-next)です。このフロントエンドはLaravelによってメンテナンスされており、Breezeが提供する従来のBladeとInertiaスタックと同じユーザーインターフェースを含んでいます。

<a name="laravel-jetstream"></a>
## Laravel Jetstream

Laravel Breezeは、Laravelアプリケーションを構築するためのシンプルで最小限の出発点を提供しますが、Jetstreamはその機能をより堅牢な機能と追加のフロントエンド技術スタックで強化します。**Laravelを初めて使用する方には、Laravel Jetstreamに進む前にLaravel Breezeで基本を学ぶことをお勧めします。**

Jetstreamは、Laravelのために美しく設計されたアプリケーションスキャフォールディングを提供し、ログイン、登録、メール検証、二要素認証、セッション管理、Laravel SanctumによるAPIサポート、およびオプションのチーム管理を含みます。Jetstreamは[Tailwind CSS](https://tailwindcss.com)を使用して設計され、[Livewire](https://livewire.laravel.com)または[Inertia](https://inertiajs.com)駆動のフロントエンドスキャフォールディングを選択できます。

Laravel Jetstreamのインストールに関する完全なドキュメントは、[公式Jetstreamドキュメント](https://jetstream.laravel.com)にあります。

