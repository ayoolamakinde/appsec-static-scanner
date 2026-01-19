# IAC Scan - Infrastructure as Code Scanning

## Overview

The IAC (Infrastructure as Code) workflow scans infrastructure definitions for security misconfigurations, compliance violations, and best practice deviations using **Checkov**. It supports multiple infrastructure frameworks and compliance standards.

**Supported Frameworks:**
- Terraform
- CloudFormation (AWS)
- Kubernetes
- Docker
- Bicep (Azure)
- ARM Templates
- Helm Charts
- Ansible
- GitLab CI/CD
- GitHub Actions

**Compliance Standards:**
- CIS Benchmarks (AWS, Azure, GCP, Kubernetes)
- PCI DSS
- SOC 2
- HIPAA
- NIST
- FedRAMP

## Table of Contents

1. [Features](#features)
2. [Setup](#setup)
3. [Usage](#usage)
4. [Configuration](#configuration)
5. [Examples](#examples)
6. [Outputs](#outputs)
7. [Custom Checks](#custom-checks)
8. [Troubleshooting](#troubleshooting)

---

## Features

✅ **Multi-Framework Support** - Terraform, CloudFormation, Kubernetes, Docker, Bicep, ARM
✅ **Compliance Standards** - CIS, PCI DSS, SOC 2, HIPAA, NIST
✅ **Selective Checks** - Run specific checks or exclude certain ones
✅ **Custom Configuration** - Provide custom Checkov config
✅ **Output Formats** - JSON, SARIF, CSV, table
✅ **GitHub Issues** - Automatic issue creation for findings
✅ **PR Comments** - Inline PR comments with violations
✅ **Policy as Code** - Define failure policies
✅ **Severity Levels** - Filter by CRITICAL, HIGH, MEDIUM, LOW

---

## Setup

### 1. Basic Setup

Add to your repository (`.github/workflows/iac-scan.yml`):

```yaml
name: IAC Scan

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### 2. Checkov Configuration (Optional)

Create `.checkov.yaml` in your repository:

```yaml
framework:
  - terraform
  - kubernetes

check:
  - CKV_AWS_1
  - CKV_K8S_1

skip-check:
  - CKV_AWS_49

severity: MEDIUM

compact: false
```

---

## Usage

### Basic Usage

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
```

### With Terraform Only

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'terraform'
      fail_on_severity: 'HIGH'
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

### With CIS Compliance

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'kubernetes'
      check_ids: 'CKV_K8S_1,CKV_K8S_3,CKV_K8S_9'
      fail_on_finding: true
    secrets:
      SECURITY_WEBHOOK: ${{ secrets.SECURITY_WEBHOOK }}
```

---

## Configuration

### Input Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `frameworks` | string | `terraform,cloudformation,kubernetes,dockerfile,bicep,arm` | Frameworks to scan |
| `check_ids` | string | `` | Specific checks to run |
| `skip_checks` | string | `CKV_DOCKER_2,CKV_AWS_49` | Checks to exclude |
| `skip_paths` | string | `tests,examples,.terraform,node_modules` | Paths to exclude |
| `severity` | string | `CRITICAL,HIGH,MEDIUM,LOW` | Severity levels to report |
| `fail_on_severity` | string | `CRITICAL,HIGH` | Fail on this severity or higher |
| `output_format` | string | `json` | Format: json, sarif, csv |
| `max_retries` | number | `3` | Retry attempts |
| `fail_on_finding` | boolean | `true` | Fail if findings found |
| `continue_on_error` | boolean | `false` | Continue even if scan fails |
| `upload_artifacts` | boolean | `true` | Upload results |
| `create_issue` | boolean | `false` | Create GitHub issue |
| `comment_pr` | boolean | `true` | Comment on PR |
| `compact_output` | boolean | `false` | Compact output format |
| `quiet_mode` | boolean | `false` | Suppress extra output |
| `download_external_modules` | boolean | `true` | Download Terraform modules |
| `config_file` | string | `.checkov.yaml` | Custom config file |

### Check ID Reference

**AWS (CloudFormation/Terraform):**
- `CKV_AWS_1` - Ensure S3 bucket has versioning
- `CKV_AWS_3` - Ensure EC2 has monitoring enabled
- `CKV_AWS_7` - Ensure RDS has encryption at rest
- `CKV_AWS_23` - Ensure SSH restricted in security group
- `CKV_AWS_24` - Ensure all traffic restricted in security group

**Kubernetes:**
- `CKV_K8S_1` - Containers must not run as root
- `CKV_K8S_3` - Ensure default namespace not used
- `CKV_K8S_8` - Ensure pod security policy enabled
- `CKV_K8S_9` - RBAC enabled
- `CKV_K8S_43` - Image pull policy is Always

**Docker:**
- `CKV_DOCKER_2` - Ensure HEALTHCHECK instruction
- `CKV_DOCKER_3` - Ensure USER instruction
- `CKV_DOCKER_8` - Ensure WORKDIR not /

---

## Examples

### Example 1: Terraform Compliance

```yaml
name: IAC - Terraform CIS

on: [pull_request]

jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'terraform'
      check_ids: 'CKV_AWS_1,CKV_AWS_3,CKV_AWS_7,CKV_AWS_23'
      skip_paths: 'tests,examples'
      fail_on_severity: 'HIGH'
```

**Scans:**
- Terraform files only
- CIS AWS checks (S3, EC2, RDS, Security Groups)
- Fails on HIGH or CRITICAL

### Example 2: Kubernetes Security

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'kubernetes'
      check_ids: 'CKV_K8S_1,CKV_K8S_3,CKV_K8S_8,CKV_K8S_9'
      fail_on_severity: 'CRITICAL'
      create_issue: true
```

**Scans:**
- Kubernetes manifests only
- CIS Kubernetes checks
- Creates issues for CRITICAL findings

### Example 3: Docker Image Security

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'dockerfile'
      skip_checks: 'CKV_DOCKER_2'  # Skip healthcheck for now
      fail_on_finding: true
```

**Scans:**
- Dockerfile only
- All checks except healthcheck
- Fails on any finding

### Example 4: Multi-Framework Enterprise

```yaml
jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      frameworks: 'terraform,cloudformation,kubernetes'
      severity: 'CRITICAL,HIGH'
      fail_on_severity: 'CRITICAL'
      download_external_modules: true
      create_issue: true
      comment_pr: true
```

**Scans:**
- Terraform, CloudFormation, and Kubernetes
- CRITICAL and HIGH findings only
- Creates issues and PR comments

### Example 5: Lenient Development

```yaml
on:
  pull_request:
    branches:
      - develop

jobs:
  iac:
    uses: your-org/appsec-static-scanner/.github/workflows/iac-scan.yml@main
    with:
      fail_on_finding: false
      comment_pr: true
      compact_output: true
      severity: 'CRITICAL,HIGH,MEDIUM'
```

**Behavior:**
- Never fails workflow
- Comments on PR with findings
- Compact output (less verbose)

---

## Outputs

### GitHub Artifacts

**File:** `iac-scan-results.json`

Example:
```json
{
  "summary": {
    "passed_checks": 45,
    "failed_checks": 8,
    "skipped_checks": 2,
    "resource_count": 55,
    "check_type_to_results": {
      "terraform": {
        "passed": 35,
        "failed": 5
      },
      "kubernetes": {
        "passed": 10,
        "failed": 3
      }
    }
  },
  "failed_checks": [
    {
      "check_id": "CKV_AWS_1",
      "check_name": "Ensure S3 bucket has versioning enabled",
      "check_result": {"result": "failed"},
      "code_block": [["resource \"aws_s3_bucket\"", "bucket = \"my-bucket\""]],
      "file_path": "main.tf",
      "file_line_range": [1, 5],
      "severity": "MEDIUM"
    }
  ]
}
```

### PR Comment

```
## 🏗️ IAC Scan Results

### Summary
| Status | Count |
|--------|-------|
| ✅ Passed | 45 |
| ❌ Failed | 8 |
| ⏭️ Skipped | 2 |

### Failed Checks
- CKV_AWS_1: S3 bucket versioning (main.tf:1)
- CKV_K8S_1: Container running as root (deployment.yaml:15)

### Resources Scanned
- terraform: 35 resources
- kubernetes: 20 pods

_🏗️ Scan: Checkov (Infrastructure as Code)_
```

---

## Common Violations

### S3 Bucket Not Versioned

```hcl
# ❌ Non-Compliant
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
  # Missing versioning
}

# ✅ Compliant
resource "aws_s3_bucket" "example" {
  bucket = "my-bucket"
}

resource "aws_s3_bucket_versioning" "example" {
  bucket = aws_s3_bucket.example.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

### Container Running as Root

```yaml
# ❌ Non-Compliant
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  containers:
  - name: app
    image: app:latest
    # Implicitly runs as root

# ✅ Compliant
apiVersion: v1
kind: Pod
metadata:
  name: example
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
  - name: app
    image: app:latest
```

### Security Group Allows SSH from Anywhere

```hcl
# ❌ Non-Compliant
resource "aws_security_group" "example" {
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # SSH from anywhere
  }
}

# ✅ Compliant
resource "aws_security_group" "example" {
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/8"]  # SSH from internal only
  }
}
```

---

## Custom Checks

### Create Custom Policy

**File:** `.checkov.yaml`

```yaml
check:
  # Include specific checks
  - CKV_AWS_1
  - CKV_AWS_3

skip-check:
  # Exclude checks
  - CKV_AWS_49  # Known false positive

custom-checks-dir:
  - ./checkov_checks

severity: MEDIUM

framework:
  - terraform
  - kubernetes
```

### Custom Python Check

**File:** `checkov_checks/custom_s3_check.py`

```python
from checkov.terraform.checks.resource.base_resource_check import BaseResourceCheck
from checkov.common.models.enums import CheckResult, CheckCategories

class S3BucketEncryption(BaseResourceCheck):
    name = "Ensure S3 bucket encryption"
    id = "CKV_AWS_999"
    supported_resources = ['aws_s3_bucket']
    categories = [CheckCategories.GENERAL_SECURITY]

    def scan_resource_conf(self, conf):
        if 'server_side_encryption_configuration' in conf:
            return CheckResult.PASSED
        return CheckResult.FAILED

check = S3BucketEncryption()
```

---

## Compliance Standards

### CIS AWS

```yaml
with:
  frameworks: 'terraform'
  check_ids: 'CKV_AWS_1,CKV_AWS_3,CKV_AWS_7,CKV_AWS_23'
```

### PCI DSS

```yaml
with:
  frameworks: 'terraform,cloudformation'
  check_ids: 'CKV_AWS_2,CKV_AWS_6,CKV_AWS_7,CKV_AWS_20'
```

### SOC 2

```yaml
with:
  frameworks: 'kubernetes'
  check_ids: 'CKV_K8S_1,CKV_K8S_3,CKV_K8S_8,CKV_K8S_9'
```

---

## Troubleshooting

### Issue: Checkov fails to download modules

**Cause:** Network issue or missing credentials

**Solution:**
```yaml
with:
  download_external_modules: false
```

### Issue: Too many false positives

**Cause:** Overly strict checks

**Solution:**
```yaml
with:
  skip_checks: 'CKV_AWS_49,CKV_AWS_52'  # Exclude known false positives
```

### Issue: Custom checks not loading

**Cause:** Directory path or Python syntax error

**Solution:**
```bash
# Validate locally
checkov -d . --check-runner framework --custom-checks-dir ./checkov_checks
```

### Issue: Scanning too slow

**Cause:** Large repository or too many checks

**Solution:**
```yaml
with:
  frameworks: 'terraform'  # Scan only needed frameworks
  quiet_mode: true  # Reduce output verbosity
```

---

## Best Practices

1. **Start with Critical Checks**
   ```yaml
   fail_on_severity: 'CRITICAL'
   ```

2. **Regular Compliance Updates**
   - Update Checkov frequently
   - Review new checks monthly
   - Adjust skip-list as needed

3. **Exclude Test/Example Code**
   ```yaml
   skip_paths: 'tests,examples,.terraform'
   ```

4. **Use Framework-Specific Scans**
   ```yaml
   # Separate workflows per framework
   frameworks: 'terraform'
   frameworks: 'kubernetes'
   ```

5. **Enforce Policy as Code**
   ```yaml
   fail_on_severity: 'HIGH'  # Fail on HIGH or CRITICAL
   create_issue: true        # Track violations
   ```

---

## See Also

- [Checkov Documentation](https://www.checkov.io)
- [Checkov Checks](https://www.checkov.io/2.Checks/)
- [SCA Scan](SCA-SCAN.md)
- [SAST Scan](SAST-SCAN.md)
- [Secrets Scan](SECRETS-SCAN.md)

---

**Last Updated:** January 18, 2026
