---
id: PROD-STRAT-0008
area: "product"
type: "strategy"
title: "Fase 1 - MVP: Infraestructura Core + MCP"
author: "product-team"
status: "planned"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["PROD-STRAT-0002", "PROD-STRAT-0001"]
---

Implementación detallada de la Fase 1 del roadmap de Alejandria. Esta fase de 6 semanas establece la infraestructura core con búsqueda semántica y el servidor MCP para agentes de IA.

---

# Fase 1 - MVP: Infraestructura Core + MCP

## Objetivo Principal

Backend funcional con búsqueda semántica y acceso MCP para agentes de IA, permitiendo que herramientas como Claude, Cursor y VS Code puedan acceder al conocimiento organizacional.

## Timeline Detallado

### Semana 1-2: Base de Datos y Embeddings

#### Configuración PostgreSQL 16+ con Apache AGE
- Instalación y configuración de PostgreSQL 16+
- Activación de Apache AGE extension para grafos
- Configuración de schemas para documentos, usuarios, relaciones
- Setup de indexes optimizados para búsqueda híbrida

#### Setup Qdrant para Búsqueda Vectorial
- Deploy de cluster Qdrant para producción
- Configuración de collections multi-embedding
- Optimización de parámetros de búsqueda y ranking
- Integración con PostgreSQL para metadata

#### Implementación de Pipeline de Embeddings
- **BGE-M3**: Para texto general y documentos
- **Jina Code**: Para código y documentación técnica  
- **SPLADE-v3**: Para búsqueda sparse y keyword
- Configuración de batch processing y caching

#### Schema Inicial
- Documents: metadata, content, embeddings, timestamps
- Users: profiles, permissions, activity
- Relationships: citations, dependencies, collaborations

### Semana 3-4: Backend Core

#### API Elixir/Phoenix
- Endpoints básicos de búsqueda y recuperación
- Middleware de autenticación y rate limiting
- Error handling y logging estructurado
- Background jobs para procesamiento asíncrono

#### Webhooks GitHub
- Configuración de webhook endpoints
- Procesamiento de commits y PRs
- Extracción de decisiones técnicas
- Sincronización con base de datos

#### Procesamiento Shortcut
- API integration para stories y epics
- Parsing de decisiones y blockers
- Clasificación por tipo y prioridad
- Mapeo a entidades del grafo

#### Integración Slack
- Conexión a Slack API
- Identificación de conversaciones importantes
- Extracción de contextos y decisiones
- Clasificación por canales y temas

### Semana 5-6: MCP Server

#### Implementación MCP Server
- Protocolo MCP v1.0 compliance
- Tools de búsqueda y recuperación
- Context management y session handling
- Error recovery y retry logic

#### Indexación Automática
- Sincronización incremental de fuentes
- Detección de cambios y actualizaciones
- Reindexing de embeddings modificados
- Monitoring de sync status

#### API Interna
- Endpoints de configuración y admin
- Health checks y diagnostics
- Metrics y performance monitoring
- User management y permissions

#### Testing con Agentes de IA
- Validación con Claude/Cursor/VS Code
- Performance testing bajo carga
- User experience testing
- Bug fixes y optimización

## Recursos Necesarios

### Personal
- **Backend Developer (Full-time)**: Elixir/Phoenix, PostgreSQL, Qdrant
- **DevOps Support (Part-time)**: Docker, CI/CD, monitoring setup  
- **AI/ML Specialist (Consulting)**: Embedding models, vector search optimization

### Infraestructura
- PostgreSQL 16+ con Apache AGE extension
- Qdrant cluster para búsqueda vectorial
- Redis para caching y sesiones
- Docker containers para desarrollo y deploy

### Dependencias Externas
- GitHub API (rate limit: 5000/hour)
- Slack API (rate limit: 1 request/second)
- Shortcut API (rate limit: 100/minute)
- nomic-embed-text-v1.5 (local deployment)

## Requisitos de Seguridad

### Authentication
- API keys para MCP server
- Rate limiting básico para prevenir abuse
- Input validation y sanitización

### Data Protection
- TLS 1.3 para todos los communications
- AES-256 encryption en PostgreSQL
- Secure storage de API keys y secrets

### Monitoring
- Security events en logs centralizados
- Anomaly detection en access patterns
- Audit trails para cambios críticos

## Estrategia de Testing

### Unit Tests
- >90% coverage en backend core
- Tests para todos los endpoints de API
- Validación de schema y data integrity
- Tests de business logic

### Integration Tests
- APIs externas (GitHub, Slack, Shortcut)
- MCP protocol compliance
- Database operations y transactions
- Error handling y recovery

### Load Testing
- 1000 queries simultáneas
- 10K queries/hora sustained load
- Performance benchmarking
- Resource utilization monitoring

### Security Tests
- OWASP Top 10 vulnerability scanning
- Penetration testing de endpoints
- API key validation y rotation
- Data encryption verification

### Performance Tests
- Search latency <100ms (p95)
- API response time <200ms promedio
- Indexing speed <30s por documento
- Memory usage <80% under load

## Entregables

### Backend Core
- Backend con búsqueda híbrida funcional
- API REST endpoints documentados
- Webhooks para GitHub, Slack, Shortcut
- Sistema de indexación automática

### MCP Integration
- MCP server para Claude, Cursor, VS Code
- Tools de búsqueda y recuperación
- Documentation para desarrolladores
- Examples de uso y tutorials

### Operations
- API interna de administración
- Monitoring y alerting setup
- CI/CD pipeline configurado
- Documentation técnica completa

## Métricas de Validación

### Technical KPIs
- **Search precision**: >85% en top-5 resultados
- **Search latency**: <100ms (p95)
- **API response time**: <200ms promedio
- **Uptime**: >99% durante testing
- **Indexing speed**: <30s por documento

### Integration KPIs
- **MCP stability**: Conexión exitosa con agentes
- **Sync success**: >99% de sincronizaciones exitosas
- **API reliability**: <0.1% error rate
- **Data freshness**: <5 minutos de latency

### Business KPIs
- **Query volume**: >100 queries/day en testing
- **User adoption**: >70% de equipos técnicos usando
- **Time to value**: <2 minutos para primera búsqueda exitosa
- **Satisfaction**: NPS >30 en beta testing

## Dependencias y Riesgos

### Dependencias Críticas
- Disponibilidad de APIs externas
- Performance de embedding models
- Stability de Qdrant cluster
- Expertise en Elixir/Phoenix

### Riesgos Mitigación
- **Search quality**: Testing continuo y feedback loops
- **API limits**: Caching y batching strategies
- **Performance**: Load testing y optimization
- **Security**: Regular security audits

## Próxima Fase

Al completar esta fase, el equipo estará listo para la Fase 2 - Visor Web, que construirá la interfaz de usuario para explorar el contenido indexado.

---
**Documentos Relacionados:**
- [Roadmap General](implementation-roadmap.md)
- [Recursos Técnicos](recursos-tecnicos.md)
- [Validación y Métricas](validacion-metricas.md)
