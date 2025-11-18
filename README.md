# Kubernetes RBAC Homework Submission

## Contents
- Part1: Basics & Tests
- Part2: Namespace-Scoped Access (monitoring-reader)
- Part3: Cluster-Wide Access (log-collector)
- Part4: Troubleshooting RBAC (broken -> fixed)
- Part5: Real-World RBAC Design
- Bonus: Label-enforced pod creation (Kyverno)
- YAML files for all resources in `Homework_4_comp488_RBAC/`
- Command outputs and explanations

---

## Prerequisites
- Access to a Kubernetes cluster (minikube, kind, or cloud)
- `kubectl` configured to a context with cluster-admin privileges (for creating ClusterRoles/ClusterRoleBindings and Kyverno)
- (Optional) Kyverno installed to enforce label policy

---
## Part 1 — Understanding the Basics

### 1.1 Inspect Default Service Accounts
Screenshot:

![Screenshot (88)](https://github.com/HammerTwo1/Homework_4_comp488_RBAC/blob/main/1.1.1.png)

### Question: Why doesn't the default service account have many permissions?

Answer: Principle of Least Privilege. If pods automatically had all rights, a compromised app could read secrets or modify resources. Admins explicitly create Roles for the exact permissions needed.



### 1.2 Test Current Permissions
```bash
kubectl run test-pod --image=bitnami/kubectl:latest --rm -it -- bash
# inside pod:
kubectl get pods
```
Error inside pod:

![Screenshot (88)](https://github.com/HammerTwo1/Homework_4_comp488_RBAC/blob/main/1.2.png)

Explanation: The kubectl running inside the pod uses the in-cluster credentials. Since that service account lacks list permission on pods, the API server forbids the action.

---
## Part 2 — Creating Namespace-Scoped Access
Files: Homework_4_comp488_RBAC/part2/*

### Apply resources
```bash
kubectl apply -f Homework_4_comp488_RBAC/part2/monitoring-namespace.yaml
kubectl apply -f Homework_4_comp488_RBAC/part2/monitoring-reader-sa.yaml
kubectl apply -f Homework_4_comp488_RBAC/part2/monitoring-read-role.yaml
kubectl apply -f Homework_4_comp488_RBAC/part2/monitoring-read-rolebinding.yaml
```

### Test
Run a debug pod that uses the monitoring-reader SA:

![Screenshot (88)](https://github.com/HammerTwo1/Homework_4_comp488_RBAC/blob/main/2.png)


---
## Part 3 — Cluster-Wide Access
Files: Homework_4_comp488_RBAC/part3/*

### Apply
```bash
kubectl apply -f Homework_4_comp488_RBAC/part3/log-collector-sa.yaml
kubectl apply -f Homework_4_comp488_RBAC/part3/log-reader-clusterrole.yaml
kubectl apply -f Homework_4_comp488_RBAC/part3/log-reader-binding.yaml
```

### Test
Run a debug pod that uses the log-collector SA:
![Screenshot (88)](https://github.com/HammerTwo1/Homework_4_comp488_RBAC/blob/main/3.png)
---
## Part 4 — Troubleshooting RBAC (broken -> fixed)
Files: Homework_4_comp488_RBAC/part4/deployment-manager-broken.yaml and deployment-manager-fixed.yaml

### Broken issues identified
1. The Role used `apiGroups: [""]` but `deployments` are in the `apps` API group. It's a mismatch that causes rules not to apply.
2. The RoleBinding was created in `default` namespace but was going to bind to a Role in `production`. RoleBinding and Role must be in the same namespace.

### Fixed YAML applied
```bash
kubectl apply -f Homework_4_comp488_RBAC/part4/deployment-manager-fixed.yaml
```
---

