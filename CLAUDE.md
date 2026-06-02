# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HUIS UI Creator is a desktop application (Electron) for customizing Sony HUIS remote control UIs. Users can drag-and-drop buttons from multiple devices, arrange them in a personalized layout, and sync to HUIS hardware via USB.

## Tech Stack

- **Language**: TypeScript 1.8.5 (compiled to ES5)
- **Framework**: Electron 1.4.10, jQuery Mobile, Backbone.js
- **Build**: Grunt.js with custom tasks in `build/tasks/`
- **Module System**: RequireJS (not ES modules)
- **CSS**: SCSS with Compass
- **Testing**: Jasmine via Testem
- **Linting**: TSLint + JSHint

## Build Commands

```bash
grunt build         # Release build (compiles TS + SCSS to www/app/)
grunt dev_debug     # Debug build with source maps
grunt lint          # Run TSLint + JSHint
grunt ci            # Run CI tests
```

Jasmine tests: `testem ci --file tests/jasmine/testem.json`

## Architecture

### Namespace & Module Pattern

All code lives under the `Garage` namespace using TypeScript's `module` keyword (pre-ES6 style):

```typescript
module Garage {
    export module Model {
        export class Item extends Backbone.Model { ... }
    }
}
```

### MVC Structure (in `app/scripts/`)

- **Models** (`Model/`): Backbone.Model subclasses — `Item`, `ButtonItem`, `LabelItem`, `ImageItem`, `Module`, `Face`, `ButtonState`
- **Views** (`View/`): Backbone.View subclasses — `Home`, `FullCustom` (editor), `PropertyArea*` variants, `*ItemView` renderers
- **Utilities** (`Util/`): `HuisFiles` (device file I/O), `ImportManager`/`ExportManager`, `HuisConnectionChecker`, `ZipManager`
- **Interfaces** (`include/interfaces.d.ts`): Global type definitions (`IPhnConfig`, `IArea`, etc.)

### Application Flow

1. `main.js` — Electron main process, creates BrowserWindow (1280x768)
2. `app/index.html` — loads jQuery Mobile, Backbone, CDP framework
3. `app/scripts/config.ts` — RequireJS path configuration
4. Pages: Splash → Home (remote list) → FullCustom (drag-drop editor)

### Global Variables (set in `app/scripts/init.ts`)

`fs` (fs-extra), `path`, `Remote`, `app`, `Menu`, `MenuItem` (Electron), `Garage` (main namespace)

## Code Style Rules

- 4-space indentation
- Double quotes only
- Semicolons required
- Max line length: 140 characters
- Strict equality (`===`), except `== null` is allowed
- No `console.debug/info/time/trace` (only `console.log/warn/error`)

## Platform-Specific Builds

- `package_win.json` — Windows (includes `usb_dev` native USB module)
- `package_darwin.json` — macOS (no USB module)
- Enable DevTools by creating a file named `debug` in the project root
