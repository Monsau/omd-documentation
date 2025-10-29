# OpenMetadata - Metadata Model & Schema

## Overview

OpenMetadata's metadata model is the foundation of the platform. It defines the structure, relationships, and properties of all metadata entities. This document provides a comprehensive guide to the metadata model, schema definitions, and best practices.

---

## Core Metadata Model

```mermaid
erDiagram
    DATA_ASSET ||--o{ TABLE : contains
    DATA_ASSET ||--o{ DASHBOARD : contains
    DATA_ASSET ||--o{ PIPELINE : contains
    DATA_ASSET ||--o{ TOPIC : contains
    DATA_ASSET ||--o{ ML_MODEL : contains
    
    TABLE ||--o{ COLUMN : has
    TABLE ||--o{ TAG : tagged_with
    TABLE ||--|{ LINEAGE : participates_in
    TABLE ||--o{ DATA_QUALITY : tested_by
    
    COLUMN ||--o{ TAG : tagged_with
    COLUMN ||--|{ LINEAGE : participates_in
    
    DASHBOARD ||--o{ CHART : contains
    DASHBOARD ||--o{ TAG : tagged_with
    DASHBOARD ||--|{ LINEAGE : uses
    
    PIPELINE ||--o{ TASK : contains
    PIPELINE ||--o{ TAG : tagged_with
    PIPELINE ||--|{ LINEAGE : produces
    
    USER ||--o{ TEAM : member_of
    USER ||--o{ OWNERSHIP : owns
    TEAM ||--o{ OWNERSHIP : owns
    
    OWNERSHIP ||--|| DATA_ASSET : applies_to
    
    GLOSSARY ||--o{ GLOSSARY_TERM : contains
    GLOSSARY_TERM ||--o{ TAG : used_as
```

---

## Entity Hierarchy

```mermaid
flowchart TB
    ROOT[EntityInterface]
    
    ROOT --> DA[Data Assets]
    ROOT --> USR[Users & Teams]
    ROOT --> GOV[Governance]
    ROOT --> SVC[Services]
    
    DA --> TBL[Table]
    DA --> DASH[Dashboard]
    DA --> PIPE[Pipeline]
    DA --> TOP[Topic]
    DA --> ML[ML Model]
    DA --> CONT[Container]
    
    USR --> USER[User]
    USR --> TEAM[Team]
    USR --> ROLE[Role]
    
    GOV --> TAG[Tag]
    GOV --> GLOSS[Glossary]
    GOV --> TERM[Glossary Term]
    GOV --> POL[Policy]
    
    SVC --> DBSVC[Database Service]
    SVC --> DASHSVC[Dashboard Service]
    SVC --> PIPESVC[Pipeline Service]
    SVC --> MSGSVC[Messaging Service]
    
    style ROOT fill:#e6f3ff
    style DA fill:#e1f5e1
    style GOV fill:#fff4e6
```

---

## Entity Types

### 1. **Data Assets**

#### Table Entity

```json
{
  "id": "uuid",
  "name": "customer_orders",
  "fullyQualifiedName": "snowflake.sales_db.public.customer_orders",
  "displayName": "Customer Orders",
  "description": "Contains all customer order transactions",
  "tableType": "Regular",
  "columns": [
    {
      "name": "order_id",
      "dataType": "BIGINT",
      "dataTypeDisplay": "bigint",
      "description": "Unique order identifier",
      "constraint": "PRIMARY_KEY",
      "tags": []
    },
    {
      "name": "customer_email",
      "dataType": "VARCHAR",
      "dataLength": 255,
      "description": "Customer email address",
      "tags": [
        {
          "tagFQN": "PII.Email",
          "source": "Classification"
        }
      ]
    }
  ],
  "database": {
    "id": "database-uuid",
    "type": "database",
    "name": "sales_db",
    "fullyQualifiedName": "snowflake.sales_db"
  },
  "databaseSchema": {
    "id": "schema-uuid",
    "type": "databaseSchema",
    "name": "public"
  },
  "service": {
    "id": "service-uuid",
    "type": "databaseService",
    "name": "snowflake"
  },
  "owner": {
    "id": "user-uuid",
    "type": "user",
    "name": "john.doe"
  },
  "tags": [
    {
      "tagFQN": "Tier.Tier1",
      "source": "Classification"
    }
  ],
  "dataModel": {
    "modelType": "DBT",
    "description": "dbt model definition"
  },
  "version": 1.2,
  "updatedAt": 1698624000000,
  "updatedBy": "admin"
}
```

#### Dashboard Entity

```json
{
  "id": "uuid",
  "name": "sales_overview",
  "fullyQualifiedName": "tableau.sales_analytics.sales_overview",
  "displayName": "Sales Overview Dashboard",
  "description": "Executive dashboard showing sales metrics",
  "dashboardType": "Dashboard",
  "dashboardUrl": "https://tableau.company.com/sales_overview",
  "charts": [
    {
      "id": "chart-uuid",
      "type": "chart",
      "name": "revenue_by_region",
      "chartType": "Bar"
    }
  ],
  "service": {
    "id": "service-uuid",
    "type": "dashboardService",
    "name": "tableau"
  },
  "owner": {
    "id": "team-uuid",
    "type": "team",
    "name": "analytics_team"
  },
  "tags": [],
  "version": 1.0
}
```

#### Pipeline Entity

```json
{
  "id": "uuid",
  "name": "daily_etl_pipeline",
  "fullyQualifiedName": "airflow.prod.daily_etl_pipeline",
  "displayName": "Daily ETL Pipeline",
  "description": "Processes daily sales data from sources to warehouse",
  "pipelineType": "ETL",
  "pipelineUrl": "https://airflow.company.com/daily_etl",
  "tasks": [
    {
      "name": "extract_sales_data",
      "displayName": "Extract Sales Data",
      "taskType": "SQL",
      "downstreamTasks": ["transform_sales_data"]
    },
    {
      "name": "transform_sales_data",
      "displayName": "Transform Sales Data",
      "taskType": "Python",
      "downstreamTasks": ["load_to_warehouse"]
    },
    {
      "name": "load_to_warehouse",
      "displayName": "Load to Warehouse",
      "taskType": "SQL",
      "downstreamTasks": []
    }
  ],
  "service": {
    "id": "service-uuid",
    "type": "pipelineService",
    "name": "airflow"
  },
  "owner": {
    "id": "team-uuid",
    "type": "team",
    "name": "data_engineering"
  }
}
```

---

### 2. **Governance Entities**

#### Tag Category & Tags

```mermaid
flowchart TB
    subgraph "Tag Categories"
        PII[PII Category]
        TIER[Tier Category]
        CLASS[Classification Category]
    end
    
    PII --> EMAIL[Email Tag]
    PII --> SSN[SSN Tag]
    PII --> PHONE[Phone Tag]
    
    TIER --> T1[Tier1 Tag]
    TIER --> T2[Tier2 Tag]
    TIER --> T3[Tier3 Tag]
    
    CLASS --> CONF[Confidential Tag]
    CLASS --> PUBLIC[Public Tag]
    CLASS --> INTERNAL[Internal Tag]
    
    style PII fill:#ffe6e6
    style TIER fill:#e6f3ff
    style CLASS fill:#fff4e6
```

**Tag Structure**:
```json
{
  "id": "uuid",
  "name": "PII",
  "displayName": "Personally Identifiable Information",
  "description": "Tags for identifying PII data",
  "categoryType": "Classification",
  "children": [
    {
      "id": "tag-uuid",
      "name": "Email",
      "displayName": "Email Address",
      "description": "Email address field",
      "style": {
        "color": "#FF0000",
        "iconURL": "email-icon.svg"
      }
    },
    {
      "id": "tag-uuid-2",
      "name": "SSN",
      "displayName": "Social Security Number",
      "description": "US Social Security Number"
    }
  ]
}
```

#### Glossary & Terms

```json
{
  "id": "uuid",
  "name": "business_glossary",
  "displayName": "Business Glossary",
  "description": "Company-wide business terms",
  "owner": {
    "id": "team-uuid",
    "type": "team",
    "name": "data_governance"
  },
  "terms": [
    {
      "id": "term-uuid",
      "name": "customer_lifetime_value",
      "displayName": "Customer Lifetime Value (CLV)",
      "description": "Total revenue expected from a customer over their lifetime",
      "synonyms": ["CLV", "LTV", "Lifetime Value"],
      "relatedTerms": [
        {
          "id": "related-term-uuid",
          "name": "customer_acquisition_cost"
        }
      ],
      "references": [
        {
          "name": "CLV Calculation Guide",
          "endpoint": "https://wiki.company.com/clv"
        }
      ],
      "tags": [
        {
          "tagFQN": "Tier.Tier1"
        }
      ]
    }
  ]
}
```

---

### 3. **User & Team Entities**

```mermaid
flowchart LR
    subgraph "Organization"
        ORG[Organization]
    end
    
    subgraph "Teams"
        ENG[Engineering Team]
        DATA[Data Team]
        BI[BI Team]
    end
    
    subgraph "Users"
        U1[John Doe<br/>Data Engineer]
        U2[Jane Smith<br/>Data Analyst]
        U3[Bob Wilson<br/>BI Developer]
    end
    
    subgraph "Roles"
        ADMIN[Admin Role]
        OWNER[Data Owner Role]
        VIEWER[Data Viewer Role]
    end
    
    ORG --> ENG
    ORG --> DATA
    ORG --> BI
    
    ENG --> U1
    DATA --> U2
    BI --> U3
    
    U1 -.has.-> ADMIN
    U2 -.has.-> OWNER
    U3 -.has.-> VIEWER
    
    style ORG fill:#e6f3ff
    style ADMIN fill:#ffe6e6
```

**User Schema**:
```json
{
  "id": "uuid",
  "name": "john.doe",
  "email": "john.doe@company.com",
  "displayName": "John Doe",
  "description": "Senior Data Engineer",
  "isAdmin": false,
  "isBot": false,
  "profile": {
    "images": {
      "image": "profile-pic-url"
    }
  },
  "teams": [
    {
      "id": "team-uuid",
      "type": "team",
      "name": "data_engineering"
    }
  ],
  "roles": [
    {
      "id": "role-uuid",
      "type": "role",
      "name": "DataOwner"
    }
  ],
  "inheritedRoles": [
    {
      "id": "role-uuid-2",
      "type": "role",
      "name": "DataViewer"
    }
  ]
}
```

**Team Schema**:
```json
{
  "id": "uuid",
  "name": "data_engineering",
  "displayName": "Data Engineering Team",
  "description": "Team responsible for data pipelines",
  "teamType": "Department",
  "parents": [
    {
      "id": "parent-team-uuid",
      "type": "team",
      "name": "engineering"
    }
  ],
  "users": [
    {
      "id": "user-uuid",
      "type": "user",
      "name": "john.doe"
    }
  ],
  "defaultRoles": [
    {
      "id": "role-uuid",
      "type": "role",
      "name": "DataEngineer"
    }
  ],
  "owns": [
    {
      "id": "table-uuid",
      "type": "table",
      "name": "customer_orders"
    }
  ]
}
```

---

## Relationships

### Entity Relationships

```mermaid
flowchart LR
    A[Table: customer_orders] -->|contains| B[Column: order_id]
    A -->|contains| C[Column: customer_id]
    
    A -->|upstream| D[Table: raw_orders]
    A -->|downstream| E[Table: orders_summary]
    
    A -->|tagged_with| F[Tag: Tier.Tier1]
    B -->|tagged_with| G[Tag: PII.None]
    C -->|tagged_with| H[Tag: PII.Indirect]
    
    A -->|owned_by| I[Team: data_engineering]
    A -->|used_by| J[Dashboard: sales_overview]
    
    K[Pipeline: daily_etl] -->|produces| A
    
    style A fill:#e1f5e1
    style F fill:#e6f3ff
    style K fill:#fff4e6
```

### Lineage Relationships

```mermaid
flowchart LR
    subgraph "Source"
        S1[(MySQL<br/>app_db.orders)]
    end
    
    subgraph "Staging"
        ST1[(Snowflake<br/>staging.raw_orders)]
    end
    
    subgraph "Transform"
        T1[dbt Model<br/>cleaned_orders]
        T2[dbt Model<br/>enriched_orders]
    end
    
    subgraph "Analytics"
        A1[(Snowflake<br/>analytics.orders_fact)]
        A2[(Snowflake<br/>analytics.customer_360)]
    end
    
    subgraph "Consumption"
        D1[Tableau Dashboard<br/>Sales Overview]
    end
    
    S1 -->|Airbyte| ST1
    ST1 -->|dbt| T1
    T1 -->|dbt| T2
    T2 -->|dbt| A1
    T2 -->|dbt| A2
    A1 --> D1
    A2 --> D1
    
    style S1 fill:#e6f3ff
    style T1 fill:#e1f5e1
    style T2 fill:#e1f5e1
    style A1 fill:#fff4e6
    style D1 fill:#ffe6e6
```

---

## Schema Definitions

### JSON Schema Structure

All entities follow JSON Schema Draft-07 standard:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://open-metadata.org/schema/entity/data/table.json",
  "title": "Table",
  "description": "A table entity represents a structured dataset",
  "type": "object",
  "properties": {
    "id": {
      "description": "Unique identifier",
      "$ref": "../../type/basic.json#/definitions/uuid"
    },
    "name": {
      "description": "Name of the table",
      "$ref": "../../type/basic.json#/definitions/entityName"
    },
    "fullyQualifiedName": {
      "description": "Fully qualified name",
      "type": "string"
    },
    "columns": {
      "description": "Columns in this table",
      "type": "array",
      "items": {
        "$ref": "#/definitions/column"
      }
    }
  },
  "required": ["id", "name", "columns"],
  "additionalProperties": false
}
```

### Custom Properties

OpenMetadata supports custom properties via **Entity Extensions**:

```json
{
  "entityType": "table",
  "customProperties": [
    {
      "name": "dataRetentionDays",
      "description": "Number of days data is retained",
      "propertyType": {
        "id": "integer",
        "type": "integer"
      }
    },
    {
      "name": "costCenter",
      "description": "Cost center for billing",
      "propertyType": {
        "id": "string",
        "type": "string"
      }
    },
    {
      "name": "isRegulatoryData",
      "description": "Subject to regulatory requirements",
      "propertyType": {
        "id": "boolean",
        "type": "boolean"
      }
    }
  ]
}
```

**Usage**:
```json
{
  "id": "table-uuid",
  "name": "customer_orders",
  "extension": {
    "dataRetentionDays": 2555,
    "costCenter": "CC-1234",
    "isRegulatoryData": true
  }
}
```

---

## Data Types

### Column Data Types

```mermaid
mindmap
  root((Data Types))
    Numeric
      TINYINT
      SMALLINT
      INT/INTEGER
      BIGINT
      FLOAT
      DOUBLE
      DECIMAL/NUMERIC
    String
      CHAR
      VARCHAR
      TEXT
      LONGTEXT
      BINARY
    Date/Time
      DATE
      TIME
      DATETIME
      TIMESTAMP
      INTERVAL
    Boolean
      BOOLEAN
      BIT
    Complex
      ARRAY
      STRUCT
      MAP
      JSON
      UNION
    Other
      UUID
      ENUM
      GEOGRAPHY
      GEOMETRY
```

### Type Mappings

| Generic Type | MySQL | PostgreSQL | Snowflake | BigQuery |
|--------------|-------|------------|-----------|----------|
| **String** | VARCHAR | VARCHAR | VARCHAR | STRING |
| **Integer** | INT | INTEGER | NUMBER | INT64 |
| **BigInt** | BIGINT | BIGINT | NUMBER | INT64 |
| **Float** | FLOAT | REAL | FLOAT | FLOAT64 |
| **Decimal** | DECIMAL | NUMERIC | NUMBER | NUMERIC |
| **Boolean** | BOOLEAN | BOOLEAN | BOOLEAN | BOOL |
| **Date** | DATE | DATE | DATE | DATE |
| **Timestamp** | TIMESTAMP | TIMESTAMP | TIMESTAMP_NTZ | TIMESTAMP |
| **JSON** | JSON | JSONB | VARIANT | JSON |
| **Array** | - | ARRAY | ARRAY | ARRAY |

---

## Versioning

### Entity Versioning

```mermaid
sequenceDiagram
    participant User
    participant API
    participant VersionControl
    participant Database
    
    User->>API: Update Table Schema
    API->>VersionControl: Check Current Version (1.0)
    VersionControl->>Database: Get Entity
    Database-->>VersionControl: Return Entity v1.0
    
    VersionControl->>VersionControl: Create Version 1.1
    VersionControl->>VersionControl: Calculate Diff
    VersionControl->>Database: Save v1.1 + Changelog
    Database-->>API: Success
    API-->>User: Entity Updated to v1.1
```

**Version Format**: `major.minor`
- **Major version**: Breaking changes (e.g., column deletion)
- **Minor version**: Non-breaking changes (e.g., add column, update description)

**Changelog**:
```json
{
  "entityType": "table",
  "entityId": "uuid",
  "version": 1.1,
  "previousVersion": 1.0,
  "changeDescription": {
    "fieldsAdded": [
      {
        "name": "columns",
        "newValue": {
          "name": "created_at",
          "dataType": "TIMESTAMP"
        }
      }
    ],
    "fieldsUpdated": [
      {
        "name": "description",
        "oldValue": "Customer orders",
        "newValue": "Customer orders with timestamps"
      }
    ],
    "fieldsDeleted": []
  },
  "updatedAt": 1698624000000,
  "updatedBy": "admin"
}
```

---

## Best Practices

### Naming Conventions

✅ **Fully Qualified Names (FQN)**:
```
service.database.schema.table
service.database.schema.table.column
```

Examples:
- `snowflake.sales_db.public.customer_orders`
- `snowflake.sales_db.public.customer_orders.order_id`
- `tableau.sales_analytics.sales_overview`

✅ **Entity Names**:
- Use lowercase with underscores
- Be descriptive and consistent
- Avoid abbreviations unless standard

❌ **Avoid**:
- Special characters (except underscore)
- Spaces in names
- Reserved keywords

### Description Guidelines

✅ **Good Descriptions**:
```markdown
Customer orders table containing all e-commerce transactions.
Includes order details, customer information, and payment status.
Updated in real-time via CDC from MySQL production database.
```

❌ **Poor Descriptions**:
```markdown
orders table
```

### Tagging Strategy

```mermaid
flowchart TB
    A[Define Tag Categories] --> B[Create Tags]
    B --> C[Apply Automated Rules]
    C --> D[Manual Tag Assignment]
    D --> E[Tag Propagation]
    E --> F[Regular Audit]
    
    style A fill:#e6f3ff
    style E fill:#e1f5e1
```

**Tag Categories**:
1. **Data Sensitivity**: PII, Confidential, Public
2. **Data Tier**: Tier1, Tier2, Tier3
3. **Domain**: Finance, Marketing, Sales, HR
4. **Environment**: Production, Staging, Development
5. **Compliance**: GDPR, HIPAA, SOC2

### Ownership Model

```mermaid
flowchart LR
    A[Data Asset] --> B{Ownership Type}
    B -->|Technical Owner| C[Data Engineering Team]
    B -->|Business Owner| D[Product Team]
    B -->|Steward| E[Data Governance Team]
    
    C --> F[Maintains & Monitors]
    D --> G[Defines & Validates]
    E --> H[Audits & Enforces]
    
    style A fill:#e6f3ff
    style C fill:#e1f5e1
    style D fill:#fff4e6
    style E fill:#ffe6e6
```

---

## Advanced Concepts

### Custom Entity Types

Create custom entity types by extending base schemas:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "https://company.com/schema/entity/customDataProduct.json",
  "title": "Data Product",
  "description": "Custom entity for data products",
  "type": "object",
  "javaType": "org.openmetadata.schema.entity.data.DataProduct",
  "javaInterfaces": ["org.openmetadata.schema.EntityInterface"],
  "properties": {
    "id": { "$ref": "../../type/basic.json#/definitions/uuid" },
    "name": { "$ref": "../../type/basic.json#/definitions/entityName" },
    "productType": {
      "type": "string",
      "enum": ["API", "Dataset", "Report", "ML_Model"]
    },
    "tables": {
      "type": "array",
      "items": { "$ref": "../../type/entityReference.json" }
    },
    "sla": {
      "type": "object",
      "properties": {
        "availabilityTarget": { "type": "number" },
        "freshnessTarget": { "type": "string" }
      }
    }
  },
  "required": ["id", "name", "productType"]
}
```

### Event-Driven Updates

```mermaid
sequenceDiagram
    participant Source
    participant ChangeCapture
    participant EventBus
    participant Subscriber1
    participant Subscriber2
    
    Source->>ChangeCapture: Entity Updated
    ChangeCapture->>ChangeCapture: Create Change Event
    ChangeCapture->>EventBus: Publish Event
    
    EventBus->>Subscriber1: Notify (Webhook)
    EventBus->>Subscriber2: Notify (Search Indexer)
    
    Subscriber1-->>EventBus: Ack
    Subscriber2->>Subscriber2: Update Search Index
    Subscriber2-->>EventBus: Ack
```

**Event Payload**:
```json
{
  "eventId": "uuid",
  "eventType": "entityUpdated",
  "entityType": "table",
  "entityId": "entity-uuid",
  "entityFQN": "snowflake.sales_db.public.customer_orders",
  "previousVersion": 1.0,
  "currentVersion": 1.1,
  "changeDescription": {
    "fieldsAdded": [...],
    "fieldsUpdated": [...],
    "fieldsDeleted": [...]
  },
  "timestamp": 1698624000000,
  "userName": "admin"
}
```

---

## API Examples

### Create Table Entity

```python
from metadata.generated.schema.entity.data.table import Table
from metadata.generated.schema.type.entityReference import EntityReference

# Create table entity
table = Table(
    name="customer_orders",
    fullyQualifiedName="snowflake.sales_db.public.customer_orders",
    description="Customer orders table",
    columns=[
        Column(
            name="order_id",
            dataType="BIGINT",
            description="Unique order identifier",
            constraint="PRIMARY_KEY"
        ),
        Column(
            name="customer_email",
            dataType="VARCHAR",
            dataLength=255,
            description="Customer email",
            tags=[TagLabel(tagFQN="PII.Email")]
        )
    ],
    database=EntityReference(id="db-uuid", type="database"),
    databaseSchema=EntityReference(id="schema-uuid", type="databaseSchema"),
    service=EntityReference(id="service-uuid", type="databaseService")
)

# Create via API
metadata.create_or_update(table)
```

### Query with Relationships

```python
# Get table with all relationships
table = metadata.get_by_name(
    entity=Table,
    fqn="snowflake.sales_db.public.customer_orders",
    fields=["columns", "tags", "owner", "database", "usageSummary", "dataModel"]
)

# Access relationships
print(f"Database: {table.database.name}")
print(f"Owner: {table.owner.name}")
print(f"Tags: {[tag.tagFQN for tag in table.tags]}")
```

---

## References

- **Schema Repository**: https://github.com/open-metadata/OpenMetadata/tree/main/openmetadata-spec/src/main/resources/json/schema
- **API Documentation**: https://docs.open-metadata.org/sdk/python
- **JSON Schema**: https://json-schema.org/

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
