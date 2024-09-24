# プロセス

- [イントロダクション](#introduction)
- [プロセスの呼び出し](#invoking-processes)
    - [プロセスオプション](#process-options)
    - [プロセス出力](#process-output)
    - [パイプライン](#process-pipelines)
- [非同期プロセス](#asynchronous-processes)
    - [プロセスIDとシグナル](#process-ids-and-signals)
    - [非同期プロセス出力](#asynchronous-process-output)
- [並行プロセス](#concurrent-processes)
    - [プールプロセスの命名](#naming-pool-processes)
    - [プールプロセスIDとシグナル](#pool-process-ids-and-signals)
- [テスト](#testing)
    - [プロセスのフェイク](#faking-processes)
    - [特定のプロセスのフェイク](#faking-specific-processes)
    - [プロセスシーケンスのフェイク](#faking-process-sequences)
    - [非同期プロセスライフサイクルのフェイク](#faking-asynchronous-process-lifecycles)
    - [利用可能なアサーション](#available-assertions)
    - [ストレイプロセスの防止](#preventing-stray-processes)

<a name="introduction"></a>
## イントロダクション

Laravelは、[Symfony Process component](https://symfony.com/doc/7.0/components/process.html) を中心に、表現力豊かで最小限のAPIを提供し、Laravelアプリケーションから外部プロセスを便利に呼び出せるようにしています。Laravelのプロセス機能は、最も一般的なユースケースと素晴らしい開発者体験に焦点を当てています。

<a name="invoking-processes"></a>
## プロセスの呼び出し

プロセスを呼び出すには、`Process`ファサードが提供する`run`メソッドと`start`メソッドを使用できます。`run`メソッドはプロセスを呼び出し、プロセスの実行が終了するまで待機しますが、`start`メソッドは非同期プロセス実行に使用されます。このドキュメント内で両方のアプローチを検討します。まず、基本的な同期プロセスを呼び出し、その結果を検査する方法を見てみましょう。

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

もちろん、`run`メソッドによって返される`Illuminate\Contracts\Process\ProcessResult`インスタンスは、プロセス結果を検査するために使用できるさまざまな便利なメソッドを提供します。

```php
$result = Process::run('ls -la');

$result->successful();
$result->failed();
$result->exitCode();
$result->output();
$result->errorOutput();
```

<a name="throwing-exceptions"></a>
#### 例外のスロー

プロセス結果があり、終了コードがゼロより大きい場合（失敗を示す）に`Illuminate\Process\Exceptions\ProcessFailedException`のインスタンスをスローしたい場合は、`throw`および`throwIf`メソッドを使用できます。プロセスが失敗しなかった場合、プロセス結果インスタンスが返されます。

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### プロセスオプション

もちろん、プロセスを呼び出す前にその動作をカスタマイズする必要があるかもしれません。幸いなことに、Laravelでは作業ディレクトリ、タイムアウト、環境変数など、さまざまなプロセス機能を調整できます。

<a name="working-directory-path"></a>
#### 作業ディレクトリパス

`path`メソッドを使用して、プロセスの作業ディレクトリを指定できます。このメソッドが呼び出されない場合、プロセスは現在実行中のPHPスクリプトの作業ディレクトリを継承します。

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### 入力

`input`メソッドを使用して、プロセスの「標準入力」を介して入力を提供できます。

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### タイムアウト

デフォルトでは、プロセスは60秒以上実行された後に`Illuminate\Process\Exceptions\ProcessTimedOutException`のインスタンスをスローします。ただし、`timeout`メソッドを介してこの動作をカスタマイズできます。

```php
$result = Process::timeout(120)->run('bash import.sh');
```

または、プロセスのタイムアウトを完全に無効にしたい場合は、`forever`メソッドを呼び出すことができます。

```php
$result = Process::forever()->run('bash import.sh');
```

`idleTimeout`メソッドを使用して、プロセスが出力を返さずに実行できる最大秒数を指定できます。

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### 環境変数

`env`メソッドを介してプロセスに環境変数を提供できます。呼び出されたプロセスは、システムで定義されたすべての環境変数も継承します。

```php
$result = Process::forever()
            ->env(['IMPORT_PATH' => __DIR__])
            ->run('bash import.sh');
```

呼び出されたプロセスから継承された環境変数を削除したい場合は、その環境変数に`false`の値を指定できます。

```php
$result = Process::forever()
            ->env(['LOAD_PATH' => false])
            ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### TTYモード

`tty`メソッドを使用して、プロセスのTTYモードを有効にできます。TTYモードは、プロセスの入力と出力をプログラムの入力と出力に接続し、プロセスとしてVimやNanoなどのエディタを開くことができます。

```php
Process::forever()->tty()->run('vim');
```

<a name="process-output"></a>
### プロセス出力

前述のように、プロセス出力は、プロセス結果の`output`（stdout）および`errorOutput`（stderr）メソッドを使用してアクセスできます。

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

ただし、出力は`run`メソッドに2番目の引数としてクロージャを渡すことでリアルタイムで収集することもできます。クロージャは2つの引数を受け取ります：出力の「タイプ」（`stdout`または`stderr`）と出力文字列自体です。

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravelは、プロセスの出力に特定の文字列が含まれているかどうかを判断するための便利な方法として、`seeInOutput`および`seeInErrorOutput`メソッドも提供しています。

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### プロセス出力の無効化

プロセスが大量の出力を書き込み、それに興味がない場合、メモリを節約するために出力の取得を完全に無効にできます。これを行うには、プロセスを構築する際に`quietly`メソッドを呼び出します。

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### パイプライン

時には、あるプロセスの出力を別のプロセスの入力にする必要があるかもしれません。これは、プロセスの出力を別のプロセスに「パイプ」することとしてよく知られています。`Process`ファサードが提供する`pipe`メソッドを使用すると、これを簡単に実現できます。`pipe`メソッドは、パイプされたプロセスを同期的に実行し、パイプラインの最後のプロセスのプロセス結果を返します。

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

パイプラインを構成する個々のプロセスをカスタマイズする必要がない場合は、コマンド文字列の配列を`pipe`メソッドに単純に渡すことができます。

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

出力は、`pipe`メソッドに2番目の引数としてクロージャを渡すことでリアルタイムで収集することもできます。クロージャは2つの引数を受け取ります：出力の「タイプ」（`stdout`または`stderr`）と出力文字列自体です。

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravelでは、`as`メソッドを介してパイプライン内の各プロセスに文字列キーを割り当てることもできます。このキーも`pipe`メソッドに提供される出力クロージャに渡され、どのプロセスの出力かを特定できます。

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
})->start(function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## 非同期プロセス

`run`メソッドは同期プロセスを呼び出しますが、`start`メソッドは非同期プロセスを呼び出すために使用できます。これにより、プロセスがバックグラウンドで実行されている間にアプリケーションが他のタスクを続行できます。プロセスが呼び出されると、`running`メソッドを使用してプロセスがまだ実行中かどうかを判断できます。

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

ご覧のとおり、プロセスの実行が終了するまで待機し、プロセス結果インスタンスを取得するために`wait`メソッドを呼び出すことができます。

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### プロセスIDとシグナル

`id`メソッドを使用して、実行中のプロセスのオペレーティングシステムが割り当てたプロセスIDを取得できます。

```php
$process = Process::start('bash import.sh');

return $process->id();
```

`signal`メソッドを使用して、実行中のプロセスに「シグナル」を送信できます。定義済みのシグナル定数のリストは、[PHPドキュメント](https://www.php.net/manual/en/pcntl.constants.php)に記載されています。

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### 非同期プロセス出力

非同期プロセスの実行中に、`output` および `errorOutput` メソッドを使用してその現在の出力全体にアクセスできます。ただし、最後に出力を取得してから発生したプロセスの出力にアクセスするために、`latestOutput` および `latestErrorOutput` を利用することもできます：

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

`run` メソッドと同様に、`start` メソッドにクロージャを第二引数として渡すことで、非同期プロセスからリアルタイムに出力を収集することもできます。クロージャは、出力の「タイプ」（`stdout` または `stderr`）と出力文字列の2つの引数を受け取ります：

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

<a name="concurrent-processes"></a>
## 並行プロセス

Laravel は、同時並行で非同期プロセスのプールを管理することも容易にしており、多くのタスクを同時に簡単に実行できます。始めるには、`pool` メソッドを呼び出します。これは、`Illuminate\Process\Pool` のインスタンスを受け取るクロージャを引数に取ります。

このクロージャ内で、プールに属するプロセスを定義できます。プロセスプールが `start` メソッドを介して開始されると、`running` メソッドを介して実行中のプロセスの [コレクション](collections.md) にアクセスできます：

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

ご覧のように、プールのすべてのプロセスが実行を終了し、`wait` メソッドを介してそれらの結果を解決することができます。`wait` メソッドは、配列アクセス可能なオブジェクトを返し、プール内の各プロセスのプロセス結果インスタンスにキーでアクセスできます：

```php
$results = $pool->wait();

echo $results[0]->output();
```

また、便宜上、`concurrently` メソッドを使用して非同期プロセスプールを開始し、すぐにその結果を待つことができます。これは、PHPの配列の分解機能と組み合わせると、特に表現力豊かな構文を提供します：

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### プールプロセスの命名

数値キーでプール結果にアクセスするのはあまり表現力がありません。そのため、Laravel は `as` メソッドを介してプール内の各プロセスに文字列キーを割り当てることができます。このキーは、`start` メソッドに提供されるクロージャにも渡され、どのプロセスの出力であるかを判断するのに役立ちます：

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>
### プールプロセスIDとシグナル

プールの `running` メソッドは、プール内で呼び出されたすべてのプロセスのコレクションを提供するため、基礎となるプールプロセスIDに簡単にアクセスできます：

```php
$processIds = $pool->running()->each->id();
```

また、便宜上、プールに対して `signal` メソッドを呼び出して、プール内のすべてのプロセスにシグナルを送信することができます：

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## テスト

多くのLaravelサービスは、テストを簡単かつ表現力豊かに書くための機能を提供しており、Laravelのプロセスサービスも例外ではありません。`Process` ファサードの `fake` メソッドを使用すると、Laravelにプロセスが呼び出されたときにスタブ/ダミーの結果を返すよう指示できます。

<a name="faking-processes"></a>
### プロセスのフェイク

Laravelのプロセスをフェイクする能力を探るために、プロセスを呼び出すルートを想像してみましょう：

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

このルートをテストする際、`Process` ファサードの `fake` メソッドを引数なしで呼び出すことで、Laravelにすべての呼び出されたプロセスに対してダミーの成功プロセス結果を返すよう指示できます。さらに、[アサーション](#available-assertions) を使用して、特定のプロセスが「実行された」ことを確認できます：

===  "Pest"
```php
<?php

use Illuminate\Process\PendingProcess;
use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Support\Facades\Process;

test('process is invoked', function () {
    Process::fake();

    $response = $this->get('/import');

    // シンプルなプロセスアサーション...
    Process::assertRan('bash import.sh');

    // または、プロセス設定の検査...
    Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
        return $process->command === 'bash import.sh' &&
               $process->timeout === 60;
    });
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Illuminate\Process\PendingProcess;
use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // シンプルなプロセスアサーション...
        Process::assertRan('bash import.sh');

        // または、プロセス設定の検査...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

前述のように、`Process` ファサードの `fake` メソッドを呼び出すと、Laravelに常に出力のない成功プロセス結果を返すよう指示されます。ただし、`Process` ファサードの `result` メソッドを使用して、フェイクプロセスの出力と終了コードを簡単に指定できます：

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>
### 特定のプロセスのフェイク

前の例で気づいたかもしれませんが、`Process` ファサードでは、`fake` メソッドに配列を渡すことで、プロセスごとに異なるフェイク結果を指定できます。

配列のキーは、フェイクしたいコマンドパターンを表し、それに関連する結果を指定します。`*` 文字をワイルドカード文字として使用できます。フェイクされていないプロセスコマンドは、実際に呼び出されます。`Process` ファサードの `result` メソッドを使用して、これらのコマンドのスタブ/フェイク結果を構築できます：

```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

フェイクプロセスの終了コードやエラー出力をカスタマイズする必要がない場合、フェイクプロセス結果を単純な文字列として指定する方が便利かもしれません：

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### プロセスシーケンスのフェイク

テストしているコードが同じコマンドで複数のプロセスを呼び出す場合、各プロセス呼び出しに異なるフェイクプロセス結果を割り当てたい場合があります。`Process` ファサードの `sequence` メソッドを使用して、これを実現できます：

```php
Process::fake([
    'ls *' => Process::sequence()
                ->push(Process::result('First invocation'))
                ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### 非同期プロセスライフサイクルのフェイク

これまで、主に `run` メソッドを使用して同期的に呼び出されるプロセスをフェイクすることについて説明してきました。しかし、`start` を介して呼び出される非同期プロセスと対話するコードをテストしようとしている場合、より高度なアプローチが必要になることがあります。

例えば、非同期プロセスと対話する次のルートを想像してみましょう：

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

このプロセスを適切にフェイクするには、`running` メソッドが `true` を返す回数を指定する必要があります。さらに、シーケンスで返される複数行の出力を指定したい場合があります。これを実現するには、`Process` ファサードの `describe` メソッドを使用できます：

```php
Process::fake([
    'bash import.sh' => Process::describe()
            ->output('First line of standard output')
            ->errorOutput('First line of error output')
            ->output('Second line of standard output')
            ->exitCode(0)
            ->iterations(3),
]);
```

上記の例を詳しく見てみましょう。`output` および `errorOutput` メソッドを使用して、シーケンスで返される複数行の出力を指定できます。`exitCode` メソッドを使用して、フェイクプロセスの最終終了コードを指定できます。最後に、`iterations` メソッドを使用して、`running` メソッドが `true` を返す回数を指定できます。

<a name="available-assertions"></a>
### 利用可能なアサーション

Laravel のプロセスサービスは、テストを簡単かつ表現力豊かに書くための機能を提供しています。以下は、利用可能なアサーションの一覧です：

- `Process::assertRan(string|Closure $command)`: 指定されたコマンドが実行されたことをアサートします。
- `Process::assertNotRan(string|Closure $command)`: 指定されたコマンドが実行されなかったことをアサートします。
- `Process::assertRanTimes(string|Closure $command, int $times)`: 指定されたコマンドが指定された回数実行されたことをアサートします。
- `Process::assertDidntRun(string|Closure $command)`: 指定されたコマンドが実行されなかったことをアサートします。
- `Process::assertRanWith(string|Closure $command, array $arguments)`: 指定されたコマンドが指定された引数で実行されたことをアサートします。
- `Process::assertRanWithoutArguments(string|Closure $command)`: 指定されたコマンドが引数なしで実行されたことをアサートします。
- `Process::assertRanIn(string|Closure $command, string $directory)`: 指定されたコマンドが指定されたディレクトリで実行されたことをアサートします。
- `Process::assertRanAsUser(string|Closure $command, string $user)`: 指定されたコマンドが指定されたユーザーとして実行されたことをアサートします。
- `Process::assertRanWithTimeout(string|Closure $command, int $timeout)`: 指定されたコマンドが指定されたタイムアウトで実行されたことをアサートします。
- `Process::assertRanWithIdleTimeout(string|Closure $command, int $idleTimeout)`: 指定されたコマンドが指定されたアイドルタイムアウトで実行されたことをアサートします。
- `Process::assertRanWithEnv(string|Closure $command, array $env)`: 指定されたコマンドが指定された環境変数で実行されたことをアサートします。
- `Process::assertRanWithoutEnv(string|Closure $command)`: 指定されたコマンドが環境変数なしで実行されたことをアサートします。
- `Process::assertRanWithInput(string|Closure $command, string $input)`: 指定されたコマンドが指定された入力で実行されたことをアサートします。
- `Process::assertRanWithoutInput(string|Closure $command)`: 指定されたコマンドが入力なしで実行されたことをアサートします。
- `Process::assertRanWithOutput(string|Closure $command, string $output)`: 指定されたコマンドが指定された出力で実行されたことをアサートします。
- `Process::assertRanWithoutOutput(string|Closure $command)`: 指定されたコマンドが出力なしで実行されたことをアサートします。
- `Process::assertRanWithErrorOutput(string|Closure $command, string $errorOutput)`: 指定されたコマンドが指定されたエラー出力で実行されたことをアサートします。
- `Process::assertRanWithoutErrorOutput(string|Closure $command)`: 指定されたコマンドがエラー出力なしで実行されたことをアサートします。
- `Process::assertRanWithExitCode(string|Closure $command, int $exitCode)`: 指定されたコマンドが指定された終了コードで実行されたことをアサートします。
- `Process::assertRanWithoutExitCode(string|Closure $command)`: 指定されたコマンドが終了コードなしで実行されたことをアサートします。
- `Process::assertRanWithException(string|Closure $command, Throwable $exception)`: 指定されたコマンドが指定された例外で実行されたことをアサートします。
- `Process::assertRanWithoutException(string|Closure $command)`: 指定されたコマンドが例外なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionMessage(string|Closure $command, string $message)`: 指定されたコマンドが指定された例外メッセージで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionMessage(string|Closure $command)`: 指定されたコマンドが例外メッセージなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionCode(string|Closure $command, int $code)`: 指定されたコマンドが指定された例外コードで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionCode(string|Closure $command)`: 指定されたコマンドが例外コードなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionTrace(string|Closure $command, string $trace)`: 指定されたコマンドが指定された例外トレースで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionTrace(string|Closure $command)`: 指定されたコマンドが例外トレースなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionFile(string|Closure $command, string $file)`: 指定されたコマンドが指定された例外ファイルで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionFile(string|Closure $command)`: 指定されたコマンドが例外ファイルなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionLine(string|Closure $command, int $line)`: 指定されたコマンドが指定された例外行で実行されたことをアサートします。
- `Process::assertRanWithoutExceptionLine(string|Closure $command)`: 指定されたコマンドが例外行なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionClass(string|Closure $command, string $class)`: 指定されたコマンドが指定された例外クラスで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionClass(string|Closure $command)`: 指定されたコマンドが例外クラスなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPrevious(string|Closure $command, Throwable $previous)`: 指定されたコマンドが指定された前の例外で実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPrevious(string|Closure $command)`: 指定されたコマンドが前の例外なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousMessage(string|Closure $command, string $message)`: 指定されたコマンドが指定された前の例外メッセージで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousMessage(string|Closure $command)`: 指定されたコマンドが前の例外メッセージなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousCode(string|Closure $command, int $code)`: 指定されたコマンドが指定された前の例外コードで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousCode(string|Closure $command)`: 指定されたコマンドが前の例外コードなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousTrace(string|Closure $command, string $trace)`: 指定されたコマンドが指定された前の例外トレースで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousTrace(string|Closure $command)`: 指定されたコマンドが前の例外トレースなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousFile(string|Closure $command, string $file)`: 指定されたコマンドが指定された前の例外ファイルで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousFile(string|Closure $command)`: 指定されたコマンドが前の例外ファイルなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousLine(string|Closure $command, int $line)`: 指定されたコマンドが指定された前の例外行で実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousLine(string|Closure $command)`: 指定されたコマンドが前の例外行なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousClass(string|Closure $command, string $class)`: 指定されたコマンドが指定された前の例外クラスで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousClass(string|Closure $command)`: 指定されたコマンドが前の例外クラスなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPrevious(string|Closure $command, Throwable $previous)`: 指定されたコマンドが指定された前の前の例外で実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPrevious(string|Closure $command)`: 指定されたコマンドが前の前の例外なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousMessage(string|Closure $command, string $message)`: 指定されたコマンドが指定された前の前の例外メッセージで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousMessage(string|Closure $command)`: 指定されたコマンドが前の前の例外メッセージなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousCode(string|Closure $command, int $code)`: 指定されたコマンドが指定された前の前の例外コードで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousCode(string|Closure $command)`: 指定されたコマンドが前の前の例外コードなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousTrace(string|Closure $command, string $trace)`: 指定されたコマンドが指定された前の前の例外トレースで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousTrace(string|Closure $command)`: 指定されたコマンドが前の前の例外トレースなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousFile(string|Closure $command, string $file)`: 指定されたコマンドが指定された前の前の例外ファイルで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousFile(string|Closure $command)`: 指定されたコマンドが前の前の例外ファイルなしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousLine(string|Closure $command, int $line)`: 指定されたコマンドが指定された前の前の例外行で実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousLine(string|Closure $command)`: 指定されたコマンドが前の前の例外行なしで実行されたことをアサートします。
- `Process::assertRanWithExceptionPreviousPreviousClass(string|Closure $command, string $class)`: 指定されたコマンドが指定された前の前の例外クラスで実行されたことをアサートします。
- `Process::assertRanWithoutExceptionPreviousPreviousClass(string|Closure $command)`: 指定されたコマンドが前の前の例外クラスなしで実行されたことをアサートします。
- `Process::

[以前に議論したように](#faking-processes)、Laravelは機能テストのためのいくつかのプロセスアサーションを提供しています。以下では、これらの各アサーションについて説明します。

<a name="assert-process-ran"></a>
#### assertRan

指定されたプロセスが呼び出されたことをアサートします：

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

`assertRan`メソッドはクロージャも受け入れます。このクロージャはプロセスのインスタンスとプロセス結果のインスタンスを受け取り、プロセスの設定オプションを検査できます。このクロージャが`true`を返す場合、アサーションは「パス」します：

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

`assertRan`クロージャに渡される`$process`は`Illuminate\Process\PendingProcess`のインスタンスであり、`$result`は`Illuminate\Contracts\Process\ProcessResult`のインスタンスです。

<a name="assert-process-didnt-run"></a>
#### assertDidntRun

指定されたプロセスが呼び出されなかったことをアサートします：

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

`assertRan`メソッドと同様に、`assertDidntRun`メソッドもクロージャを受け入れます。このクロージャはプロセスのインスタンスとプロセス結果のインスタンスを受け取り、プロセスの設定オプションを検査できます。このクロージャが`true`を返す場合、アサーションは「失敗」します：

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

指定されたプロセスが指定された回数呼び出されたことをアサートします：

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

`assertRanTimes`メソッドもクロージャを受け入れます。このクロージャはプロセスのインスタンスとプロセス結果のインスタンスを受け取り、プロセスの設定オプションを検査できます。このクロージャが`true`を返し、プロセスが指定された回数呼び出された場合、アサーションは「パス」します：

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### 迷子プロセスの防止

個々のテストまたは完全なテストスイート全体で、呼び出されたすべてのプロセスが偽装されていることを確認したい場合は、`preventStrayProcesses`メソッドを呼び出すことができます。このメソッドを呼び出した後、対応する偽装結果を持たないプロセスは、実際のプロセスを開始する代わりに例外をスローします：

```php
use Illuminate\Support\Facades\Process;

Process::preventStrayProcesses();

Process::fake([
    'ls *' => 'Test output...',
]);

// 偽装されたレスポンスが返されます...
Process::run('ls -la');

// 例外がスローされます...
Process::run('bash import.sh');
```
