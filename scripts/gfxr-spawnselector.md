# RedM Spawn Selector UI

Spawn point selection UI for RedM. Allows players to choose their spawn location from predefined map locations or return to their last known position.

## Features

- Visual spawn location selector with background images
- "Last Location" option to spawn where you left off
- Western-themed UI with Vue.js frontend
- Configurable spawn locations with custom coordinates

## Dependencies

| Dependency | Description |
|------------|-------------|
| VORP Framework | Required (triggers `vorp:initCharacter` event) |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-spawnselector-redm
   ```

2. **Configure spawn locations:**
   - Edit `config.lua` with your desired coordinates

## Configuration

### config.lua

```lua
Config = {}

Config.Locations = {
    {
        image = "image-ambarino.png",
        name = "Ambarino",
        coords = vector3(-552.0, 632.0, 127.0)
    },
    {
        image = "image-hannover.png",
        name = "Hannover",
        coords = vector3(0, 0, 0)
    },
    {
        image = "image-elizabeth.png",
        name = "Elizabeth",
        coords = vector3(-552.0, 632.0, 127.0)
    },
    {
        image = "image-austin.png",
        name = "Austin",
        coords = vector3(-552.0, 632.0, 127.0)
    }
}
```

Each location requires:

| Field | Type | Description |
|-------|------|-------------|
| image | string | Background image filename (placed in `ui/assets/`) |
| name | string | Display name shown in the UI |
| coords | vector3 | Spawn coordinates (x, y, z) |

## Events

### Triggering the Spawn Selector

The spawn selector is opened via a client event:

```lua
TriggerClientEvent('gfxr-spawnselector:Init:Vorp', source, playerCoords, heading, isDead)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| playerCoords | vector3 | Player's last known coordinates |
| heading | number | Player's last heading/direction |
| isDead | boolean | Whether the player was dead |

### NUI Callbacks

| Callback | Description |
|----------|-------------|
| `SpawnLastLocation` | Player chose to spawn at their last location |
| `SpawnLocation` | Player selected a specific spawn point |

Both callbacks trigger `vorp:initCharacter` with the appropriate coordinates and heading.

## Adding New Locations

1. Add your location image to the `ui/assets/` folder
2. Add a new entry to `Config.Locations` in `config.lua`:
   ```lua
   {
       image = "your-image.png",
       name = "Location Name",
       coords = vector3(x, y, z)
   }
   ```
3. Restart the resource
