<title>Funnel on Kubernetes</title>

# Funnel on Kubernetes

Deploys [Funnel](https://github.com/ohsu-comp-bio/funnel) using the Helm chart in this directory (`charts/compute`, chart name `gdi-funnel`). The chart renders a Deployment + Service (+ optional Ingress) for the Funnel server, a PVC for its work directory, and server/worker config secrets. The Kubernetes compute backend then submits each task as its own Job in the same namespace.

Requirements: cluster access, `kubectl`, `helm` v3, a container registry the cluster can pull from.

## Before you start

- **The `funnel-sa` ServiceAccount is not created by this chart.** `templates/funnel-deployment.yaml` sets `serviceAccount: {{ .Values.funnel.serviceAccount }}` (default `funnel-sa`), but there's no `ServiceAccount`/`Role`/`RoleBinding` template here — it must already exist in your namespace with permission to create/manage `Jobs` and `PersistentVolumeClaims` (the Kubernetes backend creates one of each per task). Ask your cluster admin to provision it if it's missing.
- **`values.yaml` ships with placeholder secrets** (`oidc.clientid`, `oidc.clientsecret`, `oidc.redirecturl`, `basicauth.user`, `basicauth.password`, all `XXX`/`SOME_HOST`) — you must override all of these, see step 2.
- **`rpcclient.user`/`rpcclient.password` in `values.yaml` are unused** — `files/funnel-server-config.yml` and `files/funnel-worker-config.yml` both take the `RPCClient` credentials from `basicauth.user`/`basicauth.password` instead. Don't be surprised the `rpcclient` values have no effect.
- **The per-task PVC requires `ReadWriteMany`** (`templates/pvc.yaml`) — make sure `pvc.storageClass` points at a StorageClass that supports it (e.g. an NFS-backed CSI class like the default `nfs-csi`).

## 1. Find (or create) your namespace

```bash
kubectl get namespaces
```
If you don't already have one, ask whoever gave you cluster access, or create it if you manage your own:
```bash
kubectl create namespace <namespace>
```

## 2. Fill in your values

```bash
cp charts/compute/values.yaml my-values.yaml
```
Edit `my-values.yaml`. At minimum:

| Field | What it's for |
|---|---|
| `image` | Funnel image to run (server and worker both use this) |
| `oidc.url` / `oidc.clientid` / `oidc.clientsecret` / `oidc.redirecturl` / `oidc.audience` | OIDC settings the server uses to authenticate API requests |
| `basicauth.user` / `basicauth.password` | Basic-auth admin credentials for the server; also used as the RPC credentials between server and worker |
| `pvc.storageClass` / `pvc.size` | StorageClass (must support `ReadWriteMany`, see above) and size for the work-directory PVC |
| `funnel.serviceAccount` | Name of the pre-existing ServiceAccount (see above), if not `funnel-sa` |
| `funnel.resources` | CPU/memory/ephemeral-storage requests+limits for the server pod |
| `ingress.enabled` / `ingress.className` / `ingress.host` | Set `ingress.host` to your domain, or set `ingress.enabled: false` and use port-forwarding instead |

See `charts/beacon/README.md` / `charts/fdp/README.md` for the values-file style used elsewhere in this repo.

## 3. Install

```bash
helm install my-funnel charts/compute -n <namespace> -f my-values.yaml
```
Creates the PVC, the server Deployment + Service (+ Ingress if enabled), and the `<release>-server-config`/`<release>-worker-config` secrets.

## 4. Check it came up

```bash
kubectl get pods -n <namespace>
kubectl logs -n <namespace> deploy/my-funnel
```
Look for one pod in `Running` state.

- Pod stuck in `CrashLoopBackOff`/`Error`? Check the logs above — a common cause is a missing/invalid `oidc.*` value (the server won't start if it can't load the OIDC service config).
- `Forbidden` errors when tasks are submitted? Revisit the ServiceAccount permissions from **Before you start**.
- Task submission fails at PVC creation? Check `pvc.storageClass` supports `ReadWriteMany`.

## Access it

With `ingress.enabled: true` (default), the server is reachable at `https://<ingress.host>`.

Without Ingress, use port-forwarding instead:
```bash
kubectl --namespace <namespace> port-forward svc/my-funnel 8000:8000
```

## Test it

```bash
curl -i https://<ingress.host>/healthz
```
Health check, expect `200 OK`.

```bash
curl -s https://<ingress.host>/v1/service-info
```
Returns server metadata (name, version, supported storage backends).

```bash
curl -s -u <basicauth.user>:<basicauth.password> -X POST https://<ingress.host>/v1/tasks \
  -d '{"name":"Hello world","executors":[{"image":"alpine","command":["echo","hello world"]}]}'
```
Creates a task, returns `{"id": "<task-id>"}`.

```bash
curl -s -u <basicauth.user>:<basicauth.password> "https://<ingress.host>/v1/tasks/<task-id>?view=FULL"
```
Checks task status/result — look for `"state": "COMPLETE"`.

## Uninstall / clean up

```bash
helm uninstall my-funnel -n <namespace>
```
Per-task PVCs created for individual tasks are cleaned up by the backend as each task's Job finishes; check `kubectl get pvc -n <namespace>` for leftovers if tasks were interrupted mid-run.
