# gfxr-kit

Kit claim system for RedM. Allows server administrators to create item kits that players can claim on cooldown timers, with optional Discord role restrictions.

## Features

- Configurable kits with custom items and cooldowns
- Cooldown management per player per kit
- Discord role-based kit access
- Western-themed UI
- Database persistence for claim history
- Multi-framework support (VORP, RSG, RedEM, QBR)

## Dependencies

| Dependency | Description |
|------------|-------------|
| Framework | One of: `vorp_core`, `rsg-core`, `redemrp`, `qbr-core` |
| SQL Driver | One of: `oxmysql`, `ghmattimysql`, `mysql-async` |
| `discord_perms` | Optional, only for Discord role-restricted kits |

## Installation

1. **Create the database table:**
   ```sql
   CREATE TABLE IF NOT EXISTS gfx_kits (
       identifier VARCHAR(100) PRIMARY KEY,
       kits JSON NOT NULL
   );
   ```

2. **Add to server.cfg:**
   ```
   ensure gfxr-kit
   ```

3. **Configure:**
   - Edit `config.lua` with your framework, kits, and items

## Configuration

### config.lua

```lua
Config = {}

Config.ServerName = "GFX Server"           -- Displayed in the UI
Config.Framework = "vorp"                   -- "vorp", "rsg", "RedEM", "qbr"
Config.SQLScript = "oxmysql"               -- "oxmysql", "ghmattimysql", "mysql-async"
Config.ServerLogo = "https://example.com/logo.png"

Config.Kits = {
    starter = {
        name = "Starter Kit",
        description = "Basic supplies for new arrivals",
        duration = 6,    -- Cooldown in hours
        image = "assets/starter-icon.png",
        items = {
            { name = "bread", count = 5 },
            { name = "revolver_ammo", count = 20 },
            { name = "pistol_ammo", count = 20 }
        },
        playerCanInteract = function(source)
            return true  -- Everyone can claim
        end
    },
    weekly = {
        name = "Weekly Kit",
        description = "Weekly supply package",
        duration = 168,  -- 7 days in hours
        image = "assets/weekly-icon.png",
        items = {
            { name = "handcuffs", count = 1 },
            { name = "repair_kit", count = 2 },
            { name = "canteen", count = 1 }
        },
        playerCanInteract = function(source)
            return true
        end
    },
    discord = {
        name = "Discord Kit",
        description = "Exclusive kit for Discord members",
        duration = 6,
        image = "assets/discord-icon.png",
        items = {
            { name = "gold_bar", count = 1 }
        },
        playerCanInteract = function(source)
            return HasPlayerHasRoleOnDiscord(source, "YOUR_ROLE_ID")
        end
    }
}
```

### Kit Properties

| Property | Type | Description |
|----------|------|-------------|
| name | string | Display name |
| description | string | Short description shown in UI |
| duration | number | Cooldown in hours before kit can be claimed again |
| image | string | Path to icon image |
| items | table | Array of `{ name, count }` item definitions |
| playerCanInteract | function | Returns `true` if player can access this kit |

## Commands

| Command | Description |
|---------|-------------|
| `/kit` | Opens the kit claim UI |

## Discord Role Restriction

To restrict a kit to players with a specific Discord role:

```lua
playerCanInteract = function(source)
    return HasPlayerHasRoleOnDiscord(source, "DISCORD_ROLE_ID")
end
```

This requires the `discord_perms` resource to be running on your server.

## Notes

- Kit claim timestamps are stored per player in the `gfx_kits` database table as JSON
- Cooldown is calculated in hours from the last claim time
- The UI automatically shows remaining cooldown time for locked kits
- Kit images should be placed in the `nui/assets/` directory
