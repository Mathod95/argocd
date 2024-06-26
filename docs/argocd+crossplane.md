
# [provider-argocd](https://github.com/crossplane-contrib/provider-argocd/blob/main/README.md)

## Overview

`provider-argocd` is the Crossplane infrastructure provider for
[Argo CD](https://argo-cd.readthedocs.io/). The provider that is built from the source code
in this repository can be installed into a Crossplane control plane and adds the
following new functionality:

* Custom Resource Definitions (CRDs) that model Argo CD resources
* Controllers to provision these resources in Argo CD based on the users desired
  state captured in CRDs they create
* Implementations of Crossplane's portable resource
  abstractions, enabling
  Argo CD resources to fulfill a user's general need for Argo CD configurations

## Getting Started and Documentation

Follow the [official docs](https://crossplane.io/docs/master/getting-started/install-configure.html#install-crossplane) to install crossplane, then these steps to get started with `provider-argocd`.

### Optional: Start a local Argo CD server
```bash
kind create cluster

kubectl create ns argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
### Create a new Argo CD user

Follow the steps in the [official documentation](https://argoproj.github.io/argo-cd/operator-manual/user-management/) to create a new user `provider-argcod`:

```bash
kubectl patch configmap/argocd-cm \
  -n argocd \
  --type merge \
  -p '{"data":{"accounts.provider-argocd":"apiKey"}}'

kubectl patch configmap/argocd-rbac-cm \
  -n argocd \
  --type merge \
  -p '{"data":{"policy.csv":"g, provider-argocd, role:admin"}}'
```

### Create an API Token

*Note:* The following steps require the [kubectl-view-secret](https://github.com/elsesiy/kubectl-view-secret) plugin and [jq](https://stedolan.github.io/jq/) to be installed.

Get the admin passwort via `kubectl`
```bash
set ARGOCD_ADMIN_SECRET $(kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo)
```

Port forward the Argo CD api to the host:
```bash
kubectl -n argocd port-forward svc/argocd-server 8443:443
```

Create a session JWT for the admin user at the Argo CD API. *Note:* You cannot use this token directly, because it will expire.
```bash
set ARGOCD_ADMIN_TOKEN $(curl -s -X POST -k -H "Content-Type: application/json" --data '{"username":"admin","password":"'$ARGOCD_ADMIN_SECRET'"}' https://localhost:8080/api/v1/session | jq -r .token)
```

Create an API token without expiration that can be used by `provider-argocd`
```bash
set ARGOCD_PROVIDER_USER "provider-argocd"

set ARGOCD_TOKEN $(curl -s -X POST -k -H "Authorization: Bearer $ARGOCD_ADMIN_TOKEN" -H "Content-Type: application/json" https://localhost:8080/api/v1/account/$ARGOCD_PROVIDER_USER/token | jq -r .token)
```

### Setup crossplane provider-argocd

Install provider-argocd:
```bash
cat << EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-argocd
spec:
  package: xpkg.upbound.io/crossplane-contrib/provider-argocd:v0.2.0
EOF
```
Create a kubernetes secret from the JWT so `provider-argocd` is able to connect to Argo CD:
```bash
kubectl create secret generic argocd-credentials -n crossplane-system --from-literal=authToken="$ARGOCD_TOKEN"
```

Configure a `ProviderConfig` with `serverAddr` pointing to an Argo CD instance:
```bash
cat << EOF | kubectl apply -f -
apiVersion: argocd.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: argocd-provider
spec:
  serverAddr: argocd-server.argocd.svc:443
  insecure: true
  plainText: false
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: argocd-credentials
      key: authToken
EOF
```