# Adventura Custom Content

Documentation and examples for creating custom D&D 5e content packs for [Adventura](https://github.com/saentari/adventura), a D&D 5th Edition companion app.

## Overview

Adventura supports importing custom content packs as JSON files. You can add classes, subclasses, species, backgrounds, feats, spells, equipment, magic items, and name data for use during character creation.

Content packs target a specific rules version:

- **SRD 5.1** — 2014 D&D rules (books published 2014-2023)
- **SRD 5.2** — 2024 D&D rules (books published 2024+)

## Files

| File | Description |
|------|-------------|
| [custom_content_guide.md](custom_content_guide.md) | User guide covering how to import, manage, and create content packs |
| [custom_content_schema.md](custom_content_schema.md) | Technical schema reference with field definitions, validation rules, and examples |
| [homebrew_example_5_1.json](homebrew_example_5_1.json) | Complete example pack for 2014 rules (SRD 5.1) |
| [homebrew_example_5_2.json](homebrew_example_5_2.json) | Complete example pack for 2024 rules (SRD 5.2) |

## Quick start

1. Download one of the example JSON files (or create your own using the schema reference)
2. Open **Settings** in Adventura
3. Tap **Custom content** > **Import JSON Content Pack**
4. Select your `.json` file and confirm

## Example content

Both example files contain the same fictional homebrew content to demonstrate every supported content type:

- **Runescribe** — a full base class (Intelligence spellcaster with runic magic)
- **Ironclad** — a Fighter subclass (damage reduction tank)
- **Glimmerfolk** — a custom species (crystalline underground dwellers)
- **Deepvein Glimmerfolk** — a species variant/lineage
- **Ruin Delver** — a background (ancient ruins explorer)
- **Glyph Touched & Crystal Marksman** — feats
- **Crystalline Barrier** — a custom spell
- **Equipment** — Crystal-Tipped Spear, Glimmerweave Chain Shirt, Rune-Etched Lantern, Deep Cavern Lizard
- **Magic items** — Frostbrand Glaive, Cloak of Crystal Resonance, Potion of Runic Sight, Stoneheart Shield
- **Names** — Glimmerfolk name generation data

The key difference between the two files: in 5.1, species grant ability score bonuses and backgrounds grant a feature; in 5.2, backgrounds grant ability scores and a feat while species provide traits only.

## License

This project is licensed under the [MIT License](LICENSE).
