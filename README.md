# Proposal: Advanced Azure DevOps Pull Request Review Agent

**Prepared for:** Technology Commission  
**Prepared by:** Actemiumsc365  
**Date:** 29 September 2026  
**Decision requested:** Approve a controlled pilot and provide one workstation or VM with a 12 GB GPU.

## Executive summary

This project proposes a **private, AI-assisted Pull Request (PR) reviewer and commenter** for Azure DevOps. It will go beyond a basic comment bot: the agent will inspect a change in the context of relevant code, tests, build results and repository rules; prioritize findings; and post concise, evidence-based comments on the relevant lines. It will help reviewers find defects and focus their time, **not replace human approval**.

The proposed pilot runs on **one machine**, not a GPU cluster. A workstation or GPU-enabled virtual machine with **32 GB RAM, 12 GB GPU VRAM and 1 TB storage** is a practical starting point for a quantized local code model, subject to testing against actual PR sizes. No claims of guaranteed vulnerability detection, compliance certification or financial return are made before evaluation.

## What makes this more than an ordinary PR bot

- **Context-aware review:** analyze changed lines alongside nearby functions, referenced types, selected callers, tests and repository guidance instead of sending an isolated diff to a model.
- **Hybrid evidence:** combine local AI reasoning with static analysis, secret scanning, dependency checks and available pipeline results. A model suggestion is not treated as a verified finding.
- **Actionable inline comments:** include the location, reason, evidence, severity and a suggested change when supported by the diff; group broader findings into a PR summary.
- **Noise control:** deduplicate related findings, ignore generated/vendor files by policy, avoid repeating resolved comments and publish only findings that can be tied to a real location or clear PR-level concern.
- **Human feedback loop:** track whether reviewers accept, dismiss or correct suggestions to improve prompts and rules; do not automatically train on proprietary code.
- **Privacy-first deployment:** inference can stay on the approved workstation or VM; cloud AI is optional and requires separate authorization.

## How it works

```text
Azure DevOps PR event
       → authenticated webhook / scheduled polling fallback
       → queued job and PR version check
       → retrieve changed files, relevant context and pipeline results
       → deterministic scanners + local quantized code model
       → validate locations, remove duplicates and rank findings
       → publish PR summary and selected inline comments
       → record audit metadata and reviewer feedback
```

The worker checks the PR's current version before publishing so comments do not refer to an outdated diff. Processing is bounded by file size, time and resource limits; large changes may receive a partial review with clear notice. The integration uses Azure DevOps REST APIs and an organization-approved identity with only the permissions required to read code and post comments. Posting an optional PR status check requires separate approval.

## Pilot capabilities

1. Summarize the purpose and scope of a PR, including changed areas and relevant tests.
2. Identify plausible correctness issues, missing edge cases and risky API or configuration changes, with citations to code where possible.
3. Surface security concerns from approved scanners and model-assisted review; escalate uncertain or high-risk findings for human verification.
4. Suggest focused tests and documentation updates rather than automatically editing code.
5. Present a short prioritized review, not a flood of low-confidence comments.
6. Show which feedback was useful through a simple review-feedback mechanism.

**Not in initial scope:** automatic approvals, merging, claims of zero-day discovery, compliance certification, model training or organization-wide rollout.

## Proposed technology

| Layer | Initial choice | Why |
|---|---|---|
| Azure DevOps integration | Service hooks + REST API | PR events, diffs and comment threads |
| Application | Python/FastAPI worker | Straightforward integration and deployment |
| Local AI | Ollama or another approved local inference runtime; quantized 7–8B code-capable model | Fits a 12 GB VRAM pilot in many configurations; benchmark model and context before selection |
| Code evidence | Language-specific parsers and existing linters/scanners | More reliable checks than model-only judgments |
| Job processing | Bounded local queue with retries | Handles bursts without losing events |
| Data | Minimal encrypted logs; SQLite or PostgreSQL if needed | Auditability without storing full source diffs indefinitely |
| Deployment | Docker on an approved Linux workstation/VM | Repeatable setup and updates |

The model is a candidate, **not a promise of accuracy**. Model size, context length, GPU allocation and throughput will be chosen after tests with real representative PRs. Existing enterprise security tooling should be integrated rather than replaced.

## Infrastructure requested

| Resource | Pilot request |
|---|---|
| Memory | **32 GB RAM** |
| GPU | **12 GB VRAM**, with supported local inference drivers (or comparable GPU-enabled VM) |
| CPU | Modern 8–12 core CPU preferred |
| Storage | **1 TB available**; SSD preferred for models and working data. A 1 TB HDD is usable but model loading, scanning and indexing will be slower; a smaller SSD for active files plus HDD for retention is an alternative. |
| Connectivity | Secure, approved access to Azure DevOps and a protected webhook endpoint, or polling if inbound access is not allowed |
| Operating environment | Approved Linux image, containers, disk encryption, monitoring and backup according to IT policy |

This is a **single-worker pilot** for one or two projects, not an organization-wide throughput guarantee. If a VPS is offered, it must include a real 12 GB GPU allocation for local GPU inference; an ordinary CPU-only VPS is a different, slower option. If budget is limited, start with rules plus CPU inference and measure performance before buying hardware.

## Security and governance

- Obtain permission to access each pilot repository and process its source code; follow data classification and retention rules.
- Use an approved service identity and least-privilege credentials kept in a secret manager; rotate and audit access.
- Authenticate webhook requests, apply network restrictions where feasible, and encrypt transport and stored data.
- Treat source code, PR comments and work items as **untrusted input** to the model: prevent prompt injection from causing unauthorized API actions or data exfiltration.
- Restrict outbound network access; keep source content out of external AI services unless explicitly approved.
- Log decisions and posting actions, redact secrets, and set a short retention period for prompts/diffs.
- Run in **comment-only mode** with human reviewers responsible for final approval and merge. An incident response contact and kill switch will be defined before pilot launch.

## Delivery plan and decision gates

**Phase 1 — Design and baseline (weeks 1–2):** Confirm pilot repositories, security approvals and hardware; collect baseline PR volume, review time and feedback quality. Define allowed files, scopes and retention.

**Phase 2 — Working pilot (weeks 3–6):** Implement event handling, context retrieval, local inference, scanners, inline commenting, retry/deduplication and audit logs. Test on sample PRs before enabling live comments.

**Phase 3 — Evaluation (weeks 7–10):** Run with one or two teams, tune thresholds, review false positives and measure latency and resource use. Keep human review mandatory.

**Go/no-go decision:** Expand only if reviewers find comments useful, data handling passes security review and throughput is acceptable on the provided machine. Otherwise refine the approach or stop without further infrastructure commitments.

## How success will be measured

Compare pilot data with the pre-pilot baseline: median time to first useful feedback; percentage of PRs reviewed successfully; suggestion acceptance and false-positive rates based on human labels; number of missed or incorrect high-severity comments; GPU memory use and end-to-end processing time; operational incidents. Targets will be agreed with the pilot teams **after establishing a baseline**, not asserted as guaranteed savings.

## Approval request

Please approve a **10-week pilot**, access to one or two Azure DevOps projects, an approved service identity and security review, and **one workstation or GPU-enabled VM with 32 GB RAM, 12 GB VRAM and 1 TB storage** (SSD preferred for active data). The pilot will produce a working reviewer, operating documentation and measured results for an informed decision on wider deployment.
