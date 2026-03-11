---
name: android-device-automation
description: >
  Vision-driven Android device automation using Midscene.
  Operates entirely from screenshots — no DOM or accessibility labels required. Can interact with all visible elements on screen regardless of technology stack.
  Control Android devices with natural language commands via ADB.
  Perform taps, swipes, text input, app launches, screenshots, and more.
  
  Trigger keywords: android, phone, mobile app, tap, swipe, install app, open app on phone, android device, mobile automation, adb, launch app, mobile screen,
  test android app, verify mobile app, QA on phone, check the app on android, test on device,
  see if the app works on phone, end-to-end test on android, visual verification on mobile

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# Android Device Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.
> 4. **Always report task results before finishing.** After completing the automation task, you MUST proactively summarize the results to the user — including key data found, actions completed, screenshots taken, and any relevant findings. Never silently end after the last automation step; the user expects a complete response in a single interaction.

Automate Android devices using `npx @midscene/android@1`. Each CLI command maps directly to an MCP tool — you (the AI agent) act as the brain, deciding which actions to take based on screenshots.

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
npx @midscene/android@1 connect
npx @midscene/android@1 connect --deviceId emulator-5554
```

When the task needs a merged report, start the flow with a stable session id:

```bash
npx @midscene/android@1 connect --deviceId emulator-5554 --sessionId android-check
```

### Take Screenshot

```bash
npx @midscene/android@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current screen state before deciding the next action.

If you are in a session-report workflow, keep passing the same session id:

```bash
npx @midscene/android@1 take_screenshot --sessionId android-check
```

### Perform Action

Use `act` to interact with the device and get the result. It autonomously handles all UI interactions internally — tapping, typing, scrolling, swiping, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx @midscene/android@1 act --prompt "type hello world in the search field and press Enter"
npx @midscene/android@1 act --prompt "long press the message bubble and tap Delete in the popup menu"

# or target-driven instructions
npx @midscene/android@1 act --prompt "open Settings and navigate to Wi-Fi settings, tell me the connected network name"
```

For multi-command verification that must be merged into one report, every `act` call must reuse the same `--sessionId`:

```bash
npx @midscene/android@1 act --prompt "type release-check in the search field" --sessionId android-check
npx @midscene/android@1 act --prompt "tap the Submit button" --sessionId android-check
```

### Export Session Report

Use this when the user wants a merged report covering multiple CLI calls:

```bash
npx @midscene/android@1 export_session_report --sessionId android-check
```

This command assembles the persisted session executions into a single HTML report and prints the generated report path.

### Verify the Generated Report

Android automation cannot render the exported host-side HTML report directly on the device. If the user explicitly wants visual verification of the merged report itself, switch to **Browser Automation** or **Chrome Bridge Automation** and open:

```bash
file:///absolute/path/to/report/index.html
```

### Disconnect

```bash
npx @midscene/android@1 disconnect
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

1. **Bring the target app to the foreground before using this skill**: For best efficiency, launch the app using ADB (e.g., `adb shell am start -n <package/activity>`) **before** invoking any midscene commands. Then take a screenshot to confirm the app is actually in the foreground. Only after visual confirmation should you proceed with UI automation using this skill. ADB commands are significantly faster than using midscene to navigate to and open apps.
2. **Be specific about UI elements**: Instead of vague descriptions, provide clear, specific details. Say `"the Wi-Fi toggle switch on the right side"` instead of `"the toggle"`.
3. **Describe locations when possible**: Help target elements by describing their position (e.g., `"the search icon at the top right"`, `"the third item in the list"`).
4. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
5. **Batch related operations into a single `act` command**: When performing consecutive operations within the same app, combine them into one `act` prompt instead of splitting them into separate commands. For example, "open Settings, tap Wi-Fi, and toggle it on" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
6. **Always report results after completion**: After finishing the automation task, you MUST proactively present the results to the user without waiting for them to ask. This includes: (1) the answer to the user's original question or the outcome of the requested task, (2) key data extracted or observed during execution, (3) screenshots and other generated files with their paths, (4) a brief summary of steps taken. Do NOT silently finish after the last automation command — the user expects complete results in a single interaction.
7. **Use `--sessionId` for multi-command verification**: If the task involves multiple CLI calls that must be merged into one report, define one stable session id at the beginning and pass it to every command that supports it.
8. **Export only after all actions are done**: `export_session_report` is the assembly step. Do not run it before the final action if the user expects one complete merged report.
9. **Use a browser-based skill to inspect the exported HTML**: The report is a host-side HTML artifact. If the user asks to verify the report itself, open it with Browser Automation or Chrome Bridge Automation after exporting.

**Example — Popup menu interaction:**

```bash
npx @midscene/android@1 act --prompt "long press the message bubble and tap Delete in the popup menu"
npx @midscene/android@1 take_screenshot
```

**Example — Form interaction:**

```bash
npx @midscene/android@1 act --prompt "fill in the username field with 'testuser' and the password field with 'pass123', then tap the Login button"
npx @midscene/android@1 take_screenshot
```

**Example — Multi-command flow with one merged report:**

```bash
npx @midscene/android@1 connect --deviceId emulator-5554 --sessionId android-check
npx @midscene/android@1 take_screenshot --sessionId android-check
npx @midscene/android@1 act --prompt "type release-check in the search field" --sessionId android-check
npx @midscene/android@1 act --prompt "tap the Submit button" --sessionId android-check
npx @midscene/android@1 export_session_report --sessionId android-check
# Then open file:///absolute/path/to/report/index.html with Browser Automation or Chrome Bridge Automation if visual report validation is required
```

## Troubleshooting

| Problem | Solution |
|---|---|
| **ADB not found** | Install Android SDK Platform Tools: `brew install android-platform-tools` (macOS) or download from [developer.android.com](https://developer.android.com/tools/releases/platform-tools). |
| **Device not listed** | Check USB connection, ensure USB debugging is enabled in Developer Options, and run `adb devices`. |
| **Device shows "unauthorized"** | Unlock the device and accept the USB debugging authorization prompt. Then run `adb devices` again. |
| **Device shows "offline"** | Disconnect and reconnect the USB cable. Run `adb kill-server && adb start-server`. |
| **Command timeout** | The device screen may be off or locked. Wake the device with `adb shell input keyevent KEYCODE_WAKEUP` and unlock it. |
| **API key error** | Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`. See [Model Configuration](https://midscenejs.com/zh/model-common-config.html). |
| **Wrong device targeted** | If multiple devices are connected, use `--deviceId <id>` flag with the `connect` command. |
