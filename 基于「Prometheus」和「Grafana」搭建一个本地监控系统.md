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
- promql：查询


### 三、