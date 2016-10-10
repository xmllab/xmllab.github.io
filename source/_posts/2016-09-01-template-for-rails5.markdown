---
layout: post
title: "Bootstrap4＋Rails5用アプリケーションテンプレート"
date: 2016-09-01 00:06:56 +0900
comments: false
categories: 
---

5月にRails5がリリースされてだんだんgemもこなれてきつつあるので満を持してアプリケーションテンプレートを作ってみました。これはBootstrap4とDeviseの認証を持ったシンプルなRails5アプリケーションを生成します。

https://github.com/tk-hamaguchi/rails_templates

細かい話はWikiにまとめてあるので、今回は使い方とできることをば。

使い方
--------

今回は`hoge_app`という名前のアプリをnewしてみます

```
### 1. Rails new の時に-mでテンプレートアドレスを指定する
$ rails new hoge_app -d mysql -T --skip-bundle -m https://raw.github.com/tk-hamaguchi/rails_templates/master/rails5_template.rb

### 2. bootstrapを使う場合は途中の質問にyesを入力
      Do you want to use bootstrap? yes

### 3. 出来上がったアプリのフォルダに移動
$ cd hoge_app

###4. DBを作成してマイグレーション
$ bundle exec rake db:drop db:create db:migrate && RAILS_ENV=test rake db:migrate

### 5. サーバーを立ち上げる
$ rails s
```

※デフォルトでデータベースは自ホストのmysqlにソケット経由で接続する設定になっています。必要に応じて`config/database.yml`を編集してください

この５ステップでbootstrap4のRails5アプリケーションが立ち上がります

できたもの
--------

このアプリは下記の機能を持ったシンプルなアプリケーションです

* ログイン・ログアウト
* アカウント登録
* セッションタイムアウト
* ログイン情報の記憶
* メールからのパスワード再設定

各々の機能はcucumberでテストが書かれているため、下記のコマンドで内容は確認できます

```
$ bundle exec cucumber features/
```

また、docker-compose用のファイルも合わせて生成するため、Railsアプリのproductionモードも下記のコマンドで試すことができます
```
$ docker-compose up
```

この場合コンテナが上がっているdocker-machineのホストIPにhttpアクセスを行うことで、ブラウザで動作を確認することができます。

※終了する場合はCtrl+Cで抜けます

もしうまくいかない場合は、Wikiの環境構築ページも参照してみるとヒントがあるかもしれません

また、このテンプレートで生成されたアプリはデフォルトのrakeタスクでユニットテスト（rspec）、インテグレーションテスト(cucumber)、詳細設計書(yard)、コーディング規約確認(rubocop)、セキュリティスキャン(breakman)を一通り実行されるのでpush前に一通り確認することができます。

```
$ bundle exec rake
```


