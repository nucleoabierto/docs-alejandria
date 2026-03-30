# ADR Template

## ADR-[Número]: [Título]

**Status**: [Proposed/Accepted/Deprecated/Superseded]  
**Date**: [YYYY-MM-DD]  
**Authors**: [Nombres]  
**Reviewers**: [Nombres]  

## Context

[Descripción del contexto y problema que esta decisión aborda. ¿Por qué necesitamos tomar esta decisión ahora?]

## Decision

[Decisión clara y específica. ¿Qué vamos a hacer?]

## Consequences

[Impactos de esta decisión en el sistema, equipo, y stakeholders.]

## Alternatives Considered

[Otras opciones que consideramos y por qué no las elegimos.]

## Implementation

[Pasos para implementar esta decisión.]

## Related Decisions

[Referencias a otros ADRs relevantes.]

---

## Ejemplos de ADRs para Alejandria

### ADR-0001: Elixir/Phoenix como backend

**Status**: Accepted  
**Date**: 2026-03-20  
**Authors**: Equipo Técnico  

**Context**: Necesitamos alta concurrencia para webhooks en tiempo real y fault-tolerance.

**Decision**: Usar Elixir/Phoenix como framework backend.

**Consequences**:
- ✅ Concurrencia masiva con BEAM VM
- ✅ Fault-tolerance inherente
- ✅ Hot code swapping
- ❌ Curva de aprendizaje más alta
- ❌ Ecosistema más pequeño que Node.js/Python

**Alternatives Considered**:
- Node.js/Express: Single-threaded, menos robusto
- Python/Django: Lento para concurrencia alta
- Go/Gin: Más verbose, sin hot code swapping

### ADR-0002: Qdrant para búsqueda vectorial

**Status**: Accepted  
**Date**: 2026-03-21  
**Authors**: Equipo Técnico  

**Context**: Necesitamos búsqueda semántica con embeddings de alta performance.

**Decision**: Usar Qdrant como motor de búsqueda vectorial.

**Consequences**:
- ✅ Open source, self-hosted
- ✅ API limpia y documentada
- ✅ Buen performance con datasets grandes
- ❌ Menos maduro que Pinecone/Weaviate
- ❌ Requiere mantenimiento de cluster

**Alternatives Considered**:
- Pinecone: Cloud-only, vendor lock-in
- Weaviate: Más complejo de configurar
- PostgreSQL pgvector: Limitado para datasets grandes
