# サービスプロバイダ

- [イントロダクション](#introduction)
- [サービスプロバイダの記述](#writing-service-providers)
    - [Registerメソッド](#the-register-method)
    - [Bootメソッド](#the-boot-method)
- [プロバイダの登録](#registering-providers)
- [遅延プロバイダ](#deferred-providers)

<a name="introduction"></a>
## イントロダクション

サービスプロバイダは、Laravelアプリケーション全体の起動処理における、初めの心臓部です。皆さんのアプリケーションと同じく、Laravelのコアサービス全部もサービスプロバイダを利用し、初期起動処理を行っています。

ところで「初期起動処理」とは何を意味しているのでしょうか？サービスコンテナによる結合や、イベントリスナ、ミドルウェア、それにルートなどを**登録する**ことを一般的に意味しています。サービスプロバイダはアプリケーション設定の中心部です。

Laravelは、メーラー、キュー、キャッシュなどのコアサービスを初期起動するために、内部で何十ものサービスプロバイダを使用しています。これらのプロバイダの多くは、「遅延」プロバイダであり、すべてのリクエストでロードされるのではなく、提供するサービスが実際に必要になったときにのみロードすることを意味します。

ユーザー定義サービスプロバイダは全て、`bootstrap/providers.php`ファイルへ登録します。このドキュメントの以降で、独自のサービスプロバイダを作成し、Laravelアプリケーションへ登録する方法を学びます。

> [!NOTE]
> Laravelがリクエストをどのように処理し、内部で動作しているかについて詳しく知りたい場合は、Laravelの[リクエストライフサイクル](/docs/{{version}}/lifecycle)に関するドキュメントを確認してください。

<a name="writing-service-providers"></a>
## サービスプロバイダの記述

すべてのサービスプロバイダは、`Illuminate\Support\ServiceProvider`クラスを拡張します。ほとんどのサービスプロバイダは、`register`と`boot`メソッドを持っています。`register`メソッドの中では**[サービスコンテナ](/docs/{{version}}/container)への登録だけ**を行わなくてはなりません。他のイベントリスナやルート、その他の機能の一部でも、`register`メソッドの中で登録しようとしてはいけません。

`make:provider` Artisanコマンドラインにより、新しいプロバイダを生成できます。Laravelは、アプリケーションの`bootstrap/providers.php`ファイルへ、新しいプロバイダーを自動的に登録します。

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Registerメソッド

すでに説明した通り、`register`メソッドの中では[サービスコンテナ](/docs/{{version}}/container)に何かを結合することだけを行わなければなりません。イベントリスナやルート、その他のどんな機能も`register`メソッドの中では決して行ってはいけません。これを守らないと、サービスプロバイダがまだロードしていないサービスを意図せず使ってしまう羽目になるでしょう。

では、基本的なサービスプロバイダを見てみましょう。サービスプロバイダメソッド中であれば、いつでも`$app`プロパティを利用でき、サービスコンテナへアクセスできます。

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 全アプリケーションサービスの登録
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

このサービスプロバイダは`register`メソッドのみを定義し、このメソッドを使用してサービスコンテナ内の`App\Services\Riak\Connection`の実装を定義します。Laravelのサービスコンテナにまだ慣れていない方は、[ドキュメント](/docs/{{version}}/container)で確認してください。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings`と`singletons`プロパティ

サービスプロバイダでシンプルな結合をたくさん登録しているのであれば、各コンテナ結合を自力で登録する代わりに、`bindings` と`singletons`プロパティを使いたくなるでしょう。フレームワークにより、サービスプロバイダがロードされる時点で、これらのプロパティがチェックされ、結合を登録します。

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 登録する必要のある全コンテナ結合
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * 登録する必要のある全コンテナシングルトン
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

<a name="the-boot-method"></a>
### Bootメソッド

では、[ビューコンポーサ](/docs/{{version}}/views#view-composers)をサービスプロバイダで登録する必要がある場合は、どうすればよいのでしょうか？　`boot`メソッドの中で行ってください。**このメソッドは、他の全サービスプロバイダが登録し終えてから呼び出されます**。つまりフレームワークにより登録された、他のサービスすべてにアクセスできるのです。

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * アプリケーションの全サービスの初期起動処理
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

<a name="boot-method-dependency-injection"></a>
#### bootメソッドの依存注入

サービスプロバイダの`boot`メソッドでは、依存をタイプヒントで指定できます。[サービスコンテナ](/docs/{{version}}/container)が、必要な依存を自動的に注入します。

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * 全アプリケーションサービスの初期起動処理
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

<a name="registering-providers"></a>
## プロバイダの登録

全てのサービスプロバイダは、`bootstrap/providers.php`設定ファイルで登録します。このファイルは、アプリケーションのサービスプロバイダのクラス名を含む配列を返します。

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

`make:providers` Artisanコマンドを実行すると、Laravelは生成したプロバイダを自動的に、`bootstrap/providers.php`ファイルへ追加します。しかし、自分でプロバイダクラスを作成した場合は、プロバイダクラスを配列へ手作業で追加する必要があります。

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // [tl! add]
];
```

<a name="deferred-providers"></a>
## 遅延プロバイダ

もし皆さんのプロバイダが、[サービスコンテナ](/docs/{{version}}/container)へコンテナ結合を登録する**だけ**であるなら、その結合が実際に必要になるまで登録を遅らせる方が良いでしょう。こうしたプロバイダのローディングを遅らせるのは、リクエストがあるたびにファイルシステムからロードされなくなるため、アプリケーションのパフォーマンスを向上させます。

Laravelは遅延サービスプロバイダが提示した全サービスのリストをコンパイルし、サービスプロバイダのクラス名と共に保存します。その後、登録されているサービスのどれか一つを依存解決する必要が起きた時のみ、Laravelはそのサービスプロバイダをロードします。

プロバイダを遅延ロードするには、`\Illuminate\Contracts\Support\DeferrableProvider`インターフェイスを実装し、`provides`メソッドを定義します。`provides`メソッドはそのプロバイダで登録したサービスコンテナ結合を返します。

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * 全アプリケーションサービスの登録
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * このプロバイダが提供するサービスの取得
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```
