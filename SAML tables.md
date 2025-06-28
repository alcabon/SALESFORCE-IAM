| Scenario | Identity Provider (IdP) | Service Provider (SP) | Who Authenticates | Who Issues Assertions | JIT Provisioning Location | Certificate Requirements | Use Cases |
|----------|-------------------------|----------------------|-------------------|----------------------|---------------------------|------------------------|-----------|
| **Salesforce as IdP** | Salesforce | External App (AWS, Office 365, Custom App) | Salesforce authenticates users | Salesforce issues SAML assertions | External SP creates/updates users | Salesforce: Signing cert<br/>SP: Validation cert | • Employee portal access<br/>• Third-party app integration<br/>• Single logout across apps |
| **Salesforce as SP** | External IdP (AD FS, Okta, Azure AD) | Salesforce | External IdP authenticates users | External IdP issues SAML assertions | Salesforce creates/updates users | IdP: Signing cert<br/>Salesforce: Validation cert | • Corporate SSO<br/>• Employee Salesforce access<br/>• Centralized identity management |
| **Federated Hub** | Primary IdP (Corporate AD) | Multiple SPs (Salesforce + Others) | Primary IdP authenticates once | Primary IdP issues to all SPs | Each SP provisions independently | Hub: Signing cert<br/>All SPs: Validation certs | • Enterprise SSO ecosystem<br/>• True single sign-on<br/>• Centralized user management |
| **Bilateral Federation** | Both systems act as IdP/SP | Both systems act as SP/IdP | Either system can authenticate | Both systems can issue assertions | Bidirectional user sync | Both: Signing + Validation certs | • Partner organizations<br/>• Cross-company collaboration<br/>• Mutual data access |
| **Multi-Org Salesforce** | Production Salesforce Org | Sandbox/Dev Orgs | Production org authenticates | Production org issues assertions | Target orgs provision users | Prod: Signing cert<br/>Sandboxes: Validation certs | • Development workflows<br/>• Testing environments<br/>• Org consolidation |
| **IdP Proxy/Gateway** | Downstream IdP (LDAP, DB) | Salesforce | Proxy validates with downstream | Proxy issues on behalf of downstream | Salesforce provisions from proxy assertions | Proxy: Signing cert<br/>Salesforce: Validation cert | • Legacy system integration<br/>• Protocol translation<br/>• Gradual IdP migration |

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

## Certificate Types by Scenario

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
