# Kubernetes Investigation Procedure

## Overview

**Data Source:** Kubernetes Cluster Logs and API  
**Environment:** Kubernetes (AWS EKS, GKE, AKS, self-managed)  
**Version:** 1.0  
**Last Updated:** 2025-07-24  
**Maintainer:** Detection & Response Team

## Data Source Description

Kubernetes is a container orchestration platform that manages containerized applications across multiple hosts. This data source procedure covers the collection and analysis of Kubernetes audit logs, API server logs, container logs, and cluster state information during security investigations. It applies to both managed Kubernetes services (EKS, GKE, AKS) and self-managed clusters.

## Access Information

### Access Methods

| Access Method | URL/Command | Authentication | Notes |
|---------------|------------|----------------|-------|
| kubectl CLI | `kubectl --kubeconfig=/path/to/config` | kubeconfig file with certificates or tokens | Primary method for cluster interaction |
| Kubernetes API | `https://<cluster-api>:<port>` | Bearer token, certificates | Direct API access when kubectl is unavailable |
| Cloud Console | AWS/GCP/Azure Console UI | Cloud provider credentials | For managed services configuration |
| Kubernetes Dashboard | `https://<dashboard-url>` | Service account token | Optional UI access if deployed |

### Required Permissions

- Cluster-level read permissions (`cluster-admin` or custom role with get/list/watch verbs)
- Access to audit logs (varies by cloud provider)
- For EKS: CloudTrail read access for control plane events
- For GKE: Cloud Audit Logs viewer role
- For AKS: Azure Activity Logs access

### Location of Logs

| Environment | Log Location | Retention Period | Notes |
|------------|--------------|------------------|-------|
| AWS EKS | CloudWatch Logs | 90 days (default) | Requires explicit enabling |
| GCP GKE | Cloud Logging | 30 days (default) | Automatically enabled |
| Azure AKS | Azure Log Analytics | 30 days (default) | Requires configuration |
| Self-managed | Master nodes: `/var/log/kube-apiserver-audit.log` | Varies by configuration | Requires manual configuration |

## Common Investigation Scenarios

### Scenario 1: Unauthorized Resource Creation

#### Query Examples

```bash
# Check for pod creation events in all namespaces
kubectl get events --all-namespaces --field-selector reason=Created,involvedObject.kind=Pod

# Review audit logs for pod creation (AWS EKS)
aws logs filter-log-events --log-group-name /aws/eks/cluster-name/cluster --filter-pattern '{ $.verb = "create" && $.objectRef.resource = "pods" }'

# Check recent pods across all namespaces
kubectl get pods --all-namespaces --sort-by=.metadata.creationTimestamp
```

**Fields of interest:**
- `user.username`: The user or service account that created the resource
- `objectRef.namespace`: The namespace where the resource was created
- `objectRef.name`: The name of the created resource
- `responseStatus.code`: Success or failure response code
- `userAgent`: The client used to create the resource

#### Interpretation Guidelines

- Look for resources created in unexpected namespaces
- Check for resources created by unexpected service accounts or users
- Identify unusual naming patterns that don't match organizational conventions
- Verify the source IP of the request is from an expected network
- Analyze the user agent string for unexpected clients

### Scenario 2: Privilege Escalation Activity

#### Query Examples

```bash
# Check for privileged containers
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.containers[].securityContext.privileged == true)'

# Review role bindings for cluster-admin permissions
kubectl get clusterrolebinding -o json | jq '.items[] | select(.roleRef.name == "cluster-admin")'

# Check for containers with host path mounts
kubectl get pods --all-namespaces -o json | jq '.items[] | select((.spec.volumes[]?.hostPath != null) or (.spec.containers[].volumeMounts[].mountPath == "/host"))'
```

**Fields of interest:**
- `roleRef.name`: The role being referenced (watch for cluster-admin)
- `subjects`: The entities being granted the permissions
- `securityContext.privileged`: Boolean indicating if container runs with elevated privileges
- `volumes.hostPath.path`: Host filesystem paths mounted into containers
- `capabilities.add`: Linux capabilities added to containers

#### Interpretation Guidelines

- Verify that cluster-admin role bindings are limited to essential system accounts
- Check that privileged containers are only used for legitimate infrastructure needs
- Ensure host path mounts are restricted to system components in kube-system
- Look for unusual capabilities added to containers (e.g., CAP_SYS_ADMIN)
- Verify that namespace role bindings don't grant excessive permissions

### Scenario 3: Suspicious Network Activity

#### Query Examples

```bash
# List all services exposing ports
kubectl get services --all-namespaces

# Check network policies
kubectl get networkpolicies --all-namespaces

# Examine pods with hostNetwork enabled
kubectl get pods --all-namespaces -o json | jq '.items[] | select(.spec.hostNetwork == true)'

# Check for NodePort and LoadBalancer services
kubectl get svc --all-namespaces -o json | jq '.items[] | select(.spec.type == "LoadBalancer" or .spec.type == "NodePort")'
```

**Fields of interest:**
- `spec.hostNetwork`: Boolean indicating if pod uses host networking
- `spec.type`: Service type (ClusterIP, NodePort, LoadBalancer)
- `spec.ports`: Port mappings for services
- `spec.loadBalancerSourceRanges`: IP CIDR ranges allowed to access LoadBalancer services
- `spec.externalTrafficPolicy`: How external traffic is handled

#### Interpretation Guidelines

- Verify that services exposed externally are intended to be public-facing
- Check that hostNetwork is only used by necessary system components
- Ensure network policies exist to restrict communication between namespaces
- Look for unusual port exposures that don't align with application requirements
- Validate that LoadBalancer services have appropriate source IP restrictions

## Data Correlation

### Correlation with Other Data Sources

| Data Source | Correlation Field | Notes |
|-------------|------------------|-------|
| Cloud Provider Logs | Source IP, Principal ID | Connect Kubernetes actions to cloud IAM principals |
| Container Runtime Logs | Container ID | Link Kubernetes pod events to container runtime events |
| Host System Logs | Node name | Correlate node issues with Kubernetes events |
| Network Flow Logs | Pod IP addresses | Map pod network activity to external communications |

### Common Correlation Techniques

1. **Pod-to-Container Correlation**
   - Description: Link Kubernetes pods to their underlying container runtime IDs
   - Example approach:
     ```bash
     # Get pod details with container IDs
     kubectl get pod <pod-name> -n <namespace> -o json | jq '.status.containerStatuses[].containerID'
     
     # Use container ID with container runtime
     crictl logs <container-id>
     ```
   - Expected outcomes: Connect pod activity to container runtime events

2. **Cloud IAM to Kubernetes Authentication Correlation**
   - Description: Connect cloud provider authentication to Kubernetes API actions
   - Example approach:
     ```bash
     # First, get Kubernetes authentication events
     kubectl logs kube-apiserver-<node> -n kube-system | grep 'authentication'
     
     # Then, search cloud provider logs for the same principal
     # AWS example
     aws cloudtrail lookup-events --lookup-attributes AttributeKey=Username,AttributeValue=<principal-name>
     ```
   - Expected outcomes: Trace complete authentication chain from cloud IAM to Kubernetes actions

## Data Collection for Investigations

### Evidence Collection Process

1. Capture cluster state information:
   ```bash
   kubectl get namespaces -o yaml > namespaces.yaml
   kubectl get pods,services,deployments,statefulsets,daemonsets --all-namespaces -o yaml > workloads.yaml
   kubectl get clusterroles,clusterrolebindings,roles,rolebindings --all-namespaces -o yaml > rbac.yaml
   ```

2. Collect relevant pod logs:
   ```bash
   kubectl logs <pod-name> -n <namespace> --all-containers --previous > pod-logs.txt
   ```

3. Extract audit logs from appropriate location:
   - For EKS:
     ```bash
     aws logs get-log-events --log-group-name /aws/eks/<cluster-name>/cluster --log-stream-name kube-apiserver-audit-<id> > audit-logs.json
     ```
   - For self-managed:
     ```bash
     cat /var/log/kube-apiserver-audit.log > audit-logs.json
     ```

4. Document configuration settings:
   ```bash
   kubectl describe nodes > nodes.txt
   kubectl get configmaps --all-namespaces -o yaml > configmaps.yaml
   ```

### Data Export Methods

| Export Method | Command/Process | Format | Notes |
|---------------|----------------|--------|-------|
| kubectl output | `kubectl get <resource> -o yaml > file.yaml` | YAML | Best for configuration data |
| kubectl logs | `kubectl logs <pod> > logs.txt` | Text | For container logs |
| Cloud provider CLI | `aws/gcloud/az` logging export commands | JSON | For audit and control plane logs |
| kubectl exec | `kubectl exec <pod> -- <command>` | Varies | For live investigation |

## Known Limitations

- Kubernetes audit logs may not capture all read operations by default (depends on audit policy)
- Container logs are lost when pods are deleted unless a logging sidecar or agent is used
- Some cloud providers limit access to control plane components
- Log retention periods vary significantly across environments
- Ephemeral containers and short-lived pods may not leave sufficient logs for investigation

## Common Errors and Troubleshooting

| Error/Issue | Cause | Resolution |
|-------------|-------|------------|
| "Error from server (Forbidden)" | Insufficient permissions | Request elevated permissions or use a service account with appropriate RBAC |
| Missing audit logs | Audit logging not enabled | Enable audit logging or check cloud provider-specific audit log location |
| Container logs unavailable | Pod terminated or restarted | Use `--previous` flag or check centralized logging solution |
| "Unable to connect to the server" | API server unreachable or kubeconfig issues | Verify network connectivity and kubeconfig settings |
| Inconsistent resource listings | Cluster-wide permissions missing | Ensure permissions across all required namespaces |

## Additional Resources

- [Kubernetes Audit Logging Documentation](https://kubernetes.io/docs/tasks/debug-application-cluster/audit/)
- [EKS Logging and Monitoring](https://docs.aws.amazon.com/eks/latest/userguide/control-plane-logs.html)
- [GKE Logging and Monitoring](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-logging)
- [AKS Logging and Monitoring](https://docs.microsoft.com/en-us/azure/aks/monitor-aks)

## Related SOPs

- [Kubernetes Pod Security Review](../../sops/kubernetes-pod-security-review.md)
- [Container Image Analysis](../../sops/container-image-analysis.md)
- [RBAC Permission Review](../../sops/rbac-permission-review.md)