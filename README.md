# Kubernetes RBAC Homework Submission

## Contents
- Part1: Basics & Tests
- Part2: Namespace-Scoped Access (monitoring-reader)
- Part3: Cluster-Wide Access (log-collector)
- Part4: Troubleshooting RBAC (broken -> fixed)
- Part5: Real-World RBAC Design
- Bonus: Label-enforced pod creation (Kyverno)
- YAML files for all resources in `manifests/`
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
Files: manifests/part2/*

### Apply resources
```bash
kubectl apply -f manifests/part2/monitoring-namespace.yaml
kubectl apply -f manifests/part2/monitoring-reader-sa.yaml
kubectl apply -f manifests/part2/monitoring-read-role.yaml
kubectl apply -f manifests/part2/monitoring-read-rolebinding.yaml
```

### Test
Run a debug pod that uses the monitoring-reader SA:

![Screenshot (88)](https://github.com/HammerTwo1/Homework_4_comp488_RBAC/blob/main/2.png)

Expected forbidden output for delete:
```
Error from server (Forbidden): pods "<name>" is forbidden: User "system:serviceaccount:monitoring:monitoring-reader" cannot delete resource "pods" in API group "" in the namespace "monitoring"
```
