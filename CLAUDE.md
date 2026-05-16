# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Fishing In The Deep** is a 3D casual fishing game (playable activity/PA) where the player earns coins by catching fish across 3 rounds. The game uses basic 3D shapes (no real assets required) for the fisherman and fish.

## Core Game Loop

1. Player starts with 100 coins and can purchase upgrades (max fish capacity, max depth)
2. Player clicks Play and stops a pendulum gauge to determine hook depth
3. Hook descends to selected depth, then rises while player drags it side-to-side to catch fish
4. Caught fish are tallied with animations and converted to coins
5. After 3 rounds, the game ends

## Key Mechanics

- **Gauge system**: min-max-min pendulum with ease-in/ease-out; pointer position determines depth
- **Depth range**: min depth = max depth - 2m
- **Upgrades**: max fish (starts at 6, caps at 10) and max depth (starts at 5m, caps at 30m)
- **Fish counter**: replaces coin counter during fishing, includes a fill bar showing progress toward max capacity
- **Hook speed-up**: triggers when max fish caught, skips remaining fish on ascent
- **First-round tutorial**: 2-second pause at depth showing "move the hook to catch fish"
- **Rarity system**: rare/amazing/legendary titles; higher depth = better chance of rare fish

## Reference Documents

- `1_PRD_Fishing_In_The_Deep.md` — Full product requirements and game design specification
