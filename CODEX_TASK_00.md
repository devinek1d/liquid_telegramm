# Codex Task 00 — Audit and build Glassgram baseline

You are a senior C++ / Qt / Windows developer working on **Glassgram**, a future open-source Windows 11 third-party client forked from Telegram Desktop.

Read `GLASSGRAM_PLAN.md` first; it is the product specification. The immediate purpose is to **verify feasibility and obtain a clean baseline build**, not to restyle the application.

Upstream source: https://github.com/telegramdesktop/tdesktop

## Scope

1. Inspect the current working directory, Git status, active branch, remote, pinned upstream commit, submodules and available disk space. Do not overwrite any user changes.
2. Read the actual repository `README.md`, `AGENTS.md`, `docs/building-win.md`, `LICENSE` and relevant project guidance **before** executing build steps. If the documents disagree, explain the conflict and choose what the checked-out source and build scripts support.
3. Inspect installed Windows 11 toolchain: Visual Studio/C++ build tools, Windows SDK, Python, Git, CMake, Qt-related dependencies; determine if the checkout is buildable on this machine. Do not install software silently or perform system-wide changes without approval.
4. Map, with verified current file paths, the following: main window, chat list, message view, message composer, theme/styles, popup menus, animations, settings, Windows application identity, local session storage and updater.
5. If prerequisites and credentials are present, build the **unaltered official baseline** in Debug and/or Release following the checked-out official instructions. Preserve complete output logs, exit codes, produced binary paths and artifacts.
6. If Telegram API credentials are needed but not provided, stop at that step and give precise instructions to obtain and pass them safely. Do not fabricate credentials, commit them or print them in logs.
7. Recommend an isolated Glassgram profile/identity strategy, including separate executable name, application identifier, update channel, shortcuts, notifications and data directory. **Do not log into or migrate a real Telegram account yet.**
8. Design a narrow technical proof-of-concept for a frosted *in-app* surface (not desktop transparency). Compare feasible Qt rendering methods, fallbacks and profiling plan. Do not implement the complete redesign in this task.
9. Create/update `docs/GLASSGRAM_BUILD_REPORT.md` with environment, commands tested, results, blockers, verified source paths, risks and recommended next task. Keep non-sensitive paths and tool versions; redact all credentials.

## Acceptance gate

- Baseline source compiles and launches on this machine **or** you produce an exact evidence-based blocker report.
- No UI code changes, no commit to upstream, no public push, no modification of the officially installed Telegram or its sessions.
- The report states clearly what was *actually* tested vs merely proposed.

## Deliver final response in Russian

- Git repo/branch/commit inspected
- Installed and missing dependencies
- Commands executed and their outcomes
- Build status and binary location if successful
- Verified paths of UI systems
- Identified risks
- Recommended smallest next Codex task

Don't start Step 1 of the visual redesign until the baseline build gate is satisfied.
