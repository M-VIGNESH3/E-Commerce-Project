# Kubernetes RBAC for ClahanStore E-Commerce

## 📚 What is RBAC?

**Role-Based Access Control (RBAC)** controls WHO can do WHAT on WHICH resources in your Kubernetes cluster.

```
WHO (Subject)     +  WHAT (Role)           =  Access
─────────────────────────────────────────────────────
User "john"       +  Role "developer"      =  Can deploy apps, view logs
Group "qa-team"   +  Role "viewer"         =  Can only view resources
ServiceAccount    +  Role "cicd-deployer"  =  Can deploy via pipeline
```

## 🏗️ RBAC Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                    CLUSTER LEVEL                                  │
│  ┌──────────────────────┐    ┌─────────────────────────────┐     │
│  │ ClusterRole:         │    │ ClusterRoleBinding:          │     │
│  │   cluster-admin      │◄───│   platform-admin-binding    │     │
│  │   (built-in)         │    │   → Group: platform-admins  │     │
│  └──────────────────────┘    │   → User: clahanstore-owner │     │
│                              └─────────────────────────────────┘  │
├──────────────────────────────────────────────────────────────────┤
│                NAMESPACE: clahanstore                             │
│                                                                   │
│  ┌─────────────────┐  ┌───────────────────┐                      │
│  │ Role:            │  │ RoleBinding:       │                     │
│  │  namespace-admin ├──┤  → clahanstore-   │                     │
│  │  (full access)   │  │    admins group   │                     │
│  └─────────────────┘  └───────────────────┘                      │
│                                                                   │
│  ┌─────────────────┐  ┌───────────────────┐                      │
│  │ Role:            │  │ RoleBinding:       │                     │
│  │  developer       ├──┤  → clahanstore-   │                     │
│  │  (no secrets)    │  │    developers     │                     │
│  └─────────────────┘  └───────────────────┘                      │
│                                                                   │
│  ┌─────────────────┐  ┌───────────────────┐                      │
│  │ Role:            │  │ RoleBinding:       │                     │
│  │  viewer          ├──┤  → clahanstore-   │                     │
│  │  (read-only)     │  │    viewers        │                     │
│  └─────────────────┘  └───────────────────┘                      │
│                                                                   │
│  ┌─────────────────┐  ┌───────────────────┐                      │
│  │ Role:            │  │ RoleBinding:       │                     │
│  │  cicd-deployer   ├──┤  → SA: cicd-      │                     │
│  │  (deploy only)   │  │    deployer       │                     │
│  └─────────────────┘  └───────────────────┘                      │
│                                                                   │
│  ┌─────────────────────────────────────────────────────────┐     │
│  │ Workload ServiceAccounts (one per microservice)          │     │
│  │  api-gateway-sa, user-service-sa, auth-service-sa, ...  │     │
│  │  All bound to "microservice-workload" role (minimal)     │     │
│  └─────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────┘
```

## 📁 File Structure

```
kubernetes/rbac/
├── README.md                        ← You are here
│
├── ── Roles (define permissions) ──
├── namespace-admin-role.yaml        ← Full namespace access
├── developer-role.yaml              ← Deploy apps, no secrets
├── viewer-role.yaml                 ← Read-only access
├── cicd-deployer-role.yaml          ← CI/CD pipeline permissions
├── workload-roles.yaml              ← Minimal pod-level permissions
│
├── ── Bindings (assign roles) ──
├── cluster-admin-binding.yaml       ← Cluster-wide admin access
├── namespace-admin-binding.yaml     ← Namespace admin binding
├── developer-binding.yaml           ← Developer group binding
├── viewer-binding.yaml              ← Viewer group binding
├── cicd-deployer-binding.yaml       ← CI/CD SA binding
├── workload-role-bindings.yaml      ← Per-service SA bindings
│
├── ── Service Accounts ──
├── cicd-deployer-sa.yaml            ← CI/CD pipeline identity
├── workload-service-accounts.yaml   ← Per-microservice identities
│
└── ── Network Security ──
    └── network-policies.yaml        ← Pod-to-pod traffic rules
```

## 🚀 How to Apply

### Step 1: Apply all RBAC resources
```bash
# Apply everything in the rbac directory
kubectl apply -f kubernetes/rbac/ -n clahanstore

# Or apply in order (recommended for first time)
kubectl apply -f kubernetes/rbac/workload-service-accounts.yaml
kubectl apply -f kubernetes/rbac/cicd-deployer-sa.yaml
kubectl apply -f kubernetes/rbac/namespace-admin-role.yaml
kubectl apply -f kubernetes/rbac/developer-role.yaml
kubectl apply -f kubernetes/rbac/viewer-role.yaml
kubectl apply -f kubernetes/rbac/cicd-deployer-role.yaml
kubectl apply -f kubernetes/rbac/workload-roles.yaml
kubectl apply -f kubernetes/rbac/cluster-admin-binding.yaml
kubectl apply -f kubernetes/rbac/namespace-admin-binding.yaml
kubectl apply -f kubernetes/rbac/developer-binding.yaml
kubectl apply -f kubernetes/rbac/viewer-binding.yaml
kubectl apply -f kubernetes/rbac/cicd-deployer-binding.yaml
kubectl apply -f kubernetes/rbac/workload-role-bindings.yaml
kubectl apply -f kubernetes/rbac/network-policies.yaml
```

### Step 2: Verify resources were created
```bash
# List all roles and bindings
kubectl get roles -n clahanstore
kubectl get rolebindings -n clahanstore
kubectl get clusterrolebindings | grep platform
kubectl get serviceaccounts -n clahanstore
kubectl get networkpolicies -n clahanstore
```

## 🧪 How to Test RBAC

### Test as a Developer (should NOT have secret access)
```bash
# Can a developer list pods? → YES
kubectl auth can-i list pods -n clahanstore \
  --as=dev-user --as-group=clahanstore-developers

# Can a developer view secrets? → NO
kubectl auth can-i get secrets -n clahanstore \
  --as=dev-user --as-group=clahanstore-developers

# Can a developer create deployments? → YES
kubectl auth can-i create deployments -n clahanstore \
  --as=dev-user --as-group=clahanstore-developers

# Can a developer exec into pods? → YES
kubectl auth can-i create pods/exec -n clahanstore \
  --as=dev-user --as-group=clahanstore-developers
```

### Test as a Viewer (should be read-only)
```bash
# Can a viewer list pods? → YES
kubectl auth can-i list pods -n clahanstore \
  --as=viewer1 --as-group=clahanstore-viewers

# Can a viewer delete pods? → NO
kubectl auth can-i delete pods -n clahanstore \
  --as=viewer1 --as-group=clahanstore-viewers

# Can a viewer create deployments? → NO
kubectl auth can-i create deployments -n clahanstore \
  --as=viewer1 --as-group=clahanstore-viewers
```

### Test CI/CD Deployer ServiceAccount
```bash
# Can CI/CD deploy? → YES
kubectl auth can-i create deployments -n clahanstore \
  --as=system:serviceaccount:clahanstore:cicd-deployer

# Can CI/CD delete RBAC roles? → NO
kubectl auth can-i delete roles -n clahanstore \
  --as=system:serviceaccount:clahanstore:cicd-deployer
```

## 🔑 Key Concepts Summary

| Concept | Scope | Purpose |
|---------|-------|---------|
| **Role** | Namespace | Define permissions within ONE namespace |
| **ClusterRole** | Cluster | Define permissions across ALL namespaces |
| **RoleBinding** | Namespace | Assign a Role to users/groups/SAs in ONE namespace |
| **ClusterRoleBinding** | Cluster | Assign a ClusterRole across ALL namespaces |
| **ServiceAccount** | Namespace | Identity for pods/automation (not humans) |
| **NetworkPolicy** | Namespace | Control pod-to-pod network traffic |

## ⚠️ Notes

1. **Replace `clahanstore-owner`** in `cluster-admin-binding.yaml` with your actual K8s username:
   ```bash
   kubectl config view --minify -o jsonpath='{.contexts[0].context.user}'
   ```

2. **Network Policies require a compatible CNI** (Calico, Cilium, or Weave). If using Flannel, policies are created but not enforced.

3. **To add workload SAs to deployments**, add `serviceAccountName` to your pod specs and redeploy with `helm upgrade`.
