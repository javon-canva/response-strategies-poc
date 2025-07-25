# Indicator of Compromise (IoC) Match Incident Playbook

## Overview

**Incident Type:** Threat Intelligence Match  
**Severity Levels:** Variable (Low to High)  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Threat Intelligence Platform Alerts](https://ui.threatstream.com/search)
- [Network Detection Systems](https://d-r-primary.kb.us-east-1.aws.found.io:9243/)
- [Endpoint Detection & Response](https://usea1-016.sentinelone.net/dashboard)
- Previous Response Examples:
  - [Network IoC Detection](https://pew-pew.canva-internal.com/caseview/86f4c07e-1523-4918-86cf-b4eecdccdc81)
  - [Host-Level IoC Detection](https://pew-pew.canva-internal.com/caseview/40df6827-de19-4f04-a1fa-151178f6985b)

## Response Strategy

### Assumptions

- Access to threat intelligence platforms and security tools
- Basic understanding of IoC types and classifications
- Ability to interact with security controls and affected systems

### Considerations

- High confidence IoCs should have already been propagated to security controls (CloudFlare, EDR, etc.) for blocking
- This playbook focuses on determining the root cause or initial vector of the threat
- IoCs may be misclassified or outdated, requiring verification
- Some IoCs may be matched from lists not owned by Canva (e.g., CloudFlare managed rules)

## Triage

### Initial Assessment

1. Identify the basic details of the match:
   - User and host involved in the interaction
   - Type of IoC (domain, IP, file hash, etc.)
   - Outcome of the interaction (blocked or allowed)
   - Timestamp and duration of activity

2. Verify the IoC's nature and reputation:
   - Access [Anomali](https://ui.threatstream.com/search) to understand the context and source
   - Check [Recorded Future](https://sandbox.recordedfuture.com/dashboard) for additional context
   - Use [URL Scan](https://www.urlscan.io/) for web-based IoCs
   - Follow the [Using Sandbox for Malware Analysis](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3139201940/Using+Sandbox+for+Malware+Analysis) SOP if needed

3. Determine the source of the interaction:
   - Search for activity immediately preceding and following the IoC match
   - Use template searches in security tools:
     - [Cloudflare DNS search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/9LWpL)
     - [Sentinel One DNS search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/10mnq)
   - Determine if files were downloaded or if suspicious websites were visited

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Current, high-confidence IoC; Interaction allowed; Evidence of additional malicious activity; Sensitive data or systems involved | 
| Medium   | Current, moderate-confidence IoC; Interaction blocked but suspicious activity observed; Limited scope of systems affected |
| Low      | Outdated or low-confidence IoC; Interaction completely blocked; No evidence of additional suspicious activity |

### Validation Criteria

After completing triage steps, assess if the investigation warrants escalation to a security incident by considering:

- Is the IoC still classified as malicious, or has it been misclassified?
- Was the interaction successfully blocked or was it allowed?
- What was the source of the activity? How did the interaction take place?
- Is there evidence of any suspicious activity before or after the match?
- Was anything downloaded to the machine or were any unauthorized changes made?

Use the [IR handbook](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2704771472/Security+Incident+Response+Handbook#Raising-an-incident) and [severity matrix](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2704771472/Security+Incident+Response+Handbook#Severity-matrix) to guide your decision.

If not deemed an incident, summarize findings in the Cydarm case, add relevant tags, and close the case.

## Investigation

### Investigation Pattern

Follow the [Threat Intelligence Match Investigation Pattern](../../investigation-patterns/threat-intelligence-match-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Network Traffic Analysis](../../data-source-procedures/network-traffic-analysis.md)
- [Endpoint Activity Analysis](../../data-source-procedures/endpoint-activity-analysis.md)
- [Threat Intelligence Platform Analysis](../../data-source-procedures/threat-intelligence-platform-analysis.md)

### Investigation Steps

1. **Analyze the Matched IoC Context**
   - Examine the threat intelligence platform for IoC context and attribution
   - Determine the threat actor, campaign, or malware family associated with the IoC
   - Check if the IoC is part of a larger set of related indicators
   - Document the IoC's classification, confidence level, and first/last seen dates

2. **Review Surrounding Activity**
   - Identify all activity from the affected user/system within a relevant timeframe (before and after)
   - Look for evidence of unusual network connections, file downloads, or process executions
   - Search for related IoCs that may have been missed
   - Map out the chronology of events leading to the IoC match

3. **Determine Impact and Scope**
   - Assess whether any payload was delivered or executed
   - Identify if data exfiltration may have occurred
   - Determine if additional systems were affected
   - Evaluate potential business impact based on affected systems and data

## Containment

### Containment Strategy

If the IoC match is confirmed to represent a legitimate threat, contain the affected resources to prevent further impact.

### Containment Steps

1. **Contain the User**
   - Follow the [User Account Containment SOP](../../sops/user-account-containment-sop.md)
   - Use the Cydarm form to initiate containment process
   - Expected outcome: User access is restricted to prevent further compromise

2. **Isolate the Device**
   - Disconnect the user's device from the network via SentinelOne
   - From the `Sentinels` side tab, select the endpoint and choose `Actions` → `Response` → `Disconnect from Network`
   - For Severity 1 and 2 incidents, containment should be implemented immediately
   - Expected outcome: Device is isolated from the network to prevent lateral movement

3. **Communicate the Incident**
   - Inform relevant stakeholders following the [Incident Communication Guidelines](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2428863510/Incident+Communication)
   - Provide clear instructions to affected users
   - Expected outcome: All necessary parties are informed of the situation and required actions

## Eradication

### Eradication Strategy

Remove the threat from the environment and identify any additional IoCs to enhance defenses.

### Eradication Steps

1. **Remove Malicious Content**
   - Use EDR capabilities to remove identified malicious files or processes
   - Remediate any compromised accounts or access
   - Expected outcome: All malicious artifacts are removed from the environment

2. **Update Threat Intelligence**
   - Use the [Sending Observables to Anomali from Cydarm](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2828241699/Sending+Observables+to+Anomali+from+Cydarm) SOP
   - Add any newly identified IoCs to the threat intelligence platform
   - Expected outcome: Enhanced detection capabilities for similar threats

## Recovery

### Recovery Strategy

Restore normal operations after confirming the threat has been eliminated.

### Recovery Steps

1. **Restore User Access**
   - Follow the un-contain procedure in the [User Account Containment SOP](../../sops/user-account-containment-sop.md#un-contain-user-procedure)
   - Verify the user can access necessary resources
   - Expected outcome: User regains appropriate access to systems

2. **Reconnect the Device**
   - Reconnect the user's device to the network via SentinelOne
   - From the `Sentinels` side tab, select the endpoint and choose `Actions` → `Response` → `Reconnect to Network`
   - Expected outcome: Device is reconnected to the network safely

3. **Communicate Resolution**
   - Inform the affected user that the incident has been remediated
   - Provide guidance on how to report any unusual behavior
   - Expected outcome: User is aware that normal operations can resume

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to eradication
- False positive rate for IoC matches
- Number of systems affected

### Detection Improvement

- Review and adjust IoC confidence levels based on investigation findings
- Evaluate the effectiveness of security controls that should have blocked the threat
- Consider implementing additional detection capabilities if gaps were identified

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [Using Sandbox for Malware Analysis](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3139201940/Using+Sandbox+for+Malware+Analysis)
- [Incident Communication Guidelines](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2428863510/Incident+Communication)

## Related Artifacts

- [IoC Management Guidelines](../../artifact-guidelines/ioc-management-guidelines.md)
- [Threat Hunting Queries](../../artifact-guidelines/threat-hunting-queries.md)