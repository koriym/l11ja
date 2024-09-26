# ヘルパー

- [はじめに](#introduction)
- [利用可能なメソッド](#available-methods)
- [その他のユーティリティ](#other-utilities)
    - [ベンチマーク](#benchmarking)
    - [日付](#dates)
    - [遅延関数](#deferred-functions)
    - [抽選](#lottery)
    - [パイプライン](#pipeline)
    - [スリープ](#sleep)

<a name="introduction"></a>
## はじめに

Laravelには、さまざまなグローバルな「ヘルパー」PHP関数が含まれています。これらの関数の多くはフレームワーク自体によって使用されますが、自分のアプリケーションでも便利だと思う場合は自由に使用できます。

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

<a name="arrays-and-objects-method-list"></a>
### 配列とオブジェクト

<div class="collection-method-list" markdown="1">

[Arr::accessible](#method-array-accessible)
[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::exists](#method-array-exists)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::hasAny](#method-array-hasany)
[Arr::isAssoc](#method-array-isassoc)
[Arr::isList](#method-array-islist)
[Arr::join](#method-array-join)
[Arr::keyBy](#method-array-keyby)
[Arr::last](#method-array-last)
[Arr::map](#method-array-map)
[Arr::mapSpread](#method-array-map-spread)
[Arr::mapWithKeys](#method-array-map-with-keys)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::prependKeysWith](#method-array-prependkeyswith)
[Arr::pull](#method-array-pull)
[Arr::query](#method-array-query)
[Arr::random](#method-array-random)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sort](#method-array-sort)
[Arr::sortDesc](#method-array-sort-desc)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::take](#method-array-take)
[Arr::toCssClasses](#method-array-to-css-classes)
[Arr::toCssStyles](#method-array-to-css-styles)
[Arr::undot](#method-array-undot)
[Arr::where](#method-array-where)
[Arr::whereNotNull](#method-array-where-not-null)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[data_forget](#method-data-forget)
[head](#method-head)
[last](#method-last)
</div>

<a name="numbers-method-list"></a>
### 数値

<div class="collection-method-list" markdown="1">

[Number::abbreviate](#method-number-abbreviate)
[Number::clamp](#method-number-clamp)
[Number::currency](#method-number-currency)
[Number::fileSize](#method-number-file-size)
[Number::forHumans](#method-number-for-humans)
[Number::format](#method-number-format)
[Number::ordinal](#method-number-ordinal)
[Number::pairs](#method-number-pairs)
[Number::percentage](#method-number-percentage)
[Number::spell](#method-number-spell)
[Number::trim](#method-number-trim)
[Number::useLocale](#method-number-use-locale)
[Number::withLocale](#method-number-with-locale)

</div>

<a name="paths-method-list"></a>
### パス

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[lang_path](#method-lang-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

<a name="urls-method-list"></a>
### URL

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[to_route](#method-to-route)
[url](#method-url)

</div>

<a name="miscellaneous-method-list"></a>
### その他

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[context](#method-context)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[decrypt](#method-decrypt)
[dd](#method-dd)
[dispatch](#method-dispatch)
[dispatch_sync](#method-dispatch-sync)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[fake](#method-fake)
[filled](#method-filled)
[info](#method-info)
[literal](#method-literal)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[once](#method-once)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[report_if](#method-report-if)
[report_unless](#method-report-unless)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[today](#method-today)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)
[when](#method-when)

</div>

<a name="arrays"></a>
## 配列とオブジェクト

<a name="method-array-accessible"></a>
#### `Arr::accessible()` {.collection-method .first-collection-method}

`Arr::accessible`メソッドは、指定された値が配列アクセス可能かどうかを判断します。

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>
#### `Arr::add()` {.collection-method}

`Arr::add`メソッドは、指定されたキーが配列に存在しないか、`null`に設定されている場合、指定されたキーと値のペアを配列に追加します。

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `Arr::collapse()` {.collection-method}

`Arr::collapse`メソッドは、配列の配列を単一の配列にフラット化します。

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()` {.collection-method}

`Arr::crossJoin`メソッドは、指定された配列をクロス結合し、すべての可能な順列のデカルト積を返します。

    use Illuminate\Support\Arr;

    $matrix = Arr::crossJoin([1, 2], ['a', 'b']);

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

    /*
        [
            [1, 'a', 'I'],
            [1, 'a', 'II'],
            [1, 'b', 'I'],
            [1, 'b', 'II'],
            [2, 'a', 'I'],
            [2, 'a', 'II'],
            [2, 'b', 'I'],
            [2, 'b', 'II'],
        ]
    */

<a name="method-array-divide"></a>
#### `Arr::divide()` {.collection-method}

`Arr::divide`メソッドは、2つの配列を返します。1つは指定された配列のキーを含み、もう1つは値を含みます。

    use Illuminate\Support\Arr;

    [$keys, $values] = Arr::divide(['name' => 'Desk']);

    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()` {.collection-method}

`Arr::dot`メソッドは、多次元配列を単一レベルの配列にフラット化し、深さを示すために「ドット」記法を使用します。

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()` {.collection-method}

`Arr::except`メソッドは、指定されたキーと値のペアを配列から削除します。

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-exists"></a>
#### `Arr::exists()` {.collection-method}

`Arr::exists`メソッドは、指定されたキーが提供された配列に存在するかどうかをチェックします。

    use Illuminate\Support\Arr;

    $array = ['name' => 'John Doe', 'age' => 17];

    $exists = Arr::exists($array, 'name');

    // true

    $exists = Arr::exists($array, 'salary');

    // false

<a name="method-array-first"></a>
#### `Arr::first()` {.collection-method}

`Arr::first`メソッドは、指定された真実テストを通過する配列の最初の要素を返します。

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 200

デフォルト値をメソッドの3番目のパラメータとして渡すこともできます。この値は、値が真実テストに合格しない場合に返されます。

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

`Arr::flatten`メソッドは、多次元配列を単一レベルの配列にフラット化します。

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

`Arr::forget`メソッドは、"ドット"記法を使用して、深くネストされた配列から指定されたキー/値のペアを削除します。

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>
#### `Arr::get()` {.collection-method}

`Arr::get`メソッドは、"ドット"記法を使用して、深くネストされた配列から値を取得します。

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $price = Arr::get($array, 'products.desk.price');

    // 100

`Arr::get`メソッドは、指定されたキーが配列に存在しない場合に返されるデフォルト値も受け入れます。

    use Illuminate\Support\Arr;

    $discount = Arr::get($array, 'products.desk.discount', 0);

    // 0

<a name="method-array-has"></a>
#### `Arr::has()` {.collection-method}

`Arr::has`メソッドは、"ドット"記法を使用して、配列内に指定されたアイテムまたはアイテムが存在するかどうかをチェックします。

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::has($array, 'product.name');

    // true

    $contains = Arr::has($array, ['product.price', 'product.discount']);

    // false

<a name="method-array-hasany"></a>
#### `Arr::hasAny()` {.collection-method}

`Arr::hasAny`メソッドは、"ドット"記法を使用して、指定されたセット内のいずれかのアイテムが配列内に存在するかどうかをチェックします。

    use Illuminate\Support\Arr;

    $array = ['product' => ['name' => 'Desk', 'price' => 100]];

    $contains = Arr::hasAny($array, 'product.name');

    // true

    $contains = Arr::hasAny($array, ['product.name', 'product.discount']);

    // true

    $contains = Arr::hasAny($array, ['category', 'product.discount']);

    // false

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()` {.collection-method}

`Arr::isAssoc`メソッドは、指定された配列が連想配列である場合に`true`を返します。配列は、ゼロから始まる連続した数値キーを持たない場合、"連想"と見なされます。

    use Illuminate\Support\Arr;

    $isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

    // true

    $isAssoc = Arr::isAssoc([1, 2, 3]);

    // false

<a name="method-array-islist"></a>
#### `Arr::isList()` {.collection-method}

`Arr::isList`メソッドは、指定された配列のキーがゼロから始まる連続した整数である場合に`true`を返します。

    use Illuminate\Support\Arr;

    $isList = Arr::isList(['foo', 'bar', 'baz']);

    // true

    $isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

    // false

<a name="method-array-join"></a>
#### `Arr::join()` {.collection-method}

`Arr::join`メソッドは、配列要素を文字列で結合します。このメソッドの第二引数を使用して、配列の最後の要素の結合文字列を指定することもできます。

    use Illuminate\Support\Arr;

    $array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

    $joined = Arr::join($array, ', ');

    // Tailwind, Alpine, Laravel, Livewire

    $joined = Arr::join($array, ', ', ' and ');

    // Tailwind, Alpine, Laravel and Livewire

<a name="method-array-keyby"></a>
#### `Arr::keyBy()` {.collection-method}

`Arr::keyBy`メソッドは、指定されたキーで配列をキー付けします。複数のアイテムが同じキーを持つ場合、新しい配列には最後のアイテムのみが表示されます。

    use Illuminate\Support\Arr;

    $array = [
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ];

    $keyed = Arr::keyBy($array, 'product_id');

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-array-last"></a>
#### `Arr::last()` {.collection-method}

`Arr::last`メソッドは、指定された真偽テストを通過する配列の最後の要素を返します。

    use Illuminate\Support\Arr;

    $array = [100, 200, 300, 110];

    $last = Arr::last($array, function (int $value, int $key) {
        return $value >= 150;
    });

    // 300

メソッドの第三引数としてデフォルト値を渡すことができます。この値は、真偽テストを通過する値がない場合に返されます。

    use Illuminate\Support\Arr;

    $last = Arr::last($array, $callback, $default);

<a name="method-array-map"></a>
#### `Arr::map()` {.collection-method}

`Arr::map`メソッドは、配列を反復処理し、各値とキーを指定されたコールバックに渡します。配列の値は、コールバックによって返された値に置き換えられます。

    use Illuminate\Support\Arr;

    $array = ['first' => 'james', 'last' => 'kirk'];

    $mapped = Arr::map($array, function (string $value, string $key) {
        return ucfirst($value);
    });

    // ['first' => 'James', 'last' => 'Kirk']

<a name="method-array-map-spread"></a>
#### `Arr::mapSpread()` {.collection-method}

`Arr::mapSpread`メソッドは、配列を反復処理し、各ネストされたアイテムの値を指定されたクロージャに渡します。クロージャは自由にアイテムを変更して返すことができ、変更されたアイテムの新しい配列を形成します。

    use Illuminate\Support\Arr;

    $array = [
        [0, 1],
        [2, 3],
        [4, 5],
        [6, 7],
        [8, 9],
    ];

    $mapped = Arr::mapSpread($array, function (int $even, int $odd) {
        return $even + $odd;
    });

    /*
        [1, 5, 9, 13, 17]
    */

<a name="method-array-map-with-keys"></a>
#### `Arr::mapWithKeys()` {.collection-method}

`Arr::mapWithKeys`メソッドは、配列を反復処理し、各値を指定されたコールバックに渡します。コールバックは、単一のキー/値ペアを含む連想配列を返す必要があります。

    use Illuminate\Support\Arr;

    $array = [
        [
            'name' => 'John',
            'department' => 'Sales',
            'email' => 'john@example.com',
        ],
        [
            'name' => 'Jane',
            'department' => 'Marketing',
            'email' => 'jane@example.com',
        ]
    ];

    $mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
        return [$item['email'] => $item['name']];
    });

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-array-only"></a>
#### `Arr::only()` {.collection-method}

`Arr::only`メソッドは、指定されたキー/値のペアのみを与えられた配列から返します。

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

    $slice = Arr::only($array, ['name', 'price']);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {.collection-method}

`Arr::pluck`メソッドは、配列から指定されたキーのすべての値を取得します。

    use Illuminate\Support\Arr;

    $array = [
        ['developer' => ['id' => 1, 'name' => 'Taylor']],
        ['developer' => ['id' => 2, 'name' => 'Abigail']],
    ];

    $names = Arr::pluck($array, 'developer.name');

    // ['Taylor', 'Abigail']

結果のリストのキーを指定することもできます。

    use Illuminate\Support\Arr;

    $names = Arr::pluck($array, 'developer.name', 'developer.id');

    // [1 => 'Taylor', 2 => 'Abigail']

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {.collection-method}

`Arr::prepend`メソッドは、アイテムを配列の先頭にプッシュします。

    use Illuminate\Support\Arr;

    $array = ['one', 'two', 'three', 'four'];

    $array = Arr::prepend($array, 'zero');

    // ['zero', 'one', 'two', 'three', 'four']

必要に応じて、値に使用するキーを指定できます。

    use Illuminate\Support\Arr;

    $array = ['price' => 100];

    $array = Arr::prepend($array, 'Desk', 'name');

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()` {.collection-method}

`Arr::prependKeysWith`メソッドは、連想配列のすべてのキー名に指定されたプレフィックスを付けます。

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Desk',
        'price' => 100,
    ];

    $keyed = Arr::prependKeysWith($array, 'product.');

    /*
        [
            'product.name' => 'Desk',
            'product.price' => 100,
        ]
    */

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

`Arr::pull`メソッドは、配列からキー/値のペアを返し、削除します。

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

メソッドの第三引数としてデフォルト値を渡すことができます。この値は、キーが存在しない場合に返されます。

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

`Arr::query`メソッドは、配列をクエリ文字列に変換します。

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Taylor',
        'order' => [
            'column' => 'created_at',
            'direction' => 'desc'
        ]
    ];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

`Arr::random`メソッドは、配列からランダムな値を返します。

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (ランダムに取得)

オプションの第二引数として返すアイテムの数を指定することもできます。この引数を指定すると、1つのアイテムが必要な場合でも配列が返されることに注意してください。

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (ランダムに取得)

<a name="method-array-set"></a>
#### `Arr::set()` {.collection-method}

`Arr::set`メソッドは、"ドット"記法を使用して、ネストされた配列内の値を設定します。

    use Illuminate\Support\Arr;
    
    $array = ['products' => ['desk' => ['price' => 100]]];
    
    Arr::set($array, 'products.desk.price', 200);
    
    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {.collection-method}

`Arr::shuffle`メソッドは、配列内のアイテムをランダムにシャッフルします。

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] - (ランダムに生成されたもの)

<a name="method-array-sort"></a>
#### `Arr::sort()` {.collection-method}

`Arr::sort`メソッドは、値によって配列をソートします。

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

また、指定されたクロージャの結果によって配列をソートすることもできます。

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()` {.collection-method}

`Arr::sortDesc`メソッドは、値によって配列を降順にソートします。

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sortDesc($array);

    // ['Table', 'Desk', 'Chair']

また、指定されたクロージャの結果によって配列をソートすることもできます。

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sortDesc($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Table'],
            ['name' => 'Desk'],
            ['name' => 'Chair'],
        ]
    */

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {.collection-method}

`Arr::sortRecursive`メソッドは、数値インデックスのサブ配列に対して`sort`関数を、連想サブ配列に対して`ksort`関数を使用して、配列を再帰的にソートします。

    use Illuminate\Support\Arr;

    $array = [
        ['Roman', 'Taylor', 'Li'],
        ['PHP', 'Ruby', 'JavaScript'],
        ['one' => 1, 'two' => 2, 'three' => 3],
    ];

    $sorted = Arr::sortRecursive($array);

    /*
        [
            ['JavaScript', 'PHP', 'Ruby'],
            ['one' => 1, 'three' => 3, 'two' => 2],
            ['Li', 'Roman', 'Taylor'],
        ]
    */

結果を降順でソートしたい場合は、`Arr::sortRecursiveDesc`メソッドを使用できます。

    $sorted = Arr::sortRecursiveDesc($array);

<a name="method-array-take"></a>
#### `Arr::take()` {.collection-method}

`Arr::take`メソッドは、指定された数のアイテムを持つ新しい配列を返します。

    use Illuminate\Support\Arr;

    $array = [0, 1, 2, 3, 4, 5];

    $chunk = Arr::take($array, 3);

    // [0, 1, 2]

また、配列の末尾から指定された数のアイテムを取得するために、負の整数を渡すこともできます。

    $array = [0, 1, 2, 3, 4, 5];

    $chunk = Arr::take($array, -2);

    // [4, 5]

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` {.collection-method}

`Arr::toCssClasses`メソッドは、条件付きでCSSクラス文字列をコンパイルします。このメソッドは、クラスの配列を受け取ります。配列のキーには追加したいクラスまたはクラス群を含み、値はブール式です。配列要素が数値キーを持つ場合、レンダリングされたクラスリストに常に含まれます。

    use Illuminate\Support\Arr;

    $isActive = false;
    $hasError = true;

    $array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

    $classes = Arr::toCssClasses($array);

    /*
        'p-4 bg-red'
    */

<a name="method-array-to-css-styles"></a>
#### `Arr::toCssStyles()` {.collection-method}

`Arr::toCssStyles`メソッドは、条件付きでCSSスタイル文字列をコンパイルします。このメソッドは、クラスの配列を受け取ります。配列のキーには追加したいクラスまたはクラス群を含み、値はブール式です。配列要素が数値キーを持つ場合、レンダリングされたクラスリストに常に含まれます。

```php
use Illuminate\Support\Arr;

$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

このメソッドは、Laravelの機能を動力としており、[Bladeコンポーネントの属性バッグとクラスをマージする](blade.md#conditionally-merge-classes)機能や、`@class` [Bladeディレクティブ](blade.md#conditional-classes)をサポートしています。

<a name="method-array-undot"></a>
#### `Arr::undot()` {.collection-method}

`Arr::undot`メソッドは、"ドット"記法を使用した一次元配列を多次元配列に展開します。

    use Illuminate\Support\Arr;

    $array = [
        'user.name' => 'Kevin Malone',
        'user.occupation' => 'Accountant',
    ];

    $array = Arr::undot($array);

    // ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]

<a name="method-array-where"></a>
#### `Arr::where()` {.collection-method}

`Arr::where`メソッドは、指定されたクロージャを使用して配列をフィルタリングします。

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function (string|int $value, int $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()` {.collection-method}

`Arr::whereNotNull`メソッドは、指定された配列からすべての`null`値を削除します。

    use Illuminate\Support\Arr;

    $array = [0, null];

    $filtered = Arr::whereNotNull($array);

    // [0 => 0]

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {.collection-method}

`Arr::wrap`メソッドは、指定された値を配列でラップします。指定された値がすでに配列である場合、変更されずに返されます。

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

指定された値が`null`の場合、空の配列が返されます。

    use Illuminate\Support\Arr;

    $array = Arr::wrap(null);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {.collection-method}

`data_fill`関数は、"ドット"記法を使用して、ネストされた配列またはオブジェクト内の欠落した値を設定します。

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_fill($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 100]]]

    data_fill($data, 'products.desk.discount', 10);

    // ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

この関数は、ワイルドカードとしてアスタリスクも受け入れ、それに応じてターゲットに値を設定します。

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2'],
        ],
    ];

    data_fill($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 100],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

<a name="method-data-get"></a>
#### `data_get()` {.collection-method}

`data_get`関数は、"ドット"記法を使用して、ネストされた配列またはオブジェクトから値を取得します。

    $data = ['products' => ['desk' => ['price' => 100]]];

    $price = data_get($data, 'products.desk.price');

    // 100

`data_get`関数は、指定されたキーが見つからない場合に返されるデフォルト値も受け入れます。

    $discount = data_get($data, 'products.desk.discount', 0);

    // 0

この関数は、アスタリスクを使用したワイルドカードも受け入れ、配列またはオブジェクトの任意のキーをターゲットにすることができます。

    $data = [
        'product-one' => ['name' => 'Desk 1', 'price' => 100],
        'product-two' => ['name' => 'Desk 2', 'price' => 150],
    ];

    data_get($data, '*.name');

    // ['Desk 1', 'Desk 2'];

`{first}`と`{last}`プレースホルダを使用して、配列の最初または最後のアイテムを取得することもできます。

    $flight = [
        'segments' => [
            ['from' => 'LHR', 'departure' => '9:00', 'to' => 'IST', 'arrival' => '15:00'],
            ['from' => 'IST', 'departure' => '16:00', 'to' => 'PKX', 'arrival' => '20:00'],
        ],
    ];

    data_get($flight, 'segments.{first}.arrival');

    // 15:00

<a name="method-data-set"></a>
#### `data_set()` {.collection-method}

`data_set`関数は、"ドット"記法を使用して、ネストされた配列またはオブジェクト内の値を設定します。

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

この関数は、アスタリスクを使用したワイルドカードも受け入れ、それに応じてターゲットに値を設定します。

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_set($data, 'products.*.price', 200);

    /*
        [
            'products' => [
                ['name' => 'Desk 1', 'price' => 200],
                ['name' => 'Desk 2', 'price' => 200],
            ],
        ]
    */

デフォルトでは、既存の値は上書きされます。値が存在しない場合にのみ値を設定したい場合は、関数の4番目の引数として`false`を渡すことができます。

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_set($data, 'products.desk.price', 200, overwrite: false);

    // ['products' => ['desk' => ['price' => 100]]]

<a name="method-data-forget"></a>
#### `data_forget()` {.collection-method}

`data_forget`関数は、"ドット"記法を使用して、ネストされた配列またはオブジェクト内の値を削除します。

    $data = ['products' => ['desk' => ['price' => 100]]];

    data_forget($data, 'products.desk.price');

    // ['products' => ['desk' => []]]

この関数は、アスタリスクを使用したワイルドカードも受け入れ、ターゲットに応じて値を削除します。

    $data = [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 150],
        ],
    ];

    data_forget($data, 'products.*.price');

    /*
        [
            'products' => [
                ['name' => 'Desk 1'],
                ['name' => 'Desk 2'],
            ],
        ]
    */

<a name="method-head"></a>
#### `head()` {.collection-method}

`head`関数は、指定された配列の最初の要素を返します。

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {.collection-method}

`last`関数は、指定された配列の最後の要素を返します。

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="numbers"></a>
## 数値

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()` {.collection-method}

`Number::abbreviate`メソッドは、提供された数値を人間が読みやすい形式で返し、単位の省略形を使用します。

    use Illuminate\Support\Number;

    $number = Number::abbreviate(1000);

    // 1K

    $number = Number::abbreviate(489939);

    // 490K

    $number = Number::abbreviate(1230000, precision: 2);

    // 1.23M

<a name="method-number-clamp"></a>
#### `Number::clamp()` {.collection-method}

`Number::clamp`メソッドは、指定された数値が指定された範囲内に収まるようにします。数値が最小値よりも低い場合は最小値が返され、数値が最大値よりも高い場合は最大値が返されます。

    use Illuminate\Support\Number;

    $number = Number::clamp(105, min: 10, max: 100);

    // 100

    $number = Number::clamp(5, min: 10, max: 100);

    // 10

    $number = Number::clamp(10, min: 10, max: 100);

    // 10

    $number = Number::clamp(20, min: 10, max: 100);

    // 20

<a name="method-number-currency"></a>
#### `Number::currency()` {.collection-method}

`Number::currency`メソッドは、指定された値を通貨表現の文字列として返します。

    use Illuminate\Support\Number;

    $currency = Number::currency(1000);

    // $1,000.00

    $currency = Number::currency(1000, in: 'EUR');

    // €1,000.00

    $currency = Number::currency(1000, in: 'EUR', locale: 'de');

    // 1.000,00 €

<a name="method-number-file-size"></a>
#### `Number::fileSize()` {.collection-method}

`Number::fileSize`メソッドは、指定されたバイト値をファイルサイズ表現の文字列として返します。

    use Illuminate\Support\Number;

    $size = Number::fileSize(1024);

    // 1 KB

    $size = Number::fileSize(1024 * 1024);

    // 1 MB

    $size = Number::fileSize(1024, precision: 2);

    // 1.00 KB

<a name="method-number-for-humans"></a>
#### `Number::forHumans()` {.collection-method}

`Number::forHumans`メソッドは、提供された数値を人間が読みやすい形式で返します。

    use Illuminate\Support\Number;

    $number = Number::forHumans(1000);

    // 1 thousand

    $number = Number::forHumans(489939);

    // 490 thousand

    $number = Number::forHumans(1230000, precision: 2);

    // 1.23 million

<a name="method-number-format"></a>
#### `Number::format()` {.collection-method}

`Number::format`メソッドは、指定された数値をロケール固有の文字列にフォーマットします。

    use Illuminate\Support\Number;

    $number = Number::format(100000);

    // 100,000

    $number = Number::format(100000, precision: 2);

    // 100,000.00

    $number = Number::format(100000.123, maxPrecision: 2);

    // 100,000.12

    $number = Number::format(100000, locale: 'de');

    // 100.000

<a name="method-number-ordinal"></a>
#### `Number::ordinal()` {.collection-method}

`Number::ordinal`メソッドは、数値の序数表現を返します。

    use Illuminate\Support\Number;

    $number = Number::ordinal(1);

    // 1st

    $number = Number::ordinal(2);

    // 2nd

    $number = Number::ordinal(21);

    // 21st

<a name="method-number-pairs"></a>
#### `Number::pairs()` {.collection-method}

`Number::pairs`メソッドは、指定された範囲とステップ値に基づいて数値のペア（サブレンジ）の配列を生成します。このメソッドは、ページネーションやバッチ処理など、大きな範囲の数値を小さな管理しやすいサブレンジに分割するのに便利です。`pairs`メソッドは、各内部配列が数値のペア（サブレンジ）を表す配列の配列を返します。

```php
use Illuminate\Support\Number;

$result = Number::pairs(25, 10);

// [[1, 10], [11, 20], [21, 25]]

$result = Number::pairs(25, 10, offset: 0);

// [[0, 10], [10, 20], [20, 25]]
```

<a name="method-number-percentage"></a>
#### `Number::percentage()` {.collection-method}

`Number::percentage`メソッドは、指定された値をパーセンテージ表現の文字列として返します。

    use Illuminate\Support\Number;

    $percentage = Number::percentage(10);

    // 10%

    $percentage = Number::percentage(10, precision: 2);

    // 10.00%

    $percentage = Number::percentage(10.123, maxPrecision: 2);

    // 10.12%

    $percentage = Number::percentage(10, precision: 2, locale: 'de');

    // 10,00%

<a name="method-number-spell"></a>
#### `Number::spell()` {.collection-method}

`Number::spell`メソッドは、指定された数値を単語の文字列に変換します。

    use Illuminate\Support\Number;

    $number = Number::spell(102);

    // one hundred and two

    $number = Number::spell(88, locale: 'fr');

    // quatre-vingt-huit

`after`引数を使用すると、すべての数値を単語で表記する値を指定できます。

    $number = Number::spell(10, after: 10);

    // 10

    $number = Number::spell(11, after: 10);

    // eleven

`until`引数を使用すると、すべての数値を単語で表記する値を指定できます。

    $number = Number::spell(5, until: 10);

    // five

    $number = Number::spell(10, until: 10);

    // 10

<a name="method-number-trim"></a>
#### `Number::trim()` {.collection-method}

`Number::trim`メソッドは、指定された数値の小数点以下の末尾のゼロを削除します。

    use Illuminate\Support\Number;

    $number = Number::trim(12.0);

    // 12

    $number = Number::trim(12.30);

    // 12.3

<a name="method-number-use-locale"></a>
#### `Number::useLocale()` {.collection-method}

`Number::useLocale`メソッドは、数値と通貨のフォーマットに影響するデフォルトの数値ロケールをグローバルに設定します。

    use Illuminate\Support\Number;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Number::useLocale('de');
    }

<a name="method-number-with-locale"></a>
#### `Number::withLocale()` {.collection-method}

`Number::withLocale`メソッドは、指定されたロケールを使用して指定されたクロージャを実行し、コールバックが実行された後に元のロケールを復元します。

    use Illuminate\Support\Number;

    $number = Number::withLocale('de', function () {
        return Number::format(1500);
    });

<a name="paths"></a>
## パス

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

`app_path`関数は、アプリケーションの`app`ディレクトリへの完全修飾パスを返します。また、`app_path`関数を使用して、アプリケーションディレクトリに対するファイルへの完全修飾パスを生成することもできます。

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

`base_path`関数は、アプリケーションのルートディレクトリへの完全修飾パスを返します。また、`base_path`関数を使用して、プロジェクトルートディレクトリに対する指定されたファイルへの完全修飾パスを生成することもできます。

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

`config_path`関数は、アプリケーションの`config`ディレクトリへの完全修飾パスを返します。また、`config_path`関数を使用して、アプリケーションの設定ディレクトリ内の指定されたファイルへの完全修飾パスを生成することもできます。

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

`database_path`関数は、アプリケーションの`database`ディレクトリへの完全修飾パスを返します。また、`database_path`関数を使用して、データベースディレクトリ内の指定されたファイルへの完全修飾パスを生成することもできます。

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

`lang_path`関数は、アプリケーションの`lang`ディレクトリへの完全修飾パスを返します。また、`lang_path`関数を使用して、ディレクトリ内の指定されたファイルへの完全修飾パスを生成することもできます。

    $path = lang_path();

    $path = lang_path('en/messages.php');

> NOTE:  
> デフォルトでは、Laravelアプリケーションのスケルトンには`lang`ディレクトリは含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisanコマンドを使用して公開できます。

<a name="method-mix"></a>
#### `mix()` {.collection-method}

`mix`関数は、[バージョン管理されたMixファイル](mix.md)へのパスを返します。

    $path = mix('css/app.css');

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

`public_path`関数は、アプリケーションの`public`ディレクトリへの完全修飾パスを返します。また、`public_path`関数を使用して、パブリックディレクトリ内の指定されたファイルへの完全修飾パスを生成することもできます。

    $path = public_path();

    $path = public_path('css/app.css');

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

`resource_path`関数は、アプリケーションの`resources`ディレクトリへの完全修飾パスを返します。また、`resource_path`関数を使用して、リソースディレクトリ内の指定されたファイルへの完全修飾パスを生成することもできます。

    $path = resource_path();

    $path = resource_path('sass/app.scss');

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

`storage_path`関数は、アプリケーションの`storage`ディレクトリへの完全修飾パスを返します。`storage`ディレクトリ内の指定されたファイルへの完全修飾パスを生成するためにも、`storage_path`関数を使用できます。

    $path = storage_path();

    $path = storage_path('app/file.txt');

<a name="urls"></a>
## URL

<a name="method-action"></a>
#### `action()` {.collection-method}

`action`関数は、指定されたコントローラアクションのURLを生成します。

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

メソッドがルートパラメータを受け入れる場合、メソッドの第2引数としてそれらを渡すことができます。

    $url = action([UserController::class, 'profile'], ['id' => 1]);

<a name="method-asset"></a>
#### `asset()` {.collection-method}

`asset`関数は、現在のリクエストのスキーム（HTTPまたはHTTPS）を使用して、アセットのURLを生成します。

    $url = asset('img/photo.jpg');

`.env`ファイルで`ASSET_URL`変数を設定することで、アセットURLのホストを構成できます。これは、Amazon S3などの外部サービスや他のCDNでアセットをホストする場合に便利です。

    // ASSET_URL=http://example.com/assets

    $url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg

<a name="method-route"></a>
#### `route()` {.collection-method}

`route`関数は、指定された[名前付きルート](routing.md#named-routes)のURLを生成します。

    $url = route('route.name');

ルートがパラメータを受け入れる場合、関数の第2引数としてそれらを渡すことができます。

    $url = route('route.name', ['id' => 1]);

デフォルトでは、`route`関数は絶対URLを生成します。相対URLを生成したい場合は、関数の第3引数として`false`を渡すことができます。

    $url = route('route.name', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

`secure_asset`関数は、HTTPSを使用してアセットのURLを生成します。

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

`secure_url`関数は、指定されたパスへの完全修飾HTTPS URLを生成します。関数の第2引数に追加のURLセグメントを渡すことができます。

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-to-route"></a>
#### `to_route()` {.collection-method}

`to_route`関数は、指定された[名前付きルート](routing.md#named-routes)の[リダイレクトHTTPレスポンス](responses.md#redirects)を生成します。

    return to_route('users.show', ['user' => 1]);

必要に応じて、リダイレクトに割り当てるHTTPステータスコードと追加のレスポンスヘッダーを、`to_route`メソッドの第3引数と第4引数として渡すことができます。

    return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);

<a name="method-url"></a>
#### `url()` {.collection-method}

`url`関数は、指定されたパスへの完全修飾URLを生成します。

    $url = url('user/profile');

    $url = url('user/profile', [1]);

パスが提供されない場合、`Illuminate\Routing\UrlGenerator`インスタンスが返されます。

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## その他

<a name="method-abort"></a>
#### `abort()` {.collection-method}

`abort`関数は、[HTTP例外](errors.md#http-exceptions)をスローします。これは、[例外ハンドラ](errors.md#handling-exceptions)によってレンダリングされます。

    abort(403);

例外のメッセージと、ブラウザに送信されるカスタムHTTPレスポンスヘッダーを提供することもできます。

    abort(403, 'Unauthorized.', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

`abort_if`関数は、指定されたブール式が`true`と評価された場合にHTTP例外をスローします。

    abort_if(! Auth::user()->isAdmin(), 403);

`abort`メソッドと同様に、例外のレスポンステキストを第3引数として、カスタムレスポンスヘッダーの配列を第4引数として関数に渡すこともできます。

<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

`abort_unless`関数は、指定されたブール式が`false`と評価された場合にHTTP例外をスローします。

    abort_unless(Auth::user()->isAdmin(), 403);

`abort`メソッドと同様に、例外のレスポンステキストを第3引数として、カスタムレスポンスヘッダーの配列を第4引数として関数に渡すこともできます。

<a name="method-app"></a>
#### `app()` {.collection-method}

`app`関数は、[サービスコンテナ](container.md)インスタンスを返します。

    $container = app();

クラスまたはインターフェース名を渡して、コンテナから解決することもできます。

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {.collection-method}

`auth`関数は、[認証](authentication.md)インスタンスを返します。`Auth`ファサードの代わりに使用できます。

    $user = auth()->user();

必要に応じて、アクセスしたいガードインスタンスを指定できます。

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {.collection-method}

`back`関数は、ユーザーの前の場所への[リダイレクトHTTPレスポンス](responses.md#redirects)を生成します。

    return back($status = 302, $headers = [], $fallback = '/');

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {.collection-method}

`bcrypt`関数は、指定された値をBcryptを使用して[ハッシュ](hashing.md)します。`Hash`ファサードの代わりにこの関数を使用できます。

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {.collection-method}

`blank`関数は、指定された値が「空白」であるかどうかを判定します。

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

`blank`の逆は、[`filled`](#method-filled)メソッドを参照してください。

<a name="method-broadcast"></a>
#### `broadcast()` {.collection-method}

`broadcast`関数は、指定された[イベント](events.md)をリスナーに[ブロードキャスト](broadcasting.md)します。

    broadcast(new UserRegistered($user));

    broadcast(new UserRegistered($user))->toOthers();

<a name="method-cache"></a>
#### `cache()` {.collection-method}

`cache`関数は、[キャッシュ](cache.md)から値を取得するために使用できます。指定されたキーがキャッシュに存在しない場合、オプションのデフォルト値が返されます。

    $value = cache('key');

    $value = cache('key', 'default');

キーと値のペアの配列を関数に渡すことで、キャッシュにアイテムを追加することもできます。また、キャッシュされた値が有効であると見なされる秒数または期間を渡す必要があります。

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {.collection-method}

`class_uses_recursive`関数は、クラスが使用するすべてのトレイトを返します。これには、親クラスが使用するすべてのトレイトも含まれます。

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect`関数は、指定された値から[コレクション](collections.md)インスタンスを作成します。

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {.collection-method}

`config`関数は、[設定](configuration.md)変数の値を取得します。設定値には、ファイル名とアクセスしたいオプションを含む「ドット」構文を使用してアクセスできます。設定オプションが存在しない場合、指定されたデフォルト値が返されます。

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

実行時にキーと値のペアの配列を渡すことで、設定変数を設定することもできます。ただし、この関数は現在のリクエストの設定値にのみ影響し、実際の設定値は更新されないことに注意してください。

    config(['app.debug' => true]);

<a name="method-context"></a>
#### `context()` {.collection-method}

`context`関数は、[現在のコンテキスト](context.md)から値を取得します。コンテキストキーが存在しない場合、指定されたデフォルト値が返されます。

    $value = context('trace_id');

    $value = context('trace_id', $default);

キーと値のペアの配列を渡すことで、コンテキスト値を設定できます。

    use Illuminate\Support\Str;

    context(['trace_id' => Str::uuid()->toString()]);

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

`cookie`関数は、新しい[クッキー](requests.md#cookies)インスタンスを作成します。

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

`csrf_field`関数は、CSRFトークンの値を含むHTMLの`hidden`入力フィールドを生成します。たとえば、[Blade構文](blade.md)を使用します。

    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

`csrf_token`関数は、現在のCSRFトークンの値を取得します。

    $token = csrf_token();

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}


`decrypt` 関数は、指定された値を[復号化](encryption.md)します。この関数は、`Crypt` ファサードの代わりに使用できます。

    $password = decrypt($value);

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 関数は、指定された変数をダンプし、スクリプトの実行を終了します。

    dd($value);

    dd($value1, $value2, $value3, ...);

スクリプトの実行を停止したくない場合は、代わりに [`dump`](#method-dump) 関数を使用してください。

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

`dispatch` 関数は、指定された [ジョブ](queues.md#creating-jobs) を Laravel の [ジョブキュー](queues.md) にプッシュします。

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()` {.collection-method}

`dispatch_sync` 関数は、指定されたジョブを [同期](queues.md#synchronous-dispatching) キューにプッシュし、即座に処理されるようにします。

    dispatch_sync(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump` 関数は、指定された変数をダンプします。

    dump($value);

    dump($value1, $value2, $value3, ...);

変数をダンプした後にスクリプトの実行を停止したい場合は、代わりに [`dd`](#method-dd) 関数を使用してください。

<a name="method-encrypt"></a>
#### `encrypt()` {.collection-method}

`encrypt` 関数は、指定された値を[暗号化](encryption.md)します。この関数は、`Crypt` ファサードの代わりに使用できます。

    $secret = encrypt('my-secret-value');

<a name="method-env"></a>
#### `env()` {.collection-method}

`env` 関数は、[環境変数](configuration.md#environment-configuration)の値を取得するか、デフォルト値を返します。

    $env = env('APP_ENV');

    $env = env('APP_ENV', 'production');

> WARNING:  
> デプロイプロセス中に `config:cache` コマンドを実行する場合、設定ファイル内でのみ `env` 関数を呼び出すようにしてください。設定がキャッシュされると、`.env` ファイルは読み込まれず、`env` 関数の呼び出しはすべて `null` を返します。

<a name="method-event"></a>
#### `event()` {.collection-method}

`event` 関数は、指定された [イベント](events.md) をリスナーにディスパッチします。

    event(new UserRegistered($user));

<a name="method-fake"></a>
#### `fake()` {.collection-method}

`fake` 関数は、コンテナから [Faker](https://github.com/FakerPHP/Faker) シングルトンを解決します。これは、モデルファクトリ、データベースシーディング、テスト、およびプロトタイプビューで偽データを作成する際に便利です。

```blade
@for($i = 0; $i < 10; $i++)
    <dl>
        <dt>Name</dt>
        <dd>{{ fake()->name() }}</dd>

        <dt>Email</dt>
        <dd>{{ fake()->unique()->safeEmail() }}</dd>
    </dl>
@endfor
```

デフォルトでは、`fake` 関数は `config/app.php` 設定ファイルの `app.faker_locale` 設定オプションを使用します。通常、この設定オプションは `APP_FAKER_LOCALE` 環境変数を介して設定されます。`fake` 関数にロケールを指定することもできます。各ロケールは個別のシングルトンとして解決されます。

    fake('nl_NL')->name()

<a name="method-filled"></a>
#### `filled()` {.collection-method}

`filled` 関数は、指定された値が "空白" でないかどうかを判断します。

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

`filled` の逆は、[`blank`](#method-blank) メソッドを参照してください。

<a name="method-info"></a>
#### `info()` {.collection-method}

`info` 関数は、アプリケーションの [ログ](logging.md) に情報を書き込みます。

    info('Some helpful information!');

コンテキストデータの配列も関数に渡すことができます。

    info('User login attempt failed.', ['id' => $user->id]);

<a name="method-literal"></a>
#### `literal()` {.collection-method}

`literal` 関数は、指定された名前付き引数をプロパティとして持つ新しい [stdClass](https://www.php.net/manual/en/class.stdclass.php) インスタンスを作成します。

    $obj = literal(
        name: 'Joe',
        languages: ['PHP', 'Ruby'],
    );

    $obj->name; // 'Joe'
    $obj->languages; // ['PHP', 'Ruby']

<a name="method-logger"></a>
#### `logger()` {.collection-method}

`logger` 関数は、[ログ](logging.md) に `debug` レベルのメッセージを書き込むために使用できます。

    logger('Debug message');

コンテキストデータの配列も関数に渡すことができます。

    logger('User has logged in.', ['id' => $user->id]);

値が関数に渡されない場合、[ロガー](logging.md) インスタンスが返されます。

    logger()->error('You are not allowed here.');

<a name="method-method-field"></a>
#### `method_field()` {.collection-method}

`method_field` 関数は、フォームの HTTP 動詞の偽装された値を含む HTML `hidden` 入力フィールドを生成します。たとえば、[Blade 構文](blade.md) を使用して以下のようにします。

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {.collection-method}

`now` 関数は、現在の時刻のための新しい `Illuminate\Support\Carbon` インスタンスを作成します。

    $now = now();

<a name="method-old"></a>
#### `old()` {.collection-method}

`old` 関数は、セッションにフラッシュされた [古い入力](requests.md#old-input) 値を[取得](requests.md#retrieving-input)します。

    $value = old('value');

    $value = old('value', 'default');

`old` 関数の第2引数として提供される "デフォルト値" は、Eloquent モデルの属性であることが多いため、Laravel では `old` 関数に第2引数として Eloquent モデル全体を渡すことができます。その場合、Laravel は `old` 関数に提供された第1引数が Eloquent 属性の名前であるとみなし、"デフォルト値" として扱います。

    {{ old('name', $user->name) }}

    // 以下と同等です...

    {{ old('name', $user) }}

<a name="method-once"></a>
#### `once()` {.collection-method}

`once` 関数は、指定されたコールバックを実行し、リクエストの期間中、メモリ内に結果をキャッシュします。同じコールバックで `once` 関数を後続で呼び出すと、以前にキャッシュされた結果が返されます。

    function random(): int
    {
        return once(function () {
            return random_int(1, 1000);
        });
    }

    random(); // 123
    random(); // 123 (キャッシュされた結果)
    random(); // 123 (キャッシュされた結果)

`once` 関数がオブジェクトインスタンス内で実行される場合、キャッシュされた結果はそのオブジェクトインスタンスに固有のものになります。

```php
<?php

class NumberService
{
    public function all(): array
    {
        return once(fn () => [1, 2, 3]);
    }
}

$service = new NumberService;

$service->all();
$service->all(); // (キャッシュされた結果)

$secondService = new NumberService;

$secondService->all();
$secondService->all(); // (キャッシュされた結果)
```

<a name="method-optional"></a>
#### `optional()` {.collection-method}

`optional` 関数は、任意の引数を受け取り、そのオブジェクトのプロパティにアクセスしたり、メソッドを呼び出したりできます。指定されたオブジェクトが `null` の場合、プロパティやメソッドはエラーを引き起こす代わりに `null` を返します。

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

`optional` 関数は、第2引数としてクロージャも受け取ります。第1引数として提供された値が `null` でない場合、クロージャが呼び出されます。

    return optional(User::find($id), function (User $user) {
        return $user->name;
    });

<a name="method-policy"></a>
#### `policy()` {.collection-method}

`policy` メソッドは、指定されたクラスの [ポリシー](authorization.md#creating-policies) インスタンスを取得します。

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

`redirect` 関数は、[リダイレクト HTTP レスポンス](responses.md#redirects) を返すか、引数なしで呼び出された場合はリダイレクタインスタンスを返します。

    return redirect($to = null, $status = 302, $headers = [], $https = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {.collection-method}

`report` 関数は、[例外ハンドラ](errors.md#handling-exceptions) を使用して例外を報告します。

    report($e);

`report` 関数は、文字列を引数として受け取ることもできます。文字列が関数に渡されると、関数は指定された文字列をメッセージとして持つ例外を作成します。

    report('Something went wrong.');

<a name="method-report-if"></a>
#### `report_if()` {.collection-method}

`report_if` 関数は、指定された条件が `true` の場合、[例外ハンドラ](errors.md#handling-exceptions) を使用して例外を報告します。

    report_if($shouldReport, $e);

    report_if($shouldReport, 'Something went wrong.');

<a name="method-report-unless"></a>
#### `report_unless()` {.collection-method}

`report_unless` 関数は、指定された条件が `false` の場合、[例外ハンドラ](errors.md#handling-exceptions) を使用して例外を報告します。

    report_unless($reportingDisabled, $e);

    report_unless($reportingDisabled, 'Something went wrong.');

<a name="method-request"></a>
#### `request()` {.collection-method}

`request` 関数は、現在の [リクエスト](requests.md) インスタンスを返すか、現在のリクエストから入力フィールドの値を取得します。

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

`rescue` 関数は、指定されたコールバックを実行し、エラーが発生した場合に例外をキャッチします。キャッチされた例外は、[例外ハンドラ](errors.md#handling-exceptions) を使用して報告されますが、スクリプトの実行は続行されます。

    return rescue(function () {
        return $this->method();
    }, 'Something went wrong.');

`rescue`関数は、指定されたクロージャを実行し、その実行中に発生する例外をすべてキャッチします。キャッチされたすべての例外は、[例外ハンドラ](errors.md#handling-exceptions)に送られますが、リクエストの処理は継続されます。

    return rescue(function () {
        return $this->method();
    });

`rescue`関数に2番目の引数を渡すこともできます。この引数は、クロージャの実行中に例外が発生した場合に返される「デフォルト」値となります。

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

`rescue`関数に`report`引数を指定して、例外を`report`関数で報告するかどうかを決定することもできます。

    return rescue(function () {
        return $this->method();
    }, report: function (Throwable $throwable) {
        return $throwable instanceof InvalidArgumentException;
    });

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

`resolve`関数は、指定されたクラスまたはインターフェース名を[サービスコンテナ](container.md)を使用してインスタンスに解決します。

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {.collection-method}

`response`関数は、[レスポンス](responses.md)インスタンスを作成するか、レスポンスファクトリのインスタンスを取得します。

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {.collection-method}

`retry`関数は、指定された最大試行回数に達するまで、指定されたコールバックを実行しようとします。コールバックが例外をスローせずに実行された場合、その戻り値が返されます。コールバックが例外をスローした場合、自動的に再試行されます。最大試行回数を超えた場合、例外がスローされます。

    return retry(5, function () {
        // 試行間に100msの間隔を置いて5回試行する...
    }, 100);

試行間のスリープ時間を手動で計算したい場合は、`retry`関数の3番目の引数としてクロージャを渡すことができます。

    use Exception;

    return retry(5, function () {
        // ...
    }, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    });

便宜上、`retry`関数の最初の引数として配列を渡すことができます。この配列は、後続の試行間のスリープ時間を決定するために使用されます。

    return retry([100, 200], function () {
        // 最初の再試行で100ms、2回目の再試行で200msスリープする...
    });

特定の条件でのみ再試行したい場合は、`retry`関数の4番目の引数としてクロージャを渡すことができます。

    use Exception;

    return retry(5, function () {
        // ...
    }, 100, function (Exception $exception) {
        return $exception instanceof RetryException;
    });

<a name="method-session"></a>
#### `session()` {.collection-method}

`session`関数は、[セッション](session.md)の値を取得または設定するために使用できます。

    $value = session('key');

キーと値のペアの配列を関数に渡すことで、値を設定できます。

    session(['chairs' => 7, 'instruments' => 3]);

値が渡されない場合、セッションストアが返されます。

    $value = session()->get('key');

    session()->put('key', $value);

<a name="method-tap"></a>
#### `tap()` {.collection-method}

`tap`関数は、任意の`$value`とクロージャの2つの引数を受け取ります。`$value`はクロージャに渡され、その後`tap`関数によって返されます。クロージャの戻り値は無関係です。

    $user = tap(User::first(), function (User $user) {
        $user->name = 'taylor';

        $user->save();
    });

クロージャが`tap`関数に渡されない場合、指定された`$value`に対して任意のメソッドを呼び出すことができます。呼び出すメソッドの戻り値は常に`$value`になり、メソッドが実際に定義されている戻り値に関係なく、`$value`になります。例えば、Eloquentの`update`メソッドは通常整数を返しますが、`tap`関数を通じて`update`メソッドの呼び出しを連鎖させることで、メソッドがモデル自体を返すように強制できます。

    $user = tap($user)->update([
        'name' => $name,
        'email' => $email,
    ]);

クラスに`tap`メソッドを追加するには、クラスに`Illuminate\Support\Traits\Tappable`トレイトを追加できます。このトレイトの`tap`メソッドは、唯一の引数としてクロージャを受け取ります。オブジェクトインスタンス自体がクロージャに渡され、その後`tap`メソッドによって返されます。

    return $user->tap(function (User $user) {
        // ...
    });

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

`throw_if`関数は、指定されたブール式が`true`と評価された場合に指定された例外をスローします。

    throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

`throw_unless`関数は、指定されたブール式が`false`と評価された場合に指定された例外をスローします。

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        'You are not allowed to access this page.'
    );

<a name="method-today"></a>
#### `today()` {.collection-method}

`today`関数は、現在の日付のための新しい`Illuminate\Support\Carbon`インスタンスを作成します。

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

`trait_uses_recursive`関数は、トレイトによって使用されるすべてのトレイトを返します。

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform`関数は、指定された値が[空白](#method-blank)でない場合にクロージャを実行し、クロージャの戻り値を返します。

    $callback = function (int $value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

関数の3番目の引数としてデフォルト値またはクロージャを渡すことができます。指定された値が空白の場合、この値が返されます。

    $result = transform(null, $callback, 'The value is blank');

    // The value is blank

<a name="method-validator"></a>
#### `validator()` {.collection-method}

`validator`関数は、指定された引数を使用して新しい[バリデータ](validation.md)インスタンスを作成します。`Validator`ファサードの代替として使用できます。

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {.collection-method}

`value`関数は、指定された値を返します。ただし、クロージャを関数に渡すと、クロージャが実行され、その戻り値が返されます。

    $result = value(true);

    // true

    $result = value(function () {
        return false;
    });

    // false

追加の引数を`value`関数に渡すことができます。最初の引数がクロージャの場合、追加のパラメータはクロージャの引数として渡されます。それ以外の場合、追加のパラメータは無視されます。

    $result = value(function (string $name) {
        return $name;
    }, 'Taylor');

    // 'Taylor'

<a name="method-view"></a>
#### `view()` {.collection-method}

`view`関数は、[ビュー](views.md)インスタンスを取得します。

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {.collection-method}

`with`関数は、指定された値を返します。2番目の引数としてクロージャが渡された場合、クロージャが実行され、その戻り値が返されます。

    $callback = function (mixed $value) {
        return is_numeric($value) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5

<a name="method-when"></a>
#### `when()` {.collection-method}

`when`関数は、指定された条件が`true`と評価された場合に指定された値を返します。それ以外の場合、`null`が返されます。2番目の引数としてクロージャが渡された場合、クロージャが実行され、その戻り値が返されます。

    $value = when(true, 'Hello World');

    $value = when(true, fn () => 'Hello World');

`when`関数は、主にHTML属性を条件付きでレンダリングするために便利です。

```blade
<div {{ when($condition, 'wire:poll="calculate"') }}>
    ...
</div>
```

<a name="other-utilities"></a>
## その他のユーティリティ

<a name="benchmarking"></a>
### ベンチマーク

アプリケーションの特定の部分のパフォーマンスをすばやくテストしたい場合があります。そのような場合、`Benchmark`サポートクラスを使用して、指定されたコールバックが完了するまでのミリ秒数を測定できます。

    <?php

    use App\Models\User;
    use Illuminate\Support\Benchmark;

    Benchmark::dd(fn () => User::find(1)); // 0.1 ms

    Benchmark::dd([
        'Scenario 1' => fn () => User::count(), // 0.5 ms
        'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
    ]);

デフォルトでは、指定されたコールバックは1回（1回の反復）実行され、その期間がブラウザ/コンソールに表示されます。

コールバックを複数回呼び出すには、コールバックを呼び出すべき反復回数をメソッドの2番目の引数として指定できます。コールバックを複数回実行する場合、`Benchmark`クラスはすべての反復でコールバックを実行するのにかかった平均ミリ秒数を返します。

    Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms

時には、コールバックの実行をベンチマークしながら、コールバックから返される値も取得したい場合があります。`value`メソッドは、コールバックから返された値とコールバックの実行にかかったミリ秒数を含むタプルを返します：

    [$count, $duration] = Benchmark::value(fn () => User::count());

<a name="dates"></a>
### 日付

Laravelには、強力な日付と時刻の操作ライブラリである[Carbon](https://carbon.nesbot.com/docs/)が含まれています。新しい`Carbon`インスタンスを作成するには、`now`関数を呼び出すことができます。この関数は、Laravelアプリケーション内でグローバルに利用可能です：

```php
$now = now();
```

または、`Illuminate\Support\Carbon`クラスを使用して新しい`Carbon`インスタンスを作成することもできます：

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

Carbonとその機能について詳しく知りたい場合は、[公式のCarbonドキュメント](https://carbon.nesbot.com/docs/)を参照してください。

<a name="deferred-functions"></a>
### 遅延関数

> WARNING:
> 遅延関数は現在ベータ版であり、コミュニティからのフィードバックを収集しています。

Laravelの[キューされたジョブ](queues.md)は、タスクをバックグラウンド処理のためにキューに入れることができますが、時には長時間実行されるキューワーカーを設定または管理せずに、単純なタスクを遅延させたい場合があります。

遅延関数を使用すると、HTTPレスポンスがユーザーに送信された後にクロージャの実行を遅延させることができ、アプリケーションの感覚を高速かつ応答性の高いものに保つことができます。クロージャの実行を遅延させるには、単にクロージャを`defer`関数に渡します：

```php
use App\Services\Metrics;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

Route::post('/orders', function (Request $request) {
    // 注文を作成...

    defer(fn () => Metrics::reportOrder($order));

    return $order;
});
```

デフォルトでは、遅延関数は、`defer`が呼び出されたHTTPレスポンス、Artisanコマンド、またはキューされたジョブが正常に完了した場合にのみ実行されます。これは、リクエストが`4xx`または`5xx`のHTTPレスポンスを返した場合、遅延関数は実行されないことを意味します。遅延関数を常に実行させたい場合は、遅延関数に`always`メソッドをチェーンすることができます：

```php
defer(fn () => Metrics::reportOrder($order))->always();
```

<a name="deferred-function-compatibility"></a>
#### 遅延関数の互換性

Laravel 10.xからLaravel 11.xにアップグレードし、アプリケーションのスケルトンにまだ`app/Http/Kernel.php`ファイルが含まれている場合、カーネルの`$middleware`プロパティの先頭に`InvokeDeferredCallbacks`ミドルウェアを追加する必要があります：

```php
protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class, // [tl! add]
    \App\Http\Middleware\TrustProxies::class,
    // ...
];
```

<a name="lottery"></a>
### 抽選

Laravelの抽選クラスは、与えられたオッズに基づいてコールバックを実行するために使用できます。これは、受信リクエストの一部に対してのみコードを実行したい場合に特に便利です：

    use Illuminate\Support\Lottery;

    Lottery::odds(1, 20)
        ->winner(fn () => $user->won())
        ->loser(fn () => $user->lost())
        ->choose();

Laravelの抽選クラスを他のLaravel機能と組み合わせることができます。例えば、例外ハンドラに遅いクエリの一部の割合のみを報告したい場合があります。また、抽選クラスは呼び出し可能であるため、呼び出し可能なものを受け入れるメソッドにクラスのインスタンスを渡すことができます：

    use Carbon\CarbonInterval;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Lottery;

    DB::whenQueryingForLongerThan(
        CarbonInterval::seconds(2),
        Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
    );

<a name="testing-lotteries"></a>
#### 抽選のテスト

Laravelは、アプリケーションの抽選呼び出しを簡単にテストできるようにするためのいくつかの簡単なメソッドを提供しています：

    // 抽選は常に勝つ...
    Lottery::alwaysWin();

    // 抽選は常に負ける...
    Lottery::alwaysLose();

    // 抽選は勝って負けて、最後に通常の動作に戻る...
    Lottery::fix([true, false]);

    // 抽選は通常の動作に戻る...
    Lottery::determineResultsNormally();

<a name="pipeline"></a>
### パイプライン

Laravelの`Pipeline`ファサードは、指定された入力を一連の呼び出し可能なクラス、クロージャ、または呼び出し可能なものを通して「パイプ」する便利な方法を提供し、各クラスに入力を検査または変更し、パイプライン内の次の呼び出し可能なものを呼び出す機会を与えます：

```php
use Closure;
use App\Models\User;
use Illuminate\Support\Facades\Pipeline;

$user = Pipeline::send($user)
            ->through([
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
                function (User $user, Closure $next) {
                    // ...

                    return $next($user);
                },
            ])
            ->then(fn (User $user) => $user);
```

ご覧のように、パイプライン内の各呼び出し可能なクラスまたはクロージャは、入力と`$next`クロージャを提供されます。`$next`クロージャを呼び出すと、パイプライン内の次の呼び出し可能なものが呼び出されます。お気づきのように、これは[ミドルウェア](middleware.md)と非常によく似ています。

パイプライン内の最後の呼び出し可能なものが`$next`クロージャを呼び出すと、`then`メソッドに提供された呼び出し可能なものが呼び出されます。通常、この呼び出し可能なものは単に指定された入力を返します。

もちろん、前述のように、パイプラインにクロージャを提供することに限定されません。呼び出し可能なクラスを提供することもできます。クラス名が提供された場合、クラスはLaravelの[サービスコンテナ](container.md)を介してインスタンス化され、依存関係を呼び出し可能なクラスに注入できます：

```php
$user = Pipeline::send($user)
            ->through([
                GenerateProfilePhoto::class,
                ActivateSubscription::class,
                SendWelcomeEmail::class,
            ])
            ->then(fn (User $user) => $user);
```

<a name="sleep"></a>
### スリープ

Laravelの`Sleep`クラスは、PHPのネイティブ`sleep`および`usleep`関数の軽量なラッパーであり、テスト可能性を高めながら、時間を扱うための開発者フレンドリーなAPIを公開しています：

    use Illuminate\Support\Sleep;

    $waiting = true;

    while ($waiting) {
        Sleep::for(1)->second();

        $waiting = /* ... */;
    }

`Sleep`クラスは、さまざまな時間単位で動作できるさまざまなメソッドを提供しています：

    // スリープ後に値を返す...
    $result = Sleep::for(1)->second()->then(fn () => 1 + 1);

    // 指定された値がtrueの間スリープする...
    Sleep::for(1)->second()->while(fn () => shouldKeepSleeping());

    // 実行を90秒間一時停止する...
    Sleep::for(1.5)->minutes();

    // 実行を2秒間一時停止する...
    Sleep::for(2)->seconds();

    // 実行を500ミリ秒間一時停止する...
    Sleep::for(500)->milliseconds();

    // 実行を5,000マイクロ秒間一時停止する...
    Sleep::for(5000)->microseconds();

    // 指定された時間まで実行を一時停止する...
    Sleep::until(now()->addMinute());

    // PHPのネイティブ"sleep"関数のエイリアス...
    Sleep::sleep(2);

    // PHPのネイティブ"usleep"関数のエイリアス...
    Sleep::usleep(5000);

時間単位を簡単に組み合わせるには、`and`メソッドを使用できます：

    Sleep::for(1)->second()->and(10)->milliseconds();

<a name="testing-sleep"></a>
#### スリープのテスト

`Sleep`クラスまたはPHPのネイティブスリープ関数を使用するコードをテストする場合、テストは実行を一時停止します。予想されるように、これによりテストスイートが大幅に遅くなります。例えば、以下のコードをテストすると想像してください：

    $waiting = /* ... */;

    $seconds = 1;

    while ($waiting) {
        Sleep::for($seconds++)->seconds();

        $waiting = /* ... */;
    }

通常、このコードをテストするには_少なくとも_1秒かかります。幸いなことに、`Sleep`クラスを使用すると、スリープを「偽造」してテストスイートを高速に保つことができます：

===  "Pest"
```php
it('waits until ready', function () {
    Sleep::fake();

    // ...
});
```

===  "PHPUnit"
```php
public function test_it_waits_until_ready()
{
    Sleep::fake();

    // ...
}
```

`Sleep`クラスを偽造すると、実際の実行一時停止はバイパスされ、テストが大幅に高速になります。

`Sleep`クラスが偽造されると、発生するはずの「スリープ」に対してアサーションを行うことができます。これを説明するために、実行を3回一時停止し、各一時停止が1秒ずつ増加するコードをテストしているとします。`assertSequence`メソッドを使用すると、コードが適切な時間「スリープ」したことをアサートし、テストを高速に保つことができます：

===  "Pest"
```php
it('checks if ready three times', function () {
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

===  "PHPUnit"
```php
public function test_it_checks_if_ready_three_times()
{
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

もちろん、`Sleep`クラスはテスト時に使用できるさまざまな他のアサーションを提供しています：

    use Carbon\CarbonInterval as Duration;
    use Illuminate\Support\Sleep;

    // スリープが3回呼び出されたことをアサート...
    Sleep::assertSleptTimes(3);

    // スリープの期間に対してアサート...
    Sleep::assertSlept(function (Duration $duration): bool {
        return /* ... */;
    }, times: 1);

    // Sleepクラスが呼び出されなかったことをアサート...
    Sleep::assertNeverSlept();

    // Sleepが呼び出された場合でも、実行一時停止が発生しなかったことをアサート...
    Sleep::assertInsomniac();

アプリケーションコード内で模擬のスリープが発生するたびに、何らかのアクションを実行することが有用な場合があります。これを実現するために、`whenFakingSleep`メソッドにコールバックを提供することができます。以下の例では、Laravelの[時間操作ヘルパー](mocking.md#interacting-with-time)を使用して、各スリープの期間だけ時間を即座に進めています。

```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // スリープを偽装する際に時間を進める...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

時間を進めることは一般的な要件であるため、`fake`メソッドは`syncWithCarbon`引数を受け入れ、テスト内でスリープする際にCarbonと同期を保つようにします。

```php
Sleep::fake(syncWithCarbon: true);

$start = now();

Sleep::for(1)->second();

$start->diffForHumans(); // 1秒前
```

Laravelは、実行を一時停止する際に内部で`Sleep`クラスを使用します。例えば、[`retry`](#method-retry)ヘルパーは、スリープする際に`Sleep`クラスを使用します。これにより、そのヘルパーを使用する際のテストのしやすさが向上します。
