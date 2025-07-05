
I'll create a mermaid diagram showing the Salesforce Experience Cloud (Community) self-registration process and how contacts and person accounts are created/needed.This diagram illustrates the Salesforce Experience Cloud self-registration process and shows:

**Key Decision Points:**
1. **Person Accounts Enabled** - Determines available options
2. **Default Account Configured** - Affects how accounts/contacts are created

**Possible Outcomes:**
- **Person Account Creation** (blue) - When Person Accounts are enabled and no default account is configured
- **Contact with Default Account** (purple) - When a default account is specified
- **Registration Failure** (red) - When neither Person Accounts nor default account are properly configured
- **Success** (green) - User can access the community

**Important Notes:**
- Person Accounts act as both Account and Contact records
- Community users always need an Account-Contact relationship
- Without proper configuration, registration will fail
- Person Accounts provide the most flexible solution for individual customer registration

This visual shows why Person Accounts are often the preferred solution for B2C communities where customers self-register individually.

```mermaid
flowchart TD
    A[User Visits Community Self-Registration Page] --> B{Person Accounts Enabled?}
    
    B -->|Yes| C[User Fills Registration Form]
    B -->|No| D{Default Account Configured?}
    
    D -->|Yes| E[User Fills Registration Form]
    D -->|No| F[Registration Fails - Error]
    
    C --> G{Default Account Configured?}
    
    G -->|Yes| H[Create Contact Record<br/>Link to Default Account]
    G -->|No| I[Create Person Account Record<br/>Acts as both Account & Contact]
    
    E --> J[Create Contact Record<br/>Link to Default Account]
    
    H --> K[Create Community User Record<br/>Link to Contact]
    I --> L[Create Community User Record<br/>Link to Person Account Contact]
    J --> M[Create Community User Record<br/>Link to Contact]
    
    K --> N[User Successfully Registered<br/>Can Access Community]
    L --> N
    M --> N
    
    F --> O[User Sees Error Message<br/>Cannot Complete Registration]
    
    style I fill:#e1f5fe
    style H fill:#f3e5f5
    style J fill:#f3e5f5
    style F fill:#ffebee
    style N fill:#e8f5e8
    style O fill:#ffebee
```
