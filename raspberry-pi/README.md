Raspberry Pi
==============

- Model B
- 給電はmicro USB タイプAからおこなう（PoEハブがあるならそれでもよさげ）
- ディスプレイ出力は HDMI-DVI
- セルフパワータイプのUSBハブを購入することを強く推奨

イメージのダウンロード
----------------------

- [Downloads | Raspberry Pi](http://www.raspberrypi.org/downloads)

```
[noguchiwataru@Macintosh] ~/Documents/repositories/github/doc/raspberry-pi
% diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            499.4 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.8 GB    disk1
   1:             Windows_FAT_32 NO NAME                 15.8 GB    disk1s1
[noguchiwataru@Macintosh] ~/Documents/repositories/github/doc/raspberry-pi
% diskutil unmountdisk /dev/disk1
Unmount of all volumes on disk1 was successful
[noguchiwataru@Macintosh] ~/Documents/repositories/github/doc/raspberry-pi
% cd ~/Downloads
[noguchiwataru@Macintosh] ~/Downloads
% unzip 2014-01-07-wheezy-raspbian.zip
Archive:  2014-01-07-wheezy-raspbian.zip
  inflating: 2014-01-07-wheezy-raspbian.img  
[noguchiwataru@Macintosh] ~/Downloads
% sudo dd if=~/Downloads/2014-01-07-wheezy-raspbian.img of=/dev/disk1 bs=4m
Password:
706+1 records in
706+1 records out
2962227200 bytes transferred in 1478.284275 secs (2003828 bytes/sec)
```

References
------------

- [Raspberry Pi初期設定 « 備忘録](http://unix-like.dyndns-web.com/?p=407)
