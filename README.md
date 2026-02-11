# Sale-ADS Organization Shared Workflows

This repository contains reusable GitHub Actions workflows shared across all Sale-ADS repositories.

## Available Workflows

### Snyk Security Scan

**File:** `.github/workflows/snyk-security.yml`

Centralized security scanning workflow using Snyk. Supports 4 scan types:

| Scan Type | Description | Default |
|-----------|-------------|---------|
| Open Source | Dependency vulnerability scanning | Enabled |
| Code (SAST) | Static application security testing | Enabled |
| Container | Docker image vulnerability scanning | Enabled |
| IaC | Infrastructure as Code misconfiguration | Disabled |

### Usage

#### Java/Maven Projects

```yaml
name: Snyk Security
on:
  pull_request:
    branches: [develop, main]
  push:
    branches: [develop, main]
  schedule:
    - cron: '0 6 * * 1'
jobs:
  security-scan:
    uses: Sale-ADS/.github/.github/workflows/snyk-security.yml@main
    with:
      language: java
      java-version: '21'
      java-distribution: 'corretto'
      docker-image-name: my-service-name
      severity-threshold: high
      continue-on-error: true
    secrets:
      snyk-token: ${{ secrets.SNYK_TOKEN }}
```

#### Node.js/TypeScript Projects

```yaml
name: Snyk Security
on:
  pull_request:
    branches: [dev, development, develop, qa, main]
  push:
    branches: [dev, development, develop, main]
  schedule:
    - cron: '0 6 * * 1'
jobs:
  security-scan:
    uses: Sale-ADS/.github/.github/workflows/snyk-security.yml@main
    with:
      language: node
      node-version: '20'
      docker-image-name: my-service-name
      severity-threshold: high
      continue-on-error: true
    secrets:
      snyk-token: ${{ secrets.SNYK_TOKEN }}
```

#### Libraries (No Docker)

```yaml
name: Snyk Security
on:
  pull_request:
    branches: [dev, development, develop, main]
  push:
    branches: [dev, development, develop, main]
  schedule:
    - cron: '0 6 * * 1'
jobs:
  security-scan:
    uses: Sale-ADS/.github/.github/workflows/snyk-security.yml@main
    with:
      language: node
      node-version: '20'
      scan-container: false
      severity-threshold: high
      continue-on-error: true
    secrets:
      snyk-token: ${{ secrets.SNYK_TOKEN }}
```

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `language` | Yes | - | `java` or `node` |
| `java-version` | No | `21` | Java version |
| `java-distribution` | No | `corretto` | Java distribution |
| `node-version` | No | `20` | Node.js version |
| `snyk-org` | No | `sale-ads` | Snyk organization ID |
| `severity-threshold` | No | `high` | `low`, `medium`, `high`, or `critical` |
| `scan-open-source` | No | `true` | Enable dependency scanning |
| `scan-code` | No | `true` | Enable SAST scanning |
| `scan-container` | No | `true` | Enable Docker scanning |
| `scan-iac` | No | `false` | Enable IaC scanning |
| `docker-image-name` | No | `''` | Docker image name |
| `dockerfile-path` | No | `./Dockerfile` | Dockerfile path |
| `continue-on-error` | No | `true` | Report-only mode |

### Secrets

| Secret | Required | Description |
|--------|----------|-------------|
| `snyk-token` | Yes | Snyk API token (org-level secret `SNYK_TOKEN`) |

### Prerequisites

1. `SNYK_TOKEN` must be configured as an organization-level secret
2. GitHub Advanced Security enabled for SARIF upload (optional)

### Rollout Phases

| Phase | severity-threshold | continue-on-error | Description |
|-------|--------------------|--------------------|-------------|
| 1 - Baseline | `high` | `true` | Report only, establish baseline |
| 2 - Enforce Critical | `critical` | `false` | Block PRs with critical vulns |
| 3 - Enforce High | `high` | `false` | Block PRs with high+ vulns |
| 4 - Continuous | `high` | `false` | Auto-fix, monitoring |
