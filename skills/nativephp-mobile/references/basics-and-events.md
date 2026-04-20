# The Basics — NativePHP Mobile v3

**Source:** `https://github.com/NativePHP/nativephp.com/tree/main/resources/views/docs/mobile/3/the-basics`

## WebView Configuration

The app UI runs inside a native WebView. Standard HTML/CSS/JS applies.

### Viewport Meta Tag
```html
<meta name="viewport" content="width=device-width, initial-scale=1">

<!-- Disable zoom: -->
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">

<!-- Edge-to-edge (required for proper safe area handling): -->
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

### Safe Area CSS Variables
Injected at runtime by NativePHP:
- `--inset-top`, `--inset-bottom`, `--inset-left`, `--inset-right`

Helper class — apply to body or containers to auto-pad safe areas:
```html
<body class="nativephp-safe-area">
```

Tailwind + manual inset combo (e.g. fixed-position header visible through the safe area):
```html
<body class="nativephp-safe-area">
<div class="fixed top-0 left-0 w-full bg-red-500 pl-[var(--inset-left)] pr-[var(--inset-right)]">
    ...
</div>
```

### Status Bar Style (Android)
Config key: `nativephp.status_bar_style` → `auto` | `light` | `dark`

### WebView Compatibility (Tailwind v4 warning)
On Android the WebView is the device's system WebView — older Android versions ship older engines. **Tailwind CSS v4** uses `@theme` and other CSS features unsupported on older WebView versions. If you target a lower `min_sdk` for device reach (v3.1 supports down to API 26 / Android 8), stay on **Tailwind v3** or use another CSS framework that produces compatible output. Test on emulators running your minimum supported Android version.

## Native Functions

### PHP Facades
```php
use Native\Mobile\Facades\Biometrics;
use Native\Mobile\Facades\Browser;
use Native\Mobile\Facades\Camera;
use Native\Mobile\Facades\Dialog;
use Native\Mobile\Facades\Geolocation;
use Native\Mobile\Facades\Haptics;
use Native\Mobile\Facades\Microphone;
use Native\Mobile\Facades\Network;
use Native\Mobile\Facades\PushNotifications;
use Native\Mobile\Facades\Scanner;
use Native\Mobile\Facades\SecureStorage;
use Native\Mobile\Facades\Share;
use Native\Mobile\Facades\Device;
use Native\Mobile\Facades\File;
use Native\Mobile\Facades\System;
```

> **Note:** Core plugin API pages (per-facade method docs) are not yet written in v3 docs. The facades above are known from usage throughout the documentation. `Device` and `File` are available but not documented with specific methods yet.

### JavaScript Library Setup
Add to `package.json` and run `npm install` after:
```json
{
    "imports": {
        "#nativephp": "./vendor/nativephp/mobile/resources/dist/native.js"
    }
}
```

Usage (Vue example):
```js
import { On, Off, Microphone, Events } from '#nativephp';

On(Events.Microphone.MicrophoneRecorded, (data) => {
    // handle recording
});

Microphone.record();
```

The JS library mirrors the PHP API with full TypeScript type support.

## Event System

### Synchronous Functions (immediate result)
```php
Haptics::vibrate();
System::flashlight();
Dialog::toast('Hello!');
```

### Asynchronous Functions (fire event on completion)
```php
Camera::getPhoto();              // → PhotoTaken event
Biometrics::prompt();            // → Completed event
PushNotifications::getToken();   // → TokenGenerated event
```

### Three Ways to Listen for Events

#### 1. `Native.on()` Global Helper (Vanilla JS / Blade)
```blade
@use(Native\Mobile\Events\Alert\ButtonPressed)
<script>
    Native.on(@js(ButtonPressed::class), (index, label) => {
        console.log(`Button ${label} pressed at index ${index}`);
    })
</script>
```

#### 2. `On` Import (SPA Frameworks — Vue/React/Svelte)
```js
import { On, Off, Events } from '#nativephp';

// Built-in events
On(Events.Alert.ButtonPressed, (index, label) => { ... });

// Custom events
On('App\\Events\\MyCustomEvent', (data) => { ... });

// Cleanup (important for SPA component lifecycle)
Off(Events.Alert.ButtonPressed, handler);
```

#### 3. `#[OnNative()]` Livewire Attribute
```php
use Native\Mobile\Attributes\OnNative;
use Native\Mobile\Events\Camera\PhotoTaken;

#[OnNative(PhotoTaken::class)]
public function handlePhoto(string $path)
{
    $this->photoPath = $path;
}
```

### Custom Events
```php
// Define custom event class
class MyButtonPressedEvent extends ButtonPressed {}

// Pass to async function
Dialog::alert('Warning!', 'Are you sure?', ['Cancel', 'Do it!'])
    ->event(MyButtonPressedEvent::class);

// Handle in Livewire
#[OnNative(MyButtonPressedEvent::class)]
public function buttonPressed()
{
    // handle custom event
}
```

### Backend PHP Event Listening
Events are dual-dispatched — they fire to both the WebView (JS/Livewire) AND the Laravel event bus simultaneously:
```php
// Standard Laravel event listener
class UpdateAvatar
{
    public function handle(PhotoTaken $event): void
    {
        $imageData = base64_encode(file_get_contents($event->path));
        $this->api->updateAvatar($imageData);
    }
}
```

## Assets

- Run `npm run build` before `native:run` when using Vite/Tailwind
- All files from Laravel root are included in the bundle
- Access files via **relative paths** using Laravel path helpers
- `storage_path()` points outside app root

### `mobile_public` Disk
For user-generated files that must survive app updates:
```php
Storage::disk('mobile_public')->url('user_content.jpg');
```

Pro tip: Set `FILESYSTEM_DISK=mobile_public` in `.env` to make it the default.

## App Icon

Place `public/icon.png`:
- 1024x1024 PNG
- No transparency
- Requires GD PHP extension (~2GB memory during build)
- Auto-resized for all Android densities + iOS sizes

## Splash Screens

- `public/splash.png` (light mode)
- `public/splash-dark.png` (dark mode)
- Minimum 1080x1920 PNG
- GD extension required
