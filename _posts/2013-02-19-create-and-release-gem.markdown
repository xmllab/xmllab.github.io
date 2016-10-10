---
layout: post
title: "10stepで行うGemの作成からリリースまで"
date: 2013-02-19 15:25:36 +0900
comments: false
categories: [Ruby, Gem]
---


最近gemを作る機会がやたら多いので、これを機にGemの作り方をまとめがてら ちょっとしたGemを作ってRubyGemsで公開するための一連の流れをメモしていきます。


1. Gemのひな形を作る
---------------

まず最初にGemファイルのひな形を作ります。
※ hogeまたはHogeのところは適当に読み替えてください
※ コミットログ(-mの後の英語っぽいもの)も適宜書き換えてください

``` sh
$ bundle gem hoge
      create  hoge/Gemfile
      create  hoge/Rakefile
      create  hoge/LICENSE.txt
      create  hoge/README.md
      create  hoge/.gitignore
      create  hoge/hoge.gemspec
      create  hoge/lib/hoge.rb
      create  hoge/lib/hoge/version.rb
Initializating git repo in /tmp/hoge
```


2. ひな形を記憶する
---------------

もともとのひな形を記憶します

``` sh
$ cd hoge
$ git add .
$ git commit -a -m 'first commit.'
[master (root-commit) 8e2ae2c] first commit.
 8 files changed, 100 insertions(+), 0 deletions(-)
 create mode 100644 .gitignore
 create mode 100644 Gemfile
 create mode 100644 LICENSE.txt
 create mode 100644 README.md
 create mode 100644 Rakefile
 create mode 100644 hoge.gemspec
 create mode 100644 lib/hoge.rb
 create mode 100644 lib/hoge/version.rb
```


3. .gitignoreを追記する
---------------

不要なファイルをコミットしないように.gitignoreを編集します

まず下記を追記します。

``` text .gitignore
vendor/bundle
*.swp
```

次に下記を削除します

``` text .gitignore
Gemfile.lock
```

ここまできたらまたコミットします

``` sh
$ git commit -a -m 'Modified .gitignore'
[master 8e30bfb] Modified .gitignore
 1 files changed, 2 insertions(+), 1 deletions(-)
```


4. 依存ライブラリをインストール
---------------

Gemfileに対してテストに必要なライブラリ等を追記します

``` sh
$ vi Gemfile
```

``` ruby Gemfile
# 下記を追記
gem 'rake'
gem 'yard', require: false
gem 'redcarpet', require: false

group :test do
  gem 'rspec', '~>2.12.0'
  gem 'cucumber'
  gem 'aruba'
end
```

rvmを使っている場合はgemsetを作っておくことをお勧めします。

``` sh
$ rvm use 1.9.3@hoge --rvmrc --create --install
```

bundlerを使って各種gemをインストールします

``` sh
$ bundle install --path vendor/bundle
Fetching gem metadata from https://rubygems.org/.........
Fetching gem metadata from https://rubygems.org/..
Installing rake (10.0.3)
Installing ffi (1.4.0) with native extensions
Installing childprocess (0.3.8)
Installing builder (3.1.4)
Installing diff-lcs (1.1.3)
Installing json (1.7.7) with native extensions
Installing gherkin (2.11.6) with native extensions
Installing cucumber (1.2.1)
Installing rspec-expectations (2.12.1)
Installing aruba (0.5.1)
Using hoge (0.0.1) from source at /tmp/hoge
Installing redcarpet (2.2.2) with native extensions
Installing rspec-core (2.12.2)
Installing rspec-mocks (2.12.2)
Installing rspec (2.12.0)
Installing yard (0.8.4.1)
Using bundler (1.2.3)
Your bundle is complete! It was installed into ./vendor/bundle
```

ここまでできたらまたコミットします

``` sh
$ git add .
$ git commit -a -m 'Config dependency.'
```


5. 実際にビルドするテストを書く
---------------

実際にビルドするためのテストを書いてみます。 まずテストのためのディレクトリを作成します。

``` sh
$ mkdir -p features/{step_definitions,support}
```

次にcucumber用の環境設定ファイルを作成します

``` sh
$ vi features/support/env.rb
```
``` ruby features/support/env.rb
# encoding: utf-8
 
 
require 'rake'
require 'aruba/cucumber'
 
 
Before do
  @features_root = File.expand_path('../../../', __FILE__)
  @dirs          = [@features_root]
end
```

そしてfeatureとstepを作成します

``` sh
$ vi features/build.feature
```
``` cucumber features/build.feature
# language: ja
 
機能: Gemのビルド
 
  シナリオ: gemコマンドを使ったビルド
    前提 プロジェクトルートにいる
    かつ ディレクトリ"pkg"が削除されている
    もし rakeタスク"build"を実行する
    ならば 標準出力が下記の正規表現にマッチする:
      """
      hoge \d.\d.\d(.\w+)? built to pkg/hoge-\d.\d.\d(.\w+)?.gem
      """
    かつ 終了コードが"0"である
    かつ ディレクトリ"pkg"が存在している
    かつ 下記のファイルが存在している:
      |pkg/hoge-0.0.1.gem|
```

``` sh
$ vi features/step_definitions/gem_steps.rb
```
``` cucumber features/step_definitions/gem_steps.rb
# encoding: utf-8
 
 
前提 /^プロジェクトルートにいる$/ do
  `pwd`.strip.should == @features_root
end
 
前提 /^ディレクトリ"(.*?)"が削除されている$/ do |path|
  FileUtils.rm_rf path
  check_directory_presence([path], false)
end
 
もし /^rakeタスク"(.*?)"を実行する$/ do |task_name|
  run_simple "rake #{task_name}"
end
 
 
ならば /^終了コードが"(.*?)"である$/ do |exit_status|
  assert_exit_status(exit_status.to_i)
end
 
ならば /^標準出力が下記の正規表現にマッチする:$/ do |expected|
  assert_matching_output(expected, all_stdout)
end
 
ならば /^ディレクトリ"(.*?)"が存在して(いる|いない)$/ do |directory,presence|
  check_directory_presence([directory], (presence == "いる"))
end
 
ならば /^下記のファイルが存在している:$/ do |files|
  check_file_presence(files.raw.map{|file_row| file_row[0]}, true)
end
```

この時点でテストが失敗することを確認します。

``` sh
$ bundle exec cucumber features

# language: ja
機能: Gemのビルド

  シナリオ: gemコマンドを使ったビルド     # features/build.feature:5
    前提プロジェクトルートにいる         # features/step_definitions/gem_steps.rb:4
    かつディレクトリ"pkg"が削除されている  # features/step_definitions/gem_steps.rb:8
    もしrakeタスク"build"を実行する  # features/step_definitions/gem_steps.rb:13
      Exit status was 1 but expected it to be 0. Output:
      
      rake aborted!
      ERROR:  While executing gem ... (Gem::InvalidSpecificationException)
          "FIXME" or "TODO" is not a description
      ~/.rvm/gems/ruby-1.9.3-p385@hoge/gems/bundler-1.2.3/lib/bundler/gem_helper.rb:146:in `sh'
      ~/.rvm/gems/ruby-1.9.3-p385@hoge/gems/bundler-1.2.3/lib/bundler/gem_helper.rb:56:in `build_gem'
      ~/.rvm/gems/ruby-1.9.3-p385@hoge/gems/bundler-1.2.3/lib/bundler/gem_helper.rb:38:in `block in install'
      Tasks: TOP => build
      (See full trace by running task with --trace)
      
       (RSpec::Expectations::ExpectationNotMetError)
      ./features/step_definitions/gem_steps.rb:14:in `/^rakeタスク"(.*?)"を実行する$/'
      features/build.feature:8:in `もしrakeタスク"build"を実行する'
    ならば標準出力が下記の正規表現にマッチする: # features/step_definitions/gem_steps.rb:22
      """
      hoge \d.\d.\d(.\w+)? built to pkg/hoge-\d.\d.\d(.\w+)?.gem
      """
    かつ終了コードが"0"である         # features/step_definitions/gem_steps.rb:18
    かつディレクトリ"pkg"が存在している   # features/step_definitions/gem_steps.rb:26
    かつ下記のファイルが存在している:      # features/step_definitions/gem_steps.rb:30
      | pkg/hoge-0.0.1.gem |

Failing Scenarios:
cucumber features/build.feature:5 # Scenario: gemコマンドを使ったビルド

1 scenario (1 failed)
7 steps (1 failed, 4 skipped, 2 passed)
0m0.982s
```

エラーメッセージに出ている通りにhoge.gemspecにあるgem.descriptionとgem.summaryをGemの説明に書き換えます

``` sh
$ vi hoge.gemspec
```
``` ruby hoge.gemspec
  gem.description   = %q{Hoge hoge}
  gem.summary       = %q{Fuga fuga}
```

もう一度テストを実行し、正常に終了することを確認します。

``` sh
$ bundle exec cucumber features
# language: ja
機能: Gemのビルド

  シナリオ: gemコマンドを使ったビルド     # features/build.feature:5
    前提プロジェクトルートにいる         # features/step_definitions/gem_steps.rb:4
    かつディレクトリ"pkg"が削除されている  # features/step_definitions/gem_steps.rb:8
    もしrakeタスク"build"を実行する  # features/step_definitions/gem_steps.rb:13
    ならば標準出力が下記の正規表現にマッチする: # features/step_definitions/gem_steps.rb:22
      """
      hoge \d.\d.\d(.\w+)? built to pkg/hoge-\d.\d.\d(.\w+)?.gem
      """
    かつ終了コードが"0"である         # features/step_definitions/gem_steps.rb:18
    かつディレクトリ"pkg"が存在している   # features/step_definitions/gem_steps.rb:26
    かつ下記のファイルが存在している:      # features/step_definitions/gem_steps.rb:30
      | pkg/hoge-0.0.1.gem |

1 scenario (1 passed)
7 steps (7 passed)
0m0.967s
```

ここまで来たらまたコミットします

``` sh
$ git add .
$ git commit -a -m 'Add build features'
[master dd40ddf] Add build features
 4 files changed, 72 insertions(+), 2 deletions(-)
 create mode 100644 features/build.feature
 create mode 100644 features/step_definitions/gem_steps.rb
 create mode 100644 features/support/env.rb
```


6. rakeタスクを作成する
---------------

rakeでテストとドキュメンテーションができるようにタスクを書いて行きます。

まずタスクファイルを入れるためのディレクトリを作成します

``` sh
$ mkdir lib/tasks
```

次にcucumber用のタスクファイルを作成します

``` sh
$ vi lib/tasks/cucumber.rake
```
``` ruby lib/tasks/cucumber.rake
require 'cucumber'
require 'cucumber/rake/task'
 
 
Cucumber::Rake::Task.new(:features) do |t|
  t.cucumber_opts = "features --format pretty"
end
```

さらにyard用のタスクファイルを作成します

``` sh
$ vi lib/tasks/yard.rake
```
``` ruby lib/tasks/yard.rake
require 'yard'
require 'yard/rake/yardoc_task'
 
YARD::Rake::YardocTask.new do |t|
  t.files   = ['app/controllers/**/*.rb','app/helpers/**/*.rb', 'app/mailers/**/*.rb', 'app/models/**/*.rb', 'lib/**/*.rb']
  t.options = ['--no-private']
  t.options << '--debug' << '--verbose' if $trace
end
```

最後にRakefileに下記を追記します

``` ruby Rakefile
Dir.glob(File.expand_path('../lib/tasks/*.rake', __FILE__)).each { |f| load f }

task :default => [ :features, :yard ]
```

実際に追加したものが動くか確認します。 下記のようになれば成功です。

``` sh
$ bundle exec rake
~/.rvm/rubies/ruby-1.9.3-p385/bin/ruby -S bundle exec cucumber features --format pretty
# language: ja
機能: Gemのビルド

  シナリオ: gemコマンドを使ったビルド     # features/build.feature:5
    前提プロジェクトルートにいる         # features/step_definitions/gem_steps.rb:4
    かつディレクトリ"pkg"が削除されている  # features/step_definitions/gem_steps.rb:8
    もしrakeタスク"build"を実行する  # features/step_definitions/gem_steps.rb:13
    ならば標準出力が下記の正規表現にマッチする: # features/step_definitions/gem_steps.rb:22
      """
      hoge \d.\d.\d(.\w+)? built to pkg/hoge-\d.\d.\d(.\w+)?.gem
      """
    かつ終了コードが"0"である         # features/step_definitions/gem_steps.rb:18
    かつディレクトリ"pkg"が存在している   # features/step_definitions/gem_steps.rb:26
    かつ下記のファイルが存在している:      # features/step_definitions/gem_steps.rb:30
      | pkg/hoge-0.0.1.gem |

1 scenario (1 passed)
7 steps (7 passed)
0m1.105s
Files:           2
Modules:         1 (    1 undocumented)
Classes:         0 (    0 undocumented)
Constants:       1 (    1 undocumented)
Methods:         0 (    0 undocumented)
 0.00% documented
```

ここまでの作業を記録します

``` sh
$ git add .
$ git commit -a -m 'Add rake tasks.'
[master 47906c7] Add rake tasks.
 3 files changed, 24 insertions(+), 0 deletions(-)
 create mode 100644 lib/tasks/cucumber.rake
 create mode 100644 lib/tasks/yard.rake
```


7. ドキュメントを記述する
---------------

READMEやモジュール(Hoge)のYARDを必要に応じて記述します。

変更した後はその作業をコミットします。

``` sh
$ git commit -a -m 'Add Documents.'
[master 563a5b1] Add Documents.
 4 files changed, 33 insertions(+), 33 deletions(-)
 rewrite README.md (90%)
```


8. Rubygemsにサインアップします
---------------

Rubygemsのサイト( https://rubygems.org )に行き、右上にある”sign up”からユーザー登録をします

サインアップが完了したら、下記のコマンドを使ってAPIキーを取得します。

※REGISTED_HANDLEの部分は、登録したHandleを指定してください
※curlがインストールされていない場合はyum install curlでインストールしてください

``` sh
$ curl -u REGISTED_HANDLE https://rubygems.org/api/v1/api_key.yaml > ~/.gem/credentials
Enter host password for user 'REGISTED_HANDLE': 登録したパスワードを入力
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0    56    0    56    0     0     95      0 --:--:-- --:--:-- --:--:--     0
```

catした時に、rubygems_api_keyに32桁の半角英数が入っていれば成功です。

``` sh
$ cat ~/.gem/credentials
---
:rubygems_api_key: 1234567890abcdefghijklmnopqrstuv
```


9. RubyGemsで公開します
---------------

下記のコマンドを実行することで、rubygemsで作成したGemを公開することができます。

``` sh
$ bundle exec rake release
hoge 0.0.1.pre built to pkg/hoge-0.0.1.gem
Tagged v0.0.1
Pushed git commits and tags
Pushed hoge 0.0.1 to rubygems.org
```


10. 公開されたGemを確認する
---------------

公開されたGemを確認するには、RubyGemsのサイト右上にある自分の名前をクリックするか、 同じく右上にあるdashbordから確認することができます。

また、下記のコマンドでインストール可能になっています。

``` sh
$ gem install hoge
```

Gemはホントに小さいプログラムでも作成／登録できるので、ちょっとしたツールを作って試してみると良いかもしれません。 途中で出てきたbundlerやgit,yard,cucumber,arubaについてそのうちまとめて記事にします。
