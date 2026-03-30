---
id: PROD-PRD-0001
area: "product"
type: "prd"
title: "PRD-001: Core Infrastructure & Multi-tenant Architecture"
author: "product-team"
status: "planned"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["PROD-STRAT-0001", "PROD-STRAT-0004", "ENG-DEC-0001", "ENG-DEC-0002", "ENG-DEF-0001"]
---

# Product Requirements Document - PRD-001: Core Infrastructure & Multi-tenant Architecture

**Version**: 1.0  
**Date**: 2026-03-30  
**Status**: Planned  
**Owner**: Backend Team  
**Stakeholders**: Engineering, DevOps, Security  

## 1. Executive Summary

### Product Vision
Infraestructura core escalable con búsqueda híbrida y aislamiento multi-tenant que soporta el crecimiento de 10 a 500 organizaciones sin cambios arquitectónicos.

### Problem Statement
Las soluciones de documentación actuales no están diseñadas para escala multi-tenant, causando data leakage, performance degradation y complejidad de gestión al crecer.

### Success Metrics (6 semanas)
- **Setup time**: < 30 minutos para nueva organización
- **Search latency**: < 100ms (p95) bajo carga multi-tenant
- **Data isolation**: 0% data leakage entre organizaciones
- **Scalability**: Soporte 100+ organizaciones sin degradación

---

## 2. Target Users & Personas

### Primary Personas

#### 1. **Backend Developer**
- **Contexto**: Implementa infraestructura core, configura bases de datos
- **Pain Points**: "¿Cómo aseguro aislamiento entre organizaciones?", "¿Cómo optimizo búsquedas multi-tenant?"
- **Goals**: Setup robusto, performance óptimo, seguridad garantizada
- **Usage Frequency**: Diaria durante desarrollo, semanal en producción

#### 2. **DevOps Engineer**
- **Contexto**: Gestiona deployment, monitoring, escalabilidad
- **Pain Points**: "¿Cómo monitoreo multi-tenant?", "¿Cómo escalo sin downtime?"
- **Goals**: Operaciones estables, métricas claras, automatización
- **Usage Frequency**: Diaria

#### 3. **Security Engineer**
- **Contexto**: Valida aislamiento de datos, compliance
- **Pain Points**: "¿Cómo prevengo data leakage?", "¿Cómo audito accesos?"
- **Goals**: Seguridad garantizada, audit trails completos
- **Usage Frequency**: Semanal

### Secondary Personas

#### 4. **Engineering Manager**
- **Usage**: Revisa métricas de performance, planea capacidad
- **Frequency**: Semanal

#### 5. **System Administrator**
- **Usage**: Gestiona organizaciones, usuarios, permisos
- **Frequency**: Diaria

---

## 3. Core Features & Requirements

### 3.1 Database Architecture

#### 3.1.1 PostgreSQL 16+ with Apache AGE

**User Story**: Como backend developer, quiero una base de datos híbrida (relacional + grafos) para manejar documentos y relaciones complejas.

**Requirements**:
- ✅ **MUST**: PostgreSQL 16+ con Apache AGE extension activada
- ✅ **MUST**: Schema multi-tenant con organization_id en todas las tablas
- ✅ **MUST**: Índices optimizados para búsqueda híbrida
- ✅ **SHOULD**: Particionamiento por organización para escalabilidad
- ✅ **SHOULD**: Row Level Security (RLS) para aislamiento adicional

**Acceptance Criteria**:
- Queries incluyen automáticamente filtro `organization_id`
- Performance < 50ms para queries de documentos
- Soporte para grafos de hasta 10M edges por organización

#### 3.1.2 Qdrant Vector Database

**User Story**: Como backend developer, quiero búsqueda vectorial escalable para soportar embeddings multi-tenant.

**Requirements**:
- ✅ **MUST**: Collections separadas por organización
- ✅ **MUST**: Metadata filtering por organization_id
- ✅ **MUST**: Soporte para múltiples embedding models
- ✅ **SHOULD**: Sharding automático por organización
- ✅ **SHOULD**: Backup y recovery por organización

**Acceptance Criteria**:
- Búsqueda semántica < 100ms (p95) con 100K vectors
- Aislamiento garantizado entre organizaciones
- Reindexing sin downtime

### 3.2 Multi-tenant Architecture

#### 3.2.1 Organization Isolation

**User Story**: Como security engineer, quiero aislamiento completo de datos entre organizaciones.

**Requirements**:
- ✅ **MUST**: Organization-based data isolation en todas las capas
- ✅ **MUST**: Separación física de datos en Qdrant
- ✅ **MUST**: Validación de organization_id en todos los endpoints
- ✅ **SHOULD**: Database-level isolation con schemas separados
- ✅ **SHOULD**: Network isolation opcional para enterprise

**Acceptance Criteria**:
- 0% data leakage entre organizaciones en testing
- Performance degradation < 5% vs single-tenant
- Audit trails completos para accesos entre organizaciones

#### 3.2.2 Role-Based Access Control

**User Story**: Como system administrator, quiero controlar permisos por organización.

**Requirements**:
- ✅ **MUST**: Roles: Admin, Editor, Viewer por organización
- ✅ **MUST**: Organization-scoped permissions
- ✅ **MUST**: JWT tokens con organization_id
- ✅ **SHOULD**: Document-level permissions (Fase 2)
- ✅ **SHOULD**: SSO integration por organización

**Acceptance Criteria**:
- Usuario solo accede a datos de su organización
- Role validation en cada API call
- Token refresh mantiene organization context

### 3.3 Search Infrastructure

#### 3.3.1 Embeddings Pipeline

**User Story**: Como backend developer, quiero procesamiento eficiente de embeddings para múltiples modelos.

**Requirements**:
- ✅ **MUST**: BGE-M3 para texto general (384 dimensions)
- ✅ **MUST**: Jina Code para código y documentación técnica
- ✅ **MUST**: SPLADE-v3 para búsqueda sparse
- ✅ **MUST**: Batch processing con Oban background jobs
- ✅ **SHOULD**: Caching de embeddings por documento
- ✅ **SHOULD**: Re-processing incremental solo para cambios

**Acceptance Criteria**:
- Embedding generation < 1s por documento
- Batch processing de 100 docs en < 30s
- Re-indexing de cambios en < 5s

#### 3.3.2 Hybrid Search Engine

**User Story**: Como backend developer, quiero búsqueda híbrida que combine semántica y full-text.

**Requirements**:
- ✅ **MUST**: Parallel search en Qdrant (semantic) + PostgreSQL (full-text)
- ✅ **MUST**: Reciprocal Rank Fusion (RRF) para merge de resultados
- ✅ **MUST**: Organization filtering en ambos motores
- ✅ **SHOULD**: Query expansion con synonyms
- ✅ **SHOULD**: Re-ranking basado en user feedback

**Acceptance Criteria**:
- Search latency < 100ms (p95)
- Top-10 resultados relevantes en > 85% de queries
- Soporte para queries complejas con filtros

### 3.4 API Core

#### 3.4.1 REST API Endpoints

**User Story**: Como backend developer, quiero endpoints RESTful para todas las operaciones core.

**Requirements**:
- ✅ **MUST**: Endpoints de búsqueda y recuperación
- ✅ **MUST**: Document CRUD operations
- ✅ **MUST**: Organization management
- ✅ **MUST**: User management y permissions
- ✅ **SHOULD**: Rate limiting por organización
- ✅ **SHOULD**: API versioning (v1, v2)

**Acceptance Criteria**:
- API response time < 200ms promedio
- Error rate < 1% bajo carga
- OpenAPI specification completa

#### 3.4.2 Authentication & Security

**User Story**: Como security engineer, quiero autenticación segura con contexto multi-tenant.

**Requirements**:
- ✅ **MUST**: JWT tokens con organization_id
- ✅ **MUST**: OAuth 2.0 (GitHub, Google) por organización
- ✅ **MUST**: Rate limiting (100 req/min por usuario)
- ✅ **SHOULD**: MFA para admin users
- ✅ **SHOULD**: Audit logging de accesos

**Acceptance Criteria**:
- Token validation < 10ms
- Secure logout con token revocation
- Audit trails completos para compliance

---

## 4. Technical Architecture

### 4.1 Database Schema

```sql
-- Organizations
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Users (multi-tenant)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  email VARCHAR(255) NOT NULL,
  role VARCHAR(50) NOT NULL CHECK (role IN ('admin', 'editor', 'viewer')),
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(organization_id, email)
);

-- Documents (multi-tenant)
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id),
  title VARCHAR(500) NOT NULL,
  content TEXT,
  metadata JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  author_id UUID REFERENCES users(id)
);

-- Enable RLS for multi-tenant isolation
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;
```

### 4.2 API Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │    │   MCP Server    │    │   Webhooks      │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│                API Gateway (Phoenix)                       │
│  - Authentication (JWT)                                    │
│  - Rate Limiting                                            │
│  - Organization Context                                    │
└───────────────────────┬───────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ PostgreSQL  │  │   Qdrant    │  │    Redis    │
│ + Apache AGE│  │ Vector DB   │  │   Cache     │
└─────────────┘  └─────────────┘  └─────────────┘
```

### 4.3 Multi-tenant Data Flow

```
1. User Request with JWT
   ├─> Decode token (user_id, organization_id, role)
   ├─> Validate organization exists
   └─> Set organization context

2. Database Query
   ├─> Add WHERE organization_id = ? automatically
   ├─> Apply RLS policies
   └─> Return organization-scoped results

3. Vector Search
   ├─> Filter Qdrant collection by organization
   ├─> Search within organization vectors
   └─> Merge with PostgreSQL results

4. Response
   ├─> Filter results based on user role
   ├─> Add organization metadata
   └─> Return scoped response
```

---

## 5. Non-Functional Requirements

### 5.1 Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| Search latency (p50) | < 50ms | Telemetry |
| Search latency (p95) | < 100ms | Telemetry |
| Search latency (p99) | < 200ms | Telemetry |
| Database query time | < 30ms | PostgreSQL logs |
| Embedding generation | < 1s per document | Oban metrics |
| API response time | < 200ms promedio | Phoenix metrics |
| Multi-tenant overhead | < 5% vs single-tenant | Load testing |

### 5.2 Scalability

| Dimension | MVP (6 meses) | Year 1 | Year 2 |
|-----------|---------------|--------|--------|
| Organizations | 10 | 100 | 500 |
| Users per org | 10 | 50 | 100 |
| Documents per org | 10K | 100K | 1M |
| Vectors per org | 50K | 500K | 5M |
| Searches/day per org | 100 | 1K | 10K |

### 5.3 Availability

- **Uptime**: 99.9% (8.76 horas downtime/año)
- **Backup**: Daily snapshots por organización, 30 días retention
- **Disaster Recovery**: RTO < 4 horas, RPO < 1 hora
- **Multi-tenant failover**: Aislamiento de fallos por organización

### 5.4 Security

- **Authentication**: JWT + OAuth 2.0 con organization context
- **Authorization**: RBAC con organization isolation
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **Data Isolation**: Row Level Security + organization filtering
- **API Security**: Rate limiting (100 req/min por usuario)
- **Audit**: Completo logging de accesos por organización

### 5.5 Observability

- **Logging**: Structured logs con organization_id y correlation IDs
- **Metrics**: Prometheus + Grafana con dashboards por organización
- **Tracing**: Distributed tracing con OpenTelemetry
- **Alerting**: Organization-specific alerting

---

## 6. Implementation Plan

### 6.1 Development Phases

#### Week 1-2: Database & Search Core
- **Day 1-3**: PostgreSQL setup con Apache AGE y RLS
- **Day 4-6**: Qdrant deployment y multi-tenant collections
- **Day 7-10**: Embeddings pipeline (BGE-M3, Jina Code, SPLADE-v3)
- **Day 11-14**: Hybrid search implementation con RRF

#### Week 3: Multi-tenant Architecture
- **Day 15-17**: Organization management y user roles
- **Day 18-19**: JWT authentication con organization context
- **Day 20-21**: API endpoints con organization filtering

#### Week 4: API Core & Security
- **Day 22-24**: REST API endpoints (documents, users, orgs)
- **Day 25-26**: Rate limiting y security middleware
- **Day 27-28**: Integration testing y security validation

### 6.2 Technical Acceptance Criteria

#### Database Tests
```elixir
# Test multi-tenant isolation
test "users cannot access other organizations" do
  org1 = insert!(:organization)
  org2 = insert!(:organization)
  user1 = insert!(:user, organization: org1)
  user2 = insert!(:user, organization: org2)
  
  doc1 = insert!(:document, organization: org1)
  
  # User from org2 cannot access org1 documents
  conn = conn_with_user(user2)
  get(conn, "/api/v1/documents/#{doc1.id}")
  assert response(conn, 404)
end
```

#### Search Performance Tests
```elixir
test "search latency under 100ms with 100K documents" do
  # Setup 100K documents for organization
  insert_bulk_documents(100_000, @organization)
  
  {time, _result} = :timer.tc(fn ->
    Search.search(@query, @organization.id)
  end)
  
  assert time / 1000 < 100  # < 100ms
end
```

### 6.3 Risk Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Multi-tenant data leakage | Critical | Low | RLS + organization filtering + automated tests |
| Performance degradation at scale | High | Medium | Horizontal scaling + caching + query optimization |
| Database migration complexity | Medium | Medium | Incremental migrations + backward compatibility |
| Qdrant scaling issues | High | Low | Alternative vector DB plan + sharding strategy |

---

## 7. Success Criteria & Metrics

### 7.1 MVP Success (Week 6)

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Multi-tenant setup time | < 30 minutos | Automated testing |
| Search latency (p95) | < 100ms | Telemetry + load testing |
| Data isolation | 0% leakage | Security audit + penetration testing |
| API availability | > 99.5% | Monitoring |
| Organizations supported | 10+ | Production metrics |
| Database performance | < 30ms queries | PostgreSQL metrics |

### 7.2 Technical Validation

| Metric | Target | Test Method |
|--------|--------|-------------|
| Embedding quality | > 85% relevance | Test set evaluation |
| Vector search accuracy | > 90% top-5 | Manual validation |
| Multi-tenant overhead | < 5% vs single-tenant | Load testing |
| Security compliance | 100% RLS coverage | Automated scans |
| API error rate | < 1% | Integration testing |

### 7.3 Scalability Validation

| Test | Target | Success Criteria |
|------|--------|------------------|
| Load test (100 concurrent users) | < 200ms response | All requests succeed |
| Scale test (100 organizations) | No performance degradation | < 5% overhead |
| Stress test (1M documents) | Search < 150ms | Linear performance |
| Failover test | < 30s recovery | Automatic failover |

---

## 8. Out of Scope (PRD-001)

### Explicitly NOT in this PRD

- ❌ **MCP Server implementation** → PRD-002
- ❌ **GitHub/Slack/Shortcut integrations** → PRD-003
- ❌ **Performance optimization avanzada** → PRD-004
- ❌ **UI/Web interface** → Fase 2
- ❌ **Knowledge graph visual** → Fase 2
- ❌ **Advanced analytics** → Fase 3
- ❌ **Mobile apps** → Fase 3
- ❌ **AI-generated content** → Fase 2

### Dependencies on Other PRDs

- **PRD-002 (MCP Server)**: Depende de API endpoints y search core
- **PRD-003 (Integration Hub)**: Depende de document storage y webhooks
- **PRD-004 (Performance)**: Optimiza components de este PRD

---

## 9. Dependencies & Risks

### 9.1 Technical Dependencies

| Dependency | Risk Level | Mitigation |
|------------|------------|------------|
| PostgreSQL 16+ with AGE | Medium | Docker container, backup plan con PostgreSQL vanilla |
| Qdrant stability | Medium | Abstraer cliente, plan de migración a Milvus/Weaviate |
| Embedding models performance | Low | Local deployment, fallback a OpenAI embeddings |
| Phoenix framework maturity | Low | Well-established, good documentation |

### 9.2 Multi-tenant Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Data leakage between orgs | Critical | Low | RLS + organization filtering + automated security tests |
| Performance degradation | High | Medium | Horizontal scaling + caching + query optimization |
| Complex migrations | Medium | Medium | Incremental migrations + backward compatibility |
| Tenant resource contention | Medium | Low | Resource limits per organization |

### 9.3 Implementation Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Search quality insufficient | Medium | Low | Test set evaluation, model swapping capability |
| Database schema complexity | Medium | Medium | Start simple, evolve based on usage |
| Vector database scaling | High | Low | Sharding strategy, monitoring setup |

---

## 10. Testing Strategy

### 10.1 Unit Tests (Integrated in Development)

**Coverage Requirements**:
- > 90% coverage en backend core modules
- 100% coverage en authentication y authorization
- 100% coverage en multi-tenant filtering

**Key Test Scenarios**:
```elixir
# Organization isolation
test "multi-tenant data isolation" do
  # Verify no cross-organization data access
end

# Search performance
test "search latency under load" do
  # Verify < 100ms response time
end

# Authentication
test "JWT token validation with organization context" do
  # Verify proper token handling
end
```

### 10.2 Integration Tests

**API Integration**:
- All endpoints con organization context
- Error handling y edge cases
- Rate limiting y security policies

**Database Integration**:
- Multi-tenant queries con RLS
- Vector search operations
- Transaction handling

**External Service Integration**:
- Embedding generation reliability
- Qdrant connection resilience
- Redis caching operations

### 10.3 Load Testing

**Performance Targets**:
- 1000 concurrent searches
- 10K searches/hour sustained
- Multi-tenant load (10 orgs simultaneously)

**Test Scenarios**:
```bash
# Search load test
k6 run --vus 1000 --duration 5m search_test.js

# Multi-tenant load test
k6 run --vus 500 --duration 10m multi_tenant_test.js
```

### 10.4 Security Testing

**Automated Security**:
- OWASP Top 10 scanning
- SQL injection testing
- Authentication bypass attempts
- Data leakage verification

**Manual Security**:
- Penetration testing por organización
- Access control validation
- Audit trail verification

---

## 11. Technical Specifications

### 11.1 Database Schema Details

```sql
-- Core multi-tenant schema
CREATE EXTENSION IF NOT EXISTS "age";
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Organizations table
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  domain VARCHAR(255),
  settings JSONB DEFAULT '{}',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Users with organization context
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  email VARCHAR(255) NOT NULL,
  name VARCHAR(255),
  role VARCHAR(50) NOT NULL CHECK (role IN ('admin', 'editor', 'viewer')),
  last_sign_in_at TIMESTAMP WITH TIME ZONE,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(organization_id, email)
);

-- Documents with full-text search
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
  title VARCHAR(500) NOT NULL,
  content TEXT,
  content_vector vector(384), -- For BGE-M3
  metadata JSONB DEFAULT '{}',
  author_id UUID REFERENCES users(id),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Full-text search index
CREATE INDEX documents_search_idx ON documents USING gin(
  to_tsvector('english', title || ' ' || COALESCE(content, ''))
);

-- Multi-tenant indexes
CREATE INDEX documents_org_idx ON documents(organization_id);
CREATE INDEX users_org_idx ON users(organization_id);
```

### 11.2 API Endpoints Specification

```elixir
# Authentication
defmodule AlejandriaWeb.AuthController do
  post "/auth/login", AuthController, :login
  post "/auth/logout", AuthController, :logout
  post "/auth/refresh", AuthController, :refresh
end

# Organizations
defmodule AlejandriaWeb.OrganizationController do
  get "/api/v1/organizations", OrganizationController, :index
  post "/api/v1/organizations", OrganizationController, :create
  get "/api/v1/organizations/:id", OrganizationController, :show
  put "/api/v1/organizations/:id", OrganizationController, :update
end

# Documents
defmodule AlejandriaWeb.DocumentController do
  get "/api/v1/documents", DocumentController, :index
  post "/api/v1/documents", DocumentController, :create
  get "/api/v1/documents/:id", DocumentController, :show
  put "/api/v1/documents/:id", DocumentController, :update
  delete "/api/v1/documents/:id", DocumentController, :delete
end

# Search
defmodule AlejandriaWeb.SearchController do
  post "/api/v1/search", SearchController, :search
  get "/api/v1/search/suggestions", SearchController, :suggestions
end
```

### 11.3 Multi-tenant Middleware

```elixir
defmodule AlejandriaWeb.MultiTenantPlug do
  @behaviour Plug

  def init(opts), do: opts

  def call(conn, _opts) do
    with {:ok, claims} <- Guardian.Plug.current_token(conn) |> Guardian.decode_and_verify(),
         org_id <- claims["organization_id"] do
      conn
      |> Plug.Conn.put_private(:organization_id, org_id)
      |> Plug.Conn.assign(:current_organization_id, org_id)
    else
      _ ->
        conn
        |> Plug.Conn.send_resp(401, "Unauthorized")
        |> Plug.Conn.halt()
    end
  end
end
```

---
