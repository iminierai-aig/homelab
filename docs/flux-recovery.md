---

````
# Flux Recovery Guide

A concise, actionable reference for fixing Git or reconciliation issues in your homelab cluster.

---

## 1️⃣ Identify the Issue

```bash
kubectl -n flux-system get gitrepository flux-system -o wide
kubectl -n flux-system describe gitrepository flux-system
kubectl -n flux-system logs deploy/source-controller --tail=100
````

**Common symptoms**

* `authentication required`
* `context deadline exceeded`
* `failed to determine revision`

---

## 2️⃣ Verify Git Settings

Check if the repo URL changed:

```bash
kubectl -n flux-system get gitrepository flux-system -o jsonpath='{.spec.url}{"\n"}'
```

If changed, patch:

```bash
kubectl -n flux-system patch gitrepository flux-system --type merge \
  -p '{"spec":{"url":"https://github.com/<new-org>/<repo>.git"}}'
```

---

## 3️⃣ Refresh Git Credentials

### HTTPS + PAT

```bash
export GITHUB_TOKEN=<PAT>
flux -n flux-system create secret git flux-system \
  --url=https://github.com/<org>/<repo>.git \
  --username=<github_user> \
  --password="$GITHUB_TOKEN" \
  --export | kubectl apply -f -
```

### SSH Deploy Key

```bash
flux -n flux-system create secret git flux-system \
  --ssh-key ~/.ssh/flux_ed25519 --export | kubectl apply -f -
kubectl -n flux-system patch gitrepository flux-system --type merge \
  -p '{"spec":{"url":"git@github.com:<org>/<repo>.git"}}'
```

> **Tip:** Use SSH keys for long-term setups — avoids PAT rotation and 2FA interruptions.

---

## 4️⃣ Reconcile Everything

```bash
flux -n flux-system reconcile source git flux-system --timeout=5m --verbose

for k in $(flux -n flux-system get kustomizations --no-header | awk '{print $1}'); do
  flux -n flux-system reconcile kustomization $k --timeout=5m --verbose
done
```

---

## 5️⃣ Verify State

```bash
flux -n flux-system get sources git
flux -n flux-system get kustomizations
```

✅ Expect `READY=True` across all objects.

---

## 6️⃣ Backup Working State

```bash
flux export all -n flux-system > flux-backup.yaml
```

Keep this file versioned in your repo for fast recovery.

---

**Note:**
This guide assumes all Flux objects live in the `flux-system` namespace.
Adapt paths and repo names for additional environments (e.g., staging, prod).

---

