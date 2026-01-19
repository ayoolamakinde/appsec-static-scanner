# Troubleshooting Guide

## Overview

Common issues encountered when using appsec-static-scanner workflows and how to resolve them.

## Table of Contents

1. [Workflow Execution Issues](#workflow-execution-issues)
2. [Tool-Specific Issues](#tool-specific-issues)
3. [Configuration Issues](#configuration-issues)
4. [Notification Issues](#notification-issues)
5. [Performance Issues](#performance-issues)
6. [Integration Issues](#integration-issues)
7. [Getting Help](#getting-help)

---

## Workflow Execution Issues

### Issue: Workflow Not Triggering

**Symptoms:**
- Workflow doesn't appear in Actions tab
- PR doesn't show checks

**Causes & Solutions:**

1. **Wrong trigger event**
   ```yaml
   # ❌ Wrong
   on: workflow_dispatch
   
   # ✅ Correct
   on: [pull_request, push]
   ```

2. **Branch filter mismatch**
   ```yaml
   # ❌ Triggers only on develop, PR to main won't trigger
   on:
     push:
       branches: [develop]
   
   # ✅ Correct
   on:
     pull_request:
       branches: [main, develop]
     push:
       branches: [main, develop]
   ```

3. **Incorrect workflow path**
   ```yaml
   # Workflow must be in .github/workflows/
   # ❌ Wrong: workflows/sca-scan.yml
   # ✅ Correct: .github/workflows/sca-scan.yml
   ```

**Verification Steps:**
```bash
# 1. Check workflow file syntax
# In repository: Actions tab > Select workflow > Logs

# 2. Verify branch protection rules
# Settings > Branches > Branch protection rules

# 3. Verify trigger conditions
git log --oneline -5  # See recent commits
```

---

### Issue: Workflow Timeout

**Symptoms:**
- Workflow stops after ~6 hours
- No completion message
- Job marked as cancelled

**Causes & Solutions:**

1. **Large repository size**
   ```yaml
   # Limit scan scope
   with:
     skip_paths: 'node_modules,vendor,.git,dist,build'
   ```

2. **History scanning too deep**
   ```yaml
   # For Git-based scans
   with:
     max_depth: 10000  # Limit git commits to scan
     scan_history: false  # Disable history scan
   ```

3. **Too many frameworks**
   ```yaml
   # ❌ Slow
   frameworks: 'terraform,cloudformation,kubernetes,dockerfile,arm,bicep'
   
   # ✅ Faster
   frameworks: 'terraform,kubernetes'
   ```

**Optimization Tips:**
```yaml
jobs:
  sca:
    timeout-minutes: 60  # Set explicit timeout
    with:
      skip_paths: 'tests,examples,vendor,node_modules,.git'
      max_retries: 2  # Reduce retries on timeout
```

---

### Issue: Workflow Fails with Unclear Error

**Symptoms:**
- Generic error message
- Stack trace in logs

**Solutions:**

1. **Check logs in detail**
   - GitHub Actions > Workflow Run > Jobs > Step Output
   - Look for actual error message

2. **Enable debug logging**
   ```yaml
   env:
     RUNNER_DEBUG: 1  # Enable GitHub Actions debug
   ```

3. **Run locally**
   ```bash
   # Install act: https://github.com/nektos/act
   act pull_request -j <job-name>
   ```

---

## Tool-Specific Issues

### SCA (Trivy) Issues

#### Issue: "No vulnerability database"

**Cause:** Database download failed

**Solution:**
```yaml
with:
  max_retries: 5  # Increase retries
  skip_dirs: '.git'  # Exclude large directories
```

Or manually clear cache:
```bash
rm -rf ~/.cache/trivy
```

#### Issue: "Unsupported image type"

**Cause:** Trivy can't scan image format

**Solution:**
```yaml
# Ensure using supported formats
# Supported: docker, oci, directory
with:
  scan_type: 'filesystem'  # Use filesystem scan
```

#### Issue: "Failed to analyze dependency"

**Cause:** Corrupted lock file or missing dependencies

**Solution:**
```bash
# Regenerate lock file
pip install -r requirements.txt --lock
npm install --package-lock-only
```

---

### SAST (Semgrep) Issues

#### Issue: "Semgrep rule not found"

**Cause:** Invalid rule ID or rule removed

**Solution:**
```bash
# List available rules
semgrep --list

# Use valid rule ID
rules_path: 'p/owasp-top-ten'  # Verified to exist
```

#### Issue: "Too many false positives"

**Cause:** Rule too strict for your code

**Solution:**
```yaml
# Option 1: Suppress specific rule
with:
  skip_patterns: 'test_*.py'  # Skip test files

# Option 2: Use stricter rules
rules_path: 'p/cwe-top-25'  # More focused

# Option 3: Custom rules
config_file: '.semgrep.yml'  # Your rules
```

Add suppression:
```python
# nosemgrep: python.insecure-function
import pickle
pickle.loads(data)  # Suppressed
```

#### Issue: "Custom rule not loading"

**Cause:** `.semgrep.yml` syntax error

**Solution:**
```bash
# Validate locally
semgrep --validate --config .semgrep.yml

# Check YAML syntax
yamllint .semgrep.yml
```

---

### IAC (Checkov) Issues

#### Issue: "Failed to download Terraform modules"

**Cause:** Network issue or module repository unavailable

**Solution:**
```yaml
with:
  download_external_modules: false  # Skip module download
```

Or pre-download:
```bash
cd terraform/
terraform init
```

#### Issue: "Check not found: CKV_AWS_999"

**Cause:** Check ID doesn't exist or version mismatch

**Solution:**
```bash
# List all checks
checkov -l

# Use valid check ID
checkov --check CKV_AWS_1
```

#### Issue: "Cannot parse Terraform file"

**Cause:** Invalid HCL syntax

**Solution:**
```bash
# Validate HCL
terraform validate

# Fix syntax errors
terraform fmt
```

---

### Secrets (TruffleHog) Issues

#### Issue: "False positive: Random string detected"

**Cause:** Entropy threshold too low

**Solution:**
```yaml
with:
  entropy_threshold: 5.0  # Increase from 4.0
```

Higher threshold = fewer false positives
Lower threshold = catches more secrets

**Threshold Guide:**
- 3.0: Very sensitive (many false positives)
- 4.0: Recommended (balanced)
- 5.0: Less sensitive (may miss some)
- 6.0: Only obvious secrets

#### Issue: "Custom detector not loading"

**Cause:** `.trufflehog.yaml` syntax error

**Solution:**
```bash
# Validate config
trufflehog filesystem . --config .trufflehog.yaml --json

# Check YAML
yamllint .trufflehog.yaml
```

#### Issue: "Verified secret test failed"

**Cause:** Detector misconfigured or detector API down

**Solution:**
```yaml
with:
  only_verified: false  # Fall back to all detections
```

---

## Configuration Issues

### Issue: Input Parameter Not Working

**Symptoms:**
- Configuration seems ignored
- Unexpected behavior

**Check:**

1. **Parameter name spelling**
   ```yaml
   # ❌ Wrong
   with:
     fail_on_sev: 'HIGH'
   
   # ✅ Correct
   with:
     fail_on_severity: 'HIGH'
   ```

2. **Parameter type mismatch**
   ```yaml
   # ❌ Wrong - should be string
   with:
     fail_on_severity: HIGH
   
   # ✅ Correct
   with:
     fail_on_severity: 'HIGH'
   ```

3. **Parameter in wrong section**
   ```yaml
   # ❌ Wrong - with values must be in 'with' block
   jobs:
     scan:
       uses: workflow@main
       env:
         fail_on_severity: 'HIGH'  # WRONG location
   
   # ✅ Correct
   jobs:
     scan:
       uses: workflow@main
       with:
         fail_on_severity: 'HIGH'
   ```

---

### Issue: Skip Patterns Not Working

**Cause:** Wrong glob pattern format

**Solution:**
```yaml
# ❌ Wrong - uses regex
skip_paths: 'tests/*.py'

# ✅ Correct - uses glob
skip_paths: 'tests,examples,vendor'

# For complex patterns
skip_paths: |
  tests
  examples
  vendor
  .terraform
  node_modules
```

---

### Issue: Config File Not Found

**Symptoms:**
- Error: "No such file or directory: .semgrep.yml"

**Solution:**

1. **Verify file exists**
   ```bash
   ls -la .semgrep.yml  # Check if file exists
   ```

2. **Use absolute path**
   ```yaml
   with:
     config_file: '${{ github.workspace }}/.semgrep.yml'
   ```

3. **Handle file in subdirectory**
   ```yaml
   with:
     config_file: 'config/.semgrep.yml'
   ```

---

## Notification Issues

### Issue: Webhook Not Receiving Messages

**Symptoms:**
- No notifications sent
- Error: "Failed to send notification"

**Causes & Solutions:**

1. **Missing or invalid webhook URL**
   ```yaml
   # Check GitHub Secrets
   # Settings > Secrets > Actions > SECURITY_WEBHOOK
   
   # Verify webhook format
   # Teams: https://outlook.webhook.office.com/webhookb2/...
   # Slack: https://hooks.slack.com/services/...
   ```

2. **Webhook token expired (Teams)**
   - Regenerate webhook in Teams
   - Update GitHub secret

3. **Slack workspace permissions**
   - Verify bot has permission to post
   - Check channel visibility

**Debugging:**
```bash
# Test webhook manually
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Test"}' \
  YOUR_WEBHOOK_URL
```

---

### Issue: Notification Payload Too Large

**Symptoms:**
- Notification fails silently
- Incomplete notification

**Solution:**
```yaml
# Reduce detail level
with:
  comment_pr: false  # Skip PR comments
  quiet_mode: true   # Less verbose output
```

---

### Issue: Notification Format Broken

**Symptoms:**
- Message garbled or incomplete
- Special characters missing

**Solution:**
```yaml
# Use JSON escaping
- name: Notify
  uses: notify-workflow@main
  with:
    message: ${{ toJson(env.FINDINGS) }}  # Proper escaping
```

---

## Performance Issues

### Issue: Workflow Runs Too Slow

**Optimization Steps:**

1. **Reduce scope**
   ```yaml
   # Limit to changed files
   skip_paths: 'tests,examples,vendor'
   ```

2. **Enable caching**
   ```yaml
   - name: Cache dependencies
     uses: actions/cache@v3
     with:
       path: ~/.trivy/cache
       key: trivy-${{ runner.os }}
   ```

3. **Parallel scanning**
   ```yaml
   jobs:
     scan-python:
       uses: workflow@main
       with:
         languages: 'python'
     
     scan-js:
       uses: workflow@main
       with:
         languages: 'javascript'
   ```

4. **Reduce severity levels**
   ```yaml
   with:
     severity: 'CRITICAL,HIGH'  # Only critical
   ```

---

### Issue: Artifact Storage Growing

**Solution:**
```yaml
# Clean old artifacts
jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Delete old artifacts
        uses: geekyeggo/delete-artifact@v2
        with:
          name: sca-scan-results
          failOnError: false
```

---

## Integration Issues

### Issue: PR Comment Not Appearing

**Causes:**
1. Insufficient permissions
2. Comment disabled in workflow
3. Workflow failed before comment step

**Solutions:**

1. **Check permissions**
   ```
   Settings > Actions > General > Workflow Permissions
   → Select "Read and write permissions"
   ```

2. **Enable comment**
   ```yaml
   with:
     comment_pr: true
   ```

3. **Ensure step runs**
   ```yaml
   - name: Comment Results
     if: always()  # Run even if previous steps failed
     uses: workflow@main
   ```

---

### Issue: GitHub Issue Not Created

**Causes:**
1. Issues disabled in repository
2. Create issue disabled in workflow
3. Insufficient permissions

**Solutions:**

1. **Enable issues**
   ```
   Settings > General > Issues
   → Ensure "Issues" is checked
   ```

2. **Enable creation**
   ```yaml
   with:
     create_issue: true
   ```

3. **Check permissions**
   ```
   Settings > Actions > General > Workflow Permissions
   ```

---

### Issue: SARIF Upload Not Working

**Cause:** Incorrect SARIF format

**Solution:**
```bash
# Validate SARIF locally
npm install -g @microsoft/sarif-multitool
codeql database create --sarif-category=sast results.sarif
```

Or disable:
```yaml
with:
  upload_sarif: false
```

---

## Getting Help

### Debug Information to Collect

When reporting issues:

1. **Workflow logs** - Full GitHub Actions logs
2. **Configuration** - Workflow YAML (redact secrets)
3. **Error message** - Complete error output
4. **Tool versions** - trivy --version, semgrep --version, etc.
5. **Environment** - OS, runner specs
6. **Reproduction steps** - How to reproduce

### Check Existing Issues

**GitHub Repositories:**
- [appsec-static-scanner](https://github.com/your-org/appsec-static-scanner/issues)
- [Trivy Issues](https://github.com/aquasecurity/trivy/issues)
- [Semgrep Issues](https://github.com/returntocorp/semgrep/issues)
- [Checkov Issues](https://github.com/bridgecrewio/checkov/issues)
- [TruffleHog Issues](https://github.com/trufflesecurity/trufflehog/issues)

### Contact Security Team

```
Email: security@your-org.com
Slack: #security-engineering
Jira: Create ticket in SECURITY project
```

---

## FAQ

### Q: How do I skip a specific finding?

**A:** Use tool-specific suppression:

```python
# Semgrep
# nosemgrep: python.flask.insecure-deserialization
data = pickle.loads(user_input)
```

```yaml
# Checkov
resource "aws_s3_bucket" "example" {
  bucket = "test"
  # checkov:skip=CKV_AWS_1:not_needed_for_now
}
```

---

### Q: How do I run scans on a schedule?

**A:**
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
```

---

### Q: Can I run scans on specific files?

**A:**
```yaml
on:
  pull_request:
    paths:
      - 'src/**'
      - 'terraform/**'
```

---

### Q: How do I configure different settings per branch?

**A:**
```yaml
jobs:
  main-branch:
    if: github.ref == 'refs/heads/main'
    with:
      fail_on_severity: 'HIGH'

  develop-branch:
    if: github.ref == 'refs/heads/develop'
    with:
      fail_on_finding: false
```

---

### Q: How do I export results for reporting?

**A:**
```yaml
with:
  output_format: 'json,sarif,csv'
  upload_artifacts: true

# Download from: Actions > Workflow Run > Artifacts
```

---

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Workflow Syntax](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
- [Secrets Management](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

---

**Last Updated:** January 18, 2026
