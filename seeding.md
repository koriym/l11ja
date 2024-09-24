# データベース: シーディング

- [はじめに](#introduction)
- [シーダーの作成](#writing-seeders)
    - [モデルファクトリの使用](#using-model-factories)
    - [追加のシーダーの呼び出し](#calling-additional-seeders)
    - [モデルイベントの無効化](#muting-model-events)
- [シーダーの実行](#running-seeders)

<a name="introduction"></a>
## はじめに

Laravelでは、シードクラスを使用してデータベースにデータをシードする機能が含まれています。すべてのシードクラスは、`database/seeders`ディレクトリに保存されます。デフォルトでは、`DatabaseSeeder`クラスが定義されています。このクラスから、`call`メソッドを使用して他のシードクラスを実行でき、シーディングの順序を制御できます。

> NOTE:  
> [マスアサインメント保護](eloquent.md#mass-assignment)は、データベースシーディング中に自動的に無効になります。

<a name="writing-seeders"></a>
## シーダーの作成

シーダーを生成するには、`make:seeder` [Artisanコマンド](artisan.md)を実行します。フレームワークによって生成されたすべてのシーダーは、`database/seeders`ディレクトリに配置されます。

```shell
php artisan make:seeder UserSeeder
```

シーダークラスには、デフォルトで`run`メソッドが1つだけ含まれています。このメソッドは、`db:seed` [Artisanコマンド](artisan.md)が実行されるときに呼び出されます。`run`メソッド内で、データベースにデータを挿入する方法を自由に選択できます。[クエリビルダ](queries.md)を使用して手動でデータを挿入するか、[Eloquentモデルファクトリ](eloquent-factories.md)を使用することができます。

例として、デフォルトの`DatabaseSeeder`クラスを修正し、`run`メソッドにデータベース挿入文を追加しましょう。

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * データベースシーダーを実行します。
         */
        public function run(): void
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@example.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> NOTE:  
> `run`メソッドのシグネチャ内で必要な依存関係をタイプヒントで指定できます。これらは、Laravelの[サービスコンテナ](container.md)を介して自動的に解決されます。

<a name="using-model-factories"></a>
### モデルファクトリの使用

もちろん、各モデルのシードの属性を手動で指定するのは面倒です。代わりに、[モデルファクトリ](eloquent-factories.md)を使用して、大量のデータベースレコードを簡単に生成できます。まず、[モデルファクトリのドキュメント](eloquent-factories.md)を参照して、ファクトリを定義する方法を学んでください。

例えば、それぞれが1つの関連する投稿を持つ50人のユーザーを作成しましょう。

    use App\Models\User;

    /**
     * データベースシーダーを実行します。
     */
    public function run(): void
    {
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }

<a name="calling-additional-seeders"></a>
### 追加のシーダーの呼び出し

`DatabaseSeeder`クラス内で、`call`メソッドを使用して追加のシードクラスを実行できます。`call`メソッドを使用すると、データベースシーディングを複数のファイルに分割でき、単一のシーダークラスが大きくなりすぎないように制御できます。`call`メソッドは、実行する必要があるシーダークラスの配列を受け取ります。

    /**
     * データベースシーダーを実行します。
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="muting-model-events"></a>
### モデルイベントの無効化

シードを実行している間、モデルがイベントをディスパッチしないようにすることができます。これは、`WithoutModelEvents`トレイトを使用して実現できます。`WithoutModelEvents`トレイトを使用すると、モデルイベントがディスパッチされなくなります。たとえ`call`メソッドを介して追加のシードクラスが実行された場合でも、イベントはディスパッチされません。

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Console\Seeds\WithoutModelEvents;

    class DatabaseSeeder extends Seeder
    {
        use WithoutModelEvents;

        /**
         * データベースシーダーを実行します。
         */
        public function run(): void
        {
            $this->call([
                UserSeeder::class,
            ]);
        }
    }

<a name="running-seeders"></a>
## シーダーの実行

データベースにシードを実行するには、`db:seed` Artisanコマンドを実行します。デフォルトでは、`db:seed`コマンドは`Database\Seeders\DatabaseSeeder`クラスを実行しますが、`--class`オプションを使用して、個別に実行する特定のシーダークラスを指定できます。

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

また、`migrate:fresh`コマンドと`--seed`オプションを組み合わせてデータベースにシードを実行することもできます。このコマンドは、すべてのテーブルをドロップし、すべてのマイグレーションを再実行します。このコマンドは、データベースを完全に再構築するのに便利です。`--seeder`オプションを使用して、実行する特定のシーダーを指定できます。

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

<a name="forcing-seeding-production"></a>
#### 本番環境でのシーダーの強制実行

一部のシーディング操作は、データを変更または失う可能性があります。本番データベースに対してシーディングコマンドを実行することを保護するために、`production`環境でシーダーが実行される前に確認を求められます。プロンプトなしでシーダーを強制的に実行するには、`--force`フラグを使用します。

```shell
php artisan db:seed --force
```
