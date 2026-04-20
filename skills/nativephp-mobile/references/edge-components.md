# EDGE Components — NativePHP Mobile v3

**Source (pinned):** `https://github.com/NativePHP/nativephp.com/tree/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/edge-components`

EDGE = Element Definition and Generation Engine

## How It Works

1. Define Blade components in PHP templates
2. EDGE compiles to JSON config at runtime
3. Native rendering pipeline generates platform-native UI
4. Components live outside the WebView lifecycle — persistent, native performance

## Where to Define

Place EDGE components in Blade files rendered on every request. Recommended: `resources/views/layouts/app.blade.php`.

### Inertia Integration
Add `router` to `window` object for navigation to work:
```typescript
import { router } from '@inertiajs/vue3';
window.router = router;
```

### Features
- Supports Liquid Glass on iOS
- Auto dark/light mode
- Hot reloading compatible

## Bottom Navigation

Max 5 items. Primary app navigation.

```blade
<native:bottom-nav label-visibility="labeled">
    <native:bottom-nav-item
        id="home"
        icon="home"
        label="Home"
        url="/home"
        :active="true"
    />
    <native:bottom-nav-item
        id="search"
        icon="search"
        label="Search"
        url="/search"
    />
    <native:bottom-nav-item
        id="profile"
        icon="person"
        label="Profile"
        url="/profile"
        badge="3"
    />
    <native:bottom-nav-item
        id="alerts"
        icon="bell"
        label="Alerts"
        url="/alerts"
        news="true"
    />
</native:bottom-nav>
```

### Bottom Nav Props
| Prop | Required | Values |
|------|----------|--------|
| `label-visibility` | No | `labeled` (default), `selected`, `unlabeled` |
| `dark` | No | Boolean — force dark mode |

### Bottom Nav Item Props
| Prop | Required | Description |
|------|----------|-------------|
| `id` | Yes | Unique identifier |
| `icon` | Yes | Cross-platform icon name |
| `label` | Yes | Display text |
| `url` | Yes | Navigation URL |
| `:active` | No | Boolean — currently active |
| `badge` | No | Badge count text |
| `news` | No | Dot indicator (boolean) |

## Top Bar

```blade
<native:top-bar
    title="Dashboard"
    subtitle="Welcome back"
    background-color="#FF0000"
    text-color="#FFFFFF"
>
    <native:top-bar-action
        id="search"
        icon="search"
        label="Search"
        :url="route('search')"
    />
    <native:top-bar-action
        id="settings"
        icon="settings"
        label="Settings"
        :url="route('settings')"
    />
</native:top-bar>
```

### Top Bar Props
| Prop | Required | Description |
|------|----------|-------------|
| `title` | Yes | Bar title text |
| `subtitle` | No | Subtitle text |
| `show-navigation-icon` | No | Show back/menu icon |
| `label` | No | Label for navigation icon |
| `background-color` | No | Hex color |
| `text-color` | No | Hex color |
| `elevation` | No | Shadow depth (Android only) |

### Top Bar Action Props
| Prop | Required | Description |
|------|----------|-------------|
| `id` | Yes | Unique identifier |
| `icon` | Yes | Cross-platform icon name |
| `label` | No | Tooltip/accessibility text |
| `url` | No | Navigation URL |

**Platform behavior:** Android shows first 3 actions as icons, rest collapse to overflow menu (⋮). iOS collapses at 5+.

## Side Navigation

```blade
<native:side-nav gestures-enabled="true">
    <native:side-nav-header
        title="My App"
        subtitle="user@example.com"
        icon="person"
    />

    <native:side-nav-item
        id="home"
        label="Home"
        icon="home"
        url="/home"
        :active="true"
    />

    <native:side-nav-item
        id="favorites"
        label="Favorites"
        icon="star"
        url="/favorites"
        badge="12"
    />

    <native:side-nav-group heading="Account" :expanded="false">
        <native:side-nav-item id="profile" label="Profile" icon="person" url="/profile" />
        <native:side-nav-item id="settings" label="Settings" icon="settings" url="/settings" />
    </native:side-nav-group>

    <native:horizontal-divider />

    <native:side-nav-item
        id="help"
        label="Help & Support"
        icon="help"
        url="https://help.example.com"
        open-in-browser="true"
    />
</native:side-nav>
```

### Side Nav Props
| Prop | Required | Description |
|------|----------|-------------|
| `gestures-enabled` | No | Enable swipe to open (Android). iOS: always enabled |

### Side Nav Item Props
| Prop | Required | Description |
|------|----------|-------------|
| `id` | Yes | Unique identifier |
| `label` | Yes | Display text |
| `icon` | Yes | Cross-platform icon name |
| `url` | Yes | Navigation URL |
| `:active` | No | Currently active (boolean) |
| `badge` | No | Badge text |
| `open-in-browser` | No | Open URL in system browser |

### Side Nav Group
Groups items under a collapsible heading:
- `heading` — Group title
- `:expanded` — Default expanded state (boolean)

### Side Nav Header
- `title` — App or user name
- `subtitle` — Email or description
- `icon` — Avatar icon

### Divider
`<native:horizontal-divider />` — visual separator between sections.

## Icon System

On iOS, icons render as [SF Symbols](https://developer.apple.com/sf-symbols/). On Android, they render as [Material Icons](https://fonts.google.com/icons?icon.set=Material+Icons). You use a single consistent `icon="..."` name and the EDGE engine translates per platform.

### Smart 4-Tier Resolution

The icon system resolves each name in this order — first match wins:

1. **Direct platform icon** —
   - **iOS:** if the name contains a `.`, it's used as a direct SF Symbol path (e.g. `car.side.fill`, `figure.walk`)
   - **Android:** any Material Icon ligature name works directly (e.g. `shopping_cart`, `qr_code_2`)
2. **Manual mapping** — explicit mappings for common aliases (`home`, `settings`, `user`, `cart` → `shopping_cart`, etc.)
3. **Smart fallback** — on iOS, tries variations like `.fill`, `.circle.fill`, `.square.fill` on unmapped names
4. **Default fallback** — circle icon if nothing matches

**Net effect:** `icon="home"` works on both platforms via the manual map. `icon="car.side.fill"` is iOS-only (passes through tier 1). `icon="shopping_cart"` is Android-only (passes through tier 1). Use tier 2 names when you want one string for both platforms.

### Full Reference
Complete manual mapping table: [pinned `icons.md`](https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/edge-components/icons.md) — covers navigation, commerce, charts, time, actions, communication, status, auth, content, device/hardware, audio.

### Legacy summary

### Cross-Platform Usage (same name works on both)
```blade
icon="home"        {{-- → house.fill (iOS) / home (Android) --}}
icon="settings"    {{-- → gearshape.fill (iOS) / settings (Android) --}}
icon="search"      {{-- → magnifyingglass (iOS) / search (Android) --}}
icon="person"      {{-- → person.fill (iOS) / person (Android) --}}
icon="star"        {{-- → star.fill (iOS) / star (Android) --}}
icon="bell"        {{-- → bell.fill (iOS) / notifications (Android) --}}
icon="help"        {{-- → questionmark.circle (iOS) / help (Android) --}}
```

### Direct Platform Icons
```blade
{{-- iOS SF Symbols (contain dot) --}}
icon="car.side.fill"
icon="flashlight.on.fill"
icon="arrow.triangle.turn.up.right.diamond"

{{-- Android Material Icons (underscores) --}}
icon="qr_code_2"
icon="shopping_cart"
icon="flight_takeoff"
```

### Platform-Specific Selection
```blade
icon="{{ \Native\Mobile\Facades\System::isIos() ? 'flashlight.on.fill' : 'flashlight_on' }}"
```

### Common Icon Reference

| Category | Icons |
|----------|-------|
| Navigation | `home`, `search`, `menu`, `arrow_back`, `close`, `more_vert` |
| Business | `store`, `shopping_cart`, `receipt`, `payments`, `account_balance` |
| Charts | `bar_chart`, `pie_chart`, `trending_up`, `analytics` |
| Time | `schedule`, `calendar_today`, `timer`, `history` |
| Actions | `add`, `edit`, `delete`, `share`, `download`, `upload`, `refresh` |
| Communication | `mail`, `chat`, `phone`, `notifications`, `forum` |
| Status | `check_circle`, `error`, `warning`, `info`, `help` |
| Auth | `lock`, `key`, `fingerprint`, `visibility`, `shield` |
| Content | `photo`, `videocam`, `mic`, `volume_up`, `file_copy` |
| Device | `bluetooth`, `wifi`, `battery_full`, `flashlight_on`, `gps_fixed` |
