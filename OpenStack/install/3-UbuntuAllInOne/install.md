# UbuntuにAllInOne構成でOpenStackをインストール(Grizzly)

DevStackを使う。

## インストール

```
sudo apt-get -y update
sudo apt-get -y upgrade
sudo apt-get -y install git
```

### ユーザーstackを追加してパス無しsudoできるようにする

```
sudo adduser stack
sudo visudo
```

```
stack ALL=(ALL) NOPASSWD: ALL
```

```
logout
```

以降はユーザーstackでログインして作業。

```
git clone git://github.com/openstack-dev/devstack.git
cd devstack
git checkout stable/grizzly
```

```
cat <<EOF >localrc
FLOATING_RANGE=192.168.0.96/27
FIXED_RANGE=10.11.12.0/24
FIXED_NETWORK_SIZE=256
FLAT_INTERFACE=eth0
ADMIN_PASSWORD=supersecret
MYSQL_PASSWORD=iheartdatabases
RABBIT_PASSWORD=flopsymopsy
SERVICE_PASSWORD=iheartksl
EOF
```

パスワード入力を促されるがそのまま空エンターでOK。

```
./stack.sh
```

## 参考リンク

- [Single Machine Guide - DevStack](http://devstack.org/guides/single-machine.html)
- [openstack-dev/devstack](https://github.com/openstack-dev/devstack/tree/stable/grizzly)
- [「オープンソース」を使ってみよう (第23回 DevStackでラクラク導入！ OpenStackを使ってみよう編)](http://www.ospn.jp/press/20120828no27-useit-oss.html)
