# Microservices Architecture Project

## Architecture Overview
- Microservices architecture with independently deployed services and separate databases
- Inter-service communication: synchronous (REST/gRPC) + asynchronous (message queues)
- Containerized deployment: Docker + Kubernetes
- Infrastructure: service discovery, configuration center, API gateway

## Project Structure (Typical Monorepo)
```
services/           # Individual microservices (user-service, order-service, etc.)
shared/
  proto/            # gRPC Proto definitions
  common-lib/       # Shared utility library
infra/
  docker-compose.yml / k8s/ / helm/  # Deployment configurations
```

## Inter-Service Communication
- Synchronous: REST (simple scenarios) or gRPC (high-performance / strongly-typed scenarios)
- Asynchronous: RabbitMQ / Kafka for event-driven decoupling
- Naming convention: use past tense for event names — `order.created`, `payment.completed`
- Configure proper timeouts and retry policies for all service-to-service calls
- Use a Circuit Breaker pattern to prevent cascading failures

## Service Decomposition Principles
- Split by business domain (DDD bounded contexts), not by technical layers
- Each service owns its own database — never access another service's database directly
- Use the Saga pattern or eventual consistency for cross-service data consistency
- Avoid distributed transactions (2PC); prefer compensation mechanisms

## Docker + Kubernetes
- One Dockerfile per service; use multi-stage builds to minimize image size
- K8s resources: Deployment + Service + ConfigMap/Secret + Ingress
- Health checks: configure both `livenessProbe` and `readinessProbe`
- Horizontal scaling: HPA based on CPU/memory or custom metrics

## Observability
- Distributed tracing: collect with OpenTelemetry, visualize with Jaeger/Zipkin
- Logging: structured JSON logs with traceId, aggregated into ELK/Loki
- Monitoring: Prometheus for metrics collection + Grafana dashboards
- Every request must carry and propagate traceId/spanId

## Local Development
```bash
docker-compose up -d              # Start infrastructure (database, MQ, Redis)
docker-compose up user-service    # Start a single service
docker-compose logs -f order-service  # Tail logs for a service
```

## API Gateway
- Centralized entry point for authentication, rate limiting, and route forwarding (Kong / APISIX / Envoy)

## Common Pitfalls
- Over-splitting services dramatically increases operational overhead — start coarse, refine later
- Do not rely on the local filesystem for state storage in a distributed environment
- Database migrations must be backward-compatible (add first, remove later; never rename columns directly)
- Always have a fallback/degradation plan when inter-service calls fail — never let the entire chain block
- Design for idempotency: both MQ consumers and API endpoints must handle duplicate invocations gracefully
