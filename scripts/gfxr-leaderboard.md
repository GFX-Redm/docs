# RedM Leaderboard UI

React-based leaderboard UI for RedM/FiveM. Displays player statistics and rankings with switchable stat categories.

## Features

- Ranked player list with statistics
- Multiple stat categories (Kills, Deaths, K/D Ratio, etc.)
- Player self-data highlighting
- Player avatars and usernames
- Modern React + TypeScript + Redux frontend
- Multi-language support

## Dependencies

| Dependency | Description |
|------------|-------------|
| Framework | ESX or QBCore (auto-detected) |
| SQL Driver | One of: `oxmysql`, `ghmattimysql`, `mysql-async` |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-leaderboard
   ```

2. **Build the UI (if not pre-built):**
   ```bash
   cd resources/[gfx]/gfxr-leaderboard/web
   npm install
   npm run build
   ```

3. **Configure:**
   - `config/locale.lua` - Language translations

## Configuration

### locale.lua

```lua
Config = {}
Config.Locale = 'en'  -- 'en' or 'fr'

Locales = {
    en = {
        leader = "Leader",
        board = "Board",
        close = "Close"
    },
    fr = {
        leader = "Classement",
        board = "Tableau",
        close = "Fermer"
    }
}
```

## Commands

| Command | Description |
|---------|-------------|
| `/show-nui` | Opens the leaderboard UI |

## NUI Callbacks

These callbacks must be implemented on the server side to provide data:

### getStatKeys

Returns available stat categories for the leaderboard tabs.

```lua
RegisterNUICallback('getStatKeys', function(_, cb)
    cb({
        { key = 'kills', label = 'Kills' },
        { key = 'deaths', label = 'Deaths' },
        { key = 'kd', label = 'K/D Ratio' }
    })
end)
```

### getItems

Returns leaderboard entries for a specific stat.

```lua
RegisterNUICallback('getItems', function(data, cb)
    local stat = data.selectedStat
    cb({
        items = {
            { index = 1, username = "Player1", avatar = "url", value = 50 },
            { index = 2, username = "Player2", avatar = "url", value = 30 }
        },
        selfData = { index = 5, username = "You", avatar = "url", value = 10 }
    })
end)
```

### closeMenu

Called when the player closes the leaderboard.

```lua
RegisterNUICallback('closeMenu', function(_, cb)
    -- Handle cleanup
    cb({})
end)
```

## Debug Mode

Enable debug mode via server convar:

```
setr gfxr-leaderboard-debugMode 1
```

## Development

For hot-reload during development:

```bash
cd web
npm run start:game   # Watch mode with in-game builds
```
