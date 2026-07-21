# Security Policy

## Supported Versions

| Version | Supported          |
|---------|--------------------|
| latest  | :white_check_mark: |
| < latest| :x:                |

Only the latest tagged release and the `main` branch receive security fixes.

## Reporting a Vulnerability

**Do not open a public issue for security vulnerabilities.**

Please report vulnerabilities privately:

1. Go to the [Security Advisories](https://github.com/NX1X/self-hosted-ghost-blog/security/advisories) page.
2. Click "Report a vulnerability".
3. Provide a clear description, affected files or services, and reproduction steps.

You can expect an initial response within 72 hours. If the report is confirmed,
a fix will be released and credited in the changelog unless you request
otherwise.

## Scope

This repository is deployment and orchestration tooling (Docker Compose, nginx
configuration, helper scripts). Vulnerabilities in upstream images
(Ghost, MySQL, nginx) should be reported to their respective projects. Issues in
how this project configures or exposes those services are in scope here.

## Security Practices

- Dependencies and base images are monitored and updated automatically via
  Renovate and Dependabot.
- CI scans the repository on every push and pull request: Gitleaks (secret
  scanning), Trivy (image and filesystem CVEs and misconfigurations), CodeQL
  (SAST), zizmor (GitHub Actions auditing), and dependency-review.
- No secrets or credentials are committed to the repository. All configuration
  is supplied at runtime via a local, gitignored `.env` file.
- All Docker images and GitHub Actions are pinned to immutable digests.
