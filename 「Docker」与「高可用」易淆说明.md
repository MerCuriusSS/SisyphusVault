---
tags:
  - Areas/Coder/基础原理
category: 技术
status: 加工
project:
application:
source:
---
### 核心困惑点：
应用为了保持可用性和资源宽裕，不应该将各种服务分开部署吗，即nginx，redis，应用，mysql等，但使用docker后，本质上就是在一台服务器内，分成了多个虚拟机分别部署nginx、redis、应用、mysql等，不仅资源被共享，当服务器挂了，整个应用还会不可用。那优势在哪？而为了保证高可用性所使用的一主一从，那架构上怎么弄？是再根据docker容器的用途分别在同一个服务器再各自创建容器备份，还是在另一个服务器重新使用docker创建容器统一作为备份，如果是这样？那不同服务器内不同docker容器的主从联系又该如何管理？

### 概念区分：

🔴Docker：