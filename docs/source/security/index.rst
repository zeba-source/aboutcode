.. _security_index:

####################################
Security Researcher Documentation
####################################

Welcome to the Security Researcher's guide for AboutCode tools. This section helps you discover vulnerabilities in software dependencies, analyze software composition for security risks, and secure your software supply chain. Use VulnerableCode, ScanCode.io, and PurlDB to identify, track, and remediate security vulnerabilities in your applications and infrastructure.

.. toctree::
   :maxdepth: 2
   :caption: Security Guides

   getting-started
   quickstart/first-vulnerability-scan

.. toctree::
   :maxdepth: 1
   :caption: Quick Access

   ../getting-started/create-sboms
   ../getting-started/consume-sboms


----

*******************
Key Tasks & Guides
*******************

**Vulnerability Scanning**

Use VulnerableCode to identify known security vulnerabilities (CVEs, security advisories) in your software dependencies. Correlate package identifiers with comprehensive vulnerability databases for accurate risk assessment.

**SBOM Analysis**

Analyze Software Bill of Materials to understand your dependency tree and identify vulnerable components. Cross-reference SBOMs with VulnerableCode to prioritize remediation efforts based on actual exposure.

**Supply Chain Security**

Track dependencies across your software supply chain using Package URLs (PURLs) and PurlDB. Monitor for new vulnerabilities in components you depend on and receive alerts when issues are disclosed.


----

**********************
Security Workflows
**********************

**Continuous Monitoring**

Set up automated vulnerability scanning in your CI/CD pipeline using ScanCode.io. Integrate with VulnerableCode APIs to check for newly disclosed vulnerabilities in your dependencies.

**Risk Assessment**

Evaluate vulnerability severity, exploitability, and business impact. Use CVSS scores and exploit availability data to prioritize patching and mitigation efforts.

**Remediation Tracking**

Document remediation steps, track patching progress, and maintain security audit trails. Generate security reports for compliance and stakeholder communication.


----

*******************
Related Resources
*******************

- :doc:`../aboutcode-projects/vulnerablecode-project` - Vulnerability database and API
- :doc:`../aboutcode-projects/scancodeio-project` - Automated security scanning
- :doc:`../aboutcode-projects/purldb-project` - Package metadata and matching
- :doc:`../aboutcode-projects/scancode-toolkit-project` - Dependency detection
