---
name: midscene-report-analysis
description: |
  Analyze complete Midscene report URLs or local HTML files to validate recorded
  pass/fail results, identify false-pass and false-fail results, diagnose failures
  and incomplete executions, and trace evidenced root causes.

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# Analyze Midscene Reports

## Workflow

### 1. Inspect the report

Require a complete report: an HTTP(S) URL, absolute HTML path, or directory containing `index.html`. Reject screenshots, summaries, or isolated records.

Inspect before analysis:

~~~bash
npx -y @midscene/cli@^1.10.13 report-tool --action inspect --htmlPath /absolute/path/report.html [--outputDir <directory>]
~~~

Continue only when inspect returns `report`, `reportStatus`, `localReport`, and `markdownFiles`.

### 2. Load and apply the analysis rules

After inspect establishes `reportStatus`, read
[cause-taxonomy.md](references/cause-taxonomy.md) completely before reviewing
evidence for assessment. Apply the matching result rules below to guide evidence
collection, but make the assessment and attribution only after full report review.

#### Recorded failure (`fail`)

| Result assessment | Use when |
| --- | --- |
| `true_fail` | At least one required condition is proven unmet. |
| `false_fail` | Every required condition is directly proven met despite the recorded failure. |
| `unverifiable` | Evidence is insufficient to determine whether the conditions were met. |
| `inconclusive` | Reliable evidence conflicts or remains inherently ambiguous. |

For `true_fail`, identify why and where the task failed and attribute the actual
failure. Attribute the erroneous failure record for `false_fail`, or the evidence
gap or conflict for `unverifiable` and `inconclusive`.

#### Recorded pass (`pass`)

| Result assessment | Use when |
| --- | --- |
| `true_pass` | Every required condition is supported. |
| `false_pass` | At least one required condition is proven unmet. |
| `unverifiable` | Evidence is insufficient for at least one required condition. |
| `inconclusive` | Reliable evidence conflicts or remains inherently ambiguous. |

For `true_pass`, attribute any evidenced recovered issue when present; otherwise
follow the taxonomy's clean-pass rule. Attribute the incorrect pass for
`false_pass`, the evidence gap for `unverifiable`, or the evidence conflict for
`inconclusive`.

#### Incomplete execution (`incomplete`)

Analyze `observedIssue` and `interruptionReason` independently. Do not call the
report passed or failed.

- In `observedIssue`, describe an evidenced internal issue that materially delayed,
  derailed, or prevented progress. State when none was observed, bound uncertainty
  without guessing, and include a recovered issue only when it changed the execution
  path.
- In `interruptionReason`, explain why the final task remained non-terminal. State a
  specific cause only when established by the report; otherwise state that the exact
  cause is not recorded. Do not reuse an earlier issue unless it remained active or
  propagated to the last recorded step.
- Attribute only the observed issue or a directly linked interruption. A lone
  non-terminal status is not a cause. If the observed issue supports a concrete
  category but the interruption does not, keep that category and state the
  interruption uncertainty only in `interruptionReason`.

### 3. Review the complete report

Read every file in `markdownFiles` to EOF and review all tasks and actions chronologically. Keep one internal evidence ledger keyed by task, step, and time; do not emit it.

- **Every step:** Inspect the instruction, model narration, executed action, machine result, and screenshots; track abnormalities and recovery.
- **Machine errors:** Preserve the exact error, owning task, code and stack when present, retries, and state transitions. Collapse wrapper duplicates; mark recovery only when the report proves it.
- **Visible page errors:** Record the exact message, surrounding state, and screenshot. Do not equate UI messages with machine errors or infer ownership without evidence.
- **Direct evidence:** Use model text only as evidence of the model's claims, intent, or decisions, never of page, application, or machine state. Verify external facts from actions, machine results, and screenshots; reject claims broader than the evidence.
- **Full coverage:** Do not sample or skip any step, including repetitive, successful, recovered, or apparently unrelated ones. Visually inspect every screenshot link, including repeats, and use `localReport` to resolve omissions. Before Step 4, report files-read/total and screenshot-links-viewed/total; proceed only at full coverage. Search, parsing, thumbnails, and contact sheets may assist but not replace review.

### 4. Determine the assessment and attribution

**Define the result conditions.** Derive every required condition and the judgment point from the governing test instruction. For each condition, record support, contradiction, and one of `met`, `unmet`, `unverifiable`, or `conflicting`. Treat earlier or parallel executions only as context or prerequisite evidence, never as separate outcomes. Do not infer a condition from the recorded status.

**Test competing explanations.** Maintain a compact register grouped by mechanism, not wording. For each plausible cause, record support, contradiction, the affected condition, recovery or propagation, and one state:

- `supported`: report evidence establishes the mechanism;
- `refuted`: report evidence contradicts it;
- `blocked`: deciding it requires a specific causal link absent from the complete report;
- `unsupported`: it remains merely possible but has no positive report evidence.

Stop a `blocked` route unless a new report fact supplies the missing link. Merge equivalent mechanisms, then apply the taxonomy selection rules.

**Assess causality and recovery.** For each abnormality, establish `observed fact -> affected condition or path -> recovery or propagation -> state at the judgment point`, then classify it as causal, contributory, unrelated, or recovered. Treat timeouts, exceptions, visible errors, and model claims as symptoms unless this chain establishes causality. Finalize `resultAssessment` for a fail or pass report, or the incomplete findings otherwise, then determine cause attribution separately under the taxonomy.

### 5. Validate the conclusion

**Run an adversarial audit.** Try to disprove the provisional assessment and attribution:

- For `true_fail` or `false_pass`, seek proof that each allegedly unmet condition was met at the judgment point.
- For `false_fail` or `true_pass`, seek any required condition that lacks direct support or is contradicted.
- For `unverifiable`, seek overlooked direct evidence; for `inconclusive`, confirm that reliable conflict cannot be ordered or reconciled.
- For `incomplete`, recheck `observedIssue` and `interruptionReason` separately and ensure any reused earlier issue remained active or propagated to the final step.
- For every cause category, check recovery, a closer owner, contrary evidence, and dependence on model narration, then reapply the taxonomy's selection and confidence rules.

**Confirm evidence traceability.** Before writing, verify each material claim against the ledger, governing instruction, actions and results, relevant screenshots, state transitions, and recovery. Keep model narration separate and do not emit a certificate. Preserve uncertainty when a condition or causal link remains unverified.

### 6. Create and render the result

After completing steps 3–5, call `analysis-template` to obtain the report's `analysisResultPath` and `schema`.

For a fail or pass report, pass the finalized assessment:

~~~bash
npx -y @midscene/cli@^1.10.13 report-tool --action analysis-template --reportStatus <fail|pass> --resultAssessment <resultAssessment> --htmlPath <localReport> [--outputDir <inspect-artifact-directory>]
~~~

For an incomplete report, omit `--resultAssessment`:

~~~bash
npx -y @midscene/cli@^1.10.13 report-tool --action analysis-template --reportStatus incomplete --htmlPath <localReport> [--outputDir <inspect-artifact-directory>]
~~~

Create the result JSON at `analysisResultPath` exactly as specified by `schema`. Read from `localReport`, but set the result's `report` to inspect's `report`.

Render the completed result JSON:

~~~bash
npx -y @midscene/cli@^1.10.13 report-tool --action render-analysis --analysisResultPath <analysisResultPath>
~~~

Return the renderer output unchanged. Do not add an analysis-JSON link, edit the input report, or change unrelated external state.

## Multiple reports

For multiple reports, run the full workflow independently per report. When subagents are available, assign reports in parallel and require each agent to provide its renderer-generated Markdown and absolute path.

After all per-report results are rendered, the primary agent creates one compact aggregate summary containing:

1. total report count, plus counts by `reportStatus` and applicable `resultAssessment`;
2. the main cross-report findings, grouped by recurring issue;
3. one concise row per report with its sequence number, status, assessment (or `incomplete`), all `causeCategories` as category/confidence pairs, one-sentence conclusion, a clickable original URL from inspect's `report` field, and a clickable absolute link to the renderer-generated Markdown.

Retain every renderer-generated Markdown file and absolute path. Save the same summary to `<batch-artifact-directory>/batch-analysis-summary.md`. Use the user-specified output directory as `batch-artifact-directory`; otherwise create one task-scoped local directory shared by the batch. Do not overwrite an unrelated file. Return the summary with a clickable link to the saved Markdown.
