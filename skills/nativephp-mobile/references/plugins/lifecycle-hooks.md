# Lifecycle Hooks — NativePHP Mobile v3

**Source:** `https://github.com/NativePHP/nativephp.com/blob/main/resources/views/docs/mobile/3/plugins/lifecycle-hooks.md`

Plugins can run custom Laravel console commands at specific points during `native:install` / `native:run` / `native:package`.

## Available Hooks

| Hook | Fires |
|------|-------|
| `pre_compile` | Before the native build starts |
| `post_compile` | After the native build finishes |
| `copy_assets` | During the asset copy phase — ideal for shipping native-only files (ML models, fonts, etc.) |
| `post_build` | After the packaged artifact is produced |

## Generate a Hook Command

```shell
php artisan native:plugin:make-hook
```

## Hook Command Structure

```php
use Native\Mobile\Plugins\Commands\NativePluginHookCommand;

class CopyAssetsCommand extends NativePluginHookCommand
{
    protected $signature = 'nativephp:my-plugin:copy-assets';

    public function handle(): int
    {
        if ($this->isAndroid()) {
            $this->copyToAndroidAssets('models/model.tflite', 'models/model.tflite');
        }

        if ($this->isIos()) {
            $this->copyToIosBundle('models/model.mlmodel', 'models/model.mlmodel');
        }

        return self::SUCCESS;
    }
}
```

## Helper Methods

| Method | Returns / Does |
|--------|----------------|
| `platform()` | Current platform string |
| `isIos()` / `isAndroid()` | Platform detection |
| `buildPath()` | Path to native build directory |
| `pluginPath()` | Path to plugin root |
| `appId()` | Consumer app bundle identifier |
| `copyToAndroidAssets($src, $dest)` | Copy file into Android assets |
| `copyToIosBundle($src, $dest)` | Copy file into iOS bundle |
| `downloadIfMissing($url, $dest)` | Download file only if not cached |
| `unzip($zipPath, $extractTo)` | Extract ZIP archive |

## Declare Hooks in Manifest

```json
{
    "hooks": {
        "copy_assets": "nativephp:my-plugin:copy-assets",
        "post_build": "nativephp:my-plugin:post-build"
    }
}
```

## Init Functions

Native code that must run once at app startup (before bridge functions become available) can be registered as init functions. Useful for: registering notification channels, initialising SDKs, pre-warming caches.
