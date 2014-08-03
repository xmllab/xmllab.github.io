---
layout: post
title: "CentOSでVagrantUpするまで"
date: 2014-03-03 22:47:17 +0900
comments: true
categories: Vagrant VirtualBox VonV Linux CentOS
---

最近ハッカソンやらアプリコンテストやらでちらほら開発してるんだけど、その中でちらほら本番環境にデプロイしたら・・・あれ？みたいなことがちょこちょこ起きてるので、短期間で効率よく試験できる環境を作ろうと思って今更ながらVagrant使ってみた。まずは仮想サーバーを立ててログインする所まで。



1. 環境
------

いつも持ち歩いてるのはMacbookAirで、開発してるのはその上でVirtualBoxを使って動かしてるCentOS。今回はさらにその上にVirtualBoxを重ねるVonVの構成。ホントはMacのVirtualBoxをサーバー的に叩けるのがいいんだろうけど、まぁきっとそのうちAWSに切り替えるだろうと思って今回はこの構成でトライ。

 アプリケーション | バージョン
:---------------:|:-----------------
 HostOS          | CentOS 6.5 64bit
 Vagrant         | 1.4.3            
 VirtualBox      | 4.3             



2. 依存ライブラリのインストール
------

VirtualBoxはDKMS(Dynamic Kernel Module Support)を使っているらしいのでインストール

``` sh

$ sudo yum install dkms

Loaded plugins: fastestmirror, presto, refresh-packagekit
Loading mirror speeds from cached hostfile
 * base: www.ftp.ne.jp
 * epel: ftp.jaist.ac.jp
 * extras: www.ftp.ne.jp
 * updates: www.ftp.ne.jp
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package dkms.noarch 0:2.2.0.3-20.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================
 Package                              Arch                                   Version                                          Repository                            Size
=========================================================================================================================================================================
Installing:
 dkms                                 noarch                                 2.2.0.3-20.el6                                   epel                                  75 k

Transaction Summary
=========================================================================================================================================================================
Install       1 Package(s)

Total download size: 75 k
Installed size: 209 k
Is this ok [y/N]: y
Downloading Packages:
Setting up and reading Presto delta metadata
Processing delta metadata
Package(s) data still to download: 75 k
dkms-2.2.0.3-20.el6.noarch.rpm                                                                                                                    |  75 kB     00:00     
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : dkms-2.2.0.3-20.el6.noarch                                                                                                                            1/1 
  Verifying  : dkms-2.2.0.3-20.el6.noarch                                                                                                                            1/1 

Installed:
  dkms.noarch 0:2.2.0.3-20.el6                                                                                                                                           

Complete!

```


3. VirtualBoxのインストール
------

そしてVirtualBoxをyumからインストール。
公式がリポジトリファイルを用意してくれているので、まずそれを取り込みます。

```sh
$ sudo wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo -P /etc/yum.repos.d/

--2014-03-01 13:09:50--  http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
download.virtualbox.org をDNSに問いあわせています... 137.254.120.26
download.virtualbox.org|137.254.120.26|:80 に接続しています... 接続しました。
HTTP による接続要求を送信しました、応答を待っています... 200 OK
長さ: 256 [text/plain]
`/etc/yum.repos.d/virtualbox.repo' に保存中

100%[===============================================================================================================================>] 256         --.-K/s 時間 0s      

2014-03-01 13:09:51 (22.7 MB/s) - `/etc/yum.repos.d/virtualbox.repo' へ保存完了 [256/256]
```

その後本体のインストール。今回はVirtualBox 4.3を利用します。

```sh
$ sudo  yum install VirtualBox-4.3 -y

Loaded plugins: fastestmirror, presto, refresh-packagekit
Loading mirror speeds from cached hostfile
 * base: www.ftp.ne.jp
 * epel: ftp.jaist.ac.jp
 * extras: www.ftp.ne.jp
 * updates: www.ftp.ne.jp
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package VirtualBox-4.3.x86_64 0:4.3.8_92456_el6-1 will be installed
--> Processing Dependency: libSDL-1.2.so.0()(64bit) for package: VirtualBox-4.3-4.3.8_92456_el6-1.x86_64
--> Running transaction check
---> Package SDL.x86_64 0:1.2.14-3.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================
 Package                                   Arch                              Version                                         Repository                             Size
=========================================================================================================================================================================
Installing:
 VirtualBox-4.3                            x86_64                            4.3.8_92456_el6-1                               virtualbox                             73 M
Installing for dependencies:
 SDL                                       x86_64                            1.2.14-3.el6                                    base                                  193 k

Transaction Summary
=========================================================================================================================================================================
Install       2 Package(s)

Total size: 73 M
Total download size: 73 M
Installed size: 148 M
Downloading Packages:
Setting up and reading Presto delta metadata
Processing delta metadata
Package(s) data still to download: 73 M
VirtualBox-4.3-4.3.8_92456_el6-1.x86_64.rpm                                                                                                       |  73 MB     00:23     
warning: rpmts_HdrFromFdno: Header V4 DSA/SHA1 Signature, key ID 98ab5139: NOKEY
Retrieving key from http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc
Importing GPG key 0x98AB5139:
 Userid: "Oracle Corporation (VirtualBox archive signing key) <info@virtualbox.org>"
 From  : http://download.virtualbox.org/virtualbox/debian/oracle_vbox.asc
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : SDL-1.2.14-3.el6.x86_64                                                                                                                               1/2 
  Installing : VirtualBox-4.3-4.3.8_92456_el6-1.x86_64                                                                                                               2/2 


Creating group 'vboxusers'. VM users must be member of that group!

No precompiled module for this kernel found -- trying to build one. Messages
emitted during module compilation will be logged to /var/log/vbox-install.log.

Stopping VirtualBox kernel modules [  OK  ]
Uninstalling old VirtualBox DKMS kernel modules [  OK  ]
Trying to register the VirtualBox kernel modules using DKMS [  OK  ]
Starting VirtualBox kernel modules [  OK  ]
  Verifying  : SDL-1.2.14-3.el6.x86_64                                                                                                                               1/2 
  Verifying  : VirtualBox-4.3-4.3.8_92456_el6-1.x86_64                                                                                                               2/2 

Installed:
  VirtualBox-4.3.x86_64 0:4.3.8_92456_el6-1                                                                                                                              

Dependency Installed:
  SDL.x86_64 0:1.2.14-3.el6                                                                                                                                              

Complete!

```

4. Vagrantのインストール
------

いよいよ本丸。Vagrantのインストール。これは本家が公開しているRPMを利用してインストールします。ラクチン〜♪

```sh
$ wget https://dl.bintray.com/mitchellh/vagrant/vagrant_1.4.3_x86_64.rpm -P /tmp/
$ sudo yum install ~/vagrant_1.4.3_x86_64.rpm

Loaded plugins: fastestmirror, presto, refresh-packagekit
Loading mirror speeds from cached hostfile
 * base: www.ftp.ne.jp
 * epel: ftp.jaist.ac.jp
 * extras: www.ftp.ne.jp
 * updates: www.ftp.ne.jp
Setting up Install Process
Examining /tmp/vagrant_1.4.3_x86_64.rpm: 1:vagrant-1.4.3-1.x86_64
Marking /tmp/vagrant_1.4.3_x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package vagrant.x86_64 1:1.4.3-1 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================================================================
 Package                             Arch                               Version                                  Repository                                         Size
=========================================================================================================================================================================
Installing:
 vagrant                             x86_64                             1:1.4.3-1                                /vagrant_1.4.3_x86_64                              53 M

Transaction Summary
=========================================================================================================================================================================
Install       1 Package(s)

Total size: 53 M
Installed size: 53 M
Is this ok [y/N]: y
Downloading Packages:
Running rpm_check_debug
Running Transaction Test
Transaction Test Succeeded
Running Transaction
  Installing : 1:vagrant-1.4.3-1.x86_64                                                                                                                              1/1 
  Verifying  : 1:vagrant-1.4.3-1.x86_64                                                                                                                              1/1 

Installed:
  vagrant.x86_64 1:1.4.3-1                                                                                                                                               

Complete!
```

5. 仮想マシンを作成
------

意外にさっくり行くようで所々時間かかる部分もあったり。
次はBOX(仮想マシンのテンプレートみたいなもの)を指定してダウンロードします。
今回は本家のチュートリアルどおりprecise32を利用します。

他のBOXについては [Vagrantbox.es](http://www.vagrantbox.es/) がいい感じにまとめてくれています。

```sh
$ mkdir vagrant-test
$ cd vagrant-test
$ vagrant init precise32 http://files.vagrantup.com/precise32.box

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```


6. 仮想マシンの起動
------

いよいよ起動！
これは意外と時間かかる。

```sh
$ vagrant up

Bringing machine 'default' up with 'virtualbox' provider...
[default] Box 'precise32' was not found. Fetching box from specified URL for
the provider 'virtualbox'. Note that if the URL does not have
a box for this provider, you should interrupt Vagrant now and add
the box yourself. Otherwise Vagrant will attempt to download the
full box prior to discovering this error.
Downloading box from URL: http://files.vagrantup.com/precise32.box
Extracting box...################################################################% (Rate: /s, Estimated time remaining: ))
Successfully added box 'precise32' with provider 'virtualbox'!
[default] Importing base box 'precise32'...
[default] Matching MAC address for NAT networking...
[default] Setting the name of the VM...
[default] Clearing any previously set forwarded ports...
[default] Fixed port collision for 22 => 2222. Now on port 2200.
[default] Clearing any previously set network interfaces...
[default] Preparing network interfaces based on configuration...
[default] Forwarding ports...
[default] -- 22 => 2200 (adapter 1)
[default] Booting VM...
[default] Waiting for machine to boot. This may take a few minutes...
[default] Machine booted and ready!
[default] The guest additions on this VM do not match the installed version of
VirtualBox! In most cases this is fine, but in rare cases it can
prevent things such as shared folders from working properly. If you see
shared folder errors, please make sure the guest additions within the
virtual machine match the version of VirtualBox you have installed on
your host and reload your VM.

Guest Additions Version: 4.2.0
VirtualBox Version: 4.3
[default] Mounting shared folders...
[default] -- /vagrant
```

まぁワーニング出たけど起動はしてるっぽい。

7. ログインしてみる
--------

どうやら下記でログインできるらしい。

```sh
$ vagrant ssh
Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic-pae i686)

 * Documentation:  https://help.ubuntu.com/
Welcome to your Vagrant-built virtual machine.
Last login: Fri Sep 14 06:22:31 2012 from 10.0.2.2
vagrant@precise32:~$ 
```

今回はとりあえず起動まで。ChefのBerkshelfと組み合わせてもっとごにょごにょできそうなので、次はCentOSのBOX作りながらちょこっとごにょごにょしてみようかな。

