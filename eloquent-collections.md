# Eloquent: コレクション

- [イントロダクション](#introduction)
- [利用可能なメソッド](#available-methods)
- [カスタムコレクション](#custom-collections)

<a name="introduction"></a>
## イントロダクション

Eloquentのメソッドで複数のモデルを返すもの（`get`メソッドを介して取得した結果やリレーションを介してアクセスした結果を含む）は、`Illuminate\Database\Eloquent\Collection`クラスのインスタンスを返します。Eloquentコレクションオブジェクトは、Laravelの[基本コレクション](collections.md)を拡張しているため、Eloquentモデルの基となる配列を流暢に操作するための数多くのメソッドを自然に継承しています。これらの便利なメソッドについては、Laravelコレクションのドキュメントを必ず確認してください！

すべてのコレクションはイテレータとしても機能し、単純なPHP配列のようにループ処理できます：

    use App\Models\User;

    $users = User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

しかし、前述のように、コレクションは配列よりもはるかに強力で、直感的なインターフェースを使用して連鎖させることができる様々なmap/reduce操作を提供しています。例えば、非アクティブなモデルをすべて削除し、残りのユーザーの名前を収集することができます：

    $names = User::all()->reject(function (User $user) {
        return $user->active === false;
    })->map(function (User $user) {
        return $user->name;
    });

<a name="eloquent-collection-conversion"></a>
#### Eloquentコレクションの変換

ほとんどのEloquentコレクションメソッドは、新しいEloquentコレクションのインスタンスを返しますが、`collapse`、`flatten`、`flip`、`keys`、`pluck`、`zip`メソッドは[基本コレクション](collections.md)のインスタンスを返します。同様に、`map`操作がEloquentモデルを含まないコレクションを返す場合、それは基本コレクションインスタンスに変換されます。

<a name="available-methods"></a>
## 利用可能なメソッド

すべてのEloquentコレクションは、基本の[Laravelコレクション](collections.md#available-methods)オブジェクトを拡張しています。したがって、基本コレクションクラスによって提供されるすべての強力なメソッドを継承しています。

さらに、`Illuminate\Database\Eloquent\Collection`クラスは、モデルコレクションを管理するのに役立つメソッドのスーパーセットを提供します。ほとんどのメソッドは`Illuminate\Database\Eloquent\Collection`インスタンスを返しますが、`modelKeys`のような一部のメソッドは`Illuminate\Support\Collection`インスタンスを返します。

<style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<div class="collection-method-list" markdown="1">

[append](#method-append)
[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[findOrFail](#method-find-or-fail)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[only](#method-only)
[setVisible](#method-setVisible)
[setHidden](#method-setHidden)
[toQuery](#method-toquery)
[unique](#method-unique)

</div>

<a name="method-append"></a>
#### `append($attributes)` {.collection-method .first-collection-method}

`append`メソッドは、コレクション内のすべてのモデルに対して[追加](eloquent-serialization.md#appending-values-to-json)する属性を指定するために使用できます。このメソッドは、属性の配列または単一の属性を受け入れます。このメソッドは、属性の配列または単一の属性を受け入れます：

    $users->append('team');

    $users->append(['team', 'is_admin']);

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)` {.collection-method}

`contains`メソッドは、指定されたモデルインスタンスがコレクションに含まれているかどうかを判断するために使用できます。このメソッドは、主キーまたはモデルインスタンスを受け入れます。このメソッドは、主キーまたはモデルインスタンスを受け入れます：

    $users->contains(1);

    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)` {.collection-method}

`diff`メソッドは、指定されたコレクションに存在しないすべてのモデルを返します：

    use App\Models\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>
#### `except($keys)` {.collection-method}

`except`メソッドは、指定された主キーを持たないすべてのモデルを返します：

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>
#### `find($key)` {.collection-method}

`find`メソッドは、指定されたキーに一致する主キーを持つモデルを返します。`$key`がモデルインスタンスの場合、`find`はその主キーに一致するモデルを返そうとします。`$key`がモデルインスタンスの場合、`find`は主キーに一致するモデルを返そうとします。`$key`がキーの配列の場合、`find`は指定された配列内の主キーを持つすべてのモデルを返します：

    $users = User::all();

    $user = $users->find(1);

<a name="method-find-or-fail"></a>
#### `findOrFail($key)` {.collection-method}

`findOrFail`メソッドは、指定されたキーに一致する主キーを持つモデルを返すか、一致するモデルがコレクション内に見つからない場合は`Illuminate\Database\Eloquent\ModelNotFoundException`例外をスローします：

    $users = User::all();

    $user = $users->findOrFail(1);

<a name="method-fresh"></a>
#### `fresh($with = [])` {.collection-method}

`fresh`メソッドは、コレクション内の各モデルの新しいインスタンスをデータベースから取得します。さらに、指定されたリレーションは積極的に読み込まれます。さらに、指定されたリレーションは積極的に読み込まれます：

    $users = $users->fresh();

    $users = $users->fresh('comments');

<a name="method-intersect"></a>
#### `intersect($items)` {.collection-method}

`intersect`メソッドは、指定されたコレクションにも存在するすべてのモデルを返します：

    use App\Models\User;

    $users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());

<a name="method-load"></a>
#### `load($relations)` {.collection-method}

`load`メソッドは、コレクション内のすべてのモデルに対して指定されたリレーションを積極的に読み込みます：

    $users->load(['comments', 'posts']);

    $users->load('comments.author');

    $users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);

<a name="method-loadMissing"></a>
#### `loadMissing($relations)` {.collection-method}

`loadMissing`メソッドは、リレーションがまだ読み込まれていない場合に、コレクション内のすべてのモデルに対して指定されたリレーションを積極的に読み込みます：

    $users->loadMissing(['comments', 'posts']);

    $users->loadMissing('comments.author');

    $users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);

<a name="method-modelKeys"></a>
#### `modelKeys()` {.collection-method}

`modelKeys`メソッドは、コレクション内のすべてのモデルの主キーを返します：

    $users->modelKeys();

    // [1, 2, 3, 4, 5]

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)` {.collection-method}

`makeVisible`メソッドは、コレクション内の各モデルで通常「非表示」になっている属性を[表示](eloquent-serialization.md#hiding-attributes-from-json)にします：

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)` {.collection-method}

`makeHidden`メソッドは、コレクション内の各モデルで通常「表示」になっている属性を[非表示](eloquent-serialization.md#hiding-attributes-from-json)にします：

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>
#### `only($keys)` {.collection-method}

`only`メソッドは、指定された主キーを持つすべてのモデルを返します：

    $users = $users->only([1, 2, 3]);

<a name="method-setVisible"></a>
#### `setVisible($attributes)` {.collection-method}

`setVisible`メソッドは、コレクション内の各モデルの表示属性を[一時的に上書き](eloquent-serialization.md#temporarily-modifying-attribute-visibility)します：

    $users = $users->setVisible(['id', 'name']);

<a name="method-setHidden"></a>
#### `setHidden($attributes)` {.collection-method}

`setHidden`メソッドは、コレクション内の各モデルの非表示属性を[一時的に上書き](eloquent-serialization.md#temporarily-modifying-attribute-visibility)します：

    $users = $users->setHidden(['email', 'password', 'remember_token']);

<a name="method-toquery"></a>
#### `toQuery()` {.collection-method}

`toQuery`メソッドは、コレクションモデルの主キーに`whereIn`制約を含むEloquentクエリビルダインスタンスを返します：

    use App\Models\User;

    $users = User::where('status', 'VIP')->get();

    $users->toQuery()->update([
        'status' => 'Administrator',
    ]);

<a name="method-unique"></a>
#### `unique($key = null, $strict = false)` {.collection-method}

`unique`メソッドは、コレクション内のすべての一意のモデルを返します。他のモデルと同じ主キーを持つモデルは削除されます：

    $users = $users->unique();

<a name="custom-collections"></a>
## カスタムコレクション

特定のモデルと対話する際にカスタム`Collection`オブジェクトを使用したい場合は、モデルに`newCollection`メソッドを定義できます：

    <?php

    namespace App\Models;

    use App\Support\UserCollection;
    use Illuminate\Database\Eloquent\Collection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 新しいEloquentコレクションインスタンスを作成します。
         *
         * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
         * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
         */
        public function newCollection(array $models = []): Collection
        {
            return new UserCollection($models);
        }
    }

`newCollection`メソッドを定義すると、Eloquentが通常`Illuminate\Database\Eloquent\Collection`インスタンスを返す場合には、代わりにカスタムコレクションのインスタンスを受け取ることになります。アプリケーション内のすべてのモデルに対してカスタムコレクションを使用したい場合は、アプリケーションのすべてのモデルが継承する基本モデルクラスに`newCollection`メソッドを定義する必要があります。
