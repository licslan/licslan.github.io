---
layout:     post
title:      tec kafka zookeeper storm/Jstorm
subtitle:   kafka zookeeper storm/Jstorm
date:       2018-12-30
author:     LICSLAN
header-img: img/zjm.jpg
catalog: true
tags:
     - tec
     - build a kafka_zookeeper_jstorm
---

今天元旦假期第一天  在家没有事情  就大致画了下图 关于jstorm  kafka  zk 集群的图<br>
实验需求： springboot-kafka 生产数据发送kafka集群  springboot-realtime (jstorm应用)消费kafka集群数据并计算处理落地或是可视化展示都可以<br>
三台虚拟机 192.168.108.128  192.168.108.130 192.168.108.131 <br>  
gitbash/xshell/vm15/IDEA/jdk8/zookeeper/kafka/jstorm/springboot/cnetos7<br>
搭建Linux环境就不这里概述了  2G 内存/每台<br>
linux3台机子的ssh免密登陆也请度娘Google都行<br>
很多资料也请搭建访问官网学习和搭建<br>

为了实验成功呢  所以建议大家先干掉Linux的防火墙机制  免得自己去开放各种端口...  这里关于防火墙的内容大家可以度娘哈<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/jstorm.png)<br>

咱们来看看zookeeper 集群搭建和kafka搭建效果吧 

![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/zk_kafka_cluster.png)
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/zk_cluster.jpg)<br>

OK  我们来看测试测试效果吧  我们启动kafka集群生产者和消费者  同时搭建了springboot-kafka 发送消费的demo<br>
并使用了postman本地测试本地运行的kafka  java程序  来简单生产数据   在Linux环境的服务器也受到了来至本地的Java程序发送的消息<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/kafka-product-consumer.jpg)<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/idea-kafka-test.jpg)<br>
![](https://raw.githubusercontent.com/licslan/licslan.github.io/master/img/postman-send-mes.jpg)<br>



待会我会将整理好的如何搭建zookeeper 和 kafka 过程写下来...<br>

后面也会写着关于jstorm/storm集群的搭建效果和过程 <br>

再者后面也会建立两个工程关于kafka消息的生产和消费的springboot-kafka项目和jstorm实时计算的项目springboot-realtime<br>

再者后面也写如何mysql/redis/kafka/jstorm/spark/flink调优.... 同时也会带上关于spark/flink机器学习如何利用的小demo  尽请期待  







