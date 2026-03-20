# 微服务架构项目

## 架构概览
- 微服务架构，各服务独立部署、独立数据库
- 服务间通信：同步（REST/gRPC）+ 异步（消息队列）
- 容器化部署：Docker + Kubernetes
- 基础设施：服务注册发现、配置中心、API 网关

## 项目结构（典型 Monorepo）
```
services/           # 各微服务（user-service, order-service 等）
shared/
  proto/            # gRPC Proto 定义
  common-lib/       # 共享工具库
infra/
  docker-compose.yml / k8s/ / helm/  # 部署配置
```

## 服务间通信
- 同步调用：REST（简单场景）或 gRPC（高性能/强类型场景）
- 异步消息：RabbitMQ / Kafka，用于事件驱动和解耦
- 命名规范：事件名用过去时 `order.created`、`payment.completed`
- 调用链路中做好超时和重试配置，设置合理的 timeout 和 retry 策略
- 使用断路器（Circuit Breaker）防止级联故障

## 服务拆分原则
- 按业务域划分（DDD 限界上下文），而非技术层
- 每个服务拥有自己的数据库，禁止直接访问其他服务的数据库
- 跨服务数据一致性使用 Saga 模式或最终一致性
- 避免分布式事务（2PC），优先选择补偿机制

## Docker + Kubernetes
- 每个服务一个 Dockerfile，使用多阶段构建减小镜像
- K8s 资源：Deployment + Service + ConfigMap/Secret + Ingress
- 健康检查：配置 `livenessProbe` 和 `readinessProbe`
- 水平扩缩容：HPA 基于 CPU/内存或自定义指标

## 可观测性
- 分布式追踪：OpenTelemetry 采集，Jaeger/Zipkin 展示
- 日志：结构化 JSON 日志，包含 traceId，统一收集到 ELK/Loki
- 监控：Prometheus 采集指标 + Grafana 面板
- 每个请求必须携带和传播 traceId/spanId

## 本地开发
```bash
docker-compose up -d              # 启动基础设施（数据库、MQ、Redis）
docker-compose up user-service    # 启动单个服务
docker-compose logs -f order-service  # 查看服务日志
```

## API 网关
- 统一入口处理认证鉴权、限流、路由转发（Kong / APISIX / Envoy）

## 常见陷阱
- 服务拆太细导致运维成本剧增，初期宜粗不宜细
- 分布式环境下不要依赖本地文件系统存储状态
- 数据库 migration 需向后兼容（先加后删，不直接改列名）
- 服务间调用失败时要有降级方案，不能让整条链路阻塞
- 注意幂等性设计：MQ 消费者和 API 接口都需要支持重复调用
