MongoDB
=========

ドキュメント指向データベース。

Installation
---------------

* リポジトリ定義を記述する。

`/etc/yum.repos.d/mongodb.repo`

```
[mongodb]
name=MongoDB Repository
baseurl=http://downloads-distro.mongodb.org/repo/redhat/os/x86_64/
gpgcheck=0
enabled=1
```

* インストール

```
yum -y install mongodb-org
```

* サービス起動

SELinuxが無効となっていることを確認する。

```
service mongod start
chkconfig mongod on
```

Getting Started
-----------------

```
> db
apache
> show dbs
admin   (empty)
apache  0.078GB
httpd   0.078GB
local   0.078GB
> use httpd
switched to db httpd
> db["access"].count();
0
> db["access"].findOne();
{
        "_id" : ObjectId("53ec573de138230a29000001"),
        "host" : "::1",
        "user" : null,
        "method" : "GET",
        "path" : "/",
        "code" : 200,
        "size" : null,
        "referer" : null,
        "agent" : "ApacheBench/2.3",
        "time" : ISODate("2014-08-14T06:29:08Z")
}
> db["access"].count();
848
> db.access.find().limit(3);
{ "_id" : ObjectId("53ec5c85e138230c0a000001"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000002"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000003"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
```

References
------------

1. [Install MongoDB on Red Hat Enterprise, CentOS, Fedora, or Amazon Linux — MongoDB Manual 2.6.4](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-red-hat-centos-or-fedora-linux/)
1. [Getting Started with MongoDB — MongoDB Manual 2.6.4](http://docs.mongodb.org/manual/tutorial/getting-started/)
