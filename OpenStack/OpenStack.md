# OpenStack

プライベートクラウドを構築するコンポーネント群のOSS。  
OpenStackを導入するにあたって参考にさせていただいたサイトや手順等をメモしていきます。

## 概要

OpenStackは大きく2つの種類に分類される。

- Cloud Controller node
- Compute Nodes

Cloud Controller nodeは制御ノードと呼ばれて、novaとかglanceとかいろいろ入ります。  
制御用のノードなので比較的計算機リソースはディスクが30GBぐらい、  
メモリが12GBぐらい必要になります。  
でも実験用とかだったら仮想マシンのイメージとかブロックストレージ等の永続化データも  
この制御ノードと一緒にされるのでディスクは1〜2TB必要。  
NIC二枚差し推奨らしい。実験用ならNIC一枚でも動きます。  

Compute Nodesは実際に仮想マシンを稼働させるノード。計算機リソースとしてのノードなのでディスクは30GBと少なめ。  
ただメモリは32GB以上推奨。  
仮想化対応必須。KVMとかVMwareとかXenとか。  
うちはKVMだね。

どのみちストレージが必要なのでCinderをどっかで稼働させないといけない。  
Compute Nodesに入れるのかな。  
要調査。

[AWS](http://aws.amazon.com/jp/) と互換性があるのでAWS触っておくと理解が速いと思う。

### 構成されるコンポーネント

- Identity Service(Keystone)
- Image delivery and registration (Glance)
- Volume Service(Cinder)
- Cloud compute (Nova)
- Dashboard (Horizon)

他に必要なのは

- Neutron
- Swift  
Amazon s3的なやつ。

## 近況

- **ネットワーク管理コンポーネントのQuantumはNeutronに名前が変わった。**  
[Fwd: [Openstack] Quantum is changing its name to... - Google グループ](https://groups.google.com/forum/#!topic/openstack-ja/ScQA_eLd2Gw)

## インストール

とりあえずオールインワン構成でインストールしてみよう。

### システム構成

- サーバー: FUJITSU Server PRIMERGY RX100 S7
- OS: CentOS 6.4 x64
- 実装メモリ: 16GB
- ディスク: RAIDなし
-- HDD 1TB Cinder用のディスク
-- SSD 80GB 制御用ノード、Novaノード用のOS格納ディスク、容量はちっちゃくていい

このサーバー好きだわ。
僕が実験で構築したネットワーク構成図を書くとこんな感じ？

![](img/experimental_network.png)

ちなみにこのマークダウンを整理する前の手順書では以下の構成でも動きました。

- サーバー: FUJITSU Server PRIMERGY RX100 S7
- OS: CentOS 6.4 x64
- 実装メモリ: 16GB
- ディスク: HDD 1TB RAIDなし 

### IPアドレス固定化

```
# /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.168.0.10
NETMASK=255.255.255.0
GATEWAY=192.168.0.1
```

### IPv6無効化

コメントアウトもしくは削除。

```
# /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

### SELinux無効化

よろしくやっといてください。

#### 必要最低限のパッケージを入れる

```
yum install -y ntp man openssh-clients &&
service ntpd start &&
chkconfig ntpd on
```

#### EPELリポジトリーを有効化する

まずepel基本パッケージを有効化します。

```
wget ftp://ftp.riken.jp/Linux/fedora/epel/6/x86_64/epel-release-6-8.noarch.rpm &&
rpm -ivh epel-release-6-8.noarch.rpm
```

さらにOpenStack Grizzlyのepelリポジトリも有効化します。

```
cat <<EOF >/etc/yum.repos.d/openstack-grizzly.repo
[epel-openstack-grizzly]
name=OpenStack Grizzly Repository for EPEL 6
baseurl=http://repos.fedorapeople.org/repos/openstack/openstack-grizzly/epel-$releasever/
enabled=1
skip_if_unavailable=1
gpgcheck=0
EOF
yum clean all
yum -y update
```

これやらないとあとで

[Keystoneが起動しない件](Keystoneが起動しない件.md)

みたいなことになるので注意。

## MySQL

```
yum install -y mysql-server memcached
service mysqld start &&
chkconfig mysqld on &&
mysql -uroot -e "set password for root@localhost=password('nova');"
mysql -uroot -pnova -e "set password for root@127.0.0.1=password('nova');"
mysql -uroot -pnova -e "set password for root@stack01=password('nova');"
```

```
service mysqld start
chkconfig mysqld on

MySQLインストール時にそのホスト名で予めレコードが作成されている模様。
ERROR 1133 (42000) at line 1: Can't find any matching row in the user table

mysql -uroot -e "set password for root@localhost=password('nova');"
mysql -uroot -pnova -e "set password for root@127.0.0.1=password('nova');"
mysql -uroot -pnova -e "GRANT ALL PRIVILEGES ON *.* TO root@stack01 IDENTIFIED BY 'nova' WITH GRANT OPTION;"
mysql -uroot -pnova -e "set password for root@stack01=password('nova');"
```

```
service memcached start
chkconfig memcached on
```

```
yum install qpid-cpp-server -y
sed -i 's/auth=yes/auth=no/' /etc/qpidd.conf
service qpidd restart
chkconfig qpidd on
```

```
yum install -y iscsi-initiator-utils qemu-kvm libvirt bridge-utils libvirt-python avahi
service messagebus restart
service avahi-daemon restart
service libvirtd restart
service messagebus start
service avahi-daemon start
```

```
yum install -y openstack-keystone
cp -a /etc/keystone /etc/keystone_bak
```


```
vi /etc/keystone/keystone.conf
```

```
diff --git a/keystone.conf b/keystone.conf
index ff2dafb..a6d2810 100644
--- a/keystone.conf
+++ b/keystone.conf
@@ -1,27 +1,27 @@
 [DEFAULT]
 log_file = /var/log/keystone/keystone.log
 # A "shared secret" between keystone and other openstack services
-# admin_token = ADMIN
+admin_token = ADMIN
 
 # The IP address of the network interface to listen on
-# bind_host = 0.0.0.0
+bind_host = 0.0.0.0
 
 # The port number which the public service listens on
-# public_port = 5000
+public_port = 5000
 
 # The port number which the public admin listens on
-# admin_port = 35357
+admin_port = 35357
 
 # The port number which the OpenStack Compute service listens on
-# compute_port = 8774
+compute_port = 8774
 
 # === Logging Options ===
 # Print debugging output
-# verbose = False
+verbose = False
 
 # Print more verbose output
 # (includes plaintext request logging, potentially including passwords)
-# debug = False
+debug = False
 
 # Name of log file to output to. If not set, logging will go to stdout.
 # log_file = keystone.log
@@ -58,9 +58,10 @@ log_file = /var/log/keystone/keystone.log
 connection = mysql://keystone:keystone@localhost/keystone
 # The SQLAlchemy connection string used to connect to the database
 # connection = sqlite:///keystone.db
+connection = mysql://keystone:password@stack01/keystone?charset=utf8
 
 # the timeout before idle sql connections are reaped
-# idle_timeout = 200
+idle_timeout = 200
 
 [identity]
 driver = keystone.identity.backends.sql.Identity
@@ -85,7 +86,7 @@ driver = keystone.token.backends.sql.Token
 # expiration = 86400
 
 [policy]
-# driver = keystone.policy.backends.rules.Policy
+driver = keystone.policy.backends.rules.Policy
 
 [ec2]
 driver = keystone.contrib.ec2.backends.sql.Ec2
@@ -99,7 +100,7 @@ driver = keystone.contrib.ec2.backends.sql.Ec2
 #cert_required = True
 
 [signing]
-#token_format = UUID
+token_format = UUID
 #certfile = /etc/keystone/ssl/certs/signing_cert.pem
 #keyfile = /etc/keystone/ssl/private/signing_key.pem
 #ca_certs = /etc/keystone/ssl/certs/ca.pem
@@ -131,6 +132,11 @@ driver = keystone.contrib.ec2.backends.sql.Ec2
 # role_id_attribute = cn
 # role_member_attribute = roleOccupant
 
+[auth]
+methods = password,token
+password = keystone.auth.methods.password.Password
+token = keystone.auth.methods.token.Token
+
 [filter:debug]
 paste.filter_factory = keystone.common.wsgi:Debug.factory
 
@@ -161,29 +167,45 @@ paste.filter_factory = keystone.contrib.s3:S3Extension.factory
 [filter:url_normalize]
 paste.filter_factory = keystone.middleware:NormalizingFilter.factory
 
+[filter:sizelimit]
+paste.filter_factory = keystone.middleware:RequestBodySizeLimiter.factory
+
 [filter:stats_monitoring]
 paste.filter_factory = keystone.contrib.stats:StatsMiddleware.factory
 
 [filter:stats_reporting]
 paste.filter_factory = keystone.contrib.stats:StatsExtension.factory
 
+[filter:access_log]
+paste.filter_factory = keystone.contrib.access:AccessLogMiddleware.factory
+
 [app:public_service]
 paste.app_factory = keystone.service:public_app_factory
 
+[app:service_v3]
+paste.app_factory = keystone.service:v3_app_factory
+
 [app:admin_service]
 paste.app_factory = keystone.service:admin_app_factory
 
 [pipeline:public_api]
-pipeline = stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug ec2_extension user_crud_extension public_service
+pipeline = access_log sizelimit stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug ec2_extension user_crud_extension public_service
+#pipeline = stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug ec2_extension user_crud_extension public_service
 
 [pipeline:admin_api]
-pipeline = stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug stats_reporting ec2_extension s3_extension crud_extension admin_service
+pipeline = access_log sizelimit stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug stats_reporting ec2_extension s3_extension crud_extension admin_service
+#pipeline = stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug stats_reporting ec2_extension s3_extension crud_extension admin_service
+
+[pipeline:api_v3]
+pipeline = access_log sizelimit stats_monitoring url_normalize token_auth admin_token_auth xml_body json_body debug stats_reporting ec2_extension s3_extension service_v3
 
 [app:public_version_service]
-paste.app_factory = keystone.service:public_version_app_factory
+pipeline = access_log sizelimit stats_monitoring url_normalize xml_body public_version_service
+#paste.app_factory = keystone.service:public_version_app_factory
 
 [app:admin_version_service]
-paste.app_factory = keystone.service:admin_version_app_factory
+pipeline = access_log sizelimit stats_monitoring url_normalize xml_body admin_version_service
+#paste.app_factory = keystone.service:admin_version_app_factory
 
 [pipeline:public_version_api]
 pipeline = stats_monitoring url_normalize xml_body public_version_service
@@ -194,9 +216,12 @@ pipeline = stats_monitoring url_normalize xml_body admin_version_service
 [composite:main]
 use = egg:Paste#urlmap
 /v2.0 = public_api
+/v3 = api_v3
 / = public_version_api
 
 [composite:admin]
 use = egg:Paste#urlmap
 /v2.0 = admin_api
+/v3 = api_v3
 / = admin_version_api
+
```


```
MYSQL_PASS_KEYSTONE=password
NOVA_CONTOLLER_HOSTNAME=stack01
mysql -uroot -pnova -e "drop database if exists keystone;"
mysql -uroot -pnova -e "create database keystone character set utf8;"
mysql -uroot -pnova -e "grant all privileges on keystone.* to 'keystone'@'%' identified by '$MYSQL_PASS_KEYSTONE';"
mysql -uroot -pnova -e "grant all privileges on keystone.* to 'keystone'@'localhost' identified by '$MYSQL_PASS_KEYSTONE';"
mysql -uroot -pnova -e "grant all privileges on keystone.* to 'keystone'@'$NOVA_CONTOLLER_HOSTNAME' identified by '$MYSQL_PASS_KEYSTONE';"
keystone-manage db_sync
```

```
chown keystone:keystone /var/log/keystone -R
service openstack-keystone restart
chkconfig openstack-keystone on
service openstack-keystone status

keystone (pid  6358) を実行中...
```

データ投入テスト

```
cd /usr/local/src ; cp -a /usr/share/openstack-keystone/sample_data.sh .

export CONTROLLER_HOST=stack01
export SERVICE_ENDPOINT=http://$CONTROLLER_HOST:35357/v2.0
sed -i "s/localhost/$CONTROLLER_HOST/" /usr/local/src/sample_data.sh
export ENABLE_ENDPOINTS=yes
/usr/local/src/sample_data.sh

-------------+-------------------------------------+
|   Property  |                Value                |
+-------------+-------------------------------------+
|   adminurl  |  http://stack01:$(admin_port)s/v2.0 |
|      id     |   7ceb3a26dfe44c57bdd465986f41beee  |
| internalurl | http://stack01:$(public_port)s/v2.0 |
|  publicurl  | http://stack01:$(public_port)s/v2.0 |
|    region   |              RegionOne              |
|  service_id |   b5432d5af04944d8b8ecf76d17f7da7d  |
+-------------+-------------------------------------+
+-------------+----------------------------------------------------+
|   Property  |                       Value                        |
+-------------+----------------------------------------------------+
|   adminurl  | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s |
|      id     |          3321f331d25b48bfbe78db2465ecd178          |
| internalurl | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s |
|  publicurl  | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s |
|    region   |                     RegionOne                      |
|  service_id |          fcd9f77749ad41b6a1c651efade50f18          |
+-------------+----------------------------------------------------+
+-------------+--------------------------------------+
|   Property  |                Value                 |
+-------------+--------------------------------------+
|   adminurl  | http://stack01:8776/v1/$(tenant_id)s |
|      id     |   c9c73f1c82f945e09e30eb13c37141bf   |
| internalurl | http://stack01:8776/v1/$(tenant_id)s |
|  publicurl  | http://stack01:8776/v1/$(tenant_id)s |
|    region   |              RegionOne               |
|  service_id |   c047229f68a9422a82ce05031680b1f3   |
+-------------+--------------------------------------+
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
|   adminurl  |       http://stack01:9292        |
|      id     | a7426193194b4279a90bd0cb6ba4de32 |
| internalurl |       http://stack01:9292        |
|  publicurl  |       http://stack01:9292        |
|    region   |            RegionOne             |
|  service_id | 154112af665f4d978cd32fd4464d68c1 |
+-------------+----------------------------------+
+-------------+------------------------------------+
|   Property  |               Value                |
+-------------+------------------------------------+
|   adminurl  | http://stack01:8773/services/Admin |
|      id     |  73cf69449b194d02928bcffd976c8460  |
| internalurl | http://stack01:8773/services/Cloud |
|  publicurl  | http://stack01:8773/services/Cloud |
|    region   |             RegionOne              |
|  service_id |  f1ddbb5a05c046feaf12e6eeee846bd3  |
+-------------+------------------------------------+
+-------------+-------------------------------------------+
|   Property  |                   Value                   |
+-------------+-------------------------------------------+
|   adminurl  |           http://stack01:8888/v1          |
|      id     |      66cd05cabc654f77bf694749f2a74ab6     |
| internalurl | http://stack01:8888/v1/AUTH_$(tenant_id)s |
|  publicurl  | http://stack01:8888/v1/AUTH_$(tenant_id)s |
|    region   |                 RegionOne                 |
|  service_id |      7d3ee110b7cb4a6581770282c7d7ca12     |
+-------------+-------------------------------------------+

export OS_SERVICE_TOKEN=ADMIN
export OS_SERVICE_ENDPOINT=http://stack01:35357/v2.0/


[root@wnoguchi src]# keystone tenant-list
+----------------------------------+---------+---------+
|                id                |   name  | enabled |
+----------------------------------+---------+---------+
| 439df062c81244a1af4f977f1450990c |   demo  |   True  |
| 8949bace05894bc4bfac9f6b1d93fdf3 | service |   True  |
+----------------------------------+---------+---------+
[root@wnoguchi src]# keystone user-list
+----------------------------------+--------+---------+-------+
|                id                |  name  | enabled | email |
+----------------------------------+--------+---------+-------+
| e9dafc41e0b747dd8947bb573a4a617a | admin  |   True  |       |
| e3930fbde3064bd497280bacf52f86ea |  ec2   |   True  |       |
| 159911ddec4f4c8da26f227f6b5b2886 | glance |   True  |       |
| 19748c3737404b12b22a54ae7c66e31c |  nova  |   True  |       |
| 81d5b89a3e7a42bb80b7081ad88e9aae | swift  |   True  |       |
+----------------------------------+--------+---------+-------+
[root@wnoguchi src]# keystone role-list
+----------------------------------+----------+
|                id                |   name   |
+----------------------------------+----------+
| 9fe2ff9ee4384b1894a90878d3e92bab | _member_ |
| 0f930c30105848aea75c599954a3de60 |  admin   |
+----------------------------------+----------+
[root@wnoguchi src]# keystone service-list
+----------------------------------+----------+--------------+---------------------------+
|                id                |   name   |     type     |        description        |
+----------------------------------+----------+--------------+---------------------------+
| f1ddbb5a05c046feaf12e6eeee846bd3 |   ec2    |     ec2      |  EC2 Compatibility Layer  |
| 154112af665f4d978cd32fd4464d68c1 |  glance  |    image     |    Glance Image Service   |
| b5432d5af04944d8b8ecf76d17f7da7d | keystone |   identity   | Keystone Identity Service |
| fcd9f77749ad41b6a1c651efade50f18 |   nova   |   compute    |    Nova Compute Service   |
| 7d3ee110b7cb4a6581770282c7d7ca12 |  swift   | object-store |       Swift Service       |
| c047229f68a9422a82ce05031680b1f3 |  volume  |    volume    |    Nova Volume Service    |
+----------------------------------+----------+--------------+---------------------------+
[root@wnoguchi src]# keystone endpoint-list
+----------------------------------+-----------+----------------------------------------------------+----------------------------------------------------+----------------------------------------------------+----------------------------------+
|                id                |   region  |                     publicurl                      |                    internalurl                     |                      adminurl                      |            service_id            |
+----------------------------------+-----------+----------------------------------------------------+----------------------------------------------------+----------------------------------------------------+----------------------------------+
| 3321f331d25b48bfbe78db2465ecd178 | RegionOne | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s | http://stack01:$(compute_port)s/v1.1/$(tenant_id)s | fcd9f77749ad41b6a1c651efade50f18 |
| 66cd05cabc654f77bf694749f2a74ab6 | RegionOne |     http://stack01:8888/v1/AUTH_$(tenant_id)s      |     http://stack01:8888/v1/AUTH_$(tenant_id)s      |               http://stack01:8888/v1               | 7d3ee110b7cb4a6581770282c7d7ca12 |
| 73cf69449b194d02928bcffd976c8460 | RegionOne |         http://stack01:8773/services/Cloud         |         http://stack01:8773/services/Cloud         |         http://stack01:8773/services/Admin         | f1ddbb5a05c046feaf12e6eeee846bd3 |
| 7ceb3a26dfe44c57bdd465986f41beee | RegionOne |        http://stack01:$(public_port)s/v2.0         |        http://stack01:$(public_port)s/v2.0         |         http://stack01:$(admin_port)s/v2.0         | b5432d5af04944d8b8ecf76d17f7da7d |
| a7426193194b4279a90bd0cb6ba4de32 | RegionOne |                http://stack01:9292                 |                http://stack01:9292                 |                http://stack01:9292                 | 154112af665f4d978cd32fd4464d68c1 |
| c9c73f1c82f945e09e30eb13c37141bf | RegionOne |        http://stack01:8776/v1/$(tenant_id)s        |        http://stack01:8776/v1/$(tenant_id)s        |        http://stack01:8776/v1/$(tenant_id)s        | c047229f68a9422a82ce05031680b1f3 |
+----------------------------------+-----------+----------------------------------------------------+----------------------------------------------------+----------------------------------------------------+----------------------------------+
```


### Image delivery and registration (Glance)のインストール



まず、SQL接続用設定を変更します。

```
# cd /etc/glance
# vi glance-api.conf glance-registry.conf
sql_connection = mysql://glance:password@stack01/glance?charset=utf8

[keystone_authtoken]の項目を変更します。

admin_tenant_name = service
admin_user = glance
admin_password = glance

[paste_deploy]の項目にkeystoneの設定を追加します。

[paste_deploy]
flavor = keystone

また、ホスト名を変更しておきます。こちらもホスト名は適切なもので設定して下さい。

sed -i "s#127.0.0.1#stack01#" /etc/glance/glance-api.conf
sed -i "s#localhost#stack01#" /etc/glance/glance-api.conf
sed -i "s#127.0.0.1#stack01#" /etc/glance/glance-registry.conf
sed -i "s#localhost#stack01#" /etc/glance/glance-registry.conf
```

このあたりは複雑なのでpatchファイル見てください。

```
MYSQL_PASS_GLANCE=password
NOVA_CONTOLLER_HOSTNAME=stack01
mysql -u root -pnova -e "drop database if exists glance;"
mysql -u root -pnova -e "create database glance character set utf8;"
mysql -u root -pnova -e "grant all privileges on glance.* to 'glance'@'%' identified by '$MYSQL_PASS_GLANCE';"
mysql -u root -pnova -e "grant all privileges on glance.* to 'glance'@'localhost' identified by '$MYSQL_PASS_GLANCE';"
mysql -u root -pnova -e "grant all privileges on glance.* to 'glance'@'$NOVA_CONTOLLER_HOSTNAME' identified by '$MYSQL_PASS_GLANCE';"
sudo glance-manage db_sync


chown glance:glance /var/log/glance -R
for i in api registry
do
  service openstack-glance-$i restart
done
for i in api registry
do
  chkconfig openstack-glance-$i on
done
```

```
export OS_USERNAME=admin
export OS_PASSWORD=secrete
export OS_TENANT_NAME=demo
export OS_AUTH_URL=http://stack01:35357/v2.0/


mkdir -p /opt/virt/fedora17 ; cd /opt/virt/fedora17

curl -O http://berrange.fedorapeople.org/images/2012-11-15/f17-x86_64-openstack-sda.qcow2

NotAuthenticated: 401 Unauthorized
This server could not verify that you are authorized to access the document you requested. Either you supplied the wrong credentials (e.g., bad password), or your browser does not understand how to supply the credentials required.
 Authentication required  
 (HTTP 500)


glance image-create --name="f17-jeos" --is-public=true --disk-format=qcow2 --container-format=ovf < /opt/virt/fedora17/f17-x86_64-openstack-sda.qcow2



+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | 1f104b5667768964d5df8c4ad1d7cd27     |
| container_format | ovf                                  |
| created_at       | 2013-08-05T09:33:23                  |
| deleted          | False                                |
| deleted_at       | None                                 |
| disk_format      | qcow2                                |
| id               | dd8143e2-e54b-4908-b56e-c76f1b5c14ac |
| is_public        | True                                 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | f17-jeos                             |
| owner            | 439df062c81244a1af4f977f1450990c     |
| protected        | False                                |
| size             | 251985920                            |
| status           | active                               |
| updated_at       | 2013-08-05T09:33:24                  |
+------------------+--------------------------------------+

[root@wnoguchi glance]# glance image-list
+--------------------------------------+----------+-------------+------------------+-----------+--------+
| ID                                   | Name     | Disk Format | Container Format | Size      | Status |
+--------------------------------------+----------+-------------+------------------+-----------+--------+
| dd8143e2-e54b-4908-b56e-c76f1b5c14ac | f17-jeos | qcow2       | ovf              | 251985920 | active |
+--------------------------------------+----------+-------------+------------------+-----------+--------+

[root@wnoguchi glance]# glance image-show dd8143e2-e54b-4908-b56e-c76f1b5c14ac
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | 1f104b5667768964d5df8c4ad1d7cd27     |
| container_format | ovf                                  |
| created_at       | 2013-08-05T09:33:23                  |
| deleted          | False                                |
| disk_format      | qcow2                                |
| id               | dd8143e2-e54b-4908-b56e-c76f1b5c14ac |
| is_public        | True                                 |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | f17-jeos                             |
| owner            | 439df062c81244a1af4f977f1450990c     |
| protected        | False                                |
| size             | 251985920                            |
| status           | active                               |
| updated_at       | 2013-08-05T09:33:24                  |
+------------------+--------------------------------------+
```


### Volume Service(Cinder)のインストール

```
yum install openstack-cinder -y

cp -a /etc/cinder /etc/cinder_bak

MYSQL_PASS_CINDER=password
NOVA_CONTOLLER_HOSTNAME=stack01
mysql -uroot -pnova -e "drop database if exists cinder;"
mysql -uroot -pnova -e "create database cinder character set utf8;"
mysql -uroot -pnova -e "grant all privileges on cinder.* to 'cinder'@'%' identified by '$MYSQL_PASS_CINDER';"
mysql -uroot -pnova -e "grant all privileges on cinder.* to 'cinder'@'localhost' identified by '$MYSQL_PASS_CINDER';"
mysql -uroot -pnova -e "grant all privileges on cinder.* to 'cinder'@'$NOVA_CONTOLLER_HOSTNAME' identified by '$MYSQL_PASS_CINDER';"
cinder-manage db sync

2013-08-05 18:43:48     INFO [migrate.versioning.api] 0 -> 1... 
2013-08-05 18:43:52     INFO [migrate.versioning.api] done
2013-08-05 18:43:52     INFO [migrate.versioning.api] 1 -> 2... 
2013-08-05 18:43:54     INFO [migrate.versioning.api] done
2013-08-05 18:43:54     INFO [migrate.versioning.api] 2 -> 3... 
2013-08-05 18:43:54     INFO [migrate.versioning.api] done
2013-08-05 18:43:54     INFO [migrate.versioning.api] 3 -> 4... 
2013-08-05 18:43:55     INFO [004_volume_type_to_uuid] Created foreign key volume_type_extra_specs_ibfk_1
2013-08-05 18:43:55     INFO [migrate.versioning.api] done
2013-08-05 18:43:55     INFO [migrate.versioning.api] 4 -> 5... 
2013-08-05 18:43:55     INFO [migrate.versioning.api] done
2013-08-05 18:43:55     INFO [migrate.versioning.api] 5 -> 6... 
2013-08-05 18:43:56     INFO [migrate.versioning.api] done
2013-08-05 18:43:56     INFO [migrate.versioning.api] 6 -> 7... 
2013-08-05 18:43:56     INFO [migrate.versioning.api] done
2013-08-05 18:43:56     INFO [migrate.versioning.api] 7 -> 8... 
2013-08-05 18:43:56     INFO [migrate.versioning.api] done
2013-08-05 18:43:56     INFO [migrate.versioning.api] 8 -> 9... 
2013-08-05 18:43:56     INFO [migrate.versioning.api] done
```

```
sed -i "s@# include /etc/cinder/volumes/@include /etc/cinder/volumes/@" /etc/tgt/conf.d/cinder.conf
echo "include /etc/tgt/conf.d/*.conf" >> /etc/tgt/targets.conf
echo "include /var/lib/cinder/volumes/*" >> /etc/tgt/conf.d/cinder.conf

service tgtd restart
chkconfig tgtd on
```

```
chown cinder:cinder /var/log/cinder/*
for i in volume api scheduler
do
  service openstack-cinder-$i restart
done
for i in volume api scheduler
do
  chkconfig openstack-cinder-$i on
done
```

```
[root@wnoguchi cinder]# cinder list
[root@wnoguchi cinder]# cinder create --display_name cinder_test 1
+---------------------+--------------------------------------+
|       Property      |                Value                 |
+---------------------+--------------------------------------+
|     attachments     |                  []                  |
|  availability_zone  |                 nova                 |
|       bootable      |                false                 |
|      created_at     |      2013-08-05T09:47:52.128907      |
| display_description |                 None                 |
|     display_name    |             cinder_test              |
|          id         | 173fcd25-c298-41c0-8917-5c5e4eb43f12 |
|       metadata      |                  {}                  |
|         size        |                  1                   |
|     snapshot_id     |                 None                 |
|     source_volid    |                 None                 |
|        status       |               creating               |
|     volume_type     |                 None                 |
+---------------------+--------------------------------------+


[root@wnoguchi cinder]# cinder list
+--------------------------------------+--------+--------------+------+-------------+----------+-------------+
|                  ID                  | Status | Display Name | Size | Volume Type | Bootable | Attached to |
+--------------------------------------+--------+--------------+------+-------------+----------+-------------+
| 173fcd25-c298-41c0-8917-5c5e4eb43f12 | error  | cinder_test  |  1   |     None    |  false   |             |
+--------------------------------------+--------+--------------+------+-------------+----------+-------------+
[root@wnoguchi cinder]# cinder delete $(cinder list | grep cinder_test | awk '{print $2}')
[root@wnoguchi cinder]# cinder list
```


### Cloud compute (Nova)のインストール

```
yum install -y openstack-nova
http://repos.fedorapeople.org/repos/openstack/openstack-grizzly/epel-6/repodata/61fe111b1f291be718a48d77219aef4c1171e05d10115a71da113b8f7e08301d-filelists.sqlite.bz2: [Errno 14] PYCURL ERROR 22 - "The requested URL returned error: 404 Not Found"
Trying other mirror.
Error: failure: repodata/61fe111b1f291be718a48d77219aef4c1171e05d10115a71da113b8f7e08301d-filelists.sqlite.bz2 from epel-openstack-grizzly: [Errno 256] No more mirrors to try.
 You could try using --skip-broken to work around the problem
 You could try running: rpm -Va --nofiles --nodigest
```

どうしてもエラーになる。
URL変わったのかな。
リポジトリメタデータクリアする。

```
yum clean all
```

もう一回。

```
yum install -y openstack-nova
```

今度はうまくいった。

[yumリポジトリのキャッシュをクリアする - think-tの晴耕雨読](http://d.hatena.ne.jp/think-t/20130408/p1)

```
MYSQL_PASS_NOVA=password
NOVA_CONTOLLER_HOSTNAME=stack01
mysql -uroot -pnova -e "drop database if exists nova;"
mysql -uroot -pnova -e "create database nova;"
mysql -uroot -pnova -e "grant all privileges on nova.* to 'nova'@'%' identified by '$MYSQL_PASS_NOVA';"
mysql -uroot -pnova -e "grant all privileges on nova.* to 'nova'@'localhost' identified by '$MYSQL_PASS_NOVA';"
mysql -uroot -pnova -e "grant all privileges on nova.* to 'nova'@'$NOVA_CONTOLLER_HOSTNAME' identified by '$MYSQL_PASS_NOVA';"
nova-manage db sync
```

サービスめっちゃ多い。

```
for proc in api metadata-api cert network compute objectstore console scheduler consoleauth xvpvncproxy conductor
do
  service openstack-nova-$proc restart
done
for proc in api metadata-api cert network compute objectstore console scheduler consoleauth xvpvncproxy conductor
do
  chkconfig openstack-nova-$proc on
done
```

サービスがちゃんと起動しているか確認。

```
[root@wnoguchi nova]# nova-manage service list
Binary           Host                                 Zone             Status     State Updated_At
nova-cert        stack01                              internal         enabled    :-)   2013-08-05 10:08:46
nova-scheduler   stack01                              internal         enabled    :-)   2013-08-05 10:08:47
nova-console     stack01                              internal         enabled    :-)   2013-08-05 10:08:47
nova-conductor   stack01                              internal         enabled    :-)   2013-08-05 10:08:48
nova-consoleauth stack01                              internal         enabled    :-)   2013-08-05 10:08:47
nova-network     stack01                              internal         enabled    :-)   2013-08-05 10:08:47
nova-compute     stack01                              nova             enabled    :-)   None      
```

### Dashboard (Horizon)のインストール

```
yum -y install openstack-dashboard openstack-nova-novncproxy

service openstack-nova-novncproxy start
chkconfig openstack-nova-novncproxy on
```


```
3.2.7.3. iptablesの設定及び再起動
iptablesの設定が必要な場合です。
iptablesにMySQL(3306)、noVNC(6080)、Apache(443,80)、Glance(9292)、Memcached(11211)、Qpid(5672)のポート開放を設定します。
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 11211 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 9292 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 6080 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 5672 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 443 -j ACCEPT' /etc/sysconfig/iptables
# sed -i '10a-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT' /etc/sysconfig/iptables
# service iptables restart
```

Apacheの再起動及び自動起動の設定

```
service httpd restart
service openstack-nova-novncproxy restart
chkconfig httpd on
chkconfig openstack-nova-novncproxy on
```

### OpenStack利用開始

```
useradd stack
cat << KEYSTONERC | tee /home/stack/keystonerc > /dev/null
export OS_NO_CACHE=True
export ADMIN_TOKEN=ADMIN
export OS_USERNAME=admin
export OS_PASSWORD=secrete
export OS_TENANT_NAME=demo
export OS_AUTH_URL=http://stack01:35357/v2.0/
KEYSTONERC
chown stack:stack /home/stack/keystonerc
```

novaマネージネットワークの構築

自宅内LANでは192.168.0.0/24のネットワーク体系を使用しているのでこのネットワークに干渉しない設定ならOKです。

```
nova-manage network create \
--label nova_network1        \
--fixed_range_v4=10.0.0.0/25 \
--bridge_interface=eth0      \
--multi_host=T
2013-08-05 19:17:12.835 8906 INFO nova.network.driver [-] Loading network driver 'nova.network.linux_net'
```

```
[root@wnoguchi nova]# nova-manage network list
id      IPv4                    IPv6            start address   DNS1            DNS2            VlanID          project          uuid           
1       10.0.0.0/25             None            10.0.0.2        8.8.4.4         None            None            None             722c6be4-ba86-45d1-ad8b-9900f4178e58
```

フローティングIPの定義

使っていない社内LAN IPのレンジを切り出して割り当てましょう。

```
nova-manage floating create --ip_range=192.168.0.112/28
```

28ビットマスクなのでネットワークアドレスとブロードキャストアドレスを除いて
14IP使用できる計算になります。

```
[root@wnoguchi nova]# nova-manage floating list
None    192.168.0.113   None    nova    br100
None    192.168.0.114   None    nova    br100
None    192.168.0.115   None    nova    br100
None    192.168.0.116   None    nova    br100
None    192.168.0.117   None    nova    br100
None    192.168.0.118   None    nova    br100
None    192.168.0.119   None    nova    br100
None    192.168.0.120   None    nova    br100
None    192.168.0.121   None    nova    br100
None    192.168.0.122   None    nova    br100
None    192.168.0.123   None    nova    br100
None    192.168.0.124   None    nova    br100
None    192.168.0.125   None    nova    br100
None    192.168.0.126   None    nova    br100
```


キーペアの作成

このあたりで「あーこれAWSのマネジメントコンソールで似たようなこと見たことある」ってなりました。

```
cd /home/stack
[root@wnoguchi stack]# nova keypair-add mykey > mykey
[root@wnoguchi stack]# chown stack:stack mykey
[root@wnoguchi stack]# chmod 600 mykey
[root@wnoguchi stack]# nova keypair-list
+-------+-------------------------------------------------+
| Name  | Fingerprint                                     |
+-------+-------------------------------------------------+
| mykey | 59:56:53:97:22:5c:ad:73:39:93:ea:7c:3e:ec:1a:3f |
+-------+-------------------------------------------------+
```

セキュリティグループの設定

セキュリティグループの設定もec2のインスタンスで既視感。
だからopenstackとec2は互換性があるって言われるのか。

```
[root@wnoguchi stack]# nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| icmp        | -1        | -1      | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
[root@wnoguchi stack]# nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| tcp         | 22        | 22      | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
[root@wnoguchi stack]# nova secgroup-list
+---------+-------------+
| Name    | Description |
+---------+-------------+
| default | default     |
+---------+-------------+
```

icmpにポートとかいう概念はないから-1になってます。

```
[root@wnoguchi stack]# nova secgroup-list-rules default
+-------------+-----------+---------+-----------+--------------+
| IP Protocol | From Port | To Port | IP Range  | Source Group |
+-------------+-----------+---------+-----------+--------------+
| icmp        | -1        | -1      | 0.0.0.0/0 |              |
| tcp         | 22        | 22      | 0.0.0.0/0 |              |
+-------------+-----------+---------+-----------+--------------+
```

ホスト名はOpenStackをインストールしたサーバーのアドレスを指定。
/dashboardを入れないとapacheのデフォルトページだけで焦ります。

http://192.168.0.10/dashboard

デフォルト

- ID: `admin`
- PASS: `secrete`

起動しない。。。

```
nova boot --flavor 1 --image dd8143e2-e54b-4908-b56e-c76f1b5c14ac f17_001 --key_name mykey

[root@wnoguchi stack]# nova image-list
+--------------------------------------+----------+--------+--------+
| ID                                   | Name     | Status | Server |
+--------------------------------------+----------+--------+--------+
| dd8143e2-e54b-4908-b56e-c76f1b5c14ac | f17-jeos | ACTIVE |        |
+--------------------------------------+----------+--------+--------+
[root@wnoguchi stack]# nova flavor-list
+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| ID | Name      | Memory_MB | Disk | Ephemeral | Swap | VCPUs | RXTX_Factor | Is_Public | extra_specs |
+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
| 1  | m1.tiny   | 512       | 0    | 0         |      | 1     | 1.0         | True      | {}          |
| 2  | m1.small  | 2048      | 20   | 0         |      | 1     | 1.0         | True      | {}          |
| 3  | m1.medium | 4096      | 40   | 0         |      | 2     | 1.0         | True      | {}          |
| 4  | m1.large  | 8192      | 80   | 0         |      | 4     | 1.0         | True      | {}          |
| 5  | m1.xlarge | 16384     | 160  | 0         |      | 8     | 1.0         | True      | {}          |
+----+-----------+-----------+------+-----------+------+-------+-------------+-----------+-------------+
[root@wnoguchi stack]# passwd stack
ユーザー stack のパスワードを変更。
新しいパスワード:
新しいパスワードを再入力してください:
passwd: 全ての認証トークンが正しく更新できました。
[root@wnoguchi stack]# nova list
+--------------------------------------+--------------+--------+----------+
| ID                                   | Name         | Status | Networks |
+--------------------------------------+--------------+--------+----------+
| 0e34a951-3204-43d6-9d69-b8d97d482cfe | vmtest       | ERROR  |          |
| 12b86b96-bbcb-4b4f-8b96-4eed67ef6cec | wataru-no-vm | ERROR  |          |
+--------------------------------------+--------------+--------+----------+


nova boot --flavor 1 --image dd8143e2-e54b-4908-b56e-c76f1b5c14ac f17_001 --key_name mykey


+-------------------------------------+--------------------------------------+
| Property                            | Value                                |
+-------------------------------------+--------------------------------------+
| OS-EXT-STS:task_state               | scheduling                           |
| image                               | f17-jeos                             |
| OS-EXT-STS:vm_state                 | building                             |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000003                    |
| flavor                              | m1.tiny                              |
| id                                  | e5cb7f4a-a752-499a-ae74-cb0357b2d95f |
| security_groups                     | [{u'name': u'default'}]              |
| user_id                             | e9dafc41e0b747dd8947bb573a4a617a     |
| OS-DCF:diskConfig                   | MANUAL                               |
| accessIPv4                          |                                      |
| accessIPv6                          |                                      |
| progress                            | 0                                    |
| OS-EXT-STS:power_state              | 0                                    |
| OS-EXT-AZ:availability_zone         | nova                                 |
| config_drive                        |                                      |
| status                              | BUILD                                |
| updated                             | 2013-08-05T10:43:56Z                 |
| hostId                              |                                      |
| OS-EXT-SRV-ATTR:host                | None                                 |
| key_name                            | mykey                                |
| OS-EXT-SRV-ATTR:hypervisor_hostname | None                                 |
| name                                | f17_001                              |
| adminPass                           | mepEtDjG8dL7                         |
| tenant_id                           | 439df062c81244a1af4f977f1450990c     |
| created                             | 2013-08-05T10:43:56Z                 |
| metadata                            | {}                                   |
+-------------------------------------+--------------------------------------+
```

```
for proc in api metadata-api cert network compute objectstore console scheduler consoleauth xvpvncproxy conductor
do
  service openstack-nova-$proc restart
done
```

ごにょごにょ。

```
[root@wnoguchi stack]# nova list
+--------------------------------------+--------------+--------+------------------------+
| ID                                   | Name         | Status | Networks               |
+--------------------------------------+--------------+--------+------------------------+
| e5cb7f4a-a752-499a-ae74-cb0357b2d95f | f17_001      | ERROR  |                        |
| 84e1c17b-6692-4b64-b12d-cf742c4d140b | f17_002      | ERROR  |                        |
| 18c186c4-9386-44ed-80ac-7a258a9e7f25 | f17_003      | BUILD  | nova_network1=10.0.0.2 |
| 0e34a951-3204-43d6-9d69-b8d97d482cfe | vmtest       | ERROR  |                        |
| 12b86b96-bbcb-4b4f-8b96-4eed67ef6cec | wataru-no-vm | ERROR  |                        |
+--------------------------------------+--------------+--------+------------------------+
```

ああ、やっと起動した。

```
vi /etc/nova/nova.conf
```

のIP設定が間違ってたのか。。。
詳しくはnova設定のパッチ見てください。

OpenStack内部ネットワークにてpingの導通を確認。

```
[root@wnoguchi stack]# ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.220 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.175 ms
64 bytes from 10.0.0.2: icmp_seq=3 ttl=64 time=0.206 ms
^C
--- 10.0.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2332ms
rtt min/avg/max/mdev = 0.175/0.200/0.220/0.022 ms
```

このままだと外からSSHできないのでフローティングIPあてました。

```
localhost:OpenStack noguchiwataru$ ping 192.168.0.114
PING 192.168.0.114 (192.168.0.114): 56 data bytes
64 bytes from 192.168.0.114: icmp_seq=0 ttl=63 time=1.839 ms
64 bytes from 192.168.0.114: icmp_seq=1 ttl=63 time=1.018 ms
64 bytes from 192.168.0.114: icmp_seq=2 ttl=63 time=0.849 ms
^C
--- 192.168.0.114 ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.849/1.235/1.839/0.432 ms
```

ちゃんとKVMにインスタンス割り当てられてるかな。

```
[root@wnoguchi stack]# virsh list --all
 Id    名前                         状態
----------------------------------------------------
 1     instance-00000005              実行中
```

うまくいってる。
ためしにSSHしてみる。

```
localhost:OpenStack noguchiwataru$ ssh root@192.168.0.114
The authenticity of host '192.168.0.114 (192.168.0.114)' can't be established.
RSA key fingerprint is ab:8a:97:a4:bf:ab:cb:cd:18:d8:dc:85:0b:2c:39:b4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.0.114' (RSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
localhost:OpenStack noguchiwataru$ vi ~/.ssh/ostackkey.key
localhost:OpenStack noguchiwataru$ chmod 600 ~/.ssh/ostackkey.key
localhost:OpenStack noguchiwataru$ ssh ec2-user@192.168.0.114 -i ~/.ssh/ostackkey.key
[ec2-user@f17-003 ~]$ ls
[ec2-user@f17-003 ~]$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.2  netmask 255.255.255.128  broadcast 10.0.0.127
        inet6 fe80::f816:3eff:fee0:6678  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:e0:66:78  txqueuelen 1000  (Ethernet)
        RX packets 393  bytes 56621 (55.2 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 300  bytes 39705 (38.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 16436
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

外にもいける。
さっき作ったsshキーペアでec2-userにログインできた。
rootは現在のところパス無しでいけるみたい。

このあたりまでくるともうec2とおんなじですね。
awsは超便利版みたいな感じ。

リブートしてもいける。

あとでまとめます。

ログイン画面  
![](img/login.png)

ログイン後  
![](img/dashboard.png)

イメージの一覧  
![](img/images.png)

インスタンス作成画面  
インスタンスのリソーステンプレートも定義できるみたい。  
![](img/create_instance.png)

インスタンスがアクティブになったことを確認する  
![](img/instance_active.png)

フローティングIPを割り当てた  
![](img/float_ip.png)

シリアルコンソールも使える  
![](img/serial.png)

よく見たらボリューム作成できてない。
cinderが死んだー。

```
2013-08-08 00:08:05     INFO [cinder.api.openstack.wsgi] http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes/detail returned with HTTP 200
2013-08-08 00:08:05     INFO [cinder.api.openstack.wsgi] POST http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes
2013-08-08 00:08:05    AUDIT [cinder.api.v1.volumes] Create volume of 50 GB
2013-08-08 00:08:05    AUDIT [cinder.api.v1.volumes] vol={'volume_metadata': [], 'availability_zone': 'nova', 'terminated_at': None, 'updated_at': None, 'snapshot_id': None, 'ec2_id': None, 'mountpoint': None, 'deleted_at': None, 'id': 'a15469cc-b17e-4baa-82df-179c5ef0d7fb', 'size': 50, 'user_id': u'e9dafc41e0b747dd8947bb573a4a617a', 'attach_time': None, 'display_description': u'', 'project_id': u'439df062c81244a1af4f977f1450990c', 'launched_at': None, 'scheduled_at': None, 'status': 'creating', 'volume_type_id': None, 'deleted': False, 'provider_location': None, 'host': None, 'source_volid': None, 'provider_auth': None, 'display_name': u'volvol', 'instance_uuid': None, 'created_at': datetime.datetime(2013, 8, 7, 15, 8, 5, 552787), 'attach_status': 'detached', 'volume_type': None, 'metadata': {}}
2013-08-08 00:08:05     INFO [cinder.api.openstack.wsgi] http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes returned with HTTP 200
2013-08-08 00:08:05    ERROR [cinder.scheduler.manager] Failed to schedule_create_volume: No valid host was found. 
2013-08-08 00:08:05     INFO [cinder.api.openstack.wsgi] GET http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes/detail
2013-08-08 00:08:05    AUDIT [cinder.api.v1.volumes] vol=<cinder.db.sqlalchemy.models.Volume object at 0x37d6b10>
2013-08-08 00:08:05    AUDIT [cinder.api.v1.volumes] vol=<cinder.db.sqlalchemy.models.Volume object at 0x37d6cd0>
2013-08-08 00:08:05     INFO [cinder.api.openstack.wsgi] http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes/detail returned with HTTP 200
```
```
[root@stack01 cinder]# for i in volume api scheduler; do   service openstack-cinder-$i restart; done
openstack-cinder-volume を停止中:                          [失敗]
openstack-cinder-volume を起動中:                          [  OK  ]
openstack-cinder-api を停止中:                             [  OK  ]
openstack-cinder-api を起動中:                             [  OK  ]
openstack-cinder-scheduler を停止中:                       [  OK  ]
openstack-cinder-scheduler を起動中:                       [  OK  ]
```

[OpenStack 2012.2で追加された新機能「Cinder」を使う - さくらのナレッジ](http://knowledge.sakura.ad.jp/tech/119/)

> バックエンドストレージとしてLVMを使用する場合、volume_groupおよびvolume_name_template項目で使用するLVMのボリュームグループ名を指定しておく。デフォルトでは「cinder-volume」となっているが、別のものを指定することも可能だ。

```
volume_group = cinder-volumes
```

```
[root@stack01 cinder]# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "vg_wnoguchi" using metadata type lvm2
```

```
vi /etc/cinder/cinder.conf
volume_group = cinder-volumes
↓
volume_group = vg_wnoguchi
```

もいっちょ

```
[root@stack01 cinder]# for i in volume api scheduler; do   service openstack-cinder-$i restart; done
openstack-cinder-volume を停止中:                          [失敗]
openstack-cinder-volume を起動中:                          [  OK  ]
openstack-cinder-api を停止中:                             [  OK  ]
openstack-cinder-api を起動中:                             [  OK  ]
openstack-cinder-scheduler を停止中:                       [  OK  ]
openstack-cinder-scheduler を起動中:                       [  OK  ]
[root@stack01 cinder]# for i in volume api scheduler; do   service openstack-cinder-$i restart; done
openstack-cinder-volume を停止中:                          [  OK  ]
openstack-cinder-volume を起動中:                          [  OK  ]
openstack-cinder-api を停止中:                             [  OK  ]
openstack-cinder-api を起動中:                             [  OK  ]
openstack-cinder-scheduler を停止中:                       [  OK  ]
openstack-cinder-scheduler を起動中:                       [  OK  ]
```

おー！

```
cinder create --display_name cinder_test 3
+---------------------+--------------------------------------+
|       Property      |                Value                 |
+---------------------+--------------------------------------+
|     attachments     |                  []                  |
|  availability_zone  |                 nova                 |
|       bootable      |                false                 |
|      created_at     |      2013-08-07T15:30:21.480541      |
| display_description |                 None                 |
|     display_name    |             cinder_test              |
|          id         | 402e25e0-eae5-4256-996c-00e7680bd2af |
|       metadata      |                  {}                  |
|         size        |                  3                   |
|     snapshot_id     |                 None                 |
|     source_volid    |                 None                 |
|        status       |               creating               |
|     volume_type     |                 None                 |
+---------------------+--------------------------------------+
```

やっぱだめ、vgの空き領域ないって。そうだよね。

```
2013-08-08 00:33:30     INFO [cinder.api.openstack.wsgi] http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes returned with HTTP 20
0
2013-08-08 00:33:30  WARNING [cinder.scheduler.filters.capacity_filter] Insufficient free space for volume creation (requested / avail): 10/0.
0
2013-08-08 00:33:30    ERROR [cinder.scheduler.manager] Failed to schedule_create_volume: No valid host was found. 
2013-08-08 00:33:30     INFO [cinder.api.openstack.wsgi] GET http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes/detail
2013-08-08 00:33:30    AUDIT [cinder.api.v1.volumes] vol=<cinder.db.sqlalchemy.models.Volume object at 0x46dff10>
2013-08-08 00:33:30     INFO [cinder.api.openstack.wsgi] http://stack01:8776/v1/439df062c81244a1af4f977f1450990c/volumes/detail returned with HTTP 200
2013-08-08 00:33:49     INFO [cinder.volume.manager] Updating volume status


[root@stack01 cinder]# vgscan
  Reading all physical volumes.  This may take a while...
  Found volume group "vg_wnoguchi" using metadata type lvm2
[root@stack01 cinder]# vgdisplay
  --- Volume group ---
  VG Name               vg_wnoguchi
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               931.02 GiB
  PE Size               4.00 MiB
  Total PE              238341
  Alloc PE / Size       238341 / 931.02 GiB
  Free  PE / Size       0 / 0   
  VG UUID               j4qO0Y-WUg9-srNC-J7cD-GcXA-QWEc-DOr2oq
```

しかたないから縮める

```   
[root@stack01 cinder]# lvscan
  ACTIVE            '/dev/vg_wnoguchi/lv_root' [927.14 GiB] inherit
  ACTIVE            '/dev/vg_wnoguchi/lv_swap' [3.88 GiB] inherit
```

むり、時間ない。あとで。

## 参考サイト

### インストール関連

- [3.1. Openstack概要 — オープンソースに関するドキュメント 1.1 documentation](http://oss.fulltrust.co.jp/doc/openstack_grizzly_centos64_yum/openstack_summary.html)
- [3. Openstackインストール手順(Grizzly)CentOS6.4(パッケージ)編 —
オープンソースに関するドキュメント 1.1
documentation](http://oss.fulltrust.co.jp/doc/openstack_grizzly_centos64_yum/)
- [さくらの専用サーバとOpenStackで作るプライベートクラウド | SourceForge.JP Magazine](http://sourceforge.jp/magazine/12/09/18/1126211)  
ちょっと記事が古めで長くて眠くなりそう。  
ただ、データセンター使ってプライベとクラウド作るのでとても参考になりそう。

#### あとで読む的な

- [mseknibilel/OpenStack-Folsom-Install-guide](https://github.com/mseknibilel/OpenStack-Folsom-Install-guide)
- [Chapter 1. OpenStack Basic Install - OpenStack Basic Install](http://docs.openstack.org/folsom/basic-install/content/)
- [OpenStack Installation Guide for Red Hat Enterprise Linux, CentOS, and Fedora - OpenStack Installation Guide for Red Hat Enterprise Linux, CentOS, and Fedora  - Grizzly, 2013.1 with Object Storage 1.8.0](http://docs.openstack.org/grizzly/openstack-compute/install/yum/content/)
- [virtualtech.jp/download/120907OpenStack.pdf](http://virtualtech.jp/download/120907OpenStack.pdf)
ちょい古め。

### 公式サイト的な

- [OpenStack](https://wiki.openstack.org/wiki/Main_Page)
- [日本OpenStackユーザ会 - Japan OpenStack User Group Japan (JOSUG) openstack.jp](http://openstack.jp/)
- [openstack (OpenStack)](https://github.com/openstack)

### マニュアルとか

- [OpenStack 運用ガイド(PDF)](http://dream.daynight.jp/openstack/openstack-ops/openstack-ops-manual-local.pdf)
- [open OpenStack Docs: current](http://docs.openstack.org/trunk/)
- [これからはじめるOpenStackリンク集 | 外道父の匠](http://blog.father.gedow.net/2013/02/19/openstack-links/)
- [オープンソースに関するドキュメント — オープンソースに関するドキュメント 1.1 documentation](http://oss.fulltrust.co.jp/doc/index.html)

### その他

- [OpenStack の利用](http://www.slideshare.net/yosshy/openstack-14884093)
- [OpenStack勉強会](http://www.slideshare.net/obara13/open-stack-16166193)
