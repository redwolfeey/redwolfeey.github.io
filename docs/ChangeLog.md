# Flood Gamemode Changelog

This changelog records the work completed during the P0 and P1 implementation passes and the follow-up boat, buoyancy, content, and stability fixes. Items that still require live server verification remain in `P0_P1_TESTING.md`.

## P0 - Security, Stability, and Data Safety

### Input validation and abuse prevention

- Added shared finite-number validation for cash, timer, shop, limit, and configuration inputs.
- Invalid `NaN`, infinity, negative, inappropriate decimal, and excessive values are rejected.
- Added per-player action rate limits for purchases, payments, party actions, and prop-protection messages.
- Shop requests accept stable item IDs while retaining validated legacy numeric-index compatibility.
- Party acceptance now requires a valid, unexpired, server-issued invitation.
- Restricted Nadmod configuration changes to known settings and safe ranges.
- Added validity checks to reviewed delayed callbacks so disconnected players and removed entities are not reused.

### Player persistence

- Moved saves to `data/flood/players/<SteamID64>.txt`.
- Added migration from legacy `data/flood/<UniqueID>.txt` saves.
- Added save schema version 2.
- Added safe defaults and logging for missing or malformed player data.
- Added persistence for owned skins and the equipped skin.
- Equipped skins are restored on spawn.
- Added automatic saves every 60 seconds alongside disconnect and shutdown saves.

### Cash and transaction safety

- Added the configurable `flood_max_cash` balance cap.
- Added disk-backed transfer journals under `data/flood/transactions/`.
- Incomplete transfers are recovered during server startup.
- Failed transfers restore both player balances.
- Added dated operational logs under `data/flood/logs/`.
- Logs cover purchases, sales, refunds, transfers, admin changes, migrations, invalid data, missing water controllers, challenges, destroyed props, and round winners.
- Added `fm_transactions` for viewing recent transactions.
- Admins can inspect another player's transactions with `fm_transactions <player>`.
- Updated `fm_pay` and its shop UI path to correctly handle exact player names containing spaces.

### Prop health, damage, sales, and refunds

- Prop health is clamped from zero to its recorded base health.
- Repair weapons can no longer heal beyond base health.
- Props are marked before sale or refund to prevent duplicate payouts.
- End-of-round refunds use recorded purchase price and remaining-health percentage.
- Sale and refund paths validate ownership and numeric values.
- Purchased props record price, health, and buoyancy metadata.
- Prop-damage rewards cannot exceed actual damage dealt to remaining health.
- Owners and party members cannot farm friendly prop damage while friendly fire is disabled.
- Prop destruction records attacker and owner information and announces the event.

### Server resilience

- Missing water controllers are logged without stopping the entire gamemode.
- Rounds continue without moving water when no compatible controller exists.
- Corrected misleading ConVar defaults and descriptions.
- Added safe fallbacks to shop availability validation when a shop table fails to load.
- Shop loading failures now produce a clear Flood error instead of crashing `pairs()`.

## P1 - Water, Boats, and Buoyancy

### Enhanced buoyancy

- Added `gamemode/server/sv_buoyancy.lua`.
- Supplemental lift applies only to CPPI-owned `prop_physics` entities during flood and fight phases.
- Buoyancy samples five points across each prop instead of relying only on its origin.
- Lift scales with submerged fraction, physics mass, global tuning, and per-item buoyancy.
- Added linear and angular water damping.
- Frozen, held, invalid, refunded, destroyed, ownerless, and blacklisted props are skipped.
- Welded boat angles are preserved instead of forcibly leveling every prop.
- Wood and plastic props receive stronger flotation defaults than metal and armor props.
- Buoyant shop props default to `1.65`; armor props default to `0.9`.
- Configured buoyancy affects Source's native physics buoyancy ratio and Flood's supplemental lift.
- Added cached inferred model and material buoyancy values.
- Added configurable processing intervals and force limits.
- Added server-side administrator debug sample points.
- Added map-configurable model blacklists.
- Added minimum rise-speed recovery for deeply submerged buoyant props.

### Model-specific flotation fixes

- Set the HeliBomb to an explicit `2.25` buoyancy multiplier.
- Set Wooden Shelf 2 (`models/props_c17/furnitureshelf001a.mdl`) to an explicit `3.5` multiplier.
- Added model-specific surface detection and recovery behavior for Wooden Shelf 2.
- Improved detection for props whose origin or ordinary sample points fall outside the detected water volume.
- Restored normal gravity when enhanced buoyancy is disabled or the active flood/fight phase ends.

### Buoyancy ConVars

- `flood_buoyancy_enabled`
- `flood_buoyancy_multiplier`
- `flood_buoyancy_damping`
- `flood_buoyancy_interval`
- `flood_buoyancy_max_force`
- `flood_buoyancy_min_rise_speed`
- `flood_buoyancy_debug`

### Boat boarding assist

- Added `gamemode/server/sv_boatboarding.lua`.
- Pressing Use (`E`) while aiming at an owned boat prop provides a controlled upward and inward boost.
- Boarding assistance is restricted to flood and fight phases.
- Ownerless, refunded, distant, non-prop, and other players' props are rejected.
- Added range, upward speed, forward speed, and cooldown controls.
- Added `flood_boarding_assist`, `flood_boarding_range`, `flood_boarding_up_speed`, `flood_boarding_forward_speed`, and `flood_boarding_cooldown`.

### Water controllers and water damage

- Added configurable water-controller target names.
- Added global rise and drain speed overrides.
- Added per-controller rise speed, drain speed, and delay configuration.
- Added `data/flood/maps/<map>.json` map configuration support.
- Added a flood progress bar to the HUD.
- Added a ten-second phase-change warning.
- Water damage now scales with `WaterLevel` and accumulated exposure time.
- Added configurable grace time before water damage starts.
- Added drowning sounds, blue damage flashes, drowning messages, and out-of-world detection and logging.
- Added `flood_water_target`, `flood_water_rise_speed`, `flood_water_drain_speed`, `flood_wh_grace`, and `flood_wh_ramp`.

## P1 - Rounds and Combat

- Replaced loose global round timers with `GM.RoundTimes` and accessor methods.
- Round state and timers synchronize once per second.
- Timer commands use validated inputs and allow safe zero-second transitions.
- Build and flood phases move to reset when fewer than two players remain.
- Winner handling covers solo survivors, surviving parties, no survivors, and timeouts.
- Late joiners remain excluded from active rounds.
- Added the `/ready` command and optional `flood_ready_required` gating.
- Added configurable direct fight-phase PvP with `flood_pvp_enabled`.
- Added configurable party friendly fire with `flood_party_friendly_fire`.
- Added per-round survival, kills, prop damage, destroyed props, and earnings statistics.
- Added an end-of-round summary.

## P1 - Economy and Shop

### Shop safety and usability

- Added stable IDs and category names for props, weapons, and skins.
- Server purchases use stable IDs with validated legacy numeric compatibility.
- Added startup validation for missing prop models, skin models, and SWEP classes.
- Missing content is marked unavailable in the UI and rejected server-side.
- Added synchronization for owned weapons, owned skins, and equipped skins.
- Added search and alphabetical sorting to prop, weapon, and skin catalogues.
- Added confirmation dialogs for expensive purchases.
- Prop tooltips show ID, category, health, model mass, buoyancy, price, and maximum refund.
- Weapon tooltips identify configured damage as Flood prop damage.
- Shop panels identify owned, equipped, unavailable, and donator-only states.
- Added configurable starting cash, cash cap, prop-damage income, skin resale rate, prop refund rate, and purchase-confirmation threshold.
- Added optional damage challenges, disabled by default.

### Devinity addon shop import

- Read the model content supplied in the `ADDON IGNORE` directory.
- Added 114 standalone Devinity props to the prop shop.
- Added 97 Devinity player models to the skin shop.
- Excluded accessory pieces, support models, and first-person arm models.
- Imported wood and plastic items as buoyant props.
- Imported metal and heavy miscellaneous items as armor props.
- Assigned increased price and health to special props.
- Assigned separate prices to regular and seasonal player models.
- Generates readable item names from model filenames.
- Confirmed all 211 imported shop model paths are unique.
- Fixed an accidental leading `+` that initially prevented `sh_shoplist.lua` from loading.

The Devinity files still need to be installed or mounted as normal Garry's Mod content. The temporary `ADDON IGNORE` directory is only the import source and is not itself a standard mounted addon location.

### Economy and shop ConVars

- `flood_starting_cash`
- `flood_max_cash`
- `flood_prop_damage_cash_multiplier`
- `flood_sell_multiplier`
- `flood_prop_refund_multiplier`
- `flood_purchase_confirm_threshold`
- `flood_challenges_enabled`
- `flood_challenge_reward`
- `flood_challenge_damage_goal`

## Interface and Party Updates

- Added party-member identification to the HUD and scoreboard.
- Improved shop close behavior so cursor and keyboard focus are restored reliably.
- Added owned, equipped, unavailable, and donator-only visual states to shop content.
- Added flood progress and phase-warning HUD information.

## Documentation

- Expanded `README.md` with installation requirements, persistence, migration, logs, transaction journals, map JSON configuration, buoyancy tuning, boat boarding, readiness, and challenges.
- Created and maintained `TODO.md` with P0/P1 implementation and testing states.
- Created `P0_P1_CHANGES.md` as the implementation summary.
- Created `P0_P1_TESTING.md` as the live runtime test checklist.
- Documented the full M9K Small Arms, Assault Rifles, and Heavy Weapons catalogue dependencies.

## Testing Status

- Static checks confirmed 114 addon prop paths and 97 addon player-model paths are registered.
- All 211 imported addon paths are unique.
- Shop-list table braces and registration order were checked after the import.
- Live multiplayer, physics, map, UI, persistence, economy, and performance checks remain listed in `P0_P1_TESTING.md`.
