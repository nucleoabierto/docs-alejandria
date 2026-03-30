---
id: ENG-GUIDE-0003
area: "engineering"
type: "guide"
title: "Testing Implementation - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-GUIDE-0002", "ENG-DEF-0001", "ENG-REF-0001", "ENG-REF-0002"]
---

Ejemplos de implementación de tests para Alejandria MVP. Contiene ejemplos de unit tests, integration tests, E2E tests y testing de MCP tools.

---

# Testing Implementation - Alejandria

Esta guía proporciona ejemplos prácticos de implementación de tests para Alejandria. Contiene ejemplos de unit tests, integration tests, E2E tests y testing de MCP tools.

## Table of Contents

1. [Unit Tests](#unit-tests)
2. [Integration Tests](#integration-tests)
3. [E2E Tests](#e2e-tests)
4. [Testing de MCP Tools](#testing-de-mcp-tools)
5. [Referencias](#referencias)

---

Sabemos que escribir tests puede ser intimidante, especialmente cuando estás empezando. Esta guía contiene ejemplos prácticos de implementación de tests para el MVP de Alejandria, para que puedas ver cómo se aplican los conceptos de la Testing Strategy en código real.

## Unit Tests

### Ejemplo: Document Service

Aquí tienes un ejemplo completo de cómo escribir unit tests para tu Document Service. Estos tests prueban la lógica de negocio de crear documentos y validar ACL:

```python
# tests/unit/test_document_service.py
import pytest
from alejandria.services.document_service import DocumentService
from alejandria.db.models import Document, User, Organization

@pytest.mark.unit
class TestDocumentService:
    
    @pytest.mark.asyncio
    async def test_create_document_success(self, db_session: AsyncSession):
        """Test creación exitosa de documento."""
        service = DocumentService(db_session)
        
        # Setup: crear usuario y organización
        user = User(email="test@example.com", name="Test User", api_key="sk-test")
        org = Organization(name="Test Org", type="personal", owner_id=user.id)
        db_session.add_all([user, org])
        await db_session.commit()
        
        # Act
        document = await service.create_document(
            title="Test Document",
            content="Test content",
            organization_id=org.id,
            owner_id=user.id,
            area_id="area-uuid",
            type_id="type-uuid",
            acl={"visibility": "private", "permissions": {"default": "read"}}
        )
        
        # Assert
        assert document.id is not None
        assert document.title == "Test Document"
        assert document.organization_id == org.id
        assert document.acl["visibility"] == "private"
    
    @pytest.mark.asyncio
    async def test_create_document_empty_title_raises_error(self, db_session: AsyncSession):
        """Test que título vacío raise error."""
        service = DocumentService(db_session)
        
        with pytest.raises(ValueError, match="title cannot be empty"):
            await service.create_document(
                title="",
                content="Test content",
                organization_id="org-uuid",
                owner_id="user-uuid",
                area_id="area-uuid",
                type_id="type-uuid",
                acl={"visibility": "private"}
            )
    
    @pytest.mark.asyncio
    async def test_acl_validation_private_requires_owner_permission(self, db_session: AsyncSession):
        """Test que ACL private valida permisos correctamente."""
        service = DocumentService(db_session)
        
        # Test caso: documento privado sin permisos default
        with pytest.raises(ValueError, match="private documents must have default permissions"):
            await service.create_document(
                title="Test",
                content="Content",
                organization_id="org-uuid",
                owner_id="user-uuid",
                area_id="area-uuid",
                type_id="type-uuid",
                acl={"visibility": "private", "permissions": {"default": "none"}}
            )
```

### Ejemplo: ACL Validation

```python
# tests/unit/test_acl.py
import pytest
from alejandria.services.acl_service import ACLService
```

Los tests de ACL son cruciales para la seguridad de Alejandria. Aquí tienes ejemplos de cómo probar diferentes escenarios de permisos:

```python
@pytest.mark.unit
class TestACLService:

@pytest.mark.unit
class TestACLService:
    
    def test_user_can_read_private_document_as_owner(self):
        """Test que owner puede leer documento privado."""
        acl = ACLService()
        document_acl = {
            "visibility": "private",
            "permissions": {"default": "read"}
        }
        
        assert acl.can_read(
            user_id="owner-id",
            document_owner_id="owner-id",
            document_acl=document_acl
        ) is True
    
    def test_user_cannot_read_private_document_as_non_owner(self):
        """Test que non-owner no puede leer documento privado sin permiso específico."""
        acl = ACLService()
        document_acl = {
            "visibility": "private",
            "permissions": {"default": "none"}
        }
        
        assert acl.can_read(
            user_id="other-id",
            document_owner_id="owner-id",
            document_acl=document_acl
        ) is False
    
    def test_user_can_read_public_document_any_user(self):
        """Test que cualquier usuario puede leer documento público."""
        acl = ACLService()
        document_acl = {
            "visibility": "public",
            "permissions": {"default": "read"}
        }
        
        assert acl.can_read(
            user_id="any-id",
            document_owner_id="owner-id",
            document_acl=document_acl
        ) is True
```

## Integration Tests

### Ejemplo: PostgreSQL Integration

Los tests de integración verifican que tu código funciona correctamente con servicios externos como PostgreSQL. Aquí tienes ejemplos:

```python
# tests/integration/test_postgres.py
import pytest
from sqlalchemy import text

@pytest.mark.integration
class TestPostgresIntegration:
    
    @pytest.mark.asyncio
    async def test_database_connection(self, test_engine):
        """Test que podemos conectar a PostgreSQL."""
        async with test_engine.begin() as conn:
            result = await conn.execute(text("SELECT 1"))
            assert result.scalar() == 1

Este test simple verifica que la conexión a PostgreSQL funciona correctamente. Es un buen punto de partida para tus tests de integración.
    
    @pytest.mark.asyncio
    async def test_foreign_key_constraint(self, db_session: AsyncSession):
        """Test que foreign keys funcionan correctamente."""
        from alejandria.db.models import Document, Organization
        
        # Intentar crear documento con organización inexistente
        doc = Document(
            organization_id="non-existent-org",
            owner_id="user-id",
            title="Test",
            acl={"visibility": "private"}
        )
        
        db_session.add(doc)
        
        # Debería fallar por foreign key constraint
        with pytest.raises(Exception):  # SQLAlchemy IntegrityError
            await db_session.commit()

Este test asegura que las restricciones de integridad referencial de PostgreSQL funcionan correctamente, protegiendo tus datos de inconsistencias.
    
    @pytest.mark.asyncio
    async def test_jsonb_indexing(self, db_session: AsyncSession):
        """Test que índices GIN en JSONB funcionan."""
        from alejandria.db.models import Document
        from sqlalchemy import text
        
        # Crear documento con ACL específico
        doc = Document(
            organization_id="org-id",
            owner_id="user-id",
            title="Test",
            acl={"visibility": "private", "permissions": {"default": "read"}}
        )
        db_session.add(doc)
        await db_session.commit()
        
        # Buscar usando índice GIN
        async with db_session.begin() as conn:
            result = await conn.execute(
                text("SELECT * FROM documents WHERE acl->>'visibility' = 'private'")
            )
            documents = result.fetchall()
            assert len(documents) == 1
```

Este test verifica que los índices GIN en JSONB funcionan correctamente, lo cual es esencial para búsquedas eficientes de ACL.

### Ejemplo: Qdrant Integration

Los tests de integración con Qdrant verifican que la búsqueda semántica funciona correctamente. Aquí tienes ejemplos:

```python
# tests/integration/test_qdrant.py
import pytest
import numpy as np

@pytest.mark.integration
class TestQdrantIntegration:
    
    @pytest.mark.asyncio
    async def test_qdrant_connection(self, qdrant_container):
        """Test que podemos conectar a Qdrant."""
        collections = qdrant_container.get_collections().collections
        assert "test_collection" in [c.name for c in collections]
    
    @pytest.mark.asyncio
    async def test_insert_and_search(self, qdrant_container):
        """Test que podemos insertar y buscar embeddings."""
        # Insertar embeddings
        qdrant_container.upsert(
            collection_name="test_collection",
            points=[
                {
                    "id": 1,
                    "vector": np.random.rand(384).tolist(),
                    "payload": {"document_id": "doc-1", "acl": {"visibility": "public"}}
                },
                {
                    "id": 2,
                    "vector": np.random.rand(384).tolist(),
                    "payload": {"document_id": "doc-2", "acl": {"visibility": "private"}}
                }
            ]
        )
        
        # Buscar con filtro
        query_vector = np.random.rand(384).tolist()
        results = qdrant_container.search(
            collection_name="test_collection",
            query_vector=query_vector,
            query_filter={
                "must": [
                    {"key": "acl.visibility", "match": {"value": "public"}}
                ]
            },
            limit=10
        )
        
        # Debería retornar solo documento público
        assert len(results) == 1
        assert results[0].payload["document_id"] == "doc-1"

Este test es crucial porque verifica que los filtros de ACL funcionan correctamente en Qdrant, asegurando que los usuarios solo vean documentos que tienen permiso para ver.
```

## E2E Tests

### Ejemplo: Document Lifecycle

Los tests E2E verifican que el flujo completo de la aplicación funciona correctamente. Son más lentos pero aseguran que todos los componentes trabajen juntos:

```python
# tests/e2e/test_document_lifecycle.py
import pytest
from httpx import AsyncClient

@pytest.mark.e2e
class TestDocumentLifecycleE2E:
    
    @pytest.mark.asyncio
    async def test_full_document_lifecycle(self, client: AsyncClient):
        """Test ciclo de vida completo de documento."""
        
        # 1. Crear usuario
        user_response = await client.post(
            "/v1/users",
            json={
                "email": "test@example.com",
                "name": "Test User",
                "password": "securepassword123"
            }
        )
        assert user_response.status_code == 201
        user_data = user_response.json()["data"]
        api_key = user_data["api_key"]
        org_id = user_data["organization_id"]
        
        # 2. Crear documento via MCP (simulado con API web para test)
        # Nota: En producción esto sería via MCP, usamos API web para testing
        doc_response = await client.post(
            "/v1/documents",
            headers={"Authorization": f"Bearer {api_key}"},
            json={
                "title": "Test Document",
                "content": "Test content",
                "organization_id": org_id,
                "area_id": "area-uuid",
                "type_id": "type-uuid",
                "acl": {"visibility": "private", "permissions": {"default": "read"}}
            }
        )
        assert doc_response.status_code == 201
        doc_id = doc_response.json()["data"]["document_id"]
        
        # 3. Buscar documento
        search_response = await client.get(
            f"/v1/documents/{doc_id}",
            headers={"Authorization": f"Bearer {api_key}"}
        )
        assert search_response.status_code == 200
        assert search_response.json()["data"]["title"] == "Test Document"
        
        # 4. Actualizar documento
        update_response = await client.patch(
            f"/v1/documents/{doc_id}",
            headers={"Authorization": f"Bearer {api_key}"},
            json={
                "changes": {
                    "title": "Updated Title"
                }
            }
        )
        assert update_response.status_code == 200
        assert update_response.json()["data"]["version"] == "2.0"
        
        # 5. Eliminar documento
        delete_response = await client.delete(
            f"/v1/documents/{doc_id}",
            headers={"Authorization": f"Bearer {api_key}"}
        )
        assert delete_response.status_code == 200
```

Este test E2E verifica todo el flujo de vida de un documento: creación, lectura, actualización y eliminación. Es un test valioso porque asegura que todos los componentes trabajen juntos correctamente.

## Testing de MCP Tools

### Simulación de MCP Client

Los tests de MCP tools verifican que los agentes de IA pueden interactuar correctamente con Alejandria. Aquí tienes ejemplos:

```python
# tests/integration/test_mcp_tools.py
import pytest
from alejandria.mcp.tools import create_document, search_documents

@pytest.mark.integration
class TestMCPTools:
    
    @pytest.mark.asyncio
    async def test_create_document_mcp_tool(self, db_session: AsyncSession):
        """Test tool MCP de crear documento."""
        # Setup
        user_id = "user-uuid"
        org_id = "org-uuid"
        
        # Act
        result = await create_document(
            title="Test Document",
            content="Test content",
            organization_id=org_id,
            area_id="area-uuid",
            type_id="type-uuid",
            acl={"visibility": "private", "permissions": {"default": "read"}},
            db_session=db_session
        )
        
        # Assert
        assert result["success"] is True
        assert "document_id" in result["data"]

Este test verifica que los agentes de IA pueden crear documentos correctamente via MCP, lo cual es esencial para el funcionamiento de Alejandria.
    
    @pytest.mark.asyncio
    async def test_search_documents_mcp_tool(self, db_session: AsyncSession, qdrant_container):
        """Test tool MCP de búsqueda de documentos."""
        # Setup: insertar documento en Qdrant
        qdrant_container.upsert(
            collection_name="test_collection",
            points=[{
                "id": 1,
                "vector": np.random.rand(384).tolist(),
                "payload": {"document_id": "doc-1", "acl": {"visibility": "public"}}
            }]
        )
        
        # Act
        result = await search_documents(
            query="test query",
            organization_id="org-uuid",
            limit=10,
            db_session=db_session,
            qdrant_client=qdrant_container
        )
        
        # Assert
        assert result["success"] is True
        assert len(result["data"]["results"]) >= 1
```

## Referencias

- **ENG-GUIDE-0003**: Testing Strategy (estrategia general)
- **ENG-DEF-0001**: System Architecture
- **ENG-REF-0001**: MVP MCP Tools Definition
- **ENG-REF-0002**: MVP Web API Specification
