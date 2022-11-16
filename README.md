# Infra and app

For demonstrating basics of Flux with OCI and Cosign.


# Steps

## 0. Install Flux (without bootstrap)

Use VSCode


## 1. Deploy podinfo with GitRepository and Kustomization
```
flux create source git podinfo --url=https://github.com/stefanprodan/podinfo --branch=master --interval=30s
flux create kustomization podinfo --target-namespace=default --source=podinfo --path="./kustomize" --prune=true --interval=30s --timeout=10s
```

```
kubectl port-forward service/podinfo 9898:9898
```

### Show Stuff
### Delete


## 2. GitOps to manage podinfo

```
flux create source git podinfo --url=https://github.com/stefanprodan/podinfo --branch=master --interval=30s --export 
flux create kustomization podinfo --target-namespace=default --source=podinfo --path="./kustomize" --prune=true --interval=30s --timeout=10s --export
```

## 3. OCIRepository for bootstrap

flux create source oci flux-oci-demo --url=oci://ghcr.io/juozasg/flux-oci-demo --tag=latest --interval=30s --secret-ref ghcr-auth --namespace=flux-system --export
flux create kustomization flux-oci-demo --target-namespace=default --source=OCIRepository/flux-oci-demo --path="./clusters/demo" --prune=true --interval=30s --timeout=10s --namespace=flux-system --target-namespace=flux-system --export

```
git add .
git commit -m "GitOps bootstrap example"
git push
```

### Everything is in Git, but not reconciling. Create OCI and kustomization manually


### Create auth secret GHCR
```
flux create secret oci ghcr-auth --url=ghcr.io  --username=$env:GITHUB_USER --password=$env:GITHUB_TOKEN
```

## 4. Setup cosign
```
cd ..
echo S3cretPassw0rd | ./cosign.exe generate-key-pair github://juozasg/flux-oci-demo
```

kubectl -n flux-system create secret generic cosign-pub --from-file=cosign.pub=cosign.pub

update OCIRepository/flux-oci-demo
```
  verify:
    provider: cosign
    secretRef:
      name: cosign-pub
```
