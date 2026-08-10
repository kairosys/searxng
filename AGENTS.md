# AGENTS.md

Config-only repo for a self-hosted [searxng](https://docs.searxng.org/) instance. No code, no build, no tests, no lint — nothing to run locally. Do not add build/test tooling.

## Layout

- `k8s/searxng-deployment.yaml` — Deployment + ClusterIP Service + internal Ingress (`searxng.localhost`). Uses image `searxng/searxng:latest`, port 8080. Config delivered via ConfigMap (not hostPath).
- `k8s/searxng-configmap.yaml` — ConfigMap containing `settings.yml` and `limiter.toml`.
- `k8s/searxng-secret.yaml` — Secret with `SEARXNG_SECRET` and `SEARXNG_BASE_URL` (gitignored).
- `k8s/searxng-external-ingress.yaml` — public Ingress for `search.furseal.net` (gitignored).

`.gitignore` excludes `data/`, `k8s/*-secret.yaml`, `k8s/*-external-ingress.yaml`. Don't `git add -f` the ignored files.

## Deploy

```sh
kubectl apply -f k8s/searxng-configmap.yaml
kubectl apply -f k8s/searxng-deployment.yaml
kubectl apply -f k8s/searxng-secret.yaml
kubectl apply -f k8s/searxng-external-ingress.yaml
```

Order matters: ConfigMap must exist before the Deployment mounts it.

## Gotchas

- JSON format (`formats: [html, json]`) is enabled **on purpose** so AI agents can query the API without 403s. Do not remove `json`.
- `SEARXNG_SECRET` (via `envFrom` on the Deployment) is the sole source of the server secret — don't add a `server.secret_key` back to the ConfigMap.
- `SEARCH_FORMATS` env var was removed from the Deployment. Formats are set only in the ConfigMap's `settings.yml`.
- Secret values and the public domain live only in the gitignored files — don't print or commit them.
- ConfigMap changes require a pod restart to take effect.
- Timezone is set to `Asia/Hong_Kong` via `TZ` env var.
