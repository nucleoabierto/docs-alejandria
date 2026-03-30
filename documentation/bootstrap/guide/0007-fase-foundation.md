---
id: BOOT-GUIDE-0007
area: "operations"
type: "guide"
title: "Fase Foundation: Sobrevivir y Crear Valor"
author: "leadership-team"
status: "active"
version: "1.0"
created: "2026-03-31"
last_updated: "2026-03-31"
related_docs: ["BOOT-GUIDE-0005", "BOOT-GUIDE-0006", "BOOT-GUIDE-0008"]
---

Guía práctica para las primeras 2 semanas críticas de implementación de documentación, enfocada en refinar los documentos bootstrap existentes y crear valor inmediato con documentación que resuelva problemas reales del equipo. **Requiere BOOT-GUIDE-0005 completada y guías bootstrap (0000-0004) como base.**

---

# Fase Foundation: De los Cimientos al Valor Inmediato

## Por Qué Esta Fase es Crítica

Ya completaste el bootstrap y tienes documentos estratégicos sólidos. Pero el equipo necesita documentación que resuelva problemas **hoy**, no en 6 meses.

**En esta fase transformamos:**
- Documentos estratégicos → Herramientas prácticas
- Documentos estáticos → Documentos vivos
- Documentos de planning → Documentos de ejecución

**Resultado esperado:** En 2 semanas, tu equipo usará activamente la documentación porque resuelve sus problemas diarios.

---

## Prerrequisitos: Bootstrap + Evaluación

**Esta guía requiere:**
- ✅ **BOOT-GUIDE-0000**: Información del Proyecto
- ✅ **BOOT-GUIDE-0001**: Visión Estratégica  
- ✅ **BOOT-GUIDE-0002**: System Overview
- ✅ **BOOT-GUIDE-0003**: Product Brief
- ✅ **BOOT-GUIDE-0004**: Strategic Roadmap
- ✅ **BOOT-GUIDE-0005**: Calidad y Versionamiento

Sin estos fundamentos, la refinería de documentos carecerá de contexto estratégico y alineación organizacional.

---

## Semana 1: Refinamiento Crítico

### Día 1-2: BOOT-GUIDE-0000 Express

**Objetivo:** Actualizar información del proyecto con datos críticos actuales

**Revisa tu BOOT-GUIDE-0000 existente y haz estas preguntas:**

**Información que ha cambiado:**
- [ ] **Equipo**: ¿Se unieron o salieron personas? ¿Roles cambiaron?
- [ ] **Estado**: ¿Pasamos de "idea" a "MVP en desarrollo"? ¿De "beta" a "operando"?
- [ ] **Recursos**: ¿Tenemos más/ menos presupuesto o tiempo?
- [ ] **Stakeholders**: ¿Hay nuevos inversionistas, clientes, o departamentos?

**Información crítica que falta:**
- [ ] **Timeline realista**: ¿Cuál es el deadline real para las próximas 4 semanas?
- [ ] **Bloqueadores actuales**: ¿Qué nos está deteniendo hoy?
- [ ] **Decisiones inmediatas**: ¿Qué necesitamos decidir esta semana?
- [ ] **Recursos críticos**: ¿Qué personas o herramientas son indispensables?

**Acción:**
1. **Actualiza BOOT-GUIDE-0000** con información crítica (1-2 horas)
2. **Marca cambios** con `[ACTUALIZADO 2026-XX-XX]` para tracking
3. **Comunica cambios** al equipo en la próxima daily/reunión

### Día 3-5: Documento de Emergencia Específico

**Objetivo:** Crear el documento que resuelve el problema más grande del equipo hoy

**Identifica el dolor más urgente:**
- [ ] **Técnico**: "No sé cómo configurar X" → Guía de setup específica
- [ ] **Proceso**: "No sé qué hacer cuando Y" → Runbook o procedimiento
- [ ] **Comunicación**: "No sé quién decide Z" → Mapa de stakeholders y decisiones
- [ ] **Producto**: "No sé qué construir ahora" → Priorización inmediata

**Crea el documento de emergencia:**

**Estructura del Documento de Emergencia:**
```markdown
---
id: EMER-XXX
area: "emergency"
type: "guide"
title: "[Problema Específico] - Guía Rápida"
author: "equipo-afectado"
status: "active"
priority: "critical"
created: "YYYY-MM-DD"
---

# [Problema Específico] - Guía Rápida

## El Problema
[Descripción clara y concisa del problema]

## Solución Inmediata
[Paso a paso para resolver el problema HOY]

## Quién Debe Hacer Qué
- [ ] **Persona X**: [Tarea específica]
- [ ] **Persona Y**: [Tarea específica]

## Cuándo
- **Hoy**: [Acciones inmediatas]
- **Esta semana**: [Seguimientos]
- **Próximo sprint**: [Mejoras]

## Recursos Necesarios
- [ ] **Herramienta X**: [Link o acceso]
- [ ] **Persona Y**: [Contacto]
- [ ] **Documento Z**: [Referencia]

## Métricas de Éxito
- [ ] [Resultado medible específico]

## Si Falla
[Plan B o contacto de emergencia]
```

**Ejemplos reales:**
- **"Setup de Desarrollo Local"** → Para nuevos desarrolladores
- **"Proceso de Deploy a Producción"** → Para equipo de operations
- **"Decisión de Features para Sprint Actual"** → Para product managers
- **"Comunicación con Clientes Críticos"** → Para equipo de ventas

**Validación:**
1. **Presenta el documento** a las personas afectadas (30 minutos)
2. **Recoge feedback** inmediato
3. **Itera** basado en sugerencias
4. **Publica** y comunica al equipo completo

### Día 6-7: Visión Mínima Funcional

**Objetivo:** Refinar BOOT-GUIDE-0001 para que conecte con el trabajo diario

**Revisa tu BOOT-GUIDE-0001 existente y haz estas preguntas:**

**Conexión con el trabajo actual:**
- [ ] **Problema diario**: ¿Cómo se manifiesta el problema que resolvemos en el trabajo de cada persona?
- [ ] **Impacto inmediato**: ¿Qué cambia en el día a día si resolvemos este problema?
- [ ] **Medición visible**: ¿Cómo podemos ver el progreso esta semana?

**Simplificación para acción:**
- [ ] **En una frase**: ¿Cuál es el resultado que buscamos esta semana?
- [ ] **En una acción**: ¿Qué debe hacer cada persona hoy para contribuir?
- [ ] **En una métrica**: ¿Cómo sabremos si estamos avanzando?

**Refina BOOT-GUIDE-0001:**

**Añade sección "Conexión con el Trabajo Diario":**
```markdown
## Conexión con el Trabajo Diario

### Para Desarrolladores
**Problema que resuelves hoy**: [Ejemplo: "Pierdes 2 horas buscando información"]
**Acción inmediata**: [Ejemplo: "Usa el template X para todo nuevo código"]
**Impacto que verás**: [Ejemplo: "Encontrarás cualquier documento en < 30 segundos"]

### Para Product Managers  
**Problema que resuelves hoy**: [Ejemplo: "No sabes qué priorizar"]
**Acción inmediata**: [Ejemplo: "Revisa el dashboard de métricas cada mañana"]
**Impacto que verás**: [Ejemplo: "Tendrás claridad sobre qué construir ahora"]

### Para Designers
**Problema que resuelves hoy**: [Ejemplo: "No sabes qué diseñar next"]
**Acción inmediata**: [Ejemplo: "Consulta el roadmap antes de empezar"]
**Impacto que verás**: [Ejemplo: "Tu diseño estará alineado con estrategia"]
```

**Validación:**
1. **Comparte la versión refinada** con el equipo (15 minutos)
2. **Pide feedback específico**: "¿Esto conecta con tu trabajo diario?"
3. **Ajusta** basado en respuestas
4. **Establece recordatorio** de revisión mensual

---

## Semana 2: Integración y Validación

### Día 8-10: Integración Cruzada

**Objetivo:** Asegurar que todos los documentos trabajen juntos sin contradicciones

**Revisa la coherencia entre tus documentos:**

**Checklist de Consistencia:**
- [ ] **BOOT-GUIDE-0000 vs BOOT-GUIDE-0001**: ¿La información del proyecto soporta la visión?
- [ ] **BOOT-GUIDE-0001 vs BOOT-GUIDE-0002**: ¿La arquitectura soporta la visión?
- [ ] **BOOT-GUIDE-0002 vs BOOT-GUIDE-0003**: ¿El roadmap es realista para la arquitectura?
- [ ] **BOOT-GUIDE-0003 vs Documento Emergencia**: ¿El documento de emergencia se alinea con el roadmap?

**Identifica y resuelve contradicciones:**

**Contradicciones Comunes y Soluciones:**

**"La visión dice X pero el roadmap dice Y"**
→ Revisa BOOT-GUIDE-0003: ¿El timeline es realista? ¿O la visión es demasiado ambiciosa?

**"La arquitectura soporta A pero el equipo necesita B"**
→ Revisa BOOT-GUIDE-0002: ¿Necesitamos ajustar el stack? ¿O priorizar diferente?

**"El documento de emergencia dice hacer X pero el roadmap dice Y"**
→ Decide: ¿Es emergencia real o priorización incorrecta?

**Acción:**
1. **Crea tabla de consistencia** (1 hora)
2. **Marca contradicciones** con `[REVISAR]`
3. **Resuelve las más críticas** primero
4. **Documenta decisiones** tomadas

### Día 11-12: Validación con Stakeholders

**Objetivo:** Asegurar que leadership y equipos clave estén alineados

**Sesión de Alineación (1 hora):**

**Invitados:**
- Leadership/management
- Leads de cada área clave
- Representantes del equipo afectado

**Agenda:**
1. **Estado actual** (5 min): Resumen de documentos refinados
2. **Consistencia encontrada** (10 min): Contradicciones y soluciones
3. **Validación de prioridades** (15 min): ¿Estamos enfocados en lo correcto?
4. **Feedback y ajustes** (20 min): Comentarios y sugerencias
5. **Next steps** (10 min): Acuerdos y responsabilidades

**Prepara para la sesión:**
```markdown
## Resumen de Foundation Phase

### Documentos Refinados
- **BOOT-GUIDE-0000**: [Cambios principales]
- **Documento Emergencia**: [Problema resuelto]
- **BOOT-GUIDE-0001**: [Conexión con trabajo diario]

### Consistencia Encontrada
- **Contradicción 1**: [Descripción] → [Solución]
- **Contradicción 2**: [Descripción] → [Solución]

### Decisiones Tomadas
- **Decisión 1**: [Qué decidimos y por qué]
- **Decisión 2**: [Qué decidimos y por qué]

### Próximos Pasos
- **Inmediato**: [Qué hacer esta semana]
- **Corto plazo**: [Qué hacer en las próximas 2 semanas]
- **Medio plazo**: [Qué hacer en el próximo mes]
```

### Día 13-14: Plan para Próxima Fase

**Objetivo:** Preparar la transición a Fase Spiral Expansion

**Evalúa el estado actual:**

**Métricas de Foundation Phase:**
- [ ] **Adopción**: ¿Qué porcentaje del equipo usa los documentos?
- [ ] **Valor**: ¿Qué problemas se resolvieron?
- [ ] **Consistencia**: ¿Qué tan alineados están los documentos?
- [ ] **Momentum**: ¿Tenemos energía para continuar?

**Identifica gaps para Spiral Phase:**
- [ ] **Documentación técnica**: ¿Necesitamos más detalle en BOOT-GUIDE-0002?
- [ ] **Documentación de producto**: ¿Necesitamos PRDs o user stories?
- [ ] **Documentación de proceso**: ¿Necesitamos guías de desarrollo?
- [ ] **Documentación de diseño**: ¿Necesitamos design system?

**Crea plan para Spiral Phase:**
```markdown
## Plan para Spiral Expansion

### Prioridades Altas (Primeras 2 semanas)
1. **[Documento X]**: [Por qué es urgente]
2. **[Documento Y]**: [Qué problema resuelve]

### Prioridades Medias (Siguientes 4 semanas)
1. **[Documento A]**: [Contexto y necesidad]
2. **[Documento B]**: [Impacto esperado]

### Recursos Necesarios
- **Personas**: [Quién necesita participar]
- **Tiempo**: [Cuánto tiempo asignar]
- **Herramientas**: [Qué necesitamos configurar]

### Métricas de Éxito para Spiral
- [ ] [Métrica 1]: [Cómo mediremos]
- [ ] [Métrica 2]: [Cómo sabremos que funcionó]
```

---

## Validación Final de Foundation Phase

### Checklist de Completitud

**Documentos Refinados ✅**
- [ ] BOOT-GUIDE-0000 actualizado con información crítica
- [ ] Documento de emergencia creado y validado
- [ ] BOOT-GUIDE-0001 conectado con trabajo diario
- [ ] Todos los documentos consistentes entre sí

**Validación del Equipo ✅**
- [ ] Stakeholders clave han revisado y aprobado
- [ ] Equipo afectado está usando los documentos
- [ ] Feedback recogido e incorporado
- [ ] Momentum mantenido para siguiente fase

**Preparación para Spiral ✅**
- [ ] Gaps identificados y priorizados
- [ ] Plan para Spiral Phase creado
- [ ] Recursos asignados para siguiente fase
- [ ] Métricas de éxito definidas

### Puntos de Decisión Críticos

**Si algo de lo anterior está incompleto:**

**"El equipo no está usando los documentos"**
→ Revisa: ¿Resuelven problemas reales? ¿Son fáciles de encontrar?

**"Hay contradicciones sin resolver"**
→ Prioriza: ¿Cuál afecta más el trabajo diario? Resuelve esa primero.

**"No tenemos recursos para Spiral Phase"**
→ Ajusta: ¿Qué podemos hacer con los recursos actuales? Enfócate en lo esencial.

---

## Siguiente Paso

Una vez completada esta Foundation Phase:

**Si eres startup/equipo pequeño:**
→ Considera si necesitas Spiral Phase o si Foundation es suficiente por ahora

**Si eres empresa mediana/grande:**
→ Continúa con [BOOT-GUIDE-0008: Fase Spiral Expansion](./0008-fase-expansion-espiral.md)

**En todos los casos:** Evalúa si el valor generado justifica la inversión en la siguiente fase.

---

## Recursos Adicionales

- **BOOT-GUIDE-0000**: Información del Proyecto (base para refinamiento)
- **BOOT-GUIDE-0001**: Visión Estratégica (base para conexión diaria)
- **BOOT-GUIDE-0002**: System Overview (base para consistencia)
- **BOOT-GUIDE-0003**: Product Brief (base para priorización)
- **BOOT-GUIDE-0004**: Strategic Roadmap (base para priorización)
- **BOOT-GUIDE-0005**: Calidad y Versionamiento (contexto organizacional)
- **BOOT-GUIDE-0006**: Mejores Prácticas (guía de escritura)

---

## Checklist de Validación para el Agente

Antes de continuar con el siguiente paso del flujo, confirma tu comprensión:

### ✅ Puntos de Revisión Obligatoria

**1. Refinamiento de Documentos:**
- [ ] BOOT-GUIDE-0000 está actualizado con información crítica
- [ ] Documento de emergencia resuelve un problema real
- [ ] BOOT-GUIDE-0001 conecta con el trabajo diario

**2. Consistencia y Validación:**
- [ ] Todos los documentos son consistentes entre sí
- [ ] Stakeholders clave han validado los cambios
- [ ] El equipo está usando activamente los documentos

**3. Preparación para Siguiente Fase:**
- [ ] Gaps para Spiral Phase están identificados
- [ ] Plan para siguiente fase está creado
- [ ] Recursos están asignados

### 🔄 Puntos de Confirmación con el Usuario

**Antes de proceder, responde:**

1. **¿Qué problema real resolvió el documento de emergencia?**
   - Tu respuesta: [Describe problema y solución implementada]
   - Confirmación del agente: [Valida impacto y valor generado]

2. **¿Cómo está el equipo usando los documentos refinados?**
   - Tu respuesta: [Describe adopción y feedback]
   - Confirmación del agente: [Identifica patrones de uso]

3. **¿Qué contradicciones encontraste y cómo las resolviste?**
   - Tu respuesta: [Lista contradicciones y soluciones]
   - Confirmación del agente: [Valida calidad de las soluciones]

4. **¿Estás listo para Spiral Phase o necesitas más tiempo en Foundation?**
   - Tu respuesta: [Describe preparación y decisión]
   - Confirmación del agente: [Ajusta plan según necesidad]

**Si hay desalineación en cualquier punto, detén el proceso y aclara antes de continuar.**
