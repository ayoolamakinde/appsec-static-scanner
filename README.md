# AppSec Static Scanner

Comprehensive, reusable GitHub Actions workflows for static application security scanning. A unified platform for SCA, SAST, IAC, and secrets detection.

## 🎯 Overview

This repository provides production-ready, reusable workflows that can be called from any repository to enforce security scanning at every stage of the development lifecycle.

**Supported Scans:**
- 🔍 **SCA** (Software Composition Analysis) - Trivy
- 🔎 **SAST** (Static Application Security Testing) - Semgrep
- 🏗️ **IAC** (Infrastructure-as-Code) - Checkov
- 🔐 **Secrets Detection** - TruffleHog

## 📋 Workflows

### 1. SCA Scan Workflow
**File:** `.github/workflows/sca-scan.yml`
**Docs:** [SCA-SCAN.md](docs/SCA-SCAN.md)

Scans dependencies and supply chain vulnerabilities using **Trivy**. Generates SBOM, detects CVEs, with configurable severity thresholds.

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      severity: 'CRITICAL,HIGH,MEDIUM,LOW'
      fail_on_severity: 'HIGH'
      generate_sbom: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. SAST Scan Workflow
**File:** `.github/workflows/sast-scan.yml`
**Docs:** [SAST-SCAN.md](docs/SAST-SCAN.md)

Analyzes source code for vulnerabilities using **Semgrep**. Supports multiple languages, custom rules, SARIF upload to GitHub Security.

```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'python,javascript,go,java'
      rules_path: 'p/owasp-top-ten'
      fail_on_severity: 'ERROR'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 3. IAC Scan Workflow
**File:** `.github/workflows/iac-scan.yml`
**Docs:** [IAC-SCAN.md](docs/IAC-SCAN.md)

Scans infrastructure code for misconfigurations using **Checkov**. Supports Terraform, CloudFormation, Kubernetes, Docker, Bicep, ARM.

```yaml
jobs:
  iac:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'terraform,kubernetes'
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 4. Secrets Scan Workflow
**File:** `.github/workflows/secrets-scan.yml`
**Docs:** [SECRETS-SCAN.md](docs/SECRETS-SCAN.md)

Detects exposed credentials and sensitive information using **TruffleHog**. Scans filesystem and git history, verifies findings.

```yaml
jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      scan_type: 'filesystem,git'
      only_verified: false
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 5. Notification Workflow (Reusable)
**File:** `.github/workflows/notify.yml`
**Docs:** [NOTIFICATIONS.md](docs/NOTIFICATIONS.md)

Unified notification system supporting **Microsoft Teams** and **Slack**. Called from other workflows to send findings to multiple channels.

```yaml
- name: Notify Results
  if: always()
  uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
  with:
    notification_type: 'vulnerability'
    severity: 'CRITICAL'
    tool_name: 'SCA'
    finding_count: 3
  secrets:
    TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
    SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

## 🚀 Quick Start

### 1. Add Workflows to Your Repository

Create `.github/workflows/security.yml` in your repository:

```yaml
name: Security Scans

on:
  pull_request:
    paths-ignore:
      - '**.md'
      - 'docs/**'
  push:
    branches:
      - main
      - develop

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      fail_on_severity: 'ERROR'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  iac:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. Configure Secrets

Add to your repository secrets:
- `SECURITY_WEBHOOK`: Webhook URL for notifications (Teams or Slack)
- `TEAMS_WEBHOOK`: Microsoft Teams webhook (optional)
- `SLACK_WEBHOOK`: Slack webhook (optional)

Instructions: [Setup Guide](docs/SETUP.md)

### 3. Customize Configuration (Optional)

Edit the `with:` parameters in your workflow to customize:
- Severity thresholds
- Languages to scan
- Frameworks to check
- Output formats
- And more...

See tool-specific documentation for all options.

## 📚 Documentation

**Tool-Specific Guides:**
- [SCA Scan](docs/SCA-SCAN.md) - Software Composition Analysis with Trivy
- [SAST Scan](docs/SAST-SCAN.md) - Static Application Security Testing with Semgrep
- [IAC Scan](docs/IAC-SCAN.md) - Infrastructure as Code scanning with Checkov
- [Secrets Scan](docs/SECRETS-SCAN.md) - Credential detection with TruffleHog

**Setup & Integration:**
- [Setup Guide](docs/SETUP.md) - Initial setup and webhook configuration
- [Notifications](docs/NOTIFICATIONS.md) - Teams and Slack integration

**Operations:**
- [Best Practices](docs/BEST-PRACTICES.md) - Organization-wide implementation strategy
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues and solutions

## 🔧 Features

### ✅ Reusable Workflows
- Call from any repository
- Consistent scanning across projects
- Single source of truth for security policies

### ✅ Flexible Configuration
- Per-scan customization
- Severity thresholds
- Framework selection
- Fail/continue on findings

### ✅ Rich Reporting
- PR comments with detailed findings
- Teams notifications for critical issues
- S3 artifact storage
- Multiple output formats (JSON, CSV, HTML)

### ✅ Performance Optimized
- Tool caching for faster scans
- Parallel scan execution
- Minimal false positives
- Configurable timeouts

### ✅ Production Ready
- Configurable approval gates
- Compliance reporting
- Audit logging
- Multi-environment support

## 🛠️ Supported Tools

| Scan Type | Tool | Purpose | Languages/Frameworks |
|-----------|------|---------|----------------------|
| **SCA** | [Trivy](https://aquasecurity.github.io/trivy/) | Dependency vulnerabilities | Python, Node.js, Go, Java, Ruby, PHP, .NET, Rust |
| **SAST** | [Semgrep](https://semgrep.dev/) | Code vulnerabilities | Python, JavaScript, TypeScript, Go, Java, C#, Ruby, PHP, Rust |
| **IAC** | [Checkov](https://www.checkov.io/) | Infrastructure misconfigs | Terraform, CloudFormation, Kubernetes, Docker, Bicep, ARM |
| **Secrets** | [TruffleHog](https://trufflesecurity.com/) | Credential detection | 800+ secret types |

## 📊 Workflow Features

### Universal Features (All Workflows)
- **PR Comments** - Detailed findings reported on pull requests
- **GitHub Issues** - Automatic issue creation for tracking
- **Artifacts** - Scan results in multiple formats (JSON, SARIF, CSV)
- **Notifications** - Teams and Slack alerts
- **Severity Filtering** - CRITICAL, HIGH, MEDIUM, LOW
- **Configurable Failure** - Fail on specific severity or all findings
- **Retry Logic** - Automatic retry with exponential backoff
- **Caching** - Tool caching for improved performance

### Tool-Specific Features

**SCA (Trivy):**
- SBOM generation (CycloneDX)
- Multi-format output (JSON, SARIF, table)
- Directory and package-specific scanning
- Severity-based failure thresholds

**SAST (Semgrep):**
- Multi-language analysis
- SARIF upload to GitHub Security tab
- Custom rule support
- Language filtering

**IAC (Checkov):**
- Multi-framework scanning
- Selective check execution
- Compliance standard mapping (CIS, PCI-DSS, SOC 2, HIPAA)
- Compact/quiet output modes

**Secrets (TruffleHog):**
- 800+ detector types
- Verified vs unverified detection
- Git history scanning
- Custom detector configuration
- Entropy threshold tuning

## 🔐 Security

- No credentials stored in workflows (use GitHub Secrets)
- Secrets masked in logs automatically
- Support for private container registries
- Audit trail of all security scans
- Workflow permissions follow least-privilege principle

## 📖 Examples

### SCA with SBOM Generation
```yaml
with:
  severity: 'CRITICAL,HIGH,MEDIUM,LOW'
  fail_on_severity: 'CRITICAL'
  generate_sbom: true
  output_format: 'sarif,json,cyclonedx'
```

### SAST with Custom Rules
```yaml
with:
  languages: 'python,javascript'
  rules_path: 'p/owasp-top-ten'
  config_file: '.semgrep.yml'
  fail_on_severity: 'ERROR'
```

### IAC Multi-Framework
```yaml
with:
  frameworks: 'terraform,kubernetes,dockerfile'
  fail_on_severity: 'HIGH'
  download_external_modules: true
```

### Secrets with History Scan
```yaml
with:
  scan_type: 'git'
  scan_history: true
  only_verified: false
  entropy_threshold: 4.0
```

## 🆘 Getting Help

**Documentation:** Check the [docs](docs/) directory for comprehensive guides

**Issues:** Found a bug? Report it in [GitHub Issues](https://github.com/ayoolamakinde/appsec-static-scanner/issues)

**Troubleshooting:** See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for common issues
