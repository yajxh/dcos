## 监控管理

DC/OS可以快速构建大规模（微）服务集群，这个愿景是有希望的，但随着服务数量的增长，它也引入了复杂性。集群中服务的数量动辄成百上千，这些服务是否健康运行，出现问题是否能够立即告警，因此，对这些服务的全天候监控变得极为必要。

对于一个规模集群的DC/OS数据中心来说，监控涉及多个层面：
- 硬件网络
- 主机操作系统
- DC/OS系统及核心服务
- 运行的在集群上的容器
- 容器中运行的应用服务


### 相关工具

从数据流动的线路来看，监控管理通常包括信息采集、数据汇集存储，分析展示及告警等阶段。根据监控层面的不同，上述各个阶段涉及的工具也各不相同。常用工具如下：

#### 信息采集

- **cAdvisor**
  
  cAdvisor（Container Advisor）让容器用户可以及时掌握正在运行的容器的资源使用情况和性能态势。它是一个运行的守护进程，能够收集，聚合，处理和导出运行容器相关的信息。

- **Logstash/Filebeat**


#### 数据汇集存储

- **Graphite**

- **ElasticSearch**

- **Prometheus**

- **InfluxDB**

#### 展示及告警

- **Grafana**

- **Kibana**

- **PromeDash**

对于应用服务层面的日志管理及分析方案，请参考[日志管理](/dcos-admin-logging.md)章节。