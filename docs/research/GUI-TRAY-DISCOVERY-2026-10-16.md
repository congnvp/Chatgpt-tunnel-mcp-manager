# GUI/Tray Discovery — CTMM
Status: PROPOSED / NOT APPROVED / NOT TESTED. Event: CTMM-20261016-011. Updated: 2026-10-16 04:09:54 Asia/Bangkok.
Scope V1-R07/R08/R09; Q03/Q06/Q07/Q08; D-ARCH-02/05/06/07; P00/P01/P05/P06/P08.
No reuse of old GTK3 C PoC code. No product GUI, service or tunnel modifications.

## Source-backed comparison (2026-10-16)
| Toolkit | Rust integration and Windows | Ubuntu GNOME Wayland/tray | Key trade-offs |
|---|---|---|---|
| Slint (1.18 line) | Direct Rust; Windows tray | Built-in StatusNotifierItem / AppIndicator; clicked() callback, dependent on desktop shell | Strong candidate for one Rust GUI. Need manual GTK-free/Wayland/menu/accessibility/licensing tests; tray separate component globals. |
| Tauri 2 | Rust backend + HTML/CSS/JS frontend; Windows tray | Linux mouse click tray events and showMenuOnLeftClick unsupported; native menu works | Rich settings quickly, but WebKitGTK + webview deps, larger frontend attack surface; one-click custom panel nonportable. |
| GTK4 + gtk-rs | Rust GTK bindings; supported Win32 GTK backend | Strong Linux GTK, but separate SNI/Ayatana/ksni tray adapter | Additional Windows packaging and tray complexity; isolated GTK3 C demo is NOT production library. |
| Qt6 + CXX-Qt | Rust/C++ bridge; Qt Windows tray | QSystemTrayIcon uses SNI; GNOME activation limited | Mature framework and Qt bindings; C++ build/deployment, licensing and integration complexity. |
| egui/iced + tray-icon/ksni | Rust renderer; Windows tray-icon | Separate StatusNotifierItem integration and event loop | Possible compact Rust option; extra wiring, DPI, accessibility and distro matrix. |

Verified docs: 
- Slint 1.17 tray release https://slint.dev/blog/slint-1.17-released ; 1.18 release 2026-09-16 https://slint.dev/blog/slint-1.18-released ; tray API https://docs.slint.dev/latest/docs/slint/reference/window/systemtrayicon/ ; event-thread marshal https://docs.rs/slint/latest/slint/fn.invoke_from_event_loop.html
- Slint license (GPLv3, desktop royalty-free with attribution, commercial) https://github.com/slint-ui/slint/blob/master/LICENSE.md
- Tauri Linux tray restrictions https://tauri.app/learn/system-tray/ ; https://docs.rs/tauri/latest/tauri/tray/struct.TrayIcon.html ; WebKit GTK prerequisites https://v2.tauri.app/start/prerequisites/
- GTK4 Rust https://gtk-rs.org/gtk4-rs/stable/latest/docs/gtk4/index.html ; Windows backend https://docs.gtk.org/gtk4/windows.html
- Qt tray https://doc.qt.io/qt-6.12/qsystemtrayicon.html ; CXX-Qt https://kdab.github.io/cxx-qt/book/
- Rust tray icon KSNI alternative https://github.com/tauri-apps/tray-icon ; GNOME AppIndicator extension https://extensions.gnome.org/extension/615/Appindicator-support/ ; protocol https://specifications.freedesktop.org/status-notifier-item/latest-single/

## Recommended UX policy (not accepted)
One-click tray activation opens/re-focuses dashboard ONLY when OS/extension delivers it. Native tray context menu ALWAYS has 'Open Dashboard', 'Copy Sanitized Diagnostics', 'Quit GUI'. Separate desktop launcher works even without tray extension. Wayland toplevel window position is compositor-controlled: no promise to anchor floating panel to tray pixel. Closing dashboard hides GUI; quitting GUI terminates GUI client alone; neither stops/restarts service, tunnel, proxy or MCP server.

Settings information architecture: Overview (service/tunnel/proxy/MCP statuses); MCP Servers (read-only list/detail until mutation gate); Tunnel and Proxy (validated proposed settings); Diagnostics (redacted errors/test provenance); Preferences (GUI autostart/tray policy). No plaintext credentials in UI model. Buttons that alter state require server-side auth/approval/audit/rollback after D-ARCH-06.

## Proposed process and state interface
```
[systemd / Windows Service] ctmm-service (Rust, headless)
            |  local authenticated IPC (ADR OPEN)
            +--> ctmm-desktop process (interactive user session; tray/settings)
            +--> ctmm-cli (optional)
            +--> management MCP (independent remote auth)
```
Suggested typed immutable Snapshot: schema_version, boot_id, revision, observed_at, service_state, tunnel_state, proxy_state, servers[], last_error_code, last_success_at, test_evidence_id. Never serialize secrets, raw auth headers, environment or unredacted logs. GUI reads authoritative GetSnapshot after connection; subscribes to revisioned changes, gap/restart -> resync. Disconnect/expired TTL -> UNKNOWN/STALE (never READY from cache). Polling-only alternative: bounded every 2–5s while visible and slower hidden; subscription with periodic resync preferred once IPC specified. Marshaling to GUI thread required; no renderer thread network calls.

IPC option assessment: Linux Unix Domain Socket with filesystem owner/peer credentials; Windows Named Pipe with service/user SID ACL. Final mechanism, permission scopes, impersonation and schema version MUST be determined by D-ARCH-02/06. GUI is neither service parent nor authority.

## Acceptance and negative test plan (ALL NOT RUN)
| ID | Stimulus | Required evidence / outcome |
|---|---|---|
| GUI-01 | boot test service with DISPLAY absent, no graphical login | fake service reaches READY, local health successful, no GUI/library dependency |
| GUI-02 | close/hide GUI | service PID, state and capabilities persist; no lifecycle command |
| GUI-03 | force-crash GUI | only GUI dies; service remains READY; relaunch resyncs from service |
| GUI-04 | IPC disconnected, denied, timeout | STALE/UNKNOWN, bounded retry; no false READY, no privileged bypass |
| GUI-05 | out-of-order, duplicated, revision-gap snapshot events | stale ignored; gap triggers GetSnapshot; boot_id rollover resets revision domain |
| GUI-06 | GNOME Wayland AppIndicator extension enabled/disabled | ON: icon/menu/click manually checked; OFF: dashboard launcher still works |
| GUI-07 | Wayland multi-monitor/scaling, compositor restart | focusable visible dashboard (exact position not required), service continues |
| GUI-08 | Ubuntu X11, KDE Wayland, Windows 11 Explorer restart | explicit tray availability, recovery/fallback per OS |
| GUI-09 | unauthorized settings mutation / secret-like fixture | denied at server; redaction verified, no drift |
| GUI-10 | mock READY/DEGRADED/OFFLINE/FAILED/STALE | screenshots+assertions on text/icon/tooltips, linked evidence IDs |
| GUI-11 | Windows service with no user logged in | service independently available, tray appears only after login |
| GUI-12 | legacy production before/after probe | running tunnel/proxy/service processes and config unchanged |
All tests need timestamp, platform/session, exact version/commit, command output and exit code, screenshot if UI, sanitized trace. Existing isolated C demo smoke tests are NOT evidence for these tests.

## Gates and rollback
P00: user accepts GUI ADR, license and one-click best-effort semantics (CURRENTLY OPEN). P01: service state contract validated before GUI. Then an isolated mock-snapshot GUI slice may be built with no connection to production tunnel. P06: binding to authenticated local IPC and visual acceptance tests. P05: only permission-gated settings edits. P08: Windows repeats negative/headless tests. Rollback only closes/uninstalls NEW GUI and reverts its isolated test settings; NEVER touches service or existing running tunnel/proxy.

Decision options: A Slint native Rust (recommended for validation), B Tauri menu-first webview, C Qt6/CXX-Qt, D GTK4 Ubuntu-first. No selection without user ADR approval.
