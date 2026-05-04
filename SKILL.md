---
description: Generate or update a DESIGN.md for the current project following the google-labs-code/design.md specification. Inventories existing design inputs (theme.json, Tailwind config, CSS custom properties, DTCG tokens, brand assets), maps them to typed tokens (colors, typography, rounded, spacing, components), writes the spec-conformant prose, and validates the result with the official `@google/design.md` CLI. Use when the user asks to "create a DESIGN.md", "document the design system", "extract design tokens to DESIGN.md", "generate design.md", or "sync the visual identity into a machine-readable format".
command: design-md
user-invocable: true
allowed-tools: Read Write Bash Glob Grep WebFetch
---

# DESIGN.md Generator

Du bist eine Design-Systems-Engineer. Generiere für das **aktuelle Projekt** eine spec-konforme `DESIGN.md` nach dem Format von `google-labs-code/design.md` (Version `alpha`).

Die kanonische Spec liegt lokal unter `${CLAUDE_SKILL_DIR}/reference/spec.md` — **lies sie zu Beginn**, wenn du sie nicht im Kopf hast. Ein Skeleton-Template liegt unter `${CLAUDE_SKILL_DIR}/templates/skeleton.md`.

Der Skill funktioniert auf beliebigen Projekten: WordPress-Themes (`theme.json`), Tailwind-Repos (v3 config oder v4 `@theme {}`), Plain-CSS-Projekte mit `:root { --foo }`, DTCG `tokens.json`, oder Greenfield-Projekte ohne Inputs.

---

## Phase 1 — Discover

Identifiziere, welche Design-Inputs das Projekt hergibt. Führe die Probes parallel via Bash aus (alle in einem Block, mit `|| true` damit ein leeres Ergebnis nicht den Block abbricht):

```bash
echo "=== theme.json ===" ; find . -maxdepth 5 -name 'theme.json' -not -path '*/node_modules/*' -not -path '*/wp-content/plugins/*' -not -path '*/vendor/*' 2>/dev/null
echo "=== tailwind.config ===" ; find . -maxdepth 4 -type f \( -name 'tailwind.config.js' -o -name 'tailwind.config.ts' -o -name 'tailwind.config.mjs' -o -name 'tailwind.config.cjs' \) -not -path '*/node_modules/*' 2>/dev/null
echo "=== DTCG tokens.json ===" ; find . -maxdepth 4 -name 'tokens.json' -not -path '*/node_modules/*' 2>/dev/null
echo "=== existing DESIGN.md ===" ; find . -maxdepth 4 -iname 'DESIGN.md' -not -path '*/node_modules/*' 2>/dev/null
echo "=== CSS files with @theme or :root ===" ; grep -rl --include='*.css' -E '(@theme[[:space:]]*\{|:root[[:space:]]*\{)' . --exclude-dir=node_modules --exclude-dir=vendor --exclude-dir=wp-content/plugins 2>/dev/null | head -20
```

**Klassifiziere** das Projekt anhand der Funde (mehrere Klassen können gleichzeitig zutreffen):

- **WordPress-Theme** wenn `theme.json` vorhanden — die Datei ist die primäre Token-Quelle (`settings.color.palette`, `settings.typography.fontSizes`, `settings.spacing.spacingSizes`, `styles.*`).
- **Tailwind v3** wenn `tailwind.config.{js,ts,mjs,cjs}` vorhanden — Tokens stehen unter `theme` oder `theme.extend`.
- **Tailwind v4 / Modern CSS** wenn ein CSS-File einen `@theme { }`-Block hat — Tokens sind CSS-Custom-Properties mit Tailwind-v4-Namespaces (`--color-*`, `--font-*`, `--text-*`, `--leading-*`, `--tracking-*`, `--font-weight-*`, `--radius-*`, `--spacing-*`).
- **Plain CSS** wenn nur `:root { --foo }` ohne Tailwind-Namespaces — eigenes Naming-Schema, manuell mappen.
- **DTCG Tokens** wenn `tokens.json` mit `$type` / `$value`-Feldern — direkter Token-Import.
- **Greenfield** wenn nichts davon vorhanden ist — User nach Brand-Inputs fragen (Phase 2).

Lies die gefundenen Quellen ein. Gib dem User in 3–5 Zeilen eine Zusammenfassung dessen, was du gefunden hast, **bevor** du weitermachst (Token-Counts pro Gruppe, erkannter Projekt-Typ, ob bereits eine `DESIGN.md` existiert).

## Phase 2 — Confirm scope

Nutze `AskUserQuestion`, um vor dem Schreiben drei Dinge zu klären (kombiniere zu einem einzigen Tool-Call mit mehreren Fragen, sofern relevant):

1. **Zielpfad** — wo soll die `DESIGN.md` landen? Default-Optionen:
   - Projekt-Root des aktuellen `cwd`
   - Theme-/Subprojekt-Verzeichnis falls erkannt (z.B. `wp-content/themes/<name>/` bei WP-Sites mit eigenem Theme-Repo)
   - Custom-Pfad
2. **Existing DESIGN.md** — falls vorhanden: Overwrite / Diff (anzeigen, nicht schreiben) / Skip / In Backup-Datei umbenennen.
3. **Brand-Vibe** — nur falls Discovery keinen klaren Brand-Kontext liefert (kein CLAUDE.md, keine README, keine Brand-Assets): kurzer Brand-Name + 1–2 Adjektive für Tonfall (z.B. "Heritage, journalistisch, premium").

Bei klarem Discovery-Ergebnis (Tokens vollständig, CLAUDE.md mit Brand-Hinweisen vorhanden) **darfst du Phase 2 für Zielpfad und Vibe überspringen** und nur bei einer existierenden `DESIGN.md` nachfragen.

## Phase 3 — Extract tokens

Mappe die gefundenen Quell-Tokens auf die DESIGN.md-Token-Gruppen. **Eckpunkte**:

### colors

- Mindestens `primary` ist erforderlich. Sinnvoll: `primary`, `secondary`, `tertiary`, `neutral`, plus `surface`, `on-surface`, `on-surface-variant`, `error` wenn ableitbar.
- Wenn die Quelle nur deskriptive Slugs hat (`brand-purple`, `accent-warm`), behalte den Slug als zusätzlichen Token-Eintrag und vergib eine semantische Rolle in einem zweiten Token-Eintrag (z.B. sowohl `accent-warm: "#B8422E"` als auch `tertiary: "{colors.accent-warm}"` via Token-Reference).
- Hex-Werte in sRGB. Großschreibung beim Hex-Code spielt keine Rolle für die Spec.

### typography

- Übliches Set: `display`, `headline-lg`, `headline-md`, `headline-sm`, `body-lg`, `body-md`, `body-sm`, `label-lg`, `label-md`, `label-sm`.
- Felder pro Eintrag: `fontFamily` (string), `fontSize` (px/em/rem), `fontWeight` (number), `lineHeight` (Dimension oder unitless multiplier), `letterSpacing` (Dimension), optional `fontFeature`, `fontVariation`.
- Bei fluiden Werten (`clamp(min, ideal, max)`): wähle den **Mid- oder Max-Wert** als Token, dokumentiere das fluide Verhalten in der Prosa unter `## Typography`. (Die Spec kennt kein `clamp`-Primitive.)

### rounded

- Map: `none`, `xs`, `sm`, `md`, `lg`, `xl`, `full`. Werte als Dimension (`px`/`em`/`rem`).
- `full` typischerweise `9999px` oder `100%`.

### spacing

- Map: `xs`, `sm`, `md`, `lg`, `xl`, optional `base`, `gutter`, `margin`. Dimensionen oder unitless-Number (z.B. column-counts).
- WordPress `spacingSizes`-Array (`20`, `30`, `40`...) → mappe auf das `xs..xl`-Schema in der Reihenfolge.

### components

- Optional aber empfohlen, sobald Buttons/Cards/Inputs eine erkennbare Stil-Definition haben.
- Properties pro Component-Eintrag: `backgroundColor`, `textColor`, `typography`, `rounded`, `padding`, `size`, `height`, `width`.
- Werte können Token-References sein: `"{colors.primary}"`. Variants als separate Einträge mit `-hover`, `-active`, `-pressed`-Suffix.

**Konsistenz-Regel:** Verwende Token-References überall, wo es geht (`"{colors.tertiary}"` statt `"#B8422E"` zum zweiten Mal). Der Linter erkennt sonst Duplikate als `orphaned-tokens`.

## Phase 4 — Compose Prose

Schreibe die 8 Markdown-Sektionen in **canonical order** (siehe `reference/spec.md`):

1. **`## Overview`** (auch erlaubt: "Brand & Style") — Brand-Personality, Zielgruppe, emotionale Wirkung. 3–6 Sätze. Quelle: CLAUDE.md / README / Hero-Microcopy / Logo-Beschreibung.
2. **`## Colors`** — eine Bullet-Liste mit deskriptiven Farbnamen + Hex + Anwendungs-Hinweis pro Token (z.B. `**Primary (#1A1C1E):** Deep ink for headlines and core text`).
3. **`## Typography`** — Schriftauswahl, Hierarchie-Regeln, evtl. Hinweis auf fluides Skalieren.
4. **`## Layout`** (auch: "Layout & Spacing") — Grid-Modell, Spacing-Skala, Container-Breiten.
5. **`## Elevation & Depth`** (auch: "Elevation") — Shadow-Strategie oder erklärt, warum Flat-Design ohne Shadows.
6. **`## Shapes`** — Corner-Radius-Philosophie.
7. **`## Components`** — kurze Beschreibung jeder Variante (was sie ist, wann sie verwendet wird).
8. **`## Do's and Don'ts`** — 4–8 Bullets, harte Regeln (z.B. "Do maintain WCAG AA contrast", "Don't mix rounded and sharp corners").

**Tonfall:** sachlich-präzise, ohne Marketing-Floskeln. Schreib auf Englisch — die Spec, alle Beispiele und die LLM-Konsumenten arbeiten auf Englisch. (Auch wenn das Projekt deutsch ist.)

**Länge:** ein DESIGN.md mit ~150–250 Zeilen Markdown ist normal. Werde nicht ausschweifend.

## Phase 5 — Validate

Schreibe die finale Datei mit `Write` an den in Phase 2 bestätigten Pfad. Validiere danach via offiziellem Linter:

```bash
cd <verzeichnis-der-design-md> && npx --yes @google/design.md lint DESIGN.md
```

Parse das JSON-Output. Reagiere wie folgt:

- **Errors** (severity `error`, z.B. `broken-ref` auf nicht-existente Token-Pfade): **Pflicht-Fix.** Korrigiere die Datei und re-linte. Maximal 3 Iterationen, danach dem User berichten.
- **Warnings** (`missing-primary`, `contrast-ratio`, `orphaned-tokens`, `missing-typography`, `section-order`): Liste sie auf, **autofixe nicht stillschweigend**. Schlage Fixes als Optionen vor (insbesondere `contrast-ratio`-Warnings sind oft bewusste Designentscheidungen).
- **Info** (`token-summary`, `missing-sections`): nur erwähnen, nicht handeln.

Wenn `npx` fehlschlägt (kein Node, kein Internet): erwähne das, fahre ohne Validation fort, schlage manuelles `npm install -g @google/design.md` vor.

## Phase 6 — Report

Schließe ab mit einer kompakten Zusammenfassung an den User:

- **Pfad** der geschriebenen `DESIGN.md` (als Markdown-Link).
- **Token-Counts** pro Gruppe: `X colors, Y typography, Z rounded, N spacing, M components`.
- **Linter-Summary**: `errors: N, warnings: N, info: N`. Bei Warnings: kurze Liste mit Pfad und Empfehlung.
- **Nächste Schritte** als Anregung (kurz, eine Zeile pro Option):
  - Tailwind v3: `npx @google/design.md export --format json-tailwind DESIGN.md > tailwind.theme.json`
  - Tailwind v4: `npx @google/design.md export --format css-tailwind DESIGN.md > theme.css`
  - DTCG-Export: `npx @google/design.md export --format dtcg DESIGN.md > tokens.json`
  - Diff gegen ältere Version: `npx @google/design.md diff DESIGN.md.old DESIGN.md`

---

## Spec-Cheat-Sheet (für schnelle Referenz)

| Aspekt | Wert |
|---|---|
| Frontmatter-Delimiter | exakt `---` (keine Whitespaces, einzelne Zeile) |
| Sektions-Heading-Level | `##` (genau zwei Hashes) |
| Optionaler Doc-Title | `#` darf vor Frontmatter-Body als Titel stehen |
| Token-Reference | `"{group.token-name}"` in Quotes |
| Color-Format | `#` + sRGB-Hex (3, 6 oder 8 Zeichen) |
| Dimension-Units | `px`, `em`, `rem` (kein `vh`/`vw`/`%`!) |
| LineHeight | Dimension **oder** unitless-Number |
| Sektion-Reihenfolge | strikt: Overview → Colors → Typography → Layout → Elevation → Shapes → Components → Do's and Don'ts |
| Duplicate Headings | Hard-Error, Datei wird abgelehnt |
| Unbekannte Sections/Properties | werden akzeptiert (mit Warning) |

Die vollständige Spec mit allen Edge-Cases liegt unter `${CLAUDE_SKILL_DIR}/reference/spec.md`. Konsultiere sie bei Unsicherheit zu Token-Schema oder Linter-Verhalten.

## Idempotenz

Wenn eine `DESIGN.md` bereits existiert, **niemals stillschweigend überschreiben**. Phase 2 fängt das ab. Bei Re-Run mit identischen Quellen sollte der Output deterministisch sein (keine Random-IDs, keine Timestamps in der Datei).
