# Custom Content Guide

## Introduction

Adventura now supports importing custom D&D 5e content! This feature allows you to:

- ✅ Import classes and subclasses from official D&D books (Xanathar's Guide, Tasha's Cauldron, etc.)
- ✅ Add homebrew species, classes, and backgrounds
- ✅ Import custom name data for species-specific name generation
- ✅ Enable/disable content packs as needed
- ✅ Track which content source each character uses
- ✅ Keep all your custom content organized in one place

---

## Quick Start

### 1. Get a Content Pack

You can either:
- **Create your own** JSON file (see [custom_content_schema.md](./custom_content_schema.md))
- **Download** pre-made packs (coming soon!)
- **Use the example** at [custom_content_example.json](./custom_content_example.json)

### 2. Import the Content Pack

1. Open **Settings** in the app
2. Tap **Custom content**
3. Tap **Import JSON Content Pack**
4. Select your `.json` file
5. Review any warnings (if present)
6. Tap **Continue** to import

### 3. Create a Character

1. Start creating a new character
2. When selecting class/species/background, you'll see:
   - **SRD content** (built-in official D&D content)
   - **Custom content** (your imported content packs)
3. Characters automatically track which source they use

---

## Managing Custom Content

### View Imported Content

**Settings → Custom content** shows all your imported content packs.

Each pack displays:
- **Name** and **version** (e.g., "Xanathar's Guide to Everything - srd_5_2")
- **Status** (Active/Inactive)
- **Import date**
- **Content count** (number of species, classes, backgrounds)

### Enable/Disable Content

Use the **toggle switch** to activate or deactivate a content pack.

- **Active** = Available when creating characters
- **Inactive** = Hidden from character creation (existing characters unchanged)

### View Pack Details

Tap **Details** to see:
- Full description
- SRD version compatibility
- Breakdown of content (species, classes, backgrounds)

### Delete Content

Tap **Delete** to permanently remove a content pack.

**⚠️ Warning:** This removes all associated content. Characters using this content will keep their data, but you won't be able to create new characters with it.

---

## Content Sources

### Official SRD

The app includes the official D&D 5e System Reference Document (SRD):
- **SRD 5.1** - 2014 D&D rules
- **SRD 5.2** - 2024 D&D rules

This content is always available and cannot be disabled.

### Custom Content

Any imported content packs appear alongside SRD content:
- **Source badges** show where content came from
- Characters track their content source
- You can enable/disable packs without affecting existing characters

### Understanding the Version System

The `version` field in content packs (`srd_5_1` or `srd_5_2`) indicates **which rule set** the content is formatted for:

- **`srd_5_1`** - Content designed for 2014 D&D rules
  - Use for books published 2014-2023 (Xanathar's Guide 2017, Tasha's Cauldron 2020, etc.)
  - Compatible with characters using 2014 rules in the app

- **`srd_5_2`** - Content designed for 2024 D&D rules
  - Use for books published 2024 onwards
  - Compatible with characters using 2024 rules in the app

**Important:** The version field tells the app how to interpret the content's mechanics, not which books you can use. Books published before 2024 (like Xanathar's Guide) should use `srd_5_1` since they were designed for the 2014 rule set.

**Example:**
- ✅ Xanathar's Guide (2017) → use `"version": "srd_5_1"`
- ✅ Tasha's Cauldron (2020) → use `"version": "srd_5_1"`
- ✅ Player's Handbook 2024 → use `"version": "srd_5_2"`

---

## Creating Content Packs

### JSON Format

Content packs use JSON format. Here's a minimal example:

```json
{
  "source": {
    "id": "my_homebrew",
    "name": "My Homebrew Content",
    "version": "srd_5_2"
  },
  "classes": [
    {
      "id": "my_class",
      "name": "Blood Hunter",
      "description": "A martial striker that uses blood magic"
    }
  ]
}
```

### Full Schema

See [custom_content_schema.md](./custom_content_schema.md) for:
- Complete field reference
- Validation rules
- Advanced examples
- Error messages

### Tips for Creating Packs

1. **Start Simple:** Begin with 1-2 items
2. **Use Clear Names:** Descriptive IDs help organize content
3. **Match SRD Version:** Choose `srd_5_1` or `srd_5_2` based on rules
4. **Test Early:** Import after each section to catch errors
5. **Add Descriptions:** Help users understand your content

---

## Validation

The app validates all imported content to ensure data integrity.

### What Gets Validated

✅ **JSON structure** - Must be valid JSON
✅ **Required fields** - All mandatory fields present
✅ **Data types** - Strings, numbers, arrays in correct format
✅ **SRD compatibility** - Version must be `srd_5_1` or `srd_5_2`
✅ **Field formats** - IDs, level keys, ability scores

### Common Errors

| Error | Fix |
|-------|-----|
| "Missing required field: name" | Add the `name` field |
| "Invalid ability score: FOO" | Use only STR, DEX, CON, INT, WIS, CHA |
| "Invalid level key: 3" | Use format like "3rd" not "3" |
| "Background cannot have more than 4 skill proficiencies" | Reduce skills to 4 or fewer |

---

## Character Data

### Source Tracking

Every character tracks:
- **Species source** (e.g., "srd" or "xanathars_guide")
- **Class source**
- **Background source**

This information is saved in the character's data and synced to the cloud.

### Backward Compatibility

- **Existing characters** automatically use `source: "srd"`
- **New characters** track their source explicitly
- **Older app versions** ignore source fields (graceful degradation)

### Data Safety

- Disabling a content pack doesn't delete character data
- Deleting a pack doesn't affect existing characters
- Characters maintain all their features even if source is removed

---

## Custom Species Names

You can add custom name data for any species (built-in or custom) by including a `"names"` array in your content pack. These names are used by the name generator during character creation.

### How It Works

- Imported names are **merged** with the built-in name pool (additive, never replaces)
- More imported names = more variety in character name suggestions
- Names support gender categories: `male`, `female`, `neutral`, and optional `familyNames`
- If a custom species has no built-in names, imported names will be the only source

### Example

```json
{
  "source": {
    "id": "my_names",
    "name": "Extra Names Pack",
    "version": "srd_5_2"
  },
  "names": [
    {
      "species": "elf",
      "male": ["Aelindor", "Berethil", "Caladorn"],
      "female": ["Adria", "Bethrynna", "Caelynn"],
      "neutral": ["Arannis", "Cyren", "Daeron"],
      "familyNames": ["Amakiir", "Galanodel", "Liadon"]
    }
  ]
}
```

See [custom_content_schema.md](./custom_content_schema.md) for the full field reference.

---

## Adding Subraces and Variants

### Understanding Species Variants

The official SRD content is limited compared to the full Player's Handbook. For example:

**What's in SRD 5.1 (2014 Rules):**
- Dwarf (Hill Dwarf only)
- Elf (High Elf only)
- Halfling (Lightfoot only)
- Gnome (Rock Gnome only)
- Human (Standard only)

**What's NOT in SRD (Player's Handbook only):**
- Mountain Dwarf, Wood Elf, Dark Elf (Drow), Stout Halfling, Forest Gnome, Variant Human

**What's in SRD 5.2 (2024 Rules):**
- Elf Lineage (High/Wood/Dark Elf) ✓
- Gnomish Lineage (Forest/Rock Gnome) ✓
- Tiefling Fiendish Legacy ✓
- Goliath Giant Ancestry ✓

You can add these missing subraces through custom content!

### Adding Subraces with Custom Content

To add subraces that aren't in the SRD (like Mountain Dwarf or Variant Human), you need to create a custom content pack.

#### Example: Adding Mountain Dwarf

```json
{
  "name": "Player's Handbook Dwarf Subraces",
  "version": "srd_5_1",
  "description": "Mountain Dwarf and Hill Dwarf subraces from the Player's Handbook 2014",
  "source": "players_handbook",
  "species": [
    {
      "name": "Mountain Dwarf",
      "description": "Mountain dwarves are strong and hardy, accustomed to a difficult life in rugged terrain. They're taller and leaner than their hill dwarf cousins. They value the skills of warfare.",
      "traits": {
        "Creature Type": "Humanoid",
        "Size": "Medium (~4–5 feet tall)",
        "Speed": "25 feet",
        "Darkvision": "See in dim light within 60 feet"
      },
      "abilityScoreBonuses": {
        "CON": 2,
        "STR": 2
      },
      "features": [
        {
          "name": "Darkvision",
          "description": "You can see in dim light within 60 feet of you as if it were bright light, and in darkness as if it were dim light."
        },
        {
          "name": "Dwarven Resilience",
          "description": "You have advantage on saving throws against poison, and you have resistance against poison damage."
        },
        {
          "name": "Dwarven Combat Training",
          "description": "You have proficiency with the battleaxe, handaxe, light hammer, and warhammer."
        },
        {
          "name": "Dwarven Armor Training",
          "description": "You have proficiency with light and medium armor."
        },
        {
          "name": "Tool Proficiency",
          "description": "You gain proficiency with one set of artisan's tools.",
          "choices": ["Smith's Tools", "Brewer's Supplies", "Mason's Tools"],
          "choiceDescription": "Choose artisan's tools:",
          "maxChoices": 1
        },
        {
          "name": "Stonecunning",
          "description": "Whenever you make an Intelligence (History) check related to the origin of stonework, you are considered proficient and add double your proficiency bonus."
        },
        {
          "name": "Languages",
          "description": "You can speak, read, and write Common and Dwarvish."
        }
      ]
    }
  ]
}
```

#### Example: Adding Variant Human

```json
{
  "name": "Player's Handbook - Variant Human",
  "version": "srd_5_1",
  "description": "Variant Human option from the Player's Handbook 2014",
  "source": "players_handbook",
  "species": [
    {
      "name": "Variant Human",
      "description": "Humans are adaptable and ambitious. The variant human trades the standard +1 to all ability scores for more customization options including a feat at 1st level.",
      "traits": {
        "Creature Type": "Humanoid",
        "Size": "Medium (~5–6 feet tall)",
        "Speed": "30 feet"
      },
      "abilityScoreBonuses": {},
      "features": [
        {
          "name": "Ability Score Increase",
          "description": "Two different ability scores of your choice increase by 1.",
          "note": "During character creation, manually apply +1 to two ability scores of your choice"
        },
        {
          "name": "Skills",
          "description": "You gain proficiency in one skill of your choice.",
          "choices": ["Acrobatics", "Animal Handling", "Arcana", "Athletics", "Deception", "History", "Insight", "Intimidation", "Investigation", "Medicine", "Nature", "Perception", "Performance", "Persuasion", "Religion", "Sleight of Hand", "Stealth", "Survival"],
          "choiceDescription": "Choose a skill proficiency:",
          "maxChoices": 1
        },
        {
          "name": "Feat",
          "description": "You gain one feat of your choice.",
          "note": "Choose from available feats during character creation"
        },
        {
          "name": "Languages",
          "description": "You can speak, read, and write Common and one extra language of your choice.",
          "choices": ["Abyssal", "Celestial", "Draconic", "Dwarvish", "Elvish", "Giant", "Gnomish", "Goblin", "Halfling", "Infernal", "Orc", "Primordial", "Sylvan", "Undercommon"],
          "choiceDescription": "Choose an extra language:",
          "maxChoices": 1
        }
      ]
    }
  ]
}
```

### Important Notes for Subraces

1. **Source Tracking**: Always include a `"source"` field (e.g., `"players_handbook"`) so users know where the content comes from

2. **Ability Score Bonuses**: For variant humans or other species with flexible ability score choices, you can:
   - Leave `abilityScoreBonuses` empty: `{}`
   - Add a note in the features explaining the player should manually apply bonuses

3. **Licensing**: Only create content packs for:
   - Content you own (official D&D books you purchased)
   - Homebrew content you created
   - Content with explicit permission to share

4. **SRD Compliance**: The built-in SRD content is free/licensed. Player's Handbook content requires owning the book.

---

## Troubleshooting

### Import Fails

**Problem:** "Validation failed" error

**Solutions:**
1. Check the error message for specific fields
2. Compare your JSON to the example file
3. Validate JSON syntax (use jsonlint.com)
4. Ensure all required fields are present

### Content Not Appearing

**Problem:** Imported content doesn't show in character creation

**Solutions:**
1. Check if the content pack is **Active** (toggle in settings)
2. Verify the SRD version matches your selected rules (5.1 vs 5.2)
3. Restart the app to refresh content lists

### Characters Missing Content

**Problem:** Character references deleted content

**Solution:** Characters keep all their data even if the source is removed. The character will still function normally, but you can't create new characters with that content until you re-import it.

---

## Best Practices

### For Official D&D Books

When importing official content (Xanathar's, Tasha's, etc.):
- Use clear source IDs: `xanathars_guide`, `tashas_cauldron`
- Include book name in the pack name
- Add descriptions explaining what's included
- Match the SRD version to the book's rules

### For Homebrew Content

When creating homebrew:
- Use unique IDs to avoid conflicts
- Document your design choices in descriptions
- Test balance with one character before importing many
- Consider creating multiple packs for different homebrew collections

### For Sharing

If sharing content packs:
- Include a README explaining the content
- Specify SRD version compatibility
- List any dependencies or prerequisites
- Provide attribution for homebrew sources

---

## Example Use Cases

### 1. Official Book Content

**Goal:** Add Cavalier and Samurai subclasses from Xanathar's Guide

**Steps:**
1. Create JSON with both subclasses
2. Set `source.id` to `"xanathars_guide"`
3. Set `version` to `"srd_5_1"` (Xanathar's was published in 2017 for 2014 rules)
4. Import and activate

### 2. Homebrew Campaign

**Goal:** Add custom species for your campaign setting

**Steps:**
1. Create JSON with your custom species
2. Use unique IDs like `"myworld_dragonborn_variant"`
3. Set `version` to match your rules
4. Import and share with players

### 3. Playtesting

**Goal:** Test homebrew content before finalizing

**Steps:**
1. Create pack with experimental content
2. Import and activate
3. Create test characters
4. Disable pack when not testing
5. Update JSON based on feedback
6. Re-import updated version

---

## Future Features

Planned enhancements:

- 📦 **Content marketplace** - Download community-created packs
- ✏️ **In-app editor** - Create content without JSON editing
- 🔄 **Pack updates** - Auto-update imported packs
- 🏷️ **Tags and filters** - Better content organization
- 🎲 **Feats and spells** - Additional content types
- 🤝 **Pack sharing** - Export and share with friends

---

## FAQ

**Q: Can I use content from official D&D books?**
A: For personal use, yes. For distribution, check copyright laws and WotC's fan content policy.

**Q: Will custom content sync to the cloud?**
A: Yes! Custom content is stored locally and syncs with your account.

**Q: Can I edit imported content?**
A: Currently no. Delete and re-import with changes. An in-app editor is planned.

**Q: What if I import the same pack twice?**
A: The app updates the existing pack with the new data.

**Q: Can custom content break my characters?**
A: No. Characters save all their data independently of content packs.

**Q: Is there a size limit for content packs?**
A: No hard limit, but packs with 50+ items may be slow to load.

---

## Support

Need help?

- **Documentation:** [custom_content_schema.md](./custom_content_schema.md)
- **Example:** [custom_content_example.json](./custom_content_example.json)
- **Issues:** [GitHub](https://github.com/anthropics/claude-code/issues)

---

## Credits

Custom content system designed for Adventura.

**Contributors:**
- Architecture & implementation
- JSON schema design
- Validation system
- UI/UX design

**Special Thanks:**
- D&D 5e SRD by Wizards of the Coast
- Community feedback and testing

---

*Happy adventuring! 🎲*
