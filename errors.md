# エラー処理

- [はじめに](#introduction)
- [設定](#configuration)
- [例外の処理](#handling-exceptions)
    - [例外の報告](#reporting-exceptions)
    - [例外のログレベル](#exception-log-levels)
    - [例外の種類による無視](#ignoring-exceptions-by-type)
    - [例外のレンダリング](#rendering-exceptions)
    - [報告可能およびレンダリング可能な例外](#renderable-exceptions)
- [報告された例外のスロットリング](#throttling-reported-exceptions)
- [HTTP例外](#http-exceptions)
    - [カスタムHTTPエラーページ](#custom-http-error-pages)

<a name="introduction"></a>
## はじめに

新しいLaravelプロジェクトを開始すると、エラーと例外の処理はすでに設定されています。しかし、いつでもアプリケーションの`bootstrap/app.php`で`withExceptions`メソッドを使用して、アプリケーションによって例外がどのように報告およびレンダリングされるかを管理できます。

`withExceptions`クロージャに提供される`$exceptions`オブジェクトは、`Illuminate\Foundation\Configuration\Exceptions`のインスタンスであり、アプリケーション内の例外処理を管理する役割を担っています。このドキュメント全体を通して、このオブジェクトについて詳しく説明します。

<a name="configuration"></a>
## 設定

`config/app.php`設定ファイルの`debug`オプションは、エラーに関するどれだけの情報がユーザーに実際に表示されるかを決定します。デフォルトでは、このオプションは`.env`ファイルに保存されている`APP_DEBUG`環境変数の値を尊重するように設定されています。

ローカル開発中は、`APP_DEBUG`環境変数を`true`に設定する必要があります。**本番環境では、この値は常に`false`である必要があります。本番環境でこの値が`true`に設定されている場合、アプリケーションのエンドユーザーに機密性の高い設定値を公開するリスクがあります。**

<a name="handling-exceptions"></a>
## 例外の処理

<a name="reporting-exceptions"></a>
### 例外の報告

Laravelでは、例外の報告は例外をログに記録したり、外部サービス[Sentry](https://github.com/getsentry/sentry-laravel)や[Flare](https://flareapp.io)に送信するために使用されます。デフォルトでは、例外は[ログ](logging.md)設定に基づいてログに記録されます。ただし、例外をどのようにログに記録するかは自由に選択できます。

異なるタイプの例外を異なる方法で報告する必要がある場合、アプリケーションの`bootstrap/app.php`で`report`例外メソッドを使用して、特定のタイプの例外を報告する必要があるときに実行されるクロージャを登録できます。Laravelは、クロージャのタイプヒントを調べることで、クロージャがどのタイプの例外を報告するかを判断します。

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->report(function (InvalidOrderException $e) {
            // ...
        });
    })

`report`メソッドを使用してカスタム例外報告コールバックを登録する場合、Laravelは依然としてアプリケーションのデフォルトログ設定を使用して例外をログに記録します。例外をデフォルトのログスタックに伝播させたくない場合は、報告コールバックを定義する際に`stop`メソッドを使用するか、コールバックから`false`を返すことができます。

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->report(function (InvalidOrderException $e) {
            // ...
        })->stop();

        $exceptions->report(function (InvalidOrderException $e) {
            return false;
        });
    })

> NOTE:  
> 特定の例外の例外報告をカスタマイズするために、[報告可能な例外](errors.md#renderable-exceptions)を利用することもできます。

<a name="global-log-context"></a>
#### グローバルログコンテキスト

利用可能な場合、Laravelは自動的に現在のユーザーのIDをすべての例外のログメッセージにコンテキストデータとして追加します。アプリケーションの`bootstrap/app.php`ファイルで`context`例外メソッドを使用して、独自のグローバルコンテキストデータを定義できます。この情報は、アプリケーションによって書き込まれるすべての例外のログメッセージに含まれます。

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->context(fn () => [
            'foo' => 'bar',
        ]);
    })

<a name="exception-log-context"></a>
#### 例外ログコンテキスト

すべてのログメッセージにコンテキストを追加することは便利ですが、特定の例外にはログに含めたい一意のコンテキストがある場合があります。アプリケーションの例外の1つに`context`メソッドを定義することで、その例外のログエントリに追加する必要がある関連データを指定できます。

    <?php

    namespace App\Exceptions;

    use Exception;

    class InvalidOrderException extends Exception
    {
        // ...

        /**
         * 例外のコンテキスト情報を取得します。
         *
         * @return array<string, mixed>
         */
        public function context(): array
        {
            return ['order_id' => $this->orderId];
        }
    }

<a name="the-report-helper"></a>
#### `report`ヘルパー

例外を報告する必要があるが、現在のリクエストの処理を続行する必要がある場合、`report`ヘルパー関数を使用して、ユーザーにエラーページをレンダリングせずに迅速に例外を報告できます。

    public function isValid(string $value): bool
    {
        try {
            // 値を検証...
        } catch (Throwable $e) {
            report($e);

            return false;
        }
    }

<a name="deduplicating-reported-exceptions"></a>
#### 報告された例外の重複排除

アプリケーション全体で`report`関数を使用している場合、同じ例外を複数回報告し、ログに重複エントリを作成することがあります。

特定の例外の単一インスタンスが一度だけ報告されるようにする場合、アプリケーションの`bootstrap/app.php`ファイルで`dontReportDuplicates`例外メソッドを呼び出すことができます。

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->dontReportDuplicates();
    })

これで、同じ例外インスタンスで`report`ヘルパーが呼び出された場合、最初の呼び出しのみが報告されます。

```php
$original = new RuntimeException('Whoops!');

report($original); // 報告される

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // 無視される
}

report($original); // 無視される
report($caught); // 無視される
```

<a name="exception-log-levels"></a>
### 例外ログレベル

アプリケーションの[ログ](logging.md)にメッセージが書き込まれるとき、メッセージは指定された[ログレベル](logging.md#log-levels)で書き込まれます。これは、ログされるメッセージの重大度や重要性を示します。

前述のように、`report`メソッドを使用してカスタム例外報告コールバックを登録する場合でも、Laravelはアプリケーションのデフォルトログ設定を使用して例外をログに記録します。ただし、ログレベルによってはメッセージがログされるチャネルに影響を与える場合があるため、特定の例外がログされるログレベルを設定したい場合があります。

これを実現するには、アプリケーションの`bootstrap/app.php`ファイルで`level`例外メソッドを使用できます。このメソッドは、最初の引数として例外タイプ、2番目の引数としてログレベルを受け取ります。

    use PDOException;
    use Psr\Log\LogLevel;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->level(PDOException::class, LogLevel::CRITICAL);
    })

<a name="ignoring-exceptions-by-type"></a>
### 例外の種類による無視

アプリケーションを構築する際、報告したくない例外の種類があるでしょう。これらの例外を無視するには、アプリケーションの`bootstrap/app.php`ファイルで`dontReport`例外メソッドを使用できます。このメソッドに提供されたクラスは、報告されることはありません。ただし、カスタムレンダリングロジックを持つことはできます。

    use App\Exceptions\InvalidOrderException;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->dontReport([
            InvalidOrderException::class,
        ]);
    })

または、単に例外クラスに`Illuminate\Contracts\Debug\ShouldntReport`インターフェースを「マーク」することもできます。このインターフェースでマークされた例外は、Laravelの例外ハンドラによって報告されることはありません。

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

内部的には、Laravelはすでに404 HTTPエラーや無効なCSRFトークンによって生成された419 HTTPレスポンスなど、いくつかのタイプのエラーを無視しています。特定のタイプの例外を無視するのをやめさせたい場合は、アプリケーションの`bootstrap/app.php`ファイルで`stopIgnoring`例外メソッドを使用できます。

    use Symfony\Component\HttpKernel\Exception\HttpException;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->stopIgnoring(HttpException::class);
    })

<a name="rendering-exceptions"></a>
### 例外のレンダリング

デフォルトでは、Laravelの例外ハンドラは例外をHTTPレスポンスに変換します。ただし、特定のタイプの例外に対してカスタムレンダリングクロージャを自由に登録できます。これは、アプリケーションの`bootstrap/app.php`ファイルで`render`例外メソッドを使用して実現できます。

`render`メソッドに渡されるクロージャは、`Illuminate\Http\Response`のインスタンスを返す必要があります。これは、`response`ヘルパーを介して生成される可能性があります。Laravelは、クロージャのタイプヒントを調べることで、クロージャがどのタイプの例外をレンダリングするかを判断します。

    use App\Exceptions\InvalidOrderException;
    use Illuminate\Http\Request;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->render(function (InvalidOrderException $e, Request $request) {
            return response()->view('errors.invalid-order', status: 500);
        });
    })

`render`メソッドを使用して、`NotFoundHttpException`のようなLaravelやSymfonyの組み込み例外のレンダリング動作をオーバーライドすることもできます。`render`メソッドに渡されたクロージャが値を返さない場合、Laravelのデフォルトの例外レンダリングが使用されます。

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

<a name="rendering-exceptions-as-json"></a>
#### 例外をJSONとしてレンダリングする

例外をレンダリングする際、Laravelはリクエストの`Accept`ヘッダに基づいて、例外をHTMLとしてレンダリングするかJSONレスポンスとしてレンダリングするかを自動的に判断します。HTMLまたはJSON例外レスポンスをレンダリングするかどうかの判断方法をカスタマイズしたい場合は、`shouldRenderJsonWhen`メソッドを使用できます。

```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }

        return $request->expectsJson();
    });
})
```

<a name="customizing-the-exception-response"></a>
#### 例外レスポンスのカスタマイズ

まれに、Laravelの例外ハンドラによってレンダリングされるHTTPレスポンス全体をカスタマイズする必要があるかもしれません。これを実現するには、`respond`メソッドを使用してレスポンスカスタマイズクロージャを登録できます。

```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }

        return $response;
    });
})
```

<a name="renderable-exceptions"></a>
### レポート可能およびレンダリング可能な例外

アプリケーションの`bootstrap/app.php`ファイルでカスタムレポートとレンダリングの動作を定義する代わりに、アプリケーションの例外に直接`report`と`render`メソッドを定義することができます。これらのメソッドが存在する場合、フレームワークによって自動的に呼び出されます。

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * 例外をレポートする。
     */
    public function report(): void
    {
        // ...
    }

    /**
     * 例外をHTTPレスポンスにレンダリングする。
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

例外がLaravelやSymfonyの組み込み例外のように既にレンダリング可能な例外を拡張している場合、例外の`render`メソッドから`false`を返すことで、例外のデフォルトのHTTPレスポンスをレンダリングすることができます。

```php
/**
 * 例外をHTTPレスポンスにレンダリングする。
 */
public function render(Request $request): Response|bool
{
    if (/** 例外がカスタムレンダリングを必要とするかどうかを判断 */) {

        return response(/* ... */);
    }

    return false;
}
```

例外に、特定の条件が満たされた場合にのみ必要なカスタムレポートロジックが含まれている場合、Laravelにデフォルトの例外処理設定を使用して例外をレポートするように指示する必要があるかもしれません。これを実現するには、例外の`report`メソッドから`false`を返すことができます。

```php
/**
 * 例外をレポートする。
 */
public function report(): bool
{
    if (/** 例外がカスタムレポートを必要とするかどうかを判断 */) {

        // ...

        return true;
    }

    return false;
}
```

> NOTE:  
> `report`メソッドに必要な依存関係をタイプヒントで指定することができ、Laravelの[サービスコンテナ](container.md)によって自動的にメソッドに注入されます。

<a name="throttling-reported-exceptions"></a>
### レポートされる例外のスロットリング

アプリケーションが非常に多くの例外をレポートする場合、実際にログに記録される例外やアプリケーションの外部エラー追跡サービスに送信される例外の数をスロットリングしたい場合があります。

例外のランダムなサンプリングレートを取るには、アプリケーションの`bootstrap/app.php`ファイルで`throttle`例外メソッドを使用できます。`throttle`メソッドは、`Lottery`インスタンスを返すべきクロージャを受け取ります。

```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

例外の種類に基づいて条件付きでサンプリングすることも可能です。特定の例外クラスのインスタンスのみをサンプリングしたい場合、そのクラスに対してのみ`Lottery`インスタンスを返すことができます。

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

外部エラー追跡サービスにログに記録される例外や送信される例外をレートリミットすることもできます。これは、例えばアプリケーションが使用しているサードパーティサービスがダウンしている場合に、ログが急増するのを防ぐのに役立ちます。

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

デフォルトでは、制限は例外のクラスをレートリミットキーとして使用します。これをカスタマイズするには、`Limit`の`by`メソッドを使用して独自のキーを指定できます。

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

もちろん、異なる例外に対して`Lottery`と`Limit`インスタンスの組み合わせを返すこともできます。

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return match (true) {
            $e instanceof BroadcastException => Limit::perMinute(300),
            $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
            default => Limit::none(),
        };
    });
})
```

<a name="http-exceptions"></a>
## HTTP例外

サーバーからのHTTPエラーコードを記述する例外もあります。例えば、これは「ページが見つかりません」エラー（404）、「認証エラー」（401）、または開発者が生成した500エラーです。アプリケーションのどこからでもこのようなレスポンスを生成するには、`abort`ヘルパーを使用できます。

```php
abort(404);
```

<a name="custom-http-error-pages"></a>
### カスタムHTTPエラーページ

Laravelを使用すると、さまざまなHTTPステータスコードのカスタムエラーページを簡単に表示できます。例えば、404 HTTPステータスコードのエラーページをカスタマイズするには、`resources/views/errors/404.blade.php`ビューテンプレートを作成します。このビューは、アプリケーションによって生成されるすべての404エラーに対してレンダリングされます。このディレクトリ内のビューは、対応するHTTPステータスコードと一致するように名前を付ける必要があります。`abort`関数によって発生する`Symfony\Component\HttpKernel\Exception\HttpException`インスタンスは、`$exception`変数としてビューに渡されます。

```html
<h2>{{ $exception->getMessage() }}</h2>
```

Laravelのデフォルトのエラーページテンプレートを`vendor:publish` Artisanコマンドを使用して公開することができます。テンプレートが公開されたら、好みに合わせてカスタマイズできます。

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### フォールバックHTTPエラーページ

特定の一連のHTTPステータスコードに対して「フォールバック」エラーページを定義することもできます。このページは、発生した特定のHTTPステータスコードに対応するページがない場合にレンダリングされます。これを実現するには、アプリケーションの`resources/views/errors`ディレクトリに`4xx.blade.php`テンプレートと`5xx.blade.php`テンプレートを定義します。

フォールバックエラーページを定義する際、フォールバックページは`404`、`500`、`503`エラーレスポンスには影響しません。Laravelにはこれらのステータスコードに対する専用の内部ページがあるためです。これらのステータスコードに対してレンダリングされるページをカスタマイズするには、それぞれに対して個別にカスタムエラーページを定義する必要があります。

