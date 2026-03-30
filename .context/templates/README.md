# Templates - Plantillas de Documentación

Este directorio contiene plantillas reutilizables para la creación de documentos en la estructura de Alejandria.

## Plantillas Disponibles

### ADR Template (`adr-template.md`)
Plantilla estándar para Architecture Decision Records (ADRs).

**Uso:**
```bash
# Copiar para nuevo ADR
cp templates/adr-template.md decisions/adr-0003-nombre-decision.md
```

**Estructura:**
- Context: Problema y necesidad de decisión
- Decision: Decisión clara y específica
- Consequences: Impactos y trade-offs
- Alternatives: Otras opciones consideradas
- Implementation: Pasos para implementación
- Related: Referencias a otros ADRs

## Convenciones

### Nomenclatura de ADRs
- Formato: `adr-####-nombre-descriptivo.md`
- Numeración secuencial por proyecto
- Nombre en kebab-case, descriptivo

### Creación de Documentos
1. Copiar plantilla apropiada
2. Reemplazar placeholders `[ ]`
3. Actualizar estado y metadata
4. Mover a categoría correspondiente

### Versionado
- Plantillas pueden evolucionar
- Documentos existentes no se modifican automáticamente
- Considerar migración para plantillas mayores

## Próximas Plantillas Planificadas

- [ ] PRD Template (Product Requirements Document)
- [ ] TDD Template (Technical Design Document) 
- [ ] Retrospetiva Template
- [ ] Incident Report Template
- [ ] User Story Template
