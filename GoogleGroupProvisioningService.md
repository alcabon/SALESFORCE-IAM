
## Configure User Provisioning for Google Workspace Integration

Here's a comprehensive configuration for user provisioning between Salesforce and Google Workspace for an insurance company.

## Prerequisites Setup

**1. Google Workspace Admin Console Configuration**
- Enable Google Workspace Directory API
- Create a service account with domain-wide delegation
- Generate and download service account credentials (JSON file)
- Grant necessary API scopes:
  - `https://www.googleapis.com/auth/admin.directory.user`
  - `https://www.googleapis.com/auth/admin.directory.group`

**2. Salesforce Connected App Creation**
```
Setup → App Manager → New Connected App
Name: Google Workspace Integration
API Name: Google_Workspace_Integration
Contact Email: admin@insurancecompany.com
Enable OAuth Settings: ✓
Callback URL: https://login.salesforce.com/services/oauth2/callback
Selected OAuth Scopes:
- Access and manage your data (api)
- Perform requests on your behalf at any time (refresh_token, offline_access)
```

## Outbound Provisioning Configuration (Salesforce → Google Workspace)

### Step 1: Enable User Provisioning
```
Connected Apps → Google Workspace Integration → Edit
→ User Provisioning Section:
  ✓ Enable User Provisioning
  Protocol: Custom (Google Directory API)
  Provisioning Class: GoogleWorkspaceProvisioningHandler
```

### Step 2: Custom Apex Provisioning Handler### Step 3: Custom Metadata for Configuration

```
Setup → Custom Metadata Types → New
Label: Google Workspace Credentials
API Name: GoogleWorkspaceCredentials

Fields:
- Service_Account_Email__c (Text)
- Private_Key__c (Long Text Area, Encrypted)
- Project_ID__c (Text)
- Domain__c (Text)
```

### Step 4: User Object Customization

```
Setup → Object Manager → User → Fields & Relationships
New Custom Field:
- Field Label: Google Workspace ID
- Field Name: Google_Workspace_ID
- Data Type: Text(255)
- Unique: ✓
- External ID: ✓
```

## Group/Role Provisioning

### Map Salesforce Profiles to Google Groups## Trigger Setup for Real-time Provisioning

```apex
trigger UserProvisioningTrigger on User (after insert, after update) {
    
    Set<Id> usersToProvision = new Set<Id>();
    Set<Id> usersToUpdateGroups = new Set<Id>();
    
    for (User user : Trigger.new) {
        
        if (Trigger.isInsert) {
            // New user - provision to Google Workspace
            if (user.IsActive && shouldProvisionUser(user)) {
                usersToProvision.add(user.Id);
            }
        }
        
        if (Trigger.isUpdate) {
            User oldUser = Trigger.oldMap.get(user.Id);
            
            // Profile changed - update groups
            if (user.ProfileId != oldUser.ProfileId) {
                usersToUpdateGroups.add(user.Id);
            }
            
            // User activated/deactivated
            if (user.IsActive != oldUser.IsActive) {
                usersToProvision.add(user.Id);
            }
        }
    }
    
    if (!usersToProvision.isEmpty()) {
        GoogleWorkspaceProvisioningService.provisionUsers(usersToProvision);
    }
    
    if (!usersToUpdateGroups.isEmpty()) {
        GoogleGroupProvisioningService.syncUserGroups(usersToUpdateGroups);
    }
}
```

## Monitoring and Error Handling

### Setup Monitoring Dashboard

```
Reports & Dashboards → New Dashboard: "Google Workspace Provisioning"

Components:
1. Users Provisioned Today (COUNT)
2. Provisioning Errors (LIST)
3. Active Google Workspace Users (COUNT)
4. Users by Department Distribution
```

### Error Logging

```apex
public class ProvisioningErrorLogger {
    
    public static void logError(String operation, Id userId, String errorMessage) {
        Provisioning_Log__c log = new Provisioning_Log__c(
            User__c = userId,
            Operation__c = operation,
            Error_Message__c = errorMessage,
            Timestamp__c = System.now(),
            Status__c = 'Failed'
        );
        
        insert log;
        
        // Send notification for critical errors
        if (errorMessage.contains('Authentication') || errorMessage.contains('Permission')) {
            sendCriticalErrorNotification(log);
        }
    }
}
```

## Security Best Practices

1. **Store credentials securely** in Protected Custom Settings or Custom Metadata
2. **Use service accounts** with minimal required permissions
3. **Implement retry logic** for transient failures
4. **Monitor provisioning logs** regularly
5. **Set up alerts** for failed provisioning attempts
6. **Regular audit** of provisioned users and groups
7. **Implement proper exception handling** in all provisioning methods

This configuration provides a comprehensive user provisioning solution between Salesforce and Google Workspace, tailored for an insurance company's organizational structure and security requirements.

```apex
public class GoogleGroupProvisioningService {
    
    // Map Salesforce Profiles to Google Groups for insurance company
    private static final Map<String, String> PROFILE_TO_GROUP_MAPPING = new Map<String, String>{
        'Insurance Agent' => 'agents@insurancecompany.com',
        'Claims Adjuster' => 'claims@insurancecompany.com',
        'Underwriter' => 'underwriters@insurancecompany.com',
        'Insurance Manager' => 'management@insurancecompany.com',
        'Customer Service' => 'customer-service@insurancecompany.com',
        'Sales Manager' => 'sales-managers@insurancecompany.com'
    };
    
    @future(callout=true)
    public static void syncUserGroups(Set<Id> userIds) {
        
        List<User> users = [SELECT Id, ProfileId, Profile.Name, Email, Google_Workspace_ID__c 
                           FROM User 
                           WHERE Id IN :userIds 
                           AND Google_Workspace_ID__c != null];
        
        for (User user : users) {
            syncUserToGroups(user);
        }
    }
    
    private static void syncUserToGroups(User user) {
        String googleGroupEmail = PROFILE_TO_GROUP_MAPPING.get(user.Profile.Name);
        
        if (String.isNotBlank(googleGroupEmail)) {
            addUserToGroup(user.Google_Workspace_ID__c, googleGroupEmail);
        }
        
        // Add user to department-specific groups
        addToDepartmentGroups(user);
        
        // Add user to company-wide groups
        addToCompanyGroups(user);
    }
    
    private static void addUserToGroup(String googleUserId, String groupEmail) {
        try {
            Map<String, Object> memberPayload = new Map<String, Object>{
                'email' => getUserEmailFromGoogleId(googleUserId),
                'role' => 'MEMBER'
            };
            
            HttpResponse response = GoogleWorkspaceAPI.callAPI(
                'POST', 
                '/admin/directory/v1/groups/' + groupEmail + '/members', 
                memberPayload
            );
            
            if (response.getStatusCode() == 200 || response.getStatusCode() == 409) {
                // Success or already member
                System.debug('User added to group: ' + groupEmail);
            } else {
                System.debug('Failed to add user to group: ' + response.getBody());
            }
            
        } catch (Exception e) {
            System.debug('Error adding user to group: ' + e.getMessage());
        }
    }
    
    private static void addToDepartmentGroups(User user) {
        // Add to department-specific distribution lists
        String departmentGroup = 'dept-' + user.Profile.Name.toLowerCase().replace(' ', '-') + '@insurancecompany.com';
        addUserToGroup(user.Google_Workspace_ID__c, departmentGroup);
    }
    
    private static void addToCompanyGroups(User user) {
        // Add all employees to company-wide groups
        addUserToGroup(user.Google_Workspace_ID__c, 'all-employees@insurancecompany.com');
        addUserToGroup(user.Google_Workspace_ID__c, 'company-announcements@insurancecompany.com');
    }
    
    private static String getUserEmailFromGoogleId(String googleUserId) {
        // Query Google Directory API to get user email
        HttpResponse response = GoogleWorkspaceAPI.callAPI(
            'GET', 
            '/admin/directory/v1/users/' + googleUserId, 
            null
        );
        
        if (response.getStatusCode() == 200) {
            Map<String, Object> userInfo = (Map<String, Object>) JSON.deserializeUntyped(response.getBody());
            return (String) userInfo.get('primaryEmail');
        }
        
        return null;
    }
}
```
