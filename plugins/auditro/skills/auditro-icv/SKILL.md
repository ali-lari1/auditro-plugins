---
name: auditro-icv
description: Use for any question about UAE ICV (In-Country Value) scores, evidence, certificates, action plans or rulesets for an organization connected to Auditro through its MCP server.
---

# Auditro ICV

Auditro is a remote MCP server for UAE ICV compliance. This skill covers how to use its tools safely. Every tool is listed in [reference.md](reference.md).

## Connect

- Endpoint: `https://app.auditro.co/mcp` (streamable HTTP, OAuth 2.1 with PKCE). Never paste a permanent token.
- Claude Code: `claude mcp add --transport http --scope user auditro https://app.auditro.co/mcp`, then `claude mcp login auditro`
- Codex: `codex mcp add auditro --url https://app.auditro.co/mcp`, then `codex mcp login auditro`
- Cursor: add `{"mcpServers": {"auditro": {"url": "https://app.auditro.co/mcp"}}}` to `.cursor/mcp.json`, then `cursor-agent mcp enable auditro` and `cursor-agent mcp login auditro`
- ChatGPT: enable developer mode, then Apps & Connectors, Create, MCP server URL `https://app.auditro.co/mcp`, Authentication OAuth

## The invariant

Auditro computes every number. The agent only selects governed candidate IDs. Never restate, round, estimate or recompute a score, point impact or cost: quote the figure the tool returned, with its ruleset and source cutoff. Auditro does not certify scores and does not guarantee tender outcomes.

## Discovery loop

1. `get_workspace` first, always. It returns the organization, your role and scopes, entity IDs, period IDs and each period's `calculationRuleSet`. Use only IDs returned here.
2. `get_icv_position` for the latest governed score, components, ruleset and completeness.
3. `list_action_candidates` for candidate actions persisted with the score (may be empty; use `get_action_plan` for governed plans).
4. `get_action_plan` to read the governed plan the team proposed in the Auditro app. Proposing is web-only.
5. `decide_action_plan` to approve, reject or request changes (needs plan approval permission and confirmed score inputs), then `update_action_progress` to record progress on an accepted action. Re-read with `get_action_plan`.

Side tools:

- `explain_score` for lineage: which components, sources and rules produced a figure.
- `get_evidence_gaps` before promising or implying a certificate outcome.
- `get_sensitivity_scenarios`, `list_rulesets`, `compare_rulesets` and `get_programme_architecture` for what-if and rule questions.

## Write hygiene

- Supply a fresh `idempotencyKey` on every write. Reuse the same key only when retrying the same request.
- Pass `expectedUpdatedAt` from the latest read on updates. On `STALE_VERSION`, re-read and ask the user before reapplying.
- After every write, re-read the record and report what it now says, not what you sent.
- Writes need full access. Confirm the intent with the user before any write.

## Long operations

`recalculate_icv`, `generate_evidence_pack`, `start_odoo_sync` and `commit_csv_import` may queue an operation. Poll `get_operation` until it completes or fails; back off between polls. Cancel or retry through the operation's offered controls rather than starting a duplicate. For `recalculate_icv`, use the period's `calculationRuleSet.id` from `get_workspace`.

## Errors

| Code | What to do |
| --- | --- |
| `VALIDATION_FAILED` | Fix the argument named in the message; do not guess values. |
| `NOT_FOUND` | Re-fetch IDs from `get_workspace`; do not invent IDs. |
| `UNSUPPORTED_RULESET` | Non-National regime. Stop and tell the user the regime is not supported for that tool. |
| `FORBIDDEN` | The connection or role lacks scope. Tell the user; do not retry. |
| `CONFLICT` | State changed or inputs do not qualify. Re-read, explain, ask. |
| `STALE_VERSION` | Re-read for the new `updatedAt`, then confirm before reapplying. |
| `FILE_TOO_LARGE` | Use `prepare_file_upload` instead of inline bytes. |
| `RATE_LIMITED` | Wait and retry once with backoff. |
| `DEPENDENCY_UNAVAILABLE` | Retry later; report the outage plainly. |

## Do not

- Do not put production customer data in examples, tickets or commit messages.
- Do not invent evidence, certificates, entity IDs or score figures.
- Do not describe `draft` rulesets as final or Dubai ICV as having a numeric score.
- Do not claim a certificate outcome; point to `get_evidence_gaps` and the certifying body.
