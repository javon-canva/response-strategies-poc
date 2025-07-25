# Publicly Available Design Incident Playbook

## Overview

**Incident Type:** Data Exposure (Canva Design)  
**Severity Levels:** Low/Medium/High/Critical  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Public Collaboration Link Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/canva_publicly_available_design_collaboration_link.md)
- [Public View Link Detection Strategy](https://github.com/Canva/detection-strategies/blob/master/canva/canva_publicly_available_design_public_view_link.md)
- [Public Collaboration Link Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/security/rules/id/93fd2fd2-69c1-485f-95af-8e780e1a125b/alerts)
- [Public View Link Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/security/rules/id/ed7da9d2-dd42-4992-a3e3-7e5105fdba30/alerts)

## Response Strategy

### Assumptions

- Access to Canva's internal security tools and audit logs
- Administrative access to the CHT portal
- Ability to modify design permissions and access
- Authority to contact design owners
- Understanding of data classification policies

### Considerations

- Initial automated triage occurs via Tines orchestration
- Canvanauts are notified by EVA bot when designs are made public
- Alerts are only sent to manual triage if:
  - Canvanaut indicates uncertainty about why the design is public
  - Canvanaut fails to respond to the notification
- False positives may occur when designs are legitimately made public
- Anonymous access could be related to legitimate Slack bot activity
- Different access methods require different investigation approaches

## Triage

### Initial Assessment

1. Identify the exposed design:
   - Review the designs listed in the case management system
   - Document the design ID and other relevant metadata
   - Access the design using appropriate method:
     - Collaboration links: accessible via Canva Team or Admin account
     - Public view links: only accessible via CHT portal with Admin account
   - Verify current sharing status and history

2. Assess design content sensitivity:
   - Review the content of the design thoroughly
   - Apply the [Data Classification and Handling Standard](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/3162606598/Data+Classification+and+Handling+Standard)
   - Document specific sensitive content if present
   - Classify as public, internal, confidential, or restricted

3. Check for legitimate bot activity:
   - Determine if access could be related to Slack link expansion
   - Check audit logs for characteristic patterns:
     - Use [this search template](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/r/s/XBLvQ)
     - Look for AWS EC2 IP addresses with VIEW_DESIGN & EXPORT activity
     - Use `canva.audit.context.request_id` to correlate with Cloudflare trace ID
     - Verify user agent is `Slackbot-LinkExpanding 1.0 (+https://api.slack.com/robots)`
   - Document findings to rule out this common false positive

4. Contact document owner:
   - Verify if the design was intentionally made public
   - Document their justification if sharing was intentional
   - Assess their understanding of data sharing policies

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | Restricted data exposed publicly; Evidence of external access to highly sensitive information; Intentional data exfiltration; Exposure affects many stakeholders or customers; Regulatory reporting required |
| High     | Confidential data exposed publicly; Limited evidence of external access; Multiple designs affected; Long exposure window; No clear business justification |
| Medium   | Internal data exposed publicly; No evidence of external access; Few designs affected; Short exposure window; Possible legitimate sharing but violates policy |
| Low      | Public or non-sensitive data exposed; Very brief exposure period; No evidence of external access; Legitimate business purpose for sharing |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Design contains confidential or restricted data per classification standard
- External access occurred from unidentified sources
- User claims they did not make the design public
- Access patterns indicate potential account compromise
- Multiple designs made public simultaneously without justification

## Investigation

### Investigation Pattern

Follow the [Data Exposure Investigation Pattern](../../investigation-patterns/data-exposure-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Canva Audit Log Analysis](../../data-source-procedures/canva-audit-log-analysis.md)
- [Authentication Log Analysis](../../data-source-procedures/authentication-log-analysis.md)
- [User Activity Analysis](../../data-source-procedures/user-activity-analysis.md)

### Investigation Steps

1. **Design Access History Analysis**
   - Review complete design access logs using this query:
     ```
     canva.audit.target.id: <design_uid> 
     AND event.action: VIEW_DESIGN 
     AND NOT client.user.email: *canva.com
     ```
   - Identify any external access patterns
   - Determine if unauthenticated views occurred (`user.name = ANONYMOUS`)
   - Document authenticated non-Canvanaut views
   - Expected outcome: Complete understanding of who accessed the design

2. **Permission Change Timeline Analysis**
   - Create timeline of design sharing changes:
     ```
     canva.audit.target.id: <design_id>
     AND event.action: UPDATE_DESIGN_ACCESS_CONTROLS 
     AND canva.audit.action.changes.type: DELETE_DESIGN_ACCESS_TOKEN
     ```
   - Document when and how the design was made public
   - Identify which user made the sharing change
   - Check for suspicious IP addresses or locations
   - Expected outcome: Complete timeline of design permission changes

3. **User Activity Analysis**
   - Review actor's (`canva.audit.actor.user.id`) other recent activities
   - Check for patterns of unusual behavior or multiple public shares
   - Correlate with user authentication logs
   - Look for suspicious geographic locations or device changes
   - Expected outcome: Determination if user activity suggests compromise

4. **Additional Exposed Design Identification**
   - Check if other designs by the same user were made public
   - Look for potential folder-level permission changes
   - Verify if other users in the same team have similar exposures
   - Check if the same sharing pattern appears in other designs
   - Expected outcome: Complete scope of affected designs

## Containment

### Containment Strategy

Quickly restrict access to the exposed design while preserving evidence and minimizing business impact.

### Containment Steps

1. **Preserve Evidence**
   - Take screenshots of the design with sensitive content
   - Document all metadata including sharing settings
   - Record audit logs showing access and permission changes
   - Create a backup copy if further analysis is needed
   - Expected outcome: Complete record of the design and its sharing status

2. **Restrict Design Access**
   - If user is responsive, guide them to make the design private
   - Provide the [internal design sharing guide](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/2934081969/Sharing+and+Publishing+Internal+Canva+Designs)
   - For non-responsive users with sensitive data exposure:
     - Access the [CHT portal](https://www.canva.com/chtportal) with Canva Admin credentials
     - Enter the design ID and access the design
     - Create a backup by clicking "Remix" and noting the case ID
     - Use the Delete function to remove public access
   - Document all containment actions with timestamps
   - Expected outcome: Public access to sensitive design is removed

3. **Account Security Assessment**
   - If suspicious activity is identified, consider account containment
   - For potentially compromised accounts, follow the [user containment procedure](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker)
   - Additionally, clear Canva app sessions using [this procedure](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2986777003/Manual+User+Containment#Canva)
   - Document justification for any account suspension
   - Expected outcome: Prevention of further unauthorized sharing

## Eradication

### Eradication Strategy

Thoroughly review access history to identify any data exposure and address underlying causes of the incident.

### Eradication Steps

1. **External Access Impact Assessment**
   - Review all access logs for external views:
     ```
     canva.audit.target.id: <design_uid> 
     AND event.action: VIEW_DESIGN 
     AND NOT client.user.email: *canva.com
     ```
   - Document unauthenticated views (`user.name = ANONYMOUS`)
   - Identify authenticated non-Canvanaut views
   - Determine if data was viewed by unauthorized parties
   - Expected outcome: Complete understanding of data exposure extent

2. **Personal Data Exposure Assessment**
   - If personal data was exposed to external parties
   - Follow the [Personal Data Incident Response Plan](https://www.canva.com/design/DAEQ56oThvU/hcGWqNaxtu2MfbQ5u9_Gjw/view)
   - Document all exposed personal data elements
   - Prepare for potential regulatory notifications if required
   - Expected outcome: Proper handling of any PII exposure

3. **Root Cause Identification**
   - Determine why the design was made public:
     - Intentional sharing with misunderstood scope
     - Accidental permission change
     - Malicious insider action
     - Compromised account
   - Document findings and contributing factors
   - Address any identified security vulnerabilities
   - Expected outcome: Clear understanding of how exposure occurred

## Recovery

### Recovery Strategy

Ensure proper design access controls are in place moving forward and educate users on secure sharing practices.

### Recovery Steps

1. **User Education**
   - Contact design owner about proper sharing practices
   - Provide guidance on [sharing internal Canva designs](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/2934081969/Sharing+and+Publishing+Internal+Canva+Designs)
   - Explain data classification policy and implications
   - Document the communication and user understanding
   - Expected outcome: Improved user awareness of secure sharing practices

2. **Design Access Restoration**
   - If design was deleted during containment, restore access
   - Share the backup/remixed copy with the original owner
   - Help establish appropriate sharing permissions
   - Verify the design is now properly secured
   - Expected outcome: Restored business functionality with proper controls

3. **Related Design Review**
   - Check other designs owned by the same user
   - Look for similar sharing patterns across the team
   - Review folder permissions if applicable
   - Correct any additional instances of over-sharing
   - Expected outcome: Comprehensive remediation of all related exposures

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time from exposure to detection
- Time from detection to containment
- Duration of public exposure
- Number of unauthorized views (if any)
- Root cause analysis results
- Effectiveness of automated detection and triage

### Detection Improvement

- Review Tines automation effectiveness
- Assess EVA bot response rates and clarity
- Evaluate alert prioritization criteria
- Consider additional monitoring for design sharing
- Improve user education on secure design sharing

## Additional Resources

- [Data Classification and Handling Standard](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/3162606598/Data+Classification+and+Handling+Standard)
- [Sharing and Publishing Internal Canva Designs Guide](https://canvadev.atlassian.net/wiki/spaces/INFOSEC/pages/2934081969/Sharing+and+Publishing+Internal+Canva+Designs)
- [Personal Data Incident Response Plan](https://www.canva.com/design/DAEQ56oThvU/hcGWqNaxtu2MfbQ5u9_Gjw/view)
- [Manual User Containment Procedures](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2986777003/Manual+User+Containment)

## Related Artifacts

- [Design Sharing Standards](../../artifact-guidelines/design-sharing-standards.md)
- [Data Classification Guidelines](../../artifact-guidelines/data-classification-guidelines.md)
- [Design Audit Log Interpretation Guide](../../artifact-guidelines/design-audit-log-interpretation-guide.md)

## Canva Audit Log Reference

### Notable Field Names

| Friendly Name | Logfile Field Name | Description | Example |
|---------------|-------------------|-------------|---------|
| Canva profile ID | `canva.audit.target.owner.user.id` | User ID who performed the action | UAGLoKc2XUg |
| Canva design ID | `canva.audit.target.id` | ID assigned to the design | DAGMtdG8SGY |
| Brand ID | `canva.audit.actor.team.id` | ID of the 'brand' or team | BABOtbk27fE |
| Additional user ID | `canva.audit.action.changes.user.id` | ID of user given access | UAFwtwwFq0A |
| Additional user name | `canva.audit.action.changes.user.display_name` | Name of user given access | John Doe |
| Additional user email | `canva.audit.action.changes.user.email` | Email of user given access | jdoe@canva.com |
| Original read permissions | `canva.audit.action.changes.access.read` | Previous READ access state | TRUE/FALSE/- |
| Original write permissions | `canva.audit.action.changes.access.write` | Previous WRITE access state | TRUE/FALSE/- |
| New read permissions | `canva.audit.action.changes.new_access.read` | New READ permissions | TRUE/FALSE/- |
| New write permissions | `canva.audit.action.changes.new_access.write` | New WRITE permissions | TRUE/FALSE/- |

### Design Permission Change Types

| Event Type | Description |
|------------|-------------|
| `UPDATE_DESIGN_LINK_ACCESS` | Design made public, private or shared with team via link |
| `GRANT_USER_DESIGN_ACCESS` | User granted access to a design |
| `UPDATE_USER_DESIGN_ACCESS` | User's permissions for a design updated |
| `REVOKE_USER_DESIGN_ACCESS` | User's permissions for a design revoked |
| `GRANT_TEAM_DESIGN_ACCESS` | Team granted access to a design |
| `UPDATE_TEAM_DESIGN_ACCESS` | Team's permissions for a design updated |
| `REVOKE_TEAM_DESIGN_ACCESS` | Team's permissions for a design revoked |
| `CREATE_DESIGN_ACCESS_TOKEN` | Public view link of a design created |
| `DELETE_DESIGN_ACCESS_TOKEN` | Public view link revoked |
| `TRASH_DESIGN` | Design moved to trash |
| `DELETE_DESIGN` | Design deleted by user or admin |