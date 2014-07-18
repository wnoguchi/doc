セキュリティグループを定義する with aws-cli
===========================================

このへんは手でやったほうが早い気がしないでもない。  
CloudFormationとかだといろいろよろしくやってくれるんだろうか。  
まだその辺の境地にも達してないけど・・・。

セキュリティグループの作成
---------------------------

```zsh
% aws ec2 create-security-group --group-name web --description "Web server role security group."
{
    "return": "true", 
    "GroupId": "sg-ffffffff"
}
```

Inbound方向の規則を定義する
---------------------------

### ping(ICMP)応答

```zsh
aws ec2 authorize-security-group-ingress --group-name web --protocol icmp --port -1 --cidr "0.0.0.0/0" | jq '.'
{
  "return": "true"
}
```

### SSH

```zsh
% aws ec2 authorize-security-group-ingress --group-name web --protocol tcp --port 22 --cidr "0.0.0.0/0" | jq '.'
{
  "return": "true"
}
```

### HTTP

```zsh
% aws ec2 authorize-security-group-ingress --group-name web --protocol tcp --port 80 --cidr "0.0.0.0/0" | jq '.'
{
  "return": "true"
}
```

Outbound方向
--------------

Outbound(authorize-security-group-egress)方向はデフォルトで全通過。

まとめると

```zsh
aws ec2 create-security-group --group-name web --description "Web server role security group." | jq '.'
aws ec2 authorize-security-group-ingress --group-name web --protocol icmp --port -1 --cidr "0.0.0.0/0" | jq '.'
aws ec2 authorize-security-group-ingress --group-name web --protocol tcp --port 22 --cidr "0.0.0.0/0" | jq '.'
aws ec2 authorize-security-group-ingress --group-name web --protocol tcp --port 80 --cidr "0.0.0.0/0" | jq '.'
```

逆の操作
----------

セキュリティグループに対して以下の様な操作を行うと22番ポートが封じられるのでSSH不能になる。  
CIDRまできちんと記述しないといけないので注意。

```
aws ec2 revoke-security-group-ingress --group-name web --protocol tcp --port 22 --cidr "0.0.0.0/0" | jq '.'
```

ただし、ICMP規則を削除するときは

```
aws ec2 revoke-security-group-ingress --group-name web --protocol icmp --port -1 --cidr "0.0.0.0/0" | jq '.'
```

に関してはいったんpingを止めないとpingをずっと受け入れ続けてしまうみたいなので、いったん `Ctrl+C` でとめてからもう一回pingを打ってみると通らなくなることが確認できる。

References
-------------

1. [authorize-security-group-ingress — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html)
1. [authorize-security-group-egress — AWS CLI 1.3.21 documentation](http://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-egress.html)
