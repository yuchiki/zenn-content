---
title: "vscode のリモートコンテナ機能を用いて、コード管理された開発環境を複数人で共有する"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vscode", "リモートコンテナ", "dotfiles", "docker"]
published: false
---

# 概要

vscode のリモートコンテナ機能を用いると、開発環境をdockerfile + 設定ファイルとして

# 前書き

# vscodeのリモートコンテナ機能とは

コンテナの中に開発環境を押し込んで、その中にディレクトリをマウントして開発するVSCodeの機能です。

## コンテナ内で開発するメリット


コンテナ内で開発することには以下のようなメリットがあります。

### local環境を汚さない
複数のプロジェクトを開発するにつれて、local マシンにはそのための様々なアプリ・設定が導入されていきます。この状態には以下のような欠点があります。
- 環境が汚れていき、なんのために入れた設定かわからなくなり、管理しきれなくなる
- 異なるプロジェクトで必要な設定・アプリ同士が衝突する
プロジェクトごとに異なるコンテナ内で開発することで、以上の問題は解消されます。
### 開発環境構築が容易で、再現性が高い
開発に必要なアプリ・設定がコンテナの形でまとめられているので、少ない労力で何度でも開発環境を再現できます。
### 開発者間での開発環境の差異が生まれにくい
開発者間で開発環境が異なっているために、チームでの開発や運用に支障をきたすことがあります。開発環境をコンテナ化することで、複数の開発者が同じ開発環境で開発できるようになります。

## VSCode の remote containersの便利機能

コンテナ内で開発する一般的な利点に加えて、vscodeの remote containersは以下の機能を備えています。

### localの.gitconfig設定をコンテナ内に引き継ぐ

コンテナ内で git commit などをしたとき、 localの名前やemail設定が引き継がれます。

### ssh keyがコンテナ内でも使える

コンテナ内でもコンテナ外と同じキーでgit pushなどができます。

### dotfilesが展開される

local側のsettings.json に設定した値を用いて、dotfilesをコンテナ内に展開します。
これによりコンテナ外でのシェル設定などがコンテナ内でも引き継げます、

### コンテナ内で使える設定・拡張を指定できる

#### 開発者間で共通の設定・拡張を導入したい

ディレクトリ内のdevcontainer.json で指定することで、そのディレクトリでremote containers機能を使ったときに自動で有効になるvscode拡張を指定できます。
これは、複数の開発者に同じvscode拡張を導入してもらいたいときに便利です。

#### 自分独自の設定・拡張を導入したい

localのsettings.jsonで指定することで、どのディレクトリでremote containers機能を使っても自動で有効になるvscode拡張を指定することもできます。キーバインドや見え方の拡張など、自分独自の設定を導入するのに便利です。

# 設定手順

## 前提条件

- docker が起動している。
- vscodeがインストールされている。

## 推奨条件
- vscodeの設定がsyncされている。
    - vscode本体の機能で設定のsyncがなされています。これを有効にしておくと、複数のマシンで同じ設定を共有できて、環境を再現しやすくなります。

## 最低限の設定

## local に remote development 拡張を入れる

VSCodeの拡張機能一覧から Remote Development を探し、導入します。

https://github.com/yuchiki/poll を gh repo fork してきて、code コマンドで開き、 remote containerとして開けることを確認します。

## 便利な設定

以下の設定を試すときは、 F1キーを押してコマンドパレットを開き、 remote-container > rebuild and reopen the container を試してコンテナを逐一再構築してみるとよさそうです。

### remote > containers: copy git config

設定画面で remote containers　と検索をかけると見つかる項目です。
これを有効にしておくと localの.gitconfigの値が コンテナ内に受け継がれます。

### remote > containers > default extensions

ここに拡張機能の識別子(ID) (拡張機能のタブで該当の拡張機能を開くと書いてあります) を入力すると、コンテナ内でもその拡張機能が有効になります。
vim キーバインドとか emacs キーバインドとか、インデントとか括弧の表示を変える機能とか入れている人にとっては便利かもしれません。


参考: [remote containers用に設定している拡張機能たち](https://growi.yuchiki.net/6272b37b9684dabde4acf44b)
### dotfiles repository

dotfilesのgit repositoryのURLを入力しておくと、そのrepositoryをpullしてきてその中の install.shを実行してくれます。
参考: https://github.com/yuchiki/dotfiles
