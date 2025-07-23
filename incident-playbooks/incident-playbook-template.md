# [Incident Type] Incident Playbook

## Overview

**Incident Type:** [Specific incident type, e.g., Malware, Credential Theft, Data Exfiltration]  
**Severity Levels:** [Critical/High/Medium/Low]  
**Response Team:** [Primary team(s) responsible for handling this incident type]  
**Version:** [X.Y]  
**Last Updated:** [YYYY-MM-DD]  
**Maintainer:** [Name/Team]

## Detection Sources

- [List detection sources, e.g., SIEM, EDR, Cloud Security Platform]
- [Include links to detection rules/strategies if available]
- [Reference relevant dashboards if applicable]

## Response Strategy

### Assumptions

- [List assumptions about the incident context]
- [Note any prerequisites or required knowledge]
- [Include potential false positive scenarios]

### Considerations

- [Security/privacy considerations]
- [Business impact considerations]
- [Cross-team coordination requirements]
- [Legal/compliance requirements if applicable]

## Triage

### Initial Assessment

1. [Step 1: How to initially validate the alert]
2. [Step 2: Reference to relevant SOP, e.g., `[Initial Evidence Collection](../../sops/templates/evidence-collection.md)`]
3. [Additional assessment steps with specific criteria]

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | [Specific conditions that make this a critical incident] |
| High     | [Specific conditions that make this a high severity incident] |
| Medium   | [Specific conditions that make this a medium severity incident] |
| Low      | [Specific conditions that make this a low severity incident] |

### Validation Criteria

- [Specific criteria to determine if this is a true positive]
- [Conditions that would indicate a false positive]
- [Decision points for escalation]

## Investigation

### Investigation Pattern

Follow the [relevant investigation pattern](../../investigation-patterns/templates/investigation-pattern-template.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Data Source 1](../../data-source-procedures/templates/data-source-template.md)
- [Data Source 2](../../data-source-procedures/templates/data-source-template.md)

### Investigation Steps

1. **[Step 1 Title]**
   - [Detailed instructions with specific commands/queries]
   - [Expected outcomes]
   - [References to related SOPs if applicable]

2. **[Step 2 Title]**
   - [Detailed instructions with specific commands/queries]
   - [Expected outcomes]
   - [References to related SOPs if applicable]

3. **[Additional steps as needed]**

## Containment

### Containment Strategy

[Overview of containment approach specific to this incident type]

### Containment Steps

1. **[Containment Step 1]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs, e.g., `[User Containment](../../sops/templates/user-containment.md)`]
   - [Expected outcomes]

2. **[Containment Step 2]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs if applicable]
   - [Expected outcomes]

## Eradication

### Eradication Strategy

[Overview of eradication approach for this incident type]

### Eradication Steps

1. **[Eradication Step 1]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs if applicable]
   - [Expected outcomes and verification methods]

2. **[Eradication Step 2]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs if applicable]
   - [Expected outcomes and verification methods]

## Recovery

### Recovery Strategy

[Overview of recovery approach for this incident type]

### Recovery Steps

1. **[Recovery Step 1]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs if applicable]
   - [Expected outcomes and verification methods]

2. **[Recovery Step 2]**
   - [Detailed instructions with specific commands/actions]
   - [Reference relevant SOPs if applicable]
   - [Expected outcomes and verification methods]

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/templates/post-incident-review.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to eradication
- Time to recovery
- [Additional incident-specific metrics]

### Detection Improvement

[Recommendations for improving detection of this incident type]
[Specific detection engineering tasks that should be considered]

## Additional Resources

- [Reference 1: Link or document]
- [Reference 2: Link or document]
- [Additional references as needed]

## Related Artifacts

- [Link to related technical artifacts following artifact guidelines](../../artifact-guidelines/templates/artifact-guideline-template.md)
- [Additional artifacts as needed]