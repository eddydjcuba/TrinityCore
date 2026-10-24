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

`zzzz-wowforever.conf` is loaded after distributed bot defaults. It caps initial
world bots, enables companion/raid support and leaves economic automation,
automated battleground population and nonstandard raid shortcuts off. These
features remain compiled and can be evaluated separately with measured limits.

The client build remains exactly `1.60.1.70425`. Keep `realmlist.gamebuild` aligned
with the actual client; adding a row to build_info does not admit every version.

Back up auth, characters, world, hotfixes and the shared playerbot database
together before future deployments. Binary rollback alone cannot undo bot
characters, inventory or progression. Never deploy a repack over this runtime.
