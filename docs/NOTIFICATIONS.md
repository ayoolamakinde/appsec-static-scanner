# Notifications Workflow

## Overview

The `notify.yml` workflow provides a unified, reusable notification system that supports both **Microsoft Teams** and **Slack**. It enables consistent security alerting across your projects regardless of your preferred communication platform.

## Features

✅ **Multi-Channel Support**: Teams and Slack simultaneously  
✅ **Smart Detection**: Automatically enables configured channels  
✅ **Rich Formatting**: Adaptive cards for Teams, Block Kit for Slack  
✅ **Flexible Severity**: CRITICAL, HIGH, MEDIUM, LOW, INFO  
✅ **Contextual Information**: Repository, branch, actor, findings count  
✅ **Action Links**: Direct links to workflows and PRs  
✅ **Reusable**: Can be called from any scanning workflow  

## Setup

### 1. Create Webhooks

**Microsoft Teams:**
1. Go to your Teams channel
2. Click `...` → Connectors
3. Search "Incoming Webhook"
4. Configure and copy URL
5. Store as `TEAMS_WEBHOOK` secret

**Slack:**
1. Go to Slack Workspace Settings → Manage Apps
2. Search "Incoming Webhooks"
3. Create New Webhook
4. Copy URL
5. Store as `SLACK_WEBHOOK` secret

### 2. Add to Repository Secrets

```bash
# GitHub CLI
gh secret set TEAMS_WEBHOOK --body "https://..."
gh secret set SLACK_WEBHOOK --body "https://..."
```

## Usage

### Basic Usage

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    outputs:
      findings: ${{ steps.scan.outputs.count }}
    steps:
      - name: Run Scan
        id: scan
        run: echo "count=5" >> $GITHUB_OUTPUT

  notify:
    needs: scan
    if: ${{ needs.scan.outputs.findings > 0 }}
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
    with:
      notification_type: 'vulnerability'
      tool_name: 'Trivy'
      severity: 'HIGH'
      finding_count: ${{ needs.scan.outputs.findings }}
      message: 'Dependencies need updating'
    secrets:
      TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Input Parameters

| Parameter | Required | Type | Description | Example |
|-----------|----------|------|-------------|---------|
| `notification_type` | ✅ | string | Alert type: vulnerability, violation, secret, finding, custom | `vulnerability` |
| `tool_name` | ✅ | string | Scanning tool name | `Trivy (SCA)` |
| `severity` | ✅ | string | CRITICAL, HIGH, MEDIUM, LOW, INFO | `CRITICAL` |
| `finding_count` | ✅ | number | Number of findings | `42` |
| `message` | ❌ | string | Custom message to include | `Please review and fix` |
| `affected_files` | ❌ | string | Comma-separated file list | `src/app.js, src/config.js` |

### Notification Types

**vulnerability** - Used for SCA/image scanning
```yaml
notification_type: 'vulnerability'
tool_name: 'Trivy (SCA)'
message: 'Dependency vulnerabilities detected'
```

**violation** - Used for IAC scanning
```yaml
notification_type: 'violation'
tool_name: 'Checkov'
message: 'Infrastructure compliance violations'
```

**secret** - Used for secrets scanning (always high severity)
```yaml
notification_type: 'secret'
tool_name: 'TruffleHog'
severity: 'CRITICAL'
message: 'Exposed credentials detected'
```

**finding** - Used for general code analysis
```yaml
notification_type: 'finding'
tool_name: 'Semgrep (SAST)'
message: 'Code security issues found'
```

**custom** - For custom alerts
```yaml
notification_type: 'custom'
tool_name: 'Custom Scanner'
message: 'Custom security alert'
```

## Examples

### Example 1: Notify on SCA Vulnerabilities

```yaml
name: SCA Scan with Notifications

on:
  pull_request:
    paths:
      - '**/package.json'
      - '**/requirements.txt'
      - '**/go.mod'

jobs:
  sca:
    runs-on: ubuntu-latest
    outputs:
      vuln_count: ${{ steps.scan.outputs.count }}
      found: ${{ steps.scan.outputs.found }}
    steps:
      - uses: actions/checkout@v4
      - name: Scan
        id: scan
        run: |
          # Run Trivy
          echo "count=3" >> $GITHUB_OUTPUT
          echo "found=true" >> $GITHUB_OUTPUT

  notify:
    if: needs.sca.outputs.found == 'true'
    needs: sca
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
    with:
      notification_type: 'vulnerability'
      tool_name: 'Trivy (SCA)'
      severity: 'HIGH'
      finding_count: ${{ needs.sca.outputs.vuln_count }}
      message: '⚠️ Vulnerable dependencies detected. Please update your dependencies.'
      affected_files: 'package.json, requirements.txt'
    secrets:
      TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Example 2: Critical Secrets Alert

```yaml
jobs:
  secrets:
    runs-on: ubuntu-latest
    outputs:
      secret_count: ${{ steps.scan.outputs.count }}
      found: ${{ steps.scan.outputs.found }}
    steps:
      - uses: actions/checkout@v4
      - name: Scan
        id: scan
        run: |
          # Run TruffleHog
          echo "count=2" >> $GITHUB_OUTPUT
          echo "found=true" >> $GITHUB_OUTPUT

  critical-alert:
    if: needs.secrets.outputs.found == 'true'
    needs: secrets
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
    with:
      notification_type: 'secret'
      tool_name: 'TruffleHog'
      severity: 'CRITICAL'
      finding_count: ${{ needs.secrets.outputs.secret_count }}
      message: '🚨 **IMMEDIATE ACTION REQUIRED**: Exposed credentials detected! Rotate all exposed keys immediately.'
      affected_files: 'src/config.js, .env.example'
    secrets:
      TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

### Example 3: Conditional Notifications

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    outputs:
      critical: ${{ steps.check.outputs.critical }}
      high: ${{ steps.check.outputs.high }}
    steps:
      - uses: actions/checkout@v4
      - name: Check
        id: check
        run: |
          echo "critical=1" >> $GITHUB_OUTPUT
          echo "high=3" >> $GITHUB_OUTPUT

  notify-critical:
    if: needs.scan.outputs.critical > 0
    needs: scan
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
    with:
      notification_type: 'vulnerability'
      tool_name: 'Trivy'
      severity: 'CRITICAL'
      finding_count: ${{ needs.scan.outputs.critical }}
    secrets:
      TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}

  notify-high:
    if: needs.scan.outputs.high > 0
    needs: scan
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/notify.yml@main
    with:
      notification_type: 'vulnerability'
      tool_name: 'Trivy'
      severity: 'HIGH'
      finding_count: ${{ needs.scan.outputs.high }}
    secrets:
      TEAMS_WEBHOOK: ${{ secrets.TEAMS_WEBHOOK }}
      SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
```

## Teams Notification Format

The Teams notification displays as an Adaptive Card:

```
╔════════════════════════════════════════════╗
║ 🔴 SCA Vulnerabilities Detected             ║
║                                            ║
║ Repository:  org/repo                      ║
║ Tool:        Trivy (SCA)                   ║
║ Severity:    HIGH                          ║
║ Findings:    5                             ║
║ Branch:      main                          ║
║ Triggered by: user                         ║
║                                            ║
║ Dependency vulnerabilities detected during ║
║ SCA scan. Please review and update         ║
║ dependencies.                              ║
║                                            ║
║ [🔍 View Workflow Run] [📋 View PR]        ║
╚════════════════════════════════════════════╝
```

## Slack Notification Format

The Slack notification displays as Block Kit:

```
┌─────────────────────────────────────────┐
│ 🔴 SCA Vulnerabilities Detected         │
├─────────────────────────────────────────┤
│ Repository:  org/repo                   │
│ Tool:        Trivy (SCA)                │
│ Severity:    HIGH                       │
│ Findings:    5                          │
│ Branch:      main                       │
│ Triggered by: user                      │
├─────────────────────────────────────────┤
│ Dependency vulnerabilities detected     │
├─────────────────────────────────────────┤
│ [🔍 View Workflow] [📋 View PR]         │
└─────────────────────────────────────────┘
```

## Severity Levels

| Severity | Emoji | Teams Color | Slack Color | Use Case |
|----------|-------|------------|------------|----------|
| **CRITICAL** | 🚨 | Red (#DC143C) | danger | Immediate action required |
| **HIGH** | 🔴 | Orange-Red (#FF4500) | danger | Review and fix soon |
| **MEDIUM** | 🟠 | Orange (#FFA500) | warning | Plan remediation |
| **LOW** | 🟡 | Gold (#FFD700) | warning | Track for future |
| **INFO** | ℹ️ | Blue (#4169E1) | good | Informational |

## Best Practices

### 1. Use Appropriate Severity

```yaml
# CRITICAL - only for actual critical issues
severity: 'CRITICAL'  # Secrets, RCE, auth bypass

# HIGH - significant risk
severity: 'HIGH'      # SQL injection, XSS, privilege escalation

# MEDIUM - should be fixed
severity: 'MEDIUM'    # Weak crypto, hardcoded values

# LOW - nice to have
severity: 'LOW'       # Code quality, best practices
```

### 2. Include Actionable Messages

```yaml
# Good ✅
message: 'Update Node.js dependencies in package.json. Run `npm audit fix`'

# Bad ❌
message: 'Dependencies have vulnerabilities'
```

### 3. List Affected Files

```yaml
affected_files: 'src/auth.js, src/db.js, src/api.js'
```

### 4. Condition Notifications

```yaml
# Only notify on critical issues
if: ${{ needs.scan.outputs.critical_count > 0 }}

# Notify only on PRs
if: github.event_name == 'pull_request'

# Notify only on main branch
if: github.ref == 'refs/heads/main'
```

## Troubleshooting

### Teams Notification Not Sent

1. Check webhook URL is correct
2. Verify webhook is enabled in Teams
3. Test manually:
   ```bash
   curl -H 'Content-Type: application/json' \
        -d '{"text":"Test"}' \
        https://your-webhook-url
   ```

### Slack Notification Not Sent

1. Verify webhook URL
2. Check Slack workspace permissions
3. Test Block Kit formatting

### Missing Fields in Notification

Ensure all required inputs are provided:
- `notification_type`
- `tool_name`
- `severity`
- `finding_count`

## See Also

- [Integrated Scan Workflow](../integrated-scan.yml)
- [SCA Scan Workflow](sca-scan.yml)
- [SAST Scan Workflow](sast-scan.yml)
- [IAC Scan Workflow](iac-scan.yml)
- [Secrets Scan Workflow](secrets-scan.yml)

---

**Last Updated**: January 18, 2026
