---
id: ADR-0005
area: "engineering"
type: "decision"
title: "sentence-transformers para Embeddings"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0002", "ADR-0004"]
---

Registro de decisión arquitectónica que justifica la selección de sentence-transformers (all-MiniLM-L6-v2) como modelo de embeddings para Alejandria. Contiene contexto del problema de búsqueda semántica multi-dominio, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0005: sentence-transformers para Embeddings

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que necesitamos un modelo de embeddings que genere representaciones vectoriales de texto para búsqueda semántica en Alejandria. El modelo debe entender lenguaje de negocio, técnico y operacional en español e inglés, ya que los documentos de las organizaciones pueden contener terminología de múltiples dominios.

El desafío técnico es encontrar un modelo que balancee:
- **Performance de búsqueda**: Capacidad de encontrar documentos semánticamente relevantes
- **Performance de inferencia**: Velocidad de generación de embeddings (latencia)
- **Tamaño del modelo**: Requerimientos de memoria y CPU
- **Costo**: Preferiblemente open source y self-hosted para evitar dependencias de APIs externas
- **Multi-dominio**: Entendimiento de lenguaje técnico, de negocio y operacional

Entendemos que necesitamos:
- Modelo open source para self-hosting (control total y costos predecibles)
- Buen performance en español e inglés
- Razonablemente rápido para generación de embeddings en tiempo real
- Capacidad de migrar a modelos más grandes cuando sea necesario
- Integración sencilla con Python (ecosistema sentence-transformers)

## Decision

Usar sentence-transformers (all-MiniLM-L6-v2) como modelo de embeddings inicial para el MVP, con plan de migración a all-mpnet-base-v2 cuando sea necesario.

**Especificación técnica:**
- **Framework**: sentence-transformers (Hugging Face transformers)
- **Modelo inicial**: all-MiniLM-L6-v2 (23MB, 384 dimensions)
- **Modelo futuro**: all-mpnet-base-v2 (420MB, 768 dimensions)
- **Idiomas**: Español e inglés (multi-lingual)
- **Embedding dimension**: 384 (inicial), 768 (futuro)
- **Distance metric**: Cosine similarity

**Razón para all-MiniLM-L6-v2 inicial:**
- Modelo ligero (23MB) - puede ejecutarse en CPU modesto
- Rápido (~10ms por embedding en CPU)
- Buen performance en español e inglés
- Suficiente para MVP con ~10k-100k documentos
- Migración sencilla a modelo más grande cuando sea necesario

sentence-transformers proporciona:
- API simple y Pythonica para generación de embeddings (piensa en esto como una interfaz intuitiva que hace el trabajo complejo por ti)
- Modelos pre-entrenados multi-linguales
- Soporte para batch processing
- Cache de embeddings para contenido sin cambios
- Integración con Hugging Face ecosystem

## Consequences

### Positivas

✅ **Open source y self-hosted**: No dependemos de APIs externas (OpenAI, Cohere), lo cual reduce costos y elimina vendor lock-in. Tenemos control total del modelo y podemos tunearlo si es necesario.

✅ **Costo cero**: Self-hosting significa costo cero por embedding, a diferencia de APIs que cobran por token. Esto es crítico para escalabilidad (millones de embeddings).

✅ **Multi-lingual**: all-MiniLM-L6-v2 está entrenado en múltiples idiomas incluyendo español e inglés, lo cual es esencial para organizaciones latinoamericanas.

✅ **Performance aceptable para MVP**: Con 384 dimensions, el modelo es rápido (~10ms por embedding en CPU) y suficientemente preciso para búsqueda semántica en MVP.

✅ **Migración sencilla**: Podemos migrar a all-mpnet-base-v2 (420MB, 768 dimensions) cuando necesitemos más precisión, re-generando embeddings en background.

✅ **Ecosistema maduro**: sentence-transformers tiene buena documentación, comunidad activa y ejemplos de uso, reduciendo curva de aprendizaje.

✅ **Batch processing**: Podemos generar embeddings en batches para eficiencia, reduciendo latencia cuando indexamos múltiples documentos.

### Negativas

❌ **Menos preciso que modelos grandes**: all-MiniLM-L6-v2 es menos preciso que modelos más grandes (all-mpnet-base-v2, OpenAI text-embedding-3). Esto es aceptable para el MVP y mitigamos con migración futura.

❌ **Requiere CPU para inferencia**: A diferencia de APIs cloud, necesitamos CPU local para generar embeddings. Esto añade overhead de infraestructura pero es aceptable para el MVP.

❌ **No especializado en dominio técnico**: El modelo es general-purpose, no fine-tuned para lenguaje técnico específico. Mitigamos esto con fine-tuning futuro si es necesario.

❌ **Re-generación de embeddings al cambiar modelo**: Si migramos a all-mpnet-base-v2, necesitamos re-generar todos los embeddings. Esto es un proceso de background que puede tomar horas, pero es manejable.

## Alternatives Considered

### OpenAI text-embedding-3 (small/large)

**Por qué no la elegimos:**
- API cloud-only, no self-hosted
- Costo por token (no escalable a millones de embeddings)
- Vendor lock-in completo
- Depende de conectividad a internet
- Menos control sobre modelo y datos

**Cuándo podría ser mejor:**
- Si el budget no fuera una preocupación
- Si no tuviéramos capacidad para infraestructura
- Si necesitáramos el mejor performance posible desde día 1

### Cohere embed models

**Por qué no la elegimos:**
- API cloud-only, no self-hosted
- Costo por token
- Vendor lock-in
- Menos adopción que OpenAI

**Cuándo podría ser mejor:**
- Si necesitáramos features específicos de Cohere
- Si el equipo tuviera experiencia previa con Cohere

### OpenAI text-embedding-ada-002 (legacy)

**Por qué no la elegimos:**
- Deprecated por OpenAI
- Menos preciso que modelos nuevos
- API cloud-only
- Costo por token

**Cuándo podría ser mejor:**
- Nunca - modelo deprecated

### Custom fine-tuned model

**Por qué no la elegimos:**
- Requiere dataset grande para fine-tuning (no tenemos en MVP)
- Complejidad de desarrollo y mantenimiento
- Requiere expertise en ML
- Overkill para MVP

**Cuándo podría ser mejor:**
- Si tuviéramos dataset grande específico del dominio
- Si el modelo general-purpose no fuera suficiente
- Si el equipo tuviera expertise en ML fine-tuning

## Implementation

### Paso 1: Instalación
```bash
pip install sentence-transformers
```

### Paso 2: Cargar modelo y generar embeddings
```python
from sentence_transformers import SentenceTransformer

# Cargar modelo (23MB, se descarga automáticamente en primer uso)
model = SentenceTransformer('all-MiniLM-L6-v2')

# Generar embedding para un texto
text = "Este es un documento técnico sobre arquitectura de software"
embedding = model.encode(text)

# embedding es un array numpy de 384 dimensions
print(embedding.shape)  # (384,)
```

### Paso 3: Batch processing para eficiencia
```python
# Generar embeddings para múltiples documentos
documents = [
    "Documento 1 sobre arquitectura",
    "Documento 2 sobre producto",
    "Documento 3 sobre operaciones"
]

# Batch encoding (más eficiente que uno por uno)
embeddings = model.encode(documents, batch_size=32, show_progress_bar=True)

# embeddings es un array (3, 384)
print(embeddings.shape)
```

### Paso 4: Cache de embeddings
```python
import hashlib
import pickle
from pathlib import Path

def get_embedding_with_cache(text, model, cache_dir=".cache"):
    # Generar hash del texto como key de cache
    text_hash = hashlib.md5(text.encode()).hexdigest()
    cache_path = Path(cache_dir) / f"{text_hash}.pkl"
    
    # Return cache si existe
    if cache_path.exists():
        with open(cache_path, 'rb') as f:
            return pickle.load(f)
    
    # Generar embedding si no existe
    embedding = model.encode(text)
    
    # Guardar en cache
    cache_path.parent.mkdir(parents=True, exist_ok=True)
    with open(cache_path, 'wb') as f:
        pickle.dump(embedding, f)
    
    return embedding
```

### Paso 5: Migración a all-mpnet-base-v2
```python
# Cuando necesitemos más precisión
model_large = SentenceTransformer('all-mpnet-base-v2')  # 420MB, 768 dimensions

# Re-generar embeddings en background
for document in documents:
    new_embedding = model_large.encode(document.content)
    # Actualizar en Qdrant
    qdrant_client.upsert(
        collection_name=f"org_{document.organization_id}",
        points=[{
            "id": document.id,
            "vector": new_embedding,
            "payload": document.payload
        }]
    )
```

### Paso 6: Testing
- Unit tests para generación de embeddings
- Integration tests con Qdrant
- Performance tests para latencia de inferencia
- Accuracy tests para calidad de búsqueda semántica

### Paso 7: Deployment
- Descargar modelo en build time (no en runtime)
- Cargar modelo en memoria al iniciar servidor
- Monitorear uso de CPU y memoria
- Plan de migración a modelo más grande documentado

## Related Decisions

- **ADR-0002**: Python + FastMCP como backend framework (sentence-transformers es Python-native)
- **ADR-0004**: Qdrant como vector database (embeddings se almacenan en Qdrant)
- **ADR-0003**: PostgreSQL como base de datos relacional (documentos originales en PostgreSQL)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Precisión del modelo:**
- Monitorear calidad de búsqueda semántica con feedback de usuarios
- Tener métricas de relevancia (click-through rate, satisfaction)
- Plan de migración a all-mpnet-base-v2 documentado
- Considerar fine-tuning si el modelo general-purpose no es suficiente

**CPU para inferencia:**
- Usar batch processing para maximizar throughput
- Considerar GPU para aceleración si es necesario
- Cache de embeddings para contenido sin cambios
- Monitorear latencia y planificar upgrade de hardware

**Re-generación de embeddings:**
- Implementar re-generación en background sin downtime
- Usar versioning de embeddings para rollback si es necesario
- Comunicar a usuarios sobre mejora de precisión
- Planificar re-generación durante horas de bajo tráfico

**Especialización en dominio:**
- Evaluar calidad de búsqueda en lenguaje técnico específico
- Considerar fine-tuning con dataset del dominio si es necesario
- Usar domain-specific vocabulary en preprocessing
- Ajustar chunking strategy para mejor representación semántica
