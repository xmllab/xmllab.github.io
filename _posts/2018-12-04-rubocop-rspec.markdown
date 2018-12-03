---
layout: post
title: "rubocopでRSpecの書き方もチェック"
date: 2018-12-04 00:15:36 +0900
comments: false
categories: [Ruby, rubocop, RSpec]
---

Rubyは１つのことを実行するにも様々な書き方ができることが特徴の言語です。これは他の言語からRubyへ移ってくる際に学習コストを下げて参入障壁を下げる効果もありますが、複数名でソースコードを共有してプロダクトを開発する場合は、このメリットがデメリットに転じるケースがあります。

複数名でコードを共有するときに各々が思い思いの書き方をした場合、他の開発者が書いたコードを参考にするために読む場合や、コードレビューを実施するタイミングで自分の書き方に再解釈し直すプロセスを経る必要があり、可読性が下がってしまいます。最悪の場合はコード内容の誤認を誘発するケースがあります。これにより、コードレビューやプルリク承認時のコード確認に時間がかかったり見落としが起きることが起きてきます。

そこで、そういった可読性や誤認識といったところから来る開発効率の低下を防ぐため、一定のコーディング規約を定めてそれを遵守することが、全体的な効率化や品質向上につながります。

その一定のコーディング規約としてよく利用されるのが、下記のルールです。

* [Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)
* [Rails Style Guide](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)
* [RSpec Style Guide](https://github.com/rubocop-hq/rspec-style-guide/blob/master/README.md)

昔からある[Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)に加え、昨今ではRailsやRSpecに対する標準ルール的なものも規定されつつあります。

「いやいや、テストコードまで縛られるのはちょっと・・・」と思う方もいるかもしれませんが、テストコード自体の正当性もプルリクやレビューの時に確認するものですし、もしそこがしっかり書けていなかった場合、その後の試験に対する信頼性を揺るがすことにもなりかねません。そのため、実はプロダクトコードよりも試験コードを重要視している人もいます。

・・・とは言っても、毎回全てのコードを目で見て、人手でチェックすることは非現実的なため、昨今ではツールを使ってコーディング規約の遵守状況のチェックをします。

ツールとしては[rubocop](https://github.com/rubocop-hq/rubocop)を使ってプロジェクトの全体を評価するといったことがデファクトスタンダードになってきていますが、これで評価できるのは上記の[Ruby Style Guide](https://github.com/fortissimo1997/ruby-style-guide/blob/japanese/README.ja.md)と[Rails Style Guide](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)の一部のみとなっています。


「では[RSpec Style Guide](https://github.com/rubocop-hq/rspec-style-guide/blob/master/README.md)に対応しているか確認するには、すべて人がチェックする必要があるのか？」


答えは、Noです。


これについても既にツールが出てきていて、[rubocop](https://github.com/rubocop-hq/rubocop)を拡張する形で実現されています。

* [rubocop-rspec](https://github.com/rubocop-hq/rubocop-rspec)

これは`Gemfile`に追加して`bundle install`した後、`.rubocop.yml`に追記するか、`rubocop`コマンド実行時に引数で指定するかの２パターンで実行することができます。もちろんポリシーの有効・無効や、オプションの指定は`.rubocop.yml`に合わせて記載することができます。

今回は`.rubocop.yml`に追記する方法を紹介します。


1. あなたのRailsプロジェクトにあるGemfileに下記を追記します。（rubocopのバージョンは、各プロジェクトで使用するバージョンを決めて固定してください）
    ```ruby
    group :development do
      gem 'rubocop', '0.60.0'
      gem 'rubocop-rspec'
    end
    ```
2. `.rubocop.yml`を規定の設定から取得します
    ```shell
    cp $(bundle show rubocop)/config/default.yml .rubocop.yml
    ```
3. その後、`.rubocop.yml`の最初に下記を追記します
    ```yaml
    require: rubocop-rspec
    
    ```
4. 後はいつも通りrubocopを実行します
    ```shell
    bundle exec rubocop -R -D -c .rubocop.yml
    ```


これでRSpecも含めたコーディング規約チェックができます。

このうちプロジェクトで不要なチェックがあった場合、`.rubocop.yml`で無効化します。
最初から作るプロジェクトの場合はそのままでも問題ないかもしれませんが、今回実験したプロジェクトでは`.rubocop.yml`下記のような設定を施しています。

```yaml
RSpec/NestedGroups:
  Enabled: false

RSpec/ContextWording:
  Enabled: false

RSpec/ExampleWording:
  Enabled: false

RSpec/MessageChain:
  Enabled: false

RSpec/NamedSubject:
  Enabled: false

RSpec/MultipleExpectations:
  Enabled: false

RSpec/ExampleLength:
  Max: 10
```

ただ、この設定はあくまでも現段階の一例でしかなく、プロジェクトメンバーの力量によって値をチューニングしながら適用していく必要があります。

今年は色々あってrubocopのサンプルをいくつか作ったので、もしよければ活用してみてください。

https://github.com/tk-hamaguchi/docs/tree/master/rubocop%E3%81%AB%E3%83%81%E3%82%A7%E3%83%83%E3%82%AF
