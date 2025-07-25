# Google Workspace Audit Logs Investigation Procedure

## Overview

**Data Source:** Google Workspace Admin Audit Logs  
**Environment:** Google Workspace  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Data Source Description

Google Workspace Audit Logs provide detailed records of administrative and user activities within the Google Workspace environment. These logs capture actions such as user account changes, role assignments, security setting modifications, email configuration changes, and other critical activities. This data source is essential for security investigations, compliance monitoring, and understanding changes to the Google Workspace environment.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Elastic | [Google Workspace Index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover) | Okta SSO | Primary method for investigations |
| Google Admin Console | [Admin Console Reports](https://admin.google.com/ac/reporting/audit) | Google Workspace Admin credentials | Direct access to logs |
| Harmony | [Checkpoint Harmony Portal](https://portal.checkpoint.com/dashboard/email&collaboration/) | Harmony credentials | For email-specific investigations |

### Required Permissions

- Elastic access with appropriate permissions
- For Google Admin Console: Super Admin role or custom role with Reports > Audit and Activity Reports permissions
- For direct API access: Service account with Reports API scope

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| Elastic | logs-google_workspace index | 90 days | Primary search interface |
| Google Admin Console | Admin > Reports > Audit | 6 months | Native Google interface |
| Google Cloud Storage | Configured export buckets | Configurable | For long-term retention |

## Common Investigation Scenarios

### Scenario 1: Admin Role Assignment Investigation

#### Query Examples

```
event.dataset:google_workspace.admin AND 
event.action:("ASSIGN_ROLE" OR "CREATE_ROLE_ASSIGNMENT") AND 
google_workspace.admin.role.name:*Admin*
```

**Fields of interest:**
- `google_workspace.admin.role.name`: The assigned admin role
- `google_workspace.admin.user.email`: User who received the role
- `source.user.email`: User who performed the assignment
- `@timestamp`: When the action occurred

#### Interpretation Guidelines

- Check if the role assignment was preceded by notification in security channels
- Review the appropriateness of the assigned role for the user's function
- Look for unusual timing or source of the assignment action
- Correlate with other administrative actions by the same user

### Scenario 2: Email Forwarding Rule Investigation

#### Query Examples

```
event.dataset:google_workspace.login AND 
event.action:("FORWARDING_ENABLED" OR "EMAIL_FORWARDING_RULE_CREATION") AND
google_workspace.login.is_second_party:false
```

**Fields of interest:**
- `google_workspace.login.forwarding_email`: Destination email for forwarding
- `user.email`: User who created the forwarding rule
- `source.ip`: IP address of the user who created the rule
- `source.geo.country_name`: Country from which the rule was created

#### Interpretation Guidelines

- Verify if the forwarding domain is an approved business partner
- Check for forwarding to personal email domains (gmail.com, outlook.com, etc.)
- Review what emails may have been forwarded after rule creation
- Assess if sensitive information could have been exposed

### Scenario 3: Security Setting Modifications

#### Query Examples

```
event.dataset:google_workspace.admin AND 
event.action:("CHANGE_APPLICATION_SETTING" OR "CHANGE_2SV_REQUIREMENT") AND
google_workspace.admin.setting.name:*security*
```

**Fields of interest:**
- `google_workspace.admin.setting.name`: The specific setting modified
- `google_workspace.admin.setting.old_value`: Previous setting value
- `google_workspace.admin.setting.new_value`: New setting value
- `source.user.email`: Administrator who made the change

#### Interpretation Guidelines

- Review if security setting changes were authorized
- Check for weakening of security controls (e.g., disabling 2SV)
- Look for other security setting changes around the same time
- Verify if change management process was followed

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Okta Logs | `source.ip`, `user.email` | Correlate authentication chain |
| Cloudflare Logs | `source.ip` | Identify web access patterns |
| Harmony Email Logs | `user.email` | Review emails that may have been forwarded |
| SentinelOne Logs | `source.ip` | Correlate with endpoint activities |

### Common Correlation Techniques

1. **Authentication Chain Analysis**
   - Description: Track user authentication across multiple systems
   - Example query:
   ```
   user.email:"suspicious.user@canva.com" AND 
   @timestamp:[now-1h TO now] AND
   (event.dataset:okta.system OR event.dataset:google_workspace.login)
   ```
   - Expected outcomes: Verify legitimate authentication flow or identify bypass

2. **Email Activity Correlation**
   - Description: Link Google Workspace configuration changes with email activities
   - Example approach:
     - First, identify forwarding rule creation in Google Workspace logs
     - Then, search Harmony logs for emails received by the user
     - Review content of emails for sensitive information
   - Expected outcomes: Assess potential data exposure from email forwarding

## Data Collection for Investigations

### Evidence Collection Process

1. For Google Workspace audit investigations:
   - Search relevant logs with appropriate time range
   - Export results in CSV or JSON format
   - Document search parameters and queries
   - Create a timeline of significant events

2. For email-specific investigations:
   - Export email headers and content from Harmony
   - Capture forwarding rule configurations
   - Document all destination email addresses
   - Save a list of potentially exposed emails

3. For admin role investigations:
   - Document all role assignments and modifications
   - Export user permission histories
   - Capture before and after state of permissions
   - Record authorization evidence (tickets, approvals)

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| Elastic UI | Use Kibana export function | CSV/JSON | Best for smaller datasets |
| Google Admin Console | Reports > Download | CSV | Limited to 1000 records |
| Google Workspace API | Custom script | JSON | For large-scale export |

## Known Limitations

- Google Workspace audit logs may be delayed up to several days
- Not all user actions are captured in detail
- Login events don't always have complete context
- Historical logs older than 6 months may not be available in Admin Console
- Different log types have varying levels of detail

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing audit events | Log delay or filtering | Expand time range; check different log sources |
| Incomplete user information | Privilege limitations | Request elevated access; correlate with other logs |
| Limited historical data | Retention period constraints | Check if logs were exported to long-term storage |
| Ambiguous action attribution | Shared account usage | Correlate with IP and device information |

## Additional Resources

- [Google Workspace Admin Audit Logs Documentation](https://support.google.com/a/answer/4579579)
- [Google Workspace Reports API](https://developers.google.com/admin-sdk/reports/reference/rest)
- [Google Admin Guide](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2435809569/Google+Admin)
- [Google Admin Roles - Custom Roles](https://canvadev.atlassian.net/wiki/spaces/IT/pages/2428437190/Google+Admin+Roles+-+Custom+Roles)

## Related SOPs

- [Google Workspace User Containment](../../sops/google-workspace-user-containment-sop.md)
- [Admin Access Review Process](../../sops/admin-access-review-sop.md)