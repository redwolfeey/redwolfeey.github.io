# Flood Development Roadmap

This document tracks proposed features, fixes, balancing work, and maintenance for the Flood gamemode. Audit marks describe what is present in the source; the separate release checklist still requires dedicated-server and multiplayer verification.

Code-audit status (2026-07-14):

- `[x]` — implemented in the current codebase
- `[-]` — partially implemented, incomplete, or still needs hardening/testing
- `[ ]` — not found in the current codebase

## Priority Legend

- **P0** — blocks normal gameplay or risks player data/security
- **P1** — important gameplay improvement or frequently visible defect
- **P2** — valuable feature, balance change, or usability improvement
- **P3** — polish or long-term idea

## P0 — Stability and Data Safety

- [-] Validate every client-supplied purchase index and numeric argument server-side. Purchase indexes are resolved against server tables, but unsafe `math.floor(args[1])` calls and incomplete bounds/type validation remain.
- [-] Reject `NaN`, infinity, decimals where inappropriate, and excessive values in cash and timer commands. Cash commands reject some illegal/negative values, but timers and upper bounds remain unprotected.
- [-] Add rate limits to purchase, payment, party, and prop-protection network messages. Prop purchases have a 0.25-second cooldown; the other paths do not.
- [ ] Make cash transfers atomic so interrupted saves cannot duplicate or destroy money.
- [ ] Replace `Player:UniqueID()` persistence keys with `SteamID64` and provide a migration path.
- [ ] Persist owned skins and the equipped skin; currently only cash and weapons are serialized.
- [ ] Add schema/version information to player save files.
- [ ] Recover gracefully from missing or malformed player data instead of throwing Lua errors.
- [ ] Save player data on a periodic timer in addition to disconnect and shutdown hooks.
- [-] Audit timers and delayed callbacks for disconnected/invalid players. Some callbacks check `IsValid`, while round/loadout, weapon-ammo, prop-seller, and UI callbacks still need review.
- [-] Prevent negative prop health, invalid owners, and duplicate refunds. Refunds validate owners and remove props, but health can fall below zero and there is no explicit one-time refund marker.
- [ ] Ensure `CleanupMap` cannot refund the same prop more than once.
- [ ] Handle maps without a water controller with a clear fallback or controlled map change instead of halting the round loop.
- [ ] Fix misleading ConVar descriptions/defaults in `gamemode/init.lua`.
- [ ] Add structured server logging for purchases, payments, admin cash changes, and round winners.

## P1 — Water, Boats, and Buoyancy

### Make props float more on the water

- [ ] Design a server-side buoyancy system that supplements Source physics for eligible boat props.
- [ ] Apply upward force based on how deeply a prop is submerged rather than using a constant upward push.
- [ ] Scale buoyancy by physics mass and configurable per-model buoyancy values.
- [ ] Sample several points across large props so wide boats remain stable instead of balancing on their origin.
- [ ] Add damping to reduce violent bobbing, spinning, and repeated water-surface bouncing.
- [ ] Preserve player construction choices; do not forcibly level or freeze boats.
- [ ] Apply buoyancy to player-owned `prop_physics` entities only.
- [ ] Skip invalid, constrained, held, frozen, destroyed, and explicitly blacklisted entities where appropriate.
- [ ] Keep the system compatible with welded multi-prop boats and calculate force without multiplying it excessively across assemblies.
- [ ] Ensure the paddle and propeller tools still move buoyant boats predictably.
- [ ] Add `flood_buoyancy_enabled` for quickly enabling or disabling enhanced flotation.
- [ ] Add `flood_buoyancy_multiplier` for global lift tuning.
- [ ] Add ConVars for damping, update interval, maximum force, and optional debug visualization.
- [ ] Add per-prop buoyancy metadata to `sh_shoplist.lua` or a dedicated shared configuration file.
- [ ] Give lightweight wooden/plastic props stronger buoyancy defaults than metal or armor props.
- [ ] Add a blacklist for props that should sink or behave normally.
- [ ] Cache physics and model configuration to avoid expensive table scans every tick.
- [ ] Process buoyancy at a controlled interval and benchmark it with 20, 100, and 300 active props.
- [ ] Network only debug information that clients actually need.
- [ ] Test flotation at low, rising, and fully raised water levels.
- [ ] Test single props, welded rafts, constrained boats, damaged boats, and partially destroyed boats.
- [ ] Verify two players standing on a small boat do not cause extreme oscillation.
- [ ] Verify boats cannot use enhanced buoyancy to fly outside water.
- [ ] Verify buoyancy cannot be exploited to launch props or players at damaging velocities.
- [ ] Document tuning instructions and recommended values for map authors/server owners.

### Water system improvements

- [-] Support multiple water controllers with different travel distances and timings. All matching controllers are already opened/closed together; per-controller configuration is absent.
- [ ] Allow maps to identify water controllers by configurable target names.
- [ ] Expose water rise and drain speed settings where the map entity supports them.
- [ ] Add a short warning countdown before the water begins rising.
- [ ] Add water-level progress to the HUD.
- [ ] Make water damage ramp up with submersion depth or time submerged.
- [ ] Add configurable grace time before water starts hurting players.
- [ ] Improve water damage feedback with sound, screen effects, and clear death attribution.
- [ ] Detect players trapped below the map or inside invalid geometry after water movement.
- [ ] Add map-specific water configuration files.

## P1 — Round and Combat Correctness

- [ ] Decide whether direct PvP damage should be enabled during the fight phase and expose it as a ConVar.
- [-] Replace the current hard-coded PvP behavior with explicit, documented damage rules. Phase checks exist, but PvP is hard-coded off and several damage paths are implicit.
- [ ] Prevent damage rewards from exceeding the prop's remaining health.
- [ ] Credit prop destruction and display the attacker/owner in the kill feed or event feed.
- [ ] Prevent owners and party members from farming money by damaging friendly props.
- [ ] Add configurable friendly-fire rules for party members.
- [ ] Confirm fire, explosions, vehicles, world damage, and physics collisions follow intended phase rules.
- [x] Scale prop health consistently from price, material, mass, or explicit shop configuration. Purchased props use the explicit `Health` value in `sh_shoplist.lua`.
- [x] Show prop health to its owner and attackers without excessive network traffic. Health is networked on the entity and shown while looking at it.
- [-] Add winner handling for last survivor, timeouts, disconnects, and simultaneous deaths. Last survivor, same-party survivors, no survivors, and timeout cases exist; edge cases still need testing.
- [x] Prevent a late joiner from affecting winner checks during an active round. Players joining in flood/fight/reset are silently killed and cannot respawn.
- [ ] Handle the player count dropping below two during build/flood/fight phases.
- [ ] Add an intermission summary showing winner, survival time, damage, kills, and earnings.
- [ ] Make timer transitions robust when an admin sets a timer to zero or an invalid value.
- [ ] Replace global round timer variables with fields owned by the gamemode/round controller.
- [x] Send round state only when it changes or on a reasonable sync interval. Current round state and timers are broadcast once per second.
- [-] Add warmup/readiness controls for small private servers. The HUD contains a players-readied display, but the round controller still starts solely from the living-player count.

## P1 — Economy and Shop

- [ ] Create a single authoritative item ID for every prop, weapon, and skin.
- [ ] Separate shop data into validated categories instead of relying on positional array indexes.
- [ ] Detect missing weapon classes/models at startup and hide or flag unavailable items.
- [ ] Decide which M9K packs are officially required and document exact dependencies.
- [ ] Add configurable starting cash.
- [ ] Balance participation rewards, winner rewards, prop damage income, and refunds together.
- [ ] Add a maximum cash setting or safely support very large balances.
- [x] Prevent buying weapons/skins outside allowed round states through both UI and console commands. Server purchase hooks require waiting/build state.
- [ ] Add purchase confirmation for expensive items.
- [-] Show owned, equipped, unavailable, and donator-only states consistently. Ownership/equip and donator checks exist, but unavailable content and presentation consistency remain.
- [ ] Add search and sorting for the prop, weapon, and skin catalogues.
- [ ] Display prop health, buoyancy, mass, and refund value before purchase.
- [ ] Display weapon damage values as gameplay values rather than implying base SWEP damage.
- [ ] Add configurable sell/refund percentages.
- [ ] Add transaction history for administrators and players.
- [ ] Add optional daily/round challenges without making the base economy mandatory.

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

- [x] Document the current gamemode, installation, round flow, ConVars, commands, persistence, and code layout in `README.md`.
