# AWS RDS Snapshot Restoration Incident Playbook

## Overview

**Incident Type:** Unauthorized Database Access/Restoration  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [CloudTrail RDS Restoration Events](https://d-r-primary.kb.us-east-1.aws.found.io:9243/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/aws_rds_snapshot_restore.md)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/17799300-8bfe-4008-9a7b-cdc1574601c8)

## Response Strategy

### Assumptions

- Familiarity with AWS, AWS Web console & CloudTrail logs
- An understanding of RDS instances & backups ([AWS Article](https://docs.aws.amazon.com/aws-backup/latest/devguide/point-in-time-recovery.html))
- Access to Canva's internal security tools
- Optional: Access to [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

### Considerations

- RDS restoration at Canva normally requires approval via the PENGY app and an assumed role with appropriate permissions
- Ensure that the [Access Requests process](https://docs.canva.tech/security/entitlements-system/access-service/) is followed as normal
- If deleting databases on production systems, request peer review before executing commands
- Database restoration may be legitimate but performed without following proper procedures

## Triage

### Initial Assessment

1. Determine if the RDS restoration was properly approved:
   - Check for RDS access approvals in [Access Requests Jira](https://canvadev.atlassian.net/jira/servicedesk/projects/ACCESS/issues/ACCESS-107131?jql=project%20%3D%20%22ACCESS%22%20and%20status%20%3D%20Done%20AND%20text%20~%20%22restore%20RDS%22%20ORDER%20BY%20created%20DESC)
   - Search for activity by the username in [Elastic](https://d-r-primary.kb.us-east-1.aws.found.io:9243/app/discover#/view/20f74b00-c668-11ee-b097-976c6ac82df4?_g=())
   - Check for activity with the same role name in [Elastic](https://d-r-primary.kb.us-east-1.aws.found.io:9243/app/discover#/view/20f74b00-c668-11ee-b097-976c6ac82df4?_g=())

2. Follow the [AWS CloudTrail Investigation Procedure](../../data-source-procedures/aws-cloudtrail-investigation.md) to analyze the activity in detail

3. Reach out to the Canvanaut directly to request confirmation that the activity is legitimate

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Restoration of production database with sensitive data; Multiple unauthorized restorations; Evidence of data exfiltration from restored database; Restoration followed by suspicious query patterns | 
| Medium   | Unauthorized restoration of non-production database; Single instance of policy violation without malicious intent; Restoration for legitimate purpose but without following process |
| Low      | Restoration with proper approval but minor process deviation; User has appropriate permissions and legitimate business need |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Activity cannot be confirmed as legitimate via Access Requests approvals or via Slack channel searching
- Canvanaut is not responding to Detection & Response Slack communications
- Canvanaut is performing a large number of restorations and using multiple roles without any approval
- Sensitive data is involved in the restoration

## Investigation

### Investigation Pattern

Follow the [AWS Database Activity Investigation Pattern](../../investigation-patterns/aws-database-activity-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [RDS Activity Analysis](../../data-source-procedures/rds-activity-analysis.md)

### Investigation Steps

1. **Analyze Context of Restoration**
   - Examine the complete sequence of events before and after the snapshot restoration
   - Determine what database was restored, from what snapshot, and to what new instance
   - Check for any data access patterns after the restoration
   - Identify if any data was exported or if unusual queries were run

2. **Review User Activity and Authorization**
   - Check for proper approval through the access request system
   - Examine the user's normal work patterns and role responsibilities
   - Verify if the user has a legitimate business need for the database access
   - Review Slack conversations or tickets related to the work being performed

3. **Assess Database Sensitivity and Impact**
   - Determine the classification of data in the restored database
   - Check if the database contains sensitive customer information, credentials, or other high-value data
   - Evaluate the potential impact if the data was accessed inappropriately
   - Document any regulatory implications of unauthorized access

## Containment

### Containment Strategy

If the restoration is confirmed to be unauthorized, take immediate steps to contain the incident and prevent unauthorized data access.

### Containment Steps

1. **Restrict User Access**
   - Follow [Bootkick SOP](../../sops/user-account-containment-sop.md) to remove access for the Canvanaut involved
   - Document all assumed AWS roles used by the user
   - Record all databases created from snapshots by the user
   - Expected outcome: User can no longer access AWS resources

2. **Notify Database Owners**
   - Inform database owners on Slack to not access the database until further notice
   - Provide context about the situation without disclosing sensitive details
   - Expected outcome: Prevent further activity on the unauthorized database

3. **Isolate Database Instance**
   - If immediate deletion is not possible, restrict network access to the database
   - Modify security groups to limit access to security team IPs only
   - Expected outcome: Database is isolated while investigation continues

## Eradication

### Eradication Strategy

Remove unauthorized database instances while preserving evidence and ensuring recoverability if needed.

### Eradication Steps

1. **Obtain Required Permissions**
   - Request AWS Role Access through the Access Requests system:
     - Form: AWS → AWS Role Access
     - Reason: "Require access to role for deletion of database, as per Detection & Response security incident"
     - Account: Supply the contents of the `user.roles` field from the alert
     - Duration: 1 Day (or as required)

   - Request AWS Delete Access through the Access Requests system:
     - Form: AWS → AWS Delete Access
     - Reason: "Required for deletion of database, as per Detection & Response security incident"
     - AWS Role: Select the role granted from the previous step
     - Duration: 4 Hours

2. **Delete Unauthorized Database Instance**
   - Access the AWS Management Console RDS section
   - Select the unauthorized database instance
   - Create a final snapshot named `[DBInstanceName]-date-deleted-by-dr-snapshot`
   - Do not retain automated backups
   - Complete the deletion process by typing "delete me" when prompted
   - Expected outcome: Unauthorized database instance is removed while preserving a snapshot for investigation

3. **Verify Deletion**
   - Confirm using Elastic search that the database has been deleted
   - Document all actions taken including snapshot names and deletion timestamps
   - Expected outcome: Confirmation that unauthorized database no longer exists

## Recovery

### Recovery Strategy

Restore appropriate access and ensure affected parties understand proper procedures.

### Recovery Steps

1. **Restore User Access**
   - Contact the coach of the boot-kicked Canvanaut
   - Confirm with the coach that the Canvanaut should have their access restored
   - Follow the appropriate process to restore access
   - Expected outcome: Legitimate access is restored

2. **Provide Education and Guidance**
   - Follow up with the Canvanaut in Slack to understand what occurred
   - Provide guidance on proper procedures for database restoration
   - Document any process gaps identified during the incident
   - Expected outcome: Improved understanding of proper procedures

3. **Verify Incident Resolution**
   - Confirm all unauthorized databases have been deleted
   - Verify that proper procedures are now being followed
   - Document any process improvements needed
   - Expected outcome: Incident fully resolved with documentation

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment (database isolation)
- Time to eradication (database deletion)
- Number of unauthorized database restorations
- Duration of unauthorized database access

### Detection Improvement

- Review and enhance RDS snapshot restoration monitoring
- Consider implementing additional controls for database restoration approvals
- Evaluate process for granting temporary elevated privileges

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [AWS RDS Deletion Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html)
- [AWS Permissions FAQs](https://canvadev.atlassian.net/wiki/spaces/CLOUDPLAT/pages/2985296016/AWS+Permissions+Customer+FAQs)

## Related Artifacts

- [RDS Backup and Restoration Guidelines](../../artifact-guidelines/rds-backup-restoration.md)
- [Database Access Review Procedures](../../artifact-guidelines/database-access-review.md)