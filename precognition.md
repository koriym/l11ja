# Precognition

- [はじめに](#introduction)
- [ライブバリデーション](#live-validation)
    - [Vueを使用する](#using-vue)
    - [VueとInertiaを使用する](#using-vue-and-inertia)
    - [Reactを使用する](#using-react)
    - [ReactとInertiaを使用する](#using-react-and-inertia)
    - [AlpineとBladeを使用する](#using-alpine)
    - [Axiosの設定](#configuring-axios)
- [バリデーションルールのカスタマイズ](#customizing-validation-rules)
- [ファイルアップロードの処理](#handling-file-uploads)
- [副作用の管理](#managing-side-effects)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

Laravel Precognitionを使用すると、将来のHTTPリクエストの結果を予測することができます。Precognitionの主な用途の一つは、アプリケーションのバックエンドのバリデーションルールを複製することなく、フロントエンドのJavaScriptアプリケーションに「ライブ」バリデーションを提供することです。Precognitionは、特にLaravelのInertiaベースの[スターターキット](starter-kits.md)と組み合わせると効果的です。

Laravelが「予知リクエスト」を受け取ると、ルートのミドルウェアをすべて実行し、ルートのコントローラの依存関係を解決します。これには、[フォームリクエスト](validation.md#form-request-validation)のバリデーションも含まれますが、実際にはルートのコントローラメソッドは実行されません。

<a name="live-validation"></a>
## ライブバリデーション

<a name="using-vue"></a>
### Vueを使用する

Laravel Precognitionを使用すると、バリデーションルールをフロントエンドのVueアプリケーションに複製することなく、ユーザーにライブバリデーションの体験を提供できます。どのように機能するかを説明するために、アプリケーション内で新しいユーザーを作成するためのフォームを構築しましょう。

まず、ルートにPrecognitionを有効にするために、`HandlePrecognitiveRequests`ミドルウェアをルート定義に追加する必要があります。また、ルートのバリデーションルールを格納するための[フォームリクエスト](validation.md#form-request-validation)を作成する必要があります。

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

次に、Laravel PrecognitionのフロントエンドヘルパーをVue用にNPM経由でインストールします。

```shell
npm install laravel-precognition-vue
```

Laravel Precognitionパッケージをインストールしたら、Precognitionの`useForm`関数を使用してフォームオブジェクトを作成できます。HTTPメソッド（`post`）、ターゲットURL（`/users`）、および初期フォームデータを提供します。

次に、ライブバリデーションを有効にするために、各入力の`change`イベントでフォームの`validate`メソッドを呼び出し、入力の名前を渡します。

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

これで、ユーザーがフォームを入力すると、Precognitionはルートのフォームリクエストのバリデーションルールに基づいてライブバリデーションを提供します。フォームの入力が変更されると、デバウンスされた「予知」バリデーションリクエストがLaravelアプリケーションに送信されます。デバウンスタイムアウトは、フォームの`setValidationTimeout`関数を呼び出して設定できます。

```js
form.setValidationTimeout(3000);
```

バリデーションリクエストが進行中の場合、フォームの`validating`プロパティは`true`になります。

```html
<div v-if="form.validating">
    Validating...
</div>
```

バリデーションリクエスト中またはフォーム送信中に返されたバリデーションエラーは、自動的にフォームの`errors`オブジェクトに入力されます。

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

フォームにエラーがあるかどうかは、フォームの`hasErrors`プロパティを使用して確認できます。

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

入力がバリデーションに合格したか失敗したかは、それぞれフォームの`valid`関数と`invalid`関数に入力の名前を渡すことで確認できます。

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> WARNING:  
> フォーム入力は、変更されてバリデーションレスポンスを受け取った後にのみ、有効または無効として表示されます。

Precognitionでフォームの入力のサブセットをバリデーションする場合、エラーを手動でクリアすると便利です。これは、フォームの`forgetError`関数を使用して行うことができます。

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

これまで見てきたように、入力の`change`イベントにフックして、ユーザーが入力と対話するときに個々の入力をバリデーションすることができます。しかし、ユーザーがまだ対話していない入力をバリデーションする必要がある場合があります。これは、ユーザーが対話したかどうかに関係なく、すべての表示されている入力をバリデーションしたい「ウィザード」を構築する場合によくあります。

Precognitionでこれを行うには、バリデーションしたいフィールドを「touched」としてマークするために、それらの名前を`touch`メソッドに渡します。次に、`validate`メソッドを`onSuccess`または`onValidationError`コールバックで呼び出します。

```html
<button
    type="button" 
    @click="form.touch(['name', 'email', 'phone']).validate({
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

もちろん、フォーム送信のレスポンスに反応してコードを実行することもできます。フォームの`submit`関数はAxiosリクエストのPromiseを返します。これにより、レスポンスペイロードにアクセスしたり、送信が成功した場合にフォーム入力をリセットしたり、リクエストが失敗した場合に対処したりする便利な方法が提供されます。

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

フォーム送信リクエストが進行中かどうかは、フォームの`processing`プロパティを検査することで確認できます。

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="using-vue-and-inertia"></a>
### VueとInertiaを使用する

> NOTE:  
> VueとInertiaを使用してLaravelアプリケーションを開発する際にスタートアップをスムーズにしたい場合は、[スターターキット](starter-kits.md)のいずれかを使用することを検討してください。Laravelのスターターキットは、新しいLaravelアプリケーションのためのバックエンドとフロントエンドの認証スキャフォールディングを提供します。

PrecognitionをVueとInertiaで使用する前に、[VueでPrecognitionを使用する](#using-vue)に関する一般的なドキュメントを確認してください。VueとInertiaを使用する場合、Inertia互換のPrecognitionライブラリをNPM経由でインストールする必要があります。

```shell
npm install laravel-precognition-vue-inertia
```

インストールが完了すると、Precognitionの`useForm`関数は、上記で説明したバリデーション機能を備えたInertiaの[フォームヘルパー](https://inertiajs.com/forms#form-helper)を返します。

フォームヘルパーの`submit`メソッドは、HTTPメソッドやURLを指定する必要がなくなり、代わりにInertiaの[訪問オプション](https://inertiajs.com/manual-visits)を最初の引数として渡すことができます。また、`submit`メソッドは上記のVueの例で見られるようなPromiseを返しません。代わりに、`submit`メソッドに与えられた訪問オプションでInertiaがサポートする[イベントコールバック](https://inertiajs.com/manual-visits#event-callbacks)を提供できます。

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit({
    preserveScroll: true,
    onSuccess: () => form.reset(),
});
</script>
```

<a name="using-react"></a>
### Reactを使用する

Laravel Precognitionを使用すると、バリデーションルールをフロントエンドのReactアプリケーションに複製することなく、ユーザーにライブバリデーションの体験を提供できます。どのように機能するかを説明するために、アプリケーション内で新しいユーザーを作成するためのフォームを構築しましょう。

まず、ルートにPrecognitionを有効にするために、`HandlePrecognitiveRequests`ミドルウェアをルート定義に追加する必要があります。また、ルートのバリデーションルールを格納するための[フォームリクエスト](validation.md#form-request-validation)を作成する必要があります。

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

次に、Laravel PrecognitionのフロントエンドヘルパーをReact用にNPM経由でインストールします。

```shell
npm install laravel-precognition-react
```

Laravel Precognitionパッケージをインストールしたら、Precognitionの`useForm`関数を使用してフォームオブジェクトを作成できます。HTTPメソッド（`post`）、ターゲットURL（`/users`）、および初期フォームデータを提供します。

ライブバリデーションを有効にするために、各入力の`change`および`blur`イベントをリッスンする必要があります。`change`イベントハンドラでは、`setData`関数を使用してフォームのデータを設定し、入力の名前と新しい値を渡します。次に、`blur`イベントハンドラで`validate`メソッドを呼び出し、入力の名前を渡します。

```jsx
import { useForm } from 'laravel-precognition-react';

export default function UserForm() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const handleChange = (event) => {
        form.setData(event.target.name, event.target.value);
    };

    const handleBlur = (event) => {
        form.validate(event.target.name);
    };

    const handleSubmit = (event) => {
        event.preventDefault();
        form.submit();
    };

    return (
        <form onSubmit={handleSubmit}>
            <label htmlFor="name">Name</label>
            <input
                id="name"
                name="name"
                value={form.data.name}
                onChange={handleChange}
                onBlur={handleBlur}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">Email</label>
            <input
                id="email"
                name="email"
                type="email"
                value={form.data.email}
                onChange={handleChange}
                onBlur={handleBlur}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button type="submit" disabled={form.processing}>
                Create User
            </button>
        </form>
    );
}
```

# Precognition

- [はじめに](#introduction)
- [ライブバリデーション](#live-validation)
    - [Vueを使用する](#using-vue)
    - [VueとInertiaを使用する](#using-vue-and-inertia)
    - [Reactを使用する](#using-react)
    - [ReactとInertiaを使用する](#using-react-and-inertia)
    - [AlpineとBladeを使用する](#using-alpine)
    - [Axiosの設定](#configuring-axios)
- [バリデーションルールのカスタマイズ](#customizing-validation-rules)
- [ファイルアップロードの処理](#handling-file-uploads)
- [副作用の管理](#managing-side-effects)
- [テスト](#testing)

<a name="introduction"></a>
## はじめに

Laravel Precognitionを使用すると、将来のHTTPリクエストの結果を予測することができます。Precognitionの主な用途の一つは、アプリケーションのバックエンドのバリデーションルールを複製することなく、フロントエンドのJavaScriptアプリケーションに「ライブ」バリデーションを提供することです。Precognitionは、特にLaravelのInertiaベースの[スターターキット](starter-kits.md)と組み合わせると効果的です。

Laravelが「予知リクエスト」を受け取ると、ルートのミドルウェアをすべて実行し、ルートのコントローラの依存関係を解決します。これには、[フォームリクエスト](validation.md#form-request-validation)のバリデーションも含まれますが、実際にはルートのコントローラメソッドは実行されません。

<a name="live-validation"></a>
## ライブバリデーション

<a name="using-vue"></a>
### Vueを使用する

Laravel Precognitionを使用すると、バリデーションルールをフロントエンドのVueアプリケーションに複製することなく、ユーザーにライブバリデーションの体験を提供できます。どのように機能するかを説明するために、アプリケーション内で新しいユーザーを作成するためのフォームを構築しましょう。

まず、ルートにPrecognitionを有効にするために、`HandlePrecognitiveRequests`ミドルウェアをルート定義に追加する必要があります。また、ルートのバリデーションルールを格納するための[フォームリクエスト](validation.md#form-request-validation)を作成する必要があります。

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

次に、Laravel PrecognitionのフロントエンドヘルパーをVue用にNPM経由でインストールします。

```shell
npm install laravel-precognition-vue
```

Laravel Precognitionパッケージをインストールしたら、Precognitionの`useForm`関数を使用してフォームオブジェクトを作成できます。HTTPメソッド（`post`）、ターゲットURL（`/users`）、および初期フォームデータを提供します。

次に、ライブバリデーションを有効にするために、各入力の`change`イベントでフォームの`validate`メソッドを呼び出し、入力の名前を渡します。

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

これで、ユーザーがフォームを入力すると、Precognitionはルートのフォームリクエストのバリデーションルールに基づいてライブバリデーションを提供します。フォームの入力が変更されると、デバウンスされた「予知」バリデーションリクエストがLaravelアプリケーションに送信されます。デバウンスタイムアウトは、フォームの`setValidationTimeout`関数を呼び出して設定できます。

```js
form.setValidationTimeout(3000);
```

バリデーションリクエストが進行中の場合、フォームの`validating`プロパティは`true`になります。

```html
<div v-if="form.validating">
    Validating...
</div>
```

バリデーションリクエスト中またはフォーム送信中に返されたバリデーションエラーは、自動的にフォームの`errors`オブジェクトに入力されます。

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

フォームにエラーがあるかどうかは、フォームの`hasErrors`プロパティを使用して確認できます。

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

入力がバリデーションに合格したか失敗したかは、それぞれフォームの`valid`関数と`invalid`関数に入力の名前を渡すことで確認できます。

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> WARNING:  
> フォーム入力は、変更されてバリデーションレスポンスを受け取った後にのみ、有効または無効として表示されます。

Precognitionでフォームの入力のサブセットをバリデーションする場合、エラーを手動でクリアすると便利です。これは、フォームの`forgetError`関数を使用して行うことができます。

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

これまで見てきたように、入力の`change`イベントにフックして、ユーザーが入力と対話するときに個々の入力をバリデーションすることができます。しかし、ユーザーがまだ対話していない入力をバリデーションする必要がある場合があります。これは、ユーザーが対話したかどうかに関係なく、すべての表示されている入力をバリデーションしたい「ウィザード」を構築する場合によくあります。

Precognitionでこれを行うには、バリデーションしたいフィールドを「touched」としてマークするために、それらの名前を`touch`メソッドに渡します。次に、`validate`メソッドを`onSuccess`または`onValidationError`コールバックで呼び出します。

```html
<button
    type="button" 
    @click="form.touch(['name', 'email', 'phone']).validate({
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

もちろん、フォーム送信のレスポンスに反応してコードを実行することもできます。フォームの`submit`関数はAxiosリクエストのPromiseを返します。これにより、レスポンスペイロードにアクセスしたり、送信が成功した場合にフォーム入力をリセットしたり、リクエストが失敗した場合に対処したりする便利な方法が提供されます。

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

フォーム送信リクエストが進行中かどうかは、フォームの`processing`プロパティを検査することで確認できます。

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="using-vue-and-inertia"></a>
### VueとInertiaを使用する

> NOTE:  
> VueとInertiaを使用してLaravelアプリケーションを開発する際にスタートアップをスムーズにしたい場合は、[スターターキット](starter-kits.md)のいずれかを使用することを検討してください。Laravelのスターターキットは、新しいLaravelアプリケーションのためのバックエンドとフロントエンドの認証スキャフォールディングを提供します。

PrecognitionをVueとInertiaで使用する前に、[VueでPrecognitionを使用する](#using-vue)に関する一般的なドキュメントを確認してください。VueとInertiaを使用する場合、Inertia互換のPrecognitionライブラリをNPM経由でインストールする必要があります。

```shell
npm install laravel-precognition-vue-inertia
```

インストールが完了すると、Precognitionの`useForm`関数は、上記で説明したバリデーション機能を備えたInertiaの[フォームヘルパー](https://inertiajs.com/forms#form-helper)を返します。

フォームヘルパーの`submit`メソッドは、HTTPメソッドやURLを指定する必要がなくなり、代わりにInertiaの[訪問オプション](https://inertiajs.com/manual-visits)を最初の引数として渡すことができます。また、`submit`メソッドは上記のVueの例で見られるようなPromiseを返しません。代わりに、`submit`メソッドに与えられた訪問オプションでInertiaがサポートする[イベントコールバック](https://inertiajs.com/manual-visits#event-callbacks)を提供できます。

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit({
    preserveScroll: true,
    onSuccess: () => form.reset(),
});
</script>
```

<a name="using-react"></a>
### Reactを使用する

Laravel Precognitionを使用すると、バリデーションルールをフロントエンドのReactアプリケーションに複製することなく、ユーザーにライブバリデーションの体験を提供できます。どのように機能するかを説明するために、アプリケーション内で新しいユーザーを作成するためのフォームを構築しましょう。

まず、ルートにPrecognitionを有効にするために、`HandlePrecognitiveRequests`ミドルウェアをルート定義に追加する必要があります。また、ルートのバリデーションルールを格納するための[フォームリクエスト](validation.md#form-request-validation)を作成する必要があります。

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

次に、Laravel PrecognitionのフロントエンドヘルパーをReact用にNPM経由でインストールします。

```shell
npm install laravel-precognition-react
```

Laravel Precognitionパッケージをインストールしたら、Precognitionの`useForm`関数を使用してフォームオブジェクトを作成できます。HTTPメソッド（`post`）、ターゲットURL（`/users`）、および初期フォームデータを提供します。

ライブバリデーションを有効にするために、各入力の`change`および`blur`イベントをリッスンする必要があります。`change`イベントハンドラでは、`setData`関数を使用してフォームのデータを設定し、入力の名前と新しい値を渡します。次に、`blur`イベントハンドラで`validate`メソッドを呼び出し、入力の名前を渡します。

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onSubmit={submit}>
            <label htmlFor="name">名前</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">メールアドレス</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                ユーザー作成
            </button>
        </form>
    );
};
```

ここで、ユーザーがフォームを入力すると、Precognitionはルートのフォームリクエスト内のバリデーションルールによって駆動されるライブバリデーション出力を提供します。フォームの入力が変更されると、デバウンスされた「予知」バリデーションリクエストがLaravelアプリケーションに送信されます。デバウンスタイムアウトは、フォームの`setValidationTimeout`関数を呼び出すことで設定できます：

```js
form.setValidationTimeout(3000);
```

バリデーションリクエストが進行中の場合、フォームの`validating`プロパティは`true`になります：

```jsx
{form.validating && <div>検証中...</div>}
```

バリデーションリクエストまたはフォーム送信中に返されるバリデーションエラーは、自動的にフォームの`errors`オブジェクトに格納されます：

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

フォームにエラーがあるかどうかは、フォームの`hasErrors`プロパティを使用して確認できます：

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

また、入力がバリデーションに合格したか失敗したかを、それぞれフォームの`valid`関数と`invalid`関数に入力の名前を渡すことで確認できます：

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> WARNING:  
> フォーム入力は、変更されてバリデーションレスポンスが受信された後にのみ、有効または無効として表示されます。

Precognitionでフォームの入力のサブセットを検証する場合、エラーを手動でクリアすることが役立ちます。これは、フォームの`forgetError`関数を使用して行うことができます：

```jsx
<input
    id="avatar"
    type="file"
    onChange={(e) => {
        form.setData('avatar', e.target.value);

        form.forgetError('avatar');
    }}
>
```

ここまで見てきたように、入力の`blur`イベントにフックして、ユーザーが操作する際に個々の入力を検証することができます；ただし、ユーザーがまだ操作していない入力を検証する必要がある場合もあります。これは、ユーザーが次のステップに移動する前に、ユーザーが操作したかどうかに関係なく、すべての表示されている入力を検証したい「ウィザード」を構築する場合に一般的です。

これをPrecognitionで行うには、検証したいフィールドを`touch`メソッドにそれらの名前を渡して「タッチ」としてマークする必要があります。そして、`onSuccess`または`onValidationError`コールバックを指定して`validate`メソッドを呼び出します：

```jsx
<button
    type="button"
    onClick={() => form.touch(['name', 'email', 'phone']).validate({
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })}
>次のステップ</button>
```

もちろん、フォーム送信のレスポンスに反応してコードを実行することもできます。フォームの`submit`関数はAxiosリクエストのプロミスを返します。これにより、レスポンスペイロードにアクセスし、フォーム送信が成功した場合にフォームの入力をリセットしたり、リクエストが失敗した場合に処理する便利な方法が提供されます：

```js
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('ユーザーが作成されました。');
        })
        .catch(error => {
            alert('エラーが発生しました。');
        });
};
```

フォーム送信リクエストが進行中かどうかは、フォームの`processing`プロパティを検査することで確認できます：

```html
<button disabled={form.processing}>
    送信
</button>
```

<a name="using-react-and-inertia"></a>
### ReactとInertiaを使用する

> NOTE:  
> LaravelアプリケーションをReactとInertiaで開発する際にスタートアップをスムーズにしたい場合は、[スターターキット](starter-kits.md)のいずれかを使用することを検討してください。Laravelのスターターキットは、新しいLaravelアプリケーションのためのバックエンドとフロントエンドの認証スキャフォールディングを提供します。

ReactとInertiaでPrecognitionを使用する前に、[ReactでPrecognitionを使用する](#using-react)に関する一般的なドキュメントを確認してください。ReactとInertiaを使用する場合、Inertia互換のPrecognitionライブラリをNPM経由でインストールする必要があります：

```shell
npm install laravel-precognition-react-inertia
```

インストールが完了すると、Precognitionの`useForm`関数は、上記で説明したバリデーション機能を備えたInertiaの[フォームヘルパー](https://inertiajs.com/forms#form-helper)を返します。

フォームヘルパーの`submit`メソッドは、HTTPメソッドやURLを指定する必要がなくなり、代わりにInertiaの[訪問オプション](https://inertiajs.com/manual-visits)を最初の引数として渡すことができます。さらに、`submit`メソッドは、上記のReactの例のようにPromiseを返しません。代わりに、`submit`メソッドに指定された訪問オプション内で、Inertiaがサポートする[イベントコールバック](https://inertiajs.com/manual-visits#event-callbacks)を提供できます：

```js
import { useForm } from 'laravel-precognition-react-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = (e) => {
    e.preventDefault();

    form.submit({
        preserveScroll: true,
        onSuccess: () => form.reset(),
    });
};
```

<a name="using-alpine"></a>
### AlpineとBladeを使用する

Laravel Precognitionを使用すると、フロントエンドのAlpineアプリケーションでバリデーションルールを複製することなく、ユーザーにライブバリデーション体験を提供できます。その仕組みを説明するために、アプリケーション内で新しいユーザーを作成するためのフォームを構築しましょう。

まず、ルートのPrecognitionを有効にするために、`HandlePrecognitiveRequests`ミドルウェアをルート定義に追加する必要があります。また、ルートのバリデーションルールを格納するための[フォームリクエスト](validation.md#form-request-validation)を作成する必要があります：

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

次に、Alpine用のLaravel PrecognitionフロントエンドヘルパーをNPM経由でインストールします：

```shell
npm install laravel-precognition-alpine
```

そして、`resources/js/app.js`ファイル内でAlpineにPrecognitionプラグインを登録します：

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

Laravel Precognitionパッケージがインストールされ、登録されたら、Precognitionの`$form`「マジック」を使用してフォームオブジェクトを作成できます。HTTPメソッド（`post`）、ターゲットURL（`/users`）、および初期フォームデータを提供します。

ライブバリデーションを有効にするには、フォームのデータを関連する入力にバインドし、各入力の`change`イベントをリッスンする必要があります。`change`イベントハンドラで、入力の名前を指定してフォームの`validate`メソッドを呼び出す必要があります：

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
    <label for="name">名前</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">メールアドレス</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        ユーザー作成
    </button>
</form>
```

ここで、ユーザーがフォームを入力すると、Precognitionはルートのフォームリクエスト内のバリデーションルールによって駆動されるライブバリデーション出力を提供します。フォームの入力が変更されると、デバウンスされた「予知」バリデーションリクエストがLaravelアプリケーションに送信されます。デバウンスタイムアウトは、フォームの`setValidationTimeout`関数を呼び出すことで設定できます：

```js
form.setValidationTimeout(3000);
```

バリデーションリクエストが進行中の場合、フォームの`validating`プロパティは`true`になります：

```html
<template x-if="form.validating">
    <div>検証中...</div>
</template>
```

バリデーションリクエストまたはフォーム送信中に返されるバリデーションエラーは、自動的にフォームの`errors`オブジェクトに格納されます：

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

フォームにエラーがあるかどうかは、フォームの`hasErrors`プロパティを使用して確認できます：

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

また、入力がバリデーションに合格したか失敗したかを、それぞれフォームの`valid`関数と`invalid`関数に入力の名前を渡すことで確認できます：

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> WARNING:  
> フォーム入力は、変更されてバリデーションレスポンスが受信された後にのみ、有効または無効として表示されます。

見てきたように、入力の`change`イベントにフックして、ユーザーが入力を操作するたびに個々の入力を検証することができます。しかし、ユーザーがまだ操作していない入力を検証する必要がある場合もあります。これは、ユーザーが次のステップに進む前に、ユーザーが操作したかどうかに関わらず、すべての表示されている入力を検証したい「ウィザード」を構築する際によくあることです。

Precognitionを使用してこれを行うには、検証したいフィールドを「触れた」状態にするために、それらの名前を`touch`メソッドに渡す必要があります。その後、`onSuccess`または`onValidationError`コールバックを指定して`validate`メソッドを呼び出します。

```html
<button
    type="button"
    @change="form.touch(['name', 'email', 'phone']).validate({
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>次のステップ</button>
```

フォーム送信リクエストが処理中かどうかは、フォームの`processing`プロパティを調べることで判断できます。

```html
<button :disabled="form.processing">
    送信
</button>
```

<a name="repopulating-old-form-data"></a>
#### 古いフォームデータの再入力

上記のユーザー作成の例では、Precognitionを使用してライブ検証を行っていますが、フォームの送信には従来のサーバーサイドのフォーム送信を行っています。そのため、サーバーサイドのフォーム送信から返された「古い」入力や検証エラーでフォームを再入力する必要があります。

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

または、XHRを介してフォームを送信したい場合は、フォームの`submit`関数を使用できます。これはAxiosリクエストのPromiseを返します。

```html
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    form.reset();

                    alert('ユーザーが作成されました。')
                })
                .catch(error => {
                    alert('エラーが発生しました。');
                });
        },
    }"
    @submit.prevent="submit"
>
```

<a name="configuring-axios"></a>
### Axiosの設定

Precognition検証ライブラリは、アプリケーションのバックエンドにリクエストを送信するために[Axios](https://github.com/axios/axios) HTTPクライアントを使用します。便宜上、アプリケーションで必要な場合にAxiosインスタンスをカスタマイズすることができます。例えば、`laravel-precognition-vue`ライブラリを使用している場合、アプリケーションの`resources/js/app.js`ファイルで各送信リクエストに追加のリクエストヘッダーを追加できます。

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

または、アプリケーションに既に設定されたAxiosインスタンスがある場合、Precognitionにそのインスタンスを使用するよう指示できます。

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

> WARNING:  
> Inertiaを使用したPrecognitionライブラリは、検証リクエストに対してのみ設定されたAxiosインスタンスを使用します。フォーム送信は常にInertiaによって送信されます。

<a name="customizing-validation-rules"></a>
## 検証ルールのカスタマイズ

リクエストの`isPrecognitive`メソッドを使用して、Precognitionリクエスト中に実行される検証ルールをカスタマイズすることができます。

例えば、ユーザー作成フォームでは、パスワードが「侵害されていない」ことを最終的なフォーム送信時にのみ検証したい場合があります。Precognition検証リクエストでは、パスワードが必須であり、最低8文字であることを検証します。`isPrecognitive`メソッドを使用することで、フォームリクエストで定義されたルールをカスタマイズできます。

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * リクエストに適用される検証ルールを取得します。
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

<a name="handling-file-uploads"></a>
## ファイルアップロードの処理

デフォルトでは、Laravel PrecognitionはPrecognition検証リクエスト中にファイルをアップロードまたは検証しません。これにより、大きなファイルが不必要に複数回アップロードされることを防ぎます。

この動作のため、アプリケーションは[対応するフォームリクエストの検証ルールをカスタマイズ](#customizing-validation-rules)して、フィールドが完全なフォーム送信時にのみ必須であることを指定する必要があります。

```php
/**
 * リクエストに適用される検証ルールを取得します。
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

すべての検証リクエストにファイルを含めたい場合は、クライアントサイドのフォームインスタンスで`validateFiles`関数を呼び出すことができます。

```js
form.validateFiles();
```

<a name="managing-side-effects"></a>
## 副作用の管理

`HandlePrecognitiveRequests`ミドルウェアをルートに追加する際には、Precognitionリクエスト中にスキップすべき_他の_ミドルウェアに副作用があるかどうかを考慮する必要があります。

例えば、アプリケーションとのユーザーの「インタラクション」の総数を増やすミドルウェアがあるかもしれませんが、Precognitionリクエストをインタラクションとしてカウントしたくない場合があります。これを実現するために、リクエストの`isPrecognitive`メソッドをチェックしてからインタラクションカウントを増やすことができます。

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * 受信リクエストを処理します。
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

<a name="testing"></a>
## テスト

テストでPrecognitionリクエストを行いたい場合、Laravelの`TestCase`には`Precognition`リクエストヘッダーを追加する`withPrecognition`ヘルパーが含まれています。

さらに、Precognitionリクエストが成功したことをアサートしたい場合（例：検証エラーが返されなかった場合）、レスポンスの`assertSuccessfulPrecognition`メソッドを使用できます。

===  "Pest"
```php
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();

    expect(User::count())->toBe(0);
});
```

===  "PHPUnit"
```php
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```
