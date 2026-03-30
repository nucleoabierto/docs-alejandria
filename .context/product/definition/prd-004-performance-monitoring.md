---
id: PROD-PRD-0004
area: "product"
type: "prd"
title: "PRD-004: Performance & Monitoring"
author: "product-team"
status: "planned"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["PROD-PRD-0001", "PROD-PRD-0002", "PROD-PRD-0003", "PROD-STRAT-0008"]
---

# Product Requirements Document - PRD-004: Performance & Monitoring

**Version**: 1.0  
**Date**: 2026-03-30  
**Status**: Planned  
**Owner**: DevOps Team  
**Stakeholders**: Engineering, DevOps, Product, Customer Success  

## 1. Executive Summary

### Product Vision
Sistema de monitoreo y optimización de rendimiento que garantice latencia sub-100ms para búsquedas, escalabilidad horizontal para 500+ organizaciones, y visibilidad completa del estado del sistema mediante métricas, alertas y dashboards en tiempo real.

### Problem Statement
Sin monitoreo proactivo y optimización de rendimiento, el sistema sufrirá degradación a medida que escala, causando mala experiencia de usuario, dificultades de diagnóstico y incapacidad para cumplir SLAs críticos.

### Success Metrics (6 semanas)
- **Search latency**: < 100ms (p95) bajo carga
- **System uptime**: > 99.9% durante operaciones normales
- **Alert coverage**: 100% de incidentes críticos detectados
- **Performance regression**: < 5% degradación mes a mes

---

## 2. Target Users & Personas

### Primary Personas

#### 1. **DevOps Engineer**
- **Contexto**: Gestiona infraestructura, monitorea salud del sistema, optimiza performance
- **Pain Points**: "¿Por qué las búsquedas están lentas?", "¿Cómo sé si el sistema está sano?", "¿Dónde está el bottleneck?"
- **Goals**: Visibilidad completa, alertas proactivas, diagnóstico rápido
- **Usage Frequency**: Diaria en operaciones, continua durante incidentes

#### 2. **Backend Developer**
- **Contexto**: Optimiza queries, identifica performance issues, valida cambios
- **Pain Points**: "Mi cambio afectó el rendimiento?", "¿Qué partes del sistema son lentas?", "¿Cómo valido optimización?"
- **Goals**: Métricas de código, profiling fácil, regression detection
- **Usage Frequency**: Semanal en desarrollo, diaria en debugging

#### 3. **Engineering Manager**
- **Contexto**: Planea capacidad, revisa SLAs, gestiona recursos
- **Pain Points**: "¿Necesitamos más recursos?", "¿Estamos cumpliendo nuestros SLAs?", "¿Cuál es el costo de performance?"
- **Goals**: Capacity planning, SLA compliance, cost optimization
- **Usage Frequency**: Semanal en planning, diaria en revisión de métricas

#### 4. **Site Reliability Engineer (SRE)**
- **Contexto**: Mantiene disponibilidad, responde a incidentes, mejora resiliencia
- **Pain Points**: "¿Cómo prevenimos futuros incidentes?", "¿Cuál es el impacto de este cambio?", "¿Cómo medimos resiliencia?"
- **Goals**: Incident prevention, impact measurement, reliability improvement
- **Usage Frequency**: Continua durante incidentes, diaria en operaciones

### Secondary Personas

#### 5. **Product Manager**
- **Usage**: Revisa performance impact en用户体验, mide KPIs de sistema
- **Frequency**: Semanal

#### 6. **Customer Success**
- **Usage**: Monitorea salud de clientes, responde a quejas de performance
- **Frequency**: Diaria

---

## 3. Core Features & Requirements

### 3.1 Performance Monitoring

#### 3.1.1 Application Performance Monitoring (APM)

**User Story**: Como DevOps engineer, quiero visibilidad completa del rendimiento de la aplicación para identificar y resolver problemas rápidamente.

**Requirements**:
- ✅ **MUST**: Distributed tracing con OpenTelemetry
- ✅ **MUST**: Request latency tracking (p50, p95, p99)
- ✅ **MUST**: Database query performance monitoring
- ✅ **MUST**: External service call tracking
- ✅ **SHOULD**: Code-level profiling para hotspots
- ✅ **SHOULD**: Memory usage y GC metrics
- ✅ **SHOULD**: CPU utilization por componente

**Acceptance Criteria**:
- Todas las requests rastreadas con correlation IDs
- Database queries > 100ms identificadas automáticamente
- External service calls > 500ms alertadas
- Memory leaks detectados con < 24h de antelación

#### 3.1.2 Search Performance Monitoring

**User Story**: Como backend developer, quiero monitorear específicamente el rendimiento de búsqueda para garantizar SLAs.

**Requirements**:
- ✅ **MUST**: Search latency breakdown (embedding, vector search, merge)
- ✅ **MUST**: Query complexity analysis y cost tracking
- ✅ **MUST**: Search quality metrics (relevance, click-through)
- ✅ **MUST**: Multi-tenant performance isolation
- ✅ **SHOULD**: Query pattern analysis
- ✅ **SHOULD**: Search result freshness tracking
- ✅ **SHOULD**: User satisfaction correlation

**Acceptance Criteria**:
- Search latency < 100ms (p95) medido continuamente
- Embedding generation < 1s por documento
- Vector search < 50ms para 100K vectors
- Multi-tenant overhead < 5% vs single-tenant

#### 3.1.3 Database Performance

**User Story**: Como SRE, quiero monitorear el rendimiento de bases de datos para prevenir cuellos de botella.

**Requirements**:
- ✅ **MUST**: PostgreSQL query performance monitoring
- ✅ **MUST**: Qdrant vector database metrics
- ✅ **MUST**: Connection pool utilization
- ✅ **MUST**: Index efficiency analysis
- ✅ **SHOULD**: Query plan analysis
- ✅ **SHOULD**: Lock contention monitoring
- ✅ **SHOULD**: Storage growth forecasting

**Acceptance Criteria**:
- Database queries > 50ms identificadas y alertadas
- Connection pool usage < 80% en operación normal
- Index usage > 95% para queries frecuentes
- Storage capacity alertado con 30 días de antelación

### 3.2 Infrastructure Monitoring

#### 3.2.1 System Metrics

**User Story**: Como DevOps engineer, quiero métricas completas de infraestructura para mantener la salud del sistema.

**Requirements**:
- ✅ **MUST**: CPU, memory, disk, network metrics
- ✅ **MUST**: Application-specific metrics (Phoenix, Oban, etc.)
- ✅ **MUST**: Container resource utilization
- ✅ **MUST**: Load balancer health y performance
- ✅ **SHOULD**: Custom business metrics
- ✅ **SHOULD**: Predictive scaling metrics
- ✅ **SHOULD**: Cost attribution por componente

**Acceptance Criteria**:
- System metrics colectadas con 1 segundo de resolución
- Alertas por umbral configurable
- Historical data retention por 90 días
- Cost tracking por organización y componente

#### 3.2.2 Multi-tenant Monitoring

**User Story**: Como engineering manager, quiero monitorear el rendimiento por organización para garantizar SLAs.

**Requirements**:
- ✅ **MUST**: Organization-specific performance metrics
- ✅ **MUST**: Tenant resource utilization tracking
- ✅ **MUST**: Performance isolation validation
- ✅ **MUST**: SLA compliance por tenant
- ✅ **SHOULD**: Noisy tenant identification
- ✅ **SHOULD**: Resource quota enforcement
- ✅ **SHOULD**: Tenant health scoring

**Acceptance Criteria**:
- Performance metrics disponibles por organización
- Noisy tenants detectados automáticamente
- Resource quotas enforced sin impacto a otros
- SLA compliance reporte diario

### 3.3 Alerting & Incident Management

#### 3.3.1 Intelligent Alerting

**User Story**: Como SRE, quiero alertas inteligentes que notifiquen problemas reales sin ruido.

**Requirements**:
- ✅ **MUST**: Multi-level alerting (warning, critical, emergency)
- ✅ **MUST**: Alert aggregation y deduplication
- ✅ **MUST**: Escalation policies basadas en impacto
- ✅ **MUST**: Alert fatigue prevention
- ✅ **SHOULD**: Machine learning para anomaly detection
- ✅ **SHOULD**: Predictive alerting basado en tendencias
- ✅ **SHOULD**: Business impact correlation

**Acceptance Criteria**:
- False positive rate < 5%
- Alert acknowledgment < 15 minutos
- Escalation automática sin respuesta en 30 minutos
- Alert fatigue prevention activa

#### 3.3.2 Incident Response

**User Story**: Como SRE, quiero herramientas eficientes para responder a incidentes rápidamente.

**Requirements**:
- ✅ **MUST**: Incident creation y tracking
- ✅ **MUST**: Automatic context collection (logs, metrics, traces)
- ✅ **MUST**: War room communication integration
- ✅ **MUST**: Post-mortem automation
- ✅ **SHOULD**: Runbook execution guidance
- ✅ **SHOULD**: Impact assessment automático
- ✅ **SHOULD**: Recovery time tracking

**Acceptance Criteria**:
- Incident creation < 1 minuto desde alerta
- Context collection automática completa
- Post-mortem generado automáticamente
- MTTR (Mean Time To Resolution) < 30 minutos

### 3.4 Performance Optimization

#### 3.4.1 Caching Strategy

**User Story**: Como backend developer, quiero caching inteligente para mejorar rendimiento sin sacrificing freshness.

**Requirements**:
- ✅ **MUST**: Multi-level caching (Redis, application, CDN)
- ✅ **MUST**: Cache invalidation inteligente
- ✅ **MUST**: Cache hit ratio monitoring
- ✅ **MUST**: Cache warming strategies
- ✅ **SHOULD**: Predictive cache preloading
- ✅ **SHOULD**: Cache size optimization
- ✅ **SHOULD**: Cache performance analytics

**Acceptance Criteria**:
- Cache hit ratio > 80% para searches frecuentes
- Cache invalidation < 5 segundos
- Cache warming para queries populares
- Memory usage < 70% de cache allocation

#### 3.4.2 Query Optimization

**User Story**: Como backend developer, quiero optimización automática de queries para mantener rendimiento.

**Requirements**:
- ✅ **MUST**: Slow query identification y reporting
- ✅ **MUST**: Query plan optimization suggestions
- ✅ **MUST**: Index recommendation engine
- ✅ **MUST**: Query pattern analysis
- ✅ **SHOULD**: Automatic query rewriting
- ✅ **SHOULD**: Database hint optimization
- ✅ **SHOULD**: Query result caching

**Acceptance Criteria**:
- Slow queries identificadas automáticamente
- Optimization suggestions con impacto estimado
- Index recommendations implementadas automáticamente
- Query performance mejorado > 20%

---

## 4. Technical Architecture

### 4.1 Monitoring Stack Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Applications  │    │   Infrastructure│    │   External APIs │
│  (Phoenix, etc)│    │  (Docker, K8s) │    │ (GitHub, Slack)│
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│                OpenTelemetry Collector                     │
│  - Traces collection                                        │
│  - Metrics aggregation                                      │
│  - Log processing                                           │
│  - Filtering y routing                                       │
└───────────────────────┬───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Prometheus  │  │   Loki      │  │   Jaeger    │
│ Metrics     │  │ Logs        │  │ Traces      │
└─────────────┘  └─────────────┘  └─────────────┘
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────────────────────────────────────────────────────┐
│                    Grafana                                  │
│  - Dashboards                                               │
│  - Alerting                                                 │
│  - Visualization                                            │
│  - Incident management                                       │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Data Flow Architecture

```
1. Application/Infrastructure → OpenTelemetry Collector
   ├─> Traces: HTTP requests, database queries, external calls
   ├─> Metrics: CPU, memory, custom business metrics
   ├─> Logs: Application logs, system logs, audit logs
   └─> Events: Custom business events, user actions

2. OpenTelemetry Collector → Storage Systems
   ├─> Prometheus: Time series metrics
   ├─> Loki: Log aggregation
   ├─> Jaeger: Distributed traces
   └─> Custom storage: Business events, analytics

3. Storage Systems → Grafana
   ├─> Dashboards: Real-time visualization
   ├─> Alerting: Rule evaluation y notifications
   ├─> Analytics: Trend analysis, forecasting
   └─> Reporting: SLA compliance, performance reports

4. Grafana → Alerting Channels
   ├─> PagerDuty: Critical incidents
   ├─> Slack: Operational alerts
   ├─> Email: Daily reports
   └─> Webhooks: Custom integrations
```

### 4.3 Monitoring Configuration

```yaml
# docker-compose.monitoring.yml
version: '3.8'
services:
  # OpenTelemetry Collector
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    config:
      receivers:
        otlp:
          protocols:
            grpc:
              endpoint: 0.0.0.0:4317
            http:
              endpoint: 0.0.0.0:4318
        prometheus:
          config:
            scrape_configs:
              - job_name: 'alejandria'
                static_configs:
                  - targets: ['app:4000']
      processors:
        batch:
          timeout: 1s
          send_batch_size: 1024
        memory_limiter:
          limit_mib: 512
      exporters:
        prometheus:
          endpoint: "0.0.0.0:8889"
        loki:
          endpoint: "http://loki:3100/loki/api/v1/push"
        jaeger:
          endpoint: jaeger:14250
          tls:
            insecure: true

  # Prometheus
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  # Loki
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"

  # Jaeger
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"
      - "14250:14250"

  # Grafana
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
```

---

## 5. Non-Functional Requirements

### 5.1 Performance Requirements

| Metric | Target | Measurement |
|--------|--------|-------------|
| Monitoring overhead | < 5% system resources | System metrics |
| Alert latency | < 30 seconds | Alert timestamps |
| Dashboard refresh | < 2 seconds | Browser timing |
| Query performance | < 1 second | Grafana metrics |
| Data retention | 90 days | Storage metrics |

### 5.2 Reliability Requirements

- **Monitoring uptime**: 99.9% (monitoring system itself)
- **Alert delivery**: > 99.5% success rate
- **Data accuracy**: 99.9% for metrics and logs
- **Backup retention**: 30 days for monitoring data

### 5.3 Security Requirements

- **Authentication**: Role-based access to monitoring
- **Authorization**: Granular permissions por dashboard
- **Data encryption**: TLS 1.3 for all monitoring traffic
- **Audit logging**: All access to monitoring systems logged

### 5.4 Scalability Requirements

| Dimension | MVP | Year 1 | Year 2 |
|-----------|-----|--------|--------|
| Metrics series | 100K | 1M | 10M |
| Log volume | 1GB/day | 10GB/day | 100GB/day |
| Trace spans | 1M/day | 10M/day | 100M/day |
| Concurrent users | 50 | 500 | 2000 |

---

## 6. Implementation Plan

### 6.1 Development Phases

#### Week 5: Core Monitoring Setup
- **Day 29-31**: OpenTelemetry instrumentation
- **Day 32-33**: Prometheus + Loki + Jaeger deployment
- **Day 34-35**: Grafana dashboards básicos

#### Week 6: Advanced Features
- **Day 36-38**: Alerting rules y escalation
- **Day 39-40**: Performance optimization features
- **Day 41-42**: Multi-tenant monitoring y SLA tracking

### 6.2 Technical Acceptance Criteria

#### Monitoring Coverage Tests
```elixir
test "all HTTP requests traced with correlation IDs" do
  # Verify complete request tracing
end

test "database queries monitored for performance" do
  # Verify query performance tracking
end

test "external service calls tracked with latency" do
  # Verify external service monitoring
end
```

#### Alerting Tests
```elixir
test "critical alerts trigger within 30 seconds" do
  # Verify alert latency
end

test "false positive rate below 5%" do
  # Verify alert quality
end
```

### 6.3 Risk Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Monitoring overhead | Medium | Medium | Efficient sampling, async processing |
| Alert fatigue | High | Medium | Smart aggregation, ML-based filtering |
| Data volume explosion | Medium | High | Retention policies, compression |
| Complex configuration | Medium | Low | Templates, automation |

---

## 7. Success Criteria & Metrics

### 7.1 MVP Success (Week 6)

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Monitoring coverage | 100% critical paths | Code instrumentation audit |
| Alert latency | < 30 seconds | Alert timestamp analysis |
| Dashboard usability | > 80% user satisfaction | User surveys |
| System visibility | Complete stack trace | End-to-end testing |

### 7.2 Performance Validation

| Metric | Target | Test Method |
|--------|--------|-------------|
| Search latency monitoring | < 100ms (p95) | Continuous monitoring |
| Database performance tracking | < 50ms queries | Query analysis |
| Multi-tenant isolation verification | < 5% overhead | Load testing |
| Cache hit ratio | > 80% | Cache metrics |

### 7.3 Operational Excellence

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| MTTR (Mean Time To Resolution) | < 30 minutes | Incident tracking |
| False positive rate | < 5% | Alert analysis |
| Monitoring system uptime | > 99.9% | System monitoring |
| User adoption | > 90% of operations team | Usage analytics |

---

## 8. Out of Scope (PRD-004)

### Explicitly NOT in this PRD

- ❌ **Advanced AI-powered anomaly detection** → Fase 2
- ❌ **Predictive auto-scaling** → Fase 2
- ❌ **Business intelligence analytics** → Fase 3
- ❌ **Customer-facing monitoring portals** → Fase 3
- ❌ **Advanced cost optimization** → Fase 2
- ❌ **Multi-cloud monitoring** → Fase 3

### Dependencies on Other PRDs

- **PRD-001 (Core Infrastructure)**: Monitorea database performance, API latency
- **PRD-002 (MCP Server)**: Monitorea AI agent performance, usage patterns
- **PRD-003 (Integration Hub)**: Monitorea webhook processing, sync health

---

## 9. Dependencies & Risks

### 9.1 Technical Dependencies

| Dependency | Risk Level | Mitigation |
|------------|------------|------------|
| OpenTelemetry stability | Low | Vendor-neutral, multiple backends |
| Prometheus scalability | Medium | Federation, remote storage |
| Grafana performance | Low | Caching, optimized queries |
| Storage costs | Medium | Retention policies, compression |

### 9.2 Operational Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Monitoring system failure | High | Low | Redundancy, health checks |
| Alert fatigue | Medium | Medium | Smart filtering, ML |
| Data overload | Medium | High | Sampling, retention |
| Configuration complexity | Medium | Low | Templates, automation |

### 9.3 Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| High monitoring costs | Medium | Medium | Efficient design, cost tracking |
| Team adoption resistance | Medium | Low | Training, clear value prop |
| Over-monitoring | Medium | Medium | Focus on critical metrics |
| Vendor lock-in | Low | Low | Open standards, export capabilities |

---

## 10. Testing Strategy

### 10.1 Unit Tests (Integrated in Development)

**Coverage Requirements**:
- > 90% coverage en monitoring instrumentation
- 100% coverage en alerting rules
- 100% coverage en metric collection

**Key Test Scenarios**:
```elixir
# OpenTelemetry instrumentation
test "HTTP requests traced with proper attributes" do
  # Verify request tracing attributes
end

# Metric collection
test "business metrics collected accurately" do
  # Verify metric accuracy
end

# Alerting logic
test "alert conditions evaluated correctly" do
  # Verify alert rule evaluation
end
```

### 10.2 Integration Tests

**End-to-End Monitoring**:
- Request tracing across system boundaries
- Alert delivery to notification channels
- Dashboard data accuracy

**Performance Monitoring**:
- Load testing with monitoring enabled
- Performance regression detection
- Multi-tenant performance isolation

### 10.3 Load Testing

**Monitoring System Performance**:
- 10K metrics/second ingestion
- 1M log entries/day processing
- 100 concurrent dashboard users

**Test Scenarios**:
```bash
# Metrics ingestion test
./test/metrics_load_test.sh --rate=10000 --duration=10m

# Alert processing test
./test/alert_processing_test.sh --alerts=1000 --concurrent=100
```

### 10.4 Chaos Testing

**System Resilience**:
- Component failure simulation
- Network partition testing
- Resource exhaustion scenarios

---

## 11. Technical Specifications

### 11.1 OpenTelemetry Configuration

```elixir
# config/telemetry.ex
defmodule Alejandria.Telemetry do
  use Supervisor
  require OpenTelemetry.Tracer

  def start_link(opts) do
    Supervisor.start_link(__MODULE__, opts, name: __MODULE__)
  end

  @impl true
  def init(_opts) do
    children = [
      # OpenTelemetry exporter
      {OpenTelemetry.Exporter,
       endpoints: [
         {:http, "http://otel-collector:4318"}
       ]},
      
      # Phoenix telemetry
      {Phoenix.PubSub, name: Alejandria.PubSub},
      
      # Database telemetry
      {Ecto.Telemetry, event_prefix: [:alejandria, :repo]}
    ]

    Supervisor.init(children, strategy: :one_for_one)
  end

  def setup_instrumentation do
    # Phoenix endpoint tracing
    Phoenix.Endpoint.instrument(:phoenix, :endpoint_start, %{
      conn: conn,
      request_path: request_path
    ) do
      OpenTelemetry.Tracer.set_attribute("http.method", conn.method)
      OpenTelemetry.Tracer.set_attribute("http.url", request_path)
      OpenTelemetry.Tracer.set_attribute("http.status_code", conn.status)
    end

    # Database query tracing
    Ecto.Telemetry.attach(
      "alejandria-ecto",
      [:alejandria, :repo, :query],
      &handle_ecto_event/4,
      nil
    )
  end

  defp handle_ecto_event(event, measurements, metadata, _config) do
    OpenTelemetry.Tracer.with_span("ecto.query", %{
      "db.statement" => metadata.query,
      "db.duration_ms" => measurements.query_time,
      "db.result_count" => metadata.result
    }) do
      # Additional processing
    end
  end
end
```

### 11.2 Prometheus Metrics Definition

```elixir
# lib/alejandria/metrics.ex
defmodule Alejandria.Metrics do
  use Prometheus.Metric

  def setup do
    # Search performance metrics
    Histogram.declare(
      name: :alejandria_search_duration_seconds,
      help: "Search request duration in seconds",
      labels: [:organization_id, :query_type],
      buckets: [0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
    )

    # Database metrics
    Histogram.declare(
      name: :alejandria_db_query_duration_seconds,
      help: "Database query duration in seconds",
      labels: [:query_type, :table],
      buckets: [0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5]
    )

    # MCP server metrics
    Counter.declare(
      name: :alejandria_mcp_requests_total,
      help: "Total MCP server requests",
      labels: [:tool, :organization_id, :status]
    )

    # Integration webhook metrics
    Counter.declare(
      name: :alejandria_webhook_received_total,
      help: "Total webhooks received",
      labels: [:source, :event_type, :organization_id]
    )

    Histogram.declare(
      name: :alejandria_webhook_processing_duration_seconds,
      help: "Webhook processing duration in seconds",
      labels: [:source, :event_type],
      buckets: [0.1, 0.5, 1, 2.5, 5, 10, 30, 60]
    )
  end

  def observe_search_duration(duration_ms, org_id, query_type) do
    Histogram.observe(
      [name: :alejandria_search_duration_seconds, 
       labels: [org_id, query_type]], 
      duration_ms / 1000
    )
  end

  def increment_mcp_requests(tool, org_id, status) do
    Counter.inc(
      [name: :alejandria_mcp_requests_total,
       labels: [tool, org_id, status]]
    )
  end
end
```

### 11.3 Grafana Dashboard Configuration

```json
{
  "dashboard": {
    "title": "Alejandria Performance Dashboard",
    "panels": [
      {
        "title": "Search Performance",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(alejandria_search_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          },
          {
            "expr": "histogram_quantile(0.50, rate(alejandria_search_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile"
          }
        ],
        "yAxes": [
          {
            "max": 0.2,
            "min": 0,
            "unit": "seconds"
          }
        ]
      },
      {
        "title": "Database Performance",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(alejandria_db_query_duration_seconds_sum[5m]) / rate(alejandria_db_query_duration_seconds_count[5m])",
            "legendFormat": "Average query time"
          }
        ]
      },
      {
        "title": "MCP Server Requests",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(alejandria_mcp_requests_total[5m])",
            "legendFormat": "Requests/sec"
          }
        ]
      },
      {
        "title": "Webhook Processing",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(alejandria_webhook_received_total[5m])",
            "legendFormat": "Webhooks/sec"
          },
          {
            "expr": "histogram_quantile(0.95, rate(alejandria_webhook_processing_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile processing time"
          }
        ]
      }
    ],
    "time": {
      "from": "now-1h",
      "to": "now"
    },
    "refresh": "5s"
  }
}
```

---

## 12. Alerting Rules Configuration

### 12.1 Prometheus Alerting Rules

```yaml
# prometheus_alerts.yml
groups:
  - name: alejandria_performance
    rules:
      # Search performance alerts
      - alert: SearchLatencyHigh
        expr: histogram_quantile(0.95, rate(alejandria_search_duration_seconds_bucket[5m])) > 0.1
        for: 2m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "Search latency is high"
          description: "95th percentile search latency is {{ $value }}s for the last 2 minutes"

      - alert: SearchLatencyCritical
        expr: histogram_quantile(0.95, rate(alejandria_search_duration_seconds_bucket[5m])) > 0.2
        for: 1m
        labels:
          severity: critical
          service: alejandria
        annotations:
          summary: "Search latency is critical"
          description: "95th percentile search latency is {{ $value }}s - immediate attention required"

      # Database performance alerts
      - alert: DatabaseQuerySlow
        expr: rate(alejandria_db_query_duration_seconds_sum[5m]) / rate(alejandria_db_query_duration_seconds_count[5m]) > 0.05
        for: 3m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "Database queries are slow"
          description: "Average database query time is {{ $value }}s"

      # MCP server alerts
      - alert: MCPServerErrorRate
        expr: rate(alejandria_mcp_requests_total{status="error"}[5m]) / rate(alejandria_mcp_requests_total[5m]) > 0.05
        for: 1m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "MCP server error rate is high"
          description: "Error rate is {{ $value | humanizePercentage }}"

      # Webhook processing alerts
      - alert: WebhookProcessingSlow
        expr: histogram_quantile(0.95, rate(alejandria_webhook_processing_duration_seconds_bucket[5m])) > 30
        for: 5m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "Webhook processing is slow"
          description: "95th percentile processing time is {{ $value }}s"

      # System resource alerts
      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes > 0.9
        for: 5m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value | humanizePercentage }}"

      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          service: alejandria
        annotations:
          summary: "High CPU usage"
          description: "CPU usage is {{ $value }}%"
```

### 12.2 Alerting Configuration

```yaml
# alertmanager.yml
global:
  smtp_smarthost: 'localhost:587'
  smtp_from: 'alerts@alejandria.ai'

route:
  group_by: ['alertname', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
    - match:
        severity: warning
      receiver: 'slack-alerts'

receivers:
  - name: 'web.hook'
    webhook_configs:
      - url: 'http://localhost:5001/'

  - name: 'pagerduty'
    pagerduty_configs:
      - routing_key: 'your-pagerduty-key'
        description: '{{ .CommonAnnotations.summary }}'
        details:
          firing: '{{ .Alerts.Firing | len }}'
          resolved: '{{ .Alerts.Resolved | len }}'

  - name: 'slack-alerts'
    slack_configs:
      - api_url: 'YOUR_SLACK_WEBHOOK_URL'
        channel: '#alerts'
        title: 'Alejandria Alert'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
```

---

**Approval**:
- [ ] Product Lead
- [ ] Engineering Lead  
- [ ] DevOps Lead
- [ ] SRE Lead

**Next Steps**:
1. Review con stakeholders (Week 5)
2. Setup monitoring infrastructure (Week 5)
3. Begin instrumentation implementation (Week 5)
