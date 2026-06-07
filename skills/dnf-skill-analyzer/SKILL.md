---
name: dnf-skill-analyzer
description: Analyze DNF (Dungeon & Fighter) skill files from PVF packages. Use this skill whenever the user asks about skill attributes, skill data, skill modification, or wants to read/parse/compare DNF skill (.skl) files. Also use when the user mentions PVF skill editing, skill parameter lookup, skill level scaling, or any task involving DNF skill mechanics — even if they don't explicitly say "analyze skill". This skill knows how to read .skl files via pvfutility MCP, parse the SKL format, interpret level scaling data, and present structured analysis.
---

# DNF Skill Analyzer

Analyze DNF (地下城与勇士) skill files stored in PVF packages using the pvfutility MCP toolset.

## When to Use

- User asks about a specific skill's attributes, damage, cooldown, level scaling
- User wants to compare skills across classes
- User wants to modify skill parameters and needs to understand the current values
- User references a skill by name (Chinese or English) or by skill code number
- User asks about skill trees, LST files, or skill file paths

## Core Workflow

### Step 1: Locate the Skill

1. Determine which class the skill belongs to. DNF has these class groups with corresponding LST files:

| Class Group | LST File | Directory |
|------------|----------|-----------|
| 男鬼剑士 (Swordman) | `skill/swordmanskill.lst` | `skill/swordman/` |
| 女鬼剑士 (ATSwordman) | `skill/atswordmanskill.lst` | `skill/atswordman/` |
| 男枪手 (Gunner) | `skill/gunnerskill.lst` | `skill/gunner/` |
| 女枪手 (ATGunner) | `skill/atgunnerskill.lst` | `skill/atgunner/` |
| 男格斗 (Fighter) | `skill/fighterskill.lst` | `skill/fighter/` |
| 女格斗 (ATFighter) | `skill/atfighterskill.lst` | `skill/atfighter/` |
| 男法师 (Mage) | `skill/mageskill.lst` | `skill/mage/` |
| 女法师 (ATMage) | `skill/atmageskill.lst` | `skill/atmage/` |
| 男圣职 (Priest) | `skill/priestskill.lst` | `skill/priest/` |
| 女圣职 (ATPriest) | `skill/atpriestskill.lst` | `skill/atpriest/` |
| 暗夜使者 (Thief) | `skill/thiefskill.lst` | `skill/thief/` |
| 守护者 (CreatorMage) | `skill/creatormage.lst` | `skill/creatormage/` |
| 魔枪士 (Demonicswordman) | `skill/demonicswordman.lst` | `skill/demonicswordman/` |

2. If the user gives a **skill code number**, read the LST file to find the skill:
   ```
   mcp__pvfutility__get_lst_file_info(file_path="skill/swordmanskill.lst")
   ```
   The LST file maps skill codes to file paths and Chinese names.

3. If the user gives a **skill name** (Chinese or English), search for it:
   ```
   mcp__pvfutility__search_pvf(keyword="鬼斩", search_folder="skill")
   ```

### Step 2: Read the SKL File

Once you have the file path, read it:
```
mcp__pvfutility__get_file_content(file_path="skill/swordman/hardattack.skl")
```

### Step 3: Parse the SKL File Format

SKL files use a section-based text format. Here are all the key sections you need to understand:

#### Basic Info Sections

| Section | Meaning | Example |
|---------|---------|---------|
| `[name]` | Skill display name (Chinese) | `` `鬼斩` `` |
| `[explain]` | Detailed skill description | Multi-line Chinese text |
| `[basic explain]` | Short description | Single line |
| `[type]` | Skill type | `[active]`, `[passive]` |
| `[skill class]` | Class category number | 1 |
| `[required level]` | Minimum level to learn | 15 |
| `[required level range]` | Level range restriction | 1 |
| `[maximum level]` | Max skill level | 30 |
| `[purchase cost]` | SP cost to learn | 50 |

#### Growtype (Class Specialization)

```
[growtype maximum level]
    0   5   0   0   0   5
```
Each number corresponds to a class advancement. Non-zero values mean that class gets bonus levels above the normal maximum.

#### Resource & Combat Sections

| Section | Meaning |
|---------|---------|
| `[consume MP]` | MP consumption: base + growth factor |
| `[cool time]` | Cooldown in milliseconds: base + per-level |
| `[auto cooltime apply]` | Whether auto-cooldown applies |
| `[durability decrease rate]` | Weapon durability loss per use |

#### Icon & Command

| Section | Meaning |
|---------|---------|
| `[icon]` | Icon sprite file + frame index (2 lines = normal + disabled) |
| `[command]` | Input command sequence |
| `[command customizing]` | Whether command can be customized |

### Step 4: Parse Dungeon vs PvP Data

SKL files have separate data blocks for dungeon and PvP:

```
[dungeon]
    [static data]
        20000  0  0  50  50  30  3500  0  7000
    [/static data]
    [level info]
        14  28  400  72  79  86  57  122  90  14  252  700  200  2000  40000
        ...  (one group per level)
    [/level info]
[/dungeon]

[pvp]
    [cool time]
        2000  2000
    [static data]
        ...
    [/static data]
    [level info]
        ...
    [/level info]
[/pvp]
```

#### Understanding [static data]

Static data contains parameters that do **NOT** change with skill level. All values form a single group. Index starts from 0.

Example: `[static data] 20000 0 0 50 50 30 3500 0 7000 [/static data]`
- Index 0: 20000
- Index 1: 0
- Index 2: 0
- Index 3: 50
- ... etc

#### Understanding [level info]

Level info contains data that **changes per skill level**. The FIRST number is the **group size** — how many values make up one level's data.

Example:
```
[level info]
    14  28  400  72  79  86  57  122  90  14  252  700  200  2000  40000
    ...
[/level info]
```
- Group size = 14 (the first number, NOT counted as data)
- Level 1 data: [28, 400, 72, 79, 86, 57, 122, 90, 14, 252, 700, 200, 2000, 40000]
- Index 0 within group = 28, Index 1 = 400, etc.

If there are 30 levels and group size is 14, there will be 30 x 14 = 420 numbers after the group size indicator.

**Simplified case** (group size = 1): Each number is one level's value directly:
```
[level info]
    1  10  30  50  70  90  ...
[/level info]
```
Level 1 = 10, Level 2 = 30, Level 3 = 50, etc.

### Step 5: Decode [level property] — The Key to Understanding Data

`[level property]` maps descriptions to the actual data values. This is the most important section for understanding what the numbers mean.

Format:
```
[level property]
    <group_count>   <reference_type>
    `<description text with <type>N</type> placeholders>`
    <source_flag>   <data_index>   <multiplier>
    ...more groups...
[/level property]
```

#### Decoding Rules

Each description line is followed by a data reference group of 3 numbers:

| Position | Name | Meaning |
|----------|------|---------|
| 1st | source_flag | **Negative** = reference `[level info]` data; **Non-negative** = reference `[static data]` |
| 2nd | data_index | Index into the referenced data array |
| 3rd | multiplier | Final value = raw_data x multiplier |

**The 1st number (source_flag) determines which array to look in:**
- If **negative** (e.g., -1): look in `[level info]` — the value changes per level
- If **non-negative** (e.g., 0, 3, 6): look in `[static data]` — the value is constant across levels

**The 2nd number (data_index) tells you which position in that array.**

**The 3rd number (multiplier) converts the raw integer to the actual game value.**

#### Worked Example

```
[level property]
    1    99
    `[里 · 鬼剑术]攻击力增加率 : <int>%%`
    -1   0   1.0
[/level property]
```

Interpretation:
1. Group count = 1, reference type = 99
2. Description: "Attack power increase rate: X%"
3. Data reference: `-1  0  1.0`
   - `-1` (negative) → read from `[level info]`
   - `0` → index 0 in each level's data group
   - `1.0` → multiply by 1.0

If dungeon level info is: `10, 30, 50, 70, 90, ...` (one value per level)
- Level 1: 10 x 1.0 = **10%**
- Level 2: 30 x 1.0 = **30%**
- Level 5: 90 x 1.0 = **90%**

#### Another Example (Multi-Group)

```
[level property]
    3    99
    `持续时间 : <float1>秒`
    -1   13   0.001
    `波动眼·天照冷却时间 : <float1>秒`
    6    6    0.001
    `波动剑·闪枪冷却时间 : <float1>秒`
    8    8    0.001
[/level property]
```

Interpretation:
1. **持续時間**: source=-1 (level info), index=13, multiplier=0.001
   - Level 1: level_info[13] x 0.001 = 40000 x 0.001 = **40秒**
2. **天照冷却**: source=6 (static data), index=6, multiplier=0.001
   - static[6] x 0.001 = 3500 x 0.001 = **3.5秒**
3. **闪枪冷却**: source=8 (static data), index=8, multiplier=0.001
   - static[8] x 0.001 = 7000 x 0.001 = **7秒**

### Step 6: Present the Analysis

Format the output as structured sections:

1. **Basic Info Table** — name, type, level requirements, SP cost, max level, growtype bonuses
2. **Resource Costs** — MP consumption, cooldown, durability
3. **Dungeon Data** — parsed static data and per-level scaling with the [level property] descriptions decoded
4. **PvP Data** — same treatment, noting differences from dungeon values
5. **Key Observations** — notable patterns (linear growth, PvP nerfs, breakpoints)

Use tables for level-by-level data when there are more than 5 levels; show key breakpoints (Lv1, Lv5, Lv10, Lv15, Lv20, Lv25, Lv30).

## Important Notes

- All SKL values are integers internally; the multiplier in [level property] converts them to meaningful units (seconds, percentages, etc.)
- Common multipliers: `0.001` for milliseconds→seconds, `1.0` for percentages
- PvP data often has severely reduced values compared to dungeon — this is intentional balance
- Some skills have empty `[command]` sections meaning they use the default input
- The `[dungeon]` block always exists; `[pvp]` block may have separate cool time and reduced values
- When group_size in level info is 1, the data is a simple flat list of values per level
- Skills with `[passive]` type typically have no cooldown and no MP cost

## Reference: pvfUtility MCP Tools

The key MCP tools for skill analysis:

| Tool | Purpose |
|------|---------|
| `get_lst_file_info` | Read skill LST to get code→name→path mapping |
| `get_file_content` | Read a .skl file's text content |
| `get_file_list` | List files in a skill directory |
| `search_pvf` | Search for skills by keyword |
| `get_pvf_root_directory` | See all available top-level directories |
| `get_pvf_pack_file_path` | Check which PVF file is currently loaded |

## Reference: pvfDoc Documentation

If deeper understanding is needed, check these docs in `D:/pvf/pvfDoc/pvfDoc/`:

| Path | Content |
|------|---------|
| `【04】技能修改/11351_教你如何查看技能文件各数据的意思.json` | How to decode static data and level info |
| `【04】技能修改/11341_鬼剑士228技能.json` | Swordman skill code reference list |
| `【04】技能修改/11343_全技能路径.json` | All skill file paths |
| `【04】技能修改/11347_改技能教学.json` | General skill modification guide |
| `【04】技能修改/11339_DAF技能PVF修改教程.json` | DAF skill modification tutorial |
| `【04】技能修改/11340_PVF技能全技能制作.json` | Full skill creation guide |
| `【04】技能修改/11350_气功师技能修改.json` | Example of specific skill mod |
| `【04】技能修改/中英技能名称对照/` | Chinese-English skill name mappings |
| `【04】技能修改/全职业技能对照表/` | Full class skill reference tables |
