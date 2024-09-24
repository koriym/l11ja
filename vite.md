# アセットバンドリング (Vite)

- [はじめに](#introduction)
- [インストールとセットアップ](#installation)
  - [Nodeのインストール](#installing-node)
  - [ViteとLaravelプラグインのインストール](#installing-vite-and-laravel-plugin)
  - [Viteの設定](#configuring-vite)
  - [スクリプトとスタイルの読み込み](#loading-your-scripts-and-styles)
- [Viteの実行](#running-vite)
- [JavaScriptの操作](#working-with-scripts)
  - [エイリアス](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [URL処理](#url-processing)
- [スタイルシートの操作](#working-with-stylesheets)
- [Bladeとルートの操作](#working-with-blade-and-routes)
  - [Viteでの静的アセットの処理](#blade-processing-static-assets)
  - [保存時のリフレッシュ](#blade-refreshing-on-save)
  - [エイリアス](#blade-aliases)
- [アセットのプリフェッチ](#asset-prefetching)
- [カスタムベースURL](#custom-base-urls)
- [環境変数](#environment-variables)
- [テストでのViteの無効化](#disabling-vite-in-tests)
- [サーバーサイドレンダリング (SSR)](#ssr)
- [スクリプトとスタイルタグの属性](#script-and-style-attributes)
  - [コンテンツセキュリティポリシー (CSP) ノンス](#content-security-policy-csp-nonce)
  - [サブリソース完全性 (SRI)](#subresource-integrity-sri)
  - [任意の属性](#arbitrary-attributes)
- [高度なカスタマイズ](#advanced-customization)
  - [開発サーバーURLの修正](#correcting-dev-server-urls)

<a name="introduction"></a>
## はじめに

[Vite](https://vitejs.dev) は、最新のフロントエンドビルドツールで、非常に高速な開発環境を提供し、本番用にコードをバンドルします。Laravelでアプリケーションを構築する場合、通常はViteを使用して、アプリケーションのCSSとJavaScriptファイルを本番用のアセットにバンドルします。

Laravelは、公式プラグインとBladeディレクティブを提供することで、Viteとシームレスに統合されています。これにより、開発と本番の両方でアセットを読み込むことができます。

> NOTE:  
> Laravel Mixを使用していますか？新しいLaravelインストールでは、ViteがLaravel Mixに代わっています。Mixのドキュメントについては、[Laravel Mix](https://laravel-mix.com/)のウェブサイトをご覧ください。Viteに切り替えたい場合は、[移行ガイド](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)を参照してください。

<a name="vite-or-mix"></a>
#### ViteとLaravel Mixの選択

Viteに移行する前は、新しいLaravelアプリケーションはアセットをバンドルする際に[Mix](https://laravel-mix.com/)を使用していました。これは[webpack](https://webpack.js.org/)をベースにしています。Viteは、リッチなJavaScriptアプリケーションを構築する際に、より高速で生産的な体験を提供することに焦点を当てています。[Inertia](https://inertiajs.com)のようなツールを使用して開発されたものを含む、シングルページアプリケーション（SPA）を開発する場合、Viteは完璧に適合します。

Viteは、JavaScriptの「スプリンクル」を使用する従来のサーバーサイドレンダリングアプリケーション、[Livewire](https://livewire.laravel.com)を使用するアプリケーションなどでもうまく機能します。ただし、JavaScriptアプリケーションで直接参照されていないアセットをビルドにコピーする機能など、Laravel Mixがサポートする一部の機能は欠けています。

<a name="migrating-back-to-mix"></a>
#### Mixに戻す

Viteのスキャフォールディングを使用して新しいLaravelアプリケーションを開始したが、Laravel Mixとwebpackに戻す必要がある場合は、問題ありません。[ViteからMixへの移行に関する公式ガイド](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix)を参照してください。

<a name="installation"></a>
## インストールとセットアップ

> NOTE:  
> 以下のドキュメントでは、Laravel Viteプラグインの手動インストールと設定方法について説明します。ただし、Laravelの[スターターキット](starter-kits.md)にはすでにこのスキャフォールディングが含まれており、LaravelとViteを使い始める最速の方法です。

<a name="installing-node"></a>
### Nodeのインストール

ViteとLaravelプラグインを実行する前に、Node.js（16+）とNPMがインストールされていることを確認する必要があります。

```sh
node -v
npm -v
```

[公式Nodeウェブサイト](https://nodejs.org/en/download/)から簡単なグラフィカルインストーラを使用して、最新バージョンのNodeとNPMを簡単にインストールできます。また、[Laravel Sail](https://laravel.com../sail)を使用している場合は、Sailを介してNodeとNPMを呼び出すことができます。

```sh
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### ViteとLaravelプラグインのインストール

新しいLaravelインストールでは、アプリケーションのルートディレクトリに`package.json`ファイルが見つかります。デフォルトの`package.json`ファイルには、ViteとLaravelプラグインを使い始めるために必要なものがすべて含まれています。NPMを介してアプリケーションのフロントエンド依存関係をインストールできます。

```sh
npm install
```

<a name="configuring-vite"></a>
### Viteの設定

Viteは、プロジェクトのルートにある`vite.config.js`ファイルを介して設定されます。このファイルは必要に応じてカスタマイズでき、アプリケーションに必要な他のプラグイン（`@vitejs/plugin-vue`や`@vitejs/plugin-react`など）をインストールできます。

Laravel Viteプラグインでは、アプリケーションのエントリポイントを指定する必要があります。これらはJavaScriptまたはCSSファイルであり、TypeScript、JSX、TSX、Sassなどのプリプロセス言語を含むことができます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

SPA（Inertiaを使用して構築されたアプリケーションを含む）を構築する場合、ViteはCSSエントリポイントなしで最適に機能します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

代わりに、JavaScriptを介してCSSをインポートする必要があります。通常、これはアプリケーションの`resources/js/app.js`ファイルで行います。

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Laravelプラグインは、複数のエントリポイントや[SSRエントリポイント](#ssr)などの高度な設定オプションもサポートしています。

<a name="working-with-a-secure-development-server"></a>
#### セキュアな開発サーバーの操作

ローカル開発ウェブサーバーがHTTPS経由でアプリケーションを提供している場合、Vite開発サーバーに接続する際に問題が発生する可能性があります。

[Laravel Herd](https://herd.laravel.com)を使用してサイトを保護している場合、または[Laravel Valet](valet.md)を使用していてアプリケーションに対して[secureコマンド](valet.md#securing-sites)を実行している場合、Laravel Viteプラグインは自動的に生成されたTLS証明書を検出して使用します。

サイトを保護するためにアプリケーションのディレクトリ名と一致しないホストを使用している場合、アプリケーションの`vite.config.js`ファイルでホストを手動で指定できます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

別のウェブサーバーを使用している場合は、信頼できる証明書を生成し、Viteを手動で設定して生成された証明書を使用する必要があります。

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

信頼できる証明書をシステムに生成できない場合は、[`@vitejs/plugin-basic-ssl`プラグイン](https://github.com/vitejs/vite-plugin-basic-ssl)をインストールして設定できます。信頼できない証明書を使用する場合、`npm run dev`コマンドを実行したときにコンソールの「Local」リンクに従って、ブラウザでViteの開発サーバーの証明書警告を受け入れる必要があります。

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### WSL2でのSailでの開発サーバーの実行

[Laravel Sail](sail.md)でVite開発サーバーをWindows Subsystem for Linux 2 (WSL2)内で実行する場合、ブラウザが開発サーバーと通信できるように、`vite.config.js`ファイルに以下の設定を追加する必要があります。

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

開発サーバーが実行されている間にファイルの変更がブラウザに反映されない場合、Viteの[`server.watch.usePolling`オプション](https://vitejs.dev/config/server-options.html#server-watch)を設定する必要があるかもしれません。

<a name="loading-your-scripts-and-styles"></a>
### スクリプトとスタイルの読み込み

Viteのエントリポイントを設定したら、アプリケーションのルートテンプレートの`<head>`に追加する`@vite()` Bladeディレクティブで参照できます。

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

JavaScriptを介してCSSをインポートしている場合は、JavaScriptエントリポイントのみを含める必要があります。

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

`@vite`ディレクティブは、Vite開発サーバーを自動検出し、Hot Module Replacementを有効にするためにViteクライアントを注入します。ビルドモードでは、コンパイルされたバージョン付きアセットを読み込みます。

必要に応じて、`@vite`ディレクティブを呼び出す際にコンパイルされたアセットのビルドパスを指定できます。

```blade
<!doctype html>
<head>
    {{-- 指定されたビルドパスは公開パスに対する相対パスです。 --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### インラインアセット

時には、アセットのバージョン付きURLにリンクするのではなく、アセットの生のコンテンツを含める必要があるかもしれません。例えば、HTMLコンテンツをPDFジェネレータに渡す際に、アセットのコンテンツをページに直接含める必要がある場合です。`Vite`ファサードが提供する`content`メソッドを使用して、Viteアセットのコンテンツを出力できます：

```blade
@use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## Viteの実行

Viteを実行する方法は2つあります。`dev`コマンドを介して開発サーバーを実行することができます。これはローカルで開発する際に便利です。開発サーバーはファイルの変更を自動的に検出し、開いているブラウザウィンドウに即座に反映します。

または、`build`コマンドを実行すると、アプリケーションのアセットをバージョン管理し、バンドルして、本番環境にデプロイする準備を整えます：

```shell
# Vite開発サーバーを実行する...
npm run dev

# 本番用にアセットをビルドしてバージョン管理する...
npm run build
```

[Sail](sail.md)をWSL2で開発サーバーとして実行している場合、[追加の設定](#configuring-hmr-in-sail-on-wsl2)が必要になるかもしれません。

<a name="working-with-scripts"></a>
## JavaScriptの操作

<a name="aliases"></a>
### エイリアス

デフォルトでは、Laravelプラグインは一般的なエイリアスを提供し、アプリケーションのアセットを便利にインポートできるようにします：

```js
{
    '@' => '/resources/js'
}
```

`'@'`エイリアスは、`vite.config.js`設定ファイルに独自のものを追加することで上書きできます：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

[Vue](https://vuejs.org/)フレームワークを使用してフロントエンドを構築したい場合は、`@vitejs/plugin-vue`プラグインもインストールする必要があります：

```sh
npm install --save-dev @vitejs/plugin-vue
```

その後、`vite.config.js`設定ファイルにプラグインを含めることができます。LaravelでVueプラグインを使用する際には、いくつかの追加オプションが必要です：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Vueプラグインは、シングルファイルコンポーネント内で参照されるアセットURLを
                    // LaravelのWebサーバーを指すように書き換えます。これを`null`に設定すると、
                    // Laravelプラグインが代わりにアセットURLをViteサーバーを指すように書き換えます。
                    base: null,

                    // Vueプラグインは絶対URLを解析し、ディスク上のファイルへの絶対パスとして扱います。
                    // これを`false`に設定すると、絶対URLはそのままになり、期待通りに
                    // 公開ディレクトリ内のアセットを参照できます。
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> NOTE:  
> Laravelの[スターターキット](starter-kits.md)には、すでに適切なLaravel、Vue、Viteの設定が含まれています。Laravel、Vue、Viteでの開発を最速で始めるには、[Laravel Breeze](starter-kits.md#breeze-and-inertia)をチェックしてください。

<a name="react"></a>
### React

[React](https://reactjs.org/)フレームワークを使用してフロントエンドを構築したい場合は、`@vitejs/plugin-react`プラグインもインストールする必要があります：

```sh
npm install --save-dev @vitejs/plugin-react
```

その後、`vite.config.js`設定ファイルにプラグインを含めることができます：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

JSXを含むファイルは、`.jsx`または`.tsx`拡張子を持つ必要があり、必要に応じてエントリーポイントを更新することを忘れないでください（[上記](#configuring-vite)のように）。

既存の`@vite`ディレクティブと一緒に追加の`@viteReactRefresh` Bladeディレクティブを含める必要もあります。

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

`@viteReactRefresh`ディレクティブは、`@vite`ディレクティブの前に呼び出す必要があります。

> NOTE:  
> Laravelの[スターターキット](starter-kits.md)には、すでに適切なLaravel、React、Viteの設定が含まれています。Laravel、React、Viteでの開発を最速で始めるには、[Laravel Breeze](starter-kits.md#breeze-and-inertia)をチェックしてください。

<a name="inertia"></a>
### Inertia

Laravel Viteプラグインは、Inertiaページコンポーネントを解決するのに役立つ便利な`resolvePageComponent`関数を提供します。以下は、Vue 3でのヘルパーの使用例です。ただし、Reactなどの他のフレームワークでもこの関数を利用できます：

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

InertiaでViteのコード分割機能を使用している場合、[アセットのプリフェッチ](#asset-prefetching)を設定することをお勧めします。

> NOTE:  
> Laravelの[スターターキット](starter-kits.md)には、すでに適切なLaravel、Inertia、Viteの設定が含まれています。Laravel、Inertia、Viteでの開発を最速で始めるには、[Laravel Breeze](starter-kits.md#breeze-and-inertia)をチェックしてください。

<a name="url-processing"></a>
### URLの処理

Viteを使用してアプリケーションのHTML、CSS、またはJSでアセットを参照する場合、いくつかの注意点があります。まず、絶対パスでアセットを参照する場合、Viteはそのアセットをビルドに含めません。したがって、そのアセットが公開ディレクトリで利用可能であることを確認する必要があります。[専用のCSSエントリーポイント](#configuring-vite)を使用する場合、ブラウザは開発中にこれらのパスをVite開発サーバーからではなく、公開ディレクトリから読み込もうとするため、絶対パスを使用することは避けてください。

相対パスでアセットを参照する場合、そのパスは参照されるファイルに対する相対パスであることを覚えておいてください。相対パスで参照されるアセットは、Viteによって書き換え、バージョン管理、およびバンドルされます。

以下のプロジェクト構造を考えてみましょう：

```nothing
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

以下の例は、Viteが相対パスと絶対パスをどのように扱うかを示しています：

```html
<!-- このアセットはViteによって処理されず、ビルドに含まれません -->
<img src="/taylor.png">

<!-- このアセットはViteによって書き換え、バージョン管理、およびバンドルされます -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## スタイルシートの操作

[ViteのCSSサポート](https://vitejs.dev/guide/features.html#css)については、Viteのドキュメントで詳しく学ぶことができます。[Tailwind](https://tailwindcss.com/)などのPostCSSプラグインを使用している場合、プロジェクトのルートに`postcss.config.js`ファイルを作成すると、Viteが自動的にそれを適用します：

```js
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

> NOTE:  
> Laravelの[スターターキット](starter-kits.md)には、すでに適切なTailwind、PostCSS、Viteの設定が含まれています。または、スターターキットを使用せずにTailwindとLaravelを使用したい場合は、[TailwindのLaravelインストールガイド](https://tailwindcss.com/docs/guides/laravel)をチェックしてください。

<a name="working-with-blade-and-routes"></a>
## Bladeとルートの操作

<a name="blade-processing-static-assets"></a>
### Viteで静的アセットを処理する

JavaScriptやCSSでアセットを参照する場合、Viteは自動的にそれらを処理し、バージョン管理します。さらに、Bladeベースのアプリケーションを構築する際に、ViteはBladeテンプレート内でのみ参照される静的アセットも処理し、バージョン管理することができます。

ただし、これを実現するためには、静的アセットをアプリケーションのエントリーポイントにインポートして、Viteに認識させる必要があります。例えば、`resources/images`に保存されたすべての画像と`resources/fonts`に保存されたすべてのフォントを処理し、バージョン管理したい場合、アプリケーションの`resources/js/app.js`エントリーポイントに以下を追加する必要があります：

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

これで、`npm run build`を実行すると、Viteがこれらのアセットを処理します。その後、`Vite::asset`メソッドを使用してこれらのアセットをBladeテンプレート内で参照できます。これにより、指定されたアセットのバージョン付きURLが返されます：

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### 保存時のリフレッシュ

アプリケーションがBladeを使用した従来のサーバーサイドレンダリングで構築されている場合、Viteはアプリケーションのビューファイルに変更を加えるたびにブラウザを自動的にリフレッシュすることで、開発ワークフローを向上させることができます。開始するには、単に`refresh`オプションを`true`に指定するだけです。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

`refresh`オプションが`true`の場合、以下のディレクトリ内のファイルを保存すると、`npm run dev`を実行している間にブラウザがページ全体をリフレッシュします。

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

`routes/**`ディレクトリを監視することは、アプリケーションのフロントエンド内でルートリンクを生成するために[Ziggy](https://github.com/tighten/ziggy)を利用している場合に便利です。

これらのデフォルトのパスがニーズに合わない場合、監視する独自のパスのリストを指定できます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

内部的には、Laravel Viteプラグインは[`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload)パッケージを使用しており、この機能の動作を微調整するための高度な設定オプションを提供しています。このレベルのカスタマイズが必要な場合、`config`定義を提供することができます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### エイリアス

JavaScriptアプリケーションでは、頻繁に参照されるディレクトリへの[エイリアスを作成](#aliases)するのが一般的です。しかし、Bladeで使用するためのエイリアスを作成することもできます。これには、`Illuminate\Support\Facades\Vite`クラスの`macro`メソッドを使用します。通常、「マクロ」は[サービスプロバイダ](providers.md)の`boot`メソッド内で定義する必要があります。

    /**
     * 任意のアプリケーションサービスをブートストラップします。
     */
    public function boot(): void
    {
        Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
    }

マクロが定義されると、テンプレート内で呼び出すことができます。たとえば、上記で定義した`image`マクロを使用して、`resources/images/logo.png`にあるアセットを参照できます。

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="asset-prefetching"></a>
## アセットのプリフェッチ

Viteのコード分割機能を使用してSPAを構築する場合、必要なアセットは各ページナビゲーションで取得されます。この動作により、UIのレンダリングが遅延する可能性があります。これが選択したフロントエンドフレームワークにとって問題となる場合、LaravelはアプリケーションのJavaScriptとCSSアセットを初期ページ読み込み時に積極的にプリフェッチする機能を提供します。

Laravelにアセットを積極的にプリフェッチするよう指示するには、[サービスプロバイダ](providers.md)の`boot`メソッド内で`Vite::prefetch`メソッドを呼び出します。

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 任意のアプリケーションサービスを登録します。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 任意のアプリケーションサービスをブートストラップします。
     */
    public function boot(): void
    {
        Vite::prefetch(concurrency: 3);
    }
}
```

上記の例では、各ページ読み込み時に最大`3`つの同時ダウンロードでアセットがプリフェッチされます。アプリケーションのニーズに合わせて同時実行数を変更したり、アプリケーションがすべてのアセットを一度にダウンロードする必要がある場合は同時実行数の制限を指定しないこともできます。

```php
/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Vite::prefetch();
}
```

デフォルトでは、プリフェッチは[ページの_load_イベント](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event)が発生したときに開始されます。プリフェッチが開始されるタイミングをカスタマイズしたい場合、Viteがリッスンするイベントを指定できます。

```php
/**
 * 任意のアプリケーションサービスをブートストラップします。
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

上記のコードにより、プリフェッチは`window`オブジェクトで手動で`vite:prefetch`イベントをディスパッチしたときに開始されます。たとえば、ページ読み込み後3秒でプリフェッチを開始することができます。

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## カスタムベースURL

Viteでコンパイルされたアセットがアプリケーションとは別のドメイン（CDNなど）にデプロイされている場合、アプリケーションの`.env`ファイル内で`ASSET_URL`環境変数を指定する必要があります。

```env
ASSET_URL=https://cdn.example.com
```

アセットURLを設定した後、すべてのアセットのURLが設定された値でプレフィックスされます。

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

[絶対URLはViteによって書き換えられない](#url-processing)ことに注意してください。したがって、絶対URLはプレフィックスされません。

<a name="environment-variables"></a>
## 環境変数

アプリケーションの`.env`ファイル内で`VITE_`をプレフィックスにして環境変数を注入することができます。

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

注入された環境変数には、`import.meta.env`オブジェクトを介してアクセスできます。

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## テストでのViteの無効化

LaravelのVite統合は、テストを実行する際にアセットを解決しようとします。これには、Vite開発サーバーを実行するか、アセットをビルドする必要があります。

テスト中にViteをモックしたい場合は、Laravelの`TestCase`クラスを拡張したテストで利用可能な`withoutVite`メソッドを呼び出すことができます。

===  "Pest"
```php
test('without vite example', function () {
    $this->withoutVite();

    // ...
});
```

===  "PHPUnit"
```php
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

すべてのテストでViteを無効にしたい場合は、ベースの`TestCase`クラスの`setUp`メソッドから`withoutVite`メソッドを呼び出すことができます。

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## サーバーサイドレンダリング（SSR）

Laravel Viteプラグインを使用すると、Viteでサーバーサイドレンダリングを設定するのが簡単になります。開始するには、`resources/js/ssr.js`にSSRエントリーポイントを作成し、Laravelプラグインに設定オプションを渡してエントリーポイントを指定します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

SSRエントリーポイントのビルドを忘れないようにするために、アプリケーションの`package.json`内の「build」スクリプトを拡張してSSRビルドを作成することをお勧めします。

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

そして、SSRサーバーをビルドして起動するには、以下のコマンドを実行します。

```sh
npm run build
node bootstrap/ssr/ssr.js
```

[Inertia](https://inertiajs.com/server-side-rendering)でSSRを使用している場合、代わりに`inertia:start-ssr` Artisanコマンドを使用してSSRサーバーを起動できます。

```sh
php artisan inertia:start-ssr
```

> NOTE:  
> Laravelの[スターターキット](starter-kits.md)には、すでに適切なLaravel、Inertia SSR、およびViteの設定が含まれています。Laravel、Inertia SSR、およびViteを最速で始めるには、[Laravel Breeze](starter-kits.md#breeze-and-inertia)をチェックしてください。

<a name="script-and-style-attributes"></a>
## スクリプトとスタイルタグの属性

<a name="content-security-policy-csp-nonce"></a>
### コンテンツセキュリティポリシー（CSP）のNonce

[コンテンツセキュリティポリシー](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)の一部として、スクリプトとスタイルタグに[`nonce`属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce)を含めたい場合、カスタム[ミドルウェア](middleware.md)内で`useCspNonce`メソッドを使用してnonceを生成または指定できます。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * 受信リクエストを処理します。
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

`useCspNonce`メソッドを呼び出した後、Laravelは自動的にすべての生成されたスクリプトとスタイルタグに`nonce`属性を含めます。

もし、Laravelの[スターターキット](starter-kits.md)に含まれる[Ziggy `@route`ディレクティブ](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy)を含む他の場所でnonceを指定する必要がある場合、`cspNonce`メソッドを使用して取得できます。

```blade
@routes(nonce: Vite::cspNonce())
```

すでに使用したいnonceがある場合は、`useCspNonce`メソッドにnonceを渡してLaravelに使用させることができます。

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### サブリソース完全性（SRI）

Viteマニフェストにアセットの`integrity`ハッシュが含まれている場合、Laravelは自動的に生成するすべてのscriptタグとstyleタグに`integrity`属性を追加し、[サブリソース完全性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)を強制します。デフォルトでは、Viteはマニフェストに`integrity`ハッシュを含みませんが、[`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPMプラグインをインストールすることで有効にできます。

```shell
npm install --save-dev vite-plugin-manifest-sri
```

その後、`vite.config.js`ファイルでこのプラグインを有効にできます。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

必要に応じて、完全性ハッシュが見つかるマニフェストキーをカスタマイズすることもできます。

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

この自動検出を完全に無効にしたい場合は、`useIntegrityKey`メソッドに`false`を渡すことができます。

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### 任意の属性

scriptタグとstyleタグに追加の属性（例えば[`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change)属性）を含める必要がある場合、`useScriptTagAttributes`と`useStyleTagAttributes`メソッドで指定できます。通常、このメソッドは[サービスプロバイダ](providers.md)から呼び出す必要があります。

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // 属性の値を指定...
    'async' => true, // 値なしの属性を指定...
    'integrity' => false, // 通常は含まれる属性を除外...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

属性を条件付きで追加する必要がある場合、コールバックを渡すことができます。このコールバックはアセットのソースパス、URL、マニフェストチャンク、およびマニフェスト全体を受け取ります。

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> WARNING:  
> Vite開発サーバーが実行されている間、`$chunk`と`$manifest`引数は`null`になります。

<a name="advanced-customization"></a>
## 高度なカスタマイズ

デフォルトでは、LaravelのViteプラグインは、ほとんどのアプリケーションで機能する合理的な規約を使用しています。しかし、時にはViteの動作をカスタマイズする必要があるかもしれません。追加のカスタマイズオプションを有効にするために、`@vite` Bladeディレクティブの代わりに使用できる以下のメソッドとオプションを提供しています。

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // "hot"ファイルをカスタマイズ...
            ->useBuildDirectory('bundle') // ビルドディレクトリをカスタマイズ...
            ->useManifestFilename('assets.json') // マニフェストファイル名をカスタマイズ...
            ->withEntryPoints(['resources/js/app.js']) // エントリーポイントを指定...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // ビルドされたアセットのバックエンドパス生成をカスタマイズ...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

`vite.config.js`ファイル内で、同じ設定を指定する必要があります。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // "hot"ファイルをカスタマイズ...
            buildDirectory: 'bundle', // ビルドディレクトリをカスタマイズ...
            input: ['resources/js/app.js'], // エントリーポイントを指定...
        }),
    ],
    build: {
      manifest: 'assets.json', // マニフェストファイル名をカスタマイズ...
    },
});
```

<a name="correcting-dev-server-urls"></a>
### 開発サーバーURLの修正

Viteエコシステム内の一部のプラグインは、スラッシュで始まるURLが常にVite開発サーバーを指すと仮定しています。しかし、Laravelの統合の性質上、そうではありません。

例えば、`vite-imagetools`プラグインは、Viteがアセットを提供している間、以下のようなURLを出力します。

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

`vite-imagetools`プラグインは、出力されたURLがViteによってインターセプトされ、プラグインが`/@imagetools`で始まるすべてのURLを処理することを期待しています。この動作を期待するプラグインを使用している場合、URLを手動で修正する必要があります。これは、`vite.config.js`ファイルで`transformOnServe`オプションを使用して行うことができます。

この特定の例では、生成されたコード内のすべての`/@imagetools`に開発サーバーURLを付加します。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

これで、Viteがアセットを提供している間、Vite開発サーバーを指すURLが出力されます。

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
