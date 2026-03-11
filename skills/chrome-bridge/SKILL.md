---
name: chrome-bridge-automation
description: |
  Vision-driven browser automation using Midscene Bridge mode. Operates entirely from screenshots — no DOM or accessibility labels required. Can interact with all visible elements on screen regardless of technology stack.

  Connects to the user's desktop Chrome browser via the Midscene Chrome Extension, preserving cookies, sessions, and login state.
  Does NOT take over the user's mouse or keyboard — operates through Chrome DevTools Protocol.

  Use this skill when the user wants to:
  - Browse, navigate, or open web pages in the user's own Chrome browser
  - Interact with pages that require login sessions, cookies, or existing browser state
  - Scrape, extract, or collect data from websites using the user's real browser
  - Fill out forms, click buttons, or interact with web elements
  - Verify, validate, or test frontend UI behavior
  - Take screenshots of web pages
  - Automate multi-step web workflows
  - Check website content or appearance
  - Test the web app in Chrome, verify it works, QA the frontend
  - Check if the page looks correct, end-to-end test the website
  - Test what was just built, validate the UI in real browser

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# Chrome Bridge Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.
> 4. **Always report task results before finishing.** After completing the automation task, you MUST proactively summarize the results to the user — including key data found, actions completed, screenshots taken, and any relevant findings. Never silently end after the last automation step; the user expects a complete response in a single interaction.

Automate the user's real Chrome browser via the Midscene Chrome Extension (Bridge mode), preserving cookies, sessions, and login state. You (the AI agent) act as the brain, deciding which actions to take based on screenshots.

If the task spans multiple CLI calls and the user wants **one merged report**, you MUST choose a stable `--sessionId`, reuse it on every command, then run `export_session_report` at the end to assemble the final HTML report.

## Command Format

**CRITICAL — Every command MUST follow this EXACT format. Do NOT modify the command prefix.**

```
npx @midscene/web@1 --bridge <subcommand> [args]
```

- `--bridge` flag is **MANDATORY** here — it activates Bridge mode to connect to the user's desktop Chrome browser

## Prerequisites

The user has already prepared Chrome and the Midscene Extension. Do NOT check browser or extension status before connecting — just connect directly.

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

### Connect to a Web Page

```bash
npx @midscene/web@1 --bridge connect --url https://example.com
```

When the task needs a merged report, start the flow with a stable session id:

```bash
npx @midscene/web@1 --bridge connect --url https://example.com --sessionId bridge-check
```

### Take Screenshot

```bash
npx @midscene/web@1 --bridge take_screenshot
```

After taking a screenshot, read the saved image file to understand the current page state before deciding the next action.

If you are in a session-report workflow, keep passing the same session id:

```bash
npx @midscene/web@1 --bridge take_screenshot --sessionId bridge-check
```

### Perform Action

Use `act` to interact with the page and get the result. It autonomously handles all UI interactions internally — clicking, typing, scrolling, hovering, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx @midscene/web@1 --bridge act --prompt "click the Login button and fill in the email field with 'user@example.com'"
npx @midscene/web@1 --bridge act --prompt "scroll down and click the Submit button"

# or target-driven instructions
npx @midscene/web@1 --bridge act --prompt "click the country dropdown and select Japan"
```

For multi-command verification that must be merged into one report, every `act` call must reuse the same `--sessionId`:

```bash
npx @midscene/web@1 --bridge act --prompt "type release-check in the Name input field" --sessionId bridge-check
npx @midscene/web@1 --bridge act --prompt "click the Complete Flow button" --sessionId bridge-check
```

### Export Session Report

Use this when the user wants a merged report covering multiple CLI calls:

```bash
npx @midscene/web@1 --bridge export_session_report --sessionId bridge-check
```

This command assembles the persisted session executions into a single HTML report and prints the generated report path.

### Open a Generated Report

After exporting, open the generated HTML report and inspect it like any other page:

```bash
npx @midscene/web@1 --bridge connect --url "file:///absolute/path/to/report/index.html"
npx @midscene/web@1 --bridge take_screenshot
```

Use this final screenshot pass to verify that the merged report actually renders the expected executions.

### Disconnect

```bash
npx @midscene/web@1 --bridge disconnect
```

## Workflow Pattern

Bridge mode connects to the user's real Chrome browser. Each CLI command establishes its own temporary connection, but **the browser, tabs, and all state (cookies, login sessions) are always preserved** regardless of whether you disconnect. This makes reconnecting lightweight and lossless.

Follow this pattern:

1. **Connect** to a URL to establish a session
2. **Take screenshot** to see the current state, make sure the page is loaded.
3. **Execute action** using `act` to perform the desired action or target-driven instructions.
4. **Report results** — summarize what was accomplished, present key findings and data extracted during the task, and list any generated files (screenshots, logs, etc.) with their paths
5. **Disconnect** only when the user's overall task is fully complete. **Do NOT disconnect** if the user may have follow-up actions — keep the session available for continued interaction in subsequent conversation turns.

If the user explicitly asks for a merged report or asks to verify multiple CLI calls in one report, use this extended pattern instead:

1. **Choose one `sessionId` up front** and reuse it for every command in the flow.
2. **Connect** with `--sessionId`.
3. **Take screenshot** and inspect the current page before acting.
4. **Run one or more `act` commands** with the same `--sessionId`.
5. **Export the merged report** with `export_session_report --sessionId ...`.
6. **Open the generated report HTML** and take another screenshot to verify it renders correctly.
7. **Report results** with the session id, report path, screenshot paths, and whether the merged report looked correct.
8. **Disconnect** only when the user's overall task is fully complete.

## Best Practices

1. **Always connect first**: Navigate to the target URL with `connect --url` before any interaction.
2. **Be specific about UI elements**: Instead of `"the button"`, say `"the blue Submit button in the contact form"`.
3. **Use natural language**: Describe what you see on the page, not CSS selectors. Say `"the red Buy Now button"` instead of `"#buy-btn"`.
4. **Handle loading states**: After navigation or actions that trigger page loads, take a screenshot to verify the page has loaded.
5. **Disconnect only when fully done**: Only disconnect when the user's overall task is completely finished and no follow-up actions are expected. In multi-turn conversations, skip the disconnect to allow continued browser interaction. Disconnecting is safe — it only closes the CLI-side bridge connection, not the browser or tabs — but reconnecting adds unnecessary overhead if the user wants to continue.
6. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
7. **Batch related operations into a single `act` command**: When performing consecutive operations within the same page, combine them into one `act` prompt instead of splitting them into separate commands. For example, "fill in the email and password fields, then click the Login button" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
8. **Always report results after completion**: After finishing the automation task, you MUST proactively present the results to the user without waiting for them to ask. This includes: (1) the answer to the user's original question or the outcome of the requested task, (2) key data extracted or observed during execution, (3) screenshots and other generated files with their paths, (4) a brief summary of steps taken. Do NOT silently finish after the last automation command — the user expects complete results in a single interaction.
9. **Use `--sessionId` for multi-command verification**: If the task involves multiple CLI calls that must be merged into one report, define one stable session id at the beginning and pass it to every command that supports it.
10. **Export only after all actions are done**: `export_session_report` is the assembly step. Do not run it before the final action if the user expects one complete merged report.
11. **Open the exported report and check it visually**: Do not stop after seeing the report file path. Load the generated `report/index.html`, take a screenshot, and verify that the merged result actually renders.

**Example — Dropdown selection:**

```bash
npx @midscene/web@1 --bridge act --prompt "click the country dropdown and select Japan"
npx @midscene/web@1 --bridge take_screenshot
```

**Example — Form interaction:**

```bash
npx @midscene/web@1 --bridge act --prompt "fill in the email field with 'user@example.com' and the password field with 'pass123', then click the Log In button"
npx @midscene/web@1 --bridge take_screenshot
```

**Example — Multi-command flow with one merged report:**

```bash
npx @midscene/web@1 --bridge connect --url https://example.com --sessionId bridge-check
npx @midscene/web@1 --bridge take_screenshot --sessionId bridge-check
npx @midscene/web@1 --bridge act --prompt "type release-check in the Name input field" --sessionId bridge-check
npx @midscene/web@1 --bridge act --prompt "click the Complete Flow button" --sessionId bridge-check
npx @midscene/web@1 --bridge export_session_report --sessionId bridge-check
npx @midscene/web@1 --bridge connect --url "file:///absolute/path/to/report/index.html"
npx @midscene/web@1 --bridge take_screenshot
```

## Troubleshooting

### Bridge Mode Connection Failures
- Ask user to check if Chrome is open with the Midscene Extension installed and enabled.
- The Midscene Extension can be installed from the Chrome Web Store: https://chromewebstore.google.com/detail/midscenejs/gbldofcpkknbggpkmbdaefngejllnief
- Check that the 'bridge mode' indicator in the extension shows "Listening" status.
- See the [Bridge Mode documentation](https://midscenejs.com/bridge-mode-by-chrome-extension.html).

### Timeouts
- Web pages may take time to load. After connecting, take a screenshot to verify readiness before interacting.
- For slow pages, wait briefly between steps.

### Screenshots Not Displaying
- The screenshot path is an absolute path to a local file. Use the Read tool to view it.
