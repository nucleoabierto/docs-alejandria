---
id: ENG-GUIDE-0006
area: "engineering"
type: "guide"
title: "API Versioning Strategy - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ENG-REF-0002", "ADR-0006"]
---

Estrategia de versioning para la API web de Alejandria. Contiene esquema de versioning, políticas de cambios, deprecation timeline, migración entre versiones, backward compatibility y comunicación de cambios a desarrolladores.

---

# API Versioning Strategy - Alejandria

Sabemos que evolucionar una API sin romper a los clientes existentes es uno de los desafíos más difíciles del desarrollo de software. Esta guía define una estrategia clara de versioning para la API de Alejandria, para que podamos innovar y mejorar sin dejar a nadie atrás.

## Table of Contents

1. [Overview](#overview)
2. [Esquema de Versioning](#esquema-de-versioning)
3. [Políticas de Cambios](#políticas-de-cambios)
4. [Implementación en FastAPI](#implementación-en-fastapi)
5. [Timeline de Deprecation](#timeline-de-deprecation)
6. [Ejemplos de Cambios](#ejemplos-de-cambios)
7. [Backward Compatibility](#backward-compatibility)
8. [Documentación de Cambios](#documentación-de-cambios)
9. [Comunicación con Desarrolladores](#comunicación-con-desarrolladores)
10. [Breaking Changes](#breaking-changes)
11. [Code Examples](#code-examples)
12. [Best Practices](#best-practices)
13. [Testing de Versiones](#testing-de-versiones)
14. [Referencias](#referencias)

---

**Enfoque**: Versioning por URL path con soporte de versiones múltiples  
**Versión actual**: v1  
**Política**: Backward compatibility dentro de cada versión major  
**Deprecation**: Mínimo 6 meses de aviso antes de eliminar versiones

## Esquema de Versioning

### Version Semántica

Usamos versionado semántico: `MAJOR.MINOR.PATCH`

- **MAJOR**: Cambios breaking (incompatibles hacia atrás)
- **MINOR**: Nuevas features, backward compatible
- **PATCH**: Bug fixes, backward compatible

**Ejemplos**:
- `v1.0.0` → `v1.1.0` (nueva feature, compatible)
- `v1.1.0` → `v1.1.1` (bug fix, compatible)
- `v1.1.1` → `v2.0.0` (breaking changes)

### Versioning por URL Path

Todas las rutas de API incluyen la versión en el path:

```
/api/v1/users
/api/v1/organizations
/api/v1/documents
```

**Ventajas**:
- Explícito y fácil de entender
- Fácil de routear en FastAPI
- Permite soportar múltiples versiones simultáneamente

**Alternativas consideradas y rechazadas**:
- Header-based versioning: Más complejo, menos explícito
- Query parameter versioning: No RESTful, difícil de cachear

## Políticas de Cambios

### Cambios que requieren nueva versión MAJOR (Breaking)

**Cambios en estructura de response**:
- Remover campos de response
- Cambiar tipos de campos (string → int)
- Cambiar nombres de campos
- Remover endpoints

**Cambios en estructura de request**:
- Remover campos requeridos
- Cambiar tipos de campos requeridos
- Cambiar validaciones de forma incompatible

**Cambios semánticos**:
- Cambiar comportamiento de un endpoint de forma incompatible
- Cambiar códigos de error HTTP para casos existentes
- Cambiar significado de campos

### Cambios que requieren nueva versión MINOR (Compatible)

**Nuevos endpoints**:
- Agregar nuevos endpoints
- Agregar nuevos métodos HTTP a recursos existentes

**Nuevos campos en response**:
- Agregar campos opcionales a responses existentes
- Agregar nuevos valores a enums (si backward compatible)

**Nuevos parámetros**:
- Agregar parámetros opcionales a endpoints existentes
- Agregar nuevos query parameters con valores por defecto

**Cambios no-breaking**:
- Mejorar documentación
- Cambiar orden de campos en JSON (no afecta clientes)
- Agregar headers de response opcionales

### Cambios PATCH (Bug Fixes)

- Corregir bugs sin cambiar comportamiento
- Mejorar performance sin cambiar API
- Corregir documentación incorrecta

## Implementación en FastAPI

### Estructura de Routers

```python
# api/routers/v1/__init__.py
from fastapi import APIRouter
from alejandria.api.routers.v1 import users, organizations, documents

router = APIRouter(prefix="/api/v1")

router.include_router(users.router, prefix="/users", tags=["users"])
router.include_router(organizations.router, prefix="/organizations", tags=["organizations"])
router.include_router(documents.router, prefix="/documents", tags=["documents"])

# api/app.py
from alejandria.api.routers.v1 import router as v1_router

app.include_router(v1_router)
```

### Soporte de Múltiples Versiones

```python
# api/routers/v2/__init__.py (cuando se cree v2)
from fastapi import APIRouter
from alejandria.api.routers.v2 import users, organizations

router = APIRouter(prefix="/api/v2")

router.include_router(users.router, prefix="/users", tags=["users"])
router.include_router(organizations.router, prefix="/organizations", tags=["organizations"])

# api/app.py
from alejandria.api.routers.v1 import router as v1_router
from alejandria.api.routers.v2 import router as v2_router

app.include_router(v1_router)
app.include_router(v2_router)
```

### Version Default

```python
# api/app.py
from fastapi import FastAPI, Request
from fastapi.responses import RedirectResponse

app = FastAPI()

@app.get("/api")
async def api_root():
    """Redirect to latest stable version"""
    return RedirectResponse(url="/api/v1")

@app.get("/api/latest")
async def api_latest():
    """Alias for latest stable version"""
    return RedirectResponse(url="/api/v1")
```

## Timeline de Deprecation

### Ciclo de Vida de una Versión

```
v1.0.0 (Lanzamiento)
    ↓
v1.1.0, v1.2.0, ... (Mejoras compatibles)
    ↓
v2.0.0 (Lanzamiento, breaking changes)
    ↓
v1.x.x en deprecation (6 meses)
    ↓
v1.x.x eliminado
```

### Políticas de Deprecation

**Aviso de deprecation**:
- Mínimo 6 meses antes de eliminar una versión
- Header `X-API-Deprecation` en responses de versión deprecated
- Documentación actualizada con fecha de eliminación

**Comunicación**:
- Email a desarrolladores registrados
- Blog post anunciando deprecation
- Changelog con fecha de eliminación

**Soporte durante deprecation**:
- Bug fixes críticos solo
- No nuevas features
- Security patches

### Headers de Deprecation

```python
# api/middleware/deprecation.py
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class DeprecationMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        
        # Agregar header de deprecation para v1 cuando v2 exista
        if request.url.path.startswith("/api/v1") and v2_exists:
            response.headers["X-API-Deprecation"] = "true"
            response.headers["X-API-Deprecation-Date"] = "2026-10-24"
            response.headers["X-API-Sunset"] = "2027-04-24"
            response.headers["Link"] = '</api/v2>; rel="successor-version"'
        
        return response
```

## Ejemplos de Cambios

### Ejemplo 1: Agregar campo a response (MINOR)

**v1.0.0**:
```json
{
  "id": "123",
  "name": "Juan",
  "email": "juan@example.com"
}
```

**v1.1.0** (compatible):
```json
{
  "id": "123",
  "name": "Juan",
  "email": "juan@example.com",
  "created_at": "2026-04-24T00:00:00Z"  // Nuevo campo opcional
}
```

**Acción**: MINOR bump (v1.0.0 → v1.1.0)

### Ejemplo 2: Remover campo (MAJOR)

**v1.0.0**:
```json
{
  "id": "123",
  "name": "Juan",
  "email": "juan@example.com",
  "phone": "+1234567890"  // Campo a remover
}
```

**v2.0.0** (breaking):
```json
{
  "id": "123",
  "name": "Juan",
  "email": "juan@example.com"
  // phone removido
}
```

**Acción**: MAJOR bump (v1.0.0 → v2.0.0), deprecation de v1 por 6 meses

### Ejemplo 3: Cambiar tipo de campo (MAJOR)

**v1.0.0**:
```json
{
  "id": "123",  // String
  "name": "Juan"
}
```

**v2.0.0** (breaking):
```json
{
  "id": 123,  // Integer
  "name": "Juan"
}
```

**Acción**: MAJOR bump (v1.0.0 → v2.0.0)

### Ejemplo 4: Nuevo endpoint (MINOR)

**v1.0.0**: Sin endpoint de búsqueda

**v1.1.0** (compatible):
```
GET /api/v1/users/search?q=juan
```

**Acción**: MINOR bump (v1.0.0 → v1.1.0)

## Backward Compatibility

### Reglas de Compatibilidad

**Para ser backward compatible**:
1. No remover campos de response
2. No cambiar tipos de campos existentes
3. No cambiar validaciones de forma más estricta
4. No cambiar códigos de error HTTP para casos existentes
5. No cambiar significado semántico de campos

**Estrategias para mantener compatibilidad**:
1. Agregar campos nuevos en lugar de cambiar existentes
2. Usar campos opcionales para cambios
3. Mantener campos viejos con deprecation warning
4. Documentar claramente cambios en changelog

### Ejemplo: Cambio compatible

**Escenario**: Queremos cambiar `name` a `first_name` y `last_name`

**v1.0.0**:
```json
{
  "id": "123",
  "name": "Juan Pérez"
}
```

**v1.1.0** (compatible):
```json
{
  "id": "123",
  "name": "Juan Pérez",  // Mantenido por compatibilidad
  "first_name": "Juan",  // Nuevo campo
  "last_name": "Pérez"   // Nuevo campo
}
```

**v2.0.0** (breaking):
```json
{
  "id": "123",
  "first_name": "Juan",
  "last_name": "Pérez"
  // name removido
}
```

## Documentación de Cambios

### Changelog

Mantener un changelog estructurado:

```markdown
# Changelog

## [v1.1.0] - 2026-05-24

### Added
- New endpoint: `GET /api/v1/users/search` for user search
- New field `created_at` to user response
- New field `updated_at` to organization response

### Changed
- Improved performance of `GET /api/v1/documents` by 50%

### Fixed
- Fixed bug where ACL validation was not applied to search results

## [v1.0.0] - 2026-04-24

### Added
- Initial MVP release
- User management endpoints
- Organization management endpoints
- Document management via MCP
```

### Actualización de Referencias

Cuando se lanza una nueva versión:
1. Actualizar `ENG-REF-0002` con especificación de nueva versión
2. Mantener especificación de versión anterior en documento separado
3. Agregar sección "Migration Guide" en documentación

## Comunicación con Desarrolladores

### Avisos de Cambios

**Para cambios MINOR**:
- Changelog en GitHub
- Release notes en documentación
- Email opcional para cambios significativos

**Para cambios MAJOR**:
- Changelog detallado
- Blog post explicando cambios
- Email a todos los desarrolladores registrados
- Webinar opcional para cambios complejos

**Para deprecation**:
- Header `X-API-Deprecation` en responses
- Email 6 meses antes de eliminación
- Recordatorio 3 meses antes
- Recordatorio 1 mes antes

### Guías de Migración

Para cada cambio MAJOR, proporcionar guía de migración:

```markdown
# Migration Guide: v1 → v2

## Breaking Changes

### 1. User response structure changed

**v1**:
```json
{
  "id": "123",
  "name": "Juan Pérez",
  "email": "juan@example.com"
}
```

**v2**:
```json
{
  "id": "123",
  "first_name": "Juan",
  "last_name": "Pérez",
  "email": "juan@example.com"
}
```

**Migration**: Parse `name` field and split by space to get `first_name` and `last_name`

### 2. Authentication header changed

**v1**: `Authorization: Bearer {api_key}`

**v2**: `Authorization: Bearer {jwt_token}`

**Migration**: Use `/api/v2/auth/login` to exchange API key for JWT token

## Code Examples

### Python (v1 → v2)

```python
# v1
response = requests.get("/api/v1/users/123", headers={
    "Authorization": f"Bearer {API_KEY}"
})

# v2
# First get JWT token
token_response = requests.post("/api/v2/auth/login", json={
    "api_key": API_KEY
})
jwt_token = token_response.json()["token"]

# Then use JWT token
response = requests.get("/api/v2/users/123", headers={
    "Authorization": f"Bearer {jwt_token}"
})
```
```

## Best Practices

### 1. Pensar en compatibilidad desde el inicio

Diseñar la API pensando en evolución futura:
- Usar campos opcionales para features que pueden cambiar
- Evitar campos que combinen múltiples valores (como `name`)
- Documentar claramente qué campos son estables vs experimentales

### 2. Minimizar breaking changes

Antes de hacer un breaking change, considerar:
- ¿Podemos agregar un campo nuevo en lugar de cambiar uno existente?
- ¿Podemos mantener el campo viejo con deprecation warning?
- ¿Es realmente necesario el cambio o podemos hacerlo compatible?

### 3. Comunicar temprano y frecuentemente

- Anunciar planes de cambios mayores con anticipación
- Solicitar feedback de desarrolladores
- Proporcionar tiempo suficiente para migración

### 4. Mantener versiones antiguas funcionales

- No romper versiones en deprecation
- Aplicar solo security patches críticos
- Documentar claramente qué es soportado vs no

### 5. Monitorear uso de versiones

- Track qué versiones están siendo usadas
- Contactar desarrolladores que usan versiones deprecated
- Usar métricas para decidir cuándo eliminar versiones

## Testing de Versiones

### Tests de Regresión

Para cada nueva versión:
1. Mantener tests de versiones anteriores
2. Agregar tests para nueva versión
3. Verificar que clientes de versión anterior aún funcionan

```python
# tests/integration/test_api_versioning.py
def test_v1_backward_compatibility(client):
    """Verificar que v1 clients aún funcionan"""
    response = client.get("/api/v1/users/123")
    assert response.status_code == 200
    # Verificar estructura de response v1
    assert "name" in response.json()

def test_v2_new_features(client):
    """Verificar nuevas features de v2"""
    response = client.get("/api/v2/users/123")
    assert response.status_code == 200
    # Verificar estructura de response v2
    assert "first_name" in response.json()
    assert "last_name" in response.json()
```

## Referencias

- **ENG-DEF-0001**: System Architecture
- **ENG-REF-0002**: MVP Web API Specification
- **ADR-0006**: FastAPI como Framework de API Web
