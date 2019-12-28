# how to use sops and helm?

## steps

1. Decrypt secrets and make temporary files.

2. Use helm

3. Remove decrypted temporary secret file.

```bash
sops -d k8s/demo/staging-secrets.yaml > decrypted-staging-secrets.yaml && \
helm upgrade --install -f ./k8s/values/staging/demo.yaml staging-demo ./k8s/demo && \
rm decrypted-staging-secrets.yaml
```

## quick check

```bash
helm upgrade --install -f ./k8s/values/staging/demo.yaml staging-demo ./k8s/demo --dry-run
```
