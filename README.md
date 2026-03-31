# socviz-skill

An LLM skill for data visualization with `ggplot2`, based on Kieran Healy's [*Data Visualization: A Practical Introduction*](https://socviz.co) (2nd ed., 2026, Princeton University Press).

## What does it do?

When you ask an LLM to write `ggplot2` code, it draws on its training data, which mixes good and bad practice from across the internet. This skill gives the model a structured reference. It encodes the workflow, geom selection logic, perceptual principles, and common anti-patterns from *Data Visualization* so the model can produce better plots with less back-and-forth.

The skill covers the full visualization workflow: choosing chart types for different data, mapping variables to aesthetics, grouping and faceting, labeling and annotation, visualizing model output, drawing maps, and polishing themes for publication.

## Installation

### Claude Code (one command)

Clone directly into your skills directory:

```bash
git clone https://github.com/statzhero/socviz-skill.git ~/.claude/skills/socviz
```

That's it. The skill is available immediately as `/socviz` in any Claude Code session.

If you prefer not to use the terminal, you can add skills from the Claude desktop app:

1. [Download this repository as a ZIP](https://github.com/statzhero/socviz-skill/archive/refs/heads/main.zip) from GitHub.
2. Open the Claude desktop app and switch to the **Code** tab.
3. Click **Customize** in the left sidebar, then select **Skills**.
4. Click the **+** button, choose **Upload a skill**, and select the ZIP file.

### Codex

Clone into your user skills directory (available across all projects):

```bash
git clone https://github.com/statzhero/socviz-skill.git ~/.agents/skills/socviz
```

If you prefer not to use the terminal, [download the ZIP](https://github.com/statzhero/socviz-skill/archive/refs/heads/main.zip), unzip it, and move the folder to `~/.agents/skills/socviz/` or `.agents/skills/socviz/` inside your project.

### Other LLMs

Paste the contents of `SKILL.md` into your system prompt or attach it as context. The reference files in `references/` can be appended when you need coverage of a specific topic.

## Test the skill

After installing, try this prompt in Claude Code:

```
/socviz Make a histogram of body mass from the penguins data, broken out by species
```

This is a case where base R struggles. `hist()` has no built-in way to split by group, so you end up layering semi-transparent histograms by hand. The skill should guide the model toward `facet_wrap(~ species)`.

## License

CC-BY-NC-ND. Based on Kieran Healy's *Data Visualization* (Princeton University Press, 2026).
