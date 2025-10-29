# OpenMetadata - Troubleshooting Guide

## Common Issues & Solutions

### Installation Issues

#### Issue: OpenMetadata won't start

**Symptoms**:
- Container exits immediately
- "Connection refused" errors
- Blank page in browser

**Solutions**:

1. **Check Database Connection**
```bash
# Test MySQL connection
mysql -h localhost -u openmetadata_user -p -e "SELECT 1"

# Check container logs
docker logs openmetadata_server
```

2. **Check Elasticsearch**
```bash
# Test Elasticsearch
curl http://localhost:9200/_cluster/health

# Restart Elasticsearch
docker restart elasticsearch
```

3. **Check Port Availability**
```bash
# Windows PowerShell
netstat -ano | findstr "8585"

# If port is in use, stop the process or change OpenMetadata port
```

4. **Verify Configuration**
```bash
# Check environment variables
docker exec openmetadata_server env | grep -E "DB_|ES_"
```

---

#### Issue: Database migration fails

**Error**: `Liquibase migration failed`

**Solutions**:

1. **Check Database Permissions**
```sql
SHOW GRANTS FOR 'openmetadata_user'@'%';
-- Should have: CREATE, ALTER, DROP, INDEX permissions
```

2. **Manual Migration**
```bash
# Backup first!
mysql dump openmetadata > backup.sql

# Re-run migrations
docker exec openmetadata_server ./bootstrap/bootstrap_storage.sh
```

3. **Clean Install** (last resort)
```bash
# Drop and recreate database
mysql -u root -p -e "DROP DATABASE openmetadata; CREATE DATABASE openmetadata;"

# Restart OpenMetadata
docker-compose down
docker-compose up -d
```

---

### Ingestion Issues

#### Issue: Ingestion pipeline fails

**Common Errors**:

1. **Connection Timeout**
```
Error: Connection to source timed out
```

**Solution**:
```python
# Increase timeout in configuration
"sourceConfig": {
    "config": {
        "connectionTimeout": 300,  # 5 minutes
        "socketTimeout": 300
    }
}
```

2. **Authentication Failed**
```
Error: Authentication failed for user
```

**Solution**:
- Verify credentials
- Check service account permissions
- Ensure OAuth tokens are valid
- Test connection manually

3. **Missing Permissions**
```
Error: Permission denied on database.schema
```

**Solution**:
```sql
-- Grant necessary permissions
GRANT SELECT ON DATABASE mydb TO openmetadata_user;
GRANT USAGE ON SCHEMA myschema TO openmetadata_user;
```

---

#### Issue: Ingestion is slow

**Symptoms**:
- Taking hours to complete
- High resource usage
- Frequent timeouts

**Solutions**:

1. **Enable Parallel Processing**
```yaml
ingestion:
  workers: 10  # Increase workers
  batchSize: 100
```

2. **Filter Unnecessary Data**
```yaml
sourceConfig:
  config:
    schemaFilterPattern:
      includes:
        - "public"
        - "production"
      excludes:
        - "temp_.*"
        - "test_.*"
```

3. **Incremental Ingestion**
```yaml
# Only ingest changed metadata
ingestion:
  type: "incremental"
  lastRunTime: "2024-01-01T00:00:00Z"
```

---

### Search Issues

#### Issue: Search returns no results

**Symptoms**:
- Search shows "No results found"
- Recently ingested data not appearing

**Solutions**:

1. **Check Elasticsearch Index**
```bash
# Check index status
curl -X GET "localhost:9200/_cat/indices?v"

# Reindex if needed
curl -X POST "localhost:9200/_reindex" -H 'Content-Type: application/json' -d'{
  "source": { "index": "table_search_index" },
  "dest": { "index": "table_search_index_new" }
}'
```

2. **Trigger Manual Reindex**
```bash
# Via API
curl -X POST "http://localhost:8585/api/v1/search/reindex" \
  -H "Authorization: Bearer $TOKEN"
```

3. **Clear Search Cache**
```bash
# Restart Elasticsearch
docker restart elasticsearch

# Clear Redis cache
redis-cli FLUSHALL
```

---

#### Issue: Search is slow

**Solutions**:

1. **Increase Elasticsearch Resources**
```yaml
# docker-compose.yml
elasticsearch:
  environment:
    - "ES_JAVA_OPTS=-Xms4g -Xmx4g"  # Increase heap
  deploy:
    resources:
      limits:
        memory: 8g
```

2. **Optimize Indices**
```bash
curl -X POST "localhost:9200/table_search_index/_forcemerge?max_num_segments=1"
```

3. **Add More Elasticsearch Nodes**
```yaml
# Scale to 3 nodes for better performance
docker-compose scale elasticsearch=3
```

---

### UI Issues

#### Issue: UI is slow or unresponsive

**Solutions**:

1. **Clear Browser Cache**
- Press Ctrl+Shift+Delete
- Clear cached images and files
- Hard refresh: Ctrl+F5

2. **Check API Performance**
```bash
# Monitor API response times
curl -w "@curl-format.txt" http://localhost:8585/api/v1/tables
```

3. **Enable Redis Caching**
```yaml
cache:
  enabled: true
  type: "redis"
  host: "redis"
  port: 6379
```

4. **Optimize Database Queries**
```sql
-- Add missing indexes
CREATE INDEX idx_fqn ON entity(fully_qualified_name);
CREATE INDEX idx_type ON entity(type);
```

---

#### Issue: Login fails or session expires

**Solutions**:

1. **Check JWT Configuration**
```yaml
authenticationConfiguration:
  provider: "basic"
  publicKeyUrls:
    - "http://localhost:8585/api/v1/system/config/jwks"
  authority: "https://accounts.google.com"
```

2. **Increase Session Timeout**
```yaml
jwtTokenConfiguration:
  ttlInSeconds: 3600  # 1 hour
```

3. **Clear Cookies**
- Clear browser cookies for OpenMetadata domain
- Logout and login again

---

### Data Quality Issues

#### Issue: Quality tests fail unexpectedly

**Solutions**:

1. **Check Test Configuration**
```json
{
  "testCase": {
    "name": "column_values_to_be_between",
    "params": {
      "minValue": 0,
      "maxValue": 100
    }
  }
}
```

2. **Review Test Results**
```bash
# Get test results via API
curl -X GET "http://localhost:8585/api/v1/dataQuality/testCases/{testCaseId}/results" \
  -H "Authorization: Bearer $TOKEN"
```

3. **Adjust Test Thresholds**
```json
{
  "testCase": {
    "threshold": {
      "type": "percentage",
      "value": 95  # Allow 5% failure rate
    }
  }
}
```

---

### Lineage Issues

#### Issue: Lineage not showing

**Symptoms**:
- Blank lineage view
- Missing upstream/downstream connections

**Solutions**:

1. **Check Lineage Ingestion**
```python
# Ensure lineage is enabled in connector
"sourceConfig": {
    "config": {
        "includeLineage": true,
        "queryLogDuration": 30  # days
    }
}
```

2. **Manual Lineage Addition**
```python
from metadata.generated.schema.type.entityLineage import EntitiesEdge

edge = EntitiesEdge(
    fromEntity=EntityReference(id="source-uuid", type="table"),
    toEntity=EntityReference(id="target-uuid", type="table")
)
metadata.add_lineage(edge)
```

3. **Check SQL Parsing**
```python
# Enable SQL parsing logs
logging.getLogger("metadata.ingestion.source.database").setLevel(logging.DEBUG)
```

---

### Performance Issues

#### Issue: High memory usage

**Solutions**:

1. **Optimize JVM Settings**
```yaml
# For API server
JAVA_OPTS: >
  -Xms2g
  -Xmx4g
  -XX:+UseG1GC
  -XX:MaxGCPauseMillis=200
```

2. **Database Connection Pooling**
```yaml
database:
  driverClass: "com.mysql.cj.jdbc.Driver"
  url: "jdbc:mysql://mysql:3306/openmetadata"
  properties:
    maximumPoolSize: 50
    minimumIdle: 10
```

3. **Enable Query Caching**
```yaml
cache:
  enabled: true
  ttl: 300  # 5 minutes
```

---

#### Issue: Database growing too large

**Solutions**:

1. **Clean Old Audit Logs**
```sql
-- Delete audit logs older than 90 days
DELETE FROM audit_log WHERE timestamp < DATE_SUB(NOW(), INTERVAL 90 DAY);
```

2. **Archive Change Events**
```sql
-- Archive old change events
INSERT INTO change_event_archive SELECT * FROM change_event 
WHERE timestamp < DATE_SUB(NOW(), INTERVAL 180 DAY);

DELETE FROM change_event WHERE timestamp < DATE_SUB(NOW(), INTERVAL 180 DAY);
```

3. **Optimize Tables**
```sql
OPTIMIZE TABLE entity;
OPTIMIZE TABLE entity_relationship;
OPTIMIZE TABLE audit_log;
```

---

### Security Issues

#### Issue: Cannot access OpenMetadata

**Error**: `403 Forbidden` or `401 Unauthorized`

**Solutions**:

1. **Check User Roles**
```bash
# Via API
curl -X GET "http://localhost:8585/api/v1/users/name/{username}" \
  -H "Authorization: Bearer $TOKEN"
```

2. **Verify Permissions**
```bash
# Check user's roles and policies
curl -X GET "http://localhost:8585/api/v1/policies" \
  -H "Authorization: Bearer $TOKEN"
```

3. **Reset Admin Password**
```bash
# In database
UPDATE user SET password = SHA2('newpassword', 256) 
WHERE email = 'admin@openmetadata.org';
```

---

### Network Issues

#### Issue: Cannot connect to external services

**Solutions**:

1. **Check Network Configuration**
```bash
# Test connectivity
docker exec openmetadata_server curl -v https://api.external-service.com

# Check DNS resolution
docker exec openmetadata_server nslookup external-service.com
```

2. **Configure Proxy**
```yaml
# docker-compose.yml
environment:
  HTTP_PROXY: "http://proxy.company.com:8080"
  HTTPS_PROXY: "http://proxy.company.com:8080"
  NO_PROXY: "localhost,127.0.0.1"
```

3. **Firewall Rules**
```bash
# Windows PowerShell (Run as Administrator)
New-NetFirewallRule -DisplayName "OpenMetadata" -Direction Inbound -Port 8585 -Protocol TCP -Action Allow
```

---

## Diagnostic Commands

### Health Checks

```bash
# OpenMetadata API health
curl http://localhost:8585/api/v1/system/status

# Database health
mysql -h localhost -u root -p -e "SELECT 'OK' as status;"

# Elasticsearch health
curl http://localhost:9200/_cluster/health

# Redis health (if used)
redis-cli ping
```

### Log Collection

```bash
# Docker logs
docker logs openmetadata_server > om-server.log
docker logs mysql > mysql.log
docker logs elasticsearch > es.log

# Application logs
docker exec openmetadata_server cat /opt/openmetadata/logs/openmetadata.log

# Kubernetes logs
kubectl logs -n openmetadata deployment/openmetadata-server
```

### Resource Monitoring

```bash
# Docker stats
docker stats openmetadata_server mysql elasticsearch

# Kubernetes resources
kubectl top pods -n openmetadata
kubectl top nodes
```

---

## Getting Help

### Before Asking for Help

✅ Collect:
1. OpenMetadata version
2. Deployment method (Docker/K8s)
3. Error messages and logs
4. Steps to reproduce
5. Configuration files (redacted)

### Where to Get Help

1. **Documentation**: https://docs.open-metadata.org
2. **Community Slack**: https://slack.open-metadata.org
3. **GitHub Issues**: https://github.com/open-metadata/OpenMetadata/issues
4. **Stack Overflow**: Tag `openmetadata`

### Creating a Good Bug Report

```markdown
## Environment
- OpenMetadata Version: 1.10.3
- Deployment: Docker Compose
- OS: Windows 11
- Browser: Chrome 120

## Issue Description
[Clear description of the problem]

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Logs
[Relevant log excerpts]

## Screenshots
[If applicable]
```

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
