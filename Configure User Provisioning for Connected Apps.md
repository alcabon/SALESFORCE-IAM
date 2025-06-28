## Configure User Provisioning for Connected Apps

User provisioning for Connected Apps enables automatic creation, update, and deactivation of user accounts between Salesforce and external applications using standardized protocols.

## Overview

**What is User Provisioning?**
- Automated process to manage user lifecycle across systems
- Eliminates manual user account creation/maintenance
- Ensures consistent user data across applications
- Supports both inbound and outbound provisioning

## Provisioning Protocols

**1. SCIM (System for Cross-domain Identity Management)**
- Industry standard for user provisioning
- RESTful API-based protocol
- Supports user and group management
- Version 2.0 is most commonly used

**2. SAML Just-in-Time (JIT) Provisioning**
- Creates users during SAML authentication
- Uses SAML assertion attributes
- Lighter weight than SCIM
- Good for initial user creation

## Configuration Steps

### For Outbound Provisioning (Salesforce → External App)

**1. Enable User Provisioning on Connected App**
```
Setup → App Manager → [Your Connected App] → Edit
→ Enable User Provisioning
→ Select provisioning protocol (SCIM 2.0)
```

**2. Configure SCIM Settings**
- **SCIM Base URL**: External app's SCIM endpoint
- **Authentication**: Bearer token, OAuth, or Basic Auth
- **User Mapping**: Map Salesforce fields to SCIM attributes

**3. Set Up User Mapping**
```
Salesforce Field → SCIM Attribute
Username → userName
Email → emails[0].value
FirstName → name.givenName
LastName → name.familyName
IsActive → active
```

**4. Configure Provisioning Policies**
- **Create Users**: When to create new accounts
- **Update Users**: Which field changes trigger updates
- **Deactivate Users**: When to disable accounts

### For Inbound Provisioning (External App → Salesforce)

**1. Create SAML SSO Configuration**
```
Setup → Single Sign-On Settings → SAML Single Sign-On Settings
→ New → Configure Identity Provider details
```

**2. Configure JIT Handler**
```apex
public class CustomJITHandler implements Auth.SamlJitHandler {
    public User createUser(Id samlSsoProviderId, Id communityId, Id portalId,
                          String federationIdentifier, Map<String, String> attributes,
                          String assertion) {
        User u = new User();
        u.Username = attributes.get('username');
        u.Email = attributes.get('email');
        u.FirstName = attributes.get('firstname');
        u.LastName = attributes.get('lastname');
        // Set required fields and insert user
        return u;
    }
    
    public void updateUser(Id userId, Id samlSsoProviderId, Id communityId, Id portalId,
                          String federationIdentifier, Map<String, String> attributes,
                          String assertion) {
        User u = [SELECT Id FROM User WHERE Id = :userId];
        // Update user fields based on attributes
        update u;
    }
}
```

## Advanced Configuration Options

**Conditional Provisioning**
- Use Flow or Apex to add business logic
- Provision based on user attributes or roles
- Different provisioning rules per user type

**Bulk Provisioning**
- Initial data sync for existing users
- Scheduled batch jobs for maintenance
- Error handling and retry mechanisms

**Group/Role Provisioning**
- Map Salesforce permission sets to external groups
- Automatic role assignment based on attributes
- Dynamic group membership updates

## Security Considerations

**Authentication Methods**
- OAuth 2.0 with refresh tokens (recommended)
- Bearer tokens with rotation
- Basic authentication (less secure)

**Data Protection**
- Encrypt sensitive data in transit
- Implement proper access controls
- Regular security audits and monitoring

**Error Handling**
- Implement retry logic for failed provisions
- Monitor provisioning logs and failures
- Set up alerts for critical errors

## Monitoring and Troubleshooting

**Login History Analysis**
- Track provisioned user login patterns
- Identify provisioning failures
- Monitor authentication success rates

**Setup Audit Trail**
- Review provisioning configuration changes
- Track permission modifications
- Audit user creation/updates

**Debug Logs**
- Enable debug logs for JIT handlers
- Review SCIM API call logs
- Analyze error messages and responses

## Best Practices

1. **Start with JIT provisioning** for simpler implementations
2. **Use SCIM for complex scenarios** requiring full lifecycle management
3. **Implement proper error handling** and logging
4. **Test thoroughly** with different user scenarios
5. **Monitor provisioning metrics** regularly
6. **Keep attribute mappings simple** initially
7. **Plan for user deprovisioning** and cleanup processes

This configuration enables seamless user management across your identity ecosystem while maintaining security and compliance requirements.
