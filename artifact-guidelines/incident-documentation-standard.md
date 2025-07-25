# Incident Documentation Standard

## Purpose

This document defines the standards for creating and maintaining incident documentation artifacts. Proper documentation is essential for effective incident response, knowledge transfer, post-incident analysis, and compliance requirements. Following these standards ensures consistency, completeness, and usability of incident documentation across the organization.

## Scope

These standards apply to all documentation created during security incident response activities, including:

- Initial incident reports
- Investigation notes and findings
- Evidence collection and analysis
- Containment and remediation actions
- Post-incident analysis
- Lessons learned and recommendations

## General Documentation Principles

### 1. Clarity and Precision

- Use clear, concise language that is easily understood
- Avoid ambiguity and technical jargon when possible
- Define all acronyms and specialized terms upon first use
- Focus on facts rather than assumptions or opinions
- Clearly distinguish between confirmed findings and working hypotheses

### 2. Chronological Structure

- Document events in chronological order
- Include accurate timestamps (with timezone) for all significant events
- Create clear timelines that show the sequence of events
- Note any gaps or uncertainties in the timeline
- Record both incident events and response actions with timestamps

### 3. Completeness

- Document all significant aspects of the incident
- Include negative findings (things checked and ruled out)
- Record what was done, found, and not found
- Document tool versions, commands, and queries used
- Capture decisions made and their rationale

### 4. Objectivity

- Focus on factual information
- Separate observations from interpretations
- Avoid assigning blame to individuals or teams
- Use neutral, professional language
- Document multiple perspectives when relevant

### 5. Security and Privacy

- Classify documentation according to sensitivity
- Redact personally identifiable information (PII) when appropriate
- Apply least-privilege access controls to documentation
- Consider legal and regulatory requirements
- Follow chain-of-custody procedures for evidence

## Required Documentation Structure

### 1. Incident Summary

Every incident documentation must include a summary section containing:

| Element | Description | Example |
|---------|-------------|---------|
| Incident ID | Unique identifier | INC-2025-0142 |
| Incident Title | Brief, descriptive title | AWS RDS Snapshot Unauthorized Restoration |
| Severity Level | Using standard severity matrix | High |
| Date/Time Discovered | When incident was detected | 2025-07-15T08:23:45Z |
| Date/Time Contained | When incident was contained | 2025-07-15T11:42:10Z |
| Date/Time Resolved | When incident was resolved | 2025-07-16T15:30:22Z |
| Lead Responder | Primary incident handler | Jane Smith |
| Affected Systems | List of impacted systems | prod-db-cluster, api-backend-service |
| Incident Type | Classification of incident | Data Access Violation |
| Executive Summary | 1-2 paragraph overview | Unauthorized restoration of production database snapshot to development environment, exposing customer data. Incident contained within 3 hours with no evidence of external data exfiltration. |

### 2. Timeline Documentation

All incident timelines must:

- Use ISO 8601 format (YYYY-MM-DDTHH:MM:SSZ) with UTC timezone
- Include both incident events and response actions
- Distinguish between detection events and actual occurrence times
- Note the source of timestamp information (log source, system, etc.)
- Indicate confidence level for estimated timestamps

Example timeline format:

```
2025-07-15T08:23:45Z [DETECTION] CloudTrail alert triggered for unauthorized RDS snapshot restoration (High confidence)
2025-07-15T08:15:32Z [INCIDENT] RDS snapshot restoration initiated by user account jdoe@example.com (High confidence, CloudTrail logs)
2025-07-15T08:30:12Z [RESPONSE] Incident response team notified and investigation initiated
2025-07-15T09:47:22Z [INCIDENT] First evidence of data access to restored database instance (Medium confidence, DB logs)
2025-07-15T10:15:08Z [RESPONSE] User account jdoe@example.com suspended
2025-07-15T11:42:10Z [RESPONSE] Unauthorized RDS instance terminated and access logs preserved
```

### 3. Technical Investigation Documentation

Technical investigation artifacts must include:

- Tools and methods used for investigation
- Search queries with timestamps and results
- Commands executed for evidence collection
- Key log entries and their interpretation
- Screenshots of relevant findings (annotated when necessary)
- Chain of custody information for collected evidence
- Analysis methodology and reasoning

Example format:

```
Tool: AWS CloudTrail Log Analysis
Query: event.source:"rds.amazonaws.com" AND event.action:"RestoreDBInstanceFromDBSnapshot" AND user.name:"jdoe@example.com"
Timestamp: 2025-07-15T09:15:22Z
Results: 2 matching events found
  Event 1: Snapshot prod-db-backup-20250714 restored to dev-test-db-instance
  Event 2: Snapshot prod-db-backup-20250713 restored to dev-analysis-instance
Analysis: Both restoration events originated from the same source IP (203.0.113.42) which is not associated with the user's normal access pattern.
```

### 4. Impact Assessment Documentation

Impact assessment documentation must include:

- Systems and services affected
- Duration of impact
- Data potentially accessed or compromised
- Users or customers affected
- Business functions impacted
- Regulatory or compliance implications
- Financial impact (if known)

Example format:

```
Affected System: Production Customer Database (RDS Instance ID: prod-customer-db-01)
Impact Type: Unauthorized Data Access
Duration: 3 hours 27 minutes (08:15 UTC to 11:42 UTC)
Data Scope: Customer records including names, email addresses, and purchase history
Data Volume: Approximately 1.2 million customer records
Regulatory Impact: Potential GDPR Article 33 notification required (under assessment)
Business Impact: No service disruption; potential reputational damage if disclosed
```

### 5. Containment and Remediation Documentation

Containment and remediation documentation must include:

- Actions taken to contain the incident
- Steps to eradicate the threat
- Remediation measures implemented
- Verification methods used
- Restoration activities performed
- Timeline of containment/remediation activities
- Individuals who performed each action

Example format:

```
Containment Action: Terminate unauthorized RDS instance
Performed By: Alex Johnson (Security Engineer)
Timestamp: 2025-07-15T11:42:10Z
Verification Method: AWS Console confirmation and CloudTrail log verification
Evidence: Screenshot of terminated instance and CloudTrail API logs saved to evidence folder

Remediation Action: Implement additional CloudTrail alerting for RDS snapshot operations
Performed By: Chris Williams (Security Architect)
Timestamp: 2025-07-16T09:30:15Z
Verification Method: Test alert triggered successfully with simulated event
Evidence: Alert configuration documentation saved to incident folder
```

### 6. Communication Documentation

All incident-related communications must be documented:

- Stakeholder notifications with timestamps
- External communications (if applicable)
- Team coordination messages
- Management updates
- Handover communications

Example format:

```
Communication Type: Management Update
Timestamp: 2025-07-15T10:00:00Z
Sender: Jane Smith (IR Lead)
Recipients: CTO, CISO, VP Engineering
Content: Incident summary and initial findings
Response: CISO requested hourly updates until containment
```

### 7. Root Cause Analysis Documentation

Root cause analysis documentation must include:

- Primary and contributing causes
- Timeline of key events leading to the incident
- Technical and process vulnerabilities identified
- Human factors considerations
- Contextual factors that influenced the incident
- Analysis methodology used

Example format:

```
Primary Cause: Excessive permissions granted to development role
Contributing Factors:
1. Lack of segmentation between production and development environments
2. Absence of approval workflow for sensitive operations
3. Insufficient monitoring of high-risk operations

Analysis Method: 5 Whys analysis + CAST (Causal Analysis using Systems Theory)
Supporting Evidence: Role permission audit trail showing permission expansion on 2025-06-30
```

### 8. Lessons Learned Documentation

Lessons learned documentation must include:

- What worked well during the response
- What could be improved
- Specific recommendations for:
  - Technical controls
  - Process improvements
  - Training needs
  - Documentation updates
- Prioritized action items with owners and deadlines

Example format:

```
Strength: Rapid detection via CloudTrail alerting (8 minutes from event to alert)
Improvement Area: Delay in containing unauthorized instance (3+ hours)
Recommendation: Implement automated containment workflow for unauthorized resource creation
Priority: High
Owner: Security Engineering Team
Target Date: 2025-08-15
Success Criteria: Automated containment successfully tested in simulation
```

## Documentation Tools and Templates

### Approved Documentation Tools

- Incident Management System: Jira/ServiceNow
- Timeline Creation: Timeline JS or integrated tool
- Evidence Collection: FTK Imager, Velociraptor
- Collaborative Documentation: Confluence, Google Docs
- Secure Storage: Evidence Management System

### Standard Templates

The following templates must be used for incident documentation:

1. [Incident Response Form](../templates/incident-response-form.md)
2. [Technical Investigation Worksheet](../templates/technical-investigation.md)
3. [Evidence Collection Log](../templates/evidence-collection-log.md)
4. [Root Cause Analysis Template](../templates/root-cause-analysis.md)
5. [Post-Incident Review Template](../templates/post-incident-review.md)

## Documentation Review and Approval

### Review Requirements

All incident documentation must be reviewed by:

1. Technical Lead: For accuracy of technical details
2. Incident Commander: For completeness and clarity
3. Legal/Compliance (for incidents with regulatory implications)

### Approval Process

1. Document author submits for review in the incident management system
2. Reviewers provide feedback or approval within established SLAs
3. Author addresses feedback and resubmits if needed
4. Final approval documented in the incident management system
5. Documentation moved to appropriate storage location

## Documentation Retention

### Retention Requirements

- All incident documentation must be retained for a minimum of 7 years
- High-severity incident documentation must be retained for 10 years
- Documentation involving regulated data follows specific regulatory retention requirements

### Storage Locations

- Active incidents: Incident Management System
- Closed incidents (< 90 days): Secure Documentation Repository
- Archived incidents (> 90 days): Long-term Secure Storage
- Evidence files: Evidence Management System with chain-of-custody controls

## Documentation Training and Resources

All incident responders must complete training on these documentation standards. The following resources are available:

1. Documentation Standards Training Course (required annually)
2. Documentation Best Practices Guide
3. Example Documentation Library
4. Documentation Mentoring Program

## Compliance and Audit

Documentation will be audited regularly for:

1. Adherence to these standards
2. Completeness and accuracy
3. Appropriate security controls
4. Retention compliance

Audit findings will be reported to the Security Operations leadership and addressed within 30 days.

## References

- [NIST SP 800-61 Computer Security Incident Handling Guide](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf)
- [SANS Incident Handler's Handbook](https://www.sans.org/reading-room/whitepapers/incident/incident-handlers-handbook-33901)
- [ISO/IEC 27035:2016 Information Security Incident Management](https://www.iso.org/standard/60803.html)
- [Organization Security Policy Document #IR-7: Incident Documentation Requirements]