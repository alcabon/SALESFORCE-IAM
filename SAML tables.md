| Scenario | Identity Provider (IdP) | Service Provider (SP) | Who Authenticates | Who Issues Assertions | JIT Provisioning Location | Certificate Requirements | Use Cases |
|----------|-------------------------|----------------------|-------------------|----------------------|---------------------------|------------------------|-----------|
| **Salesforce as IdP** | Salesforce | External App (AWS, Office 365, Custom App) | Salesforce authenticates users | Salesforce issues SAML assertions | External SP creates/updates users | Salesforce: Signing cert<br/>SP: Validation cert | • Employee portal access<br/>• Third-party app integration<br/>• Single logout across apps |
| **Salesforce as SP** | External IdP (AD FS, Okta, Azure AD) | Salesforce | External IdP authenticates users | External IdP issues SAML assertions | Salesforce creates/updates users | IdP: Signing cert<br/>Salesforce: Validation cert | • Corporate SSO<br/>• Employee Salesforce access<br/>• Centralized identity management |
| **Federated Hub** | Primary IdP (Corporate AD) | Multiple SPs (Salesforce + Others) | Primary IdP authenticates once | Primary IdP issues to all SPs | Each SP provisions independently | Hub: Signing cert<br/>All SPs: Validation certs | • Enterprise SSO ecosystem<br/>• True single sign-on<br/>• Centralized user management |
| **Bilateral Federation** | Both systems act as IdP/SP | Both systems act as SP/IdP | Either system can authenticate | Both systems can issue assertions | Bidirectional user sync | Both: Signing + Validation certs | • Partner organizations<br/>• Cross-company collaboration<br/>• Mutual data access |
| **Multi-Org Salesforce** | Production Salesforce Org | Sandbox/Dev Orgs | Production org authenticates | Production org issues assertions | Target orgs provision users | Prod: Signing cert<br/>Sandboxes: Validation certs | • Development workflows<br/>• Testing environments<br/>• Org consolidation |
| **IdP Proxy/Gateway** | Downstream IdP (LDAP, DB) | Salesforce | Proxy validates with downstream | Proxy issues on behalf of downstream | Salesforce provisions from proxy assertions | Proxy: Signing cert<br/>Salesforce: Validation cert | • Legacy system integration<br/>• Protocol translation<br/>• Gradual IdP migration |
| **PingFederate as Central Hub** | PingFederate | Salesforce + Multiple SPs | PingFederate authenticates via adapters | PingFederate issues to all SPs | Each SP provisions independently | PingFederate: Signing cert<br/>All SPs: Validation certs | • Enterprise federation hub<br/>• Protocol bridging (SAML/OAuth/OIDC)<br/>• Complex authentication policies |
| **PingFederate + AD Federation** | PingFederate (front-end)<br/>Active Directory (back-end) | Salesforce | AD authenticates via PingFed adapter | PingFederate issues SAML assertions | Salesforce JIT from PingFed attributes | PingFederate: Signing cert<br/>Salesforce: Validation cert | • Windows integrated auth<br/>• Step-up authentication<br/>• MFA enforcement layer |
| **Salesforce → PingFederate Bridge** | Salesforce | PingFederate SP connections | Salesforce authenticates users | Salesforce issues to PingFederate | PingFederate provisions from SF assertions | Salesforce: Signing cert<br/>PingFederate: Validation cert | • Salesforce as employee portal<br/>• Legacy app modernization<br/>• Gradual cloud migration |
| **PingFederate Proxy Mode** | External IdP (Azure AD, Okta) | PingFederate (proxy) + Salesforce | External IdP authenticates | PingFed proxies/transforms assertions | Salesforce provisions from transformed assertions | External IdP + PingFed: Signing certs<br/>Salesforce: Validation cert | • Multi-vendor environments<br/>• Attribute transformation<br/>• Policy enforcement layer |

## Key Lifecycle Phases

### 1. **Initial Authentication**
- **IdP Role**: Validates user credentials (password, MFA, biometrics)
- **SP Role**: Redirects unauthenticated users to IdP
- **Certificate Use**: Request signing (optional), response validation

### 2. **Assertion Issuance**
- **IdP Role**: Creates SAML assertion with user attributes
- **SP Role**: Receives and validates SAML response
- **Certificate Use**: Response signing (required), assertion encryption (optional)

### 3. **User Provisioning (JIT)**
- **Create**: New user account creation based on assertion attributes
- **Update**: Existing user attribute synchronization
- **Mapping**: Federation ID linking between systems

### 4. **Session Management**
- **IdP Role**: Maintains authentication session
- **SP Role**: Maintains application session
- **Coordination**: Single logout coordination

### 5. **Certificate Lifecycle**
- **Renewal**: Automated or manual certificate rotation
- **Distribution**: Certificate sharing between IdP/SP
- **Validation**: Ongoing certificate trust verification

## PingFederate-Specific Considerations

### **PingFederate Certificate Strategies**
| Configuration | Certificate Approach | Anchored Trust Support | Operational Notes |
|--------------|---------------------|----------------------|-------------------|
| **Internal Enterprise CA** | PingFed uses corporate PKI | Full support with certificate chaining | Ideal for large enterprises with established PKI |
| **Self-Signed Certificates** | Manual trust establishment | Explicit trust only | Common in development/testing environments |
| **Public CA Certificates** | Commercial CA (DigiCert, etc.) | Universal anchored trust | Required for external partner integrations |
| **Mixed Certificate Strategy** | Different certs per SP connection | Configurable per partner | Flexible but complex to manage |

### **PingFederate Authentication Adapters**
| Adapter Type | Integration Method | Certificate Requirements | Use Cases |
|-------------|-------------------|------------------------|-----------|
| **HTML Form** | Direct credential validation | PingFed signing cert only | Simple username/password |
| **Windows Kerberos** | Integrated Windows Auth | Domain trust + PingFed cert | Corporate desktop SSO |
| **LDAP/AD** | Directory server lookup | LDAPS cert + PingFed cert | Enterprise directory integration |
| **RADIUS** | Network authentication | RADIUS shared secret + certs | Hardware token integration |
| **SAML IdP** | Federated authentication | Upstream IdP + PingFed certs | Multi-tier federation |

### **PingFederate Deployment Patterns**

**Pattern 1: Central Authentication Hub**
- PingFederate authenticates all users
- Issues SAML to Salesforce and other SPs
- Single certificate management point
- Supports anchored trust with proper CA strategy

**Pattern 2: Protocol Translation Gateway**
- Receives SAML from upstream IdP
- Transforms/enriches attributes
- Re-issues SAML to Salesforce
- Requires certificate trust from both directions

**Pattern 3: Legacy Integration Bridge**
- Connects modern SAML apps to legacy auth systems
- Protocol conversion (Kerberos → SAML)
- Certificate complexity depends on legacy constraints

| Certificate Type | Best For | Trust Model | Operational Overhead |
|-----------------|----------|-------------|---------------------|
| **Self-Signed** | Development, Internal Apps | Explicit Trust | Low Cost, Manual Trust Setup |
| **CA-Signed** | Production, External Partners | Anchored Trust | Higher Cost, Automatic Trust |
| **Enterprise CA** | Large Organizations | Hybrid Trust | Medium Cost, Centralized Management |
| **Mutual TLS** | High Security | Bidirectional Trust | High Overhead, Maximum Security |

## Federation ID Strategies

| Strategy | Implementation | Benefits | Challenges |
|----------|---------------|----------|------------|
| **Email Address** | User.Email = SAML NameID | Simple, Universal | Email changes break auth |
| **Employee ID** | User.FederationIdentifier = EmployeeID | Stable, Unique | Requires HR system sync |
| **Domain\\Username** | Custom concatenation | Windows AD compatible | Domain changes affect users |
| **UUID/GUID** | Immutable identifier | Never changes | Requires careful initial mapping |
