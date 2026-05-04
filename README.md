# design-md-skill

> **AI agents can't see your Figma. They can read `DESIGN.md`.**
> This Claude Code skill writes one for you — in 30 seconds, from the design tokens you already have in your code.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](#license)
[![Made for Claude Code](https://img.shields.io/badge/built%20for-Claude%20Code-D97706.svg)](https://docs.claude.com/claude-code)
[![Spec: DESIGN.md alpha](https://img.shields.io/badge/spec-DESIGN.md%20alpha-6750A4.svg)](https://github.com/google-labs-code/design.md)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

---

## The problem every designer-with-AI now has

You spent weeks tuning the brand. Headline weights, the warm off-white that isn't pure white, the corner radius that's *just* soft enough, the one accent color used at most once per screen.

Then you open Claude Code, Cursor, or any other AI coding agent — and you're back to typing *"use #6750A4 for the buttons, no, slightly more purple, and the rounded corners should be 20px not 16px, and please don't use that grey on grey…"* every. single. session.

The new open standard for fixing this is **`DESIGN.md`** — a plain-text design system file from the [Google Labs `design.md` project](https://github.com/google-labs-code/design.md). One file in your repo, every AI agent stays on-brand.

But writing one by hand is tedious. **This skill writes it for you, automatically, from the design system you've already built into your code.**

---

## What this skill does

```
You:    /design-md
Claude: ✓ Found theme.json with 13 colors, 12 type sizes, 6 radii.
        ✓ Wrote DESIGN.md (24 colors, 12 typography, 15 components).
        ✓ Validated with @google/design.md lint — 0 errors.
```

That's it. No questions for non-essentials. No reverse-engineering from pixels. No filling out a form.

The skill reads what your project already has — `theme.json`, `tailwind.config.js`, your CSS variables, your `tokens.json` — and translates it into the canonical `DESIGN.md` format. Then it runs the official linter on the result so you know it's spec-conformant.

---

## Why DESIGN.md matters (90 seconds)

**For designers:** It's a single file in plain English with your colors, fonts, spacing, and component rules. Anyone — human or AI — can read it. You write it once, and from then on, every coding agent that touches the project starts from your design system, not its own guess.

**For developers:** `DESIGN.md` is YAML front-matter with typed design tokens (`colors`, `typography`, `rounded`, `spacing`, `components`) plus markdown prose for the *why*. The official `@google/design.md` CLI exports it to **Tailwind v3, Tailwind v4, or W3C DTCG `tokens.json`** — so it slots into any modern stack without lock-in.

**For everyone else:** Think of it as your brand's **README for AI**. It travels with the codebase, gets reviewed in PRs, and eliminates the *"Claude, can you make it look more on-brand"* loop.

---

## Install (60 seconds, copy-paste)

You need [Claude Code](https://docs.claude.com/claude-code) installed and a Mac/Linux/WSL terminal.

```bash
git clone https://github.com/s-a-s-k-i-a/design-md-skill ~/.claude/skills/design-md
```

Done. Open Claude Code in any project and run:

```
/design-md
```

That's the entire setup. The skill is auto-discovered — no plugin manager, no `npm install`, no build step.

> **Updating later:** `git -C ~/.claude/skills/design-md pull`

---

## How to use it

### Most projects: just run it

`cd` into your project. Run `/design-md`. Done.

The skill auto-detects what kind of project you have:

| Your project has… | Skill reads from… |
|---|---|
| `theme.json` (WordPress block theme) | The full theme.json — palette, font sizes, spacing, custom CSS variables |
| `tailwind.config.{js,ts}` (Tailwind v3) | `theme.extend` — colors, fontSize, borderRadius, spacing |
| A CSS file with `@theme { … }` (Tailwind v4) | `--color-*`, `--font-*`, `--text-*`, `--radius-*`, `--spacing-*` |
| Plain CSS with `:root { --foo }` | Your custom properties |
| `tokens.json` (W3C DTCG) | The token tree directly |
| **Nothing yet (greenfield)** | A short interview: brand name + 1-line vibe, then writes a starter |

### Tailored output

Already have a `DESIGN.md`? The skill **never silently overwrites**. It asks: overwrite, diff-only, skip, or backup-and-replace.

Multiple sub-projects in one repo (e.g. WordPress site with a custom theme)? It detects the theme directory and writes `DESIGN.md` *there* so it ships with the theme repo.

### Export to your stack

Once you have a `DESIGN.md`, the official CLI converts it to whatever you need:

```bash
# Tailwind v3 config
npx @google/design.md export --format json-tailwind DESIGN.md > tailwind.theme.json

# Tailwind v4 CSS @theme block
npx @google/design.md export --format css-tailwind DESIGN.md > theme.css

# W3C DTCG tokens.json (Figma Variables, Style Dictionary, etc.)
npx @google/design.md export --format dtcg DESIGN.md > tokens.json
```

---

## How it compares

There are several `DESIGN.md` tools in the wild — they solve different problems. Here's where this one fits:

| Tool | What it does | Input |
|---|---|---|
| [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) | Curated DESIGN.md files for famous brands (Stripe, Apple, …). Drop one in to **clone someone else's identity**. | Pre-made files |
| [VoltAgent/awesome-claude-design](https://github.com/VoltAgent/awesome-claude-design) | Skill that bundles the above for Claude Code. Same use case: **clone someone else's identity**. | Pre-made files |
| [albertzhangz10/design-system-skill](https://github.com/albertzhangz10/design-system-skill) | Generates DESIGN.md from **screenshots, PDFs, and reference images**. Vision-based. | Images, PDFs |
| [jasonhnd/design-md-generator](https://github.com/jasonhnd/design-md-generator) | Crawls a **live website URL** with Playwright and reverse-engineers tokens from rendered pixels. | URL |
| [JobYu/design-md-generator](https://github.com/JobYu/design-md-generator) | Cursor agent skill — **guided interview** flow, Cursor-specific. | Q&A interview |
| [hamen/material-3-skill](https://github.com/hamen/material-3-skill) | Pre-built Material Design 3 system. **Pick MD3 wholesale.** | Material spec |
| **`design-md-skill` (this one)** | **Translates the design tokens already in your code into a spec-conformant DESIGN.md.** No reverse-engineering, no images, no template — your single source of truth, made AI-readable. | Your project's source |

**The point:** if your design system already lives in `theme.json`, Tailwind config, or CSS variables, that's *the* truth. This skill respects it and codifies it. The other tools are great for the moments you don't have one yet (greenfield) or want to study someone else's (awesome-design-md is a fantastic study collection).

---

## What you get (sample output)

A short preview of the YAML front-matter the skill produces:

```yaml
---
version: alpha
name: <Your Brand>
colors:
  primary: "#1A1C1E"
  secondary: "#6C7278"
  tertiary: "#B8422E"
  neutral: "#F7F5F2"
typography:
  display:
    fontFamily: Public Sans
    fontSize: 4.5rem
    fontWeight: 700
    lineHeight: 1.08
    letterSpacing: -0.03em
  body-md:
    fontFamily: Public Sans
    fontSize: 1rem
    fontWeight: 400
    lineHeight: 1.6
rounded:
  sm: 4px
  md: 8px
  lg: 20px
  full: 100px
spacing:
  xs: 4px; sm: 8px; md: 16px; lg: 24px; xl: 32px
components:
  button-primary:
    backgroundColor: "{colors.tertiary}"
    textColor: "#FFFFFF"
    rounded: "{rounded.full}"
    padding: 14px 32px
---
```

…followed by a markdown body with eight sections explaining the *why*: Overview, Colors, Typography, Layout, Elevation & Depth, Shapes, Components, Do's and Don'ts. Exactly what an AI agent needs to make on-brand decisions when the rules don't cover an edge case.

---

## Features

- 🔎 **Auto-detects** project type (WordPress, Tailwind v3, Tailwind v4, plain CSS, DTCG, greenfield)
- 📐 **Spec-conformant** — uses the canonical [google-labs-code/design.md](https://github.com/google-labs-code/design.md) format
- 🧪 **Validates with the official linter** — `npx @google/design.md lint` runs automatically; errors are auto-fixed in-loop
- 💧 **Fluid-typography aware** — understands `clamp()` values and captures them sensibly
- 🛡️ **Idempotent** — never silently overwrites; always asks before replacing existing files
- 📦 **Offline-capable** — the spec is bundled; no network needed for the skill itself
- 🔓 **Zero lock-in** — output is a plain markdown file; the official CLI exports to Tailwind v3/v4 and W3C DTCG
- 🌍 **No external services** — your design system never leaves your machine

---

## Requirements

- [Claude Code](https://docs.claude.com/claude-code) installed
- macOS, Linux, or WSL with `bash`
- Node.js (only for the validation step — `npx @google/design.md lint`); skill works without it but skips validation
- That's it. No API keys, no accounts, no cloud.

---

## What it doesn't do (yet)

Honest about limits:

- **No vision.** Doesn't extract tokens from screenshots, Figma exports, or PDFs. Use [albertzhangz10/design-system-skill](https://github.com/albertzhangz10/design-system-skill) for that.
- **No URL crawling.** Doesn't reverse-engineer a live site. Use [jasonhnd/design-md-generator](https://github.com/jasonhnd/design-md-generator) for that.
- **No Figma sync** *yet*. The official CLI exports DTCG tokens.json, which Figma Variables can import — but bidirectional sync is on the roadmap.

---

## Roadmap

- [ ] Optional Figma Variables ↔ DESIGN.md round-trip via DTCG
- [ ] Diff-only mode (`/design-md --diff`) that previews changes before writing
- [ ] CI helper: a GitHub Action that flags PRs whose CSS drifts from `DESIGN.md`
- [ ] Adapter for Style Dictionary input

PRs welcome.

---

## Contributing

Issues and PRs are very welcome. The skill is intentionally simple — three files, ~200 lines of instructions — so contributions are cheap to review and merge.

If you find the skill useful, **starring the repo helps other designers and devs find it.** That's the whole pitch for the star button.

---

## Credits

- The `DESIGN.md` format and the bundled spec are © Google LLC, used under Apache-2.0.
  See [google-labs-code/design.md](https://github.com/google-labs-code/design.md).
- Skill scaffolding patterns inspired by the [hamen/material-3-skill](https://github.com/hamen/material-3-skill) and the [Anthropic Skills](https://github.com/anthropics) examples.

---

## License

MIT — see [LICENSE](LICENSE).

The bundled `reference/spec.md` is governed by Apache-2.0 (see header inside that file).
