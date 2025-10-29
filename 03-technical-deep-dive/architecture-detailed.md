# OpenMetadata - Detailed System Architecture

## Overview

OpenMetadata is built on a modern, cloud-native architecture designed for scalability, extensibility, and performance. This document provides an in-depth look at the technical architecture, components, and design decisions.

---

## High-Level Architecture

```mermaid
flowchart TB
    subgraph "Client Layer"
        UI[Web UI<br/>React + TypeScript]
        API_CLIENT[API Clients<br/>Python SDK, REST]
        CLI[CLI Tools<br/>Metadata CLI]
    end
    
    subgraph "API Gateway Layer"
        GATEWAY[API Gateway<br/>Kong/NGINX]
        AUTH[Authentication<br/>JWT/OAuth2]
    end
    
    subgraph "Application Layer"
        METADATA_API[Metadata API Server<br/>Spring Boot]
        INGESTION[Ingestion Framework<br/>Python]
        LINEAGE[Lineage Engine]
        QUALITY[Quality Engine]
        GOVERNANCE[Governance Engine]
    end
    
    subgraph "Data Layer"
        MYSQL[(MySQL/PostgreSQL<br/>Metadata Store)]
        ES[(Elasticsearch<br/>Search Index)]
        CACHE[(Redis Cache<br/>Session/Performance)]
    end
    
    subgraph "External Integrations"
        SOURCES[Data Sources<br/>100+ Connectors]
        STORAGE[Object Storage<br/>S3/Azure/GCS]
        MESSAGING[Message Queue<br/>Kafka/RabbitMQ]
    end
    
    UI --> GATEWAY
    API_CLIENT --> GATEWAY
    CLI --> GATEWAY
    
    GATEWAY --> AUTH
    AUTH --> METADATA_API
    
    METADATA_API --> INGESTION
    METADATA_API --> LINEAGE
    METADATA_API --> QUALITY
    METADATA_API --> GOVERNANCE
    
    METADATA_API --> MYSQL
    METADATA_API --> ES
    METADATA_API --> CACHE
    
    INGESTION --> SOURCES
    METADATA_API --> STORAGE
    INGESTION --> MESSAGING
    
    style UI fill:#e6f3ff
    style METADATA_API fill:#e1f5e1
    style MYSQL fill:#fff4e6
    style ES fill:#ffe6e6
```

---

## Core Components

### 1. **Metadata API Server**

```mermaid
classDiagram
    class MetadataAPIServer {
        +SpringBootApplication
        +RESTful Endpoints
        +GraphQL Support
        +WebSocket Support
        -EntityRepository
        -EventHandler
        -SearchService
        +createEntity()
        +updateEntity()
        +deleteEntity()
        +search()
    }
    
    class EntityService {
        +CRUD Operations
        +Validation
        +Versioning
        +Relationships
        +handleEntity()
        +validateSchema()
    }
    
    class SearchService {
        +ElasticsearchClient
        +Index Management
        +Query Building
        +indexEntity()
        +search()
        +suggest()
    }
    
    class EventService {
        +ChangeCapture
        +Webhooks
        +ActivityFeed
        +publishEvent()
        +subscribeToEvents()
    }
    
    MetadataAPIServer --> EntityService
    MetadataAPIServer --> SearchService
    MetadataAPIServer --> EventService
```

**Technology Stack**:
- **Framework**: Spring Boot 2.7+
- **Language**: Java 11+
- **API Standard**: OpenAPI 3.0
- **Authentication**: JWT, OAuth2, SAML
- **Port**: 8585 (default)

**Key Responsibilities**:
- RESTful API endpoints for all metadata operations
- Entity lifecycle management (CRUD)
- Schema validation using JSON Schema
- Event publishing for change notifications
- Access control and authorization
- Caching for performance optimization

---

### 2. **Metadata Store (MySQL/PostgreSQL)**

```mermaid
erDiagram
    ENTITY ||--o{ ENTITY_RELATIONSHIP : has
    ENTITY ||--o{ ENTITY_EXTENSION : has
    ENTITY ||--o{ TAG_USAGE : tagged_with
    ENTITY ||--o{ CHANGE_EVENT : generates
    
    USER ||--o{ TEAM_MEMBERSHIP : member_of
    USER ||--o{ ROLE_ASSIGNMENT : has
    TEAM ||--o{ TEAM_MEMBERSHIP : contains
    ROLE ||--o{ ROLE_ASSIGNMENT : assigned_to
    ROLE ||--o{ POLICY : enforces
    
    ENTITY {
        string id PK
        string fqn UK
        string type
        json metadata
        timestamp created
        timestamp updated
        string created_by
        string updated_by
    }
    
    TAG {
        string id PK
        string name
        string category
        string description
    }
    
    USER {
        string id PK
        string email UK
        string name
        json profile
        boolean is_admin
    }
```

**Schema Design**:
- **Tables**: 50+ core tables
- **Indexes**: Optimized for FQN lookups, searches, and joins
- **Partitioning**: By entity type and date
- **Replication**: Master-slave for read scaling

**Key Tables**:
- `entity` - Core metadata entities
- `entity_relationship` - Entity connections
- `entity_extension` - Custom properties
- `tag_usage` - Tagging relationships
- `user`, `team`, `role` - Access control
- `change_event` - Audit trail

---

### 3. **Search Engine (Elasticsearch/OpenSearch)**

```mermaid
flowchart LR
    A[Metadata Changes] --> B[Change Event]
    B --> C[Indexing Pipeline]
    C --> D[Document Builder]
    D --> E{Entity Type}
    
    E -->|Table| F[Table Index]
    E -->|Dashboard| G[Dashboard Index]
    E -->|Pipeline| H[Pipeline Index]
    E -->|Topic| I[Topic Index]
    
    F --> J[Search Cluster]
    G --> J
    H --> J
    I --> J
    
    K[User Query] --> L[Query Parser]
    L --> M[Search Executor]
    M --> J
    J --> N[Ranked Results]
    
    style C fill:#e1f5e1
    style J fill:#e6f3ff
    style N fill:#fff4e6
```

**Index Strategy**:
- **Separate indices** per entity type (table, dashboard, pipeline, etc.)
- **Sharding**: Configurable based on data volume
- **Replication**: 1 replica minimum for HA
- **Refresh interval**: 1 second for near real-time search

**Features**:
- Full-text search with relevance scoring
- Fuzzy matching for typos
- Faceted search and filtering
- Auto-suggestions and completion
- Aggregations for analytics

---

## Ingestion Framework

```mermaid
sequenceDiagram
    participant User
    participant Workflow
    participant Source
    participant Processor
    participant Sink
    participant MetadataAPI
    
    User->>Workflow: Start Ingestion
    Workflow->>Source: Initialize Connector
    Source->>Source: Connect to Data Source
    
    loop For Each Entity
        Source->>Source: Extract Metadata
        Source->>Processor: Send Entity
        Processor->>Processor: Transform & Enrich
        Processor->>Processor: Add Lineage
        Processor->>Processor: Add Tags
        Processor->>Sink: Send Processed Entity
        Sink->>MetadataAPI: POST /api/v1/tables
        MetadataAPI-->>Sink: 201 Created
    end
    
    Workflow->>User: Ingestion Complete
```

**Components**:

1. **Source Connectors** (100+)
   - Database: MySQL, PostgreSQL, Oracle, SQL Server, etc.
   - Data Warehouse: Snowflake, BigQuery, Redshift, etc.
   - BI Tools: Tableau, PowerBI, Looker, etc.
   - Pipeline: Airflow, Dagster, Prefect, etc.
   - Messaging: Kafka, Pulsar, RabbitMQ, etc.

2. **Processors**
   - Schema extraction
   - Lineage parsing (SQL, dbt)
   - PII detection
   - Tag propagation
   - Custom transformations

3. **Sink**
   - Metadata API client
   - Batch operations
   - Error handling
   - Retry logic

---

## Data Lineage Architecture

```mermaid
flowchart TB
    subgraph "Lineage Sources"
        SQL[SQL Queries]
        DBT[dbt Models]
        AIRFLOW[Airflow DAGs]
        SPARK[Spark Jobs]
    end
    
    subgraph "Lineage Extraction"
        PARSER[SQL Parser<br/>ANTLR4]
        DBT_PARSER[dbt Parser]
        DAG_PARSER[DAG Parser]
    end
    
    subgraph "Lineage Graph"
        NODES[Entity Nodes]
        EDGES[Lineage Edges]
        COLUMN[Column Lineage]
    end
    
    subgraph "Lineage API"
        QUERY[Query Lineage]
        IMPACT[Impact Analysis]
        UPSTREAM[Upstream Trace]
        DOWNSTREAM[Downstream Trace]
    end
    
    SQL --> PARSER
    DBT --> DBT_PARSER
    AIRFLOW --> DAG_PARSER
    SPARK --> PARSER
    
    PARSER --> NODES
    DBT_PARSER --> NODES
    DAG_PARSER --> NODES
    
    NODES --> EDGES
    EDGES --> COLUMN
    
    COLUMN --> QUERY
    COLUMN --> IMPACT
    COLUMN --> UPSTREAM
    COLUMN --> DOWNSTREAM
    
    style PARSER fill:#e6f3ff
    style NODES fill:#e1f5e1
    style QUERY fill:#fff4e6
```

**Lineage Types**:
- **Table-level**: Source → Target relationships
- **Column-level**: Column → Column transformations
- **Pipeline-level**: Task dependencies
- **Automated**: Extracted from SQL, dbt, Airflow
- **Manual**: User-defined lineage

---

## Data Quality Framework

```mermaid
flowchart LR
    subgraph "Test Definition"
        A[Built-in Tests]
        B[Custom Tests]
        C[Test Suites]
    end
    
    subgraph "Test Execution"
        D[Scheduler]
        E[Test Runner]
        F[Result Collector]
    end
    
    subgraph "Quality Dashboard"
        G[Test Results]
        H[Trends]
        I[Alerts]
    end
    
    A --> C
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    G --> I
    
    style E fill:#e1f5e1
    style H fill:#e6f3ff
```

**Test Types**:
- **Schema Tests**: Column presence, data types
- **Integrity Tests**: Unique, not null, foreign keys
- **Volume Tests**: Row count, freshness
- **Custom SQL Tests**: Business logic validation
- **Profiling Tests**: Statistical anomalies

**Execution**:
- **Scheduled**: Cron-based execution
- **On-demand**: Manual trigger
- **Event-driven**: On data changes
- **Distributed**: Parallel execution

---

## Security Architecture

```mermaid
flowchart TB
    subgraph "Authentication Layer"
        A[LDAP/AD]
        B[OAuth2/OIDC]
        C[SAML SSO]
        D[JWT Tokens]
    end
    
    subgraph "Authorization Layer"
        E[RBAC Engine]
        F[Policy Engine]
        G[Resource Permissions]
    end
    
    subgraph "Data Protection"
        H[Encryption at Rest]
        I[Encryption in Transit]
        J[PII Masking]
        K[Audit Logging]
    end
    
    A --> D
    B --> D
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    G --> I
    G --> J
    G --> K
    
    style D fill:#e6f3ff
    style F fill:#e1f5e1
    style K fill:#fff4e6
```

**Security Features**:
- **Multi-factor authentication**
- **Fine-grained RBAC** (Role-Based Access Control)
- **Attribute-based policies**
- **Column-level masking**
- **Comprehensive audit trails**
- **TLS 1.3** for transport encryption
- **AES-256** for data at rest

---

## Deployment Architecture

### Docker Deployment

```mermaid
flowchart TB
    subgraph "Docker Host"
        subgraph "Application Stack"
            OM[openmetadata-server]
            ING[ingestion]
        end
        
        subgraph "Data Stack"
            DB[(mysql)]
            ES[(elasticsearch)]
        end
        
        subgraph "Optional"
            CACHE[(redis)]
            MQ[(kafka)]
        end
    end
    
    OM --> DB
    OM --> ES
    OM --> CACHE
    ING --> MQ
    ING --> OM
    
    style OM fill:#e1f5e1
    style DB fill:#e6f3ff
    style ES fill:#fff4e6
```

### Kubernetes Deployment

```mermaid
flowchart TB
    subgraph "Kubernetes Cluster"
        subgraph "Ingress"
            ING[Ingress Controller<br/>nginx/traefik]
        end
        
        subgraph "Application Pods"
            API1[OM Server Pod 1]
            API2[OM Server Pod 2]
            API3[OM Server Pod 3]
        end
        
        subgraph "Stateful Sets"
            DB[MySQL StatefulSet]
            ES[Elasticsearch StatefulSet]
        end
        
        subgraph "Jobs"
            CRON[CronJobs<br/>Ingestion Tasks]
        end
    end
    
    ING --> API1
    ING --> API2
    ING --> API3
    
    API1 --> DB
    API2 --> DB
    API3 --> DB
    
    API1 --> ES
    API2 --> ES
    API3 --> ES
    
    CRON --> API1
    
    style ING fill:#e6f3ff
    style API1 fill:#e1f5e1
    style API2 fill:#e1f5e1
    style API3 fill:#e1f5e1
```

**Kubernetes Resources**:
- **Deployments**: API server (3+ replicas)
- **StatefulSets**: MySQL, Elasticsearch
- **Services**: LoadBalancer/ClusterIP
- **ConfigMaps**: Configuration
- **Secrets**: Credentials
- **PersistentVolumes**: Data storage
- **CronJobs**: Scheduled ingestion

---

## Scalability & Performance

### Horizontal Scaling

```mermaid
flowchart LR
    A[Load Balancer] --> B[API Server 1]
    A --> C[API Server 2]
    A --> D[API Server 3]
    
    B --> E[(MySQL Primary)]
    C --> E
    D --> E
    
    E --> F[(MySQL Replica 1)]
    E --> G[(MySQL Replica 2)]
    
    B --> H[ES Node 1]
    C --> I[ES Node 2]
    D --> J[ES Node 3]
    
    style A fill:#e6f3ff
    style E fill:#e1f5e1
```

**Scaling Strategy**:
- **API Server**: Stateless, scale to N instances
- **MySQL**: Primary-replica, read replicas for scaling
- **Elasticsearch**: Shard distribution, scale nodes
- **Redis**: Cluster mode for caching
- **Ingestion**: Parallel workers

### Performance Optimization

**Caching Layers**:
1. **Application Cache** (Redis)
   - User sessions
   - Frequently accessed entities
   - Search results
   - TTL: 1-60 minutes

2. **Database Query Cache**
   - Prepared statements
   - Connection pooling

3. **CDN Cache** (CloudFront/CloudFlare)
   - Static assets
   - UI resources

**Database Optimization**:
- **Indexes** on FQN, type, timestamps
- **Partitioning** by date/type
- **Query optimization** with EXPLAIN
- **Connection pooling** (HikariCP)

**Search Optimization**:
- **Index optimization** for query patterns
- **Aggregation caching**
- **Query result pagination**

---

## High Availability

```mermaid
flowchart TB
    subgraph "Region 1 - Primary"
        LB1[Load Balancer]
        API1A[API Server]
        API1B[API Server]
        DB1[MySQL Primary]
        ES1[ES Cluster]
    end
    
    subgraph "Region 2 - DR"
        LB2[Load Balancer]
        API2A[API Server]
        API2B[API Server]
        DB2[MySQL Replica]
        ES2[ES Cluster]
    end
    
    LB1 --> API1A
    LB1 --> API1B
    API1A --> DB1
    API1B --> DB1
    API1A --> ES1
    API1B --> ES1
    
    DB1 -.Replication.-> DB2
    ES1 -.Cross-Region Replication.-> ES2
    
    LB2 --> API2A
    LB2 --> API2B
    API2A --> DB2
    API2B --> DB2
    API2A --> ES2
    API2B --> ES2
    
    style DB1 fill:#e1f5e1
    style DB2 fill:#e6f3ff
```

**HA Features**:
- **Multi-region deployment**
- **Active-passive failover**
- **Database replication**
- **Elasticsearch cross-region replication**
- **Health checks and auto-recovery**
- **Backup and restore**

---

## Monitoring & Observability

```mermaid
flowchart LR
    subgraph "Metrics"
        A[Prometheus]
        B[Grafana Dashboards]
    end
    
    subgraph "Logging"
        C[Application Logs]
        D[ELK Stack]
    end
    
    subgraph "Tracing"
        E[Jaeger/Zipkin]
        F[Distributed Tracing]
    end
    
    subgraph "Alerting"
        G[PagerDuty]
        H[Slack]
        I[Email]
    end
    
    A --> B
    C --> D
    E --> F
    B --> G
    B --> H
    B --> I
    
    style A fill:#e6f3ff
    style D fill:#e1f5e1
    style F fill:#fff4e6
```

**Monitoring Stack**:
- **Metrics**: Prometheus + Grafana
- **Logging**: ELK (Elasticsearch, Logstash, Kibana)
- **Tracing**: Jaeger or Zipkin
- **Alerting**: PagerDuty, Slack, Email

**Key Metrics**:
- API response times
- Database query performance
- Search latency
- Ingestion throughput
- Error rates
- Resource utilization

---

## Technology Stack Summary

| Component | Technology | Version |
|-----------|-----------|---------|
| **Backend** | Java Spring Boot | 2.7+ |
| **Frontend** | React + TypeScript | 18+ |
| **Database** | MySQL / PostgreSQL | 8+ / 14+ |
| **Search** | Elasticsearch / OpenSearch | 7.10+ / 1.3+ |
| **Cache** | Redis | 6+ |
| **Message Queue** | Kafka (optional) | 3+ |
| **Ingestion** | Python | 3.8+ |
| **Container** | Docker | 20+ |
| **Orchestration** | Kubernetes | 1.23+ |
| **Monitoring** | Prometheus + Grafana | Latest |
| **API Gateway** | Kong / NGINX | Latest |

---

## Design Principles

### 1. **API-First Architecture**
- Everything accessible via REST API
- OpenAPI 3.0 specification
- Versioned APIs for backwards compatibility

### 2. **Cloud-Native Design**
- Containerized deployment
- Horizontal scalability
- Resilience and fault tolerance

### 3. **Standards-Based**
- JSON Schema for metadata models
- OpenAPI for API specification
- OAuth2/OIDC for authentication

### 4. **Extensibility**
- Plugin architecture for connectors
- Custom entity types
- Extensible workflow engine

### 5. **Performance**
- Caching at multiple layers
- Async processing for heavy operations
- Efficient indexing and search

---

## Security Considerations

### Network Security
- **TLS/SSL** for all communications
- **VPC** isolation in cloud
- **Firewall rules** for access control
- **Private endpoints** for databases

### Application Security
- **Input validation** on all endpoints
- **SQL injection prevention**
- **XSS protection**
- **CSRF tokens**
- **Rate limiting**

### Data Security
- **Encryption at rest** (AES-256)
- **Encryption in transit** (TLS 1.3)
- **PII detection and masking**
- **Audit logging**
- **Data retention policies**

---

## Best Practices

### Deployment
✅ Use managed services for MySQL and Elasticsearch  
✅ Enable auto-scaling for API servers  
✅ Set up monitoring and alerting  
✅ Configure regular backups  
✅ Use infrastructure as code (Terraform/Helm)

### Performance
✅ Enable Redis caching  
✅ Optimize database indexes  
✅ Use CDN for static assets  
✅ Configure connection pooling  
✅ Monitor and tune JVM settings

### Security
✅ Enable HTTPS/TLS everywhere  
✅ Use strong authentication (MFA)  
✅ Implement least-privilege access  
✅ Regular security audits  
✅ Keep dependencies updated

---

## References

- **Official Documentation**: https://docs.open-metadata.org/deployment
- **Architecture Guide**: https://docs.open-metadata.org/developers/architecture
- **GitHub**: https://github.com/open-metadata/OpenMetadata
- **Community Slack**: https://slack.open-metadata.org/

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
