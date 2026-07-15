# Regeneration Project Notes

## Purpose

This folder contains a single-file browser RPG, `reincarnation_rpg_v2_no_duplicate_ids.html`, plus external assets under `assets/`.

The game is a Korean reincarnation RPG. The player hunts monsters, levels up, fights staged bosses, dies/rebirths into different races, and eventually reaches a final clear screen.

## Main Files

- `reincarnation_rpg_v2_no_duplicate_ids.html`
  - Main UI, CSS, rendering, combat logic, state management, and debug tools.
  - Loads balance data from `assets/balance_data.js`.
- `assets/balance_data.js`
  - Externalized balance/config data.
  - Contains `window.GAME_BALANCE` with:
    - `MAX_REST`
    - `RACES`
    - `MONSTERS`
    - `TRAITS`
    - `BOSSES`
- `assets/extracted/`
  - Extracted image assets from the original inline/base64 HTML.
  - Includes boss sprites and mob sprite.
- `Clear_img.png`
  - Final clear screen image. Must remain next to the HTML file unless the HTML path is updated.

## Current Important Settings

- `DEV_MODE` is currently `true` in the HTML.
- Debug controls are hidden unless `DEV_MODE` is true.
- Debug features include:
  - stage skip
  - boss instant kill
  - boss instant kill during active boss combat
- Before publishing, consider switching `DEV_MODE` back to `false`.

## Asset Paths Required For Deployment

Keep this structure when uploading to GitHub/GitHub Pages or similar static hosting:

```text
/
├─ reincarnation_rpg_v2_no_duplicate_ids.html
├─ Clear_img.png
└─ assets/
   ├─ balance_data.js
   └─ extracted/
      ├─ asset_00.png
      ├─ asset_01.png
      ├─ ...
      ├─ mob_sprite.png
      ├─ boss_sprite_01.png
      ├─ boss_sprite_02.png
      └─ boss_sprite_03.png
```

The game avoids `fetch()` for balance data so it can work from static hosting and usually from direct local file opening as well.

## Recent Work Summary

- Boss status effects:
  - curse now reduces outgoing player damage.
  - frozen turn now skips player action and lets the enemy act.
- Mobile layout:
  - uses viewport/safe-area fixes and responsive combat sizing.
- Final clear state:
  - final boss victory shows a clear panel using `Clear_img.png`.
- Asset extraction:
  - large inline base64 assets were moved into `assets/extracted/`.
  - HTML should no longer contain large base64 blobs.
- Debug mode:
  - `DEV_MODE` flag added.
  - debug skip and instant boss kill controls added.
- Shared traits:
  - `guard` was added as the 11th trait.
  - It reduces incoming damage by 10%, or by an additional 10% while defending.
  - `magic` now boosts player attacks based on the player's attack style (`race.canMagic`), not the enemy type.
  - `instinct` now boosts outgoing melee-style player attacks and increases all incoming damage by 10%.
  - `bossKill` display text says "damage dealt to bosses +50%" to avoid confusing it with boss attack power.
- Vampire race and day/night:
  - `vampire` was added as a reincarnation race.
  - Vampire can appear in any boss rebirth pool only if the player died while holding the common `vampire` trait.
  - Current vampire rebirth weight is 50, so most boss pools give vampire about a 33.3% chance when the condition is met.
  - Day/night state is stored on `state.timeOfDay` with `state.timeTick`.
  - Time advances on valid combat actions and rest; every 3 ticks toggles day/night.
  - Vampire race has day attack penalty and night attack bonus.
  - Vampire normal attacks cost 5% max HP, but cannot reduce the player below 1 HP by cost alone.
  - Vampire racial leech and common `vampire` trait leech do not stack additively; `getVampireLeechRate()` uses the highest applicable rate.
  - Current leech rates:
    - common `vampire` trait: 30%
    - vampire race daytime: 15%
    - vampire race nighttime: 45%
    - vampire skill `피의 갈증`: at least 25% daytime, 55% nighttime
- Skeleton balance:
  - Bosses now have `attackType` in `assets/balance_data.js`.
  - Skeleton physical resistance only applies to physical boss attacks.
  - Skeleton magic weakness now applies to magical boss attacks.
  - Skeleton fear affects normal enemies by 20% and boss attacks by 15%.
- Poop poison:
  - `독액` applies 4-turn poison.
  - Poison damage is `floor(maxEnemyHp * poisonTick * 0.04)`, so normal ticks are 4%, 8%, 12%, and 16% of enemy max HP.
  - If the skill cooldown is ready while poison is still active, the skill button changes to `독 증폭`.
  - `독 증폭` immediately deals the next poison tick, then reapplies a fresh 4-turn poison if the enemy survives.
  - This is intended to create a choice between direct attack damage plus the pending poison tick, or spending the action to front-load and refresh poison.
- Combat async hardening:
  - combat uses `combatSeq`/`token`.
  - delayed combat callbacks use `scheduleCombat()` to avoid stale timers affecting new combats.
- Balance data externalization:
  - `RACES`, `MONSTERS`, `TRAITS`, `BOSSES`, `MAX_REST` moved to `assets/balance_data.js`.
- Boss 3 sprite:
  - `boss_sprite_03.png` is used by Iron Colossus.
  - Current config uses first frame only with `spriteFrameSequence:[0]` and `spriteBreathe:true` because multi-frame playback caused horizontal sliding.
- Boss sprites:
  - All bosses now have `boss_sprite_01.png` through `boss_sprite_10.png` mapped in `assets/balance_data.js`.
  - Bosses 4 through 10 use single-image sprites with `spriteFrames:1`, `spriteFrameSequence:[0]`, and `spriteBreathe:true`.
  - Bosses 1 and 2 still use their original sprite-sheet animation setup.
- Monster sprites:
  - Normal monsters can now define optional sprite fields just like bosses.
  - `문블룸 페어리` currently uses `assets/extracted/mob_2.png` as a single-image breathing sprite.
- Boss 10 pattern:
  - Eclipse Queen now uses a phase-based mixed special pattern in `applyBossAttack()`.
  - See the dedicated notes below before changing final boss balance.

## Boss Sprite Rendering Notes

Boss sprites are rendered on `#enemy-sprite-anim` canvas.

Relevant boss config fields in `assets/balance_data.js`:

- `sprite`
- `spriteFrames`
- `spriteFrameWidth`
- `spriteFrameHeight`
- `spriteCropX`
- `spriteCropY`
- `spriteCanvasWidth`
- `spriteCanvasHeight`
- `spriteDisplayWidth`
- `spriteDisplayHeight`
- `spriteInterval`
- `spriteFrameSequence`
- `spriteBreathe`

For `boss_sprite_03.png`, the current image is `436x125`.

Current Iron Colossus approach:

- `spriteFrames: 5`
- `spriteFrameWidth: 87.2`
- `spriteFrameHeight: 125`
- `spriteCropX: 1.5`
- `spriteFrameSequence: [0]`
- `spriteBreathe: true`

Reason: all-frame playback visually moved left/right, so the stable first frame is drawn and the canvas itself breathes via CSS.

## Boss 10 Pattern Notes

Boss 10 is `이클립스 퀸` / Eclipse Queen.

Data entry:

- `assets/balance_data.js`
- `BOSSES` entry with `stage:10`
- current base stats: `hp:468`, `atk:48`
- `special:'종말'`
- `specialDesc:'매 라운드 특수 공격 혼합'`

Runtime behavior lives mostly in `applyBossAttack()` inside `reincarnation_rpg_v2_no_duplicate_ids.html`.

Phase logic:

- Phase 1: boss HP above 60%.
- Phase 2: boss HP at or below 60%, above 30%.
- Phase 3: boss HP at or below 30%.
- Phase changes log Korean danger messages:
  - phase 2: `이클립스 퀸의 봉인이 풀린다...!`
  - phase 3: `종말이 시작된다!!!`

Special pool:

- Base pool: `curse`, `lightning_freeze`, `laser`.
- Phase 2 adds `flame`.
- Phase 3 adds `vampire` and `mirror`.

Special pick count:

- Phase 1 picks 1 special from the current pool.
- Phase 2 and 3 pick 2 specials from the current pool.

Special effects:

- `curse`: 40% chance to set `combat.cursed = true`, reducing later player damage through curse logic.
- `lightning_freeze`:
  - In phase 1, it can apply lightning stun: 30% chance, `combat.stunned = 2`.
  - In phase 2+, it has a 50% branch toward freeze, then 40% chance to set `combat.frozen = true`.
  - Otherwise it can still use lightning stun at 30%.
- `laser`: 30% chance to multiply current boss attack by 1.5.
- `flame`: 40% chance to immediately deal `floor(player max HP * 0.04)` before the main boss attack resolves.
- `vampire`: sets `combat.eclipseVampire = true`; after the boss deals damage, boss heals for `floor(bossAtk * 0.2)`.
- `mirror`: 30% chance to set `combat.eclipseMirrorReady = true`; the next player attack hit can be nullified.

Other phase behavior:

- Phase 3 multiplies base boss attack by 1.2 before later modifiers.
- Normal strong attack warning logic still applies afterward:
  - `combat.warnNextTurn` has a 25% chance each boss turn.
  - warned strong attack multiplies boss attack by 2.5.
  - defending against a warned strong attack reduces damage to 20% and stuns the boss for 2 turns.

Interaction with player attacks:

- Normal attack checks `combat.eclipseMirrorReady`; if true, it is consumed and the attack damage becomes 0.
- Human double attack checks mirror on the first hit in the current implementation.
- Boss 2 still has its old random mirror behavior separately.

Interaction with traits:

- `guard` applies to boss damage after defending/strong-attack handling and after `instinct`.
- `guard` damage multiplier is:
  - 0.9 normally.
  - 0.8 while defending.

## Verification Commands

Use bundled Node if available:

```powershell
& 'C:\Users\황성구\.cache\codex-runtimes\codex-primary-runtime\dependencies\node\bin\node.exe' -e "const fs=require('fs'); const html=fs.readFileSync('reincarnation_rpg_v2_no_duplicate_ids.html','utf8'); const scripts=[...html.matchAll(/<script(?![^>]*src=)[^>]*>([\s\S]*?)<\/script>/gi)].map(m=>m[1]); scripts.forEach(s=>new Function(s)); console.log('inline syntax ok:', scripts.length, 'scripts');"
```

Check external balance data:

```powershell
& 'C:\Users\황성구\.cache\codex-runtimes\codex-primary-runtime\dependencies\node\bin\node.exe' -e "const fs=require('fs'); const vm=require('vm'); const balance=fs.readFileSync('assets/balance_data.js','utf8'); const ctx={window:{}}; vm.createContext(ctx); vm.runInContext(balance, ctx); const d=ctx.window.GAME_BALANCE; console.log({races:d.RACES.length, monsters:d.MONSTERS.length, traits:d.TRAITS.length, bosses:d.BOSSES.length});"
```

Expected current counts:

- races: 6
- monsters: 20
- traits: 11
- bosses: 10

Check duplicate IDs:

```powershell
$html = Get-Content -LiteralPath .\reincarnation_rpg_v2_no_duplicate_ids.html -Raw -Encoding UTF8; [regex]::Matches($html, 'id="([^"]+)"') | ForEach-Object { $_.Groups[1].Value } | Group-Object | Where-Object Count -gt 1 | Sort-Object Count -Descending | Format-Table -AutoSize
```

If the duplicate ID command prints nothing, that is good.

## Cautions For Future Edits

- Do not move `Clear_img.png` unless updating the HTML path.
- Do not re-inline large base64 images into the HTML.
- Prefer editing boss/race/monster numbers in `assets/balance_data.js`.
- Keep `DEV_MODE` in HTML, not in balance data.
- When changing combat delays, preserve the token guard pattern:
  - `beginCombat()`
  - `isCombatActive()`
  - `scheduleCombat()`
  - `endCombat(victory, token)`
- If editing boss sprites, inspect the actual PNG dimensions first. Frame width is often `imageWidth / frameCount`, and may be fractional.
