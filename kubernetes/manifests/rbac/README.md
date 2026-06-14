# RBAC

Who gets what permissions in the `gitlab-runner` namespace.

## The three objects

```
ServiceAccount (gitlab-runner)
        │
        │  RoleBinding ──► Role (gitlab-runner)
        │
        ▼
  Runner Deployment pod          uses K8s API to spawn/manage CI jobs

ServiceAccount (gitlab-runner-jobs)
        │
        │  (no RoleBinding)
        ▼
  CI job pods                      no K8s API access
```

| File | Kind | Purpose |
|------|------|---------|
| `serviceaccount.yaml` | ServiceAccount | Identity for the **runner manager** pod |
| `serviceaccount-jobs.yaml` | ServiceAccount | Identity for **CI job** pods (no permissions) |
| `role.yaml` | Role | What the runner manager may do in this namespace |
| `rolebinding.yaml` | RoleBinding | Grants the Role to `gitlab-runner` SA only |

## Why two service accounts?

Previously both the runner and every CI job used `gitlab-runner`. That meant **any job script could call the Kubernetes API** with full runner permissions — create pods, read secrets, etc.

Now:

- **`gitlab-runner`** — runner pod only. Has the Role below.
- **`gitlab-runner-jobs`** — job pods only. No RoleBinding, so no API access.

Set in `vars.env`:

```
JOB_SERVICE_ACCOUNT=gitlab-runner-jobs
```

## What the runner Role allows (and why)

These permissions apply **only inside `gitlab-runner` namespace** (Role, not ClusterRole).

| Resource | Verbs | Why the runner needs it |
|----------|-------|-------------------------|
| `pods` | create, delete, get, list, watch | Spawn CI job pods, monitor status, clean up |
| `pods/exec` | create, delete, get, patch | Run `.gitlab-ci.yml` scripts inside job containers |
| `pods/attach` | create, delete, get, patch | Stream logs and attach to running jobs |
| `pods/log` | get, list | Read build logs |
| `secrets` | create, delete, get, update | Inject masked/protected CI variables per job |
| `serviceaccounts` | get | Confirm `gitlab-runner-jobs` exists before creating job pods |
| `services` | create, get | Expose ports when using `CI_DEBUG_SERVICES` |

## What was removed (and why)

| Removed | Reason |
|---------|--------|
| `configmaps` | Not needed for basic CI. Add back only if you mount ConfigMaps in job pods |
| `secrets` list/watch/patch | Runner creates/deletes its own job secrets; no need to enumerate all secrets |
| `services` delete/list/watch | Runner creates services; does not need to delete or list all services |
| `pods` patch | Not required for default executor behavior in v18 |

## Optional permissions (not included)

Add to `role.yaml` only if you enable the feature:

| Feature | Extra rules |
|---------|-------------|
| ConfigMap volumes in CI | `configmaps`: create, delete, get, update |
| `NamespacePerJob` | `namespaces`: create, delete (needs ClusterRole) |
| Pause pods / autoscaler | `deployments`, `priorityclasses` (ClusterRole for PriorityClass) |
| Pod disruption budget | `poddisruptionbudgets`: create, get |
| Debug pod events | `events`: list, watch |

See [GitLab Runner — Configure runner API permissions](https://docs.gitlab.com/runner/executors/kubernetes/#configure-runner-api-permissions).

## Verify permissions

```bash
# Runner SA can create pods
kubectl auth can-i create pods \
  --as=system:serviceaccount:gitlab-runner:gitlab-runner \
  -n gitlab-runner

# Job SA cannot
kubectl auth can-i create pods \
  --as=system:serviceaccount:gitlab-runner:gitlab-runner-jobs \
  -n gitlab-runner
# expected: no
```

## Helm equivalent

In `kubernetes/helm/values.yaml`, set explicit `rbac.rules` (empty rules = `*` on everything in namespace). Use the same verb list as `role.yaml` above.
