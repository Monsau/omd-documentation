# OpenMetadata - APIs & Integration Guide

## Overview

OpenMetadata provides a comprehensive REST API for programmatic access to all platform functionality. This guide covers the API architecture, endpoints, authentication, and integration patterns.

---

## API Architecture

```mermaid
flowchart TB
    subgraph "Client Applications"
        UI[Web UI]
        SDK[Python SDK]
        CLI[CLI Tools]
        CUSTOM[Custom Apps]
    end
    
    subgraph "API Gateway"
        AUTH[Authentication Layer<br/>JWT/OAuth2]
        RATE[Rate Limiting]
        VALID[Request Validation]
    end
    
    subgraph "API Server"
        ROUTER[API Router]
        CTRL[Controllers]
        SVC[Service Layer]
    end
    
    subgraph "Data Layer"
        REPO[Repository]
        CACHE[Cache]
        DB[(Database)]
        ES[(Search)]
    end
    
    UI --> AUTH
    SDK --> AUTH
    CLI --> AUTH
    CUSTOM --> AUTH
    
    AUTH --> RATE
    RATE --> VALID
    VALID --> ROUTER
    
    ROUTER --> CTRL
    CTRL --> SVC
    SVC --> REPO
    REPO --> CACHE
    REPO --> DB
    REPO --> ES
    
    style AUTH fill:#e6f3ff
    style ROUTER fill:#e1f5e1
    style DB fill:#fff4e6
```

---

## API Endpoints

### Base URL

```
Production: https://openmetadata.company.com/api
Development: http://localhost:8585/api
```

### API Versioning

```
/api/v1/*
```

Current version: **v1**

---

## Authentication

### JWT Token Authentication

```mermaid
sequenceDiagram
    participant Client
    participant AuthServer
    participant APIGateway
    participant ResourceServer
    
    Client->>AuthServer: POST /api/v1/auth/login<br/>{email, password}
    AuthServer->>AuthServer: Validate Credentials
    AuthServer-->>Client: 200 OK<br/>{accessToken, refreshToken}
    
    Client->>APIGateway: GET /api/v1/tables<br/>Authorization: Bearer {accessToken}
    APIGateway->>APIGateway: Validate Token
    APIGateway->>ResourceServer: Forward Request
    ResourceServer-->>APIGateway: Response Data
    APIGateway-->>Client: 200 OK<br/>{tables}
    
    Note over Client,AuthServer: Token Expires
    
    Client->>AuthServer: POST /api/v1/auth/refresh<br/>{refreshToken}
    AuthServer-->>Client: 200 OK<br/>{newAccessToken}
```

### Authentication Methods

#### 1. **Basic Authentication** (Login)

```bash
curl -X POST "https://openmetadata.company.com/api/v1/users/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "admin@company.com",
    "password": "your_password"
  }'
```

**Response**:
```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tokenType": "Bearer",
  "expiresIn": 3600
}
```

#### 2. **JWT Bearer Token**

```bash
curl -X GET "https://openmetadata.company.com/api/v1/tables" \
  -H "Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
```

#### 3. **API Key** (Bot Accounts)

```bash
curl -X GET "https://openmetadata.company.com/api/v1/tables" \
  -H "X-API-Key: your-api-key-here"
```

#### 4. **OAuth2 / OIDC**

Supports external identity providers:
- Google
- Okta
- Auth0
- Azure AD
- Custom OIDC providers

---

## Core API Endpoints

### Tables API

```mermaid
flowchart LR
    A[Tables API<br/>/api/v1/tables] --> B[GET<br/>List Tables]
    A --> C[GET<br/>Get Table by ID/FQN]
    A --> D[POST<br/>Create Table]
    A --> E[PATCH<br/>Update Table]
    A --> F[PUT<br/>Full Update]
    A --> G[DELETE<br/>Delete Table]
    A --> H[GET<br/>Table Versions]
    
    style A fill:#e6f3ff
    style B fill:#e1f5e1
    style D fill:#fff4e6
```

#### List Tables

```http
GET /api/v1/tables
```

**Query Parameters**:
- `fields`: Comma-separated list (columns, owner, tags, usageSummary)
- `database`: Filter by database FQN
- `limit`: Number of results (default: 10, max: 100)
- `before`: Cursor for pagination
- `after`: Cursor for pagination

**Example**:
```bash
curl -X GET "https://openmetadata.company.com/api/v1/tables?fields=columns,owner,tags&database=snowflake.sales_db&limit=20" \
  -H "Authorization: Bearer ${TOKEN}"
```

**Response**:
```json
{
  "data": [
    {
      "id": "uuid",
      "name": "customer_orders",
      "fullyQualifiedName": "snowflake.sales_db.public.customer_orders",
      "description": "Customer orders table",
      "columns": [...],
      "owner": {...},
      "tags": [...]
    }
  ],
  "paging": {
    "total": 150,
    "after": "cursor-token"
  }
}
```

#### Get Table by FQN

```http
GET /api/v1/tables/name/{fqn}
```

**Example**:
```bash
curl -X GET "https://openmetadata.company.com/api/v1/tables/name/snowflake.sales_db.public.customer_orders?fields=columns,owner,tags,usageSummary" \
  -H "Authorization: Bearer ${TOKEN}"
```

#### Create Table

```http
POST /api/v1/tables
```

**Request Body**:
```json
{
  "name": "customer_orders",
  "description": "Customer orders table",
  "columns": [
    {
      "name": "order_id",
      "dataType": "BIGINT",
      "description": "Unique order identifier",
      "constraint": "PRIMARY_KEY"
    },
    {
      "name": "customer_email",
      "dataType": "VARCHAR",
      "dataLength": 255,
      "description": "Customer email",
      "tags": [
        {
          "tagFQN": "PII.Email",
          "source": "Classification"
        }
      ]
    }
  ],
  "database": "snowflake.sales_db",
  "databaseSchema": "snowflake.sales_db.public"
}
```

#### Update Table (PATCH)

```http
PATCH /api/v1/tables/{id}
```

**JSON Patch Format** (RFC 6902):
```json
[
  {
    "op": "add",
    "path": "/description",
    "value": "Updated description"
  },
  {
    "op": "add",
    "path": "/tags/-",
    "value": {
      "tagFQN": "Tier.Tier1",
      "source": "Classification"
    }
  }
]
```

**Example**:
```bash
curl -X PATCH "https://openmetadata.company.com/api/v1/tables/${TABLE_ID}" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json-patch+json" \
  -d '[
    {
      "op": "add",
      "path": "/description",
      "value": "Updated description"
    }
  ]'
```

---

### Search API

```http
GET /api/v1/search/query
```

**Query Parameters**:
- `q`: Search query string
- `index`: Entity type (table, dashboard, pipeline, etc.)
- `from`: Offset (default: 0)
- `size`: Number of results (default: 10, max: 100)
- `filters`: JSON filters
- `sort_field`: Sort field
- `sort_order`: asc/desc

**Example**:
```bash
curl -X GET "https://openmetadata.company.com/api/v1/search/query?q=customer&index=table&size=20&filters=database:snowflake.sales_db" \
  -H "Authorization: Bearer ${TOKEN}"
```

**Response**:
```json
{
  "hits": {
    "total": {
      "value": 45
    },
    "hits": [
      {
        "_index": "table_search_index",
        "_id": "table-uuid",
        "_score": 12.5,
        "_source": {
          "id": "uuid",
          "name": "customer_orders",
          "fullyQualifiedName": "snowflake.sales_db.public.customer_orders",
          "description": "Customer orders table",
          "entityType": "table"
        }
      }
    ]
  }
}
```

---

### Lineage API

```mermaid
flowchart TB
    A[Table: raw_orders] --> B[Table: cleaned_orders]
    B --> C[Table: enriched_orders]
    C --> D[Table: orders_fact]
    C --> E[Table: customer_360]
    D --> F[Dashboard: Sales Overview]
    E --> F
    
    style A fill:#e6f3ff
    style C fill:#e1f5e1
    style F fill:#fff4e6
```

#### Get Entity Lineage

```http
GET /api/v1/lineage/{entityType}/name/{fqn}
```

**Query Parameters**:
- `upstreamDepth`: Levels of upstream lineage (default: 1, max: 3)
- `downstreamDepth`: Levels of downstream lineage (default: 1, max: 3)

**Example**:
```bash
curl -X GET "https://openmetadata.company.com/api/v1/lineage/table/name/snowflake.sales_db.public.customer_orders?upstreamDepth=2&downstreamDepth=2" \
  -H "Authorization: Bearer ${TOKEN}"
```

**Response**:
```json
{
  "entity": {
    "id": "table-uuid",
    "type": "table",
    "fullyQualifiedName": "snowflake.sales_db.public.customer_orders"
  },
  "nodes": [
    {
      "id": "upstream-table-uuid",
      "type": "table",
      "fullyQualifiedName": "snowflake.sales_db.staging.raw_orders"
    },
    {
      "id": "downstream-table-uuid",
      "type": "table",
      "fullyQualifiedName": "snowflake.analytics.orders_fact"
    }
  ],
  "edges": [
    {
      "fromEntity": "upstream-table-uuid",
      "toEntity": "table-uuid"
    },
    {
      "fromEntity": "table-uuid",
      "toEntity": "downstream-table-uuid"
    }
  ],
  "upstreamEdges": 1,
  "downstreamEdges": 1
}
```

#### Add Lineage Edge

```http
PUT /api/v1/lineage
```

**Request Body**:
```json
{
  "edge": {
    "fromEntity": {
      "id": "source-table-uuid",
      "type": "table"
    },
    "toEntity": {
      "id": "target-table-uuid",
      "type": "table"
    },
    "lineageDetails": {
      "sqlQuery": "INSERT INTO target SELECT * FROM source WHERE date > '2024-01-01'",
      "columnsLineage": [
        {
          "fromColumns": ["source.order_id"],
          "toColumn": "target.order_id"
        }
      ]
    }
  }
}
```

---

### Data Quality API

#### List Test Suites

```http
GET /api/v1/dataQuality/testSuites
```

#### Create Test Definition

```http
POST /api/v1/dataQuality/testDefinitions
```

**Request Body**:
```json
{
  "name": "columnValuesToBeBetween",
  "description": "Test if column values are between min and max",
  "testPlatforms": ["OpenMetadata"],
  "supportedDataTypes": ["NUMBER", "INT", "BIGINT"],
  "parameterDefinition": [
    {
      "name": "minValue",
      "dataType": "NUMBER",
      "required": true
    },
    {
      "name": "maxValue",
      "dataType": "NUMBER",
      "required": true
    }
  ]
}
```

#### Execute Test Case

```http
POST /api/v1/dataQuality/testCases/{testCaseId}/execute
```

---

## Python SDK Integration

### Installation

```bash
pip install openmetadata-ingestion==1.10.3
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

# Initialize client
server_config = OpenMetadataConnection(
    hostPort="https://openmetadata.company.com/api",
    authProvider=AuthProvider.openmetadata,
    securityConfig=OpenMetadataJWTClientConfig(
        jwtToken="your-jwt-token"
    )
)

metadata = OpenMetadata(server_config)

# Check connection
metadata.health_check()
```

### CRUD Operations

```python
from metadata.generated.schema.entity.data.table import Table
from metadata.generated.schema.type.entityReference import EntityReference

# Create
table = Table(
    name="customer_orders",
    fullyQualifiedName="snowflake.sales_db.public.customer_orders",
    columns=[...],
    database=EntityReference(id="db-uuid", type="database")
)
created_table = metadata.create_or_update(table)

# Read
table = metadata.get_by_name(
    entity=Table,
    fqn="snowflake.sales_db.public.customer_orders",
    fields=["columns", "owner", "tags"]
)

# Update
table.description = "Updated description"
updated_table = metadata.create_or_update(table)

# Delete
metadata.delete(Table, entity_id=table.id)
```

### Search

```python
from metadata.generated.schema.entity.data.table import Table

# Simple search
tables = metadata.es_search_from_fqn(
    entity_type=Table,
    fqn_search_string="customer"
)

# Advanced search
search_results = metadata.es_search(
    entity_type=Table,
    query="customer orders",
    size=20,
    filters={"database": "snowflake.sales_db"}
)
```

### Lineage

```python
# Get lineage
lineage = metadata.get_lineage_by_name(
    entity=Table,
    fqn="snowflake.sales_db.public.customer_orders",
    up_depth=2,
    down_depth=2
)

# Add lineage
from metadata.generated.schema.type.entityLineage import EntitiesEdge

edge = EntitiesEdge(
    fromEntity=EntityReference(id="source-uuid", type="table"),
    toEntity=EntityReference(id="target-uuid", type="table")
)
metadata.add_lineage(edge)
```

---

## REST API Integration Patterns

### Pagination

```mermaid
sequenceDiagram
    participant Client
    participant API
    
    Client->>API: GET /api/v1/tables?limit=100
    API-->>Client: {data: [...], paging: {after: "cursor1"}}
    
    Client->>API: GET /api/v1/tables?limit=100&after=cursor1
    API-->>Client: {data: [...], paging: {after: "cursor2"}}
    
    Client->>API: GET /api/v1/tables?limit=100&after=cursor2
    API-->>Client: {data: [...], paging: {}}
    
    Note over Client,API: No more results
```

### Bulk Operations

```python
# Bulk create
tables = [table1, table2, table3]
for table in tables:
    try:
        metadata.create_or_update(table)
    except Exception as e:
        print(f"Failed to create {table.name}: {e}")
```

### Webhooks

```mermaid
sequenceDiagram
    participant OM as OpenMetadata
    participant Webhook as Webhook Endpoint
    participant YourApp as Your Application
    
    OM->>OM: Entity Updated
    OM->>OM: Generate Event
    OM->>Webhook: POST /webhook<br/>{event payload}
    
    Webhook->>YourApp: Process Event
    YourApp->>YourApp: Update Cache
    YourApp->>YourApp: Trigger Workflow
    
    YourApp-->>Webhook: 200 OK
    Webhook-->>OM: Ack
```

**Configure Webhook**:
```json
{
  "name": "my-webhook",
  "description": "Webhook for entity updates",
  "endpoint": "https://your-app.com/webhook",
  "secretKey": "your-secret-key",
  "enabled": true,
  "batchSize": 10,
  "timeout": 10,
  "eventFilters": [
    {
      "entityType": "table",
      "eventType": "entityUpdated"
    }
  ]
}
```

**Webhook Payload**:
```json
{
  "eventId": "uuid",
  "eventType": "entityUpdated",
  "entityType": "table",
  "entityId": "table-uuid",
  "entityFQN": "snowflake.sales_db.public.customer_orders",
  "changeDescription": {...},
  "timestamp": 1698624000000,
  "userName": "admin"
}
```

---

## Error Handling

### HTTP Status Codes

| Code | Description | Action |
|------|-------------|--------|
| **200** | Success | Process response |
| **201** | Created | Entity created successfully |
| **204** | No Content | Delete successful |
| **400** | Bad Request | Check request format |
| **401** | Unauthorized | Refresh token |
| **403** | Forbidden | Check permissions |
| **404** | Not Found | Entity doesn't exist |
| **409** | Conflict | Entity already exists |
| **429** | Too Many Requests | Implement retry with backoff |
| **500** | Internal Server Error | Contact support |
| **503** | Service Unavailable | Retry later |

### Error Response Format

```json
{
  "code": 400,
  "message": "Invalid request body",
  "responseType": "BAD_REQUEST",
  "details": {
    "field": "columns",
    "error": "columns field is required"
  }
}
```

### Retry Logic

```python
import time
from requests.exceptions import RequestException

def api_call_with_retry(func, max_retries=3, backoff_factor=2):
    for attempt in range(max_retries):
        try:
            return func()
        except RequestException as e:
            if attempt == max_retries - 1:
                raise
            wait_time = backoff_factor ** attempt
            print(f"Retry {attempt + 1}/{max_retries} after {wait_time}s")
            time.sleep(wait_time)
```

---

## Rate Limiting

**Default Limits**:
- **Authenticated Users**: 1000 requests/hour
- **API Keys**: 5000 requests/hour
- **Admin Users**: 10000 requests/hour

**Rate Limit Headers**:
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1698624000
```

---

## Best Practices

### 1. **Use Field Selection**

✅ **Good**: Only request needed fields
```python
table = metadata.get_by_name(
    entity=Table,
    fqn="snowflake.sales_db.public.customer_orders",
    fields=["columns", "owner"]  # Only what you need
)
```

❌ **Bad**: Request all fields
```python
table = metadata.get_by_name(
    entity=Table,
    fqn="snowflake.sales_db.public.customer_orders",
    fields=["*"]  # Too much data
)
```

### 2. **Implement Caching**

```python
from functools import lru_cache
from datetime import datetime, timedelta

class CachedMetadata:
    def __init__(self, metadata):
        self.metadata = metadata
        self.cache = {}
        self.cache_ttl = timedelta(minutes=5)
    
    def get_table(self, fqn):
        if fqn in self.cache:
            cached_time, cached_data = self.cache[fqn]
            if datetime.now() - cached_time < self.cache_ttl:
                return cached_data
        
        data = self.metadata.get_by_name(entity=Table, fqn=fqn)
        self.cache[fqn] = (datetime.now(), data)
        return data
```

### 3. **Batch Operations**

```python
# Process in batches
batch_size = 100
for i in range(0, len(entities), batch_size):
    batch = entities[i:i + batch_size]
    for entity in batch:
        try:
            metadata.create_or_update(entity)
        except Exception as e:
            logging.error(f"Failed: {e}")
    time.sleep(1)  # Rate limiting
```

### 4. **Error Handling**

```python
from metadata.ingestion.ometa.client import APIError

try:
    table = metadata.get_by_name(entity=Table, fqn=fqn)
except APIError as e:
    if e.status_code == 404:
        print(f"Table not found: {fqn}")
    elif e.status_code == 401:
        print("Authentication failed")
        # Refresh token
    else:
        print(f"API error: {e}")
```

---

## Integration Examples

### Example 1: Sync dbt Models

```python
import json
from pathlib import Path

def sync_dbt_models(metadata, manifest_path):
    """Sync dbt models to OpenMetadata"""
    
    with open(manifest_path) as f:
        manifest = json.load(f)
    
    for node_id, node in manifest['nodes'].items():
        if node['resource_type'] == 'model':
            table = Table(
                name=node['name'],
                description=node.get('description', ''),
                columns=[
                    Column(
                        name=col_name,
                        dataType=col['type'],
                        description=col.get('description', '')
                    )
                    for col_name, col in node['columns'].items()
                ],
                database=EntityReference(id=db_id, type="database")
            )
            
            metadata.create_or_update(table)
            print(f"Synced: {node['name']}")
```

### Example 2: Custom Data Quality Checks

```python
def run_custom_quality_checks(metadata, table_fqn):
    """Run custom data quality checks"""
    
    table = metadata.get_by_name(entity=Table, fqn=table_fqn)
    
    # Check 1: Row count
    row_count = execute_query(f"SELECT COUNT(*) FROM {table_fqn}")
    if row_count == 0:
        create_test_result(metadata, table.id, "row_count", "FAILED")
    
    # Check 2: Freshness
    last_updated = execute_query(
        f"SELECT MAX(updated_at) FROM {table_fqn}"
    )
    if datetime.now() - last_updated > timedelta(days=1):
        create_test_result(metadata, table.id, "freshness", "FAILED")
```

---

## References

- **API Documentation**: https://docs.open-metadata.org/v1.10.x/sdk/python
- **OpenAPI Spec**: https://openmetadata.company.com/swagger-ui/
- **Python SDK**: https://pypi.org/project/openmetadata-ingestion/
- **GitHub**: https://github.com/open-metadata/OpenMetadata

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
