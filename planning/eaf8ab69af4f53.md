# プランニング

スタート: Node.jsのバージョン管理に悩んでいる
メソッド: Node.jsとパッケージマネージャーのバージョン管理にpnpmを使う
ゴール: プロジェクト毎に異なるバージョンの環境を使い、開発を行えるようにする

## ターゲットは誰か

1. 複数のプロジェクトに参画しており、それぞれ異なるバージョンのNode.jsを使っている人
2. Node.jsの環境構築に再現性がなく、キッティングに時間が取られてしまう人
3. 非システムエンジニアの自分でも簡単にNode.jsのバージョン管理ができるようにしたいと考えている人

## この記事を読んだ読者に、次に何をしてほしいのか

pnpmを使うことで、Node.jsのバージョンを機にすることなく、プロジェクトの開発に専念してほしい

## この記事で解説しないこと

VoltaやnvmなどのNode.jsのバージョン管理システムについて

## 読者が解決したい問題

1. Node.jsのバージョンを意識せずに、プロジェクトの開発を行いたい

# Outline

1. はじめに
    1.  この記事の目的
    2.  対象読者
    3.  背景
2. pnpmとは
3. 前提
4. pnpmのinstall
5. プロジェクトの環境設定
    1. package.json
    2. .npmrc
6. 動作確認
    1. シンプルな構成での動作確認
    2. packageの追加
    3. ~~vite環境での動作確認~~
    4. browser-syncを使った動作確認
7. まとめ

# ソース

## 前提

Node.jsのv18.x以上がインストールされている

## pnpmのインストール

バージョン: 9.12.1
2024年10月現在、pnpmのlatestをインストールします。

環境はWindows 10,11です。
curlを使ってのインストールもできますが、Windows Defenderが実行可能ファイルをブロックすることがあるみたいです。
そのため、よほどの理由がない場合はnpmのグローバルインストールを推奨しているようです。
今回はnpmのグローバルインストールで行います。

```msshell:powershell
npm install -g pnpm@9.12.1
```
省略した場合はlatestがインストールできるので、基本的にバージョン指定は不要です。

インストールするNode.jsのバージョンは、最低18.x以上が必要です

```msshell:powershell
node -v
```

https://pnpm.io/ja/installation

## Node.jsのバージョン管理

今まではfnmやnvm,Voltaなどを使っていましたが、pnpmを使用することで基本的にはこれらのツールは不要になります。

## pnpmのメリット

1. ディスク容量の節約
2. 高速なインストール

## viteの実行

```msshell:powershell
pnpm run dev
```

## pnpm nodeの実行バージョン

```js:main.js
console.log(process.versions.node)
```

## pnpmの設定

```ini:.npmrc
use-node-version=16.16.0
manage-package-manager-versions=true
```
use-node-version
default: 未定義
タイプ: semver

プロジェクトの実行に必要なNode.jsのバージョンを指定します。pnpmは指定されたNode.jsのバージョンを自動的にインストールし、利用します。
`pnpm run` および `pnpm node` コマンドでは指定されたNode.jsのバージョンで実行します。

manage-package-manager-versions
default: false
タイプ: boolean

有効な場合、package.jsonの「packageManager」フィールドのpnpmのバージョンを自動でダウンロードし、実行します。
これにより、プロジェクトのメンバー間でpnpmのバージョンを統一でき、トラブルの発生を抑制できます。

```jsonc:package.json
{
//   ...省略
  "engines": {
    "node": ">=20.17.0"
  },
  "packageManager": "pnpm@9.10.0"
}
```

enginesを指定することで、指定したバージョン以外での `npm install` で警告を出すことができます。

# 参考資料

1. [package.jsonに"engines"を設定すると「このバージョンのNode.jsでしか動かない」を表明できる #JavaScript - Qiita](https://qiita.com/suin/items/994458418c737cc9c3e8)
2. [pnpmのメリット | pnpm](https://pnpm.io/ja/next/motivation)
3. [corepackを使っていた人こそpnpmをおすすめしたい！ #Node.js - Qiita](https://qiita.com/oekazuma/items/c972b1e071047f30d1ec)
4. [pnpmとcorepackを使ったNode.jsとpnpmのバージョン管理のまとめ | 株式会社一創](https://www.issoh.co.jp/tech/details/2765/)


## 補足的資料

1. [Node.jsのバージョン管理はVoltaに決定](https://zenn.dev/aiueda/articles/7dcecaa05d4f24)

# コンテンツ

## はじめに

1. この記事の目的

- プロジェクト毎に異なるNode.jsのバージョンを使う
- プロジェクトメンバー間でパッケージマネージャー、Node.jsのバージョンを統一し、トラブルをなくす
- 非エンジニアがNode.jsのバージョンを気にすることなく利用できるようにする

2. 対象読者
- Node.jsのバージョン管理を煩わしいと感じている人、もしくはこれからバージョン管理を始めようとしている人
- Node.jsを使ってHTMLやCSSのコーディングをしている人
- Node.jsの環境を管理している人

3. 背景

- Node.jsはツールとして、フロントエンドの開発によく使う
- Node.jsはアップデートが早い
- 複数のプロジェクトで使うNode.jsのバージョンが違うのはよくあること
- バージョンが異なることで、トラブルが起きやすい
- バージョン管理ツールのデファクト・スタンダードはよく変わる
- 非エンジニアでもNode.jsのバージョンを意識することなく利用できるようにしたい
- パッケージマネージャーでありながらNode.jsのバージョン管理もできるpnpmを紹介する

## pnpmについて

- ディスク容量の節約
- 高速なインストール
- Node.jsのバージョン管理
- 厳格な依存関係

## 前提

- Node.jsの `v18.0以上` 可能であれば `v20.15以上(latest)` をインストールしている
- プロジェクトで古いバージョンのNode.jsを使っている場合、一度アンインストールする
- 後々、pnpmの設定を各プロジェクトに組み込むことで古い環境を使うことができるため、問題なし
- 環境はwindows 10以上です

## pnpmのインストール

- npmでグローバルインストールします
- npmでのインストールは今後せず、pnpmでインストールする
- 原則、グローバルインストールはしない
- 自分しか使用しない場合でも、プロジェクトメンバーに相談して利用するか検討するほうがいい
- インストールするpnpmのバージョンは指定しなくてもOK
- シェルからインストールすることもできますが、今回はnpmを使ってインストールします

## プロジェクトの環境設定

- Node.jsのバージョン管理
    - オプション プロジェクトで使うNode.jsのバージョンを明確に示す
    - .gitignoreにnode_modulesが入っているか確認する
    - pnpmの環境では、node_modulesは絶対にバージョン管理してはならない
    - Node.jsのバージョンアップをする場合、.npmrcを書き換えるだけOK
- pnpmのバージョンを指定する
    - .npmrcにパッケージマネージャーの管理を設定を追加する
    - package.jsonにpnpmのバージョンを設定

## 動作確認

- 実行中のNode.jsのバージョンを表示する
    - main.jsに実行中のNode.jsのバージョンを出力する処理を書く

## まとめ

- pnpmを使うことで、意識することなくNode.jsのバージョンを切り替えることができる
- 定期的なバージョンアップは、.npmrcとpackage.jsonを書き換えることで対応できるようになる
- 高速なインストール、ディスクの節約、厳格な依存関係の管理は大きなメリット
- プロジェクトメンバー間でバージョンの統一ができるため、環境固有のトラブルを防ぐことができる
- トラブルの調査と解決に時間が取られず、開発に専念できるため生産性が上がる