# Kaniko build test

Minimal pipeline to verify your Kubernetes runner can build container images (no Docker-in-Docker).

Copy these files into a GitLab project, commit, and push. The job runs on runners tagged `k3s` and `kaniko`.

```bash
cp .gitlab-ci.yml Dockerfile /path/to/your/gitlab/project/
```

The first run uses `--no-push` (no registry required). To push to GitLab Container Registry, see [`../../kubernetes/helm/README.md`](../../kubernetes/helm/README.md).
