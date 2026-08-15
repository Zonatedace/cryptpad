# CryptPad (self-hosted diagrams)

Open-source [CryptPad](https://cryptpad.org) packaged for Docker and Kubernetes.

This is the existing tool we picked instead of writing a Lucidchart clone. CryptPad already has:

- A **diagram** app built on the diagrams.net / draw.io engine (flowcharts, UML, network, ER — the Lucid-like part)
- A **whiteboard** for looser sketching
- **Live collaboration** on a shared pad
- **Share links** with view / edit access, passwords, and expiration
- End-to-end encryption (the host cannot read pad contents)

CryptPad itself is AGPL-3.0. This repo is MIT deploy configuration.

## Why this, not raw draw.io or Excalidraw

| Tool | Lucid-like shapes | Live collab | Share links | Self-host |
|---|---|---|---|---|
| diagrams.net / draw.io | Yes | Weak | File / Drive | Easy |
| Excalidraw | Sketchy, not formal | Excellent on excalidraw.com; official image does not | Rooms | Partial |
| **CryptPad** | Diagram app = draw.io | Built-in | Built-in | Official image |

## Local

```bash
cp .env.example .env
# set CPAD_LOGIN_SALT=$(openssl rand -hex 32)
mkdir -p data/blob data/block data/data data/datastore customize
docker compose up -d
docker compose logs -f cryptpad
```

Open http://localhost:3080 and use the first-run setup URL printed in the logs.

Change `loginSalt` after anyone has registered and every password stops working.

## Kubernetes (brandonslab)

Chart lives in this repo (`deploy/helm/cryptpad`) and is mirrored at `Infrastructure/k3s/charts/cryptpad`.

```bash
kubectl create namespace cryptpad
kubectl -n cryptpad create secret generic cryptpad-secrets \
  --from-literal=loginSalt="$(openssl rand -hex 32)"

helm upgrade --install cryptpad deploy/helm/cryptpad -n cryptpad
```

Public hostnames (wire through tunnel `brandonslab-k3s-local`, **no Cloudflare Access** so share links work for other people):

| Host | Service |
|---|---|
| `https://draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` |
| `https://sandbox.draw.brandonslab.work` | `http://cryptpad.cryptpad.svc.cluster.local:80` |

Both names must hit the same service. CryptPad uses the sandbox hostname for XSS isolation; it is not a second app.

After the pod is Ready:

```bash
kubectl -n cryptpad logs deploy/cryptpad -c cryptpad | findstr /i "http"
```

Open the printed `/install/#...` URL **once** to create the admin account. Then New → **Diagram** (or Whiteboard). Share is the link icon in the pad.

## What you get

- New diagram → Lucid-style canvas (shapes, connectors, layers)
- Share → anyone with the link can view or edit live
- Presence + realtime edits over websockets
- Pads expire if nobody pins them (default 90 days); register an account to keep them

OnlyOffice (spreadsheets / docs / slides) is **off** on purpose. Turn `installOnlyOffice` on in values if you want the rest of the office suite.

## Upstream

- App: https://github.com/cryptpad/cryptpad
- Image: `cryptpad/cryptpad:version-2026.5.1`
- Admin docs: https://docs.cryptpad.org/en/admin_guide/installation.html
