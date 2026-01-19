# Secrets Scan - Credential Detection

## Overview

The Secrets Scan workflow detects exposed credentials, API keys, tokens, and other sensitive data in your codebase using **TruffleHog**. It can scan both current code and git history, making it essential for preventing credential leaks.

**Detects Over 800+ Secret Types:**
- AWS Keys (Access Key ID, Secret Access Key)
- GitHub Personal Access Tokens
- Private Keys (RSA, PEM, SSH)
- API Keys (Stripe, Twilio, SendGrid, etc.)
- Database Credentials (MongoDB, MySQL, PostgreSQL)
- OAuth Tokens
- Slack Webhooks
- And many more...

**Detection Methods:**
- Pattern Matching (signatures)
- Entropy Analysis (randomness)
- Verification (found in wild databases)
- Custom Detectors

## Table of Contents

1. [Features](#features)
2. [Setup](#setup)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Examples](#examples)
6. [Outputs](#outputs)
7. [Custom Detectors](#custom-detectors)
8. [Remediation](#remediation)
9. [Troubleshooting](#troubleshooting)

---

## Features

✅ **800+ Secret Types** - Comprehensive detector coverage
✅ **Verified vs Unverified** - Distinguish between confirmed and suspected leaks
✅ **Git History Scanning** - Find secrets committed in the past
✅ **Custom Detectors** - Define organization-specific patterns
✅ **Entropy Threshold** - Tune sensitivity for false positives
✅ **Configurable Detectors** - Enable/disable specific detectors
✅ **Critical Issues** - Auto-create urgent issues for verified secrets
✅ **PR Comments** - Notify developers of findings
✅ **Suppression Support** - Skip known false positives

---

## Setup

### 1. Basic Setup

Add to your repository (`.github/workflows/secrets-scan.yml`):

```yaml
name: Secrets Scan

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. Configure Suppressions (Optional)

Create `.trufflehog.yaml`:

```yaml
detectors:
  - type: Slack
    enabled: true
  - type: GitHub
    enabled: true
  - type: AWS
    enabled: true

skip_paths:
  - tests
  - docs
  - .git

entropy_threshold: 4.0

only_verified: false
```

---

## Usage

### Basic Usage

```yaml
jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
```

### Verify-Only Mode

```yaml
jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      only_verified: true  # Only report confirmed leaks
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### Full History Scan

```yaml
jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      scan_type: 'git'  # Scan entire git history
      scan_history: true
      max_depth: 1000000  # Scan entire history
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

---

## Configuration

### Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `scan_type` | string | `filesystem,git` | Scan type: filesystem, git, or both |
| `include_detectors` | string | `` | Detectors to enable (comma-separated) |
| `exclude_detectors` | string | `` | Detectors to disable |
| `skip_paths` | string | `.git,node_modules,vendor,build,dist` | Paths to exclude |
| `entropy_threshold` | number | `4.0` | Entropy threshold (3.0-8.0) |
| `scan_history` | boolean | `true` | Scan git history |
| `only_verified` | boolean | `false` | Report only verified secrets |
| `max_depth` | number | `1000000` | Max git commits to scan |
| `max_retries` | number | `3` | Retry attempts |
| `fail_on_finding` | boolean | `true` | Fail if secrets found |
| `continue_on_error` | boolean | `false` | Continue even if scan fails |
| `upload_artifacts` | boolean | `true` | Upload results |
| `create_issue` | boolean | `true` | Create GitHub issue |
| `comment_pr` | boolean | `true` | Comment on PR |
| `config_file` | string | `.trufflehog.yaml` | Custom config file |

### Available Detectors

**Cloud Providers:**
- AWS, Azure, GCP
- DigitalOcean, Alibaba, Heroku

**Authentication:**
- GitHub, GitLab, Bitbucket
- AWS Keys, Google Service Account
- SSH Private Keys

**Communication:**
- Slack, Discord, Teams
- SendGrid, Twilio, Mailchimp

**Databases:**
- MongoDB, MySQL, PostgreSQL
- Firebase, Supabase, PlanetScale

**APIs & Services:**
- Stripe, Twilio, Datadog
- PagerDuty, Notion, Confluence

---

## Examples

### Example 1: Standard PR Check

```yaml
name: Secrets - PR Check

on: [pull_request]

jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      scan_type: 'filesystem'
      only_verified: false
      entropy_threshold: 4.0
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

**Behavior:**
- Scans only new/modified files
- Reports verified + unverified
- Fails on any finding

### Example 2: Verified Secrets Only

```yaml
name: Secrets - Critical Only

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      only_verified: true
      fail_on_finding: true
      create_issue: true
```

**Behavior:**
- Only reports confirmed leaks
- Creates critical issues
- Never allows verified secrets

### Example 3: Full History Audit

```yaml
name: Secrets - Full Audit

on:
  workflow_dispatch:  # Manual trigger

jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      scan_type: 'git'
      scan_history: true
      max_depth: 1000000
      entropy_threshold: 3.5
      create_issue: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

**Behavior:**
- Scans entire git history
- Lower entropy threshold (more sensitive)
- Creates issues for all findings

### Example 4: Custom Detector Config

Create `.trufflehog.yaml`:
```yaml
detectors:
  - type: AWS
    enabled: true
  - type: GitHub
    enabled: true
  - type: Slack
    enabled: true
  - type: Stripe
    enabled: false  # Disabled
```

Then use:
```yaml
jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      config_file: '.trufflehog.yaml'
      fail_on_finding: true
```

### Example 5: Lenient Development

```yaml
on:
  pull_request:
    branches:
      - develop

jobs:
  secrets:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: false
      comment_pr: true
      entropy_threshold: 5.0  # Higher = less sensitive
```

**Behavior:**
- Never fails workflow
- Comments on PR
- Higher entropy threshold (fewer false positives)

---

## Outputs

### GitHub Artifacts

**File:** `secrets-scan-results.json`

Example:
```json
{
  "found_secrets": [
    {
      "verified": true,
      "finding": {
        "data": "AKIA2JXMPL4V7EXAMPLE",
        "type": "AWS Access Key",
        "raw": "aws_access_key_id = AKIA2JXMPL4V7EXAMPLE"
      },
      "file_path": "config/.env",
      "line_number": 42,
      "remediation": "Rotate AWS access key immediately"
    },
    {
      "verified": false,
      "finding": {
        "data": "sk_live_4eC39HqLyjWDarhtT657tMo5k",
        "type": "Stripe API Key",
        "entropy": 4.8
      },
      "file_path": "app/payments.js",
      "line_number": 15
    }
  ],
  "summary": {
    "total_found": 2,
    "verified": 1,
    "unverified": 1,
    "high_entropy": 3
  }
}
```

### PR Comment

```
## 🔐 Secrets Scan Results

### ⚠️ CRITICAL: Verified Secrets Found!

**1 verified secret detected (AWS Access Key)**

| Type | File | Line | Action |
|------|------|------|--------|
| AWS Key | config/.env | 42 | 🔴 ROTATE IMMEDIATELY |

### ℹ️ Unverified Findings

1 unverified finding (Stripe API Key - app/payments.js:15)

**IMPORTANT**: If verified secrets were committed:
1. Rotate the credential immediately
2. Check if used elsewhere
3. Review access logs for misuse
4. Force-push history changes (if safe)

_🔐 Scan: TruffleHog (Secrets Detection)_
```

---

## Secret Types

### AWS Access Keys

```
Pattern: AKIA + 16 alphanumeric characters
Example: AKIAIOSFODNN7EXAMPLE
Risk: Full AWS account access
Action: Rotate immediately
```

**Remediation:**
```bash
# 1. Rotate the key
aws iam create-access-key --user-name user

# 2. Delete old key
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE

# 3. Remove from git
git filter-branch --tree-filter 'grep -r "AKIA" && exit 1; true' -- --all
```

### GitHub Personal Access Token

```
Pattern: ghp_ or ghu_ or ghs_ prefixes
Example: ghp_1234567890abcdefghijklmnopqrstuv
Risk: Repository access, code review, secret exposure
Action: Revoke immediately
```

**Remediation:**
```bash
# 1. Revoke in GitHub Settings > Developer Settings > Personal Access Tokens
# 2. Review audit logs for usage
# 3. Remove from code and git history
git filter-branch --tree-filter 'grep -v "ghp_" ' -- --all
```

### Private SSH Key

```
Pattern: BEGIN RSA/EC/OPENSSH PRIVATE KEY
Example: -----BEGIN RSA PRIVATE KEY-----
Risk: Server access, authentication bypass
Action: Revoke and regenerate
```

**Remediation:**
```bash
# 1. Remove old key from authorized_hosts on servers
# 2. Generate new key pair
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519

# 3. Update authorized_keys
# 4. Remove from git
git filter-branch --tree-filter 'find . -name "*.pem" -delete' -- --all
```

---

## Custom Detectors

### Create Custom Pattern

**File:** `.trufflehog.yaml`

```yaml
custom_keywords:
  - pattern: 'internal_api_key'
    keywords:
      - 'internal_api_key'
    regex: 'internal_api_key["\']?[:=\s]+["\']?[A-Za-z0-9]{32}["\']?'
    entropy_min: 3.0
    entropy_max: 8.0
```

### Example: Custom Organization Secret

```yaml
custom_keywords:
  - pattern: 'acme_corp_token'
    keywords:
      - 'acme_token'
      - 'acme_secret'
    regex: 'acme_[a-z0-9]{40}'
    entropy_min: 3.5
```

---

## Suppression

### Suppress False Positive

**File:** `.trufflehog-suppressions.yaml`

```yaml
suppressions:
  - hash: "abc123def456"  # Finding hash
    justification: "False positive - test data"
    author: "john.doe"
    date: "2024-01-18"
```

### Inline Suppression

```python
# trufflehog:ignore=aws
AWS_KEY = "AKIA_TEST_DATA"
```

---

## Best Practices

1. **Zero-Trust Approach**
   ```yaml
   only_verified: false  # Report all potential secrets
   fail_on_finding: true
   ```

2. **Rotate Immediately**
   - Verified secret = critical incident
   - Rotate within minutes
   - Audit logs for misuse

3. **Regular History Scans**
   ```yaml
   # Monthly full history audit
   scan_type: 'git'
   max_depth: 1000000
   ```

4. **Prevent Future Leaks**
   - Use GitHub secret scanning
   - Use environment variables
   - Use secret management tools

5. **Document Suppressions**
   ```yaml
   suppressions:
     - hash: "abc123"
       justification: "Mock credential for testing"
       author: "security-team"
   ```

---

## Remediation Guide

### If Secret Found in PR

1. **STOP** - Don't merge
2. **Evaluate Risk** - Verified or unverified?
3. **Communicate** - Notify team
4. **Rotate** - Change credential immediately
5. **Audit** - Check access logs
6. **Remove** - Force-push with history rewrite
7. **Monitor** - Watch for misuse

### Git History Cleanup

```bash
# WARNING: Rewrites history, coordinate with team

# Remove single file
git filter-branch --tree-filter 'rm -f .env' -- --all

# Remove pattern
git filter-branch --tree-filter 'find . -name "*.pem" -delete' -- --all

# Notify team
git push -f origin main
```

---

## Troubleshooting

### Issue: Too many false positives

**Cause:** Entropy threshold too low

**Solution:**
```yaml
with:
  entropy_threshold: 5.0  # Increase from 4.0
```

### Issue: Missed secrets in history

**Cause:** max_depth too low

**Solution:**
```yaml
with:
  scan_type: 'git'
  max_depth: 1000000  # Full history
```

### Issue: Specific detector not working

**Cause:** Detector disabled or not matching

**Solution:**
```yaml
with:
  include_detectors: 'AWS,GitHub,Stripe'
  exclude_detectors: ''
```

### Issue: Custom detector not loading

**Cause:** YAML syntax error in `.trufflehog.yaml`

**Solution:**
```bash
# Validate locally
trufflehog filesystem . --config .trufflehog.yaml
```

---

## See Also

- [TruffleHog Documentation](https://trufflesecurity.com)
- [Detector Types](https://github.com/trufflesecurity/trufflehog#detectors)
- [SCA Scan](SCA-SCAN.md)
- [SAST Scan](SAST-SCAN.md)
- [IAC Scan](IAC-SCAN.md)

---

**Last Updated:** January 18, 2026
