Fluentd
==========

ログ収集ツール。

Installation
---------------

* CentOS 6.5

```
curl -L http://toolbelt.treasuredata.com/sh/install-redhat.sh | sh
service td-agent start
chkconfig td-agent on
```

* テスト

```
curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
```

* `/var/log/td-agent/td-agent.log`

```
2014-08-14 04:08:07 +0000 debug.test: {"json":"message"}
```

ApacheのログをFluentdに流してMongoDBにストアする
------------------------------------------------

MongoDBのインストールは省略。

* `/etc/td-agent/td-agent.conf` を構成する。

```
<source>
  type tail
  format apache2
  path /var/log/httpd/access_log
  pos_file /var/log/td-agent/httpd.access_log.pos
  tag mongo.httpd.access
</source>
```

td-agentから読み取れるかなって思ったけどだめだった。

```
2014-08-14 15:16:04 +0900 [error]: Permission denied - /var/log/httpd/access_log
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:549:in `initialize'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:549:in `open'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:549:in `on_notify'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:341:in `on_notify'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:427:in `call'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:427:in `on_timer'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/cool.io-1.1.1/lib/cool.io/loop.rb:96:in `run_once'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/cool.io-1.1.1/lib/cool.io/loop.rb:96:in `run'
  2014-08-14 15:16:04 +0900 [error]: /usr/lib64/fluent/ruby/lib/ruby/gems/1.9.1/gems/fluentd-0.10.50/lib/fluent/plugin/in_tail.rb:212:in `run'
```

SELinuxが効いているのがだめかと思ったけど、そもそも td-agent ユーザでのパーミッションが不足しているのが原因だった。

td-agentから読み取れるようにする。

```
chmod o+rx /var/log/httpd/
chmod o+r /var/log/httpd/access_log
```

そしてサービスリスタート。  
ログを見ると

```
2014-08-14 15:16:47 +0900 [info]: shutting down fluentd
2014-08-14 15:16:47 +0900 [info]: process finished code=0
2014-08-14 15:16:48 +0900 [info]: starting fluentd-0.10.50
2014-08-14 15:16:48 +0900 [info]: reading config file path="/etc/td-agent/td-agent.conf"
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-mixin-config-placeholders' version '0.2.4'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-mixin-plaintextformatter' version '0.2.6'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-flume' version '0.1.1'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-mongo' version '0.7.3'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-rewrite-tag-filter' version '1.4.1'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-s3' version '0.4.0'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-scribe' version '0.10.10'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-td' version '0.10.20'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-td-monitoring' version '0.1.2'
2014-08-14 15:16:48 +0900 [info]: gem 'fluent-plugin-webhdfs' version '0.2.2'
2014-08-14 15:16:48 +0900 [info]: gem 'fluentd' version '0.10.50'
2014-08-14 15:16:48 +0900 [info]: using configuration file: <ROOT>
  <match td.*.*>
    type tdlog
    apikey YOUR_API_KEY
    auto_create_table
    buffer_type file
    buffer_path /var/log/td-agent/buffer/td
  </match>
  <match debug.**>
    type stdout
  </match>
  <source>
    type forward
  </source>
  <source>
    type http
    port 8888
  </source>
  <source>
    type debug_agent
    bind 127.0.0.1
    port 24230
  </source>
  <source>
    type tail
    format apache2
    path /var/log/httpd/access_log
    pos_file /var/log/td-agent/httpd.access_log.pos
    tag mongo.httpd.access
  </source>
</ROOT>
2014-08-14 15:16:48 +0900 [info]: adding source type="forward"
2014-08-14 15:16:48 +0900 [info]: adding source type="http"
2014-08-14 15:16:48 +0900 [info]: adding source type="debug_agent"
2014-08-14 15:16:48 +0900 [info]: adding source type="tail"
2014-08-14 15:16:48 +0900 [info]: adding match pattern="td.*.*" type="tdlog"
2014-08-14 15:16:48 +0900 [info]: adding match pattern="debug.**" type="stdout"
2014-08-14 15:16:48 +0900 [info]: listening fluent socket on 0.0.0.0:24224
2014-08-14 15:16:48 +0900 [info]: listening dRuby uri="druby://127.0.0.1:24230" object="Engine"
2014-08-14 15:16:48 +0900 [info]: following tail of /var/log/httpd/access_log
```

ポジションファイルも構成されている。

```
$ cat /var/log/td-agent/httpd.access_log.pos
/var/log/httpd/access_log       00000000000000a9        000402e7
```

ここで初めてこのログをMongoDBに流すように設定する。  
デフォルトでMongoDBがリッスンするポートは 27017 となっている。  
MongoDBに対してフラッシュされる間隔は10秒となっている。  
すぐにフラッシュしたい場合はこれを短くする。

```
<match mongo.*.*>
  # plugin type
  type mongo

  # mongodb db + collection
  database httpd
  collection access

  # mongodb host + port
  host localhost
  port 27017

  # interval
  flush_interval 10s
</match>
```

* 再起動

```
service td-agent restart
```

### abでアクセスログの流しこみをテストする

* MongoDBのコンソールに接続

```
[root@localhost html]# mongo
MongoDB shell version: 2.6.4
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
> use httpd
switched to db httpd
> db["access"].count();
0
> db
httpd
> use httpd
switched to db httpd
> db
httpd
> db["access"].count();
0
```

* ここで ab を流す。

```
ab -n 100 -c 10 http://localhost/
```

10秒ぐらい待つ。

```
> db["access"].count();
106
> db["access"].find().limit(5);
{ "_id" : ObjectId("53ec5c85e138230c0a000001"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000002"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000003"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000004"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
{ "_id" : ObjectId("53ec5c85e138230c0a000005"), "host" : "::1", "user" : null, "method" : "GET", "path" : "/", "code" : 200, "size" : null, "referer" : null, "agent" : "ApacheBench/2.3", "time" : ISODate("2014-08-14T06:51:47Z") }
> db["access"].findOne();
{
        "_id" : ObjectId("53ec5c85e138230c0a000001"),
        "host" : "::1",
        "user" : null,
        "method" : "GET",
        "path" : "/",
        "code" : 200,
        "size" : null,
        "referer" : null,
        "agent" : "ApacheBench/2.3",
        "time" : ISODate("2014-08-14T06:51:47Z")
}
```

MongoDBに格納できてるみたい。

References
------------

1. [Installing Fluentd Using rpm Package | Fluentd](http://docs.fluentd.org/articles/install-by-rpm#step0-before-installation)
1. [fluentdを試してみた - wyukawa’s blog](http://d.hatena.ne.jp/wyukawa/20120207/1328625443)
