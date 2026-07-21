# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.1] - 2026-07-21

Initial public release.

### Added

- Ghost CMS with a MySQL backend and an nginx reverse proxy, orchestrated with
  Docker Compose. Optional profiles for Tinybird web analytics and ActivityPub
  federation.
- nginx hardening: TLS 1.2/1.3, security headers, HSTS, OCSP stapling, admin
  login rate-limiting, bot/scanner-path blocking, and static-asset caching.
- Migration tooling from a Ghost CLI installation (`scripts/migrate.sh`,
  `scripts/config-to-env.js`).
- CI/CD deploy over a Cloudflare Tunnel (no inbound SSH) and a nightly Cloudflare
  R2 off-site backup workflow with retention-based pruning.
- Security automation: Renovate, Gitleaks, Trivy, CodeQL, zizmor,
  dependency-review, and ShellCheck/Hadolint/actionlint.
- Deployment guide under `docs/` (including a WARP option).

### Security

- All Docker images and GitHub Actions pinned to immutable SHA digests.
- Containers run with `no-new-privileges`, dropped capabilities on nginx, PID
  limits, bounded logging, and an edge/internal network split that isolates the
  database from the reverse proxy.
- No secrets are committed; all configuration is supplied at runtime via a local,
  gitignored `.env`. Trivy fails CI on CRITICAL misconfigurations.
