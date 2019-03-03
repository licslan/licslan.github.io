---
layout:     post
title:      big data real time compute
subtitle:   spark streaming sql   
date:       2019-03-03
author:     licslan
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - tec
---  


--------------------------------------------------------------


			spark大数据实时计算平台搭建总结  

										Author by Weilin Huang
--------------------------------------------------------------

本总结是经过学习Michael__PK老师的课程后而总结


搭建环境要求

由于环境中的依赖的中间件和软件较多  所以将虚拟机内存设置到8g 4 core

硬件：小米 i7 8代  8g  4 core   VM14  centOs7  NAT模式

开发工具：IDEA  postman  SublimeText3 

依赖中间件：zookeeper-3.10  maven-3.6  jdk8  hbase-1.59  flume1.9  kafka-2.0.0  spark-2.3  hadoop-3.2  MySql

涉及到编程语言：python2.7  scala-2.11  Java

使用框架：springboot echarts jQuery

flume：日志收集  注意flume的2种方式 push  pull  如果与spark streaming对接的话
zookeeper：消息协调器 注册中心
maven：打包工具
hbase: nosql 数据库
Kafka：消息中间件
Hadoop：离线计算框架
MySQL：数据库
spark：集离线计算(spark core)，批量计算(spark core)，实时流式计算（Spark Streaming），机器学习（MLlib），SQL查询(spark SQL)，图像计算(GraphX)


设计思路：
	1.使用python脚本编写日志生产器  生产一定规则的数据 比如生产100万条数据  大概110兆数据  MAC 16g  大概55秒钟可以生成  可以配置Linux 定时任务

	2.对生产的数据，上传到服务器指定的地方，并且整合flume框架, flume  主要是写配置文件并启动  主要配置avro-source  channel  sink三者配置关系

	3.将对接好flume的数据对接到Kafka消息队列中

	4.将对接好的Kafka消息队列对接到Spark Streaming上

	5.对接Spark Streaming 后将数据经过计算（数据清洗）存到Hbase  (对于Hbase的rowkey设计一定要结合实际业务场景设计)/或是存到RMDBS

	6.搭建spring boot可视化数据，数据经由hbase取出来在页面上显示，其实这个时候一切顺利的话是可以实时动态显示数据的



	实时流式计算应用场景：

		说说本人之前的一家公司业务场景，对于用户历史数据收集，并整理入库，并使用随机森林算法对该数据进行训练，生成模型数据，
		并对后面实时录入的客户数据信息进行决策判断，哪些用户是具备偿还能力的，并放款给改客户，减少坏账的可能

		电商行业中给用户推荐可能喜欢的商品，提高订单转化率

		电信行业，对于用户的流量包数据进行分析，推荐适合的流量包，并实时提醒用户，流量用了多少

		对于一些敏感的传感器数据进行实时检测，在科学实验中也有运用

		电商公司一些反欺诈手段检测，减少资产损失 例如前端时间PDD的失误，风控部门建设也离不开该技术手段来检测

		城市大脑对车流量信息实时预测

		城市大型公园活动/旅游景点的人流量实时检测

		舆情监测 对于负面消息 快速识别检测


	所以对于建设大数据实时流式计算平台对于一家上了规模的公司是很有必要的
	当然实时流式计算框架也有很多，比如常用的Storm  Kafka  Flink  Spark  他们之间其实都是不错的技术手段
	对于Spark Flink  式比较不多的一站式解决手段  	

	当然人工智能 机器学习日前发展也非常快，很多算法是上个世纪就有了，目前发展这么快主要是要由于移动互联网快速发展，产生了很多大量的数据，
	同时随着网速带宽不断成熟，4G 5G 不断普及，给我们带来了很多现在可以做的事情。现在互联网市场公司大概分类这几类，
	I BAT一线互联网公司  有技术也有数据
	II 有技术没有数据 
	III 有数据没有技术（技术不成熟）
	但是需要将这些数据整合起来，BI 商业智能  分析整合数据  ，产生新的价值。


