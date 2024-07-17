### FluxCD

Installation FluxCD cli
```
curl -s https://fluxcd.io/install.sh | sudo bash
```

Installation FluxCD
```
flux install
```

Export your GitHub credentials to your terminal so that Flux may use them to bootstrap your GitHub repository
```
# Your GitHub username
export GITHUB_USER=xxxxxxxxxxxxxx

# Your GitHub PAT (Personal Access Token)
export GITHUB_TOKEN=xxxxxxxxxxxxxxxxxxx
```

Run the bootstrap for a repository on your personal GitHub account
```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=fluxcd-weavegitops \
  --branch=main \
  --path=clusters/eks-bottlerocket-cluster/dev \
  --personal
```

### Application

Put in `clusters/eks-bottlerocket-cluster/dev` k8s resources like `01-nginx-deployment.yaml`

Check resources
```
kubectl get deploy,po -n default
NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           63m

NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-86dcfdf4c6-d8xq9   1/1     Running   0          36m
pod/nginx-deployment-86dcfdf4c6-xlgs8   1/1     Running   0          36m
```

### Weave GitOps
Weave GitOps installation
```
curl --silent --location "https://github.com/weaveworks/weave-gitops/releases/download/v0.38.0/gitops-$(uname)-$(uname -m).tar.gz" | tar xz -C /tmp
sudo mv /tmp/gitops /usr/local/bin
gitops version
```

Weave GitOps Dashboard creation (`flux-system` namespace)
```
gitops create dashboard weave-gitops-dashboard-flux-system \
  --password=AeHoo+h1aingi9caih1a \
  --export > ./clusters/eks-bottlerocket-cluster/dev/02-weave-gitops-dashboard-flux-system.yaml
```

Weave GitOps Dashboard creation (`default` namespace)
```
gitops create dashboard weave-gitops-dashboard-default \
  --namespace=default \
  --password=AeHoo+h1aingi9caih1a \
  --export > ./clusters/eks-bottlerocket-cluster/dev/02-weave-gitops-dashboard-default.yaml
```

Weave GitOps Dashboard port forwarding (`flux-system` namespace)
```
kubectl -n flux-system port-forward svc/weave-gitops-dashboard-flux-system 9001:9001
```

Open in browser http://127.0.0.1:9001
- username: admin
- password: AeHoo+h1aingi9caih1a

Weave GitOps Dashboard port forwarding (`default` namespace)
```
kubectl -n default port-forward svc/weave-gitops-dashboard-default 9002:9001
```

Open in browser http://127.0.0.1:9002
- username: admin
- password: AeHoo+h1aingi9caih1a


### Capacitor

Capacitor resources
```
---
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: OCIRepository
metadata:
  name: capacitor
  namespace: flux-system
spec:
  interval: 12h
  url: oci://ghcr.io/gimlet-io/capacitor-manifests
  ref:
    semver: ">=0.1.0"
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: capacitor
  namespace: flux-system
spec:
  targetNamespace: flux-system
  interval: 1h
  retryInterval: 2m
  timeout: 5m
  wait: true
  prune: true
  path: "./"
  sourceRef:
    kind: OCIRepository
    name: capacitor
```

Capacitor port forwarding
```
kubectl -n flux-system port-forward svc/capacitor 9000:9000
```

Open in browser http://127.0.0.1:9000

### Flagger 
Add Flagger Helm repository:
```
helm repo add flagger https://flagger.app
```

Install Flagger’s Canary CRD:
```
kubectl apply -f https://raw.githubusercontent.com/fluxcd/flagger/main/artifacts/flagger/crd.yaml
```

Deploy Flagger for Istio:
```
helm upgrade -i flagger flagger/flagger \
--namespace=istio-system \
--create-namespace
--set crd.create=false \
--set meshProvider=istio \
--set metricsServer=http://prometheus:9090
```

Check chart
```
helm ls -n istio-system
NAME    NAMESPACE       REVISION        UPDATED                                         STATUS          CHART           APP VERSION
flagger istio-system    1               2024-07-18 01:38:43.833644952 +0300 EEST        deployed        flagger-1.37.0  1.37.0
```

Flagger comes with a Grafana dashboard made for canary analysis. Install Grafana with Helm:
```
helm upgrade -i flagger-grafana flagger/grafana \
--set url=http://prometheus:9090
```
Run the port forward command:
```
kubectl -n flux-system port-forward svc/flagger-grafana 3000:80
```

http://localhost:3000

### GitRepository and Kustomization
```
kubectl get gitrepository -n flux-system
NAME          URL                                                AGE   READY   STATUS
flux-system   ssh://git@github.com/iliusa77/fluxcd-weavegitops   17h   True    stored artifact for revision 'main@sha1:53765784d01708c884b47c876fdd11656f193d7a'

kubectl get kustomization -n flux-system
NAME          AGE   READY   STATUS
capacitor     13h   True    Applied revision: v0.4.2@sha256:1e72940be8383cd5054e3efcff8ec36f27e33959a4b940fc4b956c932083578b
flux-system   17h   True    Applied revision: main@sha1:53765784d01708c884b47c876fdd11656f193d7a
```