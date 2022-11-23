1. Run the `oc apply -f create-namespace.yaml` file to create the application namespace
2. Install the GitOps operator via `oc apply -f install-gitops.yaml`
3. Allow NATS to run as root: `oc adm policy add-scc-to-user privileged system:admin:default`
4. Log into ArgoCD:
   ``` bash
   ARGOCD_ADMIN_PASSWORD=$(oc -n openshift-gitops get secret openshift-gitops-cluster -o jsonpath="{.data.*}" | base64 -d)

    ARGOCD_ROUTE=$(oc -n openshift-gitops get route openshift-gitops-server -o jsonpath='{.spec.host}')

    argocd login $ARGOCD_ROUTE --username admin --password $ARGOCD_ADMIN_PASSWORD
    ```
5. Deploy NATS via OpenShift GitOps `oc apply -f install-nats`


