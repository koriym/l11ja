# 文字列

- [はじめに](#introduction)
- [利用可能なメソッド](#available-methods)

<a name="introduction"></a>
## はじめに

Laravelには、文字列値を操作するためのさまざまな関数が含まれています。これらの関数の多くはフレームワーク自体で使用されていますが、自分のアプリケーションで便利だと思う場合は自由に使用できます。

<a name="available-methods"></a>
## 利用可能なメソッド

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<a name="strings-method-list"></a>
### 文字列

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::apa](#method-str-apa)
[Str::ascii](#method-str-ascii)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::between](#method-str-between)
[Str::betweenFirst](#method-str-between-first)
[Str::camel](#method-camel-case)
[Str::charAt](#method-char-at)
[Str::chopStart](#method-str-chop-start)
[Str::chopEnd](#method-str-chop-end)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::deduplicate](#method-deduplicate)
[Str::endsWith](#method-ends-with)
[Str::excerpt](#method-excerpt)
[Str::finish](#method-str-finish)
[Str::headline](#method-str-headline)
[Str::inlineMarkdown](#method-str-inline-markdown)
[Str::is](#method-str-is)
[Str::isAscii](#method-str-is-ascii)
[Str::isJson](#method-str-is-json)
[Str::isUlid](#method-str-is-ulid)
[Str::isUrl](#method-str-is-url)
[Str::isUuid](#method-str-is-uuid)
[Str::kebab](#method-kebab-case)
[Str::lcfirst](#method-str-lcfirst)
[Str::length](#method-str-length)
[Str::limit](#method-str-limit)
[Str::lower](#method-str-lower)
[Str::markdown](#method-str-markdown)
[Str::mask](#method-str-mask)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::padBoth](#method-str-padboth)
[Str::padLeft](#method-str-padleft)
[Str::padRight](#method-str-padright)
[Str::password](#method-str-password)
[Str::plural](#method-str-plural)
[Str::pluralStudly](#method-str-plural-studly)
[Str::position](#method-str-position)
[Str::random](#method-str-random)
[Str::remove](#method-str-remove)
[Str::repeat](#method-str-repeat)
[Str::replace](#method-str-replace)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::replaceMatches](#method-str-replace-matches)
[Str::replaceStart](#method-str-replace-start)
[Str::replaceEnd](#method-str-replace-end)
[Str::reverse](#method-str-reverse)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::squish](#method-str-squish)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::substr](#method-str-substr)
[Str::substrCount](#method-str-substrcount)
[Str::substrReplace](#method-str-substrreplace)
[Str::swap](#method-str-swap)
[Str::take](#method-take)
[Str::title](#method-title-case)
[Str::toBase64](#method-str-to-base64)
[Str::toHtmlString](#method-str-to-html-string)
[Str::transliterate](#method-str-transliterate)
[Str::trim](#method-str-trim)
[Str::ltrim](#method-str-ltrim)
[Str::rtrim](#method-str-rtrim)
[Str::ucfirst](#method-str-ucfirst)
[Str::ucsplit](#method-str-ucsplit)
[Str::upper](#method-str-upper)
[Str::ulid](#method-str-ulid)
[Str::unwrap](#method-str-unwrap)
[Str::uuid](#method-str-uuid)
[Str::wordCount](#method-str-word-count)
[Str::wordWrap](#method-str-word-wrap)
[Str::words](#method-str-words)
[Str::wrap](#method-str-wrap)
[str](#method-str)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

<a name="fluent-strings-method-list"></a>
### Fluent Strings

<div class="collection-method-list" markdown="1">

[after](#method-fluent-str-after)
[afterLast](#method-fluent-str-after-last)
[apa](#method-fluent-str-apa)
[append](#method-fluent-str-append)
[ascii](#method-fluent-str-ascii)
[basename](#method-fluent-str-basename)
[before](#method-fluent-str-before)
[beforeLast](#method-fluent-str-before-last)
[between](#method-fluent-str-between)
[betweenFirst](#method-fluent-str-between-first)
[camel](#method-fluent-str-camel)
[charAt](#method-fluent-str-char-at)
[classBasename](#method-fluent-str-class-basename)
[chopStart](#method-fluent-str-chop-start)
[chopEnd](#method-fluent-str-chop-end)
[contains](#method-fluent-str-contains)
[containsAll](#method-fluent-str-contains-all)
[deduplicate](#method-fluent-str-deduplicate)
[dirname](#method-fluent-str-dirname)
[endsWith](#method-fluent-str-ends-with)
[exactly](#method-fluent-str-exactly)
[excerpt](#method-fluent-str-excerpt)
[explode](#method-fluent-str-explode)
[finish](#method-fluent-str-finish)
[headline](#method-fluent-str-headline)
[inlineMarkdown](#method-fluent-str-inline-markdown)
[is](#method-fluent-str-is)
[isAscii](#method-fluent-str-is-ascii)
[isEmpty](#method-fluent-str-is-empty)
[isNotEmpty](#method-fluent-str-is-not-empty)
[isJson](#method-fluent-str-is-json)
[isUlid](#method-fluent-str-is-ulid)
[isUrl](#method-fluent-str-is-url)
[isUuid](#method-fluent-str-is-uuid)
[kebab](#method-fluent-str-kebab)
[lcfirst](#method-fluent-str-lcfirst)
[length](#method-fluent-str-length)
[limit](#method-fluent-str-limit)
[lower](#method-fluent-str-lower)
[markdown](#method-fluent-str-markdown)
[mask](#method-fluent-str-mask)
[match](#method-fluent-str-match)
[matchAll](#method-fluent-str-match-all)
[isMatch](#method-fluent-str-is-match)
[newLine](#method-fluent-str-new-line)
[padBoth](#method-fluent-str-padboth)
[padLeft](#method-fluent-str-padleft)
[padRight](#method-fluent-str-padright)
[pipe](#method-fluent-str-pipe)
[plural](#method-fluent-str-plural)
[position](#method-fluent-str-position)
[prepend](#method-fluent-str-prepend)
[remove](#method-fluent-str-remove)
[repeat](#method-fluent-str-repeat)
[replace](#method-fluent-str-replace)
[replaceArray](#method-fluent-str-replace-array)
[replaceFirst](#method-fluent-str-replace-first)
[replaceLast](#method-fluent-str-replace-last)
[replaceMatches](#method-fluent-str-replace-matches)
[replaceStart](#method-fluent-str-replace-start)
[replaceEnd](#method-fluent-str-replace-end)
[scan](#method-fluent-str-scan)
[singular](#method-fluent-str-singular)
[slug](#method-fluent-str-slug)
[snake](#method-fluent-str-snake)
[split](#method-fluent-str-split)
[squish](#method-fluent-str-squish)
[start](#method-fluent-str-start)
[startsWith](#method-fluent-str-starts-with)
[stripTags](#method-fluent-str-strip-tags)
[studly](#method-fluent-str-studly)
[substr](#method-fluent-str-substr)
[substrReplace](#method-fluent-str-substrreplace)
[swap](#method-fluent-str-swap)
[take](#method-fluent-str-take)
[tap](#method-fluent-str-tap)
[test](#method-fluent-str-test)
[title](#method-fluent-str-title)
[toBase64](#method-fluent-str-to-base64)
[transliterate](#method-fluent-str-transliterate)
[trim](#method-fluent-str-trim)
[ltrim](#method-fluent-str-ltrim)
[rtrim](#method-fluent-str-rtrim)
[ucfirst](#method-fluent-str-ucfirst)
[ucsplit](#method-fluent-str-ucsplit)
[unwrap](#method-fluent-str-unwrap)
[upper](#method-fluent-str-upper)
[when](#method-fluent-str-when)
[whenContains](#method-fluent-str-when-contains)
[whenContainsAll](#method-fluent-str-when-contains-all)
[whenEmpty](#method-fluent-str-when-empty)
[whenNotEmpty](#method-fluent-str-when-not-empty)
[whenStartsWith](#method-fluent-str-when-starts-with)
[whenEndsWith](#method-fluent-str-when-ends-with)
[whenExactly](#method-fluent-str-when-exactly)
[whenNotExactly](#method-fluent-str-when-not-exactly)
[whenIs](#method-fluent-str-when-is)
[whenIsAscii](#method-fluent-str-when-is-ascii)
[whenIsUlid](#method-fluent-str-when-is-ulid)
[whenIsUuid](#method-fluent-str-when-is-uuid)
[whenTest](#method-fluent-str-when-test)
[wordCount](#method-fluent-str-word-count)
[words](#method-fluent-str-words)

</div>

<a name="strings"></a>
## 文字列

<a name="method-__"></a>
#### `__()` {.collection-method}

`__`関数は、[言語ファイル](localization.md)を使用して、指定された翻訳文字列または翻訳キーを翻訳します。

    echo __('Welcome to our application');

    echo __('messages.welcome');

指定された翻訳文字列またはキーが存在しない場合、`__`関数は指定された値を返します。したがって、上記の例を使用すると、翻訳キーが存在しない場合、`__`関数は`messages.welcome`を返します。

<a name="method-class-basename"></a>
#### `class_basename()` {.collection-method}

`class_basename`関数は、指定されたクラスの名前を、そのクラスの名前空間を削除して返します。

    $class = class_basename('Foo\Bar\Baz');

    // Baz

<a name="method-e"></a>
#### `e()` {.collection-method}

`e`関数は、PHPの`htmlspecialchars`関数を、`double_encode`オプションをデフォルトで`true`に設定して実行します。

    echo e('<html>foo</html>');

    // &lt;html&gt;foo&lt;/html&gt;

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {.collection-method}

`preg_replace_array`関数は、文字列内の指定されたパターンを配列を使用して順次置換します。

    $string = 'The event will take place between :start and :end';

    $replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-after"></a>
#### `Str::after()` {.collection-method}

`Str::after`メソッドは、文字列内の指定された値の後にあるすべてを返します。文字列内に値が存在しない場合、文字列全体が返されます。

    use Illuminate\Support\Str;

    $slice = Str::after('This is my name', 'This is');

    // ' my name'

<a name="method-str-after-last"></a>
#### `Str::afterLast()` {.collection-method}

`Str::afterLast`メソッドは、文字列内で指定された値が最後に現れた後のすべてを返します。文字列内に値が存在しない場合は、文字列全体が返されます。

    use Illuminate\Support\Str;

    $slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

    // 'Controller'

<a name="method-str-apa"></a>
#### `Str::apa()` {.collection-method}

`Str::apa`メソッドは、指定された文字列を[APAガイドライン](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case)に従ってタイトルケースに変換します。

    use Illuminate\Support\Str;

    $title = Str::apa('Creating A Project');

    // 'Creating a Project'

<a name="method-str-ascii"></a>
#### `Str::ascii()` {.collection-method}

`Str::ascii`メソッドは、文字列をASCII値に変換しようとします。

    use Illuminate\Support\Str;

    $slice = Str::ascii('û');

    // 'u'

<a name="method-str-before"></a>
#### `Str::before()` {.collection-method}

`Str::before`メソッドは、文字列内で指定された値の前のすべてを返します。

    use Illuminate\Support\Str;

    $slice = Str::before('This is my name', 'my name');

    // 'This is '

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` {.collection-method}

`Str::beforeLast`メソッドは、文字列内で指定された値が最後に現れる前のすべてを返します。

    use Illuminate\Support\Str;

    $slice = Str::beforeLast('This is my name', 'is');

    // 'This '

<a name="method-str-between"></a>
#### `Str::between()` {.collection-method}

`Str::between`メソッドは、文字列内の2つの値の間の部分を返します。

    use Illuminate\Support\Str;

    $slice = Str::between('This is my name', 'This', 'name');

    // ' is my '

<a name="method-str-between-first"></a>
#### `Str::betweenFirst()` {.collection-method}

`Str::betweenFirst`メソッドは、文字列内の2つの値の間の最小の部分を返します。

    use Illuminate\Support\Str;

    $slice = Str::betweenFirst('[a] bc [d]', '[', ']');

    // 'a'

<a name="method-camel-case"></a>
#### `Str::camel()` {.collection-method}

`Str::camel`メソッドは、指定された文字列を`camelCase`に変換します。

    use Illuminate\Support\Str;

    $converted = Str::camel('foo_bar');

    // 'fooBar'

<a name="method-char-at"></a>
#### `Str::charAt()` {.collection-method}

`Str::charAt`メソッドは、指定されたインデックスの文字を返します。インデックスが範囲外の場合、`false`が返されます。

    use Illuminate\Support\Str;

    $character = Str::charAt('This is my name.', 6);

    // 's'

<a name="method-str-chop-start"></a>
#### `Str::chopStart()` {.collection-method}

`Str::chopStart`メソッドは、文字列の先頭に指定された値が現れる場合にのみ、その値を削除します。

    use Illuminate\Support\Str;

    $url = Str::chopStart('https://laravel.com', 'https://');

    // 'laravel.com'

また、2番目の引数として配列を渡すこともできます。文字列が配列内のいずれかの値で始まる場合、その値が文字列から削除されます。

    use Illuminate\Support\Str;

    $url = Str::chopStart('http://laravel.com', ['https://', 'http://']);

    // 'laravel.com'

<a name="method-str-chop-end"></a>
#### `Str::chopEnd()` {.collection-method}

`Str::chopEnd`メソッドは、文字列の末尾に指定された値が現れる場合にのみ、その値を削除します。

    use Illuminate\Support\Str;

    $url = Str::chopEnd('app/Models/Photograph.php', '.php');

    // 'app/Models/Photograph'

また、2番目の引数として配列を渡すこともできます。文字列が配列内のいずれかの値で終わる場合、その値が文字列から削除されます。

    use Illuminate\Support\Str;

    $url = Str::chopEnd('laravel.com/index.php', ['/index.html', '/index.php']);

    // 'laravel.com'

<a name="method-str-contains"></a>
#### `Str::contains()` {.collection-method}

`Str::contains`メソッドは、指定された文字列に指定された値が含まれているかどうかを判断します。デフォルトでは、このメソッドは大文字と小文字を区別します。

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'my');

    // true

また、配列を渡して、指定された文字列に配列内のいずれかの値が含まれているかどうかを判断することもできます。

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', ['my', 'foo']);

    // true

大文字と小文字を区別しないようにするには、`ignoreCase`引数を`true`に設定します。

    use Illuminate\Support\Str;

    $contains = Str::contains('This is my name', 'MY', ignoreCase: true);

    // true

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` {.collection-method}

`Str::containsAll`メソッドは、指定された文字列に指定された配列内のすべての値が含まれているかどうかを判断します。

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['my', 'name']);

    // true

大文字と小文字を区別しないようにするには、`ignoreCase`引数を`true`に設定します。

    use Illuminate\Support\Str;

    $containsAll = Str::containsAll('This is my name', ['MY', 'NAME'], ignoreCase: true);

    // true
    
<a name="method-deduplicate"></a>
#### `Str::deduplicate()` {.collection-method}

`Str::deduplicate`メソッドは、指定された文字列内の連続する文字のインスタンスをその文字の単一のインスタンスに置き換えます。デフォルトでは、このメソッドはスペースを重複排除します。

    use Illuminate\Support\Str;

    $result = Str::deduplicate('The   Laravel   Framework');

    // The Laravel Framework

重複排除する文字を指定するには、メソッドの2番目の引数として渡します。

    use Illuminate\Support\Str;

    $result = Str::deduplicate('The---Laravel---Framework', '-');

    // The-Laravel-Framework

<a name="method-ends-with"></a>
#### `Str::endsWith()` {.collection-method}

`Str::endsWith`メソッドは、指定された文字列が指定された値で終わるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', 'name');

    // true

また、配列を渡して、指定された文字列が配列内のいずれかの値で終わるかどうかを判断することもできます。

    use Illuminate\Support\Str;

    $result = Str::endsWith('This is my name', ['name', 'foo']);

    // true

    $result = Str::endsWith('This is my name', ['this', 'foo']);

    // false

<a name="method-excerpt"></a>
#### `Str::excerpt()` {.collection-method}

`Str::excerpt`メソッドは、指定された文字列内で最初に一致するフレーズに基づいて抜粋を抽出します。

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'my', [
        'radius' => 3
    ]);

    // '...is my na...'

`radius`オプション（デフォルトは`100`）を使用して、切り取られた文字列の両側に表示される文字数を定義できます。

さらに、`omission`オプションを使用して、切り取られた文字列の前後に追加される文字列を定義できます。

    use Illuminate\Support\Str;

    $excerpt = Str::excerpt('This is my name', 'name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-str-finish"></a>
#### `Str::finish()` {.collection-method}

`Str::finish`メソッドは、文字列が指定された値で終わっていない場合に、その値の単一のインスタンスを文字列に追加します。

    use Illuminate\Support\Str;

    $adjusted = Str::finish('this/string', '/');

    // this/string/

    $adjusted = Str::finish('this/string/', '/');

    // this/string/

<a name="method-str-headline"></a>
#### `Str::headline()` {.collection-method}

`Str::headline`メソッドは、大文字と小文字、ハイフン、またはアンダースコアで区切られた文字列を、各単語の最初の文字が大文字になるスペース区切りの文字列に変換します。

    use Illuminate\Support\Str;

    $headline = Str::headline('steve_jobs');

    // Steve Jobs

    $headline = Str::headline('EmailNotificationSent');

    // Email Notification Sent

<a name="method-str-inline-markdown"></a>
#### `Str::inlineMarkdown()` {.collection-method}

`Str::inlineMarkdown`メソッドは、GitHubフレーバーのMarkdownを[CommonMark](https://commonmark.thephpleague.com/)を使用してインラインHTMLに変換します。ただし、`markdown`メソッドとは異なり、生成されたすべてのHTMLをブロックレベルの要素でラップしません。

    use Illuminate\Support\Str;

    $html = Str::inlineMarkdown('**Laravel**');

    // <strong>Laravel</strong>

#### Markdownセキュリティ

デフォルトでは、Markdownは生のHTMLをサポートしており、生のユーザー入力と共に使用するとクロスサイトスクリプティング（XSS）の脆弱性が露呈します。[CommonMarkセキュリティドキュメント](https://commonmark.thephpleague.com/security/)に従い、`html_input`オプションを使用して生のHTMLをエスケープまたは削除するか、`allow_unsafe_links`オプションを使用して安全でないリンクを許可するかを指定できます。生のHTMLを一部許可する必要がある場合は、コンパイルされたMarkdownをHTML Purifierを通して渡すべきです。

    use Illuminate\Support\Str;

    Str::inlineMarkdown('Inject: <script>alert("Hello XSS!");</script>', [
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // Inject: alert(&quot;Hello XSS!&quot;);

<a name="method-str-is"></a>
#### `Str::is()` {.collection-method}

`Str::is`メソッドは、指定された文字列が指定されたパターンに一致するかどうかを判断します。アスタリスクをワイルドカードとして使用できます。

    use Illuminate\Support\Str;

    $matches = Str::is('foo*', 'foobar');

    // true

    $matches = Str::is('baz*', 'foobar');

    // false

<a name="method-str-is-ascii"></a>
#### `Str::isAscii()` {.collection-method}

`Str::isAscii`メソッドは、指定された文字列が7ビットASCIIであるかどうかを判断します。

    use Illuminate\Support\Str;

    $isAscii = Str::isAscii('Taylor');

    // true

    $isAscii = Str::isAscii('ü');

    // false

<a name="method-str-is-json"></a>
#### `Str::isJson()` {.collection-method}

`Str::isJson`メソッドは、指定された文字列が有効なJSONであるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::isJson('[1,2,3]');

// true

$result = Str::isJson('{"first": "John", "last": "Doe"}');

// true

$result = Str::isJson('{first: "John", last: "Doe"}');

// false

<a name="method-str-is-url"></a>
#### `Str::isUrl()` {.collection-method}

`Str::isUrl`メソッドは、指定された文字列が有効なURLであるかどうかを判定します。

    use Illuminate\Support\Str;

    $isUrl = Str::isUrl('http://example.com');

    // true

    $isUrl = Str::isUrl('laravel');

    // false

`isUrl`メソッドは幅広いプロトコルを有効とみなします。しかし、`isUrl`メソッドにそれらを渡すことで、有効とみなすべきプロトコルを指定できます。

    $isUrl = Str::isUrl('http://example.com', ['http', 'https']);

<a name="method-str-is-ulid"></a>
#### `Str::isUlid()` {.collection-method}

`Str::isUlid`メソッドは、指定された文字列が有効なULIDであるかどうかを判定します。

    use Illuminate\Support\Str;

    $isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

    // true

    $isUlid = Str::isUlid('laravel');

    // false

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` {.collection-method}

`Str::isUuid`メソッドは、指定された文字列が有効なUUIDであるかどうかを判定します。

    use Illuminate\Support\Str;

    $isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

    // true

    $isUuid = Str::isUuid('laravel');

    // false

<a name="method-kebab-case"></a>
#### `Str::kebab()` {.collection-method}

`Str::kebab`メソッドは、指定された文字列を`kebab-case`に変換します。

    use Illuminate\Support\Str;

    $converted = Str::kebab('fooBar');

    // foo-bar

<a name="method-str-lcfirst"></a>
#### `Str::lcfirst()` {.collection-method}

`Str::lcfirst`メソッドは、指定された文字列の最初の文字を小文字にして返します。

    use Illuminate\Support\Str;

    $string = Str::lcfirst('Foo Bar');

    // foo Bar

<a name="method-str-length"></a>
#### `Str::length()` {.collection-method}

`Str::length`メソッドは、指定された文字列の長さを返します。

    use Illuminate\Support\Str;

    $length = Str::length('Laravel');

    // 7

<a name="method-str-limit"></a>
#### `Str::limit()` {.collection-method}

`Str::limit`メソッドは、指定された文字列を指定された長さに切り詰めます。

    use Illuminate\Support\Str;

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

    // The quick brown fox...

切り詰められた文字列の末尾に追加される文字列を変更するために、メソッドに第三引数を渡すことができます。

    $truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

    // The quick brown fox (...)

文字列を切り詰める際に完全な単語を保持したい場合は、`preserveWords`引数を使用できます。この引数が`true`の場合、文字列は最も近い完全な単語の境界で切り詰められます。

    $truncated = Str::limit('The quick brown fox', 12, preserveWords: true);

    // The quick...

<a name="method-str-lower"></a>
#### `Str::lower()` {.collection-method}

`Str::lower`メソッドは、指定された文字列を小文字に変換します。

    use Illuminate\Support\Str;

    $converted = Str::lower('LARAVEL');

    // laravel

<a name="method-str-markdown"></a>
#### `Str::markdown()` {.collection-method}

`Str::markdown`メソッドは、GitHub風のMarkdownを[CommonMark](https://commonmark.thephpleague.com/)を使用してHTMLに変換します。

    use Illuminate\Support\Str;

    $html = Str::markdown('# Laravel');

    // <h1>Laravel</h1>

    $html = Str::markdown('# Taylor <b>Otwell</b>', [
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

#### Markdownのセキュリティ

デフォルトでは、Markdownは生のHTMLをサポートしており、生のユーザー入力と共に使用するとクロスサイトスクリプティング（XSS）の脆弱性が露呈します。[CommonMarkのセキュリティドキュメント](https://commonmark.thephpleague.com/security/)に従い、生のHTMLをエスケープまたは除去するために`html_input`オプションを使用し、安全でないリンクを許可するかどうかを指定するために`allow_unsafe_links`オプションを使用できます。生のHTMLを一部許可する必要がある場合は、コンパイルされたMarkdownをHTML Purifierを通して渡すべきです。

    use Illuminate\Support\Str;

    Str::markdown('Inject: <script>alert("Hello XSS!");</script>', [
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // <p>Inject: alert(&quot;Hello XSS!&quot;);</p>

<a name="method-str-mask"></a>
#### `Str::mask()` {.collection-method}

`Str::mask`メソッドは、文字列の一部を繰り返し文字でマスクし、メールアドレスや電話番号などの文字列のセグメントを難読化するために使用できます。

    use Illuminate\Support\Str;

    $string = Str::mask('taylor@example.com', '*', 3);

    // tay***************

必要に応じて、`mask`メソッドに負の数を第三引数として渡すことができます。これにより、メソッドは文字列の終わりから指定された距離でマスクを開始するように指示されます。

    $string = Str::mask('taylor@example.com', '*', -15, 3);

    // tay***@example.com

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {.collection-method}

`Str::orderedUuid`メソッドは、インデックス付きデータベースカラムに効率的に格納できる「タイムスタンプ優先」のUUIDを生成します。このメソッドを使用して生成された各UUIDは、以前にこのメソッドを使用して生成されたUUIDの後にソートされます。

    use Illuminate\Support\Str;

    return (string) Str::orderedUuid();

<a name="method-str-padboth"></a>
#### `Str::padBoth()` {.collection-method}

`Str::padBoth`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が所望の長さに達するまで、文字列の両側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::padBoth('James', 10, '_');

    // '__James___'

    $padded = Str::padBoth('James', 10);

    // '  James   '

<a name="method-str-padleft"></a>
#### `Str::padLeft()` {.collection-method}

`Str::padLeft`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が所望の長さに達するまで、文字列の左側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::padLeft('James', 10, '-=');

    // '-=-=-James'

    $padded = Str::padLeft('James', 10);

    // '     James'

<a name="method-str-padright"></a>
#### `Str::padRight()` {.collection-method}

`Str::padRight`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が所望の長さに達するまで、文字列の右側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::padRight('James', 10, '-');

    // 'James-----'

    $padded = Str::padRight('James', 10);

    // 'James     '

<a name="method-str-password"></a>
#### `Str::password()` {.collection-method}

`Str::password`メソッドは、指定された長さの安全でランダムなパスワードを生成するために使用できます。パスワードは、文字、数字、記号、スペースの組み合わせで構成されます。デフォルトでは、パスワードは32文字長です。

    use Illuminate\Support\Str;

    $password = Str::password();

    // 'EbJo2vE-AS:U,$%_gkrV4n,q~1xy/-_4'

    $password = Str::password(12);

    // 'qwuar>#V|i]N'

<a name="method-str-plural"></a>
#### `Str::plural()` {.collection-method}

`Str::plural`メソッドは、単数形の単語文字列を複数形に変換します。この関数は、[Laravelの複数形化機能がサポートする任意の言語](localization.md#pluralization-language)をサポートしています。

    use Illuminate\Support\Str;

    $plural = Str::plural('car');

    // cars

    $plural = Str::plural('child');

    // children

関数に第二引数として整数を渡すことで、文字列の単数形または複数形を取得できます。

    use Illuminate\Support\Str;

    $plural = Str::plural('child', 2);

    // children

    $singular = Str::plural('child', 1);

    // child

<a name="method-str-plural-studly"></a>
#### `Str::pluralStudly()` {.collection-method}

`Str::pluralStudly`メソッドは、スタッディキャップスケースでフォーマットされた単数形の単語文字列を複数形に変換します。この関数は、[Laravelの複数形化機能がサポートする任意の言語](localization.md#pluralization-language)をサポートしています。

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman');

    // VerifiedHumans

    $plural = Str::pluralStudly('UserFeedback');

    // UserFeedback

関数に第二引数として整数を渡すことで、文字列の単数形または複数形を取得できます。

    use Illuminate\Support\Str;

    $plural = Str::pluralStudly('VerifiedHuman', 2);

    // VerifiedHumans

    $singular = Str::pluralStudly('VerifiedHuman', 1);

    // VerifiedHuman

<a name="method-str-position"></a>
#### `Str::position()` {.collection-method}

`Str::position`メソッドは、文字列内で部分文字列が最初に出現する位置を返します。指定された文字列に部分文字列が存在しない場合、`false`が返されます。

    use Illuminate\Support\Str;

    $position = Str::position('Hello, World!', 'Hello');

    // 0

    $position = Str::position('Hello, World!', 'W');

    // 7

<a name="method-str-random"></a>
#### `Str::random()` {.collection-method}

`Str::random`メソッドは、指定された長さのランダムな文字列を生成します。この関数はPHPの`random_bytes`関数を使用します。

    use Illuminate\Support\Str;

    $random = Str::random(40);

テスト中に`Str::random`メソッドが返す値を「偽装」することが有用な場合があります。これを実現するには、`createRandomStringsUsing`メソッドを使用できます。

    Str::createRandomStringsUsing(function () {
        return 'fake-random-string';
    });

`random`メソッドを通常のランダムな文字列の生成に戻すには、`createRandomStringsNormally`メソッドを呼び出すことができます。

    Str::createRandomStringsNormally();

<a name="method-str-remove"></a>
#### `Str::remove()` {.collection-method}

`Str::remove`メソッドは、指定された値または値の配列を文字列から削除します。

    use Illuminate\Support\Str;

    $string = 'Peter Piper picked a peck of pickled peppers.';

    $removed = Str::remove('e', $string);

    // Ptr Pipr pickd a pck of pickld ppprs.

`remove` メソッドに `false` を第三引数として渡すことで、文字列を削除する際に大文字と小文字を区別しないようにすることもできます。

<a name="method-str-repeat"></a>
#### `Str::repeat()` {.collection-method}

`Str::repeat` メソッドは、指定された文字列を繰り返します:

```php
use Illuminate\Support\Str;

$string = 'a';

$repeat = Str::repeat($string, 5);

// aaaaa
```

<a name="method-str-replace"></a>
#### `Str::replace()` {.collection-method}

`Str::replace` メソッドは、文字列内の指定された文字列を置換します:

    use Illuminate\Support\Str;

    $string = 'Laravel 10.x';

    $replaced = Str::replace('10.x', '11.x', $string);

    // Laravel 11.x

`replace` メソッドは `caseSensitive` 引数も受け付けます。デフォルトでは、`replace` メソッドは大文字と小文字を区別します:

    Str::replace('Framework', 'Laravel', caseSensitive: false);

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {.collection-method}

`Str::replaceArray` メソッドは、配列を使用して文字列内の指定された値を順次置換します:

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

    // The event will take place between 8:30 and 9:00

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {.collection-method}

`Str::replaceFirst` メソッドは、文字列内で最初に出現する指定された値を置換します:

    use Illuminate\Support\Str;

    $replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {.collection-method}

`Str::replaceLast` メソッドは、文字列内で最後に出現する指定された値を置換します:

    use Illuminate\Support\Str;

    $replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

    // the quick brown fox jumps over a lazy dog

<a name="method-str-replace-matches"></a>
#### `Str::replaceMatches()` {.collection-method}

`Str::replaceMatches` メソッドは、パターンに一致する文字列のすべての部分を指定された置換文字列で置換します:

    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches(
        pattern: '/[^A-Za-z0-9]++/',
        replace: '',
        subject: '(+1) 501-555-1000'
    )

    // '15015551000'

`replaceMatches` メソッドは、指定されたパターンに一致する文字列の各部分で呼び出されるクロージャも受け付けます。クロージャ内で置換ロジックを実行し、置換された値を返すことができます:

    use Illuminate\Support\Str;

    $replaced = Str::replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    }, '123');

    // '[1][2][3]'

<a name="method-str-replace-start"></a>
#### `Str::replaceStart()` {.collection-method}

`Str::replaceStart` メソッドは、指定された値が文字列の先頭にある場合にのみ、最初に出現する指定された値を置換します:

    use Illuminate\Support\Str;

    $replaced = Str::replaceStart('Hello', 'Laravel', 'Hello World');

    // Laravel World

    $replaced = Str::replaceStart('World', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-replace-end"></a>
#### `Str::replaceEnd()` {.collection-method}

`Str::replaceEnd` メソッドは、指定された値が文字列の末尾にある場合にのみ、最後に出現する指定された値を置換します:

    use Illuminate\Support\Str;

    $replaced = Str::replaceEnd('World', 'Laravel', 'Hello World');

    // Hello Laravel

    $replaced = Str::replaceEnd('Hello', 'Laravel', 'Hello World');

    // Hello World

<a name="method-str-reverse"></a>
#### `Str::reverse()` {.collection-method}

`Str::reverse` メソッドは、指定された文字列を反転します:

    use Illuminate\Support\Str;

    $reversed = Str::reverse('Hello World');

    // dlroW olleH

<a name="method-str-singular"></a>
#### `Str::singular()` {.collection-method}

`Str::singular` メソッドは、文字列を単数形に変換します。この関数は、[Laravelの複数形化機能がサポートする言語](localization.md#pluralization-language)をサポートしています:

    use Illuminate\Support\Str;

    $singular = Str::singular('cars');

    // car

    $singular = Str::singular('children');

    // child

<a name="method-str-slug"></a>
#### `Str::slug()` {.collection-method}

`Str::slug` メソッドは、指定された文字列からURLフレンドリーな「スラッグ」を生成します:

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` {.collection-method}

`Str::snake` メソッドは、指定された文字列を `snake_case` に変換します:

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

    $converted = Str::snake('fooBar', '-');

    // foo-bar

<a name="method-str-squish"></a>
#### `Str::squish()` {.collection-method}

`Str::squish` メソッドは、文字列からすべての余分な空白を削除します。これには、単語間の余分な空白も含まれます:

    use Illuminate\Support\Str;

    $string = Str::squish('    laravel    framework    ');

    // laravel framework

<a name="method-str-start"></a>
#### `Str::start()` {.collection-method}

`Str::start` メソッドは、文字列が指定された値で始まっていない場合に、その値の1つのインスタンスを文字列に追加します:

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` {.collection-method}

`Str::startsWith` メソッドは、指定された文字列が指定された値で始まるかどうかを判断します:

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

配列で可能な値を渡すと、`startsWith` メソッドは文字列がその中のいずれかの値で始まる場合に `true` を返します:

    $result = Str::startsWith('This is my name', ['This', 'That', 'There']);

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` {.collection-method}

`Str::studly` メソッドは、指定された文字列を `StudlyCase` に変換します:

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-str-substr"></a>
#### `Str::substr()` {.collection-method}

`Str::substr` メソッドは、開始位置と長さのパラメータで指定された文字列の部分を返します:

    use Illuminate\Support\Str;

    $converted = Str::substr('The Laravel Framework', 4, 7);

    // Laravel

<a name="method-str-substrcount"></a>
#### `Str::substrCount()` {.collection-method}

`Str::substrCount` メソッドは、指定された文字列内で指定された値が出現する回数を返します:

    use Illuminate\Support\Str;

    $count = Str::substrCount('If you like ice cream, you will like snow cones.', 'like');

    // 2

<a name="method-str-substrreplace"></a>
#### `Str::substrReplace()` {.collection-method}

`Str::substrReplace` メソッドは、文字列の一部のテキストを置換します。第三引数で指定された位置から始まり、第四引数で指定された文字数を置換します。メソッドの第四引数に `0` を渡すと、指定された位置に文字列を挿入し、既存の文字を置換しません:

    use Illuminate\Support\Str;

    $result = Str::substrReplace('1300', ':', 2);
    // 13:

    $result = Str::substrReplace('1300', ':', 2, 0);
    // 13:00

<a name="method-str-swap"></a>
#### `Str::swap()` {.collection-method}

`Str::swap` メソッドは、PHPの `strtr` 関数を使用して、指定された文字列内の複数の値を置換します:

    use Illuminate\Support\Str;

    $string = Str::swap([
        'Tacos' => 'Burritos',
        'great' => 'fantastic',
    ], 'Tacos are great!');

    // Burritos are fantastic!

<a name="method-take"></a>
#### `Str::take()` {.collection-method}

`Str::take` メソッドは、文字列の先頭から指定された数の文字を返します:

    use Illuminate\Support\Str;

    $taken = Str::take('Build something amazing!', 5);

    // Build

<a name="method-title-case"></a>
#### `Str::title()` {.collection-method}

`Str::title` メソッドは、指定された文字列を `Title Case` に変換します:

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-to-base64"></a>
#### `Str::toBase64()` {.collection-method}

`Str::toBase64` メソッドは、指定された文字列をBase64に変換します:

    use Illuminate\Support\Str;

    $base64 = Str::toBase64('Laravel');

    // TGFyYXZlbA==

<a name="method-str-to-html-string"></a>
#### `Str::toHtmlString()` {.collection-method}

`Str::toHtmlString` メソッドは、文字列インスタンスを `Illuminate\Support\HtmlString` のインスタンスに変換します。これはBladeテンプレートで表示できます:

    use Illuminate\Support\Str;

    $htmlString = Str::of('Nuno Maduro')->toHtmlString();

<a name="method-str-transliterate"></a>
#### `Str::transliterate()` {.collection-method}

`Str::transliterate` メソッドは、指定された文字列を最も近いASCII表現に変換しようとします:

    use Illuminate\Support\Str;

    $email = Str::transliterate('ⓣⓔⓢⓣ@ⓛⓐⓡⓐⓥⓔⓛ.ⓒⓞⓜ');

    // 'test@laravel.com'

<a name="method-str-trim"></a>
#### `Str::trim()` {.collection-method}

`Str::trim` メソッドは、指定された文字列の先頭と末尾から空白文字（または他の文字）を取り除きます。PHPのネイティブ `trim` 関数とは異なり、`Str::trim` メソッドはUnicodeの空白文字も取り除きます:

    use Illuminate\Support\Str;

    $string = Str::trim(' foo bar ');

    // 'foo bar'

<a name="method-str-ltrim"></a>
#### `Str::ltrim()` {.collection-method}

`Str::ltrim` メソッドは、指定された文字列の先頭から空白文字（または他の文字）を取り除きます。PHPのネイティブ `ltrim` 関数とは異なり、`Str::ltrim` メソッドはUnicodeの空白文字も取り除きます:
```

```php
use Illuminate\Support\Str;

$char = Str::of('Laravel')->charAt(1);

// 'a'
```

<a name="method-str-ltrim"></a>
#### `Str::ltrim()` {.collection-method}

`Str::ltrim`メソッドは、指定された文字列の先頭から空白（または他の文字）を取り除きます。PHPのネイティブ`rtrim`関数とは異なり、`Str::ltrim`メソッドはUnicodeの空白文字も取り除きます。

```php
use Illuminate\Support\Str;

$string = Str::ltrim('  foo bar  ');

// 'foo bar  '
```

<a name="method-str-rtrim"></a>
#### `Str::rtrim()` {.collection-method}

`Str::rtrim`メソッドは、指定された文字列の末尾から空白（または他の文字）を取り除きます。PHPのネイティブ`rtrim`関数とは異なり、`Str::rtrim`メソッドはUnicodeの空白文字も取り除きます。

```php
use Illuminate\Support\Str;

$string = Str::rtrim('  foo bar  ');

// '  foo bar'
```

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()` {.collection-method}

`Str::ucfirst`メソッドは、指定された文字列の最初の文字を大文字にして返します。

```php
use Illuminate\Support\Str;

$string = Str::ucfirst('foo bar');

// Foo bar
```

<a name="method-str-ucsplit"></a>
#### `Str::ucsplit()` {.collection-method}

`Str::ucsplit`メソッドは、指定された文字列を大文字で分割して配列にします。

```php
use Illuminate\Support\Str;

$segments = Str::ucsplit('FooBar');

// [0 => 'Foo', 1 => 'Bar']
```

<a name="method-str-upper"></a>
#### `Str::upper()` {.collection-method}

`Str::upper`メソッドは、指定された文字列を大文字に変換します。

```php
use Illuminate\Support\Str;

$string = Str::upper('laravel');

// LARAVEL
```

<a name="method-str-ulid"></a>
#### `Str::ulid()` {.collection-method}

`Str::ulid`メソッドは、ULID（コンパクトで時間順の一意の識別子）を生成します。

```php
use Illuminate\Support\Str;

return (string) Str::ulid();

// 01gd6r360bp37zj17nxb55yv40
```

指定されたULIDが作成された日時を表す`Illuminate\Support\Carbon`の日付インスタンスを取得したい場合は、LaravelのCarbon統合によって提供される`createFromId`メソッドを使用できます。

```php
use Illuminate\Support\Carbon;
use Illuminate\Support\Str;

$date = Carbon::createFromId((string) Str::ulid());
```

テスト中に`Str::ulid`メソッドが返す値を「偽装」することが有用な場合があります。これを行うには、`createUlidsUsing`メソッドを使用できます。

```php
use Symfony\Component\Uid\Ulid;

Str::createUlidsUsing(function () {
    return new Ulid('01HRDBNHHCKNW2AK4Z29SN82T9');
});
```

`ulid`メソッドに通常通りULIDを生成させるように指示するには、`createUlidsNormally`メソッドを呼び出します。

```php
Str::createUlidsNormally();
```

<a name="method-str-unwrap"></a>
#### `Str::unwrap()` {.collection-method}

`Str::unwrap`メソッドは、指定された文字列の先頭と末尾から指定された文字列を取り除きます。

```php
use Illuminate\Support\Str;

Str::unwrap('-Laravel-', '-');

// Laravel

Str::unwrap('{framework: "Laravel"}', '{', '}');

// framework: "Laravel"
```

<a name="method-str-uuid"></a>
#### `Str::uuid()` {.collection-method}

`Str::uuid`メソッドは、UUID（バージョン4）を生成します。

```php
use Illuminate\Support\Str;

return (string) Str::uuid();
```

テスト中に`Str::uuid`メソッドが返す値を「偽装」することが有用な場合があります。これを行うには、`createUuidsUsing`メソッドを使用できます。

```php
use Ramsey\Uuid\Uuid;

Str::createUuidsUsing(function () {
    return Uuid::fromString('eadbfeac-5258-45c2-bab7-ccb9b5ef74f9');
});
```

`uuid`メソッドに通常通りUUIDを生成させるように指示するには、`createUuidsNormally`メソッドを呼び出します。

```php
Str::createUuidsNormally();
```

<a name="method-str-word-count"></a>
#### `Str::wordCount()` {.collection-method}

`Str::wordCount`メソッドは、文字列に含まれる単語の数を返します。

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-word-wrap"></a>
#### `Str::wordWrap()` {.collection-method}

`Str::wordWrap`メソッドは、文字列を指定された文字数で折り返します。

```php
use Illuminate\Support\Str;

$text = "The quick brown fox jumped over the lazy dog."

Str::wordWrap($text, characters: 20, break: "<br />\n");

/*
The quick brown fox<br />
jumped over the lazy<br />
dog.
*/
```

<a name="method-str-words"></a>
#### `Str::words()` {.collection-method}

`Str::words`メソッドは、文字列内の単語数を制限します。このメソッドの第3引数を介して、切り捨てられた文字列の末尾に追加する文字列を指定できます。

```php
use Illuminate\Support\Str;

return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

// Perfectly balanced, as >>>
```

<a name="method-str-wrap"></a>
#### `Str::wrap()` {.collection-method}

`Str::wrap`メソッドは、指定された文字列を追加の文字列または文字列のペアでラップします。

```php
use Illuminate\Support\Str;

Str::wrap('Laravel', '"');

// "Laravel"

Str::wrap('is', before: 'This ', after: ' Laravel!');

// This is Laravel!
```

<a name="method-str"></a>
#### `str()` {.collection-method}

`str`関数は、指定された文字列の新しい`Illuminate\Support\Stringable`インスタンスを返します。この関数は`Str::of`メソッドと同等です。

```php
$string = str('Taylor')->append(' Otwell');

// 'Taylor Otwell'
```

`str`関数に引数が提供されない場合、関数は`Illuminate\Support\Str`のインスタンスを返します。

```php
$snake = str()->snake('FooBar');

// 'foo_bar'
```

<a name="method-trans"></a>
#### `trans()` {.collection-method}

`trans`関数は、[言語ファイル](localization.md)を使用して指定された翻訳キーを翻訳します。

```php
echo trans('messages.welcome');
```

指定された翻訳キーが存在しない場合、`trans`関数は指定されたキーを返します。したがって、上記の例では、翻訳キーが存在しない場合、`trans`関数は`messages.welcome`を返します。

<a name="method-trans-choice"></a>
#### `trans_choice()` {.collection-method}

`trans_choice`関数は、指定された翻訳キーを屈折させて翻訳します。

```php
echo trans_choice('messages.notifications', $unreadCount);
```

指定された翻訳キーが存在しない場合、`trans_choice`関数は指定されたキーを返します。したがって、上記の例では、翻訳キーが存在しない場合、`trans_choice`関数は`messages.notifications`を返します。

<a name="fluent-strings"></a>
## 流れるような文字列

流れるような文字列は、文字列値を操作するためのより流暢でオブジェクト指向のインターフェースを提供し、従来の文字列操作と比較して、より読みやすい構文で複数の文字列操作を連鎖させることができます。

<a name="method-fluent-str-after"></a>
#### `after` {.collection-method}

`after`メソッドは、文字列内の指定された値の後にあるすべてを返します。文字列内に値が存在しない場合、文字列全体が返されます。

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->after('This is');

// ' my name'
```

<a name="method-fluent-str-after-last"></a>
#### `afterLast` {.collection-method}

`afterLast`メソッドは、文字列内の指定された値の最後の出現以降のすべてを返します。文字列内に値が存在しない場合、文字列全体が返されます。

```php
use Illuminate\Support\Str;

$slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

// 'Controller'
```

<a name="method-fluent-str-apa"></a>
#### `apa` {.collection-method}

`apa`メソッドは、指定された文字列を[APAガイドライン](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case)に従ってタイトルケースに変換します。

```php
use Illuminate\Support\Str;

$converted = Str::of('a nice title uses the correct case')->apa();

// A Nice Title Uses the Correct Case
```

<a name="method-fluent-str-append"></a>
#### `append` {.collection-method}

`append`メソッドは、指定された値を文字列に追加します。

```php
use Illuminate\Support\Str;

$string = Str::of('Taylor')->append(' Otwell');

// 'Taylor Otwell'
```

<a name="method-fluent-str-ascii"></a>
#### `ascii` {.collection-method}

`ascii`メソッドは、文字列をASCII値に変換しようとします。

```php
use Illuminate\Support\Str;

$string = Str::of('ü')->ascii();

// 'u'
```

<a name="method-fluent-str-basename"></a>
#### `basename` {.collection-method}

`basename`メソッドは、指定された文字列の末尾の名前コンポーネントを返します。

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->basename();

// 'baz'
```

必要に応じて、末尾のコンポーネントから削除する「拡張子」を指定できます。

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

// 'baz'
```

<a name="method-fluent-str-before"></a>
#### `before` {.collection-method}

`before`メソッドは、文字列内の指定された値の前にあるすべてを返します。

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->before('my name');

// 'This is '
```

<a name="method-fluent-str-before-last"></a>
#### `beforeLast` {.collection-method}

`beforeLast`メソッドは、文字列内の指定された値の最後の出現の前にあるすべてを返します。

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->beforeLast('is');

// 'This '
```

<a name="method-fluent-str-between"></a>
#### `between` {.collection-method}

`between`メソッドは、2つの値の間の文字列の部分を返します。

```php
use Illuminate\Support\Str;

$converted = Str::of('This is my name')->between('This', 'name');

// ' is my '
```

<a name="method-fluent-str-between-first"></a>
#### `betweenFirst` {.collection-method}

`betweenFirst`メソッドは、2つの値の間の文字列の最小の部分を返します。

```php
use Illuminate\Support\Str;

$converted = Str::of('[a] bc [d]')->betweenFirst('[', ']');

// 'a'
```

<a name="method-fluent-str-camel"></a>
#### `camel` {.collection-method}

`camel`メソッドは、指定された文字列を`camelCase`に変換します。

```php
use Illuminate\Support\Str;

$converted = Str::of('foo_bar')->camel();

// 'fooBar'
```

<a name="method-fluent-str-char-at"></a>
#### `charAt` {.collection-method}

`charAt`メソッドは、指定されたインデックスの文字を返します。インデックスが範囲外の場合、`false`が返されます。

    use Illuminate\Support\Str;

    $character = Str::of('This is my name.')->charAt(6);

    // 's'

<a name="method-fluent-str-class-basename"></a>
#### `classBasename` {.collection-method}

`classBasename`メソッドは、指定されたクラスの名前を返しますが、クラスの名前空間は削除されます。

    use Illuminate\Support\Str;

    $class = Str::of('Foo\Bar\Baz')->classBasename();

    // 'Baz'

<a name="method-fluent-str-chop-start"></a>
#### `chopStart` {.collection-method}

`chopStart`メソッドは、文字列の先頭に指定された値がある場合に、その値の最初の出現を削除します。

    use Illuminate\Support\Str;

    $url = Str::of('https://laravel.com')->chopStart('https://');

    // 'laravel.com'

配列を渡すこともできます。文字列が配列内のいずれかの値で始まる場合、その値は文字列から削除されます。

    use Illuminate\Support\Str;

    $url = Str::of('http://laravel.com')->chopStart(['https://', 'http://']);

    // 'laravel.com'

<a name="method-fluent-str-chop-end"></a>
#### `chopEnd` {.collection-method}

`chopEnd`メソッドは、文字列の末尾に指定された値がある場合に、その値の最後の出現を削除します。

    use Illuminate\Support\Str;

    $url = Str::of('https://laravel.com')->chopEnd('.com');

    // 'https://laravel'

配列を渡すこともできます。文字列が配列内のいずれかの値で終わる場合、その値は文字列から削除されます。

    use Illuminate\Support\Str;

    $url = Str::of('http://laravel.com')->chopEnd(['.com', '.io']);

    // 'http://laravel'

<a name="method-fluent-str-contains"></a>
#### `contains` {.collection-method}

`contains`メソッドは、指定された文字列に指定された値が含まれているかどうかを判断します。デフォルトでは、このメソッドは大文字と小文字を区別します。

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('my');

    // true

配列の値を渡すこともできます。指定された文字列に配列内のいずれかの値が含まれているかどうかを判断します。

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains(['my', 'foo']);

    // true

大文字と小文字を区別しないようにするには、`ignoreCase`引数を`true`に設定します。

    use Illuminate\Support\Str;

    $contains = Str::of('This is my name')->contains('MY', ignoreCase: true);

    // true

<a name="method-fluent-str-contains-all"></a>
#### `containsAll` {.collection-method}

`containsAll`メソッドは、指定された文字列に指定された配列内のすべての値が含まれているかどうかを判断します。

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

    // true

大文字と小文字を区別しないようにするには、`ignoreCase`引数を`true`に設定します。

    use Illuminate\Support\Str;

    $containsAll = Str::of('This is my name')->containsAll(['MY', 'NAME'], ignoreCase: true);

    // true
    
<a name="method-fluent-str-deduplicate"></a>
#### `deduplicate` {.collection-method}

`deduplicate`メソッドは、指定された文字列内の連続する文字のインスタンスを、その文字の単一のインスタンスに置き換えます。デフォルトでは、このメソッドはスペースを重複排除します。

    use Illuminate\Support\Str;

    $result = Str::of('The   Laravel   Framework')->deduplicate();

    // The Laravel Framework

重複排除する別の文字を指定するには、その文字をメソッドの第2引数として渡します。

    use Illuminate\Support\Str;

    $result = Str::of('The---Laravel---Framework')->deduplicate('-');

    // The-Laravel-Framework

<a name="method-fluent-str-dirname"></a>
#### `dirname` {.collection-method}

`dirname`メソッドは、指定された文字列の親ディレクトリ部分を返します。

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname();

    // '/foo/bar'

必要に応じて、文字列からトリムするディレクトリレベルの数を指定できます。

    use Illuminate\Support\Str;

    $string = Str::of('/foo/bar/baz')->dirname(2);

    // '/foo'

<a name="method-fluent-str-ends-with"></a>
#### `endsWith` {.collection-method}

`endsWith`メソッドは、指定された文字列が指定された値で終わるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith('name');

    // true

配列の値を渡すこともできます。指定された文字列が配列内のいずれかの値で終わるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->endsWith(['name', 'foo']);

    // true

    $result = Str::of('This is my name')->endsWith(['this', 'foo']);

    // false

<a name="method-fluent-str-exactly"></a>
#### `exactly` {.collection-method}

`exactly`メソッドは、指定された文字列が別の文字列と完全に一致するかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('Laravel')->exactly('Laravel');

    // true

<a name="method-fluent-str-excerpt"></a>
#### `excerpt` {.collection-method}

`excerpt`メソッドは、文字列内で最初に一致するフレーズに一致する抜粋を抽出します。

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('my', [
        'radius' => 3
    ]);

    // '...is my na...'

`radius`オプション（デフォルトは`100`）を使用して、切り捨てられた文字列の両側に表示される文字数を定義できます。

さらに、`omission`オプションを使用して、切り捨てられた文字列の前後に追加される文字列を変更できます。

    use Illuminate\Support\Str;

    $excerpt = Str::of('This is my name')->excerpt('name', [
        'radius' => 3,
        'omission' => '(...) '
    ]);

    // '(...) my name'

<a name="method-fluent-str-explode"></a>
#### `explode` {.collection-method}

`explode`メソッドは、指定された区切り文字で文字列を分割し、分割された各部分を含むコレクションを返します。

    use Illuminate\Support\Str;

    $collection = Str::of('foo bar baz')->explode(' ');

    // collect(['foo', 'bar', 'baz'])

<a name="method-fluent-str-finish"></a>
#### `finish` {.collection-method}

`finish`メソッドは、文字列が指定された値で終わっていない場合に、その値の単一のインスタンスを文字列に追加します。

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->finish('/');

    // this/string/

    $adjusted = Str::of('this/string/')->finish('/');

    // this/string/

<a name="method-fluent-str-headline"></a>
#### `headline` {.collection-method}

`headline`メソッドは、大文字と小文字、ハイフン、またはアンダースコアで区切られた文字列を、各単語の最初の文字が大文字になるスペース区切りの文字列に変換します。

    use Illuminate\Support\Str;

    $headline = Str::of('taylor_otwell')->headline();

    // Taylor Otwell

    $headline = Str::of('EmailNotificationSent')->headline();

    // Email Notification Sent

<a name="method-fluent-str-inline-markdown"></a>
#### `inlineMarkdown` {.collection-method}

`inlineMarkdown`メソッドは、[CommonMark](https://commonmark.thephpleague.com/)を使用してGitHub風のMarkdownをインラインHTMLに変換します。ただし、`markdown`メソッドとは異なり、生成されたすべてのHTMLをブロックレベル要素でラップしません。

    use Illuminate\Support\Str;

    $html = Str::of('**Laravel**')->inlineMarkdown();

    // <strong>Laravel</strong>

#### Markdownセキュリティ

デフォルトでは、Markdownは生のHTMLをサポートしており、生のユーザー入力と共に使用するとクロスサイトスクリプティング（XSS）の脆弱性が発生します。[CommonMarkセキュリティドキュメント](https://commonmark.thephpleague.com/security/)に従い、`html_input`オプションを使用して生のHTMLをエスケープまたは削除し、`allow_unsafe_links`オプションを使用して安全でないリンクを許可するかどうかを指定できます。生のHTMLを一部許可する必要がある場合は、コンパイルされたMarkdownをHTML Purifierを通して渡すべきです。

    use Illuminate\Support\Str;

    Str::of('Inject: <script>alert("Hello XSS!");</script>')->inlineMarkdown([
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // Inject: alert(&quot;Hello XSS!&quot;);

<a name="method-fluent-str-is"></a>
#### `is` {.collection-method}

`is`メソッドは、指定された文字列が指定されたパターンに一致するかどうかを判断します。アスタリスクはワイルドカードとして使用できます。

    use Illuminate\Support\Str;

    $matches = Str::of('foobar')->is('foo*');

    // true

    $matches = Str::of('foobar')->is('baz*');

    // false

<a name="method-fluent-str-is-ascii"></a>
#### `isAscii` {.collection-method}

`isAscii`メソッドは、指定された文字列がASCII文字列であるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('Taylor')->isAscii();

    // true

    $result = Str::of('ü')->isAscii();

    // false

<a name="method-fluent-str-is-empty"></a>
#### `isEmpty` {.collection-method}

`isEmpty`メソッドは、指定された文字列が空であるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isEmpty();

    // true

    $result = Str::of('Laravel')->trim()->isEmpty();

    // false

<a name="method-fluent-str-is-not-empty"></a>
#### `isNotEmpty` {.collection-method}

`isNotEmpty`メソッドは、指定された文字列が空でないかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('  ')->trim()->isNotEmpty();

    // false

    $result = Str::of('Laravel')->trim()->isNotEmpty();

    // true

<a name="method-fluent-str-is-json"></a>
#### `isJson` {.collection-method}

`isJson`メソッドは、指定された文字列が有効なJSONであるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('[1,2,3]')->isJson();

    // true

    $result = Str::of('{"first": "John", "last": "Doe"}')->isJson();

    // true

    $result = Str::of('{first: "John", last: "Doe"}')->isJson();

    // false

<a name="method-fluent-str-is-ulid"></a>
#### `isUlid` {.collection-method}

`isUlid`メソッドは、指定された文字列がULIDであるかどうかを判断します。

    use Illuminate\Support\Str;

    $result = Str::of('01gd6r360bp37zj17nxb55yv40')->isUlid();

    // true

    $result = Str::of('Taylor')->isUlid();

    // false
```

<a name="method-fluent-str-is-url"></a>
#### `isUrl` {.collection-method}

`isUrl`メソッドは、指定された文字列がURLであるかどうかを判定します。

    use Illuminate\Support\Str;

    $result = Str::of('http://example.com')->isUrl();

    // true

    $result = Str::of('Taylor')->isUrl();

    // false

`isUrl`メソッドは、幅広いプロトコルを有効と見なします。ただし、`isUrl`メソッドにそれらを渡すことで、有効と見なすべきプロトコルを指定できます。

    $result = Str::of('http://example.com')->isUrl(['http', 'https']);

<a name="method-fluent-str-is-uuid"></a>
#### `isUuid` {.collection-method}

`isUuid`メソッドは、指定された文字列がUUIDであるかどうかを判定します。

    use Illuminate\Support\Str;

    $result = Str::of('5ace9ab9-e9cf-4ec6-a19d-5881212a452c')->isUuid();

    // true

    $result = Str::of('Taylor')->isUuid();

    // false

<a name="method-fluent-str-kebab"></a>
#### `kebab` {.collection-method}

`kebab`メソッドは、指定された文字列を`kebab-case`に変換します。

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->kebab();

    // foo-bar

<a name="method-fluent-str-lcfirst"></a>
#### `lcfirst` {.collection-method}

`lcfirst`メソッドは、指定された文字列の最初の文字を小文字にして返します。

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->lcfirst();

    // foo Bar

<a name="method-fluent-str-length"></a>
#### `length` {.collection-method}

`length`メソッドは、指定された文字列の長さを返します。

    use Illuminate\Support\Str;

    $length = Str::of('Laravel')->length();

    // 7

<a name="method-fluent-str-limit"></a>
#### `limit` {.collection-method}

`limit`メソッドは、指定された文字列を指定された長さに切り詰めます。

    use Illuminate\Support\Str;

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

    // The quick brown fox...

切り詰められた文字列の末尾に追加される文字列を変更するために、第二引数を渡すこともできます。

    $truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

    // The quick brown fox (...)

文字列を切り詰める際に完全な単語を保持したい場合は、`preserveWords`引数を使用できます。この引数が`true`の場合、文字列は最も近い完全な単語の境界で切り詰められます。

    $truncated = Str::of('The quick brown fox')->limit(12, preserveWords: true);

    // The quick...

<a name="method-fluent-str-lower"></a>
#### `lower` {.collection-method}

`lower`メソッドは、指定された文字列を小文字に変換します。

    use Illuminate\Support\Str;

    $result = Str::of('LARAVEL')->lower();

    // 'laravel'

<a name="method-fluent-str-markdown"></a>
#### `markdown` {.collection-method}

`markdown`メソッドは、GitHub風のMarkdownをHTMLに変換します。

    use Illuminate\Support\Str;

    $html = Str::of('# Laravel')->markdown();

    // <h1>Laravel</h1>

    $html = Str::of('# Taylor <b>Otwell</b>')->markdown([
        'html_input' => 'strip',
    ]);

    // <h1>Taylor Otwell</h1>

#### Markdownセキュリティ

デフォルトでは、Markdownは生のHTMLをサポートしており、生のユーザー入力と共に使用するとクロスサイトスクリプティング（XSS）の脆弱性が露呈します。[CommonMarkセキュリティドキュメント](https://commonmark.thephpleague.com/security/)に従い、`html_input`オプションを使用して生のHTMLをエスケープまたは削除し、`allow_unsafe_links`オプションを使用して安全でないリンクを許可するかどうかを指定できます。生のHTMLを一部許可する必要がある場合は、コンパイルされたMarkdownをHTML Purifierを通して渡すべきです。

    use Illuminate\Support\Str;

    Str::of('Inject: <script>alert("Hello XSS!");</script>')->markdown([
        'html_input' => 'strip',
        'allow_unsafe_links' => false,
    ]);

    // <p>Inject: alert(&quot;Hello XSS!&quot;);</p>

<a name="method-fluent-str-mask"></a>
#### `mask` {.collection-method}

`mask`メソッドは、文字列の一部を繰り返し文字でマスクし、メールアドレスや電話番号などの文字列のセグメントを難読化するために使用できます。

    use Illuminate\Support\Str;

    $string = Str::of('taylor@example.com')->mask('*', 3);

    // tay***************

必要に応じて、`mask`メソッドに負の数を第三または第四引数として渡すことができます。これにより、文字列の終わりから指定された距離でマスクを開始するようにメソッドに指示します。

    $string = Str::of('taylor@example.com')->mask('*', -15, 3);

    // tay***@example.com

    $string = Str::of('taylor@example.com')->mask('*', 4, -4);

    // tayl**********.com

<a name="method-fluent-str-match"></a>
#### `match` {.collection-method}

`match`メソッドは、指定された正規表現パターンに一致する文字列の部分を返します。

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->match('/bar/');

    // 'bar'

    $result = Str::of('foo bar')->match('/foo (.*)/');

    // 'bar'

<a name="method-fluent-str-match-all"></a>
#### `matchAll` {.collection-method}

`matchAll`メソッドは、指定された正規表現パターンに一致する文字列の部分を含むコレクションを返します。

    use Illuminate\Support\Str;

    $result = Str::of('bar foo bar')->matchAll('/bar/');

    // collect(['bar', 'bar'])

式内で一致するグループを指定すると、Laravelは最初の一致するグループの一致を含むコレクションを返します。

    use Illuminate\Support\Str;

    $result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

    // collect(['un', 'ly']);

一致が見つからない場合、空のコレクションが返されます。

<a name="method-fluent-str-is-match"></a>
#### `isMatch` {.collection-method}

`isMatch`メソッドは、文字列が指定された正規表現に一致する場合に`true`を返します。

    use Illuminate\Support\Str;

    $result = Str::of('foo bar')->isMatch('/foo (.*)/');

    // true

    $result = Str::of('laravel')->isMatch('/foo (.*)/');

    // false

<a name="method-fluent-str-new-line"></a>
#### `newLine` {.collection-method}

`newLine`メソッドは、文字列に「行末」文字を追加します。

    use Illuminate\Support\Str;

    $padded = Str::of('Laravel')->newLine()->append('Framework');

    // 'Laravel
    //  Framework'

<a name="method-fluent-str-padboth"></a>
#### `padBoth` {.collection-method}

`padBoth`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が希望の長さになるまで、文字列の両側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padBoth(10, '_');

    // '__James___'

    $padded = Str::of('James')->padBoth(10);

    // '  James   '

<a name="method-fluent-str-padleft"></a>
#### `padLeft` {.collection-method}

`padLeft`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が希望の長さになるまで、文字列の左側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padLeft(10, '-=');

    // '-=-=-James'

    $padded = Str::of('James')->padLeft(10);

    // '     James'

<a name="method-fluent-str-padright"></a>
#### `padRight` {.collection-method}

`padRight`メソッドは、PHPの`str_pad`関数をラップし、最終的な文字列が希望の長さになるまで、文字列の右側を別の文字列でパディングします。

    use Illuminate\Support\Str;

    $padded = Str::of('James')->padRight(10, '-');

    // 'James-----'

    $padded = Str::of('James')->padRight(10);

    // 'James     '

<a name="method-fluent-str-pipe"></a>
#### `pipe` {.collection-method}

`pipe`メソッドは、現在の値を指定されたコールバックに渡すことで文字列を変換できます。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

    // 'Checksum: a5c95b86291ea299fcbe64458ed12702'

    $closure = Str::of('foo')->pipe(function (Stringable $str) {
        return 'bar';
    });

    // 'bar'

<a name="method-fluent-str-plural"></a>
#### `plural` {.collection-method}

`plural`メソッドは、単数形の単語文字列を複数形に変換します。この関数は、[Laravelの複数形化機能がサポートする任意の言語](localization.md#pluralization-language)をサポートします。

    use Illuminate\Support\Str;

    $plural = Str::of('car')->plural();

    // cars

    $plural = Str::of('child')->plural();

    // children

関数に第二引数として整数を渡すことで、文字列の単数形または複数形を取得できます。

    use Illuminate\Support\Str;

    $plural = Str::of('child')->plural(2);

    // children

    $plural = Str::of('child')->plural(1);

    // child

<a name="method-fluent-str-position"></a>
#### `position` {.collection-method}

`position`メソッドは、文字列内で最初に出現する部分文字列の位置を返します。部分文字列が文字列内に存在しない場合、`false`が返されます。

    use Illuminate\Support\Str;

    $position = Str::of('Hello, World!')->position('Hello');

    // 0

    $position = Str::of('Hello, World!')->position('W');

    // 7

<a name="method-fluent-str-prepend"></a>
#### `prepend` {.collection-method}

`prepend`メソッドは、指定された値を文字列の先頭に追加します。

    use Illuminate\Support\Str;

    $string = Str::of('Framework')->prepend('Laravel ');

    // Laravel Framework

<a name="method-fluent-str-remove"></a>
#### `remove` {.collection-method}

`remove`メソッドは、指定された値または値の配列を文字列から削除します。

    use Illuminate\Support\Str;

    $string = Str::of('Arkansas is quite beautiful!')->remove('quite');

    // Arkansas is beautiful!

文字列を削除する際に大文字と小文字を区別しないようにするには、第二引数に`false`を渡すことができます。

<a name="method-fluent-str-repeat"></a>
#### `repeat` {.collection-method}

`repeat`メソッドは、指定された文字列を繰り返します。

```php
use Illuminate\Support\Str;

$repeated = Str::of('a')->repeat(5);

// aaaaa
```

<a name="method-fluent-str-replace"></a>
#### `replace` {.collection-method}

`replace`メソッドは、文字列内の指定された文字列を置き換えます。

    use Illuminate\Support\Str;

    $replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

    // Laravel 7.x

`replace`メソッドは、`caseSensitive`引数も受け付けます。デフォルトでは、`replace`メソッドは大文字と小文字を区別します：

    $replaced = Str::of('macOS 13.x')->replace(
        'macOS', 'iOS', caseSensitive: false
    );

<a name="method-fluent-str-replace-array"></a>
#### `replaceArray` {.collection-method}

`replaceArray`メソッドは、配列を使用して文字列内の指定された値を順次置換します：

    use Illuminate\Support\Str;

    $string = 'The event will take place between ? and ?';

    $replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

    // The event will take place between 8:30 and 9:00

<a name="method-fluent-str-replace-first"></a>
#### `replaceFirst` {.collection-method}

`replaceFirst`メソッドは、文字列内で最初に出現する指定された値を置換します：

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

    // a quick brown fox jumps over the lazy dog

<a name="method-fluent-str-replace-last"></a>
#### `replaceLast` {.collection-method}

`replaceLast`メソッドは、文字列内で最後に出現する指定された値を置換します：

    use Illuminate\Support\Str;

    $replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

    // the quick brown fox jumps over a lazy dog

<a name="method-fluent-str-replace-matches"></a>
#### `replaceMatches` {.collection-method}

`replaceMatches`メソッドは、パターンに一致する文字列のすべての部分を指定された置換文字列で置換します：

    use Illuminate\Support\Str;

    $replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

    // '15015551000'

`replaceMatches`メソッドは、指定されたパターンに一致する文字列の各部分でクロージャを呼び出すこともできます。クロージャ内で置換ロジックを実行し、置換された値を返すことができます：

    use Illuminate\Support\Str;

    $replaced = Str::of('123')->replaceMatches('/\d/', function (array $matches) {
        return '['.$matches[0].']';
    });

    // '[1][2][3]'

<a name="method-fluent-str-replace-start"></a>
#### `replaceStart` {.collection-method}

`replaceStart`メソッドは、指定された値が文字列の先頭に現れる場合にのみ、最初に出現する指定された値を置換します：

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceStart('Hello', 'Laravel');

    // Laravel World

    $replaced = Str::of('Hello World')->replaceStart('World', 'Laravel');

    // Hello World

<a name="method-fluent-str-replace-end"></a>
#### `replaceEnd` {.collection-method}

`replaceEnd`メソッドは、指定された値が文字列の末尾に現れる場合にのみ、最後に出現する指定された値を置換します：

    use Illuminate\Support\Str;

    $replaced = Str::of('Hello World')->replaceEnd('World', 'Laravel');

    // Hello Laravel

    $replaced = Str::of('Hello World')->replaceEnd('Hello', 'Laravel');

    // Hello World

<a name="method-fluent-str-scan"></a>
#### `scan` {.collection-method}

`scan`メソッドは、[`sscanf` PHP関数](https://www.php.net/manual/en/function.sscanf.php)でサポートされている形式に従って、文字列から入力をコレクションに解析します：

    use Illuminate\Support\Str;

    $collection = Str::of('filename.jpg')->scan('%[^.].%s');

    // collect(['filename', 'jpg'])

<a name="method-fluent-str-singular"></a>
#### `singular` {.collection-method}

`singular`メソッドは、文字列を単数形に変換します。この関数は、[Laravelの複数形化サポート](localization.md#pluralization-language)でサポートされている任意の言語をサポートします：

    use Illuminate\Support\Str;

    $singular = Str::of('cars')->singular();

    // car

    $singular = Str::of('children')->singular();

    // child

<a name="method-fluent-str-slug"></a>
#### `slug` {.collection-method}

`slug`メソッドは、指定された文字列からURLフレンドリーな「スラッグ」を生成します：

    use Illuminate\Support\Str;

    $slug = Str::of('Laravel Framework')->slug('-');

    // laravel-framework

<a name="method-fluent-str-snake"></a>
#### `snake` {.collection-method}

`snake`メソッドは、指定された文字列を`snake_case`に変換します：

    use Illuminate\Support\Str;

    $converted = Str::of('fooBar')->snake();

    // foo_bar

<a name="method-fluent-str-split"></a>
#### `split` {.collection-method}

`split`メソッドは、正規表現を使用して文字列をコレクションに分割します：

    use Illuminate\Support\Str;

    $segments = Str::of('one, two, three')->split('/[\s,]+/');

    // collect(["one", "two", "three"])

<a name="method-fluent-str-squish"></a>
#### `squish` {.collection-method}

`squish`メソッドは、文字列からすべての余分な空白を削除します。これには、単語間の余分な空白も含まれます：

    use Illuminate\Support\Str;

    $string = Str::of('    laravel    framework    ')->squish();

    // laravel framework

<a name="method-fluent-str-start"></a>
#### `start` {.collection-method}

`start`メソッドは、指定された値が文字列の先頭にまだない場合に、その値の単一のインスタンスを文字列に追加します：

    use Illuminate\Support\Str;

    $adjusted = Str::of('this/string')->start('/');

    // /this/string

    $adjusted = Str::of('/this/string')->start('/');

    // /this/string

<a name="method-fluent-str-starts-with"></a>
#### `startsWith` {.collection-method}

`startsWith`メソッドは、指定された文字列が指定された値で始まるかどうかを判断します：

    use Illuminate\Support\Str;

    $result = Str::of('This is my name')->startsWith('This');

    // true

<a name="method-fluent-str-strip-tags"></a>
#### `stripTags` {.collection-method}

`stripTags`メソッドは、文字列からすべてのHTMLおよびPHPタグを削除します：

    use Illuminate\Support\Str;

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags();

    // Taylor Otwell

    $result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags('<b>');

    // Taylor <b>Otwell</b>

<a name="method-fluent-str-studly"></a>
#### `studly` {.collection-method}

`studly`メソッドは、指定された文字列を`StudlyCase`に変換します：

    use Illuminate\Support\Str;

    $converted = Str::of('foo_bar')->studly();

    // FooBar

<a name="method-fluent-str-substr"></a>
#### `substr` {.collection-method}

`substr`メソッドは、指定された開始位置と長さのパラメータによって指定された文字列の部分を返します：

    use Illuminate\Support\Str;

    $string = Str::of('Laravel Framework')->substr(8);

    // Framework

    $string = Str::of('Laravel Framework')->substr(8, 5);

    // Frame

<a name="method-fluent-str-substrreplace"></a>
#### `substrReplace` {.collection-method}

`substrReplace`メソッドは、文字列の一部のテキストを置換します。これは、第二引数で指定された位置から始まり、第三引数で指定された文字数を置換します。メソッドの第三引数に`0`を渡すと、指定された位置に文字列を挿入し、既存の文字を置換しません：

    use Illuminate\Support\Str;

    $string = Str::of('1300')->substrReplace(':', 2);

    // 13:

    $string = Str::of('The Framework')->substrReplace(' Laravel', 3, 0);

    // The Laravel Framework

<a name="method-fluent-str-swap"></a>
#### `swap` {.collection-method}

`swap`メソッドは、PHPの`strtr`関数を使用して、文字列内の複数の値を置換します：

    use Illuminate\Support\Str;

    $string = Str::of('Tacos are great!')
        ->swap([
            'Tacos' => 'Burritos',
            'great' => 'fantastic',
        ]);

    // Burritos are fantastic!

<a name="method-fluent-str-take"></a>
#### `take` {.collection-method}

`take`メソッドは、文字列の先頭から指定された数の文字を返します：

    use Illuminate\Support\Str;

    $taken = Str::of('Build something amazing!')->take(5);

    // Build

<a name="method-fluent-str-tap"></a>
#### `tap` {.collection-method}

`tap`メソッドは、文字列を指定されたクロージャに渡し、文字列自体に影響を与えずに文字列を調査および操作できるようにします。クロージャによって何が返されても、`tap`メソッドは元の文字列を返します：

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Laravel')
        ->append(' Framework')
        ->tap(function (Stringable $string) {
            dump('String after append: '.$string);
        })
        ->upper();

    // LARAVEL FRAMEWORK

<a name="method-fluent-str-test"></a>
#### `test` {.collection-method}

`test`メソッドは、文字列が指定された正規表現パターンに一致するかどうかを判断します：

    use Illuminate\Support\Str;

    $result = Str::of('Laravel Framework')->test('/Laravel/');

    // true

<a name="method-fluent-str-title"></a>
#### `title` {.collection-method}

`title`メソッドは、指定された文字列を`Title Case`に変換します：

    use Illuminate\Support\Str;

    $converted = Str::of('a nice title uses the correct case')->title();

    // A Nice Title Uses The Correct Case

<a name="method-fluent-str-to-base64"></a>
#### `toBase64()` {.collection-method}

`toBase64`メソッドは、指定された文字列をBase64に変換します：

    use Illuminate\Support\Str;

    $base64 = Str::of('Laravel')->toBase64();

    // TGFyYXZlbA==

<a name="method-fluent-str-transliterate"></a>
#### `transliterate` {.collection-method}

`transliterate`メソッドは、指定された文字列を最も近いASCII表現に変換しようとします：

    use Illuminate\Support\Str;

    $email = Str::of('ⓣⓔⓢⓣ@ⓛⓐⓡⓐⓥⓔⓛ.ⓒⓞⓜ')->transliterate()

    // 'test@laravel.com'

<a name="method-fluent-str-trim"></a>
#### `trim` {.collection-method}

`trim`メソッドは、指定された文字列をトリミングします。PHPのネイティブ`trim`関数とは異なり、Laravelの`trim`メソッドはUnicodeの空白文字も削除します：

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->trim();
    
    // 'Laravel'
    
    $string = Str::of('/Laravel/')->trim('/');
    
    // 'Laravel'

<a name="method-fluent-str-ltrim"></a>
#### `ltrim` {.collection-method}

`ltrim`メソッドは文字列の左側をトリムします。PHPのネイティブ`ltrim`関数とは異なり、Laravelの`ltrim`メソッドはUnicodeの空白文字も削除します。

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->ltrim();

    // 'Laravel  '

    $string = Str::of('/Laravel/')->ltrim('/');

    // 'Laravel/'

<a name="method-fluent-str-rtrim"></a>
#### `rtrim` {.collection-method}

`rtrim`メソッドは指定された文字列の右側をトリムします。PHPのネイティブ`rtrim`関数とは異なり、Laravelの`rtrim`メソッドはUnicodeの空白文字も削除します。

    use Illuminate\Support\Str;

    $string = Str::of('  Laravel  ')->rtrim();

    // '  Laravel'

    $string = Str::of('/Laravel/')->rtrim('/');

    // '/Laravel'

<a name="method-fluent-str-ucfirst"></a>
#### `ucfirst` {.collection-method}

`ucfirst`メソッドは指定された文字列の最初の文字を大文字にして返します。

    use Illuminate\Support\Str;

    $string = Str::of('foo bar')->ucfirst();

    // Foo bar

<a name="method-fluent-str-ucsplit"></a>
#### `ucsplit` {.collection-method}

`ucsplit`メソッドは指定された文字列を大文字で分割し、コレクションにします。

    use Illuminate\Support\Str;

    $string = Str::of('Foo Bar')->ucsplit();

    // collect(['Foo', 'Bar'])

<a name="method-fluent-str-unwrap"></a>
#### `unwrap` {.collection-method}

`unwrap`メソッドは指定された文字列の最初と最後から指定された文字列を削除します。

    use Illuminate\Support\Str;

    Str::of('-Laravel-')->unwrap('-');

    // Laravel

    Str::of('{framework: "Laravel"}')->unwrap('{', '}');

    // framework: "Laravel"

<a name="method-fluent-str-upper"></a>
#### `upper` {.collection-method}

`upper`メソッドは指定された文字列を大文字に変換します。

    use Illuminate\Support\Str;

    $adjusted = Str::of('laravel')->upper();

    // LARAVEL

<a name="method-fluent-str-when"></a>
#### `when` {.collection-method}

`when`メソッドは指定された条件が`true`の場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Taylor')
                    ->when(true, function (Stringable $string) {
                        return $string->append(' Otwell');
                    });

    // 'Taylor Otwell'

必要に応じて、`when`メソッドの第3引数として別のクロージャを渡すことができます。このクロージャは条件パラメータが`false`と評価された場合に実行されます。

<a name="method-fluent-str-when-contains"></a>
#### `whenContains` {.collection-method}

`whenContains`メソッドは、文字列が指定された値を含む場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains('tony', function (Stringable $string) {
                    return $string->title();
                });

    // 'Tony Stark'

必要に応じて、`when`メソッドの第3引数として別のクロージャを渡すことができます。このクロージャは文字列が指定された値を含まない場合に実行されます。

また、文字列が配列内のいずれかの値を含むかどうかを判断するために、配列を渡すこともできます。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                ->whenContains(['tony', 'hulk'], function (Stringable $string) {
                    return $string->title();
                });

    // Tony Stark

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll` {.collection-method}

`whenContainsAll`メソッドは、文字列が指定されたすべての部分文字列を含む場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('tony stark')
                    ->whenContainsAll(['tony', 'stark'], function (Stringable $string) {
                        return $string->title();
                    });

    // 'Tony Stark'

必要に応じて、`when`メソッドの第3引数として別のクロージャを渡すことができます。このクロージャは条件パラメータが`false`と評価された場合に実行されます。

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty` {.collection-method}

`whenEmpty`メソッドは、文字列が空の場合に指定されたクロージャを呼び出します。クロージャが値を返す場合、その値も`whenEmpty`メソッドによって返されます。クロージャが値を返さない場合、フルーエントな文字列インスタンスが返されます。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('  ')->whenEmpty(function (Stringable $string) {
        return $string->trim()->prepend('Laravel');
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty` {.collection-method}

`whenNotEmpty`メソッドは、文字列が空でない場合に指定されたクロージャを呼び出します。クロージャが値を返す場合、その値も`whenNotEmpty`メソッドによって返されます。クロージャが値を返さない場合、フルーエントな文字列インスタンスが返されます。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('Framework')->whenNotEmpty(function (Stringable $string) {
        return $string->prepend('Laravel ');
    });

    // 'Laravel Framework'

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith` {.collection-method}

`whenStartsWith`メソッドは、文字列が指定された部分文字列で始まる場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenStartsWith('disney', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith` {.collection-method}

`whenEndsWith`メソッドは、文字列が指定された部分文字列で終わる場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('disney world')->whenEndsWith('world', function (Stringable $string) {
        return $string->title();
    });

    // 'Disney World'

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly` {.collection-method}

`whenExactly`メソッドは、文字列が指定された文字列と完全に一致する場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-not-exactly"></a>
#### `whenNotExactly` {.collection-method}

`whenNotExactly`メソッドは、文字列が指定された文字列と完全に一致しない場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('framework')->whenNotExactly('laravel', function (Stringable $string) {
        return $string->title();
    });

    // 'Framework'

<a name="method-fluent-str-when-is"></a>
#### `whenIs` {.collection-method}

`whenIs`メソッドは、文字列が指定されたパターンに一致する場合に指定されたクロージャを呼び出します。アスタリスクをワイルドカードとして使用できます。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('foo/bar')->whenIs('foo/*', function (Stringable $string) {
        return $string->append('/baz');
    });

    // 'foo/bar/baz'

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii` {.collection-method}

`whenIsAscii`メソッドは、文字列が7ビットASCIIの場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel')->whenIsAscii(function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel'

<a name="method-fluent-str-when-is-ulid"></a>
#### `whenIsUlid` {.collection-method}

`whenIsUlid`メソッドは、文字列が有効なULIDの場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;

    $string = Str::of('01gd6r360bp37zj17nxb55yv40')->whenIsUlid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // '01gd6r36'

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid` {.collection-method}

`whenIsUuid`メソッドは、文字列が有効なUUIDの場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('a0a2a2d2-0b87-4a18-83f2-2529882be2de')->whenIsUuid(function (Stringable $string) {
        return $string->substr(0, 8);
    });

    // 'a0a2a2d2'

<a name="method-fluent-str-when-test"></a>
#### `whenTest` {.collection-method}

`whenTest`メソッドは、文字列が指定された正規表現に一致する場合に指定されたクロージャを呼び出します。クロージャはフルーエントな文字列インスタンスを受け取ります。

    use Illuminate\Support\Str;
    use Illuminate\Support\Stringable;

    $string = Str::of('laravel framework')->whenTest('/laravel/', function (Stringable $string) {
        return $string->title();
    });

    // 'Laravel Framework'

<a name="method-fluent-str-word-count"></a>
#### `wordCount` {.collection-method}

`wordCount`メソッドは、文字列に含まれる単語の数を返します。

```php
use Illuminate\Support\Str;

Str::of('Hello, world!')->wordCount(); // 2
```

<a name="method-fluent-str-words"></a>
#### `words` {.collection-method}

`words`メソッドは、文字列内の単語数を制限します。必要に応じて、切り捨てられた文字列に追加される文字列を指定できます。

    use Illuminate\Support\Str;

    $string = Str::of('完璧にバランスが取れている、すべてのものがそうあるべきだ。')->words(3, ' >>>');

    // 完璧にバランスが取れている、すべてのものが >>>
