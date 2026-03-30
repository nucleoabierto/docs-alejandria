---
id: PRD-0002
area: "product"
type: "definition"
title: "MVP Document Management via MCP - Core Features"
author: "product-team"
status: "active"
version: "1.1"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["PROD-STRAT-0001", "ENG-DEF-0001", "PRD-0001", "PRD-0003", "PRD-0004", "ADR-0002", "ADR-0003", "ADR-0004", "ADR-0005"]
---

Documento de requerimientos de producto que define las operaciones core de gestión de documentos via MCP para el MVP de Alejandria. Contiene análisis del problema de creación y edición de documentos por agentes de IA, user stories para CRUD operations, requerimientos funcionales para create, read, update, delete y list documents, consideraciones técnicas y métricas de éxito. ACL y búsqueda semántica están documentados en PRD-0003 y PRD-0004 respectivamente.

---

# PRD-0002: MVP Document Management via MCP - Core Features

**Status**: Active
**Date**: 2026-04-24
**Author**: Product Team
**Reviewer**: Engineering Lead
**Target Release**: MVP v1.0

## 1. Overview

### Problem Statement

Sabemos que los agentes de IA (Claude, Cursor, GitHub Copilot) necesitan crear, editar y acceder a documentos organizacionales para capturar el conocimiento que generan durante tus conversaciones y tareas. Actualmente, no existe una forma nativa para que los agentes de IA escriban directamente en un sistema de conocimiento organizacional con permisos granulares y estructura de versionado.

El problema es que el conocimiento valioso generado por los agentes de IA se pierde en conversaciones transitorias, sin ser capturado en un sistema persistente y searchable. Esto perpetúa el problema de pérdida de memoria institucional que Alejandria busca resolver.

### Success Metrics

- **Adopción**: 50 usuarios activos/semana creando documentos via MCP en el primer mes
- **Engagement**: 60% de usuarios que crean un documento retornan para crear otro en la misma semana
- **Completitud**: 70% de documentos creados via MCP tienen título, contenido y metadata completa
- **Satisfacción**: NPS > 40 para experiencia de creación de documentos via MCP

### Goals

- **Goal 1**: Permitir que agentes de IA creen documentos organizacionales con estructura y metadata apropiada
- **Goal 2**: Habilitar edición de documentos existentes con control de versiones
- **Goal 3**: Proveer recuperación de documentos para que agentes de IA accedan a contenido específico
- **Goal 4**: Habilitar eliminación y listado de documentos para gestión completa

**Nota**: ACL y búsqueda semántica están documentadas en PRD-0003 y PRD-0004 respectivamente.

## 2. User Analysis

### Target Users

**Primary Users:**
- Tú, como desarrollador que usa Claude Desktop/CLI, Cursor, o VS Code con MCP
- Tú, como Product Manager que usa agentes de IA para documentar decisiones
- Tú, como líder técnico que captura conocimiento arquitectónico via agentes de IA

**Secondary Users:**
- Agentes de IA (Claude, Cursor, GitHub Copilot) que actúan en nombre de usuarios humanos
- Operaciones que documentan procesos y procedimientos via agentes de IA

### User Stories

#### User Story 1: Crear documento nuevo

**Como** desarrollador usando Claude Desktop  
**quiero** crear un nuevo documento organizacional via MCP  
**para que** capture el conocimiento técnico generado durante nuestra conversación

**Acceptance Criteria:**
- [ ] Tú puedes invocar `create_document` via MCP con título, contenido, organización y ACL
- [ ] El documento se guarda en PostgreSQL con metadata completa
- [ ] Se genera embeddings del contenido y se indexa en Qdrant
- [ ] El documento tiene versión inicial (1.0) con contenido completo
- [ ] Tú recibes confirmación con document_id
- [ ] El ACL se respeta (private/public, permisos por defecto, permisos específicos)

#### User Story 2: Editar documento existente

**Como** product manager usando Cursor  
**quiero** editar un documento existente via MCP  
**para que** actualice información desactualizada sin perder historial

**Acceptance Criteria:**
- [ ] Tú puedes invocar `update_document` via MCP con document_id y cambios
- [ ] El sistema verifica tus permisos de escritura antes de permitir edición
- [ ] Se crea una nueva versión del documento con contenido completo
- [ ] La versión anterior se preserva en historial
- [ ] Se regeneran embeddings del contenido actualizado
- [ ] Tú recibes confirmación con nueva versión

#### User Story 3: Recuperar documento específico

**Como** desarrollador usando VS Code  
**quiero** recuperar un documento específico por ID  
**para que** pueda leer el contenido completo

**Acceptance Criteria:**
- [ ] Tú puedes invocar `get_document` via MCP con document_id
- [ ] El sistema verifica tus permisos de lectura antes de retornar documento
- [ ] Tú recibes documento con contenido, metadata y versión actual
- [ ] Tú puedes solicitar versiones específicas si es necesario

#### User Story 4: Eliminar documento

**Como** owner de un documento  
**quiero** eliminar un documento que ya no es relevante  
**para que** mantenga el sistema limpio y organizado

**Acceptance Criteria:**
- [ ] Tú puedes invocar `delete_document` via MCP con document_id
- [ ] El sistema verifica que eres el owner antes de permitir eliminación
- [ ] El documento se marca como deleted (soft delete) en PostgreSQL
- [ ] Los embeddings se eliminan de Qdrant
- [ ] Tú recibes confirmación de eliminación

### User Journey

**Flujo de creación de documento:**
1. Tú conversas con un agente de IA (Claude, Cursor)
2. El agente genera conocimiento valioso (decisión técnica, arquitectura, proceso)
3. Solicitas al agente: "Guarda esto como un documento en Alejandria"
4. El agente invoca `create_document` via MCP con título, contenido, organización
5. El sistema valida ACL y permisos (ver PRD-0003)
6. El sistema guarda el documento en PostgreSQL
7. El sistema genera embeddings y indexa en Qdrant (ver PRD-0004)
8. El sistema retorna document_id al agente
9. El agente te confirma: "Documento guardado exitosamente con ID: xyz"

## 3. Functional Requirements

### Core Features

#### Feature 1: Crear documento (create_document)

**Description**: Endpoint MCP para crear nuevos documentos organizacionales

**Requirements:**
- [ ] Aceptar parámetros: title, content, organization_id, area, type, acl
- [ ] Validar que organization_id pertenece al usuario
- [ ] Validar que area y type son valores permitidos
- [ ] Generar document_id único
- [ ] Crear registro en documents con metadata
- [ ] Crear versión inicial (1.0) en document_versions con contenido completo
- [ ] Aplicar ACL (visibility, permissions)
- [ ] Generar embeddings del contenido
- [ ] Chunkear contenido en unidades semánticas
- [ ] Indexar embeddings en Qdrant con metadata
- [ ] Retornar document_id y confirmación

**Edge Cases:**
- [Content vacío]: Rechazar con error "content cannot be empty"
- [Título duplicado]: Permitir (no es único en organización)
- [Organización inválida]: Rechazar con error "organization not found or access denied"
- [ACL inválido]: Rechazar con error "invalid ACL configuration"

#### Feature 2: Editar documento (update_document)

**Description**: Endpoint MCP para editar documentos existentes con control de versiones

**Requirements:**
- [ ] Aceptar parámetros: document_id, changes (patch operations)
- [ ] Validar que usuario tiene permisos de escritura (owner o en ACL)
- [ ] Crear nueva versión en document_versions con contenido completo
- [ ] Actualizar current_version_id en documents
- [ ] Actualizar metadata modificada (title, description, status)
- [ ] Regenerar embeddings del contenido actualizado
- [ ] Re-indexar embeddings en Qdrant
- [ ] Retornar nueva versión y confirmación

**Edge Cases:**
- [Documento no encontrado]: Rechazar con error "document not found"
- [Sin permisos]: Rechazar con error "write permission denied"
- [Concurrent edits]: Implementar optimistic locking para evitar conflictos
- [Documento deleted]: Rechazar con error "document is deleted"

#### Feature 3: Recuperar documento (get_document)

**Description**: Endpoint MCP para recuperar documento específico

**Requirements:**
- [ ] Aceptar parámetros: document_id, version_id (opcional)
- [ ] Validar que usuario tiene permisos de lectura
- [ ] Si version_id es especificado, retornar esa versión específica
- [ ] Si version_id no es especificado, retornar versión actual
- [ ] Retornar contenido, metadata y versión
- [ ] Incluir información de ACL para contexto

**Edge Cases:**
- [Documento no encontrado]: Rechazar con error "document not found"
- [Sin permisos]: Rechazar con error "read permission denied"
- [Versión no encontrada]: Rechazar con error "version not found"
- [Documento deleted]: Rechazar con error "document is deleted"

#### Feature 4: Eliminar documento (delete_document)

**Description**: Endpoint MCP para eliminar documentos (soft delete)

**Requirements:**
- [ ] Aceptar parámetros: document_id
- [ ] Validar que usuario es el owner del documento
- [ ] Marcar documento como deleted en PostgreSQL (soft delete)
- [ ] Eliminar embeddings de Qdrant
- [ ] Preservar historial de versiones en PostgreSQL
- [ ] Retornar confirmación de eliminación

**Edge Cases:**
- [Documento no encontrado]: Rechazar con error "document not found"
- [No es owner]: Rechazar con error "only owner can delete document"
- [Documento ya deleted]: Retornar success (idempotent)
- [Documento en uso por otros]: Permitir eliminación pero notificar impacto

#### Feature 5: Listar documentos (list_documents)

**Description**: Endpoint MCP para listar documentos de una organización

**Requirements:**
- [ ] Aceptar parámetros: organization_id, filters (area, type, status), limit, offset
- [ ] Validar que organization_id pertenece al usuario
- [ ] Filtrar documentos por ACL del usuario
- [ ] Aplicar filtros adicionales si son especificados
- [ ] Paginar resultados con limit y offset
- [ ] Retornar lista con metadata resumida (sin contenido completo)

**Edge Cases:**
- [Organización inválida]: Rechazar con error "organization not found or access denied"
- [Sin documentos]: Retornar lista vacía
- [Filtros inválidos]: Ignorar filtros inválidos y aplicar válidos
- [Offset beyond results]: Retornar lista vacía

### Non-Functional Requirements

#### Performance
- [ ] Response time < 500ms para create_document
- [ ] Response time < 300ms para update_document
- [ ] Response time < 100ms para get_document
- [ ] Response time < 100ms para delete_document
- [ ] Response time < 100ms para list_documents
- [ ] Support 100 concurrent users
- [ ] Handle 1000 documents per organization

#### Security
- [ ] Authentication required (API keys)
- [ ] Authorization per document (ACL)
- [ ] Data encryption at rest (PostgreSQL)
- [ ] TLS 1.3 for all connections
- [ ] API keys encrypted in database
- [ ] Audit logging for all operations

#### Accessibility
- [ ] Clear error messages in Spanish and English
- [ ] Consistent API structure across all endpoints
- [ ] Well-documented schema for AI agents
- [ ] Support for both Spanish and English queries

#### Scalability
- [ ] Handle 10,000 documents per organization
- [ ] Support 100 organizations in MVP
- [ ] 10M embeddings in Qdrant with <100ms latency
- [ ] 1M documents in PostgreSQL with <100ms queries

**Nota**: Para detalles completos sobre ACL, ver PRD-0003. Para detalles sobre búsqueda semántica, ver PRD-0004.

## 4. Technical Considerations

### Dependencies

**Internal:**
- PostgreSQL database (documents, organizations, users, ACL)
- Qdrant vector database (embeddings, semantic search)
- sentence-transformers model (embedding generation)
- FastMCP framework (MCP integration)

**External:**
- Hugging Face (model downloads for sentence-transformers)
- Docker (deployment containers)

### Constraints

**Technical:**
- Python 3.11+ required
- FastMCP framework for MCP integration
- sentence-transformers for embeddings
- PostgreSQL 15+ for relational data
- Qdrant 1.7+ for vector search

**Business:**
- MVP timeline: 6 weeks
- Target: 50 active users/week in first month
- Budget: Self-hosted infrastructure only (no cloud APIs)

**Timeline:**
- Week 1-2: Core CRUD operations (create, read, update, delete)
- Week 3: ACL implementation and testing
- Week 4: Semantic search integration
- Week 5: Performance optimization and testing
- Week 6: Documentation and deployment

### Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Performance de ACL filtering en búsquedas | High | Pre-indexar ACL en Qdrant, usar colecciones por organización |
| Complejidad de gestión de permisos granulares | Medium | Simplificar ACL en MVP (private/public + permisos por defecto) |
| Adopción de FastMCP vs SDK oficial de MCP | Medium | Monitorear adopción de FastMCP, tener plan de migración |
| Latencia de generación de embeddings | Medium | Usar batch processing, cache de embeddings, modelo ligero inicial |
| Consistencia entre PostgreSQL y Qdrant | High | Implementar transacciones o eventual consistency con retries |

## 5. Design Requirements

### UI/UX Requirements

**Design System:**
- No UI requerido para MVP (solo MCP endpoints)
- Schema automático generado por FastMCP
- Documentación automática para agentes de IA

**User Flow:**
- Conversación natural con agente de IA
- Agente invoca endpoints MCP transparentemente
- Usuario recibe confirmación en conversación

**Wireframes:**
- No aplicable (MVP sin UI)

### Brand Requirements

**Colors:**
- No aplicable (MVP sin UI)

**Typography:**
- No aplicable (MVP sin UI)

**Imagery:**
- No aplicable (MVP sin UI)

## 6. Out of Scope

### Features NOT Included

- **Web UI para gestión de documentos**: MVP es solo MCP, web UI es fase futura
- **Ingestión automática desde fuentes externas**: GitHub, Jira, Slack, Notion son fase futura
- **Knowledge graph**: Conexiones semánticas entre documentos es fase futura
- **Real-time collaboration**: Edición simultánea por múltiples usuarios es fase futura
- **Advanced ACL**: Roles y permisos complejos son fase futura
- **OAuth 2.0 + SSO**: MVP usa API keys simples, OAuth es fase futura
- **PII detection y redaction**: Compliance con GDPR es fase futura

### Future Considerations

- **Enhancement 1**: Web UI para gestión visual de documentos
- **Enhancement 2**: Ingestión automática desde GitHub, Jira, Slack, Notion
- **Enhancement 3**: Knowledge graph para conectar decisiones estratégicas con implementación técnica
- **Enhancement 4**: Real-time collaboration con WebSocket
- **Enhancement 5**: Advanced ACL con roles y permisos granulares
- **Enhancement 6**: OAuth 2.0 + SSO (Google, GitHub, Okta)
- **Enhancement 7**: PII detection y redaction para compliance con GDPR

## 7. Success Metrics & KPIs

### Primary Metrics

- **Adopción**: 50 usuarios activos/semana creando documentos via MCP
- **Engagement**: 60% de usuarios retornan semanalmente para crear documentos
- **Completitud**: 70% de documentos tienen metadata completa
- **Satisfacción**: NPS > 40 para experiencia de creación de documentos

### Secondary Metrics

- **Documentos creados**: 1000 documentos indexados en primer mes
- **Latencia promedio**: <300ms para operaciones MCP típicas
- **Error rate**: <5% error rate para operaciones MCP

### Tracking

**Analytics Events:**
- `document_created`: Cuando se crea un documento via MCP
- `document_updated`: Cuando se edita un documento via MCP
- `document_retrieved`: Cuando se recupera un documento via MCP
- `document_deleted`: Cuando se elimina un documento via MCP
- `document_listed`: Cuando se listan documentos via MCP

**Dashboard:**
- KPIs: Usuarios activos, documentos creados, latencia, error rate
- Gráficos: Tendencia de creación de documentos, distribución por área/tipo
- Alertas: Error rate >5%, latencia >1s, documentos creados <10/semana

---

## Related Documents

- [System Architecture](../engineering/definition/mvp-system-architecture.md)
- [Product Brief](../product/strategy/product-brief.md)
- [PRD-0001: Organization and User Management](./mvp-organization-user-management.md)
- [PRD-0003: MVP ACL and Permissions](./mvp-acl-permissions.md)
- [PRD-0004: MVP Semantic Search](./mvp-semantic-search.md)
- [ADR-0002: Python + FastMCP](../engineering/decision/python-fastmcp-backend.md)
- [ADR-0004: Qdrant Vector Database](../engineering/decision/qdrant-vector-database.md)
- [ADR-0003: PostgreSQL](../engineering/decision/postgresql-relational-database.md)
- [ADR-0005: sentence-transformers](../engineering/decision/sentence-transformers-embeddings.md)

## Approval

**Product Lead**: ___________________ Date: ________

**Engineering Lead**: ___________________ Date: ________

**Design Lead**: ___________________ Date: ________
