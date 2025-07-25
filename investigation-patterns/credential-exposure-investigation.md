# Credential Exposure Investigation Pattern

## Overview

**Pattern Type:** Credential Security  
**Applicable Incident Types:** Leaked Secrets, Compromised Credentials, Phishing, Malware Infections  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Pattern Description

This investigation pattern provides a structured approach for analyzing exposed credentials and secrets. It applies to all types of credential exposure incidents including leaked passwords, API keys, tokens, and other secrets. The pattern focuses on determining the validity of exposed credentials, identifying the source of exposure, assessing if unauthorized access occurred, and determining the full scope of impact.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [Spycloud Investigation](../../data-source-procedures/spycloud-investigation.md)
   - [Authentication Log Analysis](../../data-source-procedures/authentication-log-analysis.md)
   - [Endpoint Security Investigation](../../data-source-procedures/endpoint-security-investigation.md)
   - [Secret Scanning Tools](../../data-source-procedures/secret-scanning-tools.md)

2. **Verify Data Availability**
   - Ensure access to authentication logs for relevant systems
   - Confirm access to secret scanning and credential exposure platforms
   - Verify endpoint security data is available if device compromise is suspected
   - Check that breach notification data is current and accessible

3. **Prepare Investigation Environment**
   - Set up secure validation environment if credential testing is required
   - Configure analysis tools for log review and correlation
   - Prepare documentation templates for tracking findings
   - Establish secure communication channel with affected users

## Investigation Methodology

### Phase 1: Credential Validation Analysis

**Purpose:** Determine if exposed credentials are valid and what access they provide

**Steps:**
1. Assess credential type and format:
   - User credentials (username/email and password)
   - API keys or tokens
   - SSH keys
   - Service account credentials
   - Certificate-based authentication material

2. Validate credential status:
   ```
   // Example validation approach for different credential types
   - For user credentials: Check authentication system status (active/disabled)
   - For API keys: Verify if still active in API management system
   - For tokens: Check expiration status and revocation status
   - For SSH keys: Verify against authorized_keys files on relevant systems
   ```

3. Perform controlled validation where necessary:
   - Use a dedicated, isolated system for validation
   - Document your source IP address and user agent
   - Perform minimal access test that doesn't modify data
   - For API keys, use read-only endpoints only
   - Follow appropriate validation SOPs based on credential type

4. Check for previous rotation:
   - Review credential rotation logs
   - Check if the secret was previously identified and rotated
   - Compare exposure date with last rotation date
   - Verify if current credentials match the exposed version

**Expected Outcomes:**
- Confirmation of credential validity
- Assessment of access level provided by credentials
- Determination if credential has been previously rotated
- Documentation of systems and data accessible with the credential

**Pivot Points:**
- If credential is confirmed valid, pivot to exposure analysis
- If credential has been previously rotated, consider closing investigation
- If credential provides high-privilege access, prioritize containment
- If credential provides access to sensitive data, assess for data exposure

### Phase 2: Exposure Source Analysis

**Purpose:** Identify how and when credentials were exposed

**Steps:**
1. Analyze credential exposure context:
   ```
   // Exposure source analysis approach
   - Check breach notification details for exposure method
   - For database dumps, identify breach timeframe and source
   - For code repository leaks, identify commit history and exposure timeline
   - For endpoint compromise, check for infostealer indicators
   ```

2. Examine potential exposure vectors:
   - Check for code commits containing credentials
   - Review communication platforms (Slack, email) for shared credentials
   - Analyze browser data for saved credentials
   - Examine endpoint security logs for credential harvesting malware
   - Check for phishing indicators targeting credential theft

3. Interview affected users:
   - Determine their knowledge of the credential exposure
   - Ask about devices used to access the credentials
   - Inquire about password reuse across services
   - Collect information on recent suspicious activity
   - Verify recent application installations or browser extensions

4. Determine exposure timeline:
   - Establish first known date of exposure
   - Identify when credential was created/last changed
   - Calculate total exposure window duration
   - Document when exposure was first detected

**Expected Outcomes:**
- Identification of root cause of credential exposure
- Complete timeline from credential creation to exposure detection
- Understanding of exposure scope (public, internal, limited)
- Assessment of exposure vector for remediation planning

**Pivot Points:**
- If exposure is due to malware, pivot to endpoint investigation
- If exposure is due to code commits, engage with AppSec team
- If exposure is due to phishing, expand investigation to email security
- If multiple credentials are exposed from same source, widen investigation scope

### Phase 3: Unauthorized Access Analysis

**Purpose:** Determine if exposed credentials were used for unauthorized access

**Steps:**
1. Review authentication logs for the affected credential:
   ```
   // Example authentication log query approach
   - Filter logs for the exposed username/credential identifier
   - Focus on timeframe after exposure date
   - Look for authentication from unusual locations/IP addresses
   - Check for unusual login times or access patterns
   - Compare with the user's normal access baseline
   ```

2. Examine access patterns and anomalies:
   - Identify authentication from unusual geographic locations
   - Look for abnormal time-of-day access patterns
   - Check for unusual user-agent strings
   - Review session duration and activity
   - Compare with known user behavioral patterns

3. Analyze service-specific activity:
   - Review specific application logs related to the credential
   - Check for unusual data access patterns
   - Look for privilege escalation attempts
   - Monitor for configuration changes
   - Examine API call patterns for abnormalities

4. Determine unauthorized actions taken:
   - Identify any data accessed or exfiltrated
   - Check for creation of new accounts or access methods
   - Look for modification of security controls
   - Examine for signs of lateral movement
   - Review any system modifications or deployed resources

**Expected Outcomes:**
- Determination if unauthorized access occurred
- Timeline of any unauthorized access events
- Assessment of actions taken during unauthorized access
- List of affected systems and data
- Documentation of persistence mechanisms if created

**Pivot Points:**
- If unauthorized access is confirmed, escalate incident severity
- If data exfiltration is detected, initiate data breach procedures
- If new accounts were created, expand investigation to those accounts
- If lateral movement occurred, widen scope to additional systems

### Phase 4: Impact and Scope Assessment

**Purpose:** Determine the full extent of the compromise and potential impact

**Steps:**
1. Map credential access scope:
   ```
   // Scope mapping approach
   - Identify all systems the credential can access directly
   - Map secondary systems accessible through connected services
   - Determine data classifications accessible with the credential
   - Review permission boundaries and privilege levels
   ```

2. Identify affected assets:
   - Create inventory of all systems potentially accessed
   - Determine criticality of affected systems
   - Identify sensitive data potentially exposed
   - Map affected business functions and services

3. Assess business impact:
   - Determine operational disruption potential
   - Calculate potential financial impact
   - Identify reputational risks
   - Evaluate regulatory and compliance implications
   - Assess customer impact if applicable

4. Determine response requirements:
   - Identify credential rotation needs
   - Determine systems requiring additional monitoring
   - Assess forensic investigation requirements
   - Evaluate notification requirements (legal, customers)
   - Plan containment and remediation approach

**Expected Outcomes:**
- Complete understanding of potential and actual impact
- Risk assessment for affected systems and data
- Prioritization framework for remediation activities
- Notification requirements determination
- Response strategy recommendations

**Pivot Points:**
- If sensitive data was exposed, initiate data breach procedures
- If critical systems were affected, focus on business continuity
- If minimal impact is confirmed, proceed to targeted remediation
- If compliance implications exist, engage legal and compliance teams

## Data Correlation Techniques

### Technique 1: Credential Exposure Timeline Analysis

**Purpose:** Create a comprehensive timeline from credential creation to detection and response

**Implementation:**
1. Collect timestamp data from all relevant events:
   - Credential creation date
   - Credential modification dates
   - Exposure date (if known)
   - First appearance in breach database
   - Authentication attempts using the credential
   - Detection of exposure
   - Response actions

2. Order events chronologically to visualize the exposure lifecycle:
   ```
   // Example approach for timeline creation
   $credential_creation = service.audit_logs.credential_creation;
   $credential_usage = authentication.logs.filter(credential_id);
   $exposure_detection = security.alerts.filter(credential_id);
   $response_actions = incident.logs.filter(credential_id);
   
   timeline = merge_and_sort_by_timestamp(
     $credential_creation,
     $credential_usage,
     $exposure_detection,
     $response_actions
   )
   ```

3. Identify gaps in the timeline that require additional investigation

**Example:**
```
2025-01-15 10:23:45 - API key created in GCP console by user@example.com
2025-01-20 14:32:10 - API key committed to GitHub repository
2025-01-20 14:45:22 - GitHub secret scanning alert generated
2025-01-22 09:17:33 - First unauthorized API call from external IP 203.0.113.42
2025-01-22 15:32:11 - Multiple API calls to list storage buckets from same IP
2025-01-23 08:22:45 - Credential exposure alert received by security team
2025-01-23 09:05:12 - API key revoked and rotated
```

### Technique 2: Authentication Anomaly Detection

**Purpose:** Identify suspicious authentication events that may indicate unauthorized use

**Implementation:**
1. Establish baseline authentication patterns:
   - Typical geographic locations
   - Normal working hours
   - Common device and user agent patterns
   - Typical access frequency and duration
   - Normal resource access patterns

2. Compare post-exposure authentication against baseline:
   ```
   // Example approach for anomaly detection
   $baseline = authentication.logs.filter(user_id).before(exposure_date);
   $post_exposure = authentication.logs.filter(user_id).after(exposure_date);
   
   anomalies = detect_anomalies(
     $baseline,
     $post_exposure,
     metrics=[
       "geolocation",
       "time_of_day",
       "user_agent",
       "session_duration",
       "resources_accessed"
     ]
   )
   ```

3. Score and prioritize anomalies based on deviation from baseline

**Example:**
```
BASELINE PATTERN:
- Authentication from Sydney, Australia (>95% of logins)
- Login times between 08:00-18:00 AEST weekdays
- MacOS with Chrome browser (90% of sessions)
- Average 3-5 service accesses per session

ANOMALOUS PATTERN DETECTED:
- Authentication from Kyiv, Ukraine (0% historical presence)
- Login at 03:17 AEST (outside normal hours)
- Windows with Firefox browser (rare combination)
- Accessed 12 different services in single session
- Accessed sensitive data areas never used before
```

### Technique 3: Credential Exposure Risk Scoring

**Purpose:** Quantify risk of credential exposure to prioritize investigation and response

**Implementation:**
1. Define risk factors and weights:
   - Credential privilege level (weight: 30%)
   - Exposure scope (public vs internal) (weight: 25%)
   - Exposure duration (weight: 15%)
   - Evidence of misuse (weight: 20%)
   - Sensitive data access (weight: 10%)

2. Calculate risk score based on factors:
   ```
   // Example approach for risk scoring
   risk_score = (
     (privilege_score * 0.3) +
     (exposure_scope_score * 0.25) +
     (exposure_duration_score * 0.15) +
     (misuse_evidence_score * 0.2) +
     (data_sensitivity_score * 0.1)
   ) * 10  // Scale to 0-100
   ```

3. Use risk score to determine investigation priority and response level

**Example:**
```
CREDENTIAL RISK ASSESSMENT:
- Admin privileges to critical system (score: 90/100)
- Public exposure on GitHub (score: 100/100)
- Exposed for 72 hours (score: 60/100)
- No evidence of misuse detected (score: 0/100)
- Provides access to sensitive customer data (score: 80/100)

OVERALL RISK SCORE: 
(90*0.3) + (100*0.25) + (60*0.15) + (0*0.2) + (80*0.1) = 65.5/100

RISK CATEGORIZATION: HIGH (requires immediate containment)
```

## Investigation Artifacts

### Required Artifacts

- **Credential Exposure Timeline**: Chronological record of all credential-related events
- **Credential Access Map**: Documentation of systems and data accessible with the credential
- **Authentication Log Analysis**: Evidence of normal vs. suspicious authentication activity
- **Exposure Source Documentation**: Analysis of how the credential was exposed
- **Impact Assessment**: Evaluation of actual and potential impact from the exposure

### Optional Artifacts

- **User Interview Notes**: Documentation from discussions with affected users
- **Credential Risk Score**: Quantitative assessment of exposure risk
- **Forensic Analysis Results**: If endpoint compromise was involved
- **Remediation Recommendations**: Specific guidance for addressing the exposure

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| Test/Development Credentials | Credentials with "test" in name; Credentials used in non-production environments; Limited access scope | Check credential naming convention; Verify environment restrictions; Confirm with development team; Check access limitations |
| Deliberately Shared Credentials | Multiple users accessing same account; Access from multiple locations; Consistent usage pattern | Confirm with team if credential sharing is authorized; Check if it matches documented shared account; Verify against approved exception list |
| Historical Credentials in Breach Databases | Credentials appear in breach notification; No evidence of recent use; May appear in multiple breaches | Check credential rotation logs; Compare breach date with last rotation date; Verify current credential format differs from exposed one |
| Self-Reporting of Own Exposure | User reports their own credential exposure; Clear explanation of cause; Immediate reporting after incident | Interview user about circumstances; Verify timeline of exposure and reporting; Check if access patterns match user's explanation |

## Investigation Completion Criteria

### Required Findings

- Clear determination of credential validity and status
- Complete understanding of exposure source and vector
- Full timeline from exposure to detection
- Assessment of unauthorized access (confirmed or ruled out)
- Evaluation of impact and affected systems
- Recommendations for containment and remediation

### Required Documentation

- Detailed credential exposure timeline
- Evidence supporting source of exposure determination
- Analysis of authentication logs during exposure window
- Assessment of actual vs. potential impact
- Containment and remediation recommendations

## Related SOPs

- [Leaked Secrets Response SOP](../../sops/leaked-secrets-response-sop.md)
- [Credential Rotation Procedures](../../sops/credential-rotation-procedures-sop.md)
- [Malware Incident Response SOP](../../sops/malware-incident-response-sop.md)

## Additional Resources

- [NIST Special Publication 800-63B: Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Spycloud Threat Research](https://spycloud.com/resources/research/)
- [Common Credential Exposure Patterns](https://www.sans.org/reading-room/whitepapers/authentication/patterns-credential-theft-39090)