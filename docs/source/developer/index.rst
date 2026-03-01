.. _developer_index:

########################################
Developer & Integrator Documentation
########################################

Welcome to the Developer and Integrator's guide for AboutCode tools. This section helps you integrate license scanning and vulnerability detection into your build pipelines, automate compliance workflows, and use AboutCode APIs programmatically. Whether you're building CI/CD integrations, custom analysis tools, or automated reporting systems, you'll find the technical documentation and examples you need here.

.. toctree::
   :maxdepth: 2
   :caption: Developer Guides

   getting-started

.. toctree::
   :maxdepth: 1
   :caption: Quick Access

   ../getting-started/start-scanning-code
   ../getting-started/create-sboms


----

*******************
Integration Guides
*******************

**CI/CD Integration**

Integrate AboutCode tools into GitHub Actions, GitLab CI, Jenkins, Azure Pipelines, and other CI/CD platforms. Automate license scanning and vulnerability detection as part of your build and deployment process.

**REST APIs**

Use AboutCode REST APIs for programmatic access to scanning results, vulnerability data, and package information. Build custom dashboards, reporting tools, and integration workflows.

**Command-Line Tools**

Master the command-line interfaces for ScanCode Toolkit, VulnerableCode CLI, and other AboutCode tools. Script automated workflows and batch processing tasks.


----

**********************
Development Workflows
**********************

**Automated Scanning**

Set up pre-commit hooks, pull request checks, and scheduled scans. Fail builds when license policy violations or critical vulnerabilities are detected.

**Custom Pipelines**

Build custom analysis pipelines using ScanCode.io's plugin architecture. Extend functionality with your own scanning rules, data enrichment, and output formats.

**Data Integration**

Export scan results to SIEM systems, issue trackers (JIRA, GitHub Issues), vulnerability management platforms, and compliance reporting tools.


----

************************
Code Examples & SDKs
************************

**Python Integration**

.. code-block:: python

   from scancode import cli
   
   # Programmatic scanning
   results = cli.run_scan(
       input_path='/path/to/code',
       license=True,
       copyright=True,
       package=True
   )

**GitHub Actions**

.. code-block:: yaml

   - name: ScanCode Scan
     uses: aboutcode-org/scancode-action@v1
     with:
       path: '.'
       output: 'scancode-results.json'

**REST API Client**

.. code-block:: python

   import requests
   
   # Query VulnerableCode
   response = requests.get(
       'https://public.vulnerablecode.io/api/packages',
       params={'purl': 'pkg:npm/lodash@4.17.15'}
   )
   vulnerabilities = response.json()


----

*******************
Related Resources
*******************

- :doc:`../aboutcode-projects/scancode-toolkit-project` - CLI documentation and API
- :doc:`../aboutcode-projects/scancodeio-project` - REST API and pipeline development
- :doc:`../aboutcode-projects/scancode-action-project` - GitHub Actions integration
- :doc:`../aboutcode-projects/vulnerablecode-project` - Vulnerability API reference
- :doc:`../aboutcode-projects/purldb-project` - Package metadata API


----

**********************
Contributing to Tools
**********************

AboutCode tools are open source and welcome contributions. Whether you're fixing bugs, adding features, improving documentation, or creating plugins, check out:

- :doc:`../contributing` - General contribution guidelines
- Individual project GitHub repositories for issue tracking
- Community channels for discussion and support
