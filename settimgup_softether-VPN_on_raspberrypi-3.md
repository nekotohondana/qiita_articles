---
title: Raspberry PiにSoftether VPNサーバーをセットする
tags: RaspberryPi
author: dauuricus
slide: false
---

2018.1.24.
##### Raspberry PiにRASPBIAN STRETCH LITE OSをセットし、Softether VPNサーバーをセットするまでを簡潔に説明無しでメモとして残す。

コマンドを全て順番どおりにターミナルにコピーペーストすれば４０分弱で全て完了するように記載できたら良いと思う。コマンドの箇所を別のテキストファイルにコピーし、そこからターミナルへペーストして作業すれば、ダウンロードにかかる時間や、Raspberry Piのパワーにもよるが４０分程度で完了できると思われる。

[Raspberry Pi公式ページ](https://www.raspberrypi.org/downloads/raspbian/)から、
RASPBIAN STRETCH LITE
Minimal image based on Debian Stretch
をダウンロードする。

[Etcher](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)などを使ってダウンロードしたRASPBIAN STRETCH LITEの.imgファイルを用意したＳＤカードに書き込む。

Raspberry PiにＳＤカードを挿しこみ、電源投入し、起動させる。
有線LANでRaspberry Piを接続し、[他のネットワーク接続機器からRaspberry PiのIPアドレスを探す。](https://qiita.com/dauuricus/items/8453e70b54ab64f22f2d#fing)
PCからsshで接続する。ただし、これは可能であればであって、デフォルトでsshがenableになっている場合は、ssh接続が可能だが、そうでない場合は、ＳＤカードにsshをenableにする書き込みを事前にしておく必要がある。

```ruby:PCターミナル
$ sudo ssh pi@192.168.0.4
```

など、LAN内のローカルアドレスは、Raspberry Piに割りふられたIPに適時書き換えてほしい。
入力すると、sudoの為、rootアカウントのパスワードが求められる。

その後、Raspberry Piのpiユーザーのパスワードが求められる。つまりこれは、初期値でユーザー`pi` パスワード`raspberry`になっている。

パスワードが通ると、プロンプトが

```ruby:PCターミナル
pi@raspberrypi:~ $
```
となる。
### ターミナル用日本語フォント
日本語フォントをインストールしておく。ただし、これはロケールを日本語にする場合に、文字化けがひどく色々な処理が非常に確認しづらくなるためであって、日本語に特にこだわりがなければ必要があるわけではない。

```ruby:pi@raspberrypi:~
 $ sudo apt-get install fonts-ipafont fonts-ipaexfont 
```

```ruby:pi@raspberrypi:~
 $ sudo raspi-config 
```

![Terminal - pi@raspberrypi: ~_157.png](https://qiita-image-store.s3.amazonaws.com/0/225786/2d0b6a8d-85e9-3c7c-e0a8-3f1446f042ac.png)


![Terminal - pi@raspberrypi: ~_153.png](https://qiita-image-store.s3.amazonaws.com/0/225786/b7d1c150-f43e-4ad5-68e6-8875f3c16dcd.png)

![Terminal - pi@raspberrypi: ~_154.png](https://qiita-image-store.s3.amazonaws.com/0/225786/b901ee30-8a5d-7580-6409-44c86703b997.png)

![Terminal - pi@raspberrypi: ~_155.png](https://qiita-image-store.s3.amazonaws.com/0/225786/a5e0c7e0-aeb9-246d-c325-0851a8c3b65b.png)

![Terminal - pi@raspberrypi: ~_156.png](https://qiita-image-store.s3.amazonaws.com/0/225786/e5a8ce03-c275-b4d0-806e-b3edea8324d8.png)

無線接続をしない場合は、以上の箇所の設定が済めば、一度RASPBIANを再起動する。

```ruby:pi@raspberrypi:~
$ sudo reboot
```

rebootされRASPBIANが起動したら（モニターレスでは確認できないが、なんとなく起動された頃合いをみはからって）、

```ruby:PCターミナル
 $ sudo ssh pi@192.168.0.4
```

などとして、Raspberry Piに接続を試みて無事接続されたら以下のコマンドで、後ほどダウンロードするSoftetherの`Make`に使う`build-essential`と、[ローカルブリッジ](https://ja.softether.org/4-docs/1-manual/3/3.6#3.6.9_tap_.E3.83.87.E3.83.90.E3.82.A4.E3.82.B9.E3.81.AE.E4.BD.BF.E7.94.A8)をつくるための`bridge-utils`をインストールする。

```ruby:pi@raspberrypi:~
$ sudo apt-get install build-essential
$ sudo apt-get install bridge-utils
```

### interfaces編集
`/etc/network/interfaces`を編集する。Nanoを使う。
ただし、これは[ローカルブリッジ接続](https://ja.softether.org/4-docs/1-manual/3/3.6#3.6.9_tap_.E3.83.87.E3.83.90.E3.82.A4.E3.82.B9.E3.81.AE.E4.BD.BF.E7.94.A8)を前提としている。ローカルブリッジしなくても、ＶＰＮとして機能させることはできるようだが。

```ruby:pi@raspberrypi:~
$ cd /etc/network/
$ sudo nano ./interfaces
```

```ruby:/etc/network/interfaces
# loopback
auto lo
iface lo inet loopback

# Ethernet
auto eth0
iface eth0 inet manual

# Bridge
auto br0
iface br0 inet dhcp
bridge_ports eth0

allow-hotplug wlan0
iface wlan0 inet manual
wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
```
nanoで以上をコピーしてペースト。そして、`interfaces`を上書き保存する。

###Softetherダウンロード
RASPBIANにsshで接続しているＰＣのブラウザで[SoftEther VPNダウンロードページ](http://ja.softether.org/5-download)からたどって、
[OS: Linux, CPU: ARM EABI (32bit)](http://www.softether-download.com/ja.aspx?product=softether)のビルド版のダウンロードアドレスをコピーする。ターミナル内にペーストするのは、ssh接続しているRASPBIANのコマンドプロンプトで`wget`コマンドの後に`Ctrl`+`Shift`+`v`でペーストされる。
下記では、リリース日: 2017-12-21のSoftEther VPN Server (Ver 4.24, Build 9652, beta)をダウンロードし、Makeしている。

```ruby:pi@xxx.xxx.xxx.xxx
$ wget http://jp.softether-download.com/files/softether/v4.24-9652-beta-2017.12.21-tree/Linux/SoftEther_VPN_Server/32bit_-_ARM_EABI/softether-vpnserver-v4.24-9652-beta-2017.12.21-linux-arm_eabi-32bit.tar.gz
```

### make
ダウンロードしたSoftetherのファイルを`tar`で解凍し、解凍してできた`vpnserver`を、`/usr/local/`へ移動し、`make`する。

```ruby:pi@raspberrypi
$ tar -zxvf softether-vpnserver-v4.24-9652-beta-2017.12.21-linux-arm_eabi-32bit.tar.gz

$ rm -v softether-vpnserver-v4.24-9652-beta-2017.12.21-linux-arm_eabi-32bit.tar.gz

$ sudo mv vpnserver /usr/local/
$ cd /usr/local/vpnserver
$ make
```

```ruby:pi@raspberrypi
cd /usr/local/vpnserver
chmod 600 *
chmod 700 vpncmd
chmod 700 vpnserver
```
```ruby:pi@raspberrypi
$ cd /etc/init.d/
$ sudo nano ./vpnserver
```
### 起動スクリプト
以下のスクリプトをコピーして、`vpnserver`へペースト。

```
#!/bin/sh
### BEGIN INIT INFO
# Provides:                 vpnserver
# Required-Start:           $local_fs $network
# Required-Stop:            $local_fs $network
# Default-Start:            2 3 4 5
# Default-Stop:             0 1 6
# Short-Description:        SoftEther VPN RTM
# Description:              Start vpnserver daemon SoftEther VPN Server
### END INIT INFO

DAEMON=/usr/local/vpnserver/vpnserver
LOCK=/var/lock/vpnserver

# tun/tapモジュールのロード
sudo modprobe tun

. /lib/lsb/init-functions
test -x $DAEMON || exit 0

case "$1" in
start)
sleep 3
log_daemon_msg "Starting SoftEther VPN Server" "vpnserver"
$DAEMON start >/dev/null 2>&1
touch $LOCK
log_end_msg 0
sleep 3

# SoftEther VPNで追加した仮想tapデバイス名を調べる
tap=`/sbin/ifconfig -a| awk '$1 ~ /^tap/ {print $1}'`
/sbin/brctl addif br0 $tap
;;

stop)
log_daemon_msg "Stopping SoftEther VPN Server" "vpnserver"
$DAEMON stop >/dev/null 2>&1
rm $LOCK
log_end_msg 0
sleep 2
;;

restart)
$DAEMON stop
sleep 2

$DAEMON start
sleep 5
# SoftEther VPNで追加した仮想tapデバイス名を調べる
tap=`/sbin/ifconfig -a| awk '$1 ~ /^tap/ {print $1}'`
/sbin/brctl addif br0 $tap
;;

status)
    if [ -e $LOCK ]
    then
        echo "vpnserver is running."
    else
        echo "vpnserver is not running."
    fi
;;
*)

echo "Usage: $0 {start|stop|restart|status}"
exit 1
esac
exit 0
```
実行権限を与えてデーモンとして登録。

```ruby:pi@raspberrypi
sudo chmod +x /etc/init.d/vpnserver
sudo update-rc.d vpnserver defaults
sudo reboot
```
ここまでで、RASPBIANでSoftether VPNサーバーが起動できるようになりました。

Raspberry Piで使えそうな[プロミスキャスのusb wifi card](https://ja.softether.org/4-docs/3-kb/VPNFAQ003)を探しています。
