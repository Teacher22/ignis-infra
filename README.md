# ignis-infra

Centralised infrastructure-as-code for all Ignis services.

## Structure

```
ignis-infra/
├── components/               # reusable building blocks (reference configs)
│   ├── postgres/
│   ├── qdrant/
│   └── redis/
├── services/                 # one folder per service — this is the deploy unit
│   ├── ignis-fit/
│   │   ├── docker-compose.yml
│   │   └── .env.example
│   └── ignis-rag-engine/
│       ├── docker-compose.yml
│       └── .env.example
└── .github/
    └── workflows/
        ├── deploy-ignis-fit.yml
        └── deploy-ignis-rag-engine.yml
```

## VM setup (first time only)

The VM needs no git access. Just create the directory structure and place the `.env` file:

```bash
mkdir -p /root/ignis-infra/services/ignis-fit
mkdir -p /root/ignis-infra/services/ignis-rag-engine

# Copy .env.example from this repo and fill in real secrets
cp services/ignis-fit/.env.example /root/ignis-infra/services/ignis-fit/.env
# edit .env with real values — this file stays on the VM, never committed
```

Also add the GitHub Actions SSH public key to `~/.ssh/authorized_keys` on the VM so deployments can connect.

## Deployments

Deployments trigger automatically on push to `main` when files under `services/<name>/` change.
You can also trigger manually via **Actions → Deploy <service> → Run workflow**.

The workflow:
1. Checks out the repo in GitHub Actions
2. `scp`s the service folder to the VM (overwrites `docker-compose.yml` only — `.env` is untouched)
3. SSHs in, pulls updated Docker images from GHCR
4. Restarts containers with `docker compose up -d --remove-orphans`

## Required GitHub secrets

Set these under **Settings → Secrets → Actions** in this repo:

| Secret | Description |
|--------|-------------|
| `VM_HOST` | IP or hostname of the target VM |
| `VM_USER` | SSH username |
| `VM_SSH_PRIVATE_KEY` | Private key for SSH access to the VM |
| `GHCR_USERNAME` | GitHub username / org (for `docker login ghcr.io`) |
| `GHCR_TOKEN` | GitHub PAT with `read:packages` scope |

## Adding a new service

1. Create `services/<service-name>/docker-compose.yml` — include the infra components it needs plus app service entries
2. Create `services/<service-name>/.env.example`
3. Copy `.github/workflows/deploy-ignis-fit.yml` → `deploy-<service-name>.yml` and update the `paths:` filter and `cd` path
4. On the VM: `cp .env.example .env`, fill secrets, `docker compose up -d`
