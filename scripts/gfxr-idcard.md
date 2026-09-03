# RedM ID Card · gfxr-idcard

Western-themed identification card system. Displays player information on a parchment-style ID card with mugshot support.

## Features

- Western/1899-era themed design
- Automatic mugshot capture on first character load
- Compatible with all frameworks (VORP, RSG, RedEM)
- Configurable theme and locale support
- Sheriff/Police mugshot authorization

## Dependencies

| Dependency | Description |
|------------|-------------|
| `gfxr-bridge` | Framework abstraction layer |
| `screenshot-basic` | Required for mugshot capture |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-bridge
   ensure screenshot-basic
   ensure gfxr-idcard
   ```

2. **Run the SQL migration:**
   ```sql
   -- Import sql/install.sql
   CREATE TABLE IF NOT EXISTS `gfxr_mugshots` (
       `id` INT(11) NOT NULL AUTO_INCREMENT,
       `identifier` VARCHAR(255) NOT NULL,
       `mugshot` TEXT NOT NULL,
       `taken_by` VARCHAR(255) DEFAULT NULL,
       `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
       PRIMARY KEY (`id`),
       UNIQUE KEY `identifier` (`identifier`)
   ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
   ```

3. **Configure:**
   - `config/client_config.lua` - Theme, mugshot settings
   - `config/locale.lua` - Language translations

## Configuration

### client_config.lua

```lua
Config = {}

Config.Debug = false

-- Theme colors
Config.Theme = {
    Primary = "#c21414",        -- Main color (seal, ID number)
    PrimaryContent = "#ffffff",
    Secondary = "#DAA520",      -- Secondary color (gold)
    Background = "#F4E4C1",     -- Parchment background
    Ink = "#1a1613",            -- Text color
    Parchment = "#F4E4C1"
}

-- ID Card information
Config.IdCard = {
    Territory = "New Hanover",
    EstablishedYear = "1899",
    Authority = "United States Marshal Service",
    Quote = "Justice is a commodity few can afford in these parts.",
    Signature = "Sheriff Curtis Malloy"
}

-- Mugshot settings
Config.Mugshot = {
    UploadURL = "https://your-server.com/upload",  -- Image upload endpoint
    FieldName = "file",                             -- Form field name
    AutoCapture = true,                             -- Auto-capture on load
    Camera = {
        Fov = 30.0,
        Distance = 0.6,
        HeightOffset = 0.63
    }
}

-- Commands
Config.Command = "idcard"              -- Open ID card
Config.MugshotCommand = "takemugshot"  -- Take mugshot
Config.Keybind = "F5"                  -- Keybind (false = disabled)

-- Jobs allowed to take mugshots (empty = everyone)
Config.MugshotJobs = {
    "sheriff",
    "police",
    "lawman"
}

Config.Locale = "en"  -- "en" or "tr"
```

## Commands

| Command | Description |
|---------|-------------|
| `/idcard` | Toggle ID card display |
| `/takemugshot` | Take mugshot of nearest player |
| `/takemugshot [id]` | Take mugshot of player with specified ID |
| `/selfmugshot` | Take your own mugshot |

## Exports (Client)

```lua
-- Open the ID card
exports['gfxr-idcard']:OpenIdCard()

-- Close the ID card
exports['gfxr-idcard']:CloseIdCard()

-- Check if ID card is open
local isOpen = exports['gfxr-idcard']:IsIdCardOpen()

-- Take mugshot of a target player
exports['gfxr-idcard']:TakeMugshot(targetServerId)

-- Take mugshot of nearest player
exports['gfxr-idcard']:TakeMugshotNearest()

-- Take your own mugshot
exports['gfxr-idcard']:TakeSelfMugshot()
```

## Exports (Server)

```lua
-- Get a player's mugshot URL
local mugshotUrl = exports['gfxr-idcard']:GetMugshot(source)
```

## Events

### Client

```lua
-- Fired when player data is received for the ID card
RegisterNetEvent('gfxr-idcard:receivePlayerData', function(data)
    -- data.firstName, data.lastName, data.mugshot, etc.
end)

-- Fired when auto mugshot capture is triggered
RegisterNetEvent('gfxr-idcard:autoCaptureMugshot', function()
    -- No mugshot found, auto-capture will begin
end)
```

### Server

```lua
-- Fired when a mugshot is saved
RegisterNetEvent('gfxr-idcard:saveMugshot', function(targetServerId, mugshotUrl)
    -- Mugshot has been saved to database
end)
```

## Mugshot Upload Server

Your upload endpoint must return the following format:

```json
{
    "url": "https://your-server.com/uploads/abc123.jpg"
}
```

### Example PHP Endpoint

```php
<?php
header('Content-Type: application/json');

$uploadDir = 'uploads/';
$fileName = uniqid() . '.jpg';
$targetPath = $uploadDir . $fileName;

if (move_uploaded_file($_FILES['file']['tmp_name'], $targetPath)) {
    echo json_encode([
        'url' => 'https://your-server.com/' . $targetPath
    ]);
} else {
    http_response_code(500);
    echo json_encode(['error' => 'Upload failed']);
}
```

## Data Sources

| UI Field | Bridge Export | Framework Source |
|----------|--------------|-----------------|
| Full Name | `GetPlayer()` | `firstname`, `lastname` |
| Date of Birth | `GetPlayer()` | `dateofbirth` / `birthdate` |
| Occupation | `GetPlayerJob()` | `job.label` |
| Mugshot | `ExecuteSql()` | `gfxr_mugshots` table |
| ID Number | `GetIdentifier()` | Generated via hash |

## Notes

- Auto mugshot capture triggers when a character loads for the first time
- If no mugshot exists, it is captured silently without notifying the player
- The ID number is generated by hashing the character identifier (stays consistent)
- Players with Sheriff/Police jobs can take mugshots of other players
