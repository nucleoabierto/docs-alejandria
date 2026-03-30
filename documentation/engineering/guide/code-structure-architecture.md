---
id: ENG-GUIDE-0004
area: "engineering"
type: "guide"
title: "Code Structure and Architecture Guide - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0001", "ADR-0002", "ADR-0003", "ADR-0004", "ADR-0005", "ADR-0006"]
---

Guía de estructura de código y arquitectura para Alejandria. Contiene organización de directorios, módulos principales, patrones de diseño, convenciones de naming y flujo de datos entre componentes para facilitar la implementación y mantenimiento del sistema.

---

# Code Structure and Architecture Guide - Alejandria

Esta guía proporciona una visión completa de la estructura de código y arquitectura de Alejandria. Explica la organización de directorios, módulos principales, convenciones de naming, patrones de diseño y flujo de datos end-to-end.

## Table of Contents

1. [Overview](#overview)
2. [Estructura de Directorios](#estructura-de-directorios)
3. [Módulos Principales](#módulos-principales)
4. [Convenciones de Naming](#convenciones-de-naming)
5. [Patrones de Diseño](#patrones-de-diseño)
6. [Flujo de Datos End-to-End](#flujo-de-datos-end-to-end)
7. [Testing Strategy](#testing-strategy)
8. [Best Practices](#best-practices)
9. [Common Tasks](#common-tasks)
10. [Referencias](#referencias)

---

Sabemos que entender la estructura de código de un proyecto nuevo puede ser abrumador. Esta guía te proporciona un mapa mental claro de cómo está organizado Alejandria, para que puedas navegar el códigobase con confianza y saber dónde poner cada pieza.

## Overview

**Lenguaje**: Python 3.11+  
**Arquitectura**: Monolito modular con MCP server y API web REST  
**Patrón principal**: Clean Architecture con separación de concerns  
**Testing**: pytest con estructura espejada al código fuente

## Estructura de Directorios

```
alejandria/
├── alejandria/                    # Paquete principal
│   ├── __init__.py
│   ├── main.py                    # Entry point para FastMCP server
│   ├── api/                       # API web REST (FastAPI)
│   │   ├── __init__.py
│   │   ├── app.py                 # FastAPI application
│   │   ├── dependencies.py       # Dependencies injection
│   │   ├── routers/               # API route handlers
│   │   │   ├── __init__.py
│   │   │   ├── auth.py           # Authentication endpoints
│   │   │   ├── users.py          # User management
│   │   │   ├── organizations.py  # Organization management
│   │   │   └── back_office.py    # Health checks, metrics
│   │   ├── middleware/            # Custom middleware
│   │   │   ├── __init__.py
│   │   │   ├── auth.py           # JWT authentication
│   │   │   ├── error_handler.py  # Global error handling
│   │   │   └── rate_limit.py     # Rate limiting
│   │   └── schemas/              # Pydantic models for API
│   │       ├── __init__.py
│   │       ├── auth.py
│   │       ├── users.py
│   │       └── organizations.py
│   ├── mcp/                       # MCP server implementation
│   │   ├── __init__.py
│   │   ├── server.py              # FastMCP server setup
│   │   ├── tools/                 # MCP tools
│   │   │   ├── __init__.py
│   │   │   ├── documents.py       # Document CRUD operations
│   │   │   └── search.py          # Semantic search
│   │   └── schemas/              # MCP tool schemas
│   │       ├── __init__.py
│   │       └── documents.py
│   ├── core/                      # Core business logic
│   │   ├── __init__.py
│   │   ├── config.py              # Configuration management
│   │   ├── security.py           # Security utilities (JWT, hashing)
│   │   └── acl.py                 # ACL validation logic
│   ├── db/                        # Database layer
│   │   ├── __init__.py
│   │   ├── session.py             # Database session management
│   │   ├── base.py                # Base SQLAlchemy model
│   │   └── models/                # SQLAlchemy models
│   │       ├── __init__.py
│   │       ├── user.py
│   │       ├── organization.py
│   │       ├── document.py
│   │       └── document_version.py
│   ├── services/                  # Business logic services
│   │   ├── __init__.py
│   │   ├── user_service.py        # User business logic
│   │   ├── org_service.py         # Organization business logic
│   │   ├── document_service.py    # Document business logic
│   │   └── search_service.py      # Semantic search logic
│   ├── repositories/              # Data access layer
│   │   ├── __init__.py
│   │   ├── user_repository.py
│   │   ├── org_repository.py
│   │   └── document_repository.py
│   ├── vector/                    # Vector database operations
│   │   ├── __init__.py
│   │   ├── client.py              # Qdrant client setup
│   │   └── embeddings.py          # Embedding generation
│   └── utils/                     # Utilities
│       ├── __init__.py
│       └── helpers.py             # Helper functions
├── tests/                         # Test directory
│   ├── __init__.py
│   ├── conftest.py                # Pytest configuration and fixtures
│   ├── unit/                      # Unit tests
│   │   ├── __init__.py
│   │   ├── test_services/
│   │   ├── test_repositories/
│   │   └── test_core/
│   ├── integration/               # Integration tests
│   │   ├── __init__.py
│   │   ├── test_api/
│   │   ├── test_mcp/
│   │   └── test_db/
│   └── e2e/                       # End-to-end tests
│       ├── __init__.py
│       └── test_flows/
├── alembic/                       # Database migrations
│   ├── versions/
│   └── env.py
├── docker-compose.yml              # Local development setup
├── Dockerfile                      # Container image
├── pyproject.toml                 # Python dependencies
├── pytest.ini                     # Pytest configuration
└── README.md
```

## Módulos Principales

### api/ - API Web REST

**Propósito**: Exponer endpoints HTTP para gestión de usuarios, organizaciones y back office.

**Componentes clave**:
- `app.py`: FastAPI application con middleware, exception handlers, routers
- `routers/`: Route handlers organizados por dominio (auth, users, organizations)
- `schemas/`: Pydantic models para request/response validation
- `middleware/`: Cross-cutting concerns (auth, error handling, rate limiting)

**Flujo de datos**:
```
HTTP Request → Middleware (auth/rate limit) → Router → Service → Repository → DB
HTTP Response ← Pydantic validation ← Service ← Repository ← DB
```

### mcp/ - MCP Server

**Propósito**: Servidor MCP que expone herramientas para agentes de IA (Claude, Cursor).

**Componentes clave**:
- `server.py`: FastMCP server setup con tool registration
- `tools/`: MCP tools implementados (create_document, search_documents, etc.)
- `schemas/`: Schemas para MCP tool inputs/outputs

**Flujo de datos**:
```
MCP Request → FastMCP server → Tool → Service → Repository → DB
MCP Response ← Tool validation ← Service ← Repository ← DB
```

### core/ - Core Business Logic

**Propósito**: Lógica de negocio central y configuración.

**Componentes clave**:
- `config.py`: Configuration management con environment variables
- `security.py`: JWT token generation/validation, password hashing
- `acl.py`: ACL validation logic (visibility, permissions)

### db/ - Database Layer

**Propósito**: Abstracción de base de datos PostgreSQL.

**Componentes clave**:
- `session.py`: Database session management con SQLAlchemy
- `base.py`: Base SQLAlchemy model con timestamps
- `models/`: SQLAlchemy models que mapean a tablas PostgreSQL

**Relación con schema**:
- `models/user.py` → tabla `users` (ENG-REF-0007)
- `models/organization.py` → tabla `organizations` (ENG-REF-0007)
- `models/document.py` → tabla `documents` (ENG-REF-0007)
- `models/document_version.py` → tabla `document_versions` (ENG-REF-0007)

### services/ - Business Logic Services

**Propósito**: Implementar lógica de negocio que orquesta repositories y otros servicios.

**Componentes clave**:
- `user_service.py`: User creation, API key generation, profile updates
- `org_service.py`: Organization creation, member management
- `document_service.py`: Document CRUD, ACL enforcement, versioning
- `search_service.py`: Semantic search with ACL filtering

**Patrón**: Services no conocen de HTTP ni MCP, solo lógica de negocio pura.

### repositories/ - Data Access Layer

**Propósito**: Abstracción de acceso a datos (PostgreSQL y Qdrant).

**Componentes clave**:
- `user_repository.py`: CRUD operations para users
- `org_repository.py`: CRUD operations para organizations
- `document_repository.py`: CRUD operations para documents con ACL queries

**Patrón**: Repositories encapsulan queries complejas y retornan domain objects.

### vector/ - Vector Database Operations

**Propósito**: Integración con Qdrant para embeddings y búsqueda semántica.

**Componentes clave**:
- `client.py`: Qdrant client setup y connection management
- `embeddings.py`: Embedding generation con sentence-transformers

**Flujo de datos**:
```
Document text → Embeddings model → Vector → Qdrant
Search query → Embeddings model → Vector → Qdrant → Results
```

## Convenciones de Naming

### Archivos

- **Snake_case**: `user_service.py`, `document_repository.py`
- **Descriptivos**: `auth.py` en lugar de `security.py` para authentication
- **Singular**: `user.py` en lugar de `users.py` para models

### Clases

- **PascalCase**: `UserService`, `DocumentRepository`, `UserCreate`
- **Sustantivos**: `User`, `Organization`, `Document`
- **Sufijos consistentes**: 
  - Services: `*Service` (UserService, DocumentService)
  - Repositories: `*Repository` (UserRepository, DocumentRepository)
  - Models: sin sufijo (User, Organization, Document)
  - Schemas: descriptivos (UserCreate, UserResponse, DocumentCreate)

### Funciones y Variables

- **Snake_case**: `get_user_by_id`, `create_document`, `is_valid_acl`
- **Verbos para funciones**: `get_`, `create_`, `update_`, `delete_`, `is_`, `has_`
- **Descriptivos**: `user_with_org` en lugar de `uwo`

### Constantes

- **UPPER_SNAKE_CASE**: `MAX_DOCUMENTS_PER_ORG`, `DEFAULT_PAGE_SIZE`
- **En `config.py`**: Configuration constants
- **En `core/`**: Business constants

## Patrones de Diseño

### Dependency Injection

**Propósito**: Facilitar testing y desacoplamiento.

**Implementación**:
```python
# api/dependencies.py
from fastapi import Depends
from alejandria.db.session import get_db
from alejandria.services.user_service import UserService

def get_user_service(db = Depends(get_db)) -> UserService:
    return UserService(db)

# Uso en router
@router.post("/users")
async def create_user(
    user_data: UserCreate,
    user_service: UserService = Depends(get_user_service)
):
    return user_service.create_user(user_data)
```

### Repository Pattern

**Propósito**: Abstraer acceso a datos y facilitar testing.

**Implementación**:
```python
# repositories/document_repository.py
class DocumentRepository:
    def __init__(self, db: Session):
        self.db = db
    
    def get_by_id(self, doc_id: str) -> Optional[Document]:
        return self.db.query(Document).filter(Document.id == doc_id).first()
    
    def create(self, document: Document) -> Document:
        self.db.add(document)
        self.db.commit()
        self.db.refresh(document)
        return document
```

### Service Layer Pattern

**Propósito**: Encapsular lógica de negocio y orquestar repositories.

**Implementación**:
```python
# services/document_service.py
class DocumentService:
    def __init__(self, doc_repo: DocumentRepository, acl: ACLValidator):
        self.doc_repo = doc_repo
        self.acl = acl
    
    def create_document(self, user_id: str, org_id: str, data: DocumentCreate) -> Document:
        # Validar permisos
        if not self.acl.can_create_document(user_id, org_id):
            raise PermissionDeniedError()
        
        # Crear documento
        document = Document(**data.dict(), owner_id=user_id, organization_id=org_id)
        return self.doc_repo.create(document)
```

### Factory Pattern

**Propósito**: Crear objetos complejos con configuración.

**Implementación**:
```python
# vector/client.py
def get_qdrant_client() -> QdrantClient:
    config = get_config()
    return QdrantClient(
        host=config.qdrant_host,
        port=config.qdrant_port,
        api_key=config.qdrant_api_key
    )
```

## Flujo de Datos End-to-End

### Creación de Documento via MCP

```
1. Agente IA llama a MCP tool "create_document"
   ↓
2. FastMCP server valida request con Pydantic schema
   ↓
3. DocumentService.create_document() valida ACL
   ↓
4. DocumentRepository crea registro en PostgreSQL
   ↓
5. EmbeddingsService genera vector
   ↓
6. QdrantClient almacena embedding con metadata
   ↓
7. FastMCP retorna document_id al agente
```

### Búsqueda Semántica via MCP

```
1. Agente IA llama a MCP tool "search_documents"
   ↓
2. FastMCP server valida request
   ↓
3. SearchService genera embedding de query
   ↓
4. QdrantClient busca vectores similares con ACL filter
   ↓
5. DocumentRepository enriches results con PostgreSQL metadata
   ↓
6. ACLValidator filtra resultados por permisos del usuario
   ↓
7. FastMCP retorna documentos al agente
```

### Creación de Usuario via API Web

```
1. HTTP POST /api/v1/users
   ↓
2. Middleware rate_limit verifica límites
   ↓
3. Router users.py valida request con Pydantic
   ↓
4. UserService.create_user() genera API key
   ↓
5. UserRepository crea registro en PostgreSQL
   ↓
6. Pydantic UserResponse serializa respuesta
   ↓
7. FastAPI retorna 201 Created con user data
```

## Testing Strategy

### Estructura de Tests

```
tests/
├── unit/                      # Tests aislados sin dependencias externas
│   ├── test_services/         # Test business logic con mocks
│   ├── test_repositories/     # Test data access con in-memory DB
│   └── test_core/             # Test core utilities (security, ACL)
├── integration/               # Tests con dependencias reales
│   ├── test_api/              # Test API endpoints con test DB
│   ├── test_mcp/              # Test MCP tools con test DB
│   └── test_db/               # Test database operations
└── e2e/                       # Tests de flujos completos
    └── test_flows/            # Test user journeys
```

### Fixtures

```python
# tests/conftest.py
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from alejandria.db.base import Base

@pytest.fixture
def db_session():
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.close()
    Base.metadata.drop_all(engine)

@pytest.fixture
def user_service(db_session):
    return UserRepository(db_session)
```

## Best Practices

### 1. Separation of Concerns

- **API layer**: Solo HTTP concerns (validation, serialization)
- **Service layer**: Solo business logic
- **Repository layer**: Solo data access
- **No cross-layer dependencies**: API no llama directamente a repositories

### 2. Dependency Injection

- Inyectar dependencies via constructor
- Usar FastAPI Depends para API layer
- Facilita testing con mocks

### 3. Error Handling

- Lanzar excepciones específicas (UserNotFoundError, PermissionDeniedError)
- Capturar en middleware para convertir a HTTP responses
- Nunca exponer detalles internos en respuestas de error

### 4. Configuration

- Todas las configuraciones en `core/config.py`
- Usar environment variables para secrets
- Valores por defecto para development

### 5. Database

- Usar SQLAlchemy ORM para operaciones CRUD
- Queries complejas en repositories
- Siempre usar sessions con context managers

### 6. Testing

- Tests unitarios con mocks
- Tests de integración con test database
- E2E tests para flujos críticos
- Coverage >80% para core logic

## Common Tasks

### Agregar nuevo endpoint API

1. Crear Pydantic schema en `api/schemas/`
2. Crear route handler en `api/routers/`
3. Crear service method en `services/`
4. Crear repository method si es necesario
5. Agregar tests en `tests/integration/test_api/`
6. Actualizar `ENG-REF-0002` si es endpoint público

### Agregar nueva herramienta MCP

1. Crear schema en `mcp/schemas/`
2. Implementar tool en `mcp/tools/`
3. Registrar tool en `mcp/server.py`
4. Crear service method si es necesario
5. Agregar tests en `tests/integration/test_mcp/`
6. Actualizar `ENG-REF-0001`

### Agregar nuevo modelo de base de datos

1. Crear SQLAlchemy model en `db/models/`
2. Crear Alembic migration
3. Crear repository en `repositories/`
4. Agregar tests en `tests/integration/test_db/`
5. Actualizar `ENG-REF-0007` si es tabla core

## Referencias

- **ENG-DEF-0001**: System Architecture
- **ADR-0001**: Docker Compose para Desarrollo Local
- **ADR-0002**: Python + FastMCP como Backend Framework
- **ADR-0003**: PostgreSQL como Base de Datos Relacional
- **ADR-0004**: Qdrant como Vector Database
- **ADR-0005**: sentence-transformers para Embeddings
- **ADR-0006**: FastAPI como Framework de API Web
- **ENG-GUIDE-0002**: Testing Strategy
- **ENG-GUIDE-0003**: Testing Implementation
