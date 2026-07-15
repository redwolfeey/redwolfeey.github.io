# Flood Development Roadmap

This document tracks proposed features, fixes, balancing work, and maintenance for the Flood gamemode. Audit marks describe what is present in the source; the separate release checklist still requires dedicated-server and multiplayer verification.

Code-audit status (2026-07-14):

- `[x]` — implemented in the current codebase
- `[-]` — partially implemented, incomplete, or still needs hardening/testing
- `[ ]` — not found in the current codebase
- `[*]` — update as we go

## Priority Legend

- **P0** — blocks normal gameplay or risks player data/security
- **P1** — important gameplay improvement or frequently visible defect
- **P2** — valuable feature, balance change, or usability improvement
- **P3** — polish or long-term idea

## P0 — Stability and Data Safety

- [x] Validate every client-supplied purchase index and numeric argument server-side. Shop indexes, cash values, phase timers, party invitations, and prop-protection settings are constrained before use.
- [x] Reject `NaN`, infinity, inappropriate decimals, negatives, and excessive cash/timer values through shared numeric validation.
- [x] Rate-limit purchases, payments, party actions, and prop-protection network messages per player and action.
- [x] Protect cash transfers with a disk-backed transaction journal, rollback, and startup recovery.
- [x] Save players by `SteamID64` and automatically migrate legacy `UniqueID` files when they next connect.
- [x] Persist owned skins and the equipped skin and restore the equipped model on spawn.
- [x] Store schema version `2` in player save files.
- [x] Recover from missing or malformed player data with validated defaults and a structured log entry.
- [x] Save connected player data every 60 seconds in addition to disconnect and shutdown hooks.
- [x] Guard player/entity references in delayed gameplay callbacks reviewed during the P0 audit.
- [x] Clamp prop health, validate refund owners/values, and mark props before sale/refund to prevent duplicate payouts.
- [x] Make `CleanupMap` refunds one-shot and calculate them from recorded purchase price and remaining-health ratio.
- [x] Continue rounds without moving water when a map lacks a controller, with a non-halting error and structured log entry.
- [x] Correct misleading bonus, water-toggle, and normal-player prop-limit ConVar descriptions.
- [x] Write structured logs for purchases, sales/refunds, transfers, admin balance/timer changes, migrations/errors, missing water, and round winners.

## P1 — Water, Boats, and Buoyancy

### Make props float more on the water

- [x] Implement server-side enhanced buoyancy for eligible boat props in `sv_buoyancy.lua`.
- [x] Scale upward force by the fraction of five sample points currently submerged.
- [x] Scale buoyancy by physics mass, global tuning, material defaults, and per-item `Buoyancy` values.
- [x] Sample the center and four quarter-height corners so large props do not balance on their origins.
- [x] Apply configurable linear and angular damping to reduce bobbing and spinning.
- [x] Preserve construction angles and motion without forcibly leveling or freezing boats.
- [x] Restrict enhanced buoyancy to CPPI-owned `prop_physics` entities during flood/fight phases.
- [x] Skip invalid, held, frozen, refunded/destroyed, ownerless, and blacklisted props while retaining welded boat support.
- [x] Apply mass-proportional force per welded component so assemblies do not receive an arbitrary assembly multiplier.
- [-] Verify the paddle and propeller remain predictable with enhanced buoyancy during multiplayer testing.
- [x] Add `flood_buoyancy_enabled`.
- [x] Add `flood_buoyancy_multiplier`.
- [x] Add damping, interval, force-cap, and debug ConVars.
- [x] Add per-prop `Buoyancy` metadata/defaults to `sh_shoplist.lua`.
- [x] Give buoyant/wood/plastic props stronger defaults than metal/armor props.
- [x] Support a model blacklist through per-map JSON configuration.
- [x] Cache inferred model/material buoyancy multipliers.
- [-] Process buoyancy at a controlled interval; benchmark 20, 100, and 300 props during the requested server test.
- [x] Keep debug sampling server-side, with no unnecessary debug networking.
- [-] Test flotation at low, rising, and fully raised water levels.
- [-] Test single props, welded rafts, constrained boats, damaged boats, and partially destroyed boats.
- [-] Verify two players standing on a small boat do not cause extreme oscillation.
- [-] Verify boats cannot use enhanced buoyancy to fly outside water.
- [-] Verify the force cap prevents prop/player launch exploits.
- [x] Document buoyancy tuning, ConVars, per-prop metadata, blacklist, and map configuration in `README.md`.

### Water system improvements

- [x] Support multiple controllers with per-name rise/drain speeds and delays.
- [x] Allow maps to identify named controllers through `flood_water_target`/map JSON.
- [x] Expose global and per-controller rise/drain speed overrides.
- [x] Show a ten-second phase-change warning before water rises.
- [x] Show flood-phase water-rise progress on the HUD.
- [x] Ramp water damage by WaterLevel depth and accumulated exposure time.
- [x] Add configurable water-damage grace time.
- [x] Add drowning sounds, a blue damage flash, drowning messages, and world damage attribution.
- [x] Detect/log/kill active players outside valid world geometry.
- [x] Load `data/flood/maps/<map>.json` water/controller/buoyancy overrides.

## P1 — Round and Combat Correctness

- [x] Expose direct fight-phase PvP through `flood_pvp_enabled` (disabled by default).
- [x] Replace hard-coded PvP behavior with explicit phase, attacker, and party rules.
- [x] Cap damage rewards to a prop's actual remaining health.
- [x] Credit/log prop destruction and announce the attacker in the event feed.
- [x] Block owner/party prop-damage farming when friendly fire is disabled.
- [x] Add `flood_party_friendly_fire` for player and prop damage.
- [-] Exercise fire, explosions, vehicles, world damage, and physics collisions during multiplayer testing.
- [x] Scale prop health consistently from price, material, mass, or explicit shop configuration. Purchased props use the explicit `Health` value in `sh_shoplist.lua`.
- [x] Show prop health to its owner and attackers without excessive network traffic. Health is networked on the entity and shown while looking at it.
- [-] Handle last survivor, same-party survivors, no survivors, timeouts, disconnects, and simultaneous deaths; edge cases require multiplayer testing.
- [x] Prevent a late joiner from affecting winner checks during an active round. Players joining in flood/fight/reset are silently killed and cannot respawn.
- [x] Reset safely when player count drops below two during build/flood; fight winner checks handle the remaining player.
- [x] Add winner messaging plus an intermission summary with survival, kills, damage, destroyed props, and earnings.
- [x] Validate/clamp admin timer changes and handle zero-second transitions.
- [x] Replace global timer variables with `GM.RoundTimes` accessors.
- [x] Send round state only when it changes or on a reasonable sync interval. Current round state and timers are broadcast once per second.
- [x] Add `/ready`, ready-count networking, and optional `flood_ready_required` gating.

## P1 — Economy and Shop

- [x] Generate stable authoritative IDs for every prop, weapon, and skin and use them in purchase commands.
- [x] Generate validated category names and ID lookup tables while retaining legacy numeric-index compatibility.
- [x] Detect missing SWEP classes/models after map load, flag them in UI, and reject their purchase server-side.
- [x] Document the required M9K Small Arms, Assault Rifles, and Heavy Weapons packs.
- [x] Add `flood_starting_cash`.
- [-] Expose participation, winner, damage-income, sell, and refund tuning together; final balance needs playtesting.
- [x] Add and enforce `flood_max_cash`.
- [x] Prevent buying weapons/skins outside allowed round states through both UI and console commands. Server purchase hooks require waiting/build state.
- [x] Add confirmation dialogs above `flood_purchase_confirm_threshold`.
- [x] Synchronize and display owned/equipped, unavailable, and donator-only states.
- [x] Add search and alphabetical sorting to props, weapons, and skins.
- [x] Display prop ID/category/health/mass/buoyancy/price/refund information before purchase.
- [x] Label weapon `Damage` explicitly as Flood prop damage.
- [x] Add configurable skin sell and prop refund multipliers.
- [x] Record dated transaction history for purchases, sales, refunds, transfers, and admin changes.
- [x] Add optional, disabled-by-default per-round damage challenges and configurable rewards.

## P2 — Building Tools and Prop Protection

- [ ] Review the allowed tool list and remove tools that bypass shop or ownership rules.
- [-] Add clear messages when a tool is blocked and explain which phase allows it. Donator-only tools show a reason; other blocked cases do not consistently explain phase/ownership rules.
- [-] Prevent players from touching another party's boat with physgun, tool gun, paddle, or propeller. Bundled CPPI/Nadmod ownership checks cover common interactions, but party-wide behavior needs verification.
- [-] Clarify and test friend permissions in the bundled Nadmod/CPPI integration. CPPI friends and boat-owner checks exist, but their intended party interaction is undocumented and untested.
- [-] Prevent conflicting prop-protection addons from partially initializing. Nadmod detects and warns about another CPPI provider but still initializes.
- [x] Add an owner display when looking at a prop. The Nadmod client HUD displays owner, health, and price.
- [ ] Add an optional boat/assembly ownership mode for welded structures.
- [ ] Add undo support that correctly refunds shop props only once.
- [-] Add per-player cleanup controls with confirmation. Nadmod includes player/name/class cleanup commands, but confirmation is absent.
- [ ] Add stuck-prop detection and safe cleanup during reset.
- [ ] Block prop spawning inside players, map doors, water controllers, and protected zones.
- [ ] Add spawn zones or map-defined build areas.
- [ ] Add ghost previews showing whether a shop prop can be placed.
- [ ] Make prop limits visible in the HUD/shop.
- [ ] Allow server owners to configure prop limits per user group without editing Lua.

## P2 — Parties and Team Play

- [ ] Persist parties across map changes or explicitly clear them with a notice.
- [ ] Add configurable maximum party size.
- [ ] Add invite expiration and spam protection.
- [ ] Allow party leaders to transfer leadership.
- [x] Handle leader disconnects deterministically. A departing leader disbands the party and clears member state.
- [ ] Add party privacy modes: invite-only, friends, and open.
- [x] Display party members distinctly on the HUD and scoreboard. Party HUD data and party names on the scoreboard are implemented.
- [ ] Add optional shared prop permissions and shared boat ownership.
- [ ] Add party-specific damage and reward rules.
- [x] Ensure team chat cannot leak to non-members or dead/spectating players unexpectedly. Team-chat messages are intercepted and sent only to current party members.
- [x] Sanitize and length-limit party names. Names are trimmed, restricted to 3–12 characters, and reserve `No Party`.
- [-] Add kick/leave/rename confirmation and clearer status feedback. Notifications exist for these actions, but confirmation prompts are absent.

## P2 — HUD, Menus, and Accessibility

- [ ] Modernize the shop layout for common 16:9, ultrawide, and low-resolution displays.
- [-] Make all panels scale with screen size instead of relying on fixed dimensions. The HUD uses screen-relative dimensions, while VGUI panels still contain fixed sizing.
- [ ] Add a first-join help screen explaining build, flood, fight, reset, and the spawn-menu key.
- [-] Show current phase, time remaining, alive count, cash, and prop usage clearly. Phase, timers, readiness/player count, health, ammo, and cash exist; alive count and prop usage are incomplete.
- [ ] Add visible warnings at 60, 30, 10, and 5 seconds before phase transitions.
- [-] Show why purchases and tool actions are unavailable. Most purchases return chat errors; tool/ownership/phase feedback is inconsistent.
- [ ] Add a settings panel for HUD scale, colors, sounds, hit markers, and screen effects.
- [ ] Add colorblind-friendly phase and health indicators.
- [ ] Improve keyboard navigation and focus behavior in VGUI panels.
- [x] Ensure closing the shop reliably restores cursor and keyboard focus. The shop close path hides the panel, remembers cursor position, and disables the screen clicker, including rename-field handling.
- [ ] Replace deprecated VGUI controls where practical.
- [-] Improve scoreboard sorting and display ping, status, party, and user group consistently. The custom scoreboard shows these fields, but sorting and consistency still need review.
- [-] Add spectator controls and current target information to the HUD. Attack/secondary attack cycle spectator targets; dedicated HUD guidance/target details are absent.
- [ ] Add an event feed for purchases, destroyed props, drownings, winners, and disconnects.
- [ ] Localize user-facing strings through a shared language table.

## P2 — Audio and Visual Feedback

- [ ] Add distinct sounds for phase transitions and countdown warnings.
- [ ] Add water-rise ambience that scales with the current water state.
- [ ] Add effects for prop damage without spawning excessive particles/decals.
- [ ] Add a clear effect when a prop is destroyed or refunded.
- [-] Improve hit markers for prop hits versus player hits. A networked hit-marker effect exists but does not distinguish target type.
- [ ] Add optional boat wake/splash effects with a client performance setting.
- [ ] Add a winner banner and end-of-round presentation.
- [ ] Ensure effects are precached and available to clients.

## P2 — Administration and Configuration

- [-] Move gameplay settings into a documented configuration module where ConVars are insufficient. Core timers, rewards, water damage, and prop limits are ConVars; item/group configuration still requires Lua edits.
- [-] Add admin commands for starting, pausing, skipping, and resetting rounds. Admins can alter the active phase timer with `!settime`, but no explicit lifecycle commands exist.
- [ ] Add commands for inspecting and repairing player save data.
- [ ] Add a safe command to reload shop configuration between rounds.
- [-] Add permission hooks so ULX, SAM, or other admin systems can override `IsAdmin()` checks. Commands honor Garry's Mod `IsAdmin()`/user groups, but there are no Flood-specific permission hooks.
- [ ] Add console-safe versions of admin commands for dedicated-server operators.
- [ ] Use SteamID64 targeting in addition to partial player-name matching.
- [ ] Reject ambiguous partial-name matches and show matching candidates.
- [ ] Record who changed a timer or balance and the before/after values.
- [ ] Add map voting/rotation hooks restricted to compatible Flood maps.
- [ ] Add server tags/version output for easier support diagnostics.

## P3 — Maps and Content

- [ ] Write a map-authoring guide for water entities, spawn points, build zones, and safe heights.
- [-] Add a startup validator that reports missing water, spawns, or required map entities. Water-controller validation exists and halts on failure; spawn/content validation does not.
- [ ] Support map-specific overrides for phase times, water behavior, and prop restrictions.
- [ ] Add optional environmental hazards besides water.
- [ ] Add multiple water-rise patterns (steady, waves, staged, sudden surge).
- [ ] Add random round modifiers such as heavy boats, low gravity, or fast water.
- [ ] Add a lightweight content-check screen for missing models/materials/weapons.
- [ ] Review every shop model for valid physics, collision, and buoyancy behavior.
- [ ] Add curated starter prop sets for new players.

## P3 — New Gameplay Features

- [-] Add boat repair gameplay with clear costs, cooldowns, and limits. A purchasable heal-stick SWEP repairs props; its economy, ownership rules, cooldowns, and limits need review.
- [ ] Rebalance the heal stick and prevent infinite repair-income loops.
- [ ] Add objective variants such as team survival, king-of-the-raft, or cargo protection.
- [ ] Add spectator betting only if it cannot damage the core economy.
- [ ] Add round statistics and personal records.
- [ ] Add achievements for survival, building, repairs, and prop destruction.
- [ ] Add optional team rounds with shared spawn/build zones.
- [ ] Add a blueprint system for saving and rebuilding boats with ownership/cost checks.
- [ ] Add server-configurable starter loadouts.
- [ ] Add weapon restrictions or rotating weapon pools.
- [ ] Add prop rarity/tiers only if they remain understandable and balanced.

## Code Quality and Performance

- [ ] Adopt one Lua formatting style and apply it consistently to new changes.
- [ ] Replace nonstandard `!=`, `!`, and `&&` syntax when touching files where standard Lua syntax improves tooling compatibility.
- [ ] Remove unused variables, duplicate hooks, dead comments, and stale retrieved-code headers.
- [ ] Avoid global functions/state where module-local functions or `GM` fields are sufficient.
- [ ] Give hooks, timers, network strings, and ConVars consistent `Flood_`/`flood_` names.
- [x] Split large systems into focused modules with clear server/client/shared boundaries. The loader separates server, client, shared, VGUI, and SWEP files by responsibility.
- [ ] Centralize color, message, phase, and item constants.
- [-] Add validity checks inside every delayed callback. Some callbacks are guarded, but several capture players/entities without revalidation.
- [ ] Replace repeated linear shop scans with item lookup tables keyed by class/ID.
- [ ] Profile `Think`, `Tick`, HUD paint, scoreboard, and VGUI refresh paths.
- [ ] Avoid broadcasting unchanged round data every second when event-based updates suffice.
- [ ] Review all network receivers for payload size, trust boundaries, and rate limits.
- [ ] Document public hooks and extension points for server customizations.

## Testing and Release Checklist

- [ ] Establish a repeatable local dedicated-server test command/configuration.
- [ ] Add a small test map or document the canonical development map.
- [ ] Test with 0, 1, 2, and several connected players.
- [ ] Test joining and disconnecting during every round phase.
- [ ] Test a complete waiting → build → flood → fight → reset cycle.
- [ ] Test timer expiry and early winner paths.
- [ ] Test all shop purchases, resales, refunds, and insufficient-funds cases.
- [ ] Test persistence across reconnect, map change, clean shutdown, and server crash recovery.
- [ ] Test admin and payment commands with malformed, ambiguous, and hostile input.
- [ ] Test prop limits for users, donators, and admins.
- [ ] Test party create/invite/rename/kick/leave flows.
- [ ] Test missing M9K content and invalid shop entries.
- [ ] Test on at least two compatible `fm_` maps.
- [ ] Test client UI at several resolutions and UI scales.
- [ ] Profile server tick time with realistic player and prop counts.
- [ ] Run a Lua error-free soak test over many automatic rounds.
- [ ] Update `README.md` whenever setup, dependencies, commands, or ConVars change.
- [ ] Maintain a changelog and increment `GM.Version` for releases.

## Completed

- [*] Document the current gamemode, installation, round flow, ConVars, commands, persistence, and code layout in `README.md`.
