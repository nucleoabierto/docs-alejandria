---
id: ENG-REF-0001
area: "engineering"
type: "reference"
title: "MVP MCP Tools Definition"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0002", "PRD-0001", "PRD-0002"]
---

Especificación técnica completa de las herramientas MCP expuestas por Alejandria. Contiene definición de cada endpoint MCP con schemas, parámetros, responses, códigos de error y ejemplos de uso.

---

# MVP MCP Tools Definition - Alejandria

Este documento define la especificación técnica de todas las herramientas MCP expuestas por el servidor FastMCP de Alejandria.

## Overview

**Framework**: FastMCP (Prefect)  
**Transporte**: stdio (puede extenderse a SSE en futuro)  
**Autenticación**: API keys (header `Authorization: Bearer <api_key>`)  
**Formato**: JSON-RPC 2.0 (protocolo MCP estándar)

## Tools de Gestión de Documentos

### create_document

Crea un nuevo documento organizacional en Alejandria.

**Endpoint**: `create_document`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  title: string,              // Título del documento (required, max 200 chars)
  content: string,            // Contenido completo del documento (required, max 1MB)
  organization_id: string,    // ID de la organización (required, UUID)
  area_id: string,            // ID del área (required, UUID, referencia a document_areas)
  type_id: string,            // ID del tipo (required, UUID, referencia a document_types)
  description?: string,       // Descripción breve (optional, max 500 chars)
  acl: {                      // Configuración de ACL (required)
    visibility: string,       // "private" | "public"
    permissions: {
      default: string,         // "read" | "write" | "none"
      specific_users?: Array<{ // Permisos específicos por usuario (optional)
        user_id: string,       // ID del usuario (UUID)
        permission: string     // "read" | "write" | "none"
      }>
    }
  },
  tags?: string[]             // Etiquetas para categorización (optional, max 10)
}
```

#### Valores de Áreas (document_areas)

**Áreas por defecto** (insertadas automáticamente en document_areas):
- `engineering`: Documentos técnicos y de ingeniería
- `product`: Documentos de producto y estrategia
- `executive`: Documentos ejecutivos y de dirección
- `operations`: Documentos operacionales y procesos

**Extensibilidad**: El usuario puede agregar nuevas áreas insertando registros en la tabla `document_areas`. Cada área tiene un UUID único que se usa como `area_id` en los documentos.

#### Valores de Tipos (document_types)

**Tipos por defecto** (insertados automáticamente en document_types):
- `definition`: Definiciones y especificaciones
- `strategy`: Estrategias y planes
- `guide`: Guías y manuales
- `record`: Registros históricos
- `reference`: Material de referencia
- `research`: Investigación y análisis

**Extensibilidad**: El usuario puede agregar nuevos tipos insertando registros en la tabla `document_types`. Cada tipo tiene un UUID único que se usa como `type_id` en los documentos.

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "document_id": "uuid",
    "version": "1.0",
    "created_at": "2026-04-24T12:00:00Z",
    "organization_id": "uuid",
    "title": "string",
    "area_id": "uuid",
    "area_name": "engineering",
    "type_id": "uuid",
    "type_name": "definition",
    "status": "active"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 400 | "title cannot be empty" | Título vacío |
| 400 | "content cannot be empty" | Contenido vacío |
| 400 | "invalid area value" | Área no válida |
| 400 | "invalid type value" | Tipo no válido |
| 403 | "organization not found or access denied" | Usuario no pertenece a organización |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
# Desde Claude Desktop/Cursor
result = await mcp.call_tool("create_document", {
  "title": "Arquitectura de Autenticación",
  "content": "Este documento describe la arquitectura...",
  "organization_id": "org-123",
  "area_id": "area-engineering-uuid",  # UUID del área "engineering"
  "type_id": "type-definition-uuid",  # UUID del tipo "definition"
  "description": "Arquitectura técnica del sistema de autenticación",
  "acl": {
    "visibility": "private",
    "permissions": {
      "default": "read"
    }
  },
  "tags": ["architecture", "security"]
})
```

---

### update_document

Edita un documento existente creando una nueva versión.

**Endpoint**: `update_document`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  document_id: string,        // ID del documento (required, UUID)
  changes: {                   // Cambios a aplicar (required)
    title?: string,           // Nuevo título (optional)
    content?: string,         // Nuevo contenido (optional)
    description?: string,     // Nueva descripción (optional)
    status?: string,           // Nuevo status (optional, enum)
    acl?: {                   // Nueva configuración de ACL (optional)
      visibility: string,
      permissions: {
        default: string,
        specific_users?: Array<{
          user_id: string,
          permission: string
        }>
      }
    },
    tags?: string[]           // Nuevas etiquetas (optional)
  },
  change_summary?: string     // Resumen de cambios (optional, max 500 chars)
}
```

#### Valores Enum

**status**: `"active"` | `"draft"` | `"deprecated"` | `"archived"`

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "document_id": "uuid",
    "version": "2.0",
    "previous_version": "1.0",
    "created_at": "2026-04-24T13:00:00Z",
    "change_summary": "Updated ACL to public"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 404 | "document not found" | Documento no existe |
| 403 | "write permission denied" | Usuario no tiene permiso de escritura |
| 403 | "document is deleted" | Documento fue eliminado |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
result = await mcp.call_tool("update_document", {
  "document_id": "doc-123",
  "changes": {
    "content": "Contenido actualizado con nueva información...",
    "status": "active"
  },
  "change_summary": "Updated content and activated document"
})
```

---

### search_documents

Busca documentos usando búsqueda semántica con filtros ACL.

**Endpoint**: `search_documents`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  query: string,              // Query de búsqueda en lenguaje natural (required, max 512 tokens)
  organization_id: string,   // ID de la organización (required, UUID)
  limit?: number,             // Número máximo de resultados (optional, default 10, max 50)
  filters?: {                 // Filtros adicionales (optional)
    area_id?: string,         // Filtrar por área_id (UUID, referencia a document_areas)
    type_id?: string,         // Filtrar por type_id (UUID, referencia a document_types)
    status?: string,           // Filtrar por status
    tags?: string[]           // Filtrar por etiquetas
  }
}
```

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "results": [
      {
        "document_id": "uuid",
        "title": "string",
        "description": "string",
        "area_id": "uuid",
        "area_name": "engineering",
        "type_id": "uuid",
        "type_name": "definition",
        "status": "active",
        "score": 0.95,
        "snippet": "Fragmento relevante del documento...",
        "metadata": {
          "created_at": "2026-04-24T12:00:00Z",
          "updated_at": "2026-04-24T13:00:00Z",
          "tags": ["tag1", "tag2"]
        }
      }
    ],
    "total": 5,
    "query": "string",
    "latency_ms": 150
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 400 | "query cannot be empty" | Query vacío |
| 403 | "organization not found or access denied" | Usuario no pertenece a organización |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
result = await mcp.call_tool("search_documents", {
  "query": "cómo funciona la autenticación en el sistema",
  "organization_id": "org-123",
  "limit": 10,
  "filters": {
    "area_id": "area-engineering-uuid",  // UUID del área "engineering"
    "status": "active"
  }
})
```

---

### get_document

Recupera un documento específico por ID.

**Endpoint**: `get_document`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  document_id: string,        // ID del documento (required, UUID)
  version_id?: string         // ID de versión específica (optional, UUID)
}
```

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "document_id": "uuid",
    "organization_id": "uuid",
    "title": "string",
    "description": "string",
    "content": "Contenido completo del documento...",
    "area_id": "uuid",
    "area_name": "engineering",
    "type_id": "uuid",
    "type_name": "definition",
    "status": "active",
    "version": "1.0",
    "created_at": "2026-04-24T12:00:00Z",
    "updated_at": "2026-04-24T13:00:00Z",
    "acl": {
      "visibility": "private",
      "permissions": {
        "default": "read"
      }
    },
    "tags": ["tag1", "tag2"],
    "metadata": {
      "owner_id": "uuid",
      "current_version_id": "uuid"
    }
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 404 | "document not found" | Documento no existe |
| 403 | "read permission denied" | Usuario no tiene permiso de lectura |
| 404 | "version not found" | Versión específica no existe |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
result = await mcp.call_tool("get_document", {
  "document_id": "doc-123"
})
```

---

### delete_document

Elimina un documento (soft delete).

**Endpoint**: `delete_document`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  document_id: string         // ID del documento (required, UUID)
}
```

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "document_id": "uuid",
    "deleted_at": "2026-04-24T14:00:00Z",
    "message": "Document marked as deleted"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 404 | "document not found" | Documento no existe |
| 403 | "only owner can delete document" | Usuario no es el owner |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
result = await mcp.call_tool("delete_document", {
  "document_id": "doc-123"
})
```

---

### list_documents

Lista documentos de una organización con filtros.

**Endpoint**: `list_documents`  
**Método**: MCP tool call

#### Parámetros

```typescript
{
  organization_id: string,   // ID de la organización (required, UUID)
  filters?: {                 // Filtros (optional)
    area_id?: string,         // Filtrar por área_id (UUID, referencia a document_areas)
    type_id?: string,         // Filtrar por type_id (UUID, referencia a document_types)
    status?: string,          // Filtrar por status
    tags?: string[]           // Filtrar por etiquetas
  },
  limit?: number,             // Límite de resultados (optional, default 20, max 100)
  offset?: number             // Offset para paginación (optional, default 0)
}
```

#### Response Exitoso

```json
{
  "success": true,
  "data": {
    "documents": [
      {
        "document_id": "uuid",
        "title": "string",
        "description": "string",
        "area_id": "uuid",
        "area_name": "engineering",
        "type_id": "uuid",
        "type_name": "definition",
        "status": "active",
        "created_at": "2026-04-24T12:00:00Z",
        "updated_at": "2026-04-24T13:00:00Z",
        "tags": ["tag1", "tag2"]
      }
    ],
    "total": 50,
    "limit": 20,
    "offset": 0
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 403 | "organization not found or access denied" | Usuario no pertenece a organización |
| 401 | "authentication required" | API key inválida o faltante |

#### Ejemplo de Uso

```python
result = await mcp.call_tool("list_documents", {
  "organization_id": "org-123",
  "filters": {
    "area": "engineering",
    "status": "active"
  },
  "limit": 20,
  "offset": 0
})
```

---

## Autenticación

Todas las llamadas a herramientas MCP requieren autenticación vía API key.

### Header de Autenticación

```
Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

### Validación

- API key se valida en cada llamada
- API key está encriptada en base de datos
- API key tiene asociado un user_id
- API key puede ser revocada y regenerada

## Rate Limiting

- **Default**: 100 requests/minuto por usuario
- **Burst**: 200 requests en ventana de 1 minuto
- **Headers de rate limit**:
  - `X-RateLimit-Limit`: Límite total
  - `X-RateLimit-Remaining`: Requests restantes
  - `X-RateLimit-Reset`: Timestamp de reset

## Versioning

La API MCP sigue versionamiento semántico basado en cambios en tools:

- **Major**: Cambios breaking en herramientas existentes
- **Minor**: Adición de nuevas herramientas
- **Patch**: Correcciones de bugs sin cambios en funcionalidad

Versión actual: `1.0.0`

## Errores Comunes

### Estructura de Error

```json
{
  "success": false,
  "error": {
    "code": 403,
    "message": "write permission denied",
    "details": {
      "document_id": "doc-123",
      "user_id": "user-456"
    }
  }
}
```

### Códigos de Error HTTP

| Código | Significado |
|--------|-------------|
| 200 | Success |
| 400 | Bad Request (parámetros inválidos) |
| 401 | Unauthorized (autenticación requerida) |
| 403 | Forbidden (permisos insuficientes) |
| 404 | Not Found (recurso no existe) |
| 429 | Too Many Requests (rate limit exceeded) |
| 500 | Internal Server Error (error del servidor) |

## Referencias

- **ADR-0002**: Python + FastMCP como Backend Framework
- **PRD-0001**: Organization and User Management
- **PRD-0002**: MVP Document Management via MCP
- **System Architecture (ENG-DEF-0001)**: Arquitectura técnica del sistema
