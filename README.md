# socviz-skill

A Claude/Codex skill for data visualization with ggplot2, based on Kieran Healy's [*Data Visualization: A Practical Introduction*](https://socviz.co) (2nd ed., 2026).

Covers chart type selection, perceptual principles, layered plot construction, faceting, labeling, model visualization, maps, color palettes, and theme customization.

## What's included

- `SKILL.md` -- Main skill with workflow, geom selection guide, and anti-patterns
- `references/` -- Seven reference files covering perception, ggplot2 fundamentals, grouping/faceting, labels/annotations, model visualization, maps, and themes

## Installation

### Claude Code

Copy the skill into your Claude Code skills directory:

```bash
# Clone
git clone https://github.com/statzhero/socviz-skill.git

# Copy to your skills directory
cp -r socviz-skill ~/.claude/skills/socviz
```

Or add it directly as a single file if you only want the core skill:

```bash
mkdir -p ~/.claude/skills/socviz
curl -o ~/.claude/skills/socviz/SKILL.md \
  https://raw.githubusercontent.com/statzhero/socviz-skill/main/SKILL.md
```

### Codex

Place the skill file where Codex can find it. Codex reads markdown instructions from the repository root or a designated instructions directory:

```bash
# Option 1: Add to your project's codex instructions
mkdir -p .codex/skills
cp -r socviz-skill/* .codex/skills/socviz/

# Option 2: Reference in AGENTS.md
echo "See [socviz skill](./skills/socviz/SKILL.md) for ggplot2 visualization guidance." >> AGENTS.md
```

Codex picks up files referenced in `AGENTS.md` at the repo root. Adding SKILL.md there (or copying its contents into your instructions) makes the skill available during Codex sessions.

### Manual (any LLM)

Paste the contents of `SKILL.md` into your system prompt or attach it as context. The reference files in `references/` provide additional detail that the skill reads on demand.

## License

CC-BY-NC-ND. Based on Kieran Healy's *Data Visualization* (Princeton University Press, 2026).
