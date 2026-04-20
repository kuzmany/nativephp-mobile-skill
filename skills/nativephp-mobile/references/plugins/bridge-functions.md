# Bridge Functions (PHP ↔ Native) — NativePHP Mobile v3

**Source (pinned):** `https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/plugins/bridge-functions.md`

## Call Flow

PHP invokes `nativephp_call()` → native bridge locates the function via manifest mapping → Swift (iOS) or Kotlin (Android) executes → JSON result returned to PHP.

## PHP Facade Side

```php
public function doSomething(array $options = []): mixed
{
    if (function_exists('nativephp_call')) {
        $result = nativephp_call('MyPlugin.DoSomething', json_encode($options));
        return json_decode($result)?->data;
    }
    return null;
}
```

The `'MyPlugin.DoSomething'` string must match the `bridge_functions[].name` in `nativephp.json`.

## Swift Implementation (iOS)

```swift
enum MyPluginFunctions {
    class DoSomething: BridgeFunction {
        func execute(parameters: [String: Any]) throws -> [String: Any] {
            let input = parameters["option"] as? String ?? ""
            // Do native work...
            return BridgeResponse.success(data: ["result": "completed"])
        }
    }
}
```

Error response:
```swift
return BridgeResponse.error(message: "Something went wrong")
```

## Kotlin Implementation (Android)

```kotlin
package com.myvendor.plugins.myplugin

import com.nativephp.mobile.bridge.BridgeFunction
import com.nativephp.mobile.bridge.BridgeResponse

object MyPluginFunctions {
    class DoSomething : BridgeFunction {
        override fun execute(parameters: Map<String, Any>): Map<String, Any> {
            val input = parameters["option"] as? String ?: ""
            // Do native work...
            return BridgeResponse.success(mapOf("result" to "completed"))
        }
    }
}
```

Error response:
```kotlin
return BridgeResponse.error("Something went wrong")
```

## Manifest Mapping

Each bridge function entry maps a single PHP name to one iOS class path and one Android class path:

```json
{
    "bridge_functions": [
        {
            "name": "MyPlugin.DoSomething",
            "ios": "MyPluginFunctions.DoSomething",
            "android": "com.myvendor.plugins.myplugin.MyPluginFunctions.DoSomething"
        }
    ]
}
```

## Gotchas

- **Name mismatch** between manifest and class path → "Bridge function not found" error at runtime
- Always rebuild (`native:run`) after changing native code — it recompiles the bridge
- Parameters are JSON-encoded on the PHP side; keep them serializable
- Return `[String: Any]` / `Map<String, Any>` — `BridgeResponse.success()` wraps it into the standard response shape
