# Setup Guide

## Quick Start

### 1. Create Repository

Create a new repository named `appsec-static-scanner` in your GitHub organization.

```bash
# Clone the repo locally
git clone git@github.com:your-org/appsec-static-scanner.git
cd appsec-static-scanner
```

### 2. Configure Webhooks

#### Teams Webhook
1. Open Microsoft Teams
2. Right-click on channel → Connectors
3. Search "Incoming Webhook"
4. Click "Configure"
5. Name: "Security Alerts"
6. Copy URL
7. In GitHub: Settings → Secrets → New organization secret
8. Name: `TEAMS_WEBHOOK`
9. Paste webhook URL

#### Slack Webhook
1. Open Slack Workspace
2. Browse Apps → Incoming Webhooks
3. Create New Webhook for your channel
4. Copy URL
5. In GitHub: Settings → Secrets → New organization secret
6. Name: `SLACK_WEBHOOK`
7. Paste webhook URL

### 3. Enable in Your Repositories

In each repository that needs security scanning, create `.github/workflows/security.yml`:

```yaml
name: Security Scanning

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
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  sast:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      fail_on_severity: 'ERROR'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}

  secrets:
    uses: your-org/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

**Note:** Each workflow runs independently and in parallel. See tool-specific documentation for configuration options.

## Configuration Options

### Workflow-Specific Configuration

Each workflow accepts input parameters for customization. Configure them in the `with:` block:

**SCA Configuration:**
```yaml
jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      severity: 'CRITICAL,HIGH,MEDIUM,LOW'
      fail_on_severity: 'HIGH'
      generate_sbom: true
      skip_dirs: 'tests,examples,vendor'
```

**SAST Configuration:**
```yaml
jobs:
  sast:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'python,javascript,go'
      rules_path: 'p/owasp-top-ten'
      fail_on_severity: 'ERROR'
```

**IAC Configuration:**
```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'terraform,kubernetes'
      fail_on_severity: 'HIGH'
```

**Secrets Configuration:**
```yaml
jobs:
  secrets:
    uses: your-org/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      scan_type: 'filesystem,git'
      entropy_threshold: 4.0
      fail_on_finding: true
```

For complete parameter reference, see the tool-specific documentation:
- [SCA-SCAN.md](SCA-SCAN.md)
- [SAST-SCAN.md](SAST-SCAN.md)
- [IAC-SCAN.md](IAC-SCAN.md)
- [SECRETS-SCAN.md](SECRETS-SCAN.md)

### Custom Tool Configuration

Store custom configuration files in your repository:

**Semgrep Custom Rules** (`.semgrep.yml`):
```yaml
rules:
  - id: custom-rule
    pattern: vulnerable_function(...)
    message: Custom vulnerability detected
    severity: ERROR
```

**Checkov Configuration** (`.checkov.yaml`):
```yaml
framework:
  - terraform
  - kubernetes

check:
  - CKV_AWS_1
  - CKV_K8S_8
```

**TruffleHog Configuration** (`.trufflehog.yaml`):
```yaml
detectors:
  - type: AWS
    enabled: true
  - type: GitHub
    enabled: true

entropy_threshold: 4.0
```

## Integration Patterns

### Pattern 1: Scan on Every PR (Strict)

```yaml
on:
  pull_request:

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'HIGH'

  sast:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      fail_on_severity: 'ERROR'

  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      fail_on_severity: 'HIGH'

  secrets:
    uses: your-org/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: true
```

### Pattern 2: Scheduled Weekly Audit

```yaml
on:
  schedule:
    - cron: '0 2 * * 1'  # Monday 2 AM UTC

jobs:
  audit:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'MEDIUM'
```

### Pattern 3: Language-Specific SAST

```yaml
on:
  pull_request:
    paths:
      - '**.py'
      - 'requirements.txt'

jobs:
  sast-python:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'python'
      rules_path: 'p/security-audit'
      fail_on_severity: 'ERROR'
```

### Pattern 4: Lenient Development Branch

```yaml
on:
  pull_request:
    branches: [develop]

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_finding: false  # Warn only
      comment_pr: true
```

## Required Permissions

Ensure your GitHub organization/repository has:

✅ **Actions read/write**
✅ **Pull requests write** (for PR comments)
✅ **Issues write** (for creating issues)
✅ **Contents read**
✅ **ID token write** (for AWS OIDC, if using S3)

### Set Permissions in Repository

Settings → Actions → General → Workflow permissions:
- ✅ Read and write permissions
- ✅ Allow GitHub Actions to create and approve pull requests

## AWS Integration (Optional)

For storing scan results in S3:

### 1. Create IAM Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::YOUR_ACCOUNT:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:aud": "sts.amazonaws.com"
        },
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:your-org/*:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

### 2. Create S3 Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::security-scan-results",
        "arn:aws:s3:::security-scan-results/*"
      ]
    }
  ]
}
```

### 3. Use in Workflow

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    role-to-assume: arn:aws:iam::YOUR_ACCOUNT:role/github-actions-security-role
    aws-region: us-east-1

- name: Upload Results
  run: |
    aws s3 cp scan-results.json s3://security-scan-results/${{ github.repository }}/
```

## Troubleshooting Setup

### Workflow Not Triggering

1. Check `.github/workflows/` directory exists
2. Verify YAML syntax is valid
3. Ensure `on:` triggers are configured
4. Check branch protection rules aren't blocking

### Permissions Errors

1. Go to Settings → Actions → General
2. Select "Read and write permissions"
3. Enable "Allow GitHub Actions to create PRs"
4. Save settings

### Secrets Not Found

1. Create secrets in organization settings (not repository)
2. Or override in repository settings
3. Verify secret names match exactly
4. Reload workflow after adding secrets

### Webhook Errors

**Teams:**
```bash
# Test webhook
curl -X POST -H 'Content-Type: application/json' \
  -d '{"text":"Test from GitHub"}' \
  YOUR_TEAMS_WEBHOOK_URL
```

**Slack:**
```bash
# Test webhook
curl -X POST -H 'Content-Type: application/json' \
  -d '{"text":"Test from GitHub"}' \
  YOUR_SLACK_WEBHOOK_URL
```

## Next Steps

1. ✅ Create repository
2. ✅ Configure webhooks
3. ✅ Add workflows to your repos
4. ✅ Test with a PR
5. ✅ Review results and notifications
6. ✅ Adjust settings based on findings

## Support

See the documentation in `docs/` directory for:
- [Notifications Setup](docs/NOTIFICATIONS.md)
- [SCA Scanning](docs/SCA-SCAN.md)
- [SAST Scanning](docs/SAST-SCAN.md)
- [IAC Scanning](docs/IAC-SCAN.md)
- [Secrets Scanning](docs/SECRETS-SCAN.md)
- [Best Practices](docs/BEST-PRACTICES.md)

---

**Next Steps**: Read [NOTIFICATIONS.md](docs/NOTIFICATIONS.md) to configure alerting.
