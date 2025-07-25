# AWS GuardDuty IAM User Anomalous Behavior Incident Playbook

## Overview

**Incident Type:** Anomalous IAM Activity  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [AWS GuardDuty](https://console.aws.amazon.com/guardduty/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/)
- [GuardDuty Finding Types Reference](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html)

## Response Strategy

### Assumptions

- Familiarity with AWS & CloudTrail logs
- Access to Canva's internal security tools
- Understanding of IAM roles and permissions

### Considerations

- Sparsely used principals in AWS can trigger false positive alerts when they are used, as the preexisting activity baseline might not be significant
- GuardDuty is a managed service with limited tuning capabilities
- The same IAM user performing different actions across multiple environments may trigger multiple alerts

## Triage

### Initial Assessment

1. Identify the specific GuardDuty finding type (e.g., `IAMUser/AnomalousBehavior:IAMPrincipal/`) from the alert
2. Review the `aws.guardduty.service.action.awsApiCallAction.api` field to understand what specific API action triggered the alert
3. Correlate the finding type with the [GuardDuty Finding Type documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html) to understand the significance
4. Follow the [AWS CloudTrail Investigation Procedure](../../data-source-procedures/aws-cloudtrail-investigation.md) to analyze the activity

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Sensitive actions performed by rarely used principals; Activity outside business hours; Multiple anomalous actions in sequence; Unauthorized access to sensitive resources | 
| Medium   | Unusual but potentially legitimate actions; Single anomalous action with identifiable actor; Known principal performing new actions |
| Low      | Confirmed as part of authorized activity; Infrequently used role or user with proper authorization |

### Validation Criteria

- If the activity corresponds to a BuildKite job, GitHub PR, and Jira ticket (all three), and the user/role is only sparsely used, this is likely a false positive
- For frequently used principals, examine why the specific API action was required in the context of the principal's normal activities
- Examine the `aws.cloudtrail.request_parameters` field to identify affected resources and search for related activity
- Perform a timeline search for days or weeks before the event to establish a baseline of how commonly the user or role is used

## Investigation

### Investigation Pattern

Follow the [AWS IAM Investigation Pattern](../../investigation-patterns/aws-iam-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [AWS GuardDuty Investigation](../../data-source-procedures/aws-guardduty-investigation.md)

### Investigation Steps

1. **Analyze the Specific GuardDuty Finding**
   - Review the complete finding details in GuardDuty or Elastic
   - Identify the specific IAM principal (user or role) involved
   - Determine which API action triggered the finding
   - Note any additional context provided by GuardDuty (IP address, resource targets)

2. **Establish Principal's Normal Behavior**
   - Search CloudTrail for historical activity of the same principal over 7-30 days
   - Identify patterns in actions, timing, and resources accessed
   - Determine if the principal is typically used by humans, services, or automation

3. **Analyze the Anomalous Activity**
   - Examine the complete session in which the anomalous activity occurred
   - Identify all actions performed before and after the triggering action
   - Determine what resources were affected
   - Look for related activity from the same source IP or session ID

4. **Validate Legitimacy**
   - For service roles, trace to BuildKite jobs, GitHub PRs, and Jira tickets
   - For human users, contact the user directly to confirm the activity
   - Cross-reference with change management records or deployment schedules

## Containment

### Containment Strategy

If the activity is determined to be unauthorized or potentially malicious, immediate containment actions should be taken to limit further access.

### Containment Steps

1. **Revoke Active Sessions**
   - Use the [AWS Session Revocation SOP](../../sops/aws-session-revocation.md) to terminate the suspicious session
   - Implement temporary restrictions on the affected principal if needed
   - Expected outcome: Unauthorized session is terminated, preventing further actions

2. **Restrict the IAM Principal**
   - Apply a restrictive policy to the IAM user or role
   - Consider temporarily attaching a deny-all policy if high risk
   - Expected outcome: Principal can no longer perform privileged actions

## Eradication

### Eradication Strategy

Determine how the anomalous behavior occurred and eliminate any persistent access paths.

### Eradication Steps

1. **Investigate Credential Access**
   - Determine how the credentials were obtained
   - Check for compromised access keys or role assumption chains
   - Expected outcome: Understanding of how access was obtained

2. **Review and Adjust Permissions**
   - Identify and remove any unnecessary permissions
   - Implement least privilege adjustments as needed
   - Expected outcome: Principal has only necessary permissions

## Recovery

### Recovery Strategy

Restore proper access controls and monitoring.

### Recovery Steps

1. **Restore Proper Access Controls**
   - Restore appropriate permissions for legitimate workloads
   - Implement improved permission boundaries if needed
   - Expected outcome: Legitimate operations can continue securely

2. **Enhance Monitoring**
   - Implement additional monitoring for the affected principal
   - Create custom alerts for sensitive actions
   - Expected outcome: Quicker detection of future anomalous behavior

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to determination (legitimate vs. malicious)
- Time to containment (if applicable)
- Number of resources accessed
- Number of API calls made

### Detection Improvement

- Consider implementing CloudTrail Insights for more precise anomaly detection
- Review IAM permission boundaries to reduce excessive permissions
- Document baseline activity patterns for critical roles

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation on GuardDuty IAM Findings](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-iam.html)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)

## Related Artifacts

- [AWS IAM Permission Review Guidelines](../../artifact-guidelines/aws-iam-permission-review.md)
- [AWS GuardDuty Tuning Documentation](../../artifact-guidelines/aws-guardduty-tuning.md)