进入正文前，需要掌握以下知识：

- 关于网络的一个知识点

`localhost 是个域名，不是地址，它可以被配置为任意的 IP 地址，通常情况下都指向 127.0.0.1(ipv4)和 ::1(ipv6)`

***

- 关于listen_addresses的简单理解：

 `允许Postgres server 监听程序绑定在某种类型（IPV4/IPV6）IP 或者某个具体IP上`
 
 `因为我们知道一个host上可以有多个网卡，每个网卡也可以绑定多个IP，该参数就是控制Postgres到底绑定在哪个或者哪几个IP上`

***
 
 
 如果postgres所在服务器，一共绑定了5个不同类型的IP，
 
 分别是192.168.100.2(IPV4)/192.168.100.3(IPV4)/192.168.100.4(IPV4)/192.168.100.5(IPV6)/192.168.100.6(IPV6)
 
 ` 当listen_addresses=192.168.100.5 时，这代表postgres只绑定在192.168.100.5,只监听所有发给192.168.100.5的请求。`
 
 ` 当listen_addresses=* 时，这代表postgres监听程序绑定在所有本地IP上。`
 
 **重点是"本地IP"**
 
 再形象描述一遍：
 
 ```
 当listen_addresses='*' 时，
 
 pgadmin客户端均可以通过以上五个IP+5432的端口号来给连接postgres并给postgres发请求。
 
 当listen_addresses='::' 这时候所有的pgadmin只能通过192.168.100.5(IPV6)或者192.168.100.6(IPV6)作为连接信息来连接了。
 
 ```
***

**以下场景只是针对TCP-IP的监听程序绑定情况，不考虑udp，在介绍 unix_socket_directories 时，会详细表述udp下的监听绑定情况**
 
***
正文开始

#### 官方文档是如何解释listen_addresses的呢？
 
 

### listen_addresses (string)



>Specifies the TCP/IP address(es) on which the server is to listen for connections from client applications. 

指定server使用那个interface（如IPV4、IPV6（若支持））进行监听。

>The value takes the form of a comma-separated list of host names and/or numeric IP addresses.

参数值value可以是 hostname 也可以是具体的IP地址或者主机名(hostname)，若配置多个则必须用,分开。

```
意思如下：
postgres=# alter system set listen_addresses = '127.0.0.1,::1';
ALTER SYSTEM

值可以是 127.0.0.1,0.0.0.0,::,::1,localhost,*,::,0.0.0.0,以及所有本地网卡所绑定IP 的排列组合
```

***

>The special entry * corresponds to all available IP interfaces. 

若值为*，则代表监听程序绑定在所有可用的IP地址上。

```
[root@pgdb ~]# netstat -anpo|grep 1921

//代表监听程序已经绑定在本地所有的IPV4 地址上，本地有几个ipv4类型的IP，监听就注册几次。

tcp        0      0 0.0.0.0:1921    0.0.0.0:*     LISTEN      28794/postgres      off (0.00/0/0)
.
//代表监听程序已经绑定在本地所有的IPV6 地址上，本地有几个ipv6类型的IP，监听就注册几次。
tcp        0      0 :::1921          :::*         LISTEN      28794/postgres      off (0.00/0/0)

结论：可以看出配置为*时，监听程序注册在本地所有的类型的所有IP地址上，
      意味着客户端可以通过连接所有本地IP进行数据库连接

unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```

***

>The entry 0.0.0.0 allows listening for all IPv4 addresses and :: allows listening for all IPv6 addresses.

若值为 0.0.0.0，则监听程序只绑定在本地所有的IPV4 地址上；若值为::，监听程序只绑定在本地所有的IPV6 地址上。

```
参数值：0 0.0.0.0

参数设置成0.0.0.0，且本地只有一个IPV4的IP的监听情况：
tcp        0      0 0.0.0.0:1921  0.0.0.0:*      LISTEN      29816/postgres      off (0.00/0/0)

//udp
unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```
***

>If the list is empty, the server does not listen on any IP interface at all, in which case only Unix-domain sockets can be used to connect to it.

当listen_addresses参数值为空，postgres server 则监听程序将不会注册在TCP-IP协议的网卡，此时只允许客户端通过Unix-domain sockets 连接。

```
[root@pgdb ~]# netstat -anpo|grep 1921
//可以看出，通过tcp-ip连接的应用程序进程状态均为FIN_WAIT2,或者CLOSE_WAIT.因为此时postgres不监听任何tcp-ip类型请求。
tcp        0      0 127.0.0.1:1921              127.0.0.1:48090             FIN_WAIT2   -                   timewait (54.46/0/0)
tcp        0      0 127.0.0.1:1921              127.0.0.1:48377             FIN_WAIT2   -                   timewait (54.46/0/0)
tcp      111      0 127.0.0.1:48009             127.0.0.1:1921              CLOSE_WAIT  2326/zabbix_server  off (0.00/0/0)

//这时，postgres server 只对unix-socket domain进行监听。
unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```
***
>The default value is localhost, which allows only local TCP/IP "loopback" connections to be made. 

默认值是localhost，则监听程序仅仅注册在本地/etc/hosts中localhost对应的IP地址上(一般是127.0.0.1或::1)。

这意味着，只有本机客户端只能通过localhost 连接数据库，或者通过udp连接数据库*

```
//ipv4的连接
tcp        0      0 127.0.0.1:1921              0.0.0.0:*                   LISTEN      31739/postgres      off (0.00/0/0)

//ipv6的连接
tcp        0      0 ::1:1921                    :::*                        LISTEN      31739/postgres      off (0.00/0/0)

unix  2      [ ACC ]     STREAM     LISTENING     281562 31739/postgres      ./.s.PGSQL.1921

-------------------------------
cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4  //ipv4解析规则
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6  //ipv6解析规则

-------------------------------

原理： localhost 根据/etc/hosts里的规则，最终被解析成 127.0.0.1和::1 。

当注释掉ipv6解析规则后，再来看看postgres server是否还监听本地的ipv6 ？
-------------------------------
cat /etc/hosts

127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4  //ipv4解析规则
#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6  //ipv6解析规则

-------------------------------
//这时，仅仅监听IPV4的本地连接，不再监听本地IPV6连接了。

tcp        0      0 127.0.0.1:1921              0.0.0.0:*                   LISTEN      31957/postgres      off (0.00/0/0)


unix  2      [ ACC ]     STREAM     LISTENING     281562 31739/postgres      ./.s.PGSQL.1921
```
***

>While client authentication (Chapter 19) allows fine-grained control over who can access the server, 
listen_addresses controls which interfaces accept connection attempts, which can help prevent repeated malicious connection requests on insecure network interfaces. 

对比来看，client authentication（客户端认证）是用于控制具体哪些客户端可以访问服务器，
而listen_addresses则是控制postgres server 具体使用哪个(IPV4/IPV6/BOTH)网络接口(interface)进行监听连接请求,对于网络接口而言，这可以有效阻止对大量恶意重复的连接

***

>This parameter can only be set at server start.

参数修改后必须重启

***

**listen_addresses 参数作为服务器端的一种连接过滤方式，修改时，必须结合以下几点：**

在之后的文章会介绍以下几点：

    1./etc/hosts
    2.unix_socket_directories
    3.pg_hba.conf
    4.当前OS对于IPV4与IPV6的支持情况



