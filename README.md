# OCP Virtualization Roadshow - CICD extension

Here are configs and playbooks for extending OCP Virtualization Roadshow with GitOps exercises.

## Prerequisites for each cluster

Install GitOps operator and instance of ArgoCD

```bash
oc create ns openshift-gitops-operator

echo | oc create -f - <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
EOF

echo | oc create -f - <<EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: latest
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

# Watch installation
watch oc get pods -n openshift-gitops-operator
watch oc get pods -n openshift-gitops
```

Create project for pipelines

```bash
oc new-project pipelines
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

- [application-pipelines.yml](./gitops/application-pipelines.yml)
- [application-virtualization.yml](./gitops/application-virtualization.yml)
