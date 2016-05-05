#1.杀死占用的端口进程 或者 遇到 Address already in use 
```
fuser -k 9999/tcp
```

#2.生成随机密钥：
```
cat /dev/urandom | base64 | head -n1 
```

#3.统计工程树下的所有 .cpp .h源码的非空行行数：
```
find . -name '*.cpp' | xargs cat | grep . | wc -l
find . -name '*.h' | xargs cat | grep . | wc -l
```

#4.linux各模块工具
![linux各模块工具](../_image/linux_debug.png)

#5.date
```
date -d @1456329600     #时间戳转字符串时间
Thu Feb 25 00:00:00 CST 2016
```
#6.top
```
在top中按M(按内存排序), P(按cpu占有率排序)
top -H -p 进程id #查看进程里的线程
```

#7.gstack
```
查看进程或线程的堆栈，方便定位cpu占有率高的问题
gstack 进程id
gstack 线程id
```

#8.tar
```
tar zxvf *.tar.gz
tar jxvf *.tar.bz2
```

#9.strace
```

```

#10.netstat
```

```

#11.nc
```

```

#12.curl
```

```

#13.ps
```

```

#14.cut
```

```

#15.split
```

```

#16.nl
```

```

#17.wc
```

```

#18.gerp
```
ps -ef | grep MyProc | grep -v 'grep'  #反向匹配
```

#19.sed
```

```

#20.xargs
```

```

#21.find
```

```
