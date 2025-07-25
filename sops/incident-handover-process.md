# Incident Response Handover Process

## Overview

**Purpose:** Ensure seamless transition of IR-related tasks, alerts, and incidents between teams across different time zones  
**SOP Type:** Procedural  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Scope

**Applies to:**
- All Detection & Response team members
- Any security incident responders
- Ongoing incidents or alerts requiring handover
- Cross-timezone collaboration

**Related Documents:**
- [Incident Response Playbooks](../incident-playbooks/)
- [Severity Matrix SOP](./severity-matrix.md)
- [Incident Documentation SOP](./incident-documentation.md)

## Prerequisites

### Required Permissions

- Access to Cydarm case management system
- Access to Slack incident channels
- Access to IR-related documentation

### Required Tools

| Tool | Purpose | Access Information |
|------|---------|-------------------|
| Cydarm | Incident case management | SSO access via Okta |
| Slack | Communication and handover threads | Access to #d-r-ir-focus and incident channels |
| Jira | Task tracking for follow-up items | SSO access via Okta |

### Assumptions

- Familiarity with incident response procedures
- Understanding of time zone differences between teams
- Access to relevant communication channels

### Considerations

- Handovers must include sufficient context for the receiving team to continue work effectively
- Sensitive information should be handled according to data classification policies
- Follow-up and acknowledgment are critical for ensuring nothing falls through the cracks

## Procedure

### Step 1: Timezone Awareness

1. Be aware of different time zones between Detection & Response Pods
2. Primary responders in each region are responsible for coordinating handovers
3. Proactively communicate if you anticipate a potential handover at the end of your workday

**Expected Outcome:** Teams are aligned on handover timing and expectations

**Verification Method:** Regular check-ins at the beginning and end of regional working hours

### Step 2: Handover Thread Usage

1. Use the automated handover threads created daily in #d-r-ir-focus
2. Post all handover information in the appropriate thread (UK > ANZ or ANZ > UK)
3. Avoid using direct messages or individual tags for handover communication

**Expected Outcome:** Centralized, visible handover communication

**Verification Method:** All handovers are documented in the designated thread

### Step 3: Incident Handovers

1. For each incident requiring handover, prepare a summary using this format:
   ```
   - [[Incident Title](https://pew-pew.canva-internal.com/caseview/<CYDARM_CASE_ID>)] - [Slack Channel](https://link_to_incident_slack_channel.com) - <Summary of progress, mitigation status, and notable items>
   ```
2. Tag the incident in Cydarm with handover:uk-anz or handover:anz-uk as appropriate
3. Ensure the receiving team has access to relevant incident channels

**Expected Outcome:** Clear documentation of incident status and next steps

**Verification Method:** Receiving team acknowledges handover and can continue the investigation

### Step 4: Alert Handovers

1. Complete as much triage as possible before handover
2. For alerts requiring handover, use this format:
   ```
   - Alert: [Alert Title](https://pew-pew.canva-internal.com/caseview/<CYDARM_CASE_ID>) - <Summary of progress and notable items>
   ```
3. Tag the alert in Cydarm with handover:uk-anz or handover:anz-uk as appropriate
4. Clearly indicate any alerts left in Triage state

**Expected Outcome:** Clear documentation of alert status and required actions

**Verification Method:** Receiving team acknowledges handover and continues alert triage

### Step 5: Task Handovers

1. For other tasks requiring handover, use this format:
   ```
   - Task: [Project description](https://canvadev.atlassian.net/browse/<TARGET_JIRA_KEY>) - Summary of action items and assigned person.
   ```
2. Include links to relevant documentation and clarify expectations

**Expected Outcome:** Clear understanding of task requirements and next steps

**Verification Method:** Receiving team acknowledges handover and can continue the task

## Special Cases and Exceptions

### Exception 1: Urgent Incidents

[Description of exception scenario]
For urgent incidents with potential immediate impact, page the on-call responder rather than relying solely on the handover process.

**Handling Instructions:**
1. Follow the [Incident Escalation SOP](./incident-escalation.md)
2. Document the escalation in the handover thread
3. Ensure a verbal handover takes place if possible

### Exception 2: Public Holidays

During public holidays in either region, additional coordination may be required.

**Handling Instructions:**
1. Notify the other region of upcoming public holidays in advance
2. Determine coverage plan and document in the handover thread
3. Consider adjusting alert thresholds or automation during periods of reduced staffing

## Troubleshooting

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| Handover missed | Thread not monitored | Send a direct message to the Primary in the receiving region |
| Incomplete information | Rushed handover | Use the standardized templates and follow up as needed |
| Uncertainty about next steps | Complex incident | Schedule a brief call between outgoing and incoming teams |

## Approval and Notification Requirements

| Scenario | Approval Required From | Notification Required To |
|---------|------------------------|--------------------------|
| Changing investigation direction | N/A | Original investigator, documented in Cydarm |
| Closing a handed-over incident | Team lead | Original investigator |
| Changing incident severity | Team lead | #d-r-ir-focus channel |

## Documentation Requirements

All handover communication should be documented in the handover thread and relevant Cydarm cases.

**Required Documentation Fields:**
- Links to relevant Cydarm cases
- Current status and progress
- Suggested next steps
- Any critical information or blockers

## Additional Resources

- [UK Priorities DACI](https://www.canva.com/design/DAGWWzvjh0o)
- [Security Incident Response Handbook](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2704771472/Security+Incident+Response+Handbook)
- [Severity Matrix](https://docs.example.com/severity-matrix)