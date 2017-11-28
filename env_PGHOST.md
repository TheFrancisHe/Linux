### 关于环境变量 PGHOST-

如果环境变量里并未配置PGHOST，则psql不加-h命令的时候，unix-domain-socket使用的是数据库参数unix_socket_directories 的默认值(一般是/tmp)
作为unix-domain-socket路径。


#### 未配置环境变量PGHOST,且修改了unix_socket_directories参数，则直接使用psql命令连接会报错：

```
postgres@pgdb-> psql
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.1921"?
```


#### 解决方法： 

        1.必须加 -h 指定 unix-domain-socket的路径（即unix_socket_directories参数所设置的值）

        2.或者环境变量里添加PGHOST，其值为unix_socket_directories 路径。

```
postgres@pgdb-> psql -h /home/postgres/pgdata
psql (9.5.7)
Type "help" for help.

postgres=# 
```
