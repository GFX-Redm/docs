# gfxr-help

In-game help menu and support ticket system for RedM. Provides an FAQ interface with categorized help topics and a ticket system for player-to-admin communication.

## Features

- Categorized FAQ sections (Tab Menu, Inventory, Sheriff, Items, etc.)
- Search functionality across all categories
- Support ticket system with two-way messaging
- Staff authorization via Steam ID whitelist
- Player Steam avatar integration
- Optional Discord logging

## Dependencies

| Dependency | Description |
|------------|-------------|
| `dz_logs` | Optional, for Discord webhook logging |

## Installation

1. **Add to server.cfg:**
   ```
   ensure gfxr-help
   ```

2. **Configure:**
   - Edit `config.lua` with your settings
   - Edit `nui/help.json` with your FAQ content

## Configuration

### config.lua

```lua
Config = {}

Config.MenuCommand = "help"  -- Command to open the help menu

-- Steam IDs of staff who can view and respond to tickets
Config.allowedToResponseTicket = {
    ["steam:11000010b28aec6"] = true
}

Config.DiscordLog = false           -- Enable Discord logging (requires dz_logs)
Config.ServerName = "GFX"           -- Server name shown in responses
Config.ServerLogo = "https://example.com/logo.png"  -- Server logo URL
```

### help.json

The FAQ content is stored in `nui/help.json`. Structure:

```json
{
    "categories": [
        {
            "name": "Getting Started",
            "icon": "fa-book",
            "questions": [
                {
                    "question": "How do I open my inventory?",
                    "answer": "Press I or use the /inventory command."
                }
            ]
        }
    ]
}
```

## Commands

| Command | Description |
|---------|-------------|
| `/help` | Opens the help menu (configurable via `Config.MenuCommand`) |

## Ticket System

### For Players
- Open the help menu and navigate to the support section
- Create a new ticket with a description of the issue
- View responses from staff and reply to messages
- Close tickets when the issue is resolved

### For Staff
- Staff members identified by Steam ID in `Config.allowedToResponseTicket`
- Can see all open tickets from players
- Can respond to tickets with messages
- Can close tickets

## Events

### Server Events

| Event | Description |
|-------|-------------|
| `gfxr-help:updateTicket` | Triggered when a message is sent or ticket created |
| `gfxr-help:closeTicket` | Triggered when a ticket is closed |

### Client Events

| Event | Description |
|-------|-------------|
| `gfxr-help:openMenu` | Opens the help menu UI |
| `gfxr-help:updateTicket` | Updates the ticket list on the client |

## Customization

- **FAQ Content:** Edit `nui/help.json` to add/modify categories and questions
- **Styling:** Modify `nui/style.css` for visual changes
- **Images:** Add custom images to the `nui/` directory
- **Fonts:** Uses Bai Jamjuree, Montserrat, and NicksonOne fonts

## Notes

- Player Steam avatars are fetched from the Steam Community API
- Tickets persist during the server session (not saved to database)
- Discord logging uses the `dz_logs` export when enabled
