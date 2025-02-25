# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel12](#laravel-12)

<a name="versioning-scheme"></a>
## バージョニング規約

Laravelとファーストパーティパッケージは、[セマンティックバージョニング](https://semver.org)にしたがっています。メジャーなフレームのリリースは、毎年（第１四半期に）リリースします。マイナーとパッチリリースはより頻繁に毎週リリースします。マイナーとパッチリリースは、**決して**ブレーキングチェンジを含みません

アプリケーションやパッケージからLaravelフレームワークやそのコンポーネントを参照する場合、Laravelのメジャーリリースにはブレークチェンジが含まれるため、常に`^11.0`のようなバージョン制約を使用する必要があります。しかし、常に１日かからずに新しいメジャーリリースへアップデートできるように、私たちは努めています。

<a name="named-arguments"></a>
#### 名前付き引数

[名前付き引数](https://www.php.net/manual/ja/functions.arguments.php#functions.named-arguments)は、Laravelの下位互換性ガイドラインの対象外です。Laravelコードベースを改善するために、必要に応じて関数の引数の名前を変更することもできます。したがって、Laravelメソッドを呼び出すときに名前付き引数を使用する場合は、パラメータ名が将来変更される可能性があることを理解した上で、慎重に行う必要があります。

<a name="support-policy"></a>
## サポートポリシー

Laravelのすべてのリリースは、バグフィックスは１８ヶ月、セキュリティフィックスは２年です。Lumenのようなその他の追加ライブラリでは、最新のメジャーリリースのみでバグフィックスを受け付けています。また、[Laravelがサポートする](/docs/{{version}}/database#introduction)データベースのサポートについても確認してください。

<div class="overflow-auto">

| バージョン | PHP (*)   | リリース             | バグフィックス期日   | セキュリティ修正期日 |
| ---------- | --------- | -------------------- | -------------------- | -------------------- |
| 9          | 8.0 - 8.2 | ２０２２年２月８日   | ２０２３年８月８日   | ２０２４年２月６日   |
| 10         | 8.1 - 8.3 | ２０２３年２月１４日 | ２０２４年８月６日   | ２０２５年２月４日   |
| 11         | 8.2 - 8.4 | ２０２４年３月１２日 | ２０２５年９月３日   | ２０２６年３月１２日  |
| 12         | 8.2 - 8.4 | ２０２５年２月２４日 | ２０２６年８月１３日  | ２０２７年２月４日   |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>End of life</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>Security fixes only</div>
    </div>
</div>

(*) 対応PHPバージョン

<a name="laravel-12"></a>
## Laravel 12

Laravel12では、Laravel11.xでの改善を引き継ぎ、上流の依存関係を更新し、React、Vue、Livewire用の新しいスターターキットを導入しました。これにはユーザー認証に[WorkOSのAuthKit](https://authkit.com)を使用するオプションも含まれます。スターターキットのWorkOS選択肢はソーシャル認証、パスキー、SSOをサポートしています。

<a name="minimal-breaking-changes"></a>
### 最低限のブレイキングチェンジ

このリリースサイクルでは、ブレイキングチェンジを最小限に抑えることに重点を置いてきました。その代わりに、既存のアプリケーションを壊さないようなクオリティ・オブ・ライフの改善を年間を通して継続的に行うことに専念してきました。

そのため、Laravel12のリリースは、既存の依存関係をアップグレードするための、比較的マイナーな「メンテナンスリリース」となっています。このことから、ほとんどのLaravelアプリケーションは、アプリケーションコードを変更することなく、Laravel12にアップグレードできます。

<a name="new-application-starter-kits"></a>
### 新しいアプリケーションスターターキット

Laravel12では、React、Vue、Livewire用の新しい[アプリケーションスターターキット](/docs/{{version}}/starter-kits)を導入しました。ReactとVueのスターターキットは、Inertia2、TypeScript、[shadcn/ui](https://ui.shadcn.com)、Tailwindを利用し、Livewireのスターターキットは、Tailwindベースの[Flux UI](https://fluxui.dev)コンポーネントライブラリとLaravel Voltを利用します。

React、Vue、Livewireのスターターキットはすべて、Laravelの組み込み認証システムを利用し、ログイン、登録、パスワードリセット、メール認証などを提供します。さらに、各スターターキットへ[WorkOS AuthKit](https://authkit.com)搭載バージョンを導入し、ソーシャル認証、パスキー、SSOサポートを提供します。WorkOSは、月間アクティブユーザー数100万人までのアプリケーションに無料で認証を提供しています。

新しいアプリケーションスターターキットの導入に伴い、Laravel BreezeとLaravel Jetstreamは追加アップデートを受けられなくなりました。

新しいスターターキットを使い始めるには、[スターターキットのドキュメント](/docs/{{version}}/starter-kits)を確認してください。