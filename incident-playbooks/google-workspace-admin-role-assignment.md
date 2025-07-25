# Google Workspace Administrator Role Assignment Incident Playbook

## Overview

**Incident Type:** Privilege Escalation  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Google Workspace Admin Audit Logs](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/RR6gz)
- [Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/google_admin_role_assigned_to_user.md)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/47dc8fe3-7710-4603-9af6-8e454e8d30b8)

## Response Strategy

### Assumptions

- Familiarity with Google Workspace logs
- Access to Canva's internal security tools
- Understanding of Google Workspace admin roles and permissions

### Considerations

- Google Workspace audit logs may be delivered up to several days after the action occurred
- Always check the `original_time` field in the alert payload to determine when the action actually took place
- Users with sufficient privileges to perform admin role assignments are typically named `PERSON-superadmin@domain.com`
- Legitimate admin role assignments are usually preceded by notification in security channels

## Triage

### Initial Assessment

1. Extract key information from the alert payload:
   - `admin.role.name`: The admin role to which a user was added
   - `admin.user.email`: The email address that was added as an admin
   - `source.user.email`: The user that performed the action (the "Actor User")
   - `original_time`: When the action actually occurred

2. Check for prior notification or context:
   - Search security Slack channels for mentions of the `admin.user.email` and/or `admin.role.name`
   - Look for related Jira tickets or change requests
   - Search the [Admin role granting GWorkspace superusers dashboard](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/RR6gz) for similar activity

3. Expand the search if no context is found:
   - Check Jira, Confluence, and other documentation sources
   - Be aware that relevant `ITSERVICE` tickets may not be searchable unless you are tagged
   - Look for related messages in `#help-it` channel

4. Contact the Actor User if necessary:
   - If no justification is found or provided justification is insufficient
   - Request a Zoom video call or out-of-band communication for visual ID
   - Confirm the activity performed aligns with their expected duties
   - If unreachable, contact their coach for verification

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Assignment of Super Admin role; Multiple admin role assignments in short timeframe; Actor User not authorized for role assignments; Assignment during non-business hours; Assignment for non-employee accounts | 
| Medium   | Assignment of limited admin role without prior notification; Assignment by authorized actor but without following proper procedure; Assignment for legitimate purpose but with improper documentation |
| Low      | Assignment with proper documentation and approval; Assignment as part of standard onboarding process with notification |

### Validation Criteria

Escalate to an incident if any of the following are true:
- No justification is found for the admin role assignment
- The Actor User or their coach cannot be reached
- The Actor User or coach does not recognize the actions performed
- There are indications the Actor User's account may be compromised
- The assigned role grants excessive privileges not required for the user's job function

## Investigation

### Investigation Pattern

Follow the [Google Workspace Access Control Investigation Pattern](../../investigation-patterns/google-workspace-access-control-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Google Workspace Audit Logs Investigation](../../data-source-procedures/google-workspace-audit-investigation.md)
- [User Authentication Activity Analysis](../../data-source-procedures/user-authentication-analysis.md)

### Investigation Steps

1. **Analyze Admin Role Assignment Context**
   - Review the complete audit log entry for the role assignment
   - Check for other administrative actions by the same Actor User
   - Examine the timing and circumstances of the assignment
   - Determine if any other accounts were granted admin privileges around the same time
   - Expected outcome: Complete understanding of the context of the role assignment

2. **Review Historical Access Patterns**
   - Examine previous role assignments by the Actor User
   - Check for previous admin role assignments to the target user
   - Look for patterns in legitimate administrative activities
   - Compare with current assignment to identify anomalies
   - Expected outcome: Determination if this activity aligns with normal patterns

3. **Examine Actions Taken by the New Admin**
   - Analyze all actions taken by the new admin user since receiving privileges
   - Look for suspicious activities like creating other admin accounts
   - Check for changes to security settings or user permissions
   - Determine if any sensitive data was accessed
   - Expected outcome: Assessment of any actions taken with the new privileges

## Containment

### Containment Strategy

If suspicious activity is confirmed, rapidly contain by removing elevated privileges and restricting access to potentially compromised accounts.

### Containment Steps

1. **Suspend User Access**
   - [Bootkick](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker) both the `admin.user.email` account AND the `source.user.email`
   - For non-canva.com Google Workspaces, page [ProdSys](https://canva.app.opsgenie.com/schedule/whoIsOnCall) (Schedule name `ProdSys_schedule`)
   - Request that ProdSys disable or suspend the affected accounts
   - Expected outcome: Potentially compromised accounts are prevented from further access

2. **Document Current Role Assignments**
   - Look up the `admin.user.email` in the [Elastic google_workspace index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
   - Document all current role assignments for the affected users
   - Note the original permissions before the suspicious change
   - Expected outcome: Complete record of permission states for later restoration

3. **Notify Stakeholders**
   - Create an incident channel and invite relevant team members
   - Notify the Google Workspace administrators
   - Alert the security team about the potential compromise
   - Expected outcome: All stakeholders are informed and coordinated

## Eradication

### Eradication Strategy

Remove unauthorized access and address any vulnerabilities that may have been exploited.

### Eradication Steps

1. **Remove Unauthorized Admin Access**
   - Page [ProdSys](https://canva.app.opsgenie.com/schedule/whoIsOnCall) (Schedule name `ProdSys_schedule`)
   - Request that the changes for `admin.user.email` are rolled back to the previous role/access level
   - Verify that admin privileges have been properly removed
   - Expected outcome: Unauthorized privileges are removed

2. **Identify Compromise Vector**
   - Determine how the Actor User account may have been compromised
   - Check for potential malware on endpoints
   - Review for credential leaks or misuse of assets
   - Expected outcome: Root cause of the compromise is identified

3. **Review Other Admin Assignments**
   - Search for other unusual admin role assignments
   - Verify the legitimacy of all recent admin role changes
   - Remove any other suspicious admin privileges
   - Expected outcome: All unauthorized admin privileges are identified and removed

## Recovery

### Recovery Strategy

Restore proper access controls and ensure the integrity of the Google Workspace environment.

### Recovery Steps

1. **Investigate Actions Taken by Compromised Accounts**
   - Search the [Elastic google_workspace index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
   - Review all actions performed by both `admin.user.email` and `source.user.email`
   - Pivot to specific application logs if necessary
   - Expected outcome: All potentially malicious actions are identified

2. **Restore Legitimate Access**
   - Return affected users to their proper access levels
   - Re-enable accounts after confirming they are secure
   - Require password changes for affected accounts
   - Expected outcome: Legitimate users regain appropriate access

3. **Document Lessons Learned**
   - Record the incident details and resolution steps
   - Identify process improvements to prevent recurrence
   - Update detection strategies if needed
   - Expected outcome: Improved processes and detection capabilities

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection of unauthorized admin assignment
- Time to containment
- Time to eradication
- Number of actions performed by unauthorized admin
- Whether sensitive data was accessed

### Detection Improvement

- Review Google Workspace admin role assignment alerting
- Consider implementing additional monitoring for specific high-privilege roles
- Evaluate process for notifying security teams of planned admin role changes
- Improve correlation between Google Workspace logs and other authentication sources

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [Google Workspace Admin Guide](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2435809569/Google+Admin)
- [Google Admin Roles - Custom Roles](https://canvadev.atlassian.net/wiki/spaces/IT/pages/2428437190/Google+Admin+Roles+-+Custom+Roles)

## Related Artifacts

- [Google Workspace Security Configuration Guidelines](../../artifact-guidelines/google-workspace-security-configuration.md)
- [Admin Access Review Procedures](../../artifact-guidelines/admin-access-review.md)