# 1. 启动/关闭
```
rabbitmq-server start       #前台启动
rabbitmq-server -detached   #后台启动
rabbitmqctl stop            #关闭
```

# 2. 监控
```
iptables -I INPUT -p tcp --dport 5672 -j ACCEPT #打开端口
iptables -I INPUT -p tcp --dport 15672 -j ACCEPT

rabbitmq-plugins enable rabbitmq_management   #添加监控插件
http://192.168.154.130:15672/                 #访问监控管理
```
