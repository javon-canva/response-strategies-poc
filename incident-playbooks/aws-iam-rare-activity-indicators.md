# AWS IAM Rare Activity Indicators Incident Playbook

## Overview

**Incident Type:** Suspicious Authentication/Access  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [AWS CloudTrail](https://console.aws.amazon.com/cloudtrail/)
- [Detection Strategy](https://github.com/Canva/detection-strategies/canva/aws_rare_source-country-asn-ua-activity-against-role.md)
- [Example Finding](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/AlS8z)

## Response Strategy

### Assumptions

- Familiarity with AWS IAM roles and CloudTrail logs
- Access to Elastic search and AWS console
- Understanding of Machine Learning anomaly detection principles
- Access to ML anomaly detection result indices in Elastic

### Considerations

- Browser or operating system updates can change user-agent strings and generate false positives
- GeoIP records aren't always perfectly accurate, especially in regions with multiple countries in close proximity
- Canvanauts may work overseas in different countries from their usual location
- IAM roles are sometimes accessed by Canvanauts in numerous teams
- False positive alerts may occur depending on where and how the role has been previously used

## Triage

### Initial Assessment

1. Access the underlying ML detection index:
   - Select `Index Management` in Elastic
   - Enable `Include hidden indices` option
   - Enter `.ml-anomalies-custom-aws_rare-activity-ml-jobs` index
   - Click on the index and select `Discover index`

2. Query the ML index for relevant anomalies:
   ```
   job_id: aws_rare-user-asn-ua-action-against-iam-role
   AND result_type: record
   AND record_score >= 50
   AND (by_field_name: (event.action OR source.geo.country_name OR source.as.organization.name OR user_agent.original))
   AND user.name: <user-from-alert>
   AND "@timestamp" >= "now-1h/h"
   ```

3. Examine key fields in the results:
   - `@timestamp`: When the anomalous activity occurred
   - `by_field_name`: What attribute triggered the anomaly (country, ASN, user-agent, action)
   - `by_field_value`: The specific value that was flagged as anomalous
   - `record_score`: The anomaly score (higher indicates more unusual)
   - `user.name`: The IAM role or user involved

4. Check for correlated activity:
   - Search for access to other systems (Okta, GitHub) from the same source IP/ASN/geolocation
   - Review Jira tickets or GitHub PRs that could correlate to the Canvanaut's activity
   - Contact the Canvanaut directly to verify the activity

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Multiple rare indicators (country, ASN, user-agent) simultaneously; High-value IAM role accessed; Suspicious actions performed; Unable to contact user to verify; Activity during unusual hours | 
| Medium   | Single rare indicator with sensitive role; Activity that cannot be immediately verified; Known role misuse patterns; Connection from sanctioned or high-risk country |
| Low      | Single rare indicator with low-sensitivity role; Verifiable legitimate travel; Recently changed corporate device causing user-agent change |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Access to multiple systems (AWS, Okta, GitHub) from the same unusual IP/location
- Unable to contact the Canvanaut to verify activity
- Activity is suspicious in nature with potential security impact
- Actions performed are inconsistent with the user's regular duties
- Multiple rare indicators observed simultaneously (rare country AND rare user-agent AND rare action)

## Investigation

### Investigation Pattern

Follow the [AWS Authentication Analysis Investigation Pattern](../../investigation-patterns/aws-authentication-analysis.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [Identity Broker Log Analysis](../../data-source-procedures/identity-broker-log-analysis.md)
- [IP Geolocation Analysis](../../data-source-procedures/ip-geolocation-analysis.md)

### Investigation Steps

1. **Analyze Authentication Context**
   - Examine the full authentication chain leading to the IAM role assumption
   - Review identity broker logs to understand how access was granted
   - Analyze the source IP address and geolocation information
   - Check user agent details for indications of unusual tools or automation
   - Expected outcome: Complete understanding of how the role was accessed

2. **Review Historical Access Patterns**
   - Compare the detected access with historical patterns for the same user and role
   - Identify how this access deviates from normal patterns
   - Check for any previous alerts or suspicious activity
   - Examine recent changes to the user's access patterns or work location
   - Expected outcome: Determination of how unusual this access truly is

3. **Examine Actions Performed**
   - Create a timeline of all actions taken using the role
   - Categorize actions by risk level and potential impact
   - Look for attempts to escalate privileges, modify security controls, or access sensitive data
   - Check for evidence of data exfiltration or resource manipulation
   - Expected outcome: Assessment of actual or attempted impact

## Containment

### Containment Strategy

If suspicious activity is confirmed, rapidly contain by restricting the compromised account's access while preserving evidence.

### Containment Steps

1. **Suspend User Access**
   - Suspend the user's [Okta access via Cydarm](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm)
   - Document the containment actions and timestamp
   - Expected outcome: User can no longer access company systems

2. **Revoke IAM Credentials**
   - Revoke the IAM role's [temporary security credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_revoke-sessions.html)
   - Use [AWS BootKicker](https://github.com/Canva/infrastructure/blob/master/go/security/incident-response/bootkick/aws/cmd/bootkick.sh) or follow [manual containment steps](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2986777003/Manual+User+Containment#AWS)
   - Document all affected roles and policies
   - Expected outcome: Active IAM sessions are terminated

3. **Notify Stakeholders**
   - Create an incident channel in Slack
   - Notify the team that owns the role
   - Alert relevant security teams
   - Expected outcome: All stakeholders are informed and coordinated

## Eradication

### Eradication Strategy

Identify and remove any persisting threats, address vulnerabilities, and prevent similar incidents.

### Eradication Steps

1. **Investigate Compromise Scope**
   - Work with the team that owns the role to determine all affected resources
   - Examine AWS CloudTrail logs for all actions taken by the role
   - Check for creation of backdoor access (new IAM users, roles, or keys)
   - Expected outcome: Complete understanding of the compromise scope

2. **Revert Unauthorized Changes**
   - Identify and revert any unauthorized changes made using the role
   - Remove any backdoor access mechanisms
   - Document all actions taken and their effect
   - Expected outcome: All unauthorized changes are reverted

3. **Address Security Vulnerabilities**
   - Identify how the unauthorized access occurred
   - Implement changes to prevent similar incidents
   - Update IAM policies or role trust relationships as needed
   - Expected outcome: Security vulnerabilities are addressed

## Recovery

### Recovery Strategy

Restore access to legitimate users and ensure proper security controls are in place.

### Recovery Steps

1. **Restore User Access**
   - Once the issue has been remediated, restore Okta access to the Canvanaut
   - Follow [uncontainment procedures](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker#Un-contain-user-procedure)
   - Verify that the user can access required systems
   - Expected outcome: Legitimate user access is restored

2. **Validate IAM Role Access**
   - Ensure that access to the IAM role is working as expected
   - Test access using appropriate credentials
   - Verify that security controls are functioning properly
   - Expected outcome: IAM role access is properly configured

3. **Implement Additional Monitoring**
   - Set up enhanced monitoring for the affected roles
   - Create custom alerts for sensitive actions
   - Document new monitoring controls
   - Expected outcome: Improved detection capabilities for similar events

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to eradication
- Number of affected AWS resources
- Number of unauthorized actions performed
- False positive rate for ML detection rule

### Detection Improvement

- Review and adjust ML anomaly detection thresholds
- Document legitimate rare access patterns
- Implement additional detection rules for specific high-risk actions
- Enhance correlation between authentication systems

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2428961864/AWS+SOPs)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [AWS Security Documentation](https://docs.aws.amazon.com/security/)

## Related Artifacts

- [AWS IAM Security Configuration Guidelines](../../artifact-guidelines/aws-iam-security.md)
- [Authentication Anomaly Investigation Checklist](../../artifact-guidelines/authentication-anomaly-checklist.md)
- [ML Detection Fine-tuning Procedures](../../artifact-guidelines/ml-detection-tuning.md)