---
id: ADR-0001
area: "engineering"
type: "decision"
title: "Docker Compose para Desarrollo Local"
author: "engineering-team"
status: "accepted"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0002", "ADR-0003", "ADR-0004", "ADR-0005"]
---

Registro de decisión arquitectónica que justifica el uso de Docker Compose como herramienta de orquestación para desarrollo local de Alejandria. Contiene contexto del problema de configuración de múltiples servicios, decisión técnica específica, consecuencias positivas y negativas, alternativas evaluadas y pasos de implementación.

---

# ADR-0001: Docker Compose para Desarrollo Local

**Status**: Accepted  
**Date**: 2026-04-24  
**Authors**: Engineering Team  
**Reviewers**: Tech Lead

## Context

Sabemos que necesitamos una forma de orquestar múltiples servicios (PostgreSQL, Qdrant, FastMCP server) para desarrollo local de Alejandria. El MVP requiere tres servicios principales que deben ejecutarse simultáneamente:
- PostgreSQL para almacenamiento relacional
- Qdrant para búsqueda vectorial
- FastMCP server con Python para endpoints MCP

El desafío técnico es que configurar y gestionar estos servicios manualmente es propenso a errores y consume tiempo. Cada desarrollador necesita:
- Instalar PostgreSQL localmente con versión específica
- Configurar Qdrant con Docker o instalación nativa
- Configurar variables de entorno para cada servicio
- Gestionar dependencias de Python (sentence-transformers, FastMCP, etc.)
- Asegurar que todos los servicios estén corriendo antes de desarrollar

Entendemos que necesitamos:
- Configuración reproducible entre desarrolladores
- Setup rápido para nuevos miembros del equipo
- Aislamiento de dependencias
- Facilidad para limpiar y recrear entornos
- Compatibilidad con deployment futuro (Docker/Kubernetes)

## Decision

Usar Docker Compose como herramienta de orquestación para desarrollo local de Alejandria.

**Especificación técnica:**
- **Orquestador**: Docker Compose v2.20+
- **Servicios**: PostgreSQL, Qdrant, FastMCP server (Python)
- **Volúmenes**: Persistencia de datos para PostgreSQL y Qdrant
- **Redes**: Bridge network para comunicación entre servicios
- **Variables de entorno**: Archivo `.env` para configuración local

Docker Compose proporciona:
- Configuración declarativa en YAML (piensa en esto como un archivo que define todo tu entorno en un solo lugar)
- Orquestación de múltiples servicios con un comando
- Volúmenes para persistencia de datos
- Networks para comunicación entre contenedores
- Reproducibilidad exacta entre desarrolladores
- Integración nativa con Docker ecosystem

## Consequences

### Positivas

✅ **Setup reproducible**: Todo desarrollador ejecuta `docker compose up` y tiene el mismo entorno funcionando. No hay "it works on my machine" porque todos usan las mismas imágenes y configuración.

✅ **Aislamiento de dependencias**: Cada servicio está en su propio contenedor con sus dependencias. PostgreSQL usa su versión específica, Python usa su venv, Qdrant usa su configuración, sin conflictos.

✅ **Persistencia de datos**: Volúmenes Docker mantienen datos de PostgreSQL y Qdrant entre reinicios. Los desarrolladores no pierden su base de datos local al reiniciar contenedores.

✅ **Rápido onboarding**: Nuevo desarrollador clona repo, ejecuta `docker compose up`, y tiene todo funcionando en minutos. No necesita instalar PostgreSQL, Qdrant o configurar variables de entorno manualmente.

✅ **Facilidad de limpieza**: `docker compose down -v` elimina todo (contenedores, volúmenes, networks). Útil para resetear entorno a estado limpio.

✅ **Compatibilidad con deployment**: Docker Compose usa los mismos contenedores Docker que deployment futuro en producción. La transición de local a production es más sencilla.

✅ **Monitoreo sencillo**: `docker compose logs` muestra logs de todos los servicios en un solo lugar. `docker compose ps` muestra estado de todos los servicios.

### Negativas

❌ **Requiere Docker instalado**: Los desarrolladores necesitan Docker Desktop instalado. Esto añade overhead inicial pero es una herramienta estándar en desarrollo moderno.

❌ **Mayor consumo de recursos**: Ejecutar contenedores consume más RAM/CPU que instalación nativa. Para MVP con servicios ligeros esto es aceptable (PostgreSQL + Qdrant + Python ~2GB RAM).

❌ **Curva de aprendizaje para Docker**: Desarrolladores sin experiencia con Docker necesitan aprender conceptos básicos (contenedores, volúmenes, networks). Esto es mitigado con guía de desarrollo local.

❌ **Debugging más complejo**: Debugging dentro de contenedores puede ser más complejo que debugging de procesos nativos. Mitigamos esto con `docker compose exec` para acceder a contenedores.

❌ **Performance overhead**: Docker añade ligero overhead de virtualización. Para MVP esto no es crítico, pero puede ser relevante para performance testing intensivo.

## Alternatives Considered

### Instalación Nativa (PostgreSQL + Qdrant + Python venv)

**Por qué no la elegimos:**
- Configuración manual propensa a errores entre desarrolladores
- Diferentes versiones de PostgreSQL/Qdrant causan inconsistencias
- Setup lento para nuevos miembros del equipo
- Dificultad para limpiar y recrear entornos
- No reproduce deployment futuro (que usa Docker)

**Cuándo podría ser mejor:**
- Si el equipo tuviera restricciones para usar Docker
- Si el proyecto fuera extremadamente simple (un solo servicio)
- Si el performance crítico requiriera instalación nativa

### Kubernetes (minikube/kind)

**Por qué no la elegimos:**
- Overkill para desarrollo local
- Curva de aprendizaje más alta
- Requiere más recursos (minikube consume ~4GB RAM)
- Configuración más compleja que Docker Compose

**Cuándo podría ser mejor:**
- Si el proyecto usara Kubernetes en producción desde día 1
- Si necesitáramos features avanzados de Kubernetes en local
- Si el equipo tuviera experiencia fuerte en Kubernetes

### Vagrant + VirtualBox

**Por qué no la elegimos:**
- Más pesado que Docker (VM completa vs contenedores)
- Performance más lento (virtualización completa)
- Curva de aprendizaje más alta
- Ecosistema menos moderno que Docker

**Cuándo podría ser mejor:**
- Si necesitáramos entorno Linux completo en Windows/Mac
- Si el proyecto tuviera dependencias de OS específicas

## Implementation

### Paso 1: Estructura de directorios
```
alejandria/
├── docker-compose.yml
├── .env.example
├── .env (gitignored)
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── alejandria/
│       ├── __init__.py
│       ├── main.py
│       └── ...
├── postgres_data/ (gitignored, volumen)
└── qdrant_data/ (gitignored, volumen)
```

### Paso 2: docker-compose.yml
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: alejandria-postgres
    environment:
      POSTGRES_DB: alejandria
      POSTGRES_USER: alejandria
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:-alejandria_dev}
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U alejandria"]
      interval: 10s
      timeout: 5s
      retries: 5

  qdrant:
    image: qdrant/qdrant:v1.7.0
    container_name: alejandria-qdrant
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_data:/qdrant/storage
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:6333/health"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: alejandria-backend
    environment:
      DATABASE_URL: postgresql://alejandria:${POSTGRES_PASSWORD:-alejandria_dev}@postgres:5432/alejandria
      QDRANT_HOST: qdrant
      QDRANT_PORT: 6333
      PYTHONUNBUFFERED: 1
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
      qdrant:
        condition: service_healthy
    volumes:
      - ./backend:/app
    command: python -m alejandria.main

volumes:
  postgres_data:
  qdrant_data:
```

### Paso 3: Dockerfile para backend
```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Instalar dependencias del sistema
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Copiar requirements e instalar
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copiar código de aplicación
COPY alejandria/ ./alejandria/

# Exponer puerto (FastMCP usa stdio, pero dejamos para debugging)
EXPOSE 8000

# Command por defecto (se override en docker-compose)
CMD ["python", "-m", "alejandria.main"]
```

### Paso 4: requirements.txt
```
fastmcp>=0.1.0
pydantic>=2.0.0
qdrant-client>=1.7.0
psycopg2-binary>=2.9.0
sentence-transformers>=2.2.0
sqlalchemy>=2.0.0
alembic>=1.12.0
python-dotenv>=1.0.0
```

### Paso 5: .env.example
```bash
# PostgreSQL
POSTGRES_PASSWORD=alejandria_dev

# Qdrant
QDRANT_HOST=qdrant
QDRANT_PORT=6333

# Backend
LOG_LEVEL=DEBUG
```

### Paso 6: Comandos de desarrollo
```bash
# Iniciar todos los servicios
docker compose up

# Iniciar en modo detached (background)
docker compose up -d

# Ver logs de todos los servicios
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f backend

# Detener servicios
docker compose down

# Detener y eliminar volúmenes (limpieza completa)
docker compose down -v

# Reconstruir contenedor backend después de cambios
docker compose up --build backend

# Ejecutar comando dentro de contenedor
docker compose exec backend python -m pytest

# Acceder a shell de contenedor
docker compose exec backend bash
```

### Paso 7: Testing
- Verificar que PostgreSQL está accesible: `docker compose exec postgres psql -U alejandria -d alejandria`
- Verificar que Qdrant está accesible: `curl http://localhost:6333/health`
- Verificar que backend está corriendo: `docker compose logs backend`
- Integration tests con servicios reales en contenedores

### Paso 8: Debugging
- Usar `docker compose exec backend bash` para acceder a contenedor
- Usar `docker compose logs -f backend` para ver logs en tiempo real
- Usar `docker compose ps` para verificar estado de servicios
- Usar `docker compose down -v` para resetear entorno si hay problemas

## Related Decisions

- **ADR-0002**: Python + FastMCP como backend framework (backend service en Docker Compose)
- **ADR-0004**: Qdrant como vector database (qdrant service en Docker Compose)
- **ADR-0003**: PostgreSQL como base de datos relacional (postgres service en Docker Compose)
- **ADR-0005**: sentence-transformers para embeddings (instalado en backend service)

## Mitigation Strategies

Para mitigar los riesgos identificados:

**Requiere Docker instalado:**
- Documentar instalación de Docker Desktop en guía de desarrollo local
- Proporcionar alternativas para sistemas sin Docker (instalación nativa) si es necesario
- Usar imágenes Docker ligeras (alpine, slim) para reducir overhead

**Mayor consumo de recursos:**
- Usar imágenes ligeras (postgres:15-alpine, python:3.11-slim)
- Configurar límites de recursos en docker-compose.yml si es necesario
- Documentar requisitos mínimos de hardware (4GB RAM recomendado)

**Curva de aprendizaje para Docker:**
- Crear guía de desarrollo local detallada con comandos comunes
- Incluir troubleshooting section en guía
- Proporcionar cheat sheet de comandos Docker Compose

**Debugging más complejo:**
- Usar `docker compose exec` para acceso interactivo a contenedores
- Configurar logging detallado en desarrollo
- Usar volumenes para montar código local en contenedor (hot reload)

**Performance overhead:**
- Usar modo nativo de Docker (no WSL2 en Windows si es posible)
- Monitorear performance y optimizar si es necesario
- Considerar instalación nativa para performance testing intensivo
