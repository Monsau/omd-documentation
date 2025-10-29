# OpenMetadata Connectors Overview - v1.10.3

## Introduction

OpenMetadata provides **100+ native connectors** to extract metadata from diverse data sources across your entire data ecosystem. This document provides a comprehensive overview of all available connectors.

---

## Connector Categories

| Category | Count | Examples |
|----------|-------|----------|
| **Databases** | 40+ | PostgreSQL, MySQL, Snowflake, BigQuery |
| **Pipelines** | 25+ | Airflow, dbt, Dagster, Fivetran |
| **Dashboards** | 20+ | Tableau, Power BI, Looker, Metabase |
| **Messaging** | 5+ | Kafka, Kinesis, Pulsar, Redpanda |
| **ML Models** | 5+ | MLflow, SageMaker, Vertex AI |
| **Storage** | 5+ | S3, GCS, Azure Blob, ADLS |
| **Search** | 2+ | Elasticsearch, OpenSearch |
| **API** | 1+ | REST API |
| **Metadata** | 3+ | Amundsen, Atlas, Alation |

**Total**: 100+ connectors

---

## Database & Data Warehouse Connectors

### Production-Ready (PROD)

#### Relational Databases

**PostgreSQL** 🟢
- Metadata: Tables, views, columns, constraints
- Lineage: Query log analysis
- Profiling: Statistics and samples
- Version: 9.6+

**MySQL** 🟢
- Metadata: Tables, views, procedures
- Lineage: Query log analysis
- Profiling: Full support
- Version: 5.7+

**Oracle** 🟢
- Metadata: Tables, views, procedures, packages
- Lineage: SQL parsing
- Profiling: Sampling support
- Version: 11g+

**SQL Server** 🟢
- Metadata: Tables, views, procedures
- Lineage: Query history
- Profiling: Full support
- Version: 2017+

**MariaDB** 🟢
- Metadata: Tables, views
- Lineage: Query logs
- Profiling: Supported
- Version: 10.3+

**IBM DB2** 🟢
- Metadata: Tables, views
- Lineage: Limited
- Profiling: Supported
- Version: 11.1+

#### Cloud Data Warehouses

**Snowflake** 🟢⭐
- Metadata: Tables, views, streams, stages
- Lineage: ACCESS_HISTORY views
- Profiling: Full support with sampling
- Usage: Query history tracking
- Cost: Warehouse usage metrics
- Special: Multi-cluster warehouse support

**Google BigQuery** 🟢⭐
- Metadata: Tables, views, datasets
- Lineage: INFORMATION_SCHEMA.JOBS
- Profiling: Full support
- Usage: Audit logs
- Cost: Query costs tracking
- Special: Partitioned table support

**Amazon Redshift** 🟢⭐
- Metadata: Tables, views, schemas
- Lineage: STL_QUERY logs
- Profiling: Sampling support
- Usage: System tables
- Special: Spectrum external tables

**Azure Synapse** 🟢
- Metadata: Tables, views, procedures
- Lineage: Query store
- Profiling: Supported
- Usage: DMVs
- Special: Dedicated/Serverless pools

**Databricks** 🟢⭐
- Metadata: Tables, views, Unity Catalog
- Lineage: Table lineage API
- Profiling: Full support
- Usage: Query history
- Special: Delta Lake support, Unity Catalog

#### NoSQL Databases

**MongoDB** 🟢
- Metadata: Databases, collections, documents
- Schema: Inferred from documents
- Profiling: Sampling support
- Version: 4.0+

**Cassandra** 🟡
- Metadata: Keyspaces, tables, columns
- Schema: CQL definitions
- Profiling: Limited
- Version: 3.11+

**Couchbase** 🟢
- Metadata: Buckets, scopes, collections
- Schema: JSON schema inference
- Profiling: Supported
- Version: 6.5+

**DynamoDB** 🟢
- Metadata: Tables, attributes
- Schema: Key schema
- Profiling: Limited (cost implications)
- Special: GSI/LSI support

**Elasticsearch** 🟢
- Metadata: Indices, mappings
- Schema: Field definitions
- Profiling: Aggregations
- Version: 7.x, 8.x

**OpenSearch** 🟡
- Metadata: Indices, mappings
- Schema: Field definitions
- Profiling: Aggregations
- Version: 1.x, 2.x

#### Specialized Databases

**ClickHouse** 🟢
- Metadata: Tables, views
- Profiling: Full support
- Special: MergeTree engines
- Version: 21.x+

**Druid** 🟡
- Metadata: Datasources, dimensions
- Profiling: Limited
- Special: Real-time ingestion

**Hive** 🟢
- Metadata: Tables, partitions
- Lineage: Hook integration
- Profiling: Supported
- Version: 2.x, 3.x

**Presto/Trino** 🟢
- Metadata: Catalogs, schemas, tables
- Lineage: Query history
- Profiling: Supported
- Version: Latest

**Datalake Storage**

**AWS S3** 🟢⭐
- Metadata: Buckets, objects, prefixes
- Schema: Inferred from files (Parquet, Avro, JSON)
- Profiling: Supported
- Special: Glue Catalog integration

**Google Cloud Storage** 🟢
- Metadata: Buckets, objects
- Schema: File inference
- Profiling: Supported

**Azure Data Lake Storage (ADLS)** 🟢
- Metadata: Containers, directories, files
- Schema: File inference
- Profiling: Supported
- Version: Gen2

**Delta Lake** 🟢
- Metadata: Tables, partitions, history
- Schema: Delta schema
- Profiling: Supported
- Special: Time travel support

---

## Pipeline & ETL Connectors

### Workflow Orchestration

**Apache Airflow** 🟢⭐
- Metadata: DAGs, tasks, runs
- Lineage: Automatic from operators
- Status: Task execution status
- Logs: Task logs
- Special: Native OpenMetadata provider

**dbt (Data Build Tool)** 🟢⭐
- Metadata: Models, sources, tests, docs
- Lineage: Automatic from SQL
- Quality: dbt test results
- Documentation: Model descriptions
- Special: manifest.json parsing, column-level lineage

**Dagster** 🟢
- Metadata: Assets, jobs, ops
- Lineage: Asset dependencies
- Runs: Execution history
- Special: Software-defined assets

**Prefect** 🟢
- Metadata: Flows, tasks
- Lineage: Task dependencies
- Runs: Flow runs
- Version: 2.x

**Apache NiFi** 🟢
- Metadata: Process groups, processors
- Lineage: Connection flows
- Status: Component status
- Version: 1.x

### Data Integration Platforms

**Fivetran** 🟢
- Metadata: Connectors, schemas
- Lineage: Source to destination
- Sync: Sync history
- Status: Connector health

**Airbyte** 🟢
- Metadata: Connections, sources, destinations
- Lineage: Connection flows
- Sync: Sync logs
- Status: Connection health

**Matillion** 🟢
- Metadata: Jobs, components
- Lineage: Data flows
- Special: Cloud data warehouse ETL

**Talend** 🟡
- Metadata: Jobs, components
- Lineage: Component connections
- Special: Enterprise ETL

### Cloud-Native ETL

**AWS Glue** 🟢
- Metadata: Jobs, crawlers, databases
- Lineage: Job lineage
- Catalog: Glue Data Catalog
- Runs: Job runs

**Azure Data Factory** 🟢
- Metadata: Pipelines, activities, datasets
- Lineage: Activity flow
- Runs: Pipeline runs
- Special: Copy activity tracking

**Google Cloud Data Fusion** 🟢
- Metadata: Pipelines, datasets
- Lineage: Pipeline lineage
- Runs: Pipeline runs
- Special: CDAP-based

**Databricks Pipeline (Delta Live Tables)** 🟢
- Metadata: Pipelines, datasets
- Lineage: DLT dependencies
- Quality: Expectations
- Special: Streaming and batch

---

## BI & Analytics Connectors

### Business Intelligence

**Tableau** 🟢⭐
- Metadata: Workbooks, views, datasources
- Lineage: Datasource to view
- Usage: View statistics
- Special: Tableau Server/Online

**Power BI** 🟢⭐
- Metadata: Reports, datasets, dashboards
- Lineage: Dataset relationships
- Usage: Usage metrics
- Special: Power BI Service API

**Looker** 🟢⭐
- Metadata: Looks, dashboards, explores
- Lineage: LookML to database
- Usage: Query history
- Special: LookML parsing

**Metabase** 🟢
- Metadata: Questions, dashboards, collections
- Lineage: Query to tables
- Usage: Query logs
- Version: Latest

**Apache Superset** 🟢
- Metadata: Charts, dashboards, datasets
- Lineage: Chart to dataset
- Usage: Query logs
- Special: Open-source BI

**Qlik Sense** 🟢
- Metadata: Apps, sheets, objects
- Lineage: Data model
- Usage: Access logs
- Special: Cloud and Enterprise

**Mode** 🟢
- Metadata: Reports, queries
- Lineage: Query lineage
- Usage: Report views
- Special: SQL-first BI

**Sisense** 🟢
- Metadata: Dashboards, widgets, datasets
- Lineage: Widget to ElastiCube
- Special: ElastiCube support

**Domo** 🟢
- Metadata: Cards, datasets, pages
- Lineage: Card to dataset
- Usage: Card views

**Thoughtspot** 🟢
- Metadata: Answers, Liveboards, worksheets
- Lineage: Worksheet to tables
- Usage: Answer views
- Special: AI-powered analytics

### Notebooks & Data Science

**Jupyter Notebook** 🟡
- Metadata: Notebooks, cells
- Lineage: Input/output cells
- Special: Limited support

**Apache Zeppelin** 🟡
- Metadata: Notebooks, paragraphs
- Lineage: Limited
- Special: Multi-language support

---

## Messaging & Streaming Connectors

**Apache Kafka** 🟢⭐
- Metadata: Topics, schemas, partitions
- Schema Registry: Avro/JSON schemas
- Lineage: Producer/consumer tracking
- Quality: Message validation
- Special: Confluent Cloud support

**AWS Kinesis** 🟢
- Metadata: Streams, shards
- Schema: Inferred
- Special: Kinesis Data Streams

**Azure Event Hubs** 🟢
- Metadata: Namespaces, event hubs
- Schema: Event schema
- Special: Event Hub integration

**Google Pub/Sub** 🟢
- Metadata: Topics, subscriptions
- Schema: Schema registry
- Special: Cloud Pub/Sub

**Redpanda** 🟢
- Metadata: Topics, schemas
- Schema Registry: Compatible with Kafka
- Special: Kafka-compatible

**Apache Pulsar** 🟡
- Metadata: Topics, namespaces
- Schema: Schema registry
- Special: Multi-tenancy

---

## ML & Model Connectors

**MLflow** 🟢⭐
- Metadata: Experiments, runs, models
- Lineage: Model training lineage
- Metrics: Model metrics
- Artifacts: Model artifacts
- Special: Model registry integration

**Amazon SageMaker** 🟢
- Metadata: Models, endpoints, training jobs
- Lineage: Training to deployment
- Metrics: Model performance
- Special: Full SageMaker integration

**Google Vertex AI** 🟢
- Metadata: Models, endpoints, training pipelines
- Lineage: Training lineage
- Metrics: Model metrics
- Special: AutoML support

**Azure Machine Learning** 🟢
- Metadata: Models, experiments, datasets
- Lineage: Training lineage
- Metrics: Run metrics
- Special: Azure ML integration

**Databricks ML** 🟢
- Metadata: Experiments, runs, models
- Lineage: MLflow integration
- Special: Unity Catalog ML

---

## Storage Connectors

**Amazon S3** 🟢⭐
- Metadata: Buckets, objects, prefixes
- Schema: File inference (Parquet, Avro, CSV, JSON)
- Profiling: File-based profiling
- Lineage: S3 events
- Special: Glue Catalog integration

**Google Cloud Storage** 🟢
- Metadata: Buckets, objects
- Schema: File inference
- Profiling: Supported
- Special: BigQuery integration

**Azure Blob Storage** 🟢
- Metadata: Containers, blobs
- Schema: File inference
- Profiling: Supported
- Special: Data Lake Gen2

**Azure Data Lake Storage (ADLS)** 🟢
- Metadata: File systems, directories, files
- Schema: File inference
- Profiling: Supported
- Version: Gen2

---

## Metadata Connector (Migration)

**Apache Atlas** 🟢
- Purpose: Migrate from Atlas to OpenMetadata
- Metadata: All entity types
- Lineage: Preserved
- Special: Bulk import

**Amundsen** 🟢
- Purpose: Migrate from Amundsen
- Metadata: Tables, users, tags
- Special: Neo4j extraction

**Alation (Sink)** 🟢
- Purpose: Export to Alation
- Metadata: Bidirectional sync
- Special: API-based integration

---

## API Connectors

**REST API** 🟢
- Purpose: Custom integrations
- Metadata: Define schema
- Lineage: Configurable
- Special: Flexible connector

---

## Connector Maturity Levels

### 🟢 PROD (Production-Ready)
- Fully tested and stable
- Complete feature support
- Recommended for production use
- Regular updates and bug fixes

### 🟡 BETA
- Core features working
- May have limitations
- Suitable for non-critical workloads
- Active development

### 🔴 ALPHA
- Experimental
- Limited features
- Not recommended for production
- Early testing phase

---

## Connector Features Matrix

| Feature | Database | Pipeline | BI | Messaging | ML | Storage |
|---------|----------|----------|----|-----------|----|---------|
| **Metadata Extraction** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Schema Discovery** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Lineage** | ✅ | ✅ | ✅ | ⚠️ | ✅ | ⚠️ |
| **Data Profiling** | ✅ | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Usage** | ✅ | ✅ | ✅ | ⚠️ | ⚠️ | ⚠️ |
| **Quality Tests** | ✅ | ⚠️ | ❌ | ⚠️ | ⚠️ | ✅ |
| **Sample Data** | ✅ | ❌ | ❌ | ⚠️ | ❌ | ✅ |

✅ Full Support | ⚠️ Partial Support | ❌ Not Applicable

---

## Connector Configuration

### General Pattern

All connectors follow similar configuration structure:

```yaml
source:
  type: <connector-type>
  serviceName: <unique-service-name>
  serviceConnection:
    config:
      type: <connector-type>
      # Connector-specific configuration
      hostPort: <host:port>
      username: <username>
      password: <password>
      database: <database>
  sourceConfig:
    config:
      type: DatabaseMetadata  # or PipelineMetadata, DashboardMetadata, etc.
      # Additional ingestion options
      includeViews: true
      includeTags: true
      markDeletedTables: true

sink:
  type: metadata-rest
  config:
    api_endpoint: http://localhost:8585/api

workflowConfig:
  openMetadataServerConfig:
    hostPort: http://localhost:8585/api
    authProvider: openmetadata
    securityConfig:
      jwtToken: <jwt-token>
```

### Example: PostgreSQL Connector

```yaml
source:
  type: postgres
  serviceName: postgres_production
  serviceConnection:
    config:
      type: Postgres
      hostPort: postgres.example.com:5432
      username: openmetadata_user
      password: ${POSTGRES_PASSWORD}
      database: analytics
      sslMode: require
  sourceConfig:
    config:
      type: DatabaseMetadata
      schemaFilterPattern:
        includes:
          - public
          - analytics
        excludes:
          - temp_.*
      tableFilterPattern:
        includes:
          - .*
        excludes:
          - test_.*
      includeViews: true
      includeTags: true
      includeTables: true
      markDeletedTables: true
      
processor:
  type: query-parser
  config:
    filter: ""
    
sink:
  type: metadata-rest
  config:
    api_endpoint: http://localhost:8585/api

workflowConfig:
  openMetadataServerConfig:
    hostPort: http://localhost:8585/api
    authProvider: openmetadata
    securityConfig:
      jwtToken: ${OM_JWT_TOKEN}
```

---

## Running Connectors

### Via OpenMetadata UI (Recommended)

1. **Navigate**: Settings → Services
2. **Add Service**: Click "Add Service"
3. **Select Type**: Choose connector type
4. **Configure**: Fill in connection details
5. **Test**: Test connection
6. **Save**: Save configuration
7. **Add Ingestion**: Create ingestion pipeline
8. **Schedule**: Set schedule or run now

### Via CLI

```bash
# Create config file
vi postgres-config.yaml

# Run ingestion
metadata ingest -c postgres-config.yaml

# Profile data
metadata profile -c postgres-config.yaml
```

### Via Python SDK

```python
from metadata.ingestion.api.workflow import Workflow

# Load configuration
config = yaml.safe_load(open("postgres-config.yaml"))

# Create workflow
workflow = Workflow.create(config)

# Execute
workflow.execute()
workflow.raise_from_status()
workflow.print_status()
workflow.stop()
```

---

## Connector Best Practices

### Performance

✅ **Use filtering** to limit scope
- Schema/table/column filters
- Reduce metadata volume
- Faster ingestion

✅ **Schedule during off-peak hours**
- Minimize impact on source systems
- Better performance
- Lower costs

✅ **Use incremental ingestion**
- Only sync changes
- Faster execution
- Reduced load

✅ **Configure sampling** for profiling
- Don't scan entire tables
- Use row limits or percentages
- Balance accuracy vs. performance

### Security

✅ **Use read-only credentials**
- Minimum required permissions
- Reduce security risks
- Follow least privilege principle

✅ **Store secrets securely**
- Use secrets manager (Vault, AWS Secrets Manager)
- Don't commit credentials
- Rotate regularly

✅ **Use SSL/TLS**
- Encrypt connections
- Verify certificates
- Secure data in transit

✅ **Network security**
- Use VPN or private networks
- Whitelist IP addresses
- Use security groups

### Reliability

✅ **Test connections first**
- Verify credentials
- Check network access
- Validate permissions

✅ **Monitor ingestion logs**
- Check for errors
- Track execution time
- Alert on failures

✅ **Handle failures gracefully**
- Configure retries
- Set timeout limits
- Alert on repeated failures

✅ **Maintain metadata quality**
- Review extracted metadata
- Enrich with descriptions
- Assign owners

---

## Custom Connector Development

### When to Build Custom Connector

- Source not in 100+ connectors
- Proprietary system
- Unique requirements
- Special metadata needs

### Development Steps

1. **Define metadata model**
2. **Implement connector interface**
3. **Handle authentication**
4. **Extract metadata**
5. **Transform to OpenMetadata format**
6. **Test thoroughly**
7. **Document usage**
8. **Consider contributing**

### Resources

- **Connector SDK**: Python-based framework
- **Examples**: GitHub repository
- **Documentation**: Developer guide
- **Community**: Slack #connector-development

---

## Roadmap & Upcoming Connectors

### Planned (v1.11+)

- **Iceberg** (Native support)
- **Hudi** (Native support)
- **Snowplow** (Analytics)
- **Segment** (CDP)
- **Databricks Model Serving**
- **Additional cloud services**

### Community Requests

Vote and contribute on GitHub!

---

## Connector Support Matrix

| Connector | Metadata | Lineage | Profiling | Usage | Quality | Docs |
|-----------|----------|---------|-----------|-------|---------|------|
| **Snowflake** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/snowflake) |
| **BigQuery** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/bigquery) |
| **Redshift** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/redshift) |
| **Databricks** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/databricks) |
| **PostgreSQL** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/postgres) |
| **MySQL** | ✅ | ✅ | ✅ | ✅ | ✅ | [Link](https://docs.open-metadata.org/connectors/database/mysql) |
| **Airflow** | ✅ | ✅ | ❌ | ✅ | ⚠️ | [Link](https://docs.open-metadata.org/connectors/pipeline/airflow) |
| **dbt** | ✅ | ✅ | ❌ | ⚠️ | ✅ | [Link](https://docs.open-metadata.org/connectors/pipeline/dbt) |
| **Tableau** | ✅ | ✅ | ❌ | ✅ | ❌ | [Link](https://docs.open-metadata.org/connectors/dashboard/tableau) |
| **Power BI** | ✅ | ✅ | ❌ | ✅ | ❌ | [Link](https://docs.open-metadata.org/connectors/dashboard/powerbi) |
| **Kafka** | ✅ | ⚠️ | ❌ | ⚠️ | ⚠️ | [Link](https://docs.open-metadata.org/connectors/messaging/kafka) |
| **S3** | ✅ | ⚠️ | ✅ | ⚠️ | ✅ | [Link](https://docs.open-metadata.org/connectors/storage/s3) |
| **MLflow** | ✅ | ✅ | ❌ | ⚠️ | ❌ | [Link](https://docs.open-metadata.org/connectors/ml-model/mlflow) |

---

## Conclusion

OpenMetadata's **100+ connectors** provide comprehensive coverage of:

✅ **Modern data stack** (Snowflake, dbt, Fivetran, etc.)  
✅ **Legacy systems** (Oracle, DB2, Teradata)  
✅ **Cloud platforms** (AWS, Azure, GCP)  
✅ **Open source** (PostgreSQL, Kafka, Airflow)  
✅ **Commercial tools** (Tableau, Power BI, Looker)  

**Can't find your connector?** Check the community or build a custom one!

---

**Document Version**: 1.0  
**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3

For detailed connector documentation: https://docs.open-metadata.org/connectors
