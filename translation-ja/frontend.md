# フロントエンド

- [イントロダクション](#introduction)
- [PHPの使用](#using-php)
    - [PHPとBlade](#php-and-blade)
    - [Livewire](#livewire)
    - [スターターキット](#php-starter-kits)
- [Using React or Vue](#using-react-or-vue)
    - [Inertia](#inertia)
    - [スターターキット](#inertia-starter-kits)
- [アセットの結合](#bundling-assets)

<a name="introduction"></a>
## イントロダクション

Laravelは、[ルーティング](/docs/{{version}}/routing)、[バリデーション](/docs/{{version}}/validation)、[キャッシュ](/docs/{{version}}/cache), [キュー](/docs/{{version}}/queues), [ファイルストレージ](/docs/{{version}}/filesystem)など、最新のウェブアプリケーション構築に必要となる全ての機能が提供されているバックエンド・フレームワークです。しかし、私たちはアプリケーションのフロントエンドを構築するための強力なアプローチを含む、美しいフルスタック体験を開発者に提供することも重要であると考えています。

Laravelでアプリケーションを構築する場合、フロントエンドの開発には主に２つの方法があります。どちらの方法を選択するかは、PHPを活用してフロントエンドを構築するか、VueやReactなどのJavaScriptフレームワークを使用するかにより決まります。以下では、こうした選択肢について説明し、あなたのアプリケーションに最適なフロントエンド開発のアプローチの情報を十分に得た上で、決定してもらえるようにします。

<a name="using-php"></a>
## PHPの使用

<a name="php-and-blade"></a>
### PHPとBlade

以前、ほとんどのPHPアプリケーションでは、リクエスト時にデータベースから取得したデータを表示するため、PHPの`echo`文を散りばめた単純なHTMLテンプレートを使用し、ブラウザでHTMLをレンダしていました。

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

このHTML表示の手法を使う場合、Laravelでは[ビュー](/docs/{{version}}/views)と[Blade](/docs/{{version}}/blade)を使用して実現できます。Bladeは非常に軽量なテンプレート言語で、データの表示や反復処理などに便利な、短い構文を提供しています。

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

この方法でアプリケーションを構築する場合、フォーム送信や他のページへの操作は、通常サーバから全く新しいHTMLドキュメントを受け取り、ページ全体をブラウザで再レンダします。現在でも多くのアプリケーションは、シンプルなBladeテンプレートを使い、この方法でフロントエンドを構築するのが、最も適していると思われます。

<a name="growing-expectations"></a>
#### 高まる期待

しかし、Webアプリケーションに対するユーザーの期待値が高まるにつれ、多くの開発者がよりダイナミックなフロントエンドを構築し、洗練した操作性を感じてもらう必要性を感じてきています。そのため、VueやReactといったJavaScriptフレームワークを用いた、アプリケーションのフロントエンド構築を選択する開発者もいます。

一方で、自分が使い慣れたバックエンド言語にこだわる人たちは、そのバックエンド言語を主に利用しながら、最新のWebアプリケーションUIの構築を可能にするソリューションを開発しました。たとえば、[Rails](https://rubyonrails.org/) のエコシステムでは、[Turbo](https://turbo.hotwired.dev/) や[Hotwire]、[Stimulus](https://stimulus.hotwired.dev/) などのライブラリ作成が勢いづいています。

Laravelのエコシステムでは、主にPHPを使い、モダンでダイナミックなフロントエンドを作りたいというニーズから、[Laravel Livewire](https://livewire.laravel.com)と[Alpine.js](https://alpinejs.dev/)が生まれました。

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com)は、VueやReactといったモダンなJavaScriptフレームワークで作られたフロントエンドのように、ダイナミックでモダン、そして生き生きとしたLaravelで動作するフロントエンドを構築するためのフレームワークです。

Livewireを使用する場合、レンダし、アプリケーションのフロントエンドから呼び出したり操作したりできるメソッドやデータを公開するUI部分をLivewire「コンポーネント」として作成します。例えば、シンプルな"Counter"コンポーネントは、以下のようなものです。

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

そして、このCounterに対応するテンプレートは、次のようになります。

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

ご覧の通り、Livewireでは、Laravelアプリケーションのフロントエンドとバックエンドをつなぐ、`wire:click`のような新しいHTML属性を書けます。さらに、シンプルなBlade式を使って、コンポーネントの現在の状態をレンダできます。

多くの人にとって、LivewireはLaravelでのフロントエンド開発に革命を起こし、Laravelの快適さを保ちながら、モダンでダイナミックなWebアプリケーションを構築することを可能にしました。通常、Livewireを使用している開発者は、[Alpine.js](https://alpinejs.dev/)も利用して、ダイアログウィンドウのレンダなど、必要な場合にのみフロントエンドにJavaScriptを「トッピング」します。

Laravelに慣れていない方は、[ビュー](/docs/{{version}}/views)と[Blade](/docs/{{version}}/blade)の基本的な使い方に、まず慣れることをお勧めします。その後、公式の[Laravel Livewireドキュメント](https://livewire.laravel.com/docs)を参照し、インタラクティブなLivewireコンポーネントでアプリケーションを次のレベルに引き上げる方法を学んでください。

<a name="php-starter-kits"></a>
### スターターキット

If you would like to build your frontend using PHP and Livewire, you can leverage our [Livewire starter kit](/docs/{{version}}/starter-kits) to jump-start your application's development.

<a name="using-react-or-vue"></a>
## Using React or Vue

Although it's possible to build modern frontends using Laravel and Livewire, many developers still prefer to leverage the power of a JavaScript framework like React or Vue. This allows developers to take advantage of the rich ecosystem of JavaScript packages and tools available via NPM.

However, without additional tooling, pairing Laravel with React or Vue would leave us needing to solve a variety of complicated problems such as client-side routing, data hydration, and authentication. Client-side routing is often simplified by using opinionated React / Vue frameworks such as [Next](https://nextjs.org/) and [Nuxt](https://nuxt.com/); however, data hydration and authentication remain complicated and cumbersome problems to solve when pairing a backend framework like Laravel with these frontend frameworks.

さらに、開発者は２つの別々のコードリポジトリを管理することになり、しばしばメンテナンス、リリース、デプロイメントを両方のリポジトリにまたがって調整する必要が起きます。こうした問題は克服できないものではありませんが、アプリケーションを開発する上で、生産的で楽しい方法とは思えません。

<a name="inertia"></a>
### Inertia

Thankfully, Laravel offers the best of both worlds. [Inertia](https://inertiajs.com) bridges the gap between your Laravel application and your modern React or Vue frontend, allowing you to build full-fledged, modern frontends using React or Vue while leveraging Laravel routes and controllers for routing, data hydration, and authentication — all within a single code repository. With this approach, you can enjoy the full power of both Laravel and React / Vue without crippling the capabilities of either tool.

LaravelアプリケーションにInertiaをインストールしたあとで、通常通りにルートとコントローラを記述します。しかし、コントローラからBladeテンプレートを返すのではなく、Inertiaページを返すようにします。

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * 指定ユーザーのプロフィールページを表示
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

An Inertia page corresponds to a React or Vue component, typically stored within the `resources/js/pages` directory of your application. The data given to the page via the `Inertia::render` method will be used to hydrate the "props" of the page component:

```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```

As you can see, Inertia allows you to leverage the full power of React or Vue when building your frontend, while providing a light-weight bridge between your Laravel powered backend and your JavaScript powered frontend.

#### サーバサイドレンダ

If you're concerned about diving into Inertia because your application requires server-side rendering, don't worry. Inertia offers [server-side rendering support](https://inertiajs.com/server-side-rendering). And, when deploying your application via [Laravel Cloud](https://cloud.laravel.com) or [Laravel Forge](https://forge.laravel.com), it's a breeze to ensure that Inertia's server-side rendering process is always running.

<a name="inertia-starter-kits"></a>
### スターターキット

If you would like to build your frontend using Inertia and Vue / React, you can leverage our [React or Vue application starter kits](/docs/{{version}}/starter-kits) to jump-start your application's development. Both of these starter kits scaffold your application's backend and frontend authentication flow using Inertia, Vue / React, [Tailwind](https://tailwindcss.com), and [Vite](https://vitejs.dev) so that you can start building your next big idea.

<a name="bundling-assets"></a>
## アセットの結合

BladeとLivewire、Vue／ReactとInertiaのどちらを使用してフロントエンドを開発するにしても、アプリケーションのCSSをプロダクション用アセットへバンドルする必要があるでしょう。もちろん、VueやReactでアプリケーションのフロントエンドを構築することを選択した場合は、コンポーネントをブラウザ用JavaScriptアセットへバンドルする必要があります。

Laravelは、デフォルトで[Vite](https://vitejs.dev)を利用してアセットをバンドルします。Viteは、ローカル開発において、ビルドが非常に速く、ほぼ瞬時のホットモジュール交換（HMR）を提供しています。[スターターキット](/docs/{{version}}/starter-kits)を含むすべての新しいLaravelアプリケーションでは、`vite.config.js`ファイルがあり、軽量なLaravel Viteプラグインがロードされ、LaravelアプリケーションでViteを楽しく使用できるようにしています。

The fastest way to get started with Laravel and Vite is by beginning your application's development using [our application starter kits](/docs/{{version}}/starter-kits), which jump-starts your application by providing frontend and backend authentication scaffolding.

> [!NOTE]
> LaravelでViteを活用するための詳細なドキュメントは、[アセットバンドルとコンパイルに関する専用のドキュメント](/docs/{{version}}/vite)を参照してください。
