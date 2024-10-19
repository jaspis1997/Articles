---
title: "pnpmを使ってNode.jsのバージョン管理から開放される"
emoji: "🍣"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pnpm", "Nodejs"]
published: false
---

## はじめに

### この記事の目的

1. プロジェクト毎に異なるバージョンのNode.jsを使えるようにする
2. プロジェクトメンバーが使うNode.jsとパッケージマネージャーのバージョンを統一する
3. バージョンの切り替えを自動化する

### 対象読者

1. Node.jsを使ってHTMLやCSSのコーディングをしている人
2. Node.jsのバージョン管理を煩わしいと感じている人、もしくはこれからバージョン管理を始めようとしている人

### 背景

昨今、Node.jsはフロントエンドの開発環境としてその地位を獲得しています。SassのコンパイルやTypeScriptのビルドだけでなく、さまざまな用途で使用されています。

また、フロントエンドの開発を取り巻く環境は変化が早く、Node.jsのバージョンアップもそれに合わせるかのように半年ごとにメジャーアップデートが行われています。そのため、複数プロジェクトに参画している場合、プロジェクト毎に異なるバージョンのNode.jsが使われることも多くあります。

異なるバージョンのNode.jsで開発することで、特定メンバーの環境にだけ起こるトラブルが発生し、そのたびに調査と修正に時間が取られることもあります。この問題を解決するためにNVMやVoltaなどのNode.jsのバージョン管理システムを導入することも検討できます。
Voltaを使えば、プロジェクトで使用するNode.jsのバージョンを指定し、自動で切り替えることが実現できます。

この記事では、パッケージマネージャーでありながら、Node.jsのバージョン管理もでき、Voltaのように自動でNode.jsのバージョンを切り替える機能も備えた `pnpm` をご紹介します。

## pnpmについて

[Fast, disk space efficient package manager | pnpm](https://pnpm.io/ja/)

公式の紹介にもあるように、 `pnpm`はnpmの2倍早いです。 `node_modules` のインストール時間を大幅に短縮できます。

`node_modules` には、pakcageのハードリンクが作成され、複数のプロジェクトで使い回すことでディスク容量を削減しています。依存するパッケージに多くのディスク容量を奪われるストレスを軽減します。

pnpmはフラットではない `node_modules` を作成するため、任意のパッケージにアクセスできず、依存関係を厳格に管理できます。詳しく知りたい方は以下の記事がわかりやすく解説しています。

[pnpm の特徴](https://zenn.dev/azukiazusa/articles/pnpm-feature)

## 前提

OSはWindows 10です。

Node.jsの`v18.x`以上をインストールしている。特段の理由がなければ `v20.x`以上がオススメ。

古いバージョンのNode.jsを使っている場合は一度アンインストールし、公式インストーラーなどでlatest版のNode.jsをインストールしてください。後々、pnpmの設定を各プロジェクトに組み込むことで古い環境を使うことができます。

```msshell:powershell
node -v
```

Node.jsのバージョンがv18(v20)以上になっていればOKです。

## pnpmのインストール

今回は `npm` を使います。npmを使うのは最後になるかもしれません。

```msshell:powershell
npm install -g pnpm
```

バージョン指定する場合
```msshell:powershell
npm install -g pnpm@9.12.1
```

pnpmのバージョンは、指定する必要はありません。これについては後々詳しく説明します。

Node.jsのTIPSとして、原則グローバルインストールするべきではありません。グローバルインストールするパッケージはその必要性を十分に検討してからするのをオススメします。

```msshell:powershell
pnpm -v
```

pnpmのバージョンが表示されればOKです。今後、 `npm` の代わりに`pnpm`を使います。

## プロジェクトの環境設定

はじめに、プロジェクトを作ります。

```msshell:powershell
pnpm init
```

プロジェクトですでに`pnpm`を使っており、node_modulesをインストールする場合は以下のコマンドを実行してください。

```msshell:powershell
pnpm install
```

Node.jsのバージョン管理、pnpmのバージョン指定の順で解説します。

### Node.jsのバージョン管理

プロジェクトで使うNode.jsのバージョンを指定する設定を追加します。
```msshell:powershell
echo use-node-version=20.17.0>.npmrc
```
プロジェクトに `.npmrc`が作成されたと思います。これはgitでバージョン管理します。

`.npmrc`のuse-node-version属性があることで、`pnpm`は指定されたバージョンで実行するようになります。

ついでに、プロジェクトで使うNode.jsバージョンも記述しておきます。pnpmを使わない場合でも、明示的にバージョンを示しておくことでエラーを防ぐ効果が見込めます。この手順は必須ではないですが、しておくほうが良いでしょう。

```diff_json:package.json
{
  "name": "someproject",
  "version": "1.0.0",
-  "description": "",
-  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
+  "engines": {
+    "node": ">=20.17.0"
+  }
-  "keywords": [],
-  "author": "",
-  "license": "ISC"
}
```

要らないフィールドもついでに削除しておきましょう。

node_modulesは .gitignoreに記述しておきましょう。

```:.gitignore
node_modules
```

プロジェクトで使うNode.jsのバージョンを上げるには、 `.npmrc` のuse-node-version属性を書き換えるだけでOKです。実行時、必要なNode.jsをインストールします。

###

`.npmrc` に以下の属性を追記します。

```diff_ini:.npmrc
use-node-version=20.0.0
+ manage-package-manager-versions=true
```

次に、 `package.json` にプロジェクトで使う `pnpm`のバージョンを記述します。Node.jsのバージョン指定とは異なり、 `package.json` に書きます。

```diff_json:package.json
{
  "name": "someproject",
  "version": "1.0.0",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "engines": {
    "node": ">=20.17.0"
  },
+ "packageManager": "pnpm@9.12.1"
}
```

これで、`pnpm`のバージョンも、Node.jsと同様、指定されたバージョンを使うようになります。そのため、グローバルにインストールする `pnpm` のバージョンは任意でOKです。

これで、環境設定は完了です。次のセクションで動作確認をしていきましょう。

### 参考記事

[package.jsonの中にenginesの定義によってnodeバージョンを提示](https://zenn.dev/ianchen0419/articles/9c101f03f319a4)

## 動作確認

### Node.jsのバージョン切り替えを検証する

プロジェクトに `main.js` ファイルを追加し、実行中のNode.jsのバージョンを出力させてみます。

```diff_javascript:main.js
+ console.log(process.versions.node)
```

実行するために、`package.json`にコマンドを追加します。

```diff_json:package.json
{
  "name": "someproject",
  "version": "1.0.0",
  "scripts": {
-    "test": "echo \"Error: no test specified\" && exit 1"
+    "test": "node main"
  },
  "engines": {
    "node": ">=20.17.0"
  },
  "packageManager": "pnpm@9.12.1"
}
```
実行してみます。
```msshell:powershell
pnpm run test
```
`run` は省略しても実行できるみたいです。(今度調べてみます。)

```msshell:powershell
...省略
20.0.0
```

`.npmrc`に記述したバージョンが出力されていればOKです。

試しに、`.npmrc`のバージョンを変更し、本当にNode.jsのバージョンを切り替えられるか検証してみます。

```diff_ini:.npmrc
- use-node-version=20.0.0
+ use-node-version=8.0.0
manage-package-manager-versions=true
```

```msshell:powershell
pnpm run test
```
```msshell:powershell
 WARN  Unsupported engine: wanted: {"node":">=20.0.0"} (current: {"node":"8.0.0","pnpm":"9.10.0"})
...省略
8.0.0
```

無事、バージョンが切り替わりました。サポートされていないバージョンであることも警告してくれてます。

## まとめ

pnpmを使うことで、意識することなくNode.jsのバージョンを管理できるようになります。定期的なバージョンアップもプロジェクトに影響を与えることなくできます。
Node.jsに加え、パッケージマネージャーのバージョンを指定し、さらには厳格な依存関係により、従来起こり得た環境固有のエラーを少なくできます。調査とトラブル解決に時間を取られることがなくなり、システム開発に専念できるため生産性の向上を図ることができます。不要なストレスをなくし、良き開発者ライフに貢献できましたら幸いです。

## 参考

1. [package.jsonに"engines"を設定すると「このバージョンのNode.jsでしか動かない」を表明できる #JavaScript - Qiita](https://qiita.com/suin/items/994458418c737cc9c3e8)
2. [pnpmのメリット | pnpm](https://pnpm.io/ja/next/motivation)
3. [corepackを使っていた人こそpnpmをおすすめしたい！ #Node.js - Qiita](https://qiita.com/oekazuma/items/c972b1e071047f30d1ec)
4. [pnpmとcorepackを使ったNode.jsとpnpmのバージョン管理のまとめ | 株式会社一創](https://www.issoh.co.jp/tech/details/2765/)
