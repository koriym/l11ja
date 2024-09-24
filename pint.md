# Laravel Pint

- [はじめに](#introduction)
- [インストール](#installation)
- [Pintの実行](#running-pint)
- [Pintの設定](#configuring-pint)
    - [プリセット](#presets)
    - [ルール](#rules)
    - [ファイル / フォルダの除外](#excluding-files-or-folders)
- [継続的インテグレーション](#continuous-integration)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## はじめに

[Laravel Pint](https://github.com/laravel/pint) は、ミニマリスト向けのオピニオンベースのPHPコードスタイルフィックスツールです。PintはPHP-CS-Fixerの上に構築されており、コードスタイルをクリーンで一貫性のある状態に保つことを簡単にします。

Pintはすべての新しいLaravelアプリケーションに自動的にインストールされるため、すぐに使用を開始できます。デフォルトでは、Pintは設定を必要とせず、Laravelのオピニオンベースのコーディングスタイルに従ってコードスタイルの問題を修正します。

<a name="installation"></a>
## インストール

PintはLaravelフレームワークの最近のリリースに含まれているため、通常はインストールが不要です。ただし、古いアプリケーションの場合は、Composerを介してLaravel Pintをインストールできます：

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## Pintの実行

プロジェクトの`vendor/bin`ディレクトリにある`pint`バイナリを呼び出すことで、Pintにコードスタイルの問題を修正させることができます：

```shell
./vendor/bin/pint
```

特定のファイルやディレクトリに対してPintを実行することもできます：

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pintは更新するすべてのファイルの詳細なリストを表示します。Pintの変更についてさらに詳細を表示するには、Pintを呼び出す際に`-v`オプションを指定します：

```shell
./vendor/bin/pint -v
```

Pintにファイルを実際に変更せずにコードスタイルのエラーを検査させたい場合は、`--test`オプションを使用できます。コードスタイルのエラーが見つかった場合、Pintはゼロ以外の終了コードを返します：

```shell
./vendor/bin/pint --test
```

Gitに基づいて未コミットの変更があるファイルのみをPintに修正させたい場合は、`--dirty`オプションを使用できます：

```shell
./vendor/bin/pint --dirty
```

コードスタイルのエラーを修正するファイルがある場合にゼロ以外の終了コードで終了するようにPintに指示したい場合は、`--repair`オプションを使用できます：

```shell
./vendor/bin/pint --repair
```

<a name="configuring-pint"></a>
## Pintの設定

前述のように、Pintは設定を必要としません。ただし、プリセット、ルール、または検査対象のフォルダをカスタマイズしたい場合は、プロジェクトのルートディレクトリに`pint.json`ファイルを作成することで行うことができます：

```json
{
    "preset": "laravel"
}
```

また、特定のディレクトリから`pint.json`を使用したい場合は、Pintを呼び出す際に`--config`オプションを指定できます：

```shell
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### プリセット

プリセットは、コードスタイルの問題を修正するために使用される一連のルールを定義します。デフォルトでは、Pintは`laravel`プリセットを使用し、Laravelのオピニオンベースのコーディングスタイルに従って問題を修正します。ただし、Pintに`--preset`オプションを指定することで異なるプリセットを指定できます：

```shell
./vendor/bin/pint --preset psr12
```

また、プロジェクトの`pint.json`ファイルでプリセットを設定することもできます：

```json
{
    "preset": "psr12"
}
```

Pintが現在サポートしているプリセットは、`laravel`、`per`、`psr12`、`symfony`、および`empty`です。

<a name="rules"></a>
### ルール

ルールは、Pintがコードスタイルの問題を修正するために使用するスタイルガイドラインです。前述のように、プリセットはほとんどのPHPプロジェクトに最適な事前定義されたルールのグループであるため、通常は個々のルールについて心配する必要はありません。

ただし、必要に応じて、`pint.json`ファイルで特定のルールを有効または無効にしたり、`empty`プリセットを使用してルールを最初から定義したりできます：

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "braces": false,
        "new_with_braces": {
            "anonymous_class": false,
            "named_class": false
        }
    }
}
```

Pintは[PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer)の上に構築されています。したがって、プロジェクトのコードスタイルの問題を修正するために、そのルールを使用できます：[PHP-CS-Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator)。

<a name="excluding-files-or-folders"></a>
### ファイル / フォルダの除外

デフォルトでは、Pintは`vendor`ディレクトリ内のファイルを除くすべての`.php`ファイルを検査します。さらにフォルダを除外したい場合は、`exclude`設定オプションを使用して行うことができます：

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

特定の名前パターンを含むすべてのファイルを除外したい場合は、`notName`設定オプションを使用して行うことができます：

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

ファイルの正確なパスを指定してファイルを除外したい場合は、`notPath`設定オプションを使用して行うことができます：

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

<a name="continuous-integration"></a>
## 継続的インテグレーション

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Laravel Pintを使用してプロジェクトのリンティングを自動化するには、[GitHub Actions](https://github.com/features/actions)を設定して、新しいコードがGitHubにプッシュされるたびにPintを実行できます。まず、GitHubの**Settings > Actions > General > Workflow permissions**でワークフローに「Read and write permissions」を付与してください。次に、`.github/workflows/lint.yml`ファイルを以下の内容で作成します：

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.3]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          extensions: json, dom, curl, libxml, mbstring
          coverage: none

      - name: Install Pint
        run: composer global require laravel/pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: "Fixes coding style"
```

