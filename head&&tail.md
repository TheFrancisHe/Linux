```
导入测试数据前，一定要询问好，是否可以关闭归档，这是最重要的。
否则，慢腾腾的熬死你。


[postgres@sds1 archivedir]$ ls -lrth|tail -10//按时间排序后，显示最后10条
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F1
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F2
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F3
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F4
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F5
-rw-------. 1 postgres postgres  16M Dec 15 14:54 0000000500000091000000F6
-rw-------. 1 postgres postgres  16M Dec 15 14:55 0000000500000091000000F7
-rw-------. 1 postgres postgres  16M Dec 15 14:55 0000000500000091000000F8
-rw-------. 1 postgres postgres  16M Dec 15 14:55 0000000500000091000000F9
-rw-------. 1 postgres postgres 9.8M Dec 15 14:56 0000000500000091000000FA
[postgres@sds1 archivedir]$ pg_archivecleanup ./ 0000000500000091000000F9
```
