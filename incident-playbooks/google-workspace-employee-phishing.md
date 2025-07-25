# Google Workspace Employee Phishing Incident Playbook

## Overview

**Incident Type:** Phishing Attempt  
**Severity Levels:** Low/Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Checkpoint Harmony](https://portal.checkpoint.com/dashboard/email&collaboration/)
- [Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/harmony_google_workspace_employee_phishing.md)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/5c145eb3-250b-4d76-963d-6d9cabf983d3)

## Response Strategy

### Assumptions

- Access to Canva's internal security tools
- Understanding of common phishing tactics and indicators
- Ability to analyze suspicious emails and links

### Considerations

- Not all emails flagged as phishing are actually malicious
- Checkpoint Harmony may not always show clear information regarding Google Workspace alerts
- The user may have already reported the phishing attempt but could have clicked on malicious links or opened attachments
- Quick response is essential to limit potential damage from successful phishing attempts

## Triage

### Initial Assessment

1. Analyze the reported phishing email:
   - In [Checkpoint Harmony](https://portal.checkpoint.com/dashboard/email&collaboration/), locate the email by searching for the sender email address under "Mail Explorer"
   - Check if the sender has been seen previously by setting a 30-day window for mail sent by this address
   - Look for similar subject lines from different senders that might indicate a coordinated campaign
   - Examine any attachments or links in the message

2. Evaluate suspicious elements:
   - For suspicious attachments, submit to [Recorded Future](https://sandbox.recordedfuture.com/) for analysis
   - For suspicious URLs, submit to [urlscan.io](https://www.urlscan.io) or [Recorded Future](https://sandbox.recordedfuture.com/)
   - Check Cloudflare logs to see if the user has visited any suspicious links
   - Look for signs of credential harvesting, malware delivery, or social engineering

3. Contact the user who reported the phishing email:
   - Determine if they clicked any links or opened any attachments
   - Ask about any unusual behavior on their system since receiving the email
   - Verify if they entered credentials or other sensitive information
   - Thank them for reporting the suspicious email

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | User clicked malicious links or opened attachments; Credentials were entered on a phishing site; Multiple users targeted in coordinated campaign; Sophisticated phishing with organization-specific details; Malware payload confirmed | 
| Medium   | Targeted phishing but no user interaction; Previously unseen sender/campaign; Convincing phishing attempt with corporate branding; Unconfirmed whether user interacted with content |
| Low      | Generic phishing detected and reported without user interaction; Known phishing campaign already being blocked; Low-quality attempt with obvious indicators; User recognized and reported without interaction |

### Validation Criteria

Escalate to an incident if any of the following are true:
- User clicked on malicious links or opened malicious attachments
- User entered credentials or sensitive information on a phishing site
- Cloudflare logs confirm visits to malicious URLs
- Multiple users report the same or similar phishing attempts
- Malware has been detected on the user's system following the phishing email

## Investigation

### Investigation Pattern

Follow the [Phishing Email Analysis Investigation Pattern](../../investigation-patterns/phishing-email-analysis.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Email Content Analysis](../../data-source-procedures/email-content-analysis.md)
- [URL and Domain Analysis](../../data-source-procedures/url-domain-analysis.md)
- [User Activity Logging](../../data-source-procedures/user-activity-logging.md)

### Investigation Steps

1. **Analyze Email Headers and Content**
   - Examine email headers for signs of spoofing or unusual routing
   - Review the email body for social engineering techniques
   - Check for brand impersonation or urgent action requests
   - Identify any obfuscation techniques used to hide malicious content
   - Expected outcome: Complete understanding of the phishing technique used

2. **Evaluate Malicious Content**
   - Analyze any attachments in a secure environment
   - Check URL reputation and destination of any links
   - Look for credential harvesting pages or malware delivery mechanisms
   - Determine the objective of the phishing attempt
   - Expected outcome: Identification of the phishing payload and purpose

3. **Assess User Impact**
   - Check Cloudflare logs for evidence of visits to malicious sites
   - Review authentication logs for any suspicious login attempts
   - Look for unusual activity on the user's account following the phishing attempt
   - Determine if credentials or other sensitive information was compromised
   - Expected outcome: Assessment of the actual impact on the user and organization

## Containment

### Containment Strategy

If the phishing attempt was successful, take quick action to limit access and prevent further compromise.

### Containment Steps

1. **Quarantine Malicious Emails**
   - Identify all instances of the phishing email in the organization
   - Quarantine all emails from the malicious sender
   - Block similar emails based on subject, content patterns, or sender domains
   - Expected outcome: Malicious emails are contained and cannot reach additional users

2. **Isolate Affected Systems**
   - If malicious files were opened, isolate the user's host
   - Scan using SentinelOne to detect malicious files
   - Use Deep Visibility search in S1 to search for the file hash of any malicious files
   - Remove any malicious files found on the host
   - Expected outcome: Potentially compromised systems are isolated

3. **Reset User Credentials**
   - If credentials may have been compromised, initiate the user bootkick process
   - Force password resets for affected accounts
   - Review recent authentication activity for signs of unauthorized access
   - Expected outcome: Compromised credentials can no longer be used by attackers

## Eradication

### Eradication Strategy

Remove the threat and prevent similar phishing attempts in the future.

### Eradication Steps

1. **Block Malicious Infrastructure**
   - Block any malicious links within Cloudflare
   - Add malicious domains to email filtering rules
   - Block the sender from sending emails to Canvanauts if appropriate
   - Expected outcome: Malicious infrastructure can no longer be accessed

2. **Remove Malicious Content**
   - Delete any remaining copies of the phishing email
   - Remove malicious files from all systems
   - Verify that no persistence mechanisms were established
   - Expected outcome: All traces of malicious content are removed

3. **Update Security Controls**
   - Add indicators of compromise to security monitoring systems
   - Update email filtering rules to catch similar attempts
   - Share intelligence with security community if appropriate
   - Expected outcome: Enhanced protection against similar threats

## Recovery

### Recovery Strategy

Restore normal operations and strengthen defenses against future phishing attempts.

### Recovery Steps

1. **Restore User Access**
   - Verify that affected systems are clean
   - Restore normal access for affected users
   - Confirm that users can access all required applications
   - Expected outcome: Users can resume normal activities securely

2. **Provide User Education**
   - Give feedback to the user who reported the phishing attempt
   - Provide specific guidance on how to identify similar attacks
   - Share lessons learned with broader organization if appropriate
   - Expected outcome: Improved user awareness and vigilance

3. **Document Indicators of Compromise**
   - Record all observed IOCs from the phishing campaign
   - Document the tactics, techniques, and procedures used
   - Share relevant details with the security team
   - Expected outcome: Improved detection of similar threats

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection of phishing attempt
- Time from receipt to user reporting
- Number of users who received similar emails
- Number of users who interacted with phishing content
- Whether credentials or systems were compromised

### Detection Improvement

- Review email filtering effectiveness
- Evaluate phishing reporting process
- Consider improvements to user awareness training
- Update automated detection rules based on observed techniques

## Additional Resources

- [Managing the Employee Phishing inbox](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3104448072/Managing+the+employee-phishing+inbox)
- [Email Security Best Practices](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/email-security-best-practices)
- [Phishing Response Playbook](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/phishing-response-playbook)

## Related Artifacts

- [Phishing Indicators Reference](../../artifact-guidelines/phishing-indicators-reference.md)
- [Email Security Configuration Standards](../../artifact-guidelines/email-security-standards.md)