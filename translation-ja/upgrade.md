# アップグレードガイド

- [11.xから12.0へのアップグレード](#upgrade-12.0)

<a name="high-impact-changes"></a>
## 影響度の高い変更

<div class="content-list" markdown="1">

- [依存パッケージの更新](#updating-dependencies)
- [Laravelインストーラのアップデート](#updating-the-laravel-installer)

</div>

<a name="low-impact-changes"></a>
## 影響度の低い変更

<div class="content-list" markdown="1">

- [Carbon3](#carbon-3)
- [並列処理結果のインデックスマッピング](#concurrency-result-index-mapping)
- [ImageバリデーションからSVGを除外](#image-validation)
- [複数スキーマのデータベース調査](#multi-schema-database-inspecting)
- [リクエストのネストした配列のマージ](#nested-array-request-merging)

</div>

<a name="upgrade-12.0"></a>
## 11.xから12.0へのアップグレード

#### アップグレード見積もり時間：５分

> [!NOTE]
> 私たちは、互換性のない変更を可能な限りすべて文書化するよう努力しています。これらの互換性のない変更の一部はフレームワークの不明瞭な部分であり、こうした変更の一部が実際にあなたのアプリケーションに影響を与える可能性があります。時間を節約したいですか？[Laravel Shift](https://laravelshift.com/) を使用すると、アプリケーションのアップグレードを自動化できます。

<a name="updating-dependencies"></a>
### 依存パッケージのアップデート

**影響の可能性： 高い**

アプリケーションの`composer.json`ファイルにある、以下の依存パッケージを更新してください。

<div class="content-list" markdown="1">

- `laravel/framework`を`^12.0`へ

</div>

<a name="carbon-3"></a>
#### Carbon3

**影響の可能性： 低い**

[Carbon2.x](https://carbon.nesbot.com/docs/)のサポートを削除しました。すべてのLaravel 12アプリケーションは[Carbon 3.x](https://carbon.nesbot.com/docs/#api-carbon-3)を必須にしました。

<a name="updating-the-laravel-installer"></a>
### Laravelインストーラのアップデート

新しいLaravelアプリケーションを作成するためにLaravelインストーラCLIツールを使用している場合は、Laravel12.xと[新しいLaravelスターターキット](https://laravel.com/starter-kits)に対応するため、インストーラを更新する必要があります。`composer global require`でLaravelインストーラをインストールした場合は、`composer global update`でインストーラを更新してください。

```shell
composer global update laravel/installer
```

PHPとLaravelを`php.new`でインストールしている場合は、お使いのオペレーティングシステムにあった`php.new`インストールコマンドを再実行するだけで、最新バージョンのPHPとLaravelインストーラをインストールすることができます：

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# 管理者として実行
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

または、[Laravel Herd](https://herd.laravel.com)バンドルのLaravelインストーラを使用している場合は、Herdインストールを最新リリースへ更新してください。

<a name="concurrency"></a>
### 並列処理

<a name="concurrency-result-index-mapping"></a>
#### 並列処理結果のインデックスマッピング

**影響の可能性： 低い**

連想配列を用い、`Concurrency::run`メソッドを呼び出すと、並行処理の結果を関連するキーと一緒に返すようになりました。

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

<a name="database"></a>
### データベース

<a name="multi-schema-database-inspecting"></a>
#### 複数スキーマのデータベース調査

**影響の可能性： 低い**

`Schema::getTables()`、`Schema::getViews()`、`Schema::getTypes()`メソッドは、すべてのスキーマの結果をデフォルトで含むようになりました。引数へ`schema`を渡すと、指定したスキーマの結果のみを取得できます。

```php
// 全スキーマの全テーブル
$tables = Schema::getTables();

// `main`スキーマの全テーブル
$table = Schema::getTables(schema: 'main');

// 'main'と'blog'スキーマの全テーブル
$table = Schema::getTables(schema: ['main', 'blog']);
```

`Schema::getTableListing()`メソッドは、スキーマ修飾されたテーブル名をデフォルトで返すようになりました。必要に応じ、`schemaQualified`引数を渡して動作を変更できます。

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$table = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$table = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

`db:table`と`db:show`コマンドは、PostgreSQLやSQL Serverと同様に、MySQL、MariaDB、SQLiteのすべてのスキーマの結果を出力するようになりました。

<a name="requests"></a>
### リクエスト

<a name="nested-array-request-merging"></a>
#### リクエストのネストした配列のマージ

**影響の可能性： 低い**

`$request->mergeIfMissing()`メソッドは、「ドット」記法を使用してネストした配列データをマージできるようになりました。これまでこのメソッドに依存して「ドット」記法バージョンのキーを含むトップレベル配列キーを作成している場合は、この新しい挙動を考慮してアプリケーションを調整する必要があるかもしれません。

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

<a name="validation"></a>
### バリデーション

<a name="image-validation"></a>
#### ImageバリデーションからSVGを除外

`image`バリデーションルールは、デフォルトでSVG画像を許可しなくなりました。もし、`image`ルールでSVGを許可したい場合は、明示的に許可する必要があります。

```php
use Illuminate\Validation\Rules\File;

'photo' => 'required|image:allow_svg'

// もしくは
'photo' => ['required', File::image(allowSvg: true)],
```

<a name="miscellaneous"></a>
### その他

さらに、`laravel/laravel`の[GitHubリポジトリ](https://github.com/laravel/laravel)の変更点をご覧になることをお勧めします。これらの変更の多くは必須ではありませんが、これらのファイルをあなたのアプリケーションへ同期させておくとよいでしょう。これらの変更の一部はこのアップグレードガイドでカバーしていますが、設定ファイルやコメントの変更など、他の変更はカバーしていません。[GitHub比較ツール](https://github.com/laravel/laravel/compare/11.x...12.x)を使えば、簡単に変更点を確認でき、どのアップデートがあなたにとって重要なのか選ぶことができます。