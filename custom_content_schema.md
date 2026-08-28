# Custom Content JSON Schema Documentation

> This file is published at
> [github.com/saentari/adventura-custom-content](https://github.com/saentari/adventura-custom-content)
> and kept byte-identical in both places. Edit it in the app repo under `docs/`,
> then copy it across — a fork here is how the two copies drifted before.

This document describes the JSON schema format for importing custom D&D 5e content into Adventura.

## Table of Contents

- [Overview](#overview)
- [Root Structure](#root-structure)
- [Source Metadata](#source-metadata)
- [Content Types](#content-types)
  - [Classes](#classes)
  - [Species](#species)
  - [Lineages](#lineages)
  - [Backgrounds](#backgrounds)
  - [Feats](#feats)
  - [Equipment and Magic Items](#equipment-and-magic-items)
  - [Names](#names)
  - [Deities](#deities)
  - [Invocations](#invocations)
  - [Monsters](#monsters)
- [Validation Rules](#validation-rules)
- [Examples](#examples)

---

## Overview

Custom content packs are JSON files that contain additional D&D content from official books (like Xanathar's Guide, Tasha's Cauldron) or homebrew content. The app validates these files on import and stores them in the local database.

**Supported Content Types:**
- Classes and Subclasses
- Species and Lineages
- Backgrounds
- Feats
- Spells
- Equipment
- Magic Items
- Deities
- Eldritch Invocations
- Monsters
- Names (species names and random-name word lists)

---

## Root Structure

```json
{
  "source": { ... },
  "classes": [ ... ],
  "species": [ ... ],
  "lineages": [ ... ],
  "backgrounds": [ ... ],
  "feats": [ ... ],
  "spells": [ ... ],
  "equipment": [ ... ],
  "magic_items": [ ... ],
  "deities": [ ... ],
  "invocations": [ ... ],
  "monsters": [ ... ],
  "names": [ ... ]
}
```

### Required Fields
- `source` (object) - Metadata about the content pack

### Optional Fields
- `classes` (array) - Base classes and subclasses
- `species` (array) - Species definitions
- `lineages` (array) - Lineages / subraces of a species
- `backgrounds` (array) - Background definitions
- `feats` (array) - Feat definitions
- `spells` (array) - Spell definitions
- `equipment` (array) - Mundane gear
- `magic_items` (array) - Anything with a `rarity`
- `deities` (array) - Deities offered at character creation
- `invocations` (array) - Eldritch Invocations
- `monsters` (array) - Stat blocks
- `names` (array) - Species name data and random-name word lists

**Note:** At least one content type must be present.

---

## Source Metadata

The `source` object identifies your content pack.

```json
{
  "source": {
    "id": "xanathars_guide",
    "name": "Xanathar's Guide to Everything",
    "version": "srd_5_1",
    "compatible_versions": ["srd_5_1"],
    "description": "Additional subclasses and options"
  }
}
```

A 2014 book declares `["srd_5_1"]` — one edition per pack. See
[Default to one edition per pack](#default-to-one-edition-per-pack) before
listing both.

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier (lowercase, underscores only) |
| `name` | string | ✅ | Display name of the content pack |
| `version` | string | ✅ | Primary SRD version: `srd_5_1` or `srd_5_2`. Drives terminology (Races/Subraces vs Species/Lineages) and is the fallback compatibility version |
| `compatible_versions` | string[] | ❌ | SRD versions this pack works with, e.g. `["srd_5_1", "srd_5_2"]`. When omitted/empty, compatibility falls back to `version` (exact match) |
| `revision` | integer | ❌ | Monotonic content revision for app-bundled system packs only. Bump it to make the app re-seed updated content. Ignored for user-imported packs (default `0`) |
| `description` | string | ❌ | Brief description of the content |

### Validation Rules
- `id` must match pattern: `^[a-z0-9_]+$`
- `version` must be exactly `srd_5_1` or `srd_5_2`
- each `compatible_versions` entry must be exactly `srd_5_1` or `srd_5_2`

### Version System Explained

The `version` field indicates which D&D rule set your content is formatted for:

- **`srd_5_1`** - For content designed with 2014 D&D rules
  - Use for books published 2014-2023 (Player's Handbook, Xanathar's Guide, Tasha's Cauldron, etc.)
  - The app will interpret mechanics using 2014 rule conventions

- **`srd_5_2`** - For content designed with 2024 D&D rules
  - Use for books published 2024 onwards
  - The app will interpret mechanics using 2024 rule conventions

**Key Point:** `version` tells the app *how to interpret* the content (which terminology and rule conventions to apply), not which rule sets it can be used with. Books published before 2024 (like Xanathar's Guide 2017 or Tasha's Cauldron 2020) should set `version` to `srd_5_1` even though they're playable under both rule sets.

To make a pack selectable under more than one rule set, list every compatible version in `compatible_versions`. Without it, a pack is only offered where the rules version exactly equals `version`.

### Default to one edition per pack

**A pack should declare only the edition it was written for.** Multi-version is an opt-in escape hatch, not the norm.

The two editions are genuinely different content, not a re-skin. Measured across the SRD:

- **Zero** of the 277 monsters present in both 5.1 and 5.2 have identical stat blocks.
- The Aboleth went from 135 HP (18d10) to 150 (20d10 + 40); the Ancient Red Dragon from 546 to 507. Goblins became Fey.
- Where a book and the SRD disagree, they disagree concretely: the 2014 Monster Manual's Hobgoblin Captain has 39 HP, SRD 5.2's has 58.

So a 2014 book, the 5.2 SRD, and a future 2024 book are three distinct things that happen to share names. Declaring a 2014 book as `["srd_5_1", "srd_5_2"]` puts 2014 stat blocks in front of someone playing 2024 rules, alongside the 2024 versions of the same creatures.

Declare `["srd_5_1"]` for a 2014 book, `["srd_5_2"]` for a 2024 one. Reach for both versions only when you mean it — genuinely edition-agnostic content (magic items and equipment are identical across editions; `srd.db` gives neither a version column), or content with no equivalent in the other edition that you would rather have than not.

Whoever imports the pack can always widen `compatible_versions` themselves if they want an older book in a newer game. That is their call to make, and it should be a choice rather than a default.

---

## Content Types

### Classes

Classes can be either **subclasses** (extending an existing SRD class) or **base classes** (entirely new classes like the Artificer).

#### Subclass Example

Subclasses use the `baseClass` field to specify which SRD class they extend:

```json
{
  "id": "xanathars_cavalier",
  "name": "Cavalier (Fighter)",
  "description": "A master of mounted combat",
  "baseClass": "fighter",
  "features": {
    "3rd": [...],
    "7th": [...],
    "10th": [...]
  },
  "tags": ["martial", "subclass", "fighter"]
}
```

#### Base Class Example

Base classes omit `baseClass` and instead provide additional fields for hit dice, saving throws, proficiencies, and optionally spellcasting:

```json
{
  "id": "tcoe_artificer",
  "name": "Artificer",
  "description": "Masters of invention...",
  "hitDie": 8,
  "savingThrows": ["CON", "INT"],
  "primaryAbilities": "Intelligence",
  "proficiencies": {
    "skillOptions": ["Arcana", "History", "Investigation", "Medicine", "Nature", "Perception", "Sleight of Hand"],
    "skillChoices": 2,
    "weaponProficiencies": ["Simple weapons"],
    "toolOptions": ["Alchemist's Supplies", "Smith's Tools", ...],
    "toolChoices": 1,
    "armorTraining": ["Light armor", "Medium armor", "Shields"]
  },
  "spellcasting": {
    "ability": "Intelligence",
    "cantripsKnown": [2, 2, 2, ...],
    "spellSlots": {
      "1": [2, 2, 3, ...],
      "2": [0, 0, 0, ...]
    }
  },
  "startingCurrency": { "gp": 100 },
  "abilityPriority": ["Intelligence", "Constitution", "Dexterity", "Wisdom", "Charisma", "Strength"],
  "features": {
    "1st": [...],
    "2nd": [...],
    "3rd": [...]
  },
  "tags": ["spellcaster", "half-caster", "artificer"]
}
```

#### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier for this class |
| `name` | string | ✅ | Display name |
| `description` | string | ❌ | Class description |
| `baseClass` | string | ❌ | For subclasses: `fighter`, `wizard`, `cleric`, etc. Omit for base classes |
| `proficiencies` | object | ❌ | For subclasses: additional weapon/armor proficiencies granted by the subclass (see below) |
| `features` | object | ❌ | Features organized by level |
| `tags` | array | ❌ | Tags for filtering/searching |

#### Subclass Proficiencies (when `baseClass` is present)

Subclasses can optionally grant additional weapon and armor proficiencies that merge with the base class proficiencies during character creation.

```json
"proficiencies": {
  "weaponProficiencies": ["Martial weapons"],
  "armorTraining": ["Medium armor", "Shields"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `weaponProficiencies` | array | ❌ | Additional weapon proficiencies (e.g. `["Martial weapons"]`, `["Scimitar"]`) |
| `armorTraining` | array | ❌ | Additional armor proficiencies (e.g. `["Medium armor", "Shields"]`, `["Heavy armor"]`) |

**Note:** Only `weaponProficiencies` and `armorTraining` are supported for subclasses. Duplicates with the base class are automatically filtered out.

#### Base Class Fields (when `baseClass` is absent)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `hitDie` | integer | ❌* | Hit die size: `6`, `8`, `10`, or `12`. Defaults to 8 |
| `savingThrows` | array | ❌* | Exactly 2 ability abbreviations: `STR`, `DEX`, `CON`, `INT`, `WIS`, `CHA` |
| `primaryAbilities` | string | ❌ | Primary abilities description (e.g. `"Intelligence"`) |
| `proficiencies` | object | ❌ | Proficiency options (see below) |
| `spellcasting` | object | ❌ | Spellcasting configuration (see below) |
| `startingCurrency` | object | ❌ | Starting gold: `{ "gp": 100 }` |
| `abilityPriority` | array | ❌ | 6 ability names for auto-fill ordering |
| `asiLevels` | array | ❌ | Class levels that grant an Ability Score Improvement. Defaults to `[4, 8, 12, 16, 19]`. Use `[4, 6, 8, 12, 14, 16, 19]` for Fighter-style classes |
| `isSpellbookCaster` | boolean | ❌ | `true` for Wizard-style classes that learn 2 leveled spells per level-up into a spellbook. Defaults to `false` |
| `nonStackingFeatures` | array | ❌ | Feature names that don't stack when multiclassing (e.g. `["Extra Attack", "Unarmored Defense"]`). Defaults to `[]` |
| `extraAttackProgression` | object | ❌ | Maps class levels to number of attacks for improved Extra Attack (e.g. `{"5": 2, "11": 3, "20": 4}` for Fighter-style). Omit for classes with no Extra Attack |
| `pactSlotCountByLevel` | array | ❌ | 20-element array of pact magic slot counts (index 0 = level 1). Only for Pact Magic casters |
| `pactSlotLevelByLevel` | array | ❌ | 20-element array of pact magic slot levels (index 0 = level 1). Only for Pact Magic casters |
| `multiclassProficiencies` | object | ❌ | What a character gains when they multiclass *into* this class: `{armor, weapons, tools, skillChoices, skillOptions}`. Without it, a homebrew class grants nothing on multiclass entry |

\* These fields are optional but recommended. A warning is shown on import if they're missing.

#### Proficiencies Object

```json
"proficiencies": {
  "skillOptions": ["Arcana", "History", "..."],
  "skillChoices": 2,
  "weaponProficiencies": ["Simple weapons"],
  "toolProficiencies": ["Calligrapher's Supplies"],
  "toolOptions": ["Alchemist's Supplies", "..."],
  "toolChoices": 1,
  "armorTraining": ["Light armor", "Medium armor", "Shields"]
}
```

`toolProficiencies` are granted outright; `toolOptions` + `toolChoices` are
picked by the player.

#### Spellcasting Object

```json
"spellcasting": {
  "ability": "Intelligence",
  "casterType": "half",
  "isPreparedCaster": true,
  "initialLevel1Spells": 2,
  "cantripsKnown": [2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 4, 4, 4, 4, 4, 4, 4]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ability` | string | ✅ | Spellcasting ability (`Intelligence`, `Wisdom`, `Charisma`, …) |
| `casterType` | string | ❌ | `full`, `half`, `third`, `pact`, or `none`. **Defaults to `full`** whenever a `spellcasting` block is present |
| `isPreparedCaster` | boolean | ❌ | `true` for Cleric/Druid/Artificer-style classes that prepare from the whole list. Default `false` |
| `cantripsKnown` | array | ❌ | 20 entries, indexed by class level (index 0 = level 1) |
| `spellsKnown` | array | ❌ | 20 entries. Only for **known**-spell casters; leave it out for prepared casters |
| `initialLevel1Spells` | integer | ❌ | Leveled spells chosen at character creation. Default `0` |

**`spellSlots` is not a field.** Slot counts come from `casterType` and the
standard tables — nothing reads an authored `spellSlots` map, so a class that
supplies one instead of a `casterType` silently gets the *full*-caster table.

**A caster needs `isPreparedCaster` or `spellsKnown`, or it never learns a
leveled spell.** With neither, the level-up flow has no rule for granting one:
the class gets its cantrips at level 1 and nothing after that, at every level,
with no error anywhere. If the class prepares spells, say so; if it learns them,
give it a `spellsKnown` table.

#### Features Object

Features are organized by level keys (e.g., `"1st"`, `"3rd"`, `"20th"`).

```json
"features": {
  "3rd": [
    {
      "name": "Unwavering Mark",
      "description": "When you hit a creature...",
      "summary": "Mark a creature you hit; it has disadvantage against your allies.",
      "choices": ["Option A", "Option B"],
      "choiceDescription": "Choose your mark:",
      "choiceDescriptions": { "Option A": "…", "Option B": "…" },
      "maxChoices": 1
    }
  ]
}
```

**Feature Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Feature name |
| `description` | string | ✅ | Feature description |
| `summary` | string | ❌ | One-line mechanical recap. Used on the sheet and in the printable PDF, where the full description is too long |
| `choices` | array | ❌ | Available choices for this feature (single-select) |
| `choiceDescription` | string | ❌ | Prompt shown above the choice list |
| `choiceDescriptions` | object | ❌ | Per-choice blurb, keyed by the choice string |
| `choiceData` | object | ❌ | Per-choice mechanical effects, keyed by the choice string (see below) |
| `maxChoices` | integer | ❌ | Maximum number of choices allowed |
| `multiSelect` | object | ❌ | Catalog-driven, level-scaling multi-select (see below) |

**The first base-class feature carrying `choices` is treated as the subclass
picker.** Its level is the level the subclass is chosen at, and each entry in
`choices` must match the `name` of a class entry whose `baseClass` is this
class's name. Don't give an earlier feature a `choices` array unless you mean
it to be the subclass choice.

##### Per-choice effects (`choiceData`)

A feature choice can carry mechanical effects, applied while that choice is the
selected one:

```json
{
  "name": "Woven Pattern",
  "description": "Choose one pattern.",
  "choices": ["Pattern of the Open Page", "Pattern of the Quick Step"],
  "choiceData": {
    "Pattern of the Open Page": {
      "effects": [
        {
          "type": "advantage",
          "data": { "on": "skill", "skills": ["Arcana", "History"], "condition": "" },
          "description": "Advantage on Arcana and History checks."
        }
      ]
    },
    "Pattern of the Quick Step": {
      "effects": [
        { "type": "speedBonus", "data": { "bonus": 10 } }
      ]
    }
  }
}
```

Three effect types resolve from `choiceData` today — `advantage` on skills
(`{on: "skill", skills: [...], condition: ""}`), `speedBonus`, and `weaponGrant`
(a whole extra attack, as in the Armorer's Lightning Launcher). Anything else is
parsed and then ignored, so leave it as prose.

**Level Key Format:**
- Must match pattern: `^\d+(st|nd|rd|th)$`
- Examples: `"1st"`, `"2nd"`, `"3rd"`, `"20th"`

##### Multi-Select Features (`multiSelect`)

Use `multiSelect` for a feature where the player knows a **growing** number of
options drawn from a catalog — Artificer Infusions, Eldritch Invocations,
Metamagic. This is different from `choices`/`maxChoices` (a fixed pick from a
flat list): the budget scales with class level and the option pool can carry
mechanical effects. A feature is treated as multi-select **only** when this
block is present; do not also set `choices` on the same feature.

```json
{
  "name": "Infuse Item",
  "description": "You've learned to imbue mundane items with magical infusions.",
  "multiSelect": {
    "storageKey": "Artificer Infusions",
    "budgetByLevel": { "2": 2, "6": 4, "10": 6, "14": 8, "18": 10 },
    "swapOnLevelUp": true,
    "catalog": [
      {
        "name": "Enhanced Defense",
        "description": "A suit of armor or a shield gains a +1 bonus to AC.",
        "levelRequirement": 2
      },
      {
        "name": "Enhanced Weapon",
        "description": "A simple or martial weapon gains a +1 bonus to attack and damage rolls.",
        "levelRequirement": 2,
        "prerequisite": "Enhanced Defense"
      }
    ]
  }
}
```

**`multiSelect` Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `storageKey` | string | ❌ | Key the selections are stored under. Defaults to the feature `name`. |
| `budgetByLevel` | object | ❌ | Map of *class* level → number known. Known count = value of the highest level key `≤` the character's level in this class (0 below the first). |
| `swapOnLevelUp` | boolean | ❌ | If true, the player may replace one known option on each class level-up. Default `false`. |
| `catalog` | array | ❌ | The selectable options (see below). |

**Catalog option Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Display name; also the stored selection token. |
| `description` | string | ❌ | Shown in the picker and detail view. |
| `levelRequirement` | integer | ❌ | Minimum class level to select this option. Default `1`. (`level_requirement` also accepted.) |
| `prerequisite` | string | ❌ | Free-form gating token (e.g. another feature's name); evaluated by the caller. |
| `effects` | array | ⚠️ | Effect objects (`type`/`data`/`description`). **Parsed but not yet applied** — see below. |
| `appliesToCategory` | string | ⚠️ | Marks an *item-targeting* option, to be bound to an owned item of this category (`armor`/`weapon`/`shield`/`wondrous`). **Not yet applied** — see below. |

**A catalog option's `effects` do nothing yet.** They are parsed off the JSON and
carried on the option, and no code reads them: selecting an option records the
choice and shows its text. The same goes for `appliesToCategory`, which waits on
the worn-item binding rule. Write the option's mechanics into its
`description` — an option whose only statement of what it does lives in an
`effects` block will read as doing nothing at the table.

---

### Species

Species (formerly races) represent character origins.

```json
{
  "id": "volos_aasimar",
  "name": "Aasimar",
  "description": "Aasimar bear within their souls the light of the heavens",
  "abilityScoreBonuses": {
    "CHA": 2,
    "WIS": 1
  },
  "features": [
    {
      "name": "Darkvision",
      "description": "You can see in dim light within 60 feet..."
    }
  ],
  "languages": ["Common", "Celestial"],
  "speed": 30,
  "size": "Medium",
  "tags": ["celestial", "volos"]
}
```

#### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `description` | string | ❌ | Species description |
| `abilityScoreBonuses` | object | ❌ | Ability score increases. **SRD 5.1 only** — under 2024 rules the background carries these |
| `features` | array | ❌ | Species features (see below) |
| `languages` | array | ❌ | Languages known outright |
| `languageChoiceCount` | integer | ❌ | Extra languages the player picks on top of `languages` |
| `speed` | integer | ❌ | Walking speed in feet |
| `size` | string | ❌ | Size category |
| `creature_type` | string | ❌ | `Humanoid`, `Fey`, `Construct`, … Shown on the sheet. Defaults to `Humanoid` |
| `characteristics` | object | ❌ | Flavour ranges for the description step: `height`, `age`, `weight`, `eyes`, `skin`, `hair` |
| `flexibleChoices` | object | ❌ | Creation-time pickers — a floating ability increase, a size choice, a feat (see below) |
| `image_url` | string | ❌ | Illustration shown on the species card |
| `tags` | array | ❌ | Tags for filtering |

#### Flexible Choices

`flexibleChoices` drives the extra pickers shown during character creation. Use
it for the Tasha's Custom Lineage shape (a floating increase plus a feat) and
for 2024 species that choose their own size.

```json
"flexibleChoices": {
  "abilityScores": { "count": 1, "increase": 2 },
  "size": { "options": ["Small", "Medium"] },
  "feats": { "count": 1 }
}
```

| Block | Shape | Effect |
|-------|-------|--------|
| `abilityScores` | `{count, increase}` **or** `{totalPoints, maxPerAbility}` | A distributor appears. `{count: 1, increase: 2}` is "+2 to one score"; `{totalPoints: 3, maxPerAbility: 2}` is the Monsters of the Multiverse "spend 3 points" style |
| `size` | `{options: ["Small", "Medium"]}` | The player picks their size |
| `feats` | `{count: 1}` | A feat picker appears at creation |
| `variableTrait` | `{options: [{type, count, range}]}` | A pick between differently-shaped traits |

The same block works on a lineage, which is where the Variant Human / Custom
Lineage version of it belongs.

#### Ability Score Bonuses

```json
"abilityScoreBonuses": {
  "STR": 2,
  "DEX": 1,
  "CON": 0,
  "INT": 0,
  "WIS": 1,
  "CHA": 0
}
```

**Valid Ability Keys:** `STR`, `DEX`, `CON`, `INT`, `WIS`, `CHA`

**Values:** Integers (typically 0-2)

#### Features Array

```json
"features": [
  {
    "name": "Crystal Sight",
    "description": "You can see in dim light within 60 feet...",
    "vision": { "type": "darkvision", "distance": 60 }
  },
  {
    "name": "Resonance",
    "description": "You have resistance to thunder damage...",
    "damageResistance": "thunder",
    "saveAdvantage": { "against": "thunder", "abilities": ["CON"] }
  },
  {
    "name": "Inner Light",
    "description": "You know the Light cantrip...",
    "grantedSpells": ["light"],
    "levelGrantedSpells": { "3": ["faerie_fire"], "5": ["misty_step"] }
  },
  {
    "name": "Cavern Lore",
    "description": "Choose one skill; you gain proficiency in it.",
    "choices": ["Arcana", "History", "Perception", "Survival"],
    "choicesAreSkillProficiencies": true,
    "maxChoices": 1
  }
]
```

**Feature Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | ✅ | Feature name |
| `description` | string | ✅ | Feature description |
| `choices` | array | ❌ | Available choices (for subraces, etc.) |
| `maxChoices` | integer | ❌ | Max choices (must be ≥ 1) |
| `choicesAreSkillProficiencies` | boolean | ❌ | The player's pick *is* the skill proficiency granted |
| `choicesAreToolProficiencies` | boolean | ❌ | The same, for tools |
| `vision` | object | ❌ | `{type, distance}` — `darkvision`, `blindsight`, `tremorsense`, `truesight` |
| `damageResistance` | string | ❌ | One damage type. Use `damageResistances` for a list |
| `saveAdvantage` | object | ❌ | `{against, abilities}` — e.g. `{"against": "poisoned", "abilities": ["CON"]}` |
| `skillProficiencies` | array | ❌ | Skills granted outright |
| `toolProficiencies` | array | ❌ | Tools granted outright |
| `weaponProficiencies` | array | ❌ | Weapons granted outright |
| `armorProficiencies` | array | ❌ | Armor granted outright |
| `speedBonus` | object | ❌ | `{flat: 5}` |
| `grantedSpells` | array | ❌ | SRD spell indices granted at level 1 |
| `levelGrantedSpells` | object | ❌ | Character level → spell indices, e.g. `{"3": ["faerie_fire"]}` |
| `minLevel` | integer | ❌ | Character level the feature (and its grants) unlocks at |
| `choiceData` | object | ❌ | Per-choice `damageResistance` / `grantedSpells` / `levelGrantedSpells`, keyed by the choice string |

**A spell grant is an index, not a name** — `faerie_fire`, not "Faerie Fire". An
index that isn't in the SRD for the pack's rules version resolves to nothing,
silently, and the character simply never receives the spell.

---

### Lineages

A lineage (subrace) extends a species. It is matched to its parent by
`baseSpecies`, and the player chooses one after picking the species.

```json
{
  "id": "hb_deepvein_glimmerfolk",
  "name": "Deepvein Glimmerfolk",
  "baseSpecies": "hb_glimmerfolk",
  "description": "Deepvein Glimmerfolk have dwelled so far beneath the surface...",
  "abilityScoreBonuses": { "CON": 1 },
  "features": [
    {
      "name": "Superior Darkvision",
      "description": "Your darkvision extends to 120 feet.",
      "vision": { "type": "darkvision", "distance": 120 }
    }
  ],
  "tags": ["underground", "homebrew"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `baseSpecies` | string | ✅ | The `id` of the species this lineage belongs to |
| `description` | string | ❌ | Lineage description |
| `abilityScoreBonuses` | object | ❌ | SRD 5.1 only, as for species |
| `features` | array | ❌ | Same feature shape as species features |
| `flexibleChoices` | object | ❌ | Same shape as the species block above. This is where Variant Human / Custom Lineage pickers belong |
| `replacesBaseSpecies` | boolean | ❌ | `true` for a Custom Lineage that replaces its parent's traits rather than adding to them |
| `tags` | array | ❌ | Tags for filtering |

---

### Backgrounds

Backgrounds represent a character's origin story. Use the same field names and structure as SRD 5.1 / 5.2 for consistency.

```json
{
  "id": "xanathars_city_watch",
  "name": "City Watch",
  "description": "You have served the community...",
  "skillProficiencies": ["Athletics", "Insight"],
  "toolProficiencies": [],
  "languageChoiceCount": 2,
  "equipment": {
    "optionA": ["A uniform in the style of your unit", "A horn with which to summon help"],
    "optionB": ["50 GP"]
  },
  "feature": "Watcher's Eye",
  "featureDescription": "Your experience in enforcing the law...",
  "feat": "Alert"
}
```

- **SRD 5.1 style:** Use `feature` (and optionally `featureDescription`). No `feat`. `languageChoiceCount` for bonus language choices.
- **SRD 5.2 style:** Use `feat` for the background feat name, plus `feature` and `featureDescription`. `abilityScores` array (e.g. `["INT", "WIS", "CHA"]`) for 5.2 ability score bonuses.

#### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `description` | string | ❌ | Background description |
| `skillProficiencies` | array | ❌ | Skill proficiency names (max 4) |
| `toolProficiencies` | array | ❌ | Tool proficiency names |
| `languageChoiceCount` | integer | ❌ | Number of bonus language choices (same as SRD) |
| `equipment` | object or array | ❌ | SRD shape: `{ "optionA": [...], "optionB": [...] }`; or flat array of strings |
| `feature` | string | ❌ | Background feature name (short; same as SRD) |
| `featureDescription` | string | ❌ | Long description of the feature (same as SRD) |
| `feat` | string | ❌ | For 5.2-style: feat name (e.g. "Alert", "Magic Initiate (Wizard)") |
| `abilityScores` | array | ❌ | For 5.2-style: exactly three abilities, e.g. `["STR", "DEX", "CON"]` |
| `spells` | array | ❌ | SRD spell indices the background adds to a spellcaster's available list |

**`abilityScores` drives the +2/+1/+0 distributor** shown when a 5.2 background
is selected, and the result is stored on the character. List three abilities;
fewer means fewer slots.

**`feat` is displayed, not granted.** The background card and the character
sheet show the feat's name, but no feat is added to the character automatically —
that is true of SRD 5.2 backgrounds too. Add it yourself at character creation
if you want its effects.

---

### Feats

Feats provide mechanical benefits a character gains either from a background (origin), at ASI levels (general), via a Fighting Style, or as an Epic Boon.

```json
{
  "id": "hb_keen_eye",
  "name": "Keen Eye",
  "description": "Your eye for detail rivals that of a master cartographer.",
  "summary": "Advantage on Wisdom (Perception) checks made to spot hidden creatures or objects. Half-cover offers no concealment from your ranged attacks.",
  "prerequisites": "",
  "category": "general",
  "minimumLevel": 4,
  "tags": ["perception", "ranged"],
  "effects": [
    {
      "type": "passiveAbility",
      "data": {"ability": "keen_eye_perception"},
      "description": "Advantage on Perception checks vs. hidden creatures"
    }
  ],
  "choices": []
}
```

#### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `description` | string | ✅ | Flavor / lore prose. Shown on the feat detail sheet. |
| `summary` | string | ❌ | Short, mechanics-focused recap of the bonuses and effects. Used in feat lists, the character sheet feat card, and the printable PDF. Keep under ~280 characters. Falls back to `description` when omitted. |
| `prerequisites` | string | ❌ | Plain-text prerequisite (e.g. `"Strength 13 or higher"`) |
| `category` | string | ❌ | One of `origin`, `general`, `fightingStyle`, `epicBoon`. Shown as a label on the feat detail sheet |
| `minimumLevel` | integer | ❌ | Earliest character level the feat can be selected (default `1`). Parsed and displayed, **not yet enforced** by the picker |
| `tags` | array | ❌ | Free-form tag strings used for search/filtering |
| `effects` | array | ❌ | Mechanical effects (`type`, `data`, `description`) — see below |
| `choices` | array | ❌ | Player decisions tied to the feat: `{id, label, type, options, maxSelections}` where `type` is `ability`, `skill`, `tool`, `weapon`, `language`, `spell`, or `spellcastingClass` |

#### Feat effects worth knowing

Most effect types take the shape documented under
[Equipment and Magic Items](#worneffects). Two are specific to feats:

```json
{
  "type": "abilityScoreChoice",
  "data": { "abilities": ["DEX", "WIS"], "bonus": 1, "max": 20 },
  "description": "+1 Dexterity or Wisdom (max 20)"
}
```

`abilityScoreChoice` is shorthand: it expands into an `abilityScoreBonus` effect
plus a matching player choice, so you don't author the choice block yourself.

```json
{ "type": "damageBonus", "data": { "bonus": 1, "type": "ranged_attack" } }
```

**`data.type` decides where a `damageBonus` lands**, not the effect's name. The
four keys are `melee_attack`, `ranged_attack`, `melee_damage` and
`ranged_damage` — the first two modify attack rolls despite the effect being
called `damageBonus`. (The `attackBonus` effect type is *not* read for this; use
`damageBonus` with the right `data.type`.)

---

### Equipment and Magic Items

`equipment` holds mundane gear — weapons, armor, tools, adventuring gear. `magic_items` holds anything magical. The two share one schema; **an item is treated as magic if, and only if, it has a `rarity`**.

#### Mundane equipment

```json
{
  "id": "hb_crystal_tipped_spear",
  "name": "Crystal-Tipped Spear",
  "index": "hb_crystal_tipped_spear",
  "equipment_category": { "index": "weapon", "name": "Weapon" },
  "weapon_category": "Martial",
  "weapon_range": "Melee",
  "damage": {
    "damage_dice": "1d8",
    "damage_type": { "index": "piercing", "name": "Piercing" }
  },
  "properties": [{ "index": "thrown", "name": "Thrown" }],
  "range": { "normal": 5 },
  "throw_range": { "normal": 20, "long": 60 },
  "cost": { "quantity": 25, "unit": "gp" },
  "weight": 4,
  "desc": "A spear tipped with a shard of resonant crystal."
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` / `index` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `equipment_category` | object | ✅ | `{index, name}`. The `name` drives typing: `Weapon`, `Armor`, `Tools`, anything else becomes adventuring gear |
| `desc` | string \| array | ❌ | Rules text. An array is joined with newlines |
| `cost` | object | ❌ | `{quantity, unit}` where unit is `cp`, `sp`, `ep`, `gp`, or `pp` |
| `weight` | number | ❌ | Pounds |

**Weapons** additionally use `weapon_category` (`Simple` / `Martial`), `weapon_range` (`Melee` / `Ranged`), `damage` (`{damage_dice, damage_type: {index}}`), `two_handed_damage` (versatile), `range` / `throw_range` (`{normal, long}`), and `properties` (array of `{index}` — `finesse`, `heavy`, `light`, `loading`, `reach`, `thrown`, `two-handed`, `versatile`, `ammunition`, `special`).

**Armor** additionally uses `armor_category` (`Light` / `Medium` / `Heavy` / `Shield`), `armor_class` (`{base, dex_bonus, max_bonus}`), `str_minimum`, and `stealth_disadvantage`.

#### Magic items

```json
{
  "id": "hb_stoneheart_shield",
  "name": "Stoneheart Shield",
  "index": "hb_stoneheart_shield",
  "equipment_category": { "index": "armor", "name": "Armor" },
  "rarity": { "name": "Rare" },
  "desc": [
    "Armor (shield), rare (requires attunement)",
    "While wielding this shield, you gain a +1 bonus to AC in addition to the shield's normal bonus."
  ],
  "requiresAttunement": true,
  "wornEffects": [
    {
      "type": "acBonus",
      "data": { "flat": 1 },
      "description": "+1 bonus to AC, in addition to the shield's normal bonus."
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `rarity` | object | ✅ | `{name}` — `Common`, `Uncommon`, `Rare`, `Very Rare`, `Legendary`, `Artifact`. Its presence is what makes the item magical |
| `desc` | array | ✅ | Rules text. **The first line is parsed** — see below |
| `requiresAttunement` | boolean | ❌ | Read from `desc[0]` when omitted |
| `wornEffects` | array | ❌ | Passive effects while worn or held |
| `checkEffects` | array | ❌ | What invoking the item does for an ability/skill check in an adventure |
| `useEffects` | array | ❌ | What using/drinking a consumable does in an adventure |

Magic items also require `id` and `name`, and — for magic armor and weapons — an `equipment_category` of `Armor` or `Weapon`. Unlike mundane equipment, `desc` **must** be an array.

#### The first `desc` line is load-bearing

Magic items in D&D state their base item and their attunement requirement only in prose, and Adventura parses that line rather than making you restate it. Use the standard SRD phrasing:

```
Armor (scale mail), very rare (requires attunement)
Weapon (any sword that deals slashing damage), very rare (requires attunement)
Wondrous item, uncommon
```

* **Magic armor and weapons must name a base item in brackets.** The item is then rebuilt on top of that base and inherits its AC or damage dice, weight, and properties — so **don't author `damage` or `armor_class` on a magic weapon or armor**. A Frostbrand Glaive is a glaive; the app gives it 1d10 slashing, heavy, reach, two-handed, for free.
* A **fixed base** (`Weapon (glaive)`) keeps the id and name you gave it.
* An **open base** (`Weapon (any sword)`, `Armor (medium or heavy)`, `Armor (any)`) expands into one item per eligible base, named `Your Item (Longsword)` with the id `your_item__longsword`.
* An item with no bracket — `Wondrous item`, `Ring`, `Potion`, `Rod` — needs no base and is left alone. Wearable ones are slotted by name (`ring`, `cloak`, `boots`, `amulet`, `gloves`, `belt`, `goggles`, …).
* Only the **first** line is scanned for `requires attunement`, so a property buried later in the text (a Hammer of Thunderbolts' *Giant's Bane*) won't gate the whole item.

Attunement is capped at 3 items, and equipping an item that needs it attunes it automatically when a slot is free.

#### `wornEffects`

Shape is `{type, data, description}`. **Only five types resolve** — anything else is accepted and then ignored, so don't reach for them:

| `type` | `data` | Effect |
|--------|--------|--------|
| `acBonus` | `{"flat": 1}` | **Adds** to Armor Class |
| `acFormula` | `{"base": 15, "abilities": ["DEX"]}` | **Sets** base Armor Class to `base` plus the listed ability modifiers. Use this when the item says "your base AC is X" (Robe of the Archmagi), not "you gain a +X bonus". The highest applicable formula wins across items and class features, so a robe can never make a monk worse |
| `savingThrowBonus` | `{"flat": 1}` | Adds to all saving throws |
| `resistance` | `{"damageType": "fire"}` | Resistance to that damage type |
| `senseGrant` | `{"sense": "darkvision", "range": 60, "stack": "increase"}` | Grants or extends a sense (`stack` is `set` or `increase`) |

`acBonus` and `acFormula` also take an optional **`condition`**, since plenty of items only pay out unarmoured:

| `condition` | Applies when |
|-------------|--------------|
| `always` (default) | Unconditionally |
| `noArmor` | No body armor worn |
| `noShield` | No shield held |
| `noArmorNoShield` | Neither — e.g. Bracers of Defense, which give a fighter in plate nothing |
| `wearingArmor` | Body armor is worn |
| `wieldingShield` | A shield is held |

```json
{
  "id": "hb_duelists_bracers",
  "name": "Duelist's Bracers",
  "equipment_category": { "index": "wondrous_item", "name": "Wondrous Item" },
  "rarity": { "name": "Rare" },
  "desc": [
    "Wondrous item, rare (requires attunement)",
    "While wearing these bracers and using no armor or shield, you gain a +2 bonus to AC."
  ],
  "wornEffects": [
    {
      "type": "acBonus",
      "data": { "flat": 2, "condition": "noArmorNoShield" },
      "description": "+2 bonus to AC while wearing no armor and using no shield."
    }
  ]
}
```

An `acFormula` is ignored entirely while body armor is worn — armor sets the base AC, and the formula is what would have replaced it.

Effects apply only while the item is worn or held, and only once it is attuned if it requires attunement.

#### `checkEffects` and `useEffects`

`checkEffects` declare how *invoking* an item helps an ability or skill check during a solo adventure — a crowbar granting advantage on Strength checks. Fields: `skill`, `ability`, `advantage`, `bonus`. An effect applies when its `ability` matches the check's ability **or** its `skill` matches the check's skill.

`useEffects` declare what happens when a consumable is used. `type` is one of `heal`, `temp_hp` (both use `amount`, a dice expression like `"2d4+2"`), `cure_condition` (uses `conditions`), `check_buff` (a one-shot edge on the next matching check), or `grant_status` (a durational state). The check-shaped types also take `skill`, `ability`, `advantage`, `bonus`, plus `uses` or `durationTurns`.

```json
{
  "id": "hb_potion_of_runic_sight",
  "name": "Potion of Runic Sight",
  "equipment_category": { "index": "potion", "name": "Potion" },
  "rarity": { "name": "Uncommon" },
  "desc": ["Potion, uncommon", "You gain advantage on Arcana checks to decipher magical writing for 1 hour."],
  "useEffects": [
    {
      "type": "grant_status",
      "status": "runic sight",
      "skill": "arcana",
      "advantage": true,
      "durationTurns": 10
    }
  ]
}
```

The app — never the AI — rolls and applies these.

#### What to leave as prose

Model only what the table above supports. A great many real magic items can't be expressed, and that's fine: the item still equips, still uses its base stats, and still shows its full rules text.

The `condition` field covers exactly one axis: whether armor or a shield is worn. Anything narrower has no rail. So an Arrow-Catching Shield's "+2 to AC *against ranged attacks*" stays prose — it is worth nothing against a melee attacker, and there is no per-attack condition to express that.

Also deliberately **not** modelled: extra damage dice on a hit (a Flame Tongue's 2d6 fire), charges, curses, ability-score overrides (an Amulet of Health setting CON to 19), auras that benefit nearby allies (a Rod of Alertness), and player-choice bonuses (a Defender's shiftable +3). Faking any of these as a flat number would quietly inflate the character sheet, which is worse than leaving them for a human to read.

**Not yet authorable:** bonuses to *spell* attack rolls and save DCs from a held implement (Wand of the War Mage, Rod of the Pact Keeper) do work, but only for items listed code-side in `MagicItemMechanics` — there is no `wornEffects` type for them, because the spell-stat helpers are given a character and never an equipment state. A homebrew wand can be held and attuned, but can't yet raise your spell attack.

**Reference:** both example packs demonstrate every `wornEffect` type, every
`useEffects` type, `checkEffects`, the `condition` vocabulary, and an open-base
weapon. They are published at
[github.com/saentari/adventura-custom-content](https://github.com/saentari/adventura-custom-content).

---

### Names

The `names` array holds two kinds of entry, told apart by their fields.

#### Species names

```json
{
  "names": [
    {
      "species": "hb_glimmerfolk",
      "male": ["Crystan", "Faceth", "Gleamor"],
      "female": ["Aurela", "Clarisa", "Gemella"],
      "neutral": ["Glint", "Lumen", "Prism"],
      "familyNames": ["Brightcore", "Crystalvein", "Deepgleam"]
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `species` | string | ✅ | Species identifier (`human`, `elf`, … or a custom species ID) |
| `male` | array | ❌ | Male first names |
| `female` | array | ❌ | Female first names |
| `neutral` | array | ❌ | Gender-neutral first names |
| `familyNames` | array | ❌ | Family/clan names appended to first names |

**Generation logic:**
1. Pick a random first name from the gender-appropriate list
2. If that list is empty, fall back to `neutral`, then any available list
3. If `familyNames` is non-empty, append a random family name
4. Imported names are **additively merged** with built-in names

Species IDs must match the app's identifiers (`human`, `elf`, `dwarf`,
`halfling`, `gnome`, `halfElf`, `halfOrc`, `tiefling`, `dragonborn`, `goliath`,
`orc`) or the `id` of a species in this pack. All entries are complete names,
not prefixes or suffixes.

#### Random name word lists

Word lists for generated adventure and party names. Identified by `category`.

```json
{
  "names": [
    {
      "category": "adventure",
      "mode": "merge",
      "adjectives": ["Crystalline", "Shimmering", "Fractured"],
      "nouns": ["Geode", "Cavern", "Lattice"]
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `category` | string | ✅ | `adventure` or `party` |
| `mode` | string | ❌ | `merge` (default) adds to the built-in pool; `replace` overrides it |
| `adjectives` | array | ✅ | Words for the adjective slot |
| `nouns` | array | ✅ | Words for the noun slot |

Names are built as **"The [Adjective] [Noun]"**. A replace-mode pack that ends
up with empty lists falls back to the built-in pool. A pack containing *only*
`names` is valid.

---

### Deities

Deities offered when a character picks a faith. They carry no mechanics.

```json
{
  "id": "hb_vexith_prismbearer",
  "name": "Vexith the Prismbearer",
  "alignment": "NG",
  "suggestedDomains": ["Light", "Knowledge"],
  "symbol": "A faceted crystal radiating seven colours of light",
  "pantheon": "Glimmerfolk Crystalline Faith",
  "description": "Goddess of crystalline light and refracted truth.",
  "tags": ["homebrew", "crystal"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `alignment` | string | ❌ | Two-letter alignment (`NG`, `LN`, `CE`, …) |
| `suggestedDomains` | array | ❌ | Domain names shown beside the deity |
| `symbol` | string | ❌ | Holy symbol |
| `pantheon` | string | ❌ | Pantheon the deity belongs to |
| `description` | string | ❌ | Flavour text |
| `tags` | array | ❌ | Tags for filtering |

---

### Invocations

Eldritch Invocations. Custom ones are merged with the SRD list when a warlock
picks invocations.

```json
{
  "id": "hb_shard_ward",
  "name": "Shard Ward",
  "index": "hb_shard_ward",
  "description": "When a creature you can see hits you with an attack...",
  "level_requirement": 7,
  "prerequisite": "Pact of the Crystal"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` / `index` | string | ✅ | Unique identifier |
| `name` | string | ✅ | Display name |
| `description` | string | ✅ | Full rules text |
| `level_requirement` | integer | ❌ | Minimum warlock level (default `1`). **This is the only gate applied to a custom invocation** |
| `prerequisite` | string | ❌ | A pact boon (`"Pact of the Blade"`) or a spell index (`"eldritch_blast"`). Shown on the detail sheet; **not enforced for custom invocations**, so treat it as guidance to the player |

---

### Monsters

Stat blocks, in the same shape as the SRD's. They appear in the library.

```json
{
  "id": "hb_crystalwyrm",
  "name": "Crystalwyrm",
  "index": "hb_crystalwyrm",
  "size": "Large",
  "type": "dragon",
  "alignment": "neutral",
  "armor_class": [{ "type": "natural", "value": 16 }],
  "hit_points": 136,
  "hit_dice": "16d10",
  "hit_points_roll": "16d10+48",
  "speed": { "walk": "30 ft.", "fly": "60 ft." },
  "strength": 19, "dexterity": 12, "constitution": 17,
  "intelligence": 14, "wisdom": 13, "charisma": 16,
  "damage_resistances": ["piercing", "slashing"],
  "condition_immunities": [{ "index": "blinded", "name": "Blinded" }],
  "senses": { "darkvision": "120 ft.", "passive_perception": 15 },
  "languages": "Draconic, Terran",
  "challenge_rating": 8,
  "proficiency_bonus": 3,
  "xp": 3900,
  "special_abilities": [{ "name": "Crystalline Body", "desc": "..." }],
  "actions": [{ "name": "Bite", "desc": "..." }],
  "reactions": [{ "name": "Refracting Scales", "desc": "..." }],
  "legendary_actions": []
}
```

Required: `id`/`index`, `name`. Everything else is optional and rendered when
present — the six ability scores, `proficiencies` (saves and skills as
`{value, proficiency: {index, name}}`), `damage_vulnerabilities`,
`damage_resistances`, `damage_immunities`, `condition_immunities`, `senses`,
`languages`, `challenge_rating`, `proficiency_bonus`, `xp`, `subtype`, and the
four action blocks (`special_abilities`, `actions`, `reactions`,
`legendary_actions`).

---

## Validation Rules

The app validates imported content against these rules:

### Global Rules
- JSON must be valid and parseable
- Must contain a `source` object
- Must contain at least one content type
- All required fields must be present and non-empty

### Field-Specific Rules

**IDs:**
- Must be unique within the content type
- Only lowercase letters, numbers, and underscores
- Pattern: `^[a-z0-9_]+$`

**Ability Scores:**
- Keys must be: `STR`, `DEX`, `CON`, `INT`, `WIS`, `CHA`
- Values must be integers

**Level Keys (for class features):**
- Must match: `^\d+(st|nd|rd|th)$`
- Examples: `1st`, `2nd`, `3rd`, `4th`, `11th`, `20th`

**Skill Proficiencies (backgrounds):**
- Maximum 4 skills per background

**Max Choices:**
- Must be a positive integer (≥ 1)

---

## Examples

### Minimal Valid Content Pack

```json
{
  "source": {
    "id": "my_homebrew",
    "name": "My Homebrew Content",
    "version": "srd_5_2"
  },
  "species": [
    {
      "id": "my_custom_species",
      "name": "Custom Species"
    }
  ]
}
```

### Complete Example

Two complete packs are published at
[github.com/saentari/adventura-custom-content](https://github.com/saentari/adventura-custom-content).
They carry the same fictional content in both editions, so diffing them shows
exactly what changes between rule sets:

- `homebrew_example_5_1.json` — SRD 5.1 (2014 rules): species grant ability score bonuses; the background grants a feature and bonus languages
- `homebrew_example_5_2.json` — SRD 5.2 (2024 rules): the background grants ability score bonuses and a feat; species provide traits and a size choice

Between them they demonstrate every field in this document that the app acts
on: a base class with its own subclasses, a multi-select feature, per-choice
effects, flexible species choices, spell grants, every magic-item effect type,
and every consumable use-effect.

They are documentation and test fixtures, not app assets — they are **not**
bundled into the build, and `test/domain/srd/homebrew_example_pack_test.dart`
imports them on every run to prove that everything they demonstrate still
resolves.

---

## Error Handling

When validation fails, the app displays specific error messages:

**Example Errors:**
```
Missing required field: name
Field "abilityScoreBonuses" must be an object
Invalid ability score: FOO (must be one of STR, DEX, CON, INT, WIS, CHA)
Background cannot have more than 4 skill proficiencies
Feature at index 2 missing required field: description
Invalid level key: 3 (must be like "3rd")
```

---

## Tips

1. **Start Small:** Create a simple pack with 1-2 items first
2. **Validate Often:** Import after adding each section to catch errors early
3. **Use Clear IDs:** Make IDs descriptive (e.g., `xanathars_cavalier` not `class1`)
4. **Match SRD Version:** Ensure your `version` matches the rules you're using
5. **Test in App:** Always test imported content by creating a character — a field the app doesn't read fails silently, so the only proof a mechanic works is seeing it on the sheet

---

## Future Enhancements

Fields the schema accepts and the app does not yet act on. They are documented
so an author knows to write the mechanic into the prose as well:

- **`multiSelect` catalog `effects` and `appliesToCategory`** — parsed, never applied
- **`minimumLevel` on a feat** — displayed, not enforced by the picker
- **`prerequisite` on a custom invocation** — displayed, not enforced
- Conditional and player-choice magic item effects (see [Equipment and Magic Items](#equipment-and-magic-items))
- Attack and damage bonuses from non-weapon sources (a Wand of the War Mage's bonus to spell attacks)
- Dependencies between content packs

---

## Support

If you encounter validation errors or need help:
1. Check the error message for specific field issues
2. Compare your JSON to the example file
3. Verify all required fields are present
4. Report issues at: [adventura-custom-content issues](https://github.com/saentari/adventura-custom-content/issues)
