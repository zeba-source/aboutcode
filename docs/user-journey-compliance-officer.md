# User Journey: First-Time Compliance Officer

## Persona Profile

**Name:** Sarah Chen  
**Role:** Open Source Compliance Officer  
**Background:** Legal background, familiar with license compliance concepts, new to automated scanning tools  
**Organization:** Mid-sized software company building a web application  
**Experience Level:** Beginner with software composition analysis tools

---

## 1. Goal

**Primary Objective:** Create a complete software bill of materials (SBOM) for a product that identifies all third-party components, their licenses, and any potential license compliance issues.

**Success Criteria:**
- Identify all open source components used in the product
- Document the license for each component
- Flag any license conflicts or policy violations
- Generate attribution documents for legal distribution
- Produce an SBOM in industry-standard format (SPDX or CycloneDX)

**Business Driver:** The company needs to deliver an SBOM to a major customer as part of a contract requirement, and the legal team must ensure no GPL-licensed code is included in the proprietary product.

---

## 2. Entry Point

**Discovery Path:**  
Sarah arrives at the AboutCode documentation homepage after searching for "open source license compliance tools." She sees the persona-based landing page with three options.

**First Click:**  
She clicks the "👔 Compliance Officer" card, which takes her to:  
→ **`legal/getting-started.html`**

**Initial Questions:**
- "What information can I get from these tools?"
- "Do I need to install software or can I use a web interface?"
- "How long will this take?"
- "What do I need to prepare before starting?"

**Recommended Entry Documentation:**
1. Start at: `legal/getting-started.html` (Compliance Getting Started)
2. Review: `getting-started/start-scanning-code.html` (Overview of scanning)
3. Understand: `aboutcode-projects/scancodeio-project.html` (Primary tool overview)

---

## 3. Step-by-Step Workflow

### Phase 1: Preparation (Day 1, Morning)

#### Step 1.1: Understand the Tool Landscape
**Action:** Read about which AboutCode tools are relevant for compliance work

**Tools to Learn:**
- **ScanCode.io** - Web-based platform for scanning packages and generating reports
- **DejaCode** - Product inventory management and policy enforcement (optional for first scan)
- **ScanCode Toolkit** - Command-line scanner (alternative if comfortable with terminal)

**Documentation Pages:**
- Read: `aboutcode-project-overview.html`
- Review: `aboutcode-projects/scancodeio-project.html`
- Bookmark: `legal/reference/license-categories-explained.html` (for later reference)

**Decision Point:** Sarah chooses ScanCode.io because it has a web interface and doesn't require command-line expertise.

---

#### Step 1.2: Install or Access ScanCode.io
**Action:** Set up access to ScanCode.io platform

**Options:**
- Install locally using Docker (IT department can help)
- Request access to company's existing instance (if available)
- Use a trial/evaluation instance for testing

**Documentation Pages:**
- Follow: `aboutcode-projects/scancodeio-project.html` → Installation link
- Reference: ScanCode.io ReadTheDocs installation guide

**What Sarah Does:** Works with IT to get a Docker instance running on her laptop for initial testing.

**Time Estimate:** 1-2 hours (with IT support)

---

#### Step 1.3: Gather Product Information
**Action:** Collect the software package or codebase to analyze

**What Sarah Needs:**
- The product source code repository location
- Any container images or build artifacts
- List of declared dependencies (package.json, requirements.txt, pom.xml, etc.)
- Access credentials to download the codebase

**Documentation Pages:**
- Review: `getting-started/start-scanning-code.html` (understand what can be scanned)

**Sarah's Preparation:**
- Downloads a ZIP of the latest release branch
- Gets the Docker container image from the build system
- Collects dependency manifest files

**Time Estimate:** 30 minutes

---

### Phase 2: First Scan (Day 1, Afternoon)

#### Step 2.1: Create Your First Project in ScanCode.io
**Action:** Upload codebase and initiate a scan

**Process:**
1. Open ScanCode.io web interface
2. Click "Create New Project"
3. Provide a project name: "MyProduct-v2.5-Compliance-Audit"
4. Upload the product ZIP file or provide repository URL
5. Select the pipeline: **"scan_codebase"** (for source code analysis)

**Documentation Pages:**
- Follow: `getting-started/start-scanning-code.html#scan-software-using-scancodeio`
- Reference: ScanCode.io tutorial for creating projects

**What Happens:** ScanCode.io processes the upload, extracts files, and begins scanning for:
- Licenses in source files
- Copyright statements
- Package manifests and dependencies

**Time Estimate:** 5 minutes to set up, 15-45 minutes for scan to complete (depending on codebase size)

---

#### Step 2.2: Review Initial Scan Results
**Action:** Understand what was found in the initial scan

**How to Navigate Results:**
1. Open the completed project in ScanCode.io
2. Review the "Packages" tab - shows detected third-party components
3. Check the "Resources" tab - shows individual files and their detected licenses
4. Look at the "Summary" section - provides overview statistics

**What Sarah Sees:**
- 247 packages detected
- 15 different licenses found
- 3 packages flagged with "AGPL-3.0" (potential policy issue!)
- Some files showing "Unknown" or "No license detected"

**Documentation Pages:**
- Review: `getting-started/start-scanning-code.html#review-scan-results`
- Reference: `legal/reference/license-categories-explained.html` (to understand license types)

**Time Estimate:** 30 minutes to explore

---

### Phase 3: Analysis and Investigation (Day 2)

#### Step 3.1: Investigate License Policy Violations
**Action:** Examine the AGPL-licensed components that violate company policy

**Investigation Process:**
1. Click on each AGPL package to see details
2. Check if these are direct dependencies or transitive (dependencies of dependencies)
3. Determine if they're actually included in the distributed product
4. Look for alternative packages with more permissive licenses

**Questions Sarah Asks:**
- "Is this package really needed in production, or is it a development tool?"
- "Are we linking to this library or just using it during the build?"
- "Can we replace this with an Apache or MIT-licensed alternative?"

**Documentation Pages:**
- Reference: `legal/guides/handle-license-conflicts.html` (hypothetical - needs to be created)
- Learn: `legal/reference/common-license-scenarios.html` (hypothetical - for GPL scenarios)

**Sarah's Finding:** Two AGPL packages are development-only test dependencies that aren't shipped. One is a transitive dependency that needs engineering review.

**Time Estimate:** 2-3 hours

---

#### Step 3.2: Handle Unclear or Missing License Information
**Action:** Resolve packages showing "Unknown" or ambiguous licenses

**Common Scenarios:**
- Package has license file but wasn't detected
- Package has multiple licenses (dual licensing)
- Package has license in README but not in standard location
- License detection confidence is below threshold

**Resolution Steps:**
1. Manually review the package source code
2. Check the package's upstream repository or package registry
3. Document findings in notes
4. May need to contact package maintainers
5. Update scan results with manual assertions (in DejaCode, if using it)

**Documentation Pages:**
- Reference: `legal/guides/understand-license-obligations.html` (hypothetical)
- Review: License detection confidence scores in scan results

**Sarah's Action:** Reviews 12 unclear packages, documents findings in a spreadsheet for now, plans to use DejaCode for formal tracking later.

**Time Estimate:** 1-2 hours

---

### Phase 4: Policy Configuration (Day 3)

#### Step 4.1: Set Up License Policies (Optional but Recommended)
**Action:** Configure which licenses are approved, restricted, or prohibited

**Tool Choice:** DejaCode for enterprise policy management

**Policy Categories Set Up:**
- **Approved:** MIT, Apache-2.0, BSD-3-Clause, BSD-2-Clause
- **Restricted (Review Required):** LGPL-2.1, LGPL-3.0, MPL-2.0
- **Prohibited:** GPL-2.0, GPL-3.0, AGPL-3.0 (for proprietary distribution)

**Documentation Pages:**
- Follow: `getting-started/manage-license-policies.html`
- Setup: DejaCode ReadTheDocs for policy configuration
- Export: Policy file for use in future scans

**Benefits:**
- Future scans automatically flag policy violations
- Consistent policy enforcement across teams
- Audit trail of decisions

**Time Estimate:** 1-2 hours

---

#### Step 4.2: Configure Integration Between Tools
**Action:** Connect ScanCode.io with DejaCode for enhanced workflow

**Integration Benefits:**
- Import scan results directly into DejaCode products
- Track components across multiple product versions
- Generate attribution documents automatically
- Maintain historical compliance records

**Documentation Pages:**
- Follow: `getting-started/create-sboms.html#import-scan-results-to-dejacode`
- Configure: DejaCode integration settings

**Sarah's Setup:** Links her ScanCode.io instance to DejaCode, creates a "Product" for her application.

**Time Estimate:** 30 minutes

---

### Phase 5: Deliverables Generation (Day 3-4)

#### Step 5.1: Generate the SBOM
**Action:** Export scan results as standards-compliant SBOM

**SBOM Format Options:**
- **SPDX** (Software Package Data Exchange) - ISO standard, legal focus
- **CycloneDX** - OWASP standard, security focus, includes vulnerability data

**Export Process in ScanCode.io:**
1. Open the completed project
2. Navigate to "Output Files" section
3. Download SPDX or CycloneDX file
4. Validate the SBOM contains all required components

**Documentation Pages:**
- Reference: `getting-started/create-sboms.html`
- Learn: SBOM format differences and when to use each

**What Sarah Gets:**
- `myproduct-v2.5-sbom.spdx.json` - Complete component inventory
- Machine-readable format for customer systems
- Human-readable HTML report for internal review

**Time Estimate:** 15 minutes

---

#### Step 5.2: Create Attribution Documents
**Action:** Generate legal notices and attribution text for distribution

**Required Outputs:**
- NOTICE file with all copyright notices
- LICENSE file with full license texts
- Attribution report for documentation

**Tools Used:**
- **DejaCode** - Primary attribution generation
- **AboutCode Toolkit** - Alternative command-line option

**Documentation Pages:**
- Follow: `getting-started/create-sboms.html#generate-attribution-and-sboms`
- Reference: `aboutcode-projects/aboutcode-toolkit-project.html` (alternative method)

**What Sarah Creates:**
- NOTICE.txt - All copyright statements (12 pages)
- LICENSES/ directory - Full license texts for all detected licenses
- Attribution.html - Formatted attribution report for product documentation

**Time Estimate:** 30 minutes

---

#### Step 5.3: Create Executive Summary Report
**Action:** Prepare findings summary for legal and engineering teams

**Report Contents:**
1. **Overview:** 247 components analyzed, 15 unique licenses
2. **Policy Compliance:** 3 violations found, 2 resolved, 1 requires engineering action
3. **Risk Assessment:** Low risk - no copyleft licenses in distributed code
4. **Action Items:** 
   - Replace one AGPL library with alternative
   - Document 5 components with unclear licenses
   - Set up automated scanning for future releases
5. **Deliverables:** SBOM file and attribution documents attached

**Documentation Pages:**
- Template: `legal/guides/create-compliance-artifacts.html` (hypothetical)

**Time Estimate:** 1 hour

---

## 4. Expected Outputs

### Primary Deliverables

#### 4.1 Software Bill of Materials (SBOM)
**File:** `myproduct-v2.5-sbom.spdx.json`

**Contents:**
- Complete list of 247 components
- Package name, version, and Package URL (PURL) for each
- Declared license for each component
- License expressions for multi-licensed components
- Supplier information where available
- File hashes for verification

**Use Cases:**
- Submit to customer as contractual requirement
- Share with security team for vulnerability scanning
- Archive for compliance records
- Use as baseline for next release comparison

---

#### 4.2 Attribution Documentation
**Files:** 
- `NOTICE.txt` - Copyright notices
- `LICENSES/` directory - Full license texts
- `Attribution.html` - Formatted report

**Contents:**
- Copyright statements from all components
- Full text of all applicable licenses
- Attribution requirements per license
- Source code availability information (if required)

**Use Cases:**
- Include in product distribution (NOTICE file)
- Add to product documentation (Attribution report)
- Provide to customers upon request
- Satisfy license requirements

---

#### 4.3 Compliance Analysis Report
**File:** `Compliance-Analysis-MyProduct-v2.5.pdf`

**Sections:**
1. Executive Summary
2. Methodology (tools used, scope of analysis)
3. Findings
   - Components by license type
   - Policy violations and resolutions
   - Unknown or unclear licenses
4. Risk Assessment
5. Recommendations
6. Appendices (detailed component lists)

**Audience:** Legal counsel, engineering leadership, customers

---

#### 4.4 Component Tracking Database
**System:** DejaCode Product Inventory

**Contents:**
- All 247 components tracked as "Product Components"
- License conclusions documented
- Usage policies assigned
- Review status for each component
- Historical tracking across versions

**Benefits:**
- Searchable inventory for future questions
- Track component updates and new versions
- Audit trail for compliance decisions
- Reusable for next release (update scan vs. full rescan)

---

### Secondary Outputs

#### 4.5 Policy Configuration Files
**Files:**
- `license-policies.yml` - Exportable policy definitions
- Used for automated scanning in CI/CD pipeline

#### 4.6 Action Items List
**Tracking:**
- 1 component to replace (AGPL dependency)
- 5 components needing manual license clarification
- Engineering tasks assigned with priorities

---

## 5. Common Pitfalls

### Pitfall 1: Incomplete Codebase Scanning
**Problem:** Only scanning the source code repository, missing compiled dependencies

**What Gets Missed:**
- Binary dependencies from package managers (npm, Maven, pip)
- Third-party libraries bundled in build artifacts
- Container base images and their components
- Vendored dependencies in third-party directories

**Solution:**
- Scan both source code AND build artifacts
- Use appropriate pipelines: "scan_codebase" + "scan_package" or "docker_image"
- Review package manifests to ensure all dependencies were detected
- Compare scan results with declared dependencies

**Documentation:** Review pipeline options in ScanCode.io tutorials

---

### Pitfall 2: Trusting Declared Licenses Without Verification
**Problem:** Assuming package metadata is accurate without checking actual files

**Risk:**
- Package.json says "MIT" but actual code has "GPL" headers
- License changed between versions but metadata not updated
- Dual-licensed projects where metadata shows only one option

**Solution:**
- Review file-level scan results, not just package-level
- Check confidence scores on license detections
- Manually verify critical or high-risk components
- Document discrepancies between declared and detected licenses

**Documentation:** `legal/guides/understand-license-obligations.html`

---

### Pitfall 3: Ignoring Transitive Dependencies
**Problem:** Only reviewing direct dependencies, missing "dependencies of dependencies"

**Example:**
- You use Package A (MIT licensed)
- Package A depends on Package B (GPL licensed)
- Your product now has GPL code even though you didn't explicitly include it

**Solution:**
- Ensure full dependency tree scanning is enabled
- Review the dependency graph in scan results
- Understand which dependencies are runtime vs. build-time
- Use policy enforcement to catch transitive violations

**Documentation:** Review dependency analysis features

---

### Pitfall 4: Confusion About License Obligations
**Problem:** Not understanding what different licenses actually require

**Common Misunderstandings:**
- "MIT license means no attribution needed" (FALSE - attribution required)
- "Using LGPL is the same as GPL" (FALSE - different linking rules)
- "Internal use has no license requirements" (PARTIAL - some licenses have internal restrictions)

**Solution:**
- Read: `legal/reference/license-categories-explained.html`
- Consult: `legal/reference/common-license-scenarios.html`
- When in doubt: Seek legal counsel for license interpretation
- Document your interpretations and decisions

**Key Learning:** Licenses have different requirements for attribution, source disclosure, and distribution.

---

### Pitfall 5: One-Time Compliance Check
**Problem:** Treating compliance as a single audit instead of ongoing process

**Reality:**
- Dependencies get updated regularly
- New vulnerabilities are discovered
- Code changes between releases
- License policies may evolve

**Solution:**
- Set up automated scanning in CI/CD pipeline
- Re-scan before each release
- Track component changes between versions
- Monitor for license changes in updated dependencies
- Subscribe to security bulletins for your components

**Documentation:** `developer/guides/ci-cd-integration-patterns.html`

---

### Pitfall 6: Poor Documentation of Decisions
**Problem:** Not recording why certain license decisions were made

**Impact:**
- Can't explain decisions in future audits
- Different team members make inconsistent decisions
- Lost knowledge when team members leave
- Difficult to respond to customer questions

**Solution:**
- Use DejaCode to document license conclusions and rationale
- Maintain a decisions log or wiki
- Tag components with review status and notes
- Create templates for common decision scenarios

---

## 6. Next Steps

### Immediate Actions (This Week)

#### 6.1 Share Results with Stakeholders
**Action Items:**
- Present compliance report to legal counsel for review
- Share AGPL finding with engineering team for remediation
- Deliver SBOM to customer (if deadline driven)
- Schedule follow-up meeting to discuss ongoing process

**Preparation:** Create a presentation summarizing findings and recommendations

---

#### 6.2 Address Policy Violations
**Action Items:**
- Work with engineering to replace AGPL dependency
- Get legal sign-off on any waiver decisions
- Document risk acceptance for any remaining issues
- Set deadline for remediation

**Timeline:** Target resolution within 2 weeks

---

#### 6.3 Complete Documentation
**Action Items:**
- Finalize manual license determinations
- Archive all compliance artifacts in shared repository
- Update product documentation with attribution information
- Create compliance folder for this product version

---

### Short-Term Goals (This Month)

#### 6.4 Implement DejaCode Product Tracking
**Why:** Move from ad-hoc spreadsheets to formal tracking system

**Benefits:**
- Track components across multiple products
- Maintain history of license decisions
- Reuse component analysis for other products
- Generate reports more easily

**Documentation:** `getting-started/create-sboms.html#import-scan-results-to-dejacode`

**Time Investment:** 4-8 hours for initial setup and data import

---

#### 6.5 Create Compliance Process Documentation
**Deliverable:** Internal wiki or document describing the compliance workflow

**Contents:**
- When to perform compliance scans (before each release)
- Who is responsible for each step
- How to use ScanCode.io and DejaCode
- Escalation process for issues
- Templates for compliance reports

**Audience:** Future compliance team members, engineering managers, legal counsel

---

#### 6.6 Train Engineering Team
**Goal:** Help developers understand compliance requirements

**Topics:**
- How to check licenses before adding dependencies
- Understanding the license policy
- How to interpret scan results
- When to escalate to compliance team

**Format:** 1-hour workshop or recorded training video

---

### Long-Term Strategy (Next Quarter)

#### 6.7 Automate Compliance Scanning in CI/CD
**Goal:** Catch compliance issues early in development

**Implementation:**
- Set up ScanCode.io API integration in CI/CD pipeline
- Configure automated scans on every pull request or release branch
- Set policy thresholds (block builds with prohibited licenses)
- Send alerts to compliance team for review items

**Documentation:** 
- `developer/guides/ci-cd-integration-patterns.html`
- `aboutcode-projects/scancode-action-project.html` (for GitHub Actions)

**Benefits:**
- Prevent prohibited licenses from merging
- Reduce compliance review time
- Create audit trail automatically
- Shift compliance left in development

---

#### 6.8 Expand to Vulnerability Scanning
**Goal:** Add security analysis alongside license compliance

**Tools to Integrate:**
- **VulnerableCode** - Open source vulnerability database
- **ScanCode.io** - Already scans for vulnerabilities if configured

**Process:**
- Enable vulnerability scanning in ScanCode.io pipelines
- Generate VEX (Vulnerability Exploitability eXchange) documents
- Track CVEs alongside license information in SBOMs
- Coordinate with security team on remediation

**Documentation:**
- `security/getting-started.html`
- `security/guides/triage-vulnerabilities.html` (hypothetical)

---

#### 6.9 Establish Metrics and KPIs
**Goal:** Measure and improve compliance program effectiveness

**Metrics to Track:**
- Time from scan to SBOM delivery
- Number of policy violations per release
- Percentage of components with clear license information
- Time to resolve compliance issues
- Number of manual interventions required

**Review Frequency:** Monthly or per-release

---

#### 6.10 Build a Component Repository
**Goal:** Create pre-approved component catalog

**Benefits:**
- Developers can choose from pre-vetted components
- Reduce duplicate license reviews
- Faster approval for common packages
- Consistent component usage across products

**Implementation:**
- Use DejaCode to maintain approved component list
- Document license obligations for each
- Publish catalog to engineering team
- Update quarterly

---

### Continuous Improvement

#### 6.11 Stay Current with AboutCode Updates
**Activities:**
- Subscribe to AboutCode project announcements
- Join community Gitter/Slack channels
- Review release notes for new features
- Attend AboutCode webinars or conferences

**Community Resources:**
- Gitter: https://gitter.im/aboutcode-org/discuss
- GitHub: https://github.com/aboutcode-org/

---

#### 6.12 Contribute Back to Community
**Opportunities:**
- Report bugs or unclear documentation
- Suggest new license detection rules
- Share compliance workflow patterns
- Contribute to license database improvements

**Impact:** Improve tools for entire community while strengthening your implementation

---

## Journey Complete!

Congratulations! Sarah has successfully:

✅ Completed her first compliance scan  
✅ Generated an SBOM and attribution documents  
✅ Identified and addressed license policy issues  
✅ Established a repeatable compliance process  
✅ Created a foundation for automated compliance

**From first scan to mature compliance program:** Sarah's journey transforms from a one-time audit to an integrated, automated compliance operation that protects her organization while enabling efficient software development.

**Time Investment Summary:**
- Initial scan and analysis: 3-4 days
- Process setup and documentation: 1 week
- Automation and integration: 2-4 weeks
- Mature program operation: Ongoing, but streamlined

**Return on Investment:**
- Faster time to delivery for customer SBOMs
- Reduced legal risk from license violations
- Improved security posture
- Auditable compliance trail
- Competitive advantage in enterprise sales

---

## Additional Resources

### Documentation Pages Referenced
- `legal/getting-started.html` - Compliance officer entry point
- `getting-started/start-scanning-code.html` - Scanning basics
- `getting-started/create-sboms.html` - SBOM generation workflow
- `getting-started/manage-license-policies.html` - Policy configuration
- `aboutcode-project-overview.html` - All AboutCode projects
- `aboutcode-projects/scancodeio-project.html` - ScanCode.io details
- `aboutcode-projects/dejacode-project.html` - DejaCode details

### External Resources
- ScanCode.io ReadTheDocs (installation and tutorials)
- DejaCode ReadTheDocs (product management)
- SPDX Specification
- CycloneDX Specification
- Open Source License texts and interpretations

### Support Channels
- Gitter: https://gitter.im/aboutcode-org/discuss
- Slack: AboutCode community workspace
- GitHub Issues: Project-specific repositories
- Email: Project mailing lists

---

*This user journey is designed to be updated as AboutCode tools evolve and new features are added. Last updated: February 2026*
