# Overlay Service

The core of the entire app. A foreground service that creates a floating WebView window.

## Key Concepts

1. **Foreground Service**: Android kills background services aggressively. You need a persistent notification to stay alive.
2. **WindowManager**: The system service that lets you add views on top of everything.
3. **WebView**: Your pet's rendering engine. HTML/CSS/JS gives you full animation control without recompiling.

## Minimal Structure

```kotlin
class OverlayService : Service() {
    private var windowManager: WindowManager? = null
    private var overlayView: WebView? = null

    override fun onCreate() {
        super.onCreate()
        startForeground(NOTIFICATION_ID, buildNotification())
        setupOverlay()
    }

    private fun setupOverlay() {
        windowManager = getSystemService(WINDOW_SERVICE) as WindowManager

        val params = WindowManager.LayoutParams(
            dpToPx(180),  // width
            dpToPx(240),  // height
            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY,
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE or
            WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS,
            PixelFormat.TRANSLUCENT
        ).apply {
            gravity = Gravity.TOP or Gravity.START
            x = 50
            y = 300
        }

        overlayView = WebView(this).apply {
            setBackgroundColor(0x00000000) // transparent
            settings.javaScriptEnabled = true
            settings.domStorageEnabled = true
            loadUrl("file:///android_asset/pet.html")
        }

        windowManager?.addView(overlayView, params)
    }

    override fun onDestroy() {
        overlayView?.let {
            windowManager?.removeView(it)
            it.destroy()
        }
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}
```

## Important Notes

- `TYPE_APPLICATION_OVERLAY` requires `SYSTEM_ALERT_WINDOW` permission (user must grant manually in settings)
- `FLAG_NOT_FOCUSABLE` ensures the overlay doesn't steal input from other apps
- `FLAG_LAYOUT_NO_LIMITS` lets the pet be dragged partially off-screen
- WebView background must be transparent (`0x00000000`) or you'll get a white box
- On some devices (Huawei, Xiaomi) you need additional manufacturer-specific permissions

## WebView Communication

Kotlin → JS:
```kotlin
overlayView?.evaluateJavascript(
    "window.petEngine && window.petEngine.onTap()", null
)
```

Your HTML file exposes a global engine object that responds to events from the native layer.

## Notification Channel

Android 8+ requires a notification channel. Use `IMPORTANCE_LOW` to avoid sound:

```kotlin
val channel = NotificationChannel(
    CHANNEL_ID,
    "Your Pet",
    NotificationManager.IMPORTANCE_LOW
).apply {
    setShowBadge(false)
}
```

## Device Compatibility

| Issue | Affected Devices | Workaround |
|-------|-----------------|-------------|
| Overlay killed in background | Huawei, Xiaomi, Oppo | Battery optimization whitelist |
| Permission not showing | Some Xiaomi | Manual grant in app settings |
| WebView rendering glitch | Android 8.0 | Set hardware acceleration on WebView |

## Next Steps

- Add touch handling → [gesture-system.md](gesture-system.md)
- Add notification rotation → [notification-whispers.md](notification-whispers.md)
