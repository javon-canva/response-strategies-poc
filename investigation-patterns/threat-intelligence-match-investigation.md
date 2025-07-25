# Threat Intelligence Match Investigation Pattern

## Overview

**Pattern Type:** Threat Intelligence Analysis  
**Applicable Incident Types:** IoC Match, Malware Detection, External Threat Actor Activity, Data Exfiltration  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Pattern Description

This investigation pattern provides a structured approach for investigating matches against threat intelligence indicators. It applies when security tools identify activity that matches known malicious indicators, helping analysts determine if the match represents a true security incident, assess its impact, and identify the full scope of the threat.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [Threat Intelligence Platforms](../../data-source-procedures/threat-intelligence-platform-analysis.md)
   - [Network Traffic Analysis](../../data-source-procedures/network-traffic-analysis.md)
   - [Endpoint Activity Analysis](../../data-source-procedures/endpoint-activity-analysis.md)
   - [DNS/Web Proxy Logs](../../data-source-procedures/dns-web-proxy-analysis.md)

2. **Verify Data Availability**
   - Ensure logs cover at least 7 days prior to the IoC match
   - Verify that endpoint logs are available for the affected system
   - Confirm access to threat intelligence platforms
   - Check that DNS and web proxy logs are properly indexed

3. **Prepare Investigation Environment**
   - Set up access to Anomali, Recorded Future, and other intelligence platforms
   - Create a secure workspace for documenting findings
   - Configure investigation timeframes in SIEM tools
   - Prepare timeline visualization tools if needed

## Investigation Methodology

### Phase 1: Indicator Validation

**Purpose:** Validate the matched indicator's reputation and determine its current threat status

**Steps:**
1. Look up the indicator in primary threat intelligence platforms:
   ```
   # Example Anomali Search
   Search in https://ui.threatstream.com/search for the IoC value
   
   # Example Recorded Future Search
   Navigate to https://sandbox.recordedfuture.com/dashboard and search for the IoC
   ```

2. Check additional reputation services:
   - For domains/URLs: Use VirusTotal, URLScan.io
   - For file hashes: Use VirusTotal, Hybrid-Analysis
   - For IP addresses: Use AbuseIPDB, GreyNoise

3. Determine the IoC's age, confidence level, and attribution:
   - Note when the IoC was first identified
   - Record the confidence score
   - Document associated threat actors or campaigns

**Expected Outcomes:**
- Confirmation that the IoC is still considered malicious
- Understanding of the IoC's historical context and threat level
- Knowledge of associated threat actors or malware families

**Pivot Points:**
- If the IoC is no longer considered malicious, document reasoning and focus on false positive analysis
- If the IoC is part of a broader campaign, expand investigation to look for related indicators
- If the IoC is high-confidence, prioritize immediate containment steps

### Phase 2: Context Analysis

**Purpose:** Understand the full context of how the IoC was matched and what actions preceded/followed it

**Steps:**
1. Create a timeline of events around the IoC match:
   ```
   # Example Elastic Search Query
   source.ip:<affected_ip> OR destination.ip:<affected_ip> OR host.name:<affected_hostname>
   AND @timestamp:[now-24h TO now]
   ```

2. For network-based IoCs, analyze the full session:
   ```
   # Example DNS Query
   event.dataset:dns AND dns.question.name:<suspicious_domain>
   
   # Example HTTP Query
   url.domain:<suspicious_domain> OR destination.ip:<suspicious_ip>
   ```

3. For endpoint-based IoCs, examine process execution chain:
   ```
   # Example EDR Query
   process.hash_md5:<hash_value> OR process.hash_sha256:<hash_value>
   ```

4. Map out user activities leading to the IoC match:
   - Review user authentication events
   - Examine email activity if applicable
   - Check file download/upload operations

**Expected Outcomes:**
- Clear understanding of what led to the IoC match
- Identification of the initial infection vector or trigger
- Timeline of events surrounding the match

**Pivot Points:**
- If the IoC match was preceded by suspicious email, pivot to email investigation
- If web browsing led to the match, expand to user browsing history
- If multiple systems are involved, expand scope to lateral movement investigation

### Phase 3: Impact Assessment

**Purpose:** Determine the potential and actual impact of the matched threat

**Steps:**
1. Assess whether payload delivery occurred:
   ```
   # Example File Creation Query
   event.category:file AND event.type:creation AND host.name:<affected_hostname>
   AND @timestamp:[<match_time-1h> TO <match_time+1h>]
   ```

2. Look for evidence of execution:
   ```
   # Example Process Execution Query
   event.category:process AND event.type:start AND host.name:<affected_hostname>
   AND @timestamp:[<match_time-1h> TO <match_time+1h>]
   ```

3. Check for data exfiltration indicators:
   ```
   # Example Network Traffic Query
   source.bytes:[1000000 TO *] AND source.ip:<internal_ip> AND destination.ip:<external_ip>
   ```

4. Examine for persistence mechanisms:
   ```
   # Example Registry or Startup Modification Query
   event.category:registry AND registry.path:*Run* AND host.name:<affected_hostname>
   ```

**Expected Outcomes:**
- Determination if active compromise occurred
- Assessment of data access or exfiltration
- Identification of any persistence mechanisms
- Understanding of the full scope of impact

**Pivot Points:**
- If persistence mechanisms are found, pivot to malware analysis
- If data exfiltration is detected, pivot to data loss investigation
- If lateral movement is identified, expand investigation to additional systems

## Data Correlation Techniques

### Technique 1: IoC Expansion

**Purpose:** Expand from the matched IoC to identify related indicators and activity

**Implementation:**
1. Extract MITRE ATT&CK techniques associated with the IoC
2. Search for additional IoCs from the same threat actor or campaign
3. Use intelligence reports to identify the full attack pattern
4. Search for related IoCs across the environment

**Example:**
```
# Search for related domains using WHOIS data
WHOIS registrant information from matched domain:<registrant_email>

# Search for related IP infrastructure
Network connections from matched IP range:<CIDR_block>
```

### Technique 2: Temporal Correlation

**Purpose:** Identify suspicious activity by correlating events in time proximity to the IoC match

**Implementation:**
1. Create a baseline of normal activity for the affected user/system
2. Look for deviations from normal patterns around the IoC match time
3. Identify time windows when multiple security events occurred
4. Look for evidence of deliberate action to coincide with low-monitoring periods

**Example:**
```
# Create baseline activity for system
host.name:<affected_hostname> AND @timestamp:[now-30d TO now-7d]

# Look for anomalies during the match period
host.name:<affected_hostname> AND @timestamp:[<match_time-6h> TO <match_time+6h>]
```

## Investigation Artifacts

### Required Artifacts

- **IoC Verification Report**: Documentation of IoC validation from threat intelligence platforms
- **Event Timeline**: Chronological sequence of events related to the IoC match
- **System State Information**: Snapshot of system configuration and running processes at detection time
- **Network Traffic Analysis**: Summary of relevant network communications related to the IoC

### Optional Artifacts

- **PCAP Files**: Packet captures of suspicious network traffic if available
- **Memory Dumps**: Memory forensics data if deeper analysis is required
- **Related IoC List**: Additional IoCs discovered during the investigation
- **User Interview Notes**: Documentation from discussions with affected users

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| Legitimate connection to recently compromised site | IoC is very recent; User has history of legitimate business with the domain | Check historical access patterns to the site; Confirm site was recently compromised; Verify no unusual activity following access |
| Security testing or red team activity | Activity originates from security team IP ranges; Testing notification may exist | Check security testing calendar; Confirm with security team; Look for test documentation |
| Shared IP infrastructure | IoC match on IP address only; Legitimate service using shared hosting | Check for specific URI paths; Analyze full HTTP requests; Verify legitimate business relationship with services on shared IP |

## Investigation Completion Criteria

### Required Findings

- Confirmation of IoC validity and current threat status
- Determination of initial access vector or trigger for the match
- Assessment of whether execution or successful compromise occurred
- Documentation of affected systems and accounts
- Evaluation of data access or exfiltration

### Required Documentation

- Summary of the threat intelligence context (actor, campaign, malware family)
- Timeline of events from initial access to detection
- List of all affected systems and accounts
- Recommended containment and remediation actions
- Assessment of the threat's business impact

## Related SOPs

- [User Account Containment SOP](../../sops/user-account-containment-sop.md)
- [Malware Response SOP](../../sops/malware-response-sop.md)
- [Evidence Collection SOP](../../sops/evidence-collection-sop.md)

## Additional Resources

- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Threat Intelligence Sharing Standards](https://oasis-open.github.io/cti-documentation/)
- [Guide to Malware Incident Prevention and Handling](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-83r1.pdf)