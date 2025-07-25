# AWS Data Exfiltration Investigation Pattern

## Overview

**Pattern Type:** Data Exfiltration Analysis  
**Applicable Incident Types:** EC2 VM Export, S3 Bucket Exfiltration, Unusual Data Transfer, Suspicious API Usage  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Pattern Description

This investigation pattern provides a structured approach for investigating potential data exfiltration from AWS environments. It applies when there are indicators of data being moved outside of authorized boundaries, whether through built-in AWS mechanisms like VM Export, unusual network traffic patterns, suspicious API usage, or other methods that could result in sensitive data being removed from controlled environments.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [AWS CloudTrail Logs](../../data-source-procedures/aws-cloudtrail-investigation.md)
   - [S3 Access Logs](../../data-source-procedures/s3-bucket-activity-analysis.md)
   - [VPC Flow Logs](../../data-source-procedures/vpc-flow-logs-analysis.md)
   - [AWS Identity Broker Logs](../../data-source-procedures/aws-identity-broker-logs.md)

2. **Verify Data Availability**
   - Ensure CloudTrail logs cover at least 30 days prior to the suspected exfiltration
   - Confirm that S3 access logging is enabled for relevant buckets
   - Verify VPC Flow Logs are enabled for the relevant VPCs/subnets
   - Check that Identity Broker logs are available to trace role assumptions

3. **Prepare Investigation Environment**
   - Set up access to required AWS accounts and regions
   - Ensure proper permissions for viewing logs and metadata
   - Prepare investigation timeline tools and documentation templates
   - Set up secure storage for any evidence collected

## Investigation Methodology

### Phase 1: Identify Scope and Context

**Purpose:** Establish what data may have been exfiltrated, by whom, and using what mechanisms

**Steps:**
1. Determine the exact AWS resources involved:
   ```
   # Example CloudTrail query for S3 GetObject operations
   event.provider: "s3.amazonaws.com" AND
   event.action: "GetObject" AND
   aws.s3.bucket.name: "<bucket_name>" AND
   @timestamp: [<start_time> TO <end_time>]
   ```

2. Identify the actors involved:
   ```
   # Example Identity Broker query
   identity_broker.audit.observer.info.credentials.aws.role : "<role_name>" AND
   @timestamp: [<start_time> TO <end_time>]
   ```

3. Establish a baseline of normal activity:
   ```
   # Example normal activity baseline query
   user.name: "<username>" AND
   @timestamp: [now-30d TO now-1d]
   ```

4. Map out the timeline of relevant events:
   - Note authentication events
   - Record resource access patterns
   - Document data transfer operations
   - Identify any preceding reconnaissance activity

**Expected Outcomes:**
- Clear understanding of the potentially affected data
- Identification of all actors involved
- Timeline of events leading to and including the suspected exfiltration
- Baseline of normal behavior to identify anomalies

**Pivot Points:**
- If multiple actors are involved, consider expanding to an account compromise investigation
- If the activity appears to be part of a larger campaign, pivot to a broader threat actor investigation
- If the data access appears to be part of normal operations, focus on determining if proper controls were bypassed

### Phase 2: Data Movement Analysis

**Purpose:** Analyze how data was moved and where it was transferred to

**Steps:**
1. For S3-based exfiltration, examine access patterns:
   ```
   # Example S3 access log query
   aws.s3.bucket.name: "<bucket_name>" AND
   aws.s3.object.key: "<sensitive_prefix>*" AND
   source.bytes: [1000000 TO *] AND
   @timestamp: [<start_time> TO <end_time>]
   ```

2. For VM Export or snapshot-based exfiltration:
   ```
   # Example CloudTrail query for VM Export or snapshot actions
   event.provider: "ec2.amazonaws.com" AND
   (event.action: "CreateInstanceExportTask" OR 
    event.action: "CreateSnapshot" OR 
    event.action: "CopySnapshot") AND
   @timestamp: [<start_time> TO <end_time>]
   ```

3. For network-based exfiltration, analyze VPC Flow Logs:
   ```
   # Example VPC Flow Log query for large outbound transfers
   source.ip: "<internal_ip_range>" AND
   destination.ip: [NOT "<internal_ip_range>"] AND
   source.bytes: [10000000 TO *] AND
   @timestamp: [<start_time> TO <end_time>]
   ```

4. Analyze the destination of data transfers:
   - Check for unknown or suspicious S3 buckets
   - Look for cross-region or cross-account transfers
   - Identify external IP addresses receiving large data volumes
   - Evaluate whether destinations are authorized data processing locations

**Expected Outcomes:**
- Identification of the specific data exfiltration mechanisms used
- Volume of data potentially exfiltrated
- Destination(s) of the exfiltrated data
- Assessment of whether transfers bypassed security controls

**Pivot Points:**
- If data was moved to another AWS account, expand investigation to that account
- If network-based exfiltration occurred, pivot to external endpoint investigation
- If the exfiltration used credentials or permissions that shouldn't exist, expand to privilege escalation investigation

### Phase 3: Impact Assessment

**Purpose:** Determine the sensitivity and impact of the data potentially exposed

**Steps:**
1. Identify the nature of affected resources:
   - For EC2 instances: Review instance tags, purpose, and data classification
   - For S3 objects: Check object tags, path conventions, and known sensitive prefixes
   - For databases: Determine table contents and data classification

2. Assess the sensitivity of exposed data:
   ```
   # Example sensitive data classification query
   aws.s3.object.key: (*pii* OR *confidential* OR *secret* OR *credential* OR *password*) AND
   aws.s3.bucket.name: "<bucket_name>" AND
   @timestamp: [<start_time> TO <end_time>]
   ```

3. Determine potential regulatory implications:
   - Check for personal data subject to privacy regulations
   - Identify financial data subject to compliance requirements
   - Note intellectual property or trade secrets

4. Evaluate business impact:
   - Assess operational impact from data exposure
   - Determine competitive disadvantages from exposed data
   - Identify potential reputation damage scenarios

**Expected Outcomes:**
- Classification of the sensitivity of exposed data
- Assessment of regulatory compliance implications
- Evaluation of business impact
- Determination of notification requirements

**Pivot Points:**
- If highly sensitive data was exposed, pivot to breach notification procedures
- If the exposure might trigger regulatory requirements, engage legal and compliance teams
- If minimal or no sensitive data was exposed, focus on control improvements

## Data Correlation Techniques

### Technique 1: IAM Activity Correlation

**Purpose:** Connect data access events with authentication and authorization activities to identify the complete chain of access

**Implementation:**
1. Start with the data access events (e.g., S3 GetObject, VM Export)
2. Trace back to the IAM principal that performed the action
3. Identify any role assumptions or credential issuance events prior to the access
4. Correlate with authentication logs to find the original user session

**Example:**
```
# Find the IAM role used
event.action: "CreateInstanceExportTask" AND
aws.cloudtrail.user_identity.type: "AssumedRole"

# Find the role assumption
event.action: "AssumeRole" AND
aws.cloudtrail.request_parameters.roleArn: "*<role_name>*"

# Find the original authentication
event.dataset: "okta.system" AND
user.name: "<username>" AND
@timestamp: [<start_time-2h> TO <start_time>]
```

### Technique 2: Timeline-based Correlation

**Purpose:** Identify suspicious patterns by correlating seemingly unrelated events that occur in sequence

**Implementation:**
1. Create a timeline of all activities from the suspected actor
2. Look for reconnaissance patterns preceding data access
3. Identify permission changes followed by unusual data access
4. Note cleanup activities following data transfers

**Example:**
```
# Create a comprehensive timeline for a suspected actor
user.name: "<username>" OR 
source.ip: "<source_ip>" OR
aws.cloudtrail.user_identity.access_key_id: "<access_key>" AND
@timestamp: [<start_time-12h> TO <end_time+2h>]
```

## Investigation Artifacts

### Required Artifacts

- **Access Timeline**: Chronological record of authentication, authorization, and data access events
- **Data Access Inventory**: Complete list of accessed data objects with sensitivity classification
- **Actor Attribution Report**: Documentation linking the activity to specific users or entities
- **Impact Assessment**: Analysis of the potential impact of the data exfiltration

### Optional Artifacts

- **Network Traffic Analysis**: Detailed analysis of any suspicious network traffic
- **Data Volume Report**: Quantification of the volume of data potentially exfiltrated
- **Permission Analysis**: Review of the permissions that allowed the exfiltration to occur
- **Previous Activity Baseline**: Comparison of the suspicious activity against normal patterns

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| Legitimate data migration | Large data transfers to new AWS account; Migration-related API calls; Prior change management records | Check for change management tickets; Confirm with system owners; Verify recipient account is authorized |
| Development activity | VM Exports to development environments; Test data transfers; Local development setup | Check if actor is a developer; Verify development patterns; Confirm data was not production/sensitive |
| Automated backups | Regular, scheduled data transfers; Consistent destinations; System service accounts | Look for regularity in scheduling; Check automation documentation; Verify backup policies |

## Investigation Completion Criteria

### Required Findings

- Confirmation of whether data exfiltration occurred
- Identification of the specific data that was accessed or transferred
- Attribution to specific actors and determination of intent
- Assessment of the sensitivity and impact of any exposed data
- Determination of any security control failures that enabled the activity

### Required Documentation

- Complete timeline of events from initial access to data transfer
- List of all affected resources and data objects
- Assessment of security control effectiveness
- Recommendations for security improvements
- Determination of notification requirements

## Related SOPs

- [User Account Containment SOP](../../sops/user-account-containment-sop.md)
- [Data Breach Response SOP](../../sops/data-breach-response-sop.md)
- [S3 Bucket Security Review SOP](../../sops/s3-bucket-security-review-sop.md)

## Additional Resources

- [AWS Security Best Practices](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/welcome.html)
- [Data Loss Prevention Strategies](https://d1.awsstatic.com/whitepapers/compliance/AWS_Data_Loss_Prevention_Solutions.pdf)
- [AWS S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html)