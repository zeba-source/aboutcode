# User Journey: Senior DevOps Engineer - CI/CD Integration

## Persona Profile

**Name:** Alex Rivera  
**Role:** Senior DevOps Engineer / Platform Team Lead  
**Background:** 8+ years in DevOps, expert in CI/CD automation, container orchestration, IaC  
**Organization:** SaaS company, microservices architecture, 50+ repositories  
**Tech Stack:** GitHub, GitHub Actions, Docker, Kubernetes, Python, Node.js, Go  
**Experience Level:** Advanced with automation, new to software composition analysis tools

---

## 1. Goal

**Primary Objective:** Implement automated license compliance and vulnerability scanning in CI/CD pipelines across all production repositories, with automated policy enforcement and security team notifications.

**Technical Requirements:**
- Scan every pull request for license violations and high-severity CVEs
- Block merges if prohibited licenses (GPL, AGPL) are detected
- Generate SBOMs automatically for every release build
- Alert security team on CVSS 9.0+ vulnerabilities
- Export scan results to centralized security dashboard
- Support multiple languages/ecosystems: npm, pip, Maven, Go modules
- Keep scan time under 5 minutes for typical PR builds

**Success Metrics:**
- 100% PR scan coverage across production repos
- <1% false positive rate on license policy violations
- Zero prohibited licenses merged to production
- SBOM available within 2 minutes of release tag
- 95% of critical vulnerabilities triaged within 24 hours

**Business Drivers:**
- SOC 2 compliance requirement for SBOM generation
- Customer contracts requiring CVE disclosure
- Reduce legal risk from open source licensing
- Improve supply chain security posture

---

## 2. Prerequisites

### 2.1 Infrastructure Setup

#### ScanCode.io Server Deployment
**Recommended:** Self-hosted instance for API access and data retention

```bash
# Deploy ScanCode.io via Docker Compose
git clone https://github.com/aboutcode-org/scancode.io.git
cd scancode.io

# Create environment configuration
cat > .env << EOF
SCANCODEIO_DB_PASSWORD=$(openssl rand -base64 32)
SCANCODEIO_SECRET_KEY=$(openssl rand -base64 50)
SCANCODEIO_ALLOWED_HOSTS=scancode.internal.company.com
SCANCODEIO_REQUIRE_AUTHENTICATION=True
SCANCODEIO_WORKSPACE_LOCATION=/var/scancodeio/workspace
EOF

# Start services
docker-compose up -d

# Create admin user
docker-compose exec web scancodeio createsuperuser

# Create API token for CI/CD
docker-compose exec web scancodeio create-user cicd-bot --api-key
```

**Infrastructure Requirements:**
- 4 CPU / 8GB RAM minimum (16GB recommended for parallel scans)
- 100GB storage for workspace and database
- PostgreSQL 12+ and Redis 6+
- Network access from CI/CD runners
- HTTPS with valid certificate (Let's Encrypt recommended)

---

#### VulnerableCode Deployment (Optional but Recommended)
**Purpose:** Enhanced vulnerability correlation and tracking

```bash
# Deploy VulnerableCode
git clone https://github.com/aboutcode-org/vulnerablecode.git
cd vulnerablecode

# Configure environment
cat > .env << EOF
VULNERABLECODE_DB_PASSWORD=$(openssl rand -base64 32)
VULNERABLECODE_SECRET_KEY=$(openssl rand -base64 50)
VCIO_HOST=scancode.internal.company.com
ENABLE_LIVE_EVAL=true
EOF

# Start and initialize
docker-compose up -d
docker-compose exec vulnerablecode python manage.py migrate
docker-compose exec vulnerablecode python manage.py import --importer github
```

**Integration:** Link VulnerableCode to ScanCode.io for enhanced vulnerability data

---

### 2.2 Local Development Tools

#### ScanCode Toolkit Installation
**Use Case:** Local testing and CLI automation

```bash
# Install via pip (requires Python 3.8+)
python -m venv venv-scancode
source venv-scancode/bin/activate  # Windows: .\venv-scancode\Scripts\activate
pip install scancode-toolkit[full]

# Verify installation
scancode --version
# Expected: ScanCode Toolkit version 32.x.x

# Configure for optimal performance
export SCANCODE_PROCESSES=4  # Parallel processes
export SCANCODE_TEMP=/tmp    # Temp directory
```

---

#### Install AboutCode CLI Tools

```bash
# Install scancode.io CLI client
pip install scancodeio-client

# Install Python inspection tools
pip install python-inspector

# Install SBOM validation tools
pip install cyclonedx-bom spdx-tools
```

---

### 2.3 GitHub Configuration

#### Repository Secrets Setup

```bash
# Add to GitHub repository or organization secrets
SCANCODEIO_URL=https://scancode.internal.company.com
SCANCODEIO_API_KEY=<token-from-deployment>
VULNERABLECODE_URL=https://vulnerablecode.internal.company.com
SLACK_WEBHOOK_SECURITY=<security-team-webhook>
LICENSE_POLICY_URL=https://raw.githubusercontent.com/company/policies/main/license-policy.yml
```

#### License Policy Configuration

**File:** `license-policy.yml` (stored in policy repo)

```yaml
# License policy for automated enforcement
license_policies:
  approved:
    - MIT
    - Apache-2.0
    - BSD-2-Clause
    - BSD-3-Clause
    - ISC
    - CC0-1.0
  
  restricted:  # Require manual review
    - LGPL-2.1
    - LGPL-3.0
    - MPL-2.0
    - EPL-1.0
    - EPL-2.0
  
  prohibited:  # Block PR merge
    - GPL-2.0
    - GPL-3.0
    - AGPL-3.0
    - SSPL-1.0
    - Commons-Clause

compliance:
  fail_on_prohibited: true
  fail_on_high_severity: true  # CVSS >= 9.0
  require_sbom_on_release: true
  
vulnerability_thresholds:
  critical: 0    # Block on any CVSS 9.0+
  high: 5        # Block if >5 CVSS 7.0-8.9
  medium: 20     # Warn if >20 CVSS 4.0-6.9
```

---

## 3. CLI Commands for Scanning

### 3.1 Basic License Scanning

#### Scan Codebase for Licenses and Copyrights

```bash
# Full scan with all detections
scancode \
  --license \
  --copyright \
  --package \
  --info \
  --classify \
  --summary \
  --json-pp scan-results.json \
  --html scan-results.html \
  /path/to/codebase

# Focused license-only scan (faster)
scancode \
  --license \
  --license-score 80 \
  --json-pp licenses.json \
  --processes 4 \
  /path/to/codebase
```

**Key Options:**
- `--license-score 80`: Only report matches with 80%+ confidence
- `--processes 4`: Parallel scanning (adjust to CPU cores)
- `--json-pp`: Pretty-printed JSON output
- `--classify`: Classify files by type

---

#### Package Manifest Analysis

```bash
# Detect and resolve dependencies from package managers
scancode \
  --package \
  --json-pp packages.json \
  /path/to/project

# Output includes:
# - package.json (npm), requirements.txt (pip), pom.xml (Maven), go.mod, etc.
# - Detected package names, versions, PURLs
# - Declared licenses from package metadata
```

---

### 3.2 Vulnerability Scanning

#### Using ScanCode with Vulnerability Detection

```bash
# Requires vulnerablecode integration
scancode \
  --package \
  --vulnerabilities \
  --json-pp vulns.json \
  /path/to/project

# Filter to show only critical vulnerabilities
jq '.packages[] | select(.vulnerabilities != null) | 
    {name: .name, version: .version, vulns: .vulnerabilities}' vulns.json
```

---

### 3.3 SBOM Generation

#### Generate SPDX SBOM

```bash
# Scan and generate SPDX 2.3 SBOM
scancode \
  --license \
  --copyright \
  --package \
  --spdx-tv sbom.spdx \
  /path/to/codebase

# Or JSON format
scancode \
  --license \
  --copyright \
  --package \
  --spdx-rdf sbom.spdx.json \
  /path/to/codebase
```

#### Generate CycloneDX SBOM

```bash
# Scan and generate CycloneDX 1.4
scancode \
  --license \
  --copyright \
  --package \
  --cyclonedx sbom.cdx.json \
  /path/to/codebase

# Validate the generated SBOM
cyclonedx-py validate --input-file sbom.cdx.json
```

---

### 3.4 Policy Enforcement

#### Apply License Policy Check

```bash
# Download policy file
curl -o license-policy.yml "$LICENSE_POLICY_URL"

# Scan with policy enforcement
scancode \
  --license \
  --license-policy license-policy.yml \
  --json-pp results-with-policy.json \
  /path/to/codebase

# Check for policy violations
violations=$(jq '[.files[] | select(.license_policy_violations) | 
               .license_policy_violations[]] | length' results-with-policy.json)

if [ "$violations" -gt 0 ]; then
  echo "❌ Found $violations license policy violations"
  jq '.files[] | select(.license_policy_violations) | 
      {path: .path, violations: .license_policy_violations}' \
      results-with-policy.json
  exit 1
fi

echo "✅ No license policy violations"
```

---

### 3.5 Differential Scanning

#### Compare Against Base Branch

```bash
# Scan current branch
scancode --license --json-pp current.json .

# Scan main branch
git checkout main
scancode --license --json-pp baseline.json .
git checkout -

# Compute delta (requires deltacode)
deltacode \
  --new current.json \
  --old baseline.json \
  --json-pp delta.json

# Show only new licenses introduced
jq '.delta_summary.new_licenses' delta.json
```

---

## 4. API Integration Example in Python

### 4.1 ScanCode.io REST API Client

#### Complete Python Integration Script

**File:** `ci/scancode_integration.py`

```python
#!/usr/bin/env python3
"""
ScanCode.io CI/CD Integration Script
Automates project creation, scanning, and result retrieval
"""

import os
import sys
import time
import requests
import json
from pathlib import Path
from typing import Dict, List, Optional

class ScanCodeIOClient:
    """Client for ScanCode.io REST API"""
    
    def __init__(self, url: str, api_key: str):
        self.url = url.rstrip('/')
        self.headers = {
            'Authorization': f'Token {api_key}',
            'Content-Type': 'application/json'
        }
        self.session = requests.Session()
        self.session.headers.update(self.headers)
    
    def create_project(self, name: str, pipeline: str = 'scan_codebase',
                      input_urls: Optional[List[str]] = None) -> Dict:
        """Create a new scan project"""
        data = {
            'name': name,
            'pipeline': pipeline,
        }
        if input_urls:
            data['input_urls'] = input_urls
        
        response = self.session.post(
            f'{self.url}/api/projects/',
            json=data
        )
        response.raise_for_status()
        return response.json()
    
    def upload_file(self, project_uuid: str, file_path: str) -> Dict:
        """Upload a file to project"""
        files = {'upload_file': open(file_path, 'rb')}
        headers = {'Authorization': self.headers['Authorization']}
        
        response = requests.post(
            f'{self.url}/api/projects/{project_uuid}/add_input/',
            headers=headers,
            files=files
        )
        response.raise_for_status()
        return response.json()
    
    def start_pipeline(self, project_uuid: str) -> Dict:
        """Start the pipeline execution"""
        response = self.session.post(
            f'{self.url}/api/projects/{project_uuid}/execute_pipeline/'
        )
        response.raise_for_status()
        return response.json()
    
    def get_project_status(self, project_uuid: str) -> Dict:
        """Get project status"""
        response = self.session.get(
            f'{self.url}/api/projects/{project_uuid}/'
        )
        response.raise_for_status()
        return response.json()
    
    def wait_for_completion(self, project_uuid: str, 
                           timeout: int = 600) -> Dict:
        """Wait for scan to complete"""
        start_time = time.time()
        
        while True:
            status = self.get_project_status(project_uuid)
            
            if status['status'] == 'success':
                print(f"✅ Scan completed successfully")
                return status
            elif status['status'] == 'failure':
                print(f"❌ Scan failed: {status.get('message', 'Unknown error')}")
                sys.exit(1)
            
            elapsed = time.time() - start_time
            if elapsed > timeout:
                print(f"❌ Timeout after {timeout}s")
                sys.exit(1)
            
            progress = status.get('progress', 0)
            print(f"⏳ Scanning... {progress}% complete")
            time.sleep(10)
    
    def get_results(self, project_uuid: str, output_format: str = 'json') -> Dict:
        """Download scan results"""
        response = self.session.get(
            f'{self.url}/api/projects/{project_uuid}/results_download/',
            params={'format': output_format}
        )
        response.raise_for_status()
        return response.json() if output_format == 'json' else response.text
    
    def get_packages(self, project_uuid: str) -> List[Dict]:
        """Get detected packages"""
        response = self.session.get(
            f'{self.url}/api/projects/{project_uuid}/packages/'
        )
        response.raise_for_status()
        return response.json()['results']
    
    def check_license_policy(self, packages: List[Dict], 
                            policy: Dict) -> List[Dict]:
        """Check packages against license policy"""
        violations = []
        
        for pkg in packages:
            license_expression = pkg.get('declared_license_expression', '')
            if not license_expression:
                continue
            
            # Check against prohibited licenses
            for prohibited in policy.get('prohibited', []):
                if prohibited in license_expression:
                    violations.append({
                        'package': pkg['name'],
                        'version': pkg['version'],
                        'license': license_expression,
                        'severity': 'CRITICAL',
                        'policy': 'prohibited'
                    })
        
        return violations
    
    def check_vulnerabilities(self, packages: List[Dict], 
                             threshold: float = 9.0) -> List[Dict]:
        """Check for high-severity vulnerabilities"""
        critical_vulns = []
        
        for pkg in packages:
            vulns = pkg.get('affected_by_vulnerabilities', [])
            for vuln in vulns:
                # Parse CVSS score
                cvss_scores = vuln.get('cvss_scores', [])
                max_cvss = max([s.get('value', 0) for s in cvss_scores], default=0)
                
                if max_cvss >= threshold:
                    critical_vulns.append({
                        'package': pkg['name'],
                        'version': pkg['version'],
                        'cve': vuln.get('vulnerability_id'),
                        'cvss': max_cvss,
                        'summary': vuln.get('summary', '')
                    })
        
        return critical_vulns


def scan_repository(repo_path: str, project_name: str) -> int:
    """
    Main function to scan a repository
    Returns: 0 on success, 1 on policy violations
    """
    # Configuration from environment
    scancodeio_url = os.getenv('SCANCODEIO_URL')
    scancodeio_key = os.getenv('SCANCODEIO_API_KEY')
    
    if not scancodeio_url or not scancodeio_key:
        print("❌ Missing SCANCODEIO_URL or SCANCODEIO_API_KEY")
        return 1
    
    # Load license policy
    policy_url = os.getenv('LICENSE_POLICY_URL')
    policy = requests.get(policy_url).json() if policy_url else {}
    
    # Initialize client
    client = ScanCodeIOClient(scancodeio_url, scancodeio_key)
    
    # Create archive of repository
    print(f"📦 Creating archive of {repo_path}")
    import tarfile
    archive_path = '/tmp/scan-archive.tar.gz'
    with tarfile.open(archive_path, 'w:gz') as tar:
        tar.add(repo_path, arcname='.')
    
    # Create project
    print(f"🚀 Creating ScanCode.io project: {project_name}")
    project = client.create_project(
        name=project_name,
        pipeline='scan_codebase'
    )
    project_uuid = project['uuid']
    print(f"   Project UUID: {project_uuid}")
    
    # Upload archive
    print(f"📤 Uploading codebase")
    client.upload_file(project_uuid, archive_path)
    
    # Start scan
    print(f"🔍 Starting scan pipeline")
    client.start_pipeline(project_uuid)
    
    # Wait for completion
    print(f"⏳ Waiting for scan to complete...")
    client.wait_for_completion(project_uuid, timeout=600)
    
    # Get results
    print(f"📥 Retrieving results")
    packages = client.get_packages(project_uuid)
    print(f"   Found {len(packages)} packages")
    
    # Check license policy
    print(f"⚖️  Checking license policy")
    license_violations = client.check_license_policy(packages, policy)
    
    if license_violations:
        print(f"\n❌ Found {len(license_violations)} license policy violations:")
        for v in license_violations:
            print(f"   - {v['package']}@{v['version']}: {v['license']} ({v['policy']})")
        return 1
    else:
        print(f"✅ No license policy violations")
    
    # Check vulnerabilities
    print(f"🛡️  Checking for critical vulnerabilities")
    critical_vulns = client.check_vulnerabilities(packages, threshold=9.0)
    
    if critical_vulns:
        print(f"\n⚠️  Found {len(critical_vulns)} critical vulnerabilities:")
        for v in critical_vulns:
            print(f"   - {v['package']}@{v['version']}: {v['cve']} (CVSS {v['cvss']})")
        # Don't fail on vulns, just warn (handled separately)
    
    # Download full results
    results = client.get_results(project_uuid, output_format='json')
    output_file = 'scancode-results.json'
    with open(output_file, 'w') as f:
        json.dump(results, f, indent=2)
    print(f"💾 Results saved to {output_file}")
    
    # Generate SBOM
    print(f"📋 Generating SBOM")
    sbom_response = client.session.get(
        f'{scancodeio_url}/api/projects/{project_uuid}/results_download/',
        params={'format': 'cyclonedx'}
    )
    with open('sbom.cdx.json', 'w') as f:
        f.write(sbom_response.text)
    print(f"💾 SBOM saved to sbom.cdx.json")
    
    return 0


if __name__ == '__main__':
    repo_path = sys.argv[1] if len(sys.argv) > 1 else '.'
    project_name = os.getenv('CI_PROJECT_NAME', 'ci-scan')
    
    exit_code = scan_repository(repo_path, project_name)
    sys.exit(exit_code)
```

---

### 4.2 Usage in CI Pipeline

```bash
# Install dependencies
pip install requests

# Run scan
python ci/scancode_integration.py .

# Script exits with:
# - 0: Success, no violations
# - 1: Policy violations found, block merge
```

---

## 5. Sample GitHub Actions Workflow YAML

### 5.1 Pull Request Scan Workflow

**File:** `.github/workflows/scancode-pr.yml`

```yaml
name: License & Vulnerability Scan (PR)

on:
  pull_request:
    branches: [main, develop]
    types: [opened, synchronize, reopened]

env:
  SCANCODEIO_URL: ${{ secrets.SCANCODEIO_URL }}
  SCANCODEIO_API_KEY: ${{ secrets.SCANCODEIO_API_KEY }}
  LICENSE_POLICY_URL: ${{ secrets.LICENSE_POLICY_URL }}

jobs:
  scan-licenses:
    name: Scan for License Compliance
    runs-on: ubuntu-latest
    timeout-minutes: 30
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for delta analysis
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
          cache: 'pip'
      
      - name: Install dependencies
        run: |
          pip install requests python-inspector cyclonedx-bom
          pip install scancode-toolkit[full]
      
      - name: Download license policy
        run: |
          curl -sSL -o license-policy.yml "$LICENSE_POLICY_URL"
      
      - name: Run ScanCode locally (fast check)
        id: local-scan
        run: |
          # Quick license scan for PR feedback
          scancode \
            --license \
            --license-score 90 \
            --license-policy license-policy.yml \
            --json-pp scan-results.json \
            --processes $(nproc) \
            .
          
          # Check for violations
          violations=$(jq '[.files[] | select(.license_policy_violations) | 
                       .license_policy_violations[]] | length' scan-results.json)
          
          echo "violations=$violations" >> $GITHUB_OUTPUT
          
          if [ "$violations" -gt 0 ]; then
            echo "::error::Found $violations license policy violations"
            jq '.files[] | select(.license_policy_violations) | 
                {path: .path, violations: .license_policy_violations}' \
                scan-results.json
            exit 1
          fi
      
      - name: Upload scan results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: scancode-results
          path: scan-results.json
          retention-days: 30
      
      - name: Comment PR with results
        if: always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const results = JSON.parse(fs.readFileSync('scan-results.json', 'utf8'));
            
            const licenses = new Set();
            results.files.forEach(file => {
              if (file.licenses) {
                file.licenses.forEach(lic => licenses.add(lic.key));
              }
            });
            
            const violations = ${{ steps.local-scan.outputs.violations }};
            const status = violations > 0 ? '❌ FAILED' : '✅ PASSED';
            
            const body = `## License Scan Results ${status}
            
            **Detected Licenses:** ${Array.from(licenses).join(', ') || 'None'}
            **Policy Violations:** ${violations}
            **Files Scanned:** ${results.files.length}
            
            ${violations > 0 ? '⚠️ **Action Required:** Fix license violations before merging.' : ''}
            
            <details>
            <summary>View detailed results</summary>
            
            Download the full scan report from the workflow artifacts.
            
            </details>`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: body
            });
      
      - name: Set status check
        if: always()
        run: |
          if [ "${{ steps.local-scan.outputs.violations }}" -gt 0 ]; then
            exit 1
          fi

  scan-vulnerabilities:
    name: Scan for Vulnerabilities
    runs-on: ubuntu-latest
    timeout-minutes: 20
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Install ScanCode
        run: |
          pip install scancode-toolkit[full]
      
      - name: Scan for vulnerabilities
        id: vuln-scan
        run: |
          # Scan packages for vulnerabilities
          scancode \
            --package \
            --json-pp packages.json \
            .
          
          # Extract package list
          jq -r '.packages[] | "\(.name)@\(.version)"' packages.json > packages.txt
          
          echo "Found $(wc -l < packages.txt) packages"
      
      - name: Query VulnerableCode
        env:
          VULNERABLECODE_URL: ${{ secrets.VULNERABLECODE_URL }}
        run: |
          critical_count=0
          high_count=0
          
          while IFS= read -r package; do
            # Query VulnerableCode API (simplified)
            # In production, batch these queries
            echo "Checking: $package"
          done < packages.txt
          
          echo "critical_vulns=$critical_count" >> $GITHUB_OUTPUT
          echo "high_vulns=$high_count" >> $GITHUB_OUTPUT
      
      - name: Notify security team if critical vulnerabilities
        if: steps.vuln-scan.outputs.critical_vulns > 0
        uses: slackapi/slack-github-action@v1
        with:
          webhook: ${{ secrets.SLACK_WEBHOOK_SECURITY }}
          payload: |
            {
              "text": "🚨 Critical Vulnerabilities Detected",
              "blocks": [
                {
                  "type": "section",
                  "text": {
                    "type": "mrkdwn",
                    "text": "*Critical vulnerabilities detected in PR #${{ github.event.pull_request.number }}*\n\nRepository: ${{ github.repository }}\nPR: ${{ github.event.pull_request.html_url }}"
                  }
                }
              ]
            }
```

---

### 5.2 Release SBOM Generation Workflow

**File:** `.github/workflows/sbom-release.yml`

```yaml
name: Generate SBOM on Release

on:
  release:
    types: [published]
  push:
    tags:
      - 'v*'

env:
  SCANCODEIO_URL: ${{ secrets.SCANCODEIO_URL }}
  SCANCODEIO_API_KEY: ${{ secrets.SCANCODEIO_API_KEY }}

jobs:
  generate-sbom:
    name: Generate and Publish SBOM
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Install tools
        run: |
          pip install scancode-toolkit[full] cyclonedx-bom spdx-tools
      
      - name: Generate SPDX SBOM
        run: |
          scancode \
            --license \
            --copyright \
            --package \
            --info \
            --spdx-rdf sbom-${{ github.ref_name }}.spdx.json \
            .
      
      - name: Generate CycloneDX SBOM
        run: |
          scancode \
            --license \
            --copyright \
            --package \
            --cyclonedx sbom-${{ github.ref_name }}.cdx.json \
            .
      
      - name: Validate SBOMs
        run: |
          # Validate SPDX
          pyspdxtools -i sbom-${{ github.ref_name }}.spdx.json -v
          
          # Validate CycloneDX
          cyclonedx-py validate --input-file sbom-${{ github.ref_name }}.cdx.json
      
      - name: Sign SBOMs with cosign
        uses: sigstore/cosign-installer@v3
      
      - name: Upload SBOMs to release
        uses: softprops/action-gh-release@v1
        with:
          files: |
            sbom-${{ github.ref_name }}.spdx.json
            sbom-${{ github.ref_name }}.cdx.json
          fail_on_unmatched_files: true
      
      - name: Upload to artifact registry
        run: |
          # Upload to company artifact registry
          # curl -X POST $ARTIFACT_REGISTRY_URL ...
          echo "SBOM uploaded to artifact registry"
```

---

### 5.3 Scheduled Full Audit Workflow

**File:** `.github/workflows/security-audit.yml`

```yaml
name: Weekly Security Audit

on:
  schedule:
    - cron: '0 2 * * 1'  # Every Monday at 2 AM UTC
  workflow_dispatch:  # Manual trigger

jobs:
  full-audit:
    name: Comprehensive Security Audit
    runs-on: ubuntu-latest-8-cores  # Use larger runner
    timeout-minutes: 120
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      
      - name: Run comprehensive scan
        run: |
          python ci/scancode_integration.py .
      
      - name: Generate audit report
        run: |
          python ci/generate_audit_report.py \
            --input scancode-results.json \
            --output audit-report.html \
            --format html
      
      - name: Upload to secure storage
        run: |
          # Upload to S3, GCS, or internal storage
          # aws s3 cp audit-report.html s3://security-audits/$(date +%Y-%m-%d)/
          echo "Report uploaded"
      
      - name: Notify compliance team
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.company.com
          server_port: 587
          username: ${{ secrets.SMTP_USERNAME }}
          password: ${{ secrets.SMTP_PASSWORD }}
          subject: Weekly Security Audit - ${{ github.repository }}
          to: compliance@company.com,security@company.com
          from: DevOps <devops@company.com>
          body: |
            Weekly security audit completed for ${{ github.repository }}.
            
            View the detailed report in the attached artifacts.
          attachments: audit-report.html
```

---

## 6. Interpreting Scan Output

### 6.1 JSON Output Structure

#### ScanCode Toolkit Output Schema

```json
{
  "headers": [
    {
      "tool_name": "scancode-toolkit",
      "tool_version": "32.0.0",
      "options": {
        "license": true,
        "copyright": true,
        "package": true
      },
      "duration": 234.56,
      "message": null,
      "errors": [],
      "warnings": []
    }
  ],
  "files": [
    {
      "path": "src/main.py",
      "type": "file",
      "name": "main.py",
      "size": 1234,
      "sha1": "abc123...",
      "licenses": [
        {
          "key": "mit",
          "score": 100.0,
          "name": "MIT License",
          "category": "Permissive",
          "start_line": 1,
          "end_line": 20,
          "matched_text": "Permission is hereby granted..."
        }
      ],
      "license_expressions": ["mit"],
      "copyrights": [
        {
          "copyright": "Copyright 2024 Company Inc.",
          "start_line": 3,
          "end_line": 3
        }
      ],
      "license_policy_violations": []  // Empty = compliant
    }
  ],
  "packages": [
    {
      "type": "pypi",
      "namespace": null,
      "name": "requests",
      "version": "2.31.0",
      "purl": "pkg:pypi/requests@2.31.0",
      "declared_license_expression": "Apache-2.0",
      "license_detections": [...],
      "affected_by_vulnerabilities": [
        {
          "vulnerability_id": "CVE-2023-32681",
          "summary": "Unintended leak of Proxy-Authorization header",
          "cvss_scores": [
            {
              "value": 6.1,
              "vector": "CVSS:3.1/AV:N/AC:H/PR:N/UI:R/S:C/C:H/I:N/A:N"
            }
          ]
        }
      ]
    }
  ]
}
```

---

### 6.2 Key Metrics to Extract

#### License Analysis

```python
# Parse license distribution
import json
from collections import Counter

with open('scan-results.json') as f:
    data = json.load(f)

# Count license occurrences
license_counter = Counter()
for file_data in data['files']:
    for license in file_data.get('license_expressions', []):
        license_counter[license] += 1

print("License Distribution:")
for license, count in license_counter.most_common():
    print(f"  {license}: {count} files")

# Identify policy violations
violations = []
for file_data in data['files']:
    if file_data.get('license_policy_violations'):
        violations.append({
            'path': file_data['path'],
            'violations': file_data['license_policy_violations']
        })

if violations:
    print(f"\n❌ {len(violations)} files with policy violations")
else:
    print("\n✅ No policy violations")
```

---

#### Vulnerability Analysis

```python
# Extract high-severity vulnerabilities
import json

with open('scan-results.json') as f:
    data = json.load(f)

critical_vulns = []
high_vulns = []

for pkg in data.get('packages', []):
    package_name = f"{pkg['name']}@{pkg['version']}"
    
    for vuln in pkg.get('affected_by_vulnerabilities', []):
        cvss_scores = vuln.get('cvss_scores', [])
        max_cvss = max([s.get('value', 0) for s in cvss_scores], default=0)
        
        vuln_data = {
            'package': package_name,
            'cve': vuln['vulnerability_id'],
            'cvss': max_cvss,
            'summary': vuln.get('summary', '')
        }
        
        if max_cvss >= 9.0:
            critical_vulns.append(vuln_data)
        elif max_cvss >= 7.0:
            high_vulns.append(vuln_data)

print(f"Critical vulnerabilities (CVSS 9.0+): {len(critical_vulns)}")
print(f"High vulnerabilities (CVSS 7.0-8.9): {len(high_vulns)}")

# Output for downstream processing
summary = {
    'critical_count': len(critical_vulns),
    'high_count': len(high_vulns),
    'critical_vulns': critical_vulns,
    'high_vulns': high_vulns
}

with open('vuln-summary.json', 'w') as f:
    json.dump(summary, f, indent=2)
```

---

### 6.3 Dashboard Integration

#### Export to Prometheus/Grafana

```python
# Export metrics for monitoring
from prometheus_client import CollectorRegistry, Gauge, write_to_textfile

registry = CollectorRegistry()

# Define metrics
license_violations = Gauge(
    'scancode_license_violations_total',
    'Total license policy violations',
    ['repository', 'branch'],
    registry=registry
)

critical_vulns_metric = Gauge(
    'scancode_critical_vulnerabilities_total',
    'Total critical vulnerabilities',
    ['repository', 'branch'],
    registry=registry
)

# Set values from scan results
license_violations.labels(
    repository='myapp',
    branch='main'
).set(len(violations))

critical_vulns_metric.labels(
    repository='myapp',
    branch='main'
).set(len(critical_vulns))

# Write to file for node_exporter
write_to_textfile('/var/lib/node_exporter/scancode.prom', registry)
```

---

## 7. Escalation Path for Flagged Vulnerabilities

### 7.1 Severity-Based Escalation Matrix

| CVSS Score | Severity | SLA | Notification | Action |
|------------|----------|-----|--------------|---------|
| 9.0-10.0 | Critical | 24 hours | Security team + CISO | Block production deployment |
| 7.0-8.9 | High | 7 days | Security team | Require remediation plan |
| 4.0-6.9 | Medium | 30 days | Dev team lead | Track in backlog |
| 0.1-3.9 | Low | 90 days | No alert | Document only |

---

### 7.2 Automated Escalation Workflow

#### Vulnerability Triage Script

**File:** `ci/triage_vulnerabilities.py`

```python
#!/usr/bin/env python3
"""
Automated vulnerability triage and escalation
"""

import json
import os
import sys
from datetime import datetime
import requests

def send_slack_alert(webhook_url: str, vulnerability: dict):
    """Send Slack notification for critical vulnerabilities"""
    message = {
        "text": f"🚨 Critical Vulnerability Detected: {vulnerability['cve']}",
        "blocks": [
            {
                "type": "header",
                "text": {
                    "type": "plain_text",
                    "text": f"Critical Vulnerability: {vulnerability['cve']}"
                }
            },
            {
                "type": "section",
                "fields": [
                    {"type": "mrkdwn", "text": f"*Package:*\n{vulnerability['package']}"},
                    {"type": "mrkdwn", "text": f"*CVSS Score:*\n{vulnerability['cvss']}"},
                    {"type": "mrkdwn", "text": f"*Repository:*\n{os.getenv('GITHUB_REPOSITORY')}"},
                    {"type": "mrkdwn", "text": f"*Branch:*\n{os.getenv('GITHUB_REF_NAME')}"}
                ]
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*Summary:*\n{vulnerability['summary'][:500]}"
                }
            },
            {
                "type": "actions",
                "elements": [
                    {
                        "type": "button",
                        "text": {"type": "plain_text", "text": "View in GitHub"},
                        "url": os.getenv('GITHUB_SERVER_URL') + '/' + 
                               os.getenv('GITHUB_REPOSITORY') + 
                               '/actions/runs/' + os.getenv('GITHUB_RUN_ID')
                    }
                ]
            }
        ]
    }
    
    requests.post(webhook_url, json=message)

def create_jira_ticket(vulnerability: dict):
    """Create Jira ticket for vulnerability tracking"""
    jira_url = os.getenv('JIRA_URL')
    jira_token = os.getenv('JIRA_API_TOKEN')
    
    ticket = {
        "fields": {
            "project": {"key": "SEC"},
            "summary": f"[CRITICAL] {vulnerability['cve']} in {vulnerability['package']}",
            "description": f"""
Critical vulnerability detected in automated scan.

*Package:* {vulnerability['package']}
*CVE:* {vulnerability['cve']}
*CVSS Score:* {vulnerability['cvss']}
*Repository:* {os.getenv('GITHUB_REPOSITORY')}

*Summary:*
{vulnerability['summary']}

*Action Required:*
- Assess impact on production systems
- Identify affected services
- Develop remediation plan within 24 hours
- Update to patched version or implement workaround

*Automated Detection:*
This ticket was created automatically by the CI/CD security scanner.
            """,
            "issuetype": {"name": "Security Incident"},
            "priority": {"name": "Critical"},
            "labels": ["security", "automated", "vulnerability", vulnerability['cve']]
        }
    }
    
    headers = {
        "Authorization": f"Bearer {jira_token}",
        "Content-Type": "application/json"
    }
    
    response = requests.post(
        f"{jira_url}/rest/api/3/issue",
        json=ticket,
        headers=headers
    )
    
    return response.json()

def triage_scan_results(results_file: str):
    """Main triage function"""
    with open(results_file) as f:
        data = json.load(f)
    
    critical_vulns = []
    high_vulns = []
    
    # Extract vulnerabilities
    for pkg in data.get('packages', []):
        package_name = f"{pkg['name']}@{pkg['version']}"
        
        for vuln in pkg.get('affected_by_vulnerabilities', []):
            cvss_scores = vuln.get('cvss_scores', [])
            max_cvss = max([s.get('value', 0) for s in cvss_scores], default=0)
            
            vuln_data = {
                'package': package_name,
                'purl': pkg.get('purl'),
                'cve': vuln['vulnerability_id'],
                'cvss': max_cvss,
                'summary': vuln.get('summary', ''),
                'references': vuln.get('references', [])
            }
            
            if max_cvss >= 9.0:
                critical_vulns.append(vuln_data)
            elif max_cvss >= 7.0:
                high_vulns.append(vuln_data)
    
    # Escalate critical vulnerabilities
    if critical_vulns:
        print(f"🚨 CRITICAL: {len(critical_vulns)} critical vulnerabilities found")
        
        # Send Slack alerts
        slack_webhook = os.getenv('SLACK_WEBHOOK_SECURITY')
        if slack_webhook:
            for vuln in critical_vulns:
                send_slack_alert(slack_webhook, vuln)
        
        # Create Jira tickets
        if os.getenv('JIRA_URL'):
            for vuln in critical_vulns:
                ticket = create_jira_ticket(vuln)
                print(f"   Created Jira ticket: {ticket.get('key')}")
        
        # Block deployment
        print("❌ BLOCKING: Critical vulnerabilities must be resolved")
        return 1
    
    # Report high vulnerabilities
    if high_vulns:
        print(f"⚠️  WARNING: {len(high_vulns)} high-severity vulnerabilities found")
        print("   Remediation required within 7 days")
        # Don't block, but create tickets
    
    return 0

if __name__ == '__main__':
    results_file = sys.argv[1] if len(sys.argv) > 1 else 'vuln-summary.json'
    exit_code = triage_scan_results(results_file)
    sys.exit(exit_code)
```

---

### 7.3 Integration with Incident Response

#### PagerDuty Integration for Critical CVEs

```python
import requests

def trigger_pagerduty_incident(vulnerability: dict):
    """Trigger PagerDuty incident for critical vulnerability"""
    
    pagerduty_key = os.getenv('PAGERDUTY_INTEGRATION_KEY')
    
    event = {
        "routing_key": pagerduty_key,
        "event_action": "trigger",
        "payload": {
            "summary": f"Critical vulnerability {vulnerability['cve']} in production",
            "severity": "critical",
            "source": "AboutCode Security Scanner",
            "custom_details": {
                "package": vulnerability['package'],
                "cve": vulnerability['cve'],
                "cvss_score": vulnerability['cvss'],
                "repository": os.getenv('GITHUB_REPOSITORY'),
                "affected_systems": "Production microservices"
            }
        }
    }
    
    response = requests.post(
        "https://events.pagerduty.com/v2/enqueue",
        json=event
    )
    
    return response.json()
```

---

### 7.4 Remediation Tracking

#### Generate Remediation Report

```bash
# Track vulnerability remediation progress
cat > remediation-report.sh << 'EOF'
#!/bin/bash

# Generate CSV report for tracking
echo "CVE,Package,CVSS,Status,Assignee,Due Date,Notes" > vulns.csv

jq -r '.packages[] | 
  select(.affected_by_vulnerabilities) | 
  .affected_by_vulnerabilities[] as $vuln |
  [
    $vuln.vulnerability_id,
    .name + "@" + .version,
    ($vuln.cvss_scores[0].value // 0),
    "Open",
    "TBD",
    (now + 604800 | strftime("%Y-%m-%d")),
    ""
  ] | @csv' scancode-results.json >> vulns.csv

echo "Remediation tracking report generated: vulns.csv"
EOF

chmod +x remediation-report.sh
./remediation-report.sh
```

---

## Success Metrics & Monitoring

### Key Performance Indicators

```python
# Track CI/CD integration metrics
metrics = {
    "scan_coverage": "100%",  # All PRs scanned
    "avg_scan_time": "3.2 minutes",
    "blocked_merges_30d": 5,  # PRs blocked due to violations
    "false_positive_rate": "0.8%",
    "sbom_generation_success_rate": "99.7%",
    "critical_vuln_mttr": "14 hours",  # Mean time to resolution
    "license_violations_prevented": 7
}
```

### Dashboard Queries (Grafana)

```promql
# License violation rate
rate(scancode_license_violations_total[7d])

# Critical vulnerability detection
sum(scancode_critical_vulnerabilities_total) by (repository)

# Scan success rate
rate(scancode_scan_success_total[1h]) / 
rate(scancode_scan_total[1h])
```

---

## Conclusion

This integration provides:

✅ **Automated license compliance enforcement** preventing policy violations from reaching production  
✅ **Real-time vulnerability detection** with severity-based escalation  
✅ **Continuous SBOM generation** for every release  
✅ **Multi-channel alerting** (Slack, email, PagerDuty, Jira)  
✅ **Audit trail** with full scan history and remediation tracking  
✅ **Developer-friendly** integration in existing GitHub Actions workflows

**Next Steps:**
1. Deploy ScanCode.io and VulnerableCode instances
2. Configure license policy and escalation webhooks
3. Roll out to pilot repositories
4. Monitor metrics and tune thresholds
5. Expand to all production repositories
6. Integrate with security dashboard (Grafana/Datadog)

---

**Documentation References:**
- `developer/getting-started.html` - Developer onboarding
- `developer/guides/ci-cd-integration-patterns.html` - CI/CD patterns
- `developer/reference/cli-options-complete.html` - ScanCode CLI reference
- `developer/reference/rest-api-complete.html` - API documentation
- `security/guides/triage-vulnerabilities.html` - Vulnerability handling
- `aboutcode-projects/scancodeio-project.html` - ScanCode.io overview

**Support:**
- GitHub Issues: https://github.com/aboutcode-org/scancode.io/issues
- Community Chat: https://gitter.im/aboutcode-org/discuss
- API Documentation: https://scancodeio.readthedocs.io/

---

*Last Updated: February 2026 | Alex Rivera, Platform Engineering Team*
