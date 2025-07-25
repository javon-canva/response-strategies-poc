# GitHub Audit Log Investigation Procedure

## Overview

**Data Source:** GitHub Audit Logs  
**Environment:** GitHub Enterprise Cloud / GitHub.com  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Data Source Description

GitHub Audit Logs provide a comprehensive record of all actions taken within GitHub organizations, repositories, and at the enterprise level. This data source is critical for investigating suspicious activity, unauthorized access, and potential security incidents related to source code repositories, CI/CD pipelines, and GitHub-based workflows. GitHub audit logs capture user authentication events, repository operations, permission changes, and administrative actions.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| GitHub UI | Organization Settings → Audit Log | GitHub organization admin | Limited to 90 days of data |
| GitHub UI | Enterprise Settings → Audit Log | GitHub enterprise admin | Enterprise-level events |
| GitHub API | `/orgs/{org}/audit-log` | Personal access token with `admin:org` scope | Programmatic access |
| Elastic | [GitHub Audit Logs Index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/ImxGE) | Okta SSO | Primary analysis method with enriched data |

### Required Permissions

- GitHub organization admin role for UI access
- Enterprise admin for enterprise-level events
- Personal access token with `admin:org` scope for API access
- Appropriate Elastic access permissions

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| GitHub UI | Organization Settings → Audit Log | 90 days | Limited search capabilities |
| GitHub API | `/orgs/{org}/audit-log` | 90 days | JSON format export |
| Elastic | `github.github_audit` index | Configurable (typically 90-180 days) | Enhanced search capabilities |
| Log Archives | AWS S3 storage | 1 year | For long-term retention |

## Common Investigation Scenarios

### Scenario 1: Suspicious Authentication Events

#### Query Examples

```
# Elastic query for authentication events from unusual locations
event.dataset:github.github_audit AND 
event.action:("user.login" OR "oauth_access.create") AND 
NOT source.geo.country_name:(United States OR Australia OR United Kingdom) AND 
@timestamp:[now-24h TO now]

# Elastic query for failed authentication attempts
event.dataset:github.github_audit AND 
event.action:("user.failed_login") AND 
user.name:<username> AND 
@timestamp:[now-24h TO now]
```

**Fields of interest:**
- `event.action`: The type of authentication event
- `user.name`: The GitHub username associated with the event
- `source.ip`: Source IP address of the authentication attempt
- `source.geo.country_name`: Country of origin for the request
- `source.as.organization.name`: ASN organization name
- `user_agent.original`: User agent string from the request

#### Interpretation Guidelines

- Check for authentication from unusual locations or ASNs for the specific user
- Multiple failed login attempts followed by a successful login is suspicious
- Review user agent strings for unusual clients or automation tools
- Verify that successful authentications have correlated activity in other logs
- Consider timezone in relation to the user's normal working hours

### Scenario 2: Suspicious Repository Activities

#### Query Examples

```
# Elastic query for sensitive repository actions
event.dataset:github.github_audit AND 
event.action:("repo.create" OR "repo.destroy" OR "repo.access" OR 
             "repo.add_member" OR "repo.config.enable_collaborators_only" OR 
             "repo.config.disable_collaborators_only" OR "protected_branch.*") AND 
@timestamp:[now-24h TO now]

# Elastic query for webhook modifications
event.dataset:github.github_audit AND 
event.action:("hook.*") AND 
@timestamp:[now-24h TO now]
```

**Fields of interest:**
- `event.action`: The specific repository action performed
- `github.repository.name`: The repository affected
- `github.repository.visibility`: Public or private repository status
- `user.name`: User who performed the action
- `github.hook.config.url`: For webhook events, the URL endpoint configured

#### Interpretation Guidelines

- Changes to protected branch settings may indicate an attempt to bypass code review
- New webhooks could represent data exfiltration channels
- Repository visibility changes (private to public) may indicate unauthorized disclosure
- Look for repository destruction followed by recreation to hide malicious commits
- Check for unusual repository permission grants, especially outside normal teams

### Scenario 3: Organization and Team Membership Changes

#### Query Examples

```
# Elastic query for organization membership changes
event.dataset:github.github_audit AND 
event.action:("org.add_member" OR "org.remove_member" OR 
             "org.update_member" OR "org.invite_member") AND 
@timestamp:[now-24h TO now]

# Elastic query for team membership and permission changes
event.dataset:github.github_audit AND 
event.action:("team.add_member" OR "team.remove_member" OR 
             "team.add_repository" OR "team.remove_repository") AND 
@timestamp:[now-24h TO now]
```

**Fields of interest:**
- `event.action`: The specific organization or team action
- `user.name`: User who performed the action
- `github.org.name`: The organization name
- `github.team.name`: For team events, the team name
- `github.target.user.login`: The user affected by the action
- `github.target.team.permission`: Permission level granted

#### Interpretation Guidelines

- Unexpected additions to privileged teams may indicate privilege escalation
- Check for users being added and then quickly removed after performing actions
- Review permission changes to ensure they follow proper authorization processes
- Examine the timing of team changes relative to other suspicious activities
- Verify organization invites are to legitimate corporate email addresses

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| CloudFlare Logs | `source.ip`, `user.name` | Correlate GitHub activity with web access patterns |
| Okta Logs | `user.name`, `source.ip` | Link GitHub authentication to Okta sign-ins |
| VPN Logs | `source.ip` | Verify if GitHub access occurred through corporate VPN |
| AWS CloudTrail | `user.name` (mapped to AWS role) | Connect GitHub actions to corresponding AWS activities |

### Common Correlation Techniques

1. **Authentication Chain Analysis**
   - Description: Trace authentication events across multiple systems
   - Example approach:
     ```
     # First, identify GitHub login
     event.dataset:github.github_audit AND user.name:<username> AND event.action:"user.login"
     
     # Then, search for corresponding Okta authentication
     event.dataset:okta.system AND user.name:<username> AND 
     @timestamp:[<github_login_time-5m> TO <github_login_time>]
     ```
   - Expected outcomes: Verification of legitimate authentication flow or identification of bypass

2. **Action-Impact Correlation**
   - Description: Connect GitHub actions to their impacts in other systems
   - Example approach:
     ```
     # Find GitHub workflow dispatch events
     event.dataset:github.github_audit AND event.action:"workflow.dispatch"
     
     # Look for corresponding AWS API calls from GitHub Actions
     event.dataset:aws.cloudtrail AND aws.cloudtrail.user_identity.session_name:*GitHubActions*
     ```
   - Expected outcomes: Connect code changes or workflow invocations to infrastructure changes

## Data Collection for Investigations

### Evidence Collection Process

1. For GitHub UI-based evidence collection:
   - Navigate to the organization or enterprise audit log
   - Apply appropriate filters (user, repository, action type)
   - Use the export feature to download as CSV or JSON
   - Maintain chain of custody documentation with timestamps

2. For API-based evidence collection:
   ```bash
   # Using GitHub CLI
   gh api --paginate /orgs/{org}/audit-log?phrase=user:{username} > github_audit_evidence.json
   
   # Using curl
   curl -H "Authorization: token $GITHUB_TOKEN" \
     "https://api.github.com/orgs/{org}/audit-log?phrase=user:{username}" > github_audit_evidence.json
   ```

3. For Elastic-based evidence collection:
   - Create and save a specific search query
   - Export the search results in JSON format
   - Document the query parameters and time range
   - Preserve the exported data in a secure location

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| GitHub UI | Audit Log → Export | CSV | Limited to 90 days, 100,000 rows |
| GitHub API | `/orgs/{org}/audit-log` | JSON | Requires API token, supports pagination |
| Elastic | Discover → Export | CSV/JSON | Most flexible query options |
| GitHub Enterprise | Contact GitHub Enterprise support | JSON | For older data beyond 90 days |

## Known Limitations

- GitHub.com audit logs are limited to 90 days retention
- Not all actions are logged at the same detail level
- Some fields may be redacted or truncated for sensitive operations
- Audit logs do not capture content changes, only metadata about actions
- Time zone differences may impact correlation across data sources
- Bot activities may not always be distinguishable from human actions

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing audit events | Limited retention period | Check archived logs if available; narrow investigation timeframe |
| Insufficient detail | Event type limitations | Correlate with other data sources; request additional context from GitHub |
| Permission denied | Insufficient access rights | Request elevated permissions; work with organization admin |
| Rate limiting | Too many API requests | Implement pagination and backoff; use bulk export instead |
| Inconsistent timestamps | Time zone differences | Standardize on UTC; account for time zone offsets in queries |

## Additional Resources

- [GitHub Audit Log Documentation](https://docs.github.com/en/organizations/keeping-your-organization-secure/managing-security-settings-for-your-organization/reviewing-the-audit-log-for-your-organization)
- [GitHub API Audit Log Reference](https://docs.github.com/en/rest/orgs/orgs#get-the-audit-log-for-an-organization)
- [GitHub Enterprise Audit Log Streaming](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/streaming-the-audit-log-for-your-enterprise)
- [GitHub Security Best Practices](https://docs.github.com/en/enterprise-cloud@latest/admin/user-management/managing-users-in-your-enterprise/best-practices-for-user-security)

## Related SOPs

- [GitHub User Containment](../../sops/github-user-containment-sop.md)
- [Repository Access Review](../../sops/repository-access-review-sop.md)
- [GitHub Organization Security Configuration](../../sops/github-organization-security-sop.md)