# Plugin Best Practices — NativePHP Mobile v3

**Source:**
- `https://github.com/NativePHP/nativephp.com/blob/main/resources/views/docs/mobile/3/plugins/validation-testing.md`
- `https://github.com/NativePHP/nativephp.com/blob/main/resources/views/docs/mobile/3/plugins/best-practices.md`

## Validate

```shell
php artisan native:plugin:validate
```

## Common Errors

| Error | Fix |
|-------|-----|
| "Bridge function not found" | Class name mismatch between manifest and native code |
| "Invalid manifest JSON" | Syntax error in `nativephp.json` |
| "Hook command not registered" | Missing from service provider |
| Plugin not discovered | Check `"type": "nativephp-plugin"` in `composer.json` + run `composer dump-autoload` |
| Native function not found | Rebuild after native code changes (`native:run`) |
| Events not firing | Check main-thread dispatch (Kotlin), event class FQCN, `#[OnNative]` reference |

## Testing Requirements

- **Physical Android device** required (not just emulator) for camera, biometrics, Bluetooth, NFC, GPS
- **Physical iOS device** required — simulator will not test camera, biometrics, Bluetooth, NFC, GPS
- Verify against each supported UI stack: Livewire v3, Livewire v4, Inertia + Vue, Inertia + React

## AI Boost for Plugin Dev

```shell
php artisan native:plugin:boost {plugin?}    # Generate AI guidelines
php artisan native:plugin:install-agent       # Install AI dev agents
```

Useful when using Claude/Cursor/etc. — these commands generate project-specific guidelines the AI assistant can reference.

## Marketplace Automated Review Checklist

Plugins submitted to the NativePHP Plugin Marketplace are checked for:

- iOS native code present in `resources/ios/Sources/`
- Android native code present in `resources/android/src/`
- JS library present in `resources/js/`
- Support email published in README
- `nativephp/mobile` listed in `composer.json` require
- iOS and Android `min_version` declared in manifest

## JS Library Stub

If your plugin exposes events to SPA frameworks, ship a JS stub in `resources/js/` and document how consumers import it via the `#nativephp` alias pattern (see `basics-and-events.md`).

## Versioning

- Align your plugin's major version with the `nativephp/mobile` major version you support (`"nativephp/mobile": "^3.0"` in require)
- Bump your plugin's major when the NativePHP baseline breaks your bridge (e.g. renamed bridge API)
- Bump minor for new bridge functions or new events
- Bump patch for bug fixes
