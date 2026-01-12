# AI開発支援

- [イントロダクション](#introduction)
    - [なぜAI開発にLaravelが最適なのか？](#why-laravel-for-ai-development)
- [Laravel Boost](#laravel-boost)
    - [インストール](#installation)
    - [利用可能なツール](#available-tools)
    - [AIガイドライン](#ai-guidelines)
    - [ドキュメント検索](#documentation-search)
    - [IDE統合](#ide-integration)
- [カスタムBoostガイドライン](#custom-guidelines)
    - [プロジェクトガイドラインの追加](#adding-project-guidelines)
    - [パッケージガイドライン](#package-guidelines)

<a name="introduction"></a>
## イントロダクション

Laravelは、AI支援開発やエージェントによる開発において、最高のフレームワークとなる独自の地位を確立しています。[Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[OpenCode](https://opencode.ai)、[Cursor](https://cursor.com)、[GitHub Copilot](https://github.com/features/copilot)といったAIコーディングエージェントの台頭は、開発者のコード記述方法を一変させました。これらのツールは、これまでにないスピードで機能全体の生成、複雑な問題のデバッグ、コードのリファクタリングを実行できます。しかし、その効果はツールがあなたのコードベースをどれだけ深く理解しているかに大きく依存します。

<a name="why-laravel-for-ai-development"></a>
### なぜAI開発にLaravelが最適なのか？

Laravelの規約を重視する設計（オピニオンな規約）と明確に定義された構造は、AI支援開発に理想的な環境を提供します。開発者がAIエージェントにコントローラの追加を依頼すると、AIはそれをどこに配置すべきか正確に判断します。新しいマイグレーションが必要な場合も、命名規則やファイルの場所を予測できます。この一貫性により、より柔軟なフレームワークでAIツールが陥りがちな「推測」という作業を排除します。

ファイル構成にとどまらず、Laravelの表現力豊かな構文と包括的なドキュメントは、AIエージェントが正確で慣習に沿った（イディオマティックな）コードを生成するために必要なコンテキスト（文脈）を与えます。Eloquentのリレーションシップ、フォームリクエスト、ミドルウェアなどの機能は、エージェントが確実に理解し、再現できるパターンに従っています。その結果、AIが生成するコードは、汎用的なPHPスニペットを繋ぎ合わせたものではなく、熟練したLaravel開発者が書いたような品質になります。

<a name="laravel-boost"></a>
## Laravel Boost

[Laravel Boost](https://github.com/laravel/boost)は、AIコーディングエージェントとあなたのLaravelアプリケーションの橋渡しをします。BoostはMCP（Model Context Protocol）サーバであり、15以上の専門的なツールを備えています。これらのツールは、アプリケーションの構造、データベース、ルートなどに関する深い洞察をAIエージェントに提供します。Boostをインストールすれば、AIエージェントは汎用的なコードアシスタントから、あなたのアプリケーションを具体的に理解したLaravelエキスパートへと進化します。

Boostは3つの主要な機能を提供します。アプリケーションの検査や操作を行うためのMCPツール群、Laravelエコシステム向けに特別に作成した構成可能なAIガイドライン、そして17,000以上のLaravel固有のナレッジを含む強力なドキュメントAPIです。

<a name="installation"></a>
### インストール

Boostは、PHP8.1以上で動作するLaravel10、11、12のアプリケーションにインストールできます。まずは、開発用依存パッケージとしてBoostをインストールしてください。

```shell
composer require laravel/boost --dev
```

インストール後、対話型インストーラを実行します。

```shell
php artisan boost:install
```

インストーラがIDEとAIエージェントを自動検出し、プロジェクトに適した統合機能を選択できるようにします。Boostは、MCP対応エディタ用の`.mcp.json`や、AIのコンテキスト用ガイドラインファイルなどの必要な設定ファイルを生成します。

> [!NOTE]
> 各開発者が独自の環境を構成したい場合は、生成される`.mcp.json`、`CLAUDE.md`、`boost.json`などの設定ファイルを`.gitignore`へ安全に追加できます。

<a name="available-tools"></a>
### 利用可能なツール

Boostは、Model Context Protocol（MCP）を通じて、包括的なツールセットをAIエージェントへ公開します。これらのツールにより、エージェントはLaravelアプリケーションを深く理解し、操作できるようになります。

<div class="content-list" markdown="1">

- **アプリケーションのイントロスペクション** - PHPやLaravelのバージョンを照会し、インストール済みパッケージを一覧表示し、アプリケーションの設定や環境変数を検査します。
- **データベースツール** - 会話画面から離れることなく、データベーススキーマを検査し、読み取り専用クエリを実行し、データ構造を把握します。
- **ルート検査** - 登録されているすべてのルートを、関連するミドルウェア、コントローラ、パラメーターとともに一覧表示します。
- **Artisanコマンド** - 利用可能なArtisanコマンドとその引数を検出し、エージェントがタスクに最適なコマンドを提案・実行できるようにします。
- **ログ分析** - アプリケーションのログファイルを読み込んで分析し、問題のデバッグを支援します。
- **ブラウザログ** - Laravelのフロントエンドツールを使用した開発時に、ブラウザのコンソールログやエラーにアクセスします。
- **Tinker統合** - Laravel Tinkerを経由してアプリケーションのコンテキスト内でPHPコードを実行し、エージェントが仮説をテストして動作を検証できるようにします。
- **ドキュメント検索** - インストールされているパッケージのバージョンに合わせて調整された結果を用いて、Laravelエコシステムのドキュメントを検索します。

</div>

<a name="ai-guidelines"></a>
### AIガイドライン

Boostは、Laravelエコシステムのために特別に作られた包括的なAIガイドラインのセットを含みます。これらのガイドラインはAIエージェントへ、Laravelコードの慣用的な書き方、フレームワークの規約に従うこと、はまりがちな間違いの回避方法を教えます。ガイドラインは構成可能であり、バージョンを認識するため、エージェントは使用している正確なパッケージバージョンに適した指示を受け取ります。

ガイドラインはLaravel自体に加え、以下を含むLaravelエコシステムの16以上のパッケージで利用可能です。

<div class="content-list" markdown="1">

- Livewire (2.x, 3.x, 4.x)
- Inertia.js (ReactおよびVueバリアント)
- Tailwind CSS (3.xおよび4.x)
- Filament (3.xおよび4.x)
- PHPUnit
- Pest PHP
- Laravel Pint
- その他多数

</div>

`boost:install`を実行すると、Boostはアプリケーションが使用しているパッケージを自動的に検出し、関連するガイドラインをプロジェクトのAIコンテキストファイルに組み込みます。

<a name="documentation-search"></a>
### ドキュメント検索

Boostは、AIエージェントが17,000件を超えるLaravelエコシステムのドキュメントにアクセスできるようにする、強力なドキュメントAPIを備えています。一般的なWeb検索とは異なり、このドキュメントはインデックス化およびベクトル化されており、使用中の正確なパッケージバージョンに一致するようフィルタリングされています。

機能の仕組みを理解する必要がある場合、エージェントはBoostのドキュメントAPIを検索し、正確でバージョン固有の情報を受け取れます。これにより、AIエージェントが古いフレームワークバージョンの非推奨メソッドや構文を提案してしまうという、よくある問題を排除します。

<a name="ide-integration"></a>
### IDEとの統合

Boostは、Model Context Protocolをサポートする一般的なIDEやAIツールと統合しています。いくつかの一般的なエディタでBoostを有効にする方法は以下の通りです。

```text tab="Claude Code"
// torchlight! {"lineNumbers": false}
Boostは通常、自動的に検出されます。手作業でのセットアップが必要な場合：

1. プロジェクトディレクトリでターミナルを開きます。
2. 次を実行します: claude mcp add laravel-boost -- php artisan boost:mcp
```

```text tab=Cursor
// torchlight! {"lineNumbers": false}
1. コマンドパレットを開きます (Cmd+Shift+P または Ctrl+Shift+P)。
2. 「MCP: Open Settings」を選択します。
3. 「laravel-boost」オプションをオンにします。
```

```text tab="VS Code"
// torchlight! {"lineNumbers": false}
1. コマンドパレットを開きます (Cmd+Shift+P または Ctrl+Shift+P)。
2. 「MCP: List Servers」を選択します。
3. 「laravel-boost」に移動し、Enterキーを押します。
4. 「Start server」を選択します。
```

```text tab=PhpStorm
// torchlight! {"lineNumbers": false}
1. Shiftキーを2回押して「Search Everywhere」を開きます。
2. 「MCP Settings」を検索し、Enterキーを押します。
3. 「laravel-boost」の横にあるチェックボックスを有効にします。
4. 「Apply」をクリックします。
```

```text tab=Codex
// torchlight! {"lineNumbers": false}
Boostは通常、自動的に検出されます。手作業でのセットアップが必要な場合：

1. プロジェクトディレクトリでターミナルを開きます。
2. 次を実行します: codex mcp add -- php artisan boost:mcp
```

```text tab=Gemini
// torchlight! {"lineNumbers": false}
Boostは通常、自動的に検出されます。手作業でのセットアップが必要な場合：

1. プロジェクトディレクトリでターミナルを開きます。
2. 次を実行します: gemini mcp add laravel-boost -- php artisan boost:mcp
```

<a name="custom-guidelines"></a>
## カスタムBoostガイドライン

Boostの組み込みガイドラインはLaravelエコシステムを包括的にカバーしていますが、AIエージェントに対してプロジェクト固有の指示を追加したい場合もあると思います。

<a name="adding-project-guidelines"></a>
### プロジェクトガイドラインの追加

プロジェクトにカスタムガイドラインを追加するには、アプリケーションの`.ai/guidelines`ディレクトリに`.blade.php`または`.md`ファイルを作成してください。

```text
.ai/
└── guidelines/
    └── api-conventions.md
    ├── architecture.md
    ├── testing-standards.blade.php
```

`boost:install` を実行すると、これらのファイルが自動的に取り込まれます。これらのガイドラインを使用して、チームのコーディング規約、アーキテクチャ上の決定事項、ドメイン固有の用語、あるいはAIエージェントがプロジェクトのためにより良いコードを書くのに役立つその他のコンテキストを文書化してください。

<a name="package-guidelines"></a>
### パッケージガイドライン

Laravelパッケージをメンテナンスしていて、ユーザーへAIガイドラインを提供したい場合は、パッケージの`resources/boost/guidelines`ディレクトリへガイドラインを含めてください。

```text
resources/
└── boost/
    └── guidelines/
        └── core.blade.php
```

AIガイドラインには、パッケージの概要、必要なファイル構造や規約、および主要機能の使用方法（コマンド例やコードスニペットを含む）を記載してください。AIがユーザーのために正しいコードを生成できるよう、簡潔かつ実践的で、ベストプラクティスに焦点を当てた内容に保ってください。以下に例を示します。

```md
## パッケージ名

このパッケージは[機能の簡単な説明]を提供します。

### 機能

- 機能1: [明確で短い説明]。
- 機能2: [明確で短い説明]。使用例:

@verbatim
<code-snippet name="機能2の使い方" lang="php">
$result = PackageName::featureTwo($param1, $param2);
</code-snippet>
@endverbatim
```

ユーザーがあなたのパッケージを含むアプリケーションにBoostをインストールすると、Boostはガイドラインを自動的に検出し、ユーザーのAIコンテキストに含めます。これにより、パッケージ作成者は、AIエージェントがパッケージを適切に使用する方法を理解できるよう支援できます。
