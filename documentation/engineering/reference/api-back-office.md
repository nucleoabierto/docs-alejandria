---
id: ENG-REF-0006
area: "engineering"
type: "reference"
title: "API Back Office - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0002", "ENG-REF-0003"]
---

Especificación de endpoints de back office para la API web de Alejandria. Contiene health check y métricas de monitoreo.

---

# API Back Office - Alejandria

Este documento define los endpoints de back office para monitoreo y métricas de la API web REST de Alejandria.

## Overview

**Framework**: FastAPI (ver ADR-0006)  
**Protocolo**: HTTPS  
**Formato**: JSON  
**Autenticación**: JWT tokens (Bearer token) - ver [API Authentication](./api-authentication.md)  
**Base URL**: `https://api.alejandria.com/v1` (producción) / `http://localhost:8000/v1` (desarrollo)

## Endpoints de Back Office

### health_check

Endpoint de health check para monitoreo.

**Endpoint**: `GET /v1/health`  
**Autenticación**: No requerida

#### Response Exitoso (200)

```json
{
  "status": "healthy",
  "timestamp": "2026-04-24T12:00:00Z",
  "services": {
    "postgres": "healthy",
    "qdrant": "healthy"
  }
}
```

#### Response Error (503)

```json
{
  "status": "unhealthy",
  "timestamp": "2026-04-24T12:00:00Z",
  "services": {
    "postgres": "unhealthy",
    "qdrant": "healthy"
  }
}
```

#### Ejemplo de Uso

```bash
curl -X GET http://localhost:8000/v1/health
```

---

### metrics

Endpoint de métricas para monitoreo (requiere admin API key).

**Endpoint**: `GET /v1/metrics`  
**Autenticación**: Requerida (admin API key)

#### Response Exitoso (200)

```json
{
  "success": true,
  "data": {
    "timestamp": "2026-04-24T12:00:00Z",
    "users": {
      "total": 1000,
      "active_today": 50
    },
    "organizations": {
      "total": 100,
      "active_today": 10
    },
    "documents": {
      "total": 10000,
      "created_today": 50
    },
    "requests": {
      "total_last_hour": 1000,
      "avg_latency_ms": 150
    }
  }
}
```

#### Errores

| Código | Mensaje | Causa |
|--------|---------|-------|
| 401 | "admin authentication required" | API key admin inválida o faltante |

#### Ejemplo de Uso

```bash
curl -X GET http://localhost:8000/v1/metrics \
  -H "Authorization: Bearer admin_sk-..."
```

## Referencias

- **ENG-REF-0002**: MVP Web API Specification (documento principal)
- **ENG-REF-0004**: API Authentication
