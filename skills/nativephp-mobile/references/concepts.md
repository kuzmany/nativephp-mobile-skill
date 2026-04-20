# Concepts — NativePHP Mobile v3

**Source:** `https://github.com/NativePHP/nativephp.com/tree/main/resources/views/docs/mobile/3/concepts`

> Queues have their own dedicated reference: see `references/queues.md`.

## Authentication

### Key Principle
Never use on-device data alone to authenticate. Always verify with an external service.

### Recommended Pattern: Auth Tokens
- Short-lived auth token (days) + longer-lived refresh token (~30 days)
- Store BOTH in SecureStorage
- **Token existence is not sufficient** — always exercise the token against the server to verify it is still valid
- Generate unique credentials per installation rather than distributing identical keys

### Laravel Sanctum
Easy token generation — but enable token expiration (disabled by default):
```php
// config/sanctum.php
'expiration' => 60 * 24 * 2, // 2 days in minutes
```

### OAuth with `Browser::auth()`
Purpose-built for secure OAuth code passing:
```dotenv
NATIVEPHP_DEEPLINK_SCHEME=myapp
```
```php
use Native\Mobile\Facades\Browser;

// Opens system browser for OAuth, redirects back via deep link
Browser::auth('https://provider.com/auth?redirect=myapp://auth/callback');
```

### Network Check Before API Calls
```php
use Native\Mobile\Facades\Network;
$status = Network::status();
```

### Security for Auth Endpoints
- Use rate limiting (no CSRF available from mobile)
- Add authentication key/API key validation
- Always HTTPS

## Databases

### SQLite Only
Deliberate security decision — prevents embedding production DB credentials in the app bundle.

### Zero Configuration
NativePHP automatically:
1. Switches connection to SQLite
2. Creates the database file
3. Runs migrations on each boot

### Seeding via Migrations (preferred for mobile)
```php
php artisan make:migration seed_app_settings
```
```php
public function up(): void
{
    DB::table('settings')->insert([
        ['key' => 'theme', 'value' => 'dark'],
        ['key' => 'language', 'value' => 'en'],
    ]);
}
```

**Critical:** Test migrations for both fresh installs AND existing app updates.

### No Remote DB Access
Use API-first pattern for server data sync:
- Store auth tokens via `SecureStorage`
- Use HTTPS always
- Cache locally in SQLite for offline support
- Token-based auth (Sanctum, Passport, OAuth2)

## Deep Links

### Custom URL Scheme (`myapp://some/path`)
```dotenv
NATIVEPHP_DEEPLINK_SCHEME=myapp
```
- Only works when app is installed
- No fallback — link fails if app not present
- Good for internal app-to-app communication

### Associated Domains / Universal Links (`https://example.net/some/path`)
```dotenv
NATIVEPHP_DEEPLINK_HOST=example.net
```
- Falls back to browser if app not installed
- Better for marketing, discovery, sharing

### Required Server-Hosted Files

**iOS** — `https://example.net/.well-known/apple-app-site-association`:
```json
{
    "applinks": {
        "apps": [],
        "details": [{
            "appID": "TEAMID.com.yourcompany.yourapp",
            "paths": ["*"]
        }]
    }
}
```

**Android** — `https://example.net/.well-known/assetlinks.json`:
```json
[{
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
        "namespace": "android_app",
        "package_name": "com.yourcompany.yourapp",
        "sha256_cert_fingerprints": ["..."]
    }
}]
```

**Note:** Associated Domains don't work in simulators — test on real device with public server.

### Use Cases
NFC tags, QR codes, email/SMS marketing campaigns, web-to-app flows.

## Push Notifications

### Platform: Firebase Cloud Messaging (FCM)
Handles both iOS (via APNS) and Android.

### Setup
1. Create Firebase project at https://console.firebase.google.com
2. Download `google-services.json` (Android) — place in Laravel app root
3. Download `GoogleService-Info.plist` (iOS) — place in Laravel app root
4. NativePHP handles the rest during build

### Get Push Token
Request the token early during app startup. Tokens can change during app restoration, backups, updates, or internal FCM operations — always request and store a fresh token at startup.

```php
use Native\Mobile\Facades\PushNotifications;

PushNotifications::getToken();
// Triggers TokenGenerated event if user approves
```

### Listen for Token (Livewire)
```php
use Native\Mobile\Events\PushNotification\TokenGenerated;

#[OnNative(TokenGenerated::class)]
public function storePushToken(APIService $api, string $token)
{
    $api->storePushToken($token);
}
```

### Server-Side Sending
Requires Firebase service account JSON (`fcm-service-account.json`). Implementation is standard Firebase — not NativePHP-specific.

## Security

### Key Principles
- Treat the device environment as potentially hostile
- Generate unique keys per installation
- Always use HTTPS
- OAuth2 with short-lived tokens (< 48 hours, high entropy)
- Rate limit auth endpoints

### SecureStorage (Keychain/Keystore)
```php
use Native\Mobile\Facades\SecureStorage;

SecureStorage::set('auth_token', $token);
$token = SecureStorage::get('auth_token');
SecureStorage::delete('auth_token');
```

Properties:
- Data persists across app sessions
- Only accessible by this app
- For small amounts of text (few KBs max)
- Backed by iOS Keychain / Android Keystore

### Device-Unique APP_KEY
NativePHP generates a **unique `APP_KEY` per device** on first run, stored in SecureStorage.

Use the `Crypt` facade normally:
```php
// Encrypt larger data using device-unique APP_KEY
$encrypted = Crypt::encryptString($sensitiveContent);
Storage::put('secure_file', $encrypted);

// Decrypt later
$decrypted = Crypt::decryptString(Storage::get('secure_file'));
```

**Warning:** Encrypted data is tied to the device key. Cannot be recovered if app is deleted or device is lost. Plan for this in UX.

### User-Provided API Keys
If users connect personal API keys (e.g., third-party services), store them encrypted via `Crypt` and decrypt only when needed for the API call.
