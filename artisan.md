# Artisan Console

- [မိတ်ဆက်](#introduction)
    - [Tinker (REPL)](#tinker)
- [Command များရေးသားခြင်း](#writing-commands)
    - [Command များထုတ်လုပ်ခြင်း](#generating-commands)
    - [Command ဖွဲ့စည်းပုံ](#command-structure)
    - [Closure Commands](#closure-commands)
    - [Isolatable Commands](#isolatable-commands)
- [Input လိုအပ်ချက်များ သတ်မှတ်ခြင်း](#defining-input-expectations)
    - [Arguments](#arguments)
    - [Options](#options)
    - [Input Arrays](#input-arrays)
    - [Input ဖော်ပြချက်များ](#input-descriptions)
    - [မရှိသော Input များအတွက် မေးမြန်းခြင်း](#prompting-for-missing-input)
- [Command I/O](#command-io)
    - [Input ရယူခြင်း](#retrieving-input)
    - [Input အတွက် မေးမြန်းခြင်း](#prompting-for-input)
    - [Output ရေးသားခြင်း](#writing-output)
- [Command များ မှတ်ပုံတင်ခြင်း](#registering-commands)
- [Program မှတဆင့် Command များ အသုံးပြုခြင်း](#programmatically-executing-commands)
    - [အခြား Command များမှ Command များကို ခေါ်ယူခြင်း](#calling-commands-from-other-commands)
- [Signal ကိုင်တွယ်ခြင်း](#signal-handling)
- [Stub ပြုပြင်မွမ်းမံခြင်း](#stub-customization)
- [Events](#events)

<a name="introduction"></a>
## မိတ်ဆက်

Artisan သည် Laravel နှင့်အတူ ပါဝင်လာသော command line interface ဖြစ်ပါသည်။ Artisan သည် သင့် application ၏ root directory တွင် `artisan` script အဖြစ် တည်ရှိပြီး သင့် application တည်ဆောက်နေစဉ် အထောက်အကူပြုမည့် command များစွာကို ပေးထားပါသည်။ ရရှိနိုင်သော Artisan command များအားလုံးကို ကြည့်ရှုရန် `list` command ကို အသုံးပြုနိုင်ပါသည်:

```shell
php artisan list
```

command တစ်ခုချင်းစီတွင် command ၏ ရရှိနိုင်သော argument နှင့် option များကို ဖော်ပြပေးသော "help" screen လည်း ပါဝင်ပါသည်။ help screen ကို ကြည့်ရှုရန် command အမည်ရှေ့တွင် `help` ကို ထည့်သွင်းပါ:

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

သင်သည် [Laravel Sail](/docs/{{version}}/sail) ကို local development environment အဖြစ် အသုံးပြုနေပါက Artisan command များကို ခေါ်ယူရန် `sail` command line ကို အသုံးပြုရန် မမေ့ပါနှင့်။ Sail သည် သင့် application ၏ Docker container များအတွင်း Artisan command များကို အသုံးပြုပေးမည် ဖြစ်ပါသည်:

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker သည် [PsySH](https://github.com/bobthecow/psysh) package မှ power ရယူထားသော Laravel framework အတွက် powerful REPL တစ်ခု ဖြစ်ပါသည်။

<a name="installation"></a>
#### ထည့်သွင်းခြင်း

Laravel application အားလုံးတွင် Tinker ကို default အနေဖြင့် ထည့်သွင်းထားပါသည်။ သို့သော် သင့် application မှ ယခင်က ဖယ်ရှားထားခဲ့ပါက Composer ကို အသုံးပြု၍ Tinker ကို ပြန်လည်ထည့်သွင်းနိုင်ပါသည်:

```shell
composer require laravel/tinker
```

> [!NOTE]  
> သင့် Laravel application နှင့် အပြန်အလှန်ဆက်သွယ်ရာတွင် hot reloading, multiline code editing နှင့် autocompletion တို့ကို အသုံးပြုလိုပါသလား? [Tinkerwell](https://tinkerwell.app) ကို စစ်ဆေးကြည့်ပါ!

<a name="usage"></a>
#### အသုံးပြုပုံ

Tinker သည် သင့် Laravel application တစ်ခုလုံးကို command line မှတဆင့် အပြန်အလှန်ဆက်သွယ်ခွင့်ပြုပါသည်။ ဥပမာ Eloquent model များ၊ job များ၊ event များ စသည်တို့ကို အသုံးပြုနိုင်ပါသည်။ Tinker environment သို့ ဝင်ရောက်ရန် `tinker` Artisan command ကို အသုံးပြုပါ:

```shell
php artisan tinker
```

သင်သည် Tinker ၏ configuration file ကို `vendor:publish` command ကို အသုံးပြု၍ publish လုပ်နိုင်ပါသည်:

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]  
> `dispatch` helper function နှင့် `Dispatchable` class ပေါ်ရှိ `dispatch` method သည် job ကို queue ပေါ်သို့ ထည့်သွင်းရန် garbage collection ပေါ်တွင် မူတည်နေပါသည်။ ထို့ကြောင့် tinker ကို အသုံးပြုသောအခါ `Bus::dispatch` သို့မဟုတ် `Queue::push` ကို အသုံးပြု၍ job များကို dispatch လုပ်သင့်ပါသည်။

<a name="command-allow-list"></a>
#### ခွင့်ပြုထားသော Command စာရင်း (Command Allow List)

Tinker သည် ၎င်း၏ shell အတွင်း မည်သည့် Artisan command များကို ခွင့်ပြုမည်ကို ဆုံးဖြတ်ရန် "allow" list တစ်ခုကို အသုံးပြုပါသည်။ default အနေဖြင့် `clear-compiled`၊ `down`၊ `env`၊ `inspire`၊ `migrate`၊ `migrate:install`၊ `up` နှင့် `optimize` command များကို အသုံးပြုနိုင်ပါသည်။ အကယ်၍ သင်သည် command များ ထပ်မံထည့်သွင်းလိုပါက ၎င်းတို့ကို သင့် `tinker.php` configuration file ရှိ `commands` array တွင် ထည့်သွင်းနိုင်ပါသည်:

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

<a name="classes-that-should-not-be-aliased"></a>
#### Alias မပြုလုပ်သင့်သော Class များ

ပုံမှန်အားဖြင့် Tinker သည် သင် အသုံးပြုသော class များကို အလိုအလျောက် alias ပြုလုပ်ပေးပါသည်။ သို့သော် အချို့ class များကို alias မပြုလုပ်စေလိုပါက `tinker.php` configuration file ရှိ `dont_alias` array တွင် ထည့်သွင်းနိုင်ပါသည်:

    'dont_alias' => [
        App\Models\User::class,
    ],

<a name="writing-commands"></a>
## Command များ ရေးသားခြင်း

Artisan မှ ပေးထားသော command များအပြင် သင်ကိုယ်တိုင် custom command များကိုလည်း တည်ဆောက်နိုင်ပါသည်။ Command များကို ပုံမှန်အားဖြင့် `app/Console/Commands` directory တွင် သိမ်းဆည်းပါသည်။ သို့သော် Composer မှ load လုပ်နိုင်သရွေ့ သင့်အနေဖြင့် သင်နှစ်သက်ရာ နေရာတွင် သိမ်းဆည်းနိုင်ပါသည်။

<a name="generating-commands"></a>
### Command များ ထုတ်လုပ်ခြင်း

Command အသစ်တစ်ခု တည်ဆောက်ရန် `make:command` Artisan command ကို အသုံးပြုနိုင်ပါသည်။ ဤ command သည် `app/Console/Commands` directory တွင် command class အသစ်တစ်ခုကို တည်ဆောက်ပေးပါမည်။ အကယ်၍ ဤ directory သင့် application တွင် မရှိသေးပါက စိတ်ပူစရာမလိုပါ - ပထမဆုံးအကြိမ် `make:command` Artisan command ကို အသုံးပြုသောအခါ အလိုအလျောက် တည်ဆောက်ပေးပါမည်:

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### Command ဖွဲ့စည်းပုံ

Command ကို ထုတ်လုပ်ပြီးနောက် class ၏ `signature` နှင့် `description` property များအတွက် သင့်လျော်သော တန်ဖိုးများကို သတ်မှတ်သင့်ပါသည်။ ဤ property များသည် `list` screen တွင် သင့် command ကို ဖော်ပြသောအခါ အသုံးပြုပါမည်။ `signature` property သည် [သင့် command ၏ input လိုအပ်ချက်များ](#defining-input-expectations) ကိုလည်း သတ်မှတ်ပေးပါသည်။ `handle` method ကို သင့် command ကို အသုံးပြုသောအခါ ခေါ်ယူပါမည်။ သင့် command ၏ logic ကို ဤ method တွင် ထည့်သွင်းနိုင်ပါသည်။

Command ဥပမာတစ်ခုကို ကြည့်ကြပါစို့။ command ၏ `handle` method မှတဆင့် လိုအပ်သော dependency များကို တောင်းခံနိုင်ကြောင်း သတိပြုပါ။ Laravel [service container](/docs/{{version}}/container) သည် ဤ method ၏ signature တွင် type-hint လုပ်ထားသော dependency အားလုံးကို အလိုအလျောက် inject လုပ်ပေးပါမည်:

    <?php

    namespace App\Console\Commands;

    use App\Models\User;
    use App\Support\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * The name and signature of the console command.
         *
         * @var string
         */
        protected $signature = 'mail:send {user}';

        /**
         * The console command description.
         *
         * @var string
         */
        protected $description = 'Send a marketing email to a user';

        /**
         * Execute the console command.
         */
        public function handle(DripEmailer $drip): void
        {
            $drip->send(User::find($this->argument('user')));
        }
    }

> [!NOTE]  
> Code တွင် ပိုမို reuse ဖြစ်စေရန် console command များကို သေးငယ်အောင်ထားပြီး ၎င်းတို့၏ လုပ်ငန်းများကို application service များသို့ လွှဲပေးခြင်းသည် ကောင်းမွန်သော အလေ့အကျင့်တစ်ခု ဖြစ်ပါသည်။ အထက်ပါ ဥပမာတွင် email ပို့ခြင်းဆိုင်ရာ "heavy lifting" ကို ပြုလုပ်ရန် service class တစ်ခုကို inject လုပ်ထားကြောင်း သတိပြုပါ။

<a name="exit-codes"></a>
#### Exit Codes

အကယ်၍ `handle` method မှ မည်သည့်အရာမျှ return မလုပ်ဘဲ command သည် အောင်မြင်စွာ အလုပ်လုပ်ပါက command သည် `0` exit code ဖြင့် အဆုံးသတ်ပါမည်။ သို့သော် `handle` method သည် command ၏ exit code ကို manual သတ်မှတ်ရန် integer တစ်ခုကို return လုပ်နိုင်ပါသည်:

    $this->error('Something went wrong.');

    return 1;

အကယ်၍ သင်သည် command ကို မည်သည့် method မှမဆို "fail" လုပ်လိုပါက `fail` method ကို အသုံးပြုနိုင်ပါသည်။ `fail` method သည် command ကို ချက်ချင်း အဆုံးသတ်ပြီး `1` exit code ဖြင့် return လုပ်ပါမည်:

    $this->fail('Something went wrong.');

<a name="closure-commands"></a>
### Closure Commands

Closure အခြေခံ command များသည် command များကို class များအဖြစ် သတ်မှတ်ရန်အတွက် နောက်ထပ်နည်းလမ်းတစ်ခု ဖြစ်ပါသည်။ route closure များသည် controller များအတွက် အခြားရွေးချယ်စရာ နည်းလမ်းတစ်ခု ဖြစ်သကဲ့သို့ command closure များသည် command class များအတွက် အခြားရွေးချယ်စရာ နည်းလမ်းတစ်ခု ဖြစ်သည်ဟု ယူဆနိုင်ပါသည်။

`routes/console.php` file သည် HTTP route များကို မသတ်မှတ်သော်လည်း သင့် application အတွက် console အခြေခံ entry point (route) များကို သတ်မှတ်ပေးပါသည်။ ဤ file အတွင်းတွင် သင်သည် closure အခြေခံ console command အားလုံးကို `Artisan::command` method ကို အသုံးပြု၍ သတ်မှတ်နိုင်ပါသည်။ `command` method သည် argument နှစ်ခုကို လက်ခံပါသည် - [command signature](#defining-input-expectations) နှင့် command ၏ argument များနှင့် option များကို လက်ခံသော closure တစ်ခု:

    Artisan::command('mail:send {user}', function (string $user) {
        $this->info("Sending email to: {$user}!");
    });

Closure သည် အခြေခံ command instance နှင့် bind ဖြစ်နေသောကြောင့် သင်သည် command class တစ်ခုတွင် ပုံမှန်အားဖြင့် အသုံးပြုနိုင်သော helper method အားလုံးကို အသုံးပြုနိုင်ပါသည်။

<a name="type-hinting-dependencies"></a>
#### Type-Hinting Dependencies

Command ၏ argument များနှင့် option များအပြင် command closure များသည် [service container](/docs/{{version}}/container) မှ resolve လုပ်စေလိုသော နောက်ထပ် dependency များကိုလည်း type-hint လုပ်နိုင်ပါသည်:

    use App\Models\User;
    use App\Support\DripEmailer;

    Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
        $drip->send(User::find($user));
    });

<a name="closure-command-descriptions"></a>
#### Closure Command ဖော်ပြချက်များ

Closure အခြေခံ command တစ်ခုကို သတ်မှတ်သောအခါ command အတွက် ဖော်ပြချက်ကို ထည့်သွင်းရန် `purpose` method ကို အသုံးပြုနိုင်ပါသည်။ ဤဖော်ပြချက်ကို `php artisan list` သို့မဟုတ် `php artisan help` command များကို အသုံးပြုသောအခါ ပြသပါမည်:

    Artisan::command('mail:send {user}', function (string $user) {
        // ...
    })->purpose('Send a marketing email to a user');

<a name="isolatable-commands"></a>
### Isolatable Commands

> [!WARNING]  
> ဤ feature ကို အသုံးပြုရန် သင့် application သည် `memcached`၊ `redis`၊ `dynamodb`၊ `database`၊ `file` သို့မဟုတ် `array` cache driver ကို သင့် application ၏ default cache driver အဖြစ် အသုံးပြုနေရပါမည်။ ထို့အပြင် server အားလုံးသည် တူညီသော central cache server နှင့် ဆက်သွယ်နေရပါမည်။

တစ်ခါတစ်ရံတွင် command တစ်ခု၏ instance တစ်ခုသာ တစ်ချိန်တည်းတွင် run နိုင်ရန် သတ်မှတ်လိုနိုင်ပါသည်။ ဤသို့ပြုလုပ်ရန် သင့် command class တွင် `Illuminate\Contracts\Console\Isolatable` interface ကို implement လုပ်နိုင်ပါသည်:

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Contracts\Console\Isolatable;

    class SendEmails extends Command implements Isolatable
    {
        // ...
    }

Command တစ်ခုကို `Isolatable` အဖြစ် သတ်မှတ်သောအခါ Laravel သည် command တွင် `--isolated` option ကို အလိုအလျောက် ထည့်သွင်းပေးပါမည်။ ထို option နှင့်အတူ command ကို အသုံးပြုသောအခါ Laravel သည် ထို command ၏ အခြား instance များ run နေခြင်း မရှိကြောင်း သေချာစေပါမည်။ Laravel သည် သင့် application ၏ default cache driver ကို အသုံးပြု၍ atomic lock တစ်ခုကို ရယူခြင်းဖြင့် ဤသို့ဆောင်ရွက်ပါသည်။ အကယ်၍ command ၏ အခြား instance များ run နေပါက command သည် အလုပ်မလုပ်ပါ။ သို့သော် command သည် အောင်မြင်သော exit status code ဖြင့် အဆုံးသတ်ပါမည်:

```shell
php artisan mail:send 1 --isolated
```

အကယ်၍ သင်သည် command အလုပ်မလုပ်သောအခါ return လုပ်စေလိုသော exit status code ကို သတ်မှတ်လိုပါက ထို status code ကို `isolated` option မှတဆင့် ပေးနိုင်ပါသည်:

```shell
php artisan mail:send 1 --isolated=12
```

<a name="lock-id"></a>
#### Lock ID

Default အနေဖြင့် Laravel သည် သင့် application ၏ cache တွင် atomic lock ကို ရယူရန် အသုံးပြုမည့် string key ကို ထုတ်လုပ်ရန် command ၏ အမည်ကို အသုံးပြုပါမည်။ သို့သော် သင်သည် ဤ key ကို customise လုပ်လိုပါက သင့် Artisan command class တွင် `isolatableId` method ကို သတ်မှတ်နိုင်ပါသည်။ ဤသို့ပြုလုပ်ခြင်းဖြင့် command ၏ argument များ သို့မဟုတ် option များကို key တွင် ထည့်သွင်းနိုင်ပါသည်:

```php
/**
 * Get the isolatable ID for the command.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

[ဘာသာပြန်ဆက်လက်ဆောင်ရွက်နေပါသည်...]
