#1.librdkafka例子使用
```
cd /librdkafka/example
./rdkafka_example_cpp -P -b 192.168.18.76:9092 -p 0 -t LULU_TEST                    #生产者
./rdkafka_example_cpp -C -b 192.168.18.76:9092 -p 0 -t LULU_TEST                    #消费者
./rdkafka_consumer_example_cpp -g GROUP -b 192.168.18.76:9092 LULU_TEST             #消费组
./rdkafka_example_cpp -L -b 192.168.18.76:9092 | grep LULU_TEST                     #查看主题
./rdkafka_consumer_example -D -b 192.168.18.76:9092 -g OperationalDecisionPlatform  #查看某消费组有几个程序以及IP等
```
