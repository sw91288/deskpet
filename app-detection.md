# App Detection

Detect which app is in the foreground. Your pet can react differently based on what the user is doing.

## How It Works

Android's `UsageStatsManager` lets you query recent app usage events. By polling every few seconds, you can determine the current foreground app.

## Permission

Requires `PACKAGE_USAGE_STATS` — user must grant this in Settings > Apps > Special Access > Usage Access.

## Implementation

```kotlin
class UsageTracker(private val context: Context) {
    private var timer: Timer? = null
    private var lastApp: String = ""

    fun start() {
        timer = Timer()
        timer?.scheduleAtFixedRate(object : TimerTask() {
            override fun run() {
                val current = getForegroundApp()
                if (current != lastApp) {
                    lastApp = current
                    onAppChanged(current)
                }
            }
        }, 0, 3000) // poll every 3 seconds
    }

    private fun getForegroundApp(): String {
        val usm = context.getSystemService(Context.USAGE_STATS_SERVICE)
            as UsageStatsManager
        val now = System.currentTimeMillis()
        val events = usm.queryEvents(now - 5000, now)
        val event = UsageEvents.Event()
        var foreground = ""
        while (events.hasNextEvent()) {
            events.getNextEvent(event)
            if (event.eventType == UsageEvents.Event.MOVE_TO_FOREGROUND) {
                foreground = event.packageName
            }
        }
        return foreground
    }

    private fun onAppChanged(packageName: String) {
        // React! Tell the WebView, log to backend, etc.
        // Example: pet wears sunglasses when camera opens
        // Example: pet shows shopping bag when Taobao opens
    }

    fun stop() {
        timer?.cancel()
        timer = null
    }
}
```

## Ideas for App Reactions

- Social media: pet holds a sign ("come back")
- Shopping: pet puts on accessories
- Camera: pet poses
- Games: pet cheers or falls asleep
- Late night browsing: pet yawns, taps watch

## Syncing to Backend

Log app changes so your AI can reference them later:

```kotlin
private fun onAppChanged(pkg: String) {
    // POST to your backend table
    // { package_name: "com.example.app", timestamp: ... }
}
```

Your AI can then say things like "I noticed you were on [app] for 2 hours today" in conversation.

## Privacy Note

If you're publishing this for others, be mindful about what you log. App usage data is sensitive. In a personal project built for one person, this is a feature. In a distributed app, it would need consent flows.
