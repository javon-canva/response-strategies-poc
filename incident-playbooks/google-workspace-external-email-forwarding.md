# Google Workspace External Email Forwarding Incident Playbook

## Overview

**Incident Type:** Data Exfiltration Risk  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Google Workspace Email Logs](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
- [Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/google_email_forwarding_rule_creation.md)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/f3728cf0-7e97-42e0-9af8-7a201b99ae67)

## Response Strategy

### Assumptions

- Familiarity with Google Workspace logs and Elastic
- Access to Canva's internal security tools including Harmony
- Understanding of data classification policies

### Considerations

- Some users may legitimately need to forward emails due to their role as contractors
- Different forwarding rules may present different risk levels depending on the destination domain
- Email forwarding could potentially expose sensitive business information or PII
- Historical context is important - is this a new behavior for the user?

## Triage

### Initial Assessment

1. Analyze forwarding rule details:
   - Access the `logs-google_workspace` index in Elastic
   - Use the [pre-designed search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover#/view/f1e93440-b93d-11ee-b097-976c6ac82df4) to find the user's email forwarding activities
   - Note the timestamp when the forwarding rule was first set up
   - If multiple rules exist, document each one separately
   - Identify the destination email address and domain

2. Examine potentially forwarded emails:
   - Open [Checkpoint Harmony](https://portal.checkpoint.com/dashboard/email&collaboration/cgs1#/)
   - Search for emails received by the user since the forwarding rule was established
   - Analyze the content and attachments of these emails to determine sensitivity
   - Check for PII or business sensitive information

3. Determine if forwarding is authorized:
   - Check if the user is a contractor with approved email forwarding
   - Look for any documentation or approval for forwarding in relevant systems
   - Verify if the destination domain is appropriate (personal email vs. legitimate partner)

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Forwarding rule sending emails with PII or highly confidential data; Multiple forwarding rules to external domains; Forwarding to personal email domain with sensitive content; Forwarding rule created outside business hours or from unusual location | 
| Medium   | Forwarding rule with limited business sensitive information; Unauthorized forwarding to legitimate business partner domain; Forwarding rule with recent sensitive emails |
| Low      | Authorized forwarding rule for contractor; Forwarding with no sensitive content detected; Short-term forwarding with appropriate business justification |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Emails contain Personally Identifiable Information (PII) such as:
  - Full names with other identifying data
  - Location data (address, current or past)
  - Email addresses of customers or third parties
  - Date and place of birth
  - Government ID information (Social security number, passport, driving license)

- Emails contain business sensitive information such as:
  - Files labeled "Confidential" or "Restricted"
  - Information listed in the [Data Classification Policy](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/785777456/Data+Classification+and+Handling+Policy)
  - Content that could cause reputational damage to Canva if made public
  - Strategic business plans or financial information

## Investigation

### Investigation Pattern

Follow the [Data Exfiltration Investigation Pattern](../../investigation-patterns/data-exfiltration-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Google Workspace Audit Logs Investigation](../../data-source-procedures/google-workspace-audit-investigation.md)
- [Email Content Analysis](../../data-source-procedures/email-content-analysis.md)

### Investigation Steps

1. **Analyze Forwarding Rule Creation Context**
   - Determine when and how the forwarding rule was created
   - Check if the rule was created from the user's typical location
   - Verify if the rule was created during normal working hours
   - Look for other unusual account activity around the same time
   - Expected outcome: Understanding of the circumstances of rule creation

2. **Assess Data Exposure Risk**
   - Review all emails that may have been forwarded
   - Categorize exposed data according to sensitivity
   - Determine the potential impact of the data exposure
   - Identify any compliance or regulatory implications
   - Expected outcome: Comprehensive understanding of data exposure risk

3. **Evaluate User Intent and Authorization**
   - Contact the user to understand why the forwarding rule was created
   - Verify if the forwarding rule has business justification
   - Check for prior approval or documentation
   - Assess if this is a policy violation or potential malicious intent
   - Expected outcome: Determination of whether forwarding was legitimate

## Containment

### Containment Strategy

Immediately stop potential data exfiltration while preserving evidence for investigation.

### Containment Steps

1. **Document Forwarded Emails**
   - Export a list of all emails that may have been sent externally via Harmony
   - Store this list within the incident channel for reference
   - Include metadata, subjects, timestamps, and sensitivity assessment
   - Expected outcome: Complete record of potentially exposed data

2. **Disable Forwarding Rule**
   - If unauthorized, request the user disable the rule via Slack
   - If the user is unresponsive or if the risk is high, request a Google Admin to disable the rule
   - Consider [requesting access yourself](https://canvadev.atlassian.net/wiki/spaces/IT/pages/2428437190/Google+Admin+Roles+-+Custom+Roles#Request-Access) to expedite the process
   - Expected outcome: External email forwarding is stopped

3. **Notify Stakeholders**
   - Create an incident channel if high severity
   - Inform relevant teams based on the type of data exposed
   - Alert document owners of potentially exposed documents
   - Expected outcome: All stakeholders are informed of the potential exposure

## Eradication

### Eradication Strategy

Ensure that no unauthorized email forwarding mechanisms remain and address the root cause.

### Eradication Steps

1. **Verify Rule Removal**
   - Confirm that all external forwarding rules have been removed
   - Check for any other forwarding mechanisms
   - Verify that no new forwarding rules have been created
   - Expected outcome: All unauthorized forwarding mechanisms are removed

2. **Review Data Access Logs**
   - Examine access logs for sensitive documents that were forwarded
   - Look for any unusual access patterns
   - Verify if the data was accessed from unusual locations
   - Expected outcome: Complete understanding of who accessed the data

3. **Assess Policy Compliance**
   - Determine if existing policies were followed
   - Identify any policy gaps related to email forwarding
   - Recommend policy improvements if needed
   - Expected outcome: Policy compliance assessment completed

## Recovery

### Recovery Strategy

Address the impact of any data exposure and implement measures to prevent recurrence.

### Recovery Steps

1. **Liaise with Relevant Teams**
   - Work with Privacy and Legal teams if PII was exposed
   - Determine necessary actions based on exposure assessment
   - Develop communication plan for affected parties if required
   - Expected outcome: Coordinated response with appropriate teams

2. **Notify Affected Parties**
   - If required by law or policy, communicate with document owners
   - Inform stakeholders whose information may have been exposed
   - Provide context about the document status and potential compromise
   - Expected outcome: Appropriate notifications completed

3. **Update Security Controls**
   - Consider implementing additional controls for email forwarding
   - Review monitoring capabilities for similar activities
   - Recommend training for users on data handling policies
   - Expected outcome: Improved security controls to prevent recurrence

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection of unauthorized forwarding rule
- Time to containment (rule removal)
- Number of emails potentially exposed
- Types and sensitivity of data potentially exposed
- Whether the exposure requires formal notification

### Detection Improvement

- Review email forwarding detection capabilities
- Evaluate alert thresholds for sensitivity
- Consider implementing domain-based filtering for forwarding destinations
- Assess potential improvements to data loss prevention tools

## Additional Resources

- [Personal Data IR Plan](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/1635222606/Personal+Data+Incident+Response+Plan)
- [Security IR Handbook](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2704771472)
- [Google Admin Guide](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2435809569/Google+Admin)
- [Data Classification Policy](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/785777456/Data+Classification+and+Handling+Policy)

## Related Artifacts

- [Email Data Protection Guidelines](../../artifact-guidelines/email-data-protection.md)
- [Data Classification Standards](../../artifact-guidelines/data-classification-standards.md)