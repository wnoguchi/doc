# NeutronでVLANとか

VyattaとかAsteriskとか入れたいねん。

## 概要

最近コードネームが Quantum から Neutron に変わりました。  
ご注意を。

## やってみる

```
yum -y install openstack-quantum
```

また、ネットワークノードおよび計算ノードにはopenstack-quantum-linuxbridgeパッケージをインストールしておく。

```
yum -y install openstack-quantum-linuxbridge
```

なお、今回使用したopenstack-quantumパッケージのバージョンは下記のとおりである。

```
rpm -q openstack-quantum
openstack-quantum-2013.1.2-2.el6.noarch
```

## Neutronのサービス登録

OpenStack管理ユーザー `stack` としてログインして `~/keystonerc` が既に読み込まれているものと仮定する。

### 変数設定

リージョン `RegionOne` を指定する。

```
NEUTRON_HOST=stack01
REGION=RegionOne
```

Neutron サービス作成。

```
keystone service-create --name=neutron --type=network --description="Neutron Network Service"
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
| description |     Neutron Network Service      |
|      id     | c86c3207100a47cba2f541f255bb5e56 |
|     name    |             neutron              |
|     type    |             network              |
+-------------+----------------------------------+
```

### リージョン `RegionOne` で登録。

#### サービスID取得

```
SERVICE_ID=`keystone service-list | grep neutron | awk '{print $2}'`
echo $SERVICE_ID
c86c3207100a47cba2f541f255bb5e56
```

#### エンドポイントの登録

```
keystone endpoint-create --region $REGION --service_id=$SERVICE_ID --publicurl "http://$NEUTRON_HOST:9696/" --adminurl "http://$NEUTRON_HOST:9696/" --internalurl "http://$NEUTRON_HOST:9696/"
+-------------+----------------------------------+
|   Property  |              Value               |
+-------------+----------------------------------+
|   adminurl  |       http://stack01:9696/       |
|      id     | c78d75efab8a46699d4f0441ad1ceb4b |
| internalurl |       http://stack01:9696/       |
|  publicurl  |       http://stack01:9696/       |
|    region   |            RegionOne             |
|  service_id | c86c3207100a47cba2f541f255bb5e56 |
+-------------+----------------------------------+
```

#### サービスの一覧を確認する

```
[stack@stack01 ~]$ keystone service-list
+----------------------------------+----------+--------------+---------------------------+
|                id                |   name   |     type     |        description        |
+----------------------------------+----------+--------------+---------------------------+
| 60f2d756fab54276afe55b9c24c20552 |   ec2    |     ec2      |  EC2 Compatibility Layer  |
| 2ac0ba678edd4b7db33eeece3028ec09 |  glance  |    image     |    Glance Image Service   |
| 0de4603b303e4295b75a25d860d6f694 | keystone |   identity   | Keystone Identity Service |
| c86c3207100a47cba2f541f255bb5e56 | neutron  |   network    |  Neutron Network Service  |
| 4b9aad7a03584f468bc742ef2f9dab69 |   nova   |   compute    |    Nova Compute Service   |
| 2ba468927ff344acb808ffac6d2307b5 |  swift   | object-store |       Swift Service       |
| 6ac3bd80e21c4f47a2f2086cda720844 |  volume  |    volume    |    Nova Volume Service    |
+----------------------------------+----------+--------------+---------------------------+
```

## 参考サイト

- [OpenStackの仮想ネットワーク管理機能「Quantum」の基本的な設定 - さくらのナレッジ](http://knowledge.sakura.ad.jp/tech/195/)
- [OpenStack の Quantum + Open vSwitch でVLAN構成を試してみた - Qiita [キータ]](http://qiita.com/m_doi/items/f561894a1f83de40e7dd)
- [OpenStack の Quantum + Linux Bridge でVLAN構成を試してみた - Qiita [キータ]](http://qiita.com/m_doi/items/f3439b78a02b20f1f4a0)
