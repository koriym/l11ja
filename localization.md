# ローカリゼーション

- [はじめに](#introduction)
    - [言語ファイルの公開](#publishing-the-language-files)
    - [ロケールの設定](#configuring-the-locale)
    - [複数形化言語](#pluralization-language)
- [翻訳文字列の定義](#defining-translation-strings)
    - [短いキーの使用](#using-short-keys)
    - [翻訳文字列をキーとして使用](#using-translation-strings-as-keys)
- [翻訳文字列の取得](#retrieving-translation-strings)
    - [翻訳文字列内のパラメータの置換](#replacing-parameters-in-translation-strings)
    - [複数形化](#pluralization)
- [パッケージ言語ファイルのオーバーライド](#overriding-package-language-files)

<a name="introduction"></a>
## はじめに

> NOTE:  
> デフォルトでは、Laravelアプリケーションのスケルトンには`lang`ディレクトリは含まれていません。Laravelの言語ファイルをカスタマイズしたい場合や、独自の言語ファイルを作成したい場合は、`lang:publish` Artisanコマンドを使用して`lang`ディレクトリをスキャフォールドする必要があります。`lang:publish`コマンドは、アプリケーション内に`lang`ディレクトリを作成し、Laravelが使用するデフォルトの言語ファイルセットを公開します。

Laravelのローカリゼーション機能は、さまざまな言語の文字列を取得するための便利な方法を提供し、アプリケーション内で複数の言語を簡単にサポートできるようにします。

Laravelは、翻訳文字列を管理するための2つの方法を提供します。まず、言語文字列は、アプリケーションの`lang`ディレクトリ内のファイルに保存される場合があります。このディレクトリ内には、アプリケーションがサポートする各言語のサブディレクトリが存在する場合があります。これは、Laravelがバリデーションエラーメッセージなどの組み込みLaravel機能の翻訳文字列を管理するために使用するアプローチです。

    /lang
        /en
            messages.php
        /es
            messages.php

または、翻訳文字列は、`lang`ディレクトリ内に配置されたJSONファイル内で定義される場合があります。このアプローチを取る場合、アプリケーションがサポートする各言語に対応するJSONファイルがこのディレクトリ内に存在します。このアプローチは、多数の翻訳可能な文字列を持つアプリケーションに推奨されます。

    /lang
        en.json
        es.json

このドキュメント内で、翻訳文字列を管理する各アプローチについて説明します。

<a name="publishing-the-language-files"></a>
### 言語ファイルの公開

デフォルトでは、Laravelアプリケーションのスケルトンには`lang`ディレクトリは含まれていません。Laravelの言語ファイルをカスタマイズしたい場合や、独自の言語ファイルを作成したい場合は、`lang:publish` Artisanコマンドを使用して`lang`ディレクトリをスキャフォールドする必要があります。`lang:publish`コマンドは、アプリケーション内に`lang`ディレクトリを作成し、Laravelが使用するデフォルトの言語ファイルセットを公開します。

```shell
php artisan lang:publish
```

<a name="configuring-the-locale"></a>
### ロケールの設定

アプリケーションのデフォルト言語は、`config/app.php`設定ファイルの`locale`設定オプションに保存されます。これは通常、`APP_LOCALE`環境変数を使用して設定されます。アプリケーションのニーズに合わせてこの値を自由に変更できます。

また、デフォルト言語に指定された翻訳文字列が存在しない場合に使用される「フォールバック言語」を設定することもできます。デフォルト言語と同様に、フォールバック言語も`config/app.php`設定ファイルで設定され、その値は通常、`APP_FALLBACK_LOCALE`環境変数を使用して設定されます。

`App`ファサードが提供する`setLocale`メソッドを使用して、単一のHTTPリクエストのデフォルト言語を実行時に変更することもできます。

    use Illuminate\Support\Facades\App;

    Route::get('/greeting/{locale}', function (string $locale) {
        if (! in_array($locale, ['en', 'es', 'fr'])) {
            abort(400);
        }

        App::setLocale($locale);

        // ...
    });

<a name="determining-the-current-locale"></a>
#### 現在のロケールの決定

`App`ファサードの`currentLocale`メソッドと`isLocale`メソッドを使用して、現在のロケールを決定したり、ロケールが指定された値であるかどうかを確認したりできます。

    use Illuminate\Support\Facades\App;

    $locale = App::currentLocale();

    if (App::isLocale('en')) {
        // ...
    }

<a name="pluralization-language"></a>
### 複数形化言語

Laravelの「複数形化器」（Eloquentやフレームワークの他の部分で単数形の文字列を複数形に変換するために使用される）に、英語以外の言語を使用するように指示できます。これは、アプリケーションのサービスプロバイダの`boot`メソッド内で`useLanguage`メソッドを呼び出すことで実現できます。複数形化器が現在サポートしている言語は、`french`、`norwegian-bokmal`、`portuguese`、`spanish`、および`turkish`です。

    use Illuminate\Support\Pluralizer;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Pluralizer::useLanguage('spanish');

        // ...
    }

> WARNING:  
> 複数形化器の言語をカスタマイズする場合、Eloquentモデルの[テーブル名](eloquent.md#table-names)を明示的に定義する必要があります。

<a name="defining-translation-strings"></a>
## 翻訳文字列の定義

<a name="using-short-keys"></a>
### 短いキーの使用

通常、翻訳文字列は`lang`ディレクトリ内のファイルに保存されます。このディレクトリ内には、アプリケーションがサポートする各言語のサブディレクトリが存在する必要があります。これは、Laravelがバリデーションエラーメッセージなどの組み込みLaravel機能の翻訳文字列を管理するために使用するアプローチです。

    /lang
        /en
            messages.php
        /es
            messages.php

すべての言語ファイルは、キー付きの文字列の配列を返します。例えば：

    <?php

    // lang/en/messages.php

    return [
        'welcome' => 'Welcome to our application!',
    ];

> WARNING:  
> 地域によって異なる言語の場合、言語ディレクトリの名前はISO 15897に従って命名する必要があります。例えば、英国英語の場合は"en_GB"を使用し、"en-gb"ではなく"en_GB"を使用する必要があります。

<a name="using-translation-strings-as-keys"></a>
### 翻訳文字列をキーとして使用

多数の翻訳可能な文字列を持つアプリケーションの場合、ビュー内でキーを参照する際に「短いキー」を使用すると混乱しやすくなり、アプリケーションがサポートするすべての翻訳文字列に対してキーを継続的に考え出すのは面倒です。

このため、Laravelは文字列の「デフォルト」翻訳をキーとして使用して翻訳文字列を定義することもサポートしています。翻訳文字列をキーとして使用する言語ファイルは、`lang`ディレクトリ内のJSONファイルとして保存されます。例えば、アプリケーションにスペイン語の翻訳がある場合、`lang/es.json`ファイルを作成する必要があります。

```json
{
    "I love programming.": "Me encanta programar."
}
```

#### キー / ファイルの競合

他の翻訳ファイル名と競合する翻訳文字列キーを定義しないでください。例えば、"NL"ロケールの`__('Action')`を翻訳し、`nl/action.php`ファイルが存在するが`nl.json`ファイルが存在しない場合、トランスレータは`nl/action.php`の内容全体を返します。

<a name="retrieving-translation-strings"></a>
## 翻訳文字列の取得

`__`ヘルパー関数を使用して、言語ファイルから翻訳文字列を取得できます。翻訳文字列を「短いキー」を使用して定義する場合、「ドット」構文を使用してキーを含むファイルとキー自体を`__`関数に渡す必要があります。例えば、`lang/en/messages.php`言語ファイルから`welcome`翻訳文字列を取得します。

    echo __('messages.welcome');

指定された翻訳文字列が存在しない場合、`__`関数は翻訳文字列キーを返します。したがって、上記の例では、翻訳文字列が存在しない場合、`__`関数は`messages.welcome`を返します。

[デフォルトの翻訳文字列を翻訳キーとして使用する](#using-translation-strings-as-keys)場合、翻訳文字列のデフォルト翻訳を`__`関数に渡す必要があります。

    echo __('I love programming.');

ここでも、翻訳文字列が存在しない場合、`__`関数は指定された翻訳文字列キーを返します。

[Bladeテンプレートエンジン](blade.md)を使用している場合、`{{ }}`エコー構文を使用して翻訳文字列を表示できます。

    {{ __('messages.welcome') }}

<a name="replacing-parameters-in-translation-strings"></a>
### 翻訳文字列内のパラメータの置換

希望する場合、翻訳文字列内にプレースホルダを定義できます。すべてのプレースホルダは`:`で始まります。例えば、プレースホルダ名を含むウェルカムメッセージを定義できます。

    'welcome' => 'Welcome, :name',

翻訳文字列を取得する際にプレースホルダを置換するには、`__`関数の2番目の引数として置換の配列を渡します。

    echo __('messages.welcome', ['name' => 'dayle']);

プレースホルダにすべて大文字が含まれている場合、または最初の文字だけが大文字の場合、翻訳された値はそれに応じて大文字になります。

    'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
    'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

<a name="object-replacement-formatting"></a>
#### オブジェクト置換のフォーマット

翻訳プレースホルダとしてオブジェクトを提供しようとすると、オブジェクトの`__toString`メソッドが呼び出されます。PHPの組み込み「マジックメソッド」の1つである[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring)メソッドです。ただし、特定のクラスの`__toString`メソッドを制御できない場合があります。例えば、やり取りしているクラスがサードパーティライブラリに属している場合などです。

このような場合、Laravelではその特定のタイプのオブジェクトのカスタムフォーマットハンドラを登録できます。これを実現するには、トランスレータの`stringable`メソッドを呼び出す必要があります。`stringable`メソッドは、フォーマットするオブジェクトのタイプをタイプヒントするクロージャを受け取ります。通常、`stringable`メソッドはアプリケーションの`AppServiceProvider`クラスの`boot`メソッド内で呼び出す必要があります。

    use Illuminate\Support\Facades\Lang;
    use Money\Money;

    /**
     * 任意のアプリケーションサービスのブートストラップ
     */
    public function boot(): void
    {
        Lang::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }

```php
    /**
     * アプリケーションサービスのブートストラップ。
     */
    public function boot(): void
    {
        Lang::stringable(function (Money $money) {
            return $money->formatTo('en_GB');
        });
    }
```

<a name="pluralization"></a>
### 複数形化

複数形化は、言語によって複数形化のルールが異なるため、複雑な問題です。しかし、Laravelは、定義した複数形化ルールに基づいて文字列を異なる方法で翻訳するのに役立ちます。`|`文字を使用して、文字列の単数形と複数形を区別できます。

    'apples' => 'There is one apple|There are many apples',

もちろん、[翻訳文字列をキーとして使用する](#using-translation-strings-as-keys)場合も複数形化がサポートされています。

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

さらに複雑な複数形化ルールを作成して、値の複数の範囲に対する翻訳文字列を指定することもできます。

    'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

複数形化オプションを持つ翻訳文字列を定義した後、`trans_choice`関数を使用して、指定された「カウント」に対する行を取得できます。この例では、カウントが1より大きいため、翻訳文字列の複数形が返されます。

    echo trans_choice('messages.apples', 10);

複数形化文字列にプレースホルダー属性を定義することもできます。これらのプレースホルダーは、`trans_choice`関数に3番目の引数として配列を渡すことで置き換えられます。

    'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

`trans_choice`関数に渡された整数値を表示したい場合は、組み込みの`:count`プレースホルダーを使用できます。

    'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',

<a name="overriding-package-language-files"></a>
## パッケージの言語ファイルのオーバーライド

一部のパッケージには独自の言語ファイルが付属している場合があります。これらの行を微調整するためにパッケージのコアファイルを変更する代わりに、`lang/vendor/{package}/{locale}`ディレクトリにファイルを配置することでオーバーライドできます。

たとえば、`skyrim/hearthfire`というパッケージの`messages.php`にある英語の翻訳文字列をオーバーライドする必要がある場合、次の場所に言語ファイルを配置する必要があります：`lang/vendor/hearthfire/en/messages.php`。このファイル内では、オーバーライドしたい翻訳文字列のみを定義する必要があります。オーバーライドしない翻訳文字列は、パッケージの元の言語ファイルから引き続き読み込まれます。

