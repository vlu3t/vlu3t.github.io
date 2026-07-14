---
title: "Procedural Dungeon Generation in Godot"
excerpt: "A graph-based dungeon generator with weighted encounter design, built in GDScript for a top-down action game."
header:
  teaser: /assets/images/procgen-teaser.png
  video:
    id: YOUR_YOUTUBE_VIDEO_ID
    provider: youtube
sidebar:
  - title: "Engine"
    text: "Godot 4.x"
  - title: "Language"
    text: "GDScript"
  - title: "Role"
    text: "Solo developer"
  - title: "Systems"
    text: "Graph generation, spatial placement, encounter design"
gallery:
  - url: assets/images/procgen2.png
    image_path: /assets/images/procgen-1.png
    alt: "Generated dungeon layout green tileset"
  - url: assets/images/procgenss.png
    image_path: /assets/images/procgen-2.png
    alt: "Generated dungeon layout purple tileset"
date: 2026-07-14
---

## Overview

Every run generates a fresh dungeon layout: rooms, corridors, loot, and enemy encounters, all built from four systems that hand off to each other in sequence.

{% include video id="YOUR_YOUTUBE_VIDEO_ID" provider="youtube" %}

The pipeline splits cleanly into four responsibilities:

1. **`RoomGraph`** decides what rooms exist and how they connect - pure topology, no world positions yet.
2. **`MapBuilder`** walks that graph breadth-first, resolving real pixel positions, spawning walls and corridors.
3. **`VoidFiller`** and **`MobSpawner`** run off the placed rooms - one dresses the world, the other populates it.
4. **`MiniMap`** reads the resulting room data to render a player-relative overview.

Each layer only needs to know about the layer below it, which makes the systems easy to tune or swap independently - I can rebalance encounters without touching room placement, or change the void tile art without touching the graph.

## Screenshots

{% include gallery caption="Generated layouts and the in-game minimap" %}

## Graph generation with backtracking

Rooms are linked on an abstract grid. For each room, I pick a random unoccupied direction and place the next room there. When all four directions around a room are already taken, generation doesn't just fail instead it backtracks to an earlier room in the chain and tries linking from there instead:

```gdscript
func backtrack(room : RoomNode, i : int) -> void:
    var iterator : int = i - 1
    while !link_bidirectional(room, nodes[i + 1]):
        if iterator <= 0:
            return
        room = nodes[iterator]
        iterator -= 1
```

This means a bad random roll degrades gracefully instead of corrupting the whole run. I also added a random branching chance that occasionally jumps the "current" room pointer backward before linking the next one, so the graph doesn't always read as one straight corridor, it grows more like an actual dungeon.

## Weighted encounter design

Now regarding the enemy spawner. Instead of spawning a fixed number of enemies per room, each room carries a weight budget, and enemies are drawn from a pool sorted by cost. A binary search finds the most expensive enemy that still fits the remaining budget:

```gdscript
func binary_search(weight_left : int) -> int:
    var left := 0
    var right := ENEMY_POOL.size() - 1
    var result := -1
    while left <= right:
        var mid : int = floor((left + right) / 2)
        if ENEMY_POOL[mid].weight <= weight_left:
            result = mid
            left = mid + 1
        else:
            right = mid - 1
    return result
```

A separate `bias_ratio` then controls whether the random pick favors cheaper or pricier enemies within the affordable range, one tunable knob that shifts a room between "many weak enemies" and "few dangerous ones" without touching any other logic. Modifiers (elite variants) are rolled as a fully separate weighted decision layered on top, so "which enemy" and "is it empowered" stay independent and easy to rebalance on their own.

## Challenges

The trickiest bug was rooms occasionally trying to generate with no valid direction left open, which without backtracking would just silently fail to place a room. Adding the backtrack step, plus pruning any orphaned nodes at the end of generation, fixed it without needing to constrain the randomness up front.
