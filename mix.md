# Laravel Mix

- [はじめに](#introduction)

<a name="introduction"></a>
## はじめに

[Laravel Mix](https://github.com/laravel-mix/laravel-mix)は、[Laracasts](https://laracasts.com)の創設者であるJeffrey Wayによって開発されたパッケージで、いくつかの一般的なCSSおよびJavaScriptのプリプロセッサを使用して、Laravelアプリケーションの[webpack](https://webpack.js.org)ビルドステップを定義するための流暢なAPIを提供します。

言い換えれば、Mixを使えば、アプリケーションのCSSとJavaScriptファイルをコンパイルして最小化するのが簡単になります。簡単なメソッドチェーンを通じて、アセットパイプラインを流暢に定義できます。例えば：

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

もしあなたがwebpackやアセットコンパイルの始め方について混乱し、圧倒されたことがあるなら、Laravel Mixが気に入るでしょう。しかし、アプリケーションを開発する際に必ずしも使用する必要はありません。あなたは自由に好きなアセットパイプラインツールを使用したり、まったく使用しないこともできます。

> NOTE:  
> Viteは新しいLaravelインストールでLaravel Mixに取って代わりました。Mixのドキュメントについては、[公式のLaravel Mix](https://laravel-mix.com/)ウェブサイトをご覧ください。Viteに切り替えたい場合は、[Vite移行ガイド](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)を参照してください。

