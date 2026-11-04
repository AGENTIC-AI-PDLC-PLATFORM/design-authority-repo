---
# ── MADR 4.0.0 keys (same meaning as in MADR) ─────────────────────────────
status: proposed                      # proposed | accepted | rejected | deprecated | superseded
date: YYYY-MM-DD                      # date of the last status change
decision-makers: [design-authority]   # [steering-committee] for pilot selection, wave plan, budget, GA, T4 agents
consulted: []                         # e.g. [coe-squad-a, bu-champions, dpo, legal, ciso-delegate]
informed: [steering-committee]

# ── Identification and ownership ──────────────────────────────────────────
id: ADR-NNN                           # next free number, never reused; matches decisions/NNN-title-with-dashes.md
owner: "{accountable person}"
proposer: "{CoE squad | BU | DA member}"
drafted-with: ""                      # "{agent}@{version}" when agent 2 (Architecture and ADR drafting) wrote the draft
rfc: ""                               # pull-request URL, added once the pull request is open

# ── Links ─────────────────────────────────────────────────────────────────
supersedes: []                        # e.g. [ADR-004]
superseded-by: []
related: []                           # other ADRs, standards, review records in reviews/ (design and post-promotion reviews)

# ── Scope tags (CI cross-checks, portal index, DA metrics) ─────────────────
zones: []                             # control-plane | build | run | regional-replica
jurisdictions: [global]               # global | eu | uk | us | br | ca | mx | cn | sg | jp | in | sa | ae | za | au | tr
                                      #   (same codes as policy/jurisdiction/ in the policy-source repo)
standards: []                         # 1 agent contract · 2 skills-file · 3 MCP server sanctioning · 4 LLM gateway policy
                                      # 5 DLP and tokenization · 6 policy-as-code · 7 promote path and human gate
                                      # 8 sandbox execution · 9 audit and evidence · 10 identity and mTLS
                                      # 11 risk tiering and EU AI Act · 12 exceptions
agents: []                            # 1–15 as numbered in the agent-role table (10 = human validation gate)
risk-tiers: []                        # T1 | T2 | T3 | T4
revisit: ""                           # date or trigger for re-examining the decision
---

# ADR-NNN: {short title, stating the problem solved and the option chosen}

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

{One sentence, Y-statement form: In the context of {situation}, facing {concern}, we decided for {chosen option} and against {rejected options}, to achieve {benefit}, accepting {cost or trade-off}.}

## Context and Problem Statement

{Two to six sentences: what forces the decision now; what it blocks and by when (for example "blocks Squad A's control-plane MVP"); the decision-backlog item; what is out of scope.}

## Decision Drivers

<!-- Ranked, most important first. Drivers that recur on this platform: isolation of the three zones and the single
Build → Run crossing (ADR-002, ADR-003); data residency and EU egress; fail-closed behaviour; auditability in the WORM log;
operability across 14 BUs and sovereign regions; time to the first gated promotion; cost; exit cost and lock-in. -->

1. {driver}
2. {driver}
3. {driver}

## Considered Options

<!-- At least two real options, including the current state when there is one.
Product names in the plan stay candidates until an ADR accepts one. -->

* {option 1}
* {option 2}
* {option 3}

## Decision Outcome

Chosen option: "{option}", because {justification against the ranked drivers}.

Confidence: {high | medium | low}, because {what the confidence rests on and what would change it}.

### Consequences

<!-- Cover operations (SLOs, on-call, DR targets RPO/RTO, capacity) and cost (infrastructure, licences, token spend),
not only design qualities. -->

* Good, because {positive consequence}
* Neutral, because {consequence}
* Bad, because {accepted trade-off}

### Confirmation

<!-- How compliance is enforced and proven. Prefer enforcement in code over procedure (the ADR-006 principle).
For a procedural control, say why and who checks it. -->

| Control | Location | Proof |
|---|---|---|
| {e.g. data-class rule evaluated by the PDP} | {policy path and policy-source pull request} | {unit test, CI check, audit event type in the WORM log} |

## Impact Assessment

### Zones and Boundaries

| Zone | Change |
|---|---|
| Shared control plane | {change, or "None"} |
| Build zone | {change, or "None"} |
| Run zone | {change, or "None"} |
| Regional replica(s) | {change, or "None"} |

* New or changed path between Build and Run: {No | Yes, through the promote path only (ADR-003); any other path needs an ADR that supersedes ADR-003}
* Credentials, keys or endpoints reachable from another zone after this change: {None | list}

### Agents, Skills, MCP Servers and Models

{Agents affected (1–15) and their risk tier; tier changes; skills bundles; MCP servers to sanction, re-scope or retire; model endpoints added to or removed from the allowlist. A T4 capability goes to the steering committee.}

### Security and Threat Model

{Threat-model entries affected (IDs, threat-model version); new assets or trust boundaries; abuse cases examined (prompt injection through MCP tool outputs, poisoned skills bundle, sandbox escape, DLP evasion by encoding or chunking, approval fatigue at the gate, cross-zone credential leakage, over-privileged MCP scopes); mitigations; residual risk; CISO delegate review.}

### Data Protection and Regulation

| Question | Answer |
|---|---|
| Data classes processed, stored or moved | |
| Jurisdiction overlays affected | |
| Residency, EU egress, cross-border transfer basis | |
| Regional replica required (ADR-010 trigger) | |
| EU AI Act classification or transparency duty | |
| Sector regimes (PCI DSS, EU DORA, SOX) | |
| Developer telemetry and works-council constraints | |
| Retention of audit events, prompts and responses | |
| DPO and Legal consulted (who, date) | |

### Standards, Policy Packs and Registers

| Item | Change |
|---|---|
| Standards created or changed (number, target version) | |
| Policy packs: global baseline, jurisdiction overlay, BU overlay | |
| Registers: agent registry, skills registry, MCP register, model allowlist | |
| Exceptions expected under Standard 12 | |

<!-- This is an optional element. Feel free to remove. -->
### Rollout and BU Impact

{Pilots A, B, C and waves 2–4 affected; certification level concerned (Bronze, Silver, Gold); work required per BU; deadline; rollback.}

<!-- This is an optional element. Feel free to remove. -->
## Pros and Cons of the Options

### {option 1}

{description, PoC result or benchmark, pointer to evidence}

* Good, because {argument}
* Neutral, because {argument}
* Bad, because {argument}

### {option 2}

{description, PoC result or benchmark, pointer to evidence}

* Good, because {argument}
* Neutral, because {argument}
* Bad, because {argument}

## Decision Record

<!-- Kept by the DA secretary. One row per event; the discussion itself stays in the pull request.
After the Board, the secretary writes the outcome here (status, date, rows, conditions) on the pull request branch.
A DA code owner then approves the pull request, which records the decision, and the secretary merges.
Nothing is pushed after the approval: any later push dismisses it. A returned RFC stays open, with its Board row added. -->

| Date | Event | By | Notes |
|---|---|---|---|
| {YYYY-MM-DD} | RFC ready for review | {owner} | Review window (2 working days, Mon–Fri CET) closes {YYYY-MM-DD HH:MM} |
| {YYYY-MM-DD} | DA Board #{n} | Design Authority | {accepted, rejected or returned} |

Conditions attached to the decision:

| # | Condition | Owner | Due | Status |
|---|---|---|---|---|
| 1 | {condition} | {owner} | {YYYY-MM-DD} | {open or met} |

Reservations recorded at the Board: {None, or name: reservation}

<!-- This is an optional element. Feel free to remove. -->
## More Information

{Design-review record, PoC report, threat-model and regulatory-matrix versions, steering-committee minutes, related ADRs.}
