# 可观测性

可观测性（Observability）是指从系统的外部输出推断其内部状态的能力。与传统监控只关注"已知的未知"不同，可观测性还帮助我们发现"未知的未知"。可观测性的三大支柱是**日志**（Logs）、**指标**（Metrics）和**链路追踪**（Traces），再辅以**告警体系**形成完整的闭环。

## 日志系统

日志是最基础的可观测性手段，记录系统运行过程中的离散事件。

### 日志级别

| 级别 | 用途 | 示例 |
|---|---|---|
| `DEBUG` | 开发调试信息，生产环境通常关闭 | 函数入参、中间变量 |
| `INFO` | 关键业务流程的正常节点 | 用户登录成功、订单创建 |
| `WARN` | 潜在问题，不影响核心功能 | 接口响应缓慢、重试成功 |
| `ERROR` | 业务异常，需要关注但系统仍可运行 | 支付回调失败、第三方接口超时 |
| `FATAL` | 系统级致命错误，通常伴随进程退出 | 数据库连接池耗尽、OOM |

!!! tip "级别使用原则"

    - 生产环境日志级别设为 `INFO`，必要时动态调整为 `DEBUG`（通过配置中心热更新）
    - 避免在循环中打 `INFO` 日志，防止日志洪峰
    - `ERROR` 日志必须附带完整的上下文信息（请求 ID、用户 ID、错误堆栈等）

### 结构化日志

相比传统的纯文本日志，**结构化日志**以 JSON 等格式输出，便于采集、检索和分析：

```json
{
  "timestamp": "2026-08-01T09:30:00.123Z",
  "level": "ERROR",
  "service": "order-service",
  "trace_id": "abc123def456",
  "user_id": 10086,
  "method": "POST",
  "path": "/api/v1/orders",
  "duration_ms": 1523,
  "error": "payment gateway timeout",
  "stack_trace": "..."
}
```

**关键字段设计**：

- **trace_id** — 贯穿整个请求链路的唯一标识，用于关联日志与链路追踪
- **span_id** — 当前操作在链路中的节点标识
- **service** — 服务名称，区分不同微服务的日志
- **duration_ms** — 操作耗时，用于识别慢请求

### 日志采集架构

典型的日志架构采用 **ELK / EFK** 方案：

```
应用 → Filebeat / Fluentd（采集） → Kafka（缓冲） → Logstash / Flink（清洗） → Elasticsearch（存储） → Kibana / Grafana（查询）
```

**核心要点**：

- **引入 Kafka 做缓冲**：解耦采集端与存储端，防止日志突增时压垮 Elasticsearch
- **日志分级存储**：ERROR 日志保留 90 天，INFO 日志保留 30 天，DEBUG 日志保留 7 天
- **索引策略**：按天或按服务创建索引（如 `order-service-2026.08.01`），便于生命周期管理

## 指标监控

指标是可聚合的时序数值数据，用于量化系统的运行状态。

### 四种指标类型

以 Prometheus 的数据模型为例：

| 类型 | 说明 | 典型场景 |
|---|---|---|
| **Counter** | 单调递增计数器，只增不减 | 请求总数、错误总数、消息处理量 |
| **Gauge** | 瞬时值，可增可减 | CPU 使用率、内存占用、连接池大小、队列深度 |
| **Histogram** | 将观测值分入预定义的桶（bucket），统计分布 | 请求延迟分位数（P50/P99） |
| **Summary** | 类似 Histogram，但在客户端计算分位数 | 精确分位数（不可聚合，慎用） |

### 监控方法论

**RED 方法**（面向服务）：

- **R**ate — 每秒请求数（QPS/RPS）
- **E**rror — 每秒错误数 / 错误率
- **D**uration — 请求耗时分布（P50 / P95 / P99）

**USE 方法**（面向资源）：

- **U**tilization — 资源使用率（CPU、内存、磁盘）
- **S**aturation — 资源饱和度（等待队列长度）
- **E**rrors — 资源错误计数

!!! note "实践建议"

    对**服务层**使用 RED 方法建立 Dashboard，对**基础设施层**使用 USE 方法建立 Dashboard。两者结合可以快速定位问题是出在应用层还是资源层。

### Prometheus 架构

```
应用（暴露 /metrics 端点） ← Prometheus Server（定时 Pull 拉取） → Alertmanager（告警路由）
                                        ↓
                              Grafana（可视化 Dashboard）
```

**核心组件**：

- **Exporter** — 将第三方系统的指标转换为 Prometheus 格式（如 MySQL Exporter、Node Exporter）
- **PromQL** — Prometheus 的查询语言，支持聚合、函数、向量运算
- **Recording Rules** — 预计算常用查询，降低实时查询开销
- **Remote Write** — 将指标数据写入远端长期存储（如 Thanos、VictoriaMetrics）

**常用 PromQL 示例**：

```promql
# 5 分钟内的平均 QPS
rate(http_requests_total[5m])

# 请求延迟的 P99
histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m]))

# 错误率
rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m])
```

## 链路追踪

链路追踪记录一个请求在分布式系统中流经各服务的完整路径，用于定位跨服务的延迟瓶颈和错误传播链。

### 核心概念

- **Trace** — 一次完整请求的调用链，由若干 Span 组成，共享同一个 Trace ID
- **Span** — 调用链中的一个操作单元（如一次 RPC 调用、一次数据库查询），记录操作名、耗时、状态码等
- **Parent Span / Child Span** — Span 之间的父子关系，形成树状结构

```
Trace: abc123
├── Span A: API Gateway (12ms)
│   ├── Span B: Order Service (8ms)
│   │   ├── Span C: MySQL Query (3ms)
│   │   └── Span D: Redis Cache (1ms)
│   └── Span E: Payment Service (45ms)  ← 瓶颈
│       └── Span F: External Bank API (40ms)
```

### 传播机制

Trace 上下文通过请求头在服务间传播。以 W3C Trace Context 标准为例：

```
traceparent: 00-<trace-id>-<span-id>-<trace-flags>
```

常见的传播方式：

- **HTTP** — 通过请求头（`traceparent`、`b3`）
- **gRPC** — 通过 Metadata
- **消息队列** — 通过消息属性 / Header

### OpenTelemetry

OpenTelemetry（OTel）是当前可观测性领域的事实标准，统一了 Traces、Metrics、Logs 的采集协议和 SDK。

```
应用（OTel SDK 埋点） → OTel Collector（采集 & 处理） → 后端存储（Jaeger / Tempo / Zipkin）
                                                              ↓
                                                    Grafana（可视化）
```

**OTel Collector 的作用**：

- **解耦**：应用只需对接 Collector，后端存储可随时更换
- **处理管线**：支持过滤、采样、数据转换、批量发送
- **多协议支持**：同时接收 OTLP、Jaeger、Zipkin 等格式

### 采样策略

全量采集在高流量场景下成本过高，需要合理的采样策略：

| 策略 | 说明 | 适用场景 |
|---|---|---|
| 头部采样（Head-based） | 在请求入口决定是否采样 | 实现简单、开销低 |
| 尾部采样（Tail-based） | 在请求结束后根据结果决定是否保留 | 保证异常请求 100% 采集 |
| 自适应采样 | 根据流量动态调整采样率 | 高流量服务 |

!!! tip "采样最佳实践"

    - 错误请求和慢请求**始终采集**，不受采样率影响
    - 正常请求按比例采样（如 1%），控制存储成本
    - 使用尾部采样时需在 OTel Collector 中配置，注意内存开销

## 告警体系

可观测性的最终目标是在问题影响用户之前发现并处理它。告警体系是连接"发现问题"与"解决问题"的桥梁。

### 告警分级

| 级别 | 含义 | 响应时效 | 通知方式 |
|---|---|---|---|
| **P0 — 致命** | 核心业务完全不可用 | 5 分钟内响应 | 电话 + 短信 + IM |
| **P1 — 严重** | 核心功能降级或部分不可用 | 15 分钟内响应 | 短信 + IM |
| **P2 — 警告** | 非核心功能异常或性能下降 | 1 小时内响应 | IM |
| **P3 — 提示** | 潜在风险，暂不影响业务 | 工作时间处理 | IM / 邮件 |

### 告警收敛

原始告警往往会产生"告警风暴"，需要通过收敛策略减少噪音：

- **分组（Grouping）** — 将相同类型的告警合并为一条（如同一服务的多个实例同时报错）
- **抑制（Inhibition）** — 高优先级告警触发后，自动抑制由它引发的低优先级告警（如数据库宕机时，抑制所有依赖它的服务的超时告警）
- **静默（Silencing）** — 在维护窗口期临时屏蔽特定告警

### On-Call 机制

- **轮值排班** — 按周 / 按天轮值，确保任何时刻都有人响应
- **升级策略** — 告警 N 分钟内未确认，自动升级通知上级或备份人员
- **事后复盘（Postmortem）** — 每次重大故障后进行根因分析，输出改进措施，避免同类问题再次发生

### 告警规则设计原则

!!! warning "避免常见陷阱"

    - **避免阈值过敏**：使用持续时间窗口（如"连续 5 分钟 P99 > 500ms"），而非瞬时值触发
    - **避免告警疲劳**：每条告警都应有明确的 Action Item，无需人工介入的告警应转为自动化处理
    - **区分症状和原因**：优先对用户可感知的**症状**（如错误率上升）告警，而非底层**原因**（如某个 Pod 重启）

