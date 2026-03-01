.. _aboutcode_home:

#########
AboutCode
#########

Welcome to AboutCode! We provide free and open source tools to help you understand
what's in your software: where it comes from, what licenses apply, and whether it
has known security issues. Whether you're managing legal compliance, securing your
supply chain, or building software, AboutCode has tools to help.


.. _who_are_you:

*************
Who are you?
*************

Choose your path to get started quickly with the tools and guides most relevant to you.

.. grid:: 1 2 3 3
    :gutter: 3
    :padding: 2

    .. grid-item-card:: 👔 Compliance Officer
        :link: legal/getting-started
        :link-type: doc
        :class-card: sd-text-center sd-font-weight-bold
        :class-title: sd-fs-5
        :shadow: md

        Manage open source licenses, create SBOMs, and ensure your organization
        meets its legal obligations.

        **Get Started →**

        +++
        License scanning • Policy enforcement • Attribution • SBOM generation

    .. grid-item-card:: 🔒 Security Researcher
        :link: security/getting-started
        :link-type: doc
        :class-card: sd-text-center sd-font-weight-bold
        :class-title: sd-fs-5
        :shadow: md

        Find vulnerabilities in your dependencies, analyze software composition,
        and secure your supply chain.

        **Get Started →**

        +++
        Vulnerability scanning • SBOM analysis • Dependency tracking • Risk assessment

    .. grid-item-card:: 💻 Developer/Integrator
        :link: developer/getting-started
        :link-type: doc
        :class-card: sd-text-center sd-font-weight-bold
        :class-title: sd-fs-5
        :shadow: md

        Integrate AboutCode tools into your build pipeline, automate scans,
        and use our APIs.

        **Get Started →**

        +++
        CLI tools • CI/CD integration • REST APIs • Custom pipelines


----

.. _quick_links:

************
Quick Links
************

Not sure where to start? Here are some popular tasks:

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item-card:: 📋 Create an SBOM
        :link: getting-started/create-sboms
        :link-type: doc

        Generate Software Bill of Materials in SPDX or CycloneDX format

    .. grid-item-card:: 🔍 Scan for Licenses
        :link: getting-started/start-scanning-code
        :link-type: doc

        Identify licenses and copyrights in your codebase

    .. grid-item-card:: 🛡️ Find Vulnerabilities
        :link: security/quickstart/first-vulnerability-scan
        :link-type: doc

        Discover known security issues in your dependencies

    .. grid-item-card:: ⚖️ Set License Policies
        :link: getting-started/manage-license-policies
        :link-type: doc

        Define which licenses are approved for your organization


----

.. _aboutcode_projects:

******************
AboutCode Projects
******************

AboutCode is a family of tools that work together:

.. grid:: 1 1 2 2
    :gutter: 3

    .. grid-item-card:: ScanCode.io
        :link: aboutcode-projects/scancodeio-project
        :link-type: doc
        :img-top: _static/images/scancodeio-icon.svg
        :img-alt: ScanCode.io

        Automated pipeline platform for scanning packages, containers, and codebases
        at scale.

    .. grid-item-card:: ScanCode Toolkit
        :link: aboutcode-projects/scancode-toolkit-project
        :link-type: doc
        :img-top: _static/images/scancode-icon.svg
        :img-alt: ScanCode Toolkit

        Command-line tool to detect licenses, copyrights, and dependencies in your code.

    .. grid-item-card:: VulnerableCode
        :link: aboutcode-projects/vulnerablecode-project
        :link-type: doc
        :img-top: _static/images/vulnerablecode-icon.svg
        :img-alt: VulnerableCode

        Open database of software vulnerabilities with tools to track and correlate CVEs.

    .. grid-item-card:: DejaCode
        :link: aboutcode-projects/dejacode-project
        :link-type: doc
        :img-top: _static/images/dejacode-icon.svg
        :img-alt: DejaCode

        Enterprise compliance platform for managing software inventories and policies.

    .. grid-item-card:: PurlDB
        :link: aboutcode-projects/purldb-project
        :link-type: doc
        :img-top: _static/images/purldb-icon.svg
        :img-alt: PurlDB

        Database of Package URLs (PURLs) with package metadata and matching capabilities.

    .. grid-item-card:: ScanCode Workbench
        :link: aboutcode-projects/scancode-workbench-project
        :link-type: doc
        :img-top: _static/images/workbench-icon.svg
        :img-alt: ScanCode Workbench

        Desktop application to review scan results and document your conclusions.

.. button-link:: aboutcode-project-overview.html
    :color: primary
    :outline:

    View All Projects


----

.. _explore_documentation:

*********************
Explore Documentation
*********************

.. toctree::
   :maxdepth: 1
   :caption: By Role

   legal/index
   security/index
   developer/index

.. toctree::
   :maxdepth: 2
   :caption: Getting Started

   getting-started/start-scanning-code
   getting-started/create-sboms
   getting-started/consume-sboms
   getting-started/manage-license-policies
   getting-started/cra-compliance

.. toctree::
   :maxdepth: 2
   :caption: All Projects

   aboutcode-project-overview
   aboutcode-projects/scancodeio-project
   aboutcode-projects/scancode-toolkit-project
   aboutcode-projects/vulnerablecode-project
   aboutcode-projects/dejacode-project
   aboutcode-projects/purldb-project
   aboutcode-projects/scancode-workbench-project
   aboutcode-projects/license-expression-project
   aboutcode-projects/scancode-licensedb-project
   aboutcode-projects/source-inspector-project
   aboutcode-projects/python-inspector-project
   aboutcode-projects/scancode-action-project
   aboutcode-projects/aboutcode-toolkit-project

.. toctree::
   :maxdepth: 2
   :caption: Data & Standards

   aboutcode-data/abcd

.. toctree::
   :maxdepth: 2
   :caption: Contributing

   contributing

.. toctree::
   :maxdepth: 1
   :caption: Community & Archive

   archive
   license


----

.. _need_help:

*********
Need Help?
*********

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item::
        :columns: 12 6 6 6

        **💬 Chat with Us**

        Join our community on `Gitter <https://gitter.im/aboutcode-org/discuss>`_
        or `Slack <https://join.slack.com/t/aboutcode-org/shared_invite/zt-1paqwxccw-IuafuiAvYJFkTqGaZsC1og>`_

    .. grid-item::
        :columns: 12 6 6 6

        **🐛 Report Issues**

        Found a bug? `Open an issue <https://github.com/aboutcode-org/>`_ on
        the relevant project repository

    .. grid-item::
        :columns: 12 6 6 6

        **📚 Browse Code**

        Explore our projects at `github.com/aboutcode-org <https://github.com/aboutcode-org/>`_

    .. grid-item::
        :columns: 12 6 6 6

        **🎓 Learn More**

        Visit `AboutCode.org <https://www.aboutcode.org/>`_ for news, events,
        and community resources


.. note::
   **New to Software Composition Analysis?**

   Software Composition Analysis (SCA) helps you understand what open source
   components are in your software, their licenses, and any security vulnerabilities.
   Think of it as creating a detailed ingredient list for your software, so you
   can make informed decisions about what you use and how you use it.
