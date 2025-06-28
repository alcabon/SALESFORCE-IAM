# Salesforce Identity Auditing Features Comparison

| Feature | Audit Trail | Identity Provider Event Log | Login History |
|---------|-------------|------------------------------|---------------|
| **Primary Purpose** | Track setup and configuration changes across the org | Monitor identity provider authentication events and SSO activities | Track user login attempts and session details |
| **Data Retention** | 6 months (up to 2 years with add-on) | 1 day (EventLogFile) or 30 days (Event Monitoring) | 6 months |
| **Activity Types Captured** | • Field changes<br>• Record creation/deletion<br>• Permission changes<br>• Profile modifications<br>• Custom object changes<br>• Workflow rule changes | • SAML assertions<br>• OAuth token requests<br>• SSO authentication events<br>• Identity provider responses<br>• Federation errors<br>• JWT token validation | • Successful logins<br>• Failed login attempts<br>• Login IP addresses<br>• Browser/device information<br>• Session duration<br>• Logout events |
| **Access Method** | Setup → Security → View Setup Audit Trail | • Event Monitoring (paid add-on)<br>• EventLogFile API<br>• Reports & Dashboards | Setup → Users → Login History |
| **User Focus** | Admin and system-level changes | Authentication and SSO events | Individual user login behavior |
| **Real-time Monitoring** | Near real-time (within minutes) | Near real-time to 24-hour delay | Real-time to near real-time |
| **Granularity** | High - shows specific field changes and values | Medium - shows authentication flow details | Medium - shows login session details |
| **Use Cases** | • Compliance auditing<br>• Change tracking<br>• Security investigations<br>• Troubleshooting configuration issues | • SSO troubleshooting<br>• Identity provider performance<br>• Authentication security monitoring<br>• SAML/OAuth debugging | • User access monitoring<br>• Suspicious login detection<br>• Geographic access analysis<br>• Login pattern analysis |
| **Data Export** | Limited export options | API-based export, CSV downloads | Limited export options |
| **Filtering Options** | • Date range<br>• User<br>• Action type<br>• Section | • Event type<br>• Date range<br>• User<br>• Login type | • Date range<br>• User<br>• Login type<br>• Status |
| **Cost** | Included in all editions | Event Monitoring requires additional license | Included in all editions |
| **Best For** | Organizations needing detailed change tracking and compliance reporting | Organizations with complex SSO implementations requiring authentication monitoring | Organizations monitoring user access patterns and login security |

## Key Differences Summary

- **Audit Trail** focuses on *what changed* in your org configuration
- **Identity Provider Event Log** focuses on *how users authenticate* through external identity providers
- **Login History** focuses on *when and where users logged in*

Each serves a different aspect of identity and access management monitoring, and they complement each other for comprehensive security oversight.
