# CLAUDE.md — Project G Design Reference

## ⚠️ No Source Code Yet — This Repo Is Design Documentation Only

As of now, `/home/user/ProjectG` contains **no game engine, no code, no build system** —
only game design documentation exported from Google Sheets (as single-line minified
HTML table dumps) plus a PDF of the same design doc. **Any future implementation work
(engine choice, data schema, gameplay systems, balancing tools, etc.) must start by
reading these source documents**, because they are the only spec that exists. This
CLAUDE.md is a distilled, human/agent-readable summary of those documents — but it is
a summary, not a replacement. When in doubt, or when a numeric value / exact column
name is needed, go back to the original file.

### Source documents (all under `Doc/`)

- `/home/user/ProjectG/Doc/Project G.pdf` — same design doc as a PDF, includes diagrams/images not present in the HTML exports (e.g. tile-map path illustrations). Worth skimming for visual context.
- `/home/user/ProjectG/Doc/Project G html/개요.html` — Overview: game concept, core systems, core play loop (95,900 bytes)
- `/home/user/ProjectG/Doc/Project G html/모드.html` — Game modes / difficulty table (17,132 bytes)
- `/home/user/ProjectG/Doc/Project G html/몬스터.html` — Monster stat table schema (15,754 bytes)
- `/home/user/ProjectG/Doc/Project G html/사건.html` — Events ("사건") table: 46 concrete world events with full data (74,107 bytes)
- `/home/user/ProjectG/Doc/Project G html/스킬.html` — Skills table: skills for the first 10 gods (30,196 bytes)
- `/home/user/ProjectG/Doc/Project G html/신.html` — Gods/deities stat & schema table for 10 gods (27,904 bytes)
- `/home/user/ProjectG/Doc/Project G html/웨이브.html` — Wave table: 10 example waves (19,457 bytes)
- `/home/user/ProjectG/Doc/Project G html/인게임 사양.html` — In-game spec: the master systems-design document (tile map, monster paths, wave flow, tower/god pricing formulas, event probability math, oracle system) (132,628 bytes — largest, most detailed file)
- `/home/user/ProjectG/Doc/Project G html/점수.html` — Score ("점수"/"WS", world-state) table: 7 score axes (15,228 bytes)
- `/home/user/ProjectG/Doc/Project G html/타워(신) 리스트.html` — Tower(God) list: mythology reference/flavor text for the Greek pantheon (20,777 bytes)

Note on the source format: these HTML files are raw Google Sheets exports (`<table>`
markup with heavy inline CSS), all on effectively one giant line. When reading them
directly, strip `<style>` blocks and parse `<tr>/<td>` structure; do not attempt to
read them as prose.

---

## 1. Project Overview (source: `Doc/Project G html/개요.html`)

**Genre**: A tower-defense game fused with a stock-market / real-estate trading
simulator. Working title in the docs is "Project G"; the trading exchanges are named
**GADSDAQ** and **UNDER-DAQ** (puns on NASDAQ), and gods are literally purchased and
sold like stocks.

**Core pitch (from the doc, translated)**:
- A. A tower-defense game that uses systems modeled on stock trading / real-estate trading.
- B. Mythological gods are used simultaneously as *towers* and as *stocks* — you buy them, place them, and they defend against waves of incoming monsters.
- C. Via an **Oracle (신탁)** system you get partial information about future world events, and use that knowledge to buy gods whose price is about to rise, or sell gods that have already appreciated — growing your capital to carry into the next round.

### Core Systems (2. 핵심 시스템)

**A. Gods (신) & Divine Candidates (신격체)**
- Gods (신) fill the role of "tower" in this TD game.
- Gods are purchased in **shares/stock units (주)**; base stats scale with `owned share count × price per share`.
- Special abilities unlock in tiers as your share count crosses thresholds; each god has a fixed total share issuance ("총 발행된 주수").
- Each god has a unique ability, so squad composition matters (example given: pairing an AoE-damage god with a single-target-damage god).
- Divine Candidates (신격체) are "IPO candidates" — unlisted stock. They cannot be used as towers while in candidate form. Depending on play conditions, a candidate can be **enshrined (봉신)** and "go public" (IPO) into a full God.

**B. Incense (향화) & Exchanges (거래소)**
- **향화 (Incense)** is the universal currency used to buy everything in the game.
- The more people worship/venerate a given god or candidate, the higher that entity's price rises.
- Incense is earned by defeating monsters or by trading on the exchanges.
- **GADSDAQ (신 거래소)** — the exchange for *listed* (IPO'd) gods. Trading happens continuously during play; prices move in real time based on world events. A purchased god is immediately available to place in defense.
- **UNDER-DAQ (신격 거래소)** — the exchange for *unlisted* candidates (신격체). When an unlisted candidate's value crosses a threshold and holds it long enough, it triggers enshrinement (봉신) — the candidate becomes a full god, moves to GADSDAQ, and its value jumps by N%.

**C. Sacred Domain (신성 영역, "Eden") & Shrines (신전)**
- The Sacred Domain is the literal tower-defense battlefield: a grid ("바둑판") of shrine tiles.
- Monster spawn points and paths run along the connective lines between shrines, forming a closed loop the monsters circulate around.
- The monster path shifts by N tiles every time a "제앙" (calamity/disaster, i.e. a boss wave) occurs.
- Loss condition: the Sacred Domain accumulates too many monsters, or a calamity (boss) is not defeated within its time limit.
- A **신전 (Shrine)** is a single grid tile. Shrines are purchased with Incense; price fluctuates based on:
  - **Monster path proximity** — tiles closer to active monster paths cost more.
  - **Buff radius** — a shrine that buffs the god placed on it will, on each calamity, relocate randomly within N tiles; whether a tile falls inside that buff-move radius affects its price.
- Shrines can be bought and sold; selling incurs a fee that decreases the longer you've held through waves.

**D. Oracle (신탁) & Mortal-World Events (인간계 이슈)**
- Over the course of play, "mortal world issues" (event instances) are generated; each shifts specific gods' valuations up or down.
  - Example: a great flood/thunderstorm event raises the value of Noah/Zeus/Thor-type gods.
  - Example: a war-outbreak event raises war gods (Ares, Achilles) but lowers finance/wealth-related gods.
- Event generation is randomized but weighted so narratives can follow logically (e.g., after a war event, a peace-themed event and a tech-development event tend to follow).
- N events are generated per time unit and shared with the player in real time; god prices update in real time as events fire.
- The same event can produce different outcomes depending on what other events co-occur with it.
- **Oracle (신탁)**: a system that previews upcoming mortal-world events ahead of time. It does not reveal all events, only a limited number, and even those are shown with parts of the text obscured/redacted. Spending Incense increases either the precision of the preview or the number of events revealed.

### Example Play Loop (3. 플레이 루프 예시)
1. **Round start setup** (randomized every round for replay variety):
   - The set of *listed* gods available is randomized each round; playing more unlocks more gods that can appear listed in future rounds.
   - The set of *unlisted candidates* is fixed in count; when one IPOs, a new random candidate fills the slot.
   - Starting "정세" (world situation/climate) is randomized each round and biases the probabilities of later events.
   - Monster lines: a random path and spawn point are generated on the map; the 4 tiles nearest the spawn point are granted to the player as starting real estate.
2. **During play**: monsters spawn continuously along the path; the player buys/places gods to defend before the fail condition triggers; world events and Oracle previews update continuously; the pacing of monster ramp-up is deliberately gentle so players have time to think about investment decisions (this is a TD game, but paced for economic strategy, not just reflexes).
3. **Game end**:
   - If the player can't hold out, the game ends and records the last wave survived.
   - There are (at least) two mode types: a staged/cleared mode with a defined clear condition, and an endless **Hell Mode** with no end, for leaderboard/score chasing. Staged clear mode doubles as tutorial/onboarding; Hell Mode is the endgame content.
   - On clearing, players may permanently retain gods they collected: retained gods can appear pre-listed in future runs, are shown as "collected" for a sense of progression, and specific per-god unlock conditions exist in-game.

A UI mock description in the doc shows a live dashboard with: current Incense held,
asset valuation total, defense status (current wave / time remaining / current monster
count), real-time Oracle info text, owned gods with price/quantity/avg. buy price
(e.g., Zeus 5% up, current price 255; Hades -2%; Hera +2%; Demeter +3%; Poseidon +5%;
Hephaistos +2%; Dionysos -3%), and a sortable trade listing panel (sorted by % gain).

---

## 2. Game Modes (source: `Doc/Project G html/모드.html`)

This file is a **schema/template table** ("모드 테이블의 예시 테이블 구성") rather than a
final content list — it defines the columns a mode-config CSV will have, with one
worked example row. Treat the column semantics below as authoritative; only one
concrete mode ("프로토타입") is actually populated.

Columns:
- `Id` (int) — mode ID.
- `#Name` — internal/planning-only label (columns prefixed `#` are excluded from the exported CSV).
- `ST_ModeName` (string) — out-of-game display string key for the mode name.
- `Path_BalanceName` (string) — path to the balance data file this mode uses.
- `StartCoin` (int) — starting Incense/currency.
- `DefaultWs07` (int) — baseline population score (WS id 7, "인류의 수" / population), used as a reference point for price formulas.
- `StartWSGroup01` (int list) — first group of world-state score IDs randomized at round start.
- `StartWS01Min` / `StartWS01Max` (int) — min/max range for that group's randomized starting values.
- `StartWSGroup02`, `StartWS02Min`, `StartWS02Max` — second such group (present as columns; empty in the example row).
- `WaveEventMax` / `WaveEventMin` (int) — max/min number of events ("사건") to generate across the run for this mode.
- `ClearWave` (int) — the wave number required to clear the mode.

**Example data row** ("프로토타입" / prototype mode):
`Id=1, Name=프로토 타입, ST_ModeName=ProtoModeName, Path_BalanceName=WaveProto, StartCoin=2000, DefaultWs07=100, StartWSGroup01=1,2,3,4,5,6, StartWS01Min=40, StartWS01Max=60, WaveEventMax=2, WaveEventMin=4, ClearWave=10`.

So the prototype mode: starts with 2000 Incense, baseline population 100, randomizes
starting values for world-state scores 1–6 (war/abundance/tech/culture/density/faith)
into the 40–60 range, generates between 2 and 4 events across the run, and is cleared
at wave 10.

No other modes (e.g. an explicit "Hell Mode" row) are present in this file even
though Hell Mode is described conceptually in 개요.html — future engineers will need
to add further mode rows themselves, following this schema.

---

## 3. Monsters (source: `Doc/Project G html/몬스터.html`)

Like 모드.html, this is primarily a **schema table** with a handful of illustrative
rows, not a full bestiary.

Columns:
- `Id` (int) — monster ID. **Boss monsters start numbering at 1000001.**
- `MonsterTypes` (int) — a **bitflag** field. Each trait is assigned a power-of-two value and summed to represent multiple simultaneous traits. Documented example trait values:
  - `1 (2^0)` **회피 (Evasion)** — dodges basic attacks with N% probability.
  - `2 (2^1)` **대시 (Dash)** — dashes forward every N seconds (ignores crowd control).
  - `4 (2^2)` **흡수 (Absorb)** — heals 2% HP whenever a nearby monster dies.
  - `8 (2^3)` **잔류 (Lingering)** — after death, remains on the field for N seconds (untargetable, but still counts toward the field's monster-count cap).
  - Example given: `MonsterTypes = 6` (2+4) means the monster has both Dash and Absorb.
- `HpRate` (int) — this monster's HP as a percentage of the wave's Base HP.
- `SpeedRate` (int) — this monster's move speed as a percentage of the wave's Base Speed.
- `ResistRate` (int) — this monster's CC-resistance as a percentage of the wave's Base Resist.
- `RewardAmountRate` (int) — this monster's currency drop as a percentage of the wave's Base Reward.
- `Path_Resource` (string) — path to the monster's prefab/resource.

Stat resolution formula noted in the sheet (and repeated/expanded in 인게임 사양.html):
`actual stat = round(WaveBaseStat × (MonsterRate/100))`, rounded to an integer.

**Example rows** (Id / MonsterTypes / HpRate / SpeedRate / ResistRate / RewardAmountRate):
| Id | Types | HpRate | SpeedRate | ResistRate | RewardRate |
|---|---|---|---|---|---|
| 1 | 0 | 100 | 120 | 100 | 100 |
| 2 | 8 (Lingering) | 80 | 80 | 100 | 100 |
| 100001 (boss) | 3 (Evasion+Dash) | 3000 | 100 | 60 | 5000 |
| 100002 (boss) | 5 (Evasion+Absorb) | 5000 | 100 | 60 | 5000 |

This is honestly sparse — only 4 example monsters exist in the doc (2 normal, 2 boss),
enough to establish the schema and rough HP/reward scale for bosses (30–50x normal
monster HP, 50x reward), but not a real content roster. A future implementer will need
substantially more monster rows to populate a real game.

---

## 4. Events / World Events (source: `Doc/Project G html/사건.html`)

This is the richest and most concretely populated system file: **46 fully data-filled
event rows** (event roman IDs 1–47, with #16 apparently skipped) representing "mortal
world issues" that shift both the 7 world-state scores (점수.html) and god prices.

### Schema
- `Id` (int), `#Desc` (planning-only Korean flavor text of the event — this doubles as the *only* description field; there's no separate player-facing template beyond `St_CaseDesc`, which is present as a column but empty in every actual row).
- `ConditionWSId` / `ConditionWSOp` / `ConditionWSValue` — gating condition: the event can only appear if the given world-state score is `≥` (`Op=0`) or `<` (`Op=1`) `ConditionWSValue`. If unmet, the event isn't included in the probability pool at all.
- `Baseweight` (int) — base spawn weight.
- `CauseWSId01`/`CauseValue01`, `CauseWSId02`/`CauseValue02` — up to two world-state scores that additionally modify the event's weight: `finalWeight = BaseWeight + WS01.currentValue × (CauseValue01/100) + WS02.currentValue × (CauseValue02/100)` (per 인게임 사양.html §4-F, weight floors at 0 if negative).
- `EffectWSGroup01[]` / `EffectWSValue01`, `EffectWSGroup02[]` / `EffectWSValue02` — world-state score ID(s) affected and the delta applied immediately: `newScore = currentScore + delta`.
- `AfterEffectWave` / `AfterEffectWSGroup[]` / `AfterEffectWSValue` — a delayed effect: N waves after the event fires, another score changes.
- `EffectGodGroup01[]`/`Value01`, `...02`/`Value02`, `...03`/`Value03` — up to 3 groups of gods whose price is multiplied by `(100+Value)/100` when this event fires.
- `St_CaseDesc` — UI string key for player-facing text (present in schema, unused/empty in all sampled rows).

### Sample of concrete events (id — description — key effects)
- **#1**: "Minor conflict between nations; a wise mediator is needed" — Baseweight 50; EffectWSGroup01=5,10 value +10 (delayed after 5 waves... — actually AfterEffectWave column used).
- **#2**: "Locust swarm devours crops" — Weight 50; WS 2,5,7 −20, WS 6 +10; Gods group 4 −10, group 3 +5.
- **#3**: "Flood from heavy rain, but soil quality improves" — Weight 50; WS 2,5,7 −10, WS 6 +10; delayed (3 waves later) WS 2 +15; Gods group 1 +15, group 3 +5.
- **#4**: "Bumper harvest, granaries overflow" — Weight 70, boosted by WS3 (+20); WS 2,5,7 +10, WS 6 +10; Gods group 4 +20, group 6 +10.
- **#5**: "Great storm sinks ships at sea" — Weight 50; WS 2 −15; Gods group 2 −10, group 3 +10.
- **#6**: "Fishing catch increases greatly" — Weight 70 (boosted by WS3 +20); WS 2 +20; Gods group 2 +20, group 4 +10.
- **#7**: "A festival is held successfully" — Gated on WS2 ≥ 80; Weight 100 (boosted by WS2 +30, WS4 +20); WS4 +10, delayed WS2 (2 waves later) +10; Gods group 4,8 +10, group 10 −10.
- **#8**: "Wild animals increase; public safety worsens, cities stop communicating" — Weight 50; WS 2,3,4 −10; delayed (3 waves) WS1 +10; Gods group 9 −20, group 7 +10.
- **#9**: "A new heroic epic becomes popular" — Weight 50 (boosted by WS4 +30); WS5 −10, WS4 +10; delayed (3 waves) WS1 +10; Gods group 1 +20, group 6 +10.
- **#10**: "New astronomical theory emerges" — Weight 50 (boosted by WS3 +30); WS3 +10, WS6 −10; Gods group 9 +10, group (1,2,3,4,5,6,7,8,10) −20.
- **#13**: "Large-scale war breaks out" — Gated WS1 ≥ 70; Weight 100 (boosted WS1 +50); WS1 +20, group(5,6,2) −20; delayed (2 waves) WS3 +10; Gods group 10 +20.
- **#14**: "A nation is conquered; harsh repression follows" — Gated WS1 ≥ 90; Weight 100 (boosted WS1 +50); WS1 −70, group(3,4) −20; Gods group 3 +15, group 8 −10.
- **#15**: "A deadly plague spreads" — Weight 50 (boosted WS5 +50); WS 2,5,7 −20, WS(3,6) +10; Gods group 3 +20, group 6 +10.
- **#17**: "New medical technology developed; many survive" — Weight 100 (boosted WS3 +60); WS 2,5,7 +30, WS6 −10; delayed(2 waves) WS7 +10; Gods group 6 +20, group 3 −20, group(1,2,4,5,6,7,8,10) −10.
- **#18/#19**: "A massive temple/library is built" — Gated WS2 ≥ 70; effects on WS2, WS6/3, and broad god-price effects (18 boosts ALL 10 god groups +10).
- **#22**: "A peace treaty ends a war" — Gated WS1 ≥ 90; WS1 −70; Gods group 10 −30.
- **#24**: "Olympic-style games held; all nations set down arms" — Gated WS2 ≥ 50; WS(2,5) +10, WS1 −10; Gods group 10 −10, group 3 −5, group 1 +10.
- **#28**: "Piracy runs rampant; sea trade suffers" — Gated WS2 < 50; WS(2,3,4) −20; Gods group 2 −20, group 9 −10.
- **#33**: "Technologists persecuted for irreligious inventions" — Gated WS3 ≥ 70; WS3 −30, WS6 +20; Gods group 9 −10, group 3 +5.
- **#39/#40**: "Solar/Lunar eclipse; people make prophecies" — WS7 +20/−30 and WS6 −30/+30 respectively (mirror-image events).
- **#41**: "Two nations form a marriage alliance" — Gated WS1 < 40; WS(2,5) +10, WS1 −10; delayed(2 waves) WS7 +10; Gods group 8 +20, group 10 −20.
- **#42**: "Luxury goods trend; wealth gap widens, class conflict rises" — Gated WS2 ≥ 70, weight 200 (high); WS2 −10, WS1 +10; Gods group 8 +20.
- **#45**: "A major scientific discovery is made, slow to spread" — Gated WS3 ≥ 80; WS3 +10, WS6 −10; delayed(3 waves) WS2 +10; Gods group (all 10) −10.
- **#46**: "Clergy corruption is exposed; citizens disillusioned" — Gated WS6 ≥ 80, weight 80 (boosted WS2 +20); WS6 −20; delayed(3 waves) WS1 +20; Gods group(all 10) −10.

(This is a representative sample of the 46 rows; consult `Doc/Project G html/사건.html`
directly for the full table — every row follows the schema above, covering themes of
war/peace, famine/harvest, plague/medicine, religion/technology conflict, exploration,
trade, and celestial omens.)

The event system is clearly the game's core "narrative economy" driver: each event is
a small multi-variable transaction against the 7 world-state scores and against 1–3
god-price groups, gated and weighted by the current world state, creating emergent
chains (e.g., war events cascade into peace events cascade into culture events).

---

## 5. Skills (source: `Doc/Project G html/스킬.html`)

Schema + fully populated skill list for the **first 10 gods** (3 skills each = 30
skills, IDs following the pattern `1{godId:03d}{skillNumber:03d}`, e.g. `1001001` =
first skill of god 1).

### Schema
- `Id` (int) — pattern `1{GodId}{SkillIndex}`.
- `#Desc` — planning-only description (doubles as the actual skill text in these examples).
- `SkillType` — 0 = Passive, 1 = Active (documented in schema but not populated per-row in the sample; skills described read mostly as passive/auto-triggering).
- `ConnectSkillId` — if this skill *upgrades* another skill rather than being standalone, the target skill's ID.
- `SkillBaseValue01`–`04` (int) — up to 4 generic numeric parameters, meaning varies per skill.
- `Path_Icon`, `St_SkillName`, `St_SkillDesc` — UI string/asset keys (present in schema, unpopulated in sample rows — the `#Desc` column carries the actual text instead).

### The 10 gods' skill kits (base skill + 2 "collection"/upgrade skills, 컬렉션/수집 스킬 = presumably unlocked at higher share-count tiers, matching 신.html's `Skill0XQuantity` thresholds)

1. **Zeus (제우스)**
   - Base (1001001): basic attack also hits 2 additional nearby random enemies.
   - Collection 1 (1001002, connects to base): every 3rd attack has 50% chance to instantly kill the damaged target.
   - Collection 2 (1001003, connects to base): basic attack now hits 5 additional nearby enemies (up from 2).
2. **Poseidon (포세이돈)**
   - Base (1002001): every 10s, summons a tidal wave from the start point moving opposite the monster path; pierces 5 monsters, stuns them 1s, deals 500% of attack power as damage.
   - Collection 1: wave move speed increases; triggers again upon reaching the entrance.
   - Collection 2: wave pierce count +5.
3. **Hades (하데스)**
   - Base: collects a "soul" per kill; at 30 souls, deals N% (value 800, i.e. presumably ~800% somehow tied to attack power) damage to all enemies on the field and consumes the souls.
   - Collection 1: attack power +N% per collected soul (10% shown, based on base attack, non-compounding).
   - Collection 2: soul threshold for base skill increases to 20 (from 30... note value shown is 20) and attack speed +1 per soul collected.
4. **Demeter (데메테르)** — "밀밭"/Wheat Field
   - Base: monsters within attack range have move speed reduced 30%.
   - Collection 1: 50% chance monster kills within range yield double currency.
   - Collection 2: at end of wave, gain an extra 10% of held currency (value fields show 20/1, exact meaning ambiguous — verify against source).
5. **Athena (아테나)**
   - Base: gods within Athena's attack range get +10% attack power.
   - Collection 1: Athena's attack range +1, and the in-range attack-power buff increases by another +10%.
   - Collection 2: attack range of gods within Athena's range +1.
6. **Apollo (아폴론)**
   - Base: attacked targets take a burn DoT of 5% attack power per second (max 3 stacks).
   - Collection 1: attacking a burning target extends 5s of +10% damage (max 3 stacks).
   - Collection 2: max stack count for all Apollo skills +5.
7. **Artemis (아르테미스)**
   - Base: repeatedly attacking the same target increases damage to it by 5% per hit.
   - Collection 1: attack range +1; 30% chance basic attacks deal +50% damage.
   - Collection 2: attack range +1; the bonus damage from switching targets is retained at 50%.
8. **Aphrodite (아프로디테)**
   - Base: enemies within attack range take +10% damage from all sources.
   - Collection 1: enemies within range take an additional DoT of 20% attack power per second.
   - Collection 2: every 10s, pulls 5 random monsters from outside the attack range into random positions inside it.
9. **Hermes (헤르메스)**
   - Base: additional Oracle purchase price −20%.
   - Collection 1: chance of higher-tier Oracle results +30%.
   - Collection 2: attack power +30% per Oracle currently held.
10. **Ares (아레스)**
    - Base: if no other god is within 10 adjacent tiles, attack speed +10.
    - Collection 1: attack power +3%(shown as value 1, likely per-enemy in range — verify) per enemy in attack range.
    - Collection 2: every 10s, attack range +2 for 3 seconds.

Only these 10 gods have documented skills — matches the 10 rows in 신.html. Additional
gods referenced elsewhere (Hephaistos, Dionysos, Hera, and the extended pantheon list
in 타워(신) 리스트.html) have **no skill data** in this file yet.

---

## 6. Gods / Deities & Tower List (sources: `Doc/Project G html/신.html`, `Doc/Project G html/타워(신) 리스트.html`)

### 6a. God/Tower stat schema and data (신.html)

Columns:
- `Id` (int), `#Name` (planning-only).
- `BaseDamage` (int) — base attack power.
- `AttackSpeed` (int) — attacks per second × 10 (i.e. value/10 = actual attacks/sec).
- `Attack Range` (int) — radius, computed as `(distance between adjacent-tile centers) × AttackRange`.
- `Attack Type` (int, enum): `0 = Normal` (closest enemy along the path within range), `1 = Finisher` (lowest current HP in range), `2 = Strong` (highest max HP in range), `3 = Random` (random target in range).
- `Skill01Id`/`Skill01Quantity`, `Skill02Id`/`Skill02Quantity`, `Skill03Id`/`Skill03Quantity` — skill references (from 스킬.html) and the share quantity required to unlock each.
- `BasePrice` (int) — base purchase price; actual market price fluctuates from this baseline.
- `MaxQuantity` (int) — maximum shares that can exist/be held for this god.
- `CoreWSID` (int) — the world-state score (from 점수.html) that most directly drives this god's price.
- `WSCorrelation` (0/1) — whether the core WS score correlates positively (0) or negatively (1) with this god's price.
- `ST_GodName`/`ST_GodDesc`/`Path_GodPrepab` — string keys and prefab path.

**Full data table (10 gods)**:

| Id | Name | BaseDmg | AtkSpd | Range | AtkType | Skill1 (qty) | Skill2 (qty) | Skill3 (qty) | BasePrice | MaxQty | CoreWSID | WSCorrelation |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 1 | Zeus | 10 | 10 | 2 | Normal(0) | 1001001 (0) | 1001002 (10) | 1001003 (40) | 500 | 50 | 5 (신앙/faith)| 0 (positive) |
| 2 | Poseidon | 15 | 5 | 4 | Normal(0) | 1002001 (0) | 1002002 (10) | 1002003 (40) | 500 | 50 | 2 (풍요/abundance) | 1 (negative) |
| 3 | Hades | 10 | 12 | 3 | Finisher(1) | 1003001 (0) | 1003002 (10) | 1003003 (40) | 500 | 50 | 1 (전쟁/war) | 0 |
| 4 | Demeter | 10 | 5 | 4 | Normal(0) | 1004001 (0) | 1004001 (20)* | 1004001 (50)* | 400 | 60 | 2 (abundance) | 0 |
| 5 | Athena | 10 | 10 | 1 | Normal(0) | 1005001 (0) | 1004001 (15)* | 1004001 (50)* | 400 | 60 | 4 (문화/culture) | 0 |
| 6 | Apollo | 10 | 10 | 5 | Normal(0) | 1006001 (0) | 1006001 (20)* | 1006001 (50)* | 400 | 60 | 4 (culture) | 0 |
| 7 | Artemis | 5 | 20 | 3 | Strong(2) | 1007001 (0) | 1007002 (20) | 1007003 (50) | 400 | 60 | 2 (abundance) | 0 |
| 8 | Aphrodite | 10 | 10 | 3 | Normal(0) | 1008001 (0) | 1008002 (20) | 1008003 (50) | 400 | 60 | 5 (faith) | 0 |
| 9 | Hermes | 5 | 20 | 3 | Normal(0) | 1009001 (0) | 1009002 (10) | 1090003* (20) | 1000 | 20 | 3 (기술/tech) | 0 |
| 10 | Ares | 10 | 13 | 2 | Finisher(1) | 1010001 (0) | 1010002 (20) | 1010003 (50) | 400 | 60 | 1 (war) | 1 (negative) |

`*` = likely data-entry typos in the source sheet (Demeter/Athena/Apollo repeat the
same skill ID `1004001`/`1005001`/`1006001` across all 3 skill slots instead of
referencing distinct IDs; Hermes skill 3 is written `1090003` instead of the expected
`1009003`). Flag these for the design team rather than silently "fixing" them when
building data-import tooling — cross-reference against 스킬.html (which does define
distinct `1004002`/`1004003`, `1005002`/`1005003`, `1006002`/`1006003`, `1009003` IDs)
if the intent was clearly the sequential skill IDs.

Price formula (also see §8 below, from 인게임 사양.html):
`currentAttackPower = BaseDamage × ownedShareCount × (currentPrice / BasePrice)`.

### 6b. Tower/God List mythology reference (타워(신) 리스트.html)

This file is primarily **flavor-text reference material** for the Greek pantheon (not
balance data) — likely intended as writer's-room lore reference rather than something
to parse programmatically. It documents, per god: title/domain, personality/epithet,
signature weapon(s), and iconography/symbols. Populated in detail for the first 5:

- **Zeus (제우스)** — Highest-ranked sky god; governs kingship, law, justice, friendship, strength (doc bluntly also flags him as "a rapist and womanizer" per the myths). Weapons: **Astrape** (lightning spear/bolt, one of the "3 great regalia" of the chief gods) and the **Aegis** (shield, shared with Athena, bears the Gorgon's head). Symbols: sky, thunder, law/justice, eagle, oak tree, Jupiter (planet).
- **Hera (헤라)** — Highest-ranked goddess; queen of the gods; also governs kingship/law/justice/friendship/strength (per doc, mirrors Zeus's domain text). No signature weapon listed. Symbols: femininity, home, cow, peacock, crown, pomegranate.
- **Poseidon (포세이돈)** — Sea god; causes storms and earthquakes when angered; rides a horse with bronze hooves and a golden mane. Weapon: **Trident** ("삼지창" — "three teeth"). Symbols: sea, storm, horse, Neptune (planet), trident, dolphin.
- **Demeter (데메테르)** — (doc text duplicates Zeus's description verbatim here, likely a copy-paste placeholder error in the source — flag for the design team). Weapon: **Cornucopia**, the horn of plenty that produces unlimited food/wealth. Symbols: harvest, earth, seasons, cornucopia, wheat, torch.
- **Athena (아테나)** — Goddess of wisdom, war, and civilization; the winged goddess of victory Nike attends her. Weapon: **Aegis** (shield adorned with the Gorgon's head, shared with Zeus). Symbols: wisdom, war, helmet, spear, armor, chariot, olive.

Beyond Athena, entries 6–10+ are present as headers/placeholders without filled-in
detail in the extracted text (blank rows in the source) — future writers/engineers
should check the source HTML/PDF directly for whether this content exists further
down or is simply unwritten.

The file also includes a **genealogy/naming reference table** for the wider
Twelve-Olympian pantheon (Greek name → Roman name → relation to Zeus):

| Greek | Roman | Relation to Zeus |
|---|---|---|
| Demeter | Ceres | Zeus's elder sister |
| Athena | Minerva | Zeus's legitimate eldest daughter |
| Apollo | Apollo | Zeus's illegitimate son |
| Artemis | Diana | Zeus's illegitimate daughter |
| Ares | Mars | Zeus's legitimate second son |
| Aphrodite | Venus | Zeus's youngest aunt, or alternatively his illegitimate daughter |
| Hephaestus | Vulcan | Zeus's legitimate firstborn son |
| Hermes | Mercury | Zeus's illegitimate son |
| Hestia | Vesta | Zeus's eldest sister |
| Dionysus | Bacchus | Zeus's illegitimate son |
| Hades | Pluto | Zeus's eldest brother and son-in-law |
| Persephone | Proserpina | Zeus's illegitimate daughter |

Notes accompanying this table: the "Twelve Olympians" traditionally refers to the
twelve deities from Demeter through Hestia (or, alternatively, excluding Hestia but
including Dionysus). Hades and Persephone, while divine, are residents of the
Underworld rather than Olympus, so they're often treated as equivalent in stature but
excluded from the canonical Twelve.

Note: this pantheon reference (12 Olympians + Hades/Persephone) is broader than the
10 gods actually implemented with stats/skills in 신.html/스킬.html — Hera,
Hephaestus, Hestia, Dionysus, and Persephone appear in lore/flavor references and in
UI mockups (개요.html mentions Hephaistos and Dionysos as owned-gods examples) but
currently have **no BaseDamage/AttackSpeed/skill data**.

---

## 7. Waves (source: `Doc/Project G html/웨이브.html`)

Schema + 10 example wave rows (WaveNumber 1–10), evidently the "WaveProto" balance
file referenced by the prototype mode in 모드.html.

### Schema
- `WaveNumber` (int) — sequential wave order.
- `WaveTime` (int, seconds) — duration of the wave.
- `OverMonsterCount` (int) — game-over threshold for monsters simultaneously on the field (blank defaults to 1).
- `WaveType` (string enum) — `Normal` (wave ends once the "monster End value" is passed) or `Boss` (ends on passing the End value, or fails if the boss isn't killed within the wave time).
- `Monster01Id` / `Monster01SpawnInterval` (seconds) — first monster type spawned and its spawn cadence.
- `Monster02Id` / `Monster02SpawnInterval` — optional second concurrent monster type/cadence (both start spawning simultaneously when the wave begins, if both are set).
- `BaseHP`, `BaseSpeed`, `BaseCcResist`, `BaseRewardAmount` (int) — the wave's baseline stats that each spawned monster's `*Rate` columns (from 몬스터.html) multiply against.

### Full prototype wave table

| Wave | Time(s) | OverMonsterCount | Type | Monster1 (interval) | Monster2 (interval) | BaseHP | BaseSpeed | BaseCcResist | BaseReward |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 30 | 90 | Normal | 1 (2.0s) | — | 100 | 2 | 1 | 10 |
| 2 | 30 | 90 | Normal | 1 (2.0s) | — | 110 | 2 | 1 | 10 |
| 3 | 30 | 90 | Normal | 1 (2.0s) | — | 120 | 2 | 1 | 10 |
| 4 | 30 | 90 | Normal | 1 (2.0s) | — | 130 | 2 | 1 | 10 |
| 5 | 30 | 90 | **Boss** | 100001 (—) | — | 140 | 2 | 1 | 10 |
| 6 | 30 | 90 | Normal | 2 (1.5s) | — | 150 | 2 | 1 | 10 |
| 7 | 30 | 90 | Normal | 2 (1.5s) | — | 160 | 2 | 1 | 10 |
| 8 | 30 | 90 | Normal | 2 (1.5s) | — | 170 | 2 | 1 | 10 |
| 9 | 30 | 90 | Normal | 2 (1.5s) | — | 180 | 2 | 1 | 10 |
| 10 | 30 | 90 | **Boss** | 100002 (—) | — | 190 | 2 | 1 | 10 |

Observations: every wave lasts a flat 30 seconds and allows up to 90 concurrent
monsters in the prototype config; BaseHP increases by a flat +10 each wave; boss
waves occur every 5th wave (matches the "5웨이브 주기로 보스 등장" rule stated in
인게임 사양.html §2-A-5), switching in a new (harder) regular monster type right after
each boss wave (Monster Id 1 for waves 1–4, Monster Id 2 for waves 6–9). This lines up
with the `ClearWave=10` value from the prototype mode row in 모드.html — the prototype
run is exactly these 10 waves.

---

## 8. Scoring / World-State (WS) System (source: `Doc/Project G html/점수.html`)

This table defines the **7 "WS" (world-state) score axes** that events (사건.html)
read from and write to, and that drive god pricing (신.html `CoreWSID`).

Schema: `Id` (int), `#Desc` (planning-only name), `MinValue`/`MaxValue` (int, clamps —
blank means unbounded).

### Full score list

| Id | Name (Korean) | Meaning | Min | Max |
|---|---|---|---|---|
| 1 | 전쟁 (War) | conflict/militarism level | 0 | 100 |
| 2 | 풍요 (Abundance) | prosperity/harvest level | 0 | 100 |
| 3 | 기술 (Technology) | tech advancement level | 0 | 100 |
| 4 | 문화 (Culture) | cultural development level | 0 | 100 |
| 5 | 인류 밀집 (Population Density) | how concentrated/urbanized humanity is | 0 | 100 |
| 6 | 신앙 (Faith) | religious devotion level | 0 | 100 |
| 7 | 인류의 수 (Population Count) | total population, used as the pricing baseline ("DefaultWs07" in mode config) | 20 | (unbounded) |

Cross-references from 인게임 사양.html §4-C describe qualitative relationships between
these scores (not strict formulas, more design intent notes):
- Rising **War (1)** raises Technology but lowers Abundance and Population Count.
- Rising **Abundance (2)** raises War and Technology, lowers Faith, raises Population Count.
- Every **Technology (3)**-related event lowers Faith and raises Abundance.
- **Population Count (7)** rising raises all god prices and raises War (more "currency" in circulation); falling does the reverse.
- **Faith (6)** rising raises all god prices and lowers Technology (again framed as "increased currency supply"); falling does the reverse.

Score IDs 1–3 are specifically called out elsewhere (인게임 사양.html §5-A) as the only
IDs usable as a god's `CoreWSID` ("Id 1~3만 코어 Id로 사용").

---

## 9. In-Game Spec — Master Systems Document (source: `Doc/Project G html/인게임 사양.html`)

This is the largest and most authoritative file (132KB) — it is the actual systems
design spec that ties together the tile map, monsters, waves, gods, events, oracle,
and pricing formulas described more loosely in the other files. It is organized into
5 numbered top-level sections.

### 9.1 Tile Map & Monster Paths (1. 타일맵 & 몬스터 경로)

**A. Basic map form**: A grid ("tile map") where tiles have visible gaps between
them; monsters travel through those gaps. The lines between tiles form the monster
path, and every monster path is a closed loop — monsters that complete the circuit
return to the entrance and go around again.

**B. Tile details**:
1. *Ownership states*: tiles are either owned (purchased with currency) or unowned.
   Only owned tiles can have a player's gods placed on them. Tile price shifts as the
   path configuration changes. UI behavior: clicking an owned tile shows its sell
   price and a sell button (selling grants currency equal to the current price and
   flips it to unowned); clicking an unowned tile shows a purchase UI (only usable if
   held currency ≥ price).
2. *Tile purchase price formula* — tiles cost more the closer they are to monster
   paths, and more the closer they are to the path's start point. Example formula
   given in the doc:
   `Price = 100W + 100/S + 6A + 3B + C`
   where:
   - `W` = current wave number
   - `S` = straight-line distance from the tile's center to the path start point
   - `A` = count of the tile's 4 directly-adjacent edges that are monster path segments
   - `B` = count of monster path segments touching the tile's surrounding "5-tile cluster"
   - `C` = count of monster path segments touching the tile's surrounding "13-tile cluster"
   - The constants (100, 6, 3, 1) are meant to be table-driven/tunable, not hardcoded.
3. *Tile sell price formula*:
   `SellPrice = PurchasePrice × ((50 + (10 × min(tileHeldWaves, 5))) / 100)`
   — i.e., sell recovers 50% of purchase price minimum, rising by 10 percentage
   points per wave held, capped at 5 waves (so max recovery is 100% at 5+ waves held).

**C. Monster path generation**:
1. *Initial path generation* (procedural, at run start):
   - Step 1: pick a random start point at a tile-grid intersection.
   - Step 2: pick a path length (must be even; measured in tile-edge segments; only
     set once at the start, later changes don't need to re-derive this).
   - Step 3: path-drawing algorithm from the start coordinate `(x,y)`:
     - At each vertex, compute candidate directions (up/down/left/right); direction
       search order is randomized each step.
     - Reject a candidate direction if `shortest remaining distance back to the start
       (excluding already-visited coords) > remaining allowed path length`.
     - Reject if remaining length ≥ 2 but the candidate target is the start point
       (i.e., can't close the loop early).
     - Reject if the candidate target coordinate has already been visited.
     - Repeat until the remaining segment count is 1 and the candidate equals the
       start — that closes the loop.
   - Step 4: monsters always move **clockwise** from the start point along the
     generated loop.
2. *Path/start-point drift every 5 waves*:
   - Check current path tile list.
   - Remove 1–3 tiles from the path's outer edge (count is table-driven min/max,
     randomized in range); only tiles touching the path's exterior are eligible, and
     tiles adjacent to the current start point are never removed.
   - Add 1–3 tiles to the path from tiles adjacent to the (possibly just-shrunk)
     remaining path tile list; never re-add just-removed tiles, and never add tiles
     touching the current start point.
   - Validate the resulting shape: (a) must remain a single connected blob — if not,
     the whole procedure restarts from scratch; (b) must not contain an internal hole
     — if so, also restart.
   - Recompute the path loop around the outside edge of the resulting tile blob.
   - Move the start point to a random location 1–2 tiles from its old position, along
     the new path.
   - Implementation note: for the "live" reshuffle case, treat this as: define the
     shape of the tile blob, then trace its outline. Any monster left outside the new
     path after a shift snaps to the nearest point on the new path.

### 9.2 Monster Spawning & Wave Progression (2. 몬스터 생성 및 웨이브 진행)

**A. Wave overview**:
1. **Time-limited waves**: waves advance on a fixed timer (`WaveTime`) regardless of
   whether the field has been cleared of monsters.
2. **Monster spawning**: on wave start, monsters spawn sequentially from the "start
   point" per the active balance table.
3. **Monster movement**: monsters use their table-defined base stats (HP, speed,
   etc.), and travel clockwise around the loop indefinitely from the start point.
   - Monsters heal **10% of max HP** every time they pass the start point.
4. **Wave transitions**: when `WaveTime` elapses, the game immediately switches to
   the next wave's config and resumes spawning from the start point.
5. **Boss waves**: occur on a 5-wave cycle; bosses have unique special skills.
6. **Game over conditions**:
   - Always active: total monsters on the field exceeds `OverMonsterCount`.
   - Boss-wave only: the boss is not killed within the wave's time limit.

**B. Mode management**: mode data (wave length / mode name / which balance file to
use / other settings) is chosen out-of-game and locked once in-game play begins;
full schema lives in the "모드" sheet (see §2 above).

**C. Wave management**: per-mode wave data lives in files named
`Wave{ModeName}.csv`; contains per-wave base HP/speed/resist/reward and per-wave
monster spawn lists/intervals; full schema in the "웨이브" sheet (see §7 above).

**D. Monster management**: per-monster base HP%/speed%/resist% plus a trait-type flag
system (e.g. flying units) plus monster-skill references — monster-skill data is
explicitly flagged in the doc as **deferred post-prototype** ("해당 내용은 추후
프로토 타입 이후에 추가"). Full schema in the "몬스터" sheet (see §3 above).

**E. Monster stat resolution formulas** (this is the authoritative version, matching
§3 above but spelled out precisely):
- `HP = Round(Wave.BaseHp × (Monster.HpRate/100))` — monster dies at HP ≤ 0.
- `Speed` = time (in units of ×100, i.e. centiseconds) to cross one tile-edge segment:
  `Round(Wave.BaseSpeed × (Monster.SpeedRate/100))`. Example: a Speed value of 10
  means the monster crosses one tile-edge in 0.1s (actual time = Speed/100 seconds).
- `Resist` = `Round(Wave.BaseResist × (Monster.ResistRate/100))` — the percentage of
  a slow/stun's effect duration actually applied. Example: Resist 80 means only 80%
  of an incoming slow/stun's duration applies.
- `RewardAmount = Round(Wave.BaseAmount × (Monster.ResistAmount/100))` — currency
  granted per kill. Example: value 10 → 10 currency per kill.

### 9.3 God (Tower) Information (3. 신(타워) 정보)

- Owned gods can be freely placed on any tile the player owns.
- Gods auto-attack using an attack power stat.
- A god's attack power scales only with owned share count and current price:
  `currentAttackPower = BaseDamage × ownedShareCount × (currentPrice / BasePrice)`
  (matches the formula surfaced in §6a above).

### 9.4 Oracle & Mortal-World Status (4. 신탁 및 하계 현황)

**A. Overview**:
1. Currently-listed gods' prices rise/fall based on the "mortal world status."
2. This status updates every wave, and each update repricing gods accordingly.
3. Before each wave starts, the mortal-world score list is (re-)seeded within a
   random range.
4. Critically: **the entire run's sequence of events is predetermined before the game
   starts** — but each event's actual numeric price impact on gods is computed only
   when that event actually fires (not precomputed).

**B. Event list**: (full schema/data documented in §4 above / 사건.html) — this
section just states that the event list defines: spawn-eligibility conditions,
spawn-probability-affecting scores, world-state score deltas, and god-price deltas
per event.

**C. World-state score list**: (full data in §8 above / 점수.html) — the same
qualitative War/Abundance/Tech/PopulationCount/Faith relationships summarized there.

**D. Game-start flow**:
1. Each world-state score is seeded to a random value within its configured range.
2. Wave-1 events are generated as follows:
   a. The total number of events to generate across the whole run-to-clear is
      randomized within `[WaveEventMin, WaveEventMax]` (from the mode config).
   b. One event is selected from the event list via weighted random selection:
      `eventProbability = eventWeight / sum(all eligible eventWeights)`; **no
      duplicate events within a single wave** are allowed.
   c. The chosen event's score deltas and metadata are stored.
   d. Steps repeat until every wave's events for the whole run are generated.

**E. Per-wave event execution flow**:
1. Compute `waveTime / numberOfEventsThisWave` to get the interval between events
   within the wave.
2. The wave's first event fires immediately at wave start, applying its score/price
   deltas.
3. After the computed interval elapses, the next event fires; repeat until all of the
   wave's events have fired.

**F. Event-appearance probability calculation** (3-step process):
1. **Filter step**: given current scores, remove any events whose gating condition
   (`ConditionWSId`/`Op`/`Value`) is not currently satisfied.
2. **Weight calculation**: for each remaining eligible event, compute
   `weight = max(0, BaseWeight + WS01.currentValue × CauseValue01 + WS02.currentValue × CauseValue02)`
   (floor at 0 if negative).
3. **Probability assignment**: `probability = event.weight / sum(all eligible weights)`
   (with the no-duplicate-per-wave rule noted again).

**G. Score update calculation upon an event firing**:
1. Check which world-state scores the fired event(s) affect (`EffectWSId`/`Value`).
2. Apply: `newScore = currentScore × (100 + value) / 100`.
   (Note: this multiplicative form appears here, while §4-B/§8 examples describe
   additive deltas like "current score + delta" — the doc is **internally
   inconsistent** between additive and multiplicative wording for WS effects; treat
   this as a design ambiguity to resolve with the design team before implementing,
   rather than silently picking one.)
3. Clamp the result to the score's configured Min/Max.

**H. Oracle (신탁) selection**:
1. Every wave start, the player may receive an Oracle preview.
2. The first Oracle each wave is free; additional Oracle lookups cost Incense.
3. Oracles come in **3 rarity tiers** × **2 content types**.
4. **Content types**:
   - *부동산 정보 (Real-estate info)*: reveals monster path info at a specified future
     point — not all paths, only a limited number (3–5) of path segments/tiles.
   - *사건 정보 (Event info)*: reveals info about 1 upcoming event at a specified
     future point.
5. **Rarity tiers** (all affect *timing precision*, not outcome certainty):
   - **일반 (Common)**: wide time-window (low precision on *when* it happens).
   - **레어 (Rare)**: medium time-window (medium precision).
   - **유니크 (Unique)**: exact — tells you the precise wave the event/change occurs on.

### 9.5 God Sale-Price Calculation (5. 신 판매 가격 계산 방식)

**A. Initial price-setting formula** (computed once, likely at listing/IPO time):
1. Pull the god's `BasePrice`, `CoreWs` (id), and `CoreWsCorrelation`.
2. Compute an initial multiplier from population and faith scores:
   `M = (currentPopulation / baselinePopulation) × (initialFaithValue / baselineFaithValue)`
   (using WS ids 4 and 5 per the doc annotation — though this looks like it may
   actually intend WS ids 7 (population) and 6 (faith) based on §1/§8 semantics;
   the doc literally says "ID 4 / 5번 값" — flag this as another spot to confirm with
   design before coding, since 점수.html's actual ids for Population Count and Faith
   are 7 and 6 respectively, not 4/5).
3. Set the first price based on WS correlation:
   - If `WSCorrelation = 0` (positive correlation):
     `price = BasePrice × (100 + CoreWs.currentValue − CoreWs.midpointValue)/100 × M`
   - If `WSCorrelation = 1` (negative correlation):
     `price = BasePrice × (100 − CoreWs.currentValue + CoreWs.midpointValue)/100 × M`
   - (Only WS ids 1–3 — War/Abundance/Technology — are valid as a god's `CoreWSID`,
     per the doc's explicit note.)

**B. Price recalculation after an event fires**:
1. Start from the current price (`A`).
2. Check whether any fired event(s) include this god in one of their
   `EffectGodGroup01/02/03[]` lists.
3. If affected, look up that group's `EffectGodValueXX` as `C` (else `C = 0`).
4. Look up the current value of the god's `CoreWSID` as `B`, noting whether the
   correlation is positive or negative.
5. Compute a `CoreWs` variable `D`:
   - If `WsCorrelation = 0` (positive): `D = (100 + B − midpoint(WS.Min,WS.Max)) / 100`
   - If `WsCorrelation = 1` (negative): `D = (100 − B + midpoint(WS.Min,WS.Max)) / 100`
6. Compute a fresh multiplier `M` (same population/faith formula as the initial
   pricing step, but using post-event population vs. pre-event population).
7. **Final formula**: `newPrice = A × (1 + (C/100)) × M × D`

This closes the loop between the event system (§4/§9.4), the world-state scores
(§8/§9.4-C), and per-god pricing (§6a/§9.3/§9.5) — this is the actual economic engine
of the game and should be the first thing modeled/prototyped in code, since nearly
every other system (Oracle, tile pricing indirectly via wave number, event weighting)
feeds into or reads from it.

---

## Summary of Known Gaps / Inconsistencies for Future Implementers

- **Monster roster is a stub**: only 4 example monster rows exist (몬스터.html);
  monster-skill data is explicitly deferred post-prototype.
- **Mode table has only 1 populated mode** (the prototype); Hell Mode and any other
  modes described conceptually in 개요.html have no data rows yet.
- **Gods 11+ (Hera, Hephaestus, Hestia, Dionysus, Persephone, etc.) have no stat/skill
  data**, only lore text in 타워(신) 리스트.html and passing UI-mockup mentions in
  개요.html.
- **Demeter/Athena/Apollo/Hermes skill ID references in 신.html look like copy-paste
  errors** (repeated skill IDs / a likely typo `1090003`) — cross-check against the
  actual distinct skill IDs defined in 스킬.html before trusting 신.html's references
  literally.
- **WS-score effect formula is stated two different ways** in 인게임 사양.html:
  additive (`current + delta`) in some places (§4-B, and implicitly in the worked
  사건.html examples) vs. multiplicative (`current × (100+value)/100`) in §9.4-G.
  Needs design clarification before implementation.
- **Initial god-pricing formula references "ID 4/5" for population/faith** (§9.5-A)
  which doesn't match 점수.html's actual ids (7 = population, 6 = faith) — needs
  design clarification.
- **타워(신) 리스트.html's Demeter entry duplicates Zeus's description text** verbatim
  (evident copy-paste error), and pantheon entries beyond Athena (#5) have no filled
  detail in the extracted content — worth checking the source PDF for whether more
  detail exists there.
