# GitHub Rare Bot Activity Incident Playbook

## Overview

**Incident Type:** Suspicious GitHub Authentication Activity  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic ML Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/pull/174)
- [Example Finding](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/DtXqk)

## Response Strategy

### Assumptions

- Access to Elastic, GitHub audit logs, and related security tools
- Understanding of GitHub authentication and bot mechanisms
- Familiarity with Canva's GitHub organization structure and bot usage

### Considerations

- This rule will trigger for human users as well, not just bot accounts
- GitHub bots are used for a variety of different purposes across different repositories and GitHub organizations
- GitHub bots can be used by Canvanauts to perform actions that their "normal" user GitHub accounts do not have permission to do
- False-positive alerts may occur when a Canvanaut accesses a GitHub bot to perform rare actions from a rare location or user agent
- The alert might trigger when a bot is used in a new capacity, such as on new infrastructure which hasn't previously been seen before

## Triage

### Initial Assessment

1. Navigate to the Machine Learning interface within Elastic to view the complete details:
   - Quick access: [Elastic Anomaly Explorer](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/ml/explorer?_g=%28ml%3A%28jobIds%3A%21%28github_rare-bot-country-asn-ua-action-against-self%29%29%2CrefreshInterval%3A%28pause%3A%21t%2Cvalue%3A30000%29%2Ctime%3A%28from%3Anow-6h%2Cto%3Anow%2Cts%3A1738919702425%29%29&_a=%28explorer%3A%28mlExplorerFilter%3A%28%29%2CmlExplorerSwimlane%3A%28%29%29%2CmlAnomaliesTable%3A%28pageIndex%3A0%2CpageSize%3A25%2CsortDirection%3Adesc%2CsortField%3Atime%29%2CmlSelectInterval%3A%28display%3A%27Show%20all%27%2Cval%3Asecond%29%29)
   - Alternative manual navigation: `Machine Learning` → `Anomaly Explorer` → filter for job ID: `github_rare-bot-country-asn-ua-action-against-self`

2. Apply necessary filters:
   - Filter for the specific username from the alert
   - Set the appropriate time range
   - Disable auto-refresh to prevent page reloading during analysis

3. Review the Anomalies table to understand what triggered the alert:
   - Sort the table chronologically by time (important as this isn't the default)
   - Examine the "Influencers" section to identify the event action, user agent, source organization (ASN), country, and bot username

4. With this information, continue investigation using the [standard GitHub index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/ImxGE) in Elastic

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Actions performed from high-risk countries; Previously unseen ASN combined with sensitive actions; Multiple suspicious actions in sequence; Bot access from unexpected networks | 
| Medium   | New user agent with known ASN/location; Unusual bot activity that can be explained; Infrequent action with minor changes to user agent |
| Low      | Minor version change in user agent; Known location with rare but expected action; Confirmed legitimate use by Canvanaut |

### Validation Criteria

To determine whether the alert is a false positive:

1. Check the IP address, ASN, and geolocation against other sources such as Okta or AWS logs

2. Evaluate user agent changes:
   - A slight change in user agent (like a Git version bump) may align with Git's release cycle
   - If the IP/ASN or geolocation has not deviated from historical norms, this suggests a false positive

3. Review the associated event action (e.g., `git.clone`) and compare with past activity for the bot

4. For suspected Codespaces activity:
   - Look for the action `repo.download_zip` from Microsoft-owned infrastructure
   - Check the token scope to verify if related to Codespaces
   - Search CloudFlare logs for the target user, expecting to see a DGA-style subdomain for `github.dev`

5. Escalate to an incident if:
   - You observe access to other systems (AWS, Okta) from the same IP address
   - You are unable to contact the team or Canvanaut that manages the bot
   - The actions performed are potentially destructive (e.g., deleting or updating protected branch settings, attempting to change organization settings)

## Investigation

### Investigation Pattern

Follow the [GitHub Authentication Analysis Pattern](../../investigation-patterns/github-authentication-analysis.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [GitHub Audit Log Analysis](../../data-source-procedures/github-audit-log-analysis.md)
- [IP Geolocation Analysis](../../data-source-procedures/ip-geolocation-analysis.md)
- [User Agent Analysis](../../data-source-procedures/user-agent-analysis.md)

### Investigation Steps

1. **Analyze Bot Activity Context**
   - Review the bot's normal usage pattern and authorized users
   - Check the [GitHub bot registry](https://docs.google.com/spreadsheets/d/1jE_Uz5AQ-4tnZQeDeIwSEuLPkYgFG_rga-1Mr59TZ7Q/edit#gid=0) to identify the bot owner
   - Review recent commits, PRs, and other actions by the bot
   - Identify which repositories the bot typically operates in

2. **Examine Authentication Source Details**
   - Analyze the source IP address and associated ASN
   - Check if the IP belongs to a known cloud provider or code hosting service
   - Look for any geographic anomalies in the login location
   - Correlate the login time with the user's expected working hours

3. **Review Actions Performed**
   - Document all actions taken by the bot during the session
   - Assess whether the actions align with the bot's normal purpose
   - Look for any unusual repository access or administrative changes
   - Check for sensitive actions like adding/modifying webhooks or deploy keys

4. **Contact Bot Owners**
   - Reach out to the team or individuals responsible for the bot
   - Verify if they were performing the observed actions
   - Ask about any recent changes to infrastructure or automation

## Containment

### Containment Strategy

If suspicious activity is confirmed, contain the bot account to prevent further unauthorized actions.

### Containment Steps

1. **Suspend the GitHub Account**
   - For bot accounts: Work with the owning team to suspend the bot account
   - For human users: Follow the [User Account Containment SOP](../../sops/user-account-containment-sop.md) to suspend GitHub access
   - Expected outcome: Unauthorized access to GitHub resources is prevented

2. **Revoke Tokens and Credentials**
   - Identify and revoke any OAuth tokens or personal access tokens associated with the bot
   - Reset any credentials stored in CI/CD systems or other integrations
   - Expected outcome: All authentication mechanisms are invalidated

3. **Restrict Repository Access**
   - If specific repositories were targeted, consider temporarily making them private
   - Review and temporarily restrict webhook configurations
   - Expected outcome: Critical repositories are protected from further manipulation

## Eradication

### Eradication Strategy

Identify and remove any malicious changes or persistent access mechanisms.

### Eradication Steps

1. **Review and Revert Changes**
   - Identify all changes made by the bot during the suspicious period
   - Revert any unauthorized commits, branch changes, or setting modifications
   - Check for newly created webhooks, deploy keys, or GitHub Apps
   - Expected outcome: All unauthorized changes are reversed

2. **Scan for Persistent Access**
   - Review organization and repository settings for unexpected changes
   - Check for new or modified GitHub Actions workflows that might contain malicious code
   - Look for any unexpected collaborators added to repositories
   - Expected outcome: Any backdoors or persistent access mechanisms are identified and removed

## Recovery

### Recovery Strategy

Restore normal bot operations with improved security controls.

### Recovery Steps

1. **Re-establish Bot Access**
   - Create new credentials for the bot account
   - Reconfigure the bot with appropriate, least-privilege access
   - Update the bot configuration in all dependent systems
   - Expected outcome: Bot functionality is restored with proper security controls

2. **Update Bot Documentation**
   - Update the [GitHub bot registry](https://docs.google.com/spreadsheets/d/1jE_Uz5AQ-4tnZQeDeIwSEuLPkYgFG_rga-1Mr59TZ7Q/edit#gid=0) with current information
   - Document the approved source locations/ASNs for the bot
   - Document the expected user agent string patterns
   - Expected outcome: Improved documentation to reduce false positives

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to actor identification
- Number of repositories or resources affected
- Duration of unauthorized access

### Detection Improvement

- Review and update the ML job parameters to reduce false positives
- Add specific exclusions for known legitimate bot activities
- Consider implementing additional detection rules for specific high-risk GitHub actions

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [GitHub bot registry](https://docs.google.com/spreadsheets/d/1jE_Uz5AQ-4tnZQeDeIwSEuLPkYgFG_rga-1Mr59TZ7Q/edit#gid=0)
- [GitHub audit logs documentation](https://docs.github.com/en/enterprise-cloud@latest/admin/monitoring-activity-in-your-enterprise/reviewing-audit-logs-for-your-enterprise/audit-log-events-for-your-enterprise)

## Related Artifacts

- [GitHub Bot Security Guidelines](../../artifact-guidelines/github-bot-security.md)
- [GitHub Access Control Standards](../../artifact-guidelines/github-access-control.md)