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

Weave GitOps Dashboard creation
```
gitops create dashboard ww-gitops \
  --password=AeHoo+h1aingi9caih1a \
  --export > ./clusters/eks-bottlerocket-cluster/dev/02-weave-gitops-dashboard.yaml
```

Weave GitOps Dashboard port forwarding
```
kubectl -n flux-system port-forward svc/ww-gitops-weave-gitops 9001:9001
```

Open in browser http://127.0.0.1:9001

username: admin
password: AeHoo+h1aingi9caih1a

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
