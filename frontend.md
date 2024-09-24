# フロントエンド

- [はじめに](#introduction)
- [PHPを使用する](#using-php)
    - [PHPとBlade](#php-and-blade)
    - [Livewire](#livewire)
    - [スターターキット](#php-starter-kits)
- [Vue / Reactを使用する](#using-vue-react)
    - [Inertia](#inertia)
    - [スターターキット](#inertia-starter-kits)
- [アセットのバンドル](#bundling-assets)

<a name="introduction"></a>
## はじめに

Laravelは、[ルーティング](routing.md)、[バリデーション](validation.md)、[キャッシュ](cache.md)、[キュー](queues.md)、[ファイルストレージ](filesystem.md)など、モダンなWebアプリケーションを構築するために必要なすべての機能を提供するバックエンドフレームワークです。しかし、開発者に美しいフルスタック体験を提供することが重要であると考えています。これには、アプリケーションのフロントエンドを構築するための強力なアプローチも含まれます。

Laravelでアプリケーションを構築する際のフロントエンド開発には、主に2つの方法があります。どちらのアプローチを選択するかは、PHPを活用してフロントエンドを構築するか、VueやReactなどのJavaScriptフレームワークを使用するかによって決まります。以下では、両方のオプションについて説明し、アプリケーションのフロントエンド開発に最適なアプローチを決定するための情報を提供します。

<a name="using-php"></a>
## PHPを使用する

<a name="php-and-blade"></a>
### PHPとBlade

過去には、ほとんどのPHPアプリケーションは、リクエスト中にデータベースから取得したデータをレンダリングするために、PHPの`echo`ステートメントを含む単純なHTMLテンプレートを使用してHTMLをブラウザにレンダリングしていました。

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Laravelでは、このHTMLレンダリングのアプローチは、[ビュー](views.md)と[Blade](blade.md)を使用して引き続き実現できます。Bladeは、データの表示やデータの反復処理などを行うための便利で短い構文を提供する非常に軽量なテンプレート言語です。

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

このようにアプリケーションを構築する場合、フォームの送信やその他のページのインタラクションは通常、サーバーから新しいHTMLドキュメントを受け取り、ブラウザによってページ全体が再レンダリングされます。今日でも、多くのアプリケーションは、単純なBladeテンプレートを使用してフロントエンドを構築することに完全に適しているかもしれません。

<a name="growing-expectations"></a>
#### 期待の高まり

しかし、ユーザーがWebアプリケーションに対する期待が成熟するにつれて、多くの開発者は、より動的でインタラクションがより洗練されたフロントエンドを構築する必要性を感じています。このため、一部の開発者は、VueやReactなどのJavaScriptフレームワークを使用してアプリケーションのフロントエンドの構築を開始することを選択しています。

他の開発者は、慣れ親しんだバックエンド言語を使用してモダンなWebアプリケーションのUIを構築できるソリューションを開発しています。例えば、[Rails](https://rubyonrails.org/)エコシステムでは、これにより[Turbo](https://turbo.hotwired.dev/)、[Hotwire](https://hotwired.dev/)、[Stimulus](https://stimulus.hotwired.dev/)などのライブラリが生まれました。

Laravelエコシステム内では、主にPHPを使用してモダンで動的なフロントエンドを作成する必要性から、[Laravel Livewire](https://livewire.laravel.com)と[Alpine.js](https://alpinejs.dev/)が作成されました。

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com)は、VueやReactのようなモダンなJavaScriptフレームワークを使用して構築されたフロントエンドのように、動的でモダンで生き生きとしたフロントエンドを構築するためのフレームワークです。

Livewireを使用する場合、UIの離散部分をレンダリングし、アプリケーションのフロントエンドから呼び出して操作できるメソッドとデータを公開するLivewire「コンポーネント」を作成します。例えば、シンプルな「カウンター」コンポーネントは次のようになります。

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

そして、カウンターの対応するテンプレートは次のように記述されます。

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

ご覧のように、Livewireを使用すると、Laravelアプリケーションのフロントエンドとバックエンドを接続する`wire:click`のような新しいHTML属性を記述できます。さらに、単純なBlade式を使用してコンポーネントの現在の状態をレンダリングできます。

多くの開発者にとって、LivewireはLaravelでのフロントエンド開発を革命的に変え、モダンで動的なWebアプリケーションを構築しながらLaravelの快適さの中にとどまることを可能にしました。通常、Livewireを使用する開発者は、ダイアログウィンドウのレンダリングなど、必要な場所にJavaScriptを「振りかける」ために[Alpine.js](https://alpinejs.dev/)も使用します。

Laravelを初めて使用する場合は、[ビュー](views.md)と[Blade](blade.md)の基本的な使用方法に慣れることをお勧めします。その後、公式の[Laravel Livewireドキュメント](https://livewire.laravel.com/docs)を参照して、インタラクティブなLivewireコンポーネントを使用してアプリケーションを次のレベルに引き上げる方法を学んでください。

<a name="php-starter-kits"></a>
### スターターキット

PHPとLivewireを使用してフロントエンドを構築したい場合は、BreezeまたはJetstreamの[スターターキット](starter-kits.md)を活用して、アプリケーションの開発をスタートさせることができます。これらのスターターキットは、[Blade](blade.md)と[Tailwind](https://tailwindcss.com)を使用してアプリケーションのバックエンドとフロントエンドの認証フローをスキャフォールディングし、次の大きなアイデアの構築を簡単に開始できるようにします。

<a name="using-vue-react"></a>
## Vue / Reactを使用する

LaravelとLivewireを使用してモダンなフロントエンドを構築することは可能ですが、多くの開発者は依然としてVueやReactなどのJavaScriptフレームワークの力を活用することを好みます。これにより、開発者はNPMを介して利用可能な豊富なJavaScriptパッケージとツールを活用できます。

しかし、追加のツールがなければ、LaravelをVueやReactと組み合わせると、クライアントサイドルーティング、データのハイドレーション、認証など、さまざまな複雑な問題を解決する必要があります。クライアントサイドルーティングは、[Nuxt](https://nuxt.com/)や[Next](https://nextjs.org/)のような意見を持つVue / Reactフレームワークを使用することで簡素化されることが多いですが、データのハイドレーションと認証は、バックエンドフレームワークとしてLaravelをこれらのフロントエンドフレームワークと組み合わせる場合、依然として複雑で面倒な問題です。

さらに、開発者は2つの別々のコードリポジトリを管理し、しばしば両方のリポジトリ間でメンテナンス、リリース、デプロイメントを調整する必要があります。これらの問題は克服できないわけではありませんが、アプリケーションを開発するための生産的で楽しい方法ではないと考えています。

<a name="inertia"></a>
### Inertia

幸いなことに、Laravelは両方の世界のベストを提供しています。[Inertia](https://inertiajs.com)は、LaravelアプリケーションとモダンなVueまたはReactフロントエンドの間のギャップを埋め、Laravelのルートとコントローラーを使用してルーティング、データのハイドレーション、認証を行いながら、VueまたはReactを使用してフルスタックのモダンなフロントエンドを構築できるようにします。このアプローチでは、LaravelとVue / Reactの両方の力を最大限に活用でき、いずれのツールの機能も制限されることはありません。

LaravelアプリケーションにInertiaをインストールした後、通常どおりルートとコントローラーを記述します。ただし、コントローラーからBladeテンプレートを返す代わりに、Inertiaページを返します。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * 指定されたユーザーのプロフィールを表示します。
     */
    public function show(string $id): Response
    {
        return Inertia::render('Users/Profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Inertiaページは、通常アプリケーションの`resources/js/Pages`ディレクトリに格納されるVueまたはReactコンポーネントに対応します。`Inertia::render`メソッドを介してページに与えられたデータは、ページコンポーネントの「props」をハイドレートするために使用されます。

```vue
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="User Profile" />

    <Layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Profile
            </h2>
        </template>

        <div class="py-12">
            Hello, {{ user.name }}
        </div>
    </Layout>
</template>
```

ご覧のように、Inertiaを使用すると、フロントエンドを構築する際にVueまたはReactの全機能を活用でき、LaravelバックエンドとJavaScriptフロントエンドの間に軽量なブリッジを提供します。

#### サーバーサイドレンダリング

アプリケーションがサーバーサイドレンダリングを必要とするためにInertiaに飛び込むことに懸念がある場合は、心配する必要はありません。Inertiaは[サーバーサイドレンダリングのサポート](https://inertiajs.com/server-side-rendering)を提供しています。また、[Laravel Forge](https://forge.laravel.com)を介してアプリケーションをデプロイする場合、Inertiaのサーバーサイドレンダリングプロセスが常に実行されていることを確認するのは簡単です。

<a name="inertia-starter-kits"></a>
### スターターキット

VueまたはReactを使用してフロントエンドを構築したい場合は、BreezeまたはJetstreamの[スターターキット](starter-kits.md)を活用して、アプリケーションの開発をスタートさせることができます。これらのスターターキットは、[Blade](blade.md)と[Tailwind](https://tailwindcss.com)を使用してアプリケーションのバックエンドとフロントエンドの認証フローをスキャフォールディングし、次の大きなアイデアの構築を簡単に開始できるようにします。

InertiaとVue / Reactを使ってフロントエンドを構築したい場合、アプリケーションの開発を迅速に開始するために、BreezeまたはJetstreamの[スターターキット](starter-kits.md#breeze-and-inertia)を活用できます。これらのスターターキットは、Inertia、Vue / React、[Tailwind](https://tailwindcss.com)、および[Vite](https://vitejs.dev)を使用して、アプリケーションのバックエンドとフロントエンドの認証フローをスキャフォールディングし、次の大きなアイデアを構築するための準備を整えます。

<a name="bundling-assets"></a>
## アセットのバンドル

フロントエンドをBladeとLivewireで開発するか、Vue / ReactとInertiaで開発するかに関わらず、アプリケーションのCSSを本番環境に対応したアセットにバンドルする必要があるでしょう。もちろん、アプリケーションのフロントエンドをVueまたはReactで構築する場合、コンポーネントをブラウザ対応のJavaScriptアセットにバンドルする必要もあります。

デフォルトで、Laravelはアセットをバンドルするために[Vite](https://vitejs.dev)を利用します。Viteは、ローカル開発中のビルド時間が非常に速く、ほぼ即時のホットモジュール置換（HMR）を提供します。すべての新しいLaravelアプリケーション、および[スターターキット](starter-kits.md)を使用するアプリケーションでは、`vite.config.js`ファイルが見つかります。このファイルは、LaravelアプリケーションでViteを使いやすくするための軽量なLaravel Viteプラグインを読み込みます。

LaravelとViteを使って最速で始める方法は、[Laravel Breeze](starter-kits.md#laravel-breeze)を使ってアプリケーションの開発を始めることです。これは、フロントエンドとバックエンドの認証スキャフォールディングを提供する、最もシンプルなスターターキットです。

> NOTE:  
> LaravelでViteを利用するための詳細なドキュメントについては、[アセットのバンドルとコンパイルに関する専用ドキュメント](vite.md)を参照してください。

