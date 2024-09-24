# バリデーション

- [はじめに](#introduction)
- [バリデーションのクイックスタート](#validation-quickstart)
    - [ルートの定義](#quick-defining-the-routes)
    - [コントローラの作成](#quick-creating-the-controller)
    - [バリデーションロジックの記述](#quick-writing-the-validation-logic)
    - [バリデーションエラーの表示](#quick-displaying-the-validation-errors)
    - [フォームの再入力](#repopulating-forms)
    - [オプションフィールドに関する注意](#a-note-on-optional-fields)
    - [バリデーションエラーのレスポンスフォーマット](#validation-error-response-format)
- [フォームリクエストによるバリデーション](#form-request-validation)
    - [フォームリクエストの作成](#creating-form-requests)
    - [フォームリクエストの認可](#authorizing-form-requests)
    - [エラーメッセージのカスタマイズ](#customizing-the-error-messages)
    - [バリデーションのための入力の準備](#preparing-input-for-validation)
- [バリデータの手動作成](#manually-creating-validators)
    - [自動リダイレクト](#automatic-redirection)
    - [名前付きエラーバッグ](#named-error-bags)
    - [エラーメッセージのカスタマイズ](#manual-customizing-the-error-messages)
    - [追加のバリデーションの実行](#performing-additional-validation)
- [バリデーション済みの入力の操作](#working-with-validated-input)
- [エラーメッセージの操作](#working-with-error-messages)
    - [言語ファイルでのカスタムメッセージの指定](#specifying-custom-messages-in-language-files)
    - [言語ファイルでの属性の指定](#specifying-attribute-in-language-files)
    - [言語ファイルでの値の指定](#specifying-values-in-language-files)
- [利用可能なバリデーションルール](#available-validation-rules)
- [条件付きでルールを追加](#conditionally-adding-rules)
- [配列のバリデーション](#validating-arrays)
    - [ネストした配列入力のバリデーション](#validating-nested-array-input)
    - [エラーメッセージのインデックスと位置](#error-message-indexes-and-positions)
- [ファイルのバリデーション](#validating-files)
- [パスワードのバリデーション](#validating-passwords)
- [カスタムバリデーションルール](#custom-validation-rules)
    - [ルールオブジェクトの使用](#using-rule-objects)
    - [クロージャの使用](#using-closures)
    - [暗黙のルール](#implicit-rules)

<a name="introduction"></a>
## はじめに

Laravelは、アプリケーションの受信データを検証するためのいくつかの異なるアプローチを提供します。すべての受信HTTPリクエストで利用可能な`validate`メソッドを使用するのが最も一般的です。しかし、他のバリデーションアプローチについても説明します。

Laravelには、データに適用できるさまざまな便利なバリデーションルールが含まれており、特定のデータベーステーブル内で値が一意であるかどうかを検証する機能も提供します。これらのバリデーションルールについて詳しく説明し、Laravelのバリデーション機能をすべて理解できるようにします。

<a name="validation-quickstart"></a>
## バリデーションのクイックスタート

Laravelの強力なバリデーション機能について学ぶために、フォームを検証し、エラーメッセージをユーザーに表示する完全な例を見てみましょう。この概要を読むことで、Laravelを使用して受信リクエストデータを検証する方法について、一般的な理解を得ることができます。

<a name="quick-defining-the-routes"></a>
### ルートの定義

まず、`routes/web.php`ファイルに以下のルートが定義されていると仮定しましょう。

    use App\Http\Controllers\PostController;

    Route::get('/post/create', [PostController::class, 'create']);
    Route::post('/post', [PostController::class, 'store']);

`GET`ルートは、ユーザーが新しいブログ投稿を作成するためのフォームを表示し、`POST`ルートは新しいブログ投稿をデータベースに保存します。

<a name="quick-creating-the-controller"></a>
### コントローラの作成

次に、これらのルートに対する受信リクエストを処理するシンプルなコントローラを見てみましょう。今のところ`store`メソッドは空のままにしておきます。

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\View\View;

    class PostController extends Controller
    {
        /**
         * 新しいブログ投稿を作成するためのフォームを表示します。
         */
        public function create(): View
        {
            return view('post.create');
        }

        /**
         * 新しいブログ投稿を保存します。
         */
        public function store(Request $request): RedirectResponse
        {
            // バリデーションとブログ投稿の保存...

            $post = /** ... */

            return to_route('post.show', ['post' => $post->id]);
        }
    }

<a name="quick-writing-the-validation-logic"></a>
### バリデーションロジックの記述

これで、新しいブログ投稿を検証するロジックを`store`メソッドに記述する準備が整いました。これを行うには、`Illuminate\Http\Request`オブジェクトによって提供される`validate`メソッドを使用します。バリデーションルールが通過すると、コードは通常どおり実行されます。しかし、バリデーションが失敗すると、`Illuminate\Validation\ValidationException`例外がスローされ、適切なエラーレスポンスが自動的にユーザーに返されます。

従来のHTTPリクエスト中にバリデーションが失敗した場合、前のURLへのリダイレクトレスポンスが生成されます。受信リクエストがXHRリクエストの場合、[バリデーションエラーメッセージを含むJSONレスポンス](#validation-error-response-format)が返されます。

`validate`メソッドをよりよく理解するために、`store`メソッドに戻りましょう。

    /**
     * 新しいブログ投稿を保存します。
     */
    public function store(Request $request): RedirectResponse
    {
        $validated = $request->validate([
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        // ブログ投稿は有効です...

        return redirect('/posts');
    }

ご覧のように、バリデーションルールは`validate`メソッドに渡されます。心配しないでください。利用可能なすべてのバリデーションルールは[ドキュメント化されています](#available-validation-rules)。繰り返しますが、バリデーションが失敗すると、適切なレスポンスが自動的に生成されます。バリデーションが通過すると、コントローラは通常どおり実行されます。

あるいは、バリデーションルールを単一の`|`区切りの文字列ではなく、ルールの配列として指定することもできます。

    $validatedData = $request->validate([
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

さらに、`validateWithBag`メソッドを使用してリクエストを検証し、エラーメッセージを[名前付きエラーバッグ](#named-error-bags)に保存することもできます。

    $validatedData = $request->validateWithBag('post', [
        'title' => ['required', 'unique:posts', 'max:255'],
        'body' => ['required'],
    ]);

<a name="stopping-on-first-validation-failure"></a>
#### 最初のバリデーション失敗時に停止

属性の最初のバリデーション失敗時にバリデーションルールの実行を停止したい場合があります。そのためには、属性に`bail`ルールを割り当てます。

    $request->validate([
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
    ]);

この例では、`title`属性の`unique`ルールが失敗した場合、`max`ルールはチェックされません。ルールは割り当てられた順序で検証されます。

<a name="a-note-on-nested-attributes"></a>
#### ネストした属性に関する注意

受信HTTPリクエストに「ネスト」したフィールドデータが含まれている場合、「ドット」構文を使用してバリデーションルールでこれらのフィールドを指定できます。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'author.name' => 'required',
        'author.description' => 'required',
    ]);

一方、フィールド名にリテラルなピリオドが含まれている場合、バックスラッシュでエスケープして、「ドット」構文として解釈されないように明示的に防ぐことができます。

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'v1\.0' => 'required',
    ]);

<a name="quick-displaying-the-validation-errors"></a>
### バリデーションエラーの表示

では、受信リクエストフィールドが指定されたバリデーションルールを通過しない場合はどうなるでしょうか？前述のように、Laravelは自動的にユーザーを前の場所にリダイレクトします。さらに、すべてのバリデーションエラーと[リクエスト入力](requests.md#retrieving-old-input)は自動的に[セッションにフラッシュ](session.md#flash-data)されます。

`Illuminate\View\Middleware\ShareErrorsFromSession`ミドルウェアによって、アプリケーションのすべてのビューで`$errors`変数が共有されます。このミドルウェアは`web`ミドルウェアグループによって提供されます。このミドルウェアが適用されると、`$errors`変数は常にビューで利用可能になり、`$errors`変数が常に定義されており、安全に使用できると仮定できます。`$errors`変数は`Illuminate\Support\MessageBag`のインスタンスになります。このオブジェクトの操作についての詳細は、[そのドキュメント](#working-with-error-messages)を参照してください。

したがって、この例では、バリデーションが失敗すると、ユーザーはコントローラの`create`メソッドにリダイレクトされ、エラーメッセージをビューに表示できます。

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

<a name="quick-customizing-the-error-messages"></a>
#### エラーメッセージのカスタマイズ

Laravelの組み込みバリデーションルールには、アプリケーションの`lang/en/validation.php`ファイルにあるエラーメッセージがあります。アプリケーションに`lang`ディレクトリがない場合、`lang:publish` Artisanコマンドを使用してLaravelに作成させることができます。

`lang/en/validation.php`ファイル内には、各バリデーションルールの翻訳エントリがあります。アプリケーションのニーズに基づいて、これらのメッセージを自由に変更または修正できます。

さらに、このファイルを別の言語ディレクトリにコピーして、アプリケーションの言語用のメッセージを翻訳することもできます。Laravelのローカリゼーションについて詳しく知りたい場合は、完全な[ローカリゼーションドキュメント](localization.md)を確認してください。

> WARNING:  
> デフォルトでは、Laravelアプリケーションのスケルトンには`lang`ディレクトリは含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisanコマンドを介してそれらを公開することができます。

<a name="quick-xhr-requests-and-validation"></a>
#### XHRリクエストとバリデーション

この例では、従来のフォームを使用してデータをアプリケーションに送信しました。しかし、多くのアプリケーションはJavaScriptを使用したフロントエンドからXHRリクエストを受け取ります。XHRリクエスト中に`validate`メソッドを使用すると、Laravelはリダイレクトレスポンスを生成しません。代わりに、Laravelは[すべてのバリデーションエラーを含むJSONレスポンス](#validation-error-response-format)を生成します。このJSONレスポンスは422 HTTPステータスコードとともに送信されます。

<a name="the-at-error-directive"></a>
#### `@error`ディレクティブ

特定の属性に対するバリデーションエラーメッセージが存在するかどうかを素早く判断するために、`@error` [Blade](blade.md)ディレクティブを使用できます。`@error`ディレクティブ内で、エラーメッセージを表示するために`$message`変数をechoすることができます：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

[名前付きエラーバッグ](#named-error-bags)を使用している場合、`@error`ディレクティブにエラーバッグの名前を2番目の引数として渡すことができます：

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### フォームの再入力

バリデーションエラーによりLaravelがリダイレクトレスポンスを生成する場合、フレームワークは自動的に[リクエストのすべての入力をセッションにフラッシュ](session.md#flash-data)します。これにより、次のリクエスト中に入力にアクセスし、ユーザーが送信しようとしたフォームを再入力することが便利になります。

前のリクエストからフラッシュされた入力を取得するには、`Illuminate\Http\Request`のインスタンスで`old`メソッドを呼び出します。`old`メソッドは、[セッション](session.md)から以前にフラッシュされた入力データを引き出します：

    $title = $request->old('title');

Laravelはグローバルな`old`ヘルパーも提供しています。[Bladeテンプレート](blade.md)内で古い入力を表示する場合、`old`ヘルパーを使用してフォームを再入力する方が便利です。指定されたフィールドの古い入力が存在しない場合、`null`が返されます：

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### オプションフィールドに関する注意

デフォルトでは、Laravelはアプリケーションのグローバルミドルウェアスタックに`TrimStrings`と`ConvertEmptyStringsToNull`ミドルウェアを含んでいます。このため、バリデーターが`null`値を無効と見なさないようにするには、"オプション"のリクエストフィールドを`nullable`とマークする必要があることが多いです。例えば：

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

この例では、`publish_at`フィールドが`null`または有効な日付表現のいずれかであることを指定しています。`nullable`修飾子がルール定義に追加されていない場合、バリデーターは`null`を無効な日付と見なします。

<a name="validation-error-response-format"></a>
### バリデーションエラーレスポンスのフォーマット

アプリケーションが`Illuminate\Validation\ValidationException`例外をスローし、受信したHTTPリクエストがJSONレスポンスを期待している場合、Laravelは自動的にエラーメッセージをフォーマットし、`422 Unprocessable Entity` HTTPレスポンスを返します。

以下に、バリデーションエラーのJSONレスポンスフォーマットの例を示します。ネストされたエラーキーは"ドット"記法形式に平坦化されていることに注意してください：

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## フォームリクエストのバリデーション

<a name="creating-form-requests"></a>
### フォームリクエストの作成

より複雑なバリデーションシナリオの場合、"フォームリクエスト"を作成することができます。フォームリクエストは、独自のバリデーションと認可ロジックをカプセル化するカスタムリクエストクラスです。フォームリクエストクラスを作成するには、`make:request` Artisan CLIコマンドを使用できます：

```shell
php artisan make:request StorePostRequest
```

生成されたフォームリクエストクラスは`app/Http/Requests`ディレクトリに配置されます。このディレクトリが存在しない場合、`make:request`コマンドを実行すると作成されます。Laravelによって生成される各フォームリクエストには、`authorize`と`rules`の2つのメソッドがあります。

ご想像の通り、`authorize`メソッドは現在認証されているユーザーがリクエストで表されるアクションを実行できるかどうかを判断する役割を果たし、`rules`メソッドはリクエストのデータに適用されるバリデーションルールを返します：

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> NOTE:  
> `rules`メソッドのシグネチャ内で必要な依存関係をタイプヒントすることができます。これらは自動的にLaravelの[サービスコンテナ](container.md)を介して解決されます。

では、バリデーションルールはどのように評価されるのでしょうか？必要なのは、コントローラーメソッドにリクエストをタイプヒントすることだけです。コントローラーメソッドが呼び出される前に、受信したフォームリクエストがバリデーションされます。つまり、コントローラーにバリデーションロジックを散らかす必要はありません：

    /**
     * Store a new blog post.
     */
    public function store(StorePostRequest $request): RedirectResponse
    {
        // The incoming request is valid...

        // Retrieve the validated input data...
        $validated = $request->validated();

        // Retrieve a portion of the validated input data...
        $validated = $request->safe()->only(['name', 'email']);
        $validated = $request->safe()->except(['name', 'email']);

        // Store the blog post...

        return redirect('/posts');
    }

バリデーションが失敗すると、リダイレクトレスポンスが生成され、ユーザーを前の場所に戻します。エラーもセッションにフラッシュされ、表示できるようになります。リクエストがXHRリクエストだった場合、[バリデーションエラーのJSON表現](#validation-error-response-format)を含む422ステータスコードのHTTPレスポンスがユーザーに返されます。

> NOTE:  
> Inertiaを使用したLaravelフロントエンドにリアルタイムのフォームリクエストバリデーションを追加する必要がありますか？[Laravel Precognition](precognition.md)をチェックしてください。

<a name="performing-additional-validation-on-form-requests"></a>
#### 追加のバリデーションの実行

最初のバリデーションが完了した後に追加のバリデーションを実行する必要がある場合、フォームリクエストの`after`メソッドを使用してこれを実現できます。

`after`メソッドは、バリデーションが完了した後に呼び出されるコールバックまたはクロージャの配列を返す必要があります。与えられたコールバックは`Illuminate\Validation\Validator`インスタンスを受け取り、必要に応じて追加のエラーメッセージを発生させることができます：

    use Illuminate\Validation\Validator;

    /**
     * Get the "after" validation callables for the request.
     */
    public function after(): array
    {
        return [
            function (Validator $validator) {
                if ($this->somethingElseIsInvalid()) {
                    $validator->errors()->add(
                        'field',
                        'Something is wrong with this field!'
                    );
                }
            }
        ];
    }

前述のように、`after`メソッドによって返される配列には、呼び出し可能なクラスを含めることもできます。これらのクラスの`__invoke`メソッドは`Illuminate\Validation\Validator`インスタンスを受け取ります：

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
use Illuminate\Validation\Validator;

/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        new ValidateUserStatus,
        new ValidateShippingTime,
        function (Validator $validator) {
            //
        }
    ];
}
```

<a name="request-stopping-on-first-validation-rule-failure"></a>
#### 最初のバリデーション失敗時に停止

リクエストクラスに`stopOnFirstFailure`プロパティを追加することで、バリデーターに単一のバリデーション失敗が発生したらすべての属性のバリデーションを停止するように指示できます：

    /**
     * Indicates if the validator should stop on the first rule failure.
     *
     * @var bool
     */
    protected $stopOnFirstFailure = true;

<a name="customizing-the-redirect-location"></a>
#### リダイレクト先のカスタマイズ

前述のように、フォームリクエストのバリデーションが失敗した場合、ユーザーを前の場所に戻すリダイレクトレスポンスが生成されます。しかし、この動作を自由にカスタマイズすることができます。そのためには、フォームリクエストに`$redirect`プロパティを定義します：

```php
/**
 * バリデーションが失敗した場合にユーザーがリダイレクトされるURI。
 *
 * @var string
 */
protected $redirect = '/dashboard';
```

または、ユーザーを名前付きルートにリダイレクトしたい場合は、代わりに `$redirectRoute` プロパティを定義できます：

```php
/**
 * バリデーションが失敗した場合にユーザーがリダイレクトされるルート。
 *
 * @var string
 */
protected $redirectRoute = 'dashboard';
```

<a name="authorizing-form-requests"></a>
### フォームリクエストの認可

フォームリクエストクラスには `authorize` メソッドも含まれています。このメソッド内で、認証されたユーザーが実際に特定のリソースを更新する権限を持っているかどうかを判断できます。たとえば、ユーザーが更新しようとしているブログコメントを実際に所有しているかどうかを判断できます。おそらく、このメソッド内で [認可ゲートとポリシー](authorization.md) とやり取りするでしょう：

```php
use App\Models\Comment;

/**
 * ユーザーがこのリクエストを行うことを認可されているかどうかを判断します。
 */
public function authorize(): bool
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

すべてのフォームリクエストはベースの Laravel リクエストクラスを拡張しているため、`user` メソッドを使用して現在認証されているユーザーにアクセスできます。また、上記の例で `route` メソッドの呼び出しに注意してください。このメソッドを使用すると、呼び出されているルートに定義された URI パラメータにアクセスできます。たとえば、以下の例では `{comment}` パラメータにアクセスできます：

```php
Route::post('/comment/{comment}');
```

したがって、アプリケーションが [ルートモデルバインディング](routing.md#route-model-binding) を利用している場合、リクエストのプロパティとして解決されたモデルにアクセスすることで、コードをさらに簡潔にすることができます：

```php
return $this->user()->can('update', $this->comment);
```

`authorize` メソッドが `false` を返す場合、403 ステータスコードの HTTP レスポンスが自動的に返され、コントローラメソッドは実行されません。

リクエストの認可ロジックをアプリケーションの別の部分で処理する予定の場合は、`authorize` メソッドを完全に削除するか、単に `true` を返すことができます：

```php
/**
 * ユーザーがこのリクエストを行うことを認可されているかどうかを判断します。
 */
public function authorize(): bool
{
    return true;
}
```

> NOTE:  
> `authorize` メソッドのシグネチャ内に必要な依存関係をタイプヒントで指定できます。これらは自動的に Laravel の [サービスコンテナ](container.md) を介して解決されます。

<a name="customizing-the-error-messages"></a>
### エラーメッセージのカスタマイズ

フォームリクエストによって使用されるエラーメッセージをカスタマイズするには、`messages` メソッドをオーバーライドします。このメソッドは、属性/ルールのペアとそれに対応するエラーメッセージの配列を返す必要があります：

```php
/**
 * 定義されたバリデーションルールのエラーメッセージを取得します。
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => 'タイトルは必須です',
        'body.required' => 'メッセージは必須です',
    ];
}
```

<a name="customizing-the-validation-attributes"></a>
#### バリデーション属性のカスタマイズ

Laravel の組み込みバリデーションルールの多くのエラーメッセージには、`:attribute` プレースホルダが含まれています。バリデーションメッセージの `:attribute` プレースホルダをカスタム属性名に置き換えたい場合は、`attributes` メソッドをオーバーライドしてカスタム名を指定できます。このメソッドは、属性/名前のペアの配列を返す必要があります：

```php
/**
 * バリデータエラーのカスタム属性を取得します。
 *
 * @return array<string, string>
 */
public function attributes(): array
{
    return [
        'email' => 'メールアドレス',
    ];
}
```

<a name="preparing-input-for-validation"></a>
### バリデーションのための入力の準備

バリデーションルールを適用する前にリクエストからのデータを準備またはサニタイズする必要がある場合は、`prepareForValidation` メソッドを使用できます：

```php
use Illuminate\Support\Str;

/**
 * バリデーションのためにデータを準備します。
 */
protected function prepareForValidation(): void
{
    $this->merge([
        'slug' => Str::slug($this->slug),
    ]);
}
```

同様に、バリデーションが完了した後にリクエストデータを正規化する必要がある場合は、`passedValidation` メソッドを使用できます：

```php
/**
 * バリデーションが成功した試行を処理します。
 */
protected function passedValidation(): void
{
    $this->replace(['name' => 'Taylor']);
}
```

<a name="manually-creating-validators"></a>
## バリデータの手動作成

リクエストの `validate` メソッドを使用したくない場合は、`Validator` [ファサード](facades.md) を使用してバリデータインスタンスを手動で作成できます。ファサードの `make` メソッドは新しいバリデータインスタンスを生成します：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PostController extends Controller
{
    /**
     * 新しいブログ記事を保存します。
     */
    public function store(Request $request): RedirectResponse
    {
        $validator = Validator::make($request->all(), [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ]);

        if ($validator->fails()) {
            return redirect('/post/create')
                        ->withErrors($validator)
                        ->withInput();
        }

        // バリデーション済みの入力を取得...
        $validated = $validator->validated();

        // バリデーション済みの入力の一部を取得...
        $validated = $validator->safe()->only(['name', 'email']);
        $validated = $validator->safe()->except(['name', 'email']);

        // ブログ記事を保存...

        return redirect('/posts');
    }
}
```

`make` メソッドに渡される最初の引数は、バリデーション対象のデータです。2番目の引数は、データに適用する必要があるバリデーションルールの配列です。

リクエストのバリデーションが失敗したかどうかを判断した後、`withErrors` メソッドを使用してセッションにエラーメッセージをフラッシュできます。このメソッドを使用すると、リダイレクト後に `$errors` 変数が自動的にビューと共有され、ユーザーに簡単に表示できるようになります。`withErrors` メソッドは、バリデータ、`MessageBag`、または PHP の `array` を受け入れます。

#### 最初のバリデーション失敗時に停止

`stopOnFirstFailure` メソッドは、バリデータに一度バリデーション失敗が発生したらすべての属性のバリデーションを停止するように指示します：

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="automatic-redirection"></a>
### 自動リダイレクト

バリデータインスタンスを手動で作成したいが、HTTP リクエストの `validate` メソッドが提供する自動リダイレクトを利用したい場合は、既存のバリデータインスタンスで `validate` メソッドを呼び出すことができます。バリデーションが失敗した場合、ユーザーは自動的にリダイレクトされます。XHR リクエストの場合は、[JSON レスポンスが返されます](#validation-error-response-format)：

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

バリデーションが失敗した場合にエラーメッセージを [名前付きエラーバッグ](#named-error-bags) に保存するには、`validateWithBag` メソッドを使用できます：

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

<a name="named-error-bags"></a>
### 名前付きエラーバッグ

1 つのページに複数のフォームがある場合、バリデーションエラーを含む `MessageBag` に名前を付け、特定のフォームのエラーメッセージを取得できるようにすることができます。これを実現するには、`withErrors` の 2 番目の引数として名前を渡します：

```php
return redirect('/register')->withErrors($validator, 'login');
```

その後、`$errors` 変数から名前付き `MessageBag` インスタンスにアクセスできます：

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### エラーメッセージのカスタマイズ

必要に応じて、バリデータインスタンスが使用するデフォルトのエラーメッセージの代わりにカスタムエラーメッセージを指定できます。カスタムメッセージを指定するには、いくつかの方法があります。まず、`Validator::make` メソッドにカスタムメッセージを 3 番目の引数として渡すことができます：

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => ':attribute フィールドは必須です。',
]);
```

この例では、`:attribute` プレースホルダは、バリデーション中のフィールドの実際の名前に置き換えられます。バリデーションメッセージで他のプレースホルダを利用することもできます。例：

```php
$messages = [
    'same' => ':attribute と :other は一致する必要があります。',
    'size' => ':attribute は正確に :size である必要があります。',
    'between' => ':attribute の値 :input は :min から :max の間である必要があります。',
    'in' => ':attribute は次のいずれかのタイプである必要があります: :values',
];
```

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### 特定の属性に対するカスタムメッセージの指定

特定の属性に対してのみカスタムエラーメッセージを指定したい場合があります。これは、「ドット」表記を使用して行うことができます。最初に属性の名前を指定し、次にルールを指定します：

```php
$messages = [
    'email.required' => 'メールアドレスを教えてください！',
];
```

<a name="specifying-custom-attribute-values"></a>
#### カスタム属性値の指定

Laravelの組み込みエラーメッセージの多くには、バリデーション対象のフィールドまたは属性の名前に置き換えられる `:attribute` プレースホルダが含まれています。特定のフィールドに対してこれらのプレースホルダを置き換える値をカスタマイズするには、`Validator::make` メソッドの第4引数としてカスタム属性の配列を渡すことができます。

    $validator = Validator::make($input, $rules, $messages, [
        'email' => 'メールアドレス',
    ]);

<a name="performing-additional-validation"></a>
### 追加のバリデーションの実行

初期バリデーションが完了した後に追加のバリデーションを実行する必要がある場合があります。これは、バリデータの `after` メソッドを使用して実現できます。`after` メソッドは、バリデーションが完了した後に呼び出されるクロージャまたはコールバックの配列を受け取ります。指定されたコールバックは `Illuminate\Validation\Validator` インスタンスを受け取り、必要に応じて追加のエラーメッセージを発生させることができます。

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make(/* ... */);

    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add(
                'field', 'このフィールドに何か問題があります！'
            );
        }
    });

    if ($validator->fails()) {
        // ...
    }

前述のように、`after` メソッドはコールバックの配列も受け取ります。これは、"バリデーション後" のロジックが呼び出し可能なクラスにカプセル化されている場合に特に便利です。これらのクラスは、`__invoke` メソッドを介して `Illuminate\Validation\Validator` インスタンスを受け取ります。

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;

$validator->after([
    new ValidateUserStatus,
    new ValidateShippingTime,
    function ($validator) {
        // ...
    },
]);
```

<a name="working-with-validated-input"></a>
## バリデーション済みの入力の操作

フォームリクエストまたは手動で作成したバリデータインスタンスを使用して受信リクエストデータをバリデーションした後、実際にバリデーションを受けた受信リクエストデータを取得したい場合があります。これはいくつかの方法で実現できます。まず、フォームリクエストまたはバリデータインスタンスで `validated` メソッドを呼び出すことができます。このメソッドは、バリデーションを受けたデータの配列を返します。

    $validated = $request->validated();

    $validated = $validator->validated();

または、フォームリクエストまたはバリデータインスタンスで `safe` メソッドを呼び出すこともできます。このメソッドは `Illuminate\Support\ValidatedInput` のインスタンスを返します。このオブジェクトは、バリデーション済みデータのサブセットまたはバリデーション済みデータ全体を取得するための `only`、`except`、`all` メソッドを公開しています。

    $validated = $request->safe()->only(['name', 'email']);

    $validated = $request->safe()->except(['name', 'email']);

    $validated = $request->safe()->all();

さらに、`Illuminate\Support\ValidatedInput` インスタンスは、配列のように反復処理したりアクセスしたりすることができます。

    // バリデーション済みデータは反復処理できます...
    foreach ($request->safe() as $key => $value) {
        // ...
    }

    // バリデーション済みデータは配列としてアクセスできます...
    $validated = $request->safe();

    $email = $validated['email'];

バリデーション済みデータに追加のフィールドを追加したい場合は、`merge` メソッドを呼び出すことができます。

    $validated = $request->safe()->merge(['name' => 'Taylor Otwell']);

バリデーション済みデータを [コレクション](collections.md) インスタンスとして取得したい場合は、`collect` メソッドを呼び出すことができます。

    $collection = $request->safe()->collect();

<a name="working-with-error-messages"></a>
## エラーメッセージの操作

`Validator` インスタンスで `errors` メソッドを呼び出すと、エラーメッセージを操作するためのさまざまな便利なメソッドを持つ `Illuminate\Support\MessageBag` インスタンスが返されます。すべてのビューで自動的に利用可能な `$errors` 変数も `MessageBag` クラスのインスタンスです。

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### フィールドの最初のエラーメッセージの取得

特定のフィールドの最初のエラーメッセージを取得するには、`first` メソッドを使用します。

    $errors = $validator->errors();

    echo $errors->first('email');

<a name="retrieving-all-error-messages-for-a-field"></a>
#### フィールドのすべてのエラーメッセージの取得

特定のフィールドのすべてのメッセージを配列として取得するには、`get` メソッドを使用します。

    foreach ($errors->get('email') as $message) {
        // ...
    }

配列フォームフィールドをバリデーションする場合、`*` 文字を使用して各配列要素のすべてのメッセージを取得できます。

    foreach ($errors->get('attachments.*') as $message) {
        // ...
    }

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### すべてのフィールドのすべてのエラーメッセージの取得

すべてのフィールドのすべてのメッセージを配列として取得するには、`all` メソッドを使用します。

    foreach ($errors->all() as $message) {
        // ...
    }

<a name="determining-if-messages-exist-for-a-field"></a>
#### フィールドにメッセージが存在するかどうかの確認

特定のフィールドにエラーメッセージが存在するかどうかを確認するには、`has` メソッドを使用します。

    if ($errors->has('email')) {
        // ...
    }

<a name="specifying-custom-messages-in-language-files"></a>
### 言語ファイルでのカスタムメッセージの指定

Laravelの組み込みバリデーションルールにはそれぞれ、アプリケーションの `lang/en/validation.php` ファイルにあるエラーメッセージがあります。アプリケーションに `lang` ディレクトリがない場合は、`lang:publish` Artisan コマンドを使用してLaravelに作成させることができます。

`lang/en/validation.php` ファイル内に、各バリデーションルールの翻訳エントリがあります。アプリケーションのニーズに基づいてこれらのメッセージを自由に変更または修正できます。

さらに、このファイルを別の言語ディレクトリにコピーして、アプリケーションの言語のメッセージを翻訳することもできます。Laravelのローカリゼーションの詳細については、完全な [ローカリゼーションドキュメント](localization.md) を確認してください。

> WARNING:  
> デフォルトでは、Laravelアプリケーションのスケルトンには `lang` ディレクトリが含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを介してそれらを公開することができます。

<a name="custom-messages-for-specific-attributes"></a>
#### 特定の属性のカスタムメッセージ

アプリケーションのバリデーション言語ファイル内で指定された属性とルールの組み合わせに使用されるエラーメッセージをカスタマイズすることができます。そのためには、アプリケーションの `lang/xx/validation.php` 言語ファイルの `custom` 配列にメッセージのカスタマイズを追加します。

    'custom' => [
        'email' => [
            'required' => 'メールアドレスを教えてください！',
            'max' => 'メールアドレスが長すぎます！'
        ],
    ],

<a name="specifying-attribute-in-language-files"></a>
### 言語ファイルでの属性の指定

Laravelの組み込みエラーメッセージの多くには、バリデーション対象のフィールドまたは属性の名前に置き換えられる `:attribute` プレースホルダが含まれています。バリデーションメッセージの `:attribute` 部分をカスタム値に置き換えたい場合は、`lang/xx/validation.php` 言語ファイルの `attributes` 配列でカスタム属性名を指定できます。

    'attributes' => [
        'email' => 'メールアドレス',
    ],

> WARNING:  
> デフォルトでは、Laravelアプリケーションのスケルトンには `lang` ディレクトリが含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを介してそれらを公開することができます。

<a name="specifying-values-in-language-files"></a>
### 言語ファイルでの値の指定

Laravelの組み込みバリデーションルールのエラーメッセージの一部には、リクエスト属性の現在の値に置き換えられる `:value` プレースホルダが含まれています。ただし、バリデーションメッセージの `:value` 部分をカスタム表現に置き換えたい場合があります。たとえば、`payment_type` の値が `cc` の場合にクレジットカード番号が必須であるというルールを考えてみましょう。

    Validator::make($request->all(), [
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

このバリデーションルールが失敗すると、次のエラーメッセージが生成されます。

```none
The credit card number field is required when payment type is cc.
```

支払いタイプの値として `cc` を表示する代わりに、`lang/xx/validation.php` 言語ファイルで `values` 配列を定義することで、よりユーザーフレンドリーな値表現を指定できます。

    'values' => [
        'payment_type' => [
            'cc' => 'クレジットカード'
        ],
    ],

> WARNING:  
> デフォルトでは、Laravelアプリケーションのスケルトンには `lang` ディレクトリが含まれていません。Laravelの言語ファイルをカスタマイズしたい場合は、`lang:publish` Artisan コマンドを介してそれらを公開することができます。

この値を定義した後、バリデーションルールは次のエラーメッセージを生成します。

```none
The credit card number field is required when payment type is クレジットカード.
```

<a name="available-validation-rules"></a>
## 利用可能なバリデーションルール

以下は、利用可能なすべてのバリデーションルールとその機能のリストです。

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

# 検証ルール

[Accepted](#rule-accepted)  
[Accepted If](#rule-accepted-if)  
[Active URL](#rule-active-url)  
[After (Date)](#rule-after)  
[After Or Equal (Date)](#rule-after-or-equal)  
[Alpha](#rule-alpha)  
[Alpha Dash](#rule-alpha-dash)  
[Alpha Numeric](#rule-alpha-num)  
[Array](#rule-array)  
[Ascii](#rule-ascii)  
[Bail](#rule-bail)  
[Before (Date)](#rule-before)  
[Before Or Equal (Date)](#rule-before-or-equal)  
[Between](#rule-between)  
[Boolean](#rule-boolean)  
[Confirmed](#rule-confirmed)  
[Contains](#rule-contains)  
[Current Password](#rule-current-password)  
[Date](#rule-date)  
[Date Equals](#rule-date-equals)  
[Date Format](#rule-date-format)  
[Decimal](#rule-decimal)  
[Declined](#rule-declined)  
[Declined If](#rule-declined-if)  
[Different](#rule-different)  
[Digits](#rule-digits)  
[Digits Between](#rule-digits-between)  
[Dimensions (Image Files)](#rule-dimensions)  
[Distinct](#rule-distinct)  
[Doesnt Start With](#rule-doesnt-start-with)  
[Doesnt End With](#rule-doesnt-end-with)  
[Email](#rule-email)  
[Ends With](#rule-ends-with)  
[Enum](#rule-enum)  
[Exclude](#rule-exclude)  
[Exclude If](#rule-exclude-if)  
[Exclude Unless](#rule-exclude-unless)  
[Exclude With](#rule-exclude-with)  
[Exclude Without](#rule-exclude-without)  
[Exists (Database)](#rule-exists)  
[Extensions](#rule-extensions)  
[File](#rule-file)  
[Filled](#rule-filled)  
[Greater Than](#rule-gt)  
[Greater Than Or Equal](#rule-gte)  
[Hex Color](#rule-hex-color)  
[Image (File)](#rule-image)  
[In](#rule-in)  
[In Array](#rule-in-array)  
[Integer](#rule-integer)  
[IP Address](#rule-ip)  
[JSON](#rule-json)  
[Less Than](#rule-lt)  
[Less Than Or Equal](#rule-lte)  
[List](#rule-list)  
[Lowercase](#rule-lowercase)  
[MAC Address](#rule-mac)  
[Max](#rule-max)  
[Max Digits](#rule-max-digits)  
[MIME Types](#rule-mimetypes)  
[MIME Type By File Extension](#rule-mimes)  
[Min](#rule-min)  
[Min Digits](#rule-min-digits)  
[Missing](#rule-missing)  
[Missing If](#rule-missing-if)  
[Missing Unless](#rule-missing-unless)  
[Missing With](#rule-missing-with)  
[Missing With All](#rule-missing-with-all)  
[Multiple Of](#rule-multiple-of)  
[Not In](#rule-not-in)  
[Not Regex](#rule-not-regex)  
[Nullable](#rule-nullable)  
[Numeric](#rule-numeric)  
[Present](#rule-present)  
[Present If](#rule-present-if)  
[Present Unless](#rule-present-unless)  
[Present With](#rule-present-with)  
[Present With All](#rule-present-with-all)  
[Prohibited](#rule-prohibited)  
[Prohibited If](#rule-prohibited-if)  
[Prohibited Unless](#rule-prohibited-unless)  
[Prohibits](#rule-prohibits)  
[Regular Expression](#rule-regex)  
[Required](#rule-required)  
[Required If](#rule-required-if)  
[Required If Accepted](#rule-required-if-accepted)  
[Required If Declined](#rule-required-if-declined)  
[Required Unless](#rule-required-unless)  
[Required With](#rule-required-with)  
[Required With All](#rule-required-with-all)  
[Required Without](#rule-required-without)  
[Required Without All](#rule-required-without-all)  
[Required Array Keys](#rule-required-array-keys)  
[Same](#rule-same)  
[Size](#rule-size)  
[Sometimes](#validating-when-present)  
[Starts With](#rule-starts-with)  
[String](#rule-string)  
[Timezone](#rule-timezone)  
[Unique (Database)](#rule-unique)  
[Uppercase](#rule-uppercase)  
[URL](#rule-url)  
[ULID](#rule-ulid)  
[UUID](#rule-uuid)

</div>

<a name="rule-accepted"></a>
### accepted

検証中のフィールドは `"yes"`、`"on"`、`1`、`"1"`、`true`、または `"true"` でなければなりません。これは「利用規約」の同意などのフィールドの検証に便利です。

<a name="rule-accepted-if"></a>
### accepted_if:anotherfield,value,...

検証中のフィールドは、別のフィールドが指定された値と等しい場合に `"yes"`、`"on"`、`1`、`"1"`、`true`、または `"true"` でなければなりません。これは「利用規約」の同意などのフィールドの検証に便利です。

<a name="rule-active-url"></a>
### active_url

検証中のフィールドは、`dns_get_record` PHP 関数に従って有効な A または AAAA レコードを持っている必要があります。提供された URL のホスト名は、`parse_url` PHP 関数を使用して抽出され、`dns_get_record` に渡されます。

<a name="rule-after"></a>
### after:_date_

検証中のフィールドは、指定された日付より後の値でなければなりません。日付は `strtotime` PHP 関数に渡され、有効な `DateTime` インスタンスに変換されます。

```php
'start_date' => 'required|date|after:tomorrow'
```

`strtotime` で評価される日付文字列の代わりに、日付と比較する別のフィールドを指定することもできます。

```php
'finish_date' => 'required|date|after:start_date'
```

<a name="rule-after-or-equal"></a>
### after\_or\_equal:_date_

検証中のフィールドは、指定された日付と等しいかそれより後の値でなければなりません。詳細については、[after](#rule-after) ルールを参照してください。

<a name="rule-alpha"></a>
### alpha

検証中のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=) および [`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) に含まれる Unicode のアルファベット文字でなければなりません。

この検証ルールを ASCII 範囲 (`a-z` および `A-Z`) の文字に制限するには、検証ルールに `ascii` オプションを指定できます。

```php
'username' => 'alpha:ascii',
```

<a name="rule-alpha-dash"></a>
### alpha_dash

検証中のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=)、[`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=)、[`\p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=) に含まれる Unicode のアルファベット数字文字、および ASCII のダッシュ (`-`) とアンダースコア (`_`) でなければなりません。

この検証ルールを ASCII 範囲 (`a-z` および `A-Z`) の文字に制限するには、検証ルールに `ascii` オプションを指定できます。

```php
'username' => 'alpha_dash:ascii',
```

<a name="rule-alpha-num"></a>
### alpha_num

検証中のフィールドは、[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=)、[`\p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=)、および [`\p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=) に含まれる Unicode のアルファベット数字文字でなければなりません。

この検証ルールを ASCII 範囲 (`a-z` および `A-Z`) の文字に制限するには、検証ルールに `ascii` オプションを指定できます。

```php
'username' => 'alpha_num:ascii',
```

<a name="rule-array"></a>
### array

検証中のフィールドは、PHP の `array` でなければなりません。

`array` ルールに追加の値が提供されると、入力配列の各キーは、ルールに提供される値のリスト内に存在しなければなりません。以下の例では、入力配列の `admin` キーは、`array` ルールに提供される値のリストに含まれていないため無効です。

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

一般的に、配列内に存在することが許可される配列キーを常に指定する必要があります。

<a name="rule-ascii"></a>
### ascii

検証中のフィールドは、完全に 7 ビット ASCII 文字でなければなりません。

<a name="rule-bail"></a>
### bail

フィールドの最初の検証失敗後に、フィールドの検証ルールの実行を停止します。

`bail` ルールは、検証失敗が発生した場合に特定のフィールドの検証のみを停止しますが、`stopOnFirstFailure` メソッドは、単一の検証失敗が発生した場合にすべての属性の検証を停止するようにバリデータに指示します。

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="rule-before"></a>
### before:_date_

検証中のフィールドは、指定された日付より前の値でなければなりません。日付は PHP の `strtotime` 関数に渡され、有効な `DateTime` インスタンスに変換されます。さらに、[`after`](#rule-after) ルールと同様に、検証中の別のフィールドの名前を `date` の値として指定できます。

<a name="rule-before-or-equal"></a>
### before\_or\_equal:_date_

検証中のフィールドは、指定された日付と等しいかそれより前の値でなければなりません。日付は PHP の `strtotime` 関数に渡され、有効な `DateTime` インスタンスに変換されます。さらに、[`after`](#rule-after) ルールと同様に、検証中の別のフィールドの名前を `date` の値として指定できます。

<a name="rule-between"></a>
### between:_min_,_max_

検証中のフィールドは、指定された _min_ と _max_ の間のサイズでなければなりません（両端を含む）。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-boolean"></a>
### boolean

検証中のフィールドは、ブール値にキャスト可能でなければなりません。受け入れられる入力は `true`、`false`、`1`、`0`、`"1"`、および `"0"` です。

<a name="rule-confirmed"></a>
### confirmed

検証中のフィールドは、`{field}_confirmation` と一致するフィールドを持っている必要があります。たとえば、検証中のフィールドが `password` の場合、`password_confirmation` フィールドが入力に存在しなければなりません。

<a name="rule-contains"></a>
### contains:_foo_,_bar_,...

検証中のフィールドは、すべての指定されたパラメータ値を含む配列でなければなりません。

<a name="rule-current-password"></a>
### current_password

検証中のフィールドは、認証されたユーザーのパスワードと一致しなければなりません。ルールの最初のパラメータを使用して、[認証ガード](authentication.md) を指定できます。

```php
'password' => 'current_password:api'
```

<a name="rule-date"></a>
### date

検証中のフィールドは、`strtotime` PHP 関数に従って有効な非相対日付でなければなりません。

<a name="rule-date-equals"></a>
### date_equals:_date_

検証中のフィールドは、指定された日付と等しい値でなければなりません。日付は `strtotime` PHP 関数に渡され、有効な `DateTime` インスタンスに変換されます。

検証対象のフィールドは、指定された日付と等しくなければなりません。日付は、有効な `DateTime` インスタンスに変換するために、PHP の `strtotime` 関数に渡されます。

<a name="rule-date-format"></a>
#### date_format:_format_,...

検証対象のフィールドは、指定されたフォーマットのいずれかに一致しなければなりません。フィールドを検証する際には、`date` または `date_format` のいずれかを使用する必要があり、両方を使用することはできません。この検証ルールは、PHP の [DateTime](https://www.php.net/manual/en/class.datetime.php) クラスがサポートするすべてのフォーマットをサポートしています。

<a name="rule-decimal"></a>
#### decimal:_min_,_max_

検証対象のフィールドは数値であり、指定された小数点以下の桁数を含まなければなりません:

    // 小数点以下2桁でなければならない (9.99)...
    'price' => 'decimal:2'

    // 小数点以下2から4桁の間でなければならない...
    'price' => 'decimal:2,4'

<a name="rule-declined"></a>
#### declined

検証対象のフィールドは `"no"`、`"off"`、`0`、`"0"`、`false`、または `"false"` でなければなりません。

<a name="rule-declined-if"></a>
#### declined_if:anotherfield,value,...

検証対象のフィールドは、別のフィールドが指定された値と等しい場合に `"no"`、`"off"`、`0`、`"0"`、`false`、または `"false"` でなければなりません。

<a name="rule-different"></a>
#### different:_field_

検証対象のフィールドは、_field_ と異なる値でなければなりません。

<a name="rule-digits"></a>
#### digits:_value_

検証対象の整数は、_value_ の正確な長さを持たなければなりません。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

検証対象の整数は、指定された _min_ と _max_ の間の長さを持たなければなりません。

<a name="rule-dimensions"></a>
#### dimensions

検証対象のファイルは、ルールのパラメータで指定された寸法制約を満たす画像でなければなりません:

    'avatar' => 'dimensions:min_width=100,min_height=200'

利用可能な制約は: _min\_width_、_max\_width_、_min\_height_、_max\_height_、_width_、_height_、_ratio_ です。

_ratio_ 制約は、幅を高さで割ったものとして表されるべきです。これは `3/2` のような分数または `1.5` のような浮動小数点数で指定できます:

    'avatar' => 'dimensions:ratio=3/2'

このルールは複数の引数を必要とするため、`Rule::dimensions` メソッドを使用してルールを流暢に構築することができます:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

配列を検証する際、検証対象のフィールドは重複する値を持ってはなりません:

    'foo.*.id' => 'distinct'

Distinct はデフォルトで緩い変数比較を使用します。厳密な比較を使用するには、検証ルール定義に `strict` パラメータを追加できます:

    'foo.*.id' => 'distinct:strict'

検証ルールの引数に `ignore_case` を追加することで、ルールが大文字と小文字の違いを無視するようにできます:

    'foo.*.id' => 'distinct:ignore_case'

<a name="rule-doesnt-start-with"></a>
#### doesnt_start_with:_foo_,_bar_,...

検証対象のフィールドは、指定された値のいずれかで始まってはなりません。

<a name="rule-doesnt-end-with"></a>
#### doesnt_end_with:_foo_,_bar_,...

検証対象のフィールドは、指定された値のいずれかで終わってはなりません。

<a name="rule-email"></a>
#### email

検証対象のフィールドは、メールアドレスとしてフォーマットされていなければなりません。この検証ルールは、メールアドレスの検証に [`egulias/email-validator`](https://github.com/egulias/EmailValidator) パッケージを利用します。デフォルトでは `RFCValidation` バリデータが適用されますが、他の検証スタイルも適用できます:

    'email' => 'email:rfc,dns'

上記の例では、`RFCValidation` と `DNSCheckValidation` の検証が適用されます。適用可能な検証スタイルの完全なリストは以下の通りです:

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`
- `filter_unicode`: `FilterEmailValidation::unicode()`

</div>

`filter` バリデータは、PHP の `filter_var` 関数を使用し、Laravel 5.8 以前のデフォルトのメール検証動作でした。

> WARNING:  
> `dns` および `spoof` バリデータには、PHP の `intl` 拡張が必要です。

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

検証対象のフィールドは、指定された値のいずれかで終わらなければなりません。

<a name="rule-enum"></a>
#### enum

`Enum` ルールは、クラスベースのルールで、検証対象のフィールドが有効な enum 値を含んでいるかどうかを検証します。`Enum` ルールは、enum の名前を唯一のコンストラクタ引数として受け取ります。プリミティブ値を検証する場合、`Enum` ルールには backed Enum を提供する必要があります:

    use App\Enums\ServerStatus;
    use Illuminate\Validation\Rule;

    $request->validate([
        'status' => [Rule::enum(ServerStatus::class)],
    ]);

`Enum` ルールの `only` および `except` メソッドを使用して、どの enum ケースを有効と見なすかを制限できます:

    Rule::enum(ServerStatus::class)
        ->only([ServerStatus::Pending, ServerStatus::Active]);

    Rule::enum(ServerStatus::class)
        ->except([ServerStatus::Pending, ServerStatus::Active]);

`when` メソッドを使用して、`Enum` ルールを条件付きで変更できます:

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rule;

Rule::enum(ServerStatus::class)
    ->when(
        Auth::user()->isAdmin(),
        fn ($rule) => $rule->only(...),
        fn ($rule) => $rule->only(...),
    );
```

<a name="rule-exclude"></a>
#### exclude

検証対象のフィールドは、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

検証対象のフィールドは、_anotherfield_ フィールドが _value_ と等しい場合、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。

複雑な条件付き除外ロジックが必要な場合、`Rule::excludeIf` メソッドを使用できます。このメソッドは、ブール値またはクロージャを受け取ります。クロージャが与えられた場合、クロージャは検証対象のフィールドを除外するかどうかを示すために `true` または `false` を返すべきです:

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

検証対象のフィールドは、_anotherfield_ フィールドが _value_ と等しくない限り、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。_value_ が `null` (`exclude_unless:name,null`) の場合、検証対象のフィールドは、比較フィールドが `null` であるか、比較フィールドがリクエストデータに存在しない限り、除外されます。

<a name="rule-exclude-with"></a>
#### exclude_with:_anotherfield_

検証対象のフィールドは、_anotherfield_ フィールドが存在する場合、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exclude-without"></a>
#### exclude_without:_anotherfield_

検証対象のフィールドは、_anotherfield_ フィールドが存在しない場合、`validate` および `validated` メソッドによって返されるリクエストデータから除外されます。

<a name="rule-exists"></a>
#### exists:_table_,_column_

検証対象のフィールドは、指定されたデータベーステーブルに存在しなければなりません。

<a name="basic-usage-of-exists-rule"></a>
#### Exists ルールの基本的な使用法

    'state' => 'exists:states'

`column` オプションが指定されていない場合、フィールド名が使用されます。したがって、この場合、ルールは `states` データベーステーブルに、リクエストの `state` 属性値に一致する `state` 列の値を持つレコードが存在することを検証します。

<a name="specifying-a-custom-column-name"></a>
#### カスタム列名の指定

検証ルールによって使用されるべきデータベース列名を明示的に指定するには、データベーステーブル名の後に列名を配置します:

    'state' => 'exists:states,abbreviation'

時には、`exists` クエリに使用される特定のデータベース接続を指定する必要があるかもしれません。これは、接続名をテーブル名の前に付けることで実現できます:

    'email' => 'exists:connection.staff,email'

テーブル名を直接指定する代わりに、テーブル名を決定するために使用される Eloquent モデルを指定することもできます:

    'user_id' => 'exists:App\Models\User,id'

検証ルールによって実行されるクエリをカスタマイズしたい場合、`Rule` クラスを使用してルールを流暢に定義できます。この例では、検証ルールを `|` 文字で区切る代わりに配列として指定します:

    use Illuminate\Database\Query\Builder;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function (Builder $query) {
                return $query->where('account_id', 1);
            }),
        ],
    ]);

`Rule::exists` メソッドによって生成される `exists` ルールによって使用されるべきデータベース列名を明示的に指定するには、`exists` メソッドの第2引数として列名を提供します:

    'state' => Rule::exists('states', 'abbreviation'),

<a name="rule-extensions"></a>
#### extensions:_foo_,_bar_,...

検証対象のファイルは、リストされた拡張子のいずれかに対応するユーザーが割り当てた拡張子を持たなければなりません:

    'photo' => ['required', 'extensions:jpg,png'],

> WARNING:  
> ファイルの検証をユーザーが割り当てた拡張子のみに依存するべきではありません。このルールは通常、[`mimes`](#rule-mimes) または [`mimetypes`](#rule-mimetypes) ルールと組み合わせて使用する必要があります。

<a name="rule-file"></a>
#### file

検証中のフィールドは、正常にアップロードされたファイルでなければなりません。

<a name="rule-filled"></a>
#### filled

検証中のフィールドは、存在する場合に空であってはなりません。

<a name="rule-gt"></a>
#### gt:_field_

検証中のフィールドは、指定された _field_ または _value_ より大きくなければなりません。2つのフィールドは同じ型でなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-gte"></a>
#### gte:_field_

検証中のフィールドは、指定された _field_ または _value_ 以上でなければなりません。2つのフィールドは同じ型でなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-hex-color"></a>
#### hex_color

検証中のフィールドは、有効な [16進数](https://developer.mozilla.org/en-US/docs/Web/CSS/hex-color) 形式の色の値を含んでいなければなりません。

<a name="rule-image"></a>
#### image

検証中のファイルは、画像（jpg、jpeg、png、bmp、gif、svg、または webp）でなければなりません。

<a name="rule-in"></a>
#### in:_foo_,_bar_,...

検証中のフィールドは、指定された値のリストに含まれていなければなりません。このルールはしばしば配列を `implode` する必要がありますが、`Rule::in` メソッドを使用してルールを流暢に構築することができます：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'zones' => [
            'required',
            Rule::in(['first-zone', 'second-zone']),
        ],
    ]);

`in` ルールが `array` ルールと組み合わされると、入力配列内の各値は `in` ルールに提供された値のリスト内に存在しなければなりません。次の例では、入力配列内の `LAS` 空港コードは無効です。なぜなら、`in` ルールに提供された空港のリストに含まれていないからです：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $input = [
        'airports' => ['NYC', 'LAS'],
    ];

    Validator::make($input, [
        'airports' => [
            'required',
            'array',
        ],
        'airports.*' => Rule::in(['NYC', 'LIT']),
    ]);

<a name="rule-in-array"></a>
#### in_array:_anotherfield_.*

検証中のフィールドは、_anotherfield_ の値の中に存在しなければなりません。

<a name="rule-integer"></a>
#### integer

検証中のフィールドは整数でなければなりません。

> WARNING:  
> この検証ルールは、入力が "integer" 変数型であることを検証しません。PHP の `FILTER_VALIDATE_INT` ルールで受け入れられる型であることのみを検証します。入力が数値であることを検証する場合は、このルールと [ `numeric` 検証ルール](#rule-numeric) を組み合わせて使用してください。

<a name="rule-ip"></a>
#### ip

検証中のフィールドは、IP アドレスでなければなりません。

<a name="ipv4"></a>
#### ipv4

検証中のフィールドは、IPv4 アドレスでなければなりません。

<a name="ipv6"></a>
#### ipv6

検証中のフィールドは、IPv6 アドレスでなければなりません。

<a name="rule-json"></a>
#### json

検証中のフィールドは、有効な JSON 文字列でなければなりません。

<a name="rule-lt"></a>
#### lt:_field_

検証中のフィールドは、指定された _field_ より小さくなければなりません。2つのフィールドは同じ型でなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-lte"></a>
#### lte:_field_

検証中のフィールドは、指定された _field_ 以下でなければなりません。2つのフィールドは同じ型でなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ規則を使用して評価されます。

<a name="rule-lowercase"></a>
#### lowercase

検証中のフィールドは、小文字でなければなりません。

<a name="rule-list"></a>
#### list

検証中のフィールドは、リストである配列でなければなりません。配列は、そのキーが 0 から `count($array) - 1` までの連続した数値で構成されている場合、リストと見なされます。

<a name="rule-mac"></a>
#### mac_address

検証中のフィールドは、MAC アドレスでなければなりません。

<a name="rule-max"></a>
#### max:_value_

検証中のフィールドは、最大 _value_ 以下でなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-max-digits"></a>
#### max_digits:_value_

検証中の整数は、最大 _value_ の長さでなければなりません。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

検証中のファイルは、指定された MIME タイプのいずれかと一致しなければなりません：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

アップロードされたファイルの MIME タイプを決定するために、ファイルの内容が読み取られ、フレームワークは MIME タイプを推測しようとします。これは、クライアントが提供した MIME タイプと異なる場合があります。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

検証中のファイルは、リストされた拡張子のいずれかに対応する MIME タイプを持っていなければなりません：

    'photo' => 'mimes:jpg,bmp,png'

拡張子のみを指定する必要がありますが、このルールは実際にはファイルの内容を読み取り、その MIME タイプを推測することでファイルの MIME タイプを検証します。MIME タイプとそれに対応する拡張子の完全なリストは、次の場所で見つけることができます：

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="mime-types-and-extensions"></a>
#### MIME タイプと拡張子

この検証ルールは、MIME タイプとユーザーがファイルに割り当てた拡張子の一致を検証しません。たとえば、`mimes:png` 検証ルールは、有効な PNG コンテンツを含むファイルを有効な PNG 画像と見なします。たとえファイルが `photo.txt` という名前であってもです。ファイルのユーザー割り当て拡張子を検証したい場合は、[`extensions`](#rule-extensions) ルールを使用してください。

<a name="rule-min"></a>
#### min:_value_

検証中のフィールドは、最小 _value_ を持っていなければなりません。文字列、数値、配列、ファイルは、[`size`](#rule-size) ルールと同じ方法で評価されます。

<a name="rule-min-digits"></a>
#### min_digits:_value_

検証中の整数は、最小 _value_ の長さでなければなりません。

<a name="rule-multiple-of"></a>
#### multiple_of:_value_

検証中のフィールドは、_value_ の倍数でなければなりません。

<a name="rule-missing"></a>
#### missing

検証中のフィールドは、入力データに存在してはなりません。

<a name="rule-missing-if"></a>
#### missing_if:_anotherfield_,_value_,...

_anotherfield_ フィールドがいずれかの _value_ と等しい場合、検証中のフィールドは存在してはなりません。

<a name="rule-missing-unless"></a>
#### missing_unless:_anotherfield_,_value_

_anotherfield_ フィールドがいずれかの _value_ と等しい場合を除き、検証中のフィールドは存在してはなりません。

<a name="rule-missing-with"></a>
#### missing_with:_foo_,_bar_,...

指定された他のフィールドのいずれかが存在する場合、検証中のフィールドは存在してはなりません。

<a name="rule-missing-with-all"></a>
#### missing_with_all:_foo_,_bar_,...

指定された他のすべてのフィールドが存在する場合、検証中のフィールドは存在してはなりません。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

検証中のフィールドは、指定された値のリストに含まれてはなりません。`Rule::notIn` メソッドを使用してルールを流暢に構築することができます：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

検証中のフィールドは、指定された正規表現に一致してはなりません。

内部的には、このルールは PHP の `preg_match` 関数を使用します。指定されたパターンは、`preg_match` で必要とされるのと同じ形式に従う必要があり、有効な区切り文字を含める必要があります。例：`'email' => 'not_regex:/^.+$/i'`。

> WARNING:  
> `regex` / `not_regex` パターンを使用する場合、特に正規表現に `|` 文字が含まれている場合、検証ルールを `|` 区切り文字ではなく配列で指定する必要があるかもしれません。

<a name="rule-nullable"></a>
#### nullable

検証中のフィールドは `null` である可能性があります。

<a name="rule-numeric"></a>
#### numeric

検証中のフィールドは、[数値](https://www.php.net/manual/en/function.is-numeric.php) でなければなりません。

<a name="rule-present"></a>
#### present

検証中のフィールドは、入力データに存在しなければなりません。

<a name="rule-present-if"></a>
#### present_if:_anotherfield_,_value_,...

_anotherfield_ フィールドがいずれかの _value_ と等しい場合、検証中のフィールドは存在しなければなりません。

<a name="rule-present-unless"></a>
#### present_unless:_anotherfield_,_value_

_anotherfield_ フィールドがいずれかの _value_ と等しい場合を除き、検証中のフィールドは存在しなければなりません。

<a name="rule-present-with"></a>
#### present_with:_foo_,_bar_,...

指定された他のフィールドのいずれかが存在する場合、検証中のフィールドは存在しなければなりません。

<a name="rule-present-with-all"></a>
#### present_with_all:_foo_,_bar_,...

指定された他のすべてのフィールドが存在する場合、検証中のフィールドは存在しなければなりません。

<a name="rule-prohibited"></a>
#### prohibited

検証中のフィールドは、存在しないか空でなければなりません。フィールドは、次のいずれかの基準を満たす場合、「空」と見なされます：

<div class="content-list" markdown="1">

- 値が `null` である。
- 値が空の文字列である。
- 値が空の配列または空の `Countable` オブジェクトである。
- 値が空のパスを持つアップロードファイルである。

</div>

<a name="rule-prohibited-if"></a>
#### prohibited_if:_anotherfield_,_value_,...

_anotherfield_ フィールドがいずれかの _value_ と等しい場合、検証中のフィールドは存在しないか空でなければなりません。フィールドは、次のいずれかの基準を満たす場合、「空」と見なされます：

<div class="content-list" markdown="1">

- 値が `null` である。
- 値が空の文字列である。
- 値が空の配列または空の `Countable` オブジェクトである。
- 値が空のパスを持つアップロードファイルである。

</div>

複雑な条件付き禁止ロジックが必要な場合、`Rule::prohibitedIf`メソッドを利用できます。このメソッドは、ブール値またはクロージャを受け取ります。クロージャが与えられた場合、クロージャはバリデーション対象のフィールドが禁止されるべきかどうかを示すために`true`または`false`を返すべきです。

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
]);
```

<a name="rule-prohibited-unless"></a>
#### prohibited_unless:_anotherfield_,_value_,...

バリデーション対象のフィールドは、_anotherfield_フィールドが任意の_value_と等しくない限り、存在しないか空である必要があります。フィールドが「空」であるとは、以下のいずれかの条件を満たすことを意味します：

<div class="content-list" markdown="1">

- 値が`null`である。
- 値が空の文字列である。
- 値が空の配列または空の`Countable`オブジェクトである。
- 値が空のパスを持つアップロードファイルである。

</div>

<a name="rule-prohibits"></a>
#### prohibits:_anotherfield_,...

バリデーション対象のフィールドが存在しないか空でない場合、_anotherfield_内のすべてのフィールドは存在しないか空である必要があります。フィールドが「空」であるとは、以下のいずれかの条件を満たすことを意味します：

<div class="content-list" markdown="1">

- 値が`null`である。
- 値が空の文字列である。
- 値が空の配列または空の`Countable`オブジェクトである。
- 値が空のパスを持つアップロードファイルである。

</div>

<a name="rule-regex"></a>
#### regex:_pattern_

バリデーション対象のフィールドは、指定された正規表現に一致する必要があります。

内部的には、このルールはPHPの`preg_match`関数を使用します。指定されたパターンは、`preg_match`が要求するのと同じフォーマットに従う必要があり、有効な区切り文字も含める必要があります。例：`'email' => 'regex:/^.+@.+$/i'`。

> WARNING:  
> `regex` / `not_regex`パターンを使用する場合、特に正規表現に`|`文字が含まれている場合、ルールを配列で指定する必要があるかもしれません。

<a name="rule-required"></a>
#### required

バリデーション対象のフィールドは、入力データに存在し、空でない必要があります。フィールドが「空」であるとは、以下のいずれかの条件を満たすことを意味します：

<div class="content-list" markdown="1">

- 値が`null`である。
- 値が空の文字列である。
- 値が空の配列または空の`Countable`オブジェクトである。
- 値がパスを持たないアップロードファイルである。

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

バリデーション対象のフィールドは、_anotherfield_フィールドが任意の_value_と等しい場合、存在し、空でない必要があります。

`required_if`ルールのためにより複雑な条件を構築したい場合、`Rule::requiredIf`メソッドを使用できます。このメソッドは、ブール値またはクロージャを受け取ります。クロージャが渡された場合、クロージャはバリデーション対象のフィールドが必要かどうかを示すために`true`または`false`を返すべきです：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
]);
```

<a name="rule-required-if-accepted"></a>
#### required_if_accepted:_anotherfield_,...

バリデーション対象のフィールドは、_anotherfield_フィールドが`"yes"`、`"on"`、`1`、`"1"`、`true`、または`"true"`と等しい場合、存在し、空でない必要があります。

<a name="rule-required-if-declined"></a>
#### required_if_declined:_anotherfield_,...

バリデーション対象のフィールドは、_anotherfield_フィールドが`"no"`、`"off"`、`0`、`"0"`、`false`、または`"false"`と等しい場合、存在し、空でない必要があります。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

バリデーション対象のフィールドは、_anotherfield_フィールドが任意の_value_と等しくない限り、存在し、空でない必要があります。これはまた、_value_が`null`でない限り、_anotherfield_がリクエストデータに存在する必要があることを意味します。_value_が`null`の場合（`required_unless:name,null`）、バリデーション対象のフィールドは、比較フィールドが`null`であるか、比較フィールドがリクエストデータから欠落している限り、必要です。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

バリデーション対象のフィールドは、他の指定されたフィールドのいずれかが存在し、空でない場合にのみ、存在し、空でない必要があります。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

バリデーション対象のフィールドは、他の指定されたフィールドすべてが存在し、空でない場合にのみ、存在し、空でない必要があります。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

バリデーション対象のフィールドは、他の指定されたフィールドのいずれかが空または存在しない場合にのみ、存在し、空でない必要があります。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

バリデーション対象のフィールドは、他の指定されたフィールドすべてが空または存在しない場合にのみ、存在し、空でない必要があります。

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

バリデーション対象のフィールドは、配列であり、少なくとも指定されたキーを含む必要があります。

<a name="rule-same"></a>
#### same:_field_

指定された_field_は、バリデーション対象のフィールドと一致する必要があります。

<a name="rule-size"></a>
#### size:_value_

バリデーション対象のフィールドは、指定された_value_と一致するサイズを持つ必要があります。文字列データの場合、_value_は文字数に対応します。数値データの場合、_value_は指定された整数値に対応します（属性は`numeric`または`integer`ルールも持つ必要があります）。配列の場合、_size_は配列の`count`に対応します。ファイルの場合、_size_はファイルサイズ（キロバイト単位）に対応します。いくつかの例を見てみましょう：

```php
// 文字列が正確に12文字であることを検証する...
'title' => 'size:12';

// 提供された整数が10であることを検証する...
'seats' => 'integer|size:10';

// 配列が正確に5つの要素を持つことを検証する...
'tags' => 'array|size:5';

// アップロードされたファイルが正確に512キロバイトであることを検証する...
'image' => 'file|size:512';
```

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

バリデーション対象のフィールドは、指定された値のいずれかで始まる必要があります。

<a name="rule-string"></a>
#### string

バリデーション対象のフィールドは、文字列である必要があります。フィールドが`null`であることも許可したい場合は、フィールドに`nullable`ルールを割り当てる必要があります。

<a name="rule-timezone"></a>
#### timezone

バリデーション対象のフィールドは、`DateTimeZone::listIdentifiers`メソッドに従った有効なタイムゾーン識別子である必要があります。

[`DateTimeZone::listIdentifiers`メソッド](https://www.php.net/manual/en/datetimezone.listidentifiers.php)によって受け入れられる引数も、このバリデーションルールに提供できます：

```php
'timezone' => 'required|timezone:all';

'timezone' => 'required|timezone:Africa';

'timezone' => 'required|timezone:per_country,US';
```

<a name="rule-unique"></a>
#### unique:_table_,_column_

バリデーション対象のフィールドは、指定されたデータベーステーブル内に存在してはなりません。

**カスタムテーブル/カラム名の指定：**

テーブル名を直接指定する代わりに、テーブル名を決定するために使用されるEloquentモデルを指定できます：

```php
'email' => 'unique:App\Models\User,email_address'
```

`column`オプションは、フィールドの対応するデータベースカラムを指定するために使用できます。`column`オプションが指定されていない場合、バリデーション対象のフィールドの名前が使用されます。

```php
'email' => 'unique:users,email_address'
```

**カスタムデータベース接続の指定**

Validatorによって行われるデータベースクエリのためにカスタム接続を設定する必要がある場合があります。これを行うには、テーブル名の前に接続名を付けます：

```php
'email' => 'unique:connection.users,email_address'
```

**ユニークルールを特定のIDを無視するように強制する：**

ユニークバリデーション中に特定のIDを無視したい場合があります。例えば、ユーザーの名前、メールアドレス、場所を含む「プロフィール更新」画面を考えてみましょう。メールアドレスがユニークであることを検証したいと思うでしょう。しかし、ユーザーが名前フィールドのみを変更し、メールフィールドを変更しない場合、ユーザーがすでにそのメールアドレスの所有者であるため、バリデーションエラーを投げたくないでしょう。

バリデータにユーザーのIDを無視するように指示するには、`Rule`クラスを使用してルールを流暢に定義します。この例では、ルールを`|`文字で区切る代わりに、バリデーションルールを配列として指定します：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> WARNING:  
> ユーザー制御のリクエスト入力を`ignore`メソッドに渡すべきではありません。代わりに、自動増分IDやEloquentモデルインスタンスからのUUIDなど、システム生成の一意のIDのみを渡すべきです。そうしないと、アプリケーションはSQLインジェクション攻撃に対して脆弱になります。

`ignore`メソッドにモデルキーの値を渡す代わりに、モデルインスタンス全体を渡すこともできます。Laravelは自動的にモデルからキーを抽出します：

```php
Rule::unique('users')->ignore($user)
```

テーブルが`id`以外の主キーカラム名を使用している場合、`ignore`メソッドを呼び出す際にカラム名を指定できます：

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

デフォルトでは、`unique`ルールはバリデーション対象の属性名に一致するカラムのユニーク性をチェックします。ただし、`unique`メソッドの第二引数として異なるカラム名を渡すことができます：

```php
Rule::unique('users', 'email_address')->ignore($user->id)
```

**追加のWhere句の追加：**

以下のMarkdownコンテンツを日本語に翻訳します。すべてのMarkdownフォーマットを保持し、ヘッダーには'#'を使用します。

クエリ条件を追加指定するには、`where`メソッドを使用してクエリをカスタマイズします。例えば、`account_id`カラムの値が`1`であるレコードのみを検索対象とするクエリ条件を追加します。

    'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))

<a name="rule-uppercase"></a>
#### uppercase

検証対象のフィールドは大文字でなければなりません。

<a name="rule-url"></a>
#### url

検証対象のフィールドは有効なURLでなければなりません。

有効と見なされるURLプロトコルを指定したい場合は、検証ルールパラメータとしてプロトコルを渡すことができます。

```php
'url' => 'url:http,https',

'game' => 'url:minecraft,steam',
```

<a name="rule-ulid"></a>
#### ulid

検証対象のフィールドは有効な[Universally Unique Lexicographically Sortable Identifier](https://github.com/ulid/spec) (ULID)でなければなりません。

<a name="rule-uuid"></a>
#### uuid

検証対象のフィールドは有効なRFC 4122（バージョン1、3、4、または5）のuniversally unique identifier (UUID)でなければなりません。

<a name="conditionally-adding-rules"></a>
## 条件付きでルールを追加する

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### 特定の値を持つフィールドの検証をスキップする

他のフィールドが特定の値を持つ場合に、特定のフィールドの検証を行わないようにしたいことがあります。これは`exclude_if`検証ルールを使用して実現できます。この例では、`has_appointment`フィールドが`false`の場合、`appointment_date`と`doctor_name`フィールドの検証は行われません。

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_if:has_appointment,false|required|date',
        'doctor_name' => 'exclude_if:has_appointment,false|required|string',
    ]);

または、`exclude_unless`ルールを使用して、他のフィールドが特定の値を持つ場合にのみ検証を行わないようにすることもできます。

    $validator = Validator::make($data, [
        'has_appointment' => 'required|boolean',
        'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
        'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
    ]);

<a name="validating-when-present"></a>
#### 存在する場合に検証する

特定の状況では、検証対象のデータにフィールドが存在する場合にのみ検証を実行したいことがあります。これを簡単に実現するには、ルールリストに`sometimes`ルールを追加します。

    $validator = Validator::make($data, [
        'email' => 'sometimes|required|email',
    ]);

上記の例では、`email`フィールドは`$data`配列に存在する場合にのみ検証されます。

> NOTE:  
> 常に存在する必要があるが空である可能性があるフィールドを検証しようとしている場合は、[オプションフィールドに関するこの注意](#a-note-on-optional-fields)を確認してください。

<a name="complex-conditional-validation"></a>
#### 複雑な条件付き検証

より複雑な条件ロジックに基づいて検証ルールを追加したい場合があります。例えば、他のフィールドの値が100より大きい場合にのみ特定のフィールドを要求したい場合や、他のフィールドが存在する場合にのみ2つのフィールドが特定の値を持つ必要がある場合などです。これらの検証ルールを追加するのは難しくありません。まず、変更されない_静的ルール_で`Validator`インスタンスを作成します。

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

ゲームコレクター向けのWebアプリケーションを想定します。ゲームコレクターが100以上のゲームを所有している場合、なぜそんなに多くのゲームを所有しているのかを説明するように要求したいとします。例えば、ゲームのリセールショップを運営しているか、単にゲームを収集するのが好きかもしれません。この要件を条件付きで追加するには、`Validator`インスタンスの`sometimes`メソッドを使用できます。

    use Illuminate\Support\Fluent;

    $validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
        return $input->games >= 100;
    });

`sometimes`メソッドに渡される最初の引数は、条件付きで検証するフィールドの名前です。2番目の引数は追加したいルールのリストです。3番目の引数として渡されるクロージャが`true`を返す場合、ルールが追加されます。このメソッドを使用すると、複雑な条件付き検証を簡単に構築できます。複数のフィールドに対して条件付き検証を一度に追加することもできます。

    $validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
        return $input->games >= 100;
    });

> NOTE:  
> クロージャに渡される`$input`パラメータは`Illuminate\Support\Fluent`のインスタンスであり、検証対象の入力とファイルにアクセスするために使用できます。

<a name="complex-conditional-array-validation"></a>
#### 複雑な条件付き配列検証

同じネストされた配列内の他のフィールドに基づいてフィールドを検証したい場合がありますが、そのインデックスが不明です。このような状況では、クロージャが検証中の配列の現在の個々のアイテムである2番目の引数を受け取ることができます。

    $input = [
        'channels' => [
            [
                'type' => 'email',
                'address' => 'abigail@example.com',
            ],
            [
                'type' => 'url',
                'address' => 'https://example.com',
            ],
        ],
    ];

    $validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
        return $item->type === 'email';
    });

    $validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
        return $item->type !== 'email';
    });

`$input`パラメータと同様に、`$item`パラメータは属性データが配列の場合は`Illuminate\Support\Fluent`のインスタンスです。それ以外の場合は文字列です。

<a name="validating-arrays"></a>
## 配列の検証

[`array`検証ルールのドキュメント](#rule-array)で説明されているように、`array`ルールは許可された配列キーのリストを受け取ります。配列内に追加のキーが存在する場合、検証は失敗します。

    use Illuminate\Support\Facades\Validator;

    $input = [
        'user' => [
            'name' => 'Taylor Otwell',
            'username' => 'taylorotwell',
            'admin' => true,
        ],
    ];

    Validator::make($input, [
        'user' => 'array:name,username',
    ]);

一般的に、配列内に存在することが許可される配列キーを常に指定する必要があります。そうしないと、バリデータの`validate`と`validated`メソッドは、検証されたすべてのデータを返します。これには、配列とそのすべてのキーが含まれます。たとえそれらのキーが他のネストされた配列検証ルールによって検証されなかったとしてもです。

<a name="validating-nested-array-input"></a>
### ネストされた配列入力の検証

ネストされた配列ベースのフォーム入力フィールドの検証は、難しいものではありません。"ドット表記"を使用して、配列内の属性を検証できます。例えば、受信HTTPリクエストに`photos[profile]`フィールドが含まれている場合、次のように検証できます。

    use Illuminate\Support\Facades\Validator;

    $validator = Validator::make($request->all(), [
        'photos.profile' => 'required|image',
    ]);

また、配列の各要素を検証することもできます。例えば、特定の配列入力フィールド内の各メールが一意であることを検証するには、次のようにします。

    $validator = Validator::make($request->all(), [
        'person.*.email' => 'email|unique:users',
        'person.*.first_name' => 'required_with:person.*.last_name',
    ]);

同様に、[言語ファイル内のカスタム検証メッセージ](#custom-messages-for-specific-attributes)を指定する際に`*`文字を使用することで、配列ベースのフィールドに対して単一の検証メッセージを簡単に使用できます。

    'custom' => [
        'person.*.email' => [
            'unique' => '各人物は一意のメールアドレスを持つ必要があります',
        ]
    ],

<a name="accessing-nested-array-data"></a>
#### ネストされた配列データへのアクセス

検証ルールを属性に割り当てる際に、特定のネストされた配列要素の値にアクセスする必要がある場合があります。これは`Rule::forEach`メソッドを使用して実現できます。`forEach`メソッドは、検証中の配列属性の各反復に対して呼び出されるクロージャを受け取り、属性の値と完全に展開された属性名を受け取ります。クロージャは配列要素に割り当てるルールの配列を返す必要があります。

    use App\Rules\HasPermission;
    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    $validator = Validator::make($request->all(), [
        'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
            return [
                Rule::exists(Company::class, 'id'),
                new HasPermission('manage-company', $value),
            ];
        }),
    ]);

<a name="error-message-indexes-and-positions"></a>
### エラーメッセージのインデックスと位置

配列の検証時に、アプリケーションによって表示されるエラーメッセージ内で、特定の項目のインデックスまたは位置を参照したい場合があります。これを実現するには、[カスタム検証メッセージ](#manual-customizing-the-error-messages)内に`：index`（`0`から始まる）と`：position`（`1`から始まる）のプレースホルダーを含めることができます。

    use Illuminate\Support\Facades\Validator;

    $input = [
        'photos' => [
            [
                'name' => 'BeachVacation.jpg',
                'description' => 'A photo of my beach vacation!',
            ],
            [
                'name' => 'GrandCanyon.jpg',
                'description' => '',
            ],
        ],
    ];

    Validator::validate($input, [
        'photos.*.description' => 'required',
    ], [
        'photos.*.description.required' => 'Please describe photo #:position.',
    ]);

上記の例に基づくと、バリデーションは失敗し、ユーザーには_"写真 #2 を説明してください。"_というエラーが表示されます。

必要に応じて、`second-index`、`second-position`、`third-index`、`third-position`などを介して、より深くネストされたインデックスと位置を参照することができます。

```php
'photos.*.attributes.*.string' => '写真 #:second-position の属性が無効です。',
```

<a name="validating-files"></a>
## ファイルのバリデーション

Laravelは、`mimes`、`image`、`min`、`max`など、アップロードされたファイルを検証するために使用できるさまざまなバリデーションルールを提供しています。ファイルを検証する際にこれらのルールを個別に指定することもできますが、Laravelは便利なファイル検証ルールビルダーも提供しています。

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

アプリケーションがユーザーによってアップロードされた画像を受け入れる場合、`File`ルールの`image`コンストラクタメソッドを使用して、アップロードされたファイルが画像であることを示すことができます。さらに、`dimensions`ルールを使用して画像の寸法を制限することもできます。

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'photo' => [
        'required',
        File::image()
            ->min(1024)
            ->max(12 * 1024)
            ->dimensions(Rule::dimensions()->maxWidth(1000)->maxHeight(500)),
    ],
]);
```

> NOTE:  
> 画像の寸法を検証するための詳細情報は、[dimensionルールのドキュメント](#rule-dimensions)で見つけることができます。

<a name="validating-files-file-sizes"></a>
#### ファイルサイズ

便宜上、最小および最大ファイルサイズは、ファイルサイズ単位を示す接尾辞を持つ文字列として指定できます。`kb`、`mb`、`gb`、`tb`の接尾辞がサポートされています。

```php
File::image()
    ->min('1kb')
    ->max('10mb')
```

<a name="validating-files-file-types"></a>
#### ファイルタイプ

`types`メソッドを呼び出す際には拡張子のみを指定する必要がありますが、このメソッドは実際にはファイルの内容を読み取り、そのMIMEタイプを推測することでファイルのMIMEタイプを検証します。MIMEタイプとそれに対応する拡張子の完全なリストは、以下の場所で見つけることができます。

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="validating-passwords"></a>
## パスワードのバリデーション

パスワードが十分なレベルの複雑さを持っていることを確認するために、Laravelの`Password`ルールオブジェクトを使用できます。

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\Password;

$validator = Validator::make($request->all(), [
    'password' => ['required', 'confirmed', Password::min(8)],
]);
```

`Password`ルールオブジェクトを使用すると、アプリケーションのパスワードの複雑さ要件を簡単にカスタマイズできます。例えば、パスワードに少なくとも1つの文字、数字、記号、または大文字と小文字の混合が必要であることを指定できます。

```php
// 少なくとも8文字が必要...
Password::min(8)

// 少なくとも1つの文字が必要...
Password::min(8)->letters()

// 少なくとも1つの大文字と1つの小文字が必要...
Password::min(8)->mixedCase()

// 少なくとも1つの数字が必要...
Password::min(8)->numbers()

// 少なくとも1つの記号が必要...
Password::min(8)->symbols()
```

さらに、`uncompromised`メソッドを使用して、パスワードが公開されたパスワードデータ侵害で漏洩していないことを確認できます。

```php
Password::min(8)->uncompromised()
```

内部的には、`Password`ルールオブジェクトは[k-匿名性](https://en.wikipedia.org/wiki/K-anonymity)モデルを使用して、ユーザーのプライバシーやセキュリティを犠牲にすることなく、[haveibeenpwned.com](https://haveibeenpwned.com)サービスを介してパスワードが漏洩したかどうかを判断します。

デフォルトでは、パスワードがデータ侵害で少なくとも1回出現した場合、それは侵害されたと見なされます。このしきい値は、`uncompromised`メソッドの最初の引数を使用してカスタマイズできます。

```php
// パスワードが同じデータ侵害で3回以上出現しないことを確認...
Password::min(8)->uncompromised(3);
```

もちろん、上記の例のすべてのメソッドを連鎖させることができます。

```php
Password::min(8)
    ->letters()
    ->mixedCase()
    ->numbers()
    ->symbols()
    ->uncompromised()
```

<a name="defining-default-password-rules"></a>
#### デフォルトのパスワードルールの定義

アプリケーションの単一の場所でパスワードのデフォルトのバリデーションルールを指定すると便利な場合があります。これは、`Password::defaults`メソッドを使用して簡単に実現できます。このメソッドはクロージャを受け取ります。クロージャは、Passwordルールのデフォルト設定を返す必要があります。通常、`defaults`ルールは、アプリケーションのサービスプロバイダの`boot`メソッド内で呼び出す必要があります。

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
                    ? $rule->mixedCase()->uncompromised()
                    : $rule;
    });
}
```

そして、特定のパスワードのバリデーションにデフォルトのルールを適用したい場合、引数なしで`defaults`メソッドを呼び出すことができます。

```php
'password' => ['required', Password::defaults()],
```

場合によっては、デフォルトのパスワードバリデーションルールに追加のバリデーションルールを添付したいことがあります。これは、`rules`メソッドを使用して実現できます。

```php
use App\Rules\ZxcvbnRule;

Password::defaults(function () {
    $rule = Password::min(8)->rules([new ZxcvbnRule]);

    // ...
});
```

<a name="custom-validation-rules"></a>
## カスタムバリデーションルール

<a name="using-rule-objects"></a>
### ルールオブジェクトの使用

Laravelはさまざまな便利なバリデーションルールを提供していますが、独自のルールを指定したい場合があります。カスタムバリデーションルールを登録する方法の1つは、ルールオブジェクトを使用することです。新しいルールオブジェクトを生成するには、`make:rule` Artisanコマンドを使用できます。このコマンドを使用して、文字列が大文字であることを検証するルールを生成しましょう。Laravelは新しいルールを`app/Rules`ディレクトリに配置します。このディレクトリが存在しない場合、Laravelはルールを作成するArtisanコマンドを実行するときに作成します。

```shell
php artisan make:rule Uppercase
```

ルールが作成されたら、その動作を定義する準備が整いました。ルールオブジェクトには`validate`という単一のメソッドが含まれています。このメソッドは、属性名、その値、およびバリデーションエラーメッセージで失敗時に呼び出されるコールバックを受け取ります。

```php
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

ルールが定義されたら、他のバリデーションルールとともにルールオブジェクトのインスタンスを渡すことで、バリデータに添付できます。

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

#### バリデーションメッセージの翻訳

`$fail`クロージャにリテラルエラーメッセージを提供する代わりに、[翻訳文字列キー](localization.md)を提供し、Laravelにエラーメッセージを翻訳させることができます。

```php
if (strtoupper($value) !== $value) {
    $fail('validation.uppercase')->translate();
}
```

必要に応じて、プレースホルダの置換と優先言語を`translate`メソッドの最初と2番目の引数として提供できます。

```php
$fail('validation.location')->translate([
    'value' => $this->value,
], 'fr')
```

#### 追加データへのアクセス

カスタムバリデーションルールクラスが検証中のすべての他のデータにアクセスする必要がある場合、ルールクラスは`Illuminate\Contracts\Validation\DataAwareRule`インターフェースを実装できます。このインターフェースは、クラスに`setData`メソッドを定義することを要求します。このメソッドは、Laravelによって自動的に呼び出され（バリデーションが進行する前に）、検証中のすべてのデータが渡されます。

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements DataAwareRule, ValidationRule
{
    /**
     * All of the data under validation.
     *
     * @var array<string, mixed>
     */
    protected $data = [];

    // ...

    /**
     * Set the data under validation.
     *
     * @param  array<string, mixed>  $data
     */
    public function setData(array $data): static
    {
        $this->data = $data;

        return $this;
    }
}
```

または、バリデーションルールがバリデーションを実行しているバリデータインスタンスにアクセスする必要がある場合、`ValidatorAwareRule`インターフェースを実装できます。

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\ValidationRule;
use Illuminate\Contracts\Validation\ValidatorAwareRule;
use Illuminate\Validation\Validator;

class Uppercase implements ValidationRule, ValidatorAwareRule
{
    /**
     * The validator instance.
     *
     * @var \Illuminate\Validation\Validator
     */
    protected $validator;

    // ...

    /**
     * Set the current validator.
     */
    public function setValidator(Validator $validator): static
    {
        $this->validator = $validator;

        return $this;
    }
}
```

```php
        /**
         * 現在のバリデーターを設定します。
         */
        public function setValidator(Validator $validator): static
        {
            $this->validator = $validator;

            return $this;
        }
    }
```

<a name="using-closures"></a>
### クロージャの使用

アプリケーション全体でカスタムルールの機能が一度しか必要ない場合は、ルールオブジェクトの代わりにクロージャを使用できます。クロージャは属性の名前、属性の値、およびバリデーションが失敗した場合に呼び出す必要がある`$fail`コールバックを受け取ります。

```php
    use Illuminate\Support\Facades\Validator;
    use Closure;

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function (string $attribute, mixed $value, Closure $fail) {
                if ($value === 'foo') {
                    $fail("The {$attribute} is invalid.");
                }
            },
        ],
    ]);
```

<a name="implicit-rules"></a>
### 暗黙のルール

デフォルトでは、バリデーションされる属性が存在しないか、空の文字列を含む場合、通常のバリデーションルール（カスタムルールを含む）は実行されません。例えば、[`unique`](#rule-unique)ルールは空の文字列に対して実行されません。

```php
    use Illuminate\Support\Facades\Validator;

    $rules = ['name' => 'unique:users,name'];

    $input = ['name' => ''];

    Validator::make($input, $rules)->passes(); // true
```

属性が空であってもカスタムルールが実行されるようにするには、そのルールは属性が必須であることを示唆する必要があります。新しい暗黙のルールオブジェクトを素早く生成するには、`make:rule` Artisanコマンドに`--implicit`オプションを付けて使用できます。

```shell
php artisan make:rule Uppercase --implicit
```

> WARNING:  
> 「暗黙の」ルールは属性が必須であることを**示唆する**だけです。実際に欠落したり空の属性を無効とするかどうかは、あなた次第です。

