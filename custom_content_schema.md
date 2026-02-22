# Custom Content JSON Schema Documentation

This document describes the JSON schema format for importing custom D&D 5e content into Adventura.

## Table of Contents

- [Overview](#overview)
- [Root Structure](#root-structure)
- [Source Metadata](#source-metadata)
- [Content Types](#content-types)
  - [Classes](#classes)
  - [Species](#species)
  - [Backgrounds](#backgrounds)
- [Validation Rules](#validation-rules)
- [Examples](#examples)

---

## Overview

Custom content packs are JSON files that contain additional D&D content from official books (like Xanathar's Guide, Tasha's Cauldron) or homebrew content. The app validates these files on import and stores them in the local database.

**Supported Content Types:**
- Classes and Subclasses
- Species (Races)
- Backgrounds
- Feats
- Equipment
- Magic Items
- Spells
- Names (species name generation data)

---

## Root Structure

```json
{
  "source": { ... },
  "classes": [ ... ],
  "species": [ ... ],
  "backgrounds": [ ... ],
  "feats": [ ... ],
  "equipment": [ ... ],
  "magic_items": [ ... ],
  "spells": [ ... ],
  "names": [ ... ]
}
```

### Required Fields
- `source` (object) - Metadata about the content pack

### Optional Fields
- `classes` (array) - List of class/subclass definitions
- `species` (array) - List of species definitions
- `backgrounds` (array) - List of background definitions
- `feats` (array) - List of feat definitions
- `equipment` (array) - List of equipment item definitions
- `magic_items` (array) - List of magic item definitions
- `spells` (array) - List of spell definitions
- `names` (array) - List of species name generation data

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
    "description": "Additional subclasses and options"
  }
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | ✅ | Unique identifier (lowercase, underscores only) |
| `name` | string | ✅ | Display name of the content pack |
| `version` | string | ✅ | SRD version compatibility: `srd_5_1` or `srd_5_2` |
| `description` | string | ❌ | Brief description of the content |

### Validation Rules
- `id` must match pattern: `^[a-z0-9_]+$`
- `version` must be exactly `srd_5_1` or `srd_5_2`

### Version System Explained

The `version` field indicates which D&D rule set your content is formatted for:

- **`srd_5_1`** - For content designed with 2014 D&D rules
  - Use for books published 2014-2023 (Player's Handbook, Xanathar's Guide, Tasha's Cauldron, etc.)
  - The app will interpret mechanics using 2014 rule conventions

- **`srd_5_2`** - For content designed with 2024 D&D rules
  - Use for books published 2024 onwards
  - The app will interpret mechanics using 2024 rule conventions

**Key Point:** Books published before 2024 (like Xanathar's Guide 2017 or Tasha's Cauldron 2020) should use `srd_5_1` even though they're compatible with both rule sets. The version tells the app *how to interpret* the content, not which books are compatible.

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

\* These fields are optional but recommended. A warning is shown on import if they're missing.

#### Proficiencies Object

```json
"proficiencies": {
  "skillOptions": ["Arcana", "History", ...],
  "skillChoices": 2,
  "weaponProficiencies": ["Simple weapons"],
  "toolOptions": ["Alchemist's Supplies", ...],
  "toolChoices": 1,
  "armorTraining": ["Light armor", "Medium armor", "Shields"]
}
```

#### Spellcasting Object

Arrays are indexed by class level (index 0 = level 1, index 19 = level 20):

```json
"spellcasting": {
  "ability": "Intelligence",
  "cantripsKnown": [2, 2, 2, 2, 2, 2, 2, 2, 2, 3, 3, 3, 3, 4, 4, 4, 4, 4, 4, 4],
  "spellSlots": {
    "1": [2, 2, 3, 3, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4],
    "2": [0, 0, 0, 0, 2, 2, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3]
  }
}
```

#### Features Object

Features are organized by level keys (e.g., `"1st"`, `"3rd"`, `"20th"`).

```json
"features": {
  "3rd": [
    {
      "name": "Unwavering Mark",
      "description": "When you hit a creature...",
      "choices": ["Option A", "Option B"],
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
| `choices` | array | ❌ | Available choices for this feature |
| `maxChoices` | integer | ❌ | Maximum number of choices allowed |

**Level Key Format:**
- Must match pattern: `^\d+(st|nd|rd|th)$`
- Examples: `"1st"`, `"2nd"`, `"3rd"`, `"20th"`

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
| `abilityScoreBonuses` | object | ❌ | Ability score increases |
| `features` | array | ❌ | Species features |
| `languages` | array | ❌ | Known languages |
| `speed` | integer | ❌ | Walking speed in feet |
| `size` | string | ❌ | Size category |
| `tags` | array | ❌ | Tags for filtering |

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
    "name": "Darkvision",
    "description": "You can see in dim light...",
    "choices": ["Protector", "Scourge", "Fallen"],
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
| `abilityScores` | array | ❌ | For 5.2-style: e.g. `["STR", "DEX", "CON"]` |

---

### Names

Names provide species-specific name generation data for character creation. When a content pack with names is active, imported names are merged with the built-in name pool for more variety.

```json
{
  "names": [
    {
      "species": "human",
      "male": ["Aldric", "Beren", "Cedric"],
      "female": ["Alara", "Brenna", "Celeste"],
      "neutral": ["Ash", "Brook", "Corin"],
      "familyNames": ["Ashford", "Blackwood", "Crowley"]
    }
  ]
}
```

#### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `species` | string | ✅ | Species identifier (e.g. `human`, `elf`, `dwarf`, or a custom species ID) |
| `male` | array | ❌ | Male first names |
| `female` | array | ❌ | Female first names |
| `neutral` | array | ❌ | Gender-neutral first names |
| `familyNames` | array | ❌ | Family/clan names appended to first names |

#### Generation Logic

1. Pick a random first name from the gender-appropriate list
2. If that list is empty, fall back to `neutral`, then any available list
3. If `familyNames` is non-empty, append a random family name
4. Imported names are **additively merged** with built-in names (more variety, never replaces)

#### Notes

- Species IDs must match the app's species identifiers (e.g. `human`, `elf`, `dwarf`, `halfling`, `gnome`, `halfElf`, `halfOrc`, `tiefling`, `dragonborn`, `goliath`, `orc`)
- For custom species, use the same ID used in the species definition (e.g. `hb_glimmerfolk`)
- All name arrays contain plain strings (complete names, not prefixes/suffixes)
- A pack with *only* `names` (no species, classes, etc.) is valid

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

See the example files in `assets/data/examples/`:
- `homebrew_example_5_1.json` - SRD 5.1 (2014 rules): species have ability score bonuses, backgrounds grant a feature and bonus languages
- `homebrew_example_5_2.json` - SRD 5.2 (2024 rules): backgrounds grant ability score bonuses and a feat, species provide traits only

See the content packs in `assets/data/custom_content/`:
- `tashas_cauldron_of_everything.json` - Full Artificer base class with 4 subclasses and levels 1-20
- `xanathars_guide_to_everything.json` - Subclasses for all 12 SRD classes

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
5. **Test in App:** Always test imported content by creating a character

---

## Future Enhancements

Planned additions to the schema:
- Feats
- Equipment and Magic Items
- Spells
- Monsters (for DM tools)
- Version ranges (e.g., compatible with 5.1 AND 5.2)
- Dependencies between content packs

---

## Support

If you encounter validation errors or need help:
1. Check the error message for specific field issues
2. Compare your JSON to the example file
3. Verify all required fields are present
4. Report issues at: [GitHub Issues](https://github.com/anthropics/claude-code/issues)
