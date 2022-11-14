# Infra and app

For demonstrating basics of Flux with OCI and Cosign


# Steps

## 0. Install Flux (without bootstrap)


flux create source git podinfo --url=https://github.com/stefanprodan/podinfo --branch=master --interval=30s --export > ./clusters/demo/podinfo-source.yaml
flux create kustomization podinfo --target-namespace=default --source=podinfo --path="./kustomize" --prune=true --interval=5m --export > ./clusters/demo/podinfo-kustomization.yaml


## 1. Deploy podinfo with GitRepository and Kustomization

