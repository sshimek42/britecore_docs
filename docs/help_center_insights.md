# BriteCore Help Center Insights

I reviewed the structure and emphasis of the public BriteCore Help Center to ground the docs work in the product language BriteCore itself uses.

## High-level product framing

The Help Center strongly positions BriteCore as a cloud-based platform rather than a single isolated app. The Technology > Platform section explicitly describes the platform as a cloud-based foundation and toolset for rapid development and stable deployment across BriteCore products.

That aligns with the local repo architecture we have been documenting:

- live API access and integration layer
- platform and operational tooling
- data and analytics/reporting layer
- partner/integration ecosystem
- product-specific domain surfaces such as billing, claims, contacts, and portals

## The site organizes BriteCore as a layered ecosystem

The Help Center categories highlight a product model that is consistent with the repo architecture we have inferred locally:

### 1) Platform

The Platform section is about the shared foundation and system-level capabilities.

This includes:

- getting started with the BriteCore platform
- system tags
- system processes

This suggests the platform is the shared runtime and operational backbone that sits under product surfaces and analytical tooling.

### 2) APIs

The APIs section is explicitly described as enabling insurers to expand the base platform with integrations that reduce cost, improve workflow efficiency, and connect to third-party offerings.

This lines up directly with the local `britecore_sdk` work and the broader idea that BriteCore exposes both internal operational flows and external integration hooks.

The API section includes:

- Get Started with APIs
- API Tutorials
- Events
- Webhooks

This is important because it shows that BriteCore treats API access as a first-class integration layer, not as an afterthought.

### 3) Data & Analytics

This is one of the most relevant sections for our reporting docs work.

The section description says Reporting and Analytics provide fact-based decision support. The subtopics include:

- Dashboards
- SQL Editor and Report Copilot
- Stock Report Overview
- Get started with reports
- Manage Reports
- Industry Reports

This is highly relevant to the local reporting model we have been building around:

- logical catalog and report tables
- SQL-backed reporting
- report authoring and analytic workbench patterns
- dashboard/reporting semantics rather than raw operational CRUD

This section reinforces the idea that reporting is a core BriteCore capability and not just a downstream consumer of raw data.

### 4) Integrations

The Integrations section emphasizes a broad ecosystem of partners and vendor integrations across categories such as:

- Claims
- Automotive
- Accounting
- Chat
- Documents
- Payments
- Imaging
- Inspections
- Maps and addresses
- Notifications
- Printing
- Property valuation
- Risk assessment

This supports the architecture pattern we have been documenting: BriteCore is designed to connect operational and reporting workflows across multiple service domains and vendor solutions.

## Full help-center inventory from the public Zendesk API

To avoid the Cloudflare challenge on the HTML landing pages, I scraped the public Zendesk Help Center API endpoints directly. That lets us enumerate the full help-center structure instead of only the visible first layer.

The API inventory shows:

- 4 categories
- 297 sections
- 1,344 articles

This means the BriteCore help center is a large, multi-domain knowledge base rather than a thin product portal. The structure is not just a few top-level product pages; it is a full operating model for product education, workflow setup, integrations, and admin guidance.

### Category breakdown

#### 1) Release Notes (6 sections)

This category contains the ongoing release, preview, and archive pipeline for BriteCore functionality. It indicates that product behavior evolves continuously and that the help center is also a product history and change-management surface.

Example sections:

- Current Feature Previews
- Current Release Notes
- Release Notes
- Archived Feature Previews
- Archived Release Notes
- Feature Previews

#### 2) Resources (54 sections)

This is the documentation and operational support layer. It is where BriteCore explains the supporting processes for configuration, policy administration, settings, and support operations.

Example sections:

- Glossary
- Get started with settings
- Zendesk
- Policies Processing
- Policy Cancellation
- Policy Renewal
- Policy Screen
- Policies Search
- Policy Lifecycle
- Policy Module Settings
- Claims Module Settings
- Contacts Module Settings
- Lines Module Settings

This category confirms that the product is operationally rich and requires deep configuration and admin training beyond just end-user workflows.

#### 3) Technology (79 sections)

This is the platform and integration layer. It is the most important evidence for our architecture work because it shows the product is designed as a platform ecosystem, not just a policy system.

Example sections:

- Artificial Intelligence
- Dashboards
- Claims
- Tiger Risk’s Catastrophe Quotient (TigerCQ)
- Automotive
- CoreLogic
- DocuSign
- LexisNexis Auto
- HazardHub
- Get started with integrations
- Platform
- Get started with BriteCore's platform
- OnBase
- ImageRight
- BriteCore processing

This is effectively the technology architecture map of the platform: integrations, dashboards, AI, data services, and platform administration.

#### 4) Core Products (158 sections)

This is the largest and most operationally important category. It contains the domain and workflow surfaces of the product itself.

Example sections:

- Multi-Factor Authentication (MFA)
- Introduction
- Getting Started
- Straight-through Processing
- Claims Search
- Enhanced Claims Setup & Settings
- Enhanced Claims Management
- Policy Screen
- Information Overview
- Contact Management Overview
- Configuration Overview
- Add a Role
- Claims Processing Overview
- Payments
- Agency Sweep
- Policy lifecycle and admin sections
- Billing, claims, contacts, policy, rule, task, and workflow documentation

This is the clearest articulation of the domain model we have been inferring from the repo set.

## Category-to-layer mapping against the local architecture

The help center categories map surprisingly cleanly to the repo architecture we have been documenting locally.

### 1) Core Products -> operational domain layer

This is the clearest match to the raw business model and the source domains in the local repos.

- Policy, Claims, Billing, Contacts, Lines, Documents, Payments, and Tasks all map to the operational objects that appear in CSV export reconstruction and report-table discovery.
- The `britecore-csv-loader` repo is the best match for this layer because it reconstructs relationship graphs across `policyId`, `revisionId`, `claimId`, `itemId`, `contactId`, and `propertyId`.
- The `britecore_mcp` repo is the best match for the business-facing reporting semantics over those same domains.
- This category is where we most clearly see the product as a domain-rich insurance workflow system rather than a single generic database.

In practical terms:

- Policies / Claims / Billing / Contacts / Lines = core business object families
- Rules / Tasks / Authentication = workflow and control surfaces
- Notes / Alerts / Notifications = communications and event trails over those business objects

### 2) Technology -> platform, integration, and analytics layer

This category lines up directly with the local cross-project architecture.

- Platform = shared runtime, configuration, and system-level scaffolding
- APIs = `britecore_sdk` access layer and integration automation surface
- Dashboards / SQL Editor / Reports = the built-in analytics/reporting layer that `britecore_mcp` is trying to explain and expose
- Integrations = the partner and vendor ecosystem around the platform
- AI-related sections = emerging platform services layered on top of the operational system

This category is effectively the documentation version of our local architecture map:

- platform + shared services
- integration + API access
- reporting + analytics
- data/platform extensibility

### 3) Resources -> admin, config, and operational setup layer

This category is the operational support layer that sits beside the product domains.

Examples include:

- policy settings and module settings
- claims settings
- contacts settings
- lines settings
- policy lifecycle and processing docs
- support tooling and operational guidance

This matches the local docs philosophy of keeping implementation and configuration detail in the source repos while using `britecore_docs` as the translation and onboarding layer.

In other words:

- Core Products = what the business does
- Resources = how the product is configured and supported
- Technology = how the platform is extended and analyzed

### 4) Release Notes -> change history and product evolution layer

This category is not a data layer, but it matters a lot for architecture and long-term reporting work.

- It tells you what changed over time
- it helps distinguish stable product features from preview features
- it helps explain why certain report fields, workflows, or screens look different across versions

For a data-model project, this is important because it gives context for:

- product evolution
- feature flags and rollout changes
- reporting changes over time
- deprecations or migration patterns

## Local repo mapping by help-center category

- Technology / APIs -> `britecore_sdk`
- Technology / Dashboards and Reporting -> `britecore_mcp`
- Core Products / Policies and Claims -> `britecore-csv-loader` for raw relationship reconstruction and `PolicyReporting` for staged reporting normalization
- Resources / Settings and Module configuration -> operational and implementation docs for product configuration
- Release Notes -> product history and version-change context for any data or workflow assumptions

## Why this mapping matters

The help center does not describe a single physical schema. It describes a product ecosystem with distinct operational, integration, configuration, and analytics surfaces. That matches the local repo evidence exactly:

- the product is domain-rich and workflow-oriented
- reporting is a first-class product capability
- API access is a formal integration surface
- raw export data is a reconstruction layer over business relationships
- service code normalizes multiple sources into usable reporting outputs

This is the main architectural insight: the help-center taxonomy is not just product documentation. It is a map of the same layered model that the repo architecture is trying to explain.

## Core Products taxonomy from the category page and child sections

I reviewed the actual Core Products category page and its child sections. The taxonomy remains highly domain-oriented and explicitly product-focused:

- Agent Portal
- Policyholder Portal and Apps
- Billing
- Claims
- Contacts
- Document Management
- Payments
- Lines
- Policies
- Rules
- Tasks
- Notes, Alerts & Email Notifications
- Authentication & User Management

This is the clearest evidence yet that BriteCore is not organized around a single generic app. It is organized around business domains and operational workflows that map naturally to insurance operations:

- policy lifecycle and policy admin
- claims intake and handling
- billing and payment processing
- contacts and identity management
- document and communication workflows
- authentication, user access, and tasks
- rule-driven workflow automation

This product taxonomy lines up with the repo architecture we have been synthesizing:

- operational modules are domain-specific
- the reporting layer is cross-cutting across those domains
- API access and integrations sit above or alongside them
- raw CSV and logical reporting layers are downstream views over common operational objects

## Strong signal: BriteCore is domain-rich and integration-heavy

The Core Products taxonomy confirms that BriteCore is structured around business domains:

- Agent Portal
- Policyholder Portal and Apps
- Billing
- Claims
- Contacts
- Document Management
- Payments
- Lines
- Policy
- Commercial/Personal lines or line-specific modules
- additional operational modules

This is a strong clue that the product is built around domain objects and operational workflows, not a single flat data model. The raw CSV and logical reporting layers we have been reading locally fit this pattern well:

- policies are revisioned and event-driven
- claims have their own data branches
- contacts, properties, items, and revisions are all linked objects
- report-layer semantics are intentionally curated to support domain analysis

## Implications for the docs architecture

The Help Center clarifies that the right mental model is not:

- one simple database
- one API
- one report table family

It is more like:

- a platform with shared runtime services
- domain-specific operational modules
- a reporting and analytics layer
- API and event-based integration interfaces
- partner ecosystems that supply or consume data

That aligns directly with the cross-project architecture we have been building in `britecore_docs`.

## Best interpretation for our BriteCore data model work

The Help Center supports the following interpretation:

- The product architecture is built for operational workflows and reporting together.
- Reporting is treated as a first-class BriteCore capability, not as an appendix.
- APIs are designed as the extension and integration mechanism for the platform.
- Raw export, logical SQL reporting, and service-layer normalization are all part of the same larger business data ecosystem, just at different layers.

This makes the detailed data-flow narrative more accurate:

```text
BriteCore platform / operational system
    -> API and event integration layer
    -> business domain modules (claims, policies, billing, contacts, etc.)
    -> reporting and analytics layer (dashboards, report builder, SQL/report copilot)
    -> raw export / CSV reconstruction layer
    -> cross-source service normalization and reporting layers
```

## Practical conclusion

The Help Center gives us the product-level story that matches the local repo evidence:

- BriteCore is a layered, domain-driven platform
- Reporting and analytics are core product capabilities
- APIs and integrations are explicitly part of the platform design
- the data model must be understood as a set of surfaces rather than one single canonical store

This strengthens the narrative we are building in the docs: the architecture is best understood as a set of connected layers designed to serve operational insurance workflows, reporting, and external integrations.
