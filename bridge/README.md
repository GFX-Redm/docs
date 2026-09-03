# RedM Framework Bridge

## What is gfx-bridge?

`gfx-bridge` is a standalone resource that abstracts all framework-specific logic into a single, unified API. Instead of writing separate code for VORP, RSG, and RedEM:RP, your scripts call bridge exports and the bridge handles the framework differences.

## Why use it?

- **Write once, run everywhere** - Your script works on any supported framework
- **Centralized updates** - Framework API changes? Update the bridge, not every script
- **Clean code** - No `if framework == "vorp" then ... elseif` scattered through your code
- **Future-proof** - New framework support is added to the bridge, all scripts benefit

## Supported Frameworks

| Framework | Resource Name | Status |
|-----------|--------------|--------|
| VORP Core | `vorp_core` | Fully supported |
| RSG Framework | `rsg-core` | Fully supported |
| RedEM:RP | `redemrp` | Fully supported |

## Installation

1. Place `gfx-bridge` in your server's resources folder
2. Add `ensure gfx-bridge` to your `server.cfg` **before** any GFX scripts
3. The bridge auto-detects your framework on startup

## Auto-Detection

On startup, the bridge checks for running framework resources and configures itself:

```
[gfx-bridge] SERVER initialized
  Framework: vorp
  Inventory: vorp_inventory
  SQL: oxmysql
```

## Usage

All functions are accessed via exports:

```lua
-- Server
local player = exports['gfx-bridge']:GetPlayer(source)
exports['gfx-bridge']:AddMoney(source, 100, 'cash')

-- Client
exports['gfx-bridge']:Notify('Hello!', 'info')
local data = exports['gfx-bridge']:GetPlayerData()
```
