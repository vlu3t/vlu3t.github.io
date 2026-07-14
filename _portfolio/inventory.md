---
title: "Data-Driven Inventory & Equipment System in Godot"
excerpt: "A modular inventory system featuring drag-and-drop equipment management, paging, and a type-safe architecture."
header:
  teaser: /assets/images/inventory-teaser.png
sidebar:
  - title: "Engine"
    text: "Godot 4.x"
  - title: "Language"
    text: "GDScript"
  - title: "Role"
    text: "Solo developer"
  - title: "Systems"
    text: "Inventory management, drag & drop UI, equipment system"
gallery:
  - url: assets/images/inv.png
    image_path: assets/images/inv.png
    alt: "Full inventory interface"

  - url: assets/images/ee.png
    image_path: assets/images/ee.png
    alt: "Inventory equipment slots"

date: 2026-07-14
---

## Overview

The inventory system is built around a single generic movement pipeline that powers every interaction between inventory pages and equipped slots. Rather than implementing separate logic for weapons, skills, summons, and consumables, each category shares the same movement code while remaining strongly typed.

The interface is designed for quick build management. Players can switch between inventory categories, drag equipment into active slots, swap items in-place, and continue collecting new items without worrying about inventory capacity.

The architecture separates gameplay data from the UI, making it easy to extend with new item categories or equipment slots without rewriting existing systems.

## Screenshots

{% include gallery caption="Inventory interface, equipment slots, and category pages" %}

## Inventory Architecture

Instead of storing every object in a single heterogeneous array, each item category owns its own typed inventory.

```text
Weapons
Active Skills
Passive Skills
Summons
Items
```

Every inventory begins with one page containing 18 slots. When the player acquires more items than the current capacity, another page is allocated automatically.

```gdscript
func add_one_page(inventory_array : Array) -> void:
    for i in range(18):
        inventory_array.append(null)
```

This approach keeps memory usage predictable while allowing inventories to grow indefinitely without changing any gameplay code.

The equipped loadout is stored separately from the inventory itself, allowing gameplay systems to reference only currently equipped abilities while the inventory acts purely as storage.

---

## Generic Item Movement

One of the goals of the system was eliminating duplicated logic.

Rather than writing different code for every interaction—inventory to equipment, equipment to inventory, inventory swapping, or equipment swapping—all transfers are handled through a single function.

```gdscript
func move_element(source_array : Array, source_index : int, source_max_size : int,
                  dest_array : Array, dest_index : int, dest_max_size : int) -> void:
```

The function automatically determines whether the operation should:

- Move an item into an empty slot
- Swap two occupied slots
- Ignore invalid operations
- Reject incompatible inventory types

Because every movement funnels through one implementation, new inventory categories automatically inherit the same behavior without requiring additional code.

---

## Type Safety

Although the movement code is generic, inventories remain strongly typed.

Before moving anything, the system verifies that both arrays contain the same resource type.

```gdscript
var source_type = source_array.get_typed_script() or source_array.get_typed_builtin()
var dest_type = dest_array.get_typed_script() or dest_array.get_typed_builtin()

if source_type != dest_type:
    printerr("Type mismatch when calling move_element")
    return
```

This prevents invalid actions such as equipping a summon into a weapon slot or placing consumable items inside the active skill inventory.

By relying on the type system instead of manual checks for every category, the implementation remains both concise and difficult to misuse.

---

## User Interface

The inventory interface is divided into category tabs across the top of the screen, allowing players to instantly switch between:

- Weapons
- Active Skills
- Passive Skills
- Summons
- Items

The currently equipped loadout is displayed along the bottom of the interface, separating permanent inventory storage from the player's active build.

The loadout consists of:

- 1 weapon slot
- 4 active skill slots
- 2 summon slots
- 2 consumable item slots

Inventory pages can be navigated independently of the equipped slots, making it possible to reorganize or replace equipment without interrupting gameplay.

The UI was designed using a consistent pixel-art visual language that matches the rest of the game's interface while emphasizing readability through strong silhouettes and high-contrast borders.

---

## Automatic Expansion

Instead of using a fixed inventory size, capacity grows only when necessary.

Whenever an item is added, the inventory searches for the last occupied slot.

```gdscript
func find_last_non_empty_index(inventory_array : Array) -> int:
    for i in range(inventory_array.size() - 1, -1, -1):
        if inventory_array[i] != null:
            return i

    return -1
```

If no free slot exists, an entirely new page is created before inserting the item.

This keeps inventories compact during the early game while allowing unlimited progression without artificial limits.

---

## Challenges

The largest challenge was designing an inventory architecture that could support every interaction without duplicating logic.

Early versions required separate functions for every movement direction, quickly leading to repetitive code and inconsistent behavior.

Refactoring the system around a single validated movement function dramatically simplified the implementation. Inventory swapping, equipping, unequipping, and hotbar management all became different uses of the same underlying algorithm.

The result is a modular inventory system where new item categories or equipment layouts can be introduced with minimal changes to the core code.
