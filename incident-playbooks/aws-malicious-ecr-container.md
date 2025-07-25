# AWS Malicious ECR Container Incident Playbook

## Overview

**Incident Type:** Unauthorized Container Image Upload  
**Severity Levels:** Medium/High  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- [Elastic Detection Rule](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/security/rules/)
- [Detection Strategy Documentation](https://github.com/Canva/detection-strategies/blob/master/canva/aws_malicious_container.md)
- [Previous Response Examples](https://pew-pew.canva-internal.com/caseview/589a2c34-9600-4a96-9c40-50b9591a21dd)

## Response Strategy

### Assumptions

- Access to AWS Console, Elastic, and other internal tools
- Understanding of container registries and ECR functionality
- Familiarity with CloudTrail log formats

### Considerations

- Most `PutImage` actions do not trigger alerts; only those outside established `deploy` pipelines
- Not all `PutImage` actions mean a new image has been pushed - tagging an existing image will also appear as a `PutImage` action in CloudTrail
- Container images deployed outside standard pipelines may represent a security risk

## Triage

### Initial Assessment

1. Gather information about the newly pushed image:
   - Note the AWS account ID in `aws.cloudtrail.recipient_account_id`
   - Note the AWS region from `cloud.region` (most ECR assets are in `us-east-1`)
   - Examine the `aws.cloudtrail.user_identity.arn` to identify the actor
   - Note the ECR repository name from `aws.cloudtrail.flattened.request_parameters.repositoryName`
   - Record the image SHA256 from `aws.cloudtrail.flattened.response_elements.image.imageId.imageDigest`
   - Identify all associated image tags from `aws.cloudtrail.flattened.request_parameters.imageTag`

2. Follow the [AWS CloudTrail Investigation Procedure](../../data-source-procedures/aws-cloudtrail-investigation.md) for detailed analysis

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| High     | Unknown image uploaded to production ECR repository; Image with unusual tags or contents; Actor cannot be identified; Suspicious activity observed in addition to image upload | 
| Medium   | Non-pipeline image upload by known actor; Image uploaded to non-production repository; Action performed outside standard process but with identifiable business need |
| Low      | Legitimate use case confirmed by actor; Testing in approved environments; Known incident response or maintenance activity |

### Validation Criteria

- Check if the action relates to known engineering work:
  - Reliability incident resolution
  - Phoenix updates (Canva.com static site)
  - Pipeline improvement/testing
- Confirm the activity aligns with expected patterns for the role and user
- Verify the user's response when contacted matches their normal behavior and provides convincing explanation

## Investigation

### Investigation Pattern

Follow the [AWS Resource Investigation Pattern](../../investigation-patterns/aws-resource-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [Container Registry Investigation](../../data-source-procedures/container-registry-investigation.md)

### Investigation Steps

1. **Identify Related Engineering Project/Work**
   - Search for the identified `imageTag`(s) in Slack
   - Look up the tags in GitHub repositories across the Canva organization
   - If necessary, expand the search to Jira tickets
   - Note that tagging could be dynamic, so exact matches might not exist in code

2. **Analyze User Activity**
   - Inspect fields: `aws.cloudtrail.user_identity.*`, `user.id` and `user.roles`
   - Pivot off `access_key_id` to find other actions performed by the actor within the same session
   - Compare the activity to typical patterns for the role using the [Common Actions Dashboard](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/dashboards#/view/f3b3bf10-ec56-11ee-b98e-57e4996492b6)
   - Check if the `user_agent` indicates Terraform/Hashicorp usage (might be local testing)

3. **Identify and Contact the Actor**
   - If a person has been identified, look up their GitHub username in Canva World
   - Search for their recent PRs across the organization
   - Validate findings by correlating the `source.ip` with [Okta logs](https://d-r-primary.kb.us-east-1.aws.found.io:9243/s/production/app/discover)
   - Contact the user via Zoom (preferred) or Slack for confirmation

## Containment

### Containment Strategy

If the upload is determined to be unauthorized or malicious, contain the actor and affected resources.

### Containment Steps

1. **Isolate the Container Image**
   - Remove the suspect image from the ECR repository or prevent further pulls
   - Document the image details before removal for further investigation
   - Expected outcome: Suspect container is no longer available for deployment

2. **Suspend User Access**
   - Follow the [User Containment SOP](../../sops/user-account-containment-sop.md) to suspend the user's access
   - Revoke the specific AWS access keys involved
   - Expected outcome: Actor can no longer perform AWS actions

## Eradication

### Eradication Strategy

Remove unauthorized container images and ensure no further unauthorized uploads can occur.

### Eradication Steps

1. **Remove Unauthorized Container Images**
   - Delete the unauthorized images from all repositories
   - Check for any deployments using these images and coordinate removal
   - Expected outcome: All instances of the unauthorized images are removed

2. **Review and Enhance Access Controls**
   - Review IAM permissions for ECR operations
   - Implement additional controls if needed
   - Expected outcome: Only authorized actors can upload images

## Recovery

### Recovery Strategy

Work with engineering teams to restore proper functionality.

### Recovery Steps

1. **Restore Standard Operations**
   - Work with release engineers to ensure proper deployment pipeline functionality
   - Validate that legitimate deployments can proceed
   - Expected outcome: Normal deployment operations resume

2. **Document Process Improvements**
   - Collaborate with engineering teams to integrate any out-of-band processes into standard pipelines
   - Provide guidance on proper container image management
   - Expected outcome: Improved processes to reduce future alerts

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Time to actor identification
- Number of repositories affected
- Whether the image was deployed to any environments

### Detection Improvement

- Consider implementing image signing and verification requirements
- Evaluate ECR repository permission boundaries
- Review alerting thresholds for container-related activities

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS Documentation](https://docs.aws.amazon.com)
- [Container Security Best Practices](https://aws.amazon.com/blogs/containers/posts/container-security-best-practices/)

## Related Artifacts

- [Container Scanning Guidelines](../../artifact-guidelines/container-scanning.md)
- [ECR Security Configuration](../../artifact-guidelines/ecr-security-configuration.md)