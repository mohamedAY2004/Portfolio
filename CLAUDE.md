# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page personal portfolio for Mohamed Younes. There is **no build step, no package.json, and no test suite** in this repo — it is two files served statically:

- `Portfolio.html` — the entire page. All content, layout, and styling live here. **This is the only file you normally edit.**
- `support.js` — the "DC" (Design Component) runtime bundle that renders the page. It is **generated** (`// GENERATED from dc-runtime/src/*.ts — do not edit`). The `dc-runtime` source project is not part of this repo; do not attempt to modify `support.js` by hand.

`Portfolio.html` also references `./resume-data.js`, which is **not present** in the repo (untracked/missing). The page does not depend on it — all resume content is hardcoded directly in the template — so a 404 for it is expected and harmless.

## Running / previewing

Open `Portfolio.html` in a browser, or serve the folder statically (e.g. `python -m http.server` from `d:\Portfolio`) and load `Portfolio.html`. `support.js` boots on `DOMContentLoaded`, loads React 18 UMD and Babel from `unpkg.com` CDNs at runtime, then mounts the component. **An internet connection is required** for the page to render, since React/Babel are fetched from the CDN.

There is nothing to build or lint. Validate changes by reloading the page in a browser and checking the browser console (the runtime logs `[dc]` errors there).

## How the page is structured (the DC runtime model)

`Portfolio.html` is authored declaratively inside a single `<x-dc>` element:

1. **`<helmet>`** — fonts, the `resume-data.js` script tag, and a global `<style>` block (including the mobile media queries; the mobile breakpoint is `max-width: 920px`).
2. **The template** — the visible markup, a big tree of inline-styled `<div>`s. Uses a shared CSS-variable palette declared on the root `.pf` element (`--bg`, `--surface`, `--accent: #E8893D`, `--text`, etc.). Sections in order: `hero`, `education`, `experience`, `projects`, `achievements`, `skills`, `contact` (these ids match the sidebar nav anchors).
3. **`<script type="text/x-dc" data-dc-script>`** — a `class Component extends DCLogic` (aka `StreamableLogic`). This is a React-component-like class with `componentDidMount` / `componentWillUnmount`. It drives all interactivity **imperatively** via `this.rootRef` and `root.querySelectorAll(...)`: nav highlighting, scroll reveals, the boot "typing" animation, counters, the architecture diagram, hover effects, and the mobile menu.

### Template directives the runtime supports

- `{{ expression }}` — interpolation in text and attributes (compiled by `compileAttr`).
- `ref="{{ rootRef }}"` — binds a DOM node to a `React.createRef()` field on the component.
- `<x-import from="...">` — import an external module (URL must be literal, not `{{ }}`-bound).
- `sc-if` / `sc-for` — conditional / list rendering primitives.
- `<helmet>` — head content injected into the document.

### Convention: data-* attributes as animation hooks

Much of the JS keys off custom `data-*` attributes rather than the directives above:

- `data-text="..."` — the literal is typed out character-by-character on boot; the element renders empty until then. A safety timeout at ~4.5s force-fills any `data-text` node still empty.
- `data-count` / `data-decimals` — animated number counters.
- `data-nav`, `data-square`, `data-hamburger`, `data-fill`, `data-herobody` — hooks for nav state, reveal fills, and mobile menu.

When adding content, follow these patterns (e.g. give a headline `data-text` if it should type in). Respect `prefers-reduced-motion`: the component branches to `fillBootInstant` / `revealAll` when reduced motion is set, so any new animation should have an instant fallback.

## Editing guidance

- Keep styling inline and driven by the existing CSS variables so light/dark and the accent color stay consistent.
- Any layout change needs a matching check against the `max-width:920px` mobile rules in the `<helmet>` `<style>` (the sidebar hides and a topbar/hamburger menu takes over below that width).
- Do not hand-edit `support.js`. If the runtime itself needs changing, that happens in the separate `dc-runtime` project and the bundle is regenerated there.
