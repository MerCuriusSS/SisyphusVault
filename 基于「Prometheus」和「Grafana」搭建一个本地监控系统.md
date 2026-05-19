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

#### 🔴 启动时配置文件——定义prometheus拉取指标的对象、频率

```yaml
# prometheus.yml
global:  
  scrape_interval: 15s  
  evaluation_interval: 15s  
  
rule_files:  
  - D:\workspace\MyProject\monitorProject\prometheus\alert-rules.yml  

# 采集不同服务的数据
scrape_configs:  
  - job_name: 'order-service'  
    metrics_path: '/actuator/prometheus'  
    static_configs:  
      - targets: ['localhost:8080']  
        labels:  
          application: 'order-service'  
          env: 'local'  
  
  - job_name: 'prometheus'  
    static_configs:  
      - targets: ['localhost:9090']

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

#### 🔴定义prometheus 预警规则
```yaml
# alert-rules.yml
groups:  
  - name: order-service-alerts  
    rules:  
      - alert: HighJvmHeapUsage  
        expr: avg_over_time(jvm_memory_used_bytes{area="heap"}[5m]) / avg_over_time(jvm_memory_max_bytes{area="heap"}[5m]) > 0.8  
        for: 2m  
        labels:  
          severity: warning  
        annotations:  
          summary: "JVM heap usage > 80% (instance {{ $labels.instance }})"  
          description: "Heap usage is {{ $value | humanizePercentage }}. Check for memory leak or increase -Xmx."  
  
      - alert: HighRequestLatency  
        expr: histogram_quantile(0.99, rate(http_server_requests_seconds_bucket[5m])) > 1  
        for: 3m  
        labels:  
          severity: warning  
        annotations:  
          summary: "P99 latency > 1s (instance {{ $labels.instance }})"  
          description: "P99 latency is {{ $value }}s. Check downstream dependencies (DB, cache) or GC pauses."  
  
      - alert: HighErrorRate  
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) / rate(http_server_requests_seconds_count[5m]) > 0.05  
        for: 2m  
        labels:  
          severity: critical  
        annotations:  
          summary: "Error rate > 5% (instance {{ $labels.instance }})"  
          description: "5xx error rate is {{ $value | humanizePercentage }}. Check logs and traces for root cause."
```

### 三、Grafana可视化prometheus采集的数据



#### 🔴下载grafana服务器并本地启动
```cmd
> cd D:\workspace\devTool\grafana\grafana\bin
> grafana.exe
```

#### 🔴 流程：
- 先在 Grafana UI 的 Explore 里试 PromQL，确认查询结果是对的。
- 在 UI 里创建面板。
- 调整图表、单位、标题、布局。
- 导出 dashboard JSON。
- 把 JSON 放进项目仓库。
- 通过 dashboard.yml 自动加载，实现「可观测性配置代码化」

```json
// 仪表盘导出json样式
{  
  "annotations": {  
    "list": [  
      {  
        "name": "告警事件",  
        "datasource": "Prometheus",  
        "expr": "ALERTS{alertstate=\"firing\"}",  
        "step": "60s",  
        "titleFormat": "{{alertname}} — {{summary}}",  
        "textFormat": "{{description}}",  
        "iconColor": "red",  
        "enable": true  
      }  
    ]  
  },  
  "editable": true,  
  "fiscalYearStartMonth": 0,  
  "graphTooltip": 1,  
  "id": null,  
  "links": [],  
  "panels": [  
    {  
      "title": "告警概览",  
      "type": "row",  
      "gridPos": { "h": 1, "w": 24, "x": 0, "y": 0 },  
      "collapsed": false  
    },  
    {  
      "title": "Firing Alerts",  
      "type": "stat",  
      "gridPos": { "h": 3, "w": 4, "x": 0, "y": 1 },  
      "targets": [  
        {  
          "expr": "count(ALERTS{alertstate=\"firing\"})",  
          "legendFormat": ""  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "short", "thresholds": { "mode": "absolute", "steps": [{ "value": 0, "color": "green" }, { "value": 1, "color": "red" }] } }  
      }  
    },  
    {  
      "title": "HighJvmHeapUsage",  
      "type": "stat",  
      "gridPos": { "h": 3, "w": 6, "x": 4, "y": 1 },  
      "targets": [  
        {  
          "expr": "ALERTS{alertname=\"HighJvmHeapUsage\", alertstate=\"firing\"}",  
          "legendFormat": ""  
        }  
      ],  
      "fieldConfig": {  
        "defaults": {  
          "unit": "string",  
          "mappings": [  
            { "type": "value", "value": "1", "text": "FIRING — 堆内存 > 80%" },  
            { "type": "value", "value": "null", "text": "OK" }  
          ],  
          "thresholds": { "mode": "absolute", "steps": [{ "value": 0, "color": "green" }, { "value": 1, "color": "red" }] }  
        }  
      }  
    },  
    {  
      "title": "HighRequestLatency",  
      "type": "stat",  
      "gridPos": { "h": 3, "w": 6, "x": 10, "y": 1 },  
      "targets": [  
        {  
          "expr": "ALERTS{alertname=\"HighRequestLatency\", alertstate=\"firing\"}",  
          "legendFormat": ""  
        }  
      ],  
      "fieldConfig": {  
        "defaults": {  
          "unit": "string",  
          "mappings": [  
            { "type": "value", "value": "1", "text": "FIRING — P99 > 1s" },  
            { "type": "value", "value": "null", "text": "OK" }  
          ],  
          "thresholds": { "mode": "absolute", "steps": [{ "value": 0, "color": "green" }, { "value": 1, "color": "red" }] }  
        }  
      }  
    },  
    {  
      "title": "HighErrorRate",  
      "type": "stat",  
      "gridPos": { "h": 3, "w": 8, "x": 16, "y": 1 },  
      "targets": [  
        {  
          "expr": "ALERTS{alertname=\"HighErrorRate\", alertstate=\"firing\"}",  
          "legendFormat": ""  
        }  
      ],  
      "fieldConfig": {  
        "defaults": {  
          "unit": "string",  
          "mappings": [  
            { "type": "value", "value": "1", "text": "FIRING — 错误率 > 5%" },  
            { "type": "value", "value": "null", "text": "OK" }  
          ],  
          "thresholds": { "mode": "absolute", "steps": [{ "value": 0, "color": "green" }, { "value": 1, "color": "red" }] }  
        }  
      }  
    },  
    {  
      "title": "QPS (requests/sec)",  
      "type": "timeseries",  
      "gridPos": { "h": 8, "w": 12, "x": 0, "y": 4 },  
      "targets": [  
        {  
          "expr": "rate(http_server_requests_seconds_count{application=\"order-service\"}[1m])",  
          "legendFormat": "{{method}} {{uri}}"  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "reqps", "custom": { "lineWidth": 2, "fillOpacity": 15 } }  
      }  
    },  
    {  
      "title": "P50 / P95 / P99 Latency",  
      "type": "timeseries",  
      "gridPos": { "h": 8, "w": 12, "x": 12, "y": 4 },  
      "targets": [  
        {  
          "expr": "histogram_quantile(0.50, rate(http_server_requests_seconds_bucket{application=\"order-service\"}[1m]))",  
          "legendFormat": "P50"  
        },  
        {  
          "expr": "histogram_quantile(0.95, rate(http_server_requests_seconds_bucket{application=\"order-service\"}[1m]))",  
          "legendFormat": "P95"  
        },  
        {  
          "expr": "histogram_quantile(0.99, rate(http_server_requests_seconds_bucket{application=\"order-service\"}[1m]))",  
          "legendFormat": "P99"  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "s", "custom": { "lineWidth": 2 } }  
      }  
    },  
    {  
      "title": "Error Rate (5xx %)",  
      "type": "timeseries",  
      "gridPos": { "h": 8, "w": 8, "x": 0, "y": 12 },  
      "targets": [  
        {  
          "expr": "(sum(rate(http_server_requests_seconds_count{application=\"order-service\", status=~\"5..\"}[1m])) / sum(rate(http_server_requests_seconds_count{application=\"order-service\"}[1m]))) * 100",  
          "legendFormat": "5xx Error %"  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "percent", "min": 0, "max": 100, "custom": { "lineWidth": 2 } }  
      }  
    },  
    {  
      "title": "Total Requests (1m rate)",  
      "type": "stat",  
      "gridPos": { "h": 4, "w": 4, "x": 8, "y": 12 },  
      "targets": [  
        {  
          "expr": "round(sum(rate(http_server_requests_seconds_count{application=\"order-service\"}[1m])))",  
          "legendFormat": "QPS"  
        }  
      ],  
      "fieldConfig": { "defaults": { "unit": "reqps" } }  
    },  
    {  
      "title": "Active Connections",  
      "type": "stat",  
      "gridPos": { "h": 4, "w": 4, "x": 12, "y": 12 },  
      "targets": [  
        {  
          "expr": "tomcat_threads_busy_threads{application=\"order-service\"}",  
          "legendFormat": "Busy"  
        }  
      ],  
      "fieldConfig": { "defaults": { "unit": "short" } }  
    },  
    {  
      "title": "Orders Created (per min)",  
      "type": "timeseries",  
      "gridPos": { "h": 8, "w": 8, "x": 16, "y": 12 },  
      "targets": [  
        {  
          "expr": "rate(orders_created_total[1m]) * 60",  
          "legendFormat": "orders/min"  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "short", "custom": { "lineWidth": 2 } }  
      }  
    },  
    {  
      "title": "Order Creation Duration",  
      "type": "timeseries",  
      "gridPos": { "h": 8, "w": 8, "x": 0, "y": 20 },  
      "targets": [  
        {  
          "expr": "rate(orders_create_duration_seconds_sum[1m]) / rate(orders_create_duration_seconds_count[1m])",  
          "legendFormat": "Avg create time"  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "unit": "s", "custom": { "lineWidth": 2 } }  
      }  
    },  
    {  
      "title": "Top Endpoints by QPS",  
      "type": "table",  
      "gridPos": { "h": 8, "w": 8, "x": 8, "y": 20 },  
      "targets": [  
        {  
          "expr": "topk(5, sum(rate(http_server_requests_seconds_count{application=\"order-service\"}[1m])) by (method, uri))",  
          "format": "table",  
          "instant": true  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "custom": { "align": "center" } },  
        "overrides": [  
          { "matcher": { "id": "byName", "options": "Value" }, "properties": [{ "id": "unit", "value": "reqps" }] }  
        ]  
      }  
    },  
    {  
      "title": "Top Endpoints by P99 Latency",  
      "type": "table",  
      "gridPos": { "h": 8, "w": 8, "x": 16, "y": 20 },  
      "targets": [  
        {  
          "expr": "topk(5, histogram_quantile(0.99, rate(http_server_requests_seconds_bucket{application=\"order-service\"}[5m])) by (method, uri))",  
          "format": "table",  
          "instant": true  
        }  
      ],  
      "fieldConfig": {  
        "defaults": { "custom": { "align": "center" } },  
        "overrides": [  
          { "matcher": { "id": "byName", "options": "Value" }, "properties": [{ "id": "unit", "value": "s" }] }  
        ]  
      }  
    }  
  ],  
  "refresh": "10s",  
  "schemaVersion": 39,  
  "tags": ["business", "order-service"],  
  "time": { "from": "now-15m", "to": "now" },  
  "title": "业务监控大盘",  
  "uid": "business-dashboard",  
  "version": 2  
}
```


#### 🔴 常用grafana 使用已有仪表盘（dashboardID）：
- 仪表市场：https://grafana.com/grafana/dashboards
- JVM：4701
- 数据库连接池：15588
- 主机 / 服务器监控：1860（搭配 node_exporter）
- Redis：12776（搭配 redis_exporter）
- Nginx 监控（网关 / 反向代理）：9614（搭配 nginx_exporter）

#### 🔴可观测性配置代码化（自动加载数据源和大盘）

##### 1）定义数据源配置文件
```yaml

```