# User Authentication Analysis Investigation Pattern

## Overview

**Pattern Type:** User Activity Analysis  
**Applicable Incident Types:** Account Compromise, Credential Theft, Insider Threat, Unauthorized Access  
**Version:** 1.0  
**Last Updated:** 2025-07-23  
**Maintainer:** Detection and Response Team

## Pattern Description

This investigation pattern provides a structured approach for analyzing authentication events and related user activity to detect and confirm potential account compromises or unauthorized access. It focuses on identifying anomalies in authentication patterns, post-authentication behavior, and establishing a comprehensive timeline of access events.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [Cloud Identity Provider Logs](cloud-provider-logs-data-source.md)
   - [VPN Access Logs](../data-source-procedures/data-source-template.md)
   - [Endpoint Security Logs](../data-source-procedures/data-source-template.md)
   - [Application Access Logs](../data-source-procedures/data-source-template.md)

2. **Verify Data Availability**
   - Ensure logs cover at least 30 days prior to suspected incident
   - Verify identity provider advanced logging is enabled
   - Confirm geo-location data is available for authentication events
   - Check that device identification data is being properly collected

3. **Prepare Investigation Environment**
   - Set up dedicated investigation workspace in SIEM
   - Prepare timeline visualization tool
   - Enable investigation case in incident management system
   - Prepare user context information (role, department, access level, manager)

## Investigation Methodology

### Phase 1: Baseline Establishment

**Purpose:** Establish normal authentication patterns for the user to identify deviations.

**Steps:**
1. Query identity provider logs for 30 days of authentication history:
   ```
   user.name:"[username]" AND event.action:("login" OR "authenticate" OR "sso") 
   | stats count by source.ip, source.geo.country_name, user_agent.original, event.outcome
   ```

2. Map the user's normal authentication patterns:
   ```
   user.name:"[username]" AND event.action:("login" OR "authenticate" OR "sso") AND event.outcome:success
   | timechart span=1h count
   ```

3. Identify commonly used devices, locations, and access times:
   ```
   user.name:"[username]" AND event.action:("login" OR "authenticate" OR "sso") AND event.outcome:success
   | stats count by source.ip, source.geo.country_name, user_agent.original
   | sort count desc
   | limit 10
   ```

**Expected Outcomes:**
- Baseline working hours and authentication frequency
- List of common devices and locations
- Typical authentication failures and retry patterns
- Normal application access sequence

**Pivot Points:**
- Multiple locations that are physically impossible to travel between
- Simultaneous authentications from different locations
- Unusual working hours without business justification
- Unrecognized devices or user agents

### Phase 2: Anomaly Detection

**Purpose:** Identify authentication events that deviate from the established baseline.

**Steps:**
1. Compare suspicious event to baseline patterns:
   ```
   user.name:"[username]" AND event.action:("login" OR "authenticate" OR "sso") AND source.geo.country_name:"[suspicious_country]"
   | sort @timestamp desc
   ```

2. Check for failed authentication attempts prior to successful login:
   ```
   user.name:"[username]" AND event.outcome:failure AND @timestamp:[time_window_before_suspicious_event]
   | sort @timestamp desc
   ```

3. Analyze authentication factors used:
   ```
   user.name:"[username]" AND event.action:"mfa" AND @timestamp:[suspicious_event_time_window]
   | stats count by event.action, event.outcome
   ```

**Expected Outcomes:**
- Confirmation of authentication anomalies
- Pattern of failed attempts (if any)
- MFA bypass attempts or unusual factor usage
- Location and device context for suspicious events

**Pivot Points:**
- Password spraying attempts preceding successful authentication
- MFA push notification approval without expected device context
- Authentication from known malicious IP addresses
- Authentication during unusual hours without business justification

### Phase 3: Post-Authentication Activity Analysis

**Purpose:** Assess actions taken after suspicious authentication to determine impact and intent.

**Steps:**
1. Create timeline of all user activity following suspicious authentication:
   ```
   user.name:"[username]" AND @timestamp:>[suspicious_authentication_time]
   | sort @timestamp asc
   ```

2. Look for privileged actions or access to sensitive data:
   ```
   user.name:"[username]" AND @timestamp:>[suspicious_authentication_time] 
   AND event.action:("admin" OR "privileged" OR "download" OR "export")
   | sort @timestamp asc
   ```

3. Check for configuration changes or permission modifications:
   ```
   user.name:"[username]" AND @timestamp:>[suspicious_authentication_time] 
   AND event.action:("create" OR "update" OR "modify" OR "permission")
   | sort @timestamp asc
   ```

**Expected Outcomes:**
- Comprehensive activity timeline post-authentication
- Assessment of accessed resources and performed actions
- Identification of potential data exfiltration or system modifications
- Determination of attack impact and scope

**Pivot Points:**
- Access to resources not normally used by the user
- Mass downloads or exports of sensitive data
- Configuration changes to authentication systems
- Creation or modification of accounts and permissions

## Data Correlation Techniques

### Technique 1: IP-Based Correlation

**Purpose:** Connect authentication events with other activities from the same source IP.

**Implementation:**
1. Identify all activity from suspicious IP across all users:
   ```
   source.ip:"[suspicious_ip]" AND event.action:("login" OR "authenticate" OR "sso")
   | stats count by user.name, event.outcome
   | sort count desc
   ```

2. Create timeline of all actions from suspicious IP:
   ```
   source.ip:"[suspicious_ip]" AND @timestamp:[relevant_time_window]
   | sort @timestamp asc
   ```

**Example:**
```
source.ip:"203.0.113.100" AND @timestamp:["2025-07-22T00:00:00.000Z" TO "2025-07-23T00:00:00.000Z"]
| sort @timestamp asc
| fields @timestamp, user.name, event.action, event.outcome
```

### Technique 2: Session-Based Correlation

**Purpose:** Track all activity within a specific user session to identify full scope of actions.

**Implementation:**
1. Extract session identifiers from authentication event:
   ```
   user.name:"[username]" AND @timestamp:"[suspicious_authentication_time]" AND event.action:"login"
   | fields session.id, user.name, source.ip
   ```

2. Track all activity using that session ID:
   ```
   session.id:"[extracted_session_id]"
   | sort @timestamp asc
   ```

**Example:**
```
session.id:"SID-12345-ABCDEF" 
| sort @timestamp asc
| fields @timestamp, user.name, event.action, resource.name, event.outcome
```

## Investigation Artifacts

### Required Artifacts

- **Authentication Timeline**: Complete chronological record of authentication events including success/failure status, location, device, and authentication factors used.
- **Post-Authentication Activity Summary**: Documentation of all actions taken after suspicious authentication with risk assessment for each action.
- **IP and Location Analysis**: Analysis of source IP addresses, ASNs, and geo-locations with reputation data.
- **Device Context Analysis**: Evaluation of device fingerprints, user agents, and historical usage patterns.

### Optional Artifacts

- **User Interview Notes**: Record of discussion with the user about their recent activities and locations.
- **Network Traffic Analysis**: Detailed analysis of network traffic from the suspicious source IP if available.
- **Attacker Technique Mapping**: Correlation of observed behaviors to MITRE ATT&CK framework techniques.

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| VPN usage changing geo-location | Sudden geo-location change but from expected device | Check if IP belongs to known VPN provider, verify with user if they were using VPN |
| Travel without notification | Authentication from new location but with normal patterns | Check corporate travel records, verify with user's manager, look for logical travel progression |
| New device or browser update | Unrecognized user agent string but from expected location | Check if user agent follows version progression (e.g., browser update), verify with user |
| Shared service account usage | Multiple authentications from different locations | Check if account is designated as a shared service account with documented multiple users |

## Investigation Completion Criteria

### Required Findings

- Determination of whether authentication was legitimate or unauthorized
- Source of authentication (user, attacker, automation)
- Complete scope of actions taken during suspicious session
- Impact assessment of accessed resources and performed actions

### Required Documentation

- Authentication anomaly analysis with specific deviations from baseline
- Timeline of events including pre-authentication, authentication, and post-authentication activities
- Evidence supporting conclusion (legitimate vs. unauthorized)
- Recommendations for containment if unauthorized access is confirmed

## Related SOPs

- [User Account Containment SOP](user-account-containment-sop.md)
- [Evidence Collection SOP](../sops/sop-template.md)
- [Post-Incident Account Recovery SOP](../sops/sop-template.md)

## Additional Resources

- [MITRE ATT&CK Initial Access Techniques](https://attack.mitre.org/tactics/TA0001/)
- [NIST Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [SANS Hunt Evil Poster](https://www.sans.org/security-resources/posters/hunt-evil/165/download)