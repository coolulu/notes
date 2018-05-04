# mongodb

# 目录
[toc]

## mongodb集群
[参考](https://docs.mongodb.com/manual/sharding/)

[参考](https://docs.mongodb.com/manual/replication/)

### 前期准备

#### 集群IP端口分配
```
mongos:
    192.168.226.128:27101

config:
    1: 192.168.226.128:27001
    2: 192.168.226.128:27002
    3: 192.168.226.128:27003

shard_1:
    1: 192.168.226.128:27211
    2: 192.168.226.128:27212
    3: 192.168.226.128:27213

shard_2:
    1: 192.168.226.128:27221
    2: 192.168.226.128:27222
    3: 192.168.226.128:27223

shard_3:
    1: 192.168.226.128:27231
    2: 192.168.226.128:27232
    3: 192.168.226.128:27233

shard_4:   #添加/删除分片时用到
    1: 192.168.226.128:27241
    2: 192.168.226.128:27242
    3: 192.168.226.128:27243
```

#### 创建用户
```
useradd mg
```

#### 目录结构
```
[mg@localhost mongodb]$ pwd
/home/mg/mongodb

[mg@localhost mongodb]$ find
./config
./config/etc
./config/etc/config_1.conf
./config/etc/config_2.conf
./config/etc/config_3.conf
./config/log
./config/data
./config/data/1
./config/data/2
./config/data/3

./shard_1
./shard_1/data
./shard_1/data/1
./shard_1/data/2
./shard_1/data/3
./shard_1/etc
./shard_1/etc/shard_1.conf
./shard_1/etc/shard_2.conf
./shard_1/etc/shard_3.conf
./shard_1/log

./shard_2
./shard_2/data
./shard_2/data/1
./shard_2/data/2
./shard_2/data/3
./shard_2/etc
./shard_2/etc/shard_1.conf
./shard_2/etc/shard_2.conf
./shard_2/etc/shard_3.conf
./shard_2/log

./shard_3
./shard_3/data
./shard_3/data/1
./shard_3/data/2
./shard_3/data/3
./shard_3/etc
./shard_3/etc/shard_1.conf
./shard_3/etc/shard_2.conf
./shard_3/etc/shard_3.conf
./shard_3/log

./mongos
./mongos/etc
./mongos/etc/mongos.conf
./mongos/log

./stop.sh
./start.sh
./ps.sh
./ns.sh
```

### 创建脚本
```
[mg@localhost mongodb]$ ls *.sh
ns.sh  ps.sh  start.sh  stop.sh
```

#### mongodb启动
```
[mg@localhost mongodb]$ cat start.sh
mongod -f config/etc/config_1.conf
mongod -f config/etc/config_2.conf
mongod -f config/etc/config_3.conf
mongod -f shard_1/etc/shard_1.conf
mongod -f shard_1/etc/shard_2.conf
mongod -f shard_1/etc/shard_3.conf
mongod -f shard_2/etc/shard_1.conf
mongod -f shard_2/etc/shard_2.conf
mongod -f shard_2/etc/shard_3.conf
mongod -f shard_3/etc/shard_1.conf
mongod -f shard_3/etc/shard_2.conf
mongod -f shard_3/etc/shard_3.conf
```

#### mongodb关闭
```
[mg@localhost mongodb]$ cat stop.sh
mongod  --shutdown --dbpath /home/mg/mongodb/config/data/1
mongod  --shutdown --dbpath /home/mg/mongodb/config/data/2
mongod  --shutdown --dbpath /home/mg/mongodb/config/data/3
mongod  --shutdown --dbpath /home/mg/mongodb/shard_1/data/1
mongod  --shutdown --dbpath /home/mg/mongodb/shard_1/data/2
mongod  --shutdown --dbpath /home/mg/mongodb/shard_1/data/3
mongod  --shutdown --dbpath /home/mg/mongodb/shard_2/data/1
mongod  --shutdown --dbpath /home/mg/mongodb/shard_2/data/2
mongod  --shutdown --dbpath /home/mg/mongodb/shard_2/data/3
mongod  --shutdown --dbpath /home/mg/mongodb/shard_3/data/1
mongod  --shutdown --dbpath /home/mg/mongodb/shard_3/data/2
mongod  --shutdown --dbpath /home/mg/mongodb/shard_3/data/3
```

#### 其他
```
[mg@localhost mongodb]$ cat ns.sh
netstat -lnap | grep 'LISTEN .*mongo'

[mg@localhost mongodb]$ cat ps.sh
ps -ef | grep 'mongo'
```

### 配置文件

#### config集群
```
[mg@localhost etc]$ cat config_1.conf
## content
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/config/log/config_1.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/config/data/1
  journal:
    enabled: true
# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/config/config_1.pid

# network interfaces
net:
  port: 27001
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: config

sharding:
    clusterRole: configsvr

-----------------------------------------
[mg@localhost etc]$ cat config_2.conf
## content
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/config/log/config_2.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/config/data/2
  journal:
    enabled: true
# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/config/config_2.pid

# network interfaces
net:
  port: 27002
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: config

sharding:
    clusterRole: configsvr

-----------------------------------------
[mg@localhost etc]$ cat config_3.conf
## content
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/config/log/config_3.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/config/data/3
  journal:
    enabled: true
# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/config/config_3.pid

# network interfaces
net:
  port: 27003
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: config

sharding:
    clusterRole: configsvr
```

#### shard_1集群
```
[mg@localhost etc]$ cat shard_1.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_1/log/shard_1.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_1/data/1
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_1/shard_1.pid

# network interfaces
net:
  port: 27211
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_1
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_2.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_1/log/shard_2.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_1/data/2
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_1/shard_2.pid

# network interfaces
net:
  port: 27212
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_1
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_3.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_1/log/shard_3.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_1/data/3
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_1/shard_3.pid

# network interfaces
net:
  port: 27213
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_1
sharding:
    clusterRole: shardsvr
```

#### shard_2集群
```
[mg@localhost etc]$ cat shard_1.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_2/log/shard_1.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_2/data/1
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_2/shard_1.pid

# network interfaces
net:
  port: 27221
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_2
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_2.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_2/log/shard_2.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_2/data/2
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_2/shard_2.pid

# network interfaces
net:
  port: 27222
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_2
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_3.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_2/log/shard_3.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_2/data/3
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_2/shard_3.pid

# network interfaces
net:
  port: 27223
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_2
sharding:
    clusterRole: shardsvr
```

#### shard_3集群
```
[mg@localhost etc]$ cat shard_1.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_3/log/shard_1.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_3/data/1
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_3/shard_1.pid

# network interfaces
net:
  port: 27231
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_3
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_2.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_3/log/shard_2.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_3/data/2
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_3/shard_2.pid

# network interfaces
net:
  port: 27232
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_3
sharding:
    clusterRole: shardsvr

-----------------------------------------
[mg@localhost etc]$ cat shard_3.conf
#配置文件内容
# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /home/mg/mongodb/shard_3/log/shard_3.log

# Where and how to store data.
storage:
  dbPath: /home/mg/mongodb/shard_3/data/3
  journal:
    enabled: true
  wiredTiger:
    engineConfig:
       cacheSizeGB: 2

# how the process runs
processManagement:
  fork: true
  pidFilePath: /home/mg/mongodb/shard_3/shard_3.pid

# network interfaces
net:
  port: 27233
  bindIp: 0.0.0.0

#operationProfiling:
replication:
    replSetName: shard_3
sharding:
    clusterRole: shardsvr
```

### 启动mongod集群
```
./start.sh

[mg@localhost mongodb]$ ./ps.sh
mg        44417      1  0 20:36 ?        00:00:08 mongod -f config/etc/config_1.conf
mg        44452      1  0 20:36 ?        00:00:08 mongod -f config/etc/config_2.conf
mg        44487      1  0 20:36 ?        00:00:07 mongod -f config/etc/config_3.conf
mg        44522      1  0 20:36 ?        00:00:07 mongod -f shard_1/etc/shard_1.conf
mg        44549      1  0 20:36 ?        00:00:05 mongod -f shard_1/etc/shard_2.conf
mg        44576      1  0 20:36 ?        00:00:07 mongod -f shard_1/etc/shard_3.conf
mg        44603      1  0 20:36 ?        00:00:07 mongod -f shard_2/etc/shard_1.conf
mg        44630      1  0 20:36 ?        00:00:05 mongod -f shard_2/etc/shard_2.conf
mg        44660      1  0 20:36 ?        00:00:06 mongod -f shard_2/etc/shard_3.conf
mg        44687      1  0 20:36 ?        00:00:07 mongod -f shard_3/etc/shard_1.conf
mg        44714      1  0 20:36 ?        00:00:05 mongod -f shard_3/etc/shard_2.conf
mg        44741      1  0 20:36 ?        00:00:06 mongod -f shard_3/etc/shard_3.conf

[mg@localhost mongodb]$ ./ns.sh
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:27211           0.0.0.0:*               LISTEN      44522/mongod
tcp        0      0 0.0.0.0:27212           0.0.0.0:*               LISTEN      44549/mongod
tcp        0      0 0.0.0.0:27213           0.0.0.0:*               LISTEN      44576/mongod
tcp        0      0 0.0.0.0:27221           0.0.0.0:*               LISTEN      44603/mongod
tcp        0      0 0.0.0.0:27222           0.0.0.0:*               LISTEN      44630/mongod
tcp        0      0 0.0.0.0:27223           0.0.0.0:*               LISTEN      44660/mongod
tcp        0      0 0.0.0.0:27001           0.0.0.0:*               LISTEN      44417/mongod
tcp        0      0 0.0.0.0:27002           0.0.0.0:*               LISTEN      44452/mongod
tcp        0      0 0.0.0.0:27003           0.0.0.0:*               LISTEN      44487/mongod
tcp        0      0 0.0.0.0:27231           0.0.0.0:*               LISTEN      44687/mongod
tcp        0      0 0.0.0.0:27232           0.0.0.0:*               LISTEN      44714/mongod
tcp        0      0 0.0.0.0:27233           0.0.0.0:*               LISTEN      44741/mongod
```

### 配置mongod集群

#### 配置config主从
```
登录config
mongo 127.0.0.1:27001

#Arbiters are not allowed in replica set configurations being used for config servers
#仲裁者是不允许在副本配置用于配置服务器
config = {
   _id : "config",
    members : [
        {_id : 1, host : "192.168.226.128:27001" },
        {_id : 2, host : "192.168.226.128:27002" },
        {_id : 3, host : "192.168.226.128:27003" }
    ]
}

rs.initiate(config);
rs.status()
```

#### 配置shard_1副本集
```
登录shard_1
mongo 127.0.0.1:27211

config = {
   _id : "shard_1",
    members : [
        {_id : 1, host : "192.168.226.128:27211" },
        {_id : 2, host : "192.168.226.128:27212", arbiterOnly: true },
        {_id : 3, host : "192.168.226.128:27213" }
    ]
}

rs.initiate(config);
rs.status()
```

#### 配置shard_2副本集
```
登录shard_2
mongo 127.0.0.1:27221

config = {
   _id : "shard_2",
    members : [
        {_id : 1, host : "192.168.226.128:27221" },
        {_id : 2, host : "192.168.226.128:27222", arbiterOnly: true },
        {_id : 3, host : "192.168.226.128:27223" }
    ]
}

rs.initiate(config);
rs.status()
```

#### 配置shard_3副本集
```
登录shard_3
mongo 127.0.0.1:27231

config = {
   _id : "shard_3",
    members : [
        {_id : 1, host : "192.168.226.128:27231" },
        {_id : 2, host : "192.168.226.128:27232", arbiterOnly: true},
        {_id : 3, host : "192.168.226.128:27233" }
    ]
}

rs.initiate(config);
rs.status()
```

### 启动mongos
```
#mongodb集群配置好后，启动mongos
mongos -f mongos/etc/mongos.conf
```

### mongos配置
```
#登录mongos, 添加分片
mongo 127.0.0.1:27101

sh.addShard("shard_1/192.168.226.128:27211,192.168.226.128:27212,192.168.226.128:27213")
sh.addShard("shard_2/192.168.226.128:27221,192.168.226.128:27222,192.168.226.128:27223")
sh.addShard("shard_3/192.168.226.128:27231,192.168.226.128:27232,192.168.226.128:27233")
sh.status()
```

#### 创建数据库和表
```
use im_test
db.createCollection("test_2")
```

#### 分片设置
```
use admin
db.runCommand({"enablesharding":"im_test"})                 #设置分片库
#db.runCommand({"shardcollection":"im_test.test", "key":{"_id":1}})         #设置分片文档
db.runCommand({"shardcollection":"im_test.test_2", "key":{"kk":"hashed"}})  #设置哈希分片文档
```
[参考](https://docs.mongodb.com/manual/core/sharding-shard-key/)

[参考](https://docs.mongodb.com/manual/core/hashed-sharding/)

#### 测试分片
```
use im_test
for (var i = 0; i < 100; i++) {
    db.test_2.insert({kk: i, name: "MACLEAN"})
}
```

#### 分片查看
```
sh.status() #如果分片太多显示不完,用printShardingStatus(db.getSisterDB("config"),1)

use config
db.databases.find()
db.chunks.find()
db.settings.find()
```

#### 平衡分片
```
sh.getBalancerState()   #是否开启平衡
sh.isBalancerRunning()  #是否正在平衡
sh.startBalancer()  #手动执行平衡
```
[参考](https://docs.mongodb.com/manual/tutorial/manage-sharded-cluster-balancer/)

### 节点操作

#### 添加节点
配置shard_4副本集
```
登录shard_4
mongo 127.0.0.1:27241

config = {
   _id : "shard_4",
    members : [
        {_id : 1, host : "192.168.226.128:27241" },
        {_id : 2, host : "192.168.226.128:27242", arbiterOnly: true },
        {_id : 3, host : "192.168.226.128:27243" }
    ]
}

rs.initiate(config);
rs.status()
```

#登录mongos, 添加分片shard_4副本集
mongo 127.0.0.1:27101
```
sh.addShard("shard_4/192.168.226.128:27241,192.168.226.128:27242,192.168.226.128:27243")
sh.status()
```

#### 删除节点
删除分片shard_4副本集
```
db.adminCommand({ removeShard : "shard_4" }),返回
{
    "msg" : "draining started successfully",
    "state" : "started",
    "shard" : "shard_3",
    "note" : "you need to drop or movePrimary these databases",
    "dbsToMove" : [ ],
    "ok" : 1
}

sh.status()查看shard_4已经是 shards: shard_4已经"draining" : true

db.runCommand({ listShards : 1})，返回
{
    "_id" : "shard_4",
    "host" : "shard_4/192.168.226.128:27241,192.168.226.128:27243",
    "draining" : true,
    "state" : 1
}

在运行db.adminCommand( { removeShard : "shard_4" } )，返回
{
    "msg" : "draining ongoing",
    "state" : "ongoing",
    "remaining" : {
        "chunks" : NumberLong(5),
        "dbs" : NumberLong(0)
    },
    "note" : "you need to drop or movePrimary these databases",
    "dbsToMove" : [ ],
    "ok" : 1
}

在运行db.adminCommand( { removeShard : "shard_4" } )，直到返回
{
    "msg" : "removeshard completed successfully",
    "state" : "completed",
    "shard" : "shard_4",
    "ok" : 1
}

在运行一次db.adminCommand( { removeShard : "shard_4" } )，返回
{
    "ok" : 0,
    "errmsg" : "Could not drop shard 'shard_4' because it does not exist",
    "code" : 70,
    "codeName" : "ShardNotFound"
}

sh.status() 查看shard_4已经不再shards里，
shard_4后面怎么搞随意，但如果要重新加入集群，要把shard_4的数据库全部删掉
```
[参考](https://docs.mongodb.com/manual/reference/command/removeShard/index.html)

## 数据操作

### 数据导出(bson)
```
mkdir dump
mongodump --host 127.0.0.1 --port 27101 -d im_test -o dump/

[mg@localhost mongodb]$ find dump/
dump/
dump/im_test
dump/im_test/test_4.metadata.json
dump/im_test/test_3.metadata.json
dump/im_test/test_2.metadata.json
dump/im_test/test_2.bson
dump/im_test/test_3.bson
dump/im_test/test_4.bson
```
[参考](https://docs.mongodb.com/manual/reference/program/mongodump/index.html)

### 数据导入(bson)
```
删除数据库im_test
创建数据库im_test,设置片键
use im_test

use admin
db.runCommand({"shardcollection":"im_test.test_2", "key":{"kk":"hashed"}})
db.runCommand({"shardcollection":"im_test.test_3", "key":{"kk":"hashed"}})
db.runCommand({"shardcollection":"im_test.test_4", "key":{"kk":"hashed"}})

导入数据
mongorestore --host 127.0.0.1 --port 27101 -d im_test dump/im_test/
```
[参考](https://docs.mongodb.com/manual/reference/program/mongorestore/index.html)

### bson数据查看
```
[mg@localhost im_test]$ bsondump test_2.bson
```
[参考](https://docs.mongodb.com/manual/reference/program/bsondump/)

### 数据导出(文本格式)
[参考](https://docs.mongodb.com/manual/reference/program/mongoimport/)

### 数据导入(文本格式)
[参考](https://docs.mongodb.com/manual/reference/program/mongoexport/)

## 服务监控

### mongotop
```
mongotop不能监控mongos,只能监控mongod
[mg@localhost mongodb]$ mongotop --host 127.0.0.1 --port 27101 10
2018-05-04T21:17:32.826+0800    cannot run mongotop against a mongos

#10是间隔时间
[mg@localhost mongodb]$  mongotop --host 127.0.0.1 --port 27211 10
2018-05-04T21:19:59.121+0800    connected to: 127.0.0.1:27211

                    ns    total    read    write    2018-05-04T21:20:04+08:00
        local.oplog.rs      2ms     2ms      0ms
    admin.system.roles      0ms     0ms      0ms
    admin.system.users      0ms     0ms      0ms
  admin.system.version      0ms     0ms      0ms
im_test.system.indexes      0ms     0ms      0ms
        im_test.test_2      0ms     0ms      0ms
        im_test.test_3      0ms     0ms      0ms
        im_test.test_4      0ms     0ms      0ms
              local.me      0ms     0ms      0ms
local.replset.election      0ms     0ms      0ms
```
[参考](https://docs.mongodb.com/manual/reference/program/mongotop/index.html)

### mongostat
```
mongostat --host 127.0.0.1 --port 27101
```
[参考](https://docs.mongodb.com/manual/reference/program/mongostat/index.html)

### mongoperf
[参考](https://docs.mongodb.com/manual/reference/program/mongoperf/)

### mongoreplay
[参考](https://docs.mongodb.com/manual/reference/program/mongoreplay/)

### mongoldap
[参考](https://docs.mongodb.com/manual/reference/program/mongoldap/)

## 用户管理
### role类型
```
数据库用户角色（Database User Roles）：
    read：授予User只读数据的权限
    readWrite：授予User读写数据的权限

数据库管理角色（Database Administration Roles）：
    dbAdmin：在当前dB中执行管理操作
    dbOwner：在当前DB中执行任意操作
    userAdmin：在当前DB中管理User

备份和还原角色（Backup and Restoration Roles）：
    backup
    restore

跨库角色（All-Database Roles）：
    readAnyDatabase：授予在所有数据库上读取数据的权限
    readWriteAnyDatabase：授予在所有数据库上读写数据的权限
    userAdminAnyDatabase：授予在所有数据库上管理User的权限
    dbAdminAnyDatabase：授予管理所有数据库的权限l

集群管理角色（Cluster Administration Roles）：
    clusterAdmin：授予管理集群的最高权限
    clusterManager：授予管理和监控集群的权限
    clusterMonitor：授予监控集群的权限，对监控工具具有readonly的权限
    hostManager：管理Server
```
[参考](https://docs.mongodb.com/manual/reference/built-in-roles/index.html)

### 创建用户
```
#创建管理员用户
use admin
db.createUser(
   {
     user: "root",
     pwd: "zxcvbnm",
     roles: [ {role:"root", db:"admin"} ]
   }
)

db.system.users.find();

#创建普通用户
#创建数据库
use im_test
db.createUser(
   {
     user: "im_test",
     pwd: "im_test",
     roles: [ {role:"readWrite", db:"im_test"} ]
   }
)
```

## 其他
[参考](https://docs.mongodb.com/manual/)


