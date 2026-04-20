# nativephp-mobile skill

An [Agent Skill](https://skills.sh) that teaches Claude Code, Cursor, Codex and other AI coding assistants how to build mobile apps with [NativePHP Mobile](https://nativephp.com) v3.0–3.1 — full Laravel apps running natively on iOS and Android.

## What it covers

- **Getting started:** environment setup (iOS/Android), installation, the `nativephp/` directory, v3.1 **Persistent Runtime** and **Android API 26+** support
- **Configuration:** `config/nativephp.php`, Android SDK keys (`compile_sdk`/`min_sdk`/`target_sdk`), Android build flags, App Store Connect, Start URL, permissions
- **The basics:** WebView + safe areas, native functions, 3 event-listener patterns (`#[OnNative]`, `Native.on()`, `#nativephp` import), Tailwind v3/v4 WebView compatibility
- **EDGE components:** Bottom Nav, Top Bar, Side Nav, 4-tier icon resolution (SF Symbols ↔ Material Icons)
- **Concepts:** authentication (Sanctum, OAuth, `Browser::auth()`), SQLite databases, deep links (custom scheme + Universal Links), push notifications (Firebase/FCM), security (SecureStorage, device-unique APP_KEY)
- **Queues (v3.1):** background ZTS worker with `QUEUE_CONNECTION=database`
- **Plugins:** scaffolding, bridge functions (PHP ↔ Swift/Kotlin), events, lifecycle hooks, manifest (`nativephp.json`), permissions/dependencies, marketplace best practices
- **Commands:** full `native:*` Artisan command reference with v3.1 flags (`native:jump` shorthands, multi-plugin registration)

All source URLs are pinned to NativePHP docs SHA [`393dffc`](https://github.com/NativePHP/nativephp.com/tree/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3) so links don't drift.

## Install

### Via [skills.sh](https://skills.sh) (recommended)

Install globally (available in all projects):

```bash
npx skills add kuzmany/nativephp-mobile-skill -a claude-code -g
```

Install to a specific project only:

```bash
cd your-nativephp-project
npx skills add kuzmany/nativephp-mobile-skill -a claude-code
```

Supported agents (targeted via `-a <agent>`): `claude-code`, `cursor`, `codex`, `opencode`, and [~40 more](https://www.npmjs.com/package/skills).

List the skill before installing to see what you're getting:

```bash
npx skills add kuzmany/nativephp-mobile-skill --list
```

### Manual install (Claude Code)

```bash
git clone https://github.com/kuzmany/nativephp-mobile-skill.git
cp -r nativephp-mobile-skill/skills/nativephp-mobile ~/.claude/skills/
```

Then restart Claude Code (or reload the skills list). The skill triggers on keywords like `nativephp`, `mobile app with laravel`, `native:run`, `build APK`, `EDGE components`, `persistent runtime`, `background jobs`, and many more.

## How it's structured

```
skills/nativephp-mobile/
├── SKILL.md                        # Entry point — triggers + quick reference
└── references/
    ├── getting-started.md          # Environment, install, config, deployment, CLI
    ├── basics-and-events.md        # WebView, events, assets, icons, splash
    ├── concepts.md                 # Auth, DB, deep links, push, security
    ├── edge-components.md          # Bottom Nav, Top Bar, Side Nav, icons
    ├── queues.md                   # v3.1 background worker
    └── plugins/
        ├── creating.md             # Scaffolding, manifest
        ├── bridge-functions.md     # PHP ↔ Swift/Kotlin
        ├── events.md               # Dispatch events from native
        ├── lifecycle-hooks.md      # pre/post_compile, copy_assets, post_build
        ├── advanced.md             # Permissions, deps, secrets, entitlements
        └── best-practices.md       # Validation, testing, marketplace
```

Total: 12 files, ~1,850 lines. SKILL.md is kept small (~170 lines) for fast context loading; deeper knowledge lives under `references/` and is loaded on demand.

## When to use this skill

Ask your AI assistant anything like:

- *"Build a NativePHP mobile app that shows a bottom nav and calls the camera"*
- *"How do I deploy this app to Play Store with AAB + signing?"*
- *"Set up push notifications via Firebase in NativePHP"*
- *"Explain the difference between persistent and classic runtime"*
- *"I need to register a NativePHP plugin that calls a Swift SDK"*
- *"How do I handle deep links (Universal Links) in NativePHP?"*

## Keeping in sync with upstream docs

The skill pins to NativePHP docs SHA [`393dffc`](https://github.com/NativePHP/nativephp.com/tree/393dffc5e070675654b64275d3b44d4923a6635f). To refresh against a newer SHA:

1. Find the latest commit under [`resources/views/docs/mobile/3`](https://github.com/NativePHP/nativephp.com/tree/main/resources/views/docs/mobile/3)
2. Diff the changelog — new v3.x features that need skill updates
3. Bump the SHA in every `**Source (pinned):**` line across `SKILL.md` and `references/**/*.md`
4. Bump `version:` in `SKILL.md` frontmatter

## Credits

- [NativePHP Mobile](https://nativephp.com/) — the framework this skill documents
- [skills.sh](https://skills.sh) / [vercel-labs/skills](https://github.com/vercel-labs/skills) — the open agent skills ecosystem

## License

MIT — see [`LICENSE`](./LICENSE).
