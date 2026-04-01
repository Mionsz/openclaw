# Repository Guidelines

- Repo: [[cc](https://github.com/openclaw/openclaw)](https://github.com/openclaw/openclaw)
- In chat replies, file references must be repo-root relative only (example: `src/telegram/index.ts:80`); never absolute paths or `~/...`.
- Do not edit files covered by security-focused `CODEOWNERS` rules unless a listed owner explicitly asked for the change or is already reviewing it with you. Treat those paths as restricted surfaces, not drive-by cleanup.

## Project Structure & Module Organization

- Source code: `src/` (CLI wiring in `src/cli`, commands in `src/commands`, web provider in `src/provider-web.ts`, infra in `src/infra`, media pipeline in `src/media`).
- Tests: colocated `*.test.ts`.
- Docs: `docs/` (images, queue, Pi config). Built output lives in `dist/`.
- Nomenclature: use "plugin" / "plugins" in docs, UI, changelogs, and contributor guidance. The bundled workspace plugin tree remains the internal package layout to avoid repo-wide churn from a rename.
- Bundled plugin naming: for repo-owned workspace plugins, keep the canonical plugin id aligned across `openclaw.plugin.json:id`, the default workspace folder name, and package names anchored to the same id (`@openclaw/<id>` or approved suffix forms like `-provider`, `-plugin`, `-speech`, `-sandbox`, `-media-understanding`). Keep `openclaw.install.npmSpec` equal to the package name and `openclaw.channel.id` equal to the plugin id when present. Exceptions must be explicit and covered by the repo invariant test.
- Plugins: live in the bundled workspace plugin tree (workspace packages). Keep plugin-only deps in the extension `package.json`; do not add them to the root `package.json` unless core uses them. Install runs `npm install --omit=dev` in plugin dir; runtime deps must live in `dependencies`. Avoid `workspace:*` in `dependencies` (npm install breaks); put `openclaw` in `devDependencies` or `peerDependencies` instead (runtime resolves `openclaw/plugin-sdk` via jiti alias).
- Import boundaries: extension production code should treat `openclaw/plugin-sdk/*` plus local `api.ts` / `runtime-api.ts` barrels as the public surface. Do not import core `src/**`, `src/plugin-sdk-internal/**`, or another extension's `src/**` directly.
- Installers served from `https://openclaw.ai/*`: live in the sibling repo `../openclaw.ai` (`public/install.sh`, `public/install-cli.sh`, `public/install.ps1`).
- Messaging channels: always consider **all** built-in + extension channels when refactoring shared logic (routing, allowlists, pairing, command gating, onboarding, docs).
  - Core channel docs: `docs/channels/`
  - Core channel code: `src/telegram`, `src/discord`, `src/slack`, `src/signal`, `src/imessage`, `src/web` (WhatsApp web), `src/channels`, `src/routing`
  - Bundled plugin channels: the workspace plugin tree (for example Matrix, Zalo, ZaloUser, Voice Call)
- When adding channels/plugins/apps/docs, update `.github/labeler.yml` and create matching GitHub labels (use existing channel/plugin label colors).

## Architecture Boundaries

- Start here for the repo map:
  - bundled workspace plugin tree = bundled plugins and the closest example surface for third-party plugins
  - `src/plugin-sdk/*` = the public plugin contract that extensions are allowed to import
  - `src/channels/*` = core channel implementation details behind the plugin/channel boundary
  - `src/plugins/*` = plugin discovery, manifest validation, loader, registry, and contract enforcement
  - `src/gateway/protocol/*` = typed Gateway control-plane and node wire protocol
- Progressive disclosure lives in local boundary guides:
  - bundled-plugin-tree `AGENTS.md`
  - `src/plugin-sdk/AGENTS.md`
  - `src/channels/AGENTS.md`
  - `src/plugins/AGENTS.md`
  - `src/gateway/protocol/AGENTS.md`
- Plugin and extension boundary (docs: `docs/plugins/building-plugins.md`, `docs/plugins/architecture.md`, `docs/plugins/sdk-overview.md`, `docs/plugins/sdk-entrypoints.md`, `docs/plugins/sdk-runtime.md`, `docs/plugins/manifest.md`, `docs/plugins/sdk-channel-plugins.md`, `docs/plugins/sdk-provider-plugins.md`; defs: `src/plugin-sdk/plugin-entry.ts`, `src/plugin-sdk/core.ts`, `src/plugin-sdk/provider-entry.ts`, `src/plugin-sdk/channel-contract.ts`, `scripts/lib/plugin-sdk-entrypoints.json`, `package.json`):
  - Extensions must cross into core only through `openclaw/plugin-sdk/*`, manifest metadata, and documented runtime helpers.
  - Core must not deep-import bundled plugin internals (`src/**`, `onboard.js`). Expose helpers through the plugin's `api.ts` or `src/plugin-sdk/<id>.ts`.
  - New plugin seams must be documented, backwards-compatible, versioned contracts. Third-party plugins exist — do not break them casually.
- Channel boundary (docs: `docs/plugins/sdk-channel-plugins.md`; defs: `src/channels/plugins/types.*.ts`, `src/plugin-sdk/channel-contract.ts`):
  - `src/channels/**` is core implementation. New plugin-facing seams go in the Plugin SDK, not channel internals.
- Provider/model boundary (docs: `docs/plugins/sdk-provider-plugins.md`, `docs/concepts/model-providers.md`; defs: `src/plugins/types.ts`, `src/plugin-sdk/provider-*.ts`):
  - Core owns the inference loop; provider plugins own provider-specific behavior through registration and typed hooks.
  - No ad hoc reads of `plugins.entries.<id>.config` from unrelated core code. Use generic seams (`resolveSyntheticAuth`, SDK facades, manifest metadata, auto-enable hooks) and honor plugin disablement + SecretRef semantics.
  - Vendor-owned tools/settings belong in the owning plugin, not core `tools.*` surfaces.
- Gateway protocol boundary (docs: `docs/gateway/protocol.md`, `docs/gateway/bridge-protocol.md`; defs: `src/gateway/protocol/schema.ts`, `src/gateway/protocol/index.ts`):
  - Protocol changes are contract changes. Prefer additive evolution; incompatible changes require versioning, docs, and client/codegen follow-through.
- Bundled plugin contract boundary (defs: `src/plugins/contracts/registry.ts`, `src/plugins/types.ts`, `src/plugins/public-artifacts.ts`):
  - Keep manifest metadata, runtime registration, SDK exports, and contract tests aligned. No hidden paths around declared interfaces.
- Extension test boundary:
  - Keep extension-owned coverage under the owning plugin package. Core tests consume bundled plugin behavior through `src/plugin-sdk/<id>.ts` or the plugin's `api.ts`, not private modules.

## Docs Linking (Mintlify)

- Docs are hosted on Mintlify (docs.openclaw.ai).
- Internal doc links in `docs/**/*.md`: root-relative, no `.md`/`.mdx` (example: `[Config](/configuration)`).
- When working with documentation, read the mintlify skill.
- For docs, UI copy, and picker lists, order services/providers alphabetically unless the section is explicitly describing runtime behavior (for example auto-detection or execution order).
- Section cross-references: use anchors on root-relative paths (example: `[Hooks](/configuration#hooks)`).
- Doc headings and anchors: avoid em dashes and apostrophes in headings because they break Mintlify anchor links.
- When the user asks for links, reply with full `https://docs.openclaw.ai/...` URLs (not root-relative).
- When you touch docs, end the reply with the `https://docs.openclaw.ai/...` URLs you referenced.
- README (GitHub): keep absolute docs URLs (`https://docs.openclaw.ai/...`) so links work on GitHub.
- Docs content must be generic: no personal device names/hostnames/paths; use placeholders like `user@gateway-host` and “gateway host”.

## Docs i18n (zh-CN)

- `docs/zh-CN/**` is generated; do not edit unless the user explicitly asks. See `docs/.i18n/README.md`.
- Pipeline: update English docs → adjust glossary (`docs/.i18n/glossary.zh-CN.json`) → run `scripts/docs-i18n`. Add glossary entries for new terms/titles before re-running.
- `pnpm docs:check-i18n-glossary` enforces glossary coverage. Translation memory: `docs/.i18n/zh-CN.tm.jsonl` (generated).
- If the pipeline is slow, ping @jospalmbier on Discord instead of hacking around it.

## Build, Test, and Development Commands

- Runtime baseline: Node **22+** (keep Node + Bun paths working).
- Install deps: `pnpm install`
- If deps are missing (for example `node_modules` missing, `vitest not found`, or `command not found`), run the repo’s package-manager install command (prefer lockfile/README-defined PM), then rerun the exact requested command once. Apply this to test/build/lint/typecheck/dev commands; if retry still fails, report the command and first actionable error.
- Pre-commit hooks: `prek install`. The hook runs the repo verification flow, including `pnpm check`. `FAST_COMMIT=1` skips `pnpm format` + `pnpm check` inside the hook only (does not change CI).
- Also supported: `bun install` (keep `pnpm-lock.yaml` + Bun patching in sync when touching deps/patches). Prefer Bun for TypeScript execution (scripts, dev, tests): `bun <file.ts>` / `bunx <tool>`.
- Run CLI in dev: `pnpm openclaw ...` (bun) or `pnpm dev`.
- Node remains supported for running built output (`dist/*`) and production installs.
- Mac packaging (dev): `scripts/package-mac-app.sh` defaults to current arch.
- Type-check/build: `pnpm build`
- TypeScript checks: `pnpm tsgo`
- Lint/format: `pnpm check`
- Local agent/dev shells default to lower-memory `OPENCLAW_LOCAL_CHECK=1` behavior for `pnpm tsgo` and `pnpm lint`; set `OPENCLAW_LOCAL_CHECK=0` in CI/shared runs.
- Format check: `pnpm format` (oxfmt --check)
- Format fix: `pnpm format:fix` (oxfmt --write)
- Gates (verification command sets that must be green):
  - **Local dev**: `pnpm check` (fast default loop; architecture policy guards are excluded).
  - **Landing** (`main`): `pnpm check` + `pnpm test` + `pnpm build` (when the touched surface can affect build output, packaging, lazy-loading, or published surfaces).
  - **CI**: whatever the workflow enforces (`check`, `check-additional`, `build-smoke`, release validation).
  - **Formatting**: pre-commit hook runs `pnpm format` before `pnpm check`. Run `pnpm format` explicitly for a standalone preflight.

- Tests: `pnpm test` (vitest); coverage: `pnpm test:coverage`
- Generated baseline artifacts live together under `docs/.generated/`.
- Config schema drift uses `pnpm config:docs:gen` / `pnpm config:docs:check`.
- Plugin SDK API drift uses `pnpm plugin-sdk:api:gen` / `pnpm plugin-sdk:api:check`.
- If you change config schema/help or the public Plugin SDK surface, update the matching baseline artifact and keep the two drift-check flows adjacent in scripts/workflows/docs guidance rather than inventing a third pattern.
- For narrowly scoped changes, prefer narrowly scoped tests that directly validate the touched behavior. If no meaningful scoped test exists, say so explicitly and use the next most direct validation available.
- Verification modes for work on `main`:
  - Default mode: `main` is relatively stable. Favor `pnpm check` and `pnpm test` near the final rebase/push point. Count pre-commit hook coverage when it already verified the tree; avoid rerunning identical checks just for ceremony.
  - Fast-commit mode: `main` is moving fast. Use `FAST_COMMIT=1 git commit ...` or `--no-verify` for intermediate commits after equivalent checks have already run locally. Verify the touched surface near the final landing point.
- Scoped tests prove the change itself but do not replace `pnpm test` as the default `main` landing bar.
- Hard gate: if the change can affect build output, packaging, lazy-loading/module boundaries, or published surfaces, `pnpm build` MUST pass before pushing `main`.
- Do not land changes with failing format, lint, type, build, or test checks when plausibly related to the touched surface. If unrelated failures exist on `origin/main`, state that clearly and ask before broadening scope.

## Coding Style & Naming Conventions

- Language: TypeScript (ESM). Prefer strict typing; avoid `any`.
- Formatting/linting via Oxlint and Oxfmt.
- Never add `@ts-nocheck` and do not add inline lint suppressions by default. Fix root causes first; only keep a suppression when the code is intentionally correct, the rule cannot express that safely, and the comment explains why.
- Do not disable `no-explicit-any`; prefer real types, `unknown`, or a narrow adapter/helper instead. Update Oxlint/Oxfmt config only when required.
- Prefer `zod` or existing schema helpers at external boundaries such as config, webhook payloads, CLI/JSON output, persisted JSON, and third-party API responses.
- Prefer discriminated unions when parameter shape changes runtime behavior.
- Prefer `Result<T, E>`-style outcomes and closed error-code unions for recoverable runtime decisions.
- Keep human-readable strings for logs, CLI output, and UI; do not use freeform strings as the source of truth for internal branching.
- Avoid `?? 0`, empty-string, empty-object, or magic-string sentinels that can change runtime meaning silently. For new optional/nullable fields that change behavior, prefer explicit unions. Do not branch on freeform `error: string` when a closed code union would work.
- Dynamic imports: do not mix `await import("x")` and static `import ... from "x"` for the same module in production code. Use a dedicated `*.runtime.ts` boundary for lazy loading. After touching lazy-loading/module boundaries, run `pnpm build` and check for `[INEFFECTIVE_DYNAMIC_IMPORT]` warnings.
- Extension package guardrails:
  - `openclaw/plugin-sdk/<subpath>` is the only public cross-package contract. If an extension needs a new seam, add a public subpath first.
  - Inside a bundled plugin package, do not use relative imports that resolve outside the package root; use `openclaw/plugin-sdk/<subpath>` instead.
  - Inside an extension, do not self-import via `openclaw/plugin-sdk/<extension>` from production files; route internal imports through `./api.ts` or `./runtime-api.ts`.
- No prototype mutation: never share class behavior via `applyPrototypeMixins`, `Object.defineProperty` on `.prototype`, or `Class.prototype` merges. Use explicit inheritance/composition. In tests, prefer per-instance stubs over prototype-level patching.
- Add brief code comments for tricky or non-obvious logic.
- Keep files concise; extract helpers instead of “V2” copies. Use existing patterns for CLI options and dependency injection via `createDefaultDeps`.
- Aim to keep files under ~700 LOC; guideline only (not a hard guardrail). Split/refactor when it improves clarity or testability.
- Naming: use **OpenClaw** for product/app/docs headings; use `openclaw` for CLI command, package/binary, paths, and config keys.
- Written English: use American spelling and grammar in code, comments, docs, and UI strings (e.g. "color" not "colour", "behavior" not "behaviour", "analyze" not "analyse").

## Testing Guidelines

- Framework: Vitest with V8 coverage thresholds (70% lines/branches/functions/statements).
- Naming: match source names with `*.test.ts`; e2e in `*.e2e.test.ts`.
- When tests need example Anthropic/OpenAI model constants, prefer `sonnet-4.6` and `gpt-5.4`; update older Anthropic/GPT examples when you touch those tests.
- Run `pnpm test` (or `pnpm test:coverage`) before pushing when you touch logic.
- Write tests to clean up timers, env, globals, mocks, sockets, temp dirs, and module state so `--isolate=false` stays green.
- Test performance guardrails:
  - Do not put `vi.resetModules()` plus `await import(...)` in `beforeEach`/per-test loops for heavy modules unless module state truly requires it. Prefer static imports or one-time `beforeAll` imports, then reset mocks/runtime state directly.
  - Inside an extension package, prefer a thin local seam (`./api.ts`, `./runtime-api.ts`) over direct `openclaw/plugin-sdk/*` imports for internal production code. Reach for direct `plugin-sdk/*` imports only when crossing a real package boundary.
  - Keep expensive runtime fallback work (snapshotting, migration, installs, bootstrap) behind dedicated `*.runtime.ts` boundaries so tests can mock the seam.
  - For import-only/runtime-wrapper tests, keep the wrapper lazy. Do not eagerly load heavy modules at module top level if the exported function can import them on demand.
- Run scoped tests via the wrapper: `pnpm test -- <path-or-filter> [vitest args...]`; do not use raw `pnpm vitest run ...` (it bypasses wrapper config/pool routing).
- Vitest constraints: `forks` pool only (no `threads`/`vmThreads`/`vmForks`), max 16 workers. For memory pressure: `OPENCLAW_TEST_PROFILE=serial OPENCLAW_TEST_SERIAL_GATEWAY=1 pnpm test`.
- Live tests (real keys): `OPENCLAW_LIVE_TEST=1 pnpm test:live` (OpenClaw-only) or `LIVE=1 pnpm test:live` (includes provider live tests). Docker: `pnpm test:docker:live-models`, `pnpm test:docker:live-gateway`. Onboarding Docker E2E: `pnpm test:docker:onboard`.
- `pnpm test:live` defaults quiet. Full logs: `OPENCLAW_LIVE_TEST_QUIET=0 pnpm test:live`. Full kit: `docs/help/testing.md`.
- Changelog: user-facing changes only. Append new entries to end of the target section (`### Changes` or `### Fixes`). One contributor mention per line (`Thanks @author`). Pure test changes generally need no changelog entry.

## Commit, PR & Git Guidelines

- Use `$openclaw-pr-maintainer` at `.agents/skills/openclaw-pr-maintainer/SKILL.md` for maintainer PR triage, review, close, search, landing workflows, auto-close labels, bug-fix evidence gates, and PR decision flow.
- `/landpr` lives in the global Codex prompts (`~/.codex/prompts/landpr.md`); when landing or merging any PR, always follow that `/landpr` process.
- Create commits with `scripts/committer "<msg>" <file...>`; avoid manual `git add`/`git commit` so staging stays scoped.
- Follow concise, action-oriented commit messages (e.g., `CLI: add verbose flag to send`).
- Group related changes; avoid bundling unrelated refactors.
- PR submission template (canonical): `.github/pull_request_template.md`
- Issue submission templates (canonical): `.github/ISSUE_TEMPLATE/`
- If `git branch -d/-D <branch>` is policy-blocked, delete the local ref directly: `git update-ref -d refs/heads/<branch>`.
- Agents MUST NOT: create/push merge commits on `main` (rebase instead); modify baseline, inventory, ignore, snapshot, or expected-failure files to silence checks without explicit approval.
- Bulk PR close/reopen safety: if a close action would affect more than 5 PRs, ask for explicit user confirmation first.

## Security, Releases & Configuration

- Web provider stores creds at `~/.openclaw/credentials/`; rerun `openclaw login` if logged out.
- Never commit or publish real phone numbers, videos, or live configuration values. Use obviously fake placeholders in docs, tests, and examples.
- Release flow: use the private [maintainer release docs](https://github.com/openclaw/maintainers/blob/main/release/README.md) for the actual runbook, `docs/reference/RELEASING.md` for the public release policy, and `$openclaw-release-maintainer` for the maintainership workflow.
- Use `$openclaw-ghsa-maintainer` at `.agents/skills/openclaw-ghsa-maintainer/SKILL.md` for GHSA advisory inspection, patch/publish flow, private-fork checks, and GHSA API validation.
- Release and publish remain explicit-approval actions even when using a skill.

## Local Runtime / Platform Notes

- Rebrand/migration issues or legacy config/service warnings: run `openclaw doctor` (see `docs/gateway/doctor.md`).
- Use `$openclaw-parallels-smoke` at `.agents/skills/openclaw-parallels-smoke/SKILL.md` for Parallels smoke, rerun, upgrade, debug, and result-interpretation workflows across macOS, Windows, and Linux guests.
- For the macOS Discord roundtrip deep dive, use the narrower `.agents/skills/parallels-discord-roundtrip/SKILL.md` companion skill.
- Never edit `node_modules` (global/Homebrew/npm/git installs too). Updates overwrite. Skill notes go in `tools.md` or `AGENTS.md`.
- If you need local-only `.agents` ignores, use `.git/info/exclude` instead of repo `.gitignore`.
- When adding a new `AGENTS.md` anywhere in the repo, also add a `CLAUDE.md` symlink pointing to it (example: `ln -s AGENTS.md CLAUDE.md`).
- CLI progress: use `src/cli/progress.ts` (`osc-progress` + `@clack/prompts` spinner); don’t hand-roll spinners/bars.
- Status output: keep tables + ANSI-safe wrapping (`src/terminal/table.ts`); `status --all` = read-only/pasteable, `status --deep` = probes.
- Gateway runs as the menubar app only (no separate LaunchAgent). Restart via the Mac app or `scripts/restart-mac.sh`. **Start/stop the gateway via the app, not ad-hoc tmux sessions; kill temporary tunnels before handoff.**
- If shared guardrails are available locally, review them; otherwise follow this repo's guidance.
- SwiftUI state management (iOS/macOS): prefer the `Observation` framework (`@Observable`, `@Bindable`) over `ObservableObject`/`@StateObject`; don’t introduce new `ObservableObject` unless required for compatibility, and migrate existing usages when touching related code.
- Connection providers: when adding a new connection, update every UI surface and docs (macOS app, web UI, mobile if applicable, onboarding/overview docs) and add matching status + configuration forms so provider lists and settings stay in sync.
- Version locations: `package.json`, `apps/android/app/build.gradle.kts`, `apps/ios/**/Info.plist`, `apps/macos/**/Info.plist`, `docs/install/updating.md`, and Peekaboo Xcode plists. "Bump version everywhere" means all of these **except** `appcast.xml` (only touch appcast for macOS Sparkle releases).
- **Restart apps:** “restart iOS/Android apps” means rebuild (recompile/install) and relaunch, not just kill/launch.
- **Device checks:** before testing, verify connected real devices (iOS/Android) before reaching for simulators/emulators.
- iOS Team ID: `security find-identity -p codesigning -v` → Apple Development TEAMID.
- A2UI bundle hash (`src/canvas-host/a2ui/.bundle.hash`): auto-generated; regenerate via `pnpm canvas:a2ui:bundle` when needed, commit as a separate commit.
- CLI palette: use `src/terminal/palette.ts` (no hardcoded colors).

## Collaboration / Safety Notes

- When working on a GitHub Issue or PR, print the full URL at the end of the task.
- Respond with high-confidence answers only: verify in code; do not guess.
- Dependencies: never update Carbon; patched deps (`pnpm.patchedDependencies`) must use exact versions (no `^`/`~`); patching/overrides/vendoring requires explicit approval.
- **Multi-agent safety:**
  - Do **not** create/apply/drop `git stash` entries (including `--autostash`) unless explicitly requested.
  - Do **not** create/remove/modify `git worktree` checkouts or switch branches unless explicitly requested.
  - When the user says "push", you may `git pull --rebase` (never discard other agents' work). "commit" = scope to your changes only. "commit all" = everything in grouped chunks.
  - Prefer grouped `commit` / `pull --rebase` / `push` cycles instead of many tiny syncs.
  - Multiple agents are OK if each has its own session. When you see unrecognized files, focus on your changes and commit only those.
- Lint/format churn:
  - If staged+unstaged diffs are formatting-only, auto-resolve without asking.
  - If commit/push already requested, auto-stage and include formatting-only follow-ups in the same commit (or a tiny follow-up commit if needed), no extra confirmation.
  - Only ask when changes are semantic (logic/data/behavior).
- Focus reports on your edits; avoid guard-rail disclaimers unless truly blocked. When multiple agents touch the same file, continue if safe.
- Bug investigations: read source code of relevant npm dependencies and all related local code before concluding; aim for high-confidence root cause.
- Tool schema guardrails:
  - Avoid `Type.Union` in tool input schemas; no `anyOf`/`oneOf`/`allOf`. Use `stringEnum`/`optionalStringEnum` (Type.Unsafe enum) for string lists, and `Type.Optional(...)` instead of `... | null`. Keep top-level tool schema as `type: "object"` with `properties`.
  - Avoid raw `format` property names in tool schemas; some validators treat `format` as a reserved keyword and reject the schema.
- Never send streaming/partial replies to external messaging surfaces (WhatsApp, Telegram); only final replies should be delivered there. Streaming/tool events may still go to internal UIs/control channel.
- Release guardrails: do not change version numbers without explicit consent. For beta Git tags (`vYYYY.M.D-beta.N`), publish npm with a matching beta version suffix (`YYYY.M.D-beta.N`) rather than a plain version on `--tag beta`.
