### Eureka、Consul、Zookeeper 三个注册中心的异同：

###### 三者都支持Spring Cloud 。

######  除了Consul是使用Go语言写的，其他两个都是使用的Java

###### 除了Eureka是AP ，其他两个都是CP

###### 服务健康检查在Eureka 上是可配支持的，其他两个都是支持

###### 对外暴露接口的话，Eureka 是HTTP，Consul 是HTTP/DNS，Zookeeper是客户端