# Publicly Available Document with PII Incident Playbook

## Overview

**Incident Type:** Data Exposure (PII)  
**Severity Levels:** Low/Medium/High/Critical  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Checkpoint Harmony DLP Solution](https://checkpoint.com/harmony/)
- [Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/harmony_public_google_document_dlp.md)
- [Previous Response Examples - PII](https://pew-pew.canva-internal.com/caseview/f050aba3-2a1a-49b7-81de-62248fd40663)
- [Previous Response Examples - Non-PII](https://pew-pew.canva-internal.com/caseview/c1f5477c-3648-44a0-bf67-92dbb33596c2)

## Response Strategy

### Assumptions

- Access to Google Workspace logs and administrative controls
- Access to the identified documents for review
- Understanding of data privacy and protection regulations
- Authority to modify document sharing permissions
- Ability to contact document owners

### Considerations

- Data sensitivity and privacy must be prioritized throughout the response
- Compliance with data protection policies is essential
- Some document sharing may be legitimate and intentional
- Balance between security controls and business needs
- Potential legal and regulatory reporting requirements for data breaches

## Triage

### Initial Assessment

1. Identify the affected document:
   - Locate the document ID from the Harmony alert in Cydarm
   - Access the document directly to inspect its contents
   - Document the type of information contained (PII, confidential data, non-sensitive)
   - Catalog specific PII elements present (if any)
   - Assess the sensitivity level of the exposed information

2. Determine sharing status:
   - Verify current sharing settings of the document
   - Check Google Workspace logs to confirm if the file is publicly accessible
   - Use [this search template in Elastic](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/zsxtL) (update the "google_workspace.drive.file.id" filter)
   - Document when the file was made public and by whom
   - Check if any Eva automation responses were triggered

3. Contact document owner:
   - Reach out to the Canvanaut who owns the document
   - Verify if the document was intentionally made public
   - Document their justification if sharing was intentional
   - Assess their understanding of data handling policies

4. Check for access evidence:
   - Review access logs for the document
   - Determine if external parties accessed the document
   - Check for download events from external sources
   - Document any evidence of data exfiltration

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | Confirmed exposure of highly sensitive PII (SSNs, financial data) to public internet; Evidence of mass downloads by external parties; Potential regulatory breach affecting large number of individuals; Intentional malicious data exposure |
| High     | Exposure of sensitive PII or confidential business data; Limited evidence of external access; Moderate number of affected individuals; Document exposed for extended period; Regulatory reporting likely required |
| Medium   | Limited PII exposure with minimal sensitivity; Brief exposure period; No evidence of external access; Small number of affected individuals; Regulatory reporting may be required |
| Low      | No actual PII or minimal non-sensitive information exposed; Document public for very short period; No evidence of external access; Document owner quickly remediated; No regulatory impact |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Document contains legitimate PII or confidential/sensitive data
- Document was accessed by anyone external to Canva
- Files were downloaded by external parties
- Document was made public without valid business justification
- Potential compliance or regulatory violation exists

## Investigation

### Investigation Pattern

Follow the [Data Exposure Investigation Pattern](../../investigation-patterns/data-exposure-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Google Workspace Audit Investigation](../../data-source-procedures/google-workspace-audit-investigation.md)
- [Data Access Log Analysis](../../data-source-procedures/data-access-log-analysis.md)
- [DLP Alert Analysis](../../data-source-procedures/dlp-alert-analysis.md)

### Investigation Steps

1. **Document Content Analysis**
   - Thoroughly examine the document contents
   - Identify and catalog all PII or sensitive data elements
   - Document the structure and context of the exposed information
   - Assess potential impact if the data were to be misused
   - Expected outcome: Complete inventory of exposed sensitive data

2. **Access and Sharing Timeline Analysis**
   - Create a timeline of document creation and sharing changes
   - Identify when the document was made publicly accessible
   - Determine which user made the sharing change and from what IP/location
   - Check for any previous restricted sharing before public exposure
   - Expected outcome: Complete timeline of document sharing history

3. **External Access Assessment**
   - Review access logs for the document during exposure period
   - Identify any non-Canva domains that accessed the document
   - Document timestamps and frequency of external access
   - Check for download, copy, or print events from external sources
   - Expected outcome: Evidence of any unauthorized access or exfiltration

4. **User Activity Analysis**
   - Review the document owner's recent activity patterns
   - Check for other documents shared publicly by the same user
   - Assess if the sharing was a one-time incident or pattern
   - Interview the user to understand intent and knowledge of policies
   - Expected outcome: Understanding of root cause and risk of recurrence

## Containment

### Containment Strategy

Quickly restrict access to the exposed document while preserving evidence for further investigation and potential legal requirements.

### Containment Steps

1. **Preserve Evidence**
   - Create an incident folder in [Google Drive](https://drive.google.com/drive/u/0/folders/1Py_7TYKTsK7nGqEPLFVIT9A0lx-6chdC)
   - Make a copy of the file with original sharing settings
   - Document all metadata including sharing settings and access logs
   - Take screenshots of relevant data and sharing configurations
   - Expected outcome: Preserved evidence for investigation and legal review

2. **Restrict Document Access**
   - Follow the [Remove anyone can view permissions from Google Drive Files SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3238561498/Removing+anyone+can+view+permissions+from+Google+Drive+Files)
   - Change sharing settings to "Restricted" or "Canva" only
   - Verify the change takes effect by checking from a non-Canva account
   - Document the time when access was restricted
   - Expected outcome: Public access to sensitive document is removed

3. **Account Actions (if necessary)**
   - If user is unaware of the activity or uncooperative
   - If suspicious or malicious intent is suspected
   - [Suspend user access](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker)
   - Document justification for account suspension
   - Expected outcome: Prevention of further unauthorized sharing

## Eradication

### Eradication Strategy

Identify the full scope of exposed data, address any related vulnerabilities, and ensure proper handling of the exposure incident.

### Eradication Steps

1. **Assess External Access Impact**
   - Use [this search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/HXRuS) to check for external downloads
   - Look specifically for `download` or `source_copy` events shared externally
   - Document all instances of external access with timestamps and domains
   - Determine the scope of potential data compromise
   - Expected outcome: Complete understanding of data exposure extent

2. **Legal and Compliance Notification**
   - If PII/confidential data was accessed externally
   - Page [legal on-call](https://canva.app.opsgenie.com/schedule/whoIsOnCall) `CURRENT-privacy-and-product-team_schedule`
   - Create a separate incident legal channel for coordination
   - Share all evidence and findings with the legal team
   - Expected outcome: Proper legal assessment and guidance for next steps

3. **Additional Document Review**
   - Check other documents owned by the same user
   - Look for similar sharing settings or additional exposure
   - Review sharing history for patterns of inappropriate sharing
   - Correct any additional instances of over-sharing
   - Expected outcome: Identification and remediation of related exposures

## Recovery

### Recovery Strategy

Restore secure operations, prevent recurrence, and ensure affected parties understand proper data handling practices.

### Recovery Steps

1. **User Education**
   - Contact document owner to explain the incident
   - Provide training on proper document sharing practices
   - Review relevant data handling policies with the user
   - Document the user's understanding and commitment to proper practices
   - Expected outcome: Improved user awareness and reduced recurrence risk

2. **Account Restoration**
   - If account was suspended, assess if restoration is appropriate
   - Follow [user account restoration procedure](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker#Un-contain-user-procedure)
   - Implement additional monitoring if necessary
   - Document restoration decision and justification
   - Expected outcome: Restored user access with appropriate controls

3. **Process Improvement**
   - Identify any policy or process gaps that contributed to the incident
   - Recommend improvements to document sharing controls
   - Consider additional DLP rules or monitoring if appropriate
   - Document lessons learned and share with relevant teams
   - Expected outcome: Improved security controls to prevent similar incidents

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time from exposure to detection
- Time from detection to containment
- Number and types of PII elements exposed
- Number of external accesses during exposure
- Whether external downloads occurred
- Root cause (intentional vs. accidental)
- Effectiveness of DLP controls

### Detection Improvement

- Review Harmony DLP rules for effectiveness
- Consider additional monitoring for document sharing
- Evaluate alert prioritization and response procedures
- Enhance user behavior analytics for sharing anomalies
- Improve integration between DLP and response systems

## Additional Resources

- [Personal Data Handling Policy](https://hub.canva.world/space/CANVAKB/2937160413/Personal+Data+Handling+Policy)
- [Information Security Policy](https://hub.canva.world/space/CANVAKB/2935504933/Information+Security+Policy)
- [Legal Data Breach Response Procedures](https://canvadev.atlassian.net/wiki/spaces/LEGAL/pages/2980184564/Data+Breach+Response)
- [Google Workspace Security Best Practices](https://canvadev.atlassian.net/wiki/spaces/SECURITY/pages/2786918531/Google+Workspace+Security)

## Related Artifacts

- [Data Classification Guidelines](../../artifact-guidelines/data-classification-guidelines.md)
- [Document Sharing Standards](../../artifact-guidelines/document-sharing-standards.md)
- [PII Handling Requirements](../../artifact-guidelines/pii-handling-requirements.md)