# Best Practices

## Overview

This guide provides organization-wide best practices for implementing and maintaining a comprehensive security scanning program using the appsec-static-scanner workflows.

## Table of Contents

1. [Workflow Strategy](#workflow-strategy)
2. [Severity & Failure Policy](#severity--failure-policy)
3. [Notification Management](#notification-management)
4. [Integration Patterns](#integration-patterns)
5. [Team Responsibilities](#team-responsibilities)
6. [Metrics & Tracking](#metrics--tracking)
7. [Tool Configuration](#tool-configuration)
8. [Remediation Process](#remediation-process)

---

## Workflow Strategy

### Branch-Based Policies

**Develop Branch (Lenient):**
```yaml
on:
  pull_request:
    branches: [develop]

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_finding: false  # Don't block development
      comment_pr: true
      create_issue: false
```

**Main Branch (Strict):**
```yaml
on:
  pull_request:
    branches: [main]

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'HIGH'
      fail_on_finding: true
      create_issue: true
```

### Scheduled Security Audits

Run comprehensive scans on a schedule:

```yaml
name: Weekly Security Audit

on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday 2 AM UTC

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      fail_on_severity: 'MEDIUM'

  sast:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      fail_on_severity: 'WARNING'

  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      fail_on_severity: 'MEDIUM'

  secrets:
    uses: your-org/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: true
```

---

## Severity & Failure Policy

### Recommendation Matrix

| Tool | CRITICAL | HIGH | MEDIUM | LOW |
|------|----------|------|--------|-----|
| **SCA** | Fail Immediately | Fail | Warn | Info |
| **SAST** | Fail Immediately | Fail | Warn | Info |
| **IAC** | Fail Immediately | Fail | Warn | Info |
| **Secrets** | Fail Immediately (if verified) | Fail | Fail | Warn |

### Implementation

**SCA (Trivy):**
```yaml
fail_on_severity: 'CRITICAL,HIGH'
fail_on_finding: false  # Warn on MEDIUM
```

**SAST (Semgrep):**
```yaml
fail_on_severity: 'ERROR'  # Errors are critical
```

**IAC (Checkov):**
```yaml
fail_on_severity: 'CRITICAL,HIGH'
```

**Secrets (TruffleHog):**
```yaml
only_verified: false  # Fail on all potential secrets
fail_on_finding: true
```

### Overrides & Exceptions

**Document all exceptions:**
```yaml
# .security-exceptions.yaml
exceptions:
  - tool: sca
    finding_id: CVE-2023-1234
    component: lodash
    justification: "Patched in next release, low risk in our context"
    approved_by: security-team
    expires: 2024-12-31

  - tool: sast
    rule_id: python.django.sql-injection
    file: app/legacy_code.py
    justification: "Legacy code, scheduled for refactoring Q2"
    approved_by: john.doe
    expires: 2024-06-30
```

---

## Notification Management

### Enable All Channels

Configure both Teams and Slack for comprehensive visibility:

```yaml
env:
  TEAMS_WEBHOOK: ${{ secrets.SECURITY_TEAMS_WEBHOOK }}
  SLACK_WEBHOOK: ${{ secrets.SECURITY_SLACK_WEBHOOK }}

jobs:
  sca:
    uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main
    with:
      create_issue: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### Notification Routing

**Teams:** Executive summaries, escalations
**Slack:** Real-time development alerts, context

**Channel Strategy:**
```
#security-alerts (All findings)
  ├─ SCA vulnerabilities
  ├─ SAST issues
  ├─ IAC violations
  └─ Secrets detected

#security-critical (P1 incidents)
  └─ Verified secrets only
     Immediate team notification

#security-metrics (Weekly reports)
  └─ Trend analysis
     Remediation progress
```

### Alert Severity Levels

```
🔴 CRITICAL: Verified secrets, CRITICAL CVE, failed mandatory checks
🟠 HIGH: High-severity SAST/IAC, unverified secrets, high-risk CVE
🟡 MEDIUM: Medium-severity findings, standard violations
🔵 INFO: Low-severity findings, metrics, informational
```

---

## Integration Patterns

### Pattern 1: Independent Scans

Each tool runs independently, allowing fine-grained control:

```yaml
name: Security Scans

on: [pull_request]

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
      frameworks: 'terraform,kubernetes'

  secrets:
    uses: your-org/appsec-static-scanner/.github/workflows/secrets-scan.yml@main
    with:
      fail_on_finding: true
```

**Benefits:**
- Parallel execution
- Independent configuration
- Tool-specific tuning

### Pattern 2: Orchestrated with Conditions

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run SCA
        if: always()
        uses: your-org/appsec-static-scanner/.github/workflows/sca-scan.yml@main

      - name: Run SAST
        if: always()
        uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main

      - name: Notify Results
        if: always()
        uses: your-org/appsec-static-scanner/.github/workflows/notify.yml@main
        with:
          notification_type: 'summary'
```

### Pattern 3: Language-Specific

```yaml
name: Security Scans

on: [pull_request]

jobs:
  sast-python:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'python'
      rules_path: 'p/security-audit'

  sast-javascript:
    uses: your-org/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'javascript,typescript'
      rules_path: 'p/nodejs'
```

---

## Team Responsibilities

### Security Team

**Responsibilities:**
- Maintain tool configurations
- Review and triage findings
- Update rule sets and check IDs
- Establish severity thresholds
- Manage exceptions

**Cadence:**
- Weekly: Review high-priority findings
- Monthly: Update tools and rules
- Quarterly: Policy review

### Development Team

**Responsibilities:**
- Fix security findings in PRs
- Request exceptions (with justification)
- Acknowledge false positives
- Keep dependencies updated
- Rotate exposed secrets

**Response SLA:**
- Critical: 4 hours
- High: 24 hours
- Medium: 1 week
- Low: 2 weeks

### DevOps/Platform Team

**Responsibilities:**
- Maintain IaC compliance
- Configure cloud security
- Review and fix infrastructure findings
- Update Terraform/CloudFormation modules

---

## Metrics & Tracking

### Key Metrics

**Vulnerability Metrics:**
- Total CVEs by severity
- Remediation rate (% fixed)
- Average remediation time
- Recurring vulnerabilities

**Finding Metrics:**
- SAST issues by language
- IAC violations by framework
- Secrets detection rate

### Tracking Tools

**GitHub Issues:**
```yaml
labels:
  - security-critical
  - security-high
  - security-medium
  - security-low
  - exception-approved
```

**GitHub Projects:**
- Board for triage
- Columns: New, In Progress, Review, Done
- Automated transitions

### Reporting

**Weekly Report (Email):**
- New findings by tool
- Remediation status
- Top 5 vulnerabilities

**Monthly Report (Presentation):**
- Trends analysis
- Risk reduction
- Metrics dashboard
- Future roadmap

### Dashboard Example

```
OVERALL SECURITY POSTURE: 87% (↑ 5% from last month)

SCA Vulnerabilities:
  Critical: 0 ↓
  High: 3 ↓
  Medium: 12 (stable)
  Low: 45 (stable)

SAST Issues:
  ERROR: 2 ↓
  WARNING: 18 ↓

IAC Violations:
  CRITICAL: 0
  HIGH: 5
  MEDIUM: 12

Secrets Detected: 0 (↓ 2 remediated)

Remediation SLA:
  On-time: 92%
  Overdue: 2 (critical)
```

---

## Tool Configuration

### Recommended Defaults

**SCA (Trivy):**
```yaml
fail_on_severity: 'HIGH'
generate_sbom: true
output_format: 'sarif,json'
skip_dirs: 'tests,examples,.terraform,node_modules'
```

**SAST (Semgrep):**
```yaml
fail_on_severity: 'ERROR'
rules_path: 'p/owasp-top-ten'
upload_sarif: true
output_format: 'json,sarif'
```

**IAC (Checkov):**
```yaml
frameworks: 'terraform,cloudformation,kubernetes,dockerfile'
fail_on_severity: 'CRITICAL,HIGH'
download_external_modules: true
skip_paths: 'tests,examples,.terraform,node_modules'
```

**Secrets (TruffleHog):**
```yaml
scan_type: 'filesystem,git'
only_verified: false
entropy_threshold: 4.0
fail_on_finding: true
```

### Environment-Specific Configs

**Development:**
```yaml
SCA:
  fail_on_finding: false
  comment_pr: true

SAST:
  fail_on_severity: 'ERROR'

IAC:
  skip_checks: 'CKV_AWS_49'  # Known false positives

Secrets:
  entropy_threshold: 5.0  # Less sensitive
```

**Production:**
```yaml
SCA:
  fail_on_severity: 'MEDIUM'
  fail_on_finding: true

SAST:
  fail_on_severity: 'WARNING'

IAC:
  fail_on_severity: 'HIGH'

Secrets:
  only_verified: false
  fail_on_finding: true
```

---

## Remediation Process

### Vulnerability Triage

**1. Assess Impact:**
- Severity level
- Affected component
- Usage in code
- Exploitability

**2. Determine Fix:**
- Update dependency
- Code change
- Workaround
- Exception

**3. Implement:**
- Create PR
- Update tests
- Run scans
- Document

**4. Review & Merge:**
- Security review
- Code review
- Testing
- Production deployment

### Exception Process

**1. Request:**
```yaml
# In GitHub issue
- Finding: CVE-2023-1234
- Reason: Exploitable only in xyz scenario, doesn't apply
- Duration: Until lodash 5.x released (Q2 2024)
- Owner: john.doe
```

**2. Review:**
- Security team approval
- CTO sign-off (if critical)
- Document in `.security-exceptions.yaml`

**3. Track:**
- Create reminder 30 days before expiry
- Validate rationale still applies
- Renew or remediate

**4. Communicate:**
- Notify stakeholders
- Document in risk register
- Update risk assessment

### Remediation SLA

```
Critical:
  Discover: Immediate
  Assess: 2 hours
  Fix: 4 hours
  Deploy: 8 hours
  Verify: 24 hours

High:
  Discover: 6 hours
  Assess: 1 day
  Fix: 2 days
  Deploy: 3 days

Medium:
  Discover: 1 day
  Assess: 1 week
  Fix: 2 weeks
  Deploy: 3 weeks

Low:
  Discover: 1 week
  Assess: 2 weeks
  Fix: 1 month
  Deploy: 1 month
```

---

## Governance

### Policy Template

```markdown
## Security Scanning Policy

### Scope
All repositories in the organization must implement:
- SCA scanning (dependencies)
- SAST scanning (source code)
- IAC scanning (infrastructure)
- Secrets scanning (credentials)

### Requirements
- Scans run on all PRs to main/production branches
- CRITICAL/HIGH findings block merge
- Exceptions require security approval
- Metrics reported monthly

### Tools
- SCA: Trivy
- SAST: Semgrep
- IAC: Checkov
- Secrets: TruffleHog

### Escalation
- Critical: Security team lead
- High: Dev team lead + Security
- Medium: Dev team lead
- Low: Dev team discretion
```

### Security Baselines

**All Repositories Must:**
- ✅ Run SCA on dependencies
- ✅ Scan source code with SAST
- ✅ Detect exposed secrets
- ✅ Comment findings on PRs

**Production Repositories Must Also:**
- ✅ Scan infrastructure with IAC
- ✅ Fail on HIGH severity
- ✅ Create issues for tracking
- ✅ Monthly metrics reporting

---

## Continuous Improvement

### Monthly Reviews

```
Week 1: Triage new findings
Week 2: Remediation tracking
Week 3: Tools/rules update
Week 4: Metrics & reporting
```

### Quarterly Planning

- Update severity thresholds
- Evaluate new tools
- Review automation gaps
- Adjust team processes
- Training needs assessment

### Annual Assessment

- Tool ROI evaluation
- Process improvements
- Regulatory alignment
- Risk strategy update
- Team capacity planning

---

## See Also

- [SCA Scan](SCA-SCAN.md)
- [SAST Scan](SAST-SCAN.md)
- [IAC Scan](IAC-SCAN.md)
- [Secrets Scan](SECRETS-SCAN.md)
- [Notifications](NOTIFICATIONS.md)
- [Setup Guide](SETUP.md)

---

**Last Updated:** January 18, 2026
