# プロンプト

- [はじめに](#introduction)
- [インストール](#installation)
- [利用可能なプロンプト](#available-prompts)
    - [テキスト](#text)
    - [テキストエリア](#textarea)
    - [パスワード](#password)
    - [確認](#confirm)
    - [選択](#select)
    - [複数選択](#multiselect)
    - [提案](#suggest)
    - [検索](#search)
    - [複数検索](#multisearch)
    - [一時停止](#pause)
- [バリデーション前の入力変換](#transforming-input-before-validation)
- [フォーム](#forms)
- [情報メッセージ](#informational-messages)
- [テーブル](#tables)
- [スピン](#spin)
- [プログレスバー](#progress)
- [ターミナルのクリア](#clear)
- [ターミナルの考慮事項](#terminal-considerations)
- [サポートされていない環境とフォールバック](#fallbacks)

<a name="introduction"></a>
## はじめに

[Laravel Prompts](https://github.com/laravel/prompts)は、プレースホルダーテキストやバリデーションなどのブラウザライクな機能を備えた、美しくユーザーフレンドリーなフォームをコマンドラインアプリケーションに追加するためのPHPパッケージです。

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Promptsは、[Artisanコンソールコマンド](artisan.md#writing-commands)でユーザー入力を受け付けるのに最適ですが、任意のコマンドラインPHPプロジェクトでも使用できます。

> NOTE:  
> Laravel PromptsはmacOS、Linux、およびWSLを使用するWindowsをサポートしています。詳細については、[サポートされていない環境とフォールバック](#fallbacks)に関するドキュメントを参照してください。

<a name="installation"></a>
## インストール

Laravel Promptsは、最新のLaravelリリースに既に含まれています。

Laravel Promptsは、Composerパッケージマネージャを使用して他のPHPプロジェクトにインストールすることもできます：

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## 利用可能なプロンプト

<a name="text"></a>
### テキスト

`text`関数は、指定された質問をユーザーに提示し、入力を受け付けてからそれを返します：

```php
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

プレースホルダーテキスト、デフォルト値、および情報ヒントを含めることもできます：

```php
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="text-required"></a>
#### 必須値

値の入力を必須にする場合は、`required`引数を渡すことができます：

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、文字列を渡すこともできます：

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### 追加のバリデーション

最後に、追加のバリデーションロジックを実行したい場合は、`validate`引数にクロージャを渡すことができます：

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

クロージャは入力された値を受け取り、バリデーションが通過した場合はエラーメッセージを返すか、`null`を返すことができます。

あるいは、Laravelの[バリデータ](validation.md)の力を活用することもできます。そのためには、属性の名前と希望するバリデーションルールを含む配列を`validate`引数に渡します：

```php
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### テキストエリア

`textarea`関数は、指定された質問をユーザーに提示し、複数行のテキストエリアを介して入力を受け付けてからそれを返します：

```php
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

プレースホルダーテキスト、デフォルト値、および情報ヒントを含めることもできます：

```php
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

<a name="textarea-required"></a>
#### 必須値

値の入力を必須にする場合は、`required`引数を渡すことができます：

```php
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、文字列を渡すこともできます：

```php
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

<a name="textarea-validation"></a>
#### 追加のバリデーション

最後に、追加のバリデーションロジックを実行したい場合は、`validate`引数にクロージャを渡すことができます：

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

クロージャは入力された値を受け取り、バリデーションが通過した場合はエラーメッセージを返すか、`null`を返すことができます。

あるいは、Laravelの[バリデータ](validation.md)の力を活用することもできます。そのためには、属性の名前と希望するバリデーションルールを含む配列を`validate`引数に渡します：

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="password"></a>
### パスワード

`password`関数は`text`関数に似ていますが、ユーザーの入力はコンソールに入力されるとマスクされます。これは、パスワードなどの機密情報を尋ねる際に便利です：

```php
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

プレースホルダーテキストと情報ヒントを含めることもできます：

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### 必須値

値の入力を必須にする場合は、`required`引数を渡すことができます：

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、文字列を渡すこともできます：

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### 追加のバリデーション

最後に、追加のバリデーションロジックを実行したい場合は、`validate`引数にクロージャを渡すことができます：

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

クロージャは入力された値を受け取り、バリデーションが通過した場合はエラーメッセージを返すか、`null`を返すことができます。

あるいは、Laravelの[バリデータ](validation.md)の力を活用することもできます。そのためには、属性の名前と希望するバリデーションルールを含む配列を`validate`引数に渡します：

```php
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### 確認

ユーザーに「はい」または「いいえ」の確認を求める必要がある場合は、`confirm`関数を使用できます。ユーザーは矢印キーを使用するか、`y`または`n`を押して応答を選択できます。この関数は`true`または`false`を返します。

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

デフォルト値、「はい」と「いいえ」のラベルのカスタマイズ、および情報ヒントを含めることもできます：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

<a name="confirm-required"></a>
#### 「はい」を必須にする

必要に応じて、`required`引数を渡すことでユーザーに「はい」を選択させることができます：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、文字列を渡すこともできます：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### 選択

ユーザーに事前定義された選択肢から選択させる必要がある場合は、`select`関数を使用できます：

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

デフォルトの選択肢と情報ヒントを指定することもできます：

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

`options`引数に連想配列を渡すことで、選択されたキーを値の代わりに返すこともできます：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

最大5つのオプションが表示されると、リストはスクロールを開始します。これをカスタマイズするには、`scroll`引数を渡します：

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-validation"></a>
#### 追加のバリデーション

他のプロンプト関数とは異なり、`select`関数は`required`引数を受け付けません。これは、何も選択できないためです。ただし、オプションを提示したいが選択を防ぎたい場合は、`validate`引数にクロージャを渡すことができます：

```php
$role = select(
    label: 'ユーザーにどの役割を割り当てますか？',
    options: [
        'member' => 'メンバー',
        'contributor' => 'コントリビューター',
        'owner' => 'オーナー',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'オーナーは既に存在します。'
            : null
);
```

`options`引数が連想配列の場合、クロージャは選択されたキーを受け取ります。そうでない場合は、選択された値を受け取ります。クロージャはエラーメッセージを返すか、バリデーションが通過した場合は`null`を返すことができます。

<a name="multiselect"></a>
### 複数選択

ユーザーが複数のオプションを選択できるようにする必要がある場合、`multiselect`関数を使用できます：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'どの権限を割り当てますか？',
    options: ['読み取り', '作成', '更新', '削除']
);
```

デフォルトの選択肢と情報ヒントを指定することもできます：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'どの権限を割り当てますか？',
    options: ['読み取り', '作成', '更新', '削除'],
    default: ['読み取り', '作成'],
    hint: '権限はいつでも更新できます。'
);
```

`options`引数に連想配列を渡して、選択されたオプションのキーを返すようにすることもできます：

```php
$permissions = multiselect(
    label: 'どの権限を割り当てますか？',
    options: [
        'read' => '読み取り',
        'create' => '作成',
        'update' => '更新',
        'delete' => '削除',
    ],
    default: ['read', 'create']
);
```

リストがスクロールを開始する前に最大5つのオプションが表示されます。これをカスタマイズするには、`scroll`引数を渡すことができます：

```php
$categories = multiselect(
    label: 'どのカテゴリを割り当てますか？',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-required"></a>
#### 値の必須化

デフォルトでは、ユーザーは0個以上のオプションを選択できます。代わりに1つ以上のオプションを強制するには、`required`引数を渡すことができます：

```php
$categories = multiselect(
    label: 'どのカテゴリを割り当てますか？',
    options: Category::pluck('name', 'id'),
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、`required`引数に文字列を渡すことができます：

```php
$categories = multiselect(
    label: 'どのカテゴリを割り当てますか？',
    options: Category::pluck('name', 'id'),
    required: '少なくとも1つのカテゴリを選択する必要があります'
);
```

<a name="multiselect-validation"></a>
#### 追加のバリデーション

オプションを提示するが選択を防ぐ必要がある場合、`validate`引数にクロージャを渡すことができます：

```php
$permissions = multiselect(
    label: 'ユーザーにどの権限を割り当てますか？',
    options: [
        'read' => '読み取り',
        'create' => '作成',
        'update' => '更新',
        'delete' => '削除',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'すべてのユーザーは読み取り権限を必要とします。'
        : null
);
```

`options`引数が連想配列の場合、クロージャは選択されたキーを受け取ります。そうでない場合は、選択された値を受け取ります。クロージャはエラーメッセージを返すか、バリデーションが通過した場合は`null`を返すことができます。

<a name="suggest"></a>
### 提案

`suggest`関数は、可能な選択肢に対して自動補完を提供するために使用できます。ユーザーは自動補完のヒントに関係なく、任意の回答を提供できます：

```php
use function Laravel\Prompts\suggest;

$name = suggest('あなたの名前は何ですか？', ['Taylor', 'Dayle']);
```

また、`suggest`関数の2番目の引数としてクロージャを渡すこともできます。クロージャは、ユーザーが入力文字を入力するたびに呼び出されます。クロージャは、ユーザーの入力を含む文字列パラメータを受け取り、自動補完のためのオプションの配列を返す必要があります：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

プレースホルダーテキスト、デフォルト値、情報ヒントを含めることもできます：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: ['Taylor', 'Dayle'],
    placeholder: '例：Taylor',
    default: $user?->name,
    hint: 'これはあなたのプロフィールに表示されます。'
);
```

<a name="suggest-required"></a>
#### 必須の値

値の入力を必須にする場合、`required`引数を渡すことができます：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、文字列を渡すこともできます：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: ['Taylor', 'Dayle'],
    required: '名前は必須です。'
);
```

<a name="suggest-validation"></a>
#### 追加のバリデーション

最後に、追加のバリデーションロジックを実行したい場合、`validate`引数にクロージャを渡すことができます：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => '名前は少なくとも3文字である必要があります。',
        strlen($value) > 255 => '名前は255文字を超えてはいけません。',
        default => null
    }
);
```

クロージャは入力された値を受け取り、エラーメッセージを返すか、バリデーションが通過した場合は`null`を返すことができます。

また、Laravelの[バリデータ](validation.md)の力を活用することもできます。そのためには、属性の名前と希望するバリデーションルールを含む配列を`validate`引数に渡します：

```php
$name = suggest(
    label: 'あなたの名前は何ですか？',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### 検索

ユーザーが選択するためのオプションが多い場合、`search`関数を使用すると、ユーザーは検索クエリを入力して結果をフィルタリングし、矢印キーを使用してオプションを選択できます：

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'メールを受け取るユーザーを検索',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

クロージャは、ユーザーがこれまでに入力したテキストを受け取り、オプションの配列を返す必要があります。連想配列を返す場合、選択されたオプションのキーが返されます。そうでない場合、その値が返されます。

プレースホルダーテキストと情報ヒントを含めることもできます：

```php
$id = search(
    label: 'メールを受け取るユーザーを検索',
    placeholder: '例：Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'ユーザーはすぐにメールを受け取ります。'
);
```

リストがスクロールを開始する前に最大5つのオプションが表示されます。これをカスタマイズするには、`scroll`引数を渡すことができます：

```php
$id = search(
    label: 'メールを受け取るユーザーを検索',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-validation"></a>
#### 追加のバリデーション

追加のバリデーションロジックを実行したい場合、`validate`引数にクロージャを渡すことができます：

```php
$id = search(
    label: 'メールを受け取るユーザーを検索',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'このユーザーはメールの受信をオプトアウトしています。';
        }
    }
);
```

`options`クロージャが連想配列を返す場合、クロージャは選択されたキーを受け取ります。そうでない場合、選択された値を受け取ります。クロージャはエラーメッセージを返すか、バリデーションが通過した場合は`null`を返すことができます。

<a name="multisearch"></a>
### 複数検索

検索可能なオプションが多く、ユーザーが複数の項目を選択できるようにする必要がある場合、`multisearch`関数を使用すると、ユーザーは検索クエリを入力して結果をフィルタリングし、矢印キーとスペースバーを使用してオプションを選択できます：

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'メールを受け取るユーザーを検索',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

クロージャは、ユーザーがこれまでに入力したテキストを受け取り、オプションの配列を返す必要があります。連想配列を返す場合、選択されたオプションのキーが返されます。そうでない場合、その値が返されます。

プレースホルダーテキストと情報ヒントを含めることもできます：

```php
$ids = multisearch(
    label: 'メールを受け取るユーザーを検索',
    placeholder: '例：Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'ユーザーはすぐにメールを受け取ります。'
);
```

リストがスクロールを開始する前に最大5つのオプションが表示されます。これをカスタマイズするには、`scroll`引数を提供することができます：

```php
$ids = multisearch(
    label: 'メールを受け取るユーザーを検索',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-required"></a>
#### 値の必須化

デフォルトでは、ユーザーはゼロ以上のオプションを選択できます。代わりに1つ以上のオプションを強制するために、`required`引数を渡すことができます:

```php
$ids = multisearch(
    label: 'メールを受け取るユーザーを検索してください',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

バリデーションメッセージをカスタマイズしたい場合は、`required`引数に文字列を渡すこともできます:

```php
$ids = multisearch(
    label: 'メールを受け取るユーザーを検索してください',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: '少なくとも1人のユーザーを選択してください。'
);
```

<a name="multisearch-validation"></a>
#### 追加のバリデーション

追加のバリデーションロジックを実行したい場合は、`validate`引数にクロージャを渡すことができます:

```php
$ids = multisearch(
    label: 'メールを受け取るユーザーを検索してください',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' はオプトアウトしています。';
        }
    }
);
```

`options`クロージャが連想配列を返す場合、クロージャは選択されたキーを受け取ります。それ以外の場合は、選択された値を受け取ります。クロージャはエラーメッセージを返すか、バリデーションが通過した場合は`null`を返すことができます。

<a name="pause"></a>
### 一時停止

`pause`関数は、ユーザーに情報テキストを表示し、Enter / Returnキーを押して続行することを確認するために待機するために使用できます:

```php
use function Laravel\Prompts\pause;

pause('続行するにはENTERを押してください。');
```

<a name="transforming-input-before-validation"></a>
## バリデーション前の入力変換

バリデーションが行われる前にプロンプトの入力を変換したい場合があります。例えば、提供された文字列から空白を削除したい場合です。これを実現するために、多くのプロンプト関数はクロージャを受け取る`transform`引数を提供します:

```php
$name = text(
    label: 'あなたの名前は何ですか？',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => '名前は少なくとも3文字以上である必要があります。',
        strlen($value) > 255 => '名前は255文字を超えてはいけません。',
        default => null
    }
);
```

<a name="forms"></a>
## フォーム

多くの場合、追加のアクションを実行する前に情報を収集するために、複数のプロンプトが順番に表示されます。`form`関数を使用して、ユーザーが完了するためのグループ化されたプロンプトセットを作成できます:

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('あなたの名前は何ですか？', required: true)
    ->password('あなたのパスワードは何ですか？', validate: ['password' => 'min:8'])
    ->confirm('利用規約に同意しますか？')
    ->submit();
```

`submit`メソッドは、フォームのプロンプトからのすべての応答を含む数値インデックス付き配列を返します。ただし、各プロンプトに`name`引数を介して名前を提供することができます。名前が提供されると、名前付きプロンプトの応答はその名前を介してアクセスできます:

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('あなたの名前は何ですか？', required: true, name: 'name')
    ->password(
        label: 'あなたのパスワードは何ですか？',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('利用規約に同意しますか？')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

`form`関数を使用する主な利点は、ユーザーが`CTRL + U`を使用してフォーム内の前のプロンプトに戻ることができることです。これにより、ユーザーはミスを修正したり、選択を変更したりすることができ、フォーム全体をキャンセルして再開する必要がなくなります。

フォーム内のプロンプトをより細かく制御する必要がある場合は、プロンプト関数を直接呼び出す代わりに`add`メソッドを呼び出すことができます。`add`メソッドは、ユーザーが提供した以前のすべての応答を渡されます:

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;

$responses = form()
    ->text('あなたの名前は何ですか？', required: true, name: 'name')
    ->add(function ($responses) {
        return text("{$responses['name']}さんは何歳ですか？");
    }, name: 'age')
    ->submit();

outro("あなたの名前は{$responses['name']}で、{$responses['age']}歳です。");
```

<a name="informational-messages"></a>
## 情報メッセージ

`note`、`info`、`warning`、`error`、`alert`関数を使用して情報メッセージを表示できます:

```php
use function Laravel\Prompts\info;

info('パッケージが正常にインストールされました。');
```

<a name="tables"></a>
## テーブル

`table`関数を使用すると、複数の行と列のデータを簡単に表示できます。必要なのは、列名とテーブルのデータを提供することだけです:

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## スピン

`spin`関数は、指定されたコールバックを実行しながらスピナーとオプションのメッセージを表示します。これは、進行中のプロセスを示し、完了時にコールバックの結果を返します:

```php
use function Laravel\Prompts\spin;

$response = spin(
    message: '応答を取得中...',
    callback: fn () => Http::get('http://example.com')
);
```

> WARNING:  
> `spin`関数は、スピナーをアニメーション化するために`pcntl` PHP拡張機能を必要とします。この拡張機能が利用できない場合、スピナーの静的バージョンが表示されます。

<a name="progress"></a>
## プログレスバー

長時間実行されるタスクの場合、タスクがどの程度完了しているかをユーザーに知らせるプログレスバーを表示すると便利です。`progress`関数を使用すると、Laravelはプログレスバーを表示し、指定された反復可能な値を反復するたびに進行させます:

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'ユーザーを更新中',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

`progress`関数はマップ関数のように動作し、コールバックの各反復の戻り値を含む配列を返します。

コールバックは`Laravel\Prompts\Progress`インスタンスも受け取ることができ、各反復でラベルとヒントを変更できます:

```php
$users = progress(
    label: 'ユーザーを更新中',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("{$user->name}を更新中")
            ->hint("作成日: {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'これには時間がかかる場合があります。'
);
```

プログレスバーの進行をより手動で制御する必要がある場合があります。まず、プロセスが反復するステップの総数を定義します。次に、各アイテムを処理した後に`advance`メソッドを介してプログレスバーを進行させます:

```php
$progress = progress(label: 'ユーザーを更新中', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="clear"></a>
## ターミナルのクリア

`clear`関数を使用してユーザーのターミナルをクリアできます:

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## ターミナルの考慮事項

<a name="terminal-width"></a>
#### ターミナルの幅

ラベル、オプション、またはバリデーションメッセージの長さがユーザーのターミナルの「列」数を超える場合、自動的に切り詰められて適合します。ユーザーがより狭いターミナルを使用している可能性がある場合、これらの文字列の長さを最小限に抑えることを検討してください。一般的に安全な最大長は、80文字のターミナルをサポートするための74文字です。

<a name="terminal-height"></a>
#### ターミナルの高さ

`scroll`引数を受け取るプロンプトの場合、設定された値は自動的にユーザーのターミナルの高さに合わせて調整され、バリデーションメッセージのスペースも含まれます。

<a name="fallbacks"></a>
## サポートされていない環境とフォールバック

Laravel PromptsはmacOS、Linux、およびWSLを使用したWindowsをサポートしています。Windows版のPHPの制限により、WSL以外のWindowsでLaravel Promptsを使用することは現在不可能です。

このため、Laravel Promptsは[Symfony Console Question Helper](https://symfony.com/doc/7.0/components/console/helpers/questionhelper.html)などの代替実装にフォールバックすることができます。

> NOTE:  
> LaravelフレームワークでLaravel Promptsを使用する場合、各プロンプトのフォールバックが設定されており、サポートされていない環境で自動的に有効になります。

<a name="fallback-conditions"></a>
#### フォールバック条件

Laravelを使用していない場合や、フォールバック動作が使用されるタイミングをカスタマイズする必要がある場合、`Prompt`クラスの`fallbackWhen`静的メソッドにブール値を渡すことができます:

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### フォールバック動作

Laravelを使用していない場合や、フォールバック動作をカスタマイズする必要がある場合、各プロンプトクラスの`fallbackUsing`静的メソッドにクロージャを渡すことができます:

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) {
    $question = new Question($prompt->label, $prompt->default);
    $question->setValidator(function ($answer) use ($prompt) {
        if ($prompt->required && $answer === null) {
            throw new \RuntimeException('This value is required.');
        }

        if ($prompt->validate) {
            $error = ($prompt->validate)($answer);

            if ($error) {
                throw new \RuntimeException($error);
            }
        }

        return $answer;
    });

    return (new SymfonyStyle(Input::createFromGlobals(), Output::createFromGlobals()))
        ->askQuestion($question);
});
```

```php
TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : '必須項目です。'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

フォールバックは、各プロンプトクラスに対して個別に設定する必要があります。クロージャはプロンプトクラスのインスタンスを受け取り、プロンプトに適した型を返す必要があります。
```

