# Glanceによるイメージの管理

## 独自イメージの作成

僕はCentOS信者なのでCentOSのイメージを作成します。

qemu-img create  -f qcow2 centos-6.4-x64.img 50G

Formatting 'centos-6.4-x64.img', fmt=qcow2 size=53687091200 encryption=off cluster_size=65536 

kvm -m 1024 -cdrom debian-7.0testing-adm64-netinst.iso \
    -drive file=debian-7.0testing.img,if=virtio,index=0 \
    -k ja -boot d -net nic -net user -nographic -vnc :99


virt-install --connect qemu:///system \
             --name centos-6.4-x64 \
             --ram=2048 \
             --vcpus=2 \
             --hvm \
             --os-type=Linux \
             --os-variant=virtio26 \
             --disk=/var/samba/centos-6.4-x64.img,device=disk,bus=virtio,size=50,sparse=true,format=qcow2,cache=writeback \
             --nographics \
             --keymap ja \
             --cdrom=/var/samba/notrsync/Linux/CentOS-6.4-x86_64-bin-DVD1.iso \
             --extra-args='console=tty0 console=ttyS0,115200n8'

## 参考サイト

↓たぶん超参考になる。

- [yumによるOSイメージ作成 — オープンソースに関するドキュメント 1.1 documentation](http://oss.fulltrust.co.jp/doc/openstack_faq_grizzly/yum/make_ami_centos.html)


- [OpenStack Folsom構築 on Wheezy (8) 独自イメージ作成 | 外道父の匠](http://blog.father.gedow.net/2013/04/08/openstack-folsom-on-wheezy-vol-08/)
- [virt-install(1): provision new virtual machines - Linux man page](http://linux.die.net/man/1/virt-install)

