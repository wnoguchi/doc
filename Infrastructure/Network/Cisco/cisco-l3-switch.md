Cisco Catalyst 3550
=====================

L3スイッチ。  
半年前に中古で買ってからまったく起動してなかった。  
勉強しよう。

- 型番: WS-C3550-48-EMI
- 10/100Base-TX x 48ポート 1000Base GBIC x 2ポート

超久しぶりに起動した時のシリアルコンソールに流れたログ
----------------------------------------------------

```
POST: Ethernet Controller Tests : End, Status Passed
POST: Loopback Tests : BeginBase ethernet MAC Address: 00:0a:41:15:13:00
Xmodem file system is available.
The password-recovery mechanism is enabled.
Initializing Flash...
flashfs[0]: 14 files, 3 directories
flashfs[0]: 0 orphaned files, 0 orphaned directories
flashfs[0]: Total bytes: 15998976
flashfs[0]: Bytes used: 5105152
flashfs[0]: Bytes available: 10893824
flashfs[0]: flashfs fsck took 18 seconds.
...done Initializing Flash.
Boot Sector Filesystem (bs:) installed, fsid: 3
Loading "flash:c3550-i5q3l2-mz.121-8.EA1c/c3550-i5q3l2-mz.121-8.EA1c.bin"...#######################################################################################################################################################################################################################################################################################################################################################################################################

File "flash:c3550-i5q3l2-mz.121-8.EA1c/c3550-i5q3l2-mz.121-8.EA1c.bin" uncompressed and installed, entry point: 0x3000
executing...

              Restricted Rights Legend

Use, duplication, or disclosure by the Government is
subject to restrictions as set forth in subparagraph
(c) of the Commercial Computer Software - Restricted
Rights clause at FAR sec. 52.227-19 and subparagraph
(c) (1) (ii) of the Rights in Technical Data and Computer
Software clause at DFARS sec. 252.227-7013.

           cisco Systems, Inc.
           170 West Tasman Drive
           San Jose, California 95134-1706



Cisco Internetwork Operating System Software
IOS (tm) C3550 Software (C3550-I5Q3L2-M), Version 12.1(8)EA1c, RELEASE SOFTWARE (fc1)
Copyright (c) 1986-2002 by cisco Systems, Inc.
Compiled Fri 15-Feb-02 10:50 by antonino
Image text-base: 0x00003000, data-base: 0x006675E0


Initializing flashfs...
flashfs[1]: 14 files, 3 directories
flashfs[1]: 0 orphaned files, 0 orphaned directories
flashfs[1]: Total bytes: 15998976
flashfs[1]: Bytes used: 5105152
flashfs[1]: Bytes available: 10893824
flashfs[1]: flashfs fsck took 8 seconds.
flashfs[1]: Initialization complete.
...done Initializing flashfs.
POST: CPU Buffer Tests : Begin
POST: CPU Buffer Tests : End, Status Passed
POST: CPU Interface Tests : Begin
POST: CPU Interface Tests : End, Status Passed
POST: Switch Core Tests : Begin
POST: Switch Core Tests : End, Status Passed
POST: CAM Subsystem Tests : Begin
POST: CAM Subsystem Tests : End, Status Passed
POST: Ethernet Controller Tests : Begin
POST: Ethernet Controller Tests : End, Status Passed
POST: Loopback Tests : Begin
POST: Loopback Tests : End, Status Passed

cisco WS-C3550-48 (PowerPC) processor (revision D0) with 65526K/8192K bytes of memory.
Processor board ID CAT0625X0T3
Last reset from warm-reset
Bridging software.
Running Layer2/3 Switching Image

Ethernet-controller 1 has 12 Fast Ethernet/IEEE 802.3 interfaces

Ethernet-controller 2 has 12 Fast Ethernet/IEEE 802.3 interfaces

Ethernet-controller 3 has 12 Fast Ethernet/IEEE 802.3 interfaces

Ethernet-controller 4 has 12 Fast Ethernet/IEEE 802.3 interfaces

Ethernet-controller 5 has 1 Gigabit Ethernet/IEEE 802.3 interface

Ethernet-controller 6 has 1 Gigabit Ethernet/IEEE 802.3 interface

48 FastEthernet/IEEE 802.3 interface(s)
2 Gigabit Ethernet/IEEE 802.3 interface(s)

The password-recovery mechanism is enabled.
32K bytes of flash-simulated non-volatile configuration memory.
Base ethernet MAC Address: 00:0A:41:15:13:00
Motherboard assembly number: 73-5701-06
Power supply part number: 34-0967-01
Motherboard serial number: CAT06240AE8
Power supply serial number: DAB06170JW3
Model revision number: D0
Motherboard revision number: C0
Model number: WS-C3550-48-EMI
System serial number: CAT0625X0T3

         --- System Configuration Dialog ---

Would you like to enter the initial configuration dialog? [yes/no]:
00:00:41: %SPANTREE-5-EXTENDED_SYSID: Extended SysId enabled for type vlan
00:00:46: %SYS-5-RESTART: System restarted --
Cisco Internetwork Operating System Software
IOS (tm) C3550 Software (C3550-I5Q3L2-M), Version 12.1(8)EA1c, RELEASE SOFTWARE (fc1)
Copyright (c) 1986-2002 by cisco Systems, Inc.
Compiled Fri 15-Feb-02 10:50 by antonino
00:00:51: %LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to up
00:00:52: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
00:01:23: %LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan1, changed state to up
00:01:23: AUTOINSTALL: Vlan1 is assigned 192.168.0.3
%Error opening tftp://255.255.255.255/network-confg (Timed out)
%Error opening tftp://255.255.255.255/cisconet.cfg (Timed out)
%Error opening tftp://255.255.255.255/router-confg (Timed out)
%Error opening tftp://255.255.255.255/ciscortr.cfg (Timed out)
% Please answer 'yes' or 'no'.
Would you like to enter the initial configuration dialog? [yes/no]: 
```

初期設定
---------

```
Would you like to enter the initial configuration dialog? [yes/no]: yes

At any point you may enter a question mark '?' for help.
Use ctrl-c to abort configuration dialog at any prompt.
Default settings are in square brackets '[]'.


Basic management setup configures only enough connectivity
for management of the system, extended setup will ask you
to configure each interface on the system

Would you like to enter basic management setup? [yes/no]: yes
Configuring global parameters:
```

### ホスト名の設定

```
  Enter host name [Switch]: foobar_sw
```

### イネーブル シークレット パスワード

暗号化される。

```
  The enable secret is a password used to protect access to
  privileged EXEC and configuration modes. This password, after
  entered, becomes encrypted in the configuration.
  Enter enable secret: *****
```

### イネーブル パスワード

プレーンテキストで保存される。

```
  The enable password is used when you do not specify an
  enable secret password, with some older software versions, and
  some boot images.
  Enter enable password: *****
```

ちなみにイネーブルシークレットパスワードと同じものを指定すると別のを指定しろって怒られる。

```
% Please choose a password that is different from the enable secret
```

### TELNETクライアントのパスワード

```
  The virtual terminal password is used to protect
  access to the router over a network interface.
  Enter virtual terminal password: *****
```

### SNMP

SNMPは今は必要ない。

```
  Configure SNMP Network Management? [no]: no
```

### 各ポートの接続状態の確認

ちなみにVlan1はデフォルトのVLAN。  
ここでは0/1はルーターにつながっていて、0/2はノートPCをつないでいる。

```
Current interface summary

Interface                  IP-Address      OK? Method Status                Protocol
Vlan1                      192.168.0.3     YES other  up                    up      
FastEthernet0/1            unassigned      YES unset  up                    up      
FastEthernet0/2            unassigned      YES unset  up                    up      
FastEthernet0/3            unassigned      YES unset  down                  down    
FastEthernet0/4            unassigned      YES unset  down                  down    
FastEthernet0/5            unassigned      YES unset  down                  down    
FastEthernet0/6            unassigned      YES unset  down                  down    
FastEthernet0/7            unassigned      YES unset  down                  down    
FastEthernet0/8            unassigned      YES unset  down                  down    
FastEthernet0/9            unassigned      YES unset  down                  down    
FastEthernet0/10           unassigned      YES unset  down                  down    
FastEthernet0/11           unassigned      YES unset  down                  down    
FastEthernet0/12           unassigned      YES unset  down                  down    
FastEthernet0/13           unassigned      YES unset  down                  down    
FastEthernet0/14           unassigned      YES unset  down                  down    
FastEthernet0/15           unassigned      YES unset  down                  down    
FastEthernet0/16           unassigned      YES unset  down                  down    
FastEthernet0/17           unassigned      YES unset  down                  down    
FastEthernet0/18           unassigned      YES unset  down                  down    
FastEthernet0/19           unassigned      YES unset  down                  down    
FastEthernet0/20           unassigned      YES unset  down                  down    
FastEthernet0/21           unassigned      YES unset  down                  down    
FastEthernet0/22           unassigned      YES unset  down                  down    
FastEthernet0/23           unassigned      YES unset  down                  down    
FastEthernet0/24           unassigned      YES unset  down                  down    
FastEthernet0/25           unassigned      YES unset  down                  down    
FastEthernet0/26           unassigned      YES unset  down                  down    
FastEthernet0/27           unassigned      YES unset  down                  down    
FastEthernet0/28           unassigned      YES unset  down                  down    
FastEthernet0/29           unassigned      YES unset  down                  down    
FastEthernet0/30           unassigned      YES unset  down                  down    
FastEthernet0/31           unassigned      YES unset  down                  down    
FastEthernet0/32           unassigned      YES unset  down                  down    
FastEthernet0/33           unassigned      YES unset  down                  down    
FastEthernet0/34           unassigned      YES unset  down                  down    
FastEthernet0/35           unassigned      YES unset  down                  down    
FastEthernet0/36           unassigned      YES unset  down                  down    
FastEthernet0/37           unassigned      YES unset  down                  down    
FastEthernet0/38           unassigned      YES unset  down                  down    
FastEthernet0/39           unassigned      YES unset  down                  down    
FastEthernet0/40           unassigned      YES unset  down                  down    
FastEthernet0/41           unassigned      YES unset  down                  down    
FastEthernet0/42           unassigned      YES unset  down                  down    
FastEthernet0/43           unassigned      YES unset  down                  down    
FastEthernet0/44           unassigned      YES unset  down                  down    
FastEthernet0/45           unassigned      YES unset  down                  down    
FastEthernet0/46           unassigned      YES unset  down                  down    
FastEthernet0/47           unassigned      YES unset  down                  down    
FastEthernet0/48           unassigned      YES unset  down                  down    
GigabitEthernet0/1         unassigned      YES unset  down                  down    
GigabitEthernet0/2         unassigned      YES unset  down                  down    
```

### 管理用インタフェース

物理ポートかVLAN名を指定する。
vlan1のメンバのアクセスを

```
Enter interface name used to connect to the
management network from the above interface summary: vlan1
```

#### Vlan1のインタフェースIPを指定

```
Configuring interface Vlan1:
  Configure IP on this interface? [yes]: yes
    IP address for this interface [192.168.0.3]: 192.168.0.200
    Subnet mask for this interface [255.255.255.0] : 255.255.255.0
    Class C network is 192.168.0.0, 24 subnet bits; mask is /24
```

1台しかないのでクラスタコマンドスイッチとして設定しない。

```
Would you like to enable as a cluster command switch? [yes/no]: no
```

### 設定内容の確認

```
The following configuration command script was created:
```

```
hostname miyako_sw
enable secret 5 ***********
enable password ***********
line vty 0 15
password ***********
no snmp-server
!
no ip routing

!
interface Vlan1
no shutdown
ip address 192.168.0.200 255.255.255.0
!
interface FastEthernet0/1
!
interface FastEthernet0/2
!
interface FastEthernet0/3
!
interface FastEthernet0/4
!
interface FastEthernet0/5
!
interface FastEthernet0/6
!
interface FastEthernet0/7
!
interface FastEthernet0/8
!
interface FastEthernet0/9
!
interface FastEthernet0/10
!
interface FastEthernet0/11
!
interface FastEthernet0/12
!
interface FastEthernet0/13
!
interface FastEthernet0/14
!
interface FastEthernet0/15
!
interface FastEthernet0/16
!
interface FastEthernet0/17
!
interface FastEthernet0/18
!
interface FastEthernet0/19
!
interface FastEthernet0/20
!
interface FastEthernet0/21
!
interface FastEthernet0/22
!
interface FastEthernet0/23
!
interface FastEthernet0/24
!
interface FastEthernet0/25
!
interface FastEthernet0/26
!
interface FastEthernet0/27
!
interface FastEthernet0/28
!
interface FastEthernet0/29
!
interface FastEthernet0/30
!
interface FastEthernet0/31
!
interface FastEthernet0/32
!
interface FastEthernet0/33
!
interface FastEthernet0/34
!
interface FastEthernet0/35
!
interface FastEthernet0/36
!
interface FastEthernet0/37
!
interface FastEthernet0/38
!         
interface FastEthernet0/39
!
interface FastEthernet0/40
!
interface FastEthernet0/41
!
interface FastEthernet0/42
!
interface FastEthernet0/43
!
interface FastEthernet0/44
!
interface FastEthernet0/45
!
interface FastEthernet0/46
!
interface FastEthernet0/47
!
interface FastEthernet0/48
!
interface GigabitEthernet0/1
!
interface GigabitEthernet0/2
!
end
```

### さいごに

保存して抜ける。そのままエンター。

```
[0] Go to the IOS command prompt without saving this config.
[1] Return back to the setup without saving this config.
[2] Save this configuration to nvram and exit.

Enter your selection [2]: 
```

```
Building configuration...
[OK]
Use the enabled mode 'configure' command to modify this configuration.




Press RETURN to get started!
```

ふー。

### ちょっと触ってみた

まずはイネーブルシークレットパスワードを入力してログイン。

```
miyako_sw>enable  
Password: 
miyako_sw#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
```

試しに1番ポートを無効化してみる。

```
miyako_sw(config)#interface FastEthernet 0/1
miyako_sw(config-if)#shutdown 
01:31:44: %LINK-5-CHANGED: Interface FastEthernet0/1, changed state to administratively down
01:31:45: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down

miyako_sw(config-if)#no shutdown 
01:32:04: %LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to down
01:32:07: %LINK-3-UPDOWN: Interface FastEthernet0/1, changed state to up
01:32:09: %LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
```

1番ポートのLEDが消えた。ノートPCからのpingも飛ばなくなった。

no shutdownでリンクを回復してみる。

オレンジになったのち、グリーンに点灯し、やがてpingの疎通が回復した。

見た感じ、この時点ではデフォルトゲートウェイが設定されていないっぽいので、設定する。

この時点でUser EXEC/Privileged EXECモードでpingを打ってみてもデフォルトゲートウェイが設定されていないので同一サブネット内の機器にはpingは打てるが、インターネットにはいけない。

```
miyako_sw>ping 192.168.0.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

miyako_sw>ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

まず現在のコンフィグモードから `exit` でグローバルコンフィグモードに戻る。

```
miyako_sw(config-if)#exit
```

デフォルトゲートウェイの設定。

```
miyako_sw(config)#ip default-gateway 192.168.0.1
```

じゃあpingを打ってみる。

```
miyako_sw(config)#exit
miyako_sw#
01:56:50: %SYS-5-CONFIG_I: Configured from console by console   
miyako_sw#ping 8.8.8.8

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 8.8.8.8, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 48/49/52 ms
miyako_sw#
```

OK。

欲を出してDNSまで設定してみる。

```
miyako_sw(config)#ip name-server 8.8.8.8    
miyako_sw(config)#end
miyako_sw#ping www.google.co.jp
02:06:13: %SYS-5-CONFIG_I: Configured from console by console
Translating "www.google.co.jp"...domain server (192.168.0.1) (8.8.8.8) [OK]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 74.125.235.151, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/17/20 ms
```

ログの時刻がずれているのが気になるのでこれも修正する。

```
miyako_sw(config)#ntp peer ntp.nict.jp
Translating "ntp.nict.jp"...domain server (192.168.0.1) (8.8.8.8) [OK]

miyako_sw(config)#end
miyako_sw#
02:13:21: %SYS-5-CONFIG_I: Configured from console by console
miyako_sw#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
miyako_sw(config)#clo
miyako_sw(config)#clock tim
miyako_sw(config)#clock timezone JST 9
miyako_sw(config)#end
miyako_sw#
02:16:23: %SYS-5-CONFIG_I: Configured from console by console
miyako_sw#now
Translating "now"...domain server (192.168.0.1) (8.8.8.8)
% Unknown command or computer name, or unable to find computer address
miyako_sw#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
miyako_sw(config)#end
miyako_sw#
02:18:18: %SYS-5-CONFIG_I: Configured from console by console
```

```
miyako_sw(config)#ntp server ntp.nict.jp
```

```
miyako_sw#show ntp associations 

      address         ref clock     st  when  poll reach  delay  offset    disp
*~133.243.238.163  .NICT.            1    60    64  377     7.4   14.53     8.0
 * master (synced), # master (unsynced), + selected, - candidate, ~ configured
```

うーん、時刻の同期はうまくいかないな。。。  
もう少し放置してみる。

最後に設定を保存する。

```
miyako_sw#copy running-config startup-config
Destination filename [startup-config]?  
Building configuration...
[OK]
```

最後にUser EXECモードに戻る。

```
miyako_sw#disable
miyako_sw>
```

- 再起動

```
reload
```

- 初期化

```
miyako_sw#write erase
Erasing the nvram filesystem will remove all files! Continue? [confirm]
[OK]
Erase of nvram: complete
miyako_sw#reload

System configuration has been modified. Save? [yes/no]: yes
Building configuration...
[OK]
Proceed with reload? [confirm]
```

工場出荷時に初期化されないじゃんって思ったら、

```
System configuration has been modified. Save? [yes/no]: no
```

って答えないといけないらしい。

`show clock`を使うと時間を表示できるらしい。

```
miyako_sw#show clock
23:41:46.690 JST Sat Mar 1 2014
```

同期できてるし、、、おいー。。。  
よくよく調べたらデフォルトで出力されるログのタイムスタンプ形式はuptimeでした・・・。

```
service timestamps log uptime
```

[Cisco機器の操作はじめから - Ciscoルータ（ 出力されたログにあわてない )](http://www.infraexpert.com/study/ciscoios7.html)

References
------------

- [Catalystスイッチとは](http://www.infraexpert.com/study/study18.html)
- [Catalystスイッチをはじめから - スイッチの基本設定](http://www.infraexpert.com/study/catalyst6.html)
- [Catalyst 2960 スイッチ ハードウェア インストレー ション ガイド - CLI ベースのセットアップ プログラムによる スイッチの設定 - Cisco Systems](http://www.cisco.com/cisco/web/support/JP/docs/SW/LANSWT-Access/CAT2960SWT/IG/003/hgcliset.html?bid=0900e4b1825ae623)
- [Cisco 基本コマンド](http://www.unix-power.net/routing/commend.html)
- [Cisco ログ取得時の推奨設定 - Cisco Support Community](https://supportforums.cisco.com/docs/DOC-11795)
- [syslogの保存とtimestamps設定の変更 « bobchan's cisco blog](http://hackerbobchan.net/wordpress/?p=68)
- [Verifying NTP Status with the show ntp associations Command - Cisco](http://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-software-releases-110/15171-ntpassoc.html)
- [copyコマンド（設定の保存）　CCNA実機で学ぶ](http://atnetwork.info/ccna/copy_command.html)
- [CiscoルータでNTPクライアントの設定をする : インフラエンジニアに成る](http://blog.livedoor.jp/ume3_/archives/51458445.html)
- [Cisco機器の操作はじめから - Ciscoルータ（ 出力されたログにあわてない )](http://www.infraexpert.com/study/ciscoios7.html)
