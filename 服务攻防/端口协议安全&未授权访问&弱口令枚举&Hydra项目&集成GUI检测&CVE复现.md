[[常见服务端口]]

## 未授权访问

项目 https://github.com/xk11z/unauthorized
参考 https://mp.weixin.qq.com/s/Qwemde7CmH_WyFKSlcN41Q

1 、FTP 未授权访问（21）
2 、LDAP 未授权访问（389）
3 、Rsync 未授权访问（873）
4 、ZooKeeper未授权访问（2181）
5 、Docker 未授权访问（2375）
6 、Docker Registry未授权（5000）
7 、Kibana 未授权访问（5601）
8 、VNC 未授权访问（5900、5901）
9 、CouchDB 未授权访问（5984）
10 、Apache Spark 未授权访问（6066、8081、8082）
11 、Redis 未授权访问（6379）
12 、Weblogic 未授权访问（7001）
13 、HadoopYARN 未授权访问（8088）
14 、JBoss 未授权访问（8080）
15 、Jenkins 未授权访问（8080）
16 、Kubernetes Api Server 未授权（8080、10250）
17 、Active MQ 未授权访问（8161）
18 、Jupyter Notebook 未授权访问（8888）
19 、Elasticsearch 未授权访问（9200、9300）
20 、Zabbix 未授权访问（10051）
21 、Memcached 未授权访问（11211）
22 、RabbitMQ 未授权访问（15672、15692、25672）
23 、MongoDB 未授权访问（27017）
24 、NFS 未授权访问（2049、20048）
25 、Dubbo 未授权访问（28096）
26 、Druid 未授权访问
27 、Solr 未授权访问
28 、SpringBoot Actuator 未授权访问
29 、SwaggerUI未授权访问漏洞
30 、Harbor未授权添加管理员漏洞
31 、Windows ipc共享未授权访问漏洞
32 、宝塔phpmyadmin未授权访问
33 、WordPress未授权访问漏洞
34 、Atlassian Crowd 未授权访问
35 、PHP-FPM Fastcgi未授权访问漏洞
36 、uWSGI未授权访问漏洞
37 、Kong未授权访问漏洞
38 、ThinkAdminV6未授权访问漏洞

## 弱口令爆破

https://github.com/BBD-YZZ/week-passwd
https://github.com/vanhauser-thc/thc-hydra

```
hydra -l administrator -P top6000.txt 192.168.1.1 rdp
```

https://github.com/maaaaz/thc-hydra-windows