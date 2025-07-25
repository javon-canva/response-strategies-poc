# AWS GuardDuty Unusual DNS Resolver Incident Playbook

## Overview

**Incident Type:** Defense Evasion - DNS Configuration Manipulation  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [AWS GuardDuty](https://console.aws.amazon.com/guardduty/)
- [GuardDuty Finding Type Documentation](https://docs.aws.amazon.com/guardduty/latest/ug/guardduty_finding-types-ec2.html#defenseevasion-ec2-unusualdnsresolver)
- [Previous Response Example](https://pew-pew.canva-internal.com/caseview/3a06513a-ce9c-4395-9205-845c31ce87ab)

## Response Strategy

### Assumptions

- Incident affects Canva-related infrastructure (not subsidiaries)
- Responder has access to [infra-tools](https://docs.canva.tech/infrastructure/tools/infra-tools/) and they are up-to-date
- Responder has access to Elastic search and AWS accounts

### Considerations

- This GuardDuty alert commonly triggers in `-cn` AWS accounts, where a common legitimate cause is `wireguard-vpn`
- Find more information about [WireGuard Infra here](https://canvadev.atlassian.net/wiki/spaces/CN/pages/918782060/WireGuard+VPN+-+To+China)
- Most other alerts for this rule originate from egress hosts in `canva-network` (565140988831)
- DNS manipulation could indicate an attempt to bypass security controls or exfiltrate data
- Changes to DNS configurations may indicate compromise

## Triage

### Initial Assessment

1. Gather basic information from the alert:
   - Identify the AWS account ID and name using `infra account <CLOUD_ACCOUNT_ID>` if not provided directly
   - Note the tags under `aws.guardduty.resource.instanceDetails.tags`
   - Check the value of tag `canva.application-id` (a value of `wireguard-vpn` may indicate a false positive)
   - Record the EC2 instance ID (`cloud.instance.id`)
   - Note the timestamp of the activity (`signal.original_time`)

2. Examine DNS history for the instance:
   - Access [Elastic Infra Logs](https://logs-infrastructure.canva-corp.com/_plugin/kibana/app/discover) or [Kibana-CN](https://logs.canva-corp.cn/_plugin/kibana/app/discover) for CN logs
   - Filter by `log_stream: <CLOUD_INSTANCE_ID>`
   - Adjust the time window around `signal.original_time`
   - Compare baseline (prior to `signal.original_time`) with current DNS activity
   - For suspicious domains, check reputation via [VirusTotal](https://www.virustotal.com/), [Anomali](https://ui.threatstream.com/classic-dashboard?type=overview), [UrlScan](https://urlscan.io/)

3. Engage with relevant stakeholders:
   - For CN-related alerts, check with WireGuard VPN team if applicable
   - For other alerts, contact the team identified in the instance tags

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Requests to known C2 or malicious domains; Communication from production/sensitive account; Evidence of data exfiltration via DNS; Multiple instances showing same pattern | 
| Medium   | Unusual DNS resolver configuration in non-production account; Changes to DNS configuration without clear malicious intent; Communication to suspicious but not confirmed malicious domains |
| Low      | Confirmed legitimate DNS configuration for specific use cases (e.g., WireGuard VPN in CN regions); Temporary DNS changes for testing |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Requests to C2 or otherwise malicious domains are identified
- The baseline DNS activity for the instance has changed significantly
- Multiple instances show similar suspicious DNS configuration changes
- DNS traffic patterns suggest data exfiltration attempts

## Investigation

### Investigation Pattern

Follow the [EC2 Suspicious DNS Activity Investigation Pattern](../../investigation-patterns/ec2-dns-activity-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [EC2 Instance Network Analysis](../../data-source-procedures/ec2-network-analysis.md)
- [DNS Traffic Analysis](../../data-source-procedures/dns-traffic-analysis.md)

### Investigation Steps

1. **Analyze DNS Configuration Changes**
   - Examine instance configuration for DNS resolver changes
   - Review CloudTrail logs for API calls related to DNS configuration
   - Check for modification of network interfaces or routing tables
   - Identify who made the changes and when
   - Expected outcome: Timeline of DNS configuration changes

2. **Examine DNS Query Patterns**
   - Analyze DNS query types and frequency
   - Look for unusual domain patterns that could indicate data exfiltration
   - Check for DNS tunneling indicators (high volume of TXT queries, abnormal query lengths)
   - Compare against known legitimate DNS patterns for the instance
   - Expected outcome: Identification of suspicious DNS activity or confirmation of legitimate use

3. **Review Instance Activity**
   - Check for unusual processes or applications running on the instance
   - Review system logs for evidence of unauthorized access
   - Look for recently installed packages or modified configuration files
   - Correlate DNS activity with other network communications
   - Expected outcome: Comprehensive understanding of instance activity context

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
   - Block specific DNS communication paths as needed
   - Consider adding an inbound rule to completely isolate the instance if necessary
   - Expected outcome: Suspicious DNS traffic is blocked while allowing investigation

3. **Notify Stakeholders**
   - Page the [cloud-networking_schedule](https://canva.app.opsgenie.com/schedule/whoIsOnCall) for assistance
   - Create an incident channel and invite relevant stakeholders including network engineers
   - Expected outcome: Appropriate teams are engaged in the response

## Eradication

### Eradication Strategy

Identify and remove any malicious code or unauthorized configurations from the affected environment.

### Eradication Steps

1. **Identify Persistence Mechanisms**
   - Work with Network Engineers to examine activity within the instance
   - Search for persistence mechanisms such as cron jobs, startup scripts, or unauthorized users
   - Check for modified DNS configuration files or settings
   - Expected outcome: All compromise indicators are identified

2. **Identify Vulnerabilities**
   - Search for vulnerabilities that might have contributed to the breach
   - Involve Application Security team as required
   - Document findings for remediation
   - Expected outcome: Root cause of the compromise is understood

3. **Remove Unauthorized Configurations**
   - Reset DNS configuration to known-good state
   - Remove malicious code or unauthorized modifications
   - Patch identified vulnerabilities
   - Expected outcome: All unauthorized changes are eliminated

## Recovery

### Recovery Strategy

Replace or restore affected resources to ensure a clean environment.

### Recovery Steps

1. **Deploy Clean Resources**
   - Scale up a new egress proxy or instance, as required
   - Ensure the new instance has proper DNS configuration and security controls
   - Work with the owning teams to restore functionality
   - Expected outcome: Clean replacement resources are operating properly

2. **Validate Security Controls**
   - Confirm that all vulnerabilities have been addressed
   - Verify that no malicious elements persist in the environment
   - Test DNS resolution to ensure proper configuration
   - Expected outcome: Environment is secure against similar threats

3. **Implement Additional Controls**
   - Consider implementing DNS monitoring for the affected environment
   - Review and tighten security group rules if needed
   - Evaluate the need for additional logging or alerting
   - Expected outcome: Enhanced security posture to prevent recurrence

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to eradication
- Number of affected instances
- Duration of unauthorized DNS configuration
- Whether data exfiltration occurred

### Detection Improvement

- Review GuardDuty configuration for potential enhancements
- Consider implementing additional DNS monitoring
- Evaluate baseline DNS activity for different application types
- Update documentation for legitimate DNS configurations (like WireGuard VPN)

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [AWS - Remediating a potentially compromised EC2 instance](https://docs.aws.amazon.com/guardduty/latest/ug/compromised-ec2.html)
- [WireGuard VPN Documentation](https://canvadev.atlassian.net/wiki/spaces/CN/pages/918782060/WireGuard+VPN+-+To+China)

## Related Artifacts

- [EC2 Forensic Analysis Guidelines](../../artifact-guidelines/ec2-forensic-analysis.md)
- [DNS Security Configuration Standards](../../artifact-guidelines/dns-security-standards.md)
- [Network Monitoring Guidelines](../../artifact-guidelines/network-monitoring-guidelines.md)