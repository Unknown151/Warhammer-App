# Warhammer App — Codebase Reference

Single-page web app for building and displaying Leagues of Votann army lists for Warhammer 40K. All logic, rendering, and styling lives in **`index.html`** (~2250 lines). Unit data, abilities, and enhancements live in **`data/`** as JSON files.

---

## File Structure

```
index.html                          All JS, CSS, and HTML
data/
  abilities.json                    Ability definitions (weapon, core, wargear, leader, detachment)
  enhancements.json                 Enhancement definitions (grouped by detachment)
  army.json                         Reference pre-built army list (used as default)
  units/
    kahl.json
    einhyr-champion.json
    uthar.json
    iron-master.json
    grimnyr.json
    memnyr-strategist.json
    arkanyst-evaluator.json
    buri-aegnirssen.json
    berehk-stornbrow.json
    einhyr-hearthguard.json
    hearthkyn-warriors.json
    thunderkyn.json
    cthonian-beserks.json
    cthonian-earthshakers.json
    hernkyn-pioneers.json
    hernkyn-yaegirs.json
    ironkin-steeljacks-heavy-volkanite.json
    ironkin-steeljacks-melee.json
    hekaton-land-fortress.json
    sagitaur.json
    kapricus-carrier.json
    kapricus-defenders.json
```

---

## Global Variables (index.html)

### Loaded Data
| Variable | Type | Source | Purpose |
|---|---|---|---|
| `abilities` | `Object` | `data/abilities.json` | Keyed ability definitions, looked up by ID |
| `enhancements` | `Object` | `data/enhancements.json` | Keyed enhancement definitions |
| `army` | `Object` | `data/army.json` | Default/reference army list |
| `units` | `Object` | `data/units/*.json` | All unit definitions keyed by unit ID |

### Army State
| Variable | Type | Default | Purpose |
|---|---|---|---|
| `armyList` | `Array<Instance>` | `[]` | Active army list — all added unit instances |
| `nextInstanceId` | `Number` | `1` | Auto-increment ID for new instances |
| `selectedDetachment` | `String` | `'hearthband'` | Currently active detachment ID |

### Buff/Stratagem State
| Variable | Type | Default | Purpose |
|---|---|---|---|
| `stratagemState` | `Object` | see below | Current toggle state of all stratagems |
| `furyYP` | `Boolean` | `false` | Whether YP was paid for Fury of the Hearth; gates Sustained Hits 1 |

```javascript
stratagemState = {
  fury:     false,  // Fury of the Hearth — Hearthband only
  closest:  false,  // Targeting Closest — Hearthband only
  firebase: false   // Firebase Control — Brandfast Oathband only
}
```

### UI State
| Variable | Type | Purpose |
|---|---|---|
| `characterState` | `Object` | `{ leaderId: bool }` — tracks slain leaders |
| `bodyguardState` | `Object` | `{ leaderId: bool }` — tracks wiped bodyguards |
| `expandedCards` | `Object` | `{ cardId: bool }` — accordion state for units tab |
| `configExpandedCards` | `Object` | `{ instanceId: bool }` — accordion state for config tab |
| `vp` | `Number` | YP (Yield Points) counter |
| `cp` | `Number` | CP (Command Points) counter |

### Constants
| Constant | Type | Purpose |
|---|---|---|
| `ALL_UNIT_IDS` | `Array<String>` | All unit IDs to load from JSON files |
| `UNIT_GROUPS` | `Object` | Groups unit IDs for the unit picker: `Characters`, `Infantry`, `Vehicles & Mounted` |
| `DETACHMENTS` | `Object` | Detachment definitions (see Detachment System) |

---

## Data Schemas

### Unit JSON (`data/units/<id>.json`)

```jsonc
{
  "id": "einhyr-hearthguard",          // matches filename, used as key in units{}
  "name": "Einhyr Hearthguard",        // display name
  "type": ["Infantry", "Exo-armour"],  // unit type labels (display only)
  "keywords": ["INFANTRY", "EXOARMOUR", "EINHYR HEARTHGUARD"],
  "isLeader": false,                   // can this unit be used as an attached leader?
  "isEpicHero": false,                 // Epic Heroes cannot take enhancements
  "isBattleline": false,               // display flag (optional)
  "points": 135,                       // base points (for baseModels count)
  "models": 5,                         // default starting model count
  "baseModels": 5,                     // minimum model count
  "maxModels": 10,                     // maximum model count
  "modelBreakdown": "1 Hesyr + 4 Hearthguard",  // flavor text (optional)

  "stats": {
    "m": "5\"",  "t": 5,  "sv": "2+",  "w": 2,  "ld": "7+",  "oc": 1
  },

  // For units with multiple different statlines (e.g. Iron-master + Ironkin):
  "modelProfiles": [
    { "name": "Iron-master", "count": 1, "stats": { ... } },
    { "name": "Ironkin Assistant", "count": 1, "stats": { ... } }
  ],

  "invulnerableSave": "5+",            // displayed in invuln row (null if none)
  "invulnerableSource": "weavefield-crest",  // ability ID that grants it (optional)
  "transportCapacity": 14,             // TRANSPORT units only
  "damagedProfile": "1-5",            // degraded profile wound range (optional)

  "weapons": {
    "ranged": [
      {
        "name": "EtaCarn plasma gun",
        "count": 5,                    // how many models have this weapon
        "range": "24\"",
        "a": "1",  "bs": "3+",  "s": "7",  "ap": "-3",  "d": "2",
        "abilities": ["rapid-fire"],   // ability IDs shown as weapon ability badges
        "inheritedAbilities": [],      // set by leader bonus system — do not put in JSON directly
        "model": "Hesyr",              // which model type has this weapon (optional)
        "weaponGroup": "l7-launcher",  // groups weapons into a choice block (optional)
        "weaponGroupLabel": "Choose one profile per shooting phase"  // choice block label
      }
    ],
    "melee": [ /* same structure, no "range" field */ ]
  },

  "abilities": ["decisive-destruction"],      // unit ability IDs
  "coreAbilities": ["feel-no-pain"],          // core abilities (shown with blue badge)
  "wargear": ["weavefield-crest"],            // static wargear, always equipped
  "wargearChoices": ["panspectral-scanner"],  // optional wargear, user picks up to 2
  "wargearOptions": ["Description text..."],  // display text for wargear rules (no mechanical effect)
  "canBeLeadBy": ["kahl", "einhyr-champion"], // which leader unit IDs can lead this unit

  // LEADER-ONLY fields:
  "canLead": ["einhyr-hearthguard", "hearthkyn-warriors"],  // which unit IDs this can lead
  "leaderBonus": {
    "text": "While this unit contains...",    // displayed in leader bonus box
    "grantsAbilities": ["lethal-hits"],       // ability IDs granted to the whole led unit's weapons
    "grantsFnp": "feel-no-pain-5",            // FNP upgrade applied to bodyguard (optional)
    "appliesTo": "ranged"                     // filter inherited abilities to weapon type (optional)
  }
}
```

### Enhancement Entry (`data/enhancements.json`)

```jsonc
{
  "quake-multigenerator": {
    "name": "Quake Multigenerator",
    "type": "Enhancement (15 pts)",      // shown as modal type label
    "points": 15,
    "detachment": "hearthband",          // only shown when this detachment is active
    "restriction": "Kâhl only",          // human-readable (display only)
    "restrictedTo": ["kahl"],            // unit IDs that can equip this
    "restrictedToWeapon": "weapon-name", // case-insensitive weapon name match (optional)
    "description": "<strong>HTML...</strong>"
  }
}
```

**Rules:**
- Max 3 enhancements active per army at once
- `detachment` prevents display when a different detachment is selected
- `restrictedTo` filters by `instance.unitId` (standalone) or `instance.leaderId` (attached)
- Enhancements are cleared from instances when switching detachment

### Ability Entry (`data/abilities.json`)

```jsonc
{
  "lethal-hits": {
    "name": "Lethal Hits",
    "type": "Weapon Ability",   // or "Core Ability", "Detachment Rule — X", etc.
    "description": "<strong>HTML...</strong>"
  }
}
```

### Army Instance (entries in `armyList[]`)

```javascript
{
  instanceId: "inst-1",           // unique key, e.g. "inst-3"
  unitId: "einhyr-hearthguard",   // key into units{}
  modelCount: 5,                  // current model count (between baseModels and maxModels)
  weapons: {
    "EtaCarn plasma gun": { equipped: true, count: 5 },
    // key = weapon name string, values track UI state
  },
  leaderId: "kahl" | null,        // attached leader unit ID (null = no leader)
  enhancementId: "ironskein" | null,  // selected enhancement ID (null = none)
  selectedWargear: ["panspectral-scanner"]  // array of selected optional wargear IDs
}
```

### Detachment Definition (`DETACHMENTS` constant in index.html)

```javascript
{
  'hearthband': {
    name: 'Hearthband',
    ruleId: 'prioritised-efficiency',         // ID used for modal lookup
    ruleType: 'Detachment Rule — Hearthband', // modal type label
    ruleDescription: '<HTML>...'              // embedded HTML — does NOT depend on abilities.json
  }
}
```

Current detachment IDs: `hearthband`, `needgaard-oathband`, `delve-assault-shift`, `brandfast-oathband`

---

## Rendering Pipeline

### Startup
```
loadData()
  → fetch abilities.json, enhancements.json, army.json (parallel)
  → fetch all unit JSONs (parallel, keyed into units{})
  → restore armyList from URL hash or localStorage
  → renderApp()
      → loadDetachment()
      → updateStratagemVisibility()
      → renderUnits()
      → renderConfigurationTab()
```

### Units Tab
```
renderUnits()
  → buildActiveArmy()     — maps armyList instances to { unit, instance, ... }
  → Sort cards alphabetically
  → For each entry:
      if entry has leaderId → renderLedUnitCard(ledUnit)
      else                  → renderSoloUnitCard(entry)
```

**`renderSoloUnitCard(entry)`** — standalone unit card:
- `unit` = `units[entry.unit]`
- Computes `furyActive`, `closestActive`, `closestBonuses`, `firebaseActive` from `stratagemState`
- Calls `renderWeapons(filteredWeapons, ..., furyActive, closestActive, closestBonuses, firebaseActive)`
- Calls `renderSoloAbilities(unit, extraWargear)`

**`renderLedUnitCard(ledUnit)`** — leader + bodyguard card:
- `leader` = `units[ledUnit.leader]`, `bodyguard` = `units[ledUnit.bodyguard]`
- Computes `furyActive = stratagemState.fury && isFuryTarget(...)` (only Einhyr Hearthguard)
- Computes `firebaseActive = stratagemState.firebase && bodyguard.keywords.includes('INFANTRY')`
- Builds `leaderInheritedAbilities` and `bodyguardInheritedAbilities` from `leaderBonus.grantsAbilities`
- Renders two `<div class="card-section">` blocks (Leader / Bodyguard)
- Visual states: `.leader-dead` (slain), `.bodyguard-wiped` (wiped)

### Configuration Tab
```
renderConfigurationTab()
  → renderDetachmentSelector()   — buttons to pick detachment, rule preview
  → armyList.map(renderArmyInstance)
```

**`renderArmyInstance(instance)`**:
- Model count adjuster
- If `!isLeader && canBeLeadBy.length > 0`: leader selector dropdown; if leader selected → enhancement selector
- If `isLeader && !unit.isEpicHero`: enhancement selector
- If `unit.wargearChoices`: wargear checkboxes (max 2)
- Weapon equipped toggles and count adjusters

### Weapon Rendering
```
renderWeapons(weapons, inheritedAbilities, appliesTo, furyActive, closestActive, closestBonuses, firebaseActive)
  → renderWeaponCategory('ranged', ..., firebaseActive)   — firebaseActive passed for ranged
  → renderWeaponCategory('melee',  ..., false)            — firebase never applies to melee
      → renderWeapon(weapon, type, ..., firebaseActive)
          → build allAbilities[]
          → if furyActive && furyYP && ranged → push Sustained Hits 1 (stratBuff)
          → if firebaseActive && ranged       → push Sustained Hits 1 (stratBuff, firebase-control)
          → if closestActive                  → push Re-roll Wound 1, Re-roll Hit 1 (closestBuff)
          → render stat grid + ability badges
```

**Weapon ability badge classes:**
- `.wab` — standard weapon ability
- `.wab.inherited` — inherited from leader bonus
- `.wab.strat-buff` — added by stratagem (gold)
- `.wab.closest-buff` — added by Targeting Closest (blue)

---

## Stratagem / Buff System

Stratagems are toggled via the buff dropdown menu or FAB buttons. Each toggle:
1. Flips `stratagemState[id]`
2. Syncs visual state (`.active` class on dropdown item + FAB)
3. Calls `updateBuffButton()` (updates badge count)
4. Calls `renderUnits()` (re-renders all unit cards with new effects)

### Stratagem Detail

| ID | Detachment | FAB | Effect |
|---|---|---|---|
| `closest` | Hearthband | `#closest-fab` 🎯 | Re-roll Wound 1 for all; AP+1 for KÂHL/EINHYR HEARTHGUARD/ÛTHAR |
| `fury` | Hearthband | none | +1 Strength ranged; Sustained Hits 1 ranged (only if `furyYP = true`) |
| `firebase` | Brandfast Oathband | `#firebase-fab` 📡 | Sustained Hits 1 on ranged for INFANTRY units |

**FAB visibility** is controlled by `updateStratagemVisibility()`, called every time `selectDetachment()` runs. FABs are hidden/reset when switching away from their owning detachment.

### Fury of the Hearth Flow
1. Toggle on → `showFuryPrompt()` → user pays/skips YP → `furyYP = true/false`
2. Toggle off → `furyYP = false`
3. Sustained Hits 1 only renders if `furyActive && furyYP`

---

## Detachment System

```javascript
const DETACHMENTS = { 'hearthband': {...}, 'needgaard-oathband': {...}, ... }
```

**Adding a new detachment:**
1. Add entry to `DETACHMENTS` with `name`, `ruleId`, `ruleType`, `ruleDescription`
2. Add ability entry in `data/abilities.json` with matching `ruleId`
3. Add enhancements in `data/enhancements.json` with `"detachment": "<id>"`
4. If the detachment has a unique buff button:
   - Add `stratagemState.<key> = false`
   - Add dropdown item with `data-brandfast="true"` (or new `data-<detachment>` attr)
   - Add FAB button HTML + CSS
   - Add visibility logic to `updateStratagemVisibility()`
   - Add reset logic to `selectDetachment()`
   - Add FAB toggle function
   - If it affects weapon display: thread a new `xyzActive` flag through `renderWeapons` → `renderWeaponCategory` → `renderWeapon`

**`selectDetachment(id)`** does:
- Sets `selectedDetachment`
- Saves to `localStorage('votann-detachment')`
- Resets detachment-specific stratagems
- Clears enhancements that don't belong to the new detachment
- Re-renders everything

---

## Enhancement System

### Filter Logic (`renderInstanceEnhancementSelector`)
An enhancement is shown if ALL conditions pass:
1. Not already used by another instance (max 3 total, tracked by `getUsedEnhancements()`)
2. `enh.detachment` matches `selectedDetachment`
3. `enh.restrictedTo` is absent OR includes `characterId`
4. `enh.restrictedToWeapon` is absent OR character has a weapon with that name

`characterId` = `instance.unitId` (standalone leader) or `instance.leaderId` (attached leader)

### Enhancement Card UI
Cards rendered as `.enh-option` divs in `.enh-option-list`:
- `.enh-option.active` — currently selected
- `.enh-option.disabled` — max 3 reached and not active

---

## Leader–Bodyguard Attachment

### Display (Units Tab)
Attachment renders as a single merged card (`renderLedUnitCard`) only when an instance has `leaderId` set AND the leader is listed in `units[bodyguard.canBeLeadBy]`.

The `buildActiveArmy()` function groups instances into `ledUnits` (leader + bodyguard pairs) and `soloUnits`.

### Leader Bonus
`leaderBonus.grantsAbilities` is injected into the bodyguard's weapons as `inheritedAbilities`. Rendered as `.wab.inherited` badges.

Optional: `grantsFnp` overrides bodyguard FNP. `appliesTo: "ranged"|"melee"` filters which weapon types receive the inherited abilities.

### Death / Wipe States

| Action | State Key | CSS Class | Effect |
|---|---|---|---|
| Toggle leader dead | `characterState[leaderId]` | `.leader-dead` | "SLAIN" overlay; inherited bonuses hidden |
| Toggle bodyguard wipe | `bodyguardState[leaderId]` | `.bodyguard-wiped` | "WIPED" overlay; leader loses "while leading" bonuses |

**Adding a unit that can be led:**
- Add `"canBeLeadBy": ["leader-id"]` to the bodyguard unit JSON
- The leader must have `"canLead": ["bodyguard-id"]` in its JSON (this controls the config dropdown options)

---

## Modal Popup System

Clicking any element with `[data-ab="<id>"]` calls `showModal(id)`.

Lookup order:
1. `abilities[id]`
2. `enhancements[id]`
3. `DETACHMENTS` — matches `ruleId` field

Displays: `name` as title, `type` as subtitle, `description` as HTML body.

---

## Points Calculation

**`calculateInstancePoints(instance)`**:
- Base: `unit.points` (for `baseModels` count)
- Scale: `(modelCount - baseModels) * unit.pointsPerModel` (if defined)
- Add: enhancement points (`enhancements[enhancementId].points`)
- Add: leader points (if `leaderId` is set, add the leader unit's `points`)

**`updateConfigPoints()`**: sums all instances and updates the `#config-points` display.

---

## localStorage Keys

| Key | Type | Default |
|---|---|---|
| `votann-theme` | `'light'` \| `'dark'` | `'dark'` |
| `votann-fancy` | `'on'` \| `'off'` | `'off'` |
| `votann-detachment` | detachment ID string | `'hearthband'` |
| `votann-army-list` | JSON `{ armyList, nextInstanceId }` | `[]` |

---

## URL Sharing

`shareArmyUrl()`:
1. Encodes `{ armyList, nextInstanceId, detachment }` → `btoa(encodeURIComponent(JSON.stringify(...)))`
2. Sets `window.location.href = base + '#' + encoded`
3. Copies to clipboard
4. Button shows "Copied!" for 2 seconds

On load, `loadArmyFromUrl()` checks `window.location.hash`, decodes, restores army state, saves to localStorage, clears hash.

---

## CSS Conventions

### Unit Cards
| Class | Meaning |
|---|---|
| `.unit-card` | Standalone unit card |
| `.unit-card.attached` | Leader+bodyguard card (gold border) |
| `.unit-card.leader-dead` | Leader marked slain |
| `.unit-card.collapsed` | Card body hidden (accordion) |

### Ability Badges
| Class | Meaning |
|---|---|
| `.ab` | Unit ability badge |
| `.ab.core` | Core ability (blue) |
| `.ab.wg` | Wargear ability (gold) |
| `.ab.inherited` | Inherited from leader (green dashed) |
| `.wab` | Weapon ability badge |
| `.wab.strat-buff` | Added by stratagem (gold highlight) |
| `.wab.closest-buff` | Added by Targeting Closest (blue highlight) |

### Defense Row
| Class | Meaning |
|---|---|
| `.invuln-row` | Invulnerable save display (gold) |
| `.invuln-row.inherited` | Invuln granted by leader (green dashed) |
| `.invuln-row.fnp-only` | FNP without invuln |

### Configuration
| Class | Meaning |
|---|---|
| `.enh-option` | Enhancement card |
| `.enh-option.active` | Selected enhancement |
| `.enh-option.disabled` | Greyed out (max reached) |
| `.leader-select` | Leader assignment dropdown |
| `.config-label` | Section label in instance card |

### Buff Dropdown
| Class | Meaning |
|---|---|
| `.buff-dropdown-item` | Single stratagem row |
| `.buff-dropdown-item.active` | Stratagem enabled |
| `[data-hearthband]` | Hidden when not Hearthband detachment |
| `[data-brandfast]` | Hidden when not Brandfast Oathband detachment |

### FABs
| ID | Detachment | Color |
|---|---|---|
| `#closest-fab` | Hearthband | Blue |
| `#firebase-fab` | Brandfast Oathband | Green |

---

## Common Patterns for Adding Features

### Add a new unit
1. Create `data/units/<id>.json` following the unit schema
2. Add the ID to `ALL_UNIT_IDS` in index.html
3. Add to `UNIT_GROUPS` in the appropriate category
4. If it can be led: add `"canBeLeadBy": [...]`
5. If it's a leader: add `"isLeader": true`, `"canLead": [...]`, `"leaderBonus": {...}`

### Add a new ability
1. Add entry to `data/abilities.json`
2. Reference by ID in unit's `"abilities"`, `"coreAbilities"`, or weapon's `"abilities"`

### Add a new enhancement
1. Add entry to `data/enhancements.json` with `"detachment": "<id>"`
2. If character-restricted: add `"restrictedTo": ["unit-id"]`

### Add a new detachment
See the "Adding a new detachment" section under Detachment System above.

### Add a new weapon buff (stratagem effect)
1. Add key to `stratagemState`
2. Compute `xyzActive` in `renderSoloUnitCard` and `renderLedUnitCard`
3. Pass `xyzActive` through `renderWeapons` → `renderWeaponCategory` → `renderWeapon`
4. In `renderWeapon`: push to `allAbilities` with `{ id, display, stratBuff: true }`
5. Add UI toggle (dropdown item + optional FAB)
