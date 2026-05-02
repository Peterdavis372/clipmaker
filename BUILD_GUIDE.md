# 📱 ClipVault for Android — Full Build Guide

ClipVault is a native Android clipboard manager that automatically captures everything you copy — text, links, emails, phone numbers, code, colors, images — and organizes them into a searchable, sortable history.

---

## 🏗️ Architecture Overview

```
ClipVault/
├── data/
│   ├── model/ClipItem.kt          # Data model with auto type detection
│   ├── db/ClipVaultDatabase.kt    # Room SQLite database
│   ├── db/ClipItemDao.kt          # Database queries
│   └── repository/ClipRepository.kt
├── service/
│   ├── ClipboardMonitorService.kt  # Foreground service (polling + listener)
│   ├── ClipVaultAccessibilityService.kt  # Background capture (Android 10+)
│   └── BootReceiver.kt             # Auto-start on boot
├── ui/
│   ├── home/MainActivity.kt        # Main screen
│   ├── home/MainViewModel.kt       # State + filtering
│   ├── home/ClipAdapter.kt         # RecyclerView adapter
│   ├── detail/DetailActivity.kt    # Full clip view
│   └── settings/SettingsActivity.kt
└── utils/ClipTypeDetector.kt       # Detects: text, URL, email, phone, code, JSON, color
```

---

## ⚙️ How Clipboard Capture Works

### Two complementary strategies:

**1. Foreground Service + Listener** (works when app is visible/recent)
- `ClipboardMonitorService` runs as a foreground service
- Registers `OnPrimaryClipChangedListener` for instant detection
- Polls every 500ms as fallback
- Visible notification: "Monitoring clipboard…"

**2. Accessibility Service** (works in FULL background on Android 10+)
- Android 10+ blocked background clipboard reads for privacy
- The only reliable workaround is an Accessibility Service
- User must grant it once in Settings → Accessibility
- Does NOT read screen content — only clipboard

---

## 🚀 Step-by-Step Build Instructions

### Prerequisites

| Tool | Version | Download |
|------|---------|----------|
| Android Studio | Hedgehog (2023.1.1) or newer | https://developer.android.com/studio |
| JDK | 17+ | Bundled with Android Studio |
| Android SDK | API 34 | Install via SDK Manager |
| Android NDK | Not needed | — |

> **Minimum device:** Android 8.0 (API 26) · **Target:** Android 14 (API 34)

---

### Step 1 — Open in Android Studio

```bash
# Clone or download the ClipVault folder, then open in Android Studio:
# File → Open → select the ClipVault/ directory
```

Android Studio will detect it as a Gradle project automatically.

---

### Step 2 — Sync Gradle

Android Studio will prompt: **"Gradle files have changed"** → click **Sync Now**.

If it doesn't auto-sync:
```
File → Sync Project with Gradle Files
```

Wait for all dependencies to download (~2–3 minutes first time).

---

### Step 3 — Configure SDK Path

If you see SDK errors, go to:
```
File → Project Structure → SDK Location
```
Set your Android SDK path (usually `C:\Users\<you>\AppData\Local\Android\Sdk` on Windows
or `/Users/<you>/Library/Android/sdk` on Mac).

---

### Step 4 — Build the APK

#### Debug build (for testing):
```
Build → Build Bundle(s) / APK(s) → Build APK(s)
```
Output: `app/build/outputs/apk/debug/app-debug.apk`

#### Release build (for distribution):
```
Build → Generate Signed Bundle / APK → APK
```
You'll need to create a keystore (Android Studio guides you through it).

Output: `app/build/outputs/apk/release/app-release.apk`

---

### Step 5 — Install on Device

#### Option A — Direct via Android Studio:
1. Enable Developer Options on your phone: Settings → About Phone → tap Build Number 7x
2. Enable USB Debugging: Settings → Developer Options → USB Debugging
3. Connect phone via USB
4. Click ▶ Run in Android Studio

#### Option B — Manual APK install:
```bash
# Via ADB
adb install app/build/outputs/apk/debug/app-debug.apk

# Or transfer the APK file to your phone and open it
# (Enable "Install from unknown sources" in Settings → Security)
```

---

### Step 6 — First-Time Setup in App

1. **Open ClipVault** — it starts the monitoring service automatically
2. **Grant Notification permission** (Android 13+ — required for the service notification)
3. **Enable Accessibility Service** — the app will prompt you:
   - Go to: **Settings → Accessibility → Downloaded Apps → ClipVault → Enable**
   - This is required for background clipboard capture
4. **Start copying things!** Everything you copy now appears in ClipVault instantly.

---

## 🔧 Key Features

| Feature | How It Works |
|---------|-------------|
| Auto-capture | Foreground service + Accessibility Service |
| Type detection | Regex-based: URL, email, phone, hex color, JSON, code |
| Search | SQLite LIKE query across content + labels |
| Filter tabs | All / Favorites / Text / Links / Images / Code |
| Pin clips | Pinned items stay at top and survive "Clear History" |
| Star clips | Favorites filter |
| Labels | Custom tag any clip |
| Swipe to delete | Left swipe with Snackbar undo |
| Long press menu | Copy / Pin / Star / Label / View / Delete |
| Auto-trim | Configurable max history (100–1000 or unlimited) |
| Dark theme | Full dark theme with indigo accent |
| Boot start | Restarts service after phone reboot |
| Image support | Captures clipboard images, saves to app storage |

---

## 📋 Permissions Explained

| Permission | Why Needed |
|-----------|-----------|
| `FOREGROUND_SERVICE` | To run clipboard monitor persistently |
| `RECEIVE_BOOT_COMPLETED` | Auto-start after reboot |
| `POST_NOTIFICATIONS` | Service notification (Android 13+) |
| `READ_MEDIA_IMAGES` | Capture image clipboard items |
| `VIBRATE` | Haptic feedback on copy |
| Accessibility Service | Background clipboard capture on Android 10+ |

---

## 🛠️ Customization Options

### Change max history limit
In `SettingsActivity` → Max History Items (default: 500)

### Change poll interval
In `ClipboardMonitorService.kt`:
```kotlin
const val POLL_INTERVAL_MS = 500L  // Change to 1000L for less battery usage
```

### Add new clip type detection
In `utils/ClipTypeDetector.kt` — add a new regex and `ClipType` enum value.

### Change theme colors
In `res/values/colors.xml` — all colors are centralized.

---

## 🐛 Troubleshooting

**Clips not being captured in background?**
→ Enable Accessibility Service in Settings → Accessibility → ClipVault

**Service keeps stopping?**
→ Go to phone's Battery settings → find ClipVault → disable battery optimization

**"Install blocked" on phone?**
→ Settings → Security → Enable "Install from unknown sources" or "Install unknown apps"

**Build fails with "SDK not found"?**
→ File → Project Structure → SDK Location → set correct path

**Gradle sync fails?**
→ Check internet connection. Try: File → Invalidate Caches → Restart

---

## 📦 Building for Play Store

1. Create a signed release APK (Step 4 above)
2. Create a Google Play Developer account ($25 one-time fee)
3. Upload APK to Play Console
4. Fill in store listing, screenshots, privacy policy
5. Publish!

**Note:** For Play Store, the Accessibility Service declaration requires extra review.
Prepare a justification: "Used solely to read clipboard content for the user's own
clipboard manager functionality. No data is transmitted externally."

---

## 🔒 Privacy Notes

- All data stored **locally on device only**
- No network requests, no telemetry, no ads
- Clipboard images saved to app's private storage (`/data/data/com.clipvault/files/clip_images/`)
- Uninstalling the app deletes all data
- Accessibility Service reads **only** clipboard — not screen content

---

## 📁 File Structure Reference

```
ClipVault/
├── build.gradle                    # Root Gradle config
├── settings.gradle                 # Module config
└── app/
    ├── build.gradle                # App dependencies
    └── src/main/
        ├── AndroidManifest.xml     # Permissions & components
        ├── java/com/clipvault/
        │   ├── ClipVaultApp.kt     # Application class
        │   ├── data/               # Database, models, repository
        │   ├── service/            # Background services
        │   ├── ui/                 # Activities & ViewModels
        │   └── utils/              # Type detection
        └── res/
            ├── layout/             # XML layouts
            ├── values/             # Colors, strings, themes
            ├── xml/                # Accessibility config, preferences
            └── drawable/           # Icons
```
