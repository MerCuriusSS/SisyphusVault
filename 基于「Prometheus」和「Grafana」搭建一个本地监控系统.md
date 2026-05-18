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
# 开放监控端点
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
### 二、使用prometheus采集暴露出来的指标
