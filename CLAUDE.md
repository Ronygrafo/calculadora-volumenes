# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A single self-contained `index.html` file — a storage volume calculator for a B2C self-storage rental business, embedded in a Wix site via HTML Component (iframe). No build system, no package manager, no external dependencies.

## Running the App

Open `index.html` directly in a browser:

```bash
open index.html
```

No server needed. Testing is manual via browser DevTools console.

## Architecture

Everything lives in a single file:

```
index.html
  ├── <style>   — all CSS using custom properties
  ├── <body>    — 3 zones: form, items list, result
  └── <script>  — structured in labeled sections (see below)
```

### Script section order (must be preserved)

```js
// ─── CONFIGURACIÓN DEL NEGOCIO  ← business constants, modify here only
// ─── ESTADO                     ← items[] array + itemCounter
// ─── LÓGICA DE CÁLCULO          ← calcVolume, calcTotal, getRecommendation
// ─── PERSISTENCIA               ← saveSession, loadSession (sessionStorage)
// ─── RENDERIZADO                ← renderList, renderResult, escapeHtml, render
// ─── EVENTOS                    ← validateForm, onAgregar, onEliminar, onTableClick
// ─── INIT                       ← init() called at bottom
```

### Changing storage unit sizes or names

Edit only `STORAGE_UNITS` at the top of the script — the rest of the code reads from it:

```js
const STORAGE_UNITS = [
  { name: 'Bodega Pequeña', maxVolume: 3 },
  { name: 'Bodega Mediana', maxVolume: 8 },
  { name: 'Bodega Grande',  maxVolume: Infinity },
];
```

### Changing the color scheme

All colors are CSS custom properties in `:root`. `--color-accent` is the primary brand color.

## Key Behaviors

- **Volume formula:** `largo × ancho × alto (cm) ÷ 1,000,000 = m³`
- **Item counter** is incremental and never reused after deletion
- **Persistence:** `sessionStorage` only — state survives page refresh but not tab close
- **Validation:** Largo, Ancho, Alto must be numbers > 0; errors shown inline, no browser alerts
- **Empty name fallback:** auto-assigned "Item N" using the counter

## Wix Integration

Copy the full contents of `index.html` into the Wix "Embed HTML" editor. Recommended iframe height: 700–900px. Re-paste the file whenever it's updated.

## Docs

- [Design spec](docs/superpowers/specs/2026-04-30-calculadora-volumenes-design.md) — business rules, UI zones, validation table, out-of-scope items
- [Implementation plan](docs/superpowers/plans/2026-04-30-calculadora-volumenes.md) — step-by-step build tasks with verification steps per task
