---
layout:     post
title:      简单使用 docker on centos7
subtitle:   How to simply use docker
date:       2018-08-26
author:     licslan
header-img: img/post-bg-ios9-web.jpg
catalog: 	 true
tags:
    - docker
---  

```
so how to use the docker and what`s the difference between the docker and vm tec? 

ok if we want to get this down first we just go to https://www.docker.com/ and see what`s going on 

i just qoute words from 'Docker Containerization Unlocks the Potential for Dev and Ops

Freedom of choice, agile operations and integrated security for legacy and cloud-native applications'


Docker的应用场景

Web 应用的自动化打包和发布。

自动化测试和持续集成、发布。

在服务型环境中部署和调整数据库或其他的后台应用。

从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建自己的PaaS环境。

Docker 的优点

1、简化程序：
Docker 让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，便可以实现虚拟化。Docker改变了虚拟化的方式，使开发者可以直接将自己的成果放入Docker中进行管理。方便快捷已经是 Docker的最大优势，过去需要用数天乃至数周的	任务，在Docker容器的处理下，只需要数秒就能完成。

2、避免选择恐惧症：
如果你有选择恐惧症，还是资深患者。Docker 帮你	打包你的纠结！比如 Docker 镜像；Docker 镜像中包含了运行环境和配置，所以 Docker 可以简化部署多种应用实例工作。比如 Web 应用、后台应用、数据库应用、大数据应用比如 Hadoop 集群、消息队列等等都可以打包成一个镜像部署。

3、节省开支：
一方面，云计算时代到来，使开发者不必为了追求效果而配置高额的硬件，Docker 改变了高性能必然高价格的思维定势。Docker 与云的结合，让云空间得到更充分的利用。不仅解决了硬件管理的问题，也改变了虚拟化的方式。

A Modern Platform for All Applications

Docker unlocks the potential of your organization by giving developers and IT the freedom to build,

manage and secure business-critical applications without the fear of technology or infrastructure lock-in.

By combining its industry-leading container engine technology, an enterprise-grade container platform and 

world-class services, Docker enables you to bring traditional and cloud native applications built on Window,

Linux and mainframe into an automated and secure supply chain, advancing dev to ops collaboration and reducing time to value.

Because Docker increases productivity and reduces the time it takes to bring applications to market, 

you now have the resources needed to invest in key digitization projects that cut across the entire value chain, 

such as application modernization, cloud migration and server consolidation. With Docker, you have the solution 

that helps you manage the diverse applications, clouds and infrastructure you have today while providing your 

business a path forward to future applications.

ok let`s get the point 

1.you should install vm and centos7 

2.run commond  yum install docker -y

3.then systemctl enable docker

4. systemctl start docker

5. next you can run commod  docker --help

6. just study those commod  just like docker pull [images name] docker ps -a  

    docker images  docker run --  docker rm/rmi  docker load and so on 

ok for now you konw how to use docker  and you will get deep in it and just go to https://www.docker.com 

```

