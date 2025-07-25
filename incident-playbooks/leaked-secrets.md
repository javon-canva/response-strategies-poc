# Leaked Secrets Incident Playbook

## Overview

**Incident Type:** Secret Leakage  
**Severity Levels:** Low/Medium/High/Critical  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Spycloud](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2878345483/Spycloud)
- [Bugcrowd](https://bugcrowd.com/canva/programs)
- Internal Reports (e.g., Canvanaut reports from Slack or GitHub)
- External Service Notifications (e.g., GitHub Secret Scanning)

## Response Strategy

### Assumptions

- Access to secret validation systems and relevant logs
- Ability to contact affected users and system owners
- Authority to suspend accounts and revoke sessions
- Access to rotate secrets across multiple platforms
- Understanding of the organization's system architecture

### Considerations

- Different secret types require different validation and rotation procedures
- Source of exposure may indicate wider compromise requiring additional investigation
- Public exposure vs. internal exposure has different risk implications
- Historical credentials may appear in breach databases but may no longer be valid
- Rotation of some secrets may impact production services

## Triage

### Initial Assessment

1. Identify the type of secret exposed:
   - User credentials (email/password)
   - API keys or tokens
   - Service account credentials
   - SSH keys
   - Database credentials
   - Encryption keys

2. Determine if the secret has been previously identified:
   - Check the Leaked Credentials Tracker
   - Search previous cases involving the same secret
   - Verify if mitigation actions have already been taken
   - If secret has already been rotated after the date of exposure, document and close

3. Assess the exposure context:
   - Document where the secret was exposed (public internet, internal systems)
   - Determine how long the secret was exposed
   - Identify who had access to the exposed secret
   - Verify if the exposure was intentional or accidental

4. For Canvanaut credentials, check Spycloud for additional context:
   - Identify the source of the breach/exposure
   - Check for other credentials from the same data breach
   - Look for device information associated with credential theft
   - Verify indicators of malware involvement

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | Secret provides administrator access to production systems; Secret enables access to sensitive customer data; Evidence of active exploitation of the leaked secret; Leaked infrastructure keys that could enable widespread access |
| High     | Secret provides access to sensitive internal systems; Secret has been publicly exposed for extended period; Credentials for privileged accounts; API keys with elevated permissions |
| Medium   | Secret provides limited access to non-critical systems; Internal exposure only; No evidence of exploitation; Minimal potential impact if used |
| Low      | Secret already rotated or expired; Secret provides read-only access to non-sensitive data; Exposure limited to authorized personnel |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Secret is valid and no previous mitigation has been applied
- Secret provides access to sensitive or critical systems
- There is evidence of exploitation or attempted use
- Multiple secrets are exposed from the same source, indicating a pattern
- Secret is associated with a confirmed device compromise

## Investigation

### Investigation Pattern

Follow the [Credential Exposure Investigation Pattern](../../investigation-patterns/credential-exposure-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Spycloud Investigation](../../data-source-procedures/spycloud-investigation.md)
- [Authentication Log Analysis](../../data-source-procedures/authentication-log-analysis.md)
- [Endpoint Security Investigation](../../data-source-procedures/endpoint-security-investigation.md)

### Investigation Steps

1. **Validate the Secret**
   - Determine if the secret is still valid and provides access
   - Follow appropriate validation procedures based on secret type:
     - For credentials, check if they work in Okta (note your IP and user agent)
     - For API keys, use minimal validation that doesn't modify data
     - For other secrets, follow AppSec's Validating Secrets Playbook
   - Document all validation steps and results
   - Expected outcome: Confirmation of secret validity and access it provides

2. **Determine Source of Exposure**
   - Review the report details for initial exposure information
   - For credentials in breach databases, analyze breach context
   - For internal reports, interview the reporting user
   - Check for signs of device compromise:
     - Review SentinelOne logs for suspicious activity
     - Check for known malware signatures on affected devices
     - Analyze browser extensions and installed applications
   - Expected outcome: Identification of how and when the secret was exposed

3. **Assess Usage and Access**
   - Review authentication logs for the affected account/secret
   - Look for access from unusual locations, devices, or user agents
   - Analyze API usage patterns for exposed API keys
   - Check for unusual activity patterns or volume
   - Determine normal vs. abnormal usage patterns
   - Expected outcome: Understanding of any unauthorized access or misuse

4. **Determine Impact Scope**
   - Identify all systems the secret provides access to
   - Determine what data or functions could be accessed
   - Check for signs of lateral movement if credentials were compromised
   - Assess if the secret was used to create persistent access
   - Determine if other accounts or secrets may be compromised
   - Expected outcome: Complete understanding of potential and actual impact

## Containment

### Containment Strategy

Quickly revoke access provided by the exposed secret while preserving evidence of any unauthorized use.

### Containment Steps

1. **Suspend User Access (if applicable)**
   - For compromised Canvanaut credentials:
     - Suspend the user's Okta access
     - Clear all active sessions using Bootkicker
     - Document the containment actions and timestamps
     - Notify the user's coach about the suspension
   - For service accounts:
     - Disable the account in relevant systems
     - Document all affected applications
   - Expected outcome: Unauthorized access via credentials is prevented

2. **Isolate Affected Devices (if applicable)**
   - If exposure is linked to device compromise:
     - For company devices, disconnect from network using SentinelOne
     - For personal devices, advise immediate disconnection and malware scanning
     - Document device information and containment status
   - Expected outcome: Potentially compromised devices are isolated

3. **Revoke and Rotate Secrets**
   - Follow the appropriate secret rotation procedure based on type:
     - [GitHub Personal Access Token Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3278969153)
     - [1Password Credential Rotation](https://canvadev.atlassian.net/wiki/spaces/CTH/pages/737018007/1Password+-+How+to#How-to-edit-1Password-items?)
     - [GCP API Key Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3290039144)
     - [Atlassian Token Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3279950796)
     - [AWS Access Key Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3279952095)
     - [Slack Configuration Token Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3278773220)
     - [Slack Webhook Rotation](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3279885538)
   - For unlisted services, contact system owner via OneTrust Systems Catalog
   - Document all rotation actions with timestamps
   - Expected outcome: All exposed secrets are invalidated and replaced

## Eradication

### Eradication Strategy

Remove all sources of secret exposure and eliminate any persistent threats related to the exposure.

### Eradication Steps

1. **Remove Exposed Secrets from Sources**
   - Delete secrets from code repositories:
     - For Canva monorepos, follow AppSec's guidance for safe removal
     - For personal repositories, make repository private or remove secret commits
     - Use force push if necessary to remove secret history
   - Clear secrets from communication channels:
     - Delete messages containing secrets from Slack
     - Remove attachments with embedded secrets
   - Document all removal actions taken
   - Expected outcome: All instances of exposed secrets are removed

2. **Remove Malware (if applicable)**
   - For company devices:
     - Follow the Compromised Canvanaut Devices SOP
     - Verify malware removal with endpoint security team
   - For personal devices:
     - Guide user through malware removal with reputable tools
     - Verify device is clean before restoring access
   - Document all remediation steps taken
   - Expected outcome: Any malware is removed from affected devices

3. **Clear Saved Credentials**
   - Guide users to remove secrets from:
     - Browser password managers
     - System keychains
     - Personal password managers
     - Local configuration files
   - Follow the Compromised Chrome Profile SOP if needed
   - Verify removal across all user devices
   - Expected outcome: No saved instances of compromised credentials remain

## Recovery

### Recovery Strategy

Restore secure operations with improved controls to prevent similar incidents in the future.

### Recovery Steps

1. **Restore User Access**
   - For contained Canvanaut accounts:
     - Reset password via secure channel (e.g., Zoom)
     - Run un-contain procedure in Cydarm to restore Okta access
     - Verify user can access required systems
   - For service accounts:
     - Configure with new secrets
     - Verify application functionality
   - Document all restoration actions
   - Expected outcome: Normal operations are restored securely

2. **Create and Deploy New Secrets**
   - Generate new secrets following security best practices
   - Update application configurations with new secrets
   - Store new credentials in approved password manager
   - Verify applications function with new credentials
   - Update credential tracking systems
   - Expected outcome: New secure secrets are deployed and functional

3. **Implement Additional Controls**
   - Update secret handling guidelines as needed
   - Review secret scanning and detection capabilities
   - Implement additional secret rotation schedules
   - Improve user awareness of secure secret handling
   - Document recommendations for preventing recurrence
   - Expected outcome: Improved security controls for secret management

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time from exposure to detection
- Time from detection to secret rotation
- Number of systems affected
- Evidence of unauthorized access (if any)
- Source of secret exposure
- Effectiveness of detection mechanisms

### Detection Improvement

- Review and update secret scanning rules
- Enhance monitoring for suspicious authentication patterns
- Improve integration with breach notification services
- Consider additional endpoint monitoring capabilities
- Update breach response playbooks based on lessons learned

## Additional Resources

- [Spycloud Guide for Incident Responders](../../sops/spycloud-guide-sop.md)
- [Validating Secrets Playbook](https://canvadev.atlassian.net/wiki/spaces/APPSEC/pages/2557378761/Secrets+Identified+in+Version+Control)
- [Compromised Canvanaut Devices SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/1563132774/Compromised+Canvanaut+Devices)
- [Compromised Chrome Profile SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3225257287/Compromised+Chrome+Profiles)
- [OneTrust Systems Catalog](https://hub.canva.world/space/CANVAKB/2796422264)

## Related Artifacts

- [Secret Management Guidelines](../../artifact-guidelines/secret-management-guidelines.md)
- [Credential Security Standards](../../artifact-guidelines/credential-security-standards.md)
- [Leaked Credentials Tracker](https://docs.google.com/spreadsheets/d/1iazwv-c_vwXWTpCbR9lJUN4hMr3CPYjKu4yDiBWNDlA/edit?gid=0#gid=0)