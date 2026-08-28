# Adventura Custom Content

Documentation and examples for creating custom D&D 5e content packs for [Adventura](https://github.com/saentari/adventura), a D&D 5th Edition companion app.

## Overview

Adventura supports importing custom content packs as JSON files. You can add classes, subclasses, species, lineages, backgrounds, feats, spells, equipment, magic items, deities, eldritch invocations, monsters, and name data for use during character creation.

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

Both example files carry the same fictional homebrew content, so diffing them
shows exactly what changes between rule sets:

- **Runescribe** — a full base class: Intelligence prepared caster, its own subclasses, a level-scaling multi-select feature, and a starting-equipment block
- **Tradition of the Warden / Tradition of the Sage** — subclasses of that homebrew class, one of which carries a feature choice with mechanical effects
- **Ironclad** — a Fighter subclass (damage reduction tank) that also grants proficiencies
- **Pact of the Crystal** — a Warlock Pact Boon added to an existing SRD feature
- **Glimmerfolk** — a species with darkvision, a resistance, a save advantage, level-scaling spell grants and a skill choice
- **Deepvein Glimmerfolk / Prismborn Glimmerfolk** — two lineages, the second showing the flexible-choice pickers (floating ability increase, size, feat)
- **Ruin Delver** — a background, in its 2014 and its 2024 shape
- **Glyph Touched & Crystal Marksman** — feats with categories, effects and player choices
- **Runic Mark, Read the Stones, Crystalline Barrier** — a cantrip, a ritual, and a leveled spell
- **Eldritch Shroud & Shard Ward** — warlock eldritch invocations
- **Deities** — Vexith the Prismbearer (NG), Or'kael the Bound (LN), Silaxis the Drowned (CE)
- **Equipment** — a versatile weapon, medium armor, a tool that helps ability checks, gear and a mount
- **Magic items** — eleven of them, covering every effect the app resolves: AC bonuses and conditional ones, an AC formula, saving-throw bonuses, resistance, a granted sense, an open-base weapon that expands over every sword, and consumables that heal, cure, grant temporary hit points, buff a check and grant a status
- **Monster** — Crystalwyrm (CR 8 dragon) in 5.1, Gloomspore (CR 4 plant) in 5.2
- **Names** — Glimmerfolk names plus adventure and party word lists

The key difference between the two files: under 5.1 species grant ability score
bonuses and the background grants a feature plus bonus languages; under 5.2 the
background grants the ability score bonuses and a feat, and the species provides
traits and a size choice.

Everything above is verified against the app on every test run — if the app
stops reading a field, the example stops demonstrating it and a test fails.

## License

This project is licensed under the [MIT License](LICENSE).
