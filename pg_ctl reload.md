不同操作系统平台，pg_reload的用法：

```
UBUNTU/DEBIAN pg_ctlcluster 9.0 main reload
RED HAT/FEDORA service postgresql reload
pg_ctl -D /var/lib/pgsql/data reload
SOLARIS pg_ctl -D /var/lib/pgsql/data reload
MAC OS pg_ctl -D /var/lib/pgsql/data reload
FREEBSD pg_ctl -D /var/lib/pgsql/data reload
```

我们知道，一些参数(其实是看pg_settings 的context字段)，是可以不用重启postgres cluster 就可以生效的，

如，使用如下命令：

```
1.操作系统命令行： pg_ctl reload (配置环境变量后)

或者

2.
postgres=# select pg_reload_conf();
 pg_reload_conf 
----------------
 t
(1 row)
```

究其根本，其实是先给 postmaster 发送了 signal 信号，然后该信号被传递给所有的 bakend server process。

当然，也可以通过：

```
3.
kill -SIGHUP pid
```
甚至给专门的一个backend process 发送信号，不过慎用，不建议在生产库上做这个操作。


---------
举例：

1.修改hba文件
```
原文件：
# "local" is for Unix domain socket connections only
local   all             all                                     trust
...
这时候有两个session，session1连接的postgres库，session2连接的benchmarksql库。

开始修改pg_hba.conf文件

修改后只能连接postgres库：
# "local" is for Unix domain socket connections only
local   postgres             all                                     trust
...

进行pg_reload


这时候session1，进行 \c benmarksql 操作会报错：

psql: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "benchmarksql"


session2 已经连接到benchmarksql库，尽管已经通过pg_reload 重载了数据库所有配置文件，
但是并不影响session2在benchmarksql库的所有操作，因为提前连上了嘛-

```

更多关于pg_reload：

https://yq.aliyun.com/articles/60259


