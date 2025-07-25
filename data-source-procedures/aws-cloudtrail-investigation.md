# AWS CloudTrail Investigation Procedure

## Overview

**Data Source:** AWS CloudTrail Logs  
**Environment:** AWS  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Data Source Description

AWS CloudTrail logs capture detailed records of API calls made within AWS environments, providing an audit trail of all actions performed on AWS resources. This data source is critical for security investigations, compliance auditing, and operational troubleshooting.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Console/UI | [Elastic CloudTrail Index](https://d-r-primary.kb.us-east-1.aws.found.io:9243/app/r/s/KDVGh) | Okta SSO | Primary method for investigations |
| CLI | AWS CLI with appropriate roles | AWS IAM credentials | For advanced filtering |

### Required Permissions

- Access to Elastic Search/Kibana
- Appropriate AWS IAM read permissions
- Access to Canva's internal security tools

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| Production | Elastic CloudTrail Index | 90 days | Primary search interface |
| All AWS Environments | AWS CloudTrail service | 90 days | Direct AWS access if needed |

## Common Investigation Scenarios

### Scenario 1: Determining Actor Behind AWS Actions

#### Query Examples

```
aws.cloudtrail.user_identity.access_key_id: "ACCESS_KEY_ID"
```

**Fields of interest:**
- `user.name`: The user or role that performed the action
- `aws.cloudtrail.user_identity.access_key_id`: Access key identifier
- `aws.cloudtrail.user_identity.principal_id`: Principal identifier
- `source.ip`: Source IP address of the request
- `aws.cloudtrail.flattened.request_parameters.clientToken`: May identify Terraform-initiated actions

#### Interpretation Guidelines

- Check if actions were performed via Terraform by looking for `clientToken: terraform-*`
- For role-based actions, examine the chain of identity assumption
- Cross-reference source IPs with Okta logs to identify human actors
- Service users can be researched in GitHub, Confluence, and other documentation

### Scenario 2: Analyzing Session Activity

#### Query Examples

```
aws.cloudtrail.user_identity.access_key_id: "ACCESS_KEY_ID" AND NOT action.event: (List* OR Get* OR Describe*)
```

**Fields of interest:**
- `action.event`: The specific AWS API action performed
- `aws.cloudtrail.event_source`: The AWS service that processed the request
- `aws.cloudtrail.flattened.request_parameters`: Parameters of the request
- `aws.cloudtrail.flattened.response_elements`: Response from the API call

#### Interpretation Guidelines

- Filter out read-only actions to focus on modifications
- Group similar actions to understand the overall workflow
- Compare activity against expected baseline for the role
- Examine the timeline to identify unusual patterns or sequences

### Scenario 3: Identifying BuildKite-initiated Actions

#### Query Examples

```
aws.cloudtrail.flattened.request_parameters.clientToken: terraform-*
```

**Fields of interest:**
- `aws.cloudtrail.flattened.request_parameters.clientToken`: Token identifying the client
- `aws.cloudtrail.user_identity.session_context.session_issuer.user_name`: The role being assumed

#### Interpretation Guidelines

- BuildKite actions typically use Terraform and have specific client token patterns
- Follow up using the BuildKite job identification process to trace to GitHub PRs and Jira tickets
- Validate whether the changes match expected deployment patterns

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Okta Logs | `source.ip` | Correlate IPs to identify human actors |
| Identity Broker Logs | `user.roles` | Identify role assumption activities |
| Wiz | `source.ip` | Identify infrastructure assets by IP |
| GitHub | `user.name` | Find service accounts defined in code |

### Common Correlation Techniques

1. **Role Assumption Chain Analysis**
   - Description: Track the complete chain of identity assumption
   - Example query:
   ```
   identity_broker.audit.observer.info.credentials.aws.role : "ROLE_NAME"
   ```
   - Expected outcomes: Identify the original user who assumed a role

2. **IP-based Actor Attribution**
   - Description: Cross-reference CloudTrail IP addresses with authentication logs
   - Example approach: Search Okta logs for the same source IP around the time of CloudTrail activity
   - Expected outcomes: Connect AWS activity to authenticated users

## Data Collection for Investigations

### Evidence Collection Process

1. Export the complete session activity using the access key ID filter
2. Document the actor identification process and findings
3. Capture any role assumption chains relevant to the investigation
4. Document resource modifications with before/after state if available

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| Kibana Export | Use Elastic UI export function | CSV/JSON | Best for smaller datasets |
| Direct Query | Custom scripts against Elasticsearch API | JSON | Better for large datasets |

## Known Limitations

- CloudTrail may delay events by up to 15 minutes
- Not all AWS services log to CloudTrail with the same level of detail
- Data plane operations are often not captured (only control plane)
- Different event types have inconsistent field structures

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing events | Event delay or filtering | Expand time range or check CloudTrail delivery settings |
| Incomplete actor information | Role assumption chains | Use identity broker logs for additional context |
| Too many results | Broad search parameters | Use additional filters like `NOT action.event: (List* OR Get* OR Describe*)` |

## Additional Resources

- [AWS CloudTrail Documentation](https://docs.aws.amazon.com/awscloudtrail/)
- [Identifying BuildKite Jobs from CloudTrail](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/3907062786/Identifying+BuildKite+Jobs+Github+PRs+from+CloudTrail)
- [AWS Identifiers Reference](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html)

## Related SOPs

- [User Account Suspension Process](../../sops/user-account-containment-sop.md)
- [AWS Incident Response](../../incident-playbooks/aws-incident-response.md)