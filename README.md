## **分布式复制系统**

### 背景

non-volatile memory使得现有计算机架构的计算和存储界限发生明显的变化，non-volatile memory拥有DRAM一样的访存性能，同时具备掉电后数据不丢失的特性。随着具有DIMM接口的NVM产业化，构建实际可用的基于non-volatile main memory的共享内存系统刻不容缓。

### 初步设计

与Farm设计类似，以2GB大小的region为基本单位管理共享内存系统，同时以region为基本单元提供分布式主-存复制功能。整个系统中存在一个全局的ConfigManage进行region（包括primary和backup）映射表的构建，并采取一定的负载均衡措施，每个server有一个RegionAlloc进行本地region分配，并向ConfigManage申请region全局地址。为了保证region映射表的高可用性，ConfigManage将最新的region映射表存储到高可用的存储服务（例如，zookeeper、LogCabin或者etcd）中，同时在内存中存储支持并发访问的映射表copy。

### 挑战

region分配过程中的一致性保证：采取两阶段的region分配方式

高可用性是整个系统的关键，ConfigManage定期向每个server发送心跳。

Crash consistency：（两阶段方式）

1 ConfigManage crash consistency：timeout的server从高可用协调服务中读取最新的region映射表信息，递增config_id，成为最新的ConfigManage，并向cluster中的其他server发送心跳。

2 server crash consistency：

​	（1）primary region server crash：ConfigManage选取最新的primary region，更新region映射表，向受影响的server发送更新后的region映射表信息。

​	（2）backup region server crash：更新region映射表，向受影响的server发送更新后的region映射表信息。



### 已经完成的部分

（1）region分配

（2）region映射表的本地cache

（3）可并发访问的region映射表



### 测试

Fixed me.

![1557038103676](C:\Users\清华\AppData\Roaming\Typora\typora-user-images\1557038103676.png)

### 待完成部分

（1）crash consistency，包括ConfigManage crash consistency和server crash consistency

