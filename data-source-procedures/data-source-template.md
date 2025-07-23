# [Data Source Name] Investigation Procedure

## Overview

**Data Source:** [Specific data source name, e.g., AWS CloudTrail, GCP Audit Logs, Okta Logs]  
**Environment:** [Specific environment or platform, e.g., AWS, GCP, On-premise]  
**Version:** [X.Y]  
**Last Updated:** [YYYY-MM-DD]  
**Maintainer:** [Name/Team]

## Data Source Description

[Brief description of what this data source is and what types of events/data it contains]

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| Console/UI | [URL or access path] | [Authentication method] | [Any specific notes] |
| API | [Endpoint or SDK] | [Authentication method] | [Any specific notes] |
| CLI | [Command example] | [Authentication method] | [Any specific notes] |

### Required Permissions

- [Permission 1]
- [Permission 2]
- [Additional permissions as needed]

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| Production | [Log path or index] | [Retention period] | [Any specific notes] |
| Development | [Log path or index] | [Retention period] | [Any specific notes] |
| [Other environments] | [Log path or index] | [Retention period] | [Any specific notes] |

## Common Investigation Scenarios

### Scenario 1: [Brief description of scenario]

#### Query Examples

```
[Query syntax example]
```

**Fields of interest:**
- `field1`: [Description of what this field represents]
- `field2`: [Description of what this field represents]
- [Additional fields as needed]

#### Interpretation Guidelines

- [How to interpret the results]
- [Common patterns to look for]
- [Potential false positives]

### Scenario 2: [Brief description of scenario]

#### Query Examples

```
[Query syntax example]
```

**Fields of interest:**
- `field1`: [Description of what this field represents]
- `field2`: [Description of what this field represents]
- [Additional fields as needed]

#### Interpretation Guidelines

- [How to interpret the results]
- [Common patterns to look for]
- [Potential false positives]

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| [Data source 1] | [Field to join on] | [How to use for correlation] |
| [Data source 2] | [Field to join on] | [How to use for correlation] |
| [Additional data sources] | [Field to join on] | [How to use for correlation] |

### Common Correlation Techniques

1. **[Technique 1]**
   - [Description of correlation technique]
   - [Example query or approach]
   - [Expected outcomes]

2. **[Technique 2]**
   - [Description of correlation technique]
   - [Example query or approach]
   - [Expected outcomes]

## Data Collection for Investigations

### Evidence Collection Process

1. [Step 1 for collecting evidence from this data source]
2. [Step 2 for collecting evidence from this data source]
3. [Additional steps as needed]

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| [Method 1] | [Command or process] | [Format] | [Any specific notes] |
| [Method 2] | [Command or process] | [Format] | [Any specific notes] |
| [Additional methods] | [Command or process] | [Format] | [Any specific notes] |

## Known Limitations

- [Limitation 1]
- [Limitation 2]
- [Additional limitations as needed]

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| [Error 1] | [Common cause] | [How to resolve] |
| [Error 2] | [Common cause] | [How to resolve] |
| [Additional errors] | [Common cause] | [How to resolve] |

## Additional Resources

- [Reference 1: Link or document]
- [Reference 2: Link or document]
- [Additional references as needed]

## Related SOPs

- [Link to related SOP 1]
- [Link to related SOP 2]
- [Additional related SOPs as needed]