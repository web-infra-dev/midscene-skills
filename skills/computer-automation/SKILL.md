---
name: desktop-computer-automation
description: |
  Vision-driven desktop automation using Midscene. Control your desktop (macOS, Windows, Linux) with natural language commands.
  Operates entirely from screenshots — no DOM or accessibility labels required. Can interact with all visible elements on screen regardless of technology stack.

  ⚠️ WARNING: This skill takes over the user's real mouse and keyboard. The user cannot use their computer while automation is running.
  → For web apps, prefer the "Browser Automation" skill instead — it runs in a headless browser and does NOT interfere with the user's mouse/keyboard.
  → Only use this skill for desktop-native applications (Electron, Qt, native macOS/Windows/Linux apps) that cannot be tested in a browser.

  Triggers: open app, press key, desktop, computer, click on screen, type text, screenshot desktop,
  launch application, switch window, desktop automation, control computer, mouse click, keyboard shortcut,
  screen capture, find on screen, read screen, verify window, close app, minimize window, maximize window,
  test desktop app, verify native app works, QA desktop feature, try the desktop app,
  launch and test native app, end-to-end test desktop, test Electron app, test native UI

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# Desktop Computer Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.
> 4. **Always report task results before finishing.** After completing the automation task, you MUST proactively summarize the results to the user — including key data found, actions completed, screenshots taken, and any relevant findings. Never silently end after the last automation step; the user expects a complete response in a single interaction.

Control your desktop (macOS, Windows, Linux) using `npx @midscene/computer@1`. Each CLI command maps directly to an MCP tool — you (the AI agent) act as the brain, deciding which actions to take based on screenshots.

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

### Connect to Desktop

```bash
npx @midscene/computer@1 connect
npx @midscene/computer@1 connect --displayId <id>
```

When the task needs a merged report, start the flow with a stable session id:

```bash
npx @midscene/computer@1 connect --displayId <id> --sessionId desktop-check
```

### List Displays

```bash
npx @midscene/computer@1 list_displays
```

### Take Screenshot

```bash
npx @midscene/computer@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current screen state before deciding the next action.

If you are in a session-report workflow, keep passing the same session id:

```bash
npx @midscene/computer@1 take_screenshot --sessionId desktop-check
```

### Perform Action

Use `act` to interact with the computer and get the result. It autonomously handles all UI interactions internally — clicking, typing, scrolling, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx @midscene/computer@1 act --prompt "type hello world in the search field and press Enter"
npx @midscene/computer@1 act --prompt "drag the file icon to the Trash"

# or target-driven instructions
npx @midscene/computer@1 act --prompt "search for the weather in Shanghai using the Chrome browser, tell me the result"
```

For multi-command verification that must be merged into one report, every `act` call must reuse the same `--sessionId`:

```bash
npx @midscene/computer@1 act --prompt "type release-check in the search field" --sessionId desktop-check
npx @midscene/computer@1 act --prompt "click the Submit button" --sessionId desktop-check
```

### Export Session Report

Use this when the user wants a merged report covering multiple CLI calls:

```bash
npx @midscene/computer@1 export_session_report --sessionId desktop-check
```

This command assembles the persisted session executions into a single HTML report and prints the generated report path.

### Verify the Generated Report

The exported report is a host-side HTML file. If the user explicitly wants visual verification of the merged report itself, prefer switching to **Browser Automation** or **Chrome Bridge Automation** and open:

```bash
file:///absolute/path/to/report/index.html
```

### Disconnect

```bash
npx @midscene/computer@1 disconnect
```

## Workflow Pattern

Since CLI commands are stateless between invocations, follow this pattern:

1. **Connect** to establish a session
2. **Health check** — observe the output of the `connect` command. If `connect` already performed a health check (screenshot and mouse movement test), no additional check is needed. If `connect` did not perform a health check, do one manually: take a screenshot and verify it succeeds, then move the mouse to a random position (`act --prompt "move the mouse to a random position"`) and verify it succeeds. If either step fails, stop and troubleshoot before continuing. Only proceed to the next steps after both checks pass without errors.
3. **Launch the target app and take screenshot** to see the current state, make sure the app is launched and visible on the screen.
4. **Execute action** using `act` to perform the desired action or target-driven instructions.
5. **Disconnect** when done
6. **Report results** — summarize what was accomplished, present key findings and data extracted during the task, and list any generated files (screenshots, logs, etc.) with their paths

If the user explicitly asks for a merged report or asks to verify multiple CLI calls in one report, use this extended pattern instead:

1. **Choose one `sessionId` up front** and reuse it for every command in the flow.
2. **Connect** with `--sessionId` and complete the health check.
3. **Take screenshot** and inspect the current desktop state before acting.
4. **Run one or more `act` commands** with the same `--sessionId`.
5. **Export the merged report** with `export_session_report --sessionId ...`.
6. **If visual report verification is required**, open the generated `report/index.html` with Browser Automation or Chrome Bridge Automation on the host machine.
7. **Disconnect** when done.
8. **Report results** with the session id, report path, screenshot paths, and whether the merged report was exported successfully.

## Best Practices

1. **Always run a health check first**: After connecting, observe the output of the `connect` command. If `connect` already performed a health check (screenshot and mouse movement test), no additional check is needed. If it did not, do one manually: take a screenshot and move the mouse to a random position. Both must succeed (no errors) before proceeding with any further operations. This catches environment issues early.
2. **Bring the target app to the foreground before using this skill**: For best efficiency, bring the app to the foreground using other means (e.g., `open -a <AppName>` on macOS, `start <AppName>` on Windows) **before** invoking any midscene commands. Then take a screenshot to confirm the app is actually in the foreground. Only after visual confirmation should you proceed with UI automation using this skill. Avoid using Spotlight, Start menu search, or other launcher-based approaches through midscene — they involve transient UI, multiple AI inference steps, and are significantly slower.
3. **Be specific about UI elements**: Instead of vague descriptions, provide clear, specific details. Say `"the red close button in the top-left corner of the Safari window"` instead of `"the close button"`.
4. **Describe locations when possible**: Help target elements by describing their position (e.g., `"the icon in the top-right corner of the menu bar"`, `"the third item in the left sidebar"`).
5. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
6. **Check for multiple displays**: If you launched an app but cannot see it on the screenshot, the app window may have opened on a different display. Use `list_displays` to check available displays. You have two options: either move the app window to the current display, or use `connect --displayId <id>` to switch to the display where the app is.
7. **Batch related operations into a single `act` command**: When performing consecutive operations within the same app, combine them into one `act` prompt instead of splitting them into separate commands. For example, "search for X, click the first result, and scroll down to see more details" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
8. **Set up `PATH` before running (macOS)**: On macOS, some commands (e.g., `system_profiler`) may not be found if the `PATH` is incomplete. Before running any midscene commands, ensure the `PATH` includes the standard system directories:
   ```bash
   export PATH="/usr/sbin:/usr/bin:/bin:/sbin:$PATH"
   ```
   This prevents screenshot failures caused by missing system utilities.
9. **Always report results after completion**: After finishing the automation task, you MUST proactively present the results to the user without waiting for them to ask. This includes: (1) the answer to the user's original question or the outcome of the requested task, (2) key data extracted or observed during execution, (3) screenshots and other generated files with their paths, (4) a brief summary of steps taken. Do NOT silently finish after the last automation command — the user expects complete results in a single interaction.
10. **Use `--sessionId` for multi-command verification**: If the task involves multiple CLI calls that must be merged into one report, define one stable session id at the beginning and pass it to every command that supports it.
11. **Export only after all actions are done**: `export_session_report` is the assembly step. Do not run it before the final action if the user expects one complete merged report.
12. **Use a browser-based skill to inspect the exported HTML**: The report is a host-side HTML artifact. If the user asks to verify the report itself, open it with Browser Automation or Chrome Bridge Automation after exporting.

**Example — Context menu interaction:**

```bash
npx @midscene/computer@1 act --prompt "right-click the file icon and select Delete from the context menu"
npx @midscene/computer@1 take_screenshot
```

**Example — Dropdown menu:**

```bash
npx @midscene/computer@1 act --prompt "open the File menu and click New Window"
npx @midscene/computer@1 take_screenshot
```

**Example — Multi-command flow with one merged report:**

```bash
npx @midscene/computer@1 connect --sessionId desktop-check
npx @midscene/computer@1 take_screenshot --sessionId desktop-check
npx @midscene/computer@1 act --prompt "type release-check in the search field" --sessionId desktop-check
npx @midscene/computer@1 act --prompt "click the Submit button" --sessionId desktop-check
npx @midscene/computer@1 export_session_report --sessionId desktop-check
# Then open file:///absolute/path/to/report/index.html with Browser Automation or Chrome Bridge Automation if visual report validation is required
```


## Troubleshooting

### macOS: Accessibility Permission Denied
Your terminal app does not have Accessibility access:
1. Open **System Settings > Privacy & Security > Accessibility**
2. Add your terminal app and enable it
3. Restart your terminal app after granting permission

### macOS: Xcode Command Line Tools Not Found
```bash
xcode-select --install
```

### API Key Not Set
Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`.

### macOS: Screenshot Fails with `system_profiler` Not Found
If `take_screenshot` fails with an error like `system_profiler: command not found`, the `PATH` environment variable is likely incomplete. Fix it by running:
```bash
export PATH="/usr/sbin:/usr/bin:/bin:/sbin:$PATH"
```
Then retry the screenshot command.

### AI Cannot Find the Element
1. Take a screenshot to verify the element is actually visible
2. Use more specific descriptions (include color, position, surrounding text)
3. Ensure the element is not hidden behind another window
