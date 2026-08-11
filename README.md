<p align="center">
  <img alt="Midscene.js" width="260" src="https://github.com/user-attachments/assets/2e0b2957-2ab7-46ec-a4a1-21459e0a7e48">
</p>

# Midscene Skills

<strong> Vision-driven cross-platform automation </strong>

- Natural-language driven UI control
- Built on [Midscene.js](https://midscenejs.com)'s vision-based automation capabilities — Operates entirely from screenshots, making cross-platform support reliable
- This repository contains Skills for the following platforms:
  - Browser (Puppeteer, headless Chrome; also supports CDP connection): [`skills/browser`](skills/browser)
  - Chrome Bridge (user's own Chrome browser): [`skills/chrome-bridge`](skills/chrome-bridge)
  - Desktop (macOS, Windows, Linux): [`skills/computer-automation`](skills/computer-automation)
  - Android (controlled via ADB): [`skills/android-automation`](skills/android-automation)
  - iOS (controlled via WebDriverAgent): [`skills/ios-automation`](skills/ios-automation)
  - HarmonyOS (controlled via HDC): [`skills/harmony-automation`](skills/harmony-automation)
  - Vitest + Midscene E2E (scaffold, convert, and manage AI-powered E2E tests for Web, Android, iOS): [`skills/vitest-midscene-e2e`](skills/vitest-midscene-e2e)
  - Midscene report analysis (validate passed and failed results, identify false-pass/false-fail outcomes, and trace root causes): [`skills/midscene-report-analysis`](skills/midscene-report-analysis)


## Safety Warning

⚠️ AI-driven UI automation may produce unpredictable results since it can control EVERYTHING on the screen. Please evaluate the risks carefully before use.

## Installation

Make sure you have [Node.js](https://nodejs.org) installed.

Then install the skills:

```bash
# General installation
npx skills add web-infra-dev/midscene-skills

# Claude Code 
npx skills add web-infra-dev/midscene-skills -a claude-code

# OpenClaw
npx skills add web-infra-dev/midscene-skills -a openclaw
```

## Model Setup

Midscene requires models with strong **visual grounding** capabilities (accurate UI element localization from screenshots).  
Because of this, you need to prepare model access and configuration separately from skill installation.

Make sure these environment variables are available in your system. You can also define them in a `.env` file in the current directory, and Midscene will load them automatically:

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

Example: Qwen3-VL

```bash
MIDSCENE_MODEL_API_KEY="your-openrouter-api-key"
MIDSCENE_MODEL_NAME="qwen/qwen3-vl-235b-a22b-instruct"
MIDSCENE_MODEL_BASE_URL="https://openrouter.ai/api/v1"
MIDSCENE_MODEL_FAMILY="qwen3-vl"
```

Example: Doubao Seed 1.6

```bash
MIDSCENE_MODEL_API_KEY="your-doubao-api-key"
MIDSCENE_MODEL_NAME="doubao-seed-1-6-250615"
MIDSCENE_MODEL_BASE_URL="https://ark.cn-beijing.volces.com/api/v3"
MIDSCENE_MODEL_FAMILY="doubao-vision"
```

Commonly used models: Doubao Seed 1.6, Qwen3-VL, Zhipu GLM-4.6V, Gemini-3-Pro, Gemini-3-Flash.

Model setup docs:

- Midscene model strategy: https://midscenejs.com/model-strategy
- Qwen3-VL: https://midscenejs.com/model-common-config#qwen3-vl
- Gemini-3-Flash: https://midscenejs.com/model-common-config#gemini-3-pro-and-gemini-3-flash
- Doubao Seed 1.6: https://midscenejs.com/model-common-config


## Use

In your chatbot or coding agent, you can say:

```
Use Midscene computer skill to open the Keynote app and create a new presentation.
```

```
Use Midscene browser skill to open the Google search page and search for "Midscene".
```

```
Use Midscene report analysis skill to analyze /absolute/path/to/midscene-report.html.
```

## Issues

For bug reports, feature requests, and discussions, please visit the main Midscene repository: https://github.com/web-infra-dev/midscene/issues

## License

MIT
