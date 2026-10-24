# WOWForever integration

Integration branch: `codex/wowforever-bots`, fork `eddydjcuba/TrinityCore`.
Baseline tag: `wowforever-before-bots-2026-10-24` (`367ac384fa20`).
Upstream Forever Bots: `27a3957d188c393772211c5707844f0489eb15fa`.
Upstream Forever: `91d49b230f`.

Build with MSVC 2022 x64, Boost 1.84, OpenSSL 3.5, MySQL 8.4,
`SCRIPTS=static`, `BUILD_PLAYERBOT_V2=1`, configuration RelWithDebInfo.
Keep database credentials and runtime configuration outside Git.

## Pull request review, 2026-10-24

Eight open PRs were examined using metadata, changed-file lists and patches.
Their inclusion here is a local integration decision, not an upstream approval.

| PR | Decision | Reason |
| --- | --- | --- |
| [120](https://github.com/advocaite/TrinityCore/pull/120) | Integrated | Four explicit class masks for Zephras quests. Narrow SQL change; verify masks in copied world DB. |
| [114](https://github.com/advocaite/TrinityCore/pull/114) | Integrated | Missing vendor ingredients and preserve-item objective handling for Traditions of the Bluff. Source author reports quest completion; local gameplay still requires a client test. |
| [113](https://github.com/advocaite/TrinityCore/pull/113) | Deferred | Pet ability initialization and teaching alter several combat/training paths. Author explicitly lists functionality as untested. Validate actual tame-family/rank data and hunter training before adopting. |
| [108](https://github.com/advocaite/TrinityCore/pull/108) | Excluded | Targets master, exceeds GitHub's 300-file diff limit. Not a small Forever crash fix; cannot safely merge it based on its title. |
| [79](https://github.com/advocaite/TrinityCore/pull/79) | Deferred | Adds reopen/reset to all chest-type objects, clears live loot and returns early in loot release. Needs mining vs ordinary-chest and multiplayer-loot lifecycle tests before changing resource generation. |
| [70](https://github.com/advocaite/TrinityCore/pull/70) | Deferred | Full-name mail lookup depends on consistent cache/full-name semantics. Review together with 68; test ambiguous recipients and surname normalization. |
| [68](https://github.com/advocaite/TrinityCore/pull/68) | Deferred | Changes the character uniqueness constraint, name cache and character packets. Requires surname-aware end-to-end tests on the exact client build before a character-schema migration. |
| [67](https://github.com/advocaite/TrinityCore/pull/67) | Deferred | Large CLI rewrite, Windows explicitly untested, plus unrelated collision edits. Production console is disabled, so this is not a current blocker. |

`zzzz-wowforever.conf` is loaded after distributed bot defaults. The full profile
uses a fixed fleet target of 100, companions/raid support, one seeded battleground,
ambient BG queuing, guilds, session rotation, trade and neutral auction-house
automation. Fixed population and proximity spawning are mutually exclusive, so
World.Enable=0 is intentional in this mode. The root playerbot.conf must set the
same TotalTarget=100 because it is loaded after the conf.d files.
The World.RssBudgetMb setting only governs the inactive proximity mode; it is
not a process-wide memory ceiling. Monitor the fixed fleet's measured footprint.
Housing and arena seeding stay off for this Classic realm. Ordinary raid
requirements and spell cast timings remain unchanged.

The previous profile hit its 6000 MB proximity threshold during roster creation.
The full profile was observed creating a WSG match, more than 80 online bots,
auction listings and guild founder candidates. These are live startup observations,
not complete validation of each dungeon, raid, profession or guild workflow.
Retail medallion teaching is now guarded by the spell store lookup to avoid
requesting spell208682 on Classic clients where it does not exist.

The client build remains exactly `1.60.1.70425`. Keep `realmlist.gamebuild` aligned
with the actual client; adding a row to build_info does not admit every version.

## Local validation

Windows RelWithDebInfo compiled successfully with all 13 PlayerbotV2 libraries.
Copied auth/characters/world/hotfixes and a separate shared database were used
for startup/migration testing. Quest masks from 120 were verified in the copy.
The quest-hub startup scan originally remained busy beyond 140 seconds; keeping
TerrainInfo references for the duration of the scan reduced it to 5.624 seconds
for 146 hubs in this environment, without replacing terrain-derived zone IDs.

Through loopback-only RA, `playerbot spawn 2` generated one valid bot and
correctly rejected a reserved name, Champion. The valid bot entered the world;
the status command reported one bot/session/snapshot, five executed intents and
zero exceptions. `server exit` shut down the test runtime cleanly. The copied
admin's game-account password was set only in test auth for RA validation;
production Battle.net credentials were not changed.

This is a startup/session smoke test, not evidence of completed raids, correct
class balance or full Blizzard fidelity. Gameplay with the exact client remains
the next validation step. Names rejected by the core must not bypass its filter.

Back up auth, characters, world, hotfixes and the shared playerbot database
together before future deployments. Binary rollback alone cannot undo bot
characters, inventory or progression. Never deploy a repack over this runtime.
