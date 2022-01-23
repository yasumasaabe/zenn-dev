---
title: "MACを新調したので環境設定(laravel)"
emoji: "🎩"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["PHP","laravel","npm","Node.js"]
published: true
---

# はじめに
PCを新調し、環境設定等について備忘録的に調査したこと等をまとめておきます。
本記事についてアドバイスや質問がありましたらよろしくお願いします。

# 開発環境
- MacBook Pro
- mac OS：：Montery
- チップ：Apple M
- PHP:7.4.27
- laravel：Laravel Framework 6.20.17
- npm：6.14.11
- node.js：v14.15.5

# Homebrewをインストール
Macのためのパッケージ管理システムです。
パッケージとは実行ファイルや設定ファイル、ライブラリを1つのファイルとしてまとめているものを指します。
要するにまとめて利用するツールを一括で管理してしまおうというものです。
※windowsでは動作しません。

```
> /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

このままではパスが通っていないので設定ファイルにパス追加する必要があります。
OSがCatalinaよりデフォルトシェルがzshに切り替わっているので設定ファイルであるzshrcにパスを追加します。
※Homebrewがインストールした際にNextstepsで次アクションが記載されているので従ってやっても問題ないです。

```
# 設定ファイルを開く
> vi ~/.zshrc

# 設定ファイルにパスを追加
> export PATH=/opt/homebrew/bin:$PATH

# 設定ファイルの変更を環境へ反映させる
> source ~/.zshrc
```
Homebrewが使用できるか確認しましょう。問題なく以下のようにレスポンスがあれば成功です。
```
> brew –version
Homebrew 3.3.11
Homebrew/homebrew-core (git revision 1b82a5f6070; last commit 2022-01-22)
Homebrew/homebrew-cask (git revision 16d2faacef; last commit 2022-01-22)
```

# PHPをインストール
PHPは、数ある言語の中でも「Web開発」に特化しているのが特徴です。
Homebrewを利用してPHPをインストールします。
```

# Homebrewで検索
> brew search PHP

# 今回はphp7.4を指定してインストール
> brew install php@7.4

# 設定ファイルにパスを通す
> echo 'export PATH="/opt/homebrew/opt/php@7.4/bin:$PATH"' >> ~/.zshrc
> $ echo 'export PATH="/opt/homebrew/opt/php@7.4/sbin:$PATH"' >> ~/.zshrc

# 設定ファイルの変更を環境へ反映させる
> source ~/.zshrc

# 念の為PHPをリスタートし、再度ターミナルを開く
> brew services start php@7.4

# phpが使用できるか確認　以下の様になっていれば成功
> php --versinon
PHP 7.4.27 (cli) (built: Dec 16 2021 18:02:37) ( NTS )
Copyright (c) The PHP Group
Zend Engine v3.4.0, Copyright (c) Zend Technologies
    with Zend OPcache v7.4.27, Copyright (c), by Zend Technologies
```

## composerをインストール
composerとはphpのパッケージ管理システムです。
全く違いますが、概念的にはHomebrewと同じパッケージ管理システムです。
composerは詳しくないので後々まとめた記事を上げるつもりですので、ご了承ください🙇‍♂️
```
# Homebrewで検索(今後は省きます)
# composerをインストール
> brew install composer

# composerのバージョンを確認
> composer --version

# ただし、現在プロジェクトで利用しているcomposerのバージョンは1系のため変更
# バージョンは語尾で指定可能
> curl -sS https://getcomposer.org/installer | php -- --version=1.7.3

# 先ほどダウンロードしたファイルをcomposer配下へ移動
> mv composer.phar /opt/homebrew/opt/composer

# composerのバージョンを確認　以下の様になっていれば成功
> composer --version
Composer version 1.7.3 2018-11-01 10:05:06

# composerパッケージをインストール
# vendorファイルが作成される
> composer install
```

## laravelをインストール
laravelとはPHPのフレームワークです。
```
# Composerで専用インストーラをインストール
> composer global require "laravel/installer"

# laravelのコマンドを実行できるゆようにパスを通す
> vi ~/.zshrc
> export PATH="$PATH:/Users/<ユーザー名>/.composer/vendor/bin"

# 設定ファイルの変更を環境へ反映させる
> source ~/.zshrc

# プロジェクト配下にてサーバを立ち上げられるか確認
> php artisan serve
```

# nodebrewをインストール
nodebrewは、1つのマシンの中で複数のバージョンのNode.jsを管理するためのツールです。
開発するプロジェクトごとに利用するNode.jsのバージョンが異なる異なる場合に便利です。

```
# Homebrewで検索(今後は省きます)
# nodebrewのインストール
> brew install nodebrew

# nodebrewを実行
> nodebrew setup

# nodebrewがインストールされたか確認
> nodebrew --version
nodebrew 1.1.0
```

## Node.jsのインストール
Node.jsはプログラミング言語ではございません。Node.jsはJavaScript実行環境を指します。
簡単にいうとJavaScriptをサーバーサイドでも使うことが可能にするということです。
すなわちNode.jsを使うことでフロントエンド、サーバーサイドで使う言語を統一できるということです。

```
# 使用可能なnodebrewのバージョンを確認
> nodebrew ls-remote

# 任意バージョンのインストール
> nodebrew install-binary {任意のバージョン}

# 安定版のインストール
> nodebrew install-binary stable

# 上記で"-darwin-arm64.tar.gz"を含むエラーが出た場合※30分くらいかかる
> nodebrew compile {任意のバージョン}

#使用するバージョンの指定
> use {任意のバージョン}

#パスを通す
> echo 'export PATH=$HOME/.nodebrew/current/bin:$PATH' >> ~/.zshrc

# 設定ファイルの変更を環境へ反映させる
> source ~/.zshrc

#再起動後、指定したバージョンが表示されるか確認
> node --version
v16.13.2
```

### Node.j15未満を利用したい場合
M1チップよりNode.js15未満は対応しなくなったので正攻法ではインストールすることができません。
そのため利用するためにはRosetta2に頼るか、今回のようにnvmでインストールするしかありません。
nvmは、Node.jsのバージョンを切り替えて使うことを可能にするツールです。
上記を見たらわかると思いますが、nvmとnodebrewは機能は同じです。Macユーザーはどちらでもいいです。

```
# nvmが入っているか確認
> command -v nvm

# Homebrewで検索(今後は省きます)
# nvmインストール
> brew install nvm

# nvmのディレクトリを作成
> mkdir ~/.nvm

# パスを通す
> vi .zshrc
 export NVM_DIR="$HOME/.nvm"
 [ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && . "/opt/homebrew/opt/nvm/nvm.sh"  # This loads nvm
 [ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && . "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion

# 設定ファイルの変更を環境へ反映させる
> source ~/.zshrc

# 使用するnode.jsをインストール
> nvm install {任意のバージョン}

# 利用するバージョンを固定する
> nvm use {バージョン}

# 指定したバージョンが表示されるか確認
> node --version
v16.13.2
```

# 参考
- Homebrew
https://zenn.dev/sawao/articles/e7e90d43f2c7f9

- Composer
https://mebee.info/2021/02/20/post-29364/

- Node.js
https://mako-note.com/ja/install-node14-on-m1-mac/