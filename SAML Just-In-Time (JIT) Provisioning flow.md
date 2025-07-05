I've created a comprehensive Mermaid sequence diagram showing the SAML Just-In-Time (JIT) Provisioning flow. The diagram illustrates:

**Key Steps:**
1. **User Access Attempt** - User tries to access Salesforce
2. **Authentication Redirect** - Salesforce redirects to Identity Provider
3. **User Authentication** - User authenticates with IdP
4. **SAML Assertion** - IdP sends user attributes to Salesforce
5. **JIT Processing** - Custom JIT handler processes the user data
6. **Dynamic Provisioning** - User is either created or updated in Salesforce
7. **Access Granted** - User gains access to Salesforce

**JIT Handler Logic:**
- **New User**: Creates user account with attributes from SAML assertion
- **Existing User**: Updates user attributes with current information from IdP

This flow ensures users are automatically provisioned or updated in real-time during the authentication process, eliminating the need for manual user management while keeping user data synchronized with the authoritative identity source.

```mermaid
sequenceDiagram
    participant User
    participant SP as Service Provider<br/>(Salesforce)
    participant IdP as Identity Provider
    participant JIT as JIT Handler<br/>(Custom Logic)
    participant SF_DB as Salesforce<br/>Database

    Note over User, SF_DB: SAML JIT Provisioning Flow

    User->>SP: 1. Attempts to access Salesforce
    SP->>IdP: 2. Redirects to IdP for authentication
    
    Note over IdP: User authenticates with IdP
    IdP->>User: 3. User provides credentials
    User->>IdP: 4. Authentication successful
    
    IdP->>SP: 5. Sends SAML Assertion with user attributes
    
    Note over SP: Salesforce receives SAML assertion
    SP->>JIT: 6. Triggers JIT Handler with user attributes
    
    Note over JIT: JIT Handler processes user data
    JIT->>SF_DB: 7. Check if user exists in Salesforce
    
    alt User does not exist
        JIT->>SF_DB: 8a. Create new user with attributes
        SF_DB->>JIT: 9a. User created successfully
    else User exists
        JIT->>SF_DB: 8b. Update existing user with new attributes
        SF_DB->>JIT: 9b. User updated successfully
    end
    
    JIT->>SP: 10. Return provisioning result
    SP->>User: 11. Grant access to Salesforce
    
    Note over User, SF_DB: User is now authenticated and provisioned
```
