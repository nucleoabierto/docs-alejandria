# Deployment Guide Template

## Deployment Guide - [Environment/Service Name]

**Version**: [X.X]  
**Last Updated**: [YYYY-MM-DD]  
**DevOps Lead**: [Name]  
**Engineering Lead**: [Name]  

## 1. Overview

### Purpose
[Descripción del propósito de este deployment guide]

### Service Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   Application   │    │   Database      │
│   (Nginx/HAProxy)│────│   (Phoenix)     │────│   (PostgreSQL)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
          │                       │                       │
          ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   CDN           │    │   Background    │    │   Vector DB     │
│   (CloudFront)  │    │   Jobs          │    │   (Qdrant)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Deployment Strategy
- **Strategy**: [Blue-Green/Canary/Rolling]
- **Downtime**: [Zero/Minimal/Scheduled]
- **Rollback**: [Automatic/Manual]
- **Environment**: [Production/Staging/Development]
- **Orchestrator**: HashiCorp Nomad

## 2. Prerequisites

### Infrastructure Requirements

#### Minimum Resources
- **CPU**: [Number] cores
- **Memory**: [Size] GB RAM
- **Storage**: [Size] GB SSD
- **Network**: [Bandwidth] Gbps

#### Software Dependencies
- **Operating System**: [Ubuntu 22.04/CentOS 8/etc]
- **Docker**: [Version]+
- **HashiCorp Nomad**: [Version]+
- **HashiCorp Consul**: [Version]+ (for service discovery)
- **PostgreSQL**: [Version]+
- **Elixir/OTP**: [Version]/[Version]

#### External Services
- **Database**: [PostgreSQL RDS/Managed]
- **Cache**: [Redis ElastiCache/Managed]
- **CDN**: [CloudFront/CloudFlare]
- **Monitoring**: [DataDog/New Relic]
- **Logging**: [ELK Stack/Papertrail]

### Access Requirements

#### Required Permissions
- **AWS/Azure/GCP**: [IAM roles needed]
- **Database**: [Database permissions]
- **Docker Registry**: [Image pull permissions]
- **Monitoring**: [Access to dashboards]

#### Network Configuration
- **Ports**: [80, 443, 4000, 5432, 6333]
- **Firewall Rules**: [Inbound/Outbound rules]
- **Load Balancer**: [Health check configuration]
- **SSL/TLS**: [Certificate management]

## 3. Environment Setup

### Environment Variables

#### Required Variables
```bash
# Database
DATABASE_URL=postgresql://user:pass@host:5432/alejandria
POOL_SIZE=10

# Application
SECRET_KEY_BASE=[generated-secret]
PORT=4000
HOSTNAME=localhost

# External Services
QDRANT_URL=http://qdrant:6333
REDIS_URL=redis://redis:6379

# Integrations
GITHUB_CLIENT_ID=[client-id]
GITHUB_CLIENT_SECRET=[client-secret]
SHORTCUT_API_TOKEN=[token]

# Monitoring
SENTRY_DSN=[sentry-dsn]
DATADOG_API_KEY=[api-key]
```

#### Optional Variables
```bash
# Performance
BEAM_VM_ARGS="+K true +A 64"
POOL_SIZE=20

# Features
ENABLE_AI_FEATURES=true
ENABLE_ANALYTICS=true

# Logging
LOG_LEVEL=info
STRUCTURED_LOGGING=true
```

### Configuration Files

#### Docker Configuration
```dockerfile
# Dockerfile
FROM elixir:1.15-alpine AS builder

# Install build dependencies
RUN apk add --no-cache build-base npm git python3

# Prepare build directory
WORKDIR /app

# Install hex + rebar
RUN mix local.hex --force && \
    mix local.rebar --force

# Install dependencies
COPY mix.exs mix.lock ./
COPY config config
RUN mix deps.get --only prod

# Install node dependencies
COPY assets/package.json assets/package-lock.json ./assets/
RUN npm --prefix ./assets ci

# Build assets
COPY assets ./assets
RUN npm run --prefix ./assets deploy
RUN mix phx.digest

# Compile and build release
COPY . .
RUN mix compile
RUN mix release

# Runtime image
FROM alpine:3.18

RUN apk add --no-cache openssl ncurses-libs

WORKDIR /app

COPY --from=builder /app/_build/prod/rel/alejandria ./

EXPOSE 4000

CMD ["bin/alejandria", "start"]
```

#### Nomad Configuration
```hcl
# nomad.hcl
datacenter = "dc1"
data_dir = "/opt/nomad/data"

bind_addr = "0.0.0.0"

server {
  enabled = true
  bootstrap_expect = 3
}

client {
  enabled = true
  network_interface = "eth0"
}

consul {
  address = "127.0.0.1:8500"
}

docker {
  volumes {
    enabled = true
  }
}
```

#### Application Job Specification
```hcl
# alejandria.nomad
job "alejandria" {
  datacenters = ["dc1"]
  type = "service"

  update {
    max_parallel = 1
    health_check = "checks"
    min_healthy_time = "10s"
    healthy_deadline = "3m"
    auto_revert = true
    canary = 1
  }

  group "api" {
    count = 3

    restart {
      attempts = 3
      delay = "15s"
      interval = "10m"
      mode = "fail"
    }

    task "alejandria" {
      driver = "docker"

      config {
        image = "alejandria:v[version]"
        ports = ["http"]
        
        env {
          DATABASE_URL = "postgresql://user:pass@postgres:5432/alejandria"
          SECRET_KEY_BASE = "${NOMAD_VAR_secret_key_base}"
          QDRANT_URL = "http://qdrant:6333"
          PORT = "4000"
        }
      }

      resources {
        cpu    = 250 # MHz
        memory = 512 # MB
      }

      service {
        name = "alejandria"
        port = "http"

        connect {
          sidecar_service {}
        }

        check {
          name     = "health"
          type     = "http"
          path     = "/health"
          interval = "10s"
          timeout  = "3s"
        }
      }
    }
  }

  group "background" {
    count = 2

    task "worker" {
      driver = "docker"

      config {
        image = "alejandria:v[version]"
        command = "mix"
        args = ["phx.server"]
        
        env {
          DATABASE_URL = "postgresql://user:pass@postgres:5432/alejandria"
          SECRET_KEY_BASE = "${NOMAD_VAR_secret_key_base}"
          QDRANT_URL = "http://qdrant:6333"
          BACKGROUND_WORKERS = "true"
        }
      }

      resources {
        cpu    = 200 # MHz
        memory = 256 # MB
      }
    }
  }
}
```

#### Consul Configuration
```hcl
# consul.hcl
datacenter = "dc1"
data_dir = "/opt/consul/data"

server = true
bootstrap_expect = 3

ui = true
client_addr = "0.0.0.0"

connect {
  enabled = true
}
```

## 4. Deployment Process

### Pre-Deployment Checklist

#### Code Readiness
- [ ] All tests passing
- [ ] Code review completed
- [ ] Security scan passed
- [ ] Performance tests completed
- [ ] Documentation updated
- [ ] Version tagged in Git

#### Environment Readiness
- [ ] Target environment available
- [ ] Database migrations tested
- [ ] Backup strategy confirmed
- [ ] Monitoring configured
- [ ] Alert rules set up
- [ ] Rollback plan ready

#### Team Readiness
- [ ] Deployment team notified
- [ ] Stakeholders informed
- [ ] Support team on standby
- [ ] Communication plan ready

### Deployment Steps

#### Step 1: Preparation
```bash
# Clone latest code
git clone https://github.com/org/alejandria.git
cd alejandria
git checkout v[version]

# Build Docker image
docker build -t alejandria:v[version] .

# Tag and push to registry
docker tag alejandria:v[version] registry.example.com/alejandria:v[version]
docker push registry.example.com/alejandria:v[version]
```

#### Step 2: Database Migrations
```bash
# Run database migrations
mix ecto.migrate --repo Alejandria.Repo

# Verify migration status
mix ecto.migrate --status

# Create database backup
pg_dump alejandria_prod > backup_before_deploy.sql
```

#### Step 3: Application Deployment
```bash
# Submit job to Nomad
nomad job run alejandria.nomad

# Watch deployment status
nomad job status alejandria
nomad job allocs alejandria

# Monitor rollout
nomad job status -monitor alejandria

# Check service registration
consul catalog services
```

#### Step 4: Verification
```bash
# Check application health
curl http://alejandria.service.consul:4000/health

# Test critical endpoints via Consul DNS
curl http://alejandria.service.consul:4000/api/v1/documents
curl http://alejandria.service.consul:4000/api/v1/search

# Verify database connectivity
nomad alloc exec <alloc-id> mix ecto.migrate --status

# Check service mesh connectivity
consul intention create -deny -source web -destination alejandria
```

#### Functional Tests
```bash
# Run smoke tests
mix test test/smoke/

# Run integration tests
mix test test/integration/

# Test critical user flows
# 1. Document creation
# 2. Search functionality
# 3. Authentication
```

#### Performance Validation
```bash
# Load test new deployment
k6 run --vus 100 --duration 5m load-test.js

# Monitor response times
curl -w "@curl-format.txt" -o /dev/null -s https://api.alejandria.dev/api/v1/search

# Check error rates
curl -s https://api.alejandria.dev/metrics | grep error_rate
```

## 5. Monitoring & Observability

### Application Metrics

#### Key Performance Indicators
- **Response Time**: p50, p95, p99
- **Throughput**: Requests per second
- **Error Rate**: Percentage of failed requests
- **Memory Usage**: Heap and total memory
- **CPU Usage**: Percentage utilization
- **Database Connections**: Active vs idle

#### Custom Metrics
```elixir
# lib/alejandria/telemetry.ex
defmodule Alejandria.Telemetry do
  def setup do
    :telemetry.attach_many(
      "alejandria-logger",
      [
        [:alejandria, :search, :start],
        [:alejandria, :search, :stop],
        [:alejandria, :documents, :create],
        [:alejandria, :embeddings, :generate]
      ],
      &handle_event/4,
      nil
    )
  end

  defp handle_event([:alejandria, :search, :start], measurements, metadata, _config) do
    # Log search start
  end

  defp handle_event([:alejandria, :search, :stop], measurements, metadata, _config) do
    # Log search completion with timing
  end
end
```

### Infrastructure Monitoring

#### System Metrics
- **CPU**: Usage percentage, load average
- **Memory**: Usage, swap, OOM events
- **Disk**: Usage, I/O operations
- **Network**: Bandwidth, connections
- **Database**: Connections, query performance

#### Monitoring Stack
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'alejandria'
    static_configs:
      - targets: ['alejandria:4000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'postgres'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'qdrant'
    static_configs:
      - targets: ['qdrant:6333']
```

### Logging Strategy

#### Application Logs
```elixir
# config/prod.exs
config :logger, level: :info

config :logger, :console,
  format: "$time $metadata[$level] $message\n",
  metadata: [:request_id, :user_id, :organization_id]

config :logger, :file,
  path: "/var/log/alejandria/app.log",
  level: :info,
  format: "$time $metadata[$level] $message\n"
```

#### Log Formats
```json
{
  "timestamp": "2026-03-30T13:00:00Z",
  "level": "info",
  "message": "Document search completed",
  "request_id": "req_123",
  "user_id": "user_456",
  "organization_id": "org_789",
  "duration_ms": 145,
  "result_count": 5
}
```

### Alerting Rules

#### Critical Alerts
```yaml
# alerts.yml
groups:
- name: alejandria-critical
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High error rate detected"
      description: "Error rate is {{ $value }} errors per second"

  - alert: HighResponseTime
    expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 1
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High response time detected"
      description: "95th percentile response time is {{ $value }} seconds"
```

## 6. Rollback Procedures

### Automatic Rollback Triggers
- **Health check failures**: > 3 consecutive failures
- **Error rate threshold**: > 5% for 5 minutes
- **Response time degradation**: p95 > 2 seconds
- **Memory usage**: > 90% for 10 minutes

### Manual Rollback Steps

#### Step 1: Assess Impact
```bash
# Check current deployment
nomad job status alejandria
nomad job history alejandria

# Check recent allocations
nomad alloc status

# Check recent events
nomad job status -verbose alejandria

# Check service health
consul catalog service alejandria
```

#### Step 2: Execute Rollback
```bash
# Rollback to previous version
nomad job revert alejandria <previous-job-id>

# Monitor rollback progress
nomad job status -monitor alejandria

# Verify rollback success
nomad job status alejandria
consul catalog service alejandria
```

#### Step 3: Post-Rollback Verification
```bash
# Test application functionality
curl http://alejandria.service.consul:4000/health

# Verify database connectivity
nomad alloc exec <alloc-id> mix ecto.migrate --status

# Check error rates
nomad alloc logs <alloc-id> | grep ERROR
```

### Rollback Communication
- **Team notification**: Slack/Teams message
- **Stakeholder update**: Email with impact assessment
- **Incident report**: Document root cause analysis
- **Post-mortem**: Schedule retrospective

## 7. Security Considerations

### Container Security
```dockerfile
# Use minimal base image
FROM elixir:1.15-alpine

# Create non-root user
RUN addgroup -g 1001 -S alejandria && \
    adduser -u 1001 -S alejandria -G alejandria

# Set proper permissions
USER alejandria
WORKDIR /app

# Remove build tools in production
RUN apk del build-base npm git python3
```

### Network Security
- **Firewall rules**: Restrict access to necessary ports only
- **VPC isolation**: Separate environments in different VPCs
- **TLS encryption**: Enforce HTTPS everywhere
- **API rate limiting**: Prevent abuse and DDoS

### Secrets Management
```yaml
# secrets.yaml (encrypted with SOPS)
apiVersion: v1
kind: Secret
metadata:
  name: alejandria-secrets
stringData:
  database-url: ENC[AES256_GCM,data:...]
  secret-key-base: ENC[AES256_GCM,data:...]
  github-client-secret: ENC[AES256_GCM,data:...]
```

### Compliance Requirements
- **Data encryption**: At rest and in transit
- **Audit logging**: All access and modifications
- **Access control**: Principle of least privilege
- **Vulnerability scanning**: Regular security scans

## 8. Backup & Disaster Recovery

### Backup Strategy

#### Database Backups
```bash
# Automated daily backup
pg_dump alejandria_prod | gzip > backup_$(date +%Y%m%d).sql.gz

# Point-in-time recovery backup
pg_basebackup -D /backup/base -Ft -z -P

# Backup verification
pg_restore --list backup_20260330.sql.gz | head -n 20
```

#### Application Backups
```bash
# Document storage backup
aws s3 sync s3://alejandria-documents s3://alejandria-backups/$(date +%Y%m%d)

# Configuration backup
nomad job status -output > backup_jobs_$(date +%Y%m%d).json
consul kv export > backup_consul_$(date +%Y%m%d).json
```

### Recovery Procedures

#### Database Recovery
```bash
# Restore from backup
gunzip -c backup_20260330.sql.gz | psql alejandria_prod

# Point-in-time recovery
pg_ctl start -D /backup/base
psql -c "SELECT pg_wal_replay_resume();"
```

#### Application Recovery
```bash
# Restore Nomad jobs
nomad job run backup_jobs_20260330.json

# Restore Consul configuration
consul kv import backup_consul_20260330.json

# Restore documents
aws s3 sync s3://alejandria-backups/20260330 s3://alejandria-documents
```

### Disaster Recovery Testing
- **Monthly drills**: Test recovery procedures
- **Documentation updates**: Keep procedures current
- **RTO/RPO targets**: Define and measure recovery objectives
- **Cross-region recovery**: Plan for region failures

## 9. Troubleshooting

### Common Issues

#### Application Won't Start
```bash
# Check logs
kubectl logs deployment/alejandria

# Check environment variables
kubectl exec -it deployment/alejandria -- env | grep DATABASE

# Check database connectivity
kubectl exec -it deployment/alejandria -- mix ecto.create
```

#### Database Connection Issues
```bash
# Test database connection
kubectl exec -it deployment/alejandria -- psql $DATABASE_URL

# Check connection pool
kubectl exec -it deployment/alejandria -- mix ecto.repo.pool

# Check database status
kubectl exec -it postgres-0 -- pg_isready
```

#### Performance Issues
```bash
# Check resource usage
kubectl top pods
kubectl top nodes

# Check application metrics
curl -s https://api.alejandria.dev/metrics | grep phoenix

# Profile application
kubectl exec -it deployment/alejandria -- :observer.start
```

### Debug Commands
```bash
# Phoenix console
kubectl exec -it deployment/alejandria -- iex -S mix

# Check running processes
kubectl exec -it deployment/alejandria -- ps aux

# Network connectivity test
kubectl exec -it deployment/alejandria -- nc -z postgres 5432
```

## 10. Maintenance

### Regular Maintenance Tasks

#### Daily
- [ ] Check system health metrics
- [ ] Review error logs
- [ ] Verify backup completion
- [ ] Monitor disk usage

#### Weekly
- [ ] Apply security patches
- [ ] Update dependencies
- [ ] Review performance metrics
- [ ] Clean up old logs

#### Monthly
- [ ] Security vulnerability scan
- [ ] Performance optimization review
- [ ] Disaster recovery test
- [ ] Capacity planning review

### Maintenance Windows
- **Schedule**: [Day of week, time]
- **Duration**: [Expected duration]
- **Notification**: [How to notify users]
- **Rollback**: [Rollback procedure]

## 11. Contact Information

### Emergency Contacts
- **DevOps Lead**: [Name] - [Phone] - [Email]
- **Engineering Lead**: [Name] - [Phone] - [Email]
- **Database Admin**: [Name] - [Phone] - [Email]
- **Infrastructure**: [Name] - [Phone] - [Email]

### Team Communication
- **Slack**: [#alejandria-deployments]
- **Email**: deploy-team@alejandria.dev
- **On-call**: [On-call schedule and rotation]

### External Support
- **Cloud Provider**: [Support contact]
- **Database Service**: [Support contact]
- **Monitoring Service**: [Support contact]

---

## Appendix

### A. Deployment Scripts

#### Automated Deployment Script
```bash
#!/bin/bash
# deploy.sh

set -e

VERSION=$1
ENVIRONMENT=$2

if [ -z "$VERSION" ] || [ -z "$ENVIRONMENT" ]; then
    echo "Usage: $0 <version> <environment>"
    exit 1
fi

echo "Deploying version $VERSION to $ENVIRONMENT"

# Build and push image
docker build -t alejandria:$VERSION .
docker tag alejandria:$VERSION registry.example.com/alejandria:$VERSION
docker push registry.example.com/alejandria:$VERSION

# Update deployment
sed "s/alejandria:v\[version\]/alejandria:$VERSION/g" alejandria.nomad > alejandria_deploy.nomad
nomad job run alejandria_deploy.nomad

# Wait for deployment
nomad job status -monitor alejandria

# Health check
sleep 30
curl http://alejandria.service.consul:4000/health

echo "Deployment completed successfully"
```

### B. Configuration Examples

#### Environment-Specific Configurations
```elixir
# config/prod.exs
import Config

config :alejandria, Alejandria.Repo,
  url: System.get_env("DATABASE_URL"),
  pool_size: String.to_integer(System.get_env("POOL_SIZE") || "10"),
  ssl: true

config :alejandria, AlejandriaWeb.Endpoint,
  url: [host: System.get_env("HOSTNAME"), port: 80],
  cache_static_manifest: "priv/static/cache_manifest.json",
  server: true

config :logger, level: :info
```

---

**Document Version**: 1.0  
**Last Updated**: [Date]  
**Next Review**: [Date]  
**Approved By**: [Name]
