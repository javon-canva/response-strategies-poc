# GitHub Authentication Analysis Investigation Pattern

## Overview

**Pattern Type:** Authentication Analysis  
**Applicable Incident Types:** Suspicious GitHub Logins, Bot Activity Anomalies, Account Compromise, Unauthorized Repository Access  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Pattern Description

This investigation pattern provides a structured approach for analyzing suspicious authentication activities within GitHub. It applies to anomalous logins, unusual bot activities, potential account compromises, and unauthorized access to repositories. The pattern focuses on correlating GitHub audit logs with other data sources to determine the legitimacy of authentication events and assess potential security implications.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [GitHub Audit Logs](../../data-source-procedures/github-audit-log-analysis.md)
   - [CloudFlare Logs](../../data-source-procedures/cloudflare-logs-analysis.md)
   - [IP Geolocation Services](../../data-source-procedures/ip-geolocation-analysis.md)
   - [Okta Authentication Logs](../../data-source-procedures/okta-logs-investigation.md)

2. **Verify Data Availability**
   - Ensure GitHub audit logs cover the time period of interest
   - Check that all required log types are available (authentication, repository, organization)
   - Verify CloudFlare logs are accessible for the relevant time period
   - Confirm access to GitHub Enterprise or organization admin settings
   - Validate that bot registry and ownership information is current

3. **Prepare Investigation Environment**
   - Set up access to GitHub audit log interface
   - Configure Elastic search queries for GitHub events
   - Create timeline tracking worksheet
   - Prepare secure communications channel for contacting users or teams

## Investigation Methodology

### Phase 1: Authentication Context Analysis

**Purpose:** Establish the baseline context of the authentication and determine initial anomaly factors

**Steps:**
1. Identify the authentication source details:
   ```
   # Elastic query for GitHub authentication events
   event.dataset:github.github_audit AND user.name:"<username>" AND 
   @timestamp:[<start_time> TO <end_time>]
   ```

2. Analyze the authentication attributes:
   - IP address and geolocation
   - ASN and organization name
   - User agent string
   - Authentication method (token, password, OAuth, etc.)
   - Time of authentication relative to user's normal patterns

3. For bot accounts, check against the bot registry:
   - Verify if the bot is documented in the [GitHub bot registry](https://docs.google.com/spreadsheets/d/1jE_Uz5AQ-4tnZQeDeIwSEuLPkYgFG_rga-1Mr59TZ7Q/)
   - Identify the owning team or individual
   - Note the expected usage patterns and permissions

4. Compare with historical authentication patterns:
   ```
   # Elastic query for historical authentication patterns
   event.dataset:github.github_audit AND user.name:"<username>" AND 
   @timestamp:[now-90d TO now]
   ```

**Expected Outcomes:**
- Clear understanding of the authentication event and its anomalous factors
- Identification of historical patterns for comparison
- Preliminary assessment of likelihood of legitimate versus suspicious authentication
- List of key factors requiring further investigation

**Pivot Points:**
- If authentication source is from a highly unusual location, pivot to potential compromise investigation
- If user agent indicates unusual software, focus on potential malware or automation
- If authentication method differs from typical patterns, examine potential credential theft
- If bot activity deviates from expected patterns, prioritize communication with owning team

### Phase 2: Action Analysis

**Purpose:** Analyze the actions taken during the authenticated session to assess intent and impact

**Steps:**
1. Create a timeline of all actions performed:
   ```
   # Elastic query for actions by the user
   event.dataset:github.github_audit AND user.name:"<username>" AND 
   @timestamp:[<auth_time> TO <end_time>]
   ```

2. Categorize actions by sensitivity and potential impact:
   - High-risk actions: branch protection changes, webhook modifications, access key creation
   - Medium-risk actions: repository creation, membership changes, workflow modifications
   - Low-risk actions: code browsing, pull request reviews, issue comments

3. For repository actions, analyze the target repositories:
   ```
   # Elastic query for repository-specific actions
   event.dataset:github.github_audit AND user.name:"<username>" AND 
   github.repository.name:"<repo_name>" AND 
   @timestamp:[<auth_time> TO <end_time>]
   ```

4. For bot accounts, examine actions against expected behavior:
   - Review typical action patterns for the specific bot
   - Compare repository access with authorized repositories
   - Analyze action frequency and volume against normal patterns

**Expected Outcomes:**
- Complete timeline of actions during the suspicious session
- Assessment of potential impact and risk level
- Identification of any unauthorized access or malicious actions
- Determination if activity matches expected patterns for the account type

**Pivot Points:**
- If high-risk actions are observed, pivot to impact assessment and containment
- If repository access extends beyond normal patterns, expand scope to repository security review
- If minimal or expected actions are observed, focus on false positive analysis
- If automation patterns are detected, examine CI/CD systems and GitHub Actions

### Phase 3: Authentication Source Validation

**Purpose:** Determine the legitimacy of the authentication source and validate against expected patterns

**Steps:**
1. For IP-based analysis, correlate with CloudFlare logs:
   ```
   # CloudFlare logs query for the source IP
   source.ip:"<ip_address>" AND 
   @timestamp:[<auth_time-1h> TO <auth_time+1h>]
   ```

2. For user-based verification, correlate with Okta authentication:
   ```
   # Okta authentication logs query
   event.dataset:okta.system AND user.name:"<username>" AND 
   @timestamp:[<auth_time-1h> TO <auth_time+1h>]
   ```

3. For GitHub Codespaces activity:
   - Look for CloudFlare access to `github.dev` domains
   - Verify `repo.download_zip` actions from Microsoft infrastructure
   - Check token scope for Codespaces permissions

4. For bot authentication analysis:
   - Check if the authentication source matches expected infrastructure
   - Verify with bot owners about any infrastructure or configuration changes
   - Examine deployment pipelines and CI/CD logs for relevant jobs

**Expected Outcomes:**
- Verification of authentication source legitimacy
- Correlation between GitHub authentication and other system access
- Confirmation of whether activity matches expected usage patterns
- Determination of whether infrastructure changes explain anomalies

**Pivot Points:**
- If authentication cannot be correlated with legitimate activity, escalate as potential compromise
- If infrastructure changes explain the anomaly, document for detection tuning
- If new usage patterns are legitimate, update documentation and baselines
- If source is confirmed malicious, initiate containment procedures

## Data Correlation Techniques

### Technique 1: Cross-Platform Authentication Chain

**Purpose:** Trace the complete authentication flow across multiple platforms to validate legitimacy

**Implementation:**
1. Start with the GitHub authentication event
2. Look for preceding authentication events in Okta or other IdP
3. Trace network access via CloudFlare or VPN logs
4. Identify device authentication via endpoint management tools

**Example:**
```
# Step 1: Find the GitHub authentication
event.dataset:github.github_audit AND user.name:"<username>" AND event.action:"authentication" AND 
@timestamp:<auth_time>

# Step 2: Look for preceding Okta authentication
event.dataset:okta.system AND user.name:"<username>" AND 
@timestamp:[<auth_time-10m> TO <auth_time>]

# Step 3: Check for CloudFlare access
source.ip:<ip_address> AND http.request.method:GET AND url.domain:github.com AND 
@timestamp:[<auth_time-5m> TO <auth_time>]
```

### Technique 2: User Agent Analysis

**Purpose:** Analyze user agent patterns to distinguish between legitimate software updates and suspicious clients

**Implementation:**
1. Extract the user agent string from the GitHub authentication event
2. Parse the components (browser/git client, version, OS)
3. Compare against historical user agents for the same user
4. Check for temporal patterns in user agent changes across the organization

**Example:**
```
# Get historical user agents for the user
event.dataset:github.github_audit AND user.name:"<username>" AND 
@timestamp:[now-90d TO now] | stats count() by user_agent.original

# Check organization-wide patterns for similar user agent
event.dataset:github.github_audit AND user_agent.original:"<user_agent>" AND 
@timestamp:[now-30d TO now] | stats count() by user.name, @timestamp
```

## Investigation Artifacts

### Required Artifacts

- **Authentication Timeline**: Chronological record of authentication events and actions
- **User Agent Analysis**: Breakdown of user agent components and comparison with baselines
- **IP Geolocation Report**: Analysis of source IP addresses with geolocation context
- **Action Impact Assessment**: Evaluation of actions performed and their potential security impact

### Optional Artifacts

- **Bot Activity Baseline**: Documentation of normal bot activity patterns for comparison
- **Visual Authentication Map**: Geographic visualization of authentication sources
- **User Interview Notes**: Documentation from discussions with account owners
- **Infrastructure Change Log**: Record of relevant infrastructure changes that might affect authentication patterns

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| GitHub Codespaces Usage | Authentication from Microsoft ASNs; repo.download_zip actions; Access to github.dev subdomains | Check CloudFlare logs for github.dev access with DGA-style subdomains; Verify token scope includes codespaces permissions; Confirm with user if they were using Codespaces |
| Git Client Update | Minor version change in user agent; Same geographic location and ASN as normal; Normal action patterns | Compare version change with Git release schedule; Verify other users showing similar version updates; Confirm normal action patterns continue |
| New CI/CD Pipeline | Bot authentication from new infrastructure; Normal action patterns for bot; Increased frequency of standard actions | Check deployment logs for new pipeline configurations; Contact bot owners to verify infrastructure changes; Compare actions with expected bot behaviors |

## Investigation Completion Criteria

### Required Findings

- Determination of authentication legitimacy (legitimate, suspicious, or confirmed malicious)
- Complete timeline of relevant actions during the session
- Assessment of potential security impact
- Identification of root cause for anomalous authentication indicators
- Recommendations for response actions if suspicious activity is confirmed

### Required Documentation

- Summary of authentication analysis and findings
- Evidence supporting the determination of legitimacy or compromise
- List of all impacted resources if suspicious activity is confirmed
- Recommended detection improvements to reduce false positives
- Updates to bot registry or documentation if legitimate new patterns are identified

## Related SOPs

- [GitHub User Containment SOP](../../sops/github-user-containment-sop.md)
- [Bot Credential Rotation SOP](../../sops/bot-credential-rotation-sop.md)
- [Repository Security Review SOP](../../sops/repository-security-review-sop.md)

## Additional Resources

- [GitHub Security Documentation](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/audit-log-events-for-your-enterprise)
- [GitHub Authentication Mechanisms](https://docs.github.com/en/authentication)
- [GitHub Bot Best Practices](https://docs.github.com/en/developers/apps/getting-started-with-apps/about-apps)
- [User Agent Analysis Guide](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)