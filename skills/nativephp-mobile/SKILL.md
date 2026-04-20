---
name: nativephp-mobile
description: This skill should be used when the user asks to "create mobile app", "nativephp", "native php", "mobile app with laravel", "ios app", "android app", "build APK", "build IPA", "sign app", "native:run", "native:install", "native:jump", "native:package", "deploy to app store", "deploy to play store", "EDGE components", "bottom nav", "top bar", "side nav", "push notifications", "firebase push", "nativephp plugin", "bridge function", "mobile deep links", "camera", "biometrics", "securestorage", "deep linking", "OnNative", "nativephp_call", "persistent runtime", "background jobs", "mobile queue", "queue worker", "laravel boost mobile", "mobile-starter", or mentions NativePHP Mobile development, Laravel mobile apps, or building iOS/Android apps with PHP. Documents NativePHP Mobile v3.0–3.1.
version: 0.2.0
---

# NativePHP Mobile v3.0–3.1

NativePHP runs full Laravel/PHP apps natively on iOS and Android — no web server, offline-first, with direct native API access via a custom PHP extension compiled into Swift (iOS) and Kotlin (Android) shells.

**Docs source (pinned):** `https://github.com/NativePHP/nativephp.com/tree/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3`

**Upstream docs home:** `https://nativephp.com/docs/mobile/3/getting-started/introduction` — every page has a "Copy as Markdown" button.

**Main code repo:** `https://github.com/nativephp/mobile-air` (renamed from `mobile`; Composer package is still `nativephp/mobile`).

## Architecture

1. Pre-compiled PHP (8.3–8.5, auto-matched from `composer.json`) bundled with Laravel code into a native shell app
2. Custom PHP extension exposes native APIs via `nativephp_call()`
3. UI rendered in a WebView — use Blade, Livewire, Vue, React, Svelte, Tailwind
4. EDGE components render truly native UI outside the WebView
5. All core APIs (Camera, Biometrics, etc.) delivered as internal plugins
6. **v3.1:** Persistent PHP runtime — Laravel kernel boots once and is reused (~5-30ms requests vs ~200-300ms in classic mode)
7. **v3.1:** ZTS (Thread-Safe) PHP enables a dedicated background queue worker

**Boot sequence (every app start):** Check bundle version → extract if newer → run migrations → clear caches → create storage symlinks.

## Quick Start

```bash
# New app with starter kit
laravel new my-app --using=nativephp/mobile-starter
cd my-app
php artisan native:jump   # No Xcode/Android Studio needed

# Existing app
composer require nativephp/mobile
php artisan native:install
php artisan native:run
```

## Key Patterns Unique to NativePHP

| Pattern | Details |
|---------|---------|
| `NATIVEPHP_APP_VERSION=DEBUG` | Forces PHP re-extraction every boot (dev only) |
| Persistent runtime (v3.1) | `NATIVEPHP_RUNTIME_MODE=persistent` (default). Falls back to `classic` if persistent boot fails |
| Background queue (v3.1) | Set `QUEUE_CONNECTION=database` — ZTS worker starts automatically (see `references/queues.md`) |
| SQLite only | No MySQL/Postgres — security by design |
| Device-unique `APP_KEY` | Generated per install, stored in native Keychain/Keystore |
| `mobile_public` disk | User files that survive app updates |
| Dual-dispatched events | Events fire to both JS (WebView) and PHP (Laravel) simultaneously |
| `#[OnNative()]` attribute | Livewire event listener for native events |
| `#nativephp` JS import | `import { On, Camera, Events } from '#nativephp'` |
| Auto migrations | Run on every boot — no manual `migrate` |
| `storage_path()` caveat | Points outside app root in mobile — use Laravel path helpers carefully |
| `nativephp/` dir is ephemeral | Add to `.gitignore`, rebuild with `native:install --force` |
| No CSRF | Mobile can't use browser CSRF — use rate limiting + auth keys |

## Platform Support (v3.1 minimums)

| Platform | Min | Default target |
|----------|-----|----------------|
| iOS | 18.0 | 18.0 |
| Android | API 26 (Android 8) | API 36 (configurable via `min_sdk`/`target_sdk`/`compile_sdk`) |
| PHP | 8.3 | matches your `composer.json` (8.3–8.5) |

## Platform Detection

```php
use Native\Mobile\Facades\System;
System::isIos();
System::isAndroid();
```

## Event Handling (3 methods)

```php
// 1. Livewire attribute
#[OnNative(PhotoTaken::class)]
public function handlePhoto(string $path) { ... }

// 2. Vanilla JS (Blade)
Native.on(@js(PhotoTaken::class), (path) => { ... })

// 3. SPA frameworks (Vue/React)
import { On, Off, Events } from '#nativephp';
On(Events.Camera.PhotoTaken, handler);
```

## Core Facades

`Biometrics`, `Browser`, `Camera`, `Device`, `Dialog`, `File`, `Geolocation`, `Haptics`, `Microphone`, `Network`, `PushNotifications`, `Scanner`, `SecureStorage`, `Share`, `System`

> **Note:** Core plugin API pages at `/docs/mobile/3/plugins/core/*` are title-only stubs upstream — no per-method reference exists yet. Use `basics-and-events.md` and `concepts.md` for usage patterns learned from the narrative docs.

## Essential Commands

| Command | Purpose |
|---------|---------|
| `native:install` | Set up native project files |
| `native:run` | Build and run on simulator/device |
| `native:run --watch` | Run with hot reloading |
| `native:jump` | Dev without local IDE (uses Jump app) — `ios`/`i`/`android`/`a` shorthands |
| `native:watch` | Hot reload watcher |
| `native:open` | Open in Xcode/Android Studio |
| `native:tail` | Tail Laravel logs (Android) |
| `native:package {platform}` | Build signed release |
| `native:release {type}` | Bump version (patch/minor/major) |
| `native:credentials` | Generate signing credentials |
| `native:plugin:create` | Scaffold new plugin |
| `native:plugin:register` | Discover & register one or many plugins (v3.1: no args = auto-discover) |
| `native:plugin:list` | List registered plugins |
| `native:plugin:validate` | Validate plugin structure |
| `native:plugin:boost` | Generate Boost AI guidelines |

Full flag reference: `references/getting-started.md` § Artisan Commands, or [pinned `commands.md`](https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/getting-started/commands.md).

## EDGE Components (Native UI)

Blade components compiled to truly native UI at runtime:

```blade
<native:bottom-nav label-visibility="labeled">
    <native:bottom-nav-item id="home" icon="home" label="Home" url="/home" :active="true" />
</native:bottom-nav>

<native:top-bar title="Dashboard" subtitle="Welcome">
    <native:top-bar-action id="search" icon="search" label="Search" :url="route('search')" />
</native:top-bar>

<native:side-nav gestures-enabled="true">
    <native:side-nav-header title="My App" subtitle="user@example.com" icon="person" />
    <native:side-nav-item id="home" label="Home" icon="home" url="/home" />
</native:side-nav>
```

Cross-platform icons: `icon="home"` → SF Symbol on iOS, Material Icon on Android.

## Required .env Keys

```dotenv
NATIVEPHP_APP_ID=com.yourcompany.yourapp    # Bundle ID — never change after publish
NATIVEPHP_APP_VERSION="DEBUG"                # Use semver for releases
NATIVEPHP_APP_VERSION_CODE="1"               # Integer build number
# Optional:
NATIVEPHP_DEEPLINK_SCHEME=myapp              # Custom URL scheme
NATIVEPHP_DEEPLINK_HOST=example.net          # Universal/Associated Domain links
NATIVEPHP_DEVELOPMENT_TEAM=ABC123            # Apple Team ID
NATIVEPHP_START_URL=/dashboard               # Initial route on app launch
NATIVEPHP_RUNTIME_MODE=persistent            # v3.1: 'persistent' (default) or 'classic'
NATIVEPHP_ANDROID_MIN_SDK=26                 # v3.1: configurable (default 26)
QUEUE_CONNECTION=database                    # v3.1: enables background queue worker
```

## Reference Files

- **`references/getting-started.md`** — Environment setup (iOS/Android requirements), installation, configuration (incl. Persistent Runtime, Android SDK keys, Android build flags, App Store Connect), Vite integration, hot reloading, deployment, app signing, CI/CD, full Artisan command reference
- **`references/basics-and-events.md`** — WebView configuration, Tailwind v3 vs v4 compatibility, safe area CSS, native functions (PHP + JS), event system (sync/async, custom events, 3 listener methods), assets, app icon, splash screens
- **`references/concepts.md`** — Authentication patterns (Sanctum, OAuth, `Browser::auth()`), SQLite databases, deep links (custom scheme + associated domains), push notifications (Firebase/FCM setup), security (SecureStorage, device-unique APP_KEY, encryption)
- **`references/queues.md`** — Background job worker (ZTS PHP), QUEUE_CONNECTION=database setup, example jobs
- **`references/edge-components.md`** — Bottom navigation, top bar, side navigation — complete Blade syntax, all props, icon 4-tier resolution strategy with cross-platform examples
- **`references/plugins/creating.md`** — Scaffolding, directory structure, `composer.json`, `nativephp.json` manifest
- **`references/plugins/bridge-functions.md`** — PHP → Swift/Kotlin call flow, error handling
- **`references/plugins/events.md`** — Dispatching events from native code
- **`references/plugins/lifecycle-hooks.md`** — `pre_compile`, `post_compile`, `copy_assets`, `post_build` hooks + helper methods
- **`references/plugins/advanced.md`** — Permissions, dependencies (Gradle, CocoaPods, Swift packages), secrets, entitlements, background modes, assets
- **`references/plugins/best-practices.md`** — Validation, testing (physical device requirements), common errors, marketplace review checklist
