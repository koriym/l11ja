# レートリミット

- [はじめに](#introduction)
    - [キャッシュ設定](#cache-configuration)
- [基本的な使用法](#basic-usage)
    - [手動での試行回数増加](#manually-incrementing-attempts)
    - [試行回数のクリア](#clearing-attempts)

<a name="introduction"></a>
## はじめに

Laravelには、アプリケーションの[キャッシュ](cache.md)と組み合わせて、指定された時間枠内で任意のアクションを制限する簡単な方法を提供する、使いやすいレートリミットの抽象化が含まれています。

> NOTE:  
> 受信HTTPリクエストのレートリミットに興味がある場合は、[レートリミッターミドルウェアのドキュメント](routing.md#rate-limiting)を参照してください。

<a name="cache-configuration"></a>
### キャッシュ設定

通常、レートリミッターは、アプリケーションの`cache`設定ファイル内の`default`キーで定義されたデフォルトのアプリケーションキャッシュを利用します。ただし、アプリケーションの`cache`設定ファイル内に`limiter`キーを定義することで、レートリミッターが使用するキャッシュドライバーを指定できます。

    'default' => env('CACHE_STORE', 'database'),

    'limiter' => 'redis',

<a name="basic-usage"></a>
## 基本的な使用法

`Illuminate\Support\Facades\RateLimiter`ファサードを使用して、レートリミッターとやり取りできます。レートリミッターが提供する最も簡単なメソッドは`attempt`メソッドで、指定された秒数間、指定されたコールバックをレートリミットします。

`attempt`メソッドは、コールバックに利用可能な試行回数が残っていない場合に`false`を返します。それ以外の場合、`attempt`メソッドはコールバックの結果または`true`を返します。`attempt`メソッドが受け取る最初の引数は、レートリミットされるアクションを表す任意の文字列であるレートリミッターの「キー」です。

    use Illuminate\Support\Facades\RateLimiter;

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perMinute = 5,
        function() {
            // メッセージを送信...
        }
    );

    if (! $executed) {
      return 'メッセージ送信回数が多すぎます！';
    }

必要に応じて、`attempt`メソッドに4番目の引数を提供できます。これは「減衰率」、つまり利用可能な試行回数がリセットされるまでの秒数です。たとえば、上記の例を修正して、2分ごとに5回の試行を許可するようにできます。

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perTwoMinutes = 5,
        function() {
            // メッセージを送信...
        },
        $decayRate = 120,
    );

<a name="manually-incrementing-attempts"></a>
### 手動での試行回数増加

レートリミッターと手動でやり取りしたい場合、他にもさまざまなメソッドが利用できます。たとえば、`tooManyAttempts`メソッドを呼び出して、指定されたレートリミッターキーが1分あたりの最大許容試行回数を超えたかどうかを判断できます。

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        return '試行回数が多すぎます！';
    }

    RateLimiter::increment('send-message:'.$user->id);

    // メッセージを送信...

あるいは、`remaining`メソッドを使用して、指定されたキーに残っている試行回数を取得できます。指定されたキーに残りの試行回数がある場合、`increment`メソッドを呼び出して試行回数の合計を増やすことができます。

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
        RateLimiter::increment('send-message:'.$user->id);

        // メッセージを送信...
    }

指定されたレートリミッターキーの値を1より大きく増やしたい場合は、`increment`メソッドに希望する量を指定できます。

    RateLimiter::increment('send-message:'.$user->id, amount: 5);

<a name="determining-limiter-availability"></a>
#### リミッターの可用性の確認

キーに試行回数が残っていない場合、`availableIn`メソッドは、さらに試行が可能になるまでの残り秒数を返します。

    use Illuminate\Support\Facades\RateLimiter;

    if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
        $seconds = RateLimiter::availableIn('send-message:'.$user->id);

        return $seconds.'秒後に再試行できます。';
    }

    RateLimiter::increment('send-message:'.$user->id);

    // メッセージを送信...

<a name="clearing-attempts"></a>
### 試行回数のクリア

`clear`メソッドを使用して、指定されたレートリミッターキーの試行回数をリセットできます。たとえば、指定されたメッセージが受信者によって読まれたときに試行回数をリセットできます。

    use App\Models\Message;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * メッセージを既読にする。
     */
    public function read(Message $message): Message
    {
        $message->markAsRead();

        RateLimiter::clear('send-message:'.$message->user_id);

        return $message;
    }
