# OpenMetadata Deployment Options - v1.10.3

## Overview

OpenMetadata offers flexible deployment options to meet various organizational needs, from local development to enterprise-scale production deployments.

---

## Deployment Decision Matrix

| Scenario | Recommended Deployment | Rationale |
|----------|----------------------|-----------|
| **Proof of Concept** | Docker Compose (Local) | Quick setup, minimal resources |
| **Development/Testing** | Docker Compose | Easy to reset, full feature access |
| **Small Production** (<100 users) | Docker Compose | Cost-effective, sufficient performance |
| **Medium Production** (100-1000 users) | Kubernetes | Scalability, high availability |
| **Enterprise Production** (1000+ users) | Kubernetes with HA | Full redundancy, performance at scale |
| **No IT Resources** | Cloud SaaS (Collate) | Zero infrastructure management |
| **Hybrid Requirements** | Kubernetes + On-Prem Connectors | Data sovereignty + convenience |

---

## Option 1: Cloud SaaS (Collate)

### Overview
Fully managed OpenMetadata hosted and maintained by Collate.

### ✅ Advantages
- **Zero infrastructure management**
- **Automatic updates** to latest version
- **99.9% SLA** guarantee
- **Professional support** included
- **Enterprise security** (SOC 2, ISO 27001)
- **Rapid deployment** (minutes, not days)
- **Scalability** handled automatically
- **No maintenance** burden

### ⚠️ Considerations
- Monthly subscription cost
- Data resides in Collate cloud
- Limited infrastructure customization
- Internet connectivity required

### Ideal For
- Organizations wanting fastest time-to-value
- Teams without dedicated infrastructure resources
- Companies preferring OpEx over CapEx
- Businesses requiring enterprise SLA

### Pricing
- **Starter**: $2,000/month
- **Professional**: $5,000/month
- **Enterprise**: Custom pricing
- **Free Trial**: 30 days

### Getting Started
1. Visit [https://getcollate.io](https://getcollate.io)
2. Sign up for free trial
3. Configure connectors via UI
4. Start using immediately

### Architecture
```
┌─────────────────────────────────────────┐
│         Collate Cloud (Managed)         │
│  ┌─────────────────────────────────┐   │
│  │   OpenMetadata Application      │   │
│  │  (Load Balanced, Auto-Scaled)   │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Managed Database & Search      │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
              ↕ HTTPS
┌─────────────────────────────────────────┐
│       Your Infrastructure               │
│  ┌─────────────────────────────────┐   │
│  │  Data Sources (Optional         │   │
│  │  Secure Connectors)             │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## Option 2: Docker Compose (Local/Single Server)

### Overview
Containerized deployment on a single machine using Docker Compose.

### ✅ Advantages
- **Quick setup** (5-10 minutes)
- **Easy to manage** with single command
- **Low resource requirements**
- **Perfect for testing** and development
- **No Kubernetes complexity**
- **Portable** across environments
- **Complete control**

### ⚠️ Considerations
- Single point of failure
- Limited horizontal scaling
- Manual updates required
- No built-in high availability

### Ideal For
- Proof of concepts
- Development and testing
- Small teams (<100 users)
- Single-server deployments
- Learning and experimentation

### System Requirements

**Minimum**:
- **CPU**: 4 cores
- **RAM**: 16 GB
- **Storage**: 50 GB
- **OS**: Linux, macOS, Windows (WSL2)
- **Docker**: 20.10+
- **Docker Compose**: v2.0+

**Recommended**:
- **CPU**: 8 cores
- **RAM**: 32 GB
- **Storage**: 100 GB SSD
- **OS**: Linux (Ubuntu 20.04+, RHEL 8+)

### Quick Start

#### Step 1: Install Prerequisites
```bash
# Install Docker (Ubuntu/Debian)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

#### Step 2: Download OpenMetadata
```bash
git clone https://github.com/open-metadata/OpenMetadata
cd OpenMetadata
git checkout 1.10.3-release
```

#### Step 3: Start OpenMetadata
```bash
cd docker/local-metadata
docker-compose up -d
```

#### Step 4: Access OpenMetadata
- **URL**: http://localhost:8585
- **Username**: admin
- **Password**: admin

#### Step 5: Verify Installation
```bash
# Check running containers
docker-compose ps

# View logs
docker-compose logs -f openmetadata-server

# Check health
curl http://localhost:8585/healthcheck
```

### Architecture
```
┌──────────────────────────────────────────┐
│          Docker Host                     │
│  ┌────────────────────────────────────┐ │
│  │  OpenMetadata Server               │ │
│  │  (Port 8585)                       │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │  MySQL Database                    │ │
│  │  (Port 3306)                       │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │  Elasticsearch                     │ │
│  │  (Port 9200, 9300)                 │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │  Ingestion Container (Optional)    │ │
│  └────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

### Configuration

#### Environment Variables (`docker-compose.yml`)
```yaml
environment:
  # Database
  DB_DRIVER_CLASS: com.mysql.cj.jdbc.Driver
  DB_SCHEME: mysql
  DB_HOST: mysql
  DB_PORT: 3306
  DB_USER: openmetadata_user
  DB_PASSWORD: openmetadata_password
  
  # Elasticsearch
  ELASTICSEARCH_HOST: elasticsearch
  ELASTICSEARCH_PORT: 9200
  
  # Authentication
  AUTHENTICATION_PROVIDER: basic
  # For SSO, change to 'google', 'okta', 'auth0', etc.
  
  # Server
  SERVER_HOST: 0.0.0.0
  SERVER_PORT: 8585
```

### Data Persistence

Volumes are configured for data persistence:
```yaml
volumes:
  - ./docker-volume/mysql:/var/lib/mysql
  - ./docker-volume/elasticsearch:/usr/share/elasticsearch/data
```

### Scaling Limitations
- Single instance (no horizontal scaling)
- Limited to single machine resources
- No automatic failover

### When to Upgrade
Consider Kubernetes when:
- Users exceed 100
- High availability required
- Need horizontal scaling
- Production-critical workload

---

## Option 3: Kubernetes

### Overview
Cloud-native deployment using Kubernetes for scalability and high availability.

### ✅ Advantages
- **Horizontal scaling** based on load
- **High availability** with multiple replicas
- **Self-healing** with automatic restarts
- **Rolling updates** with zero downtime
- **Resource efficiency** with scheduling
- **Production-grade** reliability
- **Multi-cloud** portability

### ⚠️ Considerations
- Kubernetes expertise required
- Higher complexity
- More infrastructure cost
- Longer initial setup time

### Ideal For
- Production deployments (100+ users)
- Mission-critical workloads
- Organizations with Kubernetes expertise
- Multi-region deployments
- High-traffic environments

### Prerequisites

**Kubernetes Cluster**:
- **Kubernetes**: v1.21+
- **Nodes**: 3+ (for HA)
- **Node Size**: 4 CPU, 16 GB RAM minimum per node
- **Storage**: Persistent volumes support
- **Ingress**: Nginx or similar

**Tools**:
- `kubectl` CLI
- `helm` v3+
- Access to container registry

### Deployment with Helm

#### Step 1: Add Helm Repository
```bash
helm repo add open-metadata https://helm.open-metadata.org/
helm repo update
```

#### Step 2: Create Namespace
```bash
kubectl create namespace openmetadata
```

#### Step 3: Create Values File (`values.yaml`)
```yaml
# Basic Configuration
global:
  clusterName: openmetadata-prod
  
# OpenMetadata Server
openmetadata:
  replicaCount: 3  # For HA
  
  image:
    repository: openmetadata/server
    tag: 1.10.3
  
  resources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 4
      memory: 8Gi
  
  config:
    authentication:
      provider: "google"  # or okta, auth0, etc.
    
    database:
      host: mysql
      port: 3306
      driverClass: com.mysql.cj.jdbc.Driver
    
    elasticsearch:
      host: elasticsearch
      port: 9200

# MySQL Configuration
mysql:
  enabled: true
  replicaCount: 3  # For HA
  auth:
    rootPassword: "root-password"
    database: "openmetadata_db"
    username: "openmetadata_user"
    password: "openmetadata_password"
  
  primary:
    persistence:
      size: 100Gi
      storageClass: "fast-ssd"
  
  resources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 4
      memory: 8Gi

# Elasticsearch Configuration
elasticsearch:
  enabled: true
  replicas: 3  # For HA
  
  volumeClaimTemplate:
    resources:
      requests:
        storage: 100Gi
    storageClassName: "fast-ssd"
  
  resources:
    requests:
      cpu: 2
      memory: 4Gi
    limits:
      cpu: 4
      memory: 8Gi

# Ingress Configuration
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: openmetadata.yourdomain.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: openmetadata-tls
      hosts:
        - openmetadata.yourdomain.com
```

#### Step 4: Install OpenMetadata
```bash
helm install openmetadata open-metadata/openmetadata \
  --namespace openmetadata \
  --values values.yaml \
  --version 1.10.3
```

#### Step 5: Verify Deployment
```bash
# Check pod status
kubectl get pods -n openmetadata

# Check services
kubectl get svc -n openmetadata

# Check ingress
kubectl get ingress -n openmetadata

# View logs
kubectl logs -n openmetadata -l app=openmetadata --tail=100
```

#### Step 6: Access OpenMetadata
```bash
# Get ingress URL
kubectl get ingress -n openmetadata

# Or port-forward for testing
kubectl port-forward -n openmetadata svc/openmetadata 8585:8585
```

### Architecture (HA Configuration)
```
┌─────────────────────────────────────────────────────┐
│              Load Balancer / Ingress                │
└─────────────────────────────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│           OpenMetadata Pods (x3)                    │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐     │
│  │  Server  │    │  Server  │    │  Server  │     │
│  │  Pod 1   │    │  Pod 2   │    │  Pod 3   │     │
│  └──────────┘    └──────────┘    └──────────┘     │
└─────────────────────────────────────────────────────┘
             ↓                    ↓
┌──────────────────────┐  ┌──────────────────────────┐
│   MySQL Cluster      │  │  Elasticsearch Cluster   │
│   (Master + Slaves)  │  │  (3 nodes)               │
│  ┌────┐ ┌────┐      │  │  ┌────┐ ┌────┐ ┌────┐   │
│  │ M  │→│ S  │→     │  │  │ ES │ │ ES │ │ ES │   │
│  └────┘ └────┘      │  │  │  1 │ │  2 │ │  3 │   │
│         ↓            │  │  └────┘ └────┘ └────┘   │
│       ┌────┐         │  │                          │
│       │ S  │         │  │                          │
│       └────┘         │  │                          │
└──────────────────────┘  └──────────────────────────┘
```

### Scaling

#### Horizontal Pod Autoscaling (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: openmetadata-hpa
  namespace: openmetadata
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: openmetadata
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Monitoring

#### Prometheus Integration
```yaml
openmetadata:
  metrics:
    enabled: true
    port: 9090
  
  serviceMonitor:
    enabled: true
    interval: 30s
```

#### Key Metrics to Monitor
- API response times
- Error rates
- Active user sessions
- Database connections
- Search query latency
- Memory and CPU usage
- Pod restart count

---

## Option 4: Bare Metal

### Overview
Direct installation on physical or virtual machines without containers.

### ✅ Advantages
- **Maximum performance** (no virtualization overhead)
- **Complete control** over environment
- **Legacy compatibility** with existing systems
- **No container runtime** required

### ⚠️ Considerations
- Manual dependency management
- More complex updates
- Less portable
- Requires system administration skills

### Ideal For
- Organizations avoiding containers
- Legacy infrastructure
- Maximum performance requirements
- Specific compliance needs

### Prerequisites

**System Requirements**:
- **OS**: Linux (Ubuntu 20.04+, RHEL 8+, CentOS 8+)
- **CPU**: 8+ cores
- **RAM**: 32+ GB
- **Storage**: 200+ GB SSD
- **Java**: OpenJDK 17
- **Python**: 3.8+
- **MySQL**: 8.0+
- **Elasticsearch**: 8.x

### Installation Steps

#### Step 1: Install Dependencies
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Java 17
sudo apt install -y openjdk-17-jdk

# Install Python 3.8+
sudo apt install -y python3.8 python3-pip

# Install MySQL 8
sudo apt install -y mysql-server

# Install Elasticsearch 8
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install elasticsearch
```

#### Step 2: Configure MySQL
```bash
sudo mysql_secure_installation

# Create database and user
sudo mysql -u root -p << EOF
CREATE DATABASE openmetadata_db;
CREATE USER 'openmetadata_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON openmetadata_db.* TO 'openmetadata_user'@'localhost';
FLUSH PRIVILEGES;
EOF
```

#### Step 3: Configure Elasticsearch
```bash
# Edit /etc/elasticsearch/elasticsearch.yml
sudo vi /etc/elasticsearch/elasticsearch.yml

# Add/modify:
cluster.name: openmetadata
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node

# Start Elasticsearch
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

#### Step 4: Download OpenMetadata
```bash
cd /opt
sudo wget https://github.com/open-metadata/OpenMetadata/releases/download/1.10.3-release/openmetadata-1.10.3.tar.gz
sudo tar -xzf openmetadata-1.10.3.tar.gz
sudo mv openmetadata-1.10.3 openmetadata
cd openmetadata
```

#### Step 5: Configure OpenMetadata
```bash
# Edit conf/openmetadata.yaml
sudo vi conf/openmetadata.yaml

# Update database settings
database:
  driverClass: com.mysql.cj.jdbc.Driver
  url: jdbc:mysql://localhost:3306/openmetadata_db
  user: openmetadata_user
  password: strong_password

# Update Elasticsearch settings
elasticsearch:
  host: localhost
  port: 9200
```

#### Step 6: Initialize Database
```bash
# Run migrations
sudo ./bootstrap/bootstrap_storage.sh
```

#### Step 7: Start OpenMetadata
```bash
# Start server
sudo ./bin/openmetadata-server-start.sh conf/openmetadata.yaml

# Or create systemd service for automatic start
sudo vi /etc/systemd/system/openmetadata.service
```

**Systemd Service File**:
```ini
[Unit]
Description=OpenMetadata Server
After=network.target mysql.service elasticsearch.service

[Service]
Type=forking
User=openmetadata
Group=openmetadata
ExecStart=/opt/openmetadata/bin/openmetadata-server-start.sh /opt/openmetadata/conf/openmetadata.yaml
ExecStop=/opt/openmetadata/bin/openmetadata-server-stop.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start openmetadata
sudo systemctl enable openmetadata
```

#### Step 8: Verify Installation
```bash
# Check service status
sudo systemctl status openmetadata

# Check logs
sudo tail -f /opt/openmetadata/logs/openmetadata.log

# Test API
curl http://localhost:8585/healthcheck
```

---

## Option 5: Cloud-Specific Deployments

### AWS

#### Using EKS (Elastic Kubernetes Service)
- Deploy using Helm on EKS cluster
- Use RDS for MySQL
- Use Amazon Elasticsearch Service
- Store backups in S3
- Use ALB for ingress

#### Using ECS (Elastic Container Service)
- Deploy containers on ECS
- Use RDS and Amazon ES
- Configure auto-scaling
- Use ALB for load balancing

### Azure

#### Using AKS (Azure Kubernetes Service)
- Deploy using Helm on AKS
- Use Azure Database for MySQL
- Use Azure Cognitive Search or self-hosted ES
- Store backups in Azure Blob Storage
- Use Azure Application Gateway

### Google Cloud Platform

#### Using GKE (Google Kubernetes Engine)
- Deploy using Helm on GKE
- Use Cloud SQL for MySQL
- Use self-hosted Elasticsearch or Elastic Cloud
- Store backups in GCS
- Use Google Cloud Load Balancer

---

## Comparison Matrix

| Feature | Docker Compose | Kubernetes | Bare Metal | Cloud SaaS |
|---------|---------------|------------|------------|------------|
| **Setup Time** | 10 min | 1-2 hours | 2-4 hours | 5 min |
| **Complexity** | Low | High | Medium | None |
| **Scalability** | Limited | Excellent | Manual | Automatic |
| **High Availability** | No | Yes | Manual | Yes |
| **Cost** | Low | Medium | Low-Medium | Medium-High |
| **Maintenance** | Low | Medium | High | None |
| **Control** | High | High | Highest | Low |
| **Updates** | Manual | Semi-auto | Manual | Automatic |
| **Best For** | Dev/Test | Production | Legacy | Quick Start |

---

## Migration Between Deployments

### From Docker to Kubernetes
1. Backup MySQL and Elasticsearch data
2. Deploy Kubernetes with same configuration
3. Restore data to Kubernetes instances
4. Update DNS to point to new deployment
5. Decommission Docker deployment

### From Self-Hosted to SaaS
1. Contact Collate for migration plan
2. Export metadata using APIs
3. Collate imports data to cloud instance
4. Validate data migration
5. Switch users to SaaS URL

---

## Best Practices

### General
- ✅ Use separate database for production
- ✅ Enable SSL/TLS for all connections
- ✅ Configure regular backups
- ✅ Implement monitoring and alerting
- ✅ Use secrets management (Vault, AWS Secrets Manager)
- ✅ Enable audit logging
- ✅ Plan for disaster recovery

### Security
- ✅ Change default passwords immediately
- ✅ Use strong passwords (16+ characters)
- ✅ Enable SSO authentication
- ✅ Restrict network access with firewalls
- ✅ Keep software up to date
- ✅ Regular security audits

### Performance
- ✅ Use SSDs for storage
- ✅ Allocate sufficient memory
- ✅ Optimize database queries
- ✅ Configure connection pooling
- ✅ Enable caching where appropriate
- ✅ Monitor resource usage

---

## Conclusion

Choose your deployment based on:

**Quick Start & Testing** → Docker Compose  
**Production (Small-Medium)** → Docker Compose or Kubernetes  
**Production (Large/Enterprise)** → Kubernetes  
**No IT Resources** → Cloud SaaS  
**Maximum Control** → Bare Metal  

---

**Document Version**: 1.0  
**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3

For detailed deployment documentation: https://docs.open-metadata.org/latest/deployment
