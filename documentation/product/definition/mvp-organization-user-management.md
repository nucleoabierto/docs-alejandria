---
id: PRD-0001
area: "product"
type: "definition"
title: "MVP Organization and User Management"
author: "product-team"
status: "draft"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["PROD-STRAT-0001", "ENG-DEF-0001", "ADR-0006", "ADR-0002", "ENG-REF-0004"]
---

Documento de requerimientos de producto que define el feature de gestión de organizaciones y usuarios para el MVP de Alejandria. Contiene análisis del problema de multi-tenancy y aislamiento de datos, user stories, requerimientos funcionales y no funcionales, consideraciones técnicas y métricas de éxito.

---

# PRD-0001: MVP Organization and User Management

**Status**: Draft  
**Date**: 2026-04-24  
**Author**: Product Team  
**Reviewer**: Engineering Lead  
**Target Release**: MVP v1.0

## 1. Overview

### Problem Statement

Sabemos que gestionar múltiples organizaciones con aislamiento completo de datos puede ser complejo. Pero para que Alejandria funcione, necesitamos asegurar que cada organización solo acceda a sus propios documentos y usuarios. Sin un sistema robusto de gestión de organizaciones y usuarios, no podemos implementar ACL granular, ni escalar a múltiples clientes (B2B multi-tenancy).

Entendemos que el MVP necesita un modelo simple pero efectivo que te permita:
- Aislamiento de datos por organización
- Gestión directa de miembros dentro de organizaciones (sin invitaciones)
- Autenticación simple (API keys) para el MVP
- Base para ACL granular por documento
- Escalabilidad futura a enterprise (SSO, roles avanzados, invitaciones)

### Success Metrics

- **Organizaciones activas**: 10 organizaciones activas usando Alejandria diariamente en el primer mes
- **Usuarios por organización**: Promedio de 5-10 usuarios por organización
- **Onboarding**: Tiempo de onboarding de organización <5 minutos
- **Satisfacción**: NPS > 40 para experiencia de creación de organizaciones

### Goals

- **Goal 1**: Permitir creación de organizaciones personales y de equipo
- **Goal 2**: Habilitar gestión directa de miembros dentro de organizaciones (sin invitaciones)
- **Goal 3**: Implementar autenticación simple con API keys para el MVP
- **Goal 4**: Proveer base para ACL granular por documento y usuario

## 2. User Analysis

### Target Users

**Primary Users:**
- Tú, como desarrollador independiente, que quieres crear una organización personal para tus proyectos
- Tú, como Product Manager, que quieres crear una organización de equipo para tu departamento
- Tú, como líder técnico, que quieres crear una organización enterprise para tu compañía

**Secondary Users:**
- Administradores de sistemas que gestionan usuarios y organizaciones
- Operaciones que configuran acceso para nuevos miembros del equipo

### User Stories

#### User Story 1: Crear organización personal

**Como** desarrollador independiente  
**quiero** crear una organización personal  
**para que** pueda gestionar mis propios proyectos y documentos

**Acceptance Criteria:**
- [ ] El usuario puede crear una organización de tipo "personal"
- [ ] El usuario es automáticamente el owner de la organización
- [ ] La organización se crea con el usuario como único miembro
- [ ] El usuario recibe organization_id y confirmation
- [ ] La organización puede tener un nombre personalizable

#### User Story 2: Crear organización de equipo

**Como** product manager  
**quiero** crear una organización de equipo  
**para que** mi departamento pueda colaborar en documentos compartidos

**Acceptance Criteria:**
- [ ] El usuario puede crear una organización de tipo "team"
- [ ] El usuario es automáticamente el owner de la organización
- [ ] La organización se crea con el usuario como único miembro inicial
- [ ] El usuario puede invitar otros miembros a la organización
- [ ] La organización puede tener un nombre personalizable

#### User Story 3: Agregar miembro directamente a organización

**Como** owner de organización  
**quiero** agregar un miembro directamente a mi organización  
**para que** puedan colaborar en documentos compartidos

**Acceptance Criteria:**
- [ ] El owner puede agregar usuarios existentes por email a su organización
- [ ] El usuario se agrega inmediatamente como miembro de la organización
- [ ] El owner puede ver lista de miembros de la organización
- [ ] El owner puede remover miembros de la organización

#### User Story 4: Generar API key

**Como** usuario  
**quiero** generar una API key  
**para que** pueda autenticarme con Alejandria via MCP

**Acceptance Criteria:**
- [ ] El usuario puede generar una API key única
- [ ] La API key se muestra solo una vez (al crear)
- [ ] El usuario puede revocar y regenerar API keys
- [ ] La API key está encriptada en base de datos
- [ ] La API key se usa para autenticación en endpoints MCP


### User Journey

**Flujo de creación de organización:**
1. Tú te registras en Alejandria (creas tu cuenta de usuario)
2. El sistema genera tu API key
3. Solicitas al agente: "Crea una organización personal llamada 'Mis Proyectos'"
4. El agente invoca `create_organization` via MCP con tipo "personal"
5. El sistema crea la organización contigo como owner
6. El sistema retorna el organization_id al agente
7. El agente te confirma: "Organización creada exitosamente con ID: xyz"

**Flujo de agregación de miembro:**
1. Como owner, solicitas al agente: "Agrega a juan@example.com a mi organización"
2. El agente invoca `add_member` via API web con email y organization_id
3. El sistema valida que eres el owner
4. El sistema verifica que el usuario existe en el sistema
5. El sistema agrega al usuario como miembro de la organización
6. Recibes confirmación de que el miembro fue agregado

## 3. Functional Requirements

### Core Features

#### Feature 1: Crear usuario (create_user)

**Description**: Endpoint para crear nuevos usuarios en Alejandria (ver ENG-REF-0004)

**Requirements:**
- [ ] Aceptar parámetros: email, name, password
- [ ] Validar que email es único
- [ ] Validar formato de email
- [ ] Validar contraseña (mínimo 8 caracteres, máximo 128)
- [ ] Generar user_id único (UUID)
- [ ] Generar API key única (formato sk-...)
- [ ] Encriptar API key en base de datos
- [ ] Crear organización personal automáticamente (type="personal")
- [ ] Retornar user_id, email, name, api_key, organization_id, created_at

**Edge Cases:**
- [Email duplicado]: Rechazar con error "email already exists" (400)
- [Email inválido]: Rechazar con error "invalid email format" (400)
- [Nombre vacío]: Rechazar con error "name cannot be empty" (400)
- [Contraseña muy corta]: Rechazar con error "password too short" (400)

**API Reference**: POST /v1/users (ENG-REF-0004)
**Schema Reference**: users table (ENG-REF-0007)

#### Feature 2: Crear organización (create_organization)

**Description**: Endpoint para crear nuevas organizaciones (ver ENG-REF-0005)

**Requirements:**
- [ ] Aceptar parámetros: name, type (enum: personal/team/enterprise)
- [ ] Validar que usuario está autenticado (JWT token)
- [ ] Validar que type es uno de: personal, team, enterprise
- [ ] Generar organization_id único (UUID)
- [ ] Crear organización con usuario autenticado como owner
- [ ] Retornar organization_id, name, type, owner_id, created_at

**Edge Cases:**
- [Tipo inválido]: Rechazar con error "invalid organization type" (400)
- [Nombre duplicado]: Permitir (no es único global)
- [No autenticado]: Rechazar con error "authentication required" (401)

**API Reference**: POST /v1/organizations (ENG-REF-0005)
**Schema Reference**: organizations table (ENG-REF-0007)

#### Feature 3: Agregar miembro (add_member)

**Description**: Endpoint para agregar usuarios existentes directamente a una organización (ver ENG-REF-0005)

**Requirements:**
- [ ] Aceptar parámetros: organization_id (path), email (body)
- [ ] Validar que usuario autenticado es owner de la organización
- [ ] Validar que email no es ya miembro de la organización
- [ ] Verificar que el usuario existe en el sistema (busca por email en users table)
- [ ] Agregar usuario como miembro con role="member"
- [ ] Retornar organization_id, user_id, email, role, joined_at

**Edge Cases:**
- [No es owner]: Rechazar con error "only owner can add members" (403)
- [Email ya miembro]: Rechazar con error "user is already a member" (400)
- [Usuario no existe]: Rechazar con error "user not found" (404)
- [Organización no encontrada]: Rechazar con error "organization not found" (404)

**API Reference**: POST /v1/organizations/{organization_id}/members (ENG-REF-0005)
**Schema Reference**: organization_members table (ENG-REF-0007)


#### Feature 4: Generar API key (regenerate_api_key)

**Description**: Endpoint para regenerar la API key del usuario autenticado (ver ENG-REF-0004)

**Requirements:**
- [ ] Validar que usuario está autenticado (JWT token)
- [ ] Generar nueva API key única (formato sk-...)
- [ ] Encriptar nueva API key en base de datos
- [ ] Revocar API key anterior (reemplazar en users table)
- [ ] Retornar user_id, api_key, created_at

**Edge Cases:**
- [No autenticado]: Rechazar con error "authentication required" (401)

**API Reference**: POST /v1/users/me/api-key (ENG-REF-0004)
**Schema Reference**: users table (ENG-REF-0007)

#### Feature 5: Listar miembros (list_members)

**Description**: Endpoint para listar miembros de una organización (ver ENG-REF-0005)

**Requirements:**
- [ ] Aceptar parámetros: organization_id (path)
- [ ] Validar que usuario autenticado es miembro de la organización
- [ ] Retornar lista de miembros con user_id, name, email, role, joined_at
- [ ] Incluir total de miembros

**Edge Cases:**
- [No es miembro]: Rechazar con error "access denied" (403)
- [Organización no encontrada]: Rechazar con error "organization not found" (404)

**API Reference**: GET /v1/organizations/{organization_id}/members (ENG-REF-0005)
**Schema Reference**: organization_members table (ENG-REF-0007)

#### Feature 6: Remover miembro (remove_member)

**Description**: Endpoint para remover miembro de una organización (ver ENG-REF-0005)

**Requirements:**
- [ ] Aceptar parámetros: organization_id (path), user_id (path)
- [ ] Validar que usuario autenticado es owner de la organización
- [ ] Validar que user_id no es el owner (no puede removerse a sí mismo)
- [ ] Remover usuario de la organización (DELETE de organization_members)
- [ ] Retornar organization_id, user_id, removed_at

**Edge Cases:**
- [No es owner]: Rechazar con error "only owner can remove members" (403)
- [Remover owner]: Rechazar con error "cannot remove owner" (400)
- [Usuario no es miembro]: Retornar success (idempotent)

**API Reference**: DELETE /v1/organizations/{organization_id}/members/{user_id} (ENG-REF-0005)
**Schema Reference**: organization_members table (ENG-REF-0007)

### Non-Functional Requirements

#### Performance
- [ ] Response time < 200ms para create_user
- [ ] Response time < 100ms para create_organization
- [ ] Response time < 300ms para add_member
- [ ] Response time < 100ms para list_members
- [ ] Response time < 100ms para remove_member
- [ ] Response time < 100ms para regenerate_api_key
- [ ] Support 1000 concurrent users
- [ ] Handle 100 organizations in MVP

#### Security
- [ ] Authentication required (API keys only for MVP)
- [ ] API keys encrypted in database
- [ ] API key regeneration support
- [ ] TLS 1.3 for all connections
- [ ] Email validation for member addition
- [ ] Rate limiting for organization creation
- [ ] Audit logging for all operations

#### Accessibility
- [ ] Clear error messages in Spanish and English
- [ ] Consistent API structure across all endpoints
- [ ] Well-documented schema for AI agents

#### Scalability
- [ ] Handle 100 users per organization
- [ ] Support 100 organizations in MVP
- [ ] 100k users total in MVP
- [ ] 10M documents across all organizations

## 4. Technical Considerations

### Dependencies

**Internal:**
- PostgreSQL database (users, organizations, organization_members)
- FastMCP framework (MCP integration)

**External:**
- Docker (deployment containers)

### Constraints

**Technical:**
- Python 3.11+ required
- FastMCP framework for MCP integration
- PostgreSQL 15+ for relational data

**Business:**
- MVP timeline: 6 weeks
- Target: 10 active organizations in first month
- Budget: Self-hosted infrastructure only

**Timeline:**
- Week 1: Core user and organization CRUD
- Week 2: Direct member management (add/remove)
- Week 3: API key management
- Week 4: Integration with document management
- Week 5: Testing and optimization
- Week 6: Documentation and deployment

### Risks

| Risk | Impact | Mitigation |
|---|---|---|
| API key security | High | Encriptar API keys en database, rotación periódica |
| Scalability de multi-tenancy | Medium | Diseñar schema con partitioning por organización |
| Complejidad de ACL con organizaciones | High | Simplificar ACL en MVP (organización + documento), roles avanzados en futuro |
| Gestión manual de miembros en MVP | Low | Implementar sistema de invitaciones en fase futura cuando sea necesario |

## 5. Design Requirements

### UI/UX Requirements

**Design System:**
- UI mínima para MVP (solo endpoints MCP)
- CLI simple para gestión de usuarios y organizaciones (opcional)
- Schema automático generado por FastMCP

**User Flow:**
- Registro inicial con email y nombre
- Generación automática de API key
- Creación de organización personal automática
- Agregación directa de miembros por email

**Wireframes:**
- No aplicable (MVP con foco en MCP)

### Brand Requirements

**Colors:**
- No aplicable (MVP con foco en MCP)

**Typography:**
- No aplicable (MVP con foco en MCP)

**Imagery:**
- No aplicable (MVP con foco en MCP)

## 6. Out of Scope

### Features NOT Included

- **Web UI para gestión de usuarios y organizaciones**: MVP es solo MCP, web UI es fase futura
- **JWT tokens para autenticación web**: MVP usa solo API keys para MCP, JWT es fase futura cuando se implemente web UI
- **OAuth 2.0 + SSO**: MVP usa API keys simples, OAuth es fase futura
- **Roles avanzados**: MVP solo tiene owner y members, roles avanzados es fase futura
- **Organizaciones enterprise con features especiales**: MVP soporta personal/team, enterprise features es fase futura
- **Sistema de invitaciones por email**: MVP usa agregación directa de miembros, invitaciones es fase futura
- **User profiles avatares**: MVP solo tiene email y nombre, avatares es fase futura

### Future Considerations

- **Enhancement 1**: Web UI para gestión visual de usuarios y organizaciones
- **Enhancement 2**: JWT tokens para autenticación web (cuando se implemente web UI)
- **Enhancement 3**: OAuth 2.0 + SSO (Google, GitHub, Okta)
- **Enhancement 4**: Roles avanzados (admin, editor, viewer)
- **Enhancement 5**: Organizaciones enterprise con SSO y features especiales
- **Enhancement 6**: Sistema de invitaciones por email con aceptación/rechazo
- **Enhancement 7**: User profiles con avatares y bio
- **Enhancement 8**: Audit logs detallados para compliance

## 7. Success Metrics & KPIs

### Primary Metrics

- **Organizaciones activas**: 10 organizaciones activas usando Alejandria diariamente en primer mes
- **Usuarios por organización**: Promedio de 5-10 usuarios por organización
- **Onboarding**: Tiempo de onboarding de organización <5 minutos
- **Satisfacción**: NPS > 40 para experiencia de creación de organizaciones

### Secondary Metrics

- **API keys activas**: 90% de usuarios tienen API key activa
- **Tasa de crecimiento**: 20% crecimiento mensual de organizaciones
- **Retención**: 70% de organizaciones activas después de 30 días

### Tracking

**Analytics Events:**
- `user_created`: Cuando se crea un usuario
- `organization_created`: Cuando se crea una organización
- `member_added`: Cuando se agrega un miembro a organización
- `member_removed`: Cuando se remueve un miembro de organización
- `api_key_generated`: Cuando se genera una API key

**Dashboard:**
- KPIs: Organizaciones activas, usuarios totales, miembros por organización, API keys
- Gráficos: Tendencia de creación de organizaciones, distribución por tipo
- Alertas: Organizaciones creadas <2/semana

## 8. Testing Strategy

### User Acceptance Testing

**Test Scenarios:**
- Crear usuario y verificar que se genera tu API key
- Crear organización personal y verificar que eres el owner
- Crear organización de equipo y verificar que se pueden agregar miembros
- Agregar miembro y verificar que se agrega correctamente
- Remover miembro y verificar que se remueve correctamente
- Generar API key y verificar que funciona para autenticación

**Test Users:**
- 5 desarrolladores creando organizaciones personales
- 3 product managers creando organizaciones de equipo
- 2 líderes técnicos invitando miembros

**Success Criteria:**
- 80% de test scenarios exitosos
- 90% de test users reportan experiencia positiva
- Tiempo de onboarding <5 minutos

### Technical Testing

**Unit Tests:**
- Coverage target >80% para core logic
- Tests para cada endpoint de usuarios y organizaciones
- Tests para validación de email
- Tests para generación de API keys
- Tests para gestión de miembros (add, list, remove)

**Integration Tests:**
- Tests de integración con PostgreSQL
- Tests de integración con FastMCP
- Tests de integración con document management (ACL por organización y por usuario)

**Performance Tests:**
- Load testing: 1000 concurrent users
- Stress testing: 100 organizations con 100 members cada una
- Latency testing: <200ms para 95% de requests

## 9. Launch Plan

### Phased Rollout

**Phase 1: Internal testing (Week 5)**
- Deploy en staging environment
- Testing interno por equipo de ingeniería
- Bug fixes y optimizaciones

**Phase 2: Beta users (Week 6)**
- Invitar 10 organizaciones beta
- Recolección de feedback
- Iteración rápida basada en feedback

**Phase 3: Full launch (Week 7)**
- Deploy en producción
- Anuncio a comunidad de usuarios
- Monitoreo intensivo primeras 48 horas

### Communication Plan

**Internal:**
- Demo del feature al equipo completo
- Documentación técnica para ingeniería
- Guía de troubleshooting para ops

**External:**
- Anuncio en Slack/Discord de comunidad
- Tutorial de cómo crear organizaciones
- Guía de cómo gestionar miembros (add, list, remove)

**Documentation:**
- Actualizar system architecture con multi-tenancy
- Crear guía de usuario para organizaciones
- Documentar API key management

## 10. Post-Launch

### Monitoring

**Health Checks:**
- Health check para PostgreSQL connection
- Health check para FastMCP server

**Alerts:**
- API key generation failures
- Organization creation failures

**Dashboards:**
- KPIs de organizaciones y usuarios
- Métricas de performance (latencia, throughput)
- Métricas de errores y failures

### Iteration Plan

**Feedback Collection:**
- Encuestas NPS mensuales a usuarios
- Interviews cualitativas con owners de organizaciones
- Análisis de logs de errores y timeouts
- Monitoreo de métricas de adopción

**Next Version:**
- Agregar roles avanzados (admin, editor, viewer)
- Implementar OAuth 2.0 + SSO
- Agregar web UI para gestión visual
- Implementar audit logs detallados

**Deprecation:**
- No features deprecate en MVP
- Plan de deprecation para API keys simples en favor de OAuth 2.0

---

## Related Documents

- [System Architecture](../engineering/definition/mvp-system-architecture.md)
- [Product Brief](../product/strategy/product-brief.md)
- [PRD-0002: MVP Document Management via MCP](./mvp-document-management-mcp.md)
- [ADR-0003: PostgreSQL](../engineering/decision/postgresql-relational-database.md)
- [ENG-REF-0007: Database Schema - Core Tables](../engineering/reference/schema-core-tables.md)

## Approval

**Product Lead**: ___________________ Date: ________

**Engineering Lead**: ___________________ Date: ________

**Design Lead**: ___________________ Date: ________
