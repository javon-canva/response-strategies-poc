# Suspicious Authentication Activity Incident Playbook

## Overview

**Incident Type:** Suspicious Authentication Activity  
**Severity Levels:** High/Medium/Low  
**Response Team:** Detection and Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-23  
**Maintainer:** Detection and Response Team

## Detection Sources

- SIEM Authentication Log Monitoring
- Cloud Identity Provider Alerts (Okta, Azure AD)
- User-Reported Suspicious Activity
- [Authentication Anomaly Detection Rule](https://example.com/detection-rules/auth-anomalies)

## Response Strategy

### Assumptions

- Authentication logs are available and properly collected
- Access to identity provider admin consoles is available
- User directory information is current and accessible
- Potential false positives may include VPN usage from unusual locations, device changes, or browser updates

### Considerations

- User productivity impact during containment actions
- Potential privacy concerns when investigating user activity
- Legal requirements for handling personal data in different regions
- Communication needs to be handled carefully to avoid tipping off potential attackers

## Triage

### Initial Assessment

1. Verify alert details including user identity, source location, device information, and authentication status (success/failure)
2. Follow the [Initial Evidence Collection SOP](../sops/sop-template.md) for proper evidence handling
3. Cross-reference with recent similar alerts to identify patterns
4. Check if the user is on the high-value target list

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | Successful authentication for privileged account (admin, service account) from suspicious location/device |
| High     | Successful authentication for standard user from known malicious source OR multiple failed authentications followed by success |
| Medium   | Successful authentication from unusual but not definitively suspicious location |
| Low      | Failed authentication attempts without successful login |

### Validation Criteria

- Confirm user location vs authentication source location using [Geo-IP Analysis](../data-source-procedures/data-source-template.md)
- Verify if device is registered and known for this user
- Check authentication timing against user's normal work patterns
- Determine if the authentication follows expected workflow patterns

## Investigation

### Investigation Pattern

Follow the [User Authentication Analysis Pattern](user-authentication-analysis-pattern.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Cloud Identity Provider Logs](cloud-provider-logs-data-source.md)
- [VPN Access Logs](../data-source-procedures/data-source-template.md)
- [Endpoint Security Logs](../data-source-procedures/data-source-template.md)

### Investigation Steps

1. **Establish Authentication Timeline**
   - Query identity provider logs for all authentication activity for the user in the past 24 hours
   - Map locations, devices, and access patterns
   - Identify any deviations from typical user behavior

2. **Post-Authentication Activity Analysis**
   - Review actions taken after suspicious authentication
   - Check for data access, privilege escalation, or configuration changes
   - Use [Post-Authentication Analysis SOP](../sops/sop-template.md) for detailed steps

3. **Account Context Analysis**
   - Review user permissions and access levels
   - Determine potential impact of compromised credentials
   - Assess lateral movement opportunities from this account

## Containment

### Containment Strategy

If suspicious authentication is confirmed, contain the account to prevent further damage while minimizing business disruption.

### Containment Steps

1. **Account Lockdown**
   - Follow the [User Account Containment SOP](user-account-containment-sop.md) to suspend the affected account
   - Force sign-out of all current sessions
   - Enable enhanced monitoring for related accounts

2. **Access Restriction**
   - Block the source IP address at network boundaries
   - Place additional verification requirements on related accounts
   - Document all containment actions using the [Containment Documentation Template](../artifact-guidelines/artifact-guideline-template.md)

## Eradication

### Eradication Strategy

Remove unauthorized access and eliminate persistence mechanisms while preparing for account recovery.

### Eradication Steps

1. **Credential Reset**
   - Reset credentials following the [Secure Credential Reset Process](../sops/sop-template.md)
   - Revoke and reissue all tokens and certificates
   - Verify no backdoor accounts or permissions were created

2. **Endpoint Verification**
   - Scan user endpoints for potential compromise
   - Check for unauthorized application access tokens
   - Verify MFA device integrity

## Recovery

### Recovery Strategy

Restore secure access for legitimate users with enhanced monitoring and verification steps.

### Recovery Steps

1. **Account Recovery**
   - Follow the [Secure Account Recovery SOP](../sops/sop-template.md)
   - Coordinate with the user to establish new security measures
   - Implement additional monitoring for the recovered account

2. **Verification and Monitoring**
   - Verify normal authentication patterns resume
   - Implement enhanced logging for the affected user
   - Document recovery steps and user communication

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../sops/sop-template.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time from suspicious login to containment
- Impact assessment: data accessed, actions taken by unauthorized user
- Effectiveness of detection controls
- User experience during recovery process

### Detection Improvement

- Review and tune authentication anomaly detection rules
- Consider implementing additional controls for high-risk users
- Evaluate MFA implementation and effectiveness

## Additional Resources

- [NIST SP 800-63B Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [MITRE ATT&CK Valid Accounts](https://attack.mitre.org/techniques/T1078/)
- [Identity Provider Security Recommendations](https://example.com/identity-provider-security)

## Related Artifacts

- [Authentication Analysis Query Library](../artifact-guidelines/artifact-guideline-template.md)
- [User Account Compromise Communication Templates](../artifact-guidelines/artifact-guideline-template.md)