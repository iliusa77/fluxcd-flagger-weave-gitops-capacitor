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