# Infra and app

For demonstrating basics of Flux with OCI and Cosign.


# Steps

## 0. Install Flux (without bootstrap)

Using VSCode


## 1. Deploy podinfo with GitRepository and Kustomization
```
flux create source git podinfo --url=https://github.com/stefanprodan/podinfo --branch=master --interval=30s
flux create kustomization podinfo --target-namespace=default --source=podinfo --path="./kustomize" --prune=true --interval=30s --timeout=10s
```

```
kubectl port-forward service/podinfo 9898:9898
```


## 2. GitOps for podinfo


### Export YAML
```
flux create source git podinfo --url=https://github.com/stefanprodan/podinfo --branch=master --interval=30s --export 
flux create kustomization podinfo --target-namespace=default --source=podinfo --path="./kustomize" --prune=true --interval=30s --timeout=10s --export
```

Add YAML to git `app.yaml`

## 3. OCIRepository for bootstrap

## Export YAMLs
```
flux create source oci flux-oci-demo --url=oci://ghcr.io/juozasg/flux-oci-demo --tag=latest --interval=30s --secret-ref ghcr-auth --namespace=flux-system --export
flux create kustomization flux-oci-demo --target-namespace=default --source=OCIRepository/flux-oci-demo --path="./clusters/demo" --prune=true --interval=30s --timeout=10s --namespace=flux-system --target-namespace=flux-system --export
```

Add YAML to git `sync.yaml`

## Push everything 
```
git add .
git commit -m "GitOps bootstrap example"
git push
```

### Everything is in Git, but not reconciling. Create OCIRepository and Kustomization manually

`kubectl apply -f sync.yml`


### Create auth secret for GitHub Container Registry
```
flux create secret oci ghcr-auth --url=ghcr.io  --username=$env:GITHUB_USER --password=$env:GITHUB_TOKEN
```

## 4. Setup cosign

### Generate keys and upload to GitHub

```
cd ..
echo S3cretPassw0rd | ./cosign.exe generate-key-pair github://juozasg/flux-oci-demo
```

### Tell Flux about cosign public key

`kubectl -n flux-system create secret generic cosign-pub --from-file=cosign.pub=cosign.pub`


### Tell Flux to verify OCIRepository using public key

update OCIRepository/flux-oci-demo:
```
  verify:
    provider: cosign
    secretRef:
      name: cosign-pub
```
