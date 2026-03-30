---
id: PROD-RES-0001
area: "product"
type: "research"
title: "Product Research Analysis - [Nombre del Proyecto]"
author: "product-team"
status: "active"
version: "1.0"
created: "YYYY-MM-DD"
last_updated: "YYYY-MM-DD"
related_docs: []
---

Documento de investigación de producto que analiza el mercado, usuarios, competencia y oportunidades para [Nombre del Proyecto]. Contiene análisis de problemas de usuarios, investigación de mercado, validación de propuesta de valor y métricas de éxito para fundamentar decisiones estratégicas de producto.

---

# Product Research Analysis - [Nombre del Proyecto]

## Qué es [Nombre del Proyecto]

Imagina tener [metáfora principal del sistema]. Así es como funciona [Nombre del Proyecto]: es la [descripción simple y memorable], diseñada para que tanto humanos como [agentes de IA/usuarios] puedan acceder al [valor principal] cuando lo necesitan.

**[Nombre del Proyecto]** es un [tipo de sistema] que combina [característica principal 1] con [característica principal 2] de herramientas que ya usas.

### El Problema que Resolvemos

Sabemos lo frustrante que es sentir que tu [equipo/organización] está [perdiendo/sufriendo] [problema principal]. La [información/recursos/valor] que necesitas está por todas partes, pero al mismo tiempo en ninguna parte:

- 📝 **En [Herramienta 1]** - [Problema específico 1]
- 🔧 **En [Herramienta 2]** - [Problema específico 2]
- 📋 **En [Herramienta 3]** - [Problema específico 3]
- 🎯 **En [Herramienta 4]** - [Problema específico 4]

### La Solución que Ofrecemos

**🧠 [Beneficio principal 1]**
- [Capacidad específica 1]
- [Capacidad específica 2]
- [Capacidad específica 3]

**🔗 [Beneficio principal 2]**
- [Capacidad específica 4]
- [Capacidad específica 5]
- [Capacidad específica 6]

## Stack Tecnológico

### [Capa 1] - El Corazón del Sistema

**[Tecnología Principal] [Versión]** - Elegimos esta tecnología porque nos permite:
- [Ventaja técnica 1]
- [Ventaja técnica 2]
- [Ventaja técnica 3]
- [Ventaja técnica 4]

**[Base de Datos]** - Donde guardamos:
- [Tipo de dato 1]
- [Tipo de dato 2]
- [Tipo de dato 3]
- [Tipo de dato 4]

**[Motor/Biblioteca]** - Nuestro [propósito del motor]:
- [Función 1]
- [Función 2]
- [Función 3]

**[Sistema de Colas/Procesamiento]** - Para [propósito]:
- [Proceso 1]
- [Proceso 2]
- [Proceso 3]

### [Capa 2] - La Magia Detrás de [Función Principal]

**[Modelo/Algoritmo]** - Nuestro [tipo de modelo]:
- [Característica 1]
- [Característica 2]
- [Característica 3]

**[Estrategia de Procesamiento]** - Dividimos [recursos] inteligentemente:
- [Parámetro 1]
- [Parámetro 2]
- [Parámetro 3]

**[Enfoque Híbrido]** - Combinamos lo mejor de ambos mundos:
- [Método 1] ([propósito])
- [Método 2] ([propósito])
- [Método de combinación] para mezclar resultados

### [Capa 3] - Todo Funcionando Junto

**[Herramienta de Infraestructura 1]** - Para [propósito]
**[Herramienta de Infraestructura 2]** - [propósito]
**[Herramienta de Infraestructura 3]** - [propósito]

## Arquitectura de Alto Nivel

La arquitectura de [Nombre del Proyecto] está diseñada pensando en la experiencia real de los [usuarios/equipos]. Todo fluye de manera natural, desde que [evento inicial] hasta que [resultado final].

```
┌─────────────────────────────────────────────────┐
│              [Capa de Interface]                 │
│  - [Endpoint 1]                                 │
│  - [Endpoint 2]                                 │
│  - [Endpoint 3]                                 │
└─────────────┬───────────────────────────────────┘
              │ [Protocolo]
              ▼
┌─────────────────────────────────────────────────┐
│         [Backend Principal] ([Lenguaje])       │
│  ┌──────────────┐  ┌──────────────┐            │
│  │  [Módulo 1]  │  │   [Módulo 2] │            │
│  │  Context     │  │   Engine     │            │
│  └──────┬───────┘  └──────┬───────┘            │
│         │                  │                     │
│  ┌──────┴───────┐  ┌──────┴───────┐            │
│  │ [Módulo 3]   │  │  [Módulo 4]  │            │
│  │   Context    │  │   Context    │            │
│  └──────────────┘  └──────────────┘            │
└─────────┬──────────────────┬───────────────────┘
          │                  │
    ┌─────┴─────┐     ┌─────┴─────┐
    ▼           ▼     ▼           ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│[Dato 1] │ │[Dato 2] │ │[Dato 3] │
│([Tipo]) │ │([Tipo]) │ │ ([Tipo]) │
└─────────┘ └─────────┘ └─────────┘
```

## Decisiones Técnicas Clave

### ADR-0001: [Tecnología Principal] como [Propósito]
**Razón**: [Razón principal]
**Trade-off**: [Compensación]

### ADR-0002: [Tecnología Secundaria] para [Función]
**Razón**: [Razón principal]
**Trade-off**: [Compensación]

### [Decisión 3]: [Tecnología] como [Propósito]
**Razón**: [Razón principal]
**Trade-off**: [Compensación]

## Data Model

### [Base de Datos Principal]

**Tablas principales:**

```sql
-- [Tabla 1]
[nombre_tabla](
  [campo_1], 
  [campo_2], 
  [campo_3],
  [campo_4],
  [campo_5],
  created_at,
  updated_at
)

-- [Tabla 2]
[nombre_tabla](
  [campo_1],
  [campo_2],
  [campo_3],
  [campo_4],
  created_at
)

-- [Tabla 3]
[nombre_tabla](
  [campo_1],
  [campo_2],
  [campo_3],
  [campo_4],
  [campo_5],
  created_at
)
```

### [Base de Datos Secundaria]

**Collection: [nombre]**
```yaml
vectors:
  size: [dimensiones]
  distance: [tipo]
payload:
  [campo_1]: [tipo]
  [campo_2]: [tipo]
  [campo_3]: [tipo]
  [campo_4]: [tipo]
```

## API Endpoints

### [Categoría 1]
- `GET /api/v1/[recurso]` - [Descripción]
- `POST /api/v1/[recurso]` - [Descripción]
- `GET /api/v1/[recurso]/:id` - [Descripción]
- `PUT /api/v1/[recurso]/:id` - [Descripción]
- `DELETE /api/v1/[recurso]/:id` - [Descripción]

### [Categoría 2]
- `GET /api/v1/[acción]?[parámetros]` - [Descripción]

### [Categoría 3]
- `POST /api/v1/[webhook]` - [Descripción]

### [Protocolo Especial]
- `[función_1]` - [Descripción]
- `[función_2]` - [Descripción]
- `[función_3]` - [Descripción]
- `[función_4]` - [Descripción]

### [Herramienta CLI]
- `[comando 1]` - [Descripción]
- `[comando 2]` - [Descripción]
- `[comando 3]` - [Descripción]
- `[comando 4]` - [Descripción]

## Flujos Principales

### 1. [Flujo Principal 1]

```
[Evento inicial]
  ↓
[Proceso 1]
  ↓
[Proceso 2]
  ↓
[Proceso 3]
  ↓
[Proceso 4]
  ↓
[Resultado final]
```

### 2. [Flujo Principal 2]

```
[Evento inicial]
  ↓
[Proceso 1]
  ↓
Ejecución paralela:
  ├─> [Proceso 2A]
  └─> [Proceso 2B]
  ↓
[Proceso 3]
  ↓
[Resultado final]
```

### 3. [Flujo Principal 3]

```
[Actor externo]
  ↓
[Protocolo]
  ↓
[Manejador interno]
  ↓
[Ejecutar operación]
  ↓
[Respuesta estructurada]
```

### 4. [Flujo Principal 4]

```
[Usuario ejecuta comando]
  ↓
[Tarea CLI]
  ↓
[Leer/Procesar datos]
  ↓
[Llamar API interna]
  ↓
[Retornar éxito/error]
```

## Módulos Principales ([Lenguaje])

### Contexts

```[lenguaje]
[NombreProyecto].[Módulo1]
  - [función_1]/[aridad]
  - [función_2]/[aridad]
  - [función_3]/[aridad]
  - [función_4]/[aridad]

[NombreProyecto].[Módulo2]
  - [función_1]/[aridad]
  - [función_2]/[aridad]
  - [función_3]/[aridad]

[NombreProyecto].[Módulo3]
  - [proceso_1]/[aridad]

[NombreProyecto].[Módulo4]
  - [función_1]/[aridad] - [descripción]
  - [función_2]/[aridad]
  - [función_3]/[aridad]

[NombreProyecto].[Módulo5]
  - [función_1]/[aridad]
  - [función_2]/[aridad]
  - [función_3]/[aridad]
  - [función_4]/[aridad]
```

### Workers ([Sistema de Colas])

```[lenguaje]
[NombreProyecto].Workers.[Worker1]
  - [Descripción del worker]
  - Prioridad: [alta/baja]
  - Queue: :[nombre_queue]

[NombreProyecto].Workers.[Worker2]
  - [Descripción del worker]
  - Prioridad: [alta/baja]
  - Queue: :[nombre_queue]
```

## Patrones de Código

### 1. [Patrón 1]

```[lenguaje]
defmodule [NombreProyecto].[Módulo] do
  alias [NombreProyecto].[Repo]
  alias [NombreProyecto].[Módulo].[Entidad]

  def [función](attrs) do
    %Entidad{}
    |> Entidad.changeset(attrs)
    |> Repo.insert()
  end
end
```

### 2. [Patrón 2]

```[lenguaje]
def [proceso](recurso) do
  recurso
  |> [paso_1]()
  |> [paso_2]()
  |> [paso_3]()
  |> [paso_4]()
end
```

### 3. [Patrón 3]

```[lenguaje]
children = [
  [Componente1],
  [Componente2],
  {[Configuración]},
  [Componente3]
]

Supervisor.start_link(children, strategy: :one_for_one)
```

## Convenciones

### Naming
- **Modules**: [Formato de módulos]
- **Functions**: [Formato de funciones]
- **Variables**: [Formato de variables]
- **Atoms**: [Formato de atoms]

### Error Handling
```[lenguaje]
# Pattern matching
case [Módulo].[función](attrs) do
  {:ok, resultado} -> # success
  {:error, changeset} -> # error
end

# With clause
with {:ok, recurso} <- [Módulo].[función](id),
     {:ok, proceso} <- [Módulo2].[función](recurso) do
  # success
else
  {:error, razón} -> # error
end
```

### Testing
```[lenguaje]
# Unit test
test "[descripción]" do
  attrs = %{[clave]: [valor]}
  assert {:ok, %Entidad{}} = [Módulo].[función](attrs)
end

# Integration test
test "[descripción]" do
  # Setup
  create_test_data()
  
  # Execute
  results = [Módulo].[función](query, org_id)
  
  # Assert
  assert length(results) > 0
end
```

## Performance Considerations

### Caching
- **[Sistema 1]**: [Propósito del cache] (TTL [tiempo])
- **[Sistema 2]**: [Propósito del cache]
- **[Sistema 3]**: [Propósito del cache]

### Indexing
```sql
-- [Base de datos] indexes
CREATE INDEX idx_[tabla]_[campo] ON [tabla]([campo]);
CREATE INDEX idx_[tabla]_[campo] ON [tabla] USING [tipo](to_tsvector('[idioma]', [campo]));
```

### [Motor] Optimization
```yaml
# [Configuración específica]
[configuración]:
  [parámetro]: [valor]
  [parámetro]: [valor]

# [Configuración específica]
[configuración]:
  [parámetro]: [valor]
  [parámetro]: [valor]
```

## Monitoring & Observability

### Telemetry Events
```[lenguaje]
:telemetry.execute(
  [:[proyecto], :[acción], :[evento]],
  %{duration: duration, result_count: count},
  %{query: query, organization_id: org_id}
)
```

### Metrics
- **[Métrica 1]**: [detalle]
- **[Métrica 2]**: [detalle]
- **[Métrica 3]**: [detalle]
- **[Métrica 4]**: [detalle]
- **[Métrica 5]**: [detalle]

### Health Checks
- `GET /health` - [descripción]
- `GET /health/[componente]` - [descripción]
- `GET /health/[componente]` - [descripción]
- `GET /health/[componente]` - [descripción]

## Common Queries

### "[Pregunta común 1]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 2]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 3]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 4]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 5]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 6]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 7]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

### "[Pregunta común 8]?"
→ [Respuesta concisa]
→ See: `[Módulo.función/aridad]`

## Troubleshooting

### [Problema 1]
1. [Paso 1]
2. [Paso 2]
3. [Paso 3]

### [Problema 2]
1. [Paso 1]
2. [Paso 2]
3. [Paso 3]

### [Problema 3]
1. [Paso 1]
2. [Paso 2]
3. [Paso 3]

## Links Útiles

- [Documento relacionado 1]
- [Documento relacionado 2]
- [Documento relacionado 3]
- [Documento relacionado 4]

**Última actualización**: [fecha]  
**Optimizado para**: [propósito principal]
