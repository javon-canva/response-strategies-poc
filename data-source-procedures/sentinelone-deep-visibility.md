# SentinelOne Deep Visibility Investigation Procedure

## Overview

**Data Source:** SentinelOne Deep Visibility  
**Environment:** Endpoint Security Platform  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Data Source Description

SentinelOne Deep Visibility is an endpoint detection and response (EDR) tool that provides extensive visibility into endpoint activities. It captures and indexes endpoint telemetry data including process execution, file operations, network connections, registry modifications, and user activities. This data source is essential for investigating security incidents, performing threat hunting, and conducting forensic analysis of endpoint activities.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| SentinelOne Console | [Management Console](https://usea1-ms.sentinelone.net/) | SSO via Okta | Primary UI method |
| API | `https://usea1-ms.sentinelone.net/api/v2/` | API Token | For automated queries |
| S1 CLI | `s1-cli query` | API Token | Command-line interface |

### Required Permissions

- At minimum: "Analyst" role in SentinelOne console
- For remediation actions: "SOC Manager" or equivalent role
- For API access: API token with appropriate scopes
- For advanced hunting: "Threat Hunter" role

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| SentinelOne Console | Deep Visibility tab | 14-30 days (configurable) | Primary interface |
| Exported Queries | Downloaded CSV/JSON files | N/A (export as needed) | For offline analysis |
| SIEM Integration | Configured SIEM platform | Depends on SIEM retention | If configured |

## Common Investigation Scenarios

### Scenario 1: Malware Execution Analysis

#### Query Examples

```
// Basic process execution query for suspected malware
EventType = "Process Creation" AND 
ProcessName CONTAINS "suspicious.exe" AND
TimestampUTC >= "2025-07-20T00:00:00Z" AND
TimestampUTC <= "2025-07-22T00:00:00Z"

// Query to find command line with suspicious parameters
EventType = "Process Creation" AND 
CommandLine CONTAINS "powershell" AND
CommandLine CONTAINS "-enc" AND
EndpointName = "AFFECTED-HOST"
```

**Fields of interest:**
- `ProcessName`: Name of the executed process
- `ProcessDisplayName`: Display name of the process
- `CommandLine`: Full command line arguments
- `ParentName`: Name of the parent process
- `ParentCommandLine`: Command line of the parent process
- `SHA1`: Hash of the executed file
- `Username`: User context of execution

#### Interpretation Guidelines

- Look for unexpected parent-child process relationships (e.g., Word spawning PowerShell)
- Check for obfuscated command lines (e.g., base64 encoded parameters)
- Identify processes running from unusual locations (e.g., temp directories)
- Watch for processes with names that mimic system processes but from wrong paths
- Review the full process lineage to understand the execution chain

### Scenario 2: Lateral Movement Investigation

#### Query Examples

```
// Query for suspicious network connections
EventType = "Network" AND 
(IPPort = 445 OR IPPort = 3389 OR IPPort = 5985 OR IPPort = 5986) AND
Direction = "Outbound" AND
EndpointName = "AFFECTED-HOST"

// Query for remote PowerShell execution
EventType = "Process Creation" AND 
CommandLine CONTAINS "wsmprovhost" OR
CommandLine CONTAINS "winrm" OR
CommandLine CONTAINS "wsman"
```

**Fields of interest:**
- `IPAddress`: Destination IP address
- `IPPort`: Destination port number
- `Direction`: Direction of the connection (Inbound/Outbound)
- `ProcessName`: Process establishing the connection
- `DnsRequest`: DNS lookups related to the connection

#### Interpretation Guidelines

- Look for connections to internal systems from compromised hosts
- Check for use of administrative protocols (SMB, WinRM, RDP)
- Identify scanning behavior (multiple connections to different hosts)
- Correlate network connections with process execution events
- Watch for unusual authentication attempts following connections

### Scenario 3: Data Exfiltration Detection

#### Query Examples

```
// Query for suspicious file operations
EventType = "File Modification" AND
(FileName CONTAINS ".zip" OR FileName CONTAINS ".rar" OR FileName CONTAINS ".7z") AND
EndpointName = "AFFECTED-HOST"

// Query for suspicious uploads
EventType = "Network" AND 
(IPPort = 21 OR IPPort = 22 OR IPPort = 443 OR IPPort = 80) AND
(BytesOut > 10000000) AND
Direction = "Outbound"
```

**Fields of interest:**
- `FileName`: Name of the affected file
- `FilePath`: Path of the affected file
- `BytesOut`: Volume of outbound traffic
- `URL`: URL for web requests
- `DNSRequest`: Domain name for network connections

#### Interpretation Guidelines

- Look for creation of archive files containing sensitive data
- Check for large outbound data transfers to unusual destinations
- Monitor for access to sensitive directories followed by network activity
- Watch for data transfers outside business hours
- Identify unauthorized cloud storage tools or file transfer utilities

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Firewall Logs | `IPAddress`, `IPPort` | Correlate endpoint connections with firewall traffic |
| DNS Logs | `DNSRequest` | Link DNS requests to actual domain resolutions |
| Proxy Logs | `URL`, `IPAddress` | Associate web requests with proxy traffic |
| Authentication Logs | `Username`, timestamp | Connect endpoint activity with authentication events |

### Common Correlation Techniques

1. **Process-Network Chain Analysis**
   - Description: Link process executions with subsequent network connections
   - Example query:
   ```
   // Find process creation followed by network connection
   $processes = EventType = "Process Creation" AND 
                CommandLine CONTAINS "powershell" AND
                EndpointName = "AFFECTED-HOST";
   
   $network = EventType = "Network" AND
              Direction = "Outbound" AND
              EndpointName = "AFFECTED-HOST";
              
   $processes | followed by [1m] $network
   ```
   - Expected outcomes: Identify processes initiating suspicious communications

2. **File-Process-Network Sequence Analysis**
   - Description: Track complete attack chain from file creation to network activity
   - Example approach:
     - First, identify file creation/modification events
     - Then, find processes that accessed those files
     - Finally, look for network connections from those processes
   - Expected outcomes: Complete visibility into data exfiltration patterns

## Data Collection for Investigations

### Evidence Collection Process

1. For malware investigation:
   - Create and save a Deep Visibility query focusing on the malicious process
   - Export process tree visualization as PDF
   - Capture screenshots of relevant timelines and relationships
   - Export full event data in JSON format for preservation

2. For lateral movement investigation:
   - Document all affected endpoints
   - Export network connection data with full context
   - Capture authentication events across the timeframe
   - Create timeline visualization of movement between systems

3. For incident response:
   - Collect all evidence before initiating remediation
   - Document automated and manual response actions
   - Export before/after state of affected systems
   - Preserve full query results for incident documentation

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| UI Export | Deep Visibility > Export Results | CSV/JSON | Limited to 100,000 events |
| Process Tree Export | Deep Visibility > Process Tree > Export | PDF/PNG | Visual representation only |
| API Export | `GET /dv/events` | JSON | Useful for large datasets |
| CLI Export | `s1-cli query --output-file events.json` | JSON | For scripted collection |

## Known Limitations

- Data retention is limited (typically 14-30 days) depending on configuration
- Complex queries may time out if returning too many results
- Some process arguments may be truncated for very long command lines
- Cross-site queries require proper site permissions
- Not all file operations are captured (based on filter configurations)
- Network data shows connections but not full packet contents

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Query timeout | Too broad query timeframe or criteria | Narrow query by time or add more specific filters |
| Missing data | Events outside retention period | Check data retention settings; export data more frequently |
| Limited process details | Process monitoring configuration issues | Review and adjust EDR configuration settings |
| "Access denied" errors | Insufficient permissions | Request appropriate role assignment |
| Inconsistent results | Query syntax errors or data indexing delays | Refine query syntax; allow time for data indexing |

## Additional Resources

- [SentinelOne Documentation](https://community.sentinelone.com/s/)
- [Deep Visibility Query Language Guide](https://community.sentinelone.com/s/article/Deep-Visibility-Query-Language)
- [SentinelOne Hunting Query Library](https://community.sentinelone.com/s/article/Threat-Hunting-Guide)
- [SentinelOne API Documentation](https://community.sentinelone.com/s/article/API-Documentation)

## Related SOPs

- [Malware Incident Response SOP](../../sops/malware-incident-response-sop.md)
- [Endpoint Isolation Procedure](../../sops/endpoint-isolation-procedure-sop.md)
- [Evidence Collection Guidelines](../../sops/evidence-collection-guidelines-sop.md)