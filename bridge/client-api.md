# Client API

All client-side bridge functions are accessed via exports.

## Notify

Send a notification to the local player.

```lua
exports['gfxr-bridge']:Notify(message, type)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| message | string | Notification text |
| type | string\|nil | "success", "error", "info" |

## GetPlayerData

Get normalized player data.

```lua
local data = exports['gfxr-bridge']:GetPlayerData()
```

**Returns:** table or nil

| Field | Type | Description |
|-------|------|-------------|
| identifier | string | Player unique ID |
| name | string | Full character name |
| firstname | string | First name |
| lastname | string | Last name |
| job | string | Job name |
| jobLabel | string | Job display name |
| jobGrade | number | Job grade level |
| money | number | Cash amount |
| gold | number | Gold amount |
| bank | number | Bank balance (RSG/RedEM; VORP has no core bank — returns `nil`) |
| rol | number | ROL currency (VORP only; `nil` on other frameworks) |
| group | string | Admin group |

## OnPlayerLoaded

Register a callback that fires when the player's character is fully loaded. If the player is already loaded when you call this, the callback fires immediately.

```lua
exports['gfxr-bridge']:OnPlayerLoaded(function()
  -- safe to read player data here
end)
```

Fires on: VORP `vorp:SelectedCharacter`, RSG `RSGCore:Client:OnPlayerLoaded`, RedEM `redemrp:playerLoaded`, and a 2 s delayed `playerSpawned` fallback for other frameworks.

## OnMoneyChange

Register a callback that fires whenever the local player's money changes — event-driven, so feature scripts never poll for money.

```lua
exports['gfxr-bridge']:OnMoneyChange(function(money)
  -- money = { cash, gold, bank }
end)
```

The callback also fires once immediately with the current snapshot, so you get an initial value on register. Internally: **RSG** uses its native `RSGCore:Client:OnMoneyChange` event; **VORP/RedEM** have no such event, so the bridge diffs the local (client-cached) character object on a low-rate loop — never a server round-trip. `bank` is `0` where the framework has no bank (VORP).

## OnNeedsChange

Register a callback that fires whenever the local player's needs change — event-driven, so HUD/metabolism scripts never poll the server for needs.

```lua
exports['gfxr-bridge']:OnNeedsChange(function(needs)
  -- needs = { hunger, thirst, stress }  (0-100)
end)
```

Fires once immediately with an initial snapshot where available. Internally (all verified against framework source):
- **RSG** — reads hunger/thirst/stress from local `metadata`; updates on `RSGCore:Player:SetPlayerData` (which re-fires on every metadata change — `RSGCore:Client:OnPlayerMetadata` does **not** exist). No server call.
- **VORP** — `vorp_metabolism` drains client-side silently with no per-tick event, so the bridge reads the current `Hunger`/`Thirst` (0–1000 → /10) via the local `vorpmetabolism:getValue` event on a low-rate (2s) **client-local** loop (no server round-trip), plus instant refresh on `setValue`/`changeValue`/`useItem`. No stress → `0`.
- **RedEM:RP** — exposes no client needs signal, so the bridge does a low-rate (3s) **server** poll internally. The only framework that hits the server for needs.

## TriggerCallback

Trigger a server callback and await response.

```lua
local result = exports['gfxr-bridge']:TriggerCallback(name, ...)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Callback name |
| ... | any | Arguments to pass |

**Returns:** any (callback result)

## GetPlayerJob

Get player job information.

```lua
local job = exports['gfxr-bridge']:GetPlayerJob()
```

**Returns:** table or nil `{ name, label, grade }`

## HasJob

Check if player has a specific job.

```lua
local isSheriff = exports['gfxr-bridge']:HasJob('sheriff')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| jobName | string | Job name to check |

**Returns:** boolean
