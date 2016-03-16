#1.启动hbase thrift
```
cd hbase/bin
./hbase-daemon.sh start thrift2
检查thrift端口
netstat -ln | grep 9090
```

#2.未整理
```
郑举  14:54:32
测了一下，c++里filter可以这么写：
std::string filterStr;
            //SingleColumnValueFilter('<family>', '<qualifier>', <compare operator>, '<comparator>', <filterIfColumnMissing_boolean>, <latest_version_boolean>)
            filterStr = "SingleColumnValueFilter('col1', 'qua', =, 'regexstring:tes(.*)er', true, false)";
            tScan.__set_filterString(filterStr);

             client.getClient()->getScannerResults(tResult, "test", tScan, 1000);
李璐  14:56:05

/SingleColumnValueFilter('<family>', '<qualifier>', <compare operator>, '<comparator>', <filterIfColumnMissing_boolean>, <latest_version_boolean>)

这个语法从哪里查到的
郑举  14:56:39
网上搜的哈
李璐  14:57:37
不错。
这个 filterStr = "SingleColumnValueFilter('col1', 'qua', =, 'regexstring:tes(.*)er', true, false)";
是什么意思
郑举  14:59:39
匹配正则表达式 搜索值为 ： tes中间是任意字符最后是er的 行
郑举  15:00:46
http://blog.csdn.net/yangyangghy/article/details/39010365
http://blog.csdn.net/guxch/article/details/12163047
http://hbase.apache.org/book.html#thrift
http://stackoverflow.com/questions/11380897/how-to-use-method-scanneropenwithscan-and-filter-string-in-thrifts-c-to-a
综合这几个链接的结果哈
郑举  15:02:21
测试数据：
hbase(main):010:0> scan 'test'
ROW                                                                  COLUMN+CELL                                                                                                                                                                                              
 row1                                                                column=col1:, timestamp=1456372547448, value=1                                                                                                                                                           
 row1                                                                column=col1:qua, timestamp=1456802575635, value=testfilter                                                                                                                                               
 row2                                                                column=col1:qua, timestamp=1456380851726, value=2                                                                                                                                                        
 row3                                                                column=col1:abc, timestamp=1456386394813, value=3                                                                                                                                                        
 row4                                                                column=col1:qua, timestamp=1456814453511, value=testfilterandmore  
李璐  15:03:31
然后 shell中过滤怎么写
郑举  15:06:50
shell不是很清楚
郑举  15:49:56
http://hbase.apache.org/0.94/book/thrift.html
所有filter的格式
李璐  15:50:37
突然感觉功能很全了
郑举  16:03:58
嗯，查询的功能比较全了
郑举  16:19:59
http://www.hadooptpoint.com/filters-in-hbase-shell/
filter shell用法
```
