.. _security_getting_started:

#######################################
Getting Started: Security Researcher
#######################################

This guide introduces you to vulnerability scanning and security analysis using AboutCode tools. Learn how to discover security issues in your software dependencies, analyze SBOMs for vulnerabilities, and implement continuous security monitoring for your applications.

************
Prerequisites
************

Before you begin, ensure you have:

- Access to your application's codebase or deployment packages
- Basic understanding of software dependencies and package managers
- Familiarity with vulnerability concepts (CVEs, security advisories)
- Access to deploy or run AboutCode security tools


******************
Quick Start Path
******************

Step 1: Install VulnerableCode
===============================

Deploy VulnerableCode to access the open vulnerability database:

.. code-block:: bash

   git clone https://github.com/aboutcode-org/vulnerablecode
   cd vulnerablecode
   # Follow installation instructions

For detailed setup, see :doc:`../aboutcode-projects/vulnerablecode-project`.

Alternatively, use a hosted instance if available to your organization.


Step 2: Scan for Dependencies
==============================

Use ScanCode Toolkit to identify packages in your application:

.. code-block:: bash

   scancode --package --json-pp packages.json /path/to/code

This detects:

- Package managers and manifest files (package.json, requirements.txt, pom.xml, etc.)
- Installed packages and their versions
- Package URLs (PURLs) for vulnerability lookup


Step 3: Check for Vulnerabilities
==================================

Query VulnerableCode API with package identifiers to find known vulnerabilities:

.. code-block:: bash

   curl "https://public.vulnerablecode.io/api/packages?purl=pkg:npm/lodash@4.17.15"

Review results for:

- CVE identifiers and vulnerability descriptions
- Severity scores (CVSS ratings)
- Affected version ranges
- Fixed versions and patching guidance


Step 4: Analyze SBOMs for Vulnerabilities
==========================================

If you have an existing SBOM, use ScanCode.io to automatically check all components:

1. Upload your SBOM (SPDX, CycloneDX) to ScanCode.io
2. Run the vulnerability matching pipeline
3. Review flagged components with known security issues

Learn more: :doc:`../getting-started/consume-sboms`


***************
Next Steps
***************

**For Continuous Security**

- Integrate vulnerability scanning into CI/CD pipelines
- Set up automated alerts for new vulnerabilities
- Implement dependency update policies and testing
- Track vulnerability remediation status

**Advanced Security Topics**

- Exploitability analysis and threat modeling
- Security policy enforcement and risk scoring
- Vulnerability disclosure and coordination
- Supply chain attack detection


**Tools to Master**

- :doc:`../aboutcode-projects/vulnerablecode-project` - Comprehensive vulnerability data
- :doc:`../aboutcode-projects/scancodeio-project` - Automated security pipelines
- :doc:`../aboutcode-projects/purldb-project` - Package universe and metadata
- :doc:`quickstart/first-vulnerability-scan` - Detailed scanning tutorial


******************
Common Questions
******************

**How current is the vulnerability data?**

VulnerableCode aggregates data from multiple authoritative sources including NVD, GitHub Security Advisories, and ecosystem-specific databases. Data is regularly updated to include new disclosures.

**What if my package isn't found?**

Package matching uses Package URLs (PURLs) for precise identification. Ensure your package identifiers include namespace, name, and version. Check PurlDB for package metadata and alternative identifiers.

**How do I prioritize vulnerabilities?**

Consider multiple factors: CVSS severity score, exploit availability, component exposure (direct vs. transitive dependency), and business impact. Focus on remotely exploitable vulnerabilities in internet-facing components first.

**Can I scan compiled binaries?**

Scanning compiled binaries is more challenging. Container image scanning works well for layered filesystems. For native binaries, focus on dependency manifests and SBOM generation during build time.


******************
Getting Help
******************

- Join the community on `Gitter <https://gitter.im/aboutcode-org/discuss>`_ or `Slack <https://join.slack.com/t/aboutcode-org/shared_invite/zt-1paqwxccw-IuafuiAvYJFkTqGaZsC1og>`_
- Report security tool issues on `GitHub <https://github.com/aboutcode-org/vulnerablecode>`_
- For responsible vulnerability disclosure, see individual project security policies
