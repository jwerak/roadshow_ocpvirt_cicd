# OCP Virtualization Roadshow - CICD extension

Here are configs and playbooks for extending OCP Virtualization Roadshow with GitOps exercises.

## Bookbag update

Update bookbag image

```bash
podman build -t quay.io/jwerak/bookbag-roadshow .
podman push quay.io/jwerak/bookbag-roadshow
```

Update image of DeploymentConfig `bookbag` in namespace `guid-bookbag`:

```bash
oc patch deploymentconfig bookbag --type='merge' --patch='{"spec":{"triggers": [{"type": "ConfigChange"}],"template":{"spec":{"containers":[{"name": "terminal", "image":"quay.io/jwerak/bookbag-roadshow"}]}}}}'
```

## Prerequisites for each cluster

Install GitOps operator and instance of ArgoCD

```bash
echo | oc apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: openshift-gitops-operator
EOF

echo | oc apply -f - <<EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
EOF

echo | oc apply -f - <<EOF
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
echo | oc apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: pipelines
EOF
```

## ArgoCD permissions

```bash
echo | oc apply -f - <<EOF
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

## Bulk apply

Simple ansible script was created to:

- create per host setup script
- copy script to jumphost
- copy script from jumphost to bastion
  - `sudo ssh root@192.168.123.100`
- execute script on bastion

### Setup clusters

**Prepare inventory variables** in file `ansible/host_vars/localhost`.
Source data are in demo.redhat.com service.

Example structure:

```yaml
roadshow_ocpvirt:
  - bastion_public_hostname: hypervisor.r8d4e.dynamic.redhatworkshops.io
    bastion_ssh_password: XXXXXXXXXX
    bastion_ssh_user_name: lab-user
    equinix_metal_project_id: 99eb5228-368f-4e00-9f40-944a1df99257
    equinix_metal_project_name: roadshow-ocpvirt-r8d4e
    guid: r8d4e
    ocp_password: XXXXXXXXXX
    ocp_url: https://console-openshift-console.apps.r8d4e.dynamic.redhatworkshops.io
    preinstall_operators: true
    vcenter_password: e.PAWH:DWMD9
    vcenter_user: roadshow-ocpvirt-r8d4e
  - bastion_public_hostname: hypervisor.fr5ze.dynamic.redhatworkshops.io
    bastion_ssh_password: XXXXXXXXXX
    bastion_ssh_user_name: lab-user
    equinix_metal_project_id: 14d33d79-3587-4ab9-96e2-c1bf92471f23
    equinix_metal_project_name: roadshow-ocpvirt-fr5ze
    guid: fr5ze
    ocp_password: XXXXXXXXXX
    ocp_url: https://console-openshift-console.apps.fr5ze.dynamic.redhatworkshops.io
    preinstall_operators: true
    vcenter_password: XXXXXXXXXX
    vcenter_user: roadshow-ocpvirt-fr5ze
```

**Create per cluster inventory** using command:

```bash
ansible-navigator run --ee false generate-inventory.yml
```

Setup clusters using command:

```bash
ansible-navigator run --ee false setup-workshop.yml
```
