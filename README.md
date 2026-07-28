# Self-Hosted Ghost CMS with Docker

Production-ready Ghost CMS deployment with Docker Compose, featuring nginx reverse proxy, optional analytics, and ActivityPub federation.

[![Release](https://img.shields.io/github/v/release/NX1X/self-hosted-ghost-blog?sort=semver&color=blue)](https://github.com/NX1X/self-hosted-ghost-blog/releases)
[![Last commit](https://img.shields.io/github/last-commit/NX1X/self-hosted-ghost-blog)](https://github.com/NX1X/self-hosted-ghost-blog/commits/main)
[![Issues](https://img.shields.io/github/issues/NX1X/self-hosted-ghost-blog)](https://github.com/NX1X/self-hosted-ghost-blog/issues)
[![Views](https://visitor-badge.laobi.icu/badge?page_id=NX1X.self-hosted-ghost-blog)](https://github.com/NX1X/self-hosted-ghost-blog)

[![CodeQL](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/codeql.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/codeql.yml)
[![Trivy](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/trivy.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/trivy.yml)
[![Gitleaks](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/gitleaks.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/gitleaks.yml)
[![zizmor](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/zizmor.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/zizmor.yml)
[![Lint](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/lint.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/lint.yml)
[![Dependency Review](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/dependency-review.yml/badge.svg)](https://github.com/NX1X/self-hosted-ghost-blog/actions/workflows/dependency-review.yml)
[![License: Apache-2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

## A Word from the Developer

I have always been a writing person. The written word is a part of my life. In
the summer of 2025 I decided to start building a customized self-hosted blog. I
found that Ghost fit my needs and started working on it. It has been ready and
in use by me since then, and now I am opening it to the public.

I want to encourage people to write non-AI-generated content, and I welcome you
to do it.

## Stack

![Ghost](https://img.shields.io/badge/Ghost-6.x-738A94?logo=ghost&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?logo=mysql&logoColor=white)
![nginx](https://img.shields.io/badge/nginx-1.28-009639?logo=nginx&logoColor=white)
![Docker](https://img.shields.io/badge/Docker_Compose-v2-2496ED?logo=docker&logoColor=white)

## Architecture

```
Internet → CDN / Tunnel → Nginx → Ghost CMS
                                → ActivityPub (optional)
                                → Tinybird Analytics (optional)
```

- **Ghost** : CMS application (port 2368 internal)
- **MySQL** : Database backend with health checks
- **Nginx** : Reverse proxy with rate limiting, bot protection, and security headers
- **ActivityPub** : Federated social networking (optional profile)
- **Tinybird** : Web analytics (optional profile)

## Quick Start

```bash
git clone https://github.com/NX1X/self-hosted-ghost-blog.git
cd self-hosted-ghost-blog

cp env.example .env
# Edit .env with your domain, database passwords, and mail settings

docker compose up -d
```

## Configuration

All configuration is done via environment variables in `.env`. Key settings:

| Variable | Description |
|---|---|
| `DOMAIN` | Your blog domain (e.g., `blog.example.com`) |
| `DATABASE_PASSWORD` | MySQL password for Ghost |
| `DATABASE_ROOT_PASSWORD` | MySQL root password |
| `mail__*` | Email configuration (required for admin features) |

### Optional Profiles

```bash
# Include analytics
docker compose --profile=analytics up -d

# Include ActivityPub federation
docker compose --profile=activitypub up -d

# Include both
COMPOSE_PROFILES=analytics,activitypub docker compose up -d
```

## Common Commands

```bash
docker compose logs -f ghost          # View Ghost logs
docker compose ps                     # Check service status
docker compose pull                   # Pull latest images
docker compose restart ghost          # Restart Ghost
docker compose exec ghost sh          # Shell into Ghost container
docker compose exec db mysql -u root -p  # MySQL CLI
```

## Migration from Ghost CLI

If you are migrating from a Ghost CLI installation, use the included migration script:

```bash
./scripts/migrate.sh
```

This will back up your existing installation, export the database, convert your config to environment variables, and set up the Docker Compose environment.

## SSL / HTTPS

Nginx expects SSL certificates at `ssl/cert.pem` and `ssl/key.pem`. You can provide these from:

- A CDN or tunnel that terminates SSL upstream (Cloudflare, etc.)
- Let's Encrypt / certbot
- Any other certificate authority

Update `nginx/nginx.conf` with your domain name before starting.

## Deployment & Backups

You can run the stack with `docker compose up -d` on any host. For zero-trust
CI/CD - deploying without exposing SSH (port 22) to the internet - the repo ships
two GitHub Actions workflows built on small, telemetry-free actions:

- **Deploy** ([`.github/workflows/deploy.yml`](.github/workflows/deploy.yml)) reaches the
  server through a [Cloudflare Tunnel](https://github.com/NX1X/cloudflare-tunnel-ssh-action)
  and restarts the stack. Only the files the server needs are synced; `.env`,
  `ssl/`, and `data/` are never overwritten.
- **Backup** ([`.github/workflows/backup.yml`](.github/workflows/backup.yml)) dumps the
  databases + content nightly and ships them to
  [Cloudflare R2](https://github.com/NX1X/Cloudflare-R2-backup-action) (S3-compatible,
  no egress fees), pruning old backups by retention.
- Need the runner on a private Zero Trust network instead of a tunnel host? Use
  [Cloudflare WARP](https://github.com/NX1X/Cloudflare-WARP-action).

Full setup, secrets, and the WARP alternative: [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

## Project Structure

```
.
├── compose.yml              # Docker Compose services (digest-pinned, hardened)
├── compose.build.yml        # Optional overlay to build the Tinybird image locally
├── env.example              # Environment variable template
├── nginx/nginx.conf         # Nginx reverse proxy config
├── mysql-init/              # MySQL initialization scripts
├── scripts/
│   ├── migrate.sh           # Ghost CLI migration tool
│   └── config-to-env.js     # Config converter
├── tinybird/                # Analytics setup tools
├── ssl/                     # SSL certificates (not committed)
├── docs/                    # Deployment guide
└── .github/
    ├── renovate.json        # Automated dependency/image updates
    └── workflows/           # CI: security scans, deploy, backup
```

## Maintenance & Security Automation

This repo ships with GitHub Actions that keep it up to date and continuously
scanned. They run on push, on pull requests, and on a weekly schedule.

| Workflow | Tool | Purpose |
|----------|------|---------|
| `gitleaks.yml` | [Gitleaks](https://github.com/gitleaks/gitleaks) | Scans the full git history and PRs for committed secrets. |
| `trivy.yml` | [Trivy](https://trivy.dev/) | Scans `compose.yml`, the Dockerfile and the filesystem for CVEs, misconfigurations and secrets; uploads results to the Security tab. |
| `lint.yml` | ShellCheck, Hadolint, actionlint | Lints shell scripts, the Dockerfile, the workflows, and validates `compose.yml`. |
| `codeql.yml` | [CodeQL](https://codeql.github.com/) | Semantic SAST (security-extended) on the JS migration script and the Actions workflows. |
| `zizmor.yml` | [zizmor](https://github.com/woodruffw/zizmor) | Audits the workflows themselves for script injection, over-broad permissions and credential persistence. |
| `dependency-review.yml` | [dependency-review](https://github.com/actions/dependency-review-action) | Blocks PRs that add vulnerable or disallowed-license dependencies. |
| `publish-tinybird.yml` | Buildx, Trivy, [attest-build-provenance](https://github.com/actions/attest-build-provenance) | Builds `tinybird/Dockerfile`, scans it, and publishes a multi-arch image with an SBOM and signed provenance to GHCR. |

### The Tinybird helper image

Every service in `compose.yml` is an upstream image except one: the Tinybird
CLI helper built from [`tinybird/Dockerfile`](tinybird/Dockerfile). It is
published to GHCR as
[`ghcr.io/nx1x/self-hosted-ghost-blog/tinybird`](https://github.com/NX1X/self-hosted-ghost-blog/pkgs/container/self-hosted-ghost-blog%2Ftinybird)
so the production server pulls it like everything else instead of running
`pip install` on each deploy. It is built for `linux/amd64` and `linux/arm64`,
scanned by Trivy before push (CRITICAL blocks the publish), and shipped with an
SBOM plus in-toto build provenance:

```bash
gh attestation verify \
  oci://ghcr.io/nx1x/self-hosted-ghost-blog/tinybird:edge \
  --repo NX1X/self-hosted-ghost-blog
```

To build it yourself rather than pulling it, use the overlay:

```bash
docker compose -f compose.yml -f compose.build.yml build
docker compose -f compose.yml -f compose.build.yml --profile analytics up
```

### Renovate setup

Dependency updates are handled by the [Mend Renovate GitHub App](https://github.com/apps/renovate),
configured entirely through [`.github/renovate.json`](.github/renovate.json). Install
the app on the repository and it opens update PRs automatically - no self-hosted
workflow or token to manage.

Image update strategy:

- `nginx`, `ghost`, `mysql`, `ghost/traffic-analytics` are pinned to version
  tags, so Renovate proposes tracked upgrades with changelogs.
- `tryghost/activitypub` / `activitypub-migrations` publish no semver tags, so
  Renovate pins and bumps their immutable `@sha256` digest instead.
- Every image is already pinned to an immutable `@sha256` digest; Renovate keeps
  those digests current and opens a PR for each change.
- The self-published `tinybird` image is exempt from the 14-day cooldown that
  guards third-party updates - its digest is produced by this repo's own
  scanned, attested build - so Renovate pins it as soon as it is published.

## Disclaimer

These Compose files and the surrounding tooling are **not affiliated with the
official Ghost project or any official Ghost Docker Compose project**. Most of
the code here is my own architecture, which I designed and built on top of
Ghost CMS. You are more than welcome to suggest improvements or open pull
requests.

## Roadmap

Planned and potential improvements are tracked in [ROADMAP.md](ROADMAP.md).

## Contributing & Community

- [CONTRIBUTING.md](CONTRIBUTING.md) : how to set up locally and propose changes
- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) : community standards
- [SECURITY.md](SECURITY.md) : how to report vulnerabilities privately
- [CHANGELOG.md](CHANGELOG.md) : release history
- [ROADMAP.md](ROADMAP.md) : what is planned

## Related projects

Part of the **nxtools** family - small, telemetry-free building blocks that the
deploy and backup workflows here are built on:

- [cloudflare-tunnel-ssh-action](https://github.com/NX1X/cloudflare-tunnel-ssh-action) : zero-trust SSH over a Cloudflare Tunnel, no exposed port 22.
- [Cloudflare-R2-backup-action](https://github.com/NX1X/Cloudflare-R2-backup-action) : upload / verify / list / prune backups in Cloudflare R2.
- [Cloudflare-WARP-action](https://github.com/NX1X/Cloudflare-WARP-action) : headless Cloudflare WARP enrollment for CI runners.

Browse the whole family under the [`nxtools`](https://github.com/search?q=topic%3Anxtools+user%3ANX1X&type=repositories) topic.

## License

Apache License 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details.

"Ghost" is a trademark of the Ghost Foundation. This project is independent and
unaffiliated; it orchestrates the official upstream images and does not
redistribute Ghost source code.
