# ELK

# 目录
[toc]

## 前期准备
```
单机环境ip: 192.168.226.128

$ ls
jdk-8u171-linux-x64.tar.gz
elasticsearch-6.2.4.tar.gz
kibana-6.2.4-linux-x86_64.tar.gz
filebeat-6.2.4-linux-x86_64.tar.gz
logstash-6.2.4.tar.gz

Elasticsearch， Logstash依赖jdk

log <-- filebeat --> logstash --> elasticsearch <-- kibana
```

## 安装jdk
```
$ tar zxvf jdk-8u171-linux-x64.tar.gz
$ mv jdk-8u171-linux-x64 /usr/local/bin/
$ vim /etc/profile
添加
export JAVA_HOME=/usr/local/bin/jdk1.8.0_171
export JRE_HOME=/usr/local/bin/jdk1.8.0_171/jre
export CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH

$ source /etc/profile
```

## Elasticsearch
Elasticsearch的默认账户为 elastic 默认密码为 changeme

### 创建账号
```
不能用root启动，创建账号/密码
useradd elk
passwd elk
```

### 修改配置
```
$ vim config/elasticsearch.yml
network.host: 0.0.0.0

$vim config/jvm.options
-Xms1g改成-Xms4g
-Xmx1g改成-Xmx4g
```
### 运行
```
$ bin/elasticsearch     #前台启动
$ bin/elasticsearch -d  #后台启动

启动失败，检查没有通过，报两个错误
[2018-05-18T17:44:59,658][INFO ][o.e.b.BootstrapChecks    ] [gFOuNlS] bound or publishing to a non-loopback address, enforcing bootstrap checks
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

错误[1]
$ vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
此文件修改后需要重新登录用户，才会生效

错误[2]
$ vim /etc/sysctl.conf
vm.max_map_count=655360
$ sysctl -p
```

### 测试
```
$ curl http://127.0.0.1:9200
{
  "name" : "anZd8rB",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "Z3k4LTtZSqKtGxRQTgFzjQ",
  "version" : {
    "number" : "6.2.4",
    "build_hash" : "ccec39f",
    "build_date" : "2018-04-12T20:37:28.497551Z",
    "build_snapshot" : false,
    "lucene_version" : "7.2.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### Query-DSL
```
http://123.207.236.28:9200/logstash-login_success-2018.07.26/doc/_search?q=userid:1380289829

curl -XGET "http://192.168.1.46:9200/logstash-login_success-*/doc/_search" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "userid": 1380289829
          }
        }
      ],
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "2018-07-15T00:00:00",
              "lte": "2018-07-19T00:00:00"
            }
          }
        }
      ]
    }
  },
  "size": 100,
  "sort": [
    {
      "@timestamp": {
        "order": "desc"
      }
    }
  ]
}'
不设置size，默认返回10个
```
[参考](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/query-dsl.html)
[参考](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/search.html)

## Kibana

### 修改配置
```
$ vim config/kibana.yml
server.port: 5601
server.host: "192.168.226.128"
elasticsearch.url: "http://192.168.226.128:9200"
elasticsearch.username: "elastic"
elasticsearch.password: "changeme"
```

### 运行
```
bin/kibana #前台启动
nohup bin/kibana & #后台启动
```

### 测试
```
http://192.168.226.128:5601
```

## Filebeat

### 修改配置
```
$ vim filebeat.yml
enabled: true                           //打开日志监听
- /home/elk/test_log/*.log              //监听的文件
include_lines: ['.* collect_event=.*']  //匹配的关键字
fields:
    module: region                      //填模块名
    ip: 127.0.0.1                       //填机器内网ip
#output.elasticsearch:                  //注释掉直连elasticsearch
#   hosts: ["localhost:9200"]
output.logstash:
hosts: ["192.168.226.128:5044"]
```

### 运行
```
./filebeat -c filebeat.yml -e -d "publish" #调试运行
nohup ./filebeat -c filebeat.yml & #后台运行
```

## Logstash

### 配置
```
$ vim config/pipeline.yml
增加
input {
    beats {
        port => "5044"
    }
}
filter {
    grok {
        match => {
            "message" => [
                " collect_event=login_success %{NUMBER:userid:int} %{WORD:username} %{IP:clientip} %{NUMBER:dev_type:int} %{NUMBER:proto_ver:int} %{NUMBER:app_ver:int}",
                " collect_event=login %{NUMBER:userid:int} %{WORD:username} %{IP:clientip} %{NUMBER:duration:float}",
                " collect_event=msg %{WORD:username} %{WORD:msg} %{IP:clientip}"
            ]
        }
    }

    kv{}

    geoip {
        source => "clientip"
    }
}
output {
    stdout { codec => rubydebug }

    elasticsearch {
        hosts => [ "192.168.226.128:9200" ]
        index => 'logstash-%{collect_event}-%{+YYYY.MM.dd}'
    }
}

$vim config/jvm.options
-Xms1g改成-Xms4g
-Xmx1g改成-Xmx4g

```

### 运行
```
bin/logstash -f config/pipelines.yml --config.reload.automatic #自动更新配置
nohup bin/logstash -f config/pipelines.yml --config.reload.automatic &
```

### 测试
```
echo " collect_event=login 120001 BBB 122.1.1.25 1.2345" >> test.log
```
### grok

#### 数据类型

#### 多项匹配配置
```
filter {
    grok {
         match => [
            "message" , "%{DATA:hostname}\|%{DATA:tag}\|%{DATA:types}\|%{DATA:uid}\|%{GREEDYDATA:msg}",
            "message" , "%{DATA:hostname}\|%{DATA:tag}\|%{GREEDYDATA:msg}"
         ]
        remove_field => ['type','_id','input_type','tags','message','beat','offset']
    }
}

or

filter {
    grok {
        match => {
            "message"=>[
                "%{DATA:hostname}\|%{DATA:tag}\|%{DATA:types}\|%{DATA:uid}\|%{GREEDYDATA:msg}",
                "%{DATA:hostname}\|%{DATA:tag}\|%{GREEDYDATA:msg}"]
            }
        }
}
```

### 测试
```
$ pwd
/home/elk/test_log

$ echo " collect_event=login 120020 UUU 141.1.1.25 20.2345" >> test.log
```

### 小坑
```
输出elasticsearch，默认预定义的模板必须只有匹配logstash-* 的索引才会应用默认模板，
所以output的index 必须logstash-开头，否则很多logstash自带的数据结构es显示不出，
而且logstash-*中不能有大写字母，否则kibana上找不到，无法创建索引
```


[参考](https://blog.csdn.net/yanggd1987/article/details/50469113)

## Metricbeat
[参考](https://www.elastic.co/guide/en/beats/metricbeat/current/index.html)

## Packetbeat
[参考](https://www.elastic.co/guide/en/beats/packetbeat/current/index.html)

## Heartbeat
[参考](https://www.elastic.co/guide/en/beats/heartbeat/current/index.html)

## Auditbeat
[参考](https://www.elastic.co/guide/en/beats/auditbeat/current/index.html)

## X-Pack
[安装x-pack](https://www.elastic.co/cn/downloads/x-pack)
[算了，还是别搞了，要钱的，只能试用一个月，还不能还原，只能删数据应用重装](https://www.elastic.co/guide/en/x-pack/current/license-management.html)
