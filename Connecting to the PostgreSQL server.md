**如何访问PostgreSQL**

数据库安装完毕，开始连接数据库。

数据库能否正确连接，与以下五要素相关：

    1.Host or host address
    2.Port
    3.Database name
    4.User
    5.Password (或者其他验证方式)


*客户端psql/pgadmin均通过libpq接口连接数据库，所以这两种连接原理和方式均相同。*

如：
```
psql -U -p -h -D 
```
可以看出，上面连接方式明确指定了具体参数。

如果连接过程中部分参数没有指定，连接则会在当前环境变量里寻找对应参数：
    
    1.PGHOST or PGHOSTADDR
    2.PGPORT 
    3.PGDATABASE
    4.PGUSER
    5.PGPASSWORD 
    
```
环境变量设置：

export PGPORT=1921
export PGHOST=$PGDATA
export PGUSER=postgres
export PGDATABASE=postgres

postgres@pgdb-> psql -h localhost //仅仅指定-h便可登陆，其余所需要素，连接自动查找环境变量了。
psql (9.5.7)
Type "help" for help.

postgres=# 

```
 
- 若在连接过程中，除密码外，其余四个参数均指定，则Postgres会在相应位置寻找密码文件（后面会介绍）

从Postgresql 9.2开始，还可以使用URI格式进行连接：

`psql postgresql://myuser:mypasswd@myhost:5432/mydb`

若并未设置任何环境变量，则连接所需5要素必须都得输入。
若环境变量设置成功，当你再URI省略任意几个要素时，这时连接会自动读取本地环境变量，寻找相关参数。

```
五要素均有：
postgres@pgdb-> psql postgresql://postgres:postgres@localhost:1921/postgres
psql (9.5.7)
Type "help" for help.

postgres=# 

若成功配置环境变量，则将个别要素省略，也可正常连接数据库：

将dbname省略掉：
postgres@pgdb-> psql postgresql://postgres:postgres@localhost:1921/
psql (9.5.7)
Type "help" for help.

postgres=# 


将user省略掉：
postgres@pgdb-> psql postgresql://:postgres@localhost:1921/
psql (9.5.7)
Type "help" for help.

postgres=# 


将password省略掉：

postgres@pgdb-> psql postgresql://postgres@localhost:1921/
psql (9.5.7)
Type "help" for help.

postgres=# 

```



