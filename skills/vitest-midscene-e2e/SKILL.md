---
name: vitest-midscene-e2e
description: "Enhances Vitest with Midscene for AI-powered UI testing across Web (Playwright), Android (ADB), and iOS (WDA). Scaffolds new projects, converts existing projects, and creates/updates/debugs/runs E2E tests using natural-language UI interactions. Triggers: write test, add test, create test, update test, fix test, debug test, run test, e2e test, midscene test, new project, convert project, init project, 写测试, 加测试, 创建测试, 更新测试, 修复测试, 调试测试, 运行测试, 新建工程, 转化工程."
user-invocable: true
argument-hint: "[create|update|run|init] <feature-name>"
---

# Vitest Midscene E2E

## Modules

| Module | Role |
|--------|------|
| **Vitest** | TypeScript test framework. Provides `describe`/`it`/`expect`/hooks for test organization, assertions, and lifecycle. |
| **Midscene** | AI-driven UI automation. Interacts with UI elements via natural language — no fragile selectors. Core API: `aiAct`. |

Supported platforms:

- **Web** — `WebTest` (Playwright Chromium): `ctx.agent` + `ctx.page`
- **Android** — `AndroidTest` (ADB + scrcpy): `ctx.agent` only
- **iOS** — `IOSTest` (WebDriverAgent): `ctx.agent` only

## Workflow

### Step 1: Clone boilerplate & ensure project ready

```bash
bash scripts/clone-boilerplate.sh
```

The boilerplate at `~/.midscene/boilerplate/vitest-all-platforms-demo/` is the canonical reference for project structure, configs, platform context classes, and test conventions. Compare the current project against it. If anything is missing, ask the user which platform(s) they need (Web / Android / iOS), then fill in what's missing using the boilerplate as the target state. Only include files for the requested platform(s). Do NOT overwrite existing configs or files. Copy `.env.example` from the boilerplate as `.env` if it doesn't exist, and prompt the user to fill in the env vars.

### Step 2: Read the **Midscene Agent API** section below before writing tests

It contains mandatory rules for using `aiAct` — the primary API for all UI operations. Do NOT skip this step.

### Step 3: Create, update, or run tests

Use the boilerplate's `e2e/` directory and `src/context/` as reference for patterns and conventions. Before running tests, ensure dependencies are installed and `.env` is configured. When debugging failures, check [troubleshooting.md](./references/troubleshooting.md).

## Midscene Agent API

`ctx.agent` is a platform-specific agent instance. All methods return Promises.

- **Web**: `PlaywrightAgent` from `@midscene/web/playwright`
- **Android**: `AndroidAgent` from `@midscene/android`
- **iOS**: `IOSAgent` from `@midscene/ios`

All three agents share the same AI methods below.

### Mandatory Rule: Use `aiAct` for User-Described Steps

> **When the user describes a UI action or state confirmation in natural language, you MUST use `aiAct` to implement it.** Do NOT decompose user instructions into `aiTap`/`aiInput`/`aiAssert` or other fine-grained APIs. Pass the user's intent directly to `aiAct` and let Midscene's AI handle the planning and execution.

```typescript
// User says: "type iPhone in the search box and click search"

// WRONG — manually decomposing into fine-grained APIs
await ctx.agent.aiInput('search box', { value: 'iPhone' });
await ctx.agent.aiTap('search button');

// CORRECT — pass intent directly to aiAct
await ctx.agent.aiAct('type "iPhone" in the search box, then click the search button');
```

Assertions, data extraction, and waiting should also be done via `aiAct` — it handles all of these. Do NOT use `aiAssert`, `aiQuery`, `aiWaitFor`, `aiTap`, or `aiInput` separately.

### aiAct(taskPrompt, opt?) — Primary API

**`aiAct` is the primary API for all UI operations and state confirmations.** It accepts natural language instructions and autonomously plans and executes multi-step interactions.

```typescript
// UI operations
await ctx.agent.aiAct('type "iPhone" in the search box, then click the search button');
await ctx.agent.aiAct('hover over the user avatar in the top right');

// State confirmations / assertions — also use aiAct
await ctx.agent.aiAct('verify the page shows "Login successful"');
await ctx.agent.aiAct('verify the error message is visible');
```

### Prompt-driven File Uploads (Web only)

When an `aiAct` prompt asks Midscene to upload files, pass `fileChooserAllowedDir` explicitly. Use the smallest directory containing that test case's fixtures, and refer to files relative to it in the prompt. Do not use the project root or a home directory. Replace `./fixtures` below with the fixture directory relative to the test process working directory.

```typescript
await ctx.agent.aiAct(
  'click the upload button and upload avatar.png',
  { fileChooserAllowedDir: './fixtures' },
);
```

**Phase splitting:** If the task prompt is too long or covers multiple distinct stages, split it into separate `aiAct` calls — one per phase. Each phase should be a self-contained logical step, and all phases combined must match the user's original intent.

```typescript
// Incorrect — prompt spans multiple pages and too many steps, AI may lose context mid-way
await ctx.agent.aiAct('click the settings button in the top nav, go to settings page, find personal info and click into it, change email to "test@example.com", change phone to "13800000000", click save, wait for success');

// Correct — split by page/stage boundary, each phase stays within one logical context
await ctx.agent.aiAct('click the settings button in the top nav, go to settings page, find personal info and click into it');
await ctx.agent.aiAct('change email to "test@example.com", change phone to "13800000000", click save');
await ctx.agent.aiAct('verify the save success message appears');
```

> `aiAction` is deprecated. Use `aiAct` or `ai` instead.

### Common Mistakes

- **Vague locators** — `'button'` is ambiguous; use `'the blue "Submit" button at the top of the page'`
- **Deprecated `aiAction`** — use `aiAct` instead
- **Ambiguous multi-element targets** — specify row/position: `'the delete button in the first product row'`

### Agent Configuration — `aiActionContext`

`aiActionContext` is a system prompt string appended to all AI actions performed by the agent. Use it to define the AI's role and expertise.

```typescript
// Set via agentOptions in setup()
const ctx = WebTest.setup('https://example.com', {
  agentOptions: {
    aiActionContext: 'You are a Web UI testing expert.',
  },
});
```

**Good examples:**
- `'You are a Web UI testing expert.'`
- `'You are an Android app testing expert who is familiar with Chinese UI.'`

**Bad examples:**
- `'Click the login button.'` — specific actions belong in `aiAct()`, not `aiActionContext`
- `'The page is in Chinese.'` — this is page description, not a system prompt

### How to Look Up More

1. In `node_modules/@midscene/web`, `node_modules/@midscene/android`, and `node_modules/@midscene/ios`, find the type definitions for the agent classes
2. If types are not enough, follow the source references in the `.d.ts` files to read the implementation code in `node_modules`
3. Download https://midscenejs.com/llms.txt, then use `grep` to search for the API or concept you need (the file is large, do not read it in full)
