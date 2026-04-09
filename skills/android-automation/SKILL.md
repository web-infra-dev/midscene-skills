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

## What `act` Can Do

Inside a single `act` call on Android, Midscene can tap, double-tap, long-press, type, clear text, scroll or swipe in any direction, pull to refresh, drag items, zoom with two fingers, press keys, and use system navigation such as Back, Home, or recent apps while working from the current visible screen.

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

### Launch an App or URL

Use the dedicated launch step when you want a deterministic starting point before the rest of the task:

```bash
npx @midscene/android@1 launch --uri https://www.ebay.com
npx @midscene/android@1 launch --uri com.android.settings
npx @midscene/android@1 launch --uri com.android.settings/.Settings
```

### Run a Raw Android Shell Command

Use this when the task needs lower-level device control that is not best expressed as a visible UI interaction:

```bash
npx @midscene/android@1 runadbshell --command "dumpsys battery"
```

This is forwarded to `adb shell` on the connected device. In practice, the underlying command is `adb -s <deviceId> shell dumpsys battery` and some environments may also include the default ADB server port, such as `adb -P 5037 -s <deviceId> shell dumpsys battery`.

### Take Screenshot

```bash
npx @midscene/android@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current screen state before deciding the next action.

### Perform Action

Use `act` to interact with the device and get the result. It autonomously handles all UI interactions internally — tapping, typing, scrolling, swiping, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx @midscene/android@1 act --prompt "type hello world in the search field and press Enter"
npx @midscene/android@1 act --prompt "long press the message bubble and tap Delete in the popup menu"

# or target-driven instructions
npx @midscene/android@1 act --prompt "open Settings and navigate to Wi-Fi settings, tell me the connected network name"
```

### Use a Reference Image for Precise Targeting

When the user provides a screenshot, icon, logo, or reference image and wants an exact visual match, prefer `tap --locate` instead of a generic `act --prompt`. Pass `--locate` as JSON. The `prompt` describes the target, `images` supplies named reference images, and `convertHttpImage2Base64: true` is useful when the image URL may not be directly accessible to the model.

```bash
npx @midscene/android@1 tap --locate '{
  "prompt": "tap the area contains the image",
  "images": [
    {
      "name": "target image",
      "url": "https://github.githubassets.com/assets/GitHub-Mark-ea2971cee799.png"
    }
  ],
  "convertHttpImage2Base64": true
}'
```

The same `locate` JSON shape also works for other commands that accept a `locate` parameter.

### Disconnect

```bash
npx @midscene/android@1 disconnect
```

### Consume Report Files

The generated HTML report is recommended for human reading first. It includes step-by-step execution details and replay videos for each operation, which makes it much easier to understand what happened and troubleshoot problems.

If another skill or tool needs to consume the report, first convert it with `report-tool` from the same platform CLI package. Prefer Markdown for LLM-based workflows. Use JSON when the report needs to be processed programmatically.

```bash
npx @midscene/android@1 report-tool --action to-markdown --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-markdown
npx @midscene/android@1 report-tool --action split --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-data
```

## Workflow Pattern

Since CLI commands are stateless between invocations, follow this pattern:

1. **Connect** to establish a session
2. **Launch the target app and take screenshot** to see the current state, make sure the app is launched and visible on the screen.
3. **Execute action** using `act` to perform the desired action or target-driven instructions.
4. **Disconnect** when done
5. **Report results** — summarize what was accomplished, present key findings and data extracted during the task, and list any generated files (screenshots, logs, etc.) with their paths

## Best Practices

1. **Bring the target app to the foreground before using this skill**: For best efficiency, launch the app using ADB (e.g., `adb shell am start -n <package/activity>`) **before** invoking any midscene commands. Then take a screenshot to confirm the app is actually in the foreground. Only after visual confirmation should you proceed with UI automation using this skill. ADB commands are significantly faster than using midscene to navigate to and open apps.
2. **Be specific about UI elements**: Instead of vague descriptions, provide clear, specific details. Say `"the Wi-Fi toggle switch on the right side"` instead of `"the toggle"`.
3. **Describe locations when possible**: Help target elements by describing their position (e.g., `"the search icon at the top right"`, `"the third item in the list"`).
4. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
5. **Batch related operations into a single `act` command**: When performing consecutive operations within the same app, combine them into one `act` prompt instead of splitting them into separate commands. For example, "open Settings, tap Wi-Fi, and toggle it on" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
6. **Always report results after completion**: After finishing the automation task, you MUST proactively present the results to the user without waiting for them to ask. This includes: (1) the answer to the user's original question or the outcome of the requested task, (2) key data extracted or observed during execution, (3) screenshots and other generated files with their paths, (4) a brief summary of steps taken. Do NOT silently finish after the last automation command — the user expects complete results in a single interaction.
7. **Prefer `tap --locate` when a reference image is provided**: If the user shares a screenshot, icon, or logo and wants that exact visual target, use `tap --locate` with a multimodal `locate` JSON object such as `{ "prompt": "...", "images": [...] }` instead of relying only on `act --prompt`.

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

## Troubleshooting

| Problem | Solution |
|---|---|
| **ADB not found** | Install Android SDK Platform Tools: `brew install android-platform-tools` (macOS) or download from [developer.android.com](https://developer.android.com/tools/releases/platform-tools). |
| **Device not listed** | Check USB connection, ensure USB debugging is enabled in Developer Options, and run `adb devices`. |
| **Device shows "unauthorized"** | Unlock the device and accept the USB debugging authorization prompt. Then run `adb devices` again. |
| **Device shows "offline"** | Disconnect and reconnect the USB cable. Run `adb kill-server && adb start-server`. |
| **Command timeout** | The device screen may be off or locked. Wake the device with `adb shell input keyevent KEYCODE_WAKEUP` and unlock it. |
| **API key error** | Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`. See [Model Configuration](https://midscenejs.com/zh/model-common-config.html). |
| **`@midscene/*` dependency version is outdated** | Check local versions with `npm ls @midscene/android @midscene/core @midscene/shared` (or `pnpm why @midscene/android`). Compare with latest versions using `npm view @midscene/android version`, `npm view @midscene/core version`, and `npm view @midscene/shared version`. Upgrade as needed (`npm i @midscene/android@latest @midscene/core@latest @midscene/shared@latest`). |
| **Wrong device targeted** | If multiple devices are connected, use `--deviceId <id>` flag with the `connect` command. |
