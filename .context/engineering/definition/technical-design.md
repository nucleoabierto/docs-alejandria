---
id: ENG-DEF-001
area: "engineering"
type: "definition"
title: "Technical Design Document - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["ENG-DEC-0001", "ENG-DEC-0002", "PROD-DEF-0001", "ENG-GUIDE-0001"]
---

Documento técnico completo que define la arquitectura y especificaciones de implementación de Alejandria. Contiene visión arquitectónica de alto nivel, stack tecnológico, modelo de datos detallado, diseño de APIs, estrategia de testing, plan de despliegue y consideraciones de escalabilidad y seguridad para el sistema de gestión de conocimiento organizacional.

---

# Technical Design Document (TDD) - Alejandria

## 1. Overview: La Tecnología que Hace Posible la Magia

Alejandria nace de una convicción profunda: **el conocimiento organizacional es el activo más valioso y, paradójicamente, el más descuidado**. Sabemos lo frustrante que es pasar 1.8 horas cada día buscando esa decisión importante que alguien tomó hace meses. Imagina un futuro donde encontrar conocimiento estratégico tome segundos, no horas.

**Este documento técnico describe cómo hacemos posible esa visión.**

Alejandria no es solo otro sistema de documentación. Es un sistema vivo de gestión de conocimiento organizacional que:

**🧠 Entiende tu mundo real**
- Conecta decisiones de negocio con implementación técnica
- Aprende automáticamente de GitHub, Slack, Notion y Jira
- Preserva el contexto que tu equipo necesita cada día

**🔗 Encuentra lo que realmente necesitas**
- Búsqueda semántica que entiende lenguaje de negocio y técnico
- Descubre relaciones que nadie más ve
- Entrega contexto relevante sin que tengas que pedirlo

**⚡ Escala con tu organización**
- Soporta miles de búsquedas simultáneas
- Crece con tu conocimiento, no lo limita
- Funciona tanto para humanos como para agentes de IA

La arquitectura que presentamos aquí está diseñada para resolver **el problema real que enfrentas cada día**: la pérdida sistemática de memoria institucional que cuesta a organizaciones como la tuya $455K-910K al año por cada equipo de 50 personas.

## 2. Architecture

### 2.1 Non-Functional Requirements

#### Performance
- **Search latency**: < 100ms (p95) para búsquedas semánticas
- **Document indexing**: < 30 segundos para documentos nuevos
- **API response time**: < 200ms promedio para endpoints CRUD
- **Concurrent users**: Soporte para 1,000 usuarios simultáneos
- **Throughput**: 10,000 búsquedas/segundo pico

#### Scalability
- **Horizontal scaling**: Soporte para múltiples instancias de backend
- **Database scaling**: PostgreSQL read replicas para consultas pesadas
- **Vector database**: Qdrant cluster con sharding automático
- **Storage growth**: 100GB/organización/año estimado
- **Search index**: 1M documentos por organización sin degradación

#### Reliability
- **Uptime**: 99.9% disponibilidad (8.76 horas downtime/mes máximo)
- **Data durability**: 99.999% (backup daily + replication)
- **Error recovery**: Automatic retry con exponential backoff
- **Graceful degradation**: Búsqueda keyword fallback si vector search falla

#### Security
- **Data encryption**: AES-256 at rest, TLS 1.3 in transit
- **Access control**: OAuth 2.0 + RBAC granular
- **Audit trail**: Todos los accesos y modificaciones logueados
- **Compliance**: GDPR, CCPA compliance ready

### 2.2 High-Level Architecture

```mermaid
graph TB
    subgraph "External Sources"
        WH["Webhooks<br/>(GitHub, etc)"]
        FE["Frontend<br/>(React)"]
        MCP["MCP Server<br/>(AI Agents)"]
    end
    
    subgraph "Alejandria Backend"
        API["Elixir/Phoenix Backend"]
        
        subgraph "Core APIs"
            DocAPI["Document API"]
            SearchAPI["Search API"]
            RelAPI["Relationships API"]
        end
        
        subgraph "Data Layer"
            PG[("PostgreSQL<br/>(Documents)")]
            QDR[("Qdrant<br/>(Embeddings)")]
            BW["Background<br/>Workers"]
        end
    end
    
    WH --> API
    FE --> API
    MCP --> API
    
    API --> DocAPI
    API --> SearchAPI
    API --> RelAPI
    
    DocAPI --> PG
    SearchAPI --> QDR
    SearchAPI --> PG
    RelAPI --> PG
    
    DocAPI --> BW
    SearchAPI --> BW
    
    style PG fill:#e1f5fe
    style QDR fill:#f3e5f5
    style BW fill:#fff3e0
    style API fill:#e8f5e8
```

### 2.2 Technology Stack

#### Backend
- **Elixir/Phoenix 1.7**: Alta concurrencia, fault-tolerance
- **PostgreSQL 15**: Almacenamiento de documentos, metadata, versiones
- **Qdrant 1.7**: Búsqueda vectorial y embeddings
- **Oban**: Background jobs para procesamiento asíncrono

#### Frontend
- **React 18**: UI moderna y reactiva
- **TailwindCSS**: Styling utility-first
- **React Query**: Manejo de estado y cache
- **React Flow** (Fase 2): Visualización de knowledge graph

#### AI/ML
- **nomic-embed-text-v1.5**: Embeddings de texto (384 dimensiones)
- **Chunking strategy**: 512 tokens con overlap 50 tokens
- **Semantic search**: Cosine similarity en Qdrant

#### Integrations
- **GitHub API**: Commits, PRs, issues
- **Shortcut API**: Tickets, user stories
- **MCP Protocol**: Agentes de IA

## 3. Data Model

### 3.1 Entity Relationship Diagram

```mermaid
erDiagram
    organizations ||--o{ documents : "has"
    documents ||--o{ document_versions : "has"
    documents ||--o{ embeddings : "contains"
    documents ||--o{ document_relationships : "source"
    documents ||--o{ document_relationships : "target"
    
    organizations {
        uuid id PK
        string name
        string slug UK
        timestamp created_at
        timestamp updated_at
    }
    
    documents {
        uuid id PK
        uuid organization_id FK
        string title
        text content
        string content_type
        string source_type
        string source_id
        jsonb metadata
        integer version
        string status
        timestamp created_at
        timestamp updated_at
    }
    
    document_versions {
        uuid id PK
        uuid document_id FK
        integer version
        text content
        jsonb diff
        uuid created_by
        timestamp created_at
    }
    
    document_relationships {
        uuid id PK
        uuid source_document_id FK
        uuid target_document_id FK
        string relationship_type
        float confidence
        jsonb metadata
        timestamp created_at
    }
    
    embeddings {
        uuid id PK
        uuid document_id FK
        integer chunk_index
        text chunk_content
        string qdrant_id
        string embedding_model
        timestamp created_at
    }
    
    style organizations fill:#e1f5fe
    style documents fill:#f3e5f5
    style document_versions fill:#fff3e0
    style document_relationships fill:#e8f5e8
    style embeddings fill:#fce4ec
```

### 3.2 PostgreSQL Schema

#### Core Tables

```sql
-- Organizations (Multi-tenancy)
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT valid_slug CHECK (slug ~ '^[a-z0-9-]+$')
);

-- Documents (Central entity)
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    content_type TEXT DEFAULT 'text/markdown',
    source_type TEXT NOT NULL, -- 'github', 'shortcut', 'manual', 'mcp', 'notion'
    source_id TEXT, -- ID en sistema origen
    metadata JSONB DEFAULT '{}',
    version INTEGER DEFAULT 1,
    status TEXT DEFAULT 'active', -- 'active', 'archived', 'deleted'
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT valid_source_type CHECK (source_type IN ('github', 'shortcut', 'manual', 'mcp', 'notion')),
    CONSTRAINT valid_status CHECK (status IN ('active', 'archived', 'deleted'))
);

-- Document Versions (Historial completo)
CREATE TABLE document_versions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    version INTEGER NOT NULL,
    content TEXT NOT NULL,
    diff JSONB, -- cambios respecto a versión anterior
    created_by UUID, -- usuario que hizo el cambio
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(document_id, version)
);

-- Relationships (Knowledge graph)
CREATE TABLE document_relationships (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    source_document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    target_document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    relationship_type TEXT NOT NULL,
    confidence FLOAT DEFAULT 1.0 CHECK (confidence >= 0 AND confidence <= 1),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    
    CONSTRAINT valid_relationship CHECK (relationship_type IN ('mentions', 'references', 'relates_to', 'implements', 'depends_on')),
    CHECK (source_document_id != target_document_id)
);

-- Embeddings (Referencia a Qdrant)
CREATE TABLE embeddings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
    chunk_index INTEGER NOT NULL,
    chunk_content TEXT NOT NULL,
    qdrant_id TEXT NOT NULL,
    embedding_model TEXT DEFAULT 'nomic-embed-text-v1.5',
    created_at TIMESTAMP DEFAULT NOW(),
    
    UNIQUE(document_id, chunk_index)
);
```

#### Performance Indexes

```sql
-- Organizations
CREATE INDEX idx_organizations_slug ON organizations(slug);

-- Documents
CREATE INDEX idx_documents_org_id ON documents(organization_id);
CREATE INDEX idx_documents_source ON documents(organization_id, source_type);
CREATE INDEX idx_documents_status ON documents(status);
CREATE INDEX idx_documents_created ON documents(created_at DESC);
CREATE INDEX idx_documents_fulltext ON documents USING gin(to_tsvector('english', title || ' ' || content));

-- Document Versions
CREATE INDEX idx_document_versions_doc_id ON document_versions(document_id, version DESC);

-- Relationships
CREATE INDEX idx_relationships_source ON document_relationships(source_document_id);
CREATE INDEX idx_relationships_target ON document_relationships(target_document_id);
CREATE INDEX idx_relationships_type ON document_relationships(relationship_type);

-- Embeddings
CREATE INDEX idx_embeddings_doc_id ON embeddings(document_id, chunk_index);
```

### 3.3 Qdrant Vector Database

#### Collection Configuration

```json
{
  "name": "documents",
  "vectors": {
    "size": 384,
    "distance": "Cosine",
    "on_disk": true
  },
  "payload_schema": {
    "document_id": "keyword",
    "chunk_index": "integer",
    "organization_id": "keyword",
    "source_type": "keyword",
    "created_at": "integer"
  },
  "optimizers_config": {
    "default_segment_number": 2,
    "max_segment_size": 200000,
    "memmap_threshold": 50000
  }
}
```

#### Search Strategy

```yaml
# Hybrid Search Approach
semantic_search:
  vector_similarity: true
  rerank: true
  limit: 10
  
keyword_search:
  postgresql_fulltext: true
  fuzzy_matching: true
  
result_fusion:
  algorithm: "reciprocal_rank_fusion"
  weights:
    semantic: 0.7
    keyword: 0.3
```

## 4. API Design

### 4.1 API Architecture Overview

```mermaid
graph TB
    subgraph "External Clients"
        WC["Web Client<br/>(React SPA)"]
        MCP["MCP Server<br/>(AI Agents)"]
        EXT["External APIs<br/>(GitHub, etc)"]
    end
    
    subgraph "API Layer"
        LB["API Gateway<br/>Load Balancer"]
        API["Alejandria API<br/>(REST + GraphQL)"]
        
        subgraph "API Services"
            DocAPI["Document API<br/>/api/v1/docs"]
            SearchAPI["Search API<br/>/api/v1/search"]
            RelAPI["Relationships API<br/>/api/v1/relationships"]
        end
    end
    
    WC --> LB
    MCP --> LB
    EXT --> LB
    
    LB --> API
    
    API --> DocAPI
    API --> SearchAPI
    API --> RelAPI
    
    style WC fill:#e3f2fd
    style MCP fill:#f3e5f5
    style EXT fill:#fff3e0
    style LB fill:#e8f5e8
    style API fill:#fce4ec
```

### 4.2 API Specifications

#### 4.2.1 Document API

**Endpoints:**

```yaml
# Document CRUD Operations
GET    /api/v1/documents          # List documents
POST   /api/v1/documents          # Create document
GET    /api/v1/documents/:id      # Get document
PUT    /api/v1/documents/:id      # Update document
DELETE /api/v1/documents/:id      # Delete document

# Document Management
GET    /api/v1/documents/:id/versions     # Get version history
POST   /api/v1/documents/:id/restore     # Restore version
GET    /api/v1/documents/:id/related     # Get related documents
```

**Request/Response Examples:**

```json
// GET /api/v1/documents?organization_id=org123&source_type=github
{
  "data": [
    {
      "id": "doc-123",
      "title": "API Authentication Strategy",
      "content_type": "text/markdown",
      "source_type": "github",
      "source_id": "commit-abc123",
      "version": 3,
      "status": "active",
      "created_at": "2026-03-30T10:00:00Z",
      "updated_at": "2026-03-30T15:30:00Z",
      "metadata": {
        "author": "john.doe",
        "repository": "alejandria/api",
        "tags": ["security", "authentication"]
      }
    }
  ],
  "pagination": {
    "total": 150,
    "page": 1,
    "per_page": 20,
    "total_pages": 8
  }
}

// POST /api/v1/documents
{
  "title": "Database Migration Strategy",
  "content": "# Database Migration\n\nThis document describes...",
  "content_type": "text/markdown",
  "source_type": "manual",
  "organization_id": "org-123",
  "metadata": {
    "author": "jane.smith",
    "tags": ["database", "migration"]
  }
}
```

#### 4.2.2 Search API

**Endpoints:**

```yaml
# Search Operations
GET    /api/v1/search              # Hybrid search
GET    /api/v1/search/semantic     # Semantic search only
GET    /api/v1/search/keyword      # Keyword search only
GET    /api/v1/search/suggestions  # Autocomplete suggestions
```

**Search Request:**

```json
// GET /api/v1/search?q=authentication&org_id=org123&limit=10
{
  "query": "authentication",
  "organization_id": "org-123",
  "filters": {
    "source_types": ["github", "manual"],
    "date_range": {
      "from": "2026-01-01",
      "to": "2026-03-30"
    },
    "tags": ["security", "api"]
  },
  "search_options": {
    "limit": 10,
    "offset": 0,
    "include_content": true,
    "highlight": true
  }
}
```

**Search Response:**

```json
{
  "results": [
    {
      "document": {
        "id": "doc-123",
        "title": "API Authentication Strategy",
        "content_type": "text/markdown",
        "source_type": "github"
      },
      "score": 0.95,
      "match_type": "semantic",
      "highlights": [
        "OAuth 2.0 <mark>authentication</mark> flow",
        "JWT token <mark>authentication</mark>"
      ],
      "chunks": [
        {
          "content": "We implement OAuth 2.0 authentication...",
          "score": 0.98,
          "position": 42
        }
      ]
    }
  ],
  "search_metadata": {
    "total_results": 25,
    "search_time_ms": 45,
    "search_strategy": "hybrid",
    "weights_used": {
      "semantic": 0.7,
      "keyword": 0.3
    }
  }
}
```

#### 4.2.3 Relationships API

**Endpoints:**

```yaml
# Relationship Operations
GET    /api/v1/relationships              # Get relationships
POST   /api/v1/relationships              # Create relationship
DELETE /api/v1/relationships/:id          # Delete relationship
GET    /api/v1/documents/:id/related     # Get related documents
GET    /api/v1/relationships/graph        # Get knowledge graph
```

**Relationship Response:**

```json
{
  "relationships": [
    {
      "id": "rel-123",
      "source_document_id": "doc-123",
      "target_document_id": "doc-456",
      "relationship_type": "implements",
      "confidence": 0.92,
      "metadata": {
        "detected_by": "semantic_analysis",
        "evidence": "Document mentions implementation details"
      },
      "created_at": "2026-03-30T10:00:00Z"
    }
  ]
}
```

### 4.3 API Standards & Conventions

#### 4.3.1 RESTful Design Principles

- **Resource-based URLs**: Nouns, not verbs (`/documents` vs `/getDocuments`)
- **HTTP Methods**: Proper use of GET, POST, PUT, DELETE
- **Status Codes**: Standard HTTP status codes (200, 201, 400, 404, 500)
- **Pagination**: `limit`, `offset`, `page` parameters
- **Filtering**: Query parameters for filtering (`?status=active&source=github`)
- **Sorting**: `sort_by` and `order` parameters

#### 4.3.2 Response Format Standards

```json
{
  "data": [...],           // Primary data
  "metadata": {...},       // Response metadata
  "links": {...},          // HATEOAS links
  "errors": [...]          // Validation errors (if any)
}
```

#### 4.3.3 Error Handling

```json
// 400 Bad Request
{
  "errors": [
    {
      "code": "INVALID_PARAMETER",
      "message": "organization_id is required",
      "field": "organization_id",
      "value": null
    }
  ],
  "error_id": "err-abc123",
  "timestamp": "2026-03-30T10:00:00Z"
}

// 500 Internal Server Error
{
  "errors": [
    {
      "code": "INTERNAL_ERROR",
      "message": "An unexpected error occurred"
    }
  ],
  "error_id": "err-def456",
  "timestamp": "2026-03-30T10:00:00Z",
  "request_id": "req-ghi789"
}
```

### 4.4 Authentication & Authorization

#### 4.4.1 Authentication Methods

```yaml
# API Keys (for MCP servers)
Authorization: Bearer api-key-abc123

# JWT Tokens (for web clients)
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

# OAuth 2.0 (for external integrations)
Authorization: Bearer oauth-access-token
```

#### 4.4.2 Authorization Scopes

```yaml
# Document scopes
documents:read     # Read documents
documents:write    # Create/update documents
documents:delete   # Delete documents

# Search scopes
search:query       # Perform searches
search:suggest     # Get suggestions

# Relationship scopes
relationships:read  # View relationships
relationships:write # Create relationships
```

### 4.5 Rate Limiting & Quotas

```yaml
# Rate Limits by Plan
Free:
  requests_per_minute: 60
  searches_per_hour: 100
  documents_per_month: 100

Business:
  requests_per_minute: 1000
  searches_per_hour: 10000
  documents_per_month: 10000

Enterprise:
  requests_per_minute: 10000
  searches_per_hour: 100000
  documents_per_month: unlimited
```

### 4.6 API Versioning Strategy

- **URL Versioning**: `/api/v1/`, `/api/v2/`
- **Backward Compatibility**: Maintain v1 for at least 12 months
- **Deprecation Policy**: 6-month deprecation notice
- **Breaking Changes**: Only in major versions

## 5. Core Components

### 5.1 Component Architecture Overview

```mermaid
graph TB
    subgraph "Alejandria Core System"
        subgraph "Ingestion Layer"
            WH["Webhook<br/>Receivers"]
            API["API<br/>Ingestor"]
            FU["File<br/>Uploader"]
        end
        
        subgraph "Processing Layer"
            DP["Document<br/>Processor"]
            CN["Content<br/>Normalizer"]
            EG["Embedding<br/>Generator"]
            KGB["Knowledge<br/>Graph Builder"]
        end
        
        subgraph "Search & Discovery Layer"
            SE["Search Engine<br/>& Ranker"]
            RD["Result<br/>Deduplicator"]
            RB["Relevance<br/>Booster"]
        end
        
        subgraph "Storage Layer"
            PG[("PostgreSQL<br/>Documents")]
            QDR[("Qdrant<br/>Vectors")]
            REDIS[("Redis<br/>Cache")]
            FS[("File<br/>Storage")]
        end
    end
    
    WH --> DP
    API --> DP
    FU --> DP
    
    DP --> CN
    CN --> EG
    EG --> KGB
    
    DP --> SE
    EG --> SE
    KGB --> SE
    
    SE --> RD
    RD --> RB
    
    DP --> PG
    SE --> QDR
    SE --> PG
    SE --> REDIS
    DP --> FS
    
    style WH fill:#e3f2fd
    style API fill:#e3f2fd
    style FU fill:#e3f2fd
    style DP fill:#e8f5e8
    style CN fill:#f3e5f5
    style EG fill:#fff3e0
    style KGB fill:#fce4ec
    style SE fill:#e1f5fe
    style PG fill:#e8f5e8
    style QDR fill:#f3e5f5
    style REDIS fill:#fff3e0
    style FS fill:#fce4ec
```

### 5.2 Document Processing Pipeline

#### 5.2.1 Processing Flow

```yaml
# Document Lifecycle Pipeline
stages:
  - name: "ingestion"
    description: "Receive documents from external sources"
    components: ["WebhookReceiver", "APIIngestor", "FileUploader"]
    
  - name: "normalization"
    description: "Standardize content format and extract metadata"
    components: ["ContentNormalizer", "MetadataExtractor", "FormatConverter"]
    
  - name: "chunking"
    description: "Split content into searchable chunks"
    components: ["ContentChunker", "OverlapManager", "ChunkValidator"]
    
  - name: "embedding"
    description: "Generate vector embeddings for semantic search"
    components: ["EmbeddingGenerator", "BatchProcessor", "QualityAssurance"]
    
  - name: "indexing"
    description: "Store in search databases and build relationships"
    components: ["VectorIndexer", "RelationshipDetector", "GraphBuilder"]
    
  - name: "notification"
    description: "Notify stakeholders of processing completion"
    components: ["EventPublisher", "WebhookNotifier", "RealtimeUpdater"]
```

#### 5.2.2 Component Specifications

**Content Normalizer:**
```yaml
responsibilities:
  - Convert markdown, HTML, plain text to standard format
  - Extract and normalize metadata (author, date, tags)
  - Clean and sanitize content
  - Detect content type and structure

input_format:
  raw_content: string
  source_metadata: object
  content_type: string

output_format:
  normalized_content: string
  extracted_metadata: object
  content_structure: object
  quality_score: float

processing_rules:
  - Remove HTML tags and preserve text
  - Normalize markdown syntax
  - Extract code blocks and preserve formatting
  - Detect and preserve document structure
```

**Content Chunker:**
```yaml
strategy: "semantic_chunking"
parameters:
  chunk_size: 512 tokens
  overlap: 50 tokens
  respect_boundaries: true
  min_chunk_size: 100 tokens

algorithms:
  - "token_based_chunking"
  - "semantic_boundary_detection"
  - "code_block_preservation"
  - "table_structure_preservation"

quality_metrics:
  semantic_coherence: >0.8
  boundary_respect: >0.9
  size_variance: <20%
```

**Embedding Generator:**
```yaml
models:
  primary: "nomic-embed-text-v1.5"
  fallback: "text-embedding-ada-002"
  dimensions: 384
  
batch_processing:
  batch_size: 32
  timeout: 30s
  retry_attempts: 3
  
quality_assurance:
  embedding_validation: true
  similarity_threshold: 0.7
  outlier_detection: true
```

### 5.3 Search Engine Architecture

#### 5.3.1 Search Pipeline

```yaml
# Hybrid Search Architecture
search_pipeline:
  query_processing:
    - "query_normalization"
    - "intent_detection"
    - "query_expansion"
    
  parallel_search:
    semantic_search:
      engine: "qdrant"
      algorithm: "cosine_similarity"
      reranking: true
      
    keyword_search:
      engine: "postgresql_fulltext"
      algorithm: "tsvector_ranking"
      fuzzy_matching: true
      
    graph_search:
      engine: "neo4j_apache_age"
      algorithm: "graph_traversal"
      relationship_weighting: true
      
  result_fusion:
    algorithm: "reciprocal_rank_fusion"
    weights:
      semantic: 0.7
      keyword: 0.2
      graph: 0.1
    
  post_processing:
    - "result_deduplication"
    - "relevance_boosting"
    - "access_control_filtering"
    - "result_highlighting"
```

#### 5.3.2 Performance Optimizations

```yaml
caching_strategy:
  query_cache:
    ttl: 1 hour
    max_size: 10000 queries
    eviction_policy: "lru"
    
  result_cache:
    ttl: 30 minutes
    max_size: 50000 results
    cache_key: "query_hash + org_id + filters"
    
  embedding_cache:
    ttl: 24 hours
    storage: "redis_cluster"
    compression: "gzip"

index_optimization:
  vector_index:
    type: "hnsw"
    m_parameter: 16
    ef_construction: 200
    
  text_index:
    type: "gin"
    configuration: "english"
    weight_columns: ["title", "content"]
```

### 5.4 Knowledge Graph Components

#### 5.4.1 Relationship Detection

```yaml
relationship_types:
  explicit:
    - "mentions" (document A mentions document B)
    - "references" (document A references document B)
    - "implements" (document A implements document B)
    
  implicit:
    - "relates_to" (semantic similarity)
    - "depends_on" (dependency detection)
    - "conflicts_with" (contradiction detection)
    
  temporal:
    - "supersedes" (version replacement)
    - "precedes" (chronological order)
    - "responds_to" (response relationship)

detection_methods:
  pattern_matching:
    - regex_patterns for references
    - citation_formats
    - link_structures
    
  semantic_analysis:
    - entity_recognition
    - topic_modeling
    - sentiment_analysis
    
  graph_algorithms:
    - community_detection
    - centrality_analysis
    - path_finding
```

#### 5.4.2 Graph Construction

```yaml
graph_schema:
  nodes:
    document:
      properties: ["id", "title", "type", "created_at", "metadata"]
      indexes: ["title", "type", "created_at"]
      
    entity:
      properties: ["name", "type", "category", "confidence"]
      indexes: ["name", "type"]
      
  edges:
    mentions:
      properties: ["confidence", "context", "position"]
      weight: 0.8
      
    implements:
      properties: ["version", "status", "verification"]
      weight: 1.0

construction_pipeline:
  - "entity_extraction"
  - "relationship_identification"
  - "confidence_scoring"
  - "graph_validation"
  - "index_optimization"
```

## 6. Integration Patterns

### 6.1 Integration Architecture Overview

```mermaid
graph TB
    subgraph "External Systems"
        GH["GitHub<br/>Connector"]
        SC["Shortcut<br/>Connector"]
        SL["Slack<br/>Connector"]
    end
    
    subgraph "Integration Layer"
        subgraph "External Connectors"
            GH2["GitHub<br/>Connector"]
            SC2["Shortcut<br/>Connector"]
            SL2["Slack<br/>Connector"]
        end
        
        subgraph "Event Processing"
            WH["Webhook<br/>Receiver"]
            EP["Event<br/>Processor"]
            MQ["Message<br/>Queue"]
            DLQ["Dead<br/>Letter"]
        end
        
        subgraph "Protocol Adapters"
            MCP["MCP<br/>Server"]
            REST["REST<br/>Adapter"]
            WS["WebSocket<br/>Adapter"]
        end
    end
    
    subgraph "Core Processing"
        DPP["Document<br/>Processing<br/>Pipeline"]
    end
    
    GH --> GH2
    SC --> SC2
    SL --> SL2
    
    GH2 --> WH
    SC2 --> WH
    SL2 --> WH
    
    WH --> EP
    EP --> MQ
    MQ --> DPP
    
    EP --> DLQ
    
    MCP --> DPP
    REST --> DPP
    WS --> DPP
    
    style GH fill:#e3f2fd
    style SC fill:#e3f2fd
    style SL fill:#e3f2fd
    style WH fill:#e8f5e8
    style EP fill:#f3e5f5
    style MQ fill:#fff3e0
    style DLQ fill:#ffebee
    style MCP fill:#fce4ec
    style REST fill:#fce4ec
    style WS fill:#fce4ec
    style DPP fill:#e1f5fe
```

### 6.2 Integration Patterns

#### 6.2.1 Webhook Pattern

```yaml
# Webhook Integration Pattern
webhook_specification:
  endpoints:
    - path: "/webhooks/github"
      events: ["push", "pull_request", "issues"]
      authentication: "signature_verification"
      
    - path: "/webhooks/shortcut"
      events: ["story_created", "story_updated"]
      authentication: "api_key"
      
    - path: "/webhooks/slack"
      events: ["message", "file_shared"]
      authentication: "verification_token"

  processing_pipeline:
    - "signature_validation"
    - "event_parsing"
    - "data_extraction"
    - "document_creation"
    - "relationship_detection"
    - "notification_dispatch"

  error_handling:
    retry_strategy: "exponential_backoff"
    max_attempts: 3
    dead_letter_queue: true
    alerting: true
```

#### 6.2.2 Polling Pattern

```yaml
# Scheduled Polling Pattern
polling_configuration:
  github:
    schedule: "every 5 minutes"
    endpoints:
      - "/repos/{org}/{repo}/commits"
      - "/repos/{org}/{repo}/pulls"
      - "/repos/{org}/{repo}/issues"
    pagination:
      per_page: 100
      max_pages: 10
      
  shortcut:
    schedule: "every 10 minutes"
    endpoints:
      - "/stories"
      - "/epics"
    filters:
      updated_since: "last_sync_time"
      state: "done"

  rate_limiting:
    requests_per_minute: 60
    burst_size: 10
    backoff_strategy: "linear"
```

#### 6.2.3 Event-Driven Pattern

```yaml
# Event-Driven Integration
event_schema:
  document_created:
    type: "object"
    properties:
      document_id: "string"
      source_type: "string"
      source_id: "string"
      organization_id: "string"
      metadata: "object"
      timestamp: "datetime"
      
  relationship_detected:
    type: "object"
    properties:
      source_document_id: "string"
      target_document_id: "string"
      relationship_type: "string"
      confidence: "number"
      evidence: "object"
      timestamp: "datetime"

event_handlers:
  document_created:
    - "trigger_embedding_generation"
    - "update_search_index"
    - "detect_relationships"
    - "send_notifications"
    
  relationship_detected:
    - "update_knowledge_graph"
    - "reindex_related_documents"
    - "trigger_analysis"
```

### 6.3 External System Connectors

#### 6.3.1 GitHub Connector

```yaml
github_connector:
  authentication:
    type: "app_installation"
    permissions:
      - "contents:read"
      - "pull_requests:read"
      - "issues:read"
      - "metadata:read"
      
  data_extraction:
    commits:
      fields: ["sha", "message", "author", "timestamp", "files_changed"]
      processing:
        - "extract_commit_message"
        - "parse_code_changes"
        - "identify_technical_decisions"
        - "generate_document_summary"
        
    pull_requests:
      fields: ["number", "title", "body", "state", "merge_commit"]
      processing:
        - "extract_pr_description"
        - "parse_review_comments"
        - "identify_requirements"
        - "generate_implementation_doc"
        
    issues:
      fields: ["number", "title", "body", "labels", "state"]
      processing:
        - "extract_issue_description"
        - "parse_user_requirements"
        - "identify_feature_requests"
        - "generate_requirement_doc"
        
  synchronization:
    strategy: "incremental_sync"
    checkpoint_storage: "database"
    conflict_resolution: "latest_wins"
    batch_size: 50
```

#### 6.3.2 Shortcut Connector

```yaml
shortcut_connector:
  authentication:
    type: "api_token"
    token_source: "environment_variable"
    
  data_extraction:
    stories:
      fields: ["id", "name", "description", "story_type", "state"]
      processing:
        - "extract_user_story"
        - "parse_acceptance_criteria"
        - "identify_business_value"
        - "generate_requirement_doc"
        
    epics:
      fields: ["id", "name", "description", "state"]
      processing:
        - "extract_epic_description"
        - "parse_business_objectives"
        - "identify_stakeholder_needs"
        - "generate_strategic_doc"
        
  mapping:
    story_types_to_document_types:
      "feature": "user_story"
      "bug": "bug_report"
      "chore": "technical_task"
      
    states_to_document_status:
      "done": "completed"
      "in progress": "in_progress"
      "ready": "planned"
```

#### 6.3.3 Slack Connector

```yaml
slack_connector:
  authentication:
    type: "bot_token"
    scopes: ["channels:history", "files:read", "users:read"]
    
  data_extraction:
    messages:
      filters:
        - "contains_decision_keywords"
        - "mentions_documents"
        - "technical_discussions"
      processing:
        - "extract_decision_context"
        - "identify_participants"
        - "parse_action_items"
        - "generate_meeting_notes"
        
    files:
      types: ["document", "text", "code"]
      processing:
        - "extract_file_content"
        - "identify_document_type"
        - "extract_metadata"
        - "generate_document_entry"
        
  real_time_processing:
    events: ["message", "file_shared", "reaction_added"]
    processing_delay: "5_seconds"
    batch_size: 10
```

### 6.4 MCP (Model Context Protocol) Integration

#### 6.4.1 MCP Server Architecture

```yaml
mcp_server:
  protocol_version: "2024-11-05"
  capabilities:
    tools: true
    resources: true
    prompts: true
    
  tools:
    search_documents:
      description: "Search documents in the knowledge base"
      parameters:
        query: {type: "string", required: true}
        organization_id: {type: "string", required: true}
        limit: {type: "integer", default: 10}
        
    get_document:
      description: "Retrieve a specific document"
      parameters:
        document_id: {type: "string", required: true}
        include_content: {type: "boolean", default: true}
        
    find_related:
      description: "Find documents related to a given document"
      parameters:
        document_id: {type: "string", required: true}
        relationship_types: {type: "array", items: {type: "string"}}
        
  resources:
    documents:
      uri_template: "alejandria://documents/{document_id}"
      mime_type: "text/markdown"
      
    search_results:
      uri_template: "alejandria://search/{query_hash}"
      mime_type: "application/json"
```

#### 6.4.2 MCP Client Integration

```yaml
mcp_client_support:
  supported_clients:
    - "claude-desktop"
    - "cursor"
    - "vscode-copilot"
    - "github-copilot"
    
  client_configurations:
    claude_desktop:
      server_name: "alejandria"
      command: "alejandria-mcp-server"
      args: ["--config", "/etc/alejandria/mcp.json"]
      
    cursor:
      server_name: "alejandria"
      command: "alejandria-mcp-server"
      args: ["--cursor-mode"]
      
  authentication:
    method: "api_key"
    key_rotation: "monthly"
    key_source: "environment_variable"
```

### 6.5 Error Handling & Resilience

#### 6.5.1 Retry Strategies

```yaml
retry_policies:
  transient_errors:
    max_attempts: 3
    backoff: "exponential"
    base_delay: "1_second"
    max_delay: "30_seconds"
    
  rate_limit_errors:
    max_attempts: 5
    backoff: "linear"
    base_delay: "60_seconds"
    max_delay: "300_seconds"
    
  network_errors:
    max_attempts: 2
    backoff: "immediate"
    circuit_breaker: true
    
  authentication_errors:
    max_attempts: 1
    backoff: "none"
    alerting: "immediate"
```

#### 6.5.2 Circuit Breaker Pattern

```yaml
circuit_breaker:
  failure_threshold: 5
  recovery_timeout: "60_seconds"
  half_open_max_calls: 3
  
  monitored_services:
    github_api:
      timeout: "30_seconds"
      error_threshold: 0.1
      
    shortcut_api:
      timeout: "15_seconds"
      error_threshold: 0.05
      
    slack_api:
      timeout: "10_seconds"
      error_threshold: 0.2
```

#### 6.5.3 Dead Letter Queue

```yaml
dead_letter_queue:
  max_size: 10000
  retention_period: "7_days"
  alert_threshold: 100
  
  processing:
    retry_attempts: 3
    manual_review_required: true
    automatic_cleanup: true
    
  monitoring:
    queue_size_alert: 1000
    age_alert: "24_hours"
    error_rate_alert: 0.1
```

## 7. Performance Considerations

### 7.1 Caching Strategy
- **Redis**: Cache de búsquedas frecuentes (TTL 1 hora)
- **Database**: Connection pooling con Poolboy
- **Embeddings**: Cache en memoria para documentos recientes

### 7.2 Scalability
- **Horizontal scaling**: Phoenix nodes con clustering
- **Database**: Read replicas para consultas pesadas
- **Qdrant**: Sharding por organización

### 7.3 Background Processing
- **Oban**: Colas prioritarias para diferentes tareas
- **Embedding generation**: Baja prioridad, batch processing
- **Webhook processing**: Alta prioridad, tiempo real

## 8. Security

### 8.1 Authentication
- **JWT tokens** para API access
- **OAuth 2.0** para integraciones (GitHub, Shortcut)
- **API keys** para MCP servers

### 8.2 Authorization
- **Organization-based isolation** (multi-tenant)
- **Role-based access control** (admin, editor, viewer)
- **Document-level permissions**

### 8.3 Data Protection
- **Encryption at rest** (PostgreSQL)
- **Encryption in transit** (TLS)
- **PII detection** y redacción automática

## 9. Monitoring & Observability

### 9.1 Metrics
- **Search latency**: p50, p95, p99
- **Embedding generation time**
- **Document processing throughput**
- **API error rates**

### 9.2 Logging
- **Structured logging** con Logger
- **Correlation IDs** para trace requests
- **Error tracking** con Sentry

### 9.3 Health Checks
- **Database connectivity**
- **Qdrant cluster status**
- **Background job queue health**

## 10. Deployment Architecture

### 10.1 Infrastructure
```
┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   CDN           │
└─────────┬───────┘    └─────────────────┘
          │
          ▼
┌─────────────────────────────────────────┐
│            Kubernetes Cluster           │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
│  │ Phoenix │  │ Qdrant  │  │ Postgres│  │
│  │ Nodes   │  │ Cluster │  │ Cluster │  │
│  └─────────┘  └─────────┘  └─────────┘  │
└─────────────────────────────────────────┘
```

### 10.2 CI/CD Pipeline
- **GitHub Actions** para build y test
- **Docker containers** para deployment
- **Helm charts** para Kubernetes
- **GitOps** con ArgoCD

## 11. Testing Strategy

### 11.1 Unit Tests
- **ExUnit** para lógica de negocio
- **Mock integraciones** externas
- **Property-based testing** para algoritmos complejos

### 11.2 Integration Tests
- **Test containers** para PostgreSQL y Qdrant
- **Webhook testing** con test endpoints
- **End-to-end scenarios** críticos

### 11.3 Performance Testing
- **Load testing** con k6 para API endpoints (target: 10,000 RPS)
- **Search performance** bajo carga (p95 < 100ms)
- **Memory profiling** para embedding generation
- **Database query optimization** con EXPLAIN ANALYZE

### 11.4 Security Testing
- **Penetration testing** trimestral con herramientas automatizadas
- **OAuth flow testing** para todos los proveedores
- **Data leakage testing** para validación de multi-tenancy
- **Input validation testing** contra inyecciones y XSS

### 11.5 Quality Gates
- **CI/CD pipeline** con test suite completo antes de deploy
- **Performance benchmarks** que deben pasar para merge
- **Security scans** automáticos en cada PR
- **Documentation coverage** >80% para APIs públicas

## 12. Deployment Strategy

### 12.1 Environments
- **Development**: Docker Compose local con servicios completos
- **Staging**: Réplica exacta de producción con datos de prueba
- **Production**: Kubernetes cluster con alta disponibilidad

### 12.2 Zero-Downtime Deployments
- **Blue-green deployments** en Kubernetes
- **Database migrations** reversibles con rollback automático
- **Feature flags** para cambios riesgosos
- **Health checks** durante rollout con rollback automático en fallas

### 12.3 Infrastructure as Code
- **Terraform** para provisioning de infraestructura
- **Helm charts** para configuración de Kubernetes
- **Ansible** para configuración de servidores
- **Monitoring** integrado con Prometheus + Grafana

### 12.4 Backup & Disaster Recovery
- **Automated backups** diarios con retención 30 días
- **Point-in-time recovery** para PostgreSQL
- **Cross-region replication** para Qdrant
- **Disaster recovery drills** mensuales con RTO < 4 horas

## 13. Migration Strategy

### 13.1 Data Migration
- **Incremental sync** para organizaciones grandes
- **Backfill strategy** para embeddings históricos
- **Rollback capability** para cambios críticos

### 13.2 Zero-Downtime Deployments
- **Blue-green deployments** en Kubernetes
- **Database migrations** reversibles
- **Feature flags** para cambios riesgosos
