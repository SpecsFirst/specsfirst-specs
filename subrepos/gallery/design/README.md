# gallery — design reference

Reference mockups of the gallery's look, given by the operator on 2026-11-09. They are
part of the spec: tasks that touch the site read them, and a review compares the built
pages with them side by side.

| File | What it is |
|------|------------|
| `index.html` | Home page (mockup B1): header, intro, tag filter, grid of cards. |
| `template.html` | Template page (mockup B2): one card, tree, Copier command. |
| `logo.svg` | The SpecsFirst logo, one path filled with `currentColor`: its color is the CSS `color` of its context (the accent on the site). |

## How to read them

- **The mockups set the look**: layout, sizes, spacing, colors, fonts, and the order of
  elements. Open them directly in a browser.
- **The spec sets the behaviour and the data**: the cards, trees, counts and links of the
  mockups are static samples. Where a mockup and `../main.md` or a task disagree, the
  spec wins; where they are silent, the mockup wins.
- Known differences, on purpose, between the mockups and the site:
  - The mockups load Inter and JetBrains Mono from Google Fonts. The site serves them
    itself (no third-party service, see ADR-007 in `../../../architecture.md`).
  - Some controls are smaller than 40 px in the mockups (filter buttons, copy button).
    On the site every target is at least 40 × 40 px.
  - The mockups have no dark mode. The site keeps one, with the tokens of
    `../main.md` § Design tokens.
  - The mockups set tag colors by hand. On the site a tag's color comes from its
    position in `taxonomy.yml` (`../main.md` § Design tokens).

## Changing the design

Edit these files in a specs pull request first, then write the task that applies the
change to the site. Keep them self-contained (one HTML file each, inline CSS, no build).
