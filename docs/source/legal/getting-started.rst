.. _legal_getting_started:

####################################
Getting Started: Compliance Officer
####################################

This guide walks you through the essential steps to begin using AboutCode tools for license compliance and SBOM management. You'll learn how to perform your first license scan, understand the results, and integrate compliance workflows into your organization's processes.

************
Prerequisites
************

Before you begin, ensure you have:

- Access to your organization's codebase or software packages
- Basic familiarity with command-line tools or web interfaces
- Understanding of your organization's license policy requirements
- Permission to scan and analyze software components


******************
Quick Start Path
******************

Step 1: Install ScanCode Toolkit
=================================

Start with ScanCode Toolkit for command-line license scanning:

.. code-block:: bash

   pip install scancode-toolkit

For detailed installation instructions, see :doc:`../aboutcode-projects/scancode-toolkit-project`.


Step 2: Run Your First License Scan
====================================

Scan a codebase to detect licenses, copyrights, and package dependencies:

.. code-block:: bash

   scancode --license --copyright --info --json-pp results.json /path/to/code

Review the JSON output to understand:

- Detected licenses and their exact locations
- Copyright statements and holders
- Package metadata and dependencies


Step 3: Generate an SBOM
=========================

Create a Software Bill of Materials for compliance documentation:

.. code-block:: bash

   scancode --spdx results.spdx /path/to/code

Learn more about SBOM generation: :doc:`../getting-started/create-sboms`


Step 4: Set Up License Policies
================================

Define which licenses are approved, flagged, or prohibited in your organization. Use DejaCode for enterprise policy management or create simple policy files for automated scanning.

See: :doc:`../getting-started/manage-license-policies`


***************
Next Steps
***************

**For Ongoing Compliance**

- Set up automated scanning in your CI/CD pipeline
- Configure ScanCode.io for scheduled package scans
- Integrate with issue tracking systems for license review workflows

**Advanced Topics**

- Creating custom license detection rules
- Handling multi-license scenarios
- Managing license obligations and attributions
- Audit trails and compliance reporting


**Tools to Explore**

- :doc:`../aboutcode-projects/scancodeio-project` - Automated pipeline platform
- :doc:`../aboutcode-projects/dejacode-project` - Enterprise compliance management
- :doc:`../aboutcode-projects/scancode-workbench-project` - Visual review interface


******************
Common Questions
******************

**How accurate is license detection?**

ScanCode provides highly accurate license detection based on full text matching. Review flagged results where confidence is low or licenses are unusual.

**How do I handle false positives?**

Use ScanCode Workbench to review and document your conclusions. Mark false positives and add notes for future reference and audit trails.

**Can I scan container images?**

Yes! ScanCode.io can scan Docker container images, extracted filesystems, and package archives. See the ScanCode.io documentation for container scanning workflows.


******************
Getting Help
******************

- Join the community on `Gitter <https://gitter.im/aboutcode-org/discuss>`_ or `Slack <https://join.slack.com/t/aboutcode-org/shared_invite/zt-1paqwxccw-IuafuiAvYJFkTqGaZsC1og>`_
- Report issues on `GitHub <https://github.com/aboutcode-org/>`_
- Check project-specific documentation for detailed guidance
