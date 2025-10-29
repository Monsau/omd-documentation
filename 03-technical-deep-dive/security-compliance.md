# OpenMetadata - Security & Compliance

## Overview

OpenMetadata provides enterprise-grade security features to protect metadata, control access, and ensure compliance with regulatory requirements. This document covers authentication, authorization, data protection, and compliance frameworks.

---

## Security Architecture

```mermaid
flowchart TB
    subgraph "Perimeter Security"
        FIREWALL[Firewall]
        WAF[Web Application Firewall]
        DDoS[DDoS Protection]
    end
    
    subgraph "Authentication Layer"
        LDAP[LDAP/Active Directory]
        OAUTH[OAuth2/OIDC]
        SAML[SAML SSO]
        JWT[JWT Token Manager]
    end
    
    subgraph "Authorization Layer"
        RBAC[RBAC Engine]
        ABAC[Policy Engine]
        ACL[Access Control Lists]
    end
    
    subgraph "Application Security"
        INPUT[Input Validation]
        CSRF[CSRF Protection]
        XSS[XSS Prevention]
        SQL[SQL Injection Prevention]
    end
    
    subgraph "Data Protection"
        ENCRYPT_REST[Encryption at Rest<br/>AES-256]
        ENCRYPT_TRANSIT[Encryption in Transit<br/>TLS 1.3]
        MASKING[PII Masking]
        TOKENIZE[Data Tokenization]
    end
    
    subgraph "Audit & Compliance"
        AUDIT[Audit Logging]
        MONITOR[Security Monitoring]
        COMPLIANCE[Compliance Reports]
    end
    
    FIREWALL --> WAF
    WAF --> DDoS
    DDoS --> LDAP
    DDoS --> OAUTH
    DDoS --> SAML
    
    LDAP --> JWT
    OAUTH --> JWT
    SAML --> JWT
    
    JWT --> RBAC
    RBAC --> ABAC
    ABAC --> ACL
    
    ACL --> INPUT
    INPUT --> CSRF
    CSRF --> XSS
    XSS --> SQL
    
    SQL --> ENCRYPT_REST
    SQL --> ENCRYPT_TRANSIT
    SQL --> MASKING
    SQL --> TOKENIZE
    
    ENCRYPT_REST --> AUDIT
    ENCRYPT_TRANSIT --> AUDIT
    MASKING --> AUDIT
    AUDIT --> MONITOR
    MONITOR --> COMPLIANCE
    
    style JWT fill:#e6f3ff
    style RBAC fill:#e1f5e1
    style AUDIT fill:#fff4e6
```

---

## Authentication

### 1. **Basic Authentication**

Simple username/password authentication for development.

```mermaid
sequenceDiagram
    participant User
    participant UI
    participant AuthService
    participant Database
    
    User->>UI: Enter Credentials
    UI->>AuthService: POST /api/v1/users/login<br/>{email, password}
    AuthService->>Database: Verify Credentials
    Database-->>AuthService: User Found
    AuthService->>AuthService: Generate JWT Token
    AuthService-->>UI: {accessToken, refreshToken}
    UI->>UI: Store Tokens
    UI-->>User: Login Success
```

**Configuration**:
```yaml
authenticationConfiguration:
  provider: "basic"
  publicKeyUrls:
    - "http://localhost:8585/api/v1/system/config/jwks"
  authority: "https://accounts.google.com"
  clientId: "your-client-id"
  callbackUrl: "http://localhost:8585/callback"
```

---

### 2. **LDAP / Active Directory**

Enterprise directory integration.

```yaml
authenticationConfiguration:
  provider: "ldap"
  ldapConfiguration:
    host: "ldap.company.com"
    port: 389
    dnAdminPrincipal: "cn=admin,dc=company,dc=com"
    dnAdminPassword: "admin_password"
    userBaseDN: "ou=users,dc=company,dc=com"
    mailAttributeName: "mail"
```

**User Mapping**:
```
LDAP Attribute → OpenMetadata Field
- cn → name
- mail → email
- memberOf → teams
- title → description
```

---

### 3. **OAuth 2.0 / OIDC**

Support for external identity providers.

```mermaid
sequenceDiagram
    participant User
    participant OM as OpenMetadata
    participant IDP as Identity Provider
    
    User->>OM: Click "Login with Google"
    OM->>IDP: Redirect to Auth Page
    IDP->>User: Show Login Page
    User->>IDP: Enter Credentials
    IDP->>IDP: Authenticate User
    IDP->>OM: Redirect with Auth Code
    OM->>IDP: Exchange Code for Token
    IDP-->>OM: Access Token + ID Token
    OM->>OM: Create/Update User
    OM->>OM: Generate JWT
    OM-->>User: Login Success
```

**Google OAuth Configuration**:
```yaml
authenticationConfiguration:
  provider: "google"
  publicKeyUrls:
    - "https://www.googleapis.com/oauth2/v3/certs"
  authority: "https://accounts.google.com"
  clientId: "your-google-client-id.apps.googleusercontent.com"
  callbackUrl: "http://localhost:8585/callback"
```

**Okta OIDC Configuration**:
```yaml
authenticationConfiguration:
  provider: "okta"
  publicKeyUrls:
    - "https://your-domain.okta.com/oauth2/v1/keys"
  authority: "https://your-domain.okta.com"
  clientId: "your-okta-client-id"
  callbackUrl: "http://localhost:8585/callback"
  scope: "openid email profile groups"
```

**Azure AD Configuration**:
```yaml
authenticationConfiguration:
  provider: "azure"
  publicKeyUrls:
    - "https://login.microsoftonline.com/common/discovery/v2.0/keys"
  authority: "https://login.microsoftonline.com/{tenant-id}"
  clientId: "your-azure-client-id"
  callbackUrl: "http://localhost:8585/callback"
```

---

### 4. **SAML 2.0 SSO**

Enterprise single sign-on.

```yaml
authenticationConfiguration:
  provider: "saml"
  samlConfiguration:
    idp:
      entityId: "https://idp.company.com"
      ssoLoginUrl: "https://idp.company.com/sso/login"
      x509Certificate: "MIIDXTCCAkWgAwIBAgIJ..."
    sp:
      entityId: "https://openmetadata.company.com"
      acs: "https://openmetadata.company.com/api/v1/saml/acs"
      callback: "https://openmetadata.company.com/saml/callback"
```

---

### 5. **JWT Token Management**

```mermaid
flowchart LR
    A[User Authenticates] --> B[Generate Access Token<br/>Expires: 1 hour]
    B --> C[Generate Refresh Token<br/>Expires: 7 days]
    C --> D{Token Valid?}
    
    D -->|Yes| E[Access Resources]
    D -->|Expired| F[Use Refresh Token]
    F --> G[Generate New Access Token]
    G --> E
    
    D -->|Invalid| H[Re-authenticate]
    
    style B fill:#e6f3ff
    style C fill:#e1f5e1
    style H fill:#ffe6e6
```

**Token Structure**:
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "key-id"
  },
  "payload": {
    "sub": "user-uuid",
    "email": "john.doe@company.com",
    "isAdmin": false,
    "iss": "https://openmetadata.company.com",
    "aud": "openmetadata",
    "exp": 1698624000,
    "iat": 1698620400
  },
  "signature": "..."
}
```

---

## Authorization

### 1. **Role-Based Access Control (RBAC)**

```mermaid
flowchart TB
    subgraph "Roles"
        ADMIN[Admin Role<br/>Full Access]
        STEWARD[Data Steward<br/>Manage Governance]
        OWNER[Data Owner<br/>Manage Owned Assets]
        CONSUMER[Data Consumer<br/>View Only]
    end
    
    subgraph "Permissions"
        P1[Create]
        P2[View]
        P3[Edit]
        P4[Delete]
        P5[Manage Users]
        P6[Manage Policies]
    end
    
    subgraph "Resources"
        R1[Tables]
        R2[Dashboards]
        R3[Pipelines]
        R4[Glossaries]
        R5[Tags]
    end
    
    ADMIN --> P1
    ADMIN --> P2
    ADMIN --> P3
    ADMIN --> P4
    ADMIN --> P5
    ADMIN --> P6
    
    STEWARD --> P2
    STEWARD --> P3
    STEWARD --> P6
    
    OWNER --> P2
    OWNER --> P3
    
    CONSUMER --> P2
    
    P1 --> R1
    P1 --> R2
    P1 --> R3
    P2 --> R1
    P2 --> R2
    P2 --> R3
    P3 --> R1
    P3 --> R2
    P4 --> R1
    P6 --> R4
    P6 --> R5
    
    style ADMIN fill:#ffe6e6
    style STEWARD fill:#e6f3ff
    style OWNER fill:#e1f5e1
    style CONSUMER fill:#fff4e6
```

**Built-in Roles**:

| Role | Permissions | Use Case |
|------|-------------|----------|
| **Admin** | Full access to all resources | System administrators |
| **Data Steward** | Manage governance, policies, tags | Governance team |
| **Data Owner** | Manage owned data assets | Asset owners |
| **Data Consumer** | View metadata, search | Analysts, engineers |

**Custom Role Example**:
```json
{
  "name": "DataEngineer",
  "displayName": "Data Engineer",
  "description": "Role for data engineering team",
  "policies": [
    {
      "name": "PipelineManagement",
      "policyType": "AccessControl",
      "rules": [
        {
          "name": "ManagePipelines",
          "resources": ["pipeline"],
          "operations": ["Create", "ViewAll", "EditAll", "Delete"],
          "effect": "allow"
        }
      ]
    }
  ]
}
```

---

### 2. **Attribute-Based Access Control (ABAC)**

Policy-based access control using attributes.

```mermaid
flowchart TB
    A[Access Request] --> B{Evaluate Policy}
    
    B --> C[User Attributes<br/>- Role: DataAnalyst<br/>- Team: Finance<br/>- Location: US]
    B --> D[Resource Attributes<br/>- Owner: Finance Team<br/>- Classification: Confidential<br/>- Region: US]
    B --> E[Environment Attributes<br/>- Time: Business Hours<br/>- Network: Corporate VPN]
    
    C --> F{Policy Engine}
    D --> F
    E --> F
    
    F -->|Match| G[ALLOW]
    F -->|No Match| H[DENY]
    
    style B fill:#e6f3ff
    style F fill:#e1f5e1
    style G fill:#d4edda
    style H fill:#ffe6e6
```

**Policy Example**:
```json
{
  "name": "FinanceDataAccessPolicy",
  "description": "Finance team can access finance data",
  "policyType": "AccessControl",
  "rules": [
    {
      "name": "FinanceTeamAccess",
      "resources": ["table"],
      "resourceCondition": "resource.owner.name == 'finance_team'",
      "operations": ["ViewAll"],
      "userCondition": "user.teams.contains('finance_team')",
      "effect": "allow"
    }
  ]
}
```

**Condition Expressions**:
```python
# User-based conditions
user.isAdmin == true
user.teams.contains('data_engineering')
user.email.endsWith('@company.com')

# Resource-based conditions
resource.tags.contains('PII.Sensitive')
resource.owner.id == user.id
resource.database.name == 'finance_db'

# Environment-based conditions
currentTime.hour >= 9 && currentTime.hour <= 18
request.ipAddress.startsWith('10.0.')
```

---

### 3. **Column-Level Security**

```mermaid
flowchart LR
    subgraph "Table: customers"
        C1[customer_id<br/>PUBLIC]
        C2[email<br/>PII - MASKED]
        C3[ssn<br/>PII - RESTRICTED]
        C4[purchase_amount<br/>PUBLIC]
    end
    
    subgraph "Access Control"
        U1[Admin<br/>View All] --> C1
        U1 --> C2
        U1 --> C3
        U1 --> C4
        
        U2[Analyst<br/>Masked PII] --> C1
        U2 -.Masked.-> C2
        U2 --> C4
        
        U3[Viewer<br/>Public Only] --> C1
        U3 --> C4
    end
    
    style C3 fill:#ffe6e6
    style C2 fill:#fff4e6
    style C1 fill:#e1f5e1
```

**Masking Rules**:
```json
{
  "name": "EmailMaskingRule",
  "description": "Mask email addresses for non-admin users",
  "maskingExpressions": [
    {
      "name": "EmailMask",
      "expression": "MASK_EMAIL(column)",
      "columnNames": ["email", "contact_email"]
    }
  ],
  "condition": "user.role != 'Admin'"
}
```

**Masking Functions**:
- `MASK_EMAIL`: john.doe@company.com → j***@company.com
- `MASK_PHONE`: +1-555-1234 → +1-***-1234
- `MASK_SSN`: 123-45-6789 → ***-**-6789
- `MASK_CREDIT_CARD`: 1234-5678-9012-3456 → ****-****-****-3456
- `NULLIFY`: Returns NULL
- `HASH`: Returns hash value

---

## Data Protection

### 1. **Encryption at Rest**

```yaml
# MySQL/PostgreSQL Encryption
database:
  encryption:
    enabled: true
    algorithm: "AES-256"
    keyManagement: "AWS-KMS"  # or Azure Key Vault, GCP KMS
    keyId: "arn:aws:kms:region:account:key/key-id"
```

### 2. **Encryption in Transit**

```yaml
# TLS Configuration
server:
  ssl:
    enabled: true
    protocol: "TLSv1.3"
    ciphers:
      - "TLS_AES_128_GCM_SHA256"
      - "TLS_AES_256_GCM_SHA384"
      - "TLS_CHACHA20_POLY1305_SHA256"
    certificatePath: "/path/to/cert.pem"
    keyPath: "/path/to/key.pem"
```

### 3. **PII Detection & Classification**

```mermaid
flowchart TB
    A[Data Source] --> B[Ingestion]
    B --> C[PII Scanner]
    
    C --> D{Pattern Matching}
    D -->|Email Pattern| E[Tag: PII.Email]
    D -->|SSN Pattern| F[Tag: PII.SSN]
    D -->|Phone Pattern| G[Tag: PII.Phone]
    D -->|Credit Card| H[Tag: PII.CreditCard]
    
    E --> I[Apply Masking Policy]
    F --> I
    G --> I
    H --> I
    
    I --> J[Store Metadata]
    
    style C fill:#e6f3ff
    style I fill:#e1f5e1
```

**PII Detection Rules**:
```python
PII_PATTERNS = {
    "EMAIL": r"[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}",
    "SSN": r"\d{3}-\d{2}-\d{4}",
    "PHONE": r"\+?1?\d{9,15}",
    "CREDIT_CARD": r"\d{4}[- ]?\d{4}[- ]?\d{4}[- ]?\d{4}",
    "IP_ADDRESS": r"\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}"
}
```

**Auto-Tagging**:
```json
{
  "piiClassification": {
    "enabled": true,
    "confidence": 0.8,
    "autoTag": true,
    "patterns": [
      {
        "name": "email",
        "regex": "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}",
        "tag": "PII.Email"
      },
      {
        "name": "ssn",
        "regex": "\\d{3}-\\d{2}-\\d{4}",
        "tag": "PII.SSN"
      }
    ]
  }
}
```

---

## Audit & Compliance

### 1. **Audit Logging**

```mermaid
flowchart LR
    A[User Action] --> B[Generate Audit Event]
    B --> C[Audit Log Store]
    C --> D[Elasticsearch]
    C --> E[S3/GCS Archive]
    
    D --> F[Real-time Monitoring]
    E --> G[Long-term Retention]
    
    F --> H[Security Alerts]
    G --> I[Compliance Reports]
    
    style B fill:#e6f3ff
    style C fill:#e1f5e1
    style H fill:#ffe6e6
```

**Audit Event Structure**:
```json
{
  "eventId": "uuid",
  "eventType": "entityUpdated",
  "entityType": "table",
  "entityId": "table-uuid",
  "entityFQN": "snowflake.sales_db.public.customer_orders",
  "userName": "john.doe",
  "userEmail": "john.doe@company.com",
  "timestamp": 1698624000000,
  "ipAddress": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "action": "UPDATE",
  "changeDescription": {
    "fieldsUpdated": [
      {
        "name": "description",
        "oldValue": "Customer orders",
        "newValue": "Customer orders table"
      }
    ]
  },
  "result": "SUCCESS"
}
```

**Audited Events**:
- User login/logout
- Entity CRUD operations
- Permission changes
- Policy updates
- Data access
- Failed access attempts
- Configuration changes

### 2. **Compliance Frameworks**

```mermaid
mindmap
  root((Compliance))
    GDPR
      Right to Access
      Right to Deletion
      Data Portability
      Consent Management
    HIPAA
      PHI Protection
      Access Controls
      Audit Trails
      Encryption
    SOC 2
      Security
      Availability
      Confidentiality
      Privacy
    CCPA
      Consumer Rights
      Data Inventory
      Disclosure
      Opt-out
```

#### GDPR Compliance

**Features**:
- ✅ Data inventory and classification
- ✅ PII identification and tagging
- ✅ Access control and audit logs
- ✅ Data lineage tracking
- ✅ Right to be forgotten (delete operations)
- ✅ Consent management

**Implementation**:
```python
# Find all PII data
tables_with_pii = metadata.search_entities(
    entity_type=Table,
    query="tags:PII.*"
)

# Track data lineage for GDPR requests
lineage = metadata.get_lineage(
    entity=Table,
    fqn="customer_data.users",
    downstream_depth=5
)

# Delete user data (Right to be forgotten)
def gdpr_delete_user(user_email):
    # Find all tables with user data
    tables = find_tables_with_email(user_email)
    
    # Generate deletion SQL
    for table in tables:
        sql = f"DELETE FROM {table.fqn} WHERE email = '{user_email}'"
        execute_sql(sql)
        
        # Log deletion
        audit_log(f"GDPR deletion: {user_email} from {table.fqn}")
```

#### HIPAA Compliance

**Features**:
- ✅ PHI (Protected Health Information) identification
- ✅ Encryption at rest and in transit
- ✅ Access controls and audit trails
- ✅ Data retention policies
- ✅ Incident response

**PHI Tagging**:
```json
{
  "tagCategory": "PHI",
  "tags": [
    "PHI.PatientName",
    "PHI.MedicalRecord",
    "PHI.DateOfBirth",
    "PHI.SSN",
    "PHI.Diagnosis",
    "PHI.Treatment"
  ]
}
```

---

## Security Best Practices

### 1. **Network Security**

```mermaid
flowchart TB
    subgraph "Public Zone"
        INTERNET[Internet]
        CDN[CDN]
        LB[Load Balancer]
    end
    
    subgraph "DMZ"
        WAF[WAF]
        PROXY[Reverse Proxy]
    end
    
    subgraph "Application Zone"
        APP1[API Server 1]
        APP2[API Server 2]
        APP3[API Server 3]
    end
    
    subgraph "Data Zone"
        DB[(Database)]
        ES[(Elasticsearch)]
        CACHE[(Redis)]
    end
    
    INTERNET --> CDN
    CDN --> LB
    LB --> WAF
    WAF --> PROXY
    PROXY --> APP1
    PROXY --> APP2
    PROXY --> APP3
    
    APP1 --> DB
    APP2 --> DB
    APP3 --> DB
    APP1 --> ES
    APP2 --> ES
    APP3 --> ES
    APP1 --> CACHE
    
    style WAF fill:#ffe6e6
    style DB fill:#e6f3ff
    style APP1 fill:#e1f5e1
```

**Security Checklist**:
- ✅ Use VPC/VNet isolation
- ✅ Configure firewall rules
- ✅ Enable DDoS protection
- ✅ Use private endpoints for databases
- ✅ Implement WAF rules
- ✅ Regular security scanning

### 2. **Application Security**

```yaml
# Security Headers
server:
  securityHeaders:
    - "Strict-Transport-Security: max-age=31536000; includeSubDomains"
    - "X-Content-Type-Options: nosniff"
    - "X-Frame-Options: DENY"
    - "X-XSS-Protection: 1; mode=block"
    - "Content-Security-Policy: default-src 'self'"
```

**Security Measures**:
- ✅ Input validation on all endpoints
- ✅ SQL injection prevention (parameterized queries)
- ✅ XSS protection (output encoding)
- ✅ CSRF tokens
- ✅ Rate limiting
- ✅ Dependency scanning
- ✅ Regular security updates

### 3. **Access Control**

**Principle of Least Privilege**:
```mermaid
flowchart TB
    A[New User] --> B{Role Required?}
    B -->|View Only| C[Data Consumer]
    B -->|Create/Edit| D[Data Owner]
    B -->|Governance| E[Data Steward]
    B -->|Full Access| F[Admin]
    
    C --> G[Grant Minimum<br/>Permissions]
    D --> G
    E --> G
    F --> H[Require MFA]
    
    G --> I[Regular Review]
    H --> I
    
    style G fill:#e6f3ff
    style H fill:#ffe6e6
    style I fill:#fff4e6
```

**Best Practices**:
- ✅ Principle of least privilege
- ✅ Regular access reviews
- ✅ Automated access provisioning
- ✅ Multi-factor authentication
- ✅ Session timeout policies
- ✅ Remove inactive accounts

### 4. **Secrets Management**

```yaml
# Using External Secrets Manager
secretsManager:
  provider: "aws-secrets-manager"  # or azure-key-vault, gcp-secret-manager
  region: "us-east-1"
  secrets:
    - name: "openmetadata/db-password"
      key: "database.password"
    - name: "openmetadata/jwt-secret"
      key: "jwt.secret"
```

**Secrets Best Practices**:
- ✅ Never store secrets in code
- ✅ Use secrets manager (AWS Secrets Manager, Azure Key Vault, etc.)
- ✅ Rotate secrets regularly
- ✅ Encrypt secrets at rest
- ✅ Audit secret access
- ✅ Use short-lived credentials

---

## Security Monitoring

### 1. **Security Metrics**

```mermaid
flowchart LR
    A[Security Events] --> B[Metrics Collection]
    
    B --> C[Failed Logins]
    B --> D[Unauthorized Access]
    B --> E[Policy Violations]
    B --> F[Anomalous Behavior]
    
    C --> G[Alert Manager]
    D --> G
    E --> G
    F --> G
    
    G --> H[PagerDuty]
    G --> I[Slack]
    G --> J[Email]
    G --> K[SIEM]
    
    style G fill:#ffe6e6
    style K fill:#e6f3ff
```

**Key Metrics**:
- Failed authentication attempts
- Unauthorized access attempts
- Policy violations
- Data access patterns
- Privilege escalation attempts
- Anomalous user behavior

### 2. **Incident Response**

```mermaid
flowchart TB
    A[Security Alert] --> B{Severity}
    
    B -->|Critical| C[Immediate Response]
    B -->|High| D[Response within 1 hour]
    B -->|Medium| E[Response within 4 hours]
    B -->|Low| F[Response within 24 hours]
    
    C --> G[Investigate]
    D --> G
    E --> G
    F --> G
    
    G --> H{Confirmed Incident?}
    H -->|Yes| I[Containment]
    H -->|No| J[Close Alert]
    
    I --> K[Eradication]
    K --> L[Recovery]
    L --> M[Post-Incident Review]
    
    style A fill:#ffe6e6
    style I fill:#e6f3ff
    style M fill:#e1f5e1
```

---

## References

- **Security Guide**: https://docs.open-metadata.org/v1.10.x/deployment/security
- **RBAC Documentation**: https://docs.open-metadata.org/v1.10.x/deployment/security/enable-security
- **Compliance**: https://docs.open-metadata.org/v1.10.x/deployment/security/compliance

---

**Last Updated**: October 29, 2025  
**OpenMetadata Version**: 1.10.3
