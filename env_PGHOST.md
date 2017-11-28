### 由于环境变量 PGHOST配置不当引起的 

`postgres@pgdb-> psql
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.1921"?`


并未配置环境变量PGHOST，则psql不加-h命令的时候，默认使用的是数据库参数unix_socket_directories 的默认值(一般是/tmp)
作为unix-domain-socket路径。

即：`psql 相当于 psql -h /tmp`


#### 情况1

若未配置环境变量PGHOST,且修改了unix_socket_directories参数，则直接使用psql命令连接会报错：

```
注：postgres server 是运行状态

postgres@pgdb-> psql
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.1921"?
```


#### 解决方法： 

        1.psql命令必须加 -h 选项 并指定 unix-domain-socket的路径（即unix_socket_directories参数所设置的值）

        2.或者环境变量里添加PGHOST，其值为unix_socket_directories 路径。

#### 过程演示

```
1.无意修改了unix_socket_directories的默认值，并将其改成了任意目录（这里假设将该参数值设置成/home/postgres/pgdata）

2.则下次使用直接使用psql连接时，便会报错：

postgres@pgdb-> psql
psql: could not connect to server: No such file or directory
        Is the server running locally and accepting
        connections on Unix domain socket "/tmp/.s.PGSQL.1921"?

3.这时，解决方法如下：

postgres@pgdb-> psql -h /home/postgres/pgdata
psql (9.5.7)
Type "help" for help.

postgres=# 


或者在环境变量里添加：

export PGHOST=/home/postgres/pgdata

然后应用环境变量，再使用psql直连：
postgres@pgdb-> psql
psql (9.5.7)
Type "help" for help.

postgres=# 

```
