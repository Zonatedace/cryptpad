# cryptpad Helm chart

Encrypted diagrams + live collaboration (upstream [CryptPad](https://github.com/cryptpad/cryptpad) `2026.5.1`).

## Install (brandonslab)

```bash
kubectl create namespace cryptpad
kubectl -n cryptpad create secret generic cryptpad-secrets \
  --from-literal=loginSalt="$(openssl rand -hex 32)"

helm upgrade --install cryptpad . -n cryptpad
```

From this repo root:

```bash
helm upgrade --install cryptpad deploy/helm/cryptpad -n cryptpad
```

## Cloudflare tunnel

Merge these public hostnames onto `brandonslab-k3s-local`. Do not replace the full ingress list.

| Host | Origin | Access |
|---|---|---|
| `draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` | none / EveryoneBypass |
| `sandbox.draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` | none / EveryoneBypass |

DNS CNAME for both → `95bab145-c27d-4407-a192-93b5627fb3a9.cfargotunnel.com` (proxied).

No Cloudflare Access. Share links have to work for people who are not you. CryptPad encrypts pads.

## First-run admin

```bash
kubectl -n cryptpad logs deploy/cryptpad -c cryptpad
```

Open the `/install/#...` URL once, create the admin account, then **New → Diagram**.

## Ops

```bash
kubectl -n cryptpad get pods,svc,pvc
kubectl -n cryptpad logs deploy/cryptpad -c cryptpad
kubectl -n cryptpad port-forward svc/cryptpad 3080:80
```
