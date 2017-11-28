- 查询当前连接的数据库名与用户名

```
postgres=# SELECT current_user;
 current_user 
--------------
 postgres
(1 row)

postgres=# SELECT current_database();
 current_database 
------------------
 postgres
(1 row)
```


- 查询所连接数据库的版本信息
```
postgres=# SELECT version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 9.5.7 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-4), 64-bit
(1 row)
```

- 查询所连接数据库的IP地址与端口（如果是UDP连接的话，IP与port都显示空）

```
postgres=# SELECT inet_server_addr(), inet_server_port();
 inet_server_addr | inet_server_port 
------------------+------------------
 192.168.100.222  |             1921
(1 row)


```
