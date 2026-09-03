# RedM Player HUD · gfxr-hud

Framework-agnostic RedM HUD with player vitals, metabolism, contextual horse widget, money, player info, clock/weather, voice indicator, minimap controller, and a fully in-game NUI settings menu. Settings are persisted entirely in the browser's localStorage — no database required.

## Features

- Player vitals: health, stamina, and Dead Eye — each shows a core value and a bar (e.g. Dead Eye bar vs. Dead Eye core)
- Needs: hunger, thirst, and stress (0–100); master toggle for servers without a metabolism resource
- Contextual horse widget: health, stamina, and cleanliness — auto-shows on mount, auto-hides on dismount
- Money widget: cash, gold, and bank (bank is 0 on VORP which has no core bank)
- Player info widget: identifier, character name, and current job
- Clock and weather widget: in-game time and a weather icon mapped from the settled weather hash
- Voice indicator: detects pma-voice, yaca, saltychat, or mumble by priority; shows range (whisper/normal/shout) and talking state
- Minimap controller: circle, square, or off shape; zoom level; fog-of-war toggle
- In-game NUI settings menu opened via F7 (raw key, polled directly on RedM) or the `hudmenu` console command — supports drag-to-reposition, scale slider, enable/disable per element, theme picker with live preview, and minimap shape selector
- Three UI themes: Authentic (Western ornate), Minimal (clean), Ornate (detailed)
- Settings persisted in the NUI's CEF localStorage — survive game restarts, no SQL required
- Event-driven money and needs updates via the bridge `OnMoneyChange` / `OnNeedsChange` — no server polling for either (see Performance below)
- Delta-only NUI pushes: only changed values are sent to the React layer, keeping idle resmon near zero

## Architecture

gfxr-hud is **fully client-side**. There are no server scripts and no database. The `fxmanifest.lua` declares only `client_scripts`; the `server_scripts` block and `sql/` directory have been removed entirely.

Settings (theme, element positions/scale/enabled state, minimap shape/zoom) are stored in the NUI's CEF localStorage under the single key `gfxr-hud:settings`. Because CEF localStorage is machine-wide, settings are shared by all characters on the same PC and persist across game restarts and resource restarts without any server interaction.

## Dependencies

| Dependency | Description |
|------------|-------------|
| `gfxr-bridge` | Framework abstraction layer (VORP / RSG / RedEM:RP) |

## Installation

1. **Add to server.cfg** (must come after `gfxr-bridge`):
   ```
   ensure gfxr-bridge
   ensure gfxr-hud
   ```

2. **NUI:** the `web/build/` folder is pre-built. If you need to rebuild after editing `web/src/`:
   ```bash
   cd resources/[gfx]/gfxr-hud/web
   npm install
   npm run build
   ```

No SQL import step is required.

## Configuration

### config/client_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.MENU_COMMAND` | `"hudmenu"` | Console command that opens the settings menu; also usable as a bindable command |
| `Config.MENU_KEYBIND` | `"F7"` | Default key passed to `RegisterKeyMapping` (FiveM key-binding UI only; ignored on RedM) |
| `Config.MENU_KEY` | `0x76` | **RedM — raw Windows virtual-key code polled via `IsRawKeyJustPressed`.** This is the key that actually opens the menu on rdr3. Default `0x76` = F7. Common codes: F1=0x70, F2=0x71 … F7=0x76 … F12=0x7B; digits 0–9 = 0x30–0x39; A–Z = 0x41–0x5A. Set to `false` to disable raw-key polling entirely. |
| `Config.MENU_CONTROL` | `false` | Optional game-control-hash fallback, checked only when raw-key polling is unavailable. Disabled by default. The old default (0xE8342FF2 = INPUT\_MULTIPLAYER\_INFO / Left Alt) clashed with targeting — that was a bug, now corrected. |
| `Config.UPDATE_INTERVAL` | `1000` | Milliseconds between local-native sampling loops (cores, horse, clock, weather) |
| `Config.FAST_UPDATE_INTERVAL` | `250` | Milliseconds between voice indicator samples |
| `Config.DELTA_EPSILON` | `1` | Minimum integer-percent change before a vital or need value is re-pushed to the NUI |
| `Config.DEFAULT_THEME` | `1` | Starting theme: `1` = Authentic, `2` = Minimal, `3` = Ornate (used until localStorage settings load in the NUI) |
| `Config.SHOW_ON_LOAD` | `true` | Show the HUD automatically once the player is loaded |
| `Config.HIDE_IN_CUTSCENE` | `true` | Auto-hide during scripted/networked cutscenes |
| `Config.HIDE_IN_PAUSE_MENU` | `true` | Auto-hide when the pause menu is open |
| `Config.HIDE_ON_FULLMAP` | `true` | Auto-hide the entire HUD when the full map (M) is open |
| `Config.NEEDS_ENABLED` | `true` | Master toggle for the hunger/thirst/stress widgets; disable on servers without a metabolism resource |
| `Config.HIDE_NATIVE_CORES` | `true` | Hide RDR2's built-in attribute cores so gfxr-hud draws its own |
| `Config.TEMPERATURE_ENABLED` | `true` | Show the ambient temperature badge |
| `Config.TEMPERATURE_UNIT` | `"C"` | Default temperature unit shown on the badge: `"C"` (Celsius) or `"F"` (Fahrenheit); players can switch in settings |
| `Config.POPULATION_ENABLED` | `true` | Show the online player count (online/max), polled from the server |
| `Config.POPULATION_INTERVAL` | `30000` | Milliseconds between server player-count polls |
| `Config.VISIBILITY.CORES` | `"visible"` | Visibility mode for player bars (health/stamina/needs): `"visible"`, `"hidden"`, or `"onchange"` |
| `Config.VISIBILITY.MONEY` | `"visible"` | Visibility mode for the money widget |
| `Config.VISIBILITY.VOICE` | `"visible"` | Visibility mode for the voice indicator |
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
| `Config.MINIMAP.BORDER_STYLE` | `"compass"` | Decorative minimap frame style (NUI only): `"compass"`, `"minimal"`, `"double"`, `"rope"`, `"studded"`, `"ornate"` |
| `Config.MINIMAP.LABEL_STYLE` | `"pill"` | Location label style shown below the minimap: `"pill"`, `"banner"`, `"ribbon"`, `"plate"`, `"minimal"` |
| `Config.LOCATION.ENABLED` | `true` | Show the township/region label under the minimap |
| `Config.LOCATION.FALLBACK` | `"Frontier"` | Label shown when no known zone matches (deep wilderness) |
| `Config.ELEMENTS` | *(see below)* | Per-element defaults; overridden by the NUI's localStorage-loaded settings |
| `Config.Notify` | `nil` | Optional function override for notifications; if nil, the bridge `Notify` is used |

#### Config.ELEMENTS

Each entry has the shape `{ ENABLED, X, Y, SCALE }` where `X`/`Y` are viewport-normalized (0–1) and `SCALE` is a multiplier.

| Element key | Default X | Default Y | Default SCALE | Notes |
|-------------|-----------|-----------|---------------|-------|
| `health` | `0.04` | `0.86` | `1.0` | |
| `stamina` | `0.09` | `0.90` | `1.0` | |
| `deadeye` | `0.14` | `0.86` | `1.0` | Disabled by default (`ENABLED = false`) |
| `hunger` | `0.04` | `0.78` | `1.0` | Hidden when `NEEDS_ENABLED = false` |
| `thirst` | `0.04` | `0.74` | `1.0` | Hidden when `NEEDS_ENABLED = false` |
| `stress` | `0.04` | `0.70` | `1.0` | Hidden when `NEEDS_ENABLED = false`; always 0 on VORP |
| `temp` | `0.50` | `0.965` | `1.0` | Ambient temperature badge; hidden when `TEMPERATURE_ENABLED = false` |
| `horse` | `0.04` | `0.58` | `1.0` | Managed by `HORSE_WIDGET` toggle |
| `money` | `0.9846` | `0.1778` | `1.0` | |
| `info` | `0.9155` | `0.1226` | `1.0` | |
| `clock` | `0.50` | `0.0356` | `1.0` | |
| `weather` | `0.53` | `0.03` | `1.0` | |
| `voice` | `0.47` | `0.92` | `1.0` | Managed by `VOICE` toggle |
| `minimap` | `0.0319` | `0.9414` | `1.15` | Managed by `MINIMAP` toggle |
| `population` | `0.972` | `0.2343` | `1.0` | Managed by `POPULATION_ENABLED` toggle |
| `player-row` | `0.50` | `0.9659` | `1.0` | Bottom-center player row |
| `horse-row` | `0.50` | `0.965` | `1.0` | Bottom-center horse row |

## Settings Persistence

Settings are saved entirely inside the NUI's CEF localStorage under the key `gfxr-hud:settings`. This includes:

- Active theme (1 = Authentic, 2 = Minimal, 3 = Ornate)
- Per-element position (X/Y), scale, and enabled state
- Minimap shape and zoom level

**Save / Reset behaviour:**

- Clicking **Save** in the settings menu writes the current layout to localStorage immediately. The minimap shape/zoom is also applied to the in-game radar via a `setMinimap` NUI callback.
- Clicking **Reset** clears the stored settings and restores the `Config.*` defaults defined in `config/client_config.lua`.

**Machine-wide scope:** because CEF localStorage is per-machine (not per-character or per-account), all characters played on the same PC share the same HUD layout. There is no per-character or per-server storage.

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

Change the minimap shape immediately (applies the radar native; does not persist — use the settings menu to persist to localStorage).

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
