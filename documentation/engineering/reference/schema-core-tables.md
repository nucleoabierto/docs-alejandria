---
id: ENG-REF-0007
area: "engineering"
type: "reference"
title: "Schema Core Tables - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0003", "PRD-0001", "PRD-0002"]
---

Especificación de las tablas core de la base de datos PostgreSQL para Alejandria. Contiene definición de users, organizations, organization_members, document_areas, document_types y documents.

---

# Schema Core Tables - Alejandria

Este documento define las tablas core de la base de datos PostgreSQL para Alejandria MVP.

## Overview

**Base de datos**: PostgreSQL 15+  
**Características clave**: Integridad referencial, JSONB para ACL flexible, índices especializados (GIN)  
**Foco**: Desarrollo local con Docker Compose (ver ADR-0001)

## Tablas

### Users

Tabla de usuarios del sistema.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email TEXT UNIQUE NOT NULL,
    name TEXT NOT NULL,
    api_key TEXT UNIQUE NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Índices**:
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_api_key ON users(api_key);
```

**Constraints**:
- `email`: UNIQUE, NOT NULL
- `api_key`: UNIQUE, NOT NULL

### Organizations

Tabla de organizaciones (personal, team, enterprise).

```sql
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    type TEXT NOT NULL CHECK (type IN ('personal', 'team', 'enterprise')),
    owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Índices**:
```sql
CREATE INDEX idx_organizations_owner_id ON organizations(owner_id);
CREATE INDEX idx_organizations_type ON organizations(type);
```

**Constraints**:
- `type`: CHECK (personal, team, enterprise)
- `owner_id`: FOREIGN KEY → users(id), ON DELETE CASCADE

### Organization Members

Tabla de membresía de usuarios en organizaciones (simplificado para MVP sin invitaciones).

```sql
CREATE TABLE organization_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role TEXT NOT NULL DEFAULT 'member' CHECK (role IN ('owner', 'member')),
    joined_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(organization_id, user_id)
);
```

**Índices**:
```sql
CREATE INDEX idx_organization_members_org_id ON organization_members(organization_id);
CREATE INDEX idx_organization_members_user_id ON organization_members(user_id);
CREATE INDEX idx_organization_members_role ON organization_members(role);
```

**Constraints**:
- `organization_id`: FOREIGN KEY → organizations(id), ON DELETE CASCADE
- `user_id`: FOREIGN KEY → users(id), ON DELETE CASCADE
- `role`: CHECK (owner, member)
- `UNIQUE(organization_id, user_id)`: Un usuario solo puede ser miembro una vez por organización

### Document Areas

Tabla de lookup para áreas de documentos (engineering, product, executive, operations).

```sql
CREATE TABLE document_areas (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Datos iniciales**:
```sql
INSERT INTO document_areas (name, description, sort_order) VALUES
('engineering', 'Documentos técnicos y de ingeniería', 1),
('product', 'Documentos de producto y estrategia', 2),
('executive', 'Documentos ejecutivos y de dirección', 3),
('operations', 'Documentos operacionales y procesos', 4);
```

**Índices**:
```sql
CREATE INDEX idx_document_areas_sort_order ON document_areas(sort_order);
```

### Document Types

Tabla de lookup para tipos de documentos (definition, strategy, guide, record, reference, research).

```sql
CREATE TABLE document_types (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL UNIQUE,
    description TEXT,
    sort_order INTEGER DEFAULT 0,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Datos iniciales**:
```sql
INSERT INTO document_types (name, description, sort_order) VALUES
('definition', 'Definiciones y especificaciones', 1),
('strategy', 'Estrategias y planes', 2),
('guide', 'Guías y manuales', 3),
('record', 'Registros históricos', 4),
('reference', 'Material de referencia', 5),
('research', 'Investigación y análisis', 6);
```

**Índices**:
```sql
CREATE INDEX idx_document_types_sort_order ON document_types(sort_order);
```

### Documents

Tabla principal de documentos organizacionales.

```sql
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    owner_id UUID NOT NULL REFERENCES users(id),
    title TEXT NOT NULL,
    description TEXT,
    current_version_id UUID,
    area_id UUID NOT NULL REFERENCES document_areas(id),
    type_id UUID NOT NULL REFERENCES document_types(id),
    status TEXT NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'draft', 'deprecated', 'archived')),
    acl JSONB NOT NULL DEFAULT '{"visibility": "private", "permissions": {"default": "read"}}',
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

**Estructura de ACL (JSONB)**:
```json
{
  "visibility": "private | public",
  "permissions": {
    "default": "read | write | none",
    "specific_users": [
      {"user_id": "uuid", "permission": "read | write | none"}
    ]
  }
}
```

**Índices**:
```sql
CREATE INDEX idx_documents_organization_id ON documents(organization_id);
CREATE INDEX idx_documents_owner_id ON documents(owner_id);
CREATE INDEX idx_documents_area_id ON documents(area_id);
CREATE INDEX idx_documents_type_id ON documents(type_id);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_documents_created_at ON documents(created_at DESC);
CREATE INDEX idx_documents_acl ON documents USING GIN (acl);
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);
```

**Constraints**:
- `organization_id`: FOREIGN KEY → organizations(id), ON DELETE CASCADE
- `owner_id`: FOREIGN KEY → users(id)
- `area_id`: FOREIGN KEY → document_areas(id)
- `type_id`: FOREIGN KEY → document_types(id)
- `status`: CHECK (active, draft, deprecated, archived)
- `acl`: JSONB con estructura definida arriba

### Document Versions

Tabla de versiones de documentos para control de cambios.

```sql
CREATE TABLE document_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    version TEXT NOT NULL,
    content TEXT NOT NULL,
    created_by UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    parent_version_id UUID REFERENCES document_versions(id),
    change_summary TEXT
);
```

**Índices**:
```sql
CREATE INDEX idx_document_versions_document_id ON document_versions(document_id);
CREATE INDEX idx_document_versions_parent_version_id ON document_versions(parent_version_id);
CREATE INDEX idx_document_versions_created_by ON document_versions(created_by);
```

**Constraints**:
- `document_id`: FOREIGN KEY → documents(id), ON DELETE CASCADE
- `created_by`: FOREIGN KEY → users(id)
- `parent_version_id`: FOREIGN KEY → document_versions(id) (para cadena de versiones)

## Relaciones

### Diagrama de Relaciones

```text
users (1) ----< (N) organizations (owner)
users (N) ----< (N) organization_members
organizations (1) ----< (N) organization_members
organizations (1) ----< (N) documents
users (1) ----< (N) documents (owner)
document_areas (1) ----< (N) documents
document_types (1) ----< (N) documents
documents (1) ----< (N) document_versions
users (1) ----< (N) document_versions (created_by)
```

### Reglas de Integridad

- **Organizaciones**: Cuando se elimina un usuario, sus organizaciones se eliminan en cascada (ON DELETE CASCADE)
- **Documentos**: Cuando se elimina una organización, todos sus documentos se eliminan en cascada
- **Versiones**: Cuando se elimina un documento, todas sus versiones se eliminan en cascada
- **Members**: Cuando se elimina una organización, todos sus miembros se eliminan en cascada

## Referencias

- **ENG-REF-0008**: ACL and Metadata (ACL structure, tags, relations)
- **ENG-REF-0009**: Migrations and Performance (migrations, backup, restore)
- **ADR-0003**: PostgreSQL como Base de Datos Relacional
- **ENG-DEF-0001**: System Architecture
