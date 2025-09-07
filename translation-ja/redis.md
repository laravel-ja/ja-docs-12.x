# Redis

- [イントロダクション](#introduction)
- [設定](#configuration)
    - [クラスタ](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [Redisの操作](#interacting-with-redis)
    - [トランザクション](#transactions)
    - [パイプラインコマンド](#pipelining-commands)
- [Pub／Sub](#pubsub)

<a name="introduction"></a>
## イントロダクション

[Redis](https://redis.io)はオープンソースの高度なキー／値保存域です。キーには[文字列](https://redis.io/docs/latest/develop/data-types/strings/)、[ハッシュ](https://redis.io/docs/latest/develop/data-types/hashes/)、[リスト](https://redis.io/docs/latest/develop/data-types/lists/)、[セット](https://redis.io/docs/latest/develop/data-types/sets/)、[ソート済みセット](https://redis.io/docs/latest/develop/data-types/sorted-sets/)を含むことができるため、データ構造サーバと呼ばれることが多いです。

LaravelでRedisを使い始める前に、PECLにより[PhpRedis](https://github.com/phpredis/phpredis)PHP拡張機能をインストールして使用することを推奨します。この拡張機能は、「ユーザーフレンドリー」なPHPパッケージに比べてインストールは複雑ですが、Redisを多用するアプリケーションのパフォーマンスが向上する可能性があります。[Laravel Sail](/docs/{{version}}/sail)を使用している場合、この拡張機能はアプリケーションのDockerコンテナにはじめからインストールしてあります。

PhpRedis拡張機能をインストールできない場合は、Composerを介して`predis/predis`パッケージをインストールしてください。PredisはすべてPHPで記述されたRedisクライアントであり、追加の拡張機能は必要ありません。

```shell
composer require predis/predis
```

<a name="configuration"></a>
## 設定

`config/database.php`設定ファイルによりアプリケーションのRedis設定を設定します。このファイル内に、アプリケーションで使用するRedisサーバを含む`redis`配列があります。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

],
```

設定ファイルで定義する各Redisサーバは、Redis接続を表す単一のURLを定義しない限り、名前、ホスト、およびポートを指定する必要があります。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => 'tcp://127.0.0.1:6379?database=0',
    ],

    'cache' => [
        'url' => 'tls://user:password@127.0.0.1:6380?database=1',
    ],

],
```

<a name="configuring-the-connection-scheme"></a>
#### 接続スキームの構成

RedisクライアントはRedisサーバへ接続するとき、デフォルトで`tcp`スキームを使用します。しかし、Redisサーバの設定配列で`scheme`設定オプションを指定すれば、TLS/SSL暗号化を使用できます。

```php
'default' => [
    'scheme' => 'tls',
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

<a name="clusters"></a>
### クラスタ

アプリケーションがRedisサーバのクラスタを利用する場合は、Redis設定の`clusters`キー内で使用するクラスタを定義してください。この設定キーはデフォルトでは存在しないため、アプリケーションの`config/database.php`設定ファイル内で作成する必要があります。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'clusters' => [
        'default' => [
            [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'username' => env('REDIS_USERNAME'),
                'password' => env('REDIS_PASSWORD'),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_DB', '0'),
            ],
        ],
    ],

    // ...
],
```

`options.cluster`の設定値が`redis`に設定されているため、Laravelはデフォルトで、ネイティブのRedisクラスタリングを使用します。Redisクラスタリングは丁寧にフェイルオーバーを処理するので、素晴らしいデフォルトオプションです。

Predisを使用する場合、Laravelはクライアントサイドシャーディングもサポートしています。しかし、クライアントサイドシャーディングはフェイルオーバーを処理しません。そのため、主に別のプライマリデータストアから利用可能な一時的なキャッシュデータに適しています。

ネイティブのRedisクラスタリングではなく、クライアントサイドのシャーディングを使用したい場合は、アプリケーションの`config/database.php`設定ファイル内の`options.cluster`設定値を削除してください。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        // ...
    ],

    // ...
],
```

<a name="predis"></a>
### Predis

アプリケーションがPredisパッケージを介してRedisを操作するようにする場合は、`REDIS_CLIENT`環境変数の値を確実に`predis`へ設定してください。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```

デフォルトの設定オプションに加え、Predisは追加の[接続パラメータ](https://github.com/nrk/predis/wiki/Connection-Parameters)をサポートしており、Redisサーバごとに定義できます。これらの追加設定オプションを使用するには、アプリケーションの`config/database.php`設定ファイルで、Redisサーバの設定に追加します。

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_write_timeout' => 60,
],
```

<a name="phpredis"></a>
### PhpRedis

デフォルトでは、LaravelはPhpRedis拡張機能を使用してRedisと通信します。LaravelがRedisとの通信に使用するクライアントは、`redis.client`設定オプションの値で決まります。これは通常、`REDIS_CLIENT`環境変数の値を反映します。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // ...
],
```

デフォルトの設定オプションに加えて、PhpRedisは`name`、`persistent`、`persistent_id`、`prefix`、`read_timeout`、`retry_interval`、`max_retries`、`backoff_algorithm`、`backoff_base`、`backoff_cap`,`timeout`、`context`の追加接続パラメータをサポートしています。これらのオプションは、`config/database.php`設定ファイルのRedisサーバ設定へ追加してください。

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

<a name="retry-and-backoff-configuration"></a>
#### リトライとバックオフの設定

`retry_interval`、`max_retries`、`backoff_algorithm`、`backoff_base`、`backoff_cap`オプションを使用して、PhpRedisクライアントがRedisサーバへの再接続を試みる方法を設定できます。以降のバックオフアルゴリズムをサポートしています。`default`、`decorrelated_jitter`、`equal_jitter`、`exponential`、`uniform`、`constant`

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

<a name="unix-socket-connections"></a>
#### Unixソケット接続

Redis接続は、TCPの代わりにUnixソケットを使用するように設定することもできます。これにより、アプリケーションと同じサーバ上のRedisインスタンスに接続する際のTCPオーバーヘッドがなくなり、パフォーマンスが向上します。Unixソケットを使用するようにRedisを設定するには、`REDIS_HOST`環境変数にRedisソケットのパスを設定し、`REDIS_PORT`環境変数に`0`を設定します。

```env
REDIS_HOST=/run/redis/redis.sock
REDIS_PORT=0
```

<a name="phpredis-serialization"></a>
#### PhpRedisのシリアライズと圧縮

PhpRedis拡張モジュールは、様々なシリアライズや圧縮アルゴリズムも設定できます。これらのアルゴリズムは、Redis設定の`options`配列で指定します。

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        'serializer' => Redis::SERIALIZER_MSGPACK,
        'compression' => Redis::COMPRESSION_LZ4,
    ],

    // ...
],
```

現在サポートしているシリアライザは、`Redis::SERIALIZER_NONE`（デフォルト）、`Redis::SERIALIZER_PHP`、`Redis::SERIALIZER_JSON`、`Redis::SERIALIZER_IGBINARY`、`Redis::SERIALIZER_MSGPACK`です。

サポートする圧縮アルゴリズムは、`Redis::COMPRESSION_NONE`（デフォルト）、`Redis::COMPRESSION_LZF`、`Redis::COMPRESSION_ZSTD`、`Redis::COMPRESSION_LZ4`です。

<a name="interacting-with-redis"></a>
## Redisの操作

`Redis`[ファサード](/docs/{{version}}/facades)でさまざまなメソッドを呼び出すことで、Redisを操作できます。`Redis`ファサードは動的メソッドをサポートしています。つまり、ファサードで[Redisコマンド](https://redis.io/commands)を呼び出すと、コマンドが直接Redisに渡されます。この例では、`Redis`ファサードで`get`メソッドを呼び出すことにより、Redisの`GET`コマンドを呼び出しています。

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 指定ユーザーのプロファイル表示
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```

上記のように、`Redis`ファサードで任意のRedisコマンドを呼び出すことができます。Laravelはマジックメソッドを使用してコマンドをRedisサーバへ渡します。Redisコマンドが引数を必要とする場合は、それらをファサードの対応するメソッドへ渡す必要があります。

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

または、`Redis`ファサードの`command`メソッドを使用してサーバにコマンドを渡すこともできます。このメソッドは、コマンドの名前を最初の引数、値の配列を２番目の引数に取ります。

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### 複数のRedis接続の使用

アプリケーションの`config/database.php`設定ファイルでは、複数のRedis接続／サーバが定義できます。`Redis`ファサードの`connection`メソッドを使用して、特定のRedis接続への接続を取得できます。

```php
$redis = Redis::connection('connection-name');
```

デフォルトのRedis接続のインスタンスを取得するには、引数なしで`connection`メソッドを呼び出してください。

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### トランザクション

`Redis`ファサードの`transaction`メソッドは、Redisのネイティブ`MULTI`および`EXEC`コマンドの便利なラッパーを提供します。`transaction`メソッドは、唯一の引数にクロージャを取ります。このクロージャはRedis接続インスタンスを受け取り、このインスタンスで必要なコマンドを発行できます。クロージャ内で発行したすべてのRedisコマンドは、単一のアトミックトランザクションとして実行します。

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> [!WARNING]
> Redisトランザクションを定義する場合、Redis接続から値を取得できません。トランザクションは単一のアトミック操作として実行し、クロージャ全体がコマンドの実行を完了するまで、操作は実行されないことに注意してください。

#### Luaスクリプト

`eval`メソッドは、単一のアトミック操作で複数のRedisコマンドを実行する別のメソッドです。ただし、`eval`メソッドには、その操作中にRedisキー／値を対話し調べられる利点があります。Redisスクリプトは、[Luaプログラミング言語](https://www.lua.org)で記述します。

`eval`メソッドは最初は少し怖いかもしれませんが、壁を超えるために基本的な例を見てみましょう。`eval`メソッドは引数をいくつか取ります。まず、Luaスクリプトを(文字列として)メソッドへ渡す必要があります。次に、スクリプトが操作するキーの数を(整数として)渡す必要があります。第三に、それらのキーの名前を渡す必要があります。最後に、スクリプト内でアクセスする必要があるその他の追加の引数を渡します。

この例では、カウンターを増分し、その新しい値を検査し、最初のカウンターの値が５より大きい場合は、２番目のカウンターを増分します。最後に、最初のカウンターの値を返します。

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

> [!WARNING]
> Redisスクリプトの詳細には、[Redisドキュメント](https://redis.io/commands/eval)を参照してください。

<a name="pipelining-commands"></a>
### パイプラインコマンド

数十のRedisコマンドを実行する必要が起きることもあるでしょう。毎コマンドごとにRedisサーバへネットワークトリップする代わりに、`pipeline`メソッドを使用できます。`pipeline`メソッドは、Redisインスタンスを受け取るクロージャを１つ引数に取ります。このRedisインスタンスですべてのコマンドを発行すると、サーバへのネットワークトリップを減らすため、すべてのコマンドを一度にRedisサーバへ送信します。コマンドは、発行した順序で実行されます。

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## Pub／Sub

Laravelは、Redisの`publish`および`subscribe`コマンドへの便利なインターフェイスを提供しています。これらのRedisコマンドを使用すると、特定の「チャンネル」でメッセージをリッスンできます。別のアプリケーションから、または別のプログラミング言語を使用してメッセージをチャンネルに公開し、アプリケーションとプロセス間の簡単な通信を可能にすることができます。

まず、`subscribe`メソッドを使用してチャンネルリスナを設定しましょう。`subscribe`メソッドを呼び出すとプロセスは長時間実行され続けるため、このメソッド呼び出しは[Artisanコマンド](/docs/{{version}}/artisan)内に配置します。

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * consoleコマンドの名前と使用方法
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * コンソールコマンドの説明
     *
     * @var string
     */
    protected $description = 'Subscribe to a Redis channel';

    /**
     * consoleコマンドの実行
     */
    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

これで、`publish`メソッドを使用してチャンネルへメッセージを発行できます。

```php
use Illuminate\Support\Facades\Redis;

Route::get('/publish', function () {
    // ...

    Redis::publish('test-channel', json_encode([
        'name' => 'Adam Wathan'
    ]));
});
```

<a name="wildcard-subscriptions"></a>
#### ワイルドカードサブスクリプション

`psubscribe`メソッドを使用すると、ワイルドカードチャンネルをサブスクライブできます。これは、すべてのチャンネルのすべてのメッセージをキャッチするのに役立ちます。チャンネル名は、引数に渡すクロージャの２番目の引数へ渡されます。

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```
