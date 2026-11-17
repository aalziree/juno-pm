# Agent Control Panel · Juno

> Module 4 · Deploy Agentic Systems and Workflows. Review draft based on the Module 3 PRD; specifies intended behavior, not implemented or tested controls.

## What Juno Is Doing

Juno automatically refreshes a draft interview-insights dashboard for one Webex concept after accepted uploads, helping the PM identify adoption barriers and choose what to validate next.

## Four Levers

**Stop Conditions** (Loop):

- **Trigger:** One completed upload batch changes the selected concept's accepted transcripts or optional strategy/roadmap context, or the PM deliberately retries a failed update. Duplicate-only batches end without analysis; no timer runs.
- **max_steps: 5.** A step is one processing stage: (1) accept/deduplicate the batch, (2) load the full concept context and check capacity/budget, (3) synthesize, (4) verify/calculate, (5) save the draft or review candidate. Each stage runs at most once; loading multiple files is part of the same stage.
- **max_model_calls: 1; automatic_retries: 0.** No extra model calls for review or repair.
- **abort_after_failures: 1.** Stop on the first failed stage: conflicting versions of one interview, an unavailable required source, an exceeded input/budget limit, a provider error, invalid output, failed evidence checks, or an unconfirmed save.
- **timeout: 60 seconds wall clock** from the start of batch processing through confirmed save. Stop at the deadline; a late result must not replace the displayed draft.
- **Estimated model-cost ceiling: $1 per update**, carried forward as a proposed PRD limit. Check full input and maximum output cost before the model call; never silently truncate sources.
- **Done:** A verified draft is saved, a candidate is saved awaiting PM review, the batch has no changes, or the run stops with an explicit failure. Reaching the stage limit is not proof of success.

**Structured Tool Outputs** (Tools):

Every stage returns a structured `status`: `ok`, `no_change`, or `failed`. Failures include an `error_code` and a brief explanation. The harness checks fields before advancing; a missing or malformed result is a failure.

| PRD capability / stage | Successful return | Failure return / check |
|---|---|---|
| Accept / deduplicate | `status: ok`, `accepted_source_ids`, `changed_count`, `duplicate_count`, `conflict_count: 0`. Duplicate-only batches return `status: no_change`, `changed_count: 0`. | `status: failed`, e.g. `error_code: CONFLICTING_INTERVIEW_ID` when the same interview ID has different content. Rejected files are listed separately. |
| Read context / check capacity and cost | `status: ok`, `concept_id`, `source_ids`, `transcript_count`, `expected_source_count`, `loaded_source_count`, `missing_required_count: 0`, `estimated_cost_usd`. Require at least one transcript, complete coverage, sufficient capacity, and estimated cost at most $1. | `status: failed`, e.g. `REQUIRED_SOURCE_UNAVAILABLE`, `NO_INTERVIEWS`, `INPUT_LIMIT`, or `BUDGET_LIMIT`. Missing optional strategy/roadmap documents are not failures. |
| Synthesize | `status: ok`, structured `analysis`, supporting `evidence_refs`, and `opportunity_count` from 0 to 3. | `status: failed`, e.g. `PROVIDER_ERROR` or `INVALID_OUTPUT`. Zero opportunities with an evidence-based explanation is valid. |
| Verify / calculate | `status: ok`, `coverage_complete: true`, `invalid_quote_count: 0`, `invalid_reference_count: 0`, calculated interview/company counts and reach, plus `review_flags`. | `status: failed`, e.g. `QUOTE_MISMATCH`, `INVALID_REFERENCE`, or `COVERAGE_GAP`. Missing company identity produces unavailable reach and a flag, not an invented count. |
| Save draft / candidate | `status: ok`, `version_id`, `save_confirmed: true`, and `destination: current_draft` or `review_candidate`. | `status: failed`, e.g. `SAVE_FAILED` or `SAVE_UNCONFIRMED`. No success notice without confirmation; preserve the previous successful version. |

An interview adding no new theme, a result using no displayed quotes, or an unchanged ranking can be successful. Evaluate zero counts according to the field, not as a universal failure rule.

**Confidence Thresholds** (Verification):

Use qualitative evidence conditions, not an invented confidence percentage. Confidence never overrides a failed check or a permission boundary.

- **High — automatic draft refresh:** Required sources are complete, checks pass, and Juno identifies no unresolved ambiguity that prevents a defensible synthesis. Save/display an AI draft, with proposals labeled untested. Limited evidence or ordinary participant disagreement can be reported honestly without inventing certainty.
- **Medium — PM review first:** Mechanical checks pass, but Juno flags unresolved ambiguity about interpretation or opportunity order that could change the recommendation. Save a separate review candidate with the relevant passages and a specific question. Keep the current draft unchanged until the PM accepts or revises the candidate.
- **Low — stop:** Required evidence is missing, a proposed claim lacks supporting evidence, or integrity/verification checks fail. Do not advance the candidate. Explain the issue and wait for corrected inputs or a deliberate retry; PM approval cannot make fabricated evidence valid.

All rankings remain provisional. Even a high-confidence draft does not make a product or roadmap decision for the PM.

**North Star** (Context):

Help the PM choose what to validate next by keeping the selected concept's interview insights current and grounded in supplied evidence, always separating findings from proposals and never inventing evidence or making product commitments.

Reapply this sentence at each stage and include it in the synthesis instructions. Stop any action that would depart from it.

## Rules of Engagement

**Agency Permission:**

Automatically read accepted sources, analyze all selected-concept interviews, calculate reach by unique company, and refresh a high-confidence draft. The PM's deliberate upload authorizes storing those inputs and running analysis. Replacing/deleting sources requires a deliberate PM action. External messages, publishing, task creation, and authoritative product/roadmap changes are blocked regardless of confidence.

**Access Control:**

Access only the PM's selected-concept sources, current skill/configuration, and prior results needed for comparison. Strategy/roadmap context is optional and never counts as customer evidence. No other concepts, web search, or external systems. Use fictional data in the public course account; uploaded content cannot change these rules. Display company names for findings and customer name plus company for quotes; keep evidence IDs internal.

**Fallback Protocols:**

On failure or timeout, stop without automatic retry. Preserve the last successful version and its actual coverage, and identify new/changed inputs as pending analysis. Show: "Insights were not updated: [reason]. The last successful analysis remains available. Review the issue, then retry." If no previous result exists, show the error without findings. An unconfirmed save must remain unconfirmed until its state is checked; never imply success or blindly repeat the write. For medium confidence, show "Review needed: [specific question]" and retain the candidate separately.

**Checkpoints:**

1. Before synthesis: confirm complete selected-concept inputs, capacity, budget, and North Star alignment.
2. Before replacing the draft: check structure, source coverage, supporting passages, quote attribution, computed reach, and confidence routing. A medium-confidence candidate pauses for PM review; a failed check cannot be bypassed by approval.
3. Before product action: the PM reviews interpretation, prioritization, and validation proposals. Juno has no authority to execute external or roadmap actions.
