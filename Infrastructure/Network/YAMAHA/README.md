YAMAHA series documents
==================================

概要
----------------------------------

YAMAHAシリーズ（主にルーター）の備忘録兼のドキュメントです。  
個人的に学習したことを記述していきます。  
主にRTX1200。VoIPルーターとしてNVR500とかも連携してみたい。  
WLX320も使ってみたいけど金欠だし、バグが多いらしいのでどうなんだろ。様子見？

シリアルコンソールケーブルはクロスの  
[Amazon.co.jp： iBUFFALO USBシリアルケーブル(USBtypeA to D-sub9ピン)1.0m ブラックスケルトン BSUSRC0610BS: パソコン・周辺機器](http://www.amazon.co.jp/iBUFFALO-USB%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%AB%E3%82%B1%E3%83%BC%E3%83%96%E3%83%AB-USBtypeA-%E3%83%96%E3%83%A9%E3%83%83%E3%82%AF%E3%82%B9%E3%82%B1%E3%83%AB%E3%83%88%E3%83%B3-BSUSRC0610BS/dp/B007SI18VW)  
がおすすめ。

よく使う操作のチートシート
----------------------------------

### 文字コードをASCIIにセット

SJISはMacやLinux端末では文字化けしやすいのでASCIIにセットする。

```
console character ascii
```

### 設定をUSBにバックアップ・リストアする

* バックアップ

```
copy config 0 usb1:/rt_config-201312142222.txt
```

* リストア

方向を逆にすればいい。再起動しないと反映されない。

```
copy config usb1:/rt_config-201312142222.txt 0
restart
```

### 出荷時設定に戻す

```
cold start
```

### ファームウェアのアップデート

再起動しないと反映されない。

```
copy exec usb1:/rtx1200.bin 0
restart
```

screenコマンド
----------------------------------

* 接続

```
screen /dev/tty.usbserial-FTGNW2EB
```

* コマンドモード遷移: `Ctrl + a`
* 終了: `:quit`

リンク集
----------------------------------

### 設定例

- [設定例](http://jp.yamaha.com/products/network/solution/?products=rtx1200&solution=&submit=%E7%B5%9E%E3%82%8A%E8%BE%BC%E3%82%80)

### ファームウェア

- [firmware release for Yamaha Network Products](http://www.rtpro.yamaha.co.jp/RT/firmware/)

### ほか

- [外部メモリファイルコピー機能](http://www.rtpro.yamaha.co.jp/RT/docs/external-memory/copy.html)

### 書籍

- [Amazon.co.jp： ヤマハルータでつくるインターネットVPN ［第3版］: 井上孝司: 本](http://www.amazon.co.jp/%E3%83%A4%E3%83%9E%E3%83%8F%E3%83%AB%E3%83%BC%E3%82%BF%E3%81%A7%E3%81%A4%E3%81%8F%E3%82%8B%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8D%E3%83%83%E3%83%88VPN-%EF%BC%BB%E7%AC%AC3%E7%89%88%EF%BC%BD-%E4%BA%95%E4%B8%8A%E5%AD%9D%E5%8F%B8/dp/4839937745/ref=sr_1_1?ie=UTF8&qid=1387725225&sr=8-1&keywords=VPN)
- [Amazon.co.jp： ヤマハルーターで挑戦 企業ネットをじぶんで作ろう: 谷山 亮治, 日経NETWORK: 本](http://www.amazon.co.jp/%E3%83%A4%E3%83%9E%E3%83%8F%E3%83%AB%E3%83%BC%E3%82%BF%E3%83%BC%E3%81%A7%E6%8C%91%E6%88%A6-%E4%BC%81%E6%A5%AD%E3%83%8D%E3%83%83%E3%83%88%E3%82%92%E3%81%98%E3%81%B6%E3%82%93%E3%81%A7%E4%BD%9C%E3%82%8D%E3%81%86-%E8%B0%B7%E5%B1%B1-%E4%BA%AE%E6%B2%BB/dp/4822267687/ref=sr_1_5?ie=UTF8&qid=1387725225&sr=8-5&keywords=VPN)
- [YAMAHAルータ実機で学ぶ　＠network Cisco・アライド実機で学ぶ](http://atnetwork.info/yamaha/)
