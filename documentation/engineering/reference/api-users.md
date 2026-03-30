---
id: ENG-REF-0004
area: "engineering"
type: "reference"
title: "API Users - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0002", "ENG-REF-0003", "PRD-0001"]
---

Especificación de endpoints de usuarios para la API web de Alejandria. Contiene crear usuario, obtener información, actualizar y regenerar API key.

---

# API Users - Alejandria

Este documento define los endpoints de gestión de usuarios para la API web REST de Alejandia.

## Overview

**Framework**: FastAPI (ver ADR-0006)  
**Protocolo**: HTTPS  
**Formato**: JSON  
**Autenticación**: JWT tokens (Bearer token) - ver [API Authentication](./api-authentication.md)  
**Base URL**: `https://api.alejandria.com/v1` (producción) / `http://localhost:8000/v1` (desarrollo)

## Endpoints de Usuarios

### create_user

Crea un nuevo usuario en Alejandria. Este endpoint es público y no requiere autenticación (onboarding inicial).

**Endpoint**: `POST /v1/users`  
**Autenticación**: No requerida

#### Request Body

```json
{
  "email": "string",      // Email del usuario (required, unique, valid email)
  "name": "string",       // Nombre del usuario (required, max 100 chars)
  "password": "string"    // Contraseña (required, min 8 chars, max 128 chars)
}
```

#### Response Exitoso (201)

```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "email": "string",
    "name": "string",
    "api_key": "sk-...",      // API key generada (solo se muestra una vez)
    "organization_id": "uuid", // Organización personal creada automáticamente
    "created_at": "2026-04-24T12:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 400 | "email already exists" | Email ya registrado |
| 400 | "invalid email format" | Email inválido |
| 400 | "name cannot be empty" | Nombre vacío |
| 400 | "password too short" | Contraseña menor a 8 caracteres |

#### Ejemplo de Uso

```bash
curl -X POST http://localhost:8000/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "email": "juan@example.com",
    "name": "Juan Pérez",
    "password": "securepassword123"
  }'
```

---

### get_user

Obtiene información de un usuario autenticado.

**Endpoint**: `GET /v1/users/me`  
**Autenticación**: Requerida

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "email": "string",
    "name": "string",
    "created_at": "2026-04-24T12:00:00Z",
    "organizations": [
      {
        "organization_id": "uuid",
        "name": "string",
        "type": "personal",
        "role": "owner",
        "joined_at": "2026-04-24T12:00:00Z"
      }
    ]
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |

#### Ejemplo de Uso

```bash
curl -X GET http://localhost:8000/v1/users/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

---

### update_user

Actualiza información del usuario autenticado.

**Endpoint**: `PATCH /v1/users/me`  
**Autenticación**: Requerida

#### Request Body

```json
{
  "name": "string",        // Nuevo nombre (optional, max 100 chars)
  "password": "string"    // Nueva contraseña (optional, min 8 chars)
}
```

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "email": "string",
    "name": "string",
    "updated_at": "2026-04-24T13:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |
| 400 | "password too short" | Contraseña menor a 8 caracteres |

#### Ejemplo de Uso

```bash
curl -X PATCH http://localhost:8000/v1/users/me \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Juan Pérez Updated"
  }'
```

---

### regenerate_api_key

Regenera la API key del usuario autenticado.

**Endpoint**: `POST /v1/users/me/api-key`  
**Autenticación**: Requerida

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "api_key": "sk-...",      // Nueva API key (solo se muestra una vez)
    "created_at": "2026-04-24T13:00:00Z"
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "authentication required" | Token inválido o faltante |

#### Ejemplo de Uso

```bash
curl -X POST http://localhost:8000/v1/users/me/api-key \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

## Referencias

- **ENG-REF-0002**: MVP Web API Specification (documento principal)
- **ENG-REF-0003**: API Authentication
- **PRD-0001**: Organization and User Management
