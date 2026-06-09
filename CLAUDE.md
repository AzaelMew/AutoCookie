# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AutoCookie is a Cookie Clicker mod (userscript) that automates game actions. Single-file architecture: `AutoCookie.js` (~694 lines). No build system, no dependencies, no tests. Loaded via Cookie Clicker's `Game.registerMod()` API or as a userscript/bookmarklet.

## Architecture

Global `AC` object holds all state and logic, organized into namespaces:
- **`AC.Autos`** — dict of automated action instances (keyed by name)
- **`AC.AutosById`** — same actions in creation order
- **`AC.Cache`** — temporary storage
- **`AC.Data`** — static reference data (e.g. `mouseUpgrades` list)
- **`AC.Display`** — DOM/menu rendering functions
- **`AC.Game`** — copies of game functions for monkey-patching
- **`AC.Settings`** — persisted settings (saved/loaded with game state)

### Automated Actions (`AC.Auto`)

Each automated action is an `AC.Auto` instance with:
- `name`, `desc`, `timeCreated` — identity (timeCreated doubles as save-data sort key, must be unique `yyyymmddhhmm` format)
- `actionFunction` — the function run on interval, auto-bound to `this`
- Variadic settings objects after the 4th arg, each requiring `name`, `desc`, `type`, `timeCreated`, `value`
- Setting types: `header`, `switch`, `slider`, `deprecated`

`AC.Auto.prototype.run(runImmediately, interval)` — starts/stops the action. Clamps interval to slider min/max. Sets `this.intvlID`.

### Registered Actions (in creation order)

| Action | Line | Purpose |
|--------|------|---------|
| Autoclicker | L283 | Clicks cookie at interval |
| Golden Cookie Clicker | L300 | Clicks golden cookies when they appear |
| Fortune Clicker | L337 | Clicks news ticker fortunes |
| Elder Pledge Buyer | L354 | Auto-buys Elder Pledge |
| Wrinkler Popper | L386 | Pops wrinklers (with preserve logic) |
| Godzamok Loop | L428 | Sell/buy cursors for Godzamok multiplier |

### Lifecycle

1. Cookie Clicker calls `AC.init()` (registered via `Game.registerMod('AutoCookie', AC)`)
2. `AC.init()` — waits 500ms, loads save data, registers `ticker` hook, monkey-patches `Game.UpdateMenu`, shows notification
3. `AC.load(saveStr)` — restores `AC.Settings` from save, then calls `AC.start()`
4. `AC.start()` — calls `.run()` on each non-deprecated auto, picks favorite cookie
5. `AC.save()` — serializes settings for game save system
6. `AC.newsTicker()` — provides custom news messages

### Display

`AC.Display.UpdateMenu()` — called on menu refresh (injected into `Game.UpdateMenu`). Renders settings into Options menu using DOM fragments. Each auto gets a collapsible section with its settings.

## Key Conventions

- **No build step.** Edit `AutoCookie.js` directly, reload in browser.
- **No npm/node.** This is a browser-only mod loaded by Cookie Clicker.
- **`timeCreated`** format `yyyymmddhhmm` (Central Time) — used as unique ID and sort key. Must be unique across all settings and autos. Never reuse.
- **Deprecation over removal.** Set `deprecated = true` on autos or setting `type = 'deprecated'` instead of deleting. Removal breaks save data.
- **Bump version on every change.** Increment `AC.Version.AC` (minor version) in every commit that modifies `AutoCookie.js`. This lets the user confirm the mod loaded the latest code.
- **`Game` global** — Cookie Clicker's game object. Available only at runtime in browser.

## Mod Loading

Loaded by Cookie Clicker via:
- Mod Manager: `Game.LoadMod('https://azaelmew.github.io/AutoCookie/AutoCookie.js')`
- Userscript: Greasemonkey/Tampermonkey with `@include` for `orteil.dashnet.org/cookieclicker/`
- Bookmarklet: javascript URL calling `Game.LoadMod`

## Fork Context

Fork of [Elekester/AutoCookie](https://github.com/Elekester/AutoCookie). Hosted at `azaelmew.github.io/AutoCookie/AutoCookie.js`.