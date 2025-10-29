# OpenMetadata v1.10.3 - Complete Documentation

[![OpenMetadata](https://img.shields.io/badge/OpenMetadata-v1.10.3-blue)](https://github.com/open-metadata/OpenMetadata)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)
[![Community](https://img.shields.io/badge/Slack-Join%20Community-purple)](https://slack.open-metadata.org/)

> **Comprehensive documentation for OpenMetadata - The Open Standard for Metadata Management**

Released: October 22, 2025

---

## 📚 Documentation Structure

This documentation is organized into multiple layers to serve different audiences and use cases:

### **Layer 1: Executive & Business Overview**
- [Executive Summary](./01-executive-summary/executive-overview.md) - High-level product overview
- [Business Value & ROI](./01-executive-summary/business-value.md) - Business benefits and value proposition
- [Use Cases & Success Stories](./01-executive-summary/use-cases.md) - Real-world implementations

### **Layer 2: Product Overview**
- [Product Introduction](./02-product-overview/product-introduction.md) - What is OpenMetadata?
- [Key Features & Capabilities](./02-product-overview/features.md) - Core functionality
- [Architecture Overview](./02-product-overview/architecture.md) - System design and components
- [Comparison with Alternatives](./02-product-overview/comparison.md) - Market positioning

### **Layer 3: Technical Deep Dive**
- [System Architecture](./03-technical-deep-dive/architecture-detailed.md) - Detailed technical architecture
- [Metadata Model & Standards](./03-technical-deep-dive/metadata-model.md) - Schema and data models
- [APIs & Integration](./03-technical-deep-dive/apis-integration.md) - REST APIs and SDKs
- [Security & Compliance](./03-technical-deep-dive/security-compliance.md) - Authentication and authorization

### **Layer 4: Deployment & Operations**
- [Deployment Options](./04-deployment-operations/deployment-options.md) - Installation methods
- [Infrastructure Requirements](./04-deployment-operations/infrastructure-requirements.md) - System requirements
- [Configuration Guide](./04-deployment-operations/configuration-guide.md) - Setup and configuration
- [Monitoring & Maintenance](./04-deployment-operations/monitoring-maintenance.md) - Operations best practices

### **Layer 5: Connectors & Integrations**
- [Connectors Overview](./05-connectors-integrations/connectors-overview.md) - All available connectors
- [Database Connectors](./05-connectors-integrations/database-connectors.md) - Database integrations
- [Pipeline & ETL Connectors](./05-connectors-integrations/pipeline-connectors.md) - Data pipeline integrations
- [BI & Analytics Connectors](./05-connectors-integrations/bi-analytics-connectors.md) - Dashboarding tools
- [Custom Connector Development](./05-connectors-integrations/custom-connectors.md) - Build your own

### **Layer 6: User Guides**
- [Getting Started Guide](./06-user-guides/getting-started.md) - Quick start for end users
- [Data Discovery](./06-user-guides/data-discovery.md) - Finding and exploring data
- [Data Governance](./06-user-guides/data-governance.md) - Implementing governance policies
- [Data Quality](./06-user-guides/data-quality.md) - Quality checks and monitoring
- [Collaboration Features](./06-user-guides/collaboration.md) - Team collaboration

### **Layer 7: Advanced Topics**
- [Data Lineage](./07-advanced-topics/data-lineage.md) - End-to-end lineage tracking
- [Data Observability](./07-advanced-topics/data-observability.md) - Monitoring data health
- [Auto-Classification & PII Detection](./07-advanced-topics/auto-classification.md) - Automated data classification
- [Custom Properties & Extensions](./07-advanced-topics/custom-properties.md) - Extending the platform
- [Performance Optimization](./07-advanced-topics/performance-optimization.md) - Scaling and tuning

### **Layer 8: SDK Reference & Developer Documentation**
- [**Python SDK Complete Guide**](./08-sdk-reference/README.md) - **Comprehensive SDK documentation**
- [SDK Quick Reference](./08-sdk-reference/05-QUICK_REFERENCE.md) - Cheat sheet for daily use
- [SDK Executive Summary](./08-sdk-reference/01-EXECUTIVE_SUMMARY.md) - Overview and benefits
- [SDK Complete Analysis](./08-sdk-reference/02-COMPLETE_ANALYSIS.md) - Deep dive into 142+ API methods
- [SDK Migration Guide](./08-sdk-reference/03-MIGRATION_GUIDE.md) - Step-by-step migration instructions
- [SDK Practical Examples](./08-sdk-reference/04-PRACTICAL_EXAMPLES.md) - 50+ code examples
- [Developer Guide](./08-developer-docs/developer-guide.md) - Contributing to OpenMetadata
- [API Reference](./08-developer-docs/api-reference.md) - REST API documentation
- [Plugin Development](./08-developer-docs/plugin-development.md) - Creating plugins

### **Layer 9: Migration & Upgrade**
- [Migration from Other Tools](./09-migration-upgrade/migration-guide.md) - Migrate from Amundsen, Atlas, etc.
- [Upgrade Guide](./09-migration-upgrade/upgrade-guide.md) - Version upgrade procedures
- [Data Migration](./09-migration-upgrade/data-migration.md) - Moving existing metadata

### **Layer 10: Reference Materials**
- [Glossary](./10-reference/glossary.md) - Terminology and definitions
- [FAQ](./10-reference/faq.md) - Frequently asked questions
- [Troubleshooting](./10-reference/troubleshooting.md) - Common issues and solutions
- [Release Notes](./10-reference/release-notes.md) - v1.10.3 release information
- [Resources & Community](./10-reference/resources-community.md) - Links and community resources

---

## 🚀 Quick Links

- **Official Website**: [https://open-metadata.org](https://open-metadata.org)
- **Documentation**: [https://docs.open-metadata.org](https://docs.open-metadata.org)
- **GitHub Repository**: [https://github.com/open-metadata/OpenMetadata](https://github.com/open-metadata/OpenMetadata)
- **Community Slack**: [https://slack.open-metadata.org](https://slack.open-metadata.org)
- **API Documentation**: [https://docs.open-metadata.org/swagger.html](https://docs.open-metadata.org/swagger.html)

---

## 📖 About This Documentation

This documentation package has been created to provide comprehensive coverage of OpenMetadata v1.10.3 for:

- **Decision Makers**: Executive summaries, business value, and ROI analysis
- **Architects**: Technical architecture, design patterns, and integration strategies
- **Developers**: API references, SDK guides, and plugin development
- **Data Engineers**: Connector setup, pipeline integration, and data quality
- **Data Analysts**: Data discovery, collaboration, and governance
- **Operations Teams**: Deployment, monitoring, and maintenance

### Documentation Philosophy

1. **Multi-layered**: Information organized by depth and audience
2. **Practical**: Real-world examples and use cases
3. **Comprehensive**: Complete coverage of features and capabilities
4. **Up-to-date**: Based on v1.10.3 (latest release as of October 2025)
5. **Open Source**: Free to use, share, and modify

---

## 🔄 Latest Updates (v1.10.3 - October 22, 2025)

### Key Improvements
- ✅ Databricks pipeline support for function parsing
- ✅ Configurable custom property fields in search settings
- ✅ Upgraded sqlalchemy-bigquery to v1.15.0

### Bug Fixes
- 🐛 Fixed table column descriptions handling
- 🐛 Resolved FQN encoding bug in Python SDK
- 🐛 Addressed protobuf version conflicts
- 🐛 Reverted dbt naming convention for consistency

[View Full Release Notes](./10-reference/release-notes.md)

---

## 🤝 Contributing to This Documentation

This is an open-source documentation project. Contributions are welcome!

### How to Contribute
1. Fork this repository
2. Make your changes
3. Submit a pull request
4. Engage with the community

### Areas for Contribution
- Additional use cases and examples
- Translation to other languages
- Corrections and clarifications
- New tutorials and guides

---

## 📄 License

This documentation is provided under the **Apache 2.0 License**, consistent with the OpenMetadata project.

OpenMetadata is an open-source project maintained by the community and sponsored by [Collate](https://getcollate.io).

---

## 🌟 Why OpenMetadata?

OpenMetadata is a **unified platform** for:
- 🔍 **Discovery**: Find and understand your data assets
- 📊 **Observability**: Monitor data quality and health
- 🛡️ **Governance**: Implement policies and compliance
- 🤝 **Collaboration**: Enable team-wide data knowledge sharing

Built on **open standards** with a **thriving community** of 1000+ active users and contributors.

---

**Last Updated**: October 29, 2025  
**Version**: 1.10.3  
**Maintained By**: Community Contributors

For questions or support, join our [Slack community](https://slack.open-metadata.org) or reach out at [sales@getcollate.io](mailto:sales@getcollate.io)
