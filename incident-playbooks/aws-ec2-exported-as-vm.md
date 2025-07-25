# AWS EC2 Exported as VM Incident Playbook

## Overview

**Incident Type:** Data Exfiltration Risk  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/aws_ec2_exported_as_vm.md)

## Response Strategy

### Assumptions

- Familiarity with AWS & CloudTrail logs
- Access to Canva's internal security tools
- Understanding of EC2 VM Export functionality and its potential security implications

### Considerations

- VM Export capability allows extraction of entire EC2 instances, including the complete operating system, applications, and data
- Exports could contain sensitive data, credentials, or proprietary information
- Legitimate use cases exist for VM Export, particularly for migration, backup, and development purposes
- The severity of the incident depends on the sensitivity of the exported instance and the intent of the export

## Triage

### Initial Assessment

1. Check if the export was successful by examining the `event.outcome` field
   - Note that unsuccessful exports may lower the severity as you're investigating an attempt rather than a successful export

2. Gather information about the export using the [Kibana search template](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
   - From `aws.cloudtrail.request_parameters`, determine:
     - ID of the EC2 instance that was exported
     - Target S3 bucket and folder
     - Output format (image filetype, hypervisor compatibility)
   - Note `cloud.account.name` and `cloud.region` to identify the account and region
   - Check `cloud.account.sensitivity` to assess potential impact

3. Establish the actor's identity
   - Determine the `user.id` that performed the action
   - Identify what `user_agent.name` was used
   - Note the AWS access key used for the operation

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Export from production or sensitive environment; Instance contains sensitive data; Export to public bucket or bucket outside normal operational boundaries; Unauthorized or suspicious actor; Evidence of attempted concealment | 
| Medium   | Export from non-production but secure environment; Instance with internal but not highly sensitive data; Export by authorized user outside normal process |
| Low      | Export from development/test environment by authorized user; Instance known not to contain sensitive data; Export part of documented process |

### Validation Criteria

- Look up the EC2 instance ID tags in the AWS console to determine:
  - Instance purpose and content
  - Owning team and contact information (Slack channel)
  - Sensitivity classification of the data

- Search for other related activity:
  - Review other AWS actions performed with the same access key using the [AWS activity search](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
  - Check user activity across all logs with noisy activity excluded
  - Verify if the exported VM was downloaded by [reviewing S3 object logs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3182036647/Query+S3+Lambda+log+files)

- Conduct a context search for the user:
  - Check [GitHub PRs](https://github.com/pulls?q=is%3Apr+author%3A%3Cusername%3E+user%3ACanva+sort%3Aupdated-desc+)
  - Use [Confluence Person search](https://canvadev.atlassian.net/wiki/people/search?q=) for relevant Jira tickets and docs
  - Search Slack for the user's activity: `from:@<username> before:<yyyy-mm-dd> after:<yyyy-mm-dd>`

## Investigation

### Investigation Pattern

Follow the [AWS Data Exfiltration Investigation Pattern](../../investigation-patterns/aws-data-exfiltration-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [S3 Bucket Activity Analysis](../../data-source-procedures/s3-bucket-activity-analysis.md)
- [EC2 Instance Investigation](../../data-source-procedures/ec2-instance-investigation.md)

### Investigation Steps

1. **Analyze the EC2 Instance**
   - Determine the purpose and content of the EC2 instance
   - Identify what data/applications were installed on the instance
   - Check instance metadata and tags for ownership information
   - Assess the sensitivity of data that may have been on the instance

2. **Examine the Export Configuration**
   - Review the exact export parameters used
   - Check if the export was targeted to an internal or external S3 bucket
   - Verify the access controls on the target S3 bucket
   - Determine if the export has any special formatting or hypervisor compatibility settings

3. **Analyze User Activity Context**
   - Review the user's authentication events around the time of the export
   - Check for geographic anomalies or unusual access patterns
   - Look for evidence of recent access changes or privilege escalation
   - Determine if the user has previous, legitimate reasons to use VM Export

## Containment

### Containment Strategy

If the export is deemed suspicious or unauthorized, take immediate steps to contain potential data exposure.

### Containment Steps

1. **Secure the Exported VM**
   - Copy VM export to the forensics account for later analysis
   - Restrict access to the S3 bucket where the export is stored
   - Consider deleting the VM export from the original location if unauthorized
   - Expected outcome: The exported VM is secured and cannot be further accessed by unauthorized parties

2. **Contain User Access**
   - If the export appears malicious, follow the [User Account Containment SOP](../../sops/user-account-containment-sop.md)
   - Revoke any active AWS sessions for the user
   - Expected outcome: The actor can no longer perform additional unauthorized actions

3. **Review Bucket Permissions**
   - Check for any recently added ACLs that grant VM Export permissions to the bucket
   - Remove any suspicious bucket policies or ACLs
   - Expected outcome: S3 bucket is secured against unauthorized exports

## Eradication

### Eradication Strategy

Eliminate access paths and permissions that allowed the unauthorized export to occur.

### Eradication Steps

1. **Review IAM Permissions**
   - Identify how the user obtained permissions to perform EC2 exports
   - Remove overly permissive IAM policies that allow VM exports
   - Consider implementing more restrictive S3 bucket policies
   - Expected outcome: IAM permissions are appropriately restricted

2. **Remove Unauthorized Access**
   - If credential compromise is suspected, rotate all affected credentials
   - Review and update IAM roles to enforce least privilege
   - Expected outcome: All unauthorized access is eliminated

## Recovery

### Recovery Strategy

Restore proper security controls and implement measures to prevent future unauthorized exports.

### Recovery Steps

1. **Update Security Configurations**
   - Remove any ACLs that were added to grant VM Export capabilities
   - Implement S3 bucket policies that explicitly deny VM export actions unless specifically authorized
   - Expected outcome: Improved security controls on EC2 instances and S3 buckets

2. **Enhance Monitoring**
   - Implement additional monitoring for VM export actions
   - Set up alerts for S3 bucket policy changes
   - Expected outcome: Improved detection capabilities for future incidents

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Whether the exported VM was accessed outside the organization
- Duration of exposure

### Detection Improvement

- Consider implementing preventative controls against VM exports in sensitive environments
- Evaluate the need for more granular IAM permissions related to VM Export
- Review monitoring capabilities for data exfiltration scenarios

## Additional Resources

- [Exporting an instance as a VM using VM Import/Export](https://docs.aws.amazon.com/vm-import/latest/userguide/vmexport.html)
- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [S3 Access Logging Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ServerLogs.html)

## Related Artifacts

- [AWS VM Export Security Guidelines](../../artifact-guidelines/aws-vm-export-security.md)
- [EC2 Data Protection Standards](../../artifact-guidelines/ec2-data-protection.md)