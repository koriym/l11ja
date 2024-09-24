# コンテキスト

- [はじめに](#introduction)
    - [仕組み](#how-it-works)
- [コンテキストのキャプチャ](#capturing-context)
    - [スタック](#stacks)
- [コンテキストの取得](#retrieving-context)
    - [アイテムの存在確認](#determining-item-existence)
- [コンテキストの削除](#removing-context)
- [隠しコンテキスト](#hidden-context)
- [イベント](#events)
    - [デハイドレート（シリアル化）
](#dehydrating)
    - [ハイドレート(復元）
](#hydrated)

<a name="introduction"></a>
## はじめに

Laravelの「コンテキスト」機能は、アプリケーション全体で情報を効率的に管理するための強力なツールです。この機能を使用すると、リクエスト、ジョブ、およびコマンドの実行全体で情報をキャプチャ、取得、共有することができます。

キャプチャされた情報は、アプリケーションのログにも自動的に含まれます。

これにより、ログエントリが書き込まれる前の周囲のコード実行履歴について、より深い洞察を得ることができます。 さらに、この機能は分散システム全体での実行フローのトレースを可能にします。これは、複雑なアプリケーションのデバッグや性能分析に非常に有用です。

<a name="how-it-works"></a>
### 仕組み

Laravelのコンテキスト機能を理解する最良の方法は、組み込みのロギング機能を使用して実際に動作を見ることです。まず、`Context`ファサードを使用して[コンテキストに情報を追加](#capturing-context)することができます。この例では、[ミドルウェア](middleware.md)を使用して、すべての受信リクエストに対してリクエストURLと一意のトレースIDをコンテキストに追加します。

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * 受信リクエストを処理します。
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```

コンテキストに追加された情報は、リクエスト全体で書き込まれるすべての[ログエントリ](logging.md)にメタデータとして自動的に追加されます。コンテキストをメタデータとして追加することで、個々のログエントリに渡される情報を`Context`を介して共有される情報と区別することができます。例えば、次のようなログエントリを書き込むとします。

```php
Log::info('ユーザーが認証されました。', ['auth_id' => Auth::id()]);
```

書き込まれたログには、ログエントリに渡された`auth_id`が含まれますが、コンテキストの`url`と`trace_id`もメタデータとして含まれます。

```
ユーザーが認証されました。 {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

コンテキストに追加された情報は、キューにディスパッチされるジョブでも利用可能です。例えば、コンテキストにいくつかの情報を追加した後に`ProcessPodcast`ジョブをキューにディスパッチするとします。

```php
// ミドルウェア内で...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// コントローラ内で...
ProcessPodcast::dispatch($podcast);
```

ジョブがディスパッチされると、現在コンテキストに保存されているすべての情報がキャプチャされ、ジョブと共有されます。キャプチャされた情報は、ジョブの実行中に現在のコンテキストに再びハイドレート(復元）
されます。したがって、ジョブのhandleメソッドがログに書き込む場合。

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * ジョブを実行します。
     */
    public function handle(): void
    {
        Log::info('ポッドキャストを処理しています。', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```

結果のログエントリには、ジョブを最初にディスパッチしたリクエスト中にコンテキストに追加された情報が含まれます。

```
ポッドキャストを処理しています。 {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

Laravelのコンテキストの組み込みロギング関連機能に焦点を当てましたが、以下のドキュメントでは、コンテキストがHTTPリクエスト/キューされたジョブの境界を越えて情報を共有する方法、さらには[隠しコンテキストデータ](#hidden-context)を追加する方法について説明します。

<a name="capturing-context"></a>
## コンテキストのキャプチャ

`Context`ファサードの`add`メソッドを使用して、現在のコンテキストに情報を保存できます。

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

一度に複数のアイテムを追加するには、連想配列を`add`メソッドに渡すことができます。

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

`add`メソッドは、同じキーを持つ既存の値を上書きします。キーがまだ存在しない場合にのみ情報をコンテキストに追加したい場合は、`addIf`メソッドを使用できます。

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

<a name="conditional-context"></a>
#### 条件付きコンテキスト

`when`メソッドを使用して、特定の条件に基づいてコンテキストにデータを追加できます。`when`メソッドに提供された最初のクロージャは、指定された条件が`true`と評価された場合に呼び出され、2番目のクロージャは条件が`false`と評価された場合に呼び出されます。

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="stacks"></a>
### スタック

コンテキストは、「スタック」を作成する機能を提供します。スタックは、追加された順序でデータを保存するリストです。`push`メソッドを呼び出してスタックに情報を追加できます。

```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

スタックは、アプリケーション全体で発生しているイベントなど、リクエストに関する履歴情報をキャプチャするのに便利です。例えば、クエリが実行されるたびにイベントリスナーを作成してスタックにプッシュし、クエリSQLと期間をタプルとしてキャプチャすることができます。

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

`stackContains`および`hiddenStackContains`メソッドを使用して、スタック内に値が存在するかどうかを判断できます。

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

`stackContains`および`hiddenStackContains`メソッドは、2番目の引数としてクロージャも受け入れ、値の比較操作をより細かく制御できます。

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## コンテキストの取得

`Context`ファサードの`get`メソッドを使用して、コンテキストから情報を取得できます。

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

`only`メソッドを使用して、コンテキスト内の情報のサブセットを取得できます。

```php
$data = Context::only(['first_key', 'second_key']);
```

`pull`メソッドを使用して、コンテキストから情報を取得し、すぐにコンテキストから削除できます。

```php
$value = Context::pull('key');
```

コンテキストに保存されているすべての情報を取得したい場合は、`all`メソッドを呼び出すことができます。

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### アイテムの存在確認

`has`メソッドを使用して、コンテキストに指定されたキーの値が保存されているかどうかを判断できます。

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}
```

`has`メソッドは、保存されている値に関係なく`true`を返します。したがって、例えば、`null`値を持つキーは存在すると見なされます。

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## コンテキストの削除

`forget`メソッドを使用して、現在のコンテキストからキーとその値を削除できます。

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

`forget`メソッドに配列を提供することで、一度に複数のキーを忘れることができます。

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## 隠しコンテキスト

コンテキストは、「隠し」データを保存する機能を提供します。この隠し情報はログに追加されず、上記のデータ取得メソッドではアクセスできません。コンテキストは、隠しコンテキスト情報と対話するための別のメソッドセットを提供します。

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

「隠し」メソッドは、上記で文書化された非隠しメソッドの機能を反映しています。

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## イベント

コンテキストは、コンテキストのハイドレート(復元）
とデハイドレート（シリアル化）
プロセスにフックするための2つのイベントをディスパッチします。

- [デハイドレート（シリアル化）
](#dehydrating)
- [ハイドレート(復元）
](#hydrated)

これらのイベントがどのように使用されるかを説明するために、アプリケーションのミドルウェアで、受信したHTTPリクエストの`Accept-Language`ヘッダに基づいて`app.locale`設定値を設定すると想像してみてください。コンテキストのイベントを使用すると、この値をリクエスト中にキャプチャし、キュー上で復元することができ、キュー上で送信される通知が正しい`app.locale`値を持つようになります。コンテキストのイベントと[隠し](#hidden-context)データを使用してこれを実現する方法を、以下のドキュメントで説明します。

<a name="dehydrating"></a>
### デハイドレート（シリアル化）
（Dehydrating）

ジョブがキューにディスパッチ（実行）されるたびに、コンテキスト内のデータは「デハイドレート（シリアル化）
」され、ジョブのペイロードとともにキャプチャされます。`Context::dehydrating`メソッドを使用すると、デハイドレート（シリアル化）
プロセス中に呼び出されるクロージャを登録できます。このクロージャ内で、キューに入れられたジョブと共有されるデータに変更を加えることができます。

通常、`dehydrating`コールバックは、アプリケーションの`AppServiceProvider`クラスの`boot`メソッド内で登録する必要があります。

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

> NOTE:  
> `dehydrating`コールバック内で`Context`ファサードを使用しないでください。それは現在のプロセスのコンテキストを変更するためです。コールバックに渡されたリポジトリにのみ変更を加えるようにしてください。

<a name="hydrated"></a>
### ハイドレート(復元）
（Hydrated）

キュー上でジョブの実行が開始されるたびに、ジョブと共有されたコンテキストは現在のコンテキストに「ハイドレート(復元）
」されます。`Context::hydrated`メソッドを使用すると、ハイドレート(復元）
プロセス中に呼び出されるクロージャを登録できます。

通常、`hydrated`コールバックは、アプリケーションの`AppServiceProvider`クラスの`boot`メソッド内で登録する必要があります。

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

> NOTE:  
> `hydrated`コールバック内で`Context`ファサードを使用しないでください。代わりに、コールバックに渡されたリポジトリにのみ変更を加えるようにしてください。

