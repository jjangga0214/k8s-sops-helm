# how to use sops and helm ?


```bash
helm upgrade --install -f ./k8s/demo/staging-values.yaml staging-demo ./k8s/demo --dry-run
```

```bash
sops -d k8s/demo/staging-secrets.yaml > temp-staging-secrets.yaml && \
helm upgrade --install -f ./k8s/demo/staging-values.yaml staging-demo ./k8s/demo && \
rm temp-staging-secrets.yaml
```
