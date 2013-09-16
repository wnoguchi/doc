# EC2チュートリアル

## SSH接続する

```
ssh ec2-user@ec2.example.com -i ~/.ssh/secure-key.pem 
The authenticity of host 'ec2.example.com (54.99.176.99)' can't be established.
RSA key fingerprint is a4:d8:87:8c:e6:31:62:f4:5d:6c:71:43:f4:8c:24:f0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'aws2.pg1x.com,54.250.176.47' (RSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2013.03-release-notes/
There are 10 security update(s) out of 21 total update(s) available
Run "sudo yum update" to apply all updates.
```
