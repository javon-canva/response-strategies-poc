# AWS GuardDuty Unusual Network Port Communication Incident Playbook

## Overview

**Incident Type:** Suspicious Network Activity  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [AWS GuardDuty](https://console.aws.amazon.com/guardduty/)
- [GuardDuty Finding Type Documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#behavior-ec2-networkportunusual)
- [Previous Response Example](https://pew-pew.canva-internal.com/caseview/95b4dd9b-66b1-4a32-99d3-c9d2c1380b51)

## Response Strategy

### Assumptions

- Incident affects Canva-related infrastructure (not subsidiaries like Serif)
- Responder has access to [infra-tools](https://docs.canva.tech/infrastructure/tools/infra-tools/) and they are up-to-date
- Responder has access to Elastic search and AWS accounts

### Considerations

- Some outbound communication to unusual ports might be legitimate for specific application requirements
- Communication to GitHub over port 22 is common for certain teams like Observability using Flux to pull from repositories
- The sensitivity of the affected AWS account and resources should factor into response urgency

## Triage

### Initial Assessment

1. Gather basic information from the alert:
   - Identify the AWS account ID and name using `infra account <CLOUD_ACCOUNT_ID>` if not provided directly
   - Note the account "flavor" (account name) and `canva.contact` for the owner
   - Identify the destination IP address/hostname (`destination.ip` or `destination.address`) and organization (`destination.as.organization.name`)
   - Record the EC2 instance ID (`cloud.instance.id`)
   - Note the timestamp of the activity (`signal.original_time`)

2. Search for related authentication and activity:
   - Query the [logs-identity-broker.audit](https://d-r-primary.kb.us-east-1.aws.found.io:9243/app/discover) index for role assumptions around the time of the alert
   - Query the [logs-aws.cloudtrail](https://d-r-primary.kb.us-east-1.aws.found.io:9243/app/discover) index for related instance activity

3. Engage with the account owners if clarification is needed:
   - Post a query in the owning team's Slack channel for context about the observed activity

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Communication to known malicious destination; Communication from production/sensitive account; Unusual port associated with exploitation (e.g., 4444, 8088); Multiple instances showing same pattern | 
| Medium   | Communication to unfamiliar but not confirmed malicious destination; Development/testing account traffic; Common but unexpected ports (e.g., 22, 80, 443) |
| Low      | Confirmed legitimate traffic that triggered anomaly detection due to infrequent usage patterns |

### Validation Criteria

- Communication to a non-expected or non-authorized destination IP address
- Unexpected communication from an AWS account where no outbound traffic should occur
- For suspected false positives involving GitHub on port 22:
  - Check if the IP falls within [GitHub's published IP ranges](https://api.github.com/meta)
  - Use WHOIS services (e.g., [Netify.ai](https://www.netify.ai/resources), [Virustotal](https://www.virustotal.com/)) to verify IP ownership
  - Validate if the instance belongs to teams that use Flux for GitHub repository pulls

## Investigation

### Investigation Pattern

Follow the [EC2 Suspicious Network Activity Investigation Pattern](../../investigation-patterns/ec2-network-activity-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [EC2 Instance Network Analysis](../../data-source-procedures/ec2-network-analysis.md)

### Investigation Steps

1. **Analyze the Network Communication Pattern**
   - Determine the port and protocol used for the unusual communication
   - Identify the frequency and volume of traffic to the destination
   - Check if other instances in the same account are communicating with the same destination
   - Review any available VPC Flow Logs for the instance

2. **Research the Destination**
   - Perform IP reputation checks using security tools
   - Use OSINT to research the destination IP and organization
   - For GitHub-related traffic, validate against GitHub's IP ranges
   - Determine if the destination is associated with known malicious activity

3. **Examine the Instance**
   - Review the instance's purpose and expected behavior
   - Check for recent changes or deployments to the instance
   - Examine available system logs for unusual processes or connections
   - Look for signs of compromise such as unexpected users, processes, or files

## Containment

### Containment Strategy

If suspicious activity is confirmed, isolate the instance to prevent further communication while maintaining forensic evidence.

### Containment Steps

1. **Snapshot the Instance**
   - Create a snapshot of the EC2 instance for forensic analysis
   - Use Wiz or other forensic tools to capture the state
   - Document the snapshot ID and process
   - Expected outcome: Evidence preserved for investigation

2. **Restrict Network Communications**
   - Create a security group with restrictive outbound rules
   - Apply the security group to the affected instance
   - Consider adding an inbound rule to completely isolate the instance if necessary
   - Expected outcome: Suspicious traffic is blocked while allowing investigation

3. **Notify Stakeholders**
   - Page the [account owners on-call roster](https://canva.app.opsgenie.com/schedule/whoIsOnCall)
   - Create an incident channel and invite relevant stakeholders
   - Expected outcome: Appropriate teams are engaged in the response

## Eradication

### Eradication Strategy

Identify and remove any malicious code or unauthorized access methods from the affected environment.

### Eradication Steps

1. **Identify Compromise Indicators**
   - Work with Network Engineers to examine activity within the instance
   - Review syslog and other available logs
   - Search for persistence mechanisms such as cron jobs, startup scripts, or unauthorized users
   - Expected outcome: All compromise indicators are identified

2. **Identify Vulnerabilities**
   - Search for vulnerabilities that might have contributed to the breach
   - Involve Application Security team as required
   - Document findings for remediation
   - Expected outcome: Root cause of the compromise is understood

## Recovery

### Recovery Strategy

Replace or restore affected resources to ensure a clean environment.

### Recovery Steps

1. **Deploy Clean Resources**
   - Scale up a new instance or egress proxy as required
   - Ensure the new instance has proper security controls in place
   - Work with the owning team to restore functionality
   - Expected outcome: Clean replacement resources are operating properly

2. **Validate Security Controls**
   - Confirm that all vulnerabilities have been addressed
   - Verify that no malicious elements persist in the environment
   - Review security groups and network configurations
   - Expected outcome: Environment is secure against similar threats

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to eradication
- Number of affected instances
- Duration of unauthorized communication

### Detection Improvement

- Review GuardDuty configuration for potential enhancements
- Consider implementing additional network monitoring
- Evaluate the need for more restrictive outbound security group rules

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [AWS - Remediating a potentially compromised EC2 instance](https://docs.aws.amazon.com/guardduty/latest/ug/compromised-ec2.html)

## Related Artifacts

- [EC2 Forensic Analysis Guidelines](../../artifact-guidelines/ec2-forensic-analysis.md)
- [Network Security Configuration Standards](../../artifact-guidelines/network-security-standards.md)