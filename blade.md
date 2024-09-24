# Bladeテンプレート

- [はじめに](#introduction)
    - [LivewireでBladeを強化する](#supercharging-blade-with-livewire)
- [データの表示](#displaying-data)
    - [HTMLエンティティのエンコーディング](#html-entity-encoding)
    - [BladeとJavaScriptフレームワーク](#blade-and-javascript-frameworks)
- [Bladeディレクティブ](#blade-directives)
    - [If文](#if-statements)
    - [Switch文](#switch-statements)
    - [ループ](#loops)
    - [ループ変数](#the-loop-variable)
    - [条件付きクラス](#conditional-classes)
    - [追加属性](#additional-attributes)
    - [サブビューのインクルード](#including-subviews)
    - [`@once`ディレクティブ](#the-once-directive)
    - [生のPHP](#raw-php)
    - [コメント](#comments)
- [コンポーネント](#components)
    - [コンポーネントのレンダリング](#rendering-components)
    - [コンポーネントへのデータの受け渡し](#passing-data-to-components)
    - [コンポーネント属性](#component-attributes)
    - [予約語](#reserved-keywords)
    - [スロット](#slots)
    - [インラインコンポーネントビュー](#inline-component-views)
    - [動的コンポーネント](#dynamic-components)
    - [コンポーネントの手動登録](#manually-registering-components)
- [匿名コンポーネント](#anonymous-components)
    - [匿名インデックスコンポーネント](#anonymous-index-components)
    - [データプロパティ / 属性](#data-properties-attributes)
    - [親データへのアクセス](#accessing-parent-data)
    - [匿名コンポーネントのパス](#anonymous-component-paths)
- [レイアウトの構築](#building-layouts)
    - [コンポーネントを使用したレイアウト](#layouts-using-components)
    - [テンプレート継承を使用したレイアウト](#layouts-using-template-inheritance)
- [フォーム](#forms)
    - [CSRFフィールド](#csrf-field)
    - [メソッドフィールド](#method-field)
    - [バリデーションエラー](#validation-errors)
- [スタック](#stacks)
- [サービスインジェクション](#service-injection)
- [インラインBladeテンプレートのレンダリング](#rendering-inline-blade-templates)
- [Bladeフラグメントのレンダリング](#rendering-blade-fragments)
- [Bladeの拡張](#extending-blade)
    - [カスタムエコーハンドラ](#custom-echo-handlers)
    - [カスタムIf文](#custom-if-statements)

<a name="introduction"></a>
## はじめに

Bladeは、Laravelに含まれるシンプルで強力なテンプレートエンジンです。他のPHPテンプレートエンジンとは異なり、Bladeはテンプレート内でプレーンなPHPコードを使用することを制限しません。実際、すべてのBladeテンプレートはプレーンなPHPコードにコンパイルされ、変更されるまでキャッシュされるため、Bladeはアプリケーションに実質的にゼロのオーバーヘッドを追加します。Bladeテンプレートファイルは`.blade.php`ファイル拡張子を使用し、通常は`resources/views`ディレクトリに保存されます。

Bladeビューは、グローバルな`view`ヘルパーを使用してルートまたはコントローラから返されることがあります。もちろん、[ビュー](views.md)のドキュメントで説明されているように、`view`ヘルパーの2番目の引数を使用してBladeビューにデータを渡すことができます。

    Route::get('/', function () {
        return view('greeting', ['name' => 'Finn']);
    });

<a name="supercharging-blade-with-livewire"></a>
### LivewireでBladeを強化する

Bladeテンプレートを次のレベルに引き上げ、動的なインターフェースを簡単に構築したいですか？[Laravel Livewire](https://livewire.laravel.com)をチェックしてください。Livewireを使用すると、ReactやVueのようなフロントエンドフレームワークでのみ可能な動的な機能を持つBladeコンポーネントを書くことができ、多くのJavaScriptフレームワークの複雑さ、クライアントサイドレンダリング、ビルドステップを回避しながら、現代のリアクティブフロントエンドを構築するための素晴らしいアプローチを提供します。

<a name="displaying-data"></a>
## データの表示

変数を波括弧で囲むことで、Bladeビューに渡されたデータを表示できます。たとえば、次のルートがあるとします。

    Route::get('/', function () {
        return view('welcome', ['name' => 'Samantha']);
    });

`name`変数の内容を次のように表示できます。

```blade
Hello, {{ $name }}.
```

> NOTE:  
> Bladeの`{{ }}`エコー文は、XSS攻撃を防ぐために、自動的にPHPの`htmlspecialchars`関数を通過します。

ビューに渡された変数の内容を表示することに限定されません。PHP関数の結果もエコーできます。実際、Bladeエコー文の中に任意のPHPコードを配置できます。

```blade
The current UNIX timestamp is {{ time() }}.
```

<a name="html-entity-encoding"></a>
### HTMLエンティティのエンコーディング

デフォルトでは、Blade（およびLaravelの`e`関数）はHTMLエンティティを二重にエンコードします。二重エンコードを無効にしたい場合は、`AppServiceProvider`の`boot`メソッドから`Blade::withoutDoubleEncoding`メソッドを呼び出します。

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * アプリケーションサービスの初期化処理
         */
        public function boot(): void
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="displaying-unescaped-data"></a>
#### エスケープされていないデータの表示

デフォルトでは、Bladeの`{{ }}`文は自動的にPHPの`htmlspecialchars`関数を通過してXSS攻撃を防ぎます。データをエスケープしたくない場合は、次の構文を使用できます。

```blade
Hello, {!! $name !!}.
```

> WARNING:  
> アプリケーションのユーザーによって提供されたコンテンツを表示する場合は、非常に注意してください。通常、ユーザー提供のデータを表示する場合は、エスケープされた二重波括弧構文を使用してXSS攻撃を防ぐ必要があります。

<a name="blade-and-javascript-frameworks"></a>
### BladeとJavaScriptフレームワーク

多くのJavaScriptフレームワークも、ブラウザに表示されるべき式を示すために「波括弧」を使用するため、`@`記号を使用してBladeレンダリングエンジンに式をそのままにしておくように指示できます。例えば：

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

この例では、`@`記号はBladeによって削除されますが、`{{ name }}`式はBladeエンジンによってそのままにされ、JavaScriptフレームワークによってレンダリングされます。

`@`記号は、Bladeディレクティブをエスケープするためにも使用できます。

```blade
{{-- Bladeテンプレート --}}
@@if()

<!-- HTML出力 -->
@if()
```

<a name="rendering-json"></a>
#### JSONのレンダリング

JavaScript変数を初期化するために配列をビューに渡してJSONとしてレンダリングすることがあります。例えば：

```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

しかし、手動で`json_encode`を呼び出す代わりに、`Illuminate\Support\Js::from`メソッドディレクティブを使用できます。`from`メソッドは、PHPの`json_encode`関数と同じ引数を受け取りますが、結果のJSONがHTMLクォート内に正しくエスケープされることを保証します。`from`メソッドは、指定されたオブジェクトまたは配列を有効なJavaScriptオブジェクトに変換する`JSON.parse` JavaScript文を返します。

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

最新バージョンのLaravelアプリケーションスケルトンには、Bladeテンプレート内でこの機能に簡単にアクセスできる`Js`ファサードが含まれています。

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> WARNING:  
> `Js::from`メソッドは、既存の変数をJSONとしてレンダリングするためにのみ使用してください。Bladeテンプレートは正規表現に基づいており、複雑な式をディレクティブに渡そうとすると、予期しない失敗が発生する可能性があります。

<a name="the-at-verbatim-directive"></a>
#### `@verbatim`ディレクティブ

テンプレートの大部分でJavaScript変数を表示する場合、HTMLを`@verbatim`ディレクティブでラップして、各Bladeエコー文に`@`記号を付ける必要をなくすことができます。

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Bladeディレクティブ

テンプレートの継承とデータの表示に加えて、Bladeは一般的なPHP制御構造（条件文やループなど）の便利なショートカットも提供します。これらのショートカットは、PHP制御構造を非常にクリーンで簡潔な方法で操作するのに役立ち、それらのPHPの対応物にも馴染みがあります。

<a name="if-statements"></a>
### If文

`@if`、`@elseif`、`@else`、`@endif`ディレクティブを使用して`if`文を構築できます。これらのディレクティブは、対応するPHPのものと同じように機能します。

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

利便性のために、Bladeは`@unless`ディレクティブも提供します。

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

条件ディレクティブに加えて、`@isset`と`@empty`ディレクティブを、それぞれのPHP関数の便利なショートカットとして使用できます。

```blade
@isset($records)
    // $recordsが定義されており、nullでない場合...
@endisset

@empty($records)
    // $recordsが「空」の場合...
@endempty
```

<a name="authentication-directives"></a>
#### 認証ディレクティブ

`@auth`と`@guest`ディレクティブを使用して、現在のユーザーが[認証されている](authentication.md)か、ゲストであるかをすばやく判断できます。

```blade
@auth
    // ユーザーは認証されています...
@endauth

@guest
    // ユーザーは認証されていません...
@endguest
```

必要に応じて、`@auth`と`@guest`ディレクティブを使用する際にチェックする認証ガードを指定できます。

```blade
@auth('admin')
    // ユーザーは認証されています...
@endauth

@guest('admin')
    // ユーザーは認証されていません...
@endguest
```

<a name="environment-directives"></a>
#### 環境ディレクティブ

`@production`ディレクティブを使用して、アプリケーションが本番環境で実行されているかどうかを確認できます。

```blade
@production
    // 本番環境で実行中...
@endproduction
```

```blade
@production
    // 本番環境固有のコンテンツ...
@endproduction
```

または、`@env`ディレクティブを使用して、アプリケーションが特定の環境で実行されているかどうかを判断することもできます：

```blade
@env('staging')
    // アプリケーションは "staging" で実行中...
@endenv

@env(['staging', 'production'])
    // アプリケーションは "staging" または "production" で実行中...
@endenv
```

<a name="section-directives"></a>
#### セクションディレクティブ

`@hasSection`ディレクティブを使用して、テンプレート継承セクションにコンテンツがあるかどうかを判断できます：

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

セクションにコンテンツがないかどうかを判断するには、`sectionMissing`ディレクティブを使用できます：

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="session-directives"></a>
#### セッションディレクティブ

`@session`ディレクティブは、[セッション](session.md)の値が存在するかどうかを判断するために使用できます。セッションの値が存在する場合、`@session`と`@endsession`ディレクティブ内のテンプレートコンテンツが評価されます。`@session`ディレクティブのコンテンツ内で、セッションの値を表示するために`$value`変数をエコーすることができます：

```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

<a name="switch-statements"></a>
### Switch文

Switch文は、`@switch`、`@case`、`@break`、`@default`、`@endswitch`ディレクティブを使用して構築できます：

```blade
@switch($i)
    @case(1)
        最初のケース...
        @break

    @case(2)
        2番目のケース...
        @break

    @default
        デフォルトのケース...
@endswitch
```

<a name="loops"></a>
### ループ

条件文に加えて、BladeはPHPのループ構造を操作するためのシンプルなディレクティブを提供します。これらのディレクティブは、それぞれのPHPの対応物と同様に機能します：

```blade
@for ($i = 0; $i < 10; $i++)
    現在の値は {{ $i }}
@endfor

@foreach ($users as $user)
    <p>これはユーザー {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>ユーザーがいません</p>
@endforelse

@while (true)
    <p>私は永遠にループしています。</p>
@endwhile
```

> NOTE:  
> `foreach`ループを反復処理する際、[ループ変数](#the-loop-variable)を使用して、ループに関する貴重な情報（例えば、最初または最後の反復処理であるかどうか）を取得できます。

ループを使用する際、`@continue`と`@break`ディレクティブを使用して、現在の反復処理をスキップしたり、ループを終了したりすることもできます：

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

また、ディレクティブ宣言内に継続または中断条件を含めることもできます：

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### ループ変数

`foreach`ループを反復処理する際、ループ内で`$loop`変数が利用可能になります。この変数は、現在のループインデックスや、これが最初または最後の反復処理であるかどうかなど、いくつかの有用な情報にアクセスできます：

```blade
@foreach ($users as $user)
    @if ($loop->first)
        これは最初の反復処理です。
    @endif

    @if ($loop->last)
        これは最後の反復処理です。
    @endif

    <p>これはユーザー {{ $user->id }}</p>
@endforeach
```

ネストされたループ内にいる場合、親ループの`$loop`変数に`parent`プロパティを介してアクセスできます：

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            これは親ループの最初の反復処理です。
        @endif
    @endforeach
@endforeach
```

`$loop`変数には、他にもいくつかの有用なプロパティが含まれています：

<div class="overflow-auto" markdown=1>

| プロパティ           | 説明                                            |
| ------------------ | ------------------------------------------------------ |
| `$loop->index`     | 現在のループ反復処理のインデックス（0から始まる）。 |
| `$loop->iteration` | 現在のループ反復処理（1から始まる）。              |
| `$loop->remaining` | ループ内の残りの反復処理。                  |
| `$loop->count`     | 反復処理される配列内のアイテムの総数。 |
| `$loop->first`     | これが最初の反復処理であるかどうか。  |
| `$loop->last`      | これが最後の反復処理であるかどうか。   |
| `$loop->even`      | これが偶数の反復処理であるかどうか。    |
| `$loop->odd`       | これが奇数の反復処理であるかどうか。     |
| `$loop->depth`     | 現在のループのネストレベル。                 |
| `$loop->parent`    | ネストされたループ内にいる場合、親のループ変数。     |

</div>

<a name="conditional-classes"></a>
### 条件付きクラスとスタイル

`@class`ディレクティブは、条件付きでCSSクラス文字列をコンパイルします。このディレクティブは、クラスの配列を受け取ります。配列のキーには追加したいクラスを含み、値はブール式です。配列要素が数値キーを持つ場合、レンダリングされたクラスリストに常に含まれます：

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

同様に、`@style`ディレクティブを使用して、HTML要素に条件付きでインラインCSSスタイルを追加できます：

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

<a name="additional-attributes"></a>
### 追加の属性

便宜上、`@checked`ディレクティブを使用して、指定されたHTMLチェックボックス入力が「チェック済み」であるかどうかを簡単に示すことができます。このディレクティブは、提供された条件が`true`と評価された場合に`checked`を出力します：

```blade
<input type="checkbox"
        name="active"
        value="active"
        @checked(old('active', $user->active)) />
```

同様に、`@selected`ディレクティブを使用して、指定された選択オプションが「選択済み」であるかどうかを示すことができます：

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

さらに、`@disabled`ディレクティブを使用して、指定された要素が「無効」であるかどうかを示すことができます：

```blade
<button type="submit" @disabled($errors->isNotEmpty())>送信</button>
```

さらに、`@readonly`ディレクティブを使用して、指定された要素が「読み取り専用」であるかどうかを示すことができます：

```blade
<input type="email"
        name="email"
        value="email@laravel.com"
        @readonly($user->isNotAdmin()) />
```

さらに、`@required`ディレクティブを使用して、指定された要素が「必須」であるかどうかを示すことができます：

```blade
<input type="text"
        name="title"
        value="title"
        @required($user->isAdmin()) />
```

<a name="including-subviews"></a>
### サブビューのインクルード

> NOTE:  
> `@include`ディレクティブを自由に使用できますが、Bladeの[コンポーネント](#components)は同様の機能を提供し、データと属性のバインディングなど、`@include`ディレクティブよりもいくつかの利点があります。

Bladeの`@include`ディレクティブを使用すると、別のビュー内にBladeビューをインクルードできます。親ビューで利用可能なすべての変数は、インクルードされたビューでも利用可能になります：

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- フォームの内容 -->
    </form>
</div>
```

インクルードされたビューは、親ビューで利用可能なすべてのデータを継承しますが、インクルードされたビューで利用可能にする追加のデータの配列を渡すこともできます：

```blade
@include('view.name', ['status' => 'complete'])
```

存在しないビューを`@include`しようとすると、Laravelはエラーをスローします。存在するかどうかわからないビューをインクルードしたい場合は、`@includeIf`ディレクティブを使用する必要があります：

```blade
@includeIf('view.name', ['status' => 'complete'])
```

指定されたブール式が`true`または`false`と評価された場合にビューをインクルードしたい場合は、`@includeWhen`と`@includeUnless`ディレクティブを使用できます：

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

指定されたビューの配列から最初に存在するビューをインクルードするには、`includeFirst`ディレクティブを使用できます：

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> WARNING:  
> Bladeビューで`__DIR__`と`__FILE__`定数を使用することは避けるべきです。キャッシュされたコンパイル済みビューの場所を参照するためです。

<a name="rendering-views-for-collections"></a>
#### コレクションのビューのレンダリング

Bladeの`@each`ディレクティブを使用して、ループとインクルードを1行にまとめることができます：

```blade
@each('view.name', $jobs, 'job')
```

`@each`ディレクティブの最初の引数は、配列またはコレクションの各要素に対してレンダリングするビューです。2番目の引数は、反復処理したい配列またはコレクションで、3番目の引数は、現在の反復処理内でビューに割り当てられる変数名です。したがって、例えば、`jobs`の配列を反復処理する場合、通常はビュー内で各ジョブを`job`変数としてアクセスしたいでしょう。現在の反復処理の配列キーは、ビュー内で`key`変数として利用可能です。

```blade
@each('view.name', $jobs, 'job')
```

`@each`ディレクティブには、4番目の引数を渡すこともできます。この引数は、指定された配列が空の場合にレンダリングされるビューを決定します。

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> WARNING:  
> `@each`を介してレンダリングされるビューは、親ビューの変数を継承しません。子ビューがこれらの変数を必要とする場合は、代わりに`@foreach`と`@include`ディレクティブを使用する必要があります。

<a name="the-once-directive"></a>
### `@once`ディレクティブ

`@once`ディレクティブを使用すると、レンダリングサイクルごとに1回だけ評価されるテンプレートの一部を定義できます。これは、[stacks](#stacks)を使用して特定のJavaScriptをページのヘッダーにプッシュする場合に便利です。例えば、ループ内で特定の[コンポーネント](#components)をレンダリングしている場合、コンポーネントが最初にレンダリングされたときにのみJavaScriptをヘッダーにプッシュしたい場合があります。

```blade
@once
    @push('scripts')
        <script>
            // カスタムJavaScript...
        </script>
    @endpush
@endonce
```

`@once`ディレクティブは、`@push`や`@prepend`ディレクティブと組み合わせてよく使用されるため、`@pushOnce`と`@prependOnce`ディレクティブも利用できます。

```blade
@pushOnce('scripts')
    <script>
        // カスタムJavaScript...
    </script>
@endPushOnce
```

<a name="raw-php"></a>
### Raw PHP

場合によっては、ビューにPHPコードを埋め込むことが便利なことがあります。Bladeの`@php`ディレクティブを使用して、テンプレート内でプレーンなPHPブロックを実行できます。

```blade
@php
    $counter = 1;
@endphp
```

また、クラスをインポートするためにPHPを使用するだけであれば、`@use`ディレクティブを使用できます。

```blade
@use('App\Models\Flight')
```

`@use`ディレクティブには、インポートされたクラスにエイリアスを付けるための2番目の引数を指定できます。

```php
@use('App\Models\Flight', 'FlightModel')
```

<a name="comments"></a>
### コメント

Bladeでは、ビューにコメントを定義することもできます。ただし、HTMLコメントとは異なり、Bladeコメントはアプリケーションによって返されるHTMLには含まれません。

```blade
{{-- このコメントはレンダリングされたHTMLには含まれません --}}
```

<a name="components"></a>
## コンポーネント

コンポーネントとスロットは、セクション、レイアウト、インクルードと同様の利点を提供しますが、コンポーネントとスロットのメンタルモデルは理解しやすいかもしれません。コンポーネントを書く方法には、クラスベースのコンポーネントと匿名コンポーネントの2つがあります。

クラスベースのコンポーネントを作成するには、`make:component` Artisanコマンドを使用できます。コンポーネントの使用方法を説明するために、シンプルな`Alert`コンポーネントを作成します。`make:component`コマンドは、コンポーネントを`app/View/Components`ディレクトリに配置します。

```shell
php artisan make:component Alert
```

`make:component`コマンドは、コンポーネントのビューテンプレートも作成します。ビューは`resources/views/components`ディレクトリに配置されます。自分のアプリケーション用のコンポーネントを書く場合、コンポーネントは`app/View/Components`ディレクトリと`resources/views/components`ディレクトリ内で自動的に検出されるため、通常はコンポーネントの登録を追加する必要はありません。

サブディレクトリ内にコンポーネントを作成することもできます。

```shell
php artisan make:component Forms/Input
```

上記のコマンドは、`app/View/Components/Forms`ディレクトリに`Input`コンポーネントを作成し、ビューは`resources/views/components/forms`ディレクトリに配置されます。

クラスを持たない匿名コンポーネント（Bladeテンプレートのみを持つコンポーネント）を作成したい場合は、`make:component`コマンドを呼び出す際に`--view`フラグを使用できます。

```shell
php artisan make:component forms.input --view
```

上記のコマンドは、`resources/views/components/forms/input.blade.php`にBladeファイルを作成します。これは、`<x-forms.input />`を介してコンポーネントとしてレンダリングできます。

<a name="manually-registering-package-components"></a>
#### パッケージコンポーネントの手動登録

自分のアプリケーション用のコンポーネントを書く場合、コンポーネントは`app/View/Components`ディレクトリと`resources/views/components`ディレクトリ内で自動的に検出されます。

ただし、Bladeコンポーネントを利用するパッケージを構築している場合は、コンポーネントクラスとそのHTMLタグエイリアスを手動で登録する必要があります。通常、コンポーネントはパッケージのサービスプロバイダの`boot`メソッドで登録する必要があります。

    use Illuminate\Support\Facades\Blade;

    /**
     * パッケージのサービスをブートストラップする
     */
    public function boot(): void
    {
        Blade::component('package-alert', Alert::class);
    }

コンポーネントが登録されると、そのタグエイリアスを使用してレンダリングできます。

```blade
<x-package-alert/>
```

または、`componentNamespace`メソッドを使用して、規約に従ってコンポーネントクラスを自動ロードすることもできます。例えば、`Nightshade`パッケージには、`Package\Views\Components`名前空間内に`Calendar`と`ColorPicker`コンポーネントがあるかもしれません。

    use Illuminate\Support\Facades\Blade;

    /**
     * パッケージのサービスをブートストラップする
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

これにより、ベンダー名前空間を使用してパッケージコンポーネントを`package-name::`構文で使用できます。

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Bladeは、コンポーネント名をパスカルケースにして、このコンポーネントにリンクされたクラスを自動的に検出します。サブディレクトリも「ドット」記法でサポートされています。

<a name="rendering-components"></a>
### コンポーネントのレンダリング

コンポーネントを表示するには、Bladeテンプレートの1つでBladeコンポーネントタグを使用できます。Bladeコンポーネントタグは、文字列`x-`で始まり、コンポーネントクラスのケバブケース名が続きます。

```blade
<x-alert/>

<x-user-profile/>
```

コンポーネントクラスが`app/View/Components`ディレクトリ内でより深くネストされている場合は、`.`文字を使用してディレクトリのネストを示すことができます。例えば、コンポーネントが`app/View/Components/Inputs/Button.php`にあると仮定すると、次のようにレンダリングできます。

```blade
<x-inputs.button/>
```

コンポーネントを条件付きでレンダリングしたい場合は、コンポーネントクラスに`shouldRender`メソッドを定義できます。`shouldRender`メソッドが`false`を返す場合、コンポーネントはレンダリングされません。

    use Illuminate\Support\Str;

    /**
     * コンポーネントをレンダリングするかどうか
     */
    public function shouldRender(): bool
    {
        return Str::length($this->message) > 0;
    }

<a name="passing-data-to-components"></a>
### コンポーネントへのデータの受け渡し

HTML属性を使用してBladeコンポーネントにデータを渡すことができます。ハードコードされたプリミティブ値は、単純なHTML属性文字列を使用してコンポーネントに渡すことができます。PHP式と変数は、`:`文字をプレフィックスとして使用する属性を介してコンポーネントに渡す必要があります。

```blade
<x-alert type="error" :message="$message"/>
```

コンポーネントのすべてのデータ属性をクラスコンストラクタで定義する必要があります。コンポーネントのすべてのパブリックプロパティは、コンポーネントのビューで自動的に利用可能になります。コンポーネントの`render`メソッドからビューにデータを渡す必要はありません。

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;
    use Illuminate\View\View;

    class Alert extends Component
    {
        /**
         * コンポーネントインスタンスを作成する
         */
        public function __construct(
            public string $type,
            public string $message,
        ) {}

        /**
         * コンポーネントを表すビュー/コンテンツを取得する
         */
        public function render(): View
        {
            return view('components.alert');
        }
    }

コンポーネントがレンダリングされると、コンポーネントのパブリック変数の内容を変数名でエコーすることで表示できます。

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### ケーシング

コンポーネントコンストラクタの引数は`camelCase`で指定する必要がありますが、HTML属性で引数名を参照する場合は`kebab-case`を使用する必要があります。例えば、次のコンポーネントコンストラクタがあるとします。

    /**
     * コンポーネントインスタンスを作成する
     */
    public function __construct(
        public string $alertType,
    ) {}

`$alertType`引数は、次のようにコンポーネントに提供できます。

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### 短縮属性構文

コンポーネントに属性を渡す際に、「短縮属性」構文を使用することもできます。これは、属性名が対応する変数名と一致する場合に便利です。

```blade
{{-- 短縮属性構文... --}}
<x-profile :$userId :$name />

{{-- これは以下と同等です... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### 属性レンダリングのエスケープ

Alpine.jsなどの一部のJavaScriptフレームワークもコロンプレフィックス付きの属性を使用するため、属性がPHP式でないことをBladeに通知するために、ダブルコロン(`::`)プレフィックスを使用できます。例えば、次のコンポーネントがあるとします。

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

Bladeによって次のHTMLがレンダリングされます。

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

<a name="component-methods"></a>
#### コンポーネントメソッド

コンポーネントテンプレートで利用可能なパブリック変数に加えて、コンポーネントの任意のパブリックメソッドを実行できます。例えば、`isSelected`メソッドを持つコンポーネントがあるとします。

    /**
     * 指定されたオプションが現在選択されているオプションであるかどうかを判断する
     */
    public function isSelected(string $option): bool
    {
        return $option === $this->selected;
    }

このメソッドは、メソッド名に一致する変数を呼び出すことで、コンポーネントテンプレートから実行できます。

```blade
<option {{ $isSelected($value) ? 'selected="selected"' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### コンポーネントクラス内での属性とスロットへのアクセス

Bladeコンポーネントでは、コンポーネント名、属性、スロットにクラスのレンダリングメソッド内からアクセスすることもできます。ただし、このデータにアクセスするには、コンポーネントの `render` メソッドからクロージャを返す必要があります。

    use Closure;

    /**
     * コンポーネントを表すビュー/コンテンツを取得する
     */
    public function render(): Closure
    {
        return function () {
            return '<div {{ $attributes }}>コンポーネントのコンテンツ</div>';
        };
    }

コンポーネントの `render` メソッドから返されるクロージャは、唯一の引数として `$data` 配列を受け取ることもできます。この配列には、コンポーネントに関する情報を提供するいくつかの要素が含まれます。

    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

        return '<div {{ $attributes }}>コンポーネントのコンテンツ</div>';
    }

> WARNING:
> `$data` 配列の要素を `render` メソッドから返される Blade 文字列に直接埋め込むべきではありません。そうすると、悪意のある属性の内容を介してリモートコード実行が可能になる可能性があります。

`componentName` は、`x-` プレフィックスの後に使用される HTML タグの名前と同じです。したがって、`<x-alert />` の `componentName` は `alert` になります。`attributes` 要素には、HTML タグに存在したすべての属性が含まれます。`slot` 要素は、コンポーネントのスロットの内容を持つ `Illuminate\Support\HtmlString` インスタンスです。

クロージャは文字列を返す必要があります。返された文字列が既存のビューに対応する場合、そのビューがレンダリングされます。それ以外の場合、返された文字列はインライン Blade ビューとして評価されます。

<a name="additional-dependencies"></a>
#### 追加の依存関係

コンポーネントが Laravel の [サービスコンテナ](container.md) からの依存関係を必要とする場合、コンポーネントのデータ属性の前にそれらをリストすることができ、コンテナによって自動的に注入されます。

```php
use App\Services\AlertCreator;

/**
 * コンポーネントインスタンスを作成する
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

<a name="hiding-attributes-and-methods"></a>
#### 属性/メソッドの非表示

コンポーネントテンプレートに変数として公開されることを防ぐために、いくつかのパブリックメソッドやプロパティを非表示にしたい場合は、コンポーネントに `$except` 配列プロパティを追加することができます。

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * コンポーネントテンプレートに公開すべきでないプロパティ/メソッド
         *
         * @var array
         */
        protected $except = ['type'];

        /**
         * コンポーネントインスタンスを作成する
         */
        public function __construct(
            public string $type,
        ) {}
    }

<a name="component-attributes"></a>
### コンポーネント属性

すでにコンポーネントにデータ属性を渡す方法を見てきましたが、コンポーネントが機能するために必要なデータの一部ではない追加の HTML 属性（例えば `class`）を指定する必要がある場合があります。通常、これらの追加属性をコンポーネントテンプレートのルート要素に渡したいと考えます。例えば、`alert` コンポーネントを次のようにレンダリングしたいとします。

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

コンポーネントのコンストラクタの一部ではないすべての属性は、自動的にコンポーネントの「属性バッグ」に追加されます。この属性バッグは、`$attributes` 変数を介してコンポーネントに自動的に提供されます。この変数をエコーすることで、コンポーネント内のすべての属性をレンダリングできます。

```blade
<div {{ $attributes }}>
    <!-- コンポーネントのコンテンツ -->
</div>
```

> WARNING:  
> コンポーネントタグ内で `@env` などのディレクティブを使用することは現在サポートされていません。例えば、`<x-alert :live="@env('production')"/>` はコンパイルされません。

<a name="default-merged-attributes"></a>
#### デフォルト/マージされた属性

属性にデフォルト値を指定したり、コンポーネントの一部の属性に追加の値をマージしたりする必要がある場合があります。これを行うには、属性バッグの `merge` メソッドを使用できます。このメソッドは、コンポーネントに常に適用される一連のデフォルト CSS クラスを定義するのに特に便利です。

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

このコンポーネントが次のように使用されると仮定します。

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

コンポーネントの最終的なレンダリングされた HTML は次のようになります。

```blade
<div class="alert alert-error mb-4">
    <!-- $message 変数の内容 -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### 条件付きでクラスをマージする

特定の条件が `true` の場合にクラスをマージしたい場合があります。これは `class` メソッドを介して行うことができます。このメソッドは、クラスを追加したいクラスまたはクラスの配列を含む配列を受け取り、値はブール式です。配列要素が数値キーを持つ場合、レンダリングされるクラスリストに常に含まれます。

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

コンポーネントに他の属性をマージする必要がある場合は、`class` メソッドに `merge` メソッドをチェーンすることができます。

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> NOTE:  
> マージされた属性を受け取らない他の HTML 要素で条件付きでクラスをコンパイルする必要がある場合は、[`@class` ディレクティブ](#conditional-classes)を使用できます。

<a name="non-class-attribute-merging"></a>
#### 非クラス属性のマージ

`class` 属性以外の属性をマージする場合、`merge` メソッドに提供される値は属性の「デフォルト」値と見なされます。ただし、`class` 属性とは異なり、これらの属性は注入された属性値とマージされません。代わりに、上書きされます。例えば、`button` コンポーネントの実装は次のようになります。

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

ボタンコンポーネントをカスタム `type` でレンダリングするには、コンポーネントを使用するときに指定できます。タイプが指定されていない場合は、`button` タイプが使用されます。

```blade
<x-button type="submit">
    Submit
</x-button>
```

この例のボタンコンポーネントのレンダリングされた HTML は次のようになります。

```blade
<button type="submit">
    Submit
</button>
```

`class` 以外の属性にデフォルト値と注入された値を結合する場合は、`prepends` メソッドを使用できます。この例では、`data-controller` 属性は常に `profile-controller` で始まり、追加の注入された `data-controller` 値はこのデフォルト値の後に配置されます。

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### 属性の取得とフィルタリング

`filter` メソッドを使用して属性をフィルタリングできます。このメソッドは、属性を保持したい場合に `true` を返すクロージャを受け取ります。

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

便宜上、`whereStartsWith` メソッドを使用して、キーが指定された文字列で始まるすべての属性を取得できます。

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

逆に、`whereDoesntStartWith` メソッドを使用して、キーが指定された文字列で始まるすべての属性を除外できます。

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

`first` メソッドを使用して、指定された属性バッグ内の最初の属性をレンダリングできます。

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

コンポーネントに属性が存在するかどうかを確認する場合は、`has` メソッドを使用できます。このメソッドは、属性名を唯一の引数として受け取り、属性が存在するかどうかを示すブール値を返します。

```blade
@if ($attributes->has('class'))
    <div>Class 属性が存在します</div>
@endif
```

`has` メソッドに配列が渡された場合、メソッドは指定されたすべての属性がコンポーネントに存在するかどうかを判断します。

```blade
@if ($attributes->has(['name', 'class']))
    <div>すべての属性が存在します</div>
@endif
```

`hasAny` メソッドを使用して、指定された属性のいずれかがコンポーネントに存在するかどうかを判断できます。

```blade
@if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>属性のいずれかが存在します</div>
@endif
```

`get` メソッドを使用して、特定の属性の値を取得できます。

```blade
{{ $attributes->get('class') }}
```

<a name="reserved-keywords"></a>
### 予約語

デフォルトでは、いくつかのキーワードが Blade の内部使用のために予約されています。これらのキーワードは、コンポーネント内でパブリックプロパティまたはメソッド名として定義することはできません。

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### スロット

多くの場合、"スロット" を介してコンポーネントに追加のコンテンツを渡す必要があります。コンポーネントスロットは、`$slot` 変数をエコーすることでレンダリングされます。この概念を探るために、`alert` コンポーネントが次のマークアップを持つと想像してみましょう。

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

コンポーネントにコンテンツを渡すために、コンポーネントにコンテンツを注入することができます。

```blade
<x-alert>
    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

場合によっては、コンポーネント内の異なる場所に複数の異なるスロットをレンダリングする必要があるかもしれません。アラートコンポーネントを修正して、"title" スロットの注入を許可してみましょう。

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

`x-slot` タグを使用して、名前付きスロットのコンテンツを定義できます。明示的な `x-slot` タグ内にないコンテンツは、`$slot` 変数にコンポーネントに渡されます。

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

スロットの `isEmpty` メソッドを呼び出して、スロットにコンテンツが含まれているかどうかを判断できます。

```blade
<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    @if ($slot->isEmpty())
        This is default content if the slot is empty.
    @else
        {{ $slot }}
    @endif
</div>
```

さらに、`hasActualContent` メソッドを使用して、スロットに HTML コメントではない "実際の" コンテンツが含まれているかどうかを判断できます。

```blade
@if ($slot->hasActualContent())
    The scope has non-comment content.
@endif
```

<a name="scoped-slots"></a>
#### スコープ付きスロット

Vue などの JavaScript フレームワークを使用したことがある場合、スロット内からコンポーネントのデータやメソッドにアクセスできる "スコープ付きスロット" に慣れているかもしれません。Laravel では、コンポーネントにパブリックメソッドやプロパティを定義し、スロット内から `$component` 変数を介してコンポーネントにアクセスすることで、同様の動作を実現できます。この例では、`x-alert` コンポーネントに `formatAlert` メソッドがコンポーネントクラスに定義されていると仮定します。

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="slot-attributes"></a>
#### スロット属性

Blade コンポーネントと同様に、CSS クラス名などの追加の [属性](#component-attributes) をスロットに割り当てることができます。

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

スロット属性とやり取りするには、スロットの変数の `attributes` プロパティにアクセスできます。属性とのやり取りの詳細については、[コンポーネント属性](#component-attributes) のドキュメントを参照してください。

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### インラインコンポーネントビュー

非常に小さなコンポーネントの場合、コンポーネントクラスとコンポーネントのビューテンプレートの両方を管理するのは面倒かもしれません。そのため、`render` メソッドから直接コンポーネントのマークアップを返すことができます。

```php
/**
 * Get the view / contents that represent the component.
 */
public function render(): string
{
    return <<<'blade'
        <div class="alert alert-danger">
            {{ $slot }}
        </div>
    blade;
}
```

<a name="generating-inline-view-components"></a>
#### インラインビューコンポーネントの生成

インラインビューをレンダリングするコンポーネントを作成するには、`make:component` コマンドを実行する際に `inline` オプションを使用できます。

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### 動的コンポーネント

コンポーネントをレンダリングする必要があるが、どのコンポーネントをレンダリングするかが実行時までわからない場合があります。このような状況では、Laravel の組み込みの `dynamic-component` コンポーネントを使用して、実行時の値または変数に基づいてコンポーネントをレンダリングできます。

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### コンポーネントの手動登録

> WARNING:  
> 以下のコンポーネントの手動登録に関するドキュメントは、ビューコンポーネントを含む Laravel パッケージを作成している場合に主に適用されます。パッケージを作成していない場合、この部分のコンポーネントドキュメントはあまり関係がないかもしれません。

独自のアプリケーション用のコンポーネントを作成する場合、コンポーネントは `app/View/Components` ディレクトリと `resources/views/components` ディレクトリ内で自動的に検出されます。

ただし、Blade コンポーネントを利用するパッケージを作成している場合や、コンポーネントを非標準的なディレクトリに配置している場合、コンポーネントクラスとその HTML タグエイリアスを手動で登録して、Laravel がコンポーネントを見つけられるようにする必要があります。通常、コンポーネントはパッケージのサービスプロバイダの `boot` メソッドで登録します。

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

コンポーネントが登録されると、そのタグエイリアスを使用してレンダリングできます。

```blade
<x-package-alert/>
```

#### パッケージコンポーネントの自動読み込み

代わりに、`componentNamespace` メソッドを使用して、規約に基づいてコンポーネントクラスを自動読み込みすることもできます。たとえば、`Nightshade` パッケージには `Package\Views\Components` 名前空間内に `Calendar` と `ColorPicker` コンポーネントがあるとします。

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

これにより、`package-name::` 構文を使用してベンダー名前空間でパッケージコンポーネントを使用できます。

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade は、コンポーネント名をパスカルケースにして、このコンポーネントにリンクされたクラスを自動的に検出します。サブディレクトリも "ドット" 表記でサポートされています。

<a name="anonymous-components"></a>
## 匿名コンポーネント

インラインコンポーネントと同様に、匿名コンポーネントは単一のファイルを介してコンポーネントを管理するメカニズムを提供します。ただし、匿名コンポーネントは単一のビューファイルを使用し、関連するクラスはありません。匿名コンポーネントを定義するには、Blade テンプレートを `resources/views/components` ディレクトリに配置するだけです。たとえば、`resources/views/components/alert.blade.php` にコンポーネントを定義した場合、次のようにレンダリングできます。

```blade
<x-alert/>
```

コンポーネントが `components` ディレクトリ内のより深い階層にネストされている場合、`.` 文字を使用してコンポーネントを示すことができます。たとえば、コンポーネントが `resources/views/components/inputs/button.blade.php` に定義されている場合、次のようにレンダリングできます。

```blade
<x-inputs.button/>
```

<a name="anonymous-index-components"></a>
### 匿名インデックスコンポーネント

コンポーネントが多くの Blade テンプレートで構成されている場合、特定のコンポーネントのテンプレートを単一のディレクトリ内にグループ化したい場合があります。たとえば、"accordion" コンポーネントが次のディレクトリ構造を持っているとします。

```none
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

このディレクトリ構造により、次のようにアコーディオンコンポーネントとそのアイテムをレンダリングできます。

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

ただし、`x-accordion` を介してアコーディオンコンポーネントをレンダリングするために、"index" アコーディオンコンポーネントテンプレートを `resources/views/components` ディレクトリに配置する必要がありました。これは、他のアコーディオン関連のテンプレートと一緒に `accordion` ディレクトリ内にネストされているのではなく、配置する必要がありました。

幸いなことに、Blade ではコンポーネントのテンプレートディレクトリ内に `index.blade.php` ファイルを配置できます。コンポーネントの `index.blade.php` テンプレートが存在する場合、それはコンポーネントの "ルート" ノードとしてレンダリングされます。したがって、上記の例で与えられた同じ Blade 構文を使用し続けることができますが、ディレクトリ構造を次のように調整します。

```none
/resources/views/components/accordion/index.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### データプロパティ / 属性

匿名コンポーネントには関連するクラスがないため、どのデータを変数としてコンポーネントに渡すべきか、どの属性をコンポーネントの [属性バッグ](#component-attributes) に配置するべきかを区別する方法が気になるかもしれません。

コンポーネントの Blade テンプレートの先頭で `@props` ディレクティブを使用して、データ変数として扱うべき属性を指定できます。コンポーネント上の他のすべての属性は、コンポーネントの属性バッグを介して利用できます。データ変数にデフォルト値を指定したい場合は、変数名を配列キーとして、デフォルト値を配列値として指定できます。

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

上記のコンポーネント定義に基づいて、次のようにコンポーネントをレンダリングできます。

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### 親データへのアクセス

子コンポーネント内から親コンポーネントのデータにアクセスしたい場合があります。このような場合、`@aware` ディレクティブを使用できます。たとえば、親 `<x-menu>` と子 `<x-menu.item>` で構成される複雑なメニューコンポーネントを構築しているとします。

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

`<x-menu>` コンポーネントは次のようになります。

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

`<x-menu.item>` コンポーネント内から親の `color` 属性にアクセスするには、コンポーネントの Blade テンプレート内で `@aware` ディレクティブを使用します。

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

`<x-menu>` コンポーネントは、以下のような実装を持つかもしれません：

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

`color` プロパティは親 (`<x-menu>`) にのみ渡されるため、`<x-menu.item>` 内では利用できません。しかし、`@aware` ディレクティブを使用することで、`<x-menu.item>` 内でも利用できるようになります：

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> WARNING:  
> `@aware` ディレクティブは、HTML属性を介して親コンポーネントに明示的に渡されていない親データにはアクセスできません。親コンポーネントに明示的に渡されていないデフォルトの `@props` 値には、`@aware` ディレクティブからアクセスできません。

<a name="anonymous-component-paths"></a>
### 匿名コンポーネントのパス

前述のように、匿名コンポーネントは通常、`resources/views/components` ディレクトリ内にBladeテンプレートを配置することで定義されます。しかし、デフォルトのパスに加えて、Laravelに他の匿名コンポーネントのパスを登録したい場合があります。

`anonymousComponentPath` メソッドは、匿名コンポーネントの場所の「パス」を最初の引数として受け取り、コンポーネントが配置されるオプションの「名前空間」を2番目の引数として受け取ります。通常、このメソッドは、アプリケーションの[サービスプロバイダ](providers.md)の `boot` メソッドから呼び出す必要があります：

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::anonymousComponentPath(__DIR__.'/../components');
    }

上記の例のように、コンポーネントパスが指定されたプレフィックスなしで登録されている場合、Bladeコンポーネントで対応するプレフィックスなしでレンダリングできます。例えば、上記のパスに `panel.blade.php` コンポーネントが存在する場合、以下のようにレンダリングできます：

```blade
<x-panel />
```

`anonymousComponentPath` メソッドの2番目の引数としてプレフィックス「名前空間」を指定できます：

    Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');

プレフィックスが提供されると、その「名前空間」内のコンポーネントは、コンポーネントをレンダリングする際にコンポーネント名に名前空間をプレフィックスとして付けることでレンダリングできます：

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## レイアウトの構築

<a name="layouts-using-components"></a>
### コンポーネントを使用したレイアウト

ほとんどのWebアプリケーションは、さまざまなページで同じ一般的なレイアウトを維持します。すべてのビューでレイアウト全体のHTMLを繰り返す必要がある場合、アプリケーションのメンテナンスは非常に面倒で困難になります。幸いなことに、このレイアウトを単一の [Bladeコンポーネント](#components) として定義し、アプリケーション全体で使用することが便利です。

<a name="defining-the-layout-component"></a>
#### レイアウトコンポーネントの定義

例えば、「todo」リストアプリケーションを構築していると想像してください。以下のような `layout` コンポーネントを定義するかもしれません：

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### レイアウトコンポーネントの適用

`layout` コンポーネントが定義されたら、そのコンポーネントを利用するBladeビューを作成できます。この例では、タスクリストを表示するシンプルなビューを定義します：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

コンポーネントに注入されたコンテンツは、`layout` コンポーネント内のデフォルトの `$slot` 変数に提供されます。お気づきのように、`layout` は `$title` スロットが提供された場合にも対応します。そうでない場合は、デフォルトのタイトルが表示されます。タスクリストビューからカスタムタイトルを注入するには、[コンポーネントドキュメント](#components)で説明されている標準のスロット構文を使用します：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

レイアウトとタスクリストビューを定義したら、ルートから `task` ビューを返すだけです：

    use App\Models\Task;

    Route::get('/tasks', function () {
        return view('tasks', ['tasks' => Task::all()]);
    });

<a name="layouts-using-template-inheritance"></a>
### テンプレート継承を使用したレイアウト

<a name="defining-a-layout"></a>
#### レイアウトの定義

レイアウトは、「テンプレート継承」を介しても作成できます。これは、[コンポーネント](#components)が導入される前にアプリケーションを構築するための主要な方法でした。

まず、簡単な例を見てみましょう。最初に、ページレイアウトを見てみましょう。ほとんどのWebアプリケーションは、さまざまなページで同じ一般的なレイアウトを維持するため、このレイアウトを単一のBladeビューとして定義すると便利です：

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

このファイルには、典型的なHTMLマークアップが含まれています。ただし、`@section` および `@yield` ディレクティブに注意してください。`@section` ディレクティブは、名前が示すように、コンテンツのセクションを定義し、`@yield` ディレクティブは、指定されたセクションの内容を表示するために使用されます。

これで、アプリケーションのレイアウトを定義したので、子ビューを定義しましょう。

<a name="extending-a-layout"></a>
#### レイアウトの拡張

子ビューを定義する際に、`@extends` Bladeディレクティブを使用して、子ビューがどのレイアウトを「継承」するかを指定します。レイアウトを継承するビューは、`@section` ディレクティブを使用してレイアウトのセクションにコンテンツを注入できます。前述の例で見たように、これらのセクションの内容は `@yield` を使用してレイアウトに表示されます：

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

この例では、`sidebar` セクションは `@@parent` ディレクティブを使用して、レイアウトのサイドバーにコンテンツを追加（上書きではなく）しています。`@@parent` ディレクティブは、ビューがレンダリングされるときにレイアウトのコンテンツに置き換えられます。

> NOTE:  
> 前述の例とは異なり、この `sidebar` セクションは `@endsection` で終わります。`@show` ではなく。`@endsection` ディレクティブはセクションを定義するだけですが、`@show` はセクションを定義し、**すぐにセクションを表示**します。

`@yield` ディレクティブは、2番目のパラメータとしてデフォルト値も受け取ります。この値は、生成されるセクションが未定義の場合にレンダリングされます：

```blade
@yield('content', 'Default content')
```

<a name="forms"></a>
## フォーム

<a name="csrf-field"></a>
### CSRFフィールド

アプリケーションでHTMLフォームを定義する際には、[CSRF保護](csrf.md)ミドルウェアがリクエストを検証できるように、フォームに隠しCSRFトークンフィールドを含める必要があります。`@csrf` Bladeディレクティブを使用して、トークンフィールドを生成できます：

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### メソッドフィールド

HTMLフォームは `PUT`、`PATCH`、または `DELETE` リクエストを作成できないため、これらのHTTP動詞を偽装するために隠し `_method` フィールドを追加する必要があります。`@method` Bladeディレクティブは、このフィールドを作成できます：

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### バリデーションエラー

`@error` ディレクティブは、指定された属性に対して [バリデーションエラーメッセージ](validation.md#quick-displaying-the-validation-errors) が存在するかどうかをすばやく確認するために使用できます。`@error` ディレクティブ内では、エラーメッセージを表示するために `$message` 変数を出力できます：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

`@error` ディレクティブは "if" 文にコンパイルされるため、属性にエラーがない場合にコンテンツをレンダリングするために `@else` ディレクティブを使用できます：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror">
```

[特定のエラーバッグの名前](validation.md#named-error-bags)を `@error` ディレクティブの2番目のパラメータとして渡すことで、複数のフォームを含むページでバリデーションエラーメッセージを取得できます：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## スタック

Bladeでは、他のビューやレイアウトの別の場所でレンダリングできる名前付きスタックにプッシュできます。これは、子ビューに必要なJavaScriptライブラリを指定するのに特に便利です：

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

指定されたブール式が `true` と評価された場合にコンテンツを `@push` したい場合、`@pushIf` ディレクティブを使用できます：

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

必要なだけ何度でもスタックにプッシュできます。スタックの完全な内容をレンダリングするには、スタックの名前を `@stack` ディレクティブに渡します：

```blade
<head>
    <!-- ヘッドの内容 -->

    @stack('scripts')
</head>
```

スタックの先頭にコンテンツを追加したい場合は、`@prepend` ディレクティブを使用する必要があります：

```blade
@push('scripts')
    これは2番目になります...
@endpush

// 後で...

@prepend('scripts')
    これは最初になります...
@endprepend
```

<a name="service-injection"></a>
## サービスの注入

`@inject` ディレクティブを使用して、Laravelの[サービスコンテナ](container.md)からサービスを取得できます。`@inject` に渡される最初の引数は、サービスが配置される変数の名前で、2番目の引数は解決したいサービスのクラスまたはインターフェース名です：

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    月間収益: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## インライン Blade テンプレートのレンダリング

時々、生の Blade テンプレート文字列を有効な HTML に変換する必要があるかもしれません。これは、`Blade` ファサードによって提供される `render` メソッドを使用して行うことができます。`render` メソッドは、Blade テンプレート文字列と、テンプレートに提供するオプションのデータ配列を受け取ります：

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel はインライン Blade テンプレートを `storage/framework/views` ディレクトリに書き込んでレンダリングします。Blade テンプレートをレンダリングした後にこれらの一時ファイルを削除したい場合は、メソッドに `deleteCachedView` 引数を指定できます：

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## Blade フラグメントのレンダリング

[Turbo](https://turbo.hotwired.dev/) や [htmx](https://htmx.org/) などのフロントエンドフレームワークを使用する場合、HTTP レスポンス内で Blade テンプレートの一部のみを返す必要があることがあります。Blade "フラグメント" を使用すると、それが可能になります。まず、Blade テンプレートの一部を `@fragment` と `@endfragment` ディレクティブ内に配置します：

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

そして、このテンプレートを使用するビューをレンダリングする際に、`fragment` メソッドを呼び出して、指定されたフラグメントのみが送信される HTTP レスポンスに含まれるように指定できます：

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

`fragmentIf` メソッドを使用すると、指定された条件に基づいてビューのフラグメントを条件付きで返すことができます。それ以外の場合は、ビュー全体が返されます：

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

`fragments` および `fragmentsIf` メソッドを使用すると、複数のビューフラグメントをレスポンスで返すことができます。フラグメントは連結されます：

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## Blade の拡張

Blade では、`directive` メソッドを使用して独自のカスタムディレクティブを定義できます。Blade コンパイラがカスタムディレクティブを見つけると、ディレクティブに含まれる式を使用して提供されたコールバックを呼び出します。

次の例では、`DateTime` のインスタンスである `$var` をフォーマットする `@datetime($var)` ディレクティブを作成します：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスの登録。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスのブートストラップ。
         */
        public function boot(): void
        {
            Blade::directive('datetime', function (string $expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

見ての通り、ディレクティブに渡された式に `format` メソッドを連鎖させます。したがって、この例では、このディレクティブによって生成される最終的な PHP は次のようになります：

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> WARNING:  
> Blade ディレクティブのロジックを更新した後、キャッシュされたすべての Blade ビューを削除する必要があります。キャッシュされた Blade ビューは、`view:clear` Artisan コマンドを使用して削除できます。

<a name="custom-echo-handlers"></a>
### カスタムエコーハンドラ

Blade を使用してオブジェクトを "echo" しようとすると、オブジェクトの `__toString` メソッドが呼び出されます。[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) メソッドは、PHP の組み込みの "マジックメソッド" の1つです。ただし、場合によっては、特定のクラスの `__toString` メソッドを制御できないことがあります。たとえば、対話しているクラスがサードパーティのライブラリに属している場合などです。

このような場合、Blade ではその特定のタイプのオブジェクトのカスタムエコーハンドラを登録できます。これを行うには、Blade の `stringable` メソッドを呼び出します。`stringable` メソッドはクロージャを受け取ります。このクロージャは、レンダリングを担当するオブジェクトのタイプをタイプヒントする必要があります。通常、`stringable` メソッドは、アプリケーションの `AppServiceProvider` クラスの `boot` メソッド内で呼び出されるべきです：

    use Illuminate\Support\Facades\Blade;
    use Money\Money;

    /**
     * 任意のアプリケーションサービスのブートストラップ。
     */
    public function boot(): void
    {
        Blade::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }

カスタムエコーハンドラが定義されると、Blade テンプレート内でオブジェクトを単純に echo できます：

```blade
コスト: {{ $money }}
```

<a name="custom-if-statements"></a>
### カスタム If ステートメント

カスタムディレクティブをプログラミングすることは、単純なカスタム条件ステートメントを定義する場合には必要以上に複雑なことがあります。そのため、Blade は `Blade::if` メソッドを提供しており、クロージャを使用してカスタム条件ディレクティブをすばやく定義できます。たとえば、アプリケーションに設定されたデフォルトの "ディスク" をチェックするカスタム条件を定義しましょう。これは、`AppServiceProvider` の `boot` メソッドで行うことができます：

    use Illuminate\Support\Facades\Blade;

    /**
     * 任意のアプリケーションサービスのブートストラップ。
     */
    public function boot(): void
    {
        Blade::if('disk', function (string $value) {
            return config('filesystems.default') === $value;
        });
    }

カスタム条件が定義されると、テンプレート内でそれを使用できます：

```blade
@disk('local')
    <!-- アプリケーションはローカルディスクを使用しています... -->
@elsedisk('s3')
    <!-- アプリケーションは s3 ディスクを使用しています... -->
@else
    <!-- アプリケーションは他のディスクを使用しています... -->
@enddisk

@unlessdisk('local')
    <!-- アプリケーションはローカルディスクを使用していません... -->
@enddisk
```
