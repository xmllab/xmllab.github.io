---
layout: post
title: "RaspberryPiのインストールと初期設定"
date: 2016-12-31 11:44:25 +0900
comments: false
categories: 
---

MacでRaspberryPiのインストールと初期設定を行う手順のメモです。

主にMacのターミナルからのコマンドベースの設定になります。

気がついたらRaspbianもsystemdベースになっていたので、[以前作ったRaspberryPi用のRecipe](https://github.com/tk-hamaguchi/itamae-plugin-recipe-raspberry_pi/blob/master/lib/itamae/plugin/recipe/raspberry_pi.rb)をベースにいくつか追加しています。


Raspbianをダウンロードする
--------

[Raspbianのダウンロードサイト](https://www.raspberrypi.org/downloads/raspbian/)からRASPBIAN JESSIE LITEのZipをダウンロードします

Safariでダウンロードした場合、ダウンロード完了後に解凍されたものが`~/Download`配下に作成されたはずです。

```
bash-3.2$ ls ~/Downloads/2016-11-25-raspbian-jessie-lite.img
```


SDカードへイメージを書き込む
--------

RaspberryPiで使うMicroSDをメモリーカードリーダーを使ってMacに接続し、フォーマットを実行します

まずMac上で`diskutil`コマンドを実行し、MicroSDの状態とパスを確認します。

```
bash-3.2$ diskutil list 
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *960.2 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh SSD           839.3 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
   4:       Microsoft Basic Data BOOTCAMP                120.0 GB   disk0s4
/dev/disk1 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                                                   *31.7 GB    disk1
bash-3.2$ 
```

上記の場合`/dev/disk1`にマッピングされています。

もしディスクがマウントされている場合は下記の手順でアンマウントします。

```
bash-3.2$ sudo diskutil unmountDisk /dev/disk1
```

続けてRaspbianのイメージを書き込みます

```
bash-3.2$ sudo dd bs=1m if=~/2016-11-25-raspbian-jessie-lite.img  of=/dev/disk1
1326+0 records in
1326+0 records out
1390411776 bytes transferred in 1103.233323 secs (1260306 bytes/sec)
bash-3.2$ 
```

書き込み完了後、MicroSDをMacから抜き出し、RaspberryPiへ差し込みます。

RaspberryPiへのシリアル接続する
--------

まずGPIO経由でRaspberryPiとMacをシリアル接続します。

自分の場合は[SWITCH SCIENCEのFTDI USBシリアル変換アダプター](https://www.switch-science.com/catalog/1032/)を使って下記のように配線しています。

|   RaspberryPi側   | USBシリアル変換アダプター側 |
|:-----------------:|:---------------------------:|
|  6 : Ground       | GND |
|  8 : GPIO14 (TXD) | RX  |
| 10 : GPIO15 (RXD) | TX  |

リンク先の通りFTDIのドライバが必要になるので、Macにインストールしておく必要があります。

そしてその後MicroUSBケーブルでUSBシリアル変換アダプターとMacを繋ぎます。

シリアルへの接続は、Macの場合、`screen`コマンドを利用して接続します。

```
bash-3.2$ sudo screen  /dev/tty.usbserial-* 115200
```

RaspberryPiの電源を入れると、screen上に起動シーケンスが表示され、ログインプロンプトが表示されます。

```
Raspbian GNU/Linux 8 raspberrypi ttyAMA0

raspberrypi login:
```

RaspberryerryPiにログインする
--------

ここで下記を入力します

| Input field       | Value     |
|-------------------|-----------|
| raspberrypi login | pi        |
| Password          | raspberry |

するとpiユーザーとしてログイン出来ます。

```
raspberrypi login: pi
Password: 
Linux raspberrypi 4.4.34-v7+ #930 SMP Wed Nov 23 15:20:41 GMT 2016 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
pi@raspberrypi:~$ 
```

その後rootユーザーに切り替えます

```
pi@raspberrypi:~$ sudo su -
root@raspberrypi:~# 
```

swap領域の無効化
--------

Raspbianはswap領域としてSDカードを利用します。これはSDカードの寿命を縮める原因の一つとなるため、無効化します。

まずswapを一時的に無効化し、データをメモリ上に移動します。

```
root@raspberrypi:~# swapoff --all
root@raspberrypi:~# 
```

次に`dphys-swapfile`を停止した後パッケージを削除し、恒久的にswapファイルを作成しないように変更します。

```
root@raspberrypi:~# systemctl stop dphys-swapfile
root@raspberrypi:~# systemctl disable dphys-swapfile
Synchronizing state for dphys-swapfile.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d dphys-swapfile defaults
Executing /usr/sbin/update-rc.d dphys-swapfile disable
insserv: warning: current start runlevel(s) (empty) of script `dphys-swapfile' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (2 3 4 5) of script `dphys-swapfile' overrides LSB defaults (empty).
```
```
root@raspberrypi:~# apt-get remove -y dphys-swapfile
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  dc
Use 'apt-get autoremove' to remove it.
The following packages will be REMOVED:
  dphys-swapfile
0 upgraded, 0 newly installed, 1 to remove and 0 not upgraded.
After this operation, 85.0 kB disk space will be freed.
Do you want to continue? [Y/n] 
(Reading database ... 31272 files and directories currently installed.)
Removing dphys-swapfile (20100506-1) ...
Processing triggers for man-db (2.7.0.2-5) ...
root@raspberrypi:~# 
```

時刻同期ツールの設定
--------

RaspberryPiが起動するタイミングでNTPによる時刻同期を行うよう、にする。実際にはNTPデーモンを停止し、起動シーケンスの中で`ntpdate`コマンドにより時刻同期を実現する。

まずNTPデーモンを停止します

```
root@raspberrypi:~# systemctl stop ntp
root@raspberrypi:~# systemctl disable ntp
Synchronizing state for ntp.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d ntp defaults
Executing /usr/sbin/update-rc.d ntp disable
insserv: warning: current start runlevel(s) (empty) of script `ntp' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (2 3 4 5) of script `ntp' overrides LSB defaults (empty).
root@raspberrypi:~# 
```

次に`ntpdate`コマンドをインストールします

```
root@raspberrypi:~# apt-get install -y ntpdate
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  liblockfile-bin liblockfile1 lockfile-progs
The following NEW packages will be installed:
  liblockfile-bin liblockfile1 lockfile-progs ntpdate
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 112 kB of archives.
After this operation, 250 kB of additional disk space will be used.
Get:1 http://mirrordirector.raspbian.org/raspbian/ jessie/main liblockfile-bin armhf 1.09-6 [18.2 kB]
Get:2 http://mirrordirector.raspbian.org/raspbian/ jessie/main liblockfile1 armhf 1.09-6 [14.7 kB]
Get:3 http://mirrordirector.raspbian.org/raspbian/ jessie/main ntpdate armhf 1:4.2.6.p5+dfsg-7+deb8u2 [69.0 kB]
Get:4 http://mirrordirector.raspbian.org/raspbian/ jessie/main lockfile-progs armhf 0.1.17 [10.6 kB]
Fetched 112 kB in 2s (38.4 kB/s)     
Selecting previously unselected package liblockfile-bin.
(Reading database ... 31253 files and directories currently installed.)
Preparing to unpack .../liblockfile-bin_1.09-6_armhf.deb ...
Unpacking liblockfile-bin (1.09-6) ...
Selecting previously unselected package liblockfile1:armhf.
Preparing to unpack .../liblockfile1_1.09-6_armhf.deb ...
Unpacking liblockfile1:armhf (1.09-6) ...
Selecting previously unselected package ntpdate.
Preparing to unpack .../ntpdate_1%3a4.2.6.p5+dfsg-7+deb8u2_armhf.deb ...
Unpacking ntpdate (1:4.2.6.p5+dfsg-7+deb8u2) ...
Selecting previously unselected package lockfile-progs.
Preparing to unpack .../lockfile-progs_0.1.17_armhf.deb ...
Unpacking lockfile-progs (0.1.17) ...
Processing triggers for man-db (2.7.0.2-5) ...
Setting up liblockfile-bin (1.09-6) ...
Setting up liblockfile1:armhf (1.09-6) ...
Setting up ntpdate (1:4.2.6.p5+dfsg-7+deb8u2) ...
Setting up lockfile-progs (0.1.17) ...
Processing triggers for libc-bin (2.19-18+deb8u6) ...
root@raspberrypi:~# 
```

起動時シーケンスを設定
--------

`/etc/fstab`と`/etc/rc.local`を編集して起動シーケンスを設定します。

SDカードの寿命を少しでも延ばすため、ログデータやテンポラリファイルを`tmpfs`上に作成するよう変更します

```
root@raspberrypi:~# cat << EOT >> /etc/fstab
tmpfs           /tmp            tmpfs   defaults,size=32m,noatime,mode=1777      0       0
tmpfs           /var/tmp        tmpfs   defaults,size=16m,noatime,mode=1777      0       0
tmpfs           /var/log        tmpfs   defaults,size=32m,noatime,mode=0755      0       0
EOT
root@raspberrypi:~# 
```

次に起動時に時刻同期と、必要なディレクトリ作成を行います。

```
root@raspberrypi:~# cat << EOT > /etc/rc.local
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

# Print the IP address
_IP=$(hostname -I) || true
if [ "$_IP" ]; then
  printf "My IP address is %s\n" "$_IP"
  ntpdate ntp.nict.jp
fi

mkdir -p /var/log/ConsoleKit
mkdir -p /var/log/samba
mkdir -p /var/log/fsck
mkdir -p /var/log/apt
mkdir -p /var/log/ntpstats
chown root.ntp /var/log/ntpstats
chown root.adm /var/log/samba
touch /var/log/lastlog
touch /var/log/wtmp
touch /var/log/btmp
chown root.utmp /var/log/lastlog
chown root.utmp /var/log/wtmp
chown root.utmp /var/log/btmp

exit 0
EOT
root@raspberrypi:~# 
```

RaspberryPiのroot領域を拡大する
--------

`raspi-config`コマンドで領域をいっぱいいっぱいまで拡げます。

```
root@raspberrypi:~# raspi-config --expand-rootfs
root@raspberrypi:~# 
```

再起動の実行
--------

ここまできたらとりあえず再起動を行います

```
root@raspberrypi:~# reboot
```

再起動後は前回同様ログインし、rootユーザーに切り替えます

```
Raspbian GNU/Linux 8 raspberrypi ttyAMA0

raspberrypi login: pi
Password: 
Linux raspberrypi 4.4.34-v7+ #930 SMP Wed Nov 23 15:20:41 GMT 2016 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
pi@raspberrypi:~$ sudo su -
root@raspberrypi:~# 
```

WiFiの設定を行う
--------

RaspberryPi3の場合はWiFiが内蔵されてます。それ以外のWiFi非搭載モデルの場合はBuffaloのWLI-UC-GNMとかを使って繋ぎます。（挿して起動するだけでwlan0として認識されます）

有線LANで接続する場合は不要ですが、WiFiを使って接続する場合、下記のようにwpa_supplicantの設定を施します。

下記の例：

|  設定項目 |  設定内容  | 設定値    | 備考 |
|-----------|------------|-----------|----|
| ssid      | SSID       | `test-ap`   | 接続先WiFiのSSID |
| proto     | プロトコル | `RSN`       | WPA2の場合`RSN`でWPAの場合はそのまま`WPA`を設定する |
| key_mgmt  | 認証方式   | `WPA-PSK`   | プリシェアードキーを使って最初の通信を確立する場合 |
| pairwise  | ユニキャスト通信の暗号化方式 | `CCMP TKIP` | AESを利用する場合`CCMP`、TKIPの場合は`TKIP`を指定する |
| group     | ブロードキャストまたはマルチキャスト通信の暗号化方式 | `CCMP TKIP` | AESを利用する場合`CCMP`、TKIPの場合は`TKIP`を指定する |
| psk       | WPAプリシェアードキー | `password_for_your_wifi` | 接続先WiFiのパスフレーズ |

設定方法の詳細は`man wpa_supplicant.conf`でも確認できます

```
root@raspberrypi:~# cat << EOT >> /etc/wpa_supplicant/wpa_supplicant.conf
network={
        ssid="test-ap"
        proto=RSN
        key_mgmt=WPA-PSK
        pairwise=CCMP TKIP
        group=CCMP TKIP
        psk="password_for_your_wifi"
}
EOT
root@raspberrypi:~# 
```

その後`ifdown`と`ifup`で無線LANのインターフェースを再起動します。

```
root@raspberrypi:~# ifdown wlan0
root@raspberrypi:~# ifup wlan0
root@raspberrypi:~# ip a s wlan0 | grep 'inet '
    inet 192.168.0.12/24 brd 192.168.0.255 scope global wlan0
root@raspberrypi:~# 
```

各種アップデートの実行
--------

まず各パッケージをアップデートします。

```
root@raspberrypi:~# apt-get update -y -qq && apt-get upgrade -y -qq
root@raspberrypi:~# apt-get dist-upgrade -y -qq
root@raspberrypi:~# 
```

次にRaspberryPiのファームウェアをアップデートしてくれる`rpi-update`をインストールします。

```
root@raspberrypi:~# apt-get install rpi-update
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following package was automatically installed and is no longer required:
  dc
Use 'apt-get autoremove' to remove it.
The following NEW packages will be installed:
  rpi-update
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 4,408 B of archives.
After this operation, 45.1 kB of additional disk space will be used.
Get:1 http://archive.raspberrypi.org/debian/ jessie/main rpi-update all 20140705 [4,408 B]
Fetched 4,408 B in 0s (4,976 B/s)                        
Selecting previously unselected package rpi-update.
(Reading database ... 31264 files and directories currently installed.)
Preparing to unpack .../rpi-update_20140705_all.deb ...
Unpacking rpi-update (20140705) ...
Setting up rpi-update (20140705) ...
root@raspberrypi:~# 
```

その後ファームウェアアップデートを実行します

```
root@raspberrypi:~# SKIP_WARNING=1 rpi-update                                                                                                                    
 *** Raspberry Pi firmware updater by Hexxeh, enhanced by AndrewS and Dom
 *** Performing self-update
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12022  100 12022    0     0  67782      0 --:--:-- --:--:-- --:--:-- 67920
 *** Relaunching after update
 *** Raspberry Pi firmware updater by Hexxeh, enhanced by AndrewS and Dom
 *** We're running for the first time
 *** Backing up files (this will take a few minutes)
 *** Backing up firmware
 *** Backing up modules 4.4.34-v7+
This update bumps to rpi-4.4.y linux tree
Be aware there could be compatibility issues with some drivers
Discussion here:
https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=144087
##############################################################
 *** Downloading specific firmware revision (this will take a few minutes)
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   168    0   168    0     0    198      0 --:--:-- --:--:-- --:--:--   198
100 51.8M  100 51.8M    0     0  2136k      0  0:00:24  0:00:24 --:--:-- 2371k
 *** Updating firmware
 *** Updating kernel modules
 *** depmod 4.4.38-v7+
 *** depmod 4.4.38+
 *** Updating VideoCore libraries
 *** Using HardFP libraries
 *** Updating SDK
 *** Running ldconfig
 *** Storing current firmware revision
 *** Deleting downloaded files
 *** Syncing changes to disk
 *** If no errors appeared, your firmware was successfully updated to 87bf11faa6ccefb7969821bff8cbc44f72be7414
 *** A reboot is needed to activate the new firmware
root@raspberrypi:~# 
```

最後に不要なパッケージを整理します

```
root@raspberrypi:~# apt-get autoremove -y -qq
The mail frontend needs a installed 'sendmail', using pager
(Reading database ... 31268 files and directories currently installed.)
Removing dc (1.06.95-9) ...
Processing triggers for install-info (5.2.0.dfsg.1-6) ...
Processing triggers for man-db (2.7.0.2-5) ...
root@raspberrypi:~# apt-get autoclean -y -qq
root@raspberrypi:~# 
```

不要なサービスの停止
--------

RaspberryPiの起動時間を短縮するため、不要なサービスを停止します

```
root@raspberrypi:~# systemctl stop avahi-daemon
root@raspberrypi:~# systemctl disable avahi-daemon
Synchronizing state for avahi-daemon.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d avahi-daemon defaults
Executing /usr/sbin/update-rc.d avahi-daemon disable
insserv: warning: current start runlevel(s) (empty) of script `avahi-daemon' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `avahi-daemon' overrides LSB defaults (0 1 6).
insserv: warning: current start runlevel(s) (empty) of script `avahi-daemon' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `avahi-daemon' overrides LSB defaults (0 1 6).
Removed symlink /etc/systemd/system/dbus-org.freedesktop.Avahi.service.
Removed symlink /etc/systemd/system/sockets.target.wants/avahi-daemon.socket.
root@raspberrypi:~# 
```
```
root@raspberrypi:~# systemctl stop triggerhappy
root@raspberrypi:~# systemctl disable triggerhappy
Synchronizing state for triggerhappy.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d triggerhappy defaults
Executing /usr/sbin/update-rc.d triggerhappy disable
insserv: warning: current start runlevel(s) (empty) of script `triggerhappy' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (0 1 2 3 4 5 6) of script `triggerhappy' overrides LSB defaults (0 1 6).
root@raspberrypi:~# 
```

タイムゾーンの変更
--------

RaspbianはデフォルトのタイムゾーンはUTCになっているため、JSTに変更します。

```
root@raspberrypi:~# timedatectl set-timezone Asia/Tokyo
```

ログイン後バナーの変更
--------

必要に応じてログイン後に表示されるバナーを修正します。
ここに今回セットアップするRaspbianの役割やREADME的なものを書いておくと後々便利です。

```
root@raspberrypi:~# echo '' > /etc/motd
root@raspberrypi:~# 
```

SSHデーモンの有効化
---------

SSHデーモンを起動し、シリアル接続をしなくてもつながるようにします。

```
root@raspberrypi:~# systemctl enable ssh
Synchronizing state for ssh.service with sysvinit using update-rc.d...
Executing /usr/sbin/update-rc.d ssh defaults
insserv: warning: current start runlevel(s) (empty) of script `ssh' overrides LSB defaults (2 3 4 5).
insserv: warning: current stop runlevel(s) (2 3 4 5) of script `ssh' overrides LSB defaults (empty).
Executing /usr/sbin/update-rc.d ssh enable
Created symlink from /etc/systemd/system/sshd.service to /lib/systemd/system/ssh.service.
root@raspberrypi:~# 
root@raspberrypi:~# systemctl start ssh
root@raspberrypi:~# 
root@raspberrypi:~# systemctl status ssh | grep Active: 
   Active: active (running) since Sat 2016-12-31 15:00:56 JST; 3min 17s ago
root@raspberrypi:~# 
```

仕上げに再起動する
--------

最後に再起動して接続の確認を始めます。

```
root@raspberrypi:~# reboot
```

ネットワーク接続が成功していれば、起動中に`My IP address is 192.168.0.12 `のような形でRaspberryPiのIPアドレスが表示されます。


SSHでの接続確認
--------

ログインプロンプトが出ていることを確認したら、ターミナルをもう一つ開いてSSHしてみましょう。

下記は、RaspberryPiに192.168.0.12が割り当てられた場合の例です。

```
bash-3.2$ ssh 192.168.0.12 -l pi
The authenticity of host '192.168.0.12 (192.168.0.12)' can't be established.
ECDSA key fingerprint is SHA256:+rgt1e7lkFflZ4snx0boVf3uJ0mKUMjfdAKY1VlD3ms.
Are you sure you want to continue connecting (yes/no)? 
```

yesと応答し、パスワード入力画面でパスワードを入力します。

```
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.12' (ECDSA) to the list of known hosts.
pi@ 192.168.0.12's password: 
```

パスワードはログインプロンプト同様に`raspberry`です。

その後、ログインできればSSHの設定完了です。

```
Last login: Sat Dec 31 11:33:17 2016

SSH is enabled and the default password for the 'pi' user has not been changed.
This is a security risk - please login as the 'pi' user and type 'passwd' to set a new password.

pi@raspberrypi:~ $ 
```

この場は一度ログアウトします。

```
pi@raspberrypi:~ $ logout
Connection to 192.168.0.12 closed.
bash-3.2$ 
bash-3.2$ exit
```

シリアル接続の切断
--------

SSHで接続できるようになったので、`screen`を終了し、シリアル接続を切断します。

ログインプロンプトが出ているターミナルで、`Control`+`a`を入力した後、`k`を入力します。

画面の下に`Really kill this window [y/n]`と出てくるので、`y`を入力します

```
Raspbian GNU/Linux 8 raspberrypi ttyAMA0

raspberrypi login:




Really kill this window [y/n] y
[screen is terminating]
bash-3.2$ 
```

screenが切断され、元のターミナルに戻ってきます。

ここまできたらGPIOからシリアルアダプタを取り外して完了です。


今後
--------

今後はSSHベースでの作業になるので、パスワードの変更等必要に応じて行いましょう。

