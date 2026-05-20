# ignis-infra

Infrastructure-as-code for the Ignis platform. Infra components and application services are fully decoupled — each has its own deployment lifecycle and can run on separate VMs.

## Structure

```
ignis-infra/
├── components/                    # infra — deployed and managed independently
│   ├── postgres/
│   ├── qdrant/
│   └── redis/
└── services/                      # apps — only containers, no infra
    ├── ignis-fit/                  # backend + frontend
    └── ignis-rag-engine/           # api + worker + web
```

## How it works

- **Components** deploy to an infra VM. Each has its own workflow triggered by changes to that component only.
- **Services** deploy to an app VM. They connect to infra via `DB_HOST`, `REDIS_HOST`, `QDRANT_HOST` env vars in their `.env` file.
- Neither side knows about the other at deploy time — fully independent lifecycles.

## GitHub secrets

### Shared
| Secret | Description |
|--------|-------------|
| `VM_SSH_PRIVATE_KEY` | SSH private key used across all deployments |
| `GHCR_USERNAME` | GitHub username for `docker login ghcr.io` |
| `GHCR_TOKEN` | GitHub PAT with `read:packages` scope |

### Per component (can point to same VM or different VMs)
| Secret | Description |
|--------|-------------|
| `POSTGRES_VM_HOST` | IP of VM running postgres |
| `POSTGRES_VM_USER` | SSH user on that VM |
| `QDRANT_VM_HOST` | IP of VM running qdrant |
| `QDRANT_VM_USER` | SSH user on that VM |
| `REDIS_VM_HOST` | IP of VM running redis |
| `REDIS_VM_USER` | SSH user on that VM |

### Per service
| Secret | Description |
|--------|-------------|
| `VM_HOST` | IP of VM running app services |
| `VM_USER` | SSH user on that VM |

## VM setup (first time per VM)

```bash
# Create directory structure
mkdir -p /home/ignis-deploy/ignis-infra/components/postgres
mkdir -p /home/ignis-deploy/ignis-infra/components/qdrant
mkdir -p /home/ignis-deploy/ignis-infra/components/redis
mkdir -p /home/ignis-deploy/ignis-infra/services/ignis-fit
mkdir -p /home/ignis-deploy/ignis-infra/services/ignis-rag-engine

# Place .env files from the .env.example templates, fill in real values
# Components: components/<name>/.env
# Services: services/<name>/.env
```

## Adding a new service

1. Create `services/<name>/docker-compose.yml` — app containers only, use `${DB_HOST}` etc. for infra addresses
2. Create `services/<name>/.env.example`
3. Copy an existing service workflow and update the `paths:` filter and `cd` path
4. On the app VM: create the directory and place the `.env` file
