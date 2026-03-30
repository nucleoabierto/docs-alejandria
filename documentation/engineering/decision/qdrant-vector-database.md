---
id: ADR-0004
area: "engineering"
type: "decision"
title: "Qdrant como Vector Database"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0002"]
---

Registro de decisión arquitectónica que justifica la selección de Qdrant como motor de búsqueda vectorial para Alejandria. Contiene contexto del problema de búsqueda semántica con filtros de seguridad, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0004: Qdrant como Vector Database

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que necesitamos un motor de búsqueda vectorial que soporte búsqueda semántica con embeddings para que los agentes de IA puedan encontrar contexto relevante en los documentos de Alejandria. El requisito crítico es que este motor debe soportar filtros de metadata (ACL, organización, usuario) para asegurar que cada búsqueda solo retorne documentos que el usuario tiene permiso para ver.

El desafío técnico es que no todos los vector databases tienen soporte nativo y eficiente para filtros de metadata. Algunos requieren post-processing de resultados (filtrar después de la búsqueda), lo cual es ineficiente y no escala. Otros tienen soporte limitado para filtros complejos (combinaciones de AND/OR en múltiples campos).

Entendemos que necesitamos:
- Búsqueda por similitud coseno con top-k eficiente
- Filtros de metadata nativos y pre-indexados (ACL, organización, usuario)
- Performance escalable para manejar ~10M embeddings en el MVP
- Opción de self-hosted para control total y costos predecibles
- Migración sencilla a managed cloud cuando sea necesario

## Decision

Usar Qdrant como vector database para almacenamiento de embeddings y búsqueda semántica.

**Especificación técnica:**
- **Vector database**: Qdrant v1.7+
- **Deployment**: Self-hosted con Docker (MVP), migración a Qdrant Cloud cuando sea necesario
- **Embedding dimension**: 384 (all-MiniLM-L6-v2) o 768 (all-mpnet-base-v2)
- **Distance metric**: Cosine similarity
- **Collections**: Una colección por organización para aislamiento
- **Filtros**: Payload filtering nativo para ACL, organización, usuario

Qdrant proporciona:
- Búsqueda por similitud coseno con HNSW index para performance (piensa en esto como un índice inteligente que entiende significado)
- Payload filtering nativo y eficiente (pre-indexado)
- API REST y gRPC para integración
- Snapshot/restore para migración zero downtime a cloud
- Self-hosted con Docker o Kubernetes
- Managed cloud option con escalabilidad automática

## Consequences

### Positivas

✅ **Filtros de metadata nativos**: Qdrant soporta filtrado por payload antes de la búsqueda vectorial, lo cual es esencial para ACL. Esto significa que solo se buscan embeddings que cumplen los filtros, mejorando performance y seguridad.

✅ **Performance escalable**: Con HNSW index, Qdrant maneja ~10M embeddings con latencia <100ms en hardware modesto. Esto es suficiente para el MVP y permite crecimiento predecible.

✅ **Self-hosted inicial**: Podemos empezar con Qdrant self-hosted en Docker, lo cual reduce costos y da control total. Cuando necesitemos escalabilidad, migramos a Qdrant Cloud con snapshot/restore zero downtime.

✅ **API limpia y documentada**: El cliente Python de Qdrant es simple de usar con buena documentación, reduciendo el tiempo de desarrollo.

✅ **Una colección por organización**: Podemos crear una colección Qdrant por organización, lo cual simplifica ACL filtering (búsqueda solo en la colección de la organización del usuario).

✅ **Open source y activo**: Qdrant es open source con comunidad activa, lo cual reduce riesgo de vendor lock-in. El proyecto tiene backing comercial y roadmap claro.

### Negativas

❌ **Menos maduro que Pinecone**: Qdrant es más joven que Pinecone (2020 vs 2019), pero tiene suficiente madurez para producción y adopción creciente.

❌ **Requiere mantenimiento de cluster**: Self-hosted requiere mantenimiento (updates, backups, monitoring). Esto es aceptable para el MVP y se elimina al migrar a Qdrant Cloud.

❌ **Limitaciones en filtros complejos**: Qdrant soporta filtros potentes pero no tan expresivos como PostgreSQL (no soporta joins complejos). Esto es aceptable porque los filtros de ACL son relativamente simples (organization_id, visibility, permissions).

❌ **Memory requirements**: Qdrant requiere RAM para mantener el HNSW index en memoria. Para ~10M embeddings de 384 dimensiones, necesitamos ~15GB RAM. Esto es manejable con instancias cloud moderadas.

## Alternatives Considered

### Pinecone

**Por qué no la elegimos:**
- Cloud-only, no permite self-hosted
- Vendor lock-in completo (no podemos migrar fácilmente a otra solución)
- Costos impredecibles a escala (pago por uso puede ser caro)
- Menos control sobre datos y compliance

**Cuándo podría ser mejor:**
- Si el equipo no tuviera capacidad para infraestructura
- Si necesitáramos escalabilidad masiva desde día 1
- Si el budget no fuera una preocupación

### Weaviate

**Por qué no la elegimos:**
- Más complejo de configurar y mantener
- Curva de aprendizaje más alta
- Documentación menos clara que Qdrant
- Payload filtering menos eficiente en algunas versiones

**Cuándo podría ser mejor:**
- Si necesitáramos features avanzadas de knowledge graph
- Si requiriéramos multi-modal search (texto + imagen + video)
- Si el equipo tuviera experiencia previa con Weaviate

### PostgreSQL pgvector

**Por qué no la elegimos:**
- Performance limitado para datasets grandes (>1M embeddings)
- Indexación menos eficiente que HNSW
- Filtros de metadata son potentes pero la búsqueda vectorial es más lenta
- No especializado en búsqueda vectorial

**Cuándo podría ser mejor:**
- Si el dataset fuera pequeño (<100k embeddings)
- Si quisiéramos simplificar el stack (solo PostgreSQL)
- Si los filtros de metadata fueran extremadamente complejos

### Milvus

**Por qué no la elegimos:**
- Más complejo de configurar y mantener
- Requiere más componentes (etcd, MinIO, Pulsar)
- Curva de aprendizaje más alta
- Overkill para el MVP

**Cuándo podría ser mejor:**
- Si necesitáramos escalabilidad masiva (>100M embeddings)
- Si tuviéramos equipo dedicado a infraestructura
- Si el proyecto fuera enterprise desde día 1

## Implementation

### Paso 1: Instalación y configuración local
```bash
# Docker Compose para desarrollo
version: '3.8'
services:
  qdrant:
    image: qdrant/qdrant:v1.7.0
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_data:/qdrant/storage
```

### Paso 2: Cliente Python
```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, Filter, FieldCondition, MatchValue

client = QdrantClient(host="localhost", port=6333)

# Crear colección por organización
client.create_collection(
    collection_name=f"org_{organization_id}",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE),
)
```

### Paso 3: Insertar embeddings con metadata
```python
client.upsert(
    collection_name=f"org_{organization_id}",
    points=[
        {
            "id": chunk_id,
            "vector": embedding,
            "payload": {
                "document_id": doc_id,
                "organization_id": organization_id,
                "acl": acl,
                "created_at": timestamp
            }
        }
    ]
)
```

### Paso 4: Búsqueda con filtros ACL
```python
search_result = client.search(
    collection_name=f"org_{organization_id}",
    query_vector=query_embedding,
    query_filter=Filter(
        must=[
            FieldCondition(key="acl.visibility", match=MatchValue(value="public")),
            # O: FieldCondition(key="acl.permissions.read", match=MatchValue(value=user_id))
        ]
    ),
    limit=10
)
```

### Paso 5: Testing
- Unit tests para operaciones CRUD
- Performance tests para latencia de búsqueda
- ACL filtering tests para verificar seguridad

### Paso 6: Deployment
- Docker container con persistencia de datos
- Environment variables para configuración
- Backup schedule (snapshots diarios)
- Health checks para monitoreo

## Related Decisions

- **ADR-0002**: Python + FastMCP como backend framework (esta decisión complementa la arquitectura)
- **ADR-0003**: PostgreSQL como base de datos relacional (Qdrant complementa PostgreSQL para embeddings)
- **ADR-0005**: sentence-transformers para embeddings (define el modelo de embeddings)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Madurez de Qdrant:**
- Monitorear el roadmap y releases del proyecto
- Contribuir a la comunidad open source
- Tener plan de contingencia para migrar a Pinecone si es necesario

**Mantenimiento de cluster:**
- Automatizar backups y restores con snapshots
- Usar Docker Compose o Kubernetes para deployment simplificado
- Implementar monitoreo y alertas tempranas
- Plan de migración a Qdrant Cloud documentado

**Limitaciones en filtros:**
- Diseñar ACL simple y eficiente desde el inicio
- Usar una colección por organización para simplificar filtros
- Validar que los filtros requeridos son soportados por Qdrant

**Memory requirements:**
- Monitorear uso de RAM y planificar upgrade de hardware
- Usar compression de embeddings si es necesario
- Considerar sharding por organización si crece significativamente
