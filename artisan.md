# Artisanコンソール

- [はじめに](#introduction)
    - [Tinker (REPL)](#tinker)
- [コマンドの作成](#writing-commands)
    - [コマンドの生成](#generating-commands)
    - [コマンドの構造](#command-structure)
    - [クロージャコマンド](#closure-commands)
    - [分離可能なコマンド](#isolatable-commands)
- [入力の期待値の定義](#defining-input-expectations)
    - [引数](#arguments)
    - [オプション](#options)
    - [入力配列](#input-arrays)
    - [入力の説明](#input-descriptions)
    - [不足している入力のプロンプト](#prompting-for-missing-input)
- [コマンドのI/O](#command-io)
    - [入力の取得](#retrieving-input)
    - [入力のプロンプト](#prompting-for-input)
    - [出力の書き込み](#writing-output)
- [コマンドの登録](#registering-commands)
- [プログラムによるコマンドの実行](#programmatically-executing-commands)
    - [他のコマンドからのコマンド呼び出し](#calling-commands-from-other-commands)
- [シグナルハンドリング](#signal-handling)
- [スタブのカスタマイズ](#stub-customization)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

Artisanは、Laravelに含まれるコマンドラインインターフェースです。Artisanはアプリケーションのルートに`artisan`スクリプトとして存在し、アプリケーションを構築する際に役立つ多くの便利なコマンドを提供します。利用可能なすべてのArtisanコマンドのリストを表示するには、`list`コマンドを使用できます。

```shell
php artisan list
```

すべてのコマンドには、コマンドの利用可能な引数とオプションを表示および説明する「ヘルプ」画面も含まれています。ヘルプ画面を表示するには、コマンド名の前に`help`を付けます。

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

ローカル開発環境として[Laravel Sail](sail.md)を使用している場合は、Artisanコマンドを呼び出す際に`sail`コマンドラインを使用することを忘れないでください。Sailは、アプリケーションのDockerコンテナ内でArtisanコマンドを実行します。

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinkerは、[PsySH](https://github.com/bobthecow/psysh)パッケージを利用した、Laravelフレームワーク用の強力なREPLです。

<a name="installation"></a>
#### インストール

すべてのLaravelアプリケーションにはデフォルトでTinkerが含まれています。ただし、以前にアプリケーションからTinkerを削除した場合は、Composerを使用してTinkerをインストールできます。

```shell
composer require laravel/tinker
```

> NOTE:  
> Laravelアプリケーションと対話する際にホットリロード、複数行コード編集、オートコンプリートを探していますか？[Tinkerwell](https://tinkerwell.app)をチェックしてください！

<a name="usage"></a>
#### 使用方法

Tinkerを使用すると、Eloquentモデル、ジョブ、イベントなど、Laravelアプリケーション全体をコマンドラインで操作できます。Tinker環境に入るには、`tinker` Artisanコマンドを実行します。

```shell
php artisan tinker
```

Tinkerの設定ファイルを公開するには、`vendor:publish`コマンドを使用します。

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> WARNING:  
> `dispatch`ヘルパ関数と`Dispatchable`クラスの`dispatch`メソッドは、ジョブをキューに配置するためにガベージコレクションに依存しています。したがって、tinkerを使用する場合は、ジョブをディスパッチするために`Bus::dispatch`または`Queue::push`を使用する必要があります。

<a name="command-allow-list"></a>
#### コマンド許可リスト

Tinkerは、どのArtisanコマンドがそのシェル内で実行できるかを決定するために「許可」リストを使用します。デフォルトでは、`clear-compiled`、`down`、`env`、`inspire`、`migrate`、`migrate:install`、`up`、`optimize`コマンドを実行できます。さらに多くのコマンドを許可したい場合は、`tinker.php`設定ファイルの`commands`配列に追加できます。

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>
#### エイリアス化すべきでないクラス



通常、Tinker はクラスと対話する際に自動的にクラスにエイリアスを付けます。しかし、一部のクラスにはエイリアスを付けたくない場合があります。これは、`tinker.php` 設定ファイルの `dont_alias` 配列にクラスをリストすることで実現できます:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>
## コマンドの作成

Artisan に付属するコマンドに加えて、独自のカスタムコマンドを作成することもできます。コマンドは通常 `app/Console/Commands` ディレクトリに保存されますが、Composer でコマンドをロードできる限り、保存場所は自由に選択できます。

<a name="generating-commands"></a>
### コマンドの生成

新しいコマンドを作成するには、`make:command` Artisan コマンドを使用できます。このコマンドは、`app/Console/Commands` ディレクトリに新しいコマンドクラスを作成します。このディレクトリがアプリケーションに存在しない場合でも心配はいりません - `make:command` Artisan コマンドを初めて実行したときに作成されます:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### コマンドの構造

コマンドを生成した後、クラスの `signature` および `description` プロパティに適切な値を定義する必要があります。これらのプロパティは、コマンドを `list` 画面に表示する際に使用されます。`signature` プロパティでは、[コマンドの入力期待値](#defining-input-expectations)を定義することもできます。コマンドが実行されると、`handle` メソッドが呼び出されます。コマンドのロジックはこのメソッドに配置できます。

例として、コマンドを見てみましょう。コマンドの `handle` メソッドを介して必要な依存関係を要求できることに注意してください。Laravel の [サービスコンテナ](container.md) は、このメソッドのシグネチャで型ヒントされたすべての依存関係を自動的に注入します:

    <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * コンソールコマンドの名前とシグネチャ。
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    /**
     * コンソールコマンドの説明。
     *
     * @var string
     */
    protected $description = 'ユーザーにマーケティングメールを送信する';

    /**
     * コンソールコマンドを実行する。
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}

> NOTE:  
> コードの再利用性を高めるために、コンソールコマンドを軽量に保ち、タスクを実行するためにアプリケーションサービスに処理を委譲することが良いプラクティスです。上記の例では、メール送信という「重い処理」を行うためにサービスクラスを注入していることに注意してください。

<a name="exit-codes"></a>
#### 終了コード

`handle` メソッドから何も返されず、コマンドが正常に実行された場合、コマンドは成功を示す `0` の終了コードで終了します。ただし、`handle` メソッドはオプションで整数を返すことで、コマンドの終了コードを手動で指定することができます。

```php
$this->error('何かがうまくいかなかった。');

return 1;
```

コマンド内の任意のメソッドからコマンドを「失敗」させたい場合は、`fail` メソッドを利用できます。`fail` メソッドはコマンドの実行を即座に終了させ、終了コード `1` を返します。

```php
$this->fail('何かがうまくいかなかった。');
```

<a name="closure-commands"></a>
### クロージャコマンド

クロージャベースのコマンドは、コンソールコマンドをクラスとして定義する代わりの方法を提供します。ルートクロージャがコントローラの代わりになるのと同様に、コマンドクロージャはコマンドクラスの代わりになると考えてください。

`routes/console.php`ファイルはHTTPルートを定義しませんが、コンソールベースのエントリポイント（ルート）をアプリケーションに定義します。このファイル内で、`Artisan::command`メソッドを使用して、すべてのクロージャベースのコンソールコマンドを定義できます。`command`メソッドは2つの引数を受け取ります：[コマンドシグネチャ](#defining-input-expectations)と、コマンドの引数とオプションを受け取るクロージャです：

    Artisan::command('mail:send {user}', function (string $user) {
        $this->info("Sending email to: {$user}!");
    });

クロージャは基礎となるコマンドインスタンスにバインドされているため、通常は完全なコマンドクラスでアクセスできるすべてのヘルパーメソッドに完全にアクセスできます。

<a name="type-hinting-dependencies"></a>
#### 依存関係のタイプヒント

コマンドの引数とオプションを受け取るだけでなく、コマンドクロージャは、[サービスコンテナ](container.md)から解決したい追加の依存関係をタイプヒントすることもできます：

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>
#### クロージャコマンドの説明

クロージャベースのコマンドを定義する際に、`purpose`メソッドを使用してコマンドに説明を追加できます。この説明は、`php artisan list`または`php artisan help`コマンドを実行したときに表示されます：

    Artisan::command('mail:send {user}', function (string $user) {
        // ...
    })->purpose('Send a marketing email to a user');

<a name="isolatable-commands"></a>
### 分離可能なコマンド

> WARNING:  
> この機能を利用するには、アプリケーションが`memcached`、`redis`、`dynamodb`、`database`、`file`、または`array`キャッシュドライバをアプリケーションのデフォルトキャッシュドライバとして使用している必要があります。さらに、すべてのサーバーが同じ中央キャッシュサーバーと通信している必要があります。

場合によっては、一度に1つのコマンドインスタンスのみが実行されるようにしたいことがあります。これを実現するために、コマンドクラスに `Illuminate\Contracts\Console\Isolatable` インターフェースを実装することができます。

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
}
```

コマンドが `Isolatable` としてマークされると、Laravelは自動的にコマンドに `--isolated` オプションを追加します。そのオプションを指定してコマンドが呼び出されると、Laravelはそのコマンドの他のインスタンスが既に実行されていないことを確認します。Laravelは、アプリケーションのデフォルトのキャッシュドライバを使用してアトミックロックを取得することでこれを実現します。もし他のインスタンスが実行中であれば、コマンドは実行されませんが、コマンドは成功の終了ステータスコードで終了します。

```shell
php artisan mail:send 1 --isolated
```

コマンドが実行できなかった場合に返す終了ステータスコードを指定したい場合は、`isolated` オプションを介して希望のステータスコードを指定できます。

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-id"></a>
#### ロックID

デフォルトでは、Laravelはコマンドの名前を使用して、アプリケーションのキャッシュでアトミックロックを取得するために使用される文字列キーを生成します。ただし、Artisanコマンドクラスに `isolatableId` メソッドを定義することでこのキーをカスタマイズできます。これにより、コマンドの引数やオプションをキーに組み込むことができます。

```php
/**
 * コマンドの分離可能なIDを取得します。
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

<a name="lock-expiration-time"></a>
#### ロックの有効期限

ロックの有効期限については、デフォルトではLaravelが自動的に管理しますが、必要に応じてカスタマイズすることも可能です。

デフォルトでは、隔離ロックはコマンドが終了した後に期限切れになります。または、コマンドが中断されて終了できない場合、ロックは1時間後に期限切れになります。ただし、コマンドに `isolationLockExpiresAt` メソッドを定義することで、ロックの有効期限を調整できます。

```php
use DateTimeInterface;
use DateInterval;

/**
 * コマンドの隔離ロックがいつ期限切れになるかを決定します。
 */
public function isolationLockExpiresAt(): DateTimeInterface|DateInterval
{
    return now()->addMinutes(5);
}
```

<a name="defining-input-expectations"></a>
## 入力の期待値の定義

コンソールコマンドを書く際、引数やオプションを通じてユーザーからの入力を収集することが一般的です。Laravelでは、コマンドの `signature` プロパティを使用して、ユーザーから期待する入力を定義することが非常に便利です。`signature` プロパティを使用すると、コマンドの名前、引数、オプションを1つの表現力豊かなルートのような構文で定義できます。

<a name="arguments"></a>
### 引数

ユーザーが提供するすべての引数とオプションは、中括弧で囲まれます。以下の例では、コマンドは1つの必須引数 `user` を定義しています。

    /**
     * コンソールコマンドの名前とシグネチャ。
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

引数をオプションにしたり、引数にデフォルト値を定義することもできます。

    // オプションの引数...
    'mail:send {user?}'

    // デフォルト値を持つオプションの引数...
    'mail:send {user=foo}'

<a name="options"></a>
### オプション

オプションは、引数と同様に、ユーザー入力の別の形式です。オプションは、コマンドラインから提供されるときに2つのハイフン (`--`) が前に付きます。オプションには、値を受け取るものと受け取らないものの2種類があります。値を受け取らないオプションは、ブール値の "スイッチ" として機能します。このタイプのオプションの例を見てみましょう。

```php
/**
 * コンソールコマンドの名前とシグネチャ。
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```

この例では、Artisanコマンドを呼び出す際に`--queue`スイッチを指定することができます。`--queue`スイッチが渡された場合、オプションの値は`true`になります。それ以外の場合、値は`false`になります：

```shell
php artisan mail:send 1 --queue
```

<a name="options-with-values"></a>
#### 値を持つオプション

次に、値を期待するオプションを見てみましょう。ユーザーがオプションに値を指定する必要がある場合、オプション名の後に`=`記号を付ける必要があります：

```php
/**
 * コンソールコマンドの名前とシグネチャ。
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```

この例では、ユーザーは次のようにオプションに値を渡すことができます。オプションがコマンドの呼び出し時に指定されない場合、その値は`null`になります：

```shell
php artisan mail:send 1 --queue=default
```

オプション名の後にデフォルト値を指定することで、オプションにデフォルト値を割り当てることができます。ユーザーがオプションの値を渡さない場合、デフォルト値が使用されます：

```php
'mail:send {user} {--queue=default}'
```

<a name="option-shortcuts"></a>
#### オプションのショートカット

オプションを定義する際にショートカットを割り当てるには、オプション名の前に指定し、`|`文字を使用してショートカットと完全なオプション名を区切ることができます：

```php
'mail:send {user} {--Q|queue}'
```

ターミナルでコマンドを呼び出す際、オプションのショートカットには単一のハイフンを付け、オプションの値を指定する際に`=`記号を含めるべきではありません：

```shell
php artisan mail:send 1 -Qdefault
```

<a name="input-arrays"></a>
### 入力配列

複数の入力値を期待する引数やオプションを定義したい場合、`*`文字を使用することができます。まず、そのような引数を指定する例を見てみましょう：

```php
'mail:send {user*}'
```

このメソッドを呼び出す際、`user`引数はコマンドラインに順番に渡すことができます。例えば、以下のコマンドは`user`の値を`1`と`2`を値とする配列に設定します。

```shell
php artisan mail:send 1 2
```

この`*`文字は、オプションの引数定義と組み合わせて、引数のゼロ以上のインスタンスを許可するために使用できます。

    'mail:send {user?*}'

<a name="option-arrays"></a>
#### オプション配列

複数の入力値を期待するオプションを定義する場合、コマンドに渡される各オプション値にはオプション名をプレフィックスとして付ける必要があります。

    'mail:send {--id=*}'

このようなコマンドは、複数の`--id`引数を渡すことで呼び出すことができます。

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### 入力の説明

引数名と説明をコロンで区切ることで、入力引数とオプションに説明を割り当てることができます。コマンドの定義に少し余裕が必要な場合は、定義を複数行に広げても構いません。

    /**
     * コンソールコマンドの名前とシグネチャ。
     *
     * @var string
     */
    protected $signature = 'mail:send
                            {user : ユーザーのID}
                            {--queue : ジョブをキューに入れるかどうか}';

<a name="prompting-for-missing-input"></a>
### 不足している入力のプロンプト

コマンドに必須の引数が含まれている場合、ユーザーはそれらが提供されないとエラーメッセージを受け取ります。代わりに、`PromptsForMissingInput`インターフェースを実装することで、必須の引数が欠落している場合に自動的にユーザーにプロンプトを表示するようにコマンドを設定できます。

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Contracts\Console\PromptsForMissingInput;


```php
class SendEmails extends Command implements PromptsForMissingInput
{
    /**
     * コンソールコマンドの名前とシグネチャ。
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    // ...
}
```

Laravelがユーザーから必須の引数を収集する必要がある場合、引数名または説明を使用して賢く質問を組み立て、ユーザーに自動的に質問します。必須の引数を収集するために使用される質問をカスタマイズしたい場合は、`promptForMissingArgumentsUsing`メソッドを実装し、引数名をキーとする質問の配列を返すことができます。

```php
/**
 * 不足している入力引数に対して返される質問を使用してプロンプトを表示します。
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'どのユーザーIDにメールを送信すべきですか？',
    ];
}
```

プレースホルダーテキストを提供するには、質問とプレースホルダーを含むタプルを使用します。

```php
return [
    'user' => ['どのユーザーIDにメールを送信すべきですか？', '例: 123'],
];
```

プロンプトを完全に制御したい場合は、ユーザーにプロンプトを表示し、その回答を返すクロージャを提供することができます。

```php
use App\Models\User;
use function Laravel\Prompts\search;

// ...

return [
    'user' => fn () => search(
        label: 'ユーザーを検索:',
        placeholder: '例: Taylor Otwell',
        options: fn ($value) => strlen($value) > 0
            ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
            : []
    ),
];
```

> NOTE:  
> 利用可能なプロンプトとその使用方法に関する詳細情報は、[Laravel Prompts](prompts.md)の包括的なドキュメントに記載されています。

ユーザーに[オプション](#options)を選択または入力するよう促したい場合、コマンドの`handle`メソッドにプロンプトを含めることができます。ただし、ユーザーが不足している引数について自動的にプロンプトされた場合にのみユーザーにプロンプトを表示したい場合は、`afterPromptingForMissingArguments`メソッドを実装することができます。

```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\confirm;

// ...

/**
 * ユーザーが不足している引数についてプロンプトされた後にアクションを実行する。
 */
protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
{
    $input->setOption('queue', confirm(
        label: 'Would you like to queue the mail?',
        default: $this->option('queue')
    ));
}
```

<a name="command-io"></a>
## コマンドの入出力

<a name="retrieving-input"></a>
### 入力の取得

コマンドの実行中に、コマンドが受け入れる引数とオプションの値にアクセスする必要がある場合があります。そのためには、`argument`メソッドと`option`メソッドを使用できます。引数またはオプションが存在しない場合、`null`が返されます。

```php
/**
 * コンソールコマンドを実行する。
 */
public function handle(): void
{
    $userId = $this->argument('user');
}
```

すべての引数を`array`として取得する場合は、`arguments`メソッドを呼び出します。

```php
$arguments = $this->arguments();
```

オプションは引数と同様に`option`メソッドを使用して取得できます。すべてのオプションを配列として取得するには、`options`メソッドを呼び出します。

```php
// 特定のオプションを取得...
$queueName = $this->option('queue');

// すべてのオプションを配列として取得...
$options = $this->options();
```

<a name="prompting-for-input"></a>
### 入力のプロンプト

ユーザーに入力を求めるプロンプトを表示する方法については、以下のセクションで詳しく説明します。

> NOTE:  
> [Laravel Prompts](prompts.md) は、コマンドラインアプリケーションに美しくユーザーフレンドリーなフォームを追加するためのPHPパッケージです。プレースホルダーテキストやバリデーションを含むブラウザのような機能を備えています。

出力を表示するだけでなく、コマンドの実行中にユーザーに入力を求めることもできます。`ask`メソッドは、指定された質問をユーザーに提示し、その入力を受け取り、その後ユーザーの入力をコマンドに返します。

    /**
     * コンソールコマンドを実行する。
     */
    public function handle(): void
    {
        $name = $this->ask('あなたの名前は何ですか？');

        // ...
    }

`ask`メソッドは、ユーザーが入力を提供しなかった場合に返されるデフォルト値を指定するためのオプションの第二引数も受け入れます。

    $name = $this->ask('あなたの名前は何ですか？', 'Taylor');

`secret`メソッドは`ask`に似ていますが、ユーザーがコンソールに入力する際にその入力が表示されません。このメソッドは、パスワードなどの機密情報を尋ねる際に便利です。

    $password = $this->secret('パスワードは何ですか？');

<a name="asking-for-confirmation"></a>
#### 確認を求める

ユーザーに簡単な「はいかいいえ」の確認を求める必要がある場合、`confirm`メソッドを使用できます。デフォルトでは、このメソッドは`false`を返します。ただし、ユーザーがプロンプトに対して`y`または`yes`と入力した場合、メソッドは`true`を返します。

    if ($this->confirm('続行しますか？')) {
        // ...
    }

必要に応じて、確認プロンプトがデフォルトで`true`を返すように指定するには、`confirm`メソッドの第二引数に`true`を渡します。

    if ($this->confirm('続行しますか？', true)) {
        // ...
    }

<a name="auto-completion"></a>
#### 自動補完

`anticipate`メソッドは、可能な選択肢の自動補完を提供するために使用できます。ユーザーは自動補完のヒントに関係なく、任意の回答を提供できます。

```php
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
```

Alternatively, you may pass a closure as the second argument to the `anticipate` method. The closure will be called each time the user types an input character. The closure should accept a string parameter containing the user's input so far, and return an array of options for auto-completion:

```
$name = $this->anticipate('What is your address?', function (string $input) {
    // Return auto-completion options...
});
```

<a name="multiple-choice-questions"></a>
#### 複数選択の質問

質問をする際に、ユーザーに事前に定義された選択肢を提示する必要がある場合、`choice`メソッドを使用できます。オプションが選択されなかった場合に返されるデフォルト値の配列インデックスを、メソッドの第3引数として渡すことができます:

$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);

さらに、`choice`メソッドは、有効な応答を選択するための最大試行回数と、複数の選択が許可されるかどうかを決定するためのオプションの第4引数と第5引数を受け入れます:

$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);

<a name="writing-output"></a>
### 出力の書き込み

コンソールに出力を送信するには、`line`、`info`、`comment`、`question`、`warn`、および`error`メソッドを使用できます。これらの各メソッドは、その目的に適したANSIカラーを使用します。例えば、ユーザーに一般的な情報を表示する場合、通常、`info`メソッドはコンソールに緑色のテキストとして表示されます:

/**
 * Execute the console command.
 */
public function handle(): void
{
    // ...

    $this->info('The command was successful!');
}

エラーメッセージを表示するには、`error`メソッドを使用します。エラーメッセージのテキストは通常、赤色で表示されます:
```

```php
$this->error('Something went wrong!');
```

プレーンで色の付いていないテキストを表示するには、`line`メソッドを使用できます：

```php
$this->line('Display this on the screen');
```

空白行を表示するには、`newLine`メソッドを使用できます：

```php
// 1行の空白行を表示...
$this->newLine();

// 3行の空白行を表示...
$this->newLine(3);
```

<a name="tables"></a>
#### テーブル

`table`メソッドを使用すると、複数の行/列のデータを簡単に正しくフォーマットできます。必要なのは、テーブルの列名とデータを提供することだけで、Laravelが自動的にテーブルの適切な幅と高さを計算します：

```php
use App\Models\User;

$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

<a name="progress-bars"></a>
#### プログレスバー

長時間かかるタスクの場合、タスクがどの程度完了しているかをユーザーに知らせるプログレスバーを表示すると便利です。`withProgressBar`メソッドを使用すると、Laravelはプログレスバーを表示し、指定された反復可能な値を反復するたびにその進行状況を進めます：

```php
use App\Models\User;

$users = $this->withProgressBar(User::all(), function (User $user) {
    $this->performTask($user);
});
```

場合によっては、プログレスバーの進行をより手動で制御する必要があります。まず、プロセスが反復するステップの総数を定義します。次に、各アイテムを処理した後にプログレスバーを進めます：

```php
$users = App\Models\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

> NOTE:  
> より高度なオプションについては、[Symfony Progress Barコンポーネントのドキュメント](https://symfony.com/doc/7.0/components/console/helpers/progressbar.html)を確認してください。

<a name="registering-commands"></a>
## コマンドの登録

デフォルトでは、Laravelは`app/Console/Commands`ディレクトリ内のすべてのコマンドを自動的に登録します。ただし、アプリケーションの`bootstrap/app.php`ファイル内で`withCommands`メソッドを使用して、LaravelにArtisanコマンドを他のディレクトリからスキャンするよう指示することもできます。

    ->withCommands([
        __DIR__.'/../app/Domain/Orders/Commands',
    ])

必要に応じて、コマンドのクラス名を`withCommands`メソッドに渡すことで、手動でコマンドを登録することもできます。

    use App\Domain\Orders\Commands\SendEmails;

    ->withCommands([
        SendEmails::class,
    ])

Artisanが起動すると、アプリケーション内のすべてのコマンドが[サービスコンテナ](container.md)によって解決され、Artisanに登録されます。

<a name="programmatically-executing-commands"></a>
## プログラムによるコマンドの実行

CLI以外でArtisanコマンドを実行したい場合があります。例えば、ルートやコントローラからArtisanコマンドを実行したい場合です。`Artisan`ファサードの`call`メソッドを使用してこれを実現できます。`call`メソッドは、コマンドのシグネチャ名またはクラス名を最初の引数として受け取り、コマンドパラメータの配列を2番目の引数として受け取ります。終了コードが返されます。

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function (string $user) {
        $exitCode = Artisan::call('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        // ...
    });

または、Artisanコマンド全体を文字列として`call`メソッドに渡すこともできます。

    Artisan::call('mail:send 1 --queue=default');

<a name="passing-array-values"></a>
#### 配列値の渡し方

コマンドが配列を受け入れるオプションを定義している場合、そのオプションに値の配列を渡すことができます。

    use Illuminate\Support\Facades\Artisan;

    Route::post('/mail', function () {
        $exitCode = Artisan::call('mail:send', [
            '--id' => [5, 13]
        ]);
    });

<a name="passing-boolean-values"></a>
#### ブール値の引き渡し

文字列値を受け入れないオプション（`migrate:refresh`コマンドの`--force`フラグなど）の値を指定する必要がある場合は、オプションの値として`true`または`false`を渡すべきです：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="queueing-artisan-commands"></a>
#### Artisanコマンドのキューイング

`Artisan`ファサードの`queue`メソッドを使用すると、Artisanコマンドをキューに入れて、[キューワーカー](queues.md)によってバックグラウンドで処理されるようにすることもできます。このメソッドを使用する前に、キューの設定とキューリスナーの実行が完了していることを確認してください：

    use Illuminate\Support\Facades\Artisan;

    Route::post('/user/{user}/mail', function (string $user) {
        Artisan::queue('mail:send', [
            'user' => $user, '--queue' => 'default'
        ]);

        // ...
    });

`onConnection`メソッドと`onQueue`メソッドを使用して、Artisanコマンドをディスパッチする接続やキューを指定できます：

    Artisan::queue('mail:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

<a name="calling-commands-from-other-commands"></a>
### 他のコマンドからのコマンド呼び出し

既存のArtisanコマンドから他のコマンドを呼び出したい場合があります。`call`メソッドを使用してこれを行うことができます。この`call`メソッドは、コマンド名とコマンドの引数/オプションの配列を受け取ります：

    /**
     * コンソールコマンドの実行
     */
    public function handle(): void
    {
        $this->call('mail:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        // ...
    }

別のコンソールコマンドを呼び出し、そのすべての出力を抑制したい場合は、`callSilently`メソッドを使用できます。`callSilently`メソッドは、`call`メソッドと同じシグネチャを持っています：

    $this->callSilently('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

<a name="signal-handling"></a>
## シグナルハンドリング

ご存知のように、オペレーティングシステムは実行中のプロセスにシグナルを送信することができます。例えば、`SIGTERM`シグナルは、オペレーティングシステムがプログラムに終了を要求する方法です。Artisanコンソールコマンドでシグナルをリッスンし、発生したときにコードを実行したい場合は、`trap`メソッドを使用できます。

    /**
     * コンソールコマンドの実行
     */
    public function handle(): void
    {
        $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

        while ($this->shouldKeepRunning) {
            // ...
        }
    }

一度に複数のシグナルをリッスンするには、`trap`メソッドにシグナルの配列を渡すことができます。

    $this->trap([SIGTERM, SIGQUIT], function (int $signal) {
        $this->shouldKeepRunning = false;

        dump($signal); // SIGTERM / SIGQUIT
    });

<a name="stub-customization"></a>
## スタブのカスタマイズ

Artisanコンソールの`make`コマンドは、コントローラ、ジョブ、マイグレーション、テストなど、さまざまなクラスを作成するために使用されます。これらのクラスは、"スタブ"ファイルを使用して生成され、入力に基づいて値が設定されます。しかし、Artisanによって生成されるファイルに小さな変更を加えたい場合があります。これを実現するには、`stub:publish`コマンドを使用して、最も一般的なスタブをアプリケーションに公開し、カスタマイズできるようにします。

```shell
php artisan stub:publish
```

公開されたスタブは、アプリケーションのルートにある`stubs`ディレクトリ内に配置されます。これらのスタブに加えた変更は、Artisanの`make`コマンドを使用して対応するクラスを生成する際に反映されます。

<a name="events"></a>
## イベント

Artisanは、コマンド実行時に3つのイベントを発行します: `Illuminate\Console\Events\ArtisanStarting`、`Illuminate\Console\Events\CommandStarting`、そして `Illuminate\Console\Events\CommandFinished`です。`ArtisanStarting`イベントは、Artisanが実行を開始した直後に発行されます。次に、`CommandStarting`イベントは、コマンドが実行される直前に発行されます。最後に、`CommandFinished`イベントは、コマンドの実行が完了した後に発行されます。

