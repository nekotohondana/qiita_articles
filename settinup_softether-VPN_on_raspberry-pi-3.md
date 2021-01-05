## Please note that this article is a translation of the Japanese tutorial written in 2018.
2018.1.24.
##### Set RASPBIAN STRETCH LITE OS on Raspberry Pi and

leave a memo without explanation briefly until setting Softether VPN
server.

If you copy and paste all the commands into the terminal in order, it
would be nice if you could write that everything would be completed in
less than 40 minutes. If you copy the command part to another text
file and paste it into the terminal from there, it will be completed
in about 40 minutes depending on the download time and the power of
the Raspberry Pi.

From [Raspberry Pi Official Page](https://www.raspberrypi.org/downloads/raspbian/) RASPBIAN STRETCH
LITE Minimal image based on Debian Stretch To download.

Write the .img file of RASPBIAN STRETCH LITE downloaded using [Etcher](https://www.raspberrypi.org/documentation/installation/installing-images/README.md)
to the prepared SD card.

Insert the SD card into the Raspberry Pi, turn on the power, and start
it.  Connect the Raspberry Pi with a wired LAN, and click [Search for
the Raspberry Pi's IP address from other network-connected devices. ](Https://qiita.com/dauuricus/items/8453e70b54ab64f22f2d#fing) Connect
with ssh from your PC. However, if this is possible, if ssh is enabled
by default, ssh connection is possible, but if not, it is necessary to
write to the SD card to enable ssh in advance. There is.

```ruby: PC terminal
$ sudo ssh pi @ 192.168.0.4
```

Please rewrite the local address in the LAN to the IP assigned to the
Raspberry Pi in a timely manner.  When you enter it, you will be asked
for the root account password because of sudo.

After that, you will be asked for the password for the Raspberry Pi pi
user. So this is by default the user `pi` password` raspberry`.

If the password passes, you will be prompted

```ruby: PC terminal pi
@ raspberrypi:
~ $
`````

### Japanese font for terminal 
Install Japanese fonts.
However, this is because when the locale is set to Japanese, the characters are garbled and it is very difficult to check various processes, and it is not necessary unless you are particular about Japanese.

```
ruby: pi @ raspberrypi:
~ $ sudo apt-get install fonts-ipafont fonts-ipaexfont
`````

```ruby: pi @ raspberrypi:
~ $ sudo raspi-config
`````

![Terminal--pi@raspberrypi:~_157.png](https://qiita-image-store.s3.amazonaws.com/0/225786/2d0b6a8d-85e9-3c7c-e0a8-3f1446f042ac.png)


![Terminal--pi@raspberrypi:~_153.png](https://qiita-image-store.s3.amazonaws.com/0/225786/b7d1c150-f43e-4ad5-68e6-8875f3c16dcd.png)

![Terminal--pi@raspberrypi:~_154.png](https://qiita-image-store.s3.amazonaws.com/0/225786/b901ee30-8a5d-7580-6409-44c86703b997.png)

![Terminal--pi@raspberrypi:~_155.png](https://qiita-image-store.s3.amazonaws.com/0/225786/a5e0c7e0-aeb9-246d-c325-0851a8c3b65b.png)

![Terminal--pi@raspberrypi:~_156.png](https://qiita-image-store.s3.amazonaws.com/0/225786/e5a8ce03-c275-b4d0-806e-b3edea8324d8.png)

If you do not want to connect wirelessly, restart RASPBIAN once you
have completed the above settings.

```ruby: pi @ raspberrypi: 
~ $ sudo reboot 
`````

After rebooting and RASPBIAN booting (I can't check it without a
monitor, but somehow when it boots up),

```ruby: PC terminal
$ sudo ssh pi @ 192.168.0.4
`````

For example, if you try to connect to Raspberry Pi and it is connected
successfully, use the following command to download `build-essential` for Softether `Make` and install `bridge-utils` for creating [Local Bridge].

```ruby: pi @ raspberrypi: ~
$ sudo apt-get install build-essential 
$ sudo apt-get install bridge-utils
`````

### interfaces edit 
Edit `/ etc / network / interfaces`. Use Nano.
However, this is [Local Bridge Connection] is assumed. It seems that it can function as a VPN without a local
bridge.

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

Copy and paste the above with nano. Then, overwrite and save
`interfaces`.

### Download Softether VPN source code
Download Follow from [SoftEther VPN download page](https://www.softether.org/5-download) with the browser of the PC
connected to RASPBIAN by ssh, Copy the download address of the build version of [OS: Linux, CPU: ARM EABI (32bit)](http://www.softether-download.com/en.aspx).
To paste in the terminal, press `Ctrl` + `Shift` + `v` after the `wget` command at the command prompt of RASPBIAN connected by ssh.  Below,
the release date: 2017-12-21 SoftEther VPN Server (Ver 4.24, Build 9652, beta) is downloaded and made.

```ruby: pi@xxx.xxx.xxx.xxx
$ wget http://jp.softether-download.com/files/softether/v4.24-9652-beta-2017.12.21-tree/Linux/SoftEther_VPN_Server/32bit_-_ARM_EABI/softether-vpnserver-v4.24-9652-beta-2017.12.21-linux-arm_eabi-32bit.tar.gz
`````

### make 
Unzip the downloaded Softether file with `tar`, move the unzipped `vpnserver` to `/ usr / local /`, and `make`.

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

### Startup script
Copy the script below and paste it into `vpnserver`.

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

# Load tun/tap
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

# Check the virtual tap device name added by SoftEther VPN
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
# Check the virtual tap device name added by SoftEther VPN
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

Register as a daemon with execute permission.

```ruby:pi@raspberrypi
sudo chmod +x /etc/init.d/vpnserver
sudo update-rc.d vpnserver defaults
sudo reboot
```
At this point, RASPBIAN can start the Softether VPN server.

I am looking for [Promiscuous usb wifi card](https://ja.softether.org/4-docs/3-kb/VPNFAQ003) that can be used with
Raspberry Pi.


Cf.
