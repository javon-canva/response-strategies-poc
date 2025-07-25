# Kubernetes Security Investigation Pattern

## Overview

**Pattern Type:** Container Orchestration Security Analysis  
**Applicable Incident Types:** EKS Security Incidents, Kubernetes Cluster Compromise, Container Breakout, Pod Security Policy Violations  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Pattern Description

This investigation pattern provides a structured approach for investigating security incidents in Kubernetes environments, including managed services like AWS EKS. It addresses the unique multi-layered security model of Kubernetes, covering infrastructure, control plane, and workload security aspects to enable comprehensive incident analysis across the entire Kubernetes security stack.

## Pre-Investigation Steps

### Data Source Preparation

1. **Identify Required Data Sources**
   - [Kubernetes Audit Logs](../../data-source-procedures/kubernetes-audit-log-analysis.md)
   - [Container Runtime Logs](../../data-source-procedures/container-runtime-security-analysis.md)
   - [Cloud Provider Control Plane Logs](../../data-source-procedures/aws-cloudtrail-investigation.md)
   - [Network Flow Logs](../../data-source-procedures/vpc-flow-logs-analysis.md)

2. **Verify Data Availability**
   - Ensure Kubernetes audit logging is enabled and accessible
   - Verify container runtime logging (Docker/containerd) is available
   - Confirm access to cloud provider logs (e.g., CloudTrail for AWS EKS)
   - Check retention periods match the investigation timeframe
   - Verify completeness of log data across all required cluster nodes

3. **Prepare Investigation Environment**
   - Set up kubectl access to the affected cluster
   - Prepare command-line tools for log analysis
   - Establish secure access to cloud provider consoles
   - Create a clean environment for reviewing potentially malicious artifacts
   - Document initial cluster state before making investigative changes

## Investigation Methodology

### Phase 1: Infrastructure and Control Plane Analysis

**Purpose:** Identify unauthorized access or configuration changes at the infrastructure and Kubernetes control plane level

**Steps:**
1. Review cloud provider control plane logs:
   ```
   # AWS CloudTrail EKS API Calls
   event.source:"eks.amazonaws.com" AND 
   event.action:("CreateCluster" OR "DeleteCluster" OR "DeleteNodegroup" OR "UpdateClusterConfig")
   ```

2. Examine Kubernetes API server authentication logs:
   ```
   # Kubernetes Authentication Events
   kubectl logs -n kube-system kube-apiserver-* | grep -E 'authentication|Authorization|access|denied'
   
   # For EKS or managed services
   cat /var/log/audit/kube-apiserver-audit.log | grep -E 'authentication|Authorization|access|denied'
   ```

3. Analyze cluster role and binding changes:
   ```
   # Review RBAC configurations
   kubectl get clusterroles,clusterrolebindings --all-namespaces -o yaml
   
   # Check for privileged roles
   kubectl get clusterroles -o json | jq '.items[] | select(.rules[].resources[] | contains("secrets"))'
   ```

4. Review node configuration and admission:
   ```
   # Check node status
   kubectl describe nodes
   
   # Review kubelet configurations
   ssh node-host "cat /var/lib/kubelet/config.yaml"
   ```

**Expected Outcomes:**
- Identification of unauthorized API calls to the cluster
- Evidence of authentication or authorization bypass attempts
- Detection of suspicious RBAC policy changes
- Understanding of potential node-level compromise

**Pivot Points:**
- If unauthorized authentication events are found, pivot to credential theft investigation
- If suspicious API calls are identified, expand to include user activity analysis
- If RBAC changes are detected, focus on privilege escalation vectors
- If node configuration changes are found, investigate potential infrastructure compromise

### Phase 2: Workload and Runtime Security Analysis

**Purpose:** Identify malicious or compromised workloads running within the cluster

**Steps:**
1. Inventory all running workloads:
   ```
   # List all pods across namespaces
   kubectl get pods --all-namespaces -o wide
   
   # Identify privileged pods
   kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.containers[].securityContext.privileged == true)'
   ```

2. Review suspicious pod specifications:
   ```
   # Examine pod with host access
   kubectl get pod <suspicious-pod> -n <namespace> -o yaml
   
   # Check for sensitive mounts
   kubectl get pod <suspicious-pod> -n <namespace> -o json | jq '.spec.volumes[]'
   ```

3. Analyze container images and origins:
   ```
   # Review image sources
   kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}" | tr -s '[[:space:]]' '\n' | sort | uniq -c
   
   # Check for unauthorized image registries
   kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}" | grep -v "authorized-registry"
   ```

4. Examine pod logs for suspicious activity:
   ```
   # Review pod logs
   kubectl logs <suspicious-pod> -n <namespace> --all-containers
   
   # Look for specific suspicious patterns
   kubectl logs <suspicious-pod> -n <namespace> | grep -E 'curl|wget|bash'
   ```

**Expected Outcomes:**
- Identification of unauthorized or suspicious workloads
- Evidence of container escape attempts or host access
- Detection of malicious container images
- Understanding of suspicious activity within containers

**Pivot Points:**
- If unauthorized images are found, investigate image supply chain
- If privileged containers are detected, focus on potential container escape attempts
- If suspicious network activity is identified, pivot to network traffic analysis
- If evidence of lateral movement is found, expand scope to connected systems

### Phase 3: Network Security Analysis

**Purpose:** Identify unauthorized network communications to, from, or within the cluster

**Steps:**
1. Review Kubernetes network policies:
   ```
   # List network policies
   kubectl get networkpolicies --all-namespaces
   
   # Check for overly permissive policies
   kubectl get networkpolicies --all-namespaces -o yaml
   ```

2. Analyze service exposures:
   ```
   # Review all services
   kubectl get svc --all-namespaces
   
   # Identify externally exposed services
   kubectl get svc --all-namespaces -o json | jq '.items[] | select(.spec.type == "LoadBalancer")'
   ```

3. Examine pod network activity:
   ```
   # For pods with shell access
   kubectl exec -it <suspicious-pod> -n <namespace> -- netstat -tulpn
   
   # For pods without shell access
   kubectl exec -it <suspicious-pod> -n <namespace> -- ss -tulpn
   ```

4. Review cloud provider flow logs for suspicious traffic:
   ```
   # Example AWS VPC Flow Logs query
   source.ip:<pod-ip> AND destination.ip:[NOT <internal-range>] AND destination.port:(22 OR 2375 OR 6379 OR 3306)
   ```

**Expected Outcomes:**
- Identification of unauthorized network communication
- Evidence of data exfiltration attempts
- Detection of command and control communications
- Understanding of lateral movement within the cluster

**Pivot Points:**
- If unauthorized external traffic is detected, investigate potential data exfiltration
- If connections to known malicious IPs are found, pivot to threat intelligence analysis
- If internal lateral movement is identified, expand scope to potentially compromised nodes
- If service exposure issues are found, focus on configuration security review

## Data Correlation Techniques

### Technique 1: Pod-to-Infrastructure Mapping

**Purpose:** Correlate suspicious pod activity with infrastructure-level events to identify the full scope of compromise

**Implementation:**
1. Identify the node where a suspicious pod is running:
   ```
   kubectl get pod <suspicious-pod> -n <namespace> -o wide
   ```

2. Examine node-level logs during the time of suspicious activity:
   ```
   # Get node instance ID (AWS)
   NODE_INSTANCE_ID=$(aws ec2 describe-instances --filters "Name=private-dns-name,Values=<node-name>" --query "Reservations[].Instances[].InstanceId" --output text)
   
   # Review CloudTrail for actions related to this node
   event.source:(ec2.amazonaws.com) AND aws.cloudtrail.request_parameters.instanceId:<NODE_INSTANCE_ID>
   ```

3. Correlate container creation time with control plane events:
   ```
   # Get pod creation time
   POD_CREATION_TIME=$(kubectl get pod <suspicious-pod> -n <namespace> -o jsonpath='{.metadata.creationTimestamp}')
   
   # Search Kubernetes audit logs around this time
   grep -A 20 -B 20 "$POD_CREATION_TIME" /var/log/audit/kube-apiserver-audit.log
   ```

**Example:**
```
# Identify pod creation time
kubectl get pod malicious-pod -n default -o jsonpath='{.metadata.creationTimestamp}'
# Output: 2025-07-24T15:30:45Z

# Search audit logs
grep -A 20 -B 20 "2025-07-24T15:3" /var/log/audit/kube-apiserver-audit.log
# Look for who created the pod and any associated events
```

### Technique 2: Container Lifecycle Analysis

**Purpose:** Trace the complete lifecycle of a suspicious container to identify initial compromise and subsequent actions

**Implementation:**
1. Extract container runtime ID from Kubernetes:
   ```
   kubectl get pod <suspicious-pod> -n <namespace> -o json | jq '.status.containerStatuses[0].containerID'
   # Example output: "containerd://7b9fc63cd43653b391a83873cb52c1771534af70c7e90b78102ea348e2f7837a"
   ```

2. Examine container runtime logs for this container:
   ```
   # For containerd
   crictl logs 7b9fc63cd43653b391a83873cb52c1771534af70c7e90b78102ea348e2f7837a
   
   # For Docker
   docker logs 7b9fc63cd43653b391a83873cb52c1771534af70c7e90b78102ea348e2f7837a
   ```

3. Review container image history:
   ```
   # Get image ID
   IMAGE_ID=$(kubectl get pod <suspicious-pod> -n <namespace> -o jsonpath='{.status.containerStatuses[0].imageID}')
   
   # Examine image layers (if accessible)
   docker image history $IMAGE_ID
   ```

4. Correlate with image registry logs:
   ```
   # For ECR (AWS)
   aws ecr describe-images --repository-name <repo-name> --image-ids imageDigest=<image-digest>
   ```

**Example:**
```
# Extract container ID
CONTAINER_ID=$(kubectl get pod malicious-pod -n default -o jsonpath='{.status.containerStatuses[0].containerID}' | sed 's/containerd:\/\///g')

# Examine container logs and events
crictl logs $CONTAINER_ID
crictl inspect $CONTAINER_ID
```

## Investigation Artifacts

### Required Artifacts

- **Cluster State Snapshot**: Point-in-time capture of running resources and configurations
- **Pod and Container Analysis**: Detailed information about suspicious pods and their contents
- **RBAC Configuration Report**: Documentation of cluster roles and bindings
- **Network Communication Map**: Visualization of pod communication patterns
- **Timeline of Events**: Chronological sequence of key events from infrastructure to workload

### Optional Artifacts

- **Container Image Analysis**: Detailed examination of suspicious container images
- **Memory or Process Dumps**: For in-depth forensic analysis of compromised containers
- **Kubernetes Custom Resource Inventory**: Documentation of custom resources that might be malicious
- **Service Account Token Usage**: Analysis of service account token usage and permissions

## Common False Positives

| Scenario | Indicators | How to Verify |
|----------|-----------|---------------|
| CI/CD Pipeline Deployment | Pod creation with elevated privileges from CI service account; Short-lived pods with build tools; Network communication to image registries | Verify against CI/CD job history; Check if image source matches approved registries; Confirm build job patterns match known workflows |
| Administrative Maintenance | Privileged pods in kube-system namespace; Temporary debug containers; Direct node SSH access | Check for corresponding change records; Verify administrator identity; Confirm activity timing with maintenance windows |
| Autoscaling or Self-Healing | Sudden pod terminations and creations; Node draining and replacement; Temporary configuration changes | Correlate with HPA/cluster autoscaler logs; Verify metrics that triggered scaling; Check for node health events preceding actions |

## Investigation Completion Criteria

### Required Findings

- Determination of initial entry point and attack vector
- Identification of all affected resources within the cluster
- Assessment of potential data or credential exposure
- Verification of malicious activity vs. false positive
- Documentation of the full attack chain if malicious

### Required Documentation

- Complete timeline from initial compromise to detection
- List of all affected resources and their current state
- Recommendations for remediation and security improvements
- Evidence preservation for potential future legal requirements
- Lessons learned and detection improvement opportunities

## Related SOPs

- [Kubernetes Cluster Containment SOP](../../sops/kubernetes-cluster-containment-sop.md)
- [Container Image Security Review SOP](../../sops/container-image-security-review-sop.md)
- [RBAC Review and Remediation SOP](../../sops/rbac-review-remediation-sop.md)

## Additional Resources

- [Kubernetes Security Documentation](https://kubernetes.io/docs/concepts/security/)
- [CNCF Kubernetes Security Best Practices](https://www.cncf.io/blog/2019/01/14/9-kubernetes-security-best-practices-everyone-must-follow/)
- [AWS EKS Security Best Practices](https://aws.github.io/aws-eks-best-practices/security/docs/)
- [Kubernetes Forensics](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)