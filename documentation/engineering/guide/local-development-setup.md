---
id: ENG-GUIDE-0001
area: "engineering"
type: "guide"
title: "Guía de Desarrollo Local"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-DEF-0001", "ADR-0001", "ADR-0002", "ADR-0003", "ADR-0004", "ADR-0005"]
---

Guía paso a paso para configurar y ejecutar el entorno de desarrollo local de Alejandria usando Docker Compose. Incluye requisitos previos, instalación, configuración, comandos comunes, troubleshooting y mejores prácticas.

---

# Guía de Desarrollo Local - Alejandria

Sabemos que configurar un entorno de desarrollo local puede ser confuso y frustrante. Esta guía te permite configurar un entorno de desarrollo local completo de Alejandria en minutos usando Docker Compose, para que puedas enfocarte en lo que realmente importa: construir código increíble.

## Table of Contents

1. [Requisitos Previos](#requisitos-previos)
2. [Configuración Inicial](#configuración-inicial)
3. [Iniciar el Entorno de Desarrollo](#iniciar-el-entorno-de-desarrollo)
4. [Comandos Comunes](#comandos-comunes)
5. [Flujo de Trabajo de Desarrollo](#flujo-de-trabajo-de-desarrollo)
6. [Troubleshooting](#troubleshooting)
7. [Mejores Prácticas](#mejores-prácticas)
8. [Referencias Rápidas (Cheat Sheet)](#referencias-rápidas-cheat-sheet)
9. [Recursos Adicionales](#recursos-adicionales)
10. [Soporte](#soporte)

---

Antes de comenzar, asegúrate de tener todo listo. No te preocupes, son herramientas comunes que probablemente ya tengas instaladas.

### Hardware Mínimo
- **CPU**: 2 cores o más
- **RAM**: 4GB mínimo (8GB recomendado para mejor experiencia)
- **Disk**: 10GB libres para volúmenes Docker

### Software Requerido
- **Docker Desktop**: v4.20+ (Mac/Windows) o Docker Engine v24+ (Linux)
  - Mac: https://www.docker.com/products/docker-desktop/
  - Windows: https://www.docker.com/products/docker-desktop/
  - Linux: https://docs.docker.com/engine/install/
- **Git**: Para clonar el repositorio
- **Editor de código**: VS Code, Cursor, o similar (opcional pero recomendado para mejor experiencia)

### Verificar Instalación de Docker

```bash
# Verificar versión de Docker
docker --version
# Output esperado: Docker version 24.x.x o superior

# Verificar versión de Docker Compose
docker compose version
# Output esperado: Docker Compose version v2.20.x o superior

# Verificar que Docker está corriendo
docker ps
# Output esperado: Lista de contenedores (puede estar vacía)
```

## Configuración Inicial

¡Excelente! Ya tienes todo lo necesario. Ahora vamos a configurar tu entorno de desarrollo. Solo toma unos minutos.

### 1. Clonar el Repositorio

```bash
# Clonar el repositorio
git clone https://github.com/tu-organizacion/alejandria.git
cd alejandria
```

¡Felicidades! Ya tienes el código en tu máquina. Ahora vamos a configurarlo.

### 2. Configurar Variables de Entorno

```bash
# Copiar archivo de ejemplo
cp .env.example .env

# Editar .env con tus preferencias (opcional)
# Valores por defecto funcionan para desarrollo
nano .env
```

Los valores por defecto funcionan perfectamente para desarrollo, así que no necesitas cambiar nada a menos que quieras personalizar algo.

**Contenido típico de .env:**
```bash
# PostgreSQL
POSTGRES_PASSWORD=alejandria_dev

# Qdrant
QDRANT_HOST=qdrant
QDRANT_PORT=6333

# Backend
LOG_LEVEL=DEBUG
```

### 3. Estructura de Directorios Esperada

Tu proyecto debería verse así. Si no coincide exactamente, no te preocupes, los nombres pueden variar ligeramente:

```
alejandria/
├── docker-compose.yml
├── .env
├── .env.example
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── alejandria/
│       ├── __init__.py
│       ├── main.py
│       └── ...
├── documentation/
│   └── ...
├── postgres_data/ (creado automáticamente por Docker)
└── qdrant_data/ (creado automáticamente por Docker)
```

## Iniciar el Entorno de Desarrollo

¡Ya casi estás listo! Ahora vamos a iniciar tu entorno de desarrollo.

### Comando Principal

```bash
# Iniciar todos los servicios (foreground con logs)
docker compose up

# O iniciar en background
docker compose up -d
```

**Qué sucede al ejecutar `docker compose up`:**
1. Docker descarga imágenes (PostgreSQL 15-alpine, Qdrant v1.7.0) si no existen localmente (la primera vez puede tardar unos minutos)
2. Construye imagen del backend (Python 3.11 + dependencias)
3. Inicia PostgreSQL y espera healthcheck
4. Inicia Qdrant y espera healthcheck
5. Inicia backend cuando dependencias están saludables
6. Muestra logs de todos los servicios en tiempo real

### Verificar que Todo Funciona

```bash
# Ver estado de todos los servicios
docker compose ps

# Output esperado:
# NAME                    STATUS                  PORTS
# alejandria-backend      Up (healthy)            0.0.0.0:8000->8000/tcp
# alejandria-postgres     Up (healthy)            0.0.0.0:5432->5432/tcp
# alejandria-qdrant       Up (healthy)            0.0.0.0:6333->6333/tcp, 0.0.0.0:6334->6334/tcp
```

¡Felicidades! Si ves los tres servicios con estado "Up (healthy)", tu entorno está funcionando perfectamente.

### Probar Conexiones

```bash
# Probar PostgreSQL
docker compose exec postgres psql -U alejandria -d alejandria -c "SELECT version();"

# Probar Qdrant
curl http://localhost:6333/health
# Output esperado: {"status":"ok"}

# Probar backend (si tiene endpoint HTTP)
curl http://localhost:8000/health
```

Si todas las conexiones funcionan, ¡estás listo para empezar a desarrollar!

## Comandos Comunes

Aquí tienes los comandos que usarás más frecuentemente. No necesitas memorizarlos todos, solo los que necesites.

### Gestión de Servicios

```bash
# Iniciar servicios (background)
docker compose up -d

# Iniciar servicios con logs en tiempo real
docker compose up

# Detener servicios
docker compose down

# Detener y eliminar volúmenes (limpieza completa)
docker compose down -v

# Reiniciar un servicio específico
docker compose restart backend

# Reconstruir un servicio después de cambios en Dockerfile
docker compose up --build backend
```

### Logs y Debugging

```bash
# Ver logs de todos los servicios
docker compose logs

# Ver logs en tiempo real
docker compose logs -f

# Ver logs de un servicio específico
docker compose logs -f backend
```

Los logs son tus amigos cuando algo no funciona como esperas. No te preocupes si ves errores al principio, es parte del proceso de aprendizaje.

# Ver últimos 100 líneas de logs
docker compose logs --tail=100 backend
```

### Acceso a Contenedores

```bash
# Acceder a shell de backend
docker compose exec backend bash

# Acceder a shell de PostgreSQL
docker compose exec postgres psql -U alejandria -d alejandria

# Ejecutar comando en backend
docker compose exec backend python -m pytest

# Ejecutar comando en PostgreSQL
docker compose exec postgres psql -U alejandria -d alejandria -c "SELECT * FROM documents;"
```

Acceder a los contenedores te permite ejecutar comandos directamente dentro del entorno, lo cual es útil para debugging y testing.

### Base de Datos

```bash
# Conectar a PostgreSQL
docker compose exec postgres psql -U alejandria -d alejandria

# Dentro de psql:
\l                    # Listar bases de datos
\dt                   # Listar tablas
\d documents          # Describir tabla documents
SELECT * FROM documents;  # Consultar datos
\q                    # Salir

# Backup de base de datos
docker compose exec postgres pg_dump -U alejandria alejandria > backup.sql

# Restore de base de datos
cat backup.sql | docker compose exec -T postgres psql -U alejandria alejandria
```

Los backups son importantes para no perder tu trabajo. Recomendamos hacer backups regulares si estás trabajando con datos importantes.

## Flujo de Trabajo de Desarrollo

Aquí te mostramos cómo trabajar efectivamente con tu entorno de desarrollo.

### Desarrollo Iterativo

```bash
# 1. Iniciar servicios en background
docker compose up -d

# 2. Editar código localmente (en backend/alejandria/)
# Los cambios se reflejan en contenedor gracias a volumen montado

# 3. Reiniciar backend para aplicar cambios
docker compose restart backend

# 4. Ver logs para verificar cambios
docker compose logs -f backend

# 5. Ejecutar tests
docker compose exec backend python -m pytest
```

Este flujo te permite desarrollar iterativamente con feedback rápido.

### Hot Reload (si configurado)

Si el backend está configurado con hot reload (usando tools como `watchfiles`), los cambios en código se aplican automáticamente sin reiniciar contenedor:

```bash
# Solo editar código localmente
# Los cambios se detectan y recargan automáticamente
# Ver logs para confirmar recarga
docker compose logs -f backend
```

El hot reload hace tu desarrollo más fluido, ya que no necesitas reiniciar manualmente después de cada cambio.

### Ejecutar Migrations (Alembic)

```bash
# Generar migration
docker compose exec backend alembic revision --autogenerate -m "Descripción del cambio"

# Aplicar migrations
docker compose exec backend alembic upgrade head

# Verificar versión actual
docker compose exec backend alembic current
```

Las migrations aseguran que tu base de datos esté siempre sincronizada con tu código. No olvides aplicarlas después de cambiar el schema.

## Troubleshooting

Sabemos que a veces las cosas no funcionan como esperamos. Aquí tienes soluciones para los problemas más comunes. No te preocupes, la mayoría de estos problemas tienen soluciones sencillas.

### Problema: Docker no está corriendo

**Síntoma:** Error "Cannot connect to the Docker daemon"

**Solución:**
```bash
# Iniciar Docker Desktop (Mac/Windows)
# O iniciar Docker service (Linux)
sudo systemctl start docker
sudo systemctl enable docker

# Verificar
docker ps
```

Este es un problema común y fácil de resolver. Solo necesitas iniciar Docker.

### Problema: Puerto ya en uso

**Síntoma:** Error "port is already allocated"

**Solución:**
```bash
# Ver qué proceso está usando el puerto
lsof -i :5432  # PostgreSQL
lsof -i :6333  # Qdrant
lsof -i :8000  # Backend

# O cambiar puertos en docker-compose.yml
ports:
  - "5433:5432"  # Usar 5433 en host en lugar de 5432
```

Si otro servicio está usando el puerto, puedes cambiar el puerto en docker-compose.yml o detener el servicio que lo está usando.

### Problema: Contenedor no inicia (crash loop)

**Síntoma:** Servicio muestra "Restarting" en `docker compose ps`

**Solución:**
```bash
# Ver logs del servicio para identificar error
docker compose logs backend

# Verificar healthcheck failures
docker compose ps

# Reconstruir contenedor
docker compose up --build backend

# Si persiste, limpiar y reiniciar
docker compose down -v
docker compose up
```

Los logs son tu mejor amigo aquí. Te dirán exactamente qué está fallando.

### Problema: Sin conexión a base de datos

**Síntoma:** Error "could not connect to server" en backend

**Solución:**
```bash
# Verificar que PostgreSQL está corriendo
docker compose ps postgres

# Ver logs de PostgreSQL
docker compose logs postgres

# Verificar conexión desde backend
docker compose exec backend ping postgres

# Verificar variables de entorno
docker compose exec backend env | grep DATABASE_URL
```

Asegúrate de que PostgreSQL esté completamente iniciado antes de intentar conectar desde el backend.

### Problema: Permisos en volúmenes (Linux)

**Síntoma:** Error "permission denied" al acceder a volúmenes

**Solución:**
```bash
# Ajustar permisos de volúmenes
sudo chown -R $USER:$USER postgres_data qdrant_data

# O usar usuario en docker-compose.yml
services:
  postgres:
    user: "${UID:-1000}:${GID:-1000}"
```

Los problemas de permisos en Linux son comunes. Ajustar los permisos suele resolver el problema.

### Problema: Imágenes Docker obsoletas

**Síntoma:** Comportamiento inesperado con versiones antiguas

**Solución:**
```bash
# Limpiar caché de imágenes
docker system prune -a

# Reconstruir desde cero
docker compose down -v
docker compose pull
docker compose up --build
```

A veces las imágenes antiguas pueden causar problemas inesperados. Limpiar y reconstruir suele resolver esto.

**Solución:**
```bash
# Pull de últimas imágenes
docker compose pull

# Reconstruir desde cero
docker compose down -v
docker compose build --no-cache
docker compose up
```

## Mejores Prácticas

Aquí tienes algunas prácticas que te ayudarán a trabajar más eficientemente.

### Desarrollo

- **Usar `docker compose up -d`** para desarrollo en background
- **Usar `docker compose logs -f`** para monitorear en tiempo real
- **Montar código como volumen** para hot reload (ya configurado en docker-compose.yml)
- **Usar `.env` para configuración local**, no commitear `.env` al repo
- **Limpiar volúmenes periódicamente** con `docker compose down -v` si hay corrupción de datos

### Testing

- **Ejecutar tests dentro de contenedor** para reproducibilidad
- **Usar base de datos separada para tests** (configurar en test env)
- **Limpiar datos de tests después de cada ejecución**

### Performance

- **Usar imágenes ligeras** (alpine, slim) ya configurado
- **Limitar recursos** si es necesario en docker-compose.yml:
  ```yaml
  services:
    backend:
      deploy:
        resources:
          limits:
            cpus: '2'
            memory: 2G
  ```

### Seguridad

- **No usar contraseñas por defecto en producción**
- **Rotar credenciales periódicamente**
- **Usar secrets de Docker Compose para producción** (no para desarrollo local)

## Referencias Rápidas (Cheat Sheet)

Aquí tienes los comandos más usados en un solo lugar para referencia rápida.

```bash
# Iniciar/Detener
docker compose up -d              # Iniciar
docker compose down               # Detener
docker compose down -v            # Detener y limpiar volúmenes

# Logs
docker compose logs               # Todos los logs
docker compose logs -f backend    # Logs en tiempo real de backend
docker compose logs --tail=50     # Últimas 50 líneas

# Estado
docker compose ps                 # Estado de servicios
docker compose top                # Procesos en contenedores

# Ejecutar comandos
docker compose exec backend bash  # Shell en backend
docker compose exec backend python -m pytest  # Ejecutar tests

# Reconstruir
docker compose build backend      # Reconstruir imagen
docker compose up --build         # Reconstruir e iniciar

# Limpieza
docker system prune -a            # Limpiar Docker system (cuidado)
docker volume prune               # Limpiar volúmenes no usados
```

## Recursos Adicionales

- **Documentación Docker Compose**: https://docs.docker.com/compose/
- **ADR-0001**: Decisión de usar Docker Compose para desarrollo local
- **System Architecture (ENG-DEF-0001)**: Arquitectura técnica del sistema
- **Guía de MCP endpoints** (próximamente): Cómo usar endpoints MCP desde Claude/Cursor

## Soporte

Si encuentras problemas no documentados en esta guía, no te preocupes, estamos aquí para ayudarte:
1. Verificar logs con `docker compose logs`
2. Consultar ADR-0001 para decisiones técnicas de Docker Compose
3. Revisar System Architecture (ENG-DEF-0001) para entender arquitectura
4. Contactar al equipo de ingeniería en Slack/Discord

¡Estamos a una conversación de distancia! No dudes en pedir ayuda si necesitas algo.
