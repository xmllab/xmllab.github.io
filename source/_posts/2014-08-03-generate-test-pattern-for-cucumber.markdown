---
layout: post
title: "サクッとテスト用の網羅パターンを生成する"
date: 2014-08-03 11:20:51 +0900
comments: false
categories: ruby cucumber integration test
---

テストを書く時にinputの組合せが複数あったとき、パターンを網羅した一覧を作ってくれる何かがあるとうれしいなー。それをcucumberのfeatureにそのままコピペしてガシガシとテスト書いてけたら楽だろうなぁと思ったので、Gem作ってみた。


1. できること
------

YAMLにパターン書いとけばテーブル作ってくれます。

<pre>
abc:
 - A
 - B
 - C

bool:
 - true
 - false

value:
 - 1
 - 10
 - 100
 - 1000
 - 1000000
</pre>
これが・・・



<pre>
| abc | bool  |   value |
| A   | true  |       1 |
| A   | true  |      10 |
| A   | true  |     100 |
| A   | true  |    1000 |
| A   | true  | 1000000 |
| A   | false |       1 |
| A   | false |      10 |
| A   | false |     100 |
| A   | false |    1000 |
| A   | false | 1000000 |
| B   | true  |       1 |
| B   | true  |      10 |
| B   | true  |     100 |
| B   | true  |    1000 |
| B   | true  | 1000000 |
| B   | false |       1 |
| B   | false |      10 |
| B   | false |     100 |
| B   | false |    1000 |
| B   | false | 1000000 |
| C   | true  |       1 |
| C   | true  |      10 |
| C   | true  |     100 |
| C   | true  |    1000 |
| C   | true  | 1000000 |
| C   | false |       1 |
| C   | false |      10 |
| C   | false |     100 |
| C   | false |    1000 |
| C   | false | 1000000 |
</pre>
こうなる。


あとはこれをcucumberとかのfeatureファイルにシナリオテンプレートとかで組み込んであげればそれっぽくなります。


2. 動作環境
------

Rubyがあれば動きます。cRubyの2.x系推奨だけど、たぶん1.xでも動くかも？？

3. インストール
-----

まだテスト書いてないので、RubyGemsのサイトには登録してないです。
なので生インストール。

``` sh
$ curl -O https://github.com/tk-hamaguchi/combination_calc/raw/master/dist/combination_calc-latest.gem
$ gem install combination_calc-latest.gem
```

4. つかってみる
------

例えば上記のようなパターンが書いてるYAMLをpattern.ymlとしておいておいた時、引数にファイルを指定すれば表が出力されます。

```
$ combination_calc pattern.yml
```

ね、簡単でしょ？

ソースはGithub（ https://github.com/tk-hamaguchi/combination_calc ）に上げてるので、
気が向いたら更新します。
