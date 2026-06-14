# GitLab Runner

Deploy GitLab Runner:

| Path | Method | Config file |
|------|--------|-------------|
| [`docker/`](docker/) + [`ansible/`](ansible/) | via Docker Compose (automated) | `docker/vars.yml` |
| [`kubernetes/helm/`](kubernetes/helm/) | Cluster via Helm | `kubernetes/helm/values.yaml` |
| [`kubernetes/manifests/`](kubernetes/manifests/) | Cluster via kubectl | `kubernetes/manifests/vars.env` |

Each folder has its own README with setup steps.

