# gfxr-inventory

React-based drag-and-drop inventory UI for RedM/FiveM. Provides a modern inventory management interface with item categorization, weight management, and cross-inventory transfers.

## Features

- Drag-and-drop item management with @dnd-kit
- Dual inventory display (player + secondary container/drop/store)
- Item weight system with visual weight bars
- Item categorization (Weapons, Armor, Consumables, All)
- Item stacking and quantity splitting
- Configurable slot count and max weight
- Multi-framework support (ESX, QBCore)
- Multi-language support

## Dependencies

| Dependency | Description |
|------------|-------------|
| Framework | ESX or QBCore (auto-detected) |
| Inventory | One of: `qb-inventory`, `esx_inventoryhud`, `qs-inventory`, `codem-inventory`, `ox_inventory` |
| SQL Driver | One of: `oxmysql`, `ghmattimysql`, `mysql-async` |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-inventory
   ```

2. **Build the UI (if not pre-built):**
   ```bash
   cd resources/[gfx]/gfxr-inventory/web
   npm install
   npm run build
   ```

3. **Configure:**
   - `config/client_config.lua` - Inventory settings
   - `config/locale.lua` - Language translations

## Configuration

### client_config.lua

```lua
Config = {
    PlayerInventorySettings = {
        slots = 20,           -- Number of player inventory slots
        maxWeight = 100       -- Max weight capacity (number or "infinite")
    }
}
```

### locale.lua

Supports English and French. Uses `_L(key)` function for string lookups.

## Commands

| Command | Description |
|---------|-------------|
| `/show-nui` | Opens the inventory UI (with test data) |

## NUI Events

### Drag Item

Fired when items are moved between slots:

```lua
RegisterNUICallback('inventory:dragItem', function(data, cb)
    -- data.from: source inventory type
    -- data.to: target inventory type
    -- data.fromId: source slot ID
    -- data.toId: target slot ID
    cb({})
end)
```

## Item Structure

```lua
{
    name = "bread",           -- Item identifier
    label = "Bread",          -- Display name
    count = 5,                -- Quantity
    weight = 0.5,             -- Weight per unit
    unique = false,           -- If true, items swap instead of stack
    description = "Fresh bread",
    image = "bread.png"       -- Item image
}
```

## Drag-and-Drop Behavior

- **Same inventory:** Items swap positions or stack if same type
- **Cross inventory:** Items transfer between player and secondary inventory
- **Unique items:** Always swap positions, never stack
- **Weight check:** Transfers are validated against max weight before completing
- **Quantity input:** Hold modifier to specify partial quantity transfers

## Debug Mode

```
setr gfxr-inventory-debugMode 1
```

## Development

```bash
cd web
npm run start:game   # Watch mode for in-game development
```
