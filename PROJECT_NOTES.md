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
- `Clear_img_v2.png`
  - Current final clear screen image. Must remain next to the HTML file unless the HTML path is updated.
- `Clear_img.png`
  - Previous final clear screen image, kept as a backup.
- `Title_img_v1.png`
  - Current title screen key visual. Must remain next to the HTML file unless the title CSS path is updated.
- `assets/endings/human_ending_v1.png`
  - Current human race ending illustration.
  - Intended for the future ending album slot and a 2.0s delayed dark overlay with scrolling narration.

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
├─ Title_img_v1.png
├─ Clear_img_v2.png
├─ Clear_img.png
└─ assets/
   ├─ balance_data.js
   ├─ endings/
   │  └─ human_ending_v1.png
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
  - final boss victory shows a clear panel using `Clear_img_v2.png`.
  - `Clear_img_v2.png` is a newly generated dark fantasy eclipse ending illustration, designed to fit above the HTML-rendered clear record and restart button.
  - Final boss victory now stores `state.clearSummary` and renders `#clear-record` with final race, level, rebirth count, HP/ATK, and traits.
  - Final clear logs use a dedicated Eclipse Queen ending message before setting `state.cleared`.
  - Restart button text is `새 윤회 시작`; restart clears the ending summary and closes DEV controls.
  - Clear screen layout uses a tighter 4/8/12px spacing rhythm:
    - `status-banner` and `trait-banner` are grouped inside `#meta-group`.
    - the clear illustration, title/copy, record card, and CTA are aligned to the same full-width column.
    - clear restart CTA is rendered inside `#clear-panel` under the record card.
    - the old `#btn-restart` remains for non-clear game-over flow only.
  - Clear screen height is optimized to avoid scrolling on normal mobile viewports:
    - `#game`/`#ui-wrap` use fixed viewport-height flex containment.
    - `#battle-stage`, `#clear-panel`, `.clear-scene`, and `.clear-img` all allow shrinking with `min-height:0`.
    - the ending image flexes to fill leftover space and shrink before text/card/button overflow.
    - very small screens may still scroll inside `#clear-panel`, using a thin themed scrollbar.
  - Clear screen entry animation:
    - `.clear-img` fades/scales in first over about 0.72s.
    - `.clear-title` appears after the image with fade + slide-up.
    - `.clear-copy` follows with the same fade + slide-up.
    - record card and CTA fade/slide in afterward for a softer finish.
    - tapping/clicking `#clear-panel` calls `skipClearAnimation()` and snaps all elements to final state.
    - reduced-motion users receive the final static state without animation.
- Asset extraction:
  - large inline base64 assets were moved into `assets/extracted/`.
  - HTML should no longer contain large base64 blobs.
- Debug mode:
  - `DEV_MODE` flag added.
  - Debug controls are collapsed by default behind the `DEV 열기` toggle.
  - debug skip and instant boss kill controls added.
  - intervention event test controls added:
    - `debug-event-select` reserves a specific intervention event for the next normal hunt.
    - `btn-debug-force-event` reserves one weighted-random intervention event for the next normal hunt.
    - Forced events are consumed once by `rollInterventionEvent()`.
- Shared traits:
  - `guard` was added as the 11th trait.
  - It reduces incoming damage by 10%, or by an additional 10% while defending.
  - `magic` now boosts player attacks based on the player's attack style (`race.canMagic`), not the enemy type.
  - `instinct` now boosts outgoing melee-style player attacks and increases all incoming damage by 10%.
  - `bossKill` display text says "damage dealt to bosses +50%" to avoid confusing it with boss attack power.
- Reincarnation intervention events:
  - Normal hunt button has a 5% chance to trigger a weighted intervention event before combat starts.
  - Boss fights do not use intervention events.
  - Current event table: `supply`, `weaponSupply`, `playerFirst`, `enemyFirst`, `rareEnemy`, `rift`, `bloodline`.
  - Event weights total 100: `supply` 18, `weaponSupply` 12, `playerFirst` 20, `enemyFirst` 15, `rareEnemy` 15, `rift` 15, `bloodline` 5.
  - All intervention events use the event modal before combat starts.
  - Non-choice events use a single `전투 시작` confirmation button; `rift` and `bloodline` provide choices.
  - `bloodline` temporarily changes `state.raceIdx` for one combat and stores `temporaryRaceRestoreIdx` on `combat`.
  - Temporary race changes currently swap race passives/skill/UI for the battle, but do not rewrite current HP/ATK growth stats.
  - Always restore temporary race in `endCombat()` before returning to idle/game-over flow.
- Status clarity UI:
  - `status-banner` shows day/night progress as `current tick / TIME_TICKS_PER_SHIFT`.
  - It also shows forced next intervention events, active combat intervention events, and temporary race state.
  - Temporary race battles mark the HUD race name with `*` and the battle player name with `(임시)`.
- Info/help window:
  - `showInfo()` now acts as a broader in-game rule reference.
  - It covers basic progression, combat rules, day/night rules, intervention event chance/effects, races, shared traits, boss stages, clear records, and DEV panel notes.
  - Event help is generated from `INTERVENTION_EVENT_TABLE` weights, with separate Korean effect descriptions in the HTML.
- Title/onboarding:
  - A title screen overlay (`#title-screen`) introduces the goal before the first play interaction.
  - The title screen uses `Title_img_v1.png`, a generated reincarnation gate key visual with race silhouettes and a dark lower text area.
  - It explains the core loop: hunt, challenge bosses, reincarnate on defeat, and clear stage 10.
  - `startGameFromTitle(openInfo)` hides the overlay and can optionally open the info/help window.
  - The title screen includes `기억의 앨범`, currently a preview popup for future race-specific ending images and clear records.
  - If a save exists, `이어하기` appears on the title screen and restores `localStorage` progress.
  - `윤회 시작` and `규칙 보고 시작` always create a fresh run and overwrite the previous current-progress save.
  - The title overlay does not change the existing initial state; it sits above `#ui-wrap`.
- Memory album:
  - `기억의 앨범` now opens a dedicated mobile album overlay (`#album-overlay`) instead of a generic event modal.
  - The album frame uses a dark fantasy panel style with a header, internal scroll body, and no page navigation buttons.
  - Ending slots are arranged in a 2-column scroll grid for mobile.
  - The human slot is currently unlocked and displays `assets/endings/human_ending_v1.png`.
  - Clicking the human slot opens `#ending-credit-overlay`, showing the human ending art first, then a delayed dark scrim and scrolling narration.
  - The remaining race slots are locked placeholders.
  - Future work: connect clear records and race-specific ending images to these slots.
- Auto save:
  - Current progress is stored in `localStorage` under `reincarnationRpgSaveV1`.
  - Saving happens only after the title has been dismissed and when no combat/intervention event is active.
  - Saved fields include current state such as race, level, HP, ATK, XP, boss stage, rebirth count, traits, day/night, rest count, clear state, and clear summary.
  - Combat state itself is not serialized; reload resumes at the safe non-combat game screen.
  - `restartGame()` clears the old current save, creates a fresh run, then saves that new run.
- Vampire race and day/night:
  - `vampire` was added as a reincarnation race.
  - Vampire can appear in any boss rebirth pool only if the player died while holding the common `vampire` trait.
  - Current vampire rebirth weight is 50, so most boss pools give vampire about a 33.3% chance when the condition is met.
  - Day/night state is stored on `state.timeOfDay` with `state.timeTick`.
  - Time advances only after combat ends or rest completes, so day/night does not change mid-combat.
  - Every 4 time ticks toggles day/night.
  - Normal hunt first-strike chance is 15% by day and 35% by night.
  - Night normal hunt victories get +15% base XP before the usual XP multipliers.
  - Night first-strike monster victories get an additional +35% base XP, stacking multiplicatively after the night hunt bonus.
  - Rest big-success chance is 12% by day and 25% by night.
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
- Earth spirit balance:
  - Earth spirit ATK increased from 6 to 7.
  - Earth defense now starts earlier and scales as HP falls:
    - 75% or lower: damage -2
    - 50% or lower: damage -5
    - 25% or lower: damage -9
    - 10% or lower: damage -14
  - Earth Barrier skill cooldown reduced from 5 turns to 4 turns.
- Poop poison:
  - `독액` applies 4-turn poison.
  - Poison damage is `floor(maxEnemyHp * poisonTick * 0.04)`, so normal ticks are 4%, 8%, 12%, and 16% of enemy max HP.
  - If the skill cooldown is ready while poison is still active, the skill button changes to `독 증폭`.
  - `독 증폭` immediately deals the next poison tick, then reapplies a fresh 4-turn poison if the enemy survives.
  - This is intended to create a choice between direct attack damage plus the pending poison tick, or spending the action to front-load and refresh poison.
- Combat async hardening:
  - combat uses `combatSeq`/`token`.
  - delayed combat callbacks use `scheduleCombat()` to avoid stale timers affecting new combats.
- Combat log clarity:
  - Enemy stun logs now use enemy action counts instead of ambiguous "turns".
  - Defense stun application says "after this, enemy actions are canceled 2 times".
  - Stun consumption logs whether the current enemy action was canceled and whether any additional action cancels remain.
  - Enemy stun state is also shown visually with `#enemy-stun-bubble` near the enemy sprite.
  - The bubble displays `xN`, matching the remaining enemy action cancellations stored in `combat.stunned`.
- Combat sound:
  - Simple attack and hit sounds are generated with the browser Web Audio API.
  - No external audio assets are required.
  - Player normal attacks, human double attacks, and vampire Blood Thirst play attack sounds.
  - Successful enemy/player damage events play short hit sounds, with heavier hits for strong attacks.
  - General UI buttons play a short click sound through `playButtonSound()`.
  - Buttons inside `#combat-actions` do not play the UI click sound to avoid overlapping attack/skill combat sounds.
  - Disabled buttons do not play click sounds.
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
  - `브램블 울프`, `아이언터스크 보어`, and `스톤쉘 크랩` use `mob_3.png`, `mob_4.png`, and `mob_5.png` respectively.
  - `글룸 스토커` uses generated shadow panther sprite `mob_6.png`.
  - All physical normal monsters now have dedicated single-image sprites:
    - `브램블 울프`: `mob_3.png`
    - `아이언터스크 보어`: `mob_4.png`
    - `스톤쉘 크랩`: `mob_5.png`
    - `글룸 스토커`: `mob_6.png`
    - `레드혼 스태그`: `mob_7.png`
    - `캐리언 리퍼`: `mob_8.png`
    - `미스트 클로`: `mob_9.png`
    - `케이브 마울러`: `mob_10.png`
    - `블러드팽`: `mob_11.png`
    - `루인 골렘`: `mob_12.png`
  - Magic normal monster sprites:
    - `윌로우 위스프`: `mob_13.png`
    - `문블룸 페어리`: `mob_14.png`
    - `크리스탈 스프라이트`: `mob_15.png`
    - `헥스 크로우`: `mob_16.png`
    - `스타폴 미믹`: `mob_17.png`
    - `머쉬룸 세이지`: `mob_18.png`
    - `데드우드 드라이어드`: `mob_19.png`
    - `아스트럴 모스`: `mob_20.png`
    - `벨벳 위치캣`: `mob_21.png`
    - `에클립스 스펙터`: `mob_22.png`
  - Current normal monster sprite baseline is single-image breathing sprites at `96x116` canvas/display (`min(96px, 24vw)` by `min(116px, 29vw)`).
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
