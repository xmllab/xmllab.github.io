---
layout: post
title: 礎によるRailsアプリのデプロイテンプレート作成
date: 2015-09-07 18:48:02 +0900
comments: false
categories: 
---

礎(いしずえ)はRuby on Railsで開発したアプリケーションのステージング環境構築を支援します。

礎でできること
-------

下記のミドルウェアをインストールまたは設定するためのItamae cookbookテンプレートを作成します。

* Nginx
* MariaDB(10.1) Server
* Redis
* RVM
* Capistrano
* Monit
* Munin
* NTP
* OpenSSH Server

※デプロイ先はCentOS6/CentOS7のみ確認できています

デプロイテンプレートはItamaeとCapistranoで作成されるため、作成後に好みに応じて編集することができます。
また、MuninやMonitがモニタリングするためのWeb画面のセットアップまで行うため、稼働状況の確認までできます。
redisおよびSidekiqの監視も含めているため、ActiveJob + Sidekiq + Redisにも対応します。


礎のインストール
--------

1.Gemfileに下記を追記する
```
group :development do
  gem 'itamae-plugin-recipe-rvm', github: 'tk-hamaguchi/itamae-plugin-recipe-rvm', require: false
  gem 'ishizue', github: 'tk-hamaguchi/ishizue'
end
```

2.bundle installでインストールし、gereratorでishizueをinstallする
```
bundle install
bundle exec rails g ishizue:install
```

3.構築後の環境にログインするための公開鍵をmisc/ssh_keysフォルダにコピーする
```
cp ~/.ssh/id_rsa.pub misc/ssh_keys
```

これで準備は完了です。


礎を試す
--------

デプロイテンプレートにはVagrantfileも含まれているため、それを使ってCentOS7のboxにインストールしてみます

1.作業ディレクトリをmisc/itamaeに移し、仮想マシンを作成します
```
cd misc/itamae/
vagrant up
```

仮想マシンはVagrantfileの中に定義されている通り192.168.33.10のIPで上がってきます。上がってこなかった場合は、後述する別ホストへのデプロイを試してください。

2.Itamaeを使ってインストールを実施します
```
itamae ssh -h 192.168.33.10 -u root --ohai roles/staging.rb --node-yaml=nodes/staging.yml
 INFO : Starting Itamae...
root@192.168.33.10's password:
```

仮想マシンのパスワードは『vagrant』が設定されています。実行に失敗する場合は何度かリトライしてみてください。-l debug 付与するとデバックログが出ます。

3.実行終了後はWebサーバーからのアクセスが可能になります。

Main: http://192.168.33.10

Monitoring: http://192.168.33.10:8282


別ホストへのデプロイ
--------

192.168.33.10以外にデプロイする場合は、下記の手順を踏んでください

1.Capistranoの設定ファイルを編集する。
```
vi misc/capistrano/config/deploy/staging.rb
        server 'YOUR_SERVER_IP', user: 'deploy', roles: %w{app db web}
```

2.その後Itamaeの実行時IPを変更先のIPに変更して実行する
```
cd misc/itamae/
itamae ssh -h YOUR_SERVER_IP -u root --ohai roles/staging.rb --node-yaml=nodes/staging.yml
```

Railsアプリのアップデートに追従する
--------

Rails アプリをバージョンアップした時にはcapistranoの実行のみで対応可能です
```
cd misc/capistrano
cap staging deploy
```

http://192.168.33.10:8282 については、適宜Nginxの設定を書き直してください



