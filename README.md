# ACS Installation

## Via Kustomize
To install the operator and instance using kustomize, first install the operator as follows:
```
oc apply -k advanced-cluster-security-operator/operator/overlays/latest
```

Once the operator is running in openshift-operators you can then install an instance with the following command:

```
oc apply -k advanced-cluster-security-operator/instance/overlays/default
```

This will create a stackrox namespace and install central as well as a the securedcluster. Stackrox requires a cluster-init bundle to be deployed to link the two, a job will run to do this. If for some reason you do not see data in ACS and no cluster appears in the Settings check the job logs to see what happened.

## Log into ACS as Admin

Get admin password:
```
oc -n stackrox get secret central-htpasswd -o go-template='{{index .data "password" | base64decode}}'
```

Get Route:
```
oc get route -n stackrox central -o jsonpath='{"https://"}{.spec.host}{"\n"}'
```

## Configure ACS Authentication

You can invoke the playbook by passing either username/password or API token to the playbook. To use it with the default Central username/password:

`ansible-playbook acs.yaml -e username=admin -e password=XXXXXX -e api_endpoint=central-stackrox.apps.kaby.demolab.local`

The actual role mappings can be configured under `vars/vars.yaml` -> `role_mappings:`

To invoke it with an API token created in ACS:

`ansible-playbook acs.yaml -e api_token=XXXXXXX -e api_endpoint=central-stackrox.apps.kaby.demolab.local`

## Synce the manifests
https://docs.openshift.com/acs/3.72/configuration/enable-offline-mode.html#update-kernel-support-packages_enable-offline-mode

https://docs.openshift.com/acs/3.72/configuration/enable-offline-mode.html#download-kernel-support-package_enable-offline-mode

https://docs.openshift.com/acs/3.72/configuration/enable-offline-mode.html#download-scanner-definitions_enable-offline-mode


# GitOps Installation

Go with `latestst` channel for now.
```
oc apply -k openshift-gitops-operator/overlays/<channel>
oc apply -k openshift-gitops-operator/overlays/latest

oc apply -k openshift-gitops-operator/overlays/preview
oc apply -k openshift-gitops-operator/overlays/stable
oc apply -k openshift-gitops-operator/overlays/gitops-1.5
oc apply -k openshift-gitops-operator/overlays/gitops-1.6
```

Argo CD cannot map users to specific roles if they have a direct ClusterRoleBinding role. You can manually change the role as role:admin on SSO through OpenShift.

1. Create a group named cluster-admins.
```
oc adm groups new cluster-admins
```

2. Add the user to the group.
```
oc adm groups add-users cluster-admins <USER>
```

3. Apply the cluster-admin ClusterRole to the group:

```
oc adm policy add-cluster-role-to-group cluster-admin cluster-admins
```

## Log into ArgoCD as Admin (not needed, info only)
```
   ARGOCD_ADMIN_PASSWORD=$(oc -n openshift-gitops get secret openshift-gitops-cluster -o jsonpath="{.data.*}" | base64 -d)

    ARGOCD_ROUTE=$(oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}')

    argocd login $ARGOCD_ROUTE --username admin --password $ARGOCD_ADMIN_PASSWORD
```


# NATS
1. Allow NATS to run as root: `oc adm policy add-scc-to-user privileged system:admin:default`
2. Create namespace and give permissions to namespace for argocd-app-controller: `oc create ns nats-deployment` and  `oc adm policy add-role-to-user admin system:serviceaccount:openshift-gitops:openshift-gitops-argocd-application-controller -n nats-deployment`
3. Deploy NATS via OpenShift GitOps `oc apply -f application-nats.yaml`

