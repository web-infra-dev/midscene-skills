---
name: ios-device-automation
description: |
  Vision-driven iOS device automation using Midscene CLI. Operates entirely from screenshots — no DOM or accessibility labels required. Can interact with all visible elements on screen regardless of technology stack.
  Control iOS devices with natural language commands via WebDriverAgent.
  
  Triggers: ios, iphone, ipad, ios app, tap on iphone, swipe, mobile app ios,
  ios device, ios testing, iphone automation, ipad automation, ios screen, ios navigate,
  test ios app, verify on iphone, QA on ipad, check the app on ios, test on ios device,
  see if the app works on iphone, end-to-end test on ios, visual verification on ios

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# iOS Device Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.
> 4. **Always report task results before finishing.** After completing the automation task, you MUST proactively summarize the results to the user — including key data found, actions completed, screenshots taken, and any relevant findings. Never silently end after the last automation step; the user expects a complete response in a single interaction.

Automate iOS devices using `npx @midscene/ios@1`. Each CLI command maps directly to an MCP tool — you (the AI agent) act as the brain, deciding which actions to take based on screenshots.

If the task spans multiple CLI calls and the user wants **one merged report**, you MUST choose a stable `--sessionId`, reuse it on every command, then run `export_session_report` at the end to assemble the final HTML report.

## Prerequisites

Midscene requires models with strong visual grounding capabilities. The following environment variables must be configured — either as system environment variables or in a `.env` file in the current working directory (Midscene loads `.env` automatically):

```bash
MIDSCENE_MODEL_API_KEY="your-api-key"
MIDSCENE_MODEL_NAME="model-name"
MIDSCENE_MODEL_BASE_URL="https://..."
MIDSCENE_MODEL_FAMILY="family-identifier"
```

Example: Gemini (Gemini-3-Flash)

```bash
MIDSCENE_MODEL_API_KEY="your-google-api-key"
MIDSCENE_MODEL_NAME="gemini-3-flash"
MIDSCENE_MODEL_BASE_URL="https://generativelanguage.googleapis.com/v1beta/openai/"
MIDSCENE_MODEL_FAMILY="gemini"
```

Example: Qwen 3.5

```bash
MIDSCENE_MODEL_API_KEY="your-aliyun-api-key"
MIDSCENE_MODEL_NAME="qwen3.5-plus"
MIDSCENE_MODEL_BASE_URL="https://dashscope.aliyuncs.com/compatible-mode/v1"
MIDSCENE_MODEL_FAMILY="qwen3.5"
MIDSCENE_MODEL_REASONING_ENABLED="false"
# If using OpenRouter, set:
# MIDSCENE_MODEL_API_KEY="your-openrouter-api-key"
# MIDSCENE_MODEL_NAME="qwen/qwen3.5-plus"
# MIDSCENE_MODEL_BASE_URL="https://openrouter.ai/api/v1"
```

Example: Doubao Seed 2.0 Lite

```bash
MIDSCENE_MODEL_API_KEY="your-doubao-api-key"
MIDSCENE_MODEL_NAME="doubao-seed-2-0-lite"
MIDSCENE_MODEL_BASE_URL="https://ark.cn-beijing.volces.com/api/v3"
MIDSCENE_MODEL_FAMILY="doubao-seed"
```

Commonly used models: Doubao Seed 2.0 Lite, Qwen 3.5, Zhipu GLM-4.6V, Gemini-3-Pro, Gemini-3-Flash.

If the model is not configured, ask the user to set it up. See [Model Configuration](https://midscenejs.com/model-common-config) for supported providers.

## Commands

### Connect to Device

```bash
npx @midscene/ios@1 connect
```

When the task needs a merged report, start the flow with a stable session id:

```bash
npx @midscene/ios@1 connect --sessionId ios-check
```

### Take Screenshot

```bash
npx @midscene/ios@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current screen state before deciding the next action.

If you are in a session-report workflow, keep passing the same session id:

```bash
npx @midscene/ios@1 take_screenshot --sessionId ios-check
```

### Perform Action

Use `act` to interact with the device and get the result. It autonomously handles all UI interactions internally — tapping, typing, scrolling, swiping, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx @midscene/ios@1 act --prompt "type hello world in the search field and press Enter"
npx @midscene/ios@1 act --prompt "tap Delete, then confirm in the alert dialog"

# or target-driven instructions
npx @midscene/ios@1 act --prompt "open Settings and navigate to Wi-Fi, tell me the connected network name"
```

For multi-command verification that must be merged into one report, every `act` call must reuse the same `--sessionId`:

```bash
npx @midscene/ios@1 act --prompt "type release-check in the search field" --sessionId ios-check
npx @midscene/ios@1 act --prompt "tap the Submit button" --sessionId ios-check
```

### Export Session Report

Use this when the user wants a merged report covering multiple CLI calls:

```bash
npx @midscene/ios@1 export_session_report --sessionId ios-check
```

This command assembles the persisted session executions into a single HTML report and prints the generated report path.

### Verify the Generated Report

iOS automation cannot render the exported host-side HTML report directly on the device. If the user explicitly wants visual verification of the merged report itself, switch to **Browser Automation** or **Chrome Bridge Automation** and open:

```bash
file:///absolute/path/to/report/index.html
```

### Disconnect

```bash
npx @midscene/ios@1 disconnect
```

## Workflow Pattern

Since CLI commands are stateless between invocations, follow this pattern:

1. **Connect** to establish a session
2. **Launch the target app and take screenshot** to see the current state, make sure the app is launched and visible on the screen.
3. **Execute action** using `act` to perform the desired action or target-driven instructions.
4. **Disconnect** when done
5. **Report results** — summarize what was accomplished, present key findings and data extracted during the task, and list any generated files (screenshots, logs, etc.) with their paths

If the user explicitly asks for a merged report or asks to verify multiple CLI calls in one report, use this extended pattern instead:

1. **Choose one `sessionId` up front** and reuse it for every command in the flow.
2. **Connect** with `--sessionId`.
3. **Take screenshot** and inspect the current screen before acting.
4. **Run one or more `act` commands** with the same `--sessionId`.
5. **Export the merged report** with `export_session_report --sessionId ...`.
6. **If visual report verification is required**, open the generated `report/index.html` with Browser Automation or Chrome Bridge Automation on the host machine.
7. **Disconnect** when done.
8. **Report results** with the session id, report path, screenshot paths, and whether the merged report was exported successfully.

## Best Practices

1. **Be specific about UI elements**: Instead of vague descriptions, provide clear, specific details. Say `"the Settings icon in the top-right corner"` instead of `"the icon"`.
2. **Describe locations when possible**: Help target elements by describing their position (e.g., `"the search icon at the top right"`, `"the third item in the list"`).
3. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
4. **Batch related operations into a single `act` command**: When performing consecutive operations within the same app, combine them into one `act` prompt instead of splitting them into separate commands. For example, "open Settings, tap Wi-Fi, and check the connected network" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
5. **Always report results after completion**: After finishing the automation task, you MUST proactively present the results to the user without waiting for them to ask. This includes: (1) the answer to the user's original question or the outcome of the requested task, (2) key data extracted or observed during execution, (3) screenshots and other generated files with their paths, (4) a brief summary of steps taken. Do NOT silently finish after the last automation command — the user expects complete results in a single interaction.
6. **Use `--sessionId` for multi-command verification**: If the task involves multiple CLI calls that must be merged into one report, define one stable session id at the beginning and pass it to every command that supports it.
7. **Export only after all actions are done**: `export_session_report` is the assembly step. Do not run it before the final action if the user expects one complete merged report.
8. **Use a browser-based skill to inspect the exported HTML**: The report is a host-side HTML artifact. If the user asks to verify the report itself, open it with Browser Automation or Chrome Bridge Automation after exporting.

**Example — Alert dialog interaction:**

```bash
npx @midscene/ios@1 act --prompt "tap the Delete button and confirm in the alert dialog"
npx @midscene/ios@1 take_screenshot
```

**Example — Form interaction:**

```bash
npx @midscene/ios@1 act --prompt "fill in the username field with 'testuser' and the password field with 'pass123', then tap the Login button"
npx @midscene/ios@1 take_screenshot
```

**Example — Multi-command flow with one merged report:**

```bash
npx @midscene/ios@1 connect --sessionId ios-check
npx @midscene/ios@1 take_screenshot --sessionId ios-check
npx @midscene/ios@1 act --prompt "type release-check in the search field" --sessionId ios-check
npx @midscene/ios@1 act --prompt "tap the Submit button" --sessionId ios-check
npx @midscene/ios@1 export_session_report --sessionId ios-check
# Then open file:///absolute/path/to/report/index.html with Browser Automation or Chrome Bridge Automation if visual report validation is required
```

## Troubleshooting

### WebDriverAgent Not Running
**Symptom:** Connection refused or timeout errors.
**Solution:**
- Ensure WebDriverAgent is installed and running on the device.
- See https://midscenejs.com/zh/usage-ios.html for setup instructions.

### Device Not Found
**Symptom:** No device detected or connection errors.
**Solution:**
- Ensure the device is connected via USB and trusted.

### API Key Issues
**Symptom:** Authentication or model errors.
**Solution:**
- Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`.
- See https://midscenejs.com/zh/model-common-config.html for details.
