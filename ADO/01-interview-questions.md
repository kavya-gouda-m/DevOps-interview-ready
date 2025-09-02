### How would you design an enterprise-scale Azure DevOps architecture for multiple business units?

Goals: scalability, security, governance, autonomy per BU, cost control, reusability.

High-level design:

-   Option A: Single organization with multiple projects — best for centralized governance and cross-team collaboration.
-   Option B: Multiple organizations (one per large BU) — if strict data/isolation/regulatory needs exist.

Projects & Repos:
- One project per product or BU; use repo-per-service with branch policies (PR reviews, build validation).

Pipeline strategy:
- Use YAML pipelines in repos (pipeline-as-code). Provide centralized templates (YAML templates) stored in a shared repo or artifact that teams import.

Agent Pools:
- Shared hosted pools for common workloads; dedicated self-hosted pools for special requirements (compliance, high IO). Use VMSS to scale self-hosted agents.

Artifacts & Package Feeds:
- Central Azure Artifacts feed per organization with upstream sources, or feed-per-BU for isolation.

Secrets & Identity:
- Central Key Vault per environment (dev/test/stage/prod) with RBAC. Use service connections and Managed Identity where possible.

Access Control & Governance: 
- Enforce branch policies, PR approvals, and enforce security checks via pipeline gates.
- Use Azure AD groups and Azure DevOps groups for RBAC; least privilege for service connections.
- Enforce policy as code (ARM/Bicep/Terraform) for infra provisioning.

Environments & Approvals:
- Define environments in Azure DevOps (e.g., dev, qa, stage, prod) with approvals & checks (e.g., business approvers, security scans).

Security & Compliance:
- Integrate SAST/DAST (SonarQube, Dependabot, Trivy) into pipelines; require passing policies for prod gate.

Secrets scanning in CI: 
- supply chain security checks (signed packages, SBOM).

Observability & Telemetry:
- Central logging/monitoring (Azure Monitor, Log Analytics, App Insights) with shared dashboards. Pipeline metrics and cost reporting.

Templates & Libraries:
- Central “DevOps Platform” team delivers YAML templates, reusable tasks, scripts, and CI/CD modules (immutable and versioned).

Self-service & Onboarding: 
- Provide starter templates, pipeline-as-code samples, and a catalog of approved service connections/agent pools.

Disaster Recovery & Backups: 
- Back up pipeline definitions, export key configurations, and have DR plan for agent pools and artifact feeds.

###  If a deployment is failing in production, how would you troubleshoot and recover quickly using Azure DevOps?

Immediate goal: Restore service quickly while preserving data integrity — then do root-cause analysis.

Triage & quick recovery steps:

Stop further changes
- Pause any automated releases and block new deployments (disable pipeline triggers).

Assess scope
- Check Azure DevOps pipeline run details — error messages, failed tasks, timestamps.
- Consult monitoring (App Insights, Azure Monitor alerts, logs) to see service errors / rollback necessity.

Quick fixes / rollbacks
- If latest deployment caused failure, roll back to the last known-good release (Azure Pipelines has previous artifacts and release history — trigger a rollback deployment).
- If using infrastructure as code (ARM/Bicep/Terraform), roll back to previous version or scale out older replicas.
- If feature flags used, toggle off the problematic feature immediately.

Containment
- If database migration caused the issue, take app offline for write traffic, run rollback migration, or point traffic to read-only endpoints while fixing.

Use deployment strategies to minimize impact
- Blue/Green or Canary — switch traffic back to the stable environment quickly (route via Azure Front Door / Application Gateway).

For pipeline-specific issues
- Re-run pipeline with diagnostic logging; run failing stages on different agent pool to isolate agent problems.

Communication
- Notify stakeholders and follow incident communication plan; update status pages.

Postmortem
- After recovery, capture root cause, fix pipeline (timeouts, retries, env checks), add tests, add pre-deploy validations.

### Describe a challenging issue you resolved in your Azure DevOps setup and how.

### How do you manage secrets across multiple environments and pipelines securely?

Goal: Ensure secrets are protected, audited, and available only to authorized pipelines and agents.

Principles
- Least privilege, separation of duties, auditability, no secrets in source control.

Azure-centric approach
- Azure Key Vault for storing secrets, certificates, keys.
- Service Connections & Managed Identities for pipelines to access Azure resources instead of long-lived credentials.
- Variable groups in Azure DevOps linked to Key Vault for pipeline access (scoped to environment/pipeline).
- Secure files (for certificates/keystores) stored in Azure DevOps secure files when needed.
- Secret scanning & policies to prevent checked-in secrets (pre-commit hooks, git-secrets, pre-receive hooks).
- Rotation & expiration: automate rotation with Key Vault + runbooks / Azure Automation Logic Apps.
- Auditing & logging: Key Vault logging to Log Analytics; pipeline run logs (but redact secrets).
- Network isolation: use Private Link / firewall rules for Key Vault; restrict access to only build agent IP ranges or VNet.
- Approval gates: require approvals for production variable groups; use Manual Intervention steps for sensitive deploys.
- Use short-lived tokens: OAuth tokens or managed identity tokens instead of static secrets

### How would you migrate an on-prem CI/CD system (like Jenkins) to Azure DevOps?

Goal: Move CI/CD to Azure DevOps with minimal downtime, preserve build/test quality, improve security and scalability.

Approach (high level):
- Assess current setup: jobs, agents, plugins, secrets, integrations, artifact storage, and pipeline triggers.
- Prioritise apps by risk/complexity and migrate incremental (pilot → critical → remaining).
- Use “lift-and-shift” for complex pipelines initially (wrap Jenkins with service connections) then refactor to native Azure DevOps YAML pipelines.

In Detail

Inventory & mapping
- Catalog repos, build steps, test suites, deploy targets, credentials and agents.

Proof-of-concept
- Pick 1–2 non-critical pipelines. Implement YAML pipelines in Azure DevOps (CI) and multi-stage YAML releases (CD).

Source control
- Move or link repositories to Azure Repos or integrate GitHub. Preserve commit history (git clone + push).

Build agents
- Replace on-prem Jenkins agents with Azure Pipelines hosted agents or self-hosted VM/VMSS agents for special tooling.

Secrets & service connections
- Replace Jenkins credentials with Azure Key Vault + variable groups + service connections (Azure SPs or Managed Identities).

Artifacts
- Move binary artifacts to Azure Artifacts, or external feeds (artifact retention, upstream sources).

Tests & quality gates
- Wire SonarQube/quality gates to pipelines; migrate post-build tests.

Cutover
- Run both systems in parallel, compare outputs. Cut traffic to new pipelines when stable.

Decommission

- Remove old Jenkins agents and servers after rollback window and audit.



Cost & Quota Management:  
- Tag resources by BU, monitor pipeline minutes (hosted agents), and enforce quotas.
