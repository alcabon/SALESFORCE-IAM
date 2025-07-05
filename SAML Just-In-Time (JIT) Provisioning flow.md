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
Great question! Here's how SAML triggers the JIT handler and how it's configured:

## **Configuration Steps**

### **1. Single Sign-On Settings**
In Salesforce Setup → Single Sign-On Settings:
- **Just-in-Time User Provisioning**: Enable this checkbox
- **User Provisioning Class**: Specify your custom Apex class name (e.g., `CustomJITHandler`)

### **2. SAML Identity Provider Settings**
Configure the SAML Identity Provider with:
- **Entity ID**: Identity Provider's unique identifier
- **SAML Login URL**: Where users authenticate
- **X.509 Certificate**: For validating SAML assertions
- **Attribute Mappings**: Map SAML attributes to Salesforce user fields

### **3. Custom JIT Handler Class**
Create an Apex class that implements `Auth.SamlJitHandler`:

```apex
global class CustomJITHandler implements Auth.SamlJitHandler {
    
    global User createUser(Id samlSsoProviderId, Id communityId, Id portalId,
                          String federationIdentifier, Map<String, String> attributes, String assertion) {
        // Create new user logic
        User u = new User();
        u.Username = attributes.get('email');
        u.Email = attributes.get('email');
        u.FirstName = attributes.get('firstName');
        u.LastName = attributes.get('lastName');
        // Set other required fields
        return u;
    }
    
    global User updateUser(Id userId, Id samlSsoProviderId, Id communityId, Id portalId,
                          String federationIdentifier, Map<String, String> attributes, String assertion) {
        // Update existing user logic
        User u = [SELECT Id, Email FROM User WHERE Id = :userId];
        u.Email = attributes.get('email');
        // Update other fields as needed
        return u;
    }
}
```

## **How the Trigger Works**

**Automatic Triggering:**
1. **SAML Assertion Processing** - When Salesforce receives a valid SAML assertion
2. **User Lookup** - Salesforce checks if user exists (based on Federation ID or Username)
3. **JIT Handler Invocation**:
   - If user doesn't exist → `createUser()` method called
   - If user exists → `updateUser()` method called
4. **Attribute Processing** - SAML attributes are passed to the handler
5. **User Provisioning** - Handler creates/updates user and returns User object

**Key Configuration Points:**
- **Federation ID Mapping**: Must be consistent between IdP and Salesforce
- **Attribute Mapping**: SAML attributes must map to Salesforce user fields
- **Profile Assignment**: JIT handler must assign appropriate Profile
- **Required Fields**: All required User fields must be populated

The JIT handler is automatically triggered by the SAML SSO process - no manual intervention needed once configured!
