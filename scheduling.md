# タスクスケジューリング

- [はじめに](#introduction)
- [スケジュールの定義](#defining-schedules)
    - [Artisanコマンドのスケジューリング](#scheduling-artisan-commands)
    - [キューされたジョブのスケジューリング](#scheduling-queued-jobs)
    - [シェルコマンドのスケジューリング](#scheduling-shell-commands)
    - [スケジュール頻度オプション](#schedule-frequency-options)
    - [タイムゾーン](#timezones)
    - [タスクの重複防止](#preventing-task-overlaps)
    - [１台のサーバーでタスクを実行](#running-tasks-on-one-server)
    - [バックグラウンドタスク](#background-tasks)
    - [メンテナンスモード](#maintenance-mode)
- [スケジューラの実行](#running-the-scheduler)
    - [サブミニットスケジュールタスク](#sub-minute-scheduled-tasks)
    - [ローカルでスケジューラを実行](#running-the-scheduler-locally)
- [タスク出力](#task-output)
- [タスクフック](#task-hooks)
- [イベント](#events)

<a name="introduction"></a>
## はじめに

過去には、サーバー上でスケジュールする必要がある各タスクに対してcron設定エントリを書いていたかもしれません。しかし、これはすぐに面倒になる可能性があります。タスクスケジュールがソース管理下になく、既存のcronエントリを表示したり、追加のエントリを追加したりするためにSSHでサーバーに接続する必要があるためです。

Laravelのコマンドスケジューラは、サーバー上のスケジュールされたタスクを管理するための新しいアプローチを提供します。スケジューラを使用すると、Laravelアプリケーション自体の中で流暢かつ表現力豊かにコマンドスケジュールを定義できます。スケジューラを使用する場合、サーバー上で必要なcronエントリは1つだけです。タスクスケジュールは通常、アプリケーションの`routes/console.php`ファイルで定義されます。

<a name="defining-schedules"></a>
## スケジュールの定義

アプリケーションの`routes/console.php`ファイルで、すべてのスケジュールされたタスクを定義できます。開始するには、例を見てみましょう。この例では、毎日真夜中に呼び出されるクロージャをスケジュールします。クロージャ内で、データベースクエリを実行してテーブルをクリアします。

    <?php

    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Schedule;

    Schedule::call(function () {
        DB::table('recent_users')->delete();
    })->daily();

クロージャを使用してスケジュールするだけでなく、[呼び出し可能なオブジェクト](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)をスケジュールすることもできます。呼び出し可能なオブジェクトは、`__invoke`メソッドを含む単純なPHPクラスです。

    Schedule::call(new DeleteRecentUsers)->daily();

`routes/console.php`ファイルをコマンド定義のみに予約したい場合は、アプリケーションの`bootstrap/app.php`ファイルで`withSchedule`メソッドを使用してスケジュールされたタスクを定義できます。このメソッドは、スケジューラのインスタンスを受け取るクロージャを受け入れます。

    use Illuminate\Console\Scheduling\Schedule;

    ->withSchedule(function (Schedule $schedule) {
        $schedule->call(new DeleteRecentUsers)->daily();
    })

スケジュールされたタスクと次に実行される予定の概要を表示したい場合は、`schedule:list` Artisanコマンドを使用できます。

```bash
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### Artisanコマンドのスケジューリング

クロージャをスケジュールするだけでなく、[Artisanコマンド](artisan.md)やシステムコマンドをスケジュールすることもできます。たとえば、`command`メソッドを使用して、コマンドの名前またはクラスを使用してArtisanコマンドをスケジュールできます。

Artisanコマンドをコマンドのクラス名を使用してスケジュールする場合、コマンドが呼び出されるときに提供されるべき追加のコマンドライン引数の配列を渡すことができます。

    use App\Console\Commands\SendEmailsCommand;
    use Illuminate\Support\Facades\Schedule;

    Schedule::command('emails:send Taylor --force')->daily();

    Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();

<a name="scheduling-artisan-closure-commands"></a>
#### Artisanクロージャコマンドのスケジューリング

クロージャで定義されたArtisanコマンドをスケジュールする場合、コマンドの定義の後にスケジューリング関連のメソッドをチェーンできます。

    Artisan::command('delete:recent-users', function () {
        DB::table('recent_users')->delete();
    })->purpose('Delete recent users')->daily();

クロージャコマンドに引数を渡す必要がある場合は、`schedule`メソッドにそれらを提供できます。

    Artisan::command('emails:send {user} {--force}', function ($user) {
        // ...
    })->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();

<a name="scheduling-queued-jobs"></a>
### キューされたジョブのスケジューリング

`job`メソッドを使用して、[キューされたジョブ](queues.md)をスケジュールできます。このメソッドは、`call`メソッドを使用してクロージャを定義してジョブをキューに入れる代わりに、キューされたジョブをスケジュールする便利な方法を提供します。

    use App\Jobs\Heartbeat;
    use Illuminate\Support\Facades\Schedule;

    Schedule::job(new Heartbeat)->everyFiveMinutes();

オプションの2番目と3番目の引数を`job`メソッドに提供して、ジョブをキューに入れるために使用するキュー名とキュー接続を指定できます。

    use App\Jobs\Heartbeat;
    use Illuminate\Support\Facades\Schedule;

    // ジョブを "heartbeats" キューに "sqs" 接続でディスパッチする...
    Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### シェルコマンドのスケジューリング

`exec`メソッドを使用して、オペレーティングシステムにコマンドを発行できます。

    use Illuminate\Support\Facades\Schedule;

    Schedule::exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### スケジュール頻度オプション

タスクを指定された間隔で実行するように設定する方法の例を既に見てきました。ただし、タスクに割り当てることができるタスクスケジュールの頻度は他にもたくさんあります。

<div class="overflow-auto" markdown=1>

| メソッド                             | 説明                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `->cron('* * * * *');`             | カスタムcronスケジュールでタスクを実行。                  |
| `->everySecond();`                 | 毎秒タスクを実行。                               |
| `->everyTwoSeconds();`             | 2秒ごとにタスクを実行。                          |
| `->everyFiveSeconds();`            | 5秒ごとにタスクを実行。                         |
| `->everyTenSeconds();`             | 10秒ごとにタスクを実行。                          |
| `->everyFifteenSeconds();`         | 15秒ごとにタスクを実行。                      |
| `->everyTwentySeconds();`          | 20秒ごとにタスクを実行。                       |
| `->everyThirtySeconds();`          | 30秒ごとにタスクを実行。                       |
| `->everyMinute();`                 | 毎分タスクを実行。                               |
| `->everyTwoMinutes();`             | 2分ごとにタスクを実行。                          |
| `->everyThreeMinutes();`           | 3分ごとにタスクを実行。                        |
| `->everyFourMinutes();`            | 4分ごとにタスクを実行。                         |
| `->everyFiveMinutes();`            | 5分ごとにタスクを実行。                         |
| `->everyTenMinutes();`             | 10分ごとにタスクを実行。                          |
| `->everyFifteenMinutes();`         | 15分ごとにタスクを実行。                      |
| `->everyThirtyMinutes();`          | 30分ごとにタスクを実行。                       |
| `->hourly();`                      | 毎時タスクを実行。                                 |
| `->hourlyAt(17);`                  | 毎時17分にタスクを実行。     |
| `->everyOddHour($minutes = 0);`    | 奇数時間ごとにタスクを実行。                             |
| `->everyTwoHours($minutes = 0);`   | 2時間ごとにタスクを実行。                            |
| `->everyThreeHours($minutes = 0);` | 3時間ごとにタスクを実行。                          |
| `->everyFourHours($minutes = 0);`  | 4時間ごとにタスクを実行。                           |
| `->everySixHours($minutes = 0);`   | 6時間ごとにタスクを実行。                            |
| `->daily();`                       | 毎日真夜中にタスクを実行。                      |
| `->dailyAt('13:00');`              | 毎日13:00にタスクを実行。                         |
| `->twiceDaily(1, 13);`             | 毎日1:00と13:00にタスクを実行。                      |
| `->twiceDailyAt(1, 13, 15);`       | 毎日1:15と13:15にタスクを実行。                      |
| `->weekly();`                      | 毎週日曜日の00:00にタスクを実行。                      |
| `->weeklyOn(1, '8:00');`           | 毎週月曜日の8:00にタスクを実行。               |
| `->monthly();`                     | 毎月1日の00:00にタスクを実行。   |
| `->monthlyOn(4, '15:00');`         | 毎月4日の15:00にタスクを実行。            |
| `->twiceMonthly(1, 16, '13:00');`  | 毎月1日と16日の13:00にタスクを実行。       |
| `->lastDayOfMonth('15:00');`       | 毎月最終日の15:00にタスクを実行。      |
| `->quarterly();`                   | 四半期の最初の日の00:00にタスクを実行。 |
| `->quarterlyOn(4, '14:00');`       | 四半期の4日の14:00にタスクを実行。          |
| `->yearly();`                      | 毎年1月1日の00:00にタスクを実行。    |
| `->yearlyOn(6, 1, '17:00');`       | 毎年6月1日の17:00にタスクを実行。            |
| `->timezone('America/New_York');`  | タスクのタイムゾーンを設定。                           |

</div>

これらのメソッドを追加の制約と組み合わせて、さらに細かく調整されたスケジュールを作成できます。たとえば、毎週月曜日にコマンドを実行するようにスケジュールすることができます。

    use Illuminate\Support\Facades\Schedule;

    $schedule->command('emails:send')->weekly()->mondays()->at('8:00');

    // 毎週月曜日の午後1時に1回実行...
    Schedule::call(function () {
        // ...
    })->weekly()->mondays()->at('13:00');

    // 平日の午前8時から午後5時まで毎時間実行...
    Schedule::command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

以下に、追加のスケジュール制約のリストを示します:

<div class="overflow-auto" markdown=1>

| メソッド                                   | 説明                                            |
| ---------------------------------------- | ------------------------------------------------------ |
| `->weekdays();`                          | タスクを平日に限定します。                            |
| `->weekends();`                          | タスクを週末に限定します。                            |
| `->sundays();`                           | タスクを日曜日に限定します。                          |
| `->mondays();`                           | タスクを月曜日に限定します。                          |
| `->tuesdays();`                          | タスクを火曜日に限定します。                          |
| `->wednesdays();`                        | タスクを水曜日に限定します。                          |
| `->thursdays();`                         | タスクを木曜日に限定します。                          |
| `->fridays();`                           | タスクを金曜日に限定します。                          |
| `->saturdays();`                         | タスクを土曜日に限定します。                          |
| `->days(array\|mixed);`                  | タスクを特定の曜日に限定します。                      |
| `->between($startTime, $endTime);`       | タスクを開始時間と終了時間の間に実行するように制限します。 |
| `->unlessBetween($startTime, $endTime);` | タスクを開始時間と終了時間の間に実行しないように制限します。 |
| `->when(Closure);`                       | 真偽テストに基づいてタスクを制限します。              |
| `->environments($env);`                  | タスクを特定の環境に限定します。                      |

</div>

<a name="day-constraints"></a>
#### 曜日制約

`days` メソッドは、タスクの実行を特定の曜日に限定するために使用できます。例えば、毎時間日曜日と水曜日にコマンドを実行するようにスケジュールすることができます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('emails:send')
                    ->hourly()
                    ->days([0, 3]);

または、タスクが実行される曜日を定義する際に、`Illuminate\Console\Scheduling\Schedule` クラスで利用可能な定数を使用することもできます:

    use Illuminate\Support\Facades;
    use Illuminate\Console\Scheduling\Schedule;

    Facades\Schedule::command('emails:send')
                    ->hourly()
                    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);

<a name="between-time-constraints"></a>
#### 時間帯制約

`between` メソッドは、タスクの実行を一日の中の特定の時間帯に限定するために使用できます:

    Schedule::command('emails:send')
                        ->hourly()
                        ->between('7:00', '22:00');

同様に、`unlessBetween` メソッドは、タスクの実行を特定の時間帯から除外するために使用できます:

    Schedule::command('emails:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

<a name="truth-test-constraints"></a>
#### 真偽テスト制約

`when` メソッドは、与えられた真偽テストの結果に基づいてタスクの実行を制限するために使用できます。言い換えれば、与えられたクロージャが `true` を返す場合、他の制約条件がタスクの実行を妨げない限り、タスクは実行されます:

    Schedule::command('emails:send')->daily()->when(function () {
        return true;
    });

`skip` メソッドは `when` メソッドの逆と見なすことができます。`skip` メソッドが `true` を返す場合、スケジュールされたタスクは実行されません:

    Schedule::command('emails:send')->daily()->skip(function () {
        return true;
    });

`when` メソッドを連鎖させる場合、スケジュールされたコマンドはすべての `when` 条件が `true` を返す場合にのみ実行されます。

<a name="environment-constraints"></a>
#### 環境制約

`environments` メソッドは、タスクを特定の環境（`APP_ENV` [環境変数](configuration.md#environment-configuration)で定義されたもの）でのみ実行するために使用できます:

    Schedule::command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);

<a name="timezones"></a>
### タイムゾーン

`timezone` メソッドを使用すると、スケジュールされたタスクの時間を特定のタイムゾーン内で解釈するように指定できます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('report:generate')
             ->timezone('America/New_York')
             ->at('2:00')

スケジュールされたすべてのタスクに同じタイムゾーンを繰り返し割り当てる場合、アプリケーションの `app` 設定ファイル内で `schedule_timezone` オプションを定義することで、すべてのスケジュールに割り当てるタイムゾーンを指定できます:

    'timezone' => env('APP_TIMEZONE', 'UTC'),

    'schedule_timezone' => 'America/Chicago',

> WARNING:  
> 一部のタイムゾーンは夏時間を使用していることに注意してください。夏時間の変更が発生すると、スケジュールされたタスクが2回実行されたり、まったく実行されなかったりする可能性があります。このため、可能な限りタイムゾーンのスケジューリングは避けることをお勧めします。

<a name="preventing-task-overlaps"></a>
### タスクの重複防止

デフォルトでは、スケジュールされたタスクは前のインスタンスがまだ実行中であっても実行されます。これを防ぐために、`withoutOverlapping` メソッドを使用できます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('emails:send')->withoutOverlapping();

この例では、`emails:send` [Artisan コマンド](artisan.md) は、まだ実行中でない場合に毎分実行されます。`withoutOverlapping` メソッドは、タスクの実行時間が大きく変動する場合に特に便利です。これにより、特定のタスクがどれだけ時間がかかるかを正確に予測することができません。

必要に応じて、「重複なし」ロックが期限切れになるまでに何分経過する必要があるかを指定できます。デフォルトでは、ロックは24時間後に期限切れになります:

    Schedule::command('emails:send')->withoutOverlapping(10);

内部的には、`withoutOverlapping` メソッドはアプリケーションの [キャッシュ](cache.md) を使用してロックを取得します。必要に応じて、`schedule:clear-cache` Artisan コマンドを使用してこれらのキャッシュロックをクリアできます。これは通常、タスクが予期せぬサーバーの問題でスタックした場合にのみ必要です。

<a name="running-tasks-on-one-server"></a>
### 1台のサーバーでタスクを実行

> WARNING:  
> この機能を利用するには、アプリケーションが `database`、`memcached`、`dynamodb`、または `redis` キャッシュドライバーをアプリケーションのデフォルトキャッシュドライバーとして使用している必要があります。さらに、すべてのサーバーが同じ中央キャッシュサーバーと通信している必要があります。

アプリケーションのスケジューラが複数のサーバーで実行されている場合、スケジュールされたジョブを1台のサーバーでのみ実行するように制限できます。例えば、毎週金曜日の夜に新しいレポートを生成するスケジュールされたタスクがあるとします。スケジューラが3台のワーカーサーバーで実行されている場合、スケジュールされたタスクは3台のサーバーすべてで実行され、レポートが3回生成されます。これは良くありません！

タスクが1台のサーバーでのみ実行されるように指示するには、スケジュールされたタスクを定義する際に `onOneServer` メソッドを使用します。最初のサーバーがタスクを取得すると、他のサーバーが同時に同じタスクを実行するのを防ぐために、ジョブに対してアトミックロックが取得されます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();

<a name="naming-unique-jobs"></a>
#### 単一サーバージョブの命名

同じジョブを異なるパラメータでディスパッチするようにスケジュールする必要がある場合、Laravelに各パラメータの組み合わせを1台のサーバーで実行するように指示することができます。これを実現するには、`name` メソッドを介して各スケジュール定義に一意の名前を割り当てることができます:

```php
Schedule::job(new CheckUptime('https://laravel.com'))
            ->name('check_uptime:laravel.com')
            ->everyFiveMinutes()
            ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
            ->name('check_uptime:vapor.laravel.com')
            ->everyFiveMinutes()
            ->onOneServer();
```

同様に、単一サーバーで実行することを意図したスケジュールされたクロージャには名前を付ける必要があります:

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### バックグラウンドタスク

デフォルトでは、同時にスケジュールされた複数のタスクは、`schedule` メソッドで定義された順序に基づいて順次実行されます。長時間実行されるタスクがある場合、これにより後続のタスクが予想よりもはるかに遅く開始される可能性があります。バックグラウンドでタスクを実行して、すべてのタスクを同時に実行できるようにする場合は、`runInBackground` メソッドを使用できます:

    use Illuminate\Support\Facades\Schedule;

    Schedule::command('analytics:report')
             ->daily()
             ->runInBackground();

> WARNING:  
> `runInBackground` メソッドは、`command` および `exec` メソッドを介してタスクをスケジュールする場合にのみ使用できます。

<a name="maintenance-mode"></a>
### メンテナンスモード

アプリケーションが[メンテナンスモード](configuration.md#maintenance-mode)の場合、アプリケーションのスケジュールされたタスクは実行されません。これは、タスクがサーバーで実行されている未完了のメンテナンス作業に干渉しないようにするためです。ただし、メンテナンスモードでもタスクを強制的に実行したい場合は、タスクを定義する際に `evenInMaintenanceMode` メソッドを呼び出すことができます:

    Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="running-the-scheduler"></a>
## スケジューラの実行

スケジュールされたタスクを定義する方法を学んだので、次にサーバー上で実際にそれらを実行する方法について説明します。`schedule:run` Artisanコマンドは、サーバーの現在時刻に基づいてすべてのスケジュールされたタスクを評価し、実行が必要かどうかを判断します。

したがって、Laravelのスケジューラを使用する場合、`schedule:run`コマンドを毎分実行する単一のcron設定エントリをサーバーに追加するだけで済みます。サーバーにcronエントリを追加する方法がわからない場合は、cronエントリを管理してくれる[Laravel Forge](https://forge.laravel.com)などのサービスを利用することを検討してください。

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### 1分未満のスケジュールされたタスク

ほとんどのオペレーティングシステムでは、cronジョブは最大で1分に1回実行されるように制限されています。しかし、Laravelのスケジューラを使用すると、タスクをより頻繁に、1秒に1回までスケジュールすることができます。

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

1分未満のタスクがアプリケーション内で定義されている場合、`schedule:run`コマンドは現在の分の終わりまで実行し続け、すぐに終了するのではなく、その分の間に必要なすべての1分未満のタスクを呼び出すことができます。

予想よりも長く実行される1分未満のタスクが、後続の1分未満のタスクの実行を遅らせる可能性があるため、すべての1分未満のタスクがキューに入れられたジョブまたはバックグラウンドコマンドをディスパッチして、実際のタスク処理を行うことをお勧めします。

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### 1分未満のタスクの中断

1分未満のタスクが定義されている場合、`schedule:run`コマンドは呼び出しの分全体を実行するため、アプリケーションをデプロイする際にコマンドを中断する必要がある場合があります。そうしないと、すでに実行中の`schedule:run`コマンドのインスタンスが、現在の分が終了するまで、以前にデプロイされたアプリケーションのコードを引き続き使用します。

進行中の`schedule:run`呼び出しを中断するには、アプリケーションのデプロイスクリプトに`schedule:interrupt`コマンドを追加できます。このコマンドは、アプリケーションのデプロイが完了した後に呼び出す必要があります。

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### ローカルでのスケジューラの実行

通常、ローカル開発マシンにスケジューラのcronエントリを追加することはありません。代わりに、`schedule:work` Artisanコマンドを使用できます。このコマンドはフォアグラウンドで実行され、コマンドを終了するまで1分ごとにスケジューラを呼び出します。

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## タスクの出力

Laravelのスケジューラは、スケジュールされたタスクによって生成された出力を操作するためのいくつかの便利なメソッドを提供します。まず、`sendOutputTo`メソッドを使用して、後で検査するために出力をファイルに送信できます。

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
```

出力を特定のファイルに追加する場合は、`appendOutputTo`メソッドを使用できます。

```php
Schedule::command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
```

`emailOutputTo`メソッドを使用すると、出力を選択したメールアドレスにメールで送信できます。タスクの出力をメールで送信する前に、Laravelの[メールサービス](mail.md)を設定する必要があります。

```php
Schedule::command('report:generate')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('taylor@example.com');
```

スケジュールされたArtisanまたはシステムコマンドがゼロ以外の終了コードで終了した場合にのみ出力をメールで送信する場合は、`emailOutputOnFailure`メソッドを使用します。

```php
Schedule::command('report:generate')
         ->daily()
         ->emailOutputOnFailure('taylor@example.com');
```

> WARNING:  
> `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo`、および`appendOutputTo`メソッドは、`command`および`exec`メソッドに対してのみ使用できます。

<a name="task-hooks"></a>
## タスクフック

`before`および`after`メソッドを使用して、スケジュールされたタスクの実行前後に実行されるコードを指定できます。

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
         ->daily()
         ->before(function () {
             // タスクが実行されようとしています...
         })
         ->after(function () {
             // タスクが実行されました...
         });
```

`onSuccess`および`onFailure`メソッドを使用すると、スケジュールされたタスクが成功または失敗した場合に実行されるコードを指定できます。失敗は、スケジュールされたArtisanまたはシステムコマンドがゼロ以外の終了コードで終了したことを示します。

```php
Schedule::command('emails:send')
         ->daily()
         ->onSuccess(function () {
             // タスクが成功しました...
         })
         ->onFailure(function () {
             // タスクが失敗しました...
         });
```

コマンドから出力が利用可能な場合、`after`、`onSuccess`、または`onFailure`フックで`Illuminate\Support\Stringable`インスタンスを型ヒントとして指定することで、フックのクロージャ定義の`$output`引数としてアクセスできます。

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
         ->daily()
         ->onSuccess(function (Stringable $output) {
             // タスクが成功しました...
         })
         ->onFailure(function (Stringable $output) {
             // タスクが失敗しました...
         });
```

<a name="pinging-urls"></a>
#### URLのピング

`pingBefore`および`thenPing`メソッドを使用すると、スケジューラはタスクの実行前後に指定されたURLに自動的にピングできます。このメソッドは、スケジュールされたタスクが開始または終了したことを[Envoyer](https://envoyer.io)などの外部サービスに通知するのに便利です。

```php
Schedule::command('emails:send')
         ->daily()
         ->pingBefore($url)
         ->thenPing($url);
```

`pingBeforeIf`および`thenPingIf`メソッドは、指定された条件が`true`の場合にのみURLにピングするために使用できます。

```php
Schedule::command('emails:send')
         ->daily()
         ->pingBeforeIf($condition, $url)
         ->thenPingIf($condition, $url);
```

`pingOnSuccess`および`pingOnFailure`メソッドは、タスクが成功または失敗した場合にのみ指定されたURLにピングするために使用できます。失敗は、スケジュールされたArtisanまたはシステムコマンドがゼロ以外の終了コードで終了したことを示します。

```php
Schedule::command('emails:send')
         ->daily()
         ->pingOnSuccess($successUrl)
         ->pingOnFailure($failureUrl);
```

<a name="events"></a>
## イベント

Laravelは、スケジューリングプロセス中にさまざまな[イベント](events.md)をディスパッチします。次のイベントのいずれかに対して[リスナーを定義](events.md)できます。

<div class="overflow-auto" markdown=1>

| イベント名 |
| --- |
| `Illuminate\Console\Events\ScheduledTaskStarting` |
| `Illuminate\Console\Events\ScheduledTaskFinished` |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped` |
| `Illuminate\Console\Events\ScheduledTaskFailed` |

</div>
