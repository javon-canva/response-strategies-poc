# GCP Cloud Audit Logs Investigation Procedure

## Overview

**Data Source:** GCP Cloud Audit Logs  
**Environment:** Google Cloud Platform  
**Version:** 1.0  
**Last Updated:** 2025-07-25  
**Maintainer:** Detection & Response Team

## Data Source Description

GCP Cloud Audit Logs provide detailed records of administrative activities and access within Google Cloud Platform environments. These logs capture who did what, where, and when within your GCP projects, similar to AWS CloudTrail. This data source is essential for security investigations, compliance monitoring, and understanding infrastructure changes.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Console/UI | [GCP Console](https://console.cloud.google.com/) → Logging → Logs Explorer | Google Workspace SSO (canva-internal.com) | Primary UI method |
| CLI | `gcloud logging read` | gcloud auth login | For advanced filtering and scripting |
| Elastic | Custom integrations | Okta SSO | May not have complete coverage |

### Required Permissions

- For Console: `logging.viewer` or higher role on the project
- For DR investigations: Request `canva.dr.read` or `canva.dr.admin` via Pengy
- For CLI: Local gcloud setup with appropriate authentication

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| Project-specific | GCP Console → Logging | 30-400 days depending on configuration | Primary source |
| Centralized | Logging → Log Router → Log Buckets | Configurable | For long-term storage |
| Special Projects | cnvprd-ww-prove-com-seed | 400 days | For Cloud Build histories |

## Common Investigation Scenarios

### Scenario 1: Determining Actor Behind GCP Actions

#### Query Examples

```
resource.type="audited_resource"
protoPayload.authenticationInfo.principalEmail=~".*@canva.com"
```

**Fields of interest:**
- `protoPayload.authenticationInfo.principalEmail`: Email of the user who performed the action
- `protoPayload.requestMetadata.callerIp`: Source IP address of the request
- `protoPayload.methodName`: The method or API called
- `protoPayload.serviceName`: The GCP service involved

#### Interpretation Guidelines

- Direct console actions will show the user's email and source IP
- Cloud Build deployments will show a Google Cloud IP as the source
- For Cloud Build actions, check the cnvprd-ww-prove-com-seed project to trace to the original actor
- Compare the activity timestamp with the alert/incident time

### Scenario 2: Investigating CloudBuild Deployments

#### Query Examples

```
# In cnvprd-ww-prove-com-seed project
resource.type="build"
resource.labels.build_id="BUILD_ID"
```

**Fields of interest:**
- `resource.labels.build_id`: Unique identifier for the build
- `textPayload`: Contains details about the build steps
- `jsonPayload.buildDetails`: Contains information about the build configuration
- `resource.labels.build_trigger_id`: If triggered automatically

#### Interpretation Guidelines

- Examine the related Cloud Source Repository commit
- Review the template changes (usually YAML or Terraform)
- Verify if changes match what was observed in alert payload
- Check commit description and actor details

### Scenario 3: Correlating Host Activity to GCP Changes

#### Query Examples

```
# Example S1 EDR query for gcloud authentication
process_name:gcloud.cmd cmdline:"auth login"
```

**Fields of interest:**
- Authentication commands: `gcloud auth login`
- Deployment commands: `pythonX ./tools/mirror_to_gcp_cloud_build.py`
- File paths in command line arguments

#### Interpretation Guidelines

- Check for proximity of authentication and deployment commands
- Look for paths to .tf files in deployment commands
- Verify timestamps align with GCP activity
- Examine host logs around the time of GCP changes

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Okta Logs | `source.ip` | Correlate IPs to identify human actors |
| GitHub | PR content and authors | Find related infrastructure changes |
| Jira | Ticket IDs in commit messages | Connect changes to authorized work |
| EDR | Command line activity | Verify local actions leading to GCP changes |

### Common Correlation Techniques

1. **PR and Deployment Correlation**
   - Description: Link infrastructure changes to approved code
   - Example approach: 
     - Identify Cloud Build activity in cnvprd-ww-prove-com-seed
     - Follow link to Cloud Source Repository commit
     - Find related PR in gcp-foundations repository
   - Expected outcomes: Verify changes were approved and legitimate

2. **Local-to-Cloud Activity Chain**
   - Description: Trace local commands to GCP changes
   - Example approach:
     - Identify `gcloud auth login` activity on host
     - Look for subsequent Terraform or deployment commands
     - Match file paths to observed infrastructure changes
   - Expected outcomes: Connect local user activity to GCP changes

## Data Collection for Investigations

### Evidence Collection Process

1. Capture Cloud Audit Logs showing the relevant activity:
   - Filter by appropriate time range and project
   - Include all relevant fields, especially authentication info
   - Export as JSON for preservation

2. Document Cloud Build and Repository evidence:
   - Screenshot or export build details
   - Capture commit information and changes
   - Save links to relevant PRs

3. Collect host evidence if available:
   - Command history showing gcloud authentication
   - Terminal logs of deployment actions
   - Local .tf files used in deployment

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| Console Export | GCP Console → Logging → Export | JSON/CSV | Good for smaller datasets |
| gcloud CLI | `gcloud logging read --format=json > export.json` | JSON | Better for scripting |
| Build History | Download from Cloud Build UI | JSON | For build-specific details |

## Known Limitations

- GCP infrastructure is not fully managed through IaC, leading to potential drift
- Gap between PRs/tickets and actual effect on infrastructure
- GitHub search may not reliably find all relevant PRs
- Some logs may have limited retention periods
- Different service-specific audit detail levels

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| Missing activity logs | Log filtering issues | Try broadening the time range or removing filters |
| Unable to trace to PR | Manual changes or search limitations | Manually review recent PRs in gcp-foundations |
| Authentication failures with CLI | Wrong account | Use canva-internal.com account for authentication |
| Limited log access | Insufficient permissions | Request canva.dr.read or admin role via Pengy |

## Additional Resources

- [GCP Cloud Audit Logs Documentation](https://cloud.google.com/logging/docs/audit)
- [Applying Terraform Changes to GCP](https://docs.canva.tech/playbooks/groups/developer-platform/buildkite/gcp/applying_tf_changes/)
- [GCP Foundations Repository](https://github.com/Canva/gcp-foundations)
- [GCP CLI Installation Guide](https://cloud.google.com/sdk/docs/install)

## Related SOPs

- [GCP User Containment SOP](../../sops/gcp-user-containment-sop.md)
- [Infrastructure Change Management](../../sops/infrastructure-change-management-sop.md)