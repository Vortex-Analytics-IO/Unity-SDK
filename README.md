# AnalyticsManager Documentation

`AnalyticsManager` is a Unity component responsible for sending analytics events to a remote server.  
It supports:

- Immediate event tracking  
- Batched event tracking  
- Automatic retry when the server becomes available  
- Session and device identification  
- Manual or automatic initialization

## Dependencies

`AnalyticsManager` requires **Newtonsoft.Json** (Json.NET) for JSON serialization, which is essential for supporting dictionaries and complex data structures.

### Installation in Your Unity Project

#### Option 1: Unity Package Manager (Recommended)

1. Open your project in Unity
2. Go to **Window** → **General** → **Package Manager**
3. Click the **+** button and select **Add package by name**
4. Enter: `com.unity.nuget.newtonsoft-json`
5. Click **Add** and wait for the installation to complete

Alternatively, you can search for "Newtonsoft Json" directly in the Package Manager and install it from there.

#### Option 2: Manual Installation (DLL)

1. Download Newtonsoft.Json DLLs from [NuGet](https://www.nuget.org/packages/Newtonsoft.Json/)
2. Extract the DLL files
3. Create a folder `Assets/Plugins` in your project (if it doesn't exist)
4. Copy the DLL files into `Assets/Plugins`
5. Restart Unity

#### Option 3: NuGetForUnity

If you have [NuGetForUnity](https://github.com/GlassToeStudio/NuGetForUnity) installed:

1. Go to **NuGet** → **Manage NuGet Packages**
2. Search for **Newtonsoft.Json**
3. Click **Install** on the latest version
4. Unity will automatically download and configure it

## Initialization

### Automatic Initialization (Recommended)

Attach the `AnalyticsManager` component to a GameObject and configure:

- **Tenant ID**
- **Server URL**
- **Platform**

When `Initialize On Start` is enabled, the system initializes automatically during `Awake()`.

### Manual Initialization

If you need to initialize analytics at runtime (e.g., after login):

```csharp
AnalyticsManager.Instance.Init(
    tenantId: "mygame",
    url: "https://analytics.myserver.com",
    platform: "STEAM"
);
```

⚠️ This must be called before sending any events.

### Internal Behavior

On initialization, the system:
1. Generates or loads a persistent device identifier
2. Creates a new session ID
3. Performs a server health check
4. Enables or disables analytics based on server availability

If the server is unreachable, events are safely queued until connectivity is restored.

## Tracking Events

### Simple Event

```csharp
AnalyticsManager.Instance.TrackEvent("app_started");
```

### Event with String Payload

```csharp
AnalyticsManager.Instance.TrackEvent("menu_opened", "settings");
```

### Event with Structured Data

```csharp
AnalyticsManager.Instance.TrackEvent("level_completed", new Dictionary<string, object>
{
    { "level", 5 },
    { "difficulty", "Hard" },
    { "time", 123.4f }
});
```

### Manual Batching
Manual batching allows you to explicitly control when analytics events are sent.

#### Add Events to Batch

```csharp
AnalyticsManager.Instance.BatchedTrackEvent("EnemyKilled");

AnalyticsManager.Instance.BatchedTrackEvent(
    "ItemCrafted",
    new Dictionary<string, object>
    {
        { "item", "MagicSword" },
        { "rarity", "Epic" }
    }
);
```

#### Send Batched Events

```csharp
AnalyticsManager.Instance.FlushManualBatch();
```

All queued events will be sent in a single request.

## Automatic Batching

When Auto Batching is enabled:

- Events are queued automatically
- The system sends batches every Auto Flush Interval seconds

If the server is unreachable, events remain queued.

## Custom Data

Custom data allows you to attach a JSON object to **every** analytics event sent by the manager.

### Setting Custom Data

```csharp
AnalyticsManager.Instance.SetCustomData(new Dictionary<string, object>
{
    { "region", "EU" },
    { "premium", true },
    { "user_level", 10 }
});
```

The custom data is automatically included in all subsequent events (both immediate and batched).

### Clearing Custom Data

To remove the custom data:

```csharp
AnalyticsManager.Instance.ClearCustomData();
```

### Behavior

- **Empty custom data** is not sent in requests (to minimize payload size)
- **Non-empty custom data** is included in every tracking event
- Changing custom data affects only **new** events; previously sent events are not modified

##  Lifecycle Handling

Analytics are flushed automatically when:
- The application loses focus (mobile background)
- The application is quitting

This ensures minimal data loss.