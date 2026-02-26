# Server API

All server-side bridge functions are accessed via exports.

## Player Functions

### GetPlayer
```lua
local player = exports['gfx-bridge']:GetPlayer(source)
```
Returns the raw framework player object.

### GetIdentifier
```lua
local id = exports['gfx-bridge']:GetIdentifier(source)
```
Returns the player's unique identifier (citizenid, charid, or license).

### GetPlayerName
```lua
local name = exports['gfx-bridge']:GetPlayerName(source)
```
Returns the character's full name (firstname + lastname).

## Money Functions

### AddMoney
```lua
exports['gfx-bridge']:AddMoney(source, amount, type)
```
| Parameter | Type | Values |
|-----------|------|--------|
| source | number | Player server ID |
| amount | number | Amount to add |
| type | string | "cash", "gold", "bank", "rol" |

### RemoveMoney
```lua
exports['gfx-bridge']:RemoveMoney(source, amount, type)
```
Same parameters as AddMoney.

### HasMoney
```lua
local has = exports['gfx-bridge']:HasMoney(source, amount, type)
```
Returns boolean.

### GetMoney
```lua
local amount = exports['gfx-bridge']:GetMoney(source, type)
```
Returns number.

## Inventory Functions

### AddItem
```lua
exports['gfx-bridge']:AddItem(source, item, count, meta)
```
| Parameter | Type | Description |
|-----------|------|-------------|
| source | number | Player server ID |
| item | string | Item name |
| count | number | Amount |
| meta | table\|nil | Item metadata |

### RemoveItem
```lua
exports['gfx-bridge']:RemoveItem(source, item, count)
```

### HasItem
```lua
local has = exports['gfx-bridge']:HasItem(source, item, count)
```
Returns boolean.

### GetItemCount
```lua
local count = exports['gfx-bridge']:GetItemCount(source, item)
```
Returns number.

### GetInventory
```lua
local items = exports['gfx-bridge']:GetInventory(source)
```
Returns table of inventory items.

### RegisterItem
```lua
exports['gfx-bridge']:RegisterItem(item, function(source, data)
    -- Handle item use
end)
```

## Callback Functions

### RegisterCallback
```lua
exports['gfx-bridge']:RegisterCallback(name, function(source, ...)
    return result
end)
```

## Notification

### Notify
```lua
exports['gfx-bridge']:Notify(source, message, type)
```

## Database

### ExecuteSql
```lua
local result = exports['gfx-bridge']:ExecuteSql(query, params)
```
Returns query results. Supports oxmysql, ghmattimysql, mysql-async.

## Job Functions

### GetPlayerJob
```lua
local job = exports['gfx-bridge']:GetPlayerJob(source)
```
Returns `{ name, label, grade }` or nil.

### SetPlayerJob
```lua
exports['gfx-bridge']:SetPlayerJob(source, job, grade)
```
