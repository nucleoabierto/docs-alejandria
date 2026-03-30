---
id: ENG-GUIDE-0005
area: "engineering"
type: "guide"
title: "Error Handling Standards - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ENG-GUIDE-0004", "ENG-REF-0002", "ENG-REF-0003"]
---

Guía de estándares de manejo de errores para Alejandria. Contiene tipos de errores, estructura de respuestas de error, códigos de error HTTP, manejo de excepciones en servicios y API, logging de errores y mejores prácticas para consistencia en toda la aplicación.

---

# Error Handling Standards - Alejandria

Sabemos que manejar errores de manera consistente es crucial para que los desarrolladores que integran nuestra API tengan una experiencia predecible. Esta guía define estándares claros para manejo de errores en Alejandria, para que cada error sea informativo, consistente y fácil de debuggear.

## Table of Contents

1. [Overview](#overview)
2. [Estructura de Respuesta de Error](#estructura-de-respuesta-de-error)
3. [Códigos de Error HTTP](#códigos-de-error-http)
4. [Jerarquía de Excepciones](#jerarquía-de-excepciones)
5. [Manejo de Errores en Services](#manejo-de-errores-en-services)
6. [Manejo de Errores en API Web (FastAPI)](#manejo-de-errores-en-api-web-fastapi)
7. [Manejo de Errores en MCP Tools](#manejo-de-errores-en-mcp-tools)
8. [Logging de Errores](#logging-de-errores)
9. [Mejores Prácticas](#mejores-prácticas)
10. [Ejemplos Completos](#ejemplos-completos)
11. [Referencias](#referencias)

---

**Principio**: Errores informativos, consistentes y no expuestos  
**Formato**: JSON estructurado para todas las respuestas de error  
**Logging**: Todos los errores se loggean con contexto suficiente  
**Seguridad**: Nunca exponer detalles internos en respuestas al cliente

## Estructura de Respuesta de Error

Todas las respuestas de error (API web y MCP) siguen esta estructura:

```json
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id '123e4567-e89b-12d3-a456-426614174000' not found",
    "details": {
      "field": "user_id",
      "value": "123e4567-e89b-12d3-a456-426614174000"
    }
  }
}
```

**Campos**:
- `code`: Identificador único del error (constante, usada por clientes para programática handling)
- `message`: Mensaje descriptivo en español, legible por humanos
- `details`: Objeto opcional con contexto adicional (field names, values, etc.)

## Códigos de Error

### Errores de Cliente (4xx)

| Código HTTP | Error Code | Descripción | Cuándo usar |
|-------------|------------|-------------|-------------|
| 400 | VALIDATION_ERROR | Request validation failed | Pydantic validation falló |
| 400 | INVALID_REQUEST_FORMAT | Request format invalid | JSON malformado, content-type incorrecto |
| 401 | UNAUTHORIZED | Authentication required | Token JWT missing o inválido |
| 401 | INVALID_API_KEY | API key invalid | API key no existe o fue revocada |
| 403 | FORBIDDEN | Permission denied | Usuario no tiene permiso para el recurso |
| 403 | INSUFFICIENT_PERMISSIONS | Insufficient permissions | ACL denegó acceso |
| 404 | NOT_FOUND | Resource not found | User, organization, document no existe |
| 404 | ENDPOINT_NOT_FOUND | Endpoint not found | Ruta de API no existe |
| 409 | CONFLICT | Resource conflict | Email ya existe, nombre de organización duplicado |
| 409 | DUPLICATE_RESOURCE | Duplicate resource | Intento de crear recurso duplicado |
| 413 | PAYLOAD_TOO_LARGE | Payload too large | Request body excede límite |
| 415 | UNSUPPORTED_MEDIA_TYPE | Unsupported media type | Content-Type no soportado |
| 422 | UNPROCESSABLE_ENTITY | Unprocessable entity | Semántica del request inválida |
| 429 | RATE_LIMIT_EXCEEDED | Rate limit exceeded | Demasiadas requests en ventana de tiempo |

### Errores de Servidor (5xx)

| Código HTTP | Error Code | Descripción | Cuándo usar |
|-------------|------------|-------------|-------------|
| 500 | INTERNAL_SERVER_ERROR | Internal server error | Error no especificado |
| 503 | SERVICE_UNAVAILABLE | Service unavailable | Mantenimiento o sobrecarga |
| 503 | DATABASE_UNAVAILABLE | Database unavailable | PostgreSQL no responde |
| 503 | VECTOR_DB_UNAVAILABLE | Vector database unavailable | Qdrant no responde |
| 504 | GATEWAY_TIMEOUT | Gateway timeout | Timeout upstream service |

## Jerarquía de Excepciones

### Excepciones Base

```python
# core/exceptions.py
class AlejandriaError(Exception):
    """Base exception for all Alejandria errors"""
    def __init__(self, message: str, details: dict = None):
        self.message = message
        self.details = details or {}
        super().__init__(message)

class ValidationError(AlejandriaError):
    """Validation error (400)"""
    status_code = 400
    error_code = "VALIDATION_ERROR"

class AuthenticationError(AlejandriaError):
    """Authentication error (401)"""
    status_code = 401
    error_code = "UNAUTHORIZED"

class PermissionError(AlejandriaError):
    """Permission error (403)"""
    status_code = 403
    error_code = "FORBIDDEN"

class NotFoundError(AlejandriaError):
    """Resource not found (404)"""
    status_code = 404
    error_code = "NOT_FOUND"

class ConflictError(AlejandriaError):
    """Resource conflict (409)"""
    status_code = 409
    error_code = "CONFLICT"

class RateLimitError(AlejandriaError):
    """Rate limit exceeded (429)"""
    status_code = 429
    error_code = "RATE_LIMIT_EXCEEDED"
```

### Excepciones Específicas

```python
# core/exceptions.py (continuación)
class UserNotFoundError(NotFoundError):
    """User not found"""
    error_code = "USER_NOT_FOUND"

class OrganizationNotFoundError(NotFoundError):
    """Organization not found"""
    error_code = "ORGANIZATION_NOT_FOUND"

class DocumentNotFoundError(NotFoundError):
    """Document not found"""
    error_code = "DOCUMENT_NOT_FOUND"

class InvalidAPIKeyError(AuthenticationError):
    """Invalid API key"""
    error_code = "INVALID_API_KEY"

class EmailAlreadyExistsError(ConflictError):
    """Email already exists"""
    error_code = "EMAIL_ALREADY_EXISTS"

class ACLDeniedError(PermissionError):
    """ACL denied access"""
    error_code = "INSUFFICIENT_PERMISSIONS"

class DatabaseUnavailableError(AlejandriaError):
    """Database unavailable"""
    status_code = 503
    error_code = "DATABASE_UNAVAILABLE"

class VectorDBUnavailableError(AlejandriaError):
    """Vector database unavailable"""
    status_code = 503
    error_code = "VECTOR_DB_UNAVAILABLE"
```

## Manejo de Errores en Services

Los services deben lanzar excepciones específicas:

```python
# services/user_service.py
from alejandria.core.exceptions import (
    UserNotFoundError,
    EmailAlreadyExistsError,
    InvalidAPIKeyError
)

class UserService:
    def get_user(self, user_id: str) -> User:
        user = self.user_repo.get_by_id(user_id)
        if not user:
            raise UserNotFoundError(
                message=f"User with id '{user_id}' not found",
                details={"field": "user_id", "value": user_id}
            )
        return user
    
    def create_user(self, data: UserCreate) -> User:
        # Verificar si email ya existe
        existing = self.user_repo.get_by_email(data.email)
        if existing:
            raise EmailAlreadyExistsError(
                message=f"User with email '{data.email}' already exists",
                details={"field": "email", "value": data.email}
            )
        
        user = User(**data.dict())
        return self.user_repo.create(user)
    
    def authenticate(self, api_key: str) -> User:
        user = self.user_repo.get_by_api_key(api_key)
        if not user:
            raise InvalidAPIKeyError(
                message="Invalid API key",
                details={"field": "api_key"}
            )
        return user
```

## Manejo de Errores en API Web

### Middleware de Error Handler

```python
# api/middleware/error_handler.py
from fastapi import Request, status
from fastapi.responses import JSONResponse
from alejandria.core.exceptions import AlejandriaError

async def alejandria_error_handler(request: Request, exc: AlejandriaError) -> JSONResponse:
    """Handler para excepciones de Alejandria"""
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.error_code,
                "message": exc.message,
                "details": exc.details
            }
        }
    )

async def validation_error_handler(request: Request, exc: ValidationError) -> JSONResponse:
    """Handler para errores de validación de Pydantic"""
    errors = []
    for error in exc.errors():
        errors.append({
            "field": ".".join(str(loc) for loc in error["loc"]),
            "message": error["msg"],
            "type": error["type"]
        })
    
    return JSONResponse(
        status_code=status.HTTP_422_UNPROCESSABLE_ENTITY,
        content={
            "error": {
                "code": "VALIDATION_ERROR",
                "message": "Request validation failed",
                "details": {"errors": errors}
            }
        }
    )

async def generic_error_handler(request: Request, exc: Exception) -> JSONResponse:
    """Handler para errores genéricos no manejados"""
    logger.error(f"Unhandled exception: {exc}", exc_info=True)
    
    return JSONResponse(
        status_code=status.HTTP_500_INTERNAL_SERVER_ERROR,
        content={
            "error": {
                "code": "INTERNAL_SERVER_ERROR",
                "message": "An unexpected error occurred",
                "details": {}
            }
        }
    )
```

### Registro en FastAPI App

```python
# api/app.py
from fastapi import FastAPI
from alejandria.api.middleware.error_handler import (
    alejandria_error_handler,
    validation_error_handler,
    generic_error_handler
)
from pydantic import ValidationError

app = FastAPI()

# Registrar exception handlers
app.add_exception_handler(AlejandriaError, alejandria_error_handler)
app.add_exception_handler(ValidationError, validation_error_handler)
app.add_exception_handler(Exception, generic_error_handler)
```

## Manejo de Errores en MCP Server

### MCP Tool Error Responses

```python
# mcp/tools/documents.py
from alejandria.core.exceptions import (
    DocumentNotFoundError,
    ACLDeniedError,
    ValidationError
)

@mcp.tool()
async def get_document(document_id: str) -> dict:
    """Get a document by ID"""
    try:
        document = document_service.get_document(document_id)
        return document_to_dict(document)
    except DocumentNotFoundError as e:
        # MCP tools retornan errores como dict con field "error"
        return {
            "error": {
                "code": e.error_code,
                "message": e.message,
                "details": e.details
            }
        }
    except ACLDeniedError as e:
        return {
            "error": {
                "code": e.error_code,
                "message": e.message,
                "details": e.details
            }
        }
```

## Logging de Errores

### Estructura de Log

Todos los errores deben loggearse con contexto suficiente:

```python
import logging
import uuid

logger = logging.getLogger(__name__)

def get_user(user_id: str):
    request_id = str(uuid.uuid4())
    
    try:
        user = user_repo.get_by_id(user_id)
        if not user:
            logger.error(
                "User not found",
                extra={
                    "request_id": request_id,
                    "user_id": user_id,
                    "error_code": "USER_NOT_FOUND"
                }
            )
            raise UserNotFoundError(...)
        
        logger.info(
            "User retrieved successfully",
            extra={
                "request_id": request_id,
                "user_id": user_id
            }
        )
        return user
    
    except Exception as e:
        logger.error(
            "Unexpected error retrieving user",
            extra={
                "request_id": request_id,
                "user_id": user_id,
                "error_type": type(e).__name__,
                "error_message": str(e)
            },
            exc_info=True
        )
        raise
```

### Niveles de Log

| Nivel | Cuándo usar | Ejemplo |
|-------|-------------|---------|
| DEBUG | Información detallada para debugging | SQL queries, valores de variables |
| INFO | Eventos normales de operación | User creado, documento actualizado |
| WARNING | Eventos inesperados pero no críticos | Retry de operación, cache miss |
| ERROR | Errores que no rompen el flujo | User no encontrado (404) |
| CRITICAL | Errores que requieren atención inmediata | Database unavailable, OOM |

## Errores Específicos por Dominio

### Authentication

```python
# Invalid API key
{
  "error": {
    "code": "INVALID_API_KEY",
    "message": "Invalid API key",
    "details": {}
  }
}

# Token expired
{
  "error": {
    "code": "TOKEN_EXPIRED",
    "message": "JWT token has expired",
    "details": {
      "expired_at": "2026-04-24T12:00:00Z"
    }
  }
}
```

### Users

```python
# User not found
{
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id '123e4567-e89b-12d3-a456-426614174000' not found",
    "details": {
      "field": "user_id",
      "value": "123e4567-e89b-12d3-a456-426614174000"
    }
  }
}

# Email already exists
{
  "error": {
    "code": "EMAIL_ALREADY_EXISTS",
    "message": "User with email 'juan@example.com' already exists",
    "details": {
      "field": "email",
      "value": "juan@example.com"
    }
  }
}
```

### Organizations

```python
# Organization not found
{
  "error": {
    "code": "ORGANIZATION_NOT_FOUND",
    "message": "Organization with id 'abc-123' not found",
    "details": {
      "field": "organization_id",
      "value": "abc-123"
    }
  }
}

# Not organization member
{
  "error": {
    "code": "NOT_ORGANIZATION_MEMBER",
    "message": "User is not a member of organization 'abc-123'",
    "details": {
      "user_id": "123e4567-e89b-12d3-a456-426614174000",
      "organization_id": "abc-123"
    }
  }
}
```

### Documents

```python
# Document not found
{
  "error": {
    "code": "DOCUMENT_NOT_FOUND",
    "message": "Document with id 'doc-456' not found",
    "details": {
      "field": "document_id",
      "value": "doc-456"
    }
  }
}

# ACL denied
{
  "error": {
    "code": "INSUFFICIENT_PERMISSIONS",
    "message": "User does not have permission to access document 'doc-456'",
    "details": {
      "user_id": "123e4567-e89b-12d3-a456-426614174000",
      "document_id": "doc-456",
      "required_permission": "read"
    }
  }
}
```

## Best Practices

### 1. Siempre usar excepciones específicas

❌ **Mal**:
```python
raise Exception("User not found")
```

✅ **Bien**:
```python
raise UserNotFoundError(
    message=f"User with id '{user_id}' not found",
    details={"field": "user_id", "value": user_id}
)
```

### 2. Nunca exponer detalles internos

❌ **Mal**:
```python
return JSONResponse(
    content={"error": {"message": str(exc), "traceback": traceback.format_exc()}}
)
```

✅ **Bien**:
```python
return JSONResponse(
    content={"error": {"code": "INTERNAL_SERVER_ERROR", "message": "An unexpected error occurred"}}
)
logger.error(f"Internal error: {exc}", exc_info=True)
```

### 3. Incluir contexto en detalles

❌ **Mal**:
```python
raise UserNotFoundError("User not found")
```

✅ **Bien**:
```python
raise UserNotFoundError(
    message=f"User with id '{user_id}' not found",
    details={"field": "user_id", "value": user_id}
)
```

### 4. Loggear antes de lanzar excepción

```python
logger.error(
    "User not found",
    extra={"user_id": user_id, "request_id": request_id}
)
raise UserNotFoundError(...)
```

### 5. Manejar errores en capas apropiadas

- **Services**: Lanzan excepciones de negocio
- **API/MCP**: Capturan excepciones y convierten a respuestas HTTP/MCP
- **Repositories**: Lanzan excepciones de datos (DatabaseError, etc.)

### 6. Usar códigos de error constantes

Los clientes pueden programáticamente manejar errores basándose en `error_code`:

```python
# Cliente API
response = requests.post("/api/v1/users", json=data)
if response.status_code == 409:
    error = response.json()["error"]
    if error["code"] == "EMAIL_ALREADY_EXISTS":
        # Mostrar mensaje al usuario
        show_error("Este email ya está registrado")
```

## Testing de Error Handling

### Tests Unitarios

```python
# tests/unit/test_user_service.py
def test_get_user_not_found(user_service, user_repo):
    user_repo.get_by_id.return_value = None
    
    with pytest.raises(UserNotFoundError) as exc_info:
        user_service.get_user("nonexistent-id")
    
    assert exc_info.value.error_code == "USER_NOT_FOUND"
    assert "nonexistent-id" in exc_info.value.details["value"]
```

### Tests de Integración

```python
# tests/integration/test_api/test_users.py
def test_get_user_not_found(client):
    response = client.get("/api/v1/users/nonexistent-id")
    
    assert response.status_code == 404
    error = response.json()["error"]
    assert error["code"] == "USER_NOT_FOUND"
    assert "nonexistent-id" in error["details"]["value"]
```

## Referencias

- **ENG-DEF-0001**: System Architecture
- **ENG-GUIDE-0004**: Code Structure and Architecture Guide
- **ENG-REF-0002**: MVP Web API Specification
- **ENG-REF-0003**: API Authentication
