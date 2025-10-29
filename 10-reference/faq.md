# OpenMetadata - Frequently Asked Questions (FAQ)

## General Questions

### What is OpenMetadata?
OpenMetadata is an open-source metadata management platform for data discovery, governance, quality, and observability. It provides a unified view of all data assets across your organization.

### Is OpenMetadata free?
Yes, OpenMetadata is open source under Apache 2.0 license. You can use it freely. Enterprise support and managed SaaS options are also available.

### Who maintains OpenMetadata?
OpenMetadata is maintained by Collate.io and an active open-source community with contributors from around the world.

### What makes OpenMetadata different from alternatives?
- 100+ out-of-the-box connectors
- Built-in data quality testing
- Column-level lineage
- Modern UI/UX
- Comprehensive collaboration features
- Active development and community

---

## Installation & Deployment

### What are the deployment options?
1. **Docker Compose** - Quick start for development/testing
2. **Kubernetes** - Production deployment with Helm charts
3. **Cloud SaaS** - Fully managed service
4. **Bare Metal** - Direct installation on servers

### What are the system requirements?
**Minimum** (Development):
- 4 CPU cores
- 16 GB RAM
- 50 GB disk space

**Recommended** (Production):
- 8+ CPU cores
- 32+ GB RAM
- 200+ GB SSD storage

### Which databases are supported?
- MySQL 8.0+
- PostgreSQL 14+

### Can I use managed database services?
Yes! AWS RDS, Azure Database, Google Cloud SQL, and other managed services are supported.

### How long does deployment take?
- **Docker Compose**: 10-15 minutes
- **Kubernetes**: 1-2 hours (with configuration)
- **Production setup**: 1-2 days (with customization)

---

## Connectors & Integration

### How many connectors does OpenMetadata support?
Over 100 connectors across databases, data warehouses, dashboards, pipelines, messaging systems, and more.

### Can I build custom connectors?
Yes! OpenMetadata provides a Python SDK and plugin framework for building custom connectors.

### How often should I run ingestion?
Depends on your needs:
- **Real-time metadata**: Continuous ingestion
- **Daily updates**: Once per day (common)
- **Weekly**: For less critical sources

### Do connectors affect source system performance?
No, connectors only read metadata (schemas, statistics), not actual data. Impact is minimal.

### Can I connect to on-premises systems?
Yes, OpenMetadata can run on-premises or in a hybrid setup connecting to both cloud and on-premises sources.

---

## Data Lineage

### What types of lineage are supported?
- **Table-level lineage**: Relationships between tables
- **Column-level lineage**: How columns transform across pipeline
- **Pipeline lineage**: Task dependencies in workflows
- **Dashboard lineage**: Data sources for visualizations

### How is lineage captured?
- **Automatically**: From SQL queries, dbt models, Airflow DAGs
- **Manually**: Via UI or API
- **Via API**: Programmatically add lineage

### Can lineage track transformations?
Yes, OpenMetadata can capture SQL transformations, dbt models, and custom transformation logic.

### How far back can lineage go?
Lineage can trace upstream and downstream multiple hops (configurable depth).

---

## Data Quality

### Does OpenMetadata include data quality testing?
Yes! Built-in data quality framework with:
- Pre-built tests (null checks, uniqueness, range validation)
- Custom SQL tests
- Automated test execution
- Quality dashboards

### Can I integrate with Great Expectations?
Yes, OpenMetadata can integrate with external quality tools like Great Expectations.

### How are quality tests scheduled?
- Via OpenMetadata scheduler
- Via external orchestrators (Airflow)
- On-demand manual execution

### What happens when a test fails?
- Incident is created
- Notifications sent (email, Slack, webhook)
- Visible in quality dashboard
- Can trigger automated actions

---

## Security & Access Control

### What authentication methods are supported?
- Basic authentication (username/password)
- LDAP / Active Directory
- OAuth 2.0 (Google, Okta, Auth0, etc.)
- SAML 2.0 SSO
- Custom OIDC providers

### What is the access control model?
OpenMetadata uses **Role-Based Access Control (RBAC)** with support for:
- Pre-defined roles (Admin, Data Steward, Data Owner, Data Consumer)
- Custom roles
- Team-based permissions
- Resource-level permissions

### Can I control access at the column level?
Yes, OpenMetadata supports column-level security with masking policies.

### Is data encrypted?
Yes:
- **At rest**: AES-256 encryption for database
- **In transit**: TLS 1.3 for all communications

### Does OpenMetadata store actual data?
No, only metadata (schemas, statistics, lineage). Actual data remains in source systems.

---

## Search & Discovery

### How does search work?
OpenMetadata uses Elasticsearch for full-text search with:
- Fuzzy matching
- Faceted filtering
- Relevance scoring
- Auto-suggestions

### Can I search across all entities?
Yes, unified search across tables, dashboards, pipelines, topics, ML models, and more.

### How is relevance determined?
Based on multiple factors:
- Text match
- Usage popularity
- User interactions
- Ownership
- Data quality

### Can I save searches?
Not directly, but you can bookmark frequently accessed entities.

---

## Collaboration

### How do teams collaborate in OpenMetadata?
- **Conversations**: Threaded discussions on entities
- **Tasks**: Assign and track work
- **Announcements**: Broadcast important updates
- **Activity feeds**: See recent changes
- **Mentions**: Tag users with @mentions

### Can I get notifications?
Yes, notifications for:
- Mentions
- Task assignments
- Data quality failures
- Ownership changes
- Schema changes

### Does OpenMetadata integrate with Slack?
Yes, notifications and alerts can be sent to Slack channels.

---

## Performance & Scalability

### How many entities can OpenMetadata handle?
Tested with:
- 1M+ tables
- 10M+ columns
- Hundreds of thousands of users

### What's the query performance like?
- **Search**: Sub-second (Elasticsearch)
- **Entity retrieval**: <100ms
- **Lineage**: <500ms for moderate depth

### How do I scale OpenMetadata?
- Horizontal scaling of API servers
- Elasticsearch cluster expansion
- Database read replicas
- Redis caching

### What's the ingestion throughput?
- **Typical**: 1,000-10,000 tables/hour
- **Optimized**: 50,000+ tables/hour with parallel workers

---

## API & SDK

### Is there a REST API?
Yes, comprehensive REST API for all operations. See [API Documentation](../03-technical-deep-dive/apis-integration.md).

### Is there an SDK?
Yes, official Python SDK (`openmetadata-ingestion`). See [SDK Reference](../08-sdk-reference/README.md).

### Can I use GraphQL?
Not currently, but planned for future releases.

### Are there SDKs for other languages?
Python is the primary SDK. Community contributions for other languages are welcome.

---

## Troubleshooting

### OpenMetadata won't start
**Check**:
1. Database connectivity
2. Elasticsearch health
3. Port availability (8585)
4. Log files for errors

### Ingestion fails
**Check**:
1. Source connectivity
2. Credentials/permissions
3. Ingestion logs
4. API server health

### Search not working
**Check**:
1. Elasticsearch status
2. Index health
3. Recent ingestion success

### UI is slow
**Possible causes**:
- Elasticsearch performance
- Database queries
- Network latency
- Browser cache

**Solutions**:
- Enable Redis caching
- Optimize database indexes
- Scale Elasticsearch
- Clear browser cache

---

## Migration & Upgrades

### Can I migrate from Amundsen/Atlas/DataHub?
Yes, migration guides and tools are available. See [Migration Guide](../09-migration-upgrade/migration-guide.md).

### How do I upgrade OpenMetadata?
Follow the upgrade procedure:
1. Backup database
2. Review release notes
3. Run database migrations
4. Update deployment
5. Verify functionality

### Is there downtime during upgrades?
- **Rolling updates**: Minimal downtime (<5 minutes)
- **Major upgrades**: 15-30 minutes recommended

### How often are releases?
- **Minor releases**: Every 2-4 weeks
- **Major releases**: Quarterly

---

## Licensing & Support

### What is the license?
Apache License 2.0 - permissive open source license allowing commercial use.

### Is commercial support available?
Yes, enterprise support packages available from Collate.io including:
- 24/7 support
- SLA guarantees
- Custom feature development
- Training and consulting

### Where can I get help?
- **Documentation**: https://docs.open-metadata.org
- **Community Slack**: https://slack.open-metadata.org
- **GitHub Issues**: https://github.com/open-metadata/OpenMetadata/issues
- **Community Forum**: Community discussions

---

## Best Practices

### How should I organize teams?
- Mirror organizational structure
- Group by business function
- Assign clear ownership

### What naming conventions should I use?
- Use lowercase with underscores
- Be descriptive and consistent
- Avoid special characters
- Follow company standards

### How often should I review metadata?
- **Tags/descriptions**: Quarterly
- **Ownership**: Monthly
- **Quality tests**: Weekly
- **Access controls**: Monthly

### Should I tag everything?
Focus on:
- PII and sensitive data
- Critical business assets
- Deprecated datasets
- Tier classification

---

## Next Steps

📚 **More Information**:
- [Getting Started Guide](../06-user-guides/getting-started.md)
- [Deployment Options](../04-deployment-operations/deployment-options.md)
- [Troubleshooting Guide](./troubleshooting.md)

💬 **Get Help**:
- Join our Slack community
- Check GitHub discussions
- Review documentation

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
