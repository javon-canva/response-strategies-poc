# SpyCloud Investigation Procedure

## Overview

**Data Source:** SpyCloud Credential Monitoring  
**Environment:** SpyCloud Portal  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Data Source Description

SpyCloud provides visibility into leaked credentials and compromised devices related to the organization's domains. It helps security teams identify and investigate potential credential breaches, allowing for prompt remediation before credentials can be used in attacks.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Console/UI | [SpyCloud Portal](https://portal.spycloud.com) | SSO via Okta | Primary access method |

### Required Permissions

- SpyCloud access (request via [onboarding process](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/1684280970/Onboarding#Spycloud))
- Sufficient permissions to view credential records and device information

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| SpyCloud Portal | Records section | Varies by breach | All monitored domains |
| SpyCloud Portal | Compromised Devices section | Varies | Info stealer logs |
| Google Sheets | [Leaked Credential Tracker](https://docs.google.com/spreadsheets/d/1iazwv-c_vwXWTpCbR9lJUN4hMr3CPYjKu4yDiBWNDlA/edit?gid=0#gid=0) | Permanent | Previously triaged credentials |

## Common Investigation Scenarios

### Scenario 1: Investigating Leaked Corporate Credentials

#### Query Examples

Navigate to [All records](https://portal.spycloud.com/watchlist/records/view/all) in the SpyCloud portal and filter by:
- Record Type: "Corporate Records"
- Username/Email: Specific username or domain
- Date Range: Recent timeframe for current investigations

**Fields of interest:**
- `Identifier`: The system/email that the credential is associated with
- `Breach Title`: The source of the credential leak
- `Sighting`: Number of times this credential combination has been seen before
- `Password`: The leaked password (viewable only within SpyCloud)

#### Interpretation Guidelines

- New sightings (Sighting = 0) require immediate investigation
- Higher sighting numbers may indicate previously known/triaged credentials
- Cross-reference with the Leaked Credential Tracker to avoid duplicate work
- Check breach title and date to understand the context of the leak

### Scenario 2: Analyzing Compromised Devices

#### Query Examples

Navigate to [Compromised Devices](https://portal.spycloud.com/compass/compromised-devices) and search by:
- Username
- Email domain
- IP address
- Date range

**Fields of interest:**
- `Target URL`: The service the credential was used for
- `Machine Details`: Operating system, browser, country
- `IP Address`: The IP address of the compromised device
- `Raw Data`: Complete info stealer logs

#### Interpretation Guidelines

- Multiple credentials from a single device often indicate infostealer malware
- Check all credentials captured to identify additional business services at risk
- Identify non-SSO services that might have been compromised
- Machine details can help identify the specific user's device

### Scenario 3: Monitoring New Credential Leaks

#### Query Examples

Filter the Records section by:
- Date Range: Last 7 days
- Record Type: All types
- Sort by: Newest first

**Fields of interest:**
- All fields mentioned in previous scenarios
- `Record Count`: Number of records in a specific breach

#### Interpretation Guidelines

- Large numbers of new records could indicate a new breach
- Check if credentials appear across multiple breaches
- Priority should be given to corporate credentials with admin access
- Monitor for patterns suggesting targeted attacks

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Endpoint Security | Username, Device | Match compromised devices to endpoint security alerts |
| Identity Logs | Username, IP | Correlate with authentication logs to detect usage of leaked credentials |
| Threat Intelligence | Breach Title | Understand context and scope of breaches |

### Common Correlation Techniques

1. **Username-based Correlation**
   - Description: Link SpyCloud records with authentication logs
   - Example approach: Search authentication logs for successful logins with leaked credentials
   - Expected outcomes: Identify unauthorized access using leaked credentials

2. **Device Information Correlation**
   - Description: Match device information from SpyCloud with endpoint inventory
   - Example approach: Use OS, browser, and IP details to identify specific devices
   - Expected outcomes: Locate compromised devices for remediation

## Data Collection for Investigations

### Evidence Collection Process

1. Document all relevant SpyCloud records in the Leaked Credential Tracker
2. Capture screenshots of relevant records (excluding passwords)
3. Document all potentially impacted services and accounts
4. Link SpyCloud records to related security incidents

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| UI Export | Use export button in Records view | CSV | Do NOT export passwords |
| Screenshot | Browser screenshot tool | Image | Redact sensitive information |

## Known Limitations

- Only domains configured in watchlists will be actively monitored
- Non-watchlisted domains will have usernames redacted and passwords unavailable
- False positives can occur from credential stuffing attempts and password reuse
- Historical data may be incomplete depending on when watchlists were configured

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing credentials | Domain not on watchlist | Contact #d-r-cti-focus to add domain to watchlist |
| Incomplete device info | Limited infostealer log data | Use available data to narrow investigation scope |
| Access issues | Permission problems | Request access through proper channels |

## Additional Resources

- [Secret Leakage Playbooks](../../incident-playbooks/generic-leaked-secrets.md)
- [Credential Theft Investigation Pattern](../../investigation-patterns/credential-theft-investigation.md)
- [SpyCloud Documentation](https://portal.spycloud.com/documentation)

## Related SOPs

- [Credential Breach Response SOP](../../sops/credential-breach-response.md)
- [User Account Containment SOP](../../sops/user-account-containment-sop.md)