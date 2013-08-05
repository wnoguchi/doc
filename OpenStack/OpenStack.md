# OpenStack導入メモ

OpenStackを導入するにあたって参考にさせていただいたサイトや手順等をメモしていきます。

## インストール

### System Requirements

- CentOS 6.4 x64

### IPアドレス固定化

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.168.0.10
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```

### IPv6無効化

コメントアウトもしくは削除。

```
# /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

### SELinux無効化

省略。

#### aaa

```
yum install -y ntp man openssh-clients &&
service ntpd start &&
chkconfig ntpd on
```

#### epel

epel入れといてね。

```
yum install -y mysql-server memcached
service mysqld start &&
chkconfig mysqld on &&
mysql -uroot -e "set password for root@localhost=password('nova');"
mysql -uroot -pnova -e "set password for root@127.0.0.1=password('nova');"
mysql -uroot -pnova -e "set password for root@stack01=password('nova');"
```

![](img/create_instance.png)
![](img/dashboard.png)
![](img/float_ip.png)
![](img/images.png)
![](img/instance_active.png)
![](img/login.png)
![](img/serial.png)

## 参考サイト

### Main

- [3. Openstackインストール手順(Grizzly)CentOS6.4(パッケージ)編 —
オープンソースに関するドキュメント 1.1
documentation](http://oss.fulltrust.co.jp/doc/openstack_grizzly_centos64_yum/)
- [オープンソースに関するドキュメント — オープンソースに関するドキュメント 1.1
documentation](http://oss.fulltrust.co.jp/doc/index.html)

### Subs.

- [さくらの専用サーバとOpenStackで作るプライベートクラウド | SourceForge.JP Magazine](http://sourceforge.jp/magazine/12/09/18/1126211)
- [OpenStack 運用ガイド(PDF)](http://dream.daynight.jp/openstack/openstack-ops/openstack-ops-manual-local.pdf)
- [これからはじめるOpenStackリンク集 | 外道父の匠](http://blog.father.gedow.net/2013/02/19/openstack-links/)
- [OpenStack の利用](http://www.slideshare.net/yosshy/openstack-14884093)
- [OpenStack勉強会](http://www.slideshare.net/obara13/open-stack-16166193)
