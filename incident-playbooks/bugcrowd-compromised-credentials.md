# Bugcrowd Compromised Credentials Incident Playbook

## Overview

**Incident Type:** Credential Compromise  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Bugcrowd Bug Bounty Program](https://bugcrowd.com/engagements/canva)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/deb423cf-b45b-4a0f-bd8e-0dcb1f8c3166)
- [Spycloud](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2878345483/Spycloud)

## Response Strategy

### Assumptions

- Access to internal security tools including Spycloud, Elastic, Okta Admin, and SentinelOne
- Understanding of credential validation processes
- Access to Bugcrowd reporting platform

### Considerations

- This playbook covers compromised Canvanaut credentials only, not service accounts or customer credentials
- Canva has a bug bounty program with Bugcrowd that includes reporting of discovered credentials
- Cases are usually automatically created in Cydarm when reports are correctly labeled as containing staff credentials
- The source of compromise (vulnerability vs. malware infection) determines the response approach
- Previous incidents may have already addressed the compromised credentials

## Triage

### Initial Assessment

1. Assess the source of the compromised credentials:
   - Check if the compromise is due to a product vulnerability
   - Determine if it's from device compromise (e.g., infostealer malware)
   - Review details provided in the Bugcrowd report
   - Identify the affected email address and domain (e.g., @canva.com, @concentrix.com, @flourish.studio)

2. Check if the credential combinations have been previously reported:
   - Search for the affected email address in [Cydarm](https://pew-pew.canva-internal.com)
   - Review previous cases regarding the same user
   - If credentials were previously rotated, mark as false positive

3. Search in [Spycloud](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2878345483/Spycloud):
   - Check if the data has already been ingested
   - Use Spycloud Compass to identify other potentially affected credentials
   - Review breach details to understand scope and source

4. Validate the credentials:
   - Test if the password combinations still work
   - Check if the credentials allow access to the MFA prompt or enable successful sign-in
   - If reported as part of a user data dump, validate a sample of 15-20 credential pairs
   - Determine which service(s) the credentials are valid for

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Valid credentials for privileged account; Evidence of active exploitation; Widespread credential compromise affecting multiple users; Credentials with access to sensitive systems or data; Compromise from targeted attack | 
| Medium   | Valid credentials requiring MFA for complete access; Limited number of affected users; Credentials from non-targeted breach; Credentials with limited access scope |
| Low      | Expired or already rotated credentials; Credentials with no valid access; Historical breach with no evidence of recent exploitation |

### Validation Criteria

Escalate to an incident if any of the following are true:
- Valid credentials are confirmed and not previously known/rotated
- Credentials belong to a Canvanaut or third-party owned by Canva
- There is evidence of attempted or successful unauthorized access
- Multiple users are affected by the same compromise source
- The compromise is due to active malware or targeted attack

## Investigation

### Investigation Pattern

Follow the [Credential Compromise Investigation Pattern](../../investigation-patterns/credential-compromise-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [Spycloud Investigation](../../data-source-procedures/spycloud-investigation.md)
- [User Authentication Activity Analysis](../../data-source-procedures/user-authentication-analysis.md)

### Investigation Steps

1. **Determine User Access Scope**
   - Follow the [Determining User Access SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2129887330/Determining+User+Access)
   - Review Okta logs from the suspected date of compromise
   - Identify potentially affected applications and services
   - Check for unusual authentication patterns or locations
   - Expected outcome: Complete understanding of potentially compromised access

2. **Identify Device Involvement**
   - Determine if company or personal device was involved in the compromise
   - For company devices, check SentinelOne for the device information
   - Use filters such as "Visible IP," "Last logged in user," and "Endpoint name"
   - Search logs in Elastic using the source IP to correlate with device information
   - Expected outcome: Identification of potentially compromised devices

3. **Assess Breach Scope and Impact**
   - Review details in Spycloud to understand the breach source
   - Check for other potentially affected users from the same breach
   - Determine if other credentials were stored on the compromised device
   - Assess potential access to sensitive data or systems
   - Expected outcome: Complete understanding of the breach scope and impact

## Containment

### Containment Strategy

Rapidly limit access to prevent unauthorized use of compromised credentials while preserving evidence for investigation.

### Containment Steps

1. **Lock User Accounts**
   - Use Bootkicker to revoke access to identified systems
   - Follow [these instructions](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker)
   - Clear all active sessions
   - For apps not behind Okta, contact app owners using [OneTrust Systems Catalog](https://hub.canva.world/space/CANVAKB/2796422264)
   - Expected outcome: Compromised credentials can no longer be used

2. **Isolate Affected Devices**
   - If a company device was compromised:
     - Disconnect it from the network using SentinelOne
     - Use "Actions > Response > Disconnect from Network" in the SentinelOne console
     - Refer to the [Compromised Canvanaut Devices SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/1563132774/Compromised+Canvanaut+Devices#Containment)
   - If a personal device was compromised:
     - Ask the user to disconnect from corporate networks
     - Request they run a malware scan on the device
   - Expected outcome: Potentially infected devices are isolated

3. **Notify Stakeholders**
   - Contact the user's coach to inform them of the security incident
   - Find the coach information on the user's [Canva World](https://canva.world) profile
   - If their coach is on leave, contact the coach's coach
   - Add the current Appsec `@cop` from [#security-appsec-team](https://canva.enterprise.slack.com/archives/C02JR12DZ1D) to the incident channel
   - Expected outcome: All stakeholders are informed and coordinated

## Eradication

### Eradication Strategy

Identify and remove the source of the compromise, eliminating any remaining malware or unauthorized access.

### Eradication Steps

1. **Remove Malware Infection**
   - For company devices:
     - Engage Tech Support to remove the malware or wipe the device
     - Consider the malware family and its persistence mechanisms
     - Follow the [SOP: Remotely wipe a network contained device](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3204678750/SOP+Remotely+wipe+a+network+contained+device) if necessary
   - For personal devices:
     - Guide the user to use their own AV solution (e.g., MalwareBytes)
     - Recommend device wiping for more persistent malware families
   - Expected outcome: Malware is removed from affected devices

2. **Clean Up Stored Credentials**
   - Remove saved credentials from browsers/keychains (not 1Password)
   - Reference the [Compromised Chrome Profile SOP](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3225257287/Compromised+Chrome+Profiles)
   - Check for credentials in personal Chrome profiles on corporate machines
   - Verify credentials are not stored on any other devices or profiles
   - Expected outcome: All instances of compromised credentials are removed

3. **Address the Source of Compromise**
   - If due to a vulnerability, coordinate with Appsec to ensure it's fixed
   - If due to malware, identify and mitigate infection vector
   - If due to a breach, ensure affected services are notified
   - Expected outcome: Root cause is addressed to prevent recurrence

## Recovery

### Recovery Strategy

Restore secure access for affected users and ensure the environment is protected against similar compromises.

### Recovery Steps

1. **Reset User Credentials**
   - Coordinate with IT to reset the user's password over Zoom
   - Run the un-contain form in Cydarm to reinstate Okta access
   - Follow [these instructions](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2833940486/Contain+user+via+Cydarm+Bootkicker#Un-contain-user-procedure)
   - Ensure the user sets a strong, unique password
   - Expected outcome: User regains secure access with new credentials

2. **Verify System Access**
   - Confirm that the user can access required systems
   - Check that MFA is properly configured and working
   - Ensure any application-specific credentials are also reset
   - Expected outcome: User has appropriate, secure access to all necessary systems

3. **Provide Security Guidance**
   - Educate the user about secure credential management
   - Recommend using 1Password for all credential storage
   - Advise on avoiding password reuse across services
   - Expected outcome: User understands how to prevent future credential compromise

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection from credential exposure
- Time to containment after report
- Number of affected systems or applications
- Whether unauthorized access occurred
- Source of credential compromise

### Detection Improvement

- Review and enhance credential monitoring capabilities
- Update Spycloud monitoring and alerting
- Evaluate additional credential exposure monitoring services
- Improve automated response to Bugcrowd reports

## Additional Resources

- [Bugcrowd Staff Credentials Cases](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3571516262/WIP+-+Bugcrowd+Staff+Credentials+Cases)
- [Appsec Playbook for Triaging Popped/Leaked Credentials](https://canvadev.atlassian.net/wiki/spaces/APPSEC/pages/2962884190/Triaging+Popped+Leaked+Credentials)
- [Monitoring for compromised third-party credentials](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2708865695/Monitoring+for+compromised+third-party+credentials)
- [Using Castle for Investigations](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3033106310/Using+Castle+for+Investigations)

## Related Artifacts

- [Credential Compromise Indicators Reference](../../artifact-guidelines/credential-compromise-indicators.md)
- [Credential Security Standards](../../artifact-guidelines/credential-security-standards.md)