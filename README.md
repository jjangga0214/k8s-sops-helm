# how to use sops and helm ?



```bash
helm upgrade --install -f ./k8s/values/staging/demo.yaml staging-demo ./k8s/demo --dry-run
```

1. Decrypt secrets and make temporary file.

2. helm

3. Remove decrypted temporary secret file.

```bash
sops -d k8s/demo/staging-secrets.yaml > temp-staging-secrets.yaml && \
helm upgrade --install -f ./k8s/demo/staging-values.yaml staging-demo ./k8s/demo && \
rm temp-staging-secrets.yaml
```
