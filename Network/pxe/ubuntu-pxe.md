# PXEブートサーバーの構築（Ubuntu PXEサーバー編）

VMware Fusionでもうまくいく。  
うん、こんどはaptのミラーサーバー立ててインスコしたい。

## 構成

- PXEブートサーバー: Ubuntu13.04

## インストール

### DHCP関連

```
sudo apt-get -y install dhcp3-server
```

### TFTPD関連

```
sudo apt-get -y install tftpd-hpa tftp inetutils-inetd
```

## TFTP関連の設定

Ubuntu12.04(Precise)のブートイメージをダウンロードしてきてTFTP bootのルートに展開。

```
cd /var/lib/tftpboot
sudo wget --inet4-only http://archive.ubuntu.com/ubuntu/dists/precise/main/installer-amd64/current/images/netboot/netboot.tar.gz
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

以下の設定で動いた。

```
# /etc/dhcp/dhcpd.conf

authoritative;

subnet 192.168.1.0 netmask 255.255.255.0 {
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    range dynamic-bootp 192.168.1.2 192.168.1.100;
    filename "/pxelinux.0";
}
```

```
sudo initctl start isc-dhcp-server
```

## 参考リンク

- [PXEでDebian/Ubuntuをネットワークインストール - Shoken OpenSource Society](http://shoken.hatenablog.com/entry/20080306/p1)
- [PXE使ってUbuntuをネットワークインストール - モーグルとカバとパウダーの日記](http://d.hatena.ne.jp/stealthinu/20110726/p1)
- [PXEInstallServer - Community Ubuntu Documentation](https://help.ubuntu.com/community/PXEInstallServer)
- [ネットワークで生きる ubuntuでubuntuをネットワークブートでインストール(12.04LTS・dhcp&tftp版)](http://hukuroufc2.blog.fc2.com/blog-entry-52.html)
- [Ubuntu 13.04 - DHCPサーバー ： Server World](http://www.server-world.info/query?os=Ubuntu_13.04&p=dhcp)

### preseedに関係してくる

- [System Automation – Part 1 – PXE and Preseed](http://www.briancarpio.com/2012/04/04/system-automation-part-1/)
- [Ubuntu Server を pxe(netboot)+preseed で全自動インストールする - INOHILOG](http://d.hatena.ne.jp/InoHiro/20110830/1314710048)
- [preseedを使ってDebian GNU/Linux 5.0.4(netinst)のインストール自動化を行う手順 - 富士山は世界遺産](http://d.hatena.ne.jp/fujisan3776/20100630/1277861431)

### 横道それる

- [Bootstrap Protocol - Wikipedia](http://ja.wikipedia.org/wiki/Bootstrap_Protocol)
- [BOOTPとは 【 BOOTstrap Protocol 】 - 意味/解説/説明/定義 ： IT用語辞典](http://e-words.jp/w/BOOTP.html)
- [Man page of dhcpd.conf](http://linuxjm.sourceforge.jp/html/dhcp2/man5/dhcpd.conf.5.html)
