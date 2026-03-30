---
id: ADR-0003
area: "engineering"
type: "decision"
title: "PostgreSQL como Base de Datos Relacional"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "PRD-0001", "PRD-0002", "ENG-REF-0007", "ADR-0001"]
---

Registro de decisión arquitectónica que justifica la selección de PostgreSQL como base de datos relacional para Alejandria. Contiene contexto del problema de almacenamiento de documentos estructurados con relaciones complejas, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0003: PostgreSQL como Base de Datos Relacional

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que necesitamos una base de datos relacional para almacenar documentos originales, organizaciones, usuarios, relaciones entre entidades, y ACL (Access Control Lists). Qdrant maneja los embeddings y búsqueda semántica, pero necesitamos un sistema para la metadata estructurada y el contenido completo de los documentos.

El desafío técnico es que el modelo de datos de Alejandria tiene relaciones complejas:
- Organizaciones tienen múltiples usuarios
- Documentos pertenecen a organizaciones
- Documentos tienen múltiples versiones con historial
- ACL granular por documento (visibility, permissions específicos por usuario)
- Tags y relaciones entre documentos

Entendemos que necesitamos:
- Integridad referencial fuerte (foreign keys, constraints)
- Consultas complejas con joins (documentos por organización, por usuario, por ACL)
- Transacciones ACID para consistencia de datos
- Soporte para JSONB para ACL flexible
- Escalabilidad predecible (~1M documentos en MVP)
- Tecnología conocida y madura por el equipo

## Decision

Usar PostgreSQL como base de datos relacional para almacenamiento de documentos, organizaciones, usuarios, relaciones y ACL.

**Especificación técnica:**
- **Base de datos**: PostgreSQL 15+
- **Schema**: Normalizado con tablas principales (organizations, users, documents, document_versions, document_acl, document_tags)
- **JSONB**: Para ACL flexible y metadata adicional
- **Índices**: En campos frecuentemente consultados (organization_id, owner, acl.visibility, created_at)
- **Backup**: Daily backups con retention 30 días
- **Partitioning**: Por organización cuando crezca (>10M documentos)

PostgreSQL proporciona:
- Integridad referencial con foreign keys y constraints (piensa en esto como reglas que protegen tus datos)
- Soporte JSONB para datos semi-estructurados (ACL, metadata)
- Consultas complejas con joins y subqueries
- Transacciones ACID para consistencia
- Índices avanzados (B-tree, GIN, GiST)
- Escalabilidad con partitioning y replication
- Ecosistema maduro y bien documentado

## Consequences

### Positivas

✅ **Integridad referencial fuerte**: Foreign keys aseguran que no haya documentos huérfanos sin organización, ni usuarios sin organización. Constraints validan datos en el nivel de base de datos, protegiendo tu información.

✅ **JSONB para ACL flexible**: ACL granular (visibility, permissions específicos por usuario) se almacena como JSONB, permitiendo estructura flexible sin cambiar schema. PostgreSQL indexa JSONB eficientemente con índices GIN.

✅ **Consultas complejas eficientes**: Joins entre documentos, organizaciones y usuarios son nativos y eficientes. Podemos consultar "todos los documentos públicos de la organización X" o "documentos donde el usuario Y tiene permiso de lectura" con una sola query.

✅ **Transacciones ACID**: Cuando creamos un documento, insertamos en `documents`, `document_versions`, `document_acl` y `document_tags` en una sola transacción. Si algo falla, todo se rollback, garantizando consistencia.

✅ **Escalabilidad predecible**: PostgreSQL maneja ~1M documentos con queries <100ms en hardware modesto. Partitioning por organización permite escalar a 100M+ documentos cuando sea necesario.

✅ **Tecnología conocida**: El equipo tiene experiencia con PostgreSQL, reduciendo curva de aprendizaje y riesgos de implementación.

✅ **Ecosistema maduro**: Herramientas de backup, monitoring, migrations (Alembic), y ORMs (SQLAlchemy) son robustas y bien documentadas.

### Negativas

❌ **No es ideal para alta write throughput**: PostgreSQL es más lento que NoSQL databases para escrituras masivas. Esto no es crítico para el MVP donde la mayoría de operaciones son reads (búsquedas).

❌ **Schema changes requieren migrations**: Cambios en schema requieren migrations con Alembic, lo cual añade overhead. Esto es aceptable porque el schema del MVP está bien definido y cambios futuros serán controlados.

❌ **Vertical scaling limitado**: PostgreSQL escala verticalmente (más CPU/RAM) mejor que horizontalmente. Para el MVP esto es aceptable, y partitioning permite cierto horizontal scaling.

❌ **JSONB queries más lentas que columnas nativas**: Consultas en JSONB son más lentas que en columnas nativas. Mitigamos esto indexando campos frecuentemente consultados de ACL (visibility) como columnas nativas.

## Alternatives Considered

### MongoDB

**Por qué no la elegimos:**
- Sin integridad referencial fuerte (no foreign keys nativos)
- Schema-less puede llevar a datos inconsistentes
- Consultas complejas con joins son menos eficientes
- Menos experiencia del equipo con MongoDB

**Cuándo podría ser mejor:**
- Si el schema fuera altamente dinámico
- Si necesitáramos alta write throughput masiva
- Si las relaciones entre entidades fueran simples

### MySQL

**Por qué no la elegimos:**
- JSON support menos maduro que PostgreSQL (MySQL 8.0+ tiene JSON, pero JSONB de PostgreSQL es más eficiente)
- Menos features avanzados (partitioning, índices especializados)
- El equipo tiene más experiencia con PostgreSQL

**Cuándo podría ser mejor:**
- Si el proyecto tuviera dependencia existente en MySQL
- Si necesitáramos features específicos de MySQL enterprise

### SQLite

**Por qué no la elegimos:**
- No soporta concurrencia real (single writer)
- No escala para múltiples usuarios simultáneos
- Sin features avanzados (partitioning, replication)
- No apropiado para producción multi-usuario

**Cuándo podría ser mejor:**
- Si fuera una aplicación desktop single-user
- Si el MVP fuera para prototipado rápido sin persistencia

## Implementation

### Paso 1: Schema de base de datos (ejemplo simplificado)

El schema completo de base de datos está documentado en ENG-REF-0007, ENG-REF-0008 y ENG-REF-0009. A continuación se muestra un ejemplo simplificado que demuestra las capacidades clave de PostgreSQL justificadas en esta decisión:

```sql
-- Ejemplo que demuestra: integridad referencial, JSONB, constraints

-- Users (ejemplo de integridad referencial)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    api_key TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Organizations (ejemplo de foreign keys y tipos con CHECK)
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    type TEXT NOT NULL CHECK (type IN ('personal', 'team', 'enterprise')),
    owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Documents (ejemplo de JSONB para ACL flexible)
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    owner_id UUID NOT NULL REFERENCES users(id),
    title TEXT NOT NULL,
    description TEXT,
    acl JSONB NOT NULL DEFAULT '{"visibility": "private", "permissions": {"default": "read"}}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Índice GIN para búsqueda eficiente en JSONB (ejemplo de índices especializados)
CREATE INDEX idx_documents_acl ON documents USING GIN (acl);
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);
```

### Paso 2: Índices para performance
```sql
-- Índices en campos frecuentemente consultados
CREATE INDEX idx_documents_organization_id ON documents(organization_id);
CREATE INDEX idx_documents_owner_id ON documents(owner_id);
CREATE INDEX idx_documents_created_at ON documents(created_at DESC);
```

### Paso 3: Cliente Python con SQLAlchemy
```python
from sqlalchemy import create_engine, Column, String, DateTime, ForeignKey, Text, JSON
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

class Document(Base):
    __tablename__ = 'documents'
    
    id = Column(String, primary_key=True)
    organization_id = Column(String, ForeignKey('organizations.id'))
    owner_id = Column(String, ForeignKey('users.id'))
    title = Column(String, nullable=False)
    description = Column(String)
    acl = Column(JSON, default={"visibility": "private", "permissions": {"default": "read"}})
    metadata = Column(JSON, default={})
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
```

### Paso 4: Migrations con Alembic
```bash
# Inicializar Alembic
alembic init migrations

# Generar migration desde schema
alembic revision --autogenerate -m "Initial schema"

# Aplicar migration
alembic upgrade head
```

### Paso 5: Testing
- Unit tests para cada modelo y query
- Integration tests con transacciones
- Performance tests para queries frecuentes
- ACL filtering tests para verificar seguridad

### Paso 6: Deployment
- Docker container con PostgreSQL 15+
- Environment variables para configuración
- Backup schedule (pg_dump diario)
- Health checks para monitoreo

## Related Decisions

- **ADR-0002**: Python + FastMCP como backend framework (PostgreSQL se integra con SQLAlchemy)
- **ADR-0004**: Qdrant como vector database (PostgreSQL complementa Qdrant para metadata estructurada)
- **ADR-0005**: sentence-transformers para embeddings (PostgreSQL almacena documentos originales)
- **ENG-REF-0007**: Schema Core Tables (schema completo de todas las tablas)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Alta write throughput:**
- Usar connection pooling para reducir overhead
- Batch inserts cuando sea posible
- Considerar async operations para writes no críticos
- Monitorear performance y optimizar queries

**Schema changes:**
- Diseñar schema cuidadosamente desde el inicio
- Usar migrations controladas con Alembic
- Documentar cambios en schema en ADRs separados
- Testing exhaustivo de migrations

**Vertical scaling:**
- Monitorear uso de CPU/RAM y planificar upgrades
- Implementar partitioning por organización cuando crezca
- Considerar read replicas para offload reads
- Plan de migración a distributed database si es necesario

**JSONB performance:**
- Indexar campos frecuentemente consultados de ACL como columnas nativas
- Usar GIN indexes para JSONB cuando sea necesario
- Profilear queries y optimizar según patrones de acceso
- Considerar denormalizar ACL crítico si es necesario
