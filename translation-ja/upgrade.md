# アップグレードガイド

- [11.xから12.0へのアップグレード](#upgrade-12.0)

<a name="high-impact-changes"></a>
## 影響度の高い変更

<div class="content-list" markdown="1">

- [依存パッケージの更新](#updating-dependencies)
- [Laravelインストーラのアップデート](#updating-the-laravel-installer)

</div>

<a name="medium-impact-changes"></a>
## 影響度が中程度の変更

<div class="content-list" markdown="1">

- [モデルとUUIDv7](#models-and-uuidv7)

</div>

<a name="low-impact-changes"></a>
## 影響度の低い変更

<div class="content-list" markdown="1">

- [Carbon3](#carbon-3)
- [並列処理結果のインデックスマッピング](#concurrency-result-index-mapping)
- [コンテナクラスの依存解決](#container-class-dependency-resolution)
- [ImageバリデーションからSVGを除外](#image-validation)
- [ローカルファイルデスクのデフォルトルートパス](#local-filesystem-disk-default-root-path)
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

- `laravel/framework` を `^12.0` へ
- `phpunit/phpunit` を `^11.0` へ
- `pestphp/pest` を `^3.0` へ

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

<a name="authentication"></a>
### 認証

<a name="updated-databasetokenrepository-constructor-signature"></a>
#### `DatabaseTokenRepository`契約の使用法のアップデート

**影響の可能性： とても低い**

`Illuminate\Auth\Passwords\DatabaseTokenRepository`クラスのコンストラクタは、`$expires`パラメータを分単位ではなく秒単位で受け取るようになった。

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

<a name="container"></a>
### コンテナ

<a name="container-class-dependency-resolution"></a>
#### コンテナクラスの依存解決

**影響の可能性： 低い**

依存注入コンテナは、クラス・インスタンスを解決するときにクラス・プロパティのデフォルト値を尊重するようにしました。以前、デフォルト値なしでクラス・インスタンスを解決するためにコンテナに依存していた場合は、この新しい動作を考慮してアプリケーションを調整する必要があるかもしれません。

```php
class Example
{
    public function __construct(public ?Carbon $date = null) {}
}

$example = resolve(Example::class);

// <= 11.x
$example->date instanceof Carbon;

// >= 12.x
$example->date === null;
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
$tables = Schema::getTables(schema: 'main');

// 'main'と'blog'スキーマの全テーブル
$tables = Schema::getTables(schema: ['main', 'blog']);
```

`Schema::getTableListing()`メソッドは、スキーマ修飾されたテーブル名をデフォルトで返すようになりました。必要に応じ、`schemaQualified`引数を渡して動作を変更できます。

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$tables = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$tables = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

`db:table`と`db:show`コマンドは、PostgreSQLやSQL Serverと同様に、MySQL、MariaDB、SQLiteのすべてのスキーマの結果を出力するようになりました。

<a name="updated-blueprint-constructor-signature"></a>
#### `Blueprint`コンストラクタ使用法の更新

**影響の可能性： とても低い**

`Illuminate\Database\Schema\Blueprint`クラスのコンストラクタは最初の引数として、`Illuminate\Database\Connection`のインスタンスを期待するようにしました。

<a name="eloquent"></a>
### Eloquent

<a name="models-and-uuidv7"></a>
#### モデルとUUIDv7

**影響の可能性：　中程度**

`HasUuids`トレイトは、UUID仕様のバージョン7（順序付きUUID）と互換性のあるUUIDを返すようにしました。モデルのIDへ順序付きUUIDv4文字列を使い続けたい場合は、`HasVersion4Uuids`traitを使用してください。

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // [tl! remove]
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // [tl! add]
```

`HasVersion7Uuids`トレイトは削除しました。以前このトレイトを使用していた場合は、代わりに`HasUuids`トレイトを使用してください。

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

<a name="storage"></a>
### Storage

<a name="local-filesystem-disk-default-root-path"></a>
#### ローカルファイルデスクのデフォルトルートパス

**影響の可能性： 低い**

アプリケーションがファイルシステム設定で明示的に`local`ディスクを定義していない場合、Laravelはローカルディスクのルートをデフォルトで`storage/app/private`にするようになりました。以前のリリースでは、デフォルトは`storage/app`でした。そのため、`Storage::disk('local')`を呼び出すと、特に設定しない限り`storage/app/private`から読み込み、`storage/app/private`へ書き込みます。以前の動作に戻すには、手作業で`local`ディスクを定義し、希望するルートパスを設定します。

<a name="validation"></a>
### バリデーション

<a name="image-validation"></a>
#### ImageバリデーションからSVGを除外

**影響の可能性： 低い**

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
