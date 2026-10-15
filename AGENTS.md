# Tampermonkey-Scripts Runbook

## Repo Scope

- Owner/escalation: Longbiao for managed userscript behavior and release/version proof.
- This repo owns source userscripts under `src/scripts/`, generated `dist/*.user.js` artifacts, metadata in `tampermonkey.config.mjs`, and release/update helpers.
- Built `dist/*.user.js` files are committed release artifacts and must stay in sync with source and metadata.

## Canonical Commands

- Install deps: `npm install`
- Build userscripts: `npm run build`
- Validate build/tests: `npm run validate`
- Patch release prep: `npm run release:patch`
- Publish/update `x-tweaks`: `npm run release:x-tweaks`
- Check installed version: `npm run chrome:check-installed`
- Trigger installed update: `npm run chrome:trigger-update`

## Routine Operations

| Trigger | Command | Expected Result | Failure Recovery |
| --- | --- | --- | --- |
| Change a userscript | `npm run validate` | Source tests pass and `dist/*.user.js` regenerates | Fix source/metadata/test together; do not edit `dist` alone |
| Add a script | Add source, metadata, tests, then `npm run validate` | New `dist/<script-id>.user.js` is generated with correct update URLs | Keep script id, test file, and metadata in sync before retrying |
| Publish `x-tweaks` update | `npm run release:x-tweaks` | Version bumps, validates, pushes, raw URL exposes new version, installed script proof succeeds | If browser/profile proof fails, use explicit `--profile` and `--chrome-root` flags; do not change source to fix profile state |

## Troubleshooting

| Trigger | Command | Expected Result | Failure Recovery |
| --- | --- | --- | --- |
| Tampermonkey does not update | `node scripts/check-installed-version.mjs --profile Default` | Installed version is visible in extension storage | Ensure `@version` increased and Chrome MV3 `Allow User Scripts` is enabled |
| Verify installed userscript state | `node scripts/check-installed-version.mjs --profile Default` | Helper checks installed script metadata in the target browser profile | Use the official `Chrome` plugin workflow to open the target profile when user-logged state is required |

## Verification

- `npm run validate` is the minimum source/build gate.
- For release/update tasks, prove both the raw GitHub userscript version and installed Tampermonkey version.
- Browser verification should use the official in-app browser for unauthenticated debug harness pages; use the official `Chrome` plugin only when verifying real installed userscript state in the default Chrome profile.
- When a userscript change is ready but Chrome/Tampermonkey auto-install is brittle or not requested, provide the GitHub raw userscript URL as a clickable Markdown link so the operator can open it in Chrome and install manually.

## Release/Deploy

- Install URLs point at raw GitHub files in `dist/`.
- Tampermonkey updates only when `@version` increases.
- `npm run release:x-tweaks` is the full publish/update/proof path for `x-tweaks`.
- Manual install handoff for `x-tweaks`: [`https://raw.githubusercontent.com/felixlark/tampermonkey-scripts/main/dist/x-tweaks.user.js`](https://raw.githubusercontent.com/felixlark/tampermonkey-scripts/main/dist/x-tweaks.user.js).

## Guardrails

- Do not use the user's real browser profile unless the task explicitly targets the installed userscript state.
- Keep generated metadata URLs, package version, and committed `dist` artifacts aligned.

## Known State

- Managed scripts include `x-tweaks` and `gitlab-tweaks`.
- The shared version source is `package.json`.

## Browser Automation Constraint
- Follow the global `~/.codex/AGENTS.md` official browser/GUI automation policy: Chrome plugin for signed-in browser state, Browser plugin for unauthenticated rendering, and Computer Use for native desktop boundaries. Do not bypass it with AppleScript or `osascript` unless the global exception rules are met.
- Keep only repo-specific verification surfaces here; do not copy the full global policy block into this runbook.

## Worktree Policy

- Follow the global `~/.codex/AGENTS.md` main-first development rule: work in the current Local checkout by default and use a worktree only when the global exception list applies.
- Branch names should use `codex/<repo>-<short-task>`; manual long-lived worktree directories should use `~/Projects/<repo>-<short-task>`.
- Initialize dependencies inside each worktree and keep ports, databases, device/simulator state, build outputs, and ignored local config isolated per checkout.
- Preserve existing dirty checkouts. Inspect `git status --short` before editing, and do not stash, commit, remove, or migrate user changes unless explicitly asked.
- After merge or abandonment, clean up with `git worktree remove <path>` and use `git worktree prune` only for stale metadata.

## 合并前评审

合并前须等待 Codex 自动评审完成，且评审意见已全部处理清零（无未解决的阻塞性评论）之后，方可合并。不得在评审结果出来之前合并。

### Pre-merge review (English translation)

Before merging, wait for Codex automatic review to finish and address all review feedback, leaving no unresolved blocking comments. Do not merge before the review results are available.

## 限制类改动规则

1. 新增或调小任何上限、槽位、名额、闸门、门槛、退避（包括并发数、负载准入、超时退避等）时，PR 正文须单列「新增限制」一节，写明位置、默认值和依据，经仓库负责人定稿后才可合并。此类改动不得绕过 PR 直接进入 main。
2. 资源类参数默认不限、只观测；确需限制时须通过显式配置开启，不得在代码中写死最大值。外部硬限制（如外部接口限流、部署锁）须在注释中写明来源。
