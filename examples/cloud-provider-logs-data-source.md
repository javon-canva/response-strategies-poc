# Cloud Identity Provider Logs Investigation Procedure

## Overview

**Data Source:** Cloud Identity Provider Authentication Logs (Okta, Azure AD, Google Workspace)  
**Environment:** Cloud  
**Version:** 1.0  
**Last Updated:** 2025-07-23  
**Maintainer:** Detection and Response Team

## Data Source Description

Cloud identity provider logs contain authentication events, access management activities, and administrative actions performed within identity management platforms. These logs are critical for investigating account compromises, unauthorized access attempts, and suspicious authentication patterns.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Console/UI | https://admin.provider.com | Admin credentials + MFA | Limited to past 90 days |
| API | https://api.provider.com/v1/logs | OAuth 2.0 / API Key | Allows programmatic access |
| SIEM | [SIEM URL] | SIEM credentials | Pre-integrated data feeds with parsing |

### Required Permissions

- Identity Provider Admin or Security Admin role
- SIEM Log Analyst role
- API access for automation/scripting

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| Production | SIEM Identity Provider index | 12 months | Full fidelity |
| Development | SIEM Dev Identity Provider index | 3 months | Limited data |
| Archive | Cloud Storage Bucket | 7 years | Compliance requirement |

## Common Investigation Scenarios

### Scenario 1: Suspicious Authentication from Unusual Location

#### Query Examples

```
user.name:"[username]" AND event.action:("login" OR "authenticate") 
AND NOT source.geo.country_name:("United States" OR "Australia")
| sort @timestamp desc
```

**Fields of interest:**
- `source.geo.country_name`: Geographic location of authentication attempt
- `source.ip`: Source IP address
- `source.as.organization_name`: ASN organization
- `user_agent.original`: Browser/device information
- `event.outcome`: Success or failure of authentication
- `authentication.method`: Method used (password, MFA, SSO, etc.)

#### Interpretation Guidelines

- Compare against user's travel history and normal work locations
- Consider VPN usage which may alter apparent geolocation
- Check for simultaneous logins from different locations
- Review previous successful and failed login attempts
- Correlate with HR systems for travel or remote work authorization

### Scenario 2: Brute Force Authentication Attempts

#### Query Examples

```
user.name:"[username]" AND event.action:("login" OR "authenticate") 
AND event.outcome:failure
| stats count by span(1h), source.ip
| where count > 5
```

**Fields of interest:**
- `event.outcome`: Success or failure of authentication
- `source.ip`: Source IP address
- `error.message`: Specific error returned
- `user_agent.original`: Browser/device information
- `network.direction`: Inbound or internal traffic

#### Interpretation Guidelines

- Determine if failures are followed by successful authentication
- Check if IP is associated with known scanning or attack infrastructure
- Review password policy and lockout configurations
- Identify if multiple users are targeted from same source (password spraying)
- Consider legitimate user error versus automated attack tools

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| VPN Logs | `source.ip` | Connect authentication to network access |
| Endpoint Logs | `user.name` | Link to local device activity |
| Application Access Logs | `user.name`, `session.id` | Follow activity after authentication |
| Network Flows | `source.ip`, `destination.ip` | Track network connections |

### Common Correlation Techniques

1. **Session Tracking**
   - Extract session identifiers from successful authentications
   - Follow session ID through application access logs
   - Map complete user journey from authentication to resource access
   - Example query:
   ```
   user.name:"[username]" AND session.id:"[session_id]"
   | sort @timestamp asc
   ```

2. **IP-based Activity Timeline**
   - Collect all events from suspicious source IP
   - Create chronological timeline of activities
   - Identify reconnaissance, access attempts, and post-authentication actions
   - Example query:
   ```
   source.ip:"[suspicious_ip]" AND @timestamp:[time_window]
   | sort @timestamp asc
   ```

## Data Collection for Investigations

### Evidence Collection Process

1. Document the search parameters, time ranges, and specific queries used
2. Capture raw log entries in their original format when possible
3. Include metadata (query time, data source version, etc.)
4. Export results in a forensically sound manner (CSV, JSON with hash verification)
5. Document chain of custody for all exported evidence

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| SIEM Export | Use SIEM export function | CSV/JSON | Includes query parameters |
| API Query | `curl -X GET https://api.provider.com/v1/logs` | JSON | Requires API credentials |
| Admin Console | Reports > Export > Date Range | CSV/PDF | Limited to UI capabilities |

## Known Limitations

- Geo-IP location data may be inaccurate for certain IP ranges
- Authentication logs may not show application-specific actions
- VPN and proxy services mask true source location
- Some cloud providers have limitations on log retention periods
- Detailed MFA information may vary between providers
- Time synchronization issues can affect event correlation

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing logs | Retention period exceeded | Check archive storage or adjust time window |
| Incomplete user information | Identity directory sync issues | Cross-reference with HR systems |
| Inconsistent timestamps | Time zone differences | Normalize all timestamps to UTC |
| Limited geo-location data | IP allocation changes | Use external IP intelligence services |

## Additional Resources

- [Identity Provider API Documentation](https://developer.provider.com/docs)
- [SIEM Query Language Guide](https://siem.example.com/docs)
- [Authentication Log Field Dictionary](https://security.example.com/field-guide)

## Related SOPs

- [User Account Containment SOP](user-account-containment-sop.md)
- [Evidence Collection SOP](../sops/sop-template.md)
- [Data Export Compliance Requirements](../sops/sop-template.md)