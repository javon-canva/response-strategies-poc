# AWS EKS Security Incident Response Playbook

## Overview

**Incident Type:** Container Orchestration Security Incident  
**Severity Levels:** Medium/High/Critical  
**Response Team:** Detection & Response Team  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Detection Sources

- AWS GuardDuty EKS Protection
- CloudTrail EKS API activity logs
- Kubernetes audit logs
- Container runtime monitoring alerts
- EDR/XDR container security alerts

## Response Strategy

### Assumptions

- Access to AWS Management Console and AWS CLI with appropriate permissions
- Familiarity with Kubernetes concepts and kubectl command line
- AWS EKS clusters have logging enabled with CloudWatch
- Access to Sentinel One or other EDR tools for container monitoring

### Considerations

- EKS is a managed Kubernetes service that requires understanding both AWS and Kubernetes security contexts
- Changes to the cluster configuration may impact production workloads
- Isolating or removing compromised resources may affect application availability
- Coordinate with application teams before taking actions that could impact services
- Kubernetes security incidents may span multiple levels (AWS infrastructure, Kubernetes control plane, and application containers)

## Triage

### Initial Assessment

1. Access AWS Console and Review Alert Details
   - Log into the AWS Console and navigate to the EKS service
   - Identify the specific EKS cluster(s) mentioned in the alert
   - Review AWS CloudTrail logs for suspicious API calls against the EKS cluster:
     ```
     event.source:"eks.amazonaws.com" AND event.action:("CreateCluster" OR "DeleteCluster" OR "DeleteNodegroup" OR "UpdateClusterConfig" OR "UpdateNodegroupConfig")
     ```
   - Review AWS GuardDuty for any EKS-related findings (EKS audit logs, runtime monitoring)
   - Examine the timeline of events to understand the sequence leading to the alert

2. Investigate Cluster and Node Configuration
   - Configure kubectl to connect to the affected cluster:
     ```bash
     aws eks update-kubeconfig --name <cluster-name> --region <region>
     ```
   - Validate the cluster status and check for unauthorized node groups:
     ```bash
     kubectl get nodes -o wide
     kubectl get pods --all-namespaces -o wide
     ```
   - Look for suspicious workloads, particularly:
     - Pods running in unexpected namespaces
     - Pods with unusual image sources or suspicious names
     - Pods with excessive privileges or host mounts
     - DaemonSets that may have been deployed across all nodes

3. Follow the [Kubernetes Investigation Procedure](../../data-source-procedures/kubernetes-investigation.md) for detailed analysis steps

### Severity Assessment Criteria

| Severity | Criteria |
|----------|----------|
| Critical | Confirmed unauthorized access to cluster admin credentials; Evidence of data exfiltration; Multiple nodes compromised; Active exploitation of critical vulnerabilities; Production impact with service disruption | 
| High     | Suspicious privileged pods deployed; Lateral movement attempts; Unauthorized deployment modifications; Unusual network activity from container workloads; Exposure of sensitive configurations |
| Medium   | Unusual API calls with limited impact; Policy violations without evidence of malicious intent; Suspicious activity in non-production clusters; Vulnerable components identified but not actively exploited |
| Low      | Minor policy violations; Expected but unusual activity pattern; Potential false positive with no evidence of compromise |

### Validation Criteria

- Analyze container activities to confirm suspicious behavior:
  - Review pod logs for suspicious activities:
    ```bash
    kubectl logs <pod-name> -n <namespace>
    ```
  - Check for unusual network connections from containers:
    ```bash
    kubectl exec -it <pod-name> -n <namespace> -- netstat -tulpn
    ```
  - Look for unauthorized changes to deployments, services, or ingress configurations:
    ```bash
    kubectl get deployments,services,ingress --all-namespaces -o yaml
    ```

- Examine RBAC configuration for unauthorized changes:
  - Check for unauthorized role bindings or service accounts:
    ```bash
    kubectl get clusterrolebindings,rolebindings --all-namespaces -o wide
    kubectl get serviceaccounts --all-namespaces -o wide
    ```
  - Look for overly permissive roles that may have been created:
    ```bash
    kubectl get clusterroles,roles --all-namespaces -o yaml
    ```

## Investigation

### Investigation Pattern

Follow the [Kubernetes Security Investigation Pattern](../../investigation-patterns/kubernetes-security-investigation.md) for this incident type.

### Data Sources

Use these data source procedures to gather evidence:
- [AWS CloudTrail Investigation](../../data-source-procedures/aws-cloudtrail-investigation.md)
- [Kubernetes Audit Log Analysis](../../data-source-procedures/kubernetes-audit-log-analysis.md)
- [Container Runtime Security Analysis](../../data-source-procedures/container-runtime-security-analysis.md)

### Investigation Steps

1. **Analyze AWS Control Plane Activity**
   - Review EKS API calls in CloudTrail
   - Examine IAM policy and role changes relevant to the EKS cluster
   - Check for changes to security groups, VPC configurations, or network ACLs
   - Look for unusual API calls from unfamiliar principals or locations

2. **Investigate Kubernetes Control Plane Activity**
   - Examine Kubernetes audit logs for suspicious activities
   - Review authentication events, particularly failed attempts
   - Look for creation of high-privilege resources (ClusterRoles, ClusterRoleBindings)
   - Check for unusual API patterns like enumeration attempts

3. **Analyze Workload and Container Activity**
   - Review container images and repositories for indicators of compromise
   - Examine container process activity using runtime monitoring tools
   - Check for unusual network connections, file access, or process executions
   - Look for evidence of lateral movement between containers or nodes

## Containment

### Containment Strategy

Implement a defense-in-depth approach to contain the incident at multiple levels: AWS IAM, Kubernetes RBAC, network isolation, and workload removal.

### Containment Steps

1. **Isolate Affected Resources**
   - If a compromised pod is identified, isolate it using network policies:
     ```bash
     kubectl create -f - <<EOF
     apiVersion: networking.k8s.io/v1
     kind: NetworkPolicy
     metadata:
       name: isolate-compromised-pod
       namespace: <namespace>
     spec:
       podSelector:
         matchLabels:
           <label-key>: <label-value>
       policyTypes:
       - Ingress
       - Egress
     EOF
     ```
   - If a node is compromised, cordon and drain it:
     ```bash
     kubectl cordon <node-name>
     kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
     ```
   - Expected outcome: Compromised resources are isolated from the network and prevented from communicating with other resources

2. **Revoke Compromised Credentials**
   - If IAM credentials were compromised, rotate all affected access keys:
     ```bash
     aws iam update-access-key --access-key-id <key-id> --status Inactive --user-name <username>
     ```
   - If Kubernetes service account tokens were compromised, delete and recreate the service account:
     ```bash
     kubectl delete serviceaccount <sa-name> -n <namespace>
     kubectl create serviceaccount <sa-name> -n <namespace>
     ```
   - Expected outcome: Compromised credentials can no longer be used to access the cluster

3. **Restrict Access to EKS API**
   - Temporarily restrict access to the EKS API endpoints by updating security groups:
     ```bash
     aws ec2 update-security-group-rule-descriptions-ingress \
       --group-id <eks-cluster-security-group-id> \
       --ip-permissions '[{"IpProtocol": "tcp", "FromPort": 443, "ToPort": 443, "IpRanges": [{"CidrIp": "<trusted-cidr>", "Description": "Emergency access restriction"}]}]'
     ```
   - Expected outcome: Only trusted IPs can access the EKS API, preventing further unauthorized configuration changes

## Eradication

### Eradication Strategy

Remove all unauthorized resources and malicious components, and patch vulnerabilities that may have been exploited.

### Eradication Steps

1. **Remove Unauthorized Resources**
   - Delete suspicious pods, deployments, or daemonsets:
     ```bash
     kubectl delete pod <pod-name> -n <namespace>
     kubectl delete deployment <deployment-name> -n <namespace>
     kubectl delete daemonset <daemonset-name> -n <namespace>
     ```
   - Remove unauthorized RBAC configurations:
     ```bash
     kubectl delete clusterrolebinding <binding-name>
     kubectl delete rolebinding <binding-name> -n <namespace>
     ```
   - Expected outcome: All unauthorized resources are removed from the cluster

2. **Patch Vulnerabilities**
   - Update EKS cluster to the latest Kubernetes version:
     ```bash
     aws eks update-cluster-version \
       --name <cluster-name> \
       --kubernetes-version <target-version>
     ```
   - Ensure node groups are using the latest AMIs:
     ```bash
     aws eks update-nodegroup-version \
       --cluster-name <cluster-name> \
       --nodegroup-name <nodegroup-name>
     ```
   - Expected outcome: Vulnerabilities that may have been exploited are patched

3. **Enhance Security Controls**
   - Enable Kubernetes audit logging if not already enabled:
     ```bash
     aws eks update-cluster-config \
       --name <cluster-name> \
       --logging '{"clusterLogging":[{"types":["audit","api","authenticator"],"enabled":true}]}'
     ```
   - Implement Pod Security Standards:
     ```bash
     kubectl apply -f - <<EOF
     apiVersion: constraints.gatekeeper.sh/v1beta1
     kind: K8sPSPPrivilegedContainer
     metadata:
       name: psp-privileged-container
     spec:
       match:
         kinds:
           - apiGroups: [""]
             kinds: ["Pod"]
       parameters:
         allowPrivilegedContainer: false
     EOF
     ```
   - Expected outcome: Enhanced security controls prevent similar incidents in the future

## Recovery

### Recovery Strategy

Restore normal cluster operations once the threat has been contained and eradicated.

### Recovery Steps

1. **Restore Normal Operations**
   - After confirming the threat has been mitigated, remove temporary network policies:
     ```bash
     kubectl delete networkpolicy isolate-compromised-pod -n <namespace>
     ```
   - Uncordon nodes if they were cordoned during containment:
     ```bash
     kubectl uncordon <node-name>
     ```
   - Reset security group rules to normal configuration:
     ```bash
     aws ec2 update-security-group-rule-descriptions-ingress \
       --group-id <eks-cluster-security-group-id> \
       --ip-permissions <original-ip-permissions>
     ```
   - Expected outcome: Cluster returns to normal operation with appropriate security controls

2. **Verify Service Functionality**
   - Ensure critical applications are running normally:
     ```bash
     kubectl get pods -n <namespace>
     kubectl describe deployment <deployment-name> -n <namespace>
     ```
   - Test application endpoints to confirm functionality
   - Expected outcome: All applications function normally with no disruption

## Post-Incident Activities

### Lessons Learned

Reference the [Post-Incident Review Process](../../sops/post-incident-review-sop.md) for conducting a comprehensive review.

### Key Metrics to Collect

- Time to detection
- Time to containment
- Number of affected resources
- Duration of service impact (if any)
- Root cause identification timeline

### Detection Improvement

- Review and update EKS-specific detection rules
- Implement additional GuardDuty EKS Protection features if not already enabled
- Consider implementing Kubernetes runtime monitoring if not already in place
- Evaluate the need for container image scanning in CI/CD pipelines

## Additional Resources

- [Generic IR SOPs](https://canvadev.atlassian.net/wiki/spaces/CDR/pages/2855797478/Standard+Operating+Procedures)
- [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html)
- [Kubernetes Security Documentation](https://kubernetes.io/docs/concepts/security/)
- [AWS EKS Security Best Practices](https://aws.github.io/aws-eks-best-practices/security/docs/)
- [AWS CLI EKS Commands Reference](https://docs.aws.amazon.com/cli/latest/reference/eks/index.html)

## Related Artifacts

- [EKS Security Configuration Baseline](../../artifact-guidelines/eks-security-baseline.md)
- [Kubernetes Pod Security Standards](../../artifact-guidelines/kubernetes-pod-security.md)