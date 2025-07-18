# 認可

- [イントロダクション](#introduction)
- [ゲート](#gates)
    - [ゲートの作成](#writing-gates)
    - [アクションの認可](#authorizing-actions-via-gates)
    - [ゲートのレスポンス](#gate-responses)
    - [ゲートチェックの割り込み](#intercepting-gate-checks)
    - [インライン認可](#inline-authorization)
- [ポリシーの作成](#creating-policies)
    - [ポリシーの生成](#generating-policies)
    - [ポリシーの登録](#registering-policies)
- [ポリシーの作成](#writing-policies)
    - [ポリシーメソッド](#policy-methods)
    - [ポリシーのレスポンス](#policy-responses)
    - [モデルのないメソッド](#methods-without-models)
    - [ゲストユーザー](#guest-users)
    - [ポリシーフィルタ](#policy-filters)
- [ポリシーを使用したアクションの認可](#authorizing-actions-using-policies)
    - [ユーザーモデル経由](#via-the-user-model)
    - [Gateファサード経由](#via-the-gate-facade)
    - [ミドルウェア経由](#via-middleware)
    - [Bladeテンプレート経由](#via-blade-templates)
    - [追加コンテキストの提供](#supplying-additional-context)
- [認可とInertia](#authorization-and-inertia)

<a name="introduction"></a>
## イントロダクション

組み込み[認証](/docs/{{version}}/authentication)サービスの提供に加え、Laravelは特定のリソースに対するユーザーアクションを認可する手軽な方法も提供しています。たとえば、あるユーザーが認証されていても、アプリケーションが管理している特定のEloquentモデルまたはデータベースレコードを更新や削除する権限を持っていない場合があるでしょう。Laravelの認可機能は、こうしたタイプの認可チェックを管理するための簡単で組織化された方法を提供します。

Laravelは、アクションを認可する2つの主要な方法を提供します。[ゲート](#gates)と[ポリシー](#creating-policies)です。ゲートとポリシーは、ルートやコントローラのようなものだと考えてください。ゲートは認可のためのクロージャベースのシンプルなアプローチを提供します。一方でポリシーはコントローラのように、特定のモデルやリソース周辺のロジックをひとかたまりにまとめます。このドキュメントでは、最初にゲートを説明し、その後でポリシーを見ていきましょう。

アプリケーションを構築するときに、ゲートのみを使用するか、ポリシーのみを使用するかを選択する必要はありません。ほとんどのアプリケーションには、ゲートとポリシーが混在する可能性が高く、それはまったく問題ありません。ゲートは、管理者ダッシュボードの表示など、モデルやリソースに関連しないアクションに最も適しています。対照的に、特定のモデルまたはリソースのアクションを認可する場合は、ポリシーを使用する必要があります。

<a name="gates"></a>
## ゲート

<a name="writing-gates"></a>
### ゲートの作成

> [!WARNING]
> ゲートは、Laravelの認可機能の基本を学ぶための優れた方法です。ただし、堅牢なLaravelアプリケーションを構築するときは、[ポリシー](#creating-policies)を使用して認可ルールを整理することを検討する必要があります。

ゲートは、指定アクションをユーザーが実行する権限があるかを判定するシンプルなクロージャです。通常、ゲートは`Gate`ファサードを使用して、`App\Providers\AppServiceProvider`クラスの`boot`メソッド内で定義します。ゲートは常に最初の引数としてユーザーインスタンスを受け取り、オプションで関連するEloquentモデルなどの追加の引数を受け取ることもできます。

以下の例では、ユーザーが特定の`App\Models\Post`モデルを更新できるかどうかを判断するためのゲートを定義します。ユーザーの`id`と、投稿を作成したユーザーの`user_id`を比較することで、このゲートは可否を判定します。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * 全アプリケーションサービスの初期起動処理
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

コントローラと同様に、ゲートもクラスコールバック配列を使用して定義できます。

```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * 全アプリケーションサービスの初期起動処理
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

<a name="authorizing-actions-via-gates"></a>
### アクションの認可

ゲートを使用してアクションを認可するには、`Gate`ファサードが提供する`allows`か`denies`メソッドを使用する必要があります。現在認証済みのユーザーをこれらのメソッドに渡す必要はないことに注意してください。Laravelは自動的にユーザーをゲートクロージャに引き渡します。認可が必要なアクションを実行する前に、アプリケーションのコントローラ内でゲート認可メソッドを呼び出すのが一般的です。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * 指定ポストの更新
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if (! Gate::allows('update-post', $post)) {
            abort(403);
        }

        // ポストの更新処理…

        return redirect('/posts');
    }
}
```

現在認証済みユーザー以外のユーザーがアクションの実行を許可されているかを確認する場合は、`Gate`ファサードで`forUser`メソッドを使用します。

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // ユーザーはポストを更新できる
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // ユーザーはポストを更新できない
}
```

`any`または`none`メソッドを使用して、一度に複数のアクション認可を確認できます。

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // ユーザーはポストを更新及び削除できる
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // ユーザーはポストを更新及び削除できない
}
```

<a name="authorizing-or-throwing-exceptions"></a>
#### 認可または例外を投げる

アクションの認可を試みて、ユーザーが指定アクションを実行できない場合、自動的に`Illuminate\Auth\Access\AuthorizationException`を投げたい場合、`Gate`ファサードの`authorize`メソッドを使用します。`AuthorizationException`のインスタンスは、Laravelによって自動的に403 HTTPレスポンスに変換されます。

```php
Gate::authorize('update-post', $post);

// アクションは認可された
```

<a name="gates-supplying-additional-context"></a>
#### 追加コンテキストの提供

アビリティを認可するためのゲートメソッド（`allows`、`denis`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）と認可[Bladeディレクティブ](#via-blade-templates)（`@can`、`@cannot`、`@canany`）は、２番目の引数として配列を取れます。これらの配列要素は、パラメータとしてゲートクロージャに渡され、認可を決定する際の追加のコンテキストに使用できます。

```php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // ユーザーはポストを作成できる
}
```

<a name="gate-responses"></a>
### ゲートのレスポンス

これまで、単純な論理値を返すゲートのみ見てきました。しかし、エラーメッセージなどのより詳細なレスポンスを返したい場合もあります。これには、ゲートから`Illuminate\Auth\Access\Response`を返してください。

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('You must be an administrator.');
});
```

ゲートから認可レスポンスを返した場合でも、`Gate::allows`メソッドは単純なブール値を返します。ただし、`Gate::inspect`メソッドを使用して、ゲートから返される完全な認可レスポンスを取得できます。

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // アクションは認可されている
} else {
    echo $response->message();
}
```

アクションが認可されていない場合に`AuthorizationException`を投げる`Gate::authorize`メソッドを使用すると、認可レスポンスが提供するエラーメッセージがHTTPレスポンスへ伝播されます。

```php
Gate::authorize('edit-settings');

// アクションは認可されている
```

<a name="customizing-gate-response-status"></a>
#### HTTPレスポンスステータスのカスタマイズ

ゲートがアクションを拒否すると、`403` HTTPレスポンスを返します。しかし場合により、別のHTTPステータスコードを返すほうが、便利なことがあります。認可チェックに失敗したときに返すHTTPステータスコードは、`Illuminate\Auth\Access\Response`クラスの`denyWithStatus`静的コンストラクタを使用してカスタマイズできます。

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyWithStatus(404);
});
```

`404`レスポンスによるリソースの隠蔽はウェブアプリケーションでは常套手段なため、使いやすいように`denyAsNotFound`メソッドを提供しています。

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

<a name="intercepting-gate-checks"></a>
### ゲートチェックの割り込み

特定のユーザーにすべての機能を付与したい場合があります。`before`メソッドを使用して、他のすべての認可チェックの前に実行するクロージャを定義できます。

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`before`クロージャがnull以外の結果を返した場合、その結果を認可チェックの結果とみなします。

`after`メソッドを使用して、他のすべての認可チェックの後に実行されるクロージャを定義できます。

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`after`クロージャが返す値は、ゲートまたはポリシーが`null`を返さない限り、認可チェックの結果を上書きしません。

<a name="inline-authorization"></a>
### インライン認可

時には、現在認証されているユーザーが、あるアクションを実行する認可を持っているかを、そのアクションに対応する専用のゲートを書かずに判断したいこともあるでしょう。Laravelでは、`Gate::allowIf`や`Gate::denyIf`メソッドを使い、「インライン」での認可チェックを行うことができます。インライン認可は、定義した["before"と"after"の認可フック](#intercepting-gate-checks)を実行しません。

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```

アクションが認可されていない場合や、現在認証されているユーザーがいない場合、Laravelは自動的に`Illuminate\Auth\Access\AuthorizationException`という例外を投げます。`AuthorizationException`のインスタンスは、Laravelの例外ハンドラが、自動的に403 HTTPレスポンスへ変換します。

<a name="creating-policies"></a>
## ポリシーの作成

<a name="generating-policies"></a>
### ポリシーの生成

ポリシーは、特定のモデルまたはリソースに関する認可ロジックを集めたクラスです。たとえば、アプリケーションがブログの場合、`App\Models\Post`モデルと投稿の作成や更新などのユーザーアクションを認可するためのPostモデルと対応する`App\Policies\PostPolicy`があるでしょう。

`make:policy`　Artisanコマンドを使用してポリシーを生成できます。生成するポリシーは`app/Policies`ディレクトリへ配置します。このディレクトリがアプリケーションに存在しない場合、Laravelが作成します。

```shell
php artisan make:policy PostPolicy
```

`make:policy`コマンドは、空のポリシークラスを生成します。リソースの表示、作成、更新、削除に関連するポリシーメソッドのサンプルを含んだクラスを生成する場合は、コマンドの実行時に`--model`オプションを指定します。

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### ポリシーの登録

<a name="policy-discovery"></a>
#### ポリシーの検出

デフォルトでは、モデルとポリシーが標準的なLaravelの命名規則に従っている限り、Laravelは自動的にポリシーを検出します。具体的にはモデルを含むディレクトリの上、またはそのディレクトリの中の`Policies`ディレクトリへ、ポリシーを置く必要があります。例えば、モデルは `app/Models`ディレクトリに配置し、ポリシーは `app/Policies`ディレクトリに配置します。この場合、Laravelは`app/Models/Policies`にあるポリシーをチェックし、次に`app/Policies`にあるポリシーをチェックします。さらに、ポリシー名はモデル名と一致し、最後に`Policy`を付ける必要があります。つまり、`User`モデルは、`UserPolicy`ポリシークラスに対応します。

独自のポリシー検出ロジックを定義したい場合は、`Gate::guessPolicyNamesUsing`メソッドを使用し、カスタムポリシー検出コールバックを登録してください。通常、このメソッドはアプリケーションの`AppServiceProvider`の`boot`メソッドから呼び出します。

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // 指定モデルのポリシークラス名を返す…
});
```

<a name="manually-registering-policies"></a>
#### ポリシーの手作業による登録

`Gate`ファサードを使用すると、アプリケーションの `AppServiceProvider`の`boot`メソッド内で、ポリシーと対応するモデルを手作業で登録できます。

```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * 全アプリケーションサービスの初期起動処理
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```

あるいは、モデルクラスに`UsePolicy`属性を指定して、そのモデルに対応するポリシーをLaravelへ通知することもできます。

```php
<?php

namespace App\Models;

use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;
use Illuminate\Database\Eloquent\Model;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

<a name="writing-policies"></a>
## ポリシーの作成

<a name="policy-methods"></a>
### ポリシーメソッド

ポリシークラスを登録したら、認可するアクションごとにメソッドを追加できます。例として、ある`App\Models\User`がある`App\Models\Post`インスタンスを更新できるかどうかを決定する`PostPolicy`で`update`メソッドを定義してみましょう。

`update`メソッドは引数として`User`と`Post`インスタンスを受け取り、そのユーザーが指定した`Post`を更新する権限があるかどうかを示す`true`または`false`を返す必要があります。したがって、この例では、ユーザーの`id`が投稿の`user_id`と一致することを確認しています。

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 指定された投稿をユーザーが更新可能か判定
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

ポリシーが認可するさまざまなアクションの必要に合わせ、ポリシーに追加のメソッドをどんどん定義できます。たとえば、`view`または`delete`メソッドを定義して、さまざまな`Post`関連のアクションを認可できますが、ポリシーメソッドには任意の名前を付けることができることを覚えておいてください。

Artisanコンソールを介してポリシーを生成するときに`--model`オプションを使用した場合、はじめから`viewAny`、`view`、`create`、`update`、`delete`、`restore`、`forceDelete`アクションのメソッドが用意されます。

> [!NOTE]
> すべてのポリシーはLaravel[サービスコンテナ](/docs/{{version}}/container)を介して解決されるため、ポリシーのコンストラクターで必要な依存関係をタイプヒントして、自動的に依存注入することができます。

<a name="policy-responses"></a>
### ポリシーのレスポンス

これまで、単純な論理値値を返すポリシーメソッドについてのみ説明してきました。しかし、エラーメッセージなどより詳細なレスポンスを返したい場合があります。これにはポリシーメソッドから`Illuminate\Auth\Access\Response`インスタンスを返してください。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 指定ポストがユーザーにより更新可能か判断
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('You do not own this post.');
}
```

ポリシーから認可レスポンスを返す場合でも、`Gate::allows`メソッドは単純な論理値を返します。ただし、`Gate::inspect`メソッドを使用して、ゲートが返す完全な認可レスポンスを取得できます。

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // アクションは認可されている
} else {
    echo $response->message();
}
```

アクションが認可されていない場合に`AuthorizationException`を投げる`Gate::authorize`メソッドを使用すると、認可レスポンスが提供するエラーメッセージがHTTPレスポンスへ伝播されます。

```php
Gate::authorize('update', $post);

// アクションは認可されている
```

<a name="customizing-policy-response-status"></a>
#### HTTPレスポンスステータスのカスタマイズ

ポリシーメソッドがアクションを拒否すると、`403` HTTPレスポンスを返します。しかし場合により、別のHTTPステータスコードを返すほうが、便利なことがあります。認可チェックに失敗したときに返すHTTPステータスコードは、`Illuminate\Auth\Access\Response`クラスの`denyWithStatus`静的コンストラクタを使用してカスタマイズできます。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 指定ポストがユーザーにより更新可能か判断
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyWithStatus(404);
}
```

`404`レスポンスによるリソースの隠蔽はウェブアプリケーションでは常套手段なため、使いやすいように`denyAsNotFound`メソッドを提供しています。

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 指定ポストがユーザーにより更新可能か判断
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### モデルのないメソッド

一部のポリシーメソッドは、現在認証済みユーザーのインスタンスのみを受け取ります。この状況は、`create`アクションを認可するばあいに頻繁に見かけます。たとえば、ブログを作成している場合、ユーザーが投稿の作成を認可されているかを確認したい場合があります。このような状況では、ポリシーメソッドはユーザーインスタンスのみを受け取る必要があります。

```php
/**
 * 指定ユーザーがポストを作成可能か判断
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### ゲストユーザー

デフォルトでは、受信HTTPリクエストが認証済みユーザーによって開始されたものでない場合、すべてのゲートとポリシーは自動的に`false`を返します。ただし、「オプションの」タイプヒントを宣言するか、ユーザーの引数定義で`null`のデフォルト値を指定することで、これらの認可チェックをゲートとポリシーに渡すことができます。

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 指定された投稿をユーザーが更新可能か判定
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### ポリシーフィルタ

ある特定のユーザーには、特定のポリシー内のすべてのアクションを認可したい場合があります。これには、ポリシーで「before」メソッドを定義します。`before`メソッドは、ポリシー上の他のメソッドの前に実行されるため、目的のポリシーメソッドが実際に呼び出される前にアクションを認可する機会に利用できます。この機能は、アプリケーション管理者にアクションの実行を許可するために最も一般的に使用されます。

```php
use App\Models\User;

/**
 * 事前認可チェックの実行
 */
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

特定のタイプのユーザー全員の認可チェックを拒否したい場合は、`before`メソッドから`false`を返してください。`null`を返す場合は、認可チェックはポリシーメソッドへ委ねられます。

> [!WARNING]
> ポリシークラスの`before`メソッドは、チェックしている機能の名前と一致する名前のメソッドがクラスに含まれていない場合は呼び出されません。

<a name="authorizing-actions-using-policies"></a>
## ポリシーを使用したアクションの認可

<a name="via-the-user-model"></a>
### ユーザーモデル経由

Laravelアプリケーションに含まれている`App\Models\User`モデルには、アクションを認可するための２つの便利なメソッド`can`と`cannot`が含まれています。`can`メソッドと`cannot`メソッドは、認可するアクションの名前と関連するモデルを受け取ります。たとえば、ユーザーが特定の`App\Models\Post`モデルを更新する権限を持っているかどうかを確認しましょう。通常、これはコントローラメソッド内で実行されます。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 指定ポストの更新
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // ポストの更新処理…

        return redirect('/posts');
    }
}
```

指定したモデルの[ポリシーが登録されている](#registering-policies)の場合、`can`メソッドは自動的に適切なポリシーを呼び出し、論理値の結果を返します。モデルにポリシーが登録されていない場合、`can`メソッドは、指定されたアクション名に一致するクロージャベースのゲートを呼び出そうとします。

<a name="user-model-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

一部のアクションは、モデルインスタンスを必要としない`create`などのポリシーメソッドに対応する場合があることに注意してください。このような状況では、クラス名を`can`メソッドに渡すことができます。クラス名は、アクションを認可するときに使用するポリシーを決定するために使用されます。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * ポスト作成
     */
    public function store(Request $request): RedirectResponse
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // ポストの作成処理…

        return redirect('/posts');
    }
}
```

<a name="via-the-gate-facade"></a>
### `Gate`ファサード経由

`AppModelsUser`モデルに用意している便利なメソッドに加え、`Gate`ファサードの`authorize`メソッドでいつでもアクションを認可できます。

`can`メソッドと同様に、このメソッドは、認可するアクションの名前とリレーションモデルを受け入れます。アクションが認可されていない場合、`authorize`メソッドは`Illuminate\Auth\Access\AuthorizationException`例外を投げ、Laravel例外ハンドラは自動的に403ステータスコードのHTTPレスポンスに変換します。

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * 指定ブログ投稿を更新
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', $post);

        // 現在のユーザーはブログ投稿を更新可能

        return redirect('/posts');
    }
}
```

<a name="controller-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

すでに説明したように、`create`などの一部のポリシーメソッドはモデルインスタンスを必要としません。このような状況では、クラス名を`authorize`メソッドに渡す必要があります。クラス名は、アクションを認可するときに使用するポリシーを決定するために使用されます。

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * 新規ブログポストの作成
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // 現在のユーザーはブログポストを作成できる

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### ミドルウェア経由

Laravelは、リクエストがルートやコントローラに到達する前にアクションを認可するミドルウェアを用意しています。デフォルトでは、`can`[ミドルウェアエイリアス](/docs/{{version}}/middleware#middleware-aliases)を使い、`Illuminate\Auth\Middleware\Authorize`ミドルウェアをルートへ指定できます。`can`ミドルウェアを使用して、ユーザーが投稿を更新するのを許可する例を見てみましょう：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーはポストの更新処理が可能
})->middleware('can:update,post');
```

この例では、`can`ミドルウェアに２つの引数を渡します。１つ目は認可するアクションの名前であり、２つ目はポリシーメソッドに渡すルートパラメータです。この場合、[暗黙のモデルバインディング](/docs/{{version}}/Routing#implicit-binding)を使用しているため、`App\Models\Post`モデルがポリシーメソッドに渡されます。ユーザーが特定のアクションを実行する権限を持っていない場合、ミドルウェアは403ステータスコードのHTTPレスポンスを返します。

これは簡単に、`can`メソッドを使い、`can`ミドルウェアをルートへ指定できます。

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 現在のユーザーはポストの更新処理が可能
})->can('update', 'post');
```

<a name="middleware-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

繰り返しますが、`create`のようないくつかのポリシーメソッドはモデルインスタンスを必要としません。このような状況では、クラス名をミドルウェアに渡すことができます。クラス名は、アクションを認可するときに使用するポリシーを決定するために使用されます。

```php
Route::post('/post', function () {
    // 現在のユーザーはポストを作成可能
})->middleware('can:create,App\Models\Post');
```

ミドルウェア定義の中で、クラス名全体を文字列で指定するのは面倒です。そのため、`can`メソッドを使って、`can`ミドルウェアをルートへ指定できます。

```php
use App\Models\Post;

Route::post('/post', function () {
    // 現在のユーザーはポストを作成可能
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### Bladeテンプレート経由

Bladeテンプレートを作成するとき、ユーザーが特定のアクションを実行する許可がある場合にのみ、ページの一部を表示したい場合があります。たとえば、ユーザーが実際に投稿を更新できる場合にのみ、ブログ投稿の更新フォームを表示したい場合があります。この状況では、`@can`および`@cannot`ディレクティブを使用できます。

```blade
@can('update', $post)
    <!-- 現在のユーザーは投稿を更新可能 -->
@elsecan('create', App\Models\Post::class)
    <!-- 現在のユーザーは新しい投稿を作成不可能 -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- 現在のユーザーは投稿を更新不可能 -->
@elsecannot('create', App\Models\Post::class)
    <!-- 現在のユーザーは新しい投稿を作成可能 -->
@endcannot
```

これらのディレクティブは、`@if`と`@unless`ステートメントを短く記述するための便利な短縮形です。上記の`@can`および`@cannot`ステートメントは、以下のステートメントと同等です。

```blade
@if (Auth::user()->can('update', $post))
    <!-- 現在のユーザーは投稿を更新可能 -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- 現在のユーザーは投稿を更新不可能 -->
@endunless
```

また、ユーザーが複数のアクションの実行を認可されているかを判定することもできます。これには、`@canany`ディレクティブを使用します。

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- 現在のユーザーは、投稿を更新、表示、削除可能 -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成可能 -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### モデルを必要としないアクション

他のほとんどの認可メソッドと同様に、アクションがモデルインスタンスを必要としない場合は、クラス名を`@can`および`@cannot`ディレクティブに渡すことができます。

```blade
@can('create', App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成可能 -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- 現在のユーザーは投稿を作成不可能 -->
@endcannot
```

<a name="supplying-additional-context"></a>
### 追加コンテキストの提供

ポリシーを使用してアクションを認可する場合、２番目の引数としてさまざまな認可関数とヘルパに配列を渡すことができます。配列の最初の要素は、呼び出すポリシーを決定するために使用され、残りの配列要素は、パラメータとしてポリシーメソッドに渡され、認可の決定を行う際の追加のコンテキストに使用できます。たとえば、追加の`$category`パラメータを含む次の`PostPolicy`メソッド定義について考えてみます。

```php
/**
 * 指定ポストがユーザーにより更新可能か判断
 */
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
```

認証済みユーザーが特定の投稿を更新できるか判断する場合、次のようにこのポリシーメソッドを呼び出すことができます。

```php
/**
 * 指定ブログポストの更新
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post): RedirectResponse
{
    Gate::authorize('update', [$post, $request->category]);

    // 現在のユーザーはブログポストを更新可能

    return redirect('/posts');
}
```

<a name="authorization-and-inertia"></a>
## 認可とInertia

認可は常にバリデータ上で処理しなくてはなりませんが、アプリケーションのUIを適切にレンダするために、フロントエンドアプリケーションに認可データを提供すると便利なことがあります。Inertiaを使用したフロントエンドへ認可情報を公開するために必要な規約をLaravelは定義してません。

しかし、LaravelのInertiaベースの[スターターキット](/docs/{{version}}/starter-kits)を使用する場合、アプリケーションはあらかじめ`HandleInertiaRequests`ミドルウェアを用意しています。このミドルウェアの`share`メソッド内から、アプリケーション内のすべてのInertiaページへ提供する共有データを返してください。この共有データは、ユーザーの認可情報を定義するための便利な場所として役立ちます。

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * デフォルトとして共有するプロップを定義
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}
```
