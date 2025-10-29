# OpenMetadata Quick Reference Guide - v1.10.3

## Essential Information

### Latest Version
**v1.10.3** - Released October 22, 2025

### Official Resources
- **Website**: https://open-metadata.org
- **Documentation**: https://docs.open-metadata.org
- **GitHub**: https://github.com/open-metadata/OpenMetadata
- **Community**: https://slack.open-metadata.org
- **API Docs**: https://docs.open-metadata.org/swagger.html

---

## Quick Start Commands

### Docker Compose
```bash
# Clone repository
git clone https://github.com/open-metadata/OpenMetadata
cd OpenMetadata/docker/local-metadata

# Start OpenMetadata
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f openmetadata-server

# Stop OpenMetadata
docker-compose down

# Stop and remove data
docker-compose down -v
```

**Access**: http://localhost:8585  
**Default Credentials**: admin/admin

### Kubernetes (Helm)
```bash
# Add Helm repository
helm repo add open-metadata https://helm.open-metadata.org/
helm repo update

# Install OpenMetadata
helm install openmetadata open-metadata/openmetadata \
  --namespace openmetadata \
  --create-namespace \
  --version 1.10.3

# Check status
kubectl get pods -n openmetadata

# View logs
kubectl logs -n openmetadata -l app=openmetadata

# Uninstall
helm uninstall openmetadata -n openmetadata
```

### Python SDK
```bash
# Install
pip install openmetadata-ingestion

# Verify installation
metadata --version
```

---

## System Architecture

```
┌──────────────────────────────────────────────────────────┐
│                     Web UI (React)                       │
│         Discovery | Lineage | Quality | Governance       │
└──────────────────────────────────────────────────────────┘
                              ↕
┌──────────────────────────────────────────────────────────┐
│              OpenMetadata Server (Java/Spring)           │
│                     REST APIs (Port 8585)                │
└──────────────────────────────────────────────────────────┘
            ↕                                   ↕
┌───────────────────────┐         ┌──────────────────────────┐
│  MySQL/PostgreSQL     │         │  Elasticsearch/          │
│  (Metadata Store)     │         │  OpenSearch (Search)     │
│  Port 3306/5432       │         │  Port 9200/9300          │
└───────────────────────┘         └──────────────────────────┘
                              ↑
┌──────────────────────────────────────────────────────────┐
│              Ingestion Framework (Python)                │
│      Airflow | dbt | Databases | BI Tools | ...         │
└──────────────────────────────────────────────────────────┘
```

---

## Key Components

| Component | Technology | Purpose | Default Port |
|-----------|-----------|---------|--------------|
| **UI** | React + TypeScript | Web interface | 8585 |
| **Server** | Java (Spring Boot) | REST API & business logic | 8585 |
| **Database** | MySQL/PostgreSQL | Metadata storage | 3306/5432 |
| **Search** | Elasticsearch/OpenSearch | Full-text search | 9200, 9300 |
| **Ingestion** | Python | Extract metadata | N/A |

---

## System Requirements

### Minimum (Dev/Test)
- **CPU**: 4 cores
- **RAM**: 16 GB
- **Storage**: 50 GB
- **OS**: Linux, macOS, Windows

### Recommended (Production)
- **CPU**: 8+ cores
- **RAM**: 32+ GB
- **Storage**: 200+ GB SSD
- **OS**: Linux (Ubuntu 20.04+, RHEL 8+)

### Scalability
- **Data Sources**: 1000+ sources supported
- **Users**: 10,000+ concurrent users
- **Assets**: Millions of metadata objects
- **Performance**: <100ms search (p95)

---

## Connector Categories

### Databases (40+)
PostgreSQL, MySQL, Oracle, SQL Server, MongoDB, Cassandra, Snowflake, BigQuery, Redshift, Databricks, etc.

### Pipelines (25+)
Airflow, dbt, Dagster, Fivetran, Airbyte, Prefect, Glue, Data Factory, Databricks Pipeline, etc.

### BI & Dashboards (20+)
Tableau, Power BI, Looker, Metabase, Superset, Qlik, Mode, Sisense, Domo, etc.

### Messaging (5+)
Kafka, Kinesis, Pulsar, Redpanda, RabbitMQ

### ML Platforms (5+)
MLflow, SageMaker, Vertex AI, Azure ML

### Storage (5+)
S3, GCS, Azure Blob, ADLS

### Search (2+)
Elasticsearch, OpenSearch

**Total**: 100+ connectors

---

## Core Features Overview

| Feature | Description | Key Benefit |
|---------|-------------|-------------|
| **Data Discovery** | Search and explore all data assets | Find data 70% faster |
| **Data Lineage** | Visualize data flow and dependencies | Understand impact |
| **Data Quality** | Automated testing and monitoring | Improve reliability 60% |
| **Data Governance** | Policies, classification, glossary | Ensure compliance |
| **Data Observability** | Monitor data health and usage | Proactive issue detection |
| **Collaboration** | Conversations, tasks, announcements | Break down silos |

---

## Configuration Files

### Docker Compose
- **Location**: `docker/local-metadata/docker-compose.yml`
- **Environment**: `.env` file
- **Custom configs**: `docker-compose.override.yml`

### Kubernetes
- **Helm values**: `values.yaml`
- **ConfigMaps**: Database, Elasticsearch configs
- **Secrets**: Passwords, API keys

### Bare Metal
- **Main config**: `conf/openmetadata.yaml`
- **Log config**: `conf/openmetadata-log4j2.xml`
- **JVM options**: `conf/openmetadata-env.sh`

---

## Common Ports

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| OpenMetadata UI/API | 8585 | HTTP | Web interface & REST API |
| MySQL | 3306 | TCP | Metadata database |
| PostgreSQL | 5432 | TCP | Metadata database (alternative) |
| Elasticsearch | 9200 | HTTP | Search API |
| Elasticsearch | 9300 | TCP | Node communication |
| OpenSearch | 9200 | HTTP | Search API (alternative) |

---

## Environment Variables (Key)

```bash
# Database
DB_DRIVER_CLASS=com.mysql.cj.jdbc.Driver
DB_SCHEME=mysql
DB_HOST=mysql
DB_PORT=3306
DB_USER=openmetadata_user
DB_PASSWORD=<strong_password>
OM_DATABASE=openmetadata_db

# Elasticsearch
ELASTICSEARCH_HOST=elasticsearch
ELASTICSEARCH_PORT=9200
ELASTICSEARCH_SCHEME=http

# Authentication
AUTHENTICATION_PROVIDER=basic
# Options: basic, google, okta, auth0, azure, custom-oidc

# Server
SERVER_HOST=0.0.0.0
SERVER_PORT=8585
SERVER_ADMIN_PORT=8586

# Airflow (if using ingestion container)
AIRFLOW_HOST=http://ingestion:8080
```

---

## API Quick Reference

### Base URL
```
http://localhost:8585/api/v1
```

### Authentication
```bash
# Login to get JWT token
curl -X POST "http://localhost:8585/api/v1/users/login" \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@example.com", "password": "admin"}'

# Use token in subsequent requests
curl -H "Authorization: Bearer <token>" \
  http://localhost:8585/api/v1/tables
```

### Common Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/tables` | GET | List all tables |
| `/tables/{id}` | GET | Get table details |
| `/tables/name/{fqn}` | GET | Get table by FQN |
| `/search/query` | GET | Search assets |
| `/lineage/{entity}/{id}` | GET | Get lineage |
| `/dataQuality/testCases` | GET | List quality tests |
| `/tags` | GET | List all tags |
| `/glossaryTerms` | GET | List glossary terms |
| `/users` | GET | List users |
| `/teams` | GET | List teams |

### Example: Search for Tables
```bash
curl "http://localhost:8585/api/v1/search/query?q=customer&index=table_search_index" \
  -H "Authorization: Bearer <token>"
```

### Example: Get Table Lineage
```bash
curl "http://localhost:8585/api/v1/lineage/table/<table-id>?upstreamDepth=3&downstreamDepth=3" \
  -H "Authorization: Bearer <token>"
```

---

## Python SDK Quick Start

### Installation
```bash
pip install openmetadata-ingestion
```

### Basic Usage
```python
from metadata.ingestion.ometa.ometa_api import OpenMetadata
from metadata.generated.schema.entity.services.connections.metadata.openMetadataConnection import (
    OpenMetadataConnection,
    AuthProvider
)
from metadata.generated.schema.security.client.openMetadataJWTClientConfig import (
    OpenMetadataJWTClientConfig
)

# Create connection
server_config = OpenMetadataConnection(
    hostPort="http://localhost:8585/api",
    authProvider=AuthProvider.openmetadata,
    securityConfig=OpenMetadataJWTClientConfig(
        jwtToken="<your-jwt-token>"
    )
)

# Initialize client
metadata = OpenMetadata(server_config)

# List all tables
tables = metadata.list_entities(entity=Table)
for table in tables.entities:
    print(f"Table: {table.fullyQualifiedName}")

# Get specific table
table = metadata.get_by_name(entity=Table, fqn="service.database.schema.table")
print(f"Table: {table.name}")
print(f"Columns: {len(table.columns)}")

# Search
results = metadata.es_search_from_fqn(
    entity_type=Table,
    fqn_search_string="customer"
)
for result in results:
    print(f"Found: {result['fullyQualifiedName']}")
```

---

## CLI Commands

### Metadata CLI
```bash
# Version
metadata --version

# Run ingestion workflow
metadata ingest -c <config.yaml>

# Profile data
metadata profile -c <config.yaml>

# Test connection
metadata test -c <config.yaml>

# Generate sample config
metadata openmetadata-imports-migration --help
```

---

## Troubleshooting Quick Guide

### Server Won't Start

**Check logs**:
```bash
# Docker
docker-compose logs openmetadata-server

# Kubernetes
kubectl logs -n openmetadata -l app=openmetadata

# Bare metal
tail -f /opt/openmetadata/logs/openmetadata.log
```

**Common issues**:
- Database connection failed → Check DB credentials
- Elasticsearch unreachable → Verify ES is running
- Port 8585 already in use → Stop conflicting service
- Out of memory → Increase JVM heap size

### Can't Connect to Database

```bash
# Test MySQL connection
mysql -h localhost -u openmetadata_user -p openmetadata_db

# Test PostgreSQL connection
psql -h localhost -U openmetadata_user -d openmetadata_db

# Check Docker network
docker network inspect openmetadata_default
```

### Search Not Working

```bash
# Check Elasticsearch health
curl http://localhost:9200/_cluster/health

# Reindex all data
curl -X PUT "http://localhost:8585/api/v1/search/reindex" \
  -H "Authorization: Bearer <token>"
```

### Connector Failures

**Check ingestion logs**:
```bash
# In OpenMetadata UI: Settings → Applications → Logs

# Or via API
curl "http://localhost:8585/api/v1/services/ingestionPipelines/<id>/pipelineStatus" \
  -H "Authorization: Bearer <token>"
```

**Common issues**:
- Invalid credentials → Update service credentials
- Network timeout → Check firewall/security groups
- SSL certificate error → Add `sslConfig` in connector
- Permission denied → Grant required permissions

---

## Performance Tuning

### JVM Settings (Server)
```bash
# Edit docker-compose.yml or openmetadata-env.sh
JAVA_OPTS="-Xms2g -Xmx4g -XX:+UseG1GC"
```

### MySQL Optimization
```sql
-- Increase buffer pool size
SET GLOBAL innodb_buffer_pool_size = 4294967296; -- 4GB

-- Increase max connections
SET GLOBAL max_connections = 500;
```

### Elasticsearch Optimization
```yaml
# elasticsearch.yml
indices.memory.index_buffer_size: 30%
indices.queries.cache.size: 15%
```

---

## Security Checklist

- [ ] Change default admin password
- [ ] Enable SSO authentication
- [ ] Configure SSL/TLS for all connections
- [ ] Restrict network access with firewall rules
- [ ] Use secrets manager for credentials
- [ ] Enable audit logging
- [ ] Regularly update to latest version
- [ ] Implement backup and disaster recovery
- [ ] Review and configure RBAC policies
- [ ] Enable encryption at rest for databases

---

## Backup & Restore

### MySQL Backup
```bash
# Backup
mysqldump -u openmetadata_user -p openmetadata_db > backup.sql

# Restore
mysql -u openmetadata_user -p openmetadata_db < backup.sql
```

### Elasticsearch Backup
```bash
# Create snapshot repository
curl -X PUT "localhost:9200/_snapshot/my_backup" \
  -H 'Content-Type: application/json' \
  -d '{ "type": "fs", "settings": { "location": "/backup" }}'

# Create snapshot
curl -X PUT "localhost:9200/_snapshot/my_backup/snapshot_1?wait_for_completion=true"

# Restore
curl -X POST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore"
```

---

## Monitoring Metrics

### Key Metrics to Track

**Application**:
- API response times (p50, p95, p99)
- Request rate (requests/second)
- Error rate (%)
- Active user sessions

**Database**:
- Connection pool usage
- Query execution time
- Slow queries
- Database size

**Search**:
- Search query latency
- Indexing rate
- Cluster health
- Disk usage

**System**:
- CPU usage (%)
- Memory usage (%)
- Disk I/O
- Network I/O

---

## Version Upgrade

### Docker Compose
```bash
# Backup data first!
docker-compose down
cd OpenMetadata
git fetch
git checkout 1.10.3-release
cd docker/local-metadata
docker-compose up -d
```

### Kubernetes
```bash
# Backup first!
helm repo update
helm upgrade openmetadata open-metadata/openmetadata \
  --namespace openmetadata \
  --version 1.10.3
```

**Always review release notes before upgrading!**

---

## Useful Keyboard Shortcuts (UI)

| Shortcut | Action |
|----------|--------|
| `/` or `Ctrl+K` | Open global search |
| `Esc` | Close modals/drawers |
| `?` | Show help/shortcuts |
| `g` then `d` | Go to Data Assets |
| `g` then `q` | Go to Quality |
| `g` then `i` | Go to Insights |

---

## Common FQN Patterns

**Fully Qualified Name (FQN)** format:
```
service.database.schema.table
service.dashboard.chart
service.pipeline.task
```

**Examples**:
```
mysql_prod.shopify.public.customers
snowflake.analytics.dbt_models.customer_lifetime_value
tableau.sales_dashboard.revenue_chart
airflow.etl_pipeline.extract_task
```

---

## Support & Community

### Get Help
- 💬 **Slack**: https://slack.open-metadata.org (1000+ members)
- 📖 **Docs**: https://docs.open-metadata.org
- 🐛 **GitHub Issues**: https://github.com/open-metadata/OpenMetadata/issues
- 📧 **Email**: support@open-metadata.org

### Report Bugs
1. Check existing issues
2. Gather logs and error messages
3. Create detailed bug report
4. Include version and deployment type

### Request Features
1. Check feature requests
2. Describe use case
3. Provide examples
4. Engage in discussion

---

## License & Usage

**License**: Apache 2.0  
**Commercial Use**: ✅ Allowed  
**Modification**: ✅ Allowed  
**Distribution**: ✅ Allowed  
**Private Use**: ✅ Allowed  
**Patent Grant**: ✅ Included

---

## Quick Tips

💡 **Pro Tips**:
- Start with 2-3 high-value data sources
- Document ownership early
- Enable auto-classification for PII
- Set up Slack notifications for quality alerts
- Use domains to organize assets
- Create custom tags for your organization
- Schedule profiling during off-peak hours
- Export metadata regularly as backup

⚠️ **Common Pitfalls**:
- Trying to catalog everything at once
- Not assigning data owners
- Ignoring quality test failures
- Over-complicated governance policies
- Insufficient server resources
- Not monitoring ingestion logs

---

## Glossary of Terms

- **FQN**: Fully Qualified Name (unique identifier)
- **Lineage**: Data flow and dependencies
- **Ingestion**: Extracting metadata from sources
- **Profiling**: Statistical analysis of data
- **Tier**: Data importance classification
- **Domain**: Business area organization
- **Glossary**: Business terminology
- **Tag**: Label for classification
- **Owner**: Responsible person/team
- **Steward**: Governance responsible person

---

**Document Version**: 1.0  
**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3

**Print this page for quick reference!**
