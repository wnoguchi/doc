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
(snip)
[ERROR] ./stack.sh:638 nova-api did not start
```

なんやねん・・・。

[python - Error in devstack script. nova-api did not start? - Stack Overflow](http://stackoverflow.com/questions/17079919/error-in-devstack-script-nova-api-did-not-start)

もういっかい。

```
./unstack.sh
./stack.sh
```

回線細すぎてエラーになる。  
一回手動でnovaのリポジトリはとってこよう。

```
git clone https://github.com/openstack/nova.git /opt/stack/nova
error: RPC failed; result=18, HTTP code = 200 MiB | 32 KiB/s   
fatal: The remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

HTTPSだとだめみたい。
gitプロトコルに切り替える。

```
stack@wstack:~/devstack$ git clone git://github.com/openstack/nova.git /opt/stack/nova
Cloning into '/opt/stack/nova'...
remote: Counting objects: 199622, done.
remote: Compressing objects: 100% (50544/50544), done.
remote: Total 199622 (delta 159731), reused 182185 (delta 143338)
Receiving objects: 100% (199622/199622), 122.27 MiB | 96 KiB/s, done.
Resolving deltas: 100% (159731/159731), done.
```

```
./unstack.sh
./stack.sh


Horizon is now available at http://192.168.0.10/
Keystone is serving at http://192.168.0.10:5000/v2.0/
Examples on using novaclient command line is in exercise.sh
The default users are: admin and demo
The password: supersecret
This is your host ip: 192.168.0.10
stack.sh completed in 245 seconds.
```

とりあえずうまくいった。
cirros立ち上げてセキュリティグループも編集したけどping飛ばない。  
ログ見てみた。

```
checking http://169.254.169.254/2009-04-04/instance-id
failed 1/20: up 1.06. request failed
failed 2/20: up 6.07. request failed
failed 3/20: up 9.07. request failed
failed 4/20: up 14.07. request failed
failed 5/20: up 17.07. request failed
failed 6/20: up 22.08. request failed
failed 7/20: up 25.08. request failed
failed 8/20: up 30.09. request failed
failed 9/20: up 33.09. request failed
failed 10/20: up 38.10. request failed
failed 11/20: up 41.10. request failed
failed 12/20: up 46.11. request failed
```

たぶんNICまわりだろうなあ・・・。

```
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service neutron
# Optional, to enable tempest configuration as part of devstack
enable_service tempest
```

かなり何度もunstack and stackを繰り返すことになりそうなのでaliasを定義する。

```
./unstack.sh && ./stack.sh
alias restack='./unstack.sh && ./stack.sh'
```

DevStackはタイミングの問題でエラーになることが多々あるから何回か打ちなおしてみるのもいいんじゃないですかね。

**うお、Quantum追加したらなんかメニュー増えた。**

![](img/quantum1.png)

![](img/quantum2.png)

これはやばいですね・・・。

## 参考リンク

- [Single Machine Guide - DevStack](http://devstack.org/guides/single-machine.html)
- [openstack-dev/devstack](https://github.com/openstack-dev/devstack/tree/stable/grizzly)
- [「オープンソース」を使ってみよう (第23回 DevStackでラクラク導入！ OpenStackを使ってみよう編)](http://www.ospn.jp/press/20120828no27-useit-oss.html)
