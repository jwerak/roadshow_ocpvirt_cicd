# OCP Virtualization Roadshow - CICD extension

Here are configs and playbooks for extending OCP Virtualization Roadshow with GitOps exercises.

## Prerequisites for each cluster

```bash
oc project pipelines
```

## ArgoCD permissions

```bash
echo | oc create -f - <<EOF
---
apiVersion: user.openshift.io/v1
kind: Group
metadata:
  name: cluster-admins
users:
- admin
EOF

oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n openshift-cnv
oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n openshift-operators
```

## Examples

Application CRs for ArgoCD. To be done manually as part of the assignment:

- [application-pipelines.yml](./application-pipelines.yml)
- [application-virtualization.yml](./application-virtualization.yml)
