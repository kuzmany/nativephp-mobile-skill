# Creating Plugins — NativePHP Mobile v3

**Source:** `https://github.com/NativePHP/nativephp.com/tree/main/resources/views/docs/mobile/3/plugins`

## What is a Plugin?

A Composer package bundling:
- PHP code (facades, events, service providers)
- Native code (Swift for iOS + Kotlin for Android)
- Manifest (`nativephp.json`)

All core NativePHP functionality (Camera, Biometrics, Push, etc.) is delivered as internal plugins. Third parties create new plugins the same way.

## Installing a Plugin (consumer side)

```shell
composer require vendor/nativephp-plugin-name
php artisan vendor:publish --tag=nativephp-plugins-provider    # first time only
php artisan native:plugin:register vendor/nativephp-plugin-name
php artisan native:plugin:list
php artisan native:run                                          # rebuild to compile native code
```

v3.1+ can also auto-discover: `php artisan native:plugin:register` (no args) discovers and registers all unregistered plugins.

```php
use Vendor\PluginName\Facades\PluginName;
$result = PluginName::doSomething(['option' => 'value']);
```

Uninstall: `php artisan native:plugin:uninstall vendor/nativephp-plugin-name`

## Scaffolding a New Plugin

```shell
php artisan native:plugin:create
```

## Directory Structure

```
my-plugin/
├── composer.json               # type: "nativephp-plugin"
├── nativephp.json              # manifest
├── src/
│   ├── MyPluginServiceProvider.php
│   ├── MyPlugin.php
│   ├── Facades/
│   ├── Events/
│   └── Commands/               # lifecycle hooks
├── resources/
│   ├── android/src/            # Kotlin bridge functions
│   ├── ios/Sources/            # Swift bridge functions
│   └── js/                     # JavaScript library stubs
```

## `composer.json`

```json
{
    "name": "vendor/nativephp-plugin-name",
    "type": "nativephp-plugin",
    "require": {
        "nativephp/mobile": "^3.0"
    },
    "extra": {
        "laravel": {
            "providers": ["Vendor\\MyPlugin\\MyPluginServiceProvider"]
        },
        "nativephp": {
            "manifest": "nativephp.json"
        }
    }
}
```

## Android Package Naming

Use vendor-namespaced packages to avoid conflicts:
```
com.myvendor.plugins.myplugin
```

## Manifest (`nativephp.json`) Overview

```json
{
    "namespace": "MyPlugin",
    "bridge_functions": [
        {
            "name": "MyPlugin.DoSomething",
            "ios": "MyPluginFunctions.DoSomething",
            "android": "com.myvendor.plugins.myplugin.MyPluginFunctions.DoSomething"
        }
    ],
    "events": [
        "Vendor\\MyPlugin\\Events\\SomethingHappened"
    ]
}
```

See `./advanced.md` for the full manifest schema (permissions, dependencies, secrets, background modes, entitlements, features, assets).

## Local Plugin Development

Use a path repository in your consumer app's `composer.json`:
```json
{
    "repositories": [
        {"type": "path", "url": "../packages/my-plugin"}
    ]
}
```

- PHP changes — immediate
- Native code changes — `native:run` required
- Manifest changes — `native:install --force`
