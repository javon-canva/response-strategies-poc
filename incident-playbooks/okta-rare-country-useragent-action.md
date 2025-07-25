# Okta Rare Country, User-Agent, and Event Action Incident Playbook

## Overview

**Incident Type:** Suspicious Authentication Activity  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic ML Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/development/app/r/s/LP3Zb)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/okta_rare_country_ua_and%20_action_by_canvanaut.md)

## Response Strategy

### Assumptions

- Access to Elastic ML anomaly detection data
- Understanding of Okta authentication processes
- Access to other identity logging sources for validation

### Considerations

- Canvanauts may work from overseas locations different from their usual location
- Many Canvanauts have applications like Slack, GitHub, and email configured on mobile devices
- False-positive alerts may occur when a Canvanaut legitimately accesses Okta from both a rare country and user-agent, such as using a different browser while working abroad or on leave

## Triage

### Initial Assessment

1. Access the underlying ML anomaly detection index to get complete details on the rare country, user-agent, and action that triggered the alert
2. Follow the steps below to query the ML index directly:
   - Navigate to Index Management in Elastic
   - Enable the option to "Include hidden indices"
   - Search for `.ml-anomalies-custom-okta-rare-ml-jobs` index
   - Click "Discover index" to open it in Discover view

3. Use the following query to find relevant anomaly records:
   ```
   job_id: okta_rare-user-country-and-ua-against-self
   AND result_type: record
   AND record_score >= 50
   AND (by_field_name: (user_agent.original OR source.geo.country_name OR event.action))
   AND user.name: <user-who-was-in-the-alert>
   AND "@timestamp" >= "now-1h/h"
   ```

4. For historical events, use a specific time range:
   ```
   AND ("@timestamp" >= "2024-05-09T21:15:00.000+1000" AND "@timestamp" <= "2024-05-09T21:30:00.000+1000")
   ```

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Access from high-risk country; Unusual action performed; Multiple authentication attempts; Failed authentication followed by success; Multiple systems accessed from same unusual location | 
| Medium   | Access from uncommon but not high-risk country; Unusual user-agent but typical actions; Limited scope of access |
| Low      | Confirmed travel by employee; Activity consistent with legitimate work; User confirmation of activity |

### Validation Criteria

- Check whether access to other systems (AWS, GitHub) occurs from the same source IP address/ASN/geolocation
- Verify if the same user-agent is being used across systems
- Check CanvaWorld to see if the Canvanaut is on leave, which may indicate they are accessing while out-of-office
- Contact the Canvanaut directly to confirm their location and activity
- Escalate to an incident if:
  - You see access to multiple systems from the same unusual IP address
  - You are unable to contact the Canvanaut
  - The activity is suspicious in nature with potential security impact

## Investigation

### Investigation Pattern

Follow the [User Authentication Analysis Pattern](../../investigation-patterns/user-authentication-analysis.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Okta Logs Investigation](../../data-source-procedures/okta-logs-investigation.md)
- [IP Geolocation Analysis](../../data-source-procedures/ip-geolocation-analysis.md)

### Investigation Steps

1. **Analyze ML Anomaly Details**
   - Add the fields `@timestamp`, `by_field_name`, and `user.name` as columns in the results
   - For each result, examine the `by_field_value` to identify exactly what triggered the alert
   - Note the `record_score` which indicates the rarity of the event
   - Document the specific country, user-agent, and action combination that triggered the alert

2. **Cross-Reference with Other Authentication Systems**
   - Check for access to other systems (AWS, GitHub, etc.) from the same IP address
   - Look for similar user-agent strings across systems
   - Identify any pattern of access across multiple systems
   - Note any authentication failures preceding successful logins

3. **User Context Research**
   - Check if the user has reported travel plans in CanvaWorld or Confluence
   - Review recent authentication patterns for the user
   - Contact the user directly to verify their location and activity
   - Confirm if they are using a new device or browser

## Containment

### Containment Strategy

If suspicious activity is confirmed, immediately suspend the user's access to prevent further unauthorized actions.

### Containment Steps

1. **Suspend Okta Access**
   - Follow the [User Account Containment SOP](../../sops/user-account-containment-sop.md) to suspend the user's Okta access
   - Use the Cydarm containment workflow for proper tracking
   - Expected outcome: User's Okta access is suspended, preventing further access to company resources

2. **Block Suspicious IP**
   - If applicable, block the suspicious IP address at the network level
   - Document all blocked IPs for reference
   - Expected outcome: Suspicious IP can no longer access company resources

## Eradication

### Eradication Strategy

Determine the root cause of the unauthorized access and eliminate the attack vector.

### Eradication Steps

1. **Determine Access Method**
   - Investigate how the unauthorized party gained access
   - Check for potential malware infection on the user's endpoint
   - Look for evidence of phishing or credential theft
   - Expected outcome: Root cause of the security breach is identified

2. **Revoke All Sessions**
   - Use the [Session Token Remediation SOP](../../sops/session-token-remediation.md) to revoke all active sessions
   - Reset any compromised credentials
   - Expected outcome: All potential unauthorized access paths are eliminated

## Recovery

### Recovery Strategy

After confirming the threat has been eradicated, restore normal access for the legitimate user.

### Recovery Steps

1. **Restore Okta Access**
   - Once remediation is complete, restore the user's Okta access
   - Follow the [user un-containment procedure](../../sops/user-account-containment-sop.md#un-contain-user-procedure)
   - Expected outcome: Legitimate user regains access to necessary resources

2. **Monitor for Additional Activity**
   - Implement enhanced monitoring for the affected user
   - Watch for any additional anomalous behavior
   - Expected outcome: Early detection of any recurrence of suspicious activity

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to user contact
- Number of systems accessed
- Duration of unauthorized access (if applicable)

### Detection Improvement

- Review anomaly detection thresholds for potential adjustments
- Consider adding additional context to alerts for faster triage
- Evaluate the effectiveness of the ML model and refine as needed

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [Okta SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2428896150/Okta+SOPs)
- [Session Token Remediation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2390099896/Remediating+Session+Tokens)

## Related Artifacts

- [Okta Security Configuration Guidelines](../../artifact-guidelines/okta-security-configuration.md)
- [Authentication Monitoring Dashboard](../../artifact-guidelines/authentication-monitoring-dashboard.md)