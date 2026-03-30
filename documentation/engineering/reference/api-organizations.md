---
id: ENG-REF-0005
area: "engineering"
type: "reference"
title: "API Organizations - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0002", "ENG-REF-0004", "PRD-0001"]
---

Especificación de endpoints de organizaciones y miembros para la API web de Alejandria. Contiene crear, listar, obtener, actualizar organizaciones y gestión de miembros.

---

# API Organizations - Alejandria

Este documento define los endpoints de gestión de organizaciones y miembros para la API web REST de Alejandria.

## Overview

**Framework**: FastAPI (ver ADR-0006)  
**Protocolo**: HTTPS  
**Formato**: JSON  
**Autenticación**: JWT tokens (Bearer token) - ver [API Authentication](./api-authentication.md)  
**Base URL**: `https://api.alejandria.com/v1` (producción) / `http://localhost:8000/v1` (desarrollo)

## Endpoints de Organizaciones

### create_organization

Crea una nueva organización.

**Endpoint**: `POST /v1/organizations`  
**Autenticación**: Requerida

#### Request Body

```json
{
  "name": "string",       // Nombre de la organización (required, max 100 chars)
  "type": "string"        // Tipo de organización (required, enum)
}
```

#### Valores Enum

**type**: `"personal"` | `"team"` | `"enterprise"`

#### Response Exitoso (201)

```json
{
  "success": true,
  "data": {
    "organization_id": "uuid",
    "name": "string",
    "type": "team",
    "owner_id": "uuid",
    "created_at": "2026-04-24T12:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 400 | "invalid organization type" | Tipo no válido |

#### Ejemplo de Uso

```bash
curl -X POST http://localhost:8000/v1/organizations \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Equipo de Ingeniería",
    "type": "team"
  }'
```

---

### list_organizations

Lista organizaciones del usuario autenticado.

**Endpoint**: `GET /v1/organizations`  
**Autenticación**: Requerida

#### Query Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| type | string | Filtrar por tipo (optional) |
| limit | integer | Límite de resultados (optional, default 20, max 100) |
| offset | integer | Offset para paginación (optional, default 0) |

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "organizations": [
      {
        "organization_id": "uuid",
        "name": "string",
        "type": "team",
        "role": "owner",
        "member_count": 5,
        "created_at": "2026-04-24T12:00:00Z"
      }
    ],
    "total": 2,
    "limit": 20,
    "offset": 0
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |

#### Ejemplo de Uso

```bash
curl -X GET "http://localhost:8000/v1/organizations?type=team&limit=20" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### get_organization

Obtiene detalles de una organización específica.

**Endpoint**: `GET /v1/organizations/{organization_id}`  
**Autenticación**: Requerida

#### Path Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| organization_id | string | ID de la organización (UUID) |

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "organization_id": "uuid",
    "name": "string",
    "type": "team",
    "owner_id": "uuid",
    "member_count": 5,
    "document_count": 120,
    "created_at": "2026-04-24T12:00:00Z",
    "updated_at": "2026-04-24T13:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 403 | "access denied" | Usuario no es miembro |
| 404 | "organization not found" | Organización no existe |

#### Ejemplo de Uso

```bash
curl -X GET http://localhost:8000/v1/organizations/org-123 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### update_organization

Actualiza información de una organización (solo owner).

**Endpoint**: `PATCH /v1/organizations/{organization_id}`  
**Autenticación**: Requerida (solo owner)

#### Path Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| organization_id | string | ID de la organización (UUID) |

#### Request Body

```json
{
  "name": "string"        // Nuevo nombre (optional, max 100 chars)
}
```

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "organization_id": "uuid",
    "name": "string",
    "updated_at": "2026-04-24T13:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 403 | "only owner can update organization" | Usuario no es owner |
| 404 | "organization not found" | Organización no existe |

#### Ejemplo de Uso

```bash
curl -X PATCH http://localhost:8000/v1/organizations/org-123 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Equipo de Ingeniería Updated"
  }'
```

---

## Endpoints de Miembros

### add_member

Agrega un miembro a una organización (solo owner).

**Endpoint**: `POST /v1/organizations/{organization_id}/members`  
**Autenticación**: Requerida (solo owner)

#### Path Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| organization_id | string | ID de la organización (UUID) |

#### Request Body

```json
{
  "email": "string"       // Email del usuario a agregar (required, valid email)
}
```

#### Response Exitoso (201)

```json
{
  "success": true,
  "data": {
    "organization_id": "uuid",
    "user_id": "uuid",
    "email": "string",
    "role": "member",
    "joined_at": "2026-04-24T12:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 403 | "only owner can add members" | Usuario no es owner |
| 404 | "organization not found" | Organización no existe |
| 404 | "user not found" | Usuario no existe en el sistema |
| 400 | "user is already a member" | Usuario ya es miembro |

#### Ejemplo de Uso

```bash
curl -X POST http://localhost:8000/v1/organizations/org-123/members \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "email": "maria@example.com"
  }'
```

---

### list_members

Lista miembros de una organización.

**Endpoint**: `GET /v1/organizations/{organization_id}/members`  
**Autenticación**: Requerida (solo miembros)

#### Path Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| organization_id | string | ID de la organización (UUID) |

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "members": [
      {
        "user_id": "uuid",
        "name": "string",
        "email": "string",
        "role": "owner",
        "joined_at": "2026-04-24T12:00:00Z"
      }
    ],
    "total": 5
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 403 | "access denied" | Usuario no es miembro |
| 404 | "organization not found" | Organización no existe |

#### Ejemplo de Uso

```bash
curl -X GET http://localhost:8000/v1/organizations/org-123/members \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### remove_member

Remueve un miembro de una organización (solo owner).

**Endpoint**: `DELETE /v1/organizations/{organization_id}/members/{user_id}`  
**Autenticación**: Requerida (solo owner)

#### Path Parameters

| Parámetro | Tipo | Descripción |
|-----------|------|-------------|
| organization_id | string | ID de la organización (UUID) |
| user_id | string | ID del usuario a remover (UUID) |

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "organization_id": "uuid",
    "user_id": "uuid",
    "removed_at": "2026-04-24T14:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 403 | "only owner can remove members" | Usuario no es owner |
| 400 | "cannot remove owner" | No se puede remover al owner |
| 404 | "organization not found" | Organización no existe |

#### Ejemplo de Uso

```bash
curl -X DELETE http://localhost:8000/v1/organizations/org-123/members/user-456 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

## Referencias

- **ENG-REF-0002**: MVP Web API Specification (documento principal)
- **ENG-REF-0004**: API Authentication
- **PRD-0001**: Organization and User Management
