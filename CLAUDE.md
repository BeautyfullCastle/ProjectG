# Project G — Game Design Reference

**Status: design-documentation-only repository.** There is no source code yet — only design docs under `Doc/`. Before implementing any feature, read the relevant section below and/or the source file it references, since numeric balance values and exact mechanics live in the spreadsheet exports, not in this summary.

## Source documents

- `Doc/Project G.pdf` — full design doc with images (same content as the HTML export below, easier to skim visually).
- `Doc/Project G html/` — Google Sheets tabs exported to HTML (one file per sheet tab), each a spec table:
  - `개요.html` — Overview (concept, core systems, play loop, session flow)
  - `모드.html` — Mode (difficulty/mode config table schema)
  - `몬스터.html` — Monster (monster stat table schema)
  - `사건.html` — Event (Human Realm event table, ~50+ example rows)
  - `스킬.html` — Skill (per-god skill table, ~30 example rows)
  - `신.html` — God (tower/god stat table schema, 10 example rows)
  - `웨이브.html` — Wave (wave/spawn schedule table schema)
  - `인게임 사양.html` — In-game Spec (largest file: tile map, monster movement, tower pricing, oracle system, event probability math — the mechanical "how it's calculated" doc)
  - `점수.html` — Score (Human Realm Score/WS axes)
  - `타워(신) 리스트.html` — Tower(God) List (Greco-Roman mythology flavor/lore reference for each god)
  - `resources/` — cell images referenced by the sheets, `sheet.css`

## 1. Concept

Project G is a **tower defense game built on top of a stock-trading / real-estate-trading simulation layer**. Gods from mythology (Greco-Roman pantheon in the current design pass — Zeus, Poseidon, Hades, Demeter, Athena, Apollo, Artemis, Aphrodite, Hermes, Ares, etc.) act as both **towers** and **tradeable stocks**.

Core pitch (from 개요.html):
- Buy/sell "shares" of gods, place them as towers to defend against waves of monsters.
- An oracle system (신탁) gives partial, obscured previews of future world events, letting the player speculate: buy gods whose price is about to rise, sell ones that have peaked, and roll the proceeds into the next round with a bigger war chest.
- It's a defense genre, but monster pressure ramps up slowly on purpose, giving players room to think about investment decisions mid-combat.

## 2. Core systems (from 개요.html)

### 2.1 Gods (신) & Godheads (신격체)
- **God (신)**: fills the tower role. Purchased by the "share" (주) — attack power scales with `shares held × price per share`. Special stats unlock in tiers as share count rises; each god has a fixed total share issuance. Gods have distinct kits meant to be combined (e.g. an AoE-damage god + a single-target-damage god).
- **Godhead (신격체)**: a candidate that *could* become a god — i.e. an unlisted/private stock. Cannot be placed as a tower while in this state. Can be "enshrined" (봉신) into a listed god depending on play conditions.

### 2.2 Currency & Exchanges
- **향화 (Incense/Offering)**: the base currency; buys everything. Price of any god/godhead rises as more people "believe in / worship" them. Earned by killing monsters or by trading stocks.
- **GADSDAQ (갓스닥)**: exchange for listed (상장) gods. Real-time buy/sell during play; price moves react to events in real time; a bought god is immediately deployable.
- **UNDER-DAQ (언더닥)**: exchange for unlisted (비상장) godheads. If an unlisted godhead's value stays above a threshold long enough, it "enshrines" (봉신) — becomes a god, moves to GADSDAQ, and its value jumps by N%.
- Trading fee (거래 수수료) applies on GADSDAQ.

### 2.3 Sacred Realm (신성 영역/에덴) & Shrines (신전)
- The **Sacred Realm** is the actual battlefield — a grid ("바둑판") of shrine tiles. Monster spawn point and path run along the connections between shrine tiles, forming a loop.
- Monster path shifts by N tiles every time a calamity (제앙) occurs.
- Loss conditions: too many monsters piled up in the Sacred Realm, or failing to kill the calamity/boss (제앙) within its time limit.
- A **shrine (신전)** is a single tile. Purchasable with 향화; price floats based on:
  - proximity to the monster path
  - buffs — a buffed shrine gives buffs to the god placed on it, and the buff zone itself relocates randomly within N tiles whenever a calamity occurs
- Shrines can be sold; sell fee decreases the longer you've held through waves.

### 2.4 Oracle (신탁) & Human Realm Events (인간계 이슈)
- As play time passes, **Human Realm events** are generated, which raise/lower god values.
  - Example: large flood/lightning storm → raises value of Noah/Zeus/Thor-type gods.
  - Example: war breaks out → raises war gods (Ares, Achilles), lowers wealth/capital-associated gods.
- Event generation is random but weighted/contextual — e.g. a war event is typically followed (after a delay) by a peace-themed event, then a tech-development-themed event.
- N events are generated per time unit; all users see them and god prices update in real time; the same event can produce different effects depending on which other events co-occur.
- **Oracle (신탁)**: previews upcoming Human Realm events ahead of time. Doesn't reveal all events — only a limited count, and only partially (text obscured). Spending 향화 can increase oracle accuracy or the number of revealed events.

### 2.5 Play loop (플레이 루프 예시)
1. **Round start (randomized each round for replay variety):**
   - Listed god list (randomized; unlocks more god variety as the player progresses across meta-runs)
   - Unlisted godhead list (fixed pool size; when one lists, a new random godhead refills the pool)
   - Starting political/world situation (정세) — randomized, influences later event probabilities
   - Monster path/spawn point — randomized on the grid; the 4 tiles nearest the spawn point are granted free as starting real estate
2. **During play:** monsters spawn continuously on the path; player buys/places gods to hold them off before the loss condition triggers; Human Realm events and oracle previews update live; pacing is deliberately gentle to leave room for investment decisions.
3. **Game end:**
   - Standard mode: run ends at the wave the player couldn't survive; that wave is the record.
   - Also: a staged/cleared mode (tutorial-friendly, gives a clear pass/fail) + a "Hell" endless mode for score-chasing veterans.
   - On clear, rewards can include: owned gods becoming eligible to appear pre-listed in future runs, and permanent collection-list entries unlocked by meeting per-god conditions in-game (mitigates loss of character progression between runs).

## 3. Data tables (schemas + example rows)

These are Google Sheets tabs meant to become CSV data tables. Columns prefixed `#` are designer-only notes, excluded from the exported CSV.

### 3.1 God / Tower (신.html)
Columns: `Id, #Name, BaseDamage, AttackSpeed(atk/s×10), AttackRange, AttackType(0=Normal-frontmost on path,1=Finisher-lowest HP,2=Strong-highest max HP,3=Random), Skill01Id/Qty, Skill02Id/Qty, Skill03Id/Qty, BasePrice, MaxQuantity, CoreWSID, WSCorrelation(0=positive,1=negative), ST_GodName, ST_GodDesc, Path_GodPrepab`

Example gods (id / dmg / atkspd / range / type / basePrice / maxQty / coreWS / correlation):
| Id | Name | Dmg | AtkSpd | Range | Type | BasePrice | MaxQty | CoreWS | Correlation |
|---|---|---|---|---|---|---|---|---|---|
| 1 | Zeus (제우스) | 10 | 10 | 2 | Normal | 500 | 50 | 5 | + |
| 2 | Poseidon (포세이돈) | 15 | 5 | 4 | Normal | 500 | 50 | 2 | − |
| 3 | Hades (하데스) | 10 | 12 | 3 | Finisher | 500 | 50 | 1 | + |
| 4 | Demeter (데메테르) | 10 | 5 | 4 | Normal | 400 | 60 | 2 | + |
| 5 | Athena (아테나) | 10 | 10 | 1 | Normal | 400 | 60 | 4 | + |
| 6 | Apollo (아폴론) | 10 | 10 | 5 | Normal | 400 | 60 | 4 | + |
| 7 | Artemis (아르테미스) | 5 | 20 | 3 | Strong | 400 | 60 | 2 | + |
| 8 | Aphrodite (아프로디테) | 10 | 10 | 3 | Normal | 400 | 60 | 5 | + |
| 9 | Hermes (헤르메스) | 5 | 20 | 3 | Normal | 1000 | 20 | 3 | + |
| 10 | Ares (아레스) | 10 | 13 | 2 | Finisher | 400 | 60 | 1 | + |

`AttackRange` × the adjacent-tile-center distance = radius of a circular attack range.

Live attack power formula (인게임 사양.html §3): `현재 공격력 = BaseDamage × 보유 주식 수 × (현재가격/BasePrice)` — i.e. damage scales with both share count *and* current market price relative to base price.

### 3.2 Skills (스킬.html)
Id format: `1{godId}{skillIndex}` (e.g. `1001001` = god 1's skill 1). Columns: `Id, #Desc, SkillType(0=Passive,1=Active), ConnectSkillId(upgrade target if this skill enhances another), SkillBaseValue01-04, Path_Icon, St_SkillName, St_SkillDesc`.

Each god has a base skill (Skill01, unlocked at 0 shares) plus two "collection" skills (Skill02/03) that unlock at higher share-count thresholds (see the `신` table's `SkillNN Quantity` columns, e.g. Zeus's collect skills unlock at 10 and 40 shares).

Sample kits:
- **Zeus** — base: basic attack also hits 2 nearby random enemies. Collect 1: every 3rd hit has 50% chance to instakill. Collect 2: basic attack hits 5 nearby enemies.
- **Poseidon** — base: every 10s, summons a tidal wave from the spawn point moving backward along the path, piercing 5 monsters, stunning 1s, dealing 500% attack as damage. Collects: wave moves faster + retriggers at path end; pierce count +5.
- **Hades** — base: collects a soul per kill; at 30 souls, deals N% damage to all monsters on field and consumes the souls. Collects: +N% attack per soul held (base, non-compounding); soul threshold +20 and +1 attack speed per soul.
- **Demeter** — base ("wheat field"): monsters in range move 30% slower. Collects: 50% chance to double currency drop from kills in range; +10% bonus currency at wave end.
- **Athena** — base: gods in range get +10% attack. Collects: +1 range and further +10% attack in range; gods in range get +1 range.
- **Apollo** — base: attacks apply a burn dealing 5% attack/sec (max 3 stacks). Collects: hitting a burning target extends +10% damage for 5s (max 3 stacks); max stack count for all Apollo skills +5.
- **Artemis** — base: repeated hits on the same target deal +5% increasing damage. Collects: +1 range, 30% chance for +50% damage; +1 range, damage bonus persists across targets.
- **Aphrodite** — base: enemies in range take +10% damage. Collects: deals 20% attack/sec to monsters in range; every 10s, pulls 5 random off-screen monsters into range.
- **Hermes** — base: oracle purchase price −20%. Collects: +30% chance of higher-rarity oracle; +30% attack per oracle held.
- **Ares** — base: +10 attack speed if no other god within 10 tiles. Collects: +3% attack per enemy in range; every 10s, +2 range for 3s.

### 3.3 Monster (몬스터.html)
Columns: `Id, MonsterTypes(bitmask, 2^n per trait), HpRate, SpeedRate, ResistRate, RewardAmountRate (all % of that wave's Base* stats), Path_MonsterPrepab`. Boss monster IDs start at `1000001`.

Trait bitmask examples: `1`=Evade (dodges basic attacks N% chance), `2`=Dash (dashes forward every Ns, ignores CC), `4`=Absorb (heals 2% HP when nearby monster dies), `8`=Lingering (stays on field N sec after death, untargetable, still counts toward monster cap). Bitmask 6 = Dash+Absorb.

Actual stat formulas (인게임 사양.html §2E):
- `HP = Round(Wave.BaseHp × Monster.HpRate/100)`
- `Speed(×100, time to cross 1 tile-segment in seconds) = Round(Wave.BaseSpeed × Monster.SpeedRate/100)`
- `Resist(% of CC/slow duration actually applied) = Round(Wave.BaseResist × Monster.ResistRate/100)`
- `RewardAmount = Round(Wave.BaseRewardAmount × Monster.RewardAmountRate/100)`

### 3.4 Wave (웨이브.html)
Columns: `WaveNumber, WaveTime(s), OverMonsterCount(gameover threshold, default 1), WaveType(Normal|Boss), Monster01Id/SpawnInterval, Monster02Id/SpawnInterval, BaseHP, BaseSpeed, BaseCcResist, BaseRewardAmount`.

Sample progression (prototype balance): 30s/wave, OverMonsterCount 90, BaseHP climbing +10/wave (100→190 over waves 1–10), boss waves at 5 and 10 (monster IDs `100001`, `100002`), normal monster spawn interval tightens from 2s → 1.5s after wave 5.

### 3.5 Mode (모드.html)
Per-mode config: `Id, #Name, ST_ModeName, Path_BalanceName, StartCoin, DefaultWs07(baseline population), StartWSGroup01/Min/Max, StartWSGroup02/Min/Max, WaveEventMax/Min(events per wave range), ClearWave`. Example: prototype mode — 2000 start coin, pop baseline 100, WS group 1 = scores [1..6] ranged 40–60, 2–4 events/wave, clears at wave 10.

### 3.6 Score / World-State axes (점수.html)
Six score IDs, each 0–100 unless noted, driving both event eligibility and god pricing:
1. 전쟁 (War)
2. 풍요 (Prosperity)
3. 기술 (Technology)
4. 문화 (Culture)
5. 인류 밀집 (Population density)
6. 신앙 (Faith)
7. 인류의 수 (Population count) — MinValue 20, no max

Interactions noted in 인게임 사양.html §4C: War↑ → Tech↑, Prosperity↓, Population↓. Prosperity↑ → War/Tech↑, Faith↓, Population↑. Tech↑ (per tech event) → Faith↓, Prosperity↑. Population↑ → all god prices↑, War↑ (more currency in circulation); Population↓ → all god prices↓. Faith↑ → all god prices↑, Tech↓; Faith↓ → all god prices↓.

### 3.7 Human Realm Events (사건.html)
~50 example events (id, description, conditions, weight, score effects, delayed effects, god-group price effects). Columns: `Id, #Desc, ConditionWSId/Op/Value (gates whether event is eligible), Baseweight, CauseWSId01-02/Value (weight modifiers), EffectWSGroup01-02[]/Value (immediate score deltas), AfterEffectWave/WSGroup[]/Value (delayed score deltas N waves later), EffectGodGroup01-03[]/Value (immediate god price multipliers, applied as (100+Value)/100), St_CaseDesc`.

Representative rows:
- "소규모 분쟁" (minor conflict) — weight 50, boosts War, later raises gods 5&10 by 10%.
- "메뚜기떼" (locust swarm) — Prosperity/Population/Faith −20, Tech +10 (delayed), Culture god −10, Tech god +5.
- "축제 성황" (festival) — gated on Prosperity ≥80, weight 100; raises Culture, delayed Population effect; several god groups ±10.
- "대규모 전쟁 발발" (major war) — gated on War ≥70, weight 100; War +20, Prosperity/Faith/Prosperity −20 (delayed), god 10 (Ares) +20%.
- Full list covers wars, plagues, medical breakthroughs, temples/libraries built, philosophers, eclipses, festivals, droughts, corruption scandals, etc.

### 3.8 In-game spec highlights (인게임 사양.html)

**Tile map & monster path**
- Grid of shrine tiles with gaps between them; the gaps between tiles are the monster path; all paths loop back to the entrance.
- Tile purchase price formula (example constants, tunable): `Price = 100×W + 100/S + 6×A + 3×B + C` where `W` = current wave, `S` = straight-line distance to path start, `A` = # of the tile's 4 adjacent edges that are monster path, `B` = path-adjacency count within a 5-tile neighborhood, `C` = within a 13-tile neighborhood.
- Tile sell price: `purchase price × (50 + 10×min(waves held,5)) / 100`.
- Path generation: random start point on tile-grid vertices; even-numbered path length chosen up front; path drawn via randomized up/down/left/right walk avoiding premature loop-closure and revisits; monsters move clockwise from the start point.
- Every 5 waves, the path partially reshapes: 1–3 tiles removed from the path's edge (not touching current start), 1–3 new tiles added (adjacent to remaining path, not touching start), then validated (no disconnected chunks, no interior holes) and re-enclosed; start point shifts 1–2 tiles along the new path. Monsters outside the new path snap to the nearest path point.

**Wave/monster loop**
- Time-limited waves: a new wave force-starts at `WaveTime` regardless of whether the field was cleared.
- Monsters heal 10% max HP each time they pass the start point (loop).
- Boss wave every 5 waves; bosses have unique skills.
- Game over: total on-field monster count exceeds `OverMonsterCount`, OR (boss wave only) boss not killed within the wave time.

**God pricing**
- Initial price: `M = (current population / baseline population) × (initial faith / baseline faith)`; then `Price = BasePrice × (100 + CoreWS_current - CoreWS_midpoint)/100 × M` if WSCorrelation is positive, or the mirrored `(100 - CoreWS_current + CoreWS_midpoint)/100` if negative.
- Price after an event: gather every god-price-affecting event's value into `C` (0 if none apply), recompute the correlation-adjusted `D` from the relevant CoreWS's current value, recompute `M` from updated population/faith, then `NewPrice = OldPrice × (1 + C/100) × M × D`.

**Event selection**
1. Filter out events whose `ConditionWS` gate isn't met.
2. Weight = `max(0, BaseWeight + CauseWS01.current×CauseValue01 + CauseWS02.current×CauseValue02)`.
3. Probability = event weight / sum of all eligible weights; no duplicate events within the same wave.
- All of a run's events for the whole session are pre-rolled before the run starts (god price effects are computed live when the event actually fires, not pre-rolled).

**Oracle (신탁)**
- One free oracle reveal per wave; additional reveals cost 향화.
- Two content types: property/path info (reveals 3–5 upcoming path segments) or event info (reveals 1 upcoming event).
- Three rarity tiers controlling timing precision: 일반(Common, wide/vague timing window), 레어(Rare, medium precision), 유니크(Unique, exact wave revealed).

## 4. Mythology reference (타워(신) 리스트.html)

Current pass covers the Greek pantheon (Zeus, Hera, Poseidon, Demeter, Athena documented in detail so far — entries include title/domain, signature weapon, and iconography), plus a name-mapping table to Roman equivalents for the twelve Olympians (Zeus/Jupiter is implied as the reference point):

| Greek | Roman | Relation to Zeus |
|---|---|---|
| Demeter | Ceres | Zeus's second-oldest sister |
| Athena | Minerva | Zeus's legitimate eldest daughter |
| Apollo | Apollo | Zeus's illegitimate son |
| Artemis | Diana | Zeus's illegitimate daughter |
| Ares | Mars | Zeus's legitimate second son |
| Aphrodite | Venus | Zeus's youngest aunt, or illegitimate daughter |
| Hephaestus | Vulcan | Zeus's legitimate eldest son |
| Hermes | Mercury | Zeus's illegitimate son |
| Hestia | Vesta | Zeus's eldest sister |
| Dionysus | Bacchus | Zeus's illegitimate son |
| Hades | Pluto | Zeus's eldest brother and son-in-law |
| Persephone | Proserpina | Zeus's illegitimate daughter |

Note: Hades and Persephone are considered peers of the Twelve Olympians in stature but are usually excluded from the canonical "12" since they reside in the underworld, not Olympus.

## 5. Notes for future implementation

- All balance numbers throughout the docs (prices, HP rates, weights, skill values) are **prototype placeholders** explicitly called out as tunable via the linked table files — don't treat any specific number as final game balance.
- Table schemas describe an intended CSV pipeline: designer sheets (Google Sheets) → CSV export (dropping `#`-prefixed columns) → consumed by game code. When source code is eventually added to this repo, the data layer should likely mirror this table/CSV structure (Mode, Wave{ModeName}, Monster, God, Skill, Event, Score) rather than inventing a new schema.
