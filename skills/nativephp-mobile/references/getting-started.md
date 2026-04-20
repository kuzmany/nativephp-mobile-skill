# Getting Started — NativePHP Mobile v3.0–3.1

**Source:** `https://github.com/NativePHP/nativephp.com/tree/main/resources/views/docs/mobile/3/getting-started`

## Environment Setup

### Requirements
- PHP **8.3–8.5** (auto-detected from `composer.json`; 8.3 is the minimum supported)
- Laravel 11+
- Recommended: Laravel Herd for PHP management

### iOS (Mac only — Apple limitation)
- macOS with Apple silicon (M1+)
- Xcode 16.0+
- Xcode Command Line Tools: `xcode-select --install`
- Homebrew + CocoaPods: `brew install cocoapods`
- Apple Developer account NOT required for simulator; required for real devices, App Store, push notifications
- iOS deployment target: **18.0+**

### Android (Mac + Windows)
- Android Studio 2024.2.1+
- Android SDK API 36 recommended; **minimum supported min_sdk = 26** (Android 8)
- JDK (check Gradle compatibility matrix)
- Windows: 7zip required, **NO WSL support — must run on native Windows**

### Android Environment Variables (macOS)
```shell
export JAVA_HOME=$(/usr/libexec/java_home -v 17)
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$JAVA_HOME/bin:$ANDROID_HOME/emulator:$ANDROID_HOME/tools:$ANDROID_HOME/tools/bin:$ANDROID_HOME/platform-tools
```

## Installation

### Pre-install `.env`
```dotenv
NATIVEPHP_APP_ID=com.yourcompany.yourapp
NATIVEPHP_APP_VERSION="DEBUG"
NATIVEPHP_APP_VERSION_CODE="1"
```

### Install Steps
```shell
composer require nativephp/mobile
php artisan native:install
php artisan native:run
```

### ICU Choice
v3.1 ships **full ICU support on iOS by default** — Filament and other `intl`-dependent packages now work on both platforms.

On Android you are still prompted during install: include ICU-enabled PHP binaries (~30 MB added) if you need the `intl` extension. Override via `--with-icu` / `--without-icu`.

### `nativephp/` Directory
Created at project root with native project files. Treat as ephemeral:
- Add to `.gitignore`
- Rebuilt with `native:install --force`

### Windows Defender
Add `C:\temp` and project folder to exclusions to speed up Composer installs.

## Configuration (`config/nativephp.php`)

### Bundle ID (`NATIVEPHP_APP_ID`)
Reverse-DNS format: `com.yourcompany.yourapp`. **Never change after publishing.**

### Version (`NATIVEPHP_APP_VERSION`)
- `DEBUG` during development — forces PHP re-extraction every boot (slower but fresh code)
- Semver `1.2.3` for releases

### Persistent Runtime (v3.1)
```php
'runtime' => [
    'mode' => env('NATIVEPHP_RUNTIME_MODE', 'persistent'),
    'reset_instances' => true,
    'gc_between_dispatches' => false,
],
```
- `mode` — `persistent` (default, reuses Laravel kernel across requests; ~5-30ms vs ~200-300ms in classic) or `classic`. Falls back to classic automatically if persistent boot fails.
- `reset_instances` — Clear resolved facade instances between dispatches (default `true`)
- `gc_between_dispatches` — Run garbage collection between dispatches (default `false`, enable if memory grows over time)

Livewire state, router state, and facade instances are reset automatically. Only add an `onReset()` hook if you have custom static state that accumulates between requests.

### Start URL
```php
'start_url' => env('NATIVEPHP_START_URL', '/'),
```
Land users on `/dashboard` or `/onboarding` instead of `/`.

### Cleanup Options
```php
'cleanup_env_keys' => ['AWS_SECRET', ...],     // Keys stripped from .env before bundling
'cleanup_exclude_files' => ['tests/', ...],     // Files excluded from bundle
```

### Permissions (all disabled by default)
```php
'permissions' => [
    'biometric' => false,           // or string for custom iOS description
    'camera' => false,
    'location' => false,
    'microphone' => false,
    'microphone_background' => false,
    'network_state' => true,        // enabled by default
    'nfc' => false,
    'push_notifications' => false,
    'storage_read' => false,
    'storage_write' => false,
    'scanner' => false,
    'vibrate' => false,
],
```

### Orientation (per device type)
```php
'orientation' => [
    'iphone' => ['portrait' => true, 'upside_down' => false, 'landscape_left' => false, 'landscape_right' => false],
    'android' => ['portrait' => true, 'landscape' => false],
],
```

### iPad Support
```php
'ipad' => true,   // WARNING: once published with iPad, cannot remove without new App ID
```

### Status Bar Style
`nativephp.status_bar_style` → `auto` | `light` | `dark`

### Android SDK Versions (v3.1)
```php
'android' => [
    'compile_sdk' => env('NATIVEPHP_ANDROID_COMPILE_SDK', 36),
    'min_sdk' => env('NATIVEPHP_ANDROID_MIN_SDK', 26),     // Android 8
    'target_sdk' => env('NATIVEPHP_ANDROID_TARGET_SDK', 36),
],
```
Rule: `compile_sdk >= target_sdk >= min_sdk`. Lowest supported `min_sdk` is 26.

### Android Build Options (v3.1)
```php
'android' => [
    'build' => [
        'minify_enabled' => env('NATIVEPHP_ANDROID_MINIFY_ENABLED', false),
        'shrink_resources' => env('NATIVEPHP_ANDROID_SHRINK_RESOURCES', false),
        'obfuscate' => env('NATIVEPHP_ANDROID_OBFUSCATE', false),
        'debug_symbols' => env('NATIVEPHP_ANDROID_DEBUG_SYMBOLS', 'FULL'),
        'parallel_builds' => env('NATIVEPHP_ANDROID_PARALLEL_BUILDS', true),
        'incremental_builds' => env('NATIVEPHP_ANDROID_INCREMENTAL_BUILDS', true),
    ],
],
```

### App Store Connect (iOS upload without CLI flags)
```php
'app_store_connect' => [
    'api_key' => env('APP_STORE_API_KEY_PATH'),
    'api_key_id' => env('APP_STORE_API_KEY_ID'),
    'api_issuer_id' => env('APP_STORE_API_ISSUER_ID'),
    'app_name' => env('APP_STORE_APP_NAME'),
],
```
Alternative to passing `--api-key=`, `--api-key-id=`, `--api-issuer-id=` flags to `native:package`.

## Development Workflow

### Vite Integration
Add to `vite.config.js`:
```js
import { nativephpMobile, nativephpHotFile } from './vendor/nativephp/mobile/resources/js/vite-plugin.js';

export default defineConfig({
    plugins: [
        laravel({
            input: ['resources/css/app.css', 'resources/js/app.js'],
            hotFile: nativephpHotFile(),
        }),
        nativephpMobile(),
    ],
});
```
`nativephpHotFile()` returns the per-platform hot reload manifest path so Vite publishes updates to the mobile WebView.

### Platform-Specific Builds
```shell
npm run build -- --mode=ios
npm run build -- --mode=android
```

### Hot Reloading
```shell
php artisan native:watch
# or combined:
php artisan native:run --watch
```

Run both platforms simultaneously (macOS):
```shell
# Terminal 1:
php artisan native:watch ios
# Terminal 2:
php artisan native:watch android
```

**Watch paths** in `nativephp.php`:
```php
'hot_reload' => ['watch_paths' => ['app', 'routes', 'config', 'database', 'public']]
```

Add to `.gitignore`: `public/ios-hot`, `public/android-hot`

**Limitation:** Full hot reloading on real iOS devices not yet supported (simulators only for non-JS changes).

### Open in Native IDE
```shell
php artisan native:open
```

### Laravel Boost
After installing `nativephp/mobile` and `laravel/boost`, run `php artisan boost:install`. v3.1 ships Laravel Boost skill support out of the box.

## Deployment

### Release Build
```dotenv
NATIVEPHP_APP_VERSION=1.2.3
NATIVEPHP_APP_VERSION_CODE=48
```
```shell
php artisan native:run --build=release
```

### Android Packaging
```bash
# Generate signing credentials
php artisan native:credentials android

# Build APK
php artisan native:package android \
  --keystore=/path/to/keystore \
  --keystore-password=mysecret \
  --key-alias=upload \
  --key-password=mysecret

# Build AAB (required for Play Store)
php artisan native:package android --build-type=bundle \
  --keystore=/path/to/keystore \
  --keystore-password=mysecret \
  --key-alias=upload \
  --key-password=mysecret

# Upload to Play Store
php artisan native:package android --build-type=bundle \
  --keystore=/path/to/keystore \
  --keystore-password=mysecret \
  --key-alias=upload \
  --key-password=mysecret \
  --upload-to-play-store \
  --play-store-track=internal \
  --google-service-key=/path/to/key.json
```

Play Store tracks: `internal` | `alpha` | `beta` | `production`

**Android env vars:** `ANDROID_KEYSTORE_FILE`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `ANDROID_KEY_PASSWORD`

**Artifact locations:**
- APK: `nativephp/android/app/build/outputs/apk/release/app-release.apk`
- AAB: `nativephp/android/app/build/outputs/bundle/release/app-release.aab`

### iOS Packaging
```bash
php artisan native:package ios \
  --export-method=app-store \
  --api-key=/path/to/key.p8 \
  --api-key-id=ABC123 \
  --api-issuer-id=... \
  --certificate-path=/path/to/dist.p12 \
  --certificate-password=mysecret \
  --provisioning-profile-path=/path/to/profile.mobileprovision \
  --team-id=ABC123

# Upload to App Store
php artisan native:package ios ... --upload-to-app-store
```

Export methods: `app-store` | `ad-hoc` | `enterprise` | `development`

**iOS env vars:** `APP_STORE_API_KEY_PATH`, `APP_STORE_API_KEY_ID`, `APP_STORE_API_ISSUER_ID`, `IOS_DISTRIBUTION_CERTIFICATE_PATH`, `IOS_DISTRIBUTION_CERTIFICATE_PASSWORD`, `IOS_DISTRIBUTION_PROVISIONING_PROFILE_PATH`, `IOS_TEAM_ID`

### CI/CD
Use `--no-tty` for non-interactive environments.

### Auto-Increment Build Numbers
When building AABs with Play Store credentials, NativePHP auto-queries Play Store for latest build number and increments. Use `--jump-by=N` to skip ahead.

### Bifrost
https://bifrost.nativephp.com — managed service for certificate handling, team collaboration, OTA updates. Removes the need to manage keystores/provisioning profiles locally.

## Complete Artisan Commands

Full reference: [pinned `commands.md`](https://github.com/NativePHP/nativephp.com/blob/main/resources/views/docs/mobile/3/getting-started/commands.md).

### Development
| Command | Flags |
|---------|-------|
| `native:install {platform?}` | `--force`, `--fresh`, `--with-icu`, `--without-icu`, `--skip-php` |
| `native:run {os?} {udid?}` | `--build=debug\|release\|bundle`, `--watch`, `--start-url=`, `--no-tty` (v3.1: warns about unregistered plugins before build) |
| `native:watch {platform?} {target?}` | `platform` accepts `ios/i`/`android/a` |
| `native:jump` | `ios`/`i`/`android`/`a` (v3.1 shorthand), `--platform`, `--host`, `--http-port`, `--laravel-port=8000`, `--no-mdns`, `--skip-build` |
| `native:open {os?}` | `os` accepts `ios/i`/`android/a` |
| `native:tail` | Android only — tail Laravel logs |
| `native:version` | |

### Building & Release
| Command | Flags |
|---------|-------|
| `native:package {platform}` | `--build-type=release\|bundle`, `--output=`, `--jump-by=`, `--no-tty`. Android adds: `--keystore=`, `--keystore-password=`, `--key-alias=`, `--key-password=`, `--fcm-key=`, `--google-service-key=`, `--upload-to-play-store`, `--play-store-track=`, `--test-push=`, `--skip-prepare`. iOS adds: `--export-method=`, `--upload-to-app-store`, `--test-upload`, `--validate-only`, `--validate-profile`, `--rebuild`, `--clean-caches`, `--api-key=`, `--api-key-id=`, `--api-issuer-id=`, `--certificate-path=`, `--certificate-password=`, `--provisioning-profile-path=`, `--team-id=` |
| `native:release {type}` | `patch\|minor\|major` |
| `native:credentials {platform?}` | `--reset` |
| `native:check-build-number` | |

### Plugin Commands
| Command | Flags |
|---------|-------|
| `native:plugin:create` | Interactive scaffold |
| `native:plugin:list` | `--json`, `--all` |
| `native:plugin:register {plugin?}` | `--remove`, `--force` — v3.1: call without `{plugin}` to auto-discover and register all unregistered plugins |
| `native:plugin:uninstall {plugin}` | `--force`, `--keep-files` |
| `native:plugin:validate {path?}` | |
| `native:plugin:make-hook` | |
| `native:plugin:boost {plugin?}` | `--force` — generate Boost AI guidelines for plugin dev |
| `native:plugin:install-agent` | `--force`, `--all` — install AI agents for plugin development |

## Upgrade Guide (v2 → v3)

### High Impact
1. **Remove NativePHP Composer repository** — v3 no longer uses private repo/license auth. Remove `nativephp.composer.sh` from `composer.json` repositories and `auth.json`.
2. **Plugin-based architecture** — core APIs are now plugins internally (no changes to PHP code needed).
3. **Rebuild required:** `php artisan native:install --force && php artisan native:run`

### Medium Impact
- **NativeServiceProvider** — publish with `php artisan vendor:publish --tag=nativephp-plugins-provider`. Third-party plugins must be explicitly registered.
- Register plugins: `php artisan native:plugin:register vendor/some-plugin` (or without args in v3.1 to auto-discover).

## Versioning

- **Patch releases:** Laravel/PHP code only, no native rebuild needed, safe `composer update`
- **Minor releases:** May include native code changes, requires `native:install --force` + app store resubmission
- **Major releases:** Breaking changes

Recommended `composer.json` constraint: `"nativephp/mobile": "^3.0"` (caret = allow patch + minor, review before bumping major).

## Support Policy
- iOS 18+
- Android 8+ (API 26, configurable via `min_sdk` — prior to v3.1 this was API 33)
- PHP 8.3 minimum (PHP 8.4 default for new apps; 8.5 supported)
