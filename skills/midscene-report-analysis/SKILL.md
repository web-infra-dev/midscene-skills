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

### 1. Read report metadata

Require a complete report: an HTTP(S) URL, absolute HTML path, or directory
containing `index.html`. Reject screenshots, summaries, or isolated records.

~~~bash
npx -y @midscene/cli@^1.12.6 report-tool --action metadata --report "<report-url-or-path>"
~~~

Require `schemaVersion: 1` and `resolvedHtmlPath`. Treat `executionStatus` as the
recorded status and verify the actual task outcome against report evidence.

### 2. Export the report evidence

Create a new task directory under the specified output directory (or locally
by default), adding a numeric suffix on name conflicts. Export Markdown there:

~~~bash
npx -y @midscene/cli@^1.12.6 report-tool --action to-markdown --htmlPath "<resolvedHtmlPath>" --outputDir "<artifact-directory>"
~~~

### 3. Load and apply the analysis rules

Read [cause-taxonomy.md](references/cause-taxonomy.md) completely before
reviewing evidence. Use these rules to guide review; assess the result and
attribute causes only after reviewing the complete report.

#### Recorded pass or failure

| Evidence at the judgment point | Recorded `pass` | Recorded `fail` |
| --- | --- | --- |
| Every required condition is supported. | `true_pass` | `false_fail` |
| At least one required condition is proven unmet. | `false_pass` | `true_fail` |
| Evidence is insufficient to determine whether required conditions were met. | `unverifiable` | `unverifiable` |
| Reliable evidence conflicts or remains inherently ambiguous. | `inconclusive` | `inconclusive` |

A `false_fail` requires direct proof that every required condition was met.
Attribute the actual failure for `true_fail`, the erroneous recorded result for
`false_pass` or `false_fail`, and the evidence gap or conflict for `unverifiable`
or `inconclusive`. For `true_pass`, attribute evidenced recovered issues;
otherwise follow the taxonomy's clean-pass rule.

#### Incomplete execution (`incomplete`)

Analyze `observedIssue` and `interruptionReason` independently. Do not call the
report passed or failed.

- `observedIssue`: Describe an evidenced internal issue that materially affected
  progress, or state that none was observed. Preserve uncertainty and include
  recovered issues only when they changed the execution path.
- `interruptionReason`: Explain why the final task remained non-terminal, or
  state that the exact cause is not recorded. Link an earlier issue only if it
  remained active or propagated to the final step.

Attribute only the observed issue or a directly linked interruption; a
non-terminal status alone is not a cause. Keep a supported issue category even
when the interruption is unexplained, confining that uncertainty to
`interruptionReason`.

### 4. Review the complete report

Read every exported Markdown file to EOF and review all tasks and actions
chronologically: instruction, model narration, action, machine result, and
screenshots. Keep internal evidence notes keyed by task, step, and time.

- **Machine errors:** Preserve the exact error, owning task, code and stack when
  present, retries, and state transitions. Collapse wrapper duplicates.
- **Visible page errors:** Record the exact message, surrounding state, and
  screenshot; distinguish UI messages from machine errors and establish
  ownership from evidence.
- **Direct evidence:** Model narration establishes only the model's claims,
  intent, or decisions. Verify page, application, and machine state from
  actions, machine results, and screenshots.
- **Full coverage:** Do not sample or skip steps. Visually inspect every
  screenshot link, including repeats, and use `resolvedHtmlPath` to resolve
  omissions. Before Step 5, report files-read/total and
  screenshot-links-viewed/total; proceed only at full coverage. Search, parsing,
  thumbnails, and contact sheets may assist but not replace review.

### 5. Assess and validate the conclusion

Derive the required conditions and judgment point from the governing test
instruction, never from the recorded status. Record each condition as `met`,
`unmet`, `unverifiable`, or `conflicting`, with supporting and contradicting
evidence. Treat earlier or parallel executions as context or prerequisite
evidence, not separate outcomes.

Compare explanations by mechanism and merge equivalent ones. Stop an
explanation with a missing causal link unless new evidence supplies it. Trace
`observed fact -> affected condition or path -> recovery or propagation ->
state at the judgment point` to distinguish causal, contributory, unrelated,
and recovered issues. Mark recovery only when the report proves it; timeouts,
exceptions, UI errors, and model claims alone do not establish causes.

Choose the provisional assessment under Step 3 and attribute causes separately
using the taxonomy. Then try to disprove the conclusion:

- For `true_fail` or `false_pass`, seek evidence that the allegedly unmet
  condition was met at the judgment point.
- For `false_fail` or `true_pass`, seek missing or contrary evidence for any
  required condition.
- For `unverifiable`, seek overlooked evidence; for `inconclusive`, check
  whether the conflict can be ordered or reconciled.
- For `incomplete`, check the observed issue and interruption independently
  against the final recorded state.
- For each cause, check counterevidence, recovery, and a closer owner; apply
  the taxonomy's selection and confidence rules.

Finalize only claims traceable to report evidence, preserving uncertainty in
unverified conditions or causal links. Do not emit internal notes or an audit
certificate.

### 6. Write the final analysis directly

Read [analysis-template.md](references/analysis-template.md) completely and use
its common structure with the findings section matching `executionStatus`.
Save it to `<artifact-directory>/analysis-result.md`, choosing the next numeric
suffix if the file already exists.

Return the final Markdown and a clickable absolute path. Do not edit the input
report or change unrelated external state.

## Multiple reports

For multiple reports, first create a fresh, uniquely named batch artifact
directory using the parent-directory and numeric-suffix rules from Step 2. Run
the full workflow independently per report and use a separate subdirectory for
each report's exported evidence and final analysis.

After all per-report analyses are complete, create one compact aggregate summary
containing:

1. total report count, plus counts by `executionStatus` and applicable
   `resultAssessment`;
2. the main cross-report findings, grouped by recurring issue;
3. one concise row per report with its sequence number, execution status,
   assessment (or `incomplete`), all cause categories as category/confidence
   pairs, one-sentence conclusion, a clickable original `source`, and a
   clickable absolute link to its analysis Markdown.

Save the same summary to `<batch-artifact-directory>/batch-analysis-summary.md`.
Do not overwrite an unrelated file. Return the summary with a clickable link to
the saved Markdown.
