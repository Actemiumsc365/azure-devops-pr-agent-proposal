# Proposal: Azure DevOps Pull Request Review Agent

**Prepared for:** High Commission Review  
**Prepared by:** Actemiumsc365  
**Date:** 29 September 2026  
**Status:** Request for approval and infrastructure support

## 1. Executive Summary

This proposal requests approval and infrastructure support to develop a secure, AI-assisted Pull Request review agent for Azure DevOps. The agent will analyze code changes, identify quality and security risks, summarize changes, and provide actionable suggestions directly on Azure DevOps Pull Requests.

The system will begin in **comment-only mode**: human developers and reviewers remain responsible for final decisions. It can use a local AI model, such as Ollama, so source code can remain inside the approved environment and operating costs can be controlled.

The requested support is either:

- A dedicated high-performance workstation/server; or
- A private virtual machine/VPS in an approved cloud or internal environment.

## 2. Problem and Opportunity

Manual Pull Request reviews can be delayed by reviewer availability and may produce inconsistent coverage. Reviewers also spend time identifying repetitive issues such as missing tests, insecure patterns, poor error handling, and documentation gaps.

The proposed agent will provide an initial automated review within minutes, allowing human reviewers to focus on architecture, business logic, and high-risk decisions. It is intended to assist—not replace—professional developers or security reviewers.

## 3. Proposed Solution

The agent will:

1. Receive Azure DevOps Pull Request events through a webhook.
2. Retrieve approved metadata, changed files, and diffs using Azure DevOps APIs.
3. Run deterministic checks for secrets, unsafe patterns, formatting, tests, and policy rules.
4. Send an appropriately limited diff to a local or approved AI model.
5. Produce a review summary and prioritized suggestions.
6. Post comments or status updates back to the Pull Request.
7. Record audit information without retaining source code unnecessarily.

### Initial capabilities

- Pull Request summary
- Potential bug and null/error-handling suggestions
- Security-pattern warnings
- Hard-coded secret detection
- Test and documentation suggestions
- Coding-standard checks
- Work-item and policy validation
- Optional build and test-result summary

## 4. Architecture

```text
Azure DevOps Pull Request
          |
          v
   Secure Webhook Receiver
          |
          v
  PR Agent and Rule Engine
      |             |
      v             v
Static/security   Local LLM
checks            (Ollama)
      \             /
       v           v
       Review Result and Policy Filter
                    |
                    v
        Azure DevOps Comments/Status
```

### Technology options

| Component | Proposed options |
|---|---|
| Service | Python/FastAPI or .NET |
| Model | Ollama with a code-capable quantized model; approved cloud model as fallback |
| Storage | PostgreSQL or encrypted local storage for audit metadata |
| Deployment | Docker on Linux VM or dedicated server |
| Monitoring | Application logs, health checks, Prometheus/Grafana if required |
| Integration | Azure DevOps REST APIs and webhooks |

## 5. Infrastructure Request

### Pilot minimum

- 8 CPU cores
- 16 GB RAM
- 500 GB SSD
- Ubuntu LTS or approved server operating system
- Reliable internal network access to Azure DevOps
- Docker support

### Recommended production configuration

- 16–32 CPU cores
- 32–64 GB RAM
- 1 TB encrypted SSD
- Optional NVIDIA GPU with sufficient VRAM for faster local model inference
- Private network placement and restricted inbound access
- Automated backups and monitoring

A GPU is helpful but not essential for the pilot. The rule engine and smaller local models can operate on CPU, although response time may be slower.

## 6. Security and Governance

Security is a primary design requirement:

- Start with comment-only permissions; do not automatically approve, reject, or merge.
- Use a dedicated Azure DevOps service identity with least-privilege access.
- Store tokens in a secret manager or protected environment variables.
- Restrict webhook access and validate signatures/secrets.
- Do not send source code to external services without explicit approval.
- Prefer local inference for confidential repositories.
- Encrypt disks, backups, and network traffic.
- Apply retention limits to diffs, prompts, and model responses.
- Keep an audit trail of agent actions and human decisions.
- Require human approval for all merge and release decisions.

## 7. Implementation Roadmap

### Phase 1 — Proof of concept (4–6 weeks)

- Provision a VM or workstation.
- Configure Azure DevOps webhook and service identity.
- Implement PR retrieval and comment posting.
- Add deterministic security and quality rules.
- Test against one non-critical project.

### Phase 2 — Assisted AI review (4–8 weeks)

- Add a local code-capable model.
- Introduce prompt templates and output validation.
- Add line-specific suggestions where supported.
- Collect reviewer feedback and measure false positives.

### Phase 3 — Controlled production pilot (4–8 weeks)

- Add monitoring, audit logs, backups, and operational documentation.
- Onboard selected teams.
- Establish security review and incident procedures.
- Tune rules and model usage based on measured results.

### Phase 4 — Scale and optimize

- Support multiple projects and queues.
- Add status checks and dashboards only after approval.
- Evaluate GPU or cloud scaling based on actual workload.

## 8. Expected Benefits and Measures

The pilot will use measured results rather than unsupported assumptions. Suggested targets are:

| Measure | Initial target |
|---|---:|
| Automated review availability | 95%+ of pilot PRs |
| Initial feedback time | Under 10 minutes |
| Availability | 99% during pilot hours |
| Actionable suggestions | At least 60% of accepted feedback |
| False-positive rate | Decrease each review cycle |
| Manual review effort | 20–30% reduction for repetitive checks |
| Security incidents caused by agent | Zero |

Results will be reviewed after the pilot before requesting wider deployment.

## 9. Cost and Resource Considerations

### Option A: Existing or dedicated hardware

Advantages include data control, predictable operating cost, and no per-token model charges. The initial request is for a capable workstation or server that can be reused for development, testing, and approved internal automation.

### Option B: Private VPS or cloud VM

Advantages include faster provisioning, snapshots, and easier remote access. The exact cost depends on provider, region, storage, network, and whether a GPU is selected. A CPU-only pilot should be used first; GPU capacity should be justified by measured workload.

Open-source software can be used for the pilot. The main ongoing costs are infrastructure, maintenance, security review, backups, and engineering time.

## 10. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Incorrect AI suggestion | Human review, confidence thresholds, comment-only mode |
| Source-code exposure | Local model, network controls, approved retention policy |
| Excessive comments | Prioritization, duplicate suppression, reviewer feedback |
| Credential compromise | Least privilege, secret storage, rotation, audit logs |
| Model latency | Rule checks first, queues, smaller models, optional GPU |
| Low adoption | Pilot with volunteers and publish measured results |
| Operational failure | Health checks, retry queues, backups, runbooks |

## 11. Approval Requested

Approval is requested for:

1. A 3–6 month controlled pilot.
2. One dedicated workstation/server or private VPS meeting the pilot specification.
3. Azure DevOps webhook and least-privilege service-account access.
4. Time from a developer/DevOps resource to implement and evaluate the pilot.
5. A security and governance review before production expansion.

## 12. Conclusion

An Azure DevOps Pull Request review agent is technically feasible without requiring a large AI platform. A controlled local deployment can provide useful automated feedback while keeping code within the approved environment. The phased approach limits cost and risk, produces measurable evidence, and allows the High Commission to decide whether further investment is justified.

**Decision requested:** Approve the pilot and provision either the recommended local server/workstation or an approved private VPS.
