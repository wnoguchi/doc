# PXE boot

## UbuntuをPXE bootでインストールする

```
sudo apt-get -y install dhcp3-server
```

```
sudo apt-get -y install tftpd-hpa tftp inetutils-inetd
```

```
cd /var/lib/tftpboot
sudo wget http://archive.ubuntu.com/ubuntu/dists/precise/main/installer-amd64/current/images/netboot/netboot.tar.gz
sudo tar xvzf netboot.tar.gz
./
./pxelinux.0
./version.info
./pxelinux.cfg
./ubuntu-installer/
./ubuntu-installer/amd64/
./ubuntu-installer/amd64/pxelinux.0
./ubuntu-installer/amd64/initrd.gz
./ubuntu-installer/amd64/boot-screens/
./ubuntu-installer/amd64/boot-screens/f10.txt
./ubuntu-installer/amd64/boot-screens/f4.txt
./ubuntu-installer/amd64/boot-screens/rqtxt.cfg
./ubuntu-installer/amd64/boot-screens/vesamenu.c32
./ubuntu-installer/amd64/boot-screens/adtxt.cfg
./ubuntu-installer/amd64/boot-screens/exithelp.cfg
./ubuntu-installer/amd64/boot-screens/f8.txt
./ubuntu-installer/amd64/boot-screens/f3.txt
./ubuntu-installer/amd64/boot-screens/menu.cfg
./ubuntu-installer/amd64/boot-screens/splash.png
./ubuntu-installer/amd64/boot-screens/f5.txt
./ubuntu-installer/amd64/boot-screens/txt.cfg
./ubuntu-installer/amd64/boot-screens/syslinux.cfg
./ubuntu-installer/amd64/boot-screens/stdmenu.cfg
./ubuntu-installer/amd64/boot-screens/f2.txt
./ubuntu-installer/amd64/boot-screens/f7.txt
./ubuntu-installer/amd64/boot-screens/prompt.cfg
./ubuntu-installer/amd64/boot-screens/f1.txt
./ubuntu-installer/amd64/boot-screens/f9.txt
./ubuntu-installer/amd64/boot-screens/f6.txt
./ubuntu-installer/amd64/pxelinux.cfg/
./ubuntu-installer/amd64/pxelinux.cfg/default
./ubuntu-installer/amd64/linux

```

```
sudo vi /etc/default/dhcp3-server
INTERFACES="eth0"
```

```
sudo vi /etc/dhcp/dhcpd.conf

(snip)

↓以下を追加

allow booting;
allow bootp;

subnet 192.168.1.0 netmask 255.255.255.0 {
        range 192.168.1.2 192.168.1.100;
        option routers 192.168.1.1;
        option broadcast-address 192.168.1.255;
        option domain-name-servers 8.8.8.8;
}
host rx100s7 {
        filename "/pxelinux.0";
        #hardware ethernet 00:0B:xx:xx:xx:xx;
        fixed-address 192.168.1.70;
        }
```

```
sudo /etc/init.d/isc-dhcp-server restart
Rather than invoking init scripts through /etc/init.d, use the service(8)
utility, e.g. service isc-dhcp-server restart

Since the script you are attempting to invoke has been converted to an
Upstart job, you may also use the stop(8) and then start(8) utilities,
e.g. stop isc-dhcp-server ; start isc-dhcp-server. The restart(8) utility is also available.
isc-dhcp-server start/running, process 26580

```

## 参考リンク

- [PXEでDebian/Ubuntuをネットワークインストール - Shoken OpenSource Society](http://shoken.hatenablog.com/entry/20080306/p1)
- [PXE使ってUbuntuをネットワークインストール - モーグルとカバとパウダーの日記](http://d.hatena.ne.jp/stealthinu/20110726/p1)

