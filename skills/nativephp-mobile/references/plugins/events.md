# Plugin Events — NativePHP Mobile v3

**Source (pinned):** `https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/plugins/events.md`

Plugins can dispatch Laravel events from native code. These events are dual-dispatched: they fire to both the WebView (JS / Livewire) and the Laravel event bus.

## Register Events in the Manifest

```json
{
    "events": [
        "Vendor\\MyPlugin\\Events\\ProcessingComplete"
    ]
}
```

## Dispatching from Swift (iOS)

```swift
LaravelBridge.shared.send?(
    "Vendor\\MyPlugin\\Events\\ProcessingComplete",
    ["result": processedData, "id": requestId]
)
```

## Dispatching from Kotlin (Android)

Must be posted to the main thread:

```kotlin
import android.os.Handler
import android.os.Looper

Handler(Looper.getMainLooper()).post {
    NativeActionCoordinator.dispatchEvent(
        activity,
        "Vendor\\MyPlugin\\Events\\ProcessingComplete",
        payload.toString()
    )
}
```

## Listening in PHP

Standard Laravel event listener:
```php
class HandleProcessingComplete
{
    public function handle(ProcessingComplete $event): void
    {
        // $event->result, $event->id
    }
}
```

Livewire attribute:
```php
#[OnNative(ProcessingComplete::class)]
public function onComplete(string $result, string $id) { ... }
```

JS / Blade:
```js
Native.on('Vendor\\MyPlugin\\Events\\ProcessingComplete', (data) => { ... })
```

## Common Issues

| Symptom | Fix |
|---------|-----|
| Event never fires on Android | Dispatch must run on main thread (see Kotlin example) |
| Event class not recognized | Ensure FQCN in manifest matches PHP class exactly, including backslashes |
| Livewire handler not called | Class reference in `#[OnNative]` must match and be discoverable (correct namespace + autoload) |
