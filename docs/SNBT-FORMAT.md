# FTB Quests SNBT Format Documentation

This document provides comprehensive documentation on the SNBT (Stringified NBT) format used by FTB Quests for quest configuration files located in `/config/ftbquests/quests`.

## Table of Contents

- [Overview](#overview)
- [File Structure](#file-structure)
- [ID Format and Validation](#id-format-and-validation)
- [Main Quest File](#main-quest-file)
- [Chapter Groups](#chapter-groups)
- [Chapters](#chapters)
- [Quests](#quests)
- [Tasks](#tasks)
- [Rewards](#rewards)
- [Quest Links](#quest-links)
- [Reward Tables](#reward-tables)
- [Examples](#examples)
- [Edge Cases and Troubleshooting](#edge-cases-and-troubleshooting)

## Overview

FTB Quests stores quest data in SNBT format, which is a human-readable text representation of Minecraft's NBT (Named Binary Tag) format. The quest system uses a file-based structure where different quest components are organized in separate files.

### Key Concepts

- **Quest File**: The main container that holds all quest-related data
- **Chapter Groups**: Collections of related chapters (tabs in the UI)
- **Chapters**: Collections of quests (displayed as quest boards)
- **Quests**: Individual quest objectives with tasks and rewards
- **Tasks**: Objectives players must complete
- **Rewards**: Items, commands, or other benefits given upon quest completion
- **Quest Links**: References to quests in other chapters

## File Structure

The quest files are organized in the following directory structure:

```
/config/ftbquests/quests/
├── data.snbt                    # Main quest file configuration
├── chapter_groups.snbt          # Chapter group definitions
├── chapters/                    # Individual chapter files
│   ├── <chapter_id>.snbt
│   └── ...
├── reward_tables/               # Reward table definitions
│   ├── <table_id>.snbt
│   └── ...
└── lang/                        # Translation files (optional)
    └── ...
```

## ID Format and Validation

### ID Structure

All quest objects (quests, chapters, tasks, rewards, etc.) use **16-character hexadecimal IDs**. These IDs are crucial for referencing objects throughout the quest system.

**Format**: `XXXXXXXXXXXXXXXX` (16 hex characters, case-insensitive)

**Example**: `"1A2B3C4D5E6F0A1B"`

### Valid ID Characteristics

- **Length**: Must be exactly 16 hexadecimal characters
- **Characters**: Only `0-9`, `a-f`, `A-F` are allowed
- **Range**: IDs are stored as signed 64-bit longs
- **Positive Values Only**: First hex digit must be `0-7` (not `8-9`, `A-F`) to ensure positive long values
- **Uniqueness**: Each ID must be unique across the entire quest file
- **Reserved IDs**: 
  - `0000000000000000` (0) - Reserved for null/missing objects
  - `0000000000000001` (1) - Reserved for the quest file itself

### ID Generation Rules

When manually creating IDs:

1. **Start with 0-7**: First hex digit must be `0-7` to avoid negative values (e.g., `1A2B...` not `9A0B...`)
2. **Use Random Values**: Generate random 16-character hex strings
3. **Avoid Collisions**: Never reuse an ID that already exists
4. **Avoid Reserved**: Don't use `0000000000000000` or `0000000000000001`
5. **Case Insensitive**: `1A2B` and `1a2b` are the same ID

### ID Parsing

IDs can be specified in multiple ways:

- **Hex String**: `"1A2B3C4D5E6F0A1B"` - Standard format
- **With Hash**: `"#1A2B3C4D5E6F0A1B"` - Hash prefix is allowed and ignored
- **Tag Reference**: `"#tag_name"` - References an object by its tag

### Invalid ID Examples

❌ **Too Short**: `"1A2B3C"` (only 6 characters)
❌ **Too Long**: `"1A2B3C4D5E6F0A1B99"` (18 characters)
❌ **Invalid Characters**: `"1A2B3C4D5E6F0G1H"` (contains G and H)
❌ **Reserved**: `"0000000000000000"` (reserved for null)
❌ **Negative Value**: `"9A0B1C2D3E4F5678"` (starts with 9, becomes negative - will be replaced)

## Main Quest File

**File**: `data.snbt`

This file contains the global quest system configuration.

### Structure

```snbt
{
	version: 13
	title: "Quest Book Title"
	icon: "minecraft:book"
	default_reward_team: false
	default_team_consume_items: false
	default_autoclaim_rewards: "disabled"
	default_quest_shape: "circle"
	default_quest_disable_jei: false
	hide_excluded_quests: false
	drop_loot_crates: false
	loot_crate_no_drop: {
		passive: 4000
		monster: 600
		boss: 0
	}
	emergency_items: [
		{
			id: "minecraft:apple"
			count: 16
		}
	]
	emergency_items_cooldown: 300
	disable_gui: false
	grid_scale: 0.5d
	pause_game: false
	lock_message: ""
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `version` | int | 13 | Quest file format version (current: 13) |
| `title` | string | "" | Title displayed in the quest book |
| `icon` | string | "" | Icon item ID for the quest book |
| `default_reward_team` | boolean | false | Whether rewards are given per-team by default |
| `default_team_consume_items` | boolean | false | Whether items are consumed per-team by default |
| `default_autoclaim_rewards` | string | "disabled" | Default reward auto-claim: "disabled", "enabled", or "no_toast" |
| `default_quest_shape` | string | "" | Default shape for all quests (e.g., "circle", "square", "diamond") |
| `default_quest_disable_jei` | boolean | false | Whether to disable JEI integration by default |
| `hide_excluded_quests` | boolean | false | Hide quests excluded from progression |
| `drop_loot_crates` | boolean | false | Whether to drop loot crates from entities |
| `loot_crate_no_drop` | compound | - | Entity weights for no loot crate drops |
| `emergency_items` | list | [] | Items given in emergency situations |
| `emergency_items_cooldown` | int | 300 | Cooldown in seconds for emergency items |
| `disable_gui` | boolean | false | Disable the quest GUI |
| `grid_scale` | double | 0.5 | Scale of the quest grid |
| `pause_game` | boolean | false | Whether to pause the game when opening quest UI |
| `lock_message` | string | "" | Message shown when quests are locked |

### Loot Crate Entity Weights

The `loot_crate_no_drop` compound defines weights for different entity types:

- `passive` - Passive mobs (animals)
- `monster` - Hostile mobs
- `boss` - Boss mobs

Higher values = less likely to drop crates (0 = always drop).

## Chapter Groups

**File**: `chapter_groups.snbt`

Chapter groups organize chapters into tabs in the quest UI.

### Structure

```snbt
{
	chapter_groups: [
		{
			id: "1A2B3C4D5E6F0A1B"
			title: "Getting Started"
			icon: "minecraft:grass_block"
		}
		{
			id: "2B3C4D5E6F0A1B2C"
			title: "Advanced Quests"
			icon: "minecraft:diamond_block"
		}
	]
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Unique 16-character hex ID |
| `title` | string | No | Display title for the chapter group |
| `icon` | string | No | Item ID for the group's icon |

### Notes

- The default chapter group (ungrouped chapters) is not listed in this file
- Chapter groups appear as tabs in the quest UI
- Order in the list determines tab order

## Chapters

**Files**: `chapters/<chapter_id>.snbt`

Chapters contain collections of quests and are displayed as quest boards.

### Structure

```snbt
{
	id: "3C4D5E6F0A1B2C3D"
	group: "1A2B3C4D5E6F0A1B"
	order_index: 0
	filename: "my_chapter"
	title: "Chapter Title"
	icon: "minecraft:crafting_table"
	default_quest_shape: "circle"
	default_quest_size: 1.0d
	default_hide_dependency_lines: false
	always_invisible: false
	default_min_width: 0
	progression_mode: "default"
	consume_items: "default"
	hide_quest_details_until_startable: false
	hide_quest_until_deps_visible: false
	hide_quest_until_deps_complete: false
	default_repeatable_quest: false
	require_sequential_tasks: false
	autofocus_id: ""
	images: [
		{
			x: 0.0d
			y: 0.0d
			width: 1.0d
			height: 1.0d
			rotation: 0.0d
			image: "minecraft:textures/block/stone.png"
			hover: []
			click: ""
			dev: false
			corner: false
			fit: false
			align_to_grid: false
		}
	]
	quests: [
		{
			id: "4D5E6F0A1B2C3D4E"
			# ... quest properties ...
		}
	]
	quest_links: [
		{
			id: "5E6F0A1B2C3D4E5F"
			# ... quest link properties ...
		}
	]
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | string | - | Unique 16-character hex ID |
| `group` | string | "" | Chapter group ID (empty = default group) |
| `order_index` | int | - | Order within the chapter group |
| `filename` | string | - | File basename (without .snbt) |
| `title` | string | "" | Display title |
| `icon` | string | "" | Item ID for chapter icon |
| `default_quest_shape` | string | "" | Default shape for quests in this chapter |
| `default_quest_size` | double | 1.0 | Default size multiplier for quests |
| `default_hide_dependency_lines` | boolean | false | Hide dependency lines by default |
| `always_invisible` | boolean | false | Chapter is always hidden |
| `default_min_width` | int | 0 | Minimum width for quest text in pixels |
| `progression_mode` | string | "default" | "default", "linear", or "flexible" |
| `consume_items` | string | "default" | "default", "true", or "false" (tristate) |
| `hide_quest_details_until_startable` | boolean | false | Hide quest details until dependencies met |
| `hide_quest_until_deps_visible` | boolean | false | Hide quest until dependencies visible |
| `hide_quest_until_deps_complete` | boolean | false | Hide quest until dependencies complete |
| `default_repeatable_quest` | boolean | false | Quests repeatable by default |
| `require_sequential_tasks` | boolean | false | Tasks must be completed in order |
| `autofocus_id` | string | "" | Quest ID to focus when opening chapter |
| `images` | list | [] | Background/decoration images |
| `quests` | list | [] | List of quests in this chapter |
| `quest_links` | list | [] | List of quest links to other chapters |

### Progression Modes

- **`default`**: Uses the global default progression mode
- **`linear`**: Quests must be completed in a specific order
- **`flexible`**: Quests can be completed in any order (default)

### Tristate Values

Properties using tristate values (`consume_items`) can be:
- `"default"` - Use the parent/global default
- `"true"` - Enable
- `"false"` - Disable

### Chapter Images

Background images can be added to chapters for decoration:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `x` | double | 0.0 | X position on the quest grid |
| `y` | double | 0.0 | Y position on the quest grid |
| `width` | double | 1.0 | Width in grid units |
| `height` | double | 1.0 | Height in grid units |
| `rotation` | double | 0.0 | Rotation in degrees |
| `image` | string | - | Resource location of the image texture |
| `hover` | list | [] | Text to show on hover |
| `click` | string | "" | Command to execute on click |
| `dev` | boolean | false | Only visible in dev mode |
| `corner` | boolean | false | Pin to corner |
| `fit` | boolean | false | Fit to bounds |
| `align_to_grid` | boolean | false | Snap to grid |

## Quests

Quests are defined within chapter files in the `quests` list.

### Structure

```snbt
{
	id: "4D5E6F0A1B2C3D4E"
	title: "Quest Title"
	icon: "minecraft:diamond"
	x: 0.0d
	y: 0.0d
	shape: "circle"
	subtitle: ["Subtitle text"]
	description: [
		"First line of description"
		"Second line of description"
	]
	dependencies: ["3C4D5E6F0A1B2C3D"]
	dependency_requirement: "all_completed"
	min_required_dependencies: 0
	hide_dependency_lines: "default"
	hide_dependent_lines: false
	hide_until_deps_visible: "default"
	hide_until_deps_complete: "default"
	hide_text_until_complete: "default"
	hide_details_until_startable: "default"
	disable_recipe_mod: "default"
	size: 1.0d
	icon_scale: 1.0d
	optional: false
	min_width: 0
	can_repeat: "default"
	invisible: false
	invisible_until_tasks: 0
	ignore_reward_blocking: false
	progression_mode: "default"
	require_sequential_tasks: "default"
	hide_lock_icon: false
	max_completable_dependents: 0
	repeat_cooldown: 0
	guide_page: ""
	tasks: [
		{
			id: "5F6A0B1C2D3E4F5A"
			type: "item"
			# ... task-specific properties ...
		}
	]
	rewards: [
		{
			id: "6A0B1C2D3E4F5A6B"
			type: "item"
			# ... reward-specific properties ...
		}
	]
}
```

### Core Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | string | - | Unique 16-character hex ID |
| `title` | string | "" | Quest title |
| `icon` | string | "" | Item ID for quest icon |
| `x` | double | 0.0 | X position on quest grid |
| `y` | double | 0.0 | Y position on quest grid |
| `shape` | string | "" | Quest shape (empty = use chapter default) |
| `subtitle` | list | [] | Subtitle lines |
| `description` | list | [] | Description lines |
| `size` | double | 0.0 | Size multiplier (0 = use chapter default) |
| `icon_scale` | double | 1.0 | Icon scale multiplier |
| `min_width` | int | 0 | Minimum width for quest text |
| `guide_page` | string | "" | Guide book page ID |

### Dependency Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dependencies` | list | [] | List of quest/chapter IDs this quest depends on |
| `dependency_requirement` | string | "all_completed" | How dependencies are evaluated |
| `min_required_dependencies` | int | 0 | Minimum number of dependencies required |
| `hide_dependency_lines` | tristate | "default" | Hide lines to dependencies |
| `hide_dependent_lines` | boolean | false | Hide lines from dependents |
| `max_completable_dependents` | int | 0 | Maximum number of dependent quests that can be completed |

### Dependency Requirements

- **`all_completed`**: All dependencies must be completed (default)
- **`one_completed`**: At least one dependency must be completed
- **`all_started`**: All dependencies must be started
- **`one_started`**: At least one dependency must be started

### Visibility Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `hide_until_deps_visible` | tristate | "default" | Hide until dependencies are visible |
| `hide_until_deps_complete` | tristate | "default" | Hide until dependencies are complete |
| `hide_text_until_complete` | tristate | "default" | Hide description until complete |
| `hide_details_until_startable` | tristate | "default" | Hide task/reward details until startable |
| `invisible` | boolean | false | Quest is completely invisible |
| `invisible_until_tasks` | int | 0 | Invisible until X tasks completed |
| `hide_lock_icon` | boolean | false | Hide the lock icon when locked |

### Behavior Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `optional` | boolean | false | Quest is optional for progression |
| `can_repeat` | tristate | "default" | Quest can be repeated |
| `repeat_cooldown` | int | 0 | Cooldown between repeats (seconds) |
| `disable_recipe_mod` | tristate | "default" | Disable JEI/REI integration |
| `ignore_reward_blocking` | boolean | false | Ignore reward blocking rules |
| `progression_mode` | string | "default" | Quest progression mode |
| `require_sequential_tasks` | tristate | "default" | Tasks must be done in order |

### Quest Shapes

Common quest shapes include:
- `circle` - Circular quest icon
- `square` - Square quest icon
- `diamond` - Diamond-shaped icon
- `rsquare` - Rounded square
- `pentagon` - Pentagon shape
- `hexagon` - Hexagon shape
- `octagon` - Octagon shape
- `heart` - Heart shape
- `gear` - Gear shape

The available shapes depend on the loaded resource packs.

## Tasks

Tasks are objectives that players must complete. They are defined in the `tasks` list within a quest.

### Common Task Properties

All tasks share these properties:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | string | - | Unique 16-character hex ID |
| `type` | string | - | Task type identifier |
| `title` | string | "" | Task title (overrides default) |
| `icon` | string | "" | Custom icon (overrides default) |
| `optional` | boolean | false | Task is optional |

### Task Types

#### Item Task

**Type**: `"item"`

Requires collecting specific items.

```snbt
{
	id: "5F6A0B1C2D3E4F5A"
	type: "item"
	item: "minecraft:diamond"
	count: 64L
	consume_items: "default"
	only_from_crafting: "default"
	match_components: "none"
	task_screen_only: false
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `item` | string or compound | - | Item ID (string) or item stack NBT (compound) |
| `count` | long | 1 | Number of items required |
| `consume_items` | tristate | "default" | Whether items are consumed |
| `only_from_crafting` | tristate | "default" | Only count crafted items |
| `match_components` | string | "none" | Component matching: "none", "fuzzy", "exact" |
| `task_screen_only` | boolean | false | Only submit via task screen |

**Note**: Items can be specified as:
- Simple string: `item: "minecraft:diamond"`
- Compound tag (1.21+): `item: { id: "minecraft:diamond" }`

#### Checkmark Task

**Type**: `"checkmark"`

Simple manual completion task.

```snbt
{
	id: "6A0B1C2D3E4F5A6B"
	type: "checkmark"
	title: "Read the instructions"
}
```

#### XP Task

**Type**: `"xp"`

Requires collecting experience points.

```snbt
{
	id: "7B1C2D3E4F5A6B7C"
	type: "xp"
	value: 100L
	points: false
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `value` | long | 100 | Amount of XP required |
| `points` | boolean | false | If true, uses XP points instead of levels |

#### Dimension Task

**Type**: `"dimension"`

Requires entering a specific dimension.

```snbt
{
	id: "1C2D3E4F5A6B7C8D"
	type: "dimension"
	dimension: "minecraft:the_nether"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dimension` | string | - | Dimension resource location |

#### Stat Task

**Type**: `"stat"`

Requires achieving a stat milestone.

```snbt
{
	id: "2D3E4F5A6B7C8D9E"
	type: "stat"
	stat: "minecraft:play_time"
	value: 72000
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `stat` | string | - | Stat resource location |
| `value` | int | 1 | Target stat value |

#### Kill Task

**Type**: `"kill"`

Requires killing specific entities.

```snbt
{
	id: "0E4F5A6B7C8D9E0F"
	type: "kill"
	entity: "minecraft:zombie"
	value: 50L
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `entity` | string | - | Entity type resource location |
| `value` | long | 1 | Number of entities to kill |

#### Location Task

**Type**: `"location"`

Requires reaching a specific location.

```snbt
{
	id: "1F5A6B7C8D9E0F1A"
	type: "location"
	dimension: "minecraft:overworld"
	x: 0
	y: 64
	z: 0
	w: 10
	h: 10
	d: 10
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `dimension` | string | - | Dimension resource location |
| `x`, `y`, `z` | int | 0 | Center coordinates |
| `w`, `h`, `d` | int | 1 | Width, height, depth of area |

#### Advancement Task

**Type**: `"advancement"`

Requires completing a Minecraft advancement.

```snbt
{
	id: "205A6B7C8D9E0F1A"
	type: "advancement"
	advancement: "minecraft:story/mine_diamond"
	criterion: ""
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `advancement` | string | - | Advancement resource location |
| `criterion` | string | "" | Specific criterion (empty = all) |

#### Observation Task

**Type**: `"observation"`

Requires observing (looking at) specific entities or blocks.

```snbt
{
	id: "316B7C8D9E0F1A2B"
	type: "observation"
	observe_type: "block"
	to_observe: "minecraft:diamond_ore"
	timer: 0L
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `observe_type` | string | "entity" | "entity" or "block" |
| `to_observe` | string | - | Entity/block resource location |
| `timer` | long | 0 | Observation time required (ticks) |

#### Biome Task

**Type**: `"biome"`

Requires entering a specific biome.

```snbt
{
	id: "427C8D9E0F1A2B3C"
	type: "biome"
	biome: "minecraft:plains"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `biome` | string | - | Biome resource location |

#### Structure Task

**Type**: `"structure"`

Requires finding a structure.

```snbt
{
	id: "538D9E0F1A2B3C4D"
	type: "structure"
	structure: "minecraft:village_plains"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `structure` | string | - | Structure resource location |

#### Fluid Task

**Type**: `"fluid"`

Requires collecting fluid.

```snbt
{
	id: "649E0F1A2B3C4D5E"
	type: "fluid"
	fluid: "minecraft:water"
	amount: 1000L
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `fluid` | string | - | Fluid resource location |
| `amount` | long | 1000 | Amount in millibuckets |

#### Energy Task

**Type**: `"ftbquests:energy"`

Requires collecting energy (requires mod support).

```snbt
{
	id: "750F1A2B3C4D5E6F"
	type: "ftbquests:energy"
	value: 10000L
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `value` | long | 1000 | Amount of energy required |

#### Stage Task

**Type**: `"ftbquests:gamestage"`

Requires having a specific game stage (Game Stages mod).

```snbt
{
	id: "161A2B3C4D5E6F0A"
	type: "ftbquests:gamestage"
	stage: "stage_name"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `stage` | string | - | Game stage name |

#### Custom Task

**Type**: `"custom"`

Custom scripted task (requires scripting support).

```snbt
{
	id: "172B3C4D5E6F0A1B"
	type: "custom"
	title: "Custom Task"
	icon: "minecraft:command_block"
	check_timer: 20
	max_input: 1L
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `check_timer` | int | 20 | Check interval in ticks |
| `max_input` | long | 1 | Maximum progress value |

## Rewards

Rewards are given when quests are completed. They are defined in the `rewards` list within a quest.

### Common Reward Properties

All rewards share these properties:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | string | - | Unique 16-character hex ID |
| `type` | string | - | Reward type identifier |
| `title` | string | "" | Reward title (overrides default) |
| `icon` | string | "" | Custom icon (overrides default) |
| `team_reward` | boolean | false | Reward is given per team |
| `auto_claim` | string | "default" | Auto-claim: "default", "disabled", "enabled", "no_toast" |
| `exclude_from_claim_all` | boolean | false | Exclude from "claim all" button |

### Reward Types

#### Item Reward

**Type**: `"item"`

Gives items to the player.

```snbt
{
	id: "083C4D5E6F0A1B2C"
	type: "item"
	item: "minecraft:diamond"
	count: 1
	random_bonus: 0
	only_one: false
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `item` | string or compound | - | Item ID (string) or item stack NBT (compound) |
| `count` | int | 1 | Number of items |
| `random_bonus` | int | 0 | Additional random items (0 to this value) |
| `only_one` | boolean | false | Only one player can claim |

**Note**: Items can be specified as:
- Simple string: `item: "minecraft:diamond"`
- Compound tag (1.21+): `item: { id: "minecraft:diamond" }`

#### Choice Reward

**Type**: `"choice"`

Allows choosing one reward from a list.

```snbt
{
	id: "194D5E6F0A1B2C3D"
	type: "choice"
	table_id: "0E4F5A6B7C8D9E0F"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `table_id` | string | - | Reward table ID to choose from |

#### Random Reward

**Type**: `"random"`

Gives a random reward from a table.

```snbt
{
	id: "205E6F0A1B2C3D4E"
	type: "random"
	table_id: "0E4F5A6B7C8D9E0F"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `table_id` | string | - | Reward table ID |

#### Loot Reward

**Type**: `"loot"`

Gives loot from a loot table.

```snbt
{
	id: "316F0A1B2C3D4E5F"
	type: "loot"
	loot_table: "minecraft:chests/simple_dungeon"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `loot_table` | string | - | Loot table resource location |

#### XP Reward

**Type**: `"xp"`

Gives experience points.

```snbt
{
	id: "4270A1B2C3D4E5F6"
	type: "xp"
	xp: 100
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `xp` | int | 100 | Amount of XP points |

#### XP Levels Reward

**Type**: `"xp_levels"`

Gives experience levels.

```snbt
{
	id: "5381B2C3D4E5F6A0"
	type: "xp_levels"
	xp_levels: 5
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `xp_levels` | int | 5 | Number of levels |

#### Command Reward

**Type**: `"command"`

Executes a command.

```snbt
{
	id: "6492C3D4E5F6A0B1"
	type: "command"
	command: "/say @p completed the quest!"
	permission_level: 2
	silent: false
	feedback_message: ""
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `command` | string | - | Command to execute (use `@p` for player) |
| `permission_level` | int | 0 | Permission level (0-4) |
| `silent` | boolean | false | Suppress command output |
| `feedback_message` | string | "" | Custom feedback message |

#### Advancement Reward

**Type**: `"advancement"`

Grants an advancement.

```snbt
{
	id: "750A3D4E5F6A0B1C"
	type: "advancement"
	advancement: "minecraft:story/mine_diamond"
	criterion: ""
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `advancement` | string | - | Advancement resource location |
| `criterion` | string | "" | Specific criterion (empty = all) |

#### Toast Reward

**Type**: `"toast"`

Shows a toast notification.

```snbt
{
	id: "161B4E5F6A0B1C2D"
	type: "toast"
	description: "You completed the quest!"
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `description` | string | "" | Toast message |

#### Stage Reward

**Type**: `"ftbquests:gamestage"`

Adds or removes a game stage.

```snbt
{
	id: "172C5F6A0B1C2D3E"
	type: "ftbquests:gamestage"
	stage: "stage_name"
	remove: false
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `stage` | string | - | Game stage name |
| `remove` | boolean | false | Remove instead of add |

#### Custom Reward

**Type**: `"custom"`

Custom scripted reward (requires scripting support).

```snbt
{
	id: "083D6A0B1C2D3E4F"
	type: "custom"
	title: "Custom Reward"
	icon: "minecraft:command_block"
}
```

## Quest Links

Quest links create references to quests in other chapters, allowing you to show the same quest in multiple locations.

### Structure

```snbt
{
	id: "194E0B1C2D3E4F5A"
	linked_quest: "4D5E6F0A1B2C3D4E"
	x: 3.0d
	y: 1.5d
}
```

### Properties

| Property | Type | Required | Description |
|----------|------|----------|-------------|
| `id` | string | Yes | Unique 16-character hex ID |
| `linked_quest` | string | Yes | ID of the quest being linked |
| `x` | double | Yes | X position on quest grid |
| `y` | double | Yes | Y position on quest grid |

## Reward Tables

**Files**: `reward_tables/<table_id>.snbt`

Reward tables define collections of rewards that can be referenced by choice and random rewards.

### Structure

```snbt
{
	id: "0E4F5A6B7C8D9E0F"
	order_index: 0
	filename: "my_rewards"
	title: "Reward Table"
	icon: "minecraft:chest"
	loot_crate: {
		drops: {
			passive: 0
			monster: 0
			boss: 0
		}
		item_name: "Custom Loot Crate"
		color: 16777215
		string_id: "my_loot_crate"
		glow: false
	}
	loot_size: 1
	rewards: [
		{
			id: "1F5A6B7C8D9E0F1A"
			type: "item"
			item: "minecraft:diamond"
			weight: 10.0f
		}
		{
			id: "205A6B7C8D9E0F1A"
			type: "item"
			item: "minecraft:gold_ingot"
			count: 5
			weight: 20.0f
		}
	]
}
```

### Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `id` | string | - | Unique 16-character hex ID |
| `order_index` | int | - | Display order |
| `filename` | string | - | File basename (without .snbt) |
| `title` | string | "" | Display title |
| `icon` | string | "" | Item ID for icon |
| `loot_size` | int | 1 | Number of rewards to give |
| `loot_crate` | compound | - | Loot crate configuration (optional) |
| `rewards` | list | [] | List of rewards with weights |

### Reward Weights

Each reward in a reward table has a `weight` property (float). Higher weights make the reward more likely to be selected. The probability is calculated as:

```
probability = reward_weight / sum_of_all_weights
```

### Loot Crates

Loot crates are physical items that drop from entities and can be opened for rewards:

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `drops` | compound | - | Entity type weights |
| `item_name` | string | "" | Display name for the crate |
| `color` | int | 16777215 | RGB color (decimal) |
| `string_id` | string | - | Unique string identifier |
| `glow` | boolean | false | Item has glowing effect |

## Examples

### Example 1: Simple Linear Quest Chain

**Chapter File** (`chapters/tutorial.snbt`):

```snbt
{
	id: "1A2B3C4D5E6F0A1B"
	group: ""
	order_index: 0
	filename: "tutorial"
	title: "Tutorial"
	icon: "minecraft:writable_book"
	default_quest_shape: "circle"
	progression_mode: "linear"
	quests: [
		{
			id: "2B3C4D5E6F0A1B2C"
			title: "Welcome!"
			icon: "minecraft:grass_block"
			x: 0.0d
			y: 0.0d
			description: [
				"Welcome to the quest book!"
				"Complete this quest to get started."
			]
			tasks: [
				{
					id: "3C4D5E6F0A1B2C3D"
					type: "checkmark"
					title: "I understand"
				}
			]
			rewards: [
				{
					id: "4D5E6F0A1B2C3D4E"
					type: "item"
					item: "minecraft:apple"
					count: 5
				}
			]
		}
		{
			id: "5E6F0A1B2C3D4E5F"
			title: "Gather Wood"
			icon: "minecraft:oak_log"
			x: 2.0d
			y: 0.0d
			dependencies: ["2B3C4D5E6F0A1B2C"]
			description: ["Collect some wood to continue."]
			tasks: [
				{
					id: "6F0A1B2C3D4E5F6A"
					type: "item"
					item: "minecraft:oak_log"
					count: 16L
				}
			]
			rewards: [
				{
					id: "7A0B1C2D3E4F5A6B"
					type: "item"
					item: "minecraft:wooden_axe"
				}
			]
		}
	]
}
```

### Example 2: Quest with Multiple Task Types

```snbt
{
	id: "1B1C2D3E4F5A6B7C"
	title: "Advanced Gathering"
	icon: "minecraft:diamond"
	x: 0.0d
	y: 2.0d
	description: ["Complete all these tasks to master gathering!"]
	tasks: [
		{
			id: "2C2D3E4F5A6B7C8D"
			type: "item"
			item: "minecraft:diamond"
			count: 10L
		}
		{
			id: "0D3E4F5A6B7C8D9E"
			type: "xp"
			value: 30L
			points: false
		}
		{
			id: "1E4F5A6B7C8D9E0F"
			type: "dimension"
			dimension: "minecraft:the_nether"
		}
	]
	rewards: [
		{
			id: "2F5A6B7C8D9E0F1A"
			type: "command"
			command: "/give @p minecraft:enchanted_book 1"
			permission_level: 2
		}
	]
}
```

### Example 3: Repeatable Quest with Cooldown

```snbt
{
	id: "3A6B7C8D9E0F1A2B"
	title: "Daily Bonus"
	icon: "minecraft:gold_ingot"
	x: 4.0d
	y: 0.0d
	can_repeat: "true"
	repeat_cooldown: 86400
	description: ["Complete this quest once per day for rewards!"]
	tasks: [
		{
			id: "4B7C8D9E0F1A2B3C"
			type: "checkmark"
			title: "Claim daily reward"
		}
	]
	rewards: [
		{
			id: "5C8D9E0F1A2B3C4D"
			type: "random"
			table_id: "6D9E0F1A2B3C4D5E"
		}
	]
}
```

### Example 4: Reward Table with Weighted Rewards

**File**: `reward_tables/daily_rewards.snbt`

```snbt
{
	id: "6D9E0F1A2B3C4D5E"
	order_index: 0
	filename: "daily_rewards"
	title: "Daily Rewards"
	icon: "minecraft:chest"
	loot_size: 1
	rewards: [
		{
			id: "7E0F1A2B3C4D5E6F"
			type: "item"
			item: "minecraft:iron_ingot"
			count: 10
			weight: 50.0f
		}
		{
			id: "1F1A2B3C4D5E6F0A"
			type: "item"
			item: "minecraft:gold_ingot"
			count: 5
			weight: 30.0f
		}
		{
			id: "2A2B3C4D5E6F0A1B"
			type: "item"
			item: "minecraft:diamond"
			count: 1
			weight: 10.0f
		}
		{
			id: "0B3C4D5E6F0A1B2C"
			type: "xp_levels"
			xp_levels: 5
			weight: 10.0f
		}
	]
}
```

### Example 5: Quest with Complex Dependencies

```snbt
{
	id: "1C4D5E6F0A1B2C3D"
	title: "Master Quest"
	icon: "minecraft:nether_star"
	x: 6.0d
	y: 0.0d
	dependencies: [
		"2B3C4D5E6F0A1B2C"
		"5E6F0A1B2C3D4E5F"
		"8B1C2D3E4F5A6B7C"
	]
	dependency_requirement: "all_completed"
	min_required_dependencies: 2
	description: [
		"Complete at least 2 of the 3 prerequisite quests."
	]
	tasks: [
		{
			id: "2D5E6F0A1B2C3D4E"
			type: "checkmark"
			title: "Ready for the challenge"
		}
	]
	rewards: [
		{
			id: "3E6F0A1B2C3D4E5F"
			type: "item"
			item: "minecraft:diamond_block"
			count: 10
		}
	]
}
```

### Example 6: Chapter with Background Image

```snbt
{
	id: "4F0A1B2C3D4E5F6A"
	group: ""
	order_index: 0
	filename: "decorated_chapter"
	title: "Decorated Chapter"
	icon: "minecraft:painting"
	images: [
		{
			x: -5.0d
			y: -5.0d
			width: 10.0d
			height: 10.0d
			rotation: 0.0d
			image: "minecraft:textures/block/grass_block_side.png"
			hover: ["This is a background decoration"]
			click: ""
			dev: false
			corner: false
			fit: false
			align_to_grid: false
		}
	]
	quests: []
}
```

## Edge Cases and Troubleshooting

### ID Validation Issues

**Problem**: Quest file fails to load with ID parsing errors.

**Solutions**:
- Ensure all IDs are exactly 16 hexadecimal characters
- Check for invalid characters (only 0-9, a-f, A-F allowed)
- Avoid using reserved IDs (0000000000000000, 0000000000000001)
- Make sure every ID is unique across the entire quest file

**Example of Invalid IDs**:
```snbt
# Too short
id: "1A2B3C"

# Invalid characters
id: "1G2H3I4J5K6L7M8N"

# Reserved
id: "0000000000000000"
```

**Correct Format**:
```snbt
id: "1A2B3C4D5E6F0123"
```

### Circular Dependencies

**Problem**: Quests have circular dependencies (A depends on B, B depends on A).

**Solutions**:
- Review dependency chains to ensure they are acyclic
- Use the `min_required_dependencies` property for complex dependencies
- Consider using `dependency_requirement: "one_completed"` for branching paths

**Example**:
```snbt
# WRONG - Circular dependency
Quest A: dependencies: ["B"]
Quest B: dependencies: ["A"]

# CORRECT - Linear dependency
Quest A: dependencies: []
Quest B: dependencies: ["A"]
```

### Dependency Depth Limits

**Problem**: Dependency chains are too deep (typically >100 levels).

**Solutions**:
- Flatten your quest structure
- Break long chains into multiple shorter chains
- Use chapter groups to organize complex quest trees

### Missing References

**Problem**: Quest references a non-existent ID (dependency, reward table, etc.).

**Solutions**:
- Verify all referenced IDs exist in the quest files
- Check for typos in ID references
- Ensure reward tables are properly defined in `reward_tables/`
- Use quest links for cross-chapter dependencies instead of direct dependencies

### Item NBT Format

**Problem**: Items not loading correctly due to NBT format changes.

**Solutions**:
- For 1.21+, use the new component format:
  ```snbt
  item: {
      id: "minecraft:diamond_sword"
      count: 1
  }
  ```
- For 1.20 and earlier (legacy):
  ```snbt
  item: "minecraft:diamond_sword"
  # or
  item: {
      id: "minecraft:diamond_sword"
      Count: 1
      tag: { ... }
  }
  ```

### Tristate Values

**Problem**: Confusion about when to use tristate vs boolean properties.

**Tristate Properties** (use "default", "true", or "false"):
- `consume_items`
- `hide_dependency_lines`
- `hide_until_deps_visible`
- `hide_until_deps_complete`
- `hide_text_until_complete`
- `hide_details_until_startable`
- `disable_recipe_mod`
- `can_repeat`
- `require_sequential_tasks`

**Boolean Properties** (use true or false):
- `optional`
- `invisible`
- `hide_lock_icon`
- `hide_dependent_lines`
- `always_invisible`

### Quest Positioning

**Problem**: Quests overlap or are positioned incorrectly.

**Solutions**:
- Use grid coordinates (typically multiples of 0.5 or 1.0)
- Plan your layout on paper first
- Use `size` property to adjust quest icon size
- Consider using `images` for layout guides during development

### File Organization

**Best Practices**:
- Use descriptive filenames for chapters (e.g., `getting_started.snbt`, not `chapter1.snbt`)
- Keep related quests in the same chapter
- Use chapter groups to organize major sections
- Document your ID generation method (e.g., timestamp-based, incremental)
- Consider using comments in your text editor (SNBT doesn't support comments in files)

### Quest Book Not Loading

**Problem**: Quest book appears empty or doesn't load.

**Troubleshooting Steps**:
1. Check the log files for parsing errors
2. Verify `data.snbt` exists and has correct `version` field
3. Ensure all `.snbt` files are valid NBT format
4. Check file permissions
5. Verify chapter files match the IDs in `chapter_groups.snbt`
6. Test with a minimal quest file to isolate the issue

### Version Compatibility

**Important**: The current quest file version is **13**. If you're migrating from older versions:

- Version 13 is used by Minecraft 1.20.1+
- Some properties may have changed between versions
- Legacy item formats are supported but should be updated
- Check CHANGELOG.md for version-specific changes

### Common Mistakes

1. **Forgetting quotes around strings**: 
   ```snbt
   # WRONG
   type: item
   
   # CORRECT
   type: "item"
   ```

2. **Using wrong number suffix**:
   ```snbt
   # For long values, use L
   count: 64L
   
   # For double values, use d
   x: 1.5d
   
   # For float values, use f
   weight: 10.0f
   ```

3. **Mismatched braces**:
   - Ensure every `{` has a matching `}`
   - Ensure every `[` has a matching `]`
   - Use a text editor with bracket matching

4. **Case sensitivity**:
   - Property names are case-sensitive: `type` not `Type`
   - Item IDs are case-sensitive: `minecraft:diamond` not `Minecraft:Diamond`
   - Hex IDs are case-insensitive: `1A2B` equals `1a2b`

### Testing Your Quests

1. **Start Small**: Create a minimal chapter with one quest
2. **Validate Syntax**: Use an NBT validator or load in-game
3. **Test Dependencies**: Verify dependency chains work as expected
4. **Test Rewards**: Ensure rewards are given correctly
5. **Test Edge Cases**: Try completing quests out of order, repeating quests, etc.

### Performance Considerations

- **Optimize Quest Count**: Very large quest files (>1000 quests) may impact performance
- **Minimize Complexity**: Deeply nested dependencies can slow down dependency checking
- **Background Images**: Large or many background images can affect rendering performance
- **Task Check Frequency**: Custom tasks with low `check_timer` values may impact TPS

### Getting Help

If you encounter issues not covered in this documentation:

1. Check the [FTB Mods Issues](https://github.com/FTBTeam/FTB-Mods-Issues) repository
2. Review the [CHANGELOG.md](../CHANGELOG.md) for recent changes
3. Examine the source code in this repository for implementation details
4. Ask for help in the FTB Discord or forum

### Useful Tools

- **NBT Editors**: Tools like NBTExplorer can help visualize and edit SNBT files
- **Text Editors**: Use editors with syntax highlighting for JSON/NBT (similar syntax)
- **Version Control**: Use Git to track changes to your quest files
- **Validation Scripts**: Consider writing scripts to validate ID uniqueness and format

---

## Additional Resources

- **Source Code**: This documentation is based on FTB Quests source code
- **Wiki**: Check the FTB Wiki for user-focused tutorials
- **Community**: Join the FTB community for quest design discussions

## Contributing

If you find errors or want to contribute to this documentation, please submit a pull request or open an issue in the repository.

---

**Last Updated**: Based on FTB Quests source code as of 2025
**Version**: Quest File Format Version 13
