# RedM Notifications · gfxr-notify

Modern notification, input dialog, and progress bar system for RedM with a Western-themed UI built in React.

## Features

- Toast-style notifications (success, error, warning, info)
- Cancellable progress bars with optional animations
- Input dialogs with multiple field types (text, number, password, select, textarea)
- Full-screen announcements
- Customizable Western-themed design via CSS variables
- Multi-language support (English, Turkish)

## Dependencies

| Dependency | Description |
|------------|-------------|
| `gfxr-bridge` | Framework abstraction layer |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-bridge
   ensure gfxr-notify
   ```

2. **Build the UI (if not pre-built):**
   ```bash
   cd resources/[gfx]/gfxr-notify/web
   npm install
   npm run build
   ```

3. **Configure:**
   - `config/client_config.lua` - Notification settings, theme
   - `config/locale.lua` - Language translations

## Configuration

### client_config.lua

```lua
Config.Locale = 'en'                    -- Language ('en' or 'tr')
Config.Debug = false                     -- Debug logging
Config.DefaultDuration = 5000            -- Default notification duration (ms)
Config.MaxVisibleNotifications = 5       -- Max stacked notifications
Config.Position = 'top-right'            -- Notification position
Config.AnimationDuration = 300           -- Animation timing (ms)
Config.ProgressBarPosition = 'bottom-center'
Config.InputOverlayOpacity = 0.7         -- Modal backdrop opacity
Config.PlaySound = true                  -- Play sound on notification
Config.SoundVolume = 0.3                 -- Sound volume

-- Theme (CSS variables)
Config.Theme = {
    ['--primary'] = '#DAA520',
    ['--success'] = '#2d5a1b',
    ['--error'] = '#8b1a1a',
    ['--warning'] = '#b8860b',
    ['--info'] = '#1a3d5c',
    ['--bg-primary'] = '#1a1410',
    ['--bg-secondary'] = '#2a2118',
    ['--text-primary'] = '#f4e4c1',
    ['--text-secondary'] = '#c4a882',
    ['--border'] = '#3a2e22',
    ['--font-family'] = "'Rye', serif"
}
```

## Client Exports

### ShowNotification

```lua
exports['gfxr-notify']:ShowNotification(message, type, duration)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| message | string | Notification text |
| type | string | `"success"`, `"error"`, `"warning"`, `"info"` |
| duration | number\|nil | Duration in ms (optional) |

### ShowProgressBar

```lua
local completed = exports['gfxr-notify']:ShowProgressBar(label, duration, options)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| label | string | Progress bar label |
| duration | number | Duration in ms |
| options | table\|nil | `{ canCancel = true, animation = { dict = "...", anim = "..." } }` |

**Returns:** `true` if completed, `false` if cancelled.

### ShowInputDialog

```lua
local result = exports['gfxr-notify']:ShowInputDialog(title, inputs)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| title | string | Dialog title |
| inputs | table | Array of input definitions |

**Input types:**
```lua
{
    { type = 'text', label = 'Name', placeholder = 'Enter name' },
    { type = 'number', label = 'Amount', placeholder = '0' },
    { type = 'password', label = 'Password' },
    { type = 'select', label = 'Option', options = { {value = '1', label = 'One'} } },
    { type = 'textarea', label = 'Reason', placeholder = 'Enter reason' }
}
```

**Returns:** Table of submitted values, or `nil` if cancelled.

### ShowAnnouncement

```lua
exports['gfxr-notify']:ShowAnnouncement(title, message, duration)
```

## Server Exports

### Notify

```lua
exports['gfxr-notify']:Notify(source, message, type, duration)
```

Send a notification to a specific player.

### NotifyAll

```lua
exports['gfxr-notify']:NotifyAll(message, type, duration)
```

Send a notification to all players.

### Announce

```lua
exports['gfxr-notify']:Announce(source, title, message, duration)
```

Send an announcement to a specific player.

### AnnounceAll

```lua
exports['gfxr-notify']:AnnounceAll(title, message, duration)
```

Send an announcement to all players.

## Test Commands

| Command | Description |
|---------|-------------|
| `/testnotify` | Display all 4 notification types |
| `/testprogress` | Show a 5-second cancellable progress bar |
| `/testinput` | Show an input dialog with sample fields |
