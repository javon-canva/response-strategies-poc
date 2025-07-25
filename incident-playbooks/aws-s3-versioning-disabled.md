# S3 Bucket Versioning Disabled Incident Playbook

## Overview

**Incident Type:** AWS Configuration Change  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/security/rules/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/aws_s3_bucket_versioning_disabled.md)

## Response Strategy

### Assumptions

- Familiarity with AWS & CloudTrail logs
- Access to Canva's internal security tools
- Understanding of S3 versioning functionality and its importance for data protection

### Considerations

- Disabling versioning on S3 buckets could be part of a data destruction attempt
- Configuration changes might be legitimate as part of infrastructure changes
- Critical data could be at risk if versioning is disabled before a delete operation

## Triage

### Initial Assessment

1. Examine the CloudTrail logs using the [CloudTrail Investigation Procedure](../../data-source-procedures/aws-cloudtrail-investigation.md)
2. Identify the affected S3 bucket, AWS account, and region from the `aws.cloudtrail.request_parameters` field
3. Check the sequence of actions performed before and after versioning was disabled

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Versioning disabled on critical production buckets; Multiple buckets affected; Followed by delete operations; Suspected malicious intent | 
| Medium   | Versioning disabled on non-critical buckets; Limited scope; No follow-up delete operations |
| Low      | Versioning change performed through proper channels with appropriate approvals |

### Validation Criteria

- Determine who performed the action (`user.id`) and with what role (`user.role`)
- Check what user agent (`user_agent.name`) was used to perform the action
- Search internal resources (Slack, Jira, GitHub PRs) for evidence that the activity was performed for legitimate reasons
- Cross-reference with change management records or directly confirm with the involved user or team

## Investigation

### Investigation Pattern

Follow the [AWS Resource Investigation Pattern](../../investigation-patterns/aws-resource-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [GitHub Repository Investigation](../../data-source-procedures/github-repository-investigation.md)

### Investigation Steps

1. **Identify Affected Resources and Context**
   - Check [CloudTrail logs](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/RULdt) for complete information
   - Identify the target bucket and account from `aws.cloudtrail.request_parameters` and `cloud.account.name`
   - Review actions performed before and after versioning was disabled
   - Determine if any objects were deleted after versioning was disabled

2. **Establish Actor and Intent**
   - Identify who performed the action (`user.id`), their role (`user.role`), and the user agent used
   - Check if the action was performed via automated means (Terraform, CI/CD) or manually
   - Search for related GitHub PRs, Jira tickets, or Slack discussions that might explain the activity
   - Contact the user directly if needed for clarification

3. **Assess Impact**
   - Determine the criticality of the affected bucket and its contents
   - Check if any data was deleted after versioning was disabled
   - Evaluate whether the change was part of a legitimate workflow or potentially malicious

## Containment

### Containment Strategy

If the activity is determined to be unauthorized or malicious, contain the actor to prevent further damage.

### Containment Steps

1. **Revoke AWS Access**
   - Follow the [AWS Manual Containment SOP](../../sops/aws-user-containment-sop.md) to revoke the AWS access keys involved
   - Notify the user or team responsible for the access key and verify if they were aware of the actions
   - Expected outcome: Actor can no longer perform unauthorized AWS actions

2. **Re-enable Versioning**
   - Re-enable versioning on the affected S3 bucket to prevent further data loss
   - Document the current state of the bucket before making changes
   - Expected outcome: S3 bucket is protected against object deletions

## Eradication

### Eradication Strategy

Ensure that any unauthorized access is completely removed and determine if any data was compromised.

### Eradication Steps

1. **Investigate Additional Activity**
   - Review all recent actions by the same user across AWS and other platforms
   - Check for any additional resources that may have been affected
   - Expected outcome: Complete understanding of the scope of unauthorized activity

2. **Rotate Compromised Credentials**
   - If the activity was malicious, rotate all associated credentials
   - Review IAM permissions to identify and remediate any overly permissive policies
   - Expected outcome: No lingering access for unauthorized actors

## Recovery

### Recovery Strategy

Restore proper configuration and recover any deleted data if necessary.

### Recovery Steps

1. **Restore Versioning Configuration**
   - Ensure versioning is re-enabled on all affected S3 buckets
   - Verify the configuration change was successful
   - Expected outcome: S3 bucket versioning is properly configured

2. **Recover Deleted Data**
   - If objects were deleted, attempt to recover them from backups or alternative sources
   - Work with resource owners to validate data integrity
   - Expected outcome: All critical data is restored

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to recovery
- Number of objects affected
- Number of buckets affected

### Detection Improvement

- Consider implementing preventative controls for critical S3 buckets (e.g., SCPs to prevent versioning disablement)
- Improve monitoring of S3 configuration changes, especially when followed by delete operations

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Incident Response Guide](https://docs.aws.amazon.com/whitepapers/latest/aws-security-incident-response-guide/aws-security-incident-response-guide.html)
- [Okta SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2428896150/Okta+SOPs)

## Related Artifacts

- [AWS S3 Protection Guidelines](../../artifact-guidelines/aws-s3-protection.md)
- [Data Protection Policies](../../artifact-guidelines/data-protection-policies.md)