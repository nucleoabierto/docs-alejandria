---
id: ENG-REF-0002
area: "engineering"
type: "reference"
title: "MVP Web API Specification"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "PRD-0001", "ADR-0006", "ENG-REF-0003", "ENG-REF-0004", "ENG-REF-0005", "ENG-REF-0006"]
---

Índice de especificación técnica de la API web REST de Alejandria. Este documento es un índice que referencia a los documentos específicos de cada módulo de la API.

---

# MVP Web API Specification - Alejandria

Este documento es un índice de la especificación técnica de la API web REST expuesta por Alejandria. La especificación detallada de cada módulo está en documentos separados.

## Overview

**Framework**: FastAPI (ver ADR-0006)  
**Protocolo**: HTTPS  
**Formato**: JSON  
**Autenticación**: API keys para MVP (ver [ENG-REF-0003](./api-authentication.md))  
**Base URL**: `https://api.alejandria.com/v1` (producción) / `http://localhost:8000/v1` (desarrollo)  
**Documentación interactiva**: Swagger UI en `/docs`, ReDoc en `/redoc`

## Documentos de Especificación

La especificación de la API web está dividida en los siguientes documentos:

### [ENG-REF-0003: API Authentication](./api-authentication.md)
Estrategia de autenticación para Alejandria. MVP usa API keys para MCP; JWT tokens para autenticación web es fase futura.

### [ENG-REF-0004: API Users](./api-users.md)
Endpoints de gestión de usuarios:
- `POST /v1/users` - Crear usuario
- `GET /v1/users/me` - Obtener información del usuario
- `PATCH /v1/users/me` - Actualizar usuario
- `POST /v1/users/me/api-key` - Regenerar API key

### [ENG-REF-0005: API Organizations](./api-organizations.md)
Endpoints de gestión de organizaciones y miembros:
- `POST /v1/organizations` - Crear organización
- `GET /v1/organizations` - Listar organizaciones
- `GET /v1/organizations/{id}` - Obtener organización
- `PATCH /v1/organizations/{id}` - Actualizar organización
- `POST /v1/organizations/{id}/members` - Agregar miembro
- `GET /v1/organizations/{id}/members` - Listar miembros
- `DELETE /v1/organizations/{id}/members/{user_id}` - Remover miembro

### [ENG-REF-0006: API Back Office](./api-back-office.md)
Endpoints de back office para monitoreo:
- `GET /v1/health` - Health check
- `GET /v1/metrics` - Métricas de monitoreo (admin API key)

## Errores Comunes

### Estructura de Error

Todos los endpoints siguen la misma estructura de respuesta de error:

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

### Códigos de Error HTTP

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

- **ENG-REF-0003**: API Authentication
- **ENG-REF-0004**: API Users
- **ENG-REF-0005**: API Organizations
- **ENG-REF-0006**: API Back Office
- **ADR-0006**: FastAPI como Framework de API Web
- **PRD-0001**: Organization and User Management
- **System Architecture (ENG-DEF-0001)**: Arquitectura técnica del sistema
