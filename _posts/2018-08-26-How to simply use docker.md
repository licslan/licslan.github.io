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

