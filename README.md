# Bricks Productivity skills

Agent skills for working on a WordPress site running **Bricks Builder** with the
[Bricks Productivity](https://wpeasy.au) plugin (BRXProd).

These are plain markdown. They carry YAML frontmatter so Claude Code loads them
as skills, but the frontmatter is inert to anything else — any agent, MCP client
or human can read the file directly.

## What these are for

The plugin registers a set of WordPress **abilities**, so an AI agent can work on
a site through the Abilities API / MCP rather than through the builder UI. The
abilities give an agent the tools. These skills give it the house standards:
which framework's tokens a site is using, what the BRXProd classes do, what the
Style Guide page will and will not let you overwrite.

## They delegate to Bricks first

Bricks 2.4+ ships its own agent layer — ~176 `bricks/…` abilities and its own
skills — and **that layer is the authority on Bricks itself**: element structure,
page edits, HTML/CSS conversion, global classes and variables, theme styles, the
save pipeline, render verification.

Nothing here restates any of it. A second, unversioned account of Bricks'
contract would drift on every Bricks release and would be worse than none. Rule 1
of the skill is to read Bricks' own guidance and load the relevant Bricks skill
before writing anything; the rest covers only what the plugin adds.

## Skills

| skill | covers |
|---|---|
| [`brxprod`](skills/brxprod/SKILL.md) | delegating to Bricks; identifying which token framework a site runs; BRXProd rails, corner and utility classes; the managed Style Guide page; bundled snippets |
| [`brxprod-notes`](skills/brxprod-notes/SKILL.md) | builder notes — reading, adding, editing, ticking off and deleting them at site, page, element or personal scope, and managing note groups |

**Why two and not one.** Notes is the only part of this plugin that Bricks has no
equivalent for, so it needs none of the "defer to Bricks' own abilities" material
that dominates the other file — which makes it genuinely self-contained rather
than a slice with ragged edges. Splitting there also sharpens both skills'
descriptions, and a skill is matched to a task by its description, so a vague one
loads less reliably. Everything else is one job — building and styling on a
BRXProd site — and stays together.

## Installing

Copy the skill folders into wherever your agent loads skills from. For Claude
Code, that is `.claude/skills/` in a project or `~/.claude/skills/` globally:

```bash
git clone https://github.com/wpeasy/bricks-productivity-skills.git
cp -r bricks-productivity-skills/skills/* ~/.claude/skills/
```

Take just one if that is all you need — they do not depend on each other.

Or read either `SKILL.md` directly and paste the parts you need — they are
written to be useful that way too.

## Requirements

Both skills assume the site can actually reach the abilities:

- **WordPress 6.9+** — where the Abilities API landed in core. On older
  WordPress the plugin registers nothing.
- **Bricks Productivity**, with **Settings → AI Tools → WordPress Abilities**
  turned on. It ships **off**, and each group under it (reads, snippet
  install, notes) is its own switch. Reads are on by default; snippet install
  and notes each have to be enabled deliberately, and notes needs Pro.
- **Bricks 2.4+** for Bricks' own abilities, which the skill delegates to.

If an ability a skill mentions is missing, its group is switched off. That is
the intended behaviour, not a fault — `brxprod/get-brxprod-context` reports which
groups are on, so an agent can name the switch rather than report a bug.

## Contributing

Corrections welcome, particularly anywhere a skill says something that is no
longer true of the current plugin or Bricks release. The skills name abilities
directly, so they need updating whenever that surface changes. Please cite what you
checked — these files are meant to be grounded in observed behaviour rather than
assumption, and a plausible-sounding correction that nobody verified is the
failure mode they exist to avoid.

## Licence

MIT — see [LICENSE](LICENSE). The plugin itself is licensed separately.
