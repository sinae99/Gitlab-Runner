# Manifests

Raw Kubernetes YAML for kubectl. Uses the **kubernetes executor** — CI jobs run as pods in the cluster.

## Configure

```bash
cp vars.env.example vars.env
```

Edit **`vars.env`**:

| Variable | What it is |
|----------|------------|
| `GITLAB_URL` | Your GitLab URL |
| `REGISTRATION_TOKEN` | Runner auth token from GitLab → **Settings → CI/CD → Runners** |
| `RUNNER_NAME` | Name shown in GitLab UI |
| `RUNNER_TAGS` | Comma-separated tags (`k3s,kaniko,docker`) |
| `JOB_IMAGE` | Default image for **CI job pods** (e.g. `alpine:3.20`) |
| `JOB_PRIVILEGED` | `false` for Kaniko builds (no DinD) |
| `JOB_SERVICE_ACCOUNT` | SA for job pods — use `gitlab-runner-jobs` (no API access) |

`vars.env` is gitignored — never commit runner tokens.

Runner container image is pinned in `deployment.yaml` (`gitlab/gitlab-runner:alpine-v18.4.0`).

RBAC details: see [`rbac/README.md`](rbac/README.md).

## Layout

```
manifests/
├── vars.env.example
├── namespace.yaml
├── pvc.yaml
├── deployment.yaml
└── rbac/             ← see rbac/README.md
    ├── serviceaccount.yaml        (runner manager)
    ├── serviceaccount-jobs.yaml   (CI jobs, no permissions)
    ├── role.yaml
    └── rolebinding.yaml
```

## Apply

```bash
kubectl apply -k .
```

## Container image builds (Kaniko)

CI jobs run **non-privileged** with `JOB_PRIVILEGED=false`. Use Kaniko in `.gitlab-ci.yml` to build images — see [`../helm/README.md`](../helm/README.md) and [`../../examples/kaniko-test/`](../../examples/kaniko-test/).

## Verify

```bash
kubectl get all -n gitlab-runner
kubectl logs -n gitlab-runner deployment/gitlab-runner -f
```

## Local cluster (k3s / k3d)

Works out of the box — PVC uses the cluster default storage class.

```bash
# k3d example
k3d cluster create gitlab-test
kubectl apply -k .
```
