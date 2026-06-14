# Docker

Compose template and runner variables for the VM deployment.

## Files

| File | Purpose |
|------|---------|
| `vars.yml` | Runner config — edit before deploy |
| `docker-compose.yml.j2` | Compose template (rendered by Ansible) |

## Configure

Edit `vars.yml`:

- `gitlab_url` — your GitLab URL
- `gitlab_registration_token` — from GitLab → Settings → CI/CD → Runners
- `gitlab_runner_name`, `gitlab_runner_tags` — runner identity

## Deploy

Deployment is automated via Ansible. See [`../ansible/README.md`](../ansible/README.md).
