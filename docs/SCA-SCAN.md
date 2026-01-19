# SCA Scan - Software Composition Analysis

## Overview

The SCA (Software Composition Analysis) workflow scans your project dependencies for known vulnerabilities using **Trivy**. It identifies vulnerable packages in your supply chain and generates comprehensive reports.

**Supported Languages:**
- Python (pip, pipenv, poetry, requirements.txt)
- JavaScript/Node.js (npm, yarn, pnpm)
- Java (Maven, Gradle)
- Go (go.mod, go.sum)
- Ruby (Gemfile, Gemfile.lock)
- PHP (composer.json, composer.lock)
- .NET (packages.config, .csproj)
- Rust (Cargo.lock)
- And many more...

## Table of Contents

1. [Features](#features)
2. [Setup](#setup)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Examples](#examples)
6. [Outputs](#outputs)
7. [Troubleshooting](#troubleshooting)

---

## Features

✅ **Multi-Language Support** - Scan dependencies across all major languages
✅ **SBOM Generation** - Generate Software Bill of Materials in CycloneDX format
✅ **Multiple Output Formats** - JSON, SARIF, table, CycloneDX
✅ **Caching** - Speed up scans with dependency caching
✅ **PR Comments** - Automatic PR comments with results
✅ **Issue Creation** - Create GitHub issues for vulnerabilities
✅ **Severity Filtering** - Configure which severity levels to report
✅ **Custom Skip Paths** - Exclude specific directories
✅ **Retry Logic** - Automatic retries on failure

---

## Setup

### 1. Basic Setup

Add to your repository (`.github/workflows/sca-scan.yml`):

```yaml
name: SCA Scan

on:
  pull_request:
    paths:
      - 'package.json'
      - 'requirements.txt'
      - 'go.mod'
      - 'pom.xml'
  push:
    branches:
      - main

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. Configure Secrets (Optional)

For notifications, add to repository secrets:
- `SECURITY_WEBHOOK` - Teams or Slack webhook URL

---

## Usage

### Basic Usage

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
```

### With Custom Configuration

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      severity: 'CRITICAL,HIGH'
      fail_on_severity: 'HIGH'
      skip_dirs: 'node_modules,vendor,tests'
      generate_sbom: true
      comment_pr: true
      create_issue: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

---

## Configuration

### Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `severity` | string | `HIGH,CRITICAL` | Severity levels to report (comma-separated) |
| `fail_on_severity` | string | `CRITICAL` | Fail on this severity or higher |
| `skip_dirs` | string | `node_modules,vendor,.venv,venv,.git,dist,build` | Directories to exclude |
| `output_format` | string | `json` | Output format: json, sarif, table, cyclonedx |
| `scan_hidden` | boolean | `false` | Scan hidden files |
| `generate_sbom` | boolean | `true` | Generate SBOM (CycloneDX) |
| `max_retries` | number | `3` | Maximum retry attempts |
| `fail_on_finding` | boolean | `true` | Fail workflow if vulnerabilities found |
| `continue_on_error` | boolean | `false` | Continue even if scan fails |
| `upload_artifacts` | boolean | `true` | Upload results as artifacts |
| `create_issue` | boolean | `false` | Create GitHub issue |
| `comment_pr` | boolean | `true` | Comment on PR |

### Severity Levels

- **CRITICAL** - Immediate security risk, requires immediate patching
- **HIGH** - Significant security risk, patch recommended soon
- **MEDIUM** - Moderate risk, plan remediation
- **LOW** - Minor risk or less critical impact

---

## Examples

### Example 1: Fail on Critical Vulnerabilities

```yaml
name: SCA - Fail on Critical

on: [pull_request, push]

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      severity: 'CRITICAL,HIGH,MEDIUM,LOW'
      fail_on_severity: 'CRITICAL'
      fail_on_finding: true
```

**Behavior:**
- Reports all severity levels
- Fails workflow only if CRITICAL found
- Doesn't fail on HIGH, MEDIUM, LOW

### Example 2: Generate SBOM Only

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      generate_sbom: true
      output_format: 'cyclonedx'
      fail_on_finding: false
```

**Behavior:**
- Generates CycloneDX SBOM
- Never fails workflow
- Uploads artifacts

### Example 3: Python-Only Scan

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      skip_dirs: 'node_modules,vendor,.git'
      severity: 'HIGH,CRITICAL'
```

**Behavior:**
- Skips node_modules (JavaScript)
- Reports HIGH and CRITICAL
- Auto-detects Python packages

### Example 4: Strict Security Posture

```yaml
jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      severity: 'CRITICAL,HIGH,MEDIUM,LOW'
      fail_on_severity: 'MEDIUM'
      create_issue: true
      comment_pr: true
```

**Behavior:**
- Reports all findings
- Fails on MEDIUM or higher
- Creates issues and PR comments
- Full transparency

### Example 5: Development vs Production

**Development branch:**
```yaml
on:
  pull_request:
    branches:
      - develop

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_finding: false
      severity: 'CRITICAL,HIGH'
```

**Production branch:**
```yaml
on:
  pull_request:
    branches:
      - main

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_finding: true
      fail_on_severity: 'HIGH'
      severity: 'CRITICAL,HIGH,MEDIUM'
```

---

## Outputs

### GitHub Artifacts

**File:** `sca-results.json` (or other format)

Example output:
```json
{
  "Results": [
    {
      "Target": "package.json",
      "Type": "npm",
      "Vulnerabilities": [
        {
          "VulnerabilityID": "CVE-2024-12345",
          "PkgName": "express",
          "InstalledVersion": "4.17.0",
          "FixedVersion": "4.17.1",
          "Title": "Denial of Service",
          "Severity": "HIGH"
        }
      ]
    }
  ]
}
```

### PR Comment

```
## 🔍 SCA Scan Results

### Summary
| Severity | Count |
|----------|-------|
| 🔴 CRITICAL | 0 |
| 🟠 HIGH | 3 |
| 🟡 MEDIUM | 7 |
| 🔵 LOW | 2 |

### Details
View detailed scan results in the artifacts.

_📊 Scan: Trivy (SCA - Dependency Vulnerabilities)_
```

### SBOM Output (CycloneDX)

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.4",
  "components": [
    {
      "type": "library",
      "name": "express",
      "version": "4.17.0",
      "purl": "pkg:npm/express@4.17.0"
    }
  ]
}
```

---

## Common Use Cases

### Use Case 1: Automatic Dependency Updates

```yaml
name: SCA - Auto Update Dependencies

on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_finding: false
      create_issue: true
```

Creates weekly reports and issues for vulnerabilities.

### Use Case 2: Security Gate for Releases

```yaml
name: SCA - Release Gate

on:
  push:
    tags:
      - 'v*'

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'CRITICAL'
      severity: 'CRITICAL,HIGH,MEDIUM'
```

Prevents releases with critical vulnerabilities.

### Use Case 3: Compliance Reporting

```yaml
name: SCA - Compliance Report

on:
  schedule:
    - cron: '0 0 1 * *'  # Monthly

jobs:
  sca:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      generate_sbom: true
      output_format: 'cyclonedx'
      upload_artifacts: true
```

Generates monthly SBOM for compliance.

---

## Troubleshooting

### Issue: "No vulnerabilities found" but workflow still fails

**Cause:** Scan may have timed out or failed silently

**Solution:**
```yaml
with:
  max_retries: 5
  continue_on_error: false
```

### Issue: Skip paths not working

**Cause:** Incorrect path format

**Solution:**
```yaml
# ❌ Wrong
skip_dirs: 'node_modules, vendor'

# ✅ Correct (no spaces)
skip_dirs: 'node_modules,vendor'
```

### Issue: SBOM generation fails

**Cause:** Unsupported language or format

**Solution:**
```yaml
# Use JSON format instead
output_format: 'json'
generate_sbom: false
```

### Issue: Artifacts not uploaded

**Cause:** Scan failed before upload step

**Solution:**
```yaml
with:
  continue_on_error: true
  upload_artifacts: true
```

---

## Best Practices

1. **Fail Fast on Production**
   ```yaml
   fail_on_severity: 'HIGH'  # For main branch
   ```

2. **Allow Minor Issues in Development**
   ```yaml
   fail_on_severity: 'CRITICAL'  # For develop branch
   ```

3. **Always Generate SBOM**
   ```yaml
   generate_sbom: true
   ```

4. **Create Issues for Tracking**
   ```yaml
   create_issue: true
   ```

5. **Schedule Regular Scans**
   ```yaml
   on:
     schedule:
       - cron: '0 0 * * 0'
   ```

---

## See Also

- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [SAST Scan](SAST-SCAN.md)
- [IAC Scan](IAC-SCAN.md)
- [Secrets Scan](SECRETS-SCAN.md)
- [Notifications](NOTIFICATIONS.md)

---

**Last Updated:** January 18, 2026
