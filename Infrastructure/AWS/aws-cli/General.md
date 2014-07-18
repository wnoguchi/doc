aws-cliで一般的なこと
======================

リージョンの確認
-----------------

```
% aws ec2 describe-regions| jq '.'
{
  "Regions": [
    {
      "RegionName": "eu-west-1",
      "Endpoint": "ec2.eu-west-1.amazonaws.com"
    },
    {
      "RegionName": "sa-east-1",
      "Endpoint": "ec2.sa-east-1.amazonaws.com"
    },
    {
      "RegionName": "us-east-1",
      "Endpoint": "ec2.us-east-1.amazonaws.com"
    },
    {
      "RegionName": "ap-northeast-1",
      "Endpoint": "ec2.ap-northeast-1.amazonaws.com"
    },
    {
      "RegionName": "us-west-2",
      "Endpoint": "ec2.us-west-2.amazonaws.com"
    },
    {
      "RegionName": "us-west-1",
      "Endpoint": "ec2.us-west-1.amazonaws.com"
    },
    {
      "RegionName": "ap-southeast-1",
      "Endpoint": "ec2.ap-southeast-1.amazonaws.com"
    },
    {
      "RegionName": "ap-southeast-2",
      "Endpoint": "ec2.ap-southeast-2.amazonaws.com"
    }
  ]
}
```

AZの確認
----------

```
% aws ec2 describe-availability-zones --region ap-northeast-1 | jq '.'
{
  "AvailabilityZones": [
    {
      "ZoneName": "ap-northeast-1a",
      "Messages": [],
      "RegionName": "ap-northeast-1",
      "State": "available"
    },
    {
      "ZoneName": "ap-northeast-1c",
      "Messages": [],
      "RegionName": "ap-northeast-1",
      "State": "available"
    }
  ]
}
```
