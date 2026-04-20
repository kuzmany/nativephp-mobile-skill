# Plugin Advanced Configuration — NativePHP Mobile v3

**Source (pinned):**
- `https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/plugins/permissions-dependencies.md`
- `https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/plugins/advanced-configuration.md`

## Permissions

### Android
Merged into `AndroidManifest.xml` automatically:
```json
{
    "android": {
        "permissions": [
            "android.permission.CAMERA",
            "android.permission.RECORD_AUDIO"
        ]
    }
}
```

### iOS
Merged into `Info.plist` as `NS*UsageDescription` entries:
```json
{
    "ios": {
        "info_plist": {
            "NSCameraUsageDescription": "This app needs camera access"
        }
    }
}
```

## Dependencies

### Android Gradle
```json
{
    "android": {
        "dependencies": {
            "implementation": ["com.google.mlkit:barcode-scanning:17.2.0"]
        }
    }
}
```

### Android Custom Maven Repositories (with auth)
```json
{
    "android": {
        "repositories": [{
            "url": "https://maven.example.com/releases",
            "credentials": {
                "username": "user",
                "password": "${MAVEN_TOKEN}"
            }
        }]
    }
}
```

### iOS CocoaPods
```json
{
    "ios": {
        "dependencies": {
            "pods": [{"name": "GoogleMLKit/BarcodeScanning", "version": "~> 4.0"}]
        }
    }
}
```

### iOS Swift Packages
```json
{
    "ios": {
        "dependencies": {
            "swift_packages": [{"url": "https://github.com/example/package", "version": "1.0.0"}]
        }
    }
}
```

## Environment Variable Placeholders

`${ENV_VAR}` syntax is supported throughout the manifest for sensitive values.

## Secrets

Build fails with a helpful message if a required secret is missing:
```json
{
    "secrets": {
        "MAPBOX_TOKEN": {
            "description": "Mapbox API token for map rendering",
            "required": true
        }
    }
}
```

## Android Manifest Components

Register Activities, Services, Receivers, Providers via the manifest.

### Features
```json
{
    "android": {
        "features": [
            {"name": "android.hardware.camera", "required": true}
        ]
    }
}
```

### Meta-Data
Application-level `<meta-data>` for SDK configuration.

## Declarative Assets

```json
{
    "assets": {
        "android": {
            "models/detector.tflite": "assets/ml/detector.tflite"
        },
        "ios": {
            "models/detector.mlmodel": "Resources/ml/detector.mlmodel"
        }
    }
}
```
Text-based assets support `${ENV_VAR}` placeholder substitution.

## iOS Background Modes

```json
{
    "ios": {
        "background_modes": ["audio", "fetch", "processing", "location", "remote-notification"]
    }
}
```

## iOS Entitlements

App Groups, Associated Domains, HealthKit, Maps, etc. — declare via manifest.

## Minimum Platform Versions

```json
{
    "android": {"min_version": 33},
    "ios": {"min_version": "18.0"}
}
```
NativePHP baseline (v3.1): Android API 26 minimum, iOS 18. A plugin can raise these for its consumer app; it cannot lower them below the framework baseline.
