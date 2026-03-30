---
id: ENG-GUIDE-001
area: "engineering"
type: "guide"
title: "Development Setup Guide"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-03-30"
last_updated: "2026-03-30"
related_docs: ["ENG-DEF-001", "ENG-DEC-0001", "PROD-STRAT-0004"]
---

Guía completa de configuración de entorno de desarrollo para Alejandria. Contiene pasos detallados para instalar dependencias, configurar base de datos, iniciar servicios, verificar instalación, y resolver problemas comunes. Diseñada para que nuevos desarrolladores puedan tener un entorno funcional en menos de 30 minutos y empezar a contribuir efectivamente al proyecto.

---

# Development Setup Guide

## Bienvenido al Equipo

Sabemos lo frustrante que puede ser configurar un nuevo entorno de desarrollo. Por eso diseñamos esta guía para que tu experiencia sea lo más fluida posible. Imagina poder empezar a contribuir a Alejandria hoy mismo, sin perder horas en configuraciones complejas.

Esta guía te llevará paso a paso desde cero hasta tener un entorno de desarrollo completamente funcional.

## Prerequisites

- **Elixir 1.15+** con OTP 26+
- **PostgreSQL 15+**
- **Docker & Docker Compose**
- **Git**

## 1. Clonar Repositorio

```bash
git clone https://github.com/tu-org/alejandria.git
cd alejandria
```

## 2. Configurar Base de Datos

```bash
# Iniciar PostgreSQL con Docker
docker-compose up -d postgres

# Crear base de datos
mix ecto.create

# Ejecutar migraciones
mix ecto.migrate
```

## 3. Configurar Qdrant

```bash
# Iniciar Qdrant con Docker
docker-compose up -d qdrant

# Verificar que esté corriendo
curl http://localhost:6333/collections
```

## 4. Instalar Dependencias

```bash
# Dependencias Elixir
mix deps.get
```

## 5. Configurar Variables de Entorno

```bash
# Copiar archivo de ejemplo
cp .env.example .env

# Editar .env con tus configuraciones
```

Variables requeridas:

```env
# Database
DATABASE_URL=postgresql://localhost/alejandria_dev

# Qdrant
QDRANT_URL=http://localhost:6333

# GitHub (opcional para desarrollo)
GITHUB_CLIENT_ID=tu_client_id
GITHUB_CLIENT_SECRET=tu_client_secret

# Secret Key Base
SECRET_KEY_BASE=$(mix phx.gen.secret)
```

## 6. Iniciar Servicios

```bash
# Backend Phoenix
mix phx.server
```

La aplicación iniciará en:
- **API REST**: http://localhost:4000
- **MCP Server**: Disponible para agentes de IA
- **Health Check**: http://localhost:4000/health

## 7. Verificar Instalación

```bash
# Test de conexión a DB
mix ecto.migrate --status

# Test de conexión a Qdrant
curl http://localhost:6333/health

# Test de API
curl http://localhost:4000/api/v1/health
```

## 8. Datos de Prueba

```bash
# Cargar seeds de prueba
mix run priv/repo/seeds.exs

# O crear manualmente:
iex -S mix
```

```elixir
# En iex
Alejandria.Documents.create_document(%{
  title: "Test Document",
  content: "# Test\nThis is a test document.",
  organization_id: "test-org-id"
})
```

## 9. Ejecutar Tests

```bash
# Tests unitarios
mix test

# Tests con coverage
mix test --cover

# Tests específicos
mix test test/alejandria/documents_test.exs
```

## 10. Desarrollo Local Tips

### Database Resets

```bash
# Reset completo
mix ecto.drop && mix ecto.create && mix ecto.migrate

# Solo migraciones
mix ecto.rollback --all && mix ecto.migrate
```

### Logs Debug

```bash
# Ver logs en tiempo real
mix phx.server --dev

# Logs específicos
config :logger, level: :debug
```

### Hot Reload

- **Backend**: Phoenix recarga automáticamente los cambios en código Elixir
- **CLI**: Los comandos de Mix se actualizan automáticamente

## 11. Troubleshooting

### Problemas Comunes

**PostgreSQL connection refused**
```bash
# Verificar que PostgreSQL esté corriendo
docker ps | grep postgres

# Reiniciar si es necesario
docker-compose restart postgres
```

**Qdrant not responding**
```bash
# Verificar logs
docker-compose logs qdrant

# Reiniciar
docker-compose restart qdrant
```

**Mix compilation errors**
```bash
# Limpiar y recompilar
mix clean --deps
mix deps.clean --all
mix deps.get
mix compile
```

### Performance Development

Para mejor performance en desarrollo:

```elixir
# config/dev.exs
config :alejandria, Alejandria.Repo,
  pool_size: 10,
  ownership_timeout: 60_000
```

## 12. CLI Tool Development

```bash
# Probar comandos CLI
mix alejandria ingest --file ./test.md --type technical
mix alejandria search "authentication"
mix alejandria list --type technical
mix alejandria relationships --doc-id doc-123
```

## 13. MCP Server Testing

```elixir
# En iex -S mix
# Probar MCP server
Alejandria.MCP.Server.handle_call("search_documents", %{
  "query" => "test query",
  "organization_id" => "test-org"
})
```

## 14. Integraciones Externas (Opcional)

### GitHub Webhook Testing

```bash
# Usar ngrok para exponer localhost
ngrok http 4000

# Configurar webhook en GitHub con URL:
# https://your-ngrok-id.ngrok.io/api/v1/webhooks/github
```

## 15. Deploy Preview

```bash
# Build para producción
mix phx.digest
mix release --env=prod

# O usar Docker
docker build -t alejandria .
docker run -p 4000:4000 alejandria
```

Antes de hacer PR:

1. Fork y branch desde `main`
2. Tests pasando: `mix test`
3. Formato correcto: `mix format --check-formatted`
4. Linting: `mix credo --strict`
5. Commits con mensajes claros

## 16. Contribución

## 17. Recursos Útiles
- [Elixir Guides](https://elixir-lang.org/getting-started/introduction.html)
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Project Issues](https://github.com/tu-org/alejandria/issues)
