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
| [`bricks-productivity`](skills/bricks-productivity/SKILL.md) | delegating to Bricks; identifying which token framework a site runs; BRXProd rails, corner and utility classes; the managed Style Guide page; builder notes; bundled snippets |

## Installing

Copy the skill folder into wherever your agent loads skills from. For Claude
Code, that is `.claude/skills/` in a project or `~/.claude/skills/` globally:

```bash
git clone https://github.com/wpeasy/bricks-productivity-skills.git
cp -r bricks-productivity-skills/skills/bricks-productivity ~/.claude/skills/
```

Or read `SKILL.md` directly and paste the parts you need — it is written to be
useful either way.

## Requirements

The skill assumes the site can actually reach the abilities:

- **WordPress 6.9+** — where the Abilities API landed in core. On older
  WordPress the plugin registers nothing.
- **Bricks Productivity**, with **Settings → AI Tools → WordPress Abilities**
  turned on. It ships **off**, and each group under it (reads, page writes,
  snippet install, notes) is its own switch. Reads are on by default; every
  write group has to be enabled deliberately.
- **Bricks 2.4+** for Bricks' own abilities, which the skill delegates to.

If an ability the skill mentions is missing, the group is switched off. That is
the intended behaviour, not a fault.

## Contributing

Corrections welcome, particularly anywhere a skill says something that is no
longer true of the current plugin or Bricks release. Please cite what you
checked — these files are meant to be grounded in observed behaviour rather than
assumption, and a plausible-sounding correction that nobody verified is the
failure mode they exist to avoid.

## Licence

MIT — see [LICENSE](LICENSE). The plugin itself is licensed separately.
