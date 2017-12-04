`如果客户端认证方式为密码验证，那么必然会涉及到修改密码`

#### 如何安全地修改密码：

- 方式1 
使用psql，连接到Postgres Server：

```
test1=> \password 
Enter new password: 
Enter it again: 
test1=> 

我将原密码hello，修改为hellojava.
```
这种修改方式相当于向postgres server 发送了如下命令：

```
ALTER USER postgres PASSWORD ' md53175af154as54df5as4d5f45sd6af';

后面的字符串是  hellojava经过md5加密后的字符串


```

**注意：尽量不要使用postgres作为用户密码，防止被攻击。**

- 方式2：可以直接发送sql修改：

`这种方式不仅仅限于psql了，其余客户端也能修改，如pgadmin等
```
ALTER USER test1 PASSWORD 'secret'

弊端：通过sql修改，有可能会将修改语句记录在相关工具的log里。
例如：通过psql 运行该条sql，则在.psql_history文件中会有相应语句的记录
      有密码泄露的风险
```
