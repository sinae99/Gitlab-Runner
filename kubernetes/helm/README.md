# Helm

Installs GitLab Runner using the [official chart](https://docs.gitlab.com/runner/install/kubernetes.html).

## Configure

```bash
cp values.example.yaml values.yaml
```

Edit `values.yaml`:

| Key | What it is |
|-----|------------|
| `gitlabUrl` | Your GitLab URL |
| `runnerToken` | Runner auth token from GitLab → **Settings → CI/CD → Runners** → New runner |
| `runners.config` → `name`, `tag_list` | Runner identity and job tags |
| `runners.config` → `[runners.kubernetes].image` | Default image for **CI job pods** (not the runner itself) |
| `runners.config` → `service_account` | Job pod SA — `gitlab-runner-jobs` (no API access) |
| `rbac.rules` | Minimal permissions — see [`../manifests/rbac/README.md`](../manifests/rbac/README.md) |

`values.yaml` is gitignored — never commit runner tokens.

## Install

Create the job service account first (Helm chart does not create it):

```bash
kubectl create namespace gitlab-runner --dry-run=client -o yaml | kubectl apply -f -
kubectl apply -f ../manifests/rbac/serviceaccount-jobs.yaml
```

Install or upgrade the chart:

```bash
helm repo add gitlab https://charts.gitlab.io
helm repo update

helm upgrade --install gitlab-runner gitlab/gitlab-runner \
  --namespace gitlab-runner \
  --create-namespace \
  -f values.yaml
```

Pass the token at the CLI instead of storing it in a file:

```bash
helm upgrade --install gitlab-runner gitlab/gitlab-runner \
  -n gitlab-runner -f values.yaml \
  --set runnerToken="$RUNNER_TOKEN"
```

## Verify

```bash
kubectl get pods -n gitlab-runner
kubectl logs -n gitlab-runner -l app=gitlab-runner
```

Runner should appear online in GitLab with tags `k3s`, `kaniko`, `docker`.

## Container image builds (Kaniko)

Job pods run **non-privileged** (`privileged = false`). They build OCI images with [Kaniko](https://github.com/GoogleContainerTools/kaniko) — **not** Docker-in-Docker. No privileged daemon is required.

Use the **debug** Kaniko image (includes a shell — required by GitLab CI):

```yaml
stages:
  - build

build:
  tags: [k3s, kaniko]   # match runner tag_list
  image:
    name: gcr.io/kaniko-project/executor:v1.23.2-debug
    entrypoint: [""]
  script:
    - /kaniko/executor
      --context "${CI_PROJECT_DIR}"
      --dockerfile "${CI_PROJECT_DIR}/Dockerfile"
      --destination "${CI_REGISTRY_IMAGE}:${CI_COMMIT_SHORT_SHA}"
```

Registry push requires Kaniko auth:

```yaml
  before_script:
    - mkdir -p /kaniko/.docker
    - |
      echo "{\"auths\":{\"${CI_REGISTRY}\":{\"auth\":\"$(printf "%s:%s" "${CI_REGISTRY_USER}" "${CI_REGISTRY_PASSWORD}" | base64 | tr -d '\n')\"}}}" \
        > /kaniko/.docker/config.json
```

How a job pod is structured:

```
GitLab job → runner creates a pod in gitlab-runner namespace
  ├── build container (kaniko executor)  → builds image, pushes to registry
  └── helper container                   → git clone, cache, artifacts
```

Test pipeline: [`../../examples/kaniko-test/`](../../examples/kaniko-test/).
