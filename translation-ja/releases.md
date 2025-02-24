# リリースノート

- [バージョニング規約](#versioning-scheme)
- [サポートポリシー](#support-policy)
- [Laravel 12](#laravel-12)

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

Laravel 12 continues the improvements made in Laravel 11.x by updating upstream dependencies and introducing new starter kits for React, Vue, and Livewire, including the option of using [WorkOS AuthKit](https://authkit.com) for user authentication. The WorkOS variant of our starter kits offers social authentication, passkeys, and SSO support.

<a name="minimal-breaking-changes"></a>
### Minimal Breaking Changes

Much of our focus during this release cycle has been minimizing breaking changes. Instead, we have dedicated ourselves to shipping continual quality of life improvements throughout the year that do not break existing applications.

Therefore, the Laravel 12 release is a relatively minor "maintenance release" in order to upgrade existing dependencies. In light of this, most Laravel applications may upgrade to Laravel 12 without changing any application code.

<a name="new-application-starter-kits"></a>
### New Application Starter Kits

Laravel 12 introduces new [application starter kits](/docs/{{version}}/starter-kits) for React, Vue, and Livewire. The React and Vue starter kits utilize Inertia 2, TypeScript, [shadcn/ui](https://ui.shadcn.com), and Tailwind, while the Livewire starter kits utilizes the Tailwind based [Flux UI](https://fluxui.dev) component library and Laravel Volt.

The React, Vue, and Livewire starter kits all utilize Laravel's built-in authentication system to offer login, registration, password reset, email verification, and more. In addition, we are introducing a [WorkOS AuthKit](https://authkit.com) powered variant of each starter kit, offering social authentication, passkeys, and SSO support. WorkOS offers free authentication for applications up to 1 million monthly active users.

With the introduction of our new application starter kits, Laravel Breeze and Laravel Jetstream will no longer receive additional updates.

To get started with our new starter kits, check out the [starter kit documentation](/docs/{{version}}/starter-kits).

