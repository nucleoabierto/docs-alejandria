---
id: ENG-REF-0003
area: "engineering"
type: "reference"
title: "API Authentication - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0002", "ENG-DEF-0001", "PRD-0001", "ADR-0006", "ADR-0002"]
---

Especificación de autenticación para Alejandria. MVP usa API keys para MCP; JWT tokens para autenticación web es fase futura cuando se implemente web UI.

---

# API Authentication - Alejandria

Este documento define la estrategia de autenticación para Alejandria.

## Overview

**Framework**: FastAPI (ver ADR-0006)  
**Protocolo**: HTTPS  
**Formato**: JSON  
**Autenticación MVP**: API keys (para MCP endpoints)  
**Autenticación Futura**: JWT tokens (para web UI - fase futura)  
**Base URL**: `https://api.alejandria.com/v1` (producción) / `http://localhost:8000/v1` (desarrollo)

## Estrategia de Autenticación

### MVP (Fase Actual)

**Método**: API keys simples  
**Uso**: Autenticación para MCP endpoints  
**Almacenamiento**: Encriptadas en PostgreSQL  
**Generación**: Automática al crear usuario, puede regenerarse  

**Razón para API keys en MVP**:
- Simplicidad para integración MCP
- No requiere UI de login
- Suficiente para multi-tenancy por organización
- Compatible con herramientas de IA (Claude, Cursor)

### Fase Futura (Web UI)

**Método**: JWT tokens (Bearer token)  
**Uso**: Autenticación para web UI REST API  
**Features**: Login con email/password, refresh tokens, SSO con OAuth 2.0  

**Razón para JWT en fase futura**:
- Requiere web UI para login/logout
- Mejor UX para aplicaciones web
- Compatible con OAuth 2.0 y SSO

## Autenticación MVP (API Keys)

### Uso de API Keys

Todos los endpoints MCP requieren API key en el header:

```
Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxx
```

### Generación de API Key

Las API keys se generan automáticamente al crear un usuario (ver ENG-REF-0004 para endpoint `POST /v1/users`). La API key se devuelve solo una vez en la respuesta de creación.

### Regeneración de API Key

**Endpoint**: `POST /v1/users/me/api-key`  
**Descripción**: Regenera la API key del usuario autenticado

#### Request

```bash
curl -X POST http://localhost:8000/v1/users/me/api-key \
  -H "Authorization: Bearer sk-xxxxxxxxxxxxxxxxxxxxxxxx"
```

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
| 401 | "authentication required" | API key inválida o faltante |

## Autenticación Fase Futura (JWT)

### Obtener Token (Login) - FASE FUTURA

**Endpoint**: `POST /v1/auth/login`  
**Descripción**: Genera token JWT para autenticación web UI  
**Estado**: No implementado en MVP, fase futura cuando se implemente web UI

#### Request Body (Fase Futura)

```json
{
  "email": "string",      // Email del usuario (required, valid email)
  "password": "string"    // Contraseña (required, min 8 chars)
}
```

#### Response Exitoso (200) - Fase Futura

```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "bearer",
    "expires_in": 3600,
    "user": {
      "user_id": "uuid",
      "email": "string",
      "name": "string"
    }
  }
}
```

#### Errores (Fase Futura)

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "invalid credentials" | Email o contraseña incorrectos |
| 400 | "email not found" | Email no registrado |
| 400 | "invalid email format" | Email inválido |

### Usar Token JWT - FASE FUTURA

Todos los endpoints web UI protegidos requerirán el token en el header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

## Estructura de Error

```json
{
  "success": false,
  "error": {
    "code": 403,
    "message": "access denied",
    "details": {
      "resource": "organization",
      "resource_id": "org-123"
    }
  }
}
```

## Códigos de Error HTTP

| Código | Significado |
|--------|-------------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request (parámetros inválidos) |
| 401 | Unauthorized (autenticación requerida) |
| 403 | Forbidden (permisos insuficientes) |
| 404 | Not Found (recurso no existe) |
| 409 | Conflict (recurso ya existe) |
| 422 | Unprocessable Entity (validación falló) |
| 429 | Too Many Requests (rate limit exceeded) |
| 500 | Internal Server Error (error del servidor) |
| 503 | Service Unavailable (servicio no disponible) |

## Rate Limiting

- **Default**: 1000 requests/hora por usuario
- **Burst**: 100 requests en ventana de 1 minuto
- **Admin endpoints**: 10000 requests/hora

### Headers de Rate Limit

- `X-RateLimit-Limit`: Límite total
- `X-RateLimit-Remaining`: Requests restantes
- `X-RateLimit-Reset`: Timestamp de reset

## Versioning

La API sigue versionamiento en URL (`/v1/`, `/v2/`, etc.). Cambios breaking requieren nueva versión mayor.

Versión actual: `v1`

## Documentación Interactiva

FastAPI genera documentación interactiva automáticamente:

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`
- **OpenAPI Schema**: `http://localhost:8000/openapi.json`

## Referencias

- **ENG-REF-0002**: MVP Web API Specification (documento principal)
- **ADR-0006**: FastAPI como Framework de API Web
- **ENG-DEF-0001**: System Architecture
