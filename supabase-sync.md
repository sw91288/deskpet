# Backend Sync (Supabase)

Connect your pet to a backend so your AI can read its state, and your AI can push state back.

## Why

- Your AI (in a chat app) can know what gestures happened, what apps were used
- Your AI can push state changes (mood, accessories, speech bubbles) to the pet
- Enables two-way communication: user ↔ pet ↔ AI ↔ user

## Suggested Tables

You'll need tables appropriate to your setup. Here's a starting schema:

### gesture_log
```sql
create table gesture_log (
    id bigserial primary key,
    gesture_type text not null,  -- tap, double_tap, long_press, fling
    x integer,
    y integer,
    created_at timestamptz default now()
);
```

### app_usage
```sql
create table app_usage (
    id bigserial primary key,
    package_name text not null,
    started_at timestamptz default now()
);
```

### pet_state
```sql
create table pet_state (
    id bigserial primary key,
    state_key text not null,     -- mood, accessory, speech_bubble, etc.
    state_value text,
    updated_at timestamptz default now()
);
```

## Posting from Android

```kotlin
private fun postToSupabase(table: String, body: JSONObject) {
    scope.launch {
        try {
            val url = URL("$SUPABASE_URL/rest/v1/$table")
            val conn = url.openConnection() as HttpURLConnection
            conn.requestMethod = "POST"
            conn.setRequestProperty("Content-Type", "application/json")
            conn.setRequestProperty("apikey", SUPABASE_KEY)
            conn.setRequestProperty("Authorization", "Bearer $SUPABASE_KEY")
            conn.setRequestProperty("Prefer", "return=minimal")
            conn.doOutput = true
            conn.outputStream.use { it.write(body.toString().toByteArray()) }
            conn.responseCode
            conn.disconnect()
        } catch (_: Exception) {}
    }
}
```

## Reading State (Pet polls for AI commands)

Your WebView JS can periodically fetch from Supabase to check if the AI pushed any state changes:

```javascript
async function pollState() {
    const res = await fetch(`${SUPABASE_URL}/rest/v1/pet_state?order=updated_at.desc&limit=1`, {
        headers: { 'apikey': SUPABASE_KEY }
    });
    const data = await res.json();
    // Apply state changes to pet
}
setInterval(pollState, 30000); // every 30s
```

## Security Notes

- Use Row Level Security (RLS) on your tables
- Consider using a service role key only server-side
- For a personal project, anon key with RLS is fine
- Never commit your keys to a public repo

## Alternatives to Supabase

Anything with a REST API works:
- Firebase Realtime Database
- PocketBase (self-hosted)
- Plain REST server
- Even a simple JSON file on a VPS

The point is: your pet and your AI need a shared data layer.
