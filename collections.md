# コレクション

- [イントロダクション](#introduction)
    - [コレクションの作成](#creating-collections)
    - [コレクションの拡張](#extending-collections)
- [利用可能なメソッド](#available-methods)
- [高階メッセージ](#higher-order-messages)
- [遅延コレクション](#lazy-collections)
    - [イントロダクション](#lazy-collection-introduction)
    - [遅延コレクションの作成](#creating-lazy-collections)
    - [Enumerable契約](#the-enumerable-contract)
    - [遅延コレクションメソッド](#lazy-collection-methods)

<a name="introduction"></a>
## イントロダクション

`Illuminate\Support\Collection`クラスは、配列データを操作するための流暢で便利なラッパーを提供します。たとえば、次のコードを見てください。`collect`ヘルパーを使用して、配列から新しいコレクションインスタンスを作成し、各要素に`strtoupper`関数を実行し、空の要素をすべて削除します。

    $collection = collect(['taylor', 'abigail', null])->map(function (?string $name) {
        return strtoupper($name);
    })->reject(function (string $name) {
        return empty($name);
    });

ご覧のとおり、`Collection`クラスを使用すると、メソッドをチェーンして、基になる配列を流暢にマッピングおよび縮小できます。一般的に、コレクションは不変であり、すべての`Collection`メソッドは完全に新しい`Collection`インスタンスを返します。

<a name="creating-collections"></a>
### コレクションの作成

前述のように、`collect`ヘルパーは、指定された配列の新しい`Illuminate\Support\Collection`インスタンスを返します。したがって、コレクションを作成するのは次のように簡単です。

    $collection = collect([1, 2, 3]);

> NOTE:  
> [Eloquent](eloquent.md)クエリの結果は常に`Collection`インスタンスとして返されます。

<a name="extending-collections"></a>
### コレクションの拡張

コレクションは「マクロ化可能」であり、実行時に`Collection`クラスに追加のメソッドを追加できます。`Illuminate\Support\Collection`クラスの`macro`メソッドは、マクロが呼び出されたときに実行されるクロージャを受け取ります。マクロクロージャは、コレクションの他のメソッドを`$this`を介してアクセスできます。たとえば、次のコードは`Collection`クラスに`toUpper`メソッドを追加します。

    use Illuminate\Support\Collection;
    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function (string $value) {
            return Str::upper($value);
        });
    });

    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

通常、コレクションマクロは[サービスプロバイダ](providers.md)の`boot`メソッドで宣言する必要があります。

<a name="macro-arguments"></a>
#### マクロ引数

必要に応じて、追加の引数を受け取るマクロを定義できます。

    use Illuminate\Support\Collection;
    use Illuminate\Support\Facades\Lang;

    Collection::macro('toLocale', function (string $locale) {
        return $this->map(function (string $value) use ($locale) {
            return Lang::get($value, [], $locale);
        });
    });

    $collection = collect(['first', 'second']);

    $translated = $collection->toLocale('es');

<a name="available-methods"></a>
## 利用可能なメソッド

このコレクションドキュメントの残りの部分では、`Collection`クラスで利用可能な各メソッドについて説明します。これらのメソッドはすべて、基になる配列を流暢に操作するためにチェーンできることを覚えておいてください。さらに、ほとんどのメソッドは新しい`Collection`インスタンスを返し、必要に応じてコレクションの元のコピーを保持できます。

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

<div class="collection-method-list" markdown="1">

[after](#method-after)
[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[before](#method-before)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsOneItem](#method-containsoneitem)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffAssocUsing](#method-diffassocusing)
[diffKeys](#method-diffkeys)
[doesntContain](#method-doesntcontain)
[dot](#method-dot)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[ensure](#method-ensure)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[hasAny](#method-hasany)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectAssoc](#method-intersectAssoc)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[lazy](#method-lazy)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[multiply](#method-multiply)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[percentage](#method-percentage)
[pipe](#method-pipe)
[pipeInto](#method-pipeinto)
[pipeThrough](#method-pipethrough)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[range](#method-range)
[reduce](#method-reduce)
[reduceSpread](#method-reduce-spread)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[select](#method-select)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[skipUntil](#method-skipuntil)
[skipWhile](#method-skipwhile)
[slice](#method-slice)
[sliding](#method-sliding)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortDesc](#method-sortdesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[sortKeysUsing](#method-sortkeysusing)
[splice](#method-splice)
[split](#method-split)
[splitIn](#method-splitin)
[sum](#method-sum)
[take](#method-take)
[takeUntil](#method-takeuntil)
[takeWhile](#method-takewhile)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[undot](#method-undot)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[value](#method-value)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

<a name="method-listing"></a>
## メソッド一覧

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="method-after"></a>
#### `after()` {.collection-method .first-collection-method}

`after`メソッドは、指定されたアイテムの後のアイテムを返します。指定されたアイテムが見つからないか、最後のアイテムである場合は`null`を返します。

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->after(3);

    // 4

    $collection->after(5);

    // null

このメソッドは、指定されたアイテムを「緩い」比較で検索します。つまり、整数値を含む文字列は、同じ値の整数と等しいと見なされます。「厳密な」比較を使用するには、メソッドに`strict`引数を指定できます。

    collect([2, 4, 6, 8])->after('4', strict: true);

    // null

または、最初のアイテムが特定の真理テストに合格するように、独自のクロージャを提供することもできます。

    collect([2, 4, 6, 8])->after(function (int $item, int $key) {
        return $item > 5;
    });

    // 8

<a name="method-all"></a>
#### `all()` {.collection-method}

`all`メソッドは、コレクションによって表される基になる配列を返します。

    collect([1, 2, 3])->all();

    // [1, 2, 3]

<a name="method-average"></a>
#### `average()` {.collection-method}

[`avg`](#method-avg)メソッドのエイリアスです。

<a name="method-avg"></a>
#### `avg()` {.collection-method}

`avg`メソッドは、指定されたキーの[平均値](https://en.wikipedia.org/wiki/Average)を返します。

    $average = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

<a name="method-before"></a>
#### `before()` {.collection-method}

`before`メソッドは、[`after`](#method-after)メソッドとは逆の動作をします。指定されたアイテムの前のアイテムを返します。指定されたアイテムが見つからないか、最初のアイテムである場合は`null`が返されます。

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->before(3);

    // 2

    $collection->before(1);

    // null

    collect([2, 4, 6, 8])->before('4', strict: true);

    // null

    collect([2, 4, 6, 8])->before(function (int $item, int $key) {
        return $item > 5;
    });

    // 4

<a name="method-chunk"></a>
#### `chunk()` {.collection-method}

`chunk`メソッドは、コレクションを指定されたサイズの複数の小さなコレクションに分割します。

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->all();

    // [[1, 2, 3, 4], [5, 6, 7]]

このメソッドは、[Bootstrap](https://getbootstrap.com/docs/5.3/layout/grid/)のようなグリッドシステムを使用する際に、[ビュー](views.md)で特に便利です。例えば、グリッドに表示したい[Eloquent](eloquent.md)モデルのコレクションがあるとします。

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

<a name="method-chunkwhile"></a>
#### `chunkWhile()` {.collection-method}

`chunkWhile`メソッドは、指定されたコールバックの評価に基づいて、コレクションを複数の小さなコレクションに分割します。クロージャに渡される`$chunk`変数を使用して、前の要素を検査することができます。

    $collection = collect(str_split('AABBCCCD'));

    $chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
        return $value === $chunk->last();
    });

    $chunks->all();

    // [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]

<a name="method-collapse"></a>
#### `collapse()` {.collection-method}

`collapse`メソッドは、配列のコレクションを単一のフラットなコレクションに折りたたみます。

    $collection = collect([
        [1, 2, 3],
        [4, 5, 6],
        [7, 8, 9],
    ]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect`メソッドは、現在コレクション内にあるアイテムを持つ新しい`Collection`インスタンスを返します。

    $collectionA = collect([1, 2, 3]);

    $collectionB = $collectionA->collect();

    $collectionB->all();

    // [1, 2, 3]

`collect`メソッドは主に、[遅延コレクション](#lazy-collections)を標準の`Collection`インスタンスに変換する際に便利です。

    $lazyCollection = LazyCollection::make(function () {
        yield 1;
        yield 2;
        yield 3;
    });

    $collection = $lazyCollection->collect();

    $collection::class;

    // 'Illuminate\Support\Collection'

    $collection->all();

    // [1, 2, 3]

> NOTE:  
> `collect`メソッドは、`Enumerable`のインスタンスがあり、非遅延のコレクションインスタンスが必要な場合に特に便利です。`collect()`は`Enumerable`契約の一部であるため、`Collection`インスタンスを取得するために安全に使用できます。

<a name="method-combine"></a>
#### `combine()` {.collection-method}

`combine`メソッドは、コレクションの値をキーとして、別の配列またはコレクションの値と結合します。

    $collection = collect(['name', 'age']);

    $combined = $collection->combine(['George', 29]);

    $combined->all();

    // ['name' => 'George', 'age' => 29]

<a name="method-concat"></a>
#### `concat()` {.collection-method}

`concat`メソッドは、指定された`array`またはコレクションの値を別のコレクションの末尾に追加します。

    $collection = collect(['John Doe']);

    $concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

    $concatenated->all();

    // ['John Doe', 'Jane Doe', 'Johnny Doe']

`concat`メソッドは、元のコレクションに連結されたアイテムの数値キーを再インデックスします。連想コレクションでキーを維持するには、[merge](#method-merge)メソッドを参照してください。

<a name="method-contains"></a>
#### `contains()` {.collection-method}

`contains`メソッドは、コレクションに指定されたアイテムが含まれているかどうかを判断します。クロージャを`contains`メソッドに渡して、指定された真偽テストに一致する要素がコレクション内に存在するかどうかを判断できます。

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->contains(function (int $value, int $key) {
        return $value > 5;
    });

    // false

または、文字列を`contains`メソッドに渡して、コレクションに指定されたアイテムの値が含まれているかどうかを判断できます。

    $collection = collect(['name' => 'Desk', 'price' => 100]);

    $collection->contains('Desk');

    // true

    $collection->contains('New York');

    // false

さらに、キーと値のペアを`contains`メソッドに渡して、指定されたペアがコレクション内に存在するかどうかを判断できます。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->contains('product', 'Bookcase');

    // false

`contains`メソッドは、アイテムの値をチェックする際に「緩い」比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいと見なされます。「厳密な」比較を使用してフィルタリングするには、[`containsStrict`](#method-containsstrict)メソッドを使用してください。

`contains`の逆の動作については、[doesntContain](#method-doesntcontain)メソッドを参照してください。

<a name="method-containsoneitem"></a>
#### `containsOneItem()` {.collection-method}

`containsOneItem`メソッドは、コレクションに1つのアイテムが含まれているかどうかを判断します。

    collect([])->containsOneItem();

    // false

    collect(['1'])->containsOneItem();

    // true

    collect(['1', '2'])->containsOneItem();

    // false

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}

このメソッドは[`contains`](#method-contains)メソッドと同じシグネチャを持ちますが、すべての値は「厳密な」比較を使用して比較されます。

> NOTE:  
> このメソッドの動作は、[Eloquentコレクション](eloquent-collections.md#method-contains)を使用する際に変更されます。

<a name="method-count"></a>
#### `count()` {.collection-method}

`count`メソッドは、コレクション内のアイテムの総数を返します。

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

`countBy`メソッドは、コレクション内の値の出現回数をカウントします。デフォルトでは、このメソッドはすべての要素の出現回数をカウントし、コレクション内の特定の「タイプ」の要素をカウントできます。

    $collection = collect([1, 2, 2, 2, 3]);

    $counted = $collection->countBy();

    $counted->all();

    // [1 => 1, 2 => 3, 3 => 1]

クロージャを`countBy`メソッドに渡して、すべてのアイテムをカスタム値でカウントすることもできます。

    $collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

    $counted = $collection->countBy(function (string $email) {
        return substr(strrchr($email, "@"), 1);
    });

    $counted->all();

    // ['gmail.com' => 2, 'yahoo.com' => 1]

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}

`crossJoin`メソッドは、コレクションの値を指定された配列またはコレクションの間でクロス結合し、すべての可能な順列を持つデカルト積を返します。

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b']);

    $matrix->all();

    /*
        [
            [1, 'a'],
            [1, 'b'],
            [2, 'a'],
            [2, 'b'],
        ]
    */

    $collection = collect([1, 2]);

    $matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

    $matrix->all();

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

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd`メソッドは、コレクションのアイテムをダンプし、スクリプトの実行を終了します。

    $collection = collect(['John Doe', 'Jane Doe']);

    $collection->dd();

    /*
        Collection {
            #items: array:2 [
                0 => "John Doe"
                1 => "Jane Doe"
            ]
        }
    */

スクリプトの実行を停止したくない場合は、代わりに[`dump`](#method-dump)メソッドを使用してください。

<a name="method-diff"></a>
#### `diff()` {.collection-method}

`diff`メソッドは、コレクションを別のコレクションまたはプレーンなPHPの`array`の値に基づいて比較します。このメソッドは、元のコレクションに存在しない指定されたコレクションの値を返します。

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

> NOTE:  
> このメソッドの動作は、[Eloquentコレクション](eloquent-collections.md#method-diff)を使用する際に変更されます。

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

`diffAssoc`メソッドは、コレクションを別のコレクションまたはプレーンなPHPの`array`のキーと値に基づいて比較します。このメソッドは、元のコレクションに存在しない指定されたコレクションのキーと値のペアを返します。

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6,
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6,
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffassocusing"></a>
#### `diffAssocUsing()` {.collection-method}

`diffAssoc`とは異なり、`diffAssocUsing`はインデックスの比較にユーザーが指定したコールバック関数を受け入れます。

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

コールバックは、ゼロより小さい、等しい、またはゼロより大きい整数を返す比較関数でなければなりません。詳細については、`diffAssocUsing`メソッドが内部的に利用しているPHP関数である[`array_diff_uassoc`](https://www.php.net/array_diff_uassoc#refsect1-function.array-diff-uassoc-parameters)のPHPドキュメントを参照してください。

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

`diffKeys`メソッドは、コレクションを別のコレクションまたはプレーンなPHP `array`とそのキーに基づいて比較します。このメソッドは、指定されたコレクションに存在しない元のコレクションのキー/値ペアを返します。

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

<a name="method-doesntcontain"></a>
#### `doesntContain()` {.collection-method}

`doesntContain`メソッドは、コレクションに指定されたアイテムが含まれていないかどうかを判断します。クロージャを`doesntContain`メソッドに渡して、指定された真偽テストに一致する要素がコレクションに存在しないかどうかを判断できます。

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});

// false
```

また、`doesntContain`メソッドに文字列を渡して、コレクションに指定されたアイテム値が含まれていないかどうかを判断することもできます。

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->doesntContain('Table');

// true

$collection->doesntContain('Desk');

// false
```

さらに、`doesntContain`メソッドにキー/値のペアを渡して、指定されたペアがコレクションに存在しないかどうかを判断することもできます。

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->doesntContain('product', 'Bookcase');

// true
```

`doesntContain`メソッドは、アイテム値をチェックする際に「緩い」比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいと見なされます。

<a name="method-dot"></a>
#### `dot()` {.collection-method}

`dot`メソッドは、多次元コレクションを深さを示す「ドット」表記を使用して単一レベルのコレクションに平坦化します。

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$flattened = $collection->dot();

$flattened->all();

// ['products.desk.price' => 100]
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump`メソッドは、コレクションのアイテムをダンプします。

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

コレクションをダンプした後にスクリプトの実行を停止したい場合は、代わりに[`dd`](#method-dd)メソッドを使用してください。

<a name="method-duplicates"></a>
#### `duplicates()` {.collection-method}

`duplicates`メソッドは、コレクションから重複する値を取得して返します。

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [2 => 'a', 4 => 'b']
```

コレクションに配列またはオブジェクトが含まれている場合、重複する値をチェックしたい属性のキーを渡すことができます。

```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

このメソッドは[`duplicates`](#method-duplicates)メソッドと同じシグネチャを持ちますが、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-each"></a>
#### `each()` {.collection-method}

`each`メソッドは、コレクション内のアイテムを反復処理し、各アイテムをクロージャに渡します。

```php
$collection = collect([1, 2, 3, 4]);

$collection->each(function (int $item, int $key) {
    // ...
});
```

アイテムの反復処理を停止したい場合は、クロージャから`false`を返すことができます。

```php
$collection->each(function (int $item, int $key) {
    if (/* condition */) {
        return false;
    }
});
```

<a name="method-eachspread"></a>
#### `eachSpread()` {.collection-method}

`eachSpread`メソッドは、コレクションのアイテムを反復処理し、各ネストされたアイテムの値を指定されたコールバックに渡します。

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

コールバックから`false`を返すことで、アイテムの反復処理を停止できます。

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

<a name="method-ensure"></a>
#### `ensure()` {.collection-method}

`ensure`メソッドは、コレクションのすべての要素が指定された型または型のリストであることを確認するために使用できます。そうでない場合、`UnexpectedValueException`がスローされます。

```php
return $collection->ensure(User::class);

return $collection->ensure([User::class, Customer::class]);
```

プリミティブ型（`string`、`int`、`float`、`bool`、`array`）も指定できます。

```php
return $collection->ensure('int');
```

> WARNING:  
> `ensure`メソッドは、後で異なる型の要素がコレクションに追加されないことを保証するものではありません。

<a name="method-every"></a>
#### `every()` {.collection-method}

`every`メソッドは、コレクションのすべての要素が指定された真偽テストに合格するかどうかを確認するために使用できます。

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});

// false
```

コレクションが空の場合、`every`メソッドはtrueを返します。

```php
$collection = collect([]);

$collection->every(function (int $value, int $key) {
    return $value > 2;
});

// true
```

<a name="method-except"></a>
#### `except()` {.collection-method}

`except`メソッドは、指定されたキーを持つアイテムを除くコレクション内のすべてのアイテムを返します。

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

`except`の逆は、[only](#method-only)メソッドを参照してください。

> NOTE:  
> このメソッドの動作は、[Eloquent Collections](eloquent-collections.md#method-except)を使用する場合に変更されます。

<a name="method-filter"></a>
#### `filter()` {.collection-method}

`filter`メソッドは、指定されたコールバックを使用してコレクションをフィルタリングし、指定された真偽テストに合格したアイテムのみを保持します。

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

コールバックが提供されない場合、コレクションの`false`と等価なすべてのエントリが削除されます。

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

`filter`の逆は、[reject](#method-reject)メソッドを参照してください。

<a name="method-first"></a>
#### `first()` {.collection-method}

`first`メソッドは、指定された真偽テストに合格するコレクション内の最初の要素を返します。

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});

// 3
```

引数なしで`first`メソッドを呼び出して、コレクション内の最初の要素を取得することもできます。コレクションが空の場合、`null`が返されます。

```php
collect([1, 2, 3, 4])->first();

// 1
```

<a name="method-first-or-fail"></a>
#### `firstOrFail()` {.collection-method}

`firstOrFail`メソッドは`first`メソッドと同じですが、結果が見つからない場合、`Illuminate\Support\ItemNotFoundException`例外がスローされます。

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});

// Throws ItemNotFoundException...
```

引数なしで`firstOrFail`メソッドを呼び出して、コレクション内の最初の要素を取得することもできます。コレクションが空の場合、`Illuminate\Support\ItemNotFoundException`例外がスローされます。

```php
collect([])->firstOrFail();

// Throws ItemNotFoundException...
```

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

`firstWhere`メソッドは、指定されたキー/値のペアを持つコレクション内の最初の要素を返します。

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

比較演算子を使用して`firstWhere`メソッドを呼び出すこともできます。

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

[where](#method-where)メソッドと同様に、`firstWhere`メソッドに1つの引数を渡すことができます。この場合、`firstWhere`メソッドは、指定されたアイテムキーの値が「真」である最初のアイテムを返します。

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

`flatMap`メソッドは、各値をクロージャに渡してコレクションを反復処理します。クロージャは値を自由に変更して返すことができ、その結果はコレクションにマージされます。配列は平坦化されます。

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function (array $values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

`flatMap`メソッドは、コレクションを反復処理し、各値を与えられたクロージャに渡します。クロージャはアイテムを自由に変更して返すことができ、それによって変更されたアイテムの新しいコレクションが形成されます。その後、配列は1レベル平坦化されます。

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function (array $values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {.collection-method}

`flatten`メソッドは、多次元コレクションを1次元に平坦化します。

    $collection = collect([
        'name' => 'taylor',
        'languages' => [
            'php', 'javascript'
        ]
    ]);

    $flattened = $collection->flatten();

    $flattened->all();

    // ['taylor', 'php', 'javascript'];

必要に応じて、`flatten`メソッドに「深さ」引数を渡すことができます。

    $collection = collect([
        'Apple' => [
            [
                'name' => 'iPhone 6S',
                'brand' => 'Apple'
            ],
        ],
        'Samsung' => [
            [
                'name' => 'Galaxy S7',
                'brand' => 'Samsung'
            ],
        ],
    ]);

    $products = $collection->flatten(1);

    $products->values()->all();

    /*
        [
            ['name' => 'iPhone 6S', 'brand' => 'Apple'],
            ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
        ]
    */

この例では、深さを指定せずに`flatten`を呼び出すと、ネストされた配列も平坦化され、`['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`となります。深さを指定することで、ネストされた配列が平坦化されるレベルを指定できます。

<a name="method-flip"></a>
#### `flip()` {.collection-method}

`flip`メソッドは、コレクションのキーとそれに対応する値を入れ替えます。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $flipped = $collection->flip();

    $flipped->all();

    // ['taylor' => 'name', 'laravel' => 'framework']

<a name="method-forget"></a>
#### `forget()` {.collection-method}

`forget`メソッドは、キーによってコレクションからアイテムを削除します。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    // 単一のキーを忘れる...
    $collection->forget('name');

    // ['framework' => 'laravel']

    // 複数のキーを忘れる...
    $collection->forget(['name', 'framework']);

    // []

> WARNING:  
> 他のほとんどのコレクションメソッドとは異なり、`forget`は新しい変更されたコレクションを返しません。呼び出されたコレクションを変更します。

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

`forPage`メソッドは、指定されたページ番号に存在するアイテムを含む新しいコレクションを返します。このメソッドは、ページ番号を最初の引数として受け取り、1ページあたりに表示するアイテム数を2番目の引数として受け取ります。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunk = $collection->forPage(2, 3);

    $chunk->all();

    // [4, 5, 6]

<a name="method-get"></a>
#### `get()` {.collection-method}

`get`メソッドは、指定されたキーのアイテムを返します。キーが存在しない場合、`null`が返されます。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('name');

    // taylor

オプションで、2番目の引数としてデフォルト値を渡すことができます。

    $collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

    $value = $collection->get('age', 34);

    // 34

メソッドのデフォルト値としてクロージャを渡すこともできます。指定されたキーが存在しない場合、クロージャの結果が返されます。

    $collection->get('email', function () {
        return 'taylor@example.com';
    });

    // taylor@example.com

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

`groupBy`メソッドは、指定されたキーによってコレクションのアイテムをグループ化します。

    $collection = collect([
        ['account_id' => 'account-x10', 'product' => 'Chair'],
        ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ['account_id' => 'account-x11', 'product' => 'Desk'],
    ]);

    $grouped = $collection->groupBy('account_id');

    $grouped->all();

    /*
        [
            'account-x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'account-x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

文字列の`key`を渡す代わりに、コールバックを渡すことができます。コールバックは、グループ化するキーとして返す値を返すべきです。

    $grouped = $collection->groupBy(function (array $item, int $key) {
        return substr($item['account_id'], -3);
    });

    $grouped->all();

    /*
        [
            'x10' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
            'x11' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ]
    */

複数のグループ化基準を配列として渡すことができます。各配列要素は、多次元配列内の対応するレベルに適用されます。

    $data = new Collection([
        10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
    ]);

    $result = $data->groupBy(['skill', function (array $item) {
        return $item['roles'];
    }], preserveKeys: true);

    /*
    [
        1 => [
            'Role_1' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_2' => [
                20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
            ],
            'Role_3' => [
                10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            ],
        ],
        2 => [
            'Role_1' => [
                30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
            ],
            'Role_2' => [
                40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
            ],
        ],
    ];
    */

<a name="method-has"></a>
#### `has()` {.collection-method}

`has`メソッドは、指定されたキーがコレクション内に存在するかどうかを判断します。

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->has('product');

    // true

    $collection->has(['product', 'amount']);

    // true

    $collection->has(['amount', 'price']);

    // false

<a name="method-hasany"></a>
#### `hasAny()` {.collection-method}

`hasAny`メソッドは、指定されたキーのいずれかがコレクション内に存在するかどうかを判断します。

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->hasAny(['product', 'price']);

    // true

    $collection->hasAny(['name', 'price']);

    // false

<a name="method-implode"></a>
#### `implode()` {.collection-method}

`implode`メソッドは、コレクション内のアイテムを結合します。その引数は、コレクション内のアイテムのタイプに依存します。コレクションに配列またはオブジェクトが含まれている場合、結合したい属性のキーと、値の間に配置したい「接着剤」文字列を渡す必要があります。

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

コレクションに単純な文字列または数値が含まれている場合、メソッドに「接着剤」を唯一の引数として渡すだけです。

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

結合される値をフォーマットしたい場合は、`implode`メソッドにクロージャを渡すことができます。

    $collection->implode(function (array $item, int $key) {
        return strtoupper($item['product']);
    }, ', ');

    // DESK, CHAIR

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

`intersect`メソッドは、元のコレクションから指定された`array`またはコレクションに存在しない値を削除します。結果のコレクションは、元のコレクションのキーを保持します。

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

> NOTE:  
> このメソッドの動作は、[Eloquent Collections](eloquent-collections.md#method-intersect)を使用する場合に変更されます。

<a name="method-intersectAssoc"></a>
#### `intersectAssoc()` {.collection-method}

`intersectAssoc`メソッドは、元のコレクションを別のコレクションまたは`array`と比較し、すべてのコレクションに存在するキー/値のペアを返します。

    $collection = collect([
        'color' => 'red',
        'size' => 'M',
        'material' => 'cotton'
    ]);

    $intersect = $collection->intersectAssoc([
        'color' => 'blue',
        'size' => 'M',
        'material' => 'polyester'
    ]);

    $intersect->all();

    // ['size' => 'M']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

`intersectByKeys`メソッドは、元のコレクションから指定された`array`またはコレクションに存在しないキーとそれに対応する値を削除します。

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

`isEmpty`メソッドは、コレクションが空の場合に`true`を返し、そうでない場合は`false`を返します。

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

`isNotEmpty`メソッドは、コレクションが空でない場合に`true`を返し、そうでない場合は`false`を返します。

    collect([])->isNotEmpty();

    // false

<a name="method-join"></a>
#### `join()` {.collection-method}

`join`メソッドは、コレクションの値を文字列で結合します。このメソッドの第二引数を使用して、最後の要素の後に追加する方法を指定することもできます。

    collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
    collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
    collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
    collect(['a'])->join(', ', ' and '); // 'a'
    collect([])->join(', ', ' and '); // ''

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

`keyBy`メソッドは、指定されたキーでコレクションをキー付けします。複数のアイテムが同じキーを持つ場合、新しいコレクションには最後のアイテムのみが表示されます。

    $collection = collect([
        ['product_id' => 'prod-100', 'name' => 'Desk'],
        ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keyed = $collection->keyBy('product_id');

    $keyed->all();

    /*
        [
            'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

メソッドにコールバックを渡すこともできます。コールバックは、コレクションのキーとする値を返す必要があります。

    $keyed = $collection->keyBy(function (array $item, int $key) {
        return strtoupper($item['product_id']);
    });

    $keyed->all();

    /*
        [
            'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
            'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
        ]
    */

<a name="method-keys"></a>
#### `keys()` {.collection-method}

`keys`メソッドは、コレクションのすべてのキーを返します。

    $collection = collect([
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]);

    $keys = $collection->keys();

    $keys->all();

    // ['prod-100', 'prod-200']

<a name="method-last"></a>
#### `last()` {.collection-method}

`last`メソッドは、指定された真偽テストを通過するコレクションの最後の要素を返します。

    collect([1, 2, 3, 4])->last(function (int $value, int $key) {
        return $value < 3;
    });

    // 2

引数なしで`last`メソッドを呼び出して、コレクションの最後の要素を取得することもできます。コレクションが空の場合、`null`が返されます。

    collect([1, 2, 3, 4])->last();

    // 4

<a name="method-lazy"></a>
#### `lazy()` {.collection-method}

`lazy`メソッドは、基になるアイテムの配列から新しい[`LazyCollection`](#lazy-collections)インスタンスを返します。

    $lazyCollection = collect([1, 2, 3, 4])->lazy();

    $lazyCollection::class;

    // Illuminate\Support\LazyCollection

    $lazyCollection->all();

    // [1, 2, 3, 4]

これは、膨大な数のアイテムを含む`Collection`に対して変換を行う必要がある場合に特に便利です。

    $count = $hugeCollection
        ->lazy()
        ->where('country', 'FR')
        ->where('balance', '>', '100')
        ->count();

コレクションを`LazyCollection`に変換することで、大量の追加メモリを割り当てることを回避できます。元のコレクションは依然としてメモリ内に値を保持しますが、後続のフィルターはそうではありません。したがって、コレクションの結果をフィルタリングする際に、事実上追加のメモリは割り当てられません。

<a name="method-macro"></a>
#### `macro()` {.collection-method}

静的な`macro`メソッドを使用すると、実行時に`Collection`クラスにメソッドを追加できます。詳細については、[コレクションの拡張](#extending-collections)に関するドキュメントを参照してください。

<a name="method-make"></a>
#### `make()` {.collection-method}

静的な`make`メソッドは、新しいコレクションインスタンスを作成します。[コレクションの作成](#creating-collections)セクションを参照してください。

<a name="method-map"></a>
#### `map()` {.collection-method}

`map`メソッドは、コレクション全体を反復処理し、各値を指定されたコールバックに渡します。コールバックはアイテムを自由に変更して返すことができ、変更されたアイテムの新しいコレクションを形成します。

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function (int $item, int $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> WARNING:  
> 他の多くのコレクションメソッドと同様に、`map`は新しいコレクションインスタンスを返します。呼び出されたコレクションを変更しません。元のコレクションを変換したい場合は、[`transform`](#method-transform)メソッドを使用してください。

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

`mapInto()`メソッドは、コレクションを反復処理し、指定されたクラスの新しいインスタンスを作成します。値はコンストラクタに渡されます。

    class Currency
    {
        /**
         * 新しい通貨インスタンスを作成します。
         */
        function __construct(
            public string $code,
        ) {}
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {.collection-method}

`mapSpread`メソッドは、コレクションのアイテムを反復処理し、各ネストされたアイテムの値を指定されたクロージャに渡します。クロージャはアイテムを自由に変更して返すことができ、変更されたアイテムの新しいコレクションを形成します。

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function (int $even, int $odd) {
        return $even + $odd;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

`mapToGroups`メソッドは、指定されたクロージャによってコレクションのアイテムをグループ化します。クロージャは、単一のキー/値ペアを含む連想配列を返す必要があり、グループ化された値の新しいコレクションを形成します。

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function (array $item, int $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->all();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johnny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

`mapWithKeys`メソッドは、コレクション全体を反復処理し、各値を指定されたクロージャに渡します。クロージャは、単一のキー/値ペアを含む連想配列を返す必要があります。

    $collection = collect([
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
    ]);

    $keyed = $collection->mapWithKeys(function (array $item, int $key) {
        return [$item['email'] => $item['name']];
    });

    $keyed->all();

    /*
        [
            'john@example.com' => 'John',
            'jane@example.com' => 'Jane',
        ]
    */

<a name="method-max"></a>
#### `max()` {.collection-method}

`max`メソッドは、指定されたキーの最大値を返します。

    $max = collect([
        ['foo' => 10],
        ['foo' => 20]
    ])->max('foo');

    // 20

    $max = collect([1, 2, 3, 4, 5])->max();

    // 5

<a name="method-median"></a>
#### `median()` {.collection-method}

`median`メソッドは、指定されたキーの[中央値](https://en.wikipedia.org/wiki/Median)を返します。

    $median = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->median('foo');

    // 15

    $median = collect([1, 1, 2, 4])->median();

    // 1.5

<a name="method-merge"></a>
#### `merge()` {.collection-method}

`merge`メソッドは、指定された配列またはコレクションを元のコレクションとマージします。指定されたアイテムの文字列キーが元のコレクションの文字列キーと一致する場合、指定されたアイテムの値は元のコレクションの値を上書きします。

    $collection = collect(['product_id' => 1, 'price' => 100]);

    $merged = $collection->merge(['price' => 200, 'discount' => false]);

    $merged->all();

    // ['product_id' => 1, 'price' => 200, 'discount' => false]

指定されたアイテムのキーが数値の場合、値はコレクションの末尾に追加されます。

    $collection = collect(['Desk', 'Chair']);

    $merged = $collection->merge(['Bookcase', 'Door']);

    $merged->all();

    // ['Desk', 'Chair', 'Bookcase', 'Door']

<a name="method-mergerecursive"></a>
#### `mergeRecursive()` {.collection-method}

`mergeRecursive`メソッドは、指定された配列またはコレクションを元のコレクションと再帰的にマージします。指定されたアイテムの文字列キーが元のコレクションの文字列キーと一致する場合、これらのキーの値は配列にマージされ、これは再帰的に行われます。

    $collection = collect(['product_id' => 1, 'price' => 100]);

```

```php
$merged = $collection->mergeRecursive([
    'product_id' => 2,
    'price' => 200,
    'discount' => false
]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```

<a name="method-min"></a>
#### `min()` {.collection-method}

`min`メソッドは、指定されたキーの最小値を返します。

```php
$min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

// 10

$min = collect([1, 2, 3, 4, 5])->min();

// 1
```

<a name="method-mode"></a>
#### `mode()` {.collection-method}

`mode`メソッドは、指定されたキーの[最頻値](https://en.wikipedia.org/wiki/Mode_(statistics))を返します。

```php
$mode = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->mode('foo');

// [10]

$mode = collect([1, 1, 2, 4])->mode();

// [1]

$mode = collect([1, 1, 2, 2])->mode();

// [1, 2]
```

<a name="method-multiply"></a>
#### `multiply()` {.collection-method}

`multiply`メソッドは、コレクション内のすべてのアイテムを指定された数だけコピーします。

```php
$users = collect([
    ['name' => 'User #1', 'email' => 'user1@example.com'],
    ['name' => 'User #2', 'email' => 'user2@example.com'],
])->multiply(3);

/*
    [
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
        ['name' => 'User #1', 'email' => 'user1@example.com'],
        ['name' => 'User #2', 'email' => 'user2@example.com'],
    ]
*/
```

<a name="method-nth"></a>
#### `nth()` {.collection-method}

`nth`メソッドは、n番目ごとの要素で構成される新しいコレクションを作成します。

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4);

// ['a', 'e']
```

オプションで、開始オフセットを2番目の引数として渡すことができます。

```php
$collection->nth(4, 1);

// ['b', 'f']
```

<a name="method-only"></a>
#### `only()` {.collection-method}

`only`メソッドは、指定されたキーを持つコレクション内のアイテムを返します。

```php
$collection = collect([
    'product_id' => 1,
    'name' => 'Desk',
    'price' => 100,
    'discount' => false
]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

`only`の逆は、[except](#method-except)メソッドを参照してください。

> NOTE:  
> このメソッドの動作は、[Eloquent Collections](eloquent-collections.md#method-only)を使用する場合に変更されます。

<a name="method-pad"></a>
#### `pad()` {.collection-method}

`pad`メソッドは、配列が指定されたサイズに達するまで、指定された値で配列を埋めます。このメソッドは、[array_pad](https://secure.php.net/manual/en/function.array-pad.php) PHP関数のように動作します。

左側にパディングするには、負のサイズを指定する必要があります。指定されたサイズの絶対値が配列の長さ以下の場合、パディングは行われません。

```php
$collection = collect(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();

// [0, 0, 'A', 'B', 'C']
```

<a name="method-partition"></a>
#### `partition()` {.collection-method}

`partition`メソッドは、PHPの配列の分割と組み合わせて使用し、指定された真偽テストに合格する要素とそうでない要素を分離できます。

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

[$underThree, $equalOrAboveThree] = $collection->partition(function (int $i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

<a name="method-percentage"></a>
#### `percentage()` {.collection-method}

`percentage`メソッドは、指定された真偽テストに合格するコレクション内のアイテムの割合を素早く決定するために使用できます。

```php
$collection = collect([1, 1, 2, 2, 2, 3]);

$percentage = $collection->percentage(fn ($value) => $value === 1);

// 33.33
```

デフォルトでは、割合は小数点以下2桁に丸められます。ただし、この動作はメソッドに2番目の引数を渡すことでカスタマイズできます。

```php
$percentage = $collection->percentage(fn ($value) => $value === 1, precision: 3);

// 33.333
```

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

`pipe`メソッドは、コレクションを指定されたクロージャに渡し、クロージャの実行結果を返します。

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function (Collection $collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pipeinto"></a>
#### `pipeInto()` {.collection-method}

`pipeInto`メソッドは、指定されたクラスの新しいインスタンスを作成し、コレクションをコンストラクタに渡します。

```php
class ResourceCollection
{
    /**
     * Create a new ResourceCollection instance.
     */
    public function __construct(
        public Collection $collection,
    ) {}
}

$collection = collect([1, 2, 3]);

$resource = $collection->pipeInto(ResourceCollection::class);

$resource->collection->all();

// [1, 2, 3]
```

<a name="method-pipethrough"></a>
#### `pipeThrough()` {.collection-method}

`pipeThrough`メソッドは、コレクションを指定されたクロージャの配列に渡し、クロージャの実行結果を返します。

```php
use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);

$result = $collection->pipeThrough([
    function (Collection $collection) {
        return $collection->merge([4, 5]);
    },
    function (Collection $collection) {
        return $collection->sum();
    },
]);

// 15
```

<a name="method-pluck"></a>
#### `pluck()` {.collection-method}

`pluck`メソッドは、指定されたキーのすべての値を取得します。

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']
```

結果のコレクションをどのようにキー付けするかを指定することもできます。

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

`pluck`メソッドは、"ドット"記法を使用してネストされた値を取得することもサポートしています。

```php
$collection = collect([
    [
        'name' => 'Laracon',
        'speakers' => [
            'first_day' => ['Rosa', 'Judith'],
        ],
    ],
    [
        'name' => 'VueConf',
        'speakers' => [
            'first_day' => ['Abigail', 'Joey'],
        ],
    ],
]);

$plucked = $collection->pluck('speakers.first_day');

$plucked->all();

// [['Rosa', 'Judith'], ['Abigail', 'Joey']]
```

もし重複するキーが存在する場合、最後に一致した要素が取得されたコレクションに挿入されます。

```php
$collection = collect([
    ['brand' => 'Tesla',  'color' => 'red'],
    ['brand' => 'Pagani', 'color' => 'white'],
    ['brand' => 'Tesla',  'color' => 'black'],
    ['brand' => 'Pagani', 'color' => 'orange'],
]);

$plucked = $collection->pluck('color', 'brand');

$plucked->all();

// ['Tesla' => 'black', 'Pagani' => 'orange']
```

<a name="method-pop"></a>
#### `pop()` {.collection-method}

`pop`メソッドは、コレクションの最後のアイテムを削除して返します。

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

`pop`メソッドに整数を渡すことで、コレクションの末尾から複数のアイテムを削除して返すことができます。

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop(3);

// collect([5, 4, 3])

$collection->all();

// [1, 2]
```

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

`prepend`メソッドは、コレクションの先頭にアイテムを追加します。

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4, 5]
```

また、先頭に追加するアイテムのキーを指定するために2番目の引数を渡すこともできます。

```php
$collection = collect(['one' => 1, 'two' => 2]);

$collection->prepend(0, 'zero');

$collection->all();

// ['zero' => 0, 'one' => 1, 'two' => 2]
```

<a name="method-pull"></a>
#### `pull()` {.collection-method}

`pull`メソッドは、キーによってコレクションからアイテムを削除して返します。

```php
$collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

$collection->pull('name');

// 'Desk'

$collection->all();

// ['product_id' => 'prod-100']
```

<a name="method-push"></a>
#### `push()` {.collection-method}

`push`メソッドは、コレクションの末尾にアイテムを追加します。

```php
$collection = collect([1, 2, 3, 4]);

$collection->push(5);

$collection->all();

// [1, 2, 3, 4, 5]
```

<a name="method-put"></a>
#### `put()` {.collection-method}

`put`メソッドは、指定されたキーと値をコレクションに設定します。

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk']);

$collection->put('price', 100);

$collection->all();

// ['product_id' => 1, 'name' => 'Desk', 'price' => 100]
```

<a name="method-random"></a>
#### `random()` {.collection-method}

`random`メソッドは、コレクションからランダムにアイテムを返します。

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (ランダムに取得)
```

`random`メソッドに整数を渡すことで、ランダムに複数のアイテムを取得することができます。常にアイテムのコレクションが返されます。

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (ランダムに取得)
```

もしコレクションインスタンスが要求された数より少ない場合、`random`メソッドは`InvalidArgumentException`をスローします。

`random`メソッドはクロージャも受け付けます。このクロージャは現在のコレクションインスタンスを受け取ります：

    use Illuminate\Support\Collection;

    $random = $collection->random(fn (Collection $items) => min(10, count($items)));

    $random->all();

    // [1, 2, 3, 4, 5] - (ランダムに取得)

<a name="method-range"></a>
#### `range()` {.collection-method}

`range`メソッドは、指定された範囲内の整数を含むコレクションを返します：

    $collection = collect()->range(3, 6);

    $collection->all();

    // [3, 4, 5, 6]

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

`reduce`メソッドは、コレクションを単一の値に縮小し、各反復の結果を後続の反復に渡します：

    $collection = collect([1, 2, 3]);

    $total = $collection->reduce(function (?int $carry, int $item) {
        return $carry + $item;
    });

    // 6

最初の反復での`$carry`の値は`null`ですが、`reduce`に第二引数を渡すことで初期値を指定できます：

    $collection->reduce(function (int $carry, int $item) {
        return $carry + $item;
    }, 4);

    // 10

`reduce`メソッドは、連想コレクションの配列キーも指定されたコールバックに渡します：

    $collection = collect([
        'usd' => 1400,
        'gbp' => 1200,
        'eur' => 1000,
    ]);

    $ratio = [
        'usd' => 1,
        'gbp' => 1.37,
        'eur' => 1.22,
    ];

    $collection->reduce(function (int $carry, int $value, int $key) use ($ratio) {
        return $carry + ($value * $ratio[$key]);
    });

    // 4264

<a name="method-reduce-spread"></a>
#### `reduceSpread()` {.collection-method}

`reduceSpread`メソッドは、コレクションを値の配列に縮小し、各反復の結果を後続の反復に渡します。このメソッドは`reduce`メソッドに似ていますが、複数の初期値を受け入れることができます：

    [$creditsRemaining, $batch] = Image::where('status', 'unprocessed')
        ->get()
        ->reduceSpread(function (int $creditsRemaining, Collection $batch, Image $image) {
            if ($creditsRemaining >= $image->creditsRequired()) {
                $batch->push($image);

                $creditsRemaining -= $image->creditsRequired();
            }

            return [$creditsRemaining, $batch];
        }, $creditsAvailable, collect());

<a name="method-reject"></a>
#### `reject()` {.collection-method}

`reject`メソッドは、指定されたクロージャを使用してコレクションをフィルタリングします。クロージャは、アイテムが結果のコレクションから削除されるべき場合に`true`を返すべきです：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function (int $value, int $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

`reject`メソッドの逆の動作をする場合は、[`filter`](#method-filter)メソッドを参照してください。

<a name="method-replace"></a>
#### `replace()` {.collection-method}

`replace`メソッドは`merge`と同様に動作しますが、文字列キーを持つ一致するアイテムを上書きするだけでなく、一致する数値キーを持つコレクション内のアイテムも上書きします：

    $collection = collect(['Taylor', 'Abigail', 'James']);

    $replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

    $replaced->all();

    // ['Taylor', 'Victoria', 'James', 'Finn']

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

このメソッドは`replace`のように動作しますが、配列に再帰的に入り込み、同じ置換プロセスを内部の値に適用します：

    $collection = collect([
        'Taylor',
        'Abigail',
        [
            'James',
            'Victoria',
            'Finn'
        ]
    ]);

    $replaced = $collection->replaceRecursive([
        'Charlie',
        2 => [1 => 'King']
    ]);

    $replaced->all();

    // ['Charlie', 'Abigail', ['James', 'King', 'Finn']]

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

`reverse`メソッドは、コレクションのアイテムの順序を逆にし、元のキーを保持します：

    $collection = collect(['a', 'b', 'c', 'd', 'e']);

    $reversed = $collection->reverse();

    $reversed->all();

    /*
        [
            4 => 'e',
            3 => 'd',
            2 => 'c',
            1 => 'b',
            0 => 'a',
        ]
    */

<a name="method-search"></a>
#### `search()` {.collection-method}

`search`メソッドは、指定された値をコレクション内で検索し、見つかった場合はそのキーを返します。アイテムが見つからない場合は`false`を返します：

    $collection = collect([2, 4, 6, 8]);

    $collection->search(4);

    // 1

検索は「緩い」比較を使用して行われます。つまり、整数値を持つ文字列は、同じ値の整数と等しいと見なされます。「厳密な」比較を使用するには、メソッドの第二引数に`true`を渡します：

    collect([2, 4, 6, 8])->search('4', strict: true);

    // false

また、指定された真偽テストを通過する最初のアイテムを検索するために、独自のクロージャを提供することもできます：

    collect([2, 4, 6, 8])->search(function (int $item, int $key) {
        return $item > 5;
    });

    // 2

<a name="method-select"></a>
#### `select()` {.collection-method}

`select`メソッドは、SQLの`SELECT`文のように、指定されたキーをコレクションから選択します：

```php
$users = collect([
    ['name' => 'Taylor Otwell', 'role' => 'Developer', 'status' => 'active'],
    ['name' => 'Victoria Faith', 'role' => 'Researcher', 'status' => 'active'],
]);

$users->select(['name', 'role']);

/*
    [
        ['name' => 'Taylor Otwell', 'role' => 'Developer'],
        ['name' => 'Victoria Faith', 'role' => 'Researcher'],
    ],
*/
```

<a name="method-shift"></a>
#### `shift()` {.collection-method}

`shift`メソッドは、コレクションから最初のアイテムを削除して返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift();

    // 1

    $collection->all();

    // [2, 3, 4, 5]

`shift`メソッドに整数を渡すことで、コレクションの先頭から複数のアイテムを削除して返すことができます：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->shift(3);

    // collect([1, 2, 3])

    $collection->all();

    // [4, 5]

<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

`shuffle`メソッドは、コレクション内のアイテムをランダムにシャッフルします：

    $collection = collect([1, 2, 3, 4, 5]);

    $shuffled = $collection->shuffle();

    $shuffled->all();

    // [3, 2, 5, 1, 4] - (ランダムに生成)

<a name="method-skip"></a>
#### `skip()` {.collection-method}

`skip`メソッドは、指定された数の要素をコレクションの先頭から削除した新しいコレクションを返します：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $collection = $collection->skip(4);

    $collection->all();

    // [5, 6, 7, 8, 9, 10]

<a name="method-skipuntil"></a>
#### `skipUntil()` {.collection-method}

`skipUntil`メソッドは、指定されたクロージャが`true`を返すまでコレクションのアイテムをスキップし、残りのアイテムを新しいコレクションインスタンスとして返します：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(function (int $item) {
        return $item >= 3;
    });

    $subset->all();

    // [3, 4]

また、指定された値が見つかるまですべてのアイテムをスキップするために、`skipUntil`メソッドに単純な値を渡すこともできます：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipUntil(3);

    $subset->all();

    // [3, 4]

> WARNING:  
> 指定された値が見つからない場合、またはクロージャが`true`を返さない場合、`skipUntil`メソッドは空のコレクションを返します。

<a name="method-skipwhile"></a>
#### `skipWhile()` {.collection-method}

`skipWhile`メソッドは、指定されたクロージャが`true`を返す間、コレクションのアイテムをスキップし、残りのアイテムを新しいコレクションとして返します：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->skipWhile(function (int $item) {
        return $item <= 3;
    });

    $subset->all();

    // [4]

> WARNING:  
> クロージャが`false`を返さない場合、`skipWhile`メソッドは空のコレクションを返します。

<a name="method-slice"></a>
#### `slice()` {.collection-method}

`slice`メソッドは、指定されたインデックスから始まるコレクションのスライスを返します：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

返されるスライスのサイズを制限したい場合は、メソッドの第二引数に希望のサイズを渡します：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

返されるスライスはデフォルトでキーを保持します。元のキーを保持したくない場合は、[`values`](#method-values)メソッドを使用して再インデックスすることができます。

<a name="method-sliding"></a>
#### `sliding()` {.collection-method}

`sliding`メソッドは、コレクション内のアイテムの「スライディングウィンドウ」ビューを表すチャンクの新しいコレクションを返します：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(2);

    $chunks->toArray();

    // [[1, 2], [2, 3], [3, 4], [4, 5]]

これは特に[`eachSpread`](#method-eachspread)メソッドと組み合わせて使用する場合に便利です：

    $transactions->sliding(2)->eachSpread(function (Collection $previous, Collection $current) {
        $current->total = $previous->total + $current->amount;
    });

オプションで、各チャンクの最初のアイテム間の距離を決定する「ステップ」値を第二引数に渡すこともできます：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(3, step: 2);

    $chunks->toArray();

    // [[1, 2, 3], [3, 4, 5]]

<a name="method-sole"></a>
#### `sole()` {.collection-method}

`sole`メソッドは、指定された真偽テストを通過するコレクション内の最初の要素を返しますが、真偽テストが正確に1つの要素に一致する場合に限ります：

    collect([1, 2, 3, 4])->sole(function (int $value, int $key) {
        return $value === 2;
    });

    // 2

`sole`メソッドにキー/値のペアを渡すこともできます。これにより、指定されたペアに完全に一致する最初の要素が返されますが、一致する要素がちょうど1つの場合に限ります。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

    $collection->sole('product', 'Chair');

    // ['product' => 'Chair', 'price' => 100]

また、引数なしで`sole`メソッドを呼び出して、コレクション内に要素が1つしかない場合に最初の要素を取得することもできます。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
    ]);

    $collection->sole();

    // ['product' => 'Desk', 'price' => 200]

`sole`メソッドによって返されるべき要素がコレクション内に存在しない場合、`\Illuminate\Collections\ItemNotFoundException`例外がスローされます。返されるべき要素が複数存在する場合は、`\Illuminate\Collections\MultipleItemsFoundException`がスローされます。

<a name="method-some"></a>
#### `some()` {.collection-method}

[`contains`](#method-contains)メソッドのエイリアスです。

<a name="method-sort"></a>
#### `sort()` {.collection-method}

`sort`メソッドはコレクションをソートします。ソートされたコレクションは元の配列のキーを保持するため、以下の例では[`values`](#method-values)メソッドを使用してキーを連続した番号のインデックスにリセットしています。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sort();

    $sorted->values()->all();

    // [1, 2, 3, 4, 5]

より高度なソートが必要な場合、独自のアルゴリズムを使用して`sort`にコールバックを渡すことができます。コレクションの`sort`メソッドが内部的に利用している[`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters)に関するPHPドキュメントを参照してください。

> NOTE:  
> ネストされた配列やオブジェクトのコレクションをソートする必要がある場合は、[`sortBy`](#method-sortby)および[`sortByDesc`](#method-sortbydesc)メソッドを参照してください。

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

`sortBy`メソッドは指定されたキーでコレクションをソートします。ソートされたコレクションは元の配列のキーを保持するため、以下の例では[`values`](#method-values)メソッドを使用してキーを連続した番号のインデックスにリセットしています。

    $collection = collect([
        ['name' => 'Desk', 'price' => 200],
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
    ]);

    $sorted = $collection->sortBy('price');

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'price' => 100],
            ['name' => 'Bookcase', 'price' => 150],
            ['name' => 'Desk', 'price' => 200],
        ]
    */

`sortBy`メソッドは、2番目の引数として[ソートフラグ](https://www.php.net/manual/en/function.sort.php)を受け取ります。

    $collection = collect([
        ['title' => 'Item 1'],
        ['title' => 'Item 12'],
        ['title' => 'Item 3'],
    ]);

    $sorted = $collection->sortBy('title', SORT_NATURAL);

    $sorted->values()->all();

    /*
        [
            ['title' => 'Item 1'],
            ['title' => 'Item 3'],
            ['title' => 'Item 12'],
        ]
    */

また、コレクションの値をどのようにソートするかを決定する独自のクロージャを渡すこともできます。

    $collection = collect([
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $sorted = $collection->sortBy(function (array $product, int $key) {
        return count($product['colors']);
    });

    $sorted->values()->all();

    /*
        [
            ['name' => 'Chair', 'colors' => ['Black']],
            ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
            ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
        ]
    */

複数の属性でコレクションをソートしたい場合、`sortBy`メソッドにソート操作の配列を渡すことができます。各ソート操作は、ソートしたい属性と希望するソートの方向を含む配列である必要があります。

    $collection = collect([
        ['name' => 'Taylor Otwell', 'age' => 34],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Abigail Otwell', 'age' => 32],
    ]);

    $sorted = $collection->sortBy([
        ['name', 'asc'],
        ['age', 'desc'],
    ]);

    $sorted->values()->all();

    /*
        [
            ['name' => 'Abigail Otwell', 'age' => 32],
            ['name' => 'Abigail Otwell', 'age' => 30],
            ['name' => 'Taylor Otwell', 'age' => 36],
            ['name' => 'Taylor Otwell', 'age' => 34],
        ]
    */

複数の属性でコレクションをソートする場合、各ソート操作を定義するクロージャを提供することもできます。

    $collection = collect([
        ['name' => 'Taylor Otwell', 'age' => 34],
        ['name' => 'Abigail Otwell', 'age' => 30],
        ['name' => 'Taylor Otwell', 'age' => 36],
        ['name' => 'Abigail Otwell', 'age' => 32],
    ]);

    $sorted = $collection->sortBy([
        fn (array $a, array $b) => $a['name'] <=> $b['name'],
        fn (array $a, array $b) => $b['age'] <=> $a['age'],
    ]);

    $sorted->values()->all();

    /*
        [
            ['name' => 'Abigail Otwell', 'age' => 32],
            ['name' => 'Abigail Otwell', 'age' => 30],
            ['name' => 'Taylor Otwell', 'age' => 36],
            ['name' => 'Taylor Otwell', 'age' => 34],
        ]
    */

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

このメソッドは[`sortBy`](#method-sortby)メソッドと同じシグネチャを持ちますが、コレクションを逆順にソートします。

<a name="method-sortdesc"></a>
#### `sortDesc()` {.collection-method}

このメソッドは[`sort`](#method-sort)メソッドとは逆の順序でコレクションをソートします。

    $collection = collect([5, 3, 1, 2, 4]);

    $sorted = $collection->sortDesc();

    $sorted->values()->all();

    // [5, 4, 3, 2, 1]

`sort`とは異なり、クロージャを`sortDesc`に渡すことはできません。代わりに、[`sort`](#method-sort)メソッドを使用し、比較を反転させる必要があります。

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

`sortKeys`メソッドは、基礎となる連想配列のキーでコレクションをソートします。

    $collection = collect([
        'id' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeys();

    $sorted->all();

    /*
        [
            'first' => 'John',
            'id' => 22345,
            'last' => 'Doe',
        ]
    */

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

このメソッドは[`sortKeys`](#method-sortkeys)メソッドと同じシグネチャを持ちますが、コレクションを逆順にソートします。

<a name="method-sortkeysusing"></a>
#### `sortKeysUsing()` {.collection-method}

`sortKeysUsing`メソッドは、コールバックを使用して基礎となる連想配列のキーでコレクションをソートします。

    $collection = collect([
        'ID' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeysUsing('strnatcasecmp');

    $sorted->all();

    /*
        [
            'first' => 'John',
            'ID' => 22345,
            'last' => 'Doe',
        ]
    */

コールバックは、ゼロ未満、ゼロ、またはゼロより大きい整数を返す比較関数である必要があります。詳細については、`sortKeysUsing`メソッドが内部的に利用しているPHPの[`uksort`](https://www.php.net/manual/en/function.uksort.php#refsect1-function.uksort-parameters)に関するドキュメントを参照してください。

<a name="method-splice"></a>
#### `splice()` {.collection-method}

`splice`メソッドは、指定されたインデックスから始まるアイテムのスライスを削除して返します。

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

結果のコレクションのサイズを制限するために、2番目の引数を渡すことができます。

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

さらに、コレクションから削除されたアイテムを置き換える新しいアイテムを含む3番目の引数を渡すことができます。

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]

<a name="method-split"></a>
#### `split()` {.collection-method}

`split`メソッドは、コレクションを指定された数のグループに分割します。

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->all();

    // [[1, 2], [3, 4], [5]]

<a name="method-splitin"></a>
#### `splitIn()` {.collection-method}

`splitIn`メソッドは、コレクションを指定された数のグループに分割します。最後のグループに残りを割り当てる前に、非終端グループを完全に埋めます。

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $groups = $collection->splitIn(3);

    $groups->all();

    // [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]

<a name="method-sum"></a>
#### `sum()` {.collection-method}

`sum`メソッドは、コレクション内のすべてのアイテムの合計を返します。

    collect([1, 2, 3, 4, 5])->sum();

    // 15

コレクションにネストされた配列やオブジェクトが含まれている場合、合計する値を決定するためにキーを渡す必要があります。

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272
```

さらに、コレクションのどの値を合計するかを決定するために、独自のクロージャを渡すこともできます。

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function (array $product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {.collection-method}

`take`メソッドは、指定された数のアイテムを持つ新しいコレクションを返します。

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

また、負の整数を渡して、コレクションの末尾から指定された数のアイテムを取得することもできます。

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-takeuntil"></a>
#### `takeUntil()` {.collection-method}

`takeUntil`メソッドは、指定されたコールバックが`true`を返すまでコレクション内のアイテムを返します。

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(function (int $item) {
        return $item >= 3;
    });

    $subset->all();

    // [1, 2]

また、`takeUntil`メソッドに単純な値を渡して、指定された値が見つかるまでアイテムを取得することもできます。

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(3);

    $subset->all();

    // [1, 2]

> WARNING:  
> 指定された値が見つからない場合、またはコールバックが`true`を返さない場合、`takeUntil`メソッドはコレクション内のすべてのアイテムを返します。

<a name="method-takewhile"></a>
#### `takeWhile()` {.collection-method}

`takeWhile`メソッドは、指定されたコールバックが`false`を返すまでコレクション内のアイテムを返します。

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeWhile(function (int $item) {
        return $item < 3;
    });

    $subset->all();

    // [1, 2]

> WARNING:  
> コールバックが`false`を返さない場合、`takeWhile`メソッドはコレクション内のすべてのアイテムを返します。

<a name="method-tap"></a>
#### `tap()` {.collection-method}

`tap`メソッドは、指定されたコールバックにコレクションを渡し、コレクションの特定のポイントでアイテムを操作できるようにしますが、コレクション自体には影響を与えません。その後、`tap`メソッドによってコレクションが返されます。

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function (Collection $collection) {
            Log::debug('Values after sorting', $collection->values()->all());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {.collection-method}

静的な`times`メソッドは、指定された回数だけ指定されたクロージャを呼び出すことで、新しいコレクションを作成します。

    $collection = Collection::times(10, function (int $number) {
        return $number * 9;
    });

    $collection->all();

    // [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

`toArray`メソッドは、コレクションをプレーンなPHPの`array`に変換します。コレクションの値が[Eloquent](eloquent.md)モデルの場合、モデルも配列に変換されます。

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toArray();

    /*
        [
            ['name' => 'Desk', 'price' => 200],
        ]
    */

> WARNING:  
> `toArray`は、コレクションのすべてのネストされたオブジェクトを`Arrayable`のインスタンスである場合に配列に変換します。コレクションの基になる生の配列を取得したい場合は、代わりに[`all`](#method-all)メソッドを使用してください。

<a name="method-tojson"></a>
#### `toJson()` {.collection-method}

`toJson`メソッドは、コレクションをJSONシリアライズされた文字列に変換します。

    $collection = collect(['name' => 'Desk', 'price' => 200]);

    $collection->toJson();

    // '{"name":"Desk", "price":200}'

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform`メソッドは、コレクションを反復処理し、指定されたコールバックをコレクション内の各アイテムに対して呼び出します。コレクション内のアイテムは、コールバックによって返される値に置き換えられます。

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->transform(function (int $item, int $key) {
        return $item * 2;
    });

    $collection->all();

    // [2, 4, 6, 8, 10]

> WARNING:  
> 他のほとんどのコレクションメソッドとは異なり、`transform`はコレクション自体を変更します。新しいコレクションを作成したい場合は、代わりに[`map`](#method-map)メソッドを使用してください。

<a name="method-undot"></a>
#### `undot()` {.collection-method}

`undot`メソッドは、"ドット"記法を使用した単一の次元のコレクションを多次元のコレクションに展開します。

    $person = collect([
        'name.first_name' => 'Marie',
        'name.last_name' => 'Valentine',
        'address.line_1' => '2992 Eagle Drive',
        'address.line_2' => '',
        'address.suburb' => 'Detroit',
        'address.state' => 'MI',
        'address.postcode' => '48219'
    ]);

    $person = $person->undot();

    $person->toArray();

    /*
        [
            "name" => [
                "first_name" => "Marie",
                "last_name" => "Valentine",
            ],
            "address" => [
                "line_1" => "2992 Eagle Drive",
                "line_2" => "",
                "suburb" => "Detroit",
                "state" => "MI",
                "postcode" => "48219",
            ],
        ]
    */

<a name="method-union"></a>
#### `union()` {.collection-method}

`union`メソッドは、指定された配列をコレクションに追加します。指定された配列に元のコレクションに既に存在するキーが含まれている場合、元のコレクションの値が優先されます。

    $collection = collect([1 => ['a'], 2 => ['b']]);

    $union = $collection->union([3 => ['c'], 1 => ['d']]);

    $union->all();

    // [1 => ['a'], 2 => ['b'], 3 => ['c']]

<a name="method-unique"></a>
#### `unique()` {.collection-method}

`unique`メソッドは、コレクション内のすべての一意のアイテムを返します。返されるコレクションは元の配列のキーを保持するため、次の例では[`values`](#method-values)メソッドを使用してキーを連続した番号のインデックスにリセットしています。

    $collection = collect([1, 1, 2, 2, 3, 4, 2]);

    $unique = $collection->unique();

    $unique->values()->all();

    // [1, 2, 3, 4]

ネストされた配列やオブジェクトを扱う場合、一意性を決定するために使用するキーを指定できます。

    $collection = collect([
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]);

    $unique = $collection->unique('brand');

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ]
    */

最後に、アイテムの一意性を決定する値を指定するために、独自のクロージャを`unique`メソッドに渡すこともできます。

    $unique = $collection->unique(function (array $item) {
        return $item['brand'].$item['type'];
    });

    $unique->values()->all();

    /*
        [
            ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
            ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
            ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
            ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
        ]
    */

`unique`メソッドは、アイテムの値をチェックする際に"緩い"比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいと見なされます。"厳密な"比較を使用してフィルタリングするには、[`uniqueStrict`](#method-uniquestrict)メソッドを使用してください。

> NOTE:  
> このメソッドの動作は、[Eloquentコレクション](eloquent-collections.md#method-unique)を使用する際に変更されます。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

このメソッドは、[`unique`](#method-unique)メソッドと同じシグネチャを持ちます。ただし、すべての値は"厳密な"比較を使用して比較されます。

<a name="method-unless"></a>
#### `unless()` {.collection-method}

`unless`メソッドは、メソッドに与えられた最初の引数が`true`と評価されない限り、指定されたコールバックを実行します。

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function (Collection $collection) {
        return $collection->push(4);
    });

    $collection->unless(false, function (Collection $collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

`unless`メソッドには、2番目のコールバックを渡すこともできます。2番目のコールバックは、`unless`メソッドに与えられた最初の引数が`true`と評価された場合に実行されます。

    $collection = collect([1, 2, 3]);

    $collection->unless(true, function (Collection $collection) {
        return $collection->push(4);
    }, function (Collection $collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

`unless`の逆の動作については、[`when`](#method-when)メソッドを参照してください。

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

[`whenNotEmpty`](#method-whennotempty)メソッドのエイリアスです。

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

[`whenEmpty`](#method-whenempty)メソッドのエイリアスです。

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

静的な`unwrap`メソッドは、適用可能な場合に、指定された値からコレクションの基になるアイテムを返します。

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-value"></a>
#### `value()` {.collection-method}

`value`メソッドは、コレクションの最初の要素から指定された値を取得します。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Speaker', 'price' => 400],
    ]);

    $value = $collection->value('price');

    // 200

<a name="method-values"></a>
#### `values()` {.collection-method}

`values`メソッドは、キーが連続した整数にリセットされた新しいコレクションを返します。

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200],
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {.collection-method}

`when`メソッドは、メソッドに渡された最初の引数が`true`と評価された場合、指定されたコールバックを実行します。コレクションインスタンスと`when`メソッドに渡された最初の引数がクロージャに提供されます。

    $collection = collect([1, 2, 3]);

    $collection->when(true, function (Collection $collection, int $value) {
        return $collection->push(4);
    });

    $collection->when(false, function (Collection $collection, int $value) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 4]

`when`メソッドには、2番目のコールバックを渡すこともできます。この2番目のコールバックは、`when`メソッドに渡された最初の引数が`false`と評価された場合に実行されます。

    $collection = collect([1, 2, 3]);

    $collection->when(false, function (Collection $collection, int $value) {
        return $collection->push(4);
    }, function (Collection $collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

`when`の逆の動作については、[`unless`](#method-unless)メソッドを参照してください。

<a name="method-whenempty"></a>
#### `whenEmpty()` {.collection-method}

`whenEmpty`メソッドは、コレクションが空の場合に指定されたコールバックを実行します。

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Michael', 'Tom']


    $collection = collect();

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Adam']

`whenEmpty`メソッドには、2番目のクロージャを渡すこともできます。この2番目のクロージャは、コレクションが空でない場合に実行されます。

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    }, function (Collection $collection) {
        return $collection->push('Taylor');
    });

    $collection->all();

    // ['Michael', 'Tom', 'Taylor']

`whenEmpty`の逆の動作については、[`whenNotEmpty`](#method-whennotempty)メソッドを参照してください。

<a name="method-whennotempty"></a>
#### `whenNotEmpty()` {.collection-method}

`whenNotEmpty`メソッドは、コレクションが空でない場合に指定されたコールバックを実行します。

    $collection = collect(['michael', 'tom']);

    $collection->whenNotEmpty(function (Collection $collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['michael', 'tom', 'adam']


    $collection = collect();

    $collection->whenNotEmpty(function (Collection $collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // []

`whenNotEmpty`メソッドには、2番目のクロージャを渡すこともできます。この2番目のクロージャは、コレクションが空の場合に実行されます。

    $collection = collect();

    $collection->whenNotEmpty(function (Collection $collection) {
        return $collection->push('adam');
    }, function (Collection $collection) {
        return $collection->push('taylor');
    });

    $collection->all();

    // ['taylor']

`whenNotEmpty`の逆の動作については、[`whenEmpty`](#method-whenempty)メソッドを参照してください。

<a name="method-where"></a>
#### `where()` {.collection-method}

`where`メソッドは、指定されたキー/値のペアでコレクションをフィルタリングします。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->where('price', 100);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`where`メソッドは、アイテムの値をチェックする際に「緩い」比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいとみなされます。「厳密な」比較を使用してフィルタリングするには、[`whereStrict`](#method-wherestrict)メソッドを使用してください。

オプションとして、比較演算子を2番目のパラメータとして渡すことができます。サポートされている演算子は、'===', '!==', '!=', '==', '=', '<>', '>', '<', '>=', '<='です。

    $collection = collect([
        ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
        ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ['name' => 'Sue', 'deleted_at' => null],
    ]);

    $filtered = $collection->where('deleted_at', '!=', null);

    $filtered->all();

    /*
        [
            ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
            ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
        ]
    */

<a name="method-wherestrict"></a>
#### `whereStrict()` {.collection-method}

このメソッドは[`where`](#method-where)メソッドと同じシグネチャを持ちますが、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-wherebetween"></a>
#### `whereBetween()` {.collection-method}

`whereBetween`メソッドは、指定されたアイテムの値が指定された範囲内にあるかどうかを判断して、コレクションをフィルタリングします。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
            ['product' => 'Door', 'price' => 100],
        ]
    */

<a name="method-wherein"></a>
#### `whereIn()` {.collection-method}

`whereIn`メソッドは、指定されたアイテムの値が指定された配列に含まれていない要素をコレクションから削除します。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Desk', 'price' => 200],
            ['product' => 'Bookcase', 'price' => 150],
        ]
    */

`whereIn`メソッドは、アイテムの値をチェックする際に「緩い」比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいとみなされます。「厳密な」比較を使用してフィルタリングするには、[`whereInStrict`](#method-whereinstrict)メソッドを使用してください。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {.collection-method}

このメソッドは[`whereIn`](#method-wherein)メソッドと同じシグネチャを持ちますが、すべての値は「厳密な」比較を使用して比較されます。

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {.collection-method}

`whereInstanceOf`メソッドは、指定されたクラスタイプでコレクションをフィルタリングします。

    use App\Models\User;
    use App\Models\Post;

    $collection = collect([
        new User,
        new User,
        new Post,
    ]);

    $filtered = $collection->whereInstanceOf(User::class);

    $filtered->all();

    // [App\Models\User, App\Models\User]

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()` {.collection-method}

`whereNotBetween`メソッドは、指定されたアイテムの値が指定された範囲外にあるかどうかを判断して、コレクションをフィルタリングします。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 80],
            ['product' => 'Pencil', 'price' => 30],
        ]
    */

<a name="method-wherenotin"></a>
#### `whereNotIn()` {.collection-method}

`whereNotIn`メソッドは、指定されたアイテムの値が指定された配列に含まれている要素をコレクションから削除します。

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotIn('price', [150, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 100],
            ['product' => 'Door', 'price' => 100],
        ]
    */

`whereNotIn`メソッドは、アイテムの値をチェックする際に「緩い」比較を使用します。つまり、整数値を持つ文字列は、同じ値の整数と等しいとみなされます。「厳密な」比較を使用してフィルタリングするには、[`whereNotInStrict`](#method-wherenotinstrict)メソッドを使用してください。

<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {.collection-method}

このメソッドは[`whereNotIn`](#method-wherenotin)メソッドと同じシグネチャを持ちますが、すべての値は「厳密な」比較を使用して比較されます。
```

このメソッドは[`whereNotIn`](#method-wherenotin)メソッドと同じシグネチャを持っています。ただし、すべての値は"strict"比較を使用して比較されます。

<a name="method-wherenotnull"></a>
#### `whereNotNull()` {.collection-method}

`whereNotNull`メソッドは、指定されたキーが`null`でないコレクションのアイテムを返します。

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNotNull('name');

    $filtered->all();

    /*
        [
            ['name' => 'Desk'],
            ['name' => 'Bookcase'],
        ]
    */

<a name="method-wherenull"></a>
#### `whereNull()` {.collection-method}

`whereNull`メソッドは、指定されたキーが`null`であるコレクションのアイテムを返します。

    $collection = collect([
        ['name' => 'Desk'],
        ['name' => null],
        ['name' => 'Bookcase'],
    ]);

    $filtered = $collection->whereNull('name');

    $filtered->all();

    /*
        [
            ['name' => null],
        ]
    */

<a name="method-wrap"></a>
#### `wrap()` {.collection-method}

静的な`wrap`メソッドは、指定された値をコレクションに適用可能な場合にその値をコレクションでラップします。

    use Illuminate\Support\Collection;

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {.collection-method}

`zip`メソッドは、指定された配列の値を元のコレクションの値と対応するインデックスでマージします。

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高階メッセージ

コレクションは、"高階メッセージ"のサポートも提供します。これは、コレクション上で一般的なアクションを実行するためのショートカットです。高階メッセージを提供するコレクションメソッドは以下の通りです：[`average`](#method-average)、[`avg`](#method-avg)、[`contains`](#method-contains)、[`each`](#method-each)、[`every`](#method-every)、[`filter`](#method-filter)、[`first`](#method-first)、[`flatMap`](#method-flatmap)、[`groupBy`](#method-groupby)、[`keyBy`](#method-keyby)、[`map`](#method-map)、[`max`](#method-max)、[`min`](#method-min)、[`partition`](#method-partition)、[`reject`](#method-reject)、[`skipUntil`](#method-skipuntil)、[`skipWhile`](#method-skipwhile)、[`some`](#method-some)、[`sortBy`](#method-sortby)、[`sortByDesc`](#method-sortbydesc)、[`sum`](#method-sum)、[`takeUntil`](#method-takeuntil)、[`takeWhile`](#method-takewhile)、および[`unique`](#method-unique)。

各高階メッセージは、コレクションインスタンス上の動的プロパティとしてアクセスできます。例えば、`each`高階メッセージを使用して、コレクション内の各オブジェクトに対してメソッドを呼び出すことができます。

    use App\Models\User;

    $users = User::where('votes', '>', 500)->get();

    $users->each->markAsVip();

同様に、`sum`高階メッセージを使用して、ユーザーコレクションの"votes"の合計数を集計することができます。

    $users = User::where('group', 'Development')->get();

    return $users->sum->votes;

<a name="lazy-collections"></a>
## 遅延コレクション

<a name="lazy-collection-introduction"></a>
### はじめに

> WARNING:  
> Laravelの遅延コレクションについて学ぶ前に、[PHPのジェネレータ](https://www.php.net/manual/en/language.generators.overview.php)について理解する時間を取ってください。

すでに強力な`Collection`クラスを補完するために、`LazyCollection`クラスはPHPの[ジェネレータ](https://www.php.net/manual/en/language.generators.overview.php)を活用して、非常に大きなデータセットを扱う際にメモリ使用量を低く保つことができます。

例えば、アプリケーションがマルチギガバイトのログファイルを処理し、Laravelのコレクションメソッドを使用してログを解析する必要があると想像してください。一度にファイル全体をメモリに読み込む代わりに、遅延コレクションを使用して、特定の時点でファイルの一部だけをメモリに保持することができます。

    use App\Models\LogEntry;
    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    })->chunk(4)->map(function (array $lines) {
        return LogEntry::fromLines($lines);
    })->each(function (LogEntry $logEntry) {
        // ログエントリを処理する...
    });

また、10,000のEloquentモデルを反復処理する必要があると想像してください。従来のLaravelコレクションを使用する場合、すべての10,000のEloquentモデルを一度にメモリに読み込む必要があります。

    use App\Models\User;

    $users = User::all()->filter(function (User $user) {
        return $user->id > 500;
    });

しかし、クエリビルダの`cursor`メソッドは`LazyCollection`インスタンスを返します。これにより、データベースに対して1つのクエリを実行するだけで、一度に1つのEloquentモデルのみをメモリに読み込むことができます。この例では、`filter`コールバックは実際に各ユーザーを個別に反復処理するまで実行されないため、メモリ使用量が大幅に削減されます。

    use App\Models\User;

    $users = User::cursor()->filter(function (User $user) {
        return $user->id > 500;
    });

    foreach ($users as $user) {
        echo $user->id;
    }

<a name="creating-lazy-collections"></a>
### 遅延コレクションの作成

遅延コレクションインスタンスを作成するには、PHPのジェネレータ関数をコレクションの`make`メソッドに渡す必要があります。

    use Illuminate\Support\LazyCollection;

    LazyCollection::make(function () {
        $handle = fopen('log.txt', 'r');

        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    });

<a name="the-enumerable-contract"></a>
### Enumerable契約

`Collection`クラスで利用可能なほとんどすべてのメソッドは、`LazyCollection`クラスでも利用可能です。これらのクラスはどちらも`Illuminate\Support\Enumerable`契約を実装しており、以下のメソッドを定義しています。

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

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectAssoc](#method-intersectAssoc)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

> WARNING:  
> `shift`、`pop`、`prepend`など、コレクションを変更するメソッドは、`LazyCollection`クラスでは**利用できません**。

<a name="lazy-collection-methods"></a>
### 遅延コレクションメソッド

`Enumerable`契約で定義されたメソッドに加えて、`LazyCollection`クラスには以下のメソッドが含まれています。

<a name="method-takeUntilTimeout"></a>
#### `takeUntilTimeout()` {.collection-method}

`takeUntilTimeout`メソッドは、指定された時間まで値を列挙する新しい遅延コレクションを返します。その時間を過ぎると、コレクションは列挙を停止します。

    $lazyCollection = LazyCollection::times(INF)
        ->takeUntilTimeout(now()->addMinute());

    $lazyCollection->each(function (int $number) {
        dump($number);

        sleep(1);
    });

    // 1
    // 2
    // ...
    // 58
    // 59

このメソッドの使用例を説明するために、カーソルを使用してデータベースから請求書を送信するアプリケーションを想像してみてください。15分ごとに実行され、最大14分間請求書を処理する[スケジュールされたタスク](scheduling.md)を定義できます。

    use App\Models\Invoice;
    use Illuminate\Support\Carbon;

    Invoice::pending()->cursor()
        ->takeUntilTimeout(
            Carbon::createFromTimestamp(LARAVEL_START)->add(14, 'minutes')
        )
        ->each(fn (Invoice $invoice) => $invoice->submit());

<a name="method-tapEach"></a>
#### `tapEach()` {.collection-method}

`each`メソッドはコレクション内の各アイテムに対して即座に指定されたコールバックを呼び出しますが、`tapEach`メソッドはアイテムがリストから1つずつ取り出されるときにのみ指定されたコールバックを呼び出します。

    // まだ何もダンプされていません...
    $lazyCollection = LazyCollection::times(INF)->tapEach(function (int $value) {
        dump($value);
    });

    // 3つのアイテムがダンプされます...
    $array = $lazyCollection->take(3)->all();

    // 1
    // 2
    // 3

<a name="method-throttle"></a>
#### `throttle()` {.collection-method}

`throttle`メソッドは、指定された秒数後に各値が返されるように遅延コレクションをスロットルします。このメソッドは、受信リクエストをレート制限する外部APIとやり取りする場合に特に便利です。

```php
use App\Models\User;

User::where('vip', true)
    ->cursor()
    ->throttle(seconds: 1)
    ->each(function (User $user) {
        // 外部APIを呼び出す...
    });
```

<a name="method-remember"></a>
#### `remember()` {.collection-method}

`remember`メソッドは、すでに列挙された値を記憶し、以降のコレクション列挙時にそれらを再取得しない新しい遅延コレクションを返します。

    // まだクエリは実行されていません...
    $users = User::cursor()->remember();

    // クエリが実行されます...
    // 最初の5人のユーザーがデータベースから取得されます...
    $users->take(5)->all();

    // 最初の5人のユーザーはコレクションのキャッシュから取得されます...
    // 残りはデータベースから取得されます...
    $users->take(20)->all();
