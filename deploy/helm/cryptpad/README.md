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

## Keycloak OIDC

Realm `brandonslab`, client `cryptpad`, issuer `https://auth.brandonslab.work/auth/realms/brandonslab`.

Redirect URI must be exactly `https://draw.brandonslab.work/ssoauth`. Store the client secret in `cryptpad-secrets` key `oidcClientSecret`. The chart clones [cryptpad/sso](https://github.com/cryptpad/sso) at pod start.

Login page shows **Log in with keycloak**. After SSO, CryptPad still asks for a pad password (`forceCpPassword`) because that password derives the encryption key. `sso.enforced` stays false so the first-run `/install/#...` admin URL still works.

## Cloudflare tunnel

Live on `brandonslab-k3s-local`. Do not replace the full ingress list when editing.

| Host | Origin | Access |
|---|---|---|
| `draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` | none (share links) |
| `sandbox-draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` | none (share links) |

Use `sandbox-draw` (one DNS label), not `sandbox.draw`. Cloudflare Universal SSL is `*.brandonslab.work` only.

DNS CNAME for both → `95bab145-c27d-4407-a192-93b5627fb3a9.cfargotunnel.com` (proxied).

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
