# openshift4-gitops
Testing OpenShift 4 Cluster Configuration With Gitops

## Installing ArgoCD on OpenShift 4

These steps will hopefully be replaced by an ArgoCD operator on OperatorHub in the near future.

```bash
oc new-project argocd
# Apply modified ArgoCD (added --insecure to disable in-container SSL) from https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
oc apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
oc create route passthrough --service=argocd-server
#oc apply -n argocd -f argocd.yaml
#oc create route edge --service=argocd-server

# Get the argoCD 'admin' password:
kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o name | cut -d'/' -f 2
# Login:
argocd login argocd-server-argocd.apps.dgoodwin-dev.new-installer.openshift.com:443
# Change the ArgoCD password:
argocd account update-password
argocd cluster add argocd/api-dgoodwin-dev-new-installer-openshift-com:6443/system:admin --in-cluster
```

# Configuring OpenShift 4

## Machine Sets

Leveraging Gitops for MachineSets would allow customers to commit their desired cluster capacity and spread across AZs.

MachineSets rely heavily on the "InfraID" for your cluster, a value generated by the installer at install time.

A standard OpenShift 4 cluster with 3 compute nodes in us-east-1 comes with 6 MachineSets, one per AZ (in my account), with only three of them scaled to 1 replicas. Each MachineSet references the generated InfraID roughly 9 times:

 - MachineSet Name
 - Selector
 - IAM Instance Profile (generated by Installer presumably)
 - Security Group Name
 - Subnet
 - AWS Tags

To check a variant of these MachineSets into git and then apply to a provisioned cluster we will need some kind of templating to feed in the infraID. How can we do this?

Ksonnet is dead. Kustomize does not think you should be doing anything based on parameters. I believe Helm requires installing another component. (Tiller) Can we make Kustomize work somehow or should we explore adding another [Parameter Overrides](https://argoproj.github.io/argo-cd/user-guide/parameters/) mechanism to Argo?







