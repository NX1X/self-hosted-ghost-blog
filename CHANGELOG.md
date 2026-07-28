# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.2.0] - 2026-07-28

### Added

- `publish-tinybird.yml` workflow: builds `tinybird/Dockerfile` for
  `linux/amd64` and `linux/arm64`, scans it with Trivy (CRITICAL blocks the
  publish), and pushes it to
  `ghcr.io/nx1x/self-hosted-ghost-blog/tinybird` with an SBOM and signed
  in-toto build provenance. Pull requests build and scan but never push.
- `compose.build.yml`: optional overlay that rebuilds the Tinybird helper image
  locally instead of pulling it from GHCR.
- `DEPLOY_PULL_PROFILES` repository variable (default `analytics activitypub`)
  controlling which compose profiles the deploy pulls images for.
- `release.yml` workflow: on a `v*` tag it publishes a GitHub Release using the
  matching `CHANGELOG.md` section, refusing to publish if that section is
  missing or empty, and points `X.Y.Z` / `X.Y` GHCR tags at the already
  published Tinybird image digest without rebuilding it. Previous releases were
  cut by hand.

### Changed

- `tinybird-login` and `tinybird-deploy` reference the published GHCR image
  instead of a local `build:` context. Previously `docker compose up` on the
  production server compiled the image there on every deploy - unscanned, not
  digest-pinnable, and requiring a build toolchain on the host.
- Renovate treats the self-published `tinybird` image as exempt from the 14-day
  supply-chain cooldown, since its digest comes from this repo's own scanned and
  attested build.
- `lint.yml` also validates `compose.yml` merged with `compose.build.yml`.
- The deploy workflow syncs `compose.build.yml` to the server so the Tinybird
  image can still be rebuilt on the box if GHCR is unreachable.

### Fixed

- The deploy workflow's bare `docker compose pull` only covered profile-less
  services, so images behind the `analytics` and `activitypub` profiles were
  never refreshed by CD. It now pulls those profiles explicitly while still
  starting only the always-on services, keeping the one-shot jobs
  operator-triggered.

## [0.1.2] - 2026-07-22

### Added

- Repository views badge in the README.
- zizmor configuration (`.github/zizmor.yml`) that ignores `ref-version-mismatch`
  for Renovate-managed digest pins (the SHA is correct; only the version comment
  drifts as major tags advance).

### Changed

- Deploy and backup workflows are manual-only (`workflow_dispatch`); their
  push/schedule triggers are commented out until deploy secrets are configured.
- Renovate batches all non-major updates (Docker + GitHub Actions) into a single
  weekly PR that auto-merges after the 14-day cooldown once CI is green; major
  and security updates stay separate and manual.
- zizmor now runs on every pull request (removed its workflow-path filter) so the
  required check can no longer deadlock PRs that don't touch workflows.
- Dependency bumps: nginx 1.28 -> 1.29, ghost/traffic-analytics 1.0.226 -> 1.0.273,
  and major GitHub Actions updates (checkout v7, setup-python v6, codeql-action v4,
  gitleaks-action v3).

### Removed

- OpenSSF Scorecard workflow and badge - it scored a solo, ruleset-protected repo
  inaccurately (it can't detect ruleset-based branch protection) and generated
  noisy Security-tab alerts.

### Fixed

- `scripts/config-to-env.js`: complete shell escaping (backslashes escaped before
  quotes), resolving CodeQL `js/incomplete-sanitization`.
- `tinybird/Dockerfile`: added a `HEALTHCHECK` (Trivy `DS-0026`).

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

[0.2.0]: https://github.com/NX1X/self-hosted-ghost-blog/compare/v0.1.2...v0.2.0
[0.1.2]: https://github.com/NX1X/self-hosted-ghost-blog/compare/v0.1.1...v0.1.2
[0.1.1]: https://github.com/NX1X/self-hosted-ghost-blog/releases/tag/v0.1.1
