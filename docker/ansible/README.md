# Ansible

Deploys the Docker-based GitLab Runner to a remote VM over SSH.

Runner config lives in [`../docker/vars.yml`](../docker/vars.yml).  
SSH target config lives in `inventory/hosts.ini`.

## Setup

```bash
cp inventory/hosts.example.ini inventory/hosts.ini
# edit inventory/hosts.ini — server IP, user, SSH key
# edit ../docker/vars.yml — GitLab URL, token, runner name
```

## Deploy

```bash
ansible -i inventory/hosts.ini runner_server -m ping
ansible-playbook playbook/deploy_runner.yml --check --diff
ansible-playbook playbook/deploy_runner.yml
```

## Verify

```bash
ssh USER@HOST docker ps
ssh USER@HOST docker exec -it gitlab-runner gitlab-runner list
```
