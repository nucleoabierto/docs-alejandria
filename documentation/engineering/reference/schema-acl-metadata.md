---
id: ENG-REF-0008
area: "engineering"
type: "reference"
title: "Schema ACL and Metadata - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0007", "ENG-REF-0009", "PRD-0002"]
---

Especificación de ACL, metadata y tablas auxiliares de la base de datos PostgreSQL para Alejandria. Contiene estructura de ACL, document_tags y document_relations.

---

# Schema ACL and Metadata - Alejandria

Este documento define la estructura de ACL, metadata y tablas auxiliares para la base de datos PostgreSQL de Alejandria MVP.

## Overview

**Base de datos**: PostgreSQL 15+  
**Características clave**: JSONB para ACL flexible, índices GIN para búsquedas eficientes  
**Foco**: Gestión de permisos y metadata de documentos

## ACL Structure

### Estructura de ACL en JSONB

La columna `acl` en la tabla `documents` usa JSONB para almacenar permisos flexibles:

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

### Reglas de ACL

**Visibility**:
- `private`: Solo accesible por owner y usuarios con permisos específicos
- `public`: Accesible por todos los miembros de la organización

**Permissions**:
- `read`: Permiso de lectura
- `write`: Permiso de escritura
- `none`: Sin permisos

**Validación**:
- Documentos privados requieren permisos default o específicos
- Owner siempre tiene permisos admin (override)
- Permisos específicos override permisos por defecto

### Índices GIN para ACL

```sql
-- Índice GIN para búsquedas eficientes en ACL
CREATE INDEX idx_documents_acl ON documents USING GIN (acl);
```

**Ejemplos de queries**:
```sql
-- Búsqueda de documentos públicos
SELECT * FROM documents 
WHERE acl->>'visibility' = 'public';

-- Búsqueda de documentos con permisos específicos
SELECT * FROM documents 
WHERE acl->'permissions'->'specific_users' @> '[{"user_id": "uuid"}]';
```

## Document Tags

### Tabla de Tags

Tabla de tags para categorización de documentos.

```sql
CREATE TABLE document_tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    tag TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(document_id, tag)
);
```

**Índices**:
```sql
CREATE INDEX idx_document_tags_document_id ON document_tags(document_id);
CREATE INDEX idx_document_tags_tag ON document_tags(tag);
```

**Constraints**:
- `document_id`: FOREIGN KEY → documents(id), ON DELETE CASCADE
- `UNIQUE(document_id, tag)`: Un tag no puede duplicarse en un documento

**Ejemplos de queries**:
```sql
-- Obtener tags de un documento
SELECT tag FROM document_tags 
WHERE document_id = 'doc-uuid';

-- Buscar documentos por tag
SELECT d.* FROM documents d
JOIN document_tags dt ON d.id = dt.document_id
WHERE dt.tag = 'architecture';

-- Agregar tag a documento
INSERT INTO document_tags (document_id, tag) 
VALUES ('doc-uuid', 'architecture');

-- Remover tag de documento
DELETE FROM document_tags 
WHERE document_id = 'doc-uuid' AND tag = 'architecture';
```

## Document Relations

### Tabla de Relaciones

Tabla de relaciones entre documentos (referencias cruzadas).

```sql
CREATE TABLE document_relations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    related_document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    relation_type TEXT NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(document_id, related_document_id)
);
```

**Índices**:
```sql
CREATE INDEX idx_document_relations_document_id ON document_relations(document_id);
CREATE INDEX idx_document_relations_related_document_id ON document_relations(related_document_id);
CREATE INDEX idx_document_relations_type ON document_relations(relation_type);
```

**Constraints**:
- `document_id`: FOREIGN KEY → documents(id), ON DELETE CASCADE
- `related_document_id`: FOREIGN KEY → documents(id), ON DELETE CASCADE
- `UNIQUE(document_id, related_document_id)`: Una relación no puede duplicarse

**Tipos de relaciones**:
- `depends_on`: El documento depende de otro documento
- `references`: El documento hace referencia a otro documento
- `supersedes`: El documento reemplaza a otro documento
- `related_to`: Relación genérica entre documentos

**Ejemplos de queries**:
```sql
-- Obtener relaciones de un documento
SELECT dr.*, d.title as related_title 
FROM document_relations dr
JOIN documents d ON dr.related_document_id = d.id
WHERE dr.document_id = 'doc-uuid';

-- Obtener documentos que referencian a este documento
SELECT d.* FROM documents d
JOIN document_relations dr ON d.id = dr.document_id
WHERE dr.related_document_id = 'doc-uuid';

-- Crear relación entre documentos
INSERT INTO document_relations (document_id, related_document_id, relation_type) 
VALUES ('doc-1', 'doc-2', 'references');

-- Remover relación entre documentos
DELETE FROM document_relations 
WHERE document_id = 'doc-1' AND related_document_id = 'doc-2';
```

## Metadata

### Columna Metadata en Documents

La columna `metadata` en la tabla `documents` usa JSONB para almacenar metadata adicional:

```json
{
  "tags": ["architecture", "security"],
  "author": "juan@example.com",
  "review_date": "2026-04-24",
  "custom_field": "custom_value"
}
```

### Índices GIN para Metadata

```sql
-- Índice GIN para búsquedas eficientes en metadata
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);
```

**Ejemplos de queries**:
```sql
-- Búsqueda por tags en metadata
SELECT * FROM documents 
WHERE metadata->'tags' ? 'architecture';

-- Búsqueda por campo custom en metadata
SELECT * FROM documents 
WHERE metadata->>'author' = 'juan@example.com';

-- Búsqueda por rango de fechas en metadata
SELECT * FROM documents 
WHERE (metadata->>'review_date')::date >= '2026-04-01';
```

## Relaciones Auxiliares

### Diagrama de Relaciones

```text
documents (1) ----< (N) document_tags
documents (1) ----< (N) document_relations
documents (1) ----< (N) document_relations (related_document_id)
```

### Reglas de Integridad

- **Tags/Relations**: Cuando se elimina un documento, sus tags y relaciones se eliminan en cascada

## Referencias

- **ENG-REF-0007**: Schema Core Tables (documento principal)
- **ENG-REF-0009**: Migrations and Performance (migrations, backup, restore)
- **PRD-0002**: MVP Document Management via MCP (requerimientos de ACL)
