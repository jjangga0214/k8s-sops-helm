# How to use sops and helm?

This example explains how to feed `.env`(`dotenv`) to k8s secrets with `helm` and [`sops`](https://github.com/mozilla/sops). But other formats(e.g. `yaml`, `json`, etc) can be used (Refer to `sops` docs for more detail).

There are `k8s/staging.env` and `k8s/prod.env`.
Both of them are `dotenv` files, and encrypted by `sops`(using `PGP`).
And there's a helm chart `demo`(`k8s/demo`).

_(Though there are `k8s/staging.decrypted.env` and `k8s/prod.decrypted.env` for your convenience, they(raw secret) are not to be committed by version control in real situation)_

When you want to deploy `demo` to `production` environment(e.g. cluster or namespace), for instance,
you take 3 steps below.

## Steps

<!-- markdownlint-disable ol-prefix -->

1. Decrypt secret(dotenv file) and make a temporary file.

```bash
sops -d k8s/prod.env > k8s/prod.decrypted.env
```

2. Use `--set-file` option of `helm`. In this example, `--set-file` create helm value named `dotenv`, which is not specified in `./k8s/values/prod/demo.yaml`.

```bash
helm upgrade --install \
  -f ./k8s/values/prod/demo.yaml \
  --set-file dotenv=./k8s/prod.decrypted.env \
  staging-demo ./k8s/demo \
```

`k8s/demo/templates/secrets.yaml` create `.env` by `.Values.dotenv`, which is populated by `--set-file`.

```yaml
data:
  .env: { { .Values.dotenv | b64enc } }
```

`k8s/demo/templates/deployment.yaml` create a volume from secret and mount `.env` to `/secret/.env`

```yaml
volumeMounts:
  - mountPath: "/secrets/"
```

Therefore application can read environment variables from `/secrets/.env`.

3. Remove temporarily decrypted secret file.

```bash
rm prod.decrypted.env
```

<!-- markdownlint-enable ol-prefix -->

### Quick Check

To see the rendered manifest files without actually applying to cluster, run with `--dry-run`.

```bash
helm upgrade --install \
  -f ./k8s/values/staging/demo.yaml \
  --set-file dotenv=./k8s/staging.decrypted.env \
  staging-demo ./k8s/demo \
  --dry-run
```

This will show base64-encoded `.env`. Of course, k8s will write decrypted content to volume.

```yaml
data:
  .env: SEVMTE89IndvcmxkIgpIST0idGhlcmUi
```

To decrypt for verification, you can run this.

```bash
echo "SEVMTE89IndvcmxkIgpIST0idGhlcmUi" | base64 -d
```

### One liner

You can copy and paste this.

```bash
sops -d k8s/.env.prod > prod.decrypted.env \
&& \
helm upgrade --install \
  -f ./k8s/values/prod/demo.yaml \
  --set-file dotenv=./k8s/prod.decrypted.env \
  staging-demo ./k8s/demo \
&& \
rm prod.decrypted.env
```
