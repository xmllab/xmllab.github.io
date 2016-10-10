---
layout: post
title: "Elasticsearchを使うRubyアプリのインテグレーションテスト"
date: 2016-09-22 16:44:50 +0900
comments: false
categories: 
---

ElasticsearchにデータをPOSTするアプリを作る時にcucumberのhookを使って立ち上げるやり方メモ。

やりたいこと
--------

* cucumberのテストを走らせる前にElasticsearchの環境をDockerコンテナを用いて準備する
* 準備した環境はelasticsearchのGemから使いやすいように環境変数ELASTICSEARCH_URLにセットする
* 既存のElasticsearchとも接続できるように、テスト実行時にELASTICSEARCH_URLがセットされている場合はDockerを使わず与えられた環境を利用する
* テスト終了時にはコンテナを破棄する

処理の全容
--------

処理自体はcucumberのhookを利用する形で`features/support/hooks.rb`に記載する

```ruby
require 'uri'
require 'cgi'
require 'docker'

$container_id = nil

AfterConfiguration do |config|
  if ENV['ELASTICSEARCH_URL'].nil? || ENV['ELASTICSEARCH_URL'].length == 0
    if ENV['DOCKER_HOST'].nil? || ENV['DOCKER_HOST'].length == 0
      raise 'Please set DOCKER_HOST or ELASTICSEARCH_URL'
    end

    Cucumber.logger.debug "Elasticsearch with Docker:\n"
    container = Docker::Container.create(
      'Image'      => 'elasticsearch:5',
      'Env'        => ['ES_JAVA_OPTS=-Xms512m -Xmx512m'],
      'Cmd'        => %w(-E bootstrap.ignore_system_bootstrap_checks=true),
      'HostConfig' => { 'PublishAllPorts' => true }
    )
    $container_id = container.id
    u = URI.parse(container.connection.url)
    Cucumber.logger.debug "  * Create Container[#{$container_id}] at #{u}.\n"

    container.start
    Cucumber.logger.debug "  * Container[#{$container_id}] starting...\n"

    100.times.each do
      break if container.streaming_logs(stdout: true) =~ /started$/
      sleep 1
    end
    Cucumber.logger.debug "  * Container[#{$container_id}] was started.\n"

    container = Docker::Container.get $container_id
    port = container.info['NetworkSettings']['Ports']['9200/tcp'][0]['HostPort']
    Cucumber.logger.debug "  * Binding port 9200/tcp to #{port}.\n"

    ENV['ELASTICSEARCH_URL'] = "#{u.host}:#{port}"
    Cucumber.logger.debug "  * Set ELASTICSEARCH_URL to #{ENV['ELASTICSEARCH_URL']}.\n\n"
  end
end

at_exit do
  unless $container_id.nil?
    Cucumber.logger.debug "\nElasticsearch with Docker:\n"
    ENV['ELASTICSEARCH_URL'] = nil
    container = Docker::Container.get $container_id
    Cucumber.logger.debug "  * Container[#{$container_id}] stopping...\n"
    container.stop
    Cucumber.logger.debug "  * Container[#{$container_id}] was stopped.\n"
    container.delete
    Cucumber.logger.debug "  * Container[#{$container_id}] was deleted.\n"
  end
end
```

解説：hookのタイミング
--------

Dockerコンテナの作成・削除のタイミングはcucumberのhookを利用します。

環境変数として`ELASTICSEARCH_URL`が設定されている場合は何もしません。

環境変数に`ELASTICSEARCH_URL`が設定されていない場合はDockerを利用しますが、環境変数に`DOCKER_HOST`が設定されていない場合はDockerも使えないと判断し、エラー終了します

``` ruby
AfterConfiguration do |config|
  if ENV['ELASTICSEARCH_URL'].nil? || ENV['ELASTICSEARCH_URL'].length == 0
    if ENV['DOCKER_HOST'].nil? || ENV['DOCKER_HOST'].length == 0
      raise 'Please set DOCKER_HOST or ELASTICSEARCH_URL'
    end

    {{コンテナ作成・起動とかの処理}}
  end
end

at_exit do
  {{コンテナの停止とか破棄の処理}}
end
```

参考： <https://github.com/cucumber/cucumber/wiki/Hooks>

解説：ログの出力
--------

ログは`Cucumber.logger`を使って出力しています。

```ruby
Cucumber.logger.debug "Elasticsearch with Docker:\n"
```

メッセージはcucumberのコマンド実行時に`-v`をつけることで表示されるようになります

```bash
$ bundle exec cucumber -v features/
Code:
  * features/support/env.rb
  * features/support/hooks.rb

Elasticsearch with Docker:
  * Create Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] at tcp://192.168.131.138:2376.
  * Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] starting...
  * Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] was started.
  * Binding port 9200/tcp to 32771.
  * Set ELASTICSEARCH_URL to 192.168.131.138:32771.

Features:

{{中略}}

Elasticsearch with Docker:
  * Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] stopping...
  * Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] was stopped.
  * Container[ae51d0a059672f600949813726ce83459930b8496a8d4d085ad4c1f543f0da11] was deleted.
```

解説：コンテナの作成
--------

コンテナは`elasticsearch:5`を利用します

コンテナ内の環境変数`ES_JAVA_OPTS`でJavaVMに割り当てるメモリを設定します。今回は512MBを当てています。

Elasticsearchの5.0.0-alpha5時点ではネットワークインターフェースをバインドする時にオプションを指定しないと立ち上がらなかったので、起動オプションに`bootstrap.ignore_system_bootstrap_checks`を設定します。

コンテナが公開しているポートは全て使用するため、`PublishAllPorts`を指定します。

最後に、試験終了後にコンテナを停止・破棄する必要があるため、コンテナIDを`$container_id`に記憶します。

```ruby
container = Docker::Container.create(
  'Image'      => 'elasticsearch:5',
  'Env'        => ['ES_JAVA_OPTS=-Xms512m -Xmx512m'],
  'Cmd'        => %w(-E bootstrap.ignore_system_bootstrap_checks=true),
  'HostConfig' => { 'PublishAllPorts' => true }
)
$container_id = container.id
```

参考： <https://www.elastic.co/blog/elasticsearch-5-0-0-alpha3-released>

解説：コンテナの起動
--------

作成したコンテナを`container.start`で起動します。

ただ、この関数が戻ったタイミングでは、まだコンテナ内でElasticsearchのプロセスが起動仕切っていません。そのため`container.streaming_logs`を使ってElasticsearchのプロセス起動ログを定期的に取得し、`started`が出力されるまで待ちます

```ruby
container.start
Cucumber.logger.debug "  * Container[#{$container_id}] starting...\n"
100.times.each do
  break if container.streaming_logs(stdout: true) =~ /started$/
  sleep 1
end
Cucumber.logger.debug "  * Container[#{$container_id}] was started.\n"
```

解説：ELASTICSEARCH_URLの生成
--------

ELASTICSEARCH_URLは`IPアドレス:ポート`の形式で設定する必要がある。この場合のIPアドレスはDockerが動いているホスト(docker-machine)のIPアドレスになり、ポートはdocker-machineの公開ポートになります。

* docker-machineのIPアドレスは`container.connection.url`から取得します。
* docker-machineの公開ポートはランダムで割り当てられる、改めてDockerから起動したコンテナの情報を取得し、対象となる公開ポートを取得します

上記２つを組み合わせてELASTICSEARCH_URLを生成し、環境変数に設定します。

```ruby
u = URI.parse(container.connection.url)
Cucumber.logger.debug "  * Create Container[#{$container_id}] at #{u}.\n"
```

```ruby
container = Docker::Container.get $container_id
port = container.info['NetworkSettings']['Ports']['9200/tcp'][0]['HostPort']
Cucumber.logger.debug "  * Binding port 9200/tcp to #{port}.\n"

ENV['ELASTICSEARCH_URL'] = "#{u.host}:#{port}"
Cucumber.logger.debug "  * Set ELASTICSEARCH_URL to #{ENV['ELASTICSEARCH_URL']}.\n\n"
```

参考： <http://www.rubydoc.info/gems/elasticsearch-transport#Setting_Hosts>

解説：コンテナの停止と削除
--------

`$container_id`にコンテナIDが記憶されている場合、既存のElasticsearchではなくDockerで作ったElasticsearchを利用しており、ELASTICSEARCH_URLに設定されているのはこのテストのみで利用するELASTICSEARCH_URLです。

そのためELASTICSEARCH_URLを破棄し、合わせて`$container_id`で取得したコンテナを停止、削除を行います

```ruby
unless $container_id.nil?
  Cucumber.logger.debug "\nElasticsearch with Docker:\n"
  ENV['ELASTICSEARCH_URL'] = nil
  container = Docker::Container.get $container_id
  Cucumber.logger.debug "  * Container[#{$container_id}] stopping...\n"
  container.stop
  Cucumber.logger.debug "  * Container[#{$container_id}] was stopped.\n"
  container.delete
  Cucumber.logger.debug "  * Container[#{$container_id}] was deleted.\n"
end
```

使ってみる
--------

実際に使用する場合、初めてコンテナを作成する時や、毎回のコンテナ停止に若干時間がかかるため、まずは`-v`オプションありで使ってみることをお勧めします。

```bash
$ bundle exec cucumber -v features/
```

また、このhookを使ったコードを下記で使用しています。

* <https://github.com/tk-hamaguchi/exes_poster>

  ↑ exceptionをいい感じにelasticsearchにpostしてくれるGem

