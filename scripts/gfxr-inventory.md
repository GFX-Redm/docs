# RedM Inventory Script · gfxr-inventory

Slot-based inventory for RedM servers that runs on VORP, RSG and RedEM:RP through `gfxr-bridge`. The server owns every transaction: moving, splitting, giving, dropping, trading, crafting and shopping are validated server side, rate limited and written to an append-only audit log. It ships with a React interface, 8 languages, an in-game admin panel and migration commands for VORP, RSG and RedEM:RP inventories.

## Features

- Slot grid with a weight limit and per-item stack limits, drag and drop between panels, splitting, merging and item combining
- Quick transfer with Shift-click and Ctrl-click, with the amounts set per player (all, 1, 5, 10, 25 or 50)
- Category filter, search, tooltips with description, weight, rarity, durability and decay, and a selected-item card
- Hotbar on keys 1 to 5 with drag-to-assign
- Two layouts (`page` and `split`), four colour themes and a settings page with per-player preferences
- Give items or money to a nearby player, with an accept or decline request
- Ground drops that appear as props, with pickup prompts and a cleanup timer
- Secure trade between two players: both sides confirm, and a cancel or disconnect returns everything
- Search a downed, cuffed or surrendering player and take items, plus job-locked police search and confiscation into an evidence stash
- Weapons as items with serial number, ammo, components, durability and custom label, equipment slots, ammo refills by dragging onto the weapon
- Stashes (personal, job, property, shared), horse saddlebags, wagon storage and container items such as bags and chests
- Shops with buy and sell tabs, stock, restocking, tax, job and grade locks and a choice of money type
- Crafting at benches with queues, tool inputs, success chance, skills and blueprints, plus hand crafting from the inventory
- Food spoilage, tool wear and expiry rules
- Wearable items with a wear and unwear toggle (clothing hashes are optional, see Clothing)
- Death rules for dropping or wiping items, and starting items for new characters (`Config.STARTER_ITEMS`, applied through the VORP layer)
- Server-side hook API (`beforeMove`, `beforeUse`, `beforeGive`, `beforeDrop`, `beforeBuy`, `beforeSell`, `beforeCraft`, `afterAdd`, `afterRemove`, `afterUse`)
- Admin panel inside the inventory: player inventories (online and offline), item catalogue and recipe editors, shop and stash editors, drop list, searchable transaction log with CSV export, security alerts and live settings
- Eight interface languages: English, German, Portuguese (BR), French, Thai, Spanish, Romanian, Turkish

## Architecture

- `server/` is authoritative. Slot locks reject concurrent actions on the same slot, a per-player rate limit applies to every action, and distance and access are checked for stashes, horses and downed targets.
- `client/` draws the interface and forwards actions. The browser never decides what happens to an item.
- Player data (identity, money, jobs) goes through `gfxr-bridge`, which detects VORP, RSG or RedEM:RP at runtime.
- The transaction log is append-only and chained with hashes. `gfxrinv verify` checks the chain.
- 17 tables under the `gfxr_inventory_` prefix are created on first start.

## Drop-in compatibility

The manifest declares `provide 'vorp_inventory'`, `provide 'rsg-inventory'` and `provide 'redemrp_inventory'`, so other resources that call those inventories keep calling the same exports. The active compatibility layer is chosen from the framework that the bridge detects.

- **VORP:** the `vorp_inventory` exports (`addItem`, `subItem`, `getItemCount`, `getItem`, `canCarryItem`, `getUserInventoryItems`, `registerUsableItem`, `registerInventory`, `openInventory`, `closeInventory`, `setItemMetadata`, `getUserWeapons`, `createWeapon`, `subWeapon` and more), the `vorp_inventoryApi()` helper, the `vorpCore:*` events and the `vorpinventory:get_slots` callback
- **RSG:** the inventory functions that `rsg-core` calls, `Player.Functions.AddItem` and related injection, and the `rsg-inventory:server:*` events, with weight converted between grams and kilograms
- **RedEM:RP:** the `redemrp_inventory:getData` table, `getItem(...).AddItem / RemoveItem / ChangeMeta`, `getPlayerInventory`, `RegisterUsableItem:<name>` and the `redemrp_inventory:*` events

The original inventory cannot run at the same time. If it is still running, gfxr-inventory stops at start-up with a clear message.

## Dependencies

| Dependency | Required | Description |
|------------|----------|-------------|
| `gfxr-bridge` | Yes | Framework abstraction layer (VORP / RSG / RedEM:RP). Must be started before gfxr-inventory |
| `oxmysql`, `ghmattimysql` or `mysql-async` | Yes | Any one of the three, detected through the bridge |
| Node.js 18+ | Only to build the UI | Needed once, on your workstation, if `web/build/` is not included in your package |

## Installation

1. **Stop the original inventory.** Remove or comment out `ensure vorp_inventory` (or the RSG or RedEM equivalent) in `server.cfg`. Keep the folder, you will want it if you roll back.

2. **Back up your database.** Migration only reads the original tables, but you should still have a copy.

   ```bash
   mysqldump -u root -p --single-transaction your_database > backup.sql
   ```

3. **Place the resource.** The folder must be named `gfxr-inventory`, because other resources call its exports by that name.

4. **Build the interface** if your package does not include `web/build/`:

   ```bash
   cd resources/[gfx]/gfxr-inventory/web
   npm ci
   npm run build
   ```

5. **Add to server.cfg:**

   ```cfg
   ensure gfxr-bridge
   ensure gfxr-inventory

   # Interface language: en | de | pt-BR | fr | th | es | ro | tr
   setr gfxr_locale "en"

   # In-game admin panel
   add_ace group.admin gfxr.inventory allow
   ```

6. **Start the server.** `Config.AUTO_MIGRATE` is on by default, so `sql/gfxr_inventory.sql` runs by itself and creates the 17 tables. You should see:

   ```
   [gfxr-inventory] schema ready (17 statements, N new columns)
   [gfxr-inventory] v2.0.0 ready | framework: vorp | items: 137
   ```

## Migrating an existing inventory

Run a dry run first. It writes nothing and only counts:

```
gfxrinv migrate vorp dry
```

Example output: `migrate vorp: {"items":214,"inventories":1832,"slots":9471,"weapons":640,"skipped":0}`

If `skipped` is above zero, some slots did not fit because of the weight or slot limit. Raise `Config.PLAYER_SLOTS` and `Config.PLAYER_MAX_WEIGHT` and dry run again, then run it for real:

```
gfxrinv migrate vorp
```

The steps are the same for RSG and RedEM:RP, only the argument changes (`rsg` or `redem`).

### What moves from VORP

| VORP source | Becomes |
|---|---|
| `items` table | Item catalogue (`gfxr_inventory_items`) |
| `character_inventories` with `inventory_type = 'default'` | `player:<charidentifier>` |
| `character_inventories` with any other `inventory_type` | `stash:<inventory_type>` |
| `items_crafted.metadata` | Slot metadata |
| `loadout` | Weapon items, with ammo, components, serial number and custom label in metadata |

### Read this before you migrate

- **Run the migration once.** It records a `vorp` key in `gfxr_inventory_migrations` but the command does not check it, so a second run adds the items again. `Config.MIGRATE_ON_FIRST_BOOT = true` does check the table.
- **The built-in catalogue wins on name clashes.** The resource ships about 135 ready-made items (`config/items_seed.lua`). If an item name already exists, the weight, stack limit and label from your old catalogue are not imported. To keep your own definitions, empty `ItemsSeed` (`ItemsSeed = {}`) and restart before you migrate.
- **Mirroring writes to the VORP `items` table.** With `Config.MIRROR_VORP_ITEMS = true` the catalogue is mirrored one way into VORP's table so old scripts that read it with SQL keep working. On a live server set it to `false`, start, migrate, check, and only then turn it back on.
- **Slots are a new limit.** VORP only had weight. The defaults are 42 slots and 35 kg, and the per-character capacity from VORP's `characters.slots` is not imported, so every character gets the configured value.

### Check the result

```
gfxrinv audit      # orphan slots, unknown items, overweight inventories, log chain
gfxrinv verify     # verifies the audit log chain
gfxrinv save       # writes dirty inventories to disk now
```

If `audit` reports `unknownItems` above zero, those slots hold item names that are missing from the catalogue. They stay hidden from the player until you create the item in the admin panel.

### Rolling back

Remove `ensure gfxr-inventory`, restore `ensure vorp_inventory` and restart. With mirroring off, the original tables are untouched and keep working as they were at migration time. Anything players did in gfxr-inventory since then stays in the `gfxr_inventory_*` tables and is not copied back.

## Configuration

Settings live in `config/server_config.lua`, `config/shared_config.lua` and `config/client_config.lua`. Most of them can also be changed in the admin panel under Settings, and those changes are stored in `gfxr_inventory_meta` and applied over the file values at start-up.

### Capacity

| Key | Default | Description |
|---|---|---|
| `Config.PLAYER_SLOTS` | `42` | Player inventory slots |
| `Config.PLAYER_MAX_WEIGHT` | `35.0` | Player capacity in kg |
| `Config.STASH_SLOTS` / `STASH_MAX_WEIGHT` | `100` / `500.0` | Default stash size |
| `Config.HORSE_SLOTS` / `HORSE_MAX_WEIGHT` | `20` / `60.0` | Saddlebag size |
| `Config.WAGON_SLOTS` / `WAGON_MAX_WEIGHT` | `60` / `250.0` | Wagon storage size |
| `Config.SLOT_TIERS_ENABLED` | `false` | Give an ACE group a bigger bag (`SLOT_TIERS`) |
| `Config.SECURE_POCKET_SLOTS` | `2` | The last N slots are hidden from searches and kept on death |

### Safety

| Key | Default | Description |
|---|---|---|
| `Config.RATE_LIMIT_PER_SEC` | `8` | Actions per second per player |
| `Config.DUPE_GUARD` | `true` | Reject concurrent actions on one slot |
| `Config.NEW_CHARACTER_COOLDOWN` | `120` | Seconds before a new character can give or drop |
| `Config.USE_SPAM_DELAY` | `800` | Milliseconds before the same item can be used again |
| `Config.TRANSFER_LIMIT` | `{ count = 40, windowSec = 60 }` | Transfers allowed per window |
| `Config.KICK_ON_CHEAT` | `false` | Kick on detected cheating |

### World and economy

| Key | Default | Description |
|---|---|---|
| `Config.DROP_CLEANUP_MINUTES` | `15` | Ground drop lifetime |
| `Config.DROP_WHEN_DIE` | `false` | Drop items on death |
| `Config.DEATH_RULES` | all off | Wipe items, weapons, money, gold or ammo on death |
| `Config.SHOP_RESTOCK_MINUTES` | `60` | Shop restock interval |
| `Config.SEARCH_JOBS` | `sheriff, police, marshal` | Jobs allowed to search and confiscate |
| `Config.EVIDENCE_STASH` | `stash:evidence` | Where confiscated items go |
| `Config.MONEY_AS_ITEM` | `false` | Show cash and gold as items in the inventory |
| `Config.TRADE_ENABLED` | `true` | Player to player trade |
| `Config.DECAY_ENABLED` | `true` | Food spoilage and tool wear |
| `Config.WEARABLES` | `true` | Wearable item slots |

### Logging

| Key | Default | Description |
|---|---|---|
| `Config.LOG_RETENTION_DAYS` | `30` | Days of transaction log kept |
| `Config.WEBHOOK_ITEMS` | empty | Discord webhook for item actions |
| `Config.WEBHOOK_ALERTS` | empty | Discord webhook for security alerts |
| `Config.LOG_TO_GFXR_ADMIN` | `true` | Also write to the log of `gfxr-admin` when present |

### Clothing

`Config.WEARABLE_HASHES` maps a wearable item to a clothing hash, and a value of `0` means "inventory badge only". Fill it in from your own clothing catalogue if you want worn items to change the character's outfit.

### Keys

| Key | Default | Action |
|---|---|---|
| `Config.KEY_OPEN` | `I` | Open the inventory |
| `Config.KEY_HOTBAR` | `1` to `5` | Use hotbar slots |
| `Config.KEY_CANCEL` | `G` | Cancel an action |
| `Config.KEY_PROMPT` | `J` | World prompts for stashes, shops, benches and drops |

## Commands

Console commands, all under `gfxrinv`:

| Command | Description |
|---------|-------------|
| `gfxrinv migrate <vorp\|rsg\|redem> [dry]` | Migrate an existing inventory, or dry run it |
| `gfxrinv audit [charid]` | Report orphan slots, unknown items, overweight inventories and log problems |
| `gfxrinv verify` | Verify the transaction log hash chain |
| `gfxrinv save` | Write dirty inventories to the database now |
| `gfxrinv reload` | Reload the item catalogue |

## Permissions

Permission nodes are listed in `config/permissions.lua`. If `gfxr-admin` is running, its permission system is used. Otherwise grant ACE permissions:

```cfg
add_ace group.admin gfxr.inventory allow              # everything
add_ace group.mod gfxr.inventory.logs.view allow      # a single node
```

## Server exports

All exports are called as `exports['gfxr-inventory']:Name(...)`.

| Export | Signature | Description |
|---|---|---|
| `AddItem` | `(src, name, count?, metadata?, slot?) -> ok, reasonKey` | Add an item to a player |
| `RemoveItem` | `(src, name, count?, metadata?, slot?) -> ok, reasonKey` | Remove an item |
| `HasItem` | `(src, name, count?) -> boolean` | Check for an item |
| `GetItemCount` | `(src, name, metadata?) -> number` | Count items, matching metadata exactly when given |
| `GetInventory` | `(src) -> item[]` | All items of a player |
| `CanCarry` | `(src, name, count?, metadata?) -> ok, reasonKey` | Weight, slot and rule check |
| `SetItemMetadata` | `(src, slot, metadata) -> ok` | Replace slot metadata |
| `UseItem` | `(src, slot) -> { ok, reasonKey }` | Run the use flow |
| `GetWeight` | `(src) -> weight, maxWeight` | Current and maximum weight in kg |
| `RegisterItem` | `(def) -> ok` | Register a catalogue item in memory |
| `RegisterUsableItem` | `(name, fn(src, item, slot))` | Make an item usable; removed when your resource stops |
| `OpenInventory` | `(src, id, opts?) -> ok` | Open `stash:x`, `horse:x`, `wagon:x` or `container:x` |
| `OpenStash` | `(src, id, opts?) -> ok, inventoryId` | Open a stash |
| `RegisterStash` | `(def) -> ok` | Register a stash with coordinates and access rules |
| `AddItemToInventory` | `(id, name, count?, metadata?) -> ok` | Add to a stash, drop or container |
| `RemoveItemFromInventory` | `(id, name, count?, metadata?) -> removed` | Remove from a stash, drop or container |
| `GetWeapons` / `AddWeapon` / `RemoveWeapon` | see source | Weapon items |
| `RegisterHook` | `(name, fn(ctx) -> false, reasonKey?) -> id` | Register a hook |
| `RemoveHook` | `(id)` | Remove a hook |
| `LearnBlueprint` | `(charId, recipeId) -> ok` | Teach a recipe |
| `IsReady` | `() -> boolean` | True once the resource has finished loading |

### Hooks

`beforeMove`, `beforeUse`, `beforeGive`, `beforeDrop`, `beforeBuy`, `beforeSell`, `beforeCraft`, `afterAdd`, `afterRemove` and `afterUse`. Returning `false, 'err_key'` from a `before*` hook rejects the action, and the key is translated for the player when it exists in the locale.

```lua
exports['gfxr-inventory']:RegisterHook('beforeMove', function(ctx)
    -- ctx: src, from{inventoryId,slot}, to{inventoryId,slot?}, item, count, metadata
    if ctx.item == 'sheriff_badge' and ctx.to.inventoryId:find('^stash:') then
        return false, 'err_locked'
    end
end)
```

### Events

| Event | Side | Payload |
|---|---|---|
| `gfxr-inventory:server:playerLoaded` | server | `(src, charId)` |
| `gfxr-inventory:server:useItem` | server | `(src, item)` for usable items without a handler |
| `gfxr-inventory:client:open` / `close` | client | Interface opened or closed |
| `gfxr-inventory:client:stateChanged` | client | `(open)` |

The state bags `IsInvActive` and `inv_busy` are also set on the local player.

## Client exports

`Open(mode?)`, `Close()`, `IsOpen()`, `OpenStash(id)`, `OpenShop(id)`, `OpenCraft(benchId?)`, `OpenHorse(id, label, kind)`, `SearchPlayer(target)`, `RobPlayer(target)`, `TradeRequest(target)`, `ClosestPlayer(maxDist)` and `GetEquippedWeapons()`.
