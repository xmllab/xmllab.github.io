---
layout: post
title: "MacでImageMagickとffmpeg、RMagick(ruby)を使って画像から動画を生成する"
date: 2014-03-09 11:20:40 +0900
comments: true
categories: mac imagemagick movie ffmpeg rmagick ruby
---


こないだ出したアプリコンテストで画像から動画生成をしてみたものの、一瞬で動画が終了するみたいなことがあったので、その調査がてらMacに環境を作ってみるなど。


1. 環境
------

今回は底まで複雑な作業をしないのと、画像素材をサーバーに転送するのが面倒だったのでMacで実施．

 アプリケーション | バージョン
:----------------:|:-----------------
 OS               | MacOS 10.9
 ImageMagick      | 6.8.7-7
 ffmpeg           | 1.2.4            
 ruby             | 2.1.1
 RMagick          | 2.13.2
 

2. インストール
------

まずはImageMagickをbrewからインストール。

```sh
$ brew install imagemagick

Warning: No developer tools installed.
You should install the Command Line Tools.
Run `xcode-select --install` to install them.
==> Installing dependencies for imagemagick: jpeg, libpng, freetype
==> Installing imagemagick dependency: jpeg
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/jpeg-8d.mavericks.bottle.tar.gz
######################################################################## 100.0%
==> Pouring jpeg-8d.mavericks.bottle.tar.gz
🍺  /usr/local/Cellar/jpeg/8d: 18 files, 780K
==> Installing imagemagick dependency: libpng
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/libpng-1.5.17.mavericks.bottle.tar.gz
######################################################################## 100.0%
==> Pouring libpng-1.5.17.mavericks.bottle.tar.gz
🍺  /usr/local/Cellar/libpng/1.5.17: 15 files, 1.0M
==> Installing imagemagick dependency: freetype
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/freetype-2.5.1.mavericks.bottle.tar.gz
######################################################################## 100.0%
==> Pouring freetype-2.5.1.mavericks.bottle.tar.gz
🍺  /usr/local/Cellar/freetype/2.5.1: 59 files, 2.7M
==> Installing imagemagick
==> Downloading https://downloads.sf.net/project/machomebrew/Bottles/imagemagick-6.8.7-7.mavericks.bottle.tar.gz
######################################################################## 100.0%
==> Pouring imagemagick-6.8.7-7.mavericks.bottle.tar.gz
🍺  /usr/local/Cellar/imagemagick/6.8.7-7: 1431 files, 20M
```

XCodeのCommandLineToolsが入ってないといわれたのでインストール

```sh
$ xcode-select --install                                                                                                                             (14-03-07 10:23:30)

xcode-select: note: install requested for command line developer tools
```

続いてffmpegをインストール

```sh
$ brew install ffmpeg

==> Installing dependencies for ffmpeg: texi2html, yasm, x264, faac, lame, xvid
==> Installing ffmpeg dependency: texi2html
==> Downloading http://download.savannah.gnu.org/releases/texi2html/texi2html-1.82.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/texi2html/1.82 --mandir=/usr/local/Cellar/texi2html/1.82/share/man --infodir=/usr/local/Cellar/texi2html/1.82/share/info
==> make install
🍺  /usr/local/Cellar/texi2html/1.82: 107 files, 2.2M, built in 11 seconds
==> Installing ffmpeg dependency: yasm
==> Downloading http://tortall.net/projects/yasm/releases/yasm-1.2.0.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/yasm/1.2.0
==> make install
🍺  /usr/local/Cellar/yasm/1.2.0: 44 files, 3.3M, built in 30 seconds
==> Installing ffmpeg dependency: x264
==> Downloading http://download.videolan.org/pub/videolan/x264/snapshots/x264-snapshot-20120812-2245-stable.tar.bz2
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/x264/r2197.4 --enable-shared
==> make install
==> Caveats
Because libx264 has a rapidly-changing API, formulae that link against
it should be reinstalled each time you upgrade x264. Examples include:
   avidemux, ffmbc, ffmpeg, gst-plugins-ugly
==> Summary
🍺  /usr/local/Cellar/x264/r2197.4: 8 files, 1.8M, built in 40 seconds
==> Installing ffmpeg dependency: faac
==> Downloading http://downloads.sourceforge.net/project/faac/faac-src/faac-1.28/faac-1.28.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/faac/1.28
==> make install
🍺  /usr/local/Cellar/faac/1.28: 13 files, 720K, built in 23 seconds
==> Installing ffmpeg dependency: lame
==> Downloading http://downloads.sourceforge.net/sourceforge/lame/lame-3.99.5.tar.gz
######################################################################## 100.0%
==> ./configure --prefix=/usr/local/Cellar/lame/3.99.5 --enable-nasm
==> make install
🍺  /usr/local/Cellar/lame/3.99.5: 25 files, 2.1M, built in 24 seconds
==> Installing ffmpeg dependency: xvid
==> Downloading http://fossies.org/unix/privat/xvidcore-1.3.2.tar.gz
######################################################################## 100.0%
==> ./configure --disable-assembly --prefix=/usr/local/Cellar/xvid/1.3.2
==> make
==> make install
🍺  /usr/local/Cellar/xvid/1.3.2: 9 files, 1.3M, built in 21 seconds
==> Installing ffmpeg
==> Downloading http://ffmpeg.org/releases/ffmpeg-1.2.4.tar.bz2
######################################################################## 100.0%
==> Patching
patching file libavfilter/vf_drawtext.c
==> ./configure --prefix=/usr/local/Cellar/ffmpeg/1.2.4 --enable-shared --enable-pthreads --enable-gpl --enable-version3 --enable-nonfree --enable-hardcoded-tables --ena
==> make install
🍺  /usr/local/Cellar/ffmpeg/1.2.4: 147 files, 26M, built in 3.7 minutes
```

これだけで入るとか楽な時代になったなぁ（￣▽￣；


続けてrmagickをインストール。スイッチが楽なのでrvmを利用してます。

```sh
$ gem install rmagick -v '2.13.2' --no-rdoc --no-ri

Building native extensions.  This could take a while...
Successfully installed rmagick-2.13.2
1 gem installed
```

もしうまく行かない時はPKG_CONFIG_PATHを指定する。
パスは上記でインストールされたImageMagickのパスを一部に利用する

```sh
$ export PKG_CONFIG_PATH=/opt/local/lib/pkgconfig:/usr/local/Cellar/imagemagick/6.8.7-7/lib/pkgconfig/:$PKG_CONFIG_PATH
```


これで環境構築は完了。

3. 変換
------

今回は下記のフォルダ構成のファイルを想定します。

```
images/
  + IMG_0001.jpg
  + IMG_0002.jpg
  + IMG_0003.jpg
  + IMG_0004.jpg
  + IMG_0005.jpg
  + IMG_0006.jpg
  + IMG_0007.jpg
  + IMG_0008.jpg
  + IMG_0009.jpg
  + IMG_0010.jpg
```

エディタで下記のようにコードを書きます

```ruby image2movie.rb
#!/usr/bin/env ruby

require 'rubygems'
require 'rmagick'

image_list = Magick::ImageList.new(*Dir.glob('images/IMG_*.jpg'))
image_list.delay = 50
image_list.write('output.mp4')
```

実行したらoutput.mp4が生成されます

```sh
$ ruby image2movie.rb
```

4. チューニング
------

動画を画像にする際のフレームレートは、Magick::ImageListインスタンスの#delay=と#ticks_per_second=で決定されます。

Magick::ImageList#ticks_per_second=は１秒間に何フレーム入れるか？というパラメータっぽく観えたけど、ticks_per_second=1にしても1fpsにならないのでたぶんうちの理解が間違ってるんだろう。
デフォルトは100になっているので、これを基準にMagick::ImageList#delay=を設定していきます。

Magick::ImageList#delay=とfpsの対応は下記の通り

 delayの値 | fps 
:---------:|:-----:
  1000     |  0.1
   200     |  0.5
   100     |   1
    50     |   2
    20     |   5
    10     |  10

上記のコードではdelay=50にしているので、生成されるmp4では１秒間に２枚の画像が表示されるはずです。



