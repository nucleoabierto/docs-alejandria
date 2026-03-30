---
id: PRD-0003
area: "product"
type: "definition"
title: "MVP ACL and Permissions"
author: "product-team"
status: "draft"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["PROD-STRAT-0001", "ENG-DEF-0001", "PRD-0001", "PRD-0002", "ADR-0003"]
---

Documento de requerimientos de producto que define el sistema de ACL (Access Control List) y permisos granulares para el MVP de Alejandria. Contiene análisis del problema de control de acceso, modelo de ACL, validación de permisos, user stories específicos de ACL, requerimientos funcionales y consideraciones técnicas.

---

# PRD-0003: MVP ACL and Permissions

**Status**: Draft  
**Date**: 2026-04-24  
**Author**: Product Team  
**Reviewer**: Engineering Lead  
**Target Release**: MVP v1.0

## 1. Overview

### Problem Statement

Sabemos que controlar quién puede acceder a qué documentos es fundamental para cualquier sistema de conocimiento organizacional. Sin permisos granulares, no podemos garantizar que la información sensible permanezca privada, ni podemos habilitar colaboración efectiva entre equipos.

Entendemos que el MVP necesita un modelo de ACL simple pero efectivo que te permita:
- Controlar visibilidad de documentos (private/public)
- Definir permisos por defecto para miembros de organización
- Otorgar permisos específicos a usuarios individuales
- Validar permisos en cada operación MCP
- Escalar a roles avanzados en fases futuras

### Success Metrics

- **Configuraciones de ACL exitosas**: 90% de documentos creados tienen ACL válida
- **Violaciones de permisos**: 0 violaciones de permisos en MVP
- **Satisfacción**: NPS > 40 para experiencia de configuración de ACL
- **Adopción de documentos privados**: 30% de documentos creados son privados

### Goals

- **Goal 1**: Implementar modelo de ACL con visibility (private/public)
- **Goal 2**: Habilitar permisos por defecto para miembros de organización
- **Goal 3**: Permitir permisos específicos por usuario individual
- **Goal 4**: Validar permisos en todas las operaciones MCP

## 2. User Analysis

### Target Users

**Primary Users:**
- Tú, como desarrollador, que quieres crear documentos privados para tu trabajo personal
- Tú, como Product Manager, que quieres compartir documentos públicos con tu equipo
- Tú, como líder técnico, que quieres otorgar acceso específico a ciertos usuarios

**Secondary Users:**
- Operaciones que configuran permisos para nuevos miembros
- Administradores de sistemas que auditan accesos

### User Stories

#### User Story 1: Control de acceso privado

**Como** owner de un documento  
**quiero** marcar un documento como privado  
**para que** solo yo pueda acceder a él

**Acceptance Criteria:**
- [ ] El owner puede crear documentos con visibility "private"
- [ ] Documentos privados solo son accesibles por el owner
- [ ] El owner puede otorgar permisos específicos a otros usuarios
- [ ] Los permisos se respetan en todas las operaciones (get, search, update, delete)

#### User Story 2: Control de acceso público

**Como** miembro de organización  
**quiero** crear documentos públicos  
**para que** todos los miembros de la organización puedan acceder

**Acceptance Criteria:**
- [ ] El usuario puede crear documentos con visibility "public"
- [ ] Documentos públicos son accesibles por todos los miembros de la organización
- [ ] Los permisos por defecto se aplican a todos los miembros
- [ ] Los permisos específicos pueden override permisos por defecto

#### User Story 3: Permisos específicos

**Como** owner de un documento  
**quiero** otorgar permisos específicos a un usuario  
**para que** ese usuario pueda leer o editar el documento

**Acceptance Criteria:**
- [ ] El owner puede agregar permisos específicos en ACL
- [ ] Los permisos específicos pueden ser "read", "write", "admin"
- [ ] Los permisos específicos override permisos por defecto
- [ ] Los permisos específicos se validan en cada operación

#### User Story 4: Validación de permisos en búsqueda

**Como** usuario  
**quiero** buscar documentos  
**para que** solo vea resultados que tengo permiso de leer

**Acceptance Criteria:**
- [ ] La búsqueda filtra resultados por ACL del usuario
- [ ] Documentos privados sin permiso no aparecen en resultados
- [ ] Documentos públicos de otras organizaciones no aparecen en resultados
- [ ] La latencia de búsqueda con filtros ACL es <500ms

## 3. ACL Model

### ACL Structure

**Schema JSONB en PostgreSQL:**

```json
{
  "visibility": "private | public",
  "permissions": {
    "default": "none | read | write | admin",
    "users": {
      "user-id-1": "read | write | admin",
      "user-id-2": "read | write | admin"
    }
  }
}
```

### Visibility Types

**private:**
- Solo accesible por owner y usuarios en lista específica
- Requiere permisos default o específicos
- Útil para documentos sensibles o trabajo personal

**public:**
- Accesible por todos los miembros de la organización
- Permisos por defecto se aplican a todos los miembros
- Útil para documentos de colaboración y conocimiento compartido

### Permission Levels

**none:**
- Sin acceso al documento
- No puede leer ni editar
- Útil para documentos que serán compartidos selectivamente

**read:**
- Puede leer el documento
- No puede editar ni eliminar
- Útil para revisión y consumo de conocimiento

**write:**
- Puede leer y editar el documento
- No puede eliminar ni cambiar ACL
- Útil para colaboración activa

**admin:**
- Puede leer, editar, eliminar y cambiar ACL
- Tiene control completo del documento
- Útil para delegación de ownership

### Validation Rules

**Reglas de validación:**

1. **Visibility private**: Requiere permisos default o específicos
2. **Visibility public**: Permite acceso a todos los miembros de la organización
3. **Permisos específicos**: Override permisos por defecto
4. **Owner siempre tiene acceso**: El owner del documento siempre tiene permisos admin

## 4. Functional Requirements

### ACL Validation

#### Feature 1: Validar ACL al crear documento

**Description**: Validar estructura de ACL al crear documento

**Requirements:**
- [ ] Validar que visibility es "private" o "public"
- [ ] Validar que permissions.default es válido si visibility es "private"
- [ ] Validar que permissions.users contiene user_ids válidos
- [ ] Validar que permissions de usuarios son "read", "write" o "admin"
- [ ] Rechazar ACL inválida con error descriptivo

**Edge Cases:**
- [ACL inválido]: Rechazar con error "invalid ACL configuration"
- [Visibility sin permisos]: Rechazar private sin permisos default o específicos
- [User_id inválido]: Rechazar con error "user not found"

#### Feature 2: Validar permisos de escritura

**Description**: Verificar que usuario tiene permisos de escritura antes de editar

**Requirements:**
- [ ] Verificar que usuario tiene permisos de escritura (owner o write/admin en ACL)
- [ ] Validar ACL antes de permitir edición
- [ ] Preservar historial de ACL en versiones
- [ ] Rechazar edición sin permisos con error "write permission denied"

**Edge Cases:**
- [Sin permisos]: Rechazar con error "write permission denied"
- [Documento deleted]: Rechazar con error "document is deleted"
- [Concurrent edits]: Implementar optimistic locking

#### Feature 3: Validar permisos de lectura

**Description**: Verificar que usuario tiene permisos de lectura antes de retornar documento

**Requirements:**
- [ ] Aplicar lógica: owner OR (visibility=public AND member) OR (specific permission)
- [ ] Verificar que usuario tiene permisos de lectura
- [ ] Incluir información de ACL para contexto
- [ ] Rechazar lectura sin permisos con error "read permission denied"

**Edge Cases:**
- [Sin permisos]: Rechazar con error "read permission denied"
- [Documento no encontrado]: Rechazar con error "document not found"
- [Documento deleted]: Rechazar con error "document is deleted"

#### Feature 4: Filtrar resultados por ACL en búsqueda

**Description**: Filtrar resultados de búsqueda por ACL del usuario

**Requirements:**
- [ ] Filtrar resultados por ACL del usuario
- [ ] Aplicar filtros en Qdrant usando metadata ACL
- [ ] Retornar solo documentos que usuario tiene permiso de leer
- [ ] Mantener latencia <500ms con filtros ACL

**Edge Cases:**
- [ACL restrictivo]: Filtrar resultados que usuario no tiene permiso de leer
- [Sin resultados]: Retornar lista vacía con mensaje "no results found"
- [Query muy largo]: Truncar a 512 tokens máximo

#### Feature 5: Validar permisos de eliminación

**Description**: Verificar que usuario es el owner antes de eliminar

**Requirements:**
- [ ] Verificar que usuario es el owner del documento
- [ ] Solo el owner puede eliminar documentos
- [ ] Rechazar eliminación sin permisos con error "only owner can delete document"
- [ ] Permitir eliminación con confirmación

**Edge Cases:**
- [No es owner]: Rechazar con error "only owner can delete document"
- [Documento ya deleted]: Retornar success (idempotent)
- [Documento en uso por otros]: Permitir eliminación pero notificar impacto

#### Feature 6: Validar permisos en listado

**Description**: Filtrar documentos por ACL del usuario al listar

**Requirements:**
- [ ] Filtrar documentos por ACL del usuario
- [ ] Retornar solo documentos que usuario tiene permiso de leer
- [ ] Aplicar paginación con limit y offset
- [ ] Incluir metadata resumida (sin contenido completo)

**Edge Cases:**
- [Sin documentos]: Retornar lista vacía
- [Filtros inválidos]: Ignorar filtros inválidos y aplicar válidos
- [Offset beyond results]: Retornar lista vacía

### Non-Functional Requirements

#### Performance
- [ ] Response time < 100ms para validación de ACL
- [ ] Response time < 500ms para búsqueda con filtros ACL
- [ ] Response time < 100ms para validación de permisos
- [ ] Support 1000 concurrent users
- [ ] Handle 1000 documents per organization

#### Security
- [ ] ACL validation en cada operación
- [ ] Audit logging para cambios de ACL
- [ ] Data encryption at rest (PostgreSQL)
- [ ] TLS 1.3 for all connections
- [ ] Rate limiting para operaciones sensibles

#### Accessibility
- [ ] Clear error messages in Spanish and English
- [ ] Consistent ACL structure across all documents
- [ ] Well-documented ACL model for AI agents

#### Scalability
- [ ] Handle 10,000 documents per organization
- [ ] Support 100 organizations in MVP
- [ ] 10M documents across all organizations
- [ ] Pre-indexar ACL en Qdrant para performance

## 5. Technical Considerations

### Dependencies

**Internal:**
- PostgreSQL database (documents con ACL JSONB)
- Qdrant vector database (metadata ACL para filtros)
- FastMCP framework (MCP integration)

**External:**
- Docker (deployment containers)

### Constraints

**Technical:**
- Python 3.11+ required
- FastMCP framework for MCP integration
- PostgreSQL 15+ for JSONB support
- Qdrant 1.7+ for metadata filtering

**Business:**
- MVP timeline: 6 weeks
- Target: 90% de documentos con ACL válida
- Budget: Self-hosted infrastructure only

**Timeline:**
- Week 1: ACL schema y validación básica
- Week 2: ACL validation en operaciones MCP
- Week 3: ACL filtering en búsqueda
- Week 4: Optimización de performance
- Week 5: Testing y optimización
- Week 6: Documentation y deployment

### Risks

| Risk | Impact | Mitigation |
|---|---|---|
| Performance de ACL filtering en búsquedas | High | Pre-indexar ACL en Qdrant, usar colecciones por organización |
| Complejidad de gestión de permisos granulares | Medium | Simplificar ACL en MVP (private/public + permisos por defecto) |
| Consistencia entre PostgreSQL y Qdrant ACL | High | Implementar eventual consistency con retries |
| Escalabilidad de ACL a gran escala | Medium | Diseñar schema con partitioning por organización |

## 6. Out of Scope

### Features NOT Included

- **Roles avanzados**: MVP solo tiene owner y members, roles avanzados (admin, editor, viewer) es fase futura
- **Herencia de permisos**: MVP no tiene herencia de permisos por carpeta o proyecto
- **Permisos temporales**: MVP no tiene permisos con expiración
- **ACL por grupo**: MVP solo tiene ACL por usuario individual
- **Audit logs detallados**: MVP tiene logging básico, audit logs detallados es fase futura
- **Approval workflows**: MVP no tiene workflows de aprobación para cambios de ACL

### Future Considerations

- **Enhancement 1**: Roles avanzados (admin, editor, viewer) con permisos predefinidos
- **Enhancement 2**: Herencia de permisos por carpeta o proyecto
- **Enhancement 3**: Permisos temporales con expiración
- **Enhancement 4**: ACL por grupo (equipos, departamentos)
- **Enhancement 5**: Audit logs detallados para compliance
- **Enhancement 6**: Approval workflows para cambios de ACL sensibles

## 7. Success Metrics & KPIs

### Primary Metrics

- **Configuraciones de ACL exitosas**: 90% de documentos creados tienen ACL válida
- **Violaciones de permisos**: 0 violaciones de permisos en MVP
- **Satisfacción**: NPS > 40 para experiencia de configuración de ACL
- **Adopción de documentos privados**: 30% de documentos creados son privados

### Secondary Metrics

- **Latencia de validación ACL**: <100ms para 95% de operaciones
- **Error rate de ACL**: <1% error rate para operaciones de ACL
- **Documentos públicos vs privados**: 70% públicos, 30% privados

### Tracking

**Analytics Events:**
- `acl_created`: Cuando se configura ACL en documento
- `acl_updated`: Cuando se actualiza ACL de documento
- `permission_denied`: Cuando se deniega acceso por falta de permisos
- `private_document_created`: Cuando se crea documento privado
- `public_document_created`: Cuando se crea documento público

**Dashboard:**
- KPIs: ACL válidas, violaciones de permisos, distribución private/public
- Gráficos: Tendencia de configuraciones de ACL, distribución por tipo
- Alertas: Violaciones de permisos, error rate ACL >5%

## 8. Testing Strategy

### User Acceptance Testing

**Test Scenarios:**
- Crear documento privado y verificar que solo owner puede acceder
- Crear documento público y verificar que todos en organización pueden acceder
- Otorgar permisos específicos y verificar que se respetan
- Buscar documentos y verificar que solo aparecen resultados con permiso
- Intentar editar documento sin permisos y verificar que se deniega

**Test Users:**
- 5 desarrolladores creando documentos privados
- 3 product managers creando documentos públicos
- 2 líderes técnicos otorgando permisos específicos

**Success Criteria:**
- 100% de test scenarios exitosos
- 0 violaciones de permisos detectadas
- Latencia de validación ACL <100ms

### Technical Testing

**Unit Tests:**
- Coverage target >90% para ACL logic
- Tests para validación de ACL structure
- Tests para validación de permisos (read, write, admin)
- Tests para filtering por ACL
- Tests para edge cases

**Integration Tests:**
- Tests de integración con PostgreSQL (ACL JSONB)
- Tests de integración con Qdrant (metadata filtering)
- Tests de integración con document management
- Tests de integración con FastMCP

**Performance Tests:**
- Load testing: 1000 concurrent users con ACL
- Stress testing: 10,000 documents con ACL filtering
- Latency testing: <100ms para 95% de ACL validations

## 9. Related Documents

- [System Architecture](../engineering/definition/mvp-system-architecture.md)
- [Product Brief](../product/strategy/product-brief.md)
- [PRD-0001: Organization and User Management](./mvp-organization-user-management.md)
- [PRD-0002: MVP Document Management via MCP](./mvp-document-management-mcp.md)
- [ADR-0003: PostgreSQL](../engineering/decision/postgresql-relational-database.md)

## Approval

**Product Lead**: ___________________ Date: ________

**Engineering Lead**: ___________________ Date: ________

**Design Lead**: ___________________ Date: ________
