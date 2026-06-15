# Tyk GitOps Workflow

## Overview

APIs are managed by **ArgoCD** syncing from the Git repo:
[github.com/kuldeepak-sudo/tyk-gitops-demo](https://github.com/kuldeepak-sudo/tyk-gitops-demo)

> ⚠️ Do **not** apply changes directly with `kubectl apply` — ArgoCD will overwrite them. Always update via Git.

---

## Steps

### 1. Navigate to the local repo

```bash
cd /Users/kd/tyk-gitops-demo
```

### 2. Find the ApiDefinition files

```bash
find . -name "*.yaml" | xargs grep -l "ApiDefinition"
```

### 3. Edit the relevant YAML

Example `hello-world` API definition:

```yaml
apiVersion: tyk.tyk.io/v1alpha1
kind: ApiDefinition
metadata:
  name: hello-world
spec:
  name: Operator hello world
  use_keyless: true
  protocol: http
  active: true
  proxy:
    target_url: http://httpbin
    listen_path: /operator-hello-world
    strip_listen_path: true
```

### 4. Commit and push to Git

```bash
git add .
git commit -m "Update hello-world listen path to /operator-hello-world"
git push origin main
```

### 5. Trigger ArgoCD sync (optional — auto-syncs within ~3 mins)

```bash
# Force sync immediately
kubectl -n argocd patch application tyk-operator-gitops-demo \
  --type merge -p '{"operation":{"sync":{"revision":"HEAD"}}}'

# Or using ArgoCD CLI
argocd app sync tyk-operator-gitops-demo
```

### 6. Watch reconciliation

```bash
kubectl -n tyk-demo get tykapis -w
```

You should see `SYNCSTATUS` change to `Successful`.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| ArgoCD keeps overwriting manual changes | Always update via Git, not `kubectl apply` |
| Sync not picking up changes | Force sync with the patch command in Step 5 |
| `SYNCSTATUS: Failed` | Check operator logs: `kubectl -n tyk-demo logs deployment/tyk-operator-controller-manager` |
| 401 Unauthorized | Re-patch `TYK_AUTH` secret with valid Dashboard token |
