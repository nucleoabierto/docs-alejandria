# Runbook Template

## Runbook - [Incident/Service Name]

**Version**: [X.X]  
**Last Updated**: [YYYY-MM-DD]  
**On-call Engineer**: [Name]  
**Service Owner**: [Name]  

## 1. Overview

### Service Description
[Descripción del servicio y su función crítica]

### Criticality Level
- **Level 1**: Critical - Business impact
- **Level 2**: High - User impact
- **Level 3**: Medium - Internal impact
- **Level 4**: Low - Minor impact

### Dependencies
- **Upstream Services**: [Services that this service depends on]
- **Downstream Services**: [Services that depend on this service]
- **External Dependencies**: [Third-party services, APIs]

### SLA Requirements
- **Uptime**: [99.9%]
- **Response Time**: [p95 < 500ms]
- **Error Rate**: [< 1%]

## 2. Monitoring & Alerting

### Key Metrics

#### Application Metrics
- **Response Time**: p50, p95, p99
- **Throughput**: Requests per second
- **Error Rate**: 5xx errors percentage
- **Queue Depth**: Background jobs pending
- **Memory Usage**: Heap and total memory
- **CPU Usage**: Percentage utilization

#### Infrastructure Metrics
- **Server Load**: CPU, memory, disk
- **Database**: Connections, query performance
- **Network**: Bandwidth, latency
- **Storage**: Disk usage, I/O operations

#### Business Metrics
- **User Activity**: Active users
- **Feature Usage**: Critical feature adoption
- **Search Performance**: Query response times
- **Document Processing**: Processing rate

### Alert Thresholds

#### Critical Alerts (Page Immediately)
| Metric | Threshold | Duration | Escalation |
|---|---|---|---|
| Service Down | HTTP 5xx > 50% | 2 minutes | On-call → Manager |
| High Error Rate | Error rate > 10% | 5 minutes | On-call → Team |
| Database Down | Connection failed | 1 minute | On-call → DBA |
| Memory Critical | Memory > 95% | 5 minutes | On-call → SRE |

#### Warning Alerts (Email/Slack)
| Metric | Threshold | Duration | Action |
|---|---|---|---|
| High Response Time | p95 > 2s | 10 minutes | Monitor |
| Queue Growth | Jobs > 1000 | 15 minutes | Investigate |
| Disk Space | Disk > 80% | 30 minutes | Plan cleanup |
| CPU High | CPU > 80% | 20 minutes | Check load |

### Dashboard Links
- **Main Dashboard**: [Link to Grafana/Datadog]
- **Infrastructure Dashboard**: [Link]
- **Business Metrics Dashboard**: [Link]
- **Error Tracking Dashboard**: [Link]

## 3. Common Incidents

### Incident 1: High Error Rate

#### Symptoms
- **Alert**: High error rate > 10%
- **User Impact**: Users seeing 500 errors
- **Dashboard Spikes**: Error rate graph spike

#### Immediate Actions
```bash
# 1. Check service status
kubectl get pods -l app=alejandria
kubectl logs deployment/alejandria --since=5m

# 2. Check recent deployments
kubectl rollout history deployment/alejandria
kubectl get events --sort-by=.metadata.creationTimestamp

# 3. Check database connectivity
kubectl exec -it deployment/alejandria -- mix ecto.migrate --status

# 4. Check external services
curl -w "%{http_code}" -s -o /dev/null https://api.github.com/rate_limit
curl -w "%{http_code}" -s -o /dev/null https://api.app.shortcut.com/api/v2/member
```

#### Investigation Steps
1. **Check recent changes**: Deployments, configuration changes
2. **Review error logs**: Look for patterns, stack traces
3. **Check dependencies**: Database, external APIs
4. **Verify resources**: CPU, memory, disk space
5. **Test endpoints**: Manual testing of critical functions

#### Common Causes & Solutions
- **Recent deployment**: Rollback to previous version
- **Database issue**: Check connection pool, query performance
- **External API failure**: Check rate limits, API status
- **Resource exhaustion**: Scale up or restart services

#### Recovery Commands
```bash
# Rollback deployment
kubectl rollout undo deployment/alejandria

# Restart service
kubectl rollout restart deployment/alejandria

# Scale up resources
kubectl scale deployment alejandria --replicas=6

# Clear cache
kubectl exec -it deployment/alejandria -- mix cache.clear
```

### Incident 2: Slow Response Times

#### Symptoms
- **Alert**: p95 response time > 2 seconds
- **User Impact**: Slow loading, timeouts
- **Dashboard**: Response time graphs increasing

#### Immediate Actions
```bash
# 1. Check current response times
curl -w "@curl-format.txt" -o /dev/null -s https://api.alejandria.dev/api/v1/search

# 2. Check resource utilization
kubectl top pods
kubectl top nodes

# 3. Check database performance
kubectl exec -it postgres-0 -- psql -c "SELECT * FROM pg_stat_activity WHERE state = 'active';"

# 4. Check background jobs
kubectl exec -it deployment/alejandria -- mix Oban.Job | grep "attempting\|retryable\> 0"
```

#### Investigation Steps
1. **Database queries**: Check slow queries, locks
2. **External API calls**: Check response times
3. **Memory usage**: Check for GC pressure
4. **Network latency**: Check connectivity issues
5. **Load patterns**: Check for traffic spikes

#### Common Causes & Solutions
- **Database slowness**: Optimize queries, add indexes
- **Memory pressure**: Add more memory or optimize
- **External API latency**: Implement caching, timeouts
- **High load**: Scale horizontally, add load balancer

#### Performance Commands
```bash
# Database performance
kubectl exec -it postgres-0 -- psql -c "SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"

# Memory profiling
kubectl exec -it deployment/alejandria -- :observer.start

# Network testing
kubectl exec -it deployment/alejandria -- ping -c 5 postgres
```

### Incident 3: Database Connection Issues

#### Symptoms
- **Alert**: Database connection failures
- **User Impact**: Service unavailable, timeouts
- **Logs**: "connection refused", "too many connections"

#### Immediate Actions
```bash
# 1. Check database status
kubectl exec -it postgres-0 -- pg_isready
kubectl get pods -l app=postgres

# 2. Check connection count
kubectl exec -it postgres-0 -- psql -c "SELECT count(*) FROM pg_stat_activity;"

# 3. Check database logs
kubectl logs postgres-0 --since=10m

# 4. Test connection from application
kubectl exec -it deployment/alejandria -- psql $DATABASE_URL -c "SELECT 1;"
```

#### Investigation Steps
1. **Database health**: Check if database is running
2. **Connection limits**: Check max_connections setting
3. **Network connectivity**: Test network between services
4. **Resource usage**: Check database CPU/memory
5. **Connection leaks**: Check for unclosed connections

#### Common Causes & Solutions
- **Database restart**: Wait for recovery or restart
- **Connection limit reached**: Increase limit or fix leaks
- **Network issues**: Check network policies, firewall
- **Resource exhaustion**: Scale database resources

#### Recovery Commands
```bash
# Restart database
kubectl delete pod postgres-0
kubectl wait --for=condition=Ready pod/postgres-0

# Increase connection limit (temporarily)
kubectl exec -it postgres-0 -- psql -c "ALTER SYSTEM SET max_connections = 200;"
kubectl exec -it postgres-0 -- pg_ctl reload

# Kill long-running queries
kubectl exec -it postgres-0 -- psql -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE query_start < now() - interval '5 minutes' AND state = 'active';"
```

### Incident 4: Background Job Failures

#### Symptoms
- **Alert**: Job queue depth > 1000
- **User Impact**: Delayed processing, stale data
- **Dashboard**: Queue depth graph increasing

#### Immediate Actions
```bash
# 1. Check queue status
kubectl exec -it deployment/alejandria -- mix Oban.Job | head -20

# 2. Check failed jobs
kubectl exec -it deployment/alejandria -- mix Oban.Job | grep "discarded\> 0"

# 3. Check job processor status
kubectl logs deployment/alejandria | grep Oban

# 4. Check specific job types
kubectl exec -it deployment/alejandria -- mix Oban.Job | grep EmbeddingJob
```

#### Investigation Steps
1. **Job types**: Identify which job types are failing
2. **Error patterns**: Look for common error messages
3. **Resource issues**: Check if jobs are resource-starved
4. **External dependencies**: Check if external services are down
5. **Data issues**: Check for corrupted or invalid data

#### Common Causes & Solutions
- **External service failure**: Jobs waiting for external APIs
- **Resource limits**: Jobs failing due to memory/CPU limits
- **Data corruption**: Invalid data causing job failures
- **Code bugs**: Logic errors in job processing

#### Job Management Commands
```bash
# Retry failed jobs
kubectl exec -it deployment/alejandria -- mix Oban.Job.retry_all

# Delete stuck jobs
kubectl exec -it deployment/alejandria -- mix Oban.Job.cancel_all

# Pause queue (emergency)
kubectl exec -it deployment/alejandria -- mix Oban.Queue.pause("default")

# Resume queue
kubectl exec -it deployment/alejandria -- mix Oban.Queue.resume("default")
```

## 4. Emergency Procedures

### Service Outage Response

#### Step 1: Acknowledge Alert (5 minutes)
- [ ] Acknowledge in monitoring system
- [ ] Post update in Slack channel
- [ ] Start incident timer
- [ ] Create incident ticket

#### Step 2: Immediate Triage (10 minutes)
- [ ] Determine scope and impact
- [ ] Identify affected users
- [ ] Check recent changes
- [ ] Verify service status

#### Step 3: Stabilize Service (30 minutes)
- [ ] Apply quick fixes
- [ ] Rollback if necessary
- [ ] Scale resources if needed
- [ ] Communicate status updates

#### Step 4: Root Cause Analysis (Ongoing)
- [ ] Collect logs and metrics
- [ ] Document timeline
- [ ] Identify root cause
- [ ] Plan permanent fix

#### Step 5: Resolution & Recovery
- [ ] Implement permanent fix
- [ ] Verify service stability
- [ ] Update documentation
- [ ] Conduct post-mortem

### Communication Templates

#### Initial Incident Alert
```
🚨 INCIDENT DECLARED 🚨

Service: Alejandria API
Severity: Critical
Impact: Users experiencing 500 errors
Started: [Time]
Investigation: In progress

Next update: 15 minutes
#incident-alejandria
```

#### Status Update
```
📊 INCIDENT UPDATE 📊

Service: Alejandria API
Status: [Investigating/Mitigated/Resolved]
Impact: [Current impact]
Actions: [What we're doing]
ETA: [Estimated resolution time]

Last update: [Time]
#incident-alejandria
```

#### Resolution Notification
```
✅ INCIDENT RESOLVED ✅

Service: Alejandria API
Duration: [Total time]
Impact: [Summary of impact]
Root Cause: [Brief description]
Prevention: [What we're doing to prevent]

Follow-up: Post-mortem scheduled
#incident-alejandria
```

## 5. Escalation Procedures

### Escalation Matrix

| Severity | Response Time | Escalation Path |
|---|---|---|
| Critical | 5 minutes | On-call → Manager → Director |
| High | 15 minutes | On-call → Manager |
| Medium | 1 hour | On-call → Team lead |
| Low | 4 hours | On-call (email) |

### Contact Information

#### On-call Rotation
- **Primary**: [Name] - [Phone] - [Slack]
- **Secondary**: [Name] - [Phone] - [Slack]
- **Manager**: [Name] - [Phone] - [Email]

#### Escalation Contacts
- **Engineering Manager**: [Name] - [Phone]
- **Director of Engineering**: [Name] - [Phone]
- **VP of Engineering**: [Name] - [Phone]
- **CTO**: [Name] - [Phone]

#### External Support
- **Cloud Provider**: [Support contact]
- **Database Service**: [Support contact]
- **CDN Provider**: [Support contact]
- **Security Team**: [Contact]

### Escalation Triggers
- **No response** from on-call within 10 minutes
- **Critical incident** not resolved in 30 minutes
- **Business impact** exceeds $[amount]
- **Executive attention** required
- **Security incident** identified

## 6. Maintenance Procedures

### Planned Maintenance

#### Pre-Maintenance Checklist
- [ ] Schedule maintenance window
- [ ] Notify stakeholders
- [ ] Create backup
- [ ] Prepare rollback plan
- [ ] Update status page

#### Maintenance Steps
1. **Put service in maintenance mode**
2. **Create full backup**
3. **Apply maintenance changes**
4. **Verify functionality**
5. **Exit maintenance mode**
6. **Monitor for issues**

#### Post-Maintenance Checklist
- [ ] Verify service health
- [ ] Update documentation
- [ ] Communicate completion
- [ ] Monitor for regressions
- [ ] Archive maintenance logs

### Emergency Maintenance

#### Emergency Rollback Procedure
```bash
# 1. Identify last known good state
kubectl rollout history deployment/alejandria

# 2. Rollback immediately
kubectl rollout undo deployment/alejandria

# 3. Verify rollback success
kubectl rollout status deployment/alejandria
curl -f https://api.alejandria.dev/health

# 4. Communicate rollback
# Post in Slack, update status page
```

#### Emergency Scaling Procedure
```bash
# Scale up immediately
kubectl scale deployment alejandria --replicas=10

# Add more database connections if needed
kubectl exec -it postgres-0 -- psql -c "ALTER SYSTEM SET max_connections = 300;"

# Monitor resource usage
kubectl top pods -w
```

## 7. Security Incidents

### Security Incident Response

#### Immediate Actions
1. **Contain**: Isolate affected systems
2. **Assess**: Determine scope and impact
3. **Notify**: Alert security team immediately
4. **Preserve**: Collect evidence, logs

#### Security Contacts
- **Security Team**: [security@company.com]
- **CISO**: [Name] - [Phone]
- **Legal**: [legal@company.com]
- **PR**: [pr@company.com]

#### Evidence Collection
```bash
# Collect system logs
kubectl logs deployment/alejandria --since=1h > security_logs.txt

# Collect network logs
tcpdump -i any -w security_capture.pcap

# Collect system state
kubectl top pods > system_state.txt
kubectl get events > k8s_events.txt
```

## 8. Documentation & Knowledge Base

### Runbook Maintenance

#### Review Schedule
- **Monthly**: Review and update runbooks
- **Quarterly**: Full review and testing
- **Annually**: Major revision and updates

#### Update Process
1. **Test procedures** in staging
2. **Document changes** with version history
3. **Train team** on new procedures
4. **Archive old versions**

### Knowledge Base Links
- **Architecture Documentation**: [Link]
- **API Documentation**: [Link]
- **Database Schema**: [Link]
- **Service Dependencies**: [Link]
- **Past Incidents**: [Link to incident database]

## 9. Training & Onboarding

### New Engineer Onboarding

#### Required Knowledge
- **Service architecture**: Understand system components
- **Monitoring tools**: Proficient with dashboards
- **Debugging skills**: Log analysis, troubleshooting
- **Communication**: Incident response protocols

#### Training Checklist
- [ ] Review architecture documentation
- [ ] Access to monitoring tools
- [ ] Shadow on-call engineer
- [ ] Practice incident scenarios
- [ ] Review past incidents

### Drills & Simulations

#### Monthly Drills
- **Scenario**: [Predefined incident scenario]
- **Duration**: 30 minutes
- **Participants**: On-call team
- **Objective**: Practice response procedures

#### Quarterly Full-Scale Simulation
- **Scenario**: Complex multi-service incident
- **Duration**: 2 hours
- **Participants**: Entire engineering team
- **Objective**: Test coordination and communication

## 10. Metrics & Improvement

### Incident Metrics

#### Key Performance Indicators
- **MTTR**: Mean Time to Resolution
- **MTBF**: Mean Time Between Failures
- **Alert Accuracy**: False positive rate
- **Response Time**: Time to acknowledge

#### Quality Metrics
- **Documentation completeness**: % of incidents documented
- **Runbook accuracy**: % of procedures tested
- **Team proficiency**: Drill performance scores

### Continuous Improvement

#### Post-Incident Review
- **What happened**: Timeline and facts
- **What went well**: Positive aspects
- **What could be improved**: Areas for enhancement
- **Action items**: Specific improvements

#### Runbook Enhancement
- **Add new scenarios**: Based on real incidents
- **Update procedures**: Based on lessons learned
- **Improve monitoring**: Add missing metrics
- **Enhance automation**: Reduce manual steps

---

## Appendix

### A. Quick Reference Commands

#### Health Checks
```bash
# Service health
curl -f https://api.alejandria.dev/health

# Database health
kubectl exec -it postgres-0 -- pg_isready

# Queue health
kubectl exec -it deployment/alejandria -- mix Oban.Queue.status
```

#### Debug Commands
```bash
# View logs
kubectl logs deployment/alejandria --since=10m -f

# Connect to container
kubectl exec -it deployment/alejandria -- bash

# Database console
kubectl exec -it postgres-0 -- psql

# Phoenix console
kubectl exec -it deployment/alejandria -- iex -S mix
```

#### Scaling Commands
```bash
# Scale application
kubectl scale deployment alejandria --replicas=6

# Scale database (if applicable)
kubectl scale statefulset postgres --replicas=2

# Add resources
kubectl patch deployment alejandria -p '{"spec":{"template":{"spec":{"containers":[{"name":"alejandria","resources":{"limits":{"memory":"2Gi"}}}]}}}}'
```

### B. Incident Timeline Template

```
[Time] - Alert triggered
[Time + 2m] - On-call acknowledged
[Time + 5m] - Initial assessment completed
[Time + 10m] - Mitigation started
[Time + 20m] - Service stabilized
[Time + 30m] - Root cause identified
[Time + 45m] - Permanent fix implemented
[Time + 50m] - Incident resolved
[Time + 60m] - Post-mortem initiated
```

### C. Communication Scripts

#### User Notification Template
```
Subject: Service Disruption - Alejandria

Dear Alejandria Users,

We are currently experiencing a service disruption affecting [feature].

Our team is actively investigating and working to resolve the issue.

Status: [Investigating/Mitigated/Resolved]
Impact: [What users are experiencing]
ETA: [When we expect resolution]

We apologize for the inconvenience and appreciate your patience.

Updates will be posted at: [Status page URL]

Best regards,
Alejandria Team
```

---

**Runbook Version**: 1.0  
**Last Updated**: [Date]  
**Next Review**: [Date]  
**Approved By**: [Name]
