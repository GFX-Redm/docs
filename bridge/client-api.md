# Client API

All client-side bridge functions are accessed via exports.

## Notify

Send a notification to the local player.

```lua
exports['gfx-bridge']:Notify(message, type)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| message | string | Notification text |
| type | string\|nil | "success", "error", "info" |

## GetPlayerData

Get normalized player data.

```lua
local data = exports['gfx-bridge']:GetPlayerData()
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
| group | string | Admin group |

## TriggerCallback

Trigger a server callback and await response.

```lua
local result = exports['gfx-bridge']:TriggerCallback(name, ...)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Callback name |
| ... | any | Arguments to pass |

**Returns:** any (callback result)

## GetPlayerJob

Get player job information.

```lua
local job = exports['gfx-bridge']:GetPlayerJob()
```

**Returns:** table or nil `{ name, label, grade }`

## HasJob

Check if player has a specific job.

```lua
local isSheriff = exports['gfx-bridge']:HasJob('sheriff')
```

| Parameter | Type | Description |
|-----------|------|-------------|
| jobName | string | Job name to check |

**Returns:** boolean
