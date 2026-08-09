# AGENTS.md

Config-only repo for a self-hosted [searxng](https://docs.searxng.org/) instance. No code, no build, no tests, no lint — nothing to run locally. Do not add build/test tooling.

## Layout

- `data/settings.yml` — searxng runtime config, mounted into the container at `/etc/searxng/settings.yml` (hostPath `/mnt/workspaces/searxng/data`, i.e. the host where the cluster runs, not this macOS path).
- `k8s/searxng-deployment.yaml` — Deployment + ClusterIP Service + internal Ingress (`searxng.localhost`). Uses image `searxng/searxng:latest`, port 8080.
- `k8s/searxng-secret.yaml` — Secret with `SEARXNG_SECRET` and `SEARXNG_BASE_URL`.
- `k8s/searxng-external-ingress.yaml` — public Ingress for `search.furseal.net`.

`.gitignore` excludes `data/`, `k8s/*-secret.yaml`, `k8s/*-external-ingress.yaml`. Repo is not yet a git repo; if `git init` is run, those are expected to stay untracked.

## Deploy

```sh
kubectl apply -f k8s/searxng-deployment.yaml
kubectl apply -f k8s/searxng-secret.yaml
kubectl apply -f k8s/searxng-external-ingress.yaml
```

## Gotchas

- `data/settings.yml` enables the JSON output format **on purpose** (`formats: [html, json]`, marked CRITICAL in the file) so AI agents can hit the API without 403s. Do not remove `json`. `SEARCH_FORMATS: "html,json"` is also set in the Deployment.
- `server.secret_key` in `settings.yml` and `SEARXNG_SECRET` in the Secret are both set; the env var wins in the container (searxng image maps env over settings).
- Secret values and the public domain live only in the gitignored files — don't print or commit them, don't "tidy up" by committing.
- hostPath mount means config changes take effect after a pod restart, not on edit.
