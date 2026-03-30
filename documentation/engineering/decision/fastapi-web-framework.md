---
id: ADR-0006
area: "engineering"
type: "decision"
title: "FastAPI como Framework de API Web"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0002", "PRD-0001", "ENG-REF-0003", "ENG-REF-0004", "ENG-REF-0005"]
---

Registro de decisión arquitectónica que justifica la selección de FastAPI como framework para la API web REST de Alejandria. Contiene contexto del problema de gestión de usuarios y organizaciones vía API web, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0006: FastAPI como Framework de API Web

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que necesitamos una API web REST para gestión de usuarios, organizaciones y operaciones de back office en Alejandia. Los endpoints MCP (implementados con FastMCP) se enfocan exclusivamente en gestión de documentos, pero las operaciones de usuario/organización requieren una API web tradicional con autenticación JWT, rate limiting, y documentación interactiva.

El desafío técnico es que necesitamos un framework web que:
- Se integre con el stack Python existente (FastMCP, SQLAlchemy, sentence-transformers)
- Proporcione autenticación JWT robusta
- Genere documentación automática (Swagger/ReDoc)
- Soporte async/await para performance
- Valide requests/responses automáticamente (type-safety)
- Sea rápido y escalable para APIs REST

Entendemos que necesitamos:
- Framework moderno y mantenido activamente
- Integración sencilla con PostgreSQL (SQLAlchemy)
- Soporte para middleware (CORS, rate limiting, logging)
- Documentación automática para desarrolladores externos
- Ecosistema maduro con plugins y extensions

## Decision

Usar FastAPI como framework para la API web REST de Alejandria.

**Especificación técnica:**
- **Framework**: FastAPI v0.104+
- **Python**: 3.11+ (mismo que FastMCP backend)
- **Autenticación**: JWT tokens con `python-jose`
- **Validación**: Pydantic v2 para schemas y validación
- **Documentación**: Swagger UI (por defecto en `/docs`) y ReDoc (en `/redoc`)
- **ORM**: SQLAlchemy 2.0 con async support
- **Middleware**: CORS, rate limiting, structured logging

FastAPI proporciona:
- Declaración de endpoints con decoradores tipo Python (piensa en esto como decoradores que hacen el trabajo pesado por ti)
- Validación automática de requests/responses con Pydantic
- Documentación interactiva automática (OpenAPI 3.0)
- Soporte nativo para async/await
- Type hints para IDE autocomplete
- Performance comparable a Node.js y Go
- Ecosistema maduro con middleware y plugins

## Consequences

### Positivas

✅ **Integración con stack Python existente**: FastAPI se integra perfectamente con FastMCP (ambos Python), SQLAlchemy (ya usado en ADR-0003), y el ecosistema Python de ML/AI (sentence-transformers). Un solo lenguaje para todo el stack reduce complejidad.

✅ **Validación automática con Pydantic**: FastAPI usa Pydantic v2 para validar requests y serializar responses automáticamente. Si un request no cumple el schema, retorna error 422 con detalles. Esto reduce código de validación manual y bugs.

✅ **Documentación automática**: FastAPI genera documentación OpenAPI 3.0 automáticamente basándose en type hints y Pydantic models. Swagger UI en `/docs` y ReDoc en `/redoc` están disponibles sin configuración adicional.

✅ **Soporte nativo async/await**: FastAPI soporta async endpoints nativamente, permitiendo manejar múltiples requests concurrentes sin bloquear. Esto es crítico para performance en APIs que llaman a bases de datos y servicios externos.

✅ **Type safety y IDE autocomplete**: Type hints y Pydantic models permiten autocomplete en IDEs (VS Code, PyCharm), reduciendo errores de desarrollo y mejorando velocidad de desarrollo.

✅ **Performance**: FastAPI es uno de los frameworks Python más rápidos, comparable a Node.js y Go en benchmarks. Usa Starlette (ASGI) para alto rendimiento y uvicorn como servidor ASGI.

✅ **Ecosistema maduro**: FastAPI tiene middleware para CORS, rate limiting, structured logging, authentication, y más. La comunidad es activa y hay muchos ejemplos y tutorials.

✅ **Testing sencillo**: FastAPI incluye `TestClient` que permite testear endpoints sin necesidad de servidor real, facilitando unit tests e integration tests.

### Negativas

❌ **Más joven que Flask/Django**: FastAPI (2018) es más joven que Flask (2010) y Django (2005). Sin embargo, tiene adopción creciente y es estable para producción.

❌ **Curva de aprendizaje para async**: Desarrolladores acostumbrados a frameworks síncronos (Flask) necesitan aprender async/await. Esto es mitigado por la simplicidad de FastAPI y abundante documentación.

❌ **Menos plugins que Django**: FastAPI tiene menos plugins/batteries-included que Django (admin panel, ORM integrado, etc.). Esto es aceptable porque solo necesitamos una API REST, no un full-stack framework.

❌ **Overhead de validación Pydantic**: Validación automática añade ligero overhead (~1-2ms por request). Esto es aceptable para la mayoría de use cases y mitigable por performance de FastAPI.

## Alternatives Considered

### Flask

**Por qué no la elegimos:**
- No soporta async/await nativamente (requiere extensions como Quart)
- Validación de requests requiere código manual o extensions (Marshmallow, Flask-RESTful)
- Documentación automática requiere extensions (Flasgger, Flask-RESTX)
- Menos type-safe que FastAPI
- Performance menor que FastAPI en benchmarks

**Cuándo podría ser mejor:**
- Si el equipo tuviera más experiencia con Flask
- Si el proyecto fuera extremadamente simple (CRUD básico sin validación compleja)
- Si necesitáramos micro-framework con mínimos dependencies

### Django REST Framework

**Por qué no la elegimos:**
- Django es full-stack framework con ORM, admin panel, templates, etc. (overkill para API REST)
- DRF añade overhead adicional sobre Django
- Configuración más compleja que FastAPI
- Menos performance que FastAPI en benchmarks
- Async support es limitado (Django 4.1+ tiene async views pero no async ORM completo)

**Cuándo podría ser mejor:**
- Si necesitáramos admin panel para back office
- Si el proyecto tuviera dependencia existente en Django
- Si necesitáramos features de Django full-stack (templates, auth integrado, etc.)

### Express.js (Node.js)

**Por qué no la elegimos:**
- Introduciría JavaScript/TypeScript al stack, añadiendo complejidad
- No se integra con ecosistema Python de ML/AI (sentence-transformers, etc.)
- Requeriría mantener dos lenguajes (Python para MCP/ML, JS para API web)
- Validación de schemas requiere libraries adicionales (Joi, Zod)
- Menos type-safe que FastAPI + Pydantic (TypeScript ayuda pero no es nativo)

**Cuándo podría ser mejor:**
- Si el equipo tuviera más experiencia en Node.js
- Si el proyecto tuviera dependencia existente en Node.js
- Si necesitáramos isomorfismo (mismo lenguaje en frontend y backend)

## Implementation

### Paso 1: Instalación de dependencias

```bash
# requirements.txt (adicionar a existente)
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
python-jose[cryptography]>=3.3.0
passlib[bcrypt]>=1.7.4
python-multipart>=0.0.6
```

### Paso 2: Estructura de directorios

```
backend/
├── alejandria/
│   ├── __init__.py
│   ├── main.py              # Entry point FastMCP (existente)
│   ├── api/
│   │   ├── __init__.py
│   │   ├── app.py           # FastAPI app
│   │   ├── dependencies.py  # Dependency injection
│   │   ├── auth.py          # JWT authentication
│   │   ├── v1/
│   │   │   ├── __init__.py
│   │   │   ├── users.py     # User endpoints
│   │   │   ├── orgs.py      # Organization endpoints
│   │   │   └── health.py    # Health check
│   │   └── schemas.py       # Pydantic models
│   └── db/
│       ├── __init__.py
│       ├── models.py        # SQLAlchemy models
│       └── session.py       # Database session
└── requirements.txt
```

### Paso 3: FastAPI app básica

```python
# alejandria/api/app.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from alejandria.api.v1 import users, orgs, health

app = FastAPI(
    title="Alejandria API",
    description="API web para gestión de usuarios y organizaciones",
    version="1.0.0",
    docs_url="/docs",
    redoc_url="/redoc"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Configurar apropiadamente en producción
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(users.router, prefix="/v1/users", tags=["users"])
app.include_router(orgs.router, prefix="/v1/organizations", tags=["organizations"])
app.include_router(health.router, prefix="/v1", tags=["health"])

@app.get("/")
async def root():
    return {"message": "Alejandria API v1.0.0"}
```

### Paso 4: Pydantic schemas para validación

```python
# alejandria/api/schemas.py
from pydantic import BaseModel, EmailStr, Field
from typing import Optional
from datetime import datetime
import uuid

class UserCreate(BaseModel):
    email: EmailStr
    name: str = Field(..., max_length=100)
    password: str = Field(..., min_length=8, max_length=128)

class UserResponse(BaseModel):
    user_id: uuid.UUID
    email: EmailStr
    name: str
    created_at: datetime

class OrganizationCreate(BaseModel):
    name: str = Field(..., max_length=100)
    type: str = Field(..., pattern="^(personal|team|enterprise)$")

class OrganizationResponse(BaseModel):
    organization_id: uuid.UUID
    name: str
    type: str
    owner_id: uuid.UUID
    created_at: datetime
```

### Paso 5: Endpoints con validación automática

```python
# alejandria/api/v1/users.py
from fastapi import APIRouter, Depends, HTTPException
from alejandria.api.schemas import UserCreate, UserResponse
from alejandria.db.models import User
from alejandria.db.session import get_db
from sqlalchemy.ext.asyncio import AsyncSession

router = APIRouter()

@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    user: UserCreate,
    db: AsyncSession = Depends(get_db)
):
    # Validación automática de UserCreate por Pydantic
    # Si email es inválido o password < 8 chars, FastAPI retorna 422 automáticamente
    
    # Verificar si email ya existe
    existing_user = await db.get(User, email=user.email)
    if existing_user:
        raise HTTPException(status_code=400, detail="email already exists")
    
    # Crear usuario
    new_user = User(
        email=user.email,
        name=user.name,
        password_hash=hash_password(user.password)  # Implementar hash_password
    )
    db.add(new_user)
    await db.commit()
    await db.refresh(new_user)
    
    # Response automáticamente serializado por Pydantic
    return UserResponse(
        user_id=new_user.id,
        email=new_user.email,
        name=new_user.name,
        created_at=new_user.created_at
    )
```

### Paso 6: Autenticación JWT

```python
# alejandria/api/auth.py
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials

SECRET_KEY = "your-secret-key-here"  # Usar environment variable
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security)
):
    token = credentials.credentials
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="invalid token")
    except JWTError:
        raise HTTPException(status_code=401, detail="invalid token")
    
    # Buscar usuario en base de datos
    # user = await get_user_by_id(user_id)
    # if user is None:
    #     raise HTTPException(status_code=401, detail="user not found")
    
    return user_id  # Retornar user object completo en implementación real
```

### Paso 7: Deployment con Uvicorn

```python
# alejandria/api/run.py
import uvicorn

if __name__ == "__main__":
    uvicorn.run(
        "alejandria.api.app:app",
        host="0.0.0.0",
        port=8000,
        reload=True,  # Solo en desarrollo
        workers=1     # Aumentar en producción
    )
```

```bash
# Ejecutar en desarrollo
python -m alejandria.api.run

# Ejecutar con uvicorn directamente
uvicorn alejandria.api.app:app --reload --host 0.0.0.0 --port 8000
```

### Paso 8: Testing

```python
# tests/test_api.py
from fastapi.testclient import TestClient
from alejandria.api.app import app

client = TestClient(app)

def test_create_user():
    response = client.post(
        "/v1/users",
        json={
            "email": "test@example.com",
            "name": "Test User",
            "password": "securepassword123"
        }
    )
    assert response.status_code == 201
    data = response.json()
    assert data["success"] is True
    assert data["data"]["email"] == "test@example.com"

def test_create_user_invalid_email():
    response = client.post(
        "/v1/users",
        json={
            "email": "invalid-email",
            "name": "Test User",
            "password": "securepassword123"
        }
    )
    assert response.status_code == 422  # Pydantic validation error

def test_create_user_short_password():
    response = client.post(
        "/v1/users",
        json={
            "email": "test@example.com",
            "name": "Test User",
            "password": "short"
        }
    )
    assert response.status_code == 422  # Pydantic validation error
```

### Paso 9: Integración con Docker Compose

```yaml
# docker-compose.yml (actualizar)
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: alejandria-backend
    environment:
      DATABASE_URL: postgresql://alejandria:${POSTGRES_PASSWORD:-alejandria_dev}@postgres:5432/alejandria
      QDRANT_HOST: qdrant
      QDRANT_PORT: 6333
    ports:
      - "8000:8000"  # FastAPI
    depends_on:
      postgres:
        condition: service_healthy
      qdrant:
        condition: service_healthy
    volumes:
      - ./backend:/app
    # Command para ejecutar ambos FastMCP y FastAPI
    command: >
      sh -c "
      python -m alejandria.main &
      uvicorn alejandria.api.app:app --host 0.0.0.0 --port 8000
      "
```

## Related Decisions

- **ADR-0002**: Python + FastMCP como backend framework (FastAPI complementa el stack Python)
- **ADR-0003**: PostgreSQL como base de datos relacional (FastAPI se integra con SQLAlchemy)
- **PRD-0001**: Organization and User Management (define requerimientos de API web)
- **ENG-REF-0002**: MVP Web API Specification (especificación detallada de endpoints)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Madurez de FastAPI:**
- FastAPI es estable y ampliamente adoptado en producción (usado por Uber, Netflix, Microsoft)
- Comunidad activa con 60k+ stars en GitHub
- Documentación excelente y muchos ejemplos

**Curva de aprendizaje async:**
- FastAPI permite endpoints síncronos si es necesario (def sin async)
- Documentación clara sobre async/await
- Equipo puede aprender gradualmente

**Menos plugins que Django:**
- Solo necesitamos API REST, no full-stack features
- FastAPI tiene middleware para CORS, rate limiting, logging
- Si necesitáramos admin panel, podemos usar herramientas separadas (Adminer, pgAdmin)

**Overhead de validación Pydantic:**
- Overhead es mínimo (~1-2ms) comparado con beneficios (reducción de bugs, documentación automática)
- Performance de FastAPI compensa este overhead
- Podemos desactivar validación en endpoints críticos de performance si es necesario
