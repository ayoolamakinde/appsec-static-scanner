# SAST Scan - Static Application Security Testing

## Overview

The SAST (Static Application Security Testing) workflow analyzes source code for security vulnerabilities using **Semgrep**. It detects code-level security issues, common programming mistakes, and compliance violations.

**Supported Languages:**
- Python
- JavaScript/TypeScript
- Java
- Go
- C/C++
- C#/.NET
- Ruby
- PHP
- Rust
- Kotlin
- And more...

**Rule Sets:**
- OWASP Top 10
- CWE Top 25
- Security best practices
- Custom rules

## Table of Contents

1. [Features](#features)
2. [Setup](#setup)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Examples](#examples)
6. [Outputs](#outputs)
7. [Custom Rules](#custom-rules)
8. [Troubleshooting](#troubleshooting)

---

## Features

✅ **Multi-Language Analysis** - Scan code across all major languages
✅ **SARIF Integration** - Upload results to GitHub Security tab
✅ **Custom Rules** - Define your own security rules
✅ **Multiple Rulesets** - OWASP, CWE, security best practices
✅ **PR Comments** - Automatic PR comments with findings
✅ **Issue Creation** - Create GitHub issues for findings
✅ **Severity Levels** - ERROR, WARNING, NOTE
✅ **Language Filtering** - Scan specific languages
✅ **Configurable Rules** - Enable/disable specific checks

---

## Setup

### 1. Basic Setup

Add to your repository (`.github/workflows/sast-scan.yml`):

```yaml
name: SAST Scan

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. Configure Custom Rules (Optional)

Create `.semgrep.yml` in your repository:

```yaml
rules:
  - id: custom-sql-injection
    pattern: |
      query = $X + user_input
    message: Potential SQL injection
    severity: ERROR
```

---

## Usage

### Basic Usage

```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
```

### With Custom Configuration

```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      severity: 'ERROR,WARNING'
      fail_on_severity: 'ERROR'
      languages: 'python,javascript,go'
      rules_path: 'p/owasp-top-ten'
      upload_sarif: true
      create_issue: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

---

## Configuration

### Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `severity` | string | `ERROR,WARNING` | Severity levels to report |
| `fail_on_severity` | string | `ERROR` | Fail on this severity or higher |
| `languages` | string | `python,javascript,go,java,typescript,csharp,ruby,php,rust` | Languages to scan |
| `rules_path` | string | `p/owasp-top-ten` | Rule set to use |
| `skip_patterns` | string | `node_modules,vendor,.git,dist,build,tests,__pycache__` | Patterns to exclude |
| `output_format` | string | `json` | Output format: json, sarif, csv |
| `max_retries` | number | `3` | Retry attempts |
| `fail_on_finding` | boolean | `true` | Fail if findings found |
| `continue_on_error` | boolean | `false` | Continue even if scan fails |
| `upload_artifacts` | boolean | `true` | Upload results |
| `upload_sarif` | boolean | `true` | Upload to GitHub Security |
| `create_issue` | boolean | `false` | Create GitHub issue |
| `comment_pr` | boolean | `true` | Comment on PR |
| `config_file` | string | `.semgrep.yml` | Custom config file |

### Available Rule Sets

- `p/owasp-top-ten` - OWASP Top 10
- `p/cwe-top-25` - CWE Top 25
- `p/security-audit` - General security
- `p/django` - Django framework
- `p/flask` - Flask framework
- `p/nodejs` - Node.js
- `p/java` - Java security
- `p/go` - Go security
- `p/generic-security` - Language-agnostic

---

## Examples

### Example 1: Python Security Scan

```yaml
name: SAST - Python

on: [pull_request]

jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'python'
      rules_path: 'p/security-audit'
      fail_on_severity: 'ERROR'
```

**Scans:**
- Python files only
- Security audit rules
- Fails on ERROR level

### Example 2: JavaScript/TypeScript Analysis

```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      languages: 'javascript,typescript'
      rules_path: 'p/nodejs'
      skip_patterns: 'node_modules,.next,build,dist'
```

**Scans:**
- JS/TS files
- Node.js rules
- Excludes common JS build folders

### Example 3: OWASP Compliance

```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      rules_path: 'p/owasp-top-ten'
      severity: 'ERROR,WARNING,NOTE'
      fail_on_severity: 'WARNING'
      create_issue: true
```

**Features:**
- OWASP Top 10 rules
- Reports all severity levels
- Fails on WARNING or higher
- Creates issues for tracking

### Example 4: Custom Rules

Create `.semgrep.yml`:
```yaml
rules:
  - id: no-hardcoded-api-keys
    patterns:
      - pattern: |
          API_KEY = "..."
      - pattern: |
          api_key = "..."
    message: Hardcoded API key detected
    severity: ERROR
```

Then use:
```yaml
jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      config_file: '.semgrep.yml'
      fail_on_finding: true
```

### Example 5: Lenient Development

```yaml
on:
  pull_request:
    branches:
      - develop

jobs:
  sast:
    uses: ayoolamakinde/appsec-static-scanner/.github/workflows/sast-scan.yml@main
    with:
      fail_on_finding: false
      comment_pr: true
      severity: 'ERROR,WARNING'
```

**Behavior:**
- Never fails workflow
- Comments on PR
- Reports findings for awareness

---

## Outputs

### GitHub Artifacts

**File:** `sast-results.json`

Example:
```json
{
  "results": [
    {
      "check_id": "python.django.security.injection.sql.sql-injection",
      "path": "app/views.py",
      "start": {"line": 42, "col": 10},
      "end": {"line": 42, "col": 25},
      "message": "Potential SQL injection",
      "severity": "ERROR",
      "extra": {
        "fingerprint": "abc123def456",
        "fixed_by_semgrep": false
      }
    }
  ]
}
```

### SARIF Upload

Results automatically uploaded to **GitHub > Security > Code scanning alerts**

### PR Comment

```
## 🔎 SAST Scan Results

### Summary
| Severity | Count |
|----------|-------|
| 🔴 ERROR | 2 |
| 🟠 WARNING | 5 |
| 🔵 NOTE | 3 |

### Details
View detailed findings in:
- Artifacts: `sast-scan-results`
- GitHub Security tab (if SARIF uploaded)

_🔍 Scan: Semgrep (SAST - Code Vulnerabilities)_
```

---

## Custom Rules

### Creating Custom Rules

**File:** `.semgrep.yml`

```yaml
rules:
  - id: insecure-random-number
    pattern: |
      import random
      ...
      random.random()
    message: Using insecure random for security purposes
    languages: [python]
    severity: WARNING

  - id: hardcoded-credentials
    patterns:
      - pattern: |
          password = "..."
      - pattern: |
          secret = "..."
    message: Hardcoded credentials detected
    languages: [python, javascript]
    severity: ERROR
```

### Rule Structure

```yaml
- id: unique-rule-id
  pattern: |
    # Pattern matching code
  message: Human-readable message
  languages: [python]
  severity: ERROR  # ERROR, WARNING, NOTE
  fix: |
    # Optional auto-fix
  metadata:
    cwe: CWE-123
    owasp: A04:2021
```

### Pattern Examples

**Simple Pattern:**
```yaml
pattern: SQL_QUERY($X + user_input)
```

**Multiple Patterns (OR):**
```yaml
patterns:
  - pattern: old_function(...)
  - pattern: deprecated_api(...)
```

**AND Patterns:**
```yaml
pattern-either:
  - patterns:
      - pattern: func(...)
      - pattern-inside: |
          if dangerous_flag:
            ...
```

---

## Rule Repositories

### Use Official Rulesets

```yaml
# OWASP Top 10
rules_path: 'p/owasp-top-ten'

# CWE Top 25
rules_path: 'p/cwe-top-25'

# Language-specific
rules_path: 'p/python'
rules_path: 'p/nodejs'
rules_path: 'p/java'

# Framework-specific
rules_path: 'p/django'
rules_path: 'p/flask'
rules_path: 'p/express'
```

---

## Common Findings

### Finding: SQL Injection

```python
# ❌ Vulnerable
query = f"SELECT * FROM users WHERE id = {user_id}"

# ✅ Safe
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

### Finding: XSS Vulnerability

```javascript
// ❌ Vulnerable
document.innerHTML = userInput;

// ✅ Safe
document.textContent = userInput;
```

### Finding: Hardcoded Secret

```python
# ❌ Vulnerable
API_KEY = "sk-1234567890abcdef"

# ✅ Safe
API_KEY = os.getenv("API_KEY")
```

---

## Troubleshooting

### Issue: No findings but workflow fails

**Cause:** Scan timed out or language not detected

**Solution:**
```yaml
with:
  languages: 'python,javascript'  # Explicitly specify
  max_retries: 5
```

### Issue: Too many false positives

**Cause:** Rules too strict

**Solution:**
```yaml
with:
  fail_on_severity: 'ERROR'  # Only fail on errors
  create_issue: false  # Reduce noise
```

### Issue: Custom rules not loading

**Cause:** `.semgrep.yml` syntax error

**Solution:**
```bash
# Validate locally
semgrep --config .semgrep.yml --validate
```

### Issue: SARIF upload fails

**Cause:** Invalid SARIF format

**Solution:**
```yaml
with:
  upload_sarif: false  # Disable temporarily
  output_format: 'json'
```

---

## Best Practices

1. **Start with OWASP rules**
   ```yaml
   rules_path: 'p/owasp-top-ten'
   ```

2. **Create custom rules for your codebase**
   ```yaml
   config_file: '.semgrep.yml'
   ```

3. **Fail on ERROR, warn on WARNING**
   ```yaml
   fail_on_severity: 'ERROR'
   ```

4. **Review and suppress false positives**
   ```python
   # nosemgrep: python.insecure-deserialization
   pickle.loads(data)
   ```

5. **Monitor trends**
   - Schedule weekly reports
   - Track reduction in findings
   - Celebrate fixes

---

## See Also

- [Semgrep Documentation](https://semgrep.dev/docs)
- [Semgrep Registry](https://semgrep.dev/r)
- [SCA Scan](SCA-SCAN.md)
- [IAC Scan](IAC-SCAN.md)
- [Secrets Scan](SECRETS-SCAN.md)

---

**Last Updated:** January 18, 2026
