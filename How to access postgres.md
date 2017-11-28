写在前面：

- postgres为了安全性，初始化后，不对非本地用户开启权限。

- 默认地，postgres只允许client通过 unix-domain socket（udp）连接。


### 一、So，如何对远程用户/客户端 开启访问权限？

1.修改参数listen_addresses='*';

2.将以下添加到pg_hba.conf第一行：

```
 Host all all 0.0.0.0/0 md5
 
```

3.重启数据库服务器

*经过以上设置后，当前postgres允许所有用户连接所有数据库，对于远程用户，需要输入密码用以验证。*

### 二、那么，各个环境究竟起什么作用？

  1.listen_addresses='*'代表允许postgres server监听所有IP地址（包括ipv4与ipv6）

  2.Host all all 0.0.0.0/0 md5代表允许所有用户均可以通过密码验证，连接所有数据库。

  3.修改listen_addresses需要数据库重启才能生效。
  

### 三、更多

  `关于listen_addresses，这里仅仅介绍了一种，该参数具体用法，我会专门写一篇blog阐述。`
  
  `pg_hba.conf的设置规则，我也会专门写一篇blog阐述。`
