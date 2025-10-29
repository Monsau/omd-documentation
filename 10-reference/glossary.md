# OpenMetadata - Glossary

## Core Concepts

### **Metadata**
Information about data - describing its structure, lineage, quality, and usage. OpenMetadata manages technical, business, and operational metadata.

### **Entity**
A metadata object in OpenMetadata (e.g., Table, Dashboard, Pipeline, Topic). Each entity has properties, relationships, and can be tagged and documented.

### **Fully Qualified Name (FQN)**
Unique identifier for an entity following the pattern: `service.database.schema.table` (e.g., `snowflake.sales_db.public.customers`)

---

## Data Assets

### **Table**
A structured dataset with rows and columns. Can represent database tables, data warehouse tables, or data lake tables.

### **Dashboard**
A visual representation of data, typically from BI tools like Tableau, PowerBI, or Looker.

### **Pipeline**
An automated workflow for data processing, orchestrated by tools like Airflow, Dagster, or Prefect.

### **Topic**
A message stream in messaging systems like Kafka, Pulsar, or RabbitMQ.

### **ML Model**
A machine learning model with associated metadata, training data, and performance metrics.

### **Container**
A logical grouping for data objects, such as S3 buckets or Azure containers.

---

## Data Lineage

### **Lineage**
The end-to-end journey of data from source to destination, showing transformations and dependencies.

### **Upstream**
Data sources that feed into a particular dataset or process.

### **Downstream**
Data assets and processes that depend on a particular dataset.

### **Column-Level Lineage**
Detailed lineage showing how individual columns are transformed and used across the data pipeline.

---

## Data Quality

### **Data Quality Test**
An automated check to verify data meets defined standards (e.g., completeness, accuracy, consistency).

### **Test Suite**
A collection of related data quality tests for an entity or set of entities.

### **Test Case**
A specific instance of a test with defined parameters and expected outcomes.

### **Data Profiling**
Statistical analysis of data to understand its characteristics (distribution, nulls, unique values, etc.).

---

## Governance

### **Tag**
A label attached to metadata entities for classification and discovery. Tags can be organized into categories.

### **Tag Category**
A grouping of related tags (e.g., PII, Tier, Domain).

### **Glossary**
A collection of business terms with standardized definitions for organization-wide consistency.

### **Glossary Term**
A specific business concept with definition, synonyms, and relationships to other terms.

### **Policy**
A rule defining access controls or governance requirements.

---

## Security & Access Control

### **RBAC (Role-Based Access Control)**
Access control model based on user roles (e.g., Admin, Data Steward, Data Consumer).

### **ABAC (Attribute-Based Access Control)**
Access control model using attributes of users, resources, and environment.

### **PII (Personally Identifiable Information)**
Data that can identify an individual person (e.g., email, SSN, phone number).

### **Masking**
Obscuring sensitive data to protect privacy while preserving usability.

---

## Integration & Ingestion

### **Connector**
A plugin that extracts metadata from external data sources.

### **Source**
The external system from which metadata is extracted (e.g., Snowflake, PostgreSQL).

### **Service**
A configured connection to an external system (e.g., "production-snowflake").

### **Ingestion Pipeline**
An automated workflow to extract and update metadata from sources.

### **Workflow**
A scheduled or manual execution of an ingestion pipeline.

---

## Collaboration

### **Owner**
A user or team responsible for a data asset (can be technical or business owner).

### **Team**
A group of users organized by function, department, or project.

### **User**
An individual with access to OpenMetadata.

### **Conversation**
A discussion thread attached to an entity for collaboration.

### **Activity Feed**
A timeline of changes and activities for entities.

### **Announcement**
A broadcast message to notify users about important updates.

---

## Technical Terms

### **API**
Application Programming Interface - RESTful endpoints for programmatic access to OpenMetadata.

### **SDK**
Software Development Kit - Python library for interacting with OpenMetadata API.

### **JSON Schema**
Standard for defining the structure of metadata entities.

### **OpenAPI**
Specification for documenting REST APIs.

### **Webhook**
HTTP callback triggered by events in OpenMetadata.

### **JWT (JSON Web Token)**
Secure token format for authentication and authorization.

---

## Architecture Terms

### **Metadata Store**
Database (MySQL/PostgreSQL) storing all metadata.

### **Search Index**
Elasticsearch index for fast metadata search and discovery.

### **Ingestion Framework**
Python-based system for extracting metadata from sources.

### **API Server**
Spring Boot application serving REST API requests.

---

## Data Observability

### **Data Observability**
Monitoring and understanding the health, quality, and reliability of data.

### **Incident**
An issue detected in data quality, freshness, or availability.

### **SLA (Service Level Agreement)**
Defined targets for data quality, freshness, and availability.

### **Freshness**
How recent the data is (time since last update).

### **Volume**
The size or row count of a dataset.

---

## Advanced Concepts

### **Schema Evolution**
Tracking changes to data structures over time.

### **Versioning**
Maintaining history of metadata changes.

### **Custom Property**
User-defined metadata fields extending standard entities.

### **Extension**
Additional metadata attached to entities beyond the standard schema.

### **Classification**
Automated tagging based on data patterns (e.g., PII detection).

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
