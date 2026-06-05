# Server API

All server-side bridge functions are accessed via exports.

## Player Functions

### GetPlayer
```lua
local player = exports['gfxr-bridge']:GetPlayer(source)
```
Returns the raw framework player object.

### GetIdentifier
```lua
local id = exports['gfxr-bridge']:GetIdentifier(source)
```
Returns the player's unique identifier (citizenid, charid, or license).

### GetPlayerName
```lua
local name = exports['gfxr-bridge']:GetPlayerName(source)
```
Returns the character's full name (firstname + lastname).

## Money Functions

### AddMoney
```lua
exports['gfxr-bridge']:AddMoney(source, amount, type)
```
| Parameter | Type | Values |
|-----------|------|--------|
| source | number | Player server ID |
| amount | number | Amount to add |
| type | string | "cash", "gold", "bank", "rol" |

### RemoveMoney
```lua
exports['gfxr-bridge']:RemoveMoney(source, amount, type)
```
Same parameters as AddMoney.

### HasMoney
```lua
local has = exports['gfxr-bridge']:HasMoney(source, amount, type)
```
Returns boolean.

### GetMoney
```lua
local amount = exports['gfxr-bridge']:GetMoney(source, type)
```
Returns number.

### GetBank
```lua
local bank = exports['gfxr-bridge']:GetBank(source)
```
Returns the player's bank balance as a number. **RSG** (`PlayerData.money.bank`) and **RedEM:RP** (`bankmoney`) expose a native bank balance. **VORP** has no standardized core bank balance — banking is a separate optional resource not tied to the character object — so VORP returns `0`. Scripts needing VORP banking must query that resource directly.

## Inventory Functions

### AddItem
```lua
exports['gfxr-bridge']:AddItem(source, item, count, meta)
```
| Parameter | Type | Description |
|-----------|------|-------------|
| source | number | Player server ID |
| item | string | Item name |
| count | number | Amount |
| meta | table\|nil | Item metadata |

### RemoveItem
```lua
exports['gfxr-bridge']:RemoveItem(source, item, count)
```

### HasItem
```lua
local has = exports['gfxr-bridge']:HasItem(source, item, count)
```
Returns boolean.

### GetItemCount
```lua
local count = exports['gfxr-bridge']:GetItemCount(source, item)
```
Returns number.

### GetInventory
```lua
local items = exports['gfxr-bridge']:GetInventory(source)
```
Returns table of inventory items.

### RegisterItem
```lua
exports['gfxr-bridge']:RegisterItem(item, function(source, data)
    -- Handle item use
end)
```

## Callback Functions

### RegisterCallback
```lua
exports['gfxr-bridge']:RegisterCallback(name, function(source, ...)
    return result
end)
```

## Notification

### Notify
```lua
exports['gfxr-bridge']:Notify(source, message, type)
```

## Database

### ExecuteSql
```lua
local result = exports['gfxr-bridge']:ExecuteSql(query, params)
```
Returns query results. Supports oxmysql, ghmattimysql, mysql-async.

## Job Functions

### GetPlayerJob
```lua
local job = exports['gfxr-bridge']:GetPlayerJob(source)
```
Returns `{ name, label, grade }` or nil.

### SetPlayerJob
```lua
exports['gfxr-bridge']:SetPlayerJob(source, job, grade)
```

## Needs / Metabolism

### GetNeeds
```lua
local needs = exports['gfxr-bridge']:GetNeeds(source)
-- needs = { hunger = 0-100, thirst = 0-100, stress = 0-100 }
```
Returns the player's hunger, thirst, and stress as `0-100` integers, framework-agnostic (VORP / RSG / RedEM:RP auto-detected). Always returns a table (zeros if the player can't be resolved). Read-only; safe to call on a tick from server-side HUD/metabolism scripts.

Per-framework sourcing (so you know what to expect):

| Framework | Source | Native range | Notes |
|---|---|---|---|
| **RSG** | `PlayerData.metadata.hunger / thirst / stress` | 0–100 | Direct, reliable. Stress supported. |
| **VORP** | character `status` JSON written by **`vorp_metabolism`** (keys `Hunger`/`Thirst`/`Metabolism`) | 0–1000 → normalized to 0–100 | Requires `vorp_metabolism` (or a resource using the same `setStatus` channel). No stress concept → `stress = 0`. Returns zeros if metabolism never ran for the character. |
| **RedEM:RP** | best-effort read of `player.status.*` / `player.*` | varies | `redemrp_status` exposes no documented server-side getter; only an `AddHungerThirst` mutation event. May return zeros — verify on your build, or extend this branch if your status resource exposes a getter. |

> Sources cached under `.claude/refs/cache/` (`vorp-metabolism`, `rsg-playerdata`, `redem-status`).
