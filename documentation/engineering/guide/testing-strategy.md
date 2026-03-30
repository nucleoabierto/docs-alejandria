---
id: ENG-GUIDE-0002
area: "engineering"
type: "guide"
title: "Testing Strategy - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0001", "ADR-0002", "ADR-0003", "ADR-0004", "ENG-GUIDE-0003"]
---

Estrategia de testing para Alejandria MVP. Contiene pirámide de testing, frameworks, estructura de directorios, configuración de pytest y mejores prácticas.

---

# Testing Strategy - Alejandria

Sabemos que testing efectivo es la diferencia entre código que funciona y código que rompe en producción. Esta guía define nuestra estrategia de testing para Alejandria, para que tengamos confianza en cada cambio que hacemos.

## Table of Contents

1. [Overview](#overview)
2. [Estrategia de Testing](#estrategia-de-testing)
3. [Frameworks y Herramientas](#frameworks-y-herramientas)
4. [Estructura de Directorios](#estructura-de-directorios)
5. [Fixtures Globales (conftest.py)](#fixtures-globales-conftestpy)
6. [Ejecutar Tests](#ejecutar-tests)
7. [Coverage](#coverage)
8. [Best Practices](#best-practices)
9. [CI/CD Integration](#cicd-integration)
10. [Troubleshooting](#troubleshooting)
11. [Referencias](#referencias)

## Overview

**Enfoque**: Desarrollo local con Docker Compose (ver ADR-0001)  
**Framework**: pytest con pytest-asyncio  
**Coverage target**: >80% para core logic  
**Foco**: Funcionalidad MCP (document management) y API web (users/organizations)

No te preocupes por lograr 80% de coverage de inmediato. Es un objetivo a largo plazo, no un requisito para empezar.

## Estrategia de Testing

### Pirámide de Testing

```
        /\
       /E2E\        10% - Tests end-to-end completos
      /------\
     /Integration\  30% - Tests de integración con servicios reales
    /------------\
   /  Unit Tests  \  60% - Tests unitarios de lógica de negocio
  /----------------\
```

**Distribución**:
- **Unit Tests (60%)**: Lógica de negocio, validaciones, transformaciones de datos
- **Integration Tests (30%)**: Integración con PostgreSQL, Qdrant, FastMCP
- **E2E Tests (10%)**: Flujos completos desde MCP hasta base de datos

Esta pirámide te ayuda a enfocarte en lo más importante: tests unitarios rápidos que te dan feedback inmediato, con algunos tests de integración y E2E para asegurar que todo funciona junto.

## Frameworks y Herramientas

### Stack de Testing

```txt
# Core Testing
pytest>=7.4.0
pytest-asyncio>=0.21.0
pytest-cov>=4.1.0

# Testing de APIs
httpx>=0.25.0           # Para tests de FastAPI
testcontainers>=3.7.0   # Para tests con contenedores Docker

# Testing de Base de Datos
pytest-postgresql>=5.0.0
factory-boy>=3.3.0      # Factories para test data

# Mocking
pytest-mock>=3.12.0
freezegun>=1.2.2        # Para mock de tiempo

# Testing de MCP
mcp-testing-lib>=0.1.0  # (cuando esté disponible, o usar custom fixtures)
```

No necesitas memorizar todas estas herramientas. pytest es el núcleo, y las demás son helpers que hacen tu vida más fácil cuando las necesitas.

### Configuración de pytest

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    --cov=alejandria
    --cov-report=html
    --cov-report=term-missing
    --cov-fail-under=80
    -v
    --tb=short
asyncio_mode = auto
```

Esta configuración ya está lista para usar. Solo necesitas crear tu archivo `pytest.ini` y pytest hará el resto por ti.

## Estructura de Directorios

```
alejandria/
├── backend/
│   ├── alejandria/
│   │   ├── __init__.py
│   │   ├── main.py              # Entry point FastMCP
```

Esta estructura mantiene tus tests organizados y separados del código de producción, lo cual es una buena práctica que te ahorrará dolores de cabeza a largo plazo.
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── app.py           # FastAPI app
│   │   │   ├── v1/
│   │   │   │   ├── users.py
│   │   │   │   ├── orgs.py
│   │   │   │   └── health.py
│   │   │   └── schemas.py
│   │   ├── db/
│   │   │   ├── models.py
│   │   │   └── session.py
│   │   ├── mcp/
│   │   │   └── tools.py         # MCP tools
│   │   └── services/
│   │       ├── document_service.py
│   │       └── search_service.py
│   └── tests/
│       ├── conftest.py          # Configuración global de pytest
│       ├── unit/
│       │   ├── test_document_service.py
│       │   ├── test_search_service.py
│       │   └── test_acl.py
│       ├── integration/
│       │   ├── test_postgres.py
│       │   ├── test_qdrant.py
│       │   └── test_mcp_tools.py
│       ├── e2e/
│       │   ├── test_document_lifecycle.py
│       │   └── test_user_org_lifecycle.py
│       └── fixtures/
│           ├── db_fixtures.py
│           └── api_fixtures.py
```

## Fixtures Globales (conftest.py)

```python
# tests/conftest.py
import pytest
import asyncio
from typing import AsyncGenerator, Generator
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import sessionmaker
from testcontainers.postgres import PostgresContainer
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams

from alejandria.api.app import app
from alejandria.db.models import Base
from alejandria.db.session import get_db

# Testcontainers para PostgreSQL
@pytest.fixture(scope="session")
def postgres_container() -> PostgresContainer:
    """PostgreSQL container para tests de integración."""
    with PostgresContainer("postgres:15-alpine") as postgres:
        yield postgres

@pytest.fixture(scope="session")
def qdrant_container():
    """Qdrant container para tests de integración."""
    # Usar Qdrant local o Docker container
    client = QdrantClient(host="localhost", port=6333)
    # Crear colección de test
    client.create_collection(
        collection_name="test_collection",
        vectors_config=VectorParams(size=384, distance=Distance.COSINE)
    )
    yield client
    # Cleanup
    client.delete_collection("test_collection")

@pytest.fixture(scope="session")
def test_engine(postgres_container: PostgresContainer):
    """Engine de SQLAlchemy para tests."""
    engine = create_async_engine(
        postgres_container.get_connection_url(),
        echo=False
    )
    yield engine
    engine.dispose()

@pytest.fixture(scope="function")
async def db_session(test_engine) -> AsyncGenerator[AsyncSession, None]:
    """Session de base de datos para cada test."""
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async_session = async_sessionmaker(
        test_engine,
        class_=AsyncSession,
        expire_on_commit=False
    )
    
    async with async_session() as session:
        yield session
    
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.fixture
async def client(db_session: AsyncSession) -> AsyncGenerator[AsyncClient, None]:
    """Cliente HTTP de prueba para FastAPI."""
    def override_get_db():
        yield db_session
    
    app.dependency_overrides[get_db] = override_get_db
    
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    
    app.dependency_overrides.clear()
```

Las fixtures pueden parecer complejas al principio, pero son herramientas poderosas que te ahorran tiempo configurando y limpiando tus tests automáticamente. No necesitas entender todo de inmediato, puedes copiar este archivo y adaptarlo según tus necesidades.

## Ejecutar Tests

### Comandos de Testing

```bash
# Ejecutar todos los tests
pytest

# Ejecutar solo unit tests
pytest tests/unit/

# Ejecutar solo integration tests
pytest tests/integration/

# Ejecutar solo E2E tests
pytest tests/e2e/

# Ejecutar con coverage
pytest --cov=alejandria --cov-report=html

# Ejecutar tests específicos
pytest tests/unit/test_document_service.py::TestDocumentService::test_create_document_success

# Ejecutar tests en paralelo (más rápido)
pytest -n auto

# Ejecutar tests con verbose output
pytest -v

# Ejecutar tests que fallen en la última ejecución
pytest --lf

# Ejecutar tests con markers específicos
pytest -m unit
pytest -m integration
pytest -m e2e
```

Estos comandos te dan flexibilidad para ejecutar tests según lo que necesites en cada momento. No necesitas memorizarlos todos, solo los que uses más frecuentemente.

### Markers

```python
# tests/conftest.py
def pytest_configure(config):
    config.addinivalue_line(
        "markers", "unit: Unit tests for business logic"
    )
    config.addinivalue_line(
        "markers", "integration: Integration tests with external services"
    )
    config.addinivalue_line(
        "markers", "e2e: End-to-end tests"
    )
    config.addinivalue_line(
        "markers", "slow: Tests that take >1s"
    )
```

Los markers te permiten categorizar tus tests y ejecutar solo los que necesitas en cada momento. Esto es especialmente útil cuando quieres ejecutar solo tests rápidos durante desarrollo.

## Coverage

### Métricas de Coverage

```bash
# Ver coverage en terminal
pytest --cov=alejandria --cov-report=term-missing

# Ver coverage en HTML
pytest --cov=alejandria --cov-report=html
# Abre htmlcov/index.html en navegador

# Ver coverage por módulo
pytest --cov=alejandria.api --cov=alejandria.services
```

El coverage HTML es especialmente útil porque te muestra exactamente qué líneas de código no están cubiertas por tests, lo cual te ayuda a identificar qué áreas necesitan más atención.

### Coverage Targets

- **Core logic**: >90% coverage
- **API endpoints**: >80% coverage
- **Services**: >85% coverage
- **Models**: >95% coverage
- **Overall**: >80% coverage

Recuerda que estos son objetivos a largo plazo. No te preocupes si no logras estos números de inmediato. Lo importante es que tus tests cubran los casos críticos de tu aplicación.

## Best Practices

Aquí tienes algunas prácticas que te ayudarán a escribir tests efectivos y mantenibles.

### 1. Tests Independientes

Cada test debe ser independiente y no depender del orden de ejecución. Esto hace que tus tests sean más confiables y más fáciles de debuggear:

```python
# ❌ MAL - Tests dependientes
def test_create_document():
    # crea documento con ID específico
    pass

def test_update_document():
    # asume que test_create_document creó el documento
    pass

# ✅ BIEN - Tests independientes
def test_create_document():
    # crea documento en setup local
    pass

def test_update_document():
    # crea su propio documento en setup local
    pass
```

### 2. Tests Determinísticos

Los tests deben ser determinísticos (no randomness). Esto significa que deben producir el mismo resultado cada vez que los ejecutas, lo cual es esencial para debugging confiable:

```python
# ❌ MAL - Randomness
def test_random_id():
    doc_id = str(uuid.uuid4())
    assert doc_id is not None

# ✅ BIEN - Deterministic
def test_specific_id():
    doc_id = "fixed-test-id"
    assert doc_id == "fixed-test-id"
```

### 3. Tests Rápidos

Los unit tests deben ser rápidos (<100ms cada uno). Tests rápidos te dan feedback inmediato y te motivan a escribir más tests porque no esperas mucho tiempo:

```python
# ❌ MAL - Test lento (espera real)
def test_slow():
    time.sleep(5)
    assert True

# ✅ BIEN - Test rápido (mock)
def test_fast(mocker):
    mocker.patch('time.sleep')
    assert True
```

### 4. Tests Descriptivos

Los nombres de tests deben describir qué están probando. Un buen nombre de test te dice exactamente qué falló sin necesidad de leer el código:

```python
# ❌ MAL - Nombre no descriptivo
def test_1():
    pass

# ✅ BIEN - Nombre descriptivo
def test_create_document_with_empty_title_raises_validation_error():
    pass
```

## CI/CD Integration

### GitHub Actions Example

Este ejemplo muestra cómo integrar tus tests en GitHub Actions para que se ejecuten automáticamente en cada push y pull request. Esto te da confianza de que tus cambios no rompen nada existente.

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      qdrant:
        image: qdrant/qdrant:v1.7.0
        options: >-
          --health-cmd curl -f http://localhost:6333/health
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-asyncio pytest-cov
      
      - name: Run tests
        run: pytest --cov=alejandria --cov-report=xml
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## Troubleshooting

Sabemos que los tests a veces fallan de maneras confusas. Aquí tienes soluciones para los problemas más comunes.

### Problema: Tests fallan con "database is locked"

**Causa**: Tests no están limpiando correctamente la base de datos después de cada test.

**Solución**: Usar fixtures que crean y dropean tablas en cada test. Esto asegura que cada test tenga una base de datos limpia:

```python
@pytest.fixture(scope="function")
async def db_session(test_engine):
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    
    async_session = async_sessionmaker(test_engine, class_=AsyncSession)
    async with async_session() as session:
        yield session
    
    async with test_engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
```

### Problema: Tests de Qdrant fallan con "collection not found"

**Causa**: Colección de test no existe o fue eliminada en test anterior.

**Solución**: Crear colección en fixture de sesión y limpiar al final. Esto asegura que la colección esté siempre disponible:

```python
@pytest.fixture(scope="session")
def qdrant_container():
    client = QdrantClient(host="localhost", port=6333)
    
    # Crear colección si no existe
    try:
        client.create_collection(
            collection_name="test_collection",
            vectors_config=VectorParams(size=384, distance=Distance.COSINE)
        )
    except:
        pass  # Ya existe
    
    yield client
    
    # Limpiar al final
    try:
        client.delete_collection("test_collection")
    except:
        pass
```

### Problema: Tests asíncronos fallan con "coroutine never awaited"

**Causa**: No usando pytest-asyncio correctamente.

**Solución**: Asegurar que:
1. `pytest.ini` tiene `asyncio_mode = auto`
2. Tests asíncronos tienen decorador `@pytest.mark.asyncio`
3. Fixture asíncronos usan `AsyncGenerator`

Los tests asíncronos pueden ser confusos al principio. Si tienes problemas con esto, verifica estos tres puntos primero.

## Referencias

- **ADR-0001**: Docker Compose para Desarrollo Local
- **ENG-DEF-0001**: System Architecture
- **ENG-REF-0001**: MVP MCP Tools Definition
- **ENG-REF-0002**: MVP Web API Specification
- **ENG-REF-0007**: Schema Core Tables
- **ENG-GUIDE-0003**: Testing Implementation (ejemplos de código)

Si encuentras problemas no documentados aquí, no dudes en pedir ayuda. Testing puede ser complicado, pero estás en el camino correcto.
