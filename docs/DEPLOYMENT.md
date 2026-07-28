# Deployment & Backups

This stack is designed to run on a single server **without exposing SSH (port 22)
or any management port to the public internet**. Access for CI/CD goes through a
[Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/),
so the only inbound ports on the box are 80/443 (and even those can be fronted by
Cloudflare).

Two workflows ship with the repo:

| Workflow | Trigger | What it does |
|----------|---------|--------------|
| [`.github/workflows/deploy.yml`](../.github/workflows/deploy.yml) | push to `main`, manual | Sync the stack over the tunnel and `docker compose up -d` |
| [`.github/workflows/backup.yml`](../.github/workflows/backup.yml) | nightly, manual | Dump DB + content, upload to Cloudflare R2, prune old backups |

Both are built on small, purpose-built actions (zero telemetry, secrets kept in
`env:` blocks, never on a command line):

- **[cloudflare-tunnel-ssh-action](https://github.com/NX1X/cloudflare-tunnel-ssh-action)** - SSH to the server through a Cloudflare Tunnel, no port 22 exposed.
- **[Cloudflare-R2-backup-action](https://github.com/NX1X/Cloudflare-R2-backup-action)** - upload / verify / list / prune backups in Cloudflare R2 (S3-compatible, no egress fees).
- **[Cloudflare-WARP-action](https://github.com/NX1X/Cloudflare-WARP-action)** - optional: put the runner on your private Zero Trust network instead of using a tunnel host.

## 1. Prerequisites

On the server:

- Docker + Docker Compose, `rsync`, and a non-root deploy user (default `deploy`)
  that can run `docker compose` (in the `docker` group).
- The stack lives in `DEPLOY_DIR` (default `/opt/ghost-blog`) with its own
  `.env` and `ssl/` - **these are never overwritten by CI** (the deploy excludes
  `.env`, `ssl/`, and `data/`).
- `cloudflared` configured to route an SSH hostname (e.g. `ssh.example.com`) to
  `localhost:22`, protected by a Cloudflare Access **service token**.

## 2. Deploy access via Cloudflare Tunnel (recommended)

1. In Cloudflare Zero Trust, create a **service token** (Access -> Service Auth).
   This gives you a client id + secret used for headless auth.
2. Publish an SSH application on a hostname (e.g. `ssh.example.com`) routed to the
   server's `localhost:22` via `cloudflared`, and add an Access policy that allows
   the service token.
3. Add an SSH keypair: put the public key in the deploy user's
   `~/.ssh/authorized_keys`; store the private key as the `DEPLOY_SSH_KEY` secret.

### Required GitHub secrets / variables

| Name | Type | Used by |
|------|------|---------|
| `CF_ACCESS_CLIENT_ID` | secret | deploy, backup |
| `CF_ACCESS_CLIENT_SECRET` | secret | deploy, backup |
| `DEPLOY_SSH_KEY` | secret | deploy, backup |
| `DEPLOY_SSH_HOST` | secret | deploy, backup |
| `DEPLOY_SSH_USER` | variable (default `deploy`) | deploy, backup |
| `DEPLOY_DIR` | variable (default `/opt/ghost-blog`) | deploy, backup |
| `DEPLOY_PULL_PROFILES` | variable (default `analytics activitypub`) | deploy |
| `DATABASE_ROOT_PASSWORD` | secret | backup (must match server `.env`) |
| `CF_ACCOUNT_ID` | secret | backup |
| `R2_ACCESS_KEY_ID` | secret | backup |
| `R2_SECRET_ACCESS_KEY` | secret | backup |
| `R2_BUCKET` | variable | backup |

Deploys run against a GitHub Environment named `production` - add required
reviewers there if you want a manual approval gate before each deploy.

### What a deploy starts

The deploy pulls images for the profiles listed in `DEPLOY_PULL_PROFILES`, but
starts only the always-on services. Everything behind a profile is a one-shot
job - `tinybird-login` is an interactive Tinybird auth step and
`activitypub-migrate` applies schema migrations - so re-running them on every
deploy would be wrong. Their images are kept current on the server, and you
trigger the jobs yourself when you need them:

```bash
docker compose --profile analytics up
```

Set `DEPLOY_PULL_PROFILES` to an empty value to skip pulling profiled images
entirely.

## 3. Off-site backups to Cloudflare R2

Create an R2 bucket and an R2 API token (Object Read & Write), then set the R2
secrets/variable above. The nightly workflow bundles a `mysqldump` of all
databases plus the Ghost content directory, uploads it under `ghost-backups/`,
and prunes anything older than 30 days. Adjust the schedule, prefix, and
`retention-days` in [`backup.yml`](../.github/workflows/backup.yml).

> Paths assume the default bind mounts (`data/ghost`, `data/mysql`). If you
> changed `UPLOAD_LOCATION` / `MYSQL_DATA_LOCATION`, update the `tar` path in the
> backup workflow to match.

Restore is the reverse: pull the bundle from R2, `gunzip` the SQL dump into the
`db` container, and untar the content into the Ghost content volume.

## 4. Alternative: WARP instead of a tunnel host

If your server sits on a private network reachable through your Cloudflare Zero
Trust org (rather than behind a single published SSH hostname), put the runner
**on that network** with [Cloudflare-WARP-action](https://github.com/NX1X/Cloudflare-WARP-action)
and SSH directly to the private address:

```yaml
- name: Join the Zero Trust network with WARP
  uses: NX1X/Cloudflare-WARP-action@3661004e1d80730d7b19191d65581fb9ff253c5a # v1.0.0
  with:
    organization: your-team          # <team>.cloudflareaccess.com
    auth-client-id: ${{ secrets.CF_ACCESS_CLIENT_ID }}
    auth-client-secret: ${{ secrets.CF_ACCESS_CLIENT_SECRET }}
    mode: warp
# ...then rsync / ssh to the server's private IP as usual.
```

Pick **one** access method per workflow - the tunnel host (sections 2-3) or WARP,
not both.
