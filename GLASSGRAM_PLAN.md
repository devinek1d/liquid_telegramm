# Glassgram: Product & Implementation Plan

**Status:** product requirements approved; implementation not yet started.  
**Target:** Windows 11 x64, initial personal use, possible public GPLv3 release.  
**Origin:** fork of Telegram Desktop (`https://github.com/telegramdesktop/tdesktop`).  
**Date:** 2026-11-08.

## 1. Product promise

A standalone third-party Telegram client that preserves Telegram Desktop's familiar information architecture and behavior, but adds a dark, restrained Apple Liquid Glass-inspired appearance and smooth, expressive interactions. It must not make text harder to read, impair fundamental messaging, or feel slower than the original client.

**Working product name:** Glassgram (not an official Telegram product). Design a distinct logo/brand before any public release.

## 2. Decisions fixed by the product owner

| Area | Decision |
|---|---|
| Visual direction | Apple Liquid Glass-inspired |
| Main theme | Dark, black/charcoal accents |
| Translucency | Moderate, frosted surfaces inside the app |
| Desktop wallpaper | Must **not** show through the application window |
| Corner shapes | Slightly rounded, not pill-shaped everywhere |
| Information architecture | Preserve Telegram Desktop's navigation/layout |
| Chat sidebar | Can collapse to a narrow avatar-only rail |
| Message bubbles | Frosted/semi-transparent |
| Chat header | Frosted glass |
| Profile/settings/search/popovers | Same functions/layout, restyled consistently |
| Icons | Custom coherent icon set permitted |
| Motion | Smooth and expressive, typically 250–350 ms |
| Chat switches | Animated |
| Sending messages | Animated |
| Hover | Scale/highlight/subtle glass sheen |
| Spring/bounce physics | Avoid |
| Ambient decoration | Not constantly moving; disable any optional ambient effects |
| User customization | Blur, opacity, motion speed, optional effects |
| Themes/presets | One Glass look initially, plus Classic switch |
| User-supplied background | Not included initially |
| Accent from Windows wallpaper | Not included |
| Performance tradeoff | Prioritize perceived smoothness, but zero noticeable jank |
| Platform | Windows 11 Pro 25H2, x64; no macOS/Linux initial scope |
| Installation | Separate `.exe` client, independent from official Telegram |
| Updates | Eventually updates from GitHub Releases, with integrity verification |
| Localization | Russian and English |
| Collaboration | No incoming community PR workflow planned initially |

## 3. MVP priorities (approved order)

1. Consistent frosted-glass appearance of main screens: chat list, chat view, header, input, bubbles, menus and key settings surfaces.
2. Animated chat transitions. Navigation stays responsive, including rapid repeated switches.
3. Message-send animation. It is *purely visual*, must not control networking, message delivery, ordering or read state.
4. Glass Appearance settings (opacity, blur, animation speed, extra effects), persisted locally.
5. Instant Classic/Glass toggle with clean restoration of the stock look.

**Next-after-MVP:** avatar-only collapsing sidebar, custom icon pass, distribution polish. It may be brought into MVP only if the five priorities are already reliable.

## 4. Non-negotiable quality requirements

- No visible animation stutter on the target Windows 11 test computer. Target a smooth 60 fps at 60 Hz as an initial benchmark; profile on the actual machine, especially during large chats and media-heavy views.
- No readable-text contrast regression. Test text, selected messages, timestamps, unread badges, search results, media captions and disabled controls in both Glass and Classic modes; use WCAG-style contrast checks as a design aid, not a substitute for visual testing.
- No autonomous, continuously moving UI. Any optional decoration is off by default or expressly toggleable; motion stops when idle/backgrounded.
- Reduced-motion behavior: disable nonessential motion or use near-instant transitions; respect system preference when technically possible.
- Basic Telegram behaviors work unchanged, including login, message send/receive, notifications, dialogs, files, images, calls, search, message history and keyboard shortcuts.
- Do not alter protocol, security boundaries, encryption behavior, retention/deletion rules, read receipts, typing status or sponsored-message handling.
- Original installed Telegram stays intact; Glassgram uses its own executable name, application identity, shortcuts, update channel and profile/session storage. Verify each separation point before use with a personal account.

## 5. Initial design tokens (proposals, to confirm in a visual prototype)

These are design starting values, not implementation claims.

| Token | Proposed value |
|---|---|
| App canvas | `#0B0C0F` |
| Deep panel | `#111318` |
| Frosted surface fill | `rgba(33,36,44,0.76)` |
| Bubble fill | `rgba(48,52,62,0.72)` |
| Border/highlight | `rgba(255,255,255,0.09)` |
| Primary text | `#F3F4F6` |
| Secondary text | `#B5B8C1` |
| Accent | Neutral white/gray, no neon |
| Panel radius | 10px |
| Bubble radius | 11px |
| Interactive button radius | 8px |
| Target blur radius | 12–20px in *application content*; adjust after profiling |
| Chat transition | 300ms, easing `out-cubic` or equivalent |
| Send animation | 280ms, easing `out-cubic` or equivalent |
| Hover response | 160–180ms, small scale 1.00→1.02 maximum |

**Important rendering distinction:** achieve the frosted effect by sampling or composing *in-app background layers* as available in the existing rendering architecture. No Windows desktop transparency is requested. Windows Mica/Acrylic should not be assumed to solve this design requirement. Graceful fallback to opaque surfaces is mandatory.

## 6. Interaction specifications

### 6.1 Chat switching

- Keep existing sidebar, title, composer and chat content semantics; animate only visual transition.
- Suggested behavior: new chat content fades and subtly translates into place; old content exits without causing a layout jump.
- Duration 300ms, no bouncy spring.
- Rapid click or keyboard navigation cancels/replaces the pending transition; never queues many transitions.
- No stale messages of another chat should remain after completion; notifications and accessibility semantics follow the active chat immediately.
- Inactive/minimized window or reduced motion: skip/shorten effect.

### 6.2 Sending a text message

- Start from the existing composer UI; animate a lightweight visual representation toward the newly inserted outgoing message bubble.
- Let the existing Telegram message-send path execute independently, immediately.
- Preserve correct success/failure state, ordering, scroll position, draft clearing and delivery indicators.
- For rapid successive sends, do not block input. Respect reduced motion and minimize/restore.
- Initially implement for **plain text only**; media, stickers, voice messages and scheduled messages keep stock behavior until explicitly designed and tested.

### 6.3 Glass surfaces

- Reuse existing component positions and keyboard/mouse actions.
- Keep text legible over backgrounds with strong variation; if needed add an adaptive dark scrim behind text.
- Blur/fill is an optimization-sensitive effect: cache when practical, invalidate only when input changes, profile GPU/CPU costs.
- Provide opaque fallback on unsupported hardware or a performance issue.

### 6.4 Appearance settings

- Glass enabled / Classic mode toggle.
- Glass opacity, blur strength, motion speed (off, fast, balanced), optional background effects (off by default).
- Changes preview instantly, persist across restart, and do not modify the official Telegram settings.
- Classic restores the original design, not a partly-glass hybrid.

### 6.5 Sidebar collapse (post-MVP)

- Collapses to a narrow, visible vertical avatar rail; expanded layout remains the familiar Telegram list.
- Preserve unread indicators, chat selection, tooltips, keyboard navigation and context menus.
- Adjustable rail width only if necessary after initial usability testing.

## 7. Delivery stages and exit gates

### Stage 0: Repo, legality and baseline build

1. Inspect upstream `README.md`, `AGENTS.md`, `docs/building-win.md`, license and the checked-out branch/commit. The real files take precedence over remembered documentation.
2. Fork upstream, record a pinned baseline commit and branching strategy (`main`, small `feature/*`).
3. Inspect Windows tools and disk space. Current upstream `dev` Windows instructions mention VS 2026, Windows SDK 10.0.26100.0, Python 3.10, an x64 developer environment and proprietary Telegram API credentials supplied by the developer. Do not assume the local toolchain is already correct.
4. Build original Telegram Desktop **without UI changes** using exact checked-out instructions. Save full logs.
5. Plan standalone identity/session separation; no testing with the user's real production account before this is implemented and checked.

**Exit:** clean baseline Debug and Release binaries launch, a documented reproducible build log exists. If this fails, stop and report rather than coding UI changes.

### Stage 1: Design proof, not a full reskin

1. Produce accurate mockups for 1440×900 and 1920×1080, dark desktop, using actual Telegram Desktop layout proportions.
2. Mockup normal and hovered list item, selected chat, message bubble, header, composer, context menu, settings, Classic mode and collapsed avatar rail.
3. Define finalized spacing, typography, color, opacity, blur and motion tokens; attach a visual comparison checklist.
4. Validate text readability before implementing.

**Exit:** approved static screen states + a motion storyboard / short proof of interaction. No source-code-wide redesign yet.

### Stage 2: Theme and frosted surfaces

1. Identify actual upstream style, widget and renderer ownership; never guess C++ paths.
2. Add a separately named appearance layer / token bundle for Glass mode.
3. Implement a single frosted surface and profile it; then extend to header, sidebar, bubbles, composer, menus, settings.
4. Support partial redraws and opaque fallback.
5. Keep Classic mode working throughout.

**Exit:** visually consistent Glass design in primary screens, no functional regression, screenshot comparisons, no substantial frame-time regression.

### Stage 3: Chat transitions

1. Implement animation with cancel/restart semantics and offscreen lifecycle checks.
2. Test fast switching, keyboard navigation, huge history, search/back navigation, minimized app and reduced motion.

**Exit:** working transitions with no stale content or stuck interaction.

### Stage 4: Message-send animation

1. Implement only plain text first, without altering the send pipeline.
2. Test success, failure, delayed network, long and multiline messages, fast repeated sends, and scroll near/beyond bottom.
3. Ensure no extra duplicate bubble or misleading delivery state.

**Exit:** visually smooth, correct state and no added input latency.

### Stage 5: Appearance settings + Classic

1. Add settings controls, persistence and live preview.
2. Implement full Classic restoration, animation disable and reduced-motion behavior.
3. Test reopening, multiple windows/accounts, theme changes and high DPI.

**Exit:** toggle is reliable; settings survive restart; Classic is visually and functionally faithful.

### Stage 6: Sidebar, icons and polish (after MVP gate)

1. Implement avatar rail collapse, unread dots and tooltips.
2. Introduce and audit custom icons; check 100%, 125%, 150% and 200% scaling.
3. Verify RU/EN truncation and RTL / emoji / Unicode layouts as appropriate to upstream support.

**Exit:** functionality at least as accessible as expanded sidebar.

### Stage 7: Release infrastructure

1. Choose a *third-party* product identity: custom icon and Glassgram name, explicit Telegram API attribution, GPLv3-compatible source publication and notices.
2. Ensure own `api_id`, upstream license preservation, no personal keys in repo, CI logs or source maps. Note: client-distributed credentials cannot be made intrinsically secret by compilation; threat-model accordingly.
3. Build an `.exe` installer only after app identity and independent data dirs are proven. A bare executable is not automatically an installer.
4. Test clean Windows 11 installation/uninstallation and simultaneous running with official Telegram.
5. Automate release via GitHub Releases, publish checksums, signatures if available and reproducible source revision; updater must cryptographically verify updates and not reuse upstream Telegram's channel.
6. Write RU/EN README, screenshots, known limitations, security notice, license and changelog.

**Exit:** a third party can install Glassgram without overwriting Telegram, log in, use core messaging and verify the release source/artifact identity.

## 8. Testing matrix

- Visual: window sizes 1280×720, 1440×900, 1920×1080; DPI 100%, 125%, 150%, 200%; dark backgrounds; long names; long/multiline/RTL/emoji messages; timestamps/unread bubbles.
- Performance: idle CPU/GPU; switching 100 chats; rapid send; large histories; videos/GIFs/stickers; minimized/restore; low-end fallback if available. Compare against the clean baseline on the same machine.
- Functionality: login/log out, different accounts, chat categories, groups/channels, pin/mute, notifications, file transfer, calls, message edits/deletes, self-destruct/retention semantics, accessibility and keyboard navigation.
- Reliability: network outage, app suspend/resume, window resize during animation, rapid theme toggle, high DPI changes, restart, upgrade/downgrade only with backups.
- Packaging: distinct application identity, shortcuts, icons, notification integration, data directories, installer/uninstaller, update rollback strategy.

## 9. Tasks to avoid for now

- New network protocol or new messaging engine.
- Breaking changes to Telegram Desktop's information architecture.
- User-provided wallpapers or automatic accent extraction.
- Spring physics or perpetual ambient animation.
- Custom media send animations, custom stickers editor, or chat-management features unrelated to the design goal.
- Real transparent windows showing the Windows desktop.
- Public auto-update before a trustworthy packaging/signature/update-verification design exists.

## 10. Repository docs and working rules

Create `docs/GLASSGRAM_DESIGN.md`, `docs/GLASSGRAM_TEST_MATRIX.md`, and `docs/GLASSGRAM_CHANGELOG.md`. Do not replace upstream `AGENTS.md` without first reading its instructions and preserving upstream-required conventions. Separate Glassgram-specific Codex rules can live under `docs/GLASSGRAM_CODEX_RULES.md` and be referenced explicitly in task prompts.

Every Codex task must provide: goal; precise scope; current upstream source paths verified by inspection; expected screenshots/video or log; build/test commands actually executed; rollback path; and remaining risks. One focused feature per commit/PR. Do not claim success without a verified build or test.

## 11. External references verified 2026-11-08

- [Official Telegram Desktop source](https://github.com/telegramdesktop/tdesktop)
- [Official Windows build guide](https://github.com/telegramdesktop/tdesktop/blob/dev/docs/building-win.md)
- [Telegram API Terms](https://core.telegram.org/api/terms)
- [Telegram Desktop LICENSE](https://github.com/telegramdesktop/tdesktop/blob/dev/LICENSE)

**Important:** requirements and exact build commands may change upstream. Codex must inspect the checked-out files before running setup or modifying code.
