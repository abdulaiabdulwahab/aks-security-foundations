# aks-security-foundations

# AKS Security Foundations

## Project Overview

This project demonstrates how to secure an Azure Kubernetes Service (AKS) cluster using several core AKS security features.

The goal is to understand how identity, access control, pod security, network policies, node patching, and image cleanup work together to protect workloads running in Kubernetes.

## Technologies Used

- Azure Kubernetes Service (AKS)
- Azure CLI
- Kubernetes
- Microsoft Entra ID
- Azure RBAC
- Cilium
- Kubernetes Network Policies
- Pod Security Admission
- AKS Image Cleaner

## Security Features Implemented

This project includes:

- Microsoft Entra ID authentication
- Azure RBAC for Kubernetes authorization
- Disabled local AKS administrator accounts
- Pod Security Admission using the `restricted` policy
- Secure Kubernetes container security contexts
- Cilium network policies
- Default-deny network rules
- Automatic AKS node security patching
- AKS Image Cleaner

## Project Architecture

```text
Administrator
     |
     v
Microsoft Entra ID
     |
     v
Azure RBAC
     |
     v
AKS Cluster
     |
     +-- Pod Security Admission
     |
     +-- Secure Containers
     |
     +-- Cilium Network Policies
     |
     +-- Image Cleaner
     |
     +-- Automatic Node Security Updates
```

## Create the Resource Group

```bash
export RG="rg-aks-security-foundations"
export LOCATION="canadacentral"
export AKS="aks-security-foundations"

az group create \
  --name "$RG" \
  --location "$LOCATION"
```

## Create the AKS Cluster

```bash
az aks create \
  --resource-group "$RG" \
  --name "$AKS" \
  --location "$LOCATION" \
  --node-count 1 \
  --enable-aad \
  --enable-azure-rbac \
  --disable-local-accounts \
  --network-plugin azure \
  --network-plugin-mode overlay \
  --network-dataplane cilium \
  --node-os-upgrade-channel SecurityPatch \
  --enable-image-cleaner \
  --image-cleaner-interval-hours 24 \
  --generate-ssh-keys
```

## Connect to the Cluster

```bash
az aks get-credentials \
  --resource-group "$RG" \
  --name "$AKS" \
  --overwrite-existing
```

Verify the cluster:

```bash
kubectl get nodes
```

## Create a Secure Namespace

```bash
kubectl create namespace secure-lab
```

Enable the Kubernetes restricted Pod Security Standard:

```bash
kubectl label --overwrite namespace secure-lab \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/audit=restricted \
  pod-security.kubernetes.io/warn=restricted
```

## Container Security

The secure workload was configured with settings such as:

```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
  seccompProfile:
    type: RuntimeDefault
```

Container-level protections included:

```yaml
securityContext:
  allowPrivilegeEscalation: false

  capabilities:
    drop:
      - ALL

  readOnlyRootFilesystem: true
```

These controls reduce the privileges available to a container if it becomes compromised.

## Network Security

A default-deny NetworkPolicy was created to block incoming traffic by default.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny-ingress
  namespace: secure-lab

spec:
  podSelector: {}

  policyTypes:
    - Ingress
```

Additional policies can then explicitly allow trusted workloads to communicate with the application.

## Security Validation

Useful commands used during the project included:

```bash
kubectl get pods -n secure-lab
```

```bash
kubectl get networkpolicy -n secure-lab
```

```bash
kubectl get namespace secure-lab --show-labels
```

```bash
az aks show \
  --resource-group "$RG" \
  --name "$AKS" \
  --query autoUpgradeProfile
```

```bash
az aks show \
  --resource-group "$RG" \
  --name "$AKS" \
  --query securityProfile.imageCleaner
```

## Troubleshooting

### Kubernetes Access Denied

If `kubectl` returns:

```text
Error from server (Forbidden)
```

verify your Azure RBAC role assignments.

### Pod Rejected

If a pod fails to deploy, inspect it with:

```bash
kubectl describe pod <pod-name> \
  -n secure-lab
```

Pod Security Admission may block workloads that run as root, use privileged mode, allow privilege escalation, or use insecure Linux capabilities.

### Network Connectivity Problems

Check the configured network policies:

```bash
kubectl get networkpolicy \
  -n secure-lab
```

Then inspect a specific policy:

```bash
kubectl describe networkpolicy \
  -n secure-lab
```

## Skills Practiced

This project provided hands-on experience with:

- AKS security
- Microsoft Entra ID
- Azure RBAC
- Kubernetes security
- Pod Security Admission
- Kubernetes Security Contexts
- Cilium
- Network Policies
- Least privilege
- Container isolation
- AKS node security patching
- AKS Image Cleaner

## Cleanup

Delete the project resources when finished:

```bash
az group delete \
  --name "$RG" \
  --yes \
  --no-wait
```

## Conclusion

This project demonstrates the foundations of securing workloads running on AKS.

Instead of relying on default Kubernetes configurations, the cluster uses centralized identity, least-privilege access, pod security controls, network isolation, automated security patching, and image cleanup.

These controls provide a stronger foundation for running containerized applications securely in Azure.