---
id: FORMAT-001
area: "operations"
type: "guide"
title: "Format Guide - Alejandria Documentation"
author: "leadership-team"
status: "active"
version: "2.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["INDEX-001"]
---

Guía de formato para todos los documentos de Alejandria. Estructura consistente para facilitar lectura y procesamiento por AI.

## Estructura de Documentos

Todo documento debe seguir esta estructura de 3 partes:

### 1. Metadata (Frontmatter)

**Estructura completa y requerida:**
```yaml
---
id: UNIQUE-ID-001
area: "área-conocimiento"
type: "tipo-contenido"
title: "Título del Documento"
author: "autor-equipo"
status: "estado"
version: "versión"
created: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
related_docs: ["DOC-001", "DOC-002"]
---
```

**Campos obligatorios:**
- `id`: Identificador único (formato: AREA-TIPO-NUMERO)
- `area`: Área de conocimiento (ver taxonomía)
- `type`: Tipo de contenido (ver taxonomía)
- `title`: Título completo del documento
- `status`: Estado actual del documento
- `last_updated`: Última actualización

**Campos recomendados:**
- `version`: Versión del documento
- `author`: Equipo o persona responsable
- `created`: Fecha de creación
- `related_docs`: Documentos relacionados (array vacío si no hay)

### 2. Summary (Resumen)
Texto conciso sin cabecera que explica QUÉ contiene el documento. Debe incluir:

**Estructura requerida:**
- **Propósito principal:** Qué problema resuelve o qué define
- **Contenido específico:** Secciones o componentes clave que incluye
- **Audiencia:** Para quién está destinado
- **Impacto:** Qué permite lograr o decidir

**Ejemplo de summary efectivo:**
> Documento estratégico completo que define la visión de Alejandria como solución al problema de pérdida de memoria institucional en equipos de software. Contiene análisis del problema actual, costo humano, visión de futuro, diferenciadores clave, principios guía y roadmap de implementación.

**Errores comunes a evitar:**
- ❌ No usar lenguaje abstracto o visionario en el summary
- ❌ No repetir el título del documento
- ❌ No incluir metáforas o lenguaje poético
- ✅ Ser específico sobre qué secciones contiene
- ✅ Mencionar el tipo de análisis o información presentada

### 3. Content (Contenido)
Contenido completo con estructura jerárquica usando `##` para secciones principales.

**Reglas importantes:**
- **NO usar divisores `---` dentro del contenido**
- Solo el separador principal entre summary y content
- Usar `##` para secciones principales
- Usar `###` para subsecciones
- Mantener estructura consistente

## Separadores

### Separador Principal (Obligatorio)
```markdown
---
```
Se usa UNA SOLA VEZ entre metadata/summary y el content principal.

**Ubicación exacta:**
- Después del summary (resumen)
- Antes del título principal `# Título`
- Es el único separador permitido en el documento

### Separadores Internos (Prohibidos)
```markdown
---
```
NO usar divisores dentro del contenido. En su lugar, usar títulos de sección `##`.

## Ejemplo Completo

```markdown
---
id: VISION-001
area: "executive"
type: "strategy"
title: "Vision - Alejandria"
author: "leadership-team"
status: "active"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["BUS-001", "PROD-001", "TECH-001"]
---

Resumen específico que explica el contenido del documento: propósito, secciones principales, audiencia e impacto.

---

# Título del Documento

## Sección Principal 1

Contenido de la sección...

### Subsección

Más contenido...

## Sección Principal 2

Más contenido...
```

## Best Practices

### Writing
- Frases cortas y directas
- Usar **negritas** para énfasis
- Emojis moderados para claridad visual
- Links internos usando ruta relativa `./documento.md`

### Structure
- Máximo 3 niveles de jerarquía (`#`, `##`, `###`)
- Secciones con nombres descriptivos
- Flow lógico de arriba hacia abajo

### Metadata
- Usar `area` y `type` para organización taxonómica
- Mantener `last_updated` con cada cambio
- Usar identificadores únicos consistentes
- IDs van en metadata, no en nombres de archivo
- `related_docs` con array vacío si no hay conexiones

## Estructura de Carpetas

### Organización Jerárquica
Los documentos se organizan físicamente siguiendo la taxonomía `area/type`:

```
.context/
├── templates/                     # Plantillas globales
│   ├── executive/
│   │   ├── strategy/
│   │   ├── definition/
│   │   ├── research/
│   │   └── record/
│   ├── product/
│   │   ├── definition/
│   │   ├── strategy/
│   │   ├── guide/
│   │   ├── research/
│   │   └── record/
│   ├── design/
│   │   ├── reference/
│   │   ├── guide/
│   │   ├── research/
│   │   └── record/
│   ├── engineering/
│   │   ├── definition/
│   │   ├── decision/
│   │   ├── guide/
│   │   ├── reference/
│   │   └── research/
│   └── operations/
│       ├── guide/
│       └── record/
├── executive/                     # Documentos reales
│   ├── strategy/
│   ├── definition/
│   ├── research/
│   └── record/
├── product/
│   ├── definition/
│   ├── strategy/
│   ├── guide/
│   ├── research/
│   └── record/
├── design/
│   ├── reference/
│   ├── guide/
│   ├── research/
│   └── record/
├── engineering/
│   ├── definition/
│   ├── decision/
│   ├── guide/
│   ├── reference/
│   └── research/
└── operations/
    ├── guide/
    └── record/
```

### Convenciones de Nombres

**Carpetas:**
- Nombres largos y descriptivos (ej: `engineering/` vs `tech/`)
- Minúsculas, sin espacios, sin guiones bajos

**Archivos:**
- Nombres naturales y descriptivos
- IDs van en metadata, no en filename
- Ejemplos: `vision.md`, `product-brief.md`, `elixir-phoenix.md`

### Documentos Multi-área
Para documentos que spans múltiples áreas:
1. Crear documento en el área "dueña" principal
2. Usar `related_docs` para referenciar aspectos en otras áreas
3. Cada aspecto tiene su propio documento específico

### Templates
Las plantillas viven en `/templates/` replicando la estructura:
- `templates/engineering/decision/adr-template.md`
- Se copian al crear nuevos documentos
- Mantienen consistencia de estructura

## Taxonomía de Campos

### `area` (Área de Conocimiento)
- `executive` - Liderazgo, visión, estrategia, negocio
- `product` - Producto, usuarios, features, roadmap
- `design` - UX, UI, componentes, visual system
- `engineering` - Arquitectura, código, decisiones técnicas
- `operations` - Deploy, monitoring, procesos, guías

### `type` (Tipo de Contenido)
- `strategy` - Visión, roadmap, OKRs, planificación
- `definition` - Specs, PRDs, arquitectura, requerimientos
- `decision` - ADRs, trade-offs, elecciones técnicas
- `guide` - Tutoriales, setup, procedimientos, how-to
- `reference` - APIs, componentes, templates, diccionarios
- `record` - Meeting notes, changelog, retrospectivas, logs
- `research` - Investigación, análisis, datos, findings

## Convenciones de ID

### Sistema Híbrido de IDs

**Tipos Establecidos (nomenclatura específica):**
- `decision` → **ADRs** (Architecture Decision Records)
  - Formato: `ADR-0001`, `ADR-0002`
  - Motivo: Estándar de industria en ingeniería

- `definition` → **PRDs** (Product Requirements Documents)  
  - Formato: `PRD-0001`, `PRD-0002`
  - Motivo: Estándar en product management

- `research` → **RFCs** (Request for Comments)
  - Formato: `RFC-0001`, `RFC-0002`
  - Motivo: Estándar para propuestas/discusiones técnicas

**Tipos Generales (estructura taxonómica):**
- `strategy` → `EXEC-STRAT-0001`, `PROD-STRAT-0001`
- `guide` → `ENG-GUIDE-0001`, `OPS-GUIDE-0001`
- `reference` → `DES-REF-0001`, `ENG-REF-0001`
- `record` → `EXEC-REC-0001`, `MEETING-REC-0001`

### Criterios de Decisión

**Usar nomenclatura específica si:**
- Existe estándar de industria reconocido
- La comunidad técnica lo usa ampliamente
- Facilita integración con herramientas externas
- El tipo de documento es universalmente reconocido

**Usar estructura taxonómica si:**
- Son documentos específicos de Alejandria
- No existe estándar previo
- El documento es interno/organizacional
- Se necesita flexibilidad en la estructura

### Ejemplos Completos

**Nomenclatura Específica:**
- `ADR-0001` - Architecture Decision Record
- `PRD-0001` - Product Requirements Document  
- `RFC-0001` - Request for Comments

**Estructura Taxonómica:**
- `EXEC-STRAT-0001` - Executive strategy document
- `PROD-DEF-0001` - Product definition document
- `ENG-GUIDE-0001` - Engineering guide
- `DES-REF-0001` - Design reference guide
- `OPS-GUIDE-0001` - Operations guide
- `EXEC-REC-0001` - Executive record

### Numeración
- Empezar en 0001 para cada tipo de documento
- Incrementar secuencialmente por tipo
- Soporta hasta 9999 documentos por tipo
- No reusar IDs eliminados
- IDs van en metadata, no en filename

**Nombres de Archivo:**
- Usar nombres naturales y descriptivos
- Ejemplo: `elixir-phoenix.md` con `id: ENG-DEC-0001`
- Facilita lectura y navegación humana

## Para AI Agents

Esta estructura está optimizada para:
- **RAG**: Summary proporciona contexto rápido
- **Parsing**: Separadores claros para procesamiento
- **Retrieval**: `area` y `type` específicos para filtrado preciso
- **Context**: Metadata funcional para propósito y organización
- **Navigation**: `related_docs` para descubrimiento de contenido relacionado
- **Lifecycle**: `status` y `version` para gestión de contenido
- **Queries compuestas**: Filtrar por área, tipo o ambos

## Validación

Antes de finalizar un documento, verifica:
1. ✅ Frontmatter completo con todos campos requeridos
2. ✅ ID único siguiendo formato AREA-TIPO-NUMERO
3. ✅ `area` y `type` definidos según taxonomía
4. ✅ Summary conciso sin cabecera
5. ✅ Separador principal único
6. ✅ Sin divisores internos `---`
7. ✅ Estructura de títulos consistente
8. ✅ Links relativos funcionales
9. ✅ Metadata funcional y sin redundancia
10. ✅ Documentos relacionados vinculados (o array vacío)

## Convenciones de ID

**Formato:** AREA-TIPO-NUMERO

**Ejemplos:**
- `EXEC-STRAT-001` - Executive strategy document
- `PROD-DEF-001` - Product definition document
- `ENG-DEC-001` - Engineering decision record
- `DES-REF-001` - Design reference guide
- `OPS-GUIDE-001` - Operations guide
- `EXEC-RES-001` - Executive research finding

**Numeración:**
- Empezar en 001 para cada combinación area-tipo
- Incrementar secuencialmente por combinación
- No reusar IDs eliminados
- IDs van en metadata, no en filename

**Nombres de Archivo:**
- Usar nombres naturales y descriptivos
- Ejemplo: `elixir-phoenix.md` con `id: ENG-DEC-001`
- Facilita lectura y navegación humana
