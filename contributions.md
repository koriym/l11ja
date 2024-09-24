# 貢献ガイド

- [バグ報告](#bug-reports)
- [サポート質問](#support-questions)
- [コア開発の議論](#core-development-discussion)
- [どのブランチ？](#which-branch)
- [コンパイル済みアセット](#compiled-assets)
- [セキュリティ脆弱性](#security-vulnerabilities)
- [コーディングスタイル](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [行動規範](#code-of-conduct)

<a name="bug-reports"></a>
## バグ報告

アクティブなコラボレーションを促進するために、Laravelはバグ報告だけでなくプルリクエストを強く推奨しています。プルリクエストは、「レビュー準備完了」（「ドラフト」状態ではない）とマークされ、新機能のすべてのテストがパスしている場合にのみレビューされます。「ドラフト」状態のまま放置された非アクティブなプルリクエストは、数日後にクローズされます。

ただし、バグ報告を提出する場合、あなたの問題にはタイトルと問題の明確な説明が含まれている必要があります。また、関連する情報をできるだけ多く含め、問題を示すコードサンプルも含める必要があります。バグ報告の目的は、自分自身と他の人々がバグを再現し、修正を開発するのを容易にすることです。

バグ報告は、同じ問題を抱える他の人々があなたと協力して問題を解決することを期待して作成されることを忘れないでください。バグ報告が自動的に何らかのアクティビティを見ることや、他の人々がすぐに修正に取り掛かることを期待しないでください。バグ報告を作成することは、自分自身と他の人々が問題の修正に向けて道を始めるのを助けるためのものです。もし貢献したい場合は、[私たちの問題トラッカーにリストされているバグ](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel)を修正することで手助けすることができます。Laravelのすべての問題を表示するには、GitHubで認証されている必要があります。

Laravelを使用中に不適切なDocBlock、PHPStan、またはIDEの警告に気づいた場合、GitHubの問題を作成しないでください。代わりに、問題を修正するためのプルリクエストを提出してください。

LaravelのソースコードはGitHubで管理されており、Laravelプロジェクトごとにリポジトリがあります：

<div class="content-list" markdown="1">

- [Laravelアプリケーション](https://github.com/laravel/laravel)
- [Laravelアート](https://github.com/laravel/art)
- [Laravelドキュメンテーション](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Folio](https://github.com/laravel/folio)
- [Laravelフレームワーク](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead) ([ビルドスクリプト](https://github.com/laravel/settler))
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Jetstream](https://github.com/laravel/jetstream)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Prompts](https://github.com/laravel/prompts)
- [Laravel Reverb](https://github.com/laravel/reverb)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravelウェブサイト](https://github.com/laravel/laravel.com)

</div>

<a name="support-questions"></a>
## サポート質問

LaravelのGitHub問題トラッカーは、Laravelのヘルプやサポートを提供するためのものではありません。代わりに、以下のチャンネルを使用してください：

<div class="content-list" markdown="1">

- [GitHubディスカッション](https://github.com/laravel/framework/discussions)
- [Laracastsフォーラム](https://laracasts.com/discuss)
- [Laravel.ioフォーラム](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## コア開発の議論

Laravelフレームワークリポジトリの[GitHubディスカッションボード](https://github.com/laravel/framework/discussions)で、新機能や既存のLaravelの動作の改善を提案することができます。新機能を提案する場合、その機能を完了するために必要なコードの少なくとも一部を実装する意思があることを示してください。

バグ、新機能、および既存の機能の実装に関する非公式な議論は、[Laravel Discordサーバー](https://discord.gg/laravel)の`#internals`チャンネルで行われます。LaravelのメンテナであるTaylor Otwellは、通常、平日の午前8時から午後5時（UTC-06:00またはアメリカ/シカゴ）にチャンネルに参加しており、他の時間帯には散発的に参加しています。

<a name="which-branch"></a>
## どのブランチ？

**すべての**バグ修正は、バグ修正をサポートする最新バージョン（現在は`10.x`）に送信する必要があります。バグ修正は、次のリリースにのみ存在する機能を修正しない限り、**決して**`master`ブランチに送信しないでください。

**現在のリリースと完全に後方互換性のある**マイナーな機能は、最新の安定ブランチ（現在は`11.x`）に送信することができます。

**メジャーな新機能または破壊的変更を伴う機能**は、常に次のリリースを含む`master`ブランチに送信する必要があります。

<a name="compiled-assets"></a>
## コンパイル済みアセット

`laravel/laravel`リポジトリの`resources/css`や`resources/js`などのコンパイル済みファイルに影響を与える変更を提出する場合、コンパイル済みファイルをコミットしないでください。それらのファイルのサイズが大きいため、メンテナーが現実的にレビューできないためです。これは、悪意のあるコードをLaravelに注入する方法として悪用される可能性があります。これを防御的に防ぐために、すべてのコンパイル済みファイルはLaravelメンテナーによって生成され、コミットされます。

<a name="security-vulnerabilities"></a>
## セキュリティ脆弱性

Laravel内でセキュリティ脆弱性を発見した場合は、Taylor Otwellにメールを送信してください：<a href="mailto:taylor@laravel.com">taylor@laravel.com</a>。すべてのセキュリティ脆弱性は迅速に対処されます。

<a name="coding-style"></a>
## コーディングスタイル

Laravelは[PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md)コーディング規約と[PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md)オートローディング規約に従います。

<a name="phpdoc"></a>
### PHPDoc

以下は、有効なLaravelドキュメンテーションブロックの例です。`@param`属性の後には2つのスペース、引数の型、さらに2つのスペース、最後に変数名が続くことに注意してください：

    /**
     * コンテナへのバインディングを登録します。
     *
     * @param  string|array  $abstract
     * @param  \Closure|string|null  $concrete
     * @param  bool  $shared
     * @return void
     *
     * @throws \Exception
     */
    public function bind($abstract, $concrete = null, $shared = false)
    {
        // ...
    }

`@param`または`@return`属性がネイティブ型の使用により冗長である場合、それらは削除できます：

    /**
     * ジョブを実行します。
     */
    public function handle(AudioProcessor $processor): void
    {
        //
    }

ただし、ネイティブ型がジェネリックである場合、`@param`または`@return`属性を使用してジェネリック型を指定してください：

    /**
     * メッセージの添付ファイルを取得します。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file'),
        ];
    }

<a name="styleci"></a>
### StyleCI

コードスタイリングが完璧でなくても心配しないでください！[StyleCI](https://styleci.io/)は、プルリクエストがマージされた後、自動的にスタイル修正をLaravelリポジトリにマージします。これにより、コードスタイルではなく、貢献の内容に集中することができます。

<a name="code-of-conduct"></a>
## 行動規範

Laravelの行動規範はRubyの行動規範に基づいています。行動規範の違反は、Taylor Otwell（taylor@laravel.com）に報告することができます：

<div class="content-list" markdown="1">

- 参加者は反対意見に対して寛容であるべきです。
- 参加者は、個人的な攻撃や個人的な発言がないようにする必要があります。
- 他者の言葉や行動を解釈する際、参加者は常に善意を前提とするべきです。
- 嫌がらせと見なされる行動は容認されません。

</div>

