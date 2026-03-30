---
id: ENG-DEC-001
area: "engineering"
type: "decision"
title: "ADR-0001: Elixir/Phoenix como Backend"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["ENG-DEC-002", "ENG-DEF-0001", "PROD-DEF-0001", "PROD-STRAT-0011"]
---

Architecture Decision Record que documenta la elección de Elixir/Phoenix como framework backend para Alejandria. Contiene análisis de requisitos técnicos de alta concurrencia y fault-tolerance, evaluación de alternativas (Node.js, Python, Go, Rust), justificación de la decisión basada en capacidades de concurrencia masiva, recuperación automática de fallos y hot code swapping, y plan de implementación con arquitectura de clustering y estrategia de despliegue.

---

# ADR-0001: Elixir/Phoenix como Backend

**Status**: Accepted  
**Date**: 2026-03-20  
**Authors**: Equipo Técnico Alejandria  
**Reviewers**: Tech Lead, Architecture Team  

## Context

Alejandria requiere un backend capaz de manejar múltiples integraciones en tiempo real (webhooks de GitHub, Shortcut, etc.), procesamiento asíncrono de documentos, generación de embeddings, y búsquedas concurrentes de alta performance. El sistema debe ser resiliente a fallos y capaz de escalar horizontalmente.

### Requisitos Técnicos

1. **Alta concurrencia**: Procesar múltiples webhooks simultáneos sin bloqueo
2. **Fault-tolerance**: Recuperación automática de fallos en workers
3. **Real-time updates**: WebSockets para notificaciones en vivo
4. **Background processing**: Jobs asíncronos para embeddings y análisis
5. **Hot code swapping**: Deploy sin downtime
6. **Escalabilidad horizontal**: Clustering nativo para múltiples nodos

### Contexto del Equipo

- Experiencia previa con sistemas distribuidos
- Familiaridad con programación funcional
- Necesidad de mantener codebase a largo plazo
- Prioridad en robustez sobre velocidad de desarrollo inicial

## Decision

**Usaremos Elixir 1.15+ con Phoenix 1.7 como framework backend para Alejandria.**

### Componentes Clave

- **Phoenix Framework**: API REST, WebSockets (LiveView opcional para admin)
- **Ecto**: ORM para PostgreSQL con migraciones versionadas
- **Oban**: Background jobs con persistencia y retry automático
- **Tesla**: Cliente HTTP para integraciones externas
- **Jason**: JSON encoding/decoding de alta performance

### Arquitectura de Deployment

```
┌─────────────────────────────────────┐
│     Load Balancer (HAProxy)        │
└──────────────┬──────────────────────┘
               │
       ┌───────┴────────┐
       ▼                ▼
┌─────────────┐  ┌─────────────┐
│  Phoenix    │  │  Phoenix    │
│  Node 1     │  │  Node 2     │
│  (Cluster)  │  │  (Cluster)  │
└─────────────┘  └─────────────┘
       │                │
       └────────┬───────┘
                ▼
       ┌─────────────────┐
       │   PostgreSQL    │
       │   (Primary +    │
       │    Replicas)    │
       └─────────────────┘
```

## Consequences

### Positivas

✅ **Concurrencia masiva**: BEAM VM maneja millones de procesos ligeros
- Cada webhook request corre en su propio proceso aislado
- No hay bloqueo entre requests concurrentes
- Latencia predecible bajo carga alta

✅ **Fault-tolerance inherente**: Supervisors y let-it-crash philosophy
- Workers que fallan se reinician automáticamente
- Errores aislados no afectan el sistema completo
- Telemetría detallada de fallos

✅ **Hot code swapping**: Deploy sin downtime
- Actualizaciones de código en caliente
- Rollback instantáneo si hay problemas
- Crítico para sistema 24/7

✅ **Clustering nativo**: Escalabilidad horizontal simple
- Nodos se descubren automáticamente con libcluster
- Estado compartido vía distributed Erlang
- Load balancing automático

✅ **Ecosystem maduro para nuestro caso de uso**:
- Oban para background jobs (mejor que Sidekiq/Celery)
- Phoenix PubSub para real-time
- Ecto para migraciones complejas
- ExUnit para testing robusto

✅ **Performance excelente**:
- ~50k requests/sec en hardware modesto
- Latencia p95 < 50ms para queries típicas
- Uso de memoria predecible

### Negativas

❌ **Curva de aprendizaje más alta**:
- Paradigma funcional requiere cambio de mentalidad
- OTP patterns (GenServers, Supervisors) son nuevos para muchos
- **Mitigación**: Pair programming, code reviews, documentación interna

❌ **Ecosistema más pequeño que Node.js/Python**:
- Menos librerías de terceros disponibles
- Algunas integraciones requieren implementación custom
- **Mitigación**: Elixir interopera con Erlang (ecosystem más grande)

❌ **Debugging más complejo en producción**:
- Procesos concurrentes dificultan tracing
- **Mitigación**: Telemetry + AppSignal/New Relic, Observer tool

❌ **Hiring más difícil**:
- Pool de developers Elixir es menor
- **Mitigación**: Entrenar equipo actual, Elixir es aprendible en 2-3 meses

### Riesgos Identificados

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Equipo no domina Elixir a tiempo | Media | Alto | Capacitación intensiva primeras 4 semanas |
| Falta de librerías para integraciones | Baja | Medio | Usar FFI con Rust/Go si necesario |
| Performance de Ecto en queries complejas | Baja | Medio | Usar SQL raw cuando sea necesario |

## Alternatives Considered

### Node.js + Express/Fastify

**Pros**:
- Equipo ya lo conoce
- Ecosystem enorme
- Rápido para prototipar

**Cons**:
- Single-threaded: concurrencia limitada
- Callback hell / Promise chains complejas
- Fault-tolerance requiere PM2/cluster module (frágil)
- Memory leaks comunes en long-running processes

**Razón de rechazo**: No maneja bien alta concurrencia ni fault-tolerance nativa.

### Python + Django/FastAPI

**Pros**:
- Excelente para ML/AI (integración con modelos)
- Ecosystem científico robusto
- Sintaxis clara

**Cons**:
- GIL limita concurrencia real
- Async Python (asyncio) es complejo y propenso a errores
- Performance inferior para I/O-bound tasks
- Deployment más complejo (Gunicorn/uWSGI)

**Razón de rechazo**: GIL es bloqueante para nuestro caso de uso de alta concurrencia.

### Go + Gin/Echo

**Pros**:
- Performance excelente
- Concurrencia con goroutines
- Binarios estáticos fáciles de deployar
- Tipado estático fuerte

**Cons**:
- Más verbose que Elixir
- No tiene hot code swapping
- Error handling tedioso (if err != nil)
- ORM débiles comparado con Ecto
- No tiene supervisors nativos

**Razón de rechazo**: Falta de hot code swapping y supervisors nativos. Go es excelente pero Elixir es superior para sistemas distribuidos resilientes.

### Rust + Actix/Axum

**Pros**:
- Performance máxima
- Memory safety sin GC
- Concurrencia segura

**Cons**:
- Curva de aprendizaje extremadamente alta (borrow checker)
- Desarrollo más lento
- Ecosystem web menos maduro
- Overkill para nuestro caso de uso

**Razón de rechazo**: Complejidad no justificada. No necesitamos performance de Rust para este sistema.

## Implementation

### Fase 1: Setup Inicial (Semana 1)

```bash
# Crear proyecto Phoenix
mix phx.new alejandria --database postgres --no-html --no-assets

# Agregar dependencias clave
# mix.exs
defp deps do
  [
    {:phoenix, "~> 1.7.0"},
    {:ecto_sql, "~> 3.10"},
    {:postgrex, ">= 0.0.0"},
    {:oban, "~> 2.15"},
    {:tesla, "~> 1.7"},
    {:jason, "~> 1.4"},
    {:telemetry_metrics, "~> 0.6"},
    {:telemetry_poller, "~> 1.0"}
  ]
end
```

### Fase 2: Arquitectura Core (Semana 2-3)

1. **Contexts**: Documents, Search, Integrations, Embeddings
2. **Background Jobs**: Oban queues para embedding generation
3. **WebSocket**: Phoenix Channels para real-time updates
4. **API**: RESTful endpoints con versioning

### Fase 3: Testing & Monitoring (Semana 4)

1. **ExUnit tests**: Unit, integration, property-based
2. **Telemetry**: Métricas custom para business logic
3. **Observability**: AppSignal o New Relic integration

### Fase 4: Production Readiness (Semana 5-6)

1. **Clustering**: libcluster con Kubernetes/Nomad
2. **Releases**: Mix releases para deployment
3. **CI/CD**: GitHub Actions con Docker builds

## Related Decisions

- **ADR-0002**: Qdrant para búsqueda vectorial (Elixir tiene cliente Qdrant oficial)
- PostgreSQL es la base de datos elegida (Elixir + Ecto + PostgreSQL es stack probado)

## References

- [Elixir Official Docs](https://elixir-lang.org/)
- [Phoenix Framework](https://www.phoenixframework.org/)
- [Oban Background Jobs](https://getoban.pro/)
- [Discord usa Elixir](https://discord.com/blog/how-discord-scaled-elixir-to-5-000-000-concurrent-users) - 5M usuarios concurrentes
- [WhatsApp usa Erlang/BEAM](https://www.erlang.org/blog/20-years-of-open-source-erlang/) - 900M usuarios

---

**Decisión final**: Elixir/Phoenix es la mejor opción para un sistema distribuido, resiliente y de alta concurrencia como Alejandria. Los trade-offs (curva de aprendizaje, ecosystem más pequeño) son aceptables dado los beneficios en fault-tolerance y escalabilidad.
