# RedM Admin Menu · gfxr-admin

Framework-agnostic admin suite for RedM. One React panel with 24 sections covers player management, moderation, reports, a live map, spawning, world control, economy, whitelist, staff chat, resources, logs and security. Access is governed by 122 permission nodes in 17 groups; roles and per-staff overrides live in the database and are edited from the panel itself. Runs on VORP, RSG and RedEM:RP through `gfxr-bridge`.

## Features

- Dashboard: live server snapshot with who is online, session activity, playtime and recent staff actions
- Players: searchable online and offline lists, full character profile, notes history, every identifier a player has connected with
- Moderation: kick, warn, mute, freeze, spectate, inspect and ban (temporary or permanent) with duration presets; bans, warns and notes persist in the database
- Offline players: ban, warn or queue an action for someone who is not connected; it applies the moment they join
- Reports: players open a report with `/report`, staff claim and reply in the same thread, nothing is ever deleted
- Live map: every player on the RDR2 map, jump to any of them, manage server blips visually
- Spectate and noclip: watch a player without being seen, or move freely with an on-screen coordinate readout
- Screenshot: pull a remote screenshot of a player's screen into the panel (needs `screenshot-basic`, see Configuration)
- Spawner: 65-entry catalogue of peds, horses, wagons and props, plus raw model input and radius cleanup
- World: weather and time control (set or freeze), announcements, restart countdown
- Economy: inspect and adjust money, items and weapons; money types follow the framework (VORP has no bank, RSG has no gold)
- Character: heal, revive, respawn, clear tasks, set health, stamina, needs, job, model, outfit, open the clothing menu, view or edit the character sheet
- Whitelist: managed from the panel when the framework does not enforce its own
- Staff chat with persistent history, staff and role management, leaderboard and playtime
- Resources: start, stop and restart server resources, with a protected list the panel refuses to touch
- Logs: append-only audit trail with categories, severities, export and a tamper-evident SHA2 chain
- Security: session tracking, identifier checks, chain verification
- Quick menu: a keyboard-only side menu (Delete key by default) that never takes NUI focus, for kick, mute, teleport and money or item actions on the fly
- Discord: per-category webhooks, an optional personal webhook per staff member, and optional avatar lookup through a bot token
- Seven interface languages: English, German, French, Thai, Spanish, Romanian, Turkish

## Architecture

- `client/` runs the panel (React NUI in `web/build/`), the key poll, noclip, spectate, overhead names, the quick menu and the coordinate tools. Every action is sent to the server; the client keeps only a cached, non-authoritative copy of the permission set for drawing the UI.
- `server/` owns everything that matters: permission checks (fail-closed, unknown nodes are rejected), rate limits, confirmation tokens for destructive actions, the database, bans, reports, logs, webhooks and the framework integration through `gfxr-bridge`.
- Player data (identity, name, money, items, jobs) is read and written through the bridge, which detects VORP, RSG or RedEM:RP at runtime. Switching framework later keeps every role and staff record.
- 21 tables under the `gfxr_admin_` prefix, created on first boot (see Database). Nothing is kept only in memory: a restart never drops a ban, a report or a log row.

## Dependencies

| Dependency | Required | Description |
|------------|----------|-------------|
| `gfxr-bridge` | Yes | Framework abstraction layer (VORP / RSG / RedEM:RP). Must be started before gfxr-admin |
| `oxmysql`, `ghmattimysql` or `mysql-async` | Yes | Any one of the three; the server waits for it at boot |
| `screenshot-basic` | Optional | Remote screenshots. Not declared as a dependency on purpose, so a server without it still starts; the screenshot button simply never appears |
| `vorp_character` clothing bridge | Optional | A one-file drop-in that lets the panel open VORP's real clothing editor on a player (see Clothing menus) |

## Installation

1. Import the schema, or let it run itself. With `Config.AUTO_MIGRATE = true` (default) `sql/gfxr_admin.sql` is executed as `CREATE TABLE IF NOT EXISTS` on every boot, which is safe to repeat. To import by hand, run the file once and set `AUTO_MIGRATE` to `false`.

2. Add to `server.cfg`, after the bridge and your MySQL resource:
   ```
   ensure oxmysql
   ensure gfxr-bridge
   ensure gfxr-admin
   ```

3. Make yourself owner from the server console. The `owner` role and the `*` node can only be granted from the console, never from inside the game:
   ```
   gfxradmin bootstrap license:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
   ```
   Any identifier kind listed in `Config.ADMIN_IDENTITY_ORDER` works (`steam:…`, `discord:…`, or the bare value). While the staff table is still empty, anyone holding the `command` ACE gets temporary owner access so you cannot lock yourself out; that door closes the moment the first owner is bootstrapped.

4. Optional: set the interface language in `server.cfg` (replicated, the client reads it too):
   ```
   setr gfxr_locale "en"
   ```

5. Open the panel in game with `/admin` or the Home key.

No NUI build step is required; `web/build/` ships pre-built. If you edit `web/src/`, rebuild with `npm install && npm run build` inside `web/`.

## Opening the panel

| What | Default | Where to change |
|------|---------|-----------------|
| Chat command | `/admin` | `Config.MENU_COMMAND` in `config/client_config.lua`. Do not use `adminMenu`, `vorp_admin` owns it |
| Panel key | Home (`0x064D1698`, INPUT_FRONTEND_SOCIAL_CLUB) | `Config.MENU_CONTROL`; set to `false` for command-only. Delete (`0x4AF4D473`) is a known-free fallback |
| Quick menu key | Delete (`0x4AF4D473`) | `Config.QUICK_MENU_CONTROL`, master switch `Config.QUICK_MENU_ENABLED` |

The key is polled with `IsControlJustPressed`; RedM has no `RegisterKeyMapping`, so there is no keybind entry in the game settings. The poll only runs for staff, so non-staff players pay nothing for it. Players who are dead cannot open the panel unless their role holds `panel.open.dead`.

## Permissions

Every action is gated by a node. Roles map nodes to `true` (allow) or `false` (deny); wildcards such as `moderation.*` are expanded, and a deny entry is how "everything except x" is expressed. The catalogue lives in `config/permissions.lua` and is the single source of truth: the server rejects any node it does not know.

| Group | Nodes |
|-------|-------|
| panel (3) | `panel.open`, `panel.open.dead`, `panel.locale.change` |
| dashboard (3) | `dashboard.view`, `dashboard.economy`, `dashboard.quickactions` |
| players (12) | `players.list.online`, `players.list.offline`, `players.list.identifiers`, `players.profile.view`, `players.profile.identifiers`, `players.profile.ip`, `players.profile.notes.view`, `players.profile.notes.add`, `players.profile.notes.delete`, `players.overhead`, `players.offline.queue`, `players.offline.queue.cancel` |
| moderation (17) | `moderation.kick`, `moderation.warn`, `moderation.warn.revoke`, `moderation.ban.view`, `moderation.ban.temp`, `moderation.ban.perm`, `moderation.ban.offline`, `moderation.ban.revoke`, `moderation.template.manage`, `moderation.whitelist.view`, `moderation.whitelist.manage`, `moderation.freeze`, `moderation.spectate`, `moderation.screenshot`, `moderation.mute`, `moderation.mute.view`, `moderation.inspect` |
| economy (11) | `economy.money.give`, `economy.money.take`, `economy.money.set`, `economy.item.give`, `economy.item.remove`, `economy.inventory.view`, `economy.inventory.clear`, `economy.stats.view`, `economy.weapon.give`, `economy.weapon.remove`, `economy.weapon.clear` |
| character (14) | `character.job.set`, `character.health.set`, `character.stamina.set`, `character.needs.set`, `character.revive`, `character.respawn`, `character.clear_tasks`, `character.heal`, `character.kill`, `character.model.set`, `character.outfit.set`, `character.clothing.open`, `character.sheet.view`, `character.sheet.edit` |
| teleport (8) | `teleport.waypoint`, `teleport.coords`, `teleport.to_player`, `teleport.bring`, `teleport.return`, `teleport.saved.use`, `teleport.saved.manage`, `teleport.guarma` |
| spawn (7) | `spawn.ped`, `spawn.horse`, `spawn.wagon`, `spawn.prop`, `spawn.raw_model`, `spawn.delete.entity`, `spawn.delete.radius` |
| server (2) | `server.resources.view`, `server.resources.manage` |
| world (9) | `world.time.set`, `world.time.freeze`, `world.weather.set`, `world.weather.freeze`, `world.announce`, `world.announce.targeted`, `world.blips.view`, `world.blips.manage`, `world.restart` |
| self (8) | `self.godmode`, `self.invisible`, `self.noclip`, `self.infinite_ammo`, `self.golden_cores`, `self.heal`, `self.revive`, `self.spectate` |
| reports (7) | `reports.view`, `reports.view.all`, `reports.claim`, `reports.respond`, `reports.resolve`, `reports.reopen`, `reports.teleport` |
| logs (5) | `logs.view`, `logs.view.security`, `logs.export`, `logs.webhook.manage`, `logs.chain.verify` |
| roles (6) | `roles.view`, `roles.create`, `roles.edit`, `roles.delete`, `roles.permission.grant`, `roles.permission.revoke` |
| staff (7) | `staff.view`, `staff.add`, `staff.remove`, `staff.role.assign`, `staff.override.edit`, `staff.chat.view`, `staff.chat.send` |
| security (2) | `security.view`, `security.chain.verify` |
| tools (1) | `tools.coords` |

Some nodes are deliberately split so a role can see without touching: whitelist view and manage, mute and mute list, staff chat read and send, resources view and manage, blips view and manage, notes view, add and delete. `players.profile.ip` is separate because it reveals personal data, and `logs.webhook.manage` is high because a webhook URL lets its holder post as the server.

`moderation.screenshot` is only placed into a session when `screenshot-basic` is actually running; granting the node alone is not enough.

### Destructive actions

These nodes ask for a one-shot confirmation in the panel before the server accepts them (token lifetime `Config.CONFIRM_TOKEN_TTL`, 15 s): `moderation.ban.temp`, `moderation.ban.perm`, `moderation.ban.offline`, `character.kill`, `economy.inventory.clear`, `economy.money.set`, `spawn.delete.radius`, `world.restart`, `roles.delete`, `staff.remove`.

## Roles

Five system roles are seeded on first migration. They can be edited in the panel, and new roles can be created beside them.

| Role | Rank | What it holds |
|------|------|---------------|
| `owner` | 100 | `*` (everything). Only grantable from the console |
| `head_admin` | 80 | Everything except `roles.delete`, `staff.remove`, `world.restart`, `panel.open.dead` |
| `admin` | 60 | Dashboard, players (no IP), moderation (no permanent bans), economy, character, teleport, spawn, world time, weather, announce and blips, self, reports, `logs.view`, `tools.coords` |
| `moderator` | 40 | Online list and profiles, kick, warn, temp ban, ban list, freeze, spectate, screenshot, mute, inspect, waypoint, coords, to-player, bring and return teleports, noclip, invisible, spectate, reports (no reopen), `logs.view`, `tools.coords` |
| `helper` | 20 | Dashboard, online list and profiles, view, claim and answer reports, teleport to player, spectate |

A per-staff override (`staff.override.edit`) adds or denies single nodes for one person on top of their role.

### Who counts as staff

- `Config.ADMIN_IDENTITY` decides whether staff rights follow the account (`"user"`, survives character swaps) or the character (`"character"`).
- A staff row is matched against every identifier the connecting player has, in `Config.ADMIN_IDENTITY_ORDER` (`license`, `license2`, `fivem`, `steam`, `discord`, `xbl`, `live` by default), so a row keyed by `steam:…` works like one keyed by `license:…`.
- With `Config.FRAMEWORK_GROUPS = true` (default) the framework's own group is read as a fallback, the way `vorp_admin` does: an admin made with VORP's `/setgroup admin` is staff here too. `Config.FRAMEWORK_GROUP_ROLES` maps `superadmin`, `admin`, `moderator`, `mod` and `helper` onto the seeded roles; a group not listed grants nothing. A row in `gfxr_admin_staff` always wins over the group. Mapping a group onto a role that holds `*` is refused unless `Config.FRAMEWORK_GROUP_ALLOW_OWNER = true`.

## Console commands

Run from the server console (txAdmin or the terminal), never from a client.

| Command | Description |
|---------|-------------|
| `gfxradmin bootstrap <identifier>` | Make the identifier an owner. The only way to grant `owner` or `*` |
| `gfxradmin grant <identifier> <role>` | Add a staff member with the given role, or change their role |
| `gfxradmin revoke <identifier>` | Remove a staff member |
| `gfxradmin whoami <serverId>` | Diagnose a connected player: resolved identity, matching staff row, rank, and whether they hold `panel.open` |
| `gfxradmin selftest <serverId>` | Run the action pipeline for a player step by step (registration, permission, validation, execution) to find where a failing page breaks |
| `gfxradmin whitelist [serverId]` | Report who enforces the whitelist (framework or this resource) and, with a server id, the real admission decision for that player |

## Player command

| Command | Description |
|---------|-------------|
| `/report [type] <message>` | Open a report. Types: `bug`, `player`, `suggestion`, `other` (`Config.REPORT_TYPES`). Without a message the report form opens instead. One report per `Config.REPORT_COOLDOWN` seconds (120); while a player already has an open or claimed report, a new `/report` is appended to that thread (`Config.REPORT_SINGLE_ACTIVE`). The server stores a snapshot of players within `Config.REPORT_SNAPSHOT_RADIUS` (50 m) with the report |

This is the only entry point available to non-staff players; everything else is a server-validated panel action.

## Configuration

### config/client_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.MENU_COMMAND` | `"admin"` | Chat command that toggles the panel |
| `Config.MENU_CONTROL` | `0x064D1698` | Control hash polled to open the panel (Home). `false` disables the key |
| `Config.MENU_CONTROL_ENABLED` | `true` | Master switch for the key poll thread |
| `Config.DEBUG` | `true` in the shipped file | Prints diagnostics to F8 and registers `/admindebug`. Set to `false` before going live; when off the command does not exist at all |
| `Config.QUICK_MENU_ENABLED` | `true` | Keyboard side menu |
| `Config.QUICK_MENU_CONTROL` | `0x4AF4D473` | Quick menu key (Delete) |
| `Config.QUICK_MENU_AMOUNTS` | `{ 1, 5, 10, 50, 100, 500, 1000 }` | Money and item amounts stepped through with the arrow keys |
| `Config.QUICK_MUTE_MINUTES` | `15` | Duration of a mute issued from the quick menu |
| `Config.QUICK_MUTE_REASON` | `'Quick mute (staff menu)'` | Reason recorded for it; the server requires one and the quick menu has no text input |
| `Config.UI_SCALE` | `1.0` | Root NUI zoom (applied with CSS `zoom`) |
| `Config.UI_AUTO_SCALE` | `true` | Derive the zoom from the screen resolution (1080p = 1.0) |
| `Config.UI_ACCENT` | `"blood"` | Panel accent: `"blood"`, `"brass"` or `"sage"` |
| `Config.PLAYERLIST_REFRESH` | `5000` | ms, online list refresh while the Players tab is open |
| `Config.COORDS_REFRESH` | `100` | ms, live coordinate tool tick while the Tools tab is open |
| `Config.NOCLIP_SPEEDS` | `{ 0.1, 0.5, 1.0, 3.0, 8.0, 20.0 }` | Six speed steps |
| `Config.NOCLIP_DEFAULT_SPEED` | `3` | Starting step index |
| `Config.NOCLIP_SENSITIVITY` | `1.0` | Mouse look multiplier |
| `Config.NOCLIP_VISIBLE` | `false` | Stay visible to others while noclipping |
| `Config.NOCLIP_CONTROLS` | table | Control hashes for up, down, speed up and down, exit, movement, sprint and boost (Space, Ctrl, Page Down, Page Up, Esc, WASD, Shift, Alt) |
| `Config.NOCLIP_BOOST` | `{ SPRINT = 3.0, FAST = 6.0 }` | Speed multipliers while Shift or Alt is held |
| `Config.OVERHEAD_NAMES` | `false` | Draw player names overhead (staff with `players.overhead`) |
| `Config.OVERHEAD_BLIPS` | `false` | Draw player blips on the minimap |
| `Config.OVERHEAD_DISTANCE` | `200.0` | Metres. Bounded by the game's streaming range; the Map tab shows everyone server-wide instead |
| `Config.TELEPORT_FADE` | `400` | ms screen fade around a teleport |
| `Config.Notify` | `nil` | Optional `function(message, type)` to route notifications through another resource; `nil` uses the bridge's Notify |

### config/shared_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.DEBUG` | `false` | Debug logging; also honours the `gfxr-admin-debugMode` convar |
| `Config.MONEY_TYPES` | per framework | Currency fields the Economy tab offers: VORP cash, gold, rol; RSG cash, bank, bloodmoney; RedEM:RP cash, gold, bank. `MONEY_TYPES_DEFAULT` is used when detection fails |
| `Config.RESPAWN_LOCATION` | Valentine-area coords | Where `character.respawn` sends a player (`revive` revives in place) |
| `Config.TOWN_PRESETS` | table | Named teleport targets shown in the Teleport tab |
| `Config.MAP_TILE_URL`, `MAP_TILE_ZOOM`, `MAP_TILE_COLS`, `MAP_TILE_ROWS` | Rockstar Social Club tiles, zoom 3, 6x5 | Tile source for the Map tab. Host your own if the NUI has no internet access |
| `Config.MAP_BOUNDS` | measured | World to map calibration; the width to height ratio must stay 1.2 to match the tile grid |
| `Config.GUARMA_COORDS`, `Config.GUARMA_ZONE_NAME` | Guarma landing point | Target of `teleport.guarma` |
| `Config.WEATHER_TYPES` | 22 names | Weather list offered by the World tab |
| `Config.LOG_CATEGORIES` | `player, admin, server, security, economy, system` | Log categories shared by the server and the Logs filter |
| `Config.LOG_SEVERITIES` | `info, warn, critical` | Log severities |

### config/server_config.lua

| Key | Default | Description |
|-----|---------|-------------|
| `Config.DB_PREFIX` | `"gfxr_admin_"` | Table prefix. Changing it also means editing `sql/gfxr_admin.sql` |
| `Config.AUTO_MIGRATE` | `true` | Run the schema on boot as `CREATE TABLE IF NOT EXISTS` |
| `Config.ADMIN_IDENTITY` | `"user"` | Staff rights follow the account (`"user"`) or the character (`"character"`) |
| `Config.ADMIN_IDENTITY_ORDER` | `license, license2, fivem, steam, discord, xbl, live` | Identifier kinds a staff row may be keyed by, most preferred first |
| `Config.OWNER_BOOTSTRAP_CONSOLE_ONLY` | `true` | `owner` and `*` only from the console |
| `Config.ACE_FALLBACK` | `true` | Temporary owner access for the `command` ACE while the staff table is empty |
| `Config.FRAMEWORK_GROUPS` | `true` | Also read staff status from the framework's group |
| `Config.FRAMEWORK_GROUP_SOURCE` | `"user"` | `"user"` reads `users.group`, `"character"` reads `characters.group` |
| `Config.FRAMEWORK_GROUP_ROLES` | superadmin, admin, moderator, mod, helper | Framework group to role name |
| `Config.FRAMEWORK_GROUP_ALLOW_OWNER` | `false` | Allow mapping a group onto a role holding `*` |
| `Config.SERVER_LOGO` | `""` | Sidebar logo file in the server root; `false` uses the panel's own logo |
| `Config.TRAFFIC_DEMO` | `false` | Fill the Traffic page with sample data for screenshots. Marked DEMO on screen |
| `Config.STORE_IP` | `true` | Store IP addresses at all (GDPR / KVKK switch). `false` never writes an IP anywhere and ignores `BAN_MATCH_IP` |
| `Config.RATE_LIMITS` | token buckets per node pattern | Longest matching pattern wins; `false` disables. Sized so normal staff work never hits them |
| `Config.RATE_LIMIT_STRIKES` | `false` | Count rate-limit hits as abuse strikes |
| `Config.ABUSE_STRIKE_LIMIT` | `5` | Consecutive permission or validation violations before the panel locks for that session and owners are notified |
| `Config.DESTRUCTIVE_NODES` | list above | Nodes that need a confirmation token |
| `Config.CONFIRM_TOKEN_TTL` | `15000` | ms, confirmation token lifetime |
| `Config.SELF_ACTION_LOCK` | all `false` | Forbid heal, revive, money or item actions on oneself |
| `Config.MAX_MONEY_PER_ACTION` | `10000000` | Validation ceilings; also `MAX_ITEM_PER_ACTION` 1000, `MAX_BAN_SECONDS` 10 years, `MAX_REASON_LENGTH` 512, `MAX_COORD` 20000, `QUERY_LIMIT_MAX` 100 |
| `Config.BAN_MATCH_IDENTIFIERS` | `license, discord, steam, live, xbl` | Identifier kinds linked to a ban and matched on connect |
| `Config.BAN_MATCH_IP` | `false` | Match bans on IP too (shared IPs cause collateral bans) |
| `Config.BAN_MESSAGE_TEMPLATE` | multi-line | Kick text for banned players. Placeholders `%reason%`, `%expires%`, `%banid%`, `%admin%` |
| `Config.BAN_TEMPLATES` | rdm, vdm, fail_rp, metagaming, toxicity, exploiting, cheating, ban_evade | Duration presets; `seconds = 0` is permanent. Also editable in the panel |
| `Config.WARN_AUTO_ESCALATE` | `{ [3] = 86400, [5] = 604800 }` | Auto-ban on the Nth active warning; `false` disables |
| `Config.DISCORD_BOT_TOKEN` | `''` | Optional bot token for player avatars. Never sent to clients; without it Discord's default avatar is shown |
| `Config.BLIP_SPRITES` | 32 verified names | Blip icons the panel may save; the server validates against this list |
| `Config.WHITELIST_MODE` | `'auto'` | Who enforces the whitelist: `'auto'` (the framework if it really does, else this resource), `'framework'`, `'own'` |
| `Config.WHITELIST_ENFORCE` | `false` | Initial state of the resource's own whitelist; later toggles are stored in the database |
| `Config.PROTECTED_RESOURCES` | bridge, oxmysql, cores, session and spawn managers | Resources the panel refuses to stop. The panel itself is always protected |
| `Config.STAFF_CHAT_RETENTION_DAYS` | `30` | Days of staff chat kept; `0` keeps forever |
| `Config.REPORT_COOLDOWN` | `120` | Seconds between reports per player |
| `Config.REPORT_SINGLE_ACTIVE` | `true` | A new `/report` appends to the player's open report |
| `Config.REPORT_SNAPSHOT_RADIUS` | `50.0` | Metres, nearby players stored with a report |
| `Config.REPORT_TYPES` | `bug, player, suggestion, other` | Report types |
| `Config.LOG_DEATHS` | `true` | Log deaths and kills (player on player as `warn`, environment as `info`) |
| `Config.LOG_SESSIONS` | `true` | Log joins and leaves |
| `Config.LOG_RETENTION_DAYS` | `0` | `0` keeps forever; otherwise rows older than N days move to `gfxr_admin_logs_archive`. Rows are moved, never deleted |
| `Config.LOG_CHAIN` | `true` | SHA2 chain hash on every log row, verifiable from the Security tab |
| `Config.CLOTHING_MENUS` | VORP, RSG, RedMRCPClothing entries | Ways to open a clothing menu on a player, tried in order; the first installed one is used |
| `Config.WEBHOOKS` | all `""` | Discord webhook per log category (`default`, `player`, `admin`, `server`, `security`, `economy`, `system`). Empty means database only |
| `Config.WEBHOOK_PER_ADMIN` | `true` | Also mirror to the personal webhook stored on the acting staff member |
| `Config.WEBHOOK_USERNAME` | `"GFX Admin"` | Webhook display name; `WEBHOOK_COLORS` sets embed colours per severity |
| `Config.WEBHOOK_BATCH_MS` | `1500` | Batching window against Discord rate limits |
| `Config.RESTART_COMMAND` | `false` | Console command run when a restart countdown ends; `false` only announces |
| `Config.ANNOUNCE_DURATION_MS` | `30000` | How long an announcement stays on screen |
| `Config.RESTART_COUNTDOWNS` | `300, 120, 60, 30, 10, 5, 4, 3, 2, 1` | Seconds remaining at which a countdown announcement fires |
| `Config.PLAYTIME_TICK` | `60` | Seconds between playtime flushes to the database |
| `Config.SPAWN_LIMIT_PER_ADMIN` | `25` | Live spawned entities per staff member |
| `Config.DELETE_RADIUS_MAX` | `100.0` | Metres, ceiling for radius cleanup |
| `Config.SCREENSHOT_ENABLED` | `false` | Remote screenshots. Off by default because `screenshot-basic` returns a black frame on some RedM installs; turn on where capture is known to work |
| `Config.SCREENSHOT_ENCODING` | `'jpg'` | `'jpg'` or `'png'` |
| `Config.SCREENSHOT_QUALITY` | `0.6` | JPEG quality |
| `Config.SCREENSHOT_TIMEOUT_MS` | `15000` | Wait for the client before giving up |
| `Config.SCREENSHOT_SAVE` | `false` | Also write the image to disk under `SCREENSHOT_DIR` (`'screenshots'`) |
| `Config.SCREENSHOT_UPLOAD` | `'server'` | `'server'` uploads to the server's own HTTP endpoint; `'fivemanage'` uploads straight to Fivemanage and needs `FIVEMANAGE_TOKEN` (a media token, which the target's client will see) |
| `Config.INSPECT_NEARBY_RADIUS` | `60.0` | Metres, "nearby players" in a state snapshot |
| `Config.CHARACTER_SHEET_EDIT` | `false` | Let the panel edit name, nickname, age, gender and XP. Off by default because these fields are the player's identity; `character.sheet.view` is enough to read them |
| `Config.JOB_LIST` | `{}` | Jobs offered in the job dropdowns. Empty lists only jobs currently in use on the server; accepts `{ "sheriff" }` or `{ sheriff = "Sheriff" }` |

## Clothing menus

`character.clothing.open` opens the target's clothing editor through whatever is installed, in the order of `Config.CLOTHING_MENUS`: VORP's real editor through the bridge file below, VORP's saved outfits export as a fallback, `rsg-appearance`, or `RedMRCPClothing`. Entries for resources whose source could not be verified are left commented out; add the client event name of your own clothing resource there.

`vorp_character` does not expose its clothing editor, so a one-file bridge is shipped in `integrations/vorp_character/gfxr_clothing_bridge.lua`:

1. Copy the file into `vorp_character/`.
2. Add `'gfxr_clothing_bridge.lua'` to the `client_scripts` block of `vorp_character/fxmanifest.lua`, after its other client files.
3. Run `refresh`, then `restart vorp_character` (a manifest change is not picked up by `restart` alone).

The file only triggers `vorp_character`'s own `PrepareClothingStore` flow, so it survives framework updates; only the manifest line has to be re-added.

## Database

Tables are created under the `gfxr_admin_` prefix by `sql/gfxr_admin.sql`; a few newer ones are added by the resource's own forward migrations on boot.

| Table | Holds |
|-------|-------|
| `roles`, `role_permissions` | Roles and their node grants and denies |
| `staff`, `staff_permissions` | Staff members, their role, and per-person overrides |
| `identifiers` | Every identifier a player has connected with; `ip` stays NULL when `STORE_IP` is off |
| `bans`, `warns`, `ban_templates` | Moderation records and duration presets |
| `reports`, `report_messages` | Report threads; never deleted, only resolved or rejected |
| `notes`, `offline_queue` | Staff notes on players and actions waiting for an offline player |
| `locations` | Saved teleport locations |
| `sessions`, `traffic` | Join and leave sessions, and the samples behind the Traffic page |
| `logs`, `logs_archive` | The append-only audit trail and its archive |
| `whitelist` | The resource's own whitelist, used only when the framework has none |
| `staff_chat` | Staff chat history |
| `blips` | Map markers, visible to everyone or to staff only |
| `mutes` | Text and voice mutes |
| `meta` | Runtime switches such as the whitelist state |

## Server exports

### HasPermission

```lua
exports['gfxr-admin']:HasPermission(source, node) -- boolean
```

Authoritative permission check for a connected player against a canonical node from `config/permissions.lua`.

### IsStaff

```lua
exports['gfxr-admin']:IsStaff(source) -- boolean
```

### GetStaffRole

```lua
exports['gfxr-admin']:GetStaffRole(source) -- { name, label, rank, color } or nil
```

### GetStaffNodes

```lua
exports['gfxr-admin']:GetStaffNodes(source) -- string[] with wildcards already expanded
```

### IsBanned

```lua
exports['gfxr-admin']:IsBanned(identifiers) -- ban row or nil
```

`identifiers` is one identifier string or a table such as `{ license = '...', discord = '...' }`.

### Log

```lua
exports['gfxr-admin']:Log({
  category = 'admin', action = 'my-script:something', severity = 'info',
  actor = source, actorName = 'Name', target = nil, targetName = nil,
  detail = 'free text', result = 'ok',
}) -- boolean
```

Writes into the same searchable, chain-hashed audit trail as the panel, so the whole suite shares one log.

### GetOnlineStaff

```lua
exports['gfxr-admin']:GetOnlineStaff() -- { [src] = { name, role, rank } }
```

### IsMuted

```lua
exports['gfxr-admin']:IsMuted(source, kind) -- boolean; kind is 'text', 'voice' or nil for either
```

### GetMute

```lua
exports['gfxr-admin']:GetMute(source) -- { text, voice, reason, by, expiresAt } or nil
```

## Client exports

### IsStaff

```lua
exports['gfxr-admin']:IsStaff() -- boolean
```

### HasPermission

```lua
exports['gfxr-admin']:HasPermission(node) -- boolean
```

Cached, UI-only check. Not authoritative; the server decides.

### IsMenuOpen

```lua
exports['gfxr-admin']:IsMenuOpen() -- boolean, true while the panel has NUI focus
```

### IsVoiceMuted

```lua
exports['gfxr-admin']:IsVoiceMuted() -- boolean
```

Voice mute only; a text mute is not included. `gfxr-hud` uses this to draw a red muted state on its voice indicator when gfxr-admin is running.

## Locale

Seven languages ship in `config/locale.lua`: `en`, `de`, `fr`, `th`, `es`, `ro`, `tr`. Select one with the replicated convar `setr gfxr_locale "tr"` in `server.cfg`; an unknown value falls back to English. Staff holding `panel.locale.change` can also switch the panel language from inside it.
