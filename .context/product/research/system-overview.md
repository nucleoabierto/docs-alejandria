---
id: PROD-RES-0001
area: "product"
type: "research"
title: "System Overview - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["PROD-DEF-0001", "PROD-DEF-0005", "ENG-DEF-0001", "EXEC-STRAT-0001"]
---

Documento técnico completo que proporciona una visión arquitectónica de alto nivel de Alejandria como sistema de gestión de conocimiento organizacional. Contiene descripción del stack tecnológico, arquitectura de componentes, flujos de datos, patrones de diseño, módulos principales y consideraciones de performance para facilitar comprensión rápida por parte de desarrolladores y agentes de IA.

---

# System Overview - Alejandria

## Qué es Alejandria

Imagina tener un compañero de equipo que recuerda cada decisión técnica, cada contexto importante y cada lección aprendida. Así es como funciona Alejandria: es la memoria colectiva de tu organización, diseñada para que tanto humanos como agentes de IA puedan acceder al conocimiento estratégico cuando lo necesitan.

**Alejandria** es un sistema vivo de gestión de conocimiento organizacional que combina búsqueda semántica con integración automática de herramientas que ya usas.

### El Problema que Resolvemos

Sabemos lo frustrante que es sentir que tu equipo está perdiendo su memoria colectiva. La información que necesitas está por todas partes, pero al mismo tiempo en ninguna parte:

- 📝 **En Slack** - Decisiones técnicas enterradas en conversaciones
- 🔧 **En GitHub** - Contexto perdido entre cientos de commits  
- 📋 **En Notion** - Estrategias que nadie actualiza
- 🎯 **En Jira/Shortcut** - Historias sin conexión con la realidad

### La Solución que Ofrecemos

**🧠 Entiende tu mundo real**
- Conecta decisiones de negocio con implementación técnica
- Aprende automáticamente de GitHub, Slack, Notion y Jira
- Preserva el contexto que tu equipo necesita cada día

**🔗 Encuentra lo que realmente necesitas**
- Búsqueda semántica que entiende lenguaje de negocio y técnico
- Descubre relaciones que nadie más ve
- Entrega contexto relevante sin que tengas que pedirlo

## Stack Tecnológico

### Backend - El Corazón del Sistema

**Elixir/Phoenix 1.7** - Elegimos esta tecnología porque nos permite:
- Manejar miles de búsquedas simultáneas sin bloqueo
- Recuperarnos automáticamente de fallos
- Actualizar código sin downtime (hot code swapping)
- Escalar horizontalmente de forma nativa

**PostgreSQL 15+** - Donde guardamos:
- Documentos completos con versionamiento
- Metadata estructurada y relaciones
- Búsqueda full-text para queries exactas

**Qdrant 1.7+** - Nuestro motor de búsqueda semántica:
- Almacenamiento de embeddings vectoriales
- Búsqueda por similitud coseno
- Filtrado avanzado por metadata

**Oban 2.15+** - Para procesamiento asíncrono:
- Generación de embeddings en background
- Procesamiento de webhooks sin bloquear
- Reintentos automáticos con backoff exponencial

### AI/ML - La Magia Detrás de la Búsqueda

**nomic-embed-text-v1.5** - Nuestro modelo de embeddings:
- 384 dimensiones para representar significado
- Optimizado para texto técnico y de negocio
- Balance entre calidad y performance

**Estrategia de Chunking** - Dividimos documentos inteligentemente:
- 512 tokens por chunk con 50 de overlap
- Respetamos párrafos y estructura del código
- Mantenemos contexto entre fragmentos

**Búsqueda Híbrida** - Combinamos lo mejor de ambos mundos:
- Búsqueda semántica en Qdrant (entiende significado)
- Búsqueda full-text en PostgreSQL (encuentra exactas)
- Reciprocal Rank Fusion para mezclar resultados

### Infrastructure - Todo Funcionando Junto

**Docker** - Para desarrollo y deployment consistente
**GitHub Actions** - CI/CD automatizado
**CLI Tool** - Administración desde línea de comandos

## Arquitectura de Alto Nivel

La arquitectura de Alejandria está diseñada pensando en la experiencia real de los equipos de software. Todo fluye de manera natural, desde que un documento se crea hasta que un agente de IA lo encuentra para ayudarte.

```
┌─────────────────────────────────────────────────┐
│              MCP Server                          │
│  - Search Endpoint                               │
│  - Ingest Endpoint                               │
│  - Relationships Endpoint                       │
└─────────────┬───────────────────────────────────┘
              │ JSON-RPC/MCP Protocol
              ▼
┌─────────────────────────────────────────────────┐
│         Phoenix Backend (Elixir)                │
│  ┌──────────────┐  ┌──────────────┐            │
│  │  Document    │  │   Search     │            │
│  │  Context     │  │   Engine     │            │
│  └──────┬───────┘  └──────┬───────┘            │
│         │                  │                     │
│  ┌──────┴───────┐  ┌──────┴───────┐            │
│  │ Integrations │  │  Embeddings  │            │
│  │   Context    │  │   Context    │            │
│  └──────────────┘  └──────────────┘            │
└─────────┬──────────────────┬───────────────────┘
          │                  │
    ┌─────┴─────┐     ┌─────┴─────┐
    ▼           ▼     ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│PostgreSQL│ │  Qdrant │ │  Oban   │
│(Docs +  │ │(Vectors)│ │ (Jobs)  │
│Metadata)│ │         │ │         │
└─────────┘ └─────────┘ └─────────┘
```

## Decisiones Técnicas Clave

### ADR-0001: Elixir/Phoenix como Backend
**Razón**: Alta concurrencia, fault-tolerance, hot code swapping
**Trade-off**: Curva de aprendizaje vs robustez del sistema

### ADR-0002: Qdrant para Búsqueda Vectorial
**Razón**: Open source, self-hosted, buen performance, API limpia
**Trade-off**: Menos maduro que Pinecone vs control total y costos

### PostgreSQL como Fuente de Verdad
**Razón**: Performance 100x mejor que Git, queries SQL nativas
**Trade-off**: Mayor complejidad de infraestructura vs velocidad

## Data Model

### PostgreSQL

**Tablas principales:**

```sql
-- Organizations (single-org mode para MVP)
organizations(id, name, slug, created_at, updated_at)

-- Documents
documents(
  id, 
  organization_id, 
  title, 
  content, 
  content_type,
  source_type,  -- 'github', 'api', 'cli', 'mcp'
  metadata,     -- JSONB
  version,
  created_at,
  updated_at
)

-- Document Versions
document_versions(
  id,
  document_id,
  version,
  content,
  diff,  -- JSONB
  created_at
)

-- Relationships
document_relationships(
  id,
  source_document_id,
  target_document_id,
  relationship_type,  -- 'mentions', 'references', 'relates_to'
  confidence,
  created_at
)

-- Embeddings (referencia a Qdrant)
embeddings(
  id,
  document_id,
  chunk_index,
  chunk_content,
  qdrant_id,
  created_at
)
```

### Qdrant

**Collection: documents**
```yaml
vectors:
  size: 384
  distance: Cosine
payload:
  document_id: keyword
  chunk_index: integer
  organization_id: keyword
  text: text
```

## API Endpoints

### Documents
- `GET /api/v1/documents` - List documents
- `POST /api/v1/documents` - Create document
- `GET /api/v1/documents/:id` - Get document
- `PUT /api/v1/documents/:id` - Update document
- `DELETE /api/v1/documents/:id` - Delete document

### Search
- `GET /api/v1/search?q=query&org_id=xxx` - Hybrid search

### Relationships
- `GET /api/v1/documents/:id/related` - Related documents

### Webhooks
- `POST /api/v1/webhooks/github` - GitHub webhook

### MCP (AI Agents)
- `search_documents` - Search via MCP
- `get_document` - Get document via MCP
- `ingest_document` - Create document via MCP
- `list_related` - Get related documents via MCP

### CLI Tool
- `alejandria ingest` - Ingest document from file
- `alejandria search` - Search documents
- `alejandria list` - List documents
- `alejandria relationships` - Show document relationships

## Flujos Principales

### 1. Document Creation Flow

```
User creates document
  ↓
Phoenix: Documents.create_document()
  ↓
Save to PostgreSQL
  ↓
Queue Oban job: EmbeddingJob
  ↓
Background: Generate embeddings
  ↓
Store vectors in Qdrant
  ↓
Detect relationships
  ↓
Save relationships to PostgreSQL
```

### 2. Search Flow

```
User searches "authentication flow"
  ↓
Phoenix: SearchEngine.search()
  ↓
Parallel execution:
  ├─> PostgreSQL: full_text_search()
  └─> Qdrant: semantic_search()
  ↓
Merge results with RRF
  ↓
Return top 10 results
```

### 3. MCP Integration Flow

```
AI Agent (Claude Code/Cursor)
  ↓
MCP Protocol: JSON-RPC
  ↓
Phoenix: MCP.Server.handle_call()
  ↓
Execute operation (search/ingest/relationships)
  ↓
Return structured response
```

### 4. CLI Tool Flow

```
User runs: alejandria ingest --file ./docs/api.md
  ↓
Mix Task: Alejandria.CLI.Ingest
  ↓
Read file content
  ↓
Phoenix: Documents.create_document()
  ↓
Queue embedding job
  ↓
Return success/error
```

## Módulos Principales (Elixir)

### Contexts

```elixir
Alejandria.Documents
  - create_document/1
  - get_document!/1
  - list_documents/1
  - update_document/2
  - delete_document/1

Alejandria.Search
  - search/2
  - full_text_search/2
  - semantic_search/2
  - merge_results/2

Alejandria.Embeddings
  - generate/1
  - chunk_content/1
  - store_vectors/2

Alejandria.Integrations
  - GitHub.process_webhook/1

Alejandria.MCP
  - handle_call/2  - MCP protocol handler
  - search_documents/2
  - ingest_document/2
  - get_document/2

Alejandria.CLI
  - ingest/1
  - search/1
  - list/1
  - relationships/1

Alejandria.Relationships
  - detect_relationships/1
  - get_related_documents/1
```

### Workers (Oban)

```elixir
Alejandria.Workers.EmbeddingJob
  - Genera embeddings para documento
  - Prioridad: baja
  - Queue: :embeddings

Alejandria.Workers.WebhookJob
  - Procesa webhooks asíncronamente
  - Prioridad: alta
  - Queue: :webhooks
```

## Patrones de Código

### 1. Context Pattern

```elixir
defmodule Alejandria.Documents do
  alias Alejandria.Repo
  alias Alejandria.Documents.Document

  def create_document(attrs) do
    %Document{}
    |> Document.changeset(attrs)
    |> Repo.insert()
  end
end
```

### 2. Pipeline Pattern

```elixir
def process_document(document) do
  document
  |> extract_metadata()
  |> chunk_content()
  |> generate_embeddings()
  |> detect_relationships()
end
```

### 3. Supervision Tree

```elixir
children = [
  Alejandria.Repo,
  AlejandriaWeb.Endpoint,
  {Oban, Application.fetch_env!(:alejandria, Oban)},
  Alejandria.Qdrant.Client
]

Supervisor.start_link(children, strategy: :one_for_one)
```

## Convenciones

### Naming
- **Modules**: PascalCase (`Alejandria.Documents`)
- **Functions**: snake_case (`create_document/1`)
- **Variables**: snake_case (`document_id`)
- **Atoms**: snake_case (`:ok`, `:error`)

### Error Handling
```elixir
# Pattern matching
case Documents.create_document(attrs) do
  {:ok, document} -> # success
  {:error, changeset} -> # error
end

# With clause
with {:ok, document} <- Documents.get_document(id),
     {:ok, embeddings} <- Embeddings.generate(document) do
  # success
else
  {:error, reason} -> # error
end
```

### Testing
```elixir
# Unit test
test "creates document with valid attrs" do
  attrs = %{title: "Test", content: "Content"}
  assert {:ok, %Document{}} = Documents.create_document(attrs)
end

# Integration test
test "search returns relevant documents" do
  # Setup
  create_test_documents()
  
  # Execute
  results = Search.search("test query", org_id)
  
  # Assert
  assert length(results) > 0
end
```

## Performance Considerations

### Caching
- **Redis**: Cache de búsquedas frecuentes (TTL 1 hora)
- **ETS**: Cache en memoria para configuración
- **Database**: Connection pooling con Poolboy

### Indexing
```sql
-- PostgreSQL indexes
CREATE INDEX idx_documents_org_id ON documents(organization_id);
CREATE INDEX idx_documents_source_type ON documents(source_type);
CREATE INDEX idx_documents_content_gin ON documents USING gin(to_tsvector('english', content));
```

### Qdrant Optimization
```yaml
# Cuantización para reducir memoria
quantization:
  scalar:
    type: int8
    quantile: 0.99

# HNSW config para balance speed/recall
hnsw_config:
  m: 16
  ef_construct: 100
```

## Monitoring & Observability

### Telemetry Events
```elixir
:telemetry.execute(
  [:alejandria, :search, :execute],
  %{duration: duration, result_count: count},
  %{query: query, organization_id: org_id}
)
```

### Metrics
- **Search latency**: p50, p95, p99
- **Embedding generation time**
- **Document processing throughput**
- **API error rates**
- **Qdrant query performance**

### Health Checks
- `GET /health` - Application health
- `GET /health/db` - Database connectivity
- `GET /health/qdrant` - Qdrant connectivity
- `GET /health/jobs` - Oban queue health

## Common Queries

### "How does search work?"
→ Hybrid search: PostgreSQL full-text + Qdrant semantic, merged with RRF
→ See: `Alejandria.Search.search/2`

### "How are embeddings generated?"
→ nomic-embed-text-v1.5, 384 dimensions, chunked at 512 tokens
→ See: `Alejandria.Embeddings.generate/1`

### "How do webhooks work?"
→ Validate signature, extract data, queue Oban job, return 200
→ See: `Alejandria.Integrations.GitHub.process_webhook/1`

### "How are relationships detected?"
→ Explicit links (markdown), mentions (regex), semantic similarity
→ See: `Alejandria.Relationships.detect_relationships/1`

### "How is multi-tenancy implemented?"
→ MVP: Single-org mode con organization_id fijo
→ Fase 2: Organization-based isolation, queries filtrados por org_id
→ See: PostgreSQL schema, `organization_id` foreign key

### "How does MCP integration work?"
→ JSON-RPC protocol, endpoints para search/ingest/relationships
→ See: `Alejandria.MCP.Server.handle_call/2`

### "How do I use the CLI tool?"
→ Mix tasks para ingest, search, list, relationships
→ See: `Alejandria.CLI.*` modules

## Troubleshooting

### Search returns no results
1. Check if documents are indexed: `SELECT COUNT(*) FROM embeddings`
2. Check Qdrant collection: `curl http://localhost:6333/collections/documents`
3. Verify organization_id filter is correct

### Embeddings not generating
1. Check Oban queue: `Oban.check_queue(:embeddings)`
2. Check worker logs for errors
3. Verify embedding service is accessible

### Webhook not processing
1. Verify signature validation
2. Check Oban queue: `Oban.check_queue(:webhooks)`
3. Review webhook payload format

## Links Útiles

- [Technical Design](../engineering/architecture/technical-design.md)
- [ADR-0001: Elixir/Phoenix](../engineering/decisions/adr-0001-elixir-phoenix-backend.md)
- [ADR-0002: Qdrant](../engineering/decisions/adr-0002-qdrant-vector-search.md)
- [Development Setup](../engineering/guides/development-setup.md)
- [API Specification](../engineering/architecture/api-specification.md)

**Última actualización**: 2026-03-30  
**Optimizado para**: RAG, semantic search, AI agent context
