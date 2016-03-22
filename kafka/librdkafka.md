#1.librdkafka例子使用
```
cd /librdkafka/example
./rdkafka_example_cpp -P -b 192.168.18.76:9092 -p 0 -t LULU_TEST                    #生产者
./rdkafka_example_cpp -C -b 192.168.18.76:9092 -p 0 -t LULU_TEST                    #消费者
./rdkafka_consumer_example_cpp -g GROUP_LULU -b 192.168.18.76:9092 LULU_TEST        #消费组
./rdkafka_example_cpp -L -b 192.168.18.76:9092 | grep LULU_TEST                     #查看主题
./rdkafka_consumer_example -D -b 192.168.18.76:9092 -g GROUP_LULU                   #查看某消费组有几个程序连接以及IP等
./rdkafka_consumer_example -O -b 192.168.18.76:9092 -g GROUP_LULU LULU_TEST:0       #查看某消费组的已经提交到[主题:分区]的offset
```

#2.配置项message.max.bytes和fetch.message.max.bytes
```
注意fetch.message.max.bytes要大于等于message.max.bytes,
否则一个message.max.bytes超过fetch.message.max.bytes消息会导致消费者和消费组堵住这个主题，
永远收不到消息
```
