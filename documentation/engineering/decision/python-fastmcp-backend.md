---
id: ADR-0002
area: "engineering"
type: "decision"
title: "Python + FastMCP como Backend Framework"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "PROD-STRAT-0001", "ADR-0001", "ADR-0006"]
---

Registro de decisión arquitectónica que justifica la selección de Python con FastMCP como framework backend para Alejandria. Contiene contexto del problema de integración MCP, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0002: Python + FastMCP como Backend Framework

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que construir un servidor MCP que permita a los agentes de IA crear, buscar y editar documentos en Alejandria es esencial para nuestro MVP. El protocolo MCP (Model Context Protocol) es el estándar emergente para que los agentes de IA accedan a herramientas y datos externos, con más de 97M de descargas mensuales.

El desafío técnico es que la implementación directa con el SDK oficial de MCP requiere manejar manualmente transporte, autenticación, lifecycle management, schema generation y validación. Esto aumenta la complejidad del desarrollo y el tiempo de iteración, lo cual es crítico para un MVP de 6 semanas donde cada día cuenta.

Entendemos que necesitamos un ecosistema maduro para ML/AI (embeddings, chunking, procesamiento de texto) y una forma simplificada de integración con MCP que permita desarrollo rápido sin sacrificar funcionalidad.

## Decision

Usar Python como lenguaje backend con el framework FastMCP de Prefect para la integración MCP.

**Especificación técnica:**
- **Lenguaje**: Python 3.11+
- **Framework MCP**: FastMCP (de Prefect)
- **Razón**: Ecosistema maduro para ML/AI + simplificación de integración MCP

FastMCP proporciona:
- Decoradores @mcp.tool para definir endpoints con schema automático (piensa en esto como decoradores que hacen el trabajo pesado por ti)
- Manejo automático de transporte, autenticación y lifecycle
- Validación y documentación automática de endpoints
- Integración nativa con Pydantic para schema validation
- Más simple y Pythonic que el SDK oficial de MCP

## Consequences

### Positivas

✅ **Ecosistema maduro para ML/AI**: Python tiene las mejores bibliotecas para embeddings (sentence-transformers), NLP (spaCy, nltk), y procesamiento de texto, lo cual es esencial para Alejandria.

✅ **Desarrollo rápido con FastMCP**: Los decoradores @mcp.tool reducen el código de integración MCP de ~100 líneas a ~10 líneas por endpoint, acelerando el desarrollo del MVP y permitiéndote enfocarte en lo que realmente importa.

✅ **Schema automático y validación**: FastMCP genera automáticamente el schema JSON para cada endpoint y valida los inputs/outputs, reduciendo bugs y mejorando la experiencia para los agentes de IA.

✅ **Documentación automática**: Cada endpoint se documenta automáticamente basándose en los tipos y docstrings, facilitando el descubrimiento para los agentes.

✅ **Manejo simplificado de lifecycle**: FastMCP maneja automáticamente conexión, reconnection, shutdown y cleanup, permitiendo al equipo enfocarse en lógica de negocio en lugar de infraestructura.

✅ **Curva de aprendizaje baja**: El equipo ya conoce Python, y FastMCP es intuitivo para desarrolladores Python, reduciendo el tiempo de onboarding.

### Negativas

❌ **Performance de runtime**: Python es más lento que lenguajes compilados (Go, Rust), pero esto no es crítico para el MVP donde el bottleneck está en la búsqueda vectorial (Qdrant) y generación de embeddings.

❌ **Dependencia de FastMCP**: FastMCP es mantenido por Prefect, no por el equipo oficial de MCP. Si Prefect discontinúa el proyecto, necesitamos migrar al SDK oficial. Sin embargo, Prefect tiene track record sólido y el proyecto es open source.

❌ **Global Interpreter Lock (GIL)**: Python tiene limitaciones para concurrencia verdadera, pero esto no es un problema para el MVP donde las operaciones son I/O bound (llamadas a Qdrant, PostgreSQL). Para escalabilidad futura, podemos usar async/await o multiprocessing.

❌ **Memory overhead**: Python consume más memoria que Go/Rust, pero esto es aceptable para el MVP con Qdrant self-hosted manejando ~10M embeddings.

## Alternatives Considered

### SDK Oficial de MCP (TypeScript/Node.js)

**Por qué no la elegimos:**
- Requiere manejo manual de transporte, autenticación y lifecycle
- Mayor complejidad de desarrollo (~2-3x más código por endpoint)
- Ecosistema ML/AI menos maduro que Python
- Schema y validación requieren implementación manual

**Cuándo podría ser mejor:**
- Si el equipo tuviera más experiencia en TypeScript
- Si necesitáramos performance de runtime crítico
- Si el proyecto fuera más grande que un MVP

### Elixir/Phoenix con SDK MCP

**Por qué no la elegimos:**
- Curva de aprendizaje más alta para el equipo
- Ecosistema ML/AI menos maduro (menos bibliotecas para embeddings, NLP)
- SDK oficial de MCP no tiene soporte nativo para Elixir
- FastMCP no existe para Elixir

**Cuándo podría ser mejor:**
- Si necesitáramos concurrencia masiva (millones de conexiones concurrentes)
- Si el proyecto requiriera fault-tolerance crítico
- Si el equipo tuviera experiencia en Elixir

### Go con SDK MCP

**Por qué no la elegimos:**
- Ecosistema ML/AI menos maduro que Python
- SDK oficial de MCP no tiene soporte nativo para Go
- Mayor verbosidad en código (más líneas por endpoint)
- FastMCP no existe para Go

**Cuándo podría ser mejor:**
- Si necesitáramos performance de runtime crítico
- Si el proyecto fuera a escala enterprise con cientos de miles de usuarios
- Si el equipo tuviera experiencia en Go

## Implementation

### Paso 1: Setup del proyecto
```bash
# Crear entorno virtual
python -m venv venv
source venv/bin/activate

# Instalar dependencias
pip install fastmcp pydantic qdrant-client psycopg2-binary sentence-transformers
```

### Paso 2: Definir endpoints MCP con FastMCP
```python
from fastmcp import FastMCP
from pydantic import BaseModel

mcp = FastMCP("alejandria")

@mcp.tool()
def create_document(
    title: str,
    content: str,
    organization_id: str,
    acl: dict
) -> dict:
    """Crear nuevo documento en Alejandria."""
    # Implementación: guardar en PostgreSQL, generar embeddings, indexar en Qdrant
    pass

@mcp.tool()
def search_documents(
    query: str,
    organization_id: str,
    limit: int = 10
) -> list[dict]:
    """Buscar documentos usando búsqueda semántica."""
    # Implementación: embedding de query, búsqueda en Qdrant, filtrar por ACL
    pass
```

### Paso 3: Configurar transporte y autenticación
FastMCP maneja automáticamente transporte (stdio o SSE) y autenticación (API keys). Solo necesitamos configurar:

```python
if __name__ == "__main__":
    mcp.run(transport="stdio")
```

### Paso 4: Testing
- Unit tests para cada endpoint MCP
- Integration tests con Qdrant y PostgreSQL
- E2E tests con Claude Desktop/CLI

### Paso 5: Deployment
- Docker container con Python 3.11+
- Environment variables para configuración
- Health checks para monitoreo

## Related Decisions

- **ADR-0004**: Qdrant como vector database (complementa esta decisión)
- **ADR-0003**: PostgreSQL como base de datos relacional (complementa esta decisión)
- **ADR-0005**: sentence-transformers para embeddings (complementa esta decisión)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Dependencia de FastMCP:**
- Mantener el código modular para facilitar migración al SDK oficial si es necesario
- Contribuir al proyecto FastMCP open source
- Monitorear el estado del proyecto y tener plan de contingencia

**Performance de Python:**
- Usar async/await para operaciones I/O bound
- Profilear código para identificar bottlenecks
- Considerar Cython o Rust para secciones críticas de performance si es necesario

**Escalabilidad:**
- Diseñar arquitectura para permitir horizontal scaling
- Usar connection pooling para PostgreSQL y Qdrant
- Implementar caching para embeddings y búsquedas frecuentes
