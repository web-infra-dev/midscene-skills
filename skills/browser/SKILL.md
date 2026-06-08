---
name: browser-automation
description: |
  Vision-driven browser automation using Midscene. Operates from screenshots — no DOM or accessibility labels needed.

  Runs in headless Puppeteer — does NOT take over the user's mouse or keyboard.
  Also supports CDP mode and Bridge mode to connect to an existing Chrome.

  Use this skill when the user wants to:
  - Browse, navigate, or open web pages
  - Scrape, extract, or collect data from websites
  - Fill out forms, click buttons, or interact with web elements
  - Verify, validate, test, or QA frontend UI behavior
  - Take screenshots of web pages
  - Automate multi-step web workflows
  - Test what was just built, see if it works in browser
  - Connect to Chrome via CDP, DevTools Protocol, or remote debugging
  - Connect to user's Chrome browser, control my browser, operate my Chrome

  Powered by Midscene.js (https://midscenejs.com)
allowed-tools:
  - Bash
---

# Browser Automation

> **CRITICAL RULES — VIOLATIONS WILL BREAK THE WORKFLOW:**
>
> 1. **Never run midscene commands in the background.** Each command must run synchronously so you can read its output (especially screenshots) before deciding the next action. Background execution breaks the screenshot-analyze-act loop.
> 2. **Run only one midscene command at a time.** Wait for the previous command to finish, read the screenshot, then decide the next action. Never chain multiple commands together.
> 3. **Allow enough time for each command to complete.** Midscene commands involve AI inference and screen interaction, which can take longer than typical shell commands. A typical command needs about 1 minute; complex `act` commands may need even longer.
> 4. **Always report task results before finishing.** After completing the automation task, you MUST proactively summarize the results to the user — including key data found, actions completed, screenshots taken, and any relevant findings. Never silently end after the last automation step; the user expects a complete response in a single interaction.

Automate web browsing using `npx -y @midscene/web@1`. By default, launches a headless Chrome via Puppeteer that **persists across CLI calls** — no session loss between commands. Also supports **CDP mode** and **Bridge mode** to connect to an existing Chrome browser.

## What `act` Can Do

Inside a single `act` call in the browser, Midscene can click, right-click, double-click, hover, type or clear text, press keys, scroll, drag, long-press, and continue through multi-step page flows based on what is currently visible. When touch input is enabled, it can also handle swipe- or pinch-style interactions on touch-oriented pages.

## When to Use

This skill has three modes. Choose based on the user's intent:

### Mode Selection Guide

| Mode | When to use | How it works |
|------|------------|-------------|
| **Puppeteer (default)** | User wants to browse a URL, scrape data, test UI — no need for their own browser | Launches a new headless Chrome, isolated from user's browser |
| **CDP mode** | User says "connect to my Chrome", "control my browser", "CDP", "remote debugging", or wants to operate their existing browser. Also use when the task **implicitly requires login state** (e.g., "check my orders", "open my dashboard", "look at my account") | Connects to user's Chrome via DevTools Protocol. Requires remote debugging enabled (`chrome://inspect` > "Allow remote debugging"). No extension needed |
| **Bridge mode** | User explicitly mentions "bridge", "extension", or has Midscene Chrome Extension installed and prefers to use it | Connects to user's Chrome via the Midscene Chrome Extension |

**CDP vs Bridge**: Both control the user's real Chrome with login sessions preserved. CDP only needs a Chrome setting toggle; Bridge needs a Chrome Extension installed. If the user doesn't specify, prefer **CDP mode** as it has fewer prerequisites.

### Precheck: detect available CDP target

Before using CDP mode, run a quick precheck to verify Chrome's remote debugging port is reachable. This avoids long timeouts when the user hasn't enabled remote debugging.

```bash
# CDP precheck (port 9222, 2s timeout) — returns "101" if Chrome is listening for DevTools
curl -s --max-time 2 -o /dev/null -w "%{http_code}" -H "Upgrade: websocket" -H "Connection: Upgrade" -H "Sec-WebSocket-Version: 13" -H "Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==" http://127.0.0.1:9222/devtools/browser
```

**How to use the precheck result:**
- Returns `101` → CDP mode is available, use `--cdp`
- Fails (curl exit 7 or HTTP `000`) → Chrome is not running with remote debugging enabled. Start Chrome with `--remote-debugging-port=9222`, wait 2-3 seconds, then re-run the precheck. If it still fails, fall back to Puppeteer mode (or use Bridge mode if the Midscene Chrome Extension is installed).

> **Bridge mode has no usable precheck.** Despite port 3766 being involved, the bridge server is started **by the CLI** (`npx ... --bridge connect` opens 3766 itself); the extension is the **client** that hooks in. Checking 3766 before the CLI runs always returns nothing. Just run `--bridge connect` directly — its first log line (`waiting for bridge to connect...` → `one client connected`) tells you whether the extension picked it up.

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

## CDP Mode (Connect to Existing Browser)

Use CDP mode to control the user's existing Chrome browser. The default CDP endpoint is `ws://127.0.0.1:9222/devtools/browser` (port 9222 is Chrome's standard remote debugging port). If the user specifies a different port, replace 9222 accordingly.

Add `--cdp <ws-endpoint>` to every command:

```bash
npx -y @midscene/web@1 connect --cdp ws://127.0.0.1:9222/devtools/browser --url https://example.com
npx -y @midscene/web@1 act --cdp ws://127.0.0.1:9222/devtools/browser --prompt "click the button"
npx -y @midscene/web@1 take_screenshot --cdp ws://127.0.0.1:9222/devtools/browser
npx -y @midscene/web@1 disconnect --cdp ws://127.0.0.1:9222/devtools/browser
```

### Important notes for CDP mode

- The browser is managed externally — `disconnect` releases the connection but does NOT close the browser. There is no `close` command in CDP mode.
- In CDP mode, `connect --url` navigates the **existing active tab** instead of opening a new tab.
- `connect` without `--url` attaches to the current active tab without navigating.
- If connection fails, ask the user to enable remote debugging: open `chrome://inspect` in Chrome and turn on "Allow remote debugging".

## Bridge Mode (Connect via Chrome Extension)

Use Bridge mode when the user explicitly mentions "bridge", "extension", or has the Midscene Chrome Extension installed. Add `--bridge` to every command:

```bash
npx -y @midscene/web@1 --bridge connect --url https://example.com
npx -y @midscene/web@1 --bridge act --prompt "click the button"
npx -y @midscene/web@1 --bridge take_screenshot
npx -y @midscene/web@1 --bridge disconnect
```

### Important notes for Bridge mode

- The user must have Chrome (or any Chromium-based browser that supports Chrome extensions, e.g. Edge, Arc, Dia) open with the Midscene Extension installed and enabled.
- Install the extension from Chrome Web Store: https://chromewebstore.google.com/detail/midscenejs/gbldofcpkknbggpkmbdaefngejllnief
- **Handshake direction**: the **CLI is the server** (listens on `127.0.0.1:3766`); the **extension is the client** that connects to it. The "Listening" status shown in the extension panel means "ready to connect to a CLI", not that the extension itself opened a port. Practically: launch `--bridge connect` first; the extension will hook in within a second or two.
- **Active tab must be a normal web page.** The extension cannot operate `chrome://`, `chrome-extension://`, the Chrome Web Store, or other privileged URLs. If the active tab is one of those, the CLI will return `Cannot access a chrome:// URL`. Ask the user to switch to a regular `http(s)://` tab before reconnecting.
- `connect` without `--url` attaches to the current active tab; `connect --url <href>` navigates that tab. The CLI does not open new tabs in bridge mode.
- `disconnect` only closes the CLI-side bridge connection, not the browser or tabs.
- If the extension is not installed, guide the user to install it or suggest switching to CDP mode instead.
- See the [Bridge Mode documentation](https://midscenejs.com/bridge-mode-by-chrome-extension.html).

## Commands

### Connect to a Web Page

```bash
npx -y @midscene/web@1 connect --url https://example.com
```

### Take Screenshot

```bash
npx -y @midscene/web@1 take_screenshot
```

After taking a screenshot, read the saved image file to understand the current page state before deciding the next action.

### Perform Action

Use `act` to interact with the page and get the result. It autonomously handles all UI interactions internally — clicking, typing, scrolling, hovering, waiting, and navigating — so you should give it complex, high-level tasks as a whole rather than breaking them into small steps. Describe **what you want to do and the desired effect** in natural language:

```bash
# specific instructions
npx -y @midscene/web@1 act --prompt "click the Login button and fill in the email field with 'user@example.com'"
npx -y @midscene/web@1 act --prompt "scroll down and click the Submit button"

# or target-driven instructions
npx -y @midscene/web@1 act --prompt "click the country dropdown and select Japan"
```

### Assert Current Page State

Use `assert` to verify that the current page satisfies a natural language condition. It does not perform UI actions; it checks the visible page state and passes only when the assertion is true. Use this for validation, QA checks, and final state verification after `act`.

```bash
npx -y @midscene/web@1 assert --prompt "there is a login button visible"
npx -y @midscene/web@1 assert --prompt "the checkout page shows the order total and a Pay button"
```

In CDP or Bridge mode, pass the same connection flags you use for other commands:

```bash
npx -y @midscene/web@1 assert --cdp ws://127.0.0.1:9222/devtools/browser --prompt "the dashboard is loaded"
npx -y @midscene/web@1 --bridge assert --prompt "the profile page shows the user's avatar"
```

By default a failed assertion throws an AI-generated reason. Pass `--message` to throw a custom error message instead, which is useful for surfacing the intended outcome in QA and CI logs.

```bash
npx -y @midscene/web@1 assert \
  --prompt "the checkout page shows an order confirmation" \
  --message "the order should be confirmed after clicking Pay"
```

When the assertion needs to compare against a reference image (icon, logo, screenshot), pass `--image` for the URL/path and `--image-name` for its display name. Each `--image` may be an http(s) link, a `data:` URI, or a local file path. Repeat both flags in matching order when you need to attach more than one image. Add `--convertHttpImage2Base64 true` when the model cannot reach the URL directly. Requires `@midscene/web@1.9.0+`.

```bash
npx -y @midscene/web@1 assert \
  --prompt "the page shows the same logo as the reference image" \
  --image "https://github.githubassets.com/assets/GitHub-Mark-ea2971cee799.png" \
  --image-name "logo" \
  --convertHttpImage2Base64 true

# or with a local file
npx -y @midscene/web@1 assert \
  --prompt "the visible header matches the supplied screenshot" \
  --image "./fixtures/header.png" \
  --image-name "header"

# multiple reference images — pair --image and --image-name by order
npx -y @midscene/web@1 assert \
  --prompt "the page shows both the logo and the header" \
  --image "./fixtures/logo.png"   --image-name "logo" \
  --image "./fixtures/header.png" --image-name "header"
```

### Use a Reference Image for Precise Targeting

When the user provides a screenshot, icon, logo, or reference image and wants an exact visual match, prefer `tap --locate` instead of a generic `act --prompt`. Pass `--locate` as JSON. The `prompt` describes the target, `images` supplies named reference images, and `convertHttpImage2Base64: true` is useful when the image URL may not be directly accessible to the model.

```bash
npx -y @midscene/web@1 tap --locate '{
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

Disconnect from the page but keep the browser running:

```bash
npx -y @midscene/web@1 disconnect
```

### Close Browser

Close the browser completely when finished (Puppeteer mode only):

```bash
npx -y @midscene/web@1 close
```


### Consume Report Files

The generated HTML report is recommended for human reading first. It includes step-by-step execution details and replay videos for each operation, which makes it much easier to understand what happened and troubleshoot problems.

If another skill or tool needs to consume the report, first convert it with `report-tool` from the same platform CLI package. Prefer Markdown for LLM-based workflows. Use JSON when the report needs to be processed programmatically.

```bash
npx -y @midscene/web@1 report-tool --action to-markdown --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-markdown
npx -y @midscene/web@1 report-tool --action split --htmlPath ./midscene_run/report/.../index.html --outputDir ./output-data
```

## Workflow Pattern

The browser **persists across CLI calls** via a background Chrome process. Follow this pattern:

1. **Connect** to a URL to open a new tab
2. **Take screenshot** to see the current state, make sure the page is loaded.
3. **Execute action** using `act` to perform the desired action or target-driven instructions, and use `assert` when you need to verify the resulting page state.
4. **Close** the browser when done (or **disconnect** to keep it for later)
5. **Report results** — summarize what was accomplished, present key findings and data extracted during the task, and list any generated files (screenshots, logs, etc.) with their paths

## Best Practices

1. **Always connect first**: Navigate to the target URL with `connect --url` before any interaction.
2. **Inspect visible state**: After navigation or actions that trigger page changes, take a screenshot and read it before deciding the next step.
3. **Use natural, specific prompts**: Describe visible UI and desired outcomes, such as `"click the blue Submit button in the contact form"`, not selectors like `"#submit"`.
4. **Batch related operations into a single `act` command**: For example, fill the email and password fields, then click Log In in one prompt. Use separate commands when you need to inspect the intermediate state.
5. **Use `assert` for verification**: Check observable page state with `assert --prompt "..."`, such as `"the success toast is visible"` or `"the cart total is $42.00"`.
6. **Prefer `tap --locate` when a reference image is provided**: If the user shares a screenshot, icon, or logo and wants that exact visual target, use `tap --locate` with a multimodal `locate` JSON object such as `{ "prompt": "...", "images": [...] }` instead of relying only on `act --prompt`.

**Example — Dropdown selection:**

```bash
npx -y @midscene/web@1 act --prompt "click the country dropdown and select Japan"
npx -y @midscene/web@1 take_screenshot
```

**Example — Form interaction:**

```bash
npx -y @midscene/web@1 act --prompt "fill in the email field with 'user@example.com' and the password field with 'pass123', then click the Log In button"
npx -y @midscene/web@1 take_screenshot
```

## Improve Precision (Deep Locate / Deep Think)

Two optional global flags help when Midscene struggles with a task. Put them anywhere in the command (before or after the sub-command); once set, the relevant operations use them by default, so you don't pass a per-call parameter.

- `--deep-locate` — spends an extra round of visual reasoning to pinpoint the target element. Use it when an action interacts with the wrong spot (location drift / offset). It applies to every operation that locates an element, including `tap --locate` and the locating that happens inside `act`.
- `--deep-think` — plans `act` with deeper reasoning (richer context and sub-goal decomposition). Use it for complex, multi-step `act` instructions; it only affects planning.

Both trade a little speed for better results, and you can combine them.

```bash
# more accurate element location (helps act's internal locating too)
npx -y @midscene/web@1 act --deep-locate --prompt "click the tiny gear icon in the top-right toolbar"

# deeper planning for a complex, multi-step act
npx -y @midscene/web@1 act --deep-think --prompt "complete the multi-step checkout form"

# combine both
npx -y @midscene/web@1 act --deep-locate --deep-think --prompt "open the settings menu, go to Preferences, and enable dark mode"
```

In CDP or Bridge mode, keep your usual `--cdp` / `--bridge` flags alongside these.

## Troubleshooting

### Connection Failures
- Ensure Chrome/Chromium is installed on the system (Puppeteer downloads its own by default).
- Check that no firewall blocks local Chrome debugging ports.

### API Key Errors
- Check `.env` file contains `MIDSCENE_MODEL_API_KEY=<your-key>`.
- Verify the key is valid for the configured model provider.

### Timeouts
- Web pages may take time to load. After connecting, take a screenshot to verify readiness before interacting.
- For slow pages, wait briefly between steps.

### `@midscene/*` Dependency Version Outdated
- Check local versions: `npm ls @midscene/web @midscene/core @midscene/shared` (or `pnpm why @midscene/web`).
- Check latest versions: `npm view @midscene/web version`, `npm view @midscene/core version`, `npm view @midscene/shared version`.
- Upgrade dependencies: `npm i @midscene/web@latest @midscene/core@latest @midscene/shared@latest`.

### Screenshots Not Displaying
- The screenshot path is an absolute path to a local file. Use the Read tool to view it.
