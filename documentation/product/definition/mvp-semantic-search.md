---
id: PRD-0004
area: "product"
type: "definition"
title: "MVP Semantic Search"
author: "product-team"
status: "draft"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["PROD-STRAT-0001", "ENG-DEF-0001", "PRD-0002", "ADR-0004", "ADR-0005"]
---

Documento de requerimientos de producto que define el feature de búsqueda semántica para el MVP de Alejandria. Contiene análisis del problema de búsqueda basada en keywords, implementación de embeddings y vector search, integración con Qdrant, user stories, requerimientos funcionales y no funcionales, consideraciones técnicas y métricas de éxito.

---

# PRD-0004: MVP Semantic Search

**Status**: Draft  
**Date**: 2026-04-24  
**Author**: Product Team  
**Reviewer**: Engineering Lead  
**Target Release**: MVP v1.0

## 1. Overview

### Problem Statement

Sabemos que buscar información basada en keywords exactos es frustrante y limitado. Cuando tú preguntas "¿cómo manejar errores en producción?", una búsqueda por keywords fallaría si el documento usa "exceptions", "failures" o "incidents". Necesitamos búsqueda que entienda el significado, no solo las palabras.

Entendemos que el MVP necesita búsqueda semántica que te permita:
- Encontrar documentos usando lenguaje natural
- Obtener resultados relevantes aunque uses palabras diferentes
- Filtrar resultados por ACL y organización
- Mantener latencia baja (<300ms)
- Escalar a millones de documentos

### Success Metrics

- **Búsquedas exitosas**: 80% de búsquedas retornan resultados relevantes
- **Latencia promedio**: <300ms para operaciones de búsqueda típicas
- **Satisfacción**: NPS > 40 para experiencia de búsqueda
- **Documentos indexados**: 1000 documentos indexados en primer mes

### Goals

- **Goal 1**: Implementar búsqueda semántica con embeddings
- **Goal 2**: Integrar con Qdrant para vector search
- **Goal 3**: Filtrar resultados por ACL y organización
- **Goal 4**: Mantener latencia <300ms para queries típicas

## 2. User Analysis

### Target Users

**Primary Users:**
- Tú, como desarrollador, que busca información técnica usando lenguaje natural
- Tú, como Product Manager, que busca decisiones de producto sin conocer keywords exactos
- Tú, como líder técnico, que busca arquitectura y decisiones pasadas

**Secondary Users:**
- Agentes de IA que necesitan contexto organizacional
- Operaciones que buscan procedimientos y guías

### User Stories

#### User Story 1: Buscar con lenguaje natural

**Como** desarrollador usando Claude Desktop  
**quiero** buscar documentos usando lenguaje natural  
**para que** encuentre contexto relevante sin conocer keywords exactos

**Acceptance Criteria:**
- [ ] Tú puedes invocar `search_documents` via MCP con query natural
- [ ] El sistema genera embedding del query y busca en Qdrant
- [ ] Los resultados se ordenan por relevancia semántica
- [ ] Tú recibes top-k resultados con snippets y metadata
- [ ] La latencia de búsqueda es <500ms para queries típicas

#### User Story 2: Búsqueda en español e inglés

**Como** usuario hispanohablante  
**quiero** buscar en español e inglés  
**para que** encuentre documentos en ambos idiomas

**Acceptance Criteria:**
- [ ] El modelo de embeddings soporta español e inglés
- [ ] Búsquedas en español retornan resultados en español
- [ ] Búsquedas en inglés retornan resultados en inglés
- [ ] Búsquedas mixtas retornan resultados relevantes

#### User Story 3: Filtrar por organización

**Como** miembro de organización  
**quiero** buscar solo documentos de mi organización  
**para que** no vea resultados de otras organizaciones

**Acceptance Criteria:**
- [ ] La búsqueda filtra por organization_id
- [ ] Solo documentos de la organización del usuario aparecen en resultados
- [ ] La latencia con filtros de organización es <300ms

#### User Story 4: Filtrar por ACL

**Como** usuario con permisos limitados  
**quiero** buscar documentos  
**para que** solo vea resultados que tengo permiso de leer

**Acceptance Criteria:**
- [ ] La búsqueda filtra resultados por ACL del usuario
- [ ] Documentos privados sin permiso no aparecen en resultados
- [ ] Documentos públicos de otras organizaciones no aparecen en resultados
- [ ] La latencia con filtros ACL es <500ms

### User Journey

**Flujo de búsqueda de documento:**
1. Tú preguntas al agente: "¿Qué sabemos sobre la arquitectura de autenticación?"
2. El agente invoca `search_documents` via MCP con query natural
3. El sistema genera embedding del query
4. El sistema busca en Qdrant con filtros ACL y organización
5. El sistema retorna top-k resultados relevantes
6. El agente te presenta los resultados con contexto
7. Tú seleccionas un documento para leer completo
8. Agente invoca `get_document` via MCP con document_id
9. Sistema retorna documento completo
10. Agente presenta contenido al usuario

## 3. Functional Requirements

### Core Features

#### Feature 1: Generar embeddings de query

**Description**: Generar embedding del query de búsqueda

**Requirements:**
- [ ] Aceptar parámetros: query (texto natural)
- [ ] Validar que query no está vacío
- [ ] Truncar query a 512 tokens máximo
- [ ] Generar embedding usando sentence-transformers
- [ ] Retornar vector de embeddings (dimension 384 inicial)

**Edge Cases:**
- [Query vacío]: Rechazar con error "query cannot be empty"
- [Query muy largo]: Truncar a 512 tokens máximo
- [Query inválido]: Rechazar con error "invalid query format"

#### Feature 2: Búsqueda por similitud coseno

**Description**: Buscar documentos por similitud coseno en Qdrant

**Requirements:**
- [ ] Aceptar parámetros: query_embedding, organization_id, limit (default 10)
- [ ] Buscar por similitud coseno en Qdrant (top-k = limit)
- [ ] Filtrar por organization_id
- [ ] Filtrar por ACL del usuario
- [ ] Reranking con metadata adicional
- [ ] Retornar resultados con snippets, scores y metadata

**Edge Cases:**
- [Sin resultados]: Retornar lista vacía con mensaje "no results found"
- [ACL restrictivo]: Filtrar resultados que usuario no tiene permiso de leer
- [Organización inválida]: Rechazar con error "organization not found"

#### Feature 3: Filtrado por ACL

**Description**: Filtrar resultados de búsqueda por ACL del usuario

**Requirements:**
- [ ] Filtrar resultados por ACL del usuario
- [ ] Aplicar filtros en Qdrant usando metadata ACL
- [ ] Retornar solo documentos que usuario tiene permiso de leer
- [ ] Mantener latencia <500ms con filtros ACL

**Edge Cases:**
- [ACL restrictivo]: Filtrar resultados que usuario no tiene permiso de leer
- [Documento deleted]: Excluir de resultados
- [Documento archivado]: Excluir de resultados (a menos que se especifique)

#### Feature 4: Filtrado por organización

**Description**: Filtrar resultados por organización del usuario

**Requirements:**
- [ ] Filtrar resultados por organization_id
- [ ] Solo documentos de la organización del usuario aparecen
- [ ] Mantener latencia <300ms con filtros de organización
- [ ] Usar colecciones por organización en Qdrant

**Edge Cases:**
- [Organización inválida]: Rechazar con error "organization not found"
- [Organización vacía]: Retornar lista vacía
- [Multi-organización]: Filtrar por todas las organizaciones del usuario

#### Feature 5: Reranking con metadata

**Description**: Reordenar resultados usando metadata adicional

**Requirements:**
- [ ] Reranking basado en scores de similitud
- [ ] Boost por recency (documentos más recientes)
- [ ] Boost por popularity (documentos más accedidos)
- [ ] Boost por área/type según contexto del query
- [ ] Retornar resultados rerankeados

**Edge Cases:**
- [Metadata faltante]: Usar scores de similitud solamente
- [Reranking fallido]: Retornar resultados originales
- [Timeout en reranking]: Retornar resultados originales

#### Feature 6: Context assembly

**Description**: Ensamblar respuesta con chunks relevantes

**Requirements:**
- [ ] Ensamblar respuesta con chunks relevantes del documento
- [ ] Incluir snippets con contexto alrededor del match
- [ ] Incluir metadata del documento (título, autor, fecha)
- [ ] Incluir confidence scores para cada resultado
- [ ] Retornar respuesta estructurada para agentes de IA

**Edge Cases:**
- [Chunks muy largos]: Truncar a 200 tokens por snippet
- [Contexto insuficiente]: Expandir snippet con chunks adyacentes
- [Multiple matches]: Combinar chunks relevantes

### Non-Functional Requirements

#### Performance
- [ ] Response time < 300ms para search_documents
- [ ] Response time < 100ms para embedding generation
- [ ] Response time < 200ms para vector search en Qdrant
- [ ] Support 100 concurrent users
- [ ] Handle 10M embeddings with <100ms latency

#### Security
- [ ] ACL filtering en cada búsqueda
- [ ] Organization filtering en cada búsqueda
- [ ] TLS 1.3 for all connections
- [ ] Rate limiting para búsquedas
- [ ] Audit logging para búsquedas sensibles

#### Accessibility
- [ ] Clear error messages in Spanish and English
- [ ] Support for both Spanish and English queries
- [ ] Consistent API structure
- [ ] Well-documented schema for AI agents

#### Scalability
- [ ] Handle 10,000 documents per organization
- [ ] Support 100 organizations in MVP
- [ ] 10M embeddings in Qdrant with <100ms latency
- [ ] 1M documents in PostgreSQL with <100ms queries

## 4. Technical Considerations

### Dependencies

**Internal:**
- Qdrant vector database (embeddings, semantic search)
- sentence-transformers model (embedding generation)
- FastMCP framework (MCP integration)

**External:**
- Hugging Face (model downloads for sentence-transformers)

### Constraints

**Technical:**
- Python 3.11+ required
- sentence-transformers for embeddings
- Qdrant 1.7+ for vector search
- Embedding dimension: 384 (initial), 768 (future)

**Business:**
- MVP timeline: 6 weeks
- Target: 50 active users/week in first month
- Budget: Self-hosted infrastructure only (no cloud APIs)

**Timeline:**
- Week 1: Embedding generation básica
- Week 2: Integración con Qdrant
- Week 3: ACL y organization filtering
- Week 4: Reranking y context assembly
- Week 5: Performance optimization
- Week 6: Testing y deployment

### Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Performance de ACL filtering en búsquedas | High | Pre-indexar ACL en Qdrant, usar colecciones por organización |
| Latencia de generación de embeddings | Medium | Usar batch processing, cache de embeddings, modelo ligero inicial |
| Consistencia entre PostgreSQL y Qdrant | High | Implementar eventual consistency con retries |
| Calidad de embeddings en español | Medium | Evaluar múltiples modelos, usar modelo entrenado en español |

## 5. Semantic Search Details

### Dependencies for Semantic Search

**Internal:**
- Qdrant vector database (embeddings, semantic search)
- sentence-transformers model (embedding generation)
- FastMCP framework (MCP integration)

**External:**
- Hugging Face (model downloads for sentence-transformers)

### Constraints for Semantic Search

**Technical:**
- Python 3.11+ required
- sentence-transformers for embeddings
- Qdrant 1.7+ for vector search
- Embedding dimension: 384 (initial), 768 (future)

**Business:**
- MVP timeline: 6 weeks
- Target: 50 active users/week in first month
- Budget: Self-hosted infrastructure only (no cloud APIs)

### Risks for Semantic Search

| Risk | Impact | Mitigation |
|---|---|---|
| Performance de ACL filtering en búsquedas | High | Pre-indexar ACL en Qdrant, usar colecciones por organización |
| Latencia de generación de embeddings | Medium | Usar batch processing, cache de embeddings, modelo ligero inicial |
| Consistencia entre PostgreSQL y Qdrant | High | Implementar transacciones o eventual consistency con retries |

### Success Metrics for Semantic Search

**Primary Metrics:**

- **Búsquedas exitosas**: 80% de búsquedas retornan resultados relevantes
- **Latencia promedio**: <300ms para operaciones de búsqueda típicas

**Secondary Metrics:**

- **Documentos creados**: 1000 documentos indexados en primer mes
- **Error rate**: <5% error rate para operaciones de búsqueda

**Tracking for Semantic Search:**

**Analytics Events:**
- `document_searched`: Cuando se busca documentos via MCP

**Dashboard:**
- Gráficos: Tendencia de búsquedas, distribución por área/tipo
- Alertas: Error rate >5%, latencia >1s

## 6. Out of Scope

### Features NOT Included

- **Hybrid search (keyword + semantic)**: MVP es solo semantic search, hybrid es fase futura
- **Query expansion**: MVP no expande queries con sinónimos
- **Personalización de resultados**: MVP no personaliza basado en historial de usuario
- **Knowledge graph**: Conexiones semánticas entre documentos es fase futura
- **Filtros avanzados (fecha, autor)**: MVP tiene filtros básicos, filtros avanzados es fase futura
- **Autocompletado de queries**: MVP no tiene autocompletado
- **Búsqueda por imagen/audio**: MVP es solo texto, multimodal es fase futura

### Future Considerations

- **Enhancement 1**: Hybrid search (keyword + semantic) con BM25
- **Enhancement 2**: Query expansion con sinónimos y related terms
- **Enhancement 3**: Personalización de resultados basado en historial
- **Enhancement 4**: Knowledge graph para conectar documentos relacionados
- **Enhancement 5**: Filtros avanzados (fecha, autor, área, type)
- **Enhancement 6**: Autocompletado de queries con sugerencias
- **Enhancement 7**: Búsqueda multimodal (imagen, audio)
- **Enhancement 8**: Reranking con ML models avanzados

## 7. Success Metrics & KPIs

### Primary Metrics

- **Búsquedas exitosas**: 80% de búsquedas retornan resultados relevantes
- **Latencia promedio**: <300ms para operaciones de búsqueda típicas
- **Documentos indexados**: 1000 documentos indexados en primer mes
- **Satisfacción**: NPS > 40 para experiencia de búsqueda

### Secondary Metrics

- **Error rate**: <5% error rate para operaciones de búsqueda
- **Queries por usuario**: 10 queries/semana por usuario activo
- **Resultados clickeados**: 60% de búsquedas resultan en clic en documento

### Tracking

**Analytics Events:**
- `document_searched`: Cuando se busca documentos via MCP
- `search_result_clicked`: Cuando se hace clic en un resultado
- `search_no_results`: Cuando búsqueda no retorna resultados
- `search_latency`: Latencia de cada búsqueda

**Dashboard:**
- KPIs: Búsquedas totales, latencia promedio, error rate, resultados clickeados
- Gráficos: Tendencia de búsquedas, distribución por área/tipo, latencia over time
- Alertas: Error rate >5%, latencia >1s, búsquedas <10/semana

## 8. Testing Strategy

### User Acceptance Testing

**Test Scenarios:**
- Buscar documentos con query en español y verificar resultados relevantes
- Buscar documentos con query en inglés y verificar resultados relevantes
- Buscar documentos y verificar que se filtran por organización
- Buscar documentos y verificar que se filtran por ACL
- Buscar con query muy largo y verificar que se trunca correctamente

**Test Users:**
- 5 desarrolladores usando Claude Desktop
- 3 product managers usando Cursor
- 2 líderes técnicos usando VS Code

**Success Criteria:**
- 80% de test scenarios exitosos
- 90% de test users reportan experiencia positiva
- Latencia promedio <300ms en todos escenarios

### Technical Testing

**Unit Tests:**
- Coverage target >80% para core logic
- Tests para embedding generation
- Tests para vector search en Qdrant
- Tests para ACL filtering
- Tests para organization filtering

**Integration Tests:**
- Tests de integración con Qdrant
- Tests de integración con sentence-transformers
- Tests de integración con FastMCP
- Tests de integración con document management

**Performance Tests:**
- Load testing: 100 concurrent users
- Stress testing: 10M embeddings in Qdrant
- Latency testing: <300ms para 95% de requests
- Endurance testing: 24 hours continuous load

## 9. Related Documents

- [System Architecture](../engineering/definition/mvp-system-architecture.md)
- [Product Brief](../product/strategy/product-brief.md)
- [PRD-0002: MVP Document Management via MCP](./mvp-document-management-mcp.md)
- [ADR-0004: Qdrant Vector Database](../engineering/decision/qdrant-vector-database.md)
- [ADR-0005: sentence-transformers](../engineering/decision/sentence-transformers-embeddings.md)

## Approval

**Product Lead**: ___________________ Date: ________

**Engineering Lead**: ___________________ Date: ________

**Design Lead**: ___________________ Date: ________
