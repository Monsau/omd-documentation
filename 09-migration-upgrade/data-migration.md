# Data Migration: Metadata Store and Search

Use this guide to migrate OpenMetadata’s backing stores: the relational metadata database and the search index. Examples cover MySQL/Postgres and Elasticsearch/OpenSearch.

## When to migrate
- Move to a managed DB service
- Change DB engine (MySQL ➜ Postgres or vice versa)
- Rebuild or change search engine cluster
- Region move or DR event

## High-level flow
```mermaid
flowchart LR
	A[Snapshot Source DB] --> B[Provision Target DB]
	B --> C[Restore Dump]
	C --> D[Point Service to Target]
	D --> E[Reindex Search]
	E --> F[Validate & Cutover]
```

## Relational DB migration
Supported engines: MySQL or Postgres.

Steps:
1) Quiesce: pause non-critical ingestion jobs; announce maintenance window
2) Snapshot: full dump + binlog/LSN position for point-in-time restore if required
3) Provision target with equal or better capacity; configure users, TLS, and encryption
4) Restore dump; verify row counts and integrity checks
5) Update OM service configuration with new JDBC URL and credentials
6) Start service and run schema migrations if required
7) Validate read/write operations and core UI flows

Cutover options:
- Cold cutover: stop writes, migrate, start on target (simplest)
- Minimal downtime: replicate binlogs/CDC to target, brief switchover at end

Rollback:
- Keep the original instance intact until validation completes
- Switch DNS/URL back and restore previous config

## Search index migration/rebuild
You can migrate indices or rebuild from the metadata DB.

Options:
- Snapshot/restore (same engine family)
- Dual-write period and switchover
- Full reindex via OM jobs (recommended for version changes)

Recommendations:
- Recreate index templates/mappings compatible with the OM version
- Validate analyzer/tokenizer settings for search relevance
- Monitor reindex progress and query error rates

## Configuration changes
Update configuration values as per the [Configuration Guide](../04-deployment-operations/configuration-guide.md) for:
- Database URL, credentials, pool sizes, SSL/TLS
- Search endpoints, credentials, index prefix, TLS
- Kafka/bootstrap brokers if cluster changed

## Validation checklist
- Service health checks pass; no migration errors in logs
- Entity pages load with owners, tags, and profiles
- Search returns expected results; facets and counts look correct
- Lineage graphs render and expand correctly
- Ingestion runs succeed; webhooks and policies evaluate

## Troubleshooting
- Authentication failures: verify connection strings and TLS certs
- Schema mismatch: ensure service ran DB migrations on first start
- Stale search results: trigger reindex job and confirm mappings
- Performance regressions: tune DB pool sizes and search shards/replicas

## Related docs
- [Upgrade Guide](./upgrade-guide.md)
- [Migration Guide](./migration-guide.md)
- [Infrastructure Requirements](../04-deployment-operations/infrastructure-requirements.md)

---

Last Updated: October 29, 2025
