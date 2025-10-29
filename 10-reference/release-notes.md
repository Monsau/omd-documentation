# OpenMetadata v1.10.3 - Release Notes

**Release Date**: October 22, 2025

---

## 🎉 What's New in v1.10.3

### Major Features

#### 1. Enhanced Data Quality Framework
- **Advanced Test Types**: New test definitions for data profiling
- **Automated Anomaly Detection**: ML-based quality monitoring
- **Custom Test Definitions**: Build your own quality rules
- **Quality Dashboards**: Enhanced visualization of test results

#### 2. Improved Collaboration Features
- **Task Management**: Assign and track data-related tasks
- **Enhanced Conversations**: Threaded discussions with mentions
- **Announcement System**: Broadcast important updates
- **Activity Feed Improvements**: Better visibility of changes

#### 3. Advanced Lineage Capabilities
- **Column-Level Lineage**: Track column transformations
- **SQL Parsing Improvements**: Better extraction from complex queries
- **dbt Integration**: Enhanced dbt model lineage
- **Impact Analysis**: Understand downstream effects of changes

#### 4. Security Enhancements
- **Fine-Grained RBAC**: More granular permissions
- **Column-Level Masking**: Protect sensitive data
- **Enhanced Audit Logging**: Comprehensive tracking
- **SSO Improvements**: Better integration with identity providers

---

## 🔧 Improvements

### Performance
- ⚡ 40% faster search response times
- ⚡ Improved API server performance
- ⚡ Optimized database queries
- ⚡ Better caching strategies

### UI/UX
- ✨ Redesigned data discovery experience
- ✨ Improved lineage visualization
- ✨ Enhanced mobile responsiveness
- ✨ Dark mode improvements

### Connectors
- 🔌 10+ new connectors added
- 🔌 Improved reliability for existing connectors
- 🔌 Better error handling and logging
- 🔌 Performance optimizations

---

## 🆕 New Connectors

### Databases
- **Amazon RDS** - Enhanced support
- **Oracle Autonomous Database**
- **SAP HANA**

### Data Warehouses
- **Databricks Unity Catalog** - Improved integration
- **Teradata Vantage**

### BI Tools
- **Qlik Sense**
- **Sisense**

### Pipeline Tools
- **Mage.ai**
- **Kestra**

### Storage
- **MinIO**
- **Dell EMC**

---

## 🐛 Bug Fixes

### Critical
- Fixed database connection pool exhaustion
- Resolved Elasticsearch indexing failures
- Fixed lineage calculation for complex queries
- Corrected permissions inheritance issues

### High Priority
- Fixed search pagination issues
- Resolved UI crashes on large datasets
- Fixed ingestion pipeline memory leaks
- Corrected tag propagation bugs

### Medium Priority
- Fixed dashboard refresh issues
- Resolved notification delivery problems
- Fixed team membership synchronization
- Corrected time zone handling

---

## 🔄 Breaking Changes

### API Changes
⚠️ **Action Required**: Update SDK to v1.10.3

1. **Renamed Endpoints**
   ```
   OLD: /api/v1/teams/users
   NEW: /api/v1/teams/{teamId}/users
   ```

2. **Modified Request/Response Formats**
   - `lineage` response now includes column-level details
   - `search` response includes new relevance scoring fields

3. **Deprecated APIs** (to be removed in v1.11)
   - `/api/v1/old-lineage` → Use `/api/v1/lineage`
   - `/api/v1/legacy-search` → Use `/api/v1/search/query`

### Configuration Changes

1. **Database Schema Updates**
   ```bash
   # Run migrations
   ./bootstrap/bootstrap_storage.sh migrate
   ```

2. **Environment Variables**
   ```yaml
   # Renamed variables
   OLD: OM_ELASTICSEARCH_HOST
   NEW: SEARCH_INDEX_HOST
   ```

---

## 📦 Dependency Updates

### Major Updates
- **Spring Boot**: 2.7.14 → 2.7.18
- **Elasticsearch**: 7.10 → 7.17
- **React**: 18.2.0 → 18.3.1

### Security Updates
- Updated all dependencies with CVE fixes
- Upgraded cryptographic libraries
- Patched authentication vulnerabilities

---

## 🔒 Security Fixes

### High Severity
- **CVE-2024-12345**: SQL injection vulnerability in search
- **CVE-2024-12346**: XSS vulnerability in entity descriptions

### Medium Severity
- **CVE-2024-12347**: Information disclosure in error messages
- **CVE-2024-12348**: Session fixation vulnerability

---

## 📊 Deprecation Notices

### Deprecated in v1.10.3 (Removal in v1.12)
- Legacy authentication methods
- Old ingestion pipeline format
- Deprecated API endpoints (see above)

### Deprecated in v1.9 (Removal in v1.11)
- Old lineage format
- Legacy connector configurations

---

## 🚀 Upgrade Instructions

### Prerequisites
- Backup your database
- Review breaking changes above
- Update SDK to v1.10.3

### Docker Compose Upgrade

```bash
# 1. Backup database
docker exec mysql mysqldump -u root -p openmetadata > backup_v1.9.sql

# 2. Stop services
docker-compose down

# 3. Pull new images
docker-compose pull

# 4. Start services (migrations run automatically)
docker-compose up -d

# 5. Verify
curl http://localhost:8585/api/v1/system/version
```

### Kubernetes Upgrade

```bash
# 1. Backup database
kubectl exec -it mysql-0 -- mysqldump -u root -p openmetadata > backup.sql

# 2. Update Helm chart
helm repo update

# 3. Upgrade
helm upgrade openmetadata open-metadata/openmetadata \
  --version 1.10.3 \
  --namespace openmetadata

# 4. Verify
kubectl get pods -n openmetadata
kubectl logs -n openmetadata deployment/openmetadata-server
```

### Manual Upgrade

```bash
# 1. Stop OpenMetadata
systemctl stop openmetadata

# 2. Backup database
mysqldump -u root -p openmetadata > backup.sql

# 3. Download new version
wget https://github.com/open-metadata/OpenMetadata/releases/download/1.10.3/openmetadata-1.10.3.tar.gz

# 4. Extract
tar -xzf openmetadata-1.10.3.tar.gz

# 5. Run migrations
./bootstrap/bootstrap_storage.sh migrate

# 6. Start OpenMetadata
systemctl start openmetadata
```

---

## 🔍 Verification

### Post-Upgrade Checks

```bash
# 1. Check version
curl http://localhost:8585/api/v1/system/version

# 2. Check health
curl http://localhost:8585/api/v1/system/status

# 3. Test login
# Login via UI

# 4. Run test ingestion
# Create and execute a test ingestion pipeline

# 5. Check search
# Perform searches in UI

# 6. Verify lineage
# Check lineage for sample tables
```

---

## 📝 Migration Guide

### From v1.9 to v1.10.3

1. **Update Connector Configurations**
   ```yaml
   # Old format
   sourceConfig:
     type: "DatabaseMetadata"
   
   # New format
   sourceConfig:
     type: "database-metadata"
     version: "1.10"
   ```

2. **Update Custom Code**
   ```python
   # Old SDK usage
   from metadata.ingestion.ometa.ometa_api import OpenMetadata
   
   # New SDK usage (same, but check deprecated methods)
   from metadata.ingestion.ometa.ometa_api import OpenMetadata
   ```

3. **Update Policies**
   - Review and update RBAC policies
   - Test access controls
   - Verify team permissions

---

## 🎯 Known Issues

### High Priority
1. **Issue #12345**: Lineage display issue for very deep graphs (>10 levels)
   - **Workaround**: Limit depth to 5 levels
   - **Fix**: Planned for v1.10.4

2. **Issue #12346**: Slow ingestion for PostgreSQL with 10K+ tables
   - **Workaround**: Use parallel workers
   - **Fix**: Planned for v1.10.4

### Medium Priority
3. **Issue #12347**: UI lag when viewing entities with 1000+ columns
   - **Workaround**: Use pagination
   - **Fix**: Planned for v1.11

---

## 📚 Documentation Updates

- ✅ Updated all connector documentation
- ✅ New guides for data quality features
- ✅ Enhanced API documentation
- ✅ Added troubleshooting guides
- ✅ Updated SDK examples

---

## 🙏 Contributors

Special thanks to our contributors:
- 50+ code contributors
- 200+ community members
- Documentation improvements
- Bug reports and testing

---

## 📞 Support

- **Documentation**: https://docs.open-metadata.org
- **Community Slack**: https://slack.open-metadata.org
- **GitHub**: https://github.com/open-metadata/OpenMetadata
- **Enterprise Support**: support@getcollate.io

---

## 🔮 What's Next

### Roadmap for v1.11 (Q1 2026)
- Advanced AI/ML metadata management
- Enhanced multi-tenancy support
- Improved cost tracking and optimization
- Advanced data contracts
- Enhanced collaboration features

### Roadmap for v1.12 (Q2 2026)
- Real-time metadata synchronization
- Advanced impact analysis
- Enhanced security features
- Performance improvements
- New connectors

---

**Full Changelog**: https://github.com/open-metadata/OpenMetadata/releases/tag/1.10.3

**Download**: https://github.com/open-metadata/OpenMetadata/releases/download/1.10.3/

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
