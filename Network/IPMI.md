# IPMI

LANから電源オンオフ。

## サーバー側

```
yum -y install OpenIPMI OpenIPMI-tools
service ipmi start && chkconfig ipmi on
ls -l /dev/ipmi*
crw-rw----. 1 root root 247, 0  8月  9 00:06 2013 /dev/ipmi0
```

スペシャルデバイスファイルが生成されていることが確認出来ればおっけー。

・現在のユーザリスト
ipmitool user list "user id"

・ユーザの設定
ipmitool user set name "user id" "user name"

・パスワードの設定
ipmitool user set password "user id" "password"

・admin権限の付与（ユーザID,権限の付与等は環境に応じて変えてください。）
ipmitool user priv "user id" 4 1

・作成したユーザの有効化
ipmitool user enable "user id"

・現在の設定の確認
ipmitool lan print "channel id"

・staticにする
ipmitool lan set "channel id" ipsrc static

・IPアドレス設定
ipmitool lan set "channel id" ipaddr "IP ADDRESS"

・サブネットマスク設定
ipmitool lan set "channel id" netmask "NETMASK"

・DefaultGateway設定
ipmitool lan set "channel id" defgw ipaddr "IP ADDRESS"

・アクセス許可
ipmitool lan set "channel id" access on
以上でユーザの設定、IPアドレスの設定が完了し使用できる状態になりました。

```
ipmitool user list
ipmitool user set name "2" "admin"
ipmitool user set password "2" "password"
ipmitool user priv "2" 4 1
ipmitool user enable "2"
ipmitool lan print "2"
ipmitool lan set "2" ipsrc static
ipmitool lan set "2" ipaddr "192.168.0.55"
ipmitool lan set "2" netmask "255.255.255.0"
ipmitool lan set "2" defgw ipaddr "192.168.0.1"
ipmitool lan set "2" access on
```


## 管理端末側

当然IPMI toolsのインストールが必要となります。

```
yum -y install OpenIPMI-tools
```

ためしにpingが通じるかやってみる。

```
[root@localhost dahdi-linux]# ping 192.168.0.55
PING 192.168.0.55 (192.168.0.55) 56(84) bytes of data.
64 bytes from 192.168.0.55: icmp_seq=1 ttl=128 time=19.1 ms
64 bytes from 192.168.0.55: icmp_seq=2 ttl=128 time=4.61 ms
64 bytes from 192.168.0.55: icmp_seq=3 ttl=128 time=3.87 ms
^C
--- 192.168.0.55 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2558ms
rtt min/avg/max/mdev = 3.871/9.212/19.152/7.035 ms
```

おお、通った。

### 電源OFF

```
[root@localhost dahdi-linux]# ipmitool  -H "192.168.0.55" -U "admin" power off
Password: 
Chassis Power Control: Down/Off
```

カリカリカリカリ・・・パチンッ。  
おーーー。シャットダウンできた。

**ただし、電源オフに関してはあらゆる操作を行なってどうしてもOSを正常終了できなくなった場合に
行うオペレーションなので、むやみやたらにやらないほうがよさそう。
再インストールしてちょっとyumかけたやつにやったらCentOS起動しなくなった。。。
再インストール直後でipmi-tools入れてなかったからだろうか？**

```
[root@localhost dahdi-linux]# ping 192.168.0.55
PING 192.168.0.55 (192.168.0.55) 56(84) bytes of data.
64 bytes from 192.168.0.55: icmp_seq=1 ttl=128 time=8.31 ms
64 bytes from 192.168.0.55: icmp_seq=2 ttl=128 time=3.81 ms
^C
--- 192.168.0.55 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1342ms
rtt min/avg/max/mdev = 3.810/6.060/8.310/2.250 ms
```

落ちてもまだpingが通じる。メンテナンスポートだもんね。

### 電源ON

```
[root@localhost dahdi-linux]# ipmitool  -H "192.168.0.55" -U "admin" power on
Password: 
Chassis Power Control: Up/On
```

普通にサーバー起動した。感動。
これで、サーバが落ちてしまった！！なんてときにも、DCに走らなくてもよくなりました。よかったよかった。（ハード故障のときはだめなケースもありますが…）

### 電源リセット

```
ipmitool  -H "192.168.1.55" -U "admin" power reset
```

### パスワード直指定する例

```
ipmitool  -H "192.168.1.55" -U "admin" -P password power reset
```

## その他

ちなみに、ホストOSを再インストールしてもIPMIの設定はサーバーの不揮発メモリに残っているので同じ情報で電源オンオフできます。
便利。

## WindowsマシンからIPMIで電源オンオフしたい

Windows版のIPMI Toolはないのか。  
あった。

[ipmiutil - IPMI Management Utilities](http://ipmiutil.sourceforge.net/)

MSIでインストールしたら楽なのかなって思ってインストールしたらどこにインストールされたのかわからない。。。  
アイコンも生成されないし、パスも通ってない、それっぽい名前のフォルダも見つからない。  
仕方ないのでlocate32でファイル検索したら `C:\Program Files (x86)\sourceforge` にインストールされてた。。。  
これは非常にわかりにくい。

IPMI Toolとは微妙にコマンド体系違うっぽい。

```
C:\Program Files (x86)\sourceforge\ipmiutil>ipmiutil power -r -N 192.168.10.55 -U admin -P password
ipmiutil ver 2.91
ireset ver 2.91
Opening lan connection to node 192.168.10.55 ...
Connecting to node  192.168.10.55
-- BMC version 4.10, IPMI version 2.0
Power State      = 00   (S0: working)
ireset: resetting ...
chassis_reset ok
ireset: IPMI_Reset ok
ipmiutil power, completed successfully
```

## MacからIPMIで電源オンオフしたい

そんなことができるのか。  
多分無いだろうとおもったけど一応homebrewで探してみる。

```
localhost:~ noguchiwataru$ brew search ipmi
ipmitool  ipmiutil  ripmime
```

あった！

```
localhost:~ noguchiwataru$ brew install ipmitool
==> Downloading http://downloads.sourceforge.net/project/ipmitool/ipmitool/1.8.1
######################################################################## 100.0%
==> Patching
patching file configure
Hunk #1 succeeded at 5030 with fuzz 1.
==> ./configure --prefix=/usr/local/Cellar/ipmitool/1.8.12 --mandir=/usr/local/C
==> make install
🍺  /usr/local/Cellar/ipmitool/1.8.12: 14 files, 1.2M, built in 20 seconds
```

そして、

```
localhost:~ noguchiwataru$ ipmitool -H "192.168.1.55" -U "admin" -P password power reset
Chassis Power Control: Reset
```

たまらないですね。

## 参考リンク

- [大量のサーバを管理するために、IPMIのお話｜サイバーエージェント 公式エンジニアブログ](http://ameblo.jp/principia-ca/entry-10983675114.html)
- [ipmiutil - IPMI Management Utilities](http://ipmiutil.sourceforge.net/)
