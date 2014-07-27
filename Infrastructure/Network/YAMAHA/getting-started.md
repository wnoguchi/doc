初期設定
=========

管理者パスワードの設定
----------------------

- 出荷時設定に戻す。

```
cold start
```

- 管理者モード移行。

```
administrator
```

- 文字コードをASCIIに。Macとかだと文字化けしてかなわない。

```
console character ascii
```

- 一般ユーザーパスワード設定。

```
# login password 
Old_Password: ←空パスキーイン
New_Password: ←パスワード設定 
New_Password: ←パスワード設定（確認） 
```

- 管理者ユーザーパスワード設定。

```
# administrator password 
Old_Password: ←空パスキーイン 
New_Password: ←パスワード設定 
New_Password: ←パスワード設定（確認） 
```

- 保存を忘れずに。

```
# save 
Saving ... CONFIG0 Done .
```

その他もろもろ
---------------

- 1時間でログインが切れるようにする。

そのままだと5分ぐらいで切れて設定作業では支障が出るから。

```
login timer 3600
```

LAN1インタフェースにTCP/IP設定
------------------------------

- LAN2/LAN3はWAN側に割り当てる。

### LAN1のIPアドレスを設定する。

以下は `192.168.0.1` をサブネットマスク `255.255.255.0` で割り当てる設定。

```
ip lan1 address 192.168.0.1/24
```

この時点でもう telnet が使える。

```
% telnet 192.168.0.1
Trying 192.168.0.1...
Connected to 192.168.0.1.
Escape character is '^]'.

Password: 

RTX1200 Rev.10.01.53 (Mon Sep  9 15:23:58 2013)
  Copyright (c) 1994-2013 Yamaha Corporation. All Rights Reserved.
  Copyright (c) 1991-1997 Regents of the University of California.
  Copyright (c) 1995-2004 Jean-loup Gailly and Mark Adler.
  Copyright (c) 1998-2000 Tokyo Institute of Technology.
  Copyright (c) 2000 Japan Advanced Institute of Science and Technology, HOKURIKU.
  Copyright (c) 2002 RSA Security Inc. All rights reserved.
  Copyright (c) 1997-2010 University of Cambridge. All rights reserved.
  Copyright (C) 1997 - 2002, Makoto Matsumoto and Takuji Nishimura, All rights reserved.
  Copyright (c) 1995 Tatu Ylonen , Espoo, Finland All rights reserved.
  Copyright (c) 1998-2004 The OpenSSL Project.  All rights reserved.
  Copyright (C) 1995-1998 Eric Young (eay@cryptsoft.com) All rights reserved.
  Copyright (c) 2006 Digital Arts Inc. All Rights Reserved.
  Copyright (C) 1994-2012 Lua.org, PUC-Rio.
  Copyright (c) 1988-1992 Carnegie Mellon University All Rights Reserved.
  Copyright (C) 2004-2007 Diego Nehab. All rights reserved.
  Copyright (c) 2005 JSON.org
00:a0:de:6b:74:13, 00:a0:de:6b:74:14, 00:a0:de:6b:74:15
Memory 128Mbytes, 3LAN, 1BRI
> administrator 
Password: 
# 
```

SSHが使用したいところだが、もう少し我慢。

DHCPサーバーを有効化する
------------------------

```
dhcp service server 
dhcp server rfc2131 compliant except remain-silent 
dhcp scope 1 192.168.0.2-192.168.0.199/24 
```

DNSリレーの設定
----------------

### 個別に指定するとき

```
dns server 8.8.8.8 8.8.4.4 
dns private address spoof on
```

### ISPから自動取得する

インターリンクは自動取得が推奨されている。

```
# PPPoE接続のpp1インタフェースの情報を使用する場合
dns server pp 1
# lan2インタフェースの情報を使用する場合
dns server dhcp lan2
```

### PPPoEの設定

* `pp` インタフェースはipsecの場合はPPPoEの時のみ使用する。
* PPTPを使用するときはこの限りではない。

```
pp select 1
pppoe auto connect on
pppoe auto disconnect off
pppoe use lan2
pp auth accept pap chap
pp auth myname ユーザID パスワード
pp always-on on
ppp lcp mru on 1454
ppp ccp type none
ip pp address xxx.xxx.xxx.xxx/28
ppp ipcp msext on
ip pp mtu 1454
ip pp nat descriptor 1
pp enable 1
ip route default gateway pp 1
```

IPアドレスを動的に取得する環境の場合は

```
ip pp address xxx.xxx.xxx.xxx/28
↓
ip ipcp ipaddress on
```

#### `pp1` から抜け出したいとき。

```
pp1# pp select none
#
```

インターネット接続はすぐに反映されないみたいなので

```
restart
```

#### インターネット接続試験

まずは `pp` インタフェースの接続状況を確認しましょう。

```
# show status pp 1
PP[01]:
Description: 
Current PPPoE session status is Connected.
Access Concentrator: BAS
1 minute 49 seconds  connection.
Received: 14 packets [840 octets]  Load: 0.0%
Transmitted: 13 packets [511 octets]  Load: 0.0%
PPP Cofigure Options
    LCP Local: Magic-Number MRU, Remote: CHAP Magic-Number MRU
    IPCP Local: Primary-DNS(xxx.xxx.xxx.xxx) Secondary-DNS(xxx.xxx.xxx.xxx), Remo
te: IP-Address
    PP IP Address Local: xxx.xxx.xxx.xxx, Remote: xxx.xxx.xxx.xxx
    CCP: None
```

pingを打ってみる。

```
# ping 8.8.8.8 
received from 8.8.8.8: icmp_seq=1 ttl=47 time=56.253ms
received from 8.8.8.8: icmp_seq=2 ttl=47 time=47.024ms
received from 8.8.8.8: icmp_seq=3 ttl=47 time=50.333ms

4 packets transmitted, 3 packets received, 25.0% packet loss
round-trip min/avg/max = 47.024/51.203/56.253 ms
```

```
# ping www.google.co.jp 
received from nrt04s07-in-f24.1e100.net (173.194.126.216): icmp_seq=0 ttl=53 time=22.926ms
received from nrt04s07-in-f24.1e100.net (173.194.126.216): icmp_seq=1 ttl=53 time=24.834ms
received from nrt04s07-in-f24.1e100.net (173.194.126.216): icmp_seq=2 ttl=53 time=22.070ms

3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max = 22.070/23.276/24.834 ms
```

当然ですが、この段階ではNATディスクリプタを記述していないのでRTX1200にぶら下がっている機器からはインターネットにpingを打つことはできません。

### NATディスクリプタの設定

PPPoE接続のときのお話。

```
nat descriptor type 1 masquerade
nat descriptor address innter 1 192.168.0.1-192.168.0.254
nat descriptor address outer 1 ipcp
# 以下はIPsecに関する静的NATディスクリプタ
nat descriptor masquerade static 1 1 192.168.0.1 udp 500
nat descriptor masquerade static 1 2 192.168.0.1 esp
```

以下はうまくいった

```
nat descriptor address outer 1 ipcp
```

以下うまくいかない

```
nat descriptor address outer 1 primary
```

以下を見ると明白だ。

- [24.3 NAT 処理の外側 IP アドレスの設定](http://www.rtpro.yamaha.co.jp/RT/manual/rt-common/nat/nat_descriptor_address_outer.html)

```
nat descriptor address outer 1 primary
```

のコマンドを叩いた瞬間にpingが途切れる。  
だからといってその場で

```
nat descriptor address outer 1 ipcp
```

に戻してもpingがもどらないので困った。  
こういう時はRTX1200を再起動するか、いったんLANケーブルを抜いてまた差すといい。
