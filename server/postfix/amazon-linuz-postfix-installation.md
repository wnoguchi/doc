# Postfix Installation on Amazon Linux

CentOSでPostfixメールサーバーを構築したことはあるけど、
Amazon Linuxで構築したことがないので手順をメモってみる。

## Guide

### Install Postfix

```
sudo -i
yum -y install postfix
service sendmail stop
chkconfig sendmail off
```

* Change default MTA selection from sendmail to postfix.

```
alternatives --config mta

2 プログラムがあり 'mta' を提供します。

  選択       コマンド
-----------------------------------------------
*+ 1           /usr/sbin/sendmail.sendmail
   2           /usr/sbin/sendmail.postfix

Enter を押して現在の選択 [+] を保持するか、選択番号を入力します:2
```

### Setting Postfix

* Create user, group, and home directory.

```
groupadd -g 10000 vmailbox
useradd -g vmailbox -u 10000 vmailbox
mkdir -p /home/vmailbox/
chown vmailbox:vmailbox /home/vmailbox
chmod 770 /home/vmailbox
```

## References

- [Amazon EC2 (Amazon Linux) での Postfix インストールと設定｜道はなくても進むのだ。](http://blog.genies-ag.jp/2011/08/amazon-ec2-amazon-linux-postfix.html)
