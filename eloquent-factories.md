# Eloquent: ファクトリ

- [はじめに](#introduction)
- [モデルファクトリの定義](#defining-model-factories)
    - [ファクトリの生成](#generating-factories)
    - [ファクトリの状態](#factory-states)
    - [ファクトリのコールバック](#factory-callbacks)
- [ファクトリを使用したモデルの作成](#creating-models-using-factories)
    - [モデルのインスタンス化](#instantiating-models)
    - [モデルの永続化](#persisting-models)
    - [シーケンス](#sequences)
- [ファクトリのリレーション](#factory-relationships)
    - [Has Many リレーション](#has-many-relationships)
    - [Belongs To リレーション](#belongs-to-relationships)
    - [多対多のリレーション](#many-to-many-relationships)
    - [ポリモーフィックリレーション](#polymorphic-relationships)
    - [ファクトリ内でのリレーションの定義](#defining-relationships-within-factories)
    - [リレーションに既存のモデルを再利用する](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## はじめに

アプリケーションのテストやデータベースのシーディングを行う際に、データベースにいくつかのレコードを挿入する必要があるかもしれません。各カラムの値を手動で指定する代わりに、Laravelでは[Eloquentモデル](eloquent.md)ごとにデフォルトの属性セットを定義することができます。

ファクトリの書き方の例を見るには、アプリケーションの `database/factories/UserFactory.php` ファイルを見てください。このファクトリはすべての新しいLaravelアプリケーションに含まれており、以下のファクトリ定義が含まれています。

    namespace Database\Factories;

    use Illuminate\Database\Eloquent\Factories\Factory;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    /**
     * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
     */
    class UserFactory extends Factory
    {
        /**
         * The current password being used by the factory.
         */
        protected static ?string $password;

        /**
         * Define the model's default state.
         *
         * @return array<string, mixed>
         */
        public function definition(): array
        {
            return [
                'name' => fake()->name(),
                'email' => fake()->unique()->safeEmail(),
                'email_verified_at' => now(),
                'password' => static::$password ??= Hash::make('password'),
                'remember_token' => Str::random(10),
            ];
        }

        /**
         * Indicate that the model's email address should be unverified.
         */
        public function unverified(): static
        {
            return $this->state(fn (array $attributes) => [
                'email_verified_at' => null,
            ]);
        }
    }

最も基本的な形では、ファクトリはLaravelの基本ファクトリクラスを拡張し、`definition` メソッドを定義するクラスです。`definition` メソッドは、ファクトリを使用してモデルを作成する際に適用されるデフォルトの属性値のセットを返します。

`fake` ヘルパーを介して、ファクトリは [Faker](https://github.com/FakerPHP/Faker) PHP ライブラリにアクセスできます。これにより、テストやシーディングのためにさまざまな種類のランダムデータを簡単に生成できます。

> NOTE:  
> アプリケーションのFakerロケールは、`config/app.php` 設定ファイルの `faker_locale` オプションを更新することで変更できます。

<a name="defining-model-factories"></a>
## モデルファクトリの定義

<a name="generating-factories"></a>
### ファクトリの生成

ファクトリを作成するには、`make:factory` [Artisanコマンド](artisan.md)を実行します。

```shell
php artisan make:factory PostFactory
```

新しいファクトリクラスは、`database/factories` ディレクトリに配置されます。

<a name="factory-and-model-discovery-conventions"></a>
#### モデルとファクトリの検出規約

ファクトリを定義したら、`Illuminate\Database\Eloquent\Factories\HasFactory` トレイトによってモデルに提供される静的 `factory` メソッドを使用して、そのモデルのファクトリインスタンスをインスタンス化できます。

`HasFactory` トレイトの `factory` メソッドは、規約を使用して適切なファクトリを決定します。具体的には、メソッドは `Database\Factories` 名前空間内で、モデル名に一致し、`Factory` で終わるクラス名を持つファクトリを探します。これらの規約が特定のアプリケーションやファクトリに適用されない場合は、モデルの `newFactory` メソッドをオーバーライドして、モデルの対応するファクトリのインスタンスを直接返すことができます。

    use Illuminate\Database\Eloquent\Factories\Factory;
    use Database\Factories\Administration\FlightFactory;

    /**
     * Create a new factory instance for the model.
     */
    protected static function newFactory(): Factory
    {
        return FlightFactory::new();
    }

次に、対応するファクトリに `model` プロパティを定義します。

    use App\Administration\Flight;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class FlightFactory extends Factory
    {
        /**
         * The name of the factory's corresponding model.
         *
         * @var class-string<\Illuminate\Database\Eloquent\Model>
         */
        protected $model = Flight::class;
    }

<a name="factory-states"></a>
### ファクトリの状態

状態操作メソッドを使用すると、モデルファクトリに適用できる離散的な変更を定義できます。たとえば、`Database\Factories\UserFactory` ファクトリには、デフォルトの属性値の1つを変更する `suspended` 状態メソッドが含まれているかもしれません。

状態変換メソッドは通常、Laravelの基本ファクトリクラスによって提供される `state` メソッドを呼び出します。`state` メソッドは、ファクトリに定義された生の属性の配列を受け取り、変更する属性の配列を返すクロージャを受け取ります。

    use Illuminate\Database\Eloquent\Factories\Factory;

    /**
     * Indicate that the user is suspended.
     */
    public function suspended(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        });
    }

<a name="trashed-state"></a>
#### "Trashed" 状態

Eloquentモデルが[ソフトデリート](eloquent.md#soft-deleting)可能な場合、組み込みの `trashed` 状態メソッドを呼び出して、作成されたモデルが既に「ソフトデリート」されていることを示すことができます。`trashed` 状態を手動で定義する必要はありません。すべてのファクトリで自動的に利用可能です。

    use App\Models\User;

    $user = User::factory()->trashed()->create();

<a name="factory-callbacks"></a>
### ファクトリのコールバック

ファクトリコールバックは、`afterMaking` および `afterCreating` メソッドを使用して登録され、モデルの作成または作成後に追加のタスクを実行できるようにします。これらのコールバックは、ファクトリクラスに `configure` メソッドを定義することで登録する必要があります。このメソッドは、ファクトリがインスタンス化されるときにLaravelによって自動的に呼び出されます。

    namespace Database\Factories;

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;

    class UserFactory extends Factory
    {
        /**
         * Configure the model factory.
         */
        public function configure(): static
        {
            return $this->afterMaking(function (User $user) {
                // ...
            })->afterCreating(function (User $user) {
                // ...
            });
        }

        // ...
    }

特定の状態に固有の追加タスクを実行するために、状態メソッド内にファクトリコールバックを登録することもできます。

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Factory;

    /**
     * Indicate that the user is suspended.
     */
    public function suspended(): Factory
    {
        return $this->state(function (array $attributes) {
            return [
                'account_status' => 'suspended',
            ];
        })->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

<a name="creating-models-using-factories"></a>
## ファクトリを使用したモデルの作成

<a name="instantiating-models"></a>
### モデルのインスタンス化

ファクトリを定義したら、`Illuminate\Database\Eloquent\Factories\HasFactory` トレイトによってモデルに提供される静的 `factory` メソッドを使用して、そのモデルのファクトリインスタンスをインスタンス化できます。モデルの作成例をいくつか見てみましょう。まず、データベースに永続化せずにモデルを作成するために `make` メソッドを使用します。

    use App\Models\User;

    $user = User::factory()->make();

`count` メソッドを使用して、多くのモデルのコレクションを作成できます。

    $users = User::factory()->count(3)->make();

<a name="applying-states"></a>
#### 状態の適用

モデルに[状態](#factory-states)を適用することもできます。複数の状態変換をモデルに適用したい場合は、状態変換メソッドを直接呼び出すだけです。

    $users = User::factory()->count(5)->suspended()->make();

<a name="overriding-attributes"></a>
#### 属性のオーバーライド

モデルのデフォルト値の一部をオーバーライドしたい場合は、`make` メソッドに値の配列を渡すことができます。指定された属性のみが置き換えられ、残りの属性はファクトリで指定されたデフォルト値のままになります。

    $user = User::factory()->make([
        'name' => 'Abigail Otwell',
    ]);

あるいは、`state` メソッドをファクトリインスタンス上で直接呼び出して、インライン状態変換を実行することもできます。

    $user = User::factory()->state([
        'name' => 'Abigail Otwell',
    ])->make();

> NOTE:  
> [マスアサインメント保護](eloquent.md#mass-assignment)は、ファクトリを使用してモデルを作成する際に自動的に無効になります。

`create`メソッドは、モデルインスタンスをインスタンス化し、Eloquentの`save`メソッドを使用してデータベースに永続化します。

    use App\Models\User;

    // App\Models\Userインスタンスを1つ作成...
    $user = User::factory()->create();

    // App\Models\Userインスタンスを3つ作成...
    $users = User::factory()->count(3)->create();

`create`メソッドに属性の配列を渡すことで、ファクトリのデフォルトのモデル属性を上書きできます。

    $user = User::factory()->create([
        'name' => 'Abigail',
    ]);

<a name="sequences"></a>
### シーケンス

場合によっては、作成される各モデルの特定の属性の値を交互に変更したいことがあります。これは、状態変換をシーケンスとして定義することで実現できます。たとえば、作成される各ユーザーの`admin`カラムの値を`Y`と`N`で交互に変更したい場合があります。

    use App\Models\User;
    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        ['admin' => 'Y'],
                        ['admin' => 'N'],
                    ))
                    ->create();

この例では、5人のユーザーが`admin`値`Y`で作成され、5人のユーザーが`admin`値`N`で作成されます。

必要に応じて、クロージャをシーケンス値として含めることができます。クロージャは、シーケンスが新しい値を必要とするたびに呼び出されます。

    use Illuminate\Database\Eloquent\Factories\Sequence;

    $users = User::factory()
                    ->count(10)
                    ->state(new Sequence(
                        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
                    ))
                    ->create();

シーケンスクロージャ内では、クロージャに注入されるシーケンスインスタンスの`$index`または`$count`プロパティにアクセスできます。`$index`プロパティには、これまでにシーケンスを通過した回数が含まれ、`$count`プロパティにはシーケンスが呼び出される合計回数が含まれます。

    $users = User::factory()
                    ->count(10)
                    ->sequence(fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index])
                    ->create();

便宜上、シーケンスは`sequence`メソッドを使用しても適用できます。これは内部的に`state`メソッドを呼び出します。`sequence`メソッドは、クロージャまたはシーケンス属性の配列を受け取ります。

    $users = User::factory()
                    ->count(2)
                    ->sequence(
                        ['name' => 'First User'],
                        ['name' => 'Second User'],
                    )
                    ->create();

<a name="factory-relationships"></a>
## ファクトリのリレーションシップ

<a name="has-many-relationships"></a>
### Has Many リレーションシップ

次に、Laravelの流暢なファクトリメソッドを使用してEloquentモデルのリレーションシップを構築する方法を探ってみましょう。まず、アプリケーションに`App\Models\User`モデルと`App\Models\Post`モデルがあるとします。また、`User`モデルが`Post`モデルとの`hasMany`リレーションシップを定義しているとします。Laravelのファクトリが提供する`has`メソッドを使用して、3つの投稿を持つユーザーを作成できます。`has`メソッドはファクトリインスタンスを受け取ります。

    use App\Models\Post;
    use App\Models\User;

    $user = User::factory()
                ->has(Post::factory()->count(3))
                ->create();

規約により、`Post`モデルを`has`メソッドに渡すと、Laravelは`User`モデルがリレーションシップを定義する`posts`メソッドを持つ必要があると想定します。必要に応じて、操作したいリレーションシップの名前を明示的に指定できます。

    $user = User::factory()
                ->has(Post::factory()->count(3), 'posts')
                ->create();

もちろん、関連モデルに対して状態操作を実行できます。さらに、状態変更が親モデルにアクセスする必要がある場合、クロージャベースの状態変換を渡すことができます。

    $user = User::factory()
                ->has(
                    Post::factory()
                            ->count(3)
                            ->state(function (array $attributes, User $user) {
                                return ['user_type' => $user->type];
                            })
                )
                ->create();

<a name="has-many-relationships-using-magic-methods"></a>
#### マジックメソッドの使用

便宜上、Laravelのマジックファクトリリレーションシップメソッドを使用してリレーションシップを構築できます。たとえば、次の例では規約を使用して、関連モデルが`User`モデルの`posts`リレーションシップメソッドを介して作成されるべきであると判断します。

    $user = User::factory()
                ->hasPosts(3)
                ->create();

マジックメソッドを使用してファクトリリレーションシップを作成する場合、関連モデルに上書きする属性の配列を渡すことができます。

    $user = User::factory()
                ->hasPosts(3, [
                    'published' => false,
                ])
                ->create();

状態変更が親モデルにアクセスする必要がある場合、クロージャベースの状態変換を提供できます。

    $user = User::factory()
                ->hasPosts(3, function (array $attributes, User $user) {
                    return ['user_type' => $user->type];
                })
                ->create();

<a name="belongs-to-relationships"></a>
### Belongs To リレーションシップ

ファクトリを使用して「has many」リレーションシップを構築する方法を探ったので、次にリレーションシップの逆を探ってみましょう。`for`メソッドを使用して、ファクトリが作成したモデルが属する親モデルを定義できます。たとえば、3つの`App\Models\Post`モデルインスタンスを1人のユーザーに属するように作成できます。

    use App\Models\Post;
    use App\Models\User;

    $posts = Post::factory()
                ->count(3)
                ->for(User::factory()->state([
                    'name' => 'Jessica Archer',
                ]))
                ->create();

既に作成された親モデルインスタンスを、作成中のモデルに関連付ける必要がある場合、そのモデルインスタンスを`for`メソッドに渡すことができます。

    $user = User::factory()->create();

    $posts = Post::factory()
                ->count(3)
                ->for($user)
                ->create();

<a name="belongs-to-relationships-using-magic-methods"></a>
#### マジックメソッドの使用

便宜上、Laravelのマジックファクトリリレーションシップメソッドを使用して「belongs to」リレーションシップを定義できます。たとえば、次の例では規約を使用して、3つの投稿が`Post`モデルの`user`リレーションシップに属するべきであると判断します。

    $posts = Post::factory()
                ->count(3)
                ->forUser([
                    'name' => 'Jessica Archer',
                ])
                ->create();

<a name="many-to-many-relationships"></a>
### Many to Many リレーションシップ

[has many リレーションシップ](#has-many-relationships)と同様に、「many to many」リレーションシップは`has`メソッドを使用して作成できます。

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->has(Role::factory()->count(3))
                ->create();

<a name="pivot-table-attributes"></a>
#### ピボットテーブルの属性

モデルをリンクするピボット/中間テーブルに設定する属性を定義する必要がある場合、`hasAttached`メソッドを使用できます。このメソッドは、ピボットテーブルの属性名と値の配列を2番目の引数として受け取ります。

    use App\Models\Role;
    use App\Models\User;

    $user = User::factory()
                ->hasAttached(
                    Role::factory()->count(3),
                    ['active' => true]
                )
                ->create();

状態変更が関連モデルにアクセスする必要がある場合、クロージャベースの状態変換を提供できます。

    $user = User::factory()
                ->hasAttached(
                    Role::factory()
                        ->count(3)
                        ->state(function (array $attributes, User $user) {
                            return ['name' => $user->name.' Role'];
                        }),
                    ['active' => true]
                )
                ->create();

作成中のモデルにアタッチしたいモデルインスタンスが既にある場合、それらのモデルインスタンスを`hasAttached`メソッドに渡すことができます。この例では、同じ3つのロールがすべての3人のユーザーにアタッチされます。

    $roles = Role::factory()->count(3)->create();

    $user = User::factory()
                ->count(3)
                ->hasAttached($roles, ['active' => true])
                ->create();

<a name="many-to-many-relationships-using-magic-methods"></a>
#### マジックメソッドの使用

便宜上、Laravelのマジックファクトリリレーションシップメソッドを使用して多対多のリレーションシップを定義できます。たとえば、次の例では規約を使用して、関連モデルが`User`モデルの`roles`リレーションシップメソッドを介して作成されるべきであると判断します。

    $user = User::factory()
                ->hasRoles(1, [
                    'name' => 'Editor'
                ])
                ->create();

<a name="polymorphic-relationships"></a>
### ポリモーフィックリレーションシップ

[ポリモーフィックリレーションシップ](eloquent-relationships.md#polymorphic-relationships)もファクトリを使用して作成できます。ポリモーフィックな「morph many」リレーションシップは、典型的な「has many」リレーションシップと同じ方法で作成されます。たとえば、`App\Models\Post`モデルが`App\Models\Comment`モデルとの`morphMany`リレーションシップを持つ場合：

    use App\Models\Post;

    $post = Post::factory()->hasComments(3)->create();

<a name="morph-to-relationships"></a>
#### Morph To リレーションシップ

マジックメソッドを使って`morphTo`リレーションを作成することはできません。代わりに、`for`メソッドを直接使用し、リレーションの名前を明示的に指定する必要があります。例えば、`Comment`モデルが`morphTo`リレーションを定義する`commentable`メソッドを持っているとします。この場合、`for`メソッドを直接使用して、1つの投稿に属する3つのコメントを作成できます:

    $comments = Comment::factory()->count(3)->for(
        Post::factory(), 'commentable'
    )->create();

<a name="polymorphic-many-to-many-relationships"></a>
#### ポリモーフィックな多対多リレーション

ポリモーフィックな「多対多」（`morphToMany` / `morphedByMany`）リレーションは、非ポリモーフィックな「多対多」リレーションと同じように作成できます:

    use App\Models\Tag;
    use App\Models\Video;

    $videos = Video::factory()
                ->hasAttached(
                    Tag::factory()->count(3),
                    ['public' => true]
                )
                ->create();

もちろん、マジックの`has`メソッドを使用してポリモーフィックな「多対多」リレーションを作成することもできます:

    $videos = Video::factory()
                ->hasTags(3, ['public' => true])
                ->create();

<a name="defining-relationships-within-factories"></a>
### ファクトリ内でのリレーションの定義

モデルファクトリ内でリレーションを定義するには、通常、リレーションの外部キーに新しいファクトリインスタンスを割り当てます。これは通常、`belongsTo`や`morphTo`リレーションのような「逆」リレーションに対して行われます。例えば、投稿を作成する際に新しいユーザーを作成したい場合、以下のようにできます:

    use App\Models\User;

    /**
     * モデルのデフォルト状態を定義する。
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

リレーションのカラムがそれを定義するファクトリに依存する場合、属性にクロージャを割り当てることができます。クロージャはファクトリの評価済み属性配列を受け取ります:

    /**
     * モデルのデフォルト状態を定義する。
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'user_id' => User::factory(),
            'user_type' => function (array $attributes) {
                return User::find($attributes['user_id'])->type;
            },
            'title' => fake()->title(),
            'content' => fake()->paragraph(),
        ];
    }

<a name="recycling-an-existing-model-for-relationships"></a>
### リレーションに既存のモデルを再利用する

他のモデルと共通のリレーションを持つモデルがある場合、ファクトリによって作成されたすべてのリレーションに対して、関連モデルの単一インスタンスを再利用するために`recycle`メソッドを使用できます。

例えば、`Airline`、`Flight`、`Ticket`モデルがあり、チケットは航空会社とフライトに属し、フライトも航空会社に属しているとします。チケットを作成する際、チケットとフライトの両方に同じ航空会社を使用したい場合、航空会社のインスタンスを`recycle`メソッドに渡すことができます:

    Ticket::factory()
        ->recycle(Airline::factory()->create())
        ->create();

共通のユーザーやチームに属するモデルがある場合、`recycle`メソッドは特に便利です。

`recycle`メソッドは既存のモデルのコレクションも受け取ります。コレクションが`recycle`メソッドに提供されると、ファクトリがそのタイプのモデルを必要とするときに、コレクションからランダムなモデルが選択されます:

    Ticket::factory()
        ->recycle($airlines)
        ->create();

