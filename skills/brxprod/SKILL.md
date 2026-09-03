---
name: brxprod
description: House standards for building and styling on a Bricks site running the Bricks Productivity plugin (BRXProd). Use when writing CSS or applying classes on such a site, reading the site's own design instructions, identifying which token framework it uses, applying BRXProd rails/corner/utility classes, working with the Style Guide page, verifying front-end markup inside nestable elements, or storing snippets. Says what to delegate to Bricks' own abilities and what is genuinely this plugin's. For builder notes, use the brxprod-notes skill.
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

**BRXProd used to expose `get-design-tokens`, `get-typography`, `create-page` and
`update-page`. They have been removed** — each duplicated the Bricks ability less
capably, and two tools that disagree about the same site is worse than one
because you cannot tell which is authoritative. If you find them on an older
install, do not use them.

## Rule 2: the site's own design instructions outrank this file

**Call `brxprod/get-design-instructions` early and follow what it returns.** It
is the site owner's own house rules, written by them in Settings → AI Tools, in
three sections: what to establish before building and how things are named, how
CSS is written, and where code is allowed to live.

Precedence, highest first:

1. **Bricks' own tooling** decides *how* a write happens (Rule 1).
2. **The site's design instructions** decide *what* to write.
3. **This file** covers only what neither of those has already said.

Two starting sets ship, and a site is on one of them unless the owner has edited
it. They are **opposites**, so do not assume either:

- **Code** — styling stays in CSS and never in Bricks' style controls; BEM
  names; container queries rather than `@media`.
- **Visual** — everything must land in Bricks' own controls; short `ai-`
  prefixed class names; Bricks' breakpoints, never container queries. On such a
  site the HTML/CSS import is expected to run with
  `options.custom_css_policy: "forbid"`, which makes Bricks reject any candidate
  needing a Code element or custom CSS. A `custom_css_forbidden` error there is
  the policy working as intended: revise the source, and do not switch the
  policy to `allow` without asking the owner.

**Do not infer which one applies from how the site looks.** Read the
instructions.

## Rule 3: what this plugin actually adds

Reach for BRXProd abilities only for these:

| ability | for |
|---|---|
| `brxprod/get-context` | which token framework this site runs, and what BRXProd has installed — **call this first** |
| `brxprod/get-design-instructions` | the site owner's own house rules — see Rule 2 |
| `brxprod/render-frontend-html` | the real front-end markup for a branch, including the nestable children Bricks' own render hides |
| `brxprod/find-style-guide` | locate the plugin-managed Style Guide page |
| `brxprod/get-site-js` | the site-wide custom scripts already in Bricks' settings, so you extend rather than duplicate them |
| `brxprod/list-snippets`, `install-snippet` | install one of the plugin's **bundled** snippets |
| `brxprod/create-snippet` | store JavaScript or PHP **you wrote** as a draft snippet |
| `brxprod/get-diagnostics` | server / WP / plugin diagnostics for support |

Each group is behind its own switch in **Settings → AI Tools → WordPress
Abilities**, all off by default except reads.

**If an ability you expect is missing, it is almost certainly switched off
rather than broken** — an unregistered ability and a nonexistent one look
identical from outside. `get-context` reports the group state: anything
it lists under `abilityGroups.unavailable` is off, and it names the switch. Tell
the user which one to turn on; do not work around it, and do not report a fault.

The common case is `install-snippet`, which is off by default because it is the
only ability that puts runnable code on the site. Notes is additionally Pro, so
on a free licence its switch cannot help.

There is no BRXProd ability that writes page content. That is not an oversight;
use Bricks'.

**Builder notes are a separate skill** — `brxprod-notes`. Six more abilities sit
behind it (`list-notes`, `create-note`, `update-note`, `delete-note`,
`list-note-groups`, `save-note-groups`). Nothing in this file is needed to use
them: Bricks has no notes feature, so none of the delegation rules apply there.

## Rule 4: establish which framework is in play before naming a token

A BRXProd site runs on one of three token systems, and **the same concept has a
different variable name in each**. Guessing produces CSS that references a
variable that does not exist, which resolves to nothing: the declaration is
dropped, no error is raised, and the spacing or colour is simply absent.

**`brxprod/get-context` answers this in one call** — it
reports the detected framework, the variable prefix actually in force, and what
BRXProd has installed. Prefer it over inferring the answer yourself.

If you are inferring it, read the variable list (`bricks/list-global-variables`
or `bricks/get-design-context`) and identify the system by prefix:

| you see | system |
|---|---|
| `brxw-*` (e.g. `brxw-space-l`, `brxw-grid-12`, `brxw-content-gap`) | **Bricks Wireframes** |
| unprefixed structural names (`--primary`, `--space-m`, `--gutter`, `--container`) | **Core Framework** |
| the same names under a user prefix (`--cf-space-m`) | Core Framework **with a prefix set** — follow whatever prefix is actually there |
| neither | the site's own framework, or none |

`brxp-*` is orthogonal — see Rule 5. Its presence tells you BRXProd features are
installed, not which framework the site uses. A site can have `brxw-*` and
`brxp-*` together, which is the common case.

**Never hardcode a framework's variable name into generated CSS.** Resolve the
concept against the list you just read. If the site has no variable for what you
need, say so and use a literal value — do not invent a plausible name.

**Core Framework's prefix is the user's to set.** Do not impose one, and do not
assume it is empty: third-party template libraries built for Core Framework
reference the bare names, so a prefixed install and an unprefixed one are both
normal. Read the actual names.

## Rule 5: BRXProd's own classes and variables

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
  or the "Add BRXProd features" button. `get-context` reports which of
  the three class categories are installed and lists their class names, so there
  is no need to assume.
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

## Rule 6: the managed Style Guide page

`brxprod/find-style-guide` locates it — it is identified by a marker
in post meta, not by its title, so searching for a page called "Style Guide" is
not the same question.

**Edit it with Bricks' own page abilities**, like any other page. BRXProd no
longer exposes a writer for it.

One thing to tell the user before you touch it: **the generator is
authoritative.** The plugin regenerates the page's `sg{n}-*` classes, so
hand-edits to those are replaced on the next *Update Style Guide Page*. The
supported workflow is to tune in the builder, then fold the change back into the
plugin's variant template. Edits to your own content on that page are safe.

## Rule 7: snippets

Two abilities, and they are **not** the same risk:

- **`install-snippet`** installs one of the plugin's own **bundled** snippets.
  It cannot install arbitrary code — the source is read server-side by id, and
  the caller sends only that id.
- **`create-snippet`** stores **code you wrote**. This is the one that puts your
  own output on the site, so hold it to a higher bar: say what it does and why
  it is needed before you write it.

Both land as a **draft** in the BRXProd group in Fluent Snippets, and neither can
activate anything — that is deliberate, not a limitation to work around. PHP runs
on every request and a mistake takes the site down, so enabling it is the owner's
decision. Tell them where it is, what it does, and that it is inactive.

Both require `unfiltered_html` **and** `install_plugins` — the same pair Fluent
Snippets demands of its own UI — and the snippets group is off by default.

Where another code manager is already installed (WPCode, Code Snippets,
WPCodeBox, Advanced Scripts), that is a perfectly good home: write the code and
hand it over. Do not offer to install Fluent merely because you cannot write to
theirs directly.

## Rule 8: say what you did not verify

**`brxprod/render-frontend-html` is the one ability here that does render**, and
it exists precisely because Bricks' own render paths cannot do this job: they run
in builder mode, which replaces the children of a nestable element with a
placeholder. So for anything you built inside a Nav, an accordion, tabs, a slider
or an off-canvas, Bricks' own verification shows you nothing, and this is the
check that works. Use it there.

Everything else is a write response, and a write response is not proof the result
looks right — Bricks' own guidance says the same, and it matters more here
because BRXProd's rails and corner classes are geometric.

When you finish, say plainly what you did not see, and point at what most needs a
human eye. Do not describe a page you have not viewed as looking good.
