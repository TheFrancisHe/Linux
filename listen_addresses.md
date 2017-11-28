关于IPV4和IPV6下的localhost：

`localhost 是个域名，不是地址，它可以被配置为任意的 IP 地址，通常情况下都指向 127.0.0.1(ipv4)和 ::1`

### listen_addresses (string)



>Specifies the TCP/IP address(es) on which the server is to listen for connections from client applications. 

指定server使用那个interface（如IPV4、IPV6（若支持））进行监听，甚至不监听客户端连接。

>The value takes the form of a comma-separated list of host names and/or numeric IP addresses.

参数值value可以是 hostname 也可以是具体的IP地址，多个必须用,分开。

```
意思如下：
postgres=# alter system set listen_addresses = '127.0.0.1,::1';
ALTER SYSTEM

值可以是 127.0.0.1,0.0.0.0,::,::1,localhost,* 的排列组合
```


>The special entry * corresponds to all available IP interfaces. 

若值为*，则代表 既监听IPV4的请求，又监听IPV6的请求。

```
[root@pgdb ~]# netstat -anpo|grep 1921

//代表监听所有的IPV4 地址

tcp        0      0 0.0.0.0:1921    0.0.0.0:*     LISTEN      28794/postgres      off (0.00/0/0)
.
//代表监听所有的IPV6 地址
tcp        0      0 :::1921          :::*         LISTEN      28794/postgres      off (0.00/0/0)

可以看出,既有IPV4的监听，也有IPV6的监听。

unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```

>The entry 0.0.0.0 allows listening for all IPv4 addresses and :: allows listening for all IPv6 addresses.

若值为 0.0.0.0，则允许监听所有IPV4的地址；若值为::，则允许监听所有IPV6的地址。

```
可以看出tcp-ip连接只有一个
tcp        0      0 0.0.0.0:1921  0.0.0.0:*      LISTEN      29816/postgres      off (0.00/0/0)

//udp
unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```


>If the list is empty, the server does not listen on any IP interface at all, in which case only Unix-domain sockets can be used to connect to it.

```
[root@pgdb ~]# netstat -anpo|grep 1921
//可以看出，所有通过localhost连接的连接状态要么是 FIN_WAIT2 要么是 CLOSE_WAIT
tcp        0      0 127.0.0.1:1921              127.0.0.1:48090             FIN_WAIT2   -                   timewait (54.46/0/0)
tcp        0      0 127.0.0.1:1921              127.0.0.1:48377             FIN_WAIT2   -                   timewait (54.46/0/0)
tcp      111      0 127.0.0.1:48009             127.0.0.1:1921              CLOSE_WAIT  2326/zabbix_server  off (0.00/0/0)

//这时，postgres server 只对unix-socket domain进行监听。
unix  2      [ ACC ]     STREAM     LISTENING     265956 29816/postgres      ./.s.PGSQL.1921
```

>The default value is localhost, which allows only local TCP/IP "loopback" connections to be made. 

默认值是localhost，它只允许本地客户端/应用程序 TCP-IP 进行 TCP/IP "loopback" 连接。

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

>While client authentication (Chapter 19) allows fine-grained control over who can access the server, 
listen_addresses controls which interfaces accept connection attempts, which can help prevent repeated malicious connection requests on insecure network interfaces. 

`对比来看，client authentication（客户端认证）是用于控制具体哪些客户端可以访问服务器，
而listen_addresses则是控制postgres server 具体使用哪个(IPV4/IPV6/BOTH)网络接口(interface)进行监听连接请求,对于网络接口而言，这可以有效阻止对大量恶意重复的连接。`


>This parameter can only be set at server start.

参数修改后必须重启


**listen_addresses 参数作为服务器端的一种连接过滤方式，修改时，必须结合以下几点：**


    1./etc/hosts
    2.unix_socket_directories
    3.pg_hba.conf
    4.当前OS对于IPV4与IPV6的支持情况



