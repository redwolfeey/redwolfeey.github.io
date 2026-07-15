# Flood Gamemode Changelog

This changelog records the work completed during the P0 and P1 implementation passes and the follow-up boat, buoyancy, content, and stability fixes. Items that still require live server verification remain in `P0_P1_TESTING.md`.

> Time estimates assume one developer familiar with Garry's Mod Lua. They cover implementation and basic static checks; live multiplayer testing, tuning, content installation, and unexpected debugging are additional.
> Estimates on related bullets may overlap because several bullets can describe parts of the same implementation.

## P0 - Security, Stability, and Data Safety

### Input validation and abuse prevention

- Added shared finite-number validation for cash, timer, shop, limit, and configuration inputs. *(est. 20-45 minutes)*
- Invalid `NaN`, infinity, negative, inappropriate decimal, and excessive values are rejected. *(est. 30-60 minutes)*
- Added per-player action rate limits for purchases, payments, party actions, and prop-protection messages. *(est. 1-2 hours)*
- Shop requests accept stable item IDs while retaining validated legacy numeric-index compatibility. *(est. 30-60 minutes)*
- Party acceptance now requires a valid, unexpired, server-issued invitation. *(est. 30-60 minutes)*
- Restricted Nadmod configuration changes to known settings and safe ranges. *(est. 20-45 minutes)*
- Added validity checks to reviewed delayed callbacks so disconnected players and removed entities are not reused. *(est. 20-45 minutes)*

### Player persistence

- Moved saves to `data/flood/players/<SteamID64>.txt`. *(est. 30-60 minutes)*
- Added migration from legacy `data/flood/<UniqueID>.txt` saves. *(est. 2-4 hours)*
- Added save schema version 2. *(est. 30-60 minutes)*
- Added safe defaults and logging for missing or malformed player data. *(est. 20-45 minutes)*
- Added persistence for owned skins and the equipped skin. *(est. 2-4 hours)*
- Equipped skins are restored on spawn. *(est. 20-45 minutes)*
- Added automatic saves every 60 seconds alongside disconnect and shutdown saves. *(est. 30-60 minutes)*

### Cash and transaction safety

- Added the configurable `flood_max_cash` balance cap. *(est. 30-60 minutes)*
- Added disk-backed transfer journals under `data/flood/transactions/`. *(est. 30-60 minutes)*
- Incomplete transfers are recovered during server startup. *(est. 30-60 minutes)*
- Failed transfers restore both player balances. *(est. 30-60 minutes)*
- Added dated operational logs under `data/flood/logs/`. *(est. 30-60 minutes)*
- Logs cover purchases, sales, refunds, transfers, admin changes, migrations, invalid data, missing water controllers, challenges, destroyed props, and round winners. *(est. 2-4 hours)*
- Added `fm_transactions` for viewing recent transactions. *(est. 30-60 minutes)*
- Admins can inspect another player's transactions with `fm_transactions <player>`. *(est. 30-60 minutes)*
- Updated `fm_pay` and its shop UI path to correctly handle exact player names containing spaces. *(est. 30-60 minutes)*

### Prop health, damage, sales, and refunds

- Prop health is clamped from zero to its recorded base health. *(est. 20-45 minutes)*
- Repair weapons can no longer heal beyond base health. *(est. 30-60 minutes)*
- Props are marked before sale or refund to prevent duplicate payouts. *(est. 1-2 hours)*
- End-of-round refunds use recorded purchase price and remaining-health percentage. *(est. 1-2 hours)*
- Sale and refund paths validate ownership and numeric values. *(est. 1-2 hours)*
- Purchased props record price, health, and buoyancy metadata. *(est. 2-4 hours)*
- Prop-damage rewards cannot exceed actual damage dealt to remaining health. *(est. 30-60 minutes)*
- Owners and party members cannot farm friendly prop damage while friendly fire is disabled. *(est. 1-2 hours)*
- Prop destruction records attacker and owner information and announces the event. *(est. 30-60 minutes)*

### Server resilience

- Missing water controllers are logged without stopping the entire gamemode. *(est. 2-4 hours)*
- Rounds continue without moving water when no compatible controller exists. *(est. 30-60 minutes)*
- Corrected misleading ConVar defaults and descriptions. *(est. 20-45 minutes)*
- Added safe fallbacks to shop availability validation when a shop table fails to load. *(est. 20-45 minutes)*
- Shop loading failures now produce a clear Flood error instead of crashing `pairs()`. *(est. 30-60 minutes)*

## P1 - Water, Boats, and Buoyancy

### Enhanced buoyancy

- Added `gamemode/server/sv_buoyancy.lua`. *(est. 2-4 hours)*
- Supplemental lift applies only to CPPI-owned `prop_physics` entities during flood and fight phases. *(est. 30-60 minutes)*
- Buoyancy samples five points across each prop instead of relying only on its origin. *(est. 2-4 hours)*
- Lift scales with submerged fraction, physics mass, global tuning, and per-item buoyancy. *(est. 2-4 hours)*
- Added linear and angular water damping. *(est. 30-60 minutes)*
- Frozen, held, invalid, refunded, destroyed, ownerless, and blacklisted props are skipped. *(est. 1-2 hours)*
- Welded boat angles are preserved instead of forcibly leveling every prop. *(est. 30-60 minutes)*
- Wood and plastic props receive stronger flotation defaults than metal and armor props. *(est. 20-45 minutes)*
- Buoyant shop props default to `1.65`; armor props default to `0.9`. *(est. 20-45 minutes)*
- Configured buoyancy affects Source's native physics buoyancy ratio and Flood's supplemental lift. *(est. 2-4 hours)*
- Added cached inferred model and material buoyancy values. *(est. 2-4 hours)*
- Added configurable processing intervals and force limits. *(est. 30-60 minutes)*
- Added server-side administrator debug sample points. *(est. 30-60 minutes)*
- Added map-configurable model blacklists. *(est. 1-2 hours)*
- Added minimum rise-speed recovery for deeply submerged buoyant props. *(est. 30-60 minutes)*

### Model-specific flotation fixes

- Set the HeliBomb to an explicit `2.25` buoyancy multiplier. *(est. 2-4 hours)*
- Set Wooden Shelf 2 (`models/props_c17/furnitureshelf001a.mdl`) to an explicit `3.5` multiplier. *(est. 30-60 minutes)*
- Added model-specific surface detection and recovery behavior for Wooden Shelf 2. *(est. 30-60 minutes)*
- Improved detection for props whose origin or ordinary sample points fall outside the detected water volume. *(est. 30-60 minutes)*
- Restored normal gravity when enhanced buoyancy is disabled or the active flood/fight phase ends. *(est. 2-4 hours)*
- Replaced dangerous first-contact lift impulses with smooth, acceleration-limited velocity recovery. *(est. 30-60 minutes)*
- Capped enhanced upward speed and reduced the native buoyancy-ratio ceiling to prevent props launching players. *(est. 2-4 hours)*

### Buoyancy ConVars

- `flood_buoyancy_enabled` *(est. 2-4 hours)*
- `flood_buoyancy_multiplier` *(est. 2-4 hours)*
- `flood_buoyancy_damping` *(est. 2-4 hours)*
- `flood_buoyancy_interval` *(est. 2-4 hours)*
- `flood_buoyancy_max_force` *(est. 2-4 hours)*
- `flood_buoyancy_min_rise_speed` *(est. 2-4 hours)*
- `flood_buoyancy_debug` *(est. 2-4 hours)*

### Boat boarding assist

- Added `gamemode/server/sv_boatboarding.lua`. *(est. 30-60 minutes)*
- Pressing Use (`E`) while aiming at an owned boat prop provides a controlled upward and inward boost. *(est. 30-60 minutes)*
- Boarding assistance is restricted to flood and fight phases. *(est. 1-2 hours)*
- Ownerless, refunded, distant, non-prop, and other players' props are rejected. *(est. 1-2 hours)*
- Added range, upward speed, forward speed, and cooldown controls. *(est. 30-60 minutes)*
- Added `flood_boarding_assist`, `flood_boarding_range`, `flood_boarding_up_speed`, `flood_boarding_forward_speed`, and `flood_boarding_cooldown`. *(est. 30-60 minutes)*

### Water controllers and water damage

- Added configurable water-controller target names. *(est. 30-60 minutes)*
- Added global rise and drain speed overrides. *(est. 30-60 minutes)*
- Added per-controller rise speed, drain speed, and delay configuration. *(est. 30-60 minutes)*
- Added `data/flood/maps/<map>.json` map configuration support. *(est. 30-60 minutes)*
- Added a flood progress bar to the HUD. *(est. 1-2 hours)*
- Added a ten-second phase-change warning. *(est. 20-45 minutes)*
- Water damage now scales with `WaterLevel` and accumulated exposure time. *(est. 30-60 minutes)*
- Added configurable grace time before water damage starts. *(est. 30-60 minutes)*
- Added drowning sounds, blue damage flashes, drowning messages, and out-of-world detection and logging. *(est. 1-2 hours)*
- Added `flood_water_target`, `flood_water_rise_speed`, `flood_water_drain_speed`, `flood_wh_grace`, and `flood_wh_ramp`. *(est. 30-60 minutes)*

## P1 - Rounds and Combat

- Replaced loose global round timers with `GM.RoundTimes` and accessor methods. *(est. 30-60 minutes)*
- Round state and timers synchronize once per second. *(est. 2-4 hours)*
- Timer commands use validated inputs and allow safe zero-second transitions. *(est. 30-60 minutes)*
- Build and flood phases move to reset when fewer than two players remain. *(est. 30-60 minutes)*
- Winner handling covers solo survivors, surviving parties, no survivors, and timeouts. *(est. 2-4 hours)*
- Late joiners remain excluded from active rounds. *(est. 20-45 minutes)*
- Added the `/ready` command and optional `flood_ready_required` gating. *(est. 1-2 hours)*
- Added configurable direct fight-phase PvP with `flood_pvp_enabled`. *(est. 30-60 minutes)*
- Added configurable party friendly fire with `flood_party_friendly_fire`. *(est. 1-2 hours)*
- Added per-round survival, kills, prop damage, destroyed props, and earnings statistics. *(est. 30-60 minutes)*
- Added an end-of-round summary. *(est. 30-60 minutes)*

## P2 - Current Implementation Pass

- Restricted normal building tools to axis and weld; disabled tools that spawn entities, remove paid props, alter physics, or bypass the Flood shop. *(est. 20-45 minutes)*
- Added build-phase, world-target, ownership, donator, and disabled-tool rejection messages. *(est. 30-60 minutes)*
- Prevented bundled Nadmod PP from replacing or partially initializing over another active CPPI provider. *(est. 30-60 minutes)*
- Registered map-created entities as World-owned after map initialization and cleanup, including client ownership synchronization. *(est. 1-2 hours)*
- Added configurable party size through `flood_party_max_size`, defaulting to 3. *(est. 20-45 minutes)*
- Added party leadership transfer with confirmation and server validation. *(est. 2-4 hours)*
- Added leave and rename confirmations. *(est. 30-60 minutes)*
- Confirmed party invitations expire after 30 seconds and use action rate limits. *(est. 1-2 hours)*
- Added HUD alive-player and owned-prop/limit counters. *(est. 1-2 hours)*
- Added visible and audible phase warnings at 60, 30, 10, and 5 seconds. *(est. 20-45 minutes)*
- Added permission-controlled `!round start|pause|skip|reset` administration. *(est. 30-60 minutes)*
- Added console-safe `flood_round_control start|pause|skip|reset` administration. *(est. 30-60 minutes)*
- Added replicated `flood_count_bots_as_players` testing support. Enabled bots count toward round minimums, ready requirements, low-player resets, and HUD totals. *(est. 1-2 hours)*

## P3 - Map and Content Foundation

- Added startup map validation for the recommended `fm_` prefix, supported spawn counts, and water controllers. *(est. 2-4 hours)*
- Added the server/admin `flood_validate_map` command. *(est. 1-2 hours)*
- Extended map JSON with validated per-phase timer overrides. *(est. 1-2 hours)*
- Added case-insensitive per-map shop prop blacklists. *(est. 1-2 hours)*
- Logged map configuration and validation reports. *(est. 20-45 minutes)*
- Added `MAP_AUTHORING.md` and `P3_CHANGES_AND_TESTING.md`. *(est. 30-60 minutes)*
- Added a scoped `fm_buildaboat_canal_b12wn` fix that disables `trigger_teleport` ID 501 during Flood/Get-on-your-boat and re-enables it during Reset. *(est. 30-60 minutes)*
- Added the public `FloodGameStateChanged(previousState, newState)` hook for phase-specific map behavior. *(est. 30-60 minutes)*

## P1 - Economy and Shop

### Shop safety and usability

- Added stable IDs and category names for props, weapons, and skins. *(est. 1-2 hours)*
- Server purchases use stable IDs with validated legacy numeric compatibility. *(est. 1-2 hours)*
- Added startup validation for missing prop models, skin models, and SWEP classes. *(est. 20-45 minutes)*
- Missing content is marked unavailable in the UI and rejected server-side. *(est. 20-45 minutes)*
- Added synchronization for owned weapons, owned skins, and equipped skins. *(est. 30-60 minutes)*
- Added search and alphabetical sorting to prop, weapon, and skin catalogues. *(est. 1-2 hours)*
- Added confirmation dialogs for expensive purchases. *(est. 1-2 hours)*
- Prop tooltips show ID, category, health, model mass, buoyancy, price, and maximum refund. *(est. 2-4 hours)*
- Weapon tooltips identify configured damage as Flood prop damage. *(est. 1-2 hours)*
- Shop panels identify owned, equipped, unavailable, and donator-only states. *(est. 30-60 minutes)*
- Added configurable starting cash, cash cap, prop-damage income, skin resale rate, prop refund rate, and purchase-confirmation threshold. *(est. 1-2 hours)*
- Added optional damage challenges, disabled by default. *(est. 1-2 hours)*

### Devinity addon shop import

- Read the model content supplied in the `ADDON IGNORE` directory. *(est. 30-60 minutes)*
- Added 114 standalone Devinity props to the prop shop. *(est. 2-4 hours)*
- Added 97 Devinity player models to the skin shop. *(est. 2-4 hours)*
- Excluded accessory pieces, support models, and first-person arm models. *(est. 20-45 minutes)*
- Imported wood and plastic items as buoyant props. *(est. 30-60 minutes)*
- Imported metal and heavy miscellaneous items as armor props. *(est. 30-60 minutes)*
- Assigned increased price and health to special props. *(est. 30-60 minutes)*
- Assigned separate prices to regular and seasonal player models. *(est. 30-60 minutes)*
- Generates readable item names from model filenames. *(est. 30-60 minutes)*
- Confirmed all 211 imported shop model paths are unique. *(est. 2-4 hours)*
- Fixed an accidental leading `+` that initially prevented `sh_shoplist.lua` from loading. *(est. 20-45 minutes)*

The Devinity files still need to be installed or mounted as normal Garry's Mod content. The temporary `ADDON IGNORE` directory is only the import source and is not itself a standard mounted addon location.

### Economy and shop ConVars

- `flood_starting_cash` *(est. 30-60 minutes)*
- `flood_max_cash` *(est. 30-60 minutes)*
- `flood_prop_damage_cash_multiplier` *(est. 30-60 minutes)*
- `flood_sell_multiplier` *(est. 30-60 minutes)*
- `flood_prop_refund_multiplier` *(est. 1-2 hours)*
- `flood_purchase_confirm_threshold` *(est. 30-60 minutes)*
- `flood_challenges_enabled` *(est. 30-60 minutes)*
- `flood_challenge_reward` *(est. 30-60 minutes)*
- `flood_challenge_damage_goal` *(est. 30-60 minutes)*

## Interface and Party Updates

- Added party-member identification to the HUD and scoreboard. *(est. 1-2 hours)*
- Fixed `/ready` so it operates during the waiting phase, displays its count in the correct HUD phase, and starts a round after two players ready up. *(est. 20-45 minutes)*
- Added `!ready` as an alternative ready command and enabled ready-required round starts by default. *(est. 1-2 hours)*
- Restricted the shop Admin tab and its cash/timer commands to the dedicated ULib permission node `flood admin commands`; the menu rebuilds when the player's access changes. *(est. 1-2 hours)*
- Added server-side party-name filtering with character restrictions, leetspeak normalization, repeated-character normalization, substring detection, and one-edit fuzzy matching for blocked offensive terms. *(est. 2-4 hours)*
- Replaced nickname-derived default party names with neutral numbered names so player nicknames cannot bypass party-name filtering. *(est. 2-4 hours)*
- Improved shop close behavior so cursor and keyboard focus are restored reliably. *(est. 20-45 minutes)*
- Added owned, equipped, unavailable, and donator-only visual states to shop content. *(est. 30-60 minutes)*
- Added flood progress and phase-warning HUD information. *(est. 1-2 hours)*
- Made the tools panel tolerate unnamed or partially registered spawnmenu categories instead of passing nil labels into VGUI. *(est. 20-45 minutes)*
- Prevented stale prop-protection refresh timers from calling removed panels after the shop is rebuilt. *(est. 20-45 minutes)*
- Cleared stale NADMOD control-panel references and guarded network/delayed refreshes before calling `ClearControls`. *(est. 20-45 minutes)*

## Documentation

- Expanded `README.md` with installation requirements, persistence, migration, logs, transaction journals, map JSON configuration, buoyancy tuning, boat boarding, readiness, and challenges. *(est. 2-4 hours)*
- Created and maintained `TODO.md` with P0/P1 implementation and testing states. *(est. 20-45 minutes)*
- Created `P0_P1_CHANGES.md` as the implementation summary. *(est. 30-60 minutes)*
- Created `P0_P1_TESTING.md` as the live runtime test checklist. *(est. 30-60 minutes)*
- Documented the full M9K Small Arms, Assault Rifles, and Heavy Weapons catalogue dependencies. *(est. 20-45 minutes)*

## Testing Status

- Static checks confirmed 114 addon prop paths and 97 addon player-model paths are registered. *(est. 2-4 hours)*
- All 211 imported addon paths are unique. *(est. 2-4 hours)*
- Shop-list table braces and registration order were checked after the import. *(est. 30-60 minutes)*
- Live multiplayer, physics, map, UI, persistence, economy, and performance checks remain listed in `P0_P1_TESTING.md`. *(est. 2-4 hours)*
