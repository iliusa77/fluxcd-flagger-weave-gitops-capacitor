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

### Weave GitOps
Installation
```
curl --silent --location "https://github.com/weaveworks/weave-gitops/releases/download/v0.38.0/gitops-$(uname)-$(uname -m).tar.gz" | tar xz -C /tmp
sudo mv /tmp/gitops /usr/local/bin
gitops version
```

Dashboard creation
```
gitops create dashboard ww-gitops \
  --password=password \
  --export > ./clusters/eks-bottlerocket-cluster/dev/components/02-weave-gitops-dashboard.yaml
```

```
gitops beta run ./clusters/eks-bottlerocket-cluster/dev/components --no-session \
    --port-forward namespace=default,resource=svc/backend,port=8080:80
```
