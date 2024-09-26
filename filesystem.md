# ファイルストレージ

- [はじめに](#introduction)
- [設定](#configuration)
    - [ローカルドライバ](#the-local-driver)
    - [パブリックディスク](#the-public-disk)
    - [ドライバの前提条件](#driver-prerequisites)
    - [スコープ付きと読み取り専用のファイルシステム](#scoped-and-read-only-filesystems)
    - [Amazon S3互換ファイルシステム](#amazon-s3-compatible-filesystems)
- [ディスクインスタンスの取得](#obtaining-disk-instances)
    - [オンデマンドディスク](#on-demand-disks)
- [ファイルの取得](#retrieving-files)
    - [ファイルのダウンロード](#downloading-files)
    - [ファイルのURL](#file-urls)
    - [一時的なURL](#temporary-urls)
    - [ファイルのメタデータ](#file-metadata)
- [ファイルの保存](#storing-files)
    - [ファイルへの先頭と末尾の追加](#prepending-appending-to-files)
    - [ファイルのコピーと移動](#copying-moving-files)
    - [自動ストリーミング](#automatic-streaming)
    - [ファイルのアップロード](#file-uploads)
    - [ファイルの可視性](#file-visibility)
- [ファイルの削除](#deleting-files)
- [ディレクトリ](#directories)
- [テスト](#testing)
- [カスタムファイルシステム](#custom-filesystems)

<a name="introduction"></a>
## はじめに

Laravelは、Frank de Jongeによる優れた[Flysystem](https://github.com/thephpleague/flysystem) PHPパッケージのおかげで、強力なファイルシステムの抽象化を提供しています。Laravel Flysystemの統合により、ローカルファイルシステム、SFTP、Amazon S3を操作するためのシンプルなドライバが提供されます。さらに良いことに、ローカルの開発マシンと本番サーバーの間でこれらのストレージオプションを切り替えるのは驚くほど簡単で、APIはどのシステムでも同じままです。

<a name="configuration"></a>
## 設定

Laravelのファイルシステムの設定ファイルは、`config/filesystems.php`にあります。このファイル内で、すべてのファイルシステムの「ディスク」を設定できます。各ディスクは、特定のストレージドライバとストレージ場所を表します。サポートされている各ドライバの設定例が設定ファイルに含まれているため、設定をストレージの好みと資格情報に反映するように変更できます。

`local`ドライバは、Laravelアプリケーションを実行しているサーバー上にローカルに保存されたファイルと対話し、`s3`ドライバはAmazonのS3クラウドストレージサービスに書き込むために使用されます。

> NOTE:  
> 必要な数のディスkを設定でき、同じドライバを使用する複数のディスクを持つこともできます。

<a name="the-local-driver"></a>
### ローカルドライバ

`local`ドライバを使用する場合、すべてのファイル操作は`filesystems`設定ファイルで定義された`root`ディレクトリに対して相対的に行われます。デフォルトでは、この値は`storage/app`ディレクトリに設定されています。したがって、次のメソッドは`storage/app/example.txt`に書き込みます。

    use Illuminate\Support\Facades\Storage;

    Storage::disk('local')->put('example.txt', 'Contents');

<a name="the-public-disk"></a>
### パブリックディスク

アプリケーションの`filesystems`設定ファイルに含まれる`public`ディスクは、公開アクセス可能なファイルを対象としています。デフォルトでは、`public`ディスクは`local`ドライバを使用し、そのファイルを`storage/app/public`に保存します。

これらのファイルをWebからアクセス可能にするには、`public/storage`から`storage/app/public`へのシンボリックリンクを作成する必要があります。このフォルダの規則を使用すると、公開アクセス可能なファイルを1つのディレクトリにまとめることができ、[Envoyer](https://envoyer.io)のようなゼロダウンタイムデプロイメントシステムを使用する際に簡単に共有できます。

シンボリックリンクを作成するには、`storage:link` Artisanコマンドを使用します。

```shell
php artisan storage:link
```

ファイルが保存され、シンボリックリンクが作成されたら、`asset`ヘルパーを使用してファイルへのURLを作成できます。

    echo asset('storage/file.txt');

`filesystems`設定ファイルに追加のシンボリックリンクを設定できます。設定された各リンクは、`storage:link`コマンドを実行すると作成されます。

    'links' => [
        public_path('storage') => storage_path('app/public'),
        public_path('images') => storage_path('app/images'),
    ],

`storage:unlink`コマンドを使用して、設定されたシンボリックリンクを破棄できます。

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### ドライバの前提条件

<a name="s3-driver-configuration"></a>
#### S3ドライバの設定

S3ドライバを使用する前に、Composerパッケージマネージャを通じてFlysystem S3パッケージをインストールする必要があります。

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

S3ディスク設定配列は、`config/filesystems.php`設定ファイルにあります。通常、S3情報と資格情報を以下の環境変数を使用して設定する必要があります。これらの環境変数は、`config/filesystems.php`設定ファイルによって参照されます。

```
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

便宜上、これらの環境変数はAWS CLIで使用される命名規則と一致しています。

<a name="ftp-driver-configuration"></a>
#### FTPドライバの設定

FTPドライバを使用する前に、Composerパッケージマネージャを介してFlysystem FTPパッケージをインストールする必要があります。

```shell
composer require league/flysystem-ftp "^3.0"
```

LaravelのFlysystem統合はFTPでもうまく機能しますが、フレームワークのデフォルトの`config/filesystems.php`設定ファイルにサンプル設定は含まれていません。FTPファイルシステムを設定する必要がある場合は、以下の設定例を使用できます。

    'ftp' => [
        'driver' => 'ftp',
        'host' => env('FTP_HOST'),
        'username' => env('FTP_USERNAME'),
        'password' => env('FTP_PASSWORD'),

        // Optional FTP Settings...
        // 'port' => env('FTP_PORT', 21),
        // 'root' => env('FTP_ROOT'),
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

<a name="sftp-driver-configuration"></a>
#### SFTPドライバの設定

SFTPドライバを使用する前に、Composerパッケージマネージャを介してFlysystem SFTPパッケージをインストールする必要があります。

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

LaravelのFlysystem統合はSFTPでもうまく機能しますが、フレームワークのデフォルトの`config/filesystems.php`設定ファイルにサンプル設定は含まれていません。SFTPファイルシステムを設定する必要がある場合は、以下の設定例を使用できます。

    'sftp' => [
        'driver' => 'sftp',
        'host' => env('SFTP_HOST'),

        // Settings for basic authentication...
        'username' => env('SFTP_USERNAME'),
        'password' => env('SFTP_PASSWORD'),

        // Settings for SSH key based authentication with encryption password...
        'privateKey' => env('SFTP_PRIVATE_KEY'),
        'passphrase' => env('SFTP_PASSPHRASE'),

        // Settings for file / directory permissions...
        'visibility' => 'private', // `private` = 0600, `public` = 0644
        'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

        // Optional SFTP Settings...
        // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
        // 'maxTries' => 4,
        // 'passphrase' => env('SFTP_PASSPHRASE'),
        // 'port' => env('SFTP_PORT', 22),
        // 'root' => env('SFTP_ROOT', ''),
        // 'timeout' => 30,
        // 'useAgent' => true,
    ],

<a name="scoped-and-read-only-filesystems"></a>
### スコープ付きと読み取り専用のファイルシステム

スコープ付きディスクを使用すると、すべてのパスが指定されたパスプレフィックスで自動的にプレフィックスされるファイルシステムを定義できます。スコープ付きファイルシステムディスクを作成する前に、Composerパッケージマネージャを介して追加のFlysystemパッケージをインストールする必要があります。

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

既存のファイルシステムディスクのスコープ付きインスタンスを作成するには、`scoped`ドライバを使用するディスクを定義します。たとえば、既存の`s3`ディスクを特定のパスプレフィックスにスコープし、スコープ付きディスクを使用するすべてのファイル操作が指定されたプレフィックスを使用するようにすることができます。

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

「読み取り専用」ディスクを使用すると、書き込み操作を許可しないファイルシステムディスクを作成できます。`read-only`設定オプションを使用する前に、Composerパッケージマネージャを介して追加のFlysystemパッケージをインストールする必要があります。

```shell
composer require league/flysystem-read-only "^3.0"
```

次に、1つ以上のディスクの設定配列に`read-only`設定オプションを含めることができます。

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>
### Amazon S3互換ファイルシステム

デフォルトでは、アプリケーションの`filesystems`設定ファイルには`s3`ディスクのディスク設定が含まれています。このディスクを使用してAmazon S3と対話するだけでなく、[MinIO](https://github.com/minio/minio)や[DigitalOcean Spaces](https://www.digitalocean.com/products/spaces)などのS3互換ファイルストレージサービスと対話することもできます。

通常、ディスクの資格情報を使用するサービスの資格情報に更新した後、`endpoint`設定オプションの値を更新するだけで済みます。このオプションの値は、通常、`AWS_ENDPOINT`環境変数を介して定義されます。

    'endpoint' => env('AWS_ENDPOINT', 'https://minio:9000'),

<a name="minio"></a>
#### MinIO

LaravelのFlysystem統合がMinIOを使用する際に適切なURLを生成するために、アプリケーションのローカルURLに一致し、URLパスにバケット名を含める`AWS_URL`環境変数を定義する必要があります。

```ini
AWS_URL=http://localhost:9000/local
```

> WARNING:  
> `temporaryUrl`メソッドを使用して一時的なストレージURLを生成することは、`endpoint`がクライアントからアクセスできない場合、MinIOでは機能しない可能性があります。

<a name="obtaining-disk-instances"></a>
## ディスクインスタンスの取得

`Storage`ファサードを使用して、設定されたディスクのいずれかを操作できます。例えば、ファサードの`put`メソッドを使用して、アバターをデフォルトディスクに保存できます。`Storage`ファサードで`disk`メソッドを最初に呼び出さずにメソッドを呼び出すと、そのメソッドは自動的にデフォルトディスクに渡されます。

    use Illuminate\Support\Facades\Storage;

    Storage::put('avatars/1', $content);

アプリケーションが複数のディスクとやり取りする場合、`Storage`ファサードの`disk`メソッドを使用して、特定のディスク上のファイルを操作します。

    Storage::disk('s3')->put('avatars/1', $content);

<a name="on-demand-disks"></a>
### オンデマンドディスク

アプリケーションの`filesystems`設定ファイルに実際に存在しない設定を使用して、実行時にディスクを作成したい場合があります。これを実現するには、設定配列を`Storage`ファサードの`build`メソッドに渡すことができます。

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>
## ファイルの取得

`get`メソッドを使用して、ファイルの内容を取得できます。ファイルの生の文字列内容がメソッドによって返されます。すべてのファイルパスは、ディスクの「ルート」位置を基準に指定する必要があることに注意してください。

    $contents = Storage::get('file.jpg');

取得するファイルにJSONが含まれている場合、`json`メソッドを使用してファイルを取得し、その内容をデコードできます。

    $orders = Storage::json('orders.json');

`exists`メソッドを使用して、ファイルがディスク上に存在するかどうかを判断できます。

    if (Storage::disk('s3')->exists('file.jpg')) {
        // ...
    }

`missing`メソッドを使用して、ファイルがディスク上に存在しないかどうかを判断できます。

    if (Storage::disk('s3')->missing('file.jpg')) {
        // ...
    }

<a name="downloading-files"></a>
### ファイルのダウンロード

`download`メソッドを使用して、指定されたパスのファイルをユーザーのブラウザにダウンロードさせるレスポンスを生成できます。`download`メソッドは、メソッドの第2引数としてファイル名を受け取り、ユーザーがファイルをダウンロードする際に表示されるファイル名を決定します。最後に、メソッドの第3引数としてHTTPヘッダの配列を渡すことができます。

    return Storage::download('file.jpg');

    return Storage::download('file.jpg', $name, $headers);

<a name="file-urls"></a>
### ファイルのURL

`url`メソッドを使用して、指定されたファイルのURLを取得できます。`local`ドライバを使用している場合、これは通常、指定されたパスの前に`/storage`を付けて、ファイルへの相対URLを返します。`s3`ドライバを使用している場合、完全修飾されたリモートURLが返されます。

    use Illuminate\Support\Facades\Storage;

    $url = Storage::url('file.jpg');

`local`ドライバを使用している場合、公開アクセス可能であるべきすべてのファイルは`storage/app/public`ディレクトリに配置する必要があります。さらに、`storage/app/public`ディレクトリを指す`public/storage`に[シンボリックリンクを作成](#the-public-disk)する必要があります。

> WARNING:  
> `local`ドライバを使用している場合、`url`の戻り値はURLエンコードされません。このため、常に有効なURLを作成するファイル名を使用してファイルを保存することをお勧めします。

<a name="url-host-customization"></a>
#### URLホストのカスタマイズ

`Storage`ファサードを使用して生成されたURLのホストを変更したい場合は、ディスクの設定配列に`url`オプションを追加または変更できます。

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
        'throw' => false,
    ],

<a name="temporary-urls"></a>
### 一時的なURL

`temporaryUrl`メソッドを使用して、`local`および`s3`ドライバを使用して保存されたファイルへの一時的なURLを作成できます。このメソッドは、パスとURLが期限切れになる時期を指定する`DateTime`インスタンスを受け取ります。

    use Illuminate\Support\Facades\Storage;

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

<a name="enabling-local-temporary-urls"></a>
#### ローカル一時URLの有効化

一時URLのサポートが`local`ドライバに導入される前にアプリケーションの開発を開始した場合、ローカル一時URLを有効にする必要があるかもしれません。そのためには、`config/filesystems.php`設定ファイル内の`local`ディスクの設定配列に`serve`オプションを追加します。

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true, // [tl! add]
    'throw' => false,
],
```

<a name="s3-request-parameters"></a>
#### S3リクエストパラメータ

追加の[S3リクエストパラメータ](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests)を指定する必要がある場合、リクエストパラメータの配列を`temporaryUrl`メソッドの第3引数として渡すことができます。

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        [
            'ResponseContentType' => 'application/octet-stream',
            'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
        ]
    );

<a name="customizing-temporary-urls"></a>
#### 一時URLのカスタマイズ

特定のストレージディスクの一時URLの作成方法をカスタマイズする必要がある場合、`buildTemporaryUrlsUsing`メソッドを使用できます。例えば、これは通常一時URLをサポートしないディスクを介して保存されたファイルをダウンロードできるコントローラがある場合に便利です。通常、このメソッドはサービスプロバイダの`boot`メソッドから呼び出す必要があります。

    <?php

    namespace App\Providers;

    use DateTime;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\Facades\URL;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Storage::disk('local')->buildTemporaryUrlsUsing(
                function (string $path, DateTime $expiration, array $options) {
                    return URL::temporarySignedRoute(
                        'files.download',
                        $expiration,
                        array_merge($options, ['path' => $path])
                    );
                }
            );
        }
    }

<a name="temporary-upload-urls"></a>
#### 一時アップロードURL

> WARNING:  
> 一時アップロードURLの生成は、`s3`ドライバでのみサポートされています。

クライアントサイドアプリケーションから直接ファイルをアップロードするために使用できる一時URLを生成する必要がある場合、`temporaryUploadUrl`メソッドを使用できます。このメソッドは、URLが期限切れになる時期を指定する`DateTime`インスタンスとともにパスを受け取ります。`temporaryUploadUrl`メソッドは、アップロードURLとアップロードリクエストで含めるべきヘッダを分解可能な連想配列を返します。

    use Illuminate\Support\Facades\Storage;

    ['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
        'file.jpg', now()->addMinutes(5)
    );

このメソッドは、クライアントサイドアプリケーションがAmazon S3などのクラウドストレージシステムに直接ファイルをアップロードする必要があるサーバーレス環境で主に役立ちます。

<a name="file-metadata"></a>
### ファイルメタデータ

ファイルの読み書きに加えて、Laravelはファイル自体に関する情報も提供します。例えば、`size`メソッドを使用して、ファイルのサイズをバイト単位で取得できます。

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

`lastModified`メソッドは、ファイルが最後に変更されたUNIXタイムスタンプを返します。

    $time = Storage::lastModified('file.jpg');

指定されたファイルのMIMEタイプは、`mimeType`メソッドを介して取得できます。

    $mime = Storage::mimeType('file.jpg');

<a name="file-paths"></a>
#### ファイルパス

`path`メソッドを使用して、指定されたファイルのパスを取得できます。`local`ドライバを使用している場合、これはファイルへの絶対パスを返します。`s3`ドライバを使用している場合、このメソッドはS3バケット内のファイルへの相対パスを返します。

    use Illuminate\Support\Facades\Storage;

    $path = Storage::path('file.jpg');

<a name="storing-files"></a>
## ファイルの保存

`put`メソッドを使用して、ディスク上にファイルの内容を保存できます。また、PHPの`resource`を`put`メソッドに渡すこともでき、Flysystemの基礎となるストリームサポートを使用します。すべてのファイルパスは、ディスクに設定された「ルート」位置を基準に指定する必要があることに注意してください。

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

<a name="failed-writes"></a>
#### 書き込み失敗

`put`メソッド（または他の「書き込み」操作）がファイルをディスクに書き込めない場合、`false`が返されます。

    if (! Storage::put('file.jpg', $contents)) {
        // ファイルをディスクに書き込めませんでした...
    }

必要に応じて、ファイルシステムディスクの設定配列内で`throw`オプションを定義できます。このオプションが`true`として定義されている場合、`put`などの「書き込み」メソッドは、書き込み操作が失敗したときに`League\Flysystem\UnableToWriteFile`のインスタンスをスローします。

    'public' => [
        'driver' => 'local',
        // ...
        'throw' => true,
    ],

<a name="prepending-appending-to-files"></a>
### ファイルへの先頭と末尾への追加

`prepend`および`append`メソッドを使用すると、ファイルの先頭または末尾に書き込むことができます。

    Storage::prepend('file.log', 'Prepended Text');

    Storage::append('file.log', 'Appended Text');

<a name="copying-moving-files"></a>
### ファイルのコピーと移動

`copy`メソッドは、既存のファイルをディスク上の新しい場所にコピーするために使用できます。一方、`move`メソッドは、既存のファイルの名前を変更したり、新しい場所に移動したりするために使用できます。

    Storage::copy('old/file.jpg', 'new/file.jpg');

    Storage::move('old/file.jpg', 'new/file.jpg');

<a name="automatic-streaming"></a>
### 自動ストリーミング

ストレージへのファイルのストリーミングは、メモリ使用量を大幅に削減します。Laravelに指定されたファイルのストリーミングを自動的に管理させたい場合は、`putFile`または`putFileAs`メソッドを使用できます。このメソッドは、`Illuminate\Http\File`または`Illuminate\Http\UploadedFile`のインスタンスを受け取り、ファイルを目的の場所に自動的にストリーミングします。

    use Illuminate\Http\File;
    use Illuminate\Support\Facades\Storage;

    // ファイル名に一意のIDを自動生成...
    $path = Storage::putFile('photos', new File('/path/to/photo'));

    // ファイル名を手動で指定...
    $path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

`putFile`メソッドについて、いくつか重要な点に注意してください。ファイル名ではなくディレクトリ名のみを指定したことに注意してください。デフォルトでは、`putFile`メソッドは一意のIDを生成してファイル名として使用します。ファイルの拡張子は、ファイルのMIMEタイプを調べて決定されます。ファイルへのパスは`putFile`メソッドによって返されるため、パス（生成されたファイル名を含む）をデータベースに保存できます。

`putFile`および`putFileAs`メソッドは、保存されたファイルの「可視性」を指定するための引数も受け取ります。これは、Amazon S3などのクラウドディスクにファイルを保存し、生成されたURLを介してファイルを公開したい場合に特に便利です。

    Storage::putFile('photos', new File('/path/to/photo'), 'public');

<a name="file-uploads"></a>
### ファイルアップロード

Webアプリケーションでファイルを保存する最も一般的なユースケースの1つは、写真やドキュメントなどのユーザーがアップロードしたファイルを保存することです。Laravelでは、アップロードされたファイルインスタンスの`store`メソッドを使用して、アップロードされたファイルを簡単に保存できます。アップロードされたファイルを保存したいパスを指定して`store`メソッドを呼び出します。

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class UserAvatarController extends Controller
    {
        /**
         * ユーザーのアバターを更新する。
         */
        public function update(Request $request): string
        {
            $path = $request->file('avatar')->store('avatars');

            return $path;
        }
    }

この例について、いくつか重要な点に注意してください。ファイル名ではなくディレクトリ名のみを指定したことに注意してください。デフォルトでは、`store`メソッドは一意のIDを生成してファイル名として使用します。ファイルの拡張子は、ファイルのMIMEタイプを調べて決定されます。ファイルへのパスは`store`メソッドによって返されるため、パス（生成されたファイル名を含む）をデータベースに保存できます。

また、`Storage`ファサードの`putFile`メソッドを呼び出して、上記の例と同じファイル保存操作を実行することもできます。

    $path = Storage::putFile('avatars', $request->file('avatar'));

<a name="specifying-a-file-name"></a>
#### ファイル名の指定

保存されたファイルに自動的にファイル名を割り当てたくない場合は、`storeAs`メソッドを使用できます。このメソッドは、パス、ファイル名、および（オプションの）ディスクを引数として受け取ります。

    $path = $request->file('avatar')->storeAs(
        'avatars', $request->user()->id
    );

また、`Storage`ファサードの`putFileAs`メソッドを使用することもできます。これは、上記の例と同じファイル保存操作を実行します。

    $path = Storage::putFileAs(
        'avatars', $request->file('avatar'), $request->user()->id
    );

> WARNING:  
> 印刷不可能な文字と無効なユニコード文字は、ファイルパスから自動的に削除されます。したがって、Laravelのファイルストレージメソッドに渡す前に、ファイルパスをサニタイズすることをお勧めします。ファイルパスは、`League\Flysystem\WhitespacePathNormalizer::normalizePath`メソッドを使用して正規化されます。

<a name="specifying-a-disk"></a>
#### ディスクの指定

デフォルトでは、このアップロードされたファイルの`store`メソッドはデフォルトのディスクを使用します。別のディスクを指定したい場合は、ディスク名を`store`メソッドの第2引数として渡します。

    $path = $request->file('avatar')->store(
        'avatars/'.$request->user()->id, 's3'
    );

`storeAs`メソッドを使用する場合、ディスク名をメソッドの第3引数として渡すことができます。

    $path = $request->file('avatar')->storeAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="other-uploaded-file-information"></a>
#### その他のアップロードされたファイル情報

アップロードされたファイルの元の名前と拡張子を取得したい場合は、`getClientOriginalName`および`getClientOriginalExtension`メソッドを使用できます。

    $file = $request->file('avatar');

    $name = $file->getClientOriginalName();
    $extension = $file->getClientOriginalExtension();

ただし、`getClientOriginalName`および`getClientOriginalExtension`メソッドは安全でないと見なされることに注意してください。ファイル名と拡張子は悪意のあるユーザーによって改ざんされる可能性があります。このため、通常は`hashName`および`extension`メソッドを使用して、指定されたファイルアップロードの名前と拡張子を取得することをお勧めします。

    $file = $request->file('avatar');

    $name = $file->hashName(); // 一意でランダムな名前を生成...
    $extension = $file->extension(); // ファイルのMIMEタイプに基づいてファイルの拡張子を決定...

<a name="file-visibility"></a>
### ファイルの可視性

LaravelのFlysystem統合では、「可視性」は複数のプラットフォーム間でのファイル権限を抽象化したものです。ファイルは`public`または`private`として宣言できます。ファイルが`public`と宣言されている場合、通常は他のユーザーがファイルにアクセスできることを示します。たとえば、S3ドライバーを使用する場合、`public`ファイルのURLを取得できます。

`put`メソッドを介してファイルを書き込む際に可視性を設定できます。

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

ファイルがすでに保存されている場合、`getVisibility`および`setVisibility`メソッドを通じてその可視性を取得および設定できます。

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

アップロードされたファイルと対話する場合、`storePublicly`および`storePubliclyAs`メソッドを使用して、アップロードされたファイルを`public`可視性で保存できます。

    $path = $request->file('avatar')->storePublicly('avatars', 's3');

    $path = $request->file('avatar')->storePubliclyAs(
        'avatars',
        $request->user()->id,
        's3'
    );

<a name="local-files-and-visibility"></a>
#### ローカルファイルと可視性

`local`ドライバーを使用する場合、`public` [可視性](#file-visibility) はディレクトリに対して`0755`の権限、ファイルに対して`0644`の権限に変換されます。アプリケーションの`filesystems`設定ファイルで権限マッピングを変更できます。

    'local' => [
        'driver' => 'local',
        'root' => storage_path('app'),
        'permissions' => [
            'file' => [
                'public' => 0644,
                'private' => 0600,
            ],
            'dir' => [
                'public' => 0755,
                'private' => 0700,
            ],
        ],
        'throw' => false,
    ],

<a name="deleting-files"></a>
## ファイルの削除

`delete`メソッドは、削除する単一のファイル名またはファイルの配列を受け取ります。

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

必要に応じて、ファイルを削除するディスクを指定できます。

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('path/file.jpg');

<a name="directories"></a>
## ディレクトリ

<a name="get-all-files-within-a-directory"></a>
#### ディレクトリ内のすべてのファイルを取得

`files`メソッドは、指定されたディレクトリ内のすべてのファイルの配列を返します。指定されたディレクトリ内のすべてのファイルとサブディレクトリ内のファイルのリストを取得したい場合は、`allFiles`メソッドを使用できます。

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

<a name="get-all-directories-within-a-directory"></a>
#### ディレクトリ内のすべてのディレクトリを取得

`directories`メソッドは、指定されたディレクトリ内のすべてのディレクトリの配列を返します。さらに、`allDirectories`メソッドを使用して、指定されたディレクトリとそのすべてのサブディレクトリ内のすべてのディレクトリのリストを取得できます。

    $directories = Storage::directories($directory);

    $directories = Storage::allDirectories($directory);

<a name="create-a-directory"></a>
#### ディレクトリの作成

`makeDirectory`メソッドは、指定されたディレクトリを作成し、必要なサブディレクトリを含みます。

    Storage::makeDirectory($directory);

<a name="delete-a-directory"></a>
#### ディレクトリの削除

最後に、`deleteDirectory`メソッドを使用して、ディレクトリとそのすべてのファイルを削除できます。

    Storage::deleteDirectory($directory);

<a name="testing"></a>
## テスト

`Storage`ファサードの`fake`メソッドを使用すると、簡単に偽のディスクを生成できます。これは、`Illuminate\Http\UploadedFile`クラスのファイル生成ユーティリティと組み合わせることで、ファイルアップロードのテストが大幅に簡素化されます。例えば：

===  "Pest"
```php
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('アルバムをアップロードできる', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
        UploadedFile::fake()->image('photo2.jpg')
    ]);

    // アップロードされたファイルのアサーション...
});
```

===  "PHPUnit"
```php
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_albums_can_be_uploaded(): void
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // 1つ以上のファイルが保存されたことをアサート...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // 1つ以上のファイルが保存されていないことをアサート...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

        // 指定されたディレクトリが空であることをアサート...
        Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
    }
}
```

デフォルトでは、`fake`メソッドは一時ディレクトリ内のすべてのファイルを削除します。これらのファイルを保持したい場合は、代わりに"persistentFake"メソッドを使用できます。ファイルアップロードのテストに関する詳細情報は、[HTTPテストドキュメントのファイルアップロードに関する情報](http-tests.md#testing-file-uploads)を参照してください。

> WARNING:  
> `image`メソッドは[GD拡張機能](https://www.php.net/manual/en/book.image.php)を必要とします。

<a name="custom-filesystems"></a>
## カスタムファイルシステム

LaravelのFlysystem統合は、いくつかの「ドライバ」を標準でサポートしています。しかし、Flysystemはこれらに限定されず、他の多くのストレージシステム用のアダプタを持っています。Laravelアプリケーションでこれらの追加アダプタのいずれかを使用したい場合は、カスタムドライバを作成できます。

カスタムファイルシステムを定義するには、Flysystemアダプタが必要です。コミュニティが管理するDropboxアダプタをプロジェクトに追加しましょう：

```shell
composer require spatie/flysystem-dropbox
```

次に、アプリケーションの[サービスプロバイダ](providers.md)の`boot`メソッド内でドライバを登録できます。これを行うには、`Storage`ファサードの`extend`メソッドを使用する必要があります：

    <?php

    namespace App\Providers;

    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Filesystem\FilesystemAdapter;
    use Illuminate\Support\Facades\Storage;
    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 任意のアプリケーションサービスを登録します。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 任意のアプリケーションサービスを起動します。
         */
        public function boot(): void
        {
            Storage::extend('dropbox', function (Application $app, array $config) {
                $adapter = new DropboxAdapter(new DropboxClient(
                    $config['authorization_token']
                ));

                return new FilesystemAdapter(
                    new Filesystem($adapter, $config),
                    $adapter,
                    $config
                );
            });
        }
    }

`extend`メソッドの最初の引数はドライバの名前で、2番目の引数は`$app`と`$config`変数を受け取るクロージャです。クロージャは`Illuminate\Filesystem\FilesystemAdapter`のインスタンスを返す必要があります。`$config`変数には、指定されたディスクの`config/filesystems.php`で定義された値が含まれます。

拡張機能のサービスプロバイダを作成して登録したら、`config/filesystems.php`設定ファイルで`dropbox`ドライバを使用できるようになります。
