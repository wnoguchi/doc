Amazon EC2
============

Tips
------

### インスタンスメタデータを取得する

```
[ec2-user@ip-172-31-25-90 ~]$ curl -w "\n" http://169.254.169.254/latest/meta-data/
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
hostname
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/
[ec2-user@ip-172-31-25-90 ~]$ curl -w "\n" http://169.254.169.254/latest/meta-data/public-hostname
ec2-54-64-60-151.ap-northeast-1.compute.amazonaws.com
[ec2-user@ip-172-31-25-90 ~]$ curl -w "\n" http://169.254.169.254/latest/meta-data/public-ipv4
54.64.60.151
```

1. [インスタンスメタデータとユーザーデータ - Amazon Elastic Compute Cloud](http://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/AESDG-chapter-instancedata.html)
