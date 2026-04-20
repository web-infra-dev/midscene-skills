---
name: harmonyos-device-automation
description: >
  Vision-driven HarmonyOS NEXT device automation using Midscene.
  Operates entirely from screenshots — no DOM or accessibility labels required. Can interact with all visible elements on screen regardless of technology stack.
  Control HarmonyOS devices with natural language commands via HDC.
  Perform taps, swipes, text input, app launches, screenshots, and more.

  Trigger keywords: harmony, harmonyos, 鸿蒙, hdc, huawei device, harmony app, harmony automation, harmony phone, harmony tablet,
  test harmony app, verify on harmonyos, QA on 鸿蒙, check the app on harmony, test on huawei device,
  see if the app works on harmony, end-to-end test on harmonyos, visual verification on 鸿蒙

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# HarmonyOS Device Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.

Automate HarmonyOS NEXT devices using `npx -y @midscene/harmony@1`. Each CLI command maps directly to an MCP tool — you (the AI agent) act as the brain, deciding which actions to take based on screenshots.

## What `act` Can Do

Inside a single `act` call on HarmonyOS, Midscene can tap, double-tap, long-press, type, clear text, scroll, drag items, press keys, and use system navigation such as Back, Home, or recent apps while working from the current visible screen. Two-finger zoom is not available because the underlying HarmonyOS automation layer does not expose multi-touch input.

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

## HDC Setup

HDC (HarmonyOS Device Connector) must be installed and accessible. Common setup:

- Install via [DevEco Studio](https://developer.huawei.com/consumer/cn/deveco-studio/)
- Or set `HDC_HOME` environment variable to point to the HDC directory

Verify HDC is working:

```bash
hdc version
hdc list targets
```

## Commands

### Connect to Device

```bash
npx -y @midscene/harmony@1 connect
npx -y @midscene/harmony@1 connect --deviceId 0123456789ABCDEF
```

### Launch an App or URL

Use the dedicated launch step when you want a deterministic starting point before the rest of the task:

```bash
npx -y @midscene/harmony@1 launch --uri com.huawei.hmos.settings
npx -y @midscene/harmony@1 launch --uri com.huawei.hmos.camera
npx -y @midscene/harmony@1 launch --uri https://www.example.com
```

### Run a Raw HarmonyOS Shell Command

Use this when the task needs lower-level device control that is not best expressed as a visible UI interaction:

```bash
npx -y @midscene/harmony@1 runhdcshell --command "hidumper -s RenderService -a screen"
```

This is forwarded to `hdc shell` on the connected device. In practice, the underlying command is `hdc -t <deviceId> shell hidumper -s RenderService -a screen`.

### Take Screenshot

```bash
npx -y @midscene/harmony@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current screen state before deciding the next action.

### Perform Action

Use `act` to interact with the device and get the result. It autonomously handles all UI interactions internally — tapping, typing, scrolling, swiping, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx -y @midscene/harmony@1 act --prompt "type hello world in the search field and press Enter"
npx -y @midscene/harmony@1 act --prompt "long press the message bubble and tap Delete in the popup menu"

# or target-driven instructions
npx -y @midscene/harmony@1 act --prompt "open Settings and navigate to Wi-Fi settings, tell me the connected network name"
```

### Use a Reference Image for Precise Targeting

When the user provides a screenshot, icon, logo, or reference image and wants an exact visual match, prefer `tap --locate` instead of a generic `act --prompt`. Pass `--locate` as JSON. The `prompt` describes the target, `images` supplies named reference images, and `convertHttpImage2Base64: true` is useful when the image URL may not be directly accessible to the model.

```bash
npx -y @midscene/harmony@1 tap --locate '{
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
npx -y @midscene/harmony@1 disconnect
```

### Consume Report Files

The generated HTML report is recommended for human reading first. It includes step-by-step execution details and replay videos for each operation, which makes it much easier to understand what happened and troubleshoot problems.

If another skill or tool needs to consume the report, first convert it with `report-tool` from the same platform CLI package. Prefer Markdown for LLM-based workflows. Use JSON when the report needs to be processed programmatically.

```bash
npx -y @midscene/harmony@1 report-tool --action to-markdown --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-markdown
npx -y @midscene/harmony@1 report-tool --action split --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-data
```

## Workflow Pattern

Since CLI commands are stateless between invocations, follow this pattern:

1. **Connect** to establish a session
2. **Launch the target app and take screenshot** to see the current state, make sure the app is launched and visible on the screen.
3. **Execute action** using `act` to perform the desired action or target-driven instructions.
4. **Disconnect** when done

## Best Practices

1. **Bring the target app to the foreground before using this skill**: For best efficiency, launch the app using HDC (e.g., `hdc shell aa start -a EntryAbility -b <bundleName>`) **before** invoking any midscene commands. Then take a screenshot to confirm the app is actually in the foreground. Only after visual confirmation should you proceed with UI automation using this skill. HDC commands are significantly faster than using midscene to navigate to and open apps.
2. **Be specific about UI elements**: Instead of vague descriptions, provide clear, specific details. Say `"the Wi-Fi toggle switch on the right side"` instead of `"the toggle"`.
3. **Describe locations when possible**: Help target elements by describing their position (e.g., `"the search icon at the top right"`, `"the third item in the list"`).
4. **Never run in background**: Every midscene command must run synchronously — background execution breaks the screenshot-analyze-act loop.
5. **Batch related operations into a single `act` command**: When performing consecutive operations within the same app, combine them into one `act` prompt instead of splitting them into separate commands. For example, "open Settings, tap Wi-Fi, and toggle it on" should be a single `act` call, not three. This reduces round-trips, avoids unnecessary screenshot-analyze cycles, and is significantly faster.
6. **Summarize report files after completion**: After finishing the automation task, collect and summarize all report files (screenshots, logs, output files, etc.) for the user. Present a clear summary of what was accomplished, what files were generated, and where they are located, making it easy for the user to review the results.
7. **Prefer `tap --locate` when a reference image is provided**: If the user shares a screenshot, icon, or logo and wants that exact visual target, use `tap --locate` with a multimodal `locate` JSON object such as `{ "prompt": "...", "images": [...] }` instead of relying only on `act --prompt`.

**Example — App launch and interaction:**

```bash
hdc shell aa start -a EntryAbility -b com.huawei.hmos.settings
npx -y @midscene/harmony@1 connect
npx -y @midscene/harmony@1 take_screenshot
npx -y @midscene/harmony@1 act --prompt "scroll down the settings list and tap About device"
npx -y @midscene/harmony@1 take_screenshot
npx -y @midscene/harmony@1 disconnect
```

**Example — Form interaction:**

```bash
npx -y @midscene/harmony@1 act --prompt "fill in the username field with 'testuser' and the password field with 'pass123', then tap the Login button"
npx -y @midscene/harmony@1 take_screenshot
```

## Common HarmonyOS Bundle Names

| App | Bundle Name |
|-----|-------------|
| Settings | com.huawei.hmos.settings |
| Camera | com.huawei.hmos.camera |
| Gallery | com.huawei.hmos.photos |
| Calendar | com.huawei.hmos.calendar |
| Clock | com.huawei.hmos.clock |
| Calculator | com.huawei.hmos.calculator |
| Browser | com.huawei.hmos.browser |
| Weather | com.huawei.hmos.weather |

## Troubleshooting

| Problem | Solution |
|---|---|
| **HDC not found** | Install via DevEco Studio or set `HDC_HOME` environment variable. |
| **Device not listed** | Check USB connection, ensure USB debugging is enabled in Developer Options, and run `hdc list targets`. |
| **Command timeout** | The device screen may be off or locked. Wake the device and unlock it. |
| **API key error** | Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`. See [Model Configuration](https://midscenejs.com/zh/model-common-config.html). |
| **`@midscene/*` dependency version is outdated** | Check local versions with `npm ls @midscene/harmony @midscene/core @midscene/shared` (or `pnpm why @midscene/harmony`). Compare with latest via `npm view @midscene/harmony version`, `npm view @midscene/core version`, and `npm view @midscene/shared version`. Upgrade if needed: `npm i @midscene/harmony@latest @midscene/core@latest @midscene/shared@latest`. |
| **Wrong device targeted** | If multiple devices are connected, use `--deviceId <id>` flag with the `connect` command. |
