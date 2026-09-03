# RedM RP Chat · gfxr-chat

Frontier-Noir-styled RP chat for RedM. A fork of the Cfx stock `chat` resource extended with proximity channels, RP commands, 3D floating text, persistent scene markers, per-player settings, a moderation suite, and an admin panel — all wired to gfxr-bridge. The upstream chat contract (`chat:addMessage`, `chat:addSuggestion`, `registerMessageHook`, etc.) is fully preserved for compatibility with other resources.

## Features

- Five channels: IC, OOC, Job, Admin, Scene — each independently toggleable; tabs are per-player (non-admins never see Admin, jobless players never see Job)
- Proximity-based broadcast for all IC speech; server-wide option for OOC
- RP commands: `/me`, `/do`, `/try`, `/whisper` (`/w`), `/shout` (`/s`), `/ooc`, `/scene`, `/jobchat`, `/achat`, `/dice`, `/coin`, `/announce`
- 3D floating text above the sender's head for `/me`, `/do`, and `/try` — rides the SKEL\_Head bone, fades with distance and time; player-selectable style (plate / minimal / bubble / parchment / off)
- Persistent world scene markers: `/scene` pins a 3D text at the writer's position; survives until the server restarts or a per-marker lifetime elapses
- Typing bubble above a player's head while they compose a message
- @mention autocomplete with sound notification; nearby-only mode available
- Bottom-left rolling RP log (scene chat): feeds `/me`, `/do`, `/try`, `/ooc` lines
- Emoji picker; message deletion (own messages or admin)
- Per-player settings: timestamps (12/24 h with optional seconds), text size per section, colour picker, font presets (journal / quill / clean), chat visibility mode (Active / Always / Hidden, L key to cycle), per-player language override
- Draggable, independently scalable sections (chat panel + scene log) with snap-to-edge / snap-to-centre in edit mode; layout persisted via KVP
- Theme editor with live preview; theme persisted via KVP
- Admin panel: player list with mute (duration picker) and kick; report queue from `/report`; rolling in-memory history search by player or text
- Moderation: word filter (censor or block mode), anti-flood sliding window with optional auto-mute, persistent mutes via SQL (`gfxr_chat_mutes` — auto-created)
- Admin commands: `/mute`, `/unmute`, `/kick`, `/clearchat`, `/clearscene`, `/announce`
- Discord logging: admin-action webhook + optional full message log per channel
- Ships en / de / pt-BR / fr / th / es / ro / tr; per-player language switchable in settings

## Architecture

gfxr-chat is a **fork of the Cfx `chat` resource**. The upstream client (`cl_chat.lua`) and server (`sv_chat.lua`) files are included unmodified to preserve the stock event/export contract. The gfxr feature layer is layered on top in `client/*.lua` and `server/*.lua`.

The NUI is a **Vue application** built with webpack and shipped pre-built in `dist/`. The `ui_page` is `dist/ui.html`. To rebuild after editing `html/`:

```
npx webpack
```

All player preferences (theme, layout, settings) are stored **client-side via KVP** — no database required for UI state. Only mutes use SQL.

## Dependencies

| Dependency | Description |
|------------|-------------|
| `gfxr-bridge` | Framework abstraction layer (VORP / RSG / RedEM:RP) |

## Installation

1. **Add to server.cfg** (must come after `gfxr-bridge`):
   ```
   ensure gfxr-bridge
   ensure gfxr-chat
   ```

2. **NUI:** `dist/` is pre-built. If you edit `html/` sources, rebuild with:
   ```bash
   cd resources/[gfx]/gfxr-chat
   npx webpack
   ```

3. **SQL:** the `gfxr_chat_mutes` table is created automatically on first start — no manual import required.

## Configuration

### config/server_config.lua

#### Channels

| Key | Default | Description |
|-----|---------|-------------|
| `Config.CHANNELS` | *(see below)* | Channel definitions (id, label, scope, enabled) |
| `Config.JOB_CHANNELS` | `{}` | Job names allowed to use the Job channel. Empty = any real job. Example: `{ sheriff = true }` |
| `Config.NO_JOB_NAMES` | `{"unemployed", "nojob", "none", "civ", "civilian", "citizen", ""}` | Job names treated as "no job"; these players cannot see or use the Job channel |
| `Config.OOC_SERVERWIDE` | `false` | If `true`, `/ooc` broadcasts to all players instead of proximity |

**Default channels:**

| id | label | scope | Default enabled |
|----|-------|-------|-----------------|
| `ic` | IC | all | true |
| `ooc` | OOC | all | true |
| `job` | Job | job | true — gated by `JOB_CHANNELS` / job check |
| `admin` | Admin | admin | true — gated by admin check |
| `scene` | Scene | all | true — read-only rolling RP log |

#### Proximity

| Key | Default (metres) | Description |
|-----|-----------------|-------------|
| `Config.PROXIMITY.WHISPER` | `3.0` | `/whisper` / `/w` radius |
| `Config.PROXIMITY.ME` | `8.0` | `/me` radius |
| `Config.PROXIMITY.DO` | `8.0` | `/do` radius |
| `Config.PROXIMITY.TRY` | `8.0` | `/try` radius |
| `Config.PROXIMITY.NORMAL` | `15.0` | Plain IC and `/ooc` (local) radius |
| `Config.PROXIMITY.OOC` | `15.0` | `/ooc` local radius (ignored when `OOC_SERVERWIDE = true`) |
| `Config.PROXIMITY.SHOUT` | `35.0` | `/shout` / `/s` radius |

#### 3D Floating Text (server gate)

| Key | Default | Description |
|-----|---------|-------------|
| `Config.FLOATING_TEXT.ENABLED` | `true` | Master switch for the server to emit `render3D` events; set `false` to disable entirely |

#### Scene Markers

| Key | Default | Description |
|-----|---------|-------------|
| `Config.SCENE_MARKER.ENABLED` | `true` | Enable persistent world scene markers |
| `Config.SCENE_MARKER.DEFAULT_DURATION` | `0` | Seconds a marker lives; `0` = until the server restarts |
| `Config.SCENE_MARKER.MAX_DURATION` | `86400` | Cap for a player-supplied duration token (seconds, 24 h) |
| `Config.SCENE_MARKER.MAX_TOTAL` | `60` | Hard cap on live markers; oldest is dropped when exceeded |
| `Config.SCENE_MARKER.CLEANUP_MS` | `5000` | Server prune interval for expired markers (ms) |

#### Typing Indicator

| Key | Default | Description |
|-----|---------|-------------|
| `Config.TYPING_INDICATOR.ENABLED` | `true` | Relay typing state to nearby players |
| `Config.TYPING_INDICATOR.RADIUS` | `0` | Broadcast radius (metres); `0` = use `PROXIMITY.NORMAL` |

#### Mentions

| Key | Default | Description |
|-----|---------|-------------|
| `Config.MENTIONS.NEARBY_ONLY` | `false` | Restrict the mention picker to players within `RADIUS` metres |
| `Config.MENTIONS.RADIUS` | `20.0` | Metres (used only when `NEARBY_ONLY = true`) |

#### Moderation

| Key | Default | Description |
|-----|---------|-------------|
| `Config.WORD_FILTER.ENABLED` | `true` | Enable the word filter |
| `Config.WORD_FILTER.WORDS` | `{}` | Case-insensitive blocked substrings |
| `Config.WORD_FILTER.MODE` | `"censor"` | `"censor"` replaces matched text with `*`; `"block"` rejects the message |
| `Config.RATE_LIMIT.MAX_MESSAGES` | `5` | Max messages per sliding window |
| `Config.RATE_LIMIT.WINDOW_MS` | `5000` | Sliding window length (ms) |
| `Config.RATE_LIMIT.MUTE_ON_VIOLATION_MS` | `10000` | Soft auto-mute on flood violation; `0` = warn only |
| `Config.MUTE_BLOCKS` | `{"ic", "ooc", "job", "scene"}` | Channels silenced by a mute (Admin channel is never blocked) |
| `Config.MAX_MESSAGE_LENGTH` | `256` | Maximum accepted message length (server re-validates even when the client enforces it) |

#### Admin Gating

A player is a chat-admin when any one of three checks passes:

| Key | Default | Description |
|-----|---------|-------------|
| `Config.ADMIN_IDENTIFIERS` | *(example entry)* | Direct `license:`/`steam:`/`discord:` allowlist — simplest and framework-agnostic |
| `Config.ADMIN_GROUPS` | `{"admin", "superadmin", "mod"}` | Framework group values. **Note:** on RSG the bridge maps group to job name, so this checks the job — prefer `ADMIN_IDENTIFIERS` or `ADMIN_ACE` on RSG servers |
| `Config.ADMIN_ACE` | `"gfxr.chat.admin"` | ACE permission string; set to `nil` to disable ACE checks |

#### Discord Logging

| Key | Default | Description |
|-----|---------|-------------|
| `Config.WEBHOOK_URL` | `""` | Admin-action log webhook (mute/kick/clear/announce); empty = disabled |
| `Config.WEBHOOK_NAME` | `"gfxr-chat"` | Bot display name for the admin webhook |
| `Config.MESSAGE_LOG.ENABLED` | `false` | Enable full chat message logging to Discord |
| `Config.MESSAGE_LOG.WEBHOOK_URL` | `""` | Separate webhook for message log; falls back to `WEBHOOK_URL` if empty |
| `Config.MESSAGE_LOG.INCLUDE_ID` | `false` | Append the sender's identifier to each logged entry |
| `Config.MESSAGE_LOG.CHANNELS` | *(see below)* | Per-type toggle; `whisper` and `admin` are `false` by default |

**MESSAGE\_LOG channel defaults:** `say`, `me`, `do`, `try`, `shout`, `ooc`, `job`, `scene`, `announce` = `true`; `whisper` = `false` (whispers are usually private RP — enable knowingly); `admin` = `false` (staff-only). The admin **History** search also indexes whispers regardless of this Discord toggle.

---

### config/client_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.MAX_MESSAGE_LENGTH` | `256` | Client-side character cap (server re-validates) |
| `Config.COMMAND_PREFIX` | `"/"` | Prefix that triggers command/autocomplete mode in the input box |
| `Config.NOTIFY` | `nil` | Optional notify override; `nil` uses gfxr-bridge `Notify` |

#### Scene Chat (bottom-left rolling log)

| Key | Default | Description |
|-----|---------|-------------|
| `Config.SCENE_CHAT.ENABLED` | `true` | Show the bottom rolling RP log |
| `Config.SCENE_CHAT.MAX_LINES` | `8` | Visible lines before the oldest scrolls out |
| `Config.SCENE_CHAT.LINE_FADE_MS` | `12000` | Per-line fade-out delay (ms) |
| `Config.SCENE_CHAT.CHANNELS` | `{"me", "do", "try", "ooc"}` | Which command types feed the scene log |

#### 3D Floating Text (client render)

| Key | Default | Description |
|-----|---------|-------------|
| `Config.FLOATING_TEXT.ENABLED` | `true` | Render 3D text above senders' heads |
| `Config.FLOATING_TEXT.DURATION_MS` | `9000` | How long text stays above the head (ms) |
| `Config.FLOATING_TEXT.MAX_DISTANCE` | `20.0` | Stop drawing past this distance (metres) |
| `Config.FLOATING_TEXT.HEAD_BONE` | `21030` | SKEL\_Head bone index (rdr3); text is anchored here |
| `Config.FLOATING_TEXT.Z_OFFSET` | `0.08` | Metres above the head bone |
| `Config.FLOATING_TEXT.HEIGHT` | `1.05` | Fallback height above ped coords when the bone cannot be resolved |

#### Scene Markers (client render)

| Key | Default | Description |
|-----|---------|-------------|
| `Config.SCENE_MARKER.ENABLED` | `true` | Render persistent world markers |
| `Config.SCENE_MARKER.MAX_DISTANCE` | `35.0` | Stop drawing a marker past this distance (metres) |
| `Config.SCENE_MARKER.HEIGHT` | `-1.0` | Metres relative to ped coords; negative places the pin at ground level |

#### Typing Indicator (client render)

| Key | Default | Description |
|-----|---------|-------------|
| `Config.TYPING_INDICATOR.ENABLED` | `true` | Render typing bubbles and send own compose heartbeats to the server |
| `Config.TYPING_INDICATOR.HEAD_BONE` | `21030` | SKEL\_Head bone index |
| `Config.TYPING_INDICATOR.Z_OFFSET` | `0.32` | Metres above the head bone |
| `Config.TYPING_INDICATOR.MAX_DISTANCE` | `22.0` | Stop drawing past this distance (metres) |

#### Default Player Settings

These are the factory defaults loaded when no KVP data is saved. Players can change all of them in the in-game settings panel.

| Key | Default | Description |
|-----|---------|-------------|
| `visibility` | `0` | Chat visibility: `0` = Active (show when open), `1` = Always, `2` = Hidden |
| `mentionSound` | `true` | Play a sound when @mentioned |
| `font` | `"journal"` | Font preset: `"journal"`, `"quill"`, or `"clean"` |
| `lang` | `""` | Player language override; empty = use server `gfxr_locale` default |
| `timestamps` | `true` | Show clock on each chat line |
| `timeFormat` | `"24"` | `"24"` or `"12"` hour clock |
| `timeSeconds` | `false` | Include seconds in timestamps |
| `sceneChat` | `true` | Show the bottom rolling RP log |
| `sceneTimestamps` | `true` | Show clock in the scene log |
| `text3dStyle` | `"plate"` | 3D text render style: `"plate"`, `"minimal"`, `"bubble"`, `"parchment"`, or `"off"` |
| `text3dScale` | `1.0` | 3D text size multiplier |

#### Layout Defaults

| Key | Default | Description |
|-----|---------|-------------|
| `Config.LAYOUT.CHAT` | `{ x=20, y=20, w=430, h=240, scale=1.0 }` | Chat panel initial position, width, message-area height, and font scale |
| `Config.LAYOUT.SCENE` | `{ x=24, y=560, w=380, scale=1.0 }` | Scene log initial position, width, and font scale |
| `Config.LAYOUT.SNAP.ENABLED` | `true` | Snap-to-edge / snap-to-centre when dragging a section |
| `Config.LAYOUT.SNAP.THRESHOLD` | `12` | Snap activation distance (px) |
| `Config.LAYOUT.SNAP.MARGIN` | `20` | Gap kept from screen edges when edge-snapping (px) |
| `Config.LAYOUT.SNAP.CENTER` | `true` | Also snap to the horizontal and vertical screen centre |
| `Config.LAYOUT.FONT_SCALE.MIN` | `0.8` | Minimum font scale |
| `Config.LAYOUT.FONT_SCALE.MAX` | `1.4` | Maximum font scale |
| `Config.LAYOUT.FONT_SCALE.STEP` | `0.05` | Font scale slider step |

## Commands

### Player commands

| Command | Alias | Description |
|---------|-------|-------------|
| `/me <text>` | | Describe an action in the third person; 3D text above head; proximity `ME` |
| `/do <text>` | | Describe a scene or state; 3D text above head; proximity `DO` |
| `/try <text>` | | Attempt an action with a server-computed random success/fail outcome; proximity `TRY` |
| `/whisper <text>` | `/w` | Whisper to very nearby players; proximity `WHISPER` |
| `/shout <text>` | `/s` | Shout to far players; proximity `SHOUT` |
| `/ooc <text>` | | Out-of-character message; local (proximity `OOC`) or server-wide depending on `OOC_SERVERWIDE` |
| `/scene [<dur>] <text>` | | Scene narration; shows in the bottom-left rolling log AND pins a 3D world marker. Optional leading duration token: `/scene 10m text` (s/m/h units, capped at `MAX_DURATION`) |
| `/jobchat <text>` | | Job channel message; delivered to all online players in the same job |
| `/dice [sides]` | | Roll a die (default d6); result broadcast to nearby players |
| `/coin` | | Flip a coin; result broadcast to nearby players |
| `/report <text>` | | Send a report to all online admins |
| `/chatsettings` | | Open the chat settings panel directly |

### Admin commands

| Command | Description |
|---------|-------------|
| `/announce <text>` | Server-wide styled announcement; usable from console too |
| `/achat <text>` | Admin-only channel message; delivered to all chat-admins |
| `/mute <serverId> [durationSec] [reason]` | Mute a player; `durationSec` 0 or omitted = permanent |
| `/unmute <serverId>` | Remove a mute |
| `/kick <serverId> [reason]` | Kick a player |
| `/clearchat` | Clear the chat for all players |
| `/clearscene` | Remove all live scene markers |
| `/chatadmin` | Open the admin panel directly |

## Admin Panel

Chat-admins see an admin icon in the chat toolbar (or can open the panel with `/chatadmin`). The panel has three tabs:

- **Players** — searchable online player list; per-player mute (with duration picker) and kick buttons; muted badge
- **Reports** — list of open `/report` submissions with player name, text, timestamp, and a Resolve button
- **History** — search up to 500 recent in-memory messages by player name or text (newest first)

## Database

One table is used, auto-created on first start:

| Table | Purpose |
|-------|---------|
| `gfxr_chat_mutes` | Persistent player mutes (`identifier`, `muted_by`, `reason`, `created_at`, `expires_at`) |

Mutes are hydrated into memory at resource start; expired rows are pruned automatically. No other SQL is used — theme, layout, and settings live in client-side KVP.

## Net Events

These events are the public integration surface other resources can listen to:

| Event | Side | Description |
|-------|------|-------------|
| `gfxr-chat:receive` (client) | client | Fired when a rich chat line arrives; payload: `{ id, channel, type, author, authorJob, text, time, mine }` |
| `gfxr-chat:messageDeleted` (client) | client | Fired when a message is deleted; payload: `{ id }` |
| `gfxr-chat:mentioned` (client) | client | Fired when this player is @mentioned; payload: `{ from }` |
| `gfxr-chat:muted` (client) | client | Fired when mute state changes; payload: `{ muted, until_ }` |
| `gfxr-chat:clear` (client) | client | Fired when an admin clears the chat |

The upstream `chat:addMessage` and `chat:addSuggestion` events from the stock chat resource are also preserved.

## Locale

Language is set via the `gfxr_locale` convar (applied to both client and server):

```
setr gfxr_locale "en"   # en | de | pt-BR | fr | th | es | ro | tr
```

Players can additionally override their own display language in the chat settings panel without affecting other players.

| Key (selected) | en | tr | fr |
|---|---|---|---|
| `chat_muted` | You are muted. | Susturuldunuz. | Vous etes reduit au silence. |
| `chat_rate_limited` | You are sending messages too fast. | Cok hizli mesaj gonderiyorsunuz. | Vous envoyez des messages trop vite. |
| `try_success` | %s succeeds. | %s basarili oldu. | %s reussit. |
| `try_fail` | %s fails. | %s basarisiz oldu. | %s echoue. |
| `announce_prefix` | ANNOUNCEMENT | DUYURU | ANNONCE |
| `tab_job` | Job | Meslek | Metier |
| `tab_scene` | Scene | Sahne | Scene |
| `set_title` | Chat Settings | Sohbet Ayarlari | Parametres du chat |
| `vis_active` | Active | Aktif | Actif |
| `vis_always` | Always | Her Zaman | Toujours |
| `vis_hidden` | Hidden | Gizli | Cache |

All eight required languages (en, de, pt-BR, fr, th, es, ro, tr) are included. `en-GB` aliases `en`.
