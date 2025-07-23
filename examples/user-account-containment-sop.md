# User Account Containment SOP

## Overview

**Purpose:** This SOP provides standardized procedures for containing potentially compromised user accounts to limit unauthorized access while preserving evidence.  
**SOP Type:** Technical  
**Version:** 1.0  
**Last Updated:** 2025-07-23  
**Maintainer:** Detection and Response Team

## Scope

**Applies to:**
- All corporate identity provider platforms (Okta, Azure AD, Google Workspace)
- Standard and privileged user accounts
- Human and service accounts
- Incident types involving potential account compromise

**Related Documents:**
- [Suspicious Authentication Activity Incident Playbook](malicious-login-playbook.md)
- [Post-Incident Account Recovery SOP](../sops/sop-template.md)
- [Evidence Collection SOP](../sops/sop-template.md)

## Prerequisites

### Required Permissions

- Identity provider administrative access
- Security monitoring platform access
- Incident management system access

### Required Tools

| Tool | Purpose | Access Information |
|------|---------|-------------------|
| Identity Provider Admin Console | Account suspension and session management | Admin portal access required |
| SIEM Platform | Log verification and monitoring | Security team access |
| Incident Management System | Documentation and tracking | D&R team access |

### Assumptions

- The incident has been validated as a potential compromise
- Basic user information has been collected (username, department, role)
- Authorization for containment has been obtained from incident commander
- Evidence preservation steps have already been initiated

### Considerations

- Business impact of containment actions on user productivity
- Timing considerations for critical business operations
- Potential for tipping off attackers if containment is partial
- Communication needs with affected user and management

## Procedure

### Step 1: Prepare for containment

1. Document the current state of the account before making changes
2. Capture current sessions, access tokens, and recently accessed resources
3. Notify the incident commander that containment is beginning
4. Prepare communication templates for the affected user and their manager

**Expected Outcome:** Complete documentation of pre-containment account state and readiness for containment actions.

**Verification Method:** Review documentation to ensure all required pre-containment information is captured.

### Step 2: Implement account suspension

1. Log in to the appropriate identity provider admin console
2. Locate the user account using the corporate email address
3. Select the suspension/disablement option appropriate to the platform
4. Choose "Security Incident" as the reason for suspension
5. Confirm the action to suspend the account

**Expected Outcome:** User account is suspended and can no longer authenticate to corporate resources.

**Verification Method:** Attempt to access the account or verify status in admin console shows as suspended.

### Step 3: Terminate active sessions

1. Navigate to the session management section in the identity provider
2. Identify all active sessions for the suspended user
3. Select "Terminate all sessions" or equivalent option
4. Verify all sessions show as terminated

**Expected Outcome:** All active sessions are terminated, forcing logout across all applications and devices.

**Verification Method:** Session status shows as terminated in admin console, and monitoring confirms no active sessions remain.

### Step 4: Secure associated accounts and access

1. Identify linked accounts and delegated access rights
2. Suspend or restrict access to connected applications and services
3. Review service account permissions granted to the user
4. Document all additional containment actions taken

**Expected Outcome:** All related access paths are contained to prevent lateral movement.

**Verification Method:** Verify all identified linked accounts and delegated access have been properly secured.

## Special Cases and Exceptions

### Exception 1: Executive or VIP accounts

[Description of special handling required for executive accounts]

**Handling Instructions:**
1. Obtain specific approval from CISO or delegate before containment
2. Coordinate timing with executive's assistant or manager
3. Offer expedited alternative access methods if required

### Exception 2: Critical service accounts

[Description of special handling required for service accounts]

**Handling Instructions:**
1. Assess business impact before containment
2. Prepare alternate service continuity plan if needed
3. Implement additional monitoring if immediate containment is not possible

## Troubleshooting

| Problem | Possible Cause | Solution |
|---------|---------------|----------|
| Unable to suspend account | Insufficient permissions | Escalate to identity platform administrator or follow emergency access procedure |
| Active sessions remain after termination | Cached credentials or offline access | Force device check-in or implement network blocks as secondary containment |
| Business critical function affected | Unknown dependency on contained account | Implement exception process and additional monitoring if containment must be delayed |

## Approval and Notification Requirements

| Scenario | Approval Required From | Notification Required To |
|---------|------------------------|--------------------------|
| Standard user containment | Incident Commander | User's manager, Security Operations |
| Privileged account containment | Incident Commander and CISO | User's manager, IT Operations, Executive team |
| Service account containment | Incident Commander and Service Owner | IT Operations, Application Owner, On-call support |

## Documentation Requirements

All containment actions must be documented in the incident management system with timestamps and specific actions taken.

**Required Documentation Fields:**
- User identity and role
- Timestamp of containment actions
- Specific containment measures implemented
- Observed account activity prior to containment
- Business impact assessment
- Notifications sent and responses received

## Additional Resources

- [Identity Provider Admin Guide](https://example.com/identity-admin)
- [Account Compromise Investigation Checklist](../investigation-patterns/investigation-pattern-template.md)
- [User Communication Templates](../artifact-guidelines/artifact-guideline-template.md)