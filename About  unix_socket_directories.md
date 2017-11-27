
>unix_socket_directories (string)
Specifies the directory of the Unix-domain socket(s) on which the server is to listen for connections from client applications. Multiple sockets can be created by listing multiple directories separated by commas. Whitespace between entries is ignored; surround a directory name with double quotes if you need to include whitespace or commas in the name. An empty value specifies not listening on any Unix-domain sockets, in which case only TCP/IP sockets can be used to connect to the server. The default value is normally /tmp, but that can be changed at build time. This parameter can only be set at server start.

定义udp连接所需socket的文件路径（我们知道，udp连接需要一个socket文件路径）在该路径下，postgres master进程监听来自客户端的udp连接。
定义参数时，空格被忽略；可以定义多个unix-socket路径；如果定义成空值，则表明只允许TCP-IP的方式连接服务器。默认的是/tmp，当然默认设置可以在
编译时候确定。

>In addition to the socket file itself, which is named .s.PGSQL.nnnn where nnnn is the server's port number, an ordinary file named .s.PGSQL.nnnn.lock will be created in each of the unix_socket_directories directories. Neither file should ever be removed manually.
这个参数和windows无关。



>PGHOST behaves the same as the host connection parameter.

PGHOST环境变量其实就是 libpq连接时候的host参数。

>host
Name of host to connect to. If this begins with a slash, it specifies Unix-domain communication rather than TCP/IP communication; the value is the name of the directory in which the socket file is stored. The default behavior when host is not specified is to connect to a Unix-domain socket in /tmp (or whatever socket directory was specified when PostgreSQL was built). On machines without Unix-domain sockets, the default is to connect to localhost.

host如果以 / 开始，则走的是udp连接，而不是tcp-ip。值默认是 /tmp （或者在编译时，指定的路径），这个值代表 unix-domain socket 文件存在的位置。
如果所在OS或者machine并不支持 udp- 那么默认是tcp-ip方式连接。



- unix_socket_directories（这是参数）

- host （在连接字符串里）

这两个参数决定走的是 udp 还是 tcp-ip

