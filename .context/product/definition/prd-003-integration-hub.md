---
id: PROD-PRD-0003
area: "product"
type: "prd"
title: "PRD-003: Integration Hub (GitHub/Slack/Shortcut)"
author: "product-team"
status: "planned"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["PROD-PRD-0001", "PROD-STRAT-0008", "PROD-STRAT-0001"]
---

# Product Requirements Document - PRD-003: Integration Hub (GitHub/Slack/Shortcut)

**Version**: 1.0  
**Date**: 2026-03-30  
**Status**: Planned  
**Owner**: Integration Team  
**Stakeholders**: Engineering, DevOps, Product, Customer Success  

## 1. Executive Summary

### Product Vision
Hub de integración centralizado que sincroniza automáticamente conocimiento de GitHub, Slack y Shortcut con Alejandria, manteniendo la documentación siempre actualizada con el contexto real del equipo.

### Problem Statement
Las decisiones técnicas y conocimiento del equipo están fragmentados en múltiples sistemas (commits, PRs, Slack, tickets), causando documentación desactualizada, pérdida de contexto y dificultad para encontrar información relevante.

### Success Metrics (6 semanas)
- **Sync success rate**: > 99% de webhooks procesados exitosamente
- **Indexing latency**: < 30 segundos desde evento hasta búsqueda
- **Coverage**: > 80% de decisiones técnicas indexadas automáticamente
- **Data freshness**: < 5 minutos de latency para contenido nuevo

---

## 2. Target Users & Personas

### Primary Personas

#### 1. **DevOps Engineer**
- **Contexto**: Configura y mantiene integraciones, monitorea sync health
- **Pain Points**: "¿Cómo aseguro que todo se sincronice?", "¿Qué pasa si falla un webhook?"
- **Goals**: Setup robusto, monitoring completo, error recovery automático
- **Usage Frequency**: Diaria en setup, semanal en operaciones

#### 2. **Backend Developer**
- **Contexto**: Commitea código, crea PRs, participa en discusiones técnicas
- **Pain Points**: "Mis decisiones no se documentan automáticamente", "Tengo que duplicar información"
- **Goals**: Decisions auto-indexadas, contexto disponible fácilmente
- **Usage Frequency**: Continua (cada commit/PR)

#### 3. **Product Manager**
- **Contexto**: Gestiona tickets, roadmap, decisiones de producto
- **Pain Points**: "¿Dónde está el contexto de esta decisión?", "¿Por qué tomamos este camino?"
- **Goals**: Decisiones de producto indexadas, contexto histórico accesible
- **Usage Frequency**: Diaria en gestión de tickets

#### 4. **Tech Lead**
- **Contexto**: Revisa PRs, mantiene coherencia técnica
- **Pain Points**: "¿Qué decisiones se tomaron en este PR?", "¿Cuál es el contexto de este cambio?"
- **Goals**: Contexto de PRs disponible, decisiones técnicas rastreables
- **Usage Frequency**: Diaria en code reviews

### Secondary Personas

#### 5. **System Administrator**
- **Usage**: Configura API keys, manage permissions
- **Frequency**: Mensual en setup

#### 6. **Customer Success**
- **Usage**: Ayuda a clientes con configuración de integraciones
- **Frequency**: Semanal

---

## 3. Core Features & Requirements

### 3.1 GitHub Integration

#### 3.1.1 Webhook Processing

**User Story**: Como developer, quiero que mis commits y PRs se indexen automáticamente para tener contexto técnico disponible.

**Requirements**:
- ✅ **MUST**: Webhook endpoints para push, pull_request, issues events
- ✅ **MUST**: HMAC signature validation con repository secrets
- ✅ **MUST**: Event filtering solo para repositorios configurados
- ✅ **MUST**: Retry logic con exponential backoff
- ✅ **SHOULD**: Event deduplication por event ID
- ✅ **SHOULD**: Bulk processing para múltiples eventos

**Acceptance Criteria**:
- Webhook configurado en < 5 minutos
- Events procesados en < 30 segundos
- 99.9% de eventos procesados exitosamente
- No eventos duplicados en búsqueda

#### 3.1.2 Content Extraction

**User Story**: Como tech lead, quiero que el contenido técnico de commits y PRs se extraiga y clasifique automáticamente.

**Requirements**:
- ✅ **MUST**: Commit message parsing para identificar decisiones técnicas
- ✅ **MUST**: PR description y comments indexing
- ✅ **MUST**: Code diff analysis para detectar patrones importantes
- ✅ **MUST**: Automatic tagging (feature, bugfix, refactor, etc.)
- ✅ **SHOULD**: ADR detection en commit messages
- ✅ **SHOULD**: Link detection entre issues y PRs

**Acceptance Criteria**:
- Commit messages > 50 caracteres se indexan como documentos
- PR descriptions > 100 caracteres se indexan
- Diffs > 500 líneas se analizan para cambios significativos
- ADRs detectados con > 80% accuracy

#### 3.1.3 Repository Management

**User Story**: Como DevOps engineer, quiero configurar qué repositorios sincronizar y cómo procesar su contenido.

**Requirements**:
- ✅ **MUST**: Repository registration por organización
- ✅ **MUST**: Selective branch filtering (main, develop, feature/*)
- ✅ **MUST**: File type filtering (.md, .js, .py, .ex, etc.)
- ✅ **MUST**: Permission validation por repository access
- ✅ **SHOULD**: Historical import de repositorios existentes
- ✅ **SHOULD**: Repository grouping por proyecto/team

**Acceptance Criteria**:
- Repository setup en < 2 minutos
- Branch filtering configurable por UI
- Historical import de 1000 commits en < 5 minutos
- Solo repositorios con permisos accesibles

### 3.2 Slack Integration

#### 3.2.1 Slack API Connection

**User Story**: Como team member, quiero que conversaciones importantes de Slack se indexen para no perder decisiones informales.

**Requirements**:
- ✅ **MUST**: Slack API integration con Bot Token
- ✅ **MUST**: Channel selection por organización
- ✅ **MUST**: Real-time message ingestion via Events API
- ✅ **MUST**: Thread context preservation
- ✅ **SHOULD**: Historical message import (last 90 días)
- ✅ **SHOULD**: User mapping con organization members

**Acceptance Criteria**:
- Slack workspace conectado en < 3 minutos
- Real-time messages indexados en < 10 segundos
- Thread context mantenido en búsquedas
- Solo mensajes de canales seleccionados

#### 3.2.2 Content Classification

**User Story**: Como product manager, quiero que decisiones de Slack se clasifiquen automáticamente para fácil búsqueda.

**Requirements**:
- ✅ **MUST**: Message filtering por keywords y patrones
- ✅ **MUST**: Decision detection ("decided", "agreed", "conclusion")
- ✅ **MUST**: Technical discussion identification
- ✅ **MUST**: Action item extraction
- ✅ **SHOULD**: Sentiment analysis para priorización
- ✅ **SHOULD**: Topic modeling por canal

**Acceptance Criteria**:
- Decision messages identificados con > 75% accuracy
- Technical discussions clasificados correctamente
- Action items extraídos con timestamps
- Noise (emojis, memes) filtrado automáticamente

#### 3.2.3 Privacy & Compliance

**User Story**: Como system administrator, quiero controlar qué contenido de Slack se indexa para cumplir con políticas.

**Requirements**:
- ✅ **MUST**: Opt-out por canal o usuario específico
- ✅ **MUST**: PII detection y redacción
- ✅ **MUST**: Message retention policies configurables
- ✅ **MUST**: GDPR compliance con derecho al olvido
- ✅ **SHOULD**: Private channel handling con permisos explícitos
- ✅ **SHOULD**: DM exclusion por defecto

**Acceptance Criteria**:
- PII redactado automáticamente
- Mensajes eliminados de Slack eliminados de Alejandria
- Private channels solo con consentimiento explícito
- DMs nunca indexados sin opt-in explícito

### 3.3 Shortcut Integration

#### 3.3.1 Shortcut API Integration

**User Story**: Como product manager, quiero que stories y epics se indexen para tener contexto de producto accesible.

**Requirements**:
- ✅ **MUST**: Shortcut API v3 integration
- ✅ **MUST**: Webhook para story created/updated/deleted
- ✅ **MUST**: Story metadata indexing (epic, iteration, labels)
- ✅ **MUST**: Comments y attachments processing
- ✅ **SHOULD**: Workflow state tracking
- ✅ **SHOULD**: Team y workspace filtering

**Acceptance Criteria**:
- Shortcut workspace conectado en < 2 minutos
- Stories indexadas en < 15 segundos
- Epic relationships mantenidas
- Comments incluidos en búsqueda

#### 3.3.2 Product Context Extraction

**User Story**: Como product manager, quiero que decisiones de producto se extraigan automáticamente de stories.

**Requirements**:
- ✅ **MUST**: Story description parsing para decisiones
- ✅ **MUST**: Acceptance criteria extraction
- ✅ **MUST**: User story mapping a features
- ✅ **MUST**: Priority y deadline tracking
- ✅ **SHOULD**: Stakeholder identification
- ✅ **SHOULD**: Business impact analysis

**Acceptance Criteria**:
- Story descriptions > 200 caracteres indexadas
- Acceptance criteria extraídos con formato
- Priority levels mantenidos en metadata
- Stakeholders mencionados identificables

### 3.4 Integration Hub Core

#### 3.4.1 Webhook Management

**User Story**: Como DevOps engineer, quiero gestionar todos los webhooks desde un lugar centralizado.

**Requirements**:
- ✅ **MUST**: Centralized webhook endpoint management
- ✅ **MUST**: Webhook health monitoring y alerting
- ✅ **MUST**: Event queue con retry logic
- ✅ **MUST**: Event deduplication y ordering
- ✅ **SHOULD**: Webhook signature validation
- ✅ **SHOULD**: Event replay capability

**Acceptance Criteria**:
- Todos los webhooks monitoreados desde dashboard
- Failed webhooks reintentados automáticamente
- Event queue procesa 10K eventos/hora
- Event replay para recuperación de datos

#### 3.4.2 Data Processing Pipeline

**User Story**: Como system administrator, quiero que los datos se procesen eficientemente sin impactar performance.

**Requirements**:
- ✅ **MUST**: Background job processing con Oban
- ✅ **MUST**: Parallel processing por integración
- ✅ **MUST**: Error handling y dead letter queue
- ✅ **MUST**: Processing telemetry y metrics
- ✅ **SHOULD**: Adaptive batching por carga
- ✅ **SHOULD**: Resource limiting por organización

**Acceptance Criteria**:
- 1000 eventos procesados/minute sin degradación
- Error rate < 0.1% en procesamiento
- Dead letter queue para eventos fallidos
- Processing metrics disponibles por dashboard

---

## 4. Technical Architecture

### 4.1 Integration Hub Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   GitHub        │    │     Slack      │    │    Shortcut     │
│   Webhooks      │    │   Events API    │    │   Webhooks      │
└─────────┬───────┘    └─────────┬───────┘    └─────────┬───────┘
          │                      │                      │
          ▼                      ▼                      ▼
┌─────────────────────────────────────────────────────────────┐
│              Integration Hub (Phoenix)                     │
│  - Webhook endpoints (/webhooks/github, /webhooks/slack)   │
│  - Event validation y filtering                            │
│  - Queue management (Oban)                                  │
│  - Content extraction y classification                      │
└───────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│              PRD-001 Core API                               │
│  - Document creation                                       │
│  - Search indexing                                         │
│  - Multi-tenant storage                                    │
│  - Embedding generation                                     │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Event Processing Flow

```
1. External System → Integration Hub: Webhook Event
   GitHub: push event with commits
   Headers: X-Hub-Signature, Content-Type

2. Integration Hub: Event Validation
   ├─> Verify signature (HMAC SHA256)
   ├─> Check organization permissions
   ├─> Filter by repository/channel/workspace
   └─> Deduplicate by event ID

3. Integration Hub: Event Classification
   ├─> Parse event type (commit, PR, issue, message)
   ├─> Extract relevant content
   ├─> Apply classification rules
   └─> Generate metadata

4. Integration Hub: Queue Processing
   ├─> Background job creation (Oban)
   ├─> Priority assignment
   ├─> Retry configuration
   └─> Dead letter queue setup

5. Background Worker: Content Processing
   ├─> Content extraction y cleaning
   ├─> Entity recognition (people, dates, decisions)
   ├─> Relationship detection
   └─> Document creation

6. PRD-001 API: Document Storage
   ├─> Create document with metadata
   ├─> Generate embeddings
   ├─> Update search index
   └─> Create relationships

7. Integration Hub: Response
   ├─> Acknowledge webhook receipt
   ├─> Return processing status
   └─> Log completion metrics
```

### 4.3 Data Models

```elixir
# GitHub Event
defmodule Alejandria.Integrations.GitHub.Event do
  defstruct [:id, :type, :repository, :organization_id, :payload]

  def process(%{type: "push", payload: payload}) do
    commits = extract_commits(payload)
    Enum.each(commits, &process_commit/1)
  end

  def process(%{type: "pull_request", payload: payload}) do
    pr_data = extract_pr_data(payload)
    create_pr_document(pr_data)
  end

  defp extract_commits(%{"commits" => commits}) do
    Enum.map(commits, fn commit ->
      %{
        id: commit["id"],
        message: commit["message"],
        author: commit["author"]["name"],
        timestamp: commit["timestamp"],
        url: commit["url"],
        added: commit["added"],
        removed: commit["removed"],
        modified: commit["modified"]
      }
    end)
  end
end

# Slack Event
defmodule Alejandria.Integrations.Slack.Event do
  defstruct [:id, :type, :channel, :organization_id, :payload]

  def process(%{type: "message", payload: payload}) do
    message_data = extract_message_data(payload)
    
    if should_index_message?(message_data) do
      create_message_document(message_data)
    end
  end

  defp should_index_message?(%{text: text}) do
    # Filter out noise and index only substantive content
    text_length = String.length(text || "")
    
    text_length > 50 && 
      !String.contains?(text, "@channel") &&
      !contains_only_emojis?(text)
  end
end

# Shortcut Event
defmodule Alejandria.Integrations.Shortcut.Event do
  defstruct [:id, :type, :entity_type, :organization_id, :payload]

  def process(%{type: "story.created", payload: payload}) do
    story_data = extract_story_data(payload)
    create_story_document(story_data)
  end

  def process(%{type: "story.updated", payload: payload}) do
    story_data = extract_story_data(payload)
    update_story_document(story_data)
  end
end
```

---

## 5. Non-Functional Requirements

### 5.1 Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| Webhook processing latency | < 30s | Telemetry |
| Event queue processing | < 1s/event | Oban metrics |
| Content extraction | < 5s/document | Background job metrics |
| Sync latency (event → searchable) | < 2 minutes | End-to-end testing |
| Concurrent webhooks | 1000/minute | Load testing |

### 5.2 Reliability

- **Webhook success rate**: > 99.5%
- **Retry success rate**: > 95% after 3 retries
- **Data consistency**: 100% no data loss
- **Recovery time**: < 5 minutes for service restart

### 5.3 Security

- **Webhook validation**: HMAC signature verification
- **API access**: Organization-scoped tokens
- **Data encryption**: TLS 1.3 in transit
- **PII protection**: Automatic detection y redaction
- **Audit logging**: All webhook events logged

### 5.4 Scalability

| Dimension | MVP | Year 1 | Year 2 |
|-----------|-----|--------|--------|
| Organizations | 10 | 100 | 500 |
| Webhooks/hour | 10K | 100K | 1M |
| Events processed/day | 50K | 500K | 5M |
| Storage per org | 10GB | 100GB | 1TB |

---

## 6. Implementation Plan

### 6.1 Development Phases

#### Week 3: GitHub Integration
- **Day 15-17**: Webhook endpoints y validation
- **Day 18-19**: Content extraction para commits y PRs
- **Day 20-21**: Repository management y permissions

#### Week 4: Slack Integration
- **Day 22-24**: Slack API connection y events
- **Day 25-26**: Message classification y filtering
- **Day 27-28**: Privacy controls y PII handling

#### Week 5: Shortcut Integration
- **Day 29-31**: Shortcut API integration
- **Day 32-33**: Story processing y metadata
- **Day 34-35**: Epic relationships y workflows

#### Week 6: Hub Core & Testing
- **Day 36-38**: Centralized webhook management
- **Day 39-40**: Event processing pipeline
- **Day 41-42**: Integration testing y performance tuning

### 6.2 Technical Acceptance Criteria

#### GitHub Integration Tests
```elixir
test "GitHub webhook signature validation" do
  # Verify HMAC SHA256 signature validation
end

test "commit message extraction and indexing" do
  # Verify commit content processed correctly
end

test "PR description and comments processing" do
  # Verify PR content indexed with relationships
end
```

#### Slack Integration Tests
```elixir
test "Slack event API real-time processing" do
  # Verify real-time message ingestion
end

test "PII detection and redaction" do
  # Verify sensitive information redacted
end
```

#### Integration Hub Tests
```elixir
test "event deduplication and ordering" do
  # Verify no duplicate events processed
end

test "retry logic for failed webhooks" do
  # Verify automatic retry with backoff
end
```

### 6.3 Risk Mitigation

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| API rate limiting | Medium | High | Queue management, batching |
| Webhook delivery failures | High | Medium | Retry logic, dead letter queue |
| Content classification accuracy | Medium | Medium | ML model training, feedback loops |
| Privacy compliance violations | Critical | Low | PII detection, opt-out controls |

---

## 7. Success Criteria & Metrics

### 7.1 MVP Success (Week 6)

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| GitHub webhook success rate | > 99% | GitHub delivery logs |
| Slack message indexing latency | < 30s | Event timestamps |
| Shortcut story processing | < 15s | API response times |
| Event queue processing | < 1s/event | Oban metrics |
| Data freshness | < 5 minutes | Search availability tests |

### 7.2 Content Quality Validation

| Metric | Target | Test Method |
|--------|--------|-------------|
| Decision detection accuracy | > 80% | Manual validation sample |
| Technical discussion classification | > 85% | User feedback |
| PII redaction effectiveness | 100% | Automated scanning |
| Duplicate detection accuracy | > 95% | Content similarity analysis |

### 7.3 User Adoption Metrics

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| Organizations with integrations | > 70% | Usage analytics |
| Daily active integrations | > 90% | Health monitoring |
| User satisfaction with sync | NPS > 30 | User surveys |
| Reduction in manual documentation | > 50% | User self-reporting |

---

## 8. Out of Scope (PRD-003)

### Explicitly NOT in this PRD

- ❌ **Real-time collaboration features** → Fase 2
- ❌ **Advanced AI-powered content analysis** → Fase 2
- ❌ **Custom integration development framework** → Fase 3
- ❌ **Mobile push notifications** → Fase 3
- ❌ **Advanced analytics dashboards** → Fase 2
- ❌ **Multi-language content processing** → Fase 3

### Dependencies on Other PRDs

- **PRD-001 (Core Infrastructure)**: Depende de document storage, search, authentication
- **PRD-002 (MCP Server)**: Complementa con AI agent access to integrated content
- **PRD-004 (Performance)**: Optimiza processing pipeline y indexing

---

## 9. Dependencies & Risks

### 9.1 External Dependencies

| Dependency | Risk Level | Mitigation |
|------------|------------|------------|
| GitHub API stability | Medium | Webhook retry, API fallbacks |
| Slack API rate limits | Medium | Queue management, batching |
| Shortcut API changes | Low | Version management, adapter pattern |
| External service uptime | Medium | Health checks, alerting |

### 9.2 Integration Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Webhook delivery failures | High | Medium | Retry logic, monitoring |
| Content classification errors | Medium | Medium | Human-in-the-loop validation |
| API rate limiting | Medium | High | Intelligent queuing |
| Permission conflicts | Medium | Low | Clear permission model |

### 9.3 Business Risks

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| User resistance to auto-indexing | Medium | Medium | Opt-out options, granular controls |
| Privacy concerns from Slack integration | High | Low | Strong privacy controls, transparency |
| Integration complexity for users | Medium | Medium | Simple setup, clear documentation |
| Content overload in search results | Medium | Medium | Smart filtering, relevance ranking |

---

## 10. Testing Strategy

### 10.1 Unit Tests (Integrated in Development)

**Coverage Requirements**:
- > 90% coverage en integration modules
- 100% coverage en webhook validation
- 100% coverage en content extraction

**Key Test Scenarios**:
```elixir
# GitHub webhook processing
test "GitHub push event processing" do
  # Verify commit extraction and indexing
end

# Slack message classification
test "Slack message decision detection" do
  # Verify decision pattern recognition
end

# Event deduplication
test "duplicate event prevention" do
  # Verify no duplicate documents created
end
```

### 10.2 Integration Tests

**External API Integration**:
- GitHub webhook delivery y processing
- Slack Events API real-time connection
- Shortcut API webhook handling

**End-to-End Flow**:
- Event ingestion → processing → searchable content
- Error handling y recovery scenarios
- Multi-organization isolation

### 10.3 Load Testing

**Performance Targets**:
- 1000 concurrent webhook events
- 10K events/hour sustained processing
- 100 organizations simultaneous activity

**Test Scenarios**:
```bash
# GitHub webhook load test
./test/github_load_test.sh --events=1000 --duration=10m

# Slack message processing test
./test/slack_processing_test.sh --messages=5000 --concurrent=100
```

### 10.4 Security Testing

**Webhook Security**:
- HMAC signature validation
- API token security
- Injection attack prevention

**Data Privacy**:
- PII detection accuracy
- Permission boundary testing
- Data leakage prevention

---

## 11. Technical Specifications

### 11.1 Webhook Endpoint Configuration

```elixir
# lib/alejandria_web/integrations/github_webhook.ex
defmodule AlejandriaWeb.Integrations.GitHubWebhook do
  use AlejandriaWeb, :controller

  def webhook(conn, %{"x-hub-signature" => signature}) do
    body = conn |> read_body()
    
    if verify_signature(body, signature) do
      event = extract_event(conn.params)
      
      # Queue for background processing
      {:ok, _job} = Alejandria.Integrations.queue_github_event(event)
      
      conn
      |> put_status(:accepted)
      |> json(%{status: "queued"})
    else
      conn
      |> put_status(:unauthorized)
      |> json(%{error: "Invalid signature"})
    end
  end

  defp verify_signature(body, signature) do
    secret = get_webhook_secret()
    expected = :crypto.mac(:hmac, :sha256, secret, body) |> Base.encode16()
    "sha256=" <> expected == signature
  end
end
```

### 11.2 Content Extraction Pipeline

```elixir
defmodule Alejandria.Integrations.ContentExtractor do
  def extract_commit_content(%{message: message, author: author, url: url}) do
    %{
      title: extract_title(message),
      content: message,
      metadata: %{
        type: "commit",
        author: author,
        url: url,
        timestamp: DateTime.utc_now(),
        tags: extract_tags(message),
        decisions: extract_decisions(message)
      }
    }
  end

  def extract_slack_content(%{text: text, user: user, channel: channel, ts: timestamp}) do
    %{
      title: extract_title(text),
      content: text,
      metadata: %{
        type: "slack_message",
        author: user,
        channel: channel,
        timestamp: DateTime.from_unix!(String.to_float(timestamp)),
        tags: extract_tags(text),
        decisions: extract_decisions(text),
        action_items: extract_action_items(text)
      }
    }
  end

  defp extract_decisions(text) do
    decision_patterns = [
      ~r/(decided|agreed|concluded|finalized)/i,
      ~r/(let's|we will|going to)/i,
      ~r/(decision|choice|plan)/i
    ]

    Enum.filter(decision_patterns, fn pattern ->
      Regex.match?(pattern, text)
    end) |> length() > 0
  end
end
```

### 11.3 Event Queue Configuration

```elixir
# lib/alejandria/integrations/queue.ex
defmodule Alejandria.Integrations.Queue do
  use Oban.Worker, queue: :integrations, max_attempts: 5

  @impl true
  def perform(%Oban.Job{args: %{"type" => "github", "event" => event}}) do
    Alejandria.Integrations.GitHub.process_event(event)
  end

  def perform(%Oban.Job{args: %{"type" => "slack", "event" => event}}) do
    Alejandria.Integrations.Slack.process_event(event)
  end

  def perform(%Oban.Job{args: %{"type" => "shortcut", "event" => event}}) do
    Alejandria.Integrations.Shortcut.process_event(event)
  end

  @impl true
  def backoff(attempt) do
    # Exponential backoff: 30s, 2min, 8min, 32min, 2h
    [30, 120, 480, 1920, 7200]
    |> Enum.at(attempt - 1, 7200)
  end
end
```

---

## 12. Configuration Examples

### 12.1 GitHub Webhook Setup

```bash
# GitHub repository webhook configuration
curl -X POST \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/owner/repo/hooks \
  -d '{
    "name": "web",
    "active": true,
    "events": ["push", "pull_request", "issues"],
    "config": {
      "url": "https://api.alejandria.ai/webhooks/github",
      "content_type": "json",
      "secret": "your-webhook-secret"
    }
  }'
```

### 12.2 Slack App Configuration

```json
{
  "display_information": {
    "name": "Alejandria Integration",
    "description": "Index important Slack conversations in Alejandria"
  },
  "features": {
    "bot_user": {
      "display_name": "Alejandria Bot",
      "always_online": true
    },
    "event_subscriptions": {
      "request_url": "https://api.alejandria.ai/webhooks/slack",
      "bot_events": [
        "message.channels",
        "message.groups",
        "message.im"
      ]
    }
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "channels:history",
        "groups:history",
        "im:history",
        "users:read"
      ]
    }
  }
}
```

### 12.3 Shortcut Webhook Setup

```bash
# Shortcut webhook configuration
curl -X POST \
  -H "Content-Type: application/json" \
  -H "Shortcut-Token: $SHORTCUT_TOKEN" \
  https://api.app.shortcut.com/api/v3/webhooks \
  -d '{
    "name": "Alejandria Integration",
    "story_created": true,
    "story_updated": true,
    "story_deleted": true,
    "url": "https://api.alejandria.ai/webhooks/shortcut"
  }'
```

---

**Approval**:
- [ ] Product Lead
- [ ] Engineering Lead  
- [ ] DevOps Lead
- [ ] Security Lead

**Next Steps**:
1. Review con stakeholders (Week 3)
2. Setup development environment for integrations (Week 3)
3. Begin GitHub integration implementation (Week 3)
