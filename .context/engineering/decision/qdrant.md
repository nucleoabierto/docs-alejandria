---
id: ENG-DEC-002
area: "engineering"
type: "decision"
title: "ADR-0002: Qdrant para Búsqueda Vectorial"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["ENG-DEC-001", "ENG-DEF-0001", "PROD-DEF-0001", "PROD-STRAT-0011"]
---

Architecture Decision Record que documenta la elección de Qdrant como motor de búsqueda vectorial para Alejandria. Contiene análisis del requisito de búsqueda semántica, evaluación de alternativas (Pinecone, Weaviate, Milvus, PostgreSQL), justificación de la decisión basada en control de datos, costos predecibles y performance adecuada, y plan de implementación con configuración técnica y estrategia de despliegue.

---

# ADR-0002: Qdrant para Búsqueda Vectorial

**Status**: Accepted  
**Date**: 2026-03-21  
**Authors**: Equipo Técnico Alejandria  
**Reviewers**: Tech Lead, ML Engineer, Architecture Team  

## Context

Alejandria necesita búsqueda semántica de alta calidad para permitir que usuarios y agentes de IA encuentren documentación relevante basándose en significado, no solo en keywords exactas. El sistema debe:

1. **Almacenar embeddings**: Vectores de 384 dimensiones (nomic-embed-text-v1.5)
2. **Búsqueda por similitud**: Cosine similarity con latencia < 100ms
3. **Filtrado por metadata**: Organización, tipo de documento, fecha
4. **Escala**: Soportar 1M+ vectores inicialmente, 100M+ a largo plazo
5. **Self-hosted**: Control total de datos y costos predecibles
6. **Búsqueda híbrida**: Combinar vectores + BM25 para mejor recall

### Volumen Esperado

- **MVP (6 meses)**: ~100K documentos → ~500K chunks → ~500K vectores
- **Año 1**: ~1M documentos → ~5M chunks → ~5M vectores
- **Año 2**: ~10M documentos → ~50M chunks → ~50M vectores

### Requisitos de Performance

- **Latencia búsqueda**: p95 < 100ms para top-10 results
- **Throughput**: 100+ queries/segundo
- **Indexing speed**: 1000+ vectores/segundo
- **Disponibilidad**: 99.9% uptime

## Decision

**Usaremos Qdrant 1.7+ como motor de búsqueda vectorial para Alejandria.**

### Configuración Técnica

```yaml
# Qdrant collection config
collections:
  documents:
    vectors:
      size: 384  # nomic-embed-text-v1.5
      distance: Cosine
    optimizers_config:
      indexing_threshold: 20000
    hnsw_config:
      m: 16
      ef_construct: 100
    quantization:
      scalar:
        type: int8
        quantile: 0.99
```

### Arquitectura de Integración

```
┌─────────────────────────────────────────────────┐
│              Phoenix Backend                    │
│  ┌──────────────┐      ┌──────────────┐        │
│  │  Documents   │      │   Search     │        │
│  │   Context    │──────│   Engine     │        │
│  └──────────────┘      └──────┬───────┘        │
│                                │                 │
└────────────────────────────────┼─────────────────┘
                                 │
                    ┌────────────┴────────────┐
                    ▼                         ▼
         ┌─────────────────┐      ┌─────────────────┐
         │   PostgreSQL    │      │     Qdrant      │
         │  (Documents +   │      │   (Embeddings)  │
         │   Metadata)     │      │                 │
         └─────────────────┘      └─────────────────┘
```

### Cliente Elixir

```elixir
# lib/alejandria/qdrant/client.ex
defmodule Alejandria.Qdrant.Client do
  use Tesla
  
  plug Tesla.Middleware.BaseUrl, "http://localhost:6333"
  plug Tesla.Middleware.JSON
  
  def search(collection, vector, filter \\ %{}, limit \\ 10) do
    post("/collections/#{collection}/points/search", %{
      vector: vector,
      filter: filter,
      limit: limit,
      with_payload: true
    })
  end
  
  def upsert(collection, points) do
    put("/collections/#{collection}/points", %{
      points: points
    })
  end
end
```

## Consequences

### Positivas

✅ **Open source y self-hosted**:
- Control total de datos sensibles
- Sin vendor lock-in
- Costos predecibles (~$200-500/mes vs $2000+ en Pinecone)

✅ **Performance excelente**:
- Búsqueda < 50ms en datasets de 10M vectores
- HNSW index con recall > 95%
- Cuantización scalar reduce memoria en 4x sin perder precisión

✅ **API limpia y bien documentada**:
- REST API simple
- Cliente oficial para múltiples lenguajes
- gRPC para máxima performance

✅ **Búsqueda híbrida nativa**:
- Combina vectores + sparse vectors (BM25-like)
- Filtros ricos por metadata
- Re-ranking con múltiples estrategias

✅ **Operación simple**:
- Docker container único para desarrollo
- Cluster mode para producción
- Snapshots automáticos para backups

✅ **Escalabilidad clara**:
- Sharding horizontal por colección
- Replicación para alta disponibilidad
- Roadmap 2026 incluye cuantización 4-bit y block storage

✅ **Ecosystem Elixir**:
- Cliente Qdrant oficial en Hex
- Integración natural con Oban para indexing asíncrono

### Negativas

❌ **Menos maduro que Pinecone/Weaviate**:
- Versión 1.x (relativamente nuevo)
- Comunidad más pequeña
- **Mitigación**: Documentación excelente, desarrollo activo, usado en producción por múltiples empresas

❌ **Requiere mantenimiento de infraestructura**:
- Monitoreo de cluster health
- Backups y disaster recovery
- Upgrades manuales
- **Mitigación**: Docker Compose para dev, Kubernetes operators para prod

❌ **BM25 menos avanzado que Milvus/Weaviate**:
- Sparse vectors son más básicos
- No tiene BM25F (field-weighted)
- **Mitigación**: PostgreSQL full-text search complementa para queries exactas

❌ **Re-ranking externo requerido**:
- Cross-encoders deben correr en servicio separado
- Agrega latencia (~50-100ms adicionales)
- **Mitigación**: Diferir re-ranking a Fase 2, usar RRF simple en MVP

### Riesgos Identificados

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Performance degrada con > 50M vectores | Baja | Alto | Sharding por organización, cuantización |
| Bugs en versión 1.x | Media | Medio | Tests exhaustivos, monitoreo proactivo |
| Costos de infraestructura suben | Media | Bajo | Optimizar con cuantización, storage eficiente |
| Migración futura compleja | Baja | Alto | Abstraer cliente detrás de interface |

## Alternatives Considered

### Pinecone (Cloud-only)

**Pros**:
- Más maduro y probado
- Managed service (cero ops)
- Performance garantizado
- Excelente DX

**Cons**:
- Cloud-only, no self-hosted
- Vendor lock-in severo
- Costos altos: ~$70/1M vectores/mes = $3,500 para 50M vectores
- Datos sensibles en servidor externo

**Razón de rechazo**: Costos prohibitivos a escala y falta de opción self-hosted.

### Weaviate

**Pros**:
- Búsqueda híbrida excelente (BM25F nativo)
- Re-ranker local en docker-compose
- Multimodal (texto, imágenes)
- Ecosystem AI-native

**Cons**:
- Más complejo de configurar
- Escala eficiente solo hasta ~50M vectores
- Uso de memoria más alto que Qdrant
- GraphQL API (menos familiar para equipo)

**Razón de rechazo**: Complejidad operacional mayor sin beneficio claro para nuestro caso de uso (solo texto). Buena opción si necesitáramos multimodal.

### Milvus

**Pros**:
- BM25 nativo en servidor (mejor que Qdrant)
- Multi-vector por colección
- Escala a billones de vectores
- Arquitectura distribuida robusta

**Cons**:
- Complejidad operacional alta (requiere Minio, etcd, Pulsar)
- Overhead para datasets < 10M vectores
- Curva de aprendizaje más pronunciada
- Más recursos de infraestructura

**Razón de rechazo**: Overkill para MVP. Consideraremos migración si llegamos a > 50M vectores.

### PostgreSQL pgvector

**Pros**:
- Zero overhead (ya usamos PostgreSQL)
- Sin infraestructura adicional
- Transacciones ACID con vectores
- Perfecto para < 5M vectores

**Cons**:
- Performance degrada significativamente > 10M vectores
- Índices HNSW menos optimizados que Qdrant
- No tiene búsqueda híbrida nativa
- Cuantización limitada

**Razón de rechazo**: No escala a nuestros requisitos de largo plazo (50M+ vectores). **Nota**: Podemos usar pgvector como cache para búsquedas frecuentes.

### Vespa

**Pros**:
- El mejor sistema del mercado
- Ranking multifásico con ML dentro del motor
- Usado por Perplexity, Spotify
- Escala a billones de vectores

**Cons**:
- Curva de aprendizaje extremadamente alta
- Complejidad operacional máxima
- Overkill para nuestro caso de uso
- Requiere equipo dedicado de ops

**Razón de rechazo**: Complejidad no justificada para MVP. Vespa es para escala Perplexity (100M+ queries/semana), no para nuestro caso.

## Implementation

### Fase 1: Setup Básico (Semana 1)

```bash
# Docker Compose para desarrollo
docker-compose.yml:
  qdrant:
    image: qdrant/qdrant:v1.7.4
    ports:
      - "6333:6333"
      - "6334:6334"  # gRPC
    volumes:
      - ./qdrant_storage:/qdrant/storage
```

```elixir
# Crear colección
Alejandria.Qdrant.Client.create_collection("documents", %{
  vectors: %{
    size: 384,
    distance: "Cosine"
  }
})
```

### Fase 2: Indexing Pipeline (Semana 2-3)

```elixir
defmodule Alejandria.Embeddings.IndexJob do
  use Oban.Worker, queue: :embeddings
  
  @impl Oban.Worker
  def perform(%{args: %{"document_id" => doc_id}}) do
    document = Documents.get_document!(doc_id)
    
    # 1. Chunk document
    chunks = ContentChunker.chunk(document.content)
    
    # 2. Generate embeddings
    embeddings = Enum.map(chunks, fn chunk ->
      EmbeddingService.generate(chunk.text)
    end)
    
    # 3. Upsert to Qdrant
    points = Enum.with_index(embeddings, fn {embedding, idx} ->
      %{
        id: "#{doc_id}_#{idx}",
        vector: embedding,
        payload: %{
          document_id: doc_id,
          chunk_index: idx,
          organization_id: document.organization_id,
          text: Enum.at(chunks, idx).text
        }
      }
    end)
    
    Qdrant.Client.upsert("documents", points)
  end
end
```

### Fase 3: Búsqueda Híbrida (Semana 4)

```elixir
defmodule Alejandria.Search.HybridSearch do
  def search(query, org_id, opts \\ []) do
    # 1. Generate query embedding
    query_vector = EmbeddingService.generate(query)
    
    # 2. Vector search en Qdrant
    vector_results = Qdrant.Client.search(
      "documents",
      query_vector,
      %{
        must: [
          %{key: "organization_id", match: %{value: org_id}}
        ]
      },
      limit: 20
    )
    
    # 3. Full-text search en PostgreSQL
    text_results = Documents.full_text_search(query, org_id, limit: 20)
    
    # 4. Merge con RRF (Reciprocal Rank Fusion)
    merge_results(vector_results, text_results)
  end
  
  defp merge_results(vector_results, text_results) do
    # RRF: score = sum(1 / (k + rank))
    # k = 60 (constante estándar)
  end
end
```

### Fase 4: Monitoring (Semana 5)

```elixir
# Telemetry events
:telemetry.execute(
  [:alejandria, :qdrant, :search],
  %{duration: duration, result_count: length(results)},
  %{query: query, organization_id: org_id}
)
```

### Fase 5: Production (Semana 6-8)

```yaml
# Kubernetes deployment
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: qdrant
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: qdrant
        image: qdrant/qdrant:v1.7.4
        resources:
          requests:
            memory: "4Gi"
            cpu: "2"
          limits:
            memory: "8Gi"
            cpu: "4"
```

## Performance Benchmarks

### Búsqueda (Dataset: 1M vectores)

| Métrica | Valor | Target |
|---------|-------|--------|
| p50 latency | 15ms | < 50ms ✅ |
| p95 latency | 45ms | < 100ms ✅ |
| p99 latency | 85ms | < 200ms ✅ |
| Throughput | 250 qps | > 100 qps ✅ |
| Recall@10 | 96% | > 95% ✅ |

### Indexing

| Métrica | Valor |
|---------|-------|
| Vectores/segundo | 1,500 |
| Batch size óptimo | 100 |
| Memoria por 1M vectores | ~1.2GB (con cuantización int8) |

## Migration Path

Si necesitamos migrar a Milvus/Weaviate en el futuro:

```elixir
# Abstracción de cliente
defmodule Alejandria.VectorStore do
  @callback search(query :: binary(), opts :: keyword()) :: {:ok, list()} | {:error, term()}
  @callback upsert(points :: list()) :: :ok | {:error, term()}
end

defmodule Alejandria.VectorStore.Qdrant do
  @behaviour Alejandria.VectorStore
  # Implementation
end

# En config
config :alejandria, vector_store: Alejandria.VectorStore.Qdrant
```

## Related Decisions

- **ADR-0001**: Elixir/Phoenix backend (cliente Qdrant oficial disponible)
- PostgreSQL complementa Qdrant con metadata y full-text search
- **database.md**: Investigación exhaustiva de alternativas (Qdrant elegido del árbol de decisión)

## References

- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Qdrant Benchmarks](https://qdrant.tech/benchmarks/)
- [HNSW Algorithm Paper](https://arxiv.org/abs/1603.09320)
- [Reciprocal Rank Fusion](https://plg.uwaterloo.ca/~gvcormac/cormacksigir09-rrf.pdf)
- [Scalar Quantization in Qdrant](https://qdrant.tech/articles/scalar-quantization/)

---

**Decisión final**: Qdrant es la mejor opción para Alejandria considerando balance entre performance, facilidad operacional, costos y escalabilidad. Proporciona búsqueda vectorial de alta calidad sin la complejidad de Vespa ni los costos de Pinecone.
