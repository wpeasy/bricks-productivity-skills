---
name: bricks-productivity
description: House standards for working on a Bricks site running the Bricks Productivity plugin (BRXProd). Use when asked to work on such a site — building or editing pages, using design tokens, applying BRXProd rails/corner/utility classes, editing the Style Guide, reading or writing builder notes, or installing bundled snippets. Says what to delegate to Bricks' own abilities and what is genuinely this plugin's.
---

# Working on a BRXProd site

This covers **how we work**, not how Bricks works. Bricks ships its own agent
layer and that layer is the authority on Bricks itself.

## Rule 1: use Bricks' own abilities and skills first

Bricks 2.4+ ships ~176 `bricks/…` abilities and its own skills. **Read
`bricks/…`'s own start-here guidance and load the relevant Bricks skill before
you write anything.** Its ability names use slashes (`bricks/get-design-context`);
the MCP tools use hyphens (`bricks-get-design-context`).

Bricks owns all of this — do not look for a BRXProd equivalent, and do not accept
a second-hand account of it from anywhere, including this file:

| task | Bricks owns it |
|---|---|
| element structure, ids, validation, normalisation | `bricks/get-page-elements`, `bricks/get-page-structure`, element validator/normaliser |
| creating or editing page content | `bricks/checkout-page-workspace`, `bricks/commit-site-edit-plan`, changesets |
| HTML/CSS → Bricks conversion | `bricks/commit-html-css-page-import` + the `bricks-html-css-to-bricks` skill |
| global classes, variables, categories, colours | `bricks/list-global-classes`, `bricks/list-global-variables`, `bricks/batch-create-global-classes`, `bricks/list-color-palettes` |
| theme styles, typography, breakpoints, pseudo-classes | `bricks/…` design + style-manager abilities |
| design system overview | `bricks/get-design-context` |
| render verification | `bricks/render-elements` |

Its save pipeline is journaled, idempotent and resumable, and it validates what
you send. Hand-building content and pushing it some other way loses all of that.

**BRXProd's own `get-design-tokens`, `get-typography`, `create-page` and
`update-page` predate that layer and overlap it.** Prefer the Bricks ability in
every case above. The one exception is `update-page`, which is deliberately
scoped to the managed Style Guide page (see below).

## Rule 2: what this plugin actually adds

Reach for BRXProd abilities only for these:

| ability | for |
|---|---|
| `bricks-productivity/find-style-guide` | locate the plugin-managed Style Guide page |
| `bricks-productivity/list-notes`, `create-note`, `update-note`, `delete-note` | builder notes at site / page / element / user scope |
| `bricks-productivity/list-note-groups`, `save-note-groups` | the note group registry |
| `bricks-productivity/list-snippets`, `install-snippet` | install a bundled snippet into Fluent Snippets |
| `bricks-productivity/get-diagnostics` | server / WP / plugin diagnostics for support |

Each group is behind its own switch in **Settings → AI Tools → WordPress
Abilities**, all off by default except reads. If an ability is missing, the group
is off — say so rather than working around it.

## Rule 3: establish which framework is in play before naming a token

A BRXProd site runs on one of three token systems, and **the same concept has a
different variable name in each**. Guessing produces CSS that references a
variable that does not exist, which resolves to nothing: the declaration is
dropped, no error is raised, and the spacing or colour is simply absent.

Read the variable list first (`bricks/list-global-variables`, or
`bricks/get-design-context`) and identify the system by prefix:

| you see | system |
|---|---|
| `brxw-*` (e.g. `brxw-space-l`, `brxw-grid-12`, `brxw-content-gap`) | **Bricks Wireframes** |
| unprefixed structural names (`--primary`, `--space-m`, `--gutter`, `--container`) | **Core Framework** |
| the same names under a user prefix (`--cf-space-m`) | Core Framework **with a prefix set** — follow whatever prefix is actually there |
| neither | the site's own framework, or none |

`brxp-*` is orthogonal — see Rule 4. Its presence tells you BRXProd features are
installed, not which framework the site uses. A site can have `brxw-*` and
`brxp-*` together, which is the common case.

**Never hardcode a framework's variable name into generated CSS.** Resolve the
concept against the list you just read. If the site has no variable for what you
need, say so and use a literal value — do not invent a plausible name.

**Core Framework's prefix is the user's to set.** Do not impose one, and do not
assume it is empty: third-party template libraries built for Core Framework
reference the bare names, so a prefixed install and an unprefixed one are both
normal. Read the actual names.

## Rule 4: BRXProd's own classes and variables

Installed by the plugin, under fixed, readable category ids:

| class category | classes |
|---|---|
| `brxp-layout-rails` | `brxp-rails`, `brxp-rail-content`, `brxp-rail-wide`, `brxp-rail-breakout`, `brxp-rail-layout`, `brxp-rail-full`, `brxp-gutter-x`, `brxp-gutter-left`, `brxp-gutter-right`, `brxp-has-bg-media`, `brxp-has-bg-media__media` |
| `brxp-corners` | 16 classes — `brxp-outset-radius-{corner}-{horizontal\|vertical}` and `brxp-inverted-radius-{corner}-{horizontal\|vertical}` |
| `brxp-utilities` | `brxp-line-clamp`, `brxp-line-clamp--2…6`, `brxp-list-none`, the zero-margin family (`brxp-m--0`, `brxp-mi--0`, `brxp-mb--0`, `brxp-mbs--0`, `brxp-mbe--0`, `brxp-mis--0`, `brxp-mie--0`) and its padding twin (`brxp-p--0`, `brxp-pi--0`, …) |

Variables live under the `brxp-layout` category ("Design Vars"): the rails
(`--brxp-page-gutter`, `--brxp-layout-width`, `--brxp-content-width`,
`--brxp-wide-width`, `--brxp-breakout-width`), the corner pair
(`--brxp-outset-radius`, `--brxp-outset-color`, `--brxp-inverted-radius`), plus
generated accessibility text colours (`--brxp-a11y-*-text`) and animation
variables.

Three things to know before using them:

- **Check they exist on this site.** All of it is opt-in — installed by Process
  or the "Add BRXProd features" button. Read the class and variable lists rather
  than assuming.
- **The two corner families work differently.** Outset paints its fillet with a
  pseudo-element, so an element has exactly **two** slots (`-horizontal` →
  `::before`, `-vertical` → `::after`) and a third pick silently renders nothing.
  Inverted is a mask over the element itself with **no such ceiling** — all four
  corners can be on at once. For inverted, `-horizontal` and `-vertical` are
  back-compat aliases for the same corner.
- **There is no `--inverted-color`.** The inverted mask cuts a real hole showing
  the true parent, so there is nothing to fill. A value written there is read by
  nothing.

These are locked, plugin-owned classes. Hand-edits to their CSS are replaced on
the next install or Process run — if a rule needs changing, that is a plugin
change, not a site change.

## Rule 5: the managed Style Guide page

`bricks-productivity/update-page` **only works on a page carrying the plugin's
Style Guide marker** and returns 403 for anything else. That is deliberate: it
cannot be used to overwrite arbitrary pages. Do not look for a way around it —
for any other page, use Bricks' own page abilities.

Two consequences worth stating to the user before you touch it:

- **The generator is authoritative.** Hand-edits to generated `sg{n}-*` classes
  are replaced the next time the Style Guide is regenerated. The supported
  workflow is to tune in the builder, then fold the change back into the plugin's
  variant template.
- **`update-page` replaces the content array wholesale.** There is no merge.

## Rule 6: notes

Builder notes exist at four locations — `site`, `page`, `element`, `user` —
addressed by a `location` object. `postId` is required for page and element;
`elementId` for element; `userId` only when naming someone other than yourself.

- **Call `list-note-groups` before creating a note** and pick a real `groupId`,
  or it lands in `g_default`.
- **`update-note` and `delete-note` take a `noteId` only** — the location is
  resolved for you. Only the fields you send are changed, so ticking a note off
  is `{noteId, done: true}` and nothing else.
- **Prefer marking a note done over deleting it.** Deletion is permanent and the
  note body is not recoverable from the audit log, which stores only the label.
- Site notes are readable by editors but writable only by administrators. If a
  write is refused, that is the permission model, not a bug.

## Rule 7: snippets

`install-snippet` installs one of the **plugin's own bundled snippets** into
Fluent Snippets, as a **draft**, in the BRXProd group. It cannot install
arbitrary code — the source is read server-side by id.

Tell the user it landed as a draft and needs activating. It requires
`unfiltered_html`, and the whole snippets group is off by default.

## Rule 8: say what you did not verify

None of these abilities render a page. A write response is not proof the result
looks right — Bricks' own guidance says the same, and it matters more here
because BRXProd's rails and corner classes are geometric.

When you finish, say plainly what you did not see, and point at what most needs a
human eye. Do not describe a page you have not viewed as looking good.
