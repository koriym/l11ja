# 設定

- [はじめに](#introduction)
- [環境設定](#environment-configuration)
    - [環境変数の種類](#environment-variable-types)
    - [環境設定の取得](#retrieving-environment-configuration)
    - [現在の環境の決定](#determining-the-current-environment)
    - [環境ファイルの暗号化](#encrypting-environment-files)
- [設定値へのアクセス](#accessing-configuration-values)
- [設定のキャッシュ](#configuration-caching)
- [設定の公開](#configuration-publishing)
- [デバッグモード](#debug-mode)
- [メンテナンスモード](#maintenance-mode)

<a name="introduction"></a>
## はじめに

Laravelフレームワークのすべての設定ファイルは、`config`ディレクトリに保存されています。各オプションには説明があるので、ファイルを見て、利用可能なオプションに慣れてください。

これらの設定ファイルを使用すると、データベース接続情報、メールサーバー情報などのようなものを設定できます。また、アプリケーションのタイムゾーンや暗号化キーなど、さまざまなコア設定値も設定できます。

<a name="the-about-command"></a>
#### `about`コマンド

Laravelは、`about` Artisanコマンドを介して、アプリケーションの設定、ドライバ、環境の概要を表示できます。

```shell
php artisan about
```

アプリケーション概要の出力の特定のセクションにのみ興味がある場合は、`--only`オプションを使用してそのセクションをフィルタリングできます：

```shell
php artisan about --only=environment
```

また、特定の設定ファイルの値を詳細に調べるには、`config:show` Artisanコマンドを使用できます：

```shell
php artisan config:show database
```

<a name="environment-configuration"></a>
## 環境設定

アプリケーションが実行されている環境に基づいて異なる設定値を持つと便利な場合があります。たとえば、ローカルでは本番サーバーとは異なるキャッシュドライバを使用したい場合があります。

これを簡単にするために、Laravelは[DotEnv](https://github.com/vlucas/phpdotenv) PHPライブラリを利用しています。新しいLaravelインストールでは、アプリケーションのルートディレクトリに、多くの一般的な環境変数を定義する`.env.example`ファイルが含まれています。Laravelのインストールプロセス中に、このファイルは自動的に`.env`にコピーされます。

Laravelのデフォルトの`.env`ファイルには、アプリケーションがローカルで実行されているか本番Webサーバーで実行されているかに基づいて異なる可能性のある一般的な設定値が含まれています。これらの値は、Laravelの`env`関数を使用して`config`ディレクトリ内の設定ファイルから読み取られます。

チームで開発している場合、アプリケーションに`.env.example`ファイルを含めて更新し続けることができます。例の設定ファイルにプレースホルダー値を入れることで、チームの他の開発者は、アプリケーションを実行するために必要な環境変数を明確に確認できます。

> NOTE:  
> `.env`ファイル内のすべての変数は、サーバーレベルまたはシステムレベルの環境変数などの外部環境変数によってオーバーライドできます。

<a name="environment-file-security"></a>
#### 環境ファイルのセキュリティ

`.env`ファイルをアプリケーションのソース管理にコミットしてはいけません。アプリケーションを使用する各開発者/サーバーには、異なる環境設定が必要になる可能性があるためです。さらに、これはセキュリティリスクになります。侵入者がソース管理リポジトリにアクセスした場合、すべての機密資格情報が公開されるためです。

ただし、Laravelの組み込みの[環境暗号化](#encrypting-environment-files)を使用して環境ファイルを暗号化することは可能です。暗号化された環境ファイルは、アプリケーションの残りの部分とともにソース管理に安全に追加できます。

<a name="additional-environment-files"></a>
#### 追加の環境ファイル

アプリケーションの環境変数をロードする前に、Laravelは`APP_ENV`環境変数が外部から提供されているか、`--env` CLI引数が指定されているかを判断します。もしそうなら、Laravelは存在する場合に`.env.[APP_ENV]`ファイルをロードしようとします。存在しない場合は、デフォルトの`.env`ファイルがロードされます。

<a name="environment-variable-types"></a>
### 環境変数の種類

`.env`ファイル内のすべての変数は通常文字列として解析されるため、`env()`関数からより広範な型を返すことができるように、いくつかの予約値が作成されています：

<div class="overflow-auto" markdown=1>

| `.env` 値 | `env()` 値 |
| ------------ | ------------- |
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

</div>

スペースを含む値を持つ環境変数を定義する必要がある場合は、値を二重引用符で囲むことでそれを行うことができます：

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### 環境設定の取得

`.env`ファイルにリストされているすべての変数は、アプリケーションがリクエストを受け取るときに`$_ENV` PHPスーパーグローバルにロードされます。ただし、設定ファイル内のこれらの変数の値を取得するには、`env`関数を使用できます。実際、Laravelの設定ファイルを確認すると、多くのオプションがすでにこの関数を使用していることがわかります：

    'debug' => env('APP_DEBUG', false),

`env`関数に渡される2番目の値は「デフォルト値」です。この値は、指定されたキーに対して環境変数が存在しない場合に返されます。

<a name="determining-the-current-environment"></a>
### 現在の環境の決定

現在のアプリケーション環境は、`.env`ファイルの`APP_ENV`変数によって決定されます。この値には、`App` [facade](facades.md)の`environment`メソッドを介してアクセスできます：

    use Illuminate\Support\Facades\App;

    $environment = App::environment();

`environment`メソッドに引数を渡して、環境が指定された値と一致するかどうかを判断することもできます。メソッドは、環境が指定された値のいずれかと一致する場合に`true`を返します：

    if (App::environment('local')) {
        // 環境はローカルです
    }

    if (App::environment(['local', 'staging'])) {
        // 環境はローカルまたはステージングです...
    }

> NOTE:  
> 現在のアプリケーション環境の検出は、サーバーレベルの`APP_ENV`環境変数を定義することでオーバーライドできます。

<a name="encrypting-environment-files"></a>
### 環境ファイルの暗号化

暗号化されていない環境ファイルをソース管理に保存してはいけません。ただし、Laravelを使用すると、環境ファイルを暗号化して、アプリケーションの残りの部分とともにソース管理に安全に追加できます。

<a name="encryption"></a>
#### 暗号化

環境ファイルを暗号化するには、`env:encrypt`コマンドを使用できます：

```shell
php artisan env:encrypt
```

`env:encrypt`コマンドを実行すると、`.env`ファイルが暗号化され、暗号化された内容が`.env.encrypted`ファイルに配置されます。復号化キーはコマンドの出力に表示され、安全なパスワードマネージャに保存する必要があります。独自の暗号化キーを提供する場合は、コマンドを呼び出すときに`--key`オプションを使用できます：

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> NOTE:  
> 提供されるキーの長さは、使用される暗号化暗号に必要なキーの長さと一致する必要があります。デフォルトでは、Laravelは`AES-256-CBC`暗号を使用します。これには32文字のキーが必要です。コマンドを呼び出すときに`--cipher`オプションを渡すことで、Laravelの[暗号化](encryption.md)でサポートされている任意の暗号を自由に使用できます。

アプリケーションに`.env`や`.env.staging`などの複数の環境ファイルがある場合、`--env`オプションを介して暗号化する必要がある環境ファイルを指定できます：

```shell
php artisan env:encrypt --env=staging
```

<a name="decryption"></a>
#### 復号化

環境ファイルを復号化するには、`env:decrypt`コマンドを使用できます。このコマンドには復号化キーが必要で、Laravelは`LARAVEL_ENV_ENCRYPTION_KEY`環境変数から取得します：

```shell
php artisan env:decrypt
```

または、キーを`--key`オプションを介してコマンドに直接提供できます：

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

`env:decrypt`コマンドが呼び出されると、Laravelは`.env.encrypted`ファイルの内容を復号化し、復号化された内容を`.env`ファイルに配置します。

`--cipher`オプションを`env:decrypt`コマンドに提供して、カスタム暗号化暗号を使用することができます：

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

アプリケーションに`.env`や`.env.staging`などの複数の環境ファイルがある場合、`--env`オプションを介して復号化する必要がある環境ファイルを指定できます：

```shell
php artisan env:decrypt --env=staging
```

既存の環境ファイルを上書きするには、`env:decrypt`コマンドに`--force`オプションを提供できます：

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## 設定値へのアクセス

アプリケーション内のどこからでも`Config` facadeまたはグローバル`config`関数を使用して設定値に簡単にアクセスできます。設定値には「ドット」構文を使用してアクセスできます。これには、アクセスしたいファイルとオプションの名前が含まれます。デフォルト値を指定することもでき、設定オプションが存在しない場合に返されます：

    use Illuminate\Support\Facades\Config;

    $value = Config::get('app.timezone');

    $value = config('app.timezone');

    // 設定値が存在しない場合にデフォルト値を取得...
    $value = config('app.timezone', 'Asia/Seoul');

ランタイム時に設定値を設定するには、`Config`ファサードの`set`メソッドを呼び出すか、配列を`config`関数に渡します。

    Config::set('app.timezone', 'America/Chicago');

    config(['app.timezone' => 'America/Chicago']);

静的解析を支援するために、`Config`ファサードは型付きの設定取得メソッドも提供しています。取得された設定値が期待される型と一致しない場合、例外がスローされます。

    Config::string('config-key');
    Config::integer('config-key');
    Config::float('config-key');
    Config::boolean('config-key');
    Config::array('config-key');

<a name="configuration-caching"></a>
## 設定のキャッシュ

アプリケーションの速度を向上させるために、`config:cache` Artisanコマンドを使用して、すべての設定ファイルを単一のファイルにキャッシュする必要があります。これにより、アプリケーションのすべての設定オプションが1つのファイルに結合され、フレームワークによって迅速に読み込まれるようになります。

通常、`php artisan config:cache`コマンドは本番環境のデプロイプロセスの一部として実行する必要があります。このコマンドは、アプリケーションの開発中に設定オプションを頻繁に変更する必要があるため、ローカル開発中に実行すべきではありません。

設定がキャッシュされると、アプリケーションの`.env`ファイルはリクエストやArtisanコマンド中にフレームワークによって読み込まれません。したがって、`env`関数はシステムレベルの環境変数のみを返します。

このため、アプリケーションの設定（`config`）ファイル内からのみ`env`関数を呼び出すようにしてください。Laravelのデフォルト設定ファイルを調べることで、この例を多く見ることができます。設定値は、[前述](#accessing-configuration-values)の`config`関数を使用して、アプリケーション内のどこからでもアクセスできます。

キャッシュされた設定をクリアするには、`config:clear`コマンドを使用できます。

```shell
php artisan config:clear
```

> WARNING:  
> デプロイプロセス中に`config:cache`コマンドを実行する場合、設定ファイル内からのみ`env`関数を呼び出していることを確認してください。設定がキャッシュされると、`.env`ファイルは読み込まれません。したがって、`env`関数はシステムレベルの環境変数のみを返します。

<a name="configuration-publishing"></a>
## 設定の公開

Laravelのほとんどの設定ファイルは、既にアプリケーションの`config`ディレクトリに公開されています。ただし、`cors.php`や`view.php`などの特定の設定ファイルは、デフォルトでは公開されていません。ほとんどのアプリケーションはこれらを変更する必要がないためです。

ただし、`config:publish` Artisanコマンドを使用して、デフォルトで公開されていない設定ファイルを公開することができます。

```shell
php artisan config:publish

php artisan config:publish --all
```

<a name="debug-mode"></a>
## デバッグモード

`config/app.php`設定ファイルの`debug`オプションは、エラーに関するどの程度の情報がユーザーに実際に表示されるかを決定します。デフォルトでは、このオプションは`.env`ファイルに保存されている`APP_DEBUG`環境変数の値を尊重するように設定されています。

> WARNING:  
> ローカル開発では、`APP_DEBUG`環境変数を`true`に設定する必要があります。**本番環境では、この値は常に`false`である必要があります。本番環境でこの変数が`true`に設定されていると、アプリケーションのエンドユーザーに機密性の高い設定値が公開されるリスクがあります。**

<a name="maintenance-mode"></a>
## メンテナンスモード

アプリケーションがメンテナンスモードの場合、アプリケーションへのすべてのリクエストに対してカスタムビューが表示されます。これにより、更新中やメンテナンス中にアプリケーションを「無効化」することが容易になります。メンテナンスモードのチェックは、アプリケーションのデフォルトのミドルウェアスタックに含まれています。アプリケーションがメンテナンスモードの場合、ステータスコード503で`Symfony\Component\HttpKernel\Exception\HttpException`インスタンスがスローされます。

メンテナンスモードを有効にするには、`down` Artisanコマンドを実行します。

```shell
php artisan down
```

すべてのメンテナンスモードのレスポンスで`Refresh` HTTPヘッダーを送信したい場合は、`down`コマンドを呼び出す際に`refresh`オプションを指定できます。`Refresh`ヘッダーは、指定された秒数後にブラウザに自動的にページを更新するよう指示します。

```shell
php artisan down --refresh=15
```

また、`down`コマンドに`retry`オプションを指定することもできます。これは`Retry-After` HTTPヘッダーの値として設定されますが、通常ブラウザはこのヘッダーを無視します。

```shell
php artisan down --retry=60
```

<a name="bypassing-maintenance-mode"></a>
#### メンテナンスモードのバイパス

シークレットトークンを使用してメンテナンスモードをバイパスできるようにするには、`secret`オプションを使用してメンテナンスモードバイパストークンを指定できます。

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

アプリケーションをメンテナンスモードにした後、このトークンに一致するアプリケーションURLに移動すると、Laravelはメンテナンスモードバイパスクッキーをブラウザに発行します。

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

Laravelにシークレットトークンを生成させたい場合は、`with-secret`オプションを使用できます。シークレットは、アプリケーションがメンテナンスモードになった後に表示されます。

```shell
php artisan down --with-secret
```

この隠しルートにアクセスすると、アプリケーションの`/`ルートにリダイレクトされます。クッキーがブラウザに発行されると、メンテナンスモードでないかのようにアプリケーションを通常通り閲覧できます。

> NOTE:  
> メンテナンスモードのシークレットは、通常、英数字とオプションでダッシュで構成されるべきです。URLで特別な意味を持つ文字（例：`?`や`&`）は避けるべきです。

<a name="maintenance-mode-on-multiple-servers"></a>
#### 複数サーバーでのメンテナンスモード

デフォルトでは、Laravelはファイルベースのシステムを使用してアプリケーションがメンテナンスモードかどうかを判断します。これは、メンテナンスモードを有効にするために、`php artisan down`コマンドをアプリケーションをホストしている各サーバーで実行する必要があることを意味します。

あるいは、Laravelはメンテナンスモードを処理するためのキャッシュベースの方法も提供しています。この方法では、`php artisan down`コマンドを1つのサーバーで実行するだけで済みます。このアプローチを使用するには、アプリケーションの`config/app.php`ファイルの「driver」設定を`cache`に変更します。次に、すべてのサーバーからアクセス可能なキャッシュ`store`を選択します。これにより、メンテナンスモードのステータスがすべてのサーバーで一貫して維持されます。

```php
'maintenance' => [
    'driver' => 'cache',
    'store' => 'database',
],
```

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### メンテナンスモードビューのプリレンダリング

デプロイ中に`php artisan down`コマンドを使用する場合、Composerの依存関係やその他のインフラストラクチャコンポーネントが更新されている間に、ユーザーがアプリケーションにアクセスすると、エラーが発生することがあります。これは、Laravelフレームワークの大部分が起動してアプリケーションがメンテナンスモードであることを判断し、テンプレートエンジンを使用してメンテナンスモードビューをレンダリングする必要があるためです。

このため、Laravelでは、リクエストサイクルの非常に早い段階で返されるメンテナンスモードビューをプリレンダリングできます。このビューは、アプリケーションの依存関係がいずれもロードされる前にレンダリングされます。`down`コマンドの`render`オプションを使用して、選択したテンプレートをプリレンダリングできます。

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### メンテナンスモードリクエストのリダイレクト

メンテナンスモード中に、ユーザーがアクセスしようとするすべてのアプリケーションURLに対してメンテナンスモードビューが表示されます。必要に応じて、Laravelにすべてのリクエストを特定のURLにリダイレクトするよう指示できます。これは、`redirect`オプションを使用して行うことができます。たとえば、すべてのリクエストを`/` URIにリダイレクトすることができます。

```shell
php artisan down --redirect=/
```

<a name="disabling-maintenance-mode"></a>
#### メンテナンスモードの無効化

メンテナンスモードを無効にするには、`up`コマンドを使用します。

```shell
php artisan up
```

> NOTE:  
> デフォルトのメンテナンスモードテンプレートは、`resources/views/errors/503.blade.php`に独自のテンプレートを定義することでカスタマイズできます。

<a name="maintenance-mode-queues"></a>
#### メンテナンスモードとキュー

アプリケーションがメンテナンスモードの間、[キュージョブ](queues.md)は処理されません。アプリケーションがメンテナンスモードから抜けると、ジョブは通常通り処理されます。

<a name="alternatives-to-maintenance-mode"></a>
#### メンテナンスモードの代替手段

メンテナンスモードでは、アプリケーションに数秒のダウンタイムが必要になるため、Laravel VaporやEnvoyerなどの代替手段を検討して、Laravelでゼロダウンタイムデプロイを実現してください。

