.. _developer_getting_started:

##########################################
Getting Started: Developer & Integrator
##########################################

This guide shows you how to integrate AboutCode tools into your development workflow, automate scanning in CI/CD pipelines, and use APIs for programmatic access. You'll learn the fundamentals of building automated compliance and security workflows using AboutCode's suite of open source tools.

************
Prerequisites
************

Before you begin, ensure you have:

- Development environment with Python 3.8+, Node.js, or Java (depending on your project)
- Access to your CI/CD system (GitHub Actions, GitLab CI, Jenkins, etc.)
- Basic understanding of REST APIs and command-line tools
- Familiarity with your build and deployment process


******************
Quick Start Path
******************

Step 1: Install Command-Line Tools
===================================

**ScanCode Toolkit**

.. code-block:: bash

   pip install scancode-toolkit
   
   # Verify installation
   scancode --version

**VulnerableCode CLI (optional)**

.. code-block:: bash

   pip install vulnerablecode
   
   # Configure API endpoint
   export VULNERABLECODE_URL="https://public.vulnerablecode.io"

For detailed installation options, see :doc:`../aboutcode-projects/scancode-toolkit-project`.


Step 2: Run a Command-Line Scan
================================

Scan your project directory to generate structured output:

.. code-block:: bash

   # Full scan with JSON output
   scancode --license --copyright --package --info \
            --json-pp output.json \
            /path/to/your/project
   
   # Quick scan for licenses only
   scancode --license --json-pp licenses.json /path/to/your/project
   
   # Generate SPDX SBOM
   scancode --spdx output.spdx /path/to/your/project

**Output Formats:**

- ``--json`` or ``--json-pp`` - JSON (pretty-printed)
- ``--spdx`` - SPDX format SBOM
- ``--cyclonedx`` - CycloneDX format SBOM
- ``--csv`` - CSV for spreadsheet analysis
- ``--html`` - HTML report for human review


Step 3: Integrate with CI/CD
=============================

**GitHub Actions Example**

Create ``.github/workflows/scancode.yml``:

.. code-block:: yaml

   name: License and Security Scan
   
   on: [push, pull_request]
   
   jobs:
     scan:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v3
         
         - name: Run ScanCode
           uses: aboutcode-org/scancode-action@v1
           with:
             path: '.'
             output: 'scancode-results.json'
         
         - name: Upload results
           uses: actions/upload-artifact@v3
           with:
             name: scancode-results
             path: scancode-results.json
         
         - name: Check for license issues
           run: |
             python scripts/check-licenses.py scancode-results.json

See :doc:`../aboutcode-projects/scancode-action-project` for more GitHub Actions options.


**GitLab CI Example**

Add to ``.gitlab-ci.yml``:

.. code-block:: yaml

   scan_licenses:
     image: python:3.11
     stage: test
     script:
       - pip install scancode-toolkit
       - scancode --license --json-pp results.json .
       - python scripts/validate-licenses.py results.json
     artifacts:
       paths:
         - results.json
       expire_in: 1 week


**Jenkins Pipeline Example**

.. code-block:: groovy

   pipeline {
       agent any
       stages {
           stage('License Scan') {
               steps {
                   sh 'pip install scancode-toolkit'
                   sh 'scancode --license --json results.json .'
                   archiveArtifacts artifacts: 'results.json'
               }
           }
       }
   }


Step 4: Use REST APIs
======================

**ScanCode.io API**

Programmatically create projects and retrieve scan results:

.. code-block:: python

   import requests
   
   SCANCODEIO_URL = "https://your-scancodeio-instance.com"
   API_KEY = "your-api-key"
   
   headers = {"Authorization": f"Token {API_KEY}"}
   
   # Create a project
   response = requests.post(
       f"{SCANCODEIO_URL}/api/projects/",
       headers=headers,
       json={
           "name": "my-project",
           "input_urls": ["https://example.com/package.tar.gz"]
       }
   )
   project_uuid = response.json()['uuid']
   
   # Run a pipeline
   requests.post(
       f"{SCANCODEIO_URL}/api/projects/{project_uuid}/pipelines/",
       headers=headers,
       json={"pipeline": "scan_codebase"}
   )
   
   # Get results
   results = requests.get(
       f"{SCANCODEIO_URL}/api/projects/{project_uuid}/results/",
       headers=headers
   ).json()

**VulnerableCode API**

Query vulnerability data:

.. code-block:: python

   import requests
   
   # Search for vulnerabilities by package
   response = requests.get(
       'https://public.vulnerablecode.io/api/packages',
       params={'purl': 'pkg:npm/express@4.17.1'}
   )
   
   package_data = response.json()['results'][0]
   vulnerabilities = package_data['affected_by_vulnerabilities']
   
   # Get vulnerability details
   for vuln in vulnerabilities:
       vuln_response = requests.get(
           f"https://public.vulnerablecode.io{vuln['url']}"
       )
       details = vuln_response.json()
       print(f"CVE: {details['vulnerability_id']}")
       print(f"Severity: {details.get('severity', 'Unknown')}")

For complete API documentation, see :doc:`../aboutcode-projects/scancodeio-project` and :doc:`../aboutcode-projects/vulnerablecode-project`.


Step 5: Process and Filter Results
===================================

**Parse JSON Output**

.. code-block:: python

   import json
   
   with open('scancode-results.json', 'r') as f:
       data = json.load(f)
   
   # Extract all detected licenses
   licenses = set()
   for file in data['files']:
       if file.get('licenses'):
           for lic in file['licenses']:
               licenses.add(lic['key'])
   
   print(f"Detected licenses: {', '.join(sorted(licenses))}")
   
   # Find GPL-licensed files
   gpl_files = []
   for file in data['files']:
       if file.get('licenses'):
           for lic in file['licenses']:
               if 'gpl' in lic['key'].lower():
                   gpl_files.append(file['path'])
   
   if gpl_files:
       print(f"Warning: GPL license found in {len(gpl_files)} files")
       for path in gpl_files[:5]:  # Show first 5
           print(f"  - {path}")


**Filter by License Policy**

.. code-block:: python

   APPROVED_LICENSES = {'mit', 'apache-2.0', 'bsd-3-clause', 'bsd-2-clause'}
   FLAGGED_LICENSES = {'gpl-2.0', 'gpl-3.0', 'agpl-3.0'}
   
   violations = []
   for file in data['files']:
       if file.get('licenses'):
           for lic in file['licenses']:
               if lic['key'] in FLAGGED_LICENSES:
                   violations.append({
                       'file': file['path'],
                       'license': lic['key']
                   })
   
   if violations:
       print(f"ERROR: Found {len(violations)} license policy violations")
       exit(1)


***************
Next Steps
***************

**Advanced Integration**

- Build custom ScanCode.io pipelines for specialized workflows
- Create pre-commit hooks for local scanning before push
- Integrate with issue tracking systems (JIRA, GitHub Issues)
- Set up automated SBOM generation and distribution
- Implement continuous vulnerability monitoring

**Scaling and Performance**

- Run distributed scans for large codebases
- Cache scan results to avoid redundant scanning
- Use incremental scanning for changed files only
- Deploy ScanCode.io for enterprise-scale automation

**Custom Development**

- Extend ScanCode with custom plugins
- Create license detection rules for proprietary licenses
- Build custom output formatters and reporters
- Develop organization-specific policy enforcement tools


**Tools and Documentation**

- :doc:`../aboutcode-projects/scancode-toolkit-project` - Full CLI reference
- :doc:`../aboutcode-projects/scancodeio-project` - API and pipeline development
- :doc:`../aboutcode-projects/scancode-action-project` - GitHub Actions integration
- :doc:`../aboutcode-projects/vulnerablecode-project` - Vulnerability API
- :doc:`../contributing` - Contributing to AboutCode projects


******************
Common Questions
******************

**How long does scanning take?**

Scan time depends on codebase size. Small projects (< 1000 files) take seconds to minutes. Large codebases (100k+ files) may take hours. Use ``--max-depth`` to limit recursion or ``--ignore`` to skip directories like ``node_modules/``.

**Can I scan private/proprietary code?**

Yes, all AboutCode tools run locally or on your infrastructure. Code is not sent to external services unless you explicitly upload to a hosted instance you control.

**How do I handle binary files?**

ScanCode extracts archives and can scan inside many binary formats. For compiled binaries, focus on dependency manifests and SBOM generation during the build process.

**How do I customize license detection?**

Create custom license rules using ScanCode's license library format. See the ScanCode documentation for adding proprietary or organization-specific licenses.

**Can I integrate with Slack/Teams for alerts?**

Yes, use webhooks in your CI/CD pipeline to send notifications. Parse scan results and post messages when policy violations or vulnerabilities are detected.


******************
Getting Help
******************

- Community chat: `Gitter <https://gitter.im/aboutcode-org/discuss>`_ or `Slack <https://join.slack.com/t/aboutcode-org/shared_invite/zt-1paqwxccw-IuafuiAvYJFkTqGaZsC1og>`_
- GitHub issues: Report bugs or request features on project repositories
- API documentation: Check project-specific docs for detailed API references
- Code examples: Browse example integrations in project repositories
