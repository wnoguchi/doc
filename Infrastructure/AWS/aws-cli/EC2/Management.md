EC2インスタンスの一般的な操作 with aws-cli
==========================================

インスタンスの操作
-------------------

### インスタンスを作成

- AMI IDを調べてメモる(Amazon Linux AMI 2014.03.2 (HVM) - `ami-29dc9228`)
- 立ち上げたいインスタンス数
- インスタンスタイプ: 現状t2.microが最小（2014/7/8現在）
- 使用するキーペア
- 適用するセキュリティグループ

```
% aws ec2 run-instances --image-id ami-29dc9228 --count 1 --instance-type t2.micro --key-name default --security-groups hoge | jq '.'
```

#### AZを指定する

`--placement AvailabilityZone=ap-northeast-1a` をつけてやればいい。

```
aws ec2 run-instances --image-id ami-29dc9228 --count 1 --instance-type t2.micro --key-name default --security-groups web --placement AvailabilityZone=ap-northeast-1a | jq '.'
```

#### DeleteOnTerminationを避ける

今ではほとんどEBSから起動されるEC2インスタンスがほとんどだが、このままでは DeleteOnTermination=true となってしまっているので、インスタンスをterminateするとインスタンスに紐付けられているボリューム自体も削除されてしまう。

これを避けたいなら

```
% aws ec2 run-instances --image-id ami-29dc9228 --count 1 --instance-type t2.micro --key-name default --security-groups hoge \
                        --block-device-mappings "[{\"DeviceName\": \"/dev/xvda\",\"Ebs\":{\"DeleteOnTermination\": \"false\"}}]" | jq '.'
```

としなければならない。面倒だが。

既に作成されたインスタンスについて DeleteOnTermination=true としたい場合は

```
% aws ec2 modify-instance-attribute --instance-id i-dfff36d1 --block-device-mappings "[{\"DeviceName\": \"/dev/xvda\",\"Ebs\":{\"DeleteOnTermination\": \"false\"}}]" | jq '.'
```

とすればいい。

### インスタンスを起動

```
% aws ec2 start-instances --instance-ids i-xxxxxxxx | jq '.'
```

### インスタンスを停止

```
% aws ec2 stop-instances --instance-ids i-xxxxxxxx | jq '.'          
```

停まったかどうか確認。

```
% aws ec2 describe-instances --instance-ids i-xxxxxxxx | jq -r '.Reservations [] .Instances [] .State .Name'
stopped
```

### インスタンスを削除(terminate)

```
% aws ec2 terminate-instances --instance-ids i-xxxxxxxx | jq '.'
```

### インスタンスをタグの値で絞り込む

タグ Name の値について `test` という値に部分一致するインスタンスの情報を取得する。  
ワイルドカードが使えるっぽい。  
jq で後から絞り込みをかけるより、こちらのほうがAWS流でスマートかもしれない。

```
aws ec2 describe-instances --filters Name=tag-key,Values=Name Name=tag-value,Values="*test*"
```

jqで欲しい値をとってくる
-------------------------

### インスタンスID

```zsh
% aws ec2 describe-instances | jq -r '.Reservations [] .Instances [] .InstanceId'
i-xxxxxxxx
```

### グローバルIPアドレス

```
% aws ec2 describe-instances | jq -r '.Reservations [] .Instances [] .PublicIpAddress'
xxx.xxx.xxx.xxx
```

### PublicDNS名

```
% aws ec2 describe-instances | jq -r '.Reservations [] .Instances [] .PublicDnsName'
ec2-xxx-xxx-xxx-xxx.ap-northeast-1.compute.amazonaws.com
```

### 指定した値に Name が部分一致（正確にはマッチ）するインスタンス情報を取得する

`test` というName属性値にマッチするインスタンス情報を列挙する例。

```
% aws ec2 describe-instances | jq '.Reservations [] .Instances [] | select(has("Tags") and (.Tags [] | .Key == "Name" and (.Value | index("test") != null)))'
```

### すべてのインスタンス情報を主要な属性のみ絞り込んで取得する

```
% aws ec2 describe-instances | jq '.Reservations [] .Instances [] | { Name: (if has("Tags") then (.Tags [] | select(.Key == "Name") | .Value) else null end), InstanceId: .InstanceId, PublicDnsName: .PublicDnsName, PublicIpAddress: .PublicIpAddress, State: .State .Name, InstanceType: .InstanceType, AvailabilityZone: .Placement .AvailabilityZone }'
```

### 指定した値に Name が部分一致（正確にはマッチ）するインスタンス情報を取得する（主要な属性のみ）

`test` というName属性値にマッチするインスタンス情報を列挙する例。  
上の属性絞り込みの合わせ技。黒魔術・・・。

```
% aws ec2 describe-instances | jq '.Reservations [] .Instances [] | select(has("Tags") and (.Tags [] | .Key == "Name" and (.Value | index("test") != null))) | { Name: (.Tags [] | select(.Key == "Name") | .Value), InstanceId: .InstanceId, PublicDnsName: .PublicDnsName, PublicIpAddress: .PublicIpAddress, State: .State .Name, InstanceType: .InstanceType, AvailabilityZone: .Placement .AvailabilityZone }'
```

SSHで接続するとき
------------------

単なる覚え書き。

```
ssh ec2-user@xxx.xxx.xxx.xxx -i ~/.ssh/foobar.pem
```

References
------------

1. [run-instances — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/run-instances.html)
1. [start-instances — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/start-instances.html)
1. [stop-instances — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/stop-instances.html)
1. [terminate-instances — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/terminate-instances.html)
1. [describe-instances — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html)
1. [ec2-modify-instance-attribute - Amazon Elastic Compute Cloud](http://docs.aws.amazon.com/AWSEC2/latest/CommandLineReference/ApiReference-cmd-ModifyInstanceAttribute.html)
1. [Alestic.com: Search Results](http://alestic.com/mt/mt-search.cgi?IncludeBlogs=1&tag=delete-on-termination&limit=20)
