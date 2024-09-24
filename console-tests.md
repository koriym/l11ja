# コンソールテスト

- [はじめに](#introduction)
- [成功 / 失敗の期待値](#success-failure-expectations)
- [入力 / 出力の期待値](#input-output-expectations)
- [コンソールイベント](#console-events)

<a name="introduction"></a>
## はじめに

HTTPテストの簡素化に加えて、Laravelはアプリケーションの[カスタムコンソールコマンド](artisan.md)をテストするためのシンプルなAPIを提供しています。

<a name="success-failure-expectations"></a>
## 成功 / 失敗の検証

まず、Artisanコマンドの終了コードに関するアサーションを行う方法を見てみましょう。これを実現するために、`artisan`メソッドを使用してテストからArtisanコマンドを呼び出します。次に、`assertExitCode`メソッドを使用して、コマンドが指定された終了コードで完了したことを確認します。

===  "Pest"

    ```php
    test('console command', function () {
        $this->artisan('inspire')->assertExitCode(0);
    });
    ```

===  "PHPUnit"

    ```php
    /**
     * コンソールコマンドのテスト
     */
    public function test_console_command(): void
    {
        $this->artisan('inspire')->assertExitCode(0);
    }
    ```

指定された終了コードでコマンドが終了しなかったことをアサートするには、`assertNotExitCode`メソッドを使用できます。

    $this->artisan('inspire')->assertNotExitCode(1);

もちろん、すべてのターミナルコマンドは通常、成功した場合にステータスコード`0`で終了し、失敗した場合にはゼロ以外の終了コードで終了します。したがって、簡単に、`assertSuccessful`と`assertFailed`アサーションを使用して、指定されたコマンドが成功した終了コードで終了したかどうかをアサートできます。

    $this->artisan('inspire')->assertSuccessful();

    $this->artisan('inspire')->assertFailed();

<a name="input-output-expectations"></a>
## 入力 / 出力のテスト

Laravelでは、`expectsQuestion`メソッドを使用してコンソールコマンドのユーザー入力を簡単に「モック」できます。さらに、`assertExitCode`と`expectsOutput`メソッドを使用して、コンソールコマンドによって出力されると予想される終了コードとテキストを指定できます。例えば、次のコンソールコマンドを考えてみましょう。

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you prefer?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you prefer '.$language.'.');
    });

このコマンドは、次のテストでテストできます。

===  "Pest"

    ```php
    test('console command', function () {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you prefer?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
             ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
             ->assertExitCode(0);
    });
    ```

===  "PHPUnit"

    ```php
    /**
     * コンソールコマンドのテスト
     */
    public function test_console_command(): void
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you prefer?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
             ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
             ->assertExitCode(0);
    }
    ```

[Laravel Prompts](prompts.md)が提供する`search`または`multisearch`関数を使用している場合、`expectsSearch`アサーションを使用してユーザーの入力、検索結果、および選択をモックできます。

===  "Pest"

    ```php
    test('console command', function () {
        $this->artisan('example')
             ->expectsSearch('What is your name?', search: 'Tay', answers: [
                'Taylor Otwell',
                'Taylor Swift',
                'Darian Taylor'
             ], answer: 'Taylor Otwell')
             ->assertExitCode(0);
    });
    ```

===  "PHPUnit"

    ```php
    /**
     * コンソールコマンドのテスト
     */
    public function test_console_command(): void
    {
        $this->artisan('example')
             ->expectsSearch('What is your name?', search: 'Tay', answers: [
                'Taylor Otwell',
                'Taylor Swift',
                'Darian Taylor'
             ], answer: 'Taylor Otwell')
             ->assertExitCode(0);
    }
    ```

`doesntExpectOutput`メソッドを使用して、コンソールコマンドが出力を生成しないことをアサートすることもできます。

===  "Pest"

    ```php
    test('console command', function () {
        $this->artisan('example')
             ->doesntExpectOutput()
             ->assertExitCode(0);
    });
    ```

===  "PHPUnit"

    ```php
    /**
     * コンソールコマンドのテスト
     */
    public function test_console_command(): void
    {
        $this->artisan('example')
                ->doesntExpectOutput()
                ->assertExitCode(0);
    }
    ```

`expectsOutputToContain`と`doesntExpectOutputToContain`メソッドを使用して、出力の一部に対してアサーションを行うこともできます。

===  "Pest"

    ```php
    test('console command', function () {
        $this->artisan('example')
             ->expectsOutputToContain('Taylor')
             ->assertExitCode(0);
    });
    ```

===  "PHPUnit"

    ```php
    /**
     * コンソールコマンドのテスト
     */
    public function test_console_command(): void
    {
        $this->artisan('example')
                ->expectsOutputToContain('Taylor')
                ->assertExitCode(0);
    }
    ```

<a name="confirmation-expectations"></a>
#### 確認の期待値

"yes"または"no"の回答形式で確認を期待するコマンドを書く場合、`expectsConfirmation`メソッドを使用できます。

    $this->artisan('module:import')
        ->expectsConfirmation('Do you really wish to run this command?', 'no')
        ->assertExitCode(1);

<a name="table-expectations"></a>
#### テーブルの期待値

コマンドがArtisanの`table`メソッドを使用して情報のテーブルを表示する場合、テーブル全体の出力期待値を書くのは面倒です。代わりに、`expectsTable`メソッドを使用できます。このメソッドは、テーブルのヘッダーを最初の引数として、テーブルのデータを2番目の引数として受け取ります。

    $this->artisan('users:all')
        ->expectsTable([
            'ID',
            'Email',
        ], [
            [1, 'taylor@example.com'],
            [2, 'abigail@example.com'],
        ]);

<a name="console-events"></a>
## コンソールイベント

デフォルトでは、`Illuminate\Console\Events\CommandStarting`と`Illuminate\Console\Events\CommandFinished`イベントは、アプリケーションのテストを実行している間にディスパッチされません。ただし、`Illuminate\Foundation\Testing\WithConsoleEvents`トレイトをクラスに追加することで、特定のテストクラスに対してこれらのイベントを有効にできます。

=== "Pest"
 
    ```php
    <?php
    
    use Illuminate\Foundation\Testing\WithConsoleEvents;
    
    uses(WithConsoleEvents::class);
    
    // ...
    ```

=== "PHPUnit"

    ```php
    <?php
    
    namespace Tests\Feature;
    
    use Illuminate\Foundation\Testing\WithConsoleEvents;
    use Tests\TestCase;
    
    class ConsoleEventTest extends TestCase
    {
        use WithConsoleEvents;
    
        // ...
    }
    ```

