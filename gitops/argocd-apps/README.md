# ArgoCD Apps - GitOps Repository

This directory contains Kubernetes manifests for deploying infrastructure and applications via ArgoCD as an **App-of-Apps** pattern.

## Directory Structure

```
argocd-apps/
├── README.md                        # This file
├── app-of-apps.yaml                # Bootstrap Application (manages other apps)
├── metallb-application.yaml         # MetalLB LoadBalancer controller
└── longhorn-application.yaml        # Longhorn distributed storage
```

## Prerequisites

- **k3s cluster** running and accessible via kubeconfig
- **ArgoCD** installed in the `argocd` namespace
- **kubectl** configured to access the cluster
- **Git repository** containing this folder (referenced in app-of-apps.yaml)

## Quick Start

### Option 1: Deploy via ArgoCD Ansible Role (Recommended)

The Ansible playbook (`../Ansible/site.yml`) automatically deploys the App-of-Apps when running the `argocd` role.

```bash
cd ../Ansible
ansible-playbook site.yml -i inventory/k3s/hosts.ini -u <user> --become --ask-become-pass
```

The bootstrap Application will be created automatically and will reconcile all child applications.

### Option 2: Manual kubectl Deployment

If ArgoCD is already running, manually apply the App-of-Apps to bootstrap the GitOps flow:

```bash
# Navigate to this directory
cd argocd-apps

# Apply the bootstrap Application (App-of-Apps pattern)
kubectl apply -f app-of-apps.yaml

# Verify it was created
kubectl -n argocd get app argocd-app-of-apps
```

## Managing Applications

### View all ArgoCD Applications

```bash
kubectl -n argocd get app
```

Expected output:
```
NAME                    SYNC STATUS   HEALTH STATUS
argocd-app-of-apps      Synced        Healthy
metallb                 Synced        Healthy
longhorn                Synced        Healthy
```

### Check individual app status

```bash
kubectl -n argocd describe app metallb
kubectl -n argocd describe app longhorn
```

### View logs and events

```bash
# ArgoCD controller logs
kubectl -n argocd logs deployment/argocd-application-controller

# Longhorn pod status
kubectl -n longhorn-system get pods

# MetalLB pod status
kubectl -n metallb-system get pods
```

### Sync applications manually

```bash
# Sync all apps
argocd app sync --all

# Or via kubectl
kubectl -n argocd patch app metallb -p '{"metadata":{"finalizers":["resources-finalizer.argocd.argoproj.io"]}}' --type merge
```

## Application Details

### MetalLB (`metallb-application.yaml`)

- **Namespace**: `metallb-system`
- **Chart**: `metallb/metallb` v0.13.10
- **Purpose**: Provides LoadBalancer service type support for bare-metal k3s
- **Configuration**:
  - Address pool: Configured via IPAddressPool CRD (see `../Ansible/roles/metallb/tasks/main.yml`)
  - Protocol: Layer 2

### Longhorn (`longhorn-application.yaml`)

- **Namespace**: `longhorn-system`
- **Chart**: `longhorn/longhorn` v1.7.0
- **Purpose**: Distributed storage for persistent volumes
- **Configuration**:
  - Default storage class: `longhorn` (automatically set as default)
  - Replica count: 3 (configurable in `../Ansible/roles/longhorn/defaults/main.yml`)
  - UI replicas: 1 (access via `kubectl port-forward`)

### App-of-Apps (`app-of-apps.yaml`)

- **Namespace**: `argocd`
- **Purpose**: Bootstrap application that references all child applications
- **Git Source**: Points to this repository's `argocd-apps` directory
- **Auto-sync**: Enabled (automatically syncs when Git changes)
- **Prune**: Enabled (removes k8s resources if deleted from Git)

## Updating Applications

Since applications use auto-sync, any changes pushed to Git will be automatically reconciled:

1. Edit the Application manifest (e.g., `metallb-application.yaml`)
2. Commit and push to Git
3. ArgoCD detects the change within ~3 minutes (or trigger manually)
4. Applications sync to the desired state

Example: Update MetalLB chart version

```yaml
# metallb-application.yaml
source:
  chart: metallb
  targetRevision: "0.14.0"  # Change version here
```

```bash
git add metallb-application.yaml
git commit -m "Update MetalLB to v0.14.0"
git push
# ArgoCD will auto-sync within 3 minutes
```

## Troubleshooting

### App shows "OutOfSync"

Check the Git repository URL and branch in `app-of-apps.yaml`:

```bash
kubectl -n argocd get app argocd-app-of-apps -o yaml | grep -A5 source
```

Ensure the `repoURL` and `targetRevision` match your repository.

### Child applications not appearing

The App-of-Apps must have read access to this directory in Git. Verify:
- Git URL is correct
- Branch exists
- Ansible role ran successfully (check logs for `Create ArgoCD bootstrap Application`)

### MetalLB services stuck in Pending

Check if MetalLB pods are running:

```bash
kubectl -n metallb-system get pods
kubectl -n metallb-system get ipaddresspool
```

Ensure address pool is configured (see `../Ansible/roles/metallb/tasks/main.yml`).

### Longhorn pods not starting

Check node resources and affinity:

```bash
kubectl -n longhorn-system describe pod <pod-name>
kubectl get nodes -o wide
```

Longhorn requires available disk space and compatible node labels.

## Next Steps

1. **Add custom applications**: Create new `*.yaml` Application manifests in this directory
2. **Use Kustomize/Helm**: Reference Kustomize overlays or Helm values in Application specs
3. **Secrets management**: Use ArgoCD + SOPS or SealedSecrets for sensitive data
4. **App-of-Apps hierarchy**: Nest Applications to organize by team/environment

## References

- [ArgoCD App-of-Apps Pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#app-of-apps)
- [ArgoCD Application CRD](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml)
- [MetalLB Documentation](https://metallb.universe.tf/)
- [Longhorn Documentation](https://longhorn.io/docs/)
