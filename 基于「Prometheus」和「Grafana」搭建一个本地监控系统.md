#Resources/coder/核心源码 

### 一、服务暴露可被监控的数据

#### 🔴actuator 组件：开放各类可受监控的端点（info、health、metrics等）

#### 🔴 micrometer 组件：JAVA版的 prometheus SDK，将指标数据转化为prometheus格式。

```xml
<dependency>  
    <groupId>org.springframework.boot</groupId>  
    <artifactId>spring-boot-starter-actuator</artifactId>  
</dependency>
  
<dependency>  
    <groupId>io.micrometer</groupId>  
    <artifactId>micrometer-registry-prometheus</artifactId>  
</dependency>
```

```yml
# application.yml

# 开放监控端点（现成）
management:  
  endpoints:  
    web:  
      exposure:  
        include: health,info,prometheus,metrics
# 将指标数据转为prometheus格式
	metrics:  
	  export:  
	    prometheus:  
	      enabled: true
```

```java
//自定义「业务采集指标」
this.ordersCreatedCounter = Counter.builder("orders.created.total")  
        .description("Total number of orders created")  
        .register(meterRegistry);  
this.createOrderTimer = Timer.builder("orders.create.duration")  
        .description("Order creation duration")  
        .register(meterRegistry);
```

### 二、使用prometheus采集暴露出来的指标

#### 🔴 定义prometheus拉取指标的对象、频率

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'order-service'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8080']

```

#### 🔴 按配置 启动prometheus服务器拉取数据
```bash
prometheus --config.file=D:\workspace\MyProject\monitorProject\prometheus\prometheus.yml
```

#### 🔴promql实现采集验证
- **定义**：查询prometheus采集数据的语法(http_server_requests_seconds_count)
- **语法结构**：`函数( 指标名称{标签匹配} [时间范围] )`
	- 指标名称：`http_requests_total、order_create_total`等
	- 标签匹配：用`{}`包裹，用来筛选维度（job、status、path等）
	- 时间范围：用于增量函数，表示取多久内的数据，如：`rate(http_requests_total[1m])`
### 三、Grafana可视化prometheus采集的数据

#### 🔴