# RedM Bridge Shared API

Shared functions are available on both client and server.

## Bridge Properties

### Bridge.FrameworkName
```lua
Bridge.FrameworkName -- "vorp" | "rsg" | "redem" | nil
```
The detected framework name.

### Bridge.InventoryName
```lua
Bridge.InventoryName -- "vorp_inventory" | "rsg-inventory" | "redemrp_inventory" | nil
```
The detected inventory resource name.

### Bridge.SQLName
```lua
Bridge.SQLName -- "oxmysql" | "ghmattimysql" | "mysql-async" | nil
```
The detected SQL resource name.

## Bridge.Init()

Called automatically on resource start. Detects framework, inventory, and SQL resources. Prints detection results to console.

## Bridge.GetFrameworkObject()

Returns the raw framework core object:
- VORP: `exports.vorp_core:GetCore()`
- RSG: `exports['rsg-core']:GetCoreObject()`
- RedEM: `exports.redemrp:getCore()`

## Bridge.TableSize(t)

Returns the number of entries in a table (works with non-sequential keys).

```lua
local count = Bridge.TableSize(myTable)
```

## Framework Detection Logic

The bridge checks resources in order:
1. `vorp_core` → Framework = "vorp"
2. `rsg-core` → Framework = "rsg"
3. `redemrp` → Framework = "redem"

First match wins. Same pattern for inventory and SQL detection.
