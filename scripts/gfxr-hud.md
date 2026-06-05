# gfxr-hud

Framework-agnostic RedM HUD with player vitals, metabolism, contextual horse widget, money, player info, clock/weather, voice indicator, minimap controller, and a fully in-game NUI settings menu with per-player SQL persistence.

## Features

- Player vitals: health, stamina, and Dead Eye — each shows a core value and a bar (e.g. Dead Eye bar vs. Dead Eye core)
- Needs: hunger, thirst, and stress (0–100); master toggle for servers without a metabolism resource
- Contextual horse widget: health, stamina, and cleanliness — auto-shows on mount, auto-hides on dismount
- Money widget: cash, gold, and bank (bank is 0 on VORP which has no core bank)
- Player info widget: identifier, character name, and current job
- Clock and weather widget: in-game time and a weather icon mapped from the settled weather hash
- Voice indicator: detects pma-voice, yaca, saltychat, or mumble by priority; shows range (whisper/normal/shout) and talking state
- Minimap controller: circle, square, or off shape; zoom level; fog-of-war toggle
- In-game NUI settings menu opened via a rebindable keybind or console command — supports drag-to-reposition, scale slider, enable/disable per element, theme picker with live preview, and minimap shape selector
- Three UI themes: Authentic (Western ornate), Minimal (clean), Ornate (detailed)
- Per-player settings persisted to SQL; first-time players receive seeded defaults
- Event-driven money and needs updates via the bridge `OnMoneyChange` / `OnNeedsChange` — no server polling for either (see Performance below)
- Delta-only NUI pushes: only changed values are sent to the React layer, keeping idle resmon near zero

## Dependencies

| Dependency | Description |
|------------|-------------|
| `gfxr-bridge` | Framework abstraction layer (VORP / RSG / RedEM:RP) |

## Installation

1. **Import the SQL schema:**
   ```sql
   -- Run once against your server database
   source resources/[gfx]/gfxr-hud/sql/gfxr_hud.sql
   ```

2. **Add to server.cfg** (must come after `gfxr-bridge`):
   ```
   ensure gfxr-bridge
   ensure gfxr-hud
   ```

3. **NUI:** the `web/build/` folder is pre-built. If you need to rebuild after editing `web/src/`:
   ```bash
   cd resources/[gfx]/gfxr-hud/web
   npm install
   npm run build
   ```

## Configuration

### config/client_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.MENU_KEYBIND` | `"F7"` | Default key for `RegisterKeyMapping`; players can rebind in GTA settings |
| `Config.MENU_COMMAND` | `"hudmenu"` | Console command that also opens the settings menu |
| `Config.UPDATE_INTERVAL` | `1000` | Milliseconds between local-native sampling loops (cores, horse, clock, weather) |
| `Config.FAST_UPDATE_INTERVAL` | `250` | Milliseconds between voice indicator samples |
| `Config.DELTA_EPSILON` | `1` | Minimum integer-percent change before a vital or need value is re-pushed to the NUI |
| `Config.DEFAULT_THEME` | `1` | Starting theme: `1` = Authentic, `2` = Minimal, `3` = Ornate (overridden by SQL on load) |
| `Config.SHOW_ON_LOAD` | `true` | Show the HUD automatically once the player is loaded |
| `Config.HIDE_IN_CUTSCENE` | `true` | Auto-hide during scripted/networked cutscenes |
| `Config.HIDE_IN_PAUSE_MENU` | `true` | Auto-hide when the pause menu is open |
| `Config.NEEDS_ENABLED` | `true` | Master toggle for the hunger/thirst/stress widgets; disable on servers without a metabolism resource |
| `Config.HORSE_WIDGET.ENABLED` | `true` | Show the horse widget |
| `Config.HORSE_WIDGET.AUTO_HIDE` | `true` | Automatically hide the widget when the player is not mounted |
| `Config.VOICE.ENABLED` | `true` | Show the voice indicator widget |
| `Config.VOICE.RANGES.whisper` | `2.0` | Proximity threshold (meters) for the whisper label |
| `Config.VOICE.RANGES.normal` | `8.0` | Proximity threshold (meters) for the normal label |
| `Config.VOICE.RANGES.shout` | `20.0` | Proximity threshold (meters) for the shout label |
| `Config.MINIMAP.ENABLED` | `true` | Show the minimap |
| `Config.MINIMAP.SHAPE` | `"circle"` | Default minimap shape: `"circle"`, `"square"`, or `"off"` |
| `Config.MINIMAP.ZOOM` | `1080` | Default radar zoom level |
| `Config.MINIMAP.HIDE_FOW` | `false` | Reveal fog of war on the minimap |
| `Config.ELEMENTS` | *(see below)* | Per-element defaults; overridden by SQL-loaded settings |
| `Config.Notify` | `nil` | Optional function override for notifications; if nil, the bridge `Notify` is used |

#### Config.ELEMENTS

Each entry has the shape `{ ENABLED, X, Y, SCALE }` where `X`/`Y` are viewport-normalized (0–1) and `SCALE` is a multiplier.

| Element key | Default X | Default Y | Notes |
|-------------|-----------|-----------|-------|
| `health` | `0.04` | `0.86` | |
| `stamina` | `0.09` | `0.90` | |
| `deadeye` | `0.14` | `0.86` | |
| `hunger` | `0.04` | `0.78` | Hidden when `NEEDS_ENABLED = false` |
| `thirst` | `0.04` | `0.74` | Hidden when `NEEDS_ENABLED = false` |
| `stress` | `0.04` | `0.70` | Hidden when `NEEDS_ENABLED = false`; always 0 on VORP |
| `horse` | `0.04` | `0.58` | Managed by `HORSE_WIDGET` toggle |
| `money` | `0.86` | `0.04` | |
| `info` | `0.86` | `0.12` | |
| `clock` | `0.47` | `0.03` | |
| `weather` | `0.53` | `0.03` | |
| `voice` | `0.47` | `0.92` | Managed by `VOICE` toggle |
| `minimap` | `0.02` | `0.74` | Managed by `MINIMAP` toggle |

All elements default to `ENABLED = true` and `SCALE = 1.0`.

### config/server_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.AUTOSAVE_DEBOUNCE` | `1500` | Milliseconds to wait after the last settings change before writing to SQL (prevents write spam while dragging elements) |
| `Config.DEFAULT_SETTINGS` | *(mirrors client defaults)* | Seed row written to SQL for first-time players; includes `theme`, `elements`, and `minimap` |
| `Config.VALID_ELEMENTS` | *(list of 13 ids)* | Whitelist of accepted element keys in `saveSettings` validation |
| `Config.SCALE_MIN` | `0.5` | Minimum accepted element scale value |
| `Config.SCALE_MAX` | `2.0` | Maximum accepted element scale value |
| `Config.VALID_THEMES` | `{[1]=true,[2]=true,[3]=true}` | Accepted theme ids |
| `Config.VALID_SHAPES` | `{circle=true,square=true,off=true}` | Accepted minimap shape strings |

## Performance

Money and needs are **event-driven**, never polled from the server by the HUD itself:

- `OnMoneyChange` — RSG fires its native `RSGCore:Client:OnMoneyChange` event; VORP/RedEM diff the local character object on a low-rate client loop. Zero server calls.
- `OnNeedsChange` — RSG reads client metadata on `RSGCore:Player:SetPlayerData`; VORP listens to `vorp_metabolism` client events with a 2 s fallback loop (all client-local). RedEM is the sole exception: the bridge does a 3 s server poll internally because RedEM exposes no client needs signal.

Only local natives (player cores, horse entity, clock, weather hash) are polled by the HUD's own `UPDATE_INTERVAL` loop. The NUI React layer never polls — it only updates when the Lua side pushes a delta.

## Client Exports

### ShowHud

Show or hide the HUD.

```lua
exports['gfxr-hud']:ShowHud(state)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| state | boolean | `true` to show, `false` to hide |

### IsHudVisible

Returns whether the HUD is currently visible.

```lua
local visible = exports['gfxr-hud']:IsHudVisible()
```

**Returns:** boolean

### SetMinimapShape

Change the minimap shape immediately (does not persist; use the settings menu to persist).

```lua
exports['gfxr-hud']:SetMinimapShape(shape)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| shape | string | `"circle"`, `"square"`, or `"off"` |

Invalid values are silently ignored.

### OpenSettings

Open the HUD settings menu programmatically.

```lua
exports['gfxr-hud']:OpenSettings()
```

No-op if the menu is already open.

## Net Events

### gfxr-hud:forceRefresh (client)

Forces the HUD to re-push its current config and locale to the NUI and clear the delta cache so all values are re-sent on the next tick. Trigger from the server when a live config reload is needed.

```lua
TriggerClientEvent('gfxr-hud:forceRefresh', source)
```

### gfxr-hud:saveSettings (server, net event)

Sent by the client when the player saves settings from the menu. The server validates, sanitizes, and debounces the write to SQL. Other resources should not fire this event directly.

### gfxr-hud:getSettings (server callback)

Registered via `exports['gfxr-bridge']:RegisterCallback`. Called by the client on player load to fetch saved settings. Returns `{ theme, elements, minimap }` from SQL, or seeds and returns defaults for first-time players.

```lua
-- Client usage (internal):
local settings = exports['gfxr-bridge']:TriggerCallback('gfxr-hud:getSettings')
```

## Locale

Three languages are included. Set `Locale = 'en'` (or `'tr'` / `'fr'`) at the top of `config/locale.lua`.

| Key | en | tr | fr |
|-----|----|----|----|
| `menu_title` | HUD Settings | HUD Ayarlari | Parametres HUD |
| `tab_elements` | Elements | Ogeler | Elements |
| `tab_layout` | Layout | Yerlesim | Disposition |
| `tab_theme` | Theme | Tema | Theme |
| `tab_minimap` | Minimap | Mini Harita | Mini-carte |
| `theme_authentic` | Authentic | Ozgun | Authentique |
| `theme_minimal` | Minimal | Minimal | Minimal |
| `theme_ornate` | Ornate | Suslu | Orne |
| `element_health` | Health | Saglik | Sante |
| `element_stamina` | Stamina | Dayaniklilik | Endurance |
| `element_deadeye` | Dead Eye | Dead Eye | Dead Eye |
| `element_hunger` | Hunger | Aclik | Faim |
| `element_thirst` | Thirst | Susuzluk | Soif |
| `element_stress` | Stress | Stres | Stress |
| `element_horse` | Horse | At | Cheval |
| `element_money` | Money | Para | Argent |
| `element_info` | Player Info | Oyuncu Bilgisi | Infos Joueur |
| `element_clock` | Clock | Saat | Horloge |
| `element_weather` | Weather | Hava Durumu | Meteo |
| `element_voice` | Voice | Ses | Voix |
| `element_minimap` | Minimap | Mini Harita | Mini-carte |
| `label_enabled` | Enabled | Etkin | Active |
| `label_scale` | Scale | Olcek | Echelle |
| `label_position` | Position | Konum | Position |
| `minimap_shape` | Minimap Shape | Mini Harita Sekli | Forme de la mini-carte |
| `shape_circle` | Circle | Daire | Cercle |
| `shape_square` | Square | Kare | Carre |
| `shape_off` | Off | Kapali | Desactive |
| `voice_whisper` | Whisper | Fisilti | Chuchotement |
| `voice_normal` | Normal | Normal | Normal |
| `voice_shout` | Shout | Bagirma | Cri |
| `horse_dirt` | Cleanliness | Temizlik | Proprete |
| `btn_save` | Save | Kaydet | Enregistrer |
| `btn_reset` | Reset | Sifirla | Reinitialiser |
| `btn_edit_layout` | Edit Layout | Yerlesimi Duzenle | Modifier la disposition |
| `btn_done` | Done | Bitti | Termine |
| `notify_saved` | HUD settings saved | HUD ayarlari kaydedildi | Parametres HUD enregistres |
| `notify_save_failed` | Could not save HUD settings | HUD ayarlari kaydedilemedi | Impossible d'enregistrer les parametres HUD |
| `notify_reset` | HUD settings reset to defaults | HUD ayarlari varsayilana sifirlandi | Parametres HUD reinitialises |

## SQL Schema

Table: `gfxr_hud_settings`

| Column | Type | Description |
|--------|------|-------------|
| `identifier` | `VARCHAR(80)` PRIMARY KEY | Player identifier (citizenid / charid / license) |
| `theme` | `INT` | Active theme id (1, 2, or 3) |
| `elements` | `LONGTEXT` | JSON object mapping element id to `{ enabled, x, y, scale }` |
| `minimap` | `LONGTEXT` | JSON object: `{ enabled, shape, zoom }` |
| `updated_at` | `TIMESTAMP` | Auto-updated on every write |

One row per player. First login inserts a seed row from `Config.DEFAULT_SETTINGS`. The `elements` and `minimap` columns are decoded by the server from JSON on read and re-encoded on write.
