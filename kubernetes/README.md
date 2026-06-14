# Kubernetes

Two ways to run GitLab Runner on a cluster:

| Path | Use when |
|------|----------|
| [`helm/`](helm/) | Helm |
| [`manifests/`](manifests/) | kubectl |

Both use the **kubernetes executor** — CI jobs spawn as pods.  

The **job image** (`alpine:latest` by default) is what your `.gitlab-ci.yml` runs in unless a job specifies otherwise. It is **not** the runner container image.

kaniko image for tasks that requires 'docker build' :  gcr.io/kaniko-project/executor:v1.23.2-debug

- Helm → `helm/values.yaml`
- Manifests → `manifests/vars.env`
