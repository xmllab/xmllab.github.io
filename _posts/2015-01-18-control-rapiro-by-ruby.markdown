---
layout: post
title: "RubyからRAPIROを制御する"
date: 2015-01-18 17:59:52 +0900
comments: false
categories: ruby rapiro
---

RAPIRO( http://www.rapiro.com/ja/ )をRubyから制御してみたかったのでラッパー作ってみた。

https://github.com/tk-hamaguchi/rapiro_wrapper

このライブラリはシリアルコマンドの#M0〜#M9( http://wiki.rapiro.com/page/serial-command_ja/ )を使わずに直接サーボモーターやLEDの制御をする#Pコマンドを使ってRAPIROを動かすライブラリです。（※まだテストしてないのでRubyGemsに上げてません。）


1. できること
------

このライブラリを使うことでこんな感じでRAPIROの動きを制御できるようになります。

```
require 'rapiro_wrapper'


commander = RapiroWrapper::Commander.new('/dev/tty.usbserial-DA00HMG6')

sleep 20

commander.execute!([
  RapiroWrapper::Head.new( left: 30 ),
  RapiroWrapper::Waist.new( right: 30 ),
  RapiroWrapper::RightSholderRoll.new( up: 160 ),
  RapiroWrapper::RightSholderPitch.new( up: 20 ),
  RapiroWrapper::RightHandGrip.new( hold: 0 ),
  RapiroWrapper::LeftSholderRoll.new( up: 30 ),
  RapiroWrapper::LeftSholderPitch.new( up: 20 ),
  RapiroWrapper::LeftHandGrip.new( open: 0 ),
  RapiroWrapper::RightFootYaw.new( open: 30 ),
  RapiroWrapper::RightFootPitch.new( open: 5 ),
  RapiroWrapper::LeftFootYaw.new( close: 20 ),
  RapiroWrapper::LeftFootPitch.new( close: 0 ),
  RapiroWrapper::Eyes.new('#00ff00')
], 5)
```

RapiroWrapper::CommanderはシリアルI/Oを管理するクラスで、シリアルポートを引数に与えて初期化します。

その後インスタンスメソッドのexecute!()により各モーター、LEDに対して操作を行います。execute!の第一引数は各サーボモーターの状態を表すRapiroWrapper::ServoMotorのサブクラスの配列を指定し、第二引数は指定した位置に何秒後に達するかをミリ秒(1〜255)で指定します。

RapiroWrapper::ServoMotorのサブクラスと初期化時の引数は下記の通りで、各々の値は角度です。

<table>
  <tr>
    <th>クラス</th>
    <th>位置</th>
    <th>引数と値の範囲</th>
  </tr>
  <tr>
    <td> RapiroWrapper::Head</td>
    <td>首(頭)</td>
    <td>left:0〜90 または right:0〜90</td>
  </tr>
  <tr>
    <td> RapiroWrapper::Waist</td>
    <td>腰</td>
    <td>left:0〜90 または right:0〜90</td>
  </tr>
  <tr>
    <td> RapiroWrapper::RightSholderRoll</td>
    <td>右肩</td>
    <td>up:0〜180 (default:0)</td>
  </tr>
  <tr>
    <td> RapiroWrapper::RightSholderPitch</td>
    <td>右腕</td>
    <td>up:0〜90 (default:0)</td>
  </tr>
  <tr>
    <td> RapiroWrapper::RightHandGrip</td>
    <td>右手</td>
    <td>hold:0〜50 または open:0〜50</td>
  </tr>
  <tr>
    <td> RapiroWrapper::LeftSholderRoll</td>
    <td>左肩</td>
    <td>up:0〜180 (default:0)</td>
  </tr>
  <tr>
    <td> RapiroWrapper::LeftSholderPitch</td>
    <td>左腕</td>
    <td>up:0〜90 (default:0)</td>
  </tr>
  <tr>
    <td> RapiroWrapper::LeftHandGrip</td>
    <td>左手</td>
    <td>hold:0〜50 または open:0〜50</td>
  </tr>
  <tr>
    <td> RapiroWrapper::RightFootYaw</td>
    <td>右足の開き</td>
    <td>close:0〜60 または open:0〜60</td>
  </tr>
  <tr>
    <td> RapiroWrapper::RightFootPitch</td>
    <td>右足</td>
    <td>close:0〜40 または open:0〜40</td>
  </tr>
  <tr>
    <td> RapiroWrapper::LeftFootYaw</td>
    <td>左足の開き</td>
    <td>close:0〜60 または open:0〜60</td>
  </tr>
  <tr>
    <td> RapiroWrapper::LeftFootPitch</td>
    <td>左足</td>
    <td>close:0〜40 または open:0〜40</td>
  </tr>
</table>

また目のLEDについては下記のとおりです。

<table>
  <tr>
    <th>クラス</th>
    <th>引数と値の範囲</th>
  </tr>
  <tr>
    <td>RapiroWrapper::Eyes</td>
    <td>Webカラーの16進記述 または red:0〜255, green:0〜255, blue:0〜255</td>
  </tr>
</table>


値を省略した場合は#M0同様各値を初期化します。

```ruby
commander.execute!([])
```


2. インストール手順
------

インストールは(1)直接ダウンロードしてインストールする か、(2)Gemfileを使ってインストールします。

(1)の手順:
```
# git clone -b master --depth 1 https://github.com/tk-hamaguchi/rapiro_wrapper.git
# cd rapiro_wrapper/
# gem build rapiro_wrapper.gemspec
# gem install rapiro_wrapper-*.gem
```

(2)のGemfileへの追記内容:
```ruby
gem 'rapiro_wrapper', github: 'tk-hamaguchi/rapiro_wrapper'
```

3. 読み込みと初期化
------

シリアルポートをRapiroWrapper::Commanderの引数に与えて初期化します。
※RaspberyPiを接続しているときはシリアルポートが'/dev/ttyAMA0'場合引数を省略できます。
``` ruby
require 'rapiro_wrapper'


commander = RapiroWrapper::Commander.new('/dev/tty.usbserial-DA00HMG6')
```


4. しぐさサンプル
-------

とりあえず全部動かしてみる

<iframe width="560" height="315" src="//www.youtube.com/embed/iA_qV0nP3kA" frameborder="0" allowfullscreen></iframe>

``` ruby
commander.execute!([
  RapiroWrapper::Head.new( left: 30 ),
  RapiroWrapper::Waist.new( right: 30 ),
  RapiroWrapper::RightSholderRoll.new( up: 160 ),
  RapiroWrapper::RightSholderPitch.new( up: 20 ),
  RapiroWrapper::RightHandGrip.new( hold: 0 ),
  RapiroWrapper::LeftSholderRoll.new( up: 30 ),
  RapiroWrapper::LeftSholderPitch.new( up: 20 ),
  RapiroWrapper::LeftHandGrip.new( open: 0 ),
  RapiroWrapper::RightFootYaw.new( open: 30 ),
  RapiroWrapper::RightFootPitch.new( open: 5 ),
  RapiroWrapper::LeftFootYaw.new( close: 20 ),
  RapiroWrapper::LeftFootPitch.new( close: 0 ),
  RapiroWrapper::Eyes.new('#00ff00')
])

sleep 5

commander.execute!([])
```

首ふり。

<iframe width="560" height="315" src="//www.youtube.com/embed/6m-REzt9UDs" frameborder="0" allowfullscreen></iframe>

``` ruby
duration = 7

2.times.each do

  commander.execute!([
    RapiroWrapper::Head.new( left: 30 )
  ], duration)

  sleep duration * 0.1

  commander.execute!([
    RapiroWrapper::Head.new( right: 30 )
  ], duration)

  sleep duration * 0.1

end

commander.execute!([],10)
```

あばれてみる。

<iframe width="560" height="315" src="//www.youtube.com/embed/UvvGZUE2Si8" frameborder="0" allowfullscreen></iframe>

``` ruby
duration = 5

5.times.each do

  commander.execute!([
    RapiroWrapper::Head.new( left: 10 ),
    RapiroWrapper::RightSholderRoll.new( up: 160 ),
    RapiroWrapper::LeftSholderRoll.new( up: 160 ),
    RapiroWrapper::RightSholderPitch.new( up: 5 ),
    RapiroWrapper::LeftSholderPitch.new( up: 5 ),
    RapiroWrapper::RightFootPitch.new( open: 0 ),
    RapiroWrapper::LeftFootPitch.new( close: 0 ),
    RapiroWrapper::Eyes.new('#ff0000')
  ], duration)

  sleep duration * 0.1

  commander.execute!([
    RapiroWrapper::Head.new( right: 10 ),
    RapiroWrapper::RightSholderRoll.new( up: 170 ),
    RapiroWrapper::LeftSholderRoll.new( up: 170 ),
    RapiroWrapper::RightSholderPitch.new( up: 40 ),
    RapiroWrapper::LeftSholderPitch.new( up: 40 ),
    RapiroWrapper::RightFootPitch.new( close: 0 ),
    RapiroWrapper::LeftFootPitch.new( open: 0 ),
    RapiroWrapper::Eyes.new('#330000')
  ], duration)

  sleep duration * 0.1

end

commander.execute!([],10)
```

ドヤァ〜

<iframe width="560" height="315" src="//www.youtube.com/embed/q26WvcQ6zVk" frameborder="0" allowfullscreen></iframe>

``` ruby
commander.execute!([
  RapiroWrapper::Head.new( left: 40 ),
  RapiroWrapper::Waist.new( right: 40 ),
  RapiroWrapper::LeftHandGrip.new( hold: 0 ),
  RapiroWrapper::RightHandGrip.new( hold: 0 ),
  RapiroWrapper::RightSholderRoll.new( up: 90 ),
  RapiroWrapper::LeftSholderRoll.new( up: 20 ),
  RapiroWrapper::LeftFootYaw.new( open: 30 ),
  RapiroWrapper::RightFootYaw.new( open: 20 ),
  RapiroWrapper::Eyes.new('#003300')
], 5)

sleep 1

commander.execute!([
  RapiroWrapper::Head.new( right: 30 ),
  RapiroWrapper::Waist.new( left: 30 ),
  RapiroWrapper::LeftHandGrip.new( hold: 0 ),
  RapiroWrapper::RightHandGrip.new( hold: 0 ),
  RapiroWrapper::RightSholderPitch.new( up: 30 ),
  RapiroWrapper::RightSholderRoll.new( up: 90 ),
  RapiroWrapper::LeftSholderRoll.new( up: 0 ),
  RapiroWrapper::LeftFootYaw.new( open: 30 ),
  RapiroWrapper::RightFootYaw.new( open: 20 ),
  RapiroWrapper::Eyes.new('#00ff00')
], 2)

sleep 5

commander.execute!([],10)
```


というわけで、人によっては使いやすいかも？

ぼちぼちテスト書くか〜
