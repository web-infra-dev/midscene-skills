# Analysis output template

Write the final result directly as Markdown using the common structure below.
Use metadata's `source` as the Report link destination. Link screenshots with
absolute local paths or HTTP(S) URLs.

## Common structure

Replace the status-specific findings placeholder with the applicable section
below. Follow [cause-taxonomy.md](cause-taxonomy.md) for category selection,
per-category confidence, and relationships between multiple categories.

```markdown
# Midscene report analysis

- **Report:** [Open report](<source>)
- **Execution status:** `<pass|fail|incomplete>`

## Conclusion

<concise conclusion supported by the evidence below>

<status-specific findings>

## Cause categories

- `<category>` (`<high|medium|low>` confidence): <reason with evidence references such as E1, E2>

## Evidence

<evidence entries>

## Limitations

<remaining evidence gaps or conflicts and their effect on the conclusion, or `none`>
```

## Status-specific findings

### Passed or failed execution

For `pass` or `fail`, use the corresponding `resultAssessment` values and
decision rules in [SKILL.md](../SKILL.md#3-load-and-apply-the-analysis-rules).

```markdown
## Result assessment

- **Assessment:** `<resultAssessment>`
- **Assessment confidence:** `<high|medium|low>` — <reason>

<required conditions, state at the judgment point, and supporting or
contradicting evidence references>
```

Assessment confidence measures support for the chosen result assessment,
independently of confidence in the cause attribution. Use `high` for direct,
consistent support, `medium` for a supported interpretation with remaining
ambiguity, and `low` when the assessment itself is tentative. An established
evidence gap can support high confidence in an `unverifiable` assessment.

Append the following section only for `true_fail`:

```markdown
## Failure details

- **Failure reason:** <why a required condition was unmet, with evidence references>
- **Failed step:** <execution/task identifier and step or timestamp>
```

Append the following section only for `false_pass`:

```markdown
## Pass-claim issue

<required condition proven unmet at the judgment point, with evidence references>
```

### Incomplete execution

For `incomplete`, use the section below without a result assessment or
assessment confidence. Keep uncertainty about the observed issue and the
interruption reason separate; the conclusion must not call the report passed
or failed.

```markdown
## Incomplete execution

- **Observed issue:** <evidenced issue and its uncertainty, or that none was observed>
- **Interruption reason:** <established reason, or that the exact reason is not recorded>
- **Last recorded step:** <execution/task identifier and step or timestamp, or that no tasks were recorded>
```

## Evidence entries

Give each entry an identifier such as `E1` and one traceable fact. Identify the
source type (instruction, action, machine result, screenshot, or model
narration) and its location using available execution/task identifiers, step
numbers, timestamps, or exported file lines. Add a screenshot link only for
screenshot evidence.

```markdown
- **E1**
  - **Source:** <source type; execution/task identifier; step, timestamp, or file location>
  - **Fact:** <fact established by this source>
  - **Screenshot:** [Open screenshot](<absolute-path-or-url>)
```
