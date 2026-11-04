\---

\# ── MADR 4.0.0 keys (same meaning as in MADR) ─────────────────────────────

status: proposed                      # proposed | accepted | rejected | deprecated | superseded

date: YYYY-MM-DD                      # date of the last status change

decision-makers: \[design-authority]   # \[steering-committee] for pilot selection, wave plan, budget, GA, T4 agents

consulted: \[ciso-delegate, network-team, security-team]  # e.g. \[coe-squad-a, bu-champions, dpo, legal, ciso-delegate]

informed: \[steering-committee]



\# ── Identification and ownership ──────────────────────────────────────────

id: ADR-NNN                           # next free number, never reused; matches decisions/NNN-title-with-dashes.md

owner: "{name}, CoE platform lead"

proposer: "CoE Squads A and B"

drafted-with: ""                      # "{agent}@{version}" when agent 2 (Architecture and ADR drafting) wrote the draft

rfc: ""                               # pull-request URL, added once the pull request is open



\# ── Links ─────────────────────────────────────────────────────────────────

supersedes: \[]                        # e.g. \[ADR-004]

superseded-by: \[]

related: \[ADR-001, ADR-002, ADR-003]  # other ADRs, standards, review records in reviews/ (design and post-promotion reviews)



\# ── Scope tags (CI cross-checks, portal index, DA metrics) ─────────────────

zones: \[control-plane, build, run]    # control-plane | build | run | regional-replica

jurisdictions: \[global]               # global | eu | uk | us | br | ca | mx | cn | sg | jp | in | sa | ae | za | au | tr

&#x20;                                     #   (same codes as policy/jurisdiction/ in the policy-source repo)

standards: \[]                         # 1 agent contract · 2 skills-file · 3 MCP server sanctioning · 4 LLM gateway policy

&#x20;                                     # 5 DLP and tokenization · 6 policy-as-code · 7 promote path and human gate

&#x20;                                     # 8 sandbox execution · 9 audit and evidence · 10 identity and mTLS

&#x20;                                     # 11 risk tiering and EU AI Act · 12 exceptions

agents: \[9, 12]                       # 1–15 as numbered in the agent-role table (10 = human validation gate)

risk-tiers: \[T2, T3]                  # T1 | T2 | T3 | T4

revisit: "first sovereign-region replica (ADR-010) or a second cloud provider"  # date or trigger for re-examining the decision

\---



\# ADR-NNN: Zone isolation in IaC: one repository with one root, state backend and pipeline identity per zone



<!--

How to use this template

\\\* One decision per file. Copy it to decisions/NNN-title-with-dashes.md and open a pull request under your own account:

\&#x20; the pull request is the RFC. Agent 2 may draft it (note it in drafted-with) but never opens it, because it is T1.

\\\* Sections without an "optional" marker stay. If one does not apply, write "None, because {reason}" rather than deleting it.

\\\* An ADR records a decision. Design detail belongs in the standard or the design-review record; lessons learned belong in the review record.

\\\* Once accepted or rejected, the file is frozen: only status, date and superseded-by may change.

\&#x20; A new decision is a new ADR that lists this one under supersedes.

\\\* A deviation from an accepted ADR is requested through Standard 12 (exceptions), not through a new ADR.






\## Summary SECTION



In the context of provisioning the three zones from Day 16, facing the risk that the IaC repository, its state or its pipelines become a second Build → Run crossing, we decided for one repository with one root module, state backend and pipeline identity per zone, planned and applied on runners inside that zone, and against per-zone repositories, CLI workspaces and shared Build-zone runners, to achieve zone isolation that holds by construction (ADR-001 to ADR-003), accepting three runner pools and a one-off bootstrap.



\## Context and Problem Statement



Every account, network and service from Day 16 on is deployed from this repository, so its structure fixes zone boundaries, which the decision-rights table gives to the DA. A shared state file, a pipeline identity with rights in two zones, a Build runner that plans Run or a peering route would each break ADR-001 to ADR-003. No standard in the catalogue covers IaC or source control, so this ADR carries its own checks (Confirmation). It blocks Day 16 (accounts by IaC) and Day 17 (connectivity); the Day 8 skeleton creates no cloud resources and does not wait for it. Decision-backlog item: none, because the Day 3 backlog has no IaC entry; this RFC adds one. Out of scope: the tool (Terraform, as named in the plan), the cloud provider and the content of the data-plane module (Days 31 onward).



\## Decision Drivers



<!-- Ranked, most important first. Drivers that recur on this platform: isolation of the three zones and the single

Build → Run crossing (ADR-002, ADR-003); data residency and EU egress; fail-closed behaviour; auditability in the WORM log;




1\. ADR-003: no Run credential, read or write, and no Run endpoint reachable from Build, including from CI.

2\. ADR-002: one control plane; one data-plane module instantiated per zone, and later per sovereign region (ADR-010).

3\. ADR-001 and the retained diagram: private networks, mTLS, egress only through the company DLP reverse proxy.

4\. Threat model v0.1 (Day 8): cross-zone credential leakage; supply chain of providers and modules.

5\. Auditability: every change is a reviewed pull request, and every apply is attributable.

6\. Dates: Day 16 provisioning, Day 17 connectivity, design review #1 on Day 22.



\## Considered Options



<!-- At least two real options, including the current state when there is one.




\* One repository, one root, state backend and pipeline identity per zone

\* One repository per zone, plus a module repository

\* One root with a CLI workspace per zone

\* Per-zone roots on shared Build-zone runners





\## Decision Outcome Section



Chosen option: "One repository, one root, state backend and pipeline identity per zone", because it is the only option that keeps credentials and state inside each zone (driver 1) while Build and Run instantiate the same data-plane module (driver 2) through one pull-request flow (driver 5).



Confidence: medium, because the structure follows HashiCorp's guidance and ADR-001 to ADR-003, but the bootstrap has not been rehearsed; a rehearsal in a throwaway account before Day 16 would raise it to high.



The chosen option fixes this layout and these rules:



```text

platform-iac/

├── bootstrap/               # stage 0, applied once on Day 16: accounts, state backends, OIDC trust, zone runners

├── zones/

│   ├── control-plane/       # root: platform-team account, instantiated once

│   ├── build/               # root: non-prod account; uses modules/data-plane

│   └── run/                 # root: prod account; uses modules/data-plane

├── modules/

│   ├── account-baseline/    # guardrails: organisation policies, no public ingress, central logging, tags

│   ├── network/             # private connectivity, egress-deny default, route to the DLP reverse proxy

│   └── data-plane/          # one module for both zones; filled from Day 31

├── policy/                  # one plan check per control in Confirmation, each with a fixture plan that must fail

├── CODEOWNERS               # zones/run/ and bootstrap/: Squad B lead + CoE security engineer; zones/control-plane/: Squad A lead

└── .github/ or .gitlab/     # plan on pull request, apply on merge, per zone

```



\* \*\*Pull request:\*\* format, validate, lint; plan each affected root on that zone's runners; run `policy/` against the plan JSON; post the plan to the pull request.

\* \*\*Merge to main:\*\* apply each affected root on that zone's runners with that zone's short-lived identity. The Run apply waits for a second approver in a protected environment.

\* \*\*Bootstrap:\*\* applied once on Day 16 by two named people from the control-plane account; its credentials are then locked as break-glass.

\* \*\*Control-plane outputs\*\* that Build and Run need are published as non-secret parameters; no zone reads another zone's state.



\### Consequences



<!-- Cover operations (SLOs, on-call, DR targets RPO/RTO, capacity) and cost (infrastructure, licences, token spend),




\* Good, because the IaC cannot become a second Build → Run crossing: credentials and state never leave their zone.

\* Good, because Build and Run run the same data-plane module: a change lands in Build first and Run follows by version.

\* Good, because one pull-request flow covers every zone, and every apply is attributable.

\* Neutral, because Run may run an older data-plane module than Build; the lag is bounded (C10).

\* Neutral, because state backends are versioned, so a lost or corrupted state can be restored; the DR targets are set at the Day 66 design review.

\* Neutral, because the licence cost depends on the IaC tool, which is out of scope, and the pipelines spend no tokens unless agents 9 and 12 run them.

\* Bad, because three runner pools add infrastructure cost, and the Run pool is production: it needs patching, monitoring and on-call.

\* Bad, because a change touching several zones needs ordered applies: control plane, then Build, then Run.



\### Confirmation



<!-- How compliance is enforced and proven. Prefer enforcement in code over procedure (the ADR-006 principle).




| Control | Location | Proof |

|---|---|---|

| C1. The Run plan and apply roles trust only Run-zone runners: 0 assumable from Build | Trust policies in `bootstrap/`; `policy/c1\\\_run\\\_trust.rego` | A test from a Build runner is refused, on every pull request touching `bootstrap/` and at design review #1 (Day 22) |

| C2. No zone identity holds permissions outside its own account: 0 | Scheduled IAM policy analysis in each account | Weekly report from Day 16, attached to design review #1 |

| C3. No Build ↔ Run peering, transit or route: 0 in plans, 0 reachable paths | `policy/c3\\\_no\\\_build\\\_run\\\_path.rego`; reachability analysis after each network apply | Plan check on every pull request; reachability report after network applies and at Day 22 |

| C4. No zone reads another zone's state: 0 `terraform\\\_remote\\\_state` or backend data sources across zones | `policy/c4\\\_no\\\_cross\\\_zone\\\_state.rego` | Plan check on every pull request |

| C5. Default routes in Build and Run target only the DLP reverse proxy: 0 others | `policy/c5\\\_egress\\\_via\\\_proxy.rego` | Plan check on every pull request; route-table export at Day 22 |

| C6. One state backend per account, with its own key, versioning and an EU region: 3 backends, 0 readers from another zone | Backend definitions in `bootstrap/`; `policy/c6\\\_state\\\_isolation.rego` | Plan check; backend access report on Day 16 and Day 22 |

| C7. No static cloud credentials; OIDC tokens last at most 1 h: 0 static keys | Pipeline definitions in `.github/` or `.gitlab/`; secret scanning | CI check and secret-scanning result on every pull request |

| C8. No change outside the pipeline, recorded break-glass excepted: 0 | Scheduled audit-log query in each account, alerting on any change by another principal | Alert history and weekly report from Day 16; each alert reviewed by the CoE security engineer |

| C9. Providers locked and module sources pinned to a tag or commit: 0 unpinned | `.terraform.lock.hcl` in each root; `policy/c9\\\_pinned\\\_sources.rego` | CI check on every pull request |

| C10. Build and Run use the same `modules/data-plane`, Run at most 1 release behind Build | `policy/c10\\\_data\\\_plane\\\_parity.rego` | Plan check on every pull request |

| C11. Every check above fails on a fixture plan that violates it: 100 % | `policy/fixtures/` | OPA unit tests (`opa test`) on every pull request |

| C12. No drift left unresolved after 2 working days: 0 | Scheduled plan of each root | Daily drift report from Day 16 |

| Bootstrap applied once, by two named people, from the control-plane account; its credentials then locked as break-glass | `bootstrap/`; break-glass record | Procedural, because no pipeline exists before the bootstrap; the CISO delegate holds the break-glass record and checks it at the risk and security review |



\## Impact Assessment



\### Zones and Boundaries



| Zone | Change |

|---|---|

| Shared control plane | New root `zones/control-plane/` with its own state backend, runner pool and pipeline identity; the bootstrap is applied from this account; publishes non-secret outputs for Build and Run |

| Build zone | New root `zones/build/` with its own state backend, runner pool and pipeline identity; instantiates `modules/data-plane` |

| Run zone | New root `zones/run/` with its own state backend, runner pool and pipeline identity; instantiates `modules/data-plane`; each apply waits for a second approver |

| Regional replica(s) | None now; a replica will get its own root, state backend and runners in its region (ADR-010) |



\* New or changed path between Build and Run: No

\* Credentials, keys or endpoints reachable from another zone after this change: None. Build and Run read the control plane's published non-secret outputs, as ADR-001 already allows; no zone holds another zone's credentials or reads its state.



\### Agents, Skills, MCP Servers and Models



Agent 9 (Build/CI, T2) may plan the Build root only. Agent 12 (IaC apply, T3) plans and applies the Run root, from the Run zone. No tier changes, skills bundles, MCP servers or model endpoints, and no T4 capability.



\### Security and Threat Model



Threat model v0.1 (Day 8), entries {IDs}: cross-zone credential leakage; supply chain of providers and modules. New assets: three state backends, three pipeline identities, three runner pools and the bootstrap credentials. New trust boundary: the identity federation between the source-control platform and each account. Abuse cases examined: cross-zone credential leakage (C1, C2, C4, C7); over-privileged automation (C2, C8); a compromised provider or module (C9); approval fatigue at the Run apply, limited because only Run applies need a second approver. Not applicable, because this change involves no agent tool calls, skills or model traffic: prompt injection through MCP tool outputs, poisoned skills bundle, sandbox escape, DLP evasion, over-privileged MCP scopes. Residual risk: a compromised Run runner can change Run, limited by ephemeral and patched runners, the second approver and C8. CISO delegate review: draft at the Day 8 risk and security review; {date} for this version.



\### Data Protection and Regulation



| Question | Answer |

|---|---|

| Data classes processed, stored or moved | Infrastructure metadata in state files and plans; state can hold secrets. No customer or personal data, apart from the identities of committers and approvers |

| Jurisdiction overlays affected | None |

| Residency, EU egress, cross-border transfer basis | State backends and runners in an EU region. Plans posted to pull requests are stored where the source-control platform is hosted: {hosting region}; secret values are redacted before posting |

| Regional replica required (ADR-010 trigger) | No |

| EU AI Act classification or transparency duty | None, because this ADR introduces no AI system; agents 9 and 12 keep their classification under Standard 11 |

| Sector regimes (PCI DSS, EU DORA, SOX) | Supports change-management evidence for production infrastructure: every Run change is a reviewed pull request with a second approver and an attributable apply (SOX ITGC; EU DORA ICT change management where the entity is a financial entity) |

| Developer telemetry and works-council constraints | Committer and approver identities are kept for attribution (driver 5), not for productivity measurement; to be raised in the works-council consultation opened on Day 15 |

| Retention of audit events, prompts and responses | Plans, pipeline logs and cloud audit logs follow Standard 9 once drafted (Day 47): {retention}. No prompts or responses |

| DPO and Legal consulted (who, date) | {who, date} |



\### Standards, Policy Packs and Registers



| Item | Change |

|---|---|

| Standards created or changed (number, target version) | None, because no standard covers IaC or source control. Standards 9 and 10, drafted on Day 47, will cover C1, C7 and C8; until then this ADR's Confirmation is the only yardstick |

| Policy packs: global baseline, jurisdiction overlay, BU overlay | None, because the checks in `policy/` belong to the CoE's IaC repository, not to policy-source (ADR-007) |

| Registers: agent registry, skills registry, MCP register, model allowlist | None |

| Exceptions expected under Standard 12 | None |



<!-- This is an optional element. Feel free to remove. -->

\### Rollout and BU Impact



No BU impact before Pilot A onboards (Day 29), and no certification level concerned. BU application IaC lives in BU repositories and is out of scope. Deadline: Day 16. Rollback: before Day 16, delete the skeleton; after Day 16, a change of structure is a new ADR that supersedes this one.



<!-- This is an optional element. Feel free to remove. -->

\## Pros and Cons of the Options



\### One repository, one root, state backend and pipeline identity per zone



Each zone has its own root module, state backend and pipeline identity, planned and applied on runners in its own account; shared modules are versioned. Evidence: HashiCorp recommends separate configurations along architectural boundaries, with common elements in reusable modules (\[CLI workspaces and alternatives](https://developer.hashicorp.com/terraform/cli/workspaces)).



\* Good, because credentials and state stay in their zone.

\* Good, because a change to the data-plane module is reviewed once and applied zone by zone.

\* Bad, because three runner pools.



\### One repository per zone, plus a module repository



The same isolation, split across four repositories. No PoC.



\* Good, because write access can differ per repository.

\* Neutral, because CODEOWNERS per root in one repository gives the same review separation.

\* Bad, because a cross-zone change becomes several coordinated pull requests, and module versions drift without one view.



\### One root with a CLI workspace per zone



One configuration and one backend, with a workspace per zone. Evidence: HashiCorp states that CLI workspaces are not appropriate for deployments requiring separate credentials and access controls (\[State workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces)).



\* Good, because there is one configuration to maintain.

\* Bad, because one backend and one identity span all zones, against ADR-003.



\### Per-zone roots on shared Build-zone runners



The first option's layout, but every root planned and applied from one runner pool in the Build zone.



\* Good, because there is one runner pool.

\* Bad, because Build runners would hold Run credentials: applying Run needs write access, and even planning Run needs read access. Rejected on driver 1 alone.



\## Decision Record



<!-- Kept by the DA secretary. One row per event; the discussion itself stays in the pull request.

After the Board, the secretary writes the outcome here (status, date, rows, conditions) on the pull request branch.

A DA code owner then approves the pull request, which records the decision, and the secretary merges.




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

\## More Information



ADR-001; ADR-002 and ADR-003 (Board #2, Day 11); threat model v0.1 (Day 8). Design review #1 (Day 22) measures C1–C12 on the provisioned zones; its record goes in `reviews/`. Reversibility: the layout is cheap to change until Day 16; state backends and identity trust are costly after Day 16; network topology is costly after Day 17. HashiCorp: \[State workspaces](https://developer.hashicorp.com/terraform/language/state/workspaces) · \[CLI workspaces and alternatives](https://developer.hashicorp.com/terraform/cli/workspaces).

