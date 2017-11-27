netstat命令用法参考链接：

https://linux.cn/article-2434-1.html

https://segmentfault.com/a/1190000008633528

对于数据库连接而言，通过netstat可以查看当前连接协议，是udp还是tcp（包括ipv4和ipv6）。

如果本地直接使用psql命令连接，则为udp：

```
窗口1：
postgres@pgdb-> psql
psql (9.5.7)
Type "help" for help.

postgres=# 

重新开一个窗口，看看psql连接协议

窗口2：

[root@pgdb ~]# netstat -anpo|grep psql
unix  3      [ ]         STREAM     CONNECTED     40742  5697/psql   

结论:可以看出来，如果命令仅仅是psql，（只要不涉及 -h -p）那么连接均为udp协议，ipc通信较快的一种。
```

既然是udp协议，通过udp协议的连接，则被pg_hba.conf 中下面的item所管理：

```
# "local" is for Unix domain socket connections only
local   all             all                                     trust

尝试将以上item改为：
# "local" is for Unix domain socket connections only
local   postgres         postgres                               md5  

然后重载下pg_hba.conf文件
postgres@pgdb-> pg_ctl reload;
server signaled

这时候再连接，就会发现只允许postgres用户连接postgres库，并且需要密码才能连接：

postgres@pgdb-> psql
Password: 

当连接其余库时，则会报如下错误：

postgres@pgdb-> psql -d benchmarksql
psql: FATAL:  no pg_hba.conf entry for host "[local]", user "postgres", database "benchmarksql"//不存在相关entry

```



