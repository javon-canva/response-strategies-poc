# AWS EFS Mount Target File System Deletion Incident Playbook

## Overview

**Incident Type:** AWS Resource Deletion  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/security/rules/id/84480cda-f532-4c96-9136-4fefeccc30dd/alerts)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/aws_efs_fileshare_mount_modified_or_deleted.md)

## Response Strategy

### Assumptions

- Understand what [AWS EFS](https://aws.amazon.com/efs/when-to-choose-efs/) is and its use cases at Canva
- Access to Canva's internal security tools
- Elastic File System (EFS) is primarily used by analytics and machine learning teams at Canva, especially for Anyscale

### Considerations

- This detection is unlikely to fire as EFS usage at Canva is limited to specific teams
- EFS resources may be in test/development environments where deletion is part of normal operations
- Unauthorized deletions could impact machine learning workloads and data availability

## Triage

### Initial Assessment

1. Examine the alert details to identify the AWS account, EFS resources affected, and actor information
2. Use the [Pre-Configured Elastic Search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover#/view/8ccf8861-bcd5-48f5-8de6-4532ef4db552?_g=()) to gather complete information
3. Follow the [AWS CloudTrail Investigation Procedure](../../data-source-procedures/aws-cloudtrail-investigation.md) to identify the actor

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Production EFS resources deleted without approval or proper process; Multiple EFS resources affected; Suspected malicious intent | 
| Medium   | Single EFS resource deleted without proper process; Impact limited to non-critical systems |
| Low      | Deletion performed through proper channels (Terraform/CI/CD) with appropriate approvals |

### Validation Criteria

- Determine if the deletion was performed via CI/CD or Terraform
- Check if the user name, source IP, and user agent are consistent with legitimate deployment patterns
- Verify if the deletion correlates with a known GitHub PR or Jira ticket
- Search Slack for context about the username or AWS resources
- If needed, contact the identified user directly to validate the activity

## Investigation

### Investigation Pattern

Follow the [AWS Resource Investigation Pattern](../../investigation-patterns/aws-resource-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [BuildKite Job Investigation](../../data-source-procedures/buildkite-job-investigation.md)

### Investigation Steps

1. **Identify Actor and Context**
   - Determine which user performed the action and what role they used
   - Identify the AWS account where the action occurred
   - Check if the action was performed via CI/CD (Terraform in BuildKite)
   - Examine the source IP address and user agent information

2. **Validate Legitimacy**
   - For CI/CD actions, locate the associated PR and verify approvals
   - For manual actions, search Slack for context about the activity
   - Contact the user directly if more clarity is needed

3. **Assess Impact**
   - Identify all affected EFS mounts and file systems
   - Determine the importance of the deleted resources
   - Identify dependent workloads and services

## Containment

### Containment Strategy

If the deletion is determined to be unauthorized or malicious, contain the actor to prevent further damage.

### Containment Steps

1. **Suspend User Access**
   - Follow the [User Containment SOP](../../sops/user-account-containment-sop.md) to suspend the user's access
   - Use the [AWS Bootkicker script](https://github.com/Canva/infrastructure/blob/master/go/security/incident-response/bootkick/aws/cmd/bootkick.sh) to revoke active AWS sessions
   - Expected outcome: User can no longer access AWS resources

2. **Notify Resource Owners**
   - Identify and contact the owners of the affected EFS resources
   - Create an incident channel and invite the relevant stakeholders
   - Expected outcome: Resource owners are aware and can assist with impact assessment

## Eradication

### Eradication Strategy

Ensure that any unauthorized access is completely removed and determine how to restore the deleted resources.

### Eradication Steps

1. **Investigate Additional Activity**
   - Review all recent actions by the same user across AWS and other platforms
   - Check for any additional resources that may have been affected
   - Expected outcome: Complete understanding of the scope of unauthorized activity

2. **Revoke Compromised Credentials**
   - If credentials were compromised, ensure all are rotated
   - Verify no unauthorized IAM changes persist
   - Expected outcome: No lingering access for unauthorized actors

## Recovery

### Recovery Strategy

Work with resource owners to restore deleted EFS resources and the data they contained.

### Recovery Steps

1. **Restore EFS Resources**
   - Work with the resource owners to recreate the EFS resources as needed
   - Re-ingest data from Snowflake or other relevant data warehouses
   - Expected outcome: EFS resources and data are restored

2. **Verify Data Integrity**
   - Confirm with resource owners that all critical data has been restored
   - Validate that dependent systems are functioning properly
   - Expected outcome: Systems return to normal operation

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to recovery
- Amount of data lost or temporarily unavailable
- Number of systems affected

### Detection Improvement

- Evaluate if detection rules for EFS modifications can be improved
- Consider implementing additional preventative controls for critical EFS resources

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [Post-Incident Action Items Process](https://docs.canva.tech/operations/reliability/incidents/post-incident/)

## Related Artifacts

- [AWS Resource Protection Guidelines](../../artifact-guidelines/aws-resource-protection.md)
- [Data Recovery Runbooks](../../artifact-guidelines/data-recovery-runbooks.md)