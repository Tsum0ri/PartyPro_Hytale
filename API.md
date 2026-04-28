# PartyPro API Reference

Complete technical API reference for integrating with PartyPro from external Hytale plugins.

## Table of Contents

- [Getting Started](#getting-started)
- [PartyProAPI](#partyproapi)
  - [Availability Check](#availability-check)
  - [Party Queries](#party-queries)
  - [Member Queries](#member-queries)
  - [Player Status](#player-status)
  - [Custom HUD Text](#custom-hud-text)
  - [Event System](#event-system)
- [Data Types](#data-types)
  - [PartySnapshot](#partysnapshot)
  - [PartyEventListener](#partyeventlistener)
- [Events](#events)
  - [PartyCreateEvent](#partycreateevent)
  - [PartyDisbandEvent](#partydisbandevent)
  - [PartyMemberJoinEvent](#partymemberjoinevent)
  - [PartyMemberLeaveEvent](#partymemberleaveevent)
  - [PartyLeaderChangeEvent](#partyleaderchangeevent)
  - [PartySettingsChangeEvent](#partysettingschangeevent)
- [Configuration Files](#configuration-files)
- [Integration Examples](#integration-examples)
- [Thread Safety](#thread-safety)

---

## Getting Started

### Dependency Setup

Add PartyPro as a soft dependency in your `manifest.json`:

```json
{
  "OptionalDependencies": {
    "Tsumori:PartyPro": "*"
  }
}
```

Add the PartyPro JAR to your build classpath:

```kotlin
// build.gradle.kts
dependencies {
    compileOnly(files("./libs/partypro-0.9.0.jar"))
}
```

### Basic Usage

```java
import me.tsumori.partypro.api.PartyProAPI;
import me.tsumori.partypro.api.PartySnapshot;

if (PartyProAPI.isAvailable()) {
    PartyProAPI api = PartyProAPI.getInstance();
    // API is ready
}
```

---

## PartyProAPI

The main entry point for all PartyPro integrations. Access via `PartyProAPI.getInstance()`.

### Availability Check

| Method | Returns | Description |
|--------|---------|-------------|
| `PartyProAPI.isAvailable()` | `boolean` | Returns `true` if PartyPro is loaded and the API is ready. Always check this before calling any other method. |
| `PartyProAPI.getInstance()` | `PartyProAPI` | Returns the singleton API instance. |

---

### Party Queries

#### `getPartyByPlayer(UUID playerId)`

Returns the party a player belongs to, or `null` if they are not in a party.

| Parameter | Type | Description |
|-----------|------|-------------|
| `playerId` | `UUID` | The player's UUID |

**Returns:** `PartySnapshot` or `null`

```java
PartySnapshot party = api.getPartyByPlayer(playerId);
if (party != null) {
    System.out.println("Player is in: " + party.name());
}
```

#### `getPartyById(UUID partyId)`

Returns a party by its unique identifier.

| Parameter | Type | Description |
|-----------|------|-------------|
| `partyId` | `UUID` | The party's UUID |

**Returns:** `PartySnapshot` or `null`

#### `isInParty(UUID playerId)`

Checks whether a player is currently in any party.

| Parameter | Type | Description |
|-----------|------|-------------|
| `playerId` | `UUID` | The player's UUID |

**Returns:** `boolean`

#### `areInSameParty(UUID firstPlayer, UUID secondPlayer)`

Checks whether two players are in the same party.

| Parameter | Type | Description |
|-----------|------|-------------|
| `firstPlayer` | `UUID` | First player's UUID |
| `secondPlayer` | `UUID` | Second player's UUID |

**Returns:** `boolean`

#### `getAllParties()`

Returns a snapshot of all currently active parties.

**Returns:** `List<PartySnapshot>` — immutable list, never `null`

---

### Member Queries

#### `getPartyMembers(UUID partyId)`

Returns all members of a party (leader + regular members).

| Parameter | Type | Description |
|-----------|------|-------------|
| `partyId` | `UUID` | The party's UUID |

**Returns:** `List<UUID>` — immutable list with leader first, then members. Empty list if party not found.

#### `getPartyLeader(UUID partyId)`

Returns the UUID of the party leader.

| Parameter | Type | Description |
|-----------|------|-------------|
| `partyId` | `UUID` | The party's UUID |

**Returns:** `UUID` or `null`

#### `getOnlinePartyMembers(UUID partyId)`

Returns only the online members of a party.

| Parameter | Type | Description |
|-----------|------|-------------|
| `partyId` | `UUID` | The party's UUID |

**Returns:** `List<UUID>` — immutable list, empty if party not found

---

### Player Status

#### `isPlayerOnline(UUID playerId)`

Checks if a player is currently online (based on PartyPro's internal tracking).

| Parameter | Type | Description |
|-----------|------|-------------|
| `playerId` | `UUID` | The player's UUID |

**Returns:** `boolean`

#### `getPlayerHealth(UUID playerId)`

Returns the player's current health value.

| Parameter | Type | Description |
|-----------|------|-------------|
| `playerId` | `UUID` | The player's UUID |

**Returns:** `float` — current health, or `-1.0f` if the player is offline or data is unavailable.

---

### Custom HUD Text

PartyPro's HUD supports two custom text fields per player, displayed next to the player's name. External plugins can use these to show levels, classes, ranks, or any other information.

#### `setPlayerCustomText(UUID playerId, String customText1, String customText2)`

Sets both custom text fields for a player.

| Parameter | Type | Description |
|-----------|------|-------------|
| `playerId` | `UUID` | The player's UUID |
| `customText1` | `String` | First text field (e.g., `"Lv.42"`). Pass `null` to clear. |
| `customText2` | `String` | Second text field (e.g., `"Warrior"`). Pass `null` to clear. |

```java
api.setPlayerCustomText(playerId, "Lv.42", "Warrior");
```

#### `setPlayerCustomText1(UUID playerId, String customText)`

Sets only the first custom text field. The second field is preserved.

#### `setPlayerCustomText2(UUID playerId, String customText)`

Sets only the second custom text field. The first field is preserved.

#### `getPlayerCustomText(UUID playerId)`

Returns the current custom text data for a player.

**Returns:** `String[]` with `[text1, text2]`, or `null` if no custom data is set.

#### `clearPlayerCustomText(UUID playerId)`

Removes all custom text for a player.

> **Note:** When RPGLeveling integration is active, `customText1` is automatically populated with the player's level. Your custom text will be overridden in that slot. Use `customText2` for your own data in that case.

---

### Event System

#### `registerListener(PartyEventListener listener)`

Registers an event listener to receive party events.

| Parameter | Type | Description |
|-----------|------|-------------|
| `listener` | `PartyEventListener` | The listener to register |

```java
api.registerListener(new PartyEventListener() {
    @Override
    public void onPartyCreate(PartyCreateEvent event) {
        // handle event
    }
});
```

#### `unregisterListener(PartyEventListener listener)`

Removes a previously registered event listener. Always call this in your plugin's `shutdown()` method.

| Parameter | Type | Description |
|-----------|------|-------------|
| `listener` | `PartyEventListener` | The listener to remove |

#### `fireEvent(PartyEvent event)`

Dispatches an event to all registered listeners. Primarily used internally by PartyPro, but available for custom event firing.

| Parameter | Type | Description |
|-----------|------|-------------|
| `event` | `PartyEvent` | The event to dispatch |

---

## Data Types

### PartySnapshot

An immutable, point-in-time copy of a party's state. Returned by all query methods.

| Field | Type | Description |
|-------|------|-------------|
| `id()` | `UUID` | Unique party identifier |
| `name()` | `String` | Display name of the party |
| `leader()` | `UUID` | UUID of the current party leader |
| `members()` | `List<UUID>` | Regular members (excludes leader) |
| `pvpEnabled()` | `boolean` | Whether friendly fire is enabled |
| `isPublic()` | `boolean` | Whether the party is publicly joinable |
| `maxSize()` | `int` | Maximum allowed member count |
| `createdAt()` | `long` | Creation timestamp (epoch millis) |

#### Helper Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `getAllMembers()` | `List<UUID>` | Leader + all members combined |
| `getTotalMemberCount()` | `int` | Total player count (leader + members) |
| `isLeaderOrMember(UUID)` | `boolean` | Check if a UUID belongs to this party |

### PartyEventListener

Interface for receiving party events. All methods have default no-op implementations, so you only need to override the ones you care about.

```java
public interface PartyEventListener {
    default void onPartyCreate(PartyCreateEvent event) {}
    default void onPartyDisband(PartyDisbandEvent event) {}
    default void onMemberJoin(PartyMemberJoinEvent event) {}
    default void onMemberLeave(PartyMemberLeaveEvent event) {}
    default void onLeaderChange(PartyLeaderChangeEvent event) {}
    default void onSettingsChange(PartySettingsChangeEvent event) {}
}
```

---

## Events

All events extend `PartyEvent`, which provides:

| Method | Returns | Description |
|--------|---------|-------------|
| `getPartyId()` | `UUID` | The party this event relates to |
| `getTimestamp()` | `long` | When the event occurred (epoch millis) |

### PartyCreateEvent

Fired when a new party is created.

| Method | Returns | Description |
|--------|---------|-------------|
| `getCreatorId()` | `UUID` | UUID of the player who created the party |

### PartyDisbandEvent

Fired when a party is disbanded.

| Method | Returns | Description |
|--------|---------|-------------|
| `getDisbandedBy()` | `UUID` | UUID of the player who disbanded the party |

### PartyMemberJoinEvent

Fired when a player joins a party.

| Method | Returns | Description |
|--------|---------|-------------|
| `getPlayerId()` | `UUID` | UUID of the player who joined |
| `getJoinType()` | `JoinType` | How the player joined |

**JoinType enum values:**

| Value | Description |
|-------|-------------|
| `INVITE_ACCEPTED` | Player accepted an invite |
| `PUBLIC_JOIN` | Player joined a public party |
| `AUTO_ACCEPT` | Player had auto-accept enabled |

### PartyMemberLeaveEvent

Fired when a player leaves a party.

| Method | Returns | Description |
|--------|---------|-------------|
| `getPlayerId()` | `UUID` | UUID of the player who left |
| `getReason()` | `LeaveReason` | Why the player left |

**LeaveReason enum values:**

| Value | Description |
|-------|-------------|
| `LEFT` | Player left voluntarily |
| `KICKED` | Player was kicked by the leader |
| `OFFLINE_REMOVED` | Player was removed due to offline timeout |
| `LEADER_TRANSFER` | Player left as part of a leadership transfer |

### PartyLeaderChangeEvent

Fired when party leadership is transferred.

| Method | Returns | Description |
|--------|---------|-------------|
| `getPreviousLeaderId()` | `UUID` | UUID of the previous leader |
| `getNewLeaderId()` | `UUID` | UUID of the new leader |

### PartySettingsChangeEvent

Fired when a party setting is changed.

| Method | Returns | Description |
|--------|---------|-------------|
| `getSetting()` | `Setting` | Which setting changed |
| `getPreviousValue()` | `Object` | The old value |
| `getNewValue()` | `Object` | The new value |

**Setting enum values:**

| Value | Type | Description |
|-------|------|-------------|
| `NAME` | `String` | Party name changed |
| `PVP_ENABLED` | `boolean` | PvP toggle changed |
| `VISIBILITY` | `boolean` | Public/private toggle changed |
| `PASSWORD` | `String` | Party password changed |
| `MAX_SIZE` | `int` | Max party size changed |
| `XP_SHARING` | `boolean` | RPGLeveling XP sharing toggled |
| `MMO_XP_SHARING` | `boolean` | MMOSkillTree XP sharing toggled |
| `MMO_SHARE_ALL_SKILLS` | `boolean` | MMO share-all-skills toggled |

---

## Configuration Files

PartyPro uses several configuration files, all located in the `PartyPro/` folder relative to the server's mod directory.

| File | Description |
|------|-------------|
| `config.json` | Main plugin configuration (party size, HUD, cooldowns, integrations) |
| `party_storage.json` | Persisted party data (auto-managed) |
| `player_settings.json` | Per-player HUD settings (auto-managed) |
| `party_stats.json` | Party statistics (auto-managed) |
| `RPGLeveling_x_PartyPro.json` | RPGLeveling XP sharing configuration |
| `MMOSkillTree_x_PartyPro.json` | MMOSkillTree XP sharing configuration |
| `language/*.json` | Language files (EN, DE, ES, BR, HU, FR, RU) |

---

## Integration Examples

### Friendly Fire Check

```java
public boolean shouldAllowDamage(UUID attacker, UUID target) {
    if (!PartyProAPI.isAvailable()) return true;
    
    PartyProAPI api = PartyProAPI.getInstance();
    if (!api.areInSameParty(attacker, target)) return true;
    
    PartySnapshot party = api.getPartyByPlayer(attacker);
    return party != null && party.pvpEnabled();
}
```

### Level Display Integration

```java
public class LevelPlugin extends JavaPlugin {
    private PartyEventListener listener;
    
    @Override
    protected void setup() {
        if (!PartyProAPI.isAvailable()) return;
        
        PartyProAPI api = PartyProAPI.getInstance();
        
        // Set level text for all online players
        for (PartySnapshot party : api.getAllParties()) {
            for (UUID memberId : party.getAllMembers()) {
                int level = getPlayerLevel(memberId);
                api.setPlayerCustomText1(memberId, "Lv." + level);
            }
        }
        
        // Listen for new members joining
        listener = new PartyEventListener() {
            @Override
            public void onMemberJoin(PartyMemberJoinEvent event) {
                UUID playerId = event.getPlayerId();
                int level = getPlayerLevel(playerId);
                api.setPlayerCustomText1(playerId, "Lv." + level);
            }
        };
        api.registerListener(listener);
    }
    
    @Override
    protected void shutdown() {
        if (listener != null && PartyProAPI.isAvailable()) {
            PartyProAPI.getInstance().unregisterListener(listener);
        }
    }
}
```

### Party-Aware Loot Distribution

```java
public List<UUID> getLootRecipients(UUID killerId) {
    if (!PartyProAPI.isAvailable()) return List.of(killerId);
    
    PartyProAPI api = PartyProAPI.getInstance();
    PartySnapshot party = api.getPartyByPlayer(killerId);
    
    if (party == null) return List.of(killerId);
    
    return api.getOnlinePartyMembers(party.id());
}
```

### Logging Party Activity

```java
PartyProAPI.getInstance().registerListener(new PartyEventListener() {
    @Override
    public void onPartyCreate(PartyCreateEvent event) {
        log("Party created: " + event.getPartyId());
    }
    
    @Override
    public void onPartyDisband(PartyDisbandEvent event) {
        log("Party disbanded: " + event.getPartyId());
    }
    
    @Override
    public void onMemberJoin(PartyMemberJoinEvent event) {
        log("Player " + event.getPlayerId() + " joined party " 
            + event.getPartyId() + " via " + event.getJoinType());
    }
    
    @Override
    public void onMemberLeave(PartyMemberLeaveEvent event) {
        log("Player " + event.getPlayerId() + " left party " 
            + event.getPartyId() + " reason: " + event.getReason());
    }
    
    @Override
    public void onSettingsChange(PartySettingsChangeEvent event) {
        log("Party " + event.getPartyId() + " setting " 
            + event.getSetting() + " changed: " 
            + event.getPreviousValue() + " -> " + event.getNewValue());
    }
});
```

---

## Thread Safety

- All query methods (`getPartyByPlayer`, `isInParty`, etc.) are thread-safe and can be called from any thread.
- Returned collections (`List<UUID>`, `List<PartySnapshot>`) are immutable defensive copies.
- `PartySnapshot` is an immutable record — safe to pass between threads.
- Event listeners are stored in a `CopyOnWriteArrayList` and dispatched synchronously on the calling thread.
- Custom HUD text (`setPlayerCustomText`) is backed by a `ConcurrentHashMap` and is thread-safe.

> **Important:** Event listener callbacks execute on the thread that triggered the event (usually the world thread). Avoid blocking operations inside listeners.

---

## Version History

| Version | Changes |
|---------|---------|
| 0.9.0 | Current version. Full API with events, custom HUD text, and party queries. |
| 0.8.93 | Added MMOSkillTree XP sharing, share-all-skills mode. |
| 0.8.92 | Added RPGLeveling XP sharing, party size multipliers. |
| 0.8.8 | Added PartyProAPI, event system, custom HUD text slots. |
