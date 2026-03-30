---
id: ENG-REF-0009
area: "engineering"
type: "reference"
title: "Schema Migrations and Performance - Alejandria"
author: "engineering-team"
status: "active"
version: "1.0"
created: "2026-04-24"
last_updated: "2026-04-24"
related_docs: ["ENG-REF-0007", "ENG-REF-0008", "ADR-0003"]
---

Especificación de migrations, performance considerations y backup/restore para la base de datos PostgreSQL de Alejandria. Contiene estrategia con Alembic, índices GIN, partitioning futuro y procedimientos de backup.

---

# Schema Migrations and Performance - Alejandria

Este documento define la estrategia de migrations, performance considerations y procedimientos de backup/restore para la base de datos PostgreSQL de Alejandria MVP.

## Overview

**Base de datos**: PostgreSQL 15+  
**Framework de migrations**: Alembic  
**Foco**: Desarrollo local con Docker Compose (ver ADR-0001)

## Migrations

### Estrategia con Alembic

```bash
# Inicializar Alembic
alembic init migrations

# Configurar conexión en alembic.ini
# sqlalchemy.url = postgresql://alejandria:password@localhost:5432/alejandria

# Generar migration inicial
alembic revision --autogenerate -m "Initial schema"

# Aplicar migration
alembic upgrade head

# Revertir migration
alembic downgrade -1

# Ver historial de migrations
alembic history
```

### Configuración de Alembic

**alembic.ini**:
```ini
[alembic]
script_location = migrations
sqlalchemy.url = postgresql://alejandria:password@localhost:5432/alejandria

[post_write_hooks]
# Formatear SQL con pg_format
post_write_hooks = black
black.type = console_scripts
black.entrypoint = black alembic/versions/
```

**env.py**:
```python
from sqlalchemy import engine_from_config, pool
from alembic import context
from alejandria.db.models import Base

target_metadata = Base.metadata

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()
```

### Orden de Creación de Tablas

Las migrations deben seguir este orden para respetar foreign keys:

1. users
2. organizations
3. organization_members
4. document_areas
5. document_types
6. documents
7. document_versions
8. document_tags
9. document_relations

### Ejemplo de Migration Inicial

**migrations/versions/001_initial_schema.py**:
```python
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

def upgrade():
    # Users
    op.create_table(
        'users',
        sa.Column('id', postgresql.UUID(as_uuid=True), primary_key=True),
        sa.Column('email', sa.Text(), nullable=False),
        sa.Column('name', sa.Text(), nullable=False),
        sa.Column('api_key', sa.Text(), nullable=False),
        sa.Column('created_at', sa.TIMESTAMP(timezone=True), server_default=sa.text('NOW()')),
        sa.Column('updated_at', sa.TIMESTAMP(timezone=True), server_default=sa.text('NOW()')),
    )
    op.create_index('idx_users_email', 'users', ['email'], unique=True)
    op.create_index('idx_users_api_key', 'users', ['api_key'], unique=True)

    # Organizations
    op.create_table(
        'organizations',
        sa.Column('id', postgresql.UUID(as_uuid=True), primary_key=True),
        sa.Column('name', sa.Text(), nullable=False),
        sa.Column('type', sa.Text(), nullable=False),
        sa.Column('owner_id', postgresql.UUID(as_uuid=True), nullable=False),
        sa.Column('created_at', sa.TIMESTAMP(timezone=True), server_default=sa.text('NOW()')),
        sa.Column('updated_at', sa.TIMESTAMP(timezone=True), server_default=sa.text('NOW()')),
        sa.CheckConstraint("type IN ('personal', 'team', 'enterprise')", name='check_organization_type'),
    )
    op.create_index('idx_organizations_owner_id', 'organizations', ['owner_id'])
    op.create_index('idx_organizations_type', 'organizations', ['type'])
    op.create_foreign_key('fk_organizations_owner_id', 'organizations', 'users', ['owner_id'], ['id'], ondelete='CASCADE')

    # ... resto de tablas

def downgrade():
    op.drop_table('document_relations')
    op.drop_table('document_tags')
    op.drop_table('document_versions')
    op.drop_table('documents')
    op.drop_table('document_types')
    op.drop_table('document_areas')
    op.drop_table('organization_members')
    op.drop_table('organizations')
    op.drop_table('users')
```

## Performance Considerations

### Índices GIN para JSONB

Los índices GIN en `acl` y `metadata` permiten búsquedas eficientes dentro de JSONB:

```sql
-- Índices ya creados en schema
CREATE INDEX idx_documents_acl ON documents USING GIN (acl);
CREATE INDEX idx_documents_metadata ON documents USING GIN (metadata);
```

**Beneficios**:
- Búsquedas por ACL: `WHERE acl->>'visibility' = 'public'` usa índice GIN
- Búsquedas por metadata: `WHERE metadata->'tags' ? 'architecture'` usa índice GIN
- Operadores JSON: `@>`, `?`, `?&`, `?|` son soportados por índice GIN

**Ejemplos de queries optimizadas**:
```sql
-- Búsqueda de documentos públicos (usa índice GIN)
EXPLAIN ANALYZE SELECT * FROM documents 
WHERE acl->>'visibility' = 'public';

-- Búsqueda por tags en metadata (usa índice GIN)
EXPLAIN ANALYZE SELECT * FROM documents 
WHERE metadata->'tags' ? 'architecture';

-- Búsqueda por multiple tags (usa índice GIN)
EXPLAIN ANALYZE SELECT * FROM documents 
WHERE metadata->'tags' ?& ARRAY['architecture', 'security'];
```

### Índices Compuestos

Para queries frecuentes que combinan múltiples columnas:

```sql
-- Índice compuesto para búsquedas por organización y área
CREATE INDEX idx_documents_org_area ON documents(organization_id, area_id);

-- Índice compuesto para búsquedas por organización y status
CREATE INDEX idx_documents_org_status ON documents(organization_id, status);

-- Índice compuesto para búsquedas por owner y fecha
CREATE INDEX idx_documents_owner_created ON documents(owner_id, created_at DESC);
```

### Partitioning Futuro

Cuando crezca a >10M documentos, considerar partitioning por organización:

```sql
-- Ejemplo futuro (no implementar en MVP)
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID NOT NULL,
    -- resto de columnas
) PARTITION BY HASH (organization_id);

-- Crear particiones
CREATE TABLE documents_org_0 PARTITION OF documents FOR VALUES WITH (MODULUS 4, REMAINDER 0);
CREATE TABLE documents_org_1 PARTITION OF documents FOR VALUES WITH (MODULUS 4, REMAINDER 1);
CREATE TABLE documents_org_2 PARTITION OF documents FOR VALUES WITH (MODULUS 4, REMAINDER 2);
CREATE TABLE documents_org_3 PARTITION OF documents FOR VALUES WITH (MODULUS 4, REMAINDER 3);

-- Índices por partición (más eficiente)
CREATE INDEX idx_documents_org_0_acl ON documents_org_0 USING GIN (acl);
CREATE INDEX idx_documents_org_1_acl ON documents_org_1 USING GIN (acl);
-- etc.
```

**Beneficios de partitioning**:
- Queries por organización son más rápidas (solo escanean una partición)
- Maintenance es más eficiente (vacuum por partición)
- Backup/restore más granular

**Cuándo implementar**:
- >10M documentos en total
- >100 organizaciones activas
- Queries por organización son >100ms

### Connection Pooling

Para producción, usar connection pooling con PgBouncer:

```yaml
# docker-compose.yml (producción)
services:
  pgbouncer:
    image: pgbouncer/pgbouncer:latest
    environment:
      - DATABASES_HOST=postgres
      - DATABASES_PORT=5432
      - DATABASES_DBNAME=alejandria
      - DATABASES_USER=alejandria
      - DATABASES_PASSWORD=alejandria_dev
      - POOL_MODE=transaction
      - MAX_CLIENT_CONN=100
      - DEFAULT_POOL_SIZE=25
```

## Backup y Restore

### Backup

```bash
# Backup completo
docker compose exec postgres pg_dump -U alejandria alejandria > backup.sql

# Backup solo schema (sin datos)
docker compose exec postgres pg_dump -U alejandria --schema-only alejandria > schema.sql

# Backup solo datos (sin schema)
docker compose exec postgres pg_dump -U alejandria --data-only alejandria > data.sql

# Backup comprimido
docker compose exec postgres pg_dump -U alejandria alejandria | gzip > backup.sql.gz

# Backup con fecha
docker compose exec postgres pg_dump -U alejandria alejandria > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restore

```bash
# Restore completo
cat backup.sql | docker compose exec -T postgres psql -U alejandria alejandria

# Restore desde archivo comprimido
gunzip -c backup.sql.gz | docker compose exec -T postgres psql -U alejandria alejandria

# Restore solo schema
cat schema.sql | docker compose exec -T postgres psql -U alejandria alejandria

# Restore solo datos
cat data.sql | docker compose exec -T postgres psql -U alejandria alejandria
```

### Automatización de Backup

**Script de backup (backup.sh)**:
```bash
#!/bin/bash
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/alejandria_backup_$DATE.sql.gz"

# Crear directorio si no existe
mkdir -p $BACKUP_DIR

# Hacer backup comprimido
docker compose exec -T postgres pg_dump -U alejandria alejandria | gzip > $BACKUP_FILE

# Mantener solo últimos 7 días de backups
find $BACKUP_DIR -name "alejandria_backup_*.sql.gz" -mtime +7 -delete

echo "Backup completado: $BACKUP_FILE"
```

**Cron job para backup diario**:
```cron
# Backup diario a las 2 AM
0 2 * * * /path/to/backup.sh >> /var/log/backup.log 2>&1
```

### Monitoring de Performance

```sql
-- Ver queries lentos
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Ver tamaño de tablas
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Ver uso de índices
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan as index_scans,
    idx_tup_read as tuples_read,
    idx_tup_fetch as tuples_fetched
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;
```

## Referencias

- **ENG-REF-0007**: Schema Core Tables (documento principal)
- **ENG-REF-0008**: ACL and Metadata (ACL structure, tags, relations)
- **ADR-0003**: PostgreSQL como Base de Datos Relacional
