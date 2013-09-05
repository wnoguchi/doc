# PXEブートサーバーの構築（CentOSサーバー編）

PXEクライアントではUbuntuをインストールします。  
preseed読み込ませるところまでやりたいなあ。

## インストール

```
yum -y install system-config-netboot syslinux xinetd tftp-server 
yum -y install dhcp
```

## TFTPD

```
vi /etc/xinetd.d/tftp 

        disable                 = no

service xinetd start
chkconfig xinetd on 

```

## DHCP

```
cp -f /usr/share/doc/dhcp*/dhcpd.conf.sample /etc/dhcp/dhcpd.conf
```

```
vi /etc/dhcp/dhcpd.conf
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.50 192.168.1.59;
  option routers 192.168.1.1;

  filename "/pxeboot/pxelinux.0";
  next-server 192.168.1.30;
}
```


```diff
diff --git a/dhcpd.conf b/dhcpd.conf
index 5eab951..14ecb18 100644
--- a/dhcpd.conf
+++ b/dhcpd.conf
@@ -24,14 +24,18 @@ log-facility local7;
 # No service will be given on this subnet, but declaring it helps the 
 # DHCP server to understand the network topology.
 
-subnet 10.152.187.0 netmask 255.255.255.0 {
+subnet 192.168.1.0 netmask 255.255.255.0 {
 }
 
 # This is a very basic subnet declaration.
 
-subnet 10.254.239.0 netmask 255.255.255.224 {
-  range 10.254.239.10 10.254.239.20;
-  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
+subnet 192.168.1.0 netmask 255.255.255.0 {
+  range 192.168.1.50 192.168.1.59;
+  option routers 192.168.1.1;
+
+  filename "/pxeboot/pxelinux.0";
+  next-server 192.168.1.30;
+
 }
 
 # This declaration allows BOOTP clients to get dynamic addresses,

```

### syslinux

```
mkdir -p /var/lib/tftpboot/pxeboot/
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/pxeboot/
mkdir -p /var/lib/tftpboot/pxeboot/
```

編集。  
DHCP再起動。

```
service dhcpd restart
```

## ブートイメージを取得して展開(Ubuntu)

なぜかipv6で読み出そうとしてめちゃくちゃ時間がかかってたので強制的にipv4にした。

```
cd /var/lib/tftpboot/pxeboot
wget http://archive.ubuntu.com/ubuntu/dists/precise/main/installer-amd64/current/images/netboot/netboot.tar.gz --net4-only
tar xvzf netboot.tar.gz
```

クライアントをブートします。  
うまくいった。

### 参考サイト

- [CentOS 5 - PXEサーバー - 構築/設定 ： Server World](http://www.server-world.info/query?os=CentOS_5&p=pxe)
- [CentOS 5 - DHCPサーバー - インストール/設定 ： Server World](http://www.server-world.info/query?os=CentOS_5&p=dhcp)
- [PXEブート用サーバを構築する - maruko2 Note.](http://www.maruko2.com/mw/PXE%E3%83%96%E3%83%BC%E3%83%88%E7%94%A8%E3%82%B5%E3%83%BC%E3%83%90%E3%82%92%E6%A7%8B%E7%AF%89%E3%81%99%E3%82%8B)
- [第265回　Ubuntu Serverをさらっと用意する方法：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0265)
- [第47回　Ubuntuのネットワークインストールとapt-mirrorの活用：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0047)
- [第47回　Ubuntuのネットワークインストールとapt-mirrorの活用：Ubuntu Weekly Recipe｜gihyo.jp … 技術評論社](http://gihyo.jp/admin/serial/01/ubuntu-recipe/0047?page=2)
