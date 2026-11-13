---
# ── MADR 4.0.0 keys (same meaning as in MADR) ─────────────────────────────
status: proposed                      # proposed | accepted | rejected | deprecated | superseded
date: YYYY-MM-DD                      # date of the last status change
decision-makers: [design-authority]   # [steering-committee] for pilot selection, wave plan, budget, GA, T4 agents
consulted: [ciso-delegate, dpo, legal, network-team, security-team]  # e.g. [coe-squad-a, bu-champions, dpo, legal, ciso-delegate]
informed: [steering-committee]

# ── Identification and ownership ──────────────────────────────────────────
id: ADR-002                           # next free number, never reused; matches decisions/NNN-title-with-dashes.md
owner: "{name}, CoE platform lead"
proposer: "CoE Squads A and B"
drafted-with: ""                      # "{agent}@{version}" when agent 2 (Architecture and ADR drafting) wrote the draft
rfc: ""                               # pull-request URL, added once the pull request is open

# ── Links ─────────────────────────────────────────────────────────────────
supersedes: []                        # e.g. [ADR-004]
superseded-by: []
related: [ADR-001, ADR-003, ADR-005, ADR-010]  # other ADRs, standards, review records in reviews/ (design and post-promotion reviews)

# ── Scope tags (CI cross-checks, portal index, DA metrics) ─────────────────
zones: [control-plane, build, run, regional-replica]  # control-plane | build | run | regional-replica
jurisdictions: [global]               # global | eu | uk | us | br | ca | mx | cn | sg | jp | in | sa | ae | za | au | tr
                                      #   (same codes as policy/jurisdiction/ in the policy-source repo)
standards: []                         # 1 agent contract · 2 skills-file · 3 MCP server sanctioning · 4 LLM gateway policy
                                      # 5 DLP and tokenization · 6 policy-as-code · 7 promote path and human gate
                                      # 8 sandbox execution · 9 audit and evidence · 10 identity and mTLS
                                      # 11 risk tiering and EU AI Act · 12 exceptions
agents: [9, 12]                       # 1–15 as numbered in the agent-role table (10 = human validation gate)
risk-tiers: [T2, T3]                  # T1 | T2 | T3 | T4
revisit: "first sovereign-region replica (ADR-010) or a second cloud provider"  # date or trigger for re-examining the decision
---

# ADR-002: Control plane once, data planes per zone and sovereign region, each zone provisioned in isolation

<!--
How to use this template
* One decision per file. Copy it to decisions/NNN-title-with-dashes.md and open a pull request under your own account:
  the pull request is the RFC. Agent 2 may draft it (note it in drafted-with) but never opens it, because it is T1.
* Sections without an "optional" marker stay. If one does not apply, write "None, because {reason}" rather than deleting it.
* An ADR records a decision. Design detail belongs in the standard or the design-review record; lessons learned belong in the review record.
* Once accepted or rejected, the file is frozen: only status, date and superseded-by may change.
  A new decision is a new ADR that lists this one under supersedes.
* A deviation from an accepted ADR is requested through Standard 12 (exceptions), not through a new ADR.
-->

## Summary

In the context of building the three-zone platform retained in ADR-001, facing the need to share registries, policy, audit and the human validation gate across zones without creating a path between them, and the risk that the IaC, its state or its pipelines become a second Build → Run crossing, we decided for one control plane deployed once, a data plane per zone (and per sovereign region where a BU's law forbids EU egress) built from one module, and all zones provisioned from one repository, each with its own root, state backend, pipeline identity and self-hosted runner pool, and against a full platform per zone, one platform for all zones, per-zone repositories, CLI workspaces, shared Build-zone runners and GitHub-hosted runners, to achieve zone isolation that holds by construction (ADR-001, ADR-003), accepting a shared control plane to keep available, three runner pools and a one-off bootstrap.

## Context and Problem Statement

The retained diagram (ADR-001) has three zones; this ADR decides what exists once, what exists per zone, and how each zone is built. Every account, network and service from Day 16 on is deployed by IaC, so the structure of that code fixes zone boundaries, which the decision-rights table gives to the DA. A shared state file, a pipeline identity with rights in two zones, a Build runner that plans Run or a peering route would each break ADR-001 or ADR-003. The pipelines run on GitHub Actions, so where their jobs execute decides where plan and apply credentials, and the state they read, can exist. No standard in the catalogue covers IaC or source control, so this ADR carries its own checks (Confirmation). It blocks Day 16 (accounts by IaC) and Day 17 (connectivity); the Day 8 skeleton creates no cloud resources and does not wait for it. Decision-backlog item: none, because the topology comes from the retained diagram walked through on Day 2 and the Day 3 backlog has no IaC entry; this RFC adds one. Out of scope: the IaC tool (Terraform, as named in the plan), the cloud provider, the products inside each plane (Day 3 decision backlog), the regional replica pattern (ADR-010) and the content of the data-plane module (Days 31 onward).

## Decision Drivers

<!-- Ranked, most important first. Drivers that recur on this platform: isolation of the three zones and the single
Build → Run crossing (ADR-002, ADR-003); data residency and EU egress; fail-closed behaviour; auditability in the WORM log;
operability across 14 BUs and sovereign regions; time to the first gated promotion; cost; exit cost and lock-in. -->

1. ADR-003: no Run credential, read or write, and no Run endpoint reachable from Build, including from CI.
2. ADR-001 and the retained diagram: three zones; private networks, mTLS, egress only through the company DLP reverse proxy.
3. One source of truth for every zone: agent and skills registries, policy bundles, DLP rules and vault, WORM audit log and the human validation gate.
4. Residency: some BUs are under laws that forbid EU egress, so their data, model traffic and keys must stay in their region.
5. Threat model v0.1 (Day 8): cross-zone credential leakage; supply chain of providers and modules.
6. Auditability: every change is a reviewed pull request, and every apply is attributable.
7. Dates: Board #2 on Day 11; Day 16 provisioning, Day 17 connectivity, design review #1 on Day 22.

## Considered Options

<!-- At least two real options, including the current state when there is one.
Product names in the plan stay candidates until an ADR accepts one. -->

Topology (what exists once, what exists per zone):

* One control plane deployed once; a data plane per zone, and per sovereign region where law requires
* A full platform per zone: control plane and data plane in each zone
* One platform for all zones: one control plane and one shared data plane

Provisioning (how each zone is built):

* One repository, one root, state backend and pipeline identity per zone
* One repository per zone, plus a module repository
* One root with a CLI workspace per zone
* Per-zone roots on shared Build-zone runners
* Per-zone roots on GitHub-hosted runners

## Decision Outcome

Chosen options: "One control plane deployed once; a data plane per zone, and per sovereign region where law requires" for the topology, and "One repository, one root, state backend and pipeline identity per zone" for provisioning, because together they give every zone one source of truth (driver 3) while keeping credentials, state and runtime inside each zone (driver 1), with one data-plane module that also serves a regional replica (driver 4), all through one pull-request flow (driver 6).

Confidence: medium, because the topology is the retained design (ADR-001) and the provisioning follows HashiCorp's guidance, but the bootstrap has not been rehearsed; a rehearsal in a throwaway account before Day 16 would raise it to high.

The topology:

* **Control plane, deployed once in its own account:** agent registry, skills-files registry (Git + OCI, signed bundles), DLP rules + key/token vault, artifact registry, WORM audit log, policy source (OPA/Cerbos → signed bundles), Temporal + orchestrator, human validation gate (`interrupt()`).
* **A data plane per zone, from one module:** Build (non-prod account, synthetic or masked data only): Design & Build agents 1–9, local OPA/Cerbos PDP, sandbox pool (gVisor/Firecracker) + CI runners + Git/SCM, DLP-MCP gateway with inline tokenization on egress, LLM gateway + MCP fleet (EU egress). Run (prod account, prod credentials and data): Deploy & Run agents 11–15, local PDP, release pipelines + runtime operators, DLP-MCP gateway, LLM gateway + MCP fleet (EU egress). Every DLP-MCP gateway is built on the company's DLP reverse-proxy gateway scheme.
* **Connectivity:** each zone reaches the control plane over private connectivity and mTLS, with no public endpoint (workload identity: ADR-005). The control plane serves each zone separately and routes no traffic between them, so Build and Run have no path to each other through it. The promote path build → gate → run is the only crossing between Build and Run (ADR-003).
* **Regional data plane:** any BU whose law forbids EU egress gets a local data plane in its region: LLM gateway, MCP fleet, DLP-MCP gateway, WORM replica, keys. What is replicated, what stays central and who holds the keys are decided in ADR-010.

Provisioning fixes this layout and these rules:

```text
platform-as-iac-repo/
├── bootstrap/               # stage 0, applied once on Day 16: accounts, state backends, OIDC trust, zone runners
├── zones/
│   ├── control-plane/       # root: platform-team account, instantiated once
│   ├── build/               # root: non-prod account; uses modules/data-plane
│   └── run/                 # root: prod account; uses modules/data-plane
├── modules/
│   ├── account-baseline/    # guardrails: organisation policies, no public ingress, central logging, tags
│   ├── network/             # private connectivity, egress-deny default, route to the DLP reverse proxy
│   └── data-plane/          # one module for every data plane; filled from Day 31
├── policy/                  # one plan check per control in Confirmation, each with a fixture plan that must fail
├── CODEOWNERS               # zones/run/ and bootstrap/: Squad B lead + CoE security engineer; zones/control-plane/: Squad A lead
└── .github/workflows/       # plan on pull request, apply on merge, per zone
```

* **Pull request:** format, validate, lint; plan each affected root on that zone's runners; run `policy/` against the plan JSON; post the plan to the pull request.
* **Merge to main:** apply each affected root on that zone's runners with that zone's short-lived identity. The Run apply waits for a second approver in a protected environment.
* **Bootstrap:** applied once on Day 16 by two named people from the control-plane account; its credentials are then locked as break-glass.
* **Control-plane outputs** that Build and Run need are published as non-secret parameters; no zone reads another zone's state.
* **Who decides what:** GitHub decides what runs: the workflows, the rulesets on `main` and the protected environments. The platform decides where and how: each zone's runners execute the jobs, and a token broker on every runner checks the job's GitHub OIDC token before handing out that zone's credentials.
* **Where jobs run:** every job that holds a zone's credentials, or plans, applies or tests a zone, runs on that zone's runners. Jobs that hold no platform credential and reach no platform endpoint (change detection, static checks, result aggregation) may run on GitHub-hosted runners.
* **Repository visibility:** the repository is private, and each zone's runner group admits this repository only and no public repository, because a public repository would let any fork's pull request run code on runners inside the zones.

### Consequences

<!-- Cover operations (SLOs, on-call, DR targets RPO/RTO, capacity) and cost (infrastructure, licences, token spend),
not only design qualities. -->

* Good, because registries, policy, the vault and the audit log have one source of truth: a revocation or a new policy applies to every zone at once.
* Good, because the IaC cannot become a second Build → Run crossing: credentials and state never leave their zone.
* Good, because a job on GitHub's machines cannot touch a zone: zone credentials exist only on that zone's runners.
* Good, because Build and Run run the same data-plane module: a change lands in Build first and Run follows by version; a regional replica reuses the same module.
* Good, because one pull-request flow covers every zone, and every apply is attributable.
* Neutral, because Run may run an older data-plane module than Build; the lag is bounded (C10).
* Neutral, because state backends are versioned, so a lost or corrupted state can be restored; DR targets are reviewed at design review #6 (Day 66).
* Neutral, because the licence cost depends on the IaC tool, which is out of scope, and the pipelines spend no tokens unless agents 9 and 12 run them.
* Neutral, because GitHub stays in the path of every change: while GitHub Actions is unavailable, applies wait or go through break-glass.
* Bad, because the control plane is a shared dependency: the orchestrator, the human validation gate and the registries run there, so its outage stops orchestrated work and promotions in every zone, and a new PDP that cannot fetch a signed bundle fails closed (ADR-004). Its RPO ≤ 1 h and RTO ≤ 4 h targets and bundle-server failover are checked at design review #6 (DR, Day 66).
* Bad, because each regional replica is one more data plane to run, patch and pay for in its region.
* Bad, because three runner pools add infrastructure cost, and the Run pool is production: it needs patching, monitoring and on-call.
* Bad, because a change touching several zones needs ordered applies: control plane, then Build, then Run.

### Confirmation

<!-- How compliance is enforced and proven. Prefer enforcement in code over procedure (the ADR-006 principle).
For a procedural control, say why and who checks it. -->

| Control | Location | Proof |
|---|---|---|
| C0. Topology: 1 control plane; Build and Run data planes from the same module (C10); 0 BUs flagged under the regional trigger served by an EU data plane | Design reviews #1 (landing zones, Day 22), #2 (control plane, Day 32), #3 (Build data plane, Day 38), #4 (Run zone and promote path, Day 45) and #5 (regional replica, Day 65); the Day 19 localization flags | Review records in `reviews/`; C3 and C10 on every pull request; the flags checked when each BU onboards |
| C1. The Run plan and apply roles trust only Run-zone runners: 0 assumable from Build | Trust policies in `bootstrap/`; `policy/c1_run_trust.rego` | A test from a Build runner is refused, on every pull request touching `bootstrap/` and at design review #1 (Day 22) |
| C2. No zone identity holds permissions outside its own account: 0 | Scheduled IAM policy analysis in each account | Weekly report from Day 16, attached to design review #1 |
| C3. No Build ↔ Run peering, transit or route: 0 in plans, 0 reachable paths | `policy/c3_no_build_run_path.rego`; reachability analysis after each network apply | Plan check on every pull request; reachability report after network applies and at Day 22 |
| C4. No zone reads another zone's state: 0 `terraform_remote_state` or backend data sources across zones | `policy/c4_no_cross_zone_state.rego` | Plan check on every pull request |
| C5. Default routes in Build and Run target only the DLP reverse proxy: 0 others | `policy/c5_egress_via_proxy.rego` | Plan check on every pull request; route-table export at Day 22 |
| C6. One state backend per account, with its own key and versioning, in an EU region (a regional replica's in its own region): 3 backends before the first replica, 0 readers from another zone | Backend definitions in `bootstrap/`; `policy/c6_state_isolation.rego` | Plan check; backend access report on Day 16 and Day 22 |
| C7. No static cloud credentials; OIDC tokens last at most 1 h: 0 static keys | Pipeline definitions in `.github/`; secret scanning | CI check and secret-scanning result on every pull request |
| C8. No change outside the pipeline, recorded break-glass excepted: 0 | Scheduled audit-log query in each account, alerting on any change by another principal | Alert history and weekly report from Day 16; each alert reviewed by the CoE security engineer |
| C9. Providers locked and module sources pinned to a tag or commit: 0 unpinned | `.terraform.lock.hcl` in each root; `policy/c9_pinned_sources.rego` | CI check on every pull request |
| C10. Build and Run use the same `modules/data-plane`, Run at most 1 release behind Build | `policy/c10_data_plane_parity.rego` | Plan check on every pull request |
| C11. Every check above fails on a fixture plan that violates it: 100 % | `policy/fixtures/` | OPA unit tests (`opa test`) on every pull request |
| C12. No drift left unresolved after 2 working days: 0 | Scheduled plan of each root | Daily drift report from Day 16 |
| C13. Zone credentials go only to jobs on that zone's own runners: 0 issued to a job on a GitHub-hosted runner | Token broker on each zone's runners: it answers only on the runner itself and refuses tokens from jobs not on a self-hosted runner | Broker unit tests on every pull request; the broker's decision log in each zone |
| C14. The repository is private, and each zone's runner group admits this repository only and no public repository: 0 other repositories | `scripts/configure-github.sh`: stops unless the repository is private, then restricts each runner group to it | Settings export at design review #1 (Day 22) and after every change to the script |
| Bootstrap applied once, by two named people, from the control-plane account; its credentials then locked as break-glass | `bootstrap/`; break-glass record | Procedural, because no pipeline exists before the bootstrap; the CISO delegate holds the break-glass record and checks it at the risk and security review |

## Impact Assessment

### Zones and Boundaries

| Zone | Change |
|---|---|
| Shared control plane | Deployed once, in its own account: registries, vault, WORM audit log, policy source, orchestrator and the human validation gate; serves Build and Run separately, with no route between them. New root `zones/control-plane/` with its own state backend, runner pool and pipeline identity; the bootstrap is applied from this account; publishes non-secret outputs for Build and Run |
| Build zone | One data plane from the shared module; synthetic or masked data only. New root `zones/build/` with its own state backend, runner pool and pipeline identity; instantiates `modules/data-plane` |
| Run zone | One data plane from the shared module; production credentials and data. New root `zones/run/` with its own state backend, runner pool and pipeline identity; instantiates `modules/data-plane`; each apply waits for a second approver |
| Regional replica(s) | A local data plane wherever a BU's law forbids EU egress, with its own root, state backend and runners in its region; the replica pattern is ADR-010 |

* New or changed path between Build and Run: No; the promote path stays the only crossing (ADR-003)
* Credentials, keys or endpoints reachable from another zone after this change: no cloud credential and no state. Build and Run reach only the services the control plane publishes to them and read its published non-secret outputs; no zone holds another zone's credentials or reads its state. One exception on the GitHub side: the GitHub App key that registers runners sits in all three zones, and its organisation-wide permission can register a runner in any runner group, Run's included. Root on any runner could therefore take Run's jobs and report false results, but could not obtain Run credentials, which only the token broker on Run's own runners issues (C13).
* Outside services that can start jobs in a zone, issue identities a zone trusts, or receive its data after this change: GitHub. It starts every job on all three runner pools; it issues the OIDC tokens each zone's token broker checks before handing out credentials; it receives the job logs and the plans posted to pull requests, with secret values redacted. The model endpoints behind each LLM gateway are outside services too; which ones, and with which data, is set by the LLM gateway policy (standard 4).

### Agents, Skills, MCP Servers and Models

Agents 1–9 run in the Build data plane and agents 11–15 in the Run data plane; the human validation gate (10) sits in the control plane. For the IaC, agent 9 (Build/CI, T2) may plan the Build root only, and agent 12 (IaC apply, T3) plans and applies the Run root, from the Run zone. No tier changes and no T4 capability. No skills bundle, MCP server or model endpoint is added: this ADR places an LLM gateway and an MCP fleet in each data plane, and standards 3 and 4 decide what runs behind them.

### Security and Threat Model

Threat model v0.1 (Day 8), entries {IDs}: cross-zone credential leakage; supply chain of providers and modules. New assets: the shared control plane (registries, vault, WORM audit log, policy source, orchestrator, gate), three state backends, three pipeline identities, three runner pools, the GitHub App key that registers runners, and the bootstrap credentials. New trust boundaries: the private, mTLS links from each zone to the control plane (ADR-005); and GitHub, outside every zone, which decides which jobs run on the zone runners and issues the OIDC tokens each zone's token broker trusts. Abuse cases examined: cross-zone credential leakage (C1, C2, C4, C7, C13); a path between zones through the control plane (C3); over-privileged automation (C2, C8); a compromised provider or module (C9); a fork's pull request running code on zone runners (C14); approval fatigue at the Run apply, limited because only Run applies need a second approver. Left to the standards that design these components rather than place them: prompt injection through MCP tool outputs and over-privileged MCP scopes (standard 3), poisoned skills bundle (standard 2), sandbox escape (standard 8), DLP evasion (standard 5). Residual risks: every zone trusts the control plane, so its compromise affects all of them, limited by mTLS, signed bundles and the absence of any route between zones through it (C3), and examined first at design review #2 (Day 32); a compromised Run runner can change Run, limited by ephemeral and patched runners, the second approver and C8; a pull request runs its own version of the workflows on the zone runners and receives that zone's read-only plan credentials, which can read the zone's state, before any review, because planning needs read access; root on any runner can use the shared GitHub App key to register a runner in Run's group and take Run's jobs, without obtaining Run credentials (C13). CISO delegate review: draft at the Day 8 risk and security review; {date} for this version.

### Data Protection and Regulation

| Question | Answer |
|---|---|
| Data classes processed, stored or moved | Build handles synthetic or masked data only, as the retained design states (ADR-006 decides how this is enforced); Run handles production data. The IaC holds infrastructure metadata in state files and plans, and state can hold secrets; no customer or personal data, apart from the identities of committers and approvers |
| Jurisdiction overlays affected | None directly; the regional trigger depends on each BU's jurisdiction |
| Residency, EU egress, cross-border transfer basis | Data planes, state backends and runners in an EU region by default; LLM gateways egress to the EU; a BU whose law forbids EU egress gets its data plane, data and keys in its own region. Job logs and the plans posted to pull requests are stored by GitHub: {hosting region}; secret values are redacted before posting, and the repository is private, so neither is public |
| Regional replica required (ADR-002 trigger) | This ADR sets the trigger: any BU whose law forbids EU egress. The BUs concerned are flagged on Day 19; the replica pattern is ADR-010 |
| EU AI Act classification or transparency duty | None, because this ADR places the platform's components and introduces no AI system of its own; agents keep their classification under Standard 11 |
| Sector regimes (PCI DSS, EU DORA, SOX) | Supports change-management evidence for production infrastructure: every Run change is a reviewed pull request with a second approver and an attributable apply (SOX ITGC; EU DORA ICT change management where the entity is a financial entity) |
| Developer telemetry and works-council constraints | Committer and approver identities are kept for attribution (driver 6), not for productivity measurement; to be raised in the works-council consultation opened on Day 15 |
| Retention of audit events, prompts and responses | Plans, pipeline logs and cloud audit logs follow Standard 9 once drafted (Day 47): {retention}. No prompts or responses |
| DPO and Legal consulted (who, date) | {who, date} |

### Standards, Policy Packs and Registers

| Item | Change |
|---|---|
| Standards created or changed (number, target version) | None, because no standard covers IaC or source control. Standards 9 and 10, drafted on Day 47, will cover C1, C7 and C8; until then this ADR's Confirmation is the only yardstick. Standard 8 (sandbox execution) governs the Build data plane's sandbox pool, not these pipeline runners: the Run runners must hold production apply credentials, which standard 8 rules out for its runners |
| Policy packs: global baseline, jurisdiction overlay, BU overlay | None, because the checks in `policy/` belong to the CoE's IaC repository, not to policy-source (ADR-007) |
| Registers: agent registry, skills registry, MCP register, model allowlist | None; the registries live in the control plane, and their content is set by standards 1 to 4 |
| Exceptions expected under Standard 12 | None, unless the Board reads standard 8 as covering these runners; the Run pool would then need one |

<!-- This is an optional element. Feel free to remove. -->
### Rollout and BU Impact

No BU impact before Pilot A onboards (Day 29), and no certification level concerned. The control plane and the Build and Run data planes are provisioned from Day 16 and run as an MVP by the end of Month 2. The first regional data plane is Pilot C's: Squad B builds its region-local accounts by IaC from Day 61, and it is reviewed (design review #5) and deployed on Day 65; later ones come with the waves that need them (Wave 4 APAC/MEA, Days 151–240). BU application IaC lives in BU repositories and is out of scope. Deadline: Day 16. Rollback: before Day 16, delete the skeleton; after Day 16, a change of structure is a new ADR that supersedes this one.

<!-- This is an optional element. Feel free to remove. -->
## Pros and Cons of the Options

### Topology: one control plane deployed once; a data plane per zone, and per sovereign region where law requires

The retained design (ADR-001): shared services once, in their own account; each zone's runtime in its own data plane, built from one module. Evidence: Microsoft's guidance on multitenant control planes shows one control plane managing several data planes, and treats the control plane as a mission-critical component whose resiliency must be planned ([Considerations for multitenant control planes](https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/considerations/control-planes)).

* Good, because registries, policy bundles, the vault and the WORM audit log have one source of truth for every zone.
* Good, because a new region reuses the data-plane module instead of needing a new design.
* Bad, because the control plane is a shared dependency: its outage stops orchestrated work and promotions in every zone. The known mitigation is static stability: each data plane keeps working on its last configuration while the control plane is impaired ([AWS Fault Isolation Boundaries: Static stability](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/static-stability.html)).

### Topology: a full platform per zone

A control plane and a data plane in each of Build and Run. Evidence: this is the Deployment Stamps pattern, independent copies of the whole system; Microsoft notes that it substantially increases operating cost, and that security policies, access control, network controls and configuration get harder to keep consistent as stamps multiply ([Deployment Stamps pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/deployment-stamp)).

* Good, because no zone depends on another account.
* Bad, because registries, policy and audit would be kept twice and drift apart: an agent revoked in one zone could stay registered in the other, and audit evidence would be split across two logs.
* Bad, because the human validation gate on the promote path would belong to one of the two zones it separates. Rejected on driver 3.

### Topology: one platform for all zones

One control plane and one data plane shared by Build and Run.

* Good, because there is one platform to run.
* Bad, because Build and Run would share runtime, network and credentials. Rejected on driver 1 alone.

### Provisioning: one repository, one root, state backend and pipeline identity per zone

Each zone has its own root module, state backend and pipeline identity, planned and applied on runners in its own account; shared modules are versioned. Evidence: HashiCorp recommends separate configurations along architectural boundaries, with common elements in reusable modules ([CLI workspaces and alternatives](https://developer.hashicorp.com/terraform/cli/workspaces)).

* Good, because credentials and state stay in their zone.
* Good, because a change to the data-plane module is reviewed once and applied zone by zone.
* Bad, because three runner pools.

### Provisioning: one repository per zone, plus a module repository

The same isolation, split across four repositories. No PoC.

* Good, because write access can differ per repository.
* Neutral, because CODEOWNERS per root in one repository gives the same review separation.
* Bad, because a cross-zone change becomes several coordinated pull requests, and module versions drift without one view.

### Provisioning: one root with a CLI workspace per zone

One configuration and one backend, with a workspace per zone. Evidence: HashiCorp states that CLI workspaces are not appropriate for deployments requiring separate credentials and access controls ([State workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces)).

* Good, because there is one configuration to maintain.
* Bad, because one backend and one identity span all zones, against ADR-003.

### Provisioning: per-zone roots on shared Build-zone runners

The first provisioning option's layout, but every root planned and applied from one runner pool in the Build zone.

* Good, because there is one runner pool.
* Bad, because Build runners would hold Run credentials: applying Run needs write access, and even planning Run needs read access. Rejected on driver 1 alone.

### Provisioning: per-zone roots on GitHub-hosted runners

The first provisioning option's layout, but every root planned and applied on runners that GitHub hosts outside the platform, with each job's GitHub identity federated directly to the zone's cloud roles.

* Good, because there is no runner pool to build, patch or pay for.
* Bad, because the pipelines' traffic would bypass the company DLP reverse proxy (driver 2).
* Bad, because plan and apply credentials, and the state they read, would exist on machines outside every zone, Run's included. Rejected on driver 1 alone.

## Decision Record

<!-- Kept by the DA secretary. One row per event; the discussion itself stays in the pull request.
After the Board, the secretary writes the outcome here (status, date, rows, conditions) on the pull request branch.
A DA code owner then approves the pull request, which records the decision, and the secretary merges.
Nothing is pushed after the approval: any later push dismisses it. A returned RFC stays open, with its Board row added. -->

| Date | Event | By | Notes |
|---|---|---|---|
| {YYYY-MM-DD} | RFC ready for review | {name}, CoE platform lead | Review window (2 working days, Mon–Fri CET) closes {YYYY-MM-DD HH:MM}, before Board #2 on Day 11 |
| {YYYY-MM-DD} | DA Board #2 | Design Authority | {accepted, rejected or returned} |

Conditions attached to the decision:

| # | Condition | Owner | Due | Status |
|---|---|---|---|---|
| 1 | {condition} | {owner} | {YYYY-MM-DD} | {open or met} |

Reservations recorded at the Board: {None, or name: reservation}

<!-- This is an optional element. Feel free to remove. -->
## More Information

ADR-001 (Board #1, Day 4); ADR-003 (Board #2, Day 11); ADR-004, policy engine and fail-closed PDP, and ADR-005, workload identity and mTLS (Board #3, Day 18); ADR-006, synthetic or masked data in Build (Board #3, Day 18); ADR-010, the regional replica pattern (Board #6, Day 39); threat model v0.1 (Day 8). Design reviews #1 to #5 (Days 22, 32, 38, 45, 65) check the topology, and #1 measures C1–C14 on the provisioned zones; their records go in `reviews/`. Reversibility: the layout is cheap to change until Day 16; state backends and identity trust are costly after Day 16; network topology is costly after Day 17. Microsoft: [Considerations for multitenant control planes](https://learn.microsoft.com/en-us/azure/architecture/guide/multitenant/considerations/control-planes) · [Deployment Stamps pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/deployment-stamp). AWS: [Static stability](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/static-stability.html). HashiCorp: [State workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces) · [CLI workspaces and alternatives](https://developer.hashicorp.com/terraform/cli/workspaces). GitHub: [Adding self-hosted runners](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/add-runners) · [REST API endpoints for self-hosted runners](https://docs.github.com/en/rest/actions/self-hosted-runners).
