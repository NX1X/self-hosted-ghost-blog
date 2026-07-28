# Contributing to self-hosted-ghost-blog

Thanks for your interest in contributing. This project is an independent
architecture built on top of Ghost CMS (see the disclaimer in the README). It
is not affiliated with the official Ghost project. Improvements, fixes, and
ideas are welcome.

## Ways to Contribute

- Report bugs or unexpected behavior via the issue templates.
- Suggest improvements to the Compose setup, nginx configuration, scripts, or
  documentation.
- Improve the docs under `docs/`.
- Open a pull request with a focused change.

## Local Setup

You need Docker and the Docker Compose v2 plugin.

```bash
git clone https://github.com/NX1X/self-hosted-ghost-blog.git
cd self-hosted-ghost-blog

cp env.example .env
# Edit .env with a domain, database passwords, and mail settings

docker compose up -d
docker compose logs -f ghost
```

Optional profiles:

```bash
docker compose --profile=analytics up -d
docker compose --profile=activitypub up -d
```

## Development Workflow

1. Create a branch from `main`.
2. Make your change.
3. Validate before opening a PR:
   ```bash
   docker compose config -q          # compose file is valid
   shellcheck scripts/*.sh           # shell scripts lint clean
   ```
4. Update `CHANGELOG.md` under the next version header. Releases are cut by
   pushing a `vX.Y.Z` tag, and `release.yml` builds the release notes from that
   section - if it is missing, the release fails.
5. Open a pull request and fill out the template.

## Changelog

- Follow the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format.
- Categorize changes: Added, Changed, Deprecated, Removed, Fixed, Security.
- Keep entries free of personal infrastructure details (domains, providers,
  hostnames). The public changelog must remain vendor-neutral.

## Commit Messages

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add optional Redis service
fix: correct nginx Origin header for Ghost admin
docs: clarify SSL setup
ci: pin trivy action to digest
chore: bump ghost image
```

## Pull Requests

- Keep PRs focused: one feature or fix per PR.
- Reference any related issues.
- All CI checks must pass before merge.
- Do not commit secrets, real domains, `.env` files, or `data/` contents.
