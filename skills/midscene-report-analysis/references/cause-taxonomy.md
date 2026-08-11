# Cause taxonomy

## Categories

| Category | General definition | Common cases |
| --- | --- | --- |
| `model_reasoning` | The test is sufficiently specified and observable, but the model makes an incorrect interpretation, decision, action, or assertion. Do not infer this category from a failed model task alone. | Selects the wrong visible element; fails to follow an explicit instruction or required step despite sufficient context; makes an unsupported success or impossibility claim; emits malformed action output; targets the wrong element before a timeout. |
| `test_design` | The test lacks information, preconditions, or observable criteria required for valid execution or judgment. | Omits a required account, login, permission, or data baseline; refers to stale or nonexistent test data; leaves the target ambiguous; requires proving an effect or absence without an observable criterion. |
| `midscene_runtime` | Evidence locates the failure in Midscene's execution or browser-control layer despite valid model intent or output. | Loses the CDP connection; detaches the browser target or frame; fails in the action bridge; parses or executes valid model output incorrectly. |
| `tested_system` | An otherwise valid test and correct action are blocked by the tested application's behavior or state. | An application-owned API or data lookup fails; dependent controls become empty or unusable; an established permission or business state is rejected or lost; a correct action receives no visible application response. |
| `external_dependency` | Evidence locates the failure outside the model, test, Midscene runtime, and tested application. Never use this category for uncertainty. | The model service returns a terminal 429 or 5xx; DNS or a proxy prevents connectivity; external authentication or environment provisioning fails. |
| `unattributed` | The report does not positively support any concrete category, or a clean `true_pass` contains no relevant issue to attribute. | Missing evidence cannot distinguish test data, permissions, runtime, or application ownership; no abnormality relevant to a clean pass was observed. |

## Selection rules

1. Use every concrete category positively supported by report facts; never list a merely unexcluded category.
2. Use `unattributed` only when no concrete category is positively supported; do not use it instead of an available low-confidence concrete attribution.
3. For `unattributed`, distinguish missing or ambiguous attribution evidence from a clean `true_pass` with no relevant issue.
4. When multiple concrete categories are supported, state whether they are joint causes or alternative explanations.

## Confidence

Assign confidence independently for each category:

| Confidence | Use when |
| --- | --- |
| `high` | Direct, corroborated evidence locates the cause in the category. |
| `medium` | The category is the best supported inference, but the full mechanism is not directly exposed. |
| `low` | Observed symptoms positively support the category, but the report cannot distinguish it from another supported owner. Do not use `low` for a merely unexcluded possibility. |

For `unattributed`, confidence measures confidence that the report supports no more specific category, not confidence in an unobserved hidden cause.

## Special cases

### Scrollable UI inspection

Use the following attribution rules:

- **`model_reasoning`:** The model reports that an element is absent or unreachable without reaching every relevant bound (top, bottom, left, and right) of every relevant scroll container. Also use this category when scrolling fails because the model chose an incorrect container, direction, distance, coordinate, or other action parameter. Example: the model scrolls the outer page to the bottom, does not scroll the newly revealed table to its right edge, and concludes that a column is absent.
- **`test_design`:** The scroll action cannot validly locate the target because the test case supplies an incorrect, ambiguous, or impossible target or precondition. Example: the test asks for a row that is not present in the specified dataset, so no amount of valid scrolling can reveal it.
- **`tested_system`:** The test case is valid and the model uses correct scroll parameters, but the tested application's scrollable region does not move or expose the target as it should. Example: a table visibly has more horizontal content, but a correctly targeted horizontal scroll produces no movement because the application control is unresponsive.

When a scroll cannot be completed, check model action parameters first, then the test case. Attribute to `tested_system` only when both are valid.
