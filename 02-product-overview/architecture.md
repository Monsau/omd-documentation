# OpenMetadata - Architecture Overview

## Introduction

OpenMetadata is built on a modern, cloud-native architecture designed for scalability, performance, and extensibility. This overview provides a high-level understanding of the system components and how they work together.

---

## High-Level Architecture

```mermaid
flowchart TB
    subgraph "User Layer"
        UI[Web UI]
        CLI[CLI Tools]
        API[REST API Clients]
    end
    
    subgraph "API & Services"
        GATEWAY[API Gateway]
        METADATA[Metadata Service]
        INGESTION[Ingestion Service]
        SEARCH[Search Service]
        QUALITY[Quality Service]
    end
    
    subgraph "Storage Layer"
        DB[(Metadata Store<br/>MySQL/PostgreSQL)]
        ES[(Search Index<br/>Elasticsearch)]
        CACHE[(Cache<br/>Redis)]
    end
    
    subgraph "External Systems"
        SOURCES[Data Sources<br/>100+ Connectors]
    end
    
    UI --> GATEWAY
    CLI --> GATEWAY
    API --> GATEWAY
    
    GATEWAY --> METADATA
    GATEWAY --> SEARCH
    GATEWAY --> QUALITY
    
    METADATA --> DB
    METADATA --> CACHE
    SEARCH --> ES
    INGESTION --> SOURCES
    INGESTION --> METADATA
    
    style GATEWAY fill:#e6f3ff
    style DB fill:#e1f5e1
    style ES fill:#fff4e6
```

---

## Core Components

### 1. **Web UI (Frontend)**
- **Technology**: React + TypeScript
- **Purpose**: User interface for metadata management
- **Features**:
  - Data discovery and search
  - Metadata editing and enrichment
  - Lineage visualization
  - Data quality dashboards
  - Collaboration tools

### 2. **API Gateway**
- **Technology**: Spring Boot
- **Purpose**: Single entry point for all API requests
- **Responsibilities**:
  - Request routing
  - Authentication/authorization
  - Rate limiting
  - API versioning

### 3. **Metadata Service**
- **Technology**: Java Spring Boot
- **Purpose**: Core metadata management
- **Features**:
  - CRUD operations for entities
  - Relationship management
  - Version control
  - Event publishing
  - Schema validation

### 4. **Ingestion Framework**
- **Technology**: Python
- **Purpose**: Extract metadata from external sources
- **Components**:
  - 100+ source connectors
  - Processors (transformations, enrichment)
  - Sinks (write to OpenMetadata)
  - Workflow orchestration

### 5. **Search Service**
- **Technology**: Elasticsearch/OpenSearch
- **Purpose**: Fast metadata search and discovery
- **Features**:
  - Full-text search
  - Faceted filtering
  - Auto-suggestions
  - Relevance scoring

### 6. **Data Quality Service**
- **Purpose**: Data quality monitoring
- **Features**:
  - Test definitions
  - Automated test execution
  - Quality dashboards
  - Alerting

---

## Data Flow

```mermaid
sequenceDiagram
    participant Source as Data Source
    participant Ingestion as Ingestion Service
    participant API as Metadata API
    participant DB as Database
    participant ES as Elasticsearch
    participant UI as Web UI
    
    Source->>Ingestion: Connect & Extract
    Ingestion->>API: POST /api/v1/tables
    API->>DB: Store Metadata
    API->>ES: Index for Search
    API-->>Ingestion: Success
    
    UI->>API: Search "customer"
    API->>ES: Query
    ES-->>API: Results
    API-->>UI: Display Results
    
    UI->>API: GET Table Details
    API->>DB: Fetch
    DB-->>API: Table Data
    API-->>UI: Render Table
```

---

## Deployment Models

### 1. **Docker Compose** (Development/Testing)
```mermaid
flowchart LR
    A[Docker Host] --> B[openmetadata-server]
    A --> C[mysql]
    A --> D[elasticsearch]
    A --> E[ingestion]
    
    style A fill:#e6f3ff
```

### 2. **Kubernetes** (Production)
```mermaid
flowchart TB
    subgraph "Kubernetes Cluster"
        ING[Ingress]
        API1[API Pod 1]
        API2[API Pod 2]
        API3[API Pod 3]
        DB[MySQL StatefulSet]
        ES[Elasticsearch Cluster]
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
    
    style ING fill:#e6f3ff
    style DB fill:#e1f5e1
    style ES fill:#fff4e6
```

### 3. **Cloud SaaS** (Managed)
- Fully managed by OpenMetadata
- No infrastructure management
- Automatic updates
- Enterprise support

---

## Security Architecture

```mermaid
flowchart TB
    A[User] --> B[Authentication<br/>LDAP/OAuth/SAML]
    B --> C[Authorization<br/>RBAC]
    C --> D[Resource Access]
    D --> E[Audit Logging]
    
    F[Encryption at Rest] --> G[Database]
    H[Encryption in Transit] --> I[TLS/SSL]
    
    style B fill:#ffe6e6
    style C fill:#e6f3ff
    style E fill:#fff4e6
```

**Security Features**:
- Multi-factor authentication
- Role-based access control
- Column-level security
- PII detection and masking
- Comprehensive audit trails
- Encryption at rest and in transit

---

## Scalability

### Horizontal Scaling
- **API Servers**: Scale to N instances
- **Elasticsearch**: Add more nodes
- **MySQL**: Primary-replica replication
- **Caching**: Redis cluster mode

### Performance Optimization
- Multi-level caching (Redis, application, CDN)
- Database indexing and partitioning
- Elasticsearch sharding
- Connection pooling

---

## Integration Points

```mermaid
mindmap
  root((OpenMetadata))
    Data Sources
      Databases
      Data Warehouses
      Data Lakes
      BI Tools
    Pipeline Tools
      Airflow
      Dagster
      Prefect
    Messaging
      Kafka
      Pulsar
    Storage
      S3
      Azure Blob
      GCS
    Identity
      LDAP
      OAuth2
      SAML
```

---

## Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | React, TypeScript | User interface |
| **Backend** | Spring Boot, Java | API server |
| **Ingestion** | Python | Metadata extraction |
| **Database** | MySQL/PostgreSQL | Metadata storage |
| **Search** | Elasticsearch | Search and indexing |
| **Cache** | Redis | Performance |
| **Container** | Docker | Packaging |
| **Orchestration** | Kubernetes | Deployment |

---

## Key Design Principles

### 1. **API-First**
All functionality accessible via REST API

### 2. **Cloud-Native**
Built for modern cloud infrastructure

### 3. **Extensible**
Plugin architecture for custom connectors

### 4. **Standards-Based**
Uses industry standards (OpenAPI, JSON Schema)

### 5. **Open Source**
Apache 2.0 license, community-driven

---

## Next Steps

📖 **For Detailed Architecture**: See [Detailed Architecture](../03-technical-deep-dive/architecture-detailed.md)

🚀 **For Deployment**: See [Deployment Options](../04-deployment-operations/deployment-options.md)

🔒 **For Security**: See [Security & Compliance](../03-technical-deep-dive/security-compliance.md)

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
