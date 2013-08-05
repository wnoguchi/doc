# OpenStack導入メモ

OpenStackを導入するにあたって参考にさせていただいたサイトや手順等をメモしていきます。

## インストール

### System Requirements

- CentOS 6.4 x64

### Installation

#### IPアドレス固定化

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.168.0.10
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```

#### IPv6無効化

コメントアウトもしくは削除。

```
# /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

#### SELinux無効化

省略。

#### aaa

```
yum install -y ntp man openssh-clients &&
service ntpd start &&
chkconfig ntpd on
```

epel入れといてね。

```
yum install -y mysql-server memcached
service mysqld start &&
chkconfig mysqld on &&
mysql -uroot -e "set password for root@localhost=password('nova');"
mysql -uroot -pnova -e "set password for root@127.0.0.1=password('nova');"
mysql -uroot -pnova -e "set password for root@stack01=password('nova');"
```

## 参考サイト

- [3. Openstackインストール手順(Grizzly)CentOS6.4(パッケージ)編 —
オープンソースに関するドキュメント 1.1
documentation](http://oss.fulltrust.co.jp/doc/openstack_grizzly_centos64_yum/)
- [オープンソースに関するドキュメント — オープンソースに関するドキュメント 1.1
documentation](http://oss.fulltrust.co.jp/doc/index.html)

