# Eloquent: シリアライズ

- [イントロダクション](#introduction)
- [モデルとコレクションのシリアライズ](#serializing-models-and-collections)
    - [配列へのシリアライズ](#serializing-to-arrays)
    - [JSONへのシリアライズ](#serializing-to-json)
- [JSONからの属性の隠蔽](#hiding-attributes-from-json)
- [JSONへの値の追加](#appending-values-to-json)
- [日付のシリアライズ](#date-serialization)

<a name="introduction"></a>
## イントロダクション

Laravelを使用してAPIを構築する際、モデルとそのリレーションを配列やJSONに変換する必要があることがよくあります。Eloquentには、これらの変換を行うための便利なメソッドが含まれており、モデルのシリアライズされた表現にどの属性を含めるかを制御できます。

> NOTE:  
> EloquentモデルとコレクションのJSONシリアライズをより堅牢に処理する方法については、[Eloquent APIリソース](eloquent-resources.md)のドキュメントを確認してください。

<a name="serializing-models-and-collections"></a>
## モデルとコレクションのシリアライズ

<a name="serializing-to-arrays"></a>
### 配列へのシリアライズ

モデルとそのロードされた[リレーション](eloquent-relationships.md)を配列に変換するには、`toArray`メソッドを使用する必要があります。このメソッドは再帰的であるため、すべての属性とすべてのリレーション（リレーションのリレーションを含む）が配列に変換されます。

    use App\Models\User;

    $user = User::with('roles')->first();

    return $user->toArray();

`attributesToArray`メソッドは、モデルの属性を配列に変換するために使用できますが、そのリレーションは含まれません。

    $user = User::first();

    return $user->attributesToArray();

また、コレクションインスタンスの`toArray`メソッドを呼び出すことで、モデルの[コレクション](eloquent-collections.md)全体を配列に変換することもできます。

    $users = User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### JSONへのシリアライズ

モデルをJSONに変換するには、`toJson`メソッドを使用する必要があります。`toArray`と同様に、`toJson`メソッドは再帰的であるため、すべての属性とリレーションがJSONに変換されます。PHPが[サポートする](https://secure.php.net/manual/en/function.json-encode.php)任意のJSONエンコーディングオプションを指定することもできます。

    use App\Models\User;

    $user = User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

あるいは、モデルまたはコレクションを文字列にキャストすることもできます。これにより、モデルまたはコレクションの`toJson`メソッドが自動的に呼び出されます。

    return (string) User::find(1);

モデルとコレクションは文字列にキャストされるとJSONに変換されるため、アプリケーションのルートまたはコントローラからEloquentオブジェクトを直接返すことができます。Laravelは、ルートまたはコントローラから返されるときに、Eloquentモデルとコレクションを自動的にJSONにシリアライズします。

    Route::get('/users', function () {
        return User::all();
    });

<a name="relationships"></a>
#### リレーション

EloquentモデルがJSONに変換されるとき、そのロードされたリレーションは自動的にJSONオブジェクトの属性として含まれます。また、Eloquentリレーションメソッドは「キャメルケース」のメソッド名を使用して定義されますが、リレーションのJSON属性は「スネークケース」になります。

<a name="hiding-attributes-from-json"></a>
## JSONからの属性の隠蔽

パスワードなどの属性がモデルの配列またはJSON表現に含まれないように制限したい場合があります。そのためには、モデルに`$hidden`プロパティを追加します。`$hidden`プロパティの配列にリストされた属性は、モデルのシリアライズされた表現に含まれません。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 配列に対して隠蔽するべき属性。
         *
         * @var array
         */
        protected $hidden = ['password'];
    }

> NOTE:  
> リレーションを隠蔽するには、Eloquentモデルの`$hidden`プロパティにリレーションのメソッド名を追加します。

あるいは、`visible`プロパティを使用して、モデルの配列とJSON表現に含めるべき属性の「許可リスト」を定義することもできます。`$visible`配列に存在しないすべての属性は、モデルが配列またはJSONに変換されるときに隠蔽されます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 配列に対して表示するべき属性。
         *
         * @var array
         */
        protected $visible = ['first_name', 'last_name'];
    }

<a name="temporarily-modifying-attribute-visibility"></a>
#### 一時的な属性の可視性の変更

特定のモデルインスタンスで通常は隠蔽されている属性を表示したい場合は、`makeVisible`メソッドを使用できます。`makeVisible`メソッドはモデルインスタンスを返します。

    return $user->makeVisible('attribute')->toArray();

同様に、通常は表示されている属性を隠蔽したい場合は、`makeHidden`メソッドを使用できます。

    return $user->makeHidden('attribute')->toArray();

すべての表示または隠蔽された属性を一時的に上書きしたい場合は、それぞれ`setVisible`メソッドと`setHidden`メソッドを使用できます。

    return $user->setVisible(['id', 'name'])->toArray();

    return $user->setHidden(['email', 'password', 'remember_token'])->toArray();

<a name="appending-values-to-json"></a>
## JSONへの値の追加

モデルを配列またはJSONに変換する際に、データベースに対応する列がない属性を追加したい場合があります。そのためには、まずその値の[アクセサ](eloquent-mutators.md)を定義します。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Casts\Attribute;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * ユーザーが管理者であるかどうかを判定します。
         */
        protected function isAdmin(): Attribute
        {
            return new Attribute(
                get: fn () => 'yes',
            );
        }
    }

アクセサを常にモデルの配列とJSON表現に追加したい場合は、モデルの`appends`プロパティに属性名を追加できます。属性名は通常、「スネークケース」のシリアライズされた表現を使用して参照されますが、アクセサのPHPメソッドは「キャメルケース」を使用して定義されます。

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * モデルの配列表現に追加するアクセサ。
         *
         * @var array
         */
        protected $appends = ['is_admin'];
    }

属性が`appends`リストに追加されると、モデルの配列とJSON表現の両方に含まれるようになります。`appends`配列内の属性は、モデルで設定された`visible`と`hidden`の設定も尊重します。

<a name="appending-at-run-time"></a>
#### 実行時に追加

実行時にモデルインスタンスに追加の属性を追加するように指示するには、`append`メソッドを使用します。または、`setAppends`メソッドを使用して、特定のモデルインスタンスの追加プロパティの配列全体を上書きすることもできます。

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## 日付のシリアライズ

<a name="customizing-the-default-date-format"></a>
#### デフォルトの日付フォーマットのカスタマイズ

`serializeDate`メソッドをオーバーライドすることで、デフォルトのシリアライズフォーマットをカスタマイズできます。このメソッドは、データベースに日付が格納される方法には影響しません。

    /**
     * 配列 / JSONシリアライズのために日付を準備します。
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d');
    }

<a name="customizing-the-date-format-per-attribute"></a>
#### 属性ごとの日付フォーマットのカスタマイズ

個々のEloquent日付属性のシリアライズフォーマットをカスタマイズするには、モデルの[キャスト宣言](eloquent-mutators.md#attribute-casting)で日付フォーマットを指定します。

    protected function casts(): array
    {
        return [
            'birthday' => 'date:Y-m-d',
            'joined_at' => 'datetime:Y-m-d H:00',
        ];
    }

